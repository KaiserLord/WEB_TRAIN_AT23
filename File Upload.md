(Tất cả những nội dung trên này đều được lấy từ Portswigger, bạn có thể truy cập vào <https://portswigger.net/web-security/file-upload> để tìm hiểu kĩ hơn)
### 1. Lỗ hỏng file upload là gì?
- Lỗ hỏng File Upload xảy ra khi một web sever cho phép người dùng upload file lên trên file hệ thống của sever mà không xác thực đủ những thứ quan trọng như tên, loại file, nội dung file và kích thước của chúng.
- Việc không thực thi đúng cách các hạn chế đối với những điều này có nghĩa là ngay cả chức năng tải lên hình ảnh cơ bản cũng có thể được sử dụng để tải lên các tệp tùy ý và có khả năng gây nguy hiểm. Điều này thậm chí có thể bao gồm các tệp tập lệnh phía máy chủ cho phép thực thi mã từ xa.
- Trong một số trường hợp, việc tải file lên cũng đủ để gây ra những thiệt hại. Các cuộc tấn công khác có thể liên quan đến HTTP Request tiếp theo cho tệp, thường là để kích hoạt quá trình thực thi tệp của máy chủ.
### 2. Tác động của lỗ hỏng file upload
- Sự ảnh hưởng của lỗ hỏng file upload phụ thuộc vào hai yếu tố chính
  - Những thành phần của flie mà trang web không xác thực kĩ, có thể là kích thước, loại file, nội dung của file và rất nhiều thứ khác
  - Những hạn chế nào được áp dụng đối với tệp sau khi nó được tải lên thành công
- Trong trường hợp xấu nhất, loại file không được xác định kĩ và cấu hình của sever cho phép một số loại file nhất định như `.php` hoặc `.jsp` được thực thi dưới dạng mã. Trong trường hợp này, hacker có khả năng upload một đoạn mã phía máy chủ có chức năng như một web shell, cấp toàn quyền cho chúng kiểm soát toàn bộ phía máy chủ 
- Nếu như tên file không được xác thực một cách cẩn thận, nó có thể cho phép hacker ghi đè lên các file quan trọng chỉ bằng cách upload các file có cùng tên. Nếu máy chủ cũng dễ bị tấn công bởi việc truyền tải thư mục, điều này có nghĩa là hacker thậm chí cũng có thể upload các file ở những vị trí không thể lường trước
- Việc không đảm bảo kích thước của tệp nằm trong ngưỡng dự kiến ​​cũng có thể kích hoạt một dạng tấn công từ chối dịch vụ (DoS), theo đó kẻ tấn công sẽ lấp đầy dung lượng đĩa trống.
### 3. Lỗ hỏng file upload phát sinh như thế nào?
- Với những mối nguy hiểm rõ ràng, rất hiếm khi các trang web thực tế không có bất kỳ hạn chế nào về những tệp mà người dùng được phép tải lên. Thông thường, các developer triển khai những cơ chế xác thực mà họ tin là vững chắc nhưng thực chất lại có lỗi tiềm ẩn hoặc rất dễ bị qua mặt.
- Ví dụ, họ có thể cố gắng đưa các loại tệp nguy hiểm vào danh sách đen, nhưng lại không tính đến sự sai lệch khi phân tích cú pháp lúc kiểm tra phần mở rộng tệp. Giống như bất kỳ danh sách đen nào, việc vô tình bỏ sót các loại tệp ít phổ biến hơn nhưng vẫn có thể gây nguy hiểm cũng rất dễ xảy ra.
- Trong các trường hợp khác, trang web có thể cố gắng kiểm tra loại tệp bằng cách xác minh các thuộc tính mà kẻ tấn công có thể dễ dàng thao túng bằng các công cụ như Burp Proxy hoặc Repeater.
- Cuối cùng, ngay cả các biện pháp xác thực mạnh mẽ cũng có thể được áp dụng một cách thiếu nhất quán trên toàn bộ hệ thống máy chủ và thư mục tạo nên trang web, dẫn đến những sự chênh lệch có thể bị lợi dụng.
### 4. Cách máy chủ web xử lý các yêu cầu đối với tệp tĩnh như thế nào? 
- Trước khi xem xét cách khai thác lỗ hổng tải tệp lên, điều quan trọng là bạn phải hiểu cơ bản về cách máy chủ xử lý các yêu cầu đối với tệp tĩnh.
- Trước đây, các trang web hầu như chỉ bao gồm các tệp tĩnh được phục vụ cho người dùng khi có yêu cầu. Do đó, đường dẫn của mỗi yêu cầu có thể được ánh xạ tỷ lệ 1:1 với cấu trúc thư mục và tệp trên hệ thống tệp của máy chủ. Ngày nay, các trang web ngày càng mang tính động nhiều hơn và đường dẫn của một yêu cầu thường không có mối quan hệ trực tiếp nào với hệ thống tệp nữa. Mặc dù vậy, các máy chủ web vẫn xử lý các yêu cầu đối với một số tệp tĩnh, bao gồm tệp kiểu giao diện (stylesheets), hình ảnh, v.v.
- Quy trình xử lý các tệp tĩnh này phần lớn vẫn giống như trước đây. Tại một thời điểm nào đó, máy chủ sẽ phân tích cú pháp đường dẫn trong yêu cầu để xác định đuôi file(như `.jpg`,`.cpp`,`.php`). Sau đó, nó sử dụng thông tin này để xác định loại tệp đang được yêu cầu, thông thường bằng cách so sánh với một danh sách các ánh xạ được cấu hình sẵn giữa phần mở rộng và kiểu MIME (MIME types). Điều gì xảy ra tiếp theo sẽ phụ thuộc vào loại tệp và cấu hình của máy chủ.
  - Nếu loại tệp này không thể thực thi, chẳng hạn như hình ảnh hoặc trang HTML tĩnh, máy chủ có thể chỉ gửi nội dung của tệp cho client trong một HTTP Response.
  - Nếu loại tệp có thể thực thi, chẳng hạn như tệp PHP, và máy chủ được cấu hình để thực thi các tệp loại này, nó sẽ gán các biến dựa trên các header và tham số trong yêu cầu HTTP trước khi chạy tập lệnh. Kết quả đầu ra sau đó có thể được gửi cho client trong một HTTP Response.
  - Nếu loại tệp có thể thực thi, nhưng máy chủ không được cấu hình để thực thi các tệp loại này, nhìn chung nó sẽ phản hồi bằng một lỗi. Tuy nhiên, trong một số trường hợp, nội dung của tệp vẫn có thể được phục vụ cho client dưới dạng văn bản thuần túy (plain text). Những sai sót   cấu hình như vậy đôi khi có thể bị khai thác để làm rò rỉ mã nguồn và các thông tin nhạy cảm khác.
  - **Mẹo**
    - Header response `Content-type`có thể cung cấp manh mối về loại tệp mà máy chủ đã sử dụng.  Nếu header này chưa được thiết lập một cách rõ ràng bởi mã nguồn ứng dụng, nó thường sẽ chứa kết quả của việc ánh xạ giữa phần mở rộng tệp và kiểu MIME (file extension/MIME type mapping)
### 5. Khai thác lỗ hỏng từ việc tải tệp không hạn chế để triển khai mã độc web shell
- Từ góc độ bảo mật, kịch bản tồi tệ nhất có thể xảy ra là khi một website cho phép bạn tải lên các tập lệnh chạy phía máy chủ (server-side scripts), chẳng hạn như PHP, Java hoặc Python, đồng thời được cấu hình để thực thi chúng dưới dạng mã chương trình. Điều này khiến việc tạo ra một Web Shell trên máy chủ trở nên vô cùng dễ dàng.

  > Web Shell là một tập lệnh độc hại cho phép kẻ tấn công thực thi các lệnh tùy ý trên một máy chủ web từ xa, đơn giản bằng cách gửi các HTTP request đến endpoint tương ứng.
- Nếu bạn có thể tải lên Web Shell thành công, về cơ bản bạn có thể giành được quyền kiểm soát rất lớn đối với máy chủ. Điều này có nghĩa là bạn có thể:
  - Đọc và ghi các tệp tùy ý.
  - Lấy cắp dữ liệu nhạy cảm.
  - Sử dụng máy chủ làm bàn đạp để tấn công các hệ thống nội bộ và những máy chủ khác nằm ngoài mạng.
- Ví dụ, đoạn mã PHP một dòng sau có thể được sử dụng để đọc các tệp tùy ý trong hệ thống tệp của máy chủ:
  ```
  <?php echo file_get_contents('/path/to/target/file'); ?>
  ```
- Sau khi tải tệp này lên thành công, việc gửi một HTTP request đến tệp độc hại sẽ khiến nội dung của tệp mục tiêu được trả về trong HTTP response.
- Một Web Shell linh hoạt hơn có thể trông như sau:
  ```
  <?php echo system($_GET['command']); ?>
  ```
- Tập lệnh này cho phép bạn truyền một lệnh hệ thống tùy ý thông qua một tham số query như sau:
  ```
  GET /example/exploit.php?command=id HTTP/1.1
  ```
### 6. Khai thác sự thiếu sót trong việc xác thực tệp được tải lên
- Trong thực tế, rất khó để bạn tìm thấy một website hoàn toàn không có biện pháp bảo vệ trước các cuộc tấn công thông qua chức năng tải tệp lên (File Upload Attacks), giống như những gì chúng ta đã thấy trong bài thực hành trước. Tuy nhiên, việc một website có triển khai các cơ chế phòng thủ không có nghĩa là những cơ chế đó đủ mạnh và an toàn. Đôi khi, bạn vẫn có thể khai thác những điểm yếu trong các cơ chế này để đạt được Web Shell, từ đó thực hiện Remote Code Execution
  #### Xác thực loại tệp có lỗ hổng
  - Khi gửi dữ liệu thông qua các biểu mẫu HTML , trình duyệt thường gửi dữ liệu được cung cấp bằng một HTTP request có phương thức `POST`, với Content-Type là: `application/x-www-form-urlencoded`. Cách này phù hợp để gửi những dữ liệu văn bản đơn giản, chẳng hạn như tên hoặc địa chỉ của bạn.Tuy nhiên, nó không phù hợp để gửi một lượng lớn dữ liệu nhị phân (Binary Data), chẳng hạn như toàn bộ nội dung của một tệp hình ảnh hoặc tài liệu PDF.Trong trường hợp này, kiểu nội dung sau sẽ được ưu tiên sử dụng:`multipart/form-data`. Hãy tưởng tượng một biểu mẫu có chứa các trường sau:
    - Tải lên một hình ảnh.
    - Nhập mô tả cho hình ảnh đó.
    - Nhập tên người dùng (Username).
  - Khi gửi biểu mẫu như vậy, trình duyệt có thể tạo ra một HTTP request có dạng như sau:
    ```
    POST /images HTTP/1.1
    Host: normal-website.com
    Content-Length: 12345
    Content-Type: multipart/form-data; boundary=---------------------------012345678901234567890123456

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="image"; filename="example.jpg"
    Content-Type: image/jpeg

    [...binary content of example.jpg...]

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="description"

    This is an interesting description of my image.

    ---------------------------012345678901234567890123456
    Content-Disposition: form-data; name="username"

    wiener
    ---------------------------012345678901234567890123456--
    ```
  - Như bạn có thể thấy, phần thân của thông điệp HTTP (Message Body) được chia thành nhiều phần riêng biệt, tương ứng với từng trường dữ liệu trong biểu mẫu. Mỗi phần chứa một header `Content-Disposition`, cung cấp một số thông tin cơ bản về trường dữ liệu mà nó liên quan đến. Các phần riêng biệt này cũng có thể chứa header `Content-Type` của riêng chúng. Header này cho máy chủ biết kiểu MIME (MIME Type) của dữ liệu được gửi thông qua trường nhập liệu đó.
  - Một trong những cách mà website có thể sử dụng để xác thực tệp tải lên là kiểm tra xem header `Content-Type` của từng trường dữ liệu có khớp với kiểu MIME được mong đợi hay không.Ví dụ: Nếu máy chủ chỉ mong đợi các tệp hình ảnh, nó có thể chỉ cho phép những kiểu dữ liệu như:
`image/jpeg`, `image/png`. Vấn đề có thể xảy ra khi máy chủ mặc nhiên tin tưởng giá trị của header này.Nếu máy chủ không thực hiện thêm bất kỳ bước xác thực nào để kiểm tra xem nội dung thực tế của tệp có đúng với kiểu MIME được khai báo hay không, thì cơ chế bảo vệ này có thể dễ dàng bị vượt qua bằng những công cụ như Burp Repeater.
### 7. Ngăn chặn việc thực thi tệp trong các thư mục mà người dùng có thể truy cập
- Mặc dù việc ngăn chặn các loại tệp nguy hiểm được tải lên ngay từ đầu rõ ràng là giải pháp tốt hơn, nhưng tuyến phòng thủ thứ hai là ngăn máy chủ thực thi bất kỳ tập lệnh (Script) nào có thể vượt qua được các cơ chế kiểm tra ban đầu.
- Để đề phòng, các máy chủ thường chỉ thực thi những tập lệnh có kiểu MIME (MIME Type) mà chúng đã được cấu hình rõ ràng để cho phép thực thi.Trong những trường hợp khác, máy chủ có thể chỉ trả về một thông báo lỗi hoặc đôi khi hiển thị nội dung của tệp dưới dạng văn bản thuần.
  ```
  GET /static/exploit.php?command=id HTTP/1.1
    Host: normal-website.com


    HTTP/1.1 200 OK
    Content-Type: text/plain
    Content-Length: 39

    <?php echo system($_GET['command']); ?>
  ```
- Hành vi này cũng có thể mang lại một lợi ích đáng chú ý: nó có thể tạo ra cách để làm lộ mã nguồn, nhưng đồng thời vô hiệu hóa mọi nỗ lực tạo Web Shell.
- Hệ thống cấu hình kiểu này thường khác nhau giữa các thư mục.Một thư mục dùng để lưu trữ những tệp do người dùng tải lên thường sẽ có các biện pháp kiểm soát nghiêm ngặt hơn so với những vị trí khác trên hệ thống tệp (File System), vốn được giả định là nằm ngoài phạm vi truy cập của người dùng cuối.Nếu bạn có thể tìm ra cách tải một tập lệnh lên một thư mục khác, nơi không được thiết kế để chứa các tệp do người dùng cung cấp, thì cuối cùng máy chủ vẫn có thể thực thi tập lệnh đó.
  > Mẹo: Các máy chủ web thường sử dụng trường filename trong những request multipart/form-data để xác định tên và vị trí lưu trữ của tệp.
- Bạn cũng cần lưu ý rằng mặc dù bạn có thể gửi tất cả request đến cùng một tên miền, nhưng phía sau thường có một máy chủ Reverse Proxy, chẳng hạn như Load Balancer. Các request của bạn thường sẽ được xử lý bởi những máy chủ bổ sung ở phía sau hệ thống. Những máy chủ này cũng có thể được cấu hình khác nhau.

### 8. Chặn các loại tệp nguy hiểm không đầy đủ
- Một trong những cách rõ ràng nhất để ngăn người dùng tải lên các tập lệnh độc hại là đưa những phần mở rộng tệp có khả năng nguy hiểm vào danh sách đen, chẳng hạn như `.php`. Tuy nhiên, phương pháp sử dụng danh sách đen vốn có những hạn chế, bởi vì rất khó để chặn một cách rõ ràng tất cả các phần mở rộng tệp có thể được sử dụng để thực thi mã. Đôi khi, những danh sách đen này vẫn có thể bị vượt qua bằng cách sử dụng các phần mở rộng tệp thay thế ít phổ biến hơn nhưng vẫn có khả năng thực thi, chẳng hạn như:`.php5`, `.shtml` và những phần khác
  #### Ghi đè cấu hình máy chủ
  - Như đã thảo luận trong phần trước, các máy chủ thường sẽ không thực thi tệp trừ khi chúng đã được cấu hình để làm điều đó. Ví dụ, trước khi một máy chủ Apache thực thi các tệp PHP được yêu cầu bởi client, các nhà phát triển có thể cần thêm những chỉ thị sau vào tệp `/etc/apache2/apache2.conf`
  ```
  LoadModule php_module /usr/lib/apache2/modules/libphp.so
  AddType application/x-httpd-php .php
  ```
  - Nhiều máy chủ cũng cho phép các nhà phát triển tạo những tệp cấu hình đặc biệt bên trong từng thư mục riêng lẻ nhằm ghi đè hoặc bổ sung một hay nhiều thiết lập toàn cục. Ví dụ, máy chủ Apache sẽ tải cấu hình dành riêng cho từng thư mục từ một tệp có tên `.htaccess`, nếu tệp này tồn tại. Tương tự, các nhà phát triển có thể thiết lập cấu hình riêng cho từng thư mục trên máy chủ IIS bằng cách sử dụng tệp web.config. Cấu hình này có thể bao gồm những chỉ thị như sau, trong trường hợp này cho phép các tệp JSON được cung cấp cho người dùng:
  ```
  <staticContent>
    <mimeMap fileExtension=".json" mimeType="application/json" />
  </staticContent>
  ```
  - Khi những tệp cấu hình này tồn tại, máy chủ web sẽ sử dụng chúng. Tuy nhiên, thông thường bạn không được phép truy cập những tệp này thông qua các HTTP request. Mặc dù vậy, đôi khi bạn có thể tìm thấy những máy chủ không ngăn chặn được việc bạn tải lên tệp cấu hình độc hại của riêng mình. Trong trường hợp này, ngay cả khi phần mở rộng tệp mà bạn cần sử dụng nằm trong danh sách đen, bạn vẫn có thể đánh lừa máy chủ bằng cách ánh xạ một phần mở rộng tệp tùy ý, do bạn tự chỉ định, sang một kiểu MIME có khả năng thực thi.
  #### Làm rối phần mở rộng tệp
  - Ngay cả những danh sách đen toàn diện nhất cũng có khả năng bị vượt qua bằng cách sử dụng các kỹ thuật làm rối truyền thống. Giả sử mã xác thực phân biệt chữ hoa chữ thường và không nhận diện được rằng `exploit.pHp` thực chất là một tệp `.php`. Nếu đoạn mã tiếp theo chịu trách nhiệm ánh xạ phần mở rộng tệp sang kiểu MIME lại không phân biệt chữ hoa chữ thường, sự khác biệt này sẽ cho phép bạn đưa các tệp PHP độc hại vượt qua bước xác thực và những tệp này cuối cùng có thể được máy chủ thực thi.
  - Bạn cũng có thể đạt được kết quả tương tự bằng cách sử dụng những kỹ thuật sau:
    - Cung cấp nhiều phần mở rộng (Provide multiple extensions): Tùy thuộc vào thuật toán được sử dụng để phân tích tên tệp, tệp sau đây có thể được hiểu là tệp PHP hoặc hình ảnh JPG: `exploit.php.jpg`
    - Thêm các ký tự ở cuối (Add trailing characters): Một số thành phần sẽ loại bỏ hoặc bỏ qua các khoảng trắng, dấu chấm và những ký tự tương tự ở cuối tên tệp: `exploit.php.`
    - Thử sử dụng URL encoding (hoặc Double URL encoding) đối với dấu chấm, dấu gạch chéo xuôi và dấu gạch chéo ngược: Nếu giá trị không được giải mã khi xác thực phần mở rộng tệp, nhưng sau đó lại được giải mã ở phía máy chủ (Server-side), điều này cũng có thể cho phép bạn tải lên những tệp độc hại mà đáng lẽ phải bị chặn. Ví dụ: `exploit%2ephp`
    - Thêm dấu chấm phẩy hoặc các ký tự Null Byte được mã hóa URL trước phần mở rộng tệp: Nếu quá trình xác thực được viết bằng một ngôn ngữ cấp cao như PHP hoặc Java, nhưng máy chủ lại xử lý tệp bằng các hàm cấp thấp hơn trong C/C++, điều này có thể gây ra sự khác biệt trong cách xác định vị trí kết thúc của tên tệp. Ví dụ: `exploit.asp;.jpg` hoặc `exploit.asp%00.jpg`
    - Thử sử dụng các ký tự Unicode đa byte (Multibyte Unicode Characters): Những ký tự này có thể được chuyển đổi thành Null Byte và dấu chấm sau quá trình chuyển đổi hoặc chuẩn hóa Unicode (Unicode Conversion or Normalization). Các chuỗi như `xC0 x2E`, `xC4 xAE` hoặc `xC0 xAE` có thể được chuyển đổi thành `x2E` nếu tên tệp được phân tích dưới dạng chuỗi UTF-8, nhưng sau đó lại được chuyển đổi thành các ký tự ASCII trước khi được sử dụng trong đường dẫn.
    - Các biện pháp bảo vệ khác liên quan đến việc loại bỏ hoặc thay thế các phần mở rộng nguy hiểm để ngăn chặn việc thực thi tệp. Nếu phép chuyển đổi này không được áp dụng đệ quy, bạn có thể định vị chuỗi bị cấm theo cách mà việc xóa nó vẫn để lại phần mở rộng tệp hợp lệ. Ví dụ: hãy xem điều gì sẽ xảy ra nếu bạn loại bỏ .php khỏi tên tệp sau:
      > exploit.p.**php**hp
    - Đây chỉ là một phần nhỏ trong số rất nhiều cách có thể được sử dụng để làm rối phần mở rộng tệp
  #### Xác thực nội dung tệp có lỗ hổng
  - Thay vì mặc nhiên tin tưởng vào Content-Type được chỉ định trong một request, các máy chủ an toàn hơn sẽ cố gắng xác minh rằng nội dung thực tế của tệp khớp với định dạng được mong đợi.
  - Trong trường hợp chức năng tải ảnh lên, máy chủ có thể cố gắng xác minh một số thuộc tính vốn có của hình ảnh, chẳng hạn như kích thước của nó. Ví dụ: nếu bạn thử tải lên tập lệnh PHP, nó sẽ không có bất kỳ kích thước nào cả. Do đó, máy chủ có thể suy luận rằng đó không thể là hình ảnh và từ chối tải lên tương ứng.
  - Tương tự, một số loại tệp nhất định luôn chứa một chuỗi byte cụ thể trong phần đầu hoặc phần cuối của chúng. Những chuỗi byte này có thể được sử dụng giống như dấu vân tay hoặc chữ ký (Fingerprint or Signature) để xác định xem nội dung tệp có khớp với định dạng được mong đợi hay không. Ví dụ, các tệp JPEG luôn bắt đầu bằng những byte sau: `FF D8 FF`
  - Đây là một phương pháp xác thực loại tệp mạnh mẽ hơn nhiều, nhưng ngay cả phương pháp này cũng không hoàn toàn an toàn tuyệt đối. Bằng cách sử dụng các công cụ chuyên dụng như ExifTool, việc tạo ra một tệp JPEG đa định dạng (Polyglot JPEG) có chứa mã độc bên trong metadata của nó có thể trở nên rất đơn giản.
  #### Khai thác các điều kiện tranh chấp trong quá trình tải tệp lên
  - Các framework hiện đại đã được tăng cường khả năng chống lại những kiểu tấn công này. Thông thường, chúng không tải tệp trực tiếp vào vị trí đích dự kiến trên hệ thống tệp. Thay vào đó, chúng thực hiện các biện pháp phòng ngừa như:
    - Tải tệp lên một thư mục tạm thời (Temporary Directory) được cách ly (Sandboxed) trước.
    - Ngẫu nhiên hóa tên tệp để tránh ghi đè lên các tệp hiện có.
    - Sau đó, thực hiện xác thực trên tệp tạm thời này.
    - Chỉ chuyển tệp đến vị trí đích sau khi xác định rằng tệp đã an toàn.
  - Tuy nhiên, các nhà phát triển đôi khi tự triển khai quy trình xử lý tải tệp lên mà không phụ thuộc vào bất kỳ framework nào. Việc triển khai tốt quy trình này vốn đã khá phức tạp, và nó còn có thể tạo ra những Race Conditions nguy hiểm, cho phép kẻ tấn công hoàn toàn vượt qua ngay cả những cơ chế xác thực mạnh mẽ nhất.
  - Ví dụ, một số website tải tệp trực tiếp vào hệ thống tệp chính, sau đó xóa tệp đi nếu nó không vượt qua bước xác thực. Kiểu triển khai này thường xuất hiện ở những website dựa vào phần mềm diệt virus và các công cụ tương tự để kiểm tra mã độc. Quá trình này có thể chỉ mất vài mili giây, nhưng trong khoảng thời gian ngắn mà tệp tồn tại trên máy chủ, kẻ tấn công vẫn có khả năng thực thi nó.
  - Những lỗ hổng này thường cực kỳ tinh vi, khiến chúng khó bị phát hiện trong quá trình kiểm thử hộp đen, trừ khi bạn tìm được cách làm lộ mã nguồn liên quan.
  #### Race Condition trong quá trình tải tệp lên dựa trên URL
  - Các điều kiện tranh chấp tương tự cũng có thể xảy ra trong những chức năng cho phép bạn tải tệp lên bằng cách cung cấp một URL. Trong trường hợp này, máy chủ phải tải tệp từ Internet và tạo một bản sao cục bộ trước khi có thể thực hiện bất kỳ bước xác thực nào.
  - Vì tệp được tải xuống bằng HTTP, các nhà phát triển không thể sử dụng những cơ chế tích hợp sẵn trong framework của họ để xác thực tệp một cách an toàn. Thay vào đó, họ có thể phải tự xây dựng quy trình riêng để lưu trữ tạm thời và xác thực tệp. Tuy nhiên, quy trình này có thể không an toàn bằng.
  - Ví dụ, nếu tệp được tải vào một thư mục tạm thời với tên được tạo ngẫu nhiên, về mặt lý thuyết, kẻ tấn công sẽ không thể khai thác bất kỳ điều kiện tranh chấp nào. Nếu không biết tên thư mục, họ sẽ không thể gửi request đến tệp để kích hoạt việc thực thi nó. Mặt khác, nếu tên thư mục ngẫu nhiên được tạo bằng các hàm giả ngẫu nhiên như `uniqid()` của PHP, thì về lý thuyết, tên thư mục này có thể bị dò tìm bằng phương pháp vét cạn.
  - Để khiến những cuộc tấn công như vậy trở nên dễ dàng hơn, bạn có thể thử kéo dài thời gian xử lý tệp, từ đó mở rộng khoảng thời gian có thể dùng để dò tìm tên thư mục bằng phương pháp Brute-force. Một cách để thực hiện điều này là tải lên một tệp có kích thước lớn hơn. Nếu tệp được xử lý theo từng khối, bạn có thể tận dụng điều này bằng cách tạo một tệp độc hại có chứa payload ở phần đầu, theo sau là một lượng lớn các byte đệm tùy ý.
### 9. Khai thác lỗ hổng tải tệp lên mà không cần thực thi mã từ xa
Trong những ví dụ mà chúng ta đã tìm hiểu cho đến nay, chúng ta có thể tải lên các tập lệnh phía máy chủ (Server-side Scripts) để thực hiện thực thi mã từ xa (Remote Code Execution). Đây là hậu quả nghiêm trọng nhất của một chức năng tải tệp lên không an toàn, nhưng những lỗ hổng này vẫn có thể bị khai thác theo những cách khác.
  #### Tải lên các tập lệnh độc hại phía client
  - Mặc dù bạn có thể không thực thi được các tập lệnh trên máy chủ, nhưng bạn vẫn có thể tải lên những tập lệnh phục vụ cho các cuộc tấn công phía client . Ví dụ, nếu bạn có thể tải lên các tệp HTML hoặc hình ảnh SVG, bạn có thể sử dụng các thẻ `<script>` để tạo ra những payload XSS được lưu trữ.
  - Nếu tệp đã tải lên sau đó xuất hiện trên một trang web được những người dùng khác truy cập, trình duyệt của họ sẽ thực thi tập lệnh khi cố gắng hiển thị trang đó. Lưu ý rằng do các hạn chế của chính sách cùng nguồn gốc , những kiểu tấn công này chỉ hoạt động nếu tệp được tải lên được phân phối từ cùng một origin với origin mà bạn đã tải tệp lên.
  #### Khai thác các lỗ hổng trong quá trình phân tích cú pháp tệp đã tải lên
  - Nếu tệp đã tải lên dường như được lưu trữ và phân phối một cách an toàn, phương án cuối cùng là thử khai thác những lỗ hổng đặc thù liên quan đến quá trình phân tích cú pháp (Parsing) hoặc xử lý các định dạng tệp khác nhau. Ví dụ, nếu bạn biết máy chủ phân tích cú pháp các tệp dựa trên XML, chẳng hạn như những tệp Microsoft Office .doc hoặc .xls, thì đây có thể là một hướng khai thác tiềm năng đối với các cuộc tấn công XML External Entity (XXE) Injection.
### 10. Tải tệp lên bằng phương thức PUT
- Cần lưu ý rằng một số máy chủ web có thể được cấu hình để hỗ trợ các request sử dụng phương thức `PUT`. Nếu không có những biện pháp phòng thủ phù hợp, điều này có thể cung cấp một phương thức thay thế để tải lên các tệp độc hại, ngay cả khi chức năng tải tệp lên không có sẵn thông qua giao diện web.
  ```
  PUT /images/exploit.php HTTP/1.1
  Host: vulnerable-website.com
  Content-Type: application/x-httpd-php
  Content-Length: 49
  
  <?php echo file_get_contents('/path/to/file'); ?>
  ```
- Bạn có thể thử gửi các request `OPTIONS` đến những endpoint khác nhau để kiểm tra xem chúng có công bố hỗ trợ phương thức `PUT` hay không.

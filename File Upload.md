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
- Từ góc nhìn bảo mật, trường hợp tệ nhất là khi một trang web chấp nhận tải lên đoạn mã phía sever như PHP,JAVA hoặc là PYTHON và cũng được cấu hình để thực thi chúng dưới dạng mã. Điều này khiến việc tạo web shell của riêng bạn trên máy chủ trở nên đơn giản.(*Web shell* là một đoạn mã độc cho phép kẻ tấn công thực thi các lệnh tùy ý trên một máy chủ web từ xa chỉ bằng cách gửi các HTTP Request đến đúng endpoint.)
- Nếu bạn có thể tải lên thành công một web shell, bạn sẽ có được quyền kiểm soát toàn bộ máy chủ. Điều này có nghĩa là bạn có thể đọc và ghi các tệp tùy ý, đánh cắp dữ liệu nhạy cảm, thậm chí sử dụng máy chủ để chuyển hướng tấn công (pivot) vào cả cơ sở hạ tầng nội bộ lẫn các máy chủ khác nằm ngoài mạng. 
### 6. Khai thác lỗ hỏng từ việc xác thực thiếu xót của file upload
- Trong thực tế sẽ không có những trang web mà không thực hiện các biện pháp bảo vệ để chống lại tấn công file upload giống như những bài trước. Mặc dù có biện pháp nhưng điều đó không đồng nghĩa với việc những biện pháp đó đủ mạnh để chống lại những cuộc tấn công đó. Thi thoảng bạn cũng có thể khai thác các lỗ hổng trong các cơ chế này để lấy web shell nhằm thực thi mã từ xa.
#### Thiếu xác thực theo loại tệp
- Khi gửi biểu mẫu HTML, trình duyệt thường gửi dữ liệu được cung cấp theo kiểu `POST` Request với Content Type `application/x-www-form-urlencoded`. Điều này hợp lệ với việc gửi những văn bản đơn giản như tên hoặc địa chỉ của bạn. Tuy nhiên, nó không phù hợp cho việc gửi lượng lớn dữ liệu nhị phân, chẳng hạn như toàn bộ tệp hình ảnh hoặc tài liệu PDF. Trong trường hợp này, Content Type `multipart/form-data` được ưu tiên.
- Hãy xem xét một biểu mẫu chứa các trường để tải hình ảnh lên, cung cấp mô tả về hình ảnh đó và nhập tên người dùng của bạn. Việc gửi biểu mẫu như vậy có thể dẫn đến yêu cầu trông giống như thế này:
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
- Như bạn có thể thấy, nội dung của request được chia thành các phần riêng biệt cho từng thông tin đầu vào của biểu mẫu. Mỗi phần chứa một tiêu đề `Content-Disposition`, cung cấp một số thông tin cơ bản về trường đầu vào mà nó liên quan đến. Các phần riêng lẻ này cũng có thể chứa tiêu đề `Content-Type` của riêng chúng, tiêu đề này cho máy chủ biết loại MIME của dữ liệu đã được gửi bằng cách sử dụng đầu vào này.
- Một cách mà các trang web có thể cố gắng xác thực việc tải tệp lên là kiểm tra xem tiêu đề `Content-Type` dành riêng cho đầu vào này có khớp với loại MIME dự kiến ​​hay không. Ví dụ: nếu máy chủ chỉ cho phép các tệp hình ảnh, thì nó có thể chỉ cho phép các loại như `image/jpeg` và `image/png`. Vấn đề có thể xuất hiện khi mà giá trị của tiêu đề hoàn toàn được sever tin tưởng. Nếu không thực hiện xác thực thêm để kiểm tra xem nội dung của tệp có thực sự khớp với loại MIME hay không,việc phòng thủ này có thể dễ dàng bị vượt qua bằng cách sử dụng các công cụ như Burp Repeater.

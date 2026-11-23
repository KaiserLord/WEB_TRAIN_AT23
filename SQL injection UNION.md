### 1. SQL injection UNION attacks
- Khi một ứng dụng dễ bị tấn công SQL injection và kết quả của truy vấn được trả về trong phản hồi của ứng dụng, bạn có thể sử dụng từ khóa UNION để truy xuất dữ liệu từ các bảng khác bên trong cơ sở dữ liệu. Kỹ thuật này thường được gọi là tấn công SQL injection UNION.
- Từ khóa UNION cho phép bạn thực thi một hoặc nhiều truy vấn SELECT bổ sung và nối kết quả của chúng vào truy vấn ban đầu. Ví dụ:
  ```
  SELECT a, b FROM table1 UNION SELECT c, d FROM table2
  ```
- Truy vấn SQL này trả về một tập kết quả duy nhất với hai cột, chứa các giá trị từ cột `a` và `b` trong `table1` và các cột `c` và `d` trong `table2`
- Để một truy vấn `UNION` hoạt động, cần đáp ứng hai yêu cầu quan trọng:
  - Các truy vấn riêng lẻ phải trả về cùng một số lượng cột.
  - Kiểu dữ liệu trong mỗi cột phải tương thích với nhau giữa các truy vấn riêng lẻ.
- Để thực hiện một cuộc tấn công SQL injection UNION, hãy đảm bảo rằng cuộc tấn công của bạn đáp ứng được hai yêu cầu này. Điều này thường liên quan đến việc xác định:
  - Có bao nhiêu cột được trả về từ truy vấn ban đầu.
  - Những cột nào được trả về từ truy vấn ban đầu có kiểu dữ liệu phù hợp để chứa kết quả từ truy vấn được chèn vào.
### 2. Xác định số lượng cột cần thiết
- Khi thực hiện một cuộc tấn công SQL Injection UNION, có hai phương pháp hiệu quả để xác định có bao nhiêu cột được trả về từ truy vấn ban đầu.
- Một phương pháp là chèn một loạt các mệnh đề `ORDER BY` và tăng dần chỉ số cột được chỉ định cho đến khi xảy ra lỗi. Ví dụ, nếu vị trí chèn là một chuỗi được đặt trong dấu nháy đơn bên trong mệnh đề `WHERE` của truy vấn ban đầu, bạn sẽ gửi:
  ```
  ' ORDER BY 1--
  ' ORDER BY 2--
  ' ORDER BY 3--
  etc.
  ```
- Chuỗi payload này thay đổi truy vấn ban đầu để sắp xếp kết quả theo các cột khác nhau trong tập kết quả. Cột trong mệnh đề ORDER BY có thể được chỉ định bằng chỉ số của nó, vì vậy bạn không cần biết tên của bất kỳ cột nào. Khi chỉ số cột được chỉ định vượt quá số lượng cột thực tế trong tập kết quả, cơ sở dữ liệu sẽ trả về lỗi, chẳng hạn như:
  ```
  The ORDER BY position number 3 is out of range of the number of items in the select list.
  ```
- Ứng dụng thực tế có thể trả về lỗi cơ sở dữ liệu trong HTTP Response, nhưng cũng có thể chỉ đưa ra một thông báo lỗi chung chung. Trong những trường hợp khác, ứng dụng có thể đơn giản là không trả về bất kỳ kết quả nào. Dù bằng cách nào, miễn là bạn có thể phát hiện một số khác biệt trong phản hồi, bạn có thể suy ra có bao nhiêu cột được trả về từ truy vấn.
- Phương pháp thứ hai là gửi một loạt payload UNION SELECT với số lượng giá trị NULL khác nhau:
  ```
  ' UNION SELECT NULL--
  ' UNION SELECT NULL,NULL--
  ' UNION SELECT NULL,NULL,NULL--
  etc.
  ```
- Nếu số lượng giá trị NULL không khớp với số lượng cột, cơ sở dữ liệu sẽ trả về lỗi, chẳng hạn như:
  ```
  All queries combined using a UNION, INTERSECT or EXCEPT operator must have an equal number of expressions in their target lists.
  ```
- Chúng ta sử dụng NULL làm các giá trị được trả về từ truy vấn `SELECT` được chèn vào bởi vì kiểu dữ liệu trong mỗi cột phải tương thích giữa truy vấn ban đầu và truy vấn được chèn vào. `NULL` có thể chuyển đổi thành mọi kiểu dữ liệu phổ biến, do đó nó tối đa hóa khả năng payload thành công khi số lượng cột chính xác.
- Tương tự như kỹ thuật `ORDER BY`, ứng dụng thực tế có thể trả về lỗi cơ sở dữ liệu trong HTTP Responses, nhưng cũng có thể trả về lỗi chung chung hoặc đơn giản là không có kết quả nào.
- Khi số lượng giá trị NULL khớp với số lượng cột, cơ sở dữ liệu sẽ trả về thêm một hàng trong tập kết quả, chứa các giá trị NULL ở mỗi cột. Ảnh hưởng của điều này đến phản hồi HTTP phụ thuộc vào mã nguồn của ứng dụng. Nếu may mắn, bạn sẽ thấy một số nội dung bổ sung trong phản hồi, chẳng hạn như một hàng bổ sung trong bảng HTML. Nếu không, các giá trị NULL có thể gây ra một lỗi khác, chẳng hạn như `NullPointerException`. Trong trường hợp xấu nhất, phản hồi có thể trông giống hệt như phản hồi được tạo ra bởi số lượng giá trị NULL không chính xác. Điều này sẽ khiến phương pháp này không hiệu quả.
### 3. Cú pháp đặc thù của từng cơ sở dữ liệu
- Trên Oracle, mọi truy vấn `SELECT` đều phải sử dụng từ khóa `FROM` và chỉ định một bảng hợp lệ. Oracle có một bảng được tích hợp sẵn tên là `dual`, có thể được sử dụng cho mục đích này. Vì vậy, các truy vấn được chèn vào trên Oracle cần có dạng:
  ```
  ' UNION SELECT NULL FROM DUAL--
  ```
- Các payload được mô tả sử dụng chuỗi chú thích hai dấu gạch ngang `--` để chú thích phần còn lại của truy vấn ban đầu nằm sau vị trí chèn. Trên MySQL, chuỗi hai dấu gạch ngang phải được theo sau bởi một dấu cách. Ngoài ra, ký tự thăng `#` cũng có thể được sử dụng để xác định một chú thích.
- Để biết thêm chi tiết về cú pháp đặc thù của từng cơ sở dữ liệu, hãy xem tài liệu: https://portswigger.net/web-security/sql-injection/cheat-sheet
### 4. Tìm các cột có kiểu dữ liệu hữu ích
- Một cuộc tấn công SQL injection UNION cho phép bạn truy xuất kết quả từ một truy vấn được chèn vào. Dữ liệu quan trọng mà bạn muốn truy xuất thường ở dạng chuỗi. Điều này có nghĩa là bạn cần tìm một hoặc nhiều cột trong kết quả của truy vấn ban đầu có kiểu dữ liệu là chuỗi hoặc tương thích với dữ liệu dạng chuỗi.
- Sau khi xác định được số lượng cột cần thiết, bạn có thể kiểm tra từng cột để xác định liệu nó có thể chứa dữ liệu dạng chuỗi hay không. Bạn có thể gửi một loạt payload `UNION SELECT` đặt một giá trị chuỗi vào từng cột theo thứ tự. Ví dụ, nếu truy vấn trả về bốn cột, bạn sẽ gửi:
  ```
  ' UNION SELECT 'a',NULL,NULL,NULL--
  ' UNION SELECT NULL,'a',NULL,NULL--
  ' UNION SELECT NULL,NULL,'a',NULL--
  ' UNION SELECT NULL,NULL,NULL,'a'--
  ```
- Nếu kiểu dữ liệu của cột không tương thích với dữ liệu dạng chuỗi, truy vấn được chèn vào sẽ gây ra lỗi cơ sở dữ liệu, chẳng hạn như:
  ```
  Conversion failed when converting the varchar value 'a' to data type int.
  ```
- Nếu không xảy ra lỗi và phản hồi của ứng dụng chứa một số nội dung bổ sung bao gồm giá trị chuỗi được chèn vào, thì cột tương ứng phù hợp để truy xuất dữ liệu dạng chuỗi.
### 5. Sử dụng tấn công SQL injection UNION để truy xuất dữ liệu quan trọng
- Khi bạn đã xác định được số lượng cột được trả về từ truy vấn ban đầu và tìm ra những cột nào có thể chứa dữ liệu dạng chuỗi, bạn đã có đủ điều kiện để truy xuất những dữ liệu quan trọng.
- Giả sử rằng:
  - Truy vấn ban đầu trả về hai cột và cả hai cột đều có thể chứa dữ liệu dạng chuỗi.
  - Vị trí chèn là một chuỗi được đặt trong dấu nháy đơn bên trong mệnh đề `WHERE`.
  - Cơ sở dữ liệu chứa một bảng có tên `users` với các cột `username` và `password`.
- Trong ví dụ này, bạn có thể truy xuất nội dung của bảng `users` bằng cách gửi dữ liệu đầu vào:
  ```
  ' UNION SELECT username, password FROM users--
  ```
- Để thực hiện cuộc tấn công này, bạn cần biết rằng tồn tại một bảng có tên `users` với hai cột có tên `username` và `password`. Nếu không có thông tin này, bạn sẽ phải đoán tên của các bảng và cột. Tất cả các cơ sở dữ liệu hiện đại đều cung cấp những cách thức để kiểm tra cấu trúc cơ sở dữ liệu và xác định các bảng cũng như những cột mà chúng chứa.
### 6. Đưa nhiều giá trị vào trong một cột duy nhất
- Trong một số trường hợp, truy vấn ở ví dụ trước có thể chỉ trả về một cột.
- Bạn có thể truy xuất nhiều giá trị cùng lúc trong một cột duy nhất bằng cách nối các giá trị đó lại với nhau.
- Bạn có thể thêm một ký tự phân cách để dễ dàng phân biệt các giá trị đã được kết hợp. Ví dụ, trên Oracle, bạn có thể gửi dữ liệu đầu vào:
  ```
  ' UNION SELECT username || '~' || password FROM users--
  ```
- Cách này sử dụng chuỗi hai dấu gạch đứng `||`, đây là toán tử nối chuỗi trong Oracle. Truy vấn được chèn vào sẽ nối các giá trị của hai trường `username` và `password` lại với nhau, được phân cách bằng ký tự `~`.
- Kết quả từ truy vấn sẽ chứa tất cả tên người dùng và mật khẩu, ví dụ:
  ```
  ...
  administrator~s3cure
  wiener~peter
  carlos~montoya
  ...
  ```
- Cú pháp nối chuỗi khác nhau tùy theo từng cơ sở dữ liệu. Để biết thêm chi tiết, hãy xem tài liệu: https://portswigger.net/web-security/sql-injection/cheat-sheet

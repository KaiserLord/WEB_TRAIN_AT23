## Đây là bản dịch từ tiếng việt sang tiếng anh về SQL Injection cơ bản của PortSwigger

### 1. SQL Injection là gì?
- SQL injection (SQLi) là một lỗ hổng bảo mật web cho phép kẻ tấn công can thiệp vào các truy vấn mà ứng dụng thực hiện đối với cơ sở dữ liệu. Điều này có thể cho phép kẻ tấn công xem những dữ liệu mà họ thường không thể truy xuất. Những dữ liệu này có thể bao gồm dữ liệu thuộc về những người dùng khác hoặc bất kỳ dữ liệu nào khác mà ứng dụng có thể truy cập. Trong nhiều trường hợp, kẻ tấn công có thể sửa đổi hoặc xóa những dữ liệu này, gây ra những thay đổi lâu dài đối với nội dung hoặc hành vi của ứng dụng.
- Trong một số tình huống, kẻ tấn công có thể nâng cấp một cuộc tấn công SQL injection để chiếm quyền kiểm soát máy chủ nền tảng hoặc cơ sở hạ tầng back-end khác. Nó cũng có thể cho phép họ thực hiện các cuộc tấn công từ chối dịch vụ (denial-of-service attacks).
### 2. Tác động của một cuộc tấn công SQL injection thành công là gì?
- Một cuộc tấn công SQL injection thành công có thể dẫn đến việc truy cập trái phép vào dữ liệu nhạy cảm, chẳng hạn như:
    - Mật khẩu.
    - Thông tin thẻ tín dụng.
    - Thông tin cá nhân của người dùng.
- Các cuộc tấn công SQL injection đã được sử dụng trong nhiều vụ rò rỉ dữ liệu nghiêm trọng trong suốt những năm qua. Những vụ việc này đã gây ra thiệt hại về danh tiếng và các khoản phạt theo quy định. Trong một số trường hợp, kẻ tấn công có thể có được một backdoor tồn tại lâu dài vào hệ thống của tổ chức, dẫn đến việc hệ thống bị xâm phạm trong thời gian dài mà có thể không bị phát hiện.
### 3. Cách phát hiện các lỗ hổng SQL injection
- Bạn có thể phát hiện SQL injection thủ công bằng cách thực hiện một tập hợp các kiểm tra có hệ thống đối với mọi điểm nhập dữ liệu trong ứng dụng. Để thực hiện điều này, thông thường bạn sẽ gửi:
  - Ký tự dấu nháy đơn `'` và tìm kiếm lỗi hoặc những dấu hiệu bất thường khác.
  - Một số cú pháp đặc thù của SQL có thể đánh giá thành giá trị gốc (ban đầu) của điểm nhập dữ liệu và một giá trị khác, sau đó tìm kiếm những khác biệt có tính hệ thống trong các phản hồi của ứng dụng.
  - Các điều kiện Boolean như `OR 1=1` và `OR 1=2`, sau đó tìm kiếm sự khác biệt trong các phản hồi của ứng dụng.
  - Các payload được thiết kế để kích hoạt độ trễ thời gian khi được thực thi bên trong một truy vấn SQL, sau đó tìm kiếm sự khác biệt về thời gian phản hồi.
  - Các payload OAST được thiết kế để kích hoạt một tương tác mạng out-of-band khi được thực thi bên trong một truy vấn SQL, đồng thời theo dõi mọi tương tác phát sinh.
- Ngoài ra, bạn có thể nhanh chóng và đáng tin cậy tìm ra phần lớn các lỗ hổng SQL injection bằng Burp Scanner.
### 4. SQL injection trong các phần khác nhau của truy vấn
- Phần lớn các lỗ hổng SQL injection xảy ra bên trong mệnh đề WHERE của một truy vấn SELECT. Hầu hết những người kiểm thử có kinh nghiệm đều quen thuộc với loại SQL injection này.
- Tuy nhiên, các lỗ hổng SQL injection có thể xảy ra tại bất kỳ vị trí nào bên trong truy vấn và trong nhiều loại truy vấn khác nhau. Một số vị trí phổ biến khác mà SQL injection có thể xuất hiện là:
    - Trong các câu lệnh UPDATE, bên trong những giá trị được cập nhật hoặc mệnh đề WHERE.
    - Trong các câu lệnh INSERT, bên trong những giá trị được chèn vào.
    - Trong các câu lệnh SELECT, bên trong tên bảng hoặc tên cột.
    - Trong các câu lệnh SELECT, bên trong mệnh đề ORDER BY.
### 5. Các ví dụ về SQL injection
- Có rất nhiều lỗ hổng SQL injection, các cuộc tấn công và kỹ thuật có thể xảy ra trong những tình huống khác nhau. Một số ví dụ SQL injection phổ biến bao gồm:
    - Truy xuất dữ liệu ẩn: trong đó bạn có thể sửa đổi một truy vấn SQL để trả về những kết quả bổ sung.
    - Phá vỡ logic của ứng dụng: trong đó bạn có thể thay đổi một truy vấn để can thiệp vào logic của ứng dụng.
    - Các cuộc tấn công UNION: trong đó bạn có thể truy xuất dữ liệu từ các bảng cơ sở dữ liệu khác nhau.
    - Blind SQL injection: trong đó kết quả của một truy vấn mà bạn kiểm soát không được trả về trong các phản hồi của ứng dụng.
### 6. Truy xuất dữ liệu ẩn
- Hãy tưởng tượng một ứng dụng mua sắm hiển thị các sản phẩm thuộc những danh mục khác nhau. Khi người dùng nhấp vào danh mục Gifts, trình duyệt của họ gửi yêu cầu đến URL:
  ```
  https://insecure-website.com/products?category=Gifts
  ```
- Điều này khiến ứng dụng thực hiện một truy vấn SQL để truy xuất thông tin chi tiết về những sản phẩm liên quan từ cơ sở dữ liệu:
  ```
  SELECT * FROM products WHERE category = 'Gifts' AND released = 1
  ```
- Truy vấn SQL này yêu cầu cơ sở dữ liệu trả về:
    - Tất cả thông tin chi tiết (`*`)
    - Từ bảng `products`
    - Trong đó category là `Gifts`
    - Và `released` là `1`.
- Điều kiện `released = 1` được sử dụng để ẩn những sản phẩm chưa được phát hành. Chúng ta có thể giả định rằng đối với các sản phẩm chưa được phát hành, `released = 0`.
- Ứng dụng không triển khai bất kỳ biện pháp phòng vệ nào chống lại các cuộc tấn công SQL injection. Điều này có nghĩa là kẻ tấn công có thể xây dựng cuộc tấn công sau, chẳng hạn:
  ```
  https://insecure-website.com/products?category=Gifts'--
  ```
- Điều này tạo ra câu truy vấn SQL:
  ```
  SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
  ```
- Điều quan trọng cần lưu ý là `--` là ký hiệu bắt đầu chú thích trong SQL. Điều này có nghĩa là phần còn lại của truy vấn được xem là một chú thích, qua đó loại bỏ phần đó khỏi truy vấn. Trong ví dụ này, điều đó có nghĩa là truy vấn không còn bao gồm `AND released = 1`. Kết quả là tất cả sản phẩm đều được hiển thị, bao gồm cả những sản phẩm chưa được phát hành.
- Bạn có thể sử dụng một cuộc tấn công tương tự để khiến ứng dụng hiển thị tất cả sản phẩm trong bất kỳ danh mục nào, bao gồm cả những danh mục mà ứng dụng không biết đến:
  ```
  https://insecure-website.com/products?category=Gifts'+OR+1=1--
  ```
- Điều này tạo ra truy vấn SQL:
  ```
  SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
  ```
- Truy vấn đã sửa đổi trả về tất cả các mục mà `category` là `Gifts`, hoặc `1` bằng `1`. Vì `1=1` luôn đúng, truy vấn trả về tất cả các mục.
- **Cảnh báo:** Hãy cẩn thận khi chèn điều kiện `OR 1=1` vào một truy vấn SQL. Ngay cả khi nó có vẻ vô hại trong ngữ cảnh mà bạn đang chèn vào, các ứng dụng thường sử dụng dữ liệu từ một yêu cầu duy nhất trong nhiều truy vấn khác nhau. Nếu điều kiện của bạn tác động đến một câu lệnh `UPDATE` hoặc `DELETE`, chẳng hạn, nó có thể dẫn đến việc mất dữ liệu ngoài ý muốn.
### 7. Phá vỡ logic của ứng dụng
- Hãy tưởng tượng một ứng dụng cho phép người dùng đăng nhập bằng tên người dùng và mật khẩu. Nếu người dùng gửi tên người dùng `wiener` và mật khẩu `bluecheese`, ứng dụng sẽ kiểm tra thông tin xác thực bằng cách thực hiện truy vấn SQL sau:
  ```
  SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
  ```
- Nếu truy vấn trả về thông tin chi tiết của một người dùng, việc đăng nhập sẽ thành công. Ngược lại, nó sẽ bị từ chối.
- Trong trường hợp này, kẻ tấn công có thể đăng nhập với tư cách bất kỳ người dùng nào mà không cần mật khẩu. Họ có thể thực hiện điều này bằng cách sử dụng chuỗi chú thích SQL `--` để loại bỏ phần kiểm tra mật khẩu khỏi mệnh đề `WHERE` của truy vấn. Ví dụ, việc gửi tên người dùng `administrator'--` và một mật khẩu trống sẽ tạo ra truy vấn sau:
  ```
  SELECT * FROM users WHERE username = 'administrator'--' AND password = ''
  ```
- Truy vấn này trả về người dùng có `username` là `administrator` và đăng nhập thành công với tư cách người dùng đó.
### 8. Truy xuất dữ liệu từ các bảng cơ sở dữ liệu khác
- Trong những trường hợp ứng dụng phản hồi bằng kết quả của một truy vấn SQL, kẻ tấn công có thể sử dụng lỗ hổng SQL injection để truy xuất dữ liệu từ những bảng khác bên trong cơ sở dữ liệu. Bạn có thể sử dụng từ khóa `UNION` để thực thi một truy vấn `SELECT` bổ sung và nối kết quả của nó vào truy vấn ban đầu.
- Ví dụ, nếu một ứng dụng thực thi truy vấn sau có chứa đầu vào của người dùng `Gifts`:
  ```
  SELECT name, description FROM products WHERE category = 'Gifts'
  ```
- Kẻ tấn công có thể gửi đầu vào:
  ```
  ' UNION SELECT username, password FROM users--
  ```
- Điều này khiến ứng dụng trả về tất cả tên người dùng và mật khẩu cùng với tên và mô tả của các sản phẩm.

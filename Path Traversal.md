### 1. Path Traversal là gì?
- Path Traversal hay còn được biết với tên gọi khác là Directory Traversal. Lỗ hỏng này cho phép hacker đọc bất kì file trên sever đang chạy ứng dụng. Những thứ đó có thể là:
  - Code ứng dụng và dữ liệu
  - Xác thực của hệ thống back-end (các dữ liệu hoặc bằng chứng số dùng để xác minh danh tính của người dùng như mật khẩu, API Keys,....)
  - Các dữ liệu nhạy cảm của hệ điều hành
- Trong một vài trường hợp, hacker có thể viết bất kì file trên sever, cho phép sửa đổi dữ liệu hoặc hành vi của ứng dụng và thậm chí có thể kiểm soát toàn bộ hệ thống
### 2. Đọc các tập tin tuỳ ý thông qua Path Traversal
- Hãy tưởng tượng một ứng dụng mua sắm hiển thị hình ảnh các mặt hàng được rao bán. Điều này có thể tải hình ảnh bằng HTML sau: `<img src="/loadImage?filename=218.png">`
-  URL `loadImage` nhận một tham số `filename` và trả về nội dung của tệp được chỉ định. File hình ảnh được lưu trữ trên đĩa ở vị trí `/var/www/images/`. Để trả về một hình ảnh, ứng dụng sẽ thêm tên tệp được yêu cầu vào thư mục cơ sở này và dùng một filesystem API để đọc nội dung của file. Nói cách khác, ứng dụng đọc từ đường dẫn tệp sau: `/var/www/images/218.png`
-  Ứng dụng này thực hiện không có biện pháp phòng vệ chống lại tấn công Path Traversal. Kết quả là, hacker có thể yêu cầu URL sau để truy xuất file `/etc/passwd` từ file hệ thống của máy chủ:
    ```
    https://insecure-website.com/loadImage?filename=../../../etc/passwd
    ```
- Điều này khiến ứng dụng đọc từ đường dẫn tệp sau: `/var/www/images/../../../etc/passwd`
- Chuỗi `../` là một thành phần hợp lệ trong đường dẫn file, có nghĩa là di chuyển lên một cấp trong cấu trúc thư mục. Ba chuỗi `../` liên tiếp sẽ đưa đường dẫn đi lên từ `/var/www/images/` đến thư mục gốc của hệ thống file. Vì vậy, file thực sự được đọc là: `/etc/passwd`
- Trên các hệ điều hành dựa trên Unix, nó là file tiêu chuẩn chứa thông tin của người dùng đã được đăng kí trên sever, nhưng hacker có thể truy xuất các tệp tùy ý khác bằng kỹ thuật tương tự. Trên Window, cả `../` và `../` đều là các chuỗi truyền tải thư mục hợp lệ. Sau đây là ví dụ về một cuộc tấn công tương đương nhằm vào máy chủ chạy Windows: `https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini`
### 3. Những trở ngại chung khi khai thác lỗ hỏng Path Traversal
- Nhiều ứng dụng đưa dữ liệu do người dùng cung cấp vào đường dẫn tệp đều triển khai các biện pháp phòng chống path traversal. Những điều này thường có thể được bỏ qua.
- Nếu một ứng dụng loại bỏ hoặc chặn các chuỗi vượt qua thư mục khỏi tên tệp do người dùng cung cấp, bạn có thể vượt qua lớp phòng thủ này bằng cách sử dụng nhiều kỹ thuật khác nhau. Bạn có thể sử dụng một đường dẫn tuyệt đối từ thư mục gốc của hệ thống tệp, chẳng hạn như `filename=/etc/passwd`, để tham chiếu trực tiếp đến một tệp mà không cần sử dụng bất kỳ chuỗi vượt qua thư mục nào
- Bạn có thể sử dụng các chuỗi vượt qua lồng nhau, chẳng hạn như `....//` hoặc `....\/`. Những chuỗi này sẽ quay trở lại thành các chuỗi vượt qua đơn giản khi chuỗi bên trong bị loại bỏ

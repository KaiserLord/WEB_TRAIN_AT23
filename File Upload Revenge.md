## I.KIẾN THỨC NỀN TẢNG VỀ FILE UPLOAD
### 1. File Upload là gì?
- File Upload là chức năng cho phép người dùng gửi một tệp từ máy tính của mình lên máy chủ của website.
- Ví dụ, bạn truy cập một website có chức năng cập nhật ảnh đại diện.
- <img width="904" height="706" alt="image" src="https://github.com/user-attachments/assets/dcbdb9e0-de2e-48cc-b81c-3f7eed950bd7" />

- Khi bạn chọn `avatar.jpg` và nhấn Upload, về cơ bản có ba thành phần tham gia:
  - Client (Browser): Trình duyệt trên máy tính của bạn.
  - HTTP Request: Gói tin chứa dữ liệu được gửi đến máy chủ.
  - Server: Máy chủ tiếp nhận và xử lý dữ liệu.
- File không tự nhiên xuất hiện trên Server. Trình duyệt phải gửi dữ liệu của file thông qua một HTTP request.
### 2. Quá trình upload diễn ra như thế nào?
<img width="952" height="756" alt="image" src="https://github.com/user-attachments/assets/fb718dde-2463-469a-b68b-1844e3918ffa" />

### 3. HTTP Request gửi file lên Server như thế nào?
- Khi upload một file, trình duyệt thường sử dụng phương thức `HTTP POST` kết hợp với `multipart/form-data`.
- Ta dùng `multipart/form-data` bởi vì một HTTP Request có thể chứa nhiều thành phần khác nhau, chẳng hạn:
  - File ảnh.
  - Tên người dùng.
  - Mô tả ảnh.
  - Các thông tin bổ sung khác.
- Lấy ví dụ về cấu trúc cơ bản của một HTTP Request:
  ```
  POST /upload HTTP/1.1
  Host: example.com
  Content-Type: multipart/form-data; boundary=----ABC123
  
  ------ABC123
  Content-Disposition: form-data; name="avatar"; filename="avatar.jpg"
  Content-Type: image/jpeg
  [NỘI DUNG NHỊ PHÂN CỦA FILE ẢNH]
  
  ------ABC123--
  ```
- Ta sẽ cùng đi phân tích HTTP Request mẫu này:
    - a. `POST /upload HTTP/1.1`
      - `POST`: Phương thức gửi dữ liệu.
      - `/upload`: Endpoint tiếp nhận yêu cầu upload.
      - `HTTP/1.1`: Phiên bản giao thức HTTP.
    - b. `Content-Type: multipart/form-data; boundary=----ABC123`
      - Đây là Content-Type của toàn bộ HTTP request.
      - Nó thông báo rằng Body chứa nhiều phần dữ liệu được phân tách bằng một chuỗi gọi là `boundary`.
    - c. `Content-Disposition`
      - Gồm có ba thành phần chính là:
        - `form-data`: 	Đây là dữ liệu của một trường trong form
        - `name="avatar"`: 	Tên trường upload
        - `filename="avatar.jpg`: Tên file được upload
    - d. `Content-Type: image/jpeg`
        - Đây là Content-Type của file chứ ko phải của HTTP Request. Nó cho biết file được tải lên có định dạng là JPEG (Một hình ảnh)
    - e. Nội dung file
        - Đây là phần dữ liệu thực tế của ảnh, thường được truyền dưới dạng nhị phân (binary).
### 4. Sever nhận được file thì sẽ làm gì?
- Sau khi nhận HTTP request, Server thường thực hiện các công việc sau :
    - Tiếp nhận dữ liệu: Server đọc HTTP request và lấy ra nội dung file.
    - Kiểm tra file: Kiểm tra những yếu tố như định dạng, kích thước, nội dung và tên file.
    - Lưu trữ: Nếu file tải lên được xác thực, file sẽ được lưu vào nơi phù hợp.
    - Phản hồi: Server trả về HTTP response để thông báo kết quả.
### 5. File được lưu ở đâu?
- Ví dụ: File ảnh sẽ được lưu tại: `/var/www/html/uploads/avatar.jpg`
- Trong đó:
    - /var/www/html: Thư mục website.
    - /uploads: Thư mục chứa các file được upload.
    - avatar.jpg: File được lưu trữ.
- Sau khi upload, website có thể cung cấp một URL để truy cập file: `https://example.com/uploads/avatar.jpg`
- Khi bạn truy cập URL này, trình duyệt gửi một HTTP request khác có dạng:
  ```
  GET /uploads/avatar.jpg HTTP/1.1
  Host: example.com
  ```
- Server tiếp nhận request, tìm file tương ứng và trả nội dung file về cho trình duyệt.
- Tuy nhiên cũng phải phân biệt giữa việc upload file và truy cập file vì đây là hai request khác nhau
- 
   | Hành động | HTTP Method | Mục đích |
   | -------- | -------- | -------- |
   | Upload File         |  POST        |  Gửi file lên sever        |
   | Truy cập file         | GET         |  	Yêu cầu Server trả file về        |
### 6. Server trả file về có phải lúc nào cũng giống nhau?
- Tuỳ vào loại file được tải lên, sever sẽ xác định nó thuộc loại file nào rồi trả về lại kết quả
  - Trường hợp 1: File tĩnh (Static File). Ví dụ: `avatar.jpg`, `style.css`, `logo.png`.
      - Server thường đọc nội dung file rồi gửi trực tiếp về trình duyệt.
      - Server không cần thực thi mã phía server bên trong file ảnh.
  - Trường hợp 2: File có thể thực thi (Executable File)
      - Ví dụ: Một file PHP được cấu hình để chạy trên server.
      - Server chuyển file đến bộ xử lý PHP, thực thi chương trình và gửi kết quả trả về trình duyệt.
      - Trình duyệt nhận kết quả thực thi, không nhất thiết nhận mã nguồn PHP ban đầu.

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

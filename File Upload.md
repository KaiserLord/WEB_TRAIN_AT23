(Tất cả những nội dung trên này đều được lấy từ Portswigger, bạn có thể truy cập vào <https://portswigger.net/web-security/file-upload> để tìm hiểu kĩ hơn)
### 1. Lỗ hỏng file upload là gì?
- Lỗ hỏng File Upload xảy ra khi một web sever cho phép người dùng upload file lên trên file hệ thống của sever mà không xác thực đủ những thứ quan trọng như tên, loại file, nội dung file và kích thước của chúng.
- Việc không thực thi đúng cách các hạn chế đối với những điều này có nghĩa là ngay cả chức năng tải lên hình ảnh cơ bản cũng có thể được sử dụng để tải lên các tệp tùy ý và có khả năng gây nguy hiểm. Điều này thậm chí có thể bao gồm các tệp tập lệnh phía máy chủ cho phép thực thi mã từ xa.
- Trong một số trường hợp, việc tải file lên cũng đủ để gây ra những thiệt hại. Các cuộc tấn công khác có thể liên quan đến HTTP Request tiếp theo cho tệp, thường là để kích hoạt quá trình thực thi tệp của máy chủ.
### 2. Tác động của lỗ hỏng file upload
- Sự ảnh hưởng của lỗ hỏng file upload phụ thuộc vào hai yếu tố chính
  - Những thành phần của flie mà trang web không xác thực kĩ, có thể là kích thước, loại file, nội dung của file và rất nhiều thứ khác
  - Những hạn chế nào được áp dụng đối với tệp sau khi nó được tải lên thành công
- Trong trường hợp xấu nhất, loại file không được xác định kĩ và cấu hình của sever cho phép một số loại file nhất định như `.php` hoặc `.jsp` được thực thi dưới dạng mã. Trong trường hợp này, hacker có khả năng upload một đoạn mã phía máy chủ có chức năng như một web shell, cấp toàn quyền cho chúng kiểm soát toàn bộ phía máy chủ (**web shell** là các đoạn mã độc hại cho phép hacker xâm nhập máy chủ web và khởi động các cuộc tấn công vào máy chủ để đánh cắp, sửa đổi thông tin hoặc phá hoại hệ thống)
- Nếu như tên file không được xác thực một cách cẩn thận, nó có thể cho phép hacker ghi đè lên các file quan trọng chỉ bằng cách upload các file có cùng tên. Nếu máy chủ cũng dễ bị tấn công bởi việc truyền tải thư mục, điều này có nghĩa là hacker thậm chí cũng có thể upload các file ở những vị trí không thể lường trước
- Việc không đảm bảo kích thước của tệp nằm trong ngưỡng dự kiến ​​cũng có thể kích hoạt một dạng tấn công từ chối dịch vụ (DoS), theo đó kẻ tấn công sẽ lấp đầy dung lượng đĩa trống.
### 3. Lỗ hỏng file upload phát sinh như thế nào?
- Với những mối nguy hiểm khá rõ ràng, rất hiếm khi các trang web thực tế không có bất kỳ hạn chế nào về những tệp mà người dùng được phép tải lên. Thông thường, các developer triển khai những cơ chế xác thực mà họ tin là vững chắc nhưng thực chất lại có lỗi tiềm ẩn hoặc rất dễ bị qua mặt.
- Ví dụ, họ có thể cố gắng đưa các loại tệp nguy hiểm vào danh sách đen, nhưng lại không tính đến sự sai lệch khi phân tích cú pháp lúc kiểm tra phần mở rộng tệp. Giống như bất kỳ danh sách đen nào, việc vô tình bỏ sót các loại tệp ít phổ biến hơn nhưng vẫn có thể gây nguy hiểm cũng rất dễ xảy ra.
- 

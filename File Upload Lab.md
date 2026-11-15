## Đây là write up của mình khi làm những bài lab về File Upload trên PortSwigger. Cũng giống như khi làm về Path Traversal, mình cũng sẽ vừa làm, vừa ôn lại kiến thức và giải thích bản chất nhé

### 1. Lab: Remote code execution via web shell upload
- Lab này yêu cầu chúng ta lấy nội dung từ file có đường dẫn là `/home/carlos/secret` và cung cấp tài khoản, mật khẩu để ta đăng nhập vào `wiener:peter`
- Sau khi đăng nhập, ta nhận được giao diện như thế này
  <img width="887" height="685" alt="image" src="https://github.com/user-attachments/assets/0b2b19d3-a581-44b8-8648-762088468d17" />

- Mình sẽ thử up một file ảnh và dùng BurpSuite để bắt Request đó lại nhé
  <img width="1486" height="811" alt="image" src="https://github.com/user-attachments/assets/c52665d3-6868-4728-8c6a-0976a4d95709" />

- Đây là Request mà mình nhận được, nó cũng khá giống Request ví dụ đã được giới thiệu trên PortSwigger.
- Bây giờ mình sẽ thử up một file php có chứa đoạn mã lấy nội dung từ đường dẫn `/home/carlos/secret` xem có được không nhé
  <img width="1491" height="806" alt="image" src="https://github.com/user-attachments/assets/10957bf7-ac2c-495f-ba59-07b5e635879b" />

- Có vẻ như bộ lọc của trang web này đã làm không kĩ, mặc dù trang web yêu cầu mình up hình ảnh để làm ảnh đại diện nhưng mình hoàn toàn có thể up được file php lên và kết quả là mình có được nội dung từ đường dẫn `/home/carlos/secret`
  <img width="1496" height="592" alt="image" src="https://github.com/user-attachments/assets/cb8b7f7c-ec11-4608-a384-aa9fa9332206" />

- Nói chung thì đây là một bài đầu, minh hoạ về lỗ hỏng file upload nên không có quá nhiều điều để nói. Đơn giản là vì web không có bộ lọc hay bất cứ yêu cầu gì về file up lên nên mình hoàn toàn có thể up một file có chứa mã độc để lấy nội dung từ cái đường dẫn được yêu cầu ha
### 2. Lab: Web shell upload via Content-Type restriction bypass
- Cũng giống như bài lab đầu tiên, nhiệm vụ của chúng ta vẫn là đăng nhập vào tài khoản cho trước và lấy nội dung từ `/home/carlos/secret`
- Mình vẫn làm giống lab đầu tiên, up thử một hình ảnh để bắt Request rồi mới up tiếp file php xem có được không nhé
  <img width="1490" height="807" alt="image" src="https://github.com/user-attachments/assets/5f4d404a-2540-4abc-8200-e0ac19e09e09" />

- Không được rồi, có vẻ như trang web đã có bộ lọc gì đó và không cho mình upload file php rồi. Mình sẽ xem lại Request khi mình up hình ảnh đầu tiên xem có gì không nhé
  <img width="745" height="807" alt="Screenshot 2026-11-15 145407" src="https://github.com/user-attachments/assets/c37152c1-79f0-4965-8f24-a571f2d7fed7" />

- Sau khi quan sát thì mình để ý thấy `Content-Type` của Request là `image/jpeg` . Vậy là không thể up file php lên được rồi. Nhưng liệu nếu mình xoá nội dung của ảnh và ghi đoạn code trích xuất nội dung giống lab 1 rồi đổi đuôi `.jpg` thành đuôi `.php` mà không chỉnh `Content-Type` thì có bypass được không nhỉ, hãy cũng mình làm thử nhé.
  <img width="1487" height="807" alt="image" src="https://github.com/user-attachments/assets/9cfdce80-1711-40de-beb6-4896c4d6bd6d" />

- Có vẻ như mình đã đoán đúng rồi, bây giờ mình sẽ xem liệu khi up lên thành công mình có lấy được nội dung bí mật không nhé. Ở lab 1, bạn có thể up trực tiếp lên và đọc được trong HTTP History, nhưng ở đây thì không nên hãy cũng làm theo mình để xem mình có lấy được nội dung ẩn ko nhé. Sau khi bạn đã thành công trong việc up load lên rồi, bạn hãy quay lại HTTP Histrory để xem lại khi mà bạn gửi ảnh đầu tiên lên thì nó được lưu ở đâu nhé
  <img width="1496" height="833" alt="Screenshot 2026-11-15 150403" src="https://github.com/user-attachments/assets/6efdc2e6-8b29-4786-9e44-089887fdcfe0" />

- Sau khi đã có được đường dẫn rồi, bạn hãy nhìn Request mà mình khoanh đỏ ở trên. Đáng lẽ ở lab 1, mình sẽ dùng đến nó, nhưng vì lab 1 cho phép mình up thẳng file php và mình chỉ cần nhấn request đó để lấy nội dung nên mình không cần dùng đến nó. Bạn chọn request đó và gửi nó đến Repeater để chỉnh sửa. Ở dòng đầu tiên, bạn hãy xoá `/academyLabHeader` sau GET đi và thay nó bằng đường dẫn mà ảnh đã lưu khi mà bạn up nó lên. Tuy nhiên, bạn hãy sửa đuôi `.jpg` thành đuôi `.php` thì mới có thể lấy được nội dung bí mật nhé
  <img width="1491" height="651" alt="image" src="https://github.com/user-attachments/assets/b5a679a0-e208-4a43-8cad-62217d593c6c" />

- Trong cách mình làm, mình chọn giữ nguyên `Content-Type` và sửa nội dung ảnh cũng như đuôi ảnh. Tuy nhiên, bạn cũng có thể chọn cách nhanh hơn là đổi `Content-Type` khi mà bạn up file php thành `image/jpeg` thì vẫn được và thực hiện các bước sau giống như mình sẽ nhanh hơn mà không cần phải đổi từ đuôi `.jpg` sang `.php` giống mình nhé. Vì cách đầu tiên là cách mình tự tìm hiểu được nên mình ghi thôi hehe

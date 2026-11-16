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

- Sau khi đã có được đường dẫn rồi, bạn hãy nhìn Request mà mình khoanh đỏ ở trên. Đáng lẽ ở lab 1, mình sẽ dùng đến nó, nhưng vì lab 1 cho phép mình up thẳng file php và mình chỉ cần nhấn request đó để lấy nội dung nên mình không cần dùng đến nó. Đây là Request cho biết website lưu ảnh đại diện tại một đường dẫn có thể truy cập trực tiếp thông qua HTTP. Bạn chọn request đó và gửi nó đến Repeater để chỉnh sửa. Ở dòng đầu tiên, bạn hãy xoá `/academyLabHeader` sau GET đi và thay nó bằng đường dẫn mà ảnh đã lưu khi mà bạn up nó lên. Tuy nhiên, bạn hãy sửa đuôi `.jpg` thành đuôi `.php` thì mới có thể lấy được nội dung bí mật nhé
  <img width="1491" height="651" alt="image" src="https://github.com/user-attachments/assets/b5a679a0-e208-4a43-8cad-62217d593c6c" />

- Trong cách mình làm, mình chọn giữ nguyên `Content-Type` và sửa nội dung ảnh cũng như đuôi ảnh. Tuy nhiên, bạn cũng có thể chọn cách nhanh hơn là đổi `Content-Type` khi mà bạn up file php thành `image/jpeg` thì vẫn được và thực hiện các bước sau giống như mình sẽ nhanh hơn mà không cần phải đổi từ đuôi `.jpg` sang `.php` giống mình nhé. Vì cách đầu tiên là cách mình tự tìm hiểu được nên mình ghi thôi hehe

#### Giải thích 
- Khi bạn gửi một file php, có phải trong response bạn nhận được có nội dung là:
  ```
  Sorry, file type application/octet-stream is not allowed
        Only image/jpeg and image/png are allowed
  Sorry, there was an error uploading your file
  ```
- Điều này có nghĩa là Website chỉ chấp nhận những website chỉ chấp nhận những tệp có MIME Type: `image/jpeg` hoặc `image/png`. Còn nếu bạn thắc mắc MIME Type là gì thì MIME Type (viết tắt của Multipurpose Internet Mail Extensions Type hay còn gọi là Media Type) là một tiêu chuẩn dùng để phân loại và định danh định dạng của dữ liệu được truyền đi trên Internet
  <img width="1070" height="545" alt="image" src="https://github.com/user-attachments/assets/a27ee6ef-a94f-4f5b-af2f-c560a875f544" />
  (Nguồn: Vinahost)
- Quay trở lại vấn đề chính thì file php bạn gửi lên ở đây đang có Content-Type là `application/octet-stream`, máy chủ nhận được thông tin cho biết tệp có MIME Type không được phép nên không cho phép bạn upload lên
- Vậy thì tại sao khi bạn đổi từ `application/octet-stream` sang `image/jpeg` thì lại upload được? Vì Server tin tưởng dữ liệu do người dùng kiểm soát. Bạn không hề biến mã PHP thành một bức ảnh JPEG mà bạn chỉ thay đổi thông tin MIME Type mà trình duyệt gửi lên để đánh lừa cơ chế kiểm tra của website.
- Vậy thì tại sao khi mà bạn up được file php đó lên, bạn lấy được secret đó? Đáng lẽ tôi nên giải thích ở lab 1 nhưng quên mất (hihi) nên tôi sẽ giải thích ở lab 2 này nhé. Bạn có nhớ cái đường dẫn mà tôi yêu cầu bạn tìm kiếm lúc này không, đó là đường dẫn mà ảnh đại diện được lưu ở đó và khi bạn upload file php thì nó cũng được lưu ở đó luôn. Lúc này, nếu máy chủ được cấu hình để thực thi PHP trong thư mục avatar, khi bạn truy cập tệp PHP, quy trình sẽ diễn ra như sau:
  > Bạn gửi GET request -> Web server nhận request và xác định đây là tệp PHP cần xử lý-> PHP interpreter thực thi mã mà bạn đã viết trong file->PHP trả nội dung tệp cho web server->HTTP Response trả về nội dung cho bạn
- Cần nhớ rằng Web Sever đọc tệp bằng quyền của tiến trình PHP đang chạy trên máy chủ do máy chủ được cấu hỉnh để chạy PHP
- Vậy thì tại sao cần phải gửi lại GET Request ? Vì upload thành công và thực thi thành công là hai sự kiện hoàn toàn khác nhau. Việc POST thành công chỉ chứng minh rằng server chấp nhận và lưu tệp. Nó chưa chứng minh rằng server sẽ thực thi tệp đó. Khi gửi GET đến bạn mới thực sự kiểm tra được liệu máy chủ có xử lý tệp PHP như một chương trình hay chỉ trả nó về như một tệp thông thường.
- Nói tóm lại: Lỗ hổng không đơn thuần nằm ở việc website cho phép upload một tệp PHP. Vấn đề sâu hơn là website đã tin tưởng thông tin do client cung cấp, lưu một tệp nguy hiểm vào vị trí có khả năng thực thi mã, rồi cho phép người dùng kích hoạt mã đó thông qua HTTP.

### 3. Lab: Web shell upload via path traversal
- Vẫn giống như 2 bài lab đầu tiên, mình vẫn thực hiện các bước cũ là up ảnh, quan sát request rồi up file php rồi chỉnh sửa xem liệu có lấy được bí mật không nhé
- Lần này mình up được file php mà không gặp bất kì trở ngại nào luôn, mình cứ nghĩ là nó sẽ giống bài một nhưng không. Khi mình chọn Request GET thì đây thì thứ mình nhận được
  <img width="1487" height="588" alt="image" src="https://github.com/user-attachments/assets/d067ea40-8c2a-4c77-8d89-2238e96ed40b" />

- Nó chỉ đơn giản là hiện đoạn code php trong file mà mình đã up lên chứ không có bí mật nào ở đây cả. Mình đọc lại đề bài và nhận ra ở đây đề bài yêu cầu ta kết hợp với một lỗ hỏng là Path Traversal. Vậy nên mình nghĩ là anh dev sau hai bài đầu tiên để mình bypass được thì có thể đã cấu hình thư mục avatars không cho thực thi PHP để hạn chế rủi ro, thành ra là server chỉ trả về nội dung tệp như một tệp thông thường.
- Vậy là thư mục `/files/avatars/` không thể thực thi được code PHP nên tôi cần phải dùng đến Path Traversal để đưa nó lên thư mục `files` và hi vọng thư mục này sẽ được cấu hình để thực thi PHP
- Tôi sẽ thử với các payload Path Traversal mà tôi đã được học trên Portswigger để tìm ra chính xác thư mục đó. Và sau một vài payload thì tôi đã chèn `../` vào tham số filename thành công
  <img width="1487" height="807" alt="image" src="https://github.com/user-attachments/assets/6156c73d-383d-4777-a2af-b2dae9e07d20" />

- Giờ thì chỉnh Request GET để xem tôi có thể lấy được Secret không nhé
  <img width="1485" height="563" alt="image" src="https://github.com/user-attachments/assets/478da68b-6a2a-44bc-9a17-ab5eae3e3738" />

### Giải thích
- Tại sao bài này lại dùng Path Traversal và payload kia lại có thể bypass qua được? Dùng Path Traversal thì tôi đã giải thích ở trên, thư mục `avatars` không được cấu hình để thực thi để thực thi PHP nên tôi muốn đưa nó lên thư mục cha của nó là thư mục `files` để xem thử nó có chạy code PHP hay không và may mắn là ở bài này thì nó chạy được
- Còn tại sao payload kia lại có thể bypass qua được? Ban đầu tôi đã nhập hai payload là `../` và `....//` và sever chỉ trả về là `The file avatars/exploit.php has been uploaded.` và tôi vẫn đang ở thư mục `avatars` mà như thế thì có thể là web đã có bộ lọc loại bỏ `../` vậy nên tôi dùng tới kĩ thuật URL Encoding mà tôi đã được học. Lúc này thì web không nhận dạng được `../` nên nó nghĩ file gửi lên là hợp lệ và cho qua và sau khi URL Encoding thì nó lại hiện về `../` và như thế là bypass qua được bộ lọc. Tất nhiên là nếu web có Double Encoding thì bạn phải Encode `%` thêm một lần nữa để bypass. Trong Response trả về, nó vẫn hiện là `avatars/../hacked.php` vì đây là đường dẫn sau khi được decode chứ không phải sau khi được chuẩn hoá. Và sau đó, bạn sửa đường dẫn trong GET Request thành `/files/avatars/../hacked.php` thì lúc này bạn đã đưa file hacked.php lên thư mục files. Thư mục này chạy file php mà bạn gửi lên và trả lại cho bạn secret
### 4. Lab: Web shell upload via extension blacklist bypass
- 
  

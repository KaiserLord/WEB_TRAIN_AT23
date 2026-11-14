## Đây là Write up của mình khi làm các lab của Portswigger về Path Traversal. Trong này sẽ bao gồm lời giải cũng như kiến thức được học trong từng bài lab 
### 1. Lab: File path traversal, simple case
- Lab này yêu cầu ta truy xuất dữ liệu từ tệp có đường dẫn là `/etc/passwd`. Cách làm bài này cũng khá dễ. Đầu tiên, bạn mở Burpsuite lên, vào Proxy và chuyển sang HTTP History để bắt Request hình ảnh. Đôi khi ở cài đặt mặc định, bạn sẽ không nhìn thấy những request đó. Để nó hiện những request ấy, bạn hãy tắt Filter on ở góc bên phải để hiển thị nhé.
  <img width="1917" height="947" alt="image" src="https://github.com/user-attachments/assets/e4885839-58f7-426e-adb8-afb10fc2db3d" />
  
- Gửi Request đến Repeater bằng phím tắt `Ctrl` + R để chỉnh sửa. Tại đây bạn hãy sửa tham số file này bằng payload `../../../etc/passwd` để bypass
  <img width="1487" height="798" alt="image" src="https://github.com/user-attachments/assets/451bc134-71e4-46d9-bb2e-43730f61cb32" />

#### Ôn lại kiến thức
- Tất nhiên, đây là một bài cơ bản nhưng chúng ta cũng cần phải giải thích rõ tại sao khi nhập chuỗi `../` chúng ta lại giải được bài để ôn lại kiến thức nhé
- Trong hệ thống file, mỗi thư mục có thể chứa các thư mục con. Lấy một ví dụ về một hệ thống file có cấu trúc như đơn giản ntn
 <img width="958" height="271" alt="image" src="https://github.com/user-attachments/assets/d82e9503-21b5-4251-972b-cb16ed8785a8" />

- Trong đó sẽ xuất hiện một chuỗi kí tự đó là `../`, trong đó:
  | Kí hiệu  | Ý nghĩa                         |
  | -------- | --------                        |
  |  `.`     |Thư mục hiện tại                 |
  |   `..`   |Thư mục cha                      |
  | `/`      |Dấu phân cách thư mục trên Linux |

- Giả sử bạn đang ở `/var/www/uploads/` thì khi bạn nhập `/var/www/uploads/../` nó sẽ đưa bạn lên `var/www/`. Nếu bạn tiếp tục thêm một chuỗi `../` vào nữa `/var/www/uploads/../../` thì bạn sẽ ở `var/`. Và nếu bạn thêm một chuỗi nữa thì bạn sẽ ở thư mục gốc.
- Đến đây sẽ có bạn thắc mắc là nếu tôi nhập `../var/www/uploads/` thì có lùi lại một thư mục không? Thì câu trả lời ở đây là không. Khi này thì `..` được xử lý trước tiên. Nó có nghĩa là: ` Từ thư mục hiện tại -> Thư mục cha -> var -> www -> images`
- Quay trở lại vấn đề chính, hệ điều hành không chỉ nhìn đường dẫn như một chuỗi ký tự thông thường. Khi xử lý đường dẫn, nó hiểu những thành phần đặc biệt giống như bảng ở trên . Vì vậy khi ứng dụng yêu cầu hệ điều hành yêu cầu truy cập `/var/www/images/../../../etc/passwd`, Hệ điều hành sẽ phân giải đường dẫn dựa trên cấu trúc thư mục thực tế. Nó không cần tạo ra một thư mục tên `..`, cũng không cần tìm một file có tên chứa `../../../`. Nó hiểu `..` là một chỉ dẫn điều hướng trong hệ thống file.

### 2. Lab: File path traversal, traversal sequences blocked with absolute path bypass
- Ta vẫn thực hiện những bước đầu giống như lab đầu tiên. Tuy nhiên, khi nhập `../../../etc/passwd` ta không thể bypass qua được.
  <img width="1487" height="422" alt="image" src="https://github.com/user-attachments/assets/05ac185f-356e-4d54-9cfe-8610b9a44118" />

- Tôi nghĩ là anh dev đã chặn chuỗi `../`, nếu mà chuỗi này xuất hiện thì không hiện được gì cả. Vậy thì tôi sẽ thử bỏ chuỗi `../` đi và chỉ giữ lại `/etc/passwd` để xem tôi có thể bypass được không
  <img width="1496" height="812" alt="image" src="https://github.com/user-attachments/assets/9a8e8b6e-f79e-49d3-9cda-91241393a8ce" />

- Ở đây anh dev đã quên việc kiểm soát đường dẫn tuyệt đối, vì thế nên tôi dùng `/etc/passwd` để từ thư mục gốc đi thằng vô luôn
### 3. Lab: File path traversal, traversal sequences stripped non-recursively
- Thực hiện các bước là payload giống như bài trước, ta không thể bypass qua được. Tôi thử research thì kiếm được payload này giống với tên đề bài
  <img width="452" height="187" alt="image" src="https://github.com/user-attachments/assets/dff83330-a9ca-4c25-9e12-b0977b340fb8" />


- Nhập thử thì tôi đã giải được bài này.
  <img width="1496" height="808" alt="image" src="https://github.com/user-attachments/assets/de23782f-80ae-4035-b593-20994df02183" />

- Nhưng vấn đề đặt ra ở đây là tại sao khi tôi nhập `....//` ba lần như thế thì tôi lại bypass qua được
- Sau khi tìm hiểu thì tôi biết được đây là lỗi do đoạn code của anh dev chống lại chuỗi `../`. Giả sử anh dev viết một đoạn code lọc chuỗi `../` như thế này: `$filename = str_replace("../", "", $filename);`. Ý nghĩa của đoạn code:
    - str_replace(): tìm chuỗi cần thay thế.
    - Nếu tìm thấy `../`, nó xóa chuỗi đó.
    - Phần còn lại được giữ nguyên.
- Bây giờ ta hãy đánh thứ tự cho chuỗi `....//` nhé. Giả sử vị trí từ 1 đến 4 là kí tự `.` còn 5,6 là `/`. Khi ta nhập chuỗi `....//` vào, bộ lọc phát hiện `../` ở vị trí 3,4,5 và tiến hành xoá nó. Và lúc này, chuỗi 6 kí tự ban đầu còn lại là `../` và thế là ta bypass thành công. Anh dev kiểm tra chuỗi đầu vào trước khi xử lý, nhưng không kiểm tra lại kết quả sau khi xử lý.
### 4. Lab: File path traversal, traversal sequences stripped with superfluous URL-decode
- Vẫn như những bài trước, tôi thử payload của những bài trước và tất nhiên là nó không được rồi. Tôi research tiếp thì kiếm được hai cái payload liên quan tới URL Decode
  <img width="650" height="296" alt="image" src="https://github.com/user-attachments/assets/9b66701a-11b3-4867-bd94-d008086ea5b1" />

- Tôi thử lần lượt hai payload thì...................... không giải được bài này. Tôi research một hồi thì có khả năng là do `..` nó không encode và tôi thử với payload này thì bypass qua được `..%252f..%252f..%252fetc/passwd`
#### Giải thích payload
- Trước tiên thì nói sơ qua về URL Encoding đi ha. URL Encoding (Percent Encoding) là cách biểu diễn các ký tự trong URL bằng dấu % theo sau bởi mã hexadecimal của byte tương ứng.
  | Column 1 | Column 2 |
  | -------- | -------- |
  | `.`      | `%2e`    |
  | `/`      | `%2f`    |
  | `%`      | `%25`    |
  | `a`      | `%61`    |
- Vậy thì câu hỏi đặt ra ở đây là tại sao URL Encoding lại bypass qua được bộ lọc? Giả sử anh dev tạo một bộ lọc đơn giản như ví dụ dưới đi:
  ```
  if (strpos($filename, "../") !== false) {
    die("Access denied");
  } 
  ```
- Đoạn code này thì nó kiểm tra xem tên file có chứa `../` không, nếu có thì từ chối. Khi mà chúng ta gửi một chuỗi đã mã hoá `%2e%2e%2fetc/passwd` thì bộ lọc nó ko thấy `../` nên nó cho phép đi qua. Sau đó thì một thành phần khác cuả ứng dụng giải mã URL thì sẽ được `../etc/passwd`
- Vậy thì double encoding là gì và tại sao phải encode hai lần? Chuỗi `../etc/passwd` khi mã hoá thì thành `%2e%2e%2fetc/passwd` đúng không. Nhưng mà `%` vẫn có thể tiếp tục encode được theo như cái bảng trên thành `%25` và chuỗi lúc này là `%252e%252e%252fetc/passwd`. Vậy thì tại sao lại phải mã hoá hai lần. Vì có những hệ thống có thể có hai tầng xử lý: Một thành phần giải mã URL lần thứ nhất và một thành phần khác giải mã URL lần thứ hai. Như vậy, nếu bộ lọc kiểm tra sau lần giải mã thứ nhất nhưng trước lần thứ hai, nó có thể không nhận ra `../`. Đến khi thành phần tiếp theo giải mã lần hai, chuỗi Path Traversal mới xuất hiện.

- Trở lại với bài lab, tại sao trong payload của bài này thì `..` không cần phải mã hoá? Vì nhiệm vụ của chúng ta là cần mã hoá sao cho vượt qua được bộ lọc của máy chủ, làm cho nó không nhận ra chuỗi `../`. Trong payload của bài này, ta đã mã hoá `/` hai lần thành `%252e` và khi vượt qua được hai bộ lọc giải mã URL thì nó lại trở về `../`
### 5. Lab: File path traversal, validation of start of path
- Đầu tiên ta hãy tiến hành quan sát Request
  <img width="1491" height="592" alt="image" src="https://github.com/user-attachments/assets/e6512fec-f7f7-44a2-8cbc-727cbe3e1a5b" />

- Khác với những lab khác, tham số `filename` ở bài này lại chứa `/var/www/images/` thay vì ẩn đi như các bài trước. Bởi vì anh dev quy định rằng tất cả ảnh phải được truy cập từ thư mục: `/var/www/images/` và tạo ra một cái cơ chế đó là nếu filename bắt đầu băng `/var/www/images/` thì cho phép, còn nếu không bắt đầu bằng chuỗi đó thì ko cho phép. Với những dạng này, ta giữ nguyên tên thư mục và thêm `../../../etc/passwd` để thành `/var/www/images/../../../etc/passwd` để có thể bypass. Còn về cơ chế thì nó giống như mình giải thích ở lab 1 vậy thôi, không có gì mới. Lỗi là nằm ở anh dev, chỉ kiểm tra đường dẫn có bắt đầu bằng /var/www/images không mà quên kiểm tra xem đường dẫn sau khi chuẩn hóa
có thực sự nằm bên trong /var/www/images không?
  <img width="1492" height="806" alt="image" src="https://github.com/user-attachments/assets/9948793c-738d-46cb-b12d-ba5c376ee19d" />

### 6.Lab: File path traversal, validation of file extension with null byte bypass
- Ta tiến hành quan sát request của bài này thì nhận ra trong tham số filename có chứa đuôi `.jpg`. Tôi thử nhập các payload bài trước để test thử thì ko bypass qua được nên tôi thử research và kiếm được payload cho bài này
  <img width="667" height="173" alt="image" src="https://github.com/user-attachments/assets/3ff95b2d-c5a7-485d-9d7b-0572ca13abbb" />

- Nhập thử payload vào thì tôi đã bypass qua được
  <img width="1487" height="803" alt="image" src="https://github.com/user-attachments/assets/c2dbaa91-4b83-4687-b75a-47c2d64576de" />

- Vậy thì tại sao khi nhập payload đó vào, ta lại bypass qua được? Để hiểu tại sao lại bypass được, trước hết ta hãy cùng nhau tìm hiểu Null Byte là gì?
- Null Byte là một byte có giá trị bằng 0, thường được biểu diễn bằng:`\0`. Trong URL Encoding nó được biểu diễn là `%00`. Câu hỏi đặt ra ở đây là tại sao một byte có giá trị 0 lại đặc biệt? Bởi vì trong một số môi trường lập trình, đặc biệt là các API xử lý chuỗi kiểu C, byte null được sử dụng để đánh dấu kết thúc chuỗi.
- Quay trở lại bài lab, khi ta nhập `../../../etc/passwd%00.png`. Bộ lọc thứ nhất yêu cầu tham số filename phải có đuôi là `.png` thì mới cho qua, nó cũng khá giống cơ chế kiểm tra của bài lab thứ 5. Sau khi qua được bộ lọc thứ nhất thì có một thành phần khác giải mã cái chuỗi đó thành `../../../etc/passwd\0.png`. Lúc này thì các API xử lý chuỗi C nó sẽ dừng lại tại Byte null, lúc này chỉ chuỗi chỉ còn lại `../../../etc/passwd`, như vậy là ta có thể bypass thành công.

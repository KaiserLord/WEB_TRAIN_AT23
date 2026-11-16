### 1. Type Juggling là gì?
- Type Juggling là cơ chế PHP tự động chuyển đổi kiểu dữ liệu trong một số ngữ cảnh nhất định để thực hiện phép toán hoặc so sánh. Nói đơn giản:
Bạn đưa cho PHP hai giá trị thuộc hai kiểu dữ liệu khác nhau. Trong một số trường hợp, PHP sẽ tự chuyển đổi một hoặc cả hai giá trị sang kiểu dữ liệu phù hợp để xử lý.
- Ví dụ:
  ```
  <?php

  $a = 123;
  $b = "123";
  var_dump($a == $b);
  ?>
  ```
- Kết quả trả về là : `bool(true)`
- Vậy câu hỏi đặt ra ở đây là tại sao một số nguyên 123 lại có thể bằng một chuỗi "123"? Bởi vì PHP sử dụng phép so sánh lỏng lẻo `==`. Trong trường hợp này, PHP thực hiện chuyển đổi kiểu dữ liệu để so sánh hai giá trị.
### 2. Loose Comparison(`==`) và Strict Comparison (`===`)
##### 2.1 Loose Comparison
- Ví dụ:
  ```
  <?php
  var_dump(123 == "123");
  ?>
  ```
- Kết quả: `bool(true)`
- Điều này xảy ra là vì PHP không yêu cầu hai toán hạng phải cùng kiểu dữ liệu trước khi so sánh. Nó có thể thực hiện Type Juggling để đưa chúng về cách biểu diễn có thể so sánh được.
#### 2.2 Strict Comparison (`===`)
- Ví dụ:
  ```
  <?php
  var_dump(123 === "123");
  ?>
  ```
- Kết quả: `bool(false)`
- Tại sao lại có sự khác nhau giữa Loose Comparision và Strict Comparison? Bởi vì Strict Comparison yêu cầu kiểu dữ liệu phải giống nhau và hai giá trị được so sánh phải bằng nhau
### 3. PHP chuyển đổi kiểu dữ liệu như thế nào?
- PHP có thể thực hiện chuyển đổi kiểu dữ liệu trong những ngữ cảnh khác nhau.
- Ta xét ba trường hợp sau để cùng nhau làm rõ:
#### Trường hợp 1: Số nguyên và chuỗi
- Ví dụ:
  ```
  <?php
  $a = 10;
  $b = "10";
  var_dump($a == $b);
  ?>
  ```
- Như bạn có thể thấy, biến a đang có kiểu dữ liệu là số nguyên, trong khi biến b đang có kiểu dữ liệu là chuỗi. Khi mà so sánh hai biến a và b thì bạn nghĩ rằng vì đây là hai kiểu dữ liệu khác nhau nên kết quả trả về sẽ là false đúng chứ? Nhưng không, ở đây PHP sử dụng `==` để so sánh hai biến a và b. Trong trường hợp này, chuỗi `"10"` được nhận diện là một numeric string (chuỗi số), nên PHP có thể so sánh chúng như hai giá trị số. Và kết quả trả về là True chứ ko phải False như bạn nghĩ
- Điểm mấu chốt ở đây là PHP không yêu cầu hai biến phải cùng kiểu dữ liệu mới được xem là bằng nhau khi bạn dùng `==`
- Và nếu như bạn thay `==` thành `===` thì kết quả trả về sẽ là False vì `===` yêu cầu cả hai biến phải cùng kiểu dữ liệu mới được xem là bằng nhau
#### Trường hợp 2: String và Boolean
- Đến với trường hợp thứ hai, bạn hãy quan sát đoạn code sau đây nhé:
  ```
  <?php
  $a = "hello";
  $b = true;
  var_dump($a == $b);
  ?>
  ```
- Có phải bạn thấy biến a là chuỗi còn biến b là kiểu dữ liệu boolean, nó là hai kiểu dữ liệu khác nhau thì cho dù có so sánh nó bằng `==` hay `===` thì kiểu gì nó cũng phải khác nhau chứ đúng không? Nhưng kết quả trả về lại khiến chúng ta hụt hẵng vì nó sẽ trả về true. WTF? Nó có giống nhau méo đâu? Sao lại trả về False? Đó ắt hẳn là suy nghĩ của bạn lúc này. Nhưng thục tế thì luôn phũ phàng, vì khi so sánh một Boolean với một kiểu dữ liệu khác bằng `==`, PHP chuyển đổi các giá trị sang Boolean để so sánh. Bạn thấy nó khốn nạn không `:)))`. Vậy nên dưới đây là bảng ép kiểu boolean của một số giá trị mà bạn cần nhớ:
  <img width="723" height="586" alt="image" src="https://github.com/user-attachments/assets/93868d55-1d22-4ac7-8aea-d85f960ee3cc" />

=> Nói tóm lại: PHP không nhất thiết so sánh nội dung ban đầu của hai biến. Nó có thể chuyển đổi chúng trước khi thực hiện phép so sánh.
**Lưu ý:** Trong PHP 7, có một quy tắc quan trọng: khi so sánh số với một chuỗi không phải numeric string, chuỗi đó có thể được chuyển thành số 0.
### 4. MAGIC HASHES
#### 4.1. Trước tiên, Hash là gì?
- Giả sử bạn có một chuỗi `"hello"`. Bạn đưa chuỗi này qua thuật toán MD5
  ```
  <?php
  $hash = md5("hello");
  echo $hash;
  ?>
  ```
- Kết quả trả về sau khi băm là: `5d41402abc4b2a76b9719d911017c592`
- Vậy thì MD5 đã làm gì? Nói đơn giản thì nó nhận dữ liệu do người dùng nhập vào và tạo ra một chuỗi Hash có độ dài cố định là 32 ký tự hexadecimal. Và điều quan trọng ở đây, hash nhận giá trị chuỗi chứ không phải là số
#### 4.2 Scientific Notation là gì?
- Trong toán học, Scientific Notation (ký hiệu khoa học) là cách biểu diễn những số rất lớn hoặc rất nhỏ bằng dạng: `ax10^b`.
- Đôi khi lập trình bạn cũng sẽ thấy kiểu này. Ví dụ: 2e4=2x10^4=20000
- Bạn có thấy chữ `e` kia không? Lấy một ví dụ khác đi: 3e2 có phải là 3x10^2 đúng chưa? Vậy nếu ta có 0e12345 thì sao?
- Nó chính là 0x10^12345=0. Còn nếu nó là 0e6789 thì nó chính là 0x10^6789=0. Bạn có thấy sự giống nhau ở đây chưa? Hai chuỗi 0e12345 và 0e6789 có phần ký tự phía sau hoàn toàn khác nhau, nhưng nếu được diễn giải dưới dạng số thì đều có giá trị bằng 0.
#### 4.3. Magic Hashes thực sự hoạt động như thế nào?
- Bây giờ, ta hãy xét đoạn code:
  ```
  <?php
  $a = "0e12345";
  $b = "0e67890";
  var_dump($a == $b);
  ?>
  ```
- Kết quả trả về từ đoạn code trên là: `bool(true)`
- Khi nhìn vào giá trị của hai biến a và b, bạn thấy giá trị của chúng khác nhau hoàn toàn. Nếu bạn dùng `===` để so sánh: `var_dump($a === $b);` thì kết quả trả về sẽ là `bool(false)`
- Nhưng nếu bạn dùng `==` để so sánh thì kết quả nhận về sẽ là `bool(true)`. Tại sao lại như vậy? Vì trong PHP, các chuỗi số có dạng số khoa học có thể được so sánh theo giá trị số. Khi này, PHP sẽ hiểu rằng ` 0e12345 -> 0 và 0e6789 -> 0` rồi sẽ tiến hành so sánh `0==0` và trả về kết quả true. Vậy thì một hiện tượng so sánh chuỗi như vậy có thể ảnh hưởng đến bảo mật website như thế nào?
- Giả sử lập trình viên muốn dữ liệu người dùng nhập vào sau khi hash phải giống với hash được lưu. Nhưng hai hash đều có dạng numeric string đặc biệt, PHP có thể so sánh chúng theo giá trị số thay vì nội dung chuỗi và khi này thì khả năng xác thực sẽ được thông qua mặc dù hai hash khau nhau về nội dung
##### 4.3 Sự nhầm lẫn giữa Magic Hashes với Hash Collision (va chạm hash)
- Hash Collision là gì? Hash Collision xảy ra khi hai đầu vào khác nhau tạo ra cùng một giá trị hash. Lấy ví dụ: A và B là hai giá trị hoàn toàn khác nhau. Nhưng sau khi hash thì Hash(A)=Hash(B). Tức là hai dữ liệu đầu vào khác nhau nhưng kết quả hash giống hệt nhau.
- Magic Hashes lại có cơ chế khác: Hash(A) và Hash(B) là hai mã hash khác nhau. Nhưng khi PHP `==` để so sánh hai chuỗi hash có dạng numeric string đặc biệt thì Hash(A) == Hash (B) và kết quả trả về có thể là `true`
#### 4.4 Nhận diện Magic Hash 
- Bắt đầu bằng 0.
- Tiếp theo là e hoặc E.
- Phần còn lại chỉ gồm các chữ số.
- Đối với hash MD5, chuỗi có 32 ký tự hexadecimal, nên ngoài hai ký tự đầu, 30 ký tự còn lại phải là chữ số.
- Bạn cũng cần nhớ: không phải hash nào bắt đầu bằng 0e cũng là Magic Hash. Ví dụ, nếu phần sau chứa chữ a, nó không còn là một numeric string hợp lệ theo dạng này.
  <img width="1025" height="363" alt="image" src="https://github.com/user-attachments/assets/ef3c8a68-d276-481b-97e1-1bc7a10dfe43" />

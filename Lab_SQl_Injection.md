### 1. Lab: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data
<img width="1917" height="1140" alt="image" src="https://github.com/user-attachments/assets/d71fc201-c080-41c6-9a47-939a3b8d74fe" />

- Khi mở lab lên, ta được một giao diện ntn. Đề bài yêu cầu ta sử dụng SQL Injection để hiển thị toàn bộ sản phẩm bao gồm sản phẩm đã phát hành và chưa được phát hành. Ta có thể tiến hành SQL Injection ở bất kì thư mục nào. Ở đây tôi sẽ lựa chọn thư mục `Lifestyle`
- Sau khi truy cập vào thư mực `Lifestyle`. Tôi nhận được giao diện ntn.
<img width="1916" height="1137" alt="image" src="https://github.com/user-attachments/assets/92008ff0-22b3-4dff-83ba-371d7bc88c77" />

- Dùng BurpSuite thì tôi biết được khi nhấn vào thư mục `Lifestyle` thì Request gửi đi là phương thức `GET` và chứa category = `Lifestyle`. Vì thế nên tôi sẽ thử nhập dấu `'` để xác định xem web này có bị lỗi hay không 
<img width="1482" height="862" alt="image" src="https://github.com/user-attachments/assets/a71ef43e-c6da-4ce7-82be-6c5ecc6a4f1e" />

- Vậy là trang web này có thể bị tấn công bằng SQL Injection. Tôi tiếp tục thêm mệnh đề comment trong SQL là `--` vào sau `Lifestyle` thì nhận được giao diện như thế này:
<img width="1490" height="860" alt="image" src="https://github.com/user-attachments/assets/ac24f99d-025f-49c5-8f5e-68342d1e826d" />

- Câu truy vấn khi tôi thực hiện điều này là:
    ```
    SELECT * FROM products WHERE category = 'Lifestyle'--' AND released = 1
    ```
- Khi này thì mệnh đề sau dấu `--` được xem là một câu ghi chú khi lập trình và bị loại bỏ hoàn toàn khỏi câu truy vấn và lúc này câu truy vấn chỉ còn lại: `SELECT * FROM products WHERE category = 'Lifestyle'`. Khi này thì sản phẩm bị ẩn đã được hiện ở mục `Lifestyle` như hình trên.
- Nhưng thay vì một sản phẩm, tôi muốn khi nhấn vào bất cứ thư mục nào thì toàn bộ sản phẩm ở thư mục đó sẽ được hiện lên bao gồm cả sản phẩm bị ẩn. Tôi sẽ chèn thêm điều kiện `OR 1=1--` vào trong câu truy vấn. Khi này, câu truy vấn của tôi sẽ là:
    ```
    SELECT * FROM products WHERE category = 'Lifestyle' OR 1=1--' AND released = 1
    ```
- Câu truy vấn lúc này sẽ thực hiện lấy toàn bộ sản phẩm ở thư mục `Lifestyle` hoặc điều kiện `1=1`. Vì điều kiện `1=1` luôn đúng, do đó thì toàn bộ sản phẩm của ứng dụng sẽ được hiển thị, kể cả sản phẩm chưa được phát hành.
- Thực hiện điều này trên thanh URL, ta sẽ được như hình dưới
<img width="1916" height="1138" alt="image" src="https://github.com/user-attachments/assets/e49f2cb0-8ebd-42b0-94da-dace5c521095" />

### 2.Lab: SQL injection vulnerability allowing login bypass
- Khi mở chall lên, ta được giao diện như thế này
<img width="1917" height="1096" alt="image" src="https://github.com/user-attachments/assets/bb6d79e7-d4a0-484a-9f7e-e2316c842f9a" />

- Chọn My account để tiến hành đăng nhập. Tại đây, ta được giao diện như thế này
<img width="1917" height="747" alt="image" src="https://github.com/user-attachments/assets/f02bb333-60ee-4c52-849d-7cda225ad95c" />

- Để xác định xem trang web có bị SQL Injection hay không, tôi thử thêm dấu `'` vào sau một tên đăng nhập bất kì và nhập được giao diện như thế này
<img width="1917" height="1133" alt="image" src="https://github.com/user-attachments/assets/b9e7b311-6827-4090-b77b-6ddb5eda0be0" />

- Điều này chứng tỏ trang web bị SQL Injection. Tôi đoán câu truy vấn lúc này là:
`SELECT * FROM users WHERE username = 'test'' AND password = 'abcxyz'`. Vì vậy tôi nghĩ mình có thể chèn thêm comment SQL `--` vào sau tên đăng nhập
- Đọc lại yêu cầu của chall thì đề bài yêu cầu tôi đăng nhập với tư cách là `administrator`. Vì thế tôi nghĩ mình sẽ thử đăng nhập bằng `administrator'--` và nhập một mật khẩu bất kì. Sau khi làm xong thì tôi đã bypass được và đăng nhập với tư cách của `administrator`
<img width="1915" height="1102" alt="image" src="https://github.com/user-attachments/assets/b76e64a9-1d60-4eb6-850d-df61619c0ac0" />

### 3. Lab: SQL injection UNION attack, determining the number of columns returned by the query
- Đề bài yêu ta tấn công bằng UNION attack và xác định số cột được trả về bởi câu truy vấn
- Khi mở chall lên thì ta sẽ được giao điện như thế này
<img width="1917" height="1097" alt="image" src="https://github.com/user-attachments/assets/c078d1b8-439e-4712-bb2c-45c41af8e3ac" />

- Đề bài yêu cầu ta tấn công bằng UNION attack, xác định số cột và trả về một hàng bổ sung chứa giá trị null.
- Theo như kiến thức mà tôi tìm hiểu, có hai phương pháp để xác định số cột trả về. Trước tiên tôi sẽ sử dụng phương pháp dùng `' ORDER BY N--` để xác định xem có bao nhiêu cột
  <img width="1917" height="946" alt="image" src="https://github.com/user-attachments/assets/87cee88c-2be7-4e79-a515-f12a8c001d07" />
  <img width="1917" height="1083" alt="image" src="https://github.com/user-attachments/assets/70a67a8f-a352-4785-829f-d833437d1969" />
  <img width="1917" height="1091" alt="image" src="https://github.com/user-attachments/assets/af13547a-bd45-4a13-9b3d-e673830c42c3" />
  <img width="1917" height="1082" alt="image" src="https://github.com/user-attachments/assets/fae66085-b927-477e-9bb3-56c29f607fbe" />
  
- Như vậy, thư mục `Gifts` của web sẽ có 3 cột. Sau khi biết được số cột, tôi sẽ dùng UNION attack với câu lệnh là ` ' UNION SELECT NULL,NULL,NULL-- ` và đã giải được chall
<img width="1917" height="1036" alt="image" src="https://github.com/user-attachments/assets/aeaa8fc4-6f31-4d37-8733-b12fa10d27d0" />

### 4. Lab: SQL injection UNION attack, finding a column containing text
- Đề bài yêu cầu ta tìm cột có chứa văn bản bằng UNION attack. Trước hết ta cần phải xác định số cột có trong các thư mục. Thư mục mà tôi lựa chọn ở đây là `Lifestyle`. Để làm được điều này, tôi sẽ sử dụng phương pháp thứ nhất đó là dùng `' ORDER BY N--`. Sau khi tiến hành thì tôi biết được thư mục `Lifestyle` có 3 cột.
<img width="1917" height="796" alt="image" src="https://github.com/user-attachments/assets/f3db5e0d-344b-49af-bb11-c4fd57c61ddf" />

- Sau khi đã biết được số cột, tôi sẽ tiến hành dùng kĩ thuật UNION attack thông qua `' UNION SELECT `. Vì ở đây có 3 cột nên tôi sẽ chọn hai giá trị `null` và một giá trị văn bản. Giá trị văn bản ở đây mà để bài yêu cầu là `RptblS`
  <img width="1917" height="205" alt="image" src="https://github.com/user-attachments/assets/c7505307-81b3-45dc-b294-2bd660b1de42" />
  
- Vậy nên tôi sẽ sử dụng `' UNION SELECT 'RptblS',NULL,NULL--` và đảo vị trí của `RptblS` để xác định xem cột nào sẽ chứa chuỗi văn bản này. Và sau khi thử thì tôi đã biết được chuỗi văn bản cần tìm nằm ở cột thứ 2
  <img width="1916" height="1077" alt="image" src="https://github.com/user-attachments/assets/200628f9-3972-4d2e-915e-d336a91944d2" />

### 5.Lab: SQL injection UNION attack, retrieving data from other tables
- Chúng ta hãy cùng đến với một hình thức khác của UNION attack, lần này ta sẽ truy xuất dữ liệu từ những bảng khác. Đây là một kết hợp các kĩ thuật đã được học trước đây
- Ta sẽ chọn thư mục `Lifestyle` để khai thác vì đề bài nói rằng các bộ lọc thư mục rất dễ bị khai thác và bạn cũng có thể chọn các thư mục khác để khai thác nếu muốn. Trước tiên hãy xác định xem trong thư mục này có bao nhiêu cột và bước tiếp theo là xác định kiểu dữ liệu của các cột
- Trước tiên thì hãy thử kiểm tra xem khi ta thêm dấu `'` vào trong tên thư mục ở URL thì liệu nó có dẫn đến lỗi gì không
  <img width="1917" height="722" alt="image" src="https://github.com/user-attachments/assets/c74b7af7-0d31-4468-9f6a-dd7fc12685e0" />
  
- Như vậy có thể xác định rằng trang web có thể bị khai thác bằng SQL INJECTION
- Bây giờ, ta sẽ tiến hành xác định số cột được sử dụng. Ta sẽ sử dụng kĩ thuật `' ORDER BY N--` để làm điều này.
  <img width="1917" height="555" alt="image" src="https://github.com/user-attachments/assets/5cbf24b4-4e56-46bd-a5ae-1f592d249ce2" />
  
- Như vậy số cột được sử dụng là 2 cột. Bước tiếp theo là xác định kiểu dữ liệu của hai cột này. Ta sẽ sử dụng kĩ thuật đã được học ở những bài trước là `'UNION SELECT 'a',NULL,NULL--`. Vì ở đây có hai cột và tôi đoán rằng kiểu dữ liệu của nó sẽ là chuỗi văn bản, do đó tôi sẽ sử dụng `'UNION SELECT 'a',NULL--`
<img width="1917" height="1035" alt="image" src="https://github.com/user-attachments/assets/858cd153-aa65-49b6-bb06-66cec722b063" />

- Vậy là ta đã biết được cột đầu tiên là chuỗi , giờ thì xác định xem liệu cột thứ hai có dùng chung kiểu dữ liệu này hay không
<img width="1321" height="1017" alt="image" src="https://github.com/user-attachments/assets/8394d876-4777-44b1-8fee-6719e0cfc8f6" />

- Như vậy, hai cột đều sử dụng kiểu dữ liệu là chuỗi. Điều này có nghĩa là nếu tôi muốn truy xuất dữ liệu từ các bảng khác, tôi có thể sử dụng hai cột này. Và mục tiêu của bài này là truy xuất tên đăng nhập và mật khẩu của administrator từ bảng users. Vậy nên tôi sẽ tiếp tục dùng UNION Attack để làm điều này. Tôi sẽ dùng câu truy vấn: `'UNION SELECT username,password FROM users--`
<img width="1487" height="676" alt="image" src="https://github.com/user-attachments/assets/5990db20-af12-45f2-96d9-ddfe0e10c305" />

- Sau khi làm xong, tôi nhận được tên đăng nhập và mật khẩu của administrator. Giờ thì lấy hai dữ kiện này và đăng nhập vào trang web thôi

  
  











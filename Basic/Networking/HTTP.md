# HTTP (Hypertext Transfer Protocol)

- HTTP là một giao thức để nạp các tài nguyên như HTML, là một giao thức tiêu chuẩn về mạng, truyền tải thông tin giữa client-server. Nó là một ứng dụng của bộ giao thức TCP/IP
- HTTP là một giao thức không lưu trạng thái (stateless protocol), tức là request hiện tại không biết gì về những request trước đó. Vì thế nếu chúng ta muốn chúng liên kết với nhau thì HTTP cookie, nó cho phép chia sẽ ngữ cảnh hoặc trạng thái.

- Ví dụ như chúng ta muốn xem nội dung của một trang web, thì chỉ cần gõ đường dẫn lên trình duyệt rồi mở nó, khi đó 1 giao thức HTTP sẽ được thực hiện để lấy được nội dung từ server, cơ chế này được gọi là client & server hay request & response.

Như chúng ta thấy ở đây thì việc này có vẻ đơn giản nhưng thực tế chúng ta đi sâu vào sẽ thấy nó là cả một quá trình đi qua nhiều network layer, ví dụ như qua 4 layer

- HTTP chỉ hỗ trợ 1 giao thức **GET**

#### HTTP flow

1. Mở một kết nối TCP
2. Gửi một HTTP message
3. Đọc phản hồi đã được gửi bởi server
4. Đóng hoặc

Nhưng chúng ta đã biết khi khởi tạo 1 kết nối TCP thì ở đây nó sẽ xảy qua quy trình kết nối 3 bước (3 way handshake process), việc phải thực hiện công việc kết nối 3 bước là một điểm yếu lớn của HTTP 0.9 và 1.0. Vì thế có sự ra đời của HTTP 1.1

#### HTTP/1 Protocol

- Với HTTP/1 hỗ trợ nhiều method hơn như (GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS), có header và TCP connection có thể được giữ lại (keep-alive) để phục vụ các connect tiếp theo

- HTTP Pipelining: một connection có thể cho nhiều request, tức là phía client có thể gọi liên tiếp các requests mà không cần đợi response. Tuy nhiên điều này gây ra một việc khá khó khăn là nếu trường hợp các response trả về không theo thứ tự như lúc gửi đi, thì nó sẽ không phân biệt được response nào của request nào, và request có thể có gói tin bị thất lạc, khiến toàn bộ request khác bị chặn cho tới khi request đo được giải quyết xong, vấn đề này là **TCP Head-of-line blocking (HOL)**, vì thế nó không được cấu hình mặc định và phải phụ thuộc vào server và client cấu hình.

- Vậy HTTP keep-alive là gì? Thay vì gửi các request vào một connection rồi đợi request trả về có thể không theo thứ tự thì chúng ta có thể sử dụng **keep-alive** để gửi từng cặp request/response, Nó cho phép chúng ta thiết lập thời gian chờ.

```
keep-alive được cấm trên HTTP/2 và HTTP/3
```

- Hỗ trợ compression/Decompression (Encoding): Đây là cở chế giúp tối ưu băng thông khi truyền thông tin, cơ chế nén phổ biến nhất là `gzip` và `deflate`

```plane
GET / HTTP/1.1
Host: www.example.com
Accept-Encoding: gzip, deflate
```

#### HTTP/2

- Được dựa trên công nghệ lõi từ `SPDY`
  Vậy SPDY là gì? Nó là một giao thức nhằm mục đích tăng tốc độ tải trang và tính bảo mật cho website.
  Nó ra đời nhằm giải quyết một số vấn đề của HTTP/1

  - Single request per connection: HTTP chỉ có thể tải về từng tài nguyên đối với một request, mỗi cái là một connection tương ứng, việc này làm tăng thời gian chờ cũng như không tận dụng được băng thông, đối với SPDY có cơ chế `Multiplexing` cho phép nhiều truy vấn trên cùng 1 kết nối.
  - Client initated requests: HTTP hoạt động theo cơ chế hỏi đáp tức là khi client request dữ liệu nào thì server sẽ trả về dữ liệu đó, nhưng đôi với với SPDY thì server có thể chủ động đẩy dữ liệu về client mà không cần client gửi request đến
  - Redundant headers: SPDY sử dụng compression header để giảm băng thông
  - Uncompressed data: Nén data là bắt buộc trong SPDY

Stream: HTTP/2 cho phép client cung cấp mức độ ưu tiên cho các luồng dữ liệu (stream)

-

# Load balancer

Load balancer là một thành phần quan trọng trong hệ thống mạng, nó giúp phân phối lưu lượng truy cập đến các server khác nhau, đảm bảo tính sẵn sàng và hiệu quả của hệ thống.

## Các loại load balancer

### 1. Round Robin (Static load balancing)

Round Robin là một phương pháp phân phối lưu lượng đơn giản nhất, nó phân phối lưu lượng đến các server theo thứ tự vòng tròn.

Ví dụ:

```
Server 1: 192.168.1.1
Server 2: 192.168.1.2
Server 3: 192.168.1.3
```

Khi sử dụng phương pháp Round Robin, các yêu cầu sẽ được phân phối như sau:

1. Yêu cầu 1 -> Server 1
2. Yêu cầu 2 -> Server 2
3. Yêu cầu 3 -> Server 3
4. Yêu cầu 4 -> Server 1 (quay lại từ đầu)
5. Yêu cầu 5 -> Server 2
6. Yêu cầu 6 -> Server 3

Và cứ tiếp tục như vậy. Phương pháp này đảm bảo mỗi server đều nhận được số lượng yêu cầu bằng nhau, nhưng không tính đến khả năng xử lý hoặc tải hiện tại của từng server.

### 2. Least Connections (Dynamic load balancing)

Least Connections là một phương pháp phân phối lưu lượng dựa trên số lượng kết nối đến server, nó phân phối lưu lượng đến server có ít kết nối nhất.

Ví dụ:

```
Server 1: 192.168.1.1
Server 2: 192.168.1.2
Server 3: 192.168.1.3
```

Khi sử dụng phương pháp Least Connections, các yêu cầu sẽ được phân phối như sau:

1. Yêu cầu 1 -> Server 1 (1 connection)
2. Yêu cầu 2 -> Server 2 (1 connection)
3. Yêu cầu 3 -> Server 3 (1 connection)
4. Yêu cầu 4 -> Server 1 (2 connections)
5. Yêu cầu 5 -> Server 2 (2 connections)
6. Yêu cầu 6 -> Server 3 (2 connections)

Và cứ tiếp tục như vậy. Phương pháp này đảm bảo mỗi server đều nhận được số lượng yêu cầu bằng nhau, nhưng không tính đến khả năng xử lý hoặc tải hiện tại của từng server.

### 3. Least Response Time

Least Response Time là một phương pháp phân phối lưu lượng dựa trên thời gian phản hồi của server và số lượng kết nối hiện tại. Nó thường ưu tiên server có thời gian phản hồi nhỏ nhất, nhưng cũng xem xét số lượng kết nối hiện tại để tránh quá tải.

Ví dụ:

```
Server 1: 192.168.1.1 (thời gian phản hồi: 10ms, 5 kết nối)
Server 2: 192.168.1.2 (thời gian phản hồi: 15ms, 3 kết nối)
Server 3: 192.168.1.3 (thời gian phản hồi: 5ms, 8 kết nối)
```

Giả sử load balancer sử dụng một thuật toán đơn giản: chọn server có tổng (thời gian phản hồi + số kết nối) nhỏ nhất. Khi đó:

1. Server 1: 10 + 5 = 15
2. Server 2: 15 + 3 = 18
3. Server 3: 5 + 8 = 13

Trong trường hợp này, yêu cầu tiếp theo sẽ được gửi đến Server 3 vì nó có tổng nhỏ nhất (13).

Sau khi xử lý yêu cầu, giả sử tình trạng các server thay đổi như sau:
```
Server 1: 192.168.1.1 (thời gian phản hồi: 9ms, 5 kết nối)
Server 2: 192.168.1.2 (thời gian phản hồi: 14ms, 3 kết nối)
Server 3: 192.168.1.3 (thời gian phản hồi: 7ms, 9 kết nối)
```

Khi đó:

1. Server 1: 9 + 5 = 14
2. Server 2: 14 + 3 = 17
3. Server 3: 7 + 9 = 16

Trong trường hợp này, yêu cầu tiếp theo sẽ được gửi đến Server 1 vì nó có tổng nhỏ nhất (14).

### 4. Resource-based method

Resource-based method là một phương pháp phân phối lưu lượng dựa trên tài nguyên của server, nó phân phối lưu lượng đến server có nhiều tài nguyên hơn.

Ví dụ:

```
Server 1: 192.168.1.1 (CPU: 80%, RAM: 60%)
Server 2: 192.168.1.2 (CPU: 90%, RAM: 70%)
Server 3: 192.168.1.3 (CPU: 70%, RAM: 50%)
```

Giả sử load balancer sử dụng một thuật toán đơn giản: chọn server có tổng (CPU + RAM) lớn nhất. Khi đó:

1. Server 1: 80% + 60% = 140
2. Server 2: 90% + 70% = 160
3. Server 3: 70% + 80% = 150

Trong trường hợp này, yêu cầu tiếp theo sẽ được gửi đến Server 1 vì nó có tổng lớn nhất (140).

Sau khi xử lý yêu cầu, giả sử tình trạng các server thay đổi như sau:

```
Server 1: 192.168.1.1 (CPU: 85%, RAM: 65%)
Server 2: 192.168.1.2 (CPU: 95%, RAM: 75%)
Server 3: 192.168.1.3 (CPU: 75%, RAM: 55%)
```

Khi đó:

1. Server 1: 85% + 65% = 150
2. Server 2: 95% + 75% = 170
3. Server 3: 75% + 55% = 130

Trong trường hợp này, yêu cầu tiếp theo sẽ được gửi đến Server 3 vì nó có tổng lớn nhất (130).

### 5. Weighted round-robin method

Weighted round-robin method là một phương pháp phân phối lưu lượng dựa trên trọng số của server, nó phân phối lưu lượng đến server có trọng số lớn hơn.

Ví dụ:

```
Server 1: 192.168.1.1 (trọng số: 1)
Server 2: 192.168.1.2 (trọng số: 2)
Server 3: 192.168.1.3 (trọng số: 3)
```

Giả sử load balancer sử dụng một thuật toán đơn giản: chọn server có trọng số lớn nhất. Khi đó:

1. Server 1: 1
2. Server 2: 2
3. Server 3: 3

Trong trường hợp này, yêu cầu tiếp theo sẽ được gửi đến Server 3 vì nó có trọng số lớn nhất (3).

### 6. IP hash method

Trong phương pháp IP hash, load balancer thực hiện một phép tính toán học, gọi là hashing, trên địa chỉ IP của client. Nó chuyển đổi địa chỉ IP của client thành một số, sau đó ánh xạ số này tới các server cụ thể.

Ưu điểm chính của phương pháp này là nó đảm bảo rằng các yêu cầu từ cùng một địa chỉ IP sẽ luôn được chuyển đến cùng một server (miễn là cấu hình server không thay đổi). Điều này rất hữu ích cho các ứng dụng yêu cầu duy trì trạng thái phiên (session) giữa client và server.

Ví dụ:

Giả sử chúng ta có 3 server backend:

```

Server 1: 192.168.1.1
Server 2: 192.168.1.2
Server 3: 192.168.1.3
```

Load balancer sử dụng một hàm hash đơn giản: chuyển đổi địa chỉ IP thành số nguyên và chia cho số lượng server (3 trong trường hợp này), lấy phần dư.

Khi có các yêu cầu từ các địa chỉ IP client khác nhau:

1. Client IP: 192.0.2.1
   Hash: (192 * 256^3 + 0 * 256^2 + 2 * 256 + 1) % 3 = 1
   Yêu cầu được chuyển đến Server 2

2. Client IP: 192.0.2.2
   Hash: (192 * 256^3 + 0 * 256^2 + 2 * 256 + 2) % 3 = 2
   Yêu cầu được chuyển đến Server 3

3. Client IP: 192.0.2.3
   Hash: (192 * 256^3 + 0 * 256^2 + 2 * 256 + 3) % 3 = 0
   Yêu cầu được chuyển đến Server 1

Như vậy, phương pháp IP hash đảm bảo rằng các yêu cầu từ cùng một địa chỉ IP sẽ luôn được chuyển đến cùng một server, giúp duy trì trạng thái phiên và đảm bảo tính nhất quán trong các yêu cầu liên tiếp từ cùng một client.

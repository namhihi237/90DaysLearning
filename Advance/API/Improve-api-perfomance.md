# Cải thiện hiệu suất API

## 1. Pagination
- Thay vì trả về toàn bộ kết quả trong DB thì chúng ta sẽ trả về từng trang kết quả
- Có 2 cách để phân trang, thứ nhất sử dụng offset và limt, hoặc tối ưu hơn trong 1 số trường hợp chúng ta có thể dùng con trỏ (pointer).

## 2. Caching
- Lưu trữ data thường xuyên được sử dụng trong cache thay vì redis, query database khi data đó không có trong cache.

## 3. Payload Compression
- Giảm data size trong quá trình gửi và nhận request làm tăng tốc độ và giảm băng thông.
- Dữ liệu trả về từ API có thể được nén bằng các thuật toán như Gzip.


## 4. Connection Pool
- Tối ưu hoá việc quản lý kêt nối tới DB hoặc các dịch vụ khác để giảm độ trễ và tải cho server. Thay vì tạo mới và đóng kết nối với DB khi có mỗi yêu cầu API, một nhóm (pool) các kết nối sẽ được tạo sẵn và tái tạo sử dụng cho  nhiều request. Điều này sẽ giúp tiết kiệm tài nguyên, đặc biệt là xử lý nhiều request đồng thời.


## 5. sync Logging
- Múc đích làm tăng tốc độ xử lý API bằng cách không làm chậm quá trình chính khi ghi log.
- Thay vì ghi log đồng bộ (blocking), API có thể sử dụng (non-blocking), tức là việc ghi log sẽ được thực hiện ở một thread khác.

## 6. Rate Limit
- Bảo về API khỏi bị lạm dụng hoặc quá tải bởi các yêu cầu từ 1 hoặc nhiều nguồn.
- Điều này có thể thực hiện bằng API Geteway hoặc một số middleware

## 7. Throttling
- Kiểm soát lưu lượng yêu cầu API để tránh quá tải cho server
- Chẳng hạn như giới hạn tốc độ xử lý yêu cầu dựa trên các quy tắc

## 8. Load Balancing
- Phân phối đều tải giữa các server để tối ưu hoá tài nguyên

## Asynchronous  Processing
- Giảm thời gian phản hồi API bằng cách xử lý các tác vụ tốn thời gian một cách bất đồng bộ.
- Sử dụng các hệ thống hàng đợi tin nhắn như RabbitMQ, Kafka, Redis để xử lý các tác vụ nặng như xử lý ảnh, video, email, notification mà không làm chậm API.

## Content Delivery Network
- Cải thiện tốc độ tải nội dung bằng cách phân phối từ các server ở gần người dùng nhất.
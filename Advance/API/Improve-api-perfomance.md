# Cải thiện hiệu suất API

## 1. Pagination
- Thay vì trả về toàn bộ kết quả trong DB thì chúng ta sẽ trả về từng trang kết quả
- Có 2 cách để phân trang, thứ nhất sử dụng offset và limt, hoặc tối ưu hơn trong 1 số trường hợp chúng ta có thể dùng con trỏ (pointer).

## 2. Caching
- Lưu trữ data thường xuyên được sử dụng trong cache thay vì redis, query database khi data đó không có trong cache.

## Payload Compression
- Giảm data size trong quá trình gửi và nhận request làm tăng tốc độ và giảm băng thông.
- Dữ liệu trả về từ API có thể được nén bằng các thuật toán như Gzip.


## Connection Pool
- Tối ưu hoá việc quản lý kêt nối tới DB hoặc các dịch vụ khác để giảm độ trễ và tải cho server. Thay vì tạo mới và đóng kết nối với DB khi có mỗi yêu cầu API, một nhóm (pool) các kết nối sẽ được tạo sẵn và tái tạo sử dụng cho  nhiều request. Điều này sẽ giúp tiết kiệm tài nguyên, đặc biệt là xử lý nhiều request đồng thời.


## Async Logging
- Múc đích làm tăng tốc độ xử lý API bằng cách không làm chậm quá trình chính khi ghi log.
- Thay vì ghi log đồng bộ (blocking), API có thể sử dụng (non-blocking), tức là việc ghi log sẽ được thực hiện ở một thread khác.
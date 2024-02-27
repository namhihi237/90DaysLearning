# JWT (Json web token)

- JWT là một tiêu chuẩn (standard) xác thực người dùng không trạng thái thường được sử dụng để trao đổi thông tin một cách an toàn giữa client và server ở định dạng JSON

### Cấu trúc của JWT

Gồm 3 phần: header, payload, signature. Mỗi phần được mã hoá dưới dạng base64

- Header Chứa thông tin về thuật toán mã hóa và loại token.

```
{
  "typ": "JWT"
  "alg": "HS256"
}
```

- Payload: Chứa dữ liệu thực tế được truyền tải, ví dụ như `username, role, exp`, v.v.

```
{
  "username": "poppy"
  "role": "admin",
  "exp": 1644768000
}
```

- Signature: Chữ ký số được tạo ra bằng cách sử dụng header, payload và secret

```
signature = HMAC-SHA256(base64urlEncode(header) + "." + base64urlEncode(payload), secret_salt )

```

### Cách thức hoạt động:

1. Client sẽ gửi thông tin xác thực để đăng nhâp
2. Server tạo ra JWT bằng cách mã hoá dữ liệu(payload) và secret
3. Client gửi JWT đến server trong header của request
4. Server giải mã JWT bằng cách sử dụng secret và xác thực dữ liệu
5. Server cho phép hoặc từ chối truy cập dựa trên thông tin xác thực trong JWT

<img src="./assets/token-based-authentication.jpg" alt="MarineGEO circle logo" style="height: 500px;"/>

### Deep dive

#### JWT hỗ trợ nhiều thuật toán mã hoá khác nhau nhưng phổ biến:

- HMAC: sử dụng khoá bí mật chung để mã hoá và xác thực token (Chúng ta sẽ có một bài về các thuật toán mã hoá sau này)
- RSA: sử dụng cặp khoá công khai và khoá bí mật để mã hoá và xác thực token

#### Claims

Payload của JWT có thể chứa nhiều thông tin nhưng một số phổ biến:

- iss (issuer): Bên cấp token
- exp: thời hạn của token
- iat: thời gian token được cấp
- nbf(not before): thời gian token bắt đầu có hiệu lực
- sub: đối tượng mà token được cấp
- aud(audience): bên dự định nhận token

#### Các vấn đề bảo mật

- Khoá bí mật cần được bảo vệ cẩn thận
- Replay attacks: Kẻ tấn công sẽ lấy token hợp lệ và sử dụng lại để truy cập trái phép. Về phần này chúng ta sẽ có một bài khác để đi sâu hơn về kiểu tấn công Replay attacks.
- Cross-site request forgery (CSRF): Kẻ tấn công có thể lừa người dùng thực hiện hành động trái pháp bằng cách sử dụng token của họ ví dụ như lừa người dùng click vào một liên kết hoặc tải nội dung độc hại.

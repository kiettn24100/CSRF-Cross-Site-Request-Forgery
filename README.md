# CSRF-Cross-Site-Request-Forgery( Giả mạo yêu cầu chéo trang ) 

Là một lỗ hổng bảo mật cho phép kẻ tấn công lừa người dùng thực hiện một hành động trên trang web mà hành động đó không phải chủ đích của họ 

Nói dễ hiểu thì hacker mượn session của bạn và lén lút ra lệnh cho trang web thực hiện một điều gì đó ( đổi mật khẩu , chuyển tiền ,...) mà Server vẫn tưởng đó là yêu cầu từ bạn lên trang web

Ví dụ dễ hiểu như , giả sử trang web changeEmail.com có chức năng đổi email . Khi mà user đổi email thì sẽ có 1 request lên server như thế này 

```
POST /email/change HTTP/1.1
Host: changeEmail.com
Cookie: session=abc123456...
Content-Type: application/x-www-form-urlencođe

email=nạn_nhân@gmail.com
```

Khi đó kẻ tấn công sẽ tạo 1 trang web độc hại riêng chứa đoạn mã HTML ẩn tự động gửi request đến **changeEmail.com** với nội dung đổi email thành **hacker123@gmail.com**

đoạn mã trông như sau: 
```html
<form action="https://changeEmail.com/email/change" method="POST">
    <input type="hidden" name="email" value="hacker123@gmail.com" />
</form>
<script>
    document.forms[0].submit(); // Tự động gửi form ngay khi trang tải xong
</script>
```
kẻ tấn công lừa bạn click vào đường link độc hại đó , trình duyệt của bạn sẽ tự động gửi request đến **changeEmail.com** . Quan trọng nhất là trình duyệt tự động đính kèm cookie của bạn vào request gửi lên **changeEmail.com** đó và máy chủ ở **changeEmail.com** thấy cookie hợp lệ và tin rằng bạn đang yêu cầu đổi email và thực hiện lệnh -> Lúc này tài khoản bị chiếm đoạt

Cơ chế hoạt động: nó sẽ cần 3 thứ 

- Các hành động trong ứng dụng mà kẻ tấn công muốn thực hiện ( ví dụ đổi password , đổi email , cấp quyền admin , ... )
- Quản lí phiên bằng Cookie : ứng dụng chỉ dựa vào phiên ( session ) để xác nhận danh tính người dùng . Trình duyệt có cơ chế tự động gửi cookie này kèm theo mọi yêu cầu đến trang web đó
- Cuối cùng cần phải biết cái tham số tương ứng với mỗi hành động cần thực hiện ( như lúc nãy , cái tham số cần truyền vào để có thể đổi email đó chính là `email` ) 

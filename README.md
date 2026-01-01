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

***KHAI THÁC***
Thứ nhất là vượt qua mã CSRF 

Nói dễ hiểu , mã thông báo CSRF là 1 loại mã ngẫu nhiên tạo ra bởi máy chủ và đău cho máy khách , để thực hiện một hành động nhạy cảm như đổi mật khẩu thì phía Client phải gửi kèm theo mã CSRF đó đến máy chủ 

Hãy phân biệt rõ 2 loại hành động , giả sử chúng ta đang ở `https://facebook.com`
Hành động XEM ( GET request ) :
- ví dụ : đọc , lướt , xem post , xem profile , tìm kiếm thông tin 
- không cần CSRF Token bởi vì bạn chỉ nhìn thôi chứ ko thay đổi gì trong database của server cả

Hành động LÀM ( POST request ) : 
- ví dụ : Đổi mật khẩu , Comment , Like , đăng bài viết , ....
- tất cả những hành động trên đều làm thay đổi dữ liệu nên đều cần CSRF Token 
ví dụ như trên: khi mình click vào nút Like một post của trên Fb thì có 1 request gửi lên Server bao gồm cả mã CSRF
ở đây mã CSRF là cái dòng fb_dtsg kia
<img width="975" height="315" alt="image" src="https://github.com/user-attachments/assets/73fd61d9-6aab-4bca-b886-43a1026f3c89" />



Ví dụ từ portswigger:
```html
<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="example@normal-website.com">
    <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button class='button' type='submit'> Update email </button>
</form>
```
đoạn code trên mô tả cách SERVER gửi mã CSRF cho CLIENT của người dùng 
- Trong thẻ <form> để đổi email, ngoài trường nhập email bình thường thì hệ thống chèn thêm một dòng 
`<input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">`
`type="hidden"`: người dùng sẽ không nhìn thấy dòng này trên giao diện web, nó chạy ngầm  
`value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">`: đây mã CSRF ngẫu nhiên , mã này chỉ có tác dụng đối với phiên hoạt động hiện tại. Chỉ máy chủ và trình duyệt của bạn mới biết được đoạn mã random này 

và giả sử khi mà người dùng nhấn vào nút UPDATE EMAIL trình duyệt sẽ gửi một request POST lên SERVER 
(cre: portswigger) 
```
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
```
như bạn đã thấy đó dữ liệu được gửi đi bao gồm cả email mới cần thay và mã CSRF 



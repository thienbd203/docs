# CSRF Protection

- [Giới thiệu](#csrf-introduction)
- [Ngăn chặn CSRF Requests](#preventing-csrf-requests)
    - [Origin Verification](#origin-verification)
    - [Loại trừ URIs](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## Giới thiệu

Cross-site request forgeries là một loại exploit độc hại mà các lệnh trái phép được thực hiện thay cho một người dùng đã xác thực. May mắn thay, Laravel giúp bạn dễ dàng bảo vệ ứng dụng của bạn khỏi các cuộc tấn công [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF).

<a name="csrf-explanation"></a>
#### Giải thích về Lỗ hổng

Trong trường hợp bạn không quen thuộc với cross-site request forgeries, hãy thảo luận về một ví dụ về cách lỗ hổng này có thể được khai thác. Hãy tưởng tượng ứng dụng của bạn có một route `/user/email` chấp nhận một request `POST` để thay đổi địa chỉ email của người dùng đã xác thực. Rất có thể, route này mong đợi một trường input `email` chứa địa chỉ email mà người dùng muốn bắt đầu sử dụng.

Nếu không có CSRF protection, một website độc hại có thể tạo một HTML form trỏ đến route `/user/email` của ứng dụng của bạn và gửi địa chỉ email của người dùng độc hại:

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

Nếu website độc hại tự động gửi form khi trang được tải, người dùng độc hại chỉ cần dụ dỗ một người dùng không nghi ngờ của ứng dụng của bạn truy cập website của họ và địa chỉ email của họ sẽ được thay đổi trong ứng dụng của bạn.

Để ngăn chặn lỗ hổng này, chúng ta cần kiểm tra mọi request `POST`, `PUT`, `PATCH`, hoặc `DELETE` đến để tìm một giá trị session bí mật mà ứng dụng độc hại không thể truy cập.

<a name="preventing-csrf-requests"></a>
## Ngăn chặn CSRF Requests

[Middleware](/docs/{{version}}/middleware) `Illuminate\Foundation\Http\Middleware\PreventRequestForgery`, được bao gồm trong nhóm middleware `web` theo mặc định, bảo vệ ứng dụng của bạn khỏi cross-site request forgeries bằng cách sử dụng cách tiếp cận hai lớp.

Trước hết, middleware kiểm tra header `Sec-Fetch-Site` của trình duyệt. Các trình duyệt hiện đại tự động đặt header này trên mỗi request, chỉ định xem nó có nguồn gốc từ cùng origin, cùng site, hay một nguồn cross-site. Nếu header chỉ định request đến từ cùng origin, request được cho phép ngay lập tức mà không cần bất kỳ xác minh token nào.

Nếu xác minh origin không vượt qua — ví dụ, vì request đến từ một trình duyệt cũ hơn không gửi header `Sec-Fetch-Site` hoặc vì kết nối không an toàn — middleware sẽ quay lại xác thực CSRF token truyền thống.

Laravel tự động tạo một CSRF "token" cho mỗi [user session](/docs/{{version}}/session) hoạt động được quản lý bởi ứng dụng. Token này được sử dụng để xác minh rằng người dùng đã xác thực là người thực sự thực hiện các request đến ứng dụng. Vì token này được lưu trữ trong session của người dùng và thay đổi mỗi khi session được regenerate, một ứng dụng độc hại không thể truy cập nó.

CSRF token của session hiện tại có thể được truy cập thông qua session của request hoặc thông qua hàm helper `csrf_token`:

```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

Bất cứ khi nào bạn định nghĩa một HTML form "POST", "PUT", "PATCH", hoặc "DELETE" trong ứng dụng của bạn, bạn nên bao gồm một trường CSRF `_token` ẩn trong form để middleware CSRF protection có thể xác thực request. Để thuận tiện, bạn có thể sử dụng directive Blade `@csrf` để tạo trường input token ẩn:

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

<a name="csrf-tokens-and-spas"></a>
#### CSRF Tokens & SPAs

Nếu bạn đang xây dựng một SPA sử dụng Laravel như một API backend, bạn nên tham khảo [tài liệu Laravel Sanctum](/docs/{{version}}/sanctum) để biết thông tin về việc xác thực với API của bạn và bảo vệ chống lại các lỗ hổng CSRF.

<a name="origin-verification"></a>
### Origin Verification

Như đã thảo luận ở trên, middleware request forgery của Laravel trước hết kiểm tra header `Sec-Fetch-Site` để xác định xem request có từ cùng origin hay không. Theo mặc định, nếu kiểm tra này không vượt qua, middleware sẽ quay lại xác thực CSRF token.

Tuy nhiên, nếu bạn muốn chỉ dựa vào xác minh origin và vô hiệu hóa hoàn toàn fallback CSRF token, bạn có thể làm điều đó bằng cách sử dụng phương thức `preventRequestForgery` trong file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(originOnly: true);
})
```

Khi sử dụng chế độ chỉ origin, các request không vượt qua xác minh origin sẽ nhận được một HTTP response `403` thay vì response `419` thường liên quan đến sự không khớp CSRF token.

> [!WARNING]
> Header `Sec-Fetch-Site` chỉ được gửi bởi các trình duyệt qua các kết nối an toàn (HTTPS). Nếu ứng dụng của bạn không được phục vụ qua HTTPS, xác minh origin sẽ không khả dụng và middleware sẽ quay lại xác thực CSRF token.

Nếu ứng dụng của bạn cần chấp nhận các request từ các subdomains (ví dụ, `dashboard.example.com` chấp nhận các request từ `example.com`), bạn có thể cho phép các request same-site ngoài các request same-origin:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(allowSameSite: true);
})
```

<a name="csrf-excluding-uris"></a>
### Loại trừ URIs khỏi CSRF Protection

Đôi khi bạn có thể muốn loại trừ một tập hợp các URIs khỏi CSRF protection. Ví dụ, nếu bạn đang sử dụng [Stripe](https://stripe.com) để xử lý thanh toán và đang sử dụng hệ thống webhook của họ, bạn sẽ cần loại trừ route handler webhook Stripe của bạn khỏi CSRF protection vì Stripe sẽ không biết CSRF token nào để gửi đến các routes của bạn.

Thông thường, bạn nên đặt các loại route này bên ngoài nhóm middleware `web` mà Laravel áp dụng cho tất cả các routes trong file `routes/web.php`. Tuy nhiên, bạn cũng có thể loại trừ các routes cụ thể bằng cách cung cấp URIs của chúng cho phương thức `preventRequestForgery` trong file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```

> [!NOTE]
> Để thuận tiện, middleware CSRF tự động bị vô hiệu hóa cho tất cả các routes khi [chạy tests](/docs/{{version}}/testing).

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

Ngoài việc kiểm tra CSRF token như một tham số POST, middleware `PreventRequestForgery` cũng sẽ kiểm tra header request `X-CSRF-TOKEN`. Bạn có thể, ví dụ, lưu trữ token trong một thẻ `meta` HTML:

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Sau đó, bạn có thể hướng dẫn một thư viện như jQuery tự động thêm token vào tất cả các request headers. Điều này cung cấp CSRF protection đơn giản, thuận tiện cho các ứng dụng dựa trên AJAX của bạn sử dụng công nghệ JavaScript cũ:

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel lưu trữ CSRF token hiện tại trong một cookie `XSRF-TOKEN` được mã hóa được bao gồm với mỗi response được tạo bởi framework. Bạn có thể sử dụng giá trị cookie để đặt header request `X-XSRF-TOKEN`.

Cookie này chủ yếu được gửi như một sự thuận tiện cho nhà phát triển vì một số frameworks và thư viện JavaScript, như Angular và Axios, tự động đặt giá trị của nó trong header `X-XSRF-TOKEN trên các request same-origin.

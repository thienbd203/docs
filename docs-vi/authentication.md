# Authentication

- [Giới thiệu](#introduction)
    - [Starter Kits](#starter-kits)
    - [Cân nhắc về Database](#introduction-database-considerations)
    - [Tổng quan về Ecosystem](#ecosystem-overview)
- [Bắt đầu nhanh với Authentication](#authentication-quickstart)
    - [Cài đặt Starter Kit](#install-a-starter-kit)
    - [Lấy thông tin User đã xác thực](#retrieving-the-authenticated-user)
    - [Bảo vệ Routes](#protecting-routes)
    - [Giới hạn đăng nhập](#login-throttling)
- [Xác thực User thủ công](#authenticating-users)
    - [Ghi nhớ User](#remembering-users)
    - [Các phương thức Authentication khác](#other-authentication-methods)
- [HTTP Basic Authentication](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication](#stateless-http-basic-authentication)
- [Đăng xuất](#logging-out)
    - [Hủy bỏ Session trên các thiết bị khác](#invalidating-sessions-on-other-devices)
- [Xác nhận mật khẩu](#password-confirmation)
    - [Cấu hình](#password-confirmation-configuration)
    - [Routing](#password-confirmation-routing)
    - [Bảo vệ Routes](#password-confirmation-protecting-routes)
- [Thêm Custom Guards](#adding-custom-guards)
    - [Closure Request Guards](#closure-request-guards)
- [Thêm Custom User Providers](#adding-custom-user-providers)
    - [User Provider Contract](#the-user-provider-contract)
    - [Authenticatable Contract](#the-authenticatable-contract)
- [Tự động rehash mật khẩu](#automatic-password-rehashing)
- [Social Authentication](/docs/{{version}}/socialite)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Nhiều ứng dụng web cung cấp cách để người dùng của họ xác thực với ứng dụng và "đăng nhập". Việc triển khai tính năng này trong các ứng dụng web có thể là một công việc phức tạp và tiềm ẩn rủi ro. Vì lý do này, Laravel nỗ lực cung cấp cho bạn các công cụ cần thiết để triển khai authentication nhanh chóng, an toàn và dễ dàng.

Về cơ bản, các cơ sở authentication của Laravel được tạo thành từ "guards" và "providers". Guards định nghĩa cách người dùng được xác thực cho mỗi request. Ví dụ, Laravel đi kèm với một `session` guard duy trì trạng thái bằng cách sử dụng session storage và cookies.

Providers định nghĩa cách người dùng được lấy từ persistent storage của bạn. Laravel đi kèm với hỗ trợ lấy người dùng bằng cách sử dụng [Eloquent](/docs/{{version}}/eloquent) và database query builder. Tuy nhiên, bạn có thể tự do định nghĩa thêm providers theo nhu cầu của ứng dụng.

File cấu hình authentication của ứng dụng nằm tại `config/auth.php`. File này chứa một số tùy chọn được ghi chép kỹ lưỡng để tinh chỉnh hành vi của các dịch vụ authentication của Laravel.

> [!NOTE]
> Guards và providers không nên bị nhầm lẫn với "roles" và "permissions". Để tìm hiểu thêm về việc ủy quyền các hành động của người dùng thông qua permissions, vui lòng tham khảo tài liệu [authorization](/docs/{{version}}/authorization).

<a name="starter-kits"></a>
### Starter Kits

Muốn bắt đầu nhanh? Cài đặt một [Laravel application starter kit](/docs/{{version}}/starter-kits) trong một ứng dụng Laravel mới. Sau khi migrate database của bạn, điều hướng trình duyệt của bạn đến `/register` hoặc bất kỳ URL nào khác được gán cho ứng dụng của bạn. Starter kits sẽ lo phần scaffolding toàn bộ hệ thống authentication của bạn!

**Ngay cả khi bạn chọn không sử dụng starter kit trong ứng dụng Laravel cuối cùng của mình, việc cài đặt một [starter kit](/docs/{{version}}/starter-kits) có thể là một cơ hội tuyệt vời để học cách triển khai tất cả các chức năng authentication của Laravel trong một dự án Laravel thực tế.** Vì Laravel starter kits chứa các authentication controllers, routes và views cho bạn, bạn có thể kiểm tra code trong các file này để học cách các tính năng authentication của Laravel có thể được triển khai.

<a name="introduction-database-considerations"></a>
### Cân nhắc về Database

Theo mặc định, Laravel bao gồm một `App\Models\User` [Eloquent model](/docs/{{version}}/eloquent) trong thư mục `app/Models` của bạn. Model này có thể được sử dụng với driver authentication Eloquent mặc định.

Nếu ứng dụng của bạn không sử dụng Eloquent, bạn có thể sử dụng `database` authentication provider sử dụng Laravel query builder. Nếu ứng dụng của bạn sử dụng MongoDB, hãy xem tài liệu [Laravel user authentication documentation](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/user-authentication/) chính thức của MongoDB.

Khi xây dựng database schema cho model `App\Models\User`, hãy đảm bảo cột password có độ dài ít nhất 60 ký tự. Tất nhiên, migration bảng `users` được bao gồm trong các ứng dụng Laravel mới đã tạo ra một cột vượt quá độ dài này.

Ngoài ra, bạn nên xác minh rằng bảng `users` (hoặc tương đương) của bạn chứa một cột `remember_token` kiểu string, nullable với độ dài 100 ký tự. Cột này sẽ được sử dụng để lưu trữ token cho người dùng chọn tùy chọn "remember me" khi đăng nhập vào ứng dụng của bạn. Một lần nữa, migration bảng `users` mặc định được bao gồm trong các ứng dụng Laravel mới đã chứa cột này.

<a name="ecosystem-overview"></a>
### Tổng quan về Ecosystem

Laravel cung cấp một số package liên quan đến authentication. Trước khi tiếp tục, chúng ta sẽ xem xét tổng quan về hệ sinh thái authentication trong Laravel và thảo luận về mục đích dự định của từng package.

Trước hết, hãy xem xét cách authentication hoạt động. Khi sử dụng trình duyệt web, người dùng sẽ cung cấp username và password của họ thông qua form đăng nhập. Nếu các thông tin đăng nhập này đúng, ứng dụng sẽ lưu trữ thông tin về người dùng đã xác thực trong [session](/docs/{{version}}/session) của người dùng. Một cookie được phát cho trình duyệt chứa session ID để các request tiếp theo đến ứng dụng có thể liên kết người dùng với session đúng. Sau khi nhận được session cookie, ứng dụng sẽ lấy dữ liệu session dựa trên session ID, ghi nhận rằng thông tin authentication đã được lưu trữ trong session, và sẽ coi người dùng là "đã xác thực".

Khi một remote service cần xác thực để truy cập API, cookies thường không được sử dụng cho authentication vì không có trình duyệt web. Thay vào đó, remote service gửi API token đến API trên mỗi request. Ứng dụng có thể xác thực token đến với bảng các API token hợp lệ và "xác thực" request như đang được thực hiện bởi người dùng liên kết với API token đó.

<a name="laravels-built-in-browser-authentication-services"></a>
#### Các dịch vụ Authentication trình duyệt tích hợp sẵn của Laravel

Laravel bao gồm các dịch vụ authentication và session tích hợp sẵn thường được truy cập thông qua các facade `Auth` và `Session`. Các tính năng này cung cấp authentication dựa trên cookie cho các request được khởi tạo từ trình duyệt web. Chúng cung cấp các phương thức cho phép bạn xác minh thông tin đăng nhập của người dùng và xác thực người dùng. Ngoài ra, các dịch vụ này sẽ tự động lưu trữ dữ liệu authentication thích hợp trong session của người dùng và phát hành session cookie của người dùng. Một cuộc thảo luận về cách sử dụng các dịch vụ này được chứa trong tài liệu này.

**Application Starter Kits**

Như đã thảo luận trong tài liệu này, bạn có thể tương tác với các dịch vụ authentication này thủ công để xây dựng lớp authentication riêng của ứng dụng. Tuy nhiên, để giúp bạn bắt đầu nhanh hơn, chúng tôi đã phát hành [free starter kits](/docs/{{version}}/starter-kits) cung cấp scaffolding hiện đại, mạnh mẽ cho toàn bộ lớp authentication.

<a name="laravels-api-authentication-services"></a>
#### Các dịch vụ Authentication API của Laravel

Laravel cung cấp hai package tùy chọn để hỗ trợ bạn quản lý API token và xác thực các request được thực hiện với API token: [Passport](/docs/{{version}}/passport) và [Sanctum](/docs/{{version}}/sanctum). Vui lòng lưu ý rằng các thư viện này và các thư viện authentication dựa trên cookie tích hợp sẵn của Laravel không loại trừ lẫn nhau. Các thư viện này chủ yếu tập trung vào authentication API token trong khi các dịch vụ authentication tích hợp sẵn tập trung vào authentication trình duyệt dựa trên cookie. Nhiều ứng dụng sẽ sử dụng cả các dịch vụ authentication dựa trên cookie tích hợp sẵn của Laravel và một trong các package authentication API của Laravel.

**Passport**

Passport là một OAuth2 authentication provider, cung cấp nhiều "grant types" OAuth2 khác nhau cho phép bạn phát hành các loại token khác nhau. Nói chung, đây là một package mạnh mẽ và phức tạp cho authentication API. Tuy nhiên, hầu hết các ứng dụng không yêu cầu các tính năng phức tạp được cung cấp bởi đặc tả OAuth2, điều này có thể gây nhầm lẫn cho cả người dùng và nhà phát triển. Ngoài ra, các nhà phát triển thường bị nhầm lẫn về cách xác thực các ứng dụng SPA hoặc ứng dụng di động bằng cách sử dụng các OAuth2 authentication providers như Passport.

**Sanctum**

Để phản hồi sự phức tạp của OAuth2 và sự nhầm lẫn của nhà phát triển, chúng tôi đã xây dựng một package authentication đơn giản và hợp lý hơn có thể xử lý cả các request web first-party từ trình duyệt web và các request API thông qua token. Mục tiêu này đã được hiện thực hóa với việc phát hành [Laravel Sanctum](/docs/{{version}}/sanctum), nên nên được coi là package authentication được ưu tiên và khuyến nghị cho các ứng dụng sẽ cung cấp UI web first-party ngoài API, hoặc sẽ được hỗ trợ bởi một single-page application (SPA) tồn tại riêng biệt với ứng dụng Laravel backend, hoặc các ứng dụng cung cấp client di động.

Laravel Sanctum là một package authentication web / API hybrid có thể quản lý toàn bộ quá trình authentication của ứng dụng. Điều này có thể thực hiện được vì khi các ứng dụng dựa trên Sanctum nhận được request, Sanctum sẽ xác định đầu tiên xem request có bao gồm session cookie tham chiếu đến một session đã xác thực hay không. Sanctum thực hiện điều này bằng cách gọi các dịch vụ authentication tích hợp sẵn của Laravel mà chúng ta đã thảo luận trước đó. Nếu request không được xác thực thông qua session cookie, Sanctum sẽ kiểm tra request để tìm API token. Nếu có API token, Sanctum sẽ xác thực request bằng cách sử dụng token đó. Để tìm hiểu thêm về quá trình này, vui lòng tham khảo tài liệu ["how it works"](/docs/{{version}}/sanctum#how-it-works) của Sanctum.

<a name="summary-choosing-your-stack"></a>
#### Tóm tắt và Chọn Stack của bạn

Tóm lại, nếu ứng dụng của bạn sẽ được truy cập bằng trình duyệt và bạn đang xây dựng một ứng dụng Laravel monolithic, ứng dụng của bạn sẽ sử dụng các dịch vụ authentication tích hợp sẵn của Laravel.

Tiếp theo, nếu ứng dụng của bạn cung cấp một API sẽ được tiêu thụ bởi các bên thứ ba, bạn sẽ chọn giữa [Passport](/docs/{{version}}/passport) hoặc [Sanctum](/docs/{{version}}/sanctum) để cung cấp authentication API token cho ứng dụng của bạn. Nói chung, Sanctum nên được ưu tiên khi có thể vì nó là một giải pháp đơn giản, hoàn chỉnh cho authentication API, authentication SPA và authentication di động, bao gồm hỗ trợ cho "scopes" hoặc "abilities".

Nếu bạn đang xây dựng một single-page application (SPA) sẽ được hỗ trợ bởi một Laravel backend, bạn nên sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum). Khi sử dụng Sanctum, bạn sẽ cần [thủ công triển khai các route authentication backend của riêng bạn](#authenticating-users) hoặc sử dụng [Laravel Fortify](/docs/{{version}}/fortify) như một dịch vụ backend authentication headless cung cấp các route và controllers cho các tính năng như đăng ký, đặt lại mật khẩu, xác minh email và nhiều hơn nữa.

Passport có thể được chọn khi ứng dụng của bạn hoàn toàn cần tất cả các tính năng được cung cấp bởi đặc tả OAuth2.

Và, nếu bạn muốn bắt đầu nhanh, chúng tôi rất vui được khuyến nghị [our application starter kits](/docs/{{version}}/starter-kits) như một cách nhanh chóng để bắt đầu một ứng dụng Laravel mới đã sử dụng stack authentication ưu tiên của chúng tôi là các dịch vụ authentication tích hợp sẵn của Laravel.

<a name="authentication-quickstart"></a>
## Bắt đầu nhanh với Authentication

> [!WARNING]
> Phần tài liệu này thảo luận về việc xác thực người dùng thông qua [Laravel application starter kits](/docs/{{version}}/starter-kits), bao gồm UI scaffolding để giúp bạn bắt đầu nhanh. Nếu bạn muốn tích hợp trực tiếp với các hệ thống authentication của Laravel, hãy xem tài liệu về [xác thực người dùng thủ công](#authenticating-users).

<a name="install-a-starter-kit"></a>
### Cài đặt Starter Kit

Trước hết, bạn nên [cài đặt một Laravel application starter kit](/docs/{{version}}/starter-kits). Starter kits của chúng tôi cung cấp các điểm bắt đầu được thiết kế đẹp mắt để tích hợp authentication vào ứng dụng Laravel mới của bạn.

<a name="retrieving-the-authenticated-user"></a>
### Lấy thông tin User đã xác thực

Sau khi tạo ứng dụng từ một starter kit và cho phép người dùng đăng ký và xác thực với ứng dụng của bạn, bạn thường sẽ cần tương tác với người dùng hiện tại đã xác thực. Khi xử lý một request đến, bạn có thể truy cập người dùng đã xác thực thông qua phương thức `user` của facade `Auth`:

```php
use Illuminate\Support\Facades\Auth;

// Retrieve the currently authenticated user...
$user = Auth::user();

// Retrieve the currently authenticated user's ID...
$id = Auth::id();
```

Ngoài ra, khi người dùng đã được xác thực, bạn có thể truy cập người dùng đã xác thực thông qua một instance `Illuminate\Http\Request`. Hãy nhớ rằng, các class được type-hint sẽ được tự động inject vào các phương thức controller của bạn. Bằng cách type-hint object `Illuminate\Http\Request`, bạn có thể truy cập thuận tiện người dùng đã xác thực từ bất kỳ phương thức controller nào trong ứng dụng của bạn thông qua phương thức `user` của request:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Update the flight information for an existing flight.
     */
    public function update(Request $request): RedirectResponse
    {
        $user = $request->user();

        // ...

        return redirect('/flights');
    }
}
```

<a name="determining-if-the-current-user-is-authenticated"></a>
#### Xác định xem User hiện tại đã được xác thực chưa

Để xác định xem người dùng thực hiện HTTP request đến đã được xác thực chưa, bạn có thể sử dụng phương thức `check` trên facade `Auth`. Phương thức này sẽ trả về `true` nếu người dùng đã được xác thực:

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // The user is logged in...
}
```

> [!NOTE]
> Mặc dù có thể xác định xem người dùng đã được xác thực bằng phương thức `check`, bạn thường sẽ sử dụng middleware để xác minh rằng người dùng đã được xác thực trước khi cho phép người dùng truy cập các route / controller nhất định. Để tìm hiểu thêm về điều này, hãy xem tài liệu về [bảo vệ routes](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Bảo vệ Routes

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để chỉ cho phép người dùng đã xác thực truy cập một route nhất định. Laravel đi kèm với một middleware `auth`, là một [middleware alias](/docs/{{version}}/middleware#middleware-aliases) cho class `Illuminate\Auth\Middleware\Authenticate`. Vì middleware này đã được alias nội bộ bởi Laravel, tất cả những gì bạn cần làm là đính kèm middleware vào định nghĩa route:

```php
Route::get('/flights', function () {
    // Only authenticated users may access this route...
})->middleware('auth');
```

<a name="redirecting-unauthenticated-users"></a>
#### Chuyển hướng User chưa xác thực

Khi middleware `auth` phát hiện một người dùng chưa xác thực, nó sẽ chuyển hướng người dùng đến [named route](/docs/{{version}}/routing#named-routes) `login`. Bạn có thể sửa đổi hành vi này bằng cách sử dụng phương thức `redirectGuestsTo` trong file `bootstrap/app.php` của ứng dụng:

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectGuestsTo('/login');

    // Using a closure...
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})
```

<a name="redirecting-authenticated-users"></a>
#### Chuyển hướng User đã xác thực

Khi middleware `guest` phát hiện một người dùng đã xác thực, nó sẽ chuyển hướng người dùng đến named route `dashboard` hoặc `home`. Bạn có thể sửa đổi hành vi này bằng cách sử dụng phương thức `redirectUsersTo` trong file `bootstrap/app.php` của ứng dụng:

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectUsersTo('/panel');

    // Using a closure...
    $middleware->redirectUsersTo(fn (Request $request) => route('panel'));
})
```

<a name="specifying-a-guard"></a>
#### Chỉ định Guard

Khi đính kèm middleware `auth` vào một route, bạn cũng có thể chỉ định "guard" nào nên được sử dụng để xác thực người dùng. Guard được chỉ định nên tương ứng với một trong các khóa trong mảng `guards` của file cấu hình `auth.php`:

```php
Route::get('/flights', function () {
    // Only authenticated users may access this route...
})->middleware('auth:admin');
```

<a name="login-throttling"></a>
### Giới hạn đăng nhập

Nếu bạn đang sử dụng một trong các [application starter kits](/docs/{{version}}/starter-kits) của chúng tôi, rate limiting sẽ được tự động áp dụng cho các lần thử đăng nhập. Theo mặc định, người dùng sẽ không thể đăng nhập trong một phút nếu họ không cung cấp thông tin đăng nhập đúng sau một số lần thử. Việc giới hạn này là duy nhất cho username / địa chỉ email của người dùng và địa chỉ IP của họ.

> [!NOTE]
> Nếu bạn muốn rate limit các route khác trong ứng dụng của bạn, hãy xem [rate limiting documentation](/docs/{{version}}/routing#rate-limiting).

<a name="authenticating-users"></a>
## Xác thực User thủ công

Bạn không bắt buộc phải sử dụng authentication scaffolding được bao gồm với [Laravel application starter kits](/docs/{{version}}/starter-kits). Nếu bạn chọn không sử dụng scaffolding này, bạn sẽ cần quản lý authentication người dùng bằng cách sử dụng các class authentication của Laravel trực tiếp. Đừng lo, nó rất đơn giản!

Chúng ta sẽ truy cập các dịch vụ authentication của Laravel thông qua [facade](/docs/{{version}}/facades) `Auth`, vì vậy chúng ta sẽ cần đảm bảo import facade `Auth` ở đầu class. Tiếp theo, hãy xem phương thức `attempt`. Phương thức `attempt` thường được sử dụng để xử lý các lần thử authentication từ form "login" của ứng dụng. Nếu authentication thành công, bạn nên regenerate [session](/docs/{{version}}/session) của người dùng để ngăn chặn [session fixation](https://en.wikipedia.org/wiki/Session_fixation):

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Handle an authentication attempt.
     */
    public function authenticate(Request $request): RedirectResponse
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ])->onlyInput('email');
    }
}
```

Phương thức `attempt` chấp nhận một mảng các cặp key / value làm đối số đầu tiên. Các giá trị trong mảng sẽ được sử dụng để tìm người dùng trong bảng database của bạn. Vì vậy, trong ví dụ trên, người dùng sẽ được lấy theo giá trị của cột `email`. Nếu người dùng được tìm thấy, mật khẩu đã hash được lưu trữ trong database sẽ được so sánh với giá trị `password` được truyền cho phương thức thông qua mảng. Bạn không nên hash giá trị `password` của request đến, vì framework sẽ tự động hash giá trị trước khi so sánh nó với mật khẩu đã hash trong database. Một session đã xác thực sẽ được bắt đầu cho người dùng nếu hai mật khẩu đã hash khớp nhau.

Hãy nhớ rằng, các dịch vụ authentication của Laravel sẽ lấy người dùng từ database của bạn dựa trên cấu hình "provider" của authentication guard của bạn. Trong file cấu hình `config/auth.php` mặc định, Eloquent user provider được chỉ định và được hướng dẫn sử dụng model `App\Models\User` khi lấy người dùng. Bạn có thể thay đổi các giá trị này trong file cấu hình của bạn dựa trên nhu cầu của ứng dụng.

Phương thức `attempt` sẽ trả về `true` nếu authentication thành công. Nếu không, `false` sẽ được trả về.

Phương thức `intended` được cung cấp bởi redirector của Laravel sẽ chuyển hướng người dùng đến URL họ đang cố gắng truy cập trước khi bị chặn bởi authentication middleware. Một URI dự phòng có thể được cung cấp cho phương thức này trong trường hợp đích dự định không khả dụng.

<a name="specifying-additional-conditions"></a>
#### Chỉ định các điều kiện bổ sung

Nếu bạn muốn, bạn cũng có thể thêm các điều kiện query bổ sung vào query authentication ngoài email và mật khẩu của người dùng. Để thực hiện điều này, chúng ta có thể đơn giản thêm các điều kiện query vào mảng được truyền cho phương thức `attempt`. Ví dụ, chúng ta có thể xác minh rằng người dùng được đánh dấu là "active":

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // Authentication was successful...
}
```

Đối với các điều kiện query phức tạp, bạn có thể cung cấp một closure trong mảng credentials của bạn. Closure này sẽ được gọi với query instance, cho phép bạn tùy chỉnh query dựa trên nhu cầu của ứng dụng:

```php
use Illuminate\Database\Eloquent\Builder;

if (Auth::attempt([
    'email' => $email,
    'password' => $password,
    fn (Builder $query) => $query->has('activeSubscription'),
])) {
    // Authentication was successful...
}
```

> [!WARNING]
> Trong các ví dụ này, `email` không phải là một tùy chọn bắt buộc, nó chỉ được sử dụng như một ví dụ. Bạn nên sử dụng bất kỳ tên cột nào tương ứng với "username" trong bảng database của bạn.

Phương thức `attemptWhen`, nhận một closure làm đối số thứ hai, có thể được sử dụng để thực hiện kiểm tra kỹ hơn về người dùng tiềm năng trước khi thực sự xác thực người dùng. Closure nhận người dùng tiềm năng và nên trả về `true` hoặc `false` để chỉ định xem người dùng có thể được xác thực hay không:

```php
if (Auth::attemptWhen([
    'email' => $email,
    'password' => $password,
], function (User $user) {
    return $user->isNotBanned();
})) {
    // Authentication was successful...
}
```

<a name="accessing-specific-guard-instances"></a>
#### Truy cập các Guard Instance cụ thể

Thông qua phương thức `guard` của facade `Auth`, bạn có thể chỉ định guard instance nào bạn muốn sử dụng khi xác thực người dùng. Điều này cho phép bạn quản lý authentication cho các phần riêng biệt của ứng dụng bằng cách sử dụng các model hoặc bảng người dùng hoàn toàn riêng biệt.

Tên guard được truyền cho phương thức `guard` nên tương ứng với một trong các guards được cấu hình trong file cấu hình `auth.php`:

```php
if (Auth::guard('admin')->attempt($credentials)) {
    // ...
}
```

<a name="remembering-users"></a>
### Ghi nhớ User

Nhiều ứng dụng web cung cấp một checkbox "remember me" trên form đăng nhập của họ. Nếu bạn muốn cung cấp chức năng "remember me" trong ứng dụng của bạn, bạn có thể truyền một giá trị boolean làm đối số thứ hai cho phương thức `attempt`.

Khi giá trị này là `true`, Laravel sẽ giữ người dùng được xác thực vô thời hạn hoặc cho đến khi họ đăng xuất thủ công. Bảng `users` của bạn phải bao gồm cột string `remember_token`, sẽ được sử dụng để lưu trữ token "remember me". Migration bảng `users` được bao gồm với các ứng dụng Laravel mới đã bao gồm cột này:

```php
use Illuminate\Support\Facades\Auth;

if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```

Nếu ứng dụng của bạn cung cấp chức năng "remember me", bạn có thể sử dụng phương thức `viaRemember` để xác định xem người dùng hiện tại đã xác thực có được xác thực bằng cookie "remember me" hay không:

```php
use Illuminate\Support\Facades\Auth;

if (Auth::viaRemember()) {
    // ...
}
```

<a name="other-authentication-methods"></a>
### Các phương thức Authentication khác

<a name="authenticate-a-user-instance"></a>
#### Xác thực một User Instance

Nếu bạn cần đặt một user instance hiện tại làm người dùng hiện tại đã xác thực, bạn có thể truyền user instance cho phương thức `login` của facade `Auth`. User instance được cung cấp phải là một implementation của [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Auth\Authenticatable`. Model `App\Models\User` được bao gồm với Laravel đã implement interface này. Phương thức authentication này hữu ích khi bạn đã có một user instance hợp lệ, chẳng hạn như ngay sau khi người dùng đăng ký với ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Auth;

Auth::login($user);
```

Bạn có thể truyền một giá trị boolean làm đối số thứ hai cho phương thức `login`. Giá trị này chỉ định xem chức năng "remember me" có được mong muốn cho session đã xác thực hay không. Hãy nhớ rằng, điều này có nghĩa là session sẽ được xác thực vô thời hạn hoặc cho đến khi người dùng đăng xuất thủ công khỏi ứng dụng:

```php
Auth::login($user, $remember = true);
```

Nếu cần, bạn có thể chỉ định một authentication guard trước khi gọi phương thức `login`:

```php
Auth::guard('admin')->login($user);
```

<a name="authenticate-a-user-by-id"></a>
#### Xác thực User theo ID

Để xác thực một người dùng bằng cách sử dụng primary key của bản ghi database của họ, bạn có thể sử dụng phương thức `loginUsingId`. Phương thức này chấp nhận primary key của người dùng bạn muốn xác thực:

```php
Auth::loginUsingId(1);
```

Bạn có thể truyền một giá trị boolean cho đối số `remember` của phương thức `loginUsingId`. Giá trị này chỉ định xem chức năng "remember me" có được mong muốn cho session đã xác thực hay không. Hãy nhớ rằng, điều này có nghĩa là session sẽ được xác thực vô thời hạn hoặc cho đến khi người dùng đăng xuất thủ công khỏi ứng dụng:

```php
Auth::loginUsingId(1, remember: true);
```

<a name="authenticate-a-user-once"></a>
#### Xác thực User một lần

Bạn có thể sử dụng phương thức `once` để xác thực một người dùng với ứng dụng cho một request duy nhất. Không có session hoặc cookie nào sẽ được sử dụng khi gọi phương thức này, và event `Login` sẽ không được dispatch:

```php
if (Auth::once($credentials)) {
    // ...
}
```

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) cung cấp một cách nhanh chóng để xác thực người dùng của ứng dụng của bạn mà không cần thiết lập một trang "login" chuyên dụng. Để bắt đầu, đính kèm middleware `auth.basic` [middleware](/docs/{{version}}/middleware) vào một route. Middleware `auth.basic` được bao gồm với framework Laravel, vì vậy bạn không cần định nghĩa nó:

```php
Route::get('/profile', function () {
    // Only authenticated users may access this route...
})->middleware('auth.basic');
```

Khi middleware đã được đính kèm vào route, bạn sẽ tự động được nhắc nhập thông tin đăng nhập khi truy cập route trong trình duyệt của bạn. Theo mặc định, middleware `auth.basic` sẽ giả định cột `email` trên bảng database `users` của bạn là "username" của người dùng.

<a name="a-note-on-fastcgi"></a>
#### Lưu ý về FastCGI

Nếu bạn đang sử dụng [PHP FastCGI](https://www.php.net/manual/en/install.fpm.php) và Apache để phục vụ ứng dụng Laravel của bạn, HTTP Basic authentication có thể không hoạt động đúng. Để sửa các vấn đề này, các dòng sau có thể được thêm vào file `.htaccess` của ứng dụng:

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

Bạn cũng có thể sử dụng HTTP Basic Authentication mà không cần đặt cookie user identifier trong session. Điều này chủ yếu hữu ích nếu bạn chọn sử dụng HTTP Authentication để xác thực các request đến API của ứng dụng. Để thực hiện điều này, [định nghĩa một middleware](/docs/{{version}}/middleware) gọi phương thức `onceBasic`. Nếu không có response nào được trả về bởi phương thức `onceBasic`, request có thể được chuyển tiếp thêm vào ứng dụng:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Symfony\Component\HttpFoundation\Response;

class AuthenticateOnceWithBasicAuth
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

Tiếp theo, đính kèm middleware vào một route:

```php
Route::get('/api/user', function () {
    // Only authenticated users may access this route...
})->middleware(AuthenticateOnceWithBasicAuth::class);
```

<a name="logging-out"></a>
## Đăng xuất

Để đăng xuất thủ công người dùng khỏi ứng dụng của bạn, bạn có thể sử dụng phương thức `logout` được cung cấp bởi facade `Auth`. Điều này sẽ xóa thông tin authentication khỏi session của người dùng để các request tiếp theo không được xác thực.

Ngoài việc gọi phương thức `logout`, bạn nên hủy session của người dùng và regenerate [CSRF token](/docs/{{version}}/csrf) của họ. Sau khi đăng xuất người dùng, bạn thường sẽ chuyển hướng người dùng đến root của ứng dụng:

```php
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

/**
 * Log the user out of the application.
 */
public function logout(Request $request): RedirectResponse
{
    Auth::logout();

    $request->session()->invalidate();

    $request->session()->regenerateToken();

    return redirect('/');
}
```

<a name="invalidating-sessions-on-other-devices"></a>
### Hủy bỏ Session trên các thiết bị khác

Laravel cũng cung cấp một cơ chế để hủy bỏ và "đăng xuất" các session của người dùng đang hoạt động trên các thiết bị khác mà không hủy bỏ session trên thiết bị hiện tại của họ. Tính năng này thường được sử dụng khi người dùng đang thay đổi hoặc cập nhật mật khẩu của họ và bạn muốn hủy bỏ các session trên các thiết bị khác trong khi giữ cho thiết bị hiện tại được xác thực.

Trước khi bắt đầu, bạn nên đảm bảo rằng middleware `Illuminate\Session\Middleware\AuthenticateSession` được bao gồm trên các routes nên nhận authentication session. Thông thường, bạn nên đặt middleware này trên một định nghĩa route group để nó có thể được áp dụng cho phần lớn các route của ứng dụng. Theo mặc định, middleware `AuthenticateSession` có thể được đính kèm vào một route bằng cách sử dụng [middleware alias](/docs/{{version}}/middleware#middleware-aliases) `auth.session`:

```php
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

Sau đó, bạn có thể sử dụng phương thức `logoutOtherDevices` được cung cấp bởi facade `Auth`. Phương thức này yêu cầu người dùng xác nhận mật khẩu hiện tại của họ, mà ứng dụng của bạn nên chấp nhận thông qua một form nhập liệu:

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

Khi phương thức `logoutOtherDevices` được gọi, các session khác của người dùng sẽ bị hủy hoàn toàn, có nghĩa là họ sẽ được "đăng xuất" khỏi tất cả các guards mà họ đã được xác thực trước đó.

<a name="password-confirmation"></a>
## Xác nhận mật khẩu

Khi xây dựng ứng dụng của bạn, bạn có thể thỉnh thoảng có các hành động nên yêu cầu người dùng xác nhận mật khẩu của họ trước khi hành động được thực hiện hoặc trước khi người dùng được chuyển hướng đến một khu vực nhạy cảm của ứng dụng. Laravel bao gồm middleware tích hợp sẵn để làm cho quá trình này trở nên dễ dàng. Việc triển khai tính năng này sẽ yêu cầu bạn định nghĩa hai route: một route để hiển thị view yêu cầu người dùng xác nhận mật khẩu và một route khác để xác nhận rằng mật khẩu hợp lệ và chuyển hướng người dùng đến đích dự định của họ.

> [!NOTE]
> Tài liệu sau thảo luận về cách tích hợp trực tiếp với các tính năng xác nhận mật khẩu của Laravel; tuy nhiên, nếu bạn muốn bắt đầu nhanh hơn, [Laravel application starter kits](/docs/{{version}}/starter-kits) bao gồm hỗ trợ cho tính năng này!

<a name="password-confirmation-configuration"></a>
### Cấu hình

Sau khi xác nhận mật khẩu, người dùng sẽ không được yêu cầu xác nhận mật khẩu lại trong ba giờ. Tuy nhiên, bạn có thể cấu hình khoảng thời gian trước khi người dùng được nhắc lại mật khẩu bằng cách thay đổi giá trị cấu hình `password_timeout` trong file cấu hình `config/auth.php` của ứng dụng.

<a name="password-confirmation-routing"></a>
### Routing

<a name="the-password-confirmation-form"></a>
#### Form xác nhận mật khẩu

Trước hết, chúng ta sẽ định nghĩa một route để hiển thị view yêu cầu người dùng xác nhận mật khẩu:

```php
Route::get('/confirm-password', function () {
    return view('auth.confirm-password');
})->middleware('auth')->name('password.confirm');
```

Như bạn có thể mong đợi, view được trả về bởi route này nên có một form chứa trường `password`. Ngoài ra, hãy thoải mái bao gồm văn bản trong view giải thích rằng người dùng đang nhập vào một khu vực được bảo vệ của ứng dụng và phải xác nhận mật khẩu của họ.

<a name="confirming-the-password"></a>
#### Xác nhận mật khẩu

Tiếp theo, chúng ta sẽ định nghĩa một route sẽ xử lý form request từ view "confirm password". Route này sẽ chịu trách nhiệm xác thực mật khẩu và chuyển hướng người dùng đến đích dự định của họ:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

Route::post('/confirm-password', function (Request $request) {
    if (! Hash::check($request->password, $request->user()->password)) {
        return back()->withErrors([
            'password' => ['The provided password does not match our records.']
        ]);
    }

    $request->session()->passwordConfirmed();

    return redirect()->intended();
})->middleware(['auth', 'throttle:6,1']);
```

Trước khi tiếp tục, hãy xem xét route này chi tiết hơn. Trước hết, trường `password` của request được xác định thực sự khớp với mật khẩu của người dùng đã xác thực. Nếu mật khẩu hợp lệ, chúng ta cần thông báo cho session của Laravel rằng người dùng đã xác nhận mật khẩu của họ. Phương thức `passwordConfirmed` sẽ đặt một timestamp trong session của người dùng mà Laravel có thể sử dụng để xác định khi người dùng cuối cùng xác nhận mật khẩu của họ. Cuối cùng, chúng ta có thể chuyển hướng người dùng đến đích dự định của họ.

<a name="password-confirmation-protecting-routes"></a>
### Bảo vệ Routes

Bạn nên đảm bảo rằng bất kỳ route nào thực hiện một hành động yêu cầu xác nhận mật khẩu gần đây được gán middleware `password.confirm`. Middleware này được bao gồm với cài đặt mặc định của Laravel và sẽ tự động lưu trữ đích dự định của người dùng trong session để người dùng có thể được chuyển hướng đến vị trí đó sau khi xác nhận mật khẩu của họ. Sau khi lưu trữ đích dự định của người dùng trong session, middleware sẽ chuyển hướng người dùng đến [named route](/docs/{{version}}/routing#named-routes) `password.confirm`:

```php
Route::get('/settings', function () {
    // ...
})->middleware(['password.confirm']);

Route::post('/settings', function () {
    // ...
})->middleware(['password.confirm']);
```

<a name="adding-custom-guards"></a>
## Thêm Custom Guards

Bạn có thể định nghĩa các authentication guard của riêng bạn bằng cách sử dụng phương thức `extend` trên facade `Auth`. Bạn nên đặt cuộc gọi của bạn đến phương thức `extend` trong một [service provider](/docs/{{version}}/providers). Vì Laravel đã đi kèm với một `AppServiceProvider`, chúng ta có thể đặt code trong provider đó:

```php
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Auth::extend('jwt', function (Application $app, string $name, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\Guard...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

Như bạn có thể thấy trong ví dụ trên, callback được truyền cho phương thức `extend` nên trả về một implementation của `Illuminate\Contracts\Auth\Guard`. Interface này chứa một số phương thức bạn sẽ cần implement để định nghĩa một custom guard. Khi custom guard của bạn đã được định nghĩa, bạn có thể tham chiếu guard trong cấu hình `guards` của file cấu hình `auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

<a name="closure-request-guards"></a>
### Closure Request Guards

Cách đơn giản nhất để triển khai một hệ thống authentication tùy chỉnh dựa trên HTTP request là bằng cách sử dụng phương thức `Auth::viaRequest`. Phương thức này cho phép bạn nhanh chóng định nghĩa quá trình authentication của bạn bằng cách sử dụng một closure duy nhất.

Để bắt đầu, gọi phương thức `Auth::viaRequest` trong phương thức `boot` của `AppServiceProvider` của ứng dụng. Phương thức `viaRequest` chấp nhận tên driver authentication làm đối số đầu tiên. Tên này có thể là bất kỳ chuỗi nào mô tả custom guard của bạn. Đối số thứ hai được truyền cho phương thức nên là một closure nhận HTTP request đến và trả về một user instance hoặc, nếu authentication thất bại, `null`:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Auth::viaRequest('custom-token', function (Request $request) {
        return User::where('token', (string) $request->token)->first();
    });
}
```

Khi driver authentication tùy chỉnh của bạn đã được định nghĩa, bạn có thể cấu hình nó như một driver trong cấu hình `guards` của file cấu hình `auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

Cuối cùng, bạn có thể tham chiếu guard khi gán authentication middleware cho một route:

```php
Route::middleware('auth:api')->group(function () {
    // ...
});
```

<a name="adding-custom-user-providers"></a>
## Thêm Custom User Providers

Nếu bạn không sử dụng một database quan hệ truyền thống để lưu trữ người dùng của bạn, bạn sẽ cần mở rộng Laravel với user provider authentication của riêng bạn. Chúng ta sẽ sử dụng phương thức `provider` trên facade `Auth` để định nghĩa một custom user provider. User provider resolver nên trả về một implementation của `Illuminate\Contracts\Auth\UserProvider`:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoUserProvider;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Auth::provider('mongo', function (Application $app, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...

            return new MongoUserProvider($app->make('mongo.connection'));
        });
    }
}
```

Sau khi bạn đã đăng ký provider bằng cách sử dụng phương thức `provider`, bạn có thể chuyển sang user provider mới trong file cấu hình `auth.php`. Trước hết, định nghĩa một `provider` sử dụng driver mới của bạn:

```php
'providers' => [
    'users' => [
        'driver' => 'mongo',
    ],
],
```

Cuối cùng, bạn có thể tham chiếu provider này trong cấu hình `guards` của bạn:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

<a name="the-user-provider-contract"></a>
### User Provider Contract

Các implementation `Illuminate\Contracts\Auth\UserProvider` chịu trách nhiệm lấy một implementation `Illuminate\Contracts\Auth\Authenticatable` từ một hệ thống lưu trữ persistent, chẳng hạn như MySQL, MongoDB, v.v. Hai interface này cho phép các cơ chế authentication của Laravel tiếp tục hoạt động bất kể cách dữ liệu người dùng được lưu trữ hoặc loại class nào được sử dụng để đại diện cho người dùng đã xác thực:

Hãy xem contract `Illuminate\Contracts\Auth\UserProvider`:

```php
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider
{
    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);
    public function rehashPasswordIfRequired(Authenticatable $user, array $credentials, bool $force = false);
}
```

Hàm `retrieveById` thường nhận một key đại diện cho người dùng, chẳng hạn như một ID auto-increment từ database MySQL. Implementation `Authenticatable` khớp với ID nên được lấy và trả về bởi phương thức.

Hàm `retrieveByToken` lấy một người dùng bằng `$identifier` duy nhất của họ và token "remember me" `$token`, thường được lưu trữ trong một cột database như `remember_token`. Như với phương thức trước đó, implementation `Authenticatable` với giá trị token khớp nên được trả về bởi phương thức này.

Phương thức `updateRememberToken` cập nhật `remember_token` của instance `$user` với `$token` mới. Một token mới được gán cho người dùng khi một lần thử authentication "remember me" thành công hoặc khi người dùng đang đăng xuất.

Phương thức `retrieveByCredentials` nhận mảng credentials được truyền cho phương thức `Auth::attempt` khi cố gắng xác thực với một ứng dụng. Phương thức sau đó nên "query" persistent storage cơ bản cho người dùng khớp với các credentials đó. Thông thường, phương thức này sẽ chạy một query với điều kiện "where" tìm kiếm một bản ghi người dùng với "username" khớp với giá trị của `$credentials['username']`. Phương thức nên trả về một implementation của `Authenticatable`. **Phương thức này không nên cố gắng thực hiện bất kỳ xác thực mật khẩu hoặc authentication nào.**

Phương thức `validateCredentials` nên so sánh `$user` được cung cấp với `$credentials` để xác thực người dùng. Ví dụ, phương thức này thường sẽ sử dụng phương thức `Hash::check` để so sánh giá trị của `$user->getAuthPassword()` với giá trị của `$credentials['password']`. Phương thức nên trả về `true` hoặc `false` chỉ định xem mật khẩu có hợp lệ hay không.

Phương thức `rehashPasswordIfRequired` nên rehash mật khẩu của `$user` được cung cấp nếu cần và được hỗ trợ. Ví dụ, phương thức này thường sẽ sử dụng phương thức `Hash::needsRehash` để xác định xem giá trị `$credentials['password']` có cần được rehash hay không. Nếu mật khẩu cần được rehash, phương thức nên sử dụng phương thức `Hash::make` để rehash mật khẩu và cập nhật bản ghi người dùng trong persistent storage cơ bản.

<a name="the-authenticatable-contract"></a>
### Authenticatable Contract

Bây giờ chúng ta đã khám phá từng phương thức trên `UserProvider`, hãy xem contract `Authenticatable`. Hãy nhớ rằng, user providers nên trả về các implementation của interface này từ các phương thức `retrieveById`, `retrieveByToken`, và `retrieveByCredentials`:

```php
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable
{
    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPasswordName();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();
}
```

Interface này đơn giản. Phương thức `getAuthIdentifierName` nên trả về tên của cột "primary key" cho người dùng và phương thức `getAuthIdentifier` nên trả về "primary key" của người dùng. Khi sử dụng một backend MySQL, điều này có thể là primary key auto-increment được gán cho bản ghi người dùng. Phương thức `getAuthPasswordName` nên trả về tên của cột mật khẩu của người dùng. Phương thức `getAuthPassword` nên trả về mật khẩu đã hash của người dùng.

Interface này cho phép hệ thống authentication hoạt động với bất kỳ class "user" nào, bất kể ORM hoặc lớp trừu tượng lưu trữ nào bạn đang sử dụng. Theo mặc định, Laravel bao gồm một class `App\Models\User` trong thư mục `app/Models` implement interface này.

<a name="automatic-password-rehashing"></a>
## Tự động rehash mật khẩu

Thuật toán hash mật khẩu mặc định của Laravel là bcrypt. "Work factor" cho bcrypt hashes có thể được điều chỉnh thông qua file cấu hình `config/hashing.php` của ứng dụng hoặc biến môi trường `BCRYPT_ROUNDS`.

Thông thường, work factor bcrypt nên được tăng theo thời gian khi sức mạnh xử lý CPU / GPU tăng. Nếu bạn tăng work factor bcrypt cho ứng dụng của bạn, Laravel sẽ gracefully và tự động rehash mật khẩu người dùng khi người dùng xác thực với ứng dụng của bạn thông qua Laravel starter kits hoặc khi bạn [xác thực người dùng thủ công](#authenticating-users) thông qua phương thức `attempt`.

Thông thường, việc rehash mật khẩu tự động không nên làm gián đoạn ứng dụng của bạn; tuy nhiên, bạn có thể vô hiệu hóa hành vi này bằng cách xuất bản file cấu hình `hashing`:

```shell
php artisan config:publish hashing
```

Khi file cấu hình đã được xuất bản, bạn có thể đặt giá trị cấu hình `rehash_on_login` thành `false`:

```php
'rehash_on_login' => false,
```

<a name="events"></a>
## Events

Laravel dispatch một loạt [events](/docs/{{version}}/events) trong quá trình authentication. Bạn có thể [định nghĩa listeners](/docs/{{version}}/events) cho bất kỳ events nào sau đây:

<div class="overflow-auto">

| Event Name                                     |
| ---------------------------------------------- |
| `Illuminate\Auth\Events\Registered`            |
| `Illuminate\Auth\Events\Attempting`            |
| `Illuminate\Auth\Events\Authenticated`         |
| `Illuminate\Auth\Events\Login`                 |
| `Illuminate\Auth\Events\Failed`                |
| `Illuminate\Auth\Events\Validated`             |
| `Illuminate\Auth\Events\Verified`              |
| `Illuminate\Auth\Events\Logout`                |
| `Illuminate\Auth\Events\CurrentDeviceLogout`   |
| `Illuminate\Auth\Events\OtherDeviceLogout`     |
| `Illuminate\Auth\Events\Lockout`               |
| `Illuminate\Auth\Events\PasswordReset`         |
| `Illuminate\Auth\Events\PasswordResetLinkSent` |

</div>

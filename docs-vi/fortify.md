# Laravel Fortify

- [Giới thiệu](#introduction)
    - [Fortify là gì?](#what-is-fortify)
    - [Khi nào nên sử dụng Fortify?](#when-should-i-use-fortify)
- [Cài đặt](#installation)
    - [Tính năng Fortify](#fortify-features)
    - [Vô hiệu hóa Views](#disabling-views)
- [Authentication](#authentication)
    - [Tùy chỉnh User Authentication](#customizing-user-authentication)
    - [Tùy chỉnh Authentication Pipeline](#customizing-the-authentication-pipeline)
    - [Tùy chỉnh Redirects](#customizing-authentication-redirects)
- [Two-Factor Authentication](#two-factor-authentication)
    - [Bật Two-Factor Authentication](#enabling-two-factor-authentication)
    - [Xác thực với Two-Factor Authentication](#authenticating-with-two-factor-authentication)
    - [Tắt Two-Factor Authentication](#disabling-two-factor-authentication)
- [Passkeys](#passkeys)
    - [Bật Passkeys](#enabling-passkeys)
    - [JavaScript Client](#passkeys-javascript-client)
    - [Xác thực với Passkeys](#authenticating-with-passkeys)
    - [Xác nhận Password với Passkeys](#confirming-password-with-passkeys)
    - [Đăng ký Passkeys](#registering-passkeys)
    - [Xóa Passkeys](#deleting-passkeys)
- [Đăng ký](#registration)
    - [Tùy chỉnh Đăng ký](#customizing-registration)
- [Password Reset](#password-reset)
    - [Yêu cầu Link Reset Password](#requesting-a-password-reset-link)
    - [Reset Password](#resetting-the-password)
    - [Tùy chỉnh Password Resets](#customizing-password-resets)
- [Email Verification](#email-verification)
    - [Bảo vệ Routes](#protecting-routes)
- [Password Confirmation](#password-confirmation)

<a name="introduction"></a>
## Giới thiệu

[Laravel Fortify](https://github.com/laravel/fortify) là một triển khai backend authentication frontend-agnostic cho Laravel. Fortify đăng ký các routes và controllers cần thiết để triển khai tất cả các tính năng authentication của Laravel, bao gồm login, registration, password reset, email verification, và nhiều hơn nữa. Sau khi cài đặt Fortify, bạn có thể chạy command `route:list` của Artisan để xem các routes mà Fortify đã đăng ký.

Vì Fortify không cung cấp user interface của riêng nó, nó được thiết kế để kết hợp với user interface của bạn thực hiện các request đến các routes mà nó đăng ký. Chúng ta sẽ thảo luận chính xác cách thực hiện các request đến các routes này trong phần còn lại của tài liệu này.

> [!NOTE]
> Hãy nhớ, Fortify là một package được thiết kế để giúp bạn bắt đầu triển khai các tính năng authentication của Laravel. **Bạn không bắt buộc phải sử dụng nó.** Bạn luôn tự do tương tác thủ công với các dịch vụ authentication của Laravel bằng cách làm theo tài liệu có sẵn trong tài liệu [authentication](/docs/{{version}}/authentication), [password reset](/docs/{{version}}/passwords), và [email verification](/docs/{{version}}/verification).

<a name="what-is-fortify"></a>
### Fortify là gì?

Như đã đề cập trước đó, Laravel Fortify là một triển khai backend authentication frontend-agnostic cho Laravel. Fortify đăng ký các routes và controllers cần thiết để triển khai tất cả các tính năng authentication của Laravel, bao gồm login, registration, password reset, email verification, và nhiều hơn nữa.

**Bạn không bắt buộc phải sử dụng Fortify để sử dụng các tính năng authentication của Laravel.** Bạn luôn tự do tương tác thủ công với các dịch vụ authentication của Laravel bằng cách làm theo tài liệu có sẵn trong tài liệu [authentication](/docs/{{version}}/authentication), [password reset](/docs/{{version}}/passwords), và [email verification](/docs/{{version}}/verification).

Nếu bạn mới bắt đầu với Laravel, bạn có thể muốn khám phá [application starter kits của chúng tôi](/docs/{{version}}/starter-kits). Application starter kits của Laravel sử dụng Fortify nội bộ để cung cấp authentication scaffolding cho ứng dụng của bạn bao gồm một user interface được xây dựng với [Tailwind CSS](https://tailwindcss.com). Điều này cho phép bạn nghiên cứu và làm quen với các tính năng authentication của Laravel.

Laravel Fortify về cơ bản lấy các routes và controllers của application starter kits của chúng tôi và cung cấp chúng như một package không bao gồm user interface. Điều này cho phép bạn vẫn nhanh chóng scaffold triển khai backend của lớp authentication của ứng dụng mà không bị ràng buộc với bất kỳ quan điểm frontend cụ thể nào.

<a name="when-should-i-use-fortify"></a>
### Khi nào nên sử dụng Fortify?

Bạn có thể tự hỏi khi nào là thích hợp để sử dụng Laravel Fortify. Đầu tiên, nếu bạn đang sử dụng một trong các [application starter kits](/docs/{{version}}/starter-kits) của Laravel, bạn không cần cài đặt Laravel Fortify vì tất cả application starter kits của Laravel đều sử dụng Fortify và đã cung cấp triển khai authentication đầy đủ.

Nếu bạn không sử dụng application starter kit và ứng dụng của bạn cần các tính năng authentication, bạn có hai lựa chọn: triển khai thủ công các tính năng authentication của ứng dụng của bạn hoặc sử dụng Laravel Fortify để cung cấp triển khai backend của các tính năng này.

Nếu bạn chọn cài đặt Fortify, user interface của bạn sẽ thực hiện các request đến các routes authentication của Fortify được chi tiết trong tài liệu này để xác thực và đăng ký người dùng.

Nếu bạn chọn tương tác thủ công với các dịch vụ authentication của Laravel thay vì sử dụng Fortify, bạn có thể làm như vậy bằng cách làm theo tài liệu có sẵn trong tài liệu [authentication](/docs/{{version}}/authentication), [password reset](/docs/{{version}}/passwords), và [email verification](/docs/{{version}}/verification).

<a name="laravel-fortify-and-laravel-sanctum"></a>
#### Laravel Fortify và Laravel Sanctum

Một số nhà phát triển bị nhầm lẫn về sự khác biệt giữa [Laravel Sanctum](/docs/{{version}}/sanctum) và Laravel Fortify. Vì hai package này giải quyết hai vấn đề khác nhau nhưng có liên quan, Laravel Fortify và Laravel Sanctum không phải là các package loại trừ lẫn nhau hoặc cạnh tranh.

Laravel Sanctum chỉ quan tâm đến việc quản lý API tokens và xác thực người dùng hiện có sử dụng session cookies hoặc tokens. Sanctum không cung cấp bất kỳ routes nào xử lý user registration, password reset, v.v.

Nếu bạn đang cố gắng xây dựng thủ công lớp authentication cho một ứng dụng cung cấp API hoặc đóng vai trò backend cho một single-page application, hoàn toàn có thể bạn sẽ sử dụng cả Laravel Fortify (cho user registration, password reset, v.v.) và Laravel Sanctum (quản lý API token, authentication session).

<a name="installation"></a>
## Cài đặt

Để bắt đầu, cài đặt Fortify sử dụng Composer package manager:

```shell
composer require laravel/fortify
```

Tiếp theo, publish các tài nguyên của Fortify sử dụng command `fortify:install` của Artisan:

```shell
php artisan fortify:install
```

Command này sẽ publish các actions của Fortify vào thư mục `app/Actions` của bạn, sẽ được tạo nếu nó không tồn tại. Ngoài ra, `FortifyServiceProvider`, file cấu hình, và tất cả các database migrations cần thiết sẽ được publish.

Tiếp theo, bạn nên migrate database của bạn:

```shell
php artisan migrate
```

<a name="fortify-features"></a>
### Tính năng Fortify

File cấu hình `fortify` chứa một mảng cấu hình `features`. Mảng này định nghĩa các backend routes / tính năng mà Fortify sẽ expose theo mặc định. Chúng tôi khuyến nghị bạn chỉ bật các tính năng sau, là các tính năng authentication cơ bản được cung cấp bởi hầu hết các ứng dụng Laravel:

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

<a name="disabling-views"></a>
### Vô hiệu hóa Views

Theo mặc định, Fortify định nghĩa các routes dự định trả về views, chẳng hạn như màn hình login hoặc màn hình registration. Tuy nhiên, nếu bạn đang xây dựng một single-page application được điều khiển bởi JavaScript, bạn có thể không cần các routes này. Vì lý do đó, bạn có thể vô hiệu hóa hoàn toàn các routes này bằng cách đặt giá trị cấu hình `views` trong file cấu hình `config/fortify.php` của ứng dụng của bạn thành `false`:

```php
'views' => false,
```

<a name="disabling-views-and-password-reset"></a>
#### Vô hiệu hóa Views và Password Reset

Nếu bạn chọn vô hiệu hóa các views của Fortify và bạn sẽ triển khai các tính năng reset password cho ứng dụng của mình, bạn vẫn nên định nghĩa một route có tên `password.reset` chịu trách nhiệm hiển thị view "reset password" của ứng dụng của bạn. Điều này là cần thiết vì notification `Illuminate\Auth\Notifications\ResetPassword` của Laravel sẽ tạo URL reset password thông qua route có tên `password.reset`.

<a name="authentication"></a>
## Authentication

Để bắt đầu, chúng ta cần hướng dẫn Fortify cách trả về view "login" của chúng ta. Hãy nhớ, Fortify là một thư viện authentication headless. Nếu bạn muốn một triển khai frontend của các tính năng authentication của Laravel đã hoàn thành cho bạn, bạn nên sử dụng một [application starter kit](/docs/{{version}}/starter-kits).

Tất cả logic render view authentication có thể được tùy chỉnh sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn. Fortify sẽ lo việc định nghĩa route `/login` trả về view này:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::loginView(function () {
        return view('auth.login');
    });

    // ...
}
```

Template login của bạn nên bao gồm một form thực hiện POST request đến `/login`. Endpoint `/login` mong đợi một string `email` / `username` và một `password`. Tên của trường email / username nên khớp với giá trị `username` trong file cấu hình `config/fortify.php`. Ngoài ra, một trường boolean `remember` có thể được cung cấp để chỉ định rằng người dùng muốn sử dụng tính năng "remember me" được cung cấp bởi Laravel.

Nếu nỗ lực login thành công, Fortify sẽ redirect bạn đến URI được cấu hình thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu request login là một XHR request, phản hồi HTTP 200 sẽ được trả về.

Nếu request không thành công, người dùng sẽ được redirect trở lại màn hình login và các lỗi validation sẽ có sẵn cho bạn thông qua biến template Blade `$errors` được chia sẻ [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Hoặc, trong trường hợp một XHR request, các lỗi validation sẽ được trả về với phản hồi HTTP 422.

<a name="customizing-user-authentication"></a>
### Tùy chỉnh User Authentication

Fortify sẽ tự động truy xuất và xác thực người dùng dựa trên thông tin xác thực được cung cấp và authentication guard được cấu hình cho ứng dụng của bạn. Tuy nhiên, đôi khi bạn có thể muốn có tùy chỉnh đầy đủ về cách thông tin xác thực login được xác thực và người dùng được truy xuất. May mắn thay, Fortify cho phép bạn dễ dàng thực hiện điều này sử dụng phương thức `Fortify::authenticateUsing`.

Phương thức này chấp nhận một closure nhận incoming HTTP request. Closure chịu trách nhiệm xác thực thông tin xác thực login được gắn vào request và trả về instance người dùng liên quan. Nếu thông tin xác thực không hợp lệ hoặc không tìm thấy người dùng, `null` hoặc `false` nên được trả về bởi closure. Thông thường, phương thức này nên được gọi từ phương thức `boot` của `FortifyServiceProvider` của bạn:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::authenticateUsing(function (Request $request) {
        $user = User::where('email', $request->email)->first();

        if ($user &&
            Hash::check($request->password, $user->password)) {
            return $user;
        }
    });

    // ...
}
```

<a name="authentication-guard"></a>
#### Authentication Guard

Bạn có thể tùy chỉnh authentication guard được sử dụng bởi Fortify trong file cấu hình `fortify` của ứng dụng của bạn. Tuy nhiên, bạn nên đảm bảo rằng guard được cấu hình là một triển khai của `Illuminate\Contracts\Auth\StatefulGuard`. Nếu bạn đang cố gắng sử dụng Laravel Fortify để xác thực một SPA, bạn nên sử dụng `web` guard mặc định của Laravel kết hợp với [Laravel Sanctum](https://laravel.com/docs/sanctum).

<a name="customizing-the-authentication-pipeline"></a>
### Tùy chỉnh Authentication Pipeline

Laravel Fortify xác thực các request login thông qua một pipeline của các classes có thể gọi. Nếu bạn muốn, bạn có thể định nghĩa một pipeline tùy chỉnh của các classes mà các request login nên được pipe qua. Mỗi class nên có một phương thức `__invoke` nhận instance `Illuminate\Http\Request` đến và, giống như [middleware](/docs/{{version}}/middleware), một biến `$next` được gọi để chuyển request đến class tiếp theo trong pipeline.

Để định nghĩa pipeline tùy chỉnh của bạn, bạn có thể sử dụng phương thức `Fortify::authenticateThrough`. Phương thức này chấp nhận một closure nên trả về mảng các classes để pipe request login qua. Thông thường, phương thức này nên được gọi từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn.

Ví dụ dưới đây chứa định nghĩa pipeline mặc định mà bạn có thể sử dụng làm điểm bắt đầu khi thực hiện các sửa đổi của riêng bạn:

```php
use Laravel\Fortify\Actions\AttemptToAuthenticate;
use Laravel\Fortify\Actions\CanonicalizeUsername;
use Laravel\Fortify\Actions\EnsureLoginIsNotThrottled;
use Laravel\Fortify\Actions\PrepareAuthenticatedSession;
use Laravel\Fortify\Actions\RedirectIfTwoFactorAuthenticatable;
use Laravel\Fortify\Features;
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            config('fortify.lowercase_usernames') ? CanonicalizeUsername::class : null,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

#### Authentication Throttling

Theo mặc định, Fortify sẽ throttle các nỗ lực xác thực sử dụng middleware `EnsureLoginIsNotThrottled`. Middleware này throttle các nỗ lực là duy nhất cho một kết hợp username và địa chỉ IP.

Một số ứng dụng có thể yêu cầu một cách tiếp cận khác để throttle các nỗ lực xác thực, chẳng hạn như throttle theo địa chỉ IP một mình. Do đó, Fortify cho phép bạn chỉ định [rate limiter](/docs/{{version}}/routing#rate-limiting) của riêng mình thông qua tùy chọn cấu hình `fortify.limiters.login`. Tất nhiên, tùy chọn cấu hình này nằm trong file cấu hình `config/fortify.php` của ứng dụng của bạn.

> [!NOTE]
> Sử dụng sự kết hợp của throttling, [two-factor authentication](/docs/{{version}}/fortify#two-factor-authentication), và một web application firewall (WAF) bên ngoài sẽ cung cấp sự phòng thủ mạnh mẽ nhất cho người dùng ứng dụng hợp pháp của bạn.

<a name="customizing-authentication-redirects"></a>
### Tùy chỉnh Redirects

Nếu nỗ lực login thành công, Fortify sẽ redirect bạn đến URI được cấu hình thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu request login là một XHR request, phản hồi HTTP 200 sẽ được trả về. Sau khi người dùng đăng xuất khỏi ứng dụng, người dùng sẽ được redirect đến URI `/`.

Nếu bạn cần tùy chỉnh nâng cao của hành vi này, bạn có thể bind các triển khai của các contracts `LoginResponse` và `LogoutResponse` vào [service container](/docs/{{version}}/container) của Laravel. Thông thường, điều này nên được thực hiện trong phương thức `register` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Contracts\LogoutResponse;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->instance(LogoutResponse::class, new class implements LogoutResponse {
        public function toResponse($request)
        {
            return redirect('/');
        }
    });
}
```

<a name="two-factor-authentication"></a>
## Two-Factor Authentication

Khi tính năng two-factor authentication của Fortify được bật, người dùng được yêu cầu nhập một token số sáu chữ số trong quá trình xác thực. Token này được tạo sử dụng một time-based one-time password (TOTP) có thể được truy xuất từ bất kỳ ứng dụng authentication mobile tương thích TOTP nào như Google Authenticator.

Trước khi bắt đầu, bạn nên trước tiên đảm bảo rằng model `App\Models\User` của ứng dụng của bạn sử dụng trait `Laravel\Fortify\TwoFactorAuthenticatable`:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
```

Tiếp theo, bạn nên xây dựng một màn hình trong ứng dụng của mình nơi người dùng có thể quản lý cài đặt two-factor authentication của họ. Màn hình này nên cho phép người dùng bật và tắt two-factor authentication, cũng như tái tạo các recovery codes two-factor authentication của họ.

> Theo mặc định, mảng `features` của file cấu hình `fortify` hướng dẫn cài đặt two-factor authentication của Fortify yêu cầu xác nhận password trước khi sửa đổi. Do đó, ứng dụng của bạn nên triển khai tính năng [password confirmation](#password-confirmation) của Fortify trước khi tiếp tục.

<a name="enabling-two-factor-authentication"></a>
### Bật Two-Factor Authentication

Để bắt đầu bật two-factor authentication, ứng dụng của bạn nên thực hiện POST request đến endpoint `/user/two-factor-authentication` được định nghĩa bởi Fortify. Nếu request thành công, người dùng sẽ được redirect trở lại URL trước đó và biến session `status` sẽ được đặt thành `two-factor-authentication-enabled`. Bạn có thể phát hiện biến session `status` này trong các template của bạn để hiển thị thông báo thành công thích hợp. Nếu request là một XHR request, phản hồi HTTP `200` sẽ được trả về.

Sau khi chọn bật two-factor authentication, người dùng vẫn phải "xác nhận" cấu hình two-factor authentication của họ bằng cách cung cấp một mã two-factor authentication hợp lệ. Vì vậy, thông báo "thành công" của bạn nên hướng dẫn người dùng rằng xác nhận two-factor authentication vẫn được yêu cầu:

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        Please finish configuring two-factor authentication below.
    </div>
@endif
```

Tiếp theo, bạn nên hiển thị mã QR two-factor authentication để người dùng quét vào ứng dụng authenticator của họ. Nếu bạn đang sử dụng Blade để render frontend của ứng dụng, bạn có thể truy xuất mã QR SVG sử dụng phương thức `twoFactorQrCodeSvg` có sẵn trên instance người dùng:

```php
$request->user()->twoFactorQrCodeSvg();
```

Nếu bạn đang xây dựng một frontend được điều khiển bởi JavaScript, bạn có thể thực hiện XHR GET request đến endpoint `/user/two-factor-qr-code` để truy xuất mã QR two-factor authentication của người dùng. Endpoint này sẽ trả về một đối tượng JSON chứa một khóa `svg`.

<a name="confirming-two-factor-authentication"></a>
#### Xác nhận Two-Factor Authentication

Ngoài việc hiển thị mã QR two-factor authentication của người dùng, bạn nên cung cấp một đầu vào văn bản nơi người dùng có thể cung cấp một mã xác thực hợp lệ để "xác nhận" cấu hình two-factor authentication của họ. Mã này nên được cung cấp cho ứng dụng Laravel thông qua POST request đến endpoint `/user/confirmed-two-factor-authentication` được định nghĩa bởi Fortify.

Nếu request thành công, người dùng sẽ được redirect trở lại URL trước đó và biến session `status` sẽ được đặt thành `two-factor-authentication-confirmed`:

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        Two-factor authentication confirmed and enabled successfully.
    </div>
@endif
```

Nếu request đến endpoint xác nhận two-factor authentication được thực hiện thông qua một XHR request, phản hồi HTTP `200` sẽ được trả về.

<a name="displaying-the-recovery-codes"></a>
#### Hiển thị Recovery Codes

Bạn cũng nên hiển thị các recovery codes two-factor của người dùng. Các recovery codes này cho phép người dùng xác thực nếu họ mất quyền truy cập vào thiết bị di động của họ. Nếu bạn đang sử dụng Blade để render frontend của ứng dụng, bạn có thể truy cập các recovery codes thông qua instance người dùng được xác thực:

```php
(array) $request->user()->recoveryCodes()
```

Nếu bạn đang xây dựng một frontend được điều khiển bởi JavaScript, bạn có thể thực hiện XHR GET request đến endpoint `/user/two-factor-recovery-codes`. Endpoint này sẽ trả về một mảng JSON chứa các recovery codes của người dùng.

Để tái tạo các recovery codes của người dùng, ứng dụng của bạn nên thực hiện POST request đến endpoint `/user/two-factor-recovery-codes`.

<a name="authenticating-with-two-factor-authentication"></a>
### Xác thực với Two-Factor Authentication

Trong quá trình xác thực, Fortify sẽ tự động redirect người dùng đến màn hình challenge two-factor authentication của ứng dụng của bạn. Tuy nhiên, nếu ứng dụng của bạn đang thực hiện XHR login request, phản hồi JSON được trả về sau một nỗ lực xác thực thành công sẽ chứa một đối tượng JSON có một thuộc tính boolean `two_factor`. Bạn nên kiểm tra giá trị này để biết liệu bạn có nên redirect đến màn hình challenge two-factor authentication của ứng dụng của mình hay không.

Để bắt đầu triển khai chức năng two-factor authentication, chúng ta cần hướng dẫn Fortify cách trả về view challenge two-factor authentication của chúng ta. Tất cả logic render view authentication của Fortify có thể được tùy chỉnh sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

Fortify sẽ lo việc định nghĩa route `/two-factor-challenge` trả về view này. Template `two-factor-challenge` của bạn nên bao gồm một form thực hiện POST request đến endpoint `/two-factor-challenge`. Action `/two-factor-challenge` mong đợi một trường `code` chứa một token TOTP hợp lệ hoặc một trường `recovery_code` chứa một trong các recovery codes của người dùng.

Nếu nỗ lực login thành công, Fortify sẽ redirect người dùng đến URI được cấu hình thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu request login là một XHR request, phản hồi HTTP 204 sẽ được trả về.

Nếu request không thành công, người dùng sẽ được redirect trở lại màn hình challenge two-factor và các lỗi validation sẽ có sẵn cho bạn thông qua biến template Blade `$errors` được chia sẻ [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Hoặc, trong trường hợp một XHR request, các lỗi validation sẽ được trả về với phản hồi HTTP 422.

<a name="disabling-two-factor-authentication"></a>
### Tắt Two-Factor Authentication

Để tắt two-factor authentication, ứng dụng của bạn nên thực hiện DELETE request đến endpoint `/user/two-factor-authentication`. Hãy nhớ, các endpoint two-factor authentication của Fortify yêu cầu [password confirmation](#password-confirmation) trước khi được gọi.

<a name="passkeys"></a>
## Passkeys

Fortify hỗ trợ xác thực passkey sử dụng WebAuthn. Passkeys cho phép người dùng xác thực mà không cần mật khẩu sử dụng các authenticator nền tảng như Face ID, Touch ID, Windows Hello, hoặc các khóa bảo mật phần cứng.

<a name="enabling-passkeys"></a>
### Bật Passkeys

Để bắt đầu, đảm bảo tính năng `passkeys` được bật trong file cấu hình `fortify` của ứng dụng của bạn:

```php
use Laravel\Fortify\Features;

'features' => [
    // ...
    Features::passkeys([
        'confirmPassword' => true,
    ]),
],
```

Tùy chọn `confirmPassword` xác định liệu Fortify có yêu cầu [password confirmation](#password-confirmation) trước khi passkeys có thể được đăng ký hoặc xóa hay không.

Tiếp theo, đảm bảo model `App\Models\User` của ứng dụng của bạn implement `Laravel\Fortify\Contracts\PasskeyUser` và sử dụng trait `Laravel\Fortify\PasskeyAuthenticatable`:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\Contracts\PasskeyUser;
use Laravel\Fortify\PasskeyAuthenticatable;

class User extends Authenticatable implements PasskeyUser
{
    use Notifiable, PasskeyAuthenticatable;
}
```

Các tùy chọn cấu hình passkeys của Fortify có thể được tùy chỉnh sử dụng mảng cấu hình `passkeys` trong file `config/fortify.php` của ứng dụng của bạn:

```php
'passkeys' => [
    'relying_party_id' => parse_url(config('app.url'), PHP_URL_HOST),
    'allowed_origins' => [config('app.url')],
    'user_handle_secret' => config('app.key'),
    'timeout' => 60000,
],
```

> [!NOTE]
> Fortify wraps Composer package `laravel/passkeys` và cấu hình nó cho bạn. Nếu bạn đang sử dụng tính năng passkeys của Fortify, bạn nên cấu hình passkeys sử dụng file `config/fortify.php` của ứng dụng của bạn. Bạn không cần publish file cấu hình `laravel/passkeys`, và bất kỳ giá trị nào được định nghĩa ở đó sẽ bị ghi đè bởi Fortify.

`relying_party_id` nên khớp với domain của ứng dụng của bạn. Mảng `allowed_origins` liệt kê các nguồn gốc trình duyệt có thể hoàn thành đăng ký và xác thực passkey. `user_handle_secret` được sử dụng để dẫn xuất các định danh người dùng mờ, đảm bảo cùng một người dùng được nhận ra qua các đăng ký passkey. Tùy chọn `timeout` kiểm soát bao lâu các hoạt động đăng ký và xác thực passkey có thể vẫn hoạt động.

Fortify áp dụng một rate limiter passkey chuyên dụng cho các routes login, xác nhận, và đăng ký passkey của nó. Nếu cần, bạn có thể tùy chỉnh nó sử dụng tùy chọn cấu hình `fortify.limiters.passkeys` và định nghĩa `RateLimiter::for(...)` tương ứng.

<a name="passkeys-javascript-client"></a>
### JavaScript Client

Nếu bạn đang xây dựng một frontend tùy chỉnh, bao gồm một ứng dụng Blade với các script phía trình duyệt, bạn có thể sử dụng package chính thức [`@laravel/passkeys`](https://www.npmjs.com/package/@laravel/passkeys). Package này xử lý các nghi thức WebAuthn trình duyệt và gửi các request đến các endpoint passkey của Fortify.

Cài đặt package qua npm:

```shell
npm install @laravel/passkeys
```

Sau đó, bạn có thể khởi tạo đăng ký và xác thực passkey từ frontend của bạn:

```js
import { Passkeys } from "@laravel/passkeys";

await Passkeys.register({ name: "MacBook Pro" });
await Passkeys.verify();
```

Nếu ứng dụng của bạn sử dụng các URI endpoint passkey tùy chỉnh, bạn có thể ghi đè các routes trên cơ sở mỗi lần gọi:

```js
await Passkeys.verify({
    routes: {
        options: "/passkeys/confirm/options",
        submit: "/passkeys/confirm",
    },
});

await Passkeys.register({
    name: "MacBook Pro",
    routes: {
        options: "/user/passkeys/options",
        submit: "/user/passkeys",
    },
});
```

Package cũng cung cấp các helpers React, Vue, và Svelte thông qua `@laravel/passkeys/react`, `@laravel/passkeys/vue`, và `@laravel/passkeys/svelte`.

<a name="authenticating-with-passkeys"></a>
### Xác thực với Passkeys

Để xác thực một người dùng với một passkey, ứng dụng của bạn nên trước tiên thực hiện GET request đến endpoint `/passkeys/login/options`. Endpoint này trả về các tùy chọn challenge WebAuthn mà frontend của bạn nên chuyển đến `navigator.credentials.get(...)`.

Sau khi trình duyệt trả về một credential, ứng dụng của bạn nên thực hiện POST request đến `/passkeys/login` với payload credential. Bạn cũng có thể bao gồm một trường boolean `remember`.

Nếu request thành công, Fortify sẽ đăng nhập người dùng vào guard được cấu hình và trả về một trong hai:

<div class="content-list" markdown="1">

- Một phản hồi redirect đến đích dự định của bạn cho các request tiêu chuẩn.
- Một phản hồi HTTP `200` chứa một payload JSON với một khóa `redirect` cho các request XHR.

</div>

<a name="confirming-password-with-passkeys"></a>
### Xác nhận Password với Passkeys

Đối với các session được xác thực, Fortify cung cấp các endpoint xác nhận passkey thỏa mãn yêu cầu xác nhận password của Laravel cho session hiện tại.

Để xác nhận với một passkey, ứng dụng của bạn nên trước tiên thực hiện GET request đến `/passkeys/confirm/options`. Endpoint này trả về các tùy chọn challenge WebAuthn mà frontend của bạn nên chuyển đến `navigator.credentials.get(...)`.

Sau khi trình duyệt trả về một credential, ứng dụng của bạn nên thực hiện POST request đến `/passkeys/confirm` với payload credential.

Nếu request thành công, Fortify đánh dấu session hiện tại là đã xác nhận password và trả về một trong hai:

<div class="content-list" markdown="1">

- Một phản hồi redirect đến đích dự định của bạn cho các request tiêu chuẩn.
- Một phản hồi HTTP `200` chứa một payload JSON với một khóa `redirect` cho các request XHR.

</div>

<a name="registering-passkeys"></a>
### Đăng ký Passkeys

Để đăng ký một passkey cho một người dùng được xác thực, ứng dụng của bạn nên trước tiên thực hiện GET request đến `/user/passkeys/options`. Endpoint này trả về các tùy chọn tạo WebAuthn mà frontend của bạn nên chuyển đến `navigator.credentials.create(...)`.

Sau khi trình duyệt trả về một credential, ứng dụng của bạn nên thực hiện POST request đến `/user/passkeys` với một trường `name` và một trường `credential` chứa đối tượng [`PublicKeyCredential`](https://developer.mozilla.org/en-US/docs/Web/API/PublicKeyCredential) được serialize được trả về bởi `navigator.credentials.create(...)`.

Nếu request thành công, Fortify sẽ trả về một trong hai:

<div class="content-list" markdown="1">

- Một phản hồi redirect trở lại với một trạng thái `passkey-registered` trong session cho các request tiêu chuẩn.
- Một phản hồi HTTP `200` với một payload JSON chứa một khóa `status`, cùng với `id` và `name` của passkey mới được đăng ký.

</div>

<a name="deleting-passkeys"></a>
### Xóa Passkeys

Để xóa một passkey, ứng dụng của bạn nên thực hiện DELETE request đến `/user/passkeys/{passkey}`.

Nếu request thành công, Fortify sẽ trả về một trong hai:

<div class="content-list" markdown="1">

- Một phản hồi redirect trở lại với một trạng thái `passkey-deleted` trong session cho các request tiêu chuẩn.
- Một phản hồi HTTP `200` với một payload JSON chứa một khóa `status` cho các request XHR.

</div>

<a name="registration"></a>
## Đăng ký

Để bắt đầu triển khai chức năng đăng ký của ứng dụng của chúng ta, chúng ta cần hướng dẫn Fortify cách trả về view "register" của chúng ta. Hãy nhớ, Fortify là một thư viện authentication headless. Nếu bạn muốn một triển khai frontend của các tính năng authentication của Laravel đã hoàn thành cho bạn, bạn nên sử dụng một [application starter kit](/docs/{{version}}/starter-kits).

Tất cả logic render view của Fortify có thể được tùy chỉnh sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::registerView(function () {
        return view('auth.register');
    });

    // ...
}
```

Fortify sẽ lo việc định nghĩa route `/register` trả về view này. Template `register` của bạn nên bao gồm một form thực hiện POST request đến endpoint `/register` được định nghĩa bởi Fortify.

Endpoint `/register` mong đợi một trường string `name`, địa chỉ email / username string, `password`, và các trường `password_confirmation`. Tên của trường email / username nên khớp với giá trị cấu hình `username` được định nghĩa trong file cấu hình `fortify` của ứng dụng của bạn.

Nếu nỗ lực đăng ký thành công, Fortify sẽ redirect người dùng đến URI được cấu hình thông qua tùy chọn cấu hình `home` trong file cấu hình `fortify` của ứng dụng của bạn. Nếu request là một XHR request, phản hồi HTTP 201 sẽ được trả về.

Nếu request không thành công, người dùng sẽ được redirect trở lại màn hình đăng ký và các lỗi validation sẽ có sẵn cho bạn thông qua biến template Blade `$errors` được chia sẻ [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Hoặc, trong trường hợp một XHR request, các lỗi validation sẽ được trả về với phản hồi HTTP 422.

<a name="customizing-registration"></a>
### Tùy chỉnh Đăng ký

Quy trình xác thực và tạo người dùng có thể được tùy chỉnh bằng cách sửa đổi action `App\Actions\Fortify\CreateNewUser` được tạo khi bạn cài đặt Laravel Fortify.

<a name="password-reset"></a>
## Password Reset

<a name="requesting-a-password-reset-link"></a>
### Yêu cầu Link Reset Password

Để bắt đầu triển khai chức năng reset password của ứng dụng của chúng ta, chúng ta cần hướng dẫn Fortify cách trả về view "forgot password" của chúng ta. Hãy nhớ, Fortify là một thư viện authentication headless. Nếu bạn muốn một triển khai frontend của các tính năng authentication của Laravel đã hoàn thành cho bạn, bạn nên sử dụng một [application starter kit](/docs/{{version}}/starter-kits).

Tất cả logic render view của Fortify có thể được tùy chỉnh sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

Fortify sẽ lo việc định nghĩa endpoint `/forgot-password` trả về view này. Template `forgot-password` của bạn nên bao gồm một form thực hiện POST request đến endpoint `/forgot-password`.

Endpoint `/forgot-password` mong đợi một trường string `email`. Tên của trường này / cột database nên khớp với giá trị cấu hình `email` trong file cấu hình `fortify` của ứng dụng của bạn.

<a name="handling-the-password-reset-link-request-response"></a>
#### Xử lý Phản hồi Yêu cầu Link Reset Password

Nếu yêu cầu link reset password thành công, Fortify sẽ redirect người dùng trở lại endpoint `/forgot-password` và gửi một email đến người dùng với một link bảo mật mà họ có thể sử dụng để reset password của họ. Nếu request là một XHR request, phản hồi HTTP 200 sẽ được trả về.

Sau khi được redirect trở lại endpoint `/forgot-password` sau một request thành công, biến session `status` có thể được sử dụng để hiển thị trạng thái của nỗ lực yêu cầu link reset password.

Giá trị của biến session `$status` sẽ khớp với một trong các chuỗi dịch được định nghĩa trong [file ngôn ngữ](/docs/{{version}}/localization) `passwords` của ứng dụng của bạn. Nếu bạn muốn tùy chỉnh giá trị này và chưa publish các file ngôn ngữ của Laravel, bạn có thể làm như vậy thông qua command Artisan `lang:publish`:

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Nếu request không thành công, người dùng sẽ được redirect trở lại màn hình yêu cầu link reset password và các lỗi validation sẽ có sẵn cho bạn thông qua biến template Blade `$errors` được chia sẻ [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Hoặc, trong trường hợp một XHR request, các lỗi validation sẽ được trả về với phản hồi HTTP 422.

<a name="resetting-the-password"></a>
### Reset Password

Để hoàn thành triển khai chức năng reset password của ứng dụng của chúng ta, chúng ta cần hướng dẫn Fortify cách trả về view "reset password" của chúng ta.

Tất cả logic render view của Fortify có thể được tùy chỉnh sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::resetPasswordView(function (Request $request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

Fortify sẽ lo việc định nghĩa route để hiển thị view này. Template `reset-password` của bạn nên bao gồm một form thực hiện POST request đến `/reset-password`.

Endpoint `/reset-password` mong đợi một trường string `email`, một trường `password`, một trường `password_confirmation`, và một trường ẩn có tên `token` chứa giá trị của `request()->route('token')`. Tên của trường "email" / cột database nên khớp với giá trị cấu hình `email` được định nghĩa trong file cấu hình `fortify` của ứng dụng của bạn.

<a name="handling-the-password-reset-response"></a>
#### Xử lý Phản hồi Reset Password

Nếu yêu cầu reset password thành công, Fortify sẽ redirect trở lại route `/login` để người dùng có thể đăng nhập với password mới của họ. Ngoài ra, một biến session `status` sẽ được đặt để bạn có thể hiển thị trạng thái thành công của reset trên màn hình login của bạn:

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

Nếu request là một XHR request, phản hồi HTTP 200 sẽ được trả về.

Nếu request không thành công, người dùng sẽ được redirect trở lại màn hình reset password và các lỗi validation sẽ có sẵn cho bạn thông qua biến template Blade `$errors` được chia sẻ [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Hoặc, trong trường hợp một XHR request, các lỗi validation sẽ được trả về với phản hồi HTTP 422.

<a name="customizing-password-resets"></a>
### Tùy chỉnh Password Resets

Quy trình reset password có thể được tùy chỉnh bằng cách sửa đổi action `App\Actions\ResetUserPassword` được tạo khi bạn cài đặt Laravel Fortify.

<a name="email-verification"></a>
## Email Verification

Sau khi đăng ký, bạn có thể muốn người dùng xác thực địa chỉ email của họ trước khi họ tiếp tục truy cập ứng dụng của bạn. Để bắt đầu, đảm bảo tính năng `emailVerification` được bật trong mảng `features` của file cấu hình `fortify` của bạn. Tiếp theo, bạn nên đảm bảo rằng class `App\Models\User` của bạn implement interface `Illuminate\Contracts\Auth\MustVerifyEmail`.

Sau khi hai bước thiết lập này đã hoàn thành, người dùng mới đăng ký sẽ nhận được một email nhắc họ xác thực quyền sở hữu địa chỉ email của họ. Tuy nhiên, chúng ta cần thông báo cho Fortify cách hiển thị màn hình xác thực email thông báo cho người dùng rằng họ cần đi click link xác thực trong email.

Tất cả logic render view của Fortify có thể được tùy chỉnh sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::verifyEmailView(function () {
        return view('auth.verify-email');
    });

    // ...
}
```

Fortify sẽ lo việc định nghĩa route hiển thị view này khi người dùng được redirect đến endpoint `/email/verify` bởi middleware `verified` tích hợp của Laravel.

Template `verify-email` của bạn nên bao gồm một thông báo thông tin hướng dẫn người dùng click link xác thực email đã được gửi đến địa chỉ email của họ.

<a name="resending-email-verification-links"></a>
#### Gửi lại Link Xác thực Email

Nếu bạn muốn, bạn có thể thêm một nút vào template `verify-email` của ứng dụng của bạn kích hoạt POST request đến endpoint `/email/verification-notification`. Khi endpoint này nhận được một request, một link xác thực email mới sẽ được gửi đến người dùng qua email, cho phép người dùng nhận được link xác thực mới nếu link trước đó bị xóa hoặc mất vô tình.

Nếu yêu cầu gửi lại email link xác thực thành công, Fortify sẽ redirect người dùng trở lại endpoint `/email/verify` với một biến session `status`, cho phép bạn hiển thị một thông báo thông tin cho người dùng thông báo cho họ rằng hoạt động thành công. Nếu request là một XHR request, phản hồi HTTP 202 sẽ được trả về:

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        A new email verification link has been emailed to you!
    </div>
@endif
```

<a name="protecting-routes"></a>
### Bảo vệ Routes

Để chỉ định rằng một route hoặc nhóm routes yêu cầu người dùng đã xác thực địa chỉ email của họ, bạn nên gắn middleware `verified` tích hợp của Laravel vào route. Alias middleware `verified` được đăng ký tự động bởi Laravel và đóng vai trò là alias cho middleware `Illuminate\Auth\Middleware\EnsureEmailIsVerified`:

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

<a name="password-confirmation"></a>
## Password Confirmation

Trong khi xây dựng ứng dụng của bạn, đôi khi bạn có thể có các hành động nên yêu cầu người dùng xác nhận password của họ trước khi hành động được thực hiện. Thông thường, các routes này được bảo vệ bởi middleware `password.confirm` tích hợp của Laravel.

Để bắt đầu triển khai chức năng xác nhận password, chúng ta cần hướng dẫn Fortify cách trả về view "password confirmation" của ứng dụng của chúng ta. Hãy nhớ, Fortify là một thư viện authentication headless. Nếu bạn muốn một triển khai frontend của các tính năng authentication của Laravel đã hoàn thành cho bạn, bạn nên sử dụng một [application starter kit](/docs/{{version}}/starter-kits).

Tất cả logic render view của Fortify có thể được tùy chỉnh sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Fortify\Fortify`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\FortifyServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify sẽ lo việc định nghĩa endpoint `/user/confirm-password` trả về view này. Template `confirm-password` của bạn nên bao gồm một form thực hiện POST request đến endpoint `/user/confirm-password`. Endpoint `/user/confirm-password` mong đợi một trường `password` chứa password hiện tại của người dùng.

Nếu password khớp với password hiện tại của người dùng, Fortify sẽ redirect người dùng đến route họ đang cố gắng truy cập. Nếu request là một XHR request, phản hồi HTTP 201 sẽ được trả về.

Nếu request không thành công, người dùng sẽ được redirect trở lại màn hình xác nhận password và các lỗi validation sẽ có sẵn cho bạn thông qua biến template Blade `$errors` được chia sẻ. Hoặc, trong trường hợp một XHR request, các lỗi validation sẽ được trả về với phản hồi HTTP 422.

# Resetting Passwords

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
    - [Model Preparation](#model-preparation)
    - [Configuring Trusted Hosts](#configuring-trusted-hosts)
- [Routing](#routing)
    - [Requesting the Password Reset Link](#requesting-the-password-reset-link)
    - [Resetting the Password](#resetting-the-password)
- [Deleting Expired Tokens](#deleting-expired-tokens)
- [Customization](#password-customization)

<a name="introduction"></a>
## Introduction

Hầu hết các ứng dụng web cung cấp cách để người dùng reset password đã quên của họ. Thay vì buộc bạn phải thực hiện lại điều này thủ công cho mỗi ứng dụng bạn tạo, Laravel cung cấp các dịch vụ tiện lợi để gửi link reset password và reset password an toàn.

> [!NOTE]
> Muốn bắt đầu nhanh? Cài đặt một [Laravel application starter kit](/docs/{{version}}/starter-kits) trong một ứng dụng Laravel mới. Các starter kit của Laravel sẽ lo việc scaffolding toàn bộ hệ thống authentication của bạn, bao gồm cả reset password đã quên.

<a name="configuration"></a>
### Configuration

File cấu hình reset password của ứng dụng của bạn được lưu trữ tại `config/auth.php`. Hãy chắc chắn xem xét các tùy chọn có sẵn cho bạn trong file này. Theo mặc định, Laravel được cấu hình để sử dụng driver reset password `database`.

Tùy chọn cấu hình `driver` của reset password định nghĩa nơi dữ liệu reset password sẽ được lưu trữ. Laravel bao gồm hai driver:

<div class="content-list" markdown="1">

- `database` - dữ liệu reset password được lưu trữ trong một cơ sở dữ liệu quan hệ.
- `cache` - dữ liệu reset password được lưu trữ trong một trong các cache-based stores của bạn.

</div>

<a name="driver-prerequisites"></a>
### Driver Prerequisites

<a name="database"></a>
#### Database

Khi sử dụng driver `database` mặc định, một table phải được tạo để lưu trữ các token reset password của ứng dụng của bạn. Thông thường, điều này được bao gồm trong migration database mặc định của Laravel `0001_01_01_000000_create_users_table.php`.

<a name="cache"></a>
#### Cache

Cũng có một driver cache có sẵn để xử lý reset password, không yêu cầu một table database chuyên dụng. Các entry được key bởi địa chỉ email của người dùng, vì vậy hãy đảm bảo bạn không sử dụng địa chỉ email làm cache key ở nơi khác trong ứng dụng của bạn:

```php
'passwords' => [
    'users' => [
        'driver' => 'cache',
        'provider' => 'users',
        'store' => 'passwords', // Optional...
        'expire' => 60,
        'throttle' => 60,
    ],
],
```

Để ngăn chặn một lệnh gọi đến `artisan cache:clear` xóa dữ liệu reset password của bạn, bạn có thể tùy chọn chỉ định một cache store riêng biệt với key cấu hình `store`. Giá trị nên tương ứng với một store được cấu hình trong giá trị cấu hình `config/cache.php` của bạn.

<a name="model-preparation"></a>
### Model Preparation

Trước khi sử dụng các tính năng reset password của Laravel, model `App\Models\User` của ứng dụng của bạn phải sử dụng trait `Illuminate\Notifications\Notifiable`. Thông thường, trait này đã được bao gồm trong model `App\Models\User` mặc định được tạo với các ứng dụng Laravel mới.

Tiếp theo, xác minh rằng model `App\Models\User` của bạn implements contract `Illuminate\Contracts\Auth\CanResetPassword`. Model `App\Models\User` được bao gồm với framework đã implement interface này, và sử dụng trait `Illuminate\Auth\Passwords\CanResetPassword` để bao gồm các methods cần thiết để implement interface.

<a name="configuring-trusted-hosts"></a>
### Configuring Trusted Hosts

Theo mặc định, Laravel sẽ phản hồi tất cả các request nó nhận được bất kể nội dung của header `Host` của HTTP request. Ngoài ra, giá trị của header `Host` sẽ được sử dụng khi tạo các URL tuyệt đối đến ứng dụng của bạn trong một web request.

Thông thường, bạn nên cấu hình web server của bạn, chẳng hạn như Nginx hoặc Apache, để chỉ gửi request đến ứng dụng của bạn khớp với một hostname nhất định. Tuy nhiên, nếu bạn không có khả năng tùy chỉnh web server của bạn trực tiếp và cần hướng dẫn Laravel chỉ phản hồi với các hostname nhất định, bạn có thể làm như vậy bằng cách sử dụng method middleware `trustHosts` trong file `bootstrap/app.php` của ứng dụng của bạn. Điều này đặc biệt quan trọng khi ứng dụng của bạn cung cấp tính năng reset password.

Để tìm hiểu thêm về method middleware này, vui lòng tham khảo [tài liệu middleware TrustHosts](/docs/{{version}}/requests#configuring-trusted-hosts).

<a name="routing"></a>
## Routing

Để implement đúng hỗ trợ cho phép người dùng reset password của họ, chúng ta sẽ cần định nghĩa một số routes. Đầu tiên, chúng ta sẽ cần một cặp routes để xử lý việc cho phép người dùng yêu cầu link reset password qua địa chỉ email của họ. Thứ hai, chúng ta sẽ cần một cặp routes để xử lý việc thực sự reset password khi người dùng truy cập link reset password được gửi qua email và hoàn thành form reset password.

<a name="requesting-the-password-reset-link"></a>
### Requesting the Password Reset Link

<a name="the-password-reset-link-request-form"></a>
#### The Password Reset Link Request Form

Đầu tiên, chúng ta sẽ định nghĩa các routes cần thiết để yêu cầu link reset password. Để bắt đầu, chúng ta sẽ định nghĩa một route trả về một view với form yêu cầu link reset password:

```php
Route::get('/forgot-password', function () {
    return view('auth.forgot-password');
})->middleware('guest')->name('password.request');
```

View được trả về bởi route này nên có một form chứa field `email`, sẽ cho phép người dùng yêu cầu link reset password cho một địa chỉ email nhất định.

<a name="password-reset-link-handling-the-form-submission"></a>
#### Handling the Form Submission

Tiếp theo, chúng ta sẽ định nghĩa một route xử lý request gửi form từ view "forgot password". Route này sẽ chịu trách nhiệm xác thực địa chỉ email và gửi request reset password đến người dùng tương ứng:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Password;

Route::post('/forgot-password', function (Request $request) {
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::ResetLinkSent
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
})->middleware('guest')->name('password.email');
```

Trước khi tiếp tục, hãy xem xét route này chi tiết hơn. Đầu tiên, attribute `email` của request được xác thực. Tiếp theo, chúng ta sẽ sử dụng "password broker" tích hợp sẵn của Laravel (thông qua `Password` facade) để gửi link reset password đến người dùng. Password broker sẽ lo việc truy xuất người dùng bằng field đã cho (trong trường hợp này, địa chỉ email) và gửi link reset password đến người dùng thông qua [hệ thống notification](/docs/{{version}}/notifications) tích hợp sẵn của Laravel.

Method `sendResetLink` trả về một "status" slug. Status này có thể được dịch bằng các [helper localization](/docs/{{version}}/localization) của Laravel để hiển thị thông báo thân thiện với người dùng về trạng thái request của họ. Bản dịch của trạng thái reset password được xác định bởi file ngôn ngữ `lang/{lang}/passwords.php` của ứng dụng của bạn. Một entry cho mỗi giá trị có thể của status slug nằm trong file ngôn ngữ `passwords`.

> [!NOTE]
> Theo mặc định, skeleton ứng dụng Laravel không bao gồm thư mục `lang`. Nếu bạn muốn tùy chỉnh các file ngôn ngữ của Laravel, bạn có thể publish chúng thông qua lệnh Artisan `lang:publish`.

Bạn có thể tự hỏi Laravel biết cách truy xuất record người dùng từ cơ sở dữ liệu của ứng dụng của bạn như thế nào khi gọi method `sendResetLink` của `Password` facade. Password broker của Laravel sử dụng "user providers" của hệ thống authentication của bạn để truy xuất các record database. User provider được sử dụng bởi password broker được cấu hình trong mảng cấu hình `passwords` của file cấu hình `config/auth.php` của bạn. Để tìm hiểu thêm về việc viết user providers tùy chỉnh, hãy tham khảo [tài liệu authentication](/docs/{{version}}/authentication#adding-custom-user-providers).

> [!NOTE]
> Khi implement reset password thủ công, bạn được yêu cầu định nghĩa nội dung của các views và routes của chính mình. Nếu bạn muốn scaffolding bao gồm tất cả logic authentication và verification cần thiết, hãy xem [Laravel application starter kits](/docs/{{version}}/starter-kits).

<a name="resetting-the-password"></a>
### Resetting the Password

<a name="the-password-reset-form"></a>
#### The Password Reset Form

Tiếp theo, chúng ta sẽ định nghĩa các routes cần thiết để thực sự reset password khi người dùng nhấp vào link reset password đã được gửi qua email và cung cấp password mới. Đầu tiên, hãy định nghĩa route sẽ hiển thị form reset password được hiển thị khi người dùng nhấp vào link reset password. Route này sẽ nhận một parameter `token` mà chúng ta sẽ sử dụng sau để xác minh request reset password:

```php
Route::get('/reset-password/{token}', function (string $token) {
    return view('auth.reset-password', ['token' => $token]);
})->middleware('guest')->name('password.reset');
```

View được trả về bởi route này nên hiển thị một form chứa field `email`, field `password`, field `password_confirmation`, và field `token` ẩn, nên chứa giá trị của `$token` bí mật được nhận bởi route của chúng ta.

<a name="password-reset-handling-the-form-submission"></a>
#### Handling the Form Submission

Tất nhiên, chúng ta cần định nghĩa một route để thực sự xử lý việc gửi form reset password. Route này sẽ chịu trách nhiệm xác thực request đến và cập nhật password của người dùng trong cơ sở dữ liệu:

```php
use App\Models\User;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Password;
use Illuminate\Support\Str;

Route::post('/reset-password', function (Request $request) {
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function (User $user, string $password) {
            $user->forceFill([
                'password' => Hash::make($password)
            ])->setRememberToken(Str::random(60));

            $user->save();

            event(new PasswordReset($user));
        }
    );

    return $status === Password::PasswordReset
        ? redirect()->route('login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
})->middleware('guest')->name('password.update');
```

Trước khi tiếp tục, hãy xem xét route này chi tiết hơn. Đầu tiên, các attribute `token`, `email`, và `password` của request được xác thực. Tiếp theo, chúng ta sẽ sử dụng "password broker" tích hợp sẵn của Laravel (thông qua `Password` facade) để xác thực credentials của request reset password.

Nếu token, địa chỉ email, và password được đưa cho password broker là hợp lệ, closure được truyền cho method `reset` sẽ được gọi. Trong closure này, nhận instance người dùng và password plain-text được cung cấp cho form reset password, chúng ta có thể cập nhật password của người dùng trong cơ sở dữ liệu.

Method `reset` trả về một "status" slug. Status này có thể được dịch bằng các [helper localization](/docs/{{version}}/localization) của Laravel để hiển thị thông báo thân thiện với người dùng về trạng thái request của họ. Bản dịch của trạng thái reset password được xác định bởi file ngôn ngữ `lang/{lang}/passwords.php` của ứng dụng của bạn. Một entry cho mỗi giá trị có thể của status slug nằm trong file ngôn ngữ `passwords`. Nếu ứng dụng của bạn không chứa thư mục `lang`, bạn có thể tạo nó bằng lệnh Artisan `lang:publish`.

Trước khi tiếp tục, bạn có thể tự hỏi Laravel biết cách truy xuất record người dùng từ cơ sở dữ liệu của ứng dụng của bạn như thế nào khi gọi method `reset` của `Password` facade. Password broker của Laravel sử dụng "user providers" của hệ thống authentication của bạn để truy xuất các record database. User provider được sử dụng bởi password broker được cấu hình trong mảng cấu hình `passwords` của file cấu hình `config/auth.php` của bạn. Để tìm hiểu thêm về việc viết user providers tùy chỉnh, hãy tham khảo [tài liệu authentication](/docs/{{version}}/authentication#adding-custom-user-providers).

<a name="deleting-expired-tokens"></a>
## Deleting Expired Tokens

Nếu bạn đang sử dụng driver `database`, các token reset password đã hết hạn sẽ vẫn còn trong cơ sở dữ liệu của bạn. Tuy nhiên, bạn có thể dễ dàng xóa các record này bằng lệnh Artisan `auth:clear-resets`:

```shell
php artisan auth:clear-resets
```

Nếu bạn muốn tự động hóa quá trình này, hãy xem xét thêm lệnh vào [scheduler](/docs/{{version}}/scheduling) của ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('auth:clear-resets')->everyFifteenMinutes();
```

<a name="password-customization"></a>
## Customization

<a name="reset-link-customization"></a>
#### Reset Link Customization

Bạn có thể tùy chỉnh URL link reset password bằng method `createUrlUsing` được cung cấp bởi class notification `ResetPassword`. Method này chấp nhận một closure nhận instance người dùng đang nhận notification cũng như token link reset password. Thông thường, bạn nên gọi method này từ method `boot` của `AppServiceProvider` của ứng dụng của bạn:

```php
use App\Models\User;
use Illuminate\Auth\Notifications\ResetPassword;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    ResetPassword::createUrlUsing(function (User $user, string $token) {
        return 'https://example.com/reset-password?token='.$token;
    });
}
```

<a name="reset-email-customization"></a>
#### Reset Email Customization

Bạn có thể dễ dàng sửa đổi class notification được sử dụng để gửi link reset password đến người dùng. Để bắt đầu, override method `sendPasswordResetNotification` trên model `App\Models\User` của bạn. Trong method này, bạn có thể gửi notification bằng bất kỳ [class notification](/docs/{{version}}/notifications) nào của riêng bạn. `$token` reset password là argument đầu tiên được nhận bởi method. Bạn có thể sử dụng `$token` này để xây dựng URL reset password theo lựa chọn của bạn và gửi notification của bạn đến người dùng:

```php
use App\Notifications\ResetPasswordNotification;

/**
 * Send a password reset notification to the user.
 *
 * @param  string  $token
 */
public function sendPasswordResetNotification($token): void
{
    $url = 'https://example.com/reset-password?token='.$token;

    $this->notify(new ResetPasswordNotification($url));
}
```

# Email Verification

- [Introduction](#introduction)
    - [Model Preparation](#model-preparation)
    - [Database Preparation](#database-preparation)
- [Routing](#verification-routing)
    - [The Email Verification Notice](#the-email-verification-notice)
    - [The Email Verification Handler](#the-email-verification-handler)
    - [Resending the Verification Email](#resending-the-verification-email)
    - [Protecting Routes](#protecting-routes)
- [Customization](#customization)
- [Events](#events)

<a name="introduction"></a>
## Introduction

Nhiều ứng dụng web yêu cầu người dùng xác minh địa chỉ email của họ trước khi sử dụng ứng dụng. Thay vì buộc bạn phải thực hiện lại tính năng này thủ công cho mỗi ứng dụng bạn tạo, Laravel cung cấp các dịch vụ tích hợp sẵn tiện lợi để gửi và xác minh các request xác minh email.

> [!NOTE]
> Muốn bắt đầu nhanh? Cài đặt một trong các [Laravel application starter kits](/docs/{{version}}/starter-kits) trong một ứng dụng Laravel mới. Các starter kits sẽ lo việc scaffolding toàn bộ hệ thống authentication của bạn, bao gồm hỗ trợ xác minh email.

<a name="model-preparation"></a>
### Model Preparation

Trước khi bắt đầu, xác minh rằng model `App\Models\User` của bạn implements contract `Illuminate\Contracts\Auth\MustVerifyEmail`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    // ...
}
```

Khi interface này đã được thêm vào model của bạn, người dùng mới đăng ký sẽ tự động được gửi một email chứa link xác minh email. Điều này diễn ra liền mạch vì Laravel tự động đăng ký [listener](/docs/{{version}}/events) `Illuminate\Auth\Listeners\SendEmailVerificationNotification` cho event `Illuminate\Auth\Events\Registered`.

Nếu bạn đang implement đăng ký thủ công trong ứng dụng của bạn thay vì sử dụng [starter kit](/docs/{{version}}/starter-kits), bạn nên đảm bảo rằng bạn đang dispatch event `Illuminate\Auth\Events\Registered` sau khi đăng ký của người dùng thành công:

```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

<a name="database-preparation"></a>
### Database Preparation

Tiếp theo, table `users` của bạn phải chứa một column `email_verified_at` để lưu trữ ngày và giờ địa chỉ email của người dùng được xác minh. Thông thường, điều này được bao gồm trong migration database mặc định của Laravel `0001_01_01_000000_create_users_table.php`.

<a name="verification-routing"></a>
## Routing

Để implement xác minh email đúng cách, ba routes sẽ cần được định nghĩa. Đầu tiên, một route sẽ cần được định nghĩa để hiển thị thông báo cho người dùng rằng họ nên nhấp vào link xác minh email trong email xác minh mà Laravel đã gửi cho họ sau khi đăng ký.

Thứ hai, một route sẽ cần được định nghĩa để xử lý các request được tạo khi người dùng nhấp vào link xác minh email trong email.

Thứ ba, một route sẽ cần được định nghĩa để gửi lại link xác minh nếu người dùng vô tình làm mất link xác minh đầu tiên.

<a name="the-email-verification-notice"></a>
### The Email Verification Notice

Như đã đề cập trước đó, một route nên được định nghĩa sẽ trả về một view hướng dẫn người dùng nhấp vào link xác minh email đã được gửi cho họ bởi Laravel sau khi đăng ký. View này sẽ được hiển thị cho người dùng khi họ cố gắng truy cập các phần khác của ứng dụng mà không xác minh địa chỉ email của họ trước. Hãy nhớ rằng, link được tự động gửi cho người dùng miễn là model `App\Models\User` của bạn implements interface `MustVerifyEmail`:

```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');
```

Route trả về thông báo xác minh email nên được đặt tên là `verification.notice`. Điều quan trọng là route được gán tên chính xác này vì middleware `verified` [được bao gồm với Laravel](#protecting-routes) sẽ tự động redirect đến tên route này nếu người dùng chưa xác minh địa chỉ email của họ.

> [!NOTE]
> Khi implement xác minh email thủ công, bạn được yêu cầu định nghĩa nội dung của view thông báo xác minh của chính mình. Nếu bạn muốn scaffolding bao gồm tất cả các views authentication và verification cần thiết, hãy xem [Laravel application starter kits](/docs/{{version}}/starter-kits).

<a name="the-email-verification-handler"></a>
### The Email Verification Handler

Tiếp theo, chúng ta cần định nghĩa một route sẽ xử lý các request được tạo khi người dùng nhấp vào link xác minh email đã được gửi cho họ. Route này nên được đặt tên là `verification.verify` và được gán các middlewares `auth` và `signed`:

```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();

    return redirect('/home');
})->middleware(['auth', 'signed'])->name('verification.verify');
```

Trước khi tiếp tục, hãy xem xét kỹ hơn route này. Đầu tiên, bạn sẽ nhận thấy chúng ta đang sử dụng một kiểu request `EmailVerificationRequest` thay vì instance `Illuminate\Http\Request` điển hình. `EmailVerificationRequest` là một [form request](/docs/{{version}}/validation#form-request-validation) được bao gồm với Laravel. Request này sẽ tự động lo việc xác thực các parameter `id` và `hash` của request.

Tiếp theo, chúng ta có thể tiếp tục trực tiếp đến việc gọi method `fulfill` trên request. Method này sẽ gọi method `markEmailAsVerified` trên người dùng đã xác thực và dispatch event `Illuminate\Auth\Events\Verified`. Method `markEmailAsVerified` có sẵn cho model `App\Models\User` mặc định thông qua class base `Illuminate\Foundation\Auth\User`. Khi địa chỉ email của người dùng đã được xác minh, bạn có thể redirect họ đến bất cứ nơi nào bạn muốn.

<a name="resending-the-verification-email"></a>
### Resending the Verification Email

Đôi khi người dùng có thể làm mất hoặc vô tình xóa email xác minh địa chỉ email. Để đáp ứng điều này, bạn có thể muốn định nghĩa một route để cho phép người dùng yêu cầu gửi lại email xác minh. Sau đó bạn có thể thực hiện request đến route này bằng cách đặt một nút gửi form đơn giản trong [view thông báo xác minh](#the-email-verification-notice) của bạn:

```php
use Illuminate\Http\Request;

Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

<a name="protecting-routes"></a>
### Protecting Routes

[Route middleware](/docs/{{version}}/middleware) có thể được sử dụng để chỉ cho phép người dùng đã xác minh truy cập một route nhất định. Laravel bao gồm một [alias middleware](/docs/{{version}}/middleware#middleware-aliases) `verified`, là một alias cho class middleware `Illuminate\Auth\Middleware\EnsureEmailIsVerified`. Vì alias này đã được đăng ký tự động bởi Laravel, tất cả những gì bạn cần làm là gắn middleware `verified` vào một định nghĩa route. Thông thường, middleware này được kết hợp với middleware `auth`:

```php
Route::get('/profile', function () {
    // Only verified users may access this route...
})->middleware(['auth', 'verified']);
```

Nếu người dùng chưa xác minh cố gắng truy cập một route đã được gán middleware này, họ sẽ tự động được redirect đến [named route](/docs/{{version}}/routing#named-routes) `verification.notice`.

<a name="customization"></a>
## Customization

<a name="verification-email-customization"></a>
#### Verification Email Customization

Mặc dù notification xác minh email mặc định nên đáp ứng các yêu cầu của hầu hết các ứng dụng, Laravel cho phép bạn tùy chỉnh cách thông báo email xác minh được xây dựng.

Để bắt đầu, truyền một closure cho method `toMailUsing` được cung cấp bởi notification `Illuminate\Auth\Notifications\VerifyEmail`. Closure sẽ nhận instance model notifiable đang nhận notification cũng như URL xác minh email đã ký mà người dùng phải truy cập để xác minh địa chỉ email của họ. Closure nên trả về một instance của `Illuminate\Notifications\Messages\MailMessage`. Thông thường, bạn nên gọi method `toMailUsing` từ method `boot` của class `AppServiceProvider` của ứng dụng của bạn:

```php
use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // ...

    VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
        return (new MailMessage)
            ->subject('Verify Email Address')
            ->line('Click the button below to verify your email address.')
            ->action('Verify Email Address', $url);
    });
}
```

> [!NOTE]
> Để tìm hiểu thêm về mail notifications, vui lòng tham khảo [tài liệu mail notification](/docs/{{version}}/notifications#mail-notifications).

<a name="events"></a>
## Events

Khi sử dụng [Laravel application starter kits](/docs/{{version}}/starter-kits), Laravel dispatch một [event](/docs/{{version}}/events) `Illuminate\Auth\Events\Verified` trong quá trình xác minh email. Nếu bạn đang xử lý xác minh email thủ công cho ứng dụng của bạn, bạn có thể muốn dispatch các event này thủ công sau khi xác minh hoàn tất.

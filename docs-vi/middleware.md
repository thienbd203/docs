# Middleware

- [Introduction](#introduction)
- [Defining Middleware](#defining-middleware)
- [Registering Middleware](#registering-middleware)
    - [Global Middleware](#global-middleware)
    - [Assigning Middleware to Routes](#assigning-middleware-to-routes)
    - [Middleware Groups](#middleware-groups)
    - [Middleware Aliases](#middleware-aliases)
    - [Sorting Middleware](#sorting-middleware)
- [Middleware Parameters](#middleware-parameters)
- [Terminable Middleware](#terminable-middleware)

<a name="introduction"></a>
## Introduction

Middleware cung cấp một cơ chế thuận tiện để kiểm tra và lọc các HTTP request đi vào ứng dụng của bạn. Ví dụ, Laravel bao gồm một middleware để xác minh người dùng của ứng dụng đã được xác thực. Nếu người dùng chưa được xác thực, middleware sẽ chuyển hướng người dùng đến màn hình đăng nhập của ứng dụng. Tuy nhiên, nếu người dùng đã được xác thực, middleware sẽ cho phép request tiếp tục đi sâu hơn vào ứng dụng.

Các middleware bổ sung có thể được viết để thực hiện nhiều nhiệm vụ khác ngoài xác thực. Ví dụ, một middleware logging có thể ghi log tất cả các request đến ứng dụng của bạn. Nhiều middleware được bao gồm trong Laravel, bao gồm middleware cho xác thực và bảo vệ CSRF; tuy nhiên, tất cả middleware do người dùng định nghĩa thường nằm trong thư mục `app/Http/Middleware` của ứng dụng.

<a name="defining-middleware"></a>
## Defining Middleware

Để tạo một middleware mới, sử dụng command Artisan `make:middleware`:

```shell
php artisan make:middleware EnsureTokenIsValid
```

Command này sẽ đặt một class `EnsureTokenIsValid` mới trong thư mục `app/Http/Middleware` của bạn. Trong middleware này, chúng ta chỉ cho phép truy cập vào route nếu input `token` được cung cấp khớp với một giá trị được chỉ định. Nếu không, chúng ta sẽ chuyển hướng người dùng trở lại URI `/home`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

Như bạn có thể thấy, nếu `token` được cung cấp không khớp với secret token của chúng ta, middleware sẽ trả về một HTTP redirect cho client; nếu không, request sẽ được chuyển tiếp sâu hơn vào ứng dụng. Để chuyển request sâu hơn vào ứng dụng (cho phép middleware "pass"), bạn nên gọi callback `$next` với `$request`.

Tốt nhất là hình dung middleware như một chuỗi các "lớp" mà HTTP request phải đi qua trước khi đến ứng dụng của bạn. Mỗi lớp có thể kiểm tra request và thậm chí từ chối nó hoàn toàn.

> [!NOTE]
> Tất cả middleware đều được giải quyết qua [service container](/docs/{{version}}/container), vì vậy bạn có thể type-hint bất kỳ dependencies nào bạn cần trong constructor của middleware.

<a name="middleware-and-responses"></a>
#### Middleware and Responses

Tất nhiên, middleware có thể thực hiện nhiệm vụ trước hoặc sau khi chuyển request sâu hơn vào ứng dụng. Ví dụ, middleware sau sẽ thực hiện một số nhiệm vụ **trước** khi request được xử lý bởi ứng dụng:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // Perform action

        return $next($request);
    }
}
```

Tuy nhiên, middleware này sẽ thực hiện nhiệm vụ của nó **sau** khi request được xử lý bởi ứng dụng:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```

<a name="registering-middleware"></a>
## Registering Middleware

<a name="global-middleware"></a>
### Global Middleware

Nếu bạn muốn một middleware chạy trong mọi HTTP request đến ứng dụng của bạn, bạn có thể thêm nó vào global middleware stack trong file `bootstrap/app.php` của ứng dụng:

```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware): void {
     $middleware->append(EnsureTokenIsValid::class);
})
```

Object `$middleware` được cung cấp cho closure `withMiddleware` là một instance của `Illuminate\Foundation\Configuration\Middleware` và chịu trách nhiệm quản lý các middleware được gán cho routes của ứng dụng. Method `append` thêm middleware vào cuối danh sách global middleware. Nếu bạn muốn thêm một middleware vào đầu danh sách, bạn nên sử dụng method `prepend`.

<a name="manually-managing-laravels-default-global-middleware"></a>
#### Manually Managing Laravel's Default Global Middleware

Nếu bạn muốn quản lý global middleware stack của Laravel thủ công, bạn có thể cung cấp default global middleware stack của Laravel cho method `use`. Sau đó, bạn có thể điều chỉnh default middleware stack khi cần thiết:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->use([
        \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class,
        // \Illuminate\Http\Middleware\TrustHosts::class,
        \Illuminate\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Http\Middleware\ValidatePostSize::class,
        \Illuminate\Foundation\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ]);
})
```

<a name="assigning-middleware-to-routes"></a>
### Assigning Middleware to Routes

Nếu bạn muốn gán middleware cho các route cụ thể, bạn có thể gọi method `middleware` khi định nghĩa route:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

Bạn có thể gán nhiều middleware cho route bằng cách chuyển một mảng tên middleware cho method `middleware`:

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

<a name="excluding-middleware"></a>
#### Excluding Middleware

Khi gán middleware cho một nhóm route, đôi khi bạn cần ngăn middleware được áp dụng cho một route cụ thể trong nhóm. Bạn có thể thực hiện điều này bằng cách sử dụng method `withoutMiddleware`:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

Bạn cũng có thể loại bỏ một tập hợp middleware nhất định từ toàn bộ [group](/docs/{{version}}/routing#route-groups) của định nghĩa route:

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

Method `withoutMiddleware` chỉ có thể loại bỏ route middleware và không áp dụng cho [global middleware](#global-middleware).

<a name="middleware-groups"></a>
### Middleware Groups

Đôi khi bạn muốn nhóm nhiều middleware dưới một key duy nhất để dễ dàng gán cho routes. Bạn có thể thực hiện điều này bằng cách sử dụng method `appendToGroup` trong file `bootstrap/app.php` của ứng dụng:

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->appendToGroup('group-name', [
        First::class,
        Second::class,
    ]);

    $middleware->prependToGroup('group-name', [
        First::class,
        Second::class,
    ]);
})
```

Middleware groups có thể được gán cho routes và controller actions bằng cách sử dụng cùng cú pháp như middleware riêng lẻ:

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

<a name="laravels-default-middleware-groups"></a>
#### Laravel's Default Middleware Groups

Laravel bao gồm các middleware group `web` và `api` được định nghĩa sẵn chứa các middleware phổ biến mà bạn có thể muốn áp dụng cho web và API routes của mình. Hãy nhớ rằng, Laravel tự động áp dụng các middleware groups này cho các file `routes/web.php` và `routes/api.php` tương ứng:

<div class="overflow-auto">

|| The `web` Middleware Group                                ||
|| --------------------------------------------------------- ||
|| `Illuminate\Cookie\Middleware\EncryptCookies`             ||
|| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` ||
|| `Illuminate\Session\Middleware\StartSession`              ||
|| `Illuminate\View\Middleware\ShareErrorsFromSession`       ||
|| `Illuminate\Foundation\Http\Middleware\PreventRequestForgery` ||
|| `Illuminate\Routing\Middleware\SubstituteBindings`        ||

</div>

<div class="overflow-auto">

|| The `api` Middleware Group                         ||
|| -------------------------------------------------- ||
|| `Illuminate\Routing\Middleware\SubstituteBindings` ||

</div>

Nếu bạn muốn thêm hoặc đặt trước middleware cho các groups này, bạn có thể sử dụng các method `web` và `api` trong file `bootstrap/app.php` của ứng dụng. Các method `web` và `api` là các lựa chọn thay thế thuận tiện cho method `appendToGroup`:

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

Bạn thậm chí có thể thay thế một trong các middleware group mặc định của Laravel bằng một middleware tùy chỉnh của riêng bạn:

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

Hoặc, bạn có thể loại bỏ một middleware hoàn toàn:

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

<a name="manually-managing-laravels-default-middleware-groups"></a>
#### Manually Managing Laravel's Default Middleware Groups

Nếu bạn muốn quản lý thủ công tất cả middleware trong các middleware group `web` và `api` mặc định của Laravel, bạn có thể định nghĩa lại các groups hoàn toàn. Ví dụ dưới đây sẽ định nghĩa các middleware group `web` và `api` với middleware mặc định của chúng, cho phép bạn tùy chỉnh khi cần thiết:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestForgery::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
    ]);

    $middleware->group('api', [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        // 'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```

> [!NOTE]
> Theo mặc định, các middleware group `web` và `api` được áp dụng tự động cho các file `routes/web.php` và `routes/api.php` tương ứng của ứng dụng bởi file `bootstrap/app.php`.

<a name="middleware-aliases"></a>
### Middleware Aliases

Bạn có thể gán aliases cho middleware trong file `bootstrap/app.php` của ứng dụng. Middleware aliases cho phép bạn định nghĩa một alias ngắn cho một middleware class nhất định, điều này có thể đặc biệt hữu ích cho middleware với tên class dài:

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

Khi middleware alias đã được định nghĩa trong file `bootstrap/app.php` của ứng dụng, bạn có thể sử dụng alias khi gán middleware cho routes:

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

Để thuận tiện, một số middleware tích hợp sẵn của Laravel được đặt alias theo mặc định. Ví dụ, middleware `auth` là một alias cho middleware `Illuminate\Auth\Middleware\Authenticate`. Dưới đây là danh sách các middleware aliases mặc định:

<div class="overflow-auto">

|| Alias              | Middleware                                                                                                    ||
|| ------------------ | ------------------------------------------------------------------------------------------------------------- ||
|| `auth`             | `Illuminate\Auth\Middleware\Authenticate`                                                                     ||
|| `auth.basic`       | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth`                                                        ||
|| `auth.session`     | `Illuminate\Session\Middleware\AuthenticateSession`                                                           ||
|| `cache.headers`    | `Illuminate\Http\Middleware\SetCacheHeaders`                                                                  ||
|| `can`              | `Illuminate\Auth\Middleware\Authorize`                                                                        ||
|| `guest`            | `Illuminate\Auth\Middleware\RedirectIfAuthenticated`                                                          ||
|| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword`                                                                  ||
|| `precognitive`     | `Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests`                                            ||
|| `signed`           | `Illuminate\Routing\Middleware\ValidateSignature`                                                             ||
|| `subscribed`       | `\Spark\Http\Middleware\VerifyBillableIsSubscribed`                                                           ||
|| `throttle`         | `Illuminate\Routing\Middleware\ThrottleRequests` hoặc `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` ||
|| `verified`         | `Illuminate\Auth\Middleware\EnsureEmailIsVerified`                                                            ||

</div>

<a name="sorting-middleware"></a>
### Sorting Middleware

Hiếm khi, bạn có thể cần middleware của mình thực thi theo một thứ tự cụ thể nhưng không có quyền kiểm soát thứ tự của chúng khi được gán cho route. Trong những tình huống này, bạn có thể chỉ định middleware priority bằng cách sử dụng method `priority` trong file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestForgery::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

<a name="middleware-parameters"></a>
## Middleware Parameters

Middleware cũng có thể nhận các tham số bổ sung. Ví dụ, nếu ứng dụng của bạn cần xác minh rằng người dùng đã xác thực có một "role" nhất định trước khi thực hiện một hành động nhất định, bạn có thể tạo một middleware `EnsureUserHasRole` nhận tên role làm một tham số bổ sung.

Các tham số middleware bổ sung sẽ được chuyển đến middleware sau tham số `$next`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }
}
```

Các tham số middleware có thể được chỉ định khi định nghĩa route bằng cách phân tách tên middleware và tham số bằng `:`:

```php
use App\Http\Middleware\EnsureUserHasRole;

Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor');
```

Nhiều tham số có thể được phân tách bằng dấu phẩy:

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor,publisher');
```

<a name="terminable-middleware"></a>
## Terminable Middleware

Đôi khi middleware có thể cần thực hiện một số công việc sau khi HTTP response đã được gửi đến trình duyệt. Nếu bạn định nghĩa một method `terminate` trên middleware của bạn và web server của bạn đang sử dụng [FastCGI](https://www.php.net/manual/en/install.fpm.php), method `terminate` sẽ tự động được gọi sau khi response được gửi đến trình duyệt:

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

Method `terminate` nên nhận cả request và response. Khi bạn đã định nghĩa một terminable middleware, bạn nên thêm nó vào danh sách routes hoặc global middleware trong file `bootstrap/app.php` của ứng dụng.

Khi gọi method `terminate` trên middleware của bạn, Laravel sẽ giải quyết một instance mới của middleware từ [service container](/docs/{{version}}/container). Nếu bạn muốn sử dụng cùng middleware instance khi các method `handle` và `terminate` được gọi, đăng ký middleware với container bằng cách sử dụng method `singleton` của container. Thông thường điều này nên được thực hiện trong method `register` của `AppServiceProvider` của bạn:

```php
use App\Http\Middleware\TerminatingMiddleware;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```

# URL Generation

- [Introduction](#introduction)
- [The Basics](#the-basics)
    - [Generating URLs](#generating-urls)
    - [Accessing the Current URL](#accessing-the-current-url)
- [URLs for Named Routes](#urls-for-named-routes)
    - [Signed URLs](#signed-urls)
- [URLs for Controller Actions](#urls-for-controller-actions)
- [Fluent URI Objects](#fluent-uri-objects)
- [Default Values](#default-values)

<a name="introduction"></a>
## Introduction

Laravel cung cấp một số helpers để hỗ trợ bạn tạo URLs cho ứng dụng của mình. Các helpers này chủ yếu hữu ích khi xây dựng các links trong các templates và phản hồi API, hoặc khi tạo các phản hồi redirect đến một phần khác của ứng dụng.

<a name="the-basics"></a>
## The Basics

<a name="generating-urls"></a>
### Generating URLs

Helper `url` có thể được sử dụng để tạo các URLs tùy ý cho ứng dụng của bạn. URL được tạo sẽ tự động sử dụng scheme (HTTP hoặc HTTPS) và host từ request hiện tại đang được xử lý bởi ứng dụng:

```php
$post = App\Models\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

Để tạo một URL với các tham số query string, bạn có thể sử dụng phương thức `query`:

```php
echo url()->query('/posts', ['search' => 'Laravel']);

// https://example.com/posts?search=Laravel

echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

// http://example.com/posts?sort=latest&search=Laravel
```

Cung cấp các tham số query string đã tồn tại trong path sẽ ghi đè giá trị hiện có của chúng:

```php
echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

// http://example.com/posts?sort=oldest
```

Các arrays của các giá trị cũng có thể được chuyển làm tham số query. Các giá trị này sẽ được key và mã hóa đúng trong URL được tạo:

```php
echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

// http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

echo urldecode($url);

// http://example.com/posts?columns[0]=title&columns[1]=body
```

<a name="accessing-the-current-url"></a>
### Accessing the Current URL

Nếu không có path nào được cung cấp cho helper `url`, một instance `Illuminate\Routing\UrlGenerator` được trả về, cho phép bạn truy cập thông tin về URL hiện tại:

```php
// Get the current URL without the query string...
echo url()->current();

// Get the current URL including the query string...
echo url()->full();
```

Mỗi phương thức này cũng có thể được truy cập thông qua [facade](/docs/{{version}}/facades) `URL`:

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

<a name="accessing-the-previous-url"></a>
#### Accessing the Previous URL

Đôi khi hữu ích để biết URL trước đó mà người dùng đang truy cập từ đó. Bạn có thể truy cập URL trước đó thông qua các phương thức `previous` và `previousPath` của helper `url`:

```php
// Get the full URL for the previous request...
echo url()->previous();

// Get the path for the previous request...
echo url()->previousPath();
```

Hoặc, thông qua [session](/docs/{{version}}/session), bạn có thể truy cập URL trước đó như một instance [fluent URI](#fluent-uri-objects):

```php
use Illuminate\Http\Request;

Route::post('/users', function (Request $request) {
    $previousUri = $request->session()->previousUri();

    // ...
});
```

Cũng có thể truy xuất tên route cho URL đã truy cập trước đó thông qua session:

```php
$previousRoute = $request->session()->previousRoute();
```

<a name="urls-for-named-routes"></a>
## URLs for Named Routes

Helper `route` có thể được sử dụng để tạo URLs đến [named routes](/docs/{{version}}/routing#named-routes). Named routes cho phép bạn tạo URLs mà không bị kết hợp với URL thực tế được định nghĩa trên route. Do đó, nếu URL của route thay đổi, không cần thực hiện thay đổi nào đối với các cuộc gọi của bạn đến hàm `route`. Ví dụ, hãy tưởng tượng ứng dụng của bạn chứa một route được định nghĩa như sau:

```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```

Để tạo một URL đến route này, bạn có thể sử dụng helper `route` như sau:

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

Tất nhiên, helper `route` cũng có thể được sử dụng để tạo URLs cho các routes với nhiều tham số:

```php
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    // ...
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```

Bất kỳ phần tử array bổ sung nào không tương ứng với các tham số định nghĩa của route sẽ được thêm vào query string của URL:

```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.com/post/1?search=rocket
```

<a name="eloquent-models"></a>
#### Eloquent Models

Bạn thường sẽ tạo URLs bằng cách sử dụng route key (thường là primary key) của [Eloquent models](/docs/{{version}}/eloquent). Vì lý do này, bạn có thể chuyển các Eloquent models làm giá trị tham số. Helper `route` sẽ tự động trích xuất route key của model:

```php
echo route('post.show', ['post' => $post]);
```

<a name="signed-urls"></a>
### Signed URLs

Laravel cho phép bạn dễ dàng tạo các URLs "signed" đến named routes. Các URLs này có một hash "signature" được thêm vào query string cho phép Laravel xác minh rằng URL chưa được sửa đổi kể từ khi nó được tạo. Signed URLs đặc biệt hữu ích cho các routes có thể truy cập công khai nhưng cần một lớp bảo vệ chống lại thao tác URL.

Ví dụ, bạn có thể sử dụng signed URLs để triển khai một link "unsubscribe" công khai được gửi email cho khách hàng của bạn. Để tạo một signed URL đến một named route, sử dụng phương thức `signedRoute` của facade `URL`:

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

Bạn có thể loại bỏ domain khỏi hash signed URL bằng cách cung cấp đối số `absolute` cho phương thức `signedRoute`:

```php
return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```

Nếu bạn muốn tạo một URL signed route tạm thời hết hạn sau một khoảng thời gian nhất định, bạn có thể sử dụng phương thức `temporarySignedRoute`. Khi Laravel xác thực một URL signed route tạm thời, nó sẽ đảm bảo rằng timestamp hết hạn được mã hóa vào signed URL chưa trôi qua:

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->plus(minutes: 30), ['user' => 1]
);
```

<a name="validating-signed-route-requests"></a>
#### Validating Signed Route Requests

Để xác minh rằng một request đến có một signature hợp lệ, bạn nên gọi phương thức `hasValidSignature` trên instance `Illuminate\Http\Request` đến:

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

Đôi khi, bạn có thể cần cho phép frontend của ứng dụng thêm dữ liệu vào một signed URL, chẳng hạn như khi thực hiện phân trang phía client. Do đó, bạn có thể chỉ định các tham số query request nên bị bỏ qua khi xác thực một signed URL bằng cách sử dụng phương thức `hasValidSignatureWhileIgnoring`. Hãy nhớ rằng, bỏ qua các tham số cho phép bất kỳ ai sửa đổi các tham số đó trên request:

```php
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```

Thay vì xác thực signed URLs bằng cách sử dụng instance request đến, bạn có thể gán middleware `signed` (`Illuminate\Routing\Middleware\ValidateSignature`) cho route. Nếu request đến không có một signature hợp lệ, middleware sẽ tự động trả về một phản hồi HTTP `403`:

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

Nếu các signed URLs của bạn không bao gồm domain trong hash URL, bạn nên cung cấp đối số `relative` cho middleware:

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed:relative');
```

<a name="responding-to-invalid-signed-routes"></a>
#### Responding to Invalid Signed Routes

Khi ai đó truy cập một signed URL đã hết hạn, họ sẽ nhận được một trang lỗi chung cho mã trạng thái HTTP `403`. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách định nghĩa một closure "render" tùy chỉnh cho ngoại lệ `InvalidSignatureException` trong file `bootstrap/app.php` của ứng dụng:

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidSignatureException $e) {
        return response()->view('errors.link-expired', status: 403);
    });
})
```

<a name="urls-for-controller-actions"></a>
## URLs for Controller Actions

Hàm `action` tạo một URL cho hành động controller đã cho:

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

Nếu phương thức controller chấp nhận các tham số route, bạn có thể chuyển một array kết hợp của các tham số route làm đối số thứ hai cho hàm:

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="fluent-uri-objects"></a>
## Fluent URI Objects

Lớp `Uri` của Laravel cung cấp một giao diện thuận tiện và fluent để tạo và thao tác URIs thông qua các objects. Lớp này bao gồm chức năng được cung cấp bởi package League URI bên dưới và tích hợp liền mạch với hệ thống routing của Laravel.

Bạn có thể tạo một instance `Uri` dễ dàng bằng cách sử dụng các phương thức tĩnh:

```php
use App\Http\Controllers\UserController;
use App\Http\Controllers\InvokableController;
use Illuminate\Support\Uri;

// Generate a URI instance from the given string...
$uri = Uri::of('https://example.com/path');

// Generate URI instances to paths, named routes, or controller actions...
$uri = Uri::to('/dashboard');
$uri = Uri::route('users.show', ['user' => 1]);
$uri = Uri::signedRoute('users.show', ['user' => 1]);
$uri = Uri::temporarySignedRoute('user.index', now()->plus(minutes: 5));
$uri = Uri::action([UserController::class, 'index']);
$uri = Uri::action(InvokableController::class);

// Generate a URI instance from the current request URL...
$uri = $request->uri();

// Generate a URI instance from the previous request URL...
$uri = $request->session()->previousUri();
```

Sau khi bạn có một instance URI, bạn có thể sửa đổi nó một cách fluent:

```php
$uri = Uri::of('https://example.com')
    ->withScheme('http')
    ->withHost('test.com')
    ->withPort(8000)
    ->withPath('/users')
    ->withQuery(['page' => 2])
    ->withFragment('section-1');
```

Để biết thêm thông tin về làm việc với các objects URI fluent, hãy tham khảo [tài liệu URI](/docs/{{version}}/helpers#uri).

<a name="default-values"></a>
## Default Values

Đối với một số ứng dụng, bạn có thể muốn chỉ định các giá trị mặc định trên phạm vi request cho một số tham số URL nhất định. Ví dụ, hãy tưởng tượng nhiều routes của bạn định nghĩa một tham số `{locale}`:

```php
Route::get('/{locale}/posts', function () {
    // ...
})->name('post.index');
```

Việc luôn chuyển `locale` mỗi lần bạn gọi helper `route` là phiền phức. Vì vậy, bạn có thể sử dụng phương thức `URL::defaults` để định nghĩa một giá trị mặc định cho tham số này sẽ luôn được áp dụng trong request hiện tại. Bạn có thể muốn gọi phương thức này từ một [route middleware](/docs/{{version}}/middleware#assigning-middleware-to-routes) để bạn có quyền truy cập vào request hiện tại:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;
use Symfony\Component\HttpFoundation\Response;

class SetDefaultLocaleForUrls
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```

Sau khi giá trị mặc định cho tham số `locale` đã được đặt, bạn không còn cần chuyển giá trị của nó khi tạo URLs thông qua helper `route`.

<a name="url-defaults-middleware-priority"></a>
#### URL Defaults and Middleware Priority

Đặt các giá trị mặc định URL có thể can thiệp vào cách xử lý của Laravel đối với các model bindings ngầm định. Do đó, bạn nên [ưu tiên middleware](/docs/{{version}}/middleware#sorting-middleware) của bạn đặt các giá trị mặc định URL để được thực thi trước middleware `SubstituteBindings` riêng của Laravel. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức middleware `priority` trong file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->prependToPriorityList(
        before: \Illuminate\Routing\Middleware\SubstituteBindings::class,
        prepend: \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    );
})
```

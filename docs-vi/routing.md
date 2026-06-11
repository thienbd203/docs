# Routing

- [Basic Routing](#basic-routing)
    - [The Default Route Files](#the-default-route-files)
    - [Redirect Routes](#redirect-routes)
    - [View Routes](#view-routes)
    - [Listing Your Routes](#listing-your-routes)
    - [Routing Customization](#routing-customization)
- [Route Parameters](#route-parameters)
    - [Required Parameters](#required-parameters)
    - [Optional Parameters](#parameters-optional-parameters)
    - [Regular Expression Constraints](#parameters-regular-expression-constraints)
- [Named Routes](#named-routes)
- [Route Groups](#route-groups)
    - [Middleware](#route-group-middleware)
    - [Controllers](#route-group-controllers)
    - [Subdomain Routing](#route-group-subdomain-routing)
    - [Route Prefixes](#route-group-prefixes)
    - [Route Name Prefixes](#route-group-name-prefixes)
- [Route Model Binding](#route-model-binding)
    - [Implicit Binding](#implicit-binding)
    - [Implicit Enum Binding](#implicit-enum-binding)
    - [Explicit Binding](#explicit-binding)
- [Fallback Routes](#fallback-routes)
- [Rate Limiting](#rate-limiting)
    - [Defining Rate Limiters](#defining-rate-limiters)
    - [Attaching Rate Limiters to Routes](#attaching-rate-limiters-to-routes)
- [Form Method Spoofing](#form-method-spoofing)
- [Accessing the Current Route](#accessing-the-current-route)
- [Cross-Origin Resource Sharing (CORS)](#cors)
- [Route Caching](#route-caching)

<a name="basic-routing"></a>
## Basic Routing

Các route Laravel cơ bản nhất chấp nhận một URI và một closure, cung cấp một phương pháp rất đơn giản và rõ ràng để định nghĩa routes và hành vi mà không cần các file cấu hình routing phức tạp:

```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

<a name="the-default-route-files"></a>
### The Default Route Files

Tất cả các route Laravel được định nghĩa trong các file route của bạn, nằm trong thư mục `routes`. Các file này được tự động tải bởi Laravel sử dụng cấu hình được chỉ định trong file `bootstrap/app.php` của ứng dụng. File `routes/web.php` định nghĩa các route dành cho web interface của bạn. Các route này được gán cho [middleware group](/docs/{{version}}/middleware#laravels-default-middleware-groups) `web`, cung cấp các tính năng như session state và CSRF protection.

Đối với hầu hết các ứng dụng, bạn sẽ bắt đầu bằng cách định nghĩa routes trong file `routes/web.php`. Các route được định nghĩa trong `routes/web.php` có thể được truy cập bằng cách nhập URL của route đã định nghĩa trong trình duyệt của bạn. Ví dụ, bạn có thể truy cập route sau bằng cách điều hướng đến `http://example.com/user` trong trình duyệt:

```php
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

<a name="api-routes"></a>
#### API Routes

Nếu ứng dụng của bạn cũng sẽ cung cấp một API stateless, bạn có thể bật API routing bằng command Artisan `install:api`:

```shell
php artisan install:api
```

Command `install:api` cài đặt [Laravel Sanctum](/docs/{{version}}/sanctum), cung cấp một guard xác thực token API mạnh mẽ nhưng đơn giản có thể được sử dụng để xác thực các consumers API bên thứ ba, SPAs, hoặc mobile applications. Ngoài ra, command `install:api` tạo file `routes/api.php`:

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

Tất nhiên, bạn có thể tự do bỏ qua middleware `auth:sanctum` trên các route nên truy cập công khai.

Các route trong `routes/api.php` là stateless và được gán cho [middleware group](/docs/{{version}}/middleware#laravels-default-middleware-groups) `api`. Ngoài ra, URI prefix `/api` được tự động áp dụng cho các route này, vì vậy bạn không cần áp dụng thủ công cho mỗi route trong file. Bạn có thể thay đổi prefix bằng cách sửa file `bootstrap/app.php` của ứng dụng:

```php
->withRouting(
    api: __DIR__.'/../routes/api.php',
    apiPrefix: 'api/admin',
    // ...
)
```

<a name="available-router-methods"></a>
#### Available Router Methods

Router cho phép bạn đăng ký các route phản hồi với bất kỳ HTTP verb nào:

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

Đôi khi bạn có thể cần đăng ký một route phản hồi với nhiều HTTP verbs. Bạn có thể làm điều đó bằng method `match`. Hoặc, bạn thậm chí có thể đăng ký một route phản hồi với tất cả HTTP verbs bằng method `any`:

```php
Route::match(['get', 'post'], '/', function () {
    // ...
});

Route::any('/', function () {
    // ...
});
```

> [!NOTE]
> Khi định nghĩa nhiều route chia sẻ cùng URI, các route sử dụng các method `get`, `post`, `put`, `patch`, `delete`, và `options` nên được định nghĩa trước các route sử dụng các method `any`, `match`, và `redirect`. Điều này đảm bảo request đến được khớp với route đúng.

<a name="dependency-injection"></a>
#### Dependency Injection

Bạn có thể type-hint bất kỳ dependencies nào cần thiết cho route của bạn trong signature callback của route. Các dependencies được khai báo sẽ tự động được giải quyết và inject vào callback bởi Laravel [service container](/docs/{{version}}/container). Ví dụ, bạn có thể type-hint class `Illuminate\Http\Request` để có HTTP request hiện tại tự động được inject vào route callback của bạn:

```php
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // ...
});
```

<a name="csrf-protection"></a>
#### CSRF Protection

Hãy nhớ rằng, bất kỳ HTML forms nào trỏ đến các route `POST`, `PUT`, `PATCH`, hoặc `DELETE` được định nghĩa trong file route `web` nên bao gồm một trường CSRF token. Nếu không, request sẽ bị từ chối. Bạn có thể đọc thêm về CSRF protection trong [CSRF documentation](/docs/{{version}}/csrf):

```blade
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

<a name="redirect-routes"></a>
### Redirect Routes

Nếu bạn đang định nghĩa một route chuyển hướng đến một URI khác, bạn có thể sử dụng method `Route::redirect`. Method này cung cấp một shortcut tiện lợi để bạn không cần định nghĩa một route hoặc controller đầy đủ để thực hiện một redirect đơn giản:

```php
Route::redirect('/here', '/there');
```

Theo mặc định, `Route::redirect` trả về mã trạng thái `302`. Bạn có thể tùy chỉnh mã trạng thái bằng tham số thứ ba tùy chọn:

```php
Route::redirect('/here', '/there', 301);
```

Hoặc, bạn có thể sử dụng method `Route::permanentRedirect` để trả về mã trạng thái `301`:

```php
Route::permanentRedirect('/here', '/there');
```

> [!WARNING]
> Khi sử dụng route parameters trong redirect routes, các tham số sau được reserved bởi Laravel và không thể sử dụng: `destination` và `status`.

<a name="view-routes"></a>
### View Routes

Nếu route của bạn chỉ cần trả về một [view](/docs/{{version}}/views), bạn có thể sử dụng method `Route::view`. Giống như method `redirect`, method này cung cấp một shortcut đơn giản để bạn không cần định nghĩa một route hoặc controller đầy đủ. Method `view` chấp nhận một URI làm argument đầu tiên và tên view làm argument thứ hai. Ngoài ra, bạn có thể cung cấp một mảng dữ liệu để truyền cho view làm argument thứ ba tùy chọn:

```php
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

> [!WARNING]
> Khi sử dụng route parameters trong view routes, các tham số sau được reserved bởi Laravel và không thể sử dụng: `view`, `data`, `status`, và `headers`.

<a name="listing-your-routes"></a>
### Listing Your Routes

Command Artisan `route:list` có thể dễ dàng cung cấp tổng quan về tất cả các route được định nghĩa bởi ứng dụng của bạn:

```shell
php artisan route:list
```

Theo mặc định, route middleware được gán cho mỗi route sẽ không được hiển thị trong đầu ra `route:list`; tuy nhiên, bạn có thể hướng dẫn Laravel hiển thị route middleware và tên middleware group bằng cách thêm tùy chọn `-v` vào command:

```shell
php artisan route:list -v

# Expand middleware groups...
php artisan route:list -vv
```

Bạn cũng có thể hướng dẫn Laravel chỉ hiển thị các route bắt đầu bằng một URI đã cho:

```shell
php artisan route:list --path=api
```

Ngoài ra, bạn có thể hướng dẫn Laravel ẩn bất kỳ route nào được định nghĩa bởi các packages bên thứ ba bằng cách cung cấp tùy chọn `--except-vendor` khi thực thi command `route:list`:

```shell
php artisan route:list --except-vendor
```

Tương tự, bạn cũng có thể hướng dẫn Laravel chỉ hiển thị các route được định nghĩa bởi các packages bên thứ ba bằng cách cung cấp tùy chọn `--only-vendor` khi thực thi command `route:list`:

```shell
php artisan route:list --only-vendor
```

<a name="routing-customization"></a>
### Routing Customization

Theo mặc định, các route của ứng dụng được cấu hình và tải bởi file `bootstrap/app.php`:

```php
<?php

use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```

Tuy nhiên, đôi khi bạn có thể muốn định nghĩa một file hoàn toàn mới để chứa một tập hợp con của các route ứng dụng. Để thực hiện điều này, bạn có thể cung cấp một closure `then` cho method `withRouting`. Trong closure này, bạn có thể đăng ký bất kỳ routes bổ sung nào cần thiết cho ứng dụng:

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

Hoặc, bạn thậm chí có thể kiểm soát hoàn toàn việc đăng ký route bằng cách cung cấp một closure `using` cho method `withRouting`. Khi argument này được truyền, không có HTTP routes nào sẽ được đăng ký bởi framework và bạn chịu trách nhiệm đăng ký thủ công tất cả các routes:

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    commands: __DIR__.'/../routes/console.php',
    using: function () {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->group(base_path('routes/web.php'));
    },
)
```

<a name="route-parameters"></a>
## Route Parameters

<a name="required-parameters"></a>
### Required Parameters

Đôi khi bạn sẽ cần capture các đoạn của URI trong route của bạn. Ví dụ, bạn có thể cần capture ID của người dùng từ URL. Bạn có thể làm điều đó bằng cách định nghĩa route parameters:

```php
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});
```

Bạn có thể định nghĩa bao nhiêu route parameters tùy theo yêu cầu của route:

```php
Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```

Route parameters luôn được bao bọc trong dấu ngoặc nhọn `{}` và nên bao gồm các ký tự chữ cái. Dấu gạch dưới (`_`) cũng có thể chấp nhận trong tên route parameter. Route parameters được inject vào route callbacks / controllers dựa trên thứ tự của chúng - tên của các arguments route callback / controller không quan trọng.

<a name="parameters-and-dependency-injection"></a>
#### Parameters and Dependency Injection

Nếu route của bạn có dependencies mà bạn muốn Laravel service container tự động inject vào callback của route, bạn nên liệt kê route parameters của bạn sau dependencies:

```php
use Illuminate\Http\Request;

Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User '.$id;
});
```

<a name="parameters-optional-parameters"></a>
### Optional Parameters

Thỉnh thoảng bạn có thể cần chỉ định một route parameter có thể không luôn hiện diện trong URI. Bạn có thể làm điều đó bằng cách đặt dấu `?` sau tên parameter. Hãy chắc chắn cung cấp một giá trị mặc định cho biến tương ứng của route:

```php
Route::get('/user/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

<a name="parameters-regular-expression-constraints"></a>
### Regular Expression Constraints

Bạn có thể giới hạn định dạng của route parameters bằng method `where` trên một route instance. Method `where` chấp nhận tên của parameter và một regular expression định nghĩa cách parameter nên được giới hạn:

```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

Để thuận tiện, một số regular expression patterns thường được sử dụng có các helper methods cho phép bạn nhanh chóng thêm pattern constraints vào các route của bạn:

```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', CategoryEnum::cases());
```

Nếu request đến không khớp với các ràng buộc pattern route, một HTTP response 404 sẽ được trả về.

<a name="parameters-global-constraints"></a>
#### Global Constraints

Nếu bạn muốn một route parameter luôn được giới hạn bởi một regular expression đã cho, bạn có thể sử dụng method `pattern`. Bạn nên định nghĩa các patterns này trong method `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}
```

Khi pattern đã được định nghĩa, nó được tự động áp dụng cho tất cả các route sử dụng tên parameter đó:

```php
Route::get('/user/{id}', function (string $id) {
    // Only executed if {id} is numeric...
});
```

<a name="parameters-encoded-forward-slashes"></a>
#### Encoded Forward Slashes

Component routing của Laravel cho phép tất cả các ký tự ngoại trừ `/` có mặt trong các giá trị route parameter. Bạn phải cho phép rõ ràng `/` là một phần của placeholder bằng một regular expression condition `where`:

```php
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

> [!WARNING]
> Encoded forward slashes chỉ được hỗ trợ trong route segment cuối cùng.

<a name="named-routes"></a>
## Named Routes

Named routes cho phép tạo thuận tiện URLs hoặc redirects cho các route cụ thể. Bạn có thể chỉ định một tên cho một route bằng cách chain method `name` vào định nghĩa route:

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

Bạn cũng có thể chỉ định tên route cho controller actions:

```php
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```

> [!WARNING]
> Tên route nên luôn là duy nhất.

<a name="generating-urls-to-named-routes"></a>
#### Generating URLs to Named Routes

Khi bạn đã gán một tên cho một route đã cho, bạn có thể sử dụng tên của route khi tạo URLs hoặc redirects thông qua các helper functions `route` và `redirect` của Laravel:

```php
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');

return to_route('profile');
```

Nếu named route định nghĩa parameters, bạn có thể truyền các parameters làm argument thứ hai cho function `route`. Các parameters đã cho sẽ tự động được chèn vào URL được tạo ở các vị trí đúng của chúng:

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1]);
```

Nếu bạn truyền các parameters bổ sung trong mảng, các cặp key / value đó sẽ tự động được thêm vào query string của URL được tạo:

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// http://example.com/user/1/profile?photos=yes
```

> [!NOTE]
> Đôi khi, bạn có thể muốn chỉ định các giá trị mặc định trên toàn request cho URL parameters, chẳng hạn như locale hiện tại. Để thực hiện điều này, bạn có thể sử dụng [method URL::defaults](/docs/{{version}}/urls#default-values).

<a name="inspecting-the-current-route"></a>
#### Inspecting the Current Route

Nếu bạn muốn xác định xem request hiện tại có được route đến một named route đã cho hay không, bạn có thể sử dụng method `named` trên một Route instance. Ví dụ, bạn có thể kiểm tra tên route hiện tại từ một route middleware:

```php
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * Handle an incoming request.
 *
 * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
 */
public function handle(Request $request, Closure $next): Response
{
    if ($request->route()->named('profile')) {
        // ...
    }

    return $next($request);
}
```

<a name="route-groups"></a>
## Route Groups

Route groups cho phép bạn chia sẻ route attributes, chẳng hạn như middleware, trên một số lượng lớn các route mà không cần định nghĩa các attributes đó trên mỗi route riêng lẻ.

Nested groups cố gắng "merge" thông minh các attributes với group cha của chúng. Middleware và các conditions `where` được merge trong khi tên và prefixes được append. Namespace delimiters và dấu gạch chéo trong URI prefixes được tự động thêm vào nơi thích hợp.

<a name="route-group-middleware"></a>
### Middleware

Để gán [middleware](/docs/{{version}}/middleware) cho tất cả các route trong một group, bạn có thể sử dụng method `middleware` trước khi định nghĩa group. Middleware được thực thi theo thứ tự chúng được liệt kê trong mảng:

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware...
    });

    Route::get('/user/profile', function () {
        // Uses first & second middleware...
    });
});
```

<a name="route-group-controllers"></a>
### Controllers

Nếu một nhóm routes đều sử dụng cùng một [controller](/docs/{{version}}/controllers), bạn có thể sử dụng method `controller` để định nghĩa controller chung cho tất cả các route trong group. Sau đó, khi định nghĩa các routes, bạn chỉ cần cung cấp controller method mà chúng gọi:

```php
use App\Http\Controllers\OrderController;

Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```

<a name="route-group-subdomain-routing"></a>
### Subdomain Routing

Route groups cũng có thể được sử dụng để xử lý subdomain routing. Subdomains có thể được gán route parameters giống như route URIs, cho phép bạn capture một phần của subdomain để sử dụng trong route hoặc controller của bạn. Subdomain có thể được chỉ định bằng cách gọi method `domain` trước khi định nghĩa group:

```php
Route::domain('{account}.example.com')->group(function () {
    Route::get('/user/{id}', function (string $account, string $id) {
        // ...
    });
});
```

<a name="route-group-prefixes"></a>
### Route Prefixes

Method `prefix` có thể được sử dụng để prefix mỗi route trong group với một URI đã cho. Ví dụ, bạn có thể muốn prefix tất cả các route URIs trong group với `admin`:

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // Matches The "/admin/users" URL
    });
});
```

<a name="route-group-name-prefixes"></a>
### Route Name Prefixes

Method `name` có thể được sử dụng để prefix mỗi tên route trong group với một chuỗi đã cho. Ví dụ, bạn có thể muốn prefix tên của tất cả các route trong group với `admin`. Chuỗi đã cho được prefix vào tên route chính xác như được chỉ định, vì vậy chúng ta sẽ chắc chắn cung cấp ký tự `.` ở cuối trong prefix:

```php
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```

<a name="route-model-binding"></a>
## Route Model Binding

Khi inject một model ID vào một route hoặc controller action, bạn thường sẽ query database để truy xuất model tương ứng với ID đó. Laravel route model binding cung cấp một cách thuận tiện để tự động inject các model instances trực tiếp vào các route của bạn. Ví dụ, thay vì inject ID của người dùng, bạn có thể inject toàn bộ `User` model instance khớp với ID đã cho.

<a name="implicit-binding"></a>
### Implicit Binding

Laravel tự động giải quyết các Eloquent models được định nghĩa trong routes hoặc controller actions có tên biến type-hinted khớp với tên route segment. Ví dụ:

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

Vì biến `$user` được type-hinted là Eloquent model `App\Models\User` và tên biến khớp với URI segment `{user}`, Laravel sẽ tự động inject model instance có ID khớp với giá trị tương ứng từ request URI. Nếu một model instance khớp không được tìm thấy trong database, một HTTP response 404 sẽ tự động được tạo ra.

Tất nhiên, implicit binding cũng có thể khi sử dụng controller methods. Một lần nữa, lưu ý URI segment `{user}` khớp với biến `$user` trong controller chứa type-hint `App\Models\User`:

```php
use App\Http\Controllers\UserController;
use App\Models\User;

// Route definition...
Route::get('/users/{user}', [UserController::class, 'show']);

// Controller method definition...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

<a name="implicit-soft-deleted-models"></a>
#### Soft Deleted Models

Thông thường, implicit model binding sẽ không truy xuất các models đã được [soft deleted](/docs/{{version}}/eloquent#soft-deleting). Tuy nhiên, bạn có thể hướng dẫn implicit binding truy xuất các models này bằng cách chain method `withTrashed` vào định nghĩa route của bạn:

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

<a name="customizing-the-default-key-name"></a>
#### Customizing the Key

Đôi khi bạn có thể muốn giải quyết Eloquent models sử dụng một cột khác ngoài `id`. Để làm điều đó, bạn có thể chỉ định cột trong định nghĩa route parameter:

```php
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

Nếu bạn muốn model binding luôn sử dụng một cột database khác ngoài `id` khi truy xuất một model class đã cho, bạn có thể override method `getRouteKeyName` trên Eloquent model:

```php
/**
 * Get the route key for the model.
 */
public function getRouteKeyName(): string
{
    return 'slug';
}
```

<a name="implicit-model-binding-scoping"></a>
#### Custom Keys and Scoping

Khi implicitly binding nhiều Eloquent models trong một định nghĩa route duy nhất, bạn có thể muốn scope Eloquent model thứ hai sao cho nó phải là một con của Eloquent model trước đó. Ví dụ, hãy xem xét định nghĩa route này truy xuất một blog post bằng slug cho một user cụ thể:

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

Khi sử dụng một implicit binding với custom key làm route parameter lồng nhau, Laravel sẽ tự động scope query để truy xuất model lồng nhau bằng parent của nó sử dụng conventions để đoán tên relationship trên parent. Trong trường hợp này, sẽ được giả định rằng model `User` có một relationship tên là `posts` (dạng số nhiều của tên route parameter) có thể được sử dụng để truy xuất model `Post`.

Nếu bạn muốn, bạn có thể hướng dẫn Laravel scope các bindings "child" ngay cả khi custom key không được cung cấp. Để làm điều đó, bạn có thể gọi method `scopeBindings` khi định nghĩa route của bạn:

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

Hoặc, bạn có thể hướng dẫn một nhóm toàn bộ các định nghĩa route sử dụng scoped bindings:

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```

Tương tự, bạn có thể hướng dẫn Laravel rõ ràng không scope bindings bằng cách gọi method `withoutScopedBindings`:

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

<a name="customizing-missing-model-behavior"></a>
#### Customizing Missing Model Behavior

Thông thường, một HTTP response 404 sẽ được tạo ra nếu một implicitly bound model không được tìm thấy. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách gọi method `missing` khi định nghĩa route của bạn. Method `missing` chấp nhận một closure sẽ được gọi nếu một implicitly bound model không thể được tìm thấy:

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

<a name="implicit-enum-binding"></a>
### Implicit Enum Binding

PHP 8.1 giới thiệu hỗ trợ cho [Enums](https://www.php.net/manual/en/language.enumerations.backed.php). Để bổ sung tính năng này, Laravel cho phép bạn type-hint một [string-backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php) trên định nghĩa route của bạn và Laravel sẽ chỉ gọi route nếu route segment đó tương ứng với một giá trị Enum hợp lệ. Nếu không, một HTTP response 404 sẽ tự động được trả về. Ví dụ, với Enum sau:

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

Bạn có thể định nghĩa một route sẽ chỉ được gọi nếu route segment `{category}` là `fruits` hoặc `people`. Nếu không, Laravel sẽ trả về một HTTP response 404:

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### Explicit Binding

Bạn không cần sử dụng implicit, convention based model resolution của Laravel để sử dụng model binding. Bạn cũng có thể định nghĩa rõ ràng cách route parameters tương ứng với models. Để đăng ký một explicit binding, sử dụng method `model` của router để chỉ định class cho một parameter đã cho. Bạn nên định nghĩa các explicit model bindings của bạn ở đầu method `boot` của class `AppServiceProvider` của bạn:

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::model('user', User::class);
}
```

Tiếp theo, định nghĩa một route chứa parameter `{user}`:

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    // ...
});
```

Vì chúng ta đã bound tất cả các parameters `{user}` với model `App\Models\User`, một instance của class đó sẽ được inject vào route. Vì vậy, ví dụ, một request đến `users/1` sẽ inject instance `User` từ database có ID là `1`.

Nếu một model instance khớp không được tìm thấy trong database, một HTTP response 404 sẽ tự động được tạo ra.

<a name="customizing-the-resolution-logic"></a>
#### Customizing the Resolution Logic

Nếu bạn muốn định nghĩa logic resolution model binding của riêng bạn, bạn có thể sử dụng method `Route::bind`. Closure bạn truyền cho method `bind` sẽ nhận giá trị của URI segment và nên trả về instance của class nên được inject vào route. Một lần nữa, tùy chỉnh này nên diễn ra trong method `boot` của `AppServiceProvider` của ứng dụng:

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });
}
```

Ngoài ra, bạn có thể override method `resolveRouteBinding` trên Eloquent model của bạn. Method này sẽ nhận giá trị của URI segment và nên trả về instance của class nên được inject vào route:

```php
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

Nếu một route đang sử dụng [implicit binding scoping](#implicit-model-binding-scoping), method `resolveChildRouteBinding` sẽ được sử dụng để giải quyết child binding của parent model:

```php
/**
 * Retrieve the child model for a bound value.
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

<a name="fallback-routes"></a>
## Fallback Routes

Sử dụng method `Route::fallback`, bạn có thể định nghĩa một route sẽ được thực thi khi không có route nào khác khớp với request đến. Thông thường, các requests không được xử lý sẽ tự động render một trang "404" thông qua exception handler của ứng dụng. Tuy nhiên, vì bạn thường sẽ định nghĩa route `fallback` trong file `routes/web.php`, tất cả middleware trong middleware group `web` sẽ áp dụng cho route. Bạn có thể tự do thêm middleware bổ sung vào route này khi cần:

```php
Route::fallback(function () {
    // ...
});
```

<a name="rate-limiting"></a>
## Rate Limiting

<a name="defining-rate-limiters"></a>
### Defining Rate Limiters

Laravel bao gồm các dịch vụ rate limiting mạnh mẽ và có thể tùy chỉnh mà bạn có thể sử dụng để giới hạn lượng traffic cho một route hoặc nhóm route đã cho. Để bắt đầu, bạn nên định nghĩa các cấu hình rate limiter đáp ứng nhu cầu của ứng dụng.

Rate limiters có thể được định nghĩa trong method `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

Rate limiters được định nghĩa sử dụng method `for` của facade `RateLimiter`. Method `for` chấp nhận một tên rate limiter và một closure trả về cấu hình limit nên áp dụng cho các routes được gán cho rate limiter. Cấu hình limit là các instances của class `Illuminate\Cache\RateLimiting\Limit`. Class này chứa các "builder" methods hữu ích để bạn có thể nhanh chóng định nghĩa limit của mình. Tên rate limiter có thể là bất kỳ chuỗi nào bạn muốn:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

Nếu request đến vượt quá rate limit đã chỉ định, một response với mã trạng thái HTTP 429 sẽ tự động được trả về bởi Laravel. Nếu bạn muốn định nghĩa response của riêng mình nên được trả về bởi một rate limit, bạn có thể sử dụng method `response`:

```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('Custom response...', 429, $headers);
    });
});
```

Vì rate limiter callbacks nhận instance HTTP request đến, bạn có thể xây dựng rate limit thích hợp một cách động dựa trên request đến hoặc user được xác thực:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()?->vipCustomer()
        ? Limit::none()
        : Limit::perHour(10);
});
```

<a name="segmenting-rate-limits"></a>
#### Segmenting Rate Limits

Đôi khi bạn có thể muốn segment rate limits theo một giá trị tùy ý. Ví dụ, bạn có thể muốn cho phép người dùng truy cập một route đã cho 100 lần mỗi phút mỗi địa chỉ IP. Để thực hiện điều này, bạn có thể sử dụng method `by` khi xây dựng rate limit của bạn:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
        ? Limit::none()
        : Limit::perMinute(100)->by($request->ip());
});
```

Để minh họa tính năng này bằng một ví dụ khác, chúng ta có thể giới hạn truy cập vào route 100 lần mỗi phút mỗi ID user được xác thực hoặc 10 lần mỗi phút mỗi địa chỉ IP cho guests:

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
        ? Limit::perMinute(100)->by($request->user()->id)
        : Limit::perMinute(10)->by($request->ip());
});
```

<a name="multiple-rate-limits"></a>
#### Multiple Rate Limits

Nếu cần, bạn có thể trả về một mảng các rate limits cho một cấu hình rate limiter đã cho. Mỗi rate limit sẽ được đánh giá cho route dựa trên thứ tự chúng được đặt trong mảng:

```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```

Nếu bạn đang gán nhiều rate limits được segment bởi các giá trị `by` giống hệt nhau, bạn nên đảm bảo rằng mỗi giá trị `by` là duy nhất. Cách dễ nhất để đạt được điều này là prefix các giá trị được cung cấp cho method `by`:

```php
RateLimiter::for('uploads', function (Request $request) {
    return [
        Limit::perMinute(10)->by('minute:'.$request->user()->id),
        Limit::perDay(1000)->by('day:'.$request->user()->id),
    ];
});
```

<a name="response-base-rate-limiting"></a>
#### Response-Based Rate Limiting

Ngoài việc rate limiting các requests đến, Laravel cho phép bạn rate limit dựa trên response sử dụng method `after`. Điều này hữu ích khi bạn chỉ muốn đếm một số responses nhất định hướng đến rate limit, chẳng hạn như validation errors, responses 404, hoặc các mã trạng thái HTTP cụ thể khác.

Method `after` chấp nhận một closure nhận response và nên trả về `true` nếu response nên được đếm hướng đến rate limit, hoặc `false` nếu nó nên bị bỏ qua. Điều này đặc biệt hữu ích để ngăn chặn enumeration attacks bằng cách giới hạn các responses 404 liên tiếp, hoặc cho phép người dùng thử lại các requests thất bại validation mà không làm cạn rate limit của họ trên một endpoint chỉ nên throttle các hoạt động thành công:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;
use Symfony\Component\HttpFoundation\Response;

RateLimiter::for('resource-not-found', function (Request $request) {
    return Limit::perMinute(10)
        ->by($request->user()?->id ?: $request->ip())
        ->after(function (Response $response) {
            // Only count 404 responses toward the rate limit to prevent enumeration...
            return $response->status() === 404;
        });
});
```

<a name="attaching-rate-limiters-to-routes"></a>
### Attaching Rate Limiters to Routes

Rate limiters có thể được gán cho routes hoặc route groups sử dụng [middleware](/docs/{{version}}/middleware) `throttle`. Middleware throttle chấp nhận tên của rate limiter bạn muốn gán cho route:

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });

    Route::post('/video', function () {
        // ...
    });
});
```

<a name="throttling-with-redis"></a>
#### Throttling With Redis

Theo mặc định, middleware `throttle` được map với class `Illuminate\Routing\Middleware\ThrottleRequests`. Tuy nhiên, nếu bạn đang sử dụng Redis làm cache driver của ứng dụng, bạn có thể muốn hướng dẫn Laravel sử dụng Redis để quản lý rate limiting. Để làm điều đó, bạn nên sử dụng method `throttleWithRedis` trong file `bootstrap/app.php` của ứng dụng. Method này map middleware `throttle` với class middleware `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis`:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->throttleWithRedis();
    // ...
})
```

<a name="form-method-spoofing"></a>
## Form Method Spoofing

HTML forms không hỗ trợ các hành động `PUT`, `PATCH`, hoặc `DELETE`. Vì vậy, khi định nghĩa các route `PUT`, `PATCH`, hoặc `DELETE` được gọi từ một HTML form, bạn sẽ cần thêm một trường ẩn `_method` vào form. Giá trị được gửi với trường `_method` sẽ được sử dụng làm HTTP request method:

```blade
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

Để thuận tiện, bạn có thể sử dụng [Blade directive](/docs/{{version}}/blade) `@method` để tạo trường input `_method`:

```blade
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>
```

<a name="accessing-the-current-route"></a>
## Accessing the Current Route

Bạn có thể sử dụng các method `current`, `currentRouteName`, và `currentRouteAction` trên facade `Route` để truy cập thông tin về route xử lý request đến:

```php
use Illuminate\Support\Facades\Route;

$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```

Bạn có thể tham khảo tài liệu API cho cả [underlying class của Route facade](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Router.html) và [Route instance](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Route.html) để xem xét tất cả các methods có sẵn trên router và route classes.

<a name="cors"></a>
## Cross-Origin Resource Sharing (CORS)

Laravel có thể tự động phản hồi các HTTP requests CORS `OPTIONS` với các giá trị bạn cấu hình. Các requests `OPTIONS` sẽ tự động được xử lý bởi [middleware](/docs/{{version}}/middleware) `HandleCors` được tự động bao gồm trong global middleware stack của ứng dụng.

Đôi khi, bạn có thể cần tùy chỉnh các giá trị cấu hình CORS cho ứng dụng của bạn. Bạn có thể làm điều đó bằng cách publishing file cấu hình `cors` sử dụng command Artisan `config:publish`:

```shell
php artisan config:publish cors
```

Command này sẽ đặt một file cấu hình `cors.php` trong thư mục `config` của ứng dụng.

> [!NOTE]
> Để biết thêm thông tin về CORS và CORS headers, hãy tham khảo [tài liệu web MDN về CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers).

<a name="route-caching"></a>
## Route Caching

Khi triển khai ứng dụng của bạn đến production, bạn nên tận dụng route cache của Laravel. Sử dụng route cache sẽ giảm đáng kể thời gian cần thiết để đăng ký tất cả các route của ứng dụng. Để tạo một route cache, thực thi command Artisan `route:cache`:

```shell
php artisan route:cache
```

Sau khi chạy command này, file cached routes của bạn sẽ được tải trên mỗi request. Hãy nhớ rằng, nếu bạn thêm bất kỳ routes mới, bạn sẽ cần tạo một route cache mới. Vì lý do này, bạn chỉ nên chạy command `route:cache` trong quá trình deployment của dự án.

Bạn có thể sử dụng command `route:clear` để xóa route cache:

```shell
php artisan route:clear
```

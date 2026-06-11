# Controllers

- [Introduction](#introduction)
- [Writing Controllers](#writing-controllers)
    - [Basic Controllers](#basic-controllers)
    - [Single Action Controllers](#single-action-controllers)
- [Controller Middleware](#controller-middleware)
    - [Middleware Attributes](#middleware-attributes)
    - [Authorization Attributes](#authorization-attributes)
- [Resource Controllers](#resource-controllers)
    - [Partial Resource Routes](#restful-partial-resource-routes)
    - [Nested Resources](#restful-nested-resources)
    - [Naming Resource Routes](#restful-naming-resource-routes)
    - [Naming Resource Route Parameters](#restful-naming-resource-route-parameters)
    - [Scoping Resource Routes](#restful-scoping-resource-routes)
    - [Localizing Resource URIs](#restful-localizing-resource-uris)
    - [Supplementing Resource Controllers](#restful-supplementing-resource-controllers)
    - [Singleton Resource Controllers](#singleton-resource-controllers)
    - [Middleware and Resource Controllers](#middleware-and-resource-controllers)
- [Dependency Injection and Controllers](#dependency-injection-and-controllers)

<a name="introduction"></a>
## Introduction

Thay vì định nghĩa tất cả logic xử lý request của bạn như closures trong các file route, bạn có thể muốn tổ chức hành vi này sử dụng các class "controller". Controllers có thể nhóm logic xử lý request liên quan vào một class duy nhất. Ví dụ, một class `UserController` có thể xử lý tất cả các requests đến liên quan đến người dùng, bao gồm hiển thị, tạo, cập nhật, và xóa người dùng. Theo mặc định, controllers được lưu trữ trong thư mục `app/Http/Controllers`.

<a name="writing-controllers"></a>
## Writing Controllers

<a name="basic-controllers"></a>
### Basic Controllers

Để nhanh chóng tạo một controller mới, bạn có thể chạy command Artisan `make:controller`. Theo mặc định, tất cả các controllers cho ứng dụng của bạn được lưu trữ trong thư mục `app/Http/Controllers`:

```shell
php artisan make:controller UserController
```

Hãy xem một ví dụ về một controller cơ bản. Một controller có thể có bất kỳ số lượng public methods nào sẽ phản hồi với các HTTP requests đến:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Khi bạn đã viết một class và method controller, bạn có thể định nghĩa một route đến method controller như sau:

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

Khi một request đến khớp với URI route đã chỉ định, method `show` trên class `App\Http\Controllers\UserController` sẽ được gọi và các route parameters sẽ được truyền vào method.

> [!NOTE]
> Controllers không **cần** extend một base class. Tuy nhiên, đôi khi thuận tiện để extend một base controller class chứa các methods nên được chia sẻ trên tất cả các controllers của bạn.

<a name="single-action-controllers"></a>
### Single Action Controllers

Nếu một controller action đặc biệt phức tạp, bạn có thể thấy thuận tiện để dành toàn bộ một controller class cho action đơn đó. Để thực hiện điều này, bạn có thể định nghĩa một method `__invoke` duy nhất trong controller:

```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     */
    public function __invoke()
    {
        // ...
    }
}
```

Khi đăng ký routes cho single action controllers, bạn không cần chỉ định một controller method. Thay vào đó, bạn có thể đơn giản truyền tên của controller cho router:

```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

Bạn có thể tạo một invokable controller bằng cách sử dụng tùy chọn `--invokable` của command Artisan `make:controller`:

```shell
php artisan make:controller ProvisionServer --invokable
```

> [!NOTE]
> Controller stubs có thể được tùy chỉnh sử dụng [stub publishing](/docs/{{version}}/artisan#stub-customization).

<a name="controller-middleware"></a>
## Controller Middleware

[Middleware](/docs/{{version}}/middleware) có thể được gán cho các routes của controller trong các file route của bạn:

```php
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```

Hoặc, bạn có thể thấy thuận tiện để chỉ định middleware trong class controller của bạn. Để làm điều này, controller của bạn nên implement interface `HasMiddleware`, quy định rằng controller nên có một static method `middleware`. Từ method này, bạn có thể trả về một mảng middleware nên được áp dụng cho các actions của controller:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController implements HasMiddleware
{
    /**
     * Get the middleware that should be assigned to the controller.
     */
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('log', only: ['index']),
            new Middleware('subscribed', except: ['store']),
        ];
    }

    // ...
}
```

Bạn cũng có thể định nghĩa controller middleware như closures, cung cấp một cách thuận tiện để định nghĩa một inline middleware mà không cần viết một middleware class hoàn chỉnh:

```php
use Closure;
use Illuminate\Http\Request;

/**
 * Get the middleware that should be assigned to the controller.
 */
public static function middleware(): array
{
    return [
        function (Request $request, Closure $next) {
            return $next($request);
        },
    ];
}
```

<a name="middleware-attributes"></a>
### Middleware Attributes

Bạn cũng có thể gán middleware cho controllers sử dụng PHP attributes:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
#[Middleware('log', only: ['index'])]
#[Middleware('subscribed', except: ['store'])]
class UserController
{
    // ...
}
```

Bạn cũng có thể đặt middleware attributes trên các controller methods riêng lẻ. Middleware được gán cho methods sẽ được merge với middleware được gán ở cấp class:

```php
<?php

namespace App\Http\Controllers;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class UserController
{
    #[Middleware('log')]
    #[Middleware('subscribed')]
    public function index()
    {
        // ...
    }

    #[Middleware(static function (Request $request, Closure $next) {
        // ...

        return $next($request);
    })]
    public function store()
    {
        // ...
    }
}
```

<a name="authorization-attributes"></a>
### Authorization Attributes

Nếu bạn đang authorizing controller actions thông qua policies, bạn có thể sử dụng attribute `Authorize` như một shortcut thuận tiện cho middleware `can`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Routing\Attributes\Controllers\Authorize;

class CommentController
{
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post)
    {
        // ...
    }

    #[Authorize('delete', 'comment')]
    public function destroy(Comment $comment)
    {
        // ...
    }
}
```

Argument đầu tiên là ability bạn muốn authorize. Argument thứ hai là model class, route parameter, hoặc parameters nên được truyền cho policy.

<a name="resource-controllers"></a>
## Resource Controllers

Nếu bạn coi mỗi Eloquent model trong ứng dụng như một "resource", điển hình là thực hiện cùng một bộ actions đối với mỗi resource trong ứng dụng. Ví dụ, hãy tưởng tượng ứng dụng của bạn chứa một model `Photo` và một model `Movie`. Có khả năng người dùng có thể tạo, đọc, cập nhật, hoặc xóa các resources này.

Vì use case phổ biến này, Laravel resource routing gán các routes create, read, update, và delete ("CRUD") điển hình cho một controller với một dòng code duy nhất. Để bắt đầu, chúng ta có thể sử dụng tùy chọn `--resource` của command Artisan `make:controller` để nhanh chóng tạo một controller để xử lý các actions này:

```shell
php artisan make:controller PhotoController --resource
```

Command này sẽ tạo một controller tại `app/Http/Controllers/PhotoController.php`. Controller sẽ chứa một method cho mỗi resource operation có sẵn. Tiếp theo, bạn có thể đăng ký một resource route trỏ đến controller:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```

Khai báo route duy nhất này tạo nhiều routes để xử lý nhiều actions khác nhau trên resource. Controller được tạo sẽ đã có các methods stubbed cho mỗi action này. Hãy nhớ rằng, bạn luôn có thể nhận được một tổng quan nhanh về các routes của ứng dụng bằng cách chạy command Artisan `route:list`.

Bạn thậm chí có thể đăng ký nhiều resource controllers cùng một lúc bằng cách truyền một mảng cho method `resources`:

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

Method `softDeletableResources` đăng ký nhiều resource controllers đều sử dụng method `withTrashed`:

```php
Route::softDeletableResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

<a name="actions-handled-by-resource-controllers"></a>
#### Actions Handled by Resource Controllers

<div class="overflow-auto">

| Verb      | URI                    | Action  | Route Name     |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

</div>

<a name="customizing-missing-model-behavior"></a>
#### Customizing Missing Model Behavior

Thông thường, một HTTP response 404 sẽ được tạo ra nếu một implicitly bound resource model không được tìm thấy. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách gọi method `missing` khi định nghĩa resource route của bạn. Method `missing` chấp nhận một closure sẽ được gọi nếu một implicitly bound model không thể được tìm thấy cho bất kỳ routes nào của resource:

```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::resource('photos', PhotoController::class)
    ->missing(function (Request $request) {
        return Redirect::route('photos.index');
    });
```

<a name="soft-deleted-models"></a>
#### Soft Deleted Models

Thông thường, implicit model binding sẽ không truy xuất các models đã được [soft deleted](/docs/{{version}}/eloquent#soft-deleting), và thay vào đó sẽ trả về một HTTP response 404. Tuy nhiên, bạn có thể hướng dẫn framework cho phép soft deleted models bằng cách gọi method `withTrashed` khi định nghĩa resource route của bạn:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();
```

Gọi `withTrashed` không có arguments sẽ cho phép soft deleted models cho các resource routes `show`, `edit`, và `update`. Bạn có thể chỉ định một tập hợp con của các routes này bằng cách truyền một mảng cho method `withTrashed`:

```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

<a name="specifying-the-resource-model"></a>
#### Specifying the Resource Model

Nếu bạn đang sử dụng [route model binding](/docs/{{version}}/routing#route-model-binding) và muốn các methods của resource controller type-hint một model instance, bạn có thể sử dụng tùy chọn `--model` khi tạo controller:

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### Generating Form Requests

Bạn có thể cung cấp tùy chọn `--requests` khi tạo một resource controller để hướng dẫn Artisan tạo [form request classes](/docs/{{version}}/validation#form-request-validation) cho các methods storage và update của controller:

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

<a name="restful-partial-resource-routes"></a>
### Partial Resource Routes

Khi khai báo một resource route, bạn có thể chỉ định một tập hợp con các actions controller nên xử lý thay vì bộ đầy đủ các actions mặc định:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

<a name="api-resource-routes"></a>
#### API Resource Routes

Khi khai báo các resource routes sẽ được tiêu thụ bởi APIs, bạn thường sẽ muốn loại bỏ các routes hiển thị HTML templates như `create` và `edit`. Để thuận tiện, bạn có thể sử dụng method `apiResource` để tự động loại bỏ hai routes này:

```php
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```

Bạn có thể đăng ký nhiều API resource controllers cùng một lúc bằng cách truyền một mảng cho method `apiResources`:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;

Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

Để nhanh chóng tạo một API resource controller không bao gồm các methods `create` hoặc `edit`, sử dụng switch `--api` khi thực thi command `make:controller`:

```shell
php artisan make:controller PhotoController --api
```

<a name="restful-nested-resources"></a>
### Nested Resources

Đôi khi bạn có thể cần định nghĩa routes đến một nested resource. Ví dụ, một photo resource có thể có nhiều comments có thể được gắn vào photo. Để nest các resource controllers, bạn có thể sử dụng ký hiệu "dot" trong khai báo route của bạn:

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```

Route này sẽ đăng ký một nested resource có thể được truy cập với các URI như sau:

```text
/photos/{photo}/comments/{comment}
```

<a name="scoping-nested-resources"></a>
#### Scoping Nested Resources

Tính năng [implicit model binding](/docs/{{version}}/routing#implicit-model-binding-scoping) của Laravel có thể tự động scope nested bindings sao cho child model được giải quyết được xác nhận thuộc về parent model. Bằng cách sử dụng method `scoped` khi định nghĩa nested resource của bạn, bạn có thể bật automatic scoping cũng như hướng dẫn Laravel field nào child resource nên được truy xuất bằng. Để biết thêm thông tin về cách thực hiện điều này, hãy xem tài liệu về [scoping resource routes](#restful-scoping-resource-routes).

<a name="shallow-nesting"></a>
#### Shallow Nesting

Thường, không hoàn toàn cần thiết để có cả parent và child IDs trong một URI vì child ID đã là một unique identifier. Khi sử dụng các unique identifiers như auto-incrementing primary keys để xác định các models của bạn trong URI segments, bạn có thể chọn sử dụng "shallow nesting":

```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();
```

Định nghĩa route này sẽ định nghĩa các routes sau:

<div class="overflow-auto">

| Verb      | URI                               | Action  | Route Name             |
| --------- | --------------------------------- | ------- | ---------------------- |
| GET       | `/photos/{photo}/comments`        | index   | photos.comments.index  |
| GET       | `/photos/{photo}/comments/create` | create  | photos.comments.create |
| POST      | `/photos/{photo}/comments`        | store   | photos.comments.store  |
| GET       | `/comments/{comment}`             | show    | comments.show          |
| GET       | `/comments/{comment}/edit`        | edit    | comments.edit          |
| PUT/PATCH | `/comments/{comment}`             | update  | comments.update        |
| DELETE    | `/comments/{comment}`             | destroy | comments.destroy       |

</div>

<a name="restful-naming-resource-routes"></a>
### Naming Resource Routes

Theo mặc định, tất cả các resource controller actions có một route name; tuy nhiên, bạn có thể override các tên này bằng cách truyền một mảng `names` với các route names mong muốn của bạn:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

<a name="restful-naming-resource-route-parameters"></a>
### Naming Resource Route Parameters

Theo mặc định, `Route::resource` sẽ tạo các route parameters cho resource routes của bạn dựa trên phiên bản "singularized" của tên resource. Bạn có thể dễ dàng override điều này trên cơ sở mỗi resource sử dụng method `parameters`. Mảng được truyền vào method `parameters` nên là một mảng associative của tên resources và tên parameters:

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

Ví dụ trên tạo URI sau cho route `show` của resource:

```text
/users/{admin_user}
```

<a name="restful-scoping-resource-routes"></a>
### Scoping Resource Routes

Tính năng [scoped implicit model binding](/docs/{{version}}/routing#implicit-model-binding-scoping) của Laravel có thể tự động scope nested bindings sao cho child model được giải quyết được xác nhận thuộc về parent model. Bằng cách sử dụng method `scoped` khi định nghĩa nested resource của bạn, bạn có thể bật automatic scoping cũng như hướng dẫn Laravel field nào child resource nên được truy xuất bằng:

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

Route này sẽ đăng ký một scoped nested resource có thể được truy cập với các URI như sau:

```text
/photos/{photo}/comments/{comment:slug}
```

Khi sử dụng một implicit binding với custom key làm route parameter lồng nhau, Laravel sẽ tự động scope query để truy xuất nested model bằng parent của nó sử dụng conventions để đoán tên relationship trên parent. Trong trường hợp này, sẽ được giả định rằng model `Photo` có một relationship tên là `comments` (số nhiều của tên route parameter) có thể được sử dụng để truy xuất model `Comment`.

<a name="restful-localizing-resource-uris"></a>
### Localizing Resource URIs

Theo mặc định, `Route::resource` sẽ tạo resource URIs sử dụng các verbs tiếng Anh và các quy tắc số nhiều. Nếu bạn cần localize các verbs action `create` và `edit`, bạn có thể sử dụng method `Route::resourceVerbs`. Điều này có thể được thực hiện ở đầu method `boot` trong `App\Providers\AppServiceProvider` của ứng dụng:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

Pluralizer của Laravel hỗ trợ [nhiều ngôn ngữ khác nhau mà bạn có thể cấu hình dựa trên nhu cầu của bạn](/docs/{{version}}/localization#pluralization-language). Khi các verbs và ngôn ngữ pluralization đã được tùy chỉnh, một đăng ký resource route như `Route::resource('publicacion', PublicacionController::class)` sẽ tạo ra các URI sau:

```text
/publicacion/crear

/publicacion/{publicaciones}/editar
```

<a name="restful-supplementing-resource-controllers"></a>
### Supplementing Resource Controllers

Nếu bạn cần thêm các routes bổ sung vào một resource controller ngoài bộ mặc định của resource routes, bạn nên định nghĩa các routes đó trước khi gọi method `Route::resource`; nếu không, các routes được định nghĩa bởi method `resource` có thể vô tình chiếm ưu thế hơn các supplemental routes của bạn:

```php
use App\Http\Controller\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

> [!NOTE]
> Hãy nhớ giữ cho các controllers của bạn tập trung. Nếu bạn thấy mình thường xuyên cần các methods ngoài bộ điển hình của resource actions, hãy cân nhắc chia controller của bạn thành hai, controllers nhỏ hơn.

<a name="singleton-resource-controllers"></a>
### Singleton Resource Controllers

Đôi khi, ứng dụng của bạn sẽ có các resources có thể chỉ có một instance duy nhất. Ví dụ, "profile" của người dùng có thể được chỉnh sửa hoặc cập nhật, nhưng người dùng có thể không có nhiều hơn một "profile". Tương tự, một hình ảnh có thể có một "thumbnail" duy nhất. Các resources này được gọi là "singleton resources", nghĩa là chỉ có một và chỉ một instance của resource có thể tồn tại. Trong các trường hợp này, bạn có thể đăng ký một resource controller "singleton":

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
```

Định nghĩa singleton resource ở trên sẽ đăng ký các routes sau. Như bạn có thể thấy, các routes "creation" không được đăng ký cho singleton resources, và các routes được đăng ký không chấp nhận một identifier vì chỉ có một instance của resource có thể tồn tại:

<div class="overflow-auto">

| Verb      | URI             | Action | Route Name     |
| --------- | --------------- | ------ | -------------- |
| GET       | `/profile`      | show   | profile.show   |
| GET       | `/profile/edit` | edit   | profile.edit   |
| PUT/PATCH | `/profile`      | update | profile.update |

</div>

Singleton resources cũng có thể được lồng nhau trong một resource tiêu chuẩn:

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

Trong ví dụ này, resource `photos` sẽ nhận tất cả các [standard resource routes](#actions-handled-by-resource-controllers); tuy nhiên, resource `thumbnail` sẽ là một singleton resource với các routes sau:

<div class="overflow-auto">

| Verb      | URI                              | Action | Route Name              |
| --------- | -------------------------------- | ------ | ----------------------- |
| GET       | `/photos/{photo}/thumbnail`      | show   | photos.thumbnail.show   |
| GET       | `/photos/{photo}/thumbnail/edit` | edit   | photos.thumbnail.edit   |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update | photos.thumbnail.update |

</div>

<a name="creatable-singleton-resources"></a>
#### Creatable Singleton Resources

Thỉnh thoảng, bạn có thể muốn định nghĩa các routes creation và storage cho một singleton resource. Để thực hiện điều này, bạn có thể gọi method `creatable` khi đăng ký singleton resource route:

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

Trong ví dụ này, các routes sau sẽ được đăng ký. Như bạn có thể thấy, một route `DELETE` cũng sẽ được đăng ký cho creatable singleton resources:

<div class="overflow-auto">

| Verb      | URI                                | Action  | Route Name               |
| --------- | ---------------------------------- | ------- | ------------------------ |
| GET       | `/photos/{photo}/thumbnail/create` | create  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | store   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | update  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | destroy | photos.thumbnail.destroy |

</div>

Nếu bạn muốn Laravel đăng ký route `DELETE` cho một singleton resource nhưng không đăng ký các routes creation hoặc storage, bạn có thể sử dụng method `destroyable`:

```php
Route::singleton(...)->destroyable();
```

<a name="api-singleton-resources"></a>
#### API Singleton Resources

Method `apiSingleton` có thể được sử dụng để đăng ký một singleton resource sẽ được thao tác thông qua một API, do đó làm cho các routes `create` và `edit` không cần thiết:

```php
Route::apiSingleton('profile', ProfileController::class);
```

Tất nhiên, API singleton resources cũng có thể là `creatable`, sẽ đăng ký các routes `store` và `destroy` cho resource:

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```
<a name="middleware-and-resource-controllers"></a>
### Middleware and Resource Controllers

Laravel cho phép bạn gán middleware cho tất cả, hoặc chỉ các methods cụ thể, của resource routes sử dụng các methods `middleware`, `middlewareFor`, và `withoutMiddlewareFor`. Các methods này cung cấp kiểm soát chi tiết về middleware nào được áp dụng cho mỗi resource action.

#### Applying Middleware to all Methods

Bạn có thể sử dụng method `middleware` để gán middleware cho tất cả các routes được tạo bởi một resource hoặc singleton resource route:

```php
Route::resource('users', UserController::class)
    ->middleware(['auth', 'verified']);

Route::singleton('profile', ProfileController::class)
    ->middleware('auth');
```

#### Applying Middleware to Specific Methods

Bạn có thể sử dụng method `middlewareFor` để gán middleware cho một hoặc nhiều methods cụ thể của một resource controller đã cho:

```php
Route::resource('users', UserController::class)
    ->middlewareFor('show', 'auth');

Route::apiResource('users', UserController::class)
    ->middlewareFor(['show', 'update'], 'auth');

Route::resource('users', UserController::class)
    ->middlewareFor('show', 'auth')
    ->middlewareFor('update', 'auth');

Route::apiResource('users', UserController::class)
    ->middlewareFor(['show', 'update'], ['auth', 'verified']);
```

Method `middlewareFor` cũng có thể được sử dụng kết hợp với singleton và API singleton resource controllers:

```php
Route::singleton('profile', ProfileController::class)
    ->middlewareFor('show', 'auth');

Route::apiSingleton('profile', ProfileController::class)
    ->middlewareFor(['show', 'update'], 'auth');
```

#### Excluding Middleware from Specific Methods

Bạn có thể sử dụng method `withoutMiddlewareFor` để loại bỏ middleware khỏi các methods cụ thể của một resource controller:

```php
Route::middleware(['auth', 'verified', 'subscribed'])->group(function () {
    Route::resource('users', UserController::class)
        ->withoutMiddlewareFor('index', ['auth', 'verified'])
        ->withoutMiddlewareFor(['create', 'store'], 'verified')
        ->withoutMiddlewareFor('destroy', 'subscribed');
});
```

<a name="dependency-injection-and-controllers"></a>
## Dependency Injection and Controllers

<a name="constructor-injection"></a>
#### Constructor Injection

Laravel [service container](/docs/{{version}}/container) được sử dụng để giải quyết tất cả các Laravel controllers. Kết quả là, bạn có thể type-hint bất kỳ dependencies nào controller của bạn có thể cần trong constructor của nó. Các dependencies được khai báo sẽ tự động được giải quyết và inject vào controller instance:

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}
}
```

<a name="method-injection"></a>
#### Method Injection

Ngoài constructor injection, bạn cũng có thể type-hint dependencies trên các methods của controller. Một use-case phổ biến cho method injection là inject instance `Illuminate\Http\Request` vào các methods controller của bạn:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->name;

        // Store the user...

        return redirect('/users');
    }
}
```

Nếu method controller của bạn cũng mong đợi input từ một route parameter, hãy liệt kê các route arguments của bạn sau các dependencies khác của bạn. Ví dụ, nếu route của bạn được định nghĩa như sau:

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

Bạn vẫn có thể type-hint `Illuminate\Http\Request` và truy cập parameter `id` của bạn bằng cách định nghĩa method controller của bạn như sau:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the given user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...

        return redirect('/users');
    }
}
```

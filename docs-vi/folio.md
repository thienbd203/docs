# Laravel Folio

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Đường dẫn Trang / URI](#page-paths-uris)
    - [Định tuyến Subdomain](#subdomain-routing)
- [Tạo Route](#creating-routes)
    - [Route Lồng nhau](#nested-routes)
    - [Route Index](#index-routes)
- [Tham số Route](#route-parameters)
- [Ràng buộc Model Route](#route-model-binding)
    - [Model Đã Xóa Mềm](#soft-deleted-models)
- [Render Hooks](#render-hooks)
- [Route Được Đặt Tên](#named-routes)
- [Middleware](#middleware)
- [Cache Route](#route-caching)

<a name="introduction"></a>
## Giới thiệu

[Laravel Folio](https://github.com/laravel/folio) là một router dựa trên trang mạnh mẽ được thiết kế để đơn giản hóa định tuyến trong các ứng dụng Laravel. Với Laravel Folio, việc tạo một route trở nên đơn giản như việc tạo một template Blade trong thư mục `resources/views/pages` của ứng dụng.

Ví dụ, để tạo một trang có thể truy cập tại URL `/greeting`, chỉ cần tạo một file `greeting.blade.php` trong thư mục `resources/views/pages` của ứng dụng:

```php
<div>
    Hello World
</div>
```

<a name="installation"></a>
## Cài đặt

Để bắt đầu, cài đặt Folio vào dự án của bạn bằng trình quản lý package Composer:

```shell
composer require laravel/folio
```

Sau khi cài đặt Folio, bạn có thể thực thi lệnh Artisan `folio:install`, lệnh này sẽ cài đặt service provider của Folio vào ứng dụng của bạn. Service provider này đăng ký thư mục mà Folio sẽ tìm kiếm route / trang:

```shell
php artisan folio:install
```

<a name="page-paths-uris"></a>
### Đường dẫn Trang / URI

Theo mặc định, Folio phục vụ các trang từ thư mục `resources/views/pages` của ứng dụng, nhưng bạn có thể tùy chỉnh các thư mục này trong phương thức `boot` của service provider Folio.

Ví dụ, đôi khi có thể thuận tiện để chỉ định nhiều đường dẫn Folio trong cùng một ứng dụng Laravel. Bạn có thể muốn có một thư mục riêng của các trang Folio cho khu vực "admin" của ứng dụng, trong khi sử dụng một thư mục khác cho các trang còn lại của ứng dụng.

Bạn có thể thực hiện điều này bằng cách sử dụng các phương thức `Folio::path` và `Folio::uri`. Phương thức `path` đăng ký một thư mục mà Folio sẽ quét tìm các trang khi định tuyến các yêu cầu HTTP đến, trong khi phương thức `uri` chỉ định "URI cơ sở" cho thư mục trang đó:

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages/guest'))->uri('/');

Folio::path(resource_path('views/pages/admin'))
    ->uri('/admin')
    ->middleware([
        '*' => [
            'auth',
            'verified',

            // ...
        ],
    ]);
```

<a name="subdomain-routing"></a>
### Định tuyến Subdomain

Bạn cũng có thể định tuyến đến các trang dựa trên subdomain của yêu cầu đến. Ví dụ, bạn có thể muốn định tuyến các yêu cầu từ `admin.example.com` đến một thư mục trang khác so với các trang Folio còn lại của bạn. Bạn có thể thực hiện điều này bằng cách gọi phương thức `domain` sau khi gọi phương thức `Folio::path`:

```php
use Laravel\Folio\Folio;

Folio::domain('admin.example.com')
    ->path(resource_path('views/pages/admin'));
```

Phương thức `domain` cũng cho phép bạn nắm bắt các phần của domain hoặc subdomain dưới dạng tham số. Các tham số này sẽ được tiêm vào template trang của bạn:

```php
use Laravel\Folio\Folio;

Folio::domain('{account}.example.com')
    ->path(resource_path('views/pages/admin'));
```

<a name="creating-routes"></a>
## Tạo Route

Bạn có thể tạo một route Folio bằng cách đặt một template Blade trong bất kỳ thư mục nào đã được mount bởi Folio. Theo mặc định, Folio mount thư mục `resources/views/pages`, nhưng bạn có thể tùy chỉnh các thư mục này trong phương thức `boot` của service provider Folio.

Khi một template Blade đã được đặt trong một thư mục được mount bởi Folio, bạn có thể truy cập nó ngay lập tức thông qua trình duyệt. Ví dụ, một trang được đặt trong `pages/schedule.blade.php` có thể được truy cập trong trình duyệt tại `http://example.com/schedule`.

Để nhanh chóng xem danh sách tất cả các trang / route Folio của bạn, bạn có thể gọi lệnh Artisan `folio:list`:

```shell
php artisan folio:list
```

<a name="nested-routes"></a>
### Route Lồng nhau

Bạn có thể tạo một route lồng nhau bằng cách tạo một hoặc nhiều thư mục trong một trong các thư mục của Folio. Ví dụ, để tạo một trang có thể truy cập thông qua `/user/profile`, hãy tạo một template `profile.blade.php` trong thư mục `pages/user`:

```shell
php artisan folio:page user/profile

# pages/user/profile.blade.php → /user/profile
```

<a name="index-routes"></a>
### Route Index

Đôi khi, bạn có thể muốn làm cho một trang nhất định trở thành "index" của một thư mục. Bằng cách đặt một template `index.blade.php` trong một thư mục Folio, bất kỳ yêu cầu nào đến gốc của thư mục đó sẽ được định tuyến đến trang đó:

```shell
php artisan folio:page index
# pages/index.blade.php → /

php artisan folio:page users/index
# pages/users/index.blade.php → /users
```

<a name="route-parameters"></a>
## Tham số Route

Thường xuyên, bạn sẽ cần có các phần của URL yêu cầu đến được tiêm vào trang của bạn để bạn có thể tương tác với chúng. Ví dụ, bạn có thể cần truy cập "ID" của người dùng có hồ sơ đang được hiển thị. Để thực hiện điều này, bạn có thể đóng gói một phần của tên file trang trong dấu ngoặc vuông:

```shell
php artisan folio:page "users/[id]"

# pages/users/[id].blade.php → /users/1
```

Các phần được nắm bắt có thể được truy cập dưới dạng biến trong template Blade của bạn:

```html
<div>
    User {{ $id }}
</div>
```

Để nắm bắt nhiều phần, bạn có thể thêm tiền tố ba dấu chấm `...` vào phần được đóng gói:

```shell
php artisan folio:page "users/[...ids]"

# pages/users/[...ids].blade.php → /users/1/2/3
```

Khi nắm bắt nhiều phần, các phần được nắm bắt sẽ được tiêm vào trang dưới dạng một mảng:

```html
<ul>
    @foreach ($ids as $id)
        <li>User {{ $id }}</li>
    @endforeach
</ul>
```

<a name="route-model-binding"></a>
## Ràng buộc Model Route

Nếu một phần wildcard của tên file template trang của bạn tương ứng với một trong các model Eloquent của ứng dụng, Folio sẽ tự động tận dụng các khả năng ràng buộc model route của Laravel và cố gắng tiêm instance model đã giải quyết vào trang của bạn:

```shell
php artisan folio:page "users/[User]"

# pages/users/[User].blade.php → /users/1
```

Các model được nắm bắt có thể được truy cập dưới dạng biến trong template Blade của bạn. Tên biến của model sẽ được chuyển đổi thành "camel case":

```html
<div>
    User {{ $user->id }}
</div>
```

#### Tùy chỉnh Khóa

Đôi khi bạn có thể muốn giải quyết các model Eloquent được ràng buộc bằng cách sử dụng một cột khác ngoài `id`. Để làm điều này, bạn có thể chỉ định cột trong tên file của trang. Ví dụ, một trang với tên file `[Post:slug].blade.php` sẽ cố gắng giải quyết model được ràng buộc thông qua cột `slug` thay vì cột `id`.

Trên Windows, bạn nên sử dụng `-` để phân tách tên model khỏi khóa: `[Post-slug].blade.php`.

#### Vị trí Model

Theo mặc định, Folio sẽ tìm kiếm model của bạn trong thư mục `app/Models` của ứng dụng. Tuy nhiên, nếu cần, bạn có thể chỉ định tên class model đầy đủ trong tên file template của bạn:

```shell
php artisan folio:page "users/[.App.Models.User]"

# pages/users/[.App.Models.User].blade.php → /users/1
```

<a name="soft-deleted-models"></a>
### Model Đã Xóa Mềm

Theo mặc định, các model đã bị xóa mềm không được truy xuất khi giải quyết các ràng buộc model ngầm định. Tuy nhiên, nếu bạn muốn, bạn có thể hướng dẫn Folio truy xuất các model đã xóa mềm bằng cách gọi hàm `withTrashed` trong template của trang:

```php
<?php

use function Laravel\Folio\{withTrashed};

withTrashed();

?>
<div>
    User {{ $user->id }}
</div>
```

<a name="render-hooks"></a>
## Render Hooks

Theo mặc định, Folio sẽ trả về nội dung của template Blade của trang dưới dạng phản hồi cho yêu cầu đến. Tuy nhiên, bạn có thể tùy chỉnh phản hồi bằng cách gọi hàm `render` trong template của trang.

Hàm `render` chấp nhận một closure sẽ nhận instance `View` đang được render bởi Folio, cho phép bạn thêm dữ liệu bổ sung vào view hoặc tùy chỉnh toàn bộ phản hồi. Ngoài việc nhận instance `View`, bất kỳ tham số route hoặc ràng buộc model bổ sung nào cũng sẽ được cung cấp cho closure `render`:

```php
<?php

use App\Models\Post;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

use function Laravel\Folio\render;

render(function (View $view, Post $post) {
    if (! Auth::user()->can('view', $post)) {
        return response('Unauthorized', 403);
    }

    return $view->with('photos', $post->author->photos);
}); ?>

<div>
    {{ $post->content }}
</div>

<div>
    This author has also taken {{ count($photos) }} photos.
</div>
```

<a name="named-routes"></a>
## Route Được Đặt Tên

Bạn có thể chỉ định một tên cho route của một trang nhất định bằng cách sử dụng hàm `name`:

```php
<?php

use function Laravel\Folio\name;

name('users.index');
```

Giống như các route được đặt tên của Laravel, bạn có thể sử dụng hàm `route` để tạo URL đến các trang Folio đã được gán tên:

```php
<a href="{{ route('users.index') }}">
    All Users
</a>
```

Nếu trang có tham số, bạn có thể chỉ cần chuyển các giá trị của chúng cho hàm `route`:

```php
route('users.show', ['user' => $user]);
```

<a name="middleware"></a>
## Middleware

Bạn có thể áp dụng middleware cho một trang cụ thể bằng cách gọi hàm `middleware` trong template của trang:

```php
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);

?>
<div>
    Dashboard
</div>
```

Hoặc, để gán middleware cho một nhóm trang, bạn có thể chuỗi phương thức `middleware` sau khi gọi phương thức `Folio::path`.

Để chỉ định các trang mà middleware nên được áp dụng, mảng middleware có thể được khóa bằng cách sử dụng các mẫu URL tương ứng của các trang mà chúng nên được áp dụng. Ký tự `*` có thể được sử dụng làm ký tự wildcard:

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        // ...
    ],
]);
```

Bạn có thể bao gồm các closure trong mảng middleware để định nghĩa middleware ẩn danh, inline:

```php
use Closure;
use Illuminate\Http\Request;
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        function (Request $request, Closure $next) {
            // ...

            return $next($request);
        },
    ],
]);
```

<a name="route-caching"></a>
## Cache Route

Khi sử dụng Folio, bạn nên luôn tận dụng [khả năng cache route của Laravel](/docs/{{version}}/routing#route-caching). Folio lắng nghe lệnh Artisan `route:cache` để đảm bảo rằng các định nghĩa trang Folio và tên route được cache đúng cách để đạt hiệu suất tối đa.

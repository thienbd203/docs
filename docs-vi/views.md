# Views

- [Giới thiệu](#introduction)
    - [Viết Views trong React / Svelte / Vue](#writing-views-in-react-svelte-or-vue)
- [Tạo và Render Views](#creating-and-rendering-views)
    - [Nested View Directories](#nested-view-directories)
    - [Tạo View Đầu Tiên Có Sẵn](#creating-the-first-available-view)
    - [Xác định xem View Có Tồn tại](#determining-if-a-view-exists)
- [Truyền Data đến Views](#passing-data-to-views)
    - [Chia sẻ Data Với Tất cả Views](#sharing-data-with-all-views)
- [View Composers](#view-composers)
    - [View Creators](#view-creators)
- [Tối ưu hóa Views](#optimizing-views)

<a name="introduction"></a>
## Giới thiệu

Tất nhiên, không thực tế để trả về toàn bộ HTML document strings trực tiếp từ routes và controllers của bạn. May mắn thay, views cung cấp một cách thuận tiện để đặt tất cả HTML của chúng ta trong các file riêng biệt.

Views tách biệt controller / application logic của bạn khỏi presentation logic và được lưu trữ trong thư mục `resources/views`. Khi sử dụng Laravel, view templates thường được viết sử dụng [Blade templating language](/docs/{{version}}/blade). Một view đơn giản có thể trông giống như sau:

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Vì view này được lưu trữ tại `resources/views/greeting.blade.php`, chúng ta có thể trả về nó sử dụng global `view` helper như sau:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

> [!NOTE]
> Tìm kiếm thêm thông tin về cách viết Blade templates? Hãy xem [Blade documentation](/docs/{{version}}/blade) đầy đủ để bắt đầu.

<a name="writing-views-in-react-svelte-or-vue"></a>
### Viết Views trong React / Svelte / Vue

Thay vì viết frontend templates của họ trong PHP thông qua Blade, nhiều developers đã bắt đầu thích viết templates của họ sử dụng React, Svelte, hoặc Vue. Laravel làm cho điều này không đau đớn nhờ [Inertia](https://inertiajs.com/), một thư viện làm cho việc kết nối React / Svelte / Vue frontend của bạn với Laravel backend của bạn trở nên dễ dàng mà không có các complexities điển hình của việc xây dựng một SPA.

[React, Svelte, và Vue application starter kits](/docs/{{version}}/starter-kits) của chúng tôi cung cấp cho bạn một starting point tuyệt vời cho ứng dụng Laravel tiếp theo của bạn được hỗ trợ bởi Inertia.

<a name="creating-and-rendering-views"></a>
## Tạo và Render Views

Bạn có thể tạo một view bằng cách đặt một file với extension `.blade.php` trong thư mục `resources/views` của ứng dụng của bạn hoặc sử dụng command Artisan `make:view`:

```shell
php artisan make:view greeting
```

Extension `.blade.php` thông báo cho framework rằng file chứa một [Blade template](/docs/{{version}}/blade). Blade templates chứa HTML cũng như các Blade directives cho phép bạn dễ dàng echo values, tạo "if" statements, iterate over data, và nhiều hơn nữa.

Khi bạn đã tạo một view, bạn có thể trả về nó từ một trong các routes hoặc controllers của ứng dụng của bạn sử dụng global `view` helper:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

Views cũng có thể được trả về sử dụng `View` facade:

```php
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```

Như bạn có thể thấy, đối số thứ nhất được truyền cho `view` helper tương ứng với tên của view file trong thư mục `resources/views`. Đối số thứ hai là một mảng data nên được có sẵn cho view. Trong trường hợp này, chúng ta đang truyền biến `name`, được hiển thị trong view sử dụng [Blade syntax](/docs/{{version}}/blade).

<a name="nested-view-directories"></a>
### Nested View Directories

Views cũng có thể được lồng trong các subdirectories của thư mục `resources/views`. "Dot" notation có thể được sử dụng để tham chiếu các nested views. Ví dụ, nếu view của bạn được lưu trữ tại `resources/views/admin/profile.blade.php`, bạn có thể trả về nó từ một trong các routes / controllers của ứng dụng của bạn như sau:

```php
return view('admin.profile', $data);
```

> [!WARNING]
> Tên view directory không nên chứa ký tự `.`.

<a name="creating-the-first-available-view"></a>
### Tạo View Đầu Tiên Có Sẵn

Sử dụng phương thức `first` của `View` facade, bạn có thể tạo view đầu tiên tồn tại trong một mảng views nhất định. Điều này có thể hữu ích nếu ứng dụng hoặc package của bạn cho phép views được customize hoặc overwritten:

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

<a name="determining-if-a-view-exists"></a>
### Xác định xem View Có Tồn tại

Nếu bạn cần xác định xem một view có tồn tại hay không, bạn có thể sử dụng `View` facade. Phương thức `exists` sẽ trả về `true` nếu view tồn tại:

```php
use Illuminate\Support\Facades\View;

if (View::exists('admin.profile')) {
    // ...
}
```

<a name="passing-data-to-views"></a>
## Truyền Data đến Views

Như bạn đã thấy trong các ví dụ trước, bạn có thể truyền một mảng data đến views để làm cho data đó có sẵn cho view:

```php
return view('greetings', ['name' => 'Victoria']);
```

Khi truyền thông tin theo cách này, data nên là một mảng với các cặp key / value. Sau khi cung cấp data cho một view, bạn có thể sau đó truy cập từng giá trị trong view của bạn sử dụng các keys của data, chẳng hạn như `<?php echo $name; ?>`.

Là một thay thế cho việc truyền một mảng data hoàn chỉnh cho hàm `view` helper, bạn có thể sử dụng phương thức `with` để thêm các phần data riêng lẻ đến view. Phương thức `with` trả về một instance của view object để bạn có thể tiếp tục chaining các phương thức trước khi trả về view:

```php
return view('greeting')
    ->with('name', 'Victoria')
    ->with('occupation', 'Astronaut');
```

<a name="sharing-data-with-all-views"></a>
### Chia sẻ Data Với Tất cả Views

Thỉnh thoảng, bạn có thể cần chia sẻ data với tất cả views được render bởi ứng dụng của bạn. Bạn có thể làm như vậy sử dụng phương thức `share` của `View` facade. Thông thường, bạn nên đặt các cuộc gọi đến phương thức `share` trong phương thức `boot` của một service provider. Bạn tự do thêm chúng vào class `App\Providers\AppServiceProvider` hoặc generate một service provider riêng để chứa chúng:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::share('key', 'value');
    }
}
```

<a name="view-composers"></a>
## View Composers

View composers là callbacks hoặc class methods được gọi khi một view được render. Nếu bạn có data mà bạn muốn được bound đến một view mỗi lần view đó được render, một view composer có thể giúp bạn tổ chức logic đó vào một location duy nhất. View composers có thể chứng minh đặc biệt hữu ích nếu cùng một view được trả về bởi nhiều routes hoặc controllers trong ứng dụng của bạn và luôn cần một particular piece of data.

Thông thường, view composers sẽ được registered trong một trong các [service providers](/docs/{{version}}/providers) của ứng dụng của bạn. Trong ví dụ này, chúng ta sẽ giả định rằng `App\Providers\AppServiceProvider` sẽ chứa logic này.

Chúng ta sẽ sử dụng phương thức `composer` của `View` facade để register view composer. Laravel không bao gồm một default directory cho class-based view composers, vì vậy bạn tự do tổ chức chúng theo cách bạn muốn. Ví dụ, bạn có thể tạo một thư mục `app/View/Composers` để chứa tất cả view composers của ứng dụng của bạn:

```php
<?php

namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades;
use Illuminate\Support\ServiceProvider;
use Illuminate\View\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // Using class-based composers...
        Facades\View::composer('profile', ProfileComposer::class);

        // Using closure-based composers...
        Facades\View::composer('welcome', function (View $view) {
            // ...
        });

        Facades\View::composer('dashboard', function (View $view) {
            // ...
        });
    }
}
```

Bây giờ chúng ta đã registered composer, phương thức `compose` của class `App\View\Composers\ProfileComposer` sẽ được thực thi mỗi lần view `profile` đang được render. Hãy xem một ví dụ về composer class:

```php
<?php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * Create a new profile composer.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * Bind data to the view.
     */
    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```

Như bạn có thể thấy, tất cả view composers được resolved thông qua [service container](/docs/{{version}}/container), vì vậy bạn có thể type-hint bất kỳ dependencies nào bạn cần trong constructor của một composer.

<a name="attaching-a-composer-to-multiple-views"></a>
#### Gắn một Composer đến Nhiều Views

Bạn có thể gắn một view composer đến nhiều views cùng một lúc bằng cách truyền một mảng views làm đối số thứ nhất cho phương thức `composer`:

```php
use App\Views\Composers\MultiComposer;
use Illuminate\Support\Facades\View;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

Phương thức `composer` cũng chấp nhận ký tự `*` như một wildcard, cho phép bạn gắn một composer đến tất cả views:

```php
use Illuminate\Support\Facades;
use Illuminate\View\View;

Facades\View::composer('*', function (View $view) {
    // ...
});
```

<a name="view-creators"></a>
### View Creators

View "creators" rất giống với view composers; tuy nhiên, chúng được thực thi ngay sau khi view được instantiated thay vì đợi cho đến khi view sắp render. Để register một view creator, sử dụng phương thức `creator`:

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

<a name="optimizing-views"></a>
## Tối ưu hóa Views

Theo mặc định, Blade template views được compiled on demand. Khi một request được thực thi render một view, Laravel sẽ xác định xem một compiled version của view có tồn tại hay không. Nếu file tồn tại, Laravel sẽ sau đó xác định xem uncompiled view đã được modified gần đây hơn compiled view hay không. Nếu compiled view không tồn tại, hoặc uncompiled view đã được modified, Laravel sẽ recompile view.

Compiling views trong request có thể có một negative impact nhỏ trên performance, vì vậy Laravel cung cấp command Artisan `view:cache` để precompile tất cả views được sử dụng bởi ứng dụng của bạn. Để tăng performance, bạn có thể muốn chạy command này như một phần của deployment process của bạn:

```shell
php artisan view:cache
```

Bạn có thể sử dụng command `view:clear` để clear view cache:

```shell
php artisan view:clear
```

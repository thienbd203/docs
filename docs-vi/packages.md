# Phát triển Package

- [Giới thiệu](#introduction)
    - [Lưu ý về Facades](#a-note-on-facades)
- [Khám phá Package](#package-discovery)
- [Service Providers](#service-providers)
- [Tài nguyên](#resources)
    - [Cấu hình](#configuration)
    - [Routes](#routes)
    - [Migrations](#migrations)
    - [File Ngôn ngữ](#language-files)
    - [Views](#views)
    - [View Components](#view-components)
    - [Lệnh Artisan "About"](#about-artisan-command)
- [Commands](#commands)
    - [Lệnh Tối ưu hóa](#optimize-commands)
    - [Lệnh Tải lại](#reload-commands)
- [Tài nguyên Công khai](#public-assets)
- [Xuất bản Nhóm File](#publishing-file-groups)

<a name="introduction"></a>
## Giới thiệu

Packages là cách chính để thêm chức năng vào Laravel. Packages có thể là bất cứ thứ gì từ một cách tuyệt vời để làm việc với ngày tháng như [Carbon](https://github.com/briannesbitt/Carbon) hoặc một package cho phép bạn liên kết file với các Eloquent model như Spatie's [Laravel Media Library](https://github.com/spatie/laravel-medialibrary).

Có nhiều loại package khác nhau. Một số package là độc lập, nghĩa là chúng hoạt động với bất kỳ framework PHP nào. Carbon và Pest là ví dụ về các package độc lập. Bất kỳ package nào trong số này đều có thể được sử dụng với Laravel bằng cách yêu cầu chúng trong file `composer.json` của bạn.

Mặt khác, một số package khác được dành riêng để sử dụng với Laravel. Các package này có thể có routes, controllers, views và cấu hình được dành riêng để cải thiện ứng dụng Laravel. Hướng dẫn này chủ yếu bao gồm việc phát triển các package dành riêng cho Laravel.

<a name="a-note-on-facades"></a>
### Lưu ý về Facades

Khi viết một ứng dụng Laravel, thường không quan trọng bạn sử dụng contracts hay facades vì cả hai đều cung cấp mức độ kiểm thử cơ bản giống nhau. Tuy nhiên, khi viết package, package của bạn thường sẽ không có quyền truy cập vào tất cả các helper kiểm thử của Laravel. Nếu bạn muốn có thể viết các kiểm thử cho package của mình như thể package được cài đặt bên trong một ứng dụng Laravel điển hình, bạn có thể sử dụng package [Orchestral Testbench](https://github.com/orchestral/testbench).

<a name="package-discovery"></a>
## Khám phá Package

File `bootstrap/providers.php` của ứng dụng Laravel chứa danh sách các service providers nên được tải bởi Laravel. Tuy nhiên, thay vì yêu cầu người dùng thêm thủ công service provider của bạn vào danh sách, bạn có thể định nghĩa provider trong phần `extra` của file `composer.json` của package để nó được tự động tải bởi Laravel. Ngoài service providers, bạn cũng có thể liệt kê bất kỳ [facades](/docs/{{version}}/facades) nào bạn muốn được đăng ký:

```json
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```

Khi package của bạn đã được cấu hình để khám phá, Laravel sẽ tự động đăng ký service providers và facades của nó khi được cài đặt, tạo ra trải nghiệm cài đặt thuận tiện cho người dùng package của bạn.

<a name="opting-out-of-package-discovery"></a>
#### Tắt Khám phá Package

Nếu bạn là người tiêu dùng của một package và muốn tắt khám phá package cho một package, bạn có thể liệt kê tên package trong phần `extra` của file `composer.json` của ứng dụng:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

Bạn có thể tắt khám phá package cho tất cả các package bằng cách sử dụng ký tự `*` bên trong chỉ thị `dont-discover` của ứng dụng:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

<a name="service-providers"></a>
## Service Providers

[Service providers](/docs/{{version}}/providers) là điểm kết nối giữa package của bạn và Laravel. Service provider chịu trách nhiệm ràng buộc các thứ vào [service container](/docs/{{version}}/container) của Laravel và thông báo cho Laravel biết nơi tải tài nguyên package như views, cấu hình và file ngôn ngữ.

Service provider mở rộng lớp `Illuminate\Support\ServiceProvider` và chứa hai phương thức: `register` và `boot`. Lớp `ServiceProvider` cơ bản nằm trong package Composer `illuminate/support`, mà bạn nên thêm vào các phụ thuộc của package riêng của mình. Để tìm hiểu thêm về cấu trúc và mục đích của service providers, hãy xem [tài liệu của họ](/docs/{{version}}/providers).

<a name="resources"></a>
## Tài nguyên

<a name="configuration"></a>
### Cấu hình

Thông thường, bạn sẽ cần xuất bản file cấu hình của package vào thư mục `config` của ứng dụng. Điều này sẽ cho phép người dùng của package dễ dàng ghi đè các tùy chọn cấu hình mặc định của bạn. Để cho phép các file cấu hình của bạn được xuất bản, gọi phương thức `publishes` từ phương thức `boot` của service provider:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/courier.php' => config_path('courier.php'),
    ]);
}
```

Bây giờ, khi người dùng của package thực hiện lệnh `vendor:publish` của Laravel, file của bạn sẽ được sao chép vào vị trí xuất bản được chỉ định. Khi cấu hình của bạn đã được xuất bản, các giá trị của nó có thể được truy cập như bất kỳ file cấu hình nào khác:

```php
$value = config('courier.option');
```

> [!WARNING]
> Bạn không nên định nghĩa closures trong các file cấu hình của mình. Chúng không thể được tuần tự hóa chính xác khi người dùng thực hiện lệnh Artisan `config:cache`.

<a name="default-package-configuration"></a>
#### Cấu hình Package Mặc định

Bạn cũng có thể hợp nhất file cấu hình package của riêng mình với bản sao đã xuất bản của ứng dụng. Điều này sẽ cho phép người dùng của bạn chỉ định nghĩa các tùy chọn mà họ thực sự muốn ghi đè trong bản sao đã xuất bản của file cấu hình. Để hợp nhất các giá trị file cấu hình, sử dụng phương thức `mergeConfigFrom` trong phương thức `register` của service provider.

Phương thức `mergeConfigFrom` chấp nhận đường dẫn đến file cấu hình của package làm đối số đầu tiên và tên bản sao file cấu hình của ứng dụng làm đối số thứ hai:

```php
/**
 * Register any package services.
 */
public function register(): void
{
    $this->mergeConfigFrom(
        __DIR__.'/../config/courier.php', 'courier'
    );
}
```

> [!WARNING]
> Phương thức này chỉ hợp nhất cấp đầu tiên của mảng cấu hình. Nếu người dùng của bạn định nghĩa một phần mảng cấu hình đa chiều, các tùy chọn bị thiếu sẽ không được hợp nhất.

<a name="routes"></a>
### Routes

Nếu package của bạn chứa routes, bạn có thể tải chúng bằng phương thức `loadRoutesFrom`. Phương thức này sẽ tự động xác định xem routes của ứng dụng có được cache hay không và sẽ không tải file routes của bạn nếu routes đã được cache:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
}
```

<a name="migrations"></a>
### Migrations

Nếu package của bạn chứa [database migrations](/docs/{{version}}/migrations), bạn có thể sử dụng phương thức `publishesMigrations` để thông báo cho Laravel rằng thư mục hoặc file đã cho chứa migrations. Khi Laravel xuất bản các migrations, nó sẽ tự động cập nhật timestamp trong tên file để phản ánh ngày và giờ hiện tại:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishesMigrations([
        __DIR__.'/../database/migrations' => database_path('migrations'),
    ]);
}
```

<a name="language-files"></a>
### File Ngôn ngữ

Nếu package của bạn chứa [language files](/docs/{{version}}/localization), bạn có thể sử dụng phương thức `loadTranslationsFrom` để thông báo cho Laravel cách tải chúng. Ví dụ, nếu package của bạn được đặt tên là `courier`, bạn nên thêm sau đây vào phương thức `boot` của service provider:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');
}
```

Các dòng dịch package được tham chiếu bằng quy tắc cú pháp `package::file.line`. Vì vậy, bạn có thể tải dòng `welcome` của package `courier` từ file `messages` như sau:

```php
echo trans('courier::messages.welcome');
```

Bạn có thể đăng ký các file dịch JSON cho package của mình bằng phương thức `loadJsonTranslationsFrom`. Phương thức này chấp nhận đường dẫn đến thư mục chứa các file dịch JSON của package:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadJsonTranslationsFrom(__DIR__.'/../lang');
}
```

<a name="publishing-language-files"></a>
#### Xuất bản File Ngôn ngữ

Nếu bạn muốn xuất bản các file ngôn ngữ của package vào thư mục `lang/vendor` của ứng dụng, bạn có thể sử dụng phương thức `publishes` của service provider. Phương thức `publishes` chấp nhận một mảng các đường dẫn package và vị trí xuất bản mong muốn của chúng. Ví dụ, để xuất bản các file ngôn ngữ cho package `courier`, bạn có thể làm như sau:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');

    $this->publishes([
        __DIR__.'/../lang' => $this->app->langPath('vendor/courier'),
    ]);
}
```

Bây giờ, khi người dùng của package thực hiện lệnh Artisan `vendor:publish` của Laravel, các file ngôn ngữ của package sẽ được xuất bản vào vị trí xuất bản được chỉ định.

<a name="views"></a>
### Views

Để đăng ký [views](/docs/{{version}}/views) của package với Laravel, bạn cần cho Laravel biết views nằm ở đâu. Bạn có thể làm điều này bằng phương thức `loadViewsFrom` của service provider. Phương thức `loadViewsFrom` chấp nhận hai đối số: đường dẫn đến các mẫu view của bạn và tên package của bạn. Ví dụ, nếu tên package của bạn là `courier`, bạn sẽ thêm sau đây vào phương thức `boot` của service provider:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');
}
```

Package views được tham chiếu bằng quy tắc cú pháp `package::view`. Vì vậy, khi đường dẫn view của bạn đã được đăng ký trong service provider, bạn có thể tải view `dashboard` từ package `courier` như sau:

```php
Route::get('/dashboard', function () {
    return view('courier::dashboard');
});
```

<a name="overriding-package-views"></a>
#### Ghi đè Package Views

Khi bạn sử dụng phương thức `loadViewsFrom`, Laravel thực sự đăng ký hai vị trí cho views của bạn: thư mục `resources/views/vendor` của ứng dụng và thư mục bạn chỉ định. Vì vậy, sử dụng package `courier` làm ví dụ, Laravel sẽ kiểm tra trước xem phiên bản tùy chỉnh của view đã được đặt trong thư mục `resources/views/vendor/courier` bởi nhà phát triển hay chưa. Sau đó, nếu view chưa được tùy chỉnh, Laravel sẽ tìm kiếm thư mục view package mà bạn đã chỉ định trong lệnh gọi `loadViewsFrom`. Điều này giúp người dùng package dễ dàng tùy chỉnh / ghi đè các view của package.

<a name="publishing-views"></a>
#### Xuất bản Views

Nếu bạn muốn làm cho views của mình có sẵn để xuất bản vào thư mục `resources/views/vendor` của ứng dụng, bạn có thể sử dụng phương thức `publishes` của service provider. Phương thức `publishes` chấp nhận một mảng các đường dẫn view package và vị trí xuất bản mong muốn của chúng:

```php
/**
 * Bootstrap the package services.
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');

    $this->publishes([
        __DIR__.'/../resources/views' => resource_path('views/vendor/courier'),
    ]);
}
```

Bây giờ, khi người dùng của package thực hiện lệnh Artisan `vendor:publish` của Laravel, các view của package sẽ được sao chép vào vị trí xuất bản được chỉ định.

<a name="view-components"></a>
### View Components

Nếu bạn đang xây dựng một package sử dụng các thành phần Blade hoặc đặt các thành phần trong các thư mục không theo quy ước, bạn sẽ cần đăng ký thủ công lớp thành phần và bí danh thẻ HTML của nó để Laravel biết nơi tìm thành phần. Bạn thường nên đăng ký các thành phần trong phương thức `boot` của service provider của package:

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

Khi thành phần của bạn đã được đăng ký, nó có thể được hiển thị bằng bí danh thẻ của nó:

```blade
<x-package-alert/>
```

<a name="autoloading-package-components"></a>
#### Tự động tải Package Components

Ngoài ra, bạn có thể sử dụng phương thức `componentNamespace` để tự động tải các lớp thành phần theo quy ước. Ví dụ, một package `Nightshade` có thể có các thành phần `Calendar` và `ColorPicker` nằm trong namespace `Nightshade\Views\Components`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

Điều này sẽ cho phép sử dụng các thành phần package theo namespace vendor của chúng bằng cú pháp `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade sẽ tự động phát hiện lớp được liên kết với thành phần này bằng cách pascal-casing tên thành phần. Các thư mục con cũng được hỗ trợ bằng cách sử dụng ký hiệu "dot".

<a name="anonymous-components"></a>
#### Anonymous Components

Nếu package của bạn chứa các thành phần ẩn danh, chúng phải được đặt trong thư mục `components` của thư mục "views" của package (như được chỉ định bởi [phương thức loadViewsFrom](#views)). Sau đó, bạn có thể hiển thị chúng bằng cách thêm tiền tố tên thành phần với namespace view của package:

```blade
<x-courier::alert />
```

<a name="about-artisan-command"></a>
### Lệnh Artisan "About"

Lệnh Artisan `about` tích hợp sẵn của Laravel cung cấp một tóm tắt về môi trường và cấu hình của ứng dụng. Packages có thể đẩy thêm thông tin vào đầu ra của lệnh này thông qua lớp `AboutCommand`. Thông thường, thông tin này có thể được thêm từ phương thức `boot` của service provider package:

```php
use Illuminate\Foundation\Console\AboutCommand;

/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    AboutCommand::add('My Package', fn () => ['Version' => '1.0.0']);
}
```

<a name="commands"></a>
## Commands

Để đăng ký các lệnh Artisan của package với Laravel, bạn có thể sử dụng phương thức `commands`. Phương thức này mong đợi một mảng tên lớp lệnh. Khi các lệnh đã được đăng ký, bạn có thể thực hiện chúng bằng cách sử dụng [Artisan CLI](/docs/{{version}}/artisan):

```php
use Courier\Console\Commands\InstallCommand;
use Courier\Console\Commands\NetworkCommand;

/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            InstallCommand::class,
            NetworkCommand::class,
        ]);
    }
}
```

<a name="optimize-commands"></a>
### Lệnh Tối ưu hóa

[lệnh optimize](/docs/{{version}}/deployment#optimization) của Laravel cache cấu hình, sự kiện, routes và views của ứng dụng. Sử dụng phương thức `optimizes`, bạn có thể đăng ký các lệnh Artisan riêng của package nên được gọi khi các lệnh `optimize` và `optimize:clear` được thực hiện:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->optimizes(
            optimize: 'package:optimize',
            clear: 'package:clear-optimizations',
        );
    }
}
```

<a name="reload-commands"></a>
### Lệnh Tải lại

[lệnh reload](/docs/{{version}}/deployment#reloading-services) của Laravel chấm dứt bất kỳ dịch vụ đang chạy nào để chúng có thể được tự động khởi động lại bởi một quy trình giám sát hệ thống. Sử dụng phương thức `reloads`, bạn có thể đăng ký các lệnh Artisan riêng của package nên được gọi khi lệnh `reload` được thực hiện:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->reloads('package:reload');
    }
}
```

<a name="public-assets"></a>
## Tài nguyên Công khai

Package của bạn có thể có tài nguyên như JavaScript, CSS và hình ảnh. Để xuất bản các tài nguyên này vào thư mục `public` của ứng dụng, sử dụng phương thức `publishes` của service provider. Trong ví dụ này, chúng tôi cũng sẽ thêm một thẻ nhóm tài nguyên `public`, có thể được sử dụng để dễ dàng xuất bản các nhóm tài nguyên liên quan:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../public' => public_path('vendor/courier'),
    ], 'public');
}
```

Bây giờ, khi người dùng của package thực hiện lệnh `vendor:publish`, các tài nguyên sẽ được sao chép vào vị trí xuất bản được chỉ định. Vì người dùng thường cần ghi đè các tài nguyên mỗi khi package được cập nhật, họ có thể sử dụng cờ `--force`:

```shell
php artisan vendor:publish --tag=public --force
```

<a name="publishing-file-groups"></a>
## Xuất bản Nhóm File

Bạn có thể muốn xuất bản các nhóm tài nguyên và tài nguyên package riêng biệt. Ví dụ, bạn có thể muốn cho phép người dùng xuất bản các file cấu hình của package mà không bị buộc phải xuất bản các tài nguyên của package. Bạn có thể làm điều này bằng cách "gắn thẻ" chúng khi gọi phương thức `publishes` từ service provider của package. Ví dụ, hãy sử dụng thẻ để định nghĩa hai nhóm xuất bản cho package `courier` (`courier-config` và `courier-migrations`) trong phương thức `boot` của service provider của package:

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'courier-config');

    $this->publishesMigrations([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'courier-migrations');
}
```

Bây giờ người dùng của bạn có thể xuất bản các nhóm này riêng biệt bằng cách tham chiếu thẻ của chúng khi thực hiện lệnh `vendor:publish`:

```shell
php artisan vendor:publish --tag=courier-config
```

Người dùng của bạn cũng có thể xuất bản tất cả các file có thể xuất bản được định nghĩa bởi service provider của package bằng cờ `--provider`:

```shell
php artisan vendor:publish --provider="Your\Package\ServiceProvider"
```

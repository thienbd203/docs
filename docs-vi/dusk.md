# Laravel Dusk

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
  - [Quản lý cài đặt ChromeDriver](#managing-chromedriver-installations)
  - [Sử dụng Trình duyệt Khác](#using-other-browsers)
- [Bắt đầu](#getting-started)
  - [Tạo Tests](#generating-tests)
  - [Đặt lại Database sau mỗi Test](#resetting-the-database-after-each-test)
  - [Chạy Tests](#running-tests)
  - [Xử lý Môi trường](#environment-handling)
- [Cơ bản về Trình duyệt](#browser-basics)
  - [Tạo Trình duyệt](#creating-browsers)
  - [Điều hướng](#navigation)
  - [Thay đổi kích thước Cửa sổ Trình duyệt](#resizing-browser-windows)
  - [Macros Trình duyệt](#browser-macros)
  - [Xác thực](#authentication)
  - [Cookies](#cookies)
  - [Thực thi JavaScript](#executing-javascript)
  - [Chụp ảnh màn hình](#taking-a-screenshot)
  - [Lưu Console Output ra Đĩa](#storing-console-output-to-disk)
  - [Lưu Page Source ra Đĩa](#storing-page-source-to-disk)
- [Tương tác với Các phần tử](#interacting-with-elements)
  - [Dusk Selectors](#dusk-selectors)
  - [Văn bản, Giá trị, và Thuộc tính](#text-values-and-attributes)
  - [Tương tác với Forms](#interacting-with-forms)
  - [Đính kèm Files](#attaching-files)
  - [Nhấn Buttons](#pressing-buttons)
  - [Click vào Links](#clicking-links)
  - [Sử dụng Bàn phím](#using-the-keyboard)
  - [Sử dụng Chuột](#using-the-mouse)
  - [Hộp thoại JavaScript](#javascript-dialogs)
  - [Tương tác với Inline Frames](#interacting-with-iframes)
  - [Phạm vi Selectors](#scoping-selectors)
  - [Chờ Các phần tử](#waiting-for-elements)
  - [Cuộn một phần tử vào View](#scrolling-an-element-into-view)
- [Các Assertions Có sẵn](#available-assertions)
- [Trang](#pages)
  - [Tạo Trang](#generating-pages)
  - [Cấu hình Trang](#configuring-pages)
  - [Điều hướng đến Trang](#navigating-to-pages)
  - [Selectors viết tắt](#shorthand-selectors)
  - [Phương thức Trang](#page-methods)
- [Components](#components)
  - [Tạo Components](#generating-components)
  - [Sử dụng Components](#using-components)
- [Tích hợp Liên tục](#continuous-integration)
  - [Heroku CI](#running-tests-on-heroku-ci)
  - [Travis CI](#running-tests-on-travis-ci)
  - [GitHub Actions](#running-tests-on-github-actions)
  - [Chipper CI](#running-tests-on-chipper-ci)

<a name="introduction"></a>

## Giới thiệu

> [!WARNING]
> [Pest 4](https://pestphp.com/) hiện đã bao gồm tính năng kiểm tra trình duyệt tự động hóa, mang lại cải thiện đáng kể về hiệu suất và khả năng sử dụng so với Laravel Dusk. Đối với các dự án mới, chúng tôi khuyên dùng Pest để kiểm tra trình duyệt.

[Laravel Dusk](https://github.com/laravel/dusk) cung cấp một API kiểm tra và tự động hóa trình duyệt biểu đạt, dễ sử dụng. Theo mặc định, Dusk không yêu cầu bạn cài đặt JDK hoặc Selenium trên máy tính cục bộ của mình. Thay vào đó, Dusk sử dụng cài đặt [ChromeDriver](https://sites.google.com/chromium.org/driver) độc lập. Tuy nhiên, bạn có thể tự do sử dụng bất kỳ driver tương thích với Selenium nào bạn muốn.

<a name="installation"></a>

## Cài đặt

Để bắt đầu, bạn nên cài đặt [Google Chrome](https://www.google.com/chrome) và thêm dependency Composer `laravel/dusk` vào dự án của mình:

```shell
composer require laravel/dusk --dev
```

> [!WARNING]
> Nếu bạn đăng ký thủ công service provider của Dusk, bạn **không bao giờ** nên đăng ký nó trong môi trường production, vì việc làm như vậy có thể dẫn đến việc người dùng tùy ý có thể xác thực với ứng dụng của bạn.

Sau khi cài đặt gói Dusk, hãy thực thi lệnh Artisan `dusk:install`. Lệnh `dusk:install` sẽ tạo thư mục `tests/Browser`, một test Dusk mẫu, và cài đặt binary Chrome Driver cho hệ điều hành của bạn:

```shell
php artisan dusk:install
```

Tiếp theo, hãy đặt biến môi trường `APP_URL` trong file `.env` của ứng dụng của bạn. Giá trị này phải khớp với URL bạn sử dụng để truy cập ứng dụng của mình trong trình duyệt.

> [!NOTE]
> Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail) để quản lý môi trường phát triển cục bộ của mình, vui lòng tham khảo tài liệu Sail về [cấu hình và chạy các test Dusk](/docs/{{version}}/sail#laravel-dusk).

<a name="managing-chromedriver-installations"></a>

### Quản lý cài đặt ChromeDriver

Nếu bạn muốn cài đặt phiên bản ChromeDriver khác với phiên bản được cài đặt bởi Laravel Dusk thông qua lệnh `dusk:install`, bạn có thể sử dụng lệnh `dusk:chrome-driver`:

```shell
# Cài đặt phiên bản mới nhất của ChromeDriver cho hệ điều hành của bạn...
php artisan dusk:chrome-driver

# Cài đặt một phiên bản cụ thể của ChromeDriver cho hệ điều hành của bạn...
php artisan dusk:chrome-driver 86

# Cài đặt một phiên bản cụ thể của ChromeDriver cho tất cả các hệ điều hành được hỗ trợ...
php artisan dusk:chrome-driver --all

# Cài đặt phiên bản ChromeDriver khớp với phiên bản Chrome / Chromium được phát hiện cho hệ điều hành của bạn...
php artisan dusk:chrome-driver --detect
```

> [!WARNING]
> Dusk yêu cầu các binary `chromedriver` phải có thể thực thi được. Nếu bạn gặp vấn đề khi chạy Dusk, bạn nên đảm bảo các binary có thể thực thi được bằng lệnh sau: `chmod -R 0755 vendor/laravel/dusk/bin/`.

<a name="using-other-browsers"></a>

### Sử dụng Trình duyệt Khác

Theo mặc định, Dusk sử dụng Google Chrome và cài đặt [ChromeDriver](https://sites.google.com/chromium.org/driver) độc lập để chạy các test trình duyệt của bạn. Tuy nhiên, bạn có thể khởi động server Selenium của riêng mình và chạy các test của mình trên bất kỳ trình duyệt nào bạn muốn.

Để bắt đầu, hãy mở file `tests/DuskTestCase.php` của bạn, đây là test case Dusk cơ bản cho ứng dụng của bạn. Trong file này, bạn có thể xóa lệnh gọi đến phương thức `startChromeDriver`. Điều này sẽ ngăn Dusk tự động khởi động ChromeDriver:

```php
/**
 * Chuẩn bị cho thực thi test Dusk.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

Tiếp theo, bạn có thể sửa đổi phương thức `driver` để kết nối với URL và port của sự lựa chọn của bạn. Ngoài ra, bạn có thể sửa đổi "desired capabilities" nên được truyền cho WebDriver:

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * Tạo instance RemoteWebDriver.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

<a name="getting-started"></a>

## Bắt đầu

<a name="generating-tests"></a>

### Tạo Tests

Để tạo một test Dusk, hãy sử dụng lệnh Artisan `dusk:make`. Test được tạo sẽ được đặt trong thư mục `tests/Browser`:

```shell
php artisan dusk:make LoginTest
```

<a name="resetting-the-database-after-each-test"></a>

### Đặt lại Database sau mỗi Test

Hầu hết các test bạn viết sẽ tương tác với các trang lấy dữ liệu từ database của ứng dụng của bạn; tuy nhiên, các test Dusk của bạn không bao giờ nên sử dụng trait `RefreshDatabase`. Trait `RefreshDatabase` tận dụng các giao dịch database sẽ không áp dụng hoặc có sẵn qua các yêu cầu HTTP. Thay vào đó, bạn có hai lựa chọn: trait `DatabaseMigrations` và trait `DatabaseTruncation`.

<a name="reset-migrations"></a>

#### Sử dụng Database Migrations

Trait `DatabaseMigrations` sẽ chạy các database migration của bạn trước mỗi test. Tuy nhiên, việc xóa và tạo lại các bảng database của bạn cho mỗi test thường chậm hơn việc cắt ngắn các bảng:

```php
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

//
```

```php
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    //
}
```

> [!WARNING]
> Database in-memory của SQLite không thể được sử dụng khi thực thi các test Dusk. Vì trình duyệt thực thi trong tiến trình riêng của nó, nó sẽ không thể truy cập các database in-memory của các tiến trình khác.

<a name="reset-truncation"></a>

#### Sử dụng Database Truncation

Trait `DatabaseTruncation` sẽ migrate database của bạn trong test đầu tiên để đảm bảo các bảng database của bạn đã được tạo đúng cách. Tuy nhiên, trong các test tiếp theo, các bảng của database sẽ chỉ được cắt ngắn - cung cấp tăng tốc độ so với việc chạy lại tất cả các database migration của bạn:

```php
<?php

use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;

pest()->use(DatabaseTruncation::class);

//
```

```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseTruncation;

    //
}
```

Theo mặc định, trait này sẽ cắt ngắn tất cả các bảng ngoại trừ bảng `migrations`. Nếu bạn muốn tùy chỉnh các bảng nên được cắt ngắn, bạn có thể định nghĩa một property `$tablesToTruncate` trên class test của mình:

> [!NOTE]
> Nếu bạn đang sử dụng Pest, bạn nên định nghĩa các property hoặc method trên class `DuskTestCase` cơ bản hoặc trên bất kỳ class nào mà file test của bạn mở rộng.

```php
/**
 * Chỉ định các bảng nên được cắt ngắn.
 *
 * @var array
 */
protected $tablesToTruncate = ['users'];
```

Ngoài ra, bạn có thể định nghĩa một property `$exceptTables` trên class test của mình để chỉ định các bảng nên được loại trừ khỏi việc cắt ngắn:

```php
/**
 * Chỉ định các bảng nên được loại trừ khỏi việc cắt ngắn.
 *
 * @var array
 */
protected $exceptTables = ['users'];
```

Để chỉ định các kết nối database nên có các bảng của chúng được cắt ngắn, bạn có thể định nghĩa một property `$connectionsToTruncate` trên class test của mình:

```php
/**
 * Chỉ định các kết nối nên có các bảng của chúng được cắt ngắn.
 *
 * @var array
 */
protected $connectionsToTruncate = ['mysql'];
```

Nếu bạn muốn thực thi code trước hoặc sau khi cắt ngắn database được thực hiện, bạn có thể định nghĩa các method `beforeTruncatingDatabase` hoặc `afterTruncatingDatabase` trên class test của mình:

```php
/**
 * Thực hiện bất kỳ công việc nào nên diễn ra trước khi database bắt đầu cắt ngắn.
 */
protected function beforeTruncatingDatabase(): void
{
    //
}

/**
 * Thực hiện bất kỳ công việc nào nên diễn ra sau khi database hoàn tất cắt ngắn.
 */
protected function afterTruncatingDatabase(): void
{
    //
}
```

<a name="running-tests"></a>

### Chạy Tests

Để chạy các test trình duyệt của bạn, hãy thực thi lệnh Artisan `dusk`:

```shell
php artisan dusk
```

Nếu bạn có các test thất bại lần cuối bạn chạy lệnh `dusk`, bạn có thể tiết kiệm thời gian bằng cách chạy lại các test thất bại trước tiên bằng lệnh `dusk:fails`:

```shell
php artisan dusk:fails
```

Lệnh `dusk` chấp nhận bất kỳ đối số nào thường được chấp nhận bởi trình chạy test Pest / PHPUnit, chẳng hạn như cho phép bạn chỉ chạy các test cho một [group](https://docs.phpunit.de/en/10.5/annotations.html#group) cụ thể:

```shell
php artisan dusk --group=foo
```

> [!NOTE]
> Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail) để quản lý môi trường phát triển cục bộ của mình, vui lòng tham khảo tài liệu Sail về [cấu hình và chạy các test Dusk](/docs/{{version}}/sail#laravel-dusk).

<a name="manually-starting-chromedriver"></a>

#### Khởi động ChromeDriver Thủ công

Theo mặc định, Dusk sẽ tự động cố gắng khởi động ChromeDriver. Nếu điều này không hoạt động cho hệ thống cụ thể của bạn, bạn có thể khởi động ChromeDriver thủ công trước khi chạy lệnh `dusk`. Nếu bạn chọn khởi động ChromeDriver thủ công, bạn nên comment dòng sau của file `tests/DuskTestCase.php` của mình:

```php
/**
 * Chuẩn bị cho thực thi test Dusk.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

Ngoài ra, nếu bạn khởi động ChromeDriver trên một port khác 9515, bạn nên sửa đổi phương thức `driver` của cùng class đó để phản ánh port đúng:

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * Tạo instance RemoteWebDriver.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:9515', DesiredCapabilities::chrome()
    );
}
```

<a name="environment-handling"></a>

### Xử lý Môi trường

Để buộc Dusk sử dụng file môi trường riêng của nó khi chạy các test, hãy tạo một file `.env.dusk.{environment}` trong gốc của dự án của bạn. Ví dụ, nếu bạn sẽ khởi động lệnh `dusk` từ môi trường `local` của mình, bạn nên tạo một file `.env.dusk.local`.

Khi chạy các test, Dusk sẽ sao lưu file `.env` của bạn và đổi tên môi trường Dusk của bạn thành `.env`. Sau khi các test đã hoàn tất, file `.env` của bạn sẽ được khôi phục.

<a name="browser-basics"></a>

## Cơ bản về Trình duyệt

<a name="creating-browsers"></a>

### Tạo Trình duyệt

Để bắt đầu, hãy viết một test xác minh rằng chúng ta có thể đăng nhập vào ứng dụng của mình. Sau khi tạo một test, chúng ta có thể sửa đổi nó để điều hướng đến trang đăng nhập, nhập một số thông tin xác thực, và nhấp vào nút "Login". Để tạo một instance trình duyệt, bạn có thể gọi phương thức `browse` từ trong test Dusk của mình:

```php
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

test('basic example', function () {
    $user = User::factory()->create([
        'email' => 'taylor@laravel.com',
    ]);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/home');
    });
});
```

```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    /**
     * Một ví dụ test trình duyệt cơ bản.
     */
    public function test_basic_example(): void
    {
        $user = User::factory()->create([
            'email' => 'taylor@laravel.com',
        ]);

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                ->type('email', $user->email)
                ->type('password', 'password')
                ->press('Login')
                ->assertPathIs('/home');
        });
    }
}
```

Như bạn có thể thấy trong ví dụ trên, phương thức `browse` chấp nhận một closure. Một instance trình duyệt sẽ tự động được truyền cho closure này bởi Dusk và là đối tượng chính được sử dụng để tương tác và thực hiện các assertions đối với ứng dụng của bạn.

<a name="creating-multiple-browsers"></a>

#### Tạo Nhiều Trình duyệt

Đôi khi bạn có thể cần nhiều trình duyệt để thực hiện đúng một test. Ví dụ, nhiều trình duyệt có thể cần thiết để kiểm tra màn hình chat tương tác với websockets. Để tạo nhiều trình duyệt, chỉ cần thêm nhiều đối số trình duyệt hơn vào chữ ký của closure được đưa cho phương thức `browse`:

```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))
        ->visit('/home')
        ->waitForText('Message');

    $second->loginAs(User::find(2))
        ->visit('/home')
        ->waitForText('Message')
        ->type('message', 'Hey Taylor')
        ->press('Send');

    $first->waitForText('Hey Taylor')
        ->assertSee('Jeffrey Way');
});
```

<a name="navigation"></a>

### Điều hướng

Phương thức `visit` có thể được sử dụng để điều hướng đến một URI cụ thể trong ứng dụng của bạn:

```php
$browser->visit('/login');
```

Bạn có thể sử dụng phương thức `visitRoute` để điều hướng đến một [named route](/docs/{{version}}/routing#named-routes):

```php
$browser->visitRoute($routeName, $parameters);
```

Bạn có thể điều hướng "back" và "forward" bằng các phương thức `back` và `forward`:

```php
$browser->back();

$browser->forward();
```

Bạn có thể sử dụng phương thức `refresh` để làm mới trang:

```php
$browser->refresh();
```

<a name="resizing-browser-windows"></a>

### Thay đổi kích thước Cửa sổ Trình duyệt

Bạn có thể sử dụng phương thức `resize` để điều chỉnh kích thước cửa sổ trình duyệt:

```php
$browser->resize(1920, 1080);
```

Phương thức `maximize` có thể được sử dụng để phóng to cửa sổ trình duyệt:

```php
$browser->maximize();
```

Phương thức `fitContent` sẽ thay đổi kích thước cửa sổ trình duyệt để khớp với kích thước nội dung của nó:

```php
$browser->fitContent();
```

Khi một test thất bại, Dusk sẽ tự động thay đổi kích thước trình duyệt để vừa với nội dung trước khi chụp ảnh màn hình. Bạn có thể vô hiệu hóa tính năng này bằng cách gọi phương thức `disableFitOnFailure` trong test của mình:

```php
$browser->disableFitOnFailure();
```

Bạn có thể sử dụng phương thức `move` để di chuyển cửa sổ trình duyệt đến một vị trí khác trên màn hình của mình:

```php
$browser->move($x = 100, $y = 100);
```

<a name="browser-macros"></a>

### Macros Trình duyệt

Nếu bạn muốn định nghĩa một phương thức trình duyệt tùy chỉnh mà bạn có thể tái sử dụng trong nhiều test khác nhau, bạn có thể sử dụng phương thức `macro` trên class `Browser`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của một [service provider](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Browser;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Đăng ký macros trình duyệt của Dusk.
     */
    public function boot(): void
    {
        Browser::macro('scrollToElement', function (string $element = null) {
            $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

            return $this;
        });
    }
}
```

Hàm `macro` chấp nhận một tên làm đối số đầu tiên, và một closure làm đối số thứ hai. Closure của macro sẽ được thực thi khi gọi macro như một phương thức trên một instance `Browser`:

```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/pay')
        ->scrollToElement('#credit-card-details')
        ->assertSee('Enter Credit Card Details');
});
```

<a name="authentication"></a>

### Xác thực

Thường xuyên, bạn sẽ kiểm tra các trang yêu cầu xác thực. Bạn có thể sử dụng phương thức `loginAs` của Dusk để tránh tương tác với màn hình đăng nhập của ứng dụng trong mỗi test. Phương thức `loginAs` chấp nhận một khóa chính liên kết với model có thể xác thực của bạn hoặc một instance model có thể xác thực:

```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
        ->visit('/home');
});
```

> [!WARNING]
> Sau khi sử dụng phương thức `loginAs`, session người dùng sẽ được duy trì cho tất cả các test trong file.

<a name="cookies"></a>

### Cookies

Bạn có thể sử dụng phương thức `cookie` để lấy hoặc đặt giá trị của một cookie được mã hóa. Theo mặc định, tất cả các cookie được tạo bởi Laravel đều được mã hóa:

```php
$browser->cookie('name');

$browser->cookie('name', 'Taylor');
```

Bạn có thể sử dụng phương thức `plainCookie` để lấy hoặc đặt giá trị của một cookie không được mã hóa:

```php
$browser->plainCookie('name');

$browser->plainCookie('name', 'Taylor');
```

Bạn có thể sử dụng phương thức `deleteCookie` để xóa cookie đã cho:

```php
$browser->deleteCookie('name');
```

<a name="executing-javascript"></a>

### Thực thi JavaScript

Bạn có thể sử dụng phương thức `script` để thực thi các câu lệnh JavaScript tùy ý trong trình duyệt:

```php
$browser->script('document.documentElement.scrollTop = 0');

$browser->script([
    'document.body.scrollTop = 0',
    'document.documentElement.scrollTop = 0',
]);

$output = $browser->script('return window.location.pathname');
```

<a name="taking-a-screenshot"></a>

### Chụp ảnh màn hình

Bạn có thể sử dụng phương thức `screenshot` để chụp ảnh màn hình và lưu nó với tên file đã cho. Tất cả các ảnh chụp màn hình sẽ được lưu trong thư mục `tests/Browser/screenshots`:

```php
$browser->screenshot('filename');
```

Phương thức `responsiveScreenshots` có thể được sử dụng để chụp một loạt ảnh màn hình tại các điểm ngắt khác nhau:

```php
$browser->responsiveScreenshots('filename');
```

Phương thức `screenshotElement` có thể được sử dụng để chụp ảnh màn hình của một phần tử cụ thể trên trang:

```php
$browser->screenshotElement('#selector', 'filename');
```

<a name="storing-console-output-to-disk"></a>

### Lưu Console Output ra Đĩa

Bạn có thể sử dụng phương thức `storeConsoleLog` để ghi console output của trình duyệt hiện tại ra đĩa với tên file đã cho. Console output sẽ được lưu trong thư mục `tests/Browser/console`:

```php
$browser->storeConsoleLog('filename');
```

<a name="storing-page-source-to-disk"></a>

### Lưu Page Source ra Đĩa

Bạn có thể sử dụng phương thức `storeSource` để ghi page source hiện tại ra đĩa với tên file đã cho. Page source sẽ được lưu trong thư mục `tests/Browser/source`:

```php
$browser->storeSource('filename');
```

<a name="interacting-with-elements"></a>

## Tương tác với Các phần tử

<a name="dusk-selectors"></a>

### Dusk Selectors

Việc chọn các CSS selector tốt để tương tác với các phần tử là một trong những phần khó nhất khi viết các test Dusk. Theo thời gian, các thay đổi frontend có thể gây ra các CSS selector như sau làm hỏng các test của bạn:

```html
// HTML...

<button>Login</button>
```

```php
// Test...

$browser->click('.login-page .container div > button');
```

Dusk selectors cho phép bạn tập trung vào việc viết các test hiệu quả thay vì nhớ các CSS selector. Để định nghĩa một selector, hãy thêm một thuộc tính `dusk` vào phần tử HTML của bạn. Sau đó, khi tương tác với một trình duyệt Dusk, hãy thêm tiền tố selector bằng `@` để thao tác phần tử đính kèm trong test của bạn:

```html
// HTML...

<button dusk="login-button">Login</button>
```

```php
// Test...

$browser->click('@login-button');
```

Nếu muốn, bạn có thể tùy chỉnh thuộc tính HTML mà Dusk selector sử dụng thông qua phương thức `selectorHtmlAttribute`. Thông thường, phương thức này nên được gọi từ phương thức `boot` của `AppServiceProvider` của ứng dụng của bạn:

```php
use Laravel\Dusk\Dusk;

Dusk::selectorHtmlAttribute('data-dusk');
```

<a name="text-values-and-attributes"></a>

### Văn bản, Giá trị, và Thuộc tính

<a name="retrieving-setting-values"></a>

#### Lấy và Đặt Giá trị

Dusk cung cấp một số phương thức để tương tác với giá trị hiện tại, văn bản hiển thị, và các thuộc tính của các phần tử trên trang. Ví dụ, để lấy "value" của một phần tử khớp với một CSS hoặc Dusk selector cụ thể, hãy sử dụng phương thức `value`:

```php
// Lấy giá trị...
$value = $browser->value('selector');

// Đặt giá trị...
$browser->value('selector', 'value');
```

Bạn có thể sử dụng phương thức `inputValue` để lấy "value" của một phần tử input có một tên field cụ thể:

````php
$value = $browser->inputValue('field');
```<a name="retrieving-text"></a>
#### Lấy Văn Bản

Phương thức `text` có thể được sử dụng để lấy văn bản hiển thị của một phần tử khớp với selector đã cho:

```php
$text = $browser->text('selector');
````

<a name="retrieving-attributes"></a>

#### Lấy Thuộc Tính

Cuối cùng, phương thức `attribute` có thể được sử dụng để lấy giá trị của một thuộc tính của phần tử khớp với selector đã cho:

```php
$attribute = $browser->attribute('selector', 'value');
```

<a name="interacting-with-forms"></a>

### Tương Tác Với Form

<a name="typing-values"></a>

#### Nhập Giá Trị

Dusk cung cấp nhiều phương thức để tương tác với form và các phần tử input. Trước tiên, hãy xem một ví dụ về nhập văn bản vào một trường input:

```php
$browser->type('email', 'taylor@laravel.com');
```

Lưu ý rằng, mặc dù phương thức chấp nhận một nếu cần thiết, chúng ta không bắt buộc phải truyền CSS selector vào phương thức `type`. Nếu không cung cấp CSS selector, Dusk sẽ tìm kiếm trường `input` hoặc `textarea` với thuộc tính `name` đã cho.

Để thêm văn bản vào một trường mà không xóa nội dung của nó, bạn có thể sử dụng phương thức `append`:

```php
$browser->type('tags', 'foo')
    ->append('tags', ', bar, baz');
```

Bạn có thể xóa giá trị của một input bằng phương thức `clear`:

```php
$browser->clear('email');
```

Bạn có thể hướng dẫn Dusk nhập chậm bằng phương thức `typeSlowly`. Theo mặc định, Dusk sẽ tạm dừng 100 mili-giây giữa các lần nhấn phím. Để tùy chỉnh khoảng thời gian giữa các lần nhấn phím, bạn có thể truyền số mili-giây thích hợp làm đối số thứ ba cho phương thức:

```php
$browser->typeSlowly('mobile', '+1 (202) 555-5555');

$browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);
```

Bạn có thể sử dụng phương thức `appendSlowly` để thêm văn bản chậm:

```php
$browser->type('tags', 'foo')
    ->appendSlowly('tags', ', bar, baz');
```

<a name="dropdowns"></a>

#### Dropdown

Để chọn một giá trị có sẵn trên phần tử `select`, bạn có thể sử dụng phương thức `select`. Giống như phương thức `type`, phương thức `select` không yêu cầu CSS selector đầy đủ. Khi truyền giá trị cho phương thức `select`, bạn nên truyền giá trị option cơ bản thay vì văn bản hiển thị:

```php
$browser->select('size', 'Large');
```

Bạn có thể chọn một tùy chọn ngẫu nhiên bằng cách bỏ qua đối số thứ hai:

```php
$browser->select('size');
```

Bằng cách cung cấp một mảng làm đối số thứ hai cho phương thức `select`, bạn có thể hướng dẫn phương thức chọn nhiều tùy chọn:

```php
$browser->select('categories', ['Art', 'Music']);
```

<a name="checkboxes"></a>

#### Checkboxes

Để "check" một checkbox input, bạn có thể sử dụng phương thức `check`. Giống như nhiều phương thức liên quan đến input khác, CSS selector đầy đủ không được yêu cầu. Nếu không tìm thấy khớp CSS selector, Dusk sẽ tìm kiếm checkbox với thuộc tính `name` khớp:

```php
$browser->check('terms');
```

Phương thức `uncheck` có thể được sử dụng để "uncheck" một checkbox input:

```php
$browser->uncheck('terms');
```

<a name="radio-buttons"></a>

#### Radio Buttons

Để "select" một tùy chọn input `radio`, bạn có thể sử dụng phương thức `radio`. Giống như nhiều phương thức liên quan đến input khác, CSS selector đầy đủ không được yêu cầu. Nếu không tìm thấy khớp CSS selector, Dusk sẽ tìm kiếm input `radio` với các thuộc tính `name` và `value` khớp:

```php
$browser->radio('size', 'large');
```

<a name="attaching-files"></a>

### Đính Kèm File

Phương thức `attach` có thể được sử dụng để đính kèm một file vào phần tử input `file`. Giống như nhiều phương thức liên quan đến input khác, CSS selector đầy đủ không được yêu cầu. Nếu không tìm thấy khớp CSS selector, Dusk sẽ tìm kiếm input `file` với thuộc tính `name` khớp:

```php
$browser->attach('photo', __DIR__.'/photos/mountains.png');
```

> [!WARNING]
> Hàm attach yêu cầu extension PHP `Zip` phải được cài đặt và bật trên server của bạn.

<a name="pressing-buttons"></a>

### Nhấn Nút

Phương thức `press` có thể được sử dụng để nhấp vào một phần tử button trên trang. Đối số được cung cấp cho phương thức `press` có thể là văn bản hiển thị của button hoặc CSS / Dusk selector:

```php
$browser->press('Login');
```

Khi gửi form, nhiều ứng dụng vô hiệu hóa nút gửi form sau khi nó được nhấn và sau đó bật lại nút khi yêu cầu HTTP của việc gửi form hoàn tất. Để nhấn một nút và đợi nút được bật lại, bạn có thể sử dụng phương thức `pressAndWaitFor`:

```php
// Nhấn nút và đợi tối đa 5 giây để nó được bật...
$browser->pressAndWaitFor('Save');

// Nhấn nút và đợi tối đa 1 giây để nó được bật...
$browser->pressAndWaitFor('Save', 1);
```

<a name="clicking-links"></a>

### Nhấp vào Liên Kết

Để nhấp vào một liên kết, bạn có thể sử dụng phương thức `clickLink` trên thể hiện browser. Phương thức `clickLink` sẽ nhấp vào liên kết có văn bản hiển thị đã cho:

```php
$browser->clickLink($linkText);
```

Bạn có thể sử dụng phương thức `seeLink` để xác định xem một liên kết có văn bản hiển thị đã cho có hiển thị trên trang hay không:

```php
if ($browser->seeLink($linkText)) {
    // ...
}
```

> [!WARNING]
> Các phương thức này tương tác với jQuery. Nếu jQuery không có sẵn trên trang, Dusk sẽ tự động đưa nó vào trang để nó có sẵn trong suốt thời gian của test.

<a name="using-the-keyboard"></a>

### Sử Dụng Bàn Phím

Phương thức `keys` cho phép bạn cung cấp các chuỗi input phức tạp hơn cho một phần tử đã cho so với phương thức `type` thường cho phép. Ví dụ, bạn có thể hướng dẫn Dusk giữ các phím modifier trong khi nhập giá trị. Trong ví dụ này, phím `shift` sẽ được giữ trong khi `taylor` được nhập vào phần tử khớp với selector đã cho. Sau khi `taylor` được nhập, `swift` sẽ được nhập mà không có bất kỳ phím modifier nào:

```php
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');
```

Một trường hợp sử dụng có giá trị khác của phương thức `keys` là gửi kết hợp "phím tắt bàn phím" cho CSS selector chính của ứng dụng của bạn:

```php
$browser->keys('.app', ['{command}', 'j']);
```

> [!NOTE]
> Tất cả các phím modifier như `{command}` đều được bao bọc trong các ký tự `{}`, và khớp với các hằng số được định nghĩa trong lớp `Facebook\WebDriver\WebDriverKeys`, có thể được [tìm thấy trên GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php).

<a name="fluent-keyboard-interactions"></a>

#### Tương Tác Bàn Phím Fluently

Dusk cũng cung cấp phương thức `withKeyboard`, cho phép bạn thực hiện các tương tác bàn phím phức tạp một cách fluently thông qua lớp `Laravel\Dusk\Keyboard`. Lớp `Keyboard` cung cấp các phương thức `press`, `release`, `type`, và `pause`:

```php
use Laravel\Dusk\Keyboard;

$browser->withKeyboard(function (Keyboard $keyboard) {
    $keyboard->press('c')
        ->pause(1000)
        ->release('c')
        ->type(['c', 'e', 'o']);
});
```

<a name="keyboard-macros"></a>

#### Keyboard Macros

Nếu bạn muốn định nghĩa các tương tác bàn phím tùy chỉnh mà bạn có thể dễ dàng tái sử dụng trong suốt bộ test của mình, bạn có thể sử dụng phương thức `macro` được cung cấp bởi lớp `Keyboard`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của một [service provider](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Facebook\WebDriver\WebDriverKeys;
use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Keyboard;
use Laravel\Dusk\OperatingSystem;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Register Dusk's browser macros.
     */
    public function boot(): void
    {
        Keyboard::macro('copy', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'c',
            ]);

            return $this;
        });

        Keyboard::macro('paste', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'v',
            ]);

            return $this;
        });
    }
}
```

Hàm `macro` chấp nhận một tên làm đối số đầu tiên và một closure làm đối số thứ hai. Closure của macro sẽ được thực thi khi gọi macro như một phương thức trên một thể hiện `Keyboard`:

```php
$browser->click('@textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->copy())
    ->click('@another-textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->paste());
```

<a name="using-the-mouse"></a>

### Sử Dụng Chuột

<a name="clicking-on-elements"></a>

#### Nhấp vào Các Phần Tử

Phương thức `click` có thể được sử dụng để nhấp vào một phần tử khớp với CSS hoặc Dusk selector đã cho:

```php
$browser->click('.selector');
```

Phương thức `clickAtXPath` có thể được sử dụng để nhấp vào một phần tử khớp với biểu thức XPath đã cho:

```php
$browser->clickAtXPath('//div[@class = "selector"]');
```

Phương thức `clickAtPoint` có thể được sử dụng để nhấp vào phần tử trên cùng tại một cặp tọa độ đã cho tương đối với vùng hiển thị của trình duyệt:

```php
$browser->clickAtPoint($x = 0, $y = 0);
```

Phương thức `doubleClick` có thể được sử dụng để mô phỏng nhấp đúp chuột:

```php
$browser->doubleClick();

$browser->doubleClick('.selector');
```

Phương thức `rightClick` có thể được sử dụng để mô phỏng nhấp chuột phải:

```php
$browser->rightClick();

$browser->rightClick('.selector');
```

Phương thức `clickAndHold` có thể được sử dụng để mô phỏng một nút chuột được nhấp và giữ. Một lệnh gọi tiếp theo đến phương thức `releaseMouse` sẽ hoàn tác hành vi này và giải phóng nút chuột:

```php
$browser->clickAndHold('.selector');

$browser->clickAndHold()
    ->pause(1000)
    ->releaseMouse();
```

Phương thức `controlClick` có thể được sử dụng để mô phỏng sự kiện `ctrl+click` trong trình duyệt:

```php
$browser->controlClick();

$browser->controlClick('.selector');
```

Phương thức `clickWhenVisible` hoặc `clickWhenEnabled` có thể được sử dụng để đợi một phần tử sẵn sàng trước khi nhấp vào nó chính xác một lần:

```php
$browser->clickWhenVisible('@save-button');
$browser->clickWhenEnabled('@submit-button');
```

<a name="mouseover"></a>

#### Mouseover

Phương thức `mouseover` có thể được sử dụng khi bạn cần di chuyển chuột qua một phần tử khớp với CSS hoặc Dusk selector đã cho:

```php
$browser->mouseover('.selector');
```

<a name="drag-drop"></a>

#### Kéo và Thả

Phương thức `drag` có thể được sử dụng để kéo một phần tử khớp với selector đã cho sang một phần tử khác:

```php
$browser->drag('.from-selector', '.to-selector');
```

Hoặc, bạn có thể kéo một phần tử theo một hướng duy nhất:

```php
$browser->dragLeft('.selector', $pixels = 10);
$browser->dragRight('.selector', $pixels = 10);
$browser->dragUp('.selector', $pixels = 10);
$browser->dragDown('.selector', $pixels = 10);
```

Cuối cùng, bạn có thể kéo một phần tử theo một offset đã cho:

```php
$browser->dragOffset('.selector', $x = 10, $y = 10);
```

<a name="javascript-dialogs"></a>

### JavaScript Dialogs

Dusk cung cấp nhiều phương thức để tương tác với JavaScript Dialogs. Ví dụ, bạn có thể sử dụng phương thức `waitForDialog` để đợi một JavaScript dialog xuất hiện. Phương thức này chấp nhận một đối số tùy chọn cho biết số giây cần đợi để dialog xuất hiện:

```php
$browser->waitForDialog($seconds = null);
```

Phương thức `assertDialogOpened` có thể được sử dụng để xác nhận rằng một dialog đã được hiển thị và chứa thông điệp đã cho:

```php
$browser->assertDialogOpened('Dialog message');
```

Nếu JavaScript dialog chứa một prompt, bạn có thể sử dụng phương thức `typeInDialog` để nhập một giá trị vào prompt:

```php
$browser->typeInDialog('Hello World');
```

Để đóng một JavaScript dialog đang mở bằng cách nhấp vào nút "OK", bạn có thể gọi phương thức `acceptDialog`:

```php
$browser->acceptDialog();
```

Để đóng một JavaScript dialog đang mở bằng cách nhấp vào nút "Cancel", bạn có thể gọi phương thức `dismissDialog`:

```php
$browser->dismissDialog();
```

<a name="interacting-with-iframes"></a>

### Tương Tác Với Inline Frames

Nếu bạn cần tương tác với các phần tử trong một iframe, bạn có thể sử dụng phương thức `withinFrame`. Tất cả các tương tác phần tử diễn ra trong closure được cung cấp cho phương thức `withinFrame` sẽ được giới hạn trong ngữ cảnh của iframe được chỉ định:

```php
$browser->withinFrame('#credit-card-details', function ($browser) {
    $browser->type('input[name="cardnumber"]', '4242424242424242')
        ->type('input[name="exp-date"]', '1224')
        ->type('input[name="cvc"]', '123')
        ->press('Pay');
});
```

<a name="scoping-selectors"></a>

### Giới Hạn Selectors

Đôi khi bạn có thể muốn thực hiện một số thao tác trong khi giới hạn tất cả các thao tác trong một selector đã cho. Ví dụ, bạn có thể muốn xác nhận rằng một số văn bản chỉ tồn tại trong một bảng và sau đó nhấp vào một nút trong bảng đó. Bạn có thể sử dụng phương thức `with` để thực hiện việc này. Tất cả các thao tác được thực hiện trong closure được cung cấp cho phương thức `with` sẽ được giới hạn trong selector gốc:

```php
$browser->with('.table', function (Browser $table) {
    $table->assertSee('Hello World')
        ->clickLink('Delete');
});
```

Đôi khi bạn có thể cần thực hiện các xác nhận bên ngoài phạm vi hiện tại. Bạn có thể sử dụng các phương thức `elsewhere` và `elsewhereWhenAvailable` để thực hiện việc này:

```php
$browser->with('.table', function (Browser $table) {
    // Phạm vi hiện tại là `body .table`...

    $browser->elsewhere('.page-title', function (Browser $title) {
        // Phạm vi hiện tại là `body .page-title`...
        $title->assertSee('Hello World');
    });

    $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
        // Phạm vi hiện tại là `body .page-title`...
        $title->assertSee('Hello World');
    });
});
```

<a name="waiting-for-elements"></a>

### Đợi Các Phần Tử

Khi kiểm tra các ứng dụng sử dụng JavaScript rộng rãi, thường cần thiết phải "đợi" một số phần tử hoặc dữ liệu có sẵn trước khi tiếp tục với một test. Dusk làm cho việc này trở nên dễ dàng. Sử dụng nhiều phương thức, bạn có thể đợi các phần tử trở nên hiển thị trên trang hoặc thậm chí đợi cho đến khi một biểu thức JavaScript đã cho đánh giá là `true`.

<a name="waiting"></a>

#### Đợi

Nếu bạn chỉ cần tạm dừng test trong một số mili-giây đã cho, hãy sử dụng phương thức `pause`:

```php
$browser->pause(1000);
```

Nếu bạn cần tạm dừng test chỉ khi một điều kiện đã cho là `true`, hãy sử dụng phương thức `pauseIf`:

```php
$browser->pauseIf(App::environment('production'), 1000);
```

Tương tự, nếu bạn cần tạm dừng test trừ khi một điều kiện đã cho là `true`, bạn có thể sử dụng phương thức `pauseUnless`:

```php
$browser->pauseUnless(App::environment('testing'), 1000);
```

<a name="waiting-for-selectors"></a>

#### Đợi Selectors

Phương thức `waitFor` có thể được sử dụng để tạm dừng thực thi test cho đến khi phần tử khớp với CSS hoặc Dusk selector đã cho được hiển thị trên trang. Theo mặc định, điều này sẽ tạm dừng test tối đa năm giây trước khi ném một ngoại lệ. Nếu cần thiết, bạn có thể truyền ngưỡng timeout tùy chỉnh làm đối số thứ hai cho phương thức:

```php
// Đợi tối đa năm giây cho selector...
$browser->waitFor('.selector');

// Đợi tối đa một giây cho selector...
$browser->waitFor('.selector', 1);
```

Bạn cũng có thể đợi cho đến khi phần tử khớp với selector đã cho chứa văn bản đã cho:

```php
// Đợi tối đa năm giây để selector chứa văn bản đã cho...
$browser->waitForTextIn('.selector', 'Hello World');

// Đợi tối đa một giây để selector chứa văn bản đã cho...
$browser->waitForTextIn('.selector', 'Hello World', 1);
```

Bạn cũng có thể đợi cho đến khi phần tử khớp với selector đã cho bị thiếu khỏi trang:

```php
// Đợi tối đa năm giây cho đến khi selector bị thiếu...
$browser->waitUntilMissing('.selector');

// Đợi tối đa một giây cho đến khi selector bị thiếu...
$browser->waitUntilMissing('.selector', 1);
```

Hoặc, bạn có thể đợi cho đến khi phần tử khớp với selector đã cho được bật hoặc tắt:

```php
// Đợi tối đa năm giây cho đến khi selector được bật...
$browser->waitUntilEnabled('.selector');

// Đợi tối đa một giây cho đến khi selector được bật...
$browser->waitUntilEnabled('.selector', 1);

// Đợi tối đa năm giây cho đến khi selector bị tắt...
$browser->waitUntilDisabled('.selector');

// Đợi tối đa một giây cho đến khi selector bị tắt...
$browser->waitUntilDisabled('.selector', 1);
```

<a name="scoping-selectors-when-available"></a>

#### Giới Hạn Selectors Khi Có Sẵn

Đôi khi, bạn có thể muốn đợi một phần tử xuất hiện khớp với một selector đã cho và sau đó tương tác với phần tử đó. Ví dụ, bạn có thể muốn đợi cho đến khi một cửa sổ modal có sẵn và sau đó nhấn nút "OK" trong modal. Phương thức `whenAvailable` có thể được sử dụng để thực hiện việc này. Tất cả các thao tác phần tử được thực hiện trong closure đã cho sẽ được giới hạn trong selector gốc:

```php
$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello World')
        ->press('OK');
});
```

<a name="waiting-for-text"></a>

#### Đợi Văn Bản

Phương thức `waitForText` có thể được sử dụng để đợi cho đến khi văn bản đã cho được hiển thị trên trang:

```php
// Đợi tối đa năm giây cho văn bản...
$browser->waitForText('Hello World');

// Đợi tối đa một giây cho văn bản...
$browser->waitForText('Hello World', 1);
```

Bạn có thể sử dụng phương thức `waitUntilMissingText` để đợi cho đến khi văn bản hiển thị đã bị xóa khỏi trang:

```php
// Đợi tối đa năm giây để văn bản bị xóa...
$browser->waitUntilMissingText('Hello World');

// Đợi tối đa một giây để văn bản bị xóa...
$browser->waitUntilMissingText('Hello World', 1);
```

<a name="waiting-for-links"></a>

#### Đợi Liên Kết

Phương thức `waitForLink` có thể được sử dụng để đợi cho đến khi văn bản liên kết đã cho được hiển thị trên trang:

```php
// Đợi tối đa năm giây cho liên kết...
$browser->waitForLink('Create');

// Đợi tối đa một giây cho liên kết...
$browser->waitForLink('Create', 1);
```

<a name="waiting-for-inputs"></a>

#### Đợi Inputs

Phương thức `waitForInput` có thể được sử dụng để đợi cho đến khi trường input đã cho hiển thị trên trang:

```php
// Đợi tối đa năm giây cho input...
$browser->waitForInput($field);

// Đợi tối đa một giây cho input...
$browser->waitForInput($field, 1);
```

<a name="waiting-on-the-page-location"></a>

#### Đợi Vị Trí Trang

Khi thực hiện xác nhận đường dẫn như `$browser->assertPathIs('/home')`, xác nhận có thể thất bại nếu `window.location.pathname` đang được cập nhật không đồng bộ. Bạn có thể sử dụng phương thức `waitForLocation` để đợi vị trí trở thành một giá trị đã cho:

```php
$browser->waitForLocation('/secret');
```

Phương thức `waitForLocation` cũng có thể được sử dụng để đợi vị trí cửa sổ hiện tại trở thành một URL đầy đủ:

```php
$browser->waitForLocation('https://example.com/path');
```

Bạn cũng có thể đợi vị trí của một [route được đặt tên](/docs/{{version}}/routing#named-routes):

```php
$browser->waitForRoute($routeName, $parameters);
```

<a name="waiting-for-page-reloads"></a>

#### Đợi Tải Lại Trang

Nếu bạn cần đợi một trang tải lại sau khi thực hiện một hành động, hãy sử dụng phương thức `waitForReload`:

```php
use Laravel\Dusk\Browser;

$browser->waitForReload(function (Browser $browser) {
    $browser->press('Submit');
})
->assertSee('Success!');
```

Vì nhu cầu đợi trang tải lại thường xảy ra sau khi nhấp vào một nút, bạn có thể sử dụng phương thức `clickAndWaitForReload` để thuận tiện:

```php
$browser->clickAndWaitForReload('.selector')
    ->assertSee('something');
```

<a name="waiting-on-javascript-expressions"></a>

#### Đợi Biểu Thức JavaScript

Đôi khi bạn có thể muốn tạm dừng thực thi test cho đến khi một biểu thức JavaScript đã cho đánh giá là `true`. Bạn có thể dễ dàng thực hiện việc này bằng phương thức `waitUntil`. Khi truyền biểu thức cho phương thức này, bạn không cần bao gồm từ khóa `return` hoặc dấu chấm phẩy kết thúc:

```php
// Đợi tối đa năm giây để biểu thức là true...
$browser->waitUntil('App.data.servers.length > 0');

// Đợi tối đa một giây để biểu thức là true...
$browser->waitUntil('App.data.servers.length > 0', 1);
```

<a name="waiting-on-vue-expressions"></a>

#### Đợi Biểu Thức Vue

Các phương thức `waitUntilVue` và `waitUntilVueIsNot` có thể được sử dụng để đợi cho đến khi một thuộc tính [Vue component](https://vuejs.org) có một giá trị đã cho:

```php
// Đợi cho đến khi thuộc tính component chứa giá trị đã cho...
$browser->waitUntilVue('user.name', 'Taylor', '@user');

// Đợi cho đến khi thuộc tính component không chứa giá trị đã cho...
$browser->waitUntilVueIsNot('user.name', null, '@user');
```

<a name="waiting-for-javascript-events"></a>

#### Đợi Sự Kiện JavaScript

Phương thức `waitForEvent` có thể được sử dụng để tạm dừng thực thi test cho đến khi một sự kiện JavaScript xảy ra:

```php
$browser->waitForEvent('load');
```

Event listener được gắn vào phạm vi hiện tại, là phần tử `body` theo mặc định. Khi sử dụng selector có phạm vi, event listener sẽ được gắn vào phần tử khớp:

```php
$browser->with('iframe', function (Browser $iframe) {
    // Đợi sự kiện load của iframe...
    $iframe->waitForEvent('load');
});
```

Bạn cũng có thể cung cấp một selector làm đối số thứ hai cho phương thức `waitForEvent` để gắn event listener vào một phần tử cụ thể:

```php
$browser->waitForEvent('load', '.selector');
```

Bạn cũng có thể đợi các sự kiện trên các đối tượng `document` và `window`:

```php
// Đợi cho đến khi document được cuộn...
$browser->waitForEvent('scroll', 'document');

// Đợi tối đa năm giây cho đến khi window được thay đổi kích thước...
$browser->waitForEvent('resize', 'window', 5);
```

<a name="waiting-with-a-callback"></a>

#### Đợi Với Callback

Nhiều phương thức "đợi" trong Dusk dựa trên phương thức `waitUsing` cơ bản. Bạn có thể sử dụng phương thức này trực tiếp để đợi một closure đã cho trả về `true`. Phương thức `waitUsing` chấp nhận số giây tối đa để đợi, khoảng thời gian mà closure nên được đánh giá, closure, và một thông báo thất bại tùy chọn:

```php
$browser->waitUsing(10, 1, function () use ($something) {
    return $something->isReady();
}, "Something wasn't ready in time.");
```

<a name="scrolling-an-element-into-view"></a>

### Cuộn Phần Tử Vào Khung Nhìn

Đôi khi bạn có thể không thể nhấp vào một phần tử vì nó nằm ngoài vùng hiển thị của trình duyệt. Phương thức `scrollIntoView` sẽ cuộn cửa sổ trình duyệt cho đến khi phần tử tại selector đã cho nằm trong khung nhìn:

```php
$browser->scrollIntoView('.selector')
    ->click('.selector');
```

<a name="available-assertions"></a>

## Các Xác Nhận Có Sẵn

Dusk cung cấp nhiều xác nhận mà bạn có thể thực hiện đối với ứng dụng của mình. Tất cả các xác nhận có sẵn được tài liệu hóa trong danh sách dưới đây:

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertSchemeIs](#assert-scheme-is)
[assertSchemeIsNot](#assert-scheme-is-not)
[assertHostIs](#assert-host-is)
[assertHostIsNot](#assert-host-is-not)
[assertPortIs](#assert-port-is)
[assertPortIsNot](#assert-port-is-not)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathEndsWith](#assert-path-ends-with)
[assertPathContains](#assert-path-contains)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertHasPlainCookie](#assert-has-plain-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertPlainCookieMissing](#assert-plain-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSeeAnythingIn](#assert-see-anything-in)
[assertSeeNothingIn](#assert-see-nothing-in)
[assertCount](#assert-count)
[assertScript](#assert-script)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertIndeterminate](#assert-indeterminate)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertSelectMissingOption](#assert-select-missing-option)
[assertValue](#assert-value)
[assertValueIsNot](#assert-value-is-not)
[assertAttribute](#assert-attribute)
[assertAttributeMissing](#assert-attribute-missing)
[assertAttributeContains](#assert-attribute-contains)
[assertAttributeDoesntContain](#assert-attribute-doesnt-contain)
[assertAriaAttribute](#assert-aria-attribute)
[assertDataAttribute](#assert-data-attribute)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertNotPresent](#assert-not-present)
[assertMissing](#assert-missing)
[assertInputPresent](#assert-input-present)
[assertInputMissing](#assert-input-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertAuthenticated](#assert-authenticated)
[assertGuest](#assert-guest)
[assertAuthenticatedAs](#assert-authenticated-as)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesntContain](#assert-vue-doesnt-contain)

</div>

<a name="assert-title"></a>

#### assertTitle

Khẳng định rằng tiêu đề trang khớp với văn bản đã cho:

```php
$browser->assertTitle($title);
```

<a name="assert-title-contains"></a>

#### assertTitleContains

Khẳng định rằng tiêu đề trang chứa văn bản đã cho:

```php
$browser->assertTitleContains($title);
```

<a name="assert-url-is"></a>

#### assertUrlIs

Khẳng định rằng URL hiện tại (không bao gồm query string) khớp với chuỗi đã cho:

```php
$browser->assertUrlIs($url);
```

<a name="assert-scheme-is"></a>

#### assertSchemeIs

Khẳng định rằng scheme của URL hiện tại khớp với scheme đã cho:

```php
$browser->assertSchemeIs($scheme);
```

<a name="assert-scheme-is-not"></a>

#### assertSchemeIsNot

Khẳng định rằng scheme của URL hiện tại không khớp với scheme đã cho:

```php
$browser->assertSchemeIsNot($scheme);
```

<a name="assert-host-is"></a>

#### assertHostIs

Khẳng định rằng host của URL hiện tại khớp với host đã cho:

```php
$browser->assertHostIs($host);
```

<a name="assert-host-is-not"></a>

#### assertHostIsNot

Khẳng định rằng host của URL hiện tại không khớp với host đã cho:

```php
$browser->assertHostIsNot($host);
```

<a name="assert-port-is"></a>

#### assertPortIs

Khẳng định rằng port của URL hiện tại khớp với port đã cho:

```php
$browser->assertPortIs($port);
```

<a name="assert-port-is-not"></a>

#### assertPortIsNot

Khẳng định rằng port của URL hiện tại không khớp với port đã cho:

```php
$browser->assertPortIsNot($port);
```

<a name="assert-path-begins-with"></a>

#### assertPathBeginsWith

Khẳng định rằng đường dẫn URL hiện tại bắt đầu bằng đường dẫn đã cho:

```php
$browser->assertPathBeginsWith('/home');
```

<a name="assert-path-ends-with"></a>

#### assertPathEndsWith

Khẳng định rằng đường dẫn URL hiện tại kết thúc bằng đường dẫn đã cho:

```php
$browser->assertPathEndsWith('/home');
```

<a name="assert-path-contains"></a>

#### assertPathContains

Khẳng định rằng đường dẫn URL hiện tại chứa đường dẫn đã cho:

```php
$browser->assertPathContains('/home');
```

<a name="assert-path-is"></a>

#### assertPathIs

Khẳng định rằng đường dẫn hiện tại khớp với đường dẫn đã cho:

```php
$browser->assertPathIs('/home');
```

<a name="assert-path-is-not"></a>

#### assertPathIsNot

Khẳng định rằng đường dẫn hiện tại không khớp với đường dẫn đã cho:

```php
$browser->assertPathIsNot('/home');
```

<a name="assert-route-is"></a>

#### assertRouteIs

Khẳng định rằng URL hiện tại khớp với URL của [route có tên](/docs/{{version}}/routing#named-routes) đã cho:

```php
$browser->assertRouteIs($name, $parameters);
```

<a name="assert-query-string-has"></a>

#### assertQueryStringHas

Khẳng định rằng tham số query string đã cho có mặt:

```php
$browser->assertQueryStringHas($name);
```

Khẳng định rằng tham số query string đã cho có mặt và có giá trị đã cho:

```php
$browser->assertQueryStringHas($name, $value);
```

<a name="assert-query-string-missing"></a>

#### assertQueryStringMissing

Khẳng định rằng tham số query string đã cho không có mặt:

```php
$browser->assertQueryStringMissing($name);
```

<a name="assert-fragment-is"></a>

#### assertFragmentIs

Khẳng định rằng fragment hash hiện tại của URL khớp với fragment đã cho:

```php
$browser->assertFragmentIs('anchor');
```

<a name="assert-fragment-begins-with"></a>

#### assertFragmentBeginsWith

Khẳng định rằng fragment hash hiện tại của URL bắt đầu bằng fragment đã cho:

```php
$browser->assertFragmentBeginsWith('anchor');
```

<a name="assert-fragment-is-not"></a>

#### assertFragmentIsNot

Khẳng định rằng fragment hash hiện tại của URL không khớp với fragment đã cho:

```php
$browser->assertFragmentIsNot('anchor');
```

<a name="assert-has-cookie"></a>

#### assertHasCookie

Khẳng định rằng cookie đã mã hóa đã cho có mặt:

```php
$browser->assertHasCookie($name);
```

<a name="assert-has-plain-cookie"></a>

#### assertHasPlainCookie

Khẳng định rằng cookie chưa mã hóa đã cho có mặt:

```php
$browser->assertHasPlainCookie($name);
```

<a name="assert-cookie-missing"></a>

#### assertCookieMissing

Khẳng định rằng cookie đã mã hóa đã cho không có mặt:

```php
$browser->assertCookieMissing($name);
```

<a name="assert-plain-cookie-missing"></a>

#### assertPlainCookieMissing

Khẳng định rằng cookie chưa mã hóa đã cho không có mặt:

```php
$browser->assertPlainCookieMissing($name);
```

<a name="assert-cookie-value"></a>

#### assertCookieValue

Khẳng định rằng cookie đã mã hóa có giá trị đã cho:

```php
$browser->assertCookieValue($name, $value);
```

<a name="assert-plain-cookie-value"></a>

#### assertPlainCookieValue

Khẳng định rằng cookie chưa mã hóa có giá trị đã cho:

```php
$browser->assertPlainCookieValue($name, $value);
```

<a name="assert-see"></a>

#### assertSee

Khẳng định rằng văn bản đã cho có mặt trên trang:

```php
$browser->assertSee($text);
```

<a name="assert-dont-see"></a>

#### assertDontSee

Khẳng định rằng văn bản đã cho không có mặt trên trang:

```php
$browser->assertDontSee($text);
```

<a name="assert-see-in"></a>

#### assertSeeIn

Khẳng định rằng văn bản đã cho có mặt trong selector:

```php
$browser->assertSeeIn($selector, $text);
```

<a name="assert-dont-see-in"></a>

#### assertDontSeeIn

Khẳng định rằng văn bản đã cho không có mặt trong selector:

```php
$browser->assertDontSeeIn($selector, $text);
```

<a name="assert-see-anything-in"></a>

#### assertSeeAnythingIn

Khẳng định rằng có bất kỳ văn bản nào có mặt trong selector:

```php
$browser->assertSeeAnythingIn($selector);
```

<a name="assert-see-nothing-in"></a>

#### assertSeeNothingIn

Khẳng định rằng không có văn bản nào có mặt trong selector:

```php
$browser->assertSeeNothingIn($selector);
```

<a name="assert-count"></a>

#### assertCount

Khẳng định rằng các phần tử khớp với selector đã cho xuất hiện số lần được chỉ định:

```php
$browser->assertCount($selector, $count);
```

<a name="assert-script"></a>

#### assertScript

Khẳng định rằng biểu thức JavaScript đã cho được đánh giá thành giá trị đã cho:

```php
$browser->assertScript('window.isLoaded')
    ->assertScript('document.readyState', 'complete');
```

<a name="assert-source-has"></a>

#### assertSourceHas

Khẳng định rằng mã nguồn đã cho có mặt trên trang:

```php
$browser->assertSourceHas($code);
```

<a name="assert-source-missing"></a>

#### assertSourceMissing

Khẳng định rằng mã nguồn đã cho không có mặt trên trang:

```php
$browser->assertSourceMissing($code);
```

<a name="assert-see-link"></a>

#### assertSeeLink

Khẳng định rằng liên kết đã cho có mặt trên trang:

```php
$browser->assertSeeLink($linkText);
```

<a name="assert-dont-see-link"></a>

#### assertDontSeeLink

Khẳng định rằng liên kết đã cho không có mặt trên trang:

```php
$browser->assertDontSeeLink($linkText);
```

<a name="assert-input-value"></a>

#### assertInputValue

Khẳng định rằng trường nhập liệu đã cho có giá trị đã cho:

```php
$browser->assertInputValue($field, $value);
```

<a name="assert-input-value-is-not"></a>

#### assertInputValueIsNot

Khẳng định rằng trường nhập liệu đã cho không có giá trị đã cho:

```php
$browser->assertInputValueIsNot($field, $value);
```

<a name="assert-checked"></a>

#### assertChecked

Khẳng định rằng checkbox đã cho được chọn:

```php
$browser->assertChecked($field);
```

<a name="assert-not-checked"></a>

#### assertNotChecked

Khẳng định rằng checkbox đã cho không được chọn:

```php
$browser->assertNotChecked($field);
```

<a name="assert-indeterminate"></a>

#### assertIndeterminate

Khẳng định rằng checkbox đã cho ở trạng thái không xác định:

```php
$browser->assertIndeterminate($field);
```

<a name="assert-radio-selected"></a>

#### assertRadioSelected

Khẳng định rằng trường radio đã cho được chọn:

```php
$browser->assertRadioSelected($field, $value);
```

<a name="assert-radio-not-selected"></a>

#### assertRadioNotSelected

Khẳng định rằng trường radio đã cho không được chọn:

```php
$browser->assertRadioNotSelected($field, $value);
```

<a name="assert-selected"></a>

#### assertSelected

Khẳng định rằng dropdown đã cho có giá trị đã cho được chọn:

```php
$browser->assertSelected($field, $value);
```

<a name="assert-not-selected"></a>

#### assertNotSelected

Khẳng định rằng dropdown đã cho không có giá trị đã cho được chọn:

```php
$browser->assertNotSelected($field, $value);
```

<a name="assert-select-has-options"></a>

#### assertSelectHasOptions

Khẳng định rằng mảng giá trị đã cho có sẵn để chọn:

```php
$browser->assertSelectHasOptions($field, $values);
```

<a name="assert-select-missing-options"></a>

#### assertSelectMissingOptions

Khẳng định rằng mảng giá trị đã cho không có sẵn để chọn:

```php
$browser->assertSelectMissingOptions($field, $values);
```

<a name="assert-select-has-option"></a>

#### assertSelectHasOption

Khẳng định rằng giá trị đã cho có sẵn để chọn trên trường đã cho:

```php
$browser->assertSelectHasOption($field, $value);
```

<a name="assert-select-missing-option"></a>

#### assertSelectMissingOption

Khẳng định rằng giá trị đã cho không có sẵn để chọn:

```php
$browser->assertSelectMissingOption($field, $value);
```

<a name="assert-value"></a>

#### assertValue

Khẳng định rằng phần tử khớp với selector đã cho có giá trị đã cho:

```php
$browser->assertValue($selector, $value);
```

<a name="assert-value-is-not"></a>

#### assertValueIsNot

Khẳng định rằng phần tử khớp với selector đã cho không có giá trị đã cho:

```php
$browser->assertValueIsNot($selector, $value);
```

<a name="assert-attribute"></a>

#### assertAttribute

Khẳng định rằng phần tử khớp với selector đã cho có giá trị đã cho trong thuộc tính được cung cấp:

```php
$browser->assertAttribute($selector, $attribute, $value);
```

<a name="assert-attribute-missing"></a>

#### assertAttributeMissing

Khẳng định rằng phần tử khớp với selector đã cho thiếu thuộc tính được cung cấp:

```php
$browser->assertAttributeMissing($selector, $attribute);
```

<a name="assert-attribute-contains"></a>

#### assertAttributeContains

Khẳng định rằng phần tử khớp với selector đã cho chứa giá trị đã cho trong thuộc tính được cung cấp:

```php
$browser->assertAttributeContains($selector, $attribute, $value);
```

<a name="assert-attribute-doesnt-contain"></a>

#### assertAttributeDoesntContain

Khẳng định rằng phần tử khớp với selector đã cho không chứa giá trị đã cho trong thuộc tính được cung cấp:

```php
$browser->assertAttributeDoesntContain($selector, $attribute, $value);
```

<a name="assert-aria-attribute"></a>

#### assertAriaAttribute

Khẳng định rằng phần tử khớp với selector đã cho có giá trị đã cho trong thuộc tính aria được cung cấp:

```php
$browser->assertAriaAttribute($selector, $attribute, $value);
```

Ví dụ, với markup `<button aria-label="Add"></button>`, bạn có thể khẳng định thuộc tính `aria-label` như sau:

```php
$browser->assertAriaAttribute('button', 'label', 'Add')
```

<a name="assert-data-attribute"></a>

#### assertDataAttribute

Khẳng định rằng phần tử khớp với selector đã cho có giá trị đã cho trong thuộc tính data được cung cấp:

```php
$browser->assertDataAttribute($selector, $attribute, $value);
```

Ví dụ, với markup `<tr id="row-1" data-content="attendees"></tr>`, bạn có thể khẳng định thuộc tính `data-content` như sau:

```php
$browser->assertDataAttribute('#row-1', 'content', 'attendees')
```

<a name="assert-visible"></a>

#### assertVisible

Khẳng định rằng phần tử khớp với selector đã cho hiển thị:

```php
$browser->assertVisible($selector);
```

<a name="assert-present"></a>

#### assertPresent

Khẳng định rằng phần tử khớp với selector đã cho có mặt trong mã nguồn:

```php
$browser->assertPresent($selector);
```

<a name="assert-not-present"></a>

#### assertNotPresent

Khẳng định rằng phần tử khớp với selector đã cho không có mặt trong mã nguồn:

```php
$browser->assertNotPresent($selector);
```

<a name="assert-missing"></a>

#### assertMissing

Khẳng định rằng phần tử khớp với selector đã cho không hiển thị:

```php
$browser->assertMissing($selector);
```

<a name="assert-input-present"></a>

#### assertInputPresent

Khẳng định rằng một trường nhập liệu với tên đã cho có mặt:

```php
$browser->assertInputPresent($name);
```

<a name="assert-input-missing"></a>

#### assertInputMissing

Khẳng định rằng một trường nhập liệu với tên đã cho không có mặt trong mã nguồn:

```php
$browser->assertInputMissing($name);
```

<a name="assert-dialog-opened"></a>

#### assertDialogOpened

Khẳng định rằng một hộp thoại JavaScript với thông điệp đã cho đã được mở:

```php
$browser->assertDialogOpened($message);
```

<a name="assert-enabled"></a>

#### assertEnabled

Khẳng định rằng trường đã cho được bật:

```php
$browser->assertEnabled($field);
```

<a name="assert-disabled"></a>

#### assertDisabled

|
|Khẳng định rằng trường đã cho bị vô hiệu hóa:

```php
$browser->assertDisabled($field);
```

<a name="assert-button-enabled"></a>

#### assertButtonEnabled

Khẳng định rằng nút đã cho được kích hoạt:

```php
$browser->assertButtonEnabled($button);
```

<a name="assert-button-disabled"></a>

#### assertButtonDisabled

Khẳng định rằng nút đã cho bị vô hiệu hóa:

```php
$browser->assertButtonDisabled($button);
```

<a name="assert-focused"></a>

#### assertFocused

Khẳng định rằng trường đã cho đang được focus:

```php
$browser->assertFocused($field);
```

<a name="assert-not-focused"></a>

#### assertNotFocused

Khẳng định rằng trường đã cho không đang được focus:

```php
$browser->assertNotFocused($field);
```

<a name="assert-authenticated"></a>

#### assertAuthenticated

Khẳng định rằng người dùng đã được xác thực:

```php
$browser->assertAuthenticated();
```

<a name="assert-guest"></a>

#### assertGuest

Khẳng định rằng người dùng chưa được xác thực:

```php
$browser->assertGuest();
```

<a name="assert-authenticated-as"></a>

#### assertAuthenticatedAs

Khẳng định rằng người dùng đã được xác thực với tư cách là người dùng đã cho:

```php
$browser->assertAuthenticatedAs($user);
```

<a name="assert-vue"></a>

#### assertVue

Dusk thậm chí cho phép bạn thực hiện các khẳng định về trạng thái dữ liệu của [component Vue](https://vuejs.org). Ví dụ, hãy tưởng tượng ứng dụng của bạn chứa component Vue sau:

    // HTML...

    <profile dusk="profile-component"></profile>

    // Component Definition...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                    name: 'Taylor'
                }
            };
        }
    });

Bạn có thể thực hiện khẳng định về trạng thái của component Vue như sau:

```php
test('vue', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->assertVue('user.name', 'Taylor', '@profile-component');
    });
});
```

```php
/**
 * A basic Vue test example.
 */
public function test_vue(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->assertVue('user.name', 'Taylor', '@profile-component');
    });
}
```

<a name="assert-vue-is-not"></a>

#### assertVueIsNot

Khẳng định rằng thuộc tính dữ liệu của component Vue đã cho không khớp với giá trị đã cho:

```php
$browser->assertVueIsNot($property, $value, $componentSelector = null);
```

<a name="assert-vue-contains"></a>

#### assertVueContains

Khẳng định rằng thuộc tính dữ liệu của component Vue đã cho là một mảng và chứa giá trị đã cho:

```php
$browser->assertVueContains($property, $value, $componentSelector = null);
```

<a name="assert-vue-doesnt-contain"></a>

#### assertVueDoesntContain

Khẳng định rằng thuộc tính dữ liệu của component Vue đã cho là một mảng và không chứa giá trị đã cho:

```php
$browser->assertVueDoesntContain($property, $value, $componentSelector = null);
```

<a name="pages"></a>

## Pages

Đôi khi, các bài kiểm tra yêu cầu thực hiện nhiều hành động phức tạp theo trình tự. Điều này có thể làm cho các bài kiểm tra của bạn khó đọc và hiểu hơn. Dusk Pages cho phép bạn định nghĩa các hành động biểu đạt có thể sau đó được thực hiện trên một trang đã cho thông qua một phương thức duy nhất. Pages cũng cho phép bạn định nghĩa các phím tắt cho các bộ chọn phổ biến cho ứng dụng của bạn hoặc cho một trang duy nhất.

<a name="generating-pages"></a>

### Generating Pages

Để tạo một đối tượng page, thực thi lệnh Artisan `dusk:page`. Tất cả các đối tượng page sẽ được đặt trong thư mục `tests/Browser/Pages` của ứng dụng của bạn:

```shell
php artisan dusk:page Login
```

<a name="configuring-pages"></a>

### Configuring Pages

Theo mặc định, các page có ba phương thức: `url`, `assert`, và `elements`. Chúng ta sẽ thảo luận về các phương thức `url` và `assert` ngay bây giờ. Phương thức `elements` sẽ được [thảo luận chi tiết hơn bên dưới](#shorthand-selectors).

<a name="the-url-method"></a>

#### The `url` Method

Phương thức `url` nên trả về đường dẫn của URL đại diện cho trang. Dusk sẽ sử dụng URL này khi điều hướng đến trang trong trình duyệt:

```php
/**
 * Get the URL for the page.
 */
public function url(): string
{
    return '/login';
}
```

<a name="the-assert-method"></a>

#### The `assert` Method

Phương thức `assert` có thể thực hiện bất kỳ khẳng định nào cần thiết để xác minh rằng trình duyệt thực sự đang ở trên trang đã cho. Không thực sự cần thiết để đặt bất kỳ thứ gì trong phương thức này; tuy nhiên, bạn có thể tự do thực hiện các khẳng định này nếu bạn muốn. Các khẳng định này sẽ được chạy tự động khi điều hướng đến trang:

```php
/**
 * Assert that the browser is on the page.
 */
public function assert(Browser $browser): void
{
    $browser->assertPathIs($this->url());
}
```

<a name="navigating-to-pages"></a>

### Navigating to Pages

Khi một trang đã được định nghĩa, bạn có thể điều hướng đến nó bằng phương thức `visit`:

```php
use Tests\Browser\Pages\Login;

$browser->visit(new Login);
```

Đôi khi bạn có thể đã ở trên một trang đã cho và cần "tải" các bộ chọn và phương thức của trang vào ngữ cảnh kiểm tra hiện tại. Điều này thường gặp khi nhấn một nút và được chuyển hướng đến một trang đã cho mà không điều hướng rõ ràng đến nó. Trong tình huống này, bạn có thể sử dụng phương thức `on` để tải trang:

```php
use Tests\Browser\Pages\CreatePlaylist;

$browser->visit('/dashboard')
    ->clickLink('Create Playlist')
    ->on(new CreatePlaylist)
    ->assertSee('@create');
```

<a name="shorthand-selectors"></a>

### Shorthand Selectors

Phương thức `elements` trong các lớp page cho phép bạn định nghĩa các phím tắt nhanh, dễ nhớ cho bất kỳ bộ chọn CSS nào trên trang của bạn. Ví dụ, hãy định nghĩa một phím tắt cho trường nhập "email" của trang đăng nhập của ứng dụng:

```php
/**
 * Get the element shortcuts for the page.
 *
 * @return array<string, string>
 */
public function elements(): array
{
    return [
        '@email' => 'input[name=email]',
    ];
}
```

Khi phím tắt đã được định nghĩa, bạn có thể sử dụng bộ chọn viết tắt ở bất kỳ nơi nào bạn thường sử dụng bộ chọn CSS đầy đủ:

```php
$browser->type('@email', 'taylor@laravel.com');
```

<a name="global-shorthand-selectors"></a>

#### Global Shorthand Selectors

Sau khi cài đặt Dusk, một lớp `Page` cơ sở sẽ được đặt trong thư mục `tests/Browser/Pages` của bạn. Lớp này chứa phương thức `siteElements` có thể được sử dụng để định nghĩa các bộ chọn viết tắt toàn cầu nên có sẵn trên mọi trang trong suốt ứng dụng của bạn:

```php
/**
 * Get the global element shortcuts for the site.
 *
 * @return array<string, string>
 */
public static function siteElements(): array
{
    return [
        '@element' => '#selector',
    ];
}
```

<a name="page-methods"></a>

### Page Methods

Ngoài các phương thức mặc định được định nghĩa trên các page, bạn có thể định nghĩa các phương thức bổ sung có thể được sử dụng trong suốt các bài kiểm tra của bạn. Ví dụ, hãy tưởng tượng chúng ta đang xây dựng một ứng dụng quản lý nhạc. Một hành động phổ biến cho một trang của ứng dụng có thể là tạo một danh sách phát. Thay vì viết lại logic để tạo danh sách phát trong mỗi bài kiểm tra, bạn có thể định nghĩa một phương thức `createPlaylist` trên một lớp page:

```php
<?php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class Dashboard extends Page
{
    // Other page methods...

    /**
     * Create a new playlist.
     */
    public function createPlaylist(Browser $browser, string $name): void
    {
        $browser->type('name', $name)
            ->check('share')
            ->press('Create Playlist');
    }
}
```

Khi phương thức đã được định nghĩa, bạn có thể sử dụng nó trong bất kỳ bài kiểm tra nào sử dụng trang. Đối tượng trình duyệt sẽ tự động được chuyển làm đối số đầu tiên cho các phương thức page tùy chỉnh:

```php
use Tests\Browser\Pages\Dashboard;

$browser->visit(new Dashboard)
    ->createPlaylist('My Playlist')
    ->assertSee('My Playlist');
```

<a name="components"></a>

## Components

Các component tương tự như "đối tượng page" của Dusk, nhưng được dự định cho các phần của UI và chức năng được tái sử dụng trong suốt ứng dụng của bạn, chẳng hạn như thanh điều hướng hoặc cửa sổ thông báo. Do đó, các component không bị ràng buộc với các URL cụ thể.

<a name="generating-components"></a>

### Generating Components

Để tạo một component, thực thi lệnh Artisan `dusk:component`. Các component mới được đặt trong thư mục `tests/Browser/Components`:

```shell
php artisan dusk:component DatePicker
```

Như được hiển thị ở trên, một "date picker" là một ví dụ về một component có thể tồn tại trong suốt ứng dụng của bạn trên nhiều trang khác nhau. Nó có thể trở nên cồng kềnh khi viết thủ công logic tự động hóa trình duyệt để chọn một ngày trong hàng chục bài kiểm tra trong suốt bộ kiểm tra của bạn. Thay vào đó, chúng ta có thể định nghĩa một component Dusk để đại diện cho date picker, cho phép chúng ta đóng gói logic đó trong component:

```php
<?php

namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DatePicker extends BaseComponent
{
    /**
     * Get the root selector for the component.
     */
    public function selector(): string
    {
        return '.date-picker';
    }

    /**
     * Assert that the browser page contains the component.
     */
    public function assert(Browser $browser): void
    {
        $browser->assertVisible($this->selector());
    }

    /**
     * Get the element shortcuts for the component.
     *
     * @return array<string, string>
     */
    public function elements(): array
    {
        return [
            '@date-field' => 'input.datepicker-input',
            '@year-list' => 'div > div.datepicker-years',
            '@month-list' => 'div > div.datepicker-months',
            '@day-list' => 'div > div.datepicker-days',
        ];
    }

    /**
     * Select the given date.
     */
    public function selectDate(Browser $browser, int $year, int $month, int $day): void
    {
        $browser->click('@date-field')
            ->within('@year-list', function (Browser $browser) use ($year) {
                $browser->click($year);
            })
            ->within('@month-list', function (Browser $browser) use ($month) {
                $browser->click($month);
            })
            ->within('@day-list', function (Browser $browser) use ($day) {
                $browser->click($day);
            });
    }
}
```

<a name="using-components"></a>

### Using Components

Khi component đã được định nghĩa, chúng ta có thể dễ dàng chọn một ngày trong date picker từ bất kỳ bài kiểm tra nào. Và, nếu logic cần thiết để chọn một ngày thay đổi, chúng ta chỉ cần cập nhật component:

```php
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;

pest()->use(DatabaseMigrations::class);

test('basic example', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->within(new DatePicker, function (Browser $browser) {
                $browser->selectDate(2019, 1, 30);
            })
            ->assertSee('January');
    });
});
```

```php
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    /**
     * A basic component test example.
     */
    public function test_basic_example(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                ->within(new DatePicker, function (Browser $browser) {
                    $browser->selectDate(2019, 1, 30);
                })
                ->assertSee('January');
        });
    }
}
```

Phương thức `component` có thể được sử dụng để lấy một đối tượng trình duyệt có phạm vi cho component đã cho:

```php
$datePicker = $browser->component(new DatePickerComponent);

$datePicker->selectDate(2019, 1, 30);

$datePicker->assertSee('January');
```

<a name="continuous-integration"></a>

## Continuous Integration

> [!WARNING]
> Hầu hết các cấu hình tích hợp liên tục của Dusk mong đợi ứng dụng Laravel của bạn được phục vụ bằng máy chủ phát triển PHP tích hợp trên cổng 8000. Do đó, trước khi tiếp tục, bạn nên đảm bảo rằng môi trường tích hợp liên tục của bạn có giá trị biến môi trường `APP_URL` là `http://127.0.0.1:8000`.

<a name="running-tests-on-heroku-ci"></a>

### Heroku CI

Để chạy các bài kiểm tra Dusk trên [Heroku CI](https://www.heroku.com/continuous-integration), thêm buildpack Google Chrome và script sau vào tệp `app.json` Heroku của bạn:

```json
{
  "environments": {
    "test": {
      "buildpacks": [
        { "url": "heroku/php" },
        {
          "url": "https://github.com/heroku/heroku-buildpack-chrome-for-testing"
        }
      ],
      "scripts": {
        "test-setup": "cp .env.testing .env",
        "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux --port=9515 > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
      }
    }
  }
}
```

<a name="running-tests-on-travis-ci"></a>

### Travis CI

Để chạy các bài kiểm tra Dusk của bạn trên [Travis CI](https://travis-ci.org), sử dụng cấu hình `.travis.yml` sau. Vì Travis CI không phải là môi trường đồ họa, chúng ta sẽ cần thực hiện một số bước bổ sung để khởi chạy trình duyệt Chrome. Ngoài ra, chúng ta sẽ sử dụng `php artisan serve` để khởi chạy máy chủ web tích hợp của PHP:

```yaml
language: php

php:
  - 8.2

addons:
  chrome: stable

install:
  - cp .env.testing .env
  - travis_retry composer install --no-interaction --prefer-dist
  - php artisan key:generate
  - php artisan dusk:chrome-driver

before_script:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
  - php artisan serve --no-reload &

script:
  - php artisan dusk
```

<a name="running-tests-on-github-actions"></a>

### GitHub Actions

Nếu bạn đang sử dụng [GitHub Actions](https://github.com/features/actions) để chạy các bài kiểm tra Dusk của bạn, bạn có thể sử dụng tệp cấu hình sau làm điểm bắt đầu. Giống như TravisCI, chúng ta sẽ sử dụng lệnh `php artisan serve` để khởi chạy máy chủ web tích hợp của PHP:

```yaml
name: CI
on: [push]
jobs:
  dusk-php:
    runs-on: ubuntu-latest
    env:
      APP_URL: "http://127.0.0.1:8000"
      DB_USERNAME: root
      DB_PASSWORD: root
      MAIL_MAILER: log
    steps:
      - uses: actions/checkout@v5
      - name: Prepare The Environment
        run: cp .env.example .env
      - name: Create Database
        run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE \`my-database\` character set UTF8mb4 collate utf8mb4_bin;"
      - name: Install Composer Dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader
      - name: Generate Application Key
        run: php artisan key:generate
      - name: Upgrade Chrome Driver
        run: php artisan dusk:chrome-driver --detect
      - name: Start Chrome Driver
        run: ./vendor/laravel/dusk/bin/chromedriver-linux --port=9515 &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: console
          path: tests/Browser/console
```

<a name="running-tests-on-chipper-ci"></a>

### Chipper CI

Nếu bạn đang sử dụng [Chipper CI](https://chipperci.com) để chạy các bài kiểm tra Dusk của bạn, bạn có thể sử dụng tệp cấu hình sau làm điểm bắt đầu. Chúng ta sẽ sử dụng máy chủ tích hợp của PHP để chạy Laravel để chúng ta có thể lắng nghe các yêu cầu:

```yaml
# file .chipperci.yml
version: 1

environment:
  php: 8.2
  node: 16

# Include Chrome in the build environment
services:
  - dusk

# Build all commits
on:
  push:
    branches: .*

pipeline:
  - name: Setup
    cmd: |
      cp -v .env.example .env
      composer install --no-interaction --prefer-dist --optimize-autoloader
      php artisan key:generate

      # Create a dusk env file, ensuring APP_URL uses BUILD_HOST
      cp -v .env .env.dusk.ci
      sed -i "s@APP_URL=.*@APP_URL=http://$BUILD_HOST:8000@g" .env.dusk.ci

  - name: Compile Assets
    cmd: |
      npm ci --no-audit
      npm run build

  - name: Browser Tests
    cmd: |
      php -S [::0]:8000 -t public 2>server.log &
      sleep 2
      php artisan dusk:chrome-driver $CHROME_DRIVER
      php artisan dusk --env=ci
```

Để tìm hiểu thêm về việc chạy các bài kiểm tra Dusk trên Chipper CI, bao gồm cách sử dụng cơ sở dữ liệu, hãy tham khảo [tài liệu Chipper CI chính thức](https://chipperci.com/docs/testing/laravel-dusk-new/).

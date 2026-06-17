# Testing: Getting Started

- [Introduction](#introduction)
- [Environment](#environment)
- [Creating Tests](#creating-tests)
- [Running Tests](#running-tests)
  - [Running Tests in Parallel](#running-tests-in-parallel)
  - [Reporting Test Coverage](#reporting-test-coverage)
  - [Profiling Tests](#profiling-tests)
- [Configuration Caching](#configuration-caching)

<a name="introduction"></a>

## Introduction

Laravel được xây dựng với tính năng testing trong tâm trí. Thực tế, hỗ trợ testing với [Pest](https://pestphp.com) và [PHPUnit](https://phpunit.de) được tích hợp sẵn và file `phpunit.xml` đã được thiết lập cho ứng dụng của bạn. Framework cũng cung cấp các helper method tiện lợi cho phép bạn test ứng dụng một cách rõ ràng.

Theo mặc định, thư mục `tests` của ứng dụng chứa hai thư mục: `Feature` và `Unit`. Unit test là các test tập trung vào một phần rất nhỏ, cô lập của code. Thực tế, hầu hết unit test có thể tập trung vào một single method. Các test trong thư mục "Unit" không khởi động ứng dụng Laravel của bạn và do đó không thể truy cập database hoặc các framework services khác của ứng dụng.

Feature test có thể test một phần lớn hơn của code, bao gồm cách các object tương tác với nhau hoặc thậm chí một HTTP request đầy đủ đến một JSON endpoint. **Nói chung, hầu hết các test của bạn nên là feature test. Các loại test này cung cấp sự tin cậy cao nhất rằng hệ thống của bạn nói chung đang hoạt động như dự định.**

File `ExampleTest.php` được cung cấp trong cả hai thư mục test `Feature` và `Unit`. Sau khi cài đặt một ứng dụng Laravel mới, thực thi lệnh `vendor/bin/pest`, `vendor/bin/phpunit`, hoặc `php artisan test` để chạy các test của bạn.

<a name="environment"></a>

## Environment

Khi chạy test, Laravel sẽ tự động thiết lập [configuration environment](/docs/{{version}}/configuration#environment-configuration) thành `testing` vì các environment variable được định nghĩa trong file `phpunit.xml`. Laravel cũng tự động cấu hình session và cache thành driver `array` để không có session hoặc cache data nào được lưu lại khi testing.

Bạn có thể tự do định nghĩa các giá trị configuration environment testing khác nếu cần. Các environment variable `testing` có thể được cấu hình trong file `phpunit.xml` của ứng dụng, nhưng hãy đảm bảo xóa configuration cache của bạn bằng lệnh Artisan `config:clear` trước khi chạy test!

<a name="the-env-testing-environment-file"></a>

#### The `.env.testing` Environment File

Ngoài ra, bạn có thể tạo file `.env.testing` trong root của project. File này sẽ được sử dụng thay cho file `.env` khi chạy Pest và PHPUnit test hoặc thực thi các lệnh Artisan với tùy chọn `--env=testing`.

<a name="creating-tests"></a>

## Creating Tests

Để tạo một test case mới, sử dụng lệnh Artisan `make:test`. Theo mặc định, test sẽ được đặt trong thư mục `tests/Feature`:

```shell
php artisan make:test UserTest
```

Nếu bạn muốn tạo một test trong thư mục `tests/Unit`, bạn có thể sử dụng tùy chọn `--unit` khi thực thi lệnh `make:test`:

```shell
php artisan make:test UserTest --unit
```

Nếu bạn có một test class chủ yếu dựa vào các tính năng testing của Laravel, nhưng một test method cụ thể không cần framework được khởi động, bạn có thể áp dụng attribute `#[UnitTest]` vào method đó để bỏ qua việc khởi động ứng dụng cho chỉ test đó.

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\Attributes\UnitTest;
use Tests\TestCase;

class LocationServiceTest extends TestCase
{
    public function test_get_coordinates_resolves_address(): void
    {
        // This test uses Laravel's testing features...
    }

    #[UnitTest]
    public function test_get_state_returns_state_from_abbreviation(): void
    {
        // This test runs without booting the application...
    }
}
```

> [!NOTE]
> Test stubs có thể được tùy chỉnh bằng [stub publishing](/docs/{{version}}/artisan#stub-customization).

Sau khi test đã được tạo, bạn có thể định nghĩa test như bình thường sử dụng Pest hoặc PHPUnit. Để chạy các test của bạn, thực thi lệnh `vendor/bin/pest`, `vendor/bin/phpunit`, hoặc `php artisan test` từ terminal:

```php
<?php

test('basic', function () {
    expect(true)->toBeTrue();
});
```

```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_basic_test(): void
    {
        $this->assertTrue(true);
    }
}
```

> [!WARNING]
> Nếu bạn định nghĩa các method `setUp` / `tearDown` của riêng mình trong một test class, hãy đảm bảo gọi các method `parent::setUp()` / `parent::tearDown()` tương ứng trên parent class. Thông thường, bạn nên gọi `parent::setUp()` ở đầu method `setUp` của riêng bạn, và `parent::tearDown()` ở cuối method `tearDown` của bạn.

<a name="running-tests"></a>

## Running Tests

Như đã đề cập trước đó, sau khi bạn đã viết test, bạn có thể chạy chúng bằng `pest` hoặc `phpunit`:

```shell tab=Pest
./vendor/bin/pest
```

```shell tab=PHPUnit
./vendor/bin/phpunit
```

Ngoài các lệnh `pest` hoặc `phpunit`, bạn có thể sử dụng lệnh Artisan `test` để chạy các test của bạn. Artisan test runner cung cấp các báo cáo test chi tiết để giúp phát triển và debug dễ dàng hơn:

```shell
php artisan test
```

Bất kỳ argument nào có thể được truyền cho lệnh `pest` hoặc `phpunit` cũng có thể được truyền cho lệnh Artisan `test`:

```shell
php artisan test --testsuite=Feature --stop-on-failure
```

<a name="running-tests-in-parallel"></a>

### Running Tests in Parallel

Theo mặc định, Laravel và Pest / PHPUnit thực thi các test của bạn tuần tự trong một single process. Tuy nhiên, bạn có thể giảm đáng kể thời gian chạy test bằng cách chạy test đồng thời trên nhiều process. Để bắt đầu, bạn nên cài đặt Composer package `brianium/paratest` như một dependency "dev". Sau đó, bao gồm tùy chọn `--parallel` khi thực thi lệnh Artisan `test`:

```shell
composer require brianium/paratest --dev

php artisan test --parallel
```

Theo mặc định, Laravel sẽ tạo nhiều process như số CPU core có sẵn trên máy của bạn. Tuy nhiên, bạn có thể điều chỉnh số lượng process bằng tùy chọn `--processes`:

```shell
php artisan test --parallel --processes=4
```

> [!WARNING]
> Khi chạy test song song, một số tùy chọn Pest / PHPUnit (như `--do-not-cache-result`) có thể không khả dụng.

<a name="parallel-testing-and-databases"></a>

#### Parallel Testing and Databases

Miễn là bạn đã cấu hình một primary database connection, Laravel tự động xử lý việc tạo và migrate một test database cho mỗi parallel process đang chạy test của bạn. Các test database sẽ được thêm hậu tố với một process token là duy nhất cho mỗi process. Ví dụ, nếu bạn có hai parallel test processes, Laravel sẽ tạo và sử dụng các test database `your_db_test_1` và `your_db_test_2`.

Theo mặc định, test database tồn tại giữa các lần gọi lệnh Artisan `test` để chúng có thể được sử dụng lại bởi các lần gọi `test` tiếp theo. Tuy nhiên, bạn có thể tạo lại chúng bằng tùy chọn `--recreate-databases`:

```shell
php artisan test --parallel --recreate-databases
```

<a name="parallel-testing-hooks"></a>

#### Parallel Testing Hooks

Thỉnh thoảng, bạn có thể cần chuẩn bị một số resources được sử dụng bởi test của ứng dụng để chúng có thể được sử dụng an toàn bởi nhiều test processes.

Sử dụng facade `ParallelTesting`, bạn có thể chỉ định code để thực thi trên `setUp` và `tearDown` của một process hoặc test case. Các closures được cung cấp nhận các biến `$token` và `$testCase` chứa process token và test case hiện tại, tương ứng:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\ParallelTesting;
use Illuminate\Support\ServiceProvider;
use PHPUnit\Framework\TestCase;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        ParallelTesting::setUpProcess(function (int $token) {
            // ...
        });

        ParallelTesting::setUpTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        // Executed when a test database is created...
        ParallelTesting::setUpTestDatabase(function (string $database, int $token) {
            Artisan::call('db:seed');
        });

        ParallelTesting::tearDownTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        ParallelTesting::tearDownProcess(function (int $token) {
            // ...
        });
    }
}
```

<a name="accessing-the-parallel-testing-token"></a>

#### Accessing the Parallel Testing Token

Nếu bạn muốn truy cập "token" của parallel process hiện tại từ bất kỳ vị trí nào khác trong test code của ứng dụng, bạn có thể sử dụng method `token`. Token này là một định danh chuỗi duy nhất cho một test process riêng lẻ và có thể được sử dụng để phân chia resources trên các parallel test processes. Ví dụ, Laravel tự động thêm token này vào cuối các test database được tạo bởi mỗi parallel testing process:

    $token = ParallelTesting::token();

<a name="reporting-test-coverage"></a>

### Reporting Test Coverage

> [!WARNING]
> Tính năng này yêu cầu [Xdebug](https://xdebug.org) hoặc [PCOV](https://pecl.php.net/package/pcov).

Khi chạy application test của bạn, bạn có thể muốn xác định xem test case của bạn có thực sự cover application code và bao nhiêu application code được sử dụng khi chạy test. Để thực hiện điều này, bạn có thể cung cấp tùy chọn `--coverage` khi gọi lệnh `test`:

```shell
php artisan test --coverage
```

<a name="enforcing-a-minimum-coverage-threshold"></a>

#### Enforcing a Minimum Coverage Threshold

Bạn có thể sử dụng tùy chọn `--min` để định nghĩa một minimum test coverage threshold cho ứng dụng của bạn. Test suite sẽ thất bại nếu threshold này không được đáp ứng:

```shell
php artisan test --coverage --min=80.3
```

<a name="profiling-tests"></a>

### Profiling Tests

Artisan test runner cũng bao gồm một cơ chế tiện lợi để liệt kê các test chậm nhất của ứng dụng. Gọi lệnh `test` với tùy chọn `--profile` để được hiển thị danh sách mười test chậm nhất của bạn, cho phép bạn dễ dàng điều tra các test nào có thể được cải thiện để tăng tốc test suite:

```shell
php artisan test --profile
```

<a name="configuration-caching"></a>

## Configuration Caching

Khi chạy test, Laravel khởi động ứng dụng cho mỗi test method riêng lẻ. Nếu không có file configuration được cache, mỗi configuration file trong ứng dụng của bạn phải được tải ở đầu của một test. Để xây dựng configuration một lần và tái sử dụng nó cho tất cả test trong một lần chạy, bạn có thể sử dụng trait `Illuminate\Foundation\Testing\WithCachedConfig`:

```php
<?php

use Illuminate\Foundation\Testing\WithCachedConfig;

pest()->use(WithCachedConfig::class);

// ...
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithCachedConfig;
use Tests\TestCase;

class ConfigTest extends TestCase
{
    use WithCachedConfig;

    // ...
}
```

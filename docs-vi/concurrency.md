# Concurrency

- [Giới thiệu](#introduction)
- [Chạy Concurrent Tasks](#running-concurrent-tasks)
    - [Named Results](#named-results)
    - [Task Timeouts](#task-timeouts)
- [Deferring Concurrent Tasks](#deferring-concurrent-tasks)

<a name="introduction"></a>
## Giới thiệu

Đôi khi bạn cần thực hiện một số slow tasks không phụ thuộc vào nhau. Trong nhiều trường hợp, significant performance improvements có thể được thực hiện bằng cách execute các tasks concurrently. `Concurrency` facade của Laravel cung cấp một simple, convenient API để execute closures concurrently.

<a name="how-it-works"></a>
#### Cách hoạt động

Laravel đạt được concurrency bằng cách serialize các closures đã cho và dispatch chúng đến một hidden Artisan CLI command, command này unserialize các closures và invoke nó trong PHP process riêng của nó. Sau khi closure đã được invoke, giá trị kết quả được serialize trở lại parent process.

`Concurrency` facade hỗ trợ ba drivers: `process` (mặc định), `fork`, và `sync`.

`fork` driver cung cấp improved performance so với default `process` driver, nhưng nó chỉ có thể được sử dụng trong PHP CLI context, vì PHP không hỗ trợ forking trong web requests. Trước khi sử dụng `fork` driver, bạn cần cài đặt `spatie/fork` package:

```shell
composer require spatie/fork
```

`sync` driver chủ yếu hữu ích trong testing khi bạn muốn disable tất cả concurrency và đơn giản execute các closures đã cho trong sequence trong parent process.

<a name="running-concurrent-tasks"></a>
## Chạy Concurrent Tasks

Để chạy concurrent tasks, bạn có thể invoke `run` method của `Concurrency` facade. `run` method chấp nhận một array của closures nên được execute simultaneously trong child PHP processes:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```

Để sử dụng một specific driver, bạn có thể sử dụng `driver` method:

```php
$results = Concurrency::driver('fork')->run(...);
```

Hoặc, để thay đổi default concurrency driver, bạn nên publish `concurrency` configuration file qua `config:publish` Artisan command và update `default` option trong file:

```shell
php artisan config:publish concurrency
```

<a name="named-results"></a>
### Named Results

Nếu bạn muốn access concurrent task results theo name thay vì theo position, bạn có thể cung cấp một associative array của closures. Mỗi result sẽ được trả về sử dụng cùng key với closure tương ứng của nó:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

$results = Concurrency::run([
    'users' => fn () => DB::table('users')->count(),
    'orders' => fn () => DB::table('orders')->count(),
]);

$userCount = $results['users'];
$orderCount = $results['orders'];
```

<a name="task-timeouts"></a>
### Task Timeouts

Khi sử dụng `process` driver (mặc định), bạn có thể specify một maximum number of seconds mà một concurrent task được phép chạy trước khi nó bị terminated bằng cách cung cấp một timeout cho `run` method:

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
], timeout: 30);
```

Bạn cũng có thể cung cấp một `CarbonInterval` instance nếu bạn thích một timeout definition expressive hơn:

```php
use Illuminate\Support\Facades\Concurrency;

use function Illuminate\Support\seconds;

Concurrency::run([...], timeout: seconds(30));
```

<a name="deferring-concurrent-tasks"></a>
## Deferring Concurrent Tasks

Nếu bạn muốn execute một array của closures concurrently, nhưng không quan tâm đến results được trả về bởi những closures đó, bạn nên cân nhắc sử dụng `defer` method. Khi `defer` method được invoke, các closures đã cho không được execute ngay lập tức. Thay vào đó, Laravel sẽ execute các closures concurrently sau khi HTTP response đã được gửi đến user:

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```

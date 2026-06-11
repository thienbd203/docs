# Laravel Telescope

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cài đặt chỉ cho Local](#local-only-installation)
    - [Cấu hình](#configuration)
    - [Dọn dẹp Dữ liệu](#data-pruning)
    - [Phân quyền Dashboard](#dashboard-authorization)
- [Nâng cấp Telescope](#upgrading-telescope)
- [Bộ lọc](#filtering)
    - [Entries](#filtering-entries)
    - [Batches](#filtering-batches)
- [Gắn thẻ](#tagging)
- [Các Watcher Có sẵn](#available-watchers)
    - [Batch Watcher](#batch-watcher)
    - [Cache Watcher](#cache-watcher)
    - [Command Watcher](#command-watcher)
    - [Dump Watcher](#dump-watcher)
    - [Event Watcher](#event-watcher)
    - [Exception Watcher](#exception-watcher)
    - [Gate Watcher](#gate-watcher)
    - [HTTP Client Watcher](#http-client-watcher)
    - [Job Watcher](#job-watcher)
    - [Log Watcher](#log-watcher)
    - [Mail Watcher](#mail-watcher)
    - [Model Watcher](#model-watcher)
    - [Notification Watcher](#notification-watcher)
    - [Query Watcher](#query-watcher)
    - [Redis Watcher](#redis-watcher)
    - [Request Watcher](#request-watcher)
    - [Schedule Watcher](#schedule-watcher)
    - [View Watcher](#view-watcher)
- [Hiển thị Avatar Người dùng](#displaying-user-avatars)

<a name="introduction"></a>
## Giới thiệu

[Laravel Telescope](https://github.com/laravel/telescope) là một công cụ hỗ trợ tuyệt vời cho môi trường phát triển Laravel local của bạn. Telescope cung cấp cái nhìn sâu sắc về các request đến ứng dụng của bạn, exceptions, log entries, database queries, queued jobs, mail, notifications, cache operations, scheduled tasks, variable dumps, và nhiều hơn nữa.

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>
## Cài đặt

Bạn có thể sử dụng Composer package manager để cài đặt Telescope vào dự án Laravel của mình:

```shell
composer require laravel/telescope
```

Sau khi cài đặt Telescope, hãy publish các assets và migrations của nó bằng lệnh Artisan `telescope:install`. Sau khi cài đặt Telescope, bạn cũng nên chạy lệnh `migrate` để tạo các bảng cần thiết để lưu trữ dữ liệu của Telescope:

```shell
php artisan telescope:install

php artisan migrate
```

Cuối cùng, bạn có thể truy cập Telescope dashboard qua route `/telescope`.

<a name="local-only-installation"></a>
### Cài đặt chỉ cho Local

Nếu bạn chỉ định sử dụng Telescope để hỗ trợ phát triển local, bạn có thể cài đặt Telescope bằng flag `--dev`:

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

Sau khi chạy `telescope:install`, bạn nên xóa đăng ký service provider `TelescopeServiceProvider` khỏi file cấu hình `bootstrap/providers.php` của ứng dụng. Thay vào đó, hãy đăng ký thủ công các service providers của Telescope trong phương thức `register` của class `App\Providers\AppServiceProvider`. Chúng ta sẽ đảm bảo môi trường hiện tại là `local` trước khi đăng ký các providers:

```php
/**
 * Đăng ký bất kỳ application services nào.
 */
public function register(): void
{
    if ($this->app->environment('local') && class_exists(\Laravel\Telescope\TelescopeServiceProvider::class)) {
        $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
        $this->app->register(TelescopeServiceProvider::class);
    }
}
```

Cuối cùng, bạn cũng nên ngăn package Telescope bị [auto-discovered](/docs/{{version}}/packages#package-discovery) bằng cách thêm sau vào file `composer.json` của bạn:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "laravel/telescope"
        ]
    }
},
```

<a name="configuration"></a>
### Cấu hình

Sau khi publish các assets của Telescope, file cấu hình chính của nó sẽ nằm tại `config/telescope.php`. File cấu hình này cho phép bạn cấu hình [các tùy chọn watcher](#available-watchers). Mỗi tùy chọn cấu hình bao gồm mô tả về mục đích của nó, vì vậy hãy đảm bảo khám phá kỹ file này.

Nếu muốn, bạn có thể tắt hoàn toàn việc thu thập dữ liệu của Telescope bằng tùy chọn cấu hình `enabled`:

```php
'enabled' => env('TELESCOPE_ENABLED', true),
```

<a name="data-pruning"></a>
### Dọn dẹp Dữ liệu

Nếu không dọn dẹp, bảng `telescope_entries` có thể tích lũy các bản ghi rất nhanh. Để giảm thiểu điều này, bạn nên [schedule](/docs/{{version}}/scheduling) lệnh Artisan `telescope:prune` để chạy hàng ngày:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune')->daily();
```

Theo mặc định, tất cả các entry cũ hơn 24 giờ sẽ bị dọn dẹp. Bạn có thể sử dụng tùy chọn `hours` khi gọi lệnh để xác định thời gian lưu giữ dữ liệu Telescope. Ví dụ, lệnh sau sẽ xóa tất cả các bản ghi được tạo hơn 48 giờ trước:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune --hours=48')->daily();
```

<a name="dashboard-authorization"></a>
### Phân quyền Dashboard

Telescope dashboard có thể được truy cập qua route `/telescope`. Theo mặc định, bạn chỉ có thể truy cập dashboard này trong môi trường `local`. Trong file `app/Providers/TelescopeServiceProvider.php` của bạn, có định nghĩa [authorization gate](/docs/{{version}}/authorization#gates). Authorization gate này kiểm soát quyền truy cập vào Telescope trong các môi trường **non-local**. Bạn có thể tự do sửa đổi gate này khi cần thiết để hạn chế quyền truy cập vào cài đặt Telescope của mình:

```php
use App\Models\User;

/**
 * Đăng ký Telescope gate.
 *
 * Gate này xác định ai có thể truy cập Telescope trong các môi trường non-local.
 */
protected function gate(): void
{
    Gate::define('viewTelescope', function (User $user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

> [!WARNING]
> Bạn nên đảm bảo thay đổi biến môi trường `APP_ENV` của bạn thành `production` trong môi trường production. Nếu không, cài đặt Telescope của bạn sẽ có sẵn công khai.

<a name="upgrading-telescope"></a>
## Nâng cấp Telescope

Khi nâng cấp lên phiên bản major mới của Telescope, điều quan trọng là bạn phải xem xét kỹ [hướng dẫn nâng cấp](https://github.com/laravel/telescope/blob/master/UPGRADE.md).

Ngoài ra, khi nâng cấp lên bất kỳ phiên bản Telescope mới nào, bạn nên publish lại các assets của Telescope:

```shell
php artisan telescope:publish
```

Để giữ cho các assets luôn cập nhật và tránh các vấn đề trong các bản cập nhật trong tương lai, bạn có thể thêm lệnh `vendor:publish --tag=laravel-assets` vào các script `post-update-cmd` trong file `composer.json` của ứng dụng:

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

<a name="filtering"></a>
## Bộ lọc

<a name="filtering-entries"></a>
### Entries

Bạn có thể lọc dữ liệu được ghi lại bởi Telescope thông qua closure `filter` được định nghĩa trong class `App\Providers\TelescopeServiceProvider` của bạn. Theo mặc định, closure này ghi lại tất cả dữ liệu trong môi trường `local` và exceptions, failed jobs, scheduled tasks, và dữ liệu với các monitored tags trong tất cả các môi trường khác:

```php
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Đăng ký bất kỳ application services nào.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filter(function (IncomingEntry $entry) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entry->isReportableException() ||
            $entry->isFailedJob() ||
            $entry->isScheduledTask() ||
            $entry->isSlowQuery() ||
            $entry->hasMonitoredTag();
    });
}
```

<a name="filtering-batches"></a>
### Batches

Trong khi closure `filter` lọc dữ liệu cho các entry riêng lẻ, bạn có thể sử dụng phương thức `filterBatch` để đăng ký một closure lọc tất cả dữ liệu cho một request hoặc console command cụ thể. Nếu closure trả về `true`, tất cả các entry sẽ được ghi lại bởi Telescope:

```php
use Illuminate\Support\Collection;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Đăng ký bất kỳ application services nào.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filterBatch(function (Collection $entries) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entries->contains(function (IncomingEntry $entry) {
            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->isSlowQuery() ||
                $entry->hasMonitoredTag();
            });
    });
}
```

<a name="tagging"></a>
## Gắn thẻ

Telescope cho phép bạn tìm kiếm các entry theo "tag". Thường thì tags là tên class Eloquent model hoặc authenticated user IDs mà Telescope tự động thêm vào các entry. Đôi khi, bạn có thể muốn gắn các custom tags của riêng mình vào các entry. Để thực hiện điều này, bạn có thể sử dụng phương thức `Telescope::tag`. Phương thức `tag` chấp nhận một closure nên trả về một mảng các tags. Các tags được trả về bởi closure sẽ được hợp nhất với bất kỳ tags nào mà Telescope sẽ tự động gắn vào entry. Thông thường, bạn nên gọi phương thức `tag` trong phương thức `register` của class `App\Providers\TelescopeServiceProvider` của bạn:

```php
use Laravel\Telescope\EntryType;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Đăng ký bất kỳ application services nào.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::tag(function (IncomingEntry $entry) {
        return $entry->type === EntryType::REQUEST
            ? ['status:'.$entry->content['response_status']]
            : [];
    });
}
```

<a name="available-watchers"></a>
## Các Watcher Có sẵn

Telescope "watchers" thu thập dữ liệu ứng dụng khi một request hoặc console command được thực thi. Bạn có thể tùy chỉnh danh sách các watchers mà bạn muốn bật trong file cấu hình `config/telescope.php` của mình:

```php
'watchers' => [
    Watchers\CacheWatcher::class => true,
    Watchers\CommandWatcher::class => true,
    // ...
],
```

Một số watchers cũng cho phép bạn cung cấp các tùy chọn tùy chỉnh bổ sung:

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 100,
    ],
    // ...
],
```

<a name="batch-watcher"></a>
### Batch Watcher

Batch watcher ghi lại thông tin về các queued [batches](/docs/{{version}}/queues#job-batching), bao gồm thông tin về job và connection.

<a name="cache-watcher"></a>
### Cache Watcher

Cache watcher ghi lại dữ liệu khi một cache key được hit, missed, updated và forgotten.

<a name="command-watcher"></a>
### Command Watcher

Command watcher ghi lại các arguments, options, exit code, và output bất cứ khi nào một lệnh Artisan được thực thi. Nếu bạn muốn loại trừ một số lệnh nhất định khỏi việc được ghi lại bởi watcher, bạn có thể chỉ định lệnh trong tùy chọn `ignore` trong file `config/telescope.php` của mình:

```php
'watchers' => [
    Watchers\CommandWatcher::class => [
        'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
        'ignore' => ['key:generate'],
    ],
    // ...
],
```

<a name="dump-watcher"></a>
### Dump Watcher

Dump watcher ghi lại và hiển thị các variable dumps của bạn trong Telescope. Khi sử dụng Laravel, các biến có thể được dump bằng hàm global `dump`. Tab dump watcher phải được mở trong trình duyệt để dump được ghi lại, nếu không, các dumps sẽ bị watcher bỏ qua.

<a name="event-watcher"></a>
### Event Watcher

Event watcher ghi lại payload, listeners, và broadcast data cho bất kỳ [events](/docs/{{version}}/events) nào được dispatch bởi ứng dụng của bạn. Các internal events của Laravel framework bị Event watcher bỏ qua.

<a name="exception-watcher"></a>
### Exception Watcher

Exception watcher ghi lại dữ liệu và stack trace cho bất kỳ reportable exceptions nào được throw bởi ứng dụng của bạn.

<a name="gate-watcher"></a>
### Gate Watcher

Gate watcher ghi lại dữ liệu và kết quả của các kiểm tra [gate và policy](/docs/{{version}}/authorization) bởi ứng dụng của bạn. Nếu bạn muốn loại trừ một số abilities nhất định khỏi việc được ghi lại bởi watcher, bạn có thể chỉ định chúng trong tùy chọn `ignore_abilities` trong file `config/telescope.php` của mình:

```php
'watchers' => [
    Watchers\GateWatcher::class => [
        'enabled' => env('TELESCOPE_GATE_WATCHER', true),
        'ignore_abilities' => ['viewNova'],
    ],
    // ...
],
```

<a name="http-client-watcher"></a>
### HTTP Client Watcher

HTTP client watcher ghi lại các [HTTP client requests](/docs/{{version}}/http-client) outgoing được thực hiện bởi ứng dụng của bạn.

<a name="job-watcher"></a>
### Job Watcher

Job watcher ghi lại dữ liệu và trạng thái của bất kỳ [jobs](/docs/{{version}}/queues) nào được dispatch bởi ứng dụng của bạn.

<a name="log-watcher"></a>
### Log Watcher

Log watcher ghi lại [log data](/docs/{{version}}/logging) cho bất kỳ logs nào được viết bởi ứng dụng của bạn.

Theo mặc định, Telescope sẽ chỉ ghi lại logs ở mức `error` trở lên. Tuy nhiên, bạn có thể sửa đổi tùy chọn `level` trong file cấu hình `config/telescope.php` của ứng dụng để sửa đổi hành vi này:

```php
'watchers' => [
    Watchers\LogWatcher::class => [
        'enabled' => env('TELESCOPE_LOG_WATCHER', true),
        'level' => 'debug',
    ],

    // ...
],
```

<a name="mail-watcher"></a>
### Mail Watcher

Mail watcher cho phép bạn xem preview trong trình duyệt của các [emails](/docs/{{version}}/mail) được gửi bởi ứng dụng của bạn cùng với dữ liệu liên quan của chúng. Bạn cũng có thể tải xuống email dưới dạng file `.eml`.

<a name="model-watcher"></a>
### Model Watcher

Model watcher ghi lại các thay đổi model bất cứ khi nào một [model event](/docs/{{version}}/eloquent#events) của Eloquent được dispatch. Bạn có thể chỉ định các model events nào nên được ghi lại thông qua tùy chọn `events` của watcher:

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
    ],
    // ...
],
```

Nếu bạn muốn ghi lại số lượng models được hydrated trong một request cụ thể, hãy bật tùy chọn `hydrations`:

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
        'hydrations' => true,
    ],
    // ...
],
```

<a name="notification-watcher"></a>
### Notification Watcher

Notification watcher ghi lại tất cả [notifications](/docs/{{version}}/notifications) được gửi bởi ứng dụng của bạn. Nếu notification kích hoạt một email và bạn đã bật mail watcher, email cũng sẽ có sẵn để preview trên màn hình mail watcher.

<a name="query-watcher"></a>
### Query Watcher

Query watcher ghi lại raw SQL, bindings, và execution time cho tất cả các queries được thực thi bởi ứng dụng của bạn. Watcher cũng gắn thẻ bất kỳ queries nào chậm hơn 100 mili-giây là `slow`. Bạn có thể tùy chỉnh ngưỡng slow query bằng tùy chọn `slow` của watcher:

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 50,
    ],
    // ...
],
```

<a name="redis-watcher"></a>
### Redis Watcher

Redis watcher ghi lại tất cả các lệnh [Redis](/docs/{{version}}/redis) được thực thi bởi ứng dụng của bạn. Nếu bạn sử dụng Redis cho caching, các lệnh cache cũng sẽ được ghi lại bởi Redis watcher.

<a name="request-watcher"></a>
### Request Watcher

Request watcher ghi lại request, headers, session, và response data liên quan đến bất kỳ requests nào được xử lý bởi ứng dụng. Bạn có thể giới hạn response data được ghi lại của mình thông qua tùy chọn `size_limit` (tính bằng kilobyte):

```php
'watchers' => [
    Watchers\RequestWatcher::class => [
        'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
        'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
    ],
    // ...
],
```

<a name="schedule-watcher"></a>
### Schedule Watcher

Schedule watcher ghi lại lệnh và output của bất kỳ [scheduled tasks](/docs/{{version}}/scheduling) nào được chạy bởi ứng dụng của bạn.

<a name="view-watcher"></a>
### View Watcher

View watcher ghi lại tên [view](/docs/{{version}}/views), path, data, và "composers" được sử dụng khi render views.

<a name="displaying-user-avatars"></a>
## Hiển thị Avatar Người dùng

Telescope dashboard hiển thị avatar người dùng cho người dùng đã được authenticated khi một entry cụ thể được lưu. Theo mặc định, Telescope sẽ lấy avatars bằng dịch vụ web Gravatar. Tuy nhiên, bạn có thể tùy chỉnh URL avatar bằng cách đăng ký một callback trong class `App\Providers\TelescopeServiceProvider` của mình. Callback sẽ nhận ID và địa chỉ email của người dùng và nên trả về URL hình ảnh avatar của người dùng:

```php
use App\Models\User;
use Laravel\Telescope\Telescope;

/**
 * Đăng ký bất kỳ application services nào.
 */
public function register(): void
{
    // ...

    Telescope::avatar(function (?string $id, ?string $email) {
        return ! is_null($id)
            ? '/avatars/'.User::find($id)->avatar_path
            : '/generic-avatar.jpg';
    });
}
```

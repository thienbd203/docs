# Upgrade Guide

- [Upgrading To 13.0 From 12.x](#upgrade-13.0)
    - [Upgrading Using AI](#upgrading-using-ai)

<a name="high-impact-changes"></a>
## High Impact Changes

<div class="content-list" markdown="1">

- [Updating Dependencies](#updating-dependencies)
- [Updating the Laravel Installer](#updating-the-laravel-installer)
- [Request Forgery Protection](#request-forgery-protection)

</div>

<a name="medium-impact-changes"></a>
## Medium Impact Changes

<div class="content-list" markdown="1">

- [Cache `serializable_classes` Configuration](#cache-serializable_classes-configuration)
- [Database `upsert` With MySQL or MariaDB](#database-upsert-mariadb-mysql)

</div>

<a name="low-impact-changes"></a>
## Low Impact Changes

<div class="content-list" markdown="1">

- [Cache Prefixes and Session Cookie Names](#cache-prefixes-and-session-cookie-names)
- [Collection Model Serialization Restores Eager-Loaded Relations](#collection-model-serialization-restores-eager-loaded-relations)
- [`Container::call` and Nullable Class Defaults](#containercall-and-nullable-class-defaults)
- [Domain Route Registration Precedence](#domain-route-registration-precedence)
- [`JobAttempted` Event Exception Payload](#jobattempted-event-exception-payload)
- [Manager `extend` Callback Binding](#manager-extend-callback-binding)
- [MySQL `DELETE` Queries With `JOIN`, `ORDER BY`, and `LIMIT`](#mysql-delete-queries-with-join-order-by-and-limit)
- [Pagination Bootstrap View Names](#pagination-bootstrap-view-names)
- [Polymorphic Pivot Table Name Generation](#polymorphic-pivot-table-name-generation)
- [`QueueBusy` Event Property Rename](#queuebusy-event-property-rename)
- [`Str` Factories Reset Between Tests](#str-factories-reset-between-tests)

</div>

<a name="upgrade-13.0"></a>
## Upgrading To 13.0 From 12.x

#### Estimated Upgrade Time: 10 Minutes

> [!NOTE]
> Chúng tôi cố gắng tài liệu hóa mọi breaking change có thể. Vì một số breaking changes này nằm trong các phần mờ nhạt của framework, chỉ một phần các thay đổi này thực sự có thể ảnh hưởng đến ứng dụng của bạn. Để tiết kiệm thời gian, bạn có thể sử dụng [Shift](https://laravelshift.com). Shift là một dịch vụ được duy trì bởi cộng đồng tự động hóa các nâng cấp Laravel.

<a name="upgrading-using-ai"></a>
### Upgrading Using AI

Bạn có thể tự động hóa nâng cấp của mình bằng cách sử dụng [Laravel Boost](https://github.com/laravel/boost). Boost là một MCP server chính thức cung cấp cho AI assistant của bạn các prompts nâng cấp có hướng dẫn — sau khi cài đặt trong bất kỳ ứng dụng Laravel 12 nào, sử dụng lệnh slash `/upgrade-laravel-v13` trong Claude Code, Cursor, OpenCode, Gemini, hoặc VS Code để bắt đầu nâng cấp lên Laravel 13. Lệnh này yêu cầu Laravel Boost `^2.0`.

<a name="updating-dependencies"></a>
### Updating Dependencies

**Likelihood Of Impact: High**

Bạn nên cập nhật các dependencies sau trong file `composer.json` của ứng dụng:

<div class="content-list" markdown="1">

- `laravel/framework` thành `^13.0`
- `laravel/boost` thành `^2.0`
- `laravel/tinker` thành `^3.0`
- `phpunit/phpunit` thành `^12.0`
- `pestphp/pest` thành `^4.0`

</div>

<a name="updating-the-laravel-installer"></a>
### Updating the Laravel Installer

Nếu bạn đang sử dụng công cụ CLI Laravel installer để tạo các ứng dụng Laravel mới, bạn nên cập nhật cài đặt installer của bạn để tương thích với Laravel 13.x.

Nếu bạn đã cài đặt Laravel installer thông qua `composer global require`, bạn có thể cập nhật installer bằng cách sử dụng `composer global update`:

```shell
composer global update laravel/installer
```

Hoặc, nếu bạn đang sử dụng bản sao Laravel installer được gói sẵn của [Laravel Herd's](https://herd.laravel.com), bạn nên cập nhật cài đặt Herd của bạn lên bản phát hành mới nhất.

<a name="cache"></a>
### Cache

<a name="cache-prefixes-and-session-cookie-names"></a>
#### Cache Prefixes and Session Cookie Names

**Likelihood Of Impact: Low**

Các tiền tố cache và Redis mặc định của Laravel hiện sử dụng các hậu tố có dấu gạch ngang.

Trong hầu hết các ứng dụng, thay đổi này sẽ không áp dụng vì các file cấu hình cấp ứng dụng đã định nghĩa các giá trị này. Điều này chủ yếu ảnh hưởng đến các ứng dụng phụ thuộc vào cấu hình dự phòng cấp framework khi các giá trị cấu hình ứng dụng tương ứng không có mặt.

Nếu ứng dụng của bạn phụ thuộc vào các mặc định được tạo này, các khóa cache và tên cookie session có thể thay đổi sau khi nâng cấp:

```php
// Laravel <= 12.x
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_cache_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_database_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_session';

// Laravel >= 13.x
Str::slug((string) env('APP_NAME', 'laravel')).'-cache-';
Str::slug((string) env('APP_NAME', 'laravel')).'-database-';
Str::slug((string) env('APP_NAME', 'laravel')).'-session';
```

Để giữ hành vi trước đó, cấu hình rõ ràng `CACHE_PREFIX`, `REDIS_PREFIX`, và `SESSION_COOKIE` trong môi trường của bạn.

<a name="store-and-repository-contracts-touch"></a>
#### `Store` and `Repository` Contracts: `touch`

**Likelihood Of Impact: Very Low**

Các contracts cache hiện bao gồm một phương thức `touch` để mở rộng TTLs của các mục. Nếu bạn duy trì các triển khai cache store tùy chỉnh, bạn nên thêm phương thức này:

```php
// Illuminate\Contracts\Cache\Store
public function touch($key, $seconds);
```

<a name="cache-serializable_classes-configuration"></a>
#### Cache `serializable_classes` Configuration

**Likelihood Of Impact: Medium**

Cấu hình cache ứng dụng mặc định hiện bao gồm một tùy chọn `serializable_classes` được đặt thành `false`. Điều này làm cứng hành vi unserialization cache để giúp ngăn chặn các cuộc tấn công chuỗi gadget deserialization PHP nếu `APP_KEY` của ứng dụng bị rò rỉ. Nếu ứng dụng của bạn lưu trữ có chủ ý các đối tượng PHP trong cache, bạn nên liệt kê rõ ràng các classes có thể được unserialized:

```php
'serializable_classes' => [
    App\Data\CachedDashboardStats::class,
    App\Support\CachedPricingSnapshot::class,
],
```

Nếu ứng dụng của bạn trước đây phụ thuộc vào unserializing các đối tượng cache tùy ý, bạn sẽ cần di chuyển việc sử dụng đó sang các danh sách cho phép class rõ ràng hoặc các payloads cache không phải đối tượng (như arrays).

<a name="container"></a>
### Container

<a name="containercall-and-nullable-class-defaults"></a>
#### `Container::call` and Nullable Class Defaults

**Likelihood Of Impact: Low**

`Container::call` hiện tôn trọng các mặc định tham số class nullable khi không có binding nào tồn tại, khớp với hành vi injection constructor được giới thiệu trong Laravel 12:

```php
$container->call(function (?Carbon $date = null) {
    return $date;
});

// Laravel <= 12.x: Carbon instance
// Laravel >= 13.x: null
```

Nếu logic injection cuộc gọi phương thức của bạn phụ thuộc vào hành vi trước đó, bạn có thể cần cập nhật nó.

<a name="contracts"></a>
### Contracts

<a name="dispatcher-contract-dispatchafterresponse"></a>
#### `Dispatcher` Contract: `dispatchAfterResponse`

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Bus\Dispatcher` hiện bao gồm phương thức `dispatchAfterResponse($command, $handler = null)`.

Nếu bạn duy trì một triển khai dispatcher tùy chỉnh, thêm phương thức này vào lớp của bạn.

<a name="responsefactory-contract-eventstream"></a>
#### `ResponseFactory` Contract: `eventStream`

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Routing\ResponseFactory` hiện bao gồm một signature `eventStream`.

Nếu bạn duy trì một triển khai tùy chỉnh của contract này, bạn nên thêm phương thức này.

<a name="mustverifyemail-contract-markemailasunverified"></a>
#### `MustVerifyEmail` Contract: `markEmailAsUnverified`

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Auth\MustVerifyEmail` hiện bao gồm `markEmailAsUnverified()`.

Nếu bạn cung cấp một triển khai tùy chỉnh của contract này, thêm phương thức này để vẫn tương thích.

<a name="database"></a>
### Database

<a name="database-upsert-mariadb-mysql"></a>
#### Database `upsert` With MySQL or MariaDB

**Likelihood Of Impact: Medium**

Laravel hiện xác minh rằng người gọi cung cấp một giá trị không rỗng cho `uniqueBy`, và sẽ ném một `InvalidArgumentException` thay vì tạo SQL không hợp lệ.

Mặc dù các drivers database MariaDB và MySQL bỏ qua giá trị `uniqueBy` và luôn sử dụng các indexes primary và unique của bảng để phát hiện các bản ghi hiện có, xác thực vẫn áp dụng. Một `InvalidArgumentException` sẽ được ném nếu `uniqueBy` trống.

<a name="mysql-delete-queries-with-join-order-by-and-limit"></a>
#### MySQL `DELETE` Queries With `JOIN`, `ORDER BY`, and `LIMIT`

**Likelihood Of Impact: Low**

Laravel hiện biên dịch các truy vấn `DELETE ... JOIN` đầy đủ bao gồm `ORDER BY` và `LIMIT` cho ngữ pháp MySQL.

Trong các phiên bản trước, các mệnh đề `ORDER BY` / `LIMIT` có thể bị bỏ qua âm thầm trên các xóa được join. Trong Laravel 13, các mệnh đề này được bao gồm trong SQL được tạo. Kết quả là, các engine database không hỗ trợ cú pháp này (như các biến thể MySQL / MariaDB tiêu chuẩn) hiện có thể ném một `QueryException` thay vì thực hiện một xóa không giới hạn.

<a name="eloquent"></a>
### Eloquent

<a name="model-booting-and-nested-instantiation"></a>
#### Model Booting and Nested Instantiation

**Likelihood Of Impact: Very Low**

Tạo một instance model mới trong khi model đó vẫn đang booting hiện không được phép và ném một `LogicException`.

Điều này ảnh hưởng đến mã khởi tạo các models từ bên trong các phương thức `boot` model hoặc các phương thức `boot*` trait:

```php
protected static function boot()
{
    parent::boot();

    // No longer allowed during booting...
    (new static())->getTable();
}
```

Di chuyển logic này ra khỏi chu kỳ boot để tránh booting lồng nhau.

<a name="polymorphic-pivot-table-name-generation"></a>
#### Polymorphic Pivot Table Name Generation

**Likelihood Of Impact: Low**

Khi tên bảng được suy ra cho các models pivot polymorphic bằng cách sử dụng các lớp model pivot tùy chỉnh, Laravel hiện tạo ra các tên số nhiều.

Nếu ứng dụng của bạn phụ thuộc vào các tên số ít được suy ra trước đó cho các bảng pivot morph và sử dụng các lớp pivot tùy chỉnh, bạn nên định nghĩa rõ ràng tên bảng trên model pivot của bạn.

<a name="collection-model-serialization-restores-eager-loaded-relations"></a>
#### Collection Model Serialization Restores Eager-Loaded Relations

**Likelihood Of Impact: Low**

Khi các collections model Eloquent được serialized và khôi phục (như trong các queued jobs), các relationships được eager-load hiện được khôi phục cho các models của collection.

Nếu mã của bạn phụ thuộc vào các relationships không có mặt sau khi deserialization, bạn có thể cần điều chỉnh logic đó.

<a name="http-client"></a>
### HTTP Client

<a name="http-client-response-throw-and-throwif-signatures"></a>
#### HTTP Client `Response::throw` and `throwIf` Signatures

**Likelihood Of Impact: Very Low**

Các phương thức response HTTP client hiện khai báo các tham số callback của chúng trong các signatures phương thức:

```php
public function throw($callback = null);
public function throwIf($condition, $callback = null);
```

Nếu bạn ghi đè các phương thức này trong các lớp response tùy chỉnh, đảm bảo các signatures phương thức của bạn tương thích.

<a name="notifications"></a>
### Notifications

<a name="default-password-reset-subject"></a>
#### Default Password Reset Subject

**Likelihood Of Impact: Very Low**

Chủ đề mail reset mật khẩu mặc định của Laravel đã thay đổi:

```text
// Laravel <= 12.x
Reset Password Notification

// Laravel >= 13.x
Reset your password
```

Nếu các tests, assertions, hoặc các ghi đè dịch của bạn phụ thuộc vào chuỗi mặc định trước đó, cập nhật chúng tương ứng.

<a name="queued-notifications-and-missing-models"></a>
#### Queued Notifications and Missing Models

**Likelihood Of Impact: Very Low**

Các notifications queued hiện tôn trọng attribute `#[DeleteWhenMissingModels]` và thuộc tính `$deleteWhenMissingModels` được định nghĩa trên lớp notification.

Trong các phiên bản trước, các models bị thiếu vẫn có thể gây ra các job notification queued thất bại trong các trường hợp bạn mong đợi chúng bị xóa.

<a name="queue"></a>
### Queue

<a name="jobattempted-event-exception-payload"></a>
#### `JobAttempted` Event Exception Payload

**Likelihood Of Impact: Low**

Event `Illuminate\Queue\Events\JobAttempted` hiện phơi bày đối tượng exception (hoặc `null`) thông qua `$exception`, thay thế thuộc tính boolean `$exceptionOccurred` trước đó:

```php
// Laravel <= 12.x
$event->exceptionOccurred;

// Laravel >= 13.x
$event->exception;
```

Nếu bạn lắng nghe event này, cập nhật mã listener của bạn tương ứng.

<a name="queuebusy-event-property-rename"></a>
#### `QueueBusy` Event Property Rename

**Likelihood Of Impact: Low**

Thuộc tính `$connection` của event `Illuminate\Queue\Events\QueueBusy` đã được đổi tên thành `$connectionName` để nhất quán với các event queue khác.

Nếu các listeners của bạn tham chiếu `$connection`, cập nhật chúng thành `$connectionName`.

<a name="queue-contract-method-additions"></a>
#### `Queue` Contract Method Additions

**Likelihood Of Impact: Very Low**

Contract `Illuminate\Contracts\Queue\Queue` hiện bao gồm các phương thức kiểm tra kích thước queue trước đây chỉ được khai báo trong docblocks.

Nếu bạn duy trì các triển khai driver queue tùy chỉnh của contract này, thêm các triển khai cho:

<div class="content-list" markdown="1">

- `pendingSize`
- `delayedSize`
- `reservedSize`
- `creationTimeOfOldestPendingJob`

</div>

<a name="routing"></a>
### Routing

<a name="domain-route-registration-precedence"></a>
#### Domain Route Registration Precedence

**Likelihood Of Impact: Low**

Các routes với một domain rõ ràng hiện được ưu tiên trước các routes không có domain trong route matching.

Điều này cho phép các routes subdomain catch-all hoạt động nhất quán ngay cả khi các routes không có domain được đăng ký trước đó. Nếu ứng dụng của bạn phụ thuộc vào mức độ ưu tiên đăng ký trước đó giữa các routes có domain và không có domain, xem xét hành vi route matching.

<a name="scheduling"></a>
### Scheduling

<a name="withscheduling-registration-timing"></a>
#### `withScheduling` Registration Timing

**Likelihood Of Impact: Very Low**

Các schedules được đăng ký thông qua `ApplicationBuilder::withScheduling()` hiện được trì hoãn cho đến khi `Schedule` được giải quyết.

Nếu ứng dụng của bạn phụ thuộc vào thời gian đăng ký schedule ngay lập tức trong bootstrap, bạn có thể cần điều chỉnh logic đó.

<a name="security"></a>
### Security

<a name="request-forgery-protection"></a>
#### Request Forgery Protection

**Likelihood Of Impact: High**

Middleware CSRF của Laravel đã được đổi tên từ `VerifyCsrfToken` thành `PreventRequestForgery`, và hiện bao gồm xác minh request-origin sử dụng header `Sec-Fetch-Site`.

`VerifyCsrfToken` và `ValidateCsrfToken` vẫn là các aliases đã deprecated, nhưng các tham chiếu trực tiếp nên được cập nhật thành `PreventRequestForgery`, đặc biệt khi loại trừ middleware trong các tests hoặc định nghĩa route:

```php
use Illuminate\Foundation\Http\Middleware\PreventRequestForgery;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;

// Laravel <= 12.x
->withoutMiddleware([VerifyCsrfToken::class]);

// Laravel >= 13.x
->withoutMiddleware([PreventRequestForgery::class]);
```

API cấu hình middleware hiện cũng cung cấp `preventRequestForgery(...)`.

<a name="support"></a>
### Support

<a name="manager-extend-callback-binding"></a>
#### Manager `extend` Callback Binding

**Likelihood Of Impact: Low**

Các closures driver tùy chỉnh được đăng ký thông qua các phương thức manager `extend` hiện được bound đến instance manager.

Nếu bạn trước đây phụ thuộc vào một đối tượng bound khác (như một instance service provider) làm `$this` bên trong các closures này, bạn nên chuyển các giá trị đó vào các captures closure bằng cách sử dụng `use (...)`.

<a name="str-factories-reset-between-tests"></a>
#### `Str` Factories Reset Between Tests

**Likelihood Of Impact: Low**

Laravel hiện đặt lại các factories `Str` tùy chỉnh trong quá trình teardown test.

Nếu các tests của bạn phụ thuộc vào các factories UUID / ULID / chuỗi ngẫu nhiên tùy chỉnh tồn tại giữa các phương thức test, bạn nên đặt chúng trong mỗi test hoặc hook setup liên quan.

<a name="jsfrom-uses-unescaped-unicode-by-default"></a>
#### `Js::from` Uses Unescaped Unicode By Default

**Likelihood Of Impact: Very Low**

`Illuminate\Support\Js::from` hiện sử dụng `JSON_UNESCAPED_UNICODE` theo mặc định.

Nếu các tests hoặc so sánh output frontend của bạn phụ thuộc vào các chuỗi Unicode đã thoát (ví dụ `\u00e8`), cập nhật các kỳ vọng của bạn.

<a name="utilities"></a>
### Utilities

<a name="symfony-polyfill"></a>
#### Symfony PHP 8.5 Polyfill and Global Function Conflicts

**Likelihood Of Impact: Low**

Laravel 13 giới thiệu một dependency trên `symfony/polyfill-php85`. Trên các phiên bản PHP dưới 8.5, polyfill này định nghĩa các hàm toàn cầu như `array_first()` và `array_last()` trừ khi chúng đã được định nghĩa trước đó trong bootstrap.

Các hàm này có thể xung đột với các packages helper cũ như `laravel/helpers` hoặc các helpers toàn cầu tùy chỉnh sử dụng cùng tên. Ví dụ, helper `array_first()` lịch sử chấp nhận một callback để trả về phần tử khớp đầu tiên, trong khi phiên bản polyfilled chỉ trả về phần tử đầu tiên của array.

Để tránh xung đột và đảm bảo hành vi nhất quán trên các phiên bản PHP, bạn nên ưu tiên các phương thức `Illuminate\Support\Arr`:

```php
use Illuminate\Support\Arr;

Arr::first($array, function ($value) {
  return /* condition */;
});
```

<a name="views"></a>
### Views

<a name="pagination-bootstrap-view-names"></a>
#### Pagination Bootstrap View Names

**Likelihood Of Impact: Low**

Các tên view phân trang nội bộ cho các mặc định Bootstrap 3 hiện rõ ràng:

```nothing
// Laravel <= 12.x
pagination::default
pagination::simple-default

// Laravel >= 13.x
pagination::bootstrap-3
pagination::simple-bootstrap-3
```

Nếu ứng dụng của bạn tham chiếu trực tiếp các tên view phân trang cũ, cập nhật các tham chiếu đó.

<a name="miscellaneous"></a>
### Miscellaneous

Chúng tôi cũng khuyến khích bạn xem các thay đổi trong [GitHub repository](https://github.com/laravel/laravel) `laravel/laravel`. Trong khi nhiều thay đổi này không được yêu cầu, bạn có thể muốn giữ các file này đồng bộ với ứng dụng của bạn. Một số thay đổi này sẽ được bao gồm trong hướng dẫn nâng cấp này, nhưng những thay đổi khác, chẳng hạn như thay đổi đối với các file cấu hình hoặc comments, sẽ không. Bạn có thể dễ dàng xem các thay đổi với [công cụ so sánh GitHub](https://github.com/laravel/laravel/compare/12.x...13.x) và chọn các cập nhật nào quan trọng đối với bạn.

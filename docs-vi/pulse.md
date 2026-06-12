# Laravel Pulse

- [Introduction](#introduction)
- [Installation](#installation)
    - [Configuration](#configuration)
- [Dashboard](#dashboard)
    - [Authorization](#dashboard-authorization)
    - [Customization](#dashboard-customization)
    - [Resolving Users](#dashboard-resolving-users)
    - [Cards](#dashboard-cards)
- [Capturing Entries](#capturing-entries)
    - [Recorders](#recorders)
    - [Filtering](#filtering)
- [Performance](#performance)
    - [Using a Different Database](#using-a-different-database)
    - [Redis Ingest](#ingest)
    - [Sampling](#sampling)
    - [Trimming](#trimming)
    - [Handling Pulse Exceptions](#pulse-exceptions)
- [Custom Cards](#custom-cards)
    - [Card Components](#custom-card-components)
    - [Styling](#custom-card-styling)
    - [Data Capture and Aggregation](#custom-card-data)

<a name="introduction"></a>
## Introduction

[Laravel Pulse](https://github.com/laravel/pulse) cung cấp cái nhìn nhanh về hiệu suất và sử dụng ứng dụng của bạn. Với Pulse, bạn có thể theo dõi các nút thắt như các jobs và endpoints chậm, tìm người dùng hoạt động nhất của bạn, và hơn thế nữa.

Để gỡ lỗi chi tiết các sự kiện riêng lẻ, hãy xem [Laravel Telescope](/docs/{{version}}/telescope).

<a name="installation"></a>
## Installation

> [!WARNING]
> Việc triển khai lưu trữ chính thức của Pulse hiện yêu cầu một database MySQL, MariaDB, hoặc PostgreSQL. Nếu bạn đang sử dụng một engine database khác, bạn sẽ cần một database MySQL, MariaDB, hoặc PostgreSQL riêng biệt cho dữ liệu Pulse của bạn.

Bạn có thể cài đặt Pulse bằng cách sử dụng trình quản lý package Composer:

```shell
composer require laravel/pulse
```

Tiếp theo, bạn nên xuất bản file cấu hình Pulse và migration bằng cách sử dụng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
```

Cuối cùng, bạn nên chạy lệnh `migrate` để tạo các bảng cần thiết để lưu trữ dữ liệu Pulse:

```shell
php artisan migrate
```

Sau khi các migrations database của Pulse đã được chạy, bạn có thể truy cập dashboard Pulse thông qua route `/pulse`.

> [!NOTE]
> Nếu bạn không muốn lưu trữ dữ liệu Pulse trong database chính của ứng dụng, bạn có thể [chỉ định một kết nối database chuyên dụng](#using-a-different-database).

<a name="configuration"></a>
### Configuration

Nhiều tùy chọn cấu hình của Pulse có thể được kiểm soát bằng cách sử dụng các biến môi trường. Để xem các tùy chọn có sẵn, đăng ký các recorders mới, hoặc cấu hình các tùy chọn nâng cao, bạn có thể xuất bản file cấu hình `config/pulse.php`:

```shell
php artisan vendor:publish --tag=pulse-config
```

<a name="dashboard"></a>
## Dashboard

<a name="dashboard-authorization"></a>
### Authorization

Dashboard Pulse có thể được truy cập thông qua route `/pulse`. Theo mặc định, bạn chỉ có thể truy cập dashboard này trong môi trường `local`, vì vậy bạn sẽ cần cấu hình authorization cho các môi trường production của bạn bằng cách tùy chỉnh gate authorization `'viewPulse'`. Bạn có thể thực hiện điều này trong file `app/Providers/AppServiceProvider.php` của ứng dụng:

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('viewPulse', function (User $user) {
        return $user->isAdmin();
    });

    // ...
}
```

<a name="dashboard-customization"></a>
### Customization

Các cards và layout dashboard Pulse có thể được cấu hình bằng cách xuất bản view dashboard. View dashboard sẽ được xuất bản đến `resources/views/vendor/pulse/dashboard.blade.php`:

```shell
php artisan vendor:publish --tag=pulse-dashboard
```

Dashboard được hỗ trợ bởi [Livewire](https://livewire.laravel.com/), và cho phép bạn tùy chỉnh các cards và layout mà không cần xây dựng lại bất kỳ tài sản JavaScript nào.

Trong file này, component `<x-pulse>` chịu trách nhiệm hiển thị dashboard và cung cấp một layout grid cho các cards. Nếu bạn muốn dashboard trải rộng toàn bộ chiều rộng màn hình, bạn có thể cung cấp prop `full-width` cho component:

```blade
<x-pulse full-width>
    ...
</x-pulse>
```

Theo mặc định, component `<x-pulse>` sẽ tạo một grid 12 cột, nhưng bạn có thể tùy chỉnh điều này bằng cách sử dụng prop `cols`:

```blade
<x-pulse cols="16">
    ...
</x-pulse>
```

Mỗi card chấp nhận một prop `cols` và `rows` để kiểm soát không gian và vị trí:

```blade
<livewire:pulse.usage cols="4" rows="2" />
```

Hầu hết các cards cũng chấp nhận một prop `expand` để hiển thị toàn bộ card thay vì cuộn:

```blade
<livewire:pulse.slow-queries expand />
```

<a name="dashboard-resolving-users"></a>
### Resolving Users

Đối với các cards hiển thị thông tin về người dùng của bạn, chẳng hạn như card Application Usage, Pulse sẽ chỉ ghi lại ID người dùng. Khi hiển thị dashboard, Pulse sẽ giải quyết các trường `name` và `email` từ model `Authenticatable` mặc định của bạn và hiển thị avatars bằng cách sử dụng dịch vụ web Gravatar.

Bạn có thể tùy chỉnh các trường và avatar bằng cách gọi phương thức `Pulse::user` trong class `App\Providers\AppServiceProvider` của ứng dụng.

Phương thức `user` chấp nhận một closure sẽ nhận model `Authenticatable` để hiển thị và nên trả về một array chứa thông tin `name`, `extra`, và `avatar` cho người dùng:

```php
use Laravel\Pulse\Facades\Pulse;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::user(fn ($user) => [
        'name' => $user->name,
        'extra' => $user->email,
        'avatar' => $user->avatar_url,
    ]);

    // ...
}
```

> [!NOTE]
> Bạn có thể tùy chỉnh hoàn toàn cách người dùng được xác thực được ghi lại và truy xuất bằng cách triển khai contract `Laravel\Pulse\Contracts\ResolvesUsers` và binding nó trong [service container](/docs/{{version}}/container#binding-a-singleton) của Laravel.

<a name="dashboard-cards"></a>
### Cards

<a name="servers-card"></a>
#### Servers

Card `<livewire:pulse.servers />` hiển thị việc sử dụng tài nguyên hệ thống cho tất cả các servers chạy lệnh `pulse:check`. Vui lòng tham khảo tài liệu về [servers recorder](#servers-recorder) để biết thêm thông tin về báo cáo tài nguyên hệ thống.

Nếu bạn thay thế một server trong hạ tầng của mình, bạn có thể muốn ngừng hiển thị server không hoạt động trong dashboard Pulse sau một khoảng thời gian nhất định. Bạn có thể thực hiện điều này bằng cách sử dụng prop `ignore-after`, chấp nhận số giây sau đó các servers không hoạt động nên được xóa khỏi dashboard Pulse. Ngoài ra, bạn có thể cung cấp một chuỗi thời gian tương đối, chẳng hạn như `1 hour` hoặc `3 days and 1 hour`:

```blade
<livewire:pulse.servers ignore-after="3 hours" />
```

<a name="application-usage-card"></a>
#### Application Usage

Card `<livewire:pulse.usage />` hiển thị top 10 người dùng gửi requests đến ứng dụng của bạn, dispatch jobs, và trải qua các requests chậm.

Nếu bạn muốn xem tất cả các metrics sử dụng trên màn hình cùng một lúc, bạn có thể bao gồm card nhiều lần và chỉ định thuộc tính `type`:

```blade
<livewire:pulse.usage type="requests" />
<livewire:pulse.usage type="slow_requests" />
<livewire:pulse.usage type="jobs" />
```

Để tìm hiểu cách tùy chỉnh cách Pulse truy xuất và hiển thị thông tin người dùng, hãy tham khảo tài liệu của chúng tôi về [resolving users](#dashboard-resolving-users).

> [!NOTE]
> Nếu ứng dụng của bạn nhận nhiều requests hoặc dispatch nhiều jobs, bạn có thể muốn bật [sampling](#sampling). Xem tài liệu [user requests recorder](#user-requests-recorder), [user jobs recorder](#user-jobs-recorder), và [slow jobs recorder](#slow-jobs-recorder) để biết thêm thông tin.

<a name="exceptions-card"></a>
#### Exceptions

Card `<livewire:pulse.exceptions />` hiển thị tần suất và tính gần đây của các ngoại lệ xảy ra trong ứng dụng của bạn. Theo mặc định, các ngoại lệ được nhóm dựa trên lớp ngoại lệ và vị trí nơi nó xảy ra. Xem tài liệu [exceptions recorder](#exceptions-recorder) để biết thêm thông tin.

<a name="queues-card"></a>
#### Queues

Card `<livewire:pulse.queues />` hiển thị throughput của các queues trong ứng dụng của bạn, bao gồm số lượng jobs được xếp hàng, đang xử lý, đã xử lý, được giải phóng, và thất bại. Xem tài liệu [queues recorder](#queues-recorder) để biết thêm thông tin.

<a name="slow-requests-card"></a>
#### Slow Requests

Card `<livewire:pulse.slow-requests />` hiển thị các requests đến ứng dụng của bạn vượt quá ngưỡng được cấu hình, mặc định là 1,000ms. Xem tài liệu [slow requests recorder](#slow-requests-recorder) để biết thêm thông tin.

<a name="slow-jobs-card"></a>
#### Slow Jobs

Card `<livewire:pulse.slow-jobs />` hiển thị các queued jobs trong ứng dụng của bạn vượt quá ngưỡng được cấu hình, mặc định là 1,000ms. Xem tài liệu [slow jobs recorder](#slow-jobs-recorder) để biết thêm thông tin.

<a name="slow-queries-card"></a>
#### Slow Queries

Card `<livewire:pulse.slow-queries />` hiển thị các truy vấn database trong ứng dụng của bạn vượt quá ngưỡng được cấu hình, mặc định là 1,000ms.

Theo mặc định, các truy vấn chậm được nhóm dựa trên truy vấn SQL (không có bindings) và vị trí nơi nó xảy ra, nhưng bạn có thể chọn không ghi lại vị trí nếu bạn muốn chỉ nhóm dựa trên truy vấn SQL.

Nếu bạn gặp vấn đề hiệu suất hiển thị do các truy vấn SQL cực lớn nhận được làm nổi bật cú pháp, bạn có thể tắt làm nổi bật bằng cách thêm prop `without-highlighting`:

```blade
<livewire:pulse.slow-queries without-highlighting />
```

Xem tài liệu [slow queries recorder](#slow-queries-recorder) để biết thêm thông tin.

<a name="slow-outgoing-requests-card"></a>
#### Slow Outgoing Requests

Card `<livewire:pulse.slow-outgoing-requests />` hiển thị các requests đi ra được thực hiện bằng cách sử dụng [HTTP client](/docs/{{version}}/http-client) của Laravel vượt quá ngưỡng được cấu hình, mặc định là 1,000ms.

Theo mặc định, các entries sẽ được nhóm theo URL đầy đủ. Tuy nhiên, bạn có thể muốn chuẩn hóa hoặc nhóm các requests đi ra tương tự bằng cách sử dụng regular expressions. Xem tài liệu [slow outgoing requests recorder](#slow-outgoing-requests-recorder) để biết thêm thông tin.

<a name="cache-card"></a>
#### Cache

Card `<livewire:pulse.cache />` hiển thị thống kênh cache hit và miss cho ứng dụng của bạn, cả toàn cầu và cho các keys riêng lẻ.

Theo mặc định, các entries sẽ được nhóm theo key. Tuy nhiên, bạn có thể muốn chuẩn hóa hoặc nhóm các keys tương tự bằng cách sử dụng regular expressions. Xem tài liệu [cache interactions recorder](#cache-interactions-recorder) để biết thêm thông tin.

<a name="capturing-entries"></a>
## Capturing Entries

Hầu hết các recorders Pulse sẽ tự động ghi lại các entries dựa trên các sự kiện framework được dispatch bởi Laravel. Tuy nhiên, [servers recorder](#servers-recorder) và một số cards bên thứ ba phải thăm dò thông tin thường xuyên. Để sử dụng các cards này, bạn phải chạy daemon `pulse:check` trên tất cả các servers ứng dụng riêng lẻ của bạn:

```php
php artisan pulse:check
```

> [!NOTE]
> Để giữ quá trình `pulse:check` chạy vĩnh viễn trong nền, bạn nên sử dụng một process monitor như Supervisor để đảm bảo rằng lệnh không ngừng chạy.

Vì lệnh `pulse:check` là một quá trình chạy dài, nó sẽ không thấy các thay đổi trong codebase của bạn mà không được khởi động lại. Bạn nên khởi động lại lệnh một cách nhẹ nhàng bằng cách gọi lệnh `pulse:restart` trong quá trình triển khai ứng dụng:

```shell
php artisan pulse:restart
```

> [!NOTE]
> Pulse sử dụng [cache](/docs/{{version}}/cache) để lưu trữ các tín hiệu khởi động lại, vì vậy bạn nên xác minh rằng một driver cache được cấu hình đúng cho ứng dụng trước khi sử dụng tính năng này.

<a name="recorders"></a>
### Recorders

Recorders chịu trách nhiệm ghi lại các entries từ ứng dụng của bạn để được ghi lại trong database Pulse. Recorders được đăng ký và cấu hình trong phần `recorders` của [file cấu hình Pulse](#configuration).

<a name="cache-interactions-recorder"></a>
#### Cache Interactions

Recorder `CacheInteractions` ghi lại thông tin về các [cache](/docs/{{version}}/cache) hits và misses xảy ra trong ứng dụng của bạn để hiển thị trên card [Cache](#cache-card).

Bạn có thể tùy chỉnh [sample rate](#sampling) và các pattern key bị bỏ qua.

Bạn cũng có thể cấu hình nhóm key để các keys tương tự được nhóm thành một entry duy nhất. Ví dụ, bạn có thể muốn xóa các ID duy nhất từ các keys lưu trữ cùng loại thông tin. Các nhóm được cấu hình bằng cách sử dụng regular expression để "tìm và thay thế" các phần của key. Một ví dụ được bao gồm trong file cấu hình:

```php
Recorders\CacheInteractions::class => [
    // ...
    'groups' => [
        // '/:\d+/' => ':*',
    ],
],
```

Pattern đầu tiên khớp sẽ được sử dụng. Nếu không có pattern nào khớp, thì key sẽ được ghi lại nguyên trạng.

<a name="exceptions-recorder"></a>
#### Exceptions

Recorder `Exceptions` ghi lại thông tin về các ngoại lệ có thể báo cáo xảy ra trong ứng dụng của bạn để hiển thị trên card [Exceptions](#exceptions-card).

Bạn có thể tùy chỉnh [sample rate](#sampling) và các pattern ngoại lệ bị bỏ qua. Bạn cũng có thể cấu hình xem có ghi lại vị trí ngoại lệ bắt nguồn từ đâu hay không. Vị trí được ghi lại sẽ được hiển thị trên dashboard Pulse có thể giúp theo dõi nguồn gốc ngoại lệ; tuy nhiên, nếu cùng một ngoại lệ xảy ra ở nhiều vị trí thì nó sẽ xuất hiện nhiều lần cho mỗi vị trí duy nhất.

<a name="queues-recorder"></a>
#### Queues

Recorder `Queues` ghi lại thông tin về các queues của ứng dụng để hiển thị trên [Queues](#queues-card).

Bạn có thể tùy chỉnh [sample rate](#sampling) và các pattern jobs bị bỏ qua.

<a name="slow-jobs-recorder"></a>
#### Slow Jobs

Recorder `SlowJobs` ghi lại thông tin về các jobs chậm xảy ra trong ứng dụng của bạn để hiển thị trên card [Slow Jobs](#slow-jobs-recorder).

Bạn có thể tùy chỉnh ngưỡng job chậm, [sample rate](#sampling), và các pattern jobs bị bỏ qua.

Bạn có thể có một số jobs mà bạn mong đợi sẽ mất nhiều thời gian hơn những jobs khác. Trong những trường hợp đó, bạn có thể cấu hình các ngưỡng cho mỗi job:

```php
Recorders\SlowJobs::class => [
    // ...
    'threshold' => [
        '#^App\\Jobs\\GenerateYearlyReports$#' => 5000,
        'default' => env('PULSE_SLOW_JOBS_THRESHOLD', 1000),
    ],
],
```

Nếu không có pattern regular expression nào khớp với classname của job, thì giá trị `'default'` sẽ được sử dụng.

<a name="slow-outgoing-requests-recorder"></a>
#### Slow Outgoing Requests

Recorder `SlowOutgoingRequests` ghi lại thông tin về các requests HTTP đi ra được thực hiện bằng cách sử dụng [HTTP client](/docs/{{version}}/http-client) của Laravel vượt quá ngưỡng được cấu hình để hiển thị trên card [Slow Outgoing Requests](#slow-outgoing-requests-card).

Bạn có thể tùy chỉnh ngưỡng request đi ra chậm, [sample rate](#sampling), và các pattern URL bị bỏ qua.

Bạn có thể có một số requests đi ra mà bạn mong đợi sẽ mất nhiều thời gian hơn những requests khác. Trong những trường hợp đó, bạn có thể cấu hình các ngưỡng cho mỗi request:

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'threshold' => [
        '#backup.zip$#' => 5000,
        'default' => env('PULSE_SLOW_OUTGOING_REQUESTS_THRESHOLD', 1000),
    ],
],
```

Nếu không có pattern regular expression nào khớp với URL của request, thì giá trị `'default'` sẽ được sử dụng.

Bạn cũng có thể cấu hình nhóm URL để các URLs tương tự được nhóm thành một entry duy nhất. Ví dụ, bạn có thể muốn xóa các ID duy nhất từ các đường dẫn URL hoặc chỉ nhóm theo domain. Các nhóm được cấu hình bằng cách sử dụng regular expression để "tìm và thay thế" các phần của URL. Một số ví dụ được bao gồm trong file cấu hình:

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'groups' => [
        // '#^https://api\.github\.com/repos/.*$#' => 'api.github.com/repos/*',
        // '#^https?://([^/]*).*$#' => '\1',
        // '#/\d+#' => '/*',
    ],
],
```

Pattern đầu tiên khớp sẽ được sử dụng. Nếu không có pattern nào khớp, thì URL sẽ được ghi lại nguyên trạng.

<a name="slow-queries-recorder"></a>
#### Slow Queries

Recorder `SlowQueries` ghi lại bất kỳ truy vấn database nào trong ứng dụng của bạn vượt quá ngưỡng được cấu hình để hiển thị trên card [Slow Queries](#slow-queries-card).

Bạn có thể tùy chỉnh ngưỡng truy vấn chậm, [sample rate](#sampling), và các pattern truy vấn bị bỏ qua. Bạn cũng có thể cấu hình xem có ghi lại vị trí truy vấn hay không. Vị trí được ghi lại sẽ được hiển thị trên dashboard Pulse có thể giúp theo dõi nguồn gốc truy vấn; tuy nhiên, nếu cùng một truy vấn được thực hiện ở nhiều vị trí thì nó sẽ xuất hiện nhiều lần cho mỗi vị trí duy nhất.

Bạn có thể có một số truy vấn mà bạn mong đợi sẽ mất nhiều thời gian hơn những truy vấn khác. Trong những trường hợp đó, bạn có thể cấu hình các ngưỡng cho mỗi truy vấn:

```php
Recorders\SlowQueries::class => [
    // ...
    'threshold' => [
        '#^insert into `yearly_reports`#' => 5000,
        'default' => env('PULSE_SLOW_QUERIES_THRESHOLD', 1000),
    ],
],
```

Nếu không có pattern regular expression nào khớp với SQL của truy vấn, thì giá trị `'default'` sẽ được sử dụng.

<a name="slow-requests-recorder"></a>
#### Slow Requests

Recorder `Requests` ghi lại thông tin về các requests được thực hiện đến ứng dụng của bạn để hiển thị trên các cards [Slow Requests](#slow-requests-card) và [Application Usage](#application-usage-card).

Bạn có thể tùy chỉnh ngưỡng route chậm, [sample rate](#sampling), và các paths bị bỏ qua.

Bạn có thể có một số requests mà bạn mong đợi sẽ mất nhiều thời gian hơn những requests khác. Trong những trường hợp đó, bạn có thể cấu hình các ngưỡng cho mỗi request:

```php
Recorders\SlowRequests::class => [
    // ...
    'threshold' => [
        '#^/admin/#' => 5000,
        'default' => env('PULSE_SLOW_REQUESTS_THRESHOLD', 1000),
    ],
],
```

Nếu không có pattern regular expression nào khớp với URL của request, thì giá trị `'default'` sẽ được sử dụng.

<a name="servers-recorder"></a>
#### Servers

Recorder `Servers` ghi lại việc sử dụng CPU, bộ nhớ, và lưu trữ của các servers hỗ trợ ứng dụng của bạn để hiển thị trên card [Servers](#servers-card). Recorder này yêu cầu [lệnh pulse:check](#capturing-entries) được chạy trên mỗi server bạn muốn giám sát.

Mỗi server báo cáo phải có một tên duy nhất. Theo mặc định, Pulse sẽ sử dụng giá trị được trả về bởi hàm `gethostname` của PHP. Nếu bạn muốn tùy chỉnh điều này, bạn có thể đặt biến môi trường `PULSE_SERVER_NAME`:

```env
PULSE_SERVER_NAME=load-balancer
```

File cấu hình Pulse cũng cho phép bạn tùy chỉnh các thư mục được giám sát.

<a name="user-jobs-recorder"></a>
#### User Jobs

Recorder `UserJobs` ghi lại thông tin về người dùng dispatch jobs trong ứng dụng của bạn để hiển thị trên card [Application Usage](#application-usage-card).

Bạn có thể tùy chỉnh [sample rate](#sampling) và các pattern jobs bị bỏ qua.

<a name="user-requests-recorder"></a>
#### User Requests

Recorder `UserRequests` ghi lại thông tin về người dùng gửi requests đến ứng dụng của bạn để hiển thị trên card [Application Usage](#application-usage-card).

Bạn có thể tùy chỉnh [sample rate](#sampling) và các pattern URL bị bỏ qua.

<a name="filtering"></a>
### Filtering

Như chúng ta đã thấy, nhiều [recorders](#recorders) cung cấp khả năng, thông qua cấu hình, "bỏ qua" các entries đến dựa trên giá trị của chúng, chẳng hạn như URL của request. Tuy nhiên, đôi khi có thể hữu ích để lọc bỏ các bản ghi dựa trên các yếu tố khác, chẳng hạn như người dùng được xác thực hiện tại. Để lọc bỏ các bản ghi này, bạn có thể chuyển một closure cho phương thức `filter` của Pulse. Thông thường, phương thức `filter` nên được gọi trong phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Pulse\Entry;
use Laravel\Pulse\Facades\Pulse;
use Laravel\Pulse\Value;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::filter(function (Entry|Value $entry) {
        return Auth::user()->isNotAdmin();
    });

    // ...
}
```

<a name="performance"></a>
## Performance

Pulse đã được thiết kế để thả vào một ứng dụng hiện có mà không yêu cầu bất kỳ hạ tầng bổ sung nào. Tuy nhiên, đối với các ứng dụng có lưu lượng cao, có một số cách để loại bỏ bất kỳ tác động nào mà Pulse có thể có trên hiệu suất ứng dụng của bạn.

<a name="using-a-different-database"></a>
### Using a Different Database

Đối với các ứng dụng có lưu lượng cao, bạn có thể muốn sử dụng một kết nối database chuyên dụng cho Pulse để tránh ảnh hưởng đến database ứng dụng.

Bạn có thể tùy chỉnh [kết nối database](/docs/{{version}}/database#configuration) được sử dụng bởi Pulse bằng cách đặt biến môi trường `PULSE_DB_CONNECTION`:

```env
PULSE_DB_CONNECTION=pulse
```

<a name="ingest"></a>
### Redis Ingest

> [!WARNING]
> Redis Ingest yêu cầu Redis 6.2 hoặc cao hơn và `phpredis` hoặc `predis` làm driver client Redis được cấu hình của ứng dụng.

Theo mặc định, Pulse sẽ lưu trữ các entries trực tiếp vào [kết nối database được cấu hình](#using-a-different-database) sau khi phản hồi HTTP đã được gửi cho client hoặc một job đã được xử lý; tuy nhiên, bạn có thể sử dụng driver ingest Redis của Pulse để gửi các entries đến một Redis stream thay thế. Điều này có thể được bật bằng cách cấu hình biến môi trường `PULSE_INGEST_DRIVER`:

```ini
PULSE_INGEST_DRIVER=redis
```

Pulse sẽ sử dụng [kết nối Redis mặc định](/docs/{{version}}/redis#configuration) của bạn theo mặc định, nhưng bạn có thể tùy chỉnh điều này thông qua biến môi trường `PULSE_REDIS_CONNECTION`:

```ini
PULSE_REDIS_CONNECTION=pulse
```

> [!WARNING]
> Khi sử dụng driver ingest Redis, cài đặt Pulse của bạn nên luôn sử dụng một kết nối Redis khác với Redis queue được hỗ trợ bởi Redis của bạn, nếu có.

Khi sử dụng ingest Redis, bạn sẽ cần chạy lệnh `pulse:work` để giám sát stream và di chuyển các entries từ Redis vào các bảng database Pulse.

```php
php artisan pulse:work
```

> [!NOTE]
> Để giữ quá trình `pulse:work` chạy vĩnh viễn trong nền, bạn nên sử dụng một process monitor như Supervisor để đảm bảo rằng Pulse worker không ngừng chạy.

Vì lệnh `pulse:work` là một quá trình chạy dài, nó sẽ không thấy các thay đổi trong codebase của bạn mà không được khởi động lại. Bạn nên khởi động lại lệnh một cách nhẹ nhàng bằng cách gọi lệnh `pulse:restart` trong quá trình triển khai ứng dụng:

```shell
php artisan pulse:restart
```

> [!NOTE]
> Pulse sử dụng [cache](/docs/{{version}}/cache) để lưu trữ các tín hiệu khởi động lại, vì vậy bạn nên xác minh rằng một driver cache được cấu hình đúng cho ứng dụng trước khi sử dụng tính năng này.

<a name="sampling"></a>
### Sampling

Theo mặc định, Pulse sẽ ghi lại mọi sự kiện liên quan xảy ra trong ứng dụng của bạn. Đối với các ứng dụng có lưu lượng cao, điều này có thể dẫn đến việc cần tổng hợp hàng triệu hàng database trong dashboard, đặc biệt là cho các khoảng thời gian dài hơn.

Thay vào đó, bạn có thể chọn bật "sampling" trên một số recorders dữ liệu Pulse nhất định. Ví dụ, đặt sample rate thành `0.1` trên recorder [User Requests](#user-requests-recorder) sẽ có nghĩa là bạn chỉ ghi lại khoảng 10% các requests đến ứng dụng của bạn. Trong dashboard, các giá trị sẽ được mở rộng và có tiền tố `~` để chỉ ra rằng chúng là một ước tính.

Nhìn chung, càng nhiều entries bạn có cho một metric cụ thể, bạn càng có thể đặt sample rate thấp hơn một cách an toàn mà không hy sinh quá nhiều độ chính xác.

<a name="trimming"></a>
### Trimming

Pulse sẽ tự động cắt các entries được lưu trữ của nó khi chúng nằm ngoài cửa sổ dashboard. Việc cắt xảy ra khi ingesting dữ liệu bằng cách sử dụng một hệ thống xổ số có thể được tùy chỉnh trong [file cấu hình Pulse](#configuration).

<a name="pulse-exceptions"></a>
### Handling Pulse Exceptions

Nếu một ngoại lệ xảy ra trong khi ghi lại dữ liệu Pulse, chẳng hạn như không thể kết nối với database lưu trữ, Pulse sẽ âm thầm thất bại để tránh ảnh hưởng đến ứng dụng của bạn.

Nếu bạn muốn tùy chỉnh cách xử lý các ngoại lệ này, bạn có thể cung cấp một closure cho phương thức `handleExceptionsUsing`:

```php
use Laravel\Pulse\Facades\Pulse;
use Illuminate\Support\Facades\Log;

Pulse::handleExceptionsUsing(function ($e) {
    Log::debug('An exception happened in Pulse', [
        'message' => $e->getMessage(),
        'stack' => $e->getTraceAsString(),
    ]);
});
```

<a name="custom-cards"></a>
## Custom Cards

Pulse cho phép bạn xây dựng các cards tùy chỉnh để hiển thị dữ liệu liên quan đến nhu cầu cụ thể của ứng dụng. Pulse sử dụng [Livewire](https://livewire.laravel.com), vì vậy bạn có thể muốn [xem tài liệu của nó](https://livewire.laravel.com/docs) trước khi xây dựng card tùy chỉnh đầu tiên của bạn.

<a name="custom-card-components"></a>
### Card Components

Tạo một card tùy chỉnh trong Laravel Pulse bắt đầu bằng cách mở rộng component Livewire `Card` cơ sở và định nghĩa một view tương ứng:

```php
namespace App\Livewire\Pulse;

use Laravel\Pulse\Livewire\Card;
use Livewire\Attributes\Lazy;

#[Lazy]
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers');
    }
}
```

Khi sử dụng tính năng [lazy loading](https://livewire.laravel.com/docs/lazy) của Livewire, component `Card` sẽ tự động cung cấp một placeholder tôn trọng các thuộc tính `cols` và `rows` được chuyển cho component của bạn.

Khi viết view tương ứng của card Pulse, bạn có thể tận dụng các component Blade của Pulse để có giao diện nhất quán:

```blade
<x-pulse::card :cols="$cols" :rows="$rows" :class="$class" wire:poll.5s="">
    <x-pulse::card-header name="Top Sellers">
        <x-slot:icon>
            ...
        </x-slot:icon>
    </x-pulse::card-header>

    <x-pulse::scroll :expand="$expand">
        ...
    </x-pulse::scroll>
</x-pulse::card>
```

Các biến `$cols`, `$rows`, `$class`, và `$expand` nên được chuyển cho các component Blade tương ứng của chúng để layout card có thể được tùy chỉnh từ view dashboard. Bạn cũng có thể muốn bao gồm thuộc tính `wire:poll.5s=""` trong view của bạn để có card tự động cập nhật.

Sau khi bạn đã định nghĩa component Livewire và template của mình, card có thể được bao gồm trong [view dashboard](#dashboard-customization):

```blade
<x-pulse>
    ...

    <livewire:pulse.top-sellers cols="4" />
</x-pulse>
```

> [!NOTE]
> Nếu card của bạn được bao gồm trong một package, bạn sẽ cần đăng ký component với Livewire bằng cách sử dụng phương thức `Livewire::component`.

<a name="custom-card-styling"></a>
### Styling

Nếu card của bạn yêu cầu styling bổ sung ngoài các classes và components được bao gồm với Pulse, có một số tùy chọn để bao gồm CSS tùy chỉnh cho các cards của bạn.

<a name="custom-card-styling-vite"></a>
#### Laravel Vite Integration

Nếu card tùy chỉnh của bạn nằm trong codebase ứng dụng và bạn đang sử dụng [tích hợp Vite](/docs/{{version}}/vite) của Laravel, bạn có thể cập nhật file `vite.config.js` của mình để bao gồm một điểm nhập CSS chuyên dụng cho card:

```js
laravel({
    input: [
        'resources/css/pulse/top-sellers.css',
        // ...
    ],
}),
```

Sau đó, bạn có thể sử dụng directive Blade `@vite` trong [view dashboard](#dashboard-customization), chỉ định điểm nhập CSS cho card:

```blade
<x-pulse>
    @vite('resources/css/pulse/top-sellers.css')

    ...
</x-pulse>
```

<a name="custom-card-styling-css"></a>
#### CSS Files

Đối với các trường hợp sử dụng khác, bao gồm các cards Pulse được chứa trong một package, bạn có thể hướng dẫn Pulse tải các stylesheets bổ sung bằng cách định nghĩa một phương thức `css` trên component Livewire của bạn trả về đường dẫn file đến file CSS của bạn:

```php
class TopSellers extends Card
{
    // ...

    protected function css()
    {
        return __DIR__.'/../../dist/top-sellers.css';
    }
}
```

Khi card này được bao gồm trên dashboard, Pulse sẽ tự động bao gồm nội dung của file này trong một thẻ `<style>` để nó không cần được xuất bản đến thư mục `public`.

<a name="custom-card-styling-tailwind"></a>
#### Tailwind CSS

Khi sử dụng Tailwind CSS, bạn nên tạo một điểm nhập CSS chuyên dụng. Ví dụ sau loại trừ các kiểu cơ bản [Preflight](https://tailwindcss.com/docs/preflight) của Tailwind đã được bao gồm bởi Pulse, và phạm vi Tailwind bằng cách sử dụng một CSS selector để tránh xung đột với các lớp Tailwind của Pulse:

```css
@import "tailwindcss/theme.css";

@custom-variant dark (&:where(.dark, .dark *));
@source "./../../views/livewire/pulse/top-sellers.blade.php";

@theme {
  /* ... */
}

#top-sellers {
  @import "tailwindcss/utilities.css" source(none);
}
```

Bạn cũng sẽ cần bao gồm một thuộc tính `id` hoặc `class` trong view của card khớp với CSS selector trong điểm nhập của bạn:

```blade
<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">
    ...
</x-pulse::card>
```

<a name="custom-card-data"></a>
### Data Capture and Aggregation

Các cards tùy chỉnh có thể tìm nạp và hiển thị dữ liệu từ bất cứ đâu; tuy nhiên, bạn có thể muốn tận dụng hệ thống ghi lại và tổng hợp dữ liệu mạnh mẽ và hiệu quả của Pulse.

<a name="custom-card-data-capture"></a>
#### Capturing Entries

Pulse cho phép bạn ghi lại "entries" bằng cách sử dụng phương thức `Pulse::record`:

```php
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

Đối số đầu tiên được cung cấp cho phương thức `record` là `type` cho entry bạn đang ghi lại, trong khi đối số thứ hai là `key` xác định cách dữ liệu được tổng hợp nên được nhóm. Đối với hầu hết các phương thức tổng hợp, bạn cũng sẽ cần chỉ định một `value` để được tổng hợp. Trong ví dụ trên, giá trị được tổng hợp là `$sale->amount`. Sau đó, bạn có thể gọi một hoặc nhiều phương thức tổng hợp (như `sum`) để Pulse có thể ghi lại các giá trị được tổng hợp trước vào "buckets" để truy xuất hiệu quả sau này.

Các phương thức tổng hợp có sẵn là:

* `avg`
* `count`
* `max`
* `min`
* `sum`

> [!NOTE]
> Khi xây dựng một card package ghi lại ID người dùng được xác thực hiện tại, bạn nên sử dụng phương thức `Pulse::resolveAuthenticatedUserId()`, tôn trọng bất kỳ [tùy chỉnh người dùng resolver](#dashboard-resolving-users) được thực hiện cho ứng dụng.

<a name="custom-card-data-retrieval"></a>
#### Retrieving Aggregate Data

Khi mở rộng component Livewire `Card` của Pulse, bạn có thể sử dụng phương thức `aggregate` để truy xuất dữ liệu được tổng hợp cho khoảng thời gian đang được xem trong dashboard:

```php
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers', [
            'topSellers' => $this->aggregate('user_sale', ['sum', 'count'])
        ]);
    }
}
```

Phương thức `aggregate` trả về một collection các đối tượng PHP `stdClass`. Mỗi đối tượng sẽ chứa thuộc tính `key` được ghi lại trước đó, cùng với các khóa cho mỗi tổng hợp được yêu cầu:

```blade
@foreach ($topSellers as $seller)
    {{ $seller->key }}
    {{ $seller->sum }}
    {{ $seller->count }}
@endforeach
```

Pulse sẽ chủ yếu truy xuất dữ liệu từ các buckets được tổng hợp trước; do đó, các tổng hợp được chỉ định phải được ghi lại trước bằng cách sử dụng phương thức `Pulse::record`. Bucket cũ nhất thường sẽ rơi một phần ngoài khoảng thời gian, vì vậy Pulse sẽ tổng hợp các entries cũ nhất để lấp đầy khoảng trống và cung cấp một giá trị chính xác cho toàn bộ khoảng thời gian, mà không cần tổng hợp toàn bộ khoảng thời gian trên mỗi request thăm dò.

Bạn cũng có thể truy xuất một giá trị tổng cho một loại nhất định bằng cách sử dụng phương thức `aggregateTotal`. Ví dụ, phương thức sau sẽ truy xuất tổng của tất cả các doanh số người dùng thay vì nhóm chúng theo người dùng:

```php
$total = $this->aggregateTotal('user_sale', 'sum');
```

<a name="custom-card-displaying-users"></a>
#### Displaying Users

Khi làm việc với các tổng hợp ghi lại ID người dùng làm key, bạn có thể giải quyết các keys thành các bản ghi người dùng bằng cách sử dụng phương thức `Pulse::resolveUsers`:

```php
$aggregates = $this->aggregate('user_sale', ['sum', 'count']);

$users = Pulse::resolveUsers($aggregates->pluck('key'));

return view('livewire.pulse.top-sellers', [
    'sellers' => $aggregates->map(fn ($aggregate) => (object) [
        'user' => $users->find($aggregate->key),
        'sum' => $aggregate->sum,
        'count' => $aggregate->count,
    ])
]);
```

Phương thức `find` trả về một object chứa các khóa `name`, `extra`, và `avatar`, mà bạn có thể tùy chọn chuyển trực tiếp cho component Blade `<x-pulse::user-card>`:

```blade
<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
```

<a name="custom-recorders"></a>
#### Custom Recorders

Các tác giả package có thể muốn cung cấp các lớp recorder để cho phép người dùng cấu hình việc ghi lại dữ liệu.

Recorders được đăng ký trong phần `recorders` của file cấu hình `config/pulse.php` của ứng dụng:

```php
[
    // ...
    'recorders' => [
        Acme\Recorders\Deployments::class => [
            // ...
        ],

        // ...
    ],
]
```

Recorders có thể lắng nghe các sự kiện bằng cách chỉ định một thuộc tính `$listen`. Pulse sẽ tự động đăng ký các listeners và gọi phương thức `record` của recorders:

```php
<?php

namespace Acme\Recorders;

use Acme\Events\Deployment;
use Illuminate\Support\Facades\Config;
use Laravel\Pulse\Facades\Pulse;

class Deployments
{
    /**
     * The events to listen for.
     *
     * @var array<int, class-string>
     */
    public array $listen = [
        Deployment::class,
    ];

    /**
     * Record the deployment.
     */
    public function record(Deployment $event): void
    {
        $config = Config::get('pulse.recorders.'.static::class);

        Pulse::record(
            // ...
        );
    }
}
```

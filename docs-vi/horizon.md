# Laravel Horizon

- [Introduction](#introduction)
- [Installation](#installation)
    - [Configuration](#configuration)
    - [Dashboard Authorization](#dashboard-authorization)
    - [Max Job Attempts](#max-job-attempts)
    - [Job Timeout](#job-timeout)
    - [Job Backoff](#job-backoff)
    - [Silenced Jobs](#silenced-jobs)
- [Balancing Strategies](#balancing-strategies)
    - [Auto Balancing](#auto-balancing)
    - [Simple Balancing](#simple-balancing)
    - [No Balancing](#no-balancing)
- [Upgrading Horizon](#upgrading-horizon)
- [Running Horizon](#running-horizon)
    - [Deploying Horizon](#deploying-horizon)
- [Tags](#tags)
- [Notifications](#notifications)
- [Metrics](#metrics)
- [Deleting Failed Jobs](#deleting-failed-jobs)
- [Clearing Jobs From Queues](#clearing-jobs-from-queues)

<a name="introduction"></a>
## Introduction

> [!NOTE]
> Trước khi đi sâu vào Laravel Horizon, bạn nên làm quen với các dịch vụ [queue](/docs/{{version}}/queues) cơ bản của Laravel. Horizon tăng cường queue của Laravel với các tính năng bổ sung có thể gây nhầm lẫn nếu bạn chưa quen với các tính năng queue cơ bản được cung cấp bởi Laravel.

[Laravel Horizon](https://github.com/laravel/horizon) cung cấp một dashboard đẹp và cấu hình được điều khiển bởi mã cho các [Redis queues](/docs/{{version}}/queues) được hỗ trợ bởi Laravel của bạn. Horizon cho phép bạn dễ dàng giám sát các metrics chính của hệ thống queue của bạn như throughput job, runtime, và các thất bại job.

Khi sử dụng Horizon, tất cả cấu hình queue worker của bạn được lưu trữ trong một file cấu hình đơn giản, dễ dàng. Bằng cách định nghĩa cấu hình worker của ứng dụng trong một file được kiểm soát phiên bản, bạn có thể dễ dàng mở rộng hoặc sửa đổi các queue workers của ứng dụng khi triển khai ứng dụng.

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## Installation

> [!WARNING]
> Laravel Horizon yêu cầu bạn sử dụng [Redis](https://redis.io) để hỗ trợ queue của bạn. Do đó, bạn nên đảm bảo rằng kết nối queue của bạn được đặt thành `redis` trong file cấu hình `config/queue.php` của ứng dụng. Hiện tại Horizon không tương thích với Redis Cluster.

Bạn có thể cài đặt Horizon vào dự án của bạn bằng cách sử dụng trình quản lý package Composer:

```shell
composer require laravel/horizon
```

Sau khi cài đặt Horizon, xuất bản các tài sản của nó bằng cách sử dụng lệnh Artisan `horizon:install`:

```shell
php artisan horizon:install
```

<a name="configuration"></a>
### Configuration

Sau khi xuất bản các tài sản của Horizon, file cấu hình chính của nó sẽ nằm tại `config/horizon.php`. File cấu hình này cho phép bạn cấu hình các tùy chọn queue worker cho ứng dụng của bạn. Mỗi tùy chọn cấu hình bao gồm một mô tả về mục đích của nó, vì vậy hãy đảm bảo khám phá kỹ lưỡng file này.

> [!WARNING]
> Horizon sử dụng một kết nối Redis có tên `horizon` nội bộ. Tên kết nối Redis này được dành riêng và không nên được gán cho một kết nối Redis khác trong file cấu hình `database.php` hoặc làm giá trị của tùy chọn `use` trong file cấu hình `horizon.php`.

<a name="environments"></a>
#### Environments

Sau khi cài đặt, tùy chọn cấu hình Horizon chính mà bạn nên làm quen là tùy chọn cấu hình `environments`. Tùy chọn cấu hình này là một array các môi trường mà ứng dụng của bạn chạy trên và định nghĩa các tùy chọn quy trình worker cho mỗi môi trường. Theo mặc định, mục này chứa một môi trường `production` và `local`. Tuy nhiên, bạn có thể tự do thêm nhiều môi trường hơn khi cần:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],

    'local' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

Bạn cũng có thể định nghĩa một môi trường wildcard (`*`) sẽ được sử dụng khi không tìm thấy môi trường phù hợp nào khác:

```php
'environments' => [
    // ...

    '*' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

Khi bạn bắt đầu Horizon, nó sẽ sử dụng các tùy chọn cấu hình quy trình worker cho môi trường mà ứng dụng của bạn đang chạy. Thông thường, môi trường được xác định bởi giá trị của [biến môi trường](/docs/{{version}}/configuration#determining-the-current-environment) `APP_ENV`. Ví dụ, môi trường Horizon `local` mặc định được cấu hình để bắt đầu ba quy trình worker và tự động cân bằng số lượng quy trình worker được gán cho mỗi queue. Môi trường `production` mặc định được cấu hình để bắt đầu tối đa 10 quy trình worker và tự động cân bằng số lượng quy trình worker được gán cho mỗi queue.

> [!WARNING]
> Bạn nên đảm bảo rằng phần `environments` của file cấu hình `horizon` của bạn chứa một mục cho mỗi [môi trường](/docs/{{version}}/configuration#environment-configuration) mà bạn dự định chạy Horizon.

<a name="supervisors"></a>
#### Supervisors

Như bạn có thể thấy trong file cấu hình mặc định của Horizon, mỗi môi trường có thể chứa một hoặc nhiều "supervisors". Theo mặc định, file cấu hình định nghĩa supervisor này là `supervisor-1`; tuy nhiên, bạn có thể tự do đặt tên cho các supervisors của bạn bất cứ gì bạn muốn. Mỗi supervisor về cơ bản chịu trách nhiệm "giám sát" một nhóm các quy trình worker và lo việc cân bằng các quy trình worker trên các queues.

Bạn có thể thêm các supervisors bổ sung vào một môi trường nhất định nếu bạn muốn định nghĩa một nhóm mới các quy trình worker nên chạy trong môi trường đó. Bạn có thể chọn làm điều này nếu bạn muốn định nghĩa một chiến lược cân bằng khác hoặc số lượng quy trình worker khác cho một queue nhất định được sử dụng bởi ứng dụng của bạn.

<a name="maintenance-mode"></a>
#### Maintenance Mode

Trong khi ứng dụng của bạn ở trong [chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), các queued jobs sẽ không được xử lý bởi Horizon trừ khi tùy chọn `force` của supervisor được định nghĩa là `true` trong file cấu hình Horizon:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'force' => true,
        ],
    ],
],
```

<a name="default-values"></a>
#### Default Values

Trong file cấu hình mặc định của Horizon, bạn sẽ nhận thấy một tùy chọn cấu hình `defaults`. Tùy chọn cấu hình này chỉ định các giá trị mặc định cho [supervisors](#supervisors) của ứng dụng. Các giá trị cấu hình mặc định của supervisor sẽ được hợp nhất vào cấu hình supervisor cho mỗi môi trường, cho phép bạn tránh sự lặp lại không cần thiết khi định nghĩa các supervisors của bạn.

<a name="dashboard-authorization"></a>
### Dashboard Authorization

Dashboard Horizon có thể được truy cập thông qua route `/horizon`. Theo mặc định, bạn chỉ có thể truy cập dashboard này trong môi trường `local`. Tuy nhiên, trong file `app/Providers/HorizonServiceProvider.php` của bạn, có một định nghĩa [authorization gate](/docs/{{version}}/authorization#gates). Authorization gate này kiểm soát quyền truy cập vào Horizon trong các môi trường **không phải local**. Bạn có thể tự do sửa đổi gate này khi cần để hạn chế quyền truy cập vào cài đặt Horizon của bạn:

```php
/**
 * Register the Horizon gate.
 *
 * This gate determines who can access Horizon in non-local environments.
 */
protected function gate(): void
{
    Gate::define('viewHorizon', function (User $user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

<a name="alternative-authentication-strategies"></a>
#### Alternative Authentication Strategies

Hãy nhớ rằng Laravel tự động inject người dùng được xác thực vào closure gate. Nếu ứng dụng của bạn đang cung cấp bảo mật Horizon thông qua một phương thức khác, chẳng hạn như hạn chế IP, thì người dùng Horizon của bạn có thể không cần "đăng nhập". Do đó, bạn sẽ cần thay đổi chữ ký closure `function (User $user)` ở trên thành `function (User $user = null)` để buộc Laravel không yêu cầu xác thực.

<a name="max-job-attempts"></a>
### Max Job Attempts

> [!NOTE]
> Trước khi tinh chỉnh các tùy chọn này, hãy đảm bảo bạn đã quen với các dịch vụ [queue](/docs/{{version}}/queues#max-job-attempts-and-timeout) mặc định của Laravel và khái niệm 'attempts'.

Bạn có thể định nghĩa số lần tối đa mà một job có thể tiêu thụ trong cấu hình supervisor:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'tries' => 10,
        ],
    ],
],
```

> [!NOTE]
> Tùy chọn này tương tự như tùy chọn `--tries` khi sử dụng lệnh Artisan để xử lý các queues.

Điều chỉnh tùy chọn `tries` là cần thiết khi sử dụng các middlewares như `WithoutOverlapping` hoặc `RateLimited` vì chúng tiêu thụ các attempts. Để xử lý điều này, hãy điều chỉnh giá trị cấu hình `tries` ở cấp supervisor hoặc bằng cách định nghĩa thuộc tính `$tries` trên class job.

Nếu bạn không đặt tùy chọn `tries`, Horizon mặc định là một lần thử, trừ khi class job định nghĩa `$tries`, có precedence trên cấu hình Horizon.

Đặt `tries` hoặc `$tries` thành 0 cho phép các attempts không giới hạn, lý tưởng khi số lượng attempts không chắc chắn. Để ngăn chặn các thất bại vô tận, bạn có thể giới hạn số lượng ngoại lệ được phép bằng cách đặt thuộc tính `$maxExceptions` trên class job.

<a name="job-timeout"></a>
### Job Timeout

Tương tự, bạn có thể đặt giá trị `timeout` ở cấp supervisor, chỉ định bao nhiêu giây một quy trình worker có thể chạy một job trước khi nó bị chấm dứt mạnh mẽ. Sau khi bị chấm dứt, job sẽ được thử lại hoặc được đánh dấu là thất bại, tùy thuộc vào cấu hình queue của bạn:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'timeout' => 60,
        ],
    ],
],
```

> [!WARNING]
> Khi sử dụng chiến lược cân bằng `auto`, Horizon sẽ coi các workers đang tiến hành là "treo" và force-kill chúng sau timeout Horizon trong quá trình thu nhỏ. Luôn đảm bảo timeout Horizon lớn hơn bất kỳ timeout cấp job nào, nếu không các jobs có thể bị chấm dứt giữa quá trình thực thi. Ngoài ra, giá trị `timeout` nên luôn ngắn hơn một vài giây so với giá trị `retry_after` được định nghĩa trong file cấu hình `config/queue.php` của bạn. Nếu không, các jobs của bạn có thể được xử lý hai lần.

<a name="job-backoff"></a>
### Job Backoff

Bạn có thể định nghĩa giá trị `backoff` ở cấp supervisor để chỉ định bao lâu Horizon nên đợi trước khi thử lại một job gặp một ngoại lệ không được xử lý:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => 10,
        ],
    ],
],
```

Bạn cũng có thể cấu hình các backoffs "exponential" bằng cách sử dụng một array cho giá trị `backoff`. Trong ví dụ này, độ trễ thử lại sẽ là 1 giây cho lần thử lại đầu tiên, 5 giây cho lần thử lại thứ hai, 10 giây cho lần thử lại thứ ba, và 10 giây cho mỗi lần thử lại tiếp theo nếu còn nhiều attempts còn lại:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => [1, 5, 10],
        ],
    ],
],
```

<a name="silenced-jobs"></a>
### Silenced Jobs

Đôi khi, bạn có thể không quan tâm đến việc xem các jobs nhất định được dispatch bởi ứng dụng hoặc các packages bên thứ ba. Thay vì các jobs này chiếm không gian trong danh sách "Completed Jobs" của bạn, bạn có thể im lặng chúng. Để bắt đầu, thêm tên class job vào tùy chọn cấu hình `silenced` trong file cấu hình `horizon` của ứng dụng:

```php
'silenced' => [
    App\Jobs\ProcessPodcast::class,
],
```

Ngoài việc im lặng các class job riêng lẻ, Horizon cũng hỗ trợ im lặng các jobs dựa trên [tags](#tags). Điều này có thể hữu ích nếu bạn muốn ẩn nhiều jobs chia sẻ một tag chung:

```php
'silenced_tags' => [
    'notifications'
],
```

Ngoài ra, job bạn muốn im lặng có thể triển khai interface `Laravel\Horizon\Contracts\Silenced`. Nếu một job triển khai interface này, nó sẽ tự động bị im lặng, ngay cả khi nó không có trong array cấu hình `silenced`:

```php
use Laravel\Horizon\Contracts\Silenced;

class ProcessPodcast implements ShouldQueue, Silenced
{
    use Queueable;

    // ...
}
```

<a name="balancing-strategies"></a>
## Balancing Strategies

Mỗi supervisor có thể xử lý một hoặc nhiều queues nhưng không giống như hệ thống queue mặc định của Laravel, Horizon cho phép bạn chọn từ ba chiến lược cân bằng worker: `auto`, `simple`, và `false`.

<a name="auto-balancing"></a>
### Auto Balancing

Chiến lược `auto`, là chiến lược mặc định, điều chỉnh số lượng quy trình worker cho mỗi queue dựa trên workload hiện tại của queue. Ví dụ, nếu queue `notifications` của bạn có 1,000 jobs đang chờ trong khi queue `default` của bạn trống, Horizon sẽ phân bổ nhiều workers hơn cho queue `notifications` của bạn cho đến khi queue trống.

Khi sử dụng chiến lược `auto`, bạn cũng có thể cấu hình các tùy chọn cấu hình `minProcesses` và `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` định nghĩa số lượng tối thiểu các quy trình worker cho mỗi queue. Giá trị này phải lớn hơn hoặc bằng 1.
- `maxProcesses` định nghĩa số lượng tối đa tổng các quy trình worker mà Horizon có thể mở rộng lên trên tất cả các queues. Giá trị này thường nên lớn hơn số lượng queues nhân với giá trị `minProcesses`. Để ngăn supervisor tạo ra bất kỳ quy trình nào, bạn có thể đặt giá trị này thành 0.

</div>

Ví dụ, bạn có thể cấu hình Horizon để duy trì ít nhất một quy trình cho mỗi queue và mở rộng lên tổng 10 quy trình worker:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default', 'notifications'],
            'balance' => 'auto',
            'autoScalingStrategy' => 'time',
            'minProcesses' => 1,
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],
],
```

Tùy chọn cấu hình `autoScalingStrategy` xác định cách Horizon sẽ phân bổ nhiều quy trình worker hơn cho các queues. Bạn có thể chọn giữa hai chiến lược:

<div class="content-list" markdown="1">

- Chiến lược `time` sẽ phân bổ workers dựa trên tổng thời gian ước tính sẽ mất để xóa queue.
- Chiến lược `size` sẽ phân bổ workers dựa trên tổng số jobs trên queue.

</div>

Các giá trị cấu hình `balanceMaxShift` và `balanceCooldown` xác định tốc độ Horizon sẽ mở rộng để đáp ứng nhu cầu worker. Trong ví dụ trên, tối đa một quy trình mới sẽ được tạo hoặc hủy mỗi ba giây. Bạn có thể tự do tinh chỉnh các giá trị này khi cần dựa trên nhu cầu của ứng dụng.

<a name="auto-queue-priorities"></a>
#### Queue Priorities and Auto Balancing

Khi sử dụng chiến lược cân bằng `auto`, Horizon không thực thi ưu tiên nghiêm ngặt giữa các queues. Thứ tự các queues trong cấu hình supervisor không ảnh hưởng đến cách các quy trình worker được phân bổ. Thay vào đó, Horizon dựa vào `autoScalingStrategy` được chọn để phân bổ động các quy trình worker dựa trên tải queue.

Ví dụ, trong cấu hình sau, queue cao không được ưu tiên hơn queue mặc định, mặc dù xuất hiện đầu tiên trong danh sách:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['high', 'default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

Nếu bạn cần thực thi ưu tiên tương đối giữa các queues, bạn có thể định nghĩa nhiều supervisors và phân bổ rõ ràng các tài nguyên xử lý:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
        'supervisor-2' => [
            // ...
            'queue' => ['images'],
            'minProcesses' => 1,
            'maxProcesses' => 1,
        ],
    ],
],
```

Trong ví dụ này, queue `default` có thể mở rộng lên 10 quy trình, trong khi queue `images` bị giới hạn một quy trình. Cấu hình này đảm bảo rằng các queues của bạn có thể mở rộng độc lập.

> [!NOTE]
> Khi dispatch các jobs tốn nhiều tài nguyên, đôi khi tốt nhất là gán chúng cho một queue chuyên dụng với giá trị `maxProcesses` giới hạn. Nếu không, các jobs này có thể tiêu thụ quá nhiều tài nguyên CPU và quá tải hệ thống của bạn.

<a name="simple-balancing"></a>
### Simple Balancing

Chiến lược `simple` phân phối các quy trình worker đều trên các queues được chỉ định. Với chiến lược này, Horizon không tự động mở rộng số lượng quy trình worker. Thay vào đó, nó sử dụng một số lượng quy trình cố định:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => 'simple',
            'processes' => 10,
        ],
    ],
],
```

Trong ví dụ trên, Horizon sẽ phân bổ 5 quy trình cho mỗi queue, chia tổng 10 đều.

Nếu bạn muốn kiểm soát số lượng quy trình worker được phân bổ cho mỗi queue riêng lẻ, bạn có thể định nghĩa nhiều supervisors:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'balance' => 'simple',
            'processes' => 10,
        ],
        'supervisor-notifications' => [
            // ...
            'queue' => ['notifications'],
            'balance' => 'simple',
            'processes' => 2,
        ],
    ],
],
```

Với cấu hình này, Horizon sẽ phân bổ 10 quy trình cho queue `default` và 2 quy trình cho queue `notifications`.

<a name="no-balancing"></a>
### No Balancing

Khi tùy chọn `balance` được đặt thành `false`, Horizon xử lý các queues nghiêm ngặt theo thứ tự chúng được liệt kê, tương tự như hệ thống queue mặc định của Laravel. Tuy nhiên, nó vẫn sẽ mở rộng số lượng quy trình worker nếu các jobs bắt đầu tích lũy:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => false,
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

Trong ví dụ trên, các jobs trong queue `default` luôn được ưu tiên hơn các jobs trong queue `notifications`. Ví dụ, nếu có 1,000 jobs trong `default` và chỉ 10 trong `notifications`, Horizon sẽ xử lý hoàn toàn tất cả các jobs `default` trước khi xử lý bất kỳ jobs nào từ `notifications`.

Bạn có thể kiểm soát khả năng mở rộng các quy trình worker của Horizon bằng cách sử dụng các tùy chọn `minProcesses` và `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` định nghĩa số lượng tối thiểu các quy trình worker tổng cộng. Giá trị này phải lớn hơn hoặc bằng 1.
- `maxProcesses` định nghĩa số lượng tối đa tổng các quy trình worker mà Horizon có thể mở rộng lên.

</div>

<a name="upgrading-horizon"></a>
## Upgrading Horizon

Khi nâng cấp lên một phiên bản chính mới của Horizon, điều quan trọng là bạn xem xét kỹ [hướng dẫn nâng cấp](https://github.com/laravel/horizon/blob/master/UPGRADE.md).

<a name="running-horizon"></a>
## Running Horizon

Sau khi bạn đã cấu hình các supervisors và workers trong file cấu hình `config/horizon.php` của ứng dụng, bạn có thể bắt đầu Horizon bằng cách sử dụng lệnh Artisan `horizon`. Lệnh đơn lẻ này sẽ bắt đầu tất cả các quy trình worker được cấu hình cho môi trường hiện tại:

```shell
php artisan horizon
```

Bạn có thể tạm dừng quy trình Horizon và hướng dẫn nó tiếp tục xử lý các jobs bằng cách sử dụng các lệnh Artisan `horizon:pause` và `horizon:continue`:

```shell
php artisan horizon:pause

php artisan horizon:continue
```

Bạn cũng có thể tạm dừng và tiếp tục các [supervisors](#supervisors) Horizon cụ thể bằng cách sử dụng các lệnh Artisan `horizon:pause-supervisor` và `horizon:continue-supervisor`:

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

Bạn có thể kiểm tra trạng thái hiện tại của quy trình Horizon bằng cách sử dụng lệnh Artisan `horizon:status`:

```shell
php artisan horizon:status
```

Bạn có thể kiểm tra trạng thái hiện tại của một [supervisor](#supervisors) Horizon cụ thể bằng cách sử dụng lệnh Artisan `horizon:supervisor-status`:

```shell
php artisan horizon:supervisor-status supervisor-1
```

Bạn có thể chấm dứt nhẹ nhàng quy trình Horizon bằng cách sử dụng lệnh Artisan `horizon:terminate`. Bất kỳ jobs nào đang được xử lý sẽ được hoàn thành và sau đó Horizon sẽ ngừng thực thi:

```shell
php artisan horizon:terminate
```

<a name="automatically-restarting-horizon"></a>
#### Automatically Restarting Horizon

Trong quá trình phát triển cục bộ, bạn có thể chạy lệnh `horizon:listen`. Khi sử dụng lệnh `horizon:listen`, bạn không phải khởi động lại Horizon thủ công khi bạn muốn tải lại mã đã cập nhật của mình. Trước khi sử dụng tính năng này, bạn nên đảm bảo rằng [Node](https://nodejs.org) được cài đặt trong môi trường phát triển cục bộ của bạn. Ngoài ra, bạn nên cài đặt thư viện theo dõi file [Chokidar](https://github.com/paulmillr/chokidar) trong dự án của bạn:

```shell
npm install --save-dev chokidar
```

Sau khi Chokidar được cài đặt, bạn có thể bắt đầu Horizon bằng cách sử dụng lệnh `horizon:listen`:

```shell
php artisan horizon:listen
```

Khi chạy trong Docker hoặc Vagrant, bạn nên sử dụng tùy chọn `--poll`:

```shell
php artisan horizon:listen --poll
```

Bạn có thể cấu hình các thư mục và files nên được theo dõi bằng cách sử dụng tùy chọn cấu hình `watch` trong file cấu hình `config/horizon.php` của ứng dụng:

```php
'watch' => [
    'app',
    'bootstrap',
    'config',
    'database',
    'public/**/*.php',
    'resources/**/*.php',
    'routes',
    'composer.lock',
    '.env',
],
```

<a name="deploying-horizon"></a>
### Deploying Horizon

Khi bạn đã sẵn sàng để triển khai Horizon đến server thực tế của ứng dụng, bạn nên cấu hình một process monitor để giám sát lệnh `php artisan horizon` và khởi động lại nó nếu nó thoát bất ngờ. Đừng lo lắng, chúng tôi sẽ thảo luận cách cài đặt một process monitor dưới đây.

Trong quá trình triển khai ứng dụng, bạn nên hướng dẫn quy trình Horizon chấm dứt để nó sẽ được khởi động lại bởi process monitor của bạn và nhận các thay đổi mã của bạn:

```shell
php artisan horizon:terminate
```

<a name="installing-supervisor"></a>
#### Installing Supervisor

Supervisor là một process monitor cho hệ điều hành Linux và sẽ tự động khởi động lại quy trình `horizon` của bạn nếu nó ngừng thực thi. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng lệnh sau. Nếu bạn không sử dụng Ubuntu, bạn có thể cài đặt Supervisor bằng cách sử dụng trình quản lý package của hệ điều hành:

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> Nếu cấu hình Supervisor bản thân nghe có vẻ quá sức, hãy xem xét sử dụng [Laravel Cloud](https://cloud.laravel.com), có thể quản lý các quy trình nền cho các ứng dụng Laravel của bạn.

<a name="supervisor-configuration"></a>
#### Supervisor Configuration

Các file cấu hình Supervisor thường được lưu trữ trong thư mục `/etc/supervisor/conf.d` của server. Trong thư mục này, bạn có thể tạo bất kỳ số lượng file cấu hình nào hướng dẫn supervisor cách các quy trình của bạn nên được giám sát. Ví dụ, hãy tạo một file `horizon.conf` bắt đầu và giám sát một quy trình `horizon`:

```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

Khi định nghĩa cấu hình Supervisor của bạn, bạn nên đảm bảo rằng giá trị của `stopwaitsecs` lớn hơn số giây tiêu thụ bởi job chạy dài nhất của bạn. Nếu không, Supervisor có thể giết job trước khi nó hoàn thành xử lý.

> [!WARNING]
> Trong khi các ví dụ trên hợp lệ cho các servers dựa trên Ubuntu, vị trí và phần mở rộng file được mong đợi của các file cấu hình Supervisor có thể thay đổi giữa các hệ điều hành server khác. Vui lòng tham khảo tài liệu của server để biết thêm thông tin.

<a name="starting-supervisor"></a>
#### Starting Supervisor

Sau khi file cấu hình đã được tạo, bạn có thể cập nhật cấu hình Supervisor và bắt đầu các quy trình được giám sát bằng cách sử dụng các lệnh sau:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> [!NOTE]
> Để biết thêm thông tin về việc chạy Supervisor, hãy tham khảo [tài liệu Supervisor](http://supervisord.org/index.html).

<a name="tags"></a>
## Tags

Horizon cho phép bạn gán "tags" cho các jobs, bao gồm mailables, broadcast events, notifications, và queued event listeners. Thực tế, Horizon sẽ tự động và thông minh tag hầu hết các jobs tùy thuộc vào các Eloquent models được gắn với job. Ví dụ, hãy xem job sau:

```php
<?php

namespace App\Jobs;

use App\Models\Video;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class RenderVideo implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Video $video,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // ...
    }
}
```

Nếu job này được queued với một instance `App\Models\Video` có thuộc tính `id` là `1`, nó sẽ tự động nhận tag `App\Models\Video:1`. Điều này là do Horizon sẽ tìm kiếm các thuộc tính của job cho bất kỳ Eloquent models nào. Nếu Eloquent models được tìm thấy, Horizon sẽ thông minh tag job bằng cách sử dụng tên class và khóa chính của model:

```php
use App\Jobs\RenderVideo;
use App\Models\Video;

$video = Video::find(1);

RenderVideo::dispatch($video);
```

<a name="manually-tagging-jobs"></a>
#### Manually Tagging Jobs

Nếu bạn muốn định nghĩa thủ công các tags cho một trong các objects có thể queue của bạn, bạn có thể định nghĩa một phương thức `tags` trên class:

```php
class RenderVideo implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the job.
     *
     * @return array<int, string>
     */
    public function tags(): array
    {
        return ['render', 'video:'.$this->video->id];
    }
}
```

<a name="manually-tagging-event-listeners"></a>
#### Manually Tagging Event Listeners

Khi truy xuất các tags cho một queued event listener, Horizon sẽ tự động chuyển instance event cho phương thức `tags`, cho phép bạn thêm dữ liệu event vào các tags:

```php
class SendRenderNotifications implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the listener.
     *
     * @return array<int, string>
     */
    public function tags(VideoRendered $event): array
    {
        return ['video:'.$event->video->id];
    }
}
```

<a name="notifications"></a>
## Notifications

> [!WARNING]
> Khi cấu hình Horizon để gửi Slack hoặc SMS notifications, bạn nên xem xét [điều kiện tiên quyết cho kênh notification liên quan](/docs/{{version}}/notifications).

Nếu bạn muốn được thông báo khi một trong các queues của bạn có thời gian chờ dài, bạn có thể sử dụng các phương thức `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, và `Horizon::routeSmsNotificationsTo`. Bạn có thể gọi các phương thức này từ phương thức `boot` của `App\Providers\HorizonServiceProvider` của ứng dụng:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    parent::boot();

    Horizon::routeSmsNotificationsTo('15556667777');
    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
}
```

<a name="configuring-notification-wait-time-thresholds"></a>
#### Configuring Notification Wait Time Thresholds

Bạn có thể cấu hình bao nhiêu giây được coi là một "long wait" trong file cấu hình `config/horizon.php` của ứng dụng. Tùy chọn cấu hình `waits` trong file này cho phép bạn kiểm soát ngưỡng chờ dài cho mỗi kết nối / queue combination. Bất kỳ kết nối / queue combination nào không được định nghĩa sẽ mặc định là ngưỡng chờ dài 60 giây:

```php
'waits' => [
    'redis:critical' => 30,
    'redis:default' => 60,
    'redis:batch' => 120,
],
```

Đặt ngưỡng của một queue thành `0` sẽ vô hiệu hóa các notifications chờ dài cho queue đó.

<a name="metrics"></a>
## Metrics

Horizon bao gồm một dashboard metrics cung cấp thông tin về thời gian chờ và throughput job và queue của bạn. Để điền vào dashboard này, bạn nên cấu hình lệnh Artisan `snapshot` của Horizon để chạy mỗi năm phút trong file `routes/console.php` của ứng dụng:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

Nếu bạn muốn xóa tất cả dữ liệu metrics, bạn có thể gọi lệnh Artisan `horizon:clear-metrics`:

```shell
php artisan horizon:clear-metrics
```

<a name="deleting-failed-jobs"></a>
## Deleting Failed Jobs

Nếu bạn muốn xóa một failed job, bạn có thể sử dụng lệnh `horizon:forget`. Lệnh `horizon:forget` chấp nhận ID hoặc UUID của failed job làm đối số duy nhất:

```shell
php artisan horizon:forget 5
```

Nếu bạn muốn xóa tất cả các failed jobs, bạn có thể cung cấp tùy chọn `--all` cho lệnh `horizon:forget`:

```shell
php artisan horizon:forget --all
```

<a name="clearing-jobs-from-queues"></a>
## Clearing Jobs From Queues

Nếu bạn muốn xóa tất cả các jobs từ queue mặc định của ứng dụng, bạn có thể làm điều đó bằng cách sử dụng lệnh Artisan `horizon:clear`:

```shell
php artisan horizon:clear
```

Bạn có thể cung cấp tùy chọn `queue` để xóa các jobs từ một queue cụ thể:

```shell
php artisan horizon:clear --queue=emails
```

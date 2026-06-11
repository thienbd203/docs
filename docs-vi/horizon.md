# Laravel Horizon

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Dashboard Authorization](#dashboard-authorization)
    - [Số lần Thử Job Tối đa](#max-job-attempts)
    - [Job Timeout](#job-timeout)
    - [Job Backoff](#job-backoff)
    - [Silenced Jobs](#silenced-jobs)
- [Chiến lược Cân bằng](#balancing-strategies)
    - [Auto Balancing](#auto-balancing)
    - [Simple Balancing](#simple-balancing)
    - [No Balancing](#no-balancing)
- [Nâng cấp Horizon](#upgrading-horizon)
- [Chạy Horizon](#running-horizon)
    - [Triển khai Horizon](#deploying-horizon)
- [Tags](#tags)
- [Notifications](#notifications)
- [Metrics](#metrics)
- [Xóa Failed Jobs](#deleting-failed-jobs)
- [Xóa Jobs khỏi Queues](#clearing-jobs-from-queues)

<a name="introduction"></a>
## Giới thiệu

> [!NOTE]
> Trước khi đi sâu vào Laravel Horizon, bạn nên làm quen với [queue services cơ bản](/docs/{{version}}/queues) của Laravel. Horizon bổ sung các tính năng bổ sung cho queue của Laravel có thể gây nhầm lẫn nếu bạn chưa quen với các tính năng queue cơ bản được cung cấp bởi Laravel.

[Laravel Horizon](https://github.com/laravel/horizon) cung cấp một dashboard đẹp và cấu hình dựa trên code cho [Redis queues](/docs/{{version}}/queues) của Laravel. Horizon cho phép bạn dễ dàng giám sát các metrics quan trọng của hệ thống queue của bạn như job throughput, runtime, và job failures.

Khi sử dụng Horizon, tất cả cấu hình queue worker của bạn được lưu trữ trong một file cấu hình đơn giản. Bằng cách định nghĩa cấu hình worker của ứng dụng của bạn trong một file được kiểm soát phiên bản, bạn có thể dễ dàng scale hoặc sửa đổi queue workers của ứng dụng của bạn khi triển khai ứng dụng.

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## Cài đặt

> [!WARNING]
> Laravel Horizon yêu cầu bạn sử dụng [Redis](https://redis.io) để cung cấp queue của bạn. Do đó, bạn nên đảm bảo rằng kết nối queue của bạn được đặt thành `redis` trong file cấu hình `config/queue.php` của ứng dụng của bạn. Hiện tại Horizon không tương thích với Redis Cluster.

Bạn có thể cài đặt Horizon vào dự án của mình sử dụng Composer package manager:

```shell
composer require laravel/horizon
```

Sau khi cài đặt Horizon, publish các assets của nó sử dụng command `horizon:install` của Artisan:

```shell
php artisan horizon:install
```

<a name="configuration"></a>
### Cấu hình

Sau khi publish các assets của Horizon, file cấu hình chính của nó sẽ nằm tại `config/horizon.php`. File cấu hình này cho phép bạn cấu hình các tùy chọn queue worker cho ứng dụng của bạn. Mỗi tùy chọn cấu hình bao gồm mô tả về mục đích của nó, vì vậy hãy đảm bảo khám phá kỹ lưỡng file này.

> [!WARNING]
> Horizon sử dụng một kết nối Redis có tên `horizon` nội bộ. Tên kết nối Redis này được dành riêng và không nên được gán cho một kết nối Redis khác trong file cấu hình `database.php` hoặc làm giá trị của tùy chọn `use` trong file cấu hình `horizon.php`.

<a name="environments"></a>
#### Environments

Sau khi cài đặt, tùy chọn cấu hình Horizon chính mà bạn nên làm quen là tùy chọn cấu hình `environments`. Tùy chọn cấu hình này là một mảng các môi trường mà ứng dụng của bạn chạy trên đó và định nghĩa các tùy chọn process worker cho mỗi môi trường. Theo mặc định, mục này chứa môi trường `production` và `local`. Tuy nhiên, bạn có thể tự do thêm nhiều môi trường hơn khi cần:

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

Khi bạn khởi động Horizon, nó sẽ sử dụng các tùy chọn cấu hình process worker cho môi trường mà ứng dụng của bạn đang chạy. Thông thường, môi trường được xác định bởi giá trị của [environment variable](/docs/{{version}}/configuration#determining-the-current-environment) `APP_ENV`. Ví dụ, môi trường Horizon `local` mặc định được cấu hình để khởi động ba process worker và tự động cân bằng số lượng process worker được gán cho mỗi queue. Môi trường `production` mặc định được cấu hình để khởi động tối đa 10 process worker và tự động cân bằng số lượng process worker được gán cho mỗi queue.

> [!WARNING]
> Bạn nên đảm bảo rằng phần `environments` của file cấu hình `horizon` của bạn chứa một mục cho mỗi [môi trường](/docs/{{version}}/configuration#environment-configuration) mà bạn dự định chạy Horizon.

<a name="supervisors"></a>
#### Supervisors

Như bạn có thể thấy trong file cấu hình mặc định của Horizon, mỗi môi trường có thể chứa một hoặc nhiều "supervisors". Theo mặc định, file cấu hình định nghĩa supervisor này là `supervisor-1`; tuy nhiên, bạn có thể tự do đặt tên cho supervisors của mình bất cứ gì bạn muốn. Mỗi supervisor về cơ bản chịu trách nhiệm "giám sát" một nhóm process worker và lo việc cân bằng process worker trên các queues.

Bạn có thể thêm supervisors bổ sung vào một môi trường nhất định nếu bạn muốn định nghĩa một nhóm process worker mới nên chạy trong môi trường đó. Bạn có thể chọn làm điều này nếu bạn muốn định nghĩa một chiến lược cân bằng khác hoặc số lượng process worker khác cho một queue nhất định được sử dụng bởi ứng dụng của bạn.

<a name="maintenance-mode"></a>
#### Maintenance Mode

Trong khi ứng dụng của bạn ở trong [maintenance mode](/docs/{{version}}/configuration#maintenance-mode), queued jobs sẽ không được xử lý bởi Horizon trừ khi tùy chọn `force` của supervisor được định nghĩa là `true` trong file cấu hình Horizon:

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

Trong file cấu hình mặc định của Horizon, bạn sẽ nhận thấy một tùy chọn cấu hình `defaults`. Tùy chọn cấu hình này chỉ định các giá trị mặc định cho [supervisors](#supervisors) của ứng dụng của bạn. Các giá trị cấu hình mặc định của supervisor sẽ được hợp nhất vào cấu hình supervisor cho mỗi môi trường, cho phép bạn tránh sự lặp lại không cần thiết khi định nghĩa supervisors của mình.

<a name="dashboard-authorization"></a>
### Dashboard Authorization

Dashboard Horizon có thể được truy cập qua route `/horizon`. Theo mặc định, bạn chỉ có thể truy cập dashboard này trong môi trường `local`. Tuy nhiên, trong file `app/Providers/HorizonServiceProvider.php` của bạn, có một định nghĩa [authorization gate](/docs/{{version}}/authorization#gates). Authorization gate này kiểm soát quyền truy cập vào Horizon trong các môi trường **non-local**. Bạn có thể tự do sửa đổi gate này khi cần để hạn chế quyền truy cập vào cài đặt Horizon của bạn:

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
#### Chiến lược Authentication Thay thế

Hãy nhớ rằng Laravel tự động inject user được xác thực vào gate closure. Nếu ứng dụng của bạn đang cung cấp security Horizon thông qua một phương thức khác, chẳng hạn như hạn chế IP, thì người dùng Horizon của bạn có thể không cần "login". Do đó, bạn sẽ cần thay đổi signature closure `function (User $user)` ở trên thành `function (User $user = null)` để buộc Laravel không yêu cầu authentication.

<a name="max-job-attempts"></a>
### Số lần Thử Job Tối đa

> [!NOTE]
> Trước khi tinh chỉnh các tùy chọn này, hãy đảm bảo bạn đã quen với [queue services mặc định](/docs/{{version}}/queues#max-job-attempts-and-timeout) của Laravel và khái niệm 'attempts'.

Bạn có thể định nghĩa số lần thử tối đa mà một job có thể tiêu thụ trong cấu hình của một supervisor:

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
> Tùy chọn này tương tự như tùy chọn `--tries` khi sử dụng command Artisan để xử lý queues.

Điều chỉnh tùy chọn `tries` là cần thiết khi sử dụng middlewares như `WithoutOverlapping` hoặc `RateLimited` vì chúng tiêu thụ attempts. Để xử lý điều này, điều chỉnh giá trị cấu hình `tries` ở cấp supervisor hoặc bằng cách định nghĩa property `$tries` trên class job.

Nếu bạn không đặt tùy chọn `tries`, Horizon mặc định là một lần thử, trừ khi class job định nghĩa `$tries`, điều này có quyền ưu tiên hơn cấu hình Horizon.

Đặt `tries` hoặc `$tries` thành 0 cho phép số lần thử không giới hạn, điều này lý tưởng khi số lần thử không chắc chắn. Để ngăn chặn các lỗi vô tận, bạn có thể giới hạn số lượng exceptions được phép bằng cách đặt property `$maxExceptions` trên class job.

<a name="job-timeout"></a>
### Job Timeout

Tương tự, bạn có thể đặt giá trị `timeout` ở cấp supervisor, chỉ định bao nhiêu giây một process worker có thể chạy một job trước khi nó bị chấm dứt mạnh mẽ. Sau khi bị chấm dứt, job sẽ được thử lại hoặc được đánh dấu là failed, tùy thuộc vào cấu hình queue của bạn:

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...¨
            'timeout' => 60,
        ],
    ],
],
```

> [!WARNING]
> Khi sử dụng chiến lược cân bằng `auto`, Horizon sẽ coi các worker đang tiến hành là "hanging" và force-kill chúng sau timeout của Horizon trong quá trình scale down. Luôn đảm bảo timeout của Horizon lớn hơn bất kỳ timeout cấp job nào, nếu không jobs có thể bị chấm dứt giữa quá trình thực thi. Ngoài ra, giá trị `timeout` phải luôn ngắn hơn vài giây so với giá trị `retry_after` được định nghĩa trong file cấu hình `config/queue.php` của bạn. Nếu không, jobs của bạn có thể được xử lý hai lần.

<a name="job-backoff"></a>
### Job Backoff

Bạn có thể định nghĩa giá trị `backoff` ở cấp supervisor để chỉ định bao lâu Horizon nên đợi trước khi thử lại một job gặp một exception không được xử lý:

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

Bạn cũng có thể cấu hình "exponential" backoffs bằng cách sử dụng một mảng cho giá trị `backoff`. Trong ví dụ này, độ trễ thử lại sẽ là 1 giây cho lần thử đầu tiên, 5 giây cho lần thử thứ hai, 10 giây cho lần thử thứ ba, và 10 giây cho mỗi lần thử tiếp theo nếu còn nhiều lần thử còn lại:

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

Đôi khi, bạn có thể không quan tâm đến việc xem các jobs nhất định được dispatch bởi ứng dụng của bạn hoặc các packages bên thứ ba. Thay vì các jobs này chiếm không gian trong danh sách "Completed Jobs" của bạn, bạn có thể tắt tiếng chúng. Để bắt đầu, thêm tên class của job vào tùy chọn cấu hình `silenced` trong file cấu hình `horizon` của ứng dụng của bạn:

```php
'silenced' => [
    App\Jobs\ProcessPodcast::class,
],
```

Ngoài việc tắt tiếng các class job riêng lẻ, Horizon cũng hỗ trợ tắt tiếng jobs dựa trên [tags](#tags). Điều này có thể hữu ích nếu bạn muốn ẩn nhiều jobs chia sẻ một tag chung:

```php
'silenced_tags' => [
    'notifications'
],
```

Ngoài ra, job bạn muốn tắt tiếng có thể implement interface `Laravel\Horizon\Contracts\Silenced`. Nếu một job implement interface này, nó sẽ tự động bị tắt tiếng, ngay cả khi nó không có trong mảng cấu hình `silenced`:

```php
use Laravel\Horizon\Contracts\Silenced;

class ProcessPodcast implements ShouldQueue, Silenced
{
    use Queueable;

    // ...
}
```

<a name="balancing-strategies"></a>
## Chiến lược Cân bằng

Mỗi supervisor có thể xử lý một hoặc nhiều queues nhưng không giống như hệ thống queue mặc định của Laravel, Horizon cho phép bạn chọn từ ba chiến lược cân bằng worker: `auto`, `simple`, và `false`.

<a name="auto-balancing"></a>
### Auto Balancing

Chiến lược `auto`, là chiến lược mặc định, điều chỉnh số lượng process worker mỗi queue dựa trên workload hiện tại của queue. Ví dụ, nếu queue `notifications` của bạn có 1,000 pending jobs trong khi queue `default` của bạn trống, Horizon sẽ phân bổ nhiều worker hơn cho queue `notifications` của bạn cho đến khi queue trống.

Khi sử dụng chiến lược `auto`, bạn cũng có thể cấu hình các tùy chọn cấu hình `minProcesses` và `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` định nghĩa số lượng process worker tối thiểu mỗi queue. Giá trị này phải lớn hơn hoặc bằng 1.
- `maxProcesses` định nghĩa số lượng process worker tối đa tổng thể mà Horizon có thể scale lên trên tất cả các queues. Giá trị này thường nên lớn hơn số lượng queues nhân với giá trị `minProcesses`. Để ngăn supervisor tạo ra bất kỳ process nào, bạn có thể đặt giá trị này thành 0.

</div>

Ví dụ, bạn có thể cấu hình Horizon để duy trì ít nhất một process mỗi queue và scale lên tổng cộng 10 process worker:

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

Tùy chọn cấu hình `autoScalingStrategy` xác định cách Horizon sẽ phân bổ nhiều process worker hơn cho queues. Bạn có thể chọn giữa hai chiến lược:

<div class="content-list" markdown="1">

- Chiến lược `time` sẽ phân bổ worker dựa trên tổng thời gian ước tính sẽ mất để xóa queue.
- Chiến lược `size` sẽ phân bổ worker dựa trên tổng số jobs trên queue.

</div>

Các giá trị cấu hình `balanceMaxShift` và `balanceCooldown` xác định tốc độ Horizon sẽ scale để đáp ứng nhu cầu worker. Trong ví dụ trên, tối đa một process mới sẽ được tạo hoặc hủy mỗi ba giây. Bạn có thể tự do tinh chỉnh các giá trị này khi cần dựa trên nhu cầu của ứng dụng của bạn.

<a name="auto-queue-priorities"></a>
#### Queue Priorities và Auto Balancing

Khi sử dụng chiến lược cân bằng `auto`, Horizon không thực thi ưu tiên nghiêm ngặt giữa các queues. Thứ tự của các queues trong cấu hình của supervisor không ảnh hưởng đến cách process worker được phân bổ. Thay vào đó, Horizon dựa vào `autoScalingStrategy` được chọn để phân bổ process worker động dựa trên tải queue.

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

Nếu bạn cần thực thi ưu tiên tương đối giữa các queues, bạn có thể định nghĩa nhiều supervisors và phân bổ rõ ràng tài nguyên xử lý:

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

Trong ví dụ này, `queue` mặc định có thể scale lên đến 10 processes, trong khi queue `images` bị giới hạn một process. Cấu hình này đảm bảo rằng queues của bạn có thể scale độc lập.

> [!NOTE]
> Khi dispatch các jobs tiêu tốn nhiều tài nguyên, đôi khi tốt nhất là gán chúng cho một queue chuyên dụng với giá trị `maxProcesses` giới hạn. Nếu không, các jobs này có thể tiêu thụ quá nhiều tài nguyên CPU và quá tải hệ thống của bạn.

<a name="simple-balancing"></a>
### Simple Balancing

Chiến lược `simple` phân phối process worker đều trên các queues được chỉ định. Với chiến lược này, Horizon không tự động scale số lượng process worker. Thay vào đó, nó sử dụng một số lượng processes cố định:

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

Trong ví dụ trên, Horizon sẽ phân bổ 5 processes cho mỗi queue, chia tổng số 10 đều.

Nếu bạn muốn kiểm soát số lượng process worker được phân bổ cho mỗi queue riêng lẻ, bạn có thể định nghĩa nhiều supervisors:

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

Với cấu hình này, Horizon sẽ phân bổ 10 processes cho queue `default` và 2 processes cho queue `notifications`.

<a name="no-balancing"></a>
### No Balancing

Khi tùy chọn `balance` được đặt thành `false`, Horizon xử lý các queues nghiêm ngặt theo thứ tự chúng được liệt kê, tương tự như hệ thống queue mặc định của Laravel. Tuy nhiên, nó vẫn sẽ scale số lượng process worker nếu jobs bắt đầu tích lũy:

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

Trong ví dụ trên, jobs trong queue `default` luôn được ưu tiên hơn jobs trong queue `notifications`. Ví dụ, nếu có 1,000 jobs trong `default` và chỉ 10 trong `notifications`, Horizon sẽ xử lý hoàn toàn tất cả jobs `default` trước khi xử lý bất kỳ jobs nào từ `notifications`.

Bạn có thể kiểm soát khả năng scale process worker của Horizon sử dụng các tùy chọn `minProcesses` và `maxProcesses`:

<div class="content-list" markdown="1">

- `minProcesses` định nghĩa số lượng process worker tối thiểu tổng thể. Giá trị này phải lớn hơn hoặc bằng 1.
- `maxProcesses` định nghĩa số lượng process worker tối đa tổng thể mà Horizon có thể scale lên.

</div>

<a name="upgrading-horizon"></a>
## Nâng cấp Horizon

Khi nâng cấp lên phiên bản chính mới của Horizon, điều quan trọng là bạn xem xét kỹ [hướng dẫn nâng cấp](https://github.com/laravel/horizon/blob/master/UPGRADE.md).

<a name="running-horizon"></a>
## Chạy Horizon

Sau khi bạn đã cấu hình supervisors và workers của mình trong file cấu hình `config/horizon.php` của ứng dụng, bạn có thể khởi động Horizon sử dụng command `horizon` của Artisan. Command đơn lẻ này sẽ khởi động tất cả các process worker được cấu hình cho môi trường hiện tại:

```shell
php artisan horizon
```

Bạn có thể tạm dừng process Horizon và hướng dẫn nó tiếp tục xử lý jobs sử dụng các command `horizon:pause` và `horizon:continue` của Artisan:

```shell
php artisan horizon:pause

php artisan horizon:continue
```

Bạn cũng có thể tạm dừng và tiếp tục các [supervisors](#supervisors) Horizon cụ thể sử dụng các command `horizon:pause-supervisor` và `horizon:continue-supervisor` của Artisan:

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

Bạn có thể kiểm tra trạng thái hiện tại của process Horizon sử dụng command `horizon:status` của Artisan:

```shell
php artisan horizon:status
```

Bạn có thể kiểm tra trạng thái hiện tại của một [supervisor](#supervisors) Horizon cụ thể sử dụng command `horizon:supervisor-status` của Artisan:

```shell
php artisan horizon:supervisor-status supervisor-1
```

Bạn có thể chấm dứt một cách nhẹ nhàng process Horizon sử dụng command `horizon:terminate` của Artisan. Bất kỳ jobs nào đang được xử lý sẽ được hoàn thành và sau đó Horizon sẽ ngừng thực thi:

```shell
php artisan horizon:terminate
```

<a name="automatically-restarting-horizon"></a>
#### Tự động Khởi động lại Horizon

Trong quá trình phát triển cục bộ, bạn có thể chạy command `horizon:listen`. Khi sử dụng command `horizon:listen`, bạn không phải khởi động lại Horizon thủ công khi bạn muốn tải lại code đã cập nhật của mình. Trước khi sử dụng tính năng này, bạn nên đảm bảo rằng [Node](https://nodejs.org) được cài đặt trong môi trường phát triển cục bộ của bạn. Ngoài ra, bạn nên cài đặt [Chokidar](https://github.com/paulmillr/chokidar) library theo dõi file trong dự án của mình:

```shell
npm install --save-dev chokidar
```

Sau khi Chokidar được cài đặt, bạn có thể khởi động Horizon sử dụng command `horizon:listen`:

```shell
php artisan horizon:listen
```

Khi chạy trong Docker hoặc Vagrant, bạn nên sử dụng tùy chọn `--poll`:

```shell
php artisan horizon:listen --poll
```

Bạn có thể cấu hình các thư mục và file nên được theo dõi sử dụng tùy chọn cấu hình `watch` trong file cấu hình `config/horizon.php` của ứng dụng của bạn:

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
### Triển khai Horizon

Khi bạn sẵn sàng triển khai Horizon lên server thực tế của ứng dụng, bạn nên cấu hình một process monitor để giám sát command `php artisan horizon` và khởi động lại nó nếu nó thoát bất ngờ. Đừng lo lắng, chúng ta sẽ thảo luận cách cài đặt một process monitor dưới đây.

Trong quá trình triển khai ứng dụng của bạn, bạn nên hướng dẫn process Horizon chấm dứt để nó sẽ được khởi động lại bởi process monitor của bạn và nhận các thay đổi code của bạn:

```shell
php artisan horizon:terminate
```

<a name="installing-supervisor"></a>
#### Cài đặt Supervisor

Supervisor là một process monitor cho hệ điều hành Linux và sẽ tự động khởi động lại process `horizon` của bạn nếu nó ngừng thực thi. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng command sau. Nếu bạn không sử dụng Ubuntu, bạn có thể cài đặt Supervisor sử dụng package manager của hệ điều hành của bạn:

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> Nếu tự cấu hình Supervisor nghe có vẻ quá sức, hãy xem xét sử dụng [Laravel Cloud](https://cloud.laravel.com), có thể quản lý các background processes cho các ứng dụng Laravel của bạn.

<a name="supervisor-configuration"></a>
#### Cấu hình Supervisor

Các file cấu hình Supervisor thường được lưu trữ trong thư mục `/etc/supervisor/conf.d` của server. Trong thư mục này, bạn có thể tạo bất kỳ số lượng file cấu hình nào hướng dẫn supervisor cách các processes của bạn nên được giám sát. Ví dụ, hãy tạo một file `horizon.conf` khởi động và giám sát một process `horizon`:

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
> Trong khi các ví dụ trên hợp lệ cho các server dựa trên Ubuntu, vị trí và phần mở rộng file được mong đợi của các file cấu hình Supervisor có thể thay đổi giữa các hệ điều hành server khác. Vui lòng tham khảo tài liệu của server của bạn để biết thêm thông tin.

<a name="starting-supervisor"></a>
#### Khởi động Supervisor

Sau khi file cấu hình đã được tạo, bạn có thể cập nhật cấu hình Supervisor và khởi động các processes được giám sát sử dụng các command sau:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> [!NOTE]
> Để biết thêm thông tin về việc chạy Supervisor, hãy tham khảo [tài liệu Supervisor](http://supervisord.org/index.html).

<a name="tags"></a>
## Tags

Horizon cho phép bạn gán "tags" cho jobs, bao gồm mailables, broadcast events, notifications, và queued event listeners. Thực tế, Horizon sẽ thông minh và tự động tag hầu hết các jobs tùy thuộc vào các Eloquent models được gắn vào job. Ví dụ, hãy xem job sau:

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

Nếu job này được queued với một instance `App\Models\Video` có attribute `id` là `1`, nó sẽ tự động nhận tag `App\Models\Video:1`. Điều này là do Horizon sẽ tìm kiếm các properties của job cho bất kỳ Eloquent models nào. Nếu Eloquent models được tìm thấy, Horizon sẽ thông minh tag job sử dụng tên class của model và primary key:

```php
use App\Jobs\RenderVideo;
use App\Models\Video;

$video = Video::find(1);

RenderVideo::dispatch($video);
```

<a name="manually-tagging-jobs"></a>
#### Tagging Jobs Thủ công

Nếu bạn muốn định nghĩa thủ công các tags cho một trong các queueable objects của mình, bạn có thể định nghĩa một phương thức `tags` trên class:

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
#### Tagging Event Listeners Thủ công

Khi lấy các tags cho một queued event listener, Horizon sẽ tự động truyền event instance vào phương thức `tags`, cho phép bạn thêm dữ liệu event vào các tags:

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

Nếu bạn muốn được thông báo khi một trong các queues của bạn có thời gian chờ dài, bạn có thể sử dụng các phương thức `Horizon::routeMailNotificationsTo`, `Horizon::routeSlackNotificationsTo`, và `Horizon::routeSmsNotificationsTo`. Bạn có thể gọi các phương thức này từ phương thức `boot` của `App\Providers\HorizonServiceProvider` của ứng dụng của bạn:

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
#### Cấu hình Ngưỡng Thời gian Chờ Notification

Bạn có thể cấu hình bao nhiêu giây được coi là "long wait" trong file cấu hình `config/horizon.php` của ứng dụng của bạn. Tùy chọn cấu hình `waits` trong file này cho phép bạn kiểm soát ngưỡng chờ dài cho mỗi kết nối / queue combination. Bất kỳ kết nối / queue combination nào không được định nghĩa sẽ mặc định là ngưỡng chờ dài 60 giây:

```php
'waits' => [
    'redis:critical' => 30,
    'redis:default' => 60,
    'redis:batch' => 120,
],
```

Đặt ngưỡng của một queue thành `0` sẽ vô hiệu hóa long wait notifications cho queue đó.

<a name="metrics"></a>
## Metrics

Horizon bao gồm một metrics dashboard cung cấp thông tin về thời gian chờ và throughput của job và queue của bạn. Để điền dashboard này, bạn nên cấu hình command `snapshot` của Horizon để chạy mỗi năm phút trong file `routes/console.php` của ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

Nếu bạn muốn xóa tất cả dữ liệu metrics, bạn có thể gọi command `horizon:clear-metrics` của Artisan:

```shell
php artisan horizon:clear-metrics
```

<a name="deleting-failed-jobs"></a>
## Xóa Failed Jobs

Nếu bạn muốn xóa một failed job, bạn có thể sử dụng command `horizon:forget`. Command `horizon:forget` chấp nhận ID hoặc UUID của failed job làm đối số duy nhất của nó:

```shell
php artisan horizon:forget 5
```

Nếu bạn muốn xóa tất cả failed jobs, bạn có thể cung cấp tùy chọn `--all` cho command `horizon:forget`:

```shell
php artisan horizon:forget --all
```

<a name="clearing-jobs-from-queues"></a>
## Xóa Jobs khỏi Queues

Nếu bạn muốn xóa tất cả jobs khỏi queue mặc định của ứng dụng, bạn có thể làm như vậy sử dụng command `horizon:clear` của Artisan:

```shell
php artisan horizon:clear
```

Bạn có thể cung cấp tùy chọn `queue` để xóa jobs từ một queue cụ thể:

```shell
php artisan horizon:clear --queue=emails
```

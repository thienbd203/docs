# Task Scheduling

- [Introduction](#introduction)
- [Defining Schedules](#defining-schedules)
    - [Scheduling Artisan Commands](#scheduling-artisan-commands)
    - [Scheduling Queued Jobs](#scheduling-queued-jobs)
    - [Scheduling Shell Commands](#scheduling-shell-commands)
    - [Schedule Frequency Options](#schedule-frequency-options)
    - [Timezones](#timezones)
    - [Preventing Task Overlaps](#preventing-task-overlaps)
    - [Running Tasks on One Server](#running-tasks-on-one-server)
    - [Background Tasks](#background-tasks)
    - [Maintenance Mode](#maintenance-mode)
    - [Pausing Scheduled Tasks](#pausing-scheduled-tasks)
    - [Schedule Groups](#schedule-groups)
- [Running the Scheduler](#running-the-scheduler)
    - [Sub-Minute Scheduled Tasks](#sub-minute-scheduled-tasks)
    - [Running the Scheduler Locally](#running-the-scheduler-locally)
- [Task Output](#task-output)
- [Task Hooks](#task-hooks)
- [Events](#events)

<a name="introduction"></a>
## Introduction

Trong quá khứ, bạn có thể đã viết một mục cấu hình cron cho mỗi task bạn cần schedule trên server của mình. Tuy nhiên, điều này có thể nhanh chóng trở nên khó chịu vì task schedule của bạn không còn trong source control và bạn phải SSH vào server để xem các mục cron hiện có hoặc thêm các mục bổ sung.

Command scheduler của Laravel cung cấp một cách tiếp cận mới để quản lý các scheduled tasks trên server của bạn. Scheduler cho phép bạn định nghĩa command schedule của mình một cách trôi chảy và expressive trong chính ứng dụng Laravel của bạn. Khi sử dụng scheduler, chỉ cần một mục cron duy nhất trên server của bạn. Task schedule của bạn thường được định nghĩa trong file `routes/console.php` của ứng dụng của bạn.

<a name="defining-schedules"></a>
## Defining Schedules

Bạn có thể định nghĩa tất cả các scheduled tasks của bạn trong file `routes/console.php` của ứng dụng. Để bắt đầu, hãy xem một ví dụ. Trong ví dụ này, chúng ta sẽ schedule một closure được gọi mỗi ngày vào nửa đêm. Trong closure, chúng ta sẽ thực thi một database query để xóa một bảng:

```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

In addition to scheduling using closures, you may also schedule [invokable objects](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke). Invokable objects are simple PHP classes that contain an `__invoke` method:

```php
Schedule::call(new DeleteRecentUsers)->daily();
```

If you prefer to reserve your `routes/console.php` file for command definitions only, you may use the `withSchedule` method in your application's `bootstrap/app.php` file to define your scheduled tasks. This method accepts a closure that receives an instance of the scheduler:

```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

If you would like to view an overview of your scheduled tasks and the next time they are scheduled to run, you may use the `schedule:list` Artisan command:

```shell
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### Scheduling Artisan Commands

Ngoài việc schedule closures, bạn cũng có thể schedule [Artisan commands](/docs/{{version}}/artisan) và system commands. Ví dụ, bạn có thể sử dụng phương thức `command` để schedule một Artisan command sử dụng tên hoặc class của command.

When scheduling Artisan commands using the command's class name, you may pass an array of additional command-line arguments that should be provided to the command when it is invoked:

```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-artisan-closure-commands"></a>
#### Scheduling Artisan Closure Commands

If you want to schedule an Artisan command defined by a closure, you may chain the scheduling related methods after the command's definition:

```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Delete recent users')->daily();
```

If you need to pass arguments to the closure command, you may provide them to the `schedule` method:

```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### Scheduling Queued Jobs

Phương thức `job` có thể được sử dụng để schedule một [queued job](/docs/{{version}}/queues). Phương thức này cung cấp một cách thuận tiện để schedule queued jobs mà không cần sử dụng phương thức `call` để định nghĩa closures để queue job:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```

Optional second and third arguments may be provided to the `job` method which specifies the queue name and queue connection that should be used to queue the job:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// Dispatch the job to the "heartbeats" queue on the "sqs" connection...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

<a name="scheduling-shell-commands"></a>
### Scheduling Shell Commands

Phương thức `exec` có thể được sử dụng để issue một command đến hệ điều hành:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

<a name="schedule-frequency-options"></a>
### Schedule Frequency Options

Chúng ta đã thấy một vài ví dụ về cách bạn có thể cấu hình một task để chạy tại các khoảng thời gian cụ thể. Tuy nhiên, có nhiều tần suất task schedule hơn mà bạn có thể gán cho một task:

<div class="overflow-auto">

| Method                             | Description                                              |
| ---------------------------------- | -------------------------------------------------------- |
| `->cron('* * * * *');`             | Run the task on a custom cron schedule.                  |
| `->everySecond();`                 | Run the task every second.                               |
| `->everyTwoSeconds();`             | Run the task every two seconds.                          |
| `->everyFiveSeconds();`            | Run the task every five seconds.                         |
| `->everyTenSeconds();`             | Run the task every ten seconds.                          |
| `->everyFifteenSeconds();`         | Run the task every fifteen seconds.                      |
| `->everyTwentySeconds();`          | Run the task every twenty seconds.                       |
| `->everyThirtySeconds();`          | Run the task every thirty seconds.                       |
| `->everyMinute();`                 | Run the task every minute.                               |
| `->everyTwoMinutes();`             | Run the task every two minutes.                          |
| `->everyThreeMinutes();`           | Run the task every three minutes.                        |
| `->everyFourMinutes();`            | Run the task every four minutes.                         |
| `->everyFiveMinutes();`            | Run the task every five minutes.                         |
| `->everyTenMinutes();`             | Run the task every ten minutes.                          |
| `->everyFifteenMinutes();`         | Run the task every fifteen minutes.                      |
| `->everyThirtyMinutes();`          | Run the task every thirty minutes.                       |
| `->hourly();`                      | Run the task every hour.                                 |
| `->hourlyAt(17);`                  | Run the task every hour at 17 minutes past the hour.     |
| `->everyOddHour($minutes = 0);`    | Run the task every odd hour.                             |
| `->everyTwoHours($minutes = 0);`   | Run the task every two hours.                            |
| `->everyThreeHours($minutes = 0);` | Run the task every three hours.                          |
| `->everyFourHours($minutes = 0);`  | Run the task every four hours.                           |
| `->everySixHours($minutes = 0);`   | Run the task every six hours.                            |
| `->daily();`                       | Run the task every day at midnight.                      |
| `->dailyAt('13:00');`              | Run the task every day at 13:00.                         |
| `->twiceDaily(1, 13);`             | Run the task daily at 1:00 & 13:00.                      |
| `->twiceDailyAt(1, 13, 15);`       | Run the task daily at 1:15 & 13:15.                      |
| `->daysOfMonth([1, 10, 20]);`      | Run the task on specific days of the month.              |
| `->weekly();`                      | Run the task every Sunday at 00:00.                      |
| `->weeklyOn(1, '8:00');`           | Run the task every week on Monday at 8:00.               |
| `->monthly();`                     | Run the task on the first day of every month at 00:00.   |
| `->monthlyOn(4, '15:00');`         | Run the task every month on the 4th at 15:00.            |
| `->twiceMonthly(1, 16, '13:00');`  | Run the task monthly on the 1st and 16th at 13:00.       |
| `->lastDayOfMonth('15:00');`       | Run the task on the last day of the month at 15:00.      |
| `->quarterly();`                   | Run the task on the first day of every quarter at 00:00. |
| `->quarterlyOn(4, '14:00');`       | Run the task every quarter on the 4th at 14:00.          |
| `->yearly();`                      | Run the task on the first day of every year at 00:00.    |
| `->yearlyOn(6, 1, '17:00');`       | Run the task every year on June 1st at 17:00.            |
| `->timezone('America/New_York');`  | Set the timezone for the task.                           |

</div>

Các phương thức này có thể được kết hợp với các ràng buộc bổ sung để tạo ra các schedule tinh chỉnh hơn chỉ chạy vào những ngày cụ thể trong tuần. Ví dụ, bạn có thể schedule một command để chạy hàng tuần vào Thứ Hai:

```php
use Illuminate\Support\Facades\Schedule;

// Run once per week on Monday at 1 PM...
Schedule::call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
Schedule::command('foo')
    ->weekdays()
    ->hourly()
    ->timezone('America/Chicago')
    ->between('8:00', '17:00');
```

Danh sách các ràng buộc schedule bổ sung có thể được tìm thấy dưới đây:

<div class="overflow-auto">

| Method                                   | Description                                            |
| ---------------------------------------- | ------------------------------------------------------ |
| `->weekdays();`                          | Limit the task to weekdays.                            |
| `->weekends();`                          | Limit the task to weekends.                            |
| `->sundays();`                           | Limit the task to Sunday.                              |
| `->mondays();`                           | Limit the task to Monday.                              |
| `->tuesdays();`                          | Limit the task to Tuesday.                             |
| `->wednesdays();`                        | Limit the task to Wednesday.                           |
| `->thursdays();`                         | Limit the task to Thursday.                            |
| `->fridays();`                           | Limit the task to Friday.                              |
| `->saturdays();`                         | Limit the task to Saturday.                            |
| `->days(array\|mixed);`                  | Limit the task to specific days.                       |
| `->between($startTime, $endTime);`       | Limit the task to run between start and end times.     |
| `->unlessBetween($startTime, $endTime);` | Limit the task to not run between start and end times. |
| `->when(Closure);`                       | Limit the task based on a truth test.                  |
| `->environments($env);`                  | Limit the task to specific environments.               |

</div>

<a name="day-constraints"></a>
#### Day Constraints

Phương thức `days` có thể được sử dụng để giới hạn việc thực thi của một task vào những ngày cụ thể trong tuần. Ví dụ, bạn có thể schedule một command để chạy hàng giờ vào Chủ Nhật và Thứ Tư:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```

Ngoài ra, bạn có thể sử dụng các hằng số có sẵn trên class `Illuminate\Console\Scheduling\Schedule` khi định nghĩa các ngày mà một task nên chạy:

```php
use Illuminate\Support\Facades;
use Illuminate\Console\Scheduling\Schedule;

Facades\Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

<a name="between-time-constraints"></a>
#### Between Time Constraints

Phương thức `between` có thể được sử dụng để giới hạn việc thực thi của một task dựa trên thời gian trong ngày:

```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```

Tương tự, phương thức `unlessBetween` có thể được sử dụng để loại trừ việc thực thi của một task trong một khoảng thời gian:

```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

<a name="truth-test-constraints"></a>
#### Truth Test Constraints

Phương thức `when` có thể được sử dụng để giới hạn việc thực thi của một task dựa trên kết quả của một truth test cụ thể. Nói cách khác, nếu closure được cung cấp trả về `true`, task sẽ thực thi miễn là không có điều kiện ràng buộc nào khác ngăn task chạy:

```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```

Phương thức `skip` có thể được xem là nghịch đảo của `when`. Nếu phương thức `skip` trả về `true`, scheduled task sẽ không được thực thi:

```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```

Khi sử dụng các phương thức `when` được chain, scheduled command sẽ chỉ thực thi nếu tất cả các điều kiện `when` trả về `true`.

<a name="environment-constraints"></a>
#### Environment Constraints

Phương thức `environments` có thể được sử dụng để thực thi tasks chỉ trên các môi trường được cung cấp (như được định nghĩa bởi [environment variable](/docs/{{version}}/configuration#environment-configuration) `APP_ENV`):

```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

<a name="timezones"></a>
### Timezones

Sử dụng phương thức `timezone`, bạn có thể xác định rằng thời gian của một scheduled task nên được diễn giải trong một timezone cụ thể:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```

Nếu bạn liên tục gán cùng một timezone cho tất cả các scheduled tasks của mình, bạn có thể xác định timezone nào nên được gán cho tất cả schedules bằng cách định nghĩa một tùy chọn `schedule_timezone` trong file cấu hình `app` của ứng dụng của bạn:

```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```

> [!WARNING]
> Nhớ rằng một số timezones sử dụng daylight savings time. Khi thay đổi daylight saving time xảy ra, scheduled task của bạn có thể chạy hai lần hoặc thậm chí không chạy tại tất cả. Vì lý do này, chúng tôi khuyên tránh timezone scheduling khi có thể.

<a name="preventing-task-overlaps"></a>
### Preventing Task Overlaps

Theo mặc định, scheduled tasks sẽ được chạy ngay cả khi instance trước của task vẫn đang chạy. Để ngăn điều này, bạn có thể sử dụng phương thức `withoutOverlapping`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```

Trong ví dụ này, [Artisan command](/docs/{{version}}/artisan) `emails:send` sẽ được chạy mỗi phút nếu nó chưa đang chạy. Phương thức `withoutOverlapping` đặc biệt hữu ích nếu bạn có các tasks thay đổi đáng kể về thời gian thực thi, ngăn bạn dự đoán chính xác một task cụ thể sẽ mất bao lâu.

Nếu cần thiết, bạn có thể xác định bao nhiêu phút phải trôi qua trước khi "without overlapping" lock hết hạn. Theo mặc định, lock sẽ hết hạn sau 24 giờ:

```php
Schedule::command('emails:send')->withoutOverlapping(10);
```

Behind the scenes, phương thức `withoutOverlapping` sử dụng [cache](/docs/{{version}}/cache) của ứng dụng của bạn để lấy locks. Nếu cần thiết, bạn có thể xóa các cache locks này sử dụng lệnh Artisan `schedule:clear-cache`. Điều này thường chỉ cần thiết nếu một task bị kẹt do một vấn đề server không mong muốn.

<a name="running-tasks-on-one-server"></a>
### Running Tasks on One Server

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng driver cache `database`, `memcached`, `dynamodb`, hoặc `redis` làm driver cache mặc định của ứng dụng. Ngoài ra, tất cả các servers phải giao tiếp với cùng một central cache server.

Nếu scheduler của ứng dụng của bạn đang chạy trên nhiều servers, bạn có thể giới hạn một scheduled job chỉ thực thi trên một server duy nhất. Ví dụ, giả sử bạn có một scheduled task tạo một báo cáo mới mỗi tối thứ Sáu. Nếu task scheduler đang chạy trên ba worker servers, scheduled task sẽ chạy trên cả ba servers và tạo báo cáo ba lần. Không tốt!

Để chỉ ra rằng task nên chỉ chạy trên một server, sử dụng phương thức `onOneServer` khi định nghĩa scheduled task. Server đầu tiên lấy được task sẽ secure một atomic lock trên job để ngăn các servers khác chạy cùng task đó cùng một lúc:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

Bạn có thể sử dụng phương thức `useCache` để tùy chỉnh cache store được sử dụng bởi scheduler để lấy các atomic locks cần thiết cho các single-server tasks:

```php
Schedule::useCache('database');
```

<a name="naming-unique-jobs"></a>
#### Naming Single Server Jobs

Đôi khi bạn cần schedule cùng một job được dispatch với các tham số khác nhau, trong khi vẫn hướng dẫn Laravel chạy mỗi permutation của job trên một server duy nhất. Để thực hiện điều này, bạn có thể gán mỗi định nghĩa schedule một tên duy nhất thông qua phương thức `name`:

```php
Schedule::job(new CheckUptime('https://laravel.com'))
    ->name('check_uptime:laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();

Schedule::job(new CheckUptime('https://vapor.laravel.com'))
    ->name('check_uptime:vapor.laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();
```

Tương tự, scheduled closures phải được gán một tên nếu chúng dự định chạy trên một server:

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

<a name="background-tasks"></a>
### Background Tasks

Theo mặc định, nhiều tasks được schedule cùng một lúc sẽ thực thi tuần tự dựa trên thứ tự chúng được định nghĩa trong phương thức `schedule` của bạn. Nếu bạn có các tasks chạy dài, điều này có thể khiến các tasks tiếp theo bắt đầu muộn hơn nhiều so với dự kiến. Nếu bạn muốn chạy tasks trong background để chúng có thể chạy đồng thời, bạn có thể sử dụng phương thức `runInBackground`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

> [!WARNING]
> Phương thức `runInBackground` chỉ có thể được sử dụng khi schedule tasks thông qua các phương thức `command` và `exec`.

<a name="maintenance-mode"></a>
### Maintenance Mode

Các scheduled tasks của ứng dụng của bạn sẽ không chạy khi ứng dụng đang trong [maintenance mode](/docs/{{version}}/configuration#maintenance-mode), vì chúng ta không muốn tasks của bạn can thiệp vào bất kỳ maintenance chưa hoàn thành nào bạn có thể thực hiện trên server của mình. Tuy nhiên, nếu bạn muốn force một task chạy ngay cả trong maintenance mode, bạn có thể gọi phương thức `evenInMaintenanceMode` khi định nghĩa task:

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

<a name="pausing-scheduled-tasks"></a>
### Pausing Scheduled Tasks

Bạn có thể tạm dừng xử lý scheduled task mà không cần thay đổi code đã deploy của bạn bằng cách sử dụng lệnh Artisan `schedule:pause`:

```shell
php artisan schedule:pause
```

Trong khi scheduler bị tạm dừng, không có scheduled tasks nào sẽ chạy. Bạn có thể tiếp tục xử lý scheduled task sử dụng lệnh `schedule:continue`:

```shell
php artisan schedule:continue
```

Nếu một task vẫn nên chạy trong khi scheduler bị tạm dừng, bạn có thể đánh dấu nó với phương thức `evenWhenPaused`:

```php
Schedule::command('emails:send')->evenWhenPaused();
```

<a name="schedule-groups"></a>
### Schedule Groups

Khi định nghĩa nhiều scheduled tasks với các cấu hình tương tự, bạn có thể sử dụng tính năng grouping task của Laravel để tránh lặp lại cùng một cài đặt cho mỗi task. Grouping tasks đơn giản hóa code của bạn và đảm bảo tính nhất quán trên các tasks liên quan.

Để tạo một nhóm scheduled tasks, gọi các phương thức cấu hình task mong muốn, theo sau là phương thức `group`. Phương thức `group` chấp nhận một closure chịu trách nhiệm định nghĩa các tasks chia sẻ cấu hình được chỉ định:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::daily()
    ->onOneServer()
    ->timezone('America/New_York')
    ->group(function () {
        Schedule::command('emails:send --force');
        Schedule::command('emails:prune');
    });
```

<a name="running-the-scheduler"></a>
## Running the Scheduler

Bây giờ chúng ta đã học cách định nghĩa scheduled tasks, hãy thảo luận cách thực sự chạy chúng trên server của chúng ta. Lệnh Artisan `schedule:run` sẽ đánh giá tất cả các scheduled tasks của bạn và xác định xem chúng có cần chạy dựa trên thời gian hiện tại của server hay không.

Vì vậy, khi sử dụng scheduler của Laravel, chúng ta chỉ cần thêm một mục cấu hình cron duy nhất vào server của chúng ta chạy lệnh `schedule:run` mỗi phút. Nếu bạn không biết cách thêm các mục cron vào server của mình, hãy cân nhắc sử dụng một managed platform như [Laravel Cloud](https://cloud.laravel.com) có thể quản lý việc thực thi scheduled task cho bạn:

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

<a name="sub-minute-scheduled-tasks"></a>
### Sub-Minute Scheduled Tasks

Trên hầu hết các hệ điều hành, cron jobs bị giới hạn chạy tối đa một lần mỗi phút. Tuy nhiên, scheduler của Laravel cho phép bạn schedule các tasks chạy tại các khoảng thời gian thường xuyên hơn, thậm chí thường xuyên như mỗi giây:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

Khi các sub-minute tasks được định nghĩa trong ứng dụng của bạn, lệnh `schedule:run` sẽ tiếp tục chạy cho đến cuối phút hiện tại thay vì thoát ngay lập tức. Điều này cho phép lệnh gọi tất cả các sub-minute tasks cần thiết trong suốt phút đó.

Vì các sub-minute tasks mất nhiều thời gian hơn dự kiến để chạy có thể trì hoãn việc thực thi của các sub-minute tasks sau, được khuyến nghị rằng tất cả các sub-minute tasks nên dispatch queued jobs hoặc background commands để xử lý việc xử lý task thực tế:

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

<a name="interrupting-sub-minute-tasks"></a>
#### Interrupting Sub-Minute Tasks

Vì lệnh `schedule:run` chạy cho toàn bộ phút invocation khi các sub-minute tasks được định nghĩa, bạn đôi khi có thể cần interrupt lệnh khi deploy ứng dụng của bạn. Nếu không, một instance của lệnh `schedule:run` đã đang chạy sẽ tiếp tục sử dụng code đã deploy trước đó của ứng dụng của bạn cho đến khi phút hiện tại kết thúc.

Để interrupt các invocation `schedule:run` đang tiến hành, bạn có thể thêm lệnh `schedule:interrupt` vào script deployment của ứng dụng của bạn. Lệnh này nên được gọi sau khi ứng dụng của bạn đã hoàn thành việc deploy:

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
### Running the Scheduler Locally

Thông thường, bạn sẽ không thêm một mục cron scheduler vào máy phát triển local của mình. Thay vào đó, bạn có thể sử dụng lệnh Artisan `schedule:work`. Lệnh này sẽ chạy trong foreground và gọi scheduler mỗi phút cho đến khi bạn terminate lệnh. Khi các sub-minute tasks được định nghĩa, scheduler sẽ tiếp tục chạy trong mỗi phút để xử lý các tasks đó:

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## Task Output

Scheduler của Laravel cung cấp một vài phương thức thuận tiện để làm việc với output được tạo bởi scheduled tasks. Đầu tiên, sử dụng phương thức `sendOutputTo`, bạn có thể gửi output đến một file để kiểm tra sau:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);
```

Nếu bạn muốn append output vào một file cụ thể, bạn có thể sử dụng phương thức `appendOutputTo`:

```php
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);
```

Sử dụng phương thức `emailOutputTo`, bạn có thể email output đến một địa chỉ email theo lựa chọn của bạn. Trước khi email output của một task, bạn nên cấu hình [email services](/docs/{{version}}/mail) của Laravel:

```php
Schedule::command('report:generate')
    ->daily()
    ->sendOutputTo($filePath)
    ->emailOutputTo('taylor@example.com');
```

Nếu bạn chỉ muốn email output nếu scheduled Artisan hoặc system command terminate với một exit code khác không, sử dụng phương thức `emailOutputOnFailure`:

```php
Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('taylor@example.com');
```

> [!WARNING]
> Các phương thức `emailOutputTo`, `emailOutputOnFailure`, `sendOutputTo`, và `appendOutputTo` chỉ dành riêng cho các phương thức `command` và `exec`.

<a name="task-hooks"></a>
## Task Hooks

Sử dụng các phương thức `before` và `after`, bạn có thể xác định code để được thực thi trước và sau khi scheduled task được thực thi:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->before(function () {
        // The task is about to execute...
    })
    ->after(function () {
        // The task has executed...
    });
```

Các phương thức `onSuccess` và `onFailure` cho phép bạn xác định code để được thực thi nếu scheduled task thành công hoặc thất bại. Một thất bại chỉ ra rằng scheduled Artisan hoặc system command terminate với một exit code khác không:

```php
Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function () {
        // The task succeeded...
    })
    ->onFailure(function () {
        // The task failed...
    });
```

Nếu output có sẵn từ command của bạn, bạn có thể truy cập nó trong các hooks `after`, `onSuccess` hoặc `onFailure` của bạn bằng cách type-hinting một instance `Illuminate\Support\Stringable` làm đối số `$output` của định nghĩa closure hook của bạn:

```php
use Illuminate\Support\Stringable;

Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function (Stringable $output) {
        // The task succeeded...
    })
    ->onFailure(function (Stringable $output) {
        // The task failed...
    });
```

<a name="pinging-urls"></a>
#### Pinging URLs

Sử dụng các phương thức `pingBefore` và `thenPing`, scheduler có thể tự động ping một URL cụ thể trước hoặc sau khi một task được thực thi. Phương thức này hữu ích để thông báo cho một dịch vụ bên ngoài, chẳng hạn như [Envoyer](https://envoyer.io), rằng scheduled task của bạn đang bắt đầu hoặc đã hoàn thành thực thi:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url);
```

Các phương thức `pingOnSuccess` và `pingOnFailure` có thể được sử dụng để ping một URL cụ thể chỉ nếu task thành công hoặc thất bại. Một thất bại chỉ ra rằng scheduled Artisan hoặc system command terminate với một exit code khác không:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

Các phương thức `pingBeforeIf`,`thenPingIf`,`pingOnSuccessIf`, và `pingOnFailureIf` có thể được sử dụng để ping một URL cụ thể chỉ nếu một điều kiện cụ thể là `true`:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBeforeIf($condition, $url)
    ->thenPingIf($condition, $url);

Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccessIf($condition, $successUrl)
    ->pingOnFailureIf($condition, $failureUrl);
```

<a name="events"></a>
## Events

Laravel dispatch một loạt các [sự kiện](/docs/{{version}}/events) trong quá trình scheduling. Bạn có thể [định nghĩa listeners](/docs/{{version}}/events) cho bất kỳ sự kiện nào sau đây:

<div class="overflow-auto">

| Event Name                                                  |
| ----------------------------------------------------------- |
| `Illuminate\Console\Events\ScheduledTaskStarting`           |
| `Illuminate\Console\Events\ScheduledTaskFinished`           |
| `Illuminate\Console\Events\ScheduledBackgroundTaskFinished` |
| `Illuminate\Console\Events\ScheduledTaskSkipped`            |
| `Illuminate\Console\Events\ScheduledTaskFailed`             |

</div>

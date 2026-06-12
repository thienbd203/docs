# Logging

- [Introduction](#introduction)
- [Configuration](#configuration)
    - [Available Channel Drivers](#available-channel-drivers)
    - [Channel Prerequisites](#channel-prerequisites)
    - [Logging Deprecation Warnings](#logging-deprecation-warnings)
- [Building Log Stacks](#building-log-stacks)
- [Writing Log Messages](#writing-log-messages)
    - [Contextual Information](#contextual-information)
    - [Writing to Specific Channels](#writing-to-specific-channels)
- [Monolog Channel Customization](#monolog-channel-customization)
    - [Customizing Monolog for Channels](#customizing-monolog-for-channels)
    - [Creating Monolog Handler Channels](#creating-monolog-handler-channels)
    - [Creating Custom Channels via Factories](#creating-custom-channels-via-factories)
- [Tailing Log Messages Using Pail](#tailing-log-messages-using-pail)
    - [Installation](#pail-installation)
    - [Usage](#pail-usage)
    - [Filtering Logs](#pail-filtering-logs)

<a name="introduction"></a>
## Introduction

Để giúp bạn tìm hiểu thêm về những gì đang xảy ra trong ứng dụng của mình, Laravel cung cấp các dịch vụ logging mạnh mẽ cho phép bạn log các thông báo đến các file, system error log, và thậm chí đến Slack để thông báo cho toàn bộ đội ngũ của bạn.

Logging của Laravel dựa trên "channels". Mỗi channel đại diện cho một cách cụ thể để viết thông tin log. Ví dụ, channel `single` viết các file log vào một file log duy nhất, trong khi channel `slack` gửi các thông báo log đến Slack. Các thông báo log có thể được viết đến nhiều channels dựa trên mức độ nghiêm trọng của chúng.

Dưới mui xe, Laravel sử dụng thư viện [Monolog](https://github.com/Seldaek/monolog), cung cấp hỗ trợ cho nhiều log handlers mạnh mẽ. Laravel giúp dễ dàng cấu hình các handlers này, cho phép bạn kết hợp và khớp chúng để tùy chỉnh xử lý log của ứng dụng.

<a name="configuration"></a>
## Configuration

Tất cả các tùy chọn cấu hình kiểm soát hành vi logging của ứng dụng được lưu trữ trong file cấu hình `config/logging.php`. File này cho phép bạn cấu hình các log channels của ứng dụng, vì vậy hãy đảm bảo xem xét từng channel có sẵn và các tùy chọn của chúng. Chúng tôi sẽ xem xét một vài tùy chọn phổ biến dưới đây.

Theo mặc định, Laravel sẽ sử dụng channel `stack` khi log các thông báo. Channel `stack` được sử dụng để tổng hợp nhiều log channels thành một channel duy nhất. Để biết thêm thông tin về việc xây dựng stacks, hãy xem tài liệu [dưới đây](#building-log-stacks).

<a name="available-channel-drivers"></a>
### Available Channel Drivers

Mỗi log channel được cung cấp bởi một "driver". Driver xác định cách và nơi thông báo log thực sự được ghi lại. Các log channel drivers sau có sẵn trong mọi ứng dụng Laravel. Một mục cho hầu hết các drivers này đã có trong file cấu hình `config/logging.php` của ứng dụng, vì vậy hãy đảm bảo xem xét file này để làm quen với nội dung của nó:

<div class="overflow-auto">

| Name         | Description                                                          |
| ------------ | -------------------------------------------------------------------- |
| `custom`     | A driver that calls a specified factory to create a channel.         |
| `daily`      | A `RotatingFileHandler` based Monolog driver which rotates daily.    |
| `errorlog`   | An `ErrorLogHandler` based Monolog driver.                           |
| `monolog`    | A Monolog factory driver that may use any supported Monolog handler. |
| `papertrail` | A `SyslogUdpHandler` based Monolog driver.                           |
| `single`     | A single file or path based logger channel (`StreamHandler`).        |
| `slack`      | A `SlackWebhookHandler` based Monolog driver.                        |
| `stack`      | A wrapper to facilitate creating "multi-channel" channels.           |
| `syslog`     | A `SyslogHandler` based Monolog driver.                              |

</div>

> [!NOTE]
> Hãy xem tài liệu về [tùy chỉnh channel nâng cao](#monolog-channel-customization) để biết thêm thông tin về các drivers `monolog` và `custom`.

<a name="configuring-the-channel-name"></a>
#### Configuring the Channel Name

Theo mặc định, Monolog được khởi tạo với một "channel name" khớp với môi trường hiện tại, chẳng hạn như `production` hoặc `local`. Để thay đổi giá trị này, bạn có thể thêm một tùy chọn `name` vào cấu hình channel của bạn:

```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

<a name="channel-prerequisites"></a>
### Channel Prerequisites

<a name="configuring-the-single-and-daily-channels"></a>
#### Configuring the Single and Daily Channels

Các channel `single` và `daily` có ba tùy chọn cấu hình tùy chọn: `bubble`, `permission`, và `locking`.

<div class="overflow-auto">

| Name         | Description                                                                   | Default |
| ------------ | ----------------------------------------------------------------------------- | ------- |
| `bubble`     | Indicates if messages should bubble up to other channels after being handled. | `true`  |
| `locking`    | Attempt to lock the log file before writing to it.                            | `false` |
| `permission` | The log file's permissions.                                                   | `0644`  |

</div>

Ngoài ra, chính sách giữ lại cho channel `daily` có thể được cấu hình thông qua biến môi trường `LOG_DAILY_DAYS` hoặc bằng cách đặt tùy chọn cấu hình `days`.

<div class="overflow-auto">

| Name   | Description                                                 | Default |
| ------ | ----------------------------------------------------------- | ------- |
| `days` | The number of days that daily log files should be retained. | `14`    |

</div>

<a name="configuring-the-papertrail-channel"></a>
#### Configuring the Papertrail Channel

Channel `papertrail` yêu cầu các tùy chọn cấu hình `host` và `port`. Các tùy chọn này có thể được định nghĩa thông qua các biến môi trường `PAPERTRAIL_URL` và `PAPERTRAIL_PORT`. Bạn có thể lấy các giá trị này từ [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app).

<a name="configuring-the-slack-channel"></a>
#### Configuring the Slack Channel

Channel `slack` yêu cầu một tùy chọn cấu hình `url`. Giá trị này có thể được định nghĩa thông qua biến môi trường `LOG_SLACK_WEBHOOK_URL`. URL này nên khớp với một URL cho một [incoming webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) mà bạn đã cấu hình cho team Slack của bạn.

Theo mặc định, Slack sẽ chỉ nhận các log ở mức `critical` và cao hơn; tuy nhiên, bạn có thể điều chỉnh điều này bằng cách sử dụng biến môi trường `LOG_LEVEL` hoặc bằng cách sửa đổi tùy chọn cấu hình `level` trong array cấu hình log channel Slack của bạn.

<a name="logging-deprecation-warnings"></a>
### Logging Deprecation Warnings

PHP, Laravel, và các thư viện khác thường thông báo cho người dùng của họ rằng một số tính năng của họ đã bị deprecated và sẽ bị xóa trong một phiên bản trong tương lai. Nếu bạn muốn log các cảnh báo deprecated này, bạn có thể chỉ định channel log `deprecations` ưa thích của mình bằng cách sử dụng biến môi trường `LOG_DEPRECATIONS_CHANNEL`, hoặc trong file cấu hình `config/logging.php` của ứng dụng:

```php
'deprecations' => [
    'channel' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),
    'trace' => env('LOG_DEPRECATIONS_TRACE', false),
],

'channels' => [
    // ...
]
```

Hoặc, bạn có thể định nghĩa một log channel có tên `deprecations`. Nếu một log channel với tên này tồn tại, nó sẽ luôn được sử dụng để log các deprecations:

```php
'channels' => [
    'deprecations' => [
        'driver' => 'single',
        'path' => storage_path('logs/php-deprecation-warnings.log'),
    ],
],
```

<a name="building-log-stacks"></a>
## Building Log Stacks

Như đã đề cập trước đó, driver `stack` cho phép bạn kết hợp nhiều channels thành một log channel duy nhất để thuận tiện. Để minh họa cách sử dụng log stacks, hãy xem một cấu hình ví dụ mà bạn có thể thấy trong một ứng dụng production:

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'], // [tl! add]
        'ignore_exceptions' => false,
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => env('LOG_LEVEL', 'debug'),
        'facility' => env('LOG_SYSLOG_FACILITY', LOG_USER),
        'replace_placeholders' => true,
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => env('LOG_SLACK_USERNAME', 'Laravel Log'),
        'emoji' => env('LOG_SLACK_EMOJI', ':boom:'),
        'level' => env('LOG_LEVEL', 'critical'),
        'replace_placeholders' => true,
    ],
],
```

Hãy phân tích cấu hình này. Đầu tiên, hãy lưu ý channel `stack` của chúng tôi tổng hợp hai channels khác thông qua tùy chọn `channels` của nó: `syslog` và `slack`. Vì vậy, khi log các thông báo, cả hai channels này sẽ có cơ hội log thông báo. Tuy nhiên, như chúng ta sẽ thấy dưới đây, liệu các channels này thực sự log thông báo hay không có thể được xác định bởi mức độ nghiêm trọng / "level" của thông báo.

<a name="log-levels"></a>
#### Log Levels

Hãy lưu ý tùy chọn cấu hình `level` có mặt trên cấu hình channel `syslog` và `slack` trong ví dụ trên. Tùy chọn này xác định mức độ "level" tối thiểu mà một thông báo phải có để được log bởi channel. Monolog, cung cấp các dịch vụ logging của Laravel, cung cấp tất cả các log levels được định nghĩa trong [specification RFC 5424](https://tools.ietf.org/html/rfc5424). Theo thứ tự giảm dần của mức độ nghiêm trọng, các log levels này là: **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info**, và **debug**.

Vì vậy, hãy tưởng tượng chúng ta log một thông báo bằng cách sử dụng phương thức `debug`:

```php
Log::debug('An informational message.');
```

Với cấu hình của chúng ta, channel `syslog` sẽ viết thông báo vào system log; tuy nhiên, vì thông báo lỗi không phải là `critical` hoặc cao hơn, nó sẽ không được gửi đến Slack. Tuy nhiên, nếu chúng ta log một thông báo `emergency`, nó sẽ được gửi đến cả system log và Slack vì mức độ `emergency` cao hơn ngưỡng mức độ tối thiểu của cả hai channels:

```php
Log::emergency('The system is down!');
```

<a name="writing-log-messages"></a>
## Writing Log Messages

Bạn có thể viết thông tin vào các log bằng cách sử dụng [facade](/docs/{{version}}/facades) `Log`. Như đã đề cập trước đó, logger cung cấp tám log levels được định nghĩa trong [specification RFC 5424](https://tools.ietf.org/html/rfc5424): **emergency**, **alert**, **critical**, **error**, **warning**, **notice**, **info** và **debug**:

```php
use Illuminate\Support\Facades\Log;

Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

Bạn có thể gọi bất kỳ phương thức nào trong số này để log một thông báo cho level tương ứng. Theo mặc định, thông báo sẽ được viết vào log channel mặc định được cấu hình bởi file cấu hình `logging` của bạn:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Support\Facades\Log;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(string $id): View
    {
        Log::info('Showing the user profile for user: {id}', ['id' => $id]);

        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

<a name="contextual-information"></a>
### Contextual Information

Một array dữ liệu ngữ cảnh có thể được chuyển cho các phương thức log. Dữ liệu ngữ cảnh này sẽ được định dạng và hiển thị với thông báo log:

```php
use Illuminate\Support\Facades\Log;

Log::info('User {id} failed to login.', ['id' => $user->id]);
```

Thỉnh thoảng, bạn có thể muốn chỉ định một số thông tin ngữ cảnh nên được bao gồm với tất cả các mục log tiếp theo trong một channel cụ thể. Ví dụ, bạn có thể muốn log một request ID được liên kết với mỗi request đến ứng dụng của bạn. Để thực hiện điều này, bạn có thể gọi phương thức `withContext` của facade `Log`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();

        Log::withContext([
            'request-id' => $requestId
        ]);

        $response = $next($request);

        $response->headers->set('Request-Id', $requestId);

        return $response;
    }
}
```

Nếu bạn muốn chia sẻ thông tin ngữ cảnh trên _tất cả_ các logging channels, bạn có thể gọi phương thức `Log::shareContext()`. Phương thức này sẽ cung cấp thông tin ngữ cảnh cho tất cả các channels được tạo và bất kỳ channels nào được tạo sau đó:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();

        Log::shareContext([
            'request-id' => $requestId
        ]);

        // ...
    }
}
```

> [!NOTE]
> Nếu bạn cần chia sẻ log context trong khi xử lý các queued jobs, bạn có thể sử dụng [job middleware](/docs/{{version}}/queues#job-middleware).

<a name="writing-to-specific-channels"></a>
### Writing to Specific Channels

Đôi khi bạn có thể muốn log một thông báo đến một channel khác với channel mặc định của ứng dụng. Bạn có thể sử dụng phương thức `channel` trên facade `Log` để truy xuất và log đến bất kỳ channel nào được định nghĩa trong file cấu hình của bạn:

```php
use Illuminate\Support\Facades\Log;

Log::channel('slack')->info('Something happened!');
```

Nếu bạn muốn tạo một logging stack on-demand bao gồm nhiều channels, bạn có thể sử dụng phương thức `stack`:

```php
Log::stack(['single', 'slack'])->info('Something happened!');
```

<a name="on-demand-channels"></a>
#### On-Demand Channels

Cũng có thể tạo một channel on-demand bằng cách cung cấp cấu hình tại runtime mà không cần cấu hình đó có trong file cấu hình `logging` của ứng dụng. Để thực hiện điều này, bạn có thể chuyển một array cấu hình cho phương thức `build` của facade `Log`:

```php
use Illuminate\Support\Facades\Log;

Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
])->info('Something happened!');
```

Bạn cũng có thể muốn bao gồm một channel on-demand trong một logging stack on-demand. Điều này có thể đạt được bằng cách bao gồm instance channel on-demand của bạn trong array được chuyển cho phương thức `stack`:

```php
use Illuminate\Support\Facades\Log;

$channel = Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
]);

Log::stack(['slack', $channel])->info('Something happened!');
```

<a name="monolog-channel-customization"></a>
## Monolog Channel Customization

<a name="customizing-monolog-for-channels"></a>
### Customizing Monolog for Channels

Đôi khi bạn cần kiểm soát hoàn toàn cách Monolog được cấu hình cho một channel hiện có. Ví dụ, bạn có thể muốn cấu hình một triển khai `FormatterInterface` Monolog tùy chỉnh cho channel `single` tích hợp sẵn của Laravel.

Để bắt đầu, định nghĩa một array `tap` trên cấu hình channel. Array `tap` nên chứa một danh sách các lớp nên có cơ hội tùy chỉnh (hoặc "tap" vào) instance Monolog sau khi nó được tạo. Không có vị trí quy ước nơi các lớp này nên được đặt, vì vậy bạn tự do tạo một thư mục trong ứng dụng của mình để chứa các lớp này:

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    'replace_placeholders' => true,
],
```

Sau khi bạn đã cấu hình tùy chọn `tap` trên channel của mình, bạn đã sẵn sàng để định nghĩa lớp sẽ tùy chỉnh instance Monolog của bạn. Lớp này chỉ cần một phương thức duy nhất: `__invoke`, nhận một instance `Illuminate\Log\Logger`. Instance `Illuminate\Log\Logger` proxy tất cả các cuộc gọi phương thức đến instance Monolog bên dưới:

```php
<?php

namespace App\Logging;

use Illuminate\Log\Logger;
use Monolog\Formatter\LineFormatter;

class CustomizeFormatter
{
    /**
     * Customize the given logger instance.
     */
    public function __invoke(Logger $logger): void
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new LineFormatter(
                '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
            ));
        }
    }
}
```

> [!NOTE]
    Tất cả các lớp "tap" của bạn được giải quyết bởi [service container](/docs/{{version}}/container), vì vậy bất kỳ constructor dependencies nào chúng cần sẽ tự động được inject.

<a name="creating-monolog-handler-channels"></a>
### Creating Monolog Handler Channels

Monolog có nhiều [handlers có sẵn](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Handler) và Laravel không bao gồm một channel tích hợp cho từng cái. Trong một số trường hợp, bạn có thể muốn tạo một channel tùy chỉnh chỉ là một instance của một handler Monolog cụ thể không có một driver log Laravel tương ứng. Các channels này có thể được tạo dễ dàng bằng cách sử dụng driver `monolog`.

Khi sử dụng driver `monolog`, tùy chọn cấu hình `handler` được sử dụng để chỉ định handler nào sẽ được khởi tạo. Tùy chọn, bất kỳ tham số constructor nào mà handler cần có thể được chỉ định bằng cách sử dụng tùy chọn cấu hình `handler_with`:

```php
'logentries' => [
    'driver'  => 'monolog',
    'handler' => Monolog\Handler\SyslogUdpHandler::class,
    'handler_with' => [
        'host' => 'my.logentries.internal.datahubhost.company.com',
        'port' => '10000',
    ],
],
```

<a name="monolog-formatters"></a>
#### Monolog Formatters

Khi sử dụng driver `monolog`, `LineFormatter` của Monolog sẽ được sử dụng làm formatter mặc định. Tuy nhiên, bạn có thể tùy chỉnh loại formatter được chuyển cho handler bằng cách sử dụng các tùy chọn cấu hình `formatter` và `formatter_with`:

```php
'browser' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\BrowserConsoleHandler::class,
    'formatter' => Monolog\Formatter\HtmlFormatter::class,
    'formatter_with' => [
        'dateFormat' => 'Y-m-d',
    ],
],
```

Nếu bạn đang sử dụng một Monolog handler có khả năng cung cấp formatter của riêng nó, bạn có thể đặt giá trị của tùy chọn cấu hình `formatter` thành `default`:

```php
'newrelic' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\NewRelicHandler::class,
    'formatter' => 'default',
],
```

<a name="monolog-processors"></a>
#### Monolog Processors

Monolog cũng có thể xử lý các thông báo trước khi log chúng. Bạn có thể tạo các processors của riêng mình hoặc sử dụng [các processors hiện có được cung cấp bởi Monolog](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Processor).

Nếu bạn muốn tùy chỉnh các processors cho một driver `monolog`, hãy thêm một giá trị cấu hình `processors` vào cấu hình channel của bạn:

```php
'memory' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\StreamHandler::class,
    'handler_with' => [
        'stream' => 'php://stderr',
    ],
    'processors' => [
        // Simple syntax...
        Monolog\Processor\MemoryUsageProcessor::class,

        // With options...
        [
            'processor' => Monolog\Processor\MemoryPeakUsageProcessor::class,
            'with' => [
                'real_usage' => true,
                'only_peak' => false,
            ],
        ],
    ],
],
```

<a name="creating-custom-channels-via-factories"></a>
### Creating Custom Channels via Factories

Nếu bạn muốn định nghĩa một channel tùy chỉnh hoàn toàn mà không muốn sử dụng bất kỳ driver hiện có nào, bạn có thể sử dụng driver `custom`. Driver `custom` lấy một factory invocation mà sẽ được gọi để tạo channel:

```php
'channels' => [
    'custom' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomChannel::class,
    ],
],
```

Khi cấu hình channel `custom`, bạn cần định nghĩa một lớp tạo channel. Lớp này chỉ cần một phương thức `__invoke` trả về một instance `Psr\Log\LoggerInterface`:

```php
<?php

namespace App\Logging;

use Monolog\Logger;

class CreateCustomChannel
{
    /**
     * Create a custom Monolog instance.
     */
    public function __invoke(array $config): Logger
    {
        return new Logger(...);
    }
}
```

<a name="tailing-log-messages-using-pail"></a>
## Tailing Log Messages Using Pail

Laravel Pail là một package cho phép bạn tail các log files của ứng dụng của bạn ngay trong terminal của bạn với định dạng đẹp và dễ đọc. Pail hỗ trợ filtering, multiple channels, và cho phép bạn theo dõi các log trong thời gian thực.

<a name="pail-installation"></a>
### Installation

Để bắt đầu, hãy cài đặt Pail vào dự án của bạn bằng Composer:

```shell
composer require laravel/pail --dev
```

<a name="pail-usage"></a>
### Usage

Để bắt đầu tailing các log, hãy chạy lệnh Artisan `pail`:

```shell
php artisan pail
```

Theo mặc định, Pail sẽ tail tất cả các log files được định nghĩa trong cấu hình logging của ứng dụng. Bạn có thể chỉ định một channel cụ thể để tail bằng cách sử dụng tùy chọn `--channel`:

```shell
php artisan pail --channel=slack
```

Để giữ cho các log mới nhất ở đầu terminal, hãy sử dụng tùy chọn `--follow`:

```shell
php artisan pail --follow
```

<a name="pail-filtering-logs"></a>
### Filtering Logs

Pail cung cấp nhiều tùy chọn để lọc các log. Bạn có thể lọc theo level, message, hoặc bất kỳ thuộc tính nào khác trong log entry.

Để lọc theo level, hãy sử dụng tùy chọn `--level`:

```shell
php artisan pail --level=error
```

Để lọc theo message, hãy sử dụng tùy chọn `--message`:

```shell
php artisan pail --message="User login failed"
```

Để lọc theo bất kỳ thuộc tính nào khác, hãy sử dụng tùy chọn `--filter`:

```shell
php artisan pail --filter="context.user_id=1"
```

Bạn cũng có thể kết hợp nhiều bộ lọc:

```shell
php artisan pail --level=error --message="User login failed" --filter="context.user_id=1"
```

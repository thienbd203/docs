# Notifications

- [Introduction](#introduction)
- [Generating Notifications](#generating-notifications)
- [Sending Notifications](#sending-notifications)
  - [Using the Notifiable Trait](#using-the-notifiable-trait)
  - [Using the Notification Facade](#using-the-notification-facade)
  - [Specifying Delivery Channels](#specifying-delivery-channels)
  - [Queueing Notifications](#queueing-notifications)
  - [On-Demand Notifications](#on-demand-notifications)
- [Mail Notifications](#mail-notifications)
  - [Formatting Mail Messages](#formatting-mail-messages)
  - [Customizing the Sender](#customizing-the-sender)
  - [Customizing the Recipient](#customizing-the-recipient)
  - [Customizing the Subject](#customizing-the-subject)
  - [Customizing the Mailer](#customizing-the-mailer)
  - [Customizing the Templates](#customizing-the-templates)
  - [Attachments](#mail-attachments)
  - [Adding Tags and Metadata](#adding-tags-metadata)
  - [Customizing the Symfony Message](#customizing-the-symfony-message)
  - [Using Mailables](#using-mailables)
  - [Previewing Mail Notifications](#previewing-mail-notifications)
- [Markdown Mail Notifications](#markdown-mail-notifications)
  - [Generating the Message](#generating-the-message)
  - [Writing the Message](#writing-the-message)
  - [Customizing the Components](#customizing-the-components)
- [Database Notifications](#database-notifications)
  - [Prerequisites](#database-prerequisites)
  - [Formatting Database Notifications](#formatting-database-notifications)
  - [Accessing the Notifications](#accessing-the-notifications)
  - [Marking Notifications as Read](#marking-notifications-as-read)
- [Broadcast Notifications](#broadcast-notifications)
  - [Prerequisites](#broadcast-prerequisites)
  - [Formatting Broadcast Notifications](#formatting-broadcast-notifications)
  - [Listening for Notifications](#listening-for-notifications)
- [SMS Notifications](#sms-notifications)
  - [Prerequisites](#sms-prerequisites)
  - [Formatting SMS Notifications](#formatting-sms-notifications)
  - [Customizing the "From" Number](#customizing-the-from-number)
  - [Adding a Client Reference](#adding-a-client-reference)
  - [Routing SMS Notifications](#routing-sms-notifications)
- [Slack Notifications](#slack-notifications)
  - [Prerequisites](#slack-prerequisites)
  - [Formatting Slack Notifications](#formatting-slack-notifications)
  - [Slack Interactivity](#slack-interactivity)
  - [Routing Slack Notifications](#routing-slack-notifications)
  - [Notifying External Slack Workspaces](#notifying-external-slack-workspaces)
- [Localizing Notifications](#localizing-notifications)
- [Testing](#testing)
- [Notification Events](#notification-events)
- [Custom Channels](#custom-channels)

<a name="introduction"></a>

## Introduction

Ngoài việc hỗ trợ [gửi email](/docs/{{version}}/mail), Laravel còn hỗ trợ gửi notifications qua nhiều kênh giao tiếp khác nhau, bao gồm email, SMS (thông qua [Vonage](https://www.vonage.com/communications-apis/), trước đây được gọi là Nexmo), và [Slack](https://slack.com). Ngoài ra, nhiều [notification channels do cộng đồng xây dựng](https://laravel-notification-channels.com/about/#suggesting-a-new-channel) đã được tạo để gửi notifications qua hàng chục kênh khác nhau! Notifications cũng có thể được lưu trữ trong database để hiển thị trong giao diện web của bạn.

Thông thường, notifications nên là các thông tin ngắn gọn, thông báo cho người dùng về một sự kiện đã xảy ra trong ứng dụng của bạn. Ví dụ, nếu bạn đang viết một ứng dụng thanh toán, bạn có thể gửi một notification "Invoice Paid" cho người dùng của mình qua các kênh email và SMS.

<a name="generating-notifications"></a>

## Generating Notifications

Trong Laravel, mỗi notification được đại diện bởi một class duy nhất thường được lưu trong thư mục `app/Notifications`. Đừng lo lắng nếu bạn không thấy thư mục này trong ứng dụng của bạn - nó sẽ được tạo ra khi bạn chạy lệnh Artisan `make:notification`:

```shell
php artisan make:notification InvoicePaid
```

Lệnh này sẽ đặt một class notification mới trong thư mục `app/Notifications` của bạn. Mỗi class notification chứa một phương thức `via` và một số lượng biến đổi các phương thức xây dựng message, chẳng hạn như `toMail` hoặc `toDatabase`, chuyển đổi notification thành một message được điều chỉnh cho kênh cụ thể đó.

<a name="sending-notifications"></a>

## Sending Notifications

<a name="using-the-notifiable-trait"></a>

### Using the Notifiable Trait

Notifications có thể được gửi theo hai cách: sử dụng phương thức `notify` của trait `Notifiable` hoặc sử dụng facade `Notification`. Trait `Notifiable` được bao gồm trong model `App\Models\User` của ứng dụng của bạn theo mặc định:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;
}
```

Phương thức `notify` được cung cấp bởi trait này mong đợi nhận một notification instance:

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

> [!NOTE]
> Nhớ rằng, bạn có thể sử dụng trait `Notifiable` trên bất kỳ model nào của bạn. Bạn không bị giới hạn chỉ bao gồm nó trong model `User` của bạn.

<a name="using-the-notification-facade"></a>

### Using the Notification Facade

Ngoài ra, bạn có thể gửi notifications qua facade `Notification`. Cách tiếp cận này hữu ích khi bạn cần gửi một notification đến nhiều notifiable entities như một collection của users. Để gửi notifications sử dụng facade, hãy truyền tất cả các notifiable entities và notification instance vào phương thức `send`:

```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));
```

Bạn cũng có thể gửi notifications ngay lập tức sử dụng phương thức `sendNow`. Phương thức này sẽ gửi notification ngay lập tức ngay cả khi notification implement interface `ShouldQueue`:

```php
Notification::sendNow($developers, new DeploymentCompleted($deployment));
```

<a name="specifying-delivery-channels"></a>

### Specifying Delivery Channels

Mỗi class notification có một phương thức `via` xác định notification sẽ được gửi trên những kênh nào. Notifications có thể được gửi trên các kênh `mail`, `database`, `broadcast`, `vonage`, và `slack`.

> [!NOTE]
> Nếu bạn muốn sử dụng các delivery channels khác như Telegram hoặc Pusher, hãy xem [Laravel Notification Channels website](http://laravel-notification-channels.com) do cộng đồng phát triển.

Phương thức `via` nhận một instance `$notifiable`, sẽ là một instance của class mà notification đang được gửi đến. Bạn có thể sử dụng `$notifiable` để xác định notification nên được gửi trên những kênh nào:

```php
/**
 * Get the notification's delivery channels.
 *
 * @return array<int, string>
 */
public function via(object $notifiable): array
{
    return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
}
```

<a name="queueing-notifications"></a>

### Queueing Notifications

> [!WARNING]
> Trước khi queue notifications, bạn nên cấu hình queue của bạn và [khởi động một worker](/docs/{{version}}/queues#running-the-queue-worker).

Gửi notifications có thể mất thời gian, đặc biệt nếu kênh cần thực hiện một API call bên ngoài để gửi notification. Để tăng tốc thời gian phản hồi của ứng dụng, hãy để notification của bạn được queued bằng cách thêm interface `ShouldQueue` và trait `Queueable` vào class của bạn. Interface và trait đã được import cho tất cả notifications được tạo bằng lệnh `make:notification`, vì vậy bạn có thể thêm chúng ngay vào class notification của bạn:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

Sau khi interface `ShouldQueue` đã được thêm vào notification của bạn, bạn có thể gửi notification như bình thường. Laravel sẽ phát hiện interface `ShouldQueue` trên class và tự động queue việc gửi notification:

```php
$user->notify(new InvoicePaid($invoice));
```

Khi queueing notifications, một queued job sẽ được tạo cho mỗi combination của recipient và channel. Ví dụ, sáu jobs sẽ được dispatch đến queue nếu notification của bạn có ba recipients và hai channels.

<a name="delaying-notifications"></a>

#### Delaying Notifications

Nếu bạn muốn trì hoãn việc gửi notification, bạn có thể chain phương thức `delay` vào notification instantiation của bạn:

```php
$delay = now()->plus(minutes: 10);

$user->notify((new InvoicePaid($invoice))->delay($delay));
```

Bạn có thể truyền một mảng vào phương thức `delay` để chỉ định thời gian trì hoãn cho các channels cụ thể:

```php
$user->notify((new InvoicePaid($invoice))->delay([
    'mail' => now()->plus(minutes: 5),
    'sms' => now()->plus(minutes: 10),
]));
```

Ngoài ra, bạn có thể định nghĩa một phương thức `withDelay` trên chính class notification. Phương thức `withDelay` nên trả về một mảng của channel names và delay values:

```php
/**
 * Determine the notification's delivery delay.
 *
 * @return array<string, \Illuminate\Support\Carbon>
 */
public function withDelay(object $notifiable): array
{
    return [
        'mail' => now()->plus(minutes: 5),
        'sms' => now()->plus(minutes: 10),
    ];
}
```

<a name="customizing-the-notification-queue-connection"></a>

#### Customizing the Notification Queue Connection

Theo mặc định, queued notifications sẽ được queued sử dụng default queue connection của ứng dụng của bạn. Nếu bạn muốn chỉ định một connection khác nên được sử dụng cho một notification cụ thể, bạn có thể gọi phương thức `onConnection` từ constructor của notification:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     */
    public function __construct()
    {
        $this->onConnection('redis');
    }
}
```

Hoặc, nếu bạn muốn chỉ định một queue connection cụ thể nên được sử dụng cho mỗi notification channel được hỗ trợ bởi notification, bạn có thể định nghĩa một phương thức `viaConnections` trên notification của bạn. Phương thức này nên trả về một mảng của các cặp channel name / queue connection name:

```php
/**
 * Determine which connections should be used for each notification channel.
 *
 * @return array<string, string>
 */
public function viaConnections(): array
{
    return [
        'mail' => 'redis',
        'database' => 'sync',
    ];
}
```

<a name="customizing-notification-channel-queues"></a>

#### Customizing Notification Channel Queues

Nếu bạn muốn chỉ định một queue cụ thể nên được sử dụng cho mỗi notification channel được hỗ trợ bởi notification, bạn có thể định nghĩa một phương thức `viaQueues` trên notification của bạn. Phương thức này nên trả về một mảng của các cặp channel name / queue name:

```php
/**
 * Determine which queues should be used for each notification channel.
 *
 * @return array<string, string>
 */
public function viaQueues(): array
{
    return [
        'mail' => 'mail-queue',
        'slack' => 'slack-queue',
    ];
}
```

<a name="customizing-queued-notification-job-properties"></a>

#### Customizing Queued Notification Job Attributes

Bạn có thể tùy chỉnh behavior của queued job bên dưới bằng cách định nghĩa queue attributes trên class notification của bạn. Các attributes này sẽ được kế thừa bởi queued job gửi notification:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Timeout;
use Illuminate\Queue\Attributes\Tries;

#[Tries(5)]
#[Timeout(120)]
#[MaxExceptions(3)]
class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

Nếu bạn muốn đảm bảo privacy và integrity của dữ liệu queued notification qua [encryption](/docs/{{version}}/encryption), hãy thêm interface `ShouldBeEncrypted` vào class notification của bạn:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue, ShouldBeEncrypted
{
    use Queueable;

    // ...
}
```

Ngoài việc định nghĩa các attributes này trực tiếp trên class notification của bạn, bạn cũng có thể định nghĩa các phương thức `backoff` và `retryUntil` để chỉ định backoff strategy và retry timeout cho queued notification job:

```php
use DateTime;

/**
 * Calculate the number of seconds to wait before retrying the notification.
 */
public function backoff(): int
{
    return 3;
}

/**
 * Determine the time at which the notification should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 5);
}
```

> [!NOTE]
> Để biết thêm thông tin về các job attributes và methods này, hãy xem tài liệu về [queued jobs](/docs/{{version}}/queues#max-job-attempts-and-timeout).

<a name="queued-notification-middleware"></a>

#### Queued Notification Middleware

Queued notifications có thể định nghĩa middleware [giống như queued jobs](/docs/{{version}}/queues#job-middleware). Để bắt đầu, định nghĩa một phương thức `middleware` trên class notification của bạn. Phương thức `middleware` sẽ nhận các biến `$notifiable` và `$channel`, cho phép bạn tùy chỉnh middleware được trả về dựa trên destination của notification:

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Get the middleware the notification job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(object $notifiable, string $channel)
{
    return match ($channel) {
        'mail' => [new RateLimited('postmark')],
        'slack' => [new RateLimited('slack')],
        default => [],
    };
}
```

<a name="queued-notifications-and-database-transactions"></a>

#### Queued Notifications and Database Transactions

Khi queued notifications được dispatch trong database transactions, chúng có thể được xử lý bởi queue trước khi database transaction đã được commit. Khi điều này xảy ra, bất kỳ cập nhật nào bạn đã thực hiện cho models hoặc database records trong database transaction có thể chưa được phản ánh trong database. Ngoài ra, bất kỳ models hoặc database records nào được tạo trong transaction có thể không tồn tại trong database. Nếu notification của bạn phụ thuộc vào các models này, các lỗi không mong muốn có thể xảy ra khi job gửi queued notification được xử lý.

Nếu tùy chọn cấu hình `after_commit` của queue connection của bạn được đặt thành `false`, bạn vẫn có thể chỉ định rằng một queued notification cụ thể nên được dispatch sau khi tất cả các database transactions đang mở đã được commit bằng cách gọi phương thức `afterCommit` khi gửi notification:

```php
use App\Notifications\InvoicePaid;

$user->notify((new InvoicePaid($invoice))->afterCommit());
```

Ngoài ra, bạn có thể gọi phương thức `afterCommit` từ constructor của notification:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> [!NOTE]
> Để biết thêm thông tin về cách giải quyết các vấn đề này, hãy xem tài liệu về [queued jobs và database transactions](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="determining-if-the-queued-notification-should-be-sent"></a>

#### Determining if a Queued Notification Should Be Sent

Sau khi một queued notification đã được dispatch cho queue để xử lý nền, nó thường sẽ được chấp nhận bởi một queue worker và được gửi đến người nhận dự định.

Tuy nhiên, nếu bạn muốn đưa ra quyết định cuối cùng về việc queued notification có nên được gửi hay không sau khi nó đang được xử lý bởi một queue worker, bạn có thể định nghĩa một phương thức `shouldSend` trên class notification. Nếu phương thức này trả về `false`, notification sẽ không được gửi:

```php
/**
 * Determine if the notification should be sent.
 */
public function shouldSend(object $notifiable, string $channel): bool
{
    return $this->invoice->isPaid();
}
```

<a name="after-sending-notifications"></a>

#### After Sending Notifications

Nếu bạn muốn thực thi code sau khi một notification đã được gửi, bạn có thể định nghĩa một phương thức `afterSending` trên class notification. Phương thức này sẽ nhận notifiable entity, tên channel, và response từ channel:

```php
/**
 * Handle the notification after it has been sent.
 */
public function afterSending(object $notifiable, string $channel, mixed $response): void
{
    // ...
}
```

<a name="on-demand-notifications"></a>

### On-Demand Notifications

Đôi khi bạn có thể cần gửi một notification đến ai đó không được lưu trữ như một "user" của ứng dụng của bạn. Sử dụng phương thức `route` của facade `Notification`, bạn có thể chỉ định thông tin routing notification ad-hoc trước khi gửi notification:

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Support\Facades\Notification;

Notification::route('mail', 'taylor@example.com')
    ->route('vonage', '5555555555')
    ->route('slack', '#slack-channel')
    ->route('broadcast', [new Channel('channel-name')])
    ->notify(new InvoicePaid($invoice));
```

Nếu bạn muốn cung cấp tên người nhận khi gửi một on-demand notification đến route `mail`, bạn có thể cung cấp một mảng chứa địa chỉ email làm key và tên làm giá trị của phần tử đầu tiên trong mảng:

```php
Notification::route('mail', [
    'barrett@example.com' => 'Barrett Blair',
])->notify(new InvoicePaid($invoice));
```

Sử dụng phương thức `routes`, bạn có thể cung cấp thông tin routing ad-hoc cho nhiều notification channels cùng một lúc:

```php
Notification::routes([
    'mail' => ['barrett@example.com' => 'Barrett Blair'],
    'vonage' => '5555555555',
])->notify(new InvoicePaid($invoice));
```

<a name="mail-notifications"></a>

## Mail Notifications

<a name="formatting-mail-messages"></a>

### Formatting Mail Messages

Nếu một notification hỗ trợ việc được gửi như một email, bạn nên định nghĩa một phương thức `toMail` trên class notification. Phương thức này sẽ nhận một thực thể `$notifiable` và nên trả về một instance `Illuminate\Notifications\Messages\MailMessage`.

Class `MailMessage` chứa một vài phương thức đơn giản giúp bạn xây dựng các email giao dịch. Mail messages có thể chứa các dòng văn bản cũng như một "call to action". Hãy xem một ví dụ về phương thức `toMail`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
        ->greeting('Hello!')
        ->line('One of your invoices has been paid!')
        ->lineIf($this->amount > 0, "Amount paid: {$this->amount}")
        ->action('View Invoice', $url)
        ->line('Thank you for using our application!');
}
```

> [!NOTE]
> Lưu ý chúng ta đang sử dụng `$this->invoice->id` trong phương thức `toMail` của chúng ta. Bạn có thể chuyển bất kỳ dữ liệu nào notification của bạn cần để tạo message của nó vào constructor của notification.

Trong ví dụ này, chúng ta đăng ký một lời chào, một dòng văn bản, một call to action, và sau đó một dòng văn bản khác. Các phương thức được cung cấp bởi object `MailMessage` làm cho việc định dạng các email giao dịch nhỏ trở nên đơn giản và nhanh chóng. Kênh mail sau đó sẽ chuyển đổi các thành phần message thành một template email HTML đẹp mắt, responsive với một bản plain-text tương ứng. Đây là một ví dụ về một email được tạo bởi kênh `mail`:

<img src="https://laravel.com/img/docs/notification-example-2.png">

> [!NOTE]
> Khi gửi mail notifications, hãy chắc chắn đặt tùy chọn cấu hình `name` trong file cấu hình `config/app.php` của bạn. Giá trị này sẽ được sử dụng trong header và footer của các mail notification messages của bạn.

<a name="error-messages"></a>

#### Error Messages

Một số notifications thông báo cho người dùng về các lỗi, chẳng hạn như thanh toán hóa đơn thất bại. Bạn có thể chỉ ra rằng một mail message liên quan đến một lỗi bằng cách gọi phương thức `error` khi xây dựng message của bạn. Khi sử dụng phương thức `error` trên một mail message, nút call to action sẽ có màu đỏ thay vì màu đen:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->error()
        ->subject('Invoice Payment Failed')
        ->line('...');
}
```

<a name="other-mail-notification-formatting-options"></a>

#### Other Mail Notification Formatting Options

Thay vì định nghĩa các "dòng" văn bản trong class notification, bạn có thể sử dụng phương thức `view` để chỉ định một custom template nên được sử dụng để render email notification:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->view(
        'mail.invoice.paid', ['invoice' => $this->invoice]
    );
}
```

Bạn có thể chỉ định một plain-text view cho mail message bằng cách truyền tên view làm phần tử thứ hai của một mảng được đưa vào phương thức `view`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->view(
        ['mail.invoice.paid', 'mail.invoice.paid-text'],
        ['invoice' => $this->invoice]
    );
}
```

Hoặc, nếu message của bạn chỉ có một plain-text view, bạn có thể sử dụng phương thức `text`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->text(
        'mail.invoice.paid-text', ['invoice' => $this->invoice]
    );
}
```

<a name="customizing-the-sender"></a>

### Customizing the Sender

Theo mặc định, địa chỉ người gửi / from của email được định nghĩa trong file cấu hình `config/mail.php`. Tuy nhiên, bạn có thể chỉ định địa chỉ from cho một notification cụ thể sử dụng phương thức `from`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->from('barrett@example.com', 'Barrett Blair')
        ->line('...');
}
```

<a name="customizing-the-recipient"></a>

### Customizing the Recipient

Khi gửi notifications qua kênh `mail`, hệ thống notification sẽ tự động tìm kiếm một thuộc tính `email` trên notifiable entity của bạn. Bạn có thể tùy chỉnh địa chỉ email nào được sử dụng để gửi notification bằng cách định nghĩa một phương thức `routeNotificationForMail` trên notifiable entity:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the mail channel.
     *
     * @return  array<string, string>|string
     */
    public function routeNotificationForMail(Notification $notification): array|string
    {
        // Return email address only...
        return $this->email_address;

        // Return email address and name...
        return [$this->email_address => $this->name];
    }
}
```

<a name="customizing-the-subject"></a>

### Customizing the Subject

Theo mặc định, subject của email là tên class của notification được định dạng thành "Title Case". Vì vậy, nếu class notification của bạn được đặt tên là `InvoicePaid`, subject của email sẽ là `Invoice Paid`. Nếu bạn muốn chỉ định một subject khác cho message, bạn có thể gọi phương thức `subject` khi xây dựng message:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Notification Subject')
        ->line('...');
}
```

<a name="customizing-the-mailer"></a>

### Customizing the Mailer

Theo mặc định, email notification sẽ được gửi sử dụng default mailer được định nghĩa trong file cấu hình `config/mail.php`. Tuy nhiên, bạn có thể chỉ định một mailer khác tại runtime bằng cách gọi phương thức `mailer` khi xây dựng message:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->mailer('postmark')
        ->line('...');
}
```

<a name="customizing-the-templates"></a>

### Customizing the Templates

Bạn có thể sửa đổi HTML và plain-text template được sử dụng bởi mail notifications bằng cách publish resources của notification package. Sau khi chạy lệnh này, mail notification templates sẽ nằm trong thư mục `resources/views/vendor/notifications`:

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>

### Attachments

Để thêm attachments vào một email notification, sử dụng phương thức `attach` khi xây dựng message của bạn. Phương thức `attach` chấp nhận đường dẫn tuyệt đối đến file làm đối số đầu tiên của nó:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attach('/path/to/file');
}
```

> [!NOTE]
> The `attach` method offered by notification mail messages also accepts [attachable objects](/docs/{{version}}/mail#attachable-objects). Please consult the comprehensive [attachable object documentation](/docs/{{version}}/mail#attachable-objects) to learn more.

Khi đính kèm files vào một message, bạn cũng có thể chỉ định tên hiển thị và / hoặc MIME type bằng cách truyền một `array` làm đối số thứ hai cho phương thức `attach`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attach('/path/to/file', [
            'as' => 'name.pdf',
            'mime' => 'application/pdf',
        ]);
}
```

Khi cần thiết, nhiều files có thể được đính kèm vào một message sử dụng phương thức `attachMany`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attachMany([
            '/path/to/forge.svg',
            '/path/to/vapor.svg' => [
                'as' => 'Logo.svg',
                'mime' => 'image/svg+xml',
            ],
        ]);
}
```

Bạn có thể sử dụng phương thức `attachFromStorageDisk` để đính kèm một file tồn tại trên một [filesystem disk](/docs/{{version}}/filesystem) cụ thể. Phương thức này chấp nhận tên disk và đường dẫn đến file trên disk đó:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
        ->to($notifiable->email)
        ->attachFromStorageDisk('s3', '/path/to/file', 'invoice.pdf', [
            'mime' => 'application/pdf',
        ]);
}
```

<a name="raw-data-attachments"></a>

#### Raw Data Attachments

Phương thức `attachData` có thể được sử dụng để đính kèm một chuỗi byte thô như một attachment. Khi gọi phương thức `attachData`, bạn nên cung cấp tên file nên được gán cho attachment:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attachData($this->pdf, 'name.pdf', [
            'mime' => 'application/pdf',
        ]);
}
```

<a name="adding-tags-metadata"></a>

### Adding Tags and Metadata

Một số nhà cung cấp email bên thứ ba như Mailgun và Postmark hỗ trợ "tags" và "metadata" của message, có thể được sử dụng để nhóm và theo dõi các email được gửi bởi ứng dụng của bạn. Bạn có thể thêm tags và metadata vào một email message qua các phương thức `tag` và `metadata`:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Comment Upvoted!')
        ->tag('upvote')
        ->metadata('comment_id', $this->comment->id);
}
```

Nếu ứng dụng của bạn đang sử dụng driver Mailgun, bạn có thể tham khảo tài liệu Mailgun để biết thêm thông tin về [tags](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#tags) và [metadata](https://documentation.mailgun.com/docs/mailgun/user-manual/sending-messages/#attaching-metadata-to-messages). Tương tự, tài liệu Postmark cũng có thể được tham khảo để biết thêm thông tin về hỗ trợ của họ cho [tags](https://postmarkapp.com/blog/tags-support-for-smtp) và [metadata](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Nếu ứng dụng của bạn đang sử dụng Amazon SES để gửi emails, bạn nên sử dụng phương thức `metadata` để đính kèm [SES "tags"](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) vào message.

<a name="customizing-the-symfony-message"></a>

### Customizing the Symfony Message

Phương thức `withSymfonyMessage` của class `MailMessage` cho phép bạn đăng ký một closure sẽ được gọi với instance Symfony Message trước khi gửi message. Điều này cho phép bạn tùy chỉnh sâu message trước khi nó được gửi:

```php
use Symfony\Component\Mime\Email;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->withSymfonyMessage(function (Email $message) {
            $message->getHeaders()->addTextHeader(
                'Custom-Header', 'Header Value'
            );
        });
}
```

<a name="using-mailables"></a>

### Using Mailables

Nếu cần thiết, bạn có thể trả về một [mailable object](/docs/{{version}}/mail) đầy đủ từ phương thức `toMail` của notification. Khi trả về một `Mailable` thay vì một `MailMessage`, bạn sẽ cần chỉ định người nhận message sử dụng phương thức `to` của mailable object:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;
use Illuminate\Mail\Mailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
        ->to($notifiable->email);
}
```

<a name="mailables-and-on-demand-notifications"></a>

#### Mailables and On-Demand Notifications

Nếu bạn đang gửi một [on-demand notification](#on-demand-notifications), instance `$notifiable` được đưa vào phương thức `toMail` sẽ là một instance của `Illuminate\Notifications\AnonymousNotifiable`, cung cấp một phương thức `routeNotificationFor` có thể được sử dụng để lấy địa chỉ email mà on-demand notification nên được gửi đến:

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;
use Illuminate\Notifications\AnonymousNotifiable;
use Illuminate\Mail\Mailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    $address = $notifiable instanceof AnonymousNotifiable
        ? $notifiable->routeNotificationFor('mail')
        : $notifiable->email;

    return (new InvoicePaidMailable($this->invoice))
        ->to($address);
}
```

<a name="previewing-mail-notifications"></a>

### Previewing Mail Notifications

Khi thiết kế một mail notification template, việc nhanh chóng preview mail message được render trong trình duyệt như một Blade template điển hình là rất tiện lợi. Vì lý do này, Laravel cho phép bạn trả về bất kỳ mail message nào được tạo bởi một mail notification trực tiếp từ một route closure hoặc controller. Khi một `MailMessage` được trả về, nó sẽ được render và hiển thị trong trình duyệt, cho phép bạn nhanh chóng preview thiết kế của nó mà không cần gửi đến một địa chỉ email thực tế:

```php
use App\Models\Invoice;
use App\Notifications\InvoicePaid;

Route::get('/notification', function () {
    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))
        ->toMail($invoice->user);
});
```

<a name="markdown-mail-notifications"></a>

## Markdown Mail Notifications

Markdown mail notifications cho phép bạn tận dụng các pre-built templates của mail notifications, trong khi cho bạn nhiều tự do hơn để viết các messages dài hơn, được tùy chỉnh. Vì messages được viết bằng Markdown, Laravel có thể render các HTML templates đẹp mắt, responsive cho messages trong khi cũng tự động tạo ra một bản plain-text tương ứng.

<a name="generating-the-message"></a>

### Generating the Message

Để tạo một notification với Markdown template tương ứng, bạn có thể sử dụng tùy chọn `--markdown` của lệnh Artisan `make:notification`:

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

Giống như tất cả mail notifications khác, notifications sử dụng Markdown templates nên định nghĩa một phương thức `toMail` trên class notification của chúng. Tuy nhiên, thay vì sử dụng các phương thức `line` và `action` để xây dựng notification, hãy sử dụng phương thức `markdown` để chỉ định tên của Markdown template nên được sử dụng. Một mảng dữ liệu bạn muốn cung cấp cho template có thể được truyền làm đối số thứ hai của phương thức:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
        ->subject('Invoice Paid')
        ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="writing-the-message"></a>

### Writing the Message

Markdown mail notifications sử dụng kết hợp của Blade components và Markdown syntax cho phép bạn dễ dàng xây dựng notifications trong khi tận dụng các notification components được tạo sẵn của Laravel:

```blade
<x-mail::message>
# Invoice Paid

Your invoice has been paid!

<x-mail::button :url="$url">
View Invoice
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> [!NOTE]
> Do not use excess indentation when writing Markdown emails. Per Markdown standards, Markdown parsers will render indented content as code blocks.

<a name="button-component"></a>

#### Button Component

Button component render một button link được căn giữa. Component chấp nhận hai đối số, một `url` và một `color` tùy chọn. Các màu được hỗ trợ là `primary`, `green`, và `red`. Bạn có thể thêm bao nhiêu button components vào một notification tùy ý:

```blade
<x-mail::button :url="$url" color="green">
View Invoice
</x-mail::button>
```

<a name="panel-component"></a>

#### Panel Component

Panel component render khối văn bản đã cho trong một panel có màu nền hơi khác so với phần còn lại của notification. Điều này cho phép bạn thu hút sự chú ý đến một khối văn bản cụ thể:

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>

#### Table Component

Table component cho phép bạn chuyển đổi một Markdown table thành một HTML table. Component chấp nhận Markdown table làm nội dung của nó. Việc căn chỉnh cột bảng được hỗ trợ sử dụng cú pháp căn chỉnh bảng Markdown mặc định:

```blade
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

<a name="customizing-the-components"></a>

### Customizing the Components

Bạn có thể export tất cả các Markdown notification components vào ứng dụng của bạn để tùy chỉnh. Để export các components, sử dụng lệnh Artisan `vendor:publish` để publish asset tag `laravel-mail`:

```shell
php artisan vendor:publish --tag=laravel-mail
```

Lệnh này sẽ publish các Markdown mail components vào thư mục `resources/views/vendor/mail`. Thư mục `mail` sẽ chứa một thư mục `html` và một thư mục `text`, mỗi thư mục chứa các đại diện tương ứng của mọi component có sẵn. Bạn có thể tùy chỉnh các components này tùy ý.

<a name="customizing-the-css"></a>

#### Customizing the CSS

Sau khi export các components, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa một file `default.css`. Bạn có thể tùy chỉnh CSS trong file này và styles của bạn sẽ tự động được in-line trong các HTML representations của Markdown notifications của bạn.

Nếu bạn muốn xây dựng một theme hoàn toàn mới cho Markdown components của Laravel, bạn có thể đặt một file CSS trong thư mục `html/themes`. Sau khi đặt tên và lưu file CSS của bạn, hãy cập nhật tùy chọn `theme` của file cấu hình `mail` để khớp với tên theme mới của bạn.

Để tùy chỉnh theme cho một notification cụ thể, bạn có thể gọi phương thức `theme` khi xây dựng mail message của notification. Phương thức `theme` chấp nhận tên của theme nên được sử dụng khi gửi notification:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->theme('invoice')
        ->subject('Invoice Paid')
        ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="database-notifications"></a>

## Database Notifications

<a name="database-prerequisites"></a>

### Prerequisites

Kênh notification `database` lưu trữ thông tin notification trong một bảng database. Bảng này sẽ chứa thông tin như loại notification cũng như một cấu trúc dữ liệu JSON mô tả notification.

Bạn có thể query bảng để hiển thị notifications trong giao diện người dùng của ứng dụng. Nhưng, trước khi bạn có thể làm điều đó, bạn sẽ cần tạo một bảng database để giữ notifications của bạn. Bạn có thể sử dụng lệnh `make:notifications-table` để tạo một [migration](/docs/{{version}}/migrations) với schema bảng thích hợp:

```shell
php artisan make:notifications-table

php artisan migrate
```

> [!NOTE]
> Nếu notifiable models của bạn đang sử dụng [UUID hoặc ULID primary keys](/docs/{{version}}/eloquent#uuid-and-ulid-keys), bạn nên thay thế phương thức `morphs` bằng [uuidMorphs](/docs/{{version}}/migrations#column-method-uuidMorphs) hoặc [ulidMorphs](/docs/{{version}}/migrations#column-method-ulidMorphs) trong notification table migration.

<a name="formatting-database-notifications"></a>

### Formatting Database Notifications

Nếu một notification hỗ trợ việc được lưu trữ trong một bảng database, bạn nên định nghĩa một phương thức `toDatabase` hoặc `toArray` trên class notification. Phương thức này sẽ nhận một thực thể `$notifiable` và nên trả về một mảng PHP thuần túy. Mảng được trả về sẽ được mã hóa thành JSON và lưu trữ trong cột `data` của bảng `notifications` của bạn. Hãy xem một ví dụ về phương thức `toArray`:

```php
/**
 * Get the array representation of the notification.
 *
 * @return array<string, mixed>
 */
public function toArray(object $notifiable): array
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

Khi một notification được lưu trữ trong database của ứng dụng của bạn, cột `type` sẽ được đặt thành tên class của notification theo mặc định, và cột `read_at` sẽ là `null`. Tuy nhiên, bạn có thể tùy chỉnh behavior này bằng cách định nghĩa các phương thức `databaseType` và `initialDatabaseReadAtValue` trong class notification của bạn:

```php
use Illuminate\Support\Carbon;

/**
 * Get the notification's database type.
 */
public function databaseType(object $notifiable): string
{
    return 'invoice-paid';
}

/**
 * Get the initial value for the "read_at" column.
 */
public function initialDatabaseReadAtValue(): ?Carbon
{
    return null;
}
```

<a name="todatabase-vs-toarray"></a>

#### `toDatabase` vs. `toArray`

Phương thức `toArray` cũng được sử dụng bởi kênh `broadcast` để xác định dữ liệu nào nên được broadcast đến frontend được hỗ trợ bởi JavaScript của bạn. Nếu bạn muốn có hai array representations khác nhau cho các kênh `database` và `broadcast`, bạn nên định nghĩa một phương thức `toDatabase` thay vì phương thức `toArray`.

<a name="accessing-the-notifications"></a>

### Accessing the Notifications

Sau khi notifications được lưu trữ trong database, bạn cần một cách thuận tiện để truy cập chúng từ các notifiable entities của bạn. Trait `Illuminate\Notifications\Notifiable`, được bao gồm trong model `App\Models\User` mặc định của Laravel, bao gồm một relationship `notifications` [Eloquent](/docs/{{version}}/eloquent-relationships) trả về các notifications cho entity. Để fetch notifications, bạn có thể truy cập phương thức này giống như bất kỳ Eloquent relationship nào khác. Theo mặc định, notifications sẽ được sắp xếp theo timestamp `created_at` với các notifications gần nhất ở đầu collection:

```php
$user = App\Models\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

Nếu bạn muốn chỉ retrieve các notifications "unread", bạn có thể sử dụng relationship `unreadNotifications`. Một lần nữa, các notifications này sẽ được sắp xếp theo timestamp `created_at` với các notifications gần nhất ở đầu collection:

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

Nếu bạn muốn chỉ retrieve các notifications "read", bạn có thể sử dụng relationship `readNotifications`:

```php
$user = App\Models\User::find(1);

foreach ($user->readNotifications as $notification) {
    echo $notification->type;
}
```

> [!NOTE]
> Để truy cập notifications của bạn từ JavaScript client, bạn nên định nghĩa một notification controller cho ứng dụng của bạn trả về các notifications cho một notifiable entity, chẳng hạn như current user. Sau đó bạn có thể thực hiện một HTTP request đến URL của controller đó từ JavaScript client của bạn.

<a name="marking-notifications-as-read"></a>

### Marking Notifications as Read

Thông thường, bạn sẽ muốn đánh dấu một notification là "read" khi người dùng xem nó. Trait `Illuminate\Notifications\Notifiable` cung cấp một phương thức `markAsRead`, cập nhật cột `read_at` trên database record của notification:

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

Tuy nhiên, thay vì loop qua từng notification, bạn có thể sử dụng phương thức `markAsRead` trực tiếp trên một collection của notifications:

```php
$user->unreadNotifications->markAsRead();
```

Bạn cũng có thể sử dụng một mass-update query để đánh dấu tất cả các notifications là đã đọc mà không cần lấy chúng từ database:

```php
$user = App\Models\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

Bạn có thể `delete` các notifications để xóa chúng khỏi bảng hoàn toàn:

```php
$user->notifications()->delete();
```

<a name="broadcast-notifications"></a>

## Broadcast Notifications

<a name="broadcast-prerequisites"></a>

### Prerequisites

Trước khi broadcasting notifications, bạn nên cấu hình và làm quen với các dịch vụ [event broadcasting](/docs/{{version}}/broadcasting) của Laravel. Event broadcasting cung cấp một cách để phản ứng với các sự kiện Laravel phía server từ frontend được hỗ trợ bởi JavaScript của bạn.

<a name="formatting-broadcast-notifications"></a>

### Formatting Broadcast Notifications

Kênh `broadcast` broadcasts notifications sử dụng các dịch vụ [event broadcasting](/docs/{{version}}/broadcasting) của Laravel, cho phép frontend được hỗ trợ bởi JavaScript của bạn catch notifications trong realtime. Nếu một notification hỗ trợ broadcasting, bạn có thể định nghĩa một phương thức `toBroadcast` trên class notification. Phương thức này sẽ nhận một thực thể `$notifiable` và nên trả về một instance `BroadcastMessage`. Nếu phương thức `toBroadcast` không tồn tại, phương thức `toArray` sẽ được sử dụng để thu thập dữ liệu nên được broadcast. Dữ liệu được trả về sẽ được mã hóa thành JSON và broadcast đến frontend được hỗ trợ bởi JavaScript của bạn. Hãy xem một ví dụ về phương thức `toBroadcast`:

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the broadcastable representation of the notification.
 */
public function toBroadcast(object $notifiable): BroadcastMessage
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

<a name="broadcast-queue-configuration"></a>

#### Broadcast Queue Configuration

Tất cả broadcast notifications đều được queued để broadcasting. Nếu bạn muốn cấu hình queue connection hoặc queue name được sử dụng để queue broadcast operation, bạn có thể sử dụng các phương thức `onConnection` và `onQueue` của `BroadcastMessage`:

```php
return (new BroadcastMessage($data))
    ->onConnection('sqs')
    ->onQueue('broadcasts');
```

<a name="customizing-the-notification-type"></a>

#### Customizing the Notification Type

Ngoài dữ liệu bạn chỉ định, tất cả broadcast notifications cũng có một trường `type` chứa tên class đầy đủ của notification. Nếu bạn muốn tùy chỉnh notification `type`, bạn có thể định nghĩa một phương thức `broadcastType` trên class notification:

```php
/**
 * Get the type of the notification being broadcast.
 */
public function broadcastType(): string
{
    return 'broadcast.message';
}
```

<a name="listening-for-notifications"></a>

### Listening for Notifications

Notifications sẽ broadcast trên một private channel được định dạng sử dụng convention `{notifiable}.{id}`. Vì vậy, nếu bạn đang gửi một notification đến một instance `App\Models\User` với ID là `1`, notification sẽ được broadcast trên private channel `App.Models.User.1`. Khi sử dụng [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation), bạn có thể dễ dàng listen cho notifications trên một channel sử dụng phương thức `notification`:

```js
Echo.private("App.Models.User." + userId).notification((notification) => {
  console.log(notification.type);
});
```

<a name="using-react-or-vue"></a>

#### Using React, Vue, or Svelte

Laravel Echo bao gồm các hooks React, Vue, và Svelte giúp việc listen cho notifications trở nên dễ dàng. Để bắt đầu, hãy gọi hook `useEchoNotification`, được sử dụng để listen cho notifications. Hook `useEchoNotification` sẽ tự động leave channels khi component tiêu thụ được unmounted:

```js tab=React
import { useEchoNotification } from "@laravel/echo-react";

useEchoNotification(`App.Models.User.${userId}`, (notification) => {
  console.log(notification.type);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoNotification } from "@laravel/echo-vue";

useEchoNotification(`App.Models.User.${userId}`, (notification) => {
  console.log(notification.type);
});
</script>
```

```svelte tab=Svelte
<script>
import { useEchoNotification } from "@laravel/echo-svelte";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
);
</script>
```

Theo mặc định, hook listen tất cả notifications. Để chỉ định các notification types bạn muốn listen, bạn có thể cung cấp một string hoặc mảng của types cho `useEchoNotification`:

```js tab=React
import { useEchoNotification } from "@laravel/echo-react";

useEchoNotification(
  `App.Models.User.${userId}`,
  (notification) => {
    console.log(notification.type);
  },
  "App.Notifications.InvoicePaid",
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoNotification } from "@laravel/echo-vue";

useEchoNotification(
  `App.Models.User.${userId}`,
  (notification) => {
    console.log(notification.type);
  },
  "App.Notifications.InvoicePaid",
);
</script>
```

```svelte tab=Svelte
<script>
import { useEchoNotification } from "@laravel/echo-svelte";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
    'App.Notifications.InvoicePaid',
);
</script>
```

Bạn cũng có thể chỉ định shape của dữ liệu notification payload, cung cấp type safety và editing convenience tốt hơn:

```ts
type InvoicePaidNotification = {
  invoice_id: number;
  created_at: string;
};

useEchoNotification<InvoicePaidNotification>(
  `App.Models.User.${userId}`,
  (notification) => {
    console.log(notification.invoice_id);
    console.log(notification.created_at);
    console.log(notification.type);
  },
  "App.Notifications.InvoicePaid",
);
```

<a name="customizing-the-notification-channel"></a>

#### Customizing the Notification Channel

Nếu bạn muốn tùy chỉnh channel mà broadcast notifications của một entity được broadcast trên, bạn có thể định nghĩa một phương thức `receivesBroadcastNotificationsOn` trên notifiable entity:

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The channels the user receives notification broadcasts on.
     */
    public function receivesBroadcastNotificationsOn(): string
    {
        return 'users.'.$this->id;
    }
}
```

<a name="sms-notifications"></a>

## SMS Notifications

<a name="sms-prerequisites"></a>

### Prerequisites

Việc gửi SMS notifications trong Laravel được hỗ trợ bởi [Vonage](https://www.vonage.com/) (trước đây được gọi là Nexmo). Trước khi bạn có thể gửi notifications qua Vonage, bạn cần cài đặt các packages `laravel/vonage-notification-channel` và `guzzlehttp/guzzle`:

```shell
composer require laravel/vonage-notification-channel guzzlehttp/guzzle
```

Package bao gồm một [file cấu hình](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php). Tuy nhiên, bạn không bắt buộc phải export file cấu hình này vào ứng dụng của bạn. Bạn có thể đơn giản sử dụng các environment variables `VONAGE_KEY` và `VONAGE_SECRET` để định nghĩa các Vonage public và secret keys của bạn.

Sau khi định nghĩa các keys của bạn, bạn nên đặt một environment variable `VONAGE_SMS_FROM` định nghĩa số điện thoại mà SMS messages của bạn nên được gửi từ theo mặc định. Bạn có thể tạo số điện thoại này trong Vonage control panel:

```ini
VONAGE_SMS_FROM=15556666666
```

<a name="formatting-sms-notifications"></a>

### Formatting SMS Notifications

Nếu một notification hỗ trợ việc được gửi như một SMS, bạn nên định nghĩa một phương thức `toVonage` trên class notification. Phương thức này sẽ nhận một thực thể `$notifiable` và nên trả về một instance `Illuminate\Notifications\Messages\VonageMessage`:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your SMS message content');
}
```

<a name="unicode-content"></a>

#### Unicode Content

Nếu SMS message của bạn sẽ chứa các ký tự unicode, bạn nên gọi phương thức `unicode` khi xây dựng instance `VonageMessage`:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your unicode message')
        ->unicode();
}
```

<a name="customizing-the-from-number"></a>

### Customizing the "From" Number

Nếu bạn muốn gửi một số notifications từ một số điện thoại khác với số điện thoại được chỉ định bởi environment variable `VONAGE_SMS_FROM` của bạn, bạn có thể gọi phương thức `from` trên một instance `VonageMessage`:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your SMS message content')
        ->from('15554443333');
}
```

<a name="adding-a-client-reference"></a>

### Adding a Client Reference

Nếu bạn muốn theo dõi chi phí cho mỗi user, team, hoặc client, bạn có thể thêm một "client reference" vào notification. Vonage sẽ cho phép bạn tạo reports sử dụng client reference này để bạn có thể hiểu rõ hơn về việc sử dụng SMS của một khách hàng cụ thể. Client reference có thể là bất kỳ string nào lên đến 40 ký tự:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->clientReference((string) $notifiable->id)
        ->content('Your SMS message content');
}
```

<a name="routing-sms-notifications"></a>

### Routing SMS Notifications

Để route Vonage notifications đến số điện thoại thích hợp, hãy định nghĩa một phương thức `routeNotificationForVonage` trên notifiable entity của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Vonage channel.
     */
    public function routeNotificationForVonage(Notification $notification): string
    {
        return $this->phone_number;
    }
}
```

<a name="slack-notifications"></a>

## Slack Notifications

<a name="slack-prerequisites"></a>

### Prerequisites

Trước khi gửi Slack notifications, bạn nên cài đặt Slack notification channel qua Composer:

```shell
composer require laravel/slack-notification-channel
```

Ngoài ra, bạn phải tạo một [Slack App](https://api.slack.com/apps?new_app=1) cho Slack workspace của bạn.

Nếu bạn chỉ cần gửi notifications đến cùng Slack workspace mà App được tạo trong đó, bạn nên đảm bảo rằng App của bạn có các scopes `chat:write`, `chat:write.public`, và `chat:write.customize`. Các scopes này có thể được thêm từ tab quản lý App "OAuth & Permissions" trong Slack.

Tiếp theo, sao chép "Bot User OAuth Token" của App và đặt nó trong một mảng cấu hình `slack` trong file cấu hình `services.php` của ứng dụng của bạn. Token này có thể được tìm thấy trong tab "OAuth & Permissions" trong Slack:

```php
'slack' => [
    'notifications' => [
        'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
        'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
    ],
],
```

<a name="slack-app-distribution"></a>

#### App Distribution

Nếu ứng dụng của bạn sẽ gửi notifications đến các Slack workspaces bên ngoài thuộc sở hữu của users của ứng dụng của bạn, bạn sẽ cần "distribute" App của bạn qua Slack. App distribution có thể được quản lý từ tab "Manage Distribution" của App trong Slack. Sau khi App của bạn đã được distributed, bạn có thể sử dụng [Socialite](/docs/{{version}}/socialite) để [obtain Slack Bot tokens](/docs/{{version}}/socialite#slack-bot-scopes) thay mặt cho users của ứng dụng của bạn.

<a name="formatting-slack-notifications"></a>

### Formatting Slack Notifications

Nếu một notification hỗ trợ việc được gửi như một Slack message, bạn nên định nghĩa một phương thức `toSlack` trên class notification. Phương thức này sẽ nhận một thực thể `$notifiable` và nên trả về một instance `Illuminate\Notifications\Slack\SlackMessage`. Bạn có thể xây dựng các notifications phong phú sử dụng [Slack's Block Kit API](https://api.slack.com/block-kit). Ví dụ sau có thể được preview trong [Slack's Block Kit builder](https://app.slack.com/block-kit-builder/T01KWS6K23Z#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Invoice%20Paid%22%7D%7D,%7B%22type%22:%22context%22,%22elements%22:%5B%7B%22type%22:%22plain_text%22,%22text%22:%22Customer%20%231234%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22An%20invoice%20has%20been%20paid.%22%7D,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20No:*%5Cn1000%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20Recipient:*%5Cntaylor@laravel.com%22%7D%5D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Congratulations!%22%7D%7D%5D%7D):

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
            $block->field("*Invoice No:*\n1000")->markdown();
            $block->field("*Invoice Recipient:*\ntaylor@laravel.com")->markdown();
        })
        ->dividerBlock()
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('Congratulations!');
        });
}
```

<a name="using-slacks-block-kit-builder-template"></a>

#### Using Slack's Block Kit Builder Template

Thay vì sử dụng các fluent message builder methods để xây dựng Block Kit message của bạn, bạn có thể cung cấp raw JSON payload được tạo bởi Slack's Block Kit Builder cho phương thức `usingBlockKitTemplate`:

```php
use Illuminate\Notifications\Slack\SlackMessage;
use Illuminate\Support\Str;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    $template = <<<JSON
        {
          "blocks": [
            {
              "type": "header",
              "text": {
                "type": "plain_text",
                "text": "Team Announcement"
              }
            },
            {
              "type": "section",
              "text": {
                "type": "plain_text",
                "text": "We are hiring!"
              }
            }
          ]
        }
    JSON;

    return (new SlackMessage)
        ->usingBlockKitTemplate($template);
}
```

<a name="slack-interactivity"></a>

### Slack Interactivity

Slack's Block Kit notification system cung cấp các tính năng mạnh mẽ để [handle user interaction](https://api.slack.com/interactivity/handling). Để sử dụng các tính năng này, Slack App của bạn nên có "Interactivity" được bật và một "Request URL" được cấu hình trỏ đến một URL được phục vụ bởi ứng dụng của bạn. Các cài đặt này có thể được quản lý từ tab quản lý App "Interactivity & Shortcuts" trong Slack.

Trong ví dụ sau, sử dụng phương thức `actionsBlock`, Slack sẽ gửi một request `POST` đến "Request URL" của bạn với một payload chứa Slack user đã click vào button, ID của button được click, và nhiều hơn nữa. Ứng dụng của bạn sau đó có thể xác định action cần thực hiện dựa trên payload. Bạn cũng nên [verify the request](https://api.slack.com/authentication/verifying-requests-from-slack) được thực hiện bởi Slack:

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
        })
        ->actionsBlock(function (ActionsBlock $block) {
             // ID defaults to "button_acknowledge_invoice"...
            $block->button('Acknowledge Invoice')->primary();

            // Manually configure the ID...
            $block->button('Deny')->danger()->id('deny_invoice');
        });
}
```

<a name="slack-confirmation-modals"></a>

#### Confirmation Modals

Nếu bạn muốn users được yêu cầu confirm một action trước khi nó được thực hiện, bạn có thể gọi phương thức `confirm` khi định nghĩa button của bạn. Phương thức `confirm` chấp nhận một message và một closure nhận một instance `ConfirmObject`:

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
        })
        ->actionsBlock(function (ActionsBlock $block) {
            $block->button('Acknowledge Invoice')
                ->primary()
                ->confirm(
                    'Acknowledge the payment and send a thank you email?',
                    function (ConfirmObject $dialog) {
                        $dialog->confirm('Yes');
                        $dialog->deny('No');
                    }
                );
        });
}
```

<a name="inspecting-slack-blocks"></a>

#### Inspecting Slack Blocks

Nếu bạn muốn nhanh chóng inspect các blocks bạn đã xây dựng, bạn có thể gọi phương thức `dd` trên instance `SlackMessage`. Phương thức `dd` sẽ tạo và dump một URL đến [Block Kit Builder](https://app.slack.com/block-kit-builder/) của Slack, hiển thị một preview của payload và notification trong trình duyệt của bạn. Bạn có thể truyền `true` cho phương thức `dd` để dump raw payload:

```php
return (new SlackMessage)
    ->text('One of your invoices has been paid!')
    ->headerBlock('Invoice Paid')
    ->dd();
```

<a name="routing-slack-notifications"></a>

### Routing Slack Notifications

Để direct Slack notifications đến Slack team và channel thích hợp, hãy định nghĩa một phương thức `routeNotificationForSlack` trên notifiable model của bạn. Phương thức này có thể trả về một trong ba giá trị:

- `null` - defer routing đến channel được cấu hình trong chính notification. Bạn có thể sử dụng phương thức `to` khi xây dựng `SlackMessage` của bạn để cấu hình channel trong notification.
- Một string chỉ định Slack channel để gửi notification đến, ví dụ `#support-channel`.
- Một instance `SlackRoute`, cho phép bạn chỉ định một OAuth token và channel name, ví dụ `SlackRoute::make($this->slack_channel, $this->slack_token)`. Phương thức này nên được sử dụng để gửi notifications đến external workspaces.

Ví dụ, trả về `#support-channel` từ phương thức `routeNotificationForSlack` sẽ gửi notification đến channel `#support-channel` trong workspace được liên kết với Bot User OAuth token nằm trong file cấu hình `services.php` của ứng dụng của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     */
    public function routeNotificationForSlack(Notification $notification): mixed
    {
        return '#support-channel';
    }
}
```

<a name="notifying-external-slack-workspaces"></a>

### Notifying External Slack Workspaces

> [!NOTE]
> Before sending notifications to external Slack workspaces, your Slack App must be [distributed](#slack-app-distribution).

Tất nhiên, bạn thường sẽ muốn gửi notifications đến các Slack workspaces thuộc sở hữu của users của ứng dụng của bạn. Để làm điều này, trước tiên bạn cần obtain một Slack OAuth token cho user. May mắn thay, [Laravel Socialite](/docs/{{version}}/socialite) bao gồm một Slack driver cho phép bạn dễ dàng authenticate users của ứng dụng của bạn với Slack và [obtain a bot token](/docs/{{version}}/socialite#slack-bot-scopes).

Sau khi bạn đã obtain bot token và lưu trữ nó trong database của ứng dụng của bạn, bạn có thể sử dụng phương thức `SlackRoute::make` để route một notification đến workspace của user. Ngoài ra, ứng dụng của bạn có thể sẽ cần cung cấp một cơ hội cho user để chỉ định channel nào notifications nên được gửi đến:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Slack\SlackRoute;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     */
    public function routeNotificationForSlack(Notification $notification): mixed
    {
        return SlackRoute::make($this->slack_channel, $this->slack_token);
    }
}
```

<a name="localizing-notifications"></a>

## Localizing Notifications

Laravel cho phép bạn gửi notifications trong một locale khác với locale hiện tại của HTTP request, và thậm chí sẽ nhớ locale này nếu notification được queued.

Để thực hiện điều này, class `Illuminate\Notifications\Notification` cung cấp một phương thức `locale` để đặt ngôn ngữ mong muốn. Ứng dụng sẽ thay đổi sang locale này khi notification đang được đánh giá và sau đó revert lại locale trước đó khi đánh giá hoàn tất:

```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
```

Localization của nhiều notifiable entries cũng có thể đạt được qua facade `Notification`:

```php
Notification::locale('es')->send(
    $users, new InvoicePaid($invoice)
);
```

<a name="user-preferred-locales"></a>

#### User Preferred Locales

Đôi khi, applications lưu trữ preferred locale của mỗi user. Bằng cách implement contract `HasLocalePreference` trên notifiable model của bạn, bạn có thể chỉ đạo Laravel sử dụng locale được lưu trữ này khi gửi một notification:

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    /**
     * Get the user's preferred locale.
     */
    public function preferredLocale(): string
    {
        return $this->locale;
    }
}
```

Sau khi bạn đã implement interface, Laravel sẽ tự động sử dụng preferred locale khi gửi notifications và mailables đến model. Vì vậy, không cần gọi phương thức `locale` khi sử dụng interface này:

```php
$user->notify(new InvoicePaid($invoice));
```

<a name="testing"></a>

## Testing

Bạn có thể sử dụng phương thức `fake` của facade `Notification` để ngăn notifications được gửi. Thông thường, việc gửi notifications không liên quan đến code bạn đang thực sự test. Rất có thể, chỉ cần assert rằng Laravel được chỉ thị để gửi một notification nhất định là đủ.

Sau khi gọi phương thức `fake` của facade `Notification`, bạn có thể assert rằng notifications được chỉ thị để gửi đến users và thậm chí inspect dữ liệu mà notifications nhận được:

```php
<?php

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;

test('orders can be shipped', function () {
    Notification::fake();

    // Perform order shipping...

    // Assert that no notifications were sent...
    Notification::assertNothingSent();

    // Assert a notification was sent to the given users...
    Notification::assertSentTo(
        [$user], OrderShipped::class
    );

    // Assert a notification was not sent...
    Notification::assertNotSentTo(
        [$user], AnotherNotification::class
    );

    // Assert a notification was sent twice...
    Notification::assertSentTimes(WeeklyReminder::class, 2);

    // Assert that a given number of notifications were sent...
    Notification::assertCount(3);
});
```

```php
<?php

namespace Tests\Feature;

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Notification::fake();

        // Perform order shipping...

        // Assert that no notifications were sent...
        Notification::assertNothingSent();

        // Assert a notification was sent to the given users...
        Notification::assertSentTo(
            [$user], OrderShipped::class
        );

        // Assert a notification was not sent...
        Notification::assertNotSentTo(
            [$user], AnotherNotification::class
        );

        // Assert a notification was sent twice...
        Notification::assertSentTimes(WeeklyReminder::class, 2);

        // Assert that a given number of notifications were sent...
        Notification::assertCount(3);
    }
}
```

Bạn có thể truyền một closure cho các phương thức `assertSentTo` hoặc `assertNotSentTo` để assert rằng một notification được gửi vượt qua một "truth test" nhất định. Nếu ít nhất một notification được gửi vượt qua truth test đã cho thì assertion sẽ thành công:

```php
Notification::assertSentTo(
    $user,
    function (OrderShipped $notification, array $channels) use ($order) {
        return $notification->order->id === $order->id;
    }
);
```

<a name="on-demand-notifications"></a>

#### On-Demand Notifications

Nếu code bạn đang test gửi [on-demand notifications](#on-demand-notifications), bạn có thể test rằng on-demand notification được gửi qua phương thức `assertSentOnDemand`:

```php
Notification::assertSentOnDemand(OrderShipped::class);
```

Bằng cách truyền một closure làm đối số thứ hai cho phương thức `assertSentOnDemand`, bạn có thể xác định xem một on-demand notification có được gửi đến "route" address đúng hay không:

```php
Notification::assertSentOnDemand(
    OrderShipped::class,
    function (OrderShipped $notification, array $channels, object $notifiable) use ($user) {
        return $notifiable->routes['mail'] === $user->email;
    }
);
```

<a name="notification-events"></a>

## Notification Events

<a name="notification-sending-event"></a>

#### Notification Sending Event

Khi một notification đang được gửi, event `Illuminate\Notifications\Events\NotificationSending` được dispatch bởi notification system. Điều này chứa "notifiable" entity và chính notification instance. Bạn có thể tạo [event listeners](/docs/{{version}}/events) cho event này trong ứng dụng của bạn:

```php
use Illuminate\Notifications\Events\NotificationSending;

class CheckNotificationStatus
{
    /**
     * Handle the event.
     */
    public function handle(NotificationSending $event): void
    {
        // ...
    }
}
```

Notification sẽ không được gửi nếu một event listener cho event `NotificationSending` trả về `false` từ phương thức `handle` của nó:

```php
/**
 * Handle the event.
 */
public function handle(NotificationSending $event): bool
{
    return false;
}
```

Trong một event listener, bạn có thể truy cập các properties `notifiable`, `notification`, và `channel` trên event để tìm hiểu thêm về notification recipient hoặc chính notification:

```php
/**
 * Handle the event.
 */
public function handle(NotificationSending $event): void
{
    // $event->channel
    // $event->notifiable
    // $event->notification
}
```

<a name="notification-sent-event"></a>

#### Notification Sent Event

Khi một notification được gửi, event `Illuminate\Notifications\Events\NotificationSent` [event](/docs/{{version}}/events) được dispatch bởi notification system. Điều này chứa "notifiable" entity và chính notification instance. Bạn có thể tạo [event listeners](/docs/{{version}}/events) cho event này trong ứng dụng của bạn:

```php
use Illuminate\Notifications\Events\NotificationSent;

class LogNotification
{
    /**
     * Handle the event.
     */
    public function handle(NotificationSent $event): void
    {
        // ...
    }
}
```

Trong một event listener, bạn có thể truy cập các properties `notifiable`, `notification`, `channel`, và `response` trên event để tìm hiểu thêm về notification recipient hoặc chính notification:

```php
/**
 * Handle the event.
 */
public function handle(NotificationSent $event): void
{
    // $event->channel
    // $event->notifiable
    // $event->notification
    // $event->response
}
```

<a name="custom-channels"></a>

## Custom Channels

Laravel đi kèm với một vài notification channels, nhưng bạn có thể muốn viết drivers của riêng mình để deliver notifications qua các channels khác. Laravel làm cho điều này trở nên đơn giản. Để bắt đầu, định nghĩa một class chứa một phương thức `send`. Phương thức nên nhận hai đối số: một `$notifiable` và một `$notification`.

Trong phương thức `send`, bạn có thể gọi các phương thức trên notification để retrieve một message object được hiểu bởi channel của bạn và sau đó gửi notification đến instance `$notifiable` tùy ý bạn:

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;

class VoiceChannel
{
    /**
     * Send the given notification.
     */
    public function send(object $notifiable, Notification $notification): void
    {
        $message = $notification->toVoice($notifiable);

        // Send notification to the $notifiable instance...
    }
}
```

Sau khi class notification channel của bạn đã được định nghĩa, bạn có thể trả về tên class từ phương thức `via` của bất kỳ notifications nào của bạn. Trong ví dụ này, phương thức `toVoice` của notification của bạn có thể trả về bất kỳ object nào bạn chọn để đại diện cho voice messages. Ví dụ, bạn có thể định nghĩa class `VoiceMessage` của riêng bạn để đại diện cho các messages này:

```php
<?php

namespace App\Notifications;

use App\Notifications\Messages\VoiceMessage;
use App\Notifications\VoiceChannel;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification
{
    use Queueable;

    /**
     * Get the notification channels.
     */
    public function via(object $notifiable): string
    {
        return VoiceChannel::class;
    }

    /**
     * Get the voice representation of the notification.
     */
    public function toVoice(object $notifiable): VoiceMessage
    {
        // ...
    }
}
```

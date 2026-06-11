# Mail

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
    - [Failover Configuration](#failover-configuration)
    - [Round Robin Configuration](#round-robin-configuration)
- [Generating Mailables](#generating-mailables)
- [Writing Mailables](#writing-mailables)
    - [Configuring the Sender](#configuring-the-sender)
    - [Configuring the View](#configuring-the-view)
    - [View Data](#view-data)
    - [Attachments](#attachments)
    - [Inline Attachments](#inline-attachments)
    - [Attachable Objects](#attachable-objects)
    - [Headers](#headers)
    - [Tags and Metadata](#tags-and-metadata)
    - [Customizing the Symfony Message](#customizing-the-symfony-message)
- [Markdown Mailables](#markdown-mailables)
    - [Generating Markdown Mailables](#generating-markdown-mailables)
    - [Writing Markdown Messages](#writing-markdown-messages)
    - [Customizing the Components](#customizing-the-components)
- [Sending Mail](#sending-mail)
    - [Queueing Mail](#queueing-mail)
- [Rendering Mailables](#rendering-mailables)
    - [Previewing Mailables in the Browser](#previewing-mailables-in-the-browser)
- [Localizing Mailables](#localizing-mailables)
- [Testing](#testing-mailables)
    - [Testing Mailable Content](#testing-mailable-content)
    - [Testing Mailable Sending](#testing-mailable-sending)
- [Mail and Local Development](#mail-and-local-development)
- [Events](#events)
- [Custom Transports](#custom-transports)
    - [Additional Symfony Transports](#additional-symfony-transports)

<a name="introduction"></a>
## Introduction

Gửi email không cần phải phức tạp. Laravel cung cấp một email API sạch, đơn giản được hỗ trợ bởi [Symfony Mailer](https://symfony.com/doc/current/mailer.html) component phổ biến. Laravel và Symfony Mailer cung cấp các drivers để gửi email qua SMTP, Cloudflare, Mailgun, Postmark, Resend, Amazon SES, và `sendmail`, cho phép bạn nhanh chóng bắt đầu gửi mail thông qua một local hoặc cloud-based service theo lựa chọn của bạn.

<a name="configuration"></a>
### Configuration

Các dịch vụ email của Laravel có thể được cấu hình qua file cấu hình `config/mail.php` của ứng dụng. Mỗi mailer được cấu hình trong file này có thể có cấu hình riêng và thậm chí "transport" riêng, cho phép ứng dụng của bạn sử dụng các dịch vụ email khác nhau để gửi các email messages nhất định. Ví dụ, ứng dụng của bạn có thể sử dụng Postmark để gửi transactional emails trong khi sử dụng Amazon SES để gửi bulk emails.

Trong file cấu hình `mail` của bạn, bạn sẽ tìm thấy một mảng cấu hình `mailers`. Mảng này chứa một entry cấu hình mẫu cho mỗi mail driver / transport chính được hỗ trợ bởi Laravel, trong khi giá trị cấu hình `default` xác định mailer nào sẽ được sử dụng theo mặc định khi ứng dụng của bạn cần gửi một email message.

<a name="driver-prerequisites"></a>
### Driver / Transport Prerequisites

Các drivers dựa trên API như Mailgun, Postmark, và Resend thường đơn giản và nhanh hơn việc gửi mail qua SMTP servers. Bất cứ khi nào có thể, chúng tôi khuyến nghị bạn sử dụng một trong các drivers này.

<a name="cloudflare-driver"></a>
#### Cloudflare Driver

Để sử dụng driver Cloudflare, cài đặt Symfony's HTTP Client qua Composer:

```shell
composer require symfony/http-client
```

Tiếp theo, bạn sẽ cần thực hiện hai thay đổi trong file cấu hình `config/mail.php` của ứng dụng. Đầu tiên, đặt default mailer của bạn thành `cloudflare`:

```php
'default' => env('MAIL_MAILER', 'cloudflare'),
```

Thứ hai, thêm mảng cấu hình sau vào mảng `mailers` của bạn:

```php
'cloudflare' => [
    'transport' => 'cloudflare',
],
```

Sau khi cấu hình default mailer của ứng dụng, thêm các tùy chọn sau vào file cấu hình `config/services.php` của bạn:

```php
'cloudflare' => [
    'account_id' => env('CLOUDFLARE_ACCOUNT_ID'),
    'key' => env('CLOUDFLARE_KEY'),
],
```

<a name="mailgun-driver"></a>
#### Mailgun Driver

Để sử dụng driver Mailgun, cài đặt Symfony's Mailgun Mailer transport qua Composer:

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

Tiếp theo, bạn sẽ cần thực hiện hai thay đổi trong file cấu hình `config/mail.php` của ứng dụng. Đầu tiên, đặt default mailer của bạn thành `mailgun`:

```php
'default' => env('MAIL_MAILER', 'mailgun'),
```

Thứ hai, thêm mảng cấu hình sau vào mảng `mailers` của bạn:

```php
'mailgun' => [
    'transport' => 'mailgun',
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

Sau khi cấu hình default mailer của ứng dụng, thêm các tùy chọn sau vào file cấu hình `config/services.php` của bạn:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
],
```

Nếu bạn không sử dụng United States [Mailgun region](https://documentation.mailgun.com/docs/mailgun/api-reference/#mailgun-regions), bạn có thể định nghĩa endpoint của region trong file cấu hình `services`:

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    'scheme' => 'https',
],
```

<a name="postmark-driver"></a>
#### Postmark Driver

Để sử dụng driver [Postmark](https://postmarkapp.com/), cài đặt Symfony's Postmark Mailer transport qua Composer:

```shell
composer require symfony/postmark-mailer symfony/http-client
```

Tiếp theo, đặt tùy chọn `default` trong file cấu hình `config/mail.php` của ứng dụng thành `postmark`. Sau khi cấu hình default mailer của ứng dụng, đảm bảo rằng file cấu hình `config/services.php` của bạn chứa các tùy chọn sau:

```php
'postmark' => [
    'key' => env('POSTMARK_API_KEY'),
],
```

Nếu bạn muốn chỉ định Postmark message stream nên được sử dụng bởi một mailer nhất định, bạn có thể thêm tùy chọn cấu hình `message_stream_id` vào mảng cấu hình của mailer. Mảng cấu hình này có thể được tìm thấy trong file cấu hình `config/mail.php` của ứng dụng:

```php
'postmark' => [
    'transport' => 'postmark',
    'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

Theo cách này, bạn cũng có thể thiết lập nhiều Postmark mailers với các message streams khác nhau.

<a name="resend-driver"></a>
#### Resend Driver

Để sử dụng driver [Resend](https://resend.com/), cài đặt Resend's PHP SDK qua Composer:

```shell
composer require resend/resend-php
```

Tiếp theo, đặt tùy chọn `default` trong file cấu hình `config/mail.php` của ứng dụng thành `resend`. Sau khi cấu hình default mailer của ứng dụng, đảm bảo rằng file cấu hình `config/services.php` của bạn chứa các tùy chọn sau:

```php
'resend' => [
    'key' => env('RESEND_API_KEY'),
],
```

<a name="ses-driver"></a>
#### SES Driver

Để sử dụng driver Amazon SES, trước tiên bạn phải cài đặt Amazon AWS SDK for PHP. Bạn có thể cài đặt library này thông qua trình quản lý gói Composer:

```shell
composer require aws/aws-sdk-php
```

Tiếp theo, đặt tùy chọn `default` trong file cấu hình `config/mail.php` của bạn thành `ses` và xác minh rằng file cấu hình `config/services.php` của bạn chứa các tùy chọn sau:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```

Để sử dụng AWS [temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html) qua một session token, bạn có thể thêm một key `token` vào cấu hình SES của ứng dụng:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'token' => env('AWS_SESSION_TOKEN'),
],
```

Để tương tác với các tính năng [subscription management](https://docs.aws.amazon.com/ses/latest/dg/sending-email-subscription-management.html) của SES, bạn có thể trả về header `X-Ses-List-Management-Options` trong mảng được trả về bởi method [headers](#headers) của một mail message:

```php
/**
 * Get the message headers.
 */
public function headers(): Headers
{
    return new Headers(
        text: [
            'X-Ses-List-Management-Options' => 'contactListName=MyContactList;topicName=MyTopic',
        ],
    );
}
```

Nếu bạn muốn định nghĩa [additional options](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail) mà Laravel nên chuyển đến method `SendEmail` của AWS SDK khi gửi một email, bạn có thể định nghĩa một mảng `options` trong cấu hình `ses` của mình:

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'options' => [
        'ConfigurationSetName' => 'MyConfigurationSet',
        'EmailTags' => [
            ['Name' => 'foo', 'Value' => 'bar'],
        ],
    ],
],
```

<a name="failover-configuration"></a>
### Failover Configuration

Đôi khi, một external service bạn đã cấu hình để gửi mail của ứng dụng có thể bị down. Trong những trường hợp này, có thể hữu ích để định nghĩa một hoặc nhiều cấu hình mail delivery backup sẽ được sử dụng trong trường hợp primary delivery driver của bạn bị down.

Để thực hiện điều này, bạn nên định nghĩa một mailer trong file cấu hình `mail` của ứng dụng sử dụng transport `failover`. Mảng cấu hình cho mailer `failover` của ứng dụng nên chứa một mảng `mailers` tham chiếu thứ tự mà các mailers được cấu hình nên được chọn để delivery:

```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

Khi bạn đã cấu hình một mailer sử dụng transport `failover`, bạn sẽ cần đặt failover mailer làm default mailer của bạn trong file `.env` của ứng dụng để sử dụng chức năng failover:

```ini
MAIL_MAILER=failover
```

<a name="round-robin-configuration"></a>
### Round Robin Configuration

Transport `roundrobin` cho phép bạn phân phối mailing workload của mình qua nhiều mailers. Để bắt đầu, định nghĩa một mailer trong file cấu hình `mail` của ứng dụng sử dụng transport `roundrobin`. Mảng cấu hình cho mailer `roundrobin` của ứng dụng nên chứa một mảng `mailers` tham chiếu các mailers được cấu hình nên được sử dụng để delivery:

```php
'mailers' => [
    'roundrobin' => [
        'transport' => 'roundrobin',
        'mailers' => [
            'ses',
            'postmark',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

Khi round robin mailer của bạn đã được định nghĩa, bạn nên đặt mailer này làm default mailer được sử dụng bởi ứng dụng của bạn bằng cách chỉ định tên của nó làm giá trị của key cấu hình `default` trong file cấu hình `mail` của ứng dụng:

```php
'default' => env('MAIL_MAILER', 'roundrobin'),
```

Transport round robin chọn một mailer ngẫu nhiên từ danh sách các mailers được cấu hình và sau đó chuyển sang mailer có sẵn tiếp theo cho mỗi email tiếp theo. Ngược lại với transport `failover`, giúp đạt *[high availability](https://en.wikipedia.org/wiki/High_availability)*, transport `roundrobin` cung cấp *[load balancing](https://en.wikipedia.org/wiki/Load_balancing_(computing))*.

<a name="generating-mailables"></a>
## Generating Mailables

Khi xây dựng các ứng dụng Laravel, mỗi loại email được gửi bởi ứng dụng của bạn được đại diện như một class "mailable". Các classes này được lưu trữ trong thư mục `app/Mail`. Đừng lo lắng nếu bạn không thấy thư mục này trong ứng dụng của bạn, vì nó sẽ được tạo cho bạn khi bạn tạo mailable class đầu tiên của mình bằng cách sử dụng command Artisan `make:mail`:

```shell
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## Writing Mailables

Khi bạn đã tạo một mailable class, hãy mở nó để chúng ta có thể khám phá nội dung của nó. Cấu hình mailable class được thực hiện trong một số methods, bao gồm các methods `envelope`, `content`, và `attachments`.

Method `envelope` trả về một object `Illuminate\Mail\Mailables\Envelope` định nghĩa subject và, đôi khi, recipients của message. Method `content` trả về một object `Illuminate\Mail\Mailables\Content` định nghĩa [Blade template](/docs/{{version}}/blade) sẽ được sử dụng để tạo nội dung message.

<a name="configuring-the-sender"></a>
### Configuring the Sender

<a name="using-the-envelope"></a>
#### Using the Envelope

Đầu tiên, hãy khám phá cấu hình sender của email. Hoặc, nói cách khác, email sẽ được "from" ai. Có hai cách để cấu hình sender. Đầu tiên, bạn có thể chỉ định address "from" trên envelope của message:

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * Get the message envelope.
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```

Nếu bạn muốn, bạn cũng có thể chỉ định một address `replyTo`:

```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);
```

<a name="using-a-global-from-address"></a>
#### Using a Global `from` Address

Tuy nhiên, nếu ứng dụng của bạn sử dụng cùng address "from" cho tất cả các emails của nó, việc thêm nó vào mỗi mailable class bạn tạo có thể trở nên cồng kềnh. Thay vào đó, bạn có thể chỉ định một global address "from" trong file cấu hình `config/mail.php` của bạn. Address này sẽ được sử dụng nếu không có address "from" nào khác được chỉ định trong mailable class:

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

Ngoài ra, bạn có thể định nghĩa một global address "reply_to" trong file cấu hình `config/mail.php` của bạn:

```php
'reply_to' => [
    'address' => 'example@example.com',
    'name' => 'App Name',
],
```

<a name="configuring-the-view"></a>
### Configuring the View

Trong method `content` của một mailable class, bạn có thể định nghĩa `view`, hoặc template nào nên được sử dụng khi rendering nội dung email. Vì mỗi email thường sử dụng một [Blade template](/docs/{{version}}/blade) để render nội dung của nó, bạn có toàn bộ sức mạnh và sự thuận tiện của Blade templating engine khi xây dựng HTML của email:

```php
/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```

> [!NOTE]
> Bạn có thể muốn tạo một thư mục `resources/views/mail` để lưu trữ tất cả các email templates của mình; tuy nhiên, bạn có thể đặt chúng ở bất cứ đâu bạn muốn trong thư mục `resources/views`.

<a name="plain-text-emails"></a>
#### Plain Text Emails

Nếu bạn muốn định nghĩa một phiên bản plain-text của email, bạn có thể chỉ định template plain-text khi tạo định nghĩa `Content` của message. Giống như tham số `view`, tham số `text` nên là một tên template sẽ được sử dụng để render nội dung của email. Bạn có thể định nghĩa cả phiên bản HTML và plain-text của message:

```php
/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

Để rõ ràng, tham số `html` có thể được sử dụng như một alias của tham số `view`:

```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

<a name="view-data"></a>
### View Data

<a name="via-public-properties"></a>
#### Via Public Properties

Thông thường, bạn sẽ muốn chuyển một số dữ liệu đến view của mình mà bạn có thể sử dụng khi rendering HTML của email. Có hai cách bạn có thể làm cho dữ liệu có sẵn cho view. Đầu tiên, bất kỳ public property nào được định nghĩa trên mailable class của bạn sẽ tự động được cung cấp cho view. Vì vậy, ví dụ, bạn có thể chuyển dữ liệu vào constructor của mailable class và đặt dữ liệu đó thành các public properties được định nghĩa trên class:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct(
        public Order $order,
    ) {}

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```

Khi dữ liệu đã được đặt thành một public property, nó sẽ tự động có sẵn trong view của bạn, vì vậy bạn có thể truy xuất nó giống như bạn truy xuất bất kỳ dữ liệu nào khác trong Blade templates:

```blade
<div>
    Price: {{ $order->price }}
</div>
```

<a name="via-the-with-parameter"></a>
#### Via the `with` Parameter:

Nếu bạn muốn tùy chỉnh định dạng dữ liệu email của mình trước khi nó được gửi đến template, bạn có thể chuyển thủ công dữ liệu của mình đến view thông qua tham số `with` của định nghĩa `Content`. Thông thường, bạn vẫn sẽ chuyển dữ liệu qua constructor của mailable class; tuy nhiên, bạn nên đặt dữ liệu này thành các properties `protected` hoặc `private` để dữ liệu không tự động được cung cấp cho template:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct(
        protected Order $order,
    ) {}

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
            with: [
                'orderName' => $this->order->name,
                'orderPrice' => $this->order->price,
            ],
        );
    }
}
```

Khi dữ liệu đã được chuyển qua tham số `with`, nó sẽ tự động có sẵn trong view của bạn, vì vậy bạn có thể truy xuất nó giống như bạn truy xuất bất kỳ dữ liệu nào khác trong Blade templates:

```blade
<div>
    Price: {{ $orderPrice }}
</div>
```

<a name="attachments"></a>
### Attachments

Để thêm attachments vào một email, bạn sẽ thêm attachments vào mảng được trả về bởi method `attachments` của message. Đầu tiên, bạn có thể thêm một attachment bằng cách cung cấp một file path cho method `fromPath` được cung cấp bởi class `Attachment`:

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file'),
    ];
}
```

Khi attaching files vào một message, bạn cũng có thể chỉ định display name và / hoặc MIME type cho attachment bằng cách sử dụng các methods `as` và `withMime`:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="attaching-files-from-disk"></a>
#### Attaching Files From Disk

Nếu bạn đã lưu trữ một file trên một trong các [filesystem disks](/docs/{{version}}/filesystem) của mình, bạn có thể attach nó vào email bằng cách sử dụng method attachment `fromStorage`:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

Tất nhiên, bạn cũng có thể chỉ định tên và MIME type của attachment:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

Method `fromStorageDisk` có thể được sử dụng nếu bạn cần chỉ định một storage disk khác với default disk của mình:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorageDisk('s3', '/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="raw-data-attachments"></a>
#### Raw Data Attachments

Method attachment `fromData` có thể được sử dụng để attach một raw string của bytes như một attachment. Ví dụ, bạn có thể sử dụng method này nếu bạn đã tạo một PDF trong bộ nhớ và muốn attach nó vào email mà không cần viết nó vào disk. Method `fromData` chấp nhận một closure giải quyết raw data bytes cũng như tên mà attachment nên được gán:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="inline-attachments"></a>
### Inline Attachments

Embedding inline images vào emails của bạn thường cồng kềnh; tuy nhiên, Laravel cung cấp một cách thuận tiện để attach images vào emails. Để embed một inline image, sử dụng method `embed` trên biến `$message` trong email template của bạn. Laravel tự động làm cho biến `$message` có sẵn cho tất cả các email templates của bạn, vì vậy bạn không cần lo lắng về việc chuyển nó thủ công:

```blade
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> [!WARNING]
> Biến `$message` không có sẵn trong các templates message plain-text vì plain-text messages không sử dụng inline attachments.

<a name="embedding-raw-data-attachments"></a>
#### Embedding Raw Data Attachments

Nếu bạn đã có một raw image data string bạn muốn embed vào một email template, bạn có thể gọi method `embedData` trên biến `$message`. Khi gọi method `embedData`, bạn sẽ cần cung cấp một filename nên được gán cho embedded image:

```blade
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

<a name="attachable-objects"></a>
### Attachable Objects

Mặc dù attaching files vào messages qua các string paths đơn giản thường đủ, trong nhiều trường hợp các attachable entities trong ứng dụng của bạn được đại diện bởi các classes. Ví dụ, nếu ứng dụng của bạn đang attaching một photo vào một message, ứng dụng của bạn cũng có thể có một model `Photo` đại diện cho photo đó. Khi đó, wouldn't it be convenient để chỉ cần chuyển model `Photo` cho method `attach`? Attachable objects cho phép bạn làm chính điều đó.

Để bắt đầu, implement interface `Illuminate\Contracts\Mail\Attachable` trên object sẽ được attachable đến messages. Interface này quy định rằng class của bạn định nghĩa một method `toMailAttachment` trả về một instance `Illuminate\Mail\Attachment`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Mail\Attachable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Mail\Attachment;

class Photo extends Model implements Attachable
{
    /**
     * Get the attachable representation of the model.
     */
    public function toMailAttachment(): Attachment
    {
        return Attachment::fromPath('/path/to/file');
    }
}
```

Khi bạn đã định nghĩa attachable object của mình, bạn có thể trả về một instance của object đó từ method `attachments` khi xây dựng một email message:

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [$this->photo];
}
```

Tất nhiên, dữ liệu attachment có thể được lưu trữ trên một remote file storage service như Amazon S3. Vì vậy, Laravel cũng cho phép bạn tạo attachment instances từ dữ liệu được lưu trữ trên một trong các [filesystem disks](/docs/{{version}}/filesystem) của ứng dụng:

```php
// Create an attachment from a file on your default disk...
return Attachment::fromStorage($this->path);

// Create an attachment from a file on a specific disk...
return Attachment::fromStorageDisk('backblaze', $this->path);
```

Ngoài ra, bạn có thể tạo attachment instances qua dữ liệu bạn có trong bộ nhớ. Để thực hiện điều này, cung cấp một closure cho method `fromData`. Closure nên trả về raw data đại diện cho attachment:

```php
return Attachment::fromData(fn () => $this->content, 'Photo Name');
```

Laravel cũng cung cấp các methods bổ sung mà bạn có thể sử dụng để tùy chỉnh attachments của mình. Ví dụ, bạn có thể sử dụng các methods `as` và `withMime` để tùy chỉnh tên file và MIME type của file:

```php
return Attachment::fromPath('/path/to/file')
    ->as('Photo Name')
    ->withMime('image/jpeg');
```

<a name="headers"></a>
### Headers

Đôi khi bạn cần attach các headers bổ sung vào outgoing message. Ví dụ, bạn có thể cần đặt một custom `Message-Id` hoặc các text headers tùy ý khác.

Để thực hiện điều này, định nghĩa một method `headers` trên mailable của bạn. Method `headers` nên trả về một instance `Illuminate\Mail\Mailables\Headers`. Class này chấp nhận các tham số `messageId`, `references`, và `text`. Tất nhiên, bạn có thể chỉ cung cấp các tham số bạn cần cho message cụ thể của mình:

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * Get the message headers.
 */
public function headers(): Headers
{
    return new Headers(
        messageId: 'custom-message-id@example.com',
        references: ['previous-message@example.com'],
        text: [
            'X-Custom-Header' => 'Custom Value',
        ],
    );
}
```

<a name="tags-and-metadata"></a>
### Tags and Metadata

Một số third-party email providers như Mailgun và Postmark hỗ trợ message "tags" và "metadata", có thể được sử dụng để nhóm và theo dõi các emails được gửi bởi ứng dụng của bạn. Bạn có thể thêm tags và metadata vào một email message thông qua định nghĩa `Envelope` của bạn:

```php
use Illuminate\Mail\Mailables\Envelope;

/**
 * Get the message envelope.
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

Nếu ứng dụng của bạn sử dụng driver Mailgun, bạn có thể tham khảo tài liệu Mailgun để biết thêm thông tin về [tags](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#tags) và [metadata](https://documentation.mailgun.com/docs/mailgun/user-manual/sending-messages/#attaching-metadata-to-messages). Tương tự, tài liệu Postmark cũng có thể được tham khảo để biết thêm thông tin về hỗ trợ của họ cho [tags](https://postmarkapp.com/blog/tags-support-for-smtp) và [metadata](https://postmarkapp.com/support/article/1125-custom-metadata-faq).

Nếu ứng dụng của bạn sử dụng Amazon SES để gửi emails, bạn nên sử dụng method `metadata` để attach [SES "tags"](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) vào message.

<a name="customizing-the-symfony-message"></a>
### Customizing the Symfony Message

Khả năng mail của Laravel được hỗ trợ bởi Symfony Mailer. Laravel cho phép bạn đăng ký các custom callbacks sẽ được gọi với Symfony Message instance trước khi gửi message. Điều này cho phép bạn tùy chỉnh sâu message trước khi nó được gửi. Để thực hiện điều này, định nghĩa một tham số `using` trên định nghĩa `Envelope` của bạn:

```php
use Illuminate\Mail\Mailables\Envelope;
use Symfony\Component\Mime\Email;

/**
 * Get the message envelope.
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        using: [
            function (Email $message) {
                // ...
            },
        ]
    );
}
```

<a name="markdown-mailables"></a>
## Markdown Mailables

Markdown mailable messages cho phép bạn tận dụng các templates và components được xây dựng sẵn của [mail notifications](/docs/{{version}}/notifications#mail-notifications) trong mailables của bạn. Vì messages được viết bằng Markdown, Laravel có thể render các HTML templates đẹp, responsive cho messages trong khi cũng tự động tạo một plain-text counterpart.

<a name="generating-markdown-mailables"></a>
### Generating Markdown Mailables

Để tạo một mailable với một Markdown template tương ứng, bạn có thể sử dụng tùy chọn `--markdown` của command Artisan `make:mail`:

```shell
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

Sau đó, khi cấu hình định nghĩa mailable `Content` trong method `content` của nó, sử dụng tham số `markdown` thay vì tham số `view`:

```php
use Illuminate\Mail\Mailables\Content;

/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        markdown: 'mail.orders.shipped',
        with: [
            'url' => $this->orderUrl,
        ],
    );
}
```

<a name="writing-markdown-messages"></a>
### Writing Markdown Messages

Markdown mailables sử dụng sự kết hợp của Blade components và Markdown syntax cho phép bạn dễ dàng xây dựng mail messages trong khi tận dụng các email UI components được xây dựng sẵn của Laravel:

```blade
<x-mail::message>
# Order Shipped

Your order has been shipped!

<x-mail::button :url="$url">
View Order
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> [!NOTE]
> Đừng sử dụng excess indentation khi viết Markdown emails. Theo tiêu chuẩn Markdown, Markdown parsers sẽ render nội dung được indent như code blocks.

<a name="button-component"></a>
#### Button Component

Component button render một button link được căn giữa. Component chấp nhận hai đối số, một `url` và một `color` tùy chọn. Các màu được hỗ trợ là `primary`, `success`, và `error`. Bạn có thể thêm bao nhiêu button components vào một message tùy thích:

```blade
<x-mail::button :url="$url" color="success">
View Order
</x-mail::button>
```

<a name="panel-component"></a>
#### Panel Component

Component panel render khối văn bản đã cho trong một panel có màu nền hơi khác so với phần còn lại của message. Điều này cho phép bạn thu hút sự chú ý đến một khối văn bản nhất định:

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### Table Component

Component table cho phép bạn chuyển đổi một Markdown table thành một HTML table. Component chấp nhận Markdown table làm nội dung của nó. Căn chỉnh cột bảng được hỗ trợ bằng cách sử dụng cú pháp căn chỉnh bảng Markdown mặc định:

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

Bạn có thể export tất cả các Markdown mail components đến ứng dụng của mình để tùy chỉnh. Để export các components, sử dụng command Artisan `vendor:publish` để publish asset tag `laravel-mail`:

```shell
php artisan vendor:publish --tag=laravel-mail
```

Command này sẽ publish các Markdown mail components đến thư mục `resources/views/vendor/mail`. Thư mục `mail` sẽ chứa một thư mục `html` và một thư mục `text`, mỗi thư mục chứa các đại diện tương ứng của mỗi component có sẵn. Bạn có thể tùy chỉnh các components này theo cách bạn thích.

<a name="customizing-the-css"></a>
#### Customizing the CSS

Sau khi export các components, thư mục `resources/views/vendor/mail/html/themes` sẽ chứa một file `default.css`. Bạn có thể tùy chỉnh CSS trong file này và styles của bạn sẽ tự động được chuyển thành inline CSS styles trong các đại diện HTML của Markdown mail messages của bạn.

Nếu bạn muốn xây dựng một theme hoàn toàn mới cho các Markdown components của Laravel, bạn có thể đặt một file CSS trong thư mục `html/themes`. Sau khi đặt tên và lưu file CSS của bạn, cập nhật tùy chọn `theme` của file cấu hình `config/mail.php` của ứng dụng để khớp với tên của theme mới của bạn.

Để tùy chỉnh theme cho một mailable cụ thể, bạn có thể đặt property `$theme` của mailable class thành tên của theme nên được sử dụng khi gửi mailable đó.

<a name="sending-mail"></a>
## Sending Mail

Để gửi một message, sử dụng method `to` trên facade `Mail` [facade](/docs/{{version}}/facades). Method `to` chấp nhận một email address, một user instance, hoặc một collection của users. Nếu bạn chuyển một object hoặc collection của objects, mailer sẽ tự động sử dụng các properties `email` và `name` của họ khi xác định recipients của email, vì vậy hãy đảm bảo các attributes này có sẵn trên objects của bạn. Khi bạn đã chỉ định recipients của mình, bạn có thể chuyển một instance của mailable class của mình cho method `send`:

```php
<?php

namespace App\Http\Controllers;

use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Ship the order...

        Mail::to($request->user())->send(new OrderShipped($order));

        return redirect('/orders');
    }
}
```

Bạn không bị giới hạn chỉ để chỉ định recipients "to" khi gửi một message. Bạn có thể tự do đặt recipients "to", "cc", và "bcc" bằng cách chaining các methods tương ứng của họ lại với nhau:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

<a name="looping-over-recipients"></a>
#### Looping Over Recipients

Thỉnh thoảng, bạn có thể cần gửi một mailable đến một danh sách recipients bằng cách lặp qua một mảng recipients / email addresses. Tuy nhiên, vì method `to` append email addresses vào danh sách recipients của mailable, mỗi lần lặp qua loop sẽ gửi một email khác đến mọi recipient trước đó. Do đó, bạn nên luôn re-create mailable instance cho mỗi recipient:

```php
foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

<a name="sending-mail-via-a-specific-mailer"></a>
#### Sending Mail via a Specific Mailer

Theo mặc định, Laravel sẽ gửi email bằng cách sử dụng mailer được cấu hình làm default mailer trong file cấu hình `mail` của ứng dụng. Tuy nhiên, bạn có thể sử dụng method `mailer` để gửi một message bằng cách sử dụng một cấu hình mailer cụ thể:

```php
Mail::mailer('postmark')
    ->to($request->user())
    ->send(new OrderShipped($order));
```

<a name="queueing-mail"></a>
### Queueing Mail

<a name="queueing-a-mail-message"></a>
#### Queueing a Mail Message

Vì gửi email messages có thể ảnh hưởng tiêu cực đến response time của ứng dụng, nhiều nhà phát triển chọn để queue email messages để gửi trong nền. Laravel làm cho điều này dễ dàng bằng cách sử dụng [unified queue API](/docs/{{version}}/queues) được xây dựng sẵn của nó. Để queue một mail message, sử dụng method `queue` trên facade `Mail` sau khi chỉ định recipients của message:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

Method này sẽ tự động lo việc đẩy một job lên queue để message được gửi trong nền. Bạn sẽ cần [cấu hình queues của bạn](/docs/{{version}}/queues) trước khi sử dụng tính năng này.

<a name="delayed-message-queueing"></a>
#### Delayed Message Queueing

Nếu bạn muốn trì hoãn delivery của một queued email message, bạn có thể sử dụng method `later`. Làm đối số đầu tiên, method `later` chấp nhận một instance `DateTime` chỉ định khi nào message nên được gửi:

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later(now()->plus(minutes: 10), new OrderShipped($order));
```

<a name="pushing-to-specific-queues"></a>
#### Pushing to Specific Queues

Vì tất cả các mailable classes được tạo bằng cách sử dụng command `make:mail` sử dụng trait `Illuminate\Bus\Queueable`, bạn có thể gọi các methods `onQueue` và `onConnection` trên bất kỳ instance mailable class nào, cho phép bạn chỉ định connection và queue name cho message:

```php
$message = (new OrderShipped($order))
    ->onConnection('sqs')
    ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

Ngoài ra, bạn có thể chỉ định connection và queue bằng cách sử dụng các attributes `Connection` và `Queue` trên mailable class:

```php
use Illuminate\Queue\Attributes\Connection;
use Illuminate\Queue\Attributes\Queue;

#[Connection('sqs')]
#[Queue('emails')]
class OrderShipped extends Mailable
{
    // ...
}
```

<a name="queueing-by-default"></a>
#### Queueing by Default

Nếu bạn có các mailable classes mà bạn muốn luôn được queued, bạn có thể implement contract `ShouldQueue` trên class. Bây giờ, ngay cả khi bạn gọi method `send` khi mailing, mailable vẫn sẽ được queued vì nó implement contract:

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    // ...
}
```

<a name="queued-mailables-and-database-transactions"></a>
#### Queued Mailables and Database Transactions

Khi queued mailables được dispatch trong database transactions, chúng có thể được xử lý bởi queue trước khi database transaction đã commit. Khi điều này xảy ra, bất kỳ cập nhật nào bạn đã thực hiện cho models hoặc database records trong database transaction có thể chưa được phản ánh trong database. Ngoài ra, bất kỳ models hoặc database records nào được tạo trong transaction có thể không tồn tại trong database. Nếu mailable của bạn phụ thuộc vào các models này, các lỗi bất ngờ có thể xảy ra khi job gửi queued mailable được xử lý.

Nếu tùy chọn cấu hình `after_commit` của queue connection của bạn được đặt thành `false`, bạn vẫn có thể chỉ định rằng một queued mailable cụ thể nên được dispatch sau khi tất cả các database transactions mở đã được commit bằng cách gọi method `afterCommit` khi gửi mail message:

```php
Mail::to($request->user())->send(
    (new OrderShipped($order))->afterCommit()
);
```

Ngoài ra, bạn có thể gọi method `afterCommit` từ constructor của mailable:

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> [!NOTE]
> Để biết thêm thông tin về cách giải quyết các vấn đề này, hãy xem lại tài liệu về [queued jobs và database transactions](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="queued-email-failures"></a>
#### Queued Email Failures

Khi một queued email fails, method `failed` trên queued mailable class sẽ được gọi nếu nó đã được định nghĩa. Instance `Throwable` gây ra queued email fail sẽ được chuyển đến method `failed`:

```php
<?php

namespace App\Mail;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Throwable;

class OrderDelayed extends Mailable implements ShouldQueue
{
    use SerializesModels;

    /**
     * Handle a queued email's failure.
     */
    public function failed(Throwable $exception): void
    {
        // ...
    }
}
```

<a name="rendering-mailables"></a>
## Rendering Mailables

Đôi khi bạn có thể muốn capture nội dung HTML của một mailable mà không gửi nó. Để thực hiện điều này, bạn có thể gọi method `render` của mailable. Method này sẽ trả về nội dung HTML được đánh giá của mailable như một string:

```php
use App\Mail\InvoicePaid;
use App\Models\Invoice;

$invoice = Invoice::find(1);

return (new InvoicePaid($invoice))->render();
```

<a name="previewing-mailables-in-the-browser"></a>
### Previewing Mailables in the Browser

Khi thiết kế template của một mailable, thuận tiện để preview nhanh mailable được render trong browser của bạn như một Blade template điển hình. Vì lý do này, Laravel cho phép bạn trả về bất kỳ mailable nào trực tiếp từ một route closure hoặc controller. Khi một mailable được trả về, nó sẽ được render và hiển thị trong browser, cho phép bạn preview nhanh thiết kế của nó mà không cần gửi nó đến một email address thực tế:

```php
Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

<a name="localizing-mailables"></a>
## Localizing Mailables

Laravel cho phép bạn gửi mailables trong một locale khác với locale hiện tại của request, và thậm chí sẽ nhớ locale này nếu mail được queued.

Để thực hiện điều này, facade `Mail` cung cấp một method `locale` để đặt ngôn ngữ mong muốn. Ứng dụng sẽ chuyển sang locale này khi template của mailable đang được đánh giá và sau đó revert lại về locale trước đó khi đánh giá hoàn tất:

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

<a name="user-preferred-locales"></a>
#### User Preferred Locales

Đôi khi, các ứng dụng lưu trữ preferred locale của mỗi user. Bằng cách implement contract `HasLocalePreference` trên một hoặc nhiều models của bạn, bạn có thể hướng dẫn Laravel sử dụng locale được lưu trữ này khi gửi mail:

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

Khi bạn đã implement interface, Laravel sẽ tự động sử dụng preferred locale khi gửi mailables và notifications đến model. Do đó, không cần gọi method `locale` khi sử dụng interface này:

```php
Mail::to($request->user())->send(new OrderShipped($order));
```

<a name="testing-mailables"></a>
## Testing

<a name="testing-mailable-content"></a>
### Testing Mailable Content

Laravel cung cấp nhiều methods để kiểm tra cấu trúc của mailable. Ngoài ra, Laravel cung cấp một số methods thuận tiện để testing rằng mailable của bạn chứa nội dung mà bạn mong đợi:

```php tab=Pest
use App\Mail\InvoicePaid;
use App\Models\User;

test('mailable content', function () {
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertDontSeeInHtml('Invoice Not Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertDontSeeInText('Invoice Not Paid');
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
});
```

```php tab=PHPUnit
use App\Mail\InvoicePaid;
use App\Models\User;

public function test_mailable_content(): void
{
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertDontSeeInHtml('Invoice Not Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertDontSeeInText('Invoice Not Paid');
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
}
```

Như bạn có thể mong đợi, các assertions "HTML" assert rằng phiên bản HTML của mailable chứa một string nhất định, trong khi các assertions "text" assert rằng phiên bản plain-text của mailable chứa một string nhất định.

<a name="testing-mailable-sending"></a>
### Testing Mailable Sending

Chúng tôi khuyến nghị testing nội dung của mailables riêng biệt với các tests của bạn assert rằng một mailable nhất định đã được "sent" đến một user cụ thể. Thông thường, nội dung của mailables không liên quan đến code bạn đang testing, và đủ để chỉ assert rằng Laravel được hướng dẫn để gửi một mailable nhất định.

Bạn có thể sử dụng method `fake` của facade `Mail` để ngăn chặn mail được gửi. Sau khi gọi method `fake` của facade `Mail`, bạn có thể sau đó assert rằng mailables được hướng dẫn để được gửi đến users và thậm chí kiểm tra dữ liệu mà mailables nhận được:

```php tab=Pest
<?php

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;

test('orders can be shipped', function () {
    Mail::fake();

    // Perform order shipping...

    // Assert that no mailables were sent...
    Mail::assertNothingSent();

    // Assert that a mailable was sent...
    Mail::assertSent(OrderShipped::class);

    // Assert a mailable was sent twice...
    Mail::assertSent(OrderShipped::class, 2);

    // Assert a mailable was sent to an email address...
    Mail::assertSent(OrderShipped::class, 'example@laravel.com');

    // Assert a mailable was sent to multiple email addresses...
    Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

    // Assert a mailable was not sent...
    Mail::assertNotSent(AnotherMailable::class);

    // Assert a mailable was sent twice...
    Mail::assertSentTimes(OrderShipped::class, 2);

    // Assert 3 total mailables were sent...
    Mail::assertSentCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Mail::fake();

        // Perform order shipping...

        // Assert that no mailables were sent...
        Mail::assertNothingSent();

        // Assert that a mailable was sent...
        Mail::assertSent(OrderShipped::class);

        // Assert a mailable was sent twice...
        Mail::assertSent(OrderShipped::class, 2);

        // Assert a mailable was sent to an email address...
        Mail::assertSent(OrderShipped::class, 'example@laravel.com');

        // Assert a mailable was sent to multiple email addresses...
        Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

        // Assert a mailable was not sent...
        Mail::assertNotSent(AnotherMailable::class);

        // Assert a mailable was sent twice...
        Mail::assertSentTimes(OrderShipped::class, 2);

        // Assert 3 total mailables were sent...
        Mail::assertSentCount(3);
    }
}
```

Nếu bạn đang queueing mailables để delivery trong nền, bạn nên sử dụng method `assertQueued` thay vì `assertSent`:

```php
Mail::assertQueued(OrderShipped::class);
Mail::assertNotQueued(OrderShipped::class);
Mail::assertNothingQueued();
Mail::assertQueuedCount(3);
```

Bạn cũng có thể assert tổng số mailables đã được gửi hoặc queued bằng cách sử dụng method `assertOutgoingCount`:

```php
Mail::assertOutgoingCount(3);
```

Bạn có thể chuyển một closure cho các methods `assertSent`, `assertNotSent`, `assertQueued`, hoặc `assertNotQueued` để assert rằng một mailable đã được gửi vượt qua một "truth test" nhất định. Nếu ít nhất một mailable đã được gửi vượt qua truth test đã cho thì assertion sẽ thành công:

```php
Mail::assertSent(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

Khi gọi các methods assertion của facade `Mail`, instance mailable được chấp nhận bởi closure được cung cấp cung cấp các methods hữu ích để kiểm tra mailable:

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasCc('...') &&
           $mail->hasBcc('...') &&
           $mail->hasReplyTo('...') &&
           $mail->hasFrom('...') &&
           $mail->hasSubject('...') &&
           $mail->hasMetadata('order_id', $mail->order->id);
           $mail->usesMailer('ses');
});
```

Instance mailable cũng bao gồm một số methods hữu ích để kiểm tra các attachments trên một mailable:

```php
use Illuminate\Mail\Mailables\Attachment;

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromPath('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromStorageDisk('s3', '/path/to/file')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($pdfData) {
    return $mail->hasAttachment(
        Attachment::fromData(fn () => $pdfData, 'name.pdf')
    );
});
```

Bạn có thể nhận thấy rằng có hai methods để assert rằng mail không được gửi: `assertNotSent` và `assertNotQueued`. Đôi khi bạn có thể muốn assert rằng không có mail nào được gửi **hoặc** queued. Để thực hiện điều này, bạn có thể sử dụng các methods `assertNothingOutgoing` và `assertNotOutgoing`:

```php
Mail::assertNothingOutgoing();

Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

<a name="mail-and-local-development"></a>
## Mail and Local Development

Khi phát triển một ứng dụng gửi email, bạn có thể không muốn thực sự gửi emails đến các live email addresses. Laravel cung cấp một số cách để "disable" việc gửi thực tế của emails trong local development.

<a name="log-driver"></a>
#### Log Driver

Thay vì gửi emails của bạn, driver mail `log` sẽ viết tất cả email messages đến log files của bạn để kiểm tra. Thông thường, driver này chỉ được sử dụng trong local development. Để biết thêm thông tin về cấu hình ứng dụng của bạn theo môi trường, hãy xem [tài liệu cấu hình](/docs/{{version}}/configuration#environment-configuration).

<a name="mailtrap"></a>
#### HELO / Mailtrap / Mailpit

Ngoài ra, bạn có thể sử dụng một service như [HELO](https://usehelo.com) hoặc [Mailtrap](https://mailtrap.io) và driver `smtp` để gửi email messages của bạn đến một "dummy" mailbox nơi bạn có thể xem chúng trong một email client thực tế. Cách tiếp cận này có lợi ích là cho phép bạn thực sự kiểm tra các emails cuối cùng trong message viewer của Mailtrap.

Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail), bạn có thể preview messages của mình bằng cách sử dụng [Mailpit](https://github.com/axllent/mailpit). Khi Sail đang chạy, bạn có thể truy cập Mailpit interface tại: `http://localhost:8025`.

<a name="using-a-global-to-address"></a>
#### Using a Global `to` Address

Cuối cùng, bạn có thể chỉ định một global address "to" bằng cách gọi method `alwaysTo` được cung cấp bởi facade `Mail`. Thông thường, method này nên được gọi từ method `boot` của một trong các service providers của ứng dụng:

```php
use Illuminate\Support\Facades\Mail;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    if ($this->app->environment('local')) {
        Mail::alwaysTo('taylor@example.com');
    }
}
```

Khi sử dụng method `alwaysTo`, bất kỳ address "cc" hoặc "bcc" bổ sung nào trên mail messages sẽ bị xóa.

<a name="events"></a>
## Events

Laravel dispatch hai events trong khi gửi mail messages. Event `MessageSending` được dispatch trước khi một message được gửi, trong khi event `MessageSent` được dispatch sau khi một message đã được gửi. Hãy nhớ rằng, các events này được dispatch khi mail đang được *sent*, không phải khi nó được queued. Bạn có thể tạo [event listeners](/docs/{{version}}/events) cho các events này trong ứng dụng:

```php
use Illuminate\Mail\Events\MessageSending;
// use Illuminate\Mail\Events\MessageSent;

class LogMessage
{
    /**
     * Handle the event.
     */
    public function handle(MessageSending $event): void
    {
        // ...
    }
}
```

<a name="custom-transports"></a>
## Custom Transports

Laravel bao gồm nhiều mail transports; tuy nhiên, bạn có thể muốn viết transports của riêng bạn để delivery email qua các services khác mà Laravel không hỗ trợ out of the box. Để bắt đầu, định nghĩa một class extends class `Symfony\Component\Mailer\Transport\AbstractTransport`. Sau đó, implement các methods `doSend` và `__toString` trên transport của bạn:

```php
<?php

namespace App\Mail;

use MailchimpTransactional\ApiClient;
use Symfony\Component\Mailer\SentMessage;
use Symfony\Component\Mailer\Transport\AbstractTransport;
use Symfony\Component\Mime\Address;
use Symfony\Component\Mime\MessageConverter;

class MailchimpTransport extends AbstractTransport
{
    /**
     * Create a new Mailchimp transport instance.
     */
    public function __construct(
        protected ApiClient $client,
    ) {
        parent::__construct();
    }

    /**
     * {@inheritDoc}
     */
    protected function doSend(SentMessage $message): void
    {
        $email = MessageConverter::toEmail($message->getOriginalMessage());

        $this->client->messages->send(['message' => [
            'from_email' => $email->getFrom(),
            'to' => collect($email->getTo())->map(function (Address $email) {
                return ['email' => $email->getAddress(), 'type' => 'to'];
            })->all(),
            'subject' => $email->getSubject(),
            'text' => $email->getTextBody(),
        ]]);
    }

    /**
     * Get the string representation of the transport.
     */
    public function __toString(): string
    {
        return 'mailchimp';
    }
}
```

Khi bạn đã định nghĩa custom transport của mình, bạn có thể đăng ký nó thông qua method `extend` được cung cấp bởi facade `Mail`. Thông thường, điều này nên được thực hiện trong method `boot` của `AppServiceProvider` của ứng dụng. Một đối số `$config` sẽ được chuyển đến closure được cung cấp cho method `extend`. Đối số này sẽ chứa mảng cấu hình được định nghĩa cho mailer trong file cấu hình `config/mail.php` của ứng dụng:

```php
use App\Mail\MailchimpTransport;
use Illuminate\Support\Facades\Mail;
use MailchimpTransactional\ApiClient;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Mail::extend('mailchimp', function (array $config = []) {
        $client = new ApiClient;

        $client->setApiKey($config['key']);

        return new MailchimpTransport($client);
    });
}
```

Khi custom transport của bạn đã được định nghĩa và đăng ký, bạn có thể tạo một định nghĩa mailer trong file cấu hình `config/mail.php` của ứng dụng sử dụng transport mới:

```php
'mailchimp' => [
    'transport' => 'mailchimp',
    'key' => env('MAILCHIMP_API_KEY'),
    // ...
],
```

<a name="additional-symfony-transports"></a>
### Additional Symfony Transports

Laravel bao gồm hỗ trợ cho một số Symfony maintained mail transports hiện có như Mailgun và Postmark. Tuy nhiên, bạn có thể muốn mở rộng Laravel với hỗ trợ cho các Symfony maintained transports bổ sung. Bạn có thể làm như vậy bằng cách yêu cầu Symfony mailer cần thiết qua Composer và đăng ký transport với Laravel. Ví dụ, bạn có thể cài đặt và đăng ký Symfony mailer "Brevo" (trước đây là "Sendinblue"):

```shell
composer require symfony/brevo-mailer symfony/http-client
```

Khi package Brevo mailer đã được cài đặt, bạn có thể thêm một entry cho Brevo API credentials của bạn vào file cấu hình `services` của ứng dụng:

```php
'brevo' => [
    'key' => env('BREVO_API_KEY'),
],
```

Tiếp theo, bạn có thể sử dụng method `extend` của facade `Mail` để đăng ký transport với Laravel. Thông thường, điều này nên được thực hiện trong method `boot` của một service provider:

```php
use Illuminate\Support\Facades\Mail;
use Symfony\Component\Mailer\Bridge\Brevo\Transport\BrevoTransportFactory;
use Symfony\Component\Mailer\Transport\Dsn;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Mail::extend('brevo', function () {
        return (new BrevoTransportFactory)->create(
            new Dsn(
                'brevo+api',
                'default',
                config('services.brevo.key')
            )
        );
    });
}
```

Khi transport của bạn đã được đăng ký, bạn có thể tạo một định nghĩa mailer trong file cấu hình `config/mail.php` của ứng dụng sử dụng transport mới:

```php
'brevo' => [
    'transport' => 'brevo',
    // ...
],
```

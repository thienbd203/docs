# Events

- [Introduction](#introduction)
- [Generating Events and Listeners](#generating-events-and-listeners)
- [Registering Events and Listeners](#registering-events-and-listeners)
    - [Event Discovery](#event-discovery)
    - [Manually Registering Events](#manually-registering-events)
    - [Closure Listeners](#closure-listeners)
- [Defining Events](#defining-events)
- [Defining Listeners](#defining-listeners)
- [Queued Event Listeners](#queued-event-listeners)
    - [Manually Interacting With the Queue](#manually-interacting-with-the-queue)
    - [Queued Event Listeners and Database Transactions](#queued-event-listeners-and-database-transactions)
    - [Queued Listener Middleware](#queued-listener-middleware)
    - [Encrypted Queued Listeners](#encrypted-queued-listeners)
    - [Unique Event Listeners](#unique-event-listeners)
        - [Keeping Listeners Unique Until Processing Begins](#keeping-listeners-unique-until-processing-begins)
        - [Unique Listener Locks](#unique-listener-locks)
    - [Handling Failed Jobs](#handling-failed-jobs)
- [Dispatching Events](#dispatching-events)
    - [Dispatching Events After Database Transactions](#dispatching-events-after-database-transactions)
    - [Deferring Events](#deferring-events)
- [Event Subscribers](#event-subscribers)
    - [Writing Event Subscribers](#writing-event-subscribers)
    - [Registering Event Subscribers](#registering-event-subscribers)
- [Testing](#testing)
    - [Faking a Subset of Events](#faking-a-subset-of-events)
    - [Scoped Events Fakes](#scoped-event-fakes)

<a name="introduction"></a>
## Introduction

Events của Laravel cung cấp một triển khai observer pattern đơn giản, cho phép bạn subscribe và lắng nghe các sự kiện khác nhau xảy ra trong ứng dụng của bạn. Event classes thường được lưu trữ trong thư mục `app/Events`, trong khi listeners của chúng được lưu trữ trong `app/Listeners`. Đừng lo lắng nếu bạn không thấy các thư mục này trong ứng dụng của bạn vì chúng sẽ được tạo cho bạn khi bạn tạo events và listeners bằng cách sử dụng các command console Artisan.

Events đóng vai trò là một cách tuyệt vời để decouple các khía cạnh khác nhau của ứng dụng của bạn, vì một event duy nhất có thể có nhiều listeners không phụ thuộc vào nhau. Ví dụ, bạn có thể muốn gửi một Slack notification cho người dùng của mình mỗi khi một đơn hàng đã được vận chuyển. Thay vì coupling code xử lý đơn hàng của bạn với code Slack notification, bạn có thể raise một event `App\Events\OrderShipped` mà một listener có thể nhận và sử dụng để dispatch một Slack notification.

<a name="generating-events-and-listeners"></a>
## Generating Events and Listeners

Để nhanh chóng tạo events và listeners, bạn có thể sử dụng các command Artisan `make:event` và `make:listener`:

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

Để thuận tiện, bạn cũng có thể gọi các command Artisan `make:event` và `make:listener` mà không cần thêm đối số. Khi bạn làm như vậy, Laravel sẽ tự động nhắc bạn nhập tên class và, khi tạo một listener, event mà nó nên lắng nghe:

```shell
php artisan make:event

php artisan make:listener
```

<a name="registering-events-and-listeners"></a>
## Registering Events and Listeners

<a name="event-discovery"></a>
### Event Discovery

Theo mặc định, Laravel sẽ tự động tìm và đăng ký event listeners của bạn bằng cách quét thư mục `Listeners` của ứng dụng. Khi Laravel tìm thấy bất kỳ method listener class nào bắt đầu bằng `handle` hoặc `__invoke`, Laravel sẽ đăng ký các methods đó như event listeners cho event được type-hint trong signature của method:

```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    /**
     * Handle the event.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```

Bạn có thể lắng nghe nhiều events bằng cách sử dụng union types của PHP:

```php
/**
 * Handle the event.
 */
public function handle(PodcastProcessed|PodcastPublished $event): void
{
    // ...
}
```

Nếu bạn định lưu trữ listeners của mình trong một thư mục khác hoặc trong nhiều thư mục, bạn có thể hướng dẫn Laravel quét các thư mục đó bằng cách sử dụng method `withEvents` trong file `bootstrap/app.php` của ứng dụng:

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```

Bạn có thể quét listeners trong nhiều thư mục tương tự bằng cách sử dụng ký tự `*` như một wildcard:

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/*/Listeners',
])
```

Command `event:list` có thể được sử dụng để liệt kê tất cả các listeners được đăng ký trong ứng dụng của bạn:

```shell
php artisan event:list
```

<a name="event-discovery-in-production"></a>
#### Event Discovery in Production

Để tăng tốc ứng dụng của bạn, bạn nên cache một manifest của tất cả listeners của ứng dụng bằng cách sử dụng các command Artisan `optimize` hoặc `event:cache`. Thông thường, command này nên được chạy như một phần của [deployment process](/docs/{{version}}/deployment#optimization) của ứng dụng. Manifest này sẽ được framework sử dụng để tăng tốc quá trình đăng ký event. Command `event:clear` có thể được sử dụng để xóa event cache.

<a name="dynamic-event-discovery"></a>
#### Dynamic Event Discovery

Để kiểm soát động xem một listener nhất định có được phát hiện hay không, bạn có thể implement interface `ShouldBeDiscovered` trên listener class và định nghĩa một method `shouldBeDiscovered` trả về một giá trị boolean. Nếu method trả về `false`, listener sẽ không được đăng ký trong quá trình event discovery:

```php
use Illuminate\Contracts\Events\ShouldBeDiscovered;

class SendPodcastNotification implements ShouldBeDiscovered
{
    /**
     * Handle the event.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }

    /**
     * Determine if the listener should be discovered.
     */
    public static function shouldBeDiscovered(): bool
    {
        return app()->environment('production');
    }
}
```

<a name="manually-registering-events"></a>
### Manually Registering Events

Sử dụng `Event` facade, bạn có thể đăng ký thủ công events và listeners tương ứng của chúng trong method `boot` của `AppServiceProvider` của ứng dụng:

```php
use App\Domain\Orders\Events\PodcastProcessed;
use App\Domain\Orders\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

Command `event:list` có thể được sử dụng để liệt kê tất cả các listeners được đăng ký trong ứng dụng của bạn:

```shell
php artisan event:list
```

<a name="closure-listeners"></a>
### Closure Listeners

Thông thường, listeners được định nghĩa như các classes; tuy nhiên, bạn cũng có thể đăng ký thủ công các event listeners dựa trên closure trong method `boot` của `AppServiceProvider` của ứng dụng:

```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // ...
    });
}
```

<a name="queueable-anonymous-event-listeners"></a>
#### Queueable Anonymous Event Listeners

Khi đăng ký các event listeners dựa trên closure, bạn có thể wrap listener closure trong function `Illuminate\Events\queueable` để hướng dẫn Laravel thực thi listener bằng cách sử dụng [queue](/docs/{{version}}/queues):

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    }));
}
```

Giống như queued jobs, bạn có thể sử dụng các methods `onConnection`, `onQueue`, và `delay` để tùy chỉnh việc thực thi của queued listener:

```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->plus(seconds: 10)));
```

Nếu bạn muốn xử lý các lỗi của anonymous queued listener, bạn có thể cung cấp một closure cho method `catch` khi định nghĩa `queueable` listener. Closure này sẽ nhận event instance và `Throwable` instance gây ra lỗi của listener:

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;
use Throwable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // The queued listener failed...
}));
```

<a name="wildcard-event-listeners"></a>
#### Wildcard Event Listeners

Bạn cũng có thể đăng ký listeners bằng cách sử dụng ký tự `*` như một tham số wildcard, cho phép bạn bắt nhiều events trên cùng một listener. Wildcard listeners nhận tên event làm đối số đầu tiên và toàn bộ mảng dữ liệu event làm đối số thứ hai:

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

<a name="defining-events"></a>
## Defining Events

Một event class về cơ bản là một data container chứa thông tin liên quan đến event. Ví dụ, giả sử một event `App\Events\OrderShipped` nhận một [Eloquent ORM](/docs/{{version}}/eloquent) object:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

Như bạn có thể thấy, event class này không chứa logic. Nó là một container cho instance `App\Models\Order` đã được mua. Trait `SerializesModels` được sử dụng bởi event sẽ serialize một cách graceful bất kỳ Eloquent models nào nếu event object được serialize bằng cách sử dụng function `serialize` của PHP, chẳng hạn như khi sử dụng [queued listeners](#queued-event-listeners).

<a name="defining-listeners"></a>
## Defining Listeners

Tiếp theo, hãy xem xét listener cho event ví dụ của chúng ta. Event listeners nhận event instances trong method `handle` của chúng. Command Artisan `make:listener`, khi được gọi với tùy chọn `--event`, sẽ tự động import event class thích hợp và type-hint event trong method `handle`. Trong method `handle`, bạn có thể thực hiện bất kỳ hành động nào cần thiết để phản hồi với event:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     */
    public function __construct() {}

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Access the order using $event->order...
    }
}
```

> [!NOTE]
> Event listeners của bạn cũng có thể type-hint bất kỳ dependencies nào chúng cần trên constructors của chúng. Tất cả event listeners đều được giải quyết qua Laravel [service container](/docs/{{version}}/container), vì vậy dependencies sẽ được inject tự động.

<a name="stopping-the-propagation-of-an-event"></a>
#### Stopping The Propagation Of An Event

Đôi khi, bạn có thể muốn ngăn chặn sự lan truyền của một event đến các listeners khác. Bạn có thể làm như vậy bằng cách trả về `false` từ method `handle` của listener.

<a name="queued-event-listeners"></a>
## Queued Event Listeners

Queueing listeners có thể hữu ích nếu listener của bạn sẽ thực hiện một nhiệm vụ chậm như gửi email hoặc thực hiện HTTP request. Trước khi sử dụng queued listeners, hãy đảm bảo [configure your queue](/docs/{{version}}/queues) và start một queue worker trên server hoặc môi trường phát triển local của bạn.

Để chỉ định rằng một listener nên được queued, thêm interface `ShouldQueue` vào listener class. Listeners được tạo bởi các command Artisan `make:listener` đã có interface này được import vào namespace hiện tại để bạn có thể sử dụng nó ngay lập tức:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

Đó là tất cả! Bây giờ, khi một event được xử lý bởi listener này được dispatch, listener sẽ tự động được queued bởi event dispatcher bằng cách sử dụng [queue system](/docs/{{version}}/queues) của Laravel. Nếu không có exceptions nào được ném khi listener được thực thi bởi queue, queued job sẽ tự động được xóa sau khi nó hoàn thành xử lý.

<a name="customizing-the-queue-connection-queue-name"></a>
#### Customizing The Queue Connection, Name, & Delay

Nếu bạn muốn tùy chỉnh queue connection, queue name, hoặc queue delay time của một event listener, bạn có thể sử dụng các attributes `Connection`, `Queue`, và `Delay` trên listener class của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Connection;
use Illuminate\Queue\Attributes\Delay;
use Illuminate\Queue\Attributes\Queue;

#[Connection('sqs')]
#[Queue('listeners')]
#[Delay(60)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```
Nếu bạn muốn định nghĩa queue connection, queue name, hoặc delay của listener tại runtime, bạn có thể định nghĩa các methods `viaConnection`, `viaQueue`, hoặc `withDelay` trên listener:

```php
/**
 * Get the name of the listener's queue connection.
 */
public function viaConnection(): string
{
    return 'sqs';
}

/**
 * Get the name of the listener's queue.
 */
public function viaQueue(): string
{
    return 'listeners';
}

/**
 * Get the number of seconds before the job should be processed.
 */
public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

<a name="conditionally-queueing-listeners"></a>
#### Conditionally Queueing Listeners

Đôi khi, bạn cần xác định xem một listener có nên được queued dựa trên một số dữ liệu chỉ có sẵn tại runtime hay không. Để thực hiện điều này, một method `shouldQueue` có thể được thêm vào một listener để xác định xem listener có nên được queued hay không. Nếu method `shouldQueue` trả về `false`, listener sẽ không được queued:

```php
<?php

namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

class RewardGiftCard implements ShouldQueue
{
    /**
     * Reward a gift card to the customer.
     */
    public function handle(OrderCreated $event): void
    {
        // ...
    }

    /**
     * Determine whether the listener should be queued.
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

<a name="manually-interacting-with-the-queue"></a>
### Manually Interacting With the Queue

Nếu bạn cần truy cập thủ công các methods `delete` và `release` của queue job bên dưới của listener, bạn có thể làm như vậy bằng cách sử dụng trait `Illuminate\Queue\InteractsWithQueue`. Trait này được import theo mặc định trên các listeners được tạo và cung cấp quyền truy cập vào các methods này:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        if ($condition) {
            $this->release(30);
        }
    }
}
```

<a name="queued-event-listeners-and-database-transactions"></a>
### Queued Event Listeners and Database Transactions

Khi queued listeners được dispatch trong database transactions, chúng có thể được xử lý bởi queue trước khi database transaction đã commit. Khi điều này xảy ra, bất kỳ cập nhật nào bạn đã thực hiện cho models hoặc database records trong database transaction có thể chưa được phản ánh trong database. Ngoài ra, bất kỳ models hoặc database records nào được tạo trong transaction có thể không tồn tại trong database. Nếu listener của bạn phụ thuộc vào các models này, các lỗi không mong muốn có thể xảy ra khi job dispatches queued listener được xử lý.

Nếu tùy chọn cấu hình `after_commit` của queue connection của bạn được đặt thành `false`, bạn vẫn có thể chỉ định rằng một queued listener cụ thể nên được dispatch sau khi tất cả các database transactions mở đã được commit bằng cách implement interface `ShouldQueueAfterCommit` trên listener class:

```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueueAfterCommit
{
    use InteractsWithQueue;
}
```

> [!NOTE]
> Để tìm hiểu thêm về cách giải quyết các vấn đề này, hãy xem lại tài liệu về [queued jobs và database transactions](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="queued-listener-middleware"></a>
### Queued Listener Middleware

Queued listeners cũng có thể sử dụng [job middleware](/docs/{{version}}/queues#job-middleware). Job middleware cho phép bạn wrap custom logic xung quanh việc thực thi của queued listeners, giảm boilerplate trong chính các listeners. Sau khi tạo job middleware, chúng có thể được gắn vào một listener bằng cách trả về chúng từ method `middleware` của listener:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use App\Jobs\Middleware\RateLimited;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Process the event...
    }

    /**
     * Get the middleware the listener should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(OrderShipped $event): array
    {
        return [new RateLimited];
    }
}
```

<a name="encrypted-queued-listeners"></a>
#### Encrypted Queued Listeners

Laravel cho phép bạn đảm bảo quyền riêng tư và tính toàn vẹn của dữ liệu của queued listener thông qua [encryption](/docs/{{version}}/encryption). Để bắt đầu, chỉ cần thêm interface `ShouldBeEncrypted` vào listener class. Khi interface này đã được thêm vào class, Laravel sẽ tự động encrypt listener của bạn trước khi đẩy nó vào queue:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="unique-event-listeners"></a>
### Unique Event Listeners

> [!WARNING]
> Unique listeners yêu cầu một cache driver hỗ trợ [locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, các cache drivers `memcached`, `redis`, `dynamodb`, `database`, `file`, và `array` hỗ trợ atomic locks.

Đôi khi, bạn muốn đảm bảo rằng chỉ có một instance của một listener cụ thể đang có trên queue tại bất kỳ thời điểm nào. Bạn có thể làm như vậy bằng cách implement interface `ShouldBeUnique` trên listener class của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    public function __invoke(LicenseSaved $event): void
    {
        // ...
    }
}
```

Trong ví dụ trên, listener `AcquireProductKey` là unique. Vì vậy, listener sẽ không được queued nếu một instance khác của listener đã có trên queue và chưa hoàn thành xử lý. Điều này đảm bảo rằng chỉ có một product key được thu được cho mỗi license, ngay cả khi license được lưu nhiều lần liên tiếp.

Trong một số trường hợp, bạn muốn định nghĩa một "key" cụ thể làm cho listener trở nên unique hoặc bạn muốn chỉ định một timeout sau đó listener không còn giữ unique. Để thực hiện điều này, bạn có thể định nghĩa các properties hoặc methods `uniqueId` và `uniqueFor` trên listener class của bạn. Các methods nhận event instance, cho phép bạn sử dụng event data để xây dựng giá trị trả về:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    /**
     * The number of seconds after which the listener's unique lock will be released.
     *
     * @var int
     */
    public $uniqueFor = 3600;

    public function __invoke(LicenseSaved $event): void
    {
        // ...
    }

    /**
     * Get the unique ID for the listener.
     */
    public function uniqueId(LicenseSaved $event): string
    {
        return 'listener:'.$event->license->id;
    }
}
```

Trong ví dụ trên, listener `AcquireProductKey` là unique theo license ID. Vì vậy, bất kỳ dispatches mới của listener cho cùng một license sẽ bị bỏ qua cho đến khi listener hiện tại đã hoàn thành xử lý. Điều này ngăn chặn việc thu được các product key trùng lặp cho cùng một license. Ngoài ra, nếu listener hiện tại không được xử lý trong vòng một giờ, unique lock sẽ được released và một listener khác với cùng unique key có thể được queued.

> [!WARNING]
> Nếu ứng dụng của bạn dispatch events từ nhiều web servers hoặc containers, bạn nên đảm bảo rằng tất cả các servers của bạn đang giao tiếp với cùng một central cache server để Laravel có thể xác định chính xác xem một listener có unique hay không.

<a name="keeping-listeners-unique-until-processing-begins"></a>
#### Keeping Listeners Unique Until Processing Begins

Theo mặc định, unique listeners được "unlocked" sau khi listener hoàn thành xử lý hoặc fails tất cả các lần thử lại của nó. Tuy nhiên, có thể có những tình huống mà bạn muốn listener của mình unlock ngay lập tức trước khi nó được xử lý. Để thực hiện điều này, listener của bạn nên implement contract `ShouldBeUniqueUntilProcessing` thay vì contract `ShouldBeUnique`:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-listener-locks"></a>
#### Unique Listener Locks

Behind the scenes, khi một listener `ShouldBeUnique` được dispatch, Laravel cố gắng thu được một [lock](/docs/{{version}}/cache#atomic-locks) với key `uniqueId`. Nếu lock đã được giữ, listener sẽ không được dispatch. Lock này được released khi listener hoàn thành xử lý hoặc fails tất cả các lần thử lại của nó. Theo mặc định, Laravel sẽ sử dụng default cache driver để thu được lock này. Tuy nhiên, nếu bạn muốn sử dụng driver khác để thu được lock, bạn có thể định nghĩa một method `uniqueVia` trả về cache driver nên được sử dụng:

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    // ...

    /**
     * Get the cache driver for the unique listener lock.
     */
    public function uniqueVia(LicenseSaved $event): Repository
    {
        return Cache::driver('redis');
    }
}
```

> [!NOTE]
> Nếu bạn chỉ cần giới hạn xử lý đồng thời của một listener, hãy sử dụng [WithoutOverlapping](/docs/{{version}}/queues#preventing-job-overlaps) job middleware thay thế.

<a name="handling-failed-jobs"></a>
### Handling Failed Jobs

Đôi khi queued event listeners của bạn có thể fail. Nếu queued listener vượt quá số lần thử tối đa như được định nghĩa bởi queue worker của bạn, method `failed` sẽ được gọi trên listener của bạn. Method `failed` nhận event instance và `Throwable` gây ra lỗi:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // ...
    }

    /**
     * Handle a job failure.
     */
    public function failed(OrderShipped $event, Throwable $exception): void
    {
        // ...
    }
}
```

<a name="specifying-queued-listener-maximum-attempts"></a>
#### Specifying Queued Listener Maximum Attempts

Nếu một trong các queued listeners của bạn gặp lỗi, bạn có thể không muốn nó tiếp tục thử lại vô thời hạn. Do đó, Laravel cung cấp nhiều cách để chỉ định bao nhiêu lần hoặc trong bao lâu một listener có thể được thử.

Bạn có thể sử dụng attribute `Tries` trên listener class của bạn để chỉ định bao nhiêu lần listener có thể được thử trước khi được coi là đã fail:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;

#[Tries(5)]
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    // ...
}
```

Là một thay thế cho việc định nghĩa bao nhiêu lần một listener có thể được thử trước khi nó fails, bạn có thể định nghĩa một thời điểm mà listener không còn được thử nữa. Điều này cho phép một listener được thử bất kỳ số lần nào trong một khung thời gian nhất định. Để định nghĩa thời điểm mà listener không còn được thử nữa, hãy thêm một method `retryUntil` vào listener class của bạn. Method này nên trả về một instance `DateTimeInterface`:

```php
use DateTimeInterface;

/**
 * Determine the time at which the listener should timeout.
 */
public function retryUntil(): DateTimeInterface
{
    return now()->plus(minutes: 5);
}
```

Nếu cả `retryUntil` và `tries` đều được định nghĩa, Laravel sẽ ưu tiên method `retryUntil`.

<a name="specifying-queued-listener-backoff"></a>
#### Specifying Queued Listener Backoff

Nếu bạn muốn cấu hình bao nhiêu giây Laravel nên đợi trước khi thử lại một listener đã gặp exception, bạn có thể sử dụng attribute `Backoff` trên listener class của bạn:

```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Backoff;

#[Backoff(3)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

Nếu bạn cần logic phức tạp hơn để xác định thời gian backoff của listeners, bạn có thể định nghĩa một method `backoff` trên listener class của bạn:

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 */
public function backoff(OrderShipped $event): int
{
    return 3;
}
```

Bạn có thể dễ dàng cấu hình "exponential" backoffs bằng cách trả về một mảng các giá trị backoff từ method `backoff`. Trong ví dụ này, độ trễ thử lại sẽ là 1 giây cho lần thử lại đầu tiên, 5 giây cho lần thử lại thứ hai, 10 giây cho lần thử lại thứ ba, và 10 giây cho mọi lần thử lại tiếp theo nếu còn nhiều lần thử lại còn lại:

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 *
 * @return list<int>
 */
public function backoff(OrderShipped $event): array
{
    return [1, 5, 10];
}
```

<a name="specifying-queued-listener-max-exceptions"></a>
#### Specifying Queued Listener Max Exceptions

Đôi khi bạn có thể muốn chỉ định rằng một queued listener có thể được thử nhiều lần, nhưng nên fail nếu các lần thử lại được kích hoạt bởi một số lượng unhandled exceptions nhất định (trái ngược với việc được release bởi method `release` trực tiếp). Để thực hiện điều này, bạn có thể sử dụng các attributes `Tries` và `MaxExceptions` trên listener class của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;

#[Tries(25)]
#[MaxExceptions(3)]
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Process the event...
    }
}
```

Trong ví dụ này, listener sẽ được thử lại tối đa 25 lần. Tuy nhiên, listener sẽ fail nếu ba unhandled exceptions được ném bởi listener.

<a name="specifying-queued-listener-timeout"></a>
#### Specifying Queued Listener Timeout

Thường thì bạn biết khoảng bao lâu bạn mong đợi queued listeners của mình sẽ mất. Vì lý do này, Laravel cho phép bạn chỉ định một giá trị "timeout". Nếu một listener đang xử lý lâu hơn số giây được chỉ định bởi giá trị timeout, worker xử lý listener sẽ exit với một lỗi. Bạn có thể định nghĩa số giây tối đa mà một listener nên được phép chạy bằng cách sử dụng attribute `Timeout` trên listener class của bạn:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Timeout;

#[Timeout(120)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

Nếu bạn muốn chỉ định rằng một listener nên được đánh dấu là failed khi timeout, bạn có thể sử dụng attribute `FailOnTimeout` trên listener class:

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\FailOnTimeout;

#[FailOnTimeout]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

<a name="dispatching-events"></a>
## Dispatching Events

Để dispatch một event, bạn có thể gọi static method `dispatch` trên event. Method này được cung cấp trên event bởi trait `Illuminate\Foundation\Events\Dispatchable`. Bất kỳ đối số nào được chuyển đến method `dispatch` sẽ được chuyển đến constructor của event:

```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Order shipment logic...

        OrderShipped::dispatch($order);

        return redirect('/orders');
    }
}
```

Nếu bạn muốn dispatch một event có điều kiện, bạn có thể sử dụng các methods `dispatchIf` và `dispatchUnless`:

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]
> Khi testing, có thể hữu ích để assert rằng các events nhất định đã được dispatch mà không thực sự kích hoạt listeners của chúng. [built-in testing helpers](#testing) của Laravel làm cho điều này trở nên dễ dàng.

<a name="dispatching-events-after-database-transactions"></a>
### Dispatching Events After Database Transactions

Đôi khi, bạn muốn hướng dẫn Laravel chỉ dispatch một event sau khi active database transaction đã commit. Để làm như vậy, bạn có thể implement interface `ShouldDispatchAfterCommit` trên event class.

Interface này hướng dẫn Laravel không dispatch event cho đến khi database transaction hiện tại được commit. Nếu transaction fails, event sẽ bị loại bỏ. Nếu không có database transaction nào đang tiến hành khi event được dispatch, event sẽ được dispatch ngay lập tức:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

<a name="deferring-events"></a>
### Deferring Events

Deferred events cho phép bạn trì hoãn việc dispatching của model events và thực thi của event listeners cho đến sau khi một block code cụ thể đã hoàn thành. Điều này đặc biệt hữu ích khi bạn cần đảm bảo rằng tất cả các records liên quan được tạo trước khi event listeners được kích hoạt.

Để defer events, hãy cung cấp một closure cho method `Event::defer()`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
});
```

Tất cả events được kích hoạt trong closure sẽ được dispatch sau khi closure được thực thi. Điều này đảm bảo rằng event listeners có quyền truy cập vào tất cả các records liên quan được tạo trong quá trình thực thi deferred. Nếu một exception xảy ra trong closure, các deferred events sẽ không được dispatch.

Để defer chỉ các events cụ thể, hãy chuyển một mảng các events làm đối số thứ hai cho method `defer`:

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
}, ['eloquent.created: '.User::class]);
```

<a name="event-subscribers"></a>
## Event Subscribers

<a name="writing-event-subscribers"></a>
### Writing Event Subscribers

Event subscribers là các classes có thể subscribe đến nhiều events từ trong chính subscriber class, cho phép bạn định nghĩa nhiều event handlers trong một class duy nhất. Subscribers nên định nghĩa một method `subscribe`, nhận một event dispatcher instance. Bạn có thể gọi method `listen` trên dispatcher được cung cấp để đăng ký event listeners:

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Register the listeners for the subscriber.
     */
    public function subscribe(Dispatcher $events): void
    {
        $events->listen(
            Login::class,
            [UserEventSubscriber::class, 'handleUserLogin']
        );

        $events->listen(
            Logout::class,
            [UserEventSubscriber::class, 'handleUserLogout']
        );
    }
}
```

Nếu các method event listener của bạn được định nghĩa trong chính subscriber, bạn có thể thấy thuận tiện hơn khi trả về một mảng các events và tên methods từ method `subscribe` của subscriber. Laravel sẽ tự động xác định tên class của subscriber khi đăng ký event listeners:

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Register the listeners for the subscriber.
     *
     * @return array<string, string>
     */
    public function subscribe(Dispatcher $events): array
    {
        return [
            Login::class => 'handleUserLogin',
            Logout::class => 'handleUserLogout',
        ];
    }
}
```

<a name="registering-event-subscribers"></a>
### Registering Event Subscribers

Sau khi viết subscriber, Laravel sẽ tự động đăng ký các handler methods trong subscriber nếu chúng tuân theo [event discovery conventions](#event-discovery) của Laravel. Nếu không, bạn có thể đăng ký thủ công subscriber của mình bằng cách sử dụng method `subscribe` của `Event` facade. Thông thường, điều này nên được thực hiện trong method `boot` của `AppServiceProvider` của ứng dụng:

```php
<?php

namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::subscribe(UserEventSubscriber::class);
    }
}
```

<a name="testing"></a>
## Testing

Khi testing code dispatches events, bạn có thể muốn hướng dẫn Laravel không thực sự thực thi listeners của event, vì code của listener có thể được test trực tiếp và riêng biệt với code dispatches event tương ứng. Tất nhiên, để test chính listener đó, bạn có thể instantiate một listener instance và gọi method `handle` trực tiếp trong test của bạn.

Sử dụng method `fake` của `Event` facade, bạn có thể ngăn chặn listeners thực thi, thực thi code under test, và sau đó assert các events đã được dispatch bởi ứng dụng của bạn bằng cách sử dụng các methods `assertDispatched`, `assertNotDispatched`, và `assertNothingDispatched`:

```php tab=Pest
<?php

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;

test('orders can be shipped', function () {
    Event::fake();

    // Perform order shipping...

    // Assert that an event was dispatched...
    Event::assertDispatched(OrderShipped::class);

    // Assert an event was dispatched twice...
    Event::assertDispatched(OrderShipped::class, 2);

    // Assert an event was dispatched once...
    Event::assertDispatchedOnce(OrderShipped::class);

    // Assert an event was not dispatched...
    Event::assertNotDispatched(OrderFailedToShip::class);

    // Assert that no events were dispatched...
    Event::assertNothingDispatched();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order shipping.
     */
    public function test_orders_can_be_shipped(): void
    {
        Event::fake();

        // Perform order shipping...

        // Assert that an event was dispatched...
        Event::assertDispatched(OrderShipped::class);

        // Assert an event was dispatched twice...
        Event::assertDispatched(OrderShipped::class, 2);

        // Assert an event was dispatched once...
        Event::assertDispatchedOnce(OrderShipped::class);

        // Assert an event was not dispatched...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // Assert that no events were dispatched...
        Event::assertNothingDispatched();
    }
}
```

Bạn có thể chuyển một closure cho các methods `assertDispatched` hoặc `assertNotDispatched` để assert rằng một event đã được dispatch vượt qua một "truth test" nhất định. Nếu ít nhất một event đã được dispatch vượt qua truth test đã cho thì assertion sẽ thành công:

```php
Event::assertDispatched(function (OrderShipped $event) use ($order) {
    return $event->order->id === $order->id;
});
```

Nếu bạn chỉ muốn assert rằng một event listener đang lắng nghe một event nhất định, bạn có thể sử dụng method `assertListening`:

```php
Event::assertListening(
    OrderShipped::class,
    SendShipmentNotification::class
);
```

> [!WARNING]
> Sau khi gọi `Event::fake()`, không có event listeners nào sẽ được thực thi. Vì vậy, nếu tests của bạn sử dụng model factories phụ thuộc vào events, chẳng hạn như tạo một UUID trong event `creating` của một model, bạn nên gọi `Event::fake()` **sau** khi sử dụng factories của bạn.

<a name="faking-a-subset-of-events"></a>
### Faking a Subset of Events

Nếu bạn chỉ muốn fake event listeners cho một tập hợp events cụ thể, bạn có thể chuyển chúng cho method `fake` hoặc `fakeFor`:

```php tab=Pest
test('orders can be processed', function () {
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Other events are dispatched as normal...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
/**
 * Test order process.
 */
public function test_orders_can_be_processed(): void
{
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Other events are dispatched as normal...
    $order->update([
        // ...
    ]);
}
```

Bạn có thể fake tất cả events ngoại trừ một tập hợp các events được chỉ định bằng cách sử dụng method `except`:

```php
Event::fake()->except([
    OrderCreated::class,
]);
```

<a name="scoped-event-fakes"></a>
### Scoped Event Fakes

Nếu bạn chỉ muốn fake event listeners cho một phần của test, bạn có thể sử dụng method `fakeFor`:

```php tab=Pest
<?php

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;

test('orders can be processed', function () {
    $order = Event::fakeFor(function () {
        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        return $order;
    });

    // Events are dispatched as normal and observers will run...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order process.
     */
    public function test_orders_can_be_processed(): void
    {
        $order = Event::fakeFor(function () {
            $order = Order::factory()->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // Events are dispatched as normal and observers will run...
        $order->update([
            // ...
        ]);
    }
}
```

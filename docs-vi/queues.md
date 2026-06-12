# Hàng đợi

- [Giới thiệu](#introduction)
    - [Kết nối vs. Hàng đợi](#connections-vs-queues)
    - [Ghi chú Driver và Điều kiện tiên quyết](#driver-prerequisites)
- [Tạo Jobs](#creating-jobs)
    - [Tạo Lớp Job](#generating-job-classes)
    - [Cấu trúc Lớp](#class-structure)
    - [Job Độc nhất](#unique-jobs)
    - [Job Debounce](#debounced-jobs)
    - [Job Được Mã hóa](#encrypted-jobs)
- [Middleware Job](#job-middleware)
    - [Giới hạn Tỷ lệ](#rate-limiting)
    - [Ngăn Chặn Ghi đè Job](#preventing-job-overlaps)
    - [Giới hạn Exception](#throttling-exceptions)
    - [Bỏ qua Jobs](#skipping-jobs)
- [Dispatch Jobs](#dispatching-jobs)
    - [Dispatch Trì hoãn](#delayed-dispatching)
    - [Dispatch Đồng bộ](#synchronous-dispatching)
    - [Dispatch Hàng loạt](#bulk-dispatching)
    - [Chuẩn bị Jobs Trước khi Dispatch](#preparing-jobs-before-dispatch)
    - [Jobs & Giao dịch Database](#jobs-and-database-transactions)
    - [Chuỗi Job](#job-chaining)
    - [Tùy chỉnh Hàng đợi và Kết nối](#customizing-the-queue-and-connection)
    - [Chỉ định Số lần Thử Tối đa / Giá trị Timeout](#max-job-attempts-and-timeout)
    - [SQS FIFO và Hàng đợi Công bằng](#sqs-fifo-and-fair-queues)
    - [Failover Hàng đợi](#queue-failover)
    - [Xử lý Lỗi](#error-handling)
- [Batching Jobs](#job-batching)
    - [Định nghĩa Jobs Có thể Batch](#defining-batchable-jobs)
    - [Dispatch Batches](#dispatching-batches)
    - [Chuỗi và Batches](#chains-and-batches)
    - [Thêm Jobs vào Batches](#adding-jobs-to-batches)
    - [Kiểm tra Batches](#inspecting-batches)
    - [Hủy Batches](#cancelling-batches)
    - [Lỗi Batch](#batch-failures)
    - [Dọn dẹp Batches](#pruning-batches)
    - [Lưu trữ Batches trong DynamoDB](#storing-batches-in-dynamodb)
- [Đặt Closures vào Hàng đợi](#queueing-closures)
- [Chạy Queue Worker](#running-the-queue-worker)
    - [Lệnh `queue:work`](#the-queue-work-command)
    - [Ưu tiên Hàng đợi](#queue-priorities)
    - [Queue Workers và Triển khai](#queue-workers-and-deployment)
    - [Phản hồi với Tín hiệu Worker](#reacting-to-worker-signals)
    - [Hết hạn và Timeout của Job](#job-expirations-and-timeouts)
    - [Tạm dừng và Tiếp tục Queue Workers](#pausing-and-resuming-queue-workers)
- [Cấu hình Supervisor](#supervisor-configuration)
- [Xử lý với Jobs Thất bại](#dealing-with-failed-jobs)
    - [Dọn dẹp Sau khi Jobs Thất bại](#cleaning-up-after-failed-jobs)
    - [Thử lại Jobs Thất bại](#retrying-failed-jobs)
    - [Bỏ qua Models Thiếu](#ignoring-missing-models)
    - [Dọn dẹp Jobs Thất bại](#pruning-failed-jobs)
    - [Lưu trữ Jobs Thất bại trong DynamoDB](#storing-failed-jobs-in-dynamodb)
    - [Vô hiệu hóa Lưu trữ Job Thất bại](#disabling-failed-job-storage)
    - [Sự kiện Job Thất bại](#failed-job-events)
- [Xóa Jobs khỏi Hàng đợi](#clearing-jobs-from-queues)
- [Giám sát Hàng đợi của Bạn](#monitoring-your-queues)
- [Kiểm thử](#testing)
    - [Giả lập Một Tập con Jobs](#faking-a-subset-of-jobs)
    - [Kiểm thử Chuỗi Job](#testing-job-chains)
    - [Kiểm thử Batch Job](#testing-job-batches)
    - [Kiểm thử Tương tác Job / Hàng đợi](#testing-job-queue-interactions)
- [Sự kiện Job](#job-events)

<a name="introduction"></a>
## Giới thiệu

Khi xây dựng ứng dụng web của bạn, bạn có thể có một số tác vụ, chẳng hạn như phân tích và lưu trữ tệp CSV đã tải lên, mất quá nhiều thời gian để thực hiện trong một yêu cầu web điển hình. May mắn thay, Laravel cho phép bạn dễ dàng tạo các jobs được xếp hàng đợi có thể được xử lý trong nền. Bằng cách chuyển các tác vụ tốn thời gian sang hàng đợi, ứng dụng của bạn có thể phản hồi các yêu cầu web với tốc độ cực nhanh và cung cấp trải nghiệm người dùng tốt hơn cho khách hàng của bạn.

Hàng đợi Laravel cung cấp một API hàng đợi thống nhất trên nhiều backend hàng đợi khác nhau, chẳng hạn như [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), hoặc thậm chí là cơ sở dữ liệu quan hệ.

Các tùy chọn cấu hình hàng đợi của Laravel được lưu trữ trong tệp cấu hình `config/queue.php` của ứng dụng của bạn. Trong tệp này, bạn sẽ tìm thấy cấu hình kết nối cho từng driver hàng đợi được bao gồm trong framework, bao gồm các driver database, [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), và [Beanstalkd](https://beanstalkd.github.io/), cũng như một driver đồng bộ sẽ thực thi jobs ngay lập tức (để sử dụng trong quá trình phát triển hoặc kiểm thử). Một driver hàng đợi `null` cũng được bao gồm sẽ loại bỏ các jobs được xếp hàng đợi.

> [!NOTE]
> Laravel Horizon là một bảng điều khiển và hệ thống cấu hình đẹp mắt cho các hàng đợi chạy bằng Redis của bạn. Hãy xem tài liệu [Horizon](/docs/{{version}}/horizon) đầy đủ để biết thêm thông tin.

<a name="connections-vs-queues"></a>
### Kết nối vs. Hàng đợi

Trước khi bắt đầu với hàng đợi Laravel, điều quan trọng là phải hiểu sự khác biệt giữa "kết nối" và "hàng đợi". Trong tệp cấu hình `config/queue.php` của bạn, có một mảng cấu hình `connections`. Tùy chọn này định nghĩa các kết nối với các dịch vụ hàng đợi backend như Amazon SQS, Beanstalk, hoặc Redis. Tuy nhiên, bất kỳ kết nối hàng đợi nào có thể có nhiều "hàng đợi" có thể được coi là các ngăn xếp hoặc đống jobs được xếp hàng đợi khác nhau.

Lưu ý rằng mỗi ví dụ cấu hình kết nối trong tệp cấu hình `queue` chứa một thuộc tính `queue`. Đây là hàng đợi mặc định mà jobs sẽ được dispatch đến khi chúng được gửi đến một kết nối nhất định. Nói cách khác, nếu bạn dispatch một job mà không xác định rõ hàng đợi nào nó nên được dispatch đến, job sẽ được đặt trên hàng đợi được định nghĩa trong thuộc tính `queue` của cấu hình kết nối:

```php
use App\Jobs\ProcessPodcast;

// Job này được gửi đến hàng đợi mặc định của kết nối mặc định...
ProcessPodcast::dispatch();

// Job này được gửi đến hàng đợi "emails" của kết nối mặc định...
ProcessPodcast::dispatch()->onQueue('emails');
```

Một số ứng dụng có thể không cần bao giờ đẩy jobs lên nhiều hàng đợi, thay vào đó thích có một hàng đợi đơn giản. Tuy nhiên, đẩy jobs đến nhiều hàng đợi có thể đặc biệt hữu ích cho các ứng dụng muốn ưu tiên hoặc phân đoạn cách jobs được xử lý, vì queue worker của Laravel cho phép bạn chỉ định các hàng đợi mà nó nên xử lý theo mức độ ưu tiên. Ví dụ, nếu bạn đẩy jobs đến một hàng đợi `high`, bạn có thể chạy một worker cung cấp cho chúng mức độ ưu tiên xử lý cao hơn:

```shell
php artisan queue:work --queue=high,default
```

<a name="driver-prerequisites"></a>
### Ghi chú Driver và Điều kiện tiên quyết

<a name="database"></a>
#### Database

Để sử dụng driver hàng đợi `database`, bạn sẽ cần một bảng database để giữ các jobs. Thông thường, điều này được bao gồm trong migration database mặc định `0001_01_01_000002_create_jobs_table.php` của Laravel; tuy nhiên, nếu ứng dụng của bạn không chứa migration này, bạn có thể sử dụng lệnh Artisan `make:queue-table` để tạo nó:

```shell
php artisan make:queue-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Để sử dụng driver hàng đợi `redis`, bạn nên cấu hình một kết nối database Redis trong tệp cấu hình `config/database.php` của bạn.

> [!WARNING]
> Các tùy chọn Redis `serializer` và `compression` không được hỗ trợ bởi driver hàng đợi `redis`.

<a name="redis-cluster"></a>
##### Redis Cluster

Nếu kết nối hàng đợi Redis của bạn sử dụng [Redis Cluster](https://redis.io/docs/latest/operate/rs/databases/durability-ha/clustering), tên hàng đợi của bạn phải chứa một [key hash tag](https://redis.io/docs/latest/develop/using-commands/keyspace/#hashtags). Điều này được yêu cầu để đảm bảo tất cả các Redis keys cho một hàng đợi nhất định được đặt vào cùng một hash slot:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', '{default}'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => null,
    'after_commit' => false,
],
```

<a name="blocking"></a>
##### Blocking

Khi sử dụng hàng đợi Redis, bạn có thể sử dụng tùy chọn cấu hình `block_for` để chỉ định driver nên chờ bao lâu để một job trở nên khả dụng trước khi lặp qua vòng lặp worker và thăm dò lại database Redis.

Điều chỉnh giá trị này dựa trên tải hàng đợi của bạn có thể hiệu quả hơn so với việc liên tục thăm dò database Redis để tìm các jobs mới. Ví dụ, bạn có thể đặt giá trị thành `5` để chỉ ra rằng driver nên chặn trong năm giây trong khi chờ một job trở nên khả dụng:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => 5,
    'after_commit' => false,
],
```

> [!WARNING]
> Đặt `block_for` thành `0` sẽ khiến queue workers chặn vô thời hạn cho đến khi một job khả dụng. Điều này cũng sẽ ngăn các tín hiệu như `SIGTERM` được xử lý cho đến khi job tiếp theo đã được xử lý.

<a name="sqs-overflow-storage"></a>
#### Lưu trữ Overflow SQS

Amazon SQS giới hạn kích thước tối đa của payload tin nhắn được xếp hàng đợi. Nếu bạn cần dispatch các jobs với payloads có thể vượt quá giới hạn này, bạn có thể cấu hình Laravel để lưu trữ các payloads SQS quá lớn trong một cache store và gửi một con trỏ qua SQS thay thế. Để bật tính năng này, thêm một mảng `overflow` vào cấu hình kết nối hàng đợi SQS của bạn:

```php
'sqs' => [
    'driver' => 'sqs',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'prefix' => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
    'queue' => env('SQS_QUEUE', 'default'),
    'suffix' => env('SQS_SUFFIX'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'after_commit' => false,
    'overflow' => [
        'enabled' => env('SQS_OVERFLOW_ENABLED', false),
        'store' => env('SQS_OVERFLOW_STORE'),
        'always' => false,
        'delete_after_processing' => true,
        'flush_on_clear' => env('SQS_OVERFLOW_FLUSH_ON_CLEAR', false),
    ],
],
```

Khi lưu trữ overflow được bật, Laravel sẽ lưu trữ các payloads có kích thước ít nhất 1 MB trong cache store được cấu hình. Nếu tùy chọn `always` là `true`, mọi payload SQS sẽ được lưu trữ trong cache store bất kể kích thước của nó. Vì các jobs được xếp hàng đợi sẽ cần truy xuất payloads của chúng từ cache store khi chúng được xử lý, bạn nên chọn một store có thể giữ các payloads cho đến khi workers của bạn xử lý chúng. Theo mặc định, các payloads được lưu trữ sẽ bị xóa sau khi jobs của chúng đã được xử lý thành công và bị xóa khỏi SQS.

Nếu tùy chọn `flush_on_clear` là `true`, cache store overflow được cấu hình sẽ được xóa khi lệnh `queue:clear` xóa hàng đợi SQS. Vì việc xóa một cache store có thể xóa tất cả các mục khỏi store đó, bạn nên cấu hình lưu trữ overflow SQS để sử dụng một cache store chuyên dụng khi bật tùy chọn này.

<a name="other-driver-prerequisites"></a>
#### Điều kiện tiên quyết Driver Khác

Các phụ thuộc sau đây được cần thiết cho các driver hàng đợi được liệt kê. Các phụ thuộc này có thể được cài đặt thông qua trình quản lý gói Composer:

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~5.0`
- Redis: `predis/predis ~2.0` hoặc extension PHP phpredis
- [MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/): `mongodb/laravel-mongodb`

</div>

<a name="creating-jobs"></a>
## Tạo Jobs

<a name="generating-job-classes"></a>
### Tạo Lớp Job

Theo mặc định, tất cả các jobs có thể xếp hàng đợi cho ứng dụng của bạn được lưu trữ trong thư mục `app/Jobs`. Nếu thư mục `app/Jobs` không tồn tại, nó sẽ được tạo khi bạn chạy lệnh Artisan `make:job`:

```shell
php artisan make:job ProcessPodcast
```

Lớp được tạo sẽ triển khai interface `Illuminate\Contracts\Queue\ShouldQueue`, chỉ ra cho Laravel rằng job nên được đẩy lên hàng đợi để chạy không đồng bộ.

> [!NOTE]
> Job stubs có thể được tùy chỉnh bằng cách sử dụng [stub publishing](/docs/{{version}}/artisan#stub-customization).

<a name="class-structure"></a>
### Cấu trúc Lớp

Các lớp job rất đơn giản, thường chỉ chứa một phương thức `handle` được gọi khi job được xử lý bởi hàng đợi. Để bắt đầu, hãy xem một lớp job ví dụ. Trong ví dụ này, chúng ta sẽ giả sử chúng ta quản lý một dịch vụ xuất bản podcast và cần xử lý các tệp podcast đã tải lên trước khi chúng được xuất bản:

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }
}
```

Trong ví dụ này, lưu ý rằng chúng ta có thể chuyển một [Eloquent model](/docs/{{version}}/eloquent) trực tiếp vào constructor của job được xếp hàng đợi. Vì trait `Queueable` mà job đang sử dụng, các Eloquent models và các relationships đã tải của chúng sẽ được serialize và unserialize một cách mượt mà khi job đang được xử lý.

Nếu job được xếp hàng đợi của bạn chấp nhận một Eloquent model trong constructor của nó, chỉ có định danh cho model sẽ được serialize lên hàng đợi. Khi job thực sự được xử lý, hệ thống hàng đợi sẽ tự động truy xuất lại toàn bộ instance model và các relationships đã tải của nó từ database. Cách tiếp cận này đối với model serialization cho phép gửi các payloads job nhỏ hơn nhiều đến driver hàng đợi của bạn.

<a name="handle-method-dependency-injection"></a>
#### Phương thức `handle` Dependency Injection

Phương thức `handle` được gọi khi job được xử lý bởi hàng đợi. Lưu ý rằng chúng ta có thể type-hint các dependencies trên phương thức `handle` của job. [service container](/docs/{{version}}/container) của Laravel tự động inject các dependencies này.

Nếu bạn muốn kiểm soát hoàn toàn cách container inject dependencies vào phương thức `handle`, bạn có thể sử dụng phương thức `bindMethod` của container. Phương thức `bindMethod` chấp nhận một callback nhận job và container. Trong callback, bạn có thể tự do gọi phương thức `handle` theo cách bạn muốn. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của [service provider](/docs/{{version}}/providers) `App\Providers\AppServiceProvider` của bạn:

```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> [!WARNING]
> Dữ liệu nhị phân, chẳng hạn như nội dung hình ảnh thô, nên được chuyển qua hàm `base64_encode` trước khi được chuyển đến một job được xếp hàng đợi. Nếu không, job có thể không serialize đúng sang JSON khi được đặt trên hàng đợi.

<a name="handling-relationships"></a>
#### Relationships Được xếp hàng đợi

Vì tất cả các relationships Eloquent model đã tải cũng được serialize khi một job được xếp hàng đợi, chuỗi job được serialize đôi khi có thể trở nên khá lớn. Hơn nữa, khi một job được deserialize và các relationships model được truy xuất lại từ database, chúng sẽ được truy xuất hoàn toàn. Bất kỳ các ràng buộc relationship nào trước đó đã được áp dụng trước khi model được serialize trong quá trình xếp hàng đợi job sẽ không được áp dụng khi job được deserialize. Do đó, nếu bạn muốn làm việc với một tập con của một relationship nhất định, bạn nên ràng buộc lại relationship đó trong job được xếp hàng đợi của mình.

Hoặc, để ngăn các relations được serialize, bạn có thể gọi phương thức `withoutRelations` trên model khi đặt giá trị thuộc tính. Phương thức này sẽ trả về một instance của model mà không có các relationships đã tải của nó:

```php
/**
 * Create a new job instance.
 */
public function __construct(
    Podcast $podcast,
) {
    $this->podcast = $podcast->withoutRelations();
}
```

Nếu bạn chỉ cần xóa các relations cụ thể trong khi giữ lại phần còn lại, bạn có thể sử dụng phương thức `withoutRelation`:

```php
$this->podcast = $podcast->withoutRelation('comments');
```

Nếu bạn đang sử dụng [PHP constructor property promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion) và muốn chỉ ra rằng một Eloquent model không nên có các relations của nó được serialize, bạn có thể sử dụng attribute `WithoutRelations`:

```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * Create a new job instance.
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast,
) {}
```

Để thuận tiện, nếu bạn muốn serialize tất cả các models mà không có relationships, bạn có thể áp dụng attribute `WithoutRelations` cho toàn bộ lớp thay vì áp dụng attribute cho từng model:

```php
<?php

namespace App\Jobs;

use App\Models\DistributionPlatform;
use App\Models\Podcast;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\WithoutRelations;

#[WithoutRelations]
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
        public DistributionPlatform $platform,
    ) {}
}
```

Nếu một job nhận được một collection hoặc mảng của các Eloquent models thay vì một model đơn lẻ, các models trong collection đó sẽ không có các relationships của chúng được khôi phục khi job được deserialize và thực thi. Điều này là để ngăn việc sử dụng tài nguyên quá mức trên các jobs xử lý số lượng lớn models.

<a name="unique-jobs"></a>
### Job Độc nhất

> [!WARNING]
> Job độc nhất yêu cầu một driver cache hỗ trợ [locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, các driver cache `memcached`, `redis`, `dynamodb`, `database`, `file`, và `array` hỗ trợ atomic locks.

> [!WARNING]
> Các ràng buộc job độc nhất không áp dụng cho các jobs trong batches.

Đôi khi, bạn có thể muốn đảm bảo rằng chỉ có một instance của một job cụ thể nằm trên hàng đợi tại bất kỳ thời điểm nào. Bạn có thể làm như vậy bằng cách triển khai interface `ShouldBeUnique` trên lớp job của bạn. Interface này không yêu cầu bạn định nghĩa bất kỳ phương thức bổ sung nào trên lớp của bạn:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...
}
```

Trong ví dụ trên, job `UpdateSearchIndex` là độc nhất. Vì vậy, job sẽ không được dispatch nếu một instance khác của job đã nằm trên hàng đợi và chưa hoàn thành xử lý.

Trong một số trường hợp, bạn có thể muốn định nghĩa một "key" cụ thể làm cho job trở nên độc nhất hoặc bạn có thể muốn chỉ định một timeout vượt quá mà job không còn giữ độc nhất. Để thực hiện điều này, bạn có thể sử dụng attribute `UniqueFor` và định nghĩa một phương thức `uniqueId` trên lớp job của bạn:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Queue\Attributes\UniqueFor;

#[UniqueFor(3600)]
class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    /**
     * The product instance.
     *
     * @var \App\Models\Product
     */
    public $product;

    /**
     * Get the unique ID for the job.
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```
Trong ví dụ trên, job `UpdateSearchIndex` là độc nhất theo một ID sản phẩm. Vì vậy, bất kỳ dispatch mới nào của job với cùng ID sản phẩm sẽ bị bỏ qua cho đến khi job hiện tại đã hoàn thành xử lý. Ngoài ra, nếu job hiện tại không được xử lý trong vòng một giờ, lock độc nhất sẽ được giải phóng và một job khác với cùng key độc nhất có thể được dispatch đến hàng đợi.

> [!WARNING]
> Nếu ứng dụng của bạn dispatch jobs từ nhiều web servers hoặc containers, bạn nên đảm bảo rằng tất cả các servers của bạn đang giao tiếp với cùng một cache server trung tâm để Laravel có thể xác định chính xác xem một job có độc nhất hay không.

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### Giữ Jobs Độc nhất Cho đến khi Bắt đầu Xử lý

Theo mặc định, các jobs độc nhất được "mở khóa" sau khi một job hoàn thành xử lý hoặc thất bại tất cả các lần thử lại của nó. Tuy nhiên, có thể có những tình huống mà bạn muốn job của mình mở khóa ngay lập tức trước khi nó được xử lý. Để thực hiện điều này, job của bạn nên triển khai contract `ShouldBeUniqueUntilProcessing` thay vì contract `ShouldBeUnique`:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-job-locks"></a>
#### Locks Job Độc nhất

Đằng sau hậu trường, khi một job `ShouldBeUnique` được dispatch, Laravel cố gắng có được một [lock](/docs/{{version}}/cache#atomic-locks) với key `uniqueId`. Nếu lock đã được giữ, job sẽ không được dispatch. Lock này được giải phóng khi job hoàn thành xử lý hoặc thất bại tất cả các lần thử lại của nó. Theo mặc định, Laravel sẽ sử dụng driver cache mặc định để có được lock này. Tuy nhiên, nếu bạn muốn sử dụng driver khác để có được lock, bạn có thể định nghĩa một phương thức `uniqueVia` trả về driver cache nên được sử dụng:

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...

    /**
     * Get the cache driver for the unique job lock.
     */
    public function uniqueVia(): Repository
    {
        return Cache::driver('redis');
    }
}
```

> [!NOTE]
> Nếu bạn chỉ cần giới hạn xử lý đồng thời của một job, hãy sử dụng [middleware job WithoutOverlapping](/docs/{{version}}/queues#preventing-job-overlaps) thay thế.

<a name="debounced-jobs"></a>
### Jobs Debounce

Đôi khi, bạn có thể muốn đảm bảo rằng khi cùng một job được dispatch nhiều lần trong một khoảng thời gian ngắn, chỉ có dispatch mới nhất thực sự thực thi. Bạn có thể làm như vậy bằng cách thêm attribute `DebounceFor` vào job của bạn:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\DebounceFor;

#[DebounceFor(30)]
class UpdateSearchIndex implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(public int $productId)
    {
    }

    /**
     * Get the debounce ID for the job.
     */
    public function debounceId(): string
    {
        return (string) $this->productId;
    }
}
```

Trong ví dụ trên, việc dispatch `UpdateSearchIndex` lặp lại cho cùng một sản phẩm trong vòng `30` giây sẽ debounce job để chỉ có dispatch mới nhất chạy.

Nếu bạn muốn giới hạn thời gian một job được dispatch lại thường xuyên có thể bị trì hoãn, bạn có thể cung cấp đối số `maxWait` cho attribute `DebounceFor`:

```php
#[DebounceFor(30, maxWait: 120)]
class UpdateSearchIndex implements ShouldQueue
{
    use Queueable;

    // ...
}
```

Bạn có thể tùy chỉnh cache store được sử dụng để theo dõi debounce bằng cách định nghĩa một phương thức `debounceVia` trên job của bạn:

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

public function debounceVia(): Repository
{
    return Cache::driver('redis');
}
```

Nếu một job debounced bị thay thế bởi một dispatch mới hơn, Laravel sẽ dispatch sự kiện `Illuminate\Queue\Events\JobDebounced` và xóa job bị thay thế khỏi hàng đợi.

> [!WARNING]
> Jobs debounced và jobs độc nhất loại trừ lẫn nhau. Một job sử dụng attribute `DebounceFor` không nên triển khai `ShouldBeUnique`.

> [!WARNING]
> Nếu ứng dụng của bạn dispatch các jobs debounced từ nhiều web servers hoặc containers, bạn nên đảm bảo rằng tất cả các servers của bạn đang giao tiếp với cùng một cache server trung tâm.

<a name="encrypted-jobs"></a>
### Jobs Được Mã hóa

Laravel cho phép bạn đảm bảo quyền riêng tư và tính toàn vẹn của dữ liệu job thông qua [mã hóa](/docs/{{version}}/encryption). Để bắt đầu, chỉ cần thêm interface `ShouldBeEncrypted` vào lớp job. Khi interface này đã được thêm vào lớp, Laravel sẽ tự động mã hóa job của bạn trước khi đẩy nó lên hàng đợi:

```php
<?php

use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="job-middleware"></a>
## Middleware Job

Middleware job cho phép bạn bọc logic tùy chỉnh xung quanh việc thực thi các jobs được xếp hàng đợi, giảm boilerplate trong chính các jobs. Ví dụ, hãy xem xét phương thức `handle` sau đây tận dụng các tính năng giới hạn tỷ lệ Redis của Laravel để chỉ cho phép một job xử lý mỗi năm giây:

```php
use Illuminate\Support\Facades\Redis;

/**
 * Execute the job.
 */
public function handle(): void
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('Lock obtained...');

        // Handle job...
    }, function () {
        // Could not obtain lock...

        return $this->release(5);
    });
}
```

Mặc dù mã này hợp lệ, việc triển khai phương thức `handle` trở nên ồn ào vì nó bị lộn xộn với logic giới hạn tỷ lệ Redis. Ngoài ra, logic giới hạn tỷ lệ này phải được nhân bản cho bất kỳ jobs nào khác mà chúng ta muốn giới hạn tỷ lệ. Thay vì giới hạn tỷ lệ trong phương thức handle, chúng ta có thể định nghĩa một middleware job xử lý giới hạn tỷ lệ:

```php
<?php

namespace App\Jobs\Middleware;

use Closure;
use Illuminate\Support\Facades\Redis;

class RateLimited
{
    /**
     * Process the queued job.
     *
     * @param  \Closure(object): void  $next
     */
    public function handle(object $job, Closure $next): void
    {
        Redis::throttle('key')
            ->block(0)->allow(1)->every(5)
            ->then(function () use ($job, $next) {
                // Lock obtained...

                $next($job);
            }, function () use ($job) {
                // Could not obtain lock...

                $job->release(5);
            });
    }
}
```

Như bạn có thể thấy, giống như [middleware route](/docs/{{version}}/middleware), middleware job nhận job đang được xử lý và một callback nên được gọi để tiếp tục xử lý job.

Bạn có thể tạo một lớp middleware job mới bằng lệnh Artisan `make:job-middleware`. Sau khi tạo middleware job, chúng có thể được gắn vào một job bằng cách trả về chúng từ phương thức `middleware` của job. Phương thức này không tồn tại trên các jobs được scaffold bởi lệnh Artisan `make:job`, vì vậy bạn sẽ cần thêm nó thủ công vào lớp job của mình:

```php
use App\Jobs\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited];
}
```

> [!NOTE]
> Middleware job cũng có thể được gán cho [event listeners có thể xếp hàng đợi](/docs/{{version}}/events#queued-event-listeners), [mailables](/docs/{{version}}/mail#queueing-mail), và [notifications](/docs/{{version}}/notifications#queueing-notifications).

<a name="rate-limiting"></a>
### Giới hạn Tỷ lệ

Mặc dù chúng ta vừa chứng minh cách viết middleware giới hạn tỷ lệ job của riêng bạn, Laravel thực sự bao gồm một middleware giới hạn tỷ lệ mà bạn có thể sử dụng để giới hạn tỷ lệ jobs. Giống như [rate limiters route](/docs/{{version}}/routing#defining-rate-limiters), rate limiters job được định nghĩa bằng phương thức `for` của facade `RateLimiter`.

Ví dụ, bạn có thể muốn cho phép người dùng sao lưu dữ liệu của họ một lần mỗi giờ trong khi áp đặt không có giới hạn như vậy đối với khách hàng cao cấp. Để thực hiện điều này, bạn có thể định nghĩa một `RateLimiter` trong phương thức `boot` của `AppServiceProvider` của bạn:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('backups', function (object $job) {
        return $job->user->vipCustomer()
            ? Limit::none()
            : Limit::perHour(1)->by($job->user->id);
    });
}
```

Trong ví dụ trên, chúng ta đã định nghĩa một giới hạn tỷ lệ hàng giờ; tuy nhiên, bạn có thể dễ dàng định nghĩa một giới hạn tỷ lệ dựa trên phút bằng phương thức `perMinute`. Ngoài ra, bạn có thể chuyển bất kỳ giá trị nào bạn muốn đến phương thức `by` của giới hạn tỷ lệ; tuy nhiên, giá trị này thường được sử dụng để phân đoạn giới hạn tỷ lệ theo khách hàng:

```php
return Limit::perMinute(50)->by($job->user->id);
```

Khi bạn đã định nghĩa giới hạn tỷ lệ của mình, bạn có thể gắn rate limiter vào job của mình bằng middleware `Illuminate\Queue\Middleware\RateLimited`. Mỗi khi job vượt quá giới hạn tỷ lệ, middleware này sẽ giải phóng job trở lại hàng đợi với một độ trễ phù hợp dựa trên thời lượng giới hạn tỷ lệ:

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited('backups')];
}
```

Việc giải phóng một job bị giới hạn tỷ lệ trở lại hàng đợi vẫn sẽ tăng tổng số `attempts` của job. Bạn có thể muốn điều chỉnh các thuộc tính `Tries` và `MaxExceptions` trên lớp job của bạn tương ứng. Hoặc, bạn có thể muốn sử dụng [phương thức retryUntil](#time-based-attempts) để định nghĩa lượng thời gian cho đến khi job không còn nên được thử lại.

Sử dụng phương thức `releaseAfter`, bạn cũng có thể chỉ định số giây phải trôi qua trước khi job được giải phóng sẽ được thử lại:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->releaseAfter(60)];
}
```

Nếu bạn không muốn một job được thử lại khi nó bị giới hạn tỷ lệ, bạn có thể sử dụng phương thức `dontRelease`:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->dontRelease()];
}
```

<a name="rate-limiting-with-redis"></a>
#### Giới hạn Tỷ lệ Với Redis

Nếu bạn đang sử dụng Redis, bạn có thể sử dụng middleware `Illuminate\Queue\Middleware\RateLimitedWithRedis`, được tinh chỉnh cho Redis và hiệu quả hơn middleware giới hạn tỷ lệ cơ bản:

```php
use Illuminate\Queue\Middleware\RateLimitedWithRedis;

public function middleware(): array
{
    return [new RateLimitedWithRedis('backups')];
}
```

Phương thức `connection` có thể được sử dụng để chỉ định kết nối Redis nào mà middleware nên sử dụng:

```php
return [(new RateLimitedWithRedis('backups'))->connection('limiter')];
```

<a name="preventing-job-overlaps"></a>
### Ngăn Chặn Ghi đè Job

Laravel bao gồm một middleware `Illuminate\Queue\Middleware\WithoutOverlapping` cho phép bạn ngăn chặn các ghi đè job dựa trên một key tùy ý. Điều này có thể hữu ích khi một job được xếp hàng đợi đang sửa đổi một tài nguyên chỉ nên được sửa đổi bởi một job tại một thời điểm.

Ví dụ, hãy tưởng tượng bạn có một job được xếp hàng đợi cập nhật điểm tín dụng của người dùng và bạn muốn ngăn chặn các ghi đè job cập nhật điểm tín dụng cho cùng một ID người dùng. Để thực hiện điều này, bạn có thể trả về middleware `WithoutOverlapping` từ phương thức `middleware` của job của bạn:

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

Việc giải phóng một job ghi đè trở lại hàng đợi vẫn sẽ tăng tổng số lần thử của job. Bạn có thể muốn điều chỉnh các thuộc tính `Tries` và `MaxExceptions` trên lớp job của bạn tương ứng. Ví dụ, để `Tries` là 1 như mặc định sẽ ngăn bất kỳ job ghi đè nào được thử lại sau này.

Bất kỳ jobs ghi đè nào cùng loại sẽ được giải phóng trở lại hàng đợi. Bạn cũng có thể chỉ định số giây phải trôi qua trước khi job được giải phóng sẽ được thử lại:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
}
```

Nếu bạn muốn xóa ngay lập tức bất kỳ jobs ghi đè nào để chúng không được thử lại, bạn có thể sử dụng phương thức `dontRelease`:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->dontRelease()];
}
```

Middleware `WithoutOverlapping` được cung cấp bởi tính năng atomic lock của Laravel. Đôi khi, job của bạn có thể thất bại hoặc timeout bất ngờ theo cách mà lock không được giải phóng. Do đó, bạn có thể định nghĩa rõ ràng một thời gian hết hạn lock bằng phương thức `expireAfter`. Ví dụ, ví dụ dưới đây sẽ hướng dẫn Laravel giải phóng lock `WithoutOverlapping` ba phút sau khi job đã bắt đầu xử lý:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
}
```

> [!WARNING]
> Middleware `WithoutOverlapping` yêu cầu một driver cache hỗ trợ [locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, các driver cache `memcached`, `redis`, `dynamodb`, `database`, `file`, và `array` hỗ trợ atomic locks.

<a name="sharing-lock-keys"></a>
#### Chia sẻ Keys Lock Across Các Lớp Job

Theo mặc định, middleware `WithoutOverlapping` sẽ chỉ ngăn chặn các jobs ghi đè của cùng một lớp. Vì vậy, mặc dù hai lớp job khác nhau có thể sử dụng cùng một key lock, chúng sẽ không bị ngăn chặn ghi đè. Tuy nhiên, bạn có thể hướng dẫn Laravel áp dụng key across các lớp job bằng phương thức `shared`:

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProviderIsDown
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}
```class ProviderIsUp
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}
```

<a name="throttling-exceptions"></a>
### Throttling Exceptions

Laravel bao gồm middleware `Illuminate\Queue\Middleware\ThrottlesExceptions` cho phép bạn throttle các exception. Khi job ném ra một số lượng exception nhất định, tất cả các lần thử tiếp theo để thực thi job sẽ bị trì hoãn cho đến khi khoảng thời gian quy định trôi qua. Middleware này đặc biệt hữu ích cho các job tương tác với các dịch vụ bên thứ ba không ổn định.

Ví dụ, hãy tưởng tượng một queued job tương tác với API bên thứ ba bắt đầu ném ra exception. Để throttle exception, bạn có thể trả về middleware `ThrottlesExceptions` từ phương thức `middleware` của job. Thông thường, middleware này nên được kết hợp với một job thực hiện [time based attempts](#time-based-attempts):

```php
use DateTime;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new ThrottlesExceptions(10, 5 * 60)];
}

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 30);
}
```

Đối số đầu tiên của constructor mà middleware chấp nhận là số lượng exception mà job có thể ném ra trước khi bị throttle, trong khi đối số thứ hai của constructor là số giây nên trôi qua trước khi job được thử lại sau khi đã bị throttle. Trong ví dụ mã trên, nếu job ném ra 10 exception liên tiếp, chúng ta sẽ đợi 5 phút trước khi thử lại job, bị giới hạn bởi giới hạn thời gian 30 phút.

Khi job ném ra exception nhưng ngưỡng exception chưa được đạt, job thường sẽ được thử lại ngay lập tức. Tuy nhiên, bạn có thể chỉ định số phút mà job như vậy nên bị trì hoãn bằng cách gọi phương thức `backoff` khi đính kèm middleware vào job:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5 * 60))->backoff(5)];
}
```

Phương thức `backoff` cũng chấp nhận một closure nhận exception đã ném, cho phép xác định độ trễ một cách động:

```php
use App\Exceptions\RateLimitedException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;
use Throwable;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5 * 60))->backoff(
        fn (Throwable $throwable) => $throwable instanceof RateLimitedException
            ? $throwable->retryAfterMinutes()
            : 5
    )];
}
```

Nội bộ, middleware này sử dụng hệ thống cache của Laravel để thực hiện rate limiting, và tên lớp của job được sử dụng làm cache "key". Bạn có thể ghi đè key này bằng cách gọi phương thức `by` khi đính kèm middleware vào job của bạn. Điều này có thể hữu ích nếu bạn có nhiều job tương tác với cùng một dịch vụ bên thứ ba và bạn muốn chúng chia sẻ một "bucket" throttling chung để đảm bảo chúng tôn trọng một giới hạn chia sẻ duy nhất:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->by('key')];
}
```

Theo mặc định, middleware này sẽ throttle mọi exception. Bạn có thể sửa đổi hành vi này bằng cách gọi phương thức `when` khi đính kèm middleware vào job của bạn. Exception sau đó sẽ chỉ được throttle nếu closure được cung cấp cho phương thức `when` trả về `true`:

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->when(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

Khác với phương thức `when`, phương thức này giải phóng job trở lại queue hoặc ném exception, phương thức `deleteWhen` cho phép bạn xóa job hoàn toàn khi một exception nhất định xảy ra:

```php
use App\Exceptions\CustomerDeletedException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(2, 10 * 60))->deleteWhen(CustomerDeletedException::class)];
}
```

Nếu bạn muốn các exception bị throttle được báo cáo cho exception handler của ứng dụng, bạn có thể làm điều đó bằng cách gọi phương thức `report` khi đính kèm middleware vào job của bạn. Tùy chọn, bạn có thể cung cấp một closure cho phương thức `report` và exception sẽ chỉ được báo cáo nếu closure đã cho trả về `true`:

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->report(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

<a name="throttling-exceptions-with-redis"></a>
#### Throttling Exceptions With Redis

Nếu bạn đang sử dụng Redis, bạn có thể sử dụng middleware `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis`, được tinh chỉnh cho Redis và hiệu quả hơn middleware throttling exception cơ bản:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis;

public function middleware(): array
{
    return [new ThrottlesExceptionsWithRedis(10, 10 * 60)];
}
```

Phương thức `connection` có thể được sử dụng để chỉ định kết nối Redis nào mà middleware nên sử dụng:

```php
return [(new ThrottlesExceptionsWithRedis(10, 10 * 60))->connection('limiter')];
```

<a name="skipping-jobs"></a>
### Skipping Jobs

Middleware `Skip` cho phép bạn chỉ định rằng một job nên được bỏ qua / xóa mà không cần sửa đổi logic của job. Phương thức `Skip::when` sẽ xóa job nếu điều kiện đã cho đánh giá là `true`, trong khi phương thức `Skip::unless` sẽ xóa job nếu điều kiện đánh giá là `false`:

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [
        Skip::when($condition),
    ];
}
```

Bạn cũng có thể truyền một `Closure` cho các phương thức `when` và `unless` để đánh giá điều kiện phức tạp hơn:

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [
        Skip::when(function (): bool {
            return $this->shouldSkip();
        }),
    ];
}
```

<a name="dispatching-jobs"></a>
## Dispatching Jobs

Khi bạn đã viết lớp job của mình, bạn có thể dispatch nó bằng phương thức `dispatch` trên chính job đó. Các đối số được truyền cho phương thức `dispatch` sẽ được đưa cho constructor của job:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast);

        return redirect('/podcasts');
    }
}
```

Nếu bạn muốn dispatch job có điều kiện, bạn có thể sử dụng các phương thức `dispatchIf` và `dispatchUnless`:

```php
ProcessPodcast::dispatchIf($accountActive, $podcast);

ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```

Trong các ứng dụng Laravel mới, kết nối `database` được định nghĩa là queue mặc định. Bạn có thể chỉ định một kết nối queue mặc định khác bằng cách thay đổi biến môi trường `QUEUE_CONNECTION` trong file `.env` của ứng dụng.

<a name="delayed-dispatching"></a>
### Delayed Dispatching

Nếu bạn muốn chỉ định rằng một job không nên có sẵn để xử lý ngay lập tức bởi queue worker, bạn có thể sử dụng phương thức `delay` khi dispatch job. Ví dụ, hãy chỉ định rằng một job không nên có sẵn để xử lý cho đến 10 phút sau khi nó đã được dispatch:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast)
            ->delay(now()->plus(minutes: 10));

        return redirect('/podcasts');
    }
}
```

Trong một số trường hợp, job có thể có độ trễ mặc định được cấu hình. Nếu bạn cần bỏ qua độ trễ này và dispatch một job để xử lý ngay lập tức, bạn có thể sử dụng phương thức `withoutDelay`:

```php
ProcessPodcast::dispatch($podcast)->withoutDelay();
```

> [!WARNING]
> Dịch vụ queue Amazon SQS có thời gian trễ tối đa là 15 phút.

<a name="synchronous-dispatching"></a>
### Synchronous Dispatching

Nếu bạn muốn dispatch một job ngay lập tức (đồng bộ), bạn có thể sử dụng phương thức `dispatchSync`. Khi sử dụng phương thức này, job sẽ không được đưa vào queue và sẽ được thực thi ngay lập tức trong tiến trình hiện tại:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatchSync($podcast);

        return redirect('/podcasts');
    }
}
```

<a name="deferred-dispatching"></a>
#### Deferred Dispatching

Sử dụng deferred synchronous dispatching, bạn có thể dispatch một job để được xử lý trong tiến trình hiện tại, nhưng sau khi phản hồi HTTP đã được gửi cho người dùng. Điều này cho phép bạn xử lý các job "queued" đồng bộ mà không làm chậm trải nghiệm ứng dụng của người dùng. Để trì hoãn thực thi của một job đồng bộ, dispatch job đến kết nối `deferred`:

```php
RecordDelivery::dispatch($order)->onConnection('deferred');
```

Kết nối `deferred` cũng đóng vai trò là [failover queue](#queue-failover) mặc định.

Tương tự, kết nối `background` xử lý các job sau khi phản hồi HTTP đã được gửi cho người dùng; tuy nhiên, job được xử lý trong một tiến trình PHP được tạo riêng, cho phép PHP-FPM / application worker có sẵn để xử lý một yêu cầu HTTP khác:

```php
RecordDelivery::dispatch($order)->onConnection('background');
```

<a name="bulk-dispatching"></a>
### Bulk Dispatching

Nếu bạn cần dispatch nhiều job độc lập cùng một lúc và không cần theo dõi hoặc callback [batch](#job-batching), bạn có thể sử dụng phương thức `bulk` của facade `Bus`. Laravel sẽ nhóm các job theo kết nối queue và tên queue được cấu hình của chúng và đẩy từng nhóm đến queue thích hợp hàng loạt:

```php
use App\Jobs\ProcessUser;
use Illuminate\Support\Facades\Bus;

Bus::bulk(
    $users->map(fn ($user) => new ProcessUser($user))
);
```

<a name="preparing-jobs-before-dispatch"></a>
### Preparing Jobs Before Dispatch

Nếu một job cần chuẩn bị hoặc kiểm tra trạng thái của nó trước khi được đẩy lên queue, job có thể thực hiện interface `Illuminate\Contracts\Queue\PreparesForDispatch`. Laravel sẽ gọi phương thức `prepareForDispatch` của job trước khi dispatch job. Nếu phương thức này trả về `false`, job sẽ không được dispatch:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\PreparesForDispatch;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Support\Facades\Cache;

class SyncPodcasts implements PreparesForDispatch, ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public array $podcastIds,
    ) {}

    /**
     * Prepare the job before dispatching.
     */
    public function prepareForDispatch(): bool
    {
        return collect($this->podcastIds)
            ->reject(fn (int $id) => Cache::has("podcast-syncing:{$id}"))
            ->isNotEmpty();
    }
}
```

<a name="jobs-and-database-transactions"></a>
### Jobs & Database Transactions

Mặc dù hoàn toàn ổn để dispatch job trong các transaction cơ sở dữ liệu, bạn nên đặc biệt chú ý để đảm bảo rằng job của bạn thực sự có thể thực thi thành công. Khi dispatch một job trong một transaction, có thể job sẽ được xử lý bởi worker trước khi transaction cha đã commit. Khi điều này xảy ra, bất kỳ cập nhật nào bạn đã thực hiện cho các model hoặc bản ghi cơ sở dữ liệu trong transaction cơ sở dữ liệu có thể chưa được phản ánh trong cơ sở dữ liệu. Ngoài ra, bất kỳ model hoặc bản ghi cơ sở dữ liệu nào được tạo trong transaction có thể không tồn tại trong cơ sở dữ liệu.

May mắn thay, Laravel cung cấp một số phương pháp để giải quyết vấn đề này. Đầu tiên, bạn có thể đặt tùy chọn kết nối `after_commit` trong mảng cấu hình kết nối queue của mình:

```php
'redis' => [
    'driver' => 'redis',
    // ...
    'after_commit' => true,
],
```

Khi tùy chọn `after_commit` là `true`, bạn có thể dispatch job trong các transaction cơ sở dữ liệu; tuy nhiên, Laravel sẽ đợi cho đến khi các transaction cơ sở dữ liệu cha mở đã được commit trước khi thực sự dispatch job. Tất nhiên, nếu không có transaction cơ sở dữ liệu nào đang mở, job sẽ được dispatch ngay lập tức.

Nếu một transaction được rollback do một exception xảy ra trong transaction, các job đã được dispatch trong transaction đó sẽ bị loại bỏ.

> [!NOTE]
> Đặt tùy chọn cấu hình `after_commit` thành `true` cũng sẽ khiến bất kỳ queued event listeners, mailables, notifications, và broadcast events nào được dispatch sau khi tất cả các transaction cơ sở dữ liệu mở đã được commit.

<a name="specifying-commit-dispatch-behavior-inline"></a>
#### Specifying Commit Dispatch Behavior Inline

Nếu bạn không đặt tùy chọn cấu hình kết nối queue `after_commit` thành `true`, bạn vẫn có thể chỉ định rằng một job cụ thể nên được dispatch sau khi tất cả các transaction cơ sở dữ liệu mở đã được commit. Để thực hiện điều này, bạn có thể chuỗi phương thức `afterCommit` vào thao tác dispatch của mình:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterCommit();
```

Tương tự, nếu tùy chọn cấu hình `after_commit` được đặt thành `true`, bạn có thể chỉ định rằng một job cụ thể nên được dispatch ngay lập tức mà không cần đợi bất kỳ transaction cơ sở dữ liệu mở nào commit:

```php
ProcessPodcast::dispatch($podcast)->beforeCommit();
```

<a name="job-chaining"></a>
### Job Chaining

Job chaining cho phép bạn chỉ định một danh sách các queued job nên được chạy theo tuần tự sau khi job chính đã thực thi thành công. Nếu một job trong chuỗi thất bại, các job còn lại sẽ không được chạy. Để thực thi một chuỗi job queued, bạn có thể sử dụng phương thức `chain` được cung cấp bởi facade `Bus`. Command bus của Laravel là một thành phần cấp thấp mà queued job dispatching được xây dựng trên đó:

```php
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

Ngoài việc chaining các instance lớp job, bạn cũng có thể chain các closure:

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    function () {
        Podcast::update(/* ... */);
    },
])->dispatch();
```

> [!WARNING]
> Xóa job bằng phương thức `$this->delete()` trong job sẽ không ngăn chặn các job được chained khỏi việc được xử lý. Chuỗi sẽ chỉ dừng thực thi nếu một job trong chuỗi thất bại.

<a name="chain-connection-queue"></a>
#### Chain Connection and Queue

Nếu bạn muốn chỉ định kết nối và queue nên được sử dụng cho các job được chained, bạn có thể sử dụng các phương thức `onConnection` và `onQueue`. Các phương thức này chỉ định kết nối queue và tên queue nên được sử dụng trừ khi queued job được gán một kết nối / queue khác một cách rõ ràng:

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->onConnection('redis')->onQueue('podcasts')->dispatch();
```

<a name="adding-jobs-to-the-chain"></a>
#### Adding Jobs to the Chain

Thỉnh thoảng, bạn có thể cần thêm vào đầu hoặc thêm vào cuối một job vào một chuỗi job hiện có từ trong một job khác trong chuỗi đó. Bạn có thể thực hiện điều này bằng cách sử dụng các phương thức `prependToChain` và `appendToChain`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    // Prepend to the current chain, run job immediately after current job...
    $this->prependToChain(new TranscribePodcast);

    // Append to the current chain, run job at end of chain...
    $this->appendToChain(new TranscribePodcast);
}
```

<a name="chain-failures"></a>
#### Chain Failures

Khi chaining job, bạn có thể sử dụng phương thức `catch` để chỉ định một closure nên được gọi nếu một job trong chuỗi thất bại. Callback đã cho sẽ nhận instance `Throwable` gây ra thất bại của job:

```php
use Illuminate\Support\Facades\Bus;
use Throwable;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->catch(function (Throwable $e) {
    // A job within the chain has failed...
})->dispatch();
```

> [!WARNING]
> Vì các callback chuỗi được serialize và thực thi tại một thời điểm sau bởi queue của Laravel, bạn không nên sử dụng biến `$this` trong các callback chuỗi.

<a name="customizing-the-queue-and-connection"></a>
### Customizing the Queue and Connection

<a name="dispatching-to-a-particular-queue"></a>
#### Dispatching to a Particular Queue

Bằng cách đẩy job đến các queue khác nhau, bạn có thể "phân loại" các queued job của mình và thậm chí ưu tiên số lượng worker bạn gán cho các queue khác nhau. Hãy nhớ rằng, điều này không đẩy job đến các "kết nối" queue khác nhau như được định nghĩa bởi file cấu hình queue của bạn, mà chỉ đến các queue cụ thể trong một kết nối duy nhất. Để chỉ định queue, sử dụng phương thức `onQueue` khi dispatch job:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');

        return redirect('/podcasts');
    }
}
```

Ngoài ra, bạn có thể chỉ định queue của job bằng cách gọi phương thức `onQueue` trong constructor của job:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onQueue('processing');
    }
}
```

<a name="dispatching-to-a-particular-connection"></a>
#### Dispatching to a Particular Connection

Nếu ứng dụng của bạn tương tác với nhiều kết nối queue, bạn có thể chỉ định kết nối nào để đẩy job đến bằng phương thức `onConnection`:

```php
<?php

namespace App\Http\Controllers;

use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');

        return redirect('/podcasts');
    }
}
```

Bạn có thể chuỗi các phương thức `onConnection` và `onQueue` lại với nhau để chỉ định kết nối và queue cho một job:

```php
ProcessPodcast::dispatch($podcast)
    ->onConnection('sqs')
    ->onQueue('processing');
```

Ngoài ra, bạn có thể chỉ định kết nối của job bằng cách gọi phương thức `onConnection` trong constructor của job:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onConnection('sqs');
    }
}
```

<a name="queue-routing"></a>
#### Queue Routing

Bạn có thể sử dụng phương thức `route` của facade `Queue` để định nghĩa một kết nối và queue mặc định cho các lớp job cụ thể. Điều này hữu ích khi bạn muốn đảm bảo các job cụ thể luôn sử dụng các queue cụ thể mà không cần chỉ định kết nối hoặc queue trên job.

Ngoài việc routing các lớp job cụ thể, bạn cũng có thể truyền một interface, trait, hoặc lớp cha cho phương thức `route`. Khi bạn làm điều này, bất kỳ job nào thực hiện interface, sử dụng trait, hoặc mở rộng lớp cha sẽ tự động sử dụng kết nối và queue được cấu hình.

Thông thường, bạn nên gọi phương thức `route` từ phương thức `boot` của một service provider:

```php
use App\Concerns\RequiresVideo;
use App\Jobs\ProcessPodcast;
use App\Jobs\ProcessVideo;
use Illuminate\Support\Facades\Queue;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
    Queue::route(RequiresVideo::class, queue: 'video');
}
```

Khi một kết nối được chỉ định mà không có queue, job sẽ được gửi đến queue mặc định:

```php
Queue::route(ProcessPodcast::class, connection: 'redis');
```

Bạn cũng có thể route nhiều lớp job cùng một lúc bằng cách truyền một mảng cho phương thức `route`:

```php
Queue::route([
    ProcessPodcast::class => ['podcasts', 'redis'], // Queue and connection
    ProcessVideo::class => 'videos', // Queue only (uses default connection)
]);
```

> [!NOTE]
> Queue routing vẫn có thể được ghi đè bởi job trên cơ sở từng job.

<a name="max-job-attempts-and-timeout"></a>
### Specifying Max Job Attempts / Timeout Values

<a name="max-attempts"></a>
#### Max Attempts

Job attempts là một khái niệm cốt lõi của hệ thống queue của Laravel và cung cấp sức mạnh cho nhiều tính năng nâng cao. Mặc dù chúng có thể gây nhầm lẫn lúc đầu, điều quan trọng là phải hiểu cách chúng hoạt động trước khi sửa đổi cấu hình mặc định.

Khi một job được dispatch, nó được đẩy lên queue. Worker sau đó chọn nó lên và cố gắng thực thi nó. Đây là một job attempt.

Tuy nhiên, một attempt không nhất thiết có nghĩa là phương thức `handle` của job đã được thực thi. Attempts cũng có thể được "tiêu thụ" theo một số cách:

<div class="content-list" markdown="1">

- Job gặp một exception không được xử lý trong quá trình thực thi.
- Job được giải phóng thủ công trở lại queue bằng cách sử dụng `$this->release()`.
- Middleware như `WithoutOverlapping` hoặc `RateLimited` không thể lấy được lock và giải phóng job.
- Job đã hết thời gian chờ (timeout).
- Phương thức `handle` của job chạy và hoàn thành mà không ném exception.

</div>

Bạn có thể không muốn tiếp tục thử một job mãi mãi. Do đó, Laravel cung cấp nhiều cách để chỉ định số lần hoặc trong bao lâu một job có thể được thử.

> [!NOTE]
> Theo mặc định, Laravel sẽ chỉ thử một job một lần. Nếu job của bạn sử dụng middleware như `WithoutOverlapping` hoặc `RateLimited`, hoặc nếu bạn đang giải phóng job thủ công, bạn có thể sẽ cần tăng số lượng attempt được phép qua tùy chọn `tries`.

Một cách tiếp cận để chỉ định số lần tối đa một job có thể được thử là qua switch `--tries` trên dòng lệnh Artisan. Điều này sẽ áp dụng cho tất cả các job được xử lý bởi worker trừ khi job đang được xử lý chỉ định số lần nó có thể được thử:

```shell
php artisan queue:work --tries=3
```

Nếu một job vượt quá số lần thử tối đa của nó, nó sẽ được coi là một job "thất bại". Để biết thêm thông tin về xử lý các job thất bại, hãy tham khảo [tài liệu failed job](#dealing-with-failed-jobs). Nếu `--tries=0` được cung cấp cho lệnh `queue:work`, job sẽ được thử lại mãi mãi.

Bạn có thể tiếp cận chi tiết hơn bằng cách định nghĩa số lần tối đa một job có thể được thử trên chính lớp job bằng thuộc tính `Tries`. Nếu số lần thử tối đa được chỉ định trên job, nó sẽ được ưu tiên hơn giá trị `--tries` được cung cấp trên dòng lệnh:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Tries;

#[Tries(5)]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

Nếu bạn cần kiểm soát động về số lần thử tối đa của một job cụ thể, bạn có thể định nghĩa một phương thức `tries` trên job:

```php
/**
 * Determine number of times the job may be attempted.
 */
public function tries(): int
{
    return 5;
}
```

<a name="time-based-attempts"></a>
#### Time Based Attempts

Là một giải pháp thay thế cho việc định nghĩa số lần một job có thể được thử trước khi nó thất bại, bạn có thể định nghĩa một thời điểm mà job không nên được thử nữa. Điều này cho phép một job được thử bất kỳ số lần nào trong một khung thời gian nhất định. Để định nghĩa thời điểm mà job không nên được thử nữa, thêm một phương thức `retryUntil` vào lớp job của bạn. Phương thức này nên trả về một instance `DateTime`:

```php
use DateTime;

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 10);
}
```

Nếu cả `retryUntil` và `tries` đều được định nghĩa, Laravel sẽ ưu tiên phương thức `retryUntil`.

> [!NOTE]
> Bạn cũng có thể định nghĩa một thuộc tính `Tries` hoặc phương thức `retryUntil` trên [queued event listeners](/docs/{{version}}/events#queued-event-listeners) và [queued notifications](/docs/{{version}}/notifications#queueing-notifications) của bạn.

<a name="max-exceptions"></a>
#### Max Exceptions

Đôi khi bạn có thể muốn chỉ định rằng một job có thể được thử nhiều lần, nhưng nên thất bại nếu các lần thử lại được kích hoạt bởi một số lượng exception không được xử lý nhất định (trái ngược với việc được giải phóng bởi phương thức `release` trực tiếp). Để thực hiện điều này, bạn có thể sử dụng các thuộc tính `Tries` và `MaxExceptions` trên lớp job của mình:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Support\Facades\Redis;

#[Tries(25)]
#[MaxExceptions(3)]
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Redis::throttle('key')->allow(10)->every(60)->then(function () {
            // Lock obtained, process the podcast...
        }, function () {
            // Unable to obtain lock...
            return $this->release(10);
        });
    }
}
```

Trong ví dụ này, job sẽ được release trong mười giây nếu ứng dụng không thể lấy được Redis lock và sẽ tiếp tục được thử lại tối đa 25 lần. Tuy nhiên, job sẽ thất bại nếu ba ngoại lệ không được xử lý được ném ra bởi job.

<a name="timeout"></a>
#### Timeout

Thường thì bạn biết ước tính thời gian job của bạn sẽ mất bao lâu. Vì lý do này, Laravel cho phép bạn chỉ định giá trị "timeout". Theo mặc định, giá trị timeout là 60 giây. Nếu một job đang xử lý lâu hơn số giây được chỉ định bởi giá trị timeout, worker đang xử lý job đó sẽ thoát với lỗi. Thông thường, worker sẽ được khởi động lại tự động bởi [process manager được cấu hình trên server của bạn](#supervisor-configuration).

Số giây tối đa mà các job có thể chạy có thể được chỉ định bằng cách sử dụng switch `--timeout` trên dòng lệnh Artisan:

```shell
php artisan queue:work --timeout=30
```

Nếu job vượt quá số lần thử tối đa của nó do liên tục timeout, nó sẽ được đánh dấu là thất bại.

Bạn cũng có thể định nghĩa số giây tối đa mà một job được phép chạy bằng cách sử dụng attribute `Timeout` trên class job. Nếu timeout được chỉ định trên job, nó sẽ được ưu tiên hơn bất kỳ timeout nào được chỉ định trên dòng lệnh:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Timeout;

#[Timeout(120)]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

Đôi khi, các quá trình chặn IO như socket hoặc kết nối HTTP đi ra có thể không tôn trọng timeout bạn chỉ định. Do đó, khi sử dụng các tính năng này, bạn nên luôn cố gắng chỉ định timeout bằng API của chúng. Ví dụ, khi sử dụng [Guzzle](https://docs.guzzlephp.org), bạn nên luôn chỉ định giá trị timeout cho kết nối và yêu cầu.

> [!WARNING]
> [Extension PCNTL](https://www.php.net/manual/en/book.pcntl.php) của PHP phải được cài đặt để có thể chỉ định timeout cho job. Ngoài ra, giá trị "timeout" của job phải luôn nhỏ hơn giá trị ["retry after"](#job-expiration) của nó. Nếu không, job có thể được thử lại trước khi nó thực sự hoàn thành việc thực thi hoặc timeout.

<a name="failing-on-timeout"></a>
#### Thất bại khi Timeout

Nếu bạn muốn chỉ định rằng một job nên được đánh dấu là [thất bại](#dealing-with-failed-jobs) khi timeout, bạn có thể sử dụng attribute `FailOnTimeout` trên class job:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\FailOnTimeout;

#[FailOnTimeout]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

> [!NOTE]
> Theo mặc định, khi một job timeout, nó tiêu tốn một lần thử và được release lại vào hàng đợi (nếu cho phép thử lại). Tuy nhiên, nếu bạn cấu hình job để thất bại khi timeout, nó sẽ không được thử lại, bất kể giá trị được đặt cho tries.

<a name="sqs-fifo-and-fair-queues"></a>
### Hàng đợi SQS FIFO và Fair

Laravel hỗ trợ [hàng đợi Amazon SQS FIFO (First-In-First-Out)](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fifo-queues.html) và [hàng đợi fair](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-fair-queues.html). Hàng đợi FIFO cho phép bạn xử lý các job theo đúng thứ tự chúng được gửi đồng thời đảm bảo xử lý chính xác một lần thông qua việc loại bỏ trùng lặp tin nhắn.

Hàng đợi FIFO yêu cầu ID nhóm tin nhắn để xác định các job nào có thể được xử lý song song. Các job có cùng ID nhóm được xử lý tuần tự, trong khi các tin nhắn có ID nhóm khác nhau có thể được xử lý đồng thời.

Laravel cung cấp phương thức `onGroup` linh hoạt để chỉ định ID nhóm tin nhắn khi dispatch các job:

```php
ProcessOrder::dispatch($order)
    ->onGroup("customer-{$order->customer_id}");
```

Hàng đợi SQS FIFO hỗ trợ loại bỏ trùng lặp tin nhắn để đảm bảo xử lý chính xác một lần. Triển khai phương thức `deduplicationId` trong class job của bạn để cung cấp ID loại bỏ trùng lặp tùy chỉnh:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessSubscriptionRenewal implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Get the job's deduplication ID.
     */
    public function deduplicationId(): string
    {
        return "renewal-{$this->subscription->id}";
    }
}
```

<a name="fair-queues"></a>
#### Hàng đợi Fair

Nếu bạn đang sử dụng hàng đợi tiêu chuẩn SQS, việc đặt nhóm tin nhắn sẽ kích hoạt hàng đợi fair. Nói cách khác, một khi bạn gán các nhóm, SQS sẽ sử dụng chúng để duy trì phân phối công bằng giữa các tenant / workload. Không cần cấu hình Laravel bổ sung nào.

Thay vì gọi `onGroup` tại thời điểm dispatch, bạn cũng có thể định nghĩa phương thức `messageGroup` trực tiếp trên job:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessOrder implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Get the job's message group.
     */
    public function messageGroup(): string
    {
        return "customer-{$this->order->customer_id}";
    }
}
```

<a name="fifo-listeners-mail-and-notifications"></a>
#### Listeners, Mail và Notifications FIFO

Khi sử dụng hàng đợi FIFO, bạn cũng sẽ cần định nghĩa các nhóm tin nhắn trên listeners, mail và notifications. Ngoài ra, bạn có thể dispatch các phiên bản được xếp hàng đợi của các đối tượng này vào hàng đợi không phải FIFO.

Để định nghĩa nhóm tin nhắn cho một [queued event listener](/docs/{{version}}/events#queued-event-listeners), hãy định nghĩa phương thức `messageGroup` trên listener. Bạn cũng có thể tùy chọn định nghĩa phương thức `deduplicationId`:

```php
<?php

namespace App\Listeners;

class SendShipmentNotification
{
    // ...

    /**
     * Get the job's message group.
     */
    public function messageGroup(): string
    {
        return 'shipments';
    }

    /**
     * Get the job's deduplication ID.
     */
    public function deduplicationId(): string
    {
        return "shipment-notification-{$this->shipment->id}";
    }
}
```

Khi gửi một [mail message](/docs/{{version}}/mail) sẽ được xếp vào hàng đợi FIFO, bạn nên gọi phương thức `onGroup` và tùy chọn phương thức `withDeduplicator` khi gửi thông báo:

```php
use App\Mail\InvoicePaid;
use Illuminate\Support\Facades\Mail;

$invoicePaid = (new InvoicePaid($invoice))
    ->onGroup('invoices')
    ->withDeduplicator(fn () => 'invoices-'.$invoice->id);

Mail::to($request->user())->send($invoicePaid);
```

Khi gửi một [notification](/docs/{{version}}/notifications) sẽ được xếp vào hàng đợi FIFO, bạn nên gọi phương thức `onGroup` và tùy chọn phương thức `withDeduplicator` khi gửi thông báo:

```php
use App\Notifications\InvoicePaid;

$invoicePaid = (new InvoicePaid($invoice))
    ->onGroup('invoices')
    ->withDeduplicator(fn () => 'invoices-'.$invoice->id);

$user->notify($invoicePaid);
```

<a name="queue-failover"></a>
### Queue Failover

Driver hàng đợi `failover` cung cấp chức năng failover tự động khi đẩy các job vào hàng đợi. Nếu kết nối hàng đợi chính của cấu hình `failover` thất bại vì bất kỳ lý do gì, Laravel sẽ tự động cố gắng đẩy job vào kết nối được cấu hình tiếp theo trong danh sách. Điều này đặc biệt hữu ích để đảm bảo tính sẵn sàng cao trong môi trường sản xuất nơi độ tin cậy của hàng đợi là quan trọng.

Để cấu hình một kết nối hàng đợi failover, hãy chỉ định driver `failover` và cung cấp một mảng tên kết nối để thử theo thứ tự. Theo mặc định, Laravel bao gồm một cấu hình failover mẫu trong file cấu hình `config/queue.php` của ứng dụng của bạn:

```php
'failover' => [
    'driver' => 'failover',
    'connections' => [
        'redis',
        'database',
        'sync',
    ],
],
```

Sau khi bạn đã cấu hình một kết nối sử dụng driver `failover`, bạn sẽ cần đặt kết nối failover làm kết nối hàng đợi mặc định của bạn trong file `.env` của ứng dụng để sử dụng chức năng failover:

```ini
QUEUE_CONNECTION=failover
```

Tiếp theo, khởi động ít nhất một worker cho mỗi kết nối trong danh sách kết nối failover của bạn:

```bash
php artisan queue:work redis
php artisan queue:work database
```

> [!NOTE]
> Bạn không cần chạy worker cho các kết nối sử dụng driver hàng đợi `sync`, `background` hoặc `deferred` vì các driver đó xử lý các job trong quy trình PHP hiện tại.

Khi một thao tác kết nối hàng đợi thất bại và failover được kích hoạt, Laravel sẽ dispatch sự kiện `Illuminate\Queue\Events\QueueFailedOver`, cho phép bạn báo cáo hoặc ghi log rằng một kết nối hàng đợi đã thất bại.

> [!NOTE]
> Nếu bạn sử dụng Laravel Horizon, hãy nhớ rằng Horizon chỉ quản lý các hàng đợi Redis. Nếu danh sách failover của bạn bao gồm `database`, bạn nên chạy một quy trình `php artisan queue:work database` thông thường cùng với Horizon.

<a name="error-handling"></a>
### Xử lý Lỗi

Nếu một ngoại lệ được ném ra trong khi job đang được xử lý, job sẽ tự động được release lại vào hàng đợi để nó có thể được thử lại. Job sẽ tiếp tục được release cho đến khi nó đã được thử số lần tối đa được phép bởi ứng dụng của bạn. Số lần thử tối đa được định nghĩa bởi switch `--tries` được sử dụng trên lệnh Artisan `queue:work`. Ngoài ra, số lần thử tối đa có thể được định nghĩa trên chính class job. Thông tin thêm về việc chạy worker hàng đợi [có thể được tìm thấy bên dưới](#running-the-queue-worker).

<a name="manually-releasing-a-job"></a>
#### Release Job Thủ công

Đôi khi bạn có thể muốn release một job thủ công trở lại vào hàng đợi để nó có thể được thử lại vào một thời điểm sau. Bạn có thể thực hiện điều này bằng cách gọi phương thức `release`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->release();
}
```

Theo mặc định, phương thức `release` sẽ release job trở lại vào hàng đợi để xử lý ngay lập tức. Tuy nhiên, bạn có thể hướng dẫn hàng đợi không làm cho job có sẵn để xử lý cho đến khi một số giây nhất định đã trôi qua bằng cách chuyển một số nguyên hoặc instance ngày cho phương thức `release`:

```php
$this->release(10);

$this->release(now()->plus(seconds: 10));
```

<a name="manually-failing-a-job"></a>
#### Thất bại Job Thủ công

Thỉnh thoảng bạn có thể cần đánh dấu thủ công một job là "thất bại". Để làm điều này, bạn có thể gọi phương thức `fail`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->fail();
}
```

Nếu bạn muốn đánh dấu job của mình là thất bại do một ngoại lệ mà bạn đã bắt, bạn có thể chuyển ngoại lệ cho phương thức `fail`. Hoặc, để thuận tiện, bạn có thể chuyển một thông báo lỗi chuỗi sẽ được chuyển đổi thành ngoại lệ cho bạn:

```php
$this->fail($exception);

$this->fail('Something went wrong.');
```

> [!NOTE]
> Để biết thêm thông tin về các job thất bại, hãy xem [tài liệu về xử lý các thất bại của job](#dealing-with-failed-jobs).

<a name="fail-jobs-on-exceptions"></a>
#### Thất bại Jobs trên Ngoại lệ Cụ thể

[Job middleware](#job-middleware) `FailOnException` cho phép bạn rút ngắn các lần thử lại khi các ngoại lệ cụ thể được ném ra. Điều này cho phép thử lại trên các ngoại lệ tạm thời như lỗi API bên ngoài, nhưng thất bại job vĩnh viễn trên các ngoại lệ persists, như quyền của người dùng bị thu hồi:

```php
<?php

namespace App\Jobs;

use App\Models\User;
use Illuminate\Auth\Access\AuthorizationException;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\Middleware\FailOnException;
use Illuminate\Support\Facades\Http;

#[Tries(3)]
class SyncChatHistory implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        $this->user->authorize('sync-chat-history');

        $response = Http::throw()->get(
            "https://chat.laravel.test/?user={$this->user->uuid}"
        );

        // ...
    }

    /**
     * Get the middleware the job should pass through.
     */
    public function middleware(): array
    {
        return [
            new FailOnException([AuthorizationException::class])
        ];
    }
}
```

<a name="job-batching"></a>
## Job Batching

Tính năng job batching của Laravel cho phép bạn dễ dàng thực thi một nhóm job song song và sau đó thực hiện một số hành động khi batch của các job đã hoàn thành việc thực thi.

Trước khi bắt đầu, bạn nên tạo một migration cơ sở dữ liệu để xây dựng một bảng sẽ chứa thông tin meta về các batch job của bạn, chẳng hạn như phần trăm hoàn thành của chúng. Migration này có thể được tạo bằng lệnh Artisan `make:queue-batches-table`:

```shell
php artisan make:queue-batches-table

php artisan migrate
```

<a name="defining-batchable-jobs"></a>
### Định nghĩa Batchable Jobs

Để định nghĩa một batchable job, bạn nên [tạo một queueable job](#creating-jobs) như bình thường; tuy nhiên, bạn nên thêm trait `Illuminate\Bus\Batchable` vào class job. Trait này cung cấp quyền truy cập vào phương thức `batch` có thể được sử dụng để truy xuất batch hiện tại mà job đang thực thi trong đó:

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportCsv implements ShouldQueue
{
    use Batchable, Queueable;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            // Determine if the batch has been cancelled...

            return;
        }

        // Import a portion of the CSV file...
    }
}
```

<a name="dispatching-batches"></a>
### Dispatching Batches

Để dispatch một batch job, bạn nên sử dụng phương thức `batch` của facade `Bus`. Tất nhiên, batching chủ yếu hữu ích khi kết hợp với các callback hoàn thành. Vì vậy, bạn có thể sử dụng các phương thức `then`, `catch` và `finally` để định nghĩa các callback hoàn thành cho batch. Mỗi callback này sẽ nhận một instance `Illuminate\Bus\Batch` khi chúng được gọi.

Khi chạy nhiều worker hàng đợi, các job trong batch sẽ được xử lý song song. Do đó, thứ tự mà các job hoàn thành có thể không giống với thứ tự mà chúng được thêm vào batch. Tham khảo tài liệu của chúng tôi về [job chains và batches](#chains-and-batches) để biết thông tin về cách chạy một chuỗi job theo tuần tự.

Trong ví dụ này, chúng ta sẽ tưởng tượng rằng chúng ta đang xếp hàng đợi một batch job mà mỗi job xử lý một số hàng nhất định từ một file CSV:

```php
use App\Jobs\ImportCsv;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch([
    new ImportCsv(1, 100),
    new ImportCsv(101, 200),
    new ImportCsv(201, 300),
    new ImportCsv(301, 400),
    new ImportCsv(401, 500),
])->before(function (Batch $batch) {
    // The batch has been created but no jobs have been added...
})->progress(function (Batch $batch) {
    // A single job has completed successfully...
})->then(function (Batch $batch) {
    // All jobs completed successfully...
})->catch(function (Batch $batch, Throwable $e) {
    // Batch job failure detected...
})->finally(function (Batch $batch) {
    // The batch has finished executing...
})->dispatch();

return $batch->id;
```

ID của batch, có thể được truy cập thông qua thuộc tính `$batch->id`, có thể được sử dụng để [query Laravel command bus](#inspecting-batches) để biết thông tin về batch sau khi nó đã được dispatch.

> [!WARNING]
> Vì các callback batch được serialize và thực thi tại một thời điểm sau bởi hàng đợi Laravel, bạn không nên sử dụng biến `$this` trong các callback. Ngoài ra, vì các batched job được bọc trong các giao dịch cơ sở dữ liệu, các câu lệnh cơ sở dữ liệu kích hoạt các commit ngầm định không nên được thực thi trong các job.

<a name="naming-batches"></a>
#### Đặt tên Batches

Một số công cụ như [Laravel Horizon](/docs/{{version}}/horizon) và [Laravel Telescope](/docs/{{version}}/telescope) có thể cung cấp thông tin debug thân thiện với người dùng hơn cho các batch nếu các batch được đặt tên. Để gán một tên tùy ý cho một batch, bạn có thể gọi phương thức `name` khi định nghĩa batch:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import CSV')->dispatch();
```

<a name="batch-connection-queue"></a>
#### Kết nối và Hàng đợi Batch

Nếu bạn muốn chỉ định kết nối và hàng đợi nên được sử dụng cho các batched job, bạn có thể sử dụng các phương thức `onConnection` và `onQueue`. Tất cả các batched job phải thực thi trong cùng một kết nối và hàng đợi:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->onConnection('redis')->onQueue('imports')->dispatch();
```

<a name="chains-and-batches"></a>
### Chains và Batches

Bạn có thể định nghĩa một tập hợp [chained jobs](#job-chaining) trong một batch bằng cách đặt các chained jobs trong một mảng. Ví dụ, chúng ta có thể thực thi hai chuỗi job song song và thực thi một callback khi cả hai chuỗi job đã hoàn thành xử lý:

```php
use App\Jobs\ReleasePodcast;
use App\Jobs\SendPodcastReleaseNotification;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

Bus::batch([
    [
        new ReleasePodcast(1),
        new SendPodcastReleaseNotification(1),
    ],
    [
        new ReleasePodcast(2),
        new SendPodcastReleaseNotification(2),
    ],
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->dispatch();
```

Ngược lại, bạn có thể chạy các batch job trong một [chain](#job-chaining) bằng cách định nghĩa các batch trong chuỗi. Ví dụ, bạn có thể chạy trước một batch job để phát hành nhiều podcast sau đó một batch job để gửi các thông báo phát hành:

```php
use App\Jobs\FlushPodcastCache;
use App\Jobs\ReleasePodcast;
use App\Jobs\SendPodcastReleaseNotification;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new FlushPodcastCache,
    Bus::batch([
        new ReleasePodcast(1),
        new ReleasePodcast(2),
    ]),
    Bus::batch([
        new SendPodcastReleaseNotification(1),
        new SendPodcastReleaseNotification(2),
    ]),
])->dispatch();
```

<a name="adding-jobs-to-batches"></a>
### Thêm Jobs vào Batches

Đôi khi có thể hữu ích để thêm các job bổ sung vào một batch từ trong một batched job. Mẫu này có thể hữu ích khi bạn cần batch hàng nghìn job có thể mất quá nhiều thời gian để dispatch trong một yêu cầu web. Vì vậy, thay vào đó, bạn có thể muốn dispatch một batch ban đầu của các job "loader" hydrate batch với nhiều job hơn:

```php
$batch = Bus::batch([
    new LoadImportBatch,
    new LoadImportBatch,
    new LoadImportBatch,
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import Contacts')->dispatch();
```

Trong ví dụ này, chúng ta sẽ sử dụng job `LoadImportBatch` để hydrate batch với các job bổ sung. Để thực hiện điều này, chúng ta có thể sử dụng phương thức `add` trên instance batch có thể được truy cập thông qua phương thức `batch` của job:

```php
use App\Jobs\ImportContacts;
use Illuminate\Support\Collection;

/**
 * Execute the job.
 */
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }

    $this->batch()->add(Collection::times(1000, function () {
        return new ImportContacts;
    }));
}
```

> [!WARNING]
> Bạn chỉ có thể thêm job vào một batch từ trong một job thuộc về cùng một batch.

<a name="inspecting-batches"></a>
### Kiểm tra Batches

Instance `Illuminate\Bus\Batch` được cung cấp cho các callback hoàn thành batch có nhiều thuộc tính và phương thức để giúp bạn tương tác và kiểm tra một batch job nhất định:

```php
// The UUID of the batch...
$batch->id;

// The name of the batch (if applicable)...
$batch->name;

// The number of jobs assigned to the batch...
$batch->totalJobs;

// The number of jobs that have not been processed by the queue...
$batch->pendingJobs;

// The number of jobs that have failed...
$batch->failedJobs;

// The number of jobs that have been processed thus far...
$batch->processedJobs();

// The completion percentage of the batch (0-100)...
$batch->progress();

// Indicates if the batch has finished executing...
$batch->finished();

// Cancel the execution of the batch...
$batch->cancel();

// Indicates if the batch has been cancelled...
$batch->cancelled();
```

<a name="returning-batches-from-routes"></a>
#### Trả về Batches từ Routes

Tất cả các instance `Illuminate\Bus\Batch` đều có thể serialize JSON, nghĩa là bạn có thể trả về chúng trực tiếp từ một trong các route của ứng dụng để truy xuất một payload JSON chứa thông tin về batch, bao gồm tiến độ hoàn thành của nó. Điều này giúp thuận tiện để hiển thị thông tin về tiến độ hoàn thành của batch trong UI của ứng dụng của bạn.

Để truy xuất một batch theo ID của nó, bạn có thể sử dụng phương thức `findBatch` của facade `Bus`:

```php
use Illuminate\Support\Facades\Bus;
use Illuminate\Support\Facades\Route;

Route::get('/batch/{batchId}', function (string $batchId) {
    return Bus::findBatch($batchId);
});
```

<a name="cancelling-batches"></a>
### Hủy Batches

Đôi khi bạn có thể cần hủy việc thực thi của một batch nhất định. Điều này có thể được thực hiện bằng cách gọi phương thức `cancel` trên instance `Illuminate\Bus\Batch`:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    if ($this->user->exceedsImportLimit()) {
        $this->batch()->cancel();

        return;
    }

    if ($this->batch()->cancelled()) {
        return;
    }
}
```

Như bạn có thể nhận thấy trong các ví dụ trước, các batched job thường nên xác định xem batch tương ứng của chúng đã bị hủy hay không trước khi tiếp tục thực thi. Tuy nhiên, để thuận tiện, bạn có thể gán [middleware](#job-middleware) `SkipIfBatchCancelled` cho job thay thế. Như tên gọi của nó, middleware này sẽ hướng dẫn Laravel không xử lý job nếu batch tương ứng của nó đã bị hủy:

```php
use Illuminate\Queue\Middleware\SkipIfBatchCancelled;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [new SkipIfBatchCancelled];
}
```

<a name="batch-failures"></a>
### Thất bại Batch

Khi một batched job thất bại, callback `catch` (nếu được gán) sẽ được gọi. Callback này chỉ được gọi cho job đầu tiên thất bại trong batch.

<a name="allowing-failures"></a>
#### Cho phép Thất bại

Khi một job trong một batch thất bại, Laravel sẽ tự động đánh dấu batch là "đã hủy". Nếu bạn muốn, bạn có thể tắt hành vi này để một thất bại job không tự động đánh dấu batch là đã hủy. Điều này có thể được thực hiện bằng cách gọi phương thức `allowFailures` khi dispatch batch:

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->allowFailures()->dispatch();
```

Bạn có thể tùy chọn cung cấp một closure cho phương thức `allowFailures`, sẽ được thực thi trên mỗi thất bại job:

```php
$batch = Bus::batch([
    // ...
])->allowFailures(function (Batch $batch, $exception) {
    // Handle individual job failures...
})->dispatch();
```

<a name="retrying-failed-batch-jobs"></a>
#### Thử lại Các Batch Job Thất bại

Để thuận tiện, Laravel cung cấp lệnh Artisan `queue:retry-batch` cho phép bạn dễ dàng thử lại tất cả các job thất bại cho một batch nhất định. Lệnh này chấp nhận UUID của batch mà các job thất bại của nó nên được thử lại:

```shell
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5
```

<a name="pruning-batches"></a>
### Pruning Batches

Nếu không có pruning, bảng `job_batches` có thể tích lũy các bản ghi rất nhanh. Để giảm thiểu điều này, bạn nên [lên lịch](/docs/{{version}}/scheduling) lệnh Artisan `queue:prune-batches` để chạy hàng ngày:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches')->daily();
```

Theo mặc định, tất cả các batch đã hoàn thành cũ hơn 24 giờ sẽ được pruning. Bạn có thể sử dụng tùy chọn `hours` khi gọi lệnh để xác định thời gian giữ dữ liệu batch. Ví dụ, lệnh sau sẽ xóa tất cả các batch đã hoàn thành hơn 48 giờ trước:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48')->daily();
```

Đôi khi, bảng `job_batches` của bạn có thể tích lũy các bản ghi batch cho các batch không bao giờ hoàn thành thành công, chẳng hạn như các batch mà một job thất bại và job đó không bao giờ được thử lại thành công. Bạn có thể hướng dẫn lệnh `queue:prune-batches` để pruning các bản ghi batch chưa hoàn thành này bằng tùy chọn `unfinished`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --unfinished=72')->daily();
```

Tương tự, bảng `job_batches` của bạn cũng có thể tích lũy các bản ghi batch cho các batch đã bị hủy. Bạn có thể hướng dẫn lệnh `queue:prune-batches` để pruning các bản ghi batch đã bị hủy này bằng tùy chọn `cancelled`:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --cancelled=72')->daily();
```

<a name="storing-batches-in-dynamodb"></a>
### Lưu trữ Batches trong DynamoDB

Laravel cũng cung cấp hỗ trợ để lưu trữ thông tin meta batch trong [DynamoDB](https://aws.amazon.com/dynamodb) thay vì cơ sở dữ liệu quan hệ. Tuy nhiên, bạn sẽ cần tạo thủ công một bảng DynamoDB để lưu trữ tất cả các bản ghi batch.

Thông thường, bảng này nên được đặt tên là `job_batches`, nhưng bạn nên đặt tên bảng dựa trên giá trị cấu hình `queue.batching.table` trong file cấu hình `queue` của ứng dụng của bạn.

<a name="dynamodb-batch-table-configuration"></a>
#### Cấu hình Bảng DynamoDB Batch

Bảng `job_batches` nên có một khóa phân vùng chính chuỗi tên là `application` và một khóa sắp xếp chính chuỗi tên là `id`. Phần `application` của khóa sẽ chứa tên ứng dụng của bạn như được định nghĩa bởi giá trị cấu hình `name` trong file cấu hình `app` của ứng dụng của bạn. Vì tên ứng dụng là một phần của khóa của bảng DynamoDB, bạn có thể sử dụng cùng một bảng để lưu trữ các batch job cho nhiều ứng dụng Laravel.

Ngoài ra, bạn có thể định nghĩa thuộc tính `ttl` cho bảng của bạn nếu bạn muốn tận dụng [batch pruning tự động](#pruning-batches-in-dynamodb).

<a name="dynamodb-configuration"></a>
#### Cấu hình DynamoDB

Tiếp theo, cài đặt AWS SDK để ứng dụng Laravel của bạn có thể giao tiếp với Amazon DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Sau đó, đặt giá trị tùy chọn cấu hình `queue.batching.driver` thành `dynamodb`. Ngoài ra, bạn nên định nghĩa các tùy chọn cấu hình `key`, `secret` và `region` trong mảng cấu hình `batching`. Các tùy chọn này sẽ được sử dụng để xác thực với AWS. Khi sử dụng driver `dynamodb`, tùy chọn cấu hình `queue.batching.database` là không cần thiết:

```php
'batching' => [
    'driver' => env('QUEUE_BATCHING_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
],
```

<a name="pruning-batches-in-dynamodb"></a>
#### Pruning Batches trong DynamoDB

Khi sử dụng [DynamoDB](https://aws.amazon.com/dynamodb) để lưu trữ thông tin batch job, các lệnh pruning điển hình được sử dụng để pruning các batch được lưu trữ trong cơ sở dữ liệu quan hệ sẽ không hoạt động. Thay vào đó, bạn có thể sử dụng [chức năng TTL gốc của DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html) để tự động xóa các bản ghi cho các batch cũ.

Nếu bạn định nghĩa bảng DynamoDB của mình với thuộc tính `ttl`, bạn có thể định nghĩa các tham số cấu hình để hướng dẫn Laravel cách pruning các bản ghi batch. Giá trị cấu hình `queue.batching.ttl_attribute` định nghĩa tên của thuộc tính giữ TTL, trong khi giá trị cấu hình `queue.batching.ttl` định nghĩa số giây sau đó một bản ghi batch có thể được xóa khỏi bảng DynamoDB, tương đối với lần cuối cùng bản ghi được cập nhật:

```php
'batching' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
    'ttl_attribute' => 'ttl',
    'ttl' => 60 * 60 * 24 * 7, // 7 days...
],
```

<a name="queueing-closures"></a>
## Queueing Closures

Thay vì dispatch một class job vào hàng đợi, bạn cũng có thể dispatch một closure. Điều này rất tốt cho các tác vụ nhanh, đơn giản cần được thực thi bên ngoài chu kỳ yêu cầu hiện tại. Khi dispatch các closure vào hàng đợi, nội dung mã của closure được ký tên mật mã để nó không thể được sửa đổi trong quá trình truyền:

```php
use App\Models\Podcast;

$podcast = Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
```

Để gán một tên cho queued closure có thể được sử dụng bởi các bảng điều khiển báo cáo hàng đợi, cũng như được hiển thị bởi lệnh `queue:work`, bạn có thể sử dụng phương thức `name`:

```php
dispatch(function () {
    // ...
})->name('Publish Podcast');
```

Sử dụng phương thức `catch`, bạn có thể cung cấp một closure nên được thực thi nếu queued closure thất bại trong việc hoàn thành thành công sau khi hết tất cả [các lần thử lại được cấu hình](#max-job-attempts-and-timeout) của hàng đợi của bạn:

```php
use Throwable;
dispatch(function () use ($podcast) {
    $podcast->publish();
})->catch(function (Throwable $e) {
    // This job has failed...
});
```

> [!WARNING]
> Since `catch` callbacks are serialized and executed at a later time by the Laravel queue, you should not use the `$this` variable within `catch` callbacks.

<a name="running-the-queue-worker"></a>
## Chạy Queue Worker

<a name="the-queue-work-command"></a>
### Lệnh `queue:work`

Laravel bao gồm một lệnh Artisan sẽ khởi động một queue worker và xử lý các job mới khi chúng được đẩy vào queue. Bạn có thể chạy worker bằng lệnh Artisan `queue:work`. Lưu ý rằng khi lệnh `queue:work` đã được khởi động, nó sẽ tiếp tục chạy cho đến khi được dừng thủ công hoặc bạn đóng terminal:

```shell
php artisan queue:work
```

> [!NOTE]
> Để giữ cho tiến trình `queue:work` chạy vĩnh viễn trong nền, bạn nên sử dụng một trình giám sát tiến trình như [Supervisor](#supervisor-configuration) để đảm bảo rằng queue worker không dừng chạy.

Bạn có thể thêm cờ `-v` khi gọi lệnh `queue:work` nếu bạn muốn các ID job đã xử lý, tên kết nối và tên queue được bao gồm trong đầu ra của lệnh:

```shell
php artisan queue:work -v
```

Hãy nhớ rằng, queue workers là các tiến trình chạy lâu dài và lưu trữ trạng thái ứng dụng đã khởi động trong bộ nhớ. Kết quả là, chúng sẽ không nhận thấy các thay đổi trong cơ sở mã của bạn sau khi chúng đã được khởi động. Vì vậy, trong quá trình triển khai của bạn, hãy đảm bảo [khởi động lại queue workers của bạn](#queue-workers-and-deployment). Ngoài ra, hãy nhớ rằng bất kỳ trạng thái tĩnh nào được tạo hoặc sửa đổi bởi ứng dụng của bạn sẽ không được tự động đặt lại giữa các job.

Ngoài ra, bạn có thể chạy lệnh `queue:listen`. Khi sử dụng lệnh `queue:listen`, bạn không phải khởi động lại worker thủ công khi bạn muốn tải lại mã đã cập nhật hoặc đặt lại trạng thái ứng dụng; tuy nhiên, lệnh này kém hiệu quả hơn đáng kể so với lệnh `queue:work`:

```shell
php artisan queue:listen
```

<a name="running-multiple-queue-workers"></a>
#### Chạy Nhiều Queue Workers

Để gán nhiều worker cho một queue và xử lý các job đồng thời, bạn chỉ cần khởi động nhiều tiến trình `queue:work`. Điều này có thể được thực hiện cục bộ thông qua nhiều tab trong terminal của bạn hoặc trong môi trường sản xuất bằng cách sử dụng cài đặt cấu hình của trình quản lý tiến trình của bạn. [Khi sử dụng Supervisor](#supervisor-configuration), bạn có thể sử dụng giá trị cấu hình `numprocs`.

<a name="specifying-the-connection-queue"></a>
#### Chỉ định Kết Nối và Queue

Bạn cũng có thể chỉ định kết nối queue mà worker nên sử dụng. Tên kết nối được truyền cho lệnh `work` phải tương ứng với một trong các kết nối được định nghĩa trong tệp cấu hình `config/queue.php` của bạn:

```shell
php artisan queue:work redis
```

Theo mặc định, lệnh `queue:work` chỉ xử lý các job cho queue mặc định trên một kết nối nhất định. Tuy nhiên, bạn có thể tùy chỉnh queue worker của mình xa hơn nữa bằng cách chỉ xử lý các queue cụ thể cho một kết nối nhất định. Ví dụ, nếu tất cả email của bạn được xử lý trong một queue `emails` trên kết nối queue `redis` của bạn, bạn có thể phát hành lệnh sau để khởi động một worker chỉ xử lý queue đó:

```shell
php artisan queue:work redis --queue=emails
```

<a name="processing-a-specified-number-of-jobs"></a>
#### Xử lý Số Lượng Job Cụ Thể

Tùy chọn `--once` có thể được sử dụng để hướng dẫn worker chỉ xử lý một job duy nhất từ queue:

```shell
php artisan queue:work --once
```

Tùy chọn `--max-jobs` có thể được sử dụng để hướng dẫn worker xử lý số lượng job nhất định và sau đó thoát. Tùy chọn này có thể hữu ích khi kết hợp với [Supervisor](#supervisor-configuration) để các worker của bạn được tự động khởi động lại sau khi xử lý một số lượng job nhất định, giải phóng bất kỳ bộ nhớ nào mà chúng có thể đã tích lũy:

```shell
php artisan queue:work --max-jobs=1000
```

<a name="processing-all-queued-jobs-then-exiting"></a>
#### Xử lý Tất Cả Job Trong Queue và Sau Đó Thoát

Tùy chọn `--stop-when-empty` có thể được sử dụng để hướng dẫn worker xử lý tất cả các job và sau đó thoát một cách êm đẹp. Tùy chọn này có thể hữu ích khi xử lý các queue Laravel trong một container Docker nếu bạn muốn tắt container sau khi queue trống:

```shell
php artisan queue:work --stop-when-empty
```

<a name="processing-jobs-for-a-given-number-of-seconds"></a>
#### Xử lý Job Trong Một Số Giây Cụ Thể

Tùy chọn `--max-time` có thể được sử dụng để hướng dẫn worker xử lý các job trong số giây nhất định và sau đó thoát. Tùy chọn này có thể hữu ích khi kết hợp với [Supervisor](#supervisor-configuration) để các worker của bạn được tự động khởi động lại sau khi xử lý các job trong một khoảng thời gian nhất định, giải phóng bất kỳ bộ nhớ nào mà chúng có thể đã tích lũy:

```shell
# Process jobs for one hour and then exit...
php artisan queue:work --max-time=3600
```

<a name="worker-sleep-duration"></a>
#### Thời Gian Sleep Của Worker

Khi các job có sẵn trên queue, worker sẽ tiếp tục xử lý các job mà không có độ trễ giữa các job. Tuy nhiên, tùy chọn `sleep` xác định số giây mà worker sẽ "ngủ" nếu không có job nào có sẵn. Tất nhiên, trong khi ngủ, worker sẽ không xử lý bất kỳ job mới nào:

```shell
php artisan queue:work --sleep=3
```

<a name="maintenance-mode-queues"></a>
#### Chế Độ Bảo Trì và Queues

Trong khi ứng dụng của bạn ở trong [chế độ bảo trì](/docs/{{version}}/configuration#maintenance-mode), không có job nào trong queue sẽ được xử lý. Các job sẽ tiếp tục được xử lý như bình thường khi ứng dụng thoát khỏi chế độ bảo trì.

Để buộc queue workers của bạn xử lý các job ngay cả khi chế độ bảo trì được bật, bạn có thể sử dụng tùy chọn `--force`:

```shell
php artisan queue:work --force
```

<a name="resource-considerations"></a>
#### Xem Xét Tài Nguyên

Queue workers daemon không "khởi động lại" framework trước khi xử lý mỗi job. Do đó, bạn nên giải phóng bất kỳ tài nguyên nặng nào sau khi mỗi job hoàn thành. Ví dụ, nếu bạn đang thao tác hình ảnh với [thư viện GD](https://www.php.net/manual/en/book.image.php), bạn nên giải phóng bộ nhớ với `imagedestroy` khi bạn hoàn tất xử lý hình ảnh.

<a name="queue-priorities"></a>
### Ưu Tiên Queue

Đôi khi bạn có thể muốn ưu tiên cách các queue của bạn được xử lý. Ví dụ, trong tệp cấu hình `config/queue.php` của bạn, bạn có thể đặt `queue` mặc định cho kết nối `redis` của bạn thành `low`. Tuy nhiên, đôi khi bạn có thể muốn đẩy một job vào một queue ưu tiên `high` như sau:

```php
dispatch((new Job)->onQueue('high'));
```

Để khởi động một worker xác minh rằng tất cả các job queue `high` được xử lý trước khi tiếp tục với bất kỳ job nào trên queue `low`, hãy chuyển danh sách tên queue được phân tách bằng dấu phẩy cho lệnh `work`:

```shell
php artisan queue:work --queue=high,low
```

<a name="queue-workers-and-deployment"></a>
### Queue Workers và Triển Khai

Vì queue workers là các tiến trình chạy lâu dài, chúng sẽ không nhận thấy các thay đổi đối với mã của bạn nếu không được khởi động lại. Vì vậy, cách đơn giản nhất để triển khai một ứng dụng sử dụng queue workers là khởi động lại các worker trong quá trình triển khai của bạn. Bạn có thể khởi động lại một cách êm đẹp tất cả các worker bằng cách phát hành lệnh `queue:restart`:

```shell
php artisan queue:restart
```

Lệnh này sẽ hướng dẫn tất cả queue workers thoát một cách êm đẹp sau khi chúng hoàn tất xử lý job hiện tại của họ để không có job hiện tại nào bị mất. Vì queue workers sẽ thoát khi lệnh `queue:restart` được thực thi, bạn nên chạy một trình quản lý tiến trình như [Supervisor](#supervisor-configuration) để tự động khởi động lại các queue workers.

> [!NOTE]
> Queue sử dụng [cache](/docs/{{version}}/cache) để lưu trữ các tín hiệu khởi động lại, vì vậy bạn nên xác minh rằng một cache driver được cấu hình đúng cho ứng dụng của bạn trước khi sử dụng tính năng này.

<a name="reacting-to-worker-signals"></a>
### Phản Hồi Với Tín Hiệu Worker

Khi một queue worker nhận được tín hiệu chấm dứt như `SIGQUIT`, `SIGTERM`, hoặc `SIGINT` trong khi xử lý một job, worker sẽ hoàn thành job hiện tại của nó trước khi thoát. Tuy nhiên, job của bạn có thể cần phản hồi với tín hiệu trước khi tiến trình bị dừng bởi máy chủ hoặc trình điều phối container của bạn. Ví dụ, một job nhập khẩu chạy dài có thể cần dừng kéo các bản ghi mới và lưu tiến trình hiện tại của nó.

Để phản hồi với các tín hiệu worker từ trong một job, hãy triển khai giao diện `Illuminate\Contracts\Queue\Interruptible` và định nghĩa một phương thức `interrupted` trên job của bạn. Số tín hiệu nhận được bởi worker sẽ được chuyển cho phương thức `interrupted`:

```php
<?php

namespace App\Jobs;

use App\Models\Import;
use Illuminate\Contracts\Queue\Interruptible;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportProducts implements ShouldQueue, Interruptible
{
    use Queueable;

    protected bool $shouldStop = false;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Import $import,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        foreach ($this->import->pendingRows() as $row) {
            if ($this->shouldStop) {
                break;
            }

            // Import the product row...
        }

        $this->import->saveProgress();
    }

    /**
     * Handle a signal received by the queue worker.
     */
    public function interrupted(int $signal): void
    {
        $this->shouldStop = true;
    }
}
```

Phương thức `interrupted` chỉ được gọi khi worker nhận được tín hiệu tiến trình trong khi job hiện đang chạy. Nó không thay thế cho [timeouts](#worker-timeouts) hoặc [phương thức `failed`](#cleaning-up-after-failed-jobs) của job.

<a name="job-expirations-and-timeouts"></a>
### Hết Hạn Job và Timeouts

<a name="job-expiration"></a>
#### Hết Hạn Job

Trong tệp cấu hình `config/queue.php` của bạn, mỗi kết nối queue định nghĩa một tùy chọn `retry_after`. Tùy chọn này chỉ định số giây mà kết nối queue nên đợi trước khi thử lại một job đang được xử lý. Ví dụ, nếu giá trị của `retry_after` được đặt thành `90`, job sẽ được giải phóng trở lại queue nếu nó đã được xử lý trong 90 giây mà không được giải phóng hoặc xóa. Thông thường, bạn nên đặt giá trị `retry_after` thành số giây tối đa mà các job của bạn nên mất để hoàn tất xử lý một cách hợp lý.

> [!WARNING]
> Kết nối queue duy nhất không chứa giá trị `retry_after` là Amazon SQS. SQS sẽ thử lại job dựa trên [Default Visibility Timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) được quản lý trong bảng điều khiển AWS.

<a name="worker-timeouts"></a>
#### Worker Timeouts

Lệnh Artisan `queue:work` cung cấp tùy chọn `--timeout`. Theo mặc định, giá trị `--timeout` là 60 giây. Nếu một job đang được xử lý lâu hơn số giây được chỉ định bởi giá trị timeout, worker xử lý job sẽ thoát với lỗi. Thông thường, worker sẽ được tự động khởi động lại bởi [trình quản lý tiến trình được cấu hình trên máy chủ của bạn](#supervisor-configuration):

```shell
php artisan queue:work --timeout=60
```

Tùy chọn cấu hình `retry_after` và tùy chọn CLI `--timeout` khác nhau, nhưng hoạt động cùng nhau để đảm bảo rằng các job không bị mất và các job chỉ được xử lý thành công một lần.

> [!WARNING]
> Giá trị `--timeout` phải luôn ngắn hơn ít nhất vài giây so với giá trị cấu hình `retry_after` của bạn. Điều này sẽ đảm bảo rằng một worker xử lý một job bị đóng băng luôn bị chấm dứt trước khi job được thử lại. Nếu tùy chọn `--timeout` của bạn dài hơn giá trị cấu hình `retry_after` của bạn, các job của bạn có thể được xử lý hai lần.

<a name="pausing-and-resuming-queue-workers"></a>
### Tạm Dừng và Tiếp Tục Queue Workers

Đôi khi bạn có thể cần tạm thời ngăn chặn một queue worker xử lý các job mới mà không dừng worker hoàn toàn. Ví dụ, bạn có thể muốn tạm dừng xử lý job trong quá trình bảo trì hệ thống. Laravel cung cấp các lệnh Artisan `queue:pause` và `queue:continue` để tạm dừng và tiếp tục queue workers.

Để tạm dừng một queue cụ thể, hãy cung cấp tên kết nối queue và tên queue:

```shell
php artisan queue:pause database:default
```

Trong ví dụ này, `database` là tên kết nối queue và `default` là tên queue. Khi một queue bị tạm dừng, bất kỳ worker nào xử lý các job từ queue đó sẽ tiếp tục hoàn thành job hiện tại của họ, nhưng sẽ không nhận bất kỳ job mới nào cho đến khi queue được tiếp tục.

Để tiếp tục xử lý các job trên một queue bị tạm dừng, hãy sử dụng lệnh `queue:continue`:

```shell
php artisan queue:continue database:default
```

Sau khi tiếp tục một queue, các worker sẽ bắt đầu xử lý các job mới từ queue đó ngay lập tức. Lưu ý rằng việc tạm dừng một queue không dừng chính tiến trình worker - nó chỉ ngăn chặn worker xử lý các job mới từ queue được chỉ định.

<a name="worker-restart-and-pause-signals"></a>
#### Tín Hiệu Khởi Động Lại và Tạm Dừng Worker

Theo mặc định, queue workers kiểm tra cache driver để tìm các tín hiệu khởi động lại và tạm dừng trên mỗi lần lặp job. Mặc dù việc kiểm tra này rất cần thiết để phản hồi với các lệnh `queue:restart` và `queue:pause`, nó có giới thiệu một chi phí hiệu suất nhỏ.

Nếu bạn cần tối ưu hóa hiệu suất và không yêu cầu các tính năng gián đoạn này, bạn có thể vô hiệu hóa việc kiểm tra này toàn cục bằng cách gọi phương thức `withoutInterruptionPolling` trên facade `Queue`. Điều này thường nên được thực hiện trong phương thức `boot` của `AppServiceProvider` của bạn:

```php
use Illuminate\Support\Facades\Queue;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Queue::withoutInterruptionPolling();
}
```

Ngoài ra, bạn có thể vô hiệu hóa kiểm tra khởi động lại hoặc tạm dừng riêng lẻ bằng cách đặt các thuộc tính tĩnh `$restartable` hoặc `$pausable` trên lớp `Illuminate\Queue\Worker`:

```php
use Illuminate\Queue\Worker;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Worker::$restartable = false;
    Worker::$pausable = false;
}
```

> [!WARNING]
> Khi kiểm tra gián đoạn bị vô hiệu hóa, workers sẽ không phản hồi với các lệnh `queue:restart` hoặc `queue:pause` (tùy thuộc vào tính năng nào bị vô hiệu hóa).

<a name="supervisor-configuration"></a>
## Cấu Hình Supervisor

Trong môi trường sản xuất, bạn cần một cách để giữ cho các tiến trình `queue:work` của bạn chạy. Một tiến trình `queue:work` có thể dừng chạy vì nhiều lý do, chẳng hạn như timeout worker bị vượt quá hoặc thực thi lệnh `queue:restart`.

Vì lý do này, bạn cần cấu hình một trình giám sát tiến trình có thể phát hiện khi các tiến trình `queue:work` của bạn thoát và tự động khởi động lại chúng. Ngoài ra, trình giám sát tiến trình có thể cho phép bạn chỉ định số lượng tiến trình `queue:work` mà bạn muốn chạy đồng thời. Supervisor là một trình giám sát tiến trình thường được sử dụng trong môi trường Linux và chúng ta sẽ thảo luận cách cấu hình nó trong tài liệu sau.

<a name="installing-supervisor"></a>
#### Cài Đặt Supervisor

Supervisor là một trình giám sát tiến trình cho hệ điều hành Linux và sẽ tự động khởi động lại các tiến trình `queue:work` của bạn nếu chúng thất bại. Để cài đặt Supervisor trên Ubuntu, bạn có thể sử dụng lệnh sau:

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> Nếu việc cấu hình và quản lý Supervisor nghe có vẻ quá phức tạp, hãy cân nhắc sử dụng [Laravel Cloud](https://cloud.laravel.com), cung cấp một nền tảng được quản lý hoàn toàn để chạy Laravel queue workers.

<a name="configuring-supervisor"></a>
#### Cấu Hình Supervisor

Các tệp cấu hình Supervisor thường được lưu trữ trong thư mục `/etc/supervisor/conf.d`. Trong thư mục này, bạn có thể tạo bất kỳ số lượng tệp cấu hình nào hướng dẫn supervisor cách các tiến trình của bạn nên được giám sát. Ví dụ, hãy tạo một tệp `laravel-worker.conf` khởi động và giám sát các tiến trình `queue:work`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

Trong ví dụ này, chỉ thị `numprocs` sẽ hướng dẫn Supervisor chạy tám tiến trình `queue:work` và giám sát tất cả chúng, tự động khởi động lại chúng nếu chúng thất bại. Bạn nên thay đổi chỉ thị `command` của cấu hình để phản ánh kết nối queue và tùy chọn worker mong muốn của bạn.

> [!WARNING]
> Bạn nên đảm bảo rằng giá trị của `stopwaitsecs` lớn hơn số giây tiêu thụ bởi job chạy dài nhất của bạn. Nếu không, Supervisor có thể giết job trước khi nó hoàn tất xử lý.

<a name="starting-supervisor"></a>
#### Khởi Động Supervisor

Sau khi tệp cấu hình đã được tạo, bạn có thể cập nhật cấu hình Supervisor và khởi động các tiến trình bằng cách sử dụng các lệnh sau:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start "laravel-worker:*"
```

Để biết thêm thông tin về Supervisor, hãy tham khảo [tài liệu Supervisor](http://supervisord.org/index.html).

<a name="dealing-with-failed-jobs"></a>
## Xử Lý Các Job Thất Bại

Đôi khi các job trong queue của bạn sẽ thất bại. Đừng lo lắng, mọi thứ không luôn đi theo kế hoạch! Laravel bao gồm một cách thuận tiện để [chỉ định số lần tối đa mà một job nên được thử](#max-job-attempts-and-timeout). Sau khi một job không đồng bộ đã vượt quá số lần này, nó sẽ được chèn vào bảng cơ sở dữ liệu `failed_jobs`. [Các job được dispatch đồng bộ](/docs/{{version}}/queues#synchronous-dispatching) thất bại không được lưu trữ trong bảng này và các ngoại lệ của chúng được xử lý ngay lập tức bởi ứng dụng.

Một migration để tạo bảng `failed_jobs` thường đã có sẵn trong các ứng dụng Laravel mới. Tuy nhiên, nếu ứng dụng của bạn không chứa migration cho bảng này, bạn có thể sử dụng lệnh `make:queue-failed-table` để tạo migration:

```shell
php artisan make:queue-failed-table

php artisan migrate
```

Khi chạy một tiến trình [queue worker](#running-the-queue-worker), bạn có thể chỉ định số lần tối đa mà một job nên được thử bằng cách sử dụng công tắc `--tries` trên lệnh `queue:work`. Nếu bạn không chỉ định giá trị cho tùy chọn `--tries`, các job sẽ chỉ được thử một lần hoặc nhiều lần như được chỉ định bởi thuộc tính `Tries` của lớp job:

```shell
php artisan queue:work redis --tries=3
```

Sử dụng tùy chọn `--backoff`, bạn có thể chỉ định số giây Laravel nên đợi trước khi thử lại một job đã gặp ngoại lệ. Theo mặc định, một job được giải phóng ngay lập tức trở lại queue để nó có thể được thử lại:

```shell
php artisan queue:work redis --tries=3 --backoff=3
```

Nếu bạn muốn cấu hình số giây Laravel nên đợi trước khi thử lại một job đã gặp ngoại lệ trên cơ sở từng job, bạn có thể sử dụng thuộc tính `Backoff` trên lớp job của bạn:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Backoff;

#[Backoff(3)]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

Nếu bạn yêu cầu logic phức tạp hơn để xác định thời gian backoff của job, bạn có thể định nghĩa một phương thức `backoff` trên lớp job của bạn:

```php
/**
 * Calculate the number of seconds to wait before retrying the job.
 */
public function backoff(): int
{
    return 3;
}
```

Bạn có thể dễ dàng cấu hình các backoff "exponential" bằng cách định nghĩa một mảng các giá trị backoff. Trong ví dụ này, độ trễ thử lại sẽ là 1 giây cho lần thử lại đầu tiên, 5 giây cho lần thử lại thứ hai, 10 giây cho lần thử lại thứ ba, và 10 giây cho mỗi lần thử lại tiếp theo nếu còn nhiều lần thử còn lại:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\Backoff;

#[Backoff([1, 5, 10])]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

<a name="cleaning-up-after-failed-jobs"></a>
### Dọn Dẹp Sau Khi Job Thất Bại

Khi một job cụ thể thất bại, bạn có thể muốn gửi cảnh báo cho người dùng của mình hoặc hoàn tác bất kỳ hành động nào đã được hoàn thành một phần bởi job. Để thực hiện điều này, bạn có thể định nghĩa một phương thức `failed` trên lớp job của bạn. Thể hiện `Throwable` gây ra job thất bại sẽ được chuyển cho phương thức `failed`:

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Throwable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }

    /**
     * Handle a job failure.
     */
    public function failed(?Throwable $exception): void
    {
        // Send user notification of failure, etc...
    }
}
```

> [!WARNING]
> Một thể hiện mới của job được khởi tạo trước khi gọi phương thức `failed`; do đó, bất kỳ sửa đổi thuộc tính lớp nào có thể đã xảy ra trong phương thức `handle` sẽ bị mất.

Một job thất bại không nhất thiết là một job gặp ngoại lệ không được xử lý. Một job cũng có thể được coi là thất bại khi nó đã sử dụng hết tất cả các lần thử được phép. Các lần thử này có thể được tiêu thụ theo một số cách:

<div class="content-list" markdown="1">

- Job bị timeout.
- Job gặp ngoại lệ không được xử lý trong quá trình thực thi.
- Job được giải phóng trở lại queue theo cách thủ công hoặc bởi một middleware.

</div>

Nếu lần thử cuối cùng thất bại do một ngoại lệ được ném trong quá trình thực thi job, ngoại lệ đó sẽ được chuyển cho phương thức `failed` của job. Tuy nhiên, nếu job thất bại vì nó đã đạt đến số lần tối đa được phép, `$exception` sẽ là một thể hiện của `Illuminate\Queue\MaxAttemptsExceededException`. Tương tự, nếu job thất bại do vượt quá timeout được cấu hình, `$exception` sẽ là một thể hiện của `Illuminate\Queue\TimeoutExceededException`.

<a name="retrying-failed-jobs"></a>
### Thử Lại Các Job Thất Bại

Để xem tất cả các job thất bại đã được chèn vào bảng cơ sở dữ liệu `failed_jobs` của bạn, bạn có thể sử dụng lệnh Artisan `queue:failed`:

```shell
php artisan queue:failed
```

Lệnh `queue:failed` sẽ liệt kê ID job, kết nối, queue, thời gian thất bại và thông tin khác về job. ID job có thể được sử dụng để thử lại job thất bại. Ví dụ, để thử lại một job thất bại có ID là `ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece`, hãy phát hành lệnh sau:

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece
```

Nếu cần thiết, bạn có thể chuyển nhiều ID cho lệnh:

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece 91401d2c-0784-4f43-824c-34f94a33c24d
```

Bạn cũng có thể thử lại tất cả các job thất bại cho một queue cụ thể:

```shell
php artisan queue:retry --queue=name
```

Để thử lại tất cả các job thất bại của bạn, hãy thực thi lệnh `queue:retry` và chuyển `all` làm ID:

```shell
php artisan queue:retry all
```

Nếu bạn muốn xóa một job thất bại, bạn có thể sử dụng lệnh `queue:forget`:

```shell
php artisan queue:forget 91401d2c-0784-4f43-824c-34f94a33c24d
```

> [!NOTE]
> Khi sử dụng [Horizon](/docs/{{version}}/horizon), bạn nên sử dụng lệnh `horizon:forget` để xóa một job thất bại thay vì lệnh `queue:forget`.

Để xóa tất cả các job thất bại của bạn từ bảng `failed_jobs`, bạn có thể sử dụng lệnh `queue:flush`:

```shell
php artisan queue:flush
```

Lệnh `queue:flush` xóa tất cả các bản ghi job thất bại khỏi queue của bạn, bất kể job thất bại cũ bao nhiêu. Bạn có thể sử dụng tùy chọn `--hours` để chỉ xóa các job đã thất bại một số giờ nhất định trở lên:

```shell
php artisan queue:flush --hours=48
```

<a name="ignoring-missing-models"></a>
### Bỏ Qua Các Model Bị Thiếu

Khi tiêm một model Eloquent vào một job, model được tuần tự hóa tự động trước khi được đặt vào queue và được truy xuất lại từ cơ sở dữ liệu khi job được xử lý. Tuy nhiên, nếu model đã bị xóa trong khi job đang chờ được xử lý bởi một worker, job của bạn có thể thất bại với `ModelNotFoundException`.

Để thuận tiện, bạn có thể chọn tự động xóa các job với model bị thiếu bằng cách sử dụng thuộc tính `DeleteWhenMissingModels` trên lớp job của bạn. Khi thuộc tính này có mặt, Laravel sẽ âm thầm loại bỏ job mà không nêu ra ngoại lệ:

```php
<?php

namespace App\Jobs;

use Illuminate\Queue\Attributes\DeleteWhenMissingModels;

#[DeleteWhenMissingModels]
class ProcessPodcast implements ShouldQueue
{
    // ...
}
```

<a name="pruning-failed-jobs"></a>
### Dọn Dẹp Các Job Thất Bại

Bạn có thể dọn dẹp các bản ghi trong bảng `failed_jobs` của ứng dụng của bạn bằng cách gọi lệnh Artisan `queue:prune-failed`:

```shell
php artisan queue:prune-failed
```

Theo mặc định, tất cả các bản ghi job thất bại cũ hơn 24 giờ sẽ được dọn dẹp. Nếu bạn cung cấp tùy chọn `--hours` cho lệnh, chỉ các bản ghi job thất bại được chèn trong N giờ cuối cùng sẽ được giữ lại. Ví dụ, lệnh sau sẽ xóa tất cả các bản ghi job thất bại được chèn hơn 48 giờ trước:

```shell
php artisan queue:prune-failed --hours=48
```

<a name="storing-failed-jobs-in-dynamodb"></a>
### Lưu Trữ Các Job Thất Bại Trong DynamoDB

Laravel cũng cung cấp hỗ trợ để lưu trữ các bản ghi job thất bại của bạn trong [DynamoDB](https://aws.amazon.com/dynamodb) thay vì một bảng cơ sở dữ liệu quan hệ. Tuy nhiên, bạn phải tạo thủ công một bảng DynamoDB để lưu trữ tất cả các bản ghi job thất bại. Thông thường, bảng này nên được đặt tên là `failed_jobs`, nhưng bạn nên đặt tên bảng dựa trên giá trị cấu hình `queue.failed.table` trong tệp cấu hình `queue` của ứng dụng của bạn.

Bảng `failed_jobs` nên có một khóa phân vùng chuỗi chính tên là `application` và một khóa sắp xếp chuỗi chính tên là `uuid`. Phần `application` của khóa sẽ chứa tên ứng dụng của bạn như được định nghĩa bởi giá trị cấu hình `name` trong tệp cấu hình `app` của ứng dụng của bạn. Vì tên ứng dụng là một phần của khóa của bảng DynamoDB, bạn có thể sử dụng cùng một bảng để lưu trữ các job thất bại cho nhiều ứng dụng Laravel.

Ngoài ra, hãy đảm bảo rằng bạn cài đặt AWS SDK để ứng dụng Laravel của bạn có thể giao tiếp với Amazon DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Tiếp theo, đặt giá trị tùy chọn cấu hình `queue.failed.driver` thành `dynamodb`. Ngoài ra, bạn nên định nghĩa các tùy chọn cấu hình `key`, `secret`, và `region` trong mảng cấu hình job thất bại. Các tùy chọn này sẽ được sử dụng để xác thực với AWS. Khi sử dụng driver `dynamodb`, tùy chọn cấu hình `queue.failed.database` là không cần thiết:

```php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],
```

<a name="disabling-failed-job-storage"></a>
### Vô Hiệu Hóa Lưu Trữ Job Thất Bại

Bạn có thể hướng dẫn Laravel loại bỏ các job thất bại mà không lưu trữ chúng bằng cách đặt giá trị tùy chọn cấu hình `queue.failed.driver` thành `null`. Thông thường, điều này có thể được thực hiện thông qua biến môi trường `QUEUE_FAILED_DRIVER`:

```ini
QUEUE_FAILED_DRIVER=null
```

<a name="failed-job-events"></a>
### Sự Kiện Job Thất Bại

Nếu bạn muốn đăng ký một trình lắng nghe sự kiện sẽ được gọi khi một job thất bại, bạn có thể sử dụng phương thức `failing` của facade `Queue`. Ví dụ, chúng ta có thể đính kèm một closure vào sự kiện này từ phương thức `boot` của `AppServiceProvider` được bao gồm với Laravel:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobFailed;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Queue::failing(function (JobFailed $event) {
            // $event->connectionName
            // $event->job
            // $event->exception
        });
    }
}
```

<a name="clearing-jobs-from-queues"></a>
## Xóa Các Job Khỏi Queues

> [!NOTE]
> Khi sử dụng [Horizon](/docs/{{version}}/horizon), bạn nên sử dụng lệnh `horizon:clear` để xóa các job khỏi queue thay vì lệnh `queue:clear`.

Nếu bạn muốn xóa tất cả các job khỏi queue mặc định của kết nối mặc định, bạn có thể làm như vậy bằng cách sử dụng lệnh Artisan `queue:clear`:

```shell
php artisan queue:clear
```

Bạn cũng có thể cung cấp đối số `connection` và tùy chọn `queue` để xóa các job từ một kết nối và queue cụ thể:

```shell
php artisan queue:clear redis --queue=emails
```

> [!WARNING]
> Việc xóa các job khỏi queues chỉ có sẵn cho các driver queue SQS, Redis và cơ sở dữ liệu. Ngoài ra, quá trình xóa tin nhắn SQS mất đến 60 giây, vì vậy các job được gửi đến queue SQS lên đến 60 giây sau khi bạn xóa queue cũng có thể bị xóa.

<a name="monitoring-your-queues"></a>
## Giám Sát Các Queue Của Bạn

Nếu queue của bạn nhận được một lượng lớn job đột ngột, nó có thể bị quá tải, dẫn đến thời gian chờ dài để các job hoàn tất. Nếu bạn muốn, Laravel có thể cảnh báo bạn khi số lượng job queue của bạn vượt quá ngưỡng được chỉ định.

Để bắt đầu, bạn nên lên lịch lệnh `queue:monitor` để [chạy mỗi phút](/docs/{{version}}/scheduling). Lệnh chấp nhận tên của các queue bạn muốn giám sát cũng như ngưỡng số lượng job mong muốn của bạn:

```shell
php artisan queue:monitor redis:default,redis:deployments --max=100
```

Việc lên lịch lệnh này một mình là không đủ để kích hoạt một thông báo cảnh báo bạn về trạng thái quá tải của queue. Khi lệnh gặp một queue có số lượng job vượt quá ngưỡng của bạn, một sự kiện `Illuminate\Queue\Events\QueueBusy` sẽ được dispatch. Bạn có thể lắng nghe sự kiện này trong `AppServiceProvider` của ứng dụng của bạn để gửi thông báo cho bạn hoặc nhóm phát triển của bạn:

```php
use App\Notifications\QueueHasLongWaitTime;
use Illuminate\Queue\Events\QueueBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (QueueBusy $event) {
        Notification::route('mail', 'dev@example.com')
            ->notify(new QueueHasLongWaitTime(
                $event->connectionName,
                $event->queue,
                $event->size
            ));
    });
}
```

<a name="testing"></a>
## Kiểm Thử

Khi kiểm tra mã dispatch các job, bạn có thể muốn hướng dẫn Laravel không thực sự thực thi job đó, vì mã của job có thể được kiểm tra trực tiếp và riêng biệt với mã dispatch nó. Tất nhiên, để kiểm tra chính job đó, bạn có thể khởi tạo một thể hiện job và gọi phương thức `handle` trực tiếp trong kiểm tra của bạn.

Bạn có thể sử dụng phương thức `fake` của facade `Queue` để ngăn chặn các job trong queue thực sự được đẩy vào queue. Sau khi gọi phương thức `fake` của facade `Queue`, bạn có thể sau đó xác nhận rằng ứng dụng đã cố gắng đẩy các job vào queue:

```php tab=Pest
<?php

use App\Jobs\AnotherJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;

test('orders can be shipped', function () {
    Queue::fake();

    // Perform order shipping...

    // Assert that no jobs were pushed...
    Queue::assertNothingPushed();

    // Assert a job was pushed to a given queue...
    Queue::assertPushedOn('queue-name', ShipOrder::class);

    // Assert a job was pushed
    Queue::assertPushed(ShipOrder::class);

    // Assert a job was pushed exactly once...
    Queue::assertPushedOnce(ShipOrder::class);

    // Assert a job was pushed twice...
    Queue::assertPushedTimes(ShipOrder::class, 2);

    // Assert a job was not pushed...
    Queue::assertNotPushed(AnotherJob::class);

    // Assert that a closure was pushed to the queue...
    Queue::assertClosurePushed();

    // Assert that a closure was not pushed...
    Queue::assertClosureNotPushed();

    // Assert the total number of jobs that were pushed...
    Queue::assertCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Jobs\AnotherJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Queue::fake();

        // Perform order shipping...

        // Assert that no jobs were pushed...
        Queue::assertNothingPushed();

        // Assert a job was pushed to a given queue...
        Queue::assertPushedOn('queue-name', ShipOrder::class);

        // Assert a job was pushed
        Queue::assertPushed(ShipOrder::class);

        // Assert a job was pushed exactly once...
        Queue::assertPushedOnce(ShipOrder::class);

        // Assert a job was pushed twice...
        Queue::assertPushedTimes(ShipOrder::class, 2);

        // Assert a job was not pushed...
        Queue::assertNotPushed(AnotherJob::class);

        // Assert that a closure was pushed to the queue...
        Queue::assertClosurePushed();

        // Assert that a closure was not pushed...
        Queue::assertClosureNotPushed();

        // Assert the total number of jobs that were pushed...
        Queue::assertCount(3);
    }
}
```

Bạn có thể chuyển một closure cho các phương thức `assertPushed`, `assertNotPushed`, `assertClosurePushed`, hoặc `assertClosureNotPushed` để xác nhận rằng một job đã được đẩy vượt qua một "kiểm tra sự thật" nhất định. Nếu ít nhất một job đã được đẩy vượt qua kiểm tra sự thật đã cho thì xác nhận sẽ thành công:

```php
use Illuminate\Queue\CallQueuedClosure;

Queue::assertPushed(function (ShipOrder $job) use ($order) {
    return $job->order->id === $order->id;
});

Queue::assertClosurePushed(function (CallQueuedClosure $job) {
    return $job->name === 'validate-order';
});
```

<a name="faking-a-subset-of-jobs"></a>
### Faking a Subset of Jobs

Nếu bạn chỉ cần fake các job cụ thể trong khi cho phép các job khác của bạn thực thi bình thường, bạn có thể truyền tên class của các job cần fake vào phương thức `fake`:

```php tab=Pest
test('orders can be shipped', function () {
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushedTimes(ShipOrder::class, 2);
});
```

```php tab=PHPUnit
public function test_orders_can_be_shipped(): void
{
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushedTimes(ShipOrder::class, 2);
}
```

Bạn có thể fake tất cả các job ngoại trừ một tập hợp các job được chỉ định bằng phương thức `except`:

```php
Queue::fake()->except([
    ShipOrder::class,
]);
```

<a name="testing-job-chains"></a>
### Testing Job Chains

Để kiểm tra chuỗi job, bạn sẽ cần sử dụng khả năng fake của facade `Bus`. Phương thức `assertChained` của facade `Bus` có thể được sử dụng để xác nhận rằng một [chuỗi job](/docs/{{version}}/queues#job-chaining) đã được dispatch. Phương thức `assertChained` chấp nhận một mảng các job được nối chuỗi làm đối số đầu tiên:

```php
use App\Jobs\RecordShipment;
use App\Jobs\ShipOrder;
use App\Jobs\UpdateInventory;
use Illuminate\Support\Facades\Bus;

Bus::fake();

// ...

Bus::assertChained([
    ShipOrder::class,
    RecordShipment::class,
    UpdateInventory::class
]);
```

Như bạn có thể thấy trong ví dụ trên, mảng các job được nối chuỗi có thể là một mảng tên class của job. Tuy nhiên, bạn cũng có thể cung cấp một mảng các instance job thực tế. Khi làm như vậy, Laravel sẽ đảm bảo rằng các instance job thuộc cùng class và có cùng giá trị thuộc tính với các job được nối chuỗi được dispatch bởi ứng dụng của bạn:

```php
Bus::assertChained([
    new ShipOrder,
    new RecordShipment,
    new UpdateInventory,
]);
```

Bạn có thể sử dụng phương thức `assertDispatchedWithoutChain` để xác nhận rằng một job đã được đẩy mà không có chuỗi job:

```php
Bus::assertDispatchedWithoutChain(ShipOrder::class);
```

<a name="testing-chain-modifications"></a>
#### Testing Chain Modifications

Nếu một job được nối chuỗi [thêm job vào đầu hoặc cuối một chuỗi hiện có](#adding-jobs-to-the-chain), bạn có thể sử dụng phương thức `assertHasChain` của job để xác nhận rằng job có chuỗi job còn lại như mong đợi:

```php
$job = new ProcessPodcast;

$job->handle();

$job->assertHasChain([
    new TranscribePodcast,
    new OptimizePodcast,
    new ReleasePodcast,
]);
```

Phương thức `assertDoesntHaveChain` có thể được sử dụng để xác nhận rằng chuỗi còn lại của job là rỗng:

```php
$job->assertDoesntHaveChain();
```

<a name="testing-chained-batches"></a>
#### Testing Chained Batches

Nếu chuỗi job của bạn [chứa một batch job](#chains-and-batches), bạn có thể xác nhận rằng batch được nối chuỗi khớp với kỳ vọng của bạn bằng cách chèn định nghĩa `Bus::chainedBatch` trong xác nhận chuỗi của bạn:

```php
use App\Jobs\ShipOrder;
use App\Jobs\UpdateInventory;
use Illuminate\Bus\PendingBatch;
use Illuminate\Support\Facades\Bus;

Bus::assertChained([
    new ShipOrder,
    Bus::chainedBatch(function (PendingBatch $batch) {
        return $batch->jobs->count() === 3;
    }),
    new UpdateInventory,
]);
```

<a name="testing-job-batches"></a>
### Testing Job Batches

Phương thức `assertBatched` của facade `Bus` có thể được sử dụng để xác nhận rằng một [batch job](/docs/{{version}}/queues#job-batching) đã được dispatch. Closure được truyền cho phương thức `assertBatched` nhận một instance của `Illuminate\Bus\PendingBatch`, có thể được sử dụng để kiểm tra các job trong batch:

```php
use Illuminate\Bus\PendingBatch;
use Illuminate\Support\Facades\Bus;

Bus::fake();

// ...

Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->name == 'Import CSV' &&
           $batch->jobs->count() === 10;
});
```

Phương thức `hasJobs` có thể được sử dụng trên batch đang chờ xử lý để xác minh rằng batch chứa các job mong đợi. Phương thức này chấp nhận một mảng các instance job, tên class, hoặc closures:

```php
Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->hasJobs([
        new ProcessCsvRow(row: 1),
        new ProcessCsvRow(row: 2),
        new ProcessCsvRow(row: 3),
    ]);
});
```

Khi sử dụng closures, closure sẽ nhận instance job. Loại job mong đợi sẽ được suy ra từ type hint của closure:

```php
Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->hasJobs([
        fn (ProcessCsvRow $job) => $job->row === 1,
        fn (ProcessCsvRow $job) => $job->row === 2,
        fn (ProcessCsvRow $job) => $job->row === 3,
    ]);
});
```

Bạn có thể sử dụng phương thức `assertBatchCount` để xác nhận rằng một số lượng batch nhất định đã được dispatch:

```php
Bus::assertBatchCount(3);
```

Bạn có thể sử dụng `assertNothingBatched` để xác nhận rằng không có batch nào được dispatch:

```php
Bus::assertNothingBatched();
```

<a name="testing-job-batch-interaction"></a>
#### Testing Job / Batch Interaction

Ngoài ra, đôi khi bạn có thể cần kiểm tra tương tác của một job riêng lẻ với batch cơ bản của nó. Ví dụ, bạn có thể cần kiểm tra xem một job có hủy xử lý tiếp theo cho batch của nó hay không. Để thực hiện việc này, bạn cần gán một batch giả cho job thông qua phương thức `withFakeBatch`. Phương thức `withFakeBatch` trả về một tuple chứa instance job và batch giả:

```php
[$job, $batch] = (new ShipOrder)->withFakeBatch();

$job->handle();

$this->assertTrue($batch->cancelled());
$this->assertEmpty($batch->added);
```

<a name="testing-job-queue-interactions"></a>
### Testing Job / Queue Interactions

Đôi khi, bạn có thể cần kiểm tra rằng một job trong hàng đợi [giải phóng chính nó trở lại hàng đợi](#manually-releasing-a-job). Hoặc, bạn có thể cần kiểm tra rằng job đã tự xóa chính nó. Bạn có thể kiểm tra các tương tác hàng đợi này bằng cách khởi tạo job và gọi phương thức `withFakeQueueInteractions`.

Sau khi các tương tác hàng đợi của job đã được fake, bạn có thể gọi phương thức `handle` trên job. Sau khi gọi job, các phương thức xác nhận khác nhau có sẵn để xác minh các tương tác hàng đợi của job:

```php
use App\Exceptions\CorruptedAudioException;
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
$job->assertDeleted();
$job->assertNotDeleted();
$job->assertFailed();
$job->assertFailedWith(CorruptedAudioException::class);
$job->assertNotFailed();
```

<a name="job-events"></a>
## Job Events

Sử dụng các phương thức `before` và `after` trên facade `Queue` [facade](/docs/{{version}}/facades), bạn có thể chỉ định các callback sẽ được thực thi trước hoặc sau khi một job trong hàng đợi được xử lý. Các callback này là cơ hội tuyệt vời để thực hiện ghi nhật ký bổ sung hoặc tăng thống kê cho bảng điều khiển. Thông thường, bạn nên gọi các phương thức này từ phương thức `boot` của một [service provider](/docs/{{version}}/providers). Ví dụ, chúng ta có thể sử dụng `AppServiceProvider` được đi kèm với Laravel:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobProcessing;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Queue::before(function (JobProcessing $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });

        Queue::after(function (JobProcessed $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });
    }
}
```

Sử dụng phương thức `looping` trên facade `Queue` [facade](/docs/{{version}}/facades), bạn có thể chỉ định các callback thực thi trước khi worker cố gắng lấy một job từ hàng đợi. Ví dụ, bạn có thể đăng ký một closure để rollback bất kỳ giao dịch nào bị mở bởi một job thất bại trước đó:

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Queue;

Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```

Laravel cũng dispatch một sự kiện `Illuminate\Queue\Events\WorkerIdle` khi một queue worker không thể lấy một job từ hàng đợi:

```php
use Illuminate\Queue\Events\WorkerIdle;
use Illuminate\Support\Facades\Event;

Event::listen(function (WorkerIdle $event) {
    // $event->connectionName
    // $event->queue
    // $event->workerOptions
});
```

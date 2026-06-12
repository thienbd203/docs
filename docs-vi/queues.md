# Queues

- [Introduction](#introduction)
    - [Connections vs. Queues](#connections-vs-queues)
    - [Driver Notes and Prerequisites](#driver-prerequisites)
- [Creating Jobs](#creating-jobs)
    - [Generating Job Classes](#generating-job-classes)
    - [Class Structure](#class-structure)
    - [Unique Jobs](#unique-jobs)
    - [Debounced Jobs](#debounced-jobs)
    - [Encrypted Jobs](#encrypted-jobs)
- [Job Middleware](#job-middleware)
    - [Rate Limiting](#rate-limiting)
    - [Preventing Job Overlaps](#preventing-job-overlaps)
    - [Throttling Exceptions](#throttling-exceptions)
    - [Skipping Jobs](#skipping-jobs)
- [Dispatching Jobs](#dispatching-jobs)
    - [Delayed Dispatching](#delayed-dispatching)
    - [Synchronous Dispatching](#synchronous-dispatching)
    - [Bulk Dispatching](#bulk-dispatching)
    - [Preparing Jobs Before Dispatch](#preparing-jobs-before-dispatch)
    - [Jobs & Database Transactions](#jobs-and-database-transactions)
    - [Job Chaining](#job-chaining)
    - [Customizing The Queue and Connection](#customizing-the-queue-and-connection)
    - [Specifying Max Job Attempts / Timeout Values](#max-job-attempts-and-timeout)
    - [SQS FIFO and Fair Queues](#sqs-fifo-and-fair-queues)
    - [Queue Failover](#queue-failover)
    - [Error Handling](#error-handling)
- [Job Batching](#job-batching)
    - [Defining Batchable Jobs](#defining-batchable-jobs)
    - [Dispatching Batches](#dispatching-batches)
    - [Chains and Batches](#chains-and-batches)
    - [Adding Jobs to Batches](#adding-jobs-to-batches)
    - [Inspecting Batches](#inspecting-batches)
    - [Cancelling Batches](#cancelling-batches)
    - [Batch Failures](#batch-failures)
    - [Pruning Batches](#pruning-batches)
    - [Storing Batches in DynamoDB](#storing-batches-in-dynamodb)
- [Queueing Closures](#queueing-closures)
- [Running the Queue Worker](#running-the-queue-worker)
    - [The `queue:work` Command](#the-queue-work-command)
    - [Queue Priorities](#queue-priorities)
    - [Queue Workers and Deployment](#queue-workers-and-deployment)
    - [Reacting to Worker Signals](#reacting-to-worker-signals)
    - [Job Expirations and Timeouts](#job-expirations-and-timeouts)
    - [Pausing and Resuming Queue Workers](#pausing-and-resuming-queue-workers)
- [Supervisor Configuration](#supervisor-configuration)
- [Dealing With Failed Jobs](#dealing-with-failed-jobs)
    - [Cleaning Up After Failed Jobs](#cleaning-up-after-failed-jobs)
    - [Retrying Failed Jobs](#retrying-failed-jobs)
    - [Ignoring Missing Models](#ignoring-missing-models)
    - [Pruning Failed Jobs](#pruning-failed-jobs)
    - [Storing Failed Jobs in DynamoDB](#storing-failed-jobs-in-dynamodb)
    - [Disabling Failed Job Storage](#disabling-failed-job-storage)
    - [Failed Job Events](#failed-job-events)
- [Clearing Jobs From Queues](#clearing-jobs-from-queues)
- [Monitoring Your Queues](#monitoring-your-queues)
- [Testing](#testing)
    - [Faking a Subset of Jobs](#faking-a-subset-of-jobs)
    - [Testing Job Chains](#testing-job-chains)
    - [Testing Job Batches](#testing-job-batches)
    - [Testing Job / Queue Interactions](#testing-job-queue-interactions)
- [Job Events](#job-events)

<a name="introduction"></a>
## Introduction

Trong khi xây dựng ứng dụng web của bạn, bạn có thể có một số nhiệm vụ, chẳng hạn như phân tích cú pháp và lưu trữ một file CSV đã tải lên, mất quá nhiều thời gian để thực hiện trong một web request điển hình. May mắn thay, Laravel cho phép bạn dễ dàng tạo queued jobs có thể được xử lý trong nền. Bằng cách chuyển các nhiệm vụ tốn thời gian sang queue, ứng dụng của bạn có thể phản hồi với các web requests với tốc độ nhanh chóng và cung cấp trải nghiệm người dùng tốt hơn cho khách hàng của bạn.

Laravel queues cung cấp một queueing API thống nhất trên nhiều queue backends khác nhau, chẳng hạn như [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), hoặc thậm chí là một relational database.

Các tùy chọn cấu hình queue của Laravel được lưu trữ trong file cấu hình `config/queue.php` của ứng dụng. Trong file này, bạn sẽ tìm thấy các cấu hình connection cho từng queue driver được bao gồm với framework, bao gồm database, [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), và [Beanstalkd](https://beanstalkd.github.io/) drivers, cũng như một synchronous driver sẽ thực thi jobs ngay lập tức (để sử dụng trong quá trình phát triển hoặc testing). Một `null` queue driver cũng được bao gồm sẽ loại bỏ queued jobs.

> [!NOTE]
> Laravel Horizon là một dashboard và hệ thống cấu hình đẹp mắt cho Redis powered queues của bạn. Xem tài liệu [Horizon documentation](/docs/{{version}}/horizon) đầy đủ để biết thêm thông tin.

<a name="connections-vs-queues"></a>
### Connections vs. Queues

Trước khi bắt đầu với Laravel queues, điều quan trọng là phải hiểu sự khác biệt giữa "connections" và "queues". Trong file cấu hình `config/queue.php` của bạn, có một mảng cấu hình `connections`. Tùy chọn này định nghĩa các connections đến các queue backend services như Amazon SQS, Beanstalk, hoặc Redis. Tuy nhiên, bất kỳ queue connection nhất định nào có thể có nhiều "queues" có thể được coi là các stacks hoặc piles khác nhau của queued jobs.

Lưu ý rằng mỗi ví dụ cấu hình connection trong file cấu hình `queue` chứa một attribute `queue`. Đây là default queue mà jobs sẽ được dispatch đến khi chúng được gửi đến một connection nhất định. Nói cách khác, nếu bạn dispatch một job mà không định nghĩa rõ ràng queue nào nó nên được dispatch đến, job sẽ được đặt vào queue được định nghĩa trong attribute `queue` của cấu hình connection:

```php
use App\Jobs\ProcessPodcast;

// This job is sent to the default connection's default queue...
ProcessPodcast::dispatch();

// This job is sent to the default connection's "emails" queue...
ProcessPodcast::dispatch()->onQueue('emails');
```

Một số ứng dụng có thể không cần bao giờ đẩy jobs lên nhiều queues, thay vào đó thích có một queue đơn giản. Tuy nhiên, đẩy jobs lên nhiều queues có thể đặc biệt hữu ích cho các ứng dụng muốn ưu tiên hoặc phân đoạn cách jobs được xử lý, vì Laravel queue worker cho phép bạn chỉ định các queues mà nó nên xử lý theo ưu tiên. Ví dụ, nếu bạn đẩy jobs lên một queue `high`, bạn có thể chạy một worker cung cấp cho chúng ưu tiên xử lý cao hơn:

```shell
php artisan queue:work --queue=high,default
```

<a name="driver-prerequisites"></a>
### Driver Notes and Prerequisites

<a name="database"></a>
#### Database

Để sử dụng `database` queue driver, bạn sẽ cần một database table để giữ jobs. Thông thường, điều này được bao gồm trong [database migration](/docs/{{version}}/migrations) mặc định của Laravel `0001_01_01_000002_create_jobs_table.php`; tuy nhiên, nếu ứng dụng của bạn không chứa migration này, bạn có thể sử dụng command Artisan `make:queue-table` để tạo nó:

```shell
php artisan make:queue-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Để sử dụng `redis` queue driver, bạn nên cấu hình một Redis database connection trong file cấu hình `config/database.php` của bạn.

> [!WARNING]
> Các tùy chọn Redis `serializer` và `compression` không được hỗ trợ bởi `redis` queue driver.

<a name="redis-cluster"></a>
##### Redis Cluster

Nếu Redis queue connection của bạn sử dụng [Redis Cluster](https://redis.io/docs/latest/operate/rs/databases/durability-ha/clustering), tên queue của bạn phải chứa một [key hash tag](https://redis.io/docs/latest/develop/using-commands/keyspace/#hashtags). Điều này được yêu cầu để đảm bảo tất cả các Redis keys cho một queue nhất định được đặt vào cùng một hash slot:

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

Khi sử dụng Redis queue, bạn có thể sử dụng tùy chọn cấu hình `block_for` để chỉ định bao lâu driver nên đợi một job trở nên có sẵn trước khi lặp lại qua worker loop và re-polling Redis database.

Điều chỉnh giá trị này dựa trên queue load của bạn có thể hiệu quả hơn so với việc liên tục polling Redis database cho các jobs mới. Ví dụ, bạn có thể đặt giá trị thành `5` để chỉ định rằng driver nên block trong năm giây trong khi đợi một job trở nên có sẵn:

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
> Đặt `block_for` thành `0` sẽ gây ra queue workers block vô thời hạn cho đến khi một job có sẵn. Điều này cũng sẽ ngăn chặn các signals như `SIGTERM` được xử lý cho đến khi job tiếp theo đã được xử lý.

<a name="sqs-overflow-storage"></a>
#### SQS Overflow Storage

Amazon SQS giới hạn kích thước tối đa của một queued message payload. Nếu bạn cần dispatch jobs với payloads có thể vượt quá giới hạn này, bạn có thể cấu hình Laravel để lưu trữ oversized SQS payloads trong một cache store và gửi một pointer qua SQS thay thế. Để bật tính năng này, thêm một mảng `overflow` vào cấu hình SQS queue connection của bạn:

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

Khi overflow storage được bật, Laravel sẽ lưu trữ payloads có kích thước ít nhất 1 MB trong cache store được cấu hình. Nếu tùy chọn `always` là `true`, mọi SQS payload sẽ được lưu trữ trong cache store bất kể kích thước của nó. Vì queued jobs sẽ cần truy xuất payloads của chúng từ cache store khi chúng được xử lý, bạn nên chọn một store có thể giữ payloads cho đến khi workers của bạn xử lý chúng. Theo mặc định, các payloads được lưu trữ sẽ bị xóa sau khi jobs của chúng đã được xử lý thành công và xóa khỏi SQS.

Nếu tùy chọn `flush_on_clear` là `true`, cache store overflow được cấu hình sẽ được flushed khi command `queue:clear` xóa SQS queue. Vì việc flush một cache store có thể xóa tất cả các items từ store đó, bạn nên cấu hình SQS overflow storage để sử dụng một cache store chuyên dụng khi bật tùy chọn này.

<a name="other-driver-prerequisites"></a>
#### Other Driver Prerequisites

Các dependencies sau đây cần thiết cho các queue drivers được liệt kê. Các dependencies này có thể được cài đặt thông qua trình quản lý gói Composer:

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~5.0`
- Redis: `predis/predis ~2.0` hoặc phpredis PHP extension
- [MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/): `mongodb/laravel-mongodb`

</div>

File queues.md rất dài (3487 dòng). Do giới hạn độ dài của một lần viết, tôi sẽ tách file thành nhiều phần. Đây là phần đầu tiên của file dịch. Bạn có muốn tôi tiếp tục dịch phần còn lại không?
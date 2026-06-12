# Redis

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Clusters](#clusters)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Tương tác với Redis](#interacting-with-redis)
    - [Giao dịch](#transactions)
    - [Pipelining Commands](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## Giới thiệu

[Redis](https://redis.io) là một kho lưu trữ key-value mã nguồn mở, nâng cao. Nó thường được gọi là máy chủ cấu trúc dữ liệu vì các key có thể chứa [strings](https://redis.io/docs/latest/develop/data-types/strings/), [hashes](https://redis.io/docs/latest/develop/data-types/hashes/), [lists](https://redis.io/docs/latest/develop/data-types/lists/), [sets](https://redis.io/docs/latest/develop/data-types/sets/), và [sorted sets](https://redis.io/docs/latest/develop/data-types/sorted-sets/).

Trước khi sử dụng Redis với Laravel, chúng tôi khuyến khích bạn cài đặt và sử dụng extension PHP [PhpRedis](https://github.com/phpredis/phpredis) thông qua PECL. Extension này phức tạp hơn để cài đặt so với các gói PHP "user-land" nhưng có thể mang lại hiệu suất tốt hơn cho các ứng dụng sử dụng Redis nhiều. Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail), extension này đã được cài đặt sẵn trong Docker container của ứng dụng của bạn.

Nếu bạn không thể cài đặt extension PhpRedis, bạn có thể cài đặt gói `predis/predis` thông qua Composer. Predis là một Redis client được viết hoàn toàn bằng PHP và không yêu cầu bất kỳ extension bổ sung nào:

```shell
composer require predis/predis
```

<a name="configuration"></a>
## Cấu hình

Bạn có thể cấu hình các cài đặt Redis của ứng dụng thông qua file cấu hình `config/database.php`. Trong file này, bạn sẽ thấy một mảng `redis` chứa các máy chủ Redis được ứng dụng của bạn sử dụng:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],

],
```

Mỗi máy chủ Redis được định nghĩa trong file cấu hình của bạn được yêu cầu phải có tên, host và port trừ khi bạn định nghĩa một URL duy nhất để đại diện cho kết nối Redis:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => 'tcp://127.0.0.1:6379?database=0',
    ],

    'cache' => [
        'url' => 'tls://user:password@127.0.0.1:6380?database=1',
    ],

],
```

<a name="configuring-the-connection-scheme"></a>
#### Cấu hình Scheme Kết nối

Theo mặc định, các Redis client sẽ sử dụng scheme `tcp` khi kết nối với các máy chủ Redis của bạn; tuy nhiên, bạn có thể sử dụng mã hóa TLS / SSL bằng cách chỉ định một tùy chọn cấu hình `scheme` trong mảng cấu hình của máy chủ Redis:

```php
'default' => [
    'scheme' => 'tls',
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
],
```

<a name="clusters"></a>
### Clusters

Nếu ứng dụng của bạn đang sử dụng một cluster của các máy chủ Redis, bạn nên định nghĩa các cluster này trong một key `clusters` của cấu hình Redis của bạn. Key cấu hình này không tồn tại theo mặc định nên bạn sẽ cần tạo nó trong file cấu hình `config/database.php` của ứng dụng:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'clusters' => [
        'default' => [
            [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'username' => env('REDIS_USERNAME'),
                'password' => env('REDIS_PASSWORD'),
                'port' => env('REDIS_PORT', '6379'),
                'database' => env('REDIS_DB', '0'),
            ],
        ],
    ],

    // ...
],
```

Theo mặc định, Laravel sẽ sử dụng Redis clustering gốc vì giá trị cấu hình `options.cluster` được đặt thành `redis`. Redis clustering là một tùy chọn mặc định tuyệt vời, vì nó xử lý failover một cách linh hoạt.

Laravel cũng hỗ trợ sharding phía client khi sử dụng Predis. Tuy nhiên, sharding phía client không xử lý failover; do đó, nó chủ yếu phù hợp cho dữ liệu cache tạm thời có sẵn từ một kho dữ liệu chính khác.

Nếu bạn muốn sử dụng sharding phía client thay vì Redis clustering gốc, bạn có thể xóa giá trị cấu hình `options.cluster` trong file cấu hình `config/database.php` của ứng dụng:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'clusters' => [
        // ...
    ],

    // ...
],
```

<a name="predis"></a>
### Predis

Nếu bạn muốn ứng dụng của mình tương tác với Redis thông qua gói Predis, bạn nên đảm bảo giá trị của biến môi trường `REDIS_CLIENT` là `predis`:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'predis'),

    // ...
],
```

Ngoài các tùy chọn cấu hình mặc định, Predis hỗ trợ các [tham số kết nối](https://github.com/nrk/predis/wiki/Connection-Parameters) bổ sung có thể được định nghĩa cho từng máy chủ Redis của bạn. Để sử dụng các tùy chọn cấu hình bổ sung này, hãy thêm chúng vào cấu hình máy chủ Redis của bạn trong file cấu hình `config/database.php` của ứng dụng:

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_write_timeout' => 60,
],
```

<a name="phpredis"></a>
### PhpRedis

Theo mặc định, Laravel sẽ sử dụng extension PhpRedis để giao tiếp với Redis. Client mà Laravel sẽ sử dụng để giao tiếp với Redis được quy định bởi giá trị của tùy chọn cấu hình `redis.client`, thường phản ánh giá trị của biến môi trường `REDIS_CLIENT`:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    // ...
],
```

Ngoài các tùy chọn cấu hình mặc định, PhpRedis hỗ trợ các tham số kết nối bổ sung sau: `name`, `persistent`, `persistent_id`, `prefix`, `read_timeout`, `retry_interval`, `max_retries`, `backoff_algorithm`, `backoff_base`, `backoff_cap`, `timeout`, và `context`. Bạn có thể thêm bất kỳ tùy chọn nào trong số này vào cấu hình máy chủ Redis của bạn trong file cấu hình `config/database.php`:

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_timeout' => 60,
    'context' => [
        // 'auth' => ['username', 'secret'],
        // 'stream' => ['verify_peer' => false],
    ],
],
```

<a name="retry-and-backoff-configuration"></a>
#### Cấu hình Thử lại và Backoff

Các tùy chọn `retry_interval`, `max_retries`, `backoff_algorithm`, `backoff_base`, và `backoff_cap` có thể được sử dụng để cấu hình cách client PhpRedis nên cố gắng kết nối lại với máy chủ Redis. Các thuật toán backoff sau được hỗ trợ: `default`, `decorrelated_jitter`, `equal_jitter`, `exponential`, `uniform`, và `constant`:

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'max_retries' => env('REDIS_MAX_RETRIES', 3),
    'backoff_algorithm' => env('REDIS_BACKOFF_ALGORITHM', 'decorrelated_jitter'),
    'backoff_base' => env('REDIS_BACKOFF_BASE', 100),
    'backoff_cap' => env('REDIS_BACKOFF_CAP', 1000),
],
```

Predis 3.4.0 và các phiên bản sau hỗ trợ cấu hình thử lại và backoff tích hợp thông qua class `Retry`. Cấu hình nó bằng tùy chọn `retry` với một trong các chiến lược sau: `NoBackoff`, `EqualBackoff`, hoặc `ExponentialBackoff`:

```php
use Predis\Retry;
use Predis\Retry\Strategy\ExponentialBackoff;

'default' => [
    'url' => env('REDIS_URL'),
    // ...
    'retry' => new Retry(
        new ExponentialBackoff(
            env('REDIS_BACKOFF_BASE', 100),
            env('REDIS_BACKOFF_CAP', 1000),
            true, // Enables jitter
        ),
        env('REDIS_MAX_RETRIES', 3)
    )
],
```

<a name="unix-socket-connections"></a>
#### Kết nối Unix Socket

Kết nối Redis cũng có thể được cấu hình để sử dụng Unix socket thay vì TCP. Điều này có thể mang lại hiệu suất được cải thiện bằng cách loại bỏ chi phí TCP cho các kết nối đến các instance Redis trên cùng máy chủ với ứng dụng của bạn. Để cấu hình Redis sử dụng Unix socket, hãy đặt biến môi trường `REDIS_HOST` của bạn thành đường dẫn của socket Redis và biến môi trường `REDIS_PORT` thành `0`:

```env
REDIS_HOST=/run/redis/redis.sock
REDIS_PORT=0
```

<a name="phpredis-serialization"></a>
#### Serialization và Nén PhpRedis

Extension PhpRedis cũng có thể được cấu hình để sử dụng nhiều serializer và thuật toán nén khác nhau. Các thuật toán này có thể được cấu hình thông qua mảng `options` của cấu hình Redis của bạn:

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        'serializer' => Redis::SERIALIZER_MSGPACK,
        'compression' => Redis::COMPRESSION_LZ4,
    ],

    // ...
],
```

Các serializer hiện được hỗ trợ bao gồm: `Redis::SERIALIZER_NONE` (mặc định), `Redis::SERIALIZER_PHP`, `Redis::SERIALIZER_JSON`, `Redis::SERIALIZER_IGBINARY`, và `Redis::SERIALIZER_MSGPACK`.

Các thuật toán nén được hỗ trợ bao gồm: `Redis::COMPRESSION_NONE` (mặc định), `Redis::COMPRESSION_LZF`, `Redis::COMPRESSION_ZSTD`, và `Redis::COMPRESSION_LZ4`.

<a name="interacting-with-redis"></a>
## Tương tác với Redis

Bạn có thể tương tác với Redis bằng cách gọi các phương thức khác nhau trên [facade](/docs/{{version}}/facades) `Redis`. Facade `Redis` hỗ trợ các phương thức động, nghĩa là bạn có thể gọi bất kỳ [lệnh Redis](https://redis.io/commands) nào trên facade và lệnh đó sẽ được chuyển trực tiếp đến Redis. Trong ví dụ này, chúng ta sẽ gọi lệnh Redis `GET` bằng cách gọi phương thức `get` trên facade `Redis`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Redis;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => Redis::get('user:profile:'.$id)
        ]);
    }
}
```

Như đã đề cập ở trên, bạn có thể gọi bất kỳ lệnh nào của Redis trên facade `Redis`. Laravel sử dụng các phương thức magic để chuyển các lệnh đến máy chủ Redis. Nếu một lệnh Redis mong đợi các đối số, bạn nên chuyển chúng đến phương thức tương ứng của facade:

```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

Ngoài ra, bạn có thể chuyển các lệnh đến máy chủ bằng phương thức `command` của facade `Redis`, chấp nhận tên của lệnh làm đối số đầu tiên và một mảng giá trị làm đối số thứ hai:

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

<a name="using-multiple-redis-connections"></a>
#### Sử dụng Nhiều Kết nối Redis

File cấu hình `config/database.php` của ứng dụng cho phép bạn định nghĩa nhiều kết nối / máy chủ Redis. Bạn có thể lấy một kết nối đến một kết nối Redis cụ thể bằng phương thức `connection` của facade `Redis`:

```php
$redis = Redis::connection('connection-name');
```

Để lấy một instance của kết nối Redis mặc định, bạn có thể gọi phương thức `connection` mà không có bất kỳ đối số bổ sung nào:

```php
$redis = Redis::connection();
```

<a name="transactions"></a>
### Giao dịch

Phương thức `transaction` của facade `Redis` cung cấp một wrapper tiện lợi xung quanh các lệnh `MULTI` và `EXEC` gốc của Redis. Phương thức `transaction` chấp nhận một closure làm đối số duy nhất. Closure này sẽ nhận một instance kết nối Redis và có thể phát hành bất kỳ lệnh nào nó muốn đến instance này. Tất cả các lệnh Redis được phát hành trong closure sẽ được thực thi trong một giao dịch nguyên tử duy nhất:

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```

> [!WARNING]
> Khi định nghĩa một giao dịch Redis, bạn không thể lấy bất kỳ giá trị nào từ kết nối Redis. Hãy nhớ rằng, giao dịch của bạn được thực thi như một thao tác nguyên tử duy nhất và thao tác đó không được thực thi cho đến khi toàn bộ closure của bạn đã hoàn thành việc thực thi các lệnh của nó.

#### Lua Scripts

Phương thức `eval` cung cấp một phương thức khác để thực thi nhiều lệnh Redis trong một thao tác nguyên tử duy nhất. Tuy nhiên, phương thức `eval` có lợi ích là có thể tương tác và kiểm tra các giá trị key Redis trong quá trình thao tác đó. Các script Redis được viết bằng [ngôn ngữ lập trình Lua](https://www.lua.org).

Phương thức `eval` có thể hơi đáng sợ lúc đầu, nhưng chúng ta sẽ khám phá một ví dụ cơ bản để làm quen. Phương thức `eval` mong đợi một số đối số. Đầu tiên, bạn nên chuyển script Lua (dưới dạng chuỗi) đến phương thức. Thứ hai, bạn nên chuyển số lượng key (dưới dạng số nguyên) mà script tương tác với. Thứ ba, bạn nên chuyển tên của các key đó. Cuối cùng, bạn có thể chuyển bất kỳ đối số bổ sung nào khác mà bạn cần truy cập trong script của mình.

Trong ví dụ này, chúng ta sẽ tăng một bộ đếm, kiểm tra giá trị mới của nó, và tăng một bộ đếm thứ hai nếu giá trị của bộ đếm đầu tiên lớn hơn năm. Cuối cùng, chúng ta sẽ trả về giá trị của bộ đếm đầu tiên:

```php
$value = Redis::eval(<<<'LUA'
    local counter = redis.call("incr", KEYS[1])

    if counter > 5 then
        redis.call("incr", KEYS[2])
    end

    return counter
LUA, 2, 'first-counter', 'second-counter');
```

> [!WARNING]
> Vui lòng tham khảo [tài liệu Redis](https://redis.io/commands/eval) để biết thêm thông tin về script Redis.

<a name="pipelining-commands"></a>
### Pipelining Commands

Đôi khi bạn có thể cần thực hiện hàng chục lệnh Redis. Thay vì thực hiện một chuyến mạng đến máy chủ Redis của bạn cho mỗi lệnh, bạn có thể sử dụng phương thức `pipeline`. Phương thức `pipeline` chấp nhận một đối số: một closure nhận một instance Redis. Bạn có thể phát hành tất cả các lệnh của mình đến instance Redis này và chúng sẽ tất cả được gửi đến máy chủ Redis cùng một lúc để giảm các chuyến mạng đến máy chủ. Các lệnh vẫn sẽ được thực thi theo thứ tự chúng được phát hành:

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::pipeline(function (Redis $pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

<a name="pubsub"></a>
## Pub / Sub

Laravel cung cấp một giao diện tiện lợi đến các lệnh `publish` và `subscribe` của Redis. Các lệnh Redis này cho phép bạn lắng nghe tin nhắn trên một "channel" nhất định. Bạn có thể xuất bản tin nhắn đến channel từ một ứng dụng khác, hoặc thậm chí sử dụng một ngôn ngữ lập trình khác, cho phép giao tiếp dễ dàng giữa các ứng dụng và quy trình.

Đầu tiên, hãy thiết lập một channel listener bằng phương thức `subscribe`. Chúng ta sẽ đặt gọi phương thức này trong một [lệnh Artisan](/docs/{{version}}/artisan) vì việc gọi phương thức `subscribe` bắt đầu một quy trình chạy dài:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Subscribe to a Redis channel';

    /**
     * Execute the console command.
     */
    public function handle(): void
    {
        Redis::subscribe(['test-channel'], function (string $message) {
            echo $message;
        });
    }
}
```

Bây giờ chúng ta có thể xuất bản tin nhắn đến channel bằng phương thức `publish`:

```php
use Illuminate\Support\Facades\Redis;

Route::get('/publish', function () {
    // ...

    Redis::publish('test-channel', json_encode([
        'name' => 'Adam Wathan'
    ]));
});
```

<a name="wildcard-subscriptions"></a>
#### Wildcard Subscriptions

Sử dụng phương thức `psubscribe`, bạn có thể đăng ký đến một channel wildcard, có thể hữu ích để bắt tất cả tin nhắn trên tất cả các channel. Tên channel sẽ được chuyển làm đối số thứ hai đến closure được cung cấp:

```php
Redis::psubscribe(['*'], function (string $message, string $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function (string $message, string $channel) {
    echo $message;
});
```

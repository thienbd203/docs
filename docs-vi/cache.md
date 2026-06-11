# Cache

- [Introduction](#introduction)
- [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Cache Usage](#cache-usage)
    - [Obtaining a Cache Instance](#obtaining-a-cache-instance)
    - [Retrieving Items From the Cache](#retrieving-items-from-the-cache)
    - [Storing Items in the Cache](#storing-items-in-the-cache)
    - [Extending Item Lifetime](#extending-item-lifetime)
    - [Removing Items From the Cache](#removing-items-from-the-cache)
    - [Cache Memoization](#cache-memoization)
    - [The Cache Helper](#the-cache-helper)
- [Cache Tags](#cache-tags)
- [Atomic Locks](#atomic-locks)
    - [Managing Locks](#managing-locks)
    - [Managing Locks Across Processes](#managing-locks-across-processes)
    - [Concurrency Limiting](#concurrency-limiting)
- [Cache Failover](#cache-failover)
- [Adding Custom Cache Drivers](#adding-custom-cache-drivers)
    - [Writing the Driver](#writing-the-driver)
    - [Registering the Driver](#registering-the-driver)
- [Events](#events)

<a name="introduction"></a>
## Introduction

Một số nhiệm vụ truy xuất hoặc xử lý dữ liệu được thực hiện bởi ứng dụng của bạn có thể tốn nhiều CPU hoặc mất vài giây để hoàn thành. Khi điều này xảy ra, việc cache dữ liệu đã truy xuất trong một khoảng thời gian là phổ biến để nó có thể được truy xuất nhanh chóng trên các requests tiếp theo cho cùng một dữ liệu. Dữ liệu được cache thường được lưu trữ trong một data store rất nhanh như [Memcached](https://memcached.org) hoặc [Redis](https://redis.io).

May mắn thay, Laravel cung cấp một API thống nhất, expressive cho nhiều cache backends khác nhau, cho phép bạn tận dụng khả năng truy xuất dữ liệu nhanh chóng của chúng và tăng tốc ứng dụng web của bạn.

<a name="configuration"></a>
## Configuration

File cấu hình cache của ứng dụng của bạn nằm tại `config/cache.php`. Trong file này, bạn có thể chỉ định cache store nào bạn muốn sử dụng theo mặc định trong toàn bộ ứng dụng của mình. Laravel hỗ trợ các caching backends phổ biến như [Memcached](https://memcached.org), [Redis](https://redis.io), [DynamoDB](https://aws.amazon.com/dynamodb), relational databases, và filesystem disks out of the box. Ngoài ra, một cache driver dựa trên file có sẵn, trong khi các cache drivers `array` và `null` cung cấp các cache backends thuận tiện cho các automated tests của bạn.

File cấu hình cache cũng chứa nhiều tùy chọn khác mà bạn có thể xem lại. Theo mặc định, Laravel được cấu hình để sử dụng cache driver `database`, lưu trữ các objects được cache đã serialize trong database của ứng dụng.

<a name="driver-prerequisites"></a>
### Driver Prerequisites

<a name="prerequisites-database"></a>
#### Database

Khi sử dụng cache driver `database`, bạn sẽ cần một database table để chứa dữ liệu cache. Thông thường, điều này được bao gồm trong [database migration](/docs/{{version}}/migrations) mặc định của Laravel `0001_01_01_000001_create_cache_table.php`; tuy nhiên, nếu ứng dụng của bạn không chứa migration này, bạn có thể sử dụng command Artisan `make:cache-table` để tạo nó:

```shell
php artisan make:cache-table

php artisan migrate
```

<a name="memcached"></a>
#### Memcached

Sử dụng driver Memcached yêu cầu [Memcached PECL package](https://pecl.php.net/package/memcached) được cài đặt. Bạn có thể liệt kê tất cả các Memcached servers của mình trong file cấu hình `config/cache.php`. File này đã chứa một entry `memcached.servers` để giúp bạn bắt đầu:

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => env('MEMCACHED_HOST', '127.0.0.1'),
            'port' => env('MEMCACHED_PORT', 11211),
            'weight' => 100,
        ],
    ],
],
```

Nếu cần thiết, bạn có thể đặt tùy chọn `host` thành một đường dẫn UNIX socket. Nếu bạn làm như vậy, tùy chọn `port` nên được đặt thành `0`:

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],
],
```

<a name="redis"></a>
#### Redis

Trước khi sử dụng Redis cache với Laravel, bạn sẽ cần cài đặt PhpRedis PHP extension qua PECL hoặc cài đặt package `predis/predis` (~2.0) qua Composer. [Laravel Sail](/docs/{{version}}/sail) đã bao gồm extension này. Ngoài ra, các nền tảng ứng dụng Laravel chính thức như [Laravel Cloud](https://cloud.laravel.com) và [Laravel Forge](https://forge.laravel.com) có PhpRedis extension được cài đặt theo mặc định.

Để biết thêm thông tin về cấu hình Redis, hãy tham khảo [Laravel documentation page](/docs/{{version}}/redis#configuration) của nó.

<a name="storage"></a>
#### Storage

Cache driver `storage` cho phép bạn lưu trữ các giá trị được cache trên bất kỳ [filesystem disks](/docs/{{version}}/filesystem) nào được cấu hình của ứng dụng. Điều này có thể hữu ích khi bạn muốn sử dụng một disk hiện có, chẳng hạn như disk S3, làm một key / value cache store:

```php
'storage' => [
    'driver' => 'storage',
    'disk' => env('CACHE_STORAGE_DISK'),
    'path' => env('CACHE_STORAGE_PATH', 'framework/cache/data'),
],
```

<a name="dynamodb"></a>
#### DynamoDB

Trước khi sử dụng cache driver [DynamoDB](https://aws.amazon.com/dynamodb), bạn phải tạo một DynamoDB table để lưu trữ tất cả dữ liệu được cache. Thông thường, table này nên được đặt tên là `cache`. Tuy nhiên, bạn nên đặt tên table dựa trên giá trị của tùy chọn cấu hình `stores.dynamodb.table` trong file cấu hình `cache`. Tên table cũng có thể được đặt qua biến môi trường `DYNAMODB_CACHE_TABLE`.

Table này cũng nên có một string partition key với tên tương ứng với giá trị của tùy chọn cấu hình `stores.dynamodb.attributes.key` trong file cấu hình `cache` của ứng dụng. Theo mặc định, partition key nên được đặt tên là `key`.

Thông thường, DynamoDB sẽ không chủ động loại bỏ các items đã hết hạn từ một table. Do đó, bạn nên [bật Time to Live (TTL)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html) trên table. Khi cấu hình các cài đặt TTL của table, bạn nên đặt tên attribute TTL thành `expires_at`.

Tiếp theo, cài đặt AWS SDK để ứng dụng Laravel của bạn có thể giao tiếp với DynamoDB:

```shell
composer require aws/aws-sdk-php
```

Ngoài ra, bạn nên đảm bảo các giá trị được cung cấp cho các tùy chọn cấu hình cache store DynamoDB. Thông thường các tùy chọn này, chẳng hạn như `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY`, nên được định nghĩa trong file cấu hình `.env` của ứng dụng:

```php
'dynamodb' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
    'endpoint' => env('DYNAMODB_ENDPOINT'),
],
```

<a name="mongodb"></a>
#### MongoDB

Nếu bạn đang sử dụng MongoDB, một cache driver `mongodb` được cung cấp bởi package chính thức `mongodb/laravel-mongodb` và có thể được cấu hình bằng cách sử dụng một database connection `mongodb`. MongoDB hỗ trợ TTL indexes, có thể được sử dụng để tự động xóa các cache items đã hết hạn.

Để biết thêm thông tin về cấu hình MongoDB, hãy tham khảo tài liệu [Cache and Locks documentation](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/) của MongoDB.

<a name="cache-usage"></a>
## Cache Usage

<a name="obtaining-a-cache-instance"></a>
### Obtaining a Cache Instance

Để thu được một cache store instance, bạn có thể sử dụng facade `Cache`, đó là những gì chúng ta sẽ sử dụng trong suốt tài liệu này. Facade `Cache` cung cấp quyền truy cập thuận tiện, ngắn gọn đến các implementations cơ bản của Laravel cache contracts:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Show a list of all users of the application.
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```

<a name="accessing-multiple-cache-stores"></a>
#### Accessing Multiple Cache Stores

Sử dụng facade `Cache`, bạn có thể truy cập các cache stores khác nhau thông qua method `store`. Key được chuyển đến method `store` nên tương ứng với một trong các stores được liệt kê trong mảng cấu hình `stores` trong file cấu hình `cache` của bạn:

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes
```

<a name="retrieving-items-from-the-cache"></a>
### Retrieving Items From the Cache

Method `get` của facade `Cache` được sử dụng để truy xuất items từ cache. Nếu item không tồn tại trong cache, `null` sẽ được trả về. Nếu bạn muốn, bạn có thể chuyển một đối số thứ hai cho method `get` chỉ định giá trị mặc định bạn muốn được trả về nếu item không tồn tại:

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

Bạn thậm chí có thể chuyển một closure làm giá trị mặc định. Kết quả của closure sẽ được trả về nếu item được chỉ định không tồn tại trong cache. Chuyển một closure cho phép bạn trì hoãn việc truy xuất các giá trị mặc định từ database hoặc external service khác:

```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

<a name="determining-item-existence"></a>
#### Determining Item Existence

Method `has` có thể được sử dụng để xác định xem một item có tồn tại trong cache hay không. Method này cũng sẽ trả về `false` nếu item tồn tại nhưng giá trị của nó là `null`:

```php
if (Cache::has('key')) {
    // ...
}
```

<a name="incrementing-decrementing-values"></a>
#### Incrementing / Decrementing Values

Các methods `increment` và `decrement` có thể được sử dụng để điều chỉnh giá trị của các items integer trong cache. Cả hai methods này chấp nhận một đối số thứ hai tùy chọn chỉ định số lượng để tăng hoặc giảm giá trị của item:

```php
// Initialize the value if it does not exist...
Cache::add('key', 0, now()->plus(hours: 4));

// Increment or decrement the value...
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

<a name="retrieve-store"></a>
#### Retrieve and Store

Đôi khi bạn có thể muốn truy xuất một item từ cache, nhưng cũng lưu trữ một giá trị mặc định nếu item được yêu cầu không tồn tại. Ví dụ, bạn có thể muốn truy xuất tất cả người dùng từ cache hoặc, nếu họ không tồn tại, truy xuất họ từ database và thêm họ vào cache. Bạn có thể làm như vậy bằng cách sử dụng method `Cache::remember`:

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

Nếu item không tồn tại trong cache, closure được chuyển đến method `remember` sẽ được thực thi và kết quả của nó sẽ được đặt trong cache.

Nếu bạn cần biết liệu item có được truy xuất từ cache thay vì bằng cách thực thi closure đã cho, bạn có thể sử dụng method `rememberWithWarmth`. Method này trả về một mảng chứa giá trị được cache và một boolean chỉ định xem item có phải là "warm" hay không, nghĩa là nó được truy xuất từ cache và không được giải quyết từ closure:

```php
[$value, $warm] = Cache::rememberWithWarmth('users', $seconds, function () {
    return DB::table('users')->get();
});
```

Bạn có thể sử dụng method `rememberForever` để truy xuất một item từ cache hoặc lưu trữ nó mãi mãi nếu nó không tồn tại:

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

<a name="swr"></a>
#### Stale While Revalidate

Khi sử dụng method `Cache::remember`, một số người dùng có thể trải qua thời gian phản hồi chậm nếu giá trị được cache đã hết hạn. Đối với một số loại dữ liệu, có thể hữu ích để cho phép dữ liệu một phần stale được phục vụ trong khi giá trị được cache được tính toán lại trong nền, ngăn chặn một số người dùng trải qua thời gian phản hồi chậm trong khi các giá trị được cache được tính toán. Điều này thường được gọi là pattern "stale-while-revalidate", và method `Cache::flexible` cung cấp một triển khai của pattern này.

Method flexible chấp nhận một mảng chỉ định bao lâu giá trị được cache được coi là "fresh" và khi nó trở nên "stale". Giá trị đầu tiên trong mảng đại diện cho số giây cache được coi là fresh, trong khi giá trị thứ hai định nghĩa bao lâu nó có thể được phục vụ như dữ liệu stale trước khi tính toán lại là cần thiết.

Nếu một request được thực hiện trong khoảng thời gian fresh (trước giá trị đầu tiên), cache được trả về ngay lập tức mà không cần tính toán lại. Nếu một request được thực hiện trong khoảng thời gian stale (giữa hai giá trị), giá trị stale được phục vụ cho người dùng, và một [deferred function](/docs/{{version}}/helpers#deferred-functions) được đăng ký để làm mới giá trị được cache sau khi response được gửi đến người dùng. Nếu một request được thực hiện sau giá trị thứ hai, cache được coi là đã hết hạn và giá trị được tính toán lại ngay lập tức, điều này có thể dẫn đến phản hồi chậm hơn cho người dùng:

```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

<a name="retrieve-delete"></a>
#### Retrieve and Delete

Nếu bạn cần truy xuất một item từ cache và sau đó xóa item, bạn có thể sử dụng method `pull`. Giống như method `get`, `null` sẽ được trả về nếu item không tồn tại trong cache:

```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

<a name="storing-items-in-the-cache"></a>
### Storing Items in the Cache

Bạn có thể sử dụng method `put` trên facade `Cache` để lưu trữ items trong cache:

```php
Cache::put('key', 'value', $seconds = 10);
```

Nếu thời gian lưu trữ không được chuyển đến method `put`, item sẽ được lưu trữ vô thời hạn:

```php
Cache::put('key', 'value');
```

Thay vì chuyển số giây như một số nguyên, bạn cũng có thể chuyển một instance `DateTime` đại diện cho thời gian hết hạn mong muốn của item được cache:

```php
Cache::put('key', 'value', now()->plus(minutes: 10));
```

<a name="store-if-not-present"></a>
#### Store if Not Present

Method `add` sẽ chỉ thêm item vào cache nếu nó chưa tồn tại trong cache store. Method sẽ trả về `true` nếu item thực sự được thêm vào cache. Nếu không, method sẽ trả về `false`. Method `add` là một operation atomic:

```php
Cache::add('key', 'value', $seconds);
```

<a name="extending-item-lifetime"></a>
### Extending Item Lifetime

Method `touch` cho phép bạn mở rộng lifetime (TTL) của một cache item hiện có. Method `touch` sẽ trả về `true` nếu cache item tồn tại và thời gian hết hạn của nó được mở rộng thành công. Nếu item không tồn tại trong cache, method sẽ trả về `false`:

```php
Cache::touch('key', 3600);
```

Bạn có thể cung cấp một instance `DateTimeInterface`, `DateInterval`, hoặc `Carbon` để chỉ định một thời gian hết hạn chính xác:

```php
Cache::touch('key', now()->addHours(2));
```

<a name="storing-items-forever"></a>
#### Storing Items Forever

Method `forever` có thể được sử dụng để lưu trữ một item trong cache vĩnh viễn. Vì các items này sẽ không hết hạn, chúng phải được xóa thủ công khỏi cache bằng cách sử dụng method `forget`:

```php
Cache::forever('key', 'value');
```

> [!NOTE]
> Nếu bạn đang sử dụng driver Memcached, các items được lưu trữ "forever" có thể bị xóa khi cache đạt giới hạn kích thước của nó.

<a name="removing-items-from-the-cache"></a>
### Removing Items From the Cache

Bạn có thể xóa items khỏi cache bằng cách sử dụng method `forget`:

```php
Cache::forget('key');
```

Bạn cũng có thể xóa items bằng cách cung cấp số giây hết hạn bằng 0 hoặc âm:

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

Bạn có thể xóa toàn bộ cache bằng cách sử dụng method `flush`:

```php
Cache::flush();
```

Bạn có thể xóa tất cả atomic locks trong cache bằng cách sử dụng method `flushLocks`:

```php
Cache::flushLocks();
```

> [!WARNING]
> Flushing cache không tôn trọng cache "prefix" được cấu hình của bạn và sẽ xóa tất cả các entries khỏi cache. Hãy cân nhắc điều này cẩn thận khi xóa một cache được chia sẻ bởi các ứng dụng khác.

<a name="cache-memoization"></a>
### Cache Memoization

Cache driver `memo` của Laravel cho phép bạn lưu trữ tạm thời các giá trị cache đã giải quyết trong bộ nhớ trong một request hoặc job thực thi duy nhất. Điều này ngăn chặn các cache hits lặp lại trong cùng một thực thi, cải thiện hiệu suất đáng kể.

Để sử dụng cache được memoized, hãy gọi method `memo`:

```php
use Illuminate\Support\Facades\Cache;

$value = Cache::memo()->get('key');
```

Method `memo` tùy chọn chấp nhận tên của một cache store, chỉ định cache store cơ bản mà driver được memoized sẽ decorate:

```php
// Using the default cache store...
$value = Cache::memo()->get('key');

// Using the Redis cache store...
$value = Cache::memo('redis')->get('key');
```

Lệnh gọi `get` đầu tiên cho một key nhất định truy xuất giá trị từ cache store của bạn, nhưng các lệnh gọi tiếp theo trong cùng request hoặc job sẽ truy xuất giá trị từ bộ nhớ:

```php
// Hits the cache...
$value = Cache::memo()->get('key');

// Does not hit the cache, returns memoized value...
$value = Cache::memo()->get('key');
```

Khi gọi các methods sửa đổi giá trị cache (như `put`, `increment`, `remember`, v.v.), cache được memoized tự động quên giá trị được memoized và ủy quyền cho method call mutating đến cache store cơ bản:

```php
Cache::memo()->put('name', 'Taylor'); // Writes to underlying cache...
Cache::memo()->get('name');           // Hits underlying cache...
Cache::memo()->get('name');           // Memoized, does not hit cache...

Cache::memo()->put('name', 'Tim');    // Forgets memoized value, writes new value...
Cache::memo()->get('name');           // Hits underlying cache again...
```

<a name="the-cache-helper"></a>
### The Cache Helper

Ngoài việc sử dụng facade `Cache`, bạn cũng có thể sử dụng function `cache` toàn cục để truy xuất và lưu trữ dữ liệu qua cache. Khi function `cache` được gọi với một đối số chuỗi đơn lẻ, nó sẽ trả về giá trị của key đã cho:

```php
$value = cache('key');
```

Nếu bạn cung cấp một mảng các cặp key / value và thời gian hết hạn cho function, nó sẽ lưu trữ các giá trị trong cache trong thời gian được chỉ định:

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->plus(minutes: 10));
```

Khi function `cache` được gọi mà không có đối số nào, nó trả về một instance của implementation `Illuminate\Contracts\Cache\Factory`, cho phép bạn gọi các methods caching khác:

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> [!NOTE]
> Khi testing các lệnh gọi đến function `cache` toàn cục, bạn có thể sử dụng method `Cache::shouldReceive` giống như khi bạn [testing the facade](/docs/{{version}}/mocking#mocking-facades).

<a name="cache-tags"></a>
## Cache Tags

> [!WARNING]
> Cache tags không được hỗ trợ khi sử dụng các cache drivers `file`, `dynamodb`, `database`, hoặc `storage`.

<a name="storing-tagged-cache-items"></a>
### Storing Tagged Cache Items

Cache tags cho phép bạn tag các items liên quan trong cache và sau đó flush tất cả các giá trị được cache đã được gán một tag nhất định. Bạn có thể truy cập một tagged cache bằng cách chuyển vào một mảng có thứ tự của các tên tag. Ví dụ, hãy truy cập một tagged cache và `put` một giá trị vào cache:

```php
use Illuminate\Support\Facades\Cache;

Cache::tags(['people', 'artists'])->put('John', $john, $seconds);
Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);
```

<a name="accessing-tagged-cache-items"></a>
### Accessing Tagged Cache Items

Các items được lưu trữ qua tags có thể không được truy xuất mà không cung cấp các tags được sử dụng để lưu trữ giá trị. Để truy xuất một tagged cache item, chuyển cùng danh sách có thứ tự của các tags cho method `tags`, sau đó gọi method `get` với key bạn muốn truy xuất:

```php
$john = Cache::tags(['people', 'artists'])->get('John');

$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

<a name="removing-tagged-cache-items"></a>
### Removing Tagged Cache Items

Bạn có thể flush tất cả các items được gán một tag hoặc danh sách các tags. Ví dụ, code sau sẽ xóa tất cả các caches được gán với `people`, `authors`, hoặc cả hai. Vì vậy, cả `Anne` và `John` sẽ bị xóa khỏi cache:

```php
Cache::tags(['people', 'authors'])->flush();
```

Ngược lại, code dưới đây sẽ chỉ xóa các giá trị được cache được gán với `authors`, vì vậy `Anne` sẽ bị xóa, nhưng không phải `John`:

```php
Cache::tags('authors')->flush();
```

<a name="atomic-locks"></a>
## Atomic Locks

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng cache driver `memcached`, `redis`, `dynamodb`, `database`, `file`, hoặc `array` làm default cache driver của ứng dụng. Ngoài ra, tất cả các servers phải giao tiếp với cùng một central cache server.

<a name="managing-locks"></a>
### Managing Locks

Atomic locks cho phép thao tác các distributed locks mà không cần lo lắng về race conditions. Ví dụ, [Laravel Cloud](https://cloud.laravel.com) sử dụng atomic locks để đảm bảo chỉ có một remote task đang được thực thi trên một server tại một thời điểm. Bạn có thể tạo và quản lý locks bằng cách sử dụng method `Cache::lock`:

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // Lock acquired for 10 seconds...

    $lock->release();
}
```

Method `get` cũng chấp nhận một closure. Sau khi closure được thực thi, Laravel sẽ tự động release lock:

```php
Cache::lock('foo', 10)->get(function () {
    // Lock acquired for 10 seconds and automatically released...
});
```

Nếu lock không có sẵn tại thời điểm bạn yêu cầu nó, bạn có thể hướng dẫn Laravel đợi một số giây được chỉ định. Nếu lock không thể được thu được trong giới hạn thời gian được chỉ định, một `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được ném:

```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);

    // Lock acquired after waiting a maximum of 5 seconds...
} catch (LockTimeoutException $e) {
    // Unable to acquire lock...
} finally {
    $lock->release();
}
```

Ví dụ trên có thể được đơn giản hóa bằng cách chuyển một closure cho method `block`. Khi một closure được chuyển đến method này, Laravel sẽ cố gắng thu được lock trong số giây được chỉ định và sẽ tự động release lock sau khi closure đã được thực thi:

```php
Cache::lock('foo', 10)->block(5, function () {
    // Lock acquired for 10 seconds after waiting a maximum of 5 seconds...
});
```

<a name="managing-locks-across-processes"></a>
### Managing Locks Across Processes

Đôi khi, bạn có thể muốn thu được một lock trong một process và release nó trong một process khác. Ví dụ, bạn có thể thu được một lock trong một web request và muốn release lock vào cuối một queued job được kích hoạt bởi request đó. Trong kịch bản này, bạn nên chuyển "owner token" có phạm vi của lock cho queued job để job có thể re-instantiate lock bằng cách sử dụng token đã cho.

Trong ví dụ dưới đây, chúng ta sẽ dispatch một queued job nếu một lock được thu được thành công. Ngoài ra, chúng ta sẽ chuyển owner token của lock cho queued job thông qua method `owner` của lock:

```php
$podcast = Podcast::find($id);

$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

Trong job `ProcessPodcast` của ứng dụng, chúng ta có thể khôi phục và release lock bằng cách sử dụng owner token:

```php
Cache::restoreLock('processing', $this->owner)->release();
```

Nếu bạn muốn release một lock mà không tôn trọng owner hiện tại của nó, bạn có thể sử dụng method `forceRelease`:

```php
Cache::lock('processing')->forceRelease();
```

<a name="concurrency-limiting"></a>
### Concurrency Limiting

Chức năng atomic lock của Laravel cũng cung cấp một số cách để giới hạn thực thi đồng thời của các closures. Sử dụng `withoutOverlapping` khi bạn muốn chỉ cho phép một running instance trên infrastructure của mình:

```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired after waiting a maximum of 10 seconds...
});
```

Theo mặc định, lock được giữ cho đến khi closure hoàn thành thực thi, và method đợi tối đa 10 giây để thu được lock. Bạn có thể tùy chỉnh các giá trị này bằng cách sử dụng các đối số bổ sung:

```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired for 120 seconds after waiting a maximum of 5 seconds...
}, lockFor: 120, waitFor: 5);
```

Nếu lock không thể được thu được trong thời gian chờ được chỉ định, một `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được ném.

Nếu bạn muốn controlled parallelism, hãy sử dụng method `funnel` để đặt số lượng tối đa các thực thi đồng thời. Method `funnel` hoạt động với bất kỳ cache driver nào hỗ trợ locks:

```php
Cache::funnel('foo')
    ->limit(3)
    ->releaseAfter(60)
    ->block(10)
    ->then(function () {
        // Concurrency lock acquired...
    }, function () {
        // Could not acquire concurrency lock...
    });
```

Key `funnel` xác định resource đang được giới hạn. Method `limit` định nghĩa số lượng thực thi đồng thời tối đa. Method `releaseAfter` đặt một timeout an toàn tính bằng giây trước khi một slot đã thu được được tự động release. Method `block` đặt bao nhiêu giây để đợi một slot có sẵn.

Nếu bạn muốn xử lý timeout thông qua exceptions thay vì cung cấp một failure closure, bạn có thể bỏ qua closure thứ hai. Một `Illuminate\Cache\Limiters\LimiterTimeoutException` sẽ được ném nếu lock không thể được thu được trong thời gian chờ được chỉ định:

```php
use Illuminate\Cache\Limiters\LimiterTimeoutException;

try {
    Cache::funnel('foo')
        ->limit(3)
        ->releaseAfter(60)
        ->block(10)
        ->then(function () {
            // Concurrency lock acquired...
        });
} catch (LimiterTimeoutException $e) {
    // Unable to acquire concurrency lock...
}
```

Nếu bạn muốn sử dụng một cache store cụ thể cho concurrency limiter, bạn có thể gọi method `funnel` trên store mong muốn:

```php
Cache::store('redis')->funnel('foo')
    ->limit(3)
    ->block(10)
    ->then(function () {
        // Concurrency lock acquired using the "redis" store...
    });
```

> [!NOTE]
> Method `funnel` yêu cầu cache store implement interface `Illuminate\Contracts\Cache\LockProvider`. Nếu bạn cố gắng sử dụng `funnel` với một cache store không hỗ trợ locks, một `BadMethodCallException` sẽ được ném.

<a name="cache-failover"></a>
## Cache Failover

Cache driver `failover` cung cấp chức năng failover tự động khi tương tác với cache. Nếu primary cache store của store `failover` fails vì bất kỳ lý do nào, Laravel sẽ tự động cố gắng sử dụng store được cấu hình tiếp theo trong danh sách. Điều này đặc biệt hữu ích để đảm bảo high availability trong các môi trường production nơi độ tin cậy của cache là quan trọng.

Để cấu hình một failover cache store, chỉ định driver `failover` và cung cấp một mảng tên store để thử theo thứ tự. Theo mặc định, Laravel bao gồm một ví dụ cấu hình failover trong file cấu hình `config/cache.php` của ứng dụng:

```php
'failover' => [
    'driver' => 'failover',
    'stores' => [
        'database',
        'array',
    ],
],
```

Khi bạn đã cấu hình một store sử dụng driver `failover`, bạn sẽ cần đặt failover store làm default cache store của bạn trong file `.env` của ứng dụng để sử dụng chức năng failover:

```ini
CACHE_STORE=failover
```

Khi một cache store operation fails và failover được kích hoạt, Laravel sẽ dispatch event `Illuminate\Cache\Events\CacheFailedOver`, cho phép bạn báo cáo hoặc log rằng một cache store đã fail.

<a name="adding-custom-cache-drivers"></a>
## Adding Custom Cache Drivers

<a name="writing-the-driver"></a>
### Writing the Driver

Để tạo custom cache driver của chúng ta, trước tiên chúng ta cần implement contract `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts). Vì vậy, một triển khai MongoDB cache có thể trông giống như sau:

```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys) {}
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

Chúng ta chỉ cần implement từng methods này bằng cách sử dụng một MongoDB connection. Để biết ví dụ về cách implement từng methods này, hãy xem `Illuminate\Cache\MemcachedStore` trong [Laravel framework source code](https://github.com/laravel/framework). Khi triển khai của chúng ta hoàn thành, chúng ta có thể hoàn thành đăng ký custom driver của chúng ta bằng cách gọi method `extend` của facade `Cache`:

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

> [!NOTE]
> Nếu bạn thắc nơi nên đặt code custom cache driver của mình, bạn có thể tạo một namespace `Extensions` trong thư mục `app` của bạn. Tuy nhiên, hãy nhớ rằng Laravel không có cấu trúc ứng dụng cứng nhắc và bạn có thể tổ chức ứng dụng của mình theo sở thích của bạn.

<a name="registering-the-driver"></a>
### Registering the Driver

Để đăng ký custom cache driver với Laravel, chúng ta sẽ sử dụng method `extend` trên facade `Cache`. Vì các service providers khác có thể cố gắng đọc các giá trị được cache trong method `boot` của họ, chúng ta sẽ đăng ký custom driver của mình trong một `booting` callback. Bằng cách sử dụng `booting` callback, chúng ta có thể đảm bảo rằng custom driver được đăng ký ngay trước khi method `boot` được gọi trên service providers của ứng dụng nhưng sau khi method `register` được gọi trên tất cả các service providers. Chúng ta sẽ đăng ký `booting` callback của mình trong method `register` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->booting(function () {
             Cache::extend('mongo', function (Application $app) {
                 return Cache::repository(new MongoStore);
             });
         });
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // ...
    }
}
```

Đối số đầu tiên được chuyển đến method `extend` là tên của driver. Điều này sẽ tương ứng với tùy chọn `driver` của bạn trong file cấu hình `config/cache.php`. Đối số thứ hai là một closure nên trả về một instance `Illuminate\Cache\Repository`. Closure sẽ được chuyển một instance `$app`, là một instance của [service container](/docs/{{version}}/container).

Khi extension của bạn đã được đăng ký, cập nhật biến môi trường `CACHE_STORE` hoặc tùy chọn `default` trong file cấu hình `config/cache.php` của ứng dụng thành tên của extension của bạn.

<a name="events"></a>
## Events

Để thực thi code trên mọi cache operation, bạn có thể lắng nghe các [events](/docs/{{version}}/events) được dispatch bởi cache:

<div class="overflow-auto">

|| Event Name                                      ||
||-------------------------------------------------||
|| `Illuminate\Cache\Events\CacheFlushed`          ||
|| `Illuminate\Cache\Events\CacheFlushing`         ||
|| `Illuminate\Cache\Events\CacheFlushFailed`      ||
|| `Illuminate\Cache\Events\CacheLocksFlushed`     ||
|| `Illuminate\Cache\Events\CacheLocksFlushing`    ||
|| `Illuminate\Cache\Events\CacheLocksFlushFailed` ||
|| `Illuminate\Cache\Events\CacheHit`              ||
|| `Illuminate\Cache\Events\CacheMissed`           ||
|| `Illuminate\Cache\Events\ForgettingKey`         ||
|| `Illuminate\Cache\Events\KeyForgetFailed`       ||
|| `Illuminate\Cache\Events\KeyForgotten`          ||
|| `Illuminate\Cache\Events\KeyWriteFailed`        ||
|| `Illuminate\Cache\Events\KeyWritten`            ||
|| `Illuminate\Cache\Events\RetrievingKey`         ||
|| `Illuminate\Cache\Events\RetrievingManyKeys`    ||
|| `Illuminate\Cache\Events\WritingKey`            ||
|| `Illuminate\Cache\Events\WritingManyKeys`       ||

</div>

Để tăng hiệu suất, bạn có thể vô hiệu hóa cache events bằng cách đặt tùy chọn cấu hình `events` thành `false` cho một cache store nhất định trong file cấu hình `config/cache.php` của ứng dụng:

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```

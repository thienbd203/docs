# HTTP Session

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Driver Prerequisites](#driver-prerequisites)
- [Interacting With the Session](#interacting-with-the-session)
    - [Retrieving Data](#retrieving-data)
    - [Storing Data](#storing-data)
    - [Flash Data](#flash-data)
    - [Deleting Data](#deleting-data)
    - [Regenerating the Session ID](#regenerating-the-session-id)
- [Session Cache](#session-cache)
- [Session Blocking](#session-blocking)
- [Adding Custom Session Drivers](#adding-custom-session-drivers)
    - [Implementing the Driver](#implementing-the-driver)
    - [Registering the Driver](#registering-the-driver)

<a name="introduction"></a>
## Introduction

Vì các ứng dụng dựa trên HTTP là stateless, sessions cung cấp một cách để lưu trữ thông tin về người dùng trên nhiều requests. Thông tin người dùng này thường được đặt trong một persistent store / backend có thể được truy cập từ các requests tiếp theo.

Laravel được gửi kèm với nhiều session backends được truy cập thông qua một API thống nhất, biểu đạt. Hỗ trợ cho các backends phổ biến như [Memcached](https://memcached.org), [Redis](https://redis.io), và databases được bao gồm.

<a name="configuration"></a>
### Configuration

File cấu hình session của ứng dụng của bạn được lưu trữ tại `config/session.php`. Hãy đảm bảo xem xét các tùy chọn có sẵn cho bạn trong file này. Theo mặc định, Laravel được cấu hình để sử dụng session driver `database`.

Tùy chọn cấu hình session `driver` định nghĩa nơi dữ liệu session sẽ được lưu trữ cho mỗi request. Laravel bao gồm nhiều drivers:

<div class="content-list" markdown="1">

- `file` - sessions được lưu trữ trong `storage/framework/sessions`.
- `cookie` - sessions được lưu trữ trong các cookies an toàn, được mã hóa.
- `database` - sessions được lưu trữ trong một database quan hệ.
- `memcached` / `redis` - sessions được lưu trữ trong một trong các stores dựa trên cache nhanh này.
- `dynamodb` - sessions được lưu trữ trong AWS DynamoDB.
- `array` - sessions được lưu trữ trong một PHP array và sẽ không được persisted.

</div>

> [!NOTE]
> Driver array chủ yếu được sử dụng trong quá trình [testing](/docs/{{version}}/testing) và ngăn chặn dữ liệu được lưu trữ trong session được persisted.

<a name="driver-prerequisites"></a>
### Driver Prerequisites

<a name="database"></a>
#### Database

Khi sử dụng session driver `database`, bạn sẽ cần đảm bảo rằng bạn có một database table để chứa dữ liệu session. Thông thường, điều này được bao gồm trong [database migration](/docs/{{version}}/migrations) mặc định của Laravel `0001_01_01_000000_create_users_table.php`; tuy nhiên, nếu vì bất kỳ lý do nào bạn không có table `sessions`, bạn có thể sử dụng lệnh Artisan `make:session-table` để tạo migration này:

```shell
php artisan make:session-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

Trước khi sử dụng Redis sessions với Laravel, bạn sẽ cần cài đặt extension PHP PhpRedis qua PECL hoặc cài đặt package `predis/predis` (~1.0) qua Composer. Để biết thêm thông tin về cấu hình Redis, hãy tham khảo tài liệu [Redis của Laravel](/docs/{{version}}/redis#configuration).

> [!NOTE]
> Biến môi trường `SESSION_CONNECTION`, hoặc tùy chọn `connection` trong file cấu hình `session.php`, có thể được sử dụng để chỉ định kết nối Redis nào được sử dụng để lưu trữ session.

<a name="interacting-with-the-session"></a>
## Interacting With the Session

<a name="retrieving-data"></a>
### Retrieving Data

Có hai cách chính để làm việc với dữ liệu session trong Laravel: helper `session` toàn cục và thông qua một instance `Request`. Đầu tiên, hãy xem cách truy cập session thông qua một instance `Request`, có thể được type-hinted trên một route closure hoặc phương thức controller. Hãy nhớ rằng, các dependencies của phương thức controller được tự động inject thông qua [service container](/docs/{{version}}/container) của Laravel:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(Request $request, string $id): View
    {
        $value = $request->session()->get('key');

        // ...

        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

Khi bạn truy xuất một mục từ session, bạn cũng có thể chuyển một giá trị mặc định làm đối số thứ hai cho phương thức `get`. Giá trị mặc định này sẽ được trả về nếu key được chỉ định không tồn tại trong session. Nếu bạn chuyển một closure làm giá trị mặc định cho phương thức `get` và key được yêu cầu không tồn tại, closure sẽ được thực thi và kết quả của nó được trả về:

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

<a name="the-global-session-helper"></a>
#### The Global Session Helper

Bạn cũng có thể sử dụng hàm PHP `session` toàn cục để truy xuất và lưu trữ dữ liệu trong session. Khi helper `session` được gọi với một đối số chuỗi duy nhất, nó sẽ trả về giá trị của key session đó. Khi helper được gọi với một array của các cặp key / value, các giá trị đó sẽ được lưu trữ trong session:

```php
Route::get('/home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

> [!NOTE]
> Có rất ít sự khác biệt thực tế giữa việc sử dụng session thông qua một instance request HTTP so với việc sử dụng helper `session` toàn cục. Cả hai phương pháp đều có thể [test](/docs/{{version}}/testing) thông qua phương thức `assertSessionHas` có sẵn trong tất cả các test cases của bạn.

<a name="retrieving-all-session-data"></a>
#### Retrieving All Session Data

Nếu bạn muốn truy xuất tất cả dữ liệu trong session, bạn có thể sử dụng phương thức `all`:

```php
$data = $request->session()->all();
```

<a name="retrieving-a-portion-of-the-session-data"></a>
#### Retrieving a Portion of the Session Data

Các phương thức `only` và `except` có thể được sử dụng để truy xuất một tập hợp con của dữ liệu session:

```php
$data = $request->session()->only(['username', 'email']);

$data = $request->session()->except(['username', 'email']);
```

<a name="determining-if-an-item-exists-in-the-session"></a>
#### Determining if an Item Exists in the Session

Để xác định xem một mục có trong session hay không, bạn có thể sử dụng phương thức `has`. Phương thức `has` trả về `true` nếu mục có mặt và không phải là `null`:

```php
if ($request->session()->has('users')) {
    // ...
}
```

Để xác định xem một mục có trong session hay không, ngay cả khi giá trị của nó là `null`, bạn có thể sử dụng phương thức `exists`:

```php
if ($request->session()->exists('users')) {
    // ...
}
```

Để xác định xem một mục không có trong session hay không, bạn có thể sử dụng phương thức `missing`. Phương thức `missing` trả về `true` nếu mục không có mặt:

```php
if ($request->session()->missing('users')) {
    // ...
}
```

<a name="storing-data"></a>
### Storing Data

Để lưu trữ dữ liệu trong session, bạn thường sẽ sử dụng phương thức `put` của instance request hoặc helper `session` toàn cục:

```php
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global "session" helper...
session(['key' => 'value']);
```

<a name="pushing-to-array-session-values"></a>
#### Pushing to Array Session Values

Phương thức `push` có thể được sử dụng để đẩy một giá trị mới lên một giá trị session là một array. Ví dụ, nếu key `user.teams` chứa một array tên team, bạn có thể đẩy một giá trị mới lên array như sau:

```php
$request->session()->push('user.teams', 'developers');
```

<a name="retrieving-deleting-an-item"></a>
#### Retrieving and Deleting an Item

Phương thức `pull` sẽ truy xuất và xóa một mục khỏi session trong một câu lệnh duy nhất:

```php
$value = $request->session()->pull('key', 'default');
```

<a name="incrementing-and-decrementing-session-values"></a>
#### Incrementing and Decrementing Session Values

Nếu dữ liệu session của bạn chứa một số nguyên bạn muốn tăng hoặc giảm, bạn có thể sử dụng các phương thức `increment` và `decrement`:

```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

<a name="flash-data"></a>
### Flash Data

Đôi khi bạn có thể muốn lưu trữ các mục trong session cho request tiếp theo. Bạn có thể làm điều này bằng cách sử dụng phương thức `flash`. Dữ liệu được lưu trữ trong session bằng phương thức này sẽ có sẵn ngay lập tức và trong request HTTP tiếp theo. Sau request HTTP tiếp theo, dữ liệu flash sẽ bị xóa. Flash data chủ yếu hữu ích cho các thông báo trạng thái ngắn hạn:

```php
$request->session()->flash('status', 'Task was successful!');
```

Nếu bạn cần duy trì flash data của mình cho một số requests, bạn có thể sử dụng phương thức `reflash`, sẽ giữ tất cả flash data cho một request bổ sung. Nếu bạn chỉ cần giữ flash data cụ thể, bạn có thể sử dụng phương thức `keep`:

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

Để duy trì flash data của mình chỉ cho request hiện tại, bạn có thể sử dụng phương thức `now`:

```php
$request->session()->now('status', 'Task was successful!');
```

<a name="deleting-data"></a>
### Deleting Data

Phương thức `forget` sẽ xóa một phần dữ liệu khỏi session. Nếu bạn muốn xóa tất cả dữ liệu khỏi session, bạn có thể sử dụng phương thức `flush`:

```php
// Forget a single key...
$request->session()->forget('name');

// Forget multiple keys...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```

<a name="regenerating-the-session-id"></a>
### Regenerating the Session ID

Tái tạo session ID thường được thực hiện để ngăn chặn người dùng độc hại khai thác một cuộc tấn công [session fixation](https://owasp.org/www-community/attacks/Session_fixation) trên ứng dụng của bạn.

Laravel tự động tái tạo session ID trong quá trình xác thực nếu bạn đang sử dụng một trong các [application starter kits](/docs/{{version}}/starter-kits) của Laravel hoặc [Laravel Fortify](/docs/{{version}}/fortify); tuy nhiên, nếu bạn cần tái tạo session ID thủ công, bạn có thể sử dụng phương thức `regenerate`:

```php
$request->session()->regenerate();
```

Nếu bạn cần tái tạo session ID và xóa tất cả dữ liệu khỏi session trong một câu lệnh duy nhất, bạn có thể sử dụng phương thức `invalidate`:

```php
$request->session()->invalidate();
```

<a name="session-cache"></a>
## Session Cache

Session cache của Laravel cung cấp một cách thuận tiện để cache dữ liệu được scoped cho một session người dùng riêng lẻ. Không giống như cache ứng dụng toàn cục, dữ liệu session cache được tự động cô lập theo session và được dọn dẹp khi session hết hạn hoặc bị hủy. Session cache hỗ trợ tất cả các [phương thức cache của Laravel](/docs/{{version}}/cache) quen thuộc như `get`, `put`, `remember`, `forget`, và nhiều hơn nữa, nhưng được scoped cho session hiện tại.

Session cache hoàn hảo để lưu trữ dữ liệu tạm thời, cụ thể cho người dùng mà bạn muốn duy trì qua nhiều requests trong cùng một session, nhưng không cần lưu trữ vĩnh viễn. Điều này bao gồm những thứ như dữ liệu form, tính toán tạm thời, phản hồi API, hoặc bất kỳ dữ liệu tạm thời nào khác nên được gắn với session của một người dùng cụ thể.

Bạn có thể truy cập session cache thông qua phương thức `cache` trên session:

```php
$discount = $request->session()->cache()->get('discount');

$request->session()->cache()->put(
    'discount', 10, now()->plus(minutes: 5)
);
```

Để biết thêm thông tin về các phương thức cache của Laravel, hãy tham khảo tài liệu [cache](/docs/{{version}}/cache).

<a name="session-blocking"></a>
## Session Blocking

> [!WARNING]
> Để sử dụng session blocking, ứng dụng của bạn phải sử dụng một cache driver hỗ trợ [atomic locks](/docs/{{version}}/cache#atomic-locks). Hiện tại, các cache drivers này bao gồm các drivers `memcached`, `dynamodb`, `redis`, `mongodb` (bao gồm trong package chính thức `mongodb/laravel-mongodb`), `database`, `file`, và `array`. Ngoài ra, bạn không thể sử dụng session driver `cookie`.

Theo mặc định, Laravel cho phép các requests sử dụng cùng một session thực thi đồng thời. Vì vậy, ví dụ, nếu bạn sử dụng một thư viện HTTP JavaScript để thực hiện hai HTTP requests đến ứng dụng của bạn, chúng sẽ thực thi cùng một lúc. Đối với nhiều ứng dụng, điều này không phải là vấn đề; tuy nhiên, mất dữ liệu session có thể xảy ra trong một tập hợp con nhỏ các ứng dụng thực hiện các requests đồng thời đến hai endpoints ứng dụng khác nhau cả hai đều ghi dữ liệu vào session.

Để giảm thiểu điều này, Laravel cung cấp chức năng cho phép bạn giới hạn các requests đồng thời cho một session cụ thể. Để bắt đầu, bạn có thể chỉ cần chain phương thức `block` vào định nghĩa route của bạn. Trong ví dụ này, một request đến endpoint `/profile` sẽ có được một session lock. Trong khi lock này được giữ, bất kỳ requests đến endpoints `/profile` hoặc `/order` nào chia sẻ cùng một session ID sẽ đợi request đầu tiên hoàn thành thực thi trước khi tiếp tục thực thi của chúng:

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);
```

Phương thức `block` chấp nhận hai đối số tùy chọn. Đối số đầu tiên được chấp nhận bởi phương thức `block` là số giây tối đa mà session lock nên được giữ trước khi được giải phóng. Tất nhiên, nếu request hoàn thành thực thi trước thời gian này, lock sẽ được giải phóng sớm hơn.

Đối số thứ hai được chấp nhận bởi phương thức `block` là số giây một request nên đợi trong khi cố gắng có được một session lock. Một `Illuminate\Contracts\Cache\LockTimeoutException` sẽ được throw nếu request không thể có được một session lock trong số giây được chỉ định.

Nếu không có đối số nào trong số này được chuyển, lock sẽ được có được tối đa 10 giây và các requests sẽ đợi tối đa 10 giây trong khi cố gắng có được một lock:

```php
Route::post('/profile', function () {
    // ...
})->block();
```

<a name="adding-custom-session-drivers"></a>
## Adding Custom Session Drivers

<a name="implementing-the-driver"></a>
### Implementing the Driver

Nếu không có session driver hiện tại nào phù hợp với nhu cầu của ứng dụng của bạn, Laravel cho phép bạn viết session handler của riêng mình. Session driver tùy chỉnh của bạn nên triển khai `SessionHandlerInterface` tích hợp sẵn của PHP. Interface này chỉ chứa một vài phương thức đơn giản. Một triển khai MongoDB stubbed trông như sau:

```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

Vì Laravel không bao gồm một thư mục mặc định để chứa các extensions của bạn. Bạn tự do đặt chúng ở bất cứ đâu bạn thích. Trong ví dụ này, chúng tôi đã tạo một thư mục `Extensions` để chứa `MongoSessionHandler`.

Vì mục đích của các phương thức này không dễ hiểu ngay lập tức, đây là tổng quan về mục đích của mỗi phương thức:

<div class="content-list" markdown="1">

- Phương thức `open` thường được sử dụng trong các hệ thống lưu trữ session dựa trên file. Vì Laravel được gửi kèm với một session driver `file`, bạn hiếm khi cần đặt bất cứ thứ gì trong phương thức này. Bạn có thể chỉ cần để phương thức này trống.
- Phương thức `close`, giống như phương thức `open`, cũng thường có thể bị bỏ qua. Đối với hầu hết các drivers, nó không cần thiết.
- Phương thức `read` nên trả về phiên bản chuỗi của dữ liệu session liên quan đến `$sessionId` đã cho. Không cần thực hiện bất kỳ serialization hoặc mã hóa nào khi truy xuất hoặc lưu trữ dữ liệu session trong driver của bạn, vì Laravel sẽ thực hiện serialization cho bạn.
- Phương thức `write` nên ghi chuỗi `$data` đã cho liên quan đến `$sessionId` vào một hệ thống lưu trữ persistent, chẳng hạn như MongoDB hoặc hệ thống lưu trữ khác theo lựa chọn của bạn. Một lần nữa, bạn không nên thực hiện bất kỳ serialization nào - Laravel đã xử lý điều đó cho bạn.
- Phương thức `destroy` nên xóa dữ liệu liên quan đến `$sessionId` khỏi lưu trữ persistent.
- Phương thức `gc` nên hủy tất cả dữ liệu session cũ hơn `$lifetime` đã cho, là một timestamp UNIX. Đối với các hệ thống tự hết hạn như Memcached và Redis, phương thức này có thể được để trống.

</div>

<a name="registering-the-driver"></a>
### Registering the Driver

Sau khi driver của bạn đã được triển khai, bạn đã sẵn sàng để đăng ký nó với Laravel. Để thêm các drivers bổ sung vào backend session của Laravel, bạn có thể sử dụng phương thức `extend` được cung cấp bởi [facade](/docs/{{version}}/facades) `Session`. Bạn nên gọi phương thức `extend` từ phương thức `boot` của một [service provider](/docs/{{version}}/providers). Bạn có thể làm điều này từ `App\Providers\AppServiceProvider` hiện có hoặc tạo một provider hoàn toàn mới:

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
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
        Session::extend('mongo', function (Application $app) {
            // Return an implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }
}
```

Sau khi session driver đã được đăng ký, bạn có thể chỉ định driver `mongo` làm session driver của ứng dụng của bạn bằng cách sử dụng biến môi trường `SESSION_DRIVER` hoặc trong file cấu hình `config/session.php` của ứng dụng.

# Facades

- [Giới thiệu](#introduction)
- [Khi nào nên sử dụng Facades](#when-to-use-facades)
    - [Facades so với Dependency Injection](#facades-vs-dependency-injection)
    - [Facades so với Helper Functions](#facades-vs-helper-functions)
- [Cách Facades hoạt động](#how-facades-work)
- [Real-Time Facades](#real-time-facades)
- [Tham khảo Facade Class](#facade-class-reference)

<a name="introduction"></a>
## Giới thiệu

Trong suốt Laravel documentation, bạn sẽ thấy các ví dụ code tương tác với Laravel features thông qua "facades". Facades cung cấp một interface "static" cho các classes có sẵn trong [service container](/docs/{{version}}/container) của application. Laravel đi kèm với nhiều facades cung cấp access đến hầu hết các Laravel features.

Laravel facades đóng vai trò là "static proxies" cho các classes bên dưới trong service container, cung cấp lợi ích của một cú pháp ngắn gọn, expressive trong khi duy trì tính testability và flexibility cao hơn so với traditional static methods. Hoàn toàn ổn nếu bạn không hoàn toàn hiểu cách facades hoạt động - chỉ cần đi theo flow và tiếp tục học về Laravel.

Tất cả Laravel facades đều được định nghĩa trong namespace `Illuminate\Support\Facades`. Vì vậy, chúng ta có thể dễ dàng access một facade như sau:

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

Trong suốt Laravel documentation, nhiều ví dụ sẽ sử dụng facades để demonstrate các features khác nhau của framework.

<a name="helper-functions"></a>
#### Helper Functions

Để bổ sung cho facades, Laravel cung cấp nhiều global "helper functions" giúp tương tác với common Laravel features dễ dàng hơn. Một số common helper functions bạn có thể tương tác là `view`, `response`, `url`, `config`, và nhiều hơn nữa. Mỗi helper function được cung cấp bởi Laravel đều được documented với feature tương ứng của chúng; tuy nhiên, một danh sách đầy đủ có sẵn trong dedicated [helper documentation](/docs/{{version}}/helpers).

Ví dụ, thay vì sử dụng facade `Illuminate\Support\Facades\Response` để generate một JSON response, chúng ta có thể đơn giản sử dụng function `response`. Vì helper functions có sẵn globally, bạn không cần import bất kỳ classes nào để sử dụng chúng:

```php
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

<a name="when-to-use-facades"></a>
## Khi nào nên sử dụng Facades

Facades có nhiều lợi ích. Chúng cung cấp một cú pháp ngắn gọn, dễ nhớ cho phép bạn sử dụng Laravel features mà không cần nhớ long class names phải được inject hoặc configure thủ công. Hơn nữa, vì cách sử dụng unique của PHP dynamic methods của chúng, chúng dễ test.

Tuy nhiên, cần phải cẩn thận khi sử dụng facades. Danger chính của facades là class "scope creep". Vì facades rất dễ sử dụng và không cần injection, có thể dễ để cho classes của bạn tiếp tục grow và sử dụng nhiều facades trong một class. Sử dụng dependency injection, potential này được mitigate bởi visual feedback mà một large constructor cung cấp cho bạn rằng class của bạn đang grow quá lớn. Vì vậy, khi sử dụng facades, hãy chú ý đặc biệt đến kích thước class của bạn để scope of responsibility của nó giữ narrow. Nếu class của bạn đang trở quá lớn, hãy cân nhắc split nó thành nhiều smaller classes.

<a name="facades-vs-dependency-injection"></a>
### Facades so với Dependency Injection

Một trong những lợi ích chính của dependency injection là khả năng swap implementations của injected class. Điều này hữu ích trong quá trình testing vì bạn có thể inject một mock hoặc stub và assert rằng các methods khác nhau đã được gọi trên stub.

Thông thường, sẽ không thể mock hoặc stub một truly static class method. Tuy nhiên, vì facades sử dụng dynamic methods để proxy method calls đến objects được resolve từ service container, chúng ta thực sự có thể test facades giống như cách chúng ta test một injected class instance. Ví dụ, với route sau:

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

Sử dụng Laravel facade testing methods, chúng ta có thể viết test sau để verify rằng method `Cache::get` đã được gọi với argument chúng ta mong đợi:

```php tab=Pest
use Illuminate\Support\Facades\Cache;

test('basic example', function () {
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```

```php tab=PHPUnit
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="facades-vs-helper-functions"></a>
### Facades so với Helper Functions

Ngoài facades, Laravel bao gồm nhiều "helper" functions có thể perform common tasks như generating views, firing events, dispatching jobs, hoặc sending HTTP responses. Nhiều helper functions này perform cùng function như một facade tương ứng. Ví dụ, facade call và helper call này là tương đương:

```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```

Không có sự khác biệt thực tế nào giữa facades và helper functions. Khi sử dụng helper functions, bạn vẫn có thể test chúng chính xác như cách bạn test facade tương ứng. Ví dụ, với route sau:

```php
Route::get('/cache', function () {
    return cache('key');
});
```

Helper `cache` sẽ gọi method `get` trên class bên dưới facade `Cache`. Vì vậy, mặc dù chúng ta đang sử dụng helper function, chúng ta có thể viết test sau để verify rằng method đã được gọi với argument chúng ta mong đợi:

```php
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="how-facades-work"></a>
## Cách Facades hoạt động

Trong một Laravel application, facade là một class cung cấp access đến một object từ container. Machinery làm cho điều này hoạt động nằm trong class `Facade`. Laravel facades, và bất kỳ custom facades nào bạn tạo, sẽ extend base class `Illuminate\Support\Facades\Facade`.

Base class `Facade` sử dụng magic-method `__callStatic()` để defer calls từ facade của bạn đến một object được resolve từ container. Trong ví dụ dưới đây, một call được thực hiện đến Laravel cache system. Bằng cách nhìn vào code này, người ta có thể giả định rằng static method `get` đang được gọi trên class `Cache`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

Lưu ý rằng gần đầu file chúng ta đang "importing" facade `Cache`. Facade này đóng vai trò là proxy để access underlying implementation của interface `Illuminate\Contracts\Cache\Factory`. Bất kỳ calls nào chúng ta thực hiện sử dụng facade sẽ được passed đến underlying instance của Laravel cache service.

Nếu chúng ta nhìn vào class `Illuminate\Support\Facades\Cache` đó, bạn sẽ thấy rằng không có static method `get`:

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

Thay vào đó, facade `Cache` extends base class `Facade` và định nghĩa method `getFacadeAccessor()`. Job của method này là trả về tên của một service container binding. Khi một user references bất kỳ static method nào trên facade `Cache`, Laravel resolve binding `cache` từ [service container](/docs/{{version}}/container) và chạy requested method (trong trường hợp này, `get`) trên object đó.

<a name="real-time-facades"></a>
## Real-Time Facades

Sử dụng real-time facades, bạn có thể treat bất kỳ class nào trong application của bạn như thể nó là một facade. Để illustrate cách này có thể được sử dụng, hãy trước tiên examine một số code không sử dụng real-time facades. Ví dụ, hãy giả định model `Podcast` của chúng ta có một method `publish`. Tuy nhiên, để publish podcast, chúng ta cần inject một instance `Publisher`:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

Injecting một publisher implementation vào method cho phép chúng ta dễ dàng test method đó một cách isolated vì chúng ta có thể mock injected publisher. Tuy nhiên, nó yêu cầu chúng ta luôn pass một publisher instance mỗi khi chúng ta gọi method `publish`. Sử dụng real-time facades, chúng ta có thể maintain cùng testability trong khi không cần explicitly pass một instance `Publisher`. Để generate một real-time facade, prefix namespace của imported class với `Facades`:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher; // [tl! remove]
use Facades\App\Contracts\Publisher; // [tl! add]
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void // [tl! remove]
    public function publish(): void // [tl! add]
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this); // [tl! remove]
        Publisher::publish($this); // [tl! add]
    }
}
```

Khi real-time facade được sử dụng, publisher implementation sẽ được resolve từ service container sử dụng phần của interface hoặc class name xuất hiện sau prefix `Facades`. Khi testing, chúng ta có thể sử dụng Laravel built-in facade testing helpers để mock method call này:

```php tab=Pest
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('podcast can be published', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A test example.
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

<a name="facade-class-reference"></a>
## Tham khảo Facade Class

Dưới đây bạn sẽ tìm thấy mọi facade và underlying class của nó. Đây là một công cụ hữu ích để quickly dig vào API documentation cho một facade root nhất định. [Service container binding](/docs/{{version}}/container) key cũng được bao gồm khi applicable.

<div class="overflow-auto">

| Facade | Class | Service Container Binding |
| --- | --- | --- |
| App | [Illuminate\Foundation\Application](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Application.html) | `app` |
| Artisan | [Illuminate\Contracts\Console\Kernel](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Console/Kernel.html) | `artisan` |
| Auth (Instance) | [Illuminate\Contracts\Auth\Guard](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Guard.html) | `auth.driver` |
| Auth | [Illuminate\Auth\AuthManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/AuthManager.html) | `auth` |
| Blade | [Illuminate\View\Compilers\BladeCompiler](https://api.laravel.com/docs/{{version}}/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html) | &nbsp; |
| Broadcast | [Illuminate\Contracts\Broadcasting\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html) | &nbsp; |
| Bus | [Illuminate\Contracts\Bus\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html) | &nbsp; |
| Cache (Instance) | [Illuminate\Cache\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/Repository.html) | `cache.store` |
| Cache | [Illuminate\Cache\CacheManager](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/CacheManager.html) | `cache` |
| Config | [Illuminate\Config\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Config/Repository.html) | `config` |
| Context | [Illuminate\Log\Context\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Log/Context/Repository.html) | &nbsp; |
| Cookie | [Illuminate\Cookie\CookieJar](https://api.laravel.com/docs/{{version}}/Illuminate/Cookie/CookieJar.html) | `cookie` |
| Crypt | [Illuminate\Encryption\Encrypter](https://api.laravel.com/docs/{{version}}/Illuminate/Encryption/Encrypter.html) | `encrypter` |
| Date | [Illuminate\Support\DateFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Support/DateFactory.html) | `date` |
| DB (Instance) | [Illuminate\Database\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Connection.html) | `db.connection` |
| DB | [Illuminate\Database\DatabaseManager](https://api.laravel.com/docs/{{version}}/Illuminate/Database/DatabaseManager.html) | `db` |
| Event | [Illuminate\Events\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Events/Dispatcher.html) | `events` |
| Exceptions (Instance) | [Illuminate\Contracts\Debug\ExceptionHandler](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Debug/ExceptionHandler.html) | &nbsp; |
| Exceptions | [Illuminate\Foundation\Exceptions\Handler](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Exceptions/Handler.html) | &nbsp; |
| File | [Illuminate\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/Filesystem.html) | `files` |
| Gate | [Illuminate\Contracts\Auth\Access\Gate](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html) | &nbsp; |
| Hash | [Illuminate\Contracts\Hashing\Hasher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Hashing/Hasher.html) | `hash` |
| Http | [Illuminate\Http\Client\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Client/Factory.html) | &nbsp; |
| Lang | [Illuminate\Translation\Translator](https://api.laravel.com/docs/{{version}}/Illuminate/Translation/Translator.html) | `translator` |
| Log | [Illuminate\Log\LogManager](https://api.laravel.com/docs/{{version}}/Illuminate/Log/LogManager.html) | `log` |
| Mail | [Illuminate\Mail\Mailer](https://api.laravel.com/docs/{{version}}/Illuminate/Mail/Mailer.html) | `mailer` |
| Notification | [Illuminate\Notifications\ChannelManager](https://api.laravel.com/docs/{{version}}/Illuminate/Notifications/ChannelManager.html) | &nbsp; |
| Password (Instance) | [Illuminate\Auth\Passwords\PasswordBroker](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker` |
| Password | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password` |
| Pipeline (Instance) | [Illuminate\Pipeline\Pipeline](https://api.laravel.com/docs/{{version}}/Illuminate/Pipeline/Pipeline.html) | &nbsp; |
| Process | [Illuminate\Process\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Process/Factory.html) | &nbsp; |
| Queue (Base Class) | [Illuminate\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/Queue.html) | &nbsp; |
| Queue (Instance) | [Illuminate\Contracts\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Queue/Queue.html) | `queue.connection` |
| Queue | [Illuminate\Queue\QueueManager](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/QueueManager.html) | `queue` |
| RateLimiter | [Illuminate\Cache\RateLimiter](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/RateLimiter.html) | &nbsp; |
| Redirect | [Illuminate\Routing\Redirector](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Redirector.html) | `redirect` |
| Redis (Instance) | [Illuminate\Redis\Connections\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/Connections/Connection.html) | `redis.connection` |
| Redis | [Illuminate\Redis\RedisManager](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/RedisManager.html) | `redis` |
| Request | [Illuminate\Http\Request](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Request.html) | `request` |
| Response (Instance) | [Illuminate\Http\Response](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Response.html) | &nbsp; |
| Response | [Illuminate\Contracts\Routing\ResponseFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html) | &nbsp; |
| Route | [Illuminate\Routing\Router](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Router.html) | `router` |
| Schedule | [Illuminate\Console\Scheduling\Schedule](https://api.laravel.com/docs/{{version}}/Illuminate/Console/Scheduling/Schedule.html) | &nbsp; |
| Schema | [Illuminate\Database\Schema\Builder](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Schema/Builder.html) | &nbsp; |
| Session (Instance) | [Illuminate\Session\Store](https://api.laravel.com/docs/{{version}}/Illuminate/Session/Store.html) | `session.store` |
| Session | [Illuminate\Session\SessionManager](https://api.laravel.com/docs/{{version}}/Illuminate/Session/SessionManager.html) | `session` |
| Storage (Instance) | [Illuminate\Contracts\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk` |
| Storage | [Illuminate\Filesystem\FilesystemManager](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/FilesystemManager.html) | `filesystem` |
| URL | [Illuminate\Routing\UrlGenerator](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/UrlGenerator.html) | `url` |
| Validator (Instance) | [Illuminate\Validation\Validator](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Validator.html) | &nbsp; |
| Validator | [Illuminate\Validation\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Factory.html) | `validator` |
| View (Instance) | [Illuminate\View\View](https://api.laravel.com/docs/{{version}}/Illuminate/View/View.html) | &nbsp; |
| View | [Illuminate\View\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/View/Factory.html) | `view` |
| Vite | [Illuminate\Foundation\Vite](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Vite.html) | &nbsp; |

</div>

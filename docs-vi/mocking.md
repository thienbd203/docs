# Mocking

- [Introduction](#introduction)
- [Mocking Objects](#mocking-objects)
- [Mocking Facades](#mocking-facades)
    - [Facade Spies](#facade-spies)
- [Interacting With Time](#interacting-with-time)

<a name="introduction"></a>
## Introduction

Khi test các ứng dụng Laravel, bạn có thể muốn "mock" một số khía cạnh của ứng dụng để chúng không thực sự được thực thi trong một test đã cho. Ví dụ, khi test một controller dispatches một event, bạn có thể muốn mock các event listeners để chúng không thực sự được thực thi trong test. Điều này cho phép bạn chỉ test HTTP response của controller mà không phải lo lắng về việc thực thi của event listeners vì event listeners có thể được test trong test case riêng của chúng.

Laravel cung cấp các methods hữu ích để mock events, jobs, và các facades khác ngay từ đầu. Các helpers này chủ yếu cung cấp một convenience layer trên Mockery để bạn không phải thực hiện thủ công các method calls Mockery phức tạp.

<a name="mocking-objects"></a>
## Mocking Objects

Khi mock một object sẽ được inject vào ứng dụng của bạn thông qua [service container](/docs/{{version}}/container) của Laravel, bạn sẽ cần bind mocked instance của bạn vào container như một `instance` binding. Điều này sẽ hướng dẫn container sử dụng mocked instance của object của bạn thay vì tự xây dựng object:

```php tab=Pest
use App\Service;
use Mockery;
use Mockery\MockInterface;

test('something can be mocked', function () {
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
        })
    );
});
```

```php tab=PHPUnit
use App\Service;
use Mockery;
use Mockery\MockInterface;

public function test_something_can_be_mocked(): void
{
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
        })
    );
}
```

Để làm điều này thuận tiện hơn, bạn có thể sử dụng method `mock` được cung cấp bởi base test case class của Laravel. Ví dụ, ví dụ sau tương đương với ví dụ trên:

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->mock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

Bạn có thể sử dụng method `partialMock` khi bạn chỉ cần mock một vài methods của một object. Các methods không được mock sẽ được thực thi bình thường khi được gọi:

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->partialMock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

Tương tự, nếu bạn muốn [spy](http://docs.mockery.io/en/latest/reference/spies.html) trên một object, base test case class của Laravel cung cấp một method `spy` như một wrapper tiện lợi quanh method `Mockery::spy`. Spies tương tự như mocks; tuy nhiên, spies ghi lại bất kỳ tương tác nào giữa spy và code đang được test, cho phép bạn tạo assertions sau khi code được thực thi:

```php
use App\Service;

$spy = $this->spy(Service::class);

// ...

$spy->shouldHaveReceived('process');
```

<a name="mocking-facades"></a>
## Mocking Facades

Khác với các static method calls truyền thống, [facades](/docs/{{version}}/facades) (bao gồm [real-time facades](/docs/{{version}}/facades#real-time-facades)) có thể được mock. Điều này cung cấp một lợi thế lớn so với các static methods truyền thống và trao cho bạn cùng testability mà bạn sẽ có nếu bạn sử dụng traditional dependency injection. Khi test, bạn thường muốn mock một call đến một Laravel facade xảy ra trong một trong các controllers của bạn. Ví dụ, hãy xem controller action sau:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Retrieve a list of all users of the application.
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

Chúng ta có thể mock call đến facade `Cache` bằng cách sử dụng method `expects`, sẽ trả về một instance của một [Mockery](https://github.com/padraic/mockery) mock. Vì facades thực sự được resolve và quản lý bởi [service container](/docs/{{version}}/container) của Laravel, chúng có nhiều testability hơn một static class điển hình. Ví dụ, hãy mock call của chúng ta đến method `get` của facade `Cache`:

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('get index', function () {
    Cache::expects('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/users');

    // ...
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    public function test_get_index(): void
    {
        Cache::expects('get')
            ->with('key')
            ->andReturn('value');

        $response = $this->get('/users');

        // ...
    }
}
```

> [!WARNING]
> Bạn không nên mock facade `Request`. Thay vào đó, truyền input bạn muốn vào các [HTTP testing methods](/docs/{{version}}/http-tests) như `get` và `post` khi chạy test của bạn. Tương tự, thay vì mock facade `Config`, hãy gọi method `Config::set` trong tests của bạn.

<a name="facade-spies"></a>
### Facade Spies

Nếu bạn muốn [spy](http://docs.mockery.io/en/latest/reference/spies.html) trên một facade, bạn có thể gọi method `spy` trên facade tương ứng. Spies tương tự như mocks; tuy nhiên, spies ghi lại bất kỳ tương tác nào giữa spy và code đang được test, cho phép bạn tạo assertions sau khi code được thực thi:

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('values are stored in cache', function () {
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
});
```

```php tab=PHPUnit
use Illuminate\Support\Facades\Cache;

public function test_values_are_stored_in_cache(): void
{
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
}
```

<a name="interacting-with-time"></a>
## Interacting With Time

Khi test, bạn có thể thỉnh thoảng cần sửa đổi thời gian được trả về bởi các helpers như `now` hoặc `Illuminate\Support\Carbon::now()`. May mắn thay, base feature test class của Laravel bao gồm các helpers cho phép bạn thao tác thời gian hiện tại:

```php tab=Pest
test('time can be manipulated', function () {
    // Travel into the future...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Travel into the past...
    $this->travel(-5)->hours();

    // Travel to an explicit time...
    $this->travelTo(now()->minus(hours: 6));

    // Return back to the present time...
    $this->travelBack();
});
```

```php tab=PHPUnit
public function test_time_can_be_manipulated(): void
{
    // Travel into the future...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Travel into the past...
    $this->travel(-5)->hours();

    // Travel to an explicit time...
    $this->travelTo(now()->minus(hours: 6));

    // Return back to the present time...
    $this->travelBack();
}
```

Bạn cũng có thể cung cấp một closure cho các time travel methods khác nhau. Closure sẽ được gọi với thời gian được đóng băng tại thời gian đã chỉ định. Sau khi closure đã thực thi, thời gian sẽ tiếp tục bình thường:

```php
$this->travel(5)->days(function () {
    // Test something five days into the future...
});

$this->travelTo(now()->mins(days: 10), function () {
    // Test something during a given moment...
});
```

Method `freezeTime` có thể được sử dụng để đóng băng thời gian hiện tại. Tương tự, method `freezeSecond` sẽ đóng băng thời gian hiện tại nhưng ở đầu của giây hiện tại:

```php
use Illuminate\Support\Carbon;

// Freeze time and resume normal time after executing closure...
$this->freezeTime(function (Carbon $time) {
    // ...
});

// Freeze time at the current second and resume normal time after executing closure...
$this->freezeSecond(function (Carbon $time) {
    // ...
})
```

Như bạn mong đợi, tất cả các methods được thảo luận ở trên chủ yếu hữu ích để test các hành vi ứng dụng nhạy cảm về thời gian, chẳng hạn như locking inactive posts trên một discussion forum:

```php tab=Pest
use App\Models\Thread;

test('forum threads lock after one week of inactivity', function () {
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    expect($thread->isLockedByInactivity())->toBeTrue();
});
```

```php tab=PHPUnit
use App\Models\Thread;

public function test_forum_threads_lock_after_one_week_of_inactivity()
{
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    $this->assertTrue($thread->isLockedByInactivity());
}
```

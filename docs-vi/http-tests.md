# HTTP Tests

- [Introduction](#introduction)
- [Making Requests](#making-requests)
    - [Customizing Request Headers](#customizing-request-headers)
    - [Cookies](#cookies)
    - [Session / Authentication](#session-and-authentication)
    - [Debugging Responses](#debugging-responses)
    - [Exception Handling](#exception-handling)
- [Testing JSON APIs](#testing-json-apis)
    - [Fluent JSON Testing](#fluent-json-testing)
- [Testing File Uploads](#testing-file-uploads)
- [Testing Views](#testing-views)
    - [Rendering Blade and Components](#rendering-blade-and-components)
- [Caching Routes](#caching-routes)
- [Available Assertions](#available-assertions)
    - [Response Assertions](#response-assertions)
    - [Authentication Assertions](#authentication-assertions)
    - [Validation Assertions](#validation-assertions)

<a name="introduction"></a>
## Introduction

Laravel cung cấp một API rất fluent để tạo HTTP request đến ứng dụng của bạn và kiểm tra các response. Ví dụ, hãy xem feature test được định nghĩa dưới đây:

```php tab=Pest
<?php

test('the application returns a successful response', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_the_application_returns_a_successful_response(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

Method `get` tạo một `GET` request vào ứng dụng, trong khi method `assertStatus` xác nhận rằng response được trả về phải có HTTP status code đã cho. Ngoài assertion đơn giản này, Laravel cũng chứa nhiều assertion để kiểm tra response headers, content, JSON structure, và nhiều hơn nữa.

<a name="making-requests"></a>
## Making Requests

Để tạo một request đến ứng dụng của bạn, bạn có thể gọi các method `get`, `post`, `put`, `patch`, hoặc `delete` trong test của bạn. Các method này không thực sự phát ra một "real" HTTP request đến ứng dụng của bạn. Thay vào đó, toàn bộ network request được mô phỏng nội bộ.

Thay vì trả về một instance `Illuminate\Http\Response`, các method test request trả về một instance `Illuminate\Testing\TestResponse`, cung cấp [nhiều assertion hữu ích](#available-assertions) cho phép bạn kiểm tra các response của ứng dụng:

```php tab=Pest
<?php

test('basic request', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_a_basic_request(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

Nói chung, mỗi test của bạn chỉ nên tạo một request đến ứng dụng. Hành vi không mong muốn có thể xảy ra nếu nhiều request được thực thi trong một single test method.

> [!NOTE]
> Để thuận tiện, CSRF middleware tự động bị tắt khi chạy test.

<a name="customizing-request-headers"></a>
### Customizing Request Headers

Bạn có thể sử dụng method `withHeaders` để tùy chỉnh headers của request trước khi nó được gửi đến ứng dụng. Method này cho phép bạn thêm bất kỳ custom headers nào bạn muốn vào request:

```php tab=Pest
<?php

test('interacting with headers', function () {
    $response = $this->withHeaders([
        'X-Header' => 'Value',
    ])->post('/user', ['name' => 'Sally']);

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_interacting_with_headers(): void
    {
        $response = $this->withHeaders([
            'X-Header' => 'Value',
        ])->post('/user', ['name' => 'Sally']);

        $response->assertStatus(201);
    }
}
```

<a name="cookies"></a>
### Cookies

Bạn có thể sử dụng method `withCookie` hoặc `withCookies` để thiết lập cookie values trước khi tạo request. Method `withCookie` chấp nhận cookie name và value như hai argument của nó, trong khi method `withCookies` chấp nhận một array của name / value pairs:

```php tab=Pest
<?php

test('interacting with cookies', function () {
    $response = $this->withCookie('color', 'blue')->get('/');

    $response = $this->withCookies([
        'color' => 'blue',
        'name' => 'Taylor',
    ])->get('/');

    //
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_cookies(): void
    {
        $response = $this->withCookie('color', 'blue')->get('/');

        $response = $this->withCookies([
            'color' => 'blue',
            'name' => 'Taylor',
        ])->get('/');

        //
    }
}
```

<a name="session-and-authentication"></a>
### Session / Authentication

Laravel cung cấp một số helper để tương tác với session trong HTTP testing. Đầu tiên, bạn có thể thiết lập session data thành một array đã cho sử dụng method `withSession`. Điều này hữu ích để load session với data trước khi phát ra request đến ứng dụng của bạn:

```php tab=Pest
<?php

test('interacting with the session', function () {
    $response = $this->withSession(['banned' => false])->get('/');

    //
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_the_session(): void
    {
        $response = $this->withSession(['banned' => false])->get('/');

        //
    }
}
```

Session của Laravel thường được sử dụng để duy trì state cho user hiện tại được authenticated. Do đó, helper method `actingAs` cung cấp một cách đơn giản để authenticate một user đã cho như current user. Ví dụ, chúng ta có thể sử dụng [model factory](/docs/{{version}}/eloquent-factories) để tạo và authenticate một user:

```php tab=Pest
<?php

use App\Models\User;

test('an action that requires authentication', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->withSession(['banned' => false])
        ->get('/');

    //
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_an_action_that_requires_authentication(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->withSession(['banned' => false])
            ->get('/');

        //
    }
}
```

Bạn cũng có thể chỉ định guard nào nên được sử dụng để authenticate user đã cho bằng cách truyền guard name như argument thứ hai cho method `actingAs`. Guard được cung cấp cho method `actingAs` cũng sẽ trở thành default guard trong suốt thời gian của test:

```php
$this->actingAs($user, 'web');
```

Nếu bạn muốn đảm bảo request là unauthenticated, bạn có thể sử dụng method `actingAsGuest`:

```php
$this->actingAsGuest();
```

<a name="debugging-responses"></a>
### Debugging Responses

Sau khi tạo một test request đến ứng dụng của bạn, các method `dump`, `dumpHeaders`, và `dumpSession` có thể được sử dụng để kiểm tra và debug response contents:

```php tab=Pest
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->dump();
    $response->dumpHeaders();
    $response->dumpSession();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->dump();
        $response->dumpHeaders();
        $response->dumpSession();
    }
}
```

Ngoài ra, bạn có thể sử dụng các method `dd`, `ddHeaders`, `ddBody`, `ddJson`, và `ddSession` để dump thông tin về response và sau đó dừng thực thi:

```php tab=Pest
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->dd();
    $response->ddHeaders();
    $response->ddBody();
    $response->ddJson();
    $response->ddSession();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->dd();
        $response->ddHeaders();
        $response->ddBody();
        $response->ddJson();
        $response->ddSession();
    }
}
```

<a name="exception-handling"></a>
### Exception Handling

Thỉnh thoảng bạn có thể cần test rằng ứng dụng của bạn đang ném một exception cụ thể. Để thực hiện điều này, bạn có thể "fake" exception handler thông qua facade `Exceptions`. Sau khi exception handler đã được fake, bạn có thể sử dụng các method `assertReported` và `assertNotReported` để tạo assertion đối với các exception được ném trong request:

```php tab=Pest
<?php

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;

test('exception is thrown', function () {
    Exceptions::fake();

    $response = $this->get('/order/1');

    // Assert an exception was thrown...
    Exceptions::assertReported(InvalidOrderException::class);

    // Assert against the exception...
    Exceptions::assertReported(function (InvalidOrderException $e) {
        return $e->getMessage() === 'The order was invalid.';
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_exception_is_thrown(): void
    {
        Exceptions::fake();

        $response = $this->get('/');

        // Assert an exception was thrown...
        Exceptions::assertReported(InvalidOrderException::class);

        // Assert against the exception...
        Exceptions::assertReported(function (InvalidOrderException $e) {
            return $e->getMessage() === 'The order was invalid.';
        });
    }
}
```

Các method `assertNotReported` và `assertNothingReported` có thể được sử dụng để xác nhận rằng một exception đã cho không được ném trong request hoặc rằng không có exception nào được ném:

```php
Exceptions::assertNotReported(InvalidOrderException::class);

Exceptions::assertNothingReported();
```

Bạn có thể hoàn toàn tắt exception handling cho một request đã cho bằng cách gọi method `withoutExceptionHandling` trước khi tạo request của bạn:

```php
$response = $this->withoutExceptionHandling()->get('/');
```

Ngoài ra, nếu bạn muốn đảm bảo rằng ứng dụng của bạn không sử dụng các tính năng đã bị deprecated bởi ngôn ngữ PHP hoặc các thư viện mà ứng dụng của bạn đang sử dụng, bạn có thể gọi method `withoutDeprecationHandling` trước khi tạo request của bạn. Khi deprecation handling bị tắt, deprecation warnings sẽ được chuyển đổi thành exceptions, do đó khiến test của bạn thất bại:

```php
$response = $this->withoutDeprecationHandling()->get('/');
```

Method `assertThrows` có thể được sử dụng để xác nhận rằng code trong một closure đã cho ném một exception của loại đã chỉ định:

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    OrderInvalid::class
);
```

Nếu bạn muốn kiểm tra và tạo assertion đối với exception được ném, bạn có thể cung cấp một closure như argument thứ hai cho method `assertThrows`:

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    fn (OrderInvalid $e) => $e->orderId() === 123;
);
```

Method `assertDoesntThrow` có thể được sử dụng để xác nhận rằng code trong một closure đã cho không ném bất kỳ exception nào:

```php
$this->assertDoesntThrow(fn () => (new ProcessOrder)->execute());
```

<a name="testing-json-apis"></a>
## Testing JSON APIs

Laravel cũng cung cấp một số helper để test JSON APIs và các response của chúng. Ví dụ, các method `json`, `getJson`, `postJson`, `putJson`, `patchJson`, `deleteJson`, và `optionsJson` có thể được sử dụng để phát ra JSON request với các HTTP verbs khác nhau. Bạn cũng có thể dễ dàng truyền data và headers cho các method này. Để bắt đầu, hãy viết một test để tạo một `POST` request đến `/api/user` và xác nhận rằng JSON data đã mong đợi được trả về:

```php tab=Pest
<?php

test('making an api request', function () {
    $response = $this->postJson('/api/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJson([
            'created' => true,
        ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_making_an_api_request(): void
    {
        $response = $this->postJson('/api/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```

Ngoài ra, JSON response data có thể được truy cập như array variables trên response, giúp bạn thuận tiện kiểm tra các giá trị riêng lẻ được trả về trong một JSON response:

```php tab=Pest
expect($response['created'])->toBeTrue();
```

```php tab=PHPUnit
$this->assertTrue($response['created']);
```

> [!NOTE]
> Method `assertJson` chuyển đổi response thành một array để xác minh rằng array đã cho tồn tại trong JSON response được trả về bởi ứng dụng. Vì vậy, nếu có các properties khác trong JSON response, test này vẫn sẽ pass miễn là fragment đã cho có mặt.

<a name="verifying-exact-match"></a>
#### Asserting Exact JSON Matches

Như đã đề cập trước đó, method `assertJson` có thể được sử dụng để xác nhận rằng một fragment của JSON tồn tại trong JSON response. Nếu bạn muốn xác minh rằng một array đã cho **khớp chính xác** với JSON được trả về bởi ứng dụng của bạn, bạn nên sử dụng method `assertExactJson`:

```php tab=Pest
<?php

test('asserting an exact json match', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertExactJson([
            'created' => true,
        ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_asserting_an_exact_json_match(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertExactJson([
                'created' => true,
            ]);
    }
}
```

<a name="verifying-json-paths"></a>
#### Asserting on JSON Paths

Nếu bạn muốn xác minh rằng JSON response chứa data đã cho tại một path đã chỉ định, bạn nên sử dụng method `assertJsonPath`:

```php tab=Pest
<?php

test('asserting a json path value', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJsonPath('team.owner.name', 'Darian');
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_asserting_a_json_paths_value(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJsonPath('team.owner.name', 'Darian');
    }
}
```

Method `assertJsonPath` cũng chấp nhận một closure, có thể được sử dụng để xác định động xem assertion có nên pass hay không:

```php
$response->assertJsonPath('team.owner.name', fn (string $name) => strlen($name) >= 3);
```

Nếu bạn cần xác nhận nhiều JSON paths cùng một lúc, bạn có thể sử dụng method `assertJsonPaths`. Giá trị mong đợi cho mỗi path cũng có thể là một closure:

```php
$response->assertJsonPaths([
    'team.owner.name' => 'Darian',
    'team.owner.email' => fn (string $email) => str($email)->is('*@laravel.com'),
    'team.members.0.name' => 'Sally',
]);
```

Bạn có thể sử dụng method `assertJsonMissingPaths` để xác nhận rằng nhiều JSON paths bị thiếu từ response:

```php
$response->assertJsonMissingPaths([
    'team.owner.password',
    'team.members.0.api_token',
]);
```

<a name="fluent-json-testing"></a>
### Fluent JSON Testing

Laravel cũng cung cấp một cách đẹp đẽ để test fluently các JSON response của ứng dụng. Để bắt đầu, truyền một closure cho method `assertJson`. Closure này sẽ được gọi với một instance của `Illuminate\Testing\Fluent\AssertableJson` có thể được sử dụng để tạo assertion đối với JSON được trả về bởi ứng dụng của bạn. Method `where` có thể được sử dụng để tạo assertion đối với một attribute cụ thể của JSON, trong khi method `missing` có thể được sử dụng để xác nhận rằng một attribute cụ thể bị thiếu từ JSON:

```php tab=Pest
use Illuminate\Testing\Fluent\AssertableJson;

test('fluent json', function () {
    $response = $this->getJson('/users/1');

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->where('id', 1)
                ->where('name', 'Victoria Faith')
                ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                ->whereNot('status', 'pending')
                ->missing('password')
                ->etc()
        );
});
```

```php tab=PHPUnit
use Illuminate\Testing\Fluent\AssertableJson;

/**
 * A basic functional test example.
 */
public function test_fluent_json(): void
{
    $response = $this->getJson('/users/1');

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->where('id', 1)
                ->where('name', 'Victoria Faith')
                ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                ->whereNot('status', 'pending')
                ->missing('password')
                ->etc()
        );
}
```

#### Understanding the `etc` Method

Trong ví dụ trên, bạn có thể đã nhận thấy chúng ta gọi method `etc` ở cuối assertion chain của chúng ta. Method này thông báo cho Laravel rằng có thể có các attributes khác có mặt trên JSON object. Nếu method `etc` không được sử dụng, test sẽ thất bại nếu có các attributes khác mà bạn không tạo assertion đối với tồn tại trên JSON object.

Ý định đằng sau hành vi này là bảo vệ bạn khỏi vô tình tiết lộ thông tin nhạy cảm trong các JSON response của bạn bằng cách buộc bạn phải hoặc là tạo assertion rõ ràng đối với attribute hoặc cho phép các attributes bổ sung thông qua method `etc`.

Tuy nhiên, bạn nên lưu ý rằng không bao gồm method `etc` trong assertion chain của bạn không đảm bảo rằng các attributes bổ sung không được thêm vào các arrays được lồng nhau trong JSON object của bạn. Method `etc` chỉ đảm bảo rằng không có attributes bổ sung nào tồn tại ở nesting level mà method `etc` được gọi.

<a name="asserting-json-attribute-presence-and-absence"></a>
#### Asserting Attribute Presence / Absence

Để xác nhận rằng một attribute có mặt hoặc vắng mặt, bạn có thể sử dụng các method `has` và `missing`:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('data')
        ->missing('message')
);
```

Ngoài ra, các method `hasAll` và `missingAll` cho phép xác nhận sự hiện diện hoặc vắng mặt của nhiều attributes cùng một lúc:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->hasAll(['status', 'data'])
        ->missingAll(['message', 'code'])
);
```

Bạn có thể sử dụng method `hasAny` để xác định xem ít nhất một trong danh sách attributes đã cho có mặt hay không:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('status')
        ->hasAny('data', 'message', 'code')
);
```

<a name="asserting-against-json-collections"></a>
#### Asserting Against JSON Collections

Thường thì, route của bạn sẽ trả về một JSON response chứa nhiều items, chẳng hạn như nhiều users:

```php
Route::get('/users', function () {
    return User::all();
});
```

Trong những tình huống này, chúng ta có thể sử dụng method `has` của fluent JSON object để tạo assertion đối với các users được bao gồm trong response. Ví dụ, hãy xác nhận rằng JSON response chứa ba users. Tiếp theo, chúng ta sẽ tạo một số assertion về user đầu tiên trong collection sử dụng method `first`. Method `first` chấp nhận một closure nhận một assertable JSON string khác mà chúng ta có thể sử dụng để tạo assertion về object đầu tiên trong JSON collection:

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has(3)
            ->first(fn (AssertableJson $json) =>
                $json->where('id', 1)
                    ->where('name', 'Victoria Faith')
                    ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                    ->missing('password')
                    ->etc()
            )
    );
```

Nếu bạn muốn tạo cùng assertion đối với mọi item trong một JSON collection, bạn có thể sử dụng method `each`:

```php
$response
  ->assertJson(fn (AssertableJson $json) =>
      $json->has(3)
          ->each(fn (AssertableJson $json) =>
              $json->whereType('id', 'integer')
                  ->whereType('name', 'string')
                  ->whereType('email', 'string')
                  ->missing('password')
                  ->etc()
          )
  );
```

<a name="scoping-json-collection-assertions"></a>
#### Scoping JSON Collection Assertions

Thỉnh thoảng, các routes của ứng dụng sẽ trả về JSON collections được gán named keys:

```php
Route::get('/users', function () {
    return [
        'meta' => [...],
        'users' => User::all(),
    ];
})
```

Khi test các routes này, bạn có thể sử dụng method `has` để xác nhận số lượng items trong collection. Ngoài ra, bạn có thể sử dụng method `has` để scope một chain của assertions:

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has('meta')
            ->has('users', 3)
            ->has('users.0', fn (AssertableJson $json) =>
                $json->where('id', 1)
                    ->where('name', 'Victoria Faith')
                    ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                    ->missing('password')
                    ->etc()
            )
    );
```

Tuy nhiên, thay vì tạo hai cuộc gọi riêng biệt đến method `has` để xác nhận đối với `users` collection, bạn có thể tạo một cuộc gọi duy nhất cung cấp một closure như parameter thứ ba. Khi làm như vậy, closure sẽ tự động được gọi và scoped đến item đầu tiên trong collection:

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has('meta')
            ->has('users', 3, fn (AssertableJson $json) =>
                $json->where('id', 1)
                    ->where('name', 'Victoria Faith')
                    ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                    ->missing('password')
                    ->etc()
            )
    );
```

<a name="asserting-json-types"></a>
#### Asserting JSON Types

Bạn có thể chỉ muốn xác nhận rằng các properties trong JSON response là của một loại nhất định. Class `Illuminate\Testing\Fluent\AssertableJson` cung cấp các method `whereType` và `whereAllType` để làm điều đó:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('id', 'integer')
        ->whereAllType([
            'users.0.name' => 'string',
            'meta' => 'array'
        ])
);
```

Bạn có thể chỉ định nhiều loại sử dụng ký tự `|`, hoặc truyền một array của types như parameter thứ hai cho method `whereType`. Assertion sẽ thành công nếu response value là bất kỳ loại nào được liệt kê:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('name', 'string|null')
        ->whereType('id', ['string', 'integer'])
);
```

Các method `whereType` và `whereAllType` nhận ra các loại sau: `string`, `integer`, `double`, `boolean`, `array`, và `null`.

<a name="testing-file-uploads"></a>
## Testing File Uploads

Class `Illuminate\Http\UploadedFile` cung cấp một method `fake` có thể được sử dụng để tạo dummy files hoặc images cho testing. Điều này, kết hợp với method `fake` của facade `Storage`, đơn giản hóa rất nhiều việc test file uploads. Ví dụ, bạn có thể kết hợp hai tính năng này để dễ dàng test một avatar upload form:

```php tab=Pest
<?php

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('avatars can be uploaded', function () {
    Storage::fake('avatars');

    $file = UploadedFile::fake()->image('avatar.jpg');

    $response = $this->post('/avatar', [
        'avatar' => $file,
    ]);

    Storage::disk('avatars')->assertExists($file->hashName());
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_avatars_can_be_uploaded(): void
    {
        Storage::fake('avatars');

        $file = UploadedFile::fake()->image('avatar.jpg');

        $response = $this->post('/avatar', [
            'avatar' => $file,
        ]);

        Storage::disk('avatars')->assertExists($file->hashName());
    }
}
```

Nếu bạn muốn xác nhận rằng một file đã cho không tồn tại, bạn có thể sử dụng method `assertMissing` được cung cấp bởi facade `Storage`:

```php
Storage::fake('avatars');

// ...

Storage::disk('avatars')->assertMissing('missing.jpg');
```

<a name="fake-file-customization"></a>
#### Fake File Customization

Khi tạo files sử dụng method `fake` được cung cấp bởi class `UploadedFile`, bạn có thể chỉ định width, height, và size của image (tính bằng kilobytes) để test tốt hơn các validation rules của ứng dụng:

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
```

Ngoài việc tạo images, bạn có thể tạo files của bất kỳ loại nào khác sử dụng method `create`:

```php
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
```

Nếu cần, bạn có thể truyền một argument `$mimeType` cho method để định nghĩa rõ ràng MIME type nên được trả về bởi file:

```php
UploadedFile::fake()->create(
    'document.pdf', $sizeInKilobytes, 'application/pdf'
);
```

<a name="testing-views"></a>
## Testing Views

Laravel cũng cho phép bạn render một view mà không tạo một simulated HTTP request đến ứng dụng. Để thực hiện điều này, bạn có thể gọi method `view` trong test của bạn. Method `view` chấp nhận view name và một optional array của data. Method trả về một instance của `Illuminate\Testing\TestView`, cung cấp một số method để thuận tiện tạo assertion về contents của view:

```php tab=Pest
<?php

test('a welcome view can be rendered', function () {
    $view = $this->view('welcome', ['name' => 'Taylor']);

    $view->assertSee('Taylor');
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_a_welcome_view_can_be_rendered(): void
    {
        $view = $this->view('welcome', ['name' => 'Taylor']);

        $view->assertSee('Taylor');
    }
}
```

Class `TestView` cung cấp các assertion methods sau: `assertSee`, `assertSeeInOrder`, `assertSeeText`, `assertSeeTextInOrder`, `assertDontSee`, và `assertDontSeeText`.

Nếu cần, bạn có thể lấy raw, rendered view contents bằng cách cast instance `TestView` thành string:

```php
$contents = (string) $this->view('welcome');
```

<a name="sharing-errors"></a>
#### Sharing Errors

Một số views có thể phụ thuộc vào errors được chia sẻ trong [global error bag được cung cấp bởi Laravel](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Để hydrate error bag với error messages, bạn có thể sử dụng method `withViewErrors`:

```php
$view = $this->withViewErrors([
    'name' => ['Please provide a valid name.']
])->view('form');

$view->assertSee('Please provide a valid name.');
```

<a name="rendering-blade-and-components"></a>
### Rendering Blade and Components

Nếu cần, bạn có thể sử dụng method `blade` để evaluate và render một raw [Blade](/docs/{{version}}/blade) string. Giống như method `view`, method `blade` trả về một instance của `Illuminate\Testing\TestView`:

```php
$view = $this->blade(
    '<x-component :name="$name" />',
    ['name' => 'Taylor']
);

$view->assertSee('Taylor');
```

Bạn có thể sử dụng method `component` để evaluate và render một [Blade component](/docs/{{version}}/blade#components). Method `component` trả về một instance của `Illuminate\Testing\TestComponent`:

```php
$view = $this->component(Profile::class, ['name' => 'Taylor']);

$view->assertSee('Taylor');
```

<a name="caching-routes"></a>
## Caching Routes

Trước khi một test chạy, Laravel khởi động một fresh instance của ứng dụng, bao gồm việc thu thập tất cả các routes được định nghĩa. Nếu ứng dụng của bạn có nhiều route files, bạn có thể muốn thêm trait `Illuminate\Foundation\Testing\WithCachedRoutes` vào các test cases của bạn. Trên các tests sử dụng trait này, routes được xây dựng một lần và lưu trữ trong memory, có nghĩa là quá trình thu thập routes chỉ chạy một lần cho tất cả tests trong suite của bạn:

```php tab=Pest
<?php

use App\Http\Controllers\UserController;
use Illuminate\Foundation\Testing\WithCachedRoutes;

pest()->use(WithCachedRoutes::class);

test('basic example', function () {
    $this->get(action([UserController::class, 'index']));

    // ...
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Http\Controllers\UserController;
use Illuminate\Foundation\Testing\WithCachedRoutes;
use Tests\TestCase;

class BasicTest extends TestCase
{
    use WithCachedRoutes;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        $response = $this->get(action([UserController::class, 'index']));

        // ...
    }
}
```

<a name="available-assertions"></a>
## Available Assertions

<a name="response-assertions"></a>
### Response Assertions

Class `Illuminate\Testing\TestResponse` của Laravel cung cấp nhiều custom assertion methods mà bạn có thể sử dụng khi test ứng dụng của bạn. Các assertions này có thể được truy cập trên response được trả về bởi các test methods `json`, `get`, `post`, `put`, và `delete`:

<style>
    .collection-method-list > p {
        columns: 14.4em 2; -moz-columns: 14.4em 2; -webkit-columns: 14.4em 2;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertAccepted](#assert-accepted)
[assertBadRequest](#assert-bad-request)
[assertClientError](#assert-client-error)
[assertConflict](#assert-conflict)
[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieNotExpired](#assert-cookie-not-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertCreated](#assert-created)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertDownload](#assert-download)
[assertExactJson](#assert-exact-json)
[assertExactJsonStructure](#assert-exact-json-structure)
[assertFailedDependency](#assert-failed-dependency)
[assertForbidden](#assert-forbidden)
[assertFound](#assert-found)
[assertGone](#assert-gone)
[assertHeader](#assert-header)
[assertHeaderContains](#assert-header-contains)
[assertHeaderMissing](#assert-header-missing)
[assertInternalServerError](#assert-internal-server-error)
[assertJson](#assert-json)
[assertJsonCount](#assert-json-count)
[assertJsonFragment](#assert-json-fragment)
[assertJsonIsArray](#assert-json-is-array)
[assertJsonIsObject](#assert-json-is-object)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
[assertJsonPath](#assert-json-path)
[assertJsonPaths](#assert-json-paths)
[assertJsonMissingPath](#assert-json-missing-path)
[assertJsonMissingPaths](#assert-json-missing-paths)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertJsonValidationErrorFor](#assert-json-validation-error-for)
[assertLocation](#assert-location)
[assertMethodNotAllowed](#assert-method-not-allowed)
[assertMovedPermanently](#assert-moved-permanently)
[assertContent](#assert-content)
[assertNoContent](#assert-no-content)
[assertStreamed](#assert-streamed)
[assertStreamedContent](#assert-streamed-content)
[assertNotFound](#assert-not-found)
[assertOk](#assert-ok)
[assertPaymentRequired](#assert-payment-required)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertRedirectBack](#assert-redirect-back)
[assertRedirectBackWithErrors](#assert-redirect-back-with-errors)
[assertRedirectBackWithoutErrors](#assert-redirect-back-without-errors)
[assertRedirectContains](#assert-redirect-contains)
[assertRedirectToRoute](#assert-redirect-to-route)
[assertRedirectToSignedRoute](#assert-redirect-to-signed-route)
[assertRequestTimeout](#assert-request-timeout)
[assertSee](#assert-see)
[assertSeeInOrder](#assert-see-in-order)
[assertSeeText](#assert-see-text)
[assertSeeTextInOrder](#assert-see-text-in-order)
[assertServerError](#assert-server-error)
[assertServiceUnavailable](#assert-service-unavailable)
[assertSessionHas](#assert-session-has)
[assertSessionHasInput](#assert-session-has-input)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionHasNoErrors](#assert-session-has-no-errors)
[assertSessionDoesntHaveErrors](#assert-session-doesnt-have-errors)
[assertSessionMissing](#assert-session-missing)
[assertSessionMissingInput](#assert-session-missing-input)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
[assertTooManyRequests](#assert-too-many-requests)
[assertUnauthorized](#assert-unauthorized)
[assertUnprocessable](#assert-unprocessable)
[assertUnsupportedMediaType](#assert-unsupported-media-type)
[assertValid](#assert-valid)
[assertInvalid](#assert-invalid)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)

</div>

<a name="assert-accepted"></a>
#### assertAccepted

Xác nhận rằng response có một accepted (202) HTTP status code:

```php
$response->assertAccepted();
```

<a name="assert-bad-request"></a>
#### assertBadRequest

Xác nhận rằng response có một bad request (400) HTTP status code:

```php
$response->assertBadRequest();
```

<a name="assert-client-error"></a>
#### assertClientError

Xác nhận rằng response có một client error (>= 400, < 500) HTTP status code:

```php
$response->assertClientError();
```

<a name="assert-conflict"></a>
#### assertConflict

Xác nhận rằng response có một conflict (409) HTTP status code:

```php
$response->assertConflict();
```

<a name="assert-cookie"></a>
#### assertCookie

Xác nhận rằng response chứa cookie đã cho:

```php
$response->assertCookie($cookieName, $value = null);
```

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

Xác nhận rằng response chứa cookie đã cho và nó đã hết hạn:

```php
$response->assertCookieExpired($cookieName);
```

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

Xác nhận rằng response chứa cookie đã cho và nó chưa hết hạn:

```php
$response->assertCookieNotExpired($cookieName);
```

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

Xác nhận rằng response không chứa cookie đã cho:

```php
$response->assertCookieMissing($cookieName);
```

<a name="assert-created"></a>
#### assertCreated

Xác nhận rằng response có một 201 HTTP status code:

```php
$response->assertCreated();
```

<a name="assert-dont-see"></a>
#### assertDontSee

Xác nhận rằng string đã cho không được chứa trong response được trả về bởi ứng dụng. Assertion này sẽ tự động escape string đã cho trừ khi bạn truyền argument thứ hai là `false`:

```php
$response->assertDontSee($value, $escape = true);
```

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

Xác nhận rằng string đã cho không được chứa trong response text. Assertion này sẽ tự động escape string đã cho trừ khi bạn truyền argument thứ hai là `false`. Method này sẽ truyền response content cho PHP function `strip_tags` trước khi tạo assertion:

```php
$response->assertDontSeeText($value, $escape = true);
```

<a name="assert-download"></a>
#### assertDownload

Xác nhận rằng response là một "download". Thông thường, điều này có nghĩa là route được gọi trả về response đã trả về một `Response::download` response, `BinaryFileResponse`, hoặc `Storage::download` response:

```php
$response->assertDownload();
```

Nếu bạn muốn, bạn có thể xác nhận rằng downloadable file được gán một file name đã cho:

```php
$response->assertDownload('image.jpg');
```

<a name="assert-exact-json"></a>
#### assertExactJson

Xác nhận rằng response chứa một exact match của JSON data đã cho:

```php
$response->assertExactJson(array $data);
```

<a name="assert-exact-json-structure"></a>
#### assertExactJsonStructure

Xác nhận rằng response chứa một exact match của JSON structure đã cho:

```php
$response->assertExactJsonStructure(array $data);
```

Method này là một biến thể chặt chẽ hơn của [assertJsonStructure](#assert-json-structure). Ngược lại với `assertJsonStructure`, method này sẽ thất bại nếu response chứa bất kỳ keys nào không được bao gồm rõ ràng trong expected JSON structure.

<a name="assert-failed-dependency"></a>
#### assertFailedDependency

Xác nhận rằng response có một failed dependency (424) HTTP status code:

```php
$response->assertFailedDependency();
```

<a name="assert-forbidden"></a>
#### assertForbidden

Xác nhận rằng response có một forbidden (403) HTTP status code:

```php
$response->assertForbidden();
```

<a name="assert-found"></a>
#### assertFound

Xác nhận rằng response có một found (302) HTTP status code:

```php
$response->assertFound();
```

<a name="assert-gone"></a>
#### assertGone

Xác nhận rằng response có một gone (410) HTTP status code:

```php
$response->assertGone();
```

<a name="assert-header"></a>
#### assertHeader

Xác nhận rằng header và value đã cho có mặt trên response:

```php
$response->assertHeader($headerName, $value = null);
```

<a name="assert-header-contains"></a>
#### assertHeaderContains

Xác nhận rằng header đã cho chứa một substring value đã cho:

```php
$response->assertHeaderContains($headerName, $value);
```

<a name="assert-header-missing"></a>
#### assertHeaderMissing

Xác nhận rằng header đã cho không có mặt trên response:

```php
$response->assertHeaderMissing($headerName);
```

<a name="assert-internal-server-error"></a>
#### assertInternalServerError

Xác nhận rằng response có một "Internal Server Error" (500) HTTP status code:

```php
$response->assertInternalServerError();
```

<a name="assert-json"></a>
#### assertJson

Xác nhận rằng response chứa JSON data đã cho:

```php
$response->assertJson(array $data, $strict = false);
```

Method `assertJson` chuyển đổi response thành một array để xác minh rằng array đã cho tồn tại trong JSON response được trả về bởi ứng dụng. Vì vậy, nếu có các properties khác trong JSON response, test này vẫn sẽ pass miễn là fragment đã cho có mặt.

<a name="assert-json-count"></a>
#### assertJsonCount

Xác nhận rằng response JSON có một array với số lượng items mong đợi tại key đã cho:

```php
$response->assertJsonCount($count, $key = null);
```

<a name="assert-json-fragment"></a>
#### assertJsonFragment

Xác nhận rằng response chứa JSON data đã cho ở bất kỳ đâu trong response:

```php
Route::get('/users', function () {
    return [
        'users' => [
            [
                'name' => 'Taylor Otwell',
            ],
        ],
    ];
});

$response->assertJsonFragment(['name' => 'Taylor Otwell']);
```

<a name="assert-json-is-array"></a>
#### assertJsonIsArray

Xác nhận rằng response JSON là một array:

```php
$response->assertJsonIsArray();
```

<a name="assert-json-is-object"></a>
#### assertJsonIsObject

Xác nhận rằng response JSON là một object:

```php
$response->assertJsonIsObject();
```

<a name="assert-json-missing"></a>
#### assertJsonMissing

Xác nhận rằng response không chứa JSON data đã cho:

```php
$response->assertJsonMissing(array $data);
```

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

Xác nhận rằng response không chứa exact JSON data:

```php
$response->assertJsonMissingExact(array $data);
```

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

Xác nhận rằng response không có JSON validation errors cho các keys đã cho:

```php
$response->assertJsonMissingValidationErrors($keys);
```

> [!NOTE]
> Method [assertValid](#assert-valid) chung chung hơn có thể được sử dụng để xác nhận rằng response không có validation errors được trả về như JSON **và** rằng không có errors được flashed đến session storage.

<a name="assert-json-path"></a>
#### assertJsonPath

Xác nhận rằng response chứa data đã cho tại path đã chỉ định:

```php
$response->assertJsonPath($path, $expectedValue);
```

Ví dụ, nếu JSON response sau được trả về bởi ứng dụng của bạn:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Bạn có thể xác nhận rằng property `name` của object `user` khớp với một value đã cho như sau:

```php
$response->assertJsonPath('user.name', 'Steve Schoger');
```

<a name="assert-json-paths"></a>
#### assertJsonPaths

Xác nhận rằng response chứa data đã cho tại các paths đã chỉ định:

```php
$response->assertJsonPaths(array $paths);
```

Ví dụ, bạn có thể xác nhận nhiều values trong response cùng một lúc:

```php
$response->assertJsonPaths([
    'user.name' => 'Steve Schoger',
    'user.email' => fn (string $email) => str($email)->endsWith('@laravel.com'),
]);
```

<a name="assert-json-missing-path"></a>
#### assertJsonMissingPath

Xác nhận rằng response không chứa path đã cho:

```php
$response->assertJsonMissingPath($path);
```

Ví dụ, nếu JSON response sau được trả về bởi ứng dụng của bạn:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Bạn có thể xác nhận rằng nó không chứa property `email` của object `user`:

```php
$response->assertJsonMissingPath('user.email');
```

<a name="assert-json-missing-paths"></a>
#### assertJsonMissingPaths

Xác nhận rằng response không chứa các paths đã cho:

```php
$response->assertJsonMissingPaths($paths);
```

Ví dụ, bạn có thể xác nhận rằng nhiều paths bị thiếu từ response:

```php
$response->assertJsonMissingPaths([
    'user.email',
    'user.password',
]);
```

<a name="assert-json-structure"></a>
#### assertJsonStructure

Xác nhận rằng response có một JSON structure đã cho:

```php
$response->assertJsonStructure(array $structure);
```

Ví dụ, nếu JSON response được trả về bởi ứng dụng của bạn chứa data sau:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

Bạn có thể xác nhận rằng JSON structure khớp với mong đợi của bạn như sau:

```php
$response->assertJsonStructure([
    'user' => [
        'name',
    ]
]);
```

Thỉnh thoảng, JSON responses được trả về bởi ứng dụng của bạn có thể chứa arrays của objects:

```json
{
    "user": [
        {
            "name": "Steve Schoger",
            "age": 55,
            "location": "Earth"
        },
        {
            "name": "Mary Schoger",
            "age": 60,
            "location": "Earth"
        }
    ]
}
```

Trong tình huống này, bạn có thể sử dụng ký tự `*` để xác nhận đối với structure của tất cả các objects trong array:

```php
$response->assertJsonStructure([
    'user' => [
        '*' => [
             'name',
             'age',
             'location'
        ]
    ]
]);
```

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

Xác nhận rằng response có các JSON validation errors đã cho cho các keys đã cho. Method này nên được sử dụng khi tạo assertion đối với responses nơi validation errors được trả về như một JSON structure thay vì được flashed đến session:

```php
$response->assertJsonValidationErrors(array $data, $responseKey = 'errors');
```

> [!NOTE]
> Method [assertInvalid](#assert-invalid) chung chung hơn có thể được sử dụng để xác nhận rằng response có validation errors được trả về như JSON **hoặc** rằng errors được flashed đến session storage.

<a name="assert-json-validation-error-for"></a>
#### assertJsonValidationErrorFor

Xác nhận response có bất kỳ JSON validation errors nào cho key đã cho:

```php
$response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');
```

<a name="assert-method-not-allowed"></a>
#### assertMethodNotAllowed

Xác nhận rằng response có một method not allowed (405) HTTP status code:

```php
$response->assertMethodNotAllowed();
```

<a name="assert-moved-permanently"></a>
#### assertMovedPermanently

Xác nhận rằng response có một moved permanently (301) HTTP status code:

```php
$response->assertMovedPermanently();
```

<a name="assert-location"></a>
#### assertLocation

Xác nhận rằng response có URI value đã cho trong header `Location`:

```php
$response->assertLocation($uri);
```

<a name="assert-content"></a>
#### assertContent

Xác nhận rằng string đã cho khớp với response content:

```php
$response->assertContent($value);
```

<a name="assert-no-content"></a>
#### assertNoContent

Xác nhận rằng response có HTTP status code đã cho và không có content:

```php
$response->assertNoContent($status = 204);
```

<a name="assert-streamed"></a>
#### assertStreamed

Xác nhận rằng response là một streamed response:

    $response->assertStreamed();

<a name="assert-streamed-content"></a>
#### assertStreamedContent

Xác nhận rằng string đã cho khớp với streamed response content:

```php
$response->assertStreamedContent($value);
```

<a name="assert-not-found"></a>
#### assertNotFound

Xác nhận rằng response có một not found (404) HTTP status code:

```php
$response->assertNotFound();
```

<a name="assert-ok"></a>
#### assertOk

Xác nhận rằng response có một 200 HTTP status code:

```php
$response->assertOk();
```

<a name="assert-payment-required"></a>
#### assertPaymentRequired

Xác nhận rằng response có một payment required (402) HTTP status code:

```php
$response->assertPaymentRequired();
```

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

Xác nhận rằng response chứa cookie unencrypted đã cho:

```php
$response->assertPlainCookie($cookieName, $value = null);
```

<a name="assert-redirect"></a>
#### assertRedirect

Xác nhận rằng response là một redirect đến URI đã cho:

```php
$response->assertRedirect($uri = null);
```

<a name="assert-redirect-back"></a>
#### assertRedirectBack

Xác nhận xem response có đang redirect back đến trang trước đó hay không:

```php
$response->assertRedirectBack();
```

<a name="assert-redirect-back-with-errors"></a>
#### assertRedirectBackWithErrors

Xác nhận xem response có đang redirect back đến trang trước đó và [session có các errors đã cho](#assert-session-has-errors) hay không:

```php
$response->assertRedirectBackWithErrors(
    array $keys = [], $format = null, $errorBag = 'default'
);
```

<a name="assert-redirect-back-without-errors"></a>
#### assertRedirectBackWithoutErrors

Xác nhận xem response có đang redirect back đến trang trước đó và session không chứa bất kỳ error messages nào hay không:

```php
$response->assertRedirectBackWithoutErrors();
```

<a name="assert-redirect-contains"></a>
#### assertRedirectContains

Xác nhận xem response có đang redirect đến một URI chứa string đã cho hay không:

```php
$response->assertRedirectContains($string);
```

<a name="assert-redirect-to-route"></a>
#### assertRedirectToRoute

Xác nhận rằng response là một redirect đến [named route](/docs/{{version}}/routing#named-routes) đã cho:

```php
$response->assertRedirectToRoute($name, $parameters = []);
```

<a name="assert-redirect-to-signed-route"></a>
#### assertRedirectToSignedRoute

Xác nhận rằng response là một redirect đến [signed route](/docs/{{version}}/urls#signed-urls) đã cho:

```php
$response->assertRedirectToSignedRoute($name = null, $parameters = []);
```

<a name="assert-request-timeout"></a>
#### assertRequestTimeout

Xác nhận rằng response có một request timeout (408) HTTP status code:

```php
$response->assertRequestTimeout();
```

<a name="assert-see"></a>
#### assertSee

Xác nhận rằng string đã cho được chứa trong response. Assertion này sẽ tự động escape string đã cho trừ khi bạn truyền argument thứ hai là `false`:

```php
$response->assertSee($value, $escape = true);
```

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

Xác nhận rằng các strings đã cho được chứa theo thứ tự trong response. Assertion này sẽ tự động escape các strings đã cho trừ khi bạn truyền argument thứ hai là `false`:

```php
$response->assertSeeInOrder(array $values, $escape = true);
```

<a name="assert-see-text"></a>
#### assertSeeText

Xác nhận rằng string đã cho được chứa trong response text. Assertion này sẽ tự động escape string đã cho trừ khi bạn truyền argument thứ hai là `false`. Response content sẽ được truyền cho PHP function `strip_tags` trước khi assertion được tạo:

```php
$response->assertSeeText($value, $escape = true);
```

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

Xác nhận rằng các strings đã cho được chứa theo thứ tự trong response text. Assertion này sẽ tự động escape các strings đã cho trừ khi bạn truyền argument thứ hai là `false`. Response content sẽ được truyền cho PHP function `strip_tags` trước khi assertion được tạo:

```php
$response->assertSeeTextInOrder(array $values, $escape = true);
```

<a name="assert-server-error"></a>
#### assertServerError

Xác nhận rằng response có một server error (>= 500 , < 600) HTTP status code:

```php
$response->assertServerError();
```

<a name="assert-service-unavailable"></a>
#### assertServiceUnavailable

Xác nhận rằng response có một "Service Unavailable" (503) HTTP status code:

```php
$response->assertServiceUnavailable();
```

<a name="assert-session-has"></a>
#### assertSessionHas

Xác nhận rằng session chứa piece of data đã cho:

```php
$response->assertSessionHas($key, $value = null);
```

Nếu cần, một closure có thể được cung cấp như argument thứ hai cho method `assertSessionHas`. Assertion sẽ pass nếu closure trả về `true`:

```php
$response->assertSessionHas($key, function (User $value) {
    return $value->name === 'Taylor Otwell';
});
```

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

Xác nhận rằng session có một value đã cho trong [flashed input array](/docs/{{version}}/responses#redirecting-with-flashed-session-data):

```php
$response->assertSessionHasInput($key, $value = null);
```

Nếu cần, một closure có thể được cung cấp như argument thứ hai cho method `assertSessionHasInput`. Assertion sẽ pass nếu closure trả về `true`:

```php
use Illuminate\Support\Facades\Crypt;

$response->assertSessionHasInput($key, function (string $value) {
    return Crypt::decryptString($value) === 'secret';
});
```

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

Xác nhận rằng session chứa một array của key / value pairs đã cho:

```php
$response->assertSessionHasAll(array $data);
```

Ví dụ, nếu session của ứng dụng chứa các keys `name` và `status`, bạn có thể xác nhận rằng cả hai đều tồn tại và có các values đã chỉ định như sau:

```php
$response->assertSessionHasAll([
    'name' => 'Taylor Otwell',
    'status' => 'active',
]);
```

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

Xác nhận rằng session chứa một error cho các `$keys` đã cho. Nếu `$keys` là một associative array, xác nhận rằng session chứa một error message cụ thể (value) cho mỗi field (key). Method này nên được sử dụng khi test các routes flash validation errors đến session thay vì trả về chúng như một JSON structure:

```php
$response->assertSessionHasErrors(
    array $keys = [], $format = null, $errorBag = 'default'
);
```

Ví dụ, để xác nhận rằng các fields `name` và `email` có validation error messages được flashed đến session, bạn có thể gọi method `assertSessionHasErrors` như sau:

```php
$response->assertSessionHasErrors(['name', 'email']);
```

Hoặc, bạn có thể xác nhận rằng một field đã cho có một validation error message cụ thể:

```php
$response->assertSessionHasErrors([
    'name' => 'The given name was invalid.'
]);
```

> [!NOTE]
> Method [assertInvalid](#assert-invalid) chung chung hơn có thể được sử dụng để xác nhận rằng response có validation errors được trả về như JSON **hoặc** rằng errors được flashed đến session storage.

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

Xác nhận rằng session chứa một error cho các `$keys` đã cho trong một [error bag](/docs/{{version}}/validation#named-error-bags) cụ thể. Nếu `$keys` là một associative array, xác nhận rằng session chứa một error message cụ thể (value) cho mỗi field (key), trong error bag:

```php
$response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);
```

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

Xác nhận rằng session không có validation errors:

```php
$response->assertSessionHasNoErrors();
```

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

Xác nhận rằng session không có validation errors cho các keys đã cho:

```php
$response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');
```

> [!NOTE]
> Method [assertValid](#assert-valid) chung chung hơn có thể được sử dụng để xác nhận rằng response không có validation errors được trả về như JSON **và** rằng không có errors được flashed đến session storage.

<a name="assert-session-missing"></a>
#### assertSessionMissing

Xác nhận rằng session không chứa key đã cho:

```php
$response->assertSessionMissing($key);
```

<a name="assert-session-missing-input"></a>
#### assertSessionMissingInput

Xác nhận rằng session thiếu input key đã cho trong flashed input array:

```php
$response->assertSessionMissingInput($key);
```

<a name="assert-status"></a>
#### assertStatus

Xác nhận rằng response có HTTP status code đã cho:

```php
$response->assertStatus($code);
```

<a name="assert-successful"></a>
#### assertSuccessful

Xác nhận rằng response có một successful (>= 200 và < 300) HTTP status code:

```php
$response->assertSuccessful();
```

<a name="assert-too-many-requests"></a>
#### assertTooManyRequests

Xác nhận rằng response có một too many requests (429) HTTP status code:

```php
$response->assertTooManyRequests();
```

<a name="assert-unauthorized"></a>
#### assertUnauthorized

Xác nhận rằng response có một unauthorized (401) HTTP status code:

```php
$response->assertUnauthorized();
```

<a name="assert-unprocessable"></a>
#### assertUnprocessable

Xác nhận rằng response có một unprocessable entity (422) HTTP status code:

```php
$response->assertUnprocessable();
```

<a name="assert-unsupported-media-type"></a>
#### assertUnsupportedMediaType

Xác nhận rằng response có một unsupported media type (415) HTTP status code:

```php
$response->assertUnsupportedMediaType();
```

<a name="assert-valid"></a>
#### assertValid

Xác nhận rằng response không có validation errors cho các keys đã cho. Method này có thể được sử dụng để tạo assertion đối với responses nơi validation errors được trả về như một JSON structure hoặc nơi validation errors đã được flashed đến session:

```php
// Assert that no validation errors are present...
$response->assertValid();

// Assert that the given keys do not have validation errors...
$response->assertValid(['name', 'email']);
```

<a name="assert-invalid"></a>
#### assertInvalid

Xác nhận rằng response có validation errors cho các keys đã cho. Method này có thể được sử dụng để tạo assertion đối với responses nơi validation errors được trả về như một JSON structure hoặc nơi validation errors đã được flashed đến session:

```php
$response->assertInvalid(['name', 'email']);
```

Bạn cũng có thể xác nhận rằng một key đã cho có một validation error message cụ thể. Khi làm như vậy, bạn có thể cung cấp toàn bộ message hoặc chỉ một phần nhỏ của message:

```php
$response->assertInvalid([
    'name' => 'The name field is required.',
    'email' => 'valid email address',
]);
```

Nếu bạn muốn xác nhận rằng các fields đã cho là các fields duy nhất có validation errors, bạn có thể sử dụng method `assertOnlyInvalid`:

```php
$response->assertOnlyInvalid(['name', 'email']);
```

<a name="assert-view-has"></a>
#### assertViewHas

Xác nhận rằng response view chứa một piece of data đã cho:

```php
$response->assertViewHas($key, $value = null);
```

Truyền một closure như argument thứ hai cho method `assertViewHas` sẽ cho phép bạn kiểm tra và tạo assertion đối với một piece cụ thể của view data:

```php
$response->assertViewHas('user', function (User $user) {
    return $user->name === 'Taylor';
});
```

Ngoài ra, view data có thể được truy cập như array variables trên response, cho phép bạn thuận tiện kiểm tra nó:

```php tab=Pest
expect($response['name'])->toBe('Taylor');
```

```php tab=PHPUnit
$this->assertEquals('Taylor', $response['name']);
```

<a name="assert-view-has-all"></a>
#### assertViewHasAll

Xác nhận rằng response view có một list của data đã cho:

```php
$response->assertViewHasAll(array $data);
```

Method này có thể được sử dụng để xác nhận rằng view đơn giản chứa data khớp với các keys đã cho:

```php
$response->assertViewHasAll([
    'name',
    'email',
]);
```

Hoặc, bạn có thể xác nhận rằng view data có mặt và có các values cụ thể:

```php
$response->assertViewHasAll([
    'name' => 'Taylor Otwell',
    'email' => 'taylor@example.com,',
]);
```

<a name="assert-view-is"></a>
#### assertViewIs

Xác nhận rằng view đã cho được trả về bởi route:

```php
$response->assertViewIs($value);
```

<a name="assert-view-missing"></a>
#### assertViewMissing

Xác nhận rằng data key đã cho không được cung cấp cho view được trả về trong response của ứng dụng:

```php
$response->assertViewMissing($key);
```

<a name="authentication-assertions"></a>
### Authentication Assertions

Laravel cũng cung cấp nhiều authentication related assertions mà bạn có thể sử dụng trong feature tests của ứng dụng. Lưu ý rằng các methods này được gọi trên test class chính nó chứ không phải instance `Illuminate\Testing\TestResponse` được trả về bởi các methods như `get` và `post`.

<a name="assert-authenticated"></a>
#### assertAuthenticated

Xác nhận rằng một user được authenticated:

```php
$this->assertAuthenticated($guard = null);
```

<a name="assert-guest"></a>
#### assertGuest

Xác nhận rằng một user không được authenticated:

```php
$this->assertGuest($guard = null);
```

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

Xác nhận rằng một user cụ thể được authenticated:

```php
$this->assertAuthenticatedAs($user, $guard = null);
```

<a name="validation-assertions"></a>
## Validation Assertions

Laravel cung cấp hai validation related assertions chính mà bạn có thể sử dụng để đảm bảo data được cung cấp trong request của bạn là valid hoặc invalid.

<a name="validation-assert-valid"></a>
#### assertValid

Xác nhận rằng response không có validation errors cho các keys đã cho. Method này có thể được sử dụng để tạo assertion đối với responses nơi validation errors được trả về như một JSON structure hoặc nơi validation errors đã được flashed đến session:

```php
// Assert that no validation errors are present...
$response->assertValid();

// Assert that the given keys do not have validation errors...
$response->assertValid(['name', 'email']);
```

<a name="validation-assert-invalid"></a>
#### assertInvalid

Xác nhận rằng response có validation errors cho các keys đã cho. Method này có thể được sử dụng để tạo assertion đối với responses nơi validation errors được trả về như một JSON structure hoặc nơi validation errors đã được flashed đến session:

```php
$response->assertInvalid(['name', 'email']);
```

Bạn cũng có thể xác nhận rằng một key đã cho có một validation error message cụ thể. Khi làm như vậy, bạn có thể cung cấp toàn bộ message hoặc chỉ một phần nhỏ của message:

```php
$response->assertInvalid([
    'name' => 'The name field is required.',
    'email' => 'valid email address',
]);
```

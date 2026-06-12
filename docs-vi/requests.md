# HTTP Requests

- [Giới thiệu](#introduction)
- [Tương tác với Request](#interacting-with-the-request)
    - [Truy cập Request](#accessing-the-request)
    - [Request Path, Host, và Method](#request-path-and-method)
    - [Request Headers](#request-headers)
    - [Request IP Address](#request-ip-address)
    - [Content Negotiation](#content-negotiation)
    - [PSR-7 Requests](#psr7-requests)
- [Input](#input)
    - [Lấy Input](#retrieving-input)
    - [Kiểm tra Input](#input-presence)
    - [Gộp Additional Input](#merging-additional-input)
    - [Old Input](#old-input)
    - [Cookies](#cookies)
    - [Input Trimming và Normalization](#input-trimming-and-normalization)
- [Files](#files)
    - [Lấy Uploaded Files](#retrieving-uploaded-files)
    - [Lưu Uploaded Files](#storing-uploaded-files)
- [Cấu hình Trusted Proxies](#configuring-trusted-proxies)
- [Cấu hình Trusted Hosts](#configuring-trusted-hosts)

<a name="introduction"></a>
## Giới thiệu

Class `Illuminate\Http\Request` của Laravel cung cấp một cách hướng đối tượng để tương tác với HTTP request hiện tại đang được xử lý bởi ứng dụng của bạn, cũng như lấy input, cookies, và files được gửi kèm với request.

<a name="interacting-with-the-request"></a>
## Tương tác với Request

<a name="accessing-the-request"></a>
### Truy cập Request

Để lấy một instance của HTTP request hiện tại thông qua dependency injection, bạn nên type-hint class `Illuminate\Http\Request` trên route closure hoặc controller method của bạn. Instance request đến sẽ được tự động inject bởi Laravel [service container](/docs/{{version}}/container):

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Store the user...

        return redirect('/users');
    }
}
```

Như đã đề cập, bạn cũng có thể type-hint class `Illuminate\Http\Request` trên route closure. Service container sẽ tự động inject request đến vào closure khi nó được thực thi:

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

<a name="dependency-injection-route-parameters"></a>
#### Dependency Injection và Route Parameters

Nếu controller method của bạn cũng đang mong đợi input từ một route parameter, bạn nên liệt kê các route parameters của bạn sau các dependencies khác. Ví dụ, nếu route của bạn được định nghĩa như sau:

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

Bạn vẫn có thể type-hint `Illuminate\Http\Request` và truy cập route parameter `id` của bạn bằng cách định nghĩa controller method như sau:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...

        return redirect('/users');
    }
}
```

<a name="request-path-and-method"></a>
### Request Path, Host, và Method

Instance `Illuminate\Http\Request` cung cấp nhiều phương thức để kiểm tra HTTP request đến và extends class `Symfony\Component\HttpFoundation\Request`. Chúng ta sẽ thảo luận về một số phương thức quan trọng nhất bên dưới.

<a name="retrieving-the-request-path"></a>
#### Lấy Request Path

Phương thức `path` trả về thông tin path của request. Vì vậy, nếu request đến nhắm đến `http://example.com/foo/bar`, phương thức `path` sẽ trả về `foo/bar`:

```php
$uri = $request->path();
```

<a name="inspecting-the-request-path"></a>
#### Kiểm tra Request Path / Route

Phương thức `is` cho phép bạn xác minh rằng request path đến khớp với một pattern nhất định. Bạn có thể sử dụng ký tự `*` như một wildcard khi sử dụng phương thức này:

```php
if ($request->is('admin/*')) {
    // ...
}
```

Sử dụng phương thức `routeIs`, bạn có thể xác định xem request đến có khớp với một [named route](/docs/{{version}}/routing#named-routes) hay không:

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

<a name="retrieving-the-request-url"></a>
#### Lấy Request URL

Để lấy URL đầy đủ cho request đến, bạn có thể sử dụng phương thức `url` hoặc `fullUrl`. Phương thức `url` sẽ trả về URL mà không có query string, trong khi phương thức `fullUrl` bao gồm query string:

```php
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```

Nếu bạn muốn thêm dữ liệu query string vào URL hiện tại, bạn có thể gọi phương thức `fullUrlWithQuery`. Phương thức này gộp mảng các biến query string đã cho với query string hiện tại:

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

Nếu bạn muốn lấy URL hiện tại mà không có một tham số query string nhất định, bạn có thể sử dụng phương thức `fullUrlWithoutQuery`:

```php
$request->fullUrlWithoutQuery(['type']);
```

<a name="retrieving-the-request-host"></a>
#### Lấy Request Host

Bạn có thể lấy "host" của request đến thông qua các phương thức `host`, `httpHost`, và `schemeAndHttpHost`:

```php
// http://localhost:8000
$request->host(); // localhost
$request->httpHost(); // localhost:8000
$request->schemeAndHttpHost(); // http://localhost:8000
```

<a name="retrieving-the-request-method"></a>
#### Lấy Request Method

Phương thức `method` sẽ trả về HTTP verb cho request. Bạn có thể sử dụng phương thức `isMethod` để xác minh rằng HTTP verb khớp với một chuỗi nhất định:

```php
$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}
```

<a name="request-headers"></a>
### Request Headers

Bạn có thể lấy một request header từ instance `Illuminate\Http\Request` sử dụng phương thức `header`. Nếu header không có trên request, `null` sẽ được trả về. Tuy nhiên, phương thức `header` chấp nhận một tham số thứ hai tùy chọn sẽ được trả về nếu header không có trên request:

```php
$value = $request->header('X-Header-Name');

$value = $request->header('X-Header-Name', 'default');
```

Phương thức `hasHeader` có thể được sử dụng để xác định xem request có chứa một header nhất định hay không:

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

Để thuận tiện, phương thức `bearerToken` có thể được sử dụng để lấy một bearer token từ header `Authorization`. Nếu không có header nào như vậy, một chuỗi rỗng sẽ được trả về:

```php
$token = $request->bearerToken();
```

<a name="request-ip-address"></a>
### Request IP Address

Phương thức `ip` có thể được sử dụng để lấy địa chỉ IP của client đã thực hiện request đến ứng dụng của bạn:

```php
$ipAddress = $request->ip();
```

Nếu bạn muốn lấy một mảng các địa chỉ IP, bao gồm tất cả các địa chỉ IP client đã được forward bởi proxies, bạn có thể sử dụng phương thức `ips`. Địa chỉ IP client "gốc" sẽ ở cuối mảng:

```php
$ipAddresses = $request->ips();
```

Nói chung, địa chỉ IP nên được coi là input không đáng tin cậy, do người dùng kiểm soát và chỉ nên được sử dụng cho mục đích thông tin.

<a name="content-negotiation"></a>
### Content Negotiation

Laravel cung cấp một số phương thức để kiểm tra các content types được yêu cầu của request đến thông qua header `Accept`. Đầu tiên, phương thức `getAcceptableContentTypes` sẽ trả về một mảng chứa tất cả các content types được chấp nhận bởi request:

```php
$contentTypes = $request->getAcceptableContentTypes();
```

Phương thức `accepts` chấp nhận một mảng các content types và trả về `true` nếu bất kỳ content types nào được chấp nhận bởi request. Nếu không, `false` sẽ được trả về:

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

Bạn có thể sử dụng phương thức `prefers` để xác định content type nào trong một mảng các content types nhất định được ưu tiên nhất bởi request. Nếu không có content types nào được cung cấp được chấp nhận bởi request, `null` sẽ được trả về:

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

Vì nhiều ứng dụng chỉ phục vụ HTML hoặc JSON, bạn có thể sử dụng phương thức `expectsJson` để nhanh chóng xác định xem request đến có mong đợi một JSON response hay không:

```php
if ($request->expectsJson()) {
    // ...
}
```

Nếu bạn cần xác định xem request có ưu tiên cụ thể Markdown hay sẽ chấp nhận Markdown trong số các content types khác, chẳng hạn như khi phục vụ AI agents hoặc các clients khác tiêu thụ Markdown responses, bạn có thể sử dụng các phương thức `wantsMarkdown` và `acceptsMarkdown`:

```php
if ($request->wantsMarkdown()) {
    // The client's most preferred content type is text/markdown...
}

if ($request->acceptsMarkdown()) {
    // The client accepts Markdown responses...
}
```

<a name="psr7-requests"></a>
### PSR-7 Requests

[PSR-7 standard](https://www.php-fig.org/psr/psr-7/) chỉ định các interfaces cho HTTP messages, bao gồm requests và responses. Nếu bạn muốn lấy một instance của PSR-7 request thay vì Laravel request, bạn sẽ cần cài đặt một vài thư viện trước. Laravel sử dụng component *Symfony HTTP Message Bridge* để chuyển đổi các Laravel requests và responses điển hình thành các implementations tương thích PSR-7:

```shell
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

Khi bạn đã cài đặt các thư viện này, bạn có thể lấy một PSR-7 request bằng cách type-hint request interface trên route closure hoặc controller method của bạn:

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```

> [!NOTE]
> Nếu bạn trả về một PSR-7 response instance từ một route hoặc controller, nó sẽ tự động được chuyển đổi trở lại thành một Laravel response instance và được hiển thị bởi framework.

<a name="input"></a>
## Input

<a name="retrieving-input"></a>
### Lấy Input

<a name="retrieving-all-input-data"></a>
#### Lấy Tất cả Input Data

Bạn có thể lấy tất cả input data của request đến dưới dạng một `array` sử dụng phương thức `all`. Phương thức này có thể được sử dụng bất kể request đến là từ một HTML form hay là một XHR request:

```php
$input = $request->all();
```

Sử dụng phương thức `collect`, bạn có thể lấy tất cả input data của request đến dưới dạng một [collection](/docs/{{version}}/collections):

```php
$input = $request->collect();
```

Phương thức `collect` cũng cho phép bạn lấy một subset của input của request đến dưới dạng một collection:

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

<a name="retrieving-an-input-value"></a>
#### Lấy Một Input Value

Sử dụng một vài phương thức đơn giản, bạn có thể truy cập tất cả user input từ instance `Illuminate\Http\Request` của bạn mà không cần lo lắng về HTTP verb nào được sử dụng cho request. Bất kể HTTP verb, phương thức `input` có thể được sử dụng để lấy user input:

```php
$name = $request->input('name');
```

Bạn có thể truyền một giá trị mặc định làm tham số thứ hai cho phương thức `input`. Giá trị này sẽ được trả về nếu input value được yêu cầu không có trên request:

```php
$name = $request->input('name', 'Sally');
```

Khi làm việc với các form chứa array inputs, sử dụng "dot" notation để truy cập các arrays:

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

Bạn có thể gọi phương thức `input` mà không có bất kỳ đối số nào để lấy tất cả các input values dưới dạng một associative array:

```php
$input = $request->input();
```

<a name="retrieving-input-from-the-query-string"></a>
#### Lấy Input Từ Query String

Trong khi phương thức `input` lấy các giá trị từ toàn bộ request payload (bao gồm query string), phương thức `query` sẽ chỉ lấy các giá trị từ query string:

```php
$name = $request->query('name');
```

Nếu query string value được yêu cầu không có, tham số thứ hai của phương thức này sẽ được trả về:

```php
$name = $request->query('name', 'Helen');
```

Bạn có thể gọi phương thức `query` mà không có bất kỳ đối số nào để lấy tất cả các query string values dưới dạng một associative array:

```php
$query = $request->query();
```

<a name="retrieving-json-input-values"></a>
#### Lấy JSON Input Values

Khi gửi JSON requests đến ứng dụng của bạn, bạn có thể truy cập JSON data thông qua phương thức `input` miễn là header `Content-Type` của request được đặt đúng thành `application/json`. Bạn thậm chí có thể sử dụng "dot" syntax để lấy các giá trị được lồng trong JSON arrays / objects:

```php
$name = $request->input('user.name');
```

<a name="retrieving-stringable-input-values"></a>
#### Lấy Stringable Input Values

Thay vì lấy input data của request dưới dạng một `string` nguyên thủy, bạn có thể sử dụng phương thức `string` để lấy request data dưới dạng một instance của [Illuminate\Support\Stringable](/docs/{{version}}/strings):

```php
$name = $request->string('name')->trim();
```

<a name="retrieving-integer-input-values"></a>
#### Lấy Integer Input Values

Để lấy input values dưới dạng integers, bạn có thể sử dụng phương thức `integer`. Phương thức này sẽ cố gắng cast input value thành một integer. Nếu input không có hoặc cast thất bại, nó sẽ trả về giá trị mặc định bạn chỉ định. Điều này đặc biệt hữu ích cho pagination hoặc các numeric inputs khác:

```php
$perPage = $request->integer('per_page');
```

<a name="retrieving-boolean-input-values"></a>
#### Lấy Boolean Input Values

Khi xử lý các HTML elements như checkboxes, ứng dụng của bạn có thể nhận các giá trị "truthy" thực sự là strings. Ví dụ, "true" hoặc "on". Để thuận tiện, bạn có thể sử dụng phương thức `boolean` để lấy các giá trị này dưới dạng booleans. Phương thức `boolean` trả về `true` cho 1, "1", true, "true", "on", và "yes". Tất cả các giá trị khác sẽ trả về `false`:

```php
$archived = $request->boolean('archived');
```

<a name="retrieving-array-input-values"></a>
#### Lấy Array Input Values

Input values chứa arrays có thể được lấy sử dụng phương thức `array`. Phương thức này sẽ luôn cast input value thành một array. Nếu request không chứa một input value với tên đã cho, một array rỗng sẽ được trả về:

```php
$versions = $request->array('versions');
```

<a name="retrieving-date-input-values"></a>
#### Lấy Date Input Values

Để thuận tiện, input values chứa dates / times có thể được lấy dưới dạng Carbon instances sử dụng phương thức `date`. Nếu request không chứa một input value với tên đã cho, `null` sẽ được trả về:

```php
$birthday = $request->date('birthday');
```

Tham số thứ hai và thứ ba được chấp nhận bởi phương thức `date` có thể được sử dụng để chỉ định format và timezone của date, tương ứng:

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

Nếu input value có nhưng có format không hợp lệ, một `InvalidArgumentException` sẽ được thrown; do đó, bạn nên validate input trước khi gọi phương thức `date`.

<a name="retrieving-interval-input-values"></a>
#### Lấy Interval Input Values

Input values chứa durations có thể được lấy dưới dạng `CarbonInterval` instances sử dụng phương thức `interval`. Nếu request không chứa một input value với tên đã cho, `null` sẽ được trả về:

```php
$duration = $request->interval('duration');
```

Nếu input value là numeric, bạn có thể cung cấp một unit làm tham số thứ hai. Unit có thể là một string như `second`, `minute`, hoặc `day`, hoặc một `Carbon\Unit` enum instance:

```php
use Carbon\Unit;

$timeout = $request->interval('timeout', 'second');

$delay = $request->interval('delay', Unit::Minute);
```

Nếu input value có nhưng có format không hợp lệ, một `InvalidArgumentException` sẽ được thrown; do đó, bạn nên validate input trước khi gọi phương thức `interval`.

<a name="retrieving-enum-input-values"></a>
#### Lấy Enum Input Values

Input values tương ứng với [PHP enums](https://www.php.net/manual/en/language.types.enumerations.php) cũng có thể được lấy từ request. Nếu request không chứa một input value với tên đã cho hoặc enum không có một backing value khớp với input value, `null` sẽ được trả về. Phương thức `enum` chấp nhận tên của input value và enum class làm tham số thứ nhất và thứ hai của nó:

```php
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```

Bạn cũng có thể cung cấp một giá trị mặc định sẽ được trả về nếu giá trị bị thiếu hoặc không hợp lệ:

```php
$status = $request->enum('status', Status::class, Status::Pending);
```

Nếu input value là một mảng các giá trị tương ứng với một PHP enum, bạn có thể sử dụng phương thức `enums` để lấy mảng các giá trị dưới dạng enum instances:

```php
use App\Enums\Product;

$products = $request->enums('products', Product::class);
```

<a name="retrieving-input-via-dynamic-properties"></a>
#### Lấy Input qua Dynamic Properties

Bạn cũng có thể truy cập user input sử dụng dynamic properties trên instance `Illuminate\Http\Request`. Ví dụ, nếu một trong các form của ứng dụng của bạn chứa một field `name`, bạn có thể truy cập giá trị của field như sau:

```php
$name = $request->name;
```

Khi sử dụng dynamic properties, Laravel sẽ đầu tiên tìm kiếm giá trị của parameter trong request payload. Nếu nó không có, Laravel sẽ tìm kiếm field trong các parameters của route đã khớp.

<a name="retrieving-a-portion-of-the-input-data"></a>
#### Lấy Một Phần của Input Data

Nếu bạn cần lấy một subset của input data, bạn có thể sử dụng các phương thức `only` và `except`. Cả hai phương thức này chấp nhận một `array` đơn hoặc một danh sách động của các đối số:

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

> [!WARNING]
> Phương thức `only` trả về tất cả các cặp key / value mà bạn yêu cầu; tuy nhiên, nó sẽ không trả về các cặp key / value không có trên request.

<a name="input-presence"></a>
### Kiểm tra Input

Bạn có thể sử dụng phương thức `has` để xác định xem một giá trị có trên request hay không. Phương thức `has` trả về `true` nếu giá trị có trên request:

```php
if ($request->has('name')) {
    // ...
}
```

Khi được cung cấp một mảng, phương thức `has` sẽ xác định xem tất cả các giá trị được chỉ định có hay không:

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

Phương thức `hasAny` trả về `true` nếu bất kỳ giá trị được chỉ định nào có:

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

Phương thức `whenHas` sẽ thực thi closure đã cho nếu một giá trị có trên request:

```php
$request->whenHas('name', function (string $input) {
    // ...
});
```

Một closure thứ hai có thể được truyền cho phương thức `whenHas` sẽ được thực thi nếu giá trị được chỉ định không có trên request:

```php
$request->whenHas('name', function (string $input) {
    // The "name" value is present...
}, function () {
    // The "name" value is not present...
});
```

Nếu bạn muốn xác định xem một giá trị có trên request và không phải là một chuỗi rỗng, bạn có thể sử dụng phương thức `filled`:

```php
if ($request->filled('name')) {
    // ...
}
```

Nếu bạn muốn xác định xem một giá trị bị thiếu từ request hoặc là một chuỗi rỗng, bạn có thể sử dụng phương thức `isNotFilled`:

```php
if ($request->isNotFilled('name')) {
    // ...
}
```

Khi được cung cấp một mảng, phương thức `isNotFilled` sẽ xác định xem tất cả các giá trị được chỉ định có bị thiếu hoặc rỗng hay không:

```php
if ($request->isNotFilled(['name', 'email'])) {
    // ...
}
```

Phương thức `anyFilled` trả về `true` nếu bất kỳ giá trị được chỉ định nào không phải là một chuỗi rỗng:

```php
if ($request->anyFilled(['name', 'email'])) {
    // ...
}
```

Phương thức `whenFilled` sẽ thực thi closure đã cho nếu một giá trị có trên request và không phải là một chuỗi rỗng:

```php
$request->whenFilled('name', function (string $input) {
    // ...
});
```

Một closure thứ hai có thể được truyền cho phương thức `whenFilled` sẽ được thực thi nếu giá trị được chỉ định không "filled":

```php
$request->whenFilled('name', function (string $input) {
    // The "name" value is filled...
}, function () {
    // The "name" value is not filled...
});
```

Để xác định xem một key nhất định bị thiếu từ request, bạn có thể sử dụng các phương thức `missing` và `whenMissing`:

```php
if ($request->missing('name')) {
    // ...
}

$request->whenMissing('name', function () {
    // The "name" value is missing...
}, function () {
    // The "name" value is present...
});
```

<a name="merging-additional-input"></a>
### Gộp Additional Input

Đôi khi bạn có thể cần gộp thủ công additional input vào input data hiện có của request. Để thực hiện điều này, bạn có thể sử dụng phương thức `merge`. Nếu một input key nhất định đã có trên request, nó sẽ được ghi đè bởi data được cung cấp cho phương thức `merge`:

```php
$request->merge(['votes' => 0]);
```

Phương thức `mergeIfMissing` có thể được sử dụng để gộp input vào request nếu các keys tương ứng không đã tồn tại trong input data của request:

```php
$request->mergeIfMissing(['votes' => 0]);
```

<a name="old-input"></a>
### Old Input

Laravel cho phép bạn giữ input từ một request trong request tiếp theo. Tính năng này đặc biệt hữu ích để re-populate forms sau khi phát hiện validation errors. Tuy nhiên, nếu bạn đang sử dụng [validation features](/docs/{{version}}/validation) có sẵn của Laravel, có thể bạn sẽ không cần sử dụng thủ công các phương thức session input flashing này trực tiếp, vì một số cơ sở validation có sẵn của Laravel sẽ gọi chúng tự động.

<a name="flashing-input-to-the-session"></a>
#### Flashing Input vào Session

Phương thức `flash` trên class `Illuminate\Http\Request` sẽ flash input hiện tại vào [session](/docs/{{version}}/session) để nó có sẵn trong request tiếp theo của user đến ứng dụng:

```php
$request->flash();
```

Bạn cũng có thể sử dụng các phương thức `flashOnly` và `flashExcept` để flash một subset của request data vào session. Các phương thức này hữu ích để giữ thông tin nhạy cảm như passwords ngoài session:

```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

<a name="flashing-input-then-redirecting"></a>
#### Flashing Input Sau đó Redirecting

Vì bạn thường sẽ muốn flash input vào session và sau đó redirect đến trang trước, bạn có thể dễ dàng chain input flashing vào một redirect sử dụng phương thức `withInput`:

```php
return redirect('/form')->withInput();

return redirect()->route('user.create')->withInput();

return redirect('/form')->withInput(
    $request->except('password')
);
```

<a name="retrieving-old-input"></a>
#### Lấy Old Input

Để lấy flashed input từ request trước, gọi phương thức `old` trên một instance của `Illuminate\Http\Request`. Phương thức `old` sẽ pull input data đã flashed trước đó từ [session](/docs/{{version}}/session):

```php
$username = $request->old('username');
```

Laravel cũng cung cấp một global `old` helper. Nếu bạn đang hiển thị old input trong một [Blade template](/docs/{{version}}/blade), thuận tiện hơn để sử dụng `old` helper để repopulate form. Nếu không có old input nào tồn tại cho field đã cho, `null` sẽ được trả về:

```blade
<input type="text" name="username" value="{{ old('username') }}">
```

<a name="cookies"></a>
### Cookies

<a name="retrieving-cookies-from-requests"></a>
#### Lấy Cookies Từ Requests

Tất cả cookies được tạo bởi Laravel framework đều được encrypted và signed với một authentication code, có nghĩa là chúng sẽ được coi là không hợp lệ nếu chúng đã được thay đổi bởi client. Để lấy một cookie value từ request, sử dụng phương thức `cookie` trên một instance `Illuminate\Http\Request`:

```php
$value = $request->cookie('name');
```

<a name="input-trimming-and-normalization"></a>
## Input Trimming và Normalization

Theo mặc định, Laravel bao gồm middleware `Illuminate\Foundation\Http\Middleware\TrimStrings` và `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull` trong global middleware stack của ứng dụng của bạn. Các middleware này sẽ tự động trim tất cả các incoming string fields trên request, cũng như convert bất kỳ empty string fields nào thành `null`. Điều này cho phép bạn không phải lo lắng về các mối quan tâm normalization này trong routes và controllers của bạn.

#### Vô hiệu hóa Input Normalization

Nếu bạn muốn vô hiệu hóa hành vi này cho tất cả các requests, bạn có thể remove hai middleware từ middleware stack của ứng dụng của bạn bằng cách gọi phương thức `$middleware->remove` trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;
use Illuminate\Foundation\Http\Middleware\TrimStrings;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->remove([
        ConvertEmptyStringsToNull::class,
        TrimStrings::class,
    ]);
})
```

Nếu bạn muốn vô hiệu hóa string trimming và empty string conversion cho một subset của các requests đến ứng dụng của bạn, bạn có thể sử dụng các phương thức middleware `trimStrings` và `convertEmptyStringsToNull` trong file `bootstrap/app.php` của ứng dụng của bạn. Cả hai phương thức chấp nhận một mảng các closures, nên trả về `true` hoặc `false` để chỉ định xem input normalization có nên được bỏ qua hay không:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->convertEmptyStringsToNull(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);

    $middleware->trimStrings(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);
})
```

<a name="files"></a>
## Files

<a name="retrieving-uploaded-files"></a>
### Lấy Uploaded Files

Bạn có thể lấy uploaded files từ một instance `Illuminate\Http\Request` sử dụng phương thức `file` hoặc sử dụng dynamic properties. Phương thức `file` trả về một instance của class `Illuminate\Http\UploadedFile`, extends PHP class `SplFileInfo` và cung cấp nhiều phương thức để tương tác với file:

```php
$file = $request->file('photo');

$file = $request->photo;
```

Bạn có thể xác định xem một file có trên request hay không sử dụng phương thức `hasFile`:

```php
if ($request->hasFile('photo')) {
    // ...
}
```

<a name="validating-successful-uploads"></a>
#### Validating Successful Uploads

Ngoài việc kiểm tra xem file có hay không, bạn có thể xác minh rằng không có vấn đề nào khi upload file thông qua phương thức `isValid`:

```php
if ($request->file('photo')->isValid()) {
    // ...
}
```

<a name="file-paths-extensions"></a>
#### File Paths và Extensions

Class `UploadedFile` cũng chứa các phương thức để truy cập fully-qualified path của file và extension của nó. Phương thức `extension` sẽ cố gắng đoán extension của file dựa trên nội dung của nó. Extension này có thể khác với extension được cung cấp bởi client:

```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```

<a name="other-file-methods"></a>
#### Các Phương thức File Khác

Có nhiều phương thức khác có sẵn trên `UploadedFile` instances. Hãy xem [API documentation cho class](https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php) để biết thêm thông tin về các phương thức này.

<a name="storing-uploaded-files"></a>
### Lưu Uploaded Files

Để lưu một uploaded file, bạn thường sẽ sử dụng một trong các [filesystems](/docs/{{version}}/filesystem) đã cấu hình của bạn. Class `UploadedFile` có một phương thức `store` sẽ di chuyển một uploaded file đến một trong các disks của bạn, có thể là một location trên local filesystem của bạn hoặc một cloud storage location như Amazon S3.

Phương thức `store` chấp nhận path nơi file nên được lưu tương đối với root directory đã cấu hình của filesystem. Path này không nên chứa một filename, vì một unique ID sẽ tự động được generated để phục vụ như filename.

Phương thức `store` cũng chấp nhận một tham số thứ hai tùy chọn cho tên của disk nên được sử dụng để lưu file. Phương thức sẽ trả về path của file tương đối với root của disk:

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

Nếu bạn không muốn một filename được tự động generated, bạn có thể sử dụng phương thức `storeAs`, chấp nhận path, filename, và tên disk làm các đối số của nó:

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

> [!NOTE]
> Để biết thêm thông tin về file storage trong Laravel, hãy xem [file storage documentation](/docs/{{version}}/filesystem) đầy đủ.

<a name="configuring-trusted-proxies"></a>
## Cấu hình Trusted Proxies

Khi chạy ứng dụng của bạn phía sau một load balancer mà terminates TLS / SSL certificates, bạn có thể nhận thấy ứng dụng của bạn đôi khi không generate HTTPS links khi sử dụng `url` helper. Thông thường điều này là vì ứng dụng của bạn đang được forwarded traffic từ load balancer của bạn trên port 80 và không biết nó nên generate secure links.

Để giải quyết điều này, bạn có thể enable middleware `Illuminate\Http\Middleware\TrustProxies` được bao gồm trong ứng dụng Laravel của bạn, cho phép bạn nhanh chóng customize các load balancers hoặc proxies nên được tin cậy bởi ứng dụng của bạn. Trusted proxies của bạn nên được chỉ định sử dụng phương thức middleware `trustProxies` trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: [
        '192.168.1.1',
        '10.0.0.0/8',
    ]);
})
```

Ngoài việc cấu hình trusted proxies, bạn cũng có thể cấu hình các proxy headers nên được tin cậy:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(headers: Request::HEADER_X_FORWARDED_FOR |
        Request::HEADER_X_FORWARDED_HOST |
        Request::HEADER_X_FORWARDED_PORT |
        Request::HEADER_X_FORWARDED_PROTO |
        Request::HEADER_X_FORWARDED_AWS_ELB
    );
})
```

> [!NOTE]
> Nếu bạn đang sử dụng AWS Elastic Load Balancing, giá trị `headers` nên là `Request::HEADER_X_FORWARDED_AWS_ELB`. Nếu load balancer của bạn sử dụng header `Forwarded` tiêu chuẩn từ [RFC 7239](https://www.rfc-editor.org/rfc/rfc7239#section-4), giá trị `headers` nên là `Request::HEADER_FORWARDED`. Để biết thêm thông tin về các constants có thể được sử dụng trong giá trị `headers`, hãy xem documentation của Symfony về [trusting proxies](https://symfony.com/doc/current/deployment/proxies.html).

<a name="trusting-all-proxies"></a>
#### Tin cậy Tất cả Proxies

Nếu bạn đang sử dụng Amazon AWS hoặc một nhà cung cấp load balancer "cloud" khác, bạn có thể không biết các địa chỉ IP của balancers thực tế của bạn. Trong trường hợp này, bạn có thể sử dụng `*` để tin cậy tất cả proxies:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: '*');
})
```

<a name="configuring-trusted-hosts"></a>
## Cấu hình Trusted Hosts

Theo mặc định, Laravel sẽ phản hồi tất cả các requests nó nhận bất kể nội dung của header `Host` của HTTP request. Ngoài ra, giá trị của header `Host` sẽ được sử dụng khi generating absolute URLs đến ứng dụng của bạn trong một web request.

Thông thường, bạn nên cấu hình web server của bạn, chẳng hạn như Nginx hoặc Apache, để chỉ gửi requests đến ứng dụng của bạn khớp với một hostname nhất định. Tuy nhiên, nếu bạn không có khả năng customize web server của bạn trực tiếp và cần hướng dẫn Laravel chỉ phản hồi với các hostnames nhất định, bạn có thể làm như vậy bằng cách enable middleware `Illuminate\Http\Middleware\TrustHosts` cho ứng dụng của bạn.

Để enable middleware `TrustHosts`, bạn nên gọi phương thức middleware `trustHosts` trong file `bootstrap/app.php` của ứng dụng của bạn. Sử dụng đối số `at` của phương thức này, bạn có thể chỉ định các hostnames mà ứng dụng của bạn nên phản hồi. Chuỗi hostname được coi như một regular expression. Các requests đến với các header `Host` khác sẽ bị reject:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$']);
})
```

Theo mặc định, các requests đến từ subdomains của URL của ứng dụng cũng được tin cậy tự động. Nếu bạn muốn vô hiệu hóa hành vi này, bạn có thể sử dụng đối số `subdomains`:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$'], subdomains: false);
})
```

Nếu bạn cần truy cập các file cấu hình hoặc database của ứng dụng để xác định trusted hosts của bạn, bạn có thể cung cấp một closure cho đối số `at`:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: fn () => config('app.trusted_hosts'));
})
```

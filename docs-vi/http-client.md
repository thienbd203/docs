# HTTP Client

- [Giới thiệu](#introduction)
- [Thực hiện Requests](#making-requests)
    - [Request Data](#request-data)
    - [Headers](#headers)
    - [Authentication](#authentication)
    - [Timeout](#timeout)
    - [Retries](#retries)
    - [Error Handling](#error-handling)
    - [Guzzle Middleware](#guzzle-middleware)
    - [Guzzle Options](#guzzle-options)
- [Concurrent Requests](#concurrent-requests)
    - [Request Pooling](#request-pooling)
    - [Request Batching](#request-batching)
- [Macros](#macros)
- [Testing](#testing)
    - [Faking Responses](#faking-responses)
    - [Inspecting Requests](#inspecting-requests)
    - [Preventing Stray Requests](#preventing-stray-requests)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một API tối giản và expressive xung quanh [Guzzle HTTP client](http://docs.guzzlephp.org/en/stable/), cho phép bạn nhanh chóng thực hiện các HTTP request đi để giao tiếp với các ứng dụng web khác. Wrapper của Laravel xung quanh Guzzle tập trung vào các use case phổ biến nhất và mang lại trải nghiệm developer tuyệt vời.

<a name="making-requests"></a>
## Thực hiện Requests

Để thực hiện requests, bạn có thể sử dụng các phương thức `head`, `get`, `post`, `put`, `patch`, và `delete` được cung cấp bởi facade `Http`. Đầu tiên, hãy xem cách thực hiện một `GET` request cơ bản đến một URL khác:

```php
use Illuminate\Support\Facades\Http;

$response = Http::get('http://example.com');
```

Phương thức `get` trả về một instance của `Illuminate\Http\Client\Response`, cung cấp nhiều phương thức có thể được sử dụng để kiểm tra response:

```php
$response->body() : string;
$response->json($key = null, $default = null) : mixed;
$response->object() : object;
$response->collect($key = null) : Illuminate\Support\Collection;
$response->resource() : resource;
$response->status() : int;
$response->successful() : bool;
$response->redirect(): bool;
$response->failed() : bool;
$response->clientError() : bool;
$response->header($header) : string;
$response->headers() : array;
```

Object `Illuminate\Http\Client\Response` cũng implement interface PHP `ArrayAccess`, cho phép bạn truy cập dữ liệu JSON response trực tiếp trên response:

```php
return Http::get('http://example.com/users/1')['name'];
```

Ngoài các phương thức response được liệt kê ở trên, các phương thức sau có thể được sử dụng để xác định xem response có status code cụ thể hay không:

```php
$response->ok() : bool;                  // 200 OK
$response->created() : bool;             // 201 Created
$response->accepted() : bool;            // 202 Accepted
$response->noContent() : bool;           // 204 No Content
$response->movedPermanently() : bool;    // 301 Moved Permanently
$response->found() : bool;               // 302 Found
$response->badRequest() : bool;          // 400 Bad Request
$response->unauthorized() : bool;        // 401 Unauthorized
$response->paymentRequired() : bool;     // 402 Payment Required
$response->forbidden() : bool;           // 403 Forbidden
$response->notFound() : bool;            // 404 Not Found
$response->requestTimeout() : bool;      // 408 Request Timeout
$response->conflict() : bool;            // 409 Conflict
$response->unprocessableEntity() : bool; // 422 Unprocessable Entity
$response->tooManyRequests() : bool;     // 429 Too Many Requests
$response->serverError() : bool;         // 500 Internal Server Error
```

<a name="uri-templates"></a>
#### URI Templates

HTTP client cũng cho phép bạn xây dựng request URLs sử dụng [URI template specification](https://www.rfc-editor.org/rfc/rfc6570). Để định nghĩa các URL parameters có thể được mở rộng bởi URI template của bạn, bạn có thể sử dụng phương thức `withUrlParameters`:

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '13.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

<a name="dumping-requests"></a>
#### Dumping Requests

Nếu bạn muốn dump outgoing request instance trước khi nó được gửi và terminate việc thực thi script, bạn có thể thêm phương thức `dd` vào đầu định nghĩa request của mình:

```php
return Http::dd()->get('http://example.com');
```

<a name="request-data"></a>
### Request Data

Tất nhiên, khi thực hiện các request `POST`, `PUT`, và `PATCH`, việc gửi thêm dữ liệu với request là rất phổ biến, vì vậy các phương thức này chấp nhận một mảng dữ liệu làm đối số thứ hai. Theo mặc định, dữ liệu sẽ được gửi sử dụng content type `application/json`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::post('http://example.com/users', [
    'name' => 'Steve',
    'role' => 'Network Administrator',
]);
```

<a name="get-request-query-parameters"></a>
#### GET Request Query Parameters

Khi thực hiện các request `GET`, bạn có thể thêm query string vào URL trực tiếp hoặc truyền một mảng key / value pairs làm đối số thứ hai cho phương thức `get`:

```php
$response = Http::get('http://example.com/users', [
    'name' => 'Taylor',
    'page' => 1,
]);
```

Ngoài ra, phương thức `withQueryParameters` có thể được sử dụng:

```php
Http::retry(3, 100)->withQueryParameters([
    'name' => 'Taylor',
    'page' => 1,
])->get('http://example.com/users');
```

<a name="sending-form-url-encoded-requests"></a>
#### Sending Form URL Encoded Requests

Nếu bạn muốn gửi dữ liệu sử dụng content type `application/x-www-form-urlencoded`, bạn nên gọi phương thức `asForm` trước khi thực hiện request:

```php
$response = Http::asForm()->post('http://example.com/users', [
    'name' => 'Sara',
    'role' => 'Privacy Consultant',
]);
```

<a name="sending-a-raw-request-body"></a>
#### Sending a Raw Request Body

Bạn có thể sử dụng phương thức `withBody` nếu bạn muốn cung cấp raw request body khi thực hiện request. Content type có thể được cung cấp qua đối số thứ hai của phương thức:

```php
$response = Http::withBody(
    base64_encode($photo), 'image/jpeg'
)->post('http://example.com/photo');
```

<a name="multi-part-requests"></a>
#### Multi-Part Requests

Nếu bạn muốn gửi files dưới dạng multi-part requests, bạn nên gọi phương thức `attach` trước khi thực hiện request. Phương thức này chấp nhận tên của file và nội dung của nó. Nếu cần, bạn có thể cung cấp đối số thứ ba sẽ được coi là filename của file, trong khi đối số thứ tư có thể được sử dụng để cung cấp headers liên quan đến file:

```php
$response = Http::attach(
    'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
)->post('http://example.com/attachments');
```

Thay vì truyền nội dung thô của một file, bạn có thể truyền một stream resource:

```php
$photo = fopen('photo.jpg', 'r');

$response = Http::attach(
    'attachment', $photo, 'photo.jpg'
)->post('http://example.com/attachments');
```

<a name="headers"></a>
### Headers

Headers có thể được thêm vào requests sử dụng phương thức `withHeaders`. Phương thức `withHeaders` này chấp nhận một mảng key / value pairs:

```php
$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

Bạn có thể sử dụng phương thức `accept` để chỉ định content type mà ứng dụng của bạn mong đợi trong response cho request của mình:

```php
$response = Http::accept('application/json')->get('http://example.com/users');
```

Để thuận tiện, bạn có thể sử dụng phương thức `acceptJson` để nhanh chóng chỉ định rằng ứng dụng của bạn mong đợi content type `application/json` trong response cho request của mình:

```php
$response = Http::acceptJson()->get('http://example.com/users');
```

Phương thức `withHeaders` merge các headers mới vào các headers hiện có của request. Nếu cần, bạn có thể thay thế hoàn toàn tất cả các headers sử dụng phương thức `replaceHeaders`:

```php
$response = Http::withHeaders([
    'X-Original' => 'foo',
])->replaceHeaders([
    'X-Replacement' => 'bar',
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

<a name="authentication"></a>
### Authentication

Bạn có thể chỉ định basic và digest authentication credentials sử dụng các phương thức `withBasicAuth` và `withDigestAuth`, tương ứng:

```php
// Basic authentication...
$response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

// Digest authentication...
$response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);
```

<a name="bearer-tokens"></a>
#### Bearer Tokens

Nếu bạn muốn nhanh chóng thêm bearer token vào header `Authorization` của request, bạn có thể sử dụng phương thức `withToken`:

```php
$response = Http::withToken('token')->post(/* ... */);
```

<a name="timeout"></a>
### Timeout

Phương thức `timeout` có thể được sử dụng để chỉ định số giây tối đa để chờ response. Theo mặc định, HTTP client sẽ timeout sau 30 giây:

```php
$response = Http::timeout(3)->get(/* ... */);
```

Nếu timeout được chỉ định bị vượt quá, một instance của `Illuminate\Http\Client\ConnectionException` sẽ được thrown.

Bạn có thể chỉ định số giây tối đa để chờ khi cố gắng kết nối đến một server sử dụng phương thức `connectTimeout`. Mặc định là 10 giây:

```php
$response = Http::connectTimeout(3)->get(/* ... */);
```

<a name="retries"></a>
### Retries

Nếu bạn muốn HTTP client tự động retry request nếu xảy ra client hoặc server error, bạn có thể sử dụng phương thức `retry`. Phương thức `retry` chấp nhận số lần tối đa request nên được thực hiện và số mili-giây mà Laravel nên chờ giữa các lần thử:

```php
$response = Http::retry(3, 100)->post(/* ... */);
```

Nếu bạn muốn tính toán thủ công số mili-giây để sleep giữa các lần thử, bạn có thể truyền một closure làm đối số thứ hai cho phương thức `retry`:

```php
use Exception;

$response = Http::retry(3, function (int $attempt, Exception $exception) {
    return $attempt * 100;
})->post(/* ... */);
```

Để thuận tiện, bạn cũng có thể cung cấp một mảng làm đối số thứ nhất cho phương thức `retry`. Mảng này sẽ được sử dụng để xác định bao nhiêu mili-giây để sleep giữa các lần thử tiếp theo:

```php
$response = Http::retry([100, 200])->post(/* ... */);
```

Nếu cần, bạn có thể truyền đối số thứ ba cho phương thức `retry`. Đối số thứ ba nên là một callable xác định xem các retries có nên được thực hiện hay không. Ví dụ, bạn có thể chỉ muốn retry request nếu request ban đầu gặp một `ConnectionException`:

```php
use Illuminate\Http\Client\PendingRequest;
use Throwable;

$response = Http::retry(3, 100, function (Throwable $exception, PendingRequest $request) {
    return $exception instanceof ConnectionException;
})->post(/* ... */);
```

Nếu một lần thử request thất bại, bạn có thể muốn thay đổi request trước khi một lần thử mới được thực hiện. Bạn có thể đạt được điều này bằng cách sửa đổi đối số request được cung cấp cho callable mà bạn đã cung cấp cho phương thức `retry`. Ví dụ, bạn có thể muốn retry request với một authorization token mới nếu lần thử đầu tiên trả về một authentication error:

```php
use Illuminate\Http\Client\PendingRequest;
use Illuminate\Http\Client\RequestException;
use Throwable;

$response = Http::withToken($this->getToken())->retry(2, 0, function (Throwable $exception, PendingRequest $request) {
    if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
        return false;
    }

    $request->withToken($this->getNewToken());

    return true;
})->post(/* ... */);
```

Nếu tất cả các requests thất bại, một instance của `Illuminate\Http\Client\RequestException` sẽ được thrown. Nếu bạn muốn tắt hành vi này, bạn có thể cung cấp một đối số `throw` với giá trị `false`. Khi bị tắt, response cuối cùng nhận được bởi client sẽ được trả về sau khi tất cả các retries đã được thực hiện:

```php
$response = Http::retry(3, 100, throw: false)->post(/* ... */);
```

> [!WARNING]
> Nếu tất cả các requests thất bại do vấn đề kết nối, một `Illuminate\Http\Client\ConnectionException` vẫn sẽ được thrown ngay cả khi đối số `throw` được đặt thành `false`.

<a name="error-handling"></a>
### Error Handling

Khác với hành vi mặc định của Guzzle, wrapper HTTP client của Laravel không throw exceptions trên client hoặc server errors (các responses mức `400` và `500` từ servers). Bạn có thể xác định xem một trong các errors này có được trả về hay không sử dụng các phương thức `successful`, `clientError`, hoặc `serverError`:

```php
// Xác định xem status code có >= 200 và < 300 không...
$response->successful();

// Xác định xem status code có >= 400 không...
$response->failed();

// Xác định xem response có status code mức 400 không...
$response->clientError();

// Xác định xem response có status code mức 500 không...
$response->serverError();

// Thực thi ngay lập tức callback đã cho nếu có client hoặc server error...
$response->onError(callable $callback);
```

<a name="throwing-exceptions"></a>
#### Throwing Exceptions

Nếu bạn có một response instance và muốn throw một instance của `Illuminate\Http\Client\RequestException` nếu response status code cho thấy client hoặc server error, bạn có thể sử dụng các phương thức `throw` hoặc `throwIf`:

```php
use Illuminate\Http\Client\Response;

$response = Http::post(/* ... */);

// Throw một exception nếu xảy ra client hoặc server error...
$response->throw();

// Throw một exception nếu xảy ra error và điều kiện đã cho là true...
$response->throwIf($condition);

// Throw một exception nếu xảy ra error và closure đã cho trả về true...
$response->throwIf(fn (Response $response) => true);

// Throw một exception nếu xảy ra error và điều kiện đã cho là false...
$response->throwUnless($condition);

// Throw một exception nếu xảy ra error và closure đã cho trả về false...
$response->throwUnless(fn (Response $response) => false);

// Throw một exception nếu response có status code cụ thể...
$response->throwIfStatus(403);

// Throw một exception trừ khi response có status code cụ thể...
$response->throwUnlessStatus(200);

// Throw một exception nếu xảy ra server error (status >500)...
$response->throwIfServerError();

// Throw một exception nếu xảy ra client error (status >400 và <500)...
$response->throwIfClientError();

return $response['user']['id'];
```

Instance `Illuminate\Http\Client\RequestException` có một property public `$response` cho phép bạn kiểm tra response được trả về.

Phương thức `throw` trả về response instance nếu không có error xảy ra, cho phép bạn chain các operations khác vào phương thức `throw`:

```php
return Http::post(/* ... */)->throw()->json();
```

Nếu bạn muốn thực hiện một số logic bổ sung trước khi exception được thrown, bạn có thể truyền một closure cho phương thức `throw`. Exception sẽ được thrown tự động sau khi closure được invoked, vì vậy bạn không cần re-throw exception từ bên trong closure:

```php
use Illuminate\Http\Client\Response;
use Illuminate\Http\Client\RequestException;

return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
    // ...
})->json();
```

Theo mặc định, messages của `RequestException` được cắt ngắn thành 120 ký tự khi được log hoặc báo cáo. Để tùy chỉnh hoặc tắt hành vi này, bạn có thể sử dụng các phương thức `truncateAt` và `dontTruncate` khi cấu hình behavior đã đăng ký của ứng dụng trong file `bootstrap/app.php` của bạn:

```php
use Illuminate\Http\Client\RequestException;

->registered(function (): void {
    // Cắt ngắn request exception messages thành 240 ký tự...
    RequestException::truncateAt(240);

    // Tắt cắt ngắn request exception message...
    RequestException::dontTruncate();
})
```

Ngoài ra, bạn có thể tùy chỉnh behavior cắt ngắn exception cho mỗi request sử dụng phương thức `truncateExceptionsAt`:

```php
return Http::truncateExceptionsAt(240)->post(/* ... */);
```

<a name="guzzle-middleware"></a>
### Guzzle Middleware

Vì HTTP client của Laravel được hỗ trợ bởi Guzzle, bạn có thể tận dụng [Guzzle Middleware](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html) để thao tác outgoing request hoặc kiểm tra incoming response. Để thao tác outgoing request, đăng ký một Guzzle middleware qua phương thức `withRequestMiddleware`:

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\RequestInterface;

$response = Http::withRequestMiddleware(
    function (RequestInterface $request) {
        return $request->withHeader('X-Example', 'Value');
    }
)->get('http://example.com');
```

Tương tự, bạn có thể kiểm tra incoming HTTP response bằng cách đăng ký một middleware qua phương thức `withResponseMiddleware`:

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\ResponseInterface;

$response = Http::withResponseMiddleware(
    function (ResponseInterface $response) {
        $header = $response->getHeader('X-Example');

        // ...

        return $response;
    }
)->get('http://example.com');
```

<a name="global-middleware"></a>
#### Global Middleware

Đôi khi, bạn muốn đăng ký một middleware áp dụng cho mọi outgoing request và incoming response. Để thực hiện điều này, bạn có thể sử dụng các phương thức `globalRequestMiddleware` và `globalResponseMiddleware`. Thông thường, các phương thức này nên được invoked trong phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\Http;

Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

<a name="guzzle-options"></a>
### Guzzle Options

Bạn có thể chỉ định các [Guzzle request options](http://docs.guzzlephp.org/en/stable/request-options.html) bổ sung cho một outgoing request sử dụng phương thức `withOptions`. Phương thức `withOptions` chấp nhận một mảng key / value pairs:

```php
$response = Http::withOptions([
    'debug' => true,
])->get('http://example.com/users');
```

<a name="global-options"></a>
#### Global Options

Để cấu hình các options mặc định cho mọi outgoing request, bạn có thể sử dụng phương thức `globalOptions`. Thông thường, phương thức này nên được invoked từ phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::globalOptions([
        'allow_redirects' => false,
    ]);
}
```

<a name="concurrent-requests"></a>
## Concurrent Requests

Đôi khi, bạn có thể muốn thực hiện nhiều HTTP requests đồng thời. Nói cách khác, bạn muốn một số requests được dispatch cùng một lúc thay vì phát hành các requests tuần tự. Điều này có thể dẫn đến cải thiện hiệu suất đáng kể khi tương tác với các HTTP APIs chậm.

<a name="request-pooling"></a>
### Request Pooling

May mắn thay, bạn có thể thực hiện điều này sử dụng phương thức `pool`. Phương thức `pool` chấp nhận một closure nhận một instance `Illuminate\Http\Client\Pool`, cho phép bạn dễ dàng thêm requests vào request pool để dispatch:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->get('http://localhost/first'),
    $pool->get('http://localhost/second'),
    $pool->get('http://localhost/third'),
]);

return $responses[0]->ok() &&
       $responses[1]->ok() &&
       $responses[2]->ok();
```

Như bạn có thể thấy, mỗi response instance có thể được truy cập dựa trên thứ tự nó được thêm vào pool. Nếu bạn muốn, bạn có thể đặt tên cho các requests sử dụng phương thức `as`, cho phép bạn truy cập các responses tương ứng theo tên:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->as('first')->get('http://localhost/first'),
    $pool->as('second')->get('http://localhost/second'),
    $pool->as('third')->get('http://localhost/third'),
]);

return $responses['first']->ok();
```

Số lượng đồng thời tối đa của request pool có thể được kiểm soát bằng cách cung cấp đối số `concurrency` cho phương thức `pool`. Giá trị này xác định số lượng tối đa các HTTP requests có thể đồng thời in-flight trong khi xử lý request pool:

```php
$responses = Http::pool(fn (Pool $pool) => [
    // ...
], concurrency: 5);
```

<a name="customizing-concurrent-requests"></a>
#### Customizing Concurrent Requests

Phương thức `pool` không thể được chain với các phương thức HTTP client khác như các phương thức `withHeaders` hoặc `middleware`. Nếu bạn muốn áp dụng custom headers hoặc middleware cho pooled requests, bạn nên cấu hình các options đó trên mỗi request trong pool:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$headers = [
    'X-Example' => 'example',
];

$responses = Http::pool(fn (Pool $pool) => [
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
]);
```

<a name="request-batching"></a>
### Request Batching

Một cách khác để làm việc với concurrent requests trong Laravel là sử dụng phương thức `batch`. Giống như phương thức `pool`, nó chấp nhận một closure nhận một instance `Illuminate\Http\Client\Batch`, cho phép bạn dễ dàng thêm requests vào request pool để dispatch, nhưng nó cũng cho phép bạn định nghĩa các completion callbacks:

```php
use Illuminate\Http\Client\Batch;
use Illuminate\Http\Client\ConnectionException;
use Illuminate\Http\Client\RequestException;
use Illuminate\Http\Client\Response;
use Illuminate\Support\Facades\Http;

$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
    $batch->get('http://localhost/third'),
])->before(function (Batch $batch) {
    // Batch đã được tạo nhưng chưa có request nào được khởi tạo...
})->progress(function (Batch $batch, int|string $key, Response $response) {
    // Một request riêng lẻ đã hoàn thành thành công...
})->then(function (Batch $batch, array $results) {
    // Tất cả requests đã hoàn thành thành công...
})->catch(function (Batch $batch, int|string $key, Response|RequestException|ConnectionException $response) {
    // Phát hiện lỗi batch request...
})->finally(function (Batch $batch, array $results) {
    // Batch đã hoàn thành thực thi...
})->send();
```

Giống như phương thức `pool`, bạn có thể sử dụng phương thức `as` để đặt tên cho các requests của mình:

```php
$responses = Http::batch(fn (Batch $batch) => [
    $batch->as('first')->get('http://localhost/first'),
    $batch->as('second')->get('http://localhost/second'),
    $batch->as('third')->get('http://localhost/third'),
])->send();
```

Sau khi một `batch` được bắt đầu bằng cách gọi phương thức `send`, bạn không thể thêm requests mới vào nó. Cố gắng làm điều này sẽ dẫn đến một exception `Illuminate\Http\Client\BatchInProgressException` được thrown.

Số lượng đồng thời tối đa của request batch có thể được kiểm soát qua phương thức `concurrency`. Giá trị này xác định số lượng tối đa các HTTP requests có thể đồng thời in-flight trong khi xử lý request batch:

```php
$responses = Http::batch(fn (Batch $batch) => [
    // ...
])->concurrency(5)->send();
```

<a name="inspecting-batches"></a>
#### Inspecting Batches

Instance `Illuminate\Http\Client\Batch` được cung cấp cho batch completion callbacks có nhiều properties và methods để giúp bạn tương tác và kiểm tra một batch requests cụ thể:

```php
// Số lượng requests được gán cho batch...
$batch->totalRequests;

// Số lượng requests chưa được xử lý...
$batch->pendingRequests;

// Số lượng requests đã thất bại...
$batch->failedRequests;

// Số lượng requests đã được xử lý cho đến nay...
$batch->processedRequests();

// Cho biết batch đã hoàn thành thực thi...
$batch->finished();

// Cho biết batch có request failures...
$batch->hasFailures();
```
<a name="deferring-batches"></a>
#### Deferring Batches

Khi phương thức `defer` được invoked, batch requests không được thực thi ngay lập tức. Thay vào đó, Laravel sẽ thực thi batch sau khi HTTP response của request ứng dụng hiện tại đã được gửi đến người dùng, giữ cho ứng dụng của bạn cảm thấy nhanh và responsive:

```php
use Illuminate\Http\Client\Batch;
use Illuminate\Support\Facades\Http;

$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
    $batch->get('http://localhost/third'),
])->then(function (Batch $batch, array $results) {
    // Tất cả requests đã hoàn thành thành công...
})->defer();
```

<a name="macros"></a>
## Macros

HTTP client của Laravel cho phép bạn định nghĩa "macros", có thể đóng vai trò là một cơ chế fluent và expressive để cấu hình các request paths và headers phổ biến khi tương tác với các services trong toàn bộ ứng dụng của bạn. Để bắt đầu, bạn có thể định nghĩa macro trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::macro('github', function () {
        return Http::withHeaders([
            'X-Example' => 'example',
        ])->baseUrl('https://github.com');
    });
}
```

Sau khi macro của bạn đã được cấu hình, bạn có thể invoke nó từ bất cứ đâu trong ứng dụng của mình để tạo một pending request với cấu hình được chỉ định:

```php
$response = Http::github()->get('/');
```

<a name="testing"></a>
## Testing

Nhiều services của Laravel cung cấp chức năng để giúp bạn dễ dàng và expressively viết tests, và HTTP client của Laravel không ngoại lệ. Phương thức `fake` của facade `Http` cho phép bạn chỉ định HTTP client trả về stubbed / dummy responses khi requests được thực hiện.

<a name="faking-responses"></a>
### Faking Responses

Ví dụ, để chỉ định HTTP client trả về các responses với status code `200` rỗng cho mọi request, bạn có thể gọi phương thức `fake` không có đối số:

```php
use Illuminate\Support\Facades\Http;

Http::fake();

$response = Http::post(/* ... */);
```

<a name="faking-specific-urls"></a>
#### Faking Specific URLs

Ngoài ra, bạn có thể truyền một mảng cho phương thức `fake`. Các keys của mảng nên đại diện cho các URL patterns mà bạn muốn fake và các responses liên quan của chúng. Ký tự `*` có thể được sử dụng làm ký tự wildcard. Bạn có thể sử dụng phương thức `response` của facade `Http` để xây dựng stub / fake responses cho các endpoints này:

```php
Http::fake([
    // Stub một JSON response cho GitHub endpoints...
    'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

    // Stub một string response cho Google endpoints...
    'google.com/*' => Http::response('Hello World', 200, $headers),
]);
```

Bất kỳ requests nào được thực hiện đến các URLs chưa được fake sẽ thực sự được thực thi. Nếu bạn muốn chỉ định một URL pattern fallback sẽ stub tất cả các URLs không khớp, bạn có thể sử dụng một ký tự `*` đơn:

```php
Http::fake([
    // Stub một JSON response cho GitHub endpoints...
    'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

    // Stub một string response cho tất cả các endpoints khác...
    '*' => Http::response('Hello World', 200, ['Headers']),
]);
```

Để thuận tiện, các responses string, JSON, và rỗng đơn giản có thể được tạo bằng cách cung cấp một string, array, hoặc integer làm response:

```php
Http::fake([
    'google.com/*' => 'Hello World',
    'github.com/*' => ['foo' => 'bar'],
    'chatgpt.com/*' => 200,
]);
```

<a name="faking-connection-exceptions"></a>
#### Faking Exceptions

Đôi khi bạn có thể cần kiểm tra behavior của ứng dụng nếu HTTP client gặp một `Illuminate\Http\Client\ConnectionException` khi cố gắng thực hiện request. Bạn có thể chỉ định HTTP client throw một connection exception sử dụng phương thức `failedConnection`:

```php
Http::fake([
    'github.com/*' => Http::failedConnection(),
]);
```

Để kiểm tra behavior của ứng dụng nếu một `Illuminate\Http\Client\RequestException` được thrown, bạn có thể sử dụng phương thức `failedRequest`:

```php
$this->mock(GithubService::class);
    ->shouldReceive('getUser')
    ->andThrow(
        Http::failedRequest(['code' => 'not_found'], 404)
    );
```

<a name="faking-response-sequences"></a>
#### Faking Response Sequences

Đôi khi bạn có thể cần chỉ định rằng một URL đơn nên trả về một chuỗi các fake responses theo một thứ tự cụ thể. Bạn có thể thực hiện điều này sử dụng phương thức `Http::sequence` để xây dựng các responses:

```php
Http::fake([
    // Stub một chuỗi các responses cho GitHub endpoints...
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->pushStatus(404),
]);
```

Khi tất cả các responses trong một response sequence đã được tiêu thụ, bất kỳ requests tiếp theo sẽ gây ra response sequence throw một exception. Nếu bạn muốn chỉ định một response mặc định nên được trả về khi một sequence rỗng, bạn có thể sử dụng phương thức `whenEmpty`:

```php
Http::fake([
    // Stub một chuỗi các responses cho GitHub endpoints...
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->whenEmpty(Http::response()),
]);
```

Nếu bạn muốn fake một chuỗi các responses nhưng không cần chỉ định một URL pattern cụ thể nên được fake, bạn có thể sử dụng phương thức `Http::fakeSequence`:

```php
Http::fakeSequence()
    ->push('Hello World', 200)
    ->whenEmpty(Http::response());
```

<a name="fake-callback"></a>
#### Fake Callback

Nếu bạn cần logic phức tạp hơn để xác định responses nào để trả về cho các endpoints cụ thể, bạn có thể truyền một closure cho phương thức `fake`. Closure này sẽ nhận một instance của `Illuminate\Http\Client\Request` và nên trả về một response instance. Trong closure của bạn, bạn có thể thực hiện bất kỳ logic nào cần thiết để xác định loại response nào để trả về:

```php
use Illuminate\Http\Client\Request;

Http::fake(function (Request $request) {
    return Http::response('Hello World', 200);
});
```

<a name="inspecting-requests"></a>
### Inspecting Requests

Khi fake responses, đôi khi bạn có thể muốn kiểm tra các requests mà client nhận được để đảm bảo ứng dụng của bạn đang gửi dữ liệu hoặc headers đúng. Bạn có thể thực hiện điều này bằng cách gọi phương thức `Http::assertSent` sau khi gọi `Http::fake`.

Phương thức `assertSent` chấp nhận một closure sẽ nhận một instance `Illuminate\Http\Client\Request` và nên trả về một giá trị boolean cho biết request có khớp với kỳ vọng của bạn hay không. Để test pass, ít nhất một request phải được issued khớp với các kỳ vọng đã cho:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::fake();

Http::withHeaders([
    'X-First' => 'foo',
])->post('http://example.com/users', [
    'name' => 'Taylor',
    'role' => 'Developer',
]);

Http::assertSent(function (Request $request) {
    return $request->hasHeader('X-First', 'foo') &&
           $request->url() == 'http://example.com/users' &&
           $request['name'] == 'Taylor' &&
           $request['role'] == 'Developer';
});
```

Nếu cần, bạn có thể assert rằng một request cụ thể không được gửi sử dụng phương thức `assertNotSent`:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::fake();

Http::post('http://example.com/users', [
    'name' => 'Taylor',
    'role' => 'Developer',
]);

Http::assertNotSent(function (Request $request) {
    return $request->url() === 'http://example.com/posts';
});
```

Bạn có thể sử dụng phương thức `assertSentCount` để assert bao nhiêu requests đã được "gửi" trong test:

```php
Http::fake();

Http::assertSentCount(5);
```

Hoặc, bạn có thể sử dụng phương thức `assertNothingSent` để assert rằng không có requests nào được gửi trong test:

```php
Http::fake();

Http::assertNothingSent();
```

<a name="recording-requests-and-responses"></a>
#### Recording Requests / Responses

Bạn có thể sử dụng phương thức `recorded` để thu thập tất cả các requests và responses tương ứng của chúng. Phương thức `recorded` trả về một collection của các mảng chứa các instances của `Illuminate\Http\Client\Request` và `Illuminate\Http\Client\Response`:

```php
Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded();

[$request, $response] = $recorded[0];
```

Ngoài ra, phương thức `recorded` chấp nhận một closure sẽ nhận một instance của `Illuminate\Http\Client\Request` và `Illuminate\Http\Client\Response` và có thể được sử dụng để filter các cặp request / response dựa trên kỳ vọng của bạn:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Http\Client\Response;

Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded(function (Request $request, Response $response) {
    return $request->url() !== 'https://laravel.com' &&
           $response->successful();
});
```

<a name="preventing-stray-requests"></a>
### Preventing Stray Requests

Nếu bạn muốn đảm bảo rằng tất cả các requests được gửi qua HTTP client đã được fake trong suốt test riêng lẻ hoặc complete test suite của bạn, bạn có thể gọi phương thức `preventStrayRequests`. Sau khi gọi phương thức này, bất kỳ requests nào không có fake response tương ứng sẽ throw một exception thay vì thực hiện HTTP request thực tế:

```php
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::fake([
    'github.com/*' => Http::response('ok'),
]);

// Một response "ok" được trả về...
Http::get('https://github.com/laravel/framework');

// Một exception được thrown...
Http::get('https://laravel.com');
```

Đôi khi, bạn có thể muốn ngăn chặn hầu hết các stray requests trong khi vẫn cho phép các requests cụ thể thực thi. Để thực hiện điều này, bạn có thể truyền một mảng các URL patterns cho phương thức `allowStrayRequests`. Bất kỳ request nào khớp với một trong các patterns đã cho sẽ được cho phép, trong khi tất cả các requests khác sẽ tiếp tục throw một exception:

```php
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::allowStrayRequests([
    'http://127.0.0.1:5000/*',
]);

// Request này được thực thi...
Http::get('http://127.0.0.1:5000/generate');

// Một exception được thrown...
Http::get('https://laravel.com');
```

<a name="events"></a>
## Events

Laravel fires ba events trong quá trình gửi HTTP requests. Event `RequestSending` được fired trước khi một request được gửi, trong khi event `ResponseReceived` được fired sau khi một response được nhận cho một request cụ thể. Event `ConnectionFailed` được fired nếu không có response nào được nhận cho một request cụ thể.

Các events `RequestSending` và `ConnectionFailed` đều chứa một property public `$request` mà bạn có thể sử dụng để kiểm tra instance `Illuminate\Http\Client\Request`. Tương tự, event `ResponseReceived` chứa một property `$request` cũng như một property `$response` có thể được sử dụng để kiểm tra instance `Illuminate\Http\Client\Response`. Bạn có thể tạo [event listeners](/docs/{{version}}/events) cho các events này trong ứng dụng của mình:

```php
use Illuminate\Http\Client\Events\RequestSending;

class LogRequest
{
    /**
     * Handle the event.
     */
    public function handle(RequestSending $event): void
    {
        // $event->request ...
    }
}
```

# Error Handling

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Handling Exceptions](#handling-exceptions)
    - [Reporting Exceptions](#reporting-exceptions)
    - [Exception Log Levels](#exception-log-levels)
    - [Ignoring Exceptions by Type](#ignoring-exceptions-by-type)
    - [Rendering Exceptions](#rendering-exceptions)
    - [Reportable and Renderable Exceptions](#renderable-exceptions)
- [Throttling Reported Exceptions](#throttling-reported-exceptions)
- [HTTP Exceptions](#http-exceptions)
    - [Custom HTTP Error Pages](#custom-http-error-pages)

<a name="introduction"></a>
## Introduction

Khi bạn bắt đầu một dự án Laravel mới, xử lý lỗi và ngoại lệ đã được cấu hình cho bạn; tuy nhiên, tại bất kỳ điểm nào, bạn có thể sử dụng phương thức `withExceptions` trong `bootstrap/app.php` của ứng dụng để quản lý cách các ngoại lệ được báo cáo và hiển thị bởi ứng dụng của bạn.

Đối tượng `$exceptions` được cung cấp cho closure `withExceptions` là một instance của `Illuminate\Foundation\Configuration\Exceptions` và chịu trách nhiệm quản lý xử lý ngoại lệ trong ứng dụng của bạn. Chúng tôi sẽ đi sâu hơn vào đối tượng này trong suốt tài liệu này.

<a name="configuration"></a>
## Configuration

Tùy chọn `debug` trong file cấu hình `config/app.php` của bạn xác định bao nhiêu thông tin về một lỗi thực sự được hiển thị cho người dùng. Theo mặc định, tùy chọn này được đặt để tôn trọng giá trị của biến môi trường `APP_DEBUG`, được lưu trữ trong file `.env` của bạn.

Trong quá trình phát triển cục bộ, bạn nên đặt biến môi trường `APP_DEBUG` thành `true`.

> [!WARNING]
> Trong môi trường production của bạn, giá trị của `APP_DEBUG` luôn phải là `false`. Nếu giá trị được đặt thành `true` trong production, bạn có nguy cơ hiển thị các giá trị cấu hình nhạy cảm cho người dùng cuối của ứng dụng.

<a name="handling-exceptions"></a>
## Handling Exceptions

<a name="reporting-exceptions"></a>
### Reporting Exceptions

Trong Laravel, báo cáo ngoại lệ được sử dụng để log các ngoại lệ hoặc gửi chúng đến một dịch vụ bên ngoài như [Sentry](https://github.com/getsentry/sentry-laravel) hoặc [Flare](https://flareapp.io). Theo mặc định, các ngoại lệ sẽ được log dựa trên cấu hình [logging](/docs/{{version}}/logging) của bạn. Tuy nhiên, bạn tự do log các ngoại lệ theo cách bạn muốn.

Nếu bạn cần báo cáo các loại ngoại lệ khác nhau theo các cách khác nhau, bạn có thể sử dụng phương thức ngoại lệ `report` trong `bootstrap/app.php` của ứng dụng để đăng ký một closure nên được thực thi khi một ngoại lệ của một loại nhất định cần được báo cáo. Laravel sẽ xác định loại ngoại lệ mà closure báo cáo bằng cách kiểm tra type-hint của closure:

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

Khi bạn đăng ký một callback báo cáo ngoại lệ tùy chỉnh bằng phương thức `report`, Laravel vẫn sẽ log ngoại lệ bằng cấu hình logging mặc định cho ứng dụng. Nếu bạn muốn ngăn chặn việc lan truyền ngoại lệ đến stack logging mặc định, bạn có thể sử dụng phương thức `stop` khi định nghĩa callback báo cáo của mình hoặc trả về `false` từ callback:

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    })->stop();

    $exceptions->report(function (InvalidOrderException $e) {
        return false;
    });
})
```

> [!NOTE]
> Để tùy chỉnh báo cáo ngoại lệ cho một ngoại lệ nhất định, bạn cũng có thể sử dụng [các ngoại lệ có thể báo cáo](/docs/{{version}}/errors#renderable-exceptions).

<a name="global-log-context"></a>
#### Global Log Context

Nếu có sẵn, Laravel tự động thêm ID người dùng hiện tại vào mọi thông báo log ngoại lệ như dữ liệu ngữ cảnh. Bạn có thể định nghĩa dữ liệu ngữ cảnh toàn cầu của riêng mình bằng cách sử dụng phương thức ngoại lệ `context` trong file `bootstrap/app.php` của ứng dụng. Thông tin này sẽ được bao gồm trong mọi thông báo log ngoại lệ được viết bởi ứng dụng của bạn:

```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

<a name="exception-log-context"></a>
#### Exception Log Context

Trong khi thêm ngữ cảnh vào mọi thông báo log có thể hữu ích, đôi khi một ngoại lệ cụ thể có thể có ngữ cảnh duy nhất mà bạn muốn bao gồm trong các log của mình. Bằng cách định nghĩa một phương thức `context` trên một trong các ngoại lệ của ứng dụng, bạn có thể chỉ định bất kỳ dữ liệu nào liên quan đến ngoại lệ đó nên được thêm vào mục log ngoại lệ:

```php
<?php

namespace App\Exceptions;

use Exception;

class InvalidOrderException extends Exception
{
    // ...

    /**
     * Get the exception's context information.
     *
     * @return array<string, mixed>
     */
    public function context(): array
    {
        return ['order_id' => $this->orderId];
    }
}
```

<a name="the-report-helper"></a>
#### The `report` Helper

Đôi khi bạn cần báo cáo một ngoại lệ nhưng tiếp tục xử lý request hiện tại. Hàm helper `report` cho phép bạn nhanh chóng báo cáo một ngoại lệ mà không hiển thị một trang lỗi cho người dùng:

```php
public function isValid(string $value): bool
{
    try {
        // Validate the value...
    } catch (Throwable $e) {
        report($e);

        return false;
    }
}
```

<a name="deduplicating-reported-exceptions"></a>
#### Deduplicating Reported Exceptions

Nếu bạn sử dụng hàm `report` trong suốt ứng dụng của mình, bạn có thể đôi khi báo cáo cùng một ngoại lệ nhiều lần, tạo ra các mục trùng lặp trong các log của mình.

Nếu bạn muốn đảm bảo rằng một instance duy nhất của một ngoại lệ chỉ được báo cáo một lần, bạn có thể gọi phương thức ngoại lệ `dontReportDuplicates` trong file `bootstrap/app.php` của ứng dụng:

```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReportDuplicates();
})
```

Bây giờ, khi hàm `report` được gọi với cùng một instance của một ngoại lệ, chỉ cuộc gọi đầu tiên sẽ được báo cáo:

```php
$original = new RuntimeException('Whoops!');

report($original); // reported

try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // ignored
}

report($original); // ignored
report($caught); // ignored
```

<a name="exception-log-levels"></a>
### Exception Log Levels

Khi các thông báo được viết vào [logs](/docs/{{version}}/logging) của ứng dụng, các thông báo được viết ở một [log level](/docs/{{version}}/logging#log-levels) được chỉ định, cho biết mức độ nghiêm trọng hoặc tầm quan trọng của thông báo được log.

Như đã lưu ý ở trên, ngay cả khi bạn đăng ký một callback báo cáo ngoại lệ tùy chỉnh bằng phương thức `report`, Laravel vẫn sẽ log ngoại lệ bằng cấu hình logging mặc định cho ứng dụng; tuy nhiên, vì log level đôi khi có thể ảnh hưởng đến các channels mà một thông báo được log, bạn có thể muốn cấu hình log level mà các ngoại lệ nhất định được log.

Để thực hiện điều này, bạn có thể sử dụng phương thức ngoại lệ `level` trong file `bootstrap/app.php` của ứng dụng. Phương thức này nhận loại ngoại lệ làm đối số đầu tiên và log level làm đối số thứ hai:

```php
use PDOException;
use Psr\Log\LogLevel;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->level(PDOException::class, LogLevel::CRITICAL);
})
```

<a name="ignoring-exceptions-by-type"></a>
### Ignoring Exceptions by Type

Khi xây dựng ứng dụng của mình, sẽ có một số loại ngoại lệ mà bạn không bao giờ muốn báo cáo. Để bỏ qua các ngoại lệ này, bạn có thể sử dụng phương thức ngoại lệ `dontReport` trong file `bootstrap/app.php` của ứng dụng. Bất kỳ lớp nào được cung cấp cho phương thức này sẽ không bao giờ được báo cáo; tuy nhiên, chúng vẫn có thể có logic hiển thị tùy chỉnh:

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReport([
        InvalidOrderException::class,
    ]);
})
```

Ngoài ra, bạn có thể chỉ đơn giản là "đánh dấu" một lớp ngoại lệ với interface `Illuminate\Contracts\Debug\ShouldntReport`. Khi một ngoại lệ được đánh dấu với interface này, nó sẽ không bao giờ được báo cáo bởi exception handler của Laravel:

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Contracts\Debug\ShouldntReport;

class PodcastProcessingException extends Exception implements ShouldntReport
{
    //
}
```

Nếu bạn cần kiểm soát nhiều hơn về khi một loại ngoại lệ cụ thể bị bỏ qua, bạn có thể cung cấp một closure cho phương thức `dontReportWhen`:

```php
use App\Exceptions\InvalidOrderException;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReportWhen(function (Throwable $e) {
        return $e instanceof PodcastProcessingException &&
               $e->reason() === 'Subscription expired';
    });
})
```

Nội bộ, Laravel đã bỏ qua một số loại lỗi cho bạn, chẳng hạn như các ngoại lệ kết quả từ các lỗi HTTP 404, các phản hồi HTTP 403 được tạo ra do sự không khớp origin, hoặc các phản hồi HTTP 419 được tạo ra do các CSRF tokens không hợp lệ. Nếu bạn muốn hướng dẫn Laravel ngừng bỏ qua một loại ngoại lệ nhất định, bạn có thể sử dụng phương thức ngoại lệ `stopIgnoring` trong file `bootstrap/app.php` của ứng dụng:

```php
use Symfony\Component\HttpKernel\Exception\HttpException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->stopIgnoring(HttpException::class);
})
```

<a name="rendering-exceptions"></a>
### Rendering Exceptions

Theo mặc định, exception handler của Laravel sẽ chuyển đổi các ngoại lệ thành một phản hồi HTTP cho bạn. Tuy nhiên, bạn tự do đăng ký một closure hiển thị tùy chỉnh cho các ngoại lệ của một loại nhất định. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức ngoại lệ `render` trong file `bootstrap/app.php` của ứng dụng.

Closure được chuyển cho phương thức `render` nên trả về một instance của `Illuminate\Http\Response`, có thể được tạo thông qua helper `response`. Laravel sẽ xác định loại ngoại lệ mà closure hiển thị bằng cách kiểm tra type-hint của closure:

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', status: 500);
    });
})
```

Bạn cũng có thể sử dụng phương thức `render` để ghi đè hành vi hiển thị cho các ngoại lệ tích hợp sẵn của Laravel hoặc Symfony như `NotFoundHttpException`. Nếu closure được đưa cho phương thức `render` không trả về một giá trị, hiển thị ngoại lệ mặc định của Laravel sẽ được sử dụng:

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
})
```

<a name="rendering-exceptions-as-json"></a>
#### Rendering Exceptions as JSON

Khi hiển thị một ngoại lệ, Laravel sẽ tự động xác định xem ngoại lệ nên được hiển thị như một phản hồi HTML hay JSON dựa trên header `Accept` của request. Nếu bạn muốn tùy chỉnh cách Laravel xác định xem có hiển thị phản hồi ngoại lệ HTML hay JSON hay không, bạn có thể sử dụng phương thức `shouldRenderJsonWhen`:

```php
use Illuminate\Http\Request;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
        if ($request->is('admin/*')) {
            return true;
        }

        return $request->expectsJson();
    });
})
```

<a name="customizing-the-exception-response"></a>
#### Customizing the Exception Response

Hiếm khi, bạn có thể cần tùy chỉnh toàn bộ phản hồi HTTP được hiển thị bởi exception handler của Laravel. Để thực hiện điều này, bạn có thể đăng ký một closure tùy chỉnh phản hồi bằng cách sử dụng phương thức `respond`:

```php
use Symfony\Component\HttpFoundation\Response;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->respond(function (Response $response) {
        if ($response->getStatusCode() === 419) {
            return back()->with([
                'message' => 'The page expired, please try again.',
            ]);
        }

        return $response;
    });
})
```

<a name="renderable-exceptions"></a>
### Reportable and Renderable Exceptions

Thay vì định nghĩa hành vi báo cáo và hiển thị tùy chỉnh trong file `bootstrap/app.php` của ứng dụng, bạn có thể định nghĩa các phương thức `report` và `render` trực tiếp trên các ngoại lệ của ứng dụng. Khi các phương thức này tồn tại, chúng sẽ tự động được gọi bởi framework:

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class InvalidOrderException extends Exception
{
    /**
     * Report the exception.
     */
    public function report(): void
    {
        // ...
    }

    /**
     * Render the exception as an HTTP response.
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

Nếu ngoại lệ của bạn mở rộng một ngoại lệ đã có thể hiển thị, chẳng hạn như một ngoại lệ tích hợp sẵn của Laravel hoặc Symfony, bạn có thể trả về `false` từ phương thức `render` của ngoại lệ để hiển thị phản hồi HTTP mặc định của ngoại lệ:

```php
/**
 * Render the exception as an HTTP response.
 */
public function render(Request $request): Response|bool
{
    if (/** Determine if the exception needs custom rendering */) {

        return response(/* ... */);
    }

    return false;
}
```

Nếu ngoại lệ của bạn chứa logic báo cáo tùy chỉnh chỉ cần thiết khi một số điều kiện nhất định được đáp ứng, bạn có thể cần hướng dẫn Laravel đôi khi báo cáo ngoại lệ bằng cách sử dụng cấu hình xử lý ngoại lệ mặc định. Để thực hiện điều này, bạn có thể trả về `false` từ phương thức `report` của ngoại lệ:

```php
/**
 * Report the exception.
 */
public function report(): bool
{
    if (/** Determine if the exception needs custom reporting */) {

        // ...

        return true;
    }

    return false;
}
```

> [!NOTE]
> Bạn có thể type-hint bất kỳ dependencies nào cần thiết của phương thức `report` và chúng sẽ tự động được inject vào phương thức bởi [service container](/docs/{{version}}/container) của Laravel.

<a name="throttling-reported-exceptions"></a>
### Throttling Reported Exceptions

Nếu ứng dụng của bạn báo cáo một số lượng rất lớn các ngoại lệ, bạn có thể muốn giới hạn bao nhiêu ngoại lệ thực sự được log hoặc gửi đến dịch vụ theo dõi lỗi bên ngoài của ứng dụng.

Để lấy một tỷ lệ mẫu ngẫu nhiên của các ngoại lệ, bạn có thể sử dụng phương thức ngoại lệ `throttle` trong file `bootstrap/app.php` của ứng dụng. Phương thức `throttle` nhận một closure nên trả về một instance `Lottery`:

```php
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        return Lottery::odds(1, 1000);
    });
})
```

Cũng có thể lấy mẫu có điều kiện dựa trên loại ngoại lệ. Nếu bạn chỉ muốn lấy mẫu các instance của một lớp ngoại lệ cụ thể, bạn có thể trả về một instance `Lottery` chỉ cho lớp đó:

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof ApiMonitoringException) {
            return Lottery::odds(1, 1000);
        }
    });
})
```

Bạn cũng có thể giới hạn tốc độ các ngoại lệ được log hoặc gửi đến một dịch vụ theo dõi lỗi bên ngoài bằng cách trả về một instance `Limit` thay vì một `Lottery`. Điều này hữu ích nếu bạn muốn bảo vệ chống lại các đợt đột biến đột ngột của các ngoại lệ làm đầy các log của bạn, ví dụ, khi một dịch vụ bên thứ ba được sử dụng bởi ứng dụng của bạn bị lỗi:

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300);
        }
    });
})
```

Theo mặc định, các giới hạn sẽ sử dụng lớp của ngoại lệ làm key giới hạn tốc độ. Bạn có thể tùy chỉnh điều này bằng cách chỉ định key của riêng bạn bằng cách sử dụng phương thức `by` trên `Limit`:

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300)->by($e->getMessage());
        }
    });
})
```

Tất nhiên, bạn có thể trả về một hỗn hợp các instance `Lottery` và `Limit` cho các ngoại lệ khác nhau:

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        return match (true) {
            $e instanceof BroadcastException => Limit::perMinute(300),
            $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
            default => Limit::none(),
        };
    });
})
```

<a name="http-exceptions"></a>
## HTTP Exceptions

Một số ngoại lệ mô tả các mã lỗi HTTP từ server. Ví dụ, điều này có thể là một lỗi "page not found" (404), một lỗi "unauthorized error" (401), hoặc thậm chí một lỗi 500 được tạo bởi nhà phát triển. Để tạo ra một phản hồi như vậy từ bất cứ đâu trong ứng dụng của bạn, bạn có thể sử dụng helper `abort`:

```php
abort(404);
```

<a name="custom-http-error-pages"></a>
### Custom HTTP Error Pages

Laravel giúp dễ dàng hiển thị các trang lỗi tùy chỉnh cho các mã trạng thái HTTP khác nhau. Ví dụ, để tùy chỉnh trang lỗi cho các mã trạng thái HTTP 404, hãy tạo một view template `resources/views/errors/404.blade.php`. View này sẽ được hiển thị cho tất cả các lỗi 404 được tạo bởi ứng dụng của bạn. Các view trong thư mục này nên được đặt tên để khớp với mã trạng thái HTTP mà chúng tương ứng. Instance `Symfony\Component\HttpKernel\Exception\HttpException` được tạo ra bởi hàm `abort` sẽ được chuyển cho view như một biến `$exception`:

```blade
<h2>{{ $exception->getMessage() }}</h2>
```

Bạn có thể publish các template trang lỗi mặc định của Laravel bằng cách sử dụng lệnh Artisan `vendor:publish`. Sau khi các template đã được publish, bạn có thể tùy chỉnh chúng theo ý thích của mình:

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### Fallback HTTP Error Pages

Bạn cũng có thể định nghĩa một trang lỗi "fallback" cho một chuỗi mã trạng thái HTTP nhất định. Trang này sẽ được hiển thị nếu không có trang tương ứng cho mã trạng thái HTTP cụ thể đã xảy ra. Để thực hiện điều này, hãy định nghĩa một template `4xx.blade.php` và một template `5xx.blade.php` trong thư mục `resources/views/errors` của ứng dụng.

Khi định nghĩa các trang lỗi fallback, các trang fallback sẽ không ảnh hưởng đến các phản hồi lỗi `404`, `500`, và `503` vì Laravel có các trang nội bộ, chuyên dụng cho các mã trạng thái này. Để tùy chỉnh các trang được hiển thị cho các mã trạng thái này, bạn nên định nghĩa một trang lỗi tùy chỉnh cho từng cái một cách riêng lẻ.

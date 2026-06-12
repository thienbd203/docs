# Context

- [Giới thiệu](#introduction)
    - [Cách hoạt động](#how-it-works)
- [Thu thập Context](#capturing-context)
    - [Stacks](#stacks)
- [Truy xuất Context](#retrieving-context)
    - [Xác định sự tồn tại của mục](#determining-item-existence)
- [Xóa Context](#removing-context)
- [Context ẩn](#hidden-context)
- [Sự kiện](#events)
    - [Dehydrating](#dehydrating)
    - [Hydrated](#hydrated)

<a name="introduction"></a>
## Giới thiệu

Khả năng "context" của Laravel cho phép bạn thu thập, truy xuất và chia sẻ thông tin trên toàn bộ các request, job và command đang thực thi trong ứng dụng của bạn. Thông tin được thu thập này cũng được bao gồm trong các log được ghi bởi ứng dụng của bạn, mang lại cho bạn cái nhìn sâu sắc hơn về lịch sử thực thi mã xung quanh đã xảy ra trước khi một mục log được ghi và cho phép bạn theo dõi các luồng thực thi trong toàn bộ hệ thống phân tán.

<a name="how-it-works"></a>
### Cách hoạt động

Cách tốt nhất để hiểu khả năng context của Laravel là xem nó hoạt động thực tế bằng cách sử dụng các tính năng logging tích hợp. Để bắt đầu, bạn có thể [thêm thông tin vào context](#capturing-context) bằng facade `Context`. Trong ví dụ này, chúng ta sẽ sử dụng [middleware](/docs/{{version}}/middleware) để thêm URL của request và một trace ID duy nhất vào context trên mọi request đến:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AddContext
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('trace_id', Str::uuid()->toString());

        return $next($request);
    }
}
```

Thông tin được thêm vào context sẽ tự động được thêm vào dưới dạng metadata cho bất kỳ [mục log nào](/docs/{{version}}/logging) được ghi trong suốt request. Việc thêm context dưới dạng metadata cho phép thông tin được truyền cho các mục log riêng biệt được phân biệt với thông tin được chia sẻ qua `Context`. Ví dụ, hãy tưởng tượng chúng ta ghi mục log sau:

```php
Log::info('User authenticated.', ['auth_id' => Auth::id()]);
```

Log được ghi sẽ chứa `auth_id` được truyền cho mục log, nhưng nó cũng sẽ chứa `url` và `trace_id` của context dưới dạng metadata:

```text
User authenticated. {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

Thông tin được thêm vào context cũng được cung cấp cho các job được gửi đến hàng đợi. Ví dụ, hãy tưởng tượng chúng ta gửi job `ProcessPodcast` đến hàng đợi sau khi thêm một số thông tin vào context:

```php
// In our middleware...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());

// In our controller...
ProcessPodcast::dispatch($podcast);
```

Khi job được gửi, bất kỳ thông tin nào hiện được lưu trữ trong context sẽ được thu thập và chia sẻ với job. Thông tin được thu thập sau đó được hydrate trở lại vào context hiện tại trong khi job đang thực thi. Vì vậy, nếu phương thức handle của job của chúng ta ghi vào log:

```php
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Log::info('Processing podcast.', [
            'podcast_id' => $this->podcast->id,
        ]);

        // ...
    }
}
```

Mục log kết quả sẽ chứa thông tin đã được thêm vào context trong request ban đầu đã gửi job:

```text
Processing podcast. {"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

Mặc dù chúng ta đã tập trung vào các tính năng liên quan đến logging tích hợp của context của Laravel, tài liệu sau sẽ minh họa cách context cho phép bạn chia sẻ thông tin qua ranh giới HTTP request / queued job và thậm chí cách thêm [dữ liệu context ẩn](#hidden-context) không được ghi cùng với các mục log.

<a name="capturing-context"></a>
## Thu thập Context

Bạn có thể lưu trữ thông tin trong context hiện tại bằng phương thức `add` của facade `Context`:

```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');
```

Để thêm nhiều mục cùng một lúc, bạn có thể truyền một mảng kết hợp cho phương thức `add`:

```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```

Phương thức `add` sẽ ghi đè bất kỳ giá trị hiện có nào chia sẻ cùng một khóa. Nếu bạn chỉ muốn thêm thông tin vào context nếu khóa chưa tồn tại, bạn có thể sử dụng phương thức `addIf`:

```php
Context::add('key', 'first');

Context::get('key');
// "first"

Context::addIf('key', 'second');

Context::get('key');
// "first"
```

Context cũng cung cấp các phương thức tiện lợi để tăng hoặc giảm một khóa nhất định. Cả hai phương thức này đều chấp nhận ít nhất một đối số: khóa để theo dõi. Một đối số thứ hai có thể được cung cấp để chỉ định số lượng mà khóa nên được tăng hoặc giảm:

```php
Context::increment('records_added');
Context::increment('records_added', 5);

Context::decrement('records_added');
Context::decrement('records_added', 5);
```

<a name="conditional-context"></a>
#### Context có điều kiện

Phương thức `when` có thể được sử dụng để thêm dữ liệu vào context dựa trên một điều kiện nhất định. Closure đầu tiên được cung cấp cho phương thức `when` sẽ được gọi nếu điều kiện đã cho đánh giá là `true`, trong khi closure thứ hai sẽ được gọi nếu điều kiện đánh giá là `false`:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;

Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

<a name="scoped-context"></a>
#### Context có phạm vi

Phương thức `scope` cung cấp một cách để tạm thời sửa đổi context trong quá trình thực thi một callback nhất định và khôi phục context về trạng thái ban đầu khi callback hoàn thành thực thi. Ngoài ra, bạn có thể truyền thêm dữ liệu nên được hợp nhất vào context (làm đối số thứ hai và thứ ba) trong khi closure thực thi.

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\Log;

Context::add('trace_id', 'abc-999');
Context::addHidden('user_id', 123);

Context::scope(
    function () {
        Context::add('action', 'adding_friend');

        $userId = Context::getHidden('user_id');

        Log::debug("Adding user [{$userId}] to friends list.");
        // Adding user [987] to friends list.  {"trace_id":"abc-999","user_name":"taylor_otwell","action":"adding_friend"}
    },
    data: ['user_name' => 'taylor_otwell'],
    hidden: ['user_id' => 987],
);

Context::all();
// [
//     'trace_id' => 'abc-999',
// ]

Context::allHidden();
// [
//     'user_id' => 123,
// ]
```

> [!WARNING]
> Nếu một đối tượng trong context được sửa đổi bên trong closure có phạm vi, sự thay đổi đó sẽ được phản ánh bên ngoài phạm vi.

<a name="stacks"></a>
### Stacks

Context cung cấp khả năng tạo "stacks", là danh sách dữ liệu được lưu trữ theo thứ tự chúng được thêm. Bạn có thể thêm thông tin vào stack bằng cách gọi phương thức `push`:

```php
use Illuminate\Support\Facades\Context;

Context::push('breadcrumbs', 'first_value');

Context::push('breadcrumbs', 'second_value', 'third_value');

Context::get('breadcrumbs');
// [
//     'first_value',
//     'second_value',
//     'third_value',
// ]
```

Stacks có thể hữu ích để thu thập thông tin lịch sử về một request, chẳng hạn như các sự kiện đang diễn ra trong toàn bộ ứng dụng của bạn. Ví dụ, bạn có thể tạo một event listener để đẩy vào stack mỗi khi một query được thực thi, thu thập query SQL và thời lượng dưới dạng một tuple:

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;

// In AppServiceProvider.php...
DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```

Bạn có thể xác định xem một giá trị có trong stack hay không bằng cách sử dụng các phương thức `stackContains` và `hiddenStackContains`:

```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}

if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```

Các phương thức `stackContains` và `hiddenStackContains` cũng chấp nhận một closure làm đối số thứ hai, cho phép kiểm soát nhiều hơn đối với thao tác so sánh giá trị:

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;

return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

<a name="retrieving-context"></a>
## Truy xuất Context

Bạn có thể truy xuất thông tin từ context bằng phương thức `get` của facade `Context`:

```php
use Illuminate\Support\Facades\Context;

$value = Context::get('key');
```

Các phương thức `only` và `except` có thể được sử dụng để truy xuất một tập con của thông tin trong context:

```php
$data = Context::only(['first_key', 'second_key']);

$data = Context::except(['first_key']);
```

Phương thức `pull` có thể được sử dụng để truy xuất thông tin từ context và ngay lập tức xóa nó khỏi context:

```php
$value = Context::pull('key');
```

Nếu dữ liệu context được lưu trữ trong một [stack](#stacks), bạn có thể pop các mục khỏi stack bằng phương thức `pop`:

```php
Context::push('breadcrumbs', 'first_value', 'second_value');

Context::pop('breadcrumbs');
// second_value

Context::get('breadcrumbs');
// ['first_value']
```

Các phương thức `remember` và `rememberHidden` có thể được sử dụng để truy xuất thông tin từ context, trong khi đặt giá trị context thành giá trị được trả về bởi closure đã cho nếu thông tin được yêu cầu không tồn tại:

```php
$permissions = Context::remember(
    'user-permissions',
    fn () => $user->permissions,
);
```

Nếu bạn muốn truy xuất tất cả thông tin được lưu trữ trong context, bạn có thể gọi phương thức `all`:

```php
$data = Context::all();
```

<a name="determining-item-existence"></a>
### Xác định sự tồn tại của mục

Bạn có thể sử dụng các phương thức `has` và `missing` để xác định xem context có bất kỳ giá trị nào được lưu trữ cho khóa đã cho hay không:

```php
use Illuminate\Support\Facades\Context;

if (Context::has('key')) {
    // ...
}

if (Context::missing('key')) {
    // ...
}
```

Phương thức `has` sẽ trả về `true` bất kể giá trị được lưu trữ. Vì vậy, ví dụ, một khóa có giá trị `null` sẽ được coi là hiện diện:

```php
Context::add('key', null);

Context::has('key');
// true
```

<a name="removing-context"></a>
## Xóa Context

Phương thức `forget` có thể được sử dụng để xóa một khóa và giá trị của nó khỏi context hiện tại:

```php
use Illuminate\Support\Facades\Context;

Context::add(['first_key' => 1, 'second_key' => 2]);

Context::forget('first_key');

Context::all();

// ['second_key' => 2]
```

Bạn có thể quên nhiều khóa cùng một lúc bằng cách cung cấp một mảng cho phương thức `forget`:

```php
Context::forget(['first_key', 'second_key']);
```

<a name="hidden-context"></a>
## Context ẩn

Context cung cấp khả năng lưu trữ dữ liệu "ẩn". Thông tin ẩn này không được thêm vào log và không thể truy cập thông qua các phương thức truy xuất dữ liệu được tài liệu hóa ở trên. Context cung cấp một tập hợp các phương thức khác nhau để tương tác với thông tin context ẩn:

```php
use Illuminate\Support\Facades\Context;

Context::addHidden('key', 'value');

Context::getHidden('key');
// 'value'

Context::get('key');
// null
```

Các phương thức "ẩn" phản ánh chức năng của các phương thức không ẩn được tài liệu hóa ở trên:

```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::popHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::exceptHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::missingHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

<a name="events"></a>
## Sự kiện

Context gửi hai sự kiện cho phép bạn hook vào quá trình hydrate và dehydrate của context.

Để minh họa cách các sự kiện này có thể được sử dụng, hãy tưởng tượng rằng trong một middleware của ứng dụng của bạn, bạn đặt giá trị cấu hình `app.locale` dựa trên header `Accept-Language` của HTTP request đến. Các sự kiện của context cho phép bạn thu thập giá trị này trong quá trình request và khôi phục nó trên hàng đợi, đảm bảo các thông báo được gửi trên hàng đợi có giá trị `app.locale` chính xác. Chúng ta có thể sử dụng các sự kiện của context và dữ liệu [ẩn](#hidden-context) để đạt được điều này, mà tài liệu sau sẽ minh họa.

<a name="dehydrating"></a>
### Dehydrating

Bất cứ khi nào một job được gửi đến hàng đợi, dữ liệu trong context sẽ được "dehydrate" và thu thập cùng với payload của job. Phương thức `Context::dehydrating` cho phép bạn đăng ký một closure sẽ được gọi trong quá trình dehydrate. Trong closure này, bạn có thể thực hiện thay đổi đối với dữ liệu sẽ được chia sẻ với queued job.

Thông thường, bạn nên đăng ký các callback `dehydrating` trong phương thức `boot` của lớp `AppServiceProvider` của ứng dụng:

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::dehydrating(function (Repository $context) {
        $context->addHidden('locale', Config::get('app.locale'));
    });
}
```

> [!NOTE]
> Bạn không nên sử dụng facade `Context` trong callback `dehydrating`, vì điều đó sẽ thay đổi context của quá trình hiện tại. Đảm bảo bạn chỉ thực hiện thay đổi đối với repository được truyền cho callback.

<a name="hydrated"></a>
### Hydrated

Bất cứ khi nào một queued job bắt đầu thực thi trên hàng đợi, bất kỳ context nào được chia sẻ với job sẽ được "hydrate" trở lại vào context hiện tại. Phương thức `Context::hydrated` cho phép bạn đăng ký một closure sẽ được gọi trong quá trình hydrate.

Thông thường, bạn nên đăng ký các callback `hydrated` trong phương thức `boot` của lớp `AppServiceProvider` của ứng dụng:

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::hydrated(function (Repository $context) {
        if ($context->hasHidden('locale')) {
            Config::set('app.locale', $context->getHidden('locale'));
        }
    });
}
```

> [!NOTE]
> Bạn không nên sử dụng facade `Context` trong callback `hydrated` và thay vào đó đảm bảo bạn chỉ thực hiện thay đổi đối với repository được truyền cho callback.

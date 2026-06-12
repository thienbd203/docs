# HTTP Responses

- [Tạo Responses](#creating-responses)
    - [Gắn Headers vào Responses](#attaching-headers-to-responses)
    - [Gắn Cookies vào Responses](#attaching-cookies-to-responses)
    - [Cookies và Encryption](#cookies-and-encryption)
- [Redirects](#redirects)
    - [Redirect đến Named Routes](#redirecting-named-routes)
    - [Redirect đến Controller Actions](#redirecting-controller-actions)
    - [Redirect đến External Domains](#redirecting-external-domains)
    - [Redirect Với Flashed Session Data](#redirecting-with-flashed-session-data)
- [Các Loại Response Khác](#other-response-types)
    - [View Responses](#view-responses)
    - [JSON Responses](#json-responses)
    - [File Downloads](#file-downloads)
    - [File Responses](#file-responses)
- [Streamed Responses](#streamed-responses)
    - [Consuming Streamed Responses](#consuming-streamed-responses)
    - [Streamed JSON Responses](#streamed-json-responses)
    - [Event Streams (SSE)](#event-streams)
    - [Streamed Downloads](#streamed-downloads)
- [Response Macros](#response-macros)

<a name="creating-responses"></a>
## Tạo Responses

<a name="strings-arrays"></a>
#### Strings và Arrays

Tất cả routes và controllers nên trả về một response để gửi trở lại browser của user. Laravel cung cấp nhiều cách khác nhau để trả về responses. Response cơ bản nhất là trả về một string từ một route hoặc controller. Framework sẽ tự động chuyển đổi string thành một HTTP response đầy đủ:

```php
Route::get('/', function () {
    return 'Hello World';
});
```

Ngoài việc trả về strings từ routes và controllers của bạn, bạn cũng có thể trả về arrays. Framework sẽ tự động chuyển đổi array thành một JSON response:

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

> [!NOTE]
> Bạn có biết bạn cũng có thể trả về [Eloquent collections](/docs/{{version}}/eloquent-collections) từ routes hoặc controllers của bạn không? Chúng sẽ tự động được chuyển đổi thành JSON. Hãy thử xem!

<a name="response-objects"></a>
#### Response Objects

Thông thường, bạn sẽ không chỉ trả về các strings hoặc arrays đơn giản từ route actions của bạn. Thay vào đó, bạn sẽ trả về các instances `Illuminate\Http\Response` đầy đủ hoặc [views](/docs/{{version}}/views).

Trả về một instance `Response` đầy đủ cho phép bạn customize HTTP status code và headers của response. Một instance `Response` kế thừa từ class `Symfony\Component\HttpFoundation\Response`, cung cấp nhiều phương thức để xây dựng HTTP responses:

```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Eloquent Models và Collections

Bạn cũng có thể trả về [Eloquent ORM](/docs/{{version}}/eloquent) models và collections trực tiếp từ routes và controllers của bạn. Khi bạn làm như vậy, Laravel sẽ tự động chuyển đổi models và collections thành JSON responses trong khi tôn trọng [hidden attributes](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) của model:

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

<a name="attaching-headers-to-responses"></a>
### Gắn Headers vào Responses

Hãy nhớ rằng hầu hết các response methods đều có thể chain, cho phép xây dựng fluent các response instances. Ví dụ, bạn có thể sử dụng phương thức `header` để thêm một loạt headers vào response trước khi gửi trở lại user:

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

Hoặc, bạn có thể sử dụng phương thức `withHeaders` để chỉ định một mảng các headers để thêm vào response:

```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

Bạn có thể remove các headers cụ thể từ một outgoing response sử dụng phương thức `withoutHeader`:

```php
return response($content)->withoutHeader('X-Debug');

return response($content)->withoutHeader(['X-Debug', 'X-Powered-By']);
```

<a name="cache-control-middleware"></a>
#### Cache Control Middleware

Laravel bao gồm một middleware `cache.headers`, có thể được sử dụng để nhanh chóng set header `Cache-Control` cho một nhóm routes. Directives nên được cung cấp sử dụng "snake case" tương đương của cache-control directive tương ứng và nên được phân tách bằng dấu chấm phẩy. Nếu `etag` được chỉ định trong danh sách directives, một MD5 hash của response content sẽ tự động được set làm ETag identifier:

```php
Route::middleware('cache.headers:public;max_age=30;s_maxage=300;stale_while_revalidate=600;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

<a name="attaching-cookies-to-responses"></a>
### Gắn Cookies vào Responses

Bạn có thể gắn một cookie vào một instance `Illuminate\Http\Response` outgoing sử dụng phương thức `cookie`. Bạn nên truyền tên, giá trị, và số phút cookie nên được coi là hợp lệ cho phương thức này:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

Phương thức `cookie` cũng chấp nhận một vài đối số khác được sử dụng ít thường xuyên hơn. Nói chung, các đối số này có cùng mục đích và ý nghĩa như các đối số sẽ được đưa cho phương thức [setcookie](https://secure.php.net/manual/en/function.setcookie.php) native của PHP:

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

Nếu bạn muốn đảm bảo rằng một cookie được gửi với outgoing response nhưng bạn chưa có instance của response đó, bạn có thể sử dụng `Cookie` facade để "queue" cookies để gắn vào response khi nó được gửi. Phương thức `queue` chấp nhận các đối số cần thiết để tạo một cookie instance. Các cookies này sẽ được gắn vào outgoing response trước khi nó được gửi đến browser:

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### Tạo Cookie Instances

Nếu bạn muốn tạo một instance `Symfony\Component\HttpFoundation\Cookie` có thể được gắn vào một response instance tại một thời điểm sau, bạn có thể sử dụng global `cookie` helper. Cookie này sẽ không được gửi trở lại client trừ khi nó được gắn vào một response instance:

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### Hết hạn Cookies Sớm

Bạn có thể remove một cookie bằng cách hết hạn nó thông qua phương thức `withoutCookie` của một outgoing response:

```php
return response('Hello World')->withoutCookie('name');
```

Nếu bạn chưa có instance của outgoing response, bạn có thể sử dụng phương thức `expire` của `Cookie` facade để hết hạn một cookie:

```php
Cookie::expire('name');
```

<a name="cookies-and-encryption"></a>
### Cookies và Encryption

Theo mặc định, nhờ middleware `Illuminate\Cookie\Middleware\EncryptCookies`, tất cả cookies được tạo bởi Laravel đều được encrypted và signed để chúng không thể được sửa đổi hoặc đọc bởi client. Nếu bạn muốn vô hiệu hóa encryption cho một subset của cookies được tạo bởi ứng dụng của bạn, bạn có thể sử dụng phương thức `encryptCookies` trong file `bootstrap/app.php` của ứng dụng của bạn:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

> [!NOTE]
> Nói chung, cookie encryption không bao giờ nên được vô hiệu hóa, vì điều này phơi bày cookies của bạn cho potential client-side data exposure và tampering.

<a name="redirects"></a>
## Redirects

Redirect responses là instances của class `Illuminate\Http\RedirectResponse`, và chứa các headers cần thiết để redirect user đến một URL khác. Có một số cách để tạo một instance `RedirectResponse`. Phương thức đơn giản nhất là sử dụng global `redirect` helper:

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

Đôi khi bạn có thể muốn redirect user đến vị trí trước của họ, chẳng hạn như khi một submitted form không hợp lệ. Bạn có thể làm như vậy bằng cách sử dụng global `back` helper function. Vì tính năng này sử dụng [session](/docs/{{version}}/session), hãy đảm bảo route gọi hàm `back` đang sử dụng middleware group `web`:

```php
Route::post('/user/profile', function () {
    // Validate the request...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
### Redirect đến Named Routes

Khi bạn gọi `redirect` helper mà không có tham số, một instance của `Illuminate\Routing\Redirector` được trả về, cho phép bạn gọi bất kỳ phương thức nào trên instance `Redirector`. Ví dụ, để tạo một `RedirectResponse` đến một named route, bạn có thể sử dụng phương thức `route`:

```php
return redirect()->route('login');
```

Nếu route của bạn có parameters, bạn có thể truyền chúng làm tham số thứ hai cho phương thức `route`:

```php
// For a route with the following URI: /profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### Populating Parameters qua Eloquent Models

Nếu bạn đang redirect đến một route với một parameter "ID" đang được populate từ một Eloquent model, bạn có thể truyền chính model đó. ID sẽ được tự động extract:

```php
// For a route with the following URI: /profile/{id}

return redirect()->route('profile', [$user]);
```

Nếu bạn muốn customize giá trị được đặt trong route parameter, bạn có thể chỉ định column trong route parameter definition (`/profile/{id:slug}`) hoặc bạn có thể override phương thức `getRouteKey` trên Eloquent model của bạn:

```php
/**
 * Get the value of the model's route key.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
### Redirect đến Controller Actions

Bạn cũng có thể tạo redirects đến [controller actions](/docs/{{version}}/controllers). Để làm như vậy, truyền controller và action name cho phương thức `action`:

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

Nếu controller route của bạn yêu cầu parameters, bạn có thể truyền chúng làm tham số thứ hai cho phương thức `action`:

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-external-domains"></a>
### Redirect đến External Domains

Đôi khi bạn có thể cần redirect đến một domain bên ngoài ứng dụng của bạn. Bạn có thể làm như vậy bằng cách gọi phương thức `away`, tạo một `RedirectResponse` mà không có bất kỳ URL encoding, validation, hoặc verification bổ sung nào:

```php
return redirect()->away('https://www.google.com');
```

<a name="redirecting-with-flashed-session-data"></a>
### Redirect Với Flashed Session Data

Redirect đến một URL mới và [flashing data vào session](/docs/{{version}}/session#flash-data) thường được thực hiện cùng lúc. Thông thường, điều này được thực hiện sau khi thực hiện thành công một action khi bạn flash một success message vào session. Để thuận tiện, bạn có thể tạo một instance `RedirectResponse` và flash data vào session trong một single, fluent method chain:

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

Sau khi user được redirect, bạn có thể hiển thị flashed message từ [session](/docs/{{version}}/session). Ví dụ, sử dụng [Blade syntax](/docs/{{version}}/blade):

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="redirecting-with-input"></a>
#### Redirect Với Input

Bạn có thể sử dụng phương thức `withInput` được cung cấp bởi instance `RedirectResponse` để flash input data của request hiện tại vào session trước khi redirect user đến một location mới. Điều này thường được thực hiện nếu user gặp validation error. Khi input đã được flash vào session, bạn có thể dễ dàng [lấy nó](/docs/{{version}}/requests#retrieving-old-input) trong request tiếp theo để repopulate form:

```php
return back()->withInput();
```

<a name="other-response-types"></a>
## Các Loại Response Khác

`response` helper có thể được sử dụng để tạo các loại response instances khác. Khi `response` helper được gọi mà không có đối số, một implementation của [contract](/docs/{{version}}/contracts) `Illuminate\Contracts\Routing\ResponseFactory` được trả về. Contract này cung cấp một số phương thức hữu ích để tạo responses.

<a name="view-responses"></a>
### View Responses

Nếu bạn cần kiểm soát status và headers của response nhưng cũng cần trả về một [view](/docs/{{version}}/views) làm content của response, bạn nên sử dụng phương thức `view`:

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

Tất nhiên, nếu bạn không cần truyền một custom HTTP status code hoặc custom headers, bạn có thể sử dụng global `view` helper function.

<a name="json-responses"></a>
### JSON Responses

Phương thức `json` sẽ tự động set header `Content-Type` thành `application/json`, cũng như chuyển đổi array đã cho thành JSON sử dụng hàm PHP `json_encode`:

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

Nếu bạn muốn tạo một JSONP response, bạn có thể sử dụng phương thức `json` kết hợp với phương thức `withCallback`:

```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### File Downloads

Phương thức `download` có thể được sử dụng để tạo một response buộc browser của user download file tại path đã cho. Phương thức `download` chấp nhận một filename làm tham số thứ hai cho phương thức, sẽ xác định filename được user nhìn thấy khi download file. Cuối cùng, bạn có thể truyền một mảng HTTP headers làm tham số thứ ba cho phương thức:

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> [!WARNING]
> Symfony HttpFoundation, quản lý file downloads, yêu cầu file đang được download có một ASCII filename.

<a name="file-responses"></a>
### File Responses

Phương thức `file` có thể được sử dụng để hiển thị một file, chẳng hạn như image hoặc PDF, trực tiếp trong browser của user thay vì initiate một download. Phương thức này chấp nhận absolute path đến file làm đối số thứ nhất và một mảng headers làm đối số thứ hai:

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

<a name="streamed-responses"></a>
## Streamed Responses

Bằng cách streaming data đến client khi nó được tạo, bạn có thể giảm đáng kể memory usage và cải thiện performance, đặc biệt cho các responses rất lớn. Streamed responses cho phép client bắt đầu xử lý data trước khi server đã finished gửi nó:

```php
Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (['developer', 'admin'] as $string) {
            echo $string;
            ob_flush();
            flush();
            sleep(2); // Simulate delay between chunks...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

Để thuận tiện, nếu closure bạn cung cấp cho phương thức `stream` trả về một [Generator](https://www.php.net/manual/en/language.generators.overview.php), Laravel sẽ tự động flush output buffer giữa các strings được trả về bởi generator, cũng như vô hiệu hóa Nginx output buffering:

```php
Route::post('/chat', function () {
    return response()->stream(function (): Generator {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

<a name="consuming-streamed-responses"></a>
### Consuming Streamed Responses

Streamed responses có thể được consumed sử dụng npm package `stream` của Laravel, cung cấp một API thuận tiện để tương tác với Laravel response và event streams. Để bắt đầu, cài đặt package `@laravel/stream-react`, `@laravel/stream-vue`, hoặc `@laravel/stream-svelte`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

```shell tab=Svelte
npm install @laravel/stream-svelte
```

Sau đó, `useStream` có thể được sử dụng để consume event stream. Sau khi cung cấp stream URL của bạn, hook sẽ tự động update `data` với response được nối khi content được trả về từ ứng dụng Laravel của bạn:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, isFetching, isStreaming, send } = useStream("chat");

    const sendMessage = () => {
        send({
            message: `Current timestamp: ${Date.now()}`,
        });
    };

    return (
        <div>
            <div>{data}</div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
            <button onClick={sendMessage}>Send Message</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, isFetching, isStreaming, send } = useStream("chat");

const sendMessage = () => {
    send({
        message: `Current timestamp: ${Date.now()}`,
    });
};
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
        <button @click="sendMessage">Send Message</button>
    </div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat");

const sendMessage = () => {
    stream.send({
        message: `Current timestamp: ${Date.now()}`,
    });
};
</script>

<div>
    <div>{$stream.data}</div>
    {#if $stream.isFetching}
        <div>Connecting...</div>
    {/if}
    {#if $stream.isStreaming}
        <div>Generating...</div>
    {/if}
    <button onclick={sendMessage}>Send Message</button>
</div>
```

Khi gửi data trở lại stream thông qua `send`, kết nối active đến stream bị hủy trước khi gửi data mới. Tất cả requests được gửi dưới dạng JSON `POST` requests.

> [!WARNING]
> Vì hook `useStream` tạo một `POST` request đến ứng dụng của bạn, một CSRF token hợp lệ được yêu cầu. Cách dễ nhất để cung cấp CSRF token là [include nó thông qua một meta tag trong head của layout ứng dụng của bạn](/docs/{{version}}/csrf#csrf-x-csrf-token).

Đối số thứ hai được đưa cho `useStream` là một options object mà bạn có thể sử dụng để customize hành vi stream consumption. Các giá trị mặc định cho object này được hiển thị bên dưới:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        id: undefined,
        initialInput: undefined,
        headers: undefined,
        csrfToken: undefined,
        onResponse: (response: Response) => void,
        onData: (data: string) => void,
        onCancel: () => void,
        onFinish: () => void,
        onError: (error: Error) => void,
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    id: undefined,
    initialInput: undefined,
    headers: undefined,
    csrfToken: undefined,
    onResponse: (response: Response) => void,
    onData: (data: string) => void,
    onCancel: () => void,
    onFinish: () => void,
    onError: (error: Error) => void,
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat", {
    id: undefined,
    initialInput: undefined,
    headers: undefined,
    csrfToken: undefined,
    onResponse: (response) => {},
    onData: (data) => {},
    onCancel: () => {},
    onFinish: () => {},
    onError: (error) => {},
});
</script>

<div>{$stream.data}</div>
```

`onResponse` được triggered sau một response khởi tạo thành công từ stream và raw [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) được truyền cho callback. `onData` được gọi khi mỗi chunk được nhận - chunk hiện tại được truyền cho callback. `onFinish` được gọi khi một stream đã finished và khi một error được thrown trong fetch / read cycle.

Theo mặc định, một request không được thực hiện đến stream khi khởi tạo. Bạn có thể truyền một initial payload cho stream bằng cách sử dụng option `initialInput`:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        initialInput: {
            message: "Introduce yourself.",
        },
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    initialInput: {
        message: "Introduce yourself.",
    },
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat", {
    initialInput: {
        message: "Introduce yourself.",
    },
});
</script>

<div>{$stream.data}</div>
```

Để hủy một stream thủ công, bạn có thể sử dụng phương thức `cancel` được trả về từ hook:

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, cancel } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <button onClick={cancel}>Cancel</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, cancel } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <button @click="cancel">Cancel</button>
    </div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat");
</script>

<div>
    <div>{$stream.data}</div>
    <button onclick={() => stream.cancel()}>Cancel</button>
</div>
```

Mỗi lần hook `useStream` được sử dụng, một `id` ngẫu nhiên được generated để xác định stream. Điều này được gửi trở lại server với mỗi request trong header `X-STREAM-ID`. Khi consuming cùng một stream từ nhiều components, bạn có thể đọc và viết đến stream bằng cách cung cấp `id` của riêng bạn:

```tsx tab=React
// App.tsx
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, id } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <StreamStatus id={id} />
        </div>
    );
}

// StreamStatus.tsx
import { useStream } from "@laravel/stream-react";

function StreamStatus({ id }) {
    const { isFetching, isStreaming } = useStream("chat", { id });

    return (
        <div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
        </div>
    );
}
```

```vue tab=Vue
<!-- App.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
import StreamStatus from "./StreamStatus.vue";

const { data, id } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <StreamStatus :id="id" />
    </div>
</template>

<!-- StreamStatus.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const props = defineProps<{
    id: string;
}>();

const { isFetching, isStreaming } = useStream("chat", { id: props.id });
</script>

<template>
    <div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
    </div>
</template>
```

```svelte tab=Svelte
<!-- App.svelte -->
<script>
import { useStream } from "@laravel/stream-svelte";
import StreamStatus from "./StreamStatus.svelte";

const stream = useStream("chat");
</script>

<div>
    <div>{$stream.data}</div>
    <StreamStatus id={stream.id} />
</div>

<!-- StreamStatus.svelte -->
<script>
import { useStream } from "@laravel/stream-svelte";

let { id } = $props();

const stream = useStream("chat", { id });
</script>

<div>
    {#if $stream.isFetching}
        <div>Connecting...</div>
    {/if}
    {#if $stream.isStreaming}
        <div>Generating...</div>
    {/if}
</div>
```

<a name="streamed-json-responses"></a>
### Streamed JSON Responses

Nếu bạn cần stream JSON data incrementally, bạn có thể sử dụng phương thức `streamJson`. Phương thức này đặc biệt hữu ích cho các datasets lớn cần được gửi progressively đến browser trong một format có thể dễ dàng được parse bởi JavaScript:

```php
use App\Models\User;

Route::get('/users.json', function () {
    return response()->streamJson([
        'users' => User::cursor(),
    ]);
});
```

Hook `useJsonStream` giống hệt với [useStream hook](#consuming-streamed-responses) ngoại trừ việc nó sẽ cố gắng parse data thành JSON khi nó đã finished streaming:

```tsx tab=React
import { useJsonStream } from "@laravel/stream-react";

type User = {
    id: number;
    name: string;
    email: string;
};

function App() {
    const { data, send } = useJsonStream<{ users: User[] }>("users");

    const loadUsers = () => {
        send({
            query: "taylor",
        });
    };

    return (
        <div>
            <ul>
                {data?.users.map((user) => (
                    <li>
                        {user.id}: {user.name}
                    </li>
                ))}
            </ul>
            <button onClick={loadUsers}>Load Users</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useJsonStream } from "@laravel/stream-vue";

type User = {
    id: number;
    name: string;
    email: string;
};

const { data, send } = useJsonStream<{ users: User[] }>("users");

const loadUsers = () => {
    send({
        query: "taylor",
    });
};
</script>

<template>
    <div>
        <ul>
            <li v-for="user in data?.users" :key="user.id">
                {{ user.id }}: {{ user.name }}
            </li>
        </ul>
        <button @click="loadUsers">Load Users</button>
    </div>
</template>
```

```svelte tab=Svelte
<script>
import { useJsonStream } from "@laravel/stream-svelte";

const stream = useJsonStream("users");

const loadUsers = () => {
    stream.send({
        query: "taylor",
    });
};
</script>

<div>
    <ul>
        {#if $stream.data?.users}
            {#each $stream.data.users as user (user.id)}
                <li>{user.id}: {user.name}</li>
            {/each}
        {/if}
    </ul>
    <button onclick={loadUsers}>Load Users</button>
</div>
```

<a name="event-streams"></a>
### Event Streams (SSE)

Phương thức `eventStream` có thể được sử dụng để trả về một server-sent events (SSE) streamed response sử dụng content type `text/event-stream`. Phương thức `eventStream` chấp nhận một closure nên [yield](https://www.php.net/manual/en/language.generators.overview.php) responses đến stream khi các responses trở nên available:

```php
Route::get('/chat', function () {
    return response()->eventStream(function () {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

Nếu bạn muốn customize tên của event, bạn có thể yield một instance của class `StreamedEvent`:

```php
use Illuminate\Http\StreamedEvent;

yield new StreamedEvent(
    event: 'update',
    data: $response->choices[0],
);
```

<a name="consuming-event-streams"></a>
#### Consuming Event Streams

Event streams có thể được consumed sử dụng npm package `stream` của Laravel, cung cấp một API thuận tiện để tương tác với Laravel event streams. Để bắt đầu, cài đặt package `@laravel/stream-react`, `@laravel/stream-vue`, hoặc `@laravel/stream-svelte`:

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

```shell tab=Svelte
npm install @laravel/stream-svelte
```

Sau đó, `useEventStream` có thể được sử dụng để consume event stream. Sau khi cung cấp stream URL của bạn, hook sẽ tự động update `message` với response được nối khi messages được trả về từ ứng dụng Laravel của bạn:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/chat");

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat");
</script>

<template>
  <div>{{ message }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useEventStream } from "@laravel/stream-svelte";

const eventStream = useEventStream("/chat");
</script>

<div>{$eventStream.message}</div>
```

Đối số thứ hai được đưa cho `useEventStream` là một options object mà bạn có thể sử dụng để customize hành vi stream consumption. Các giá trị mặc định cho object này được hiển thị bên dưới:

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/stream", {
    eventName: "update",
    onMessage: (message) => {
      //
    },
    onError: (error) => {
      //
    },
    onComplete: () => {
      //
    },
    endSignal: "</stream>",
    glue: " ",
  });

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat", {
  eventName: "update",
  onMessage: (message) => {
    // ...
  },
  onError: (error) => {
    // ...
  },
  onComplete: () => {
    // ...
  },
  endSignal: "</stream>",
  glue: " ",
});
</script>
```

```svelte tab=Svelte
<script>
import { useEventStream } from "@laravel/stream-svelte";

const eventStream = useEventStream("/chat", {
    eventName: "update",
    onMessage: (event) => {
        //
    },
    onError: (error) => {
        //
    },
    onComplete: () => {
        //
    },
    endSignal: "</stream>",
    glue: " ",
    replace: false,
});
</script>
```

Event streams cũng có thể được consumed thủ công thông qua một object [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) bởi frontend của ứng dụng của bạn. Phương thức `eventStream` sẽ tự động gửi một update `</stream>` đến event stream khi stream hoàn thành:

```js
const source = new EventSource('/chat');

source.addEventListener('update', (event) => {
    if (event.data === '</stream>') {
        source.close();

        return;
    }

    console.log(event.data);
});
```

Để customize event cuối cùng được gửi đến event stream, bạn có thể cung cấp một instance `StreamedEvent` cho đối số `endStreamWith` của phương thức `eventStream`:

```php
return response()->eventStream(function () {
    // ...
}, endStreamWith: new StreamedEvent(event: 'update', data: '</stream>'));
```

<a name="streamed-downloads"></a>
### Streamed Downloads

Đôi khi bạn có thể muốn chuyển đổi string response của một operation nhất định thành một downloadable response mà không cần viết contents của operation vào disk. Bạn có thể sử dụng phương thức `streamDownload` trong scenario này. Phương thức này chấp nhận một callback, filename, và một mảng headers tùy chọn làm các đối số của nó:

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
        ->contents()
        ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

<a name="response-macros"></a>
## Response Macros

Nếu bạn muốn định nghĩa một custom response mà bạn có thể re-use trong nhiều routes và controllers của bạn, bạn có thể sử dụng phương thức `macro` trên `Response` facade. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của một trong các [service providers](/docs/{{version}}/providers) của ứng dụng của bạn, chẳng hạn như service provider `App\Providers\AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

Hàm `macro` chấp nhận một tên làm đối số thứ nhất và một closure làm đối số thứ hai. Closure của macro sẽ được thực thi khi gọi tên macro từ một implementation `ResponseFactory` hoặc `response` helper:

```php
return response()->caps('foo');
```

# Laravel Sanctum

- [Giới thiệu](#introduction)
    - [Cách hoạt động](#how-it-works)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
    - [Ghi đè Default Models](#overriding-default-models)
- [API Token Authentication](#api-token-authentication)
    - [Issuing API Tokens](#issuing-api-tokens)
    - [Token Abilities](#token-abilities)
    - [Bảo vệ Routes](#protecting-routes)
    - [Revoking Tokens](#revoking-tokens)
    - [Token Expiration](#token-expiration)
- [SPA Authentication](#spa-authentication)
    - [Cấu hình](#spa-configuration)
    - [Authenticating](#spa-authenticating)
    - [Bảo vệ Routes](#protecting-spa-routes)
    - [Authorizing Private Broadcast Channels](#authorizing-private-broadcast-channels)
- [Mobile Application Authentication](#mobile-application-authentication)
    - [Issuing API Tokens](#issuing-mobile-api-tokens)
    - [Bảo vệ Routes](#protecting-mobile-api-routes)
    - [Revoking Tokens](#revoking-mobile-api-tokens)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Sanctum](https://github.com/laravel/sanctum) cung cấp một hệ thống authentication nhẹ cho SPAs (single page applications), mobile applications, và các API dựa trên token đơn giản. Sanctum cho phép mỗi người dùng của ứng dụng tạo nhiều API tokens cho tài khoản của họ. Các tokens này có thể được cấp abilities / scopes chỉ định các actions mà tokens được phép thực hiện.

<a name="how-it-works"></a>
### Cách hoạt động

Laravel Sanctum tồn tại để giải quyết hai vấn đề riêng biệt. Hãy thảo luận từng vấn đề trước khi đi sâu vào thư viện.

<a name="how-it-works-api-tokens"></a>
#### API Tokens

Đầu tiên, Sanctum là một package đơn giản bạn có thể sử dụng để cấp API tokens cho người dùng mà không có sự phức tạp của OAuth. Tính năng này được lấy cảm hứng từ GitHub và các ứng dụng khác cấp "personal access tokens". Ví dụ, hãy tưởng tượng "account settings" của ứng dụng có một màn hình nơi người dùng có thể tạo một API token cho tài khoản của họ. Bạn có thể sử dụng Sanctum để tạo và quản lý các tokens đó. Các tokens này thường có thời gian hết hạn rất dài (năm), nhưng có thể được revoke thủ công bởi người dùng bất cứ lúc nào.

Laravel Sanctum cung cấp tính năng này bằng cách lưu trữ user API tokens trong một database table duy nhất và authenticate các HTTP requests đến qua header `Authorization` nên chứa một API token hợp lệ.

<a name="how-it-works-spa-authentication"></a>
#### SPA Authentication

Thứ hai, Sanctum tồn tại để cung cấp một cách đơn giản để authenticate single page applications (SPAs) cần giao tiếp với một API được hỗ trợ bởi Laravel. Các SPAs này có thể tồn tại trong cùng repository với ứng dụng Laravel của bạn hoặc có thể là một repository hoàn toàn riêng biệt, chẳng hạn như một SPA được tạo bằng Next.js hoặc Nuxt.

Đối với tính năng này, Sanctum không sử dụng bất kỳ loại token nào. Thay vào đó, Sanctum sử dụng các dịch vụ authentication session dựa trên cookie tích hợp của Laravel. Thông thường, Sanctum sử dụng authentication guard `web` của Laravel để thực hiện điều này. Điều này cung cấp các lợi ích của CSRF protection, session authentication, cũng như bảo vệ chống lại rò rỉ authentication credentials qua XSS.

Sanctum sẽ chỉ cố gắng authenticate bằng cookies khi request đến xuất phát từ SPA frontend của chính bạn. Khi Sanctum kiểm tra một HTTP request đến, nó sẽ trước tiên kiểm tra một authentication cookie và, nếu không có, Sanctum sẽ sau đó kiểm tra header `Authorization` cho một API token hợp lệ.

> [!NOTE]
> Hoàn toàn ổn khi sử dụng Sanctum chỉ cho API token authentication hoặc chỉ cho SPA authentication. Chỉ vì bạn sử dụng Sanctum không có nghĩa là bạn bắt buộc phải sử dụng cả hai tính năng nó cung cấp.

<a name="installation"></a>
## Cài đặt

Bạn có thể cài đặt Laravel Sanctum qua lệnh Artisan `install:api`:

```shell
php artisan install:api
```

Tiếp theo, nếu bạn định sử dụng Sanctum để authenticate một SPA, hãy tham khảo phần [SPA Authentication](#spa-authentication) của tài liệu này.

<a name="configuration"></a>
## Cấu hình

<a name="overriding-default-models"></a>
### Ghi đè Default Models

Mặc dù thường không cần thiết, bạn có thể tự do extend model `PersonalAccessToken` được sử dụng nội bộ bởi Sanctum:

```php
use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

class PersonalAccessToken extends SanctumPersonalAccessToken
{
    // ...
}
```

Sau đó, bạn có thể hướng dẫn Sanctum sử dụng custom model của bạn thông qua phương thức `usePersonalAccessTokenModel` được cung cấp bởi Sanctum. Thông thường, bạn nên gọi phương thức này trong phương thức `boot` của file `AppServiceProvider` của ứng dụng:

```php
use App\Models\Sanctum\PersonalAccessToken;
use Laravel\Sanctum\Sanctum;

/**
 * Bootstrap bất kỳ application services nào.
 */
public function boot(): void
{
    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
}
```

<a name="api-token-authentication"></a>
## API Token Authentication

> [!NOTE]
> Bạn không nên sử dụng API tokens để authenticate SPA first-party của chính mình. Thay vào đó, hãy sử dụng [SPA authentication features](#spa-authentication) tích hợp của Sanctum.

<a name="issuing-api-tokens"></a>
### Issuing API Tokens

Sanctum cho phép bạn cấp API tokens / personal access tokens có thể được sử dụng để authenticate API requests đến ứng dụng của bạn. Khi thực hiện requests bằng API tokens, token nên được bao gồm trong header `Authorization` như một `Bearer` token.

Để bắt đầu cấp tokens cho người dùng, User model của bạn nên sử dụng trait `Laravel\Sanctum\HasApiTokens`:

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Để cấp một token, bạn có thể sử dụng phương thức `createToken`. Phương thức `createToken` trả về một instance `Laravel\Sanctum\NewAccessToken`. API tokens được hashed bằng SHA-256 hashing trước khi được lưu trữ trong database của bạn, nhưng bạn có thể truy cập giá trị plain-text của token bằng thuộc tính `plainTextToken` của instance `NewAccessToken`. Bạn nên hiển thị giá trị này cho người dùng ngay sau khi token đã được tạo:

```php
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
    $token = $request->user()->createToken($request->token_name);

    return ['token' => $token->plainTextToken];
});
```

Bạn có thể truy cập tất cả các tokens của người dùng bằng relationship Eloquent `tokens` được cung cấp bởi trait `HasApiTokens`:

```php
foreach ($user->tokens as $token) {
    // ...
}
```

<a name="token-abilities"></a>
### Token Abilities

Sanctum cho phép bạn gán "abilities" cho tokens. Abilities phục vụ mục đích tương tự như "scopes" của OAuth. Bạn có thể chuyển một mảng string abilities làm đối số thứ hai cho phương thức `createToken`:

```php
return $user->createToken('token-name', ['server:update'])->plainTextToken;
```

Khi xử lý một request đến được authenticate bởi Sanctum, bạn có thể xác định xem token có một ability cụ thể hay không bằng các phương thức `tokenCan` hoặc `tokenCant`:

```php
if ($user->tokenCan('server:update')) {
    // ...
}

if ($user->tokenCant('server:update')) {
    // ...
}
```

<a name="token-ability-middleware"></a>
#### Token Ability Middleware

Sanctum cũng bao gồm hai middleware có thể được sử dụng để xác minh rằng request đến được authenticate với một token đã được cấp một ability cụ thể. Để bắt đầu, hãy định nghĩa các middleware aliases sau trong file `bootstrap/app.php` của ứng dụng:

```php
use Laravel\Sanctum\Http\Middleware\CheckAbilities;
use Laravel\Sanctum\Http\Middleware\CheckForAnyAbility;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'abilities' => CheckAbilities::class,
        'ability' => CheckForAnyAbility::class,
    ]);
})
```

Middleware `abilities` có thể được gán cho một route để xác minh rằng token của request đến có tất cả các abilities được liệt kê:

```php
Route::get('/orders', function () {
    // Token has both "check-status" and "place-orders" abilities...
})->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);
```

Middleware `ability` có thể được gán cho một route để xác minh rằng token của request đến có *ít nhất một* trong các abilities được liệt kê:

```php
Route::get('/orders', function () {
    // Token has the "check-status" or "place-orders" ability...
})->middleware(['auth:sanctum', 'ability:check-status,place-orders']);
```

<a name="first-party-ui-initiated-requests"></a>
#### First-Party UI Initiated Requests

Để thuận tiện, phương thức `tokenCan` sẽ luôn trả về `true` nếu request authenticated đến là từ SPA first-party của bạn và bạn đang sử dụng [SPA authentication](#spa-authentication) tích hợp của Sanctum.

Tuy nhiên, điều này không nhất thiết có nghĩa là ứng dụng của bạn phải cho phép người dùng thực hiện action. Thông thường, [authorization policies](/docs/{{version}}/authorization#creating-policies) của ứng dụng sẽ xác định xem token đã được cấp permission để thực hiện các abilities cũng như kiểm tra xem user instance có nên được phép thực hiện action hay không.

Ví dụ, nếu chúng ta tưởng tượng một ứng dụng quản lý servers, điều này có thể có nghĩa là kiểm tra xem token có được phép update servers **và** server thuộc về người dùng:

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

Lúc đầu, cho phép phương thức `tokenCan` được gọi và luôn trả về `true` cho các requests được khởi tạo bởi UI first-party có thể có vẻ kỳ lạ; tuy nhiên, rất thuận tiện khi có thể luôn giả định rằng một API token có sẵn và có thể được kiểm tra qua phương thức `tokenCan`. Bằng cách tiếp cận này, bạn có thể luôn gọi phương thức `tokenCan` trong authorization policies của ứng dụng mà không cần lo lắng về việc request được kích hoạt từ UI của ứng dụng hay được khởi tạo bởi một trong các consumers third-party của API.

<a name="protecting-routes"></a>
### Bảo vệ Routes

Để bảo vệ routes để tất cả các requests đến phải được authenticate, bạn nên gán authentication guard `sanctum` cho các protected routes của bạn trong các file route `routes/web.php` và `routes/api.php`. Guard này sẽ đảm bảo rằng các requests đến được authenticate như các requests authenticated cookie stateful hoặc chứa một header API token hợp lệ nếu request đến từ third party.

Bạn có thể tự hỏi tại sao chúng tôi đề xuất bạn authenticate các routes trong file `routes/web.php` của ứng dụng bằng guard `sanctum`. Hãy nhớ rằng, Sanctum sẽ trước tiên cố gắng authenticate các requests đến bằng cookie authentication session điển hình của Laravel. Nếu cookie đó không có thì Sanctum sẽ cố gắng authenticate request bằng một token trong header `Authorization` của request. Ngoài ra, authenticate tất cả các requests bằng Sanctum đảm bảo rằng chúng ta có thể luôn gọi phương thức `tokenCan` trên user instance được authenticate hiện tại:

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-tokens"></a>
### Revoking Tokens

Bạn có thể "revoke" tokens bằng cách xóa chúng khỏi database bằng relationship `tokens` được cung cấp bởi trait `Laravel\Sanctum\HasApiTokens`:

```php
// Revoke all tokens...
$user->tokens()->delete();

// Revoke the token that was used to authenticate the current request...
$request->user()->currentAccessToken()->delete();

// Revoke a specific token...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="token-expiration"></a>
### Token Expiration

Theo mặc định, tokens Sanctum không bao giờ hết hạn và chỉ có thể bị vô hiệu hóa bằng [revoking the token](#revoking-tokens). Tuy nhiên, nếu bạn muốn cấu hình thời gian hết hạn cho API tokens của ứng dụng, bạn có thể làm như vậy thông qua tùy chọn cấu hình `expiration` được định nghĩa trong file cấu hình `sanctum` của ứng dụng. Tùy chọn cấu hình này định nghĩa số phút cho đến khi một token được cấp sẽ được coi là hết hạn:

```php
'expiration' => 525600,
```

Nếu bạn muốn chỉ định thời gian hết hạn của từng token một cách độc lập, bạn có thể làm như vậy bằng cách cung cấp thời gian hết hạn làm đối số thứ ba cho phương thức `createToken`:

```php
return $user->createToken(
    'token-name', ['*'], now()->plus(weeks: 1)
)->plainTextToken;
```

Nếu bạn đã cấu hình thời gian hết hạn token cho ứng dụng, bạn cũng có thể muốn [schedule a task](/docs/{{version}}/scheduling) để prune các tokens hết hạn của ứng dụng. May mắn thay, Sanctum bao gồm một lệnh Artisan `sanctum:prune-expired` mà bạn có thể sử dụng để thực hiện điều này. Ví dụ, bạn có thể cấu hình một scheduled task để xóa tất cả các bản ghi database token hết hạn đã hết hạn ít nhất 24 giờ:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

<a name="spa-authentication"></a>
## SPA Authentication

Sanctum cũng tồn tại để cung cấp một phương pháp đơn giản để authenticate single page applications (SPAs) cần giao tiếp với một API được hỗ trợ bởi Laravel. Các SPAs này có thể tồn tại trong cùng repository với ứng dụng Laravel của bạn hoặc có thể là một repository hoàn toàn riêng biệt.

Đối với tính năng này, Sanctum không sử dụng bất kỳ loại token nào. Thay vào đó, Sanctum sử dụng các dịch vụ authentication session dựa trên cookie tích hợp của Laravel. Cách tiếp cận authentication này cung cấp các lợi ích của CSRF protection, session authentication, cũng như bảo vệ chống lại rò rỉ authentication credentials qua XSS.

> [!WARNING]
> Để authenticate, SPA và API của bạn phải chia sẻ cùng top-level domain. Tuy nhiên, chúng có thể được đặt trên các subdomains khác nhau. Ngoài ra, bạn nên đảm bảo rằng bạn gửi header `Accept: application/json` và header `Referer` hoặc `Origin` với request của bạn.

<a name="spa-configuration"></a>
### Cấu hình

<a name="configuring-your-first-party-domains"></a>
#### Configuring Your First-Party Domains

Đầu tiên, bạn nên cấu hình các domains mà SPA của bạn sẽ thực hiện requests từ đó. Bạn có thể cấu hình các domains này bằng tùy chọn cấu hình `stateful` trong file cấu hình `sanctum` của bạn. Cài đặt cấu hình này xác định các domains sẽ duy trì authentication "stateful" bằng Laravel session cookies khi thực hiện requests đến API của bạn.

Để hỗ trợ bạn thiết lập các domains stateful first-party, Sanctum cung cấp hai helper functions mà bạn có thể bao gồm trong cấu hình. Đầu tiên, `Sanctum::currentApplicationUrlWithPort()` sẽ trả về URL ứng dụng hiện tại từ biến môi trường `APP_URL`, và `Sanctum::currentRequestHost()` sẽ inject một placeholder vào danh sách domain stateful mà, tại runtime, sẽ được thay thế bởi host từ request hiện tại để tất cả các requests với cùng domain được coi là stateful.

> [!WARNING]
> Nếu bạn đang truy cập ứng dụng qua một URL bao gồm một port (`127.0.0.1:8000`), bạn nên đảm bảo rằng bạn bao gồm số port với domain.

<a name="sanctum-middleware"></a>
#### Sanctum Middleware

Tiếp theo, bạn nên hướng dẫn Laravel rằng các requests đến từ SPA của bạn có thể authenticate bằng Laravel session cookies, trong khi vẫn cho phép requests từ third parties hoặc mobile applications authenticate bằng API tokens. Điều này có thể được thực hiện dễ dàng bằng cách gọi phương thức middleware `statefulApi` trong file `bootstrap/app.php` của ứng dụng:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->statefulApi();
})
```

<a name="cors-and-cookies"></a>
#### CORS và Cookies

Nếu bạn gặp khó khăn khi authenticate với ứng dụng từ một SPA thực thi trên một subdomain riêng biệt, bạn có thể đã cấu hình sai CORS (Cross-Origin Resource Sharing) hoặc cài đặt session cookie.

File cấu hình `config/cors.php` không được publish theo mặc định. Nếu bạn cần tùy chỉnh các tùy chọn CORS của Laravel, bạn nên publish file cấu hình `cors` hoàn chỉnh bằng lệnh Artisan `config:publish`:

```shell
php artisan config:publish cors
```

Tiếp theo, bạn nên đảm bảo rằng cấu hình CORS của ứng dụng đang trả về header `Access-Control-Allow-Credentials` với giá trị `True`. Điều này có thể được thực hiện bằng cách đặt tùy chọn `supports_credentials` trong file cấu hình `config/cors.php` của ứng dụng thành `true`.

Ngoài ra, bạn nên bật các tùy chọn `withCredentials` và `withXSRFToken` trên instance `axios` global của ứng dụng. Điều này có thể được thực hiện trong file `resources/js/app.js` của bạn. Nếu bạn không sử dụng Axios để thực hiện HTTP requests từ frontend, bạn nên thực hiện cấu hình tương đương trên HTTP client của chính mình:

```js
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
```

Cuối cùng, bạn nên đảm bảo cấu hình domain session cookie của ứng dụng hỗ trợ bất kỳ subdomain nào của root domain của bạn. Bạn có thể thực hiện điều này bằng cách thêm tiền tố domain với một `.` trong file cấu hình `config/session.php` của ứng dụng:

```php
'domain' => '.domain.com',
```

<a name="spa-authenticating"></a>
### Authenticating

<a name="csrf-protection"></a>
#### CSRF Protection

Để authenticate SPA của bạn, trang "login" của SPA nên trước tiên thực hiện một request đến endpoint `/sanctum/csrf-cookie` để khởi tạo CSRF protection cho ứng dụng:

```js
axios.get('/sanctum/csrf-cookie').then(response => {
    // Login...
});
```

Trong quá trình request này, Laravel sẽ đặt một cookie `XSRF-TOKEN` chứa CSRF token hiện tại. Token này sau đó nên được URL decode và chuyển trong header `X-XSRF-TOKEN` trên các requests tiếp theo, mà một số thư viện HTTP client như Axios và Angular HttpClient sẽ tự động làm cho bạn. Nếu thư viện HTTP JavaScript của bạn không đặt giá trị cho bạn, bạn sẽ cần đặt thủ công header `X-XSRF-TOKEN` để khớp với giá trị URL decode của cookie `XSRF-TOKEN` được đặt bởi route này.

<a name="logging-in"></a>
#### Logging In

Sau khi CSRF protection đã được khởi tạo, bạn nên thực hiện một request `POST` đến route `/login` của ứng dụng Laravel. Route `/login` này có thể được [implemented manually](/docs/{{version}}/authentication#authenticating-users) hoặc sử dụng một package authentication headless như [Laravel Fortify](/docs/{{version}}/fortify).

Nếu request login thành công, bạn sẽ được authenticate và các requests tiếp theo đến routes của ứng dụng sẽ tự động được authenticate qua session cookie mà ứng dụng Laravel cấp cho client của bạn. Ngoài ra, vì ứng dụng của bạn đã thực hiện một request đến route `/sanctum/csrf-cookie`, các requests tiếp theo nên tự động nhận CSRF protection miễn là client HTTP JavaScript của bạn gửi giá trị của cookie `XSRF-TOKEN` trong header `X-XSRF-TOKEN`.

Tất nhiên, nếu session của người dùng hết hạn do thiếu hoạt động, các requests tiếp theo đến ứng dụng Laravel có thể nhận được phản hồi lỗi HTTP 401 hoặc 419. Trong trường hợp này, bạn nên redirect người dùng đến trang login của SPA.

Vì cách tiếp cận SPA authentication này dựa trên session, bạn có thể sử dụng các dịch vụ authentication tiêu chuẩn của Laravel, bao gồm chức năng ["remember me"](/docs/{{version}}/authentication#remembering-users).

> [!WARNING]
> Bạn có thể tự do viết endpoint `/login` của riêng mình; tuy nhiên, bạn nên đảm bảo rằng nó authenticate người dùng bằng các dịch vụ authentication session tiêu chuẩn [mà Laravel cung cấp](/docs/{{version}}/authentication#authenticating-users). Thông thường, điều này có nghĩa là sử dụng authentication guard `web`.

<a name="protecting-spa-routes"></a>
### Bảo vệ Routes

Để bảo vệ routes để tất cả các requests đến phải được authenticate, bạn nên gán authentication guard `sanctum` cho các API routes của bạn trong file `routes/api.php`. Guard này sẽ đảm bảo rằng các requests đến được authenticate như các requests authenticated stateful từ SPA của bạn hoặc chứa một header API token hợp lệ nếu request đến từ third party:

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="authorizing-private-broadcast-channels"></a>
### Authorizing Private Broadcast Channels

Nếu SPA của bạn cần authenticate với [private / presence broadcast channels](/docs/{{version}}/broadcasting#authorizing-channels), bạn nên xóa entry `channels` khỏi phương thức `withRouting` có trong file `bootstrap/app.php` của ứng dụng. Thay vào đó, bạn nên gọi phương thức `withBroadcasting` để bạn có thể chỉ định middleware chính xác cho các broadcasting routes của ứng dụng:

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        // ...
    )
    ->withBroadcasting(
        __DIR__.'/../routes/channels.php',
        ['prefix' => 'api', 'middleware' => ['api', 'auth:sanctum']],
    )
```

Tiếp theo, để các requests authorization của Pusher thành công, bạn sẽ cần cung cấp một Pusher `authorizer` tùy chỉnh khi khởi tạo [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation). Điều này cho phép ứng dụng của bạn cấu hình Pusher để sử dụng instance `axios` được [cấu hình đúng cho cross-domain requests](#cors-and-cookies):

```js
window.Echo = new Echo({
    broadcaster: "pusher",
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    encrypted: true,
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(false, response.data);
                })
                .catch(error => {
                    callback(true, error);
                });
            }
        };
    },
})
```

<a name="mobile-application-authentication"></a>
## Mobile Application Authentication

Bạn cũng có thể sử dụng tokens Sanctum để authenticate các requests của mobile application đến API của bạn. Quá trình authenticate mobile application requests tương tự như authenticate third-party API requests; tuy nhiên, có những khác biệt nhỏ trong cách bạn sẽ cấp API tokens.

<a name="issuing-mobile-api-tokens"></a>
### Issuing API Tokens

Để bắt đầu, hãy tạo một route chấp nhận email / username, password, và device name của người dùng, sau đó đổi các credentials đó lấy một token Sanctum mới. "Device name" được cung cấp cho endpoint này là cho mục đích thông tin và có thể là bất kỳ giá trị nào bạn muốn. Nói chung, giá trị device name nên là một tên mà người dùng sẽ nhận ra, chẳng hạn như "Nuno's iPhone 17".

Thông thường, bạn sẽ thực hiện một request đến token endpoint từ màn hình "login" của mobile application. Endpoint sẽ trả về API token plain-text mà sau đó có thể được lưu trữ trên thiết bị mobile và sử dụng để thực hiện các API requests bổ sung:

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});
```

Khi mobile application sử dụng token để thực hiện một API request đến ứng dụng của bạn, nó nên chuyển token trong header `Authorization` như một `Bearer` token.

> [!NOTE]
> Khi cấp tokens cho một mobile application, bạn cũng có thể tự do chỉ định [token abilities](#token-abilities).

<a name="protecting-mobile-api-routes"></a>
### Bảo vệ Routes

Như đã được tài liệu hóa trước đó, bạn có thể bảo vệ routes để tất cả các requests đến phải được authenticate bằng cách gán authentication guard `sanctum` cho các routes:

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-mobile-api-tokens"></a>
### Revoking Tokens

Để cho phép người dùng revoke API tokens được cấp cho thiết bị mobile, bạn có thể liệt kê chúng theo tên, cùng với một nút "Revoke", trong một phần "account settings" của UI ứng dụng web của bạn. Khi người dùng nhấp vào nút "Revoke", bạn có thể xóa token khỏi database. Hãy nhớ rằng, bạn có thể truy cập các API tokens của người dùng thông qua relationship `tokens` được cung cấp bởi trait `Laravel\Sanctum\HasApiTokens`:

```php
// Revoke all tokens...
$user->tokens()->delete();

// Revoke a specific token...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="testing"></a>
## Testing

Trong khi testing, phương thức `Sanctum::actingAs` có thể được sử dụng để authenticate một người dùng và chỉ định các abilities nên được cấp cho token của họ:

```php tab=Pest
use App\Models\User;
use Laravel\Sanctum\Sanctum;

test('task list can be retrieved', function () {
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
    );

    $response = $this->get('/api/task');

    $response->assertOk();
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Sanctum\Sanctum;

public function test_task_list_can_be_retrieved(): void
{
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
    );

    $response = $this->get('/api/task');

    $response->assertOk();
}
```

Nếu bạn muốn cấp tất cả abilities cho token, bạn nên bao gồm `*` trong danh sách ability được cung cấp cho phương thức `actingAs`:

```php
Sanctum::actingAs(
    User::factory()->create(),
    ['*']
);
```

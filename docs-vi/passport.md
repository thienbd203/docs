# Laravel Passport

- [Giới thiệu](#introduction)
    - [Passport hay Sanctum?](#passport-or-sanctum)
- [Cài đặt](#installation)
    - [Triển khai Passport](#deploying-passport)
    - [Nâng cấp Passport](#upgrading-passport)
- [Cấu hình](#configuration)
    - [Thời gian tồn tại của Token](#token-lifetimes)
    - [Ghi đè Model mặc định](#overriding-default-models)
    - [Ghi đè Routes](#overriding-routes)
- [Authorization Code Grant](#authorization-code-grant)
    - [Quản lý Clients](#managing-clients)
    - [Yêu cầu Tokens](#requesting-tokens)
    - [Quản lý Tokens](#managing-tokens)
    - [Làm mới Tokens](#refreshing-tokens)
    - [Thu hồi Tokens](#revoking-tokens)
    - [Xóa Tokens](#purging-tokens)
- [Authorization Code Grant Với PKCE](#code-grant-pkce)
    - [Tạo Client](#creating-a-auth-pkce-grant-client)
    - [Yêu cầu Tokens](#requesting-auth-pkce-grant-tokens)
- [Device Authorization Grant](#device-authorization-grant)
    - [Tạo Device Code Grant Client](#creating-a-device-authorization-grant-client)
    - [Yêu cầu Tokens](#requesting-device-authorization-grant-tokens)
- [Password Grant](#password-grant)
    - [Tạo Password Grant Client](#creating-a-password-grant-client)
    - [Yêu cầu Tokens](#requesting-password-grant-tokens)
    - [Yêu cầu tất cả Scopes](#requesting-all-scopes)
    - [Tùy chỉnh User Provider](#customizing-the-user-provider)
    - [Tùy chỉnh trường Username](#customizing-the-username-field)
    - [Tùy chỉnh xác thực mật khẩu](#customizing-the-password-validation)
- [Implicit Grant](#implicit-grant)
- [Client Credentials Grant](#client-credentials-grant)
- [Personal Access Tokens](#personal-access-tokens)
    - [Tạo Personal Access Client](#creating-a-personal-access-client)
    - [Tùy chỉnh User Provider](#customizing-the-user-provider-for-pat)
    - [Quản lý Personal Access Tokens](#managing-personal-access-tokens)
- [Bảo vệ Routes](#protecting-routes)
    - [Thông qua Middleware](#via-middleware)
    - [Truyền Access Token](#passing-the-access-token)
- [Token Scopes](#token-scopes)
    - [Định nghĩa Scopes](#defining-scopes)
    - [Scope mặc định](#default-scope)
    - [Gán Scopes cho Tokens](#assigning-scopes-to-tokens)
    - [Kiểm tra Scopes](#checking-scopes)
- [Xác thực SPA](#spa-authentication)
- [Sự kiện](#events)
- [Kiểm thử](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Passport](https://github.com/laravel/passport) cung cấp một triển khai server OAuth2 hoàn chỉnh cho ứng dụng Laravel của bạn chỉ trong vài phút. Passport được xây dựng dựa trên [League OAuth2 server](https://github.com/thephpleague/oauth2-server) được duy trì bởi Andy Millington và Simon Hamp.

> [!NOTE]
> Tài liệu này giả định rằng bạn đã quen thuộc với OAuth2. Nếu bạn không biết gì về OAuth2, hãy cân nhắc làm quen với [terminology](https://oauth2.thephpleague.com/terminology/) và các tính năng chung của OAuth2 trước khi tiếp tục.

<a name="passport-or-sanctum"></a>
### Passport hay Sanctum?

Trước khi bắt đầu, bạn có thể muốn xác định xem ứng dụng của bạn sẽ được phục vụ tốt hơn bởi Laravel Passport hay [Laravel Sanctum](/docs/{{version}}/sanctum). Nếu ứng dụng của bạn hoàn toàn cần hỗ trợ OAuth2, thì bạn nên sử dụng Laravel Passport.

Tuy nhiên, nếu bạn đang cố gắng xác thực một ứng dụng trang đơn (single-page application), ứng dụng di động, hoặc phát hành API tokens, bạn nên sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum). Laravel Sanctum không hỗ trợ OAuth2; tuy nhiên, nó cung cấp trải nghiệm phát triển xác thực API đơn giản hơn nhiều.

<a name="installation"></a>
## Cài đặt

Bạn có thể cài đặt Laravel Passport thông qua lệnh Artisan `install:api`:

```shell
php artisan install:api --passport
```

Lệnh này sẽ xuất bản và chạy các migration cơ sở dữ liệu cần thiết để tạo các bảng mà ứng dụng của bạn cần để lưu trữ OAuth2 clients và access tokens. Lệnh này cũng sẽ tạo các khóa mã hóa cần thiết để tạo các access token an toàn.

Sau khi chạy lệnh `install:api`, hãy thêm trait `Laravel\Passport\HasApiTokens` và interface `Laravel\Passport\Contracts\OAuthenticatable` vào model `App\Models\User` của bạn. Trait này sẽ cung cấp một vài phương thức trợ giúp cho model của bạn cho phép bạn kiểm tra token và scopes của người dùng đã xác thực:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Cuối cùng, trong file cấu hình `config/auth.php` của ứng dụng, bạn nên định nghĩa một guard xác thực `api` và đặt tùy chọn `driver` thành `passport`. Điều này sẽ hướng dẫn ứng dụng của bạn sử dụng `TokenGuard` của Passport khi xác thực các yêu cầu API đến:

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

<a name="deploying-passport"></a>
### Triển khai Passport

Khi triển khai Passport lên các máy chủ của ứng dụng lần đầu tiên, bạn có thể sẽ cần chạy lệnh `passport:keys`. Lệnh này tạo ra các khóa mã hóa mà Passport cần để tạo access tokens. Các khóa được tạo thường không được giữ trong source control:

```shell
php artisan passport:keys
```

Nếu cần thiết, bạn có thể định nghĩa đường dẫn nơi các khóa của Passport nên được tải từ. Bạn có thể sử dụng phương thức `Passport::loadKeysFrom` để thực hiện việc này. Thông thường, phương thức này nên được gọi từ phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
}
```

<a name="loading-keys-from-the-environment"></a>
#### Tải Khóa Từ Môi trường

Ngoài ra, bạn có thể xuất bản file cấu hình của Passport bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag=passport-config
```

Sau khi file cấu hình đã được xuất bản, bạn có thể tải các khóa mã hóa của ứng dụng bằng cách định nghĩa chúng dưới dạng biến môi trường:

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="upgrading-passport"></a>
### Nâng cấp Passport

Khi nâng cấp lên phiên bản chính mới của Passport, điều quan trọng là bạn phải xem xét kỹ [hướng dẫn nâng cấp](https://github.com/laravel/passport/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Cấu hình

<a name="token-lifetimes"></a>
### Thời gian tồn tại của Token

Theo mặc định, Passport phát hành các access token tồn tại lâu sẽ hết hạn sau một năm. Nếu bạn muốn cấu hình thời gian tồn tại token dài hơn / ngắn hơn, bạn có thể sử dụng các phương thức `tokensExpireIn`, `refreshTokensExpireIn`, và `personalAccessTokensExpireIn`. Các phương thức này nên được gọi từ phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Carbon\CarbonInterval;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensExpireIn(CarbonInterval::days(15));
    Passport::refreshTokensExpireIn(CarbonInterval::days(30));
    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
}
```

> [!WARNING]
> Các cột `expires_at` trên các bảng cơ sở dữ liệu của Passport là chỉ đọc và chỉ phục vụ mục đích hiển thị. Khi phát hành tokens, Passport lưu trữ thông tin hết hạn trong các token đã ký và mã hóa. Nếu bạn cần vô hiệu hóa một token, bạn nên [thu hồi nó](#revoking-tokens).

<a name="overriding-default-models"></a>
### Ghi đè Model mặc định

Bạn có thể mở rộng các model được sử dụng nội bộ bởi Passport bằng cách định nghĩa model của riêng bạn và mở rộng model Passport tương ứng:

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

Sau khi định nghĩa model của bạn, bạn có thể hướng dẫn Passport sử dụng model tùy chỉnh của bạn thông qua class `Laravel\Passport\Passport`. Thông thường, bạn nên thông báo cho Passport về các model tùy chỉnh của bạn trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\DeviceCode;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::useTokenModel(Token::class);
    Passport::useRefreshTokenModel(RefreshToken::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::useClientModel(Client::class);
    Passport::useDeviceCodeModel(DeviceCode::class);
}
```

<a name="overriding-routes"></a>
### Ghi đè Routes

Đôi khi bạn có thể muốn tùy chỉnh các routes được định nghĩa bởi Passport. Để thực hiện việc này, trước tiên bạn cần bỏ qua các routes được đăng ký bởi Passport bằng cách thêm `Passport::ignoreRoutes` vào phương thức `register` của `AppServiceProvider` của ứng dụng:

```php
use Laravel\Passport\Passport;

/**
 * Register any application services.
 */
public function register(): void
{
    Passport::ignoreRoutes();
}
```

Sau đó, bạn có thể sao chép các routes được định nghĩa bởi Passport trong [file routes của nó](https://github.com/laravel/passport/blob/master/routes/web.php) vào file `routes/web.php` của ứng dụng và sửa đổi chúng theo ý muốn:

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport routes...
});
```

<a name="authorization-code-grant"></a>
## Authorization Code Grant

Sử dụng OAuth2 thông qua authorization codes là cách mà hầu hết các nhà phát triển quen thuộc với OAuth2. Khi sử dụng authorization codes, một ứng dụng client sẽ chuyển hướng người dùng đến máy chủ của bạn nơi họ sẽ phê duyệt hoặc từ chối yêu cầu phát hành access token cho client.

Để bắt đầu, chúng ta cần hướng dẫn Passport cách trả về view "authorization" của chúng ta.

Tất cả logic hiển thị của view authorization có thể được tùy chỉnh bằng cách sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Passport\Passport`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::authorizationView('auth.oauth.authorize');

    // By providing a closure...
    Passport::authorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );
}
```

Passport sẽ tự động định nghĩa route `/oauth/authorize` trả về view này. Template `auth.oauth.authorize` của bạn nên bao gồm một form thực hiện yêu cầu POST đến route `passport.authorizations.approve` để phê duyệt authorization và một form thực hiện yêu cầu DELETE đến route `passport.authorizations.deny` để từ chối authorization. Các route `passport.authorizations.approve` và `passport.authorizations.deny` mong đợi các trường `state`, `client_id`, và `auth_token`.

<a name="managing-clients"></a>
### Quản lý Clients

Các nhà phát triển xây dựng các ứng dụng cần tương tác với API của ứng dụng bạn sẽ cần đăng ký ứng dụng của họ với bạn bằng cách tạo một "client". Thông thường, điều này bao gồm cung cấp tên ứng dụng của họ và một URI mà ứng dụng của bạn có thể chuyển hướng đến sau khi người dùng phê duyệt yêu cầu authorization của họ.

<a name="managing-first-party-clients"></a>
#### First-Party Clients

Cách đơn giản nhất để tạo một client là sử dụng lệnh Artisan `passport:client`. Lệnh này có thể được sử dụng để tạo các client first-party hoặc kiểm tra chức năng OAuth2 của bạn. Khi bạn chạy lệnh `passport:client`, Passport sẽ nhắc bạn nhập thêm thông tin về client của bạn và sẽ cung cấp cho bạn một client ID và secret:

```shell
php artisan passport:client
```

Nếu bạn muốn cho phép nhiều redirect URIs cho client của bạn, bạn có thể chỉ định chúng bằng danh sách được phân tách bằng dấu phẩy khi được nhắc nhập URI bởi lệnh `passport:client`. Bất kỳ URI nào chứa dấu phẩy nên được mã hóa URI:

```shell
https://third-party-app.com/callback,https://example.com/oauth/redirect
```

<a name="managing-third-party-clients"></a>
#### Third-Party Clients

Vì người dùng của ứng dụng bạn sẽ không thể sử dụng lệnh `passport:client`, bạn có thể sử dụng phương thức `createAuthorizationCodeGrantClient` của class `Laravel\Passport\ClientRepository` để đăng ký một client cho một người dùng nhất định:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

// Creating an OAuth app client that belongs to the given user...
$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient(
    user: $user,
    name: 'Example App',
    redirectUris: ['https://third-party-app.com/callback'],
    confidential: false,
    enableDeviceFlow: true
);

// Retrieving all the OAuth app clients that belong to the user...
$clients = $user->oauthApps()->get();
```

Phương thức `createAuthorizationCodeGrantClient` trả về một instance của `Laravel\Passport\Client`. Bạn có thể hiển thị `$client->id` làm client ID và `$client->plainSecret` làm client secret cho người dùng.

<a name="requesting-tokens"></a>
### Yêu cầu Tokens

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### Chuyển hướng để Authorization

Sau khi một client đã được tạo, các nhà phát triển có thể sử dụng client ID và secret của họ để yêu cầu một authorization code và access token từ ứng dụng của bạn. Đầu tiên, ứng dụng tiêu thụ nên thực hiện một yêu cầu chuyển hướng đến route `/oauth/authorize` của ứng dụng của bạn như sau:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

Tham số `prompt` có thể được sử dụng để chỉ định hành vi xác thực của ứng dụng Passport.

Nếu giá trị `prompt` là `none`, Passport sẽ luôn ném một lỗi xác thực nếu người dùng chưa được xác thực với ứng dụng Passport. Nếu giá trị là `consent`, Passport sẽ luôn hiển thị màn hình phê duyệt authorization, ngay cả khi tất cả các scopes đã được cấp trước đó cho ứng dụng tiêu thụ. Khi giá trị là `login`, ứng dụng Passport sẽ luôn nhắc người dùng đăng nhập lại vào ứng dụng, ngay cả khi họ đã có một session hiện có.

Nếu không có giá trị `prompt` nào được cung cấp, người dùng sẽ được nhắc authorization chỉ khi họ chưa trước đó cấp quyền truy cập cho ứng dụng tiêu thụ cho các scopes được yêu cầu.

> [!NOTE]
> Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi Passport. Bạn không cần định nghĩa thủ công route này.

<a name="approving-the-request"></a>
#### Phê duyệt Yêu cầu

Khi nhận được các yêu cầu authorization, Passport sẽ tự động phản hồi dựa trên giá trị của tham số `prompt` (nếu có) và có thể hiển thị một template cho người dùng cho phép họ phê duyệt hoặc từ chối yêu cầu authorization. Nếu họ phê duyệt yêu cầu, họ sẽ được chuyển hướng trở lại `redirect_uri` được chỉ định bởi ứng dụng tiêu thụ. `redirect_uri` phải khớp với URL `redirect` được chỉ định khi client được tạo.

Đôi khi bạn có thể muốn bỏ qua lời nhắc authorization, chẳng hạn như khi ủy quyền cho một client first-party. Bạn có thể thực hiện việc này bằng cách [mở rộng model `Client`](#overriding-default-models) và định nghĩa một phương thức `skipsAuthorization`. Nếu `skipsAuthorization` trả về `true`, client sẽ được phê duyệt và người dùng sẽ được chuyển hướng trở lại `redirect_uri` ngay lập tức, trừ khi ứng dụng tiêu thụ đã đặt rõ ràng tham số `prompt` khi chuyển hướng để authorization:

```php
<?php

namespace App\Models\Passport;

use Illuminate\Contracts\Auth\Authenticatable;
use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * Determine if the client should skip the authorization prompt.
     *
     * @param  \Laravel\Passport\Scope[]  $scopes
     */
    public function skipsAuthorization(Authenticatable $user, array $scopes): bool
    {
        return $this->firstParty();
    }
}
```

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### Chuyển đổi Authorization Codes thành Access Tokens

Nếu người dùng phê duyệt yêu cầu authorization, họ sẽ được chuyển hướng trở lại ứng dụng tiêu thụ. Người tiêu thụ nên trước tiên xác thực tham số `state` với giá trị đã được lưu trữ trước khi chuyển hướng. Nếu tham số state khớp thì người tiêu thụ nên phát hành một yêu cầu `POST` đến ứng dụng của bạn để yêu cầu một access token. Yêu cầu nên bao gồm authorization code đã được phát hành bởi ứng dụng của bạn khi người dùng phê duyệt yêu cầu authorization:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

Route `/oauth/token` này sẽ trả về một phản hồi JSON chứa các thuộc tính `access_token`, `refresh_token`, và `expires_in`. Thuộc tính `expires_in` chứa số giây cho đến khi access token hết hạn.

> [!NOTE]
> Giống như route `/oauth/authorize`, route `/oauth/token` được định nghĩa cho bạn bởi Passport. Không cần định nghĩa thủ công route này.

<a name="managing-tokens"></a>
### Quản lý Tokens

Bạn có thể truy xuất các token được ủy quyền của người dùng bằng phương thức `tokens` của trait `Laravel\Passport\HasApiTokens`. Ví dụ, điều này có thể được sử dụng để cung cấp cho người dùng của bạn một bảng điều khiển để theo dõi các kết nối của họ với các ứng dụng bên thứ ba:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Retrieving all of the valid tokens for the user...
$tokens = $user->tokens()
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get();

// Retrieving all the user's connections to third-party OAuth app clients...
$connections = $tokens->load('client')
    ->reject(fn (Token $token) => $token->client->firstParty())
    ->groupBy('client_id')
    ->map(fn (Collection $tokens) => [
        'client' => $tokens->first()->client,
        'scopes' => $tokens->pluck('scopes')->flatten()->unique()->values()->all(),
        'tokens_count' => $tokens->count(),
    ])
    ->values();
```

<a name="refreshing-tokens"></a>
### Làm mới Tokens

Nếu ứng dụng của bạn phát hành các access token tồn tại ngắn, người dùng sẽ cần làm mới access token của họ thông qua refresh token đã được cung cấp cho họ khi access token được phát hành:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

Route `/oauth/token` này sẽ trả về một phản hồi JSON chứa các thuộc tính `access_token`, `refresh_token`, và `expires_in`. Thuộc tính `expires_in` chứa số giây cho đến khi access token hết hạn.

<a name="revoking-tokens"></a>
### Thu hồi Tokens

Bạn có thể thu hồi một token bằng cách sử dụng phương thức `revoke` trên model `Laravel\Passport\Token`. Bạn có thể thu hồi refresh token của một token bằng cách sử dụng phương thức `revoke` trên model `Laravel\Passport\RefreshToken`:

```php
use Laravel\Passport\Passport;
use Laravel\Passport\Token;

$token = Passport::token()->find($tokenId);

// Revoke an access token...
$token->revoke();

// Revoke the token's refresh token...
$token->refreshToken?->revoke();

// Revoke all of the user's tokens...
User::find($userId)->tokens()->each(function (Token $token) {
    $token->revoke();
    $token->refreshToken?->revoke();
});
```

<a name="purging-tokens"></a>
### Xóa Tokens

Khi các token đã bị thu hồi hoặc hết hạn, bạn có thể muốn xóa chúng khỏi cơ sở dữ liệu. Lệnh Artisan `passport:purge` được bao gồm trong Passport có thể thực hiện việc này cho bạn:

```shell
# Purge revoked and expired tokens, auth codes, and device codes...
php artisan passport:purge

# Only purge tokens expired for more than 6 hours...
php artisan passport:purge --hours=6

# Only purge revoked tokens, auth codes, and device codes...
php artisan passport:purge --revoked

# Only purge expired tokens, auth codes, and device codes...
php artisan passport:purge --expired
```

Bạn cũng có thể cấu hình một [scheduled job](/docs/{{version}}/scheduling) trong file `routes/console.php` của ứng dụng để tự động cắt bỏ các token của bạn theo lịch:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('passport:purge')->hourly();
```

<a name="code-grant-pkce"></a>
## Authorization Code Grant Với PKCE

Authorization Code grant với "Proof Key for Code Exchange" (PKCE) là một cách an toàn để xác thực các ứng dụng trang đơn hoặc ứng dụng di động để truy cập API của bạn. Grant này nên được sử dụng khi bạn không thể đảm bảo rằng client secret sẽ được lưu trữ một cách bảo mật hoặc để giảm thiểu mối đe dọa có authorization code bị chặn bởi kẻ tấn công. Một sự kết hợp của "code verifier" và "code challenge" thay thế client secret khi trao đổi authorization code để lấy access token.

<a name="creating-a-auth-pkce-grant-client"></a>
### Tạo Client

Trước khi ứng dụng của bạn có thể phát hành tokens thông qua authorization code grant với PKCE, bạn sẽ cần tạo một client được bật PKCE. Bạn có thể thực hiện việc này bằng cách sử dụng lệnh Artisan `passport:client` với tùy chọn `--public`:

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### Yêu cầu Tokens

<a name="code-verifier-code-challenge"></a>
#### Code Verifier và Code Challenge

Vì authorization grant này không cung cấp client secret, các nhà phát triển sẽ cần tạo một sự kết hợp của code verifier và code challenge để yêu cầu một token.

Code verifier nên là một chuỗi ngẫu nhiên từ 43 đến 128 ký tự chứa các chữ cái, số, và các ký tự `"-"`, `"."`, `"_"`, `"~"`, như được định nghĩa trong [đặc tả RFC 7636](https://tools.ietf.org/html/rfc7636).

Code challenge nên là một chuỗi được mã hóa Base64 với các ký tự an toàn cho URL và tên tệp. Các ký tự `'='` ở cuối nên được loại bỏ và không có ngắt dòng, khoảng trắng, hoặc các ký tự bổ sung khác nên có mặt.

```php
$encoded = base64_encode(hash('sha256', $codeVerifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### Chuyển hướng để Authorization

Sau khi một client đã được tạo, bạn có thể sử dụng client ID và code verifier và code challenge được tạo để yêu cầu một authorization code và access token từ ứng dụng của bạn. Đầu tiên, ứng dụng tiêu thụ nên thực hiện một yêu cầu chuyển hướng đến route `/oauth/authorize` của ứng dụng của bạn:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put(
        'code_verifier', $codeVerifier = Str::random(128)
    );

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $codeVerifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### Chuyển đổi Authorization Codes thành Access Tokens

Nếu người dùng phê duyệt yêu cầu authorization, họ sẽ được chuyển hướng trở lại ứng dụng tiêu thụ. Người tiêu thụ nên xác thực tham số `state` với giá trị đã được lưu trữ trước khi chuyển hướng, như trong Authorization Code Grant tiêu chuẩn.

Nếu tham số state khớp, người tiêu thụ nên phát hành một yêu cầu `POST` đến ứng dụng của bạn để yêu cầu một access token. Yêu cầu nên bao gồm authorization code đã được phát hành bởi ứng dụng của bạn khi người dùng phê duyệt yêu cầu authorization cùng với code verifier được tạo ban đầu:

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

<a name="device-authorization-grant"></a>
## Device Authorization Grant

OAuth2 device authorization grant cho phép các thiết bị không có trình duyệt hoặc có đầu vào hạn chế, chẳng hạn như TV và máy chơi game, để nhận được một access token bằng cách trao đổi một "device code". Khi sử dụng device flow, client thiết bị sẽ hướng dẫn người dùng sử dụng một thiết bị phụ, chẳng hạn như máy tính hoặc điện thoại thông minh và kết nối đến máy chủ của bạn nơi họ sẽ nhập "user code" được cung cấp và phê duyệt hoặc từ chối yêu cầu truy cập.

Để bắt đầu, chúng ta cần hướng dẫn Passport cách trả về view "user code" và "authorization" của chúng ta.

Tất cả logic hiển thị của view authorization có thể được tùy chỉnh bằng cách sử dụng các phương thức thích hợp có sẵn thông qua class `Laravel\Passport\Passport`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng.

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::deviceUserCodeView('auth.oauth.device.user-code');
    Passport::deviceAuthorizationView('auth.oauth.device.authorize');

    // By providing a closure...
    Passport::deviceUserCodeView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/UserCode')
    );

    Passport::deviceAuthorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );

    // ...
}
```

Passport sẽ tự động định nghĩa các routes trả về các view này. Template `auth.oauth.device.user-code` của bạn nên bao gồm một form thực hiện yêu cầu GET đến route `passport.device.authorizations.authorize`. Route `passport.device.authorizations.authorize` mong đợi một tham số truy vấn `user_code`.

Template `auth.oauth.device.authorize` của bạn nên bao gồm một form thực hiện yêu cầu POST đến route `passport.device.authorizations.approve` để phê duyệt authorization và một form thực hiện yêu cầu DELETE đến route `passport.device.authorizations.deny` để từ chối authorization. Các route `passport.device.authorizations.approve` và `passport.device.authorizations.deny` mong đợi các trường `state`, `client_id`, và `auth_token`.

<a name="creating-a-device-authorization-grant-client"></a>
### Tạo Device Authorization Grant Client

Trước khi ứng dụng của bạn có thể phát hành tokens thông qua device authorization grant, bạn sẽ cần tạo một client được bật device flow. Bạn có thể thực hiện việc này bằng cách sử dụng lệnh Artisan `passport:client` với tùy chọn `--device`. Lệnh này sẽ tạo một client first-party được bật device flow và cung cấp cho bạn một client ID và secret:

```shell
php artisan passport:client --device
```

Ngoài ra, bạn có thể sử dụng phương thức `createDeviceAuthorizationGrantClient` trên class `ClientRepository` để đăng ký một client bên thứ ba thuộc về người dùng nhất định:

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

$client = app(ClientRepository::class)->createDeviceAuthorizationGrantClient(
    user: $user,
    name: 'Example Device',
    confidential: false,
);
```

<a name="requesting-device-authorization-grant-tokens"></a>
### Yêu cầu Tokens

<a name="device-code"></a>
#### Yêu cầu Device Code

Sau khi một client đã được tạo, các nhà phát triển có thể sử dụng client ID của họ để yêu cầu một device code từ ứng dụng của bạn. Đầu tiên, thiết bị tiêu thụ nên thực hiện một yêu cầu `POST` đến route `/oauth/device/code` của ứng dụng của bạn để yêu cầu một device code:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

Điều này sẽ trả về một phản hồi JSON chứa các thuộc tính `device_code`, `user_code`, `verification_uri`, `interval`, và `expires_in`. Thuộc tính `expires_in` chứa số giây cho đến khi device code hết hạn. Thuộc tính `interval` chứa số giây mà thiết bị tiêu thụ nên chờ giữa các yêu cầu khi polling route `/oauth/token` để tránh các lỗi giới hạn tốc độ.

> [!NOTE]
> Hãy nhớ rằng, route `/oauth/device/code` đã được định nghĩa bởi Passport. Bạn không cần định nghĩa thủ công route này.

<a name="user-code"></a>
#### Hiển thị Verification URI và User Code

Sau khi một yêu cầu device code đã được nhận, thiết bị tiêu thụ nên hướng dẫn người dùng sử dụng một thiết bị khác và truy cập `verification_uri` được cung cấp và nhập `user_code` để phê duyệt yêu cầu authorization.

<a name="polling-token-request"></a>
#### Polling Yêu cầu Token

Vì người dùng sẽ sử dụng một thiết bị riêng để cấp (hoặc từ chối) quyền truy cập, thiết bị tiêu thụ nên polling route `/oauth/token` của ứng dụng để xác định khi nào người dùng đã phản hồi yêu cầu. Thiết bị tiêu thụ nên sử dụng `interval` polling tối thiểu được cung cấp trong phản hồi JSON khi yêu cầu device code để tránh các lỗi giới hạn tốc độ:

```php
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Sleep;

$interval = 5;

do {
    Sleep::for($interval)->seconds();

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret', // Required for confidential clients only...
        'device_code' => 'the-device-code',
    ]);

    if ($response->json('error') === 'slow_down') {
        $interval += 5;
    }
} while (in_array($response->json('error'), ['authorization_pending', 'slow_down']));

return $response->json();
```

Nếu người dùng đã phê duyệt yêu cầu authorization, điều này sẽ trả về một phản hồi JSON chứa các thuộc tính `access_token`, `refresh_token`, và `expires_in`. Thuộc tính `expires_in` chứa số giây cho đến khi access token hết hạn.

<a name="password-grant"></a>
## Password Grant

> [!WARNING]
> Chúng tôi không còn khuyến nghị sử dụng password grant tokens. Thay vào đó, bạn nên chọn [một loại grant hiện được khuyến nghị bởi OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

OAuth2 password grant cho phép các client first-party khác của bạn, chẳng hạn như ứng dụng di động, để nhận được một access token bằng cách sử dụng địa chỉ email / tên người dùng và mật khẩu. Điều này cho phép bạn phát hành các access token một cách an toàn cho các client first-party của bạn mà không yêu cầu người dùng của bạn đi qua toàn bộ luồng chuyển hướng authorization code OAuth2.

Để bật password grant, hãy gọi phương thức `enablePasswordGrant` trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enablePasswordGrant();
}
```

<a name="creating-a-password-grant-client"></a>
### Tạo Password Grant Client

Trước khi ứng dụng của bạn có thể phát hành tokens thông qua password grant, bạn sẽ cần tạo một password grant client. Bạn có thể thực hiện việc này bằng cách sử dụng lệnh Artisan `passport:client` với tùy chọn `--password`.

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### Yêu cầu Tokens

Sau khi bạn đã bật grant và đã tạo một password grant client, bạn có thể yêu cầu một access token bằng cách phát hành một yêu cầu `POST` đến route `/oauth/token` với địa chỉ email và mật khẩu của người dùng. Hãy nhớ rằng, route này đã được đăng ký bởi Passport nên không cần định nghĩa nó thủ công. Nếu yêu cầu thành công, bạn sẽ nhận được một `access_token` và `refresh_token` trong phản hồi JSON từ máy chủ:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

> [!NOTE]
> Hãy nhớ rằng, access tokens tồn tại lâu theo mặc định. Tuy nhiên, bạn có thể tự do [cấu hình thời gian tồn tại access token tối đa của bạn](#configuration) nếu cần.

<a name="requesting-all-scopes"></a>
### Yêu cầu Tất cả Scopes

Khi sử dụng password grant hoặc client credentials grant, bạn có thể muốn ủy quyền token cho tất cả các scopes được hỗ trợ bởi ứng dụng của bạn. Bạn có thể thực hiện việc này bằng cách yêu cầu scope `*`. Nếu bạn yêu cầu scope `*`, phương thức `can` trên instance token sẽ luôn trả về `true`. Scope này chỉ có thể được gán cho một token được phát hành bằng cách sử dụng grant `password` hoặc `client_credentials`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '*',
]);
```

<a name="customizing-the-user-provider"></a>
### Tùy chỉnh User Provider

Nếu ứng dụng của bạn sử dụng nhiều hơn một [authentication user provider](/docs/{{version}}/authentication#introduction), bạn có thể chỉ định user provider nào mà password grant client sử dụng bằng cách cung cấp tùy chọn `--provider` khi tạo client thông qua lệnh `artisan passport:client --password`. Tên provider được cung cấp nên khớp với một provider hợp lệ được định nghĩa trong file cấu hình `config/auth.php` của ứng dụng. Sau đó bạn có thể [bảo vệ route của bạn bằng middleware](#multiple-authentication-guards) để đảm bảo rằng chỉ người dùng từ provider được chỉ định của guard được ủy quyền.

<a name="customizing-the-username-field"></a>
### Tùy chỉnh trường Username

Khi xác thực bằng cách sử dụng password grant, Passport sẽ sử dụng thuộc tính `email` của model có thể xác thực của bạn làm "username". Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách định nghĩa một phương thức `findForPassport` trên model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Bridge\Client;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Find the user instance for the given username.
     */
    public function findForPassport(string $username, Client $client): User
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### Tùy chỉnh Xác thực Mật khẩu

Khi xác thực bằng cách sử dụng password grant, Passport sẽ sử dụng thuộc tính `password` của model của bạn để xác thực mật khẩu đã cho. Nếu model của bạn không có thuộc tính `password` hoặc bạn muốn tùy chỉnh logic xác thực mật khẩu, bạn có thể định nghĩa một phương thức `validateForPassportPasswordGrant` trên model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Validate the password of the user for the Passport password grant.
     */
    public function validateForPassportPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant"></a>
## Implicit Grant

> [!WARNING]
> Chúng tôi không còn khuyến nghị sử dụng implicit grant tokens. Thay vào đó, bạn nên chọn [một loại grant hiện được khuyến nghị bởi OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

Implicit grant tương tự như authorization code grant; tuy nhiên, token được trả về cho client mà không cần trao đổi authorization code. Grant này thường được sử dụng nhất cho các ứng dụng JavaScript hoặc ứng dụng di động nơi thông tin xác thực của client không thể được lưu trữ một cách an toàn. Để bật grant, hãy gọi phương thức `enableImplicitGrant` trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

Trước khi ứng dụng của bạn có thể phát hành tokens thông qua implicit grant, bạn sẽ cần tạo một implicit grant client. Bạn có thể thực hiện việc này bằng cách sử dụng lệnh Artisan `passport:client` với tùy chọn `--implicit`.

```shell
php artisan passport:client --implicit
```

Sau khi grant đã được bật và một implicit client đã được tạo, các nhà phát triển có thể sử dụng client ID của họ để yêu cầu một access token từ ứng dụng của bạn. Ứng dụng tiêu thụ nên thực hiện một yêu cầu chuyển hướng đến route `/oauth/authorize` của ứng dụng của bạn như sau:

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'token',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

> [!NOTE]
> Hãy nhớ rằng, route `/oauth/authorize` đã được định nghĩa bởi Passport. Bạn không cần định nghĩa thủ công route này.

<a name="client-credentials-grant"></a>
## Client Credentials Grant

Client credentials grant phù hợp cho xác thực máy-đến-máy (machine-to-machine). Ví dụ, bạn có thể sử dụng grant này trong một scheduled job đang thực hiện các nhiệm vụ bảo trì qua một API.

Trước khi ứng dụng của bạn có thể phát hành tokens thông qua client credentials grant, bạn sẽ cần tạo một client credentials grant client. Bạn có thể thực hiện việc này bằng cách sử dụng tùy chọn `--client` của lệnh Artisan `passport:client`:

```shell
php artisan passport:client --client
```

Tiếp theo, gán middleware `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner` cho một route:

```php
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

Route::get('/orders', function (Request $request) {
    // Access token is valid and the client is resource owner...
})->middleware(EnsureClientIsResourceOwner::class);
```

Để hạn chế quyền truy cập vào route cho các scopes cụ thể, bạn có thể cung cấp danh sách các scopes cần thiết cho phương thức `using`:

```php
Route::get('/orders', function (Request $request) {
    // Access token is valid, the client is resource owner, and has both "servers:read" and "servers:create" scopes...
})->middleware(EnsureClientIsResourceOwner::using('servers:read', 'servers:create'));
```

> [!WARNING]
> [OAuth2 server cơ bản](https://oauth2.thephpleague.com/database-setup/#:~:text=Please%20note%20that,the%20bearer%20token.) đặt claim `sub` của token thành định danh của client cho các client credentials tokens. Theo mặc định, Passport sử dụng UUIDs cho clients, vì vậy điều này không thể xung đột với khóa chính số nguyên của người dùng. Tuy nhiên, nếu bạn đã đặt `Passport::$clientUuids` thành `false`, một client credentials token có thể vô tình giải quyết một người dùng có ID khớp với ID của client. Trong những trường hợp như vậy, sử dụng middleware này không thể đảm bảo rằng token đến là một client credentials token.

<a name="retrieving-tokens"></a>
### Truy xuất Tokens

Để truy xuất một token bằng cách sử dụng loại grant này, hãy thực hiện một yêu cầu đến endpoint `oauth/token`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'servers:read servers:create',
]);

return $response->json()['access_token'];
```

<a name="personal-access-tokens"></a>
## Personal Access Tokens

Đôi khi, người dùng của bạn có thể muốn phát hành access tokens cho chính họ mà không cần đi qua luồng chuyển hướng authorization code điển hình. Cho phép người dùng phát hành tokens cho chính họ thông qua UI của ứng dụng có thể hữu ích để cho phép người dùng thử nghiệm với API của bạn hoặc có thể phục vụ như một cách tiếp cận đơn giản hơn để phát hành access tokens nói chung.

> [!NOTE]
> Nếu ứng dụng của bạn sử dụng Passport chủ yếu để phát hành personal access tokens, hãy cân nhắc sử dụng [Laravel Sanctum](/docs/{{version}}/sanctum), thư viện first-party nhẹ của Laravel để phát hành API access tokens.

<a name="creating-a-personal-access-client"></a>
### Tạo Personal Access Client

Trước khi ứng dụng của bạn có thể phát hành personal access tokens, bạn sẽ cần tạo một personal access client. Bạn có thể thực hiện việc này bằng cách thực hiện lệnh Artisan `passport:client` với tùy chọn `--personal`. Nếu bạn đã chạy lệnh `passport:install`, bạn không cần chạy lệnh này:

```shell
php artisan passport:client --personal
```

<a name="customizing-the-user-provider-for-pat"></a>
### Tùy chỉnh User Provider

Nếu ứng dụng của bạn sử dụng nhiều hơn một [authentication user provider](/docs/{{version}}/authentication#introduction), bạn có thể chỉ định user provider nào mà personal access grant client sử dụng bằng cách cung cấp tùy chọn `--provider` khi tạo client thông qua lệnh `artisan passport:client --personal`. Tên provider được cung cấp nên khớp với một provider hợp lệ được định nghĩa trong file cấu hình `config/auth.php` của ứng dụng. Sau đó bạn có thể [bảo vệ route của bạn bằng middleware](#multiple-authentication-guards) để đảm bảo rằng chỉ người dùng từ provider được chỉ định của guard được ủy quyền.

<a name="managing-personal-access-tokens"></a>
### Quản lý Personal Access Tokens

Sau khi bạn đã tạo một personal access client, bạn có thể phát hành tokens cho một người dùng nhất định bằng cách sử dụng phương thức `createToken` trên instance model `App\Models\User`. Phương thức `createToken` chấp nhận tên của token làm đối số đầu tiên và một mảng tùy chọn của [scopes](#token-scopes) làm đối số thứ hai:

```php
use App\Models\User;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Creating a token without scopes...
$token = $user->createToken('My Token')->accessToken;

// Creating a token with scopes...
$token = $user->createToken('My Token', ['user:read', 'orders:create'])->accessToken;

// Creating a token with all scopes...
$token = $user->createToken('My Token', ['*'])->accessToken;

// Retrieving all the valid personal access tokens that belong to the user...
$tokens = $user->tokens()
    ->with('client')
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get()
    ->filter(fn (Token $token) => $token->client->hasGrantType('personal_access'));
```

<a name="protecting-routes"></a>
## Bảo vệ Routes

<a name="via-middleware"></a>
### Thông qua Middleware

Passport bao gồm một [authentication guard](/docs/{{version}}/authentication#adding-custom-guards) sẽ xác thực access tokens trên các yêu cầu đến. Sau khi bạn đã cấu hình guard `api` để sử dụng driver `passport`, bạn chỉ cần chỉ định middleware `auth:api` trên bất kỳ routes nào nên yêu cầu một access token hợp lệ:

```php
Route::get('/user', function () {
    // Only API authenticated users may access this route...
})->middleware('auth:api');
```

> [!WARNING]
> Nếu bạn đang sử dụng [client credentials grant](#client-credentials-grant), bạn nên sử dụng [middleware `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner`](#client-credentials-grant) để bảo vệ các routes của bạn thay vì middleware `auth:api`.

<a name="multiple-authentication-guards"></a>
#### Nhiều Authentication Guards

Nếu ứng dụng của bạn xác thực các loại người dùng khác nhau có thể sử dụng các model Eloquent hoàn toàn khác nhau, bạn có thể sẽ cần định nghĩa một cấu hình guard cho mỗi loại user provider trong ứng dụng. Điều này cho phép bạn bảo vệ các yêu cầu dành cho các user provider cụ thể. Ví dụ, với cấu hình guard sau trong file cấu hình `config/auth.php`:

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],
],
```

Route sau sẽ sử dụng guard `api-customers`, sử dụng user provider `customers`, để xác thực các yêu cầu đến:

```php
Route::get('/customer', function () {
    // ...
})->middleware('auth:api-customers');
```

> [!NOTE]
> Để biết thêm thông tin về việc sử dụng nhiều user providers với Passport, vui lòng tham khảo [tài liệu personal access tokens](#customizing-the-user-provider-for-pat) và [tài liệu password grant](#customizing-the-user-provider).

<a name="passing-the-access-token"></a>
### Truyền Access Token

Khi gọi các routes được bảo vệ bởi Passport, người tiêu thụ API của ứng dụng nên chỉ định access token của họ làm token `Bearer` trong header `Authorization` của yêu cầu. Ví dụ, khi sử dụng Facade `Http`:

```php
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => "Bearer $accessToken",
])->get('https://passport-app.test/api/user');

return $response->json();
```

<a name="token-scopes"></a>
## Token Scopes

Scopes cho phép các client API của bạn yêu cầu một tập hợp quyền cụ thể khi yêu cầu authorization để truy cập một tài khoản. Ví dụ, nếu bạn đang xây dựng một ứng dụng thương mại điện tử, không phải tất cả người tiêu thụ API đều cần khả năng đặt hàng. Thay vào đó, bạn có thể cho phép người tiêu thụ chỉ yêu cầu authorization để truy cập trạng thái vận chuyển đơn hàng. Nói cách khác, scopes cho phép người dùng của ứng dụng giới hạn các hành động mà một ứng dụng bên thứ ba có thể thực hiện thay mặt họ.

<a name="defining-scopes"></a>
### Định nghĩa Scopes

Bạn có thể định nghĩa các scopes của API bằng cách sử dụng phương thức `Passport::tokensCan` trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng. Phương thức `tokensCan` chấp nhận một mảng tên scope và mô tả scope. Mô tả scope có thể là bất cứ thứ gì bạn muốn và sẽ được hiển thị cho người dùng trên màn hình phê duyệt authorization:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensCan([
        'user:read' => 'Retrieve the user info',
        'orders:create' => 'Place orders',
        'orders:read:status' => 'Check order status',
    ]);
}
```

<a name="default-scope"></a>
### Scope mặc định

Nếu một client không yêu cầu bất kỳ scopes cụ thể nào, bạn có thể cấu hình server Passport của bạn để đính kèm các scopes mặc định vào token bằng cách sử dụng phương thức `defaultScopes`. Thông thường, bạn nên gọi phương thức này từ phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'user:read' => 'Retrieve the user info',
    'orders:create' => 'Place orders',
    'orders:read:status' => 'Check order status',
]);

Passport::defaultScopes([
    'user:read',
    'orders:create',
]);
```

<a name="assigning-scopes-to-tokens"></a>
### Gán Scopes cho Tokens

<a name="when-requesting-authorization-codes"></a>
#### Khi Yêu cầu Authorization Codes

Khi yêu cầu một access token bằng cách sử dụng authorization code grant, người tiêu thụ nên chỉ định các scopes mong muốn của họ làm tham số chuỗi truy vấn `scope`. Tham số `scope` nên là danh sách các scopes được phân tách bằng khoảng trắng:

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="when-issuing-personal-access-tokens"></a>
#### Khi Phát hành Personal Access Tokens

Nếu bạn đang phát hành personal access tokens bằng cách sử dụng phương thức `createToken` của model `App\Models\User`, bạn có thể truyền mảng các scopes mong muốn làm đối số thứ hai cho phương thức:

```php
$token = $user->createToken('My Token', ['orders:create'])->accessToken;
```

<a name="checking-scopes"></a>
### Kiểm tra Scopes

Passport bao gồm hai middleware có thể được sử dụng để xác minh rằng một yêu cầu đến được xác thực với một token đã được cấp một scope nhất định.

<a name="check-for-all-scopes"></a>
#### Kiểm tra Tất cả Scopes

Middleware `Laravel\Passport\Http\Middleware\CheckToken` có thể được gán cho một route để xác minh rằng access token của yêu cầu đến có tất cả các scopes được liệt kê:

```php
use Laravel\Passport\Http\Middleware\CheckToken;

Route::get('/orders', function () {
    // Access token has both "orders:read" and "orders:create" scopes...
})->middleware(['auth:api', CheckToken::using('orders:read', 'orders:create')]);
```

<a name="check-for-any-scopes"></a>
#### Kiểm tra Bất kỳ Scopes nào

Middleware `Laravel\Passport\Http\Middleware\CheckTokenForAnyScope` có thể được gán cho một route để xác minh rằng access token của yêu cầu đến có *ít nhất một* trong các scopes được liệt kê:

```php
use Laravel\Passport\Http\Middleware\CheckTokenForAnyScope;

Route::get('/orders', function () {
    // Access token has either "orders:read" or "orders:create" scope...
})->middleware(['auth:api', CheckTokenForAnyScope::using('orders:read', 'orders:create')]);
```

<a name="scope-attributes"></a>
#### Thuộc tính Scope

Nếu ứng dụng của bạn sử dụng [thuộc tính middleware controller](/docs/{{version}}/controllers#middleware-attributes), bạn có thể sử dụng thuộc tính `Laravel\Passport\Attributes\AuthorizeToken` làm một phím tắt thuận tiện cho middleware scope của Passport:

```php
<?php

namespace App\Http\Controllers;

use Laravel\Passport\Attributes\AuthorizeToken;

#[AuthorizeToken('orders:read')]
#[AuthorizeToken('orders:create', only: ['store'])]
class OrderController
{
    #[AuthorizeToken(['orders:read', 'orders:create'], anyScope: true)]
    public function index()
    {
        // Access token has either "orders:read" or "orders:create" scope...
    }

    public function store()
    {
        // Access token has both "orders:read" and "orders:create" scopes...
    }
}
```

Theo mặc định, thuộc tính `AuthorizeToken` yêu cầu tất cả các scopes đã cho. Nếu bạn truyền `anyScope: true`, yêu cầu được ủy quyền khi token có ít nhất một trong các scopes đã cho.

<a name="checking-scopes-on-a-token-instance"></a>
#### Kiểm tra Scopes trên một Instance Token

Sau khi một yêu cầu được xác thực bằng access token đã vào ứng dụng của bạn, bạn vẫn có thể kiểm tra xem token có một scope nhất định hay không bằng cách sử dụng phương thức `tokenCan` trên instance `App\Models\User` đã xác thực:

```php
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('orders:create')) {
        // ...
    }
});
```

<a name="additional-scope-methods"></a>
#### Các Phương thức Scope Bổ sung

Phương thức `scopeIds` sẽ trả về một mảng tất cả các ID / tên đã định nghĩa:

```php
use Laravel\Passport\Passport;

Passport::scopeIds();
```

Phương thức `scopes` sẽ trả về một mảng tất cả các scopes đã định nghĩa dưới dạng các instance của `Laravel\Passport\Scope`:

```php
Passport::scopes();
```

Phương thức `scopesFor` sẽ trả về một mảng các instance `Laravel\Passport\Scope` khớp với các ID / tên đã cho:

```php
Passport::scopesFor(['user:read', 'orders:create']);
```

Bạn có thể xác định xem một scope nhất định đã được định nghĩa hay chưa bằng cách sử dụng phương thức `hasScope`:

```php
Passport::hasScope('orders:create');
```

<a name="spa-authentication"></a>
## Xác thực SPA

Khi xây dựng một API, có thể cực kỳ hữu ích khi có thể tiêu thụ API của chính bạn từ ứng dụng JavaScript của bạn. Cách tiếp cận phát triển API này cho phép ứng dụng của chính bạn tiêu thụ cùng một API mà bạn đang chia sẻ với thế giới. Cùng một API có thể được tiêu thụ bởi ứng dụng web của bạn, ứng dụng di động, ứng dụng bên thứ ba, và bất kỳ SDK nào mà bạn có thể xuất bản trên các trình quản lý gói khác nhau.

Thông thường, nếu bạn muốn tiêu thụ API của bạn từ ứng dụng JavaScript, bạn sẽ cần gửi thủ công một access token đến ứng dụng và truyền nó với mỗi yêu cầu đến ứng dụng của bạn. Tuy nhiên, Passport bao gồm một middleware có thể xử lý việc này cho bạn. Tất cả những gì bạn cần làm là thêm middleware `CreateFreshApiToken` vào nhóm middleware `web` trong file `bootstrap/app.php` của ứng dụng:

```php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        CreateFreshApiToken::class,
    ]);
})
```

> [!WARNING]
> Bạn nên đảm bảo rằng middleware `CreateFreshApiToken` là middleware cuối cùng được liệt kê trong stack middleware của bạn.

Middleware này sẽ đính kèm một cookie `laravel_token` vào các phản hồi đi của bạn. Cookie này chứa một JWT được mã hóa mà Passport sẽ sử dụng để xác thực các yêu cầu API từ ứng dụng JavaScript của bạn. JWT có thời gian tồn tại bằng với giá trị cấu hình `session.lifetime` của bạn. Bây giờ, vì trình duyệt sẽ tự động gửi cookie với tất cả các yêu cầu tiếp theo, bạn có thể thực hiện các yêu cầu đến API của ứng dụng mà không cần truyền rõ ràng một access token:

```js
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

<a name="customizing-the-cookie-name"></a>
#### Tùy chỉnh Tên Cookie

Nếu cần, bạn có thể tùy chỉnh tên của cookie `laravel_token` bằng cách sử dụng phương thức `Passport::cookie`. Thông thường, phương thức này nên được gọi từ phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::cookie('custom_name');
}
```

<a name="csrf-protection"></a>
#### Bảo vệ CSRF

Khi sử dụng phương thức xác thực này, bạn sẽ cần đảm bảo một header token CSRF hợp lệ được bao gồm trong các yêu cầu của bạn. Khung JavaScript mặc định của Laravel được bao gồm với ứng dụng skeleton và tất cả các bộ khởi đầu bao gồm một instance [Axios](https://github.com/axios/axios), sẽ tự động sử dụng giá trị cookie `XSRF-TOKEN` được mã hóa để gửi một header `X-XSRF-TOKEN` trên các yêu cầu cùng nguồn gốc.

> [!NOTE]
> Nếu bạn chọn gửi header `X-CSRF-TOKEN` thay vì `X-XSRF-TOKEN`, bạn sẽ cần sử dụng token không được mã hóa được cung cấp bởi `csrf_token()`.

<a name="events"></a>
## Sự kiện

Passport kích hoạt các sự kiện khi phát hành access tokens và refresh tokens. Bạn có thể [lắng nghe các sự kiện này](/docs/{{version}}/events) để cắt bỏ hoặc thu hồi các access token khác trong cơ sở dữ liệu của bạn:

<div class="overflow-auto">

| Tên Sự kiện                                    |
| --------------------------------------------- |
| `Laravel\Passport\Events\AccessTokenCreated`  |
| `Laravel\Passport\Events\AccessTokenRevoked`  |
| `Laravel\Passport\Events\RefreshTokenCreated` |

</div>

<a name="testing"></a>
## Kiểm thử

Phương thức `actingAs` của Passport có thể được sử dụng để chỉ định người dùng đã xác thực hiện tại cũng như các scopes của họ. Đối số đầu tiên được đưa cho phương thức `actingAs` là instance người dùng và đối số thứ hai là một mảng các scopes nên được cấp cho token của người dùng:

```php tab=Pest
use App\Models\User;
use Laravel\Passport\Passport;

test('orders can be created', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Passport\Passport;

public function test_orders_can_be_created(): void
{
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
}
```

Phương thức `actingAsClient` của Passport có thể được sử dụng để chỉ định client đã xác thực hiện tại cũng như các scopes của họ. Đối số đầu tiên được đưa cho phương thức `actingAsClient` là instance client và đối số thứ hai là một mảng các scopes nên được cấp cho token của client:

```php tab=Pest
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

test('servers can be retrieved', function () {
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_servers_can_be_retrieved(): void
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
}
```

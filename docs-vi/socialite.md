# Laravel Socialite

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Nâng cấp Socialite](#upgrading-socialite)
- [Cấu hình](#configuration)
- [Xác thực](#authentication)
    - [Định tuyến](#routing)
    - [Xác thực và Lưu trữ](#authentication-and-storage)
    - [Phạm vi Truy cập](#access-scopes)
    - [Phạm vi Bot Slack](#slack-bot-scopes)
    - [Tham số Tùy chọn](#optional-parameters)
- [Lấy Thông tin Người dùng](#retrieving-user-details)
- [Kiểm thử](#testing)

<a name="introduction"></a>
## Giới thiệu

Ngoài xác thực dựa trên biểu mẫu thông thường, Laravel cũng cung cấp một cách đơn giản và thuận tiện để xác thực với các nhà cung cấp OAuth bằng [Laravel Socialite](https://github.com/laravel/socialite). Socialite hiện hỗ trợ xác thực qua Facebook, X, LinkedIn, Google, GitHub, GitLab, Bitbucket và Slack.

> [!NOTE]
> Bộ điều hợp cho các nền tảng khác có sẵn thông qua trang web [Socialite Providers](https://socialiteproviders.com/) do cộng đồng điều hành.

<a name="installation"></a>
## Cài đặt

Để bắt đầu với Socialite, hãy sử dụng trình quản lý gói Composer để thêm gói vào các phụ thuộc của dự án:

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## Nâng cấp Socialite

Khi nâng cấp lên phiên bản chính mới của Socialite, điều quan trọng là bạn phải xem xét kỹ [hướng dẫn nâng cấp](https://github.com/laravel/socialite/blob/master/UPGRADE.md).

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng Socialite, bạn sẽ cần thêm thông tin xác thực cho các nhà cung cấp OAuth mà ứng dụng của bạn sử dụng. Thông thường, các thông tin xác thực này có thể được lấy bằng cách tạo một "ứng dụng dành cho nhà phát triển" trong bảng điều khiển của dịch vụ mà bạn sẽ xác thực.

Các thông tin xác thực này nên được đặt trong tệp cấu hình `config/services.php` của ứng dụng và nên sử dụng khóa `facebook`, `x`, `linkedin-openid`, `google`, `github`, `gitlab`, `bitbucket`, `slack`, hoặc `slack-openid`, tùy thuộc vào các nhà cung cấp mà ứng dụng của bạn yêu cầu:

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => 'http://example.com/callback-url',
],
```

> [!NOTE]
> Nếu tùy chọn `redirect` chứa một đường dẫn tương đối, nó sẽ tự động được giải quyết thành một URL đầy đủ.

<a name="authentication"></a>
## Xác thực

<a name="routing"></a>
### Định tuyến

Để xác thực người dùng bằng nhà cung cấp OAuth, bạn sẽ cần hai route: một để chuyển hướng người dùng đến nhà cung cấp OAuth và một khác để nhận callback từ nhà cung cấp sau khi xác thực. Các route ví dụ dưới đây minh họa việc triển khai cả hai route:

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/redirect', function () {
    return Socialite::driver('github')->redirect();
});

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // $user->token
});
```

Phương thức `redirect` được cung cấp bởi facade `Socialite` sẽ xử lý việc chuyển hướng người dùng đến nhà cung cấp OAuth, trong khi phương thức `user` sẽ kiểm tra yêu cầu đến và lấy thông tin người dùng từ nhà cung cấp sau khi họ đã phê duyệt yêu cầu xác thực.

<a name="authentication-and-storage"></a>
### Xác thực và Lưu trữ

Sau khi người dùng đã được lấy từ nhà cung cấp OAuth, bạn có thể xác định xem người dùng có tồn tại trong cơ sở dữ liệu của ứng dụng hay không và [xác thực người dùng](/docs/{{version}}/authentication#authenticate-a-user-instance). Nếu người dùng không tồn tại trong cơ sở dữ liệu của ứng dụng, bạn thường sẽ tạo một bản ghi mới trong cơ sở dữ liệu để đại diện cho người dùng:

```php
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Laravel\Socialite\Socialite;

Route::get('/auth/callback', function () {
    $githubUser = Socialite::driver('github')->user();

    $user = User::updateOrCreate([
        'github_id' => $githubUser->id,
    ], [
        'name' => $githubUser->name,
        'email' => $githubUser->email,
        'github_token' => $githubUser->token,
        'github_refresh_token' => $githubUser->refreshToken,
    ]);

    Auth::login($user);

    return redirect('/dashboard');
});
```

> [!NOTE]
> Để biết thêm thông tin về những thông tin người dùng nào có sẵn từ các nhà cung cấp OAuth cụ thể, vui lòng tham khảo tài liệu về [lấy thông tin người dùng](#retrieving-user-details).

<a name="access-scopes"></a>
### Phạm vi Truy cập

Trước khi chuyển hướng người dùng, bạn có thể sử dụng phương thức `scopes` để chỉ định các "phạm vi" nên được bao gồm trong yêu cầu xác thực. Phương thức này sẽ hợp nhất tất cả các phạm vi đã chỉ định trước đó với các phạm vi mà bạn chỉ định:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('github')
    ->scopes(['read:user', 'public_repo'])
    ->redirect();
```

Bạn có thể ghi đè tất cả các phạm vi hiện có trên yêu cầu xác thực bằng phương thức `setScopes`:

```php
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

<a name="slack-bot-scopes"></a>
### Phạm vi Bot Slack

API của Slack cung cấp [các loại token truy cập khác nhau](https://api.slack.com/authentication/token-types), mỗi loại có bộ [phạm vi quyền](https://api.slack.com/scopes) riêng. Socialite tương thích với cả hai loại token truy cập Slack sau:

<div class="content-list" markdown="1">

- Bot (có tiền tố `xoxb-`)
- User (có tiền tố `xoxp-`)

</div>

Theo mặc định, driver `slack` sẽ tạo một token `user` và việc gọi phương thức `user` của driver sẽ trả về chi tiết người dùng.

Token bot chủ yếu hữu ích nếu ứng dụng của bạn sẽ gửi thông báo đến các không gian làm việc Slack bên ngoài thuộc sở hữu của người dùng ứng dụng. Để tạo token bot, hãy gọi phương thức `asBotUser` trước khi chuyển hướng người dùng đến Slack để xác thực:

```php
return Socialite::driver('slack')
    ->asBotUser()
    ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
    ->redirect();
```

Ngoài ra, bạn phải gọi phương thức `asBotUser` trước khi gọi phương thức `user` sau khi Slack chuyển hướng người dùng trở lại ứng dụng của bạn sau khi xác thực:

```php
$user = Socialite::driver('slack')->asBotUser()->user();
```

Khi tạo token bot, phương thức `user` vẫn sẽ trả về một thể hiện `Laravel\Socialite\Two\User`; tuy nhiên, chỉ có thuộc tính `token` sẽ được điền dữ liệu. Token này có thể được lưu trữ để [gửi thông báo đến các không gian làm việc Slack của người dùng đã xác thực](/docs/{{version}}/notifications#notifying-external-slack-workspaces).

<a name="optional-parameters"></a>
### Tham số Tùy chọn

Một số nhà cung cấp OAuth hỗ trợ các tham số tùy chọn khác trên yêu cầu chuyển hướng. Để bao gồm bất kỳ tham số tùy chọn nào trong yêu cầu, hãy gọi phương thức `with` với một mảng kết hợp:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

> [!WARNING]
> Khi sử dụng phương thức `with`, hãy cẩn thận không chuyển bất kỳ từ khóa dành riêng nào như `state` hoặc `response_type`.

<a name="retrieving-user-details"></a>
## Lấy Thông tin Người dùng

Sau khi người dùng được chuyển hướng trở lại route callback xác thực của ứng dụng, bạn có thể lấy chi tiết người dùng bằng phương thức `user` của Socialite. Đối tượng người dùng được trả về bởi phương thức `user` cung cấp nhiều thuộc tính và phương thức khác nhau mà bạn có thể sử dụng để lưu trữ thông tin về người dùng trong cơ sở dữ liệu của riêng mình.

Các thuộc tính và phương thức khác nhau có thể có sẵn trên đối tượng này tùy thuộc vào việc nhà cung cấp OAuth mà bạn đang xác thực có hỗ trợ OAuth 1.0 hay OAuth 2.0:

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // OAuth 2.0 providers...
    $token = $user->token;
    $refreshToken = $user->refreshToken;
    $expiresIn = $user->expiresIn;

    // OAuth 1.0 providers...
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All providers...
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();
});
```

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### Lấy Thông tin Người dùng Từ Token

Nếu bạn đã có một token truy cập hợp lệ cho người dùng, bạn có thể lấy chi tiết người dùng của họ bằng phương thức `userFromToken` của Socialite:

```php
use Laravel\Socialite\Socialite;

$user = Socialite::driver('github')->userFromToken($token);
```

Nếu bạn đang sử dụng Facebook Limited Login thông qua ứng dụng iOS, Facebook sẽ trả về một token OIDC thay vì token truy cập. Giống như token truy cập, token OIDC có thể được cung cấp cho phương thức `userFromToken` để lấy chi tiết người dùng.

<a name="stateless-authentication"></a>
#### Xác thực Không trạng thái

Phương thức `stateless` có thể được sử dụng để tắt xác minh trạng thái phiên. Điều này hữu ích khi thêm xác thực xã hội vào API không trạng thái không sử dụng phiên dựa trên cookie:

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')->stateless()->user();
```

<a name="testing"></a>
## Kiểm thử

Laravel Socialite cung cấp một cách thuận tiện để kiểm tra các luồng xác thực OAuth mà không thực hiện yêu cầu thực tế đến các nhà cung cấp OAuth. Phương thức `fake` cho phép bạn giả lập hành vi của nhà cung cấp OAuth và định nghĩa dữ liệu người dùng nên được trả về.

<a name="faking-the-redirect"></a>
#### Giả lập Chuyển hướng

Để kiểm tra xem ứng dụng của bạn có chuyển hướng người dùng đến nhà cung cấp OAuth đúng cách hay không, bạn có thể gọi phương thức `fake` trước khi thực hiện yêu cầu đến route chuyển hướng của mình. Điều này sẽ khiến Socialite trả về chuyển hướng đến một URL ủy quyền giả thay vì chuyển hướng đến nhà cung cấp OAuth thực tế:

```php
use Laravel\Socialite\Socialite;

test('user is redirected to github', function () {
    Socialite::fake('github');

    $response = $this->get('/auth/github/redirect');

    $response->assertRedirect();
});
```

<a name="faking-the-callback"></a>
#### Giả lập Callback

Để kiểm tra route callback của ứng dụng, bạn có thể gọi phương thức `fake` và cung cấp một thể hiện `User` nên được trả về khi ứng dụng của bạn yêu cầu chi tiết người dùng từ nhà cung cấp. Thể hiện `User` có thể được tạo bằng phương thức `map`:

```php
use Laravel\Socialite\Socialite;
use Laravel\Socialite\Two\User;

test('user can login with github', function () {
    Socialite::fake('github', (new User)->map([
        'id' => 'github-123',
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
    ]));

    $response = $this->get('/auth/github/callback');

    $response->assertRedirect('/dashboard');

    $this->assertDatabaseHas('users', [
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
        'github_id' => 'github-123',
    ]);
});
```

Theo mặc định, thể hiện `User` cũng sẽ bao gồm thuộc tính `token`. Nếu cần, bạn có thể chỉ định thủ công các thuộc tính bổ sung trên thể hiện `User`:

```php
$fakeUser = (new User)->map([
    'id' => 'github-123',
    'name' => 'Jason Beggs',
    'email' => 'jason@example.com',
])->setToken('fake-token')
  ->setRefreshToken('fake-refresh-token')
  ->setExpiresIn(3600)
  ->setApprovedScopes(['read', 'write'])
```

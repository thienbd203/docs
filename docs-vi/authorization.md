# Authorization

- [Giới thiệu](#introduction)
- [Gates](#gates)
    - [Viết Gates](#writing-gates)
    - [Xác thực Actions](#authorizing-actions-via-gates)
    - [Gate Responses](#gate-responses)
    - [Chặn Gate Checks](#intercepting-gate-checks)
    - [Inline Authorization](#inline-authorization)
- [Tạo Policies](#creating-policies)
    - [Tạo Policies](#generating-policies)
    - [Đăng ký Policies](#registering-policies)
- [Viết Policies](#writing-policies)
    - [Policy Methods](#policy-methods)
    - [Policy Responses](#policy-responses)
    - [Methods Without Models](#methods-without-models)
    - [Guest Users](#guest-users)
    - [Policy Filters](#policy-filters)
- [Xác thực Actions bằng Policies](#authorizing-actions-using-policies)
    - [Thông qua User Model](#via-the-user-model)
    - [Thông qua Gate Facade](#via-the-gate-facade)
    - [Thông qua Middleware](#via-middleware)
    - [Thông qua Blade Templates](#via-blade-templates)
    - [Cung cấp Additional Context](#supplying-additional-context)
- [Authorization & Inertia](#authorization-and-inertia)

<a name="introduction"></a>
## Giới thiệu

Ngoài việc cung cấp các dịch vụ [authentication](/docs/{{version}}/authentication) tích hợp sẵn, Laravel cũng cung cấp một cách đơn giản để ủy quyền các hành động của người dùng đối với một tài nguyên nhất định. Ví dụ, mặc dù người dùng đã được xác thực, họ có thể không được ủy quyền để cập nhật hoặc xóa một số Eloquent models hoặc database records được quản lý bởi ứng dụng của bạn. Các tính năng authorization của Laravel cung cấp một cách dễ dàng, có tổ chức để quản lý các loại kiểm tra authorization này.

Laravel cung cấp hai cách chính để ủy quyền các hành động: [gates](#gates) và [policies](#creating-policies). Hãy coi gates và policies như routes và controllers. Gates cung cấp một cách tiếp cận đơn giản dựa trên closure cho authorization trong khi policies, giống như controllers, nhóm logic xung quanh một model hoặc tài nguyên cụ thể. Trong tài liệu này, chúng ta sẽ khám phá gates trước và sau đó xem xét policies.

Bạn không cần phải chọn giữa việc chỉ sử dụng gates hoặc chỉ sử dụng policies khi xây dựng một ứng dụng. Hầu hết các ứng dụng có khả năng sẽ chứa một sự kết hợp của gates và policies, và điều đó hoàn toàn ổn! Gates phù hợp nhất cho các hành động không liên quan đến bất kỳ model hoặc tài nguyên nào, chẳng hạn như xem một dashboard quản trị viên. Ngược lại, policies nên được sử dụng khi bạn muốn ủy quyền một hành động cho một model hoặc tài nguyên cụ thể.

<a name="gates"></a>
## Gates

<a name="writing-gates"></a>
### Viết Gates

> [!WARNING]
> Gates là một cách tuyệt vời để tìm hiểu các tính năng cơ bản của authorization của Laravel; tuy nhiên, khi xây dựng các ứng dụng Laravel mạnh mẽ, bạn nên cân nhắc sử dụng [policies](#creating-policies) để tổ chức các quy tắc authorization của bạn.

Gates đơn giản là các closures xác định xem người dùng có được ủy quyền để thực hiện một hành động nhất định hay không. Thông thường, gates được định nghĩa trong phương thức `boot` của class `App\Providers\AppServiceProvider` bằng cách sử dụng facade `Gate`. Gates luôn nhận một user instance làm đối số đầu tiên và có thể tùy ý nhận các đối số bổ sung như một Eloquent model liên quan.

Trong ví dụ này, chúng ta sẽ định nghĩa một gate để xác định xem người dùng có thể cập nhật một model `App\Models\Post` nhất định hay không. Gate sẽ thực hiện điều này bằng cách so sánh `id` của người dùng với `user_id` của người dùng đã tạo bài viết:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });
}
```

Giống như controllers, gates cũng có thể được định nghĩa bằng cách sử dụng một mảng callback class:

```php
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('update-post', [PostPolicy::class, 'update']);
}
```

<a name="authorizing-actions-via-gates"></a>
### Xác thực Actions

Để ủy quyền một hành động bằng cách sử dụng gates, bạn nên sử dụng các phương thức `allows` hoặc `denies` được cung cấp bởi facade `Gate`. Lưu ý rằng bạn không bắt buộc phải truyền người dùng hiện tại đã xác thực cho các phương thức này. Laravel sẽ tự động lo việc truyền người dùng vào gate closure. Việc gọi các phương thức authorization gate trong các controllers của ứng dụng trước khi thực hiện một hành động yêu cầu authorization là phổ biến:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * Update the given post.
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if (! Gate::allows('update-post', $post)) {
            abort(403);
        }

        // Update the post...

        return redirect('/posts');
    }
}
```

Nếu bạn muốn xác định xem một người dùng khác với người dùng hiện tại đã xác thực có được ủy quyền để thực hiện một hành động hay không, bạn có thể sử dụng phương thức `forUser` trên facade `Gate`:

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // The user can update the post...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // The user can't update the post...
}
```

Bạn có thể ủy quyền nhiều hành động cùng một lúc bằng cách sử dụng các phương thức `any` hoặc `none`:

```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // The user can update or delete the post...
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // The user can't update or delete the post...
}
```

<a name="authorizing-or-throwing-exceptions"></a>
#### Xác thực hoặc Throwing Exceptions

Nếu bạn muốn cố gắng ủy quyền một hành động và tự động throw một `Illuminate\Auth\Access\AuthorizationException` nếu người dùng không được phép thực hiện hành động đã cho, bạn có thể sử dụng phương thức `authorize` của facade `Gate`. Các instance của `AuthorizationException` sẽ tự động được chuyển đổi thành một HTTP response 403 bởi Laravel:

```php
Gate::authorize('update-post', $post);

// The action is authorized...
```

<a name="gates-supplying-additional-context"></a>
#### Cung cấp Additional Context

Các phương thức gate để ủy quyền abilities (`allows`, `denies`, `check`, `any`, `none`, `authorize`, `can`, `cannot`) và các [Blade directives](#via-blade-templates) authorization (`@can`, `@cannot`, `@canany`) có thể nhận một mảng làm đối số thứ hai của chúng. Các phần tử mảng này được truyền làm tham số cho gate closure, và có thể được sử dụng cho additional context khi đưa ra quyết định authorization:

```php
use App\Models\Category;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
    if (! $user->canPublishToGroup($category->group)) {
        return false;
    } elseif ($pinned && ! $user->canPinPosts()) {
        return false;
    }

    return true;
});

if (Gate::check('create-post', [$category, $pinned])) {
    // The user can create the post...
}
```

<a name="gate-responses"></a>
### Gate Responses

Cho đến nay, chúng ta chỉ đã xem xét các gates trả về các giá trị boolean đơn giản. Tuy nhiên, đôi khi bạn có thể muốn trả về một response chi tiết hơn, bao gồm cả thông báo lỗi. Để làm điều này, bạn có thể trả về một `Illuminate\Auth\Access\Response` từ gate của bạn:

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('You must be an administrator.');
});
```

Ngay cả khi bạn trả về một authorization response từ gate của bạn, phương thức `Gate::allows` vẫn sẽ trả về một giá trị boolean đơn giản; tuy nhiên, bạn có thể sử dụng phương thức `Gate::inspect` để nhận được authorization response đầy đủ được trả về bởi gate:

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
```

Khi sử dụng phương thức `Gate::authorize`, sẽ throw một `AuthorizationException` nếu hành động không được ủy quyền, thông báo lỗi được cung cấp bởi authorization response sẽ được truyền đến HTTP response:

```php
Gate::authorize('edit-settings');

// The action is authorized...
```

<a name="customizing-gate-response-status"></a>
#### Tùy chỉnh HTTP Response Status

Khi một hành động bị từ chối thông qua một Gate, một HTTP response `403` được trả về; tuy nhiên, đôi khi có thể hữu ích để trả về một mã trạng thái HTTP thay thế. Bạn có thể tùy chỉnh mã trạng thái HTTP được trả về cho một kiểm tra authorization thất bại bằng cách sử dụng static constructor `denyWithStatus` trên class `Illuminate\Auth\Access\Response`:

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyWithStatus(404);
});
```

Vì việc ẩn tài nguyên thông qua một response `404` là một pattern rất phổ biến cho các ứng dụng web, phương thức `denyAsNotFound` được cung cấp để thuận tiện:

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyAsNotFound();
});
```

<a name="intercepting-gate-checks"></a>
### Chặn Gate Checks

Đôi khi, bạn có thể muốn cấp tất cả các abilities cho một người dùng cụ thể. Bạn có thể sử dụng phương thức `before` để định nghĩa một closure được chạy trước tất cả các kiểm tra authorization khác:

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

Nếu closure `before` trả về một kết quả không null, kết quả đó sẽ được coi là kết quả của kiểm tra authorization.

Bạn có thể sử dụng phương thức `after` để định nghĩa một closure được thực thi sau tất cả các kiểm tra authorization khác:

```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

Các giá trị được trả về bởi các closure `after` sẽ không ghi đè kết quả của kiểm tra authorization trừ khi gate hoặc policy trả về `null`.

<a name="inline-authorization"></a>
### Inline Authorization

Thỉnh thoảng, bạn có thể muốn xác định xem người dùng hiện tại đã xác thực có được ủy quyền để thực hiện một hành động nhất định mà không cần viết một gate chuyên dụng tương ứng với hành động đó. Laravel cho phép bạn thực hiện các loại kiểm tra authorization "inline" này thông qua các phương thức `Gate::allowIf` và `Gate::denyIf`. Inline authorization không thực hiện bất kỳ ["before" hoặc "after" authorization hooks](#intercepting-gate-checks) nào đã được định nghĩa:

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::allowIf(fn (User $user) => $user->isAdministrator());

Gate::denyIf(fn (User $user) => $user->banned());
```

Nếu hành động không được ủy quyền hoặc nếu không có người dùng nào hiện tại được xác thực, Laravel sẽ tự động throw một exception `Illuminate\Auth\Access\AuthorizationException`. Các instance của `AuthorizationException` sẽ tự động được chuyển đổi thành một HTTP response 403 bởi exception handler của Laravel.

<a name="creating-policies"></a>
## Tạo Policies

<a name="generating-policies"></a>
### Tạo Policies

Policies là các class tổ chức logic authorization xung quanh một model hoặc tài nguyên cụ thể. Ví dụ, nếu ứng dụng của bạn là một blog, bạn có thể có một model `App\Models\Post` và một `App\Policies\PostPolicy` tương ứng để ủy quyền các hành động của người dùng như tạo hoặc cập nhật bài viết.

Bạn có thể tạo một policy bằng cách sử dụng lệnh Artisan `make:policy`. Policy được tạo sẽ được đặt trong thư mục `app/Policies`. Nếu thư mục này không tồn tại trong ứng dụng của bạn, Laravel sẽ tạo nó cho bạn:

```shell
php artisan make:policy PostPolicy
```

Lệnh `make:policy` sẽ tạo một class policy trống. Nếu bạn muốn tạo một class với các phương thức policy ví dụ liên quan đến việc xem, tạo, cập nhật và xóa tài nguyên, bạn có thể cung cấp một tùy chọn `--model` khi thực hiện lệnh:

```shell
php artisan make:policy PostPolicy --model=Post
```

<a name="registering-policies"></a>
### Đăng ký Policies

<a name="policy-discovery"></a>
#### Policy Discovery

Theo mặc định, Laravel tự động phát hiện policies miễn là model và policy tuân theo các quy ước đặt tên tiêu chuẩn của Laravel. Cụ thể, policies phải nằm trong một thư mục `Policies` tại hoặc trên thư mục chứa các models của bạn. Vì vậy, ví dụ, các models có thể được đặt trong thư mục `app/Models` trong khi các policies có thể được đặt trong thư mục `app/Policies`. Trong tình huống này, Laravel sẽ kiểm tra các policies trong `app/Models/Policies` sau đó `app/Policies`. Ngoài ra, tên policy phải khớp với tên model và có hậu tố `Policy`. Vì vậy, một model `User` sẽ tương ứng với một class policy `UserPolicy`.

Nếu bạn muốn định nghĩa logic discovery policy của riêng mình, bạn có thể đăng ký một callback discovery policy tùy chỉnh bằng cách sử dụng phương thức `Gate::guessPolicyNamesUsing`. Thông thường, phương thức này nên được gọi từ phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // Return the name of the policy class for the given model...
});
```

<a name="manually-registering-policies"></a>
#### Đăng ký Policies Thủ công

Sử dụng facade `Gate`, bạn có thể đăng ký thủ công các policies và các models tương ứng của chúng trong phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
use App\Models\Order;
use App\Policies\OrderPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::policy(Order::class, OrderPolicy::class);
}
```

Ngoài ra, bạn có thể đặt attribute `UsePolicy` trên một class model để thông báo cho Laravel về policy tương ứng của model:

```php
<?php

namespace App\Models;

use App\Policies\OrderPolicy;
use Illuminate\Database\Eloquent\Attributes\UsePolicy;
use Illuminate\Database\Eloquent\Model;

#[UsePolicy(OrderPolicy::class)]
class Order extends Model
{
    //
}
```

<a name="writing-policies"></a>
## Viết Policies

<a name="policy-methods"></a>
### Policy Methods

Khi class policy đã được đăng ký, bạn có thể thêm các phương thức cho mỗi hành động mà nó ủy quyền. Ví dụ, hãy định nghĩa một phương thức `update` trên `PostPolicy` của chúng ta xác định xem một `App\Models\User` nhất định có thể cập nhật một instance `App\Models\Post` nhất định hay không.

Phương thức `update` sẽ nhận một `User` và một instance `Post` làm đối số của nó, và nên trả về `true` hoặc `false` chỉ định xem người dùng có được ủy quyền để cập nhật `Post` đã cho hay không. Vì vậy, trong ví dụ này, chúng ta sẽ xác minh rằng `id` của người dùng khớp với `user_id` trên bài viết:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

Bạn có thể tiếp tục định nghĩa các phương thức bổ sung trên policy theo nhu cầu cho các hành động khác nhau mà nó ủy quyền. Ví dụ, bạn có thể định nghĩa các phương thức `view` hoặc `delete` để ủy quyền các hành động liên quan đến `Post` khác nhau, nhưng hãy nhớ rằng bạn có thể tự do đặt tên cho các phương thức policy của bạn theo bất kỳ cách nào bạn thích.

Nếu bạn đã sử dụng tùy chọn `--model` khi tạo policy của bạn thông qua Artisan console, nó sẽ đã chứa các phương thức cho các hành động `viewAny`, `view`, `create`, `update`, `delete`, `restore`, và `forceDelete`.

> [!NOTE]
> Tất cả các policies được giải quyết thông qua [service container](/docs/{{version}}/container) của Laravel, cho phép bạn type-hint bất kỳ dependencies cần thiết nào trong constructor của policy để chúng được tự động inject.

<a name="policy-responses"></a>
### Policy Responses

Cho đến nay, chúng ta chỉ đã xem xét các phương thức policy trả về các giá trị boolean đơn giản. Tuy nhiên, đôi khi bạn có thể muốn trả về một response chi tiết hơn, bao gồm cả thông báo lỗi. Để làm điều này, bạn có thể trả về một instance `Illuminate\Auth\Access\Response` từ phương thức policy của bạn:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Determine if the given post can be updated by the user.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::deny('You do not own this post.');
}
```

Khi trả về một authorization response từ policy của bạn, phương thức `Gate::allows` vẫn sẽ trả về một giá trị boolean đơn giản; tuy nhiên, bạn có thể sử dụng phương thức `Gate::inspect` để nhận được authorization response đầy đủ được trả về bởi gate:

```php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
```

Khi sử dụng phương thức `Gate::authorize`, sẽ throw một `AuthorizationException` nếu hành động không được ủy quyền, thông báo lỗi được cung cấp bởi authorization response sẽ được truyền đến HTTP response:

```php
Gate::authorize('update', $post);

// The action is authorized...
```

<a name="customizing-policy-response-status"></a>
#### Tùy chỉnh HTTP Response Status

Khi một hành động bị từ chối thông qua một phương thức policy, một HTTP response `403` được trả về; tuy nhiên, đôi khi có thể hữu ích để trả về một mã trạng thái HTTP thay thế. Bạn có thể tùy chỉnh mã trạng thái HTTP được trả về cho một kiểm tra authorization thất bại bằng cách sử dụng static constructor `denyWithStatus` trên class `Illuminate\Auth\Access\Response`:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Determine if the given post can be updated by the user.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyWithStatus(404);
}
```

Vì việc ẩn tài nguyên thông qua một response `404` là một pattern rất phổ biến cho các ứng dụng web, phương thức `denyAsNotFound` được cung cấp để thuận tiện:

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Determine if the given post can be updated by the user.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyAsNotFound();
}
```

<a name="methods-without-models"></a>
### Methods Without Models

Một số phương thức policy chỉ nhận một instance của người dùng hiện tại đã xác thực. Tình huống này phổ biến nhất khi ủy quyền các hành động `create`. Ví dụ, nếu bạn đang tạo một blog, bạn có thể muốn xác định xem người dùng có được ủy quyền để tạo bất kỳ bài viết nào không. Trong những tình huống này, phương thức policy của bạn chỉ nên mong đợi nhận một user instance:

```php
/**
 * Determine if the given user can create posts.
 */
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```

<a name="guest-users"></a>
### Guest Users

Theo mặc định, tất cả các gates và policies tự động trả về `false` nếu HTTP request đến không được khởi tạo bởi một người dùng đã xác thực. Tuy nhiên, bạn có thể cho phép các kiểm tra authorization này truyền qua đến gates và policies của bạn bằng cách khai báo một type-hint "optional" hoặc cung cấp một giá trị mặc định `null` cho định nghĩa đối số người dùng:

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     */
    public function update(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

<a name="policy-filters"></a>
### Policy Filters

Đối với một số người dùng nhất định, bạn có thể muốn ủy quyền tất cả các hành động trong một policy nhất định. Để thực hiện điều này, định nghĩa một phương thức `before` trên policy. Phương thức `before` sẽ được thực thi trước bất kỳ phương thức nào khác trên policy, cho bạn cơ hội ủy quyền hành động trước khi phương thức policy dự định thực sự được gọi. Tính năng này thường được sử dụng phổ biến nhất để ủy quyền cho các quản trị viên ứng dụng thực hiện bất kỳ hành động nào:

```php
use App\Models\User;

/**
 * Perform pre-authorization checks.
 */
public function before(User $user, string $ability): bool|null
{
    if ($user->isAdministrator()) {
        return true;
    }

    return null;
}
```

Nếu bạn muốn từ chối tất cả các kiểm tra authorization cho một loại người dùng cụ thể thì bạn có thể trả về `false` từ phương thức `before`. Nếu `null` được trả về, kiểm tra authorization sẽ chuyển đến phương thức policy.

> [!WARNING]
> Phương thức `before` của một class policy sẽ không được gọi nếu class không chứa một phương thức với tên khớp với tên của ability đang được kiểm tra.

<a name="authorizing-actions-using-policies"></a>
## Xác thực Actions bằng Policies

<a name="via-the-user-model"></a>
### Thông qua User Model

Model `App\Models\User` được bao gồm với ứng dụng Laravel của bạn bao gồm hai phương thức hữu ích để ủy quyền các hành động: `can` và `cannot`. Các phương thức `can` và `cannot` nhận tên của hành động bạn muốn ủy quyền và model liên quan. Ví dụ, hãy xác định xem người dùng có được ủy quyền để cập nhật một model `App\Models\Post` nhất định hay không. Thông thường, điều này sẽ được thực hiện trong một phương thức controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Update the given post.
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }

        // Update the post...

        return redirect('/posts');
    }
}
```

Nếu một [policy được đăng ký](#registering-policies) cho model đã cho, phương thức `can` sẽ tự động gọi policy thích hợp và trả về kết quả boolean. Nếu không có policy nào được đăng ký cho model, phương thức `can` sẽ cố gắng gọi Gate dựa trên closure khớp với tên hành động đã cho.

<a name="user-model-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Hãy nhớ rằng, một số hành động có thể tương ứng với các phương thức policy như `create` không yêu cầu một model instance. Trong những tình huống này, bạn có thể truyền một tên class cho phương thức `can`. Tên class sẽ được sử dụng để xác định policy nào sẽ sử dụng khi ủy quyền hành động:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Create a post.
     */
    public function store(Request $request): RedirectResponse
    {
        if ($request->user()->cannot('create', Post::class)) {
            abort(403);
        }

        // Create the post...

        return redirect('/posts');
    }
}
```

<a name="via-the-gate-facade"></a>
### Thông qua `Gate` Facade

Ngoài các phương thức hữu ích được cung cấp cho model `App\Models\User`, bạn luôn có thể ủy quyền các hành động thông qua phương thức `authorize` của facade `Gate`.

Giống như phương thức `can`, phương thức này chấp nhận tên của hành động bạn muốn ủy quyền và model liên quan. Nếu hành động không được ủy quyền, phương thức `authorize` sẽ throw một exception `Illuminate\Auth\Access\AuthorizationException` mà exception handler của Laravel sẽ tự động chuyển đổi thành một HTTP response với mã trạng thái 403:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * Update the given blog post.
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        Gate::authorize('update', $post);

        // The current user can update the blog post...

        return redirect('/posts');
    }
}
```

<a name="controller-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Như đã thảo luận trước đó, một số phương thức policy như `create` không yêu cầu một model instance. Trong những tình huống này, bạn nên truyền một tên class cho phương thức `authorize`. Tên class sẽ được sử dụng để xác định policy nào sẽ sử dụng khi ủy quyền hành động:

```php
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

/**
 * Create a new blog post.
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request): RedirectResponse
{
    Gate::authorize('create', Post::class);

    // The current user can create blog posts...

    return redirect('/posts');
}
```

<a name="via-middleware"></a>
### Thông qua Middleware

Laravel bao gồm một middleware có thể ủy quyền các hành động trước khi request đến thậm chí đến routes hoặc controllers của bạn. Theo mặc định, middleware `Illuminate\Auth\Middleware\Authorize` có thể được đính kèm vào một route bằng cách sử dụng [middleware alias](/docs/{{version}}/middleware#middleware-aliases) `can`, được đăng ký tự động bởi Laravel. Hãy khám phá một ví dụ về việc sử dụng middleware `can` để ủy quyền rằng người dùng có thể cập nhật một bài viết:

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');
```

Trong ví dụ này, chúng ta đang truyền hai đối số cho middleware `can`. Đối số đầu tiên là tên của hành động chúng ta muốn ủy quyền và đối số thứ hai là route parameter chúng ta muốn truyền cho phương thức policy. Trong trường hợp này, vì chúng ta đang sử dụng [implicit model binding](/docs/{{version}}/routing#implicit-binding), một model `App\Models\Post` sẽ được truyền cho phương thức policy. Nếu người dùng không được ủy quyền để thực hiện hành động đã cho, một HTTP response với mã trạng thái 403 sẽ được trả về bởi middleware.

Để thuận tiện, bạn cũng có thể đính kèm middleware `can` vào route của bạn bằng cách sử dụng phương thức `can`:

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->can('update', 'post');
```

Nếu bạn đang sử dụng [controller middleware attributes](/docs/{{version}}/controllers#middleware-attributes), bạn có thể áp dụng middleware `can` thông qua attribute `Authorize`:

```php
use Illuminate\Routing\Attributes\Controllers\Authorize;

#[Authorize('update', 'post')]
public function update(Post $post)
{
    // The current user may update the post...
}
```

<a name="middleware-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Một lần nữa, một số phương thức policy như `create` không yêu cầu một model instance. Trong những tình huống này, bạn có thể truyền một tên class cho middleware. Tên class sẽ được sử dụng để xác định policy nào sẽ sử dụng khi ủy quyền hành động:

```php
Route::post('/post', function () {
    // The current user may create posts...
})->middleware('can:create,App\Models\Post');
```

Việc chỉ định tên class đầy đủ trong một định nghĩa middleware dạng chuỗi có thể trở nên cồng kềnh. Vì lý do đó, bạn có thể chọn đính kèm middleware `can` vào route của bạn bằng cách sử dụng phương thức `can`:

```php
use App\Models\Post;

Route::post('/post', function () {
    // The current user may create posts...
})->can('create', Post::class);
```

<a name="via-blade-templates"></a>
### Thông qua Blade Templates

Khi viết các Blade templates, bạn có thể muốn hiển thị một phần của trang chỉ khi người dùng được ủy quyền để thực hiện một hành động nhất định. Ví dụ, bạn có thể muốn hiển thị một form cập nhật cho một bài viết blog chỉ khi người dùng thực sự có thể cập nhật bài viết. Trong tình huống này, bạn có thể sử dụng các directives `@can` và `@cannot`:

```blade
@can('update', $post)
    <!-- The current user can update the post... -->
@elsecan('create', App\Models\Post::class)
    <!-- The current user can create new posts... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- The current user cannot update the post... -->
@elsecannot('create', App\Models\Post::class)
    <!-- The current user cannot create new posts... -->
@endcannot
```

Các directives này là các phím tắt thuận tiện để viết các câu lệnh `@if` và `@unless`. Các câu lệnh `@can` và `@cannot` ở trên tương đương với các câu lệnh sau:

```blade
@if (Auth::user()->can('update', $post))
    <!-- The current user can update the post... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- The current user cannot update the post... -->
@endunless
```

Bạn cũng có thể xác định xem người dùng có được ủy quyền để thực hiện bất kỳ hành động nào từ một mảng các hành động đã cho hay không. Để thực hiện điều này, sử dụng directive `@canany`:

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- The current user can update, view, or delete the post... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- The current user can create a post... -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### Actions That Don't Require Models

Giống như hầu hết các phương thức authorization khác, bạn có thể truyền một tên class cho các directives `@can` và `@cannot` nếu hành động không yêu cầu một model instance:

```blade
@can('create', App\Models\Post::class)
    <!-- The current user can create posts... -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- The current user can't create posts... -->
@endcannot
```

<a name="supplying-additional-context"></a>
### Cung cấp Additional Context

Khi ủy quyền các hành động bằng cách sử dụng policies, bạn có thể truyền một mảng làm đối số thứ hai cho các hàm và helpers authorization khác nhau. Phần tử đầu tiên trong mảng sẽ được sử dụng để xác định policy nào nên được gọi, trong khi các phần tử mảng còn lại được truyền làm tham số cho phương thức policy và có thể được sử dụng cho additional context khi đưa ra quyết định authorization. Ví dụ, hãy xem xét định nghĩa phương thức `PostPolicy` sau chứa một tham số bổ sung `$category`:

```php
/**
 * Determine if the given post can be updated by the user.
 */
public function update(User $user, Post $post, int $category): bool
{
    return $user->id === $post->user_id &&
           $user->canUpdateCategory($category);
}
```

Khi cố gắng xác định xem người dùng đã xác thực có thể cập nhật một bài viết nhất định hay không, chúng ta có thể gọi phương thức policy này như sau:

```php
/**
 * Update the given blog post.
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function update(Request $request, Post $post): RedirectResponse
{
    Gate::authorize('update', [$post, $request->category]);

    // The current user can update the blog post...

    return redirect('/posts');
}
```

<a name="authorization-and-inertia"></a>
## Authorization & Inertia

Mặc dù authorization phải luôn được xử lý trên máy chủ, nhưng thường có thể thuận tiện để cung cấp cho ứng dụng frontend của bạn dữ liệu authorization để hiển thị đúng UI của ứng dụng. Laravel không định nghĩa một quy ước bắt buộc để exposing thông tin authorization cho một frontend được hỗ trợ bởi Inertia.

Tuy nhiên, nếu bạn đang sử dụng một trong các [starter kits](/docs/{{version}}/starter-kits) dựa trên Inertia của Laravel, ứng dụng của bạn đã chứa một middleware `HandleInertiaRequests`. Trong phương thức `share` của middleware này, bạn có thể trả về dữ liệu được chia sẻ sẽ được cung cấp cho tất cả các trang Inertia trong ứng dụng của bạn. Dữ liệu được chia sẻ này có thể phục vụ như một vị trí thuận tiện để định nghĩa thông tin authorization cho người dùng:

```php
<?php

namespace App\Http\Middleware;

use App\Models\Post;
use Illuminate\Http\Request;
use Inertia\Middleware;

class HandleInertiaRequests extends Middleware
{
    // ...

    /**
     * Define the props that are shared by default.
     *
     * @return array<string, mixed>
     */
    public function share(Request $request)
    {
        return [
            ...parent::share($request),
            'auth' => [
                'user' => $request->user(),
                'permissions' => [
                    'post' => [
                        'create' => $request->user()->can('create', Post::class),
                    ],
                ],
            ],
        ];
    }
}
```

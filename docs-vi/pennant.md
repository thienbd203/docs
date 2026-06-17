# Laravel Pennant

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
- [Định nghĩa Features](#defining-features)
  - [Features dựa trên Class](#class-based-features)
- [Kiểm tra Features](#checking-features)
  - [Thực thi có điều kiện](#conditional-execution)
  - [Trait `HasFeatures`](#the-has-features-trait)
  - [Blade Directive](#blade-directive)
  - [Middleware](#middleware)
  - [Chặn kiểm tra Feature](#intercepting-feature-checks)
  - [Cache trong bộ nhớ](#in-memory-cache)
- [Scope](#scope)
  - [Chỉ định Scope](#specifying-the-scope)
  - [Scope mặc định](#default-scope)
  - [Scope nullable](#nullable-scope)
  - [Xác định Scope](#identifying-scope)
  - [Serialize Scope](#serializing-scope)
- [Giá trị Feature phong phú](#rich-feature-values)
- [Lấy nhiều Features](#retrieving-multiple-features)
- [Eager Loading](#eager-loading)
- [Cập nhật giá trị](#updating-values)
  - [Cập nhật hàng loạt](#bulk-updates)
  - [Xóa Features](#purging-features)
- [Testing](#testing)
- [Thêm Pennant Drivers tùy chỉnh](#adding-custom-pennant-drivers)
  - [Triển khai Driver](#implementing-the-driver)
  - [Đăng ký Driver](#registering-the-driver)
  - [Định nghĩa Features bên ngoài](#defining-features-externally)
- [Events](#events)

<a name="introduction"></a>

## Giới thiệu

[Laravel Pennant](https://github.com/laravel/pennant) là một gói feature flag đơn giản và nhẹ nhàng - không có những thứ không cần thiết. Feature flags cho phép bạn triển khai dần dần các tính năng ứng dụng mới một cách tự tin, A/B test các thiết kế giao diện mới, bổ sung cho chiến lược phát triển trunk-based, và nhiều hơn nữa.

<a name="installation"></a>

## Cài đặt

Đầu tiên, cài đặt Pennant vào dự án của bạn bằng trình quản lý gói Composer:

```shell
composer require laravel/pennant
```

Tiếp theo, bạn nên xuất bản file cấu hình và migration của Pennant bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
```

Cuối cùng, bạn nên chạy migration cơ sở dữ liệu của ứng dụng. Điều này sẽ tạo bảng `features` mà Pennant sử dụng để hỗ trợ driver `database` của nó:

```shell
php artisan migrate
```

<a name="configuration"></a>

## Cấu hình

Sau khi xuất bản tài sản của Pennant, file cấu hình của nó sẽ nằm tại `config/pennant.php`. File cấu hình này cho phép bạn chỉ định cơ chế lưu trữ mặc định mà Pennant sẽ sử dụng để lưu trữ các giá trị feature flag đã giải quyết.

Pennant bao gồm hỗ trợ lưu trữ các giá trị feature flag đã giải quyết trong một mảng trong bộ nhớ thông qua driver `array`. Hoặc, Pennant có thể lưu trữ các giá trị feature flag đã giải quyết một cách liên tục trong cơ sở dữ liệu quan hệ thông qua driver `database`, đây là cơ chế lưu trữ mặc định được sử dụng bởi Pennant.

<a name="defining-features"></a>

## Định nghĩa Features

Để định nghĩa một feature, bạn có thể sử dụng phương thức `define` được cung cấp bởi facade `Feature`. Bạn sẽ cần cung cấp tên cho feature, cũng như một closure sẽ được gọi để giải quyết giá trị ban đầu của feature.

Thông thường, các feature được định nghĩa trong một service provider bằng facade `Feature`. Closure sẽ nhận "scope" cho việc kiểm tra feature. Phổ biến nhất, scope là người dùng hiện đang được xác thực. Trong ví dụ này, chúng ta sẽ định nghĩa một feature để triển khai dần dần một API mới cho người dùng của ứng dụng:

```php
<?php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Lottery;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::define('new-api', fn (User $user) => match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        });
    }
}
```

Như bạn có thể thấy, chúng ta có các quy tắc sau cho feature của mình:

- Tất cả thành viên nội bộ nên sử dụng API mới.
- Bất kỳ khách hàng có lưu lượng truy cập cao nào không nên sử dụng API mới.
- Nếu không, feature sẽ được gán ngẫu nhiên cho người dùng với xác suất 1 trên 100 là hoạt động.

Lần đầu tiên feature `new-api` được kiểm tra cho một người dùng cụ thể, kết quả của closure sẽ được lưu trữ bởi driver lưu trữ. Lần tiếp theo feature được kiểm tra với cùng người dùng đó, giá trị sẽ được truy xuất từ bộ lưu trữ và closure sẽ không được gọi.

Để thuận tiện, nếu định nghĩa feature chỉ trả về một lottery, bạn có thể bỏ qua closure hoàn toàn:

    Feature::define('site-redesign', Lottery::odds(1, 1000));

<a name="class-based-features"></a>

### Features dựa trên Class

Pennant cũng cho phép bạn định nghĩa các feature dựa trên class. Khác với định nghĩa feature dựa trên closure, không cần đăng ký feature dựa trên class trong một service provider. Để tạo một feature dựa trên class, bạn có thể gọi lệnh Artisan `pennant:feature`. Theo mặc định, class feature sẽ được đặt trong thư mục `app/Features` của ứng dụng:

```shell
php artisan pennant:feature NewApi
```

Khi viết một class feature, bạn chỉ cần định nghĩa phương thức `resolve`, sẽ được gọi để giải quyết giá trị ban đầu của feature cho một scope cụ thể. Một lần nữa, scope thường sẽ là người dùng hiện đang được xác thực:

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Resolve the feature's initial value.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

Nếu bạn muốn giải quyết thủ công một instance của feature dựa trên class, bạn có thể gọi phương thức `instance` trên facade `Feature`:

```php
use Illuminate\Support\Facades\Feature;

$instance = Feature::instance(NewApi::class);
```

> [!NOTE]
> Các class feature được giải quyết thông qua [container](/docs/{{version}}/container), vì vậy bạn có thể inject dependencies vào constructor của class feature khi cần.

#### Tùy chỉnh tên Feature được lưu trữ

Theo mặc định, Pennant sẽ lưu trữ tên class đầy đủ của class feature. Nếu bạn muốn tách rời tên feature được lưu trữ khỏi cấu trúc nội bộ của ứng dụng, bạn có thể thêm attribute `Name` trên class feature. Giá trị của attribute này sẽ được lưu trữ thay cho tên class:

```php
<?php

namespace App\Features;

use Laravel\Pennant\Attributes\Name;

#[Name('new-api')]
class NewApi
{
    // ...
}
```

<a name="checking-features"></a>

## Kiểm tra Features

Để xác định xem một feature có hoạt động hay không, bạn có thể sử dụng phương thức `active` trên facade `Feature`. Theo mặc định, các feature được kiểm tra dựa trên người dùng hiện đang được xác thực:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active('new-api')
            ? $this->resolveNewApiResponse($request)
            : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

Mặc dù các feature được kiểm tra dựa trên người dùng hiện đang được xác thực theo mặc định, bạn có thể dễ dàng kiểm tra feature dựa trên người dùng khác hoặc [scope](#scope). Để thực hiện điều này, hãy sử dụng phương thức `for` được cung cấp bởi facade `Feature`:

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

Pennant cũng cung cấp một số phương thức tiện ích bổ sung có thể hữu ích khi xác định xem một feature có hoạt động hay không:

```php
// Determine if all of the given features are active...
Feature::allAreActive(['new-api', 'site-redesign']);

// Determine if any of the given features are active...
Feature::someAreActive(['new-api', 'site-redesign']);

// Determine if a feature is inactive...
Feature::inactive('new-api');

// Determine if all of the given features are inactive...
Feature::allAreInactive(['new-api', 'site-redesign']);

// Determine if any of the given features are inactive...
Feature::someAreInactive(['new-api', 'site-redesign']);
```

> [!NOTE]
> Khi sử dụng Pennant ngoài ngữ cảnh HTTP, chẳng hạn như trong lệnh Artisan hoặc job được xếp hàng, bạn thường nên [chỉ định rõ scope của feature](#specifying-the-scope). Ngoài ra, bạn có thể định nghĩa một [scope mặc định](#default-scope) bao gồm cả ngữ cảnh HTTP đã xác thực và ngữ cảnh chưa xác thực.

<a name="checking-class-based-features"></a>

#### Kiểm tra Features dựa trên Class

Đối với các feature dựa trên class, bạn nên cung cấp tên class khi kiểm tra feature:

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active(NewApi::class)
            ? $this->resolveNewApiResponse($request)
            : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

<a name="conditional-execution"></a>

### Thực thi có điều kiện

Phương thức `when` có thể được sử dụng để thực hiện một closure cụ thể một cách trôi chảy nếu feature hoạt động. Ngoài ra, một closure thứ hai có thể được cung cấp và sẽ được thực thi nếu feature không hoạt động:

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::when(NewApi::class,
            fn () => $this->resolveNewApiResponse($request),
            fn () => $this->resolveLegacyApiResponse($request),
        );
    }

    // ...
}
```

Phương thức `unless` đóng vai trò là nghịch đảo của phương thức `when`, thực thi closure đầu tiên nếu feature không hoạt động:

```php
return Feature::unless(NewApi::class,
    fn () => $this->resolveLegacyApiResponse($request),
    fn () => $this->resolveNewApiResponse($request),
);
```

<a name="the-has-features-trait"></a>

### Trait `HasFeatures`

Trait `HasFeatures` của Pennant có thể được thêm vào model `User` của ứng dụng (hoặc bất kỳ model nào khác có features) để cung cấp một cách trôi chảy, thuận tiện để kiểm tra features trực tiếp từ model:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Pennant\Concerns\HasFeatures;

class User extends Authenticatable
{
    use HasFeatures;

    // ...
}
```

Sau khi trait đã được thêm vào model của bạn, bạn có thể dễ dàng kiểm tra features bằng cách gọi phương thức `features`:

```php
if ($user->features()->active('new-api')) {
    // ...
}
```

Tất nhiên, phương thức `features` cung cấp quyền truy cập vào nhiều phương thức tiện ích khác để tương tác với features:

```php
// Values...
$value = $user->features()->value('purchase-button')
$values = $user->features()->values(['new-api', 'purchase-button']);

// State...
$user->features()->active('new-api');
$user->features()->allAreActive(['new-api', 'server-api']);
$user->features()->someAreActive(['new-api', 'server-api']);

$user->features()->inactive('new-api');
$user->features()->allAreInactive(['new-api', 'server-api']);
$user->features()->someAreInactive(['new-api', 'server-api']);

// Conditional execution...
$user->features()->when('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);

$user->features()->unless('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);
```

<a name="blade-directive"></a>

### Blade Directive

Để làm cho việc kiểm tra features trong Blade trở nên liền mạch, Pennant cung cấp các directive `@feature` và `@featureany`:

```blade
@feature('site-redesign')
    <!-- 'site-redesign' is active -->
@else
    <!-- 'site-redesign' is inactive -->
@endfeature

@featureany(['site-redesign', 'beta'])
    <!-- 'site-redesign' or `beta` is active -->
@endfeatureany
```

<a name="middleware"></a>

### Middleware

Pennant cũng bao gồm một [middleware](/docs/{{version}}/middleware) có thể được sử dụng để xác minh người dùng hiện đang được xác thực có quyền truy cập vào một feature trước khi route được gọi. Bạn có thể gán middleware cho một route và chỉ định các feature cần thiết để truy cập route. Nếu bất kỳ feature nào được chỉ định không hoạt động cho người dùng hiện đang được xác thực, phản hồi HTTP `400 Bad Request` sẽ được trả về bởi route. Nhiều feature có thể được truyền cho phương thức tĩnh `using`.

```php
use Illuminate\Support\Facades\Route;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

Route::get('/api/servers', function () {
    // ...
})->middleware(EnsureFeaturesAreActive::using('new-api', 'servers-api'));
```

<a name="customizing-the-response"></a>

#### Tùy chỉnh phản hồi

Nếu bạn muốn tùy chỉnh phản hồi được trả về bởi middleware khi một trong các feature được liệt kê không hoạt động, bạn có thể sử dụng phương thức `whenInactive` được cung cấp bởi middleware `EnsureFeaturesAreActive`. Thông thường, phương thức này nên được gọi trong phương thức `boot` của một trong các service provider của ứng dụng:

```php
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    EnsureFeaturesAreActive::whenInactive(
        function (Request $request, array $features) {
            return new Response(status: 403);
        }
    );

    // ...
}
```

<a name="intercepting-feature-checks"></a>

### Chặn kiểm tra Feature

Đôi khi có thể hữu ích để thực hiện một số kiểm tra trong bộ nhớ trước khi truy xuất giá trị được lưu trữ của một feature cụ thể. Hãy tưởng tượng bạn đang phát triển một API mới sau một feature flag và muốn khả năng vô hiệu hóa API mới mà không mất bất kỳ giá trị feature nào đã giải quyết trong bộ lưu trữ. Nếu bạn nhận thấy một lỗi trong API mới, bạn có thể dễ dàng vô hiệu hóa nó cho mọi người ngoại trừ thành viên nội bộ, sửa lỗi, sau đó bật lại API mới cho những người dùng trước đó đã có quyền truy cập vào feature.

Bạn có thể đạt được điều này với phương thức `before` của [feature dựa trên class](#class-based-features). Khi có mặt, phương thức `before` luôn chạy trong bộ nhớ trước khi truy xuất giá trị từ bộ lưu trữ. Nếu một giá trị khác `null` được trả về từ phương thức, nó sẽ được sử dụng thay cho giá trị được lưu trữ của feature trong suốt thời gian của request:

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Run an always-in-memory check before the stored value is retrieved.
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }
    }

    /**
     * Resolve the feature's initial value.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

Bạn cũng có thể sử dụng feature này để lên lịch triển khai toàn cầu của một feature trước đó nằm sau một feature flag:

```php
<?php

namespace App\Features;

use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Config;

class NewApi
{
    /**
     * Run an always-in-memory check before the stored value is retrieved.
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }

        if (Carbon::parse(Config::get('features.new-api.rollout-date'))->isPast()) {
            return true;
        }
    }

    // ...
}
```

<a name="in-memory-cache"></a>

### Cache trong bộ nhớ

Khi kiểm tra một feature, Pennant sẽ tạo một cache trong bộ nhớ của kết quả. Nếu bạn đang sử dụng driver `database`, điều này có nghĩa là kiểm tra lại cùng một feature flag trong một request duy nhất sẽ không kích hoạt các truy vấn cơ sở dữ liệu bổ sung. Điều này cũng đảm bảo rằng feature có kết quả nhất quán trong suốt thời gian của request.

Nếu bạn cần xóa thủ công cache trong bộ nhớ, bạn có thể sử dụng phương thức `flushCache` được cung cấp bởi facade `Feature`:

```php
Feature::flushCache();
```

<a name="scope"></a>

## Scope

<a name="specifying-the-scope"></a>

### Chỉ định Scope

Như đã thảo luận, các feature thường được kiểm tra dựa trên người dùng hiện đang được xác thực. Tuy nhiên, điều này có thể không luôn phù hợp với nhu cầu của bạn. Do đó, có thể chỉ định scope mà bạn muốn kiểm tra một feature cụ thể thông qua phương thức `for` của facade `Feature`:

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

Tất nhiên, scope của feature không giới hạn ở "người dùng". Hãy tưởng tượng bạn đã xây dựng một trải nghiệm thanh toán mới mà bạn đang triển khai cho toàn bộ các nhóm thay vì từng người dùng. Có lẽ bạn muốn các nhóm cũ hơn có tốc độ triển khai chậm hơn so với các nhóm mới hơn. Closure giải quyết feature của bạn có thể trông giống như sau:

```php
use App\Models\Team;
use Illuminate\Support\Carbon;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('billing-v2', function (Team $team) {
    if ($team->created_at->isAfter(new Carbon('1st Jan, 2023'))) {
        return true;
    }

    if ($team->created_at->isAfter(new Carbon('1st Jan, 2019'))) {
        return Lottery::odds(1 / 100);
    }

    return Lottery::odds(1 / 1000);
});
```

Bạn sẽ nhận thấy rằng closure chúng ta đã định nghĩa không mong đợi một `User`, mà thay vào đó mong đợi một model `Team`. Để xác định xem feature này có hoạt động cho nhóm của người dùng hay không, bạn nên truyền nhóm cho phương thức `for` được cung cấp bởi facade `Feature`:

```php
if (Feature::for($user->team)->active('billing-v2')) {
    return redirect('/billing/v2');
}

// ...
```

<a name="default-scope"></a>

### Scope mặc định

Cũng có thể tùy chỉnh scope mặc định mà Pennant sử dụng để kiểm tra features. Ví dụ, có lẽ tất cả các feature của bạn được kiểm tra dựa trên nhóm của người dùng hiện đang được xác thực thay vì người dùng. Thay vì phải gọi `Feature::for($user->team)` mỗi lần bạn kiểm tra một feature, bạn có thể chỉ định nhóm làm scope mặc định. Thông thường, điều này nên được thực hiện trong một trong các service provider của ứng dụng:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::resolveScopeUsing(fn ($driver) => Auth::user()?->team);

        // ...
    }
}
```

Nếu không có scope nào được cung cấp rõ ràng thông qua phương thức `for`, việc kiểm tra feature bây giờ sẽ sử dụng nhóm của người dùng hiện đang được xác thực làm scope mặc định:

```php
Feature::active('billing-v2');

// Is now equivalent to...

Feature::for($user->team)->active('billing-v2');
```

<a name="nullable-scope"></a>

### Scope Nullable

Nếu scope bạn cung cấp khi kiểm tra một feature là `null` và định nghĩa của feature không hỗ trợ `null` thông qua một loại nullable hoặc bằng cách bao gồm `null` trong một loại union, Pennant sẽ tự động trả về `false` làm giá trị kết quả của feature.

Vì vậy, nếu scope bạn truyền cho một feature có thể là `null` và bạn muốn bộ giải quyết giá trị của feature được gọi, bạn nên tính đến điều đó trong định nghĩa feature của mình. Một scope `null` có thể xảy ra nếu bạn kiểm tra một feature trong lệnh Artisan, job được xếp hàng, hoặc route chưa xác thực. Vì thường không có người dùng được xác thực trong các ngữ cảnh này, scope mặc định sẽ là `null`.

Nếu bạn không luôn [chỉ định rõ scope feature của mình](#specifying-the-scope) thì bạn nên đảm bảo loại của scope là "nullable" và xử lý giá trị scope `null` trong logic định nghĩa feature của mình:

```php
use App\Models\User;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('new-api', fn (User $user) => match (true) {// [tl! remove]
Feature::define('new-api', fn (User|null $user) => match (true) {// [tl! add]
    $user === null => true,// [tl! add]
    $user->isInternalTeamMember() => true,
    $user->isHighTrafficCustomer() => false,
    default => Lottery::odds(1 / 100),
});
```

<a name="identifying-scope"></a>

### Xác định Scope

Các driver lưu trữ `array` và `database` tích hợp sẵn của Pennant biết cách lưu trữ đúng các định danh scope cho tất cả các loại dữ liệu PHP cũng như các model Eloquent. Tuy nhiên, nếu ứng dụng của bạn sử dụng một driver Pennant bên thứ ba, driver đó có thể không biết cách lưu trữ đúng một định danh cho một model Eloquent hoặc các loại tùy chỉnh khác trong ứng dụng của bạn.

Vì lý do này, Pennant cho phép bạn định dạng các giá trị scope để lưu trữ bằng cách triển khai contract `FeatureScopeable` trên các đối tượng trong ứng dụng của bạn được sử dụng làm scope Pennant.

Ví dụ, hãy tưởng tượng bạn đang sử dụng hai driver feature khác nhau trong một ứng dụng duy nhất: driver `database` tích hợp sẵn và driver "Flag Rocket" bên thứ ba. Driver "Flag Rocket" không biết cách lưu trữ đúng một model Eloquent. Thay vào đó, nó yêu cầu một instance `FlagRocketUser`. Bằng cách triển khai `toFeatureIdentifier` được định nghĩa bởi contract `FeatureScopeable`, chúng ta có thể tùy chỉnh giá trị scope có thể lưu trữ được cung cấp cho mỗi driver được sử dụng bởi ứng dụng của chúng ta:

```php
<?php

namespace App\Models;

use FlagRocket\FlagRocketUser;
use Illuminate\Database\Eloquent\Model;
use Laravel\Pennant\Contracts\FeatureScopeable;

class User extends Model implements FeatureScopeable
{
    /**
     * Cast the object to a feature scope identifier for the given driver.
     */
    public function toFeatureIdentifier(string $driver): mixed
    {
        return match($driver) {
            'database' => $this,
            'flag-rocket' => FlagRocketUser::fromId($this->flag_rocket_id),
        };
    }
}
```

<a name="serializing-scope"></a>

### Serialize Scope

Theo mặc định, Pennant sẽ sử dụng tên class đầy đủ khi lưu trữ một feature liên kết với một model Eloquent. Nếu bạn đã sử dụng [Eloquent morph map](/docs/{{version}}/eloquent-relationships#custom-polymorphic-types), bạn có thể chọn để Pennant cũng sử dụng morph map để tách rời feature được lưu trữ khỏi cấu trúc ứng dụng.

Để đạt được điều này, sau khi định nghĩa Eloquent morph map của bạn trong một service provider, bạn có thể gọi phương thức `useMorphMap` của facade `Feature`:

```php
use Illuminate\Database\Eloquent\Relations\Relation;
use Laravel\Pennant\Feature;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);

Feature::useMorphMap();
```

<a name="rich-feature-values"></a>

## Giá trị Feature phong phú

Cho đến nay, chúng ta chủ yếu hiển thị các feature ở trạng thái nhị phân, nghĩa là chúng要么 "hoạt động"要么 "không hoạt động", nhưng Pennant cũng cho phép bạn lưu trữ các giá trị phong phú.

Ví dụ, hãy tưởng tượng bạn đang kiểm tra ba màu mới cho nút "Mua ngay" của ứng dụng. Thay vì trả về `true` hoặc `false` từ định nghĩa feature, bạn có thể trả về một chuỗi:

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn (User $user) => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

Bạn có thể truy xuất giá trị của feature `purchase-button` bằng phương thức `value`:

```php
$color = Feature::value('purchase-button');
```

Blade directive tích hợp sẵn của Pennant cũng giúp dễ dàng hiển thị nội dung có điều kiện dựa trên giá trị hiện tại của feature:

```blade
@feature('purchase-button', 'blue-sapphire')
    <!-- 'blue-sapphire' is active -->
@elsefeature('purchase-button', 'seafoam-green')
    <!-- 'seafoam-green' is active -->
@elsefeature('purchase-button', 'tart-orange')
    <!-- 'tart-orange' is active -->
@endfeature
```

> [!NOTE]
> Khi sử dụng các giá trị phong phú, điều quan trọng cần biết là một feature được coi là "hoạt động" khi nó có bất kỳ giá trị nào khác `false`.

Khi gọi phương thức [có điều kiện `when`](#conditional-execution), giá trị phong phú của feature sẽ được cung cấp cho closure đầu tiên:

```php
Feature::when('purchase-button',
    fn ($color) => /* ... */,
    fn () => /* ... */,
);
```

Tương tự, khi gọi phương thức có điều kiện `unless`, giá trị phong phú của feature sẽ được cung cấp cho closure thứ hai tùy chọn:

```php
Feature::unless('purchase-button',
    fn () => /* ... */,
    fn ($color) => /* ... */,
);
```

<a name="retrieving-multiple-features"></a>

## Lấy nhiều Features

Phương thức `values` cho phép truy xuất nhiều features cho một scope cụ thể:

```php
Feature::values(['billing-v2', 'purchase-button']);

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
// ]
```

Hoặc, bạn có thể sử dụng phương thức `all` để truy xuất các giá trị của tất cả các feature đã định nghĩa cho một scope cụ thể:

```php
Feature::all();

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

Tuy nhiên, các feature dựa trên class được đăng ký động và không được Pennant biết cho đến khi chúng được kiểm tra rõ ràng. Điều này có nghĩa là các feature dựa trên class của ứng dụng của bạn có thể không xuất hiện trong kết quả được trả về bởi phương thức `all` nếu chúng chưa được kiểm tra trong request hiện tại.

Nếu bạn muốn đảm bảo rằng các class feature luôn được bao gồm khi sử dụng phương thức `all`, bạn có thể sử dụng khả năng khám phá feature của Pennant. Để bắt đầu, hãy gọi phương thức `discover` trong một trong các service provider của ứng dụng:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::discover();

        // ...
    }
}
```

Phương thức `discover` sẽ đăng ký tất cả các class feature trong thư mục `app/Features` của ứng dụng. Phương thức `all` bây giờ sẽ bao gồm các class này trong kết quả của nó, bất kể chúng đã được kiểm tra trong request hiện tại hay chưa:

```php
Feature::all();

// [
//     'App\Features\NewApi' => true,
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

<a name="eager-loading"></a>

## Eager Loading

Mặc dù Pennant giữ một cache trong bộ nhớ của tất cả các feature đã giải quyết cho một request duy nhất, vẫn có thể gặp vấn đề về hiệu suất. Để giảm bớt điều này, Pennant cung cấp khả năng eager load các giá trị feature.

Để minh họa điều này, hãy tưởng tượng rằng chúng ta đang kiểm tra xem một feature có hoạt động trong một vòng lặp hay không:

```php
use Laravel\Pennant\Feature;

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

Giả sử chúng ta đang sử dụng driver cơ sở dữ liệu, mã này sẽ thực hiện một truy vấn cơ sở dữ liệu cho mỗi người dùng trong vòng lặp - thực hiện hàng trăm truy vấn tiềm năng. Tuy nhiên, sử dụng phương thức `load` của Pennant, chúng ta có thể loại bỏ nút thắt hiệu suất tiềm năng này bằng cách eager load các giá trị feature cho một tập hợp người dùng hoặc scope:

```php
Feature::for($users)->load(['notifications-beta']);

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

Để load các giá trị feature chỉ khi chúng chưa được load, bạn có thể sử dụng phương thức `loadMissing`:

```php
Feature::for($users)->loadMissing([
    'new-api',
    'purchase-button',
    'notifications-beta',
]);
```

Bạn có thể load tất cả các feature đã định nghĩa bằng phương thức `loadAll`:

```php
Feature::for($users)->loadAll();
```

<a name="updating-values"></a>

## Cập nhật giá trị

Khi giá trị của một feature được giải quyết lần đầu tiên, driver cơ bản sẽ lưu trữ kết quả trong bộ lưu trữ. Điều này thường cần thiết để đảm bảo trải nghiệm nhất quán cho người dùng của bạn trên các request. Tuy nhiên, đôi khi, bạn có thể muốn cập nhật thủ công giá trị được lưu trữ của feature.

Để thực hiện điều này, bạn có thể sử dụng các phương thức `activate` và `deactivate` để bật/tắt một feature:

```php
use Laravel\Pennant\Feature;

// Activate the feature for the default scope...
Feature::activate('new-api');

// Deactivate the feature for the given scope...
Feature::for($user->team)->deactivate('billing-v2');
```

Cũng có thể đặt thủ công một giá trị phong phú cho một feature bằng cách cung cấp một đối số thứ hai cho phương thức `activate`:

```php
Feature::activate('purchase-button', 'seafoam-green');
```

Để hướng dẫn Pennant quên giá trị được lưu trữ cho một feature, bạn có thể sử dụng phương thức `forget`. Khi feature được kiểm tra lại, Pennant sẽ giải quyết giá trị của feature từ định nghĩa feature của nó:

```php
Feature::forget('purchase-button');
```

<a name="bulk-updates"></a>

### Cập nhật hàng loạt

Để cập nhật các giá trị feature được lưu trữ hàng loạt, bạn có thể sử dụng các phương thức `activateForEveryone` và `deactivateForEveryone`.

Ví dụ, hãy tưởng tượng bạn giờ đây tự tin vào sự ổn định của feature `new-api` và đã chọn màu `'purchase-button'` tốt nhất cho quy trình thanh toán của mình - bạn có thể cập nhật giá trị được lưu trữ cho tất cả người dùng tương ứng:

```php
use Laravel\Pennant\Feature;

Feature::activateForEveryone('new-api');

Feature::activateForEveryone('purchase-button', 'seafoam-green');
```

Ngoài ra, bạn có thể vô hiệu hóa feature cho tất cả người dùng:

```php
Feature::deactivateForEveryone('new-api');
```

> [!NOTE]
> Điều này sẽ chỉ cập nhật các giá trị feature đã giải quyết đã được lưu trữ bởi driver lưu trữ của Pennant. Bạn cũng sẽ cần cập nhật định nghĩa feature trong ứng dụng của mình.

<a name="purging-features"></a>

### Xóa Features

Đôi khi, có thể hữu ích để xóa toàn bộ một feature khỏi bộ lưu trữ. Điều này thường cần thiết nếu bạn đã xóa feature khỏi ứng dụng của mình hoặc bạn đã thực hiện các điều chỉnh đối với định nghĩa feature mà bạn muốn triển khai cho tất cả người dùng.

Bạn có thể xóa tất cả các giá trị được lưu trữ cho một feature bằng phương thức `purge`:

```php
// Purging a single feature...
Feature::purge('new-api');

// Purging multiple features...
Feature::purge(['new-api', 'purchase-button']);
```

Nếu bạn muốn xóa _tất cả_ các feature khỏi bộ lưu trữ, bạn có thể gọi phương thức `purge` mà không có bất kỳ đối số nào:

```php
Feature::purge();
```

Vì có thể hữu ích để xóa các feature như một phần của pipeline triển khai ứng dụng, Pennant bao gồm một lệnh Artisan `pennant:purge` sẽ xóa các feature được cung cấp khỏi bộ lưu trữ:

```shell
php artisan pennant:purge new-api

php artisan pennant:purge new-api purchase-button
```

Cũng có thể xóa tất cả các feature _ngoại trừ_ những feature trong danh sách feature cụ thể. Ví dụ, hãy tưởng tượng bạn muốn xóa tất cả các feature nhưng giữ các giá trị cho các feature "new-api" và "purchase-button" trong bộ lưu trữ. Để thực hiện điều này, bạn có thể truyền tên các feature đó cho tùy chọn `--except`:

```shell
php artisan pennant:purge --except=new-api --except=purchase-button
```

Để thuận tiện, lệnh `pennant:purge` cũng hỗ trợ cờ `--except-registered`. Cờ này chỉ ra rằng tất cả các feature ngoại trừ những feature được đăng ký rõ ràng trong một service provider nên được xóa:

```shell
php artisan pennant:purge --except-registered
```

<a name="testing"></a>

## Testing

Khi kiểm tra mã tương tác với feature flags, cách dễ nhất để kiểm soát giá trị trả về của feature flag trong các bài kiểm tra của bạn là đơn giản là định nghĩa lại feature. Ví dụ, hãy tưởng tượng bạn có feature sau được định nghĩa trong một trong các service provider của ứng dụng:

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn () => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

Để sửa đổi giá trị trả về của feature trong các bài kiểm tra của bạn, bạn có thể định nghĩa lại feature ở đầu bài kiểm tra. Bài kiểm tra sau sẽ luôn vượt qua, mặc dù việc triển khai `Arr::random()` vẫn còn trong service provider:

```php
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define('purchase-button', 'seafoam-green');

    expect(Feature::value('purchase-button'))->toBe('seafoam-green');
});
```

```php
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define('purchase-button', 'seafoam-green');

    $this->assertSame('seafoam-green', Feature::value('purchase-button'));
}
```

Cách tiếp cận tương tự có thể được sử dụng cho các feature dựa trên class:

```php
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define(NewApi::class, true);

    expect(Feature::value(NewApi::class))->toBeTrue();
});
```

```php
use App\Features\NewApi;
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define(NewApi::class, true);

    $this->assertTrue(Feature::value(NewApi::class));
}
```

Nếu feature của bạn trả về một instance `Lottery`, có một số [helper kiểm tra hữu ích có sẵn](/docs/{{version}}/helpers#testing-lotteries).

<a name="store-configuration"></a>

#### Cấu hình Store

Bạn có thể cấu hình store mà Pennant sẽ sử dụng trong quá trình kiểm tra bằng cách định nghĩa biến môi trường `PENNANT_STORE` trong file `phpunit.xml` của ứng dụng:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit colors="true">
    <!-- ... -->
    <php>
        <env name="PENNANT_STORE" value="array"/>
        <!-- ... -->
    </php>
</phpunit>
```

<a name="adding-custom-pennant-drivers"></a>

## Thêm Pennant Drivers tùy chỉnh

<a name="implementing-the-driver"></a>

#### Triển khai Driver

Nếu không có driver lưu trữ hiện có nào của Pennant phù hợp với nhu cầu của ứng dụng, bạn có thể viết driver lưu trữ của riêng mình. Driver tùy chỉnh của bạn nên triển khai interface `Laravel\Pennant\Contracts\Driver`:

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;

class RedisFeatureDriver implements Driver
{
    public function define(string $feature, callable $resolver): void {}
    public function defined(): array {}
    public function getAll(array $features): array {}
    public function get(string $feature, mixed $scope): mixed {}
    public function set(string $feature, mixed $scope, mixed $value): void {}
    public function setForAllScopes(string $feature, mixed $value): void {}
    public function delete(string $feature, mixed $scope): void {}
    public function purge(array|null $features): void {}
}
```

Bây giờ, chúng ta chỉ cần triển khai từng phương thức này bằng một kết nối Redis. Để xem ví dụ về cách triển khai từng phương thức này, hãy xem `Laravel\Pennant\Drivers\DatabaseDriver` trong [mã nguồn Pennant](https://github.com/laravel/pennant/blob/1.x/src/Drivers/DatabaseDriver.php)

> [!NOTE]
> Laravel không đi kèm với một thư mục để chứa các phần mở rộng của bạn. Bạn có thể đặt chúng ở bất cứ đâu bạn thích. Trong ví dụ này, chúng ta đã tạo một thư mục `Extensions` để chứa `RedisFeatureDriver`.

<a name="registering-the-driver"></a>

#### Đăng ký Driver

Sau khi driver của bạn đã được triển khai, bạn đã sẵn sàng để đăng ký nó với Laravel. Để thêm các driver bổ sung vào Pennant, bạn có thể sử dụng phương thức `extend` được cung cấp bởi facade `Feature`. Bạn nên gọi phương thức `extend` từ phương thức `boot` của một trong các [service provider](/docs/{{version}}/providers) của ứng dụng:

```php
<?php

namespace App\Providers;

use App\Extensions\RedisFeatureDriver;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::extend('redis', function (Application $app) {
            return new RedisFeatureDriver($app->make('redis'), $app->make('events'), []);
        });
    }
}
```

Sau khi driver đã được đăng ký, bạn có thể sử dụng driver `redis` trong file cấu hình `config/pennant.php` của ứng dụng:

```php
'stores' => [

    'redis' => [
        'driver' => 'redis',
        'connection' => null,
    ],

    // ...

],
```

<a name="defining-features-externally"></a>

### Định nghĩa Features bên ngoài

Nếu driver của bạn là một wrapper xung quanh một nền tảng feature flag bên thứ ba, bạn có thể sẽ định nghĩa các feature trên nền tảng thay vì sử dụng phương thức `Feature::define` của Pennant. Nếu đó là trường hợp, driver tùy chỉnh của bạn cũng nên triển khai interface `Laravel\Pennant\Contracts\DefinesFeaturesExternally`:

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;
use Laravel\Pennant\Contracts\DefinesFeaturesExternally;

class FeatureFlagServiceDriver implements Driver, DefinesFeaturesExternally
{
    /**
     * Get the features defined for the given scope.
     */
    public function definedFeaturesForScope(mixed $scope): array {}

    /* ... */
}
```

Phương thức `definedFeaturesForScope` nên trả về danh sách tên feature được định nghĩa cho scope được cung cấp.

<a name="events"></a>

## Events

Pennant gửi đi nhiều sự kiện có thể hữu ích khi theo dõi feature flags trong suốt ứng dụng của bạn.

### `Laravel\Pennant\Events\FeatureRetrieved`

Sự kiện này được gửi đi bất cứ khi nào một [feature được kiểm tra](#checking-features). Sự kiện này có thể hữu ích để tạo và theo dõi các chỉ số đối với việc sử dụng feature flag trong suốt ứng dụng của bạn.

### `Laravel\Pennant\Events\FeatureResolved`

Sự kiện này được gửi đi lần đầu tiên giá trị của một feature được giải quyết cho một scope cụ thể.

### `Laravel\Pennant\Events\UnknownFeatureResolved`

Sự kiện này được gửi đi lần đầu tiên một feature không xác định được giải quyết cho một scope cụ thể. Lắng nghe sự kiện này có thể hữu ích nếu bạn có ý định xóa một feature flag nhưng đã vô tình để lại các tham chiếu rải rác đến nó trong suốt ứng dụng:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnknownFeatureResolved;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::listen(function (UnknownFeatureResolved $event) {
            Log::error("Resolving unknown feature [{$event->feature}].");
        });
    }
}
```

### `Laravel\Pennant\Events\DynamicallyRegisteringFeatureClass`

Sự kiện này được gửi đi khi một [feature dựa trên class](#class-based-features) được kiểm tra động lần đầu tiên trong một request.

### `Laravel\Pennant\Events\UnexpectedNullScopeEncountered`

Sự kiện này được gửi đi khi một scope `null` được truyền cho một định nghĩa feature [không hỗ trợ null](#nullable-scope).

Tình huống này được xử lý một cách nhẹ nhàng và feature sẽ trả về `false`. Tuy nhiên, nếu bạn muốn chọn không tham gia hành vi mặc định nhẹ nhàng của feature này, bạn có thể đăng ký một listener cho sự kiện này trong phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnexpectedNullScopeEncountered;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(UnexpectedNullScopeEncountered::class, fn () => abort(500));
}
```

### `Laravel\Pennant\Events\FeatureUpdated`

Sự kiện này được gửi đi khi cập nhật một feature cho một scope, thường bằng cách gọi `activate` hoặc `deactivate`.

### `Laravel\Pennant\Events\FeatureUpdatedForAllScopes`

Sự kiện này được gửi đi khi cập nhật một feature cho tất cả các scope, thường bằng cách gọi `activateForEveryone` hoặc `deactivateForEveryone`.

### `Laravel\Pennant\Events\FeatureDeleted`

Sự kiện này được gửi đi khi xóa một feature cho một scope, thường bằng cách gọi `forget`.

### `Laravel\Pennant\Events\FeaturesPurged`

Sự kiện này được gửi đi khi xóa các feature cụ thể.

### `Laravel\Pennant\Events\AllFeaturesPurged`

Sự kiện này được gửi đi khi xóa tất cả các feature.

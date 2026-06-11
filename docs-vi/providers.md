# Service Providers

- [Giới thiệu](#introduction)
- [Viết Service Providers](#writing-service-providers)
    - [Register Method](#the-register-method)
    - [Boot Method](#the-boot-method)
- [Đăng ký Providers](#registering-providers)
- [Deferred Providers](#deferred-providers)

<a name="introduction"></a>
## Giới thiệu

Service providers là nơi trung tâm của tất cả Laravel application bootstrapping. Application của bạn, cũng như tất cả Laravel core services, đều được bootstrap thông qua service providers.

Nhưng, chúng ta có ý nghĩa gì là "bootstrapped"? Nói chung, chúng ta có ý nghĩa là **đăng ký** things, bao gồm đăng ký service container bindings, event listeners, middleware, và thậm chí cả routes. Service providers là nơi trung tâm để cấu hình application của bạn.

Laravel sử dụng hàng chục service providers nội bộ để bootstrap các core services của nó, chẳng hạn như mailer, queue, cache, và các services khác. Nhiều providers này là "deferred" providers, có nghĩa là chúng sẽ không được load trên mọi request, nhưng chỉ khi services mà chúng cung cấp thực sự cần thiết.

Tất cả user-defined service providers đều được đăng ký trong file `bootstrap/providers.php`. Trong documentation sau đây, bạn sẽ học cách viết service providers của riêng bạn và đăng ký chúng với Laravel application của bạn.

> [!NOTE]
> Nếu bạn muốn tìm hiểu thêm về cách Laravel xử lý requests và hoạt động nội bộ, hãy xem documentation của chúng tôi về Laravel [request lifecycle](/docs/{{version}}/lifecycle).

<a name="writing-service-providers"></a>
## Viết Service Providers

Tất cả service providers đều extend class `Illuminate\Support\ServiceProvider`. Hầu hết service providers chứa một `register` và một `boot` method. Trong `register` method, bạn nên **chỉ bind things vào [service container](/docs/{{version}}/container)**. Bạn không bao giờ nên cố gắng đăng ký bất kỳ event listeners, routes, hoặc bất kỳ piece of functionality nào khác trong `register` method.

Artisan CLI có thể generate một provider mới thông qua command `make:provider`. Laravel sẽ tự động đăng ký provider mới của bạn trong file `bootstrap/providers.php` của application:

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### Register Method

Như đã đề cập trước đó, trong `register` method, bạn nên chỉ bind things vào [service container](/docs/{{version}}/container). Bạn không bao giờ nên cố gắng đăng ký bất kỳ event listeners, routes, hoặc bất kỳ piece of functionality nào khác trong `register` method. Nếu không, bạn có thể vô tình sử dụng một service được cung cấp bởi một service provider chưa được load.

Hãy xem một service provider cơ bản. Trong bất kỳ service provider methods nào của bạn, bạn luôn có access đến property `$app` cung cấp access đến service container:

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection(config('riak'));
        });
    }
}
```

Service provider này chỉ định nghĩa một `register` method, và sử dụng method đó để định nghĩa implementation của `App\Services\Riak\Connection` trong service container. Nếu bạn chưa quen với Laravel service container, hãy xem [documentation của nó](/docs/{{version}}/container).

<a name="the-bindings-and-singletons-properties"></a>
#### Properties `bindings` và `singletons`

Nếu service provider của bạn đăng ký nhiều simple bindings, bạn có thể muốn sử dụng properties `bindings` và `singletons` thay vì đăng ký thủ công từng container binding. Khi service provider được load bởi framework, nó sẽ tự động kiểm tra các properties này và đăng ký bindings của chúng:

```php
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

<a name="the-boot-method"></a>
### Boot Method

Vậy, nếu chúng ta cần đăng ký một [view composer](/docs/{{version}}/views#view-composers) trong service provider của mình thì sao? Điều này nên được thực hiện trong `boot` method. **Method này được gọi sau khi tất cả các service providers khác đã được đăng ký**, có nghĩa là bạn có access đến tất cả các services khác đã được đăng ký bởi framework:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::composer('view', function () {
            // ...
        });
    }
}
```

<a name="boot-method-dependency-injection"></a>
#### Boot Method Dependency Injection

Bạn có thể type-hint dependencies cho `boot` method của service provider. [Service container](/docs/{{version}}/container) sẽ tự động inject bất kỳ dependencies nào bạn cần:

```php
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * Bootstrap any application services.
 */
public function boot(ResponseFactory $response): void
{
    $response->macro('serialized', function (mixed $value) {
        // ...
    });
}
```

<a name="registering-providers"></a>
## Đăng ký Providers

Tất cả service providers đều được đăng ký trong file cấu hình `bootstrap/providers.php`. File này trả về một array chứa class names của service providers của application:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

Khi bạn invoke command `make:provider` Artisan, Laravel sẽ tự động thêm provider được generate vào file `bootstrap/providers.php`. Tuy nhiên, nếu bạn đã tạo thủ công provider class, bạn nên thêm thủ công provider class vào array:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ComposerServiceProvider::class, // [tl! add]
];
```

<a name="deferred-providers"></a>
## Deferred Providers

Nếu provider của bạn **chỉ** đăng ký bindings trong [service container](/docs/{{version}}/container), bạn có thể chọn defer việc đăng ký của nó cho đến khi một trong các registered bindings thực sự cần thiết. Việc defer loading của một provider như vậy sẽ cải thiện performance của application, vì nó không được load từ filesystem trên mọi request.

Laravel compile và lưu trữ một danh sách tất cả các services được cung cấp bởi deferred service providers, cùng với tên của service provider class của nó. Sau đó, chỉ khi bạn cố gắng resolve một trong các services này, Laravel mới load service provider.

Để defer loading của một provider, implement interface `\Illuminate\Contracts\Support\DeferrableProvider` và định nghĩa một `provides` method. `provides` method nên trả về service container bindings được đăng ký bởi provider:

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * Get the services provided by the provider.
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```

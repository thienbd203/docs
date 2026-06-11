# Service Container

- [Giới thiệu](#introduction)
    - [Zero Configuration Resolution](#zero-configuration-resolution)
    - [Khi nào nên sử dụng Container](#when-to-use-the-container)
- [Binding](#binding)
    - [Binding Basics](#binding-basics)
    - [Binding Interfaces sang Implementations](#binding-interfaces-to-implementations)
    - [Contextual Binding](#contextual-binding)
    - [Contextual Attributes](#contextual-attributes)
    - [Binding Primitives](#binding-primitives)
    - [Binding Typed Variadics](#binding-typed-variadics)
    - [Tagging](#tagging)
    - [Extending Bindings](#extending-bindings)
- [Resolving](#resolving)
    - [Make Method](#the-make-method)
    - [Automatic Injection](#automatic-injection)
- [Method Invocation và Injection](#method-invocation-and-injection)
- [Container Events](#container-events)
    - [Rebinding](#rebinding)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## Giới thiệu

Laravel service container là một công cụ mạnh mẽ để quản lý class dependencies và thực hiện dependency injection. Dependency injection là một fancy phrase về cơ bản có nghĩa là: class dependencies được "injected" vào class thông qua constructor hoặc, trong một số trường hợp, "setter" methods.

Hãy xem một ví dụ đơn giản:

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use Illuminate\View\View;

class PodcastController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * Show information about the given podcast.
     */
    public function show(string $id): View
    {
        return view('podcasts.show', [
            'podcast' => $this->apple->findPodcast($id)
        ]);
    }
}
```

Trong ví dụ này, `PodcastController` cần retrieve podcasts từ một data source như Apple Music. Vì vậy, chúng ta sẽ **inject** một service có thể retrieve podcasts. Vì service được inject, chúng ta có thể dễ dàng "mock", hoặc tạo một dummy implementation của `AppleMusic` service khi testing application của chúng ta.

Một deep understanding của Laravel service container là essential để xây dựng một powerful, large application, cũng như để contribute đến Laravel core.

<a name="zero-configuration-resolution"></a>
### Zero Configuration Resolution

Nếu một class không có dependencies hoặc chỉ phụ thuộc vào các concrete classes khác (không phải interfaces), container không cần được chỉ dẫn cách resolve class đó. Ví dụ, bạn có thể đặt code sau trong file `routes/web.php` của bạn:

```php
<?php

class Service
{
    // ...
}

Route::get('/', function (Service $service) {
    dd($service::class);
});
```

Trong ví dụ này, hitting route `/` của application sẽ tự động resolve class `Service` và inject nó vào route's handler của bạn. Đây là game changing. Nó có nghĩa là bạn có thể develop application của bạn và tận dụng dependency injection mà không cần lo lắng về bloated configuration files.

May mắn thay, nhiều classes bạn sẽ viết khi building Laravel application tự động nhận dependencies của họ thông qua container, bao gồm [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [middleware](/docs/{{version}}/middleware), và nhiều hơn nữa. Ngoài ra, bạn có thể type-hint dependencies trong `handle` method của [queued jobs](/docs/{{version}}/queues). Khi bạn nếm thử sức mạnh của automatic và zero configuration dependency injection, nó cảm thấy impossible để develop mà không có nó.

<a name="when-to-use-the-container"></a>
### Khi nào nên sử dụng Container

Nhờ zero configuration resolution, bạn sẽ thường type-hint dependencies trên routes, controllers, event listeners, và nơi khác mà không bao giờ manually interact với container. Ví dụ, bạn có thể type-hint object `Illuminate\Http\Request` trên route definition của bạn để bạn có thể dễ dàng access current request. Mặc dù chúng ta không bao giờ phải interact với container để viết code này, nó đang quản lý injection của các dependencies này behind the scenes:

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

Trong nhiều trường hợp, nhờ automatic dependency injection và [facades](/docs/{{version}}/facades), bạn có thể build Laravel applications mà không bao giờ manually bind hoặc resolve bất kỳ thứ gì từ container. **Vậy, khi nào bạn sẽ manually interact với container?** Hãy examine hai situations.

Đầu tiên, nếu bạn viết một class implement một interface và bạn muốn type-hint interface đó trên một route hoặc class constructor, bạn phải [tell container cách resolve interface đó](#binding-interfaces-to-implementations). Thứ hai, nếu bạn đang [viết một Laravel package](/docs/{{version}}/packages) mà bạn có kế hoạch share với các Laravel developers khác, bạn có thể cần bind package's services của bạn vào container.

<a name="binding"></a>
## Binding

<a name="binding-basics"></a>
### Binding Basics

<a name="simple-bindings"></a>
#### Simple Bindings

Hầu hết tất cả service container bindings của bạn sẽ được registered trong [service providers](/docs/{{version}}/providers), vì vậy hầu hết các ví dụ này sẽ demonstrate sử dụng container trong context đó.

Trong một service provider, bạn luôn có access đến container thông qua property `$this->app`. Chúng ta có thể register một binding sử dụng method `bind`, truyền class hoặc interface name mà chúng ta muốn register cùng với một closure trả về một instance của class:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Lưu ý rằng chúng ta nhận container itself như một argument cho resolver. Chúng ta sau đó có thể sử dụng container để resolve sub-dependencies của object mà chúng ta đang build.

Như đã đề cập, bạn sẽ thường interact với container trong service providers; tuy nhiên, nếu bạn muốn interact với container bên ngoài một service provider, bạn có thể làm điều đó thông qua `App` [facade](/docs/{{version}}/facades):

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\App;

App::bind(Transistor::class, function (Application $app) {
    // ...
});
```

Bạn có thể sử dụng method `bindIf` để register một container binding chỉ khi một binding chưa được register cho given type:

```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Để tiện lợi, bạn có thể omit việc cung cấp class hoặc interface name mà bạn muốn register như một separate argument và thay vì cho phép Laravel infer type từ return type của closure mà bạn cung cấp cho method `bind`:

```php
App::bind(function (Application $app): Transistor {
    return new Transistor($app->make(PodcastParser::class));
});
```

> [!NOTE]
> Không cần bind classes vào container nếu chúng không phụ thuộc vào bất kỳ interfaces nào. Container không cần được chỉ dẫn cách build các objects này, vì nó có thể tự động resolve các objects này sử dụng reflection.

<a name="binding-a-singleton"></a>
#### Binding A Singleton

Method `singleton` bind một class hoặc interface vào container chỉ nên được resolve một lần. Khi một singleton binding được resolve, cùng object instance sẽ được trả về trên subsequent calls vào container:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Bạn có thể sử dụng method `singletonIf` để register một singleton container binding chỉ khi một binding chưa được register cho given type:

```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="singleton-attribute"></a>
#### Singleton Attribute

Ngoài ra, bạn có thể mark một interface hoặc class với attribute `#[Singleton]` để chỉ định cho container rằng nó nên được resolve một lần:

```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Singleton;

#[Singleton]
class Transistor
{
    // ...
}
```

<a name="binding-scoped"></a>
#### Binding Scoped Singletons

Method `scoped` bind một class hoặc interface vào container chỉ nên được resolve một lần trong một given Laravel request / job lifecycle. Trong khi method này tương tự với method `singleton`, instances được register sử dụng method `scoped` sẽ được flushed bất cứ khi nào Laravel application bắt đầu một "lifecycle" mới, chẳng hạn như khi một [Laravel Octane](/docs/{{version}}/octane) worker processes một new request hoặc khi một Laravel [queue worker](/docs/{{version}}/queues) processes một new job:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

Bạn có thể sử dụng method `scopedIf` để register một scoped container binding chỉ khi một binding chưa được register cho given type:

```php
$this->app->scopedIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="scoped-attribute"></a>
#### Scoped Attribute

Ngoài ra, bạn có thể mark một interface hoặc class với attribute `#[Scoped]` để chỉ định cho container rằng nó nên được resolve một lần trong một given Laravel request / job lifecycle:

```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Scoped;

#[Scoped]
class Transistor
{
    // ...
}
```

<a name="binding-instances"></a>
#### Binding Instances

Bạn cũng có thể bind một existing object instance vào container sử dụng method `instance`. Given instance sẽ luôn được trả về trên subsequent calls vào container:

```php
use App\Services\Transistor;
use App\Services\PodcastParser;

$service = new Transistor(new PodcastParser);

$this->app->instance(Transistor::class, $service);
```

<a name="binding-interfaces-to-implementations"></a>
### Binding Interfaces sang Implementations

Một very powerful feature của service container là khả năng bind một interface vào một given implementation. Ví dụ, hãy giả định chúng ta có một interface `EventPusher` và một implementation `RedisEventPusher`. Khi chúng ta đã coded implementation `RedisEventPusher` của interface này, chúng ta có thể register nó với service container như sau:

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;

$this->app->bind(EventPusher::class, RedisEventPusher::class);
```

Statement này tells container rằng nó nên inject `RedisEventPusher` khi một class cần một implementation của `EventPusher`. Bây giờ chúng ta có thể type-hint interface `EventPusher` trong constructor của một class được resolve bởi container. Hãy nhớ, controllers, event listeners, middleware, và nhiều other types của classes trong Laravel applications luôn được resolve sử dụng container:

```php
use App\Contracts\EventPusher;

/**
 * Create a new class instance.
 */
public function __construct(
    protected EventPusher $pusher,
) {}
```

<a name="bind-attribute"></a>
#### Bind Attribute

Laravel cũng cung cấp một attribute `Bind` cho added convenience. Bạn có thể apply attribute này vào bất kỳ interface nào để tell Laravel implementation nào nên được tự động injected bất cứ khi nào interface đó được requested. Khi sử dụng attribute `Bind`, không cần perform bất kỳ additional service registration nào trong service providers của application.

Ngoài ra, nhiều attributes `Bind` có thể được đặt trên một interface để configure một implementation khác nên được injected cho một given set của environments:

```php
<?php

namespace App\Contracts;

use App\Services\FakeEventPusher;
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;

#[Bind(RedisEventPusher::class)]
#[Bind(FakeEventPusher::class, environments: ['local', 'testing'])]
interface EventPusher
{
    // ...
}
```

Hơn nữa, attributes [Singleton](#singleton-attribute) và [Scoped](#scoped-attribute) có thể được applied để chỉ định nếu container bindings nên được resolve một lần hoặc một lần per request / job lifecycle:

```php
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;
use Illuminate\Container\Attributes\Singleton;

#[Bind(RedisEventPusher::class)]
#[Singleton]
interface EventPusher
{
    // ...
}
```

<a name="contextual-binding"></a>
### Contextual Binding

Đôi khi bạn có thể có hai classes sử dụng cùng interface, nhưng bạn muốn inject different implementations vào mỗi class. Ví dụ, hai controllers có thể phụ thuộc vào different implementations của `Illuminate\Contracts\Filesystem\Filesystem` [contract](/docs/{{version}}/contracts). Laravel cung cấp một simple, fluent interface để define behavior này:

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

$this->app->when(PhotoController::class)
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('local');
    });

$this->app->when([VideoController::class, UploadController::class])
    ->needs(Filesystem::class)
    ->give(function () {
        return Storage::disk('s3');
    });
```

<a name="contextual-attributes"></a>
### Contextual Attributes

Vì contextual binding thường được sử dụng để inject implementations của drivers hoặc configuration values, Laravel cung cấp nhiều contextual binding attributes cho phép inject các types của values này mà không cần manually define contextual bindings trong service providers của bạn.

Ví dụ, attribute `Storage` có thể được sử dụng để inject một specific [storage disk](/docs/{{version}}/filesystem):

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Container\Attributes\Storage;
use Illuminate\Contracts\Filesystem\Filesystem;

class PhotoController extends Controller
{
    public function __construct(
        #[Storage('local')] protected Filesystem $filesystem
    ) {
        // ...
    }
}
```

Ngoài attribute `Storage`, Laravel cung cấp attributes `Auth`, `Cache`, `Config`, `Context`, `DB`, `Give`, `Log`, `RouteParameter`, và [Tag](#tagging):

```php
<?php

namespace App\Http\Controllers;

use App\Contracts\UserRepository;
use App\Models\Photo;
use App\Repositories\DatabaseRepository;
use Illuminate\Container\Attributes\Auth;
use Illuminate\Container\Attributes\Cache;
use Illuminate\Container\Attributes\Config;
use Illuminate\Container\Attributes\Context;
use Illuminate\Container\Attributes\DB;
use Illuminate\Container\Attributes\Give;
use Illuminate\Container\Attributes\Log;
use Illuminate\Container\Attributes\RouteParameter;
use Illuminate\Container\Attributes\Tag;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Database\Connection;
use Psr\Log\LoggerInterface;

class PhotoController extends Controller
{
    public function __construct(
        #[Auth('web')] protected Guard $auth,
        #[Cache('redis')] protected Repository $cache,
        #[Config('app.timezone')] protected string $timezone,
        #[Context('uuid')] protected string $uuid,
        #[Context('ulid', hidden: true)] protected string $ulid,
        #[DB('mysql')] protected Connection $connection,
        #[Give(DatabaseRepository::class)] protected UserRepository $users,
        #[Log('daily')] protected LoggerInterface $log,
        #[RouteParameter('photo')] protected Photo $photo,
        #[Tag('reports')] protected iterable $reports,
    ) {
        // ...
    }
}
```

Hơn nữa, Laravel cung cấp một attribute `CurrentUser` để inject currently authenticated user vào một given route hoặc class:

```php
use App\Models\User;
use Illuminate\Container\Attributes\CurrentUser;

Route::get('/user', function (#[CurrentUser] User $user) {
    return $user;
})->middleware('auth');
```

<a name="defining-custom-attributes"></a>
#### Defining Custom Attributes

Bạn có thể tạo contextual attributes của riêng bạn bằng cách implementing contract `Illuminate\Contracts\Container\ContextualAttribute`. Container sẽ gọi attribute's `resolve` method của bạn, nên resolve value nên được inject vào class sử dụng attribute. Trong ví dụ dưới đây, chúng ta sẽ re-implement built-in attribute `Config` của Laravel:

```php
<?php

namespace App\Attributes;

use Attribute;
use Illuminate\Contracts\Container\Container;
use Illuminate\Contracts\Container\ContextualAttribute;
use ReflectionParameter;

#[Attribute(Attribute::TARGET_PARAMETER)]
class Config implements ContextualAttribute
{
    /**
     * Create a new attribute instance.
     */
    public function __construct(public string $key, public mixed $default = null)
    {
    }

    /**
     * Resolve the configuration value.
     *
     * @param  self  $attribute
     * @param  \Illuminate\Contracts\Container\Container  $container
     * @param  \ReflectionParameter  $parameter
     * @return mixed
     */
    public static function resolve(self $attribute, Container $container, ReflectionParameter $parameter)
    {
        return $container->make('config')->get($attribute->key, $attribute->default);
    }
}
```

<a name="binding-primitives"></a>
### Binding Primitives

Đôi khi bạn có thể có một class nhận một số injected classes, nhưng cũng cần một injected primitive value như một integer. Bạn có thể dễ dàng sử dụng contextual binding để inject bất kỳ value nào mà class của bạn có thể cần:

```php
use App\Http\Controllers\UserController;

$this->app->when(UserController::class)
    ->needs('$variableName')
    ->give($value);
```

Đôi khi một class có thể phụ thuộc vào một array của [tagged](#tagging) instances. Sử dụng method `giveTagged`, bạn có thể dễ dàng inject tất cả container bindings với tag đó:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

Nếu bạn cần inject một value từ một trong configuration files của application, bạn có thể sử dụng method `giveConfig`:

```php
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');
```

<a name="binding-typed-variadics"></a>
### Binding Typed Variadics

Thỉnh thoảng, bạn có thể có một class nhận một array của typed objects sử dụng một variadic constructor argument:

```php
<?php

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
    /**
     * The filter instances.
     *
     * @var array
     */
    protected $filters;

    /**
     * Create a new class instance.
     */
    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}
```

Sử dụng contextual binding, bạn có thể resolve dependency này bằng cách cung cấp method `give` với một closure trả về một array của resolved `Filter` instances:

```php
$this->app->when(Firewall::class)
    ->needs(Filter::class)
    ->give(function (Application $app) {
          return [
              $app->make(NullFilter::class),
              $app->make(ProfanityFilter::class),
              $app->make(TooLongFilter::class),
          ];
    });
```

Để tiện lợi, bạn cũng có thể chỉ cung cấp một array của class names để được resolve bởi container bất cứ khi nào `Firewall` cần `Filter` instances:

```php
$this->app->when(Firewall::class)
    ->needs(Filter::class)
    ->give([
        NullFilter::class,
        ProfanityFilter::class,
        TooLongFilter::class,
    ]);
```

<a name="variadic-tag-dependencies"></a>
#### Variadic Tag Dependencies

Đôi khi một class có thể có một variadic dependency được type-hint như một given class (`Report ...$reports`). Sử dụng các methods `needs` và `giveTagged`, bạn có thể dễ dàng inject tất cả container bindings với [tag](#tagging) đó cho given dependency:

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

<a name="tagging"></a>
### Tagging

Thỉnh thoảng, bạn có thể cần resolve tất cả một "category" nhất định của binding. Ví dụ, có thể bạn đang building một report analyzer nhận một array của nhiều different `Report` interface implementations. Sau khi register các `Report` implementations, bạn có thể assign cho họ một tag sử dụng method `tag`:

```php
$this->app->bind(CpuReport::class, function () {
    // ...
});

$this->app->bind(MemoryReport::class, function () {
    // ...
});

$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');
```

Khi services đã được tagged, bạn có thể dễ dàng resolve tất cả chúng thông qua method `tagged` của container:

```php
$this->app->bind(ReportAnalyzer::class, function (Application $app) {
    return new ReportAnalyzer($app->tagged('reports'));
});
```

<a name="extending-bindings"></a>
### Extending Bindings

Method `extend` cho phép modification của resolved services. Ví dụ, khi một service được resolve, bạn có thể run additional code để decorate hoặc configure service. Method `extend` chấp nhận hai arguments, service class mà bạn đang extending và một closure nên trả về modified service. Closure nhận service đang được resolve và container instance:

```php
$this->app->extend(Service::class, function (Service $service, Application $app) {
    return new DecoratedService($service);
});
```

<a name="resolving"></a>
## Resolving

<a name="the-make-method"></a>
### Method `make`

Bạn có thể sử dụng method `make` để resolve một class instance từ container. Method `make` chấp nhận tên của class hoặc interface mà bạn muốn resolve:

```php
use App\Services\Transistor;

$transistor = $this->app->make(Transistor::class);
```

Nếu một số dependencies của class của bạn không thể được resolve thông qua container, bạn có thể inject chúng bằng cách truyền chúng như một associative array vào method `makeWith`. Ví dụ, chúng ta có thể manually pass constructor argument `$id` được yêu cầu bởi service `Transistor`:

```php
use App\Services\Transistor;

$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

Method `bound` có thể được sử dụng để determine nếu một class hoặc interface đã được explicitly bound trong container:

```php
if ($this->app->bound(Transistor::class)) {
    // ...
}
```

Nếu bạn ở bên ngoài một service provider trong một location của code của bạn không có access đến variable `$app`, bạn có thể sử dụng `App` [facade](/docs/{{version}}/facades) hoặc `app` [helper](/docs/{{version}}/helpers#method-app) để resolve một class instance từ container:

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;

$transistor = App::make(Transistor::class);

$transistor = app(Transistor::class);
```

Nếu bạn muốn có Laravel container instance itself được inject vào một class đang được resolve bởi container, bạn có thể type-hint class `Illuminate\Container\Container` trên constructor của class của bạn:

```php
use Illuminate\Container\Container;

/**
 * Create a new class instance.
 */
public function __construct(
    protected Container $container,
) {}
```

<a name="automatic-injection"></a>
### Automatic Injection

Ngoài ra, và quan trọng, bạn có thể type-hint dependency trong constructor của một class được resolve bởi container, bao gồm [controllers](/docs/{{version}}/controllers), [event listeners](/docs/{{version}}/events), [middleware](/docs/{{version}}/middleware), và nhiều hơn nữa. Ngoài ra, bạn có thể type-hint dependencies trong `handle` method của [queued jobs](/docs/{{version}}/queues). Trong practice, đây là cách hầu hết objects của bạn nên được resolve bởi container.

Ví dụ, bạn có thể type-hint một service được define bởi application của bạn trong constructor của một controller. Service sẽ tự động được resolve và inject vào class:

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;

class PodcastController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * Show information about the given podcast.
     */
    public function show(string $id): Podcast
    {
        return $this->apple->findPodcast($id);
    }
}
```

<a name="method-invocation-and-injection"></a>
## Method Invocation và Injection

Đôi khi bạn có thể muốn invoke một method trên một object instance trong khi cho phép container tự động inject dependencies của method đó. Ví dụ, với class sau:

```php
<?php

namespace App;

use App\Services\AppleMusic;

class PodcastStats
{
    /**
     * Generate a new podcast stats report.
     */
    public function generate(AppleMusic $apple): array
    {
        return [
            // ...
        ];
    }
}
```

Bạn có thể invoke method `generate` thông qua container như sau:

```php
use App\PodcastStats;
use Illuminate\Support\Facades\App;

$stats = App::call([new PodcastStats, 'generate']);
```

Method `call` chấp nhận bất kỳ PHP callable nào. Method `call` của container thậm chí có thể được sử dụng để invoke một closure trong khi tự động inject dependencies của nó:

```php
use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;

$result = App::call(function (AppleMusic $apple) {
    // ...
});
```

<a name="container-events"></a>
## Container Events

Service container fires một event mỗi khi nó resolve một object. Bạn có thể listen đến event này sử dụng method `resolving`:

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;

$this->app->resolving(Transistor::class, function (Transistor $transistor, Application $app) {
    // Called when container resolves objects of type "Transistor"...
});

$this->app->resolving(function (mixed $object, Application $app) {
    // Called when container resolves object of any type...
});
```

Như bạn có thể thấy, object đang được resolve sẽ được passed vào callback, cho phép bạn set bất kỳ additional properties nào trên object trước khi nó được đưa cho consumer của nó.

<a name="rebinding"></a>
### Rebinding

Method `rebinding` cho phép bạn listen khi một service được re-bound vào container, có nghĩa là nó được register lại hoặc overridden sau initial binding của nó. Điều này có thể hữu ích khi bạn cần update dependencies hoặc modify behavior mỗi khi một specific binding được updated:

```php
use App\Contracts\PodcastPublisher;
use App\Services\SpotifyPublisher;
use App\Services\TransistorPublisher;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(PodcastPublisher::class, SpotifyPublisher::class);

$this->app->rebinding(
    PodcastPublisher::class,
    function (Application $app, PodcastPublisher $newInstance) {
        //
    },
);

// New binding will trigger rebinding closure...
$this->app->bind(PodcastPublisher::class, TransistorPublisher::class);
```

<a name="psr-11"></a>
## PSR-11

Laravel service container implement interface [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md). Do đó, bạn có thể type-hint PSR-11 container interface để obtain một instance của Laravel container:

```php
use App\Services\Transistor;
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);

    // ...
});
```

Một exception được thrown nếu given identifier không thể được resolve. Exception sẽ là một instance của `Psr\Container\NotFoundExceptionInterface` nếu identifier chưa bao giờ được bound. Nếu identifier đã được bound nhưng không thể được resolve, một instance của `Psr\Container\ContainerExceptionInterface` sẽ được thrown.

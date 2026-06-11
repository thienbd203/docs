# Eloquent: Bắt Đầu

- [Giới thiệu](#introduction)
- [Tạo Model Classes](#generating-model-classes)
- [Quy ước Eloquent Model](#eloquent-model-conventions)
    - [Tên Table](#table-names)
    - [Primary Keys](#primary-keys)
    - [UUID và ULID Keys](#uuid-and-ulid-keys)
    - [Timestamps](#timestamps)
    - [Database Connections](#database-connections)
    - [Giá trị mặc định của Attribute](#default-attribute-values)
    - [Cấu hình Eloquent Strictness](#configuring-eloquent-strictness)
- [Lấy Models](#retrieving-models)
    - [Collections](#collections)
    - [Chunking Results](#chunking-results)
    - [Chunk Sử Dụng Lazy Collections](#chunking-using-lazy-collections)
    - [Cursors](#cursors)
    - [Subquery Nâng Cao](#advanced-subqueries)
- [Lấy Single Models / Aggregates](#retrieving-single-models)
    - [Lấy hoặc Tạo Models](#retrieving-or-creating-models)
    - [Lấy Aggregates](#retrieving-aggregates)
- [Insert và Update Models](#inserting-and-updating-models)
    - [Inserts](#inserts)
    - [Updates](#updates)
    - [Mass Assignment](#mass-assignment)
    - [Upserts](#upserts)
- [Xóa Models](#deleting-models)
    - [Soft Deleting](#soft-deleting)
    - [Query Soft Deleted Models](#querying-soft-deleted-models)
- [Pruning Models](#pruning-models)
- [Replicating Models](#replicating-models)
- [Query Scopes](#query-scopes)
    - [Global Scopes](#global-scopes)
    - [Local Scopes](#local-scopes)
    - [Pending Attributes](#pending-attributes)
- [So Sánh Models](#comparing-models)
- [Events](#events)
    - [Sử Dụng Closures](#events-using-closures)
    - [Observers](#observers)
    - [Muting Events](#muting-events)

<a name="introduction"></a>
## Giới thiệu

Laravel includes Eloquent, một object-relational mapper (ORM) làm cho việc interact với database của bạn trở nên enjoyable. Khi sử dụng Eloquent, mỗi database table có một corresponding "Model" được sử dụng để interact với table đó. Ngoài việc retrieving records từ database table, Eloquent models cho phép bạn insert, update, và delete records từ table đó.

> [!NOTE]
> Trước khi bắt đầu, hãy chắc chắn configure một database connection trong application's `config/database.php` configuration file của bạn. Để biết thêm thông tin về configuring database của bạn, hãy check out [the database configuration documentation](/docs/{{version}}/database#configuration).

<a name="generating-model-classes"></a>
## Tạo Model Classes

Để bắt đầu, hãy tạo một Eloquent model. Models thường live trong `app\Models` directory và extend `Illuminate\Database\Eloquent\Model` class. Bạn có thể sử dụng `make:model` [Artisan command](/docs/{{version}}/artisan) để generate một new model:

```shell
php artisan make:model Flight
```

Nếu bạn muốn generate một [database migration](/docs/{{version}}/migrations) khi bạn generate model, bạn có thể sử dụng `--migration` hoặc `-m` option:

```shell
php artisan make:model Flight --migration
```

Bạn có thể generate various other types của classes khi generating một model, như factories, seeders, policies, controllers, và form requests. Ngoài ra, những options này có thể được combined để create multiple classes cùng một lúc:

```shell
# Generate a model and a FlightFactory class...
php artisan make:model Flight --factory
php artisan make:model Flight -f

# Generate a model and a FlightSeeder class...
php artisan make:model Flight --seed
php artisan make:model Flight -s

# Generate a model and a FlightController class...
php artisan make:model Flight --controller
php artisan make:model Flight -c

# Generate a model, FlightController resource class, and form request classes...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR

# Generate a model and a FlightPolicy class...
php artisan make:model Flight --policy

# Generate a model and a migration, factory, seeder, and controller...
php artisan make:model Flight -mfsc

# Shortcut to generate a model, migration, factory, seeder, policy, controller, and form requests...
php artisan make:model Flight --all
php artisan make:model Flight -a

# Generate a pivot model...
php artisan make:model Member --pivot
php artisan make:model Member -p
```

<a name="inspecting-models"></a>
#### Inspecting Models

Đôi khi có thể khó determine tất cả available attributes và relationships của một model chỉ bằng cách skimming code của nó. Thay vào đó, hãy thử `model:show` Artisan command, mà cung cấp một convenient overview của tất cả model's attributes và relations:

```shell
php artisan model:show Flight
```

<a name="eloquent-model-conventions"></a>
## Quy ước Eloquent Model

Models được generate bởi `make:model` command sẽ được placed trong `app/Models` directory. Hãy examine một basic model class và discuss một số key conventions của Eloquent:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // ...
}
```

<a name="table-names"></a>
### Tên Table

Sau khi glancing ở example trên, bạn có thể đã noticed rằng chúng ta không tell Eloquent database table nào corresponds với `Flight` model của chúng ta. Theo convention, "snake case", plural name của class sẽ được sử dụng như table name trừ khi một name khác được explicitly specified. Vì vậy, trong trường hợp này, Eloquent sẽ assume `Flight` model stores records trong `flights` table, trong khi một `AirTrafficController` model sẽ store records trong một `air_traffic_controllers` table.

Nếu model's corresponding database table của bạn không fit convention này, bạn có thể manually specify model's table name sử dụng `Table` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table('my_flights')]
class Flight extends Model
{
    // ...
}
```


<a name="primary-keys"></a>
### Primary Keys

Eloquent cũng sẽ assume rằng mỗi model's corresponding database table có một primary key column tên là `id`. Nếu cần thiết, bạn có thể specify một column khác serves như model's primary key của bạn sử dụng `key` argument trên `Table` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(key: 'flight_id')]
class Flight extends Model
{
    // ...
}
```

Ngoài ra, Eloquent assumes rằng primary key là một incrementing integer value, có nghĩa là Eloquent sẽ automatically cast primary key thành một integer. Nếu bạn muốn sử dụng một non-incrementing hoặc một non-numeric primary key, bạn nên specify `keyType` và `incrementing` arguments trên `Table` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
class Flight extends Model
{
    // ...
}
```

Nếu bạn chỉ cần disable auto-incrementing IDs, bạn có thể sử dụng `WithoutIncrementing` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\WithoutIncrementing;
use Illuminate\Database\Eloquent\Model;

#[WithoutIncrementing]
class Flight extends Model
{
    // ...
}
```

<a name="composite-primary-keys"></a>
#### "Composite" Primary Keys

Eloquent requires each model to have at least one uniquely identifying "ID" that can serve as its primary key. "Composite" primary keys are not supported by Eloquent models. However, you are free to add additional multi-column, unique indexes to your database tables in addition to the table's uniquely identifying primary key.

<a name="uuid-and-ulid-keys"></a>
### UUID và ULID Keys

Thay vì sử dụng auto-incrementing integers như Eloquent model's primary keys của bạn, bạn có thể choose sử dụng UUIDs thay thế. UUIDs là universally unique alpha-numeric identifiers có độ dài 36 characters.

Nếu bạn muốn một model sử dụng một UUID key thay vì một auto-incrementing integer key, bạn có thể sử dụng `Illuminate\Database\Eloquent\Concerns\HasUuids` trait trên model. Tất nhiên, bạn nên ensure rằng model có một [UUID equivalent primary key column](/docs/{{version}}/migrations#column-method-uuid):

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUuids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Europe']);

$article->id; // "018f2b5c-6a7f-7b12-9d6f-2f8a4e0c9c11"
```

Theo mặc định, `HasUuids` trait sẽ generate [UUIDv7](/docs/{{version}}/strings#method-str-uuid7) identifiers cho models của bạn. Những UUIDs này efficient hơn cho indexed database storage vì chúng có thể được sorted lexicographically.

Bạn có thể override UUID generation process cho một given model bằng cách defining một `newUniqueId` method trên model. Ngoài ra, bạn có thể specify những columns nên nhận UUIDs bằng cách defining một `uniqueIds` method trên model:

```php
use Ramsey\Uuid\Uuid;

/**
 * Generate a new UUID for the model.
 */
public function newUniqueId(): string
{
    return (string) Uuid::uuid4();
}

/**
 * Get the columns that should receive a unique identifier.
 *
 * @return array<int, string>
 */
public function uniqueIds(): array
{
    return ['id', 'discount_code'];
}
```

Nếu bạn muốn, bạn có thể choose utilize "ULIDs" thay vì UUIDs. ULIDs tương tự như UUIDs; tuy nhiên, chúng chỉ có độ dài 26 characters. Như ordered UUIDs, ULIDs là lexicographically sortable cho efficient database indexing. Để utilize ULIDs, bạn nên sử dụng `Illuminate\Database\Eloquent\Concerns\HasUlids` trait trên model của bạn. Bạn cũng nên ensure rằng model có một [ULID equivalent primary key column](/docs/{{version}}/migrations#column-method-ulid):

```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUlids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Asia']);

$article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"
```

<a name="timestamps"></a>
### Timestamps

Theo mặc định, Eloquent expects `created_at` và `updated_at` columns tồn tại trên model's corresponding database table. Eloquent sẽ automatically set những column's values này khi models được created hoặc updated. Nếu bạn không muốn những columns này được automatically managed bởi Eloquent, bạn có thể set `timestamps` thành `false` trên model's `Table` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(timestamps: false)]
class Flight extends Model
{
    // ...
}
```

Nếu bạn chỉ cần disable timestamps, bạn có thể sử dụng `WithoutTimestamps` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\WithoutTimestamps;
use Illuminate\Database\Eloquent\Model;

#[WithoutTimestamps]
class Flight extends Model
{
    // ...
}
```

Nếu bạn cần customize format của model's timestamps của bạn, bạn có thể sử dụng `dateFormat` argument trên `Table` attribute. Điều này determine cách date attributes được stored trong database cũng như format của chúng khi model được serialized thành một array hoặc JSON:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(dateFormat: 'U')]
class Flight extends Model
{
    // ...
}
```

Nếu bạn chỉ cần define một date format, bạn có thể sử dụng `DateFormat` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\DateFormat;
use Illuminate\Database\Eloquent\Model;

#[DateFormat('U')]
class Flight extends Model
{
    // ...
}
```

Nếu bạn cần customize names của columns được sử dụng để store timestamps, bạn có thể define `CREATED_AT` và `UPDATED_AT` constants trên model của bạn:

```php
<?php

class Flight extends Model
{
    /**
     * The name of the "created at" column.
     *
     * @var string|null
     */
    public const CREATED_AT = 'creation_date';

    /**
     * The name of the "updated at" column.
     *
     * @var string|null
     */
    public const UPDATED_AT = 'updated_date';
}
```

Nếu bạn muốn perform model operations mà không có model có `updated_at` timestamp của nó được modified, bạn có thể operate trên model trong một closure được given vào `withoutTimestamps` method:

```php
Model::withoutTimestamps(fn () => $post->increment('reads'));
```

<a name="database-connections"></a>
### Database Connections

Theo mặc định, tất cả Eloquent models sẽ sử dụng default database connection được configured cho application của bạn. Nếu bạn muốn specify một connection khác nên được sử dụng khi interacting với một particular model, bạn có thể sử dụng `Connection` attribute:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Connection;
use Illuminate\Database\Eloquent\Model;

#[Connection('mysql')]
class Flight extends Model
{
    // ...
}
```

<a name="default-attribute-values"></a>
### Giá trị mặc định của Attribute

Theo mặc định, một newly instantiated model instance sẽ không chứa bất kỳ attribute values nào. Nếu bạn muốn define default values cho một số model's attributes của bạn, bạn có thể define một `$attributes` property trên model của bạn. Attribute values được placed trong `$attributes` array nên ở trong raw, "storable" format của chúng như thể chúng vừa được read từ database:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The model's default values for attributes.
     *
     * @var array<string, mixed>
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
}
```

<a name="configuring-eloquent-strictness"></a>
### Cấu hình Eloquent Strictness

Laravel offers một số methods cho phép bạn configure Eloquent's behavior và "strictness" trong một variety của situations.

Đầu tiên, `preventLazyLoading` method accepts một optional boolean argument indicates nếu lazy loading nên được prevented. Ví dụ, bạn có thể wish chỉ disable lazy loading trong non-production environments để production environment của bạn sẽ continue function normally ngay cả khi một lazy loaded relationship accidentally present trong production code. Thông thường, method này nên được invoked trong `boot` method của application's `AppServiceProvider` của bạn:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

Ngoài ra, bạn có thể instruct Laravel để throw một exception khi attempting để fill một unfillable attribute bằng cách invoking `preventSilentlyDiscardingAttributes` method. Điều này có thể giúp prevent unexpected errors trong local development khi attempting để set một attribute chưa được added vào model's `fillable` array:

```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

<a name="retrieving-models"></a>
## Lấy Models

Khi bạn đã created một model và [its associated database table](/docs/{{version}}/migrations#generating-migrations), bạn đã sẵn sàng để bắt đầu retrieving data từ database của bạn. Bạn có thể think của mỗi Eloquent model như một powerful [query builder](/docs/{{version}}/queries) cho phép bạn fluently query database table associated với model. Model's `all` method sẽ retrieve tất cả records từ model's associated database table:

```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

<a name="building-queries"></a>
#### Building Queries

Eloquent `all` method sẽ return tất cả results trong model's table. Tuy nhiên, vì mỗi Eloquent model serves như một [query builder](/docs/{{version}}/queries), bạn có thể add additional constraints vào queries và sau đó invoke `get` method để retrieve results:

```php
$flights = Flight::where('active', 1)
    ->orderBy('name')
    ->limit(10)
    ->get();
```

> [!NOTE]
> Vì Eloquent models là query builders, bạn nên review tất cả methods được cung cấp bởi Laravel's [query builder](/docs/{{version}}/queries). Bạn có thể sử dụng bất kỳ methods nào trong số này khi writing Eloquent queries của bạn.

<a name="refreshing-models"></a>
#### Refreshing Models

Nếu bạn đã có một instance của một Eloquent model được retrieved từ database, bạn có thể "refresh" model sử dụng `fresh` và `refresh` methods. `fresh` method sẽ re-retrieve model từ database. Existing model instance sẽ không bị affected:

```php
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

`refresh` method sẽ re-hydrate existing model sử dụng fresh data từ database. Ngoài ra, tất cả loaded relationships của nó cũng sẽ được refreshed:

```php
$flight = Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

<a name="collections"></a>
### Collections

Như chúng ta đã seen, Eloquent methods như `all` và `get` retrieve multiple records từ database. Tuy nhiên, những methods này không return một plain PHP array. Thay vào đó, một instance của `Illuminate\Database\Eloquent\Collection` được returned.

Eloquent `Collection` class extends Laravel's base `Illuminate\Support\Collection` class, mà cung cấp một [variety của helpful methods](/docs/{{version}}/collections#available-methods) cho interacting với data collections. Ví dụ, `reject` method có thể được sử dụng để remove models từ một collection dựa trên results của một invoked closure:

```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;
});
```

Ngoài những methods được cung cấp bởi Laravel's base collection class, Eloquent collection class cung cấp [a few extra methods](/docs/{{version}}/eloquent-collections#available-methods) được specifically intended cho interacting với collections của Eloquent models.

Vì tất cả Laravel's collections implement PHP's iterable interfaces, bạn có thể loop qua collections như thể chúng là một array:

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

<a name="chunking-results"></a>
### Chunking Results

Application của bạn có thể run out of memory nếu bạn attempt để load tens of thousands của Eloquent records qua `all` hoặc `get` methods. Thay vì sử dụng những methods này, `chunk` method có thể được sử dụng để process large numbers của models efficient hơn.

`chunk` method sẽ retrieve một subset của Eloquent models, passing chúng vào một closure cho processing. Vì chỉ current chunk của Eloquent models được retrieved tại một thời điểm, `chunk` method sẽ cung cấp significantly reduced memory usage khi working với một large number của models:

```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

First argument được passed vào `chunk` method là số lượng records bạn muốn nhận per "chunk". Closure được passed như second argument sẽ được invoked cho mỗi chunk được retrieved từ database. Một database query sẽ được executed để retrieve mỗi chunk của records được passed vào closure.

Nếu bạn đang filtering results của `chunk` method dựa trên một column mà bạn cũng sẽ updating trong khi iterating qua results, bạn nên sử dụng `chunkById` method. Sử dụng `chunk` method trong những scenarios này có thể lead đến unexpected và inconsistent results. Internally, `chunkById` method sẽ luôn retrieve models với một `id` column lớn hơn last model trong previous chunk:

```php
Flight::where('departed', true)
    ->chunkById(200, function (Collection $flights) {
        $flights->each->update(['departed' => false]);
    }, column: 'id');
```

Vì `chunkById` và `lazyById` methods add "where" conditions của riêng họ vào query đang được executed, bạn nên typically [logically group](/docs/{{version}}/queries#logical-grouping) conditions của riêng bạn trong một closure:

```php
Flight::where(function ($query) {
    $query->where('delayed', true)->orWhere('cancelled', true);
})->chunkById(200, function (Collection $flights) {
    $flights->each->update([
        'departed' => false,
        'cancelled' => true
    ]);
}, column: 'id');
```

<a name="chunking-using-lazy-collections"></a>
### Chunk Sử Dụng Lazy Collections

`lazy` method hoạt động tương tự như [the `chunk` method](#chunking-results) theo nghĩa là, behind the scenes, nó executes query trong chunks. Tuy nhiên, thay vì passing mỗi chunk trực tiếp vào một callback như is, `lazy` method returns một flattened [LazyCollection](/docs/{{version}}/collections#lazy-collections) của Eloquent models, mà lets bạn interact với results như một single stream:

```php
use App\Models\Flight;

foreach (Flight::lazy() as $flight) {
    // ...
}
```

Nếu bạn đang filtering results của `lazy` method dựa trên một column mà bạn cũng sẽ updating trong khi iterating qua results, bạn nên sử dụng `lazyById` method. Internally, `lazyById` method sẽ luôn retrieve models với một `id` column lớn hơn last model trong previous chunk:

```php
Flight::where('departed', true)
    ->lazyById(200, column: 'id')
    ->each->update(['departed' => false]);
```

Bạn có thể filter results dựa trên descending order của `id` sử dụng `lazyByIdDesc` method.

<a name="cursors"></a>
### Cursors

Tương tự như `lazy` method, `cursor` method có thể được sử dụng để significantly reduce application's memory consumption của bạn khi iterating qua tens of thousands của Eloquent model records.

`cursor` method sẽ chỉ execute một single database query; tuy nhiên, individual Eloquent models sẽ không được hydrated cho đến khi chúng được actually iterated over. Do đó, chỉ một Eloquent model được giữ trong memory tại bất kỳ given time nào trong khi iterating qua cursor.

> [!WARNING]
> Vì `cursor` method chỉ ever holds một single Eloquent model trong memory tại một thời điểm, nó không thể eager load relationships. Nếu bạn cần eager load relationships, hãy consider sử dụng [the `lazy` method](#chunking-using-lazy-collections) thay thế.

Internally, `cursor` method sử dụng PHP [generators](https://www.php.net/manual/en/language.generators.overview.php) để implement functionality này:

```php
use App\Models\Flight;

foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    // ...
}
```

`cursor` returns một `Illuminate\Support\LazyCollection` instance. [Lazy collections](/docs/{{version}}/collections#lazy-collections) cho phép bạn sử dụng nhiều collection methods có sẵn trên typical Laravel collections trong khi chỉ loading một single model vào memory tại một thời điểm:

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

Mặc dù `cursor` method sử dụng far less memory hơn một regular query (bằng cách chỉ holding một single Eloquent model trong memory tại một thời điểm), nó vẫn sẽ eventually run out of memory. Điều này [do PHP's PDO driver internally caching tất cả raw query results trong buffer của nó](https://www.php.net/manual/en/mysqlinfo.concepts.buffering.php). Nếu bạn đang dealing với một very large number của Eloquent records, hãy consider sử dụng [the `lazy` method](#chunking-using-lazy-collections) thay thế.

<a name="advanced-subqueries"></a>
### Subquery Nâng Cao

<a name="subquery-selects"></a>
#### Subquery Selects

Eloquent cũng offers advanced subquery support, mà cho phép bạn pull information từ related tables trong một single query. Ví dụ, hãy imagine rằng chúng ta có một table của flight `destinations` và một table của `flights` đến destinations. `flights` table chứa một `arrived_at` column indicates khi flight arrived tại destination.

Sử dụng subquery functionality có sẵn cho query builder's `select` và `addSelect` methods, chúng ta có thể select tất cả `destinations` và name của flight mà most recently arrived tại destination đó sử dụng một single query:

```php
use App\Models\Destination;
use App\Models\Flight;

return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();
```

<a name="subquery-ordering"></a>
#### Subquery Ordering

Ngoài ra, query builder's `orderBy` function supports subqueries. Continuing sử dụng flight example của chúng ta, chúng ta có thể sử dụng functionality này để sort tất cả destinations dựa trên khi last flight arrived tại destination đó. Again, điều này có thể được done trong khi executing một single database query:

```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```

<a name="retrieving-single-models"></a>
## Lấy Single Models / Aggregates

Ngoài việc retrieving tất cả records matching một given query, bạn cũng có thể retrieve single records sử dụng `find`, `first`, hoặc `firstWhere` methods. Thay vì returning một collection của models, những methods này return một single model instance:

```php
use App\Models\Flight;

// Retrieve a model by its primary key...
$flight = Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = Flight::where('active', 1)->first();

// Alternative to retrieving the first model matching the query constraints...
$flight = Flight::firstWhere('active', 1);
```

Đôi khi bạn có thể wish perform một số other action nếu không có results nào được tìm thấy. `findOr` và `firstOr` methods sẽ return một single model instance hoặc, nếu không có results nào được tìm thấy, execute given closure. Value được returned bởi closure sẽ được considered như result của method:

```php
$flight = Flight::findOr(1, function () {
    // ...
});

$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});
```

<a name="not-found-exceptions"></a>
#### Not Found Exceptions

Đôi khi bạn có thể wish throw một exception nếu một model không được tìm thấy. Điều này đặc biệt hữu ích trong routes hoặc controllers. `findOrFail` và `firstOrFail` methods sẽ retrieve first result của query; tuy nhiên, nếu không có result nào được tìm thấy, một `Illuminate\Database\Eloquent\ModelNotFoundException` sẽ được thrown:

```php
$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();
```

Nếu `ModelNotFoundException` không được caught, một 404 HTTP response được automatically sent back đến client:

```php
use App\Models\Flight;

Route::get('/api/flights/{id}', function (string $id) {
    return Flight::findOrFail($id);
});
```

<a name="retrieving-or-creating-models"></a>
### Lấy hoặc Tạo Models

`firstOrCreate` method sẽ attempt để locate một database record sử dụng given column / value pairs. Nếu model không thể được tìm thấy trong database, một record sẽ được inserted với attributes resulting từ merging first array argument với optional second array argument.

`firstOrNew` method, như `firstOrCreate`, sẽ attempt để locate một record trong database matching given attributes. Tuy nhiên, nếu một model không được tìm thấy, một new model instance sẽ được returned. Note rằng model được returned bởi `firstOrNew` chưa được persisted vào database. Bạn sẽ cần manually call `save` method để persist nó:

```php
use App\Models\Flight;

// Retrieve flight by name or create it if it doesn't exist...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);

// Retrieve flight by name or create it with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// Retrieve flight by name or instantiate a new Flight instance...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);

// Retrieve flight by name or instantiate with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

<a name="retrieving-aggregates"></a>
### Lấy Aggregates

Khi interacting với Eloquent models, bạn cũng có thể sử dụng `count`, `sum`, `max`, và other [aggregate methods](/docs/{{version}}/queries#aggregates) được cung cấp bởi Laravel [query builder](/docs/{{version}}/queries). Như bạn có thể expect, những methods này return một scalar value thay vì một Eloquent model instance:

```php
$count = Flight::where('active', 1)->count();

$max = Flight::where('active', 1)->max('price');
```

<a name="inserting-and-updating-models"></a>
## Insert và Update Models

<a name="inserts"></a>
### Inserts

Tất nhiên, khi sử dụng Eloquent, chúng ta không chỉ cần retrieve models từ database. Chúng ta cũng cần insert new records. May mắn thay, Eloquent làm cho điều này đơn giản. Để insert một new record vào database, bạn nên instantiate một new model instance và set attributes trên model. Sau đó, call `save` method trên model instance:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Flight;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Store a new flight in the database.
     */
    public function store(Request $request): RedirectResponse
    {
        // Validate the request...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();

        return redirect('/flights');
    }
}
```

Trong example này, chúng ta assign `name` field từ incoming HTTP request vào `name` attribute của `App\Models\Flight` model instance. Khi chúng ta call `save` method, một record sẽ được inserted vào database. Model's `created_at` và `updated_at` timestamps sẽ automatically được set khi `save` method được called, nên không có need để set chúng manually.

Nếu bạn muốn save model trong một database transaction, bạn có thể sử dụng `saveOrFail` method. Nếu một exception được thrown trong save, transaction sẽ automatically được rolled back:

```php
$flight->saveOrFail();
```

Hoặc, bạn có thể sử dụng `create` method để "save" một new model sử dụng một single PHP statement. Inserted model instance sẽ được returned cho bạn bởi `create` method:

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

Tuy nhiên, trước khi sử dụng `create` method, bạn sẽ cần specify một `Fillable` hoặc `Guarded` attribute trên model class của bạn. Những attributes này được required vì tất cả Eloquent models được protected chống mass assignment vulnerabilities theo mặc định. Để tìm hiểu thêm về mass assignment, hãy consult [mass assignment documentation](#mass-assignment).

<a name="updates"></a>
### Updates

`save` method cũng có thể được sử dụng để update models đã tồn tại trong database. Để update một model, bạn nên retrieve nó và set bất kỳ attributes bạn muốn update. Sau đó, bạn nên call model's `save` method. Again, `updated_at` timestamp sẽ automatically được updated, nên không có need để manually set value của nó:

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
```

Nếu bạn muốn update model trong một database transaction, bạn có thể sử dụng `updateOrFail` method. Nếu một exception được thrown trong update, transaction sẽ automatically được rolled back:

```php
$flight->updateOrFail(['name' => 'Paris to London']);
```

Đôi khi, bạn có thể cần update một existing model hoặc create một new model nếu không có matching model nào tồn tại. Như `firstOrCreate` method, `updateOrCreate` method persists model, nên không có need để manually call `save` method.

Trong example dưới đây, nếu một flight tồn tại với một `departure` location của `Oakland` và một `destination` location của `San Diego`, `price` và `discounted` columns của nó sẽ được updated. Nếu không có flight nào như vậy tồn tại, một new flight sẽ được created có attributes resulting từ merging first argument array với second argument array:

```php
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

Khi sử dụng methods như `firstOrCreate` hoặc `updateOrCreate`, bạn có thể không biết liệu một new model đã được created hay một existing one đã được updated. `wasRecentlyCreated` property indicates nếu model được created trong current lifecycle của nó:

```php
$flight = Flight::updateOrCreate(
    // ...
);

if ($flight->wasRecentlyCreated) {
    // New flight record was inserted...
}
```

<a name="mass-updates"></a>
#### Mass Updates

Updates cũng có thể được performed đối với models matching một given query. Trong example này, tất cả flights là `active` và có một `destination` của `San Diego` sẽ được marked như delayed:

```php
Flight::where('active', 1)
    ->where('destination', 'San Diego')
    ->update(['delayed' => 1]);
```

`update` method expects một array của column và value pairs representing columns nên được updated. `update` method returns số lượng affected rows.

> [!WARNING]
> Khi issuing một mass update qua Eloquent, `saving`, `saved`, `updating`, và `updated` model events sẽ không được fired cho updated models. Điều này là vì models không bao giờ được actually retrieved khi issuing một mass update.

<a name="examining-attribute-changes"></a>
#### Examining Attribute Changes

Eloquent cung cấp `isDirty`, `isClean`, và `wasChanged` methods để examine internal state của model của bạn và determine cách attributes của nó đã changed từ khi model được originally retrieved.

`isDirty` method determines nếu bất kỳ model's attributes nào đã changed kể từ khi model được retrieved. Bạn có thể pass một specific attribute name hoặc một array của attributes vào `isDirty` method để determine nếu bất kỳ attributes nào là "dirty". `isClean` method sẽ determine nếu một attribute đã remained unchanged kể từ khi model được retrieved. Method này cũng accepts một optional attribute argument:

```php
use App\Models\User;

$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->isDirty(); // true
$user->isDirty('title'); // true
$user->isDirty('first_name'); // false
$user->isDirty(['first_name', 'title']); // true

$user->isClean(); // false
$user->isClean('title'); // false
$user->isClean('first_name'); // true
$user->isClean(['first_name', 'title']); // false

$user->save();

$user->isDirty(); // false
$user->isClean(); // true
```

`wasChanged` method determines nếu bất kỳ attributes nào đã changed khi model được last saved trong current request cycle. Nếu cần thiết, bạn có thể pass một attribute name để xem nếu một particular attribute đã changed:

```php
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->save();

$user->wasChanged(); // true
$user->wasChanged('title'); // true
$user->wasChanged(['title', 'slug']); // true
$user->wasChanged('first_name'); // false
$user->wasChanged(['first_name', 'title']); // true
```

`getOriginal` method returns một array chứa original attributes của model bất kể bất kỳ changes nào đến model kể từ khi nó được retrieved. Nếu cần thiết, bạn có thể pass một specific attribute name để get original value của một particular attribute:

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->name = 'Jack';
$user->name; // Jack

$user->getOriginal('name'); // John
$user->getOriginal(); // Array of original attributes...
```

`getChanges` method returns một array chứa attributes đã changed khi model được last saved, trong khi `getPrevious` method returns một array chứa original attribute values trước khi model được last saved:

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->update([
    'name' => 'Jack',
    'email' => 'jack@example.com',
]);

$user->getChanges();

/*
    [
        'name' => 'Jack',
        'email' => 'jack@example.com',
    ]
*/

$user->getPrevious();

/*
    [
        'name' => 'John',
        'email' => 'john@example.com',
    ]
*/
```

<a name="mass-assignment"></a>
### Mass Assignment

Bạn có thể sử dụng `create` method để "save" một new model sử dụng một single PHP statement. Inserted model instance sẽ được returned cho bạn bởi method:

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

Tuy nhiên, trước khi sử dụng `create` method, bạn sẽ cần specify một `Fillable` hoặc `Guarded` attribute trên model class của bạn. Những attributes này được required vì tất cả Eloquent models được protected chống mass assignment vulnerabilities theo mặc định.

Một mass assignment vulnerability xảy ra khi một user passes một unexpected HTTP request field và field đó changes một column trong database của bạn mà bạn không expect. Ví dụ, một malicious user có thể send một `is_admin` parameter qua một HTTP request, mà sau đó được passed vào model's `create` method của bạn, cho phép user escalate themselves thành một administrator.

Vì vậy, để bắt đầu, bạn nên define những model attributes bạn muốn make mass assignable. Bạn có thể làm điều này sử dụng `Fillable` attribute trên model. Ví dụ, hãy làm `name` attribute của `Flight` model của chúng ta mass assignable:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Model;

#[Fillable(['name'])]
class Flight extends Model
{
    // ...
}
```

Khi bạn đã specified những attributes là mass assignable, bạn có thể sử dụng `create` method để insert một new record vào database. `create` method returns newly created model instance:

```php
$flight = Flight::create(['name' => 'London to Paris']);
```

Nếu bạn đã có một model instance, bạn có thể sử dụng `fill` method để populate nó với một array của attributes:

```php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```

<a name="mass-assignment-json-columns"></a>
#### Mass Assignment và JSON Columns

Khi assigning JSON columns, mỗi column's mass assignable key phải được specified trong model's `Fillable` attribute của bạn. Để đảm bảo security, Laravel không support updating nested JSON attributes khi sử dụng `Guarded` attribute:

```php
use Illuminate\Database\Eloquent\Attributes\Fillable;

#[Fillable(['options->enabled'])]
class Flight extends Model
{
    // ...
}
```

<a name="allowing-mass-assignment"></a>
#### Allowing Mass Assignment

Nếu bạn muốn make tất cả attributes của bạn mass assignable, bạn có thể sử dụng `Unguarded` attribute trên model của bạn. Nếu bạn chọn để unguard model của bạn, bạn nên take special care để luôn hand-craft arrays được passed vào Eloquent's `fill`, `create`, và `update` methods:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Unguarded;
use Illuminate\Database\Eloquent\Model;

#[Unguarded]
class Flight extends Model
{
    // ...
}
```

<a name="mass-assignment-exceptions"></a>
#### Mass Assignment Exceptions

Theo mặc định, attributes không được included trong `Fillable` attribute được silently discarded khi performing mass-assignment operations. Trong production, đây là expected behavior; tuy nhiên, trong local development nó có thể lead đến confusion về lý do model changes không taking effect.

Nếu bạn muốn, bạn có thể instruct Laravel để throw một exception khi attempting để fill một unfillable attribute bằng cách invoking `preventSilentlyDiscardingAttributes` method. Thông thường, method này nên được invoked trong `boot` method của application's `AppServiceProvider` class của bạn:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventSilentlyDiscardingAttributes($this->app->isLocal());
}
```

<a name="upserts"></a>
### Upserts

Eloquent's `upsert` method có thể được sử dụng để update hoặc create records trong một single, atomic operation. Method's first argument consists của values để insert hoặc update, trong khi second argument lists column(s) uniquely identify records trong associated table. Method's third và final argument là một array của columns nên được updated nếu một matching record đã tồn tại trong database. `upsert` method sẽ automatically set `created_at` và `updated_at` timestamps nếu timestamps được enabled trên model:

```php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], uniqueBy: ['departure', 'destination'], update: ['price']);
```

> [!WARNING]
> Tất cả databases ngoại trừ SQL Server require columns trong second argument của `upsert` method để có một "primary" hoặc "unique" index. Ngoài ra, MariaDB và MySQL database drivers ignore second argument của `upsert` method và luôn sử dụng "primary" và "unique" indexes của table để detect existing records.

<a name="deleting-models"></a>
## Xóa Models

Để delete một model, bạn có thể call `delete` method trên model instance:

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```

Nếu bạn muốn delete model trong một database transaction, bạn có thể sử dụng `deleteOrFail` method. Nếu một exception được thrown trong delete, transaction sẽ automatically được rolled back:

```php
$flight->deleteOrFail();
```

<a name="deleting-an-existing-model-by-its-primary-key"></a>
#### Deleting an Existing Model by its Primary Key

Trong example trên, chúng ta đang retrieving model từ database trước khi calling `delete` method. Tuy nhiên, nếu bạn biết primary key của model, bạn có thể delete model mà không explicitly retrieving nó bằng cách calling `destroy` method. Ngoài việc accepting single primary key, `destroy` method sẽ accept multiple primary keys, một array của primary keys, hoặc một [collection](/docs/{{version}}/collections) của primary keys:

```php
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);

Flight::destroy(collect([1, 2, 3]));
```

Nếu bạn đang utilizing [soft deleting models](#soft-deleting), bạn có thể permanently delete models qua `forceDestroy` method:

```php
Flight::forceDestroy(1);
```

> [!WARNING]
> `destroy` method loads mỗi model individually và calls `delete` method để `deleting` và `deleted` events được properly dispatched cho mỗi model.

<a name="deleting-models-using-queries"></a>
#### Deleting Models Using Queries

Tất nhiên, bạn có thể build một Eloquent query để delete tất cả models matching query's criteria của bạn. Trong example này, chúng ta sẽ delete tất cả flights được marked như inactive. Như mass updates, mass deletes sẽ không dispatch model events cho models được deleted:

```php
$deleted = Flight::where('active', 0)->delete();
```

Để delete tất cả models trong một table, bạn nên execute một query mà không adding bất kỳ conditions:

```php
$deleted = Flight::query()->delete();
```

> [!WARNING]
> Khi executing một mass delete statement qua Eloquent, `deleting` và `deleted` model events sẽ không được dispatched cho deleted models. Điều này là vì models không bao giờ actually retrieved khi executing delete statement.

<a name="soft-deleting"></a>
### Soft Deleting

Ngoài việc actually removing records từ database của bạn, Eloquent cũng có thể "soft delete" models. Khi models được soft deleted, chúng không actually removed từ database của bạn. Thay vào đó, một `deleted_at` attribute được set trên model indicating date và time tại đó model được "deleted". Để enable soft deletes cho một model, add `Illuminate\Database\Eloquent\SoftDeletes` trait vào model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```

> [!NOTE]
> The `SoftDeletes` trait will automatically cast the `deleted_at` attribute to a `DateTime` / `Carbon` instance for you.

Bạn cũng nên add `deleted_at` column vào database table của bạn. Laravel [schema builder](/docs/{{version}}/migrations) chứa một helper method để create column này:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});

Schema::table('flights', function (Blueprint $table) {
    $table->dropSoftDeletes();
});
```

Bây giờ, khi bạn call `delete` method trên model, `deleted_at` column sẽ được set đến current date và time. Tuy nhiên, model's database record sẽ được left trong table. Khi querying một model sử dụng soft deletes, soft deleted models sẽ automatically được excluded từ tất cả query results.

Để determine nếu một given model instance đã được soft deleted, bạn có thể sử dụng `trashed` method:

```php
if ($flight->trashed()) {
    // ...
}
```

<a name="restoring-soft-deleted-models"></a>
#### Restoring Soft Deleted Models

Đôi khi bạn có thể muốn "un-delete" một soft deleted model. Để restore một soft deleted model, bạn có thể call `restore` method trên một model instance. `restore` method sẽ set model's `deleted_at` column đến `null`:

```php
$flight->restore();
```

Bạn cũng có thể sử dụng `restore` method trong một query để restore multiple models. Một lần nữa, như các "mass" operations khác, điều này sẽ không dispatch bất kỳ model events nào cho models được restored:

```php
Flight::withTrashed()
    ->where('airline_id', 1)
    ->restore();
```

`restore` method cũng có thể được sử dụng khi building [relationship](/docs/{{version}}/eloquent-relationships) queries:

```php
$flight->history()->restore();
```

<a name="permanently-deleting-models"></a>
#### Permanently Deleting Models

Đôi khi bạn có thể cần truly remove một model từ database của bạn. Bạn có thể sử dụng `forceDelete` method để permanently remove một soft deleted model từ database table:

```php
$flight->forceDelete();
```

Bạn cũng có thể sử dụng `forceDelete` method khi building Eloquent relationship queries:

```php
$flight->history()->forceDelete();
```

<a name="querying-soft-deleted-models"></a>
### Querying Soft Deleted Models

<a name="including-soft-deleted-models"></a>
#### Including Soft Deleted Models

Như noted ở trên, soft deleted models sẽ automatically được excluded từ query results. Tuy nhiên, bạn có thể force soft deleted models được included trong query's results bằng cách calling `withTrashed` method trên query:

```php
use App\Models\Flight;

$flights = Flight::withTrashed()
    ->where('account_id', 1)
    ->get();
```

`withTrashed` method cũng có thể được called khi building một [relationship](/docs/{{version}}/eloquent-relationships) query:

```php
$flight->history()->withTrashed()->get();
```

<a name="retrieving-only-soft-deleted-models"></a>
#### Retrieving Only Soft Deleted Models

`onlyTrashed` method sẽ retrieve **only** soft deleted models:

```php
$flights = Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

<a name="pruning-models"></a>
## Pruning Models

Đôi khi bạn có thể muốn periodically delete models không còn needed. Để accomplish điều này, bạn có thể add `Illuminate\Database\Eloquent\Prunable` hoặc `Illuminate\Database\Eloquent\MassPrunable` trait vào models bạn muốn periodically prune. Sau khi adding một trong traits vào model, implement một `prunable` method mà returns một Eloquent query builder resolves models không còn needed:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Prunable;

class Flight extends Model
{
    use Prunable;

    /**
     * Get the prunable model query.
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->minus(months: 1));
    }
}
```

Khi marking models như `Prunable`, bạn cũng có thể define một `pruning` method trên model. Method này sẽ được called trước khi model được deleted. Method này có thể useful để delete bất kỳ additional resources associated với model, như stored files, trước khi model được permanently removed từ database:

```php
/**
 * Prepare the model for pruning.
 */
protected function pruning(): void
{
    // ...
}
```

Sau khi configuring prunable model của bạn, bạn nên schedule `model:prune` Artisan command trong application's `routes/console.php` file của bạn. Bạn được free để chọn appropriate interval tại đó command này nên được run:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('model:prune')->daily();
```

Behind the scenes, `model:prune` command sẽ automatically detect "Prunable" models trong application's `app/Models` directory của bạn. Nếu models của bạn ở một different location, bạn có thể sử dụng `--model` option để specify model class names:

```php
Schedule::command('model:prune', [
    '--model' => [Address::class, Flight::class],
])->daily();
```

Nếu bạn muốn exclude certain models từ being pruned trong khi pruning tất cả other detected models, bạn có thể sử dụng `--except` option:

```php
Schedule::command('model:prune', [
    '--except' => [Address::class, Flight::class],
])->daily();
```

Bạn có thể test `prunable` query của bạn bằng cách executing `model:prune` command với `--pretend` option. Khi pretending, `model:prune` command sẽ đơn giản report bao nhiêu records sẽ được pruned nếu command được actually run:

```shell
php artisan model:prune --pretend
```

> [!WARNING]
> Soft deleting models sẽ được permanently deleted (`forceDelete`) nếu chúng match prunable query.

<a name="mass-pruning"></a>
#### Mass Pruning

Khi models được marked với `Illuminate\Database\Eloquent\MassPrunable` trait, models được deleted từ database sử dụng mass-deletion queries. Do đó, `pruning` method sẽ không được invoked, cũng như `deleting` và `deleted` model events sẽ không được dispatched. Điều này là vì models không bao giờ actually retrieved trước khi deletion, do đó làm pruning process efficient hơn nhiều:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\MassPrunable;

class Flight extends Model
{
    use MassPrunable;

    /**
     * Get the prunable model query.
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->minus(months: 1));
    }
}
```

<a name="replicating-models"></a>
## Replicating Models

Bạn có thể create một unsaved copy của một existing model instance sử dụng `replicate` method. Method này đặc biệt useful khi bạn có model instances share nhiều của cùng attributes:

```php
use App\Models\Address;

$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);

$billing = $shipping->replicate()->fill([
    'type' => 'billing'
]);

$billing->save();
```

Để exclude một hoặc nhiều attributes từ being replicated đến new model, bạn có thể pass một array đến `replicate` method:

```php
$flight = Flight::create([
    'destination' => 'LAX',
    'origin' => 'LHR',
    'last_flown' => '2020-03-04 11:00:00',
    'last_pilot_id' => 747,
]);

$flight = $flight->replicate([
    'last_flown',
    'last_pilot_id'
]);
```

<a name="query-scopes"></a>
## Query Scopes

<a name="global-scopes"></a>
### Global Scopes

Global scopes allow bạn để add constraints đến tất cả queries cho một given model. Laravel's own [soft delete](#soft-deleting) functionality utilizes global scopes để chỉ retrieve "non-deleted" models từ database. Writing your own global scopes có thể provide một convenient, easy way để make sure mỗi query cho một given model receives certain constraints.

<a name="generating-scopes"></a>
#### Generating Scopes

Để generate một new global scope, bạn có thể invoke `make:scope` Artisan command, mà sẽ place generated scope trong application's `app/Models/Scopes` directory của bạn:

```shell
php artisan make:scope AncientScope
```

<a name="writing-global-scopes"></a>
#### Writing Global Scopes

Writing một global scope là đơn giản. Đầu tiên, sử dụng `make:scope` command để generate một class implements `Illuminate\Database\Eloquent\Scope` interface. `Scope` interface requires bạn để implement một method: `apply`. `apply` method có thể add `where` constraints hoặc other types của clauses đến query khi cần thiết:

```php
<?php

namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class AncientScope implements Scope
{
    /**
     * Apply the scope to a given Eloquent query builder.
     */
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('created_at', '<', now()->minus(years: 2000));
    }
}
```

> [!NOTE]
> Nếu global scope của bạn đang adding columns đến select clause của query, bạn nên sử dụng `addSelect` method thay vì `select`. Điều này sẽ prevent unintentional replacement của query's existing select clause.

<a name="applying-global-scopes"></a>
#### Applying Global Scopes

Để assign một global scope đến một model, bạn có thể đơn giản place `ScopedBy` attribute trên model:

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Attributes\ScopedBy;

#[ScopedBy([AncientScope::class])]
class User extends Model
{
    //
}
```

Hoặc, bạn có thể manually register global scope bằng cách overriding model's `booted` method và invoke model's `addGlobalScope` method. `addGlobalScope` method accepts một instance của scope của bạn như its only argument:

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::addGlobalScope(new AncientScope);
    }
}
```

Sau khi adding scope trong example trên đến `App\Models\User` model, một call đến `User::all()` method sẽ execute following SQL query:

```sql
select * from `users` where `created_at` < 0021-02-18 00:00:00
```

<a name="anonymous-global-scopes"></a>
#### Anonymous Global Scopes

Eloquent cũng allows bạn để define global scopes sử dụng closures, mà đặc biệt useful cho simple scopes không warrant một separate class của riêng chúng. Khi defining một global scope sử dụng một closure, bạn nên provide một scope name của riêng bạn như first argument đến `addGlobalScope` method:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::addGlobalScope('ancient', function (Builder $builder) {
            $builder->where('created_at', '<', now()->minus(years: 2000));
        });
    }
}
```

<a name="removing-global-scopes"></a>
#### Removing Global Scopes

Nếu bạn muốn remove một global scope cho một given query, bạn có thể sử dụng `withoutGlobalScope` method. Method này accepts class name của global scope như its only argument:

```php
User::withoutGlobalScope(AncientScope::class)->get();
```

Hoặc, nếu bạn defined global scope sử dụng một closure, bạn nên pass string name mà bạn assigned đến global scope:

```php
User::withoutGlobalScope('ancient')->get();
```

Nếu bạn muốn remove several hoặc thậm chí tất cả của query's global scopes, bạn có thể sử dụng `withoutGlobalScopes` và `withoutGlobalScopesExcept` methods:

```php
// Remove tất cả của global scopes...
User::withoutGlobalScopes()->get();

// Remove một số của global scopes...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();

// Remove tất cả global scopes ngoại trừ given ones...
User::withoutGlobalScopesExcept([
    SecondScope::class,
])->get();
```

<a name="local-scopes"></a>
### Local Scopes

Local scopes allow bạn để define common sets của query constraints mà bạn có thể easily re-use throughout application của bạn. Ví dụ, bạn có thể cần frequently retrieve tất cả users được considered "popular". Để define một scope, add `Scope` attribute đến một Eloquent method.

Scopes nên luôn return cùng query builder instance hoặc `void`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     */
    #[Scope]
    protected function popular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }

    /**
     * Scope a query to only include active users.
     */
    #[Scope]
    protected function active(Builder $query): void
    {
        $query->where('active', 1);
    }
}
```

<a name="utilizing-a-local-scope"></a>
#### Utilizing a Local Scope

Khi scope đã được defined, bạn có thể call scope methods khi querying model. Bạn thậm chí có thể chain calls đến various scopes:

```php
use App\Models\User;

$users = User::popular()->active()->orderBy('created_at')->get();
```

Combining multiple Eloquent model scopes qua một `or` query operator có thể require sử dụng closures để achieve correct [logical grouping](/docs/{{version}}/queries#logical-grouping):

```php
$users = User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

Tuy nhiên, vì điều này có thể cumbersome, Laravel provides một "higher order" `orWhere` method allows bạn để fluently chain scopes cùng nhau mà không sử dụng closures:

```php
$users = User::popular()->orWhere->active()->get();
```

<a name="dynamic-scopes"></a>
#### Dynamic Scopes

Đôi khi bạn có thể muốn define một scope accepts parameters. Để bắt đầu, chỉ cần add additional parameters của bạn đến scope method's signature của bạn. Scope parameters nên được defined sau `$query` parameter:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     */
    #[Scope]
    protected function ofType(Builder $query, string $type): void
    {
        $query->where('type', $type);
    }
}
```

Khi expected arguments đã được added đến scope method's signature của bạn, bạn có thể pass arguments khi calling scope:

```php
$users = User::ofType('admin')->get();
```

Attributed scope methods nên là `protected`. Khi calling một attributed scope từ trong model class, call scope qua một query builder instance, như `static::query()->ofType('admin')`, để ensure call được routed qua Eloquent's scope handling.

<a name="pending-attributes"></a>
### Pending Attributes

Nếu bạn muốn sử dụng scopes để create models có cùng attributes như những được sử dụng để constrain scope, bạn có thể sử dụng `withAttributes` method khi building scope query:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * Scope the query to only include drafts.
     */
    #[Scope]
    protected function draft(Builder $query): void
    {
        $query->withAttributes([
            'hidden' => true,
        ]);
    }
}
```

`withAttributes` method sẽ add `where` conditions đến query sử dụng given attributes, và nó cũng sẽ add given attributes đến bất kỳ models được created qua scope:

```php
$draft = Post::draft()->create(['title' => 'In Progress']);

$draft->hidden; // true
```

Để instruct `withAttributes` method để không add `where` conditions đến query, bạn có thể set `asConditions` argument đến `false`:

```php
$query->withAttributes([
    'hidden' => true,
], asConditions: false);
```

<a name="comparing-models"></a>
## Comparing Models

Đôi khi bạn có thể cần determine nếu hai models là "same" hoặc không. `is` và `isNot` methods có thể được sử dụng để quickly verify hai models có cùng primary key, table, và database connection hoặc không:

```php
if ($post->is($anotherPost)) {
    // ...
}

if ($post->isNot($anotherPost)) {
    // ...
}
```

`is` và `isNot` methods cũng available khi sử dụng `belongsTo`, `hasOne`, `morphTo`, và `morphOne` [relationships](/docs/{{version}}/eloquent-relationships). Method này đặc biệt helpful khi bạn muốn compare một related model mà không issuing một query để retrieve model đó:

```php
if ($post->author()->is($user)) {
    // ...
}
```

<a name="events"></a>
## Events

> [!NOTE]
> Muốn broadcast Eloquent events của bạn trực tiếp đến client-side application của bạn? Check out Laravel's [model event broadcasting](/docs/{{version}}/broadcasting#model-broadcasting).

Eloquent models dispatch several events, allowing bạn để hook vào following moments trong model's lifecycle: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `trashed`, `forceDeleting`, `forceDeleted`, `restoring`, `restored`, và `replicating`.

`retrieved` event sẽ dispatch khi một existing model được retrieved từ database. Khi một new model được saved lần đầu tiên, `creating` và `created` events sẽ dispatch. `updating` / `updated` events sẽ dispatch khi một existing model được modified và `save` method được called. `saving` / `saved` events sẽ dispatch khi một model được created hoặc updated - ngay cả khi model's attributes không được changed. Event names ending với `-ing` được dispatched trước bất kỳ changes đến model được persisted, trong khi events ending với `-ed` được dispatched sau khi changes đến model được persisted.

Để bắt đầu listening đến model events, define một `$dispatchesEvents` property trên Eloquent model của bạn. Property này maps various points của Eloquent model's lifecycle đến [event classes](/docs/{{version}}/events) của riêng bạn. Mỗi model event class nên expect để receive một instance của affected model qua constructor của nó:

```php
<?php

namespace App\Models;

use App\Events\UserDeleted;
use App\Events\UserSaved;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The event map for the model.
     *
     * @var array<string, string>
     */
    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```

Sau khi defining và mapping Eloquent events của bạn, bạn có thể sử dụng [event listeners](/docs/{{version}}/events#defining-listeners) để handle events.

> [!WARNING]
> Khi issuing một mass update hoặc delete query qua Eloquent, `saved`, `updated`, `deleting`, và `deleted` model events sẽ không được dispatched cho affected models. Điều này là vì models không bao giờ actually retrieved khi performing mass updates hoặc deletes.

<a name="events-using-closures"></a>
### Using Closures

Thay vì sử dụng custom event classes, bạn có thể register closures execute khi various model events được dispatched. Thông thường, bạn nên register những closures này trong `booted` method của model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::created(function (User $user) {
            // ...
        });
    }
}
```

Nếu cần thiết, bạn có thể utilize [queueable anonymous event listeners](/docs/{{version}}/events#queueable-anonymous-event-listeners) khi registering model events. Điều này sẽ instruct Laravel để execute model event listener trong background sử dụng application's [queue](/docs/{{version}}/queues) của bạn:

```php
use function Illuminate\Events\queueable;

static::created(queueable(function (User $user) {
    // ...
}));
```

<a name="observers"></a>
### Observers

<a name="defining-observers"></a>
#### Defining Observers

Nếu bạn đang listening cho nhiều events trên một given model, bạn có thể sử dụng observers để group tất cả listeners của bạn vào một single class. Observer classes có method names reflect Eloquent events bạn muốn listen for. Mỗi trong những methods này receives affected model như only argument của họ. `make:observer` Artisan command là easiest way để create một new observer class:

```shell
php artisan make:observer UserObserver --model=User
```

Command này sẽ place new observer trong `app/Observers` directory của bạn. Nếu directory này không tồn tại, Artisan sẽ create nó cho bạn. Fresh observer của bạn sẽ trông như sau:

```php
<?php

namespace App\Observers;

use App\Models\User;

class UserObserver
{
    /**
     * Handle the User "created" event.
     */
    public function created(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "updated" event.
     */
    public function updated(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "deleted" event.
     */
    public function deleted(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "restored" event.
     */
    public function restored(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "forceDeleted" event.
     */
    public function forceDeleted(User $user): void
    {
        // ...
    }
}
```

Để register một observer, bạn có thể place `ObservedBy` attribute trên corresponding model:

```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([UserObserver::class])]
class User extends Authenticatable
{
    //
}
```

Hoặc, bạn có thể manually register một observer bằng cách invoking `observe` method trên model bạn muốn observe. Bạn có thể register observers trong `boot` method của application's `AppServiceProvider` class của bạn:

```php
use App\Models\User;
use App\Observers\UserObserver;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    User::observe(UserObserver::class);
}
```

> [!NOTE]
> Có additional events một observer có thể listen to, như `saving` và `retrieved`. Những events được described trong [events](#events) documentation.

<a name="observers-and-database-transactions"></a>
#### Observers và Database Transactions

Khi models đang được created trong một database transaction, bạn có thể muốn instruct một observer để chỉ execute event handlers của nó sau khi database transaction được committed. Bạn có thể accomplish điều này bằng cách implementing `ShouldHandleEventsAfterCommit` interface trên observer của bạn. Nếu một database transaction không in progress, event handlers sẽ execute immediately:

```php
<?php

namespace App\Observers;

use App\Models\User;
use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;

class UserObserver implements ShouldHandleEventsAfterCommit
{
    /**
     * Handle the User "created" event.
     */
    public function created(User $user): void
    {
        // ...
    }
}
```

<a name="muting-events"></a>
### Muting Events

Đôi khi bạn có thể cần temporarily "mute" tất cả events fired bởi một model. Bạn có thể achieve điều này sử dụng `withoutEvents` method. `withoutEvents` method accepts một closure như only argument của nó. Bất kỳ code executed trong closure này sẽ không dispatch model events, và bất kỳ value returned bởi closure sẽ được returned bởi `withoutEvents` method:

```php
use App\Models\User;

$user = User::withoutEvents(function () {
    User::findOrFail(1)->delete();

    return User::find(2);
});
```

<a name="saving-a-single-model-without-events"></a>
#### Saving a Single Model Without Events

Đôi khi bạn có thể muốn "save" một given model mà không dispatching bất kỳ events. Bạn có thể accomplish điều này sử dụng `saveQuietly` method:

```php
$user = User::findOrFail(1);

$user->name = 'Victoria Faith';

$user->saveQuietly();
```

Bạn cũng có thể "update", "delete", "soft delete", "restore", và "replicate" một given model mà không dispatching bất kỳ events:

```php
$user->deleteQuietly();
$user->forceDeleteQuietly();
$user->restoreQuietly();
```

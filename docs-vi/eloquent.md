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

Eloquent provides the `isDirty`, `isClean`, and `wasChanged` methods to examine the internal state of your model and determine how its attributes have changed from when the model was originally retrieved.

The `isDirty` method determines if any of the model's attributes have been changed since the model was retrieved. You may pass a specific attribute name or an array of attributes to the `isDirty` method to determine if any of the attributes are "dirty". The `isClean` method will determine if an attribute has remained unchanged since the model was retrieved. This method also accepts an optional attribute argument:

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

The `wasChanged` method determines if any attributes were changed when the model was last saved within the current request cycle. If needed, you may pass an attribute name to see if a particular attribute was changed:

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

The `getOriginal` method returns an array containing the original attributes of the model regardless of any changes to the model since it was retrieved. If needed, you may pass a specific attribute name to get the original value of a particular attribute:

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->name = 'Jack';
$user->name; // Jack

$user->getOriginal('name'); // John
$user->getOriginal(); // Array of original attributes...
```

The `getChanges` method returns an array containing the attributes that changed when the model was last saved, while the `getPrevious` method returns an array containing the original attribute values before the model was last saved:

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

You may use the `create` method to "save" a new model using a single PHP statement. The inserted model instance will be returned to you by the method:

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

However, before using the `create` method, you will need to specify either a `Fillable` or `Guarded` attribute on your model class. These attributes are required because all Eloquent models are protected against mass assignment vulnerabilities by default.

A mass assignment vulnerability occurs when a user passes an unexpected HTTP request field and that field changes a column in your database that you did not expect. For example, a malicious user might send an `is_admin` parameter through an HTTP request, which is then passed to your model's `create` method, allowing the user to escalate themselves to an administrator.

So, to get started, you should define which model attributes you want to make mass assignable. You may do this using the `Fillable` attribute on the model. For example, let's make the `name` attribute of our `Flight` model mass assignable:

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

Once you have specified which attributes are mass assignable, you may use the `create` method to insert a new record in the database. The `create` method returns the newly created model instance:

```php
$flight = Flight::create(['name' => 'London to Paris']);
```

If you already have a model instance, you may use the `fill` method to populate it with an array of attributes:

```php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```

<a name="mass-assignment-json-columns"></a>
#### Mass Assignment and JSON Columns

When assigning JSON columns, each column's mass assignable key must be specified in your model's `Fillable` attribute. For security, Laravel does not support updating nested JSON attributes when using the `Guarded` attribute:

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

If you would like to make all of your attributes mass assignable, you may use the `Unguarded` attribute on your model. If you choose to unguard your model, you should take special care to always hand-craft the arrays passed to Eloquent's `fill`, `create`, and `update` methods:

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

By default, attributes that are not included in the `Fillable` attribute are silently discarded when performing mass-assignment operations. In production, this is expected behavior; however, during local development it can lead to confusion as to why model changes are not taking effect.

If you wish, you may instruct Laravel to throw an exception when attempting to fill an unfillable attribute by invoking the `preventSilentlyDiscardingAttributes` method. Typically, this method should be invoked in the `boot` method of your application's `AppServiceProvider` class:

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

Eloquent's `upsert` method may be used to update or create records in a single, atomic operation. The method's first argument consists of the values to insert or update, while the second argument lists the column(s) that uniquely identify records within the associated table. The method's third and final argument is an array of the columns that should be updated if a matching record already exists in the database. The `upsert` method will automatically set the `created_at` and `updated_at` timestamps if timestamps are enabled on the model:

```php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], uniqueBy: ['departure', 'destination'], update: ['price']);
```

> [!WARNING]
> All databases except SQL Server require the columns in the second argument of the `upsert` method to have a "primary" or "unique" index. In addition, the MariaDB and MySQL database drivers ignore the second argument of the `upsert` method and always use the "primary" and "unique" indexes of the table to detect existing records.

<a name="deleting-models"></a>
## Deleting Models

To delete a model, you may call the `delete` method on the model instance:

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```

If you would like to delete the model within a database transaction, you may use the `deleteOrFail` method. If an exception is thrown during the delete, the transaction will automatically be rolled back:

```php
$flight->deleteOrFail();
```

<a name="deleting-an-existing-model-by-its-primary-key"></a>
#### Deleting an Existing Model by its Primary Key

In the example above, we are retrieving the model from the database before calling the `delete` method. However, if you know the primary key of the model, you may delete the model without explicitly retrieving it by calling the `destroy` method. In addition to accepting the single primary key, the `destroy` method will accept multiple primary keys, an array of primary keys, or a [collection](/docs/{{version}}/collections) of primary keys:

```php
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);

Flight::destroy(collect([1, 2, 3]));
```

If you are utilizing [soft deleting models](#soft-deleting), you may permanently delete models via the `forceDestroy` method:

```php
Flight::forceDestroy(1);
```

> [!WARNING]
> The `destroy` method loads each model individually and calls the `delete` method so that the `deleting` and `deleted` events are properly dispatched for each model.

<a name="deleting-models-using-queries"></a>
#### Deleting Models Using Queries

Of course, you may build an Eloquent query to delete all models matching your query's criteria. In this example, we will delete all flights that are marked as inactive. Like mass updates, mass deletes will not dispatch model events for the models that are deleted:

```php
$deleted = Flight::where('active', 0)->delete();
```

To delete all models in a table, you should execute a query without adding any conditions:

```php
$deleted = Flight::query()->delete();
```

> [!WARNING]
> When executing a mass delete statement via Eloquent, the `deleting` and `deleted` model events will not be dispatched for the deleted models. This is because the models are never actually retrieved when executing the delete statement.

<a name="soft-deleting"></a>
### Soft Deleting

In addition to actually removing records from your database, Eloquent can also "soft delete" models. When models are soft deleted, they are not actually removed from your database. Instead, a `deleted_at` attribute is set on the model indicating the date and time at which the model was "deleted". To enable soft deletes for a model, add the `Illuminate\Database\Eloquent\SoftDeletes` trait to the model:

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

You should also add the `deleted_at` column to your database table. The Laravel [schema builder](/docs/{{version}}/migrations) contains a helper method to create this column:

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

Now, when you call the `delete` method on the model, the `deleted_at` column will be set to the current date and time. However, the model's database record will be left in the table. When querying a model that uses soft deletes, the soft deleted models will automatically be excluded from all query results.

To determine if a given model instance has been soft deleted, you may use the `trashed` method:

```php
if ($flight->trashed()) {
    // ...
}
```

<a name="restoring-soft-deleted-models"></a>
#### Restoring Soft Deleted Models

Sometimes you may wish to "un-delete" a soft deleted model. To restore a soft deleted model, you may call the `restore` method on a model instance. The `restore` method will set the model's `deleted_at` column to `null`:

```php
$flight->restore();
```

You may also use the `restore` method in a query to restore multiple models. Again, like other "mass" operations, this will not dispatch any model events for the models that are restored:

```php
Flight::withTrashed()
    ->where('airline_id', 1)
    ->restore();
```

The `restore` method may also be used when building [relationship](/docs/{{version}}/eloquent-relationships) queries:

```php
$flight->history()->restore();
```

<a name="permanently-deleting-models"></a>
#### Permanently Deleting Models

Sometimes you may need to truly remove a model from your database. You may use the `forceDelete` method to permanently remove a soft deleted model from the database table:

```php
$flight->forceDelete();
```

You may also use the `forceDelete` method when building Eloquent relationship queries:

```php
$flight->history()->forceDelete();
```

<a name="querying-soft-deleted-models"></a>
### Querying Soft Deleted Models

<a name="including-soft-deleted-models"></a>
#### Including Soft Deleted Models

As noted above, soft deleted models will automatically be excluded from query results. However, you may force soft deleted models to be included in a query's results by calling the `withTrashed` method on the query:

```php
use App\Models\Flight;

$flights = Flight::withTrashed()
    ->where('account_id', 1)
    ->get();
```

The `withTrashed` method may also be called when building a [relationship](/docs/{{version}}/eloquent-relationships) query:

```php
$flight->history()->withTrashed()->get();
```

<a name="retrieving-only-soft-deleted-models"></a>
#### Retrieving Only Soft Deleted Models

The `onlyTrashed` method will retrieve **only** soft deleted models:

```php
$flights = Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

<a name="pruning-models"></a>
## Pruning Models

Sometimes you may want to periodically delete models that are no longer needed. To accomplish this, you may add the `Illuminate\Database\Eloquent\Prunable` or `Illuminate\Database\Eloquent\MassPrunable` trait to the models you would like to periodically prune. After adding one of the traits to the model, implement a `prunable` method which returns an Eloquent query builder that resolves the models that are no longer needed:

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

When marking models as `Prunable`, you may also define a `pruning` method on the model. This method will be called before the model is deleted. This method can be useful for deleting any additional resources associated with the model, such as stored files, before the model is permanently removed from the database:

```php
/**
 * Prepare the model for pruning.
 */
protected function pruning(): void
{
    // ...
}
```

After configuring your prunable model, you should schedule the `model:prune` Artisan command in your application's `routes/console.php` file. You are free to choose the appropriate interval at which this command should be run:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('model:prune')->daily();
```

Behind the scenes, the `model:prune` command will automatically detect "Prunable" models within your application's `app/Models` directory. If your models are in a different location, you may use the `--model` option to specify the model class names:

```php
Schedule::command('model:prune', [
    '--model' => [Address::class, Flight::class],
])->daily();
```

If you wish to exclude certain models from being pruned while pruning all other detected models, you may use the `--except` option:

```php
Schedule::command('model:prune', [
    '--except' => [Address::class, Flight::class],
])->daily();
```

You may test your `prunable` query by executing the `model:prune` command with the `--pretend` option. When pretending, the `model:prune` command will simply report how many records would be pruned if the command were to actually run:

```shell
php artisan model:prune --pretend
```

> [!WARNING]
> Soft deleting models will be permanently deleted (`forceDelete`) if they match the prunable query.

<a name="mass-pruning"></a>
#### Mass Pruning

When models are marked with the `Illuminate\Database\Eloquent\MassPrunable` trait, models are deleted from the database using mass-deletion queries. Therefore, the `pruning` method will not be invoked, nor will the `deleting` and `deleted` model events be dispatched. This is because the models are never actually retrieved before deletion, thus making the pruning process much more efficient:

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

You may create an unsaved copy of an existing model instance using the `replicate` method. This method is particularly useful when you have model instances that share many of the same attributes:

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

To exclude one or more attributes from being replicated to the new model, you may pass an array to the `replicate` method:

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

Global scopes allow you to add constraints to all queries for a given model. Laravel's own [soft delete](#soft-deleting) functionality utilizes global scopes to only retrieve "non-deleted" models from the database. Writing your own global scopes can provide a convenient, easy way to make sure every query for a given model receives certain constraints.

<a name="generating-scopes"></a>
#### Generating Scopes

To generate a new global scope, you may invoke the `make:scope` Artisan command, which will place the generated scope in your application's `app/Models/Scopes` directory:

```shell
php artisan make:scope AncientScope
```

<a name="writing-global-scopes"></a>
#### Writing Global Scopes

Writing a global scope is simple. First, use the `make:scope` command to generate a class that implements the `Illuminate\Database\Eloquent\Scope` interface. The `Scope` interface requires you to implement one method: `apply`. The `apply` method may add `where` constraints or other types of clauses to the query as needed:

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
> If your global scope is adding columns to the select clause of the query, you should use the `addSelect` method instead of `select`. This will prevent the unintentional replacement of the query's existing select clause.

<a name="applying-global-scopes"></a>
#### Applying Global Scopes

To assign a global scope to a model, you may simply place the `ScopedBy` attribute on the model:

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

Or, you may manually register the global scope by overriding the model's `booted` method and invoke the model's `addGlobalScope` method. The `addGlobalScope` method accepts an instance of your scope as its only argument:

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

After adding the scope in the example above to the `App\Models\User` model, a call to the `User::all()` method will execute the following SQL query:

```sql
select * from `users` where `created_at` < 0021-02-18 00:00:00
```

<a name="anonymous-global-scopes"></a>
#### Anonymous Global Scopes

Eloquent also allows you to define global scopes using closures, which is particularly useful for simple scopes that do not warrant a separate class of their own. When defining a global scope using a closure, you should provide a scope name of your own choosing as the first argument to the `addGlobalScope` method:

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

If you would like to remove a global scope for a given query, you may use the `withoutGlobalScope` method. This method accepts the class name of the global scope as its only argument:

```php
User::withoutGlobalScope(AncientScope::class)->get();
```

Or, if you defined the global scope using a closure, you should pass the string name that you assigned to the global scope:

```php
User::withoutGlobalScope('ancient')->get();
```

If you would like to remove several or even all of the query's global scopes, you may use the `withoutGlobalScopes` and `withoutGlobalScopesExcept` methods:

```php
// Remove all of the global scopes...
User::withoutGlobalScopes()->get();

// Remove some of the global scopes...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();

// Remove all global scopes except the given ones...
User::withoutGlobalScopesExcept([
    SecondScope::class,
])->get();
```

<a name="local-scopes"></a>
### Local Scopes

Local scopes allow you to define common sets of query constraints that you may easily re-use throughout your application. For example, you may need to frequently retrieve all users that are considered "popular". To define a scope, add the `Scope` attribute to an Eloquent method.

Scopes should always return the same query builder instance or `void`:

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

Once the scope has been defined, you may call the scope methods when querying the model. You can even chain calls to various scopes:

```php
use App\Models\User;

$users = User::popular()->active()->orderBy('created_at')->get();
```

Combining multiple Eloquent model scopes via an `or` query operator may require the use of closures to achieve the correct [logical grouping](/docs/{{version}}/queries#logical-grouping):

```php
$users = User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

However, since this can be cumbersome, Laravel provides a "higher order" `orWhere` method that allows you to fluently chain scopes together without the use of closures:

```php
$users = User::popular()->orWhere->active()->get();
```

<a name="dynamic-scopes"></a>
#### Dynamic Scopes

Sometimes you may wish to define a scope that accepts parameters. To get started, just add your additional parameters to your scope method's signature. Scope parameters should be defined after the `$query` parameter:

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

Once the expected arguments have been added to your scope method's signature, you may pass the arguments when calling the scope:

```php
$users = User::ofType('admin')->get();
```

Attributed scope methods should be `protected`. When calling an attributed scope from within the model class, call the scope through a query builder instance, such as `static::query()->ofType('admin')`, to ensure the call is routed through Eloquent's scope handling.

<a name="pending-attributes"></a>
### Pending Attributes

If you would like to use scopes to create models that have the same attributes as those used to constrain the scope, you may use the `withAttributes` method when building the scope query:

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

The `withAttributes` method will add `where` conditions to the query using the given attributes, and it will also add the given attributes to any models created via the scope:

```php
$draft = Post::draft()->create(['title' => 'In Progress']);

$draft->hidden; // true
```

To instruct the `withAttributes` method to not add `where` conditions to the query, you may set the `asConditions` argument to `false`:

```php
$query->withAttributes([
    'hidden' => true,
], asConditions: false);
```

<a name="comparing-models"></a>
## Comparing Models

Sometimes you may need to determine if two models are the "same" or not. The `is` and `isNot` methods may be used to quickly verify two models have the same primary key, table, and database connection or not:

```php
if ($post->is($anotherPost)) {
    // ...
}

if ($post->isNot($anotherPost)) {
    // ...
}
```

The `is` and `isNot` methods are also available when using the `belongsTo`, `hasOne`, `morphTo`, and `morphOne` [relationships](/docs/{{version}}/eloquent-relationships). This method is particularly helpful when you would like to compare a related model without issuing a query to retrieve that model:

```php
if ($post->author()->is($user)) {
    // ...
}
```

<a name="events"></a>
## Events

> [!NOTE]
> Want to broadcast your Eloquent events directly to your client-side application? Check out Laravel's [model event broadcasting](/docs/{{version}}/broadcasting#model-broadcasting).

Eloquent models dispatch several events, allowing you to hook into the following moments in a model's lifecycle: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `trashed`, `forceDeleting`, `forceDeleted`, `restoring`, `restored`, and `replicating`.

The `retrieved` event will dispatch when an existing model is retrieved from the database. When a new model is saved for the first time, the `creating` and `created` events will dispatch. The `updating` / `updated` events will dispatch when an existing model is modified and the `save` method is called. The `saving` / `saved` events will dispatch when a model is created or updated - even if the model's attributes have not been changed. Event names ending with `-ing` are dispatched before any changes to the model are persisted, while events ending with `-ed` are dispatched after the changes to the model are persisted.

To start listening to model events, define a `$dispatchesEvents` property on your Eloquent model. This property maps various points of the Eloquent model's lifecycle to your own [event classes](/docs/{{version}}/events). Each model event class should expect to receive an instance of the affected model via its constructor:

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

After defining and mapping your Eloquent events, you may use [event listeners](/docs/{{version}}/events#defining-listeners) to handle the events.

> [!WARNING]
> When issuing a mass update or delete query via Eloquent, the `saved`, `updated`, `deleting`, and `deleted` model events will not be dispatched for the affected models. This is because the models are never actually retrieved when performing mass updates or deletes.

<a name="events-using-closures"></a>
### Using Closures

Instead of using custom event classes, you may register closures that execute when various model events are dispatched. Typically, you should register these closures in the `booted` method of your model:

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

If needed, you may utilize [queueable anonymous event listeners](/docs/{{version}}/events#queueable-anonymous-event-listeners) when registering model events. This will instruct Laravel to execute the model event listener in the background using your application's [queue](/docs/{{version}}/queues):

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

If you are listening for many events on a given model, you may use observers to group all of your listeners into a single class. Observer classes have method names which reflect the Eloquent events you wish to listen for. Each of these methods receives the affected model as their only argument. The `make:observer` Artisan command is the easiest way to create a new observer class:

```shell
php artisan make:observer UserObserver --model=User
```

This command will place the new observer in your `app/Observers` directory. If this directory does not exist, Artisan will create it for you. Your fresh observer will look like the following:

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

To register an observer, you may place the `ObservedBy` attribute on the corresponding model:

```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([UserObserver::class])]
class User extends Authenticatable
{
    //
}
```

Or, you may manually register an observer by invoking the `observe` method on the model you wish to observe. You may register observers in the `boot` method of your application's `AppServiceProvider` class:

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
> There are additional events an observer can listen to, such as `saving` and `retrieved`. These events are described within the [events](#events) documentation.

<a name="observers-and-database-transactions"></a>
#### Observers and Database Transactions

When models are being created within a database transaction, you may want to instruct an observer to only execute its event handlers after the database transaction is committed. You may accomplish this by implementing the `ShouldHandleEventsAfterCommit` interface on your observer. If a database transaction is not in progress, the event handlers will execute immediately:

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

You may occasionally need to temporarily "mute" all events fired by a model. You may achieve this using the `withoutEvents` method. The `withoutEvents` method accepts a closure as its only argument. Any code executed within this closure will not dispatch model events, and any value returned by the closure will be returned by the `withoutEvents` method:

```php
use App\Models\User;

$user = User::withoutEvents(function () {
    User::findOrFail(1)->delete();

    return User::find(2);
});
```

<a name="saving-a-single-model-without-events"></a>
#### Saving a Single Model Without Events

Sometimes you may wish to "save" a given model without dispatching any events. You may accomplish this using the `saveQuietly` method:

```php
$user = User::findOrFail(1);

$user->name = 'Victoria Faith';

$user->saveQuietly();
```

You may also "update", "delete", "soft delete", "restore", and "replicate" a given model without dispatching any events:

```php
$user->deleteQuietly();
$user->forceDeleteQuietly();
$user->restoreQuietly();
```

# Eloquent: Mutators & Casting

- [Giới thiệu](#introduction)
- [Accessors và Mutators](#accessors-and-mutators)
    - [Định nghĩa một Accessor](#defining-an-accessor)
    - [Định nghĩa một Mutator](#defining-a-mutator)
- [Attribute Casting](#attribute-casting)
    - [Array và JSON Casting](#array-and-json-casting)
    - [Binary Casting](#binary-casting)
    - [Date Casting](#date-casting)
    - [Enum Casting](#enum-casting)
    - [Encrypted Casting](#encrypted-casting)
    - [Query Time Casting](#query-time-casting)
- [Custom Casts](#custom-casts)
    - [Value Object Casting](#value-object-casting)
    - [Array / JSON Serialization](#array-json-serialization)
    - [Inbound Casting](#inbound-casting)
    - [Cast Parameters](#cast-parameters)
    - [Comparing Cast Values](#comparing-cast-values)
    - [Castables](#castables)

<a name="introduction"></a>
## Giới thiệu

Accessors, mutators, và attribute casting cho phép bạn transform Eloquent attribute values khi bạn retrieve hoặc set chúng trên model instances. Ví dụ, bạn có thể muốn sử dụng [Laravel encrypter](/docs/{{version}}/encryption) để encrypt một value trong khi nó được stored trong database, và sau đó automatically decrypt attribute khi bạn access nó trên một Eloquent model. Hoặc, bạn có thể muốn convert một JSON string được stored trong database của bạn thành một array khi nó được accessed qua Eloquent model của bạn.

<a name="accessors-and-mutators"></a>
## Accessors và Mutators

<a name="defining-an-accessor"></a>
### Định nghĩa một Accessor

Một accessor transforms một Eloquent attribute value khi nó được accessed. Để define một accessor, tạo một protected method trên model của bạn để represent accessible attribute. Method name này nên tương ứng với "camel case" representation của true underlying model attribute / database column khi applicable.

Trong ví dụ này, chúng ta sẽ define một accessor cho `first_name` attribute. Accessor sẽ automatically được gọi bởi Eloquent khi attempting để retrieve value của `first_name` attribute. Tất cả attribute accessor / mutator methods phải declare một return type-hint của `Illuminate\Database\Eloquent\Casts\Attribute`:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the user's first name.
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
        );
    }
}
```

Tất cả accessor methods return một `Attribute` instance mà define cách attribute sẽ được accessed và, optionally, mutated. Trong ví dụ này, chúng ta chỉ defining cách attribute sẽ được accessed. Để làm như vậy, chúng ta supply `get` argument vào `Attribute` class constructor.

Như bạn có thể thấy, original value của column được passed vào accessor, cho phép bạn manipulate và return value. Để access value của accessor, bạn có thể đơn giản access `first_name` attribute trên một model instance:

```php
use App\Models\User;

$user = User::find(1);

$firstName = $user->first_name;
```

> [!NOTE]
> Nếu bạn muốn những computed values này được thêm vào array / JSON representations của model của bạn, [bạn sẽ cần append chúng](/docs/{{version}}/eloquent-serialization#appending-values-to-json).

<a name="building-value-objects-from-multiple-attributes"></a>
#### Building Value Objects Từ Multiple Attributes

Đôi khi accessor của bạn có thể cần transform multiple model attributes thành một single "value object". Để làm như vậy, `get` closure của bạn có thể accept một second argument của `$attributes`, mà sẽ automatically được supplied vào closure và sẽ chứa một array của tất cả model's current attributes:

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Interact with the user's address.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

<a name="accessor-caching"></a>
#### Accessor Caching

Khi returning value objects từ accessors, bất kỳ changes nào được thực hiện trên value object sẽ automatically được synced back vào model trước khi model được saved. Điều này có thể thực hiện được vì Eloquent retains instances returned bởi accessors để nó có thể return cùng instance mỗi lần accessor được invoked:

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Updated Address Line 1 Value';
$user->address->lineTwo = 'Updated Address Line 2 Value';

$user->save();
```

Tuy nhiên, bạn có thể đôi khi muốn enable caching cho primitive values như strings và booleans, đặc biệt nếu chúng là computationally intensive. Để accomplish điều này, bạn có thể invoke `shouldCache` method khi defining accessor của bạn:

```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```

Nếu bạn muốn disable object caching behavior của attributes, bạn có thể invoke `withoutObjectCaching` method khi defining attribute:

```php
/**
 * Interact with the user's address.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    )->withoutObjectCaching();
}
```

<a name="defining-a-mutator"></a>
### Định nghĩa một Mutator

Một mutator transforms một Eloquent attribute value khi nó được set. Để define một mutator, bạn có thể provide `set` argument khi defining attribute của bạn. Hãy define một mutator cho `first_name` attribute. Mutator này sẽ automatically được gọi khi chúng ta attempt để set value của `first_name` attribute trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Interact with the user's first name.
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value),
        );
    }
}
```

Mutator closure sẽ nhận value đang được set trên attribute, cho phép bạn manipulate value và return manipulated value. Để sử dụng mutator của chúng ta, chúng ta chỉ cần set `first_name` attribute trên một Eloquent model:

```php
use App\Models\User;

$user = User::find(1);

$user->first_name = 'Sally';
```

Trong ví dụ này, `set` callback sẽ được gọi với value `Sally`. Mutator sẽ sau đó apply `strtolower` function vào name và set resulting value của nó trong model's internal `$attributes` array.

<a name="mutating-multiple-attributes"></a>
#### Mutating Multiple Attributes

Đôi khi mutator của bạn có thể cần set multiple attributes trên underlying model. Để làm như vậy, bạn có thể return một array từ `set` closure. Mỗi key trong array nên tương ứng với một underlying attribute / database column liên quan với model:

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Interact with the user's address.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

<a name="attribute-casting"></a>
## Attribute Casting

Attribute casting cung cấp functionality tương tự như accessors và mutators mà không requiring bạn define bất kỳ additional methods nào trên model của bạn. Thay vào đó, model's `casts` method cung cấp một convenient way để convert attributes thành common data types.

`casts` method nên return một array trong đó key là tên của attribute đang được cast và value là type bạn muốn cast column thành. Supported cast types là:

<div class="content-list" markdown="1">

- `array`
- `AsFluent::class`
- `AsStringable::class`
- `AsUri::class`
- `boolean`
- `collection`
- `date`
- `datetime`
- `immutable_date`
- `immutable_datetime`
- <code>decimal:&lt;precision&gt;</code>
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
- `hashed`
- `integer`
- `object`
- `real`
- `string`
- `timestamp`

</div>

Để demonstrate attribute casting, hãy cast `is_admin` attribute, được stored trong database của chúng ta như một integer (`0` hoặc `1`) thành một boolean value:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'is_admin' => 'boolean',
        ];
    }
}
```

Sau khi defining cast, `is_admin` attribute sẽ luôn được cast thành một boolean khi bạn access nó, ngay cả khi underlying value được stored trong database như một integer:

```php
$user = App\Models\User::find(1);

if ($user->is_admin) {
    // ...
}
```

Nếu bạn cần add một new, temporary cast tại runtime, bạn có thể sử dụng `mergeCasts` method. Những cast definitions này sẽ được added vào bất kỳ casts nào đã được defined trên model:

```php
$user->mergeCasts([
    'is_admin' => 'integer',
    'options' => 'object',
]);
```

> [!WARNING]
> Attributes là `null` sẽ không được cast. Ngoài ra, bạn không bao giờ nên define một cast (hoặc một attribute) có cùng tên với một relationship hoặc assign một cast vào model's primary key.

<a name="stringable-casting"></a>
#### Stringable Casting

Bạn có thể sử dụng `Illuminate\Database\Eloquent\Casts\AsStringable` cast class để cast một model attribute thành một [fluent Illuminate\Support\Stringable object](/docs/{{version}}/strings#fluent-strings-method-list):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\AsStringable;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'directory' => AsStringable::class,
        ];
    }
}
```

<a name="array-and-json-casting"></a>
### Array và JSON Casting

`array` cast đặc biệt hữu ích khi working với columns được stored như serialized JSON. Ví dụ, nếu database của bạn có một `JSON` hoặc `TEXT` field type chứa serialized JSON, thêm `array` cast vào attribute đó sẽ automatically deserialize attribute thành một PHP array khi bạn access nó trên Eloquent model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => 'array',
        ];
    }
}
```

Khi cast đã được defined, bạn có thể access `options` attribute và nó sẽ automatically được deserialize từ JSON thành một PHP array. Khi bạn set value của `options` attribute, given array sẽ automatically được serialize lại thành JSON cho storage:

```php
use App\Models\User;

$user = User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;

$user->save();
```

Để update một single field của một JSON attribute với một syntax ngắn gọn hơn, bạn có thể [make attribute mass assignable](/docs/{{version}}/eloquent#mass-assignment-json-columns) và sử dụng `->` operator khi calling `update` method:

```php
$user = User::find(1);

$user->update(['options->key' => 'value']);
```

<a name="json-and-unicode"></a>
#### JSON và Unicode

Nếu bạn muốn store một array attribute như JSON với unescaped Unicode characters, bạn có thể sử dụng `json:unicode` cast:

```php
/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => 'json:unicode',
    ];
}
```

<a name="array-object-and-collection-casting"></a>
#### Array Object và Collection Casting

Mặc dù standard `array` cast là sufficient cho nhiều applications, nó có một số disadvantages. Vì `array` cast trả về một primitive type, không thể mutate một offset của array trực tiếp. Ví dụ, code sau sẽ trigger một PHP error:

```php
$user = User::find(1);

$user->options['key'] = $value;
```

Để giải quyết điều này, Laravel cung cấp một `AsArrayObject` cast mà cast JSON attribute của bạn thành một [ArrayObject](https://www.php.net/manual/en/class.arrayobject.php) class. Feature này được implemented sử dụng Laravel's [custom cast](#custom-casts) implementation, cho phép Laravel intelligently cache và transform mutated object sao cho individual offsets có thể được modified mà không triggering một PHP error. Để sử dụng `AsArrayObject` cast, đơn giản assign nó vào một attribute:

```php
use Illuminate\Database\Eloquent\Casts\AsArrayObject;

/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsArrayObject::class,
    ];
}
```

Tương tự, Laravel cung cấp một `AsCollection` cast mà cast JSON attribute của bạn thành một Laravel [Collection](/docs/{{version}}/collections) instance:

```php
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::class,
    ];
}
```

Nếu bạn muốn `AsCollection` cast để instantiate một custom collection class thay vì Laravel's base collection class, bạn có thể provide collection class name như một cast argument:

```php
use App\Collections\OptionCollection;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::using(OptionCollection::class),
    ];
}
```

`of` method có thể được sử dụng để indicate collection items nên được mapped vào một given class qua collection's [mapInto method](/docs/{{version}}/collections#method-mapinto):

```php
use App\ValueObjects\Option;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::of(Option::class)
    ];
}
```

Khi mapping collections thành objects, object nên implement `Illuminate\Contracts\Support\Arrayable` và `JsonSerializable` interfaces để define cách instances của họ nên được serialized vào database như JSON:

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Support\Arrayable;
use JsonSerializable;

class Option implements Arrayable, JsonSerializable
{
    public string $name;
    public mixed $value;
    public bool $isLocked;

    /**
     * Create a new Option instance.
     */
    public function __construct(array $data)
    {
        $this->name = $data['name'];
        $this->value = $data['value'];
        $this->isLocked = $data['is_locked'];
    }

    /**
     * Get the instance as an array.
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'value' => $this->value,
            'is_locked' => $this->isLocked,
        ];
    }

    /**
     * Specify the data which should be serialized to JSON.
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function jsonSerialize(): array
    {
        return $this->toArray();
    }
}
```

<a name="binary-casting"></a>
### Binary Casting

Nếu Eloquent model của bạn có một [binary type](/docs/{{version}}/migrations#column-method-binary) `uuid` hoặc `ulid` column ngoài model's auto-incrementing ID column, bạn có thể sử dụng `AsBinary` cast để automatically cast value thành và từ binary representation của nó:

```php
use Illuminate\Database\Eloquent\Casts\AsBinary;

/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'uuid' => AsBinary::uuid(),
        'ulid' => AsBinary::ulid(),
    ];
}
```

Khi cast đã được defined trên model, bạn có thể set UUID / ULID attribute value thành một object instance hoặc một string. Eloquent sẽ automatically cast value thành binary representation của nó. Khi retrieving attribute's value, bạn sẽ luôn nhận được một plain-text string value:

```php
use Illuminate\Support\Str;

$user->uuid = Str::uuid();

return $user->uuid;

// "6e8cdeed-2f32-40bd-b109-1e4405be2140"
```

<a name="date-casting"></a>
### Date Casting

Theo mặc định, Eloquent sẽ cast `created_at` và `updated_at` columns thành instances của [Carbon](https://github.com/briannesbitt/Carbon), mà extends PHP `DateTime` class và cung cấp một assortment của helpful methods. Bạn có thể cast additional date attributes bằng cách defining additional date casts trong model's `casts` method. Thông thường, dates nên được cast sử dụng `datetime` hoặc `immutable_datetime` cast types.

Khi defining một `date` hoặc `datetime` cast, bạn cũng có thể specify date's format. Format này sẽ được sử dụng khi [model được serialized thành một array hoặc JSON](/docs/{{version}}/eloquent-serialization):

```php
/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'created_at' => 'datetime:Y-m-d',
    ];
}
```

Khi một column được cast như một date, bạn có thể set corresponding model attribute value thành một UNIX timestamp, date string (`Y-m-d`), date-time string, hoặc một `DateTime` / `Carbon` instance. Date's value sẽ được correctly converted và stored trong database của bạn.

Bạn có thể customize default serialization format cho tất cả model's dates của bạn bằng cách defining một `serializeDate` method trên model của bạn. Method này không affect cách dates của bạn được formatted cho storage trong database:

```php
/**
 * Prepare a date for array / JSON serialization.
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

Để specify format nên được sử dụng khi actually storing model's dates trong database của bạn, bạn nên sử dụng `dateFormat` argument trên model's `Table` attribute:

```php
use Illuminate\Database\Eloquent\Attributes\Table;

#[Table(dateFormat: 'U')]
class Flight extends Model
{
    // ...
}
```

<a name="date-casting-and-timezones"></a>
#### Date Casting, Serialization, và Timezones

Theo mặc định, `date` và `datetime` casts sẽ serialize dates thành một UTC ISO-8601 date string (`YYYY-MM-DDTHH:MM:SS.uuuuuuZ`), bất kể timezone được specified trong application's `timezone` configuration option. Bạn được strongly encouraged để luôn sử dụng serialization format này, cũng như store application's dates của bạn trong UTC timezone bằng cách không changing application's `timezone` configuration option từ default `UTC` value của nó. Consistently sử dụng UTC timezone trong toàn bộ application của bạn sẽ cung cấp maximum level của interoperability với các date manipulation libraries khác được viết trong PHP và JavaScript.

Nếu một custom format được applied vào `date` hoặc `datetime` cast, như `datetime:Y-m-d H:i:s`, inner timezone của Carbon instance sẽ được sử dụng trong date serialization. Thông thường, đây sẽ là timezone được specified trong application's `timezone` configuration option. Tuy nhiên, điều quan trọng cần note là `timestamp` columns như `created_at` và `updated_at` được exempt từ behavior này và luôn được formatted trong UTC, bất kể application's timezone setting.

<a name="enum-casting"></a>
### Enum Casting

Eloquent cũng cho phép bạn cast attribute values của bạn thành PHP [Enums](https://www.php.net/manual/en/language.enumerations.backed.php). Để accomplish điều này, bạn có thể specify attribute và enum bạn muốn cast trong model's `casts` method:

```php
use App\Enums\ServerStatus;

/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'status' => ServerStatus::class,
    ];
}
```

Khi bạn đã defined cast trên model của bạn, specified attribute sẽ automatically được cast thành và từ một enum khi bạn interact với attribute:

```php
if ($server->status == ServerStatus::Provisioned) {
    $server->status = ServerStatus::Ready;

    $server->save();
}
```

<a name="casting-arrays-of-enums"></a>
#### Casting Arrays của Enums

Đôi khi bạn có thể cần model của bạn để store một array của enum values trong một single column. Để accomplish điều này, bạn có thể utilize `AsEnumArrayObject` hoặc `AsEnumCollection` casts được cung cấp bởi Laravel:

```php
use App\Enums\ServerStatus;
use Illuminate\Database\Eloquent\Casts\AsEnumCollection;

/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'statuses' => AsEnumCollection::of(ServerStatus::class),
    ];
}
```

<a name="encrypted-casting"></a>
### Encrypted Casting

`encrypted` cast sẽ encrypt một model's attribute value sử dụng Laravel's built-in [encryption](/docs/{{version}}/encryption) features. Ngoài ra, `encrypted:array`, `encrypted:collection`, `encrypted:object`, `AsEncryptedArrayObject`, và `AsEncryptedCollection` casts hoạt động như unencrypted counterparts của họ; tuy nhiên, như bạn có thể expect, underlying value được encrypted khi stored trong database của bạn.

Vì final length của encrypted text không predictable và dài hơn plain text counterpart của nó, hãy chắc chắn associated database column là của `TEXT` type hoặc lớn hơn. Ngoài ra, vì values được encrypted trong database, bạn sẽ không thể query hoặc search encrypted attribute values.

<a name="key-rotation"></a>
#### Key Rotation

Như bạn có thể biết, Laravel encrypts strings sử dụng `key` configuration value được specified trong application's `app` configuration file. Thông thường, value này tương ứng với value của `APP_KEY` environment variable. Nếu bạn cần rotate application's encryption key của bạn, bạn có thể [gracefully làm như vậy](/docs/{{version}}/encryption#gracefully-rotating-encryption-keys).

<a name="query-time-casting"></a>
### Query Time Casting

Đôi khi bạn có thể cần apply casts trong khi executing một query, như khi selecting một raw value từ một table. Ví dụ, consider query sau:

```php
use App\Models\Post;
use App\Models\User;

$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
        ->whereColumn('user_id', 'users.id')
])->get();
```

`last_posted_at` attribute trên results của query này sẽ là một simple string. Nó sẽ tuyệt vời nếu chúng ta có thể apply một `datetime` cast vào attribute này khi executing query. May mắn thay, chúng ta có thể accomplish điều này sử dụng `withCasts` method:

```php
$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
        ->whereColumn('user_id', 'users.id')
])->withCasts([
    'last_posted_at' => 'datetime'
])->get();
```

<a name="custom-casts"></a>
## Custom Casts

Laravel có một variety của built-in, helpful cast types; tuy nhiên, bạn có thể đôi khi cần define custom cast types của riêng bạn. Để tạo một cast, execute `make:cast` Artisan command. New cast class sẽ được đặt trong `app/Casts` directory của bạn:

```shell
php artisan make:cast AsJson
```

Tất cả custom cast classes implement `CastsAttributes` interface. Classes implement interface này phải define một `get` và `set` method. `get` method chịu trách nhiệm transforming một raw value từ database thành một cast value, trong khi `set` method nên transform một cast value thành một raw value có thể được stored trong database. Ví dụ, chúng ta sẽ re-implement built-in `json` cast type như một custom cast type:

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;

class AsJson implements CastsAttributes
{
    /**
     * Cast the given value.
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, mixed>
     */
    public function get(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): array {
        return json_decode($value, true);
    }

    /**
     * Prepare the given value for storage.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): string {
        return json_encode($value);
    }
}
```

Khi bạn đã defined một custom cast type, bạn có thể attach nó vào một model attribute sử dụng class name của nó:

```php
<?php

namespace App\Models;

use App\Casts\AsJson;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the attributes that should be cast.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsJson::class,
        ];
    }
}
```

<a name="value-object-casting"></a>
### Value Object Casting

Bạn không bị giới hạn casting values thành primitive types. Bạn cũng có thể cast values thành objects. Defining custom casts mà cast values thành objects rất tương tự như casting thành primitive types; tuy nhiên, nếu value object của bạn bao gồm nhiều hơn một database column, `set` method phải return một array của key / value pairs sẽ được sử dụng để set raw, storable values trên model. Nếu value object của bạn chỉ affects một single column, bạn nên đơn giản return storable value.

Ví dụ, chúng ta sẽ define một custom cast class mà casts multiple model values thành một single `Address` value object. Chúng ta sẽ assume `Address` value object có hai public properties: `lineOne` và `lineTwo`:

```php
<?php

namespace App\Casts;

use App\ValueObjects\Address;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;
use InvalidArgumentException;

class AsAddress implements CastsAttributes
{
    /**
     * Cast the given value.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function get(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): Address {
        return new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two']
        );
    }

    /**
     * Prepare the given value for storage.
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, string>
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): array {
        if (! $value instanceof Address) {
            throw new InvalidArgumentException('The given value is not an Address instance.');
        }

        return [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ];
    }
}
```

Khi casting thành value objects, bất kỳ changes nào được thực hiện trên value object sẽ automatically được synced back vào model trước khi model được saved:

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Updated Address Value';

$user->save();
```

> [!NOTE]
> Nếu bạn plan để serialize Eloquent models của bạn chứa value objects thành JSON hoặc arrays, bạn nên implement `Illuminate\Contracts\Support\Arrayable` và `JsonSerializable` interfaces trên value object.

<a name="value-object-caching"></a>
#### Value Object Caching

Khi attributes được cast thành value objects được resolved, chúng được cached bởi Eloquent. Do đó, cùng object instance sẽ được returned nếu attribute được accessed lại.

Nếu bạn muốn disable object caching behavior của custom cast classes, bạn có thể declare một public `withoutObjectCaching` property trên custom cast class của bạn:

```php
class AsAddress implements CastsAttributes
{
    public bool $withoutObjectCaching = true;

    // ...
}
```

<a name="array-json-serialization"></a>
### Array / JSON Serialization

Khi một Eloquent model được convert thành một array hoặc JSON sử dụng `toArray` và `toJson` methods, custom cast value objects của bạn sẽ thường được serialized miễn là chúng implement `Illuminate\Contracts\Support\Arrayable` và `JsonSerializable` interfaces. Tuy nhiên, khi sử dụng value objects được cung cấp bởi third-party libraries, bạn có thể không có khả năng để add những interfaces này vào object.

Do đó, bạn có thể specify rằng custom cast class của bạn sẽ chịu trách nhiệm serializing value object. Để làm như vậy, custom cast class của bạn nên implement `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes` interface. Interface này states rằng class của bạn nên chứa một `serialize` method mà nên return serialized form của value object của bạn:

```php
/**
 * Get the serialized representation of the value.
 *
 * @param  array<string, mixed>  $attributes
 */
public function serialize(
    Model $model,
    string $key,
    mixed $value,
    array $attributes,
): string {
    return (string) $value;
}
```

<a name="inbound-casting"></a>
### Inbound Casting

Đôi khi, bạn có thể cần viết một custom cast class chỉ transforms values đang được set trên model và không thực hiện bất kỳ operations nào khi attributes đang được retrieved từ model.

Inbound only custom casts nên implement `CastsInboundAttributes` interface, mà chỉ requires một `set` method để được defined. `make:cast` Artisan command có thể được invoked với `--inbound` option để generate một inbound only cast class:

```shell
php artisan make:cast AsHash --inbound
```

Một classic example của một inbound only cast là một "hashing" cast. Ví dụ, chúng ta có thể define một cast mà hashes inbound values qua một given algorithm:

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;
use Illuminate\Database\Eloquent\Model;

class AsHash implements CastsInboundAttributes
{
    /**
     * Create a new cast class instance.
     */
    public function __construct(
        protected string|null $algorithm = null,
    ) {}

    /**
     * Prepare the given value for storage.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): string {
        return is_null($this->algorithm)
            ? bcrypt($value)
            : hash($this->algorithm, $value);
    }
}
```

<a name="cast-parameters"></a>
### Cast Parameters

Khi attaching một custom cast vào một model, cast parameters có thể được specified bằng cách separating chúng từ class name sử dụng một `:` character và comma-delimiting multiple parameters. Parameters sẽ được passed vào constructor của cast class:

```php
/**
 * Get the attributes that should be cast.
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'secret' => AsHash::class.':sha256',
    ];
}
```

<a name="comparing-cast-values"></a>
### Comparing Cast Values

Nếu bạn muốn define cách hai given cast values nên được compared để determine xem chúng đã được changed hay chưa, custom cast class của bạn có thể implement `Illuminate\Contracts\Database\Eloquent\ComparesCastableAttributes` interface. Điều này cho phép bạn có fine-grained control over những values Eloquent considers changed và do đó saves vào database khi một model được updated.

Interface này states rằng class của bạn nên chứa một `compare` method mà nên return `true` nếu given values được considered equal:

```php
/**
 * Determine if the given values are equal.
 *
 * @param  \Illuminate\Database\Eloquent\Model  $model
 * @param  string  $key
 * @param  mixed  $firstValue
 * @param  mixed  $secondValue
 * @return bool
 */
public function compare(
    Model $model,
    string $key,
    mixed $firstValue,
    mixed $secondValue
): bool {
    return $firstValue === $secondValue;
}
```

<a name="castables"></a>
### Castables

Bạn có thể muốn cho phép application's value objects của bạn để define custom cast classes của riêng họ. Thay vì attaching custom cast class vào model của bạn, bạn có thể alternatively attach một value object class mà implements `Illuminate\Contracts\Database\Eloquent\Castable` interface:

```php
use App\ValueObjects\Address;

protected function casts(): array
{
    return [
        'address' => Address::class,
    ];
}
```

Objects implement `Castable` interface phải define một `castUsing` method mà trả về class name của custom caster class chịu trách nhiệm casting thành và từ `Castable` class:

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Database\Eloquent\Castable;
use App\Casts\AsAddress;

class Address implements Castable
{
    /**
     * Get the name of the caster class to use when casting from / to this cast target.
     *
     * @param  array<string, mixed>  $arguments
     */
    public static function castUsing(array $arguments): string
    {
        return AsAddress::class;
    }
}
```

Khi sử dụng `Castable` classes, bạn vẫn có thể provide arguments trong `casts` method definition. Arguments sẽ được passed vào `castUsing` method:

```php
use App\ValueObjects\Address;

protected function casts(): array
{
    return [
        'address' => Address::class.':argument',
    ];
}
```

<a name="anonymous-cast-classes"></a>
#### Castables & Anonymous Cast Classes

Bằng cách combining "castables" với PHP's [anonymous classes](https://www.php.net/manual/en/language.oop5.anonymous.php), bạn có thể define một value object và casting logic của nó như một single castable object. Để accomplish điều này, return một anonymous class từ value object's `castUsing` method. Anonymous class nên implement `CastsAttributes` interface:

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Database\Eloquent\Castable;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

class Address implements Castable
{
    // ...

    /**
     * Get the caster class to use when casting from / to this cast target.
     *
     * @param  array<string, mixed>  $arguments
     */
    public static function castUsing(array $arguments): CastsAttributes
    {
        return new class implements CastsAttributes
        {
            public function get(
                Model $model,
                string $key,
                mixed $value,
                array $attributes,
            ): Address {
                return new Address(
                    $attributes['address_line_one'],
                    $attributes['address_line_two']
                );
            }

            public function set(
                Model $model,
                string $key,
                mixed $value,
                array $attributes,
            ): array {
                return [
                    'address_line_one' => $value->lineOne,
                    'address_line_two' => $value->lineTwo,
                ];
            }
        };
    }
}
```

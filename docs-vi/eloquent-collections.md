# Eloquent: Collections

- [Giới thiệu](#introduction)
- [Available Methods](#available-methods)
- [Custom Collections](#custom-collections)

<a name="introduction"></a>
## Giới thiệu

Tất cả Eloquent methods trả về nhiều hơn một model result sẽ trả về instances của class `Illuminate\Database\Eloquent\Collection`, bao gồm cả results được lấy qua method `get` hoặc được truy cập qua một relationship. Eloquent collection object extends Laravel's [base collection](/docs/{{version}}/collections), nên nó tự nhiên kế thừa hàng chục methods được sử dụng để làm việc một cách mượt mà với underlying array của Eloquent models. Hãy chắc chắn để review Laravel collection documentation để tìm hiểu tất cả về những helpful methods này!

Tất cả collections cũng đóng vai trò là iterators, cho phép bạn loop qua chúng như thể chúng là simple PHP arrays:

```php
use App\Models\User;

$users = User::where('active', 1)->get();

foreach ($users as $user) {
    echo $user->name;
}
```

Tuy nhiên, như đã đề cập trước đó, collections mạnh hơn arrays nhiều và expose một variety của map / reduce operations có thể được chained sử dụng một intuitive interface. Ví dụ, chúng ta có thể remove tất cả inactive models và sau đó gather first name cho mỗi remaining user:

```php
$names = User::all()->reject(function (User $user) {
    return $user->active === false;
})->map(function (User $user) {
    return $user->name;
});
```

<a name="eloquent-collection-conversion"></a>
#### Eloquent Collection Conversion

Trong khi hầu hết Eloquent collection methods trả về một new instance của một Eloquent collection, các methods `collapse`, `flatten`, `flip`, `keys`, `pluck`, và `zip` trả về một [base collection](/docs/{{version}}/collections) instance. Tương tự, nếu một `map` operation trả về một collection không chứa bất kỳ Eloquent models nào, nó sẽ được convert thành một base collection instance.

<a name="available-methods"></a>
## Available Methods

Tất cả Eloquent collections extend base [Laravel collection](/docs/{{version}}/collections#available-methods) object; do đó, chúng kế thừa tất cả powerful methods được cung cấp bởi base collection class.

Ngoài ra, class `Illuminate\Database\Eloquent\Collection` cung cấp một superset của methods để hỗ trợ việc quản lý model collections của bạn. Hầu hết methods trả về `Illuminate\Database\Eloquent\Collection` instances; tuy nhiên, một số methods, như `modelKeys`, trả về một `Illuminate\Support\Collection` instance.

<style>
    .collection-method-list > p {
        columns: 14.4em 1; -moz-columns: 14.4em 1; -webkit-columns: 14.4em 1;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<div class="collection-method-list" markdown="1">

[append](#method-append)
[contains](#method-contains)
[diff](#method-diff)
[except](#method-except)
[find](#method-find)
[findOrFail](#method-find-or-fail)
[fresh](#method-fresh)
[intersect](#method-intersect)
[load](#method-load)
[loadMissing](#method-loadMissing)
[modelKeys](#method-modelKeys)
[makeVisible](#method-makeVisible)
[makeHidden](#method-makeHidden)
[mergeVisible](#method-mergeVisible)
[mergeHidden](#method-mergeHidden)
[only](#method-only)
[partition](#method-partition)
[setAppends](#method-setAppends)
[setVisible](#method-setVisible)
[setHidden](#method-setHidden)
[toQuery](#method-toquery)
[unique](#method-unique)
[withoutAppends](#method-withoutAppends)

</div>

<a name="method-append"></a>
#### `append($attributes)` {.collection-method .first-collection-method}

Method `append` có thể được sử dụng để indicate rằng một attribute nên được [appended](/docs/{{version}}/eloquent-serialization#appending-values-to-json) cho mỗi model trong collection. Method này chấp nhận một array của attributes hoặc một single attribute:

```php
$users->append('team');

$users->append(['team', 'is_admin']);
```

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)` {.collection-method}

Method `contains` có thể được sử dụng để determine xem một given model instance có được chứa bởi collection hay không. Method này chấp nhận một primary key hoặc một model instance:

```php
$users->contains(1);

$users->contains(User::find(1));
```

<a name="method-diff"></a>
#### `diff($items)` {.collection-method}

Method `diff` trả về tất cả models không có trong given collection:

```php
use App\Models\User;

$users = $users->diff(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-except"></a>
#### `except($keys)` {.collection-method}

Method `except` trả về tất cả models không có given primary keys:

```php
$users = $users->except([1, 2, 3]);
```

<a name="method-find"></a>
#### `find($key)` {.collection-method}

Method `find` trả về model có primary key matching với given key. Nếu `$key` là một model instance, `find` sẽ attempt để return một model matching với primary key. Nếu `$key` là một array của keys, `find` sẽ return tất cả models có primary key trong given array:

```php
$users = User::all();

$user = $users->find(1);
```

<a name="method-find-or-fail"></a>
#### `findOrFail($key)` {.collection-method}

Method `findOrFail` trả về model có primary key matching với given key hoặc throws một `Illuminate\Database\Eloquent\ModelNotFoundException` exception nếu không có matching model nào được tìm thấy trong collection:

```php
$users = User::all();

$user = $users->findOrFail(1);
```

<a name="method-fresh"></a>
#### `fresh($with = [])` {.collection-method}

Method `fresh` retrieves một fresh instance của mỗi model trong collection từ database. Ngoài ra, bất kỳ specified relationships nào sẽ được eager loaded:

```php
$users = $users->fresh();

$users = $users->fresh('comments');
```

<a name="method-intersect"></a>
#### `intersect($items)` {.collection-method}

Method `intersect` trả về tất cả models cũng có trong given collection:

```php
use App\Models\User;

$users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-load"></a>
#### `load($relations)` {.collection-method}

Method `load` eager loads given relationships cho tất cả models trong collection:

```php
$users->load(['comments', 'posts']);

$users->load('comments.author');

$users->load(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

<a name="method-loadMissing"></a>
#### `loadMissing($relations)` {.collection-method}

Method `loadMissing` eager loads given relationships cho tất cả models trong collection nếu relationships chưa được loaded:

```php
$users->loadMissing(['comments', 'posts']);

$users->loadMissing('comments.author');

$users->loadMissing(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

<a name="method-modelKeys"></a>
#### `modelKeys()` {.collection-method}

Method `modelKeys` trả về primary keys cho tất cả models trong collection:

```php
$users->modelKeys();

// [1, 2, 3, 4, 5]
```

<a name="method-makeVisible"></a>
#### `makeVisible($attributes)` {.collection-method}

Method `makeVisible` [makes attributes visible](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) thường là "hidden" trên mỗi model trong collection:

```php
$users = $users->makeVisible(['address', 'phone_number']);
```

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)` {.collection-method}

Method `makeHidden` [hides attributes](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) thường là "visible" trên mỗi model trong collection:

```php
$users = $users->makeHidden(['address', 'phone_number']);
```

<a name="method-mergeVisible"></a>
#### `mergeVisible($attributes)` {.collection-method}

Method `mergeVisible` [makes additional attributes visible](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) trong khi retaining existing visible attributes:

```php
$users = $users->mergeVisible(['middle_name']);
```

<a name="method-mergeHidden"></a>
#### `mergeHidden($attributes)` {.collection-method}

Method `mergeHidden` [hides additional attributes](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json) trong khi retaining existing hidden attributes:

```php
$users = $users->mergeHidden(['last_login_at']);
```

<a name="method-only"></a>
#### `only($keys)` {.collection-method}

Method `only` trả về tất cả models có given primary keys:

```php
$users = $users->only([1, 2, 3]);
```

<a name="method-partition"></a>
#### `partition` {.collection-method}

Method `partition` trả về một instance của `Illuminate\Support\Collection` chứa `Illuminate\Database\Eloquent\Collection` collection instances:

```php
$partition = $users->partition(fn ($user) => $user->age > 18);

dump($partition::class);    // Illuminate\Support\Collection
dump($partition[0]::class); // Illuminate\Database\Eloquent\Collection
dump($partition[1]::class); // Illuminate\Database\Eloquent\Collection
```

<a name="method-setAppends"></a>
#### `setAppends($attributes)` {.collection-method}

Method `setAppends` tạm thời overrides tất cả [appended attributes](/docs/{{version}}/eloquent-serialization#appending-values-to-json) trên mỗi model trong collection:

```php
$users = $users->setAppends(['is_admin']);
```

<a name="method-setVisible"></a>
#### `setVisible($attributes)` {.collection-method}

Method `setVisible` [tạm thời overrides](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) tất cả visible attributes trên mỗi model trong collection:

```php
$users = $users->setVisible(['id', 'name']);
```

<a name="method-setHidden"></a>
#### `setHidden($attributes)` {.collection-method}

Method `setHidden` [tạm thời overrides](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) tất cả hidden attributes trên mỗi model trong collection:

```php
$users = $users->setHidden(['email', 'password', 'remember_token']);
```

<a name="method-toquery"></a>
#### `toQuery()` {.collection-method}

Method `toQuery` trả về một Eloquent query builder instance chứa một `whereIn` constraint trên collection model's primary keys:

```php
use App\Models\User;

$users = User::where('status', 'VIP')->get();

$users->toQuery()->update([
    'status' => 'Administrator',
]);
```

<a name="method-unique"></a>
#### `unique($key = null, $strict = false)` {.collection-method}

Method `unique` trả về tất cả unique models trong collection. Bất kỳ models có cùng primary key với một model khác trong collection sẽ được removed:

```php
$users = $users->unique();
```

<a name="method-withoutAppends"></a>
#### `withoutAppends()` {.collection-method}

Method `withoutAppends` tạm thời removes tất cả [appended attributes](/docs/{{version}}/eloquent-serialization#appending-values-to-json) trên mỗi model trong collection:

```php
$users = $users->withoutAppends();
```

<a name="custom-collections"></a>
## Custom Collections

Nếu bạn muốn sử dụng một custom `Collection` object khi interacting với một given model, bạn có thể add `CollectedBy` attribute vào model của bạn:

```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Attributes\CollectedBy;
use Illuminate\Database\Eloquent\Model;

#[CollectedBy(UserCollection::class)]
class User extends Model
{
    // ...
}
```

Hoặc, bạn có thể define một `newCollection` method trên model của bạn:

```php
<?php

namespace App\Models;

use App\Support\UserCollection;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Create a new Eloquent Collection instance.
     *
     * @param  array<int, \Illuminate\Database\Eloquent\Model>  $models
     * @return \Illuminate\Database\Eloquent\Collection<int, \Illuminate\Database\Eloquent\Model>
     */
    public function newCollection(array $models = []): Collection
    {
        $collection = new UserCollection($models);

        if (Model::isAutomaticallyEagerLoadingRelationships()) {
            $collection->withRelationshipAutoloading();
        }

        return $collection;
    }
}
```

Khi bạn đã define một `newCollection` method hoặc added `CollectedBy` attribute vào model của bạn, bạn sẽ nhận được một instance của custom collection của bạn bất cứ khi nào Eloquent thường sẽ return một `Illuminate\Database\Eloquent\Collection` instance.

Nếu bạn muốn sử dụng một custom collection cho mỗi model trong application của bạn, bạn nên define `newCollection` method trên một base model class được extended bởi tất cả models trong application của bạn.

# Eloquent: Relationships

- [Introduction](#introduction)
- [Defining Relationships](#defining-relationships)
    - [One to One / Has One](#one-to-one)
    - [One to Many / Has Many](#one-to-many)
    - [One to Many (Inverse) / Belongs To](#one-to-many-inverse)
    - [Has One of Many](#has-one-of-many)
    - [Has One Through](#has-one-through)
    - [Has Many Through](#has-many-through)
- [Scoped Relationships](#scoped-relationships)
- [Many to Many Relationships](#many-to-many)
    - [Retrieving Intermediate Table Columns](#retrieving-intermediate-table-columns)
    - [Filtering Queries via Intermediate Table Columns](#filtering-queries-via-intermediate-table-columns)
    - [Ordering Queries via Intermediate Table Columns](#ordering-queries-via-intermediate-table-columns)
    - [Defining Custom Intermediate Table Models](#defining-custom-intermediate-table-models)
- [Polymorphic Relationships](#polymorphic-relationships)
    - [One to One](#one-to-one-polymorphic-relations)
    - [One to Many](#one-to-many-polymorphic-relations)
    - [One of Many](#one-of-many-polymorphic-relations)
    - [Many to Many](#many-to-many-polymorphic-relations)
    - [Custom Polymorphic Types](#custom-polymorphic-types)
- [Dynamic Relationships](#dynamic-relationships)
- [Querying Relations](#querying-relations)
    - [Relationship Methods vs. Dynamic Properties](#relationship-methods-vs-dynamic-properties)
    - [Querying Relationship Existence](#querying-relationship-existence)
    - [Querying Relationship Absence](#querying-relationship-absence)
    - [Querying Morph To Relationships](#querying-morph-to-relationships)
- [Aggregating Related Models](#aggregating-related-models)
    - [Counting Related Models](#counting-related-models)
    - [Other Aggregate Functions](#other-aggregate-functions)
    - [Counting Related Models on Morph To Relationships](#counting-related-models-on-morph-to-relationships)
- [Eager Loading](#eager-loading)
    - [Constraining Eager Loads](#constraining-eager-loads)
    - [Lazy Eager Loading](#lazy-eager-loading)
    - [Automatic Eager Loading](#automatic-eager-loading)
    - [Preventing Lazy Loading](#preventing-lazy-loading)
- [Inserting and Updating Related Models](#inserting-and-updating-related-models)
    - [The `save` Method](#the-save-method)
    - [The `create` Method](#the-create-method)
    - [Belongs To Relationships](#updating-belongs-to-relationships)
    - [Many to Many Relationships](#updating-many-to-many-relationships)
- [Touching Parent Timestamps](#touching-parent-timestamps)

<a name="introduction"></a>
## Introduction

Database tables thường related với nhau. Ví dụ, một blog post có thể có nhiều comments hoặc một order có thể related đến user đã placed nó. Eloquent makes managing và working với những relationships này easy, và supports một variety của common relationships:

<div class="content-list" markdown="1">

- [One To One](#one-to-one)
- [One To Many](#one-to-many)
- [Many To Many](#many-to-many)
- [Has One Through](#has-one-through)
- [Has Many Through](#has-many-through)
- [One To One (Polymorphic)](#one-to-one-polymorphic-relations)
- [One To Many (Polymorphic)](#one-to-many-polymorphic-relations)
- [Many To Many (Polymorphic)](#many-to-many-polymorphic-relations)

</div>

<a name="defining-relationships"></a>
## Defining Relationships

Eloquent relationships được defined như methods trên Eloquent model classes của bạn. Vì relationships cũng serve như powerful [query builders](/docs/{{version}}/queries), defining relationships như methods provides powerful method chaining và querying capabilities. Ví dụ, chúng ta có thể chain additional query constraints trên `posts` relationship này:

```php
$user->posts()->where('active', 1)->get();
```

Tuy nhiên, trước khi diving quá deep vào sử dụng relationships, hãy học cách define mỗi type của relationship supported bởi Eloquent.

<a name="one-to-one"></a>
### One to One / Has One

Một one-to-one relationship là một very basic type của database relationship. Ví dụ, một `User` model có thể được associated với một `Phone` model. Để define relationship này, chúng ta sẽ place một `phone` method trên `User` model. `phone` method nên call `hasOne` method và return result của nó. `hasOne` method available đến model của bạn qua model's `Illuminate\Database\Eloquent\Model` base class:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * Get the phone associated with the user.
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

First argument passed đến `hasOne` method là name của related model class. Khi relationship được defined, chúng ta có thể retrieve related record sử dụng Eloquent's dynamic properties. Dynamic properties allow bạn để access relationship methods như thể chúng là properties defined trên model:

```php
$phone = User::find(1)->phone;
```

Eloquent determines foreign key của relationship dựa trên parent model name. Trong case này, `Phone` model được automatically assumed để có một `user_id` foreign key. Nếu bạn muốn override convention này, bạn có thể pass một second argument đến `hasOne` method:

```php
return $this->hasOne(Phone::class, 'foreign_key');
```

Ngoài ra, Eloquent assumes rằng foreign key nên có một value matching primary key column của parent. Nói cách khác, Eloquent sẽ look cho value của user's `id` column trong `user_id` column của `Phone` record. Nếu bạn muốn relationship để sử dụng một primary key value khác `id` hoặc model's primary key của bạn, bạn có thể pass một third argument đến `hasOne` method:

```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

<a name="one-to-one-defining-the-inverse-of-the-relationship"></a>
#### Defining the Inverse of the Relationship

Vì vậy, chúng ta có thể access `Phone` model từ `User` model của chúng ta. Tiếp theo, hãy define một relationship trên `Phone` model sẽ let chúng ta access user owns phone. Chúng ta có thể define inverse của một `hasOne` relationship sử dụng `belongsTo` method:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * Get the user that owns the phone.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

Khi invoking `user` method, Eloquent sẽ attempt để find một `User` model có một `id` matches `user_id` column trên `Phone` model.

Eloquent determines foreign key name bằng cách examining name của relationship method và suffixing method name với `_id`. Vì vậy, trong case này, Eloquent assumes rằng `Phone` model có một `user_id` column. Tuy nhiên, nếu foreign key trên `Phone` model không phải `user_id`, bạn có thể pass một custom key name như second argument đến `belongsTo` method:

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```

Nếu parent model không sử dụng `id` như primary key của nó, hoặc bạn muốn find associated model sử dụng một different column, bạn có thể pass một third argument đến `belongsTo` method specifying parent table's custom key:

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

<a name="one-to-many"></a>
### One to Many / Has Many

Một one-to-many relationship được sử dụng để define relationships nơi một single model là parent đến một hoặc nhiều child models. Ví dụ, một blog post có thể có một infinite number của comments. Như tất cả other Eloquent relationships, one-to-many relationships được defined bằng cách defining một method trên Eloquent model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

Remember, Eloquent sẽ automatically determine proper foreign key column cho `Comment` model. Theo convention, Eloquent sẽ take "snake case" name của parent model và suffix nó với `_id`. Vì vậy, trong example này, Eloquent sẽ assume foreign key column trên `Comment` model là `post_id`.

Khi relationship method đã được defined, chúng ta có thể access [collection](/docs/{{version}}/eloquent-collections) của related comments bằng cách accessing `comments` property. Remember, vì Eloquent provides "dynamic relationship properties", chúng ta có thể access relationship methods như thể chúng được defined như properties trên model:

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

Since all relationships also serve as query builders, you may add further constraints to the relationship query by calling the `comments` method and continuing to chain conditions onto the query:

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

Like the `hasOne` method, you may also override the foreign and local keys by passing additional arguments to the `hasMany` method:

```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

<a name="automatically-hydrating-parent-models-on-children"></a>
#### Automatically Hydrating Parent Models on Children

Even when utilizing Eloquent eager loading, "N + 1" query problems can arise if you try to access the parent model from a child model while looping through the child models:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

In the example above, an "N + 1" query problem has been introduced because, even though comments were eager loaded for every `Post` model, Eloquent does not automatically hydrate the parent `Post` on each child `Comment` model.

If you would like Eloquent to automatically hydrate parent models onto their children, you may invoke the `chaperone` method when defining a `hasMany` relationship:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```

Or, if you would like to opt-in to automatic parent hydration at run time, you may invoke the `chaperone` model when eager loading the relationship:

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-to-many-inverse"></a>
### One to Many (Inverse) / Belongs To

Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. To define the inverse of a `hasMany` relationship, define a relationship method on the child model which calls the `belongsTo` method:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * Get the post that owns the comment.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

Once the relationship has been defined, we can retrieve a comment's parent post by accessing the `post` "dynamic relationship property":

```php
use App\Models\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```

In the example above, Eloquent will attempt to find a `Post` model that has an `id` which matches the `post_id` column on the `Comment` model.

Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with a `_` followed by the name of the parent model's primary key column. So, in this example, Eloquent will assume the `Post` model's foreign key on the `comments` table is `post_id`.

However, if the foreign key for your relationship does not follow these conventions, you may pass a custom foreign key name as the second argument to the `belongsTo` method:

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```

If your parent model does not use `id` as its primary key, or you wish to find the associated model using a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
}
```

<a name="default-models"></a>
#### Default Models

The `belongsTo`, `hasOne`, `hasOneThrough`, and `morphOne` relationships allow you to define a default model that will be returned if the given relationship is `null`. This pattern is often referred to as the [Null Object pattern](https://en.wikipedia.org/wiki/Null_Object_pattern) and can help remove conditional checks in your code. In the following example, the `user` relation will return an empty `App\Models\User` model if no user is attached to the `Post` model:

```php
/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

To populate the default model with attributes, you may pass an array or closure to the `withDefault` method:

```php
/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}

/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Guest Author';
    });
}
```

<a name="querying-belongs-to-relationships"></a>
#### Querying Belongs To Relationships

When querying for the children of a "belongs to" relationship, you may manually build the `where` clause to retrieve the corresponding Eloquent models:

```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```

However, you may find it more convenient to use the `whereBelongsTo` method, which will automatically determine the proper relationship and foreign key for the given model:

```php
$posts = Post::whereBelongsTo($user)->get();
```

You may also provide a [collection](/docs/{{version}}/eloquent-collections) instance to the `whereBelongsTo` method. When doing so, Laravel will retrieve models that belong to any of the parent models within the collection:

```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```

By default, Laravel will determine the relationship associated with the given model based on the class name of the model; however, you may specify the relationship name manually by providing it as the second argument to the `whereBelongsTo` method:

```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

<a name="has-one-of-many"></a>
### Has One of Many

Sometimes a model may have many related models, yet you want to easily retrieve the "latest" or "oldest" related model of the relationship. For example, a `User` model may be related to many `Order` models, but you want to define a convenient way to interact with the most recent order the user has placed. You may accomplish this using the `hasOne` relationship type combined with the `ofMany` methods:

```php
/**
 * Get the user's most recent order.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

Likewise, you may define a method to retrieve the "oldest", or first, related model of a relationship:

```php
/**
 * Get the user's oldest order.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

By default, the `latestOfMany` and `oldestOfMany` methods will retrieve the latest or oldest related model based on the model's primary key, which must be sortable. However, sometimes you may wish to retrieve a single model from a larger relationship using a different sorting criteria.

For example, using the `ofMany` method, you may retrieve the user's most expensive order. The `ofMany` method accepts the sortable column as its first argument and which aggregate function (`min` or `max`) to apply when querying for the related model:

```php
/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

> [!WARNING]
> Because PostgreSQL does not support executing the `MAX` function against UUID columns, it is not currently possible to use one-of-many relationships in combination with PostgreSQL UUID columns.

<a name="converting-many-relationships-to-has-one-relationships"></a>
#### Converting "Many" Relationships to Has One Relationships

Often, when retrieving a single model using the `latestOfMany`, `oldestOfMany`, or `ofMany` methods, you already have a "has many" relationship defined for the same model. For convenience, Laravel allows you to easily convert this relationship into a "has one" relationship by invoking the `one` method on the relationship:

```php
/**
 * Get the user's orders.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

You may also use the `one` method to convert `HasManyThrough` relationships to `HasOneThrough` relationships:

```php
public function latestDeployment(): HasOneThrough
{
    return $this->deployments()->one()->latestOfMany();
}
```

<a name="advanced-has-one-of-many-relationships"></a>
#### Advanced Has One of Many Relationships

It is possible to construct more advanced "has one of many" relationships. For example, a `Product` model may have many associated `Price` models that are retained in the system even after new pricing is published. In addition, new pricing data for the product may be able to be published in advance to take effect at a future date via a `published_at` column.

So, in summary, we need to retrieve the latest published pricing where the published date is not in the future. In addition, if two prices have the same published date, we will prefer the price with the greatest ID. To accomplish this, we must pass an array to the `ofMany` method that contains the sortable columns which determine the latest price. In addition, a closure will be provided as the second argument to the `ofMany` method. This closure will be responsible for adding additional publish date constraints to the relationship query:

```php
/**
 * Get the current pricing for the product.
 */
public function currentPricing(): HasOne
{
    return $this->hasOne(Price::class)->ofMany([
        'published_at' => 'max',
        'id' => 'max',
    ], function (Builder $query) {
        $query->where('published_at', '<', now());
    });
}
```

<a name="has-one-through"></a>
### Has One Through

Relationship "has-one-through" định nghĩa một one-to-one relationship với một model khác. Tuy nhiên, relationship này chỉ ra rằng model đang khai báo có thể được matched với một instance của một model khác bằng cách đi _through_ một model thứ ba.

Ví dụ, trong một ứng dụng cửa hàng sửa xe, mỗi `Mechanic` model có thể được associated với một `Car` model, và mỗi `Car` model có thể được associated với một `Owner` model. Trong khi mechanic và owner không có direct relationship trong database, mechanic có thể access owner _through_ `Car` model. Hãy xem các tables cần thiết để define relationship này:

```text
mechanics
    id - integer
    name - string

cars
    id - integer
    model - string
    mechanic_id - integer

owners
    id - integer
    name - string
    car_id - integer
```

Bây giờ chúng ta đã examined table structure cho relationship, hãy define relationship trên `Mechanic` model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

Argument đầu tiên passed đến `hasOneThrough` method là name của final model chúng ta muốn access, trong khi argument thứ hai là name của intermediate model.

Hoặc, nếu relevant relationships đã được defined trên tất cả các models involved trong relationship, bạn có thể fluently define một "has-one-through" relationship bằng cách invoking `through` method và supplying names của những relationships đó. Ví dụ, nếu `Mechanic` model có một `cars` relationship và `Car` model có một `owner` relationship, bạn có thể define một "has-one-through" relationship connecting mechanic và owner như sau:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-one-through-key-conventions"></a>
#### Key Conventions

Typical Eloquent foreign key conventions sẽ được sử dụng khi performing relationship's queries. Nếu bạn muốn customize keys của relationship, bạn có thể pass chúng như third và fourth arguments đến `hasOneThrough` method. Argument thứ ba là name của foreign key trên intermediate model. Argument thứ tư là name của foreign key trên final model. Argument thứ năm là local key, trong khi argument thứ sáu là local key của intermediate model:

```php
class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(
            Owner::class,
            Car::class,
            'mechanic_id', // Foreign key on the cars table...
            'car_id', // Foreign key on the owners table...
            'id', // Local key on the mechanics table...
            'id' // Local key on the cars table...
        );
    }
}
```

Hoặc, như đã discussed earlier, nếu relevant relationships đã được defined trên tất cả các models involved trong relationship, bạn có thể fluently define một "has-one-through" relationship bằng cách invoking `through` method và supplying names của những relationships đó. Approach này offers advantage của reusing key conventions đã được defined trên existing relationships:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-many-through"></a>
### Has Many Through

Relationship "has-many-through" provides một convenient way để access distant relations via một intermediate relation. Ví dụ, hãy assume chúng ta đang building một deployment platform như [Laravel Cloud](https://cloud.laravel.com). Một `Application` model có thể access nhiều `Deployment` models through một intermediate `Environment` model. Sử dụng example này, bạn có thể easily gather tất cả deployments cho một given application. Hãy xem các tables cần thiết để define relationship này:

```text
applications
    id - integer
    name - string

environments
    id - integer
    application_id - integer
    name - string

deployments
    id - integer
    environment_id - integer
    commit_hash - string
```

Bây giờ chúng ta đã examined table structure cho relationship, hãy define relationship trên `Application` model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    /**
     * Get all of the deployments for the application.
     */
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```

Argument đầu tiên passed đến `hasManyThrough` method là name của final model chúng ta muốn access, trong khi argument thứ hai là name của intermediate model.

Hoặc, nếu relevant relationships đã được defined trên tất cả các models involved trong relationship, bạn có thể fluently define một "has-many-through" relationship bằng cách invoking `through` method và supplying names của những relationships đó. Ví dụ, nếu `Application` model có một `environments` relationship và `Environment` model có một `deployments` relationship, bạn có thể define một "has-many-through" relationship connecting application và deployments như sau:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

Mặc dù `Deployment` model's table không chứa một `application_id` column, `hasManyThrough` relation provides access đến một application's deployments via `$application->deployments`. Để retrieve những models này, Eloquent inspects `application_id` column trên intermediate `Environment` model's table. Sau khi finding relevant environment IDs, chúng được sử dụng để query `Deployment` model's table.

<a name="has-many-through-key-conventions"></a>
#### Key Conventions

Typical Eloquent foreign key conventions sẽ được sử dụng khi performing relationship's queries. Nếu bạn muốn customize keys của relationship, bạn có thể pass chúng như third và fourth arguments đến `hasManyThrough` method. Argument thứ ba là name của foreign key trên intermediate model. Argument thứ tư là name của foreign key trên final model. Argument thứ năm là local key, trong khi argument thứ sáu là local key của intermediate model:

```php
class Application extends Model
{
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(
            Deployment::class,
            Environment::class,
            'application_id', // Foreign key on the environments table...
            'environment_id', // Foreign key on the deployments table...
            'id', // Local key on the applications table...
            'id' // Local key on the environments table...
        );
    }
}
```

Hoặc, như đã discussed earlier, nếu relevant relationships đã được defined trên tất cả các models involved trong relationship, bạn có thể fluently define một "has-many-through" relationship bằng cách invoking `through` method và supplying names của những relationships đó. Approach này offers advantage của reusing key conventions đã được defined trên existing relationships:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

<a name="scoped-relationships"></a>
### Scoped Relationships

Rất common để add additional methods đến models mà constrain relationships. Ví dụ, bạn có thể add một `featuredPosts` method đến một `User` model mà constrains broader `posts` relationship với một additional `where` constraint:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get the user's posts.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class)->latest();
    }

    /**
     * Get the user's featured posts.
     */
    public function featuredPosts(): HasMany
    {
        return $this->posts()->where('featured', true);
    }
}
```

Tuy nhiên, nếu bạn attempt để create một model via `featuredPosts` method, `featured` attribute của nó sẽ không được set thành `true`. Nếu bạn muốn create models via relationship methods và cũng specify attributes mà nên được added đến tất cả models created via relationship đó, bạn có thể sử dụng `withAttributes` method khi building relationship query:

```php
/**
 * Get the user's featured posts.
 */
public function featuredPosts(): HasMany
{
    return $this->posts()->withAttributes(['featured' => true]);
}
```

`withAttributes` method sẽ add `where` conditions đến query sử dụng given attributes, và nó cũng sẽ add given attributes đến bất kỳ models nào created via relationship method:

```php
$post = $user->featuredPosts()->create(['title' => 'Featured Post']);

$post->featured; // true
```

Để instruct `withAttributes` method để không add `where` conditions đến query, bạn có thể set `asConditions` argument thành `false`:

```php
return $this->posts()->withAttributes(['featured' => true], asConditions: false);
```

<a name="many-to-many"></a>
## Many to Many Relationships

Many-to-many relations hơi more complicated hơn `hasOne` và `hasMany` relationships. Một example của many-to-many relationship là một user mà có nhiều roles và những roles đó cũng được shared bởi other users trong application. Ví dụ, một user có thể được assigned role của "Author" và "Editor"; tuy nhiên, những roles đó cũng có thể được assigned đến other users. Vì vậy, một user có nhiều roles và một role có nhiều users.

<a name="many-to-many-table-structure"></a>
#### Table Structure

Để define relationship này, ba database tables cần thiết: `users`, `roles`, và `role_user`. `role_user` table được derived từ alphabetical order của related model names và chứa `user_id` và `role_id` columns. Table này được sử dụng như một intermediate table linking users và roles.

Remember, vì một role có thể belong đến nhiều users, chúng ta không thể simply place một `user_id` column trên `roles` table. Điều này sẽ mean rằng một role chỉ có thể belong đến một single user. Để provide support cho roles being assigned đến multiple users, `role_user` table cần thiết. Chúng ta có thể summarize relationship's table structure như sau:

```text
users
    id - integer
    name - string

roles
    id - integer
    name - string

role_user
    user_id - integer
    role_id - integer
```

<a name="many-to-many-model-structure"></a>
#### Model Structure

Many-to-many relationships được defined bằng cách writing một method mà returns result của `belongsToMany` method. `belongsToMany` method được provided bởi `Illuminate\Database\Eloquent\Model` base class mà được sử dụng bởi tất cả application's Eloquent models của bạn. Ví dụ, hãy define một `roles` method trên `User` model của chúng ta. Argument đầu tiên passed đến method này là name của related model class:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    /**
     * The roles that belong to the user.
     */
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

Khi relationship đã được defined, bạn có thể access user's roles sử dụng `roles` dynamic relationship property:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    // ...
}
```

Vì tất cả relationships cũng serve như query builders, bạn có thể add further constraints đến relationship query bằng cách calling `roles` method và continuing để chain conditions lên query:

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

Để determine table name của relationship's intermediate table, Eloquent sẽ join hai related model names trong alphabetical order. Tuy nhiên, bạn có thể tự do override convention này. Bạn có thể làm điều đó bằng cách passing một second argument đến `belongsToMany` method:

```php
return $this->belongsToMany(Role::class, 'role_user');
```

Ngoài việc customizing name của intermediate table, bạn cũng có thể customize column names của keys trên table bằng cách passing additional arguments đến `belongsToMany` method. Argument thứ ba là foreign key name của model mà bạn đang defining relationship, trong khi argument thứ tư là foreign key name của model mà bạn đang joining đến:

```php
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

<a name="many-to-many-defining-the-inverse-of-the-relationship"></a>
#### Defining the Inverse of the Relationship

Để define "inverse" của một many-to-many relationship, bạn nên define một method trên related model mà cũng returns result của `belongsToMany` method. Để complete user / role example của chúng ta, hãy define `users` method trên `Role` model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

Như bạn có thể thấy, relationship được defined exactly the same như `User` model counterpart của nó với exception của referencing `App\Models\User` model. Vì chúng ta đang reusing `belongsToMany` method, tất cả usual table và key customization options đều available khi defining "inverse" của many-to-many relationships.

<a name="retrieving-intermediate-table-columns"></a>
### Retrieving Intermediate Table Columns

Như bạn đã learned, working với many-to-many relations requires presence của một intermediate table. Eloquent provides một số very helpful ways của interacting với table này. Ví dụ, hãy assume `User` model của chúng ta có nhiều `Role` models mà nó related đến. Sau khi accessing relationship này, chúng ta có thể access intermediate table sử dụng `pivot` attribute trên models:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

Notice rằng mỗi `Role` model chúng ta retrieve được automatically assigned một `pivot` attribute. Attribute này chứa một model representing intermediate table.

Theo mặc định, chỉ model keys sẽ present trên `pivot` model. Nếu intermediate table của bạn chứa extra attributes, bạn phải specify chúng khi defining relationship:

```php
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');
```

Nếu bạn muốn intermediate table của bạn có `created_at` và `updated_at` timestamps mà được automatically maintained bởi Eloquent, call `withTimestamps` method khi defining relationship:

```php
return $this->belongsToMany(Role::class)->withTimestamps();
```

> [!WARNING]
> Intermediate tables that utilize Eloquent's automatically maintained timestamps are required to have both `created_at` and `updated_at` timestamp columns.

<a name="customizing-the-pivot-attribute-name"></a>
#### Customizing the `pivot` Attribute Name

Như đã noted previously, attributes từ intermediate table có thể được accessed trên models via `pivot` attribute. Tuy nhiên, bạn có thể tự do customize name của attribute này để better reflect purpose của nó trong application của bạn.

Ví dụ, nếu application của bạn chứa users mà có thể subscribe đến podcasts, bạn likely có một many-to-many relationship giữa users và podcasts. Nếu đây là case, bạn có thể wish để rename intermediate table attribute của bạn thành `subscription` thay vì `pivot`. Điều này có thể được done sử dụng `as` method khi defining relationship:

```php
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();
```

Khi custom intermediate table attribute đã được specified, bạn có thể access intermediate table data sử dụng customized name:

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

<a name="filtering-queries-via-intermediate-table-columns"></a>
### Filtering Queries via Intermediate Table Columns

Bạn cũng có thể filter results returned bởi `belongsToMany` relationship queries sử dụng `wherePivot`, `wherePivotIn`, `wherePivotNotIn`, `wherePivotBetween`, `wherePivotNotBetween`, `wherePivotNull`, và `wherePivotNotNull` methods khi defining relationship:

```php
return $this->belongsToMany(Role::class)
    ->wherePivot('approved', 1);

return $this->belongsToMany(Role::class)
    ->wherePivotIn('priority', [1, 2]);

return $this->belongsToMany(Role::class)
    ->wherePivotNotIn('priority', [1, 2]);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNull('expired_at');

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotNull('expired_at');
```

`wherePivot` adds một where clause constraint đến query, nhưng không add specified value khi creating new models via defined relationship. Nếu bạn cần cả query và create relationships với một particular pivot value, bạn có thể sử dụng `withPivotValue` method:

```php
return $this->belongsToMany(Role::class)
    ->withPivotValue('approved', 1);
```

<a name="ordering-queries-via-intermediate-table-columns"></a>
### Ordering Queries via Intermediate Table Columns

Bạn có thể order results returned bởi `belongsToMany` relationship queries sử dụng `orderByPivot` và `orderByPivotDesc` methods. Trong following example, chúng ta sẽ retrieve tất cả latest badges cho user:

```php
return $this->belongsToMany(Badge::class)
    ->where('rank', 'gold')
    ->orderByPivotDesc('created_at');
```

<a name="defining-custom-intermediate-table-models"></a>
### Defining Custom Intermediate Table Models

Nếu bạn muốn define một custom model để represent intermediate table của many-to-many relationship của bạn, bạn có thể call `using` method khi defining relationship. Custom pivot models give bạn opportunity để define additional behavior trên pivot model, như methods và casts.

Custom many-to-many pivot models nên extend `Illuminate\Database\Eloquent\Relations\Pivot` class trong khi custom polymorphic many-to-many pivot models nên extend `Illuminate\Database\Eloquent\Relations\MorphPivot` class. Ví dụ, chúng ta có thể define một `Role` model mà sử dụng một custom `RoleUser` pivot model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class)->using(RoleUser::class);
    }
}
```

Khi defining `RoleUser` model, bạn nên extend `Illuminate\Database\Eloquent\Relations\Pivot` class:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    // ...
}
```

> [!WARNING]
> Pivot models có thể không sử dụng `SoftDeletes` trait. Nếu bạn cần soft delete pivot records hãy consider converting pivot model của bạn thành một actual Eloquent model.

<a name="custom-pivot-models-and-incrementing-ids"></a>
#### Custom Pivot Models and Incrementing IDs

Nếu bạn đã defined một many-to-many relationship mà sử dụng một custom pivot model, và pivot model đó có một auto-incrementing primary key, bạn nên ensure custom pivot model class của bạn sử dụng `Table` attribute với `incrementing` set thành `true`:

```php
use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Relations\Pivot;

#[Table(incrementing: true)]
class RoleUser extends Pivot
{
    // ...
}
```

<a name="polymorphic-relationships"></a>
## Polymorphic Relationships

Một polymorphic relationship allows child model để belong đến more than one type của model sử dụng một single association. Ví dụ, imagine bạn đang building một application mà allows users để share blog posts và videos. Trong application như vậy, một `Comment` model có thể belong đến cả `Post` và `Video` models.

<a name="one-to-one-polymorphic-relations"></a>
### One to One (Polymorphic)

<a name="one-to-one-polymorphic-table-structure"></a>
#### Table Structure

Một one-to-one polymorphic relation là similar đến một typical one-to-one relation; tuy nhiên, child model có thể belong đến more than one type của model sử dụng một single association. Ví dụ, một blog `Post` và một `User` có thể share một polymorphic relation đến một `Image` model. Sử dụng một one-to-one polymorphic relation allows bạn để có một single table của unique images mà có thể được associated với posts và users. First, hãy examine table structure:

```text
posts
    id - integer
    name - string

users
    id - integer
    name - string

images
    id - integer
    url - string
    imageable_type - string
    imageable_id - integer
```

Note `imageable_id` và `imageable_type` columns trên `images` table. `imageable_id` column sẽ chứa ID value của post hoặc user, trong khi `imageable_type` column sẽ chứa class name của parent model. `imageable_type` column được sử dụng bởi Eloquent để determine "type" nào của parent model để return khi accessing `imageable` relation. Trong case này, column sẽ chứa hoặc `App\Models\Post` hoặc `App\Models\User`.

<a name="one-to-one-polymorphic-model-structure"></a>
#### Model Structure

Tiếp theo, hãy examine model definitions cần thiết để build relationship này:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Image extends Model
{
    /**
     * Get the parent imageable model (user or post).
     */
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class Post extends Model
{
    /**
     * Get the post's image.
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class User extends Model
{
    /**
     * Get the user's image.
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

<a name="one-to-one-polymorphic-retrieving-the-relationship"></a>
#### Retrieving the Relationship

Khi database table và models của bạn đã được defined, bạn có thể access relationships via models của bạn. Ví dụ, để retrieve image cho một post, chúng ta có thể access `image` dynamic relationship property:

```php
use App\Models\Post;

$post = Post::find(1);

$image = $post->image;
```

Bạn có thể retrieve parent của polymorphic model bằng cách accessing name của method mà performs call đến `morphTo`. Trong case này, đó là `imageable` method trên `Image` model. Vì vậy, chúng ta sẽ access method đó như một dynamic relationship property:

```php
use App\Models\Image;

$image = Image::find(1);

$imageable = $image->imageable;
```

`imageable` relation trên `Image` model sẽ return hoặc một `Post` hoặc `User` instance, tùy thuộc vào type nào của model owns image.

<a name="morph-one-to-one-key-conventions"></a>
#### Key Conventions

Nếu necessary, bạn có thể specify name của "id" và "type" columns utilized bởi polymorphic child model của bạn. Nếu bạn làm điều đó, ensure rằng bạn luôn pass name của relationship như first argument đến `morphTo` method. Thông thường, value này nên match method name, vì vậy bạn có thể sử dụng PHP's `__FUNCTION__` constant:

```php
/**
 * Get the model that the image belongs to.
 */
public function imageable(): MorphTo
{
    return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
}
```

<a name="one-to-many-polymorphic-relations"></a>
### One to Many (Polymorphic)

<a name="one-to-many-polymorphic-table-structure"></a>
#### Table Structure

Một one-to-many polymorphic relation là similar đến một typical one-to-many relation; tuy nhiên, child model có thể belong đến more than one type của model sử dụng một single association. Ví dụ, imagine users của application của bạn có thể "comment" trên posts và videos. Sử dụng polymorphic relationships, bạn có thể sử dụng một single `comments` table để contain comments cho cả posts và videos. First, hãy examine table structure cần thiết để build relationship này:

```text
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_type - string
    commentable_id - integer
```

<a name="one-to-many-polymorphic-model-structure"></a>
#### Model Structure

Tiếp theo, hãy examine model definitions cần thiết để build relationship này:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    /**
     * Get the parent commentable model (post or video).
     */
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Post extends Model
{
    /**
     * Get all of the post's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Video extends Model
{
    /**
     * Get all of the video's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

<a name="one-to-many-polymorphic-retrieving-the-relationship"></a>
#### Retrieving the Relationship

Khi database table và models của bạn đã được defined, bạn có thể access relationships via model's dynamic relationship properties của bạn. Ví dụ, để access tất cả comments cho một post, chúng ta có thể sử dụng `comments` dynamic property:

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->comments as $comment) {
    // ...
}
```

Bạn cũng có thể retrieve parent của một polymorphic child model bằng cách accessing name của method mà performs call đến `morphTo`. Trong case này, đó là `commentable` method trên `Comment` model. Vì vậy, chúng ta sẽ access method đó như một dynamic relationship property để access comment's parent model:

```php
use App\Models\Comment;

$comment = Comment::find(1);

$commentable = $comment->commentable;
```

`commentable` relation trên `Comment` model sẽ return hoặc một `Post` hoặc `Video` instance, tùy thuộc vào type nào của model là comment's parent.

<a name="polymorphic-automatically-hydrating-parent-models-on-children"></a>
#### Automatically Hydrating Parent Models on Children

Ngay cả khi utilizing Eloquent eager loading, "N + 1" query problems có thể arise nếu bạn attempt để access parent model từ một child model trong khi looping qua child models:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->commentable->title;
    }
}
```

In the example above, an "N + 1" query problem has been introduced because, even though comments were eager loaded for every `Post` model, Eloquent does not automatically hydrate the parent `Post` on each child `Comment` model.

If you would like Eloquent to automatically hydrate parent models onto their children, you may invoke the `chaperone` method when defining a `morphMany` relationship:

```php
class Post extends Model
{
    /**
     * Get all of the post's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable')->chaperone();
    }
}
```

Or, if you would like to opt-in to automatic parent hydration at run time, you may invoke the `chaperone` model when eager loading the relationship:

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-of-many-polymorphic-relations"></a>
### One of Many (Polymorphic)

Sometimes a model may have many related models, yet you want to easily retrieve the "latest" or "oldest" related model of the relationship. For example, a `User` model may be related to many `Image` models, but you want to define a convenient way to interact with the most recent image the user has uploaded. You may accomplish this using the `morphOne` relationship type combined with the `ofMany` methods:

```php
/**
 * Get the user's most recent image.
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}
```

Likewise, you may define a method to retrieve the "oldest", or first, related model of a relationship:

```php
/**
 * Get the user's oldest image.
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}
```

By default, the `latestOfMany` and `oldestOfMany` methods will retrieve the latest or oldest related model based on the model's primary key, which must be sortable. However, sometimes you may wish to retrieve a single model from a larger relationship using a different sorting criteria.

For example, using the `ofMany` method, you may retrieve the user's most "liked" image. The `ofMany` method accepts the sortable column as its first argument and which aggregate function (`min` or `max`) to apply when querying for the related model:

```php
/**
 * Get the user's most popular image.
 */
public function bestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('likes', 'max');
}
```

> [!NOTE]
> It is possible to construct more advanced "one of many" relationships. For more information, please consult the [has one of many documentation](#advanced-has-one-of-many-relationships).

<a name="many-to-many-polymorphic-relations"></a>
### Many to Many (Polymorphic)

<a name="many-to-many-polymorphic-table-structure"></a>
#### Table Structure

Many-to-many polymorphic relations are slightly more complicated than "morph one" and "morph many" relationships. For example, a `Post` model and `Video` model could share a polymorphic relation to a `Tag` model. Using a many-to-many polymorphic relation in this situation would allow your application to have a single table of unique tags that may be associated with posts or videos. First, let's examine the table structure required to build this relationship:

```text
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_type - string
    taggable_id - integer
```

> [!NOTE]
> Before diving into polymorphic many-to-many relationships, you may benefit from reading the documentation on typical [many-to-many relationships](#many-to-many).

<a name="many-to-many-polymorphic-model-structure"></a>
#### Model Structure

Next, we're ready to define the relationships on the models. The `Post` and `Video` models will both contain a `tags` method that calls the `morphToMany` method provided by the base Eloquent model class.

The `morphToMany` method accepts the name of the related model as well as the "relationship name". Based on the name we assigned to our intermediate table name and the keys it contains, we will refer to the relationship as "taggable":

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Post extends Model
{
    /**
     * Get all of the tags for the post.
     */
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-defining-the-inverse-of-the-relationship"></a>
#### Defining the Inverse of the Relationship

Next, on the `Tag` model, you should define a method for each of its possible parent models. So, in this example, we will define a `posts` method and a `videos` method. Both of these methods should return the result of the `morphedByMany` method.

The `morphedByMany` method accepts the name of the related model as well as the "relationship name". Based on the name we assigned to our intermediate table name and the keys it contains, we will refer to the relationship as "taggable":

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    /**
     * Get all of the posts that are assigned this tag.
     */
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    /**
     * Get all of the videos that are assigned this tag.
     */
    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-retrieving-the-relationship"></a>
#### Retrieving the Relationship

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the tags for a post, you may use the `tags` dynamic relationship property:

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}
```

You may retrieve the parent of a polymorphic relation from the polymorphic child model by accessing the name of the method that performs the call to `morphedByMany`. In this case, that is the `posts` or `videos` methods on the `Tag` model:

```php
use App\Models\Tag;

$tag = Tag::find(1);

foreach ($tag->posts as $post) {
    // ...
}

foreach ($tag->videos as $video) {
    // ...
}
```

<a name="custom-polymorphic-types"></a>
### Custom Polymorphic Types

By default, Laravel will use the fully qualified class name to store the "type" of the related model. For instance, given the one-to-many relationship example above where a `Comment` model may belong to a `Post` or a `Video` model, the default `commentable_type` would be either `App\Models\Post` or `App\Models\Video`, respectively. However, you may wish to decouple these values from your application's internal structure.

For example, instead of using the model names as the "type", we may use simple strings such as `post` and `video`. By doing so, the polymorphic "type" column values in our database will remain valid even if the models are renamed:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);
```

You may call the `enforceMorphMap` method in the `boot` method of your `App\Providers\AppServiceProvider` class or create a separate service provider if you wish.

You may determine the morph alias of a given model at runtime using the model's `getMorphClass` method. Conversely, you may determine the fully-qualified class name associated with a morph alias using the `Relation::getMorphedModel` method:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

$alias = $post->getMorphClass();

$class = Relation::getMorphedModel($alias);
```

> [!WARNING]
> When adding a "morph map" to your existing application, every morphable `*_type` column value in your database that still contains a fully-qualified class will need to be converted to its "map" name.

<a name="dynamic-relationships"></a>
### Dynamic Relationships

You may use the `resolveRelationUsing` method to define relations between Eloquent models at runtime. While not typically recommended for normal application development, this may occasionally be useful when developing Laravel packages.

The `resolveRelationUsing` method accepts the desired relationship name as its first argument. The second argument passed to the method should be a closure that accepts the model instance and returns a valid Eloquent relationship definition. Typically, you should configure dynamic relationships within the boot method of a [service provider](/docs/{{version}}/providers):

```php
use App\Models\Order;
use App\Models\Customer;

Order::resolveRelationUsing('customer', function (Order $orderModel) {
    return $orderModel->belongsTo(Customer::class, 'customer_id');
});
```

> [!WARNING]
> When defining dynamic relationships, always provide explicit key name arguments to the Eloquent relationship methods.

<a name="querying-relations"></a>
## Querying Relations

Since all Eloquent relationships are defined via methods, you may call those methods to obtain an instance of the relationship without actually executing a query to load the related models. In addition, all types of Eloquent relationships also serve as [query builders](/docs/{{version}}/queries), allowing you to continue to chain constraints onto the relationship query before finally executing the SQL query against your database.

For example, imagine a blog application in which a `User` model has many associated `Post` models:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get all of the posts for the user.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

You may query the `posts` relationship and add additional constraints to the relationship like so:

```php
use App\Models\User;

$user = User::find(1);

$user->posts()->where('active', 1)->get();
```

You are able to use any of the Laravel [query builder's](/docs/{{version}}/queries) methods on the relationship, so be sure to explore the query builder documentation to learn about all of the methods that are available to you.

<a name="chaining-orwhere-clauses-after-relationships"></a>
#### Chaining `orWhere` Clauses After Relationships

As demonstrated in the example above, you are free to add additional constraints to relationships when querying them. However, use caution when chaining `orWhere` clauses onto a relationship, as the `orWhere` clauses will be logically grouped at the same level as the relationship constraint:

```php
$user->posts()
    ->where('active', 1)
    ->orWhere('votes', '>=', 100)
    ->get();
```

The example above will generate the following SQL. As you can see, the `or` clause instructs the query to return _any_ post with greater than 100 votes. The query is no longer constrained to a specific user:

```sql
select *
from posts
where user_id = ? and active = 1 or votes >= 100
```

In most situations, you should use [logical groups](/docs/{{version}}/queries#logical-grouping) to group the conditional checks between parentheses:

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
    ->where(function (Builder $query) {
        return $query->where('active', 1)
            ->orWhere('votes', '>=', 100);
    })
    ->get();
```

The example above will produce the following SQL. Note that the logical grouping has properly grouped the constraints and the query remains constrained to a specific user:

```sql
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relationship-methods-vs-dynamic-properties"></a>
### Relationship Methods vs. Dynamic Properties

If you do not need to add additional constraints to an Eloquent relationship query, you may access the relationship as if it were a property. For example, continuing to use our `User` and `Post` example models, we may access all of a user's posts like so:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

Dynamic relationship properties perform "lazy loading", meaning they will only load their relationship data when you actually access them. Because of this, developers often use [eager loading](#eager-loading) to pre-load relationships they know will be accessed after loading the model. Eager loading provides a significant reduction in SQL queries that must be executed to load a model's relations.

<a name="querying-relationship-existence"></a>
### Querying Relationship Existence

When retrieving model records, you may wish to limit your results based on the existence of a relationship. For example, imagine you want to retrieve all blog posts that have at least one comment. To do so, you may pass the name of the relationship to the `has` and `orHas` methods:

```php
use App\Models\Post;

// Retrieve all posts that have at least one comment...
$posts = Post::has('comments')->get();
```

You may also specify an operator and count value to further customize the query:

```php
// Retrieve all posts that have three or more comments...
$posts = Post::has('comments', '>=', 3)->get();
```

Nested `has` statements may be constructed using "dot" notation. For example, you may retrieve all posts that have at least one comment that has at least one image:

```php
// Retrieve posts that have at least one comment with images...
$posts = Post::has('comments.images')->get();
```

If you need even more power, you may use the `whereHas` and `orWhereHas` methods to define additional query constraints on your `has` queries, such as inspecting the content of a comment:

```php
use Illuminate\Database\Eloquent\Builder;

// Retrieve posts with at least one comment containing words like code%...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();

// Retrieve posts with at least ten comments containing words like code%...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
}, '>=', 10)->get();
```

> [!WARNING]
> Eloquent does not currently support querying for relationship existence across databases. The relationships must exist within the same database.

<a name="many-to-many-relationship-existence-queries"></a>
#### Many to Many Relationship Existence Queries

The `whereAttachedTo` method may be used to query for models that have a many to many attachment to a model or collection of models:

```php
$users = User::whereAttachedTo($role)->get();
```

You may also provide a [collection](/docs/{{version}}/eloquent-collections) instance to the `whereAttachedTo` method. When doing so, Laravel will retrieve models that are attached to any of the models within the collection:

```php
$tags = Tag::whereLike('name', '%laravel%')->get();

$posts = Post::whereAttachedTo($tags)->get();
```

<a name="inline-relationship-existence-queries"></a>
#### Inline Relationship Existence Queries

If you would like to query for a relationship's existence with a single, simple where condition attached to the relationship query, you may find it more convenient to use the `whereRelation`, `orWhereRelation`, `whereMorphRelation`, and `orWhereMorphRelation` methods. For example, we may query for all posts that have unapproved comments:

```php
use App\Models\Post;

$posts = Post::whereRelation('comments', 'is_approved', false)->get();
```

Of course, like calls to the query builder's `where` method, you may also specify an operator:

```php
$posts = Post::whereRelation(
    'comments', 'created_at', '>=', now()->minus(hours: 1)
)->get();
```

<a name="querying-relationship-absence"></a>
### Querying Relationship Absence

When retrieving model records, you may wish to limit your results based on the absence of a relationship. For example, imagine you want to retrieve all blog posts that **don't** have any comments. To do so, you may pass the name of the relationship to the `doesntHave` and `orDoesntHave` methods:

```php
use App\Models\Post;

$posts = Post::doesntHave('comments')->get();
```

If you need even more power, you may use the `whereDoesntHave` and `orWhereDoesntHave` methods to add additional query constraints to your `doesntHave` queries, such as inspecting the content of a comment:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();
```

You may use "dot" notation to execute a query against a nested relationship. For example, the following query will retrieve all posts that do not have comments as well as posts that have comments where none of the comments are from banned users:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 1);
})->get();
```

<a name="querying-morph-to-relationships"></a>
### Querying Morph To Relationships

To query the existence of "morph to" relationships, you may use the `whereHasMorph` and `whereDoesntHaveMorph` methods. These methods accept the name of the relationship as their first argument. Next, the methods accept the names of the related models that you wish to include in the query. Finally, you may provide a closure which customizes the relationship query:

```php
use App\Models\Comment;
use App\Models\Post;
use App\Models\Video;
use Illuminate\Database\Eloquent\Builder;

// Retrieve comments associated to posts or videos with a title like code%...
$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();

// Retrieve comments associated to posts with a title not like code%...
$comments = Comment::whereDoesntHaveMorph(
    'commentable',
    Post::class,
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();
```

You may occasionally need to add query constraints based on the "type" of the related polymorphic model. The closure passed to the `whereHasMorph` method may receive a `$type` value as its second argument. This argument allows you to inspect the "type" of the query that is being built:

```php
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $query, string $type) {
        $column = $type === Post::class ? 'content' : 'title';

        $query->where($column, 'like', 'code%');
    }
)->get();
```

Sometimes you may want to query for the children of a "morph to" relationship's parent. You may accomplish this using the `whereMorphedTo` and `whereNotMorphedTo` methods, which will automatically determine the proper morph type mapping for the given model. These methods accept the name of the `morphTo` relationship as their first argument and the related parent model as their second argument:

```php
$comments = Comment::whereMorphedTo('commentable', $post)
    ->orWhereMorphedTo('commentable', $video)
    ->get();
```

<a name="querying-all-morph-to-related-models"></a>
#### Querying All Related Models

Instead of passing an array of possible polymorphic models, you may provide `*` as a wildcard value. This will instruct Laravel to retrieve all of the possible polymorphic types from the database. Laravel will execute an additional query in order to perform this operation:

```php
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph('commentable', '*', function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();
```

<a name="aggregating-related-models"></a>
## Aggregating Related Models

<a name="counting-related-models"></a>
### Counting Related Models

Sometimes you may want to count the number of related models for a given relationship without actually loading the models. To accomplish this, you may use the `withCount` method. The `withCount` method will place a `{relation}_count` attribute on the resulting models:

```php
use App\Models\Post;

$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

By passing an array to the `withCount` method, you may add the "counts" for multiple relations as well as add additional constraints to the queries:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'code%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

You may also alias the relationship count result, allowing multiple counts on the same relationship:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    },
])->get();

echo $posts[0]->comments_count;
echo $posts[0]->pending_comments_count;
```

<a name="deferred-count-loading"></a>
#### Deferred Count Loading

Using the `loadCount` method, you may load a relationship count after the parent model has already been retrieved:

```php
$book = Book::first();

$book->loadCount('genres');
```

If you need to set additional query constraints on the count query, you may pass an array keyed by the relationships you wish to count. The array values should be closures which receive the query builder instance:

```php
$book->loadCount(['reviews' => function (Builder $query) {
    $query->where('rating', 5);
}])
```

<a name="relationship-counting-and-custom-select-statements"></a>
#### Relationship Counting and Custom Select Statements

If you're combining `withCount` with a `select` statement, ensure that you call `withCount` after the `select` method:

```php
$posts = Post::select(['title', 'body'])
    ->withCount('comments')
    ->get();
```

<a name="other-aggregate-functions"></a>
### Other Aggregate Functions

In addition to the `withCount` method, Eloquent provides `withMin`, `withMax`, `withAvg`, `withSum`, and `withExists` methods. These methods will place a `{relation}_{function}_{column}` attribute on your resulting models:

```php
use App\Models\Post;

$posts = Post::withSum('comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->comments_sum_votes;
}
```

If you wish to access the result of the aggregate function using another name, you may specify your own alias:

```php
$posts = Post::withSum('comments as total_comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->total_comments;
}
```

Like the `loadCount` method, deferred versions of these methods are also available. These additional aggregate operations may be performed on Eloquent models that have already been retrieved:

```php
$post = Post::first();

$post->loadSum('comments', 'votes');
```

If you're combining these aggregate methods with a `select` statement, ensure that you call the aggregate methods after the `select` method:

```php
$posts = Post::select(['title', 'body'])
    ->withExists('comments')
    ->get();
```

<a name="counting-related-models-on-morph-to-relationships"></a>
### Counting Related Models on Morph To Relationships

If you would like to eager load a "morph to" relationship, as well as related model counts for the various entities that may be returned by that relationship, you may utilize the `with` method in combination with the `morphTo` relationship's `morphWithCount` method.

In this example, let's assume that `Photo` and `Post` models may create `ActivityFeed` models. We will assume the `ActivityFeed` model defines a "morph to" relationship named `parentable` that allows us to retrieve the parent `Photo` or `Post` model for a given `ActivityFeed` instance. Additionally, let's assume that `Photo` models "have many" `Tag` models and `Post` models "have many" `Comment` models.

Now, let's imagine we want to retrieve `ActivityFeed` instances and eager load the `parentable` parent models for each `ActivityFeed` instance. In addition, we want to retrieve the number of tags that are associated with each parent photo and the number of comments that are associated with each parent post:

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$activities = ActivityFeed::with([
    'parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWithCount([
            Photo::class => ['tags'],
            Post::class => ['comments'],
        ]);
    }])->get();
```

<a name="morph-to-deferred-count-loading"></a>
#### Deferred Count Loading

Let's assume we have already retrieved a set of `ActivityFeed` models and now we would like to load the nested relationship counts for the various `parentable` models associated with the activity feeds. You may use the `loadMorphCount` method to accomplish this:

```php
$activities = ActivityFeed::with('parentable')->get();

$activities->loadMorphCount('parentable', [
    Photo::class => ['tags'],
    Post::class => ['comments'],
]);
```

<a name="eager-loading"></a>
## Eager Loading

When accessing Eloquent relationships as properties, the related models are "lazy loaded". This means the relationship data is not actually loaded until you first access the property. However, Eloquent can "eager load" relationships at the time you query the parent model. Eager loading alleviates the "N + 1" query problem. To illustrate the N + 1 query problem, consider a `Book` model that "belongs to" to an `Author` model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * Get the author that wrote the book.
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }
}
```

Now, let's retrieve all books and their authors:

```php
use App\Models\Book;

$books = Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

This loop will execute one query to retrieve all of the books within the database table, then another query for each book in order to retrieve the book's author. So, if we have 25 books, the code above would run 26 queries: one for the original book, and 25 additional queries to retrieve the author of each book.

Thankfully, we can use eager loading to reduce this operation to just two queries. When building a query, you may specify which relationships should be eager loaded using the `with` method:

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

For this operation, only two queries will be executed - one query to retrieve all of the books and one query to retrieve all of the authors for all of the books:

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

<a name="eager-loading-multiple-relationships"></a>
#### Eager Loading Multiple Relationships

Sometimes you may need to eager load several different relationships. To do so, just pass an array of relationships to the `with` method:

```php
$books = Book::with(['author', 'publisher'])->get();
```

<a name="nested-eager-loading"></a>
#### Nested Eager Loading

To eager load a relationship's relationships, you may use "dot" syntax. For example, let's eager load all of the book's authors and all of the author's personal contacts:

```php
$books = Book::with('author.contacts')->get();
```

Alternatively, you may specify nested eager loaded relationships by providing a nested array to the `with` method, which can be convenient when eager loading multiple nested relationships:

```php
$books = Book::with([
    'author' => [
        'contacts',
        'publisher',
    ],
])->get();
```

<a name="nested-eager-loading-morphto-relationships"></a>
#### Nested Eager Loading `morphTo` Relationships

If you would like to eager load a `morphTo` relationship, as well as nested relationships on the various entities that may be returned by that relationship, you may use the `with` method in combination with the `morphTo` relationship's `morphWith` method. To help illustrate this method, let's consider the following model:

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * Get the parent of the activity feed record.
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

In this example, let's assume `Event`, `Photo`, and `Post` models may create `ActivityFeed` models. Additionally, let's assume that `Event` models belong to a `Calendar` model, `Photo` models are associated with `Tag` models, and `Post` models belong to an `Author` model.

Using these model definitions and relationships, we may retrieve `ActivityFeed` model instances and eager load all `parentable` models and their respective nested relationships:

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$activities = ActivityFeed::query()
    ->with(['parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);
    }])->get();
```

<a name="eager-loading-specific-columns"></a>
#### Eager Loading Specific Columns

You may not always need every column from the relationships you are retrieving. For this reason, Eloquent allows you to specify which columns of the relationship you would like to retrieve:

```php
$books = Book::with('author:id,name,book_id')->get();
```

> [!WARNING]
> When using this feature, you should always include the `id` column and any relevant foreign key columns in the list of columns you wish to retrieve.

<a name="eager-loading-by-default"></a>
#### Eager Loading by Default

Sometimes you might want to always load some relationships when retrieving a model. To accomplish this, you may define a `$with` property on the model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * The relationships that should always be loaded.
     *
     * @var array
     */
    protected $with = ['author'];

    /**
     * Get the author that wrote the book.
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }

    /**
     * Get the genre of the book.
     */
    public function genre(): BelongsTo
    {
        return $this->belongsTo(Genre::class);
    }
}
```

If you would like to remove an item from the `$with` property for a single query, you may use the `without` method:

```php
$books = Book::without('author')->get();
```

If you would like to override all items within the `$with` property for a single query, you may use the `withOnly` method:

```php
$books = Book::withOnly('genre')->get();
```

<a name="constraining-eager-loads"></a>
### Constraining Eager Loads

Sometimes you may wish to eager load a relationship but also specify additional query conditions for the eager loading query. You can accomplish this by passing an array of relationships to the `with` method where the array key is a relationship name and the array value is a closure that adds additional constraints to the eager loading query:

```php
use App\Models\User;

$users = User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%code%');
}])->get();
```

In this example, Eloquent will only eager load posts where the post's `title` column contains the word `code`. You may call other [query builder](/docs/{{version}}/queries) methods to further customize the eager loading operation:

```php
$users = User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

<a name="constraining-eager-loading-of-morph-to-relationships"></a>
#### Constraining Eager Loading of `morphTo` Relationships

If you are eager loading a `morphTo` relationship, Eloquent will run multiple queries to fetch each type of related model. You may add additional constraints to each of these queries using the `MorphTo` relation's `constrain` method:

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$comments = Comment::with(['commentable' => function (MorphTo $morphTo) {
    $morphTo->constrain([
        Post::class => function ($query) {
            $query->whereNull('hidden_at');
        },
        Video::class => function ($query) {
            $query->where('type', 'educational');
        },
    ]);
}])->get();
```

In this example, Eloquent will only eager load posts that have not been hidden and videos that have a `type` value of "educational".

<a name="constraining-eager-loads-with-relationship-existence"></a>
#### Constraining Eager Loads With Relationship Existence

You may sometimes find yourself needing to check for the existence of a relationship while simultaneously loading the relationship based on the same conditions. For example, you may wish to only retrieve `User` models that have child `Post` models matching a given query condition while also eager loading the matching posts. You may accomplish this using the `withWhereHas` method:

```php
use App\Models\User;

$users = User::withWhereHas('posts', function ($query) {
    $query->where('featured', true);
})->get();
```

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Sometimes you may need to eager load a relationship after the parent model has already been retrieved. For example, this may be useful if you need to dynamically decide whether to load related models:

```php
use App\Models\Book;

$books = Book::all();

if ($condition) {
    $books->load('author', 'publisher');
}
```

If you need to set additional query constraints on the eager loading query, you may pass an array keyed by the relationships you wish to load. The array values should be closure instances which receive the query instance:

```php
$author->load(['books' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

To load a relationship only when it has not already been loaded, use the `loadMissing` method:

```php
$book->loadMissing('author');
```

<a name="nested-lazy-eager-loading-morphto"></a>
#### Nested Lazy Eager Loading and `morphTo`

If you would like to eager load a `morphTo` relationship, as well as nested relationships on the various entities that may be returned by that relationship, you may use the `loadMorph` method.

This method accepts the name of the `morphTo` relationship as its first argument, and an array of model / relationship pairs as its second argument. To help illustrate this method, let's consider the following model:

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * Get the parent of the activity feed record.
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

In this example, let's assume `Event`, `Photo`, and `Post` models may create `ActivityFeed` models. Additionally, let's assume that `Event` models belong to a `Calendar` model, `Photo` models are associated with `Tag` models, and `Post` models belong to an `Author` model.

Using these model definitions and relationships, we may retrieve `ActivityFeed` model instances and eager load all `parentable` models and their respective nested relationships:

```php
$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);
```

<a name="automatic-eager-loading"></a>
### Automatic Eager Loading

> [!WARNING]
> This feature is currently in beta in order to gather community feedback. The behavior and functionality of this feature may change even on patch releases.

In many cases, Laravel can automatically eager load the relationships you access. To enable automatic eager loading, you should invoke the `Model::automaticallyEagerLoadRelationships` method within the `boot` method of your application's `AppServiceProvider`:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::automaticallyEagerLoadRelationships();
}
```

When this feature is enabled, Laravel will attempt to automatically load any relationships you access that have not been previously loaded. For example, consider the following scenario:

```php
use App\Models\User;

$users = User::all();

foreach ($users as $user) {
    foreach ($user->posts as $post) {
        foreach ($post->comments as $comment) {
            echo $comment->content;
        }
    }
}
```

Typically, the code above would execute a query for each user in order to retrieve their posts, as well as a query for each post to retrieve its comments. However, when the `automaticallyEagerLoadRelationships` feature has been enabled, Laravel will automatically [lazy eager load](#lazy-eager-loading) the posts for all users in the user collection when you attempt to access the posts on any of the retrieved users. Likewise, when you attempt to access the comments for any retrieved post, all comments will be lazy eager loaded for all posts that were originally retrieved.

If you do not want to globally enable automatic eager loading, you can still enable this feature for a single Eloquent collection instance by invoking the `withRelationshipAutoloading` method on the collection:

```php
$users = User::where('vip', true)->get();

return $users->withRelationshipAutoloading();
```

<a name="preventing-lazy-loading"></a>
### Preventing Lazy Loading

As previously discussed, eager loading relationships can often provide significant performance benefits to your application. Therefore, if you would like, you may instruct Laravel to always prevent the lazy loading of relationships. To accomplish this, you may invoke the `preventLazyLoading` method offered by the base Eloquent model class. Typically, you should call this method within the `boot` method of your application's `AppServiceProvider` class.

The `preventLazyLoading` method accepts an optional boolean argument that indicates if lazy loading should be prevented. For example, you may wish to only disable lazy loading in non-production environments so that your production environment will continue to function normally even if a lazy loaded relationship is accidentally present in production code:

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

After preventing lazy loading, Eloquent will throw a `Illuminate\Database\LazyLoadingViolationException` exception when your application attempts to lazy load any Eloquent relationship.

You may customize the behavior of lazy loading violations using the `handleLazyLoadingViolationsUsing` method. For example, using this method, you may instruct lazy loading violations to only be logged instead of interrupting the application's execution with exceptions:

```php
Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
    $class = $model::class;

    info("Attempted to lazy load [{$relation}] on model [{$class}].");
});
```

<a name="inserting-and-updating-related-models"></a>
## Inserting and Updating Related Models

<a name="the-save-method"></a>
### The `save` Method

Eloquent provides convenient methods for adding new models to relationships. For example, perhaps you need to add a new comment to a post. Instead of manually setting the `post_id` attribute on the `Comment` model you may insert the comment using the relationship's `save` method:

```php
use App\Models\Comment;
use App\Models\Post;

$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$post->comments()->save($comment);
```

Note that we did not access the `comments` relationship as a dynamic property. Instead, we called the `comments` method to obtain an instance of the relationship. The `save` method will automatically add the appropriate `post_id` value to the new `Comment` model.

If you need to save multiple related models, you may use the `saveMany` method:

```php
$post = Post::find(1);

$post->comments()->saveMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another new comment.']),
]);
```

The `save` and `saveMany` methods will persist the given model instances, but will not add the newly persisted models to any in-memory relationships that are already loaded onto the parent model. If you plan on accessing the relationship after using the `save` or `saveMany` methods, you may wish to use the `refresh` method to reload the model and its relationships:

```php
$post->comments()->save($comment);

$post->refresh();

// All comments, including the newly saved comment...
$post->comments;
```

<a name="the-push-method"></a>
#### Recursively Saving Models and Relationships

If you would like to `save` your model and all of its associated relationships, you may use the `push` method. In this example, the `Post` model will be saved as well as its comments and the comment's authors:

```php
$post = Post::find(1);

$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';

$post->push();
```

The `pushQuietly` method may be used to save a model and its associated relationships without raising any events:

```php
$post->pushQuietly();
```

<a name="the-create-method"></a>
### The `create` Method

In addition to the `save` and `saveMany` methods, you may also use the `create` method, which accepts an array of attributes, creates a model, and inserts it into the database. The difference between `save` and `create` is that `save` accepts a full Eloquent model instance while `create` accepts a plain PHP `array`. The newly created model will be returned by the `create` method:

```php
use App\Models\Post;

$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

You may use the `createMany` method to create multiple related models:

```php
$post = Post::find(1);

$post->comments()->createMany([
    ['message' => 'A new comment.'],
    ['message' => 'Another new comment.'],
]);
```

The `createQuietly` and `createManyQuietly` methods may be used to create a model(s) without dispatching any events:

```php
$user = User::find(1);

$user->posts()->createQuietly([
    'title' => 'Post title.',
]);

$user->posts()->createManyQuietly([
    ['title' => 'First post.'],
    ['title' => 'Second post.'],
]);
```

You may also use the `findOrNew`, `firstOrNew`, `firstOrCreate`, and `updateOrCreate` methods to [create and update models on relationships](/docs/{{version}}/eloquent#upserts).

> [!NOTE]
> Before using the `create` method, be sure to review the [mass assignment](/docs/{{version}}/eloquent#mass-assignment) documentation.

<a name="updating-belongs-to-relationships"></a>
### Belongs To Relationships

If you would like to assign a child model to a new parent model, you may use the `associate` method. In this example, the `User` model defines a `belongsTo` relationship to the `Account` model. This `associate` method will set the foreign key on the child model:

```php
use App\Models\Account;

$account = Account::find(10);

$user->account()->associate($account);

$user->save();
```

To remove a parent model from a child model, you may use the `dissociate` method. This method will set the relationship's foreign key to `null`:

```php
$user->account()->dissociate();

$user->save();
```

<a name="updating-many-to-many-relationships"></a>
### Many to Many Relationships

<a name="attaching-detaching"></a>
#### Attaching / Detaching

Eloquent also provides methods to make working with many-to-many relationships more convenient. For example, let's imagine a user can have many roles and a role can have many users. You may use the `attach` method to attach a role to a user by inserting a record in the relationship's intermediate table:

```php
use App\Models\User;

$user = User::find(1);

$user->roles()->attach($roleId);
```

When attaching a relationship to a model, you may also pass an array of additional data to be inserted into the intermediate table:

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

Sometimes it may be necessary to remove a role from a user. To remove a many-to-many relationship record, use the `detach` method. The `detach` method will delete the appropriate record out of the intermediate table; however, both models will remain in the database:

```php
// Detach a single role from the user...
$user->roles()->detach($roleId);

// Detach all roles from the user...
$user->roles()->detach();
```

For convenience, `attach` and `detach` also accept arrays of IDs as input:

```php
$user = User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires],
]);
```

<a name="syncing-associations"></a>
#### Syncing Associations

You may also use the `sync` method to construct many-to-many associations. The `sync` method accepts an array of IDs to place on the intermediate table. Any IDs that are not in the given array will be removed from the intermediate table. So, after this operation is complete, only the IDs in the given array will exist in the intermediate table:

```php
$user->roles()->sync([1, 2, 3]);
```

You may also pass additional intermediate table values with the IDs:

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

If you would like to insert the same intermediate table values with each of the synced model IDs, you may use the `syncWithPivotValues` method:

```php
$user->roles()->syncWithPivotValues([1, 2, 3], ['active' => true]);
```

If you do not want to detach existing IDs that are missing from the given array, you may use the `syncWithoutDetaching` method:

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

<a name="toggling-associations"></a>
#### Toggling Associations

The many-to-many relationship also provides a `toggle` method which "toggles" the attachment status of the given related model IDs. If the given ID is currently attached, it will be detached. Likewise, if it is currently detached, it will be attached:

```php
$user->roles()->toggle([1, 2, 3]);
```

You may also pass additional intermediate table values with the IDs:

```php
$user->roles()->toggle([
    1 => ['expires' => true],
    2 => ['expires' => true],
]);
```

<a name="transactional-pivot-operations"></a>
#### Transactional Pivot Operations

Each of the pivot operations discussed above also has an `OrFail` variant (`attachOrFail`, `detachOrFail`, `syncOrFail`, `syncWithoutDetachingOrFail`, and `toggleOrFail`) that wraps the operation within a database transaction, so that all changes are automatically rolled back if an exception is thrown:

```php
$user->roles()->attachOrFail([1, 2, 3]);

$user->roles()->syncOrFail([1, 2, 3]);
```

<a name="updating-a-record-on-the-intermediate-table"></a>
#### Updating a Record on the Intermediate Table

If you need to update an existing row in your relationship's intermediate table, you may use the `updateExistingPivot` method. This method accepts the intermediate record foreign key and an array of attributes to update:

```php
$user = User::find(1);

$user->roles()->updateExistingPivot($roleId, [
    'active' => false,
]);
```

<a name="touching-parent-timestamps"></a>
## Touching Parent Timestamps

When a model defines a `belongsTo` or `belongsToMany` relationship to another model, such as a `Comment` which belongs to a `Post`, it is sometimes helpful to update the parent's timestamp when the child model is updated.

For example, when a `Comment` model is updated, you may want to automatically "touch" the `updated_at` timestamp of the owning `Post` so that it is set to the current date and time. To accomplish this, you may use the `Touches` attribute on your child model containing the names of the relationships that should have their `updated_at` timestamps updated when the child model is updated:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Touches;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

#[Touches(['post'])]
class Comment extends Model
{
    /**
     * Get the post that the comment belongs to.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

> [!WARNING]
> Parent model timestamps will only be updated if the child model is updated using Eloquent's `save` method.

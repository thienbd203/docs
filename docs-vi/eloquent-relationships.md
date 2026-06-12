# Eloquent: Relationships

- [Giới thiệu](#introduction)
- [Định nghĩa Mối quan hệ](#defining-relationships)
    - [Một đến Một / Has One](#one-to-one)
    - [Một đến Nhiều / Has Many](#one-to-many)
    - [Một đến Nhiều (Nghịch đảo) / Belongs To](#one-to-many-inverse)
    - [Has One of Many](#has-one-of-many)
    - [Has One Through](#has-one-through)
    - [Has Many Through](#has-many-through)
- [Scoped Relationships](#scoped-relationships)
- [Mối quan hệ Nhiều đến Nhiều](#many-to-many)
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
## Giới thiệu

Các bảng database thường liên quan đến nhau. Ví dụ, một bài đăng blog có thể có nhiều bình luận hoặc một đơn hàng có thể liên quan đến người dùng đã đặt nó. Eloquent giúp việc quản lý và làm việc với các mối quan hệ này trở nên dễ dàng, và hỗ trợ nhiều loại mối quan hệ phổ biến:

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
## Định nghĩa Mối quan hệ

Các mối quan hệ Eloquent được định nghĩa là các phương thức trên các lớp model Eloquent của bạn. Vì các mối quan hệ cũng đóng vai trò là các [query builder](/docs/{{version}}/queries) mạnh mẽ, việc định nghĩa các mối quan hệ dưới dạng phương thức cung cấp khả năng chuỗi phương thức và truy vấn mạnh mẽ. Ví dụ, chúng ta có thể chuỗi các điều kiện truy vấn bổ sung vào mối quan hệ `posts` này:

```php
$user->posts()->where('active', 1)->get();
```

Tuy nhiên, trước khi đi sâu vào việc sử dụng các mối quan hệ, hãy tìm hiểu cách định nghĩa từng loại mối quan hệ được Eloquent hỗ trợ.

<a name="one-to-one"></a>
### Một đến Một / Has One

Mối quan hệ một-một là một loại mối quan hệ database rất cơ bản. Ví dụ, một model `User` có thể liên kết với một model `Phone`. Để định nghĩa mối quan hệ này, chúng ta sẽ đặt một phương thức `phone` trên model `User`. Phương thức `phone` nên gọi phương thức `hasOne` và trả về kết quả của nó. Phương thức `hasOne` có sẵn cho model của bạn thông qua lớp cơ sở `Illuminate\Database\Eloquent\Model` của model:

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

Đối số đầu tiên được truyền cho phương thức `hasOne` là tên của lớp model liên quan. Sau khi mối quan hệ được định nghĩa, chúng ta có thể truy xuất bản ghi liên quan bằng cách sử dụng các thuộc tính động của Eloquent. Các thuộc tính động cho phép bạn truy cập các phương thức mối quan hệ như thể chúng là các thuộc tính được định nghĩa trên model:

```php
$phone = User::find(1)->phone;
```

Eloquent xác định khóa ngoại của mối quan hệ dựa trên tên của model cha. Trong trường hợp này, model `Phone` được giả định tự động có một khóa ngoại `user_id`. Nếu bạn muốn ghi đè quy ước này, bạn có thể truyền đối số thứ hai cho phương thức `hasOne`:

```php
return $this->hasOne(Phone::class, 'foreign_key');
```

Ngoài ra, Eloquent giả định rằng khóa ngoại nên có một giá trị khớp với cột khóa chính của model cha. Nói cách khác, Eloquent sẽ tìm giá trị của cột `id` của người dùng trong cột `user_id` của bản ghi `Phone`. Nếu bạn muốn mối quan hệ sử dụng một giá trị khóa chính khác `id` hoặc khóa chính của model của bạn, bạn có thể truyền đối số thứ ba cho phương thức `hasOne`:

```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

<a name="one-to-one-defining-the-inverse-of-the-relationship"></a>
#### Định nghĩa Nghịch đảo của Mối quan hệ

Vì vậy, chúng ta có thể truy cập model `Phone` từ model `User` của mình. Tiếp theo, hãy định nghĩa một mối quan hệ trên model `Phone` cho phép chúng ta truy cập người dùng sở hữu điện thoại. Chúng ta có thể định nghĩa nghịch đảo của mối quan hệ `hasOne` bằng cách sử dụng phương thức `belongsTo`:

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

Khi gọi phương thức `user`, Eloquent sẽ cố gắng tìm một model `User` có một `id` khớp với cột `user_id` trên model `Phone`.

Eloquent xác định tên khóa ngoại bằng cách kiểm tra tên của phương thức mối quan hệ và thêm hậu tố `_id` vào tên phương thức. Vì vậy, trong trường hợp này, Eloquent giả định rằng model `Phone` có một cột `user_id`. Tuy nhiên, nếu khóa ngoại trên model `Phone` không phải là `user_id`, bạn có thể truyền tên khóa tùy chỉnh làm đối số thứ hai cho phương thức `belongsTo`:

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```

Nếu model cha không sử dụng `id` làm khóa chính của nó, hoặc bạn muốn tìm model liên quan bằng cách sử dụng một cột khác, bạn có thể truyền đối số thứ ba cho phương thức `belongsTo` chỉ định khóa tùy chỉnh của bảng cha:

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
### Một đến Nhiều / Has Many

Mối quan hệ một-nhiều được sử dụng để định nghĩa các mối quan hệ trong đó một model là cha của một hoặc nhiều model con. Ví dụ, một bài đăng blog có thể có vô số bình luận. Giống như tất cả các mối quan hệ Eloquent khác, các mối quan hệ một-nhiều được định nghĩa bằng cách định nghĩa một phương thức trên model Eloquent của bạn:

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

Hãy nhớ rằng, Eloquent sẽ tự động xác định cột khóa ngoại thích hợp cho model `Comment`. Theo quy ước, Eloquent sẽ lấy tên "snake case" của model cha và thêm hậu tố `_id`. Vì vậy, trong ví dụ này, Eloquent sẽ giả định cột khóa ngoại trên model `Comment` là `post_id`.

Sau khi phương thức mối quan hệ đã được định nghĩa, chúng ta có thể truy cập [collection](/docs/{{version}}/eloquent-collections) của các bình luận liên quan bằng cách truy cập thuộc tính `comments`. Hãy nhớ rằng, vì Eloquent cung cấp "các thuộc tính mối quan hệ động", chúng ta có thể truy cập các phương thức mối quan hệ như thể chúng được định nghĩa là các thuộc tính trên model:

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

Vì tất cả các mối quan hệ cũng đóng vai trò là query builder, bạn có thể thêm các ràng buộc thêm vào truy vấn mối quan hệ bằng cách gọi phương thức `comments` và tiếp tục chuỗi các điều kiện vào truy vấn:

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

Giống như phương thức `hasOne`, bạn cũng có thể ghi đè các khóa ngoại và cục bộ bằng cách truyền các đối số bổ sung cho phương thức `hasMany`:

```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

<a name="automatically-hydrating-parent-models-on-children"></a>
#### Tự động Hydrate Model Cha trên Model Con

Ngay cả khi sử dụng eager loading của Eloquent, các vấn đề truy vấn "N + 1" có thể phát sinh nếu bạn cố gắng truy cập model cha từ một model con trong khi lặp qua các model con:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

Trong ví dụ trên, một vấn đề truy vấn "N + 1" đã được đưa ra vì, mặc dù các bình luận đã được eager load cho mọi model `Post`, Eloquent không tự động hydrate model `Post` cha trên mỗi model `Comment` con.

Nếu bạn muốn Eloquent tự động hydrate các model cha vào các model con của chúng, bạn có thể gọi phương thức `chaperone` khi định nghĩa một mối quan hệ `hasMany`:

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

Hoặc, nếu bạn muốn opt-in vào việc hydrate cha tự động tại thời điểm chạy, bạn có thể gọi model `chaperone` khi eager load mối quan hệ:

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-to-many-inverse"></a>
### Một đến Nhiều (Nghịch đảo) / Belongs To

Bây giờ chúng ta có thể truy cập tất cả các bình luận của một bài đăng, hãy định nghĩa một mối quan hệ để cho phép một bình luận truy cập bài đăng cha của nó. Để định nghĩa nghịch đảo của mối quan hệ `hasMany`, hãy định nghĩa một phương thức mối quan hệ trên model con gọi phương thức `belongsTo`:

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

Sau khi mối quan hệ đã được định nghĩa, chúng ta có thể truy xuất bài đăng cha của một bình luận bằng cách truy cập thuộc tính "mối quan hệ động" `post`:

```php
use App\Models\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```

Trong ví dụ trên, Eloquent sẽ cố gắng tìm một model `Post` có một `id` khớp với cột `post_id` trên model `Comment`.

Eloquent xác định tên khóa ngoại mặc định bằng cách kiểm tra tên của phương thức mối quan hệ và thêm hậu tố một `_` theo sau là tên của cột khóa chính của model cha. Vì vậy, trong ví dụ này, Eloquent sẽ giả định rằng khóa ngoại của model `Post` trên bảng `comments` là `post_id`.

Tuy nhiên, nếu khóa ngoại cho mối quan hệ của bạn không tuân theo các quy ước này, bạn có thể truyền tên khóa ngoại tùy chỉnh làm đối số thứ hai cho phương thức `belongsTo`:

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```

Nếu model cha của bạn không sử dụng `id` làm khóa chính của nó, hoặc bạn muốn tìm model liên quan bằng cách sử dụng một cột khác, bạn có thể truyền đối số thứ ba cho phương thức `belongsTo` chỉ định khóa tùy chỉnh của bảng cha của bạn:

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
#### Model Mặc định

Các mối quan hệ `belongsTo`, `hasOne`, `hasOneThrough`, và `morphOne` cho phép bạn định nghĩa một model mặc định sẽ được trả về nếu mối quan hệ đã cho là `null`. Mẫu này thường được gọi là [Null Object pattern](https://en.wikipedia.org/wiki/Null_Object_pattern) và có thể giúp loại bỏ các kiểm tra điều kiện trong mã của bạn. Trong ví dụ sau, mối quan hệ `user` sẽ trả về một model `App\Models\User` trống nếu không có người dùng nào được gắn vào model `Post`:

```php
/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

Để điền model mặc định với các thuộc tính, bạn có thể truyền một mảng hoặc closure cho phương thức `withDefault`:

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
#### Truy vấn Mối quan hệ Belongs To

Khi truy vấn cho các con của một mối quan hệ "belongs to", bạn có thể xây dựng thủ công mệnh đề `where` để truy xuất các model Eloquent tương ứng:

```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```

Tuy nhiên, bạn có thể thấy việc sử dụng phương thức `whereBelongsTo` tiện lợi hơn, phương thức này sẽ tự động xác định mối quan hệ thích hợp và khóa ngoại cho model đã cho:

```php
$posts = Post::whereBelongsTo($user)->get();
```

Bạn cũng có thể cung cấp một instance [collection](/docs/{{version}}/eloquent-collections) cho phương thức `whereBelongsTo`. Khi làm như vậy, Laravel sẽ truy xuất các model thuộc về bất kỳ model cha nào trong collection:

```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```

Theo mặc định, Laravel sẽ xác định mối quan hệ liên kết với model đã cho dựa trên tên lớp của model; tuy nhiên, bạn có thể chỉ định tên mối quan hệ thủ công bằng cách cung cấp nó làm đối số thứ hai cho phương thức `whereBelongsTo`:

```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

<a name="has-one-of-many"></a>
### Has One of Many

Đôi khi một model có thể có nhiều model liên quan, nhưng bạn muốn dễ dàng truy xuất model liên quan "mới nhất" hoặc "cũ nhất" của mối quan hệ. Ví dụ, một model `User` có thể liên quan đến nhiều model `Order`, nhưng bạn muốn định nghĩa một cách thuận tiện để tương tác với đơn hàng gần nhất mà người dùng đã đặt. Bạn có thể thực hiện điều này bằng cách sử dụng loại mối quan hệ `hasOne` kết hợp với các phương thức `ofMany`:

```php
/**
 * Get the user's most recent order.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

Tương tự, bạn có thể định nghĩa một phương thức để truy xuất model liên quan "cũ nhất", hoặc đầu tiên, của một mối quan hệ:

```php
/**
 * Get the user's oldest order.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

Theo mặc định, các phương thức `latestOfMany` và `oldestOfMany` sẽ truy xuất model liên quan mới nhất hoặc cũ nhất dựa trên khóa chính của model, phải có thể sắp xếp được. Tuy nhiên, đôi khi bạn có thể muốn truy xuất một model duy nhất từ một mối quan hệ lớn hơn bằng cách sử dụng tiêu chí sắp xếp khác.

Ví dụ, sử dụng phương thức `ofMany`, bạn có thể truy xuất đơn hàng đắt nhất của người dùng. Phương thức `ofMany` chấp nhận cột có thể sắp xếp làm đối số đầu tiên và hàm tổng hợp nào (`min` hoặc `max`) để áp dụng khi truy vấn cho model liên quan:

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
> Vì PostgreSQL không hỗ trợ thực thi hàm `MAX` đối với các cột UUID, hiện không thể sử dụng các mối quan hệ one-of-many kết hợp với các cột UUID của PostgreSQL.

<a name="converting-many-relationships-to-has-one-relationships"></a>
#### Chuyển đổi Mối quan hệ "Many" thành Mối quan hệ Has One

Thường thì, khi truy xuất một model duy nhất bằng cách sử dụng các phương thức `latestOfMany`, `oldestOfMany`, hoặc `ofMany`, bạn đã có một mối quan hệ "has many" được định nghĩa cho cùng một model đó. Để thuận tiện, Laravel cho phép bạn dễ dàng chuyển đổi mối quan hệ này thành một mối quan hệ "has one" bằng cách gọi phương thức `one` trên mối quan hệ:

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

Bạn cũng có thể sử dụng phương thức `one` để chuyển đổi các mối quan hệ `HasManyThrough` thành các mối quan hệ `HasOneThrough`:

```php
public function latestDeployment(): HasOneThrough
{
    return $this->deployments()->one()->latestOfMany();
}
```

<a name="advanced-has-one-of-many-relationships"></a>
#### Mối quan hệ Has One of Many Nâng cao

Có thể xây dựng các mối quan hệ "has one of many" nâng cao hơn. Ví dụ, một model `Product` có thể có nhiều model `Price` liên quan được giữ lại trong hệ thống ngay cả sau khi giá mới được xuất bản. Ngoài ra, dữ liệu giá mới cho sản phẩm có thể được xuất bản trước để có hiệu lực vào một ngày trong tương lai thông qua một cột `published_at`.

Vì vậy, tóm lại, chúng ta cần truy xuất giá đã xuất bản mới nhất trong đó ngày xuất bản không phải trong tương lai. Ngoài ra, nếu hai giá có cùng ngày xuất bản, chúng ta sẽ ưu tiên giá có ID lớn nhất. Để thực hiện điều này, chúng ta phải truyền một mảng cho phương thức `ofMany` chứa các cột có thể sắp xếp xác định giá mới nhất. Ngoài ra, một closure sẽ được cung cấp làm đối số thứ hai cho phương thức `ofMany`. Closure này sẽ chịu trách nhiệm thêm các ràng buộc ngày xuất bản bổ sung vào truy vấn mối quan hệ:

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

Mối quan hệ "has-one-through" định nghĩa một mối quan hệ một-một với một model khác. Tuy nhiên, mối quan hệ này chỉ ra rằng model khai báo có thể được khớp với một instance của một model khác bằng cách đi _qua_ một model thứ ba.

Ví dụ, trong một ứng dụng cửa hàng sửa chữa xe, mỗi model `Mechanic` có thể liên kết với một model `Car`, và mỗi model `Car` có thể liên kết với một model `Owner`. Mặc dù thợ sửa chữa và chủ sở hữu không có mối quan hệ trực tiếp trong database, thợ sửa chữa có thể truy cập chủ sở hữu _qua_ model `Car`. Hãy xem các bảng cần thiết để định nghĩa mối quan hệ này:

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

Bây giờ chúng ta đã xem xét cấu trúc bảng cho mối quan hệ, hãy định nghĩa mối quan hệ trên model `Mechanic`:

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

Đối số đầu tiên được truyền cho phương thức `hasOneThrough` là tên của model cuối cùng chúng ta muốn truy cập, trong khi đối số thứ hai là tên của model trung gian.

Hoặc, nếu các mối quan hệ liên quan đã được định nghĩa trên tất cả các model tham gia vào mối quan hệ, bạn có thể định nghĩa trôi chảy một mối quan hệ "has-one-through" bằng cách gọi phương thức `through` và cung cấp tên của các mối quan hệ đó. Ví dụ, nếu model `Mechanic` có một mối quan hệ `cars` và model `Car` có một mối quan hệ `owner`, bạn có thể định nghĩa một mối quan hệ "has-one-through" kết nối thợ sửa chữa và chủ sở hữu như sau:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-one-through-key-conventions"></a>
#### Quy ước Khóa

Các quy ước khóa ngoại Eloquent điển hình sẽ được sử dụng khi thực hiện các truy vấn của mối quan hệ. Nếu bạn muốn tùy chỉnh các khóa của mối quan hệ, bạn có thể truyền chúng làm đối số thứ ba và thứ tư cho phương thức `hasOneThrough`. Đối số thứ ba là tên của khóa ngoại trên model trung gian. Đối số thứ tư là tên của khóa ngoại trên model cuối cùng. Đối số thứ năm là khóa cục bộ, trong khi đối số thứ sáu là khóa cục bộ của model trung gian:

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

Hoặc, như đã thảo luận trước đó, nếu các mối quan hệ liên quan đã được định nghĩa trên tất cả các model tham gia vào mối quan hệ, bạn có thể định nghĩa trôi chảy một mối quan hệ "has-one-through" bằng cách gọi phương thức `through` và cung cấp tên của các mối quan hệ đó. Cách tiếp cận này mang lại lợi ích của việc tái sử dụng các quy ước khóa đã được định nghĩa trên các mối quan hệ hiện có:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-many-through"></a>
### Has Many Through

Mối quan hệ "has-many-through" cung cấp một cách thuận tiện để truy cập các mối quan hệ xa thông qua một mối quan hệ trung gian. Ví dụ, hãy giả sử chúng ta đang xây dựng một nền tảng triển khai như [Laravel Cloud](https://cloud.laravel.com). Một model `Application` có thể truy cập nhiều model `Deployment` thông qua một model `Environment` trung gian. Sử dụng ví dụ này, bạn có thể dễ dàng thu thập tất cả các triển khai cho một ứng dụng đã cho. Hãy xem các bảng cần thiết để định nghĩa mối quan hệ này:

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

Bây giờ chúng ta đã xem xét cấu trúc bảng cho mối quan hệ, hãy định nghĩa mối quan hệ trên model `Application`:

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

Đối số đầu tiên được truyền cho phương thức `hasManyThrough` là tên của model cuối cùng chúng ta muốn truy cập, trong khi đối số thứ hai là tên của model trung gian.

Hoặc, nếu các mối quan hệ liên quan đã được định nghĩa trên tất cả các model tham gia vào mối quan hệ, bạn có thể định nghĩa trôi chảy một mối quan hệ "has-many-through" bằng cách gọi phương thức `through` và cung cấp tên của các mối quan hệ đó. Ví dụ, nếu model `Application` có một mối quan hệ `environments` và model `Environment` có một mối quan hệ `deployments`, bạn có thể định nghĩa một mối quan hệ "has-many-through" kết nối ứng dụng và các triển khai như sau:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

Mặc dù bảng của model `Deployment` không chứa cột `application_id`, mối quan hệ `hasManyThrough` cung cấp quyền truy cập vào các triển khai của một ứng dụng thông qua `$application->deployments`. Để truy xuất các model này, Eloquent kiểm tra cột `application_id` trên bảng của model `Environment` trung gian. Sau khi tìm thấy các ID môi trường liên quan, chúng được sử dụng để truy vấn bảng của model `Deployment`.

<a name="has-many-through-key-conventions"></a>
#### Quy ước Khóa

Các quy ước khóa ngoại Eloquent điển hình sẽ được sử dụng khi thực hiện các truy vấn của mối quan hệ. Nếu bạn muốn tùy chỉnh các khóa của mối quan hệ, bạn có thể truyền chúng làm đối số thứ ba và thứ tư cho phương thức `hasManyThrough`. Đối số thứ ba là tên của khóa ngoại trên model trung gian. Đối số thứ tư là tên của khóa ngoại trên model cuối cùng. Đối số thứ năm là khóa cục bộ, trong khi đối số thứ sáu là khóa cục bộ của model trung gian:

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

Hoặc, như đã thảo luận trước đó, nếu các mối quan hệ liên quan đã được định nghĩa trên tất cả các model tham gia vào mối quan hệ, bạn có thể định nghĩa trôi chảy một mối quan hệ "has-many-through" bằng cách gọi phương thức `through` và cung cấp tên của các mối quan hệ đó. Cách tiếp cận này mang lại lợi ích của việc tái sử dụng các quy ước khóa đã được định nghĩa trên các mối quan hệ hiện có:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

<a name="scoped-relationships"></a>
### Scoped Relationships

Rất phổ biến khi thêm các phương thức bổ sung vào các model để ràng buộc các mối quan hệ. Ví dụ, bạn có thể thêm một phương thức `featuredPosts` vào model `User` ràng buộc mối quan hệ `posts` rộng hơn với một ràng buộc `where` bổ sung:

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

Tuy nhiên, nếu bạn cố gắng tạo một model thông qua phương thức `featuredPosts`, thuộc tính `featured` của nó sẽ không được đặt thành `true`. Nếu bạn muốn tạo các model thông qua các phương thức mối quan hệ và cũng chỉ định các thuộc tính nên được thêm vào tất cả các model được tạo thông qua mối quan hệ đó, bạn có thể sử dụng phương thức `withAttributes` khi xây dựng truy vấn mối quan hệ:

```php
/**
 * Get the user's featured posts.
 */
public function featuredPosts(): HasMany
{
    return $this->posts()->withAttributes(['featured' => true]);
}
```

Phương thức `withAttributes` sẽ thêm các điều kiện `where` vào truy vấn bằng cách sử dụng các thuộc tính đã cho, và nó cũng sẽ thêm các thuộc tính đã cho vào bất kỳ model nào được tạo thông qua phương thức mối quan hệ:

```php
$post = $user->featuredPosts()->create(['title' => 'Featured Post']);

$post->featured; // true
```

Để hướng dẫn phương thức `withAttributes` không thêm các điều kiện `where` vào truy vấn, bạn có thể đặt đối số `asConditions` thành `false`:

```php
return $this->posts()->withAttributes(['featured' => true], asConditions: false);
```

<a name="many-to-many"></a>
## Mối quan hệ Nhiều đến Nhiều

Các mối quan hệ nhiều-đến-nhiều phức tạp hơn một chút so với các mối quan hệ `hasOne` và `hasMany`. Một ví dụ về mối quan hệ nhiều-đến-nhiều là một người dùng có nhiều vai trò và các vai trò đó cũng được chia sẻ bởi những người dùng khác trong ứng dụng. Ví dụ, một người dùng có thể được gán vai trò "Author" và "Editor"; tuy nhiên, các vai trò đó cũng có thể được gán cho những người dùng khác. Vì vậy, một người dùng có nhiều vai trò và một vai trò có nhiều người dùng.

<a name="many-to-many-table-structure"></a>
#### Cấu trúc Bảng

Để định nghĩa mối quan hệ này, cần có ba bảng database: `users`, `roles`, và `role_user`. Bảng `role_user` được lấy từ thứ tự bảng chữ cái của các tên model liên quan và chứa các cột `user_id` và `role_id`. Bảng này được sử dụng làm bảng trung gian kết nối người dùng và vai trò.

Hãy nhớ rằng, vì một vai trò có thể thuộc về nhiều người dùng, chúng ta không thể đơn giản đặt một cột `user_id` trên bảng `roles`. Điều này sẽ có nghĩa là một vai trò chỉ có thể thuộc về một người dùng duy nhất. Để cung cấp hỗ trợ cho việc gán vai trò cho nhiều người dùng, bảng `role_user` là cần thiết. Chúng ta có thể tóm tắt cấu trúc bảng của mối quan hệ như sau:

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
#### Cấu trúc Model

Các mối quan hệ nhiều-đến-nhiều được định nghĩa bằng cách viết một phương thức trả về kết quả của phương thức `belongsToMany`. Phương thức `belongsToMany` được cung cấp bởi lớp cơ sở `Illuminate\Database\Eloquent\Model` được sử dụng bởi tất cả các model Eloquent của ứng dụng của bạn. Ví dụ, hãy định nghĩa một phương thức `roles` trên model `User` của chúng ta. Đối số đầu tiên được truyền cho phương thức này là tên của lớp model liên quan:

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

Sau khi mối quan hệ được định nghĩa, bạn có thể truy cập các vai trò của người dùng bằng cách sử dụng thuộc tính mối quan hệ động `roles`:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    // ...
}
```

Vì tất cả các mối quan hệ cũng đóng vai trò là query builder, bạn có thể thêm các ràng buộc thêm vào truy vấn mối quan hệ bằng cách gọi phương thức `roles` và tiếp tục chuỗi các điều kiện vào truy vấn:

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

Để xác định tên bảng của bảng trung gian của mối quan hệ, Eloquent sẽ nối hai tên model liên quan theo thứ tự bảng chữ cái. Tuy nhiên, bạn có thể ghi đè quy ước này. Bạn có thể làm như vậy bằng cách truyền đối số thứ hai cho phương thức `belongsToMany`:

```php
return $this->belongsToMany(Role::class, 'role_user');
```

Ngoài việc tùy chỉnh tên của bảng trung gian, bạn cũng có thể tùy chỉnh tên cột của các khóa trên bảng bằng cách truyền các đối số bổ sung cho phương thức `belongsToMany`. Đối số thứ ba là tên khóa ngoại của model mà bạn đang định nghĩa mối quan hệ, trong khi đối số thứ tư là tên khóa ngoại của model mà bạn đang kết nối tới:

```php
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

<a name="many-to-many-defining-the-inverse-of-the-relationship"></a>
#### Định nghĩa Nghịch đảo của Mối quan hệ

Để định nghĩa "nghịch đảo" của một mối quan hệ nhiều-đến-nhiều, bạn nên định nghĩa một phương thức trên model liên quan cũng trả về kết quả của phương thức `belongsToMany`. Để hoàn thành ví dụ người dùng / vai trò của chúng ta, hãy định nghĩa phương thức `users` trên model `Role`:

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

Như bạn có thể thấy, mối quan hệ được định nghĩa chính xác giống như đối tác model `User` của nó với ngoại lệ là tham chiếu đến model `App\Models\User`. Vì chúng ta đang tái sử dụng phương thức `belongsToMany`, tất cả các tùy chọn tùy chỉnh bảng và khóa thông thường đều có sẵn khi định nghĩa "nghịch đảo" của các mối quan hệ nhiều-đến-nhiều.

<a name="retrieving-intermediate-table-columns"></a>
### Retrieving Intermediate Table Columns

Như bạn đã học, làm việc với các mối quan hệ nhiều-đến-nhiều đòi hỏi sự hiện diện của một bảng trung gian. Eloquent cung cấp một số cách rất hữu ích để tương tác với bảng này. Ví dụ, hãy giả sử model `User` của chúng ta có nhiều model `Role` liên quan. Sau khi truy cập mối quan hệ này, chúng ta có thể truy cập bảng trung gian bằng cách sử dụng thuộc tính `pivot` trên các model:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

Lưu ý rằng mỗi model `Role` chúng ta truy xuất được tự động gán một thuộc tính `pivot`. Thuộc tính này chứa một model đại diện cho bảng trung gian.

Theo mặc định, chỉ có các khóa của model sẽ có trên model `pivot`. Nếu bảng trung gian của bạn chứa các thuộc tính bổ sung, bạn phải chỉ định chúng khi định nghĩa mối quan hệ:

```php
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');
```

Nếu bạn muốn bảng trung gian của mình có các timestamp `created_at` và `updated_at` được duy trì tự động bởi Eloquent, hãy gọi phương thức `withTimestamps` khi định nghĩa mối quan hệ:

```php
return $this->belongsToMany(Role::class)->withTimestamps();
```

> [!WARNING]
> Các bảng trung gian sử dụng các timestamp được duy trì tự động của Eloquent được yêu cầu phải có cả hai cột timestamp `created_at` và `updated_at`.

<a name="customizing-the-pivot-attribute-name"></a>
#### Tùy chỉnh Tên Thuộc tính `pivot`

Như đã lưu ý trước đó, các thuộc tính từ bảng trung gian có thể được truy cập trên các model thông qua thuộc tính `pivot`. Tuy nhiên, bạn có thể tùy chỉnh tên của thuộc tính này để phản ánh tốt hơn mục đích của nó trong ứng dụng của bạn.

Ví dụ, nếu ứng dụng của bạn chứa người dùng có thể đăng ký podcast, bạn có thể có một mối quan hệ nhiều-đến-nhiều giữa người dùng và podcast. Nếu đây là trường hợp, bạn có thể muốn đổi tên thuộc tính bảng trung gian của mình thành `subscription` thay vì `pivot`. Điều này có thể được thực hiện bằng cách sử dụng phương thức `as` khi định nghĩa mối quan hệ:

```php
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();
```

Sau khi thuộc tính bảng trung gian tùy chỉnh đã được chỉ định, bạn có thể truy cập dữ liệu bảng trung gian bằng cách sử dụng tên tùy chỉnh:

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

<a name="filtering-queries-via-intermediate-table-columns"></a>
### Filtering Queries via Intermediate Table Columns

Bạn cũng có thể lọc các kết quả được trả về bởi các truy vấn mối quan hệ `belongsToMany` bằng cách sử dụng các phương thức `wherePivot`, `wherePivotIn`, `wherePivotNotIn`, `wherePivotBetween`, `wherePivotNotBetween`, `wherePivotNull`, và `wherePivotNotNull` khi định nghĩa mối quan hệ:

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

`wherePivot` thêm một ràng buộc mệnh đề where vào truy vấn, nhưng không thêm giá trị đã chỉ định khi tạo các model mới thông qua mối quan hệ đã định nghĩa. Nếu bạn cần cả truy vấn và tạo các mối quan hệ với một giá trị pivot cụ thể, bạn có thể sử dụng phương thức `withPivotValue`:

```php
return $this->belongsToMany(Role::class)
    ->withPivotValue('approved', 1);
```

<a name="ordering-queries-via-intermediate-table-columns"></a>
### Ordering Queries via Intermediate Table Columns

Bạn có thể sắp xếp các kết quả được trả về bởi các truy vấn mối quan hệ `belongsToMany` bằng cách sử dụng các phương thức `orderByPivot` và `orderByPivotDesc`. Trong ví dụ sau, chúng ta sẽ truy xuất tất cả các huy hiệu mới nhất cho người dùng:

```php
return $this->belongsToMany(Badge::class)
    ->where('rank', 'gold')
    ->orderByPivotDesc('created_at');
```

<a name="defining-custom-intermediate-table-models"></a>
### Defining Custom Intermediate Table Models

Nếu bạn muốn định nghĩa một model tùy chỉnh để đại diện cho bảng trung gian của mối quan hệ nhiều-đến-nhiều của mình, bạn có thể gọi phương thức `using` khi định nghĩa mối quan hệ. Các model pivot tùy chỉnh cho bạn cơ hội định nghĩa hành vi bổ sung trên model pivot, chẳng hạn như các phương thức và casts.

Các model pivot nhiều-đến-nhiều tùy chỉnh nên mở rộng lớp `Illuminate\Database\Eloquent\Relations\Pivot`, trong khi các model pivot nhiều-đến-nhiều polymorphic tùy chỉnh nên mở rộng lớp `Illuminate\Database\Eloquent\Relations\MorphPivot`. Ví dụ, chúng ta có thể định nghĩa một model `Role` sử dụng một model pivot `RoleUser` tùy chỉnh:

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

Khi định nghĩa model `RoleUser`, bạn nên mở rộng lớp `Illuminate\Database\Eloquent\Relations\Pivot`:

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
> Các model pivot không thể sử dụng trait `SoftDeletes`. Nếu bạn cần soft delete các bản ghi pivot, hãy cân nhắc chuyển đổi model pivot của bạn thành một model Eloquent thực tế.

<a name="custom-pivot-models-and-incrementing-ids"></a>
#### Custom Pivot Models and Incrementing IDs

Nếu bạn đã định nghĩa một mối quan hệ nhiều-đến-nhiều sử dụng một model pivot tùy chỉnh, và model pivot đó có một khóa chính tự tăng, bạn nên đảm bảo lớp model pivot tùy chỉnh của bạn sử dụng thuộc tính `Table` với `incrementing` được đặt thành `true`:

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

Mối quan hệ polymorphic cho phép model con thuộc về nhiều hơn một loại model bằng cách sử dụng một liên kết duy nhất. Ví dụ, hãy tưởng tượng bạn đang xây dựng một ứng dụng cho phép người dùng chia sẻ bài đăng blog và video. Trong một ứng dụng như vậy, một model `Comment` có thể thuộc về cả model `Post` và `Video`.

<a name="one-to-one-polymorphic-relations"></a>
### One to One (Polymorphic)

<a name="one-to-one-polymorphic-table-structure"></a>
#### Cấu trúc Bảng

Mối quan hệ một-một polymorphic tương tự như một mối quan hệ một-một điển hình; tuy nhiên, model con có thể thuộc về nhiều hơn một loại model bằng cách sử dụng một liên kết duy nhất. Ví dụ, một bài đăng blog `Post` và một `User` có thể chia sẻ một mối quan hệ polymorphic với một model `Image`. Sử dụng một mối quan hệ một-một polymorphic cho phép bạn có một bảng duy nhất của các hình ảnh duy nhất có thể được liên kết với bài đăng và người dùng. Trước tiên, hãy xem cấu trúc bảng:

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

Lưu ý các cột `imageable_id` và `imageable_type` trên bảng `images`. Cột `imageable_id` sẽ chứa giá trị ID của bài đăng hoặc người dùng, trong khi cột `imageable_type` sẽ chứa tên lớp của model cha. Cột `imageable_type` được Eloquent sử dụng để xác định "loại" model cha nào sẽ trả về khi truy cập mối quan hệ `imageable`. Trong trường hợp này, cột sẽ chứa `App\Models\Post` hoặc `App\Models\User`.

<a name="one-to-one-polymorphic-model-structure"></a>
#### Cấu trúc Model

Tiếp theo, hãy xem các định nghĩa model cần thiết để xây dựng mối quan hệ này:

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

Sau khi bảng database và các model của bạn được định nghĩa, bạn có thể truy cập các mối quan hệ thông qua các model của bạn. Ví dụ, để truy xuất hình ảnh cho một bài đăng, chúng ta có thể truy cập thuộc tính mối quan hệ động `image`:

```php
use App\Models\Post;

$post = Post::find(1);

$image = $post->image;
```

Bạn có thể truy xuất cha của model polymorphic bằng cách truy cập tên của phương thức thực hiện lệnh gọi đến `morphTo`. Trong trường hợp này, đó là phương thức `imageable` trên model `Image`. Vì vậy, chúng ta sẽ truy cập phương thức đó như một thuộc tính mối quan hệ động:

```php
use App\Models\Image;

$image = Image::find(1);

$imageable = $image->imageable;
```

Mối quan hệ `imageable` trên model `Image` sẽ trả về một instance `Post` hoặc `User`, tùy thuộc vào loại model nào sở hữu hình ảnh.

<a name="morph-one-to-one-key-conventions"></a>
#### Key Conventions

Nếu cần thiết, bạn có thể chỉ định tên của các cột "id" và "type" được sử dụng bởi model con polymorphic của bạn. Nếu bạn làm như vậy, hãy đảm bảo rằng bạn luôn truyền tên của mối quan hệ làm đối số đầu tiên cho phương thức `morphTo`. Thông thường, giá trị này nên khớp với tên phương thức, vì vậy bạn có thể sử dụng hằng số `__FUNCTION__` của PHP:

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
#### Cấu trúc Bảng

Mối quan hệ một-nhiều polymorphic tương tự như một mối quan hệ một-nhiều điển hình; tuy nhiên, model con có thể thuộc về nhiều hơn một loại model bằng cách sử dụng một liên kết duy nhất. Ví dụ, hãy tưởng tượng người dùng của ứng dụng của bạn có thể "bình luận" về bài đăng và video. Sử dụng các mối quan hệ polymorphic, bạn có thể sử dụng một bảng `comments` duy nhất để chứa bình luận cho cả bài đăng và video. Trước tiên, hãy xem cấu trúc bảng cần thiết để xây dựng mối quan hệ này:

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
#### Cấu trúc Model

Tiếp theo, hãy xem các định nghĩa model cần thiết để xây dựng mối quan hệ này:

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

Sau khi bảng database và các model của bạn được định nghĩa, bạn có thể truy cập các mối quan hệ thông qua các thuộc tính mối quan hệ động của model. Ví dụ, để truy cập tất cả các bình luận cho một bài đăng, chúng ta có thể sử dụng thuộc tính động `comments`:

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->comments as $comment) {
    // ...
}
```

Bạn cũng có thể truy xuất cha của một model con polymorphic bằng cách truy cập tên của phương thức thực hiện lệnh gọi đến `morphTo`. Trong trường hợp này, đó là phương thức `commentable` trên model `Comment`. Vì vậy, chúng ta sẽ truy cập phương thức đó như một thuộc tính mối quan hệ động để truy cập model cha của bình luận:

```php
use App\Models\Comment;

$comment = Comment::find(1);

$commentable = $comment->commentable;
```

Mối quan hệ `commentable` trên model `Comment` sẽ trả về một instance `Post` hoặc `Video`, tùy thuộc vào loại model nào là cha của bình luận.

<a name="polymorphic-automatically-hydrating-parent-models-on-children"></a>
#### Automatically Hydrating Parent Models on Children

Ngay cả khi sử dụng eager loading của Eloquent, các vấn đề truy vấn "N + 1" có thể phát sinh nếu bạn cố gắng truy cập model cha từ một model con trong khi lặp qua các model con:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->commentable->title;
    }
}
```

Trong ví dụ trên, một vấn đề truy vấn "N + 1" đã được đưa ra vì, mặc dù các bình luận đã được eager load cho mọi model `Post`, Eloquent không tự động hydrate model `Post` cha trên mỗi model `Comment` con.

Nếu bạn muốn Eloquent tự động hydrate các model cha vào các model con của chúng, bạn có thể gọi phương thức `chaperone` khi định nghĩa một mối quan hệ `morphMany`:

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

Hoặc, nếu bạn muốn opt-in vào việc hydrate cha tự động tại thời điểm chạy, bạn có thể gọi model `chaperone` khi eager load mối quan hệ:

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-of-many-polymorphic-relations"></a>
### One of Many (Polymorphic)

Đôi khi một model có thể có nhiều model liên quan, nhưng bạn muốn dễ dàng truy xuất model liên quan "mới nhất" hoặc "cũ nhất" của mối quan hệ. Ví dụ, một model `User` có thể liên quan đến nhiều model `Image`, nhưng bạn muốn định nghĩa một cách thuận tiện để tương tác với hình ảnh gần nhất mà người dùng đã tải lên. Bạn có thể thực hiện điều này bằng cách sử dụng loại mối quan hệ `morphOne` kết hợp với các phương thức `ofMany`:

```php
/**
 * Get the user's most recent image.
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}
```

Tương tự, bạn có thể định nghĩa một phương thức để truy xuất model liên quan "cũ nhất", hoặc đầu tiên, của một mối quan hệ:

```php
/**
 * Get the user's oldest image.
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}
```

Theo mặc định, các phương thức `latestOfMany` và `oldestOfMany` sẽ truy xuất model liên quan mới nhất hoặc cũ nhất dựa trên khóa chính của model, phải có thể sắp xếp được. Tuy nhiên, đôi khi bạn có thể muốn truy xuất một model duy nhất từ một mối quan hệ lớn hơn bằng cách sử dụng tiêu chí sắp xếp khác.

Ví dụ, sử dụng phương thức `ofMany`, bạn có thể truy xuất hình ảnh được "thích" nhất của người dùng. Phương thức `ofMany` chấp nhận cột có thể sắp xếp làm đối số đầu tiên và hàm tổng hợp nào (`min` hoặc `max`) để áp dụng khi truy vấn cho model liên quan:

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
> Có thể xây dựng các mối quan hệ "one of many" nâng cao hơn. Để biết thêm thông tin, vui lòng tham khảo [tài liệu has one of many](#advanced-has-one-of-many-relationships).

<a name="many-to-many-polymorphic-relations"></a>
### Many to Many (Polymorphic)

<a name="many-to-many-polymorphic-table-structure"></a>
#### Cấu trúc Bảng

Các mối quan hệ nhiều-đến-nhiều polymorphic phức tạp hơn một chút so với các mối quan hệ "morph one" và "morph many". Ví dụ, một model `Post` và model `Video` có thể chia sẻ một mối quan hệ polymorphic với một model `Tag`. Sử dụng một mối quan hệ nhiều-đến-nhiều polymorphic trong tình huống này sẽ cho phép ứng dụng của bạn có một bảng duy nhất của các thẻ duy nhất có thể được liên kết với bài đăng hoặc video. Trước tiên, hãy xem cấu trúc bảng cần thiết để xây dựng mối quan hệ này:

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
> Trước khi đi sâu vào các mối quan hệ nhiều-đến-nhiều polymorphic, bạn có thể hưởng lợi từ việc đọc tài liệu về các mối quan hệ nhiều-đến-nhiều điển hình [many-to-many relationships](#many-to-many).

<a name="many-to-many-polymorphic-model-structure"></a>
#### Cấu trúc Model

Tiếp theo, chúng ta đã sẵn sàng để định nghĩa các mối quan hệ trên các model. Các model `Post` và `Video` sẽ đều chứa một phương thức `tags` gọi phương thức `morphToMany` được cung cấp bởi lớp model Eloquent cơ sở.

Phương thức `morphToMany` chấp nhận tên của model liên quan cũng như "tên mối quan hệ". Dựa trên tên chúng ta gán cho tên bảng trung gian của chúng ta và các khóa nó chứa, chúng ta sẽ gọi mối quan hệ là "taggable":

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
#### Định nghĩa Nghịch đảo của Mối quan hệ

Tiếp theo, trên model `Tag`, bạn nên định nghĩa một phương thức cho mỗi model cha có thể của nó. Vì vậy, trong ví dụ này, chúng ta sẽ định nghĩa một phương thức `posts` và một phương thức `videos`. Cả hai phương thức này nên trả về kết quả của phương thức `morphedByMany`.

Phương thức `morphedByMany` chấp nhận tên của model liên quan cũng như "tên mối quan hệ". Dựa trên tên chúng ta gán cho tên bảng trung gian của chúng ta và các khóa nó chứa, chúng ta sẽ gọi mối quan hệ là "taggable":

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

Sau khi bảng database và các model của bạn được định nghĩa, bạn có thể truy cập các mối quan hệ thông qua các model của bạn. Ví dụ, để truy cập tất cả các thẻ cho một bài đăng, bạn có thể sử dụng thuộc tính mối quan hệ động `tags`:

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}
```

Bạn có thể truy xuất cha của một mối quan hệ polymorphic từ model con polymorphic bằng cách truy cập tên của phương thức thực hiện lệnh gọi đến `morphedByMany`. Trong trường hợp này, đó là các phương thức `posts` hoặc `videos` trên model `Tag`:

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

Theo mặc định, Laravel sẽ sử dụng tên lớp đầy đủ để lưu trữ "loại" của model liên quan. Ví dụ, với ví dụ mối quan hệ một-nhiều ở trên trong đó một model `Comment` có thể thuộc về một model `Post` hoặc `Video`, `commentable_type` mặc định sẽ là `App\Models\Post` hoặc `App\Models\Video`, tương ứng. Tuy nhiên, bạn có thể muốn tách rời các giá trị này khỏi cấu trúc nội bộ của ứng dụng của bạn.

Ví dụ, thay vì sử dụng tên model làm "loại", chúng ta có thể sử dụng các chuỗi đơn giản như `post` và `video`. Bằng cách làm như vậy, các giá trị cột "loại" polymorphic trong database của chúng ta sẽ vẫn hợp lệ ngay cả khi các model được đổi tên:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);
```

Bạn có thể gọi phương thức `enforceMorphMap` trong phương thức `boot` của lớp `App\Providers\AppServiceProvider` của bạn hoặc tạo một service provider riêng biệt nếu bạn muốn.

Bạn có thể xác định bí danh morph của một model đã cho tại thời điểm chạy bằng cách sử dụng phương thức `getMorphClass` của model. Ngược lại, bạn có thể xác định tên lớp đầy đủ được liên kết với một bí danh morph bằng cách sử dụng phương thức `Relation::getMorphedModel`:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

$alias = $post->getMorphClass();

$class = Relation::getMorphedModel($alias);
```

> [!WARNING]
> Khi thêm một "morph map" vào ứng dụng hiện có của bạn, mọi giá trị cột `*_type` có thể morph trong database của bạn vẫn chứa một lớp đầy đủ sẽ cần được chuyển đổi thành tên "map" của nó.

<a name="dynamic-relationships"></a>
### Dynamic Relationships

Bạn có thể sử dụng phương thức `resolveRelationUsing` để định nghĩa các mối quan hệ giữa các model Eloquent tại thời điểm chạy. Mặc dù thường không được khuyến nghị cho phát triển ứng dụng bình thường, điều này có thể đôi khi hữu ích khi phát triển các gói Laravel.

Phương thức `resolveRelationUsing` chấp nhận tên mối quan hệ mong muốn làm đối số đầu tiên. Đối số thứ hai được truyền cho phương thức nên là một closure chấp nhận instance model và trả về một định nghĩa mối quan hệ Eloquent hợp lệ. Thông thường, bạn nên cấu hình các mối quan hệ động trong phương thức boot của một [service provider](/docs/{{version}}/providers):

```php
use App\Models\Order;
use App\Models\Customer;

Order::resolveRelationUsing('customer', function (Order $orderModel) {
    return $orderModel->belongsTo(Customer::class, 'customer_id');
});
```

> [!WARNING]
> Khi định nghĩa các mối quan hệ động, luôn cung cấp các đối số tên khóa rõ ràng cho các phương thức mối quan hệ Eloquent.

<a name="querying-relations"></a>
## Querying Relations

Vì tất cả các mối quan hệ Eloquent được định nghĩa thông qua các phương thức, bạn có thể gọi các phương thức đó để lấy một instance của mối quan hệ mà không thực sự thực thi một truy vấn để tải các model liên quan. Ngoài ra, tất cả các loại mối quan hệ Eloquent cũng đóng vai trò là [query builders](/docs/{{version}}/queries), cho phép bạn tiếp tục chuỗi các ràng buộc vào truy vấn mối quan hệ trước khi cuối cùng thực thi truy vấn SQL đối với database của bạn.

Ví dụ, hãy tưởng tượng một ứng dụng blog trong đó một model `User` có nhiều model `Post` liên quan:

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

Bạn có thể truy vấn mối quan hệ `posts` và thêm các ràng buộc bổ sung vào mối quan hệ như sau:

```php
use App\Models\User;

$user = User::find(1);

$user->posts()->where('active', 1)->get();
```

Bạn có thể sử dụng bất kỳ phương thức nào của [query builder](/docs/{{version}}/queries) của Laravel trên mối quan hệ, vì vậy hãy đảm bảo khám phá tài liệu query builder để tìm hiểu về tất cả các phương thức có sẵn cho bạn.

<a name="chaining-orwhere-clauses-after-relationships"></a>
#### Chaining `orWhere` Clauses After Relationships

Như được minh họa trong ví dụ trên, bạn có thể tự do thêm các ràng buộc bổ sung vào các mối quan hệ khi truy vấn chúng. Tuy nhiên, hãy thận trọng khi chuỗi các mệnh đề `orWhere` vào một mối quan hệ, vì các mệnh đề `orWhere` sẽ được nhóm logic ở cùng cấp với ràng buộc mối quan hệ:

```php
$user->posts()
    ->where('active', 1)
    ->orWhere('votes', '>=', 100)
    ->get();
```

Ví dụ trên sẽ tạo ra SQL sau. Như bạn có thể thấy, mệnh đề `or` hướng dẫn truy vấn trả về _bất kỳ_ bài đăng nào có hơn 100 phiếu bầu. Truy vấn không còn bị ràng buộc với một người dùng cụ thể:

```sql
select *
from posts
where user_id = ? and active = 1 or votes >= 100
```

Trong hầu hết các tình huống, bạn nên sử dụng [nhóm logic](/docs/{{version}}/queries#logical-grouping) để nhóm các kiểm tra điều kiện giữa các dấu ngoặc đơn:

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
    ->where(function (Builder $query) {
        return $query->where('active', 1)
            ->orWhere('votes', '>=', 100);
    })
    ->get();
```

Ví dụ trên sẽ tạo ra SQL sau. Lưu ý rằng nhóm logic đã nhóm các ràng buộc đúng cách và truy vấn vẫn bị ràng buộc với một người dùng cụ thể:

```sql
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relationship-methods-vs-dynamic-properties"></a>
### Relationship Methods vs. Dynamic Properties

Nếu bạn không cần thêm các ràng buộc bổ sung vào một truy vấn mối quan hệ Eloquent, bạn có thể truy cập mối quan hệ như thể nó là một thuộc tính. Ví dụ, tiếp tục sử dụng các model ví dụ `User` và `Post` của chúng ta, chúng ta có thể truy cập tất cả các bài đăng của một người dùng như sau:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

Các thuộc tính mối quan hệ động thực hiện "lazy loading", nghĩa là chúng sẽ chỉ tải dữ liệu mối quan hệ khi bạn thực sự truy cập chúng. Vì lý do này, các nhà phát triển thường sử dụng [eager loading](#eager-loading) để tải trước các mối quan hệ mà họ biết sẽ được truy cập sau khi tải model. Eager loading cung cấp sự giảm đáng kể các truy vấn SQL phải được thực thi để tải các mối quan hệ của một model.

<a name="querying-relationship-existence"></a>
### Querying Relationship Existence

Khi truy xuất các bản ghi model, bạn có thể muốn giới hạn kết quả của mình dựa trên sự tồn tại của một mối quan hệ. Ví dụ, hãy tưởng tượng bạn muốn truy xuất tất cả các bài đăng blog có ít nhất một bình luận. Để làm như vậy, bạn có thể truyền tên của mối quan hệ cho các phương thức `has` và `orHas`:

```php
use App\Models\Post;

// Retrieve all posts that have at least one comment...
$posts = Post::has('comments')->get();
```

Bạn cũng có thể chỉ định một toán tử và giá trị đếm để tùy chỉnh thêm truy vấn:

```php
// Retrieve all posts that have three or more comments...
$posts = Post::has('comments', '>=', 3)->get();
```

Các câu lệnh `has` lồng nhau có thể được xây dựng bằng cách sử dụng ký hiệu "dot". Ví dụ, bạn có thể truy xuất tất cả các bài đăng có ít nhất một bình luận có ít nhất một hình ảnh:

```php
// Retrieve posts that have at least one comment with images...
$posts = Post::has('comments.images')->get();
```

Nếu bạn cần thêm sức mạnh hơn, bạn có thể sử dụng các phương thức `whereHas` và `orWhereHas` để định nghĩa các ràng buộc truy vấn bổ sung trên các truy vấn `has` của bạn, chẳng hạn như kiểm tra nội dung của một bình luận:

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
> Eloquent hiện không hỗ trợ truy vấn sự tồn tại của mối quan hệ trên các database. Các mối quan hệ phải tồn tại trong cùng một database.

<a name="many-to-many-relationship-existence-queries"></a>
#### Many to Many Relationship Existence Queries

Phương thức `whereAttachedTo` có thể được sử dụng để truy vấn cho các model có một mối quan hệ nhiều-đến-nhiều với một model hoặc collection của các model:

```php
$users = User::whereAttachedTo($role)->get();
```

Bạn cũng có thể cung cấp một instance [collection](/docs/{{version}}/eloquent-collections) cho phương thức `whereAttachedTo`. Khi làm như vậy, Laravel sẽ truy xuất các model được gắn vào bất kỳ model cha nào trong collection:

```php
$tags = Tag::whereLike('name', '%laravel%')->get();

$posts = Post::whereAttachedTo($tags)->get();
```

<a name="inline-relationship-existence-queries"></a>
#### Inline Relationship Existence Queries

Nếu bạn muốn truy vấn sự tồn tại của một mối quan hệ với một điều kiện where đơn giản được gắn vào truy vấn mối quan hệ, bạn có thể thấy việc sử dụng các phương thức `whereRelation`, `orWhereRelation`, `whereMorphRelation`, và `orWhereMorphRelation` tiện lợi hơn. Ví dụ, chúng ta có thể truy vấn cho tất cả các bài đăng có bình luận chưa được phê duyệt:

```php
use App\Models\Post;

$posts = Post::whereRelation('comments', 'is_approved', false)->get();
```

Tất nhiên, giống như các lệnh gọi đến phương thức `where` của query builder, bạn cũng có thể chỉ định một toán tử:

```php
$posts = Post::whereRelation(
    'comments', 'created_at', '>=', now()->minus(hours: 1)
)->get();
```

<a name="querying-relationship-absence"></a>
### Querying Relationship Absence

Khi truy xuất các bản ghi model, bạn có thể muốn giới hạn kết quả của mình dựa trên sự vắng mặt của một mối quan hệ. Ví dụ, hãy tưởng tượng bạn muốn truy xuất tất cả các bài đăng blog **không** có bất kỳ bình luận nào. Để làm như vậy, bạn có thể truyền tên của mối quan hệ cho các phương thức `doesntHave` và `orDoesntHave`:

```php
use App\Models\Post;

$posts = Post::doesntHave('comments')->get();
```

Nếu bạn cần thêm sức mạnh hơn, bạn có thể sử dụng các phương thức `whereDoesntHave` và `orWhereDoesntHave` để thêm các ràng buộc truy vấn bổ sung vào các truy vấn `doesntHave` của bạn, chẳng hạn như kiểm tra nội dung của một bình luận:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();
```

Bạn có thể sử dụng ký hiệu "dot" để thực thi một truy vấn đối với một mối quan hệ lồng nhau. Ví dụ, truy vấn sau sẽ truy xuất tất cả các bài đăng không có bình luận cũng như các bài đăng có bình luận trong đó không có bình luận nào từ người dùng bị cấm:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 1);
})->get();
```

<a name="querying-morph-to-relationships"></a>
### Querying Morph To Relationships

Để truy vấn sự tồn tại của các mối quan hệ "morph to", bạn có thể sử dụng các phương thức `whereHasMorph` và `whereDoesntHaveMorph`. Các phương thức này chấp nhận tên của mối quan hệ làm đối số đầu tiên. Tiếp theo, các phương thức chấp nhận tên của các model liên quan mà bạn muốn bao gồm trong truy vấn. Cuối cùng, bạn có thể cung cấp một closure tùy chỉnh truy vấn mối quan hệ:

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

Đôi khi bạn có thể cần thêm các ràng buộc truy vấn dựa trên "loại" của model polymorphic liên quan. Closure được truyền cho phương thức `whereHasMorph` có thể nhận một giá trị `$type` làm đối số thứ hai. Đối số này cho phép bạn kiểm tra "loại" của truy vấn đang được xây dựng:

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

Đôi khi bạn muốn truy vấn cho các con của một cha của mối quan hệ "morph to". Bạn có thể thực hiện điều này bằng cách sử dụng các phương thức `whereMorphedTo` và `whereNotMorphedTo`, phương thức này sẽ tự động xác định ánh xạ loại morph thích hợp cho model đã cho. Các phương thức này chấp nhận tên của mối quan hệ `morphTo` làm đối số đầu tiên và model cha liên quan làm đối số thứ hai:

```php
$comments = Comment::whereMorphedTo('commentable', $post)
    ->orWhereMorphedTo('commentable', $video)
    ->get();
```

<a name="querying-all-morph-to-related-models"></a>
#### Querying All Related Models

Thay vì truyền một mảng các model polymorphic có thể, bạn có thể cung cấp `*` làm giá trị wildcard. Điều này sẽ hướng dẫn Laravel truy xuất tất cả các loại polymorphic có thể từ database. Laravel sẽ thực thi một truy vấn bổ sung để thực hiện thao tác này:

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

Đôi khi bạn muốn đếm số lượng model liên quan cho một mối quan hệ đã cho mà không thực sự tải các model. Để thực hiện điều này, bạn có thể sử dụng phương thức `withCount`. Phương thức `withCount` sẽ đặt một thuộc tính `{relation}_count` trên các model kết quả:

```php
use App\Models\Post;

$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

Bằng cách truyền một mảng cho phương thức `withCount`, bạn có thể thêm "số đếm" cho nhiều mối quan hệ cũng như thêm các ràng buộc bổ sung vào các truy vấn:

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'code%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

Bạn cũng có thể đặt tên cho kết quả đếm mối quan hệ, cho phép nhiều số đếm trên cùng một mối quan hệ:

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

Sử dụng phương thức `loadCount`, bạn có thể tải số đếm mối quan hệ sau khi model cha đã được truy xuất:

```php
$book = Book::first();

$book->loadCount('genres');
```

Nếu bạn cần đặt các ràng buộc truy vấn bổ sung trên truy vấn đếm, bạn có thể truyền một mảng được khóa bởi các mối quan hệ bạn muốn đếm. Các giá trị mảng nên là các closure nhận instance query builder:

```php
$book->loadCount(['reviews' => function (Builder $query) {
    $query->where('rating', 5);
}])
```

<a name="relationship-counting-and-custom-select-statements"></a>
#### Relationship Counting and Custom Select Statements

Nếu bạn đang kết hợp `withCount` với một câu lệnh `select`, hãy đảm bảo rằng bạn gọi `withCount` sau phương thức `select`:

```php
$posts = Post::select(['title', 'body'])
    ->withCount('comments')
    ->get();
```

<a name="other-aggregate-functions"></a>
### Other Aggregate Functions

Ngoài phương thức `withCount`, Eloquent cung cấp các phương thức `withMin`, `withMax`, `withAvg`, `withSum`, và `withExists`. Các phương thức này sẽ đặt một thuộc tính `{relation}_{function}_{column}` trên các model kết quả của bạn:

```php
use App\Models\Post;

$posts = Post::withSum('comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->comments_sum_votes;
}
```

Nếu bạn muốn truy cập kết quả của hàm tổng hợp bằng cách sử dụng một tên khác, bạn có thể chỉ định bí danh của riêng bạn:

```php
$posts = Post::withSum('comments as total_comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->total_comments;
}
```

Giống như phương thức `loadCount`, các phiên bản deferred của các phương thức này cũng có sẵn. Các hoạt động tổng hợp bổ sung này có thể được thực hiện trên các model Eloquent đã được truy xuất:

```php
$post = Post::first();

$post->loadSum('comments', 'votes');
```

Nếu bạn đang kết hợp các phương thức tổng hợp này với một câu lệnh `select`, hãy đảm bảo rằng bạn gọi các phương thức tổng hợp sau phương thức `select`:

```php
$posts = Post::select(['title', 'body'])
    ->withExists('comments')
    ->get();
```

<a name="counting-related-models-on-morph-to-relationships"></a>
### Counting Related Models on Morph To Relationships

Nếu bạn muốn eager load một mối quan hệ "morph to", cũng như số đếm model liên quan cho các thực thể khác nhau có thể được trả về bởi mối quan hệ đó, bạn có thể sử dụng phương thức `with` kết hợp với phương thức `morphWithCount` của mối quan hệ `morphTo`.

Trong ví dụ này, hãy giả sử rằng các model `Photo` và `Post` có thể tạo ra các model `ActivityFeed`. Chúng ta sẽ giả định rằng model `ActivityFeed` định nghĩa một mối quan hệ "morph to" tên là `parentable` cho phép chúng ta truy xuất model cha `Photo` hoặc `Post` cho một instance `ActivityFeed` đã cho. Ngoài ra, hãy giả định rằng các model `Photo` "có nhiều" model `Tag` và các model `Post` "có nhiều" model `Comment`.

Bây giờ, hãy tưởng tượng chúng ta muốn truy xuất các instance `ActivityFeed` và eager load các model cha `parentable` cho mỗi instance `ActivityFeed`. Ngoài ra, chúng ta muốn truy xuất số lượng thẻ được liên kết với mỗi ảnh cha và số lượng bình luận được liên kết với mỗi bài đăng cha:

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

Hãy giả sử chúng ta đã truy xuất một tập hợp các model `ActivityFeed` và bây giờ chúng ta muốn tải các số đếm mối quan hệ lồng nhau cho các model `parentable` khác nhau liên quan đến các hoạt động. Bạn có thể sử dụng phương thức `loadMorphCount` để thực hiện điều này:

```php
$activities = ActivityFeed::with('parentable')->get();

$activities->loadMorphCount('parentable', [
    Photo::class => ['tags'],
    Post::class => ['comments'],
]);
```

<a name="eager-loading"></a>
## Eager Loading

Khi truy cập các mối quan hệ Eloquent dưới dạng thuộc tính, các model liên quan được "lazy loaded". Điều này có nghĩa là dữ liệu mối quan hệ không thực sự được tải cho đến khi bạn lần đầu tiên truy cập thuộc tính. Tuy nhiên, Eloquent có thể "eager load" các mối quan hệ tại thời điểm bạn truy vấn model cha. Eager loading làm giảm vấn đề truy vấn "N + 1". Để minh họa vấn đề truy vấn N + 1, hãy xem xét một model `Book` "belongs to" một model `Author`:

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

Bây giờ, hãy truy xuất tất cả các sách và tác giả của chúng:

```php
use App\Models\Book;

$books = Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

Vòng lặp này sẽ thực thi một truy vấn để truy xuất tất cả các sách trong bảng database, sau đó một truy vấn khác cho mỗi sách để truy xuất tác giả của sách. Vì vậy, nếu chúng ta có 25 sách, mã trên sẽ chạy 26 truy vấn: một cho sách gốc, và 25 truy vấn bổ sung để truy xuất tác giả của mỗi sách.

May mắn thay, chúng ta có thể sử dụng eager loading để giảm thao tác này xuống chỉ còn hai truy vấn. Khi xây dựng một truy vấn, bạn có thể chỉ định các mối quan hệ nên được eager load bằng cách sử dụng phương thức `with`:

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

Đối với thao tác này, chỉ hai truy vấn sẽ được thực thi - một truy vấn để truy xuất tất cả các sách và một truy vấn để truy xuất tất cả các tác giả cho tất cả các sách:

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

<a name="eager-loading-multiple-relationships"></a>
#### Eager Loading Multiple Relationships

Đôi khi bạn có thể cần eager load một số mối quan hệ khác nhau. Để làm như vậy, chỉ cần truyền một mảng các mối quan hệ cho phương thức `with`:

```php
$books = Book::with(['author', 'publisher'])->get();
```

<a name="nested-eager-loading"></a>
#### Nested Eager Loading

Để eager load các mối quan hệ của một mối quan hệ, bạn có thể sử dụng ký hiệu "dot". Ví dụ, hãy eager load tất cả các tác giả của sách và tất cả các liên hệ cá nhân của tác giả:

```php
$books = Book::with('author.contacts')->get();
```

Ngoài ra, bạn có thể chỉ định các mối quan hệ eager load lồng nhau bằng cách cung cấp một mảng lồng nhau cho phương thức `with`, điều này có thể thuận tiện khi eager load nhiều mối quan hệ lồng nhau:

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

Nếu bạn muốn eager load một mối quan hệ `morphTo`, cũng như các mối quan hệ lồng nhau trên các thực thể khác nhau có thể được trả về bởi mối quan hệ đó, bạn có thể sử dụng phương thức `with` kết hợp với phương thức `morphWith` của mối quan hệ `morphTo`. Để giúp minh họa phương thức này, hãy xem xét model sau:

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

Trong ví dụ này, hãy giả sử các model `Event`, `Photo`, và `Post` có thể tạo ra các model `ActivityFeed`. Ngoài ra, hãy giả định rằng các model `Event` thuộc về một model `Calendar`, các model `Photo` được liên kết với các model `Tag`, và các model `Post` thuộc về một model `Author`.

Sử dụng các định nghĩa và mối quan hệ model này, chúng ta có thể truy xuất các instance model `ActivityFeed` và eager load tất cả các model `parentable` và các mối quan hệ lồng nhau tương ứng của chúng:

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

Bạn có thể không luôn cần mọi cột từ các mối quan hệ bạn đang truy xuất. Vì lý do này, Eloquent cho phép bạn chỉ định các cột của mối quan hệ bạn muốn truy xuất:

```php
$books = Book::with('author:id,name,book_id')->get();
```

> [!WARNING]
> Khi sử dụng tính năng này, bạn nên luôn bao gồm cột `id` và bất kỳ cột khóa ngoại liên quan nào trong danh sách các cột bạn muốn truy xuất.

<a name="eager-loading-by-default"></a>
#### Eager Loading by Default

Đôi khi bạn có thể muốn luôn tải một số mối quan hệ khi truy xuất một model. Để thực hiện điều này, bạn có thể định nghĩa một thuộc tính `$with` trên model:

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

Nếu bạn muốn xóa một mục khỏi thuộc tính `$with` cho một truy vấn duy nhất, bạn có thể sử dụng phương thức `without`:

```php
$books = Book::without('author')->get();
```

Nếu bạn muốn ghi đè tất cả các mục trong thuộc tính `$with` cho một truy vấn duy nhất, bạn có thể sử dụng phương thức `withOnly`:

```php
$books = Book::withOnly('genre')->get();
```

<a name="constraining-eager-loads"></a>
### Constraining Eager Loads

Đôi khi bạn có thể muốn eager load một mối quan hệ nhưng cũng chỉ định các điều kiện truy vấn bổ sung cho truy vấn eager loading. Bạn có thể thực hiện điều này bằng cách truyền một mảng các mối quan hệ cho phương thức `with` trong đó khóa mảng là tên mối quan hệ và giá trị mảng là một closure thêm các ràng buộc bổ sung vào truy vấn eager loading:

```php
use App\Models\User;

$users = User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%code%');
}])->get();
```

Trong ví dụ này, Eloquent sẽ chỉ eager load các bài đăng trong đó cột `title` của bài đăng chứa từ `code`. Bạn có thể gọi các phương thức [query builder](/docs/{{version}}/queries) khác để tùy chỉnh thêm thao tác eager loading:

```php
$users = User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

<a name="constraining-eager-loading-of-morph-to-relationships"></a>
#### Constraining Eager Loading of `morphTo` Relationships

Nếu bạn đang eager load một mối quan hệ `morphTo`, Eloquent sẽ chạy nhiều truy vấn để lấy từng loại model liên quan. Bạn có thể thêm các ràng buộc bổ sung vào từng truy vấn này bằng cách sử dụng phương thức `constrain` của mối quan hệ `MorphTo`:

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

Trong ví dụ này, Eloquent sẽ chỉ eager load các bài đăng chưa bị ẩn và các video có giá trị `type` là "educational".

<a name="constraining-eager-loads-with-relationship-existence"></a>
#### Constraining Eager Loads With Relationship Existence

Đôi khi bạn có thể thấy mình cần kiểm tra sự tồn tại của một mối quan hệ trong khi đồng thời tải mối quan hệ dựa trên cùng các điều kiện. Ví dụ, bạn có thể chỉ muốn truy xuất các model `User` có các model con `Post` khớp với một điều kiện truy vấn đã cho trong khi cũng eager load các bài đăng khớp. Bạn có thể thực hiện điều này bằng cách sử dụng phương thức `withWhereHas`:

```php
use App\Models\User;

$users = User::withWhereHas('posts', function ($query) {
    $query->where('featured', true);
})->get();
```

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Đôi khi bạn có thể cần eager load một mối quan hệ sau khi model cha đã được truy xuất. Ví dụ, điều này có thể hữu ích nếu bạn cần quyết định động có nên tải các model liên quan hay không:

```php
use App\Models\Book;

$books = Book::all();

if ($condition) {
    $books->load('author', 'publisher');
}
```

Nếu bạn cần đặt các ràng buộc truy vấn bổ sung trên truy vấn eager loading, bạn có thể truyền một mảng được khóa bởi các mối quan hệ bạn muốn tải. Các giá trị mảng nên là các instance closure nhận instance truy vấn:

```php
$author->load(['books' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

Để tải một mối quan hệ chỉ khi nó chưa được tải, hãy sử dụng phương thức `loadMissing`:

```php
$book->loadMissing('author');
```

<a name="nested-lazy-eager-loading-morphto"></a>
#### Nested Lazy Eager Loading và `morphTo`

Nếu bạn muốn eager load một mối quan hệ `morphTo`, cũng như các mối quan hệ lồng nhau trên các thực thể khác nhau có thể được trả về bởi mối quan hệ đó, bạn có thể sử dụng phương thức `loadMorph`.

Phương thức này chấp nhận tên của mối quan hệ `morphTo` làm đối số đầu tiên, và một mảng các cặp model / mối quan hệ làm đối số thứ hai. Để giúp minh họa phương thức này, hãy xem xét model sau:

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

Trong ví dụ này, hãy giả sử các model `Event`, `Photo`, và `Post` có thể tạo ra các model `ActivityFeed`. Ngoài ra, hãy giả định rằng các model `Event` thuộc về một model `Calendar`, các model `Photo` được liên kết với các model `Tag`, và các model `Post` thuộc về một model `Author`.

Sử dụng các định nghĩa và mối quan hệ model này, chúng ta có thể truy xuất các instance model `ActivityFeed` và eager load tất cả các model `parentable` và các mối quan hệ lồng nhau tương ứng của chúng:

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
> Tính năng này hiện đang ở beta để thu thập phản hồi từ cộng đồng. Hành vi và chức năng của tính năng này có thể thay đổi ngay cả trên các bản vá.

Trong nhiều trường hợp, Laravel có thể tự động eager load các mối quan hệ bạn truy cập. Để bật eager loading tự động, bạn nên gọi phương thức `Model::automaticallyEagerLoadRelationships` trong phương thức `boot` của `AppServiceProvider` của ứng dụng của bạn:

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

Khi tính năng này được bật, Laravel sẽ cố gắng tự động tải bất kỳ mối quan hệ nào bạn truy cập chưa được tải trước đó. Ví dụ, hãy xem xét tình huống sau:

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

Thông thường, mã trên sẽ thực thi một truy vấn cho mỗi người dùng để truy xuất các bài đăng của họ, cũng như một truy vấn cho mỗi bài đăng để truy xuất các bình luận của nó. Tuy nhiên, khi tính năng `automaticallyEagerLoadRelationships` đã được bật, Laravel sẽ tự động [lazy eager load](#lazy-eager-loading) các bài đăng cho tất cả người dùng trong collection người dùng khi bạn cố gắng truy cập các bài đăng trên bất kỳ người dùng nào đã được truy xuất. Tương tự, khi bạn cố gắng truy cập các bình luận cho bất kỳ bài đăng nào đã được truy xuất, tất cả các bình luận sẽ được lazy eager load cho tất cả các bài đăng đã được truy xuất ban đầu.

Nếu bạn không muốn bật eager loading tự động toàn cục, bạn vẫn có thể bật tính năng này cho một instance collection Eloquent duy nhất bằng cách gọi phương thức `withRelationshipAutoloading` trên collection:

```php
$users = User::where('vip', true)->get();

return $users->withRelationshipAutoloading();
```

<a name="preventing-lazy-loading"></a>
### Preventing Lazy Loading

Như đã thảo luận trước đó, eager loading các mối quan hệ thường có thể mang lại lợi ích hiệu suất đáng kể cho ứng dụng của bạn. Do đó, nếu bạn muốn, bạn có thể hướng dẫn Laravel luôn ngăn chặn lazy loading của các mối quan hệ. Để thực hiện điều này, bạn có thể gọi phương thức `preventLazyLoading` được cung cấp bởi lớp model Eloquent cơ sở. Thông thường, bạn nên gọi phương thức này trong phương thức `boot` của lớp `AppServiceProvider` của ứng dụng của bạn.

Phương thức `preventLazyLoading` chấp nhận một đối số boolean tùy chọn chỉ ra liệu lazy loading có nên được ngăn chặn hay không. Ví dụ, bạn có thể chỉ muốn vô hiệu hóa lazy loading trong các môi trường không sản xuất để môi trường sản xuất của bạn sẽ tiếp tục hoạt động bình thường ngay cả khi một mối quan hệ lazy load vô tình có trong mã sản xuất:

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

Sau khi ngăn chặn lazy loading, Eloquent sẽ ném một ngoại lệ `Illuminate\Database\LazyLoadingViolationException` khi ứng dụng của bạn cố gắng lazy load bất kỳ mối quan hệ Eloquent nào.

Bạn có thể tùy chỉnh hành vi của các vi phạm lazy loading bằng cách sử dụng phương thức `handleLazyLoadingViolationsUsing`. Ví dụ, sử dụng phương thức này, bạn có thể hướng dẫn các vi phạm lazy loading chỉ được ghi nhật ký thay vì làm gián đoạn việc thực thi ứng dụng với các ngoại lệ:

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

Eloquent cung cấp các phương thức thuận tiện để thêm các model mới vào các mối quan hệ. Ví dụ, có thể bạn cần thêm một bình luận mới vào một bài đăng. Thay vì đặt thủ công thuộc tính `post_id` trên model `Comment`, bạn có thể chèn bình luận bằng cách sử dụng phương thức `save` của mối quan hệ:

```php
use App\Models\Comment;
use App\Models\Post;

$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$post->comments()->save($comment);
```

Lưu ý rằng chúng ta không truy cập mối quan hệ `comments` như một thuộc tính động. Thay vào đó, chúng ta gọi phương thức `comments` để lấy một instance của mối quan hệ. Phương thức `save` sẽ tự động thêm giá trị `post_id` thích hợp vào model `Comment` mới.

Nếu bạn cần lưu nhiều model liên quan, bạn có thể sử dụng phương thức `saveMany`:

```php
$post = Post::find(1);

$post->comments()->saveMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another new comment.']),
]);
```

Các phương thức `save` và `saveMany` sẽ duy trì các instance model đã cho, nhưng sẽ không thêm các model mới được duy trì vào bất kỳ mối quan hệ trong bộ nhớ nào đã được tải trên model cha. Nếu bạn định truy cập mối quan hệ sau khi sử dụng các phương thức `save` hoặc `saveMany`, bạn có thể muốn sử dụng phương thức `refresh` để tải lại model và các mối quan hệ của nó:

```php
$post->comments()->save($comment);

$post->refresh();

// All comments, including the newly saved comment...
$post->comments;
```

<a name="the-push-method"></a>
#### Recursively Saving Models and Relationships

Nếu bạn muốn `save` model của bạn và tất cả các mối quan hệ liên quan của nó, bạn có thể sử dụng phương thức `push`. Trong ví dụ này, model `Post` sẽ được lưu cũng như các bình luận của nó và tác giả của bình luận:

```php
$post = Post::find(1);

$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';

$post->push();
```

Phương thức `pushQuietly` có thể được sử dụng để lưu một model và các mối quan hệ liên quan của nó mà không kích hoạt bất kỳ sự kiện nào:

```php
$post->pushQuietly();
```

<a name="the-create-method"></a>
### The `create` Method

Ngoài các phương thức `save` và `saveMany`, bạn cũng có thể sử dụng phương thức `create`, phương thức này chấp nhận một mảng các thuộc tính, tạo một model, và chèn nó vào database. Sự khác biệt giữa `save` và `create` là `save` chấp nhận một instance model Eloquent đầy đủ trong khi `create` chấp nhận một mảng `array` PHP thuần túy. Model mới được tạo sẽ được trả về bởi phương thức `create`:

```php
use App\Models\Post;

$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

Bạn có thể sử dụng phương thức `createMany` để tạo nhiều model liên quan:

```php
$post = Post::find(1);

$post->comments()->createMany([
    ['message' => 'A new comment.'],
    ['message' => 'Another new comment.'],
]);
```

Các phương thức `createQuietly` và `createManyQuietly` có thể được sử dụng để tạo một (hoặc nhiều) model mà không gửi bất kỳ sự kiện nào:

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

Bạn cũng có thể sử dụng các phương thức `findOrNew`, `firstOrNew`, `firstOrCreate`, và `updateOrCreate` để [tạo và cập nhật các model trên các mối quan hệ](/docs/{{version}}/eloquent#upserts).

> [!NOTE]
> Trước khi sử dụng phương thức `create`, hãy đảm bảo xem lại tài liệu [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

<a name="updating-belongs-to-relationships"></a>
### Belongs To Relationships

Nếu bạn muốn gán một model con cho một model cha mới, bạn có thể sử dụng phương thức `associate`. Trong ví dụ này, model `User` định nghĩa một mối quan hệ `belongsTo` với model `Account`. Phương thức `associate` này sẽ đặt khóa ngoại trên model con:

```php
use App\Models\Account;

$account = Account::find(10);

$user->account()->associate($account);

$user->save();
```

Để xóa một model cha khỏi một model con, bạn có thể sử dụng phương thức `dissociate`. Phương thức này sẽ đặt khóa ngoại của mối quan hệ thành `null`:

```php
$user->account()->dissociate();

$user->save();
```

<a name="updating-many-to-many-relationships"></a>
### Many to Many Relationships

<a name="attaching-detaching"></a>
#### Attaching / Detaching

Eloquent cũng cung cấp các phương thức để làm việc với các mối quan hệ nhiều-đến-nhiều thuận tiện hơn. Ví dụ, hãy tưởng tượng một người dùng có thể có nhiều vai trò và một vai trò có thể có nhiều người dùng. Bạn có thể sử dụng phương thức `attach` để gắn một vai trò cho một người dùng bằng cách chèn một bản ghi trong bảng trung gian của mối quan hệ:

```php
use App\Models\User;

$user = User::find(1);

$user->roles()->attach($roleId);
```

Khi gắn một mối quan hệ vào một model, bạn cũng có thể truyền một mảng dữ liệu bổ sung để chèn vào bảng trung gian:

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

Đôi khi có thể cần thiết để xóa một vai trò khỏi một người dùng. Để xóa một bản ghi mối quan hệ nhiều-đến-nhiều, hãy sử dụng phương thức `detach`. Phương thức `detach` sẽ xóa bản ghi thích hợp khỏi bảng trung gian; tuy nhiên, cả hai model sẽ vẫn còn trong database:

```php
// Detach a single role from the user...
$user->roles()->detach($roleId);

// Detach all roles from the user...
$user->roles()->detach();
```

Để thuận tiện, `attach` và `detach` cũng chấp nhận các mảng ID làm đầu vào:

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

Bạn cũng có thể sử dụng phương thức `sync` để xây dựng các liên kết nhiều-đến-nhiều. Phương thức `sync` chấp nhận một mảng ID để đặt trên bảng trung gian. Bất kỳ ID nào không có trong mảng đã cho sẽ bị xóa khỏi bảng trung gian. Vì vậy, sau khi thao tác này hoàn tất, chỉ có các ID trong mảng đã cho sẽ tồn tại trong bảng trung gian:

```php
$user->roles()->sync([1, 2, 3]);
```

Bạn cũng có thể truyền các giá trị bảng trung gian bổ sung với các ID:

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

Nếu bạn muốn chèn cùng một giá trị bảng trung gian với mỗi ID model được đồng bộ hóa, bạn có thể sử dụng phương thức `syncWithPivotValues`:

```php
$user->roles()->syncWithPivotValues([1, 2, 3], ['active' => true]);
```

Nếu bạn không muốn xóa các ID hiện có bị thiếu khỏi mảng đã cho, bạn có thể sử dụng phương thức `syncWithoutDetaching`:

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

<a name="toggling-associations"></a>
#### Toggling Associations

Mối quan hệ nhiều-đến-nhiều cũng cung cấp một phương thức `toggle` "chuyển đổi" trạng thái gắn của các ID model liên quan đã cho. Nếu ID đã cho hiện đang được gắn, nó sẽ bị tách. Tương tự, nếu nó hiện đang bị tách, nó sẽ được gắn:

```php
$user->roles()->toggle([1, 2, 3]);
```

Bạn cũng có thể truyền các giá trị bảng trung gian bổ sung với các ID:

```php
$user->roles()->toggle([
    1 => ['expires' => true],
    2 => ['expires' => true],
]);
```

<a name="transactional-pivot-operations"></a>
#### Transactional Pivot Operations

Mỗi thao tác pivot được thảo luận ở trên cũng có một biến thể `OrFail` (`attachOrFail`, `detachOrFail`, `syncOrFail`, `syncWithoutDetachingOrFail`, và `toggleOrFail`) bao bọc thao tác trong một giao dịch database, để tất cả các thay đổi được tự động rollback nếu một ngoại lệ được ném:

```php
$user->roles()->attachOrFail([1, 2, 3]);
$user->roles()->syncOrFail([1, 2, 3]);
```

<a name="updating-a-record-on-the-intermediate-table"></a>
#### Updating a Record on the Intermediate Table

Nếu bạn cần cập nhật một hàng hiện có trong bảng trung gian của mối quan hệ của bạn, bạn có thể sử dụng phương thức `updateExistingPivot`. Phương thức này chấp nhận khóa ngoại của bản ghi trung gian và một mảng các thuộc tính để cập nhật:

```php
$user = User::find(1);

$user->roles()->updateExistingPivot($roleId, [
    'active' => false,
]);
```

<a name="touching-parent-timestamps"></a>
## Touching Parent Timestamps

Khi một model định nghĩa một mối quan hệ `belongsTo` hoặc `belongsToMany` với một model khác, chẳng hạn như một `Comment` thuộc về một `Post`, đôi khi có ích để cập nhật timestamp của cha khi model con được cập nhật.

Ví dụ, khi một model `Comment` được cập nhật, bạn có thể muốn tự động "touch" timestamp `updated_at` của `Post` sở hữu để nó được đặt thành ngày và giờ hiện tại. Để thực hiện điều này, bạn có thể sử dụng thuộc tính `Touches` trên model con của bạn chứa tên của các mối quan hệ nên có timestamp `updated_at` của chúng được cập nhật khi model con được cập nhật:

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
> Timestamp của model cha sẽ chỉ được cập nhật nếu model con được cập nhật bằng cách sử dụng phương thức `save` của Eloquent.

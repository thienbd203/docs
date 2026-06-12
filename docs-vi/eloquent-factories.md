# Eloquent: Factories

- [Introduction](#introduction)
- [Defining Model Factories](#defining-model-factories)
    - [Generating Factories](#generating-factories)
    - [Factory States](#factory-states)
    - [Factory Callbacks](#factory-callbacks)
- [Creating Models Using Factories](#creating-models-using-factories)
    - [Instantiating Models](#instantiating-models)
    - [Persisting Models](#persisting-models)
    - [Sequences](#sequences)
- [Factory Relationships](#factory-relationships)
    - [Has Many Relationships](#has-many-relationships)
    - [Belongs To Relationships](#belongs-to-relationships)
    - [Many to Many Relationships](#many-to-many-relationships)
    - [Polymorphic Relationships](#polymorphic-relationships)
    - [Defining Relationships Within Factories](#defining-relationships-within-factories)
    - [Recycling an Existing Model for Relationships](#recycling-an-existing-model-for-relationships)

<a name="introduction"></a>
## Introduction

Khi test ứng dụng của bạn hoặc seed database của bạn, bạn có thể cần chèn một vài bản ghi vào database của mình. Thay vì chỉ định thủ công giá trị của mỗi cột, Laravel cho phép bạn định nghĩa một tập hợp các thuộc tính mặc định cho mỗi [Eloquent model](/docs/{{version}}/eloquent) của bạn bằng cách sử dụng model factories.

Để xem một ví dụ về cách viết một factory, hãy xem file `database/factories/UserFactory.php` trong ứng dụng của bạn. Factory này được bao gồm với tất cả các ứng dụng Laravel mới và chứa định nghĩa factory sau:

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\User>
 */
class UserFactory extends Factory
{
    /**
     * The current password being used by the factory.
     */
    protected static ?string $password;

    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }

    /**
     * Indicate that the model's email address should be unverified.
     */
    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }
}
```

Như bạn có thể thấy, ở dạng cơ bản nhất, factories là các lớp mở rộng lớp factory cơ bản của Laravel và định nghĩa một phương thức `definition`. Phương thức `definition` trả về tập hợp mặc định các giá trị thuộc tính nên được áp dụng khi tạo một model bằng cách sử dụng factory.

Thông qua helper `fake`, factories có quyền truy cập vào thư viện PHP [Faker](https://github.com/FakerPHP/Faker), cho phép bạn thuận tiện tạo nhiều loại dữ liệu ngẫu nhiên khác nhau để test và seed.

> [!NOTE]
> Bạn có thể thay đổi locale Faker của ứng dụng bằng cách cập nhật tùy chọn `faker_locale` trong file cấu hình `config/app.php`.

<a name="defining-model-factories"></a>
## Defining Model Factories

<a name="generating-factories"></a>
### Generating Factories

Để tạo một factory, thực thi lệnh Artisan `make:factory`:

```shell
php artisan make:factory PostFactory
```

Lớp factory mới sẽ được đặt trong thư mục `database/factories` của bạn.

<a name="factory-and-model-discovery-conventions"></a>
#### Model and Factory Discovery Conventions

Sau khi bạn đã định nghĩa các factories của mình, bạn có thể sử dụng phương thức tĩnh `factory` được cung cấp cho các models của bạn bởi trait `Illuminate\Database\Eloquent\Factories\HasFactory` để khởi tạo một instance factory cho model đó.

Phương thức `factory` của trait `HasFactory` sẽ sử dụng các quy ước để xác định factory thích hợp cho model mà trait được gán. Cụ thể, phương thức sẽ tìm kiếm một factory trong namespace `Database\Factories` có tên lớp khớp với tên model và được hậu tố bằng `Factory`. Nếu các quy ước này không áp dụng cho ứng dụng hoặc factory cụ thể của bạn, bạn có thể thêm attribute `UseFactory` vào model để chỉ định thủ công factory của model:

```php
use Illuminate\Database\Eloquent\Attributes\UseFactory;
use Database\Factories\Administration\FlightFactory;

#[UseFactory(FlightFactory::class)]
class Flight extends Model
{
    // ...
}
```

Ngoài ra, bạn có thể ghi đè phương thức `newFactory` trên model của mình để trả về một instance của factory tương ứng của model trực tiếp:

```php
use Database\Factories\Administration\FlightFactory;

/**
 * Create a new factory instance for the model.
 */
protected static function newFactory()
{
    return FlightFactory::new();
}
```

Sau đó, sử dụng attribute `UseModel` trên factory tương ứng để chỉ định model:

```php
use App\Administration\Flight;
use Illuminate\Database\Eloquent\Factories\Attributes\UseModel;
use Illuminate\Database\Eloquent\Factories\Factory;

#[UseModel(Flight::class)]
class FlightFactory extends Factory
{
    // ...
}
```

<a name="factory-states"></a>
### Factory States

Các phương thức thao tác state cho phép bạn định nghĩa các sửa đổi rời rạc có thể được áp dụng cho các model factories của bạn theo bất kỳ sự kết hợp nào. Ví dụ, factory `Database\Factories\UserFactory` của bạn có thể chứa một phương thức state `suspended` sửa đổi một trong các giá trị thuộc tính mặc định của nó.

Các phương thức chuyển đổi state thường gọi phương thức `state` được cung cấp bởi lớp factory cơ bản của Laravel. Phương thức `state` chấp nhận một closure sẽ nhận array các thuộc tính thô được định nghĩa cho factory và nên trả về một array các thuộc tính để sửa đổi:

```php
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Indicate that the user is suspended.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    });
}
```

<a name="trashed-state"></a>
#### "Trashed" State

Nếu Eloquent model của bạn có thể được [soft deleted](/docs/{{version}}/eloquent#soft-deleting), bạn có thể gọi phương thức state tích hợp sẵn `trashed` để chỉ định rằng model được tạo nên đã được "soft deleted". Bạn không cần định nghĩa thủ công state `trashed` vì nó tự động có sẵn cho tất cả các factories:

```php
use App\Models\User;

$user = User::factory()->trashed()->create();
```

<a name="factory-callbacks"></a>
### Factory Callbacks

Các factory callbacks được đăng ký bằng cách sử dụng các phương thức `afterMaking` và `afterCreating` và cho phép bạn thực hiện các nhiệm vụ bổ sung sau khi làm hoặc tạo một model. Bạn nên đăng ký các callbacks này bằng cách định nghĩa một phương thức `configure` trên lớp factory của bạn. Phương thức này sẽ tự động được gọi bởi Laravel khi factory được khởi tạo:

```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    /**
     * Configure the model factory.
     */
    public function configure(): static
    {
        return $this->afterMaking(function (User $user) {
            // ...
        })->afterCreating(function (User $user) {
            // ...
        });
    }

    // ...
}
```

Bạn cũng có thể đăng ký các factory callbacks trong các phương thức state để thực hiện các nhiệm vụ bổ sung cụ thể cho một state nhất định:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Indicate that the user is suspended.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    })->afterMaking(function (User $user) {
        // ...
    })->afterCreating(function (User $user) {
        // ...
    });
}
```

<a name="creating-models-using-factories"></a>
## Creating Models Using Factories

<a name="instantiating-models"></a>
### Instantiating Models

Sau khi bạn đã định nghĩa các factories của mình, bạn có thể sử dụng phương thức tĩnh `factory` được cung cấp cho các models của bạn bởi trait `Illuminate\Database\Eloquent\Factories\HasFactory` để khởi tạo một instance factory cho model đó. Hãy xem một vài ví dụ về việc tạo models. Đầu tiên, chúng tôi sẽ sử dụng phương thức `make` để tạo models mà không persist chúng vào database:

```php
use App\Models\User;

$user = User::factory()->make();
```

Bạn có thể tạo một collection nhiều models bằng cách sử dụng phương thức `count`:

```php
$users = User::factory()->count(3)->make();
```

<a name="applying-states"></a>
#### Applying States

Bạn cũng có thể áp dụng bất kỳ [states](#factory-states) nào của mình cho các models. Nếu bạn muốn áp dụng nhiều chuyển đổi state cho các models, bạn có thể chỉ cần gọi trực tiếp các phương thức chuyển đổi state:

```php
$users = User::factory()->count(5)->suspended()->make();
```

<a name="overriding-attributes"></a>
#### Overriding Attributes

Nếu bạn muốn ghi đè một số giá trị mặc định của các models của mình, bạn có thể chuyển một array các giá trị cho phương thức `make`. Chỉ các thuộc tính được chỉ định sẽ được thay thế trong khi các thuộc tính còn lại vẫn được đặt thành các giá trị mặc định của chúng như được chỉ định bởi factory:

```php
$user = User::factory()->make([
    'name' => 'Abigail Otwell',
]);
```

Ngoài ra, phương thức `state` có thể được gọi trực tiếp trên instance factory để thực hiện một chuyển đổi state inline:

```php
$user = User::factory()->state([
    'name' => 'Abigail Otwell',
])->make();
```

> [!NOTE]
> [Mass assignment protection](/docs/{{version}}/eloquent#mass-assignment) được tự động tắt khi tạo models bằng cách sử dụng factories.

<a name="persisting-models"></a>
### Persisting Models

Phương thức `create` khởi tạo các instance model và persist chúng vào database bằng cách sử dụng phương thức `save` của Eloquent:

```php
use App\Models\User;

// Create a single App\Models\User instance...
$user = User::factory()->create();

// Create three App\Models\User instances...
$users = User::factory()->count(3)->create();
```

Bạn có thể ghi đè các thuộc tính model mặc định của factory bằng cách chuyển một array các thuộc tính cho phương thức `create`:

```php
$user = User::factory()->create([
    'name' => 'Abigail',
]);
```

<a name="sequences"></a>
### Sequences

Đôi khi bạn có thể muốn thay đổi giá trị của một thuộc tính model nhất định cho mỗi model được tạo. Bạn có thể thực hiện điều này bằng cách định nghĩa một chuyển đổi state như một sequence. Ví dụ, bạn có thể muốn thay đổi giá trị của một cột `admin` giữa `Y` và `N` cho mỗi người dùng được tạo:

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        ['admin' => 'Y'],
        ['admin' => 'N'],
    ))
    ->create();
```

Trong ví dụ này, năm người dùng sẽ được tạo với giá trị `admin` là `Y` và năm người dùng sẽ được tạo với giá trị `admin` là `N`.

Nếu cần thiết, bạn có thể bao gồm một closure làm giá trị sequence. Closure sẽ được gọi mỗi khi sequence cần một giá trị mới:

```php
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['role' => UserRoles::all()->random()],
    ))
    ->create();
```

Trong một closure sequence, bạn có thể truy cập thuộc tính `$index` trên instance sequence được inject vào closure. Thuộc tính `$index` chứa số lần lặp qua sequence đã xảy ra cho đến nay:

```php
$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['name' => 'Name '.$sequence->index],
    ))
    ->create();
```

Để thuận tiện, sequences cũng có thể được áp dụng bằng cách sử dụng phương thức `sequence`, đơn giản gọi phương thức `state` nội bộ. Phương thức `sequence` chấp nhận một closure hoặc arrays của các thuộc tính được sequence:

```php
$users = User::factory()
    ->count(2)
    ->sequence(
        ['name' => 'First User'],
        ['name' => 'Second User'],
    )
    ->create();
```

<a name="factory-relationships"></a>
## Factory Relationships

<a name="has-many-relationships"></a>
### Has Many Relationships

Tiếp theo, hãy khám phá xây dựng các relationships Eloquent model bằng cách sử dụng các phương thức factory fluent của Laravel. Đầu tiên, hãy giả sử ứng dụng của chúng ta có một model `App\Models\User` và một model `App\Models\Post`. Ngoài ra, hãy giả sử rằng model `User` định nghĩa một relationship `hasMany` với `Post`. Chúng ta có thể tạo một người dùng có ba bài viết bằng cách sử dụng phương thức `has` được cung cấp bởi các factories của Laravel. Phương thức `has` chấp nhận một instance factory:

```php
use App\Models\Post;
use App\Models\User;

$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();
```

Theo quy ước, khi chuyển một model `Post` cho phương thức `has`, Laravel sẽ giả định rằng model `User` phải có một phương thức `posts` định nghĩa relationship. Nếu cần thiết, bạn có thể chỉ định rõ tên của relationship mà bạn muốn thao tác:

```php
$user = User::factory()
    ->has(Post::factory()->count(3), 'posts')
    ->create();
```

Tất nhiên, bạn có thể thực hiện các thao tác state trên các models liên quan. Ngoài ra, bạn có thể chuyển một chuyển đổi state dựa trên closure nếu thay đổi state của bạn yêu cầu quyền truy cập vào model cha:

```php
$user = User::factory()
    ->has(
        Post::factory()
            ->count(3)
            ->state(function (array $attributes, User $user) {
                return ['user_type' => $user->type];
            })
    )
    ->create();
```

<a name="has-many-relationships-using-magic-methods"></a>
#### Using Magic Methods

Để thuận tiện, bạn có thể sử dụng các phương thức relationship factory magic của Laravel để xây dựng các relationships. Ví dụ, ví dụ sau sẽ sử dụng quy ước để xác định rằng các models liên quan nên được tạo thông qua một phương thức relationship `posts` trên model `User`:

```php
$user = User::factory()
    ->hasPosts(3)
    ->create();
```

Khi sử dụng các phương thức magic để tạo các factory relationships, bạn có thể chuyển một array các thuộc tính để ghi đè trên các models liên quan:

```php
$user = User::factory()
    ->hasPosts(3, [
        'published' => false,
    ])
    ->create();
```

Bạn cũng có thể chuyển nhiều array thuộc tính để tạo các models liên quan với state per-model. Laravel sẽ áp dụng từng array theo trình tự:

```php
$user = User::factory()
    ->hasPosts(
        ['title' => 'First Post'],
        ['title' => 'Second Post'],
        ['title' => 'Third Post'],
    )
    ->create();
```

Bạn có thể cung cấp một chuyển đổi state dựa trên closure nếu thay đổi state của bạn yêu cầu quyền truy cập vào model cha:

```php
$user = User::factory()
    ->hasPosts(3, function (array $attributes, User $user) {
        return ['user_type' => $user->type];
    })
    ->create();
```

<a name="belongs-to-relationships"></a>
### Belongs To Relationships

Bây giờ chúng ta đã khám phá cách xây dựng các relationships "has many" bằng cách sử dụng factories, hãy khám phá nghịch đảo của relationship. Phương thức `for` có thể được sử dụng để định nghĩa model cha mà các models được tạo bởi factory thuộc về. Ví dụ, chúng ta có thể tạo ba instance model `App\Models\Post` thuộc về một người dùng duy nhất:

```php
use App\Models\Post;
use App\Models\User;

$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state([
        'name' => 'Jessica Archer',
    ]))
    ->create();
```

Nếu bạn đã có một instance model cha nên được liên kết với các models bạn đang tạo, bạn có thể chuyển instance model cho phương thức `for`:

```php
$user = User::factory()->create();

$posts = Post::factory()
    ->count(3)
    ->for($user)
    ->create();
```

<a name="belongs-to-relationships-using-magic-methods"></a>
#### Using Magic Methods

Để thuận tiện, bạn có thể sử dụng các phương thức relationship factory magic của Laravel để định nghĩa các relationships "belongs to". Ví dụ, ví dụ sau sẽ sử dụng quy ước để xác định rằng ba bài viết nên thuộc về relationship `user` trên model `Post`:

```php
$posts = Post::factory()
    ->count(3)
    ->forUser([
        'name' => 'Jessica Archer',
    ])
    ->create();
```

<a name="many-to-many-relationships"></a>
### Many to Many Relationships

Giống như [has many relationships](#has-many-relationships), các relationships "many to many" có thể được tạo bằng cách sử dụng phương thức `has`:

```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->has(Role::factory()->count(3))
    ->create();
```

<a name="pivot-table-attributes"></a>
#### Pivot Table Attributes

Nếu bạn cần định nghĩa các thuộc tính nên được đặt trên bảng pivot / trung gian liên kết các models, bạn có thể sử dụng phương thức `hasAttached`. Phương thức này chấp nhận một array tên và giá trị thuộc tính bảng pivot làm đối số thứ hai của nó:

```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();
```

Bạn có thể cung cấp một chuyển đổi state dựa trên closure nếu thay đổi state của bạn yêu cầu quyền truy cập vào model liên quan:

```php
$user = User::factory()
    ->hasAttached(
        Role::factory()
            ->count(3)
            ->state(function (array $attributes, User $user) {
                return ['name' => $user->name.' Role'];
            }),
        ['active' => true]
    )
    ->create();
```

Bạn cũng có thể chuyển một array các arrays pivot để cung cấp dữ liệu pivot duy nhất cho mỗi model liên quan:

```php
$user = User::factory()
    ->hasAttached(
        Role::factory(),
        [
            ['active' => true],
            ['active' => false],
        ]
    )
    ->create();
```

Nếu bạn đã có các instance model mà bạn muốn được gắn vào các models bạn đang tạo, bạn có thể chuyển các instance model cho phương thức `hasAttached`. Trong ví dụ này, cùng ba roles sẽ được gắn vào tất cả ba người dùng:

```php
$roles = Role::factory()->count(3)->create();

$users = User::factory()
    ->count(3)
    ->hasAttached($roles, ['active' => true])
    ->create();
```

<a name="many-to-many-relationships-using-magic-methods"></a>
#### Using Magic Methods

Để thuận tiện, bạn có thể sử dụng các phương thức relationship factory magic của Laravel để định nghĩa các relationships many to many. Ví dụ, ví dụ sau sẽ sử dụng quy ước để xác định rằng các models liên quan nên được tạo thông qua một phương thức relationship `roles` trên model `User`:

```php
$user = User::factory()
    ->hasRoles(1, [
        'name' => 'Editor'
    ])
    ->create();
```

<a name="polymorphic-relationships"></a>
### Polymorphic Relationships

[Polymorphic relationships](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) cũng có thể được tạo bằng cách sử dụng factories. Các relationships polymorphic "morph many" được tạo theo cùng một cách như các relationships "has many" điển hình. Ví dụ, nếu model `App\Models\Post` có một relationship `morphMany` với model `App\Models\Comment`:

```php
use App\Models\Post;

$post = Post::factory()->hasComments(3)->create();
```

<a name="morph-to-relationships"></a>
#### Morph To Relationships

Các phương thức magic không thể được sử dụng để tạo các relationships `morphTo`. Thay vào đó, phương thức `for` phải được sử dụng trực tiếp và tên của relationship phải được cung cấp rõ ràng. Ví dụ, hãy tưởng tượng rằng model `Comment` có một phương thức `commentable` định nghĩa một relationship `morphTo`. Trong tình huống này, chúng ta có thể tạo ba bình luận thuộc về một bài viết duy nhất bằng cách sử dụng trực tiếp phương thức `for`:

```php
$comments = Comment::factory()->count(3)->for(
    Post::factory(), 'commentable'
)->create();
```

<a name="polymorphic-many-to-many-relationships"></a>
#### Polymorphic Many to Many Relationships

Các relationships polymorphic "many to many" (`morphToMany` / `morphedByMany`) có thể được tạo giống như các relationships "many to many" không polymorphic:

```php
use App\Models\Tag;
use App\Models\Video;

$video = Video::factory()
    ->hasAttached(
        Tag::factory()->count(3),
        ['public' => true]
    )
    ->create();
```

Tất nhiên, phương thức magic `has` cũng có thể được sử dụng để tạo các relationships polymorphic "many to many":

```php
$video = Video::factory()
    ->hasTags(3, ['public' => true])
    ->create();
```

<a name="defining-relationships-within-factories"></a>
### Defining Relationships Within Factories

Để định nghĩa một relationship trong model factory của bạn, bạn thường sẽ gán một instance factory mới cho foreign key của relationship. Điều này thường được thực hiện cho các relationships "inverse" như các relationships `belongsTo` và `morphTo`. Ví dụ, nếu bạn muốn tạo một người dùng mới khi tạo một bài viết, bạn có thể làm như sau:

```php
use App\Models\User;

/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'title' => fake()->title(),
        'content' => fake()->paragraph(),
    ];
}
```

Nếu các cột của relationship phụ thuộc vào factory định nghĩa nó, bạn có thể gán một closure cho một thuộc tính. Closure sẽ nhận array thuộc tính được đánh giá của factory:

```php
/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'user_type' => function (array $attributes) {
            return User::find($attributes['user_id'])->type;
        },
        'title' => fake()->title(),
        'content' => fake()->paragraph(),
    ];
}
```

<a name="recycling-an-existing-model-for-relationships"></a>
### Recycling an Existing Model for Relationships

Nếu bạn có các models chia sẻ một relationship chung với một model khác, bạn có thể sử dụng phương thức `recycle` để đảm bảo một instance duy nhất của model liên quan được tái sử dụng cho tất cả các relationships được tạo bởi factory.

Ví dụ, hãy tưởng tượng bạn có các models `Airline`, `Flight`, và `Ticket`, nơi vé thuộc về một hãng hàng và một chuyến bay, và chuyến bay cũng thuộc về một hãng hàng. Khi tạo vé, bạn có thể muốn cùng một hãng hàng cho cả vé và chuyến bay, vì vậy bạn có thể chuyển một instance airline cho phương thức `recycle`:

```php
Ticket::factory()
    ->recycle(Airline::factory()->create())
    ->create();
```

Bạn có thể thấy phương thức `recycle` đặc biệt hữu ích nếu bạn có các models thuộc về một người dùng hoặc team chung.

Phương thức `recycle` cũng chấp nhận một collection các models hiện có. Khi một collection được cung cấp cho phương thức `recycle`, một model ngẫu nhiên từ collection sẽ được chọn khi factory cần một model của loại đó:

```php
Ticket::factory()
    ->recycle($airlines)
    ->create();
```

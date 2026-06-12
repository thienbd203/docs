# Eloquent: API Resources

- [Introduction](#introduction)
- [Generating Resources](#generating-resources)
- [Concept Overview](#concept-overview)
    - [Resource Collections](#resource-collections)
- [Writing Resources](#writing-resources)
    - [Data Wrapping](#data-wrapping)
    - [Pagination](#pagination)
    - [Conditional Attributes](#conditional-attributes)
    - [Conditional Relationships](#conditional-relationships)
    - [Adding Meta Data](#adding-meta-data)
- [JSON:API Resources](#jsonapi-resources)
    - [Generating JSON:API Resources](#generating-jsonapi-resources)
    - [Defining Attributes](#defining-jsonapi-attributes)
    - [Defining Relationships](#defining-jsonapi-relationships)
    - [Resource Type and ID](#jsonapi-resource-type-and-id)
    - [Sparse Fieldsets and Includes](#jsonapi-sparse-fieldsets-and-includes)
    - [Links and Meta](#jsonapi-links-and-meta)
- [Resource Responses](#resource-responses)

<a name="introduction"></a>
## Introduction

Khi xây dựng một API, bạn có thể cần một lớp transformation nằm giữa các Eloquent models của bạn và các phản hồi JSON thực sự được trả về cho người dùng ứng dụng của bạn. Ví dụ, bạn có thể muốn hiển thị một số thuộc tính cho một tập hợp con người dùng và không phải những người khác, hoặc bạn có thể muốn luôn bao gồm một số relationships nhất định trong biểu diễn JSON của các models của bạn. Các lớp resource của Eloquent cho phép bạn chuyển đổi các models và model collections của mình thành JSON một cách biểu đạt và dễ dàng.

Tất nhiên, bạn luôn có thể chuyển đổi các Eloquent models hoặc collections thành JSON bằng cách sử dụng các phương thức `toJson` của chúng; tuy nhiên, các resources của Eloquent cung cấp kiểm soát chi tiết và mạnh mẽ hơn về serialization JSON của các models và relationships của bạn.

<a name="generating-resources"></a>
## Generating Resources

Để tạo một lớp resource, bạn có thể sử dụng lệnh Artisan `make:resource`. Theo mặc định, resources sẽ được đặt trong thư mục `app/Http/Resources` của ứng dụng. Resources mở rộng lớp `Illuminate\Http\Resources\Json\JsonResource`:

```shell
php artisan make:resource UserResource
```

<a name="generating-resource-collections"></a>
#### Resource Collections

Ngoài việc tạo các resources chuyển đổi các models riêng lẻ, bạn có thể tạo các resources chịu trách nhiệm chuyển đổi các collections của models. Điều này cho phép các phản hồi JSON của bạn bao gồm các links và thông tin meta khác liên quan đến toàn bộ một collection của một resource nhất định.

Để tạo một resource collection, bạn nên sử dụng cờ `--collection` khi tạo resource. Hoặc, bao gồm từ `Collection` trong tên resource sẽ chỉ định cho Laravel rằng nó nên tạo một resource collection. Resource collections mở rộng lớp `Illuminate\Http\Resources\Json\ResourceCollection`:

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

<a name="concept-overview"></a>
## Concept Overview

> [!NOTE]
> Đây là tổng quan cấp cao về resources và resource collections. Bạn được khuyến khích mạnh mời đọc các phần khác của tài liệu này để có hiểu biết sâu hơn về tùy chỉnh và sức mạnh được cung cấp cho bạn bởi resources.

Trước khi đi sâu vào tất cả các tùy chọn có sẵn cho bạn khi viết resources, hãy trước tiên xem tổng quan cấp cao về cách resources được sử dụng trong Laravel. Một lớp resource đại diện cho một model duy nhất cần được chuyển đổi thành một cấu trúc JSON. Ví dụ, đây là một lớp resource `UserResource` đơn giản:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

Mỗi lớp resource định nghĩa một phương thức `toArray` trả về array các thuộc tính nên được chuyển đổi thành JSON khi resource được trả về như một phản hồi từ một route hoặc phương thức controller.

Lưu ý rằng chúng ta có thể truy cập trực tiếp các thuộc tính model từ biến `$this`. Điều này là vì một lớp resource sẽ tự động proxy quyền truy cập thuộc tính và phương thức xuống model bên dưới để truy cập thuận tiện. Sau khi resource được định nghĩa, nó có thể được trả về từ một route hoặc controller. Resource chấp nhận instance model bên dưới thông qua constructor của nó:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

Để thuận tiện, bạn có thể sử dụng phương thức `toResource` của model, sẽ sử dụng các quy ước framework để tự động khám phá resource bên dưới của model:

```php
return User::findOrFail($id)->toResource();
```

Khi gọi phương thức `toResource`, Laravel sẽ cố gắng định vị một resource khớp với tên model và tùy chọn được hậu tố bằng `Resource` trong namespace `Http\Resources` gần nhất với namespace của model.

Nếu lớp resource của bạn không tuân theo quy ước đặt tên này hoặc nằm trong một namespace khác, bạn có thể chỉ định resource mặc định cho model bằng cách sử dụng attribute `UseResource`:

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserResource;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResource;

#[UseResource(CustomUserResource::class)]
class User extends Model
{
    // ...
}
```

Ngoài ra, bạn có thể chỉ định lớp resource bằng cách chuyển nó cho phương thức `toResource`:

```php
return User::findOrFail($id)->toResource(CustomUserResource::class);
```

<a name="resource-collections"></a>
### Resource Collections

Nếu bạn đang trả về một collection của resources hoặc một phản hồi được phân trang, bạn nên sử dụng phương thức `collection` được cung cấp bởi lớp resource của bạn khi tạo instance resource trong route hoặc controller của bạn:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

Hoặc, để thuận tiện, bạn có thể sử dụng phương thức `toResourceCollection` của Eloquent collection, sẽ sử dụng các quy ước framework để tự động khám phá resource collection bên dưới của model:

```php
return User::all()->toResourceCollection();
```

Khi gọi phương thức `toResourceCollection`, Laravel sẽ cố gắng định vị một resource collection khớp với tên model và được hậu tố bằng `Collection` trong namespace `Http\Resources` gần nhất với namespace của model.

Nếu lớp resource collection của bạn không tuân theo quy ước đặt tên này hoặc nằm trong một namespace khác, bạn có thể chỉ định resource collection mặc định cho model bằng cách sử dụng attribute `UseResourceCollection`:

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserCollection;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResourceCollection;

#[UseResourceCollection(CustomUserCollection::class)]
class User extends Model
{
    // ...
}
```

Ngoài ra, bạn có thể chỉ định lớp resource collection bằng cách chuyển nó cho phương thức `toResourceCollection`:

```php
return User::all()->toResourceCollection(CustomUserCollection::class);
```

<a name="custom-resource-collections"></a>
#### Custom Resource Collections

Theo mặc định, resource collections không cho phép bất kỳ thêm meta data tùy chỉnh nào có thể cần được trả về với collection của bạn. Nếu bạn muốn tùy chỉnh phản hồi resource collection, bạn có thể tạo một resource chuyên dụng để đại diện cho collection:

```shell
php artisan make:resource UserCollection
```

Sau khi lớp resource collection đã được tạo, bạn có thể dễ dàng định nghĩa bất kỳ meta data nào nên được bao gồm với phản hồi:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<int|string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

Sau khi định nghĩa resource collection của bạn, nó có thể được trả về từ một route hoặc controller:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

Hoặc, để thuận tiện, bạn có thể sử dụng phương thức `toResourceCollection` của Eloquent collection, sẽ sử dụng các quy ước framework để tự động khám phá resource collection bên dưới của model:

```php
return User::all()->toResourceCollection();
```

Khi gọi phương thức `toResourceCollection`, Laravel sẽ cố gắng định vị một resource collection khớp với tên model và được hậu tố bằng `Collection` trong namespace `Http\Resources` gần nhất với namespace của model.

<a name="preserving-collection-keys"></a>
#### Preserving Collection Keys

Khi trả về một resource collection từ một route, Laravel đặt lại các keys của collection để chúng theo thứ tự số. Tuy nhiên, bạn có thể sử dụng attribute `PreserveKeys` trên lớp resource của bạn chỉ định xem các keys gốc của collection nên được giữ lại hay không:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Attributes\PreserveKeys;
use Illuminate\Http\Resources\Json\JsonResource;

#[PreserveKeys]
class UserResource extends JsonResource
{
    // ...
}
```

Khi thuộc tính `preserveKeys` được đặt thành `true`, các keys collection sẽ được giữ lại khi collection được trả về từ một route hoặc controller:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

<a name="customizing-the-underlying-resource-class"></a>
#### Customizing the Underlying Resource Class

Thông thường, thuộc tính `$this->collection` của một resource collection được tự động điền với kết quả của ánh xạ mỗi mục của collection sang lớp resource singular của nó. Lớp resource singular được giả định là tên lớp của collection mà không có phần `Collection` ở cuối tên lớp. Ngoài ra, tùy thuộc vào sở thích cá nhân của bạn, lớp resource singular có thể hoặc không được hậu tố bằng `Resource`.

Ví dụ, `UserCollection` sẽ cố gắng ánh xạ các instance người dùng đã cho vào resource `UserResource`. Để tùy chỉnh hành vi này, bạn có thể sử dụng attribute `Collects` trên resource collection của bạn:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Attributes\Collects;
use Illuminate\Http\Resources\Json\ResourceCollection;

#[Collects(Member::class)]
class UserCollection extends ResourceCollection
{
    // ...
}
```

<a name="writing-resources"></a>
## Writing Resources

> [!NOTE]
> Nếu bạn chưa đọc [tổng quan khái niệm](#concept-overview), bạn được khuyến khích mạnh mời làm điều đó trước khi tiếp tục với tài liệu này.

Resources chỉ cần chuyển đổi một model nhất định thành một array. Vì vậy, mỗi resource chứa một phương thức `toArray` chuyển đổi các thuộc tính của model thành một array thân thiện với API có thể được trả về từ các routes hoặc controllers của ứng dụng của bạn:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

Sau khi một resource đã được định nghĩa, nó có thể được trả về trực tiếp từ một route hoặc controller:

```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toUserResource();
});
```

<a name="relationships"></a>
#### Relationships

Nếu bạn muốn bao gồm các resources liên quan trong phản hồi của mình, bạn có thể thêm chúng vào array được trả về bởi phương thức `toArray` của resource. Trong ví dụ này, chúng tôi sẽ sử dụng phương thức `collection` của resource `PostResource` để thêm các bài viết blog của người dùng vào phản hồi resource:

```php
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

> [!NOTE]
> Nếu bạn muốn bao gồm các relationships chỉ khi chúng đã được tải, hãy xem tài liệu về [conditional relationships](#conditional-relationships).

<a name="writing-resource-collections"></a>
#### Resource Collections

Trong khi resources chuyển đổi một model duy nhất thành một array, resource collections chuyển đổi một collection của models thành một array. Tuy nhiên, không hoàn toàn cần thiết phải định nghĩa một lớp resource collection cho mỗi một trong các models của bạn vì tất cả các Eloquent model collections đều cung cấp một phương thức `toResourceCollection` để tạo một resource collection "ad-hoc" trên fly:

```php
use App\Models\User;

Route::get('/users', function () {
    return User::all()->toResourceCollection();
});
```

Tuy nhiên, nếu bạn cần tùy chỉnh meta data được trả về với collection, cần thiết phải định nghĩa resource collection của riêng bạn:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

Giống như các resources singular, resource collections có thể được trả về trực tiếp từ routes hoặc controllers:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

Hoặc, để thuận tiện, bạn có thể sử dụng phương thức `toResourceCollection` của Eloquent collection, sẽ sử dụng các quy ước framework để tự động khám phá resource collection bên dưới của model:

```php
return User::all()->toResourceCollection();
```

Khi gọi phương thức `toResourceCollection`, Laravel sẽ cố gắng định vị một resource collection khớp với tên model và được hậu tố bằng `Collection` trong namespace `Http\Resources` gần nhất với namespace của model.

<a name="data-wrapping"></a>
### Data Wrapping

Theo mặc định, resource ngoài cùng của bạn được bọc trong một key `data` khi phản hồi resource được chuyển đổi thành JSON. Vì vậy, ví dụ, một phản hồi resource collection điển hình trông như sau:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ]
}
```

Nếu bạn muốn tắt việc bọc resource ngoài cùng, bạn nên gọi phương thức `withoutWrapping` trên lớp cơ bản `Illuminate\Http\Resources\Json\JsonResource`. Thông thường, bạn nên gọi phương thức này từ `AppServiceProvider` hoặc một [service provider](/docs/{{version}}/providers) khác được tải trên mọi request đến ứng dụng của bạn:

```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Support\ServiceProvider;

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
        JsonResource::withoutWrapping();
    }
}
```

> [!WARNING]
> Phương thức `withoutWrapping` chỉ ảnh hưởng đến phản hồi ngoài cùng và sẽ không xóa các keys `data` mà bạn thêm thủ công vào các resource collections của riêng mình.

<a name="wrapping-nested-resources"></a>
#### Wrapping Nested Resources

Bạn có toàn quyền tự do để xác định cách các relationships của resource của bạn được bọc. Nếu bạn muốn tất cả các resource collections được bọc trong một key `data`, bất kể việc lồng ghép của chúng, bạn nên định nghĩa một lớp resource collection cho mỗi resource và trả về collection trong một key `data`.

Bạn có thể tự hỏi liệu điều này sẽ gây ra resource ngoài cùng của bạn được bọc trong hai keys `data` hay không. Đừng lo lắng, Laravel sẽ không bao giờ để các resources của bạn bị bọc nhầm hai lần, vì vậy bạn không cần lo lắng về mức độ lồng ghép của resource collection mà bạn đang chuyển đổi:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return ['data' => $this->collection];
    }
}
```

<a name="data-wrapping-and-pagination"></a>
#### Data Wrapping and Pagination

Khi trả về các collections được phân trang thông qua một phản hồi resource, Laravel sẽ bọc dữ liệu resource của bạn trong một key `data` ngay cả khi phương thức `withoutWrapping` đã được gọi. Điều này là do các phản hồi được phân trang luôn chứa các keys `meta` và `links` với thông tin về trạng thái của paginator:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="pagination"></a>
### Pagination

Bạn có thể chuyển một instance paginator Laravel cho phương thức `collection` của một resource hoặc cho một resource collection tùy chỉnh:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```

Hoặc, để thuận tiện, bạn có thể sử dụng phương thức `toResourceCollection` của paginator, sẽ sử dụng các quy ước framework để tự động khám phá resource collection bên dưới của model được phân trang:

```php
return User::paginate()->toResourceCollection();
```

Các phản hồi được phân trang luôn chứa các keys `meta` và `links` với thông tin về trạng thái của paginator:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="customizing-the-pagination-information"></a>
#### Customizing the Pagination Information

Nếu bạn muốn tùy chỉnh thông tin được bao gồm trong các keys `links` hoặc `meta` của phản hồi phân trang, bạn có thể định nghĩa một phương thức `paginationInformation` trên resource. Phương thức này sẽ nhận dữ liệu `$paginated` và array `$default` thông tin, là một array chứa các keys `links` và `meta`:

```php
/**
 * Customize the pagination information for the resource.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  array  $paginated
 * @param  array  $default
 * @return array
 */
public function paginationInformation($request, $paginated, $default)
{
    $default['links']['custom'] = 'https://example.com';

    return $default;
}
```

<a name="conditional-attributes"></a>
### Conditional Attributes

Đôi khi bạn có thể muốn chỉ bao gồm một thuộc tính trong một phản hồi resource nếu một điều kiện nhất định được đáp ứng. Ví dụ, bạn có thể muốn chỉ bao gồm một giá trị nếu người dùng hiện tại là một "administrator". Laravel cung cấp nhiều phương thức helper để hỗ trợ bạn trong tình huống này. Phương thức `when` có thể được sử dụng để thêm có điều kiện một thuộc tính vào một phản hồi resource:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

Trong ví dụ này, key `secret` sẽ chỉ được trả về trong phản hồi resource cuối cùng nếu phương thức `isAdmin` của người dùng được xác thực trả về `true`. Nếu phương thức trả về `false`, key `secret` sẽ bị xóa khỏi phản hồi resource trước khi nó được gửi cho client. Phương thức `when` cho phép bạn định nghĩa các resources của mình một cách biểu đạt mà không cần dùng đến các câu lệnh điều kiện khi xây dựng array.

Phương thức `when` cũng chấp nhận một closure làm đối số thứ hai của nó, cho phép bạn tính toán giá trị kết quả chỉ khi điều kiện đã cho là `true`:

```php
'secret' => $this->when($request->user()->isAdmin(), function () {
    return 'secret-value';
}),
```

Phương thức `whenHas` có thể được sử dụng để bao gồm một thuộc tính nếu nó thực sự có mặt trên model bên dưới:

```php
'name' => $this->whenHas('name'),
```

Ngoài ra, phương thức `whenNotNull` có thể được sử dụng để bao gồm một thuộc tính trong phản hồi resource nếu thuộc tính không phải là null:

```php
'name' => $this->whenNotNull($this->name),
```

<a name="merging-conditional-attributes"></a>
#### Merging Conditional Attributes

Đôi khi bạn có thể có một số thuộc tính nên chỉ được bao gồm trong phản hồi resource dựa trên cùng một điều kiện. Trong trường hợp này, bạn có thể sử dụng phương thức `mergeWhen` để bao gồm các thuộc tính trong phản hồi chỉ khi điều kiện đã cho là `true`:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen($request->user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

Một lần nữa, nếu điều kiện đã cho là `false`, các thuộc tính này sẽ bị xóa khỏi phản hồi resource trước khi nó được gửi cho client.

> [!WARNING]
> Phương thức `mergeWhen` không nên được sử dụng trong các arrays trộn các keys chuỗi và số. Ngoài ra, nó không nên được sử dụng trong các arrays với các keys số không được sắp xếp theo trình tự.

<a name="conditional-relationships"></a>
### Conditional Relationships

Ngoài việc tải có điều kiện các thuộc tính, bạn có thể bao gồm có điều kiện các relationships trên các phản hồi resource của bạn dựa trên việc relationship đã được tải trên model hay chưa. Điều này cho phép controller của bạn quyết định relationships nào nên được tải trên model và resource của bạn có thể dễ dàng bao gồm chúng chỉ khi chúng thực sự đã được tải. Cuối cùng, điều này giúp dễ dàng tránh các vấn đề truy vấn "N+1" trong các resources của bạn.

Phương thức `whenLoaded` có thể được sử dụng để tải có điều kiện một relationship. Để tránh tải relationships không cần thiết, phương thức này chấp nhận tên của relationship thay vì chính relationship:

```php
use App\Http\Resources\PostResource;

/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

Trong ví dụ này, nếu relationship chưa được tải, key `posts` sẽ bị xóa khỏi phản hồi resource trước khi nó được gửi cho client.

<a name="conditional-relationship-counts"></a>
#### Conditional Relationship Counts

Ngoài việc bao gồm có điều kiện các relationships, bạn có thể bao gồm có điều kiện các "counts" relationship trên các phản hồi resource của bạn dựa trên việc count của relationship đã được tải trên model hay chưa:

```php
new UserResource($user->loadCount('posts'));
```

Phương thức `whenCounted` có thể được sử dụng để bao gồm có điều kiện count của một relationship trong phản hồi resource của bạn. Phương thức này tránh bao gồm không cần thiết thuộc tính nếu count của các relationships không có mặt:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts_count' => $this->whenCounted('posts'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

Trong ví dụ này, nếu count của relationship `posts` chưa được tải, key `posts_count` sẽ bị xóa khỏi phản hồi resource trước khi nó được gửi cho client.

Các loại aggregates khác, chẳng hạn như `avg`, `sum`, `min`, và `max` cũng có thể được tải có điều kiện bằng cách sử dụng phương thức `whenAggregated`:

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
'words_min' => $this->whenAggregated('posts', 'words', 'min'),
'words_max' => $this->whenAggregated('posts', 'words', 'max'),
```

<a name="conditional-pivot-information"></a>
#### Conditional Pivot Information

Ngoài việc bao gồm có điều kiện thông tin relationship trong các phản hồi resource của bạn, bạn có thể bao gồm có điều kiện dữ liệu từ các bảng trung gian của các relationships many-to-many bằng cách sử dụng phương thức `whenPivotLoaded`. Phương thức `whenPivotLoaded` chấp nhận tên bảng pivot làm đối số đầu tiên của nó. Đối số thứ hai nên là một closure trả về giá trị được trả về nếu thông tin pivot có sẵn trên model:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_user', function () {
            return $this->pivot->expires_at;
        }),
    ];
}
```

Nếu relationship của bạn đang sử dụng một [model bảng trung gian tùy chỉnh](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models), bạn có thể chuyển một instance của model bảng trung gian làm đối số đầu tiên cho phương thức `whenPivotLoaded`:

```php
'expires_at' => $this->whenPivotLoaded(new Membership, function () {
    return $this->pivot->expires_at;
}),
```

Nếu bảng trung gian của bạn đang sử dụng một accessor khác với `pivot`, bạn có thể sử dụng phương thức `whenPivotLoadedAs`:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
            return $this->subscription->expires_at;
        }),
    ];
}
```

<a name="adding-meta-data"></a>
### Adding Meta Data

Một số tiêu chuẩn JSON API yêu cầu thêm meta data vào các phản hồi resource và resource collections của bạn. Điều này thường bao gồm những thứ như `links` đến resource hoặc các resources liên quan, hoặc meta data về chính resource. Nếu bạn cần trả về meta data bổ sung về một resource, bao gồm nó trong phương thức `toArray` của bạn. Ví dụ, bạn có thể bao gồm thông tin `links` khi chuyển đổi một resource collection:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```

Khi trả về meta data bổ sung từ các resources của bạn, bạn không bao giờ phải lo lắng về việc vô tình ghi đè các keys `links` hoặc `meta` được thêm tự động bởi Laravel khi trả về các phản hồi được phân trang. Bất kỳ `links` bổ sung nào bạn định nghĩa sẽ được hợp nhất với các links được cung cấp bởi paginator.

<a name="top-level-meta-data"></a>
#### Top Level Meta Data

Đôi khi bạn có thể muốn chỉ bao gồm một số meta data nhất định với một phản hồi resource nếu resource là resource ngoài cùng được trả về. Thông thường, điều này bao gồm thông tin meta về phản hồi như một toàn bộ. Để định nghĩa meta data này, thêm một phương thức `with` vào lớp resource của bạn. Phương thức này nên trả về một array meta data để được bao gồm với phản hồi resource chỉ khi resource là resource ngoài cùng đang được chuyển đổi:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }

    /**
     * Get additional data that should be returned with the resource array.
     *
     * @return array<string, mixed>
     */
    public function with(Request $request): array
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }
}
```

<a name="adding-meta-data-when-constructing-resources"></a>
#### Adding Meta Data When Constructing Resources

Bạn cũng có thể thêm dữ liệu cấp cao nhất khi xây dựng các instance resource trong route hoặc controller của bạn. Phương thức `additional`, có sẵn trên tất cả các resources, chấp nhận một array dữ liệu nên được thêm vào phản hồi resource:

```php
return User::all()
    ->load('roles')
    ->toResourceCollection()
    ->additional(['meta' => [
        'key' => 'value',
    ]]);
```

<a name="jsonapi-resources"></a>
## JSON:API Resources

Laravel được gửi kèm với `JsonApiResource`, một lớp resource tạo ra các phản hồi tuân thủ [specification JSON:API](https://jsonapi.org/). Nó mở rộng lớp `JsonResource` tiêu chuẩn và tự động xử lý cấu trúc đối tượng resource, relationships, sparse fieldsets, includes, đánh giá thuộc tính lười biếng, và đặt header `Content-Type` thành `application/vnd.api+json`.

> [!NOTE]
> Các resources JSON:API của Laravel xử lý serialization của các phản hồi của bạn. Nếu bạn cũng cần phân tích các tham số truy vấn JSON:API đến như filters và sorts, [Laravel Query Builder của Spatie](https://spatie.be/docs/laravel-query-builder) là một package đồng hành tuyệt vời.

<a name="generating-jsonapi-resources"></a>
### Generating JSON:API Resources

Để tạo một resource JSON:API, sử dụng lệnh Artisan `make:resource` với cờ `--json-api`:

```shell
php artisan make:resource PostResource --json-api
```

Lớp được tạo sẽ mở rộng `Illuminate\Http\Resources\JsonApi\JsonApiResource` và bao gồm các thuộc tính `$attributes` và `$relationships` để bạn định nghĩa:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\JsonApi\JsonApiResource;

class PostResource extends JsonApiResource
{
    /**
     * The resource's attributes.
     */
    public $attributes = [
        // ...
    ];

    /**
     * The resource's relationships.
     */
    public $relationships = [
        // ...
    ];
}
```

Các resources JSON:API có thể được trả về từ routes và controllers giống như các resources tiêu chuẩn:

```php
use App\Http\Resources\PostResource;
use App\Models\Post;

Route::get('/api/posts/{post}', function (Post $post) {
    return new PostResource($post);
});
```

Hoặc, để thuận tiện, bạn có thể sử dụng phương thức `toResource` của model:

```php
Route::get('/api/posts/{post}', function (Post $post) {
    return $post->toResource();
});
```

Điều này sẽ tạo ra một phản hồi tuân thủ JSON:API:

```json
{
    "data": {
        "id": "1",
        "type": "posts",
        "attributes": {
            "title": "Hello World",
            "body": "This is my first post."
        }
    }
}
```

Để trả về một collection các resources JSON:API, sử dụng phương thức `collection` hoặc phương thức thuận tiện `toResourceCollection`:

```php
return PostResource::collection(Post::all());

return Post::all()->toResourceCollection();
```

<a name="defining-jsonapi-attributes"></a>
### Defining Attributes

Có hai cách để định nghĩa các thuộc tính được bao gồm trong resource JSON:API của bạn.

Cách tiếp cận đơn giản nhất là định nghĩa một thuộc tính `$attributes` trên resource của bạn. Bạn có thể liệt kê tên thuộc tính làm giá trị, sẽ được đọc trực tiếp từ model bên dưới:

```php
public $attributes = [
    'title',
    'body',
    'created_at',
];
```

Nếu một thuộc tính tốn kém để tính toán, bạn có thể trả về nó từ `toAttributes` như một closure để nó chỉ được đánh giá khi thuộc tính thực sự cần thiết trong phản hồi.

Hoặc, để kiểm soát đầy đủ các thuộc tính của resource, bạn có thể ghi đè phương thức `toAttributes` trên resource:

```php
/**
 * Get the resource's attributes.
 *
 * @return array<string, mixed>
 */
public function toAttributes(Request $request): array
{
    return [
        'title' => $this->title,
        'body' => $this->body,
        'is_published' => fn () => $this->published_at !== null,
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

<a name="defining-jsonapi-relationships"></a>
### Defining Relationships

Các resources JSON:API hỗ trợ định nghĩa các relationships tuân thủ specification JSON:API. Các relationships chỉ được serialized khi được yêu cầu bởi client thông qua tham số truy vấn `include`.

#### The `$relationships` Property

Bạn có thể định nghĩa các relationships có thể bao gồm của resource của bạn thông qua thuộc tính `$relationships` trên resource:

```php
public $relationships = [
    'author',
    'comments',
];
```

Khi liệt kê tên relationship làm giá trị, Laravel sẽ giải quyết relationship Eloquent tương ứng và tự động khám phá lớp resource thích hợp. Nếu bạn cần chỉ định rõ lớp resource, bạn có thể định nghĩa relationship như một cặp key / class:

```php
use App\Http\Resources\UserResource;

public $relationships = [
    'author' => UserResource::class,
    'comments',
];
```

Ngoài ra, bạn có thể ghi đè phương thức `toRelationships` trên resource:

```php
/**
 * Get the resource's relationships.
 */
public function toRelationships(Request $request): array
{
    return [
        'author' => UserResource::class,
        'comments' => fn () => CommentResource::collection(
            $request->user()->is($this->resource)
                ? $this->comments
                : $this->comments->where('is_public', true),
        ),
    ];
}
```

Sử dụng closures cho bạn nhiều quyền kiểm soát hơn về payload relationship, trong khi vẫn chỉ giải quyết relationship khi client yêu cầu nó.

#### Including Relationships

Clients có thể yêu cầu các resources liên quan bằng cách sử dụng tham số truy vấn `include`:

```
GET /api/posts/1?include=author,comments
```

Điều này tạo ra một phản hồi với các đối tượng định danh tài nguyên trong key `relationships` và các đối tượng resource đầy đủ trong array `included` cấp cao nhất:

```json
{
    "data": {
        "id": "1",
        "type": "posts",
        "attributes": {
            "title": "Hello World"
        },
        "relationships": {
            "author": {
                "data": {
                    "id": "1",
                    "type": "users"
                }
            },
            "comments": {
                "data": [
                    {
                        "id": "1",
                        "type": "comments"
                    }
                ]
            }
        }
    },
    "included": [
        {
            "id": "1",
            "type": "users",
            "attributes": {
                "name": "Taylor Otwell"
            }
        },
        {
            "id": "1",
            "type": "comments",
            "attributes": {
                "body": "Great post!"
            }
        }
    ]
}
```

Các relationships lồng nhau có thể được bao gồm bằng cách sử dụng ký hiệu chấm:

```
GET /api/posts/1?include=comments.author
```

<a name="jsonapi-relationship-depth"></a>
#### Relationship Depth

Theo mặc định, các includes relationship lồng nhau bị giới hạn ở độ sâu tối đa. Bạn có thể tùy chỉnh giới hạn này bằng cách sử dụng phương thức `maxRelationshipDepth`, thường trong một trong các service providers của ứng dụng:

```php
use Illuminate\Http\Resources\JsonApi\JsonApiResource;

JsonApiResource::maxRelationshipDepth(3);
```

<a name="jsonapi-resource-type-and-id"></a>
### Resource Type and ID

Theo mặc định, `type` của resource được dẫn xuất từ tên lớp resource. Ví dụ, `PostResource` tạo ra type `posts` và `BlogPostResource` tạo ra `blog-posts`. `id` của resource được giải quyết từ primary key của model.

Nếu bạn cần tùy chỉnh các giá trị này, bạn có thể ghi đè các phương thức `toType` và `toId` trên resource:

```php
/**
 * Get the resource's type.
 */
public function toType(Request $request): string
{
    return 'articles';
}

/**
 * Get the resource's ID.
 */
public function toId(Request $request): string
{
    return (string) $this->uuid;
}
```

Điều này đặc biệt hữu ích khi type của resource nên khác với tên lớp của nó, chẳng hạn như khi một `AuthorResource` bọc một model `User` và nên xuất ra type `authors`.

<a name="jsonapi-sparse-fieldsets-and-includes"></a>
### Sparse Fieldsets and Includes

Các resources JSON:API hỗ trợ [sparse fieldsets](https://jsonapi.org/format/#fetching-sparse-fieldsets), cho phép clients yêu cầu chỉ các thuộc tính cụ thể cho mỗi loại resource bằng cách sử dụng tham số truy vấn `fields`:

```
GET /api/posts?fields[posts]=title,created_at&fields[users]=name
```

Điều này sẽ chỉ bao gồm các thuộc tính `title` và `created_at` cho các resources `posts`, và thuộc tính `name` cho các resources `users`.

<a name="jsonapi-ignoring-query-string"></a>
#### Ignoring the Query String

Nếu bạn muốn tắt lọc sparse fieldset cho một phản hồi resource nhất định, bạn có thể gọi phương thức `ignoreFieldsAndIncludesInQueryString`:

```php
return $post->toResource()
    ->ignoreFieldsAndIncludesInQueryString();
```

<a name="jsonapi-including-previously-loaded-relationships"></a>
#### Including Previously Loaded Relationships

Theo mặc định, các relationships chỉ được bao gồm trong phản hồi khi được yêu cầu thông qua tham số truy vấn `include`. Nếu bạn muốn bao gồm tất cả các relationships đã được eager-load trước bất kể query string, bạn có thể gọi phương thức `includePreviouslyLoadedRelationships`:

```php
return $post->load('author', 'comments')
    ->toResource()
    ->includePreviouslyLoadedRelationships();
```

<a name="jsonapi-links-and-meta"></a>
### Links and Meta

Bạn có thể thêm các links và thông tin meta vào các đối tượng resource JSON:API của bạn bằng cách ghi đè các phương thức `toLinks` và `toMeta` trên resource:

```php
/**
 * Get the resource's links.
 */
public function toLinks(Request $request): array
{
    return [
        'self' => route('api.posts.show', $this->resource),
    ];
}

/**
 * Get the resource's meta information.
 */
public function toMeta(Request $request): array
{
    return [
        'readable_created_at' => $this->created_at->diffForHumans(),
    ];
}
```

Điều này sẽ thêm các keys `links` và `meta` vào đối tượng resource trong phản hồi:

```json
{
    "data": {
        "id": "1",
        "type": "posts",
        "attributes": {
            "title": "Hello World"
        },
        "links": {
            "self": "https://example.com/api/posts/1"
        },
        "meta": {
            "readable_created_at": "2 hours ago"
        }
    }
}
```

<a name="resource-responses"></a>
## Resource Responses

Như bạn đã đọc, resources có thể được trả về trực tiếp từ routes và controllers:

```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toResource();
});
```

Tuy nhiên, đôi khi bạn cần tùy chỉnh phản hồi HTTP đi trước khi nó được gửi cho client. Có hai cách để thực hiện điều này. Đầu tiên, bạn có thể chain phương thức `response` vào resource. Phương thức này sẽ trả về một instance `Illuminate\Http\JsonResponse`, cho bạn kiểm soát đầy đủ các headers của phản hồi:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user', function () {
    return User::find(1)
        ->toResource()
        ->response()
        ->header('X-Value', 'True');
});
```

Ngoài ra, bạn có thể định nghĩa một phương thức `withResponse` trong chính resource. Phương thức này sẽ được gọi khi resource được trả về như resource ngoài cùng trong một phản hồi:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
        ];
    }

    /**
     * Customize the outgoing response for the resource.
     */
    public function withResponse(Request $request, JsonResponse $response): void
    {
        $response->header('X-Value', 'True');
    }
}
```

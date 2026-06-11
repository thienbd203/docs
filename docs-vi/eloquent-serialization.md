# Eloquent: Serialization

- [Introduction](#introduction)
- [Serializing Models and Collections](#serializing-models-and-collections)
    - [Serializing to Arrays](#serializing-to-arrays)
    - [Serializing to JSON](#serializing-to-json)
- [Hiding Attributes From JSON](#hiding-attributes-from-json)
- [Appending Values to JSON](#appending-values-to-json)
- [Date Serialization](#date-serialization)

<a name="introduction"></a>
## Introduction

Khi xây dựng API bằng Laravel, bạn thường cần chuyển đổi các models và relationships của mình thành arrays hoặc JSON. Eloquent bao gồm các phương thức thuận tiện để thực hiện các chuyển đổi này, cũng như kiểm soát các thuộc tính được bao gồm trong biểu diễn serialized của các models của bạn.

> [!NOTE]
> Để có một cách mạnh mẽ hơn để xử lý serialization JSON của Eloquent model và collection, hãy xem tài liệu về [Eloquent API resources](/docs/{{version}}/eloquent-resources).

<a name="serializing-models-and-collections"></a>
## Serializing Models and Collections

<a name="serializing-to-arrays"></a>
### Serializing to Arrays

Để chuyển đổi một model và các [relationships](/docs/{{version}}/eloquent-relationships) đã tải của nó thành một array, bạn nên sử dụng phương thức `toArray`. Phương thức này là đệ quy, vì vậy tất cả các thuộc tính và tất cả các relationships (bao gồm cả các relationships của relationships) sẽ được chuyển đổi thành arrays:

```php
use App\Models\User;

$user = User::with('roles')->first();

return $user->toArray();
```

Phương thức `attributesToArray` có thể được sử dụng để chuyển đổi các thuộc tính của một model thành một array nhưng không phải là các relationships của nó:

```php
$user = User::first();

return $user->attributesToArray();
```

Bạn cũng có thể chuyển đổi toàn bộ [collections](/docs/{{version}}/eloquent-collections) của models thành arrays bằng cách gọi phương thức `toArray` trên instance collection:

```php
$users = User::all();

return $users->toArray();
```

<a name="serializing-to-json"></a>
### Serializing to JSON

Để chuyển đổi một model thành JSON, bạn nên sử dụng phương thức `toJson`. Giống như `toArray`, phương thức `toJson` là đệ quy, vì vậy tất cả các thuộc tính và relationships sẽ được chuyển đổi thành JSON. Bạn cũng có thể chỉ định bất kỳ tùy chọn mã hóa JSON nào được [hỗ trợ bởi PHP](https://secure.php.net/manual/en/function.json-encode.php):

```php
use App\Models\User;

$user = User::find(1);

return $user->toJson();

return $user->toJson(JSON_PRETTY_PRINT);
```

Ngoài ra, bạn có thể cast một model hoặc collection thành một string, sẽ tự động gọi phương thức `toJson` trên model hoặc collection:

```php
return (string) User::find(1);
```

Vì các models và collections được chuyển đổi thành JSON khi được cast thành string, bạn có thể trả về các đối tượng Eloquent trực tiếp từ các routes hoặc controllers của ứng dụng. Laravel sẽ tự động serialize các Eloquent models và collections của bạn thành JSON khi chúng được trả về từ routes hoặc controllers:

```php
Route::get('/users', function () {
    return User::all();
});
```

<a name="relationships"></a>
#### Relationships

Khi một Eloquent model được chuyển đổi thành JSON, các relationships đã tải của nó sẽ tự động được bao gồm như các thuộc tính trên đối tượng JSON. Ngoài ra, mặc dù các phương thức relationship của Eloquent được định nghĩa bằng tên phương thức "camel case", thuộc tính JSON của một relationship sẽ là "snake case".

<a name="hiding-attributes-from-json"></a>
## Hiding Attributes From JSON

Đôi khi bạn có thể muốn giới hạn các thuộc tính, chẳng hạn như passwords, được bao gồm trong biểu diễn array hoặc JSON của model của bạn. Để làm điều này, bạn có thể sử dụng attribute `Hidden` trên model của bạn. Các thuộc tính được liệt kê trong attribute `Hidden` sẽ không được bao gồm trong biểu diễn serialized của model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Hidden;
use Illuminate\Database\Eloquent\Model;

#[Hidden(['password'])]
class User extends Model
{
    // ...
}
```


> [!NOTE]
> Để ẩn relationships, hãy thêm tên phương thức relationship vào attribute `Hidden` của Eloquent model của bạn.

Ngoài ra, bạn có thể sử dụng attribute `Visible` để định nghĩa một "allow list" của các thuộc tính nên được bao gồm trong biểu diễn array và JSON của model của bạn. Tất cả các thuộc tính không có trong attribute `Visible` sẽ bị ẩn khi model được chuyển đổi thành một array hoặc JSON:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Visible;
use Illuminate\Database\Eloquent\Model;

#[Visible(['first_name', 'last_name'])]
class User extends Model
{
    // ...
}
```

<a name="temporarily-modifying-attribute-visibility"></a>
#### Temporarily Modifying Attribute Visibility

Nếu bạn muốn làm cho một số thuộc tính thường bị ẩn trở nên nhìn thấy trên một instance model cụ thể, bạn có thể sử dụng các phương thức `makeVisible` hoặc `mergeVisible`. Phương thức `makeVisible` trả về instance model:

```php
return $user->makeVisible('attribute')->toArray();

return $user->mergeVisible(['name', 'email'])->toArray();
```

Tương tự, nếu bạn muốn ẩn một số thuộc tính thường nhìn thấy, bạn có thể sử dụng các phương thức `makeHidden` hoặc `mergeHidden`:

```php
return $user->makeHidden('attribute')->toArray();

return $user->mergeHidden(['name', 'email'])->toArray();
```

Nếu bạn muốn tạm thời ghi đè tất cả các thuộc tính nhìn thấy hoặc ẩn, bạn có thể sử dụng các phương thức `setVisible` và `setHidden` tương ứng:

```php
return $user->setVisible(['id', 'name'])->toArray();

return $user->setHidden(['email', 'password', 'remember_token'])->toArray();
```

<a name="appending-values-to-json"></a>
## Appending Values to JSON

Thỉnh thoảng, khi chuyển đổi models thành arrays hoặc JSON, bạn có thể muốn thêm các thuộc tính không có cột tương ứng trong database của bạn. Để làm điều này, trước tiên hãy định nghĩa một [accessor](/docs/{{version}}/eloquent-mutators) cho giá trị:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Determine if the user is an administrator.
     */
    protected function isAdmin(): Attribute
    {
        return new Attribute(
            get: fn () => 'yes',
        );
    }
}
```

Nếu bạn muốn accessor luôn được thêm vào biểu diễn array và JSON của model của bạn, bạn có thể sử dụng attribute `Appends` trên model của bạn. Lưu ý rằng tên thuộc tính thường được tham chiếu bằng biểu diễn serialized "snake case" của chúng, mặc dù phương thức PHP của accessor được định nghĩa bằng "camel case":

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Appends;
use Illuminate\Database\Eloquent\Model;

#[Appends(['is_admin'])]
class User extends Model
{
    // ...
}
```

Sau khi thuộc tính đã được thêm vào danh sách `appends`, nó sẽ được bao gồm trong cả biểu diễn array và JSON của model. Các thuộc tính trong array `appends` cũng sẽ tôn trọng các cài đặt `visible` và `hidden` được cấu hình trên model.

<a name="appending-at-run-time"></a>
#### Appending at Run Time

Tại runtime, bạn có thể hướng dẫn một instance model để thêm các thuộc tính bổ sung bằng cách sử dụng các phương thức `append` hoặc `mergeAppends`. Hoặc, bạn có thể sử dụng phương thức `setAppends` để ghi đè toàn bộ array của các thuộc tính được thêm cho một instance model cụ thể:

```php
return $user->append('is_admin')->toArray();

return $user->mergeAppends(['is_admin', 'status'])->toArray();

return $user->setAppends(['is_admin'])->toArray();
```

Tương tự, nếu bạn muốn xóa tất cả các thuộc tính được thêm từ một model, bạn có thể sử dụng phương thức `withoutAppends`:

```php
return $user->withoutAppends()->toArray();
```

<a name="date-serialization"></a>
## Date Serialization

<a name="customizing-the-default-date-format"></a>
#### Customizing the Default Date Format

Bạn có thể tùy chỉnh định dạng serialization mặc định bằng cách ghi đè phương thức `serializeDate`. Phương thức này không ảnh hưởng đến cách ngày tháng của bạn được định dạng để lưu trữ trong database:

```php
/**
 * Prepare a date for array / JSON serialization.
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

<a name="customizing-the-date-format-per-attribute"></a>
#### Customizing the Date Format per Attribute

Bạn có thể tùy chỉnh định dạng serialization của các thuộc tính ngày Eloquent riêng lẻ bằng cách chỉ định định dạng ngày trong các [khai báo cast](/docs/{{version}}/eloquent-mutators#attribute-casting) của model:

```php
protected function casts(): array
{
    return [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
}
```

# Prompts

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Các Prompts Có Sẵn](#available-prompts)
    - [Text](#text)
    - [Textarea](#textarea)
    - [Number](#number)
    - [Password](#password)
    - [Confirm](#confirm)
    - [Select](#select)
    - [Multi-select](#multiselect)
    - [Suggest](#suggest)
    - [Search](#search)
    - [Multi-search](#multisearch)
    - [Pause](#pause)
    - [Autocomplete](#autocomplete)
- [Chuyển đổi Input Trước Khi Validate](#transforming-input-before-validation)
- [Forms](#forms)
- [Thông báo Thông tin](#informational-messages)
- [Bảng](#tables)
- [Spin](#spin)
- [Thanh Tiến độ](#progress)
- [Task](#task)
- [Stream](#stream)
- [Tiêu đề Terminal](#terminal-title)
- [Xóa Terminal](#clear)
- [Các Lưu ý về Terminal](#terminal-considerations)
- [Môi trường Không được Hỗ trợ và Fallbacks](#fallbacks)
- [Testing](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Prompts](https://github.com/laravel/prompts) là một gói PHP để thêm các form đẹp mắt và thân thiện với người dùng vào các ứng dụng dòng lệnh của bạn, với các tính năng giống như trình duyệt bao gồm văn bản placeholder và validation.

<img src="https://laravel.com/img/docs/prompts-example.png">

Laravel Prompts rất phù hợp để chấp nhận input từ người dùng trong các [lệnh console Artisan](/docs/{{version}}/artisan#writing-commands) của bạn, nhưng nó cũng có thể được sử dụng trong bất kỳ dự án PHP dòng lệnh nào.

> [!NOTE]
> Laravel Prompts hỗ trợ macOS, Linux và Windows với WSL. Để biết thêm thông tin, vui lòng xem tài liệu của chúng tôi về [môi trường không được hỗ trợ & fallbacks](#fallbacks).

<a name="installation"></a>
## Cài đặt

Laravel Prompts đã được bao gồm trong bản phát hành mới nhất của Laravel.

Laravel Prompts cũng có thể được cài đặt trong các dự án PHP khác của bạn bằng cách sử dụng trình quản lý gói Composer:

```shell
composer require laravel/prompts
```

<a name="available-prompts"></a>
## Các Prompts Có Sẵn

<a name="text"></a>
### Text

Hàm `text` sẽ hỏi người dùng với câu hỏi đã cho, chấp nhận input của họ, và sau đó trả về nó:

```php
use function Laravel\Prompts\text;

$name = text('What is your name?');
```

Bạn cũng có thể bao gồm văn bản placeholder, giá trị mặc định, và gợi ý thông tin:

```php
$name = text(
    label: 'What is your name?',
    placeholder: 'E.g. Taylor Otwell',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="text-required"></a>
#### Giá trị Bắt buộc

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền đối số `required`:

```php
$name = text(
    label: 'What is your name?',
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể truyền một chuỗi:

```php
$name = text(
    label: 'What is your name?',
    required: 'Your name is required.'
);
```

<a name="text-validation"></a>
#### Validation Bổ sung

Cuối cùng, nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Closure sẽ nhận giá trị đã được nhập và có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) của Laravel. Để làm điều này, hãy cung cấp một mảng chứa tên của thuộc tính và các quy tắc validation mong muốn cho đối số `validate`:

```php
$name = text(
    label: 'What is your name?',
    validate: ['name' => 'required|max:255|unique:users']
);
```

<a name="textarea"></a>
### Textarea

Hàm `textarea` sẽ hỏi người dùng với câu hỏi đã cho, chấp nhận input của họ thông qua một textarea đa dòng, và sau đó trả về nó:

```php
use function Laravel\Prompts\textarea;

$story = textarea('Tell me a story.');
```

Bạn cũng có thể bao gồm văn bản placeholder, giá trị mặc định, và gợi ý thông tin:

```php
$story = textarea(
    label: 'Tell me a story.',
    placeholder: 'This is a story about...',
    hint: 'This will be displayed on your profile.'
);
```

<a name="textarea-required"></a>
#### Giá trị Bắt buộc

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền đối số `required`:

```php
$story = textarea(
    label: 'Tell me a story.',
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể truyền một chuỗi:

```php
$story = textarea(
    label: 'Tell me a story.',
    required: 'A story is required.'
);
```

<a name="textarea-validation"></a>
#### Validation Bổ sung

Cuối cùng, nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: fn (string $value) => match (true) {
        strlen($value) < 250 => 'The story must be at least 250 characters.',
        strlen($value) > 10000 => 'The story must not exceed 10,000 characters.',
        default => null
    }
);
```

Closure sẽ nhận giá trị đã được nhập và có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) của Laravel. Để làm điều này, hãy cung cấp một mảng chứa tên của thuộc tính và các quy tắc validation mong muốn cho đối số `validate`:

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: ['story' => 'required|max:10000']
);
```

<a name="number"></a>
### Number

Hàm `number` sẽ hỏi người dùng với câu hỏi đã cho, chấp nhận input số của họ, và sau đó trả về nó. Hàm `number` cho phép người dùng sử dụng các phím mũi tên lên và xuống để thao tác với số:

```php
use function Laravel\Prompts\number;

$number = number('How many copies would you like?');
```

Bạn cũng có thể bao gồm văn bản placeholder, giá trị mặc định, và gợi ý thông tin:

```php
$name = number(
    label: 'How many copies would you like?',
    placeholder: '5',
    default: 1,
    hint: 'This will be determine how many copies to create.'
);
```

<a name="number-required"></a>
#### Giá trị Bắt buộc

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền đối số `required`:

```php
$copies = number(
    label: 'How many copies would you like?',
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể truyền một chuỗi:

```php
$copies = number(
    label: 'How many copies would you like?',
    required: 'A number of copies is required.'
);
```

<a name="number-validation"></a>
#### Validation Bổ sung

Cuối cùng, nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$copies = number(
    label: 'How many copies would you like?',
    validate: fn (?int $value) => match (true) {
        $value < 1 => 'At least one copy is required.',
        $value > 100 => 'You may not create more than 100 copies.',
        default => null
    }
);
```

Closure sẽ nhận giá trị đã được nhập và có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) của Laravel. Để làm điều này, hãy cung cấp một mảng chứa tên của thuộc tính và các quy tắc validation mong muốn cho đối số `validate`:

```php
$copies = number(
    label: 'How many copies would you like?',
    validate: ['copies' => 'required|integer|min:1|max:100']
);
```

<a name="password"></a>
### Password

Hàm `password` tương tự như hàm `text`, nhưng input của người dùng sẽ được che khi họ gõ trong console. Điều này hữu ích khi hỏi thông tin nhạy cảm như mật khẩu:

```php
use function Laravel\Prompts\password;

$password = password('What is your password?');
```

Bạn cũng có thể bao gồm văn bản placeholder và gợi ý thông tin:

```php
$password = password(
    label: 'What is your password?',
    placeholder: 'password',
    hint: 'Minimum 8 characters.'
);
```

<a name="password-required"></a>
#### Giá trị Bắt buộc

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền đối số `required`:

```php
$password = password(
    label: 'What is your password?',
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể truyền một chuỗi:

```php
$password = password(
    label: 'What is your password?',
    required: 'The password is required.'
);
```

<a name="password-validation"></a>
#### Validation Bổ sung

Cuối cùng, nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$password = password(
    label: 'What is your password?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'The password must be at least 8 characters.',
        default => null
    }
);
```

Closure sẽ nhận giá trị đã được nhập và có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) của Laravel. Để làm điều này, hãy cung cấp một mảng chứa tên của thuộc tính và các quy tắc validation mong muốn cho đối số `validate`:

```php
$password = password(
    label: 'What is your password?',
    validate: ['password' => 'min:8']
);
```

<a name="confirm"></a>
### Confirm

Nếu bạn cần hỏi người dùng để xác nhận "có hoặc không", bạn có thể sử dụng hàm `confirm`. Người dùng có thể sử dụng các phím mũi tên hoặc nhấn `y` hoặc `n` để chọn câu trả lời của họ. Hàm này sẽ trả về `true` hoặc `false`.

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm('Do you accept the terms?');
```

Bạn cũng có thể bao gồm giá trị mặc định, văn bản tùy chỉnh cho các nhãn "Yes" và "No", và gợi ý thông tin:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    default: false,
    yes: 'I accept',
    no: 'I decline',
    hint: 'The terms must be accepted to continue.'
);
```

<a name="confirm-required"></a>
#### Yêu cầu "Yes"

Nếu cần thiết, bạn có thể yêu cầu người dùng của mình chọn "Yes" bằng cách truyền đối số `required`:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể truyền một chuỗi:

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: 'You must accept the terms to continue.'
);
```

<a name="select"></a>
### Select

Nếu bạn cần người dùng chọn từ một tập hợp các lựa chọn được xác định trước, bạn có thể sử dụng hàm `select`:

```php
use function Laravel\Prompts\select;

$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner']
);
```

Bạn cũng có thể chỉ định lựa chọn mặc định và gợi ý thông tin:

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'The role may be changed at any time.'
);
```

Bạn cũng có thể truyền một mảng kết hợp cho đối số `options` để trả về khóa của lựa chọn được chọn thay vì giá trị của nó:

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    default: 'owner'
);
```

Tối đa năm tùy chọn sẽ được hiển thị trước khi danh sách bắt đầu cuộn. Bạn có thể tùy chỉnh điều này bằng cách truyền đối số `scroll`:

```php
$role = select(
    label: 'Which category would you like to assign?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="select-info"></a>
#### Thông tin Phụ

Đối số `info` có thể được sử dụng để hiển thị thông tin bổ sung về tùy chọn hiện đang được làm nổi bật. Khi một closure được cung cấp, nó sẽ nhận giá trị của tùy chọn hiện đang được làm nổi bật và nên trả về một chuỗi hoặc `null`:

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    info: fn (string $value) => match ($value) {
        'member' => 'Can view and comment.',
        'contributor' => 'Can view, comment, and edit.',
        'owner' => 'Full access to all resources.',
        default => null,
    }
);
```

Bạn cũng có thể truyền một chuỗi tĩnh cho đối số `info` nếu thông tin không phụ thuộc vào tùy chọn được làm nổi bật:

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    info: 'The role may be changed at any time.'
);
```

<a name="select-validation"></a>
#### Validation Bổ sung

Khác với các hàm prompt khác, hàm `select` không chấp nhận đối số `required` vì không thể không chọn gì cả. Tuy nhiên, bạn có thể truyền một closure cho đối số `validate` nếu bạn cần hiển thị một tùy chọn nhưng ngăn việc chọn nó:

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'An owner already exists.'
            : null
);
```

Nếu đối số `options` là một mảng kết hợp, thì closure sẽ nhận khóa được chọn, nếu không nó sẽ nhận giá trị được chọn. Closure có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

<a name="multiselect"></a>
### Multi-select

Nếu bạn cần người dùng có thể chọn nhiều tùy chọn, bạn có thể sử dụng hàm `multiselect`:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete']
);
```

Bạn cũng có thể chỉ định các lựa chọn mặc định và gợi ý thông tin:

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    default: ['Read', 'Create'],
    hint: 'Permissions may be updated at any time.'
);
```

Bạn cũng có thể truyền một mảng kết hợp cho đối số `options` để trả về các khóa của các tùy chọn được chọn thay vì giá trị của chúng:

```php
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    default: ['read', 'create']
);
```

Tối đa năm tùy chọn sẽ được hiển thị trước khi danh sách bắt đầu cuộn. Bạn có thể tùy chỉnh điều này bằng cách truyền đối số `scroll`:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="multiselect-info"></a>
#### Thông tin Phụ

Đối số `info` có thể được sử dụng để hiển thị thông tin bổ sung về tùy chọn hiện đang được làm nổi bật. Khi một closure được cung cấp, nó sẽ nhận giá trị của tùy chọn hiện đang được làm nổi bật và nên trả về một chuỗi hoặc `null`:

```php
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    info: fn (string $value) => match ($value) {
        'read' => 'View resources and their properties.',
        'create' => 'Create new resources.',
        'update' => 'Modify existing resources.',
        'delete' => 'Permanently remove resources.',
        default => null,
    }
);
```

<a name="multiselect-required"></a>
#### Yêu cầu một Giá trị

Theo mặc định, người dùng có thể chọn không hoặc nhiều tùy chọn. Bạn có thể truyền đối số `required` để thực thi một hoặc nhiều tùy chọn thay thế:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn có thể cung cấp một chuỗi cho đối số `required`:

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: 'You must select at least one category'
);
```

<a name="multiselect-validation"></a>
#### Validation Bổ sung

Bạn có thể truyền một closure cho đối số `validate` nếu bạn cần hiển thị một tùy chọn nhưng ngăn việc chọn nó:

```php
$permissions = multiselect(
    label: 'What permissions should the user have?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'All users require the read permission.'
        : null
);
```

Nếu đối số `options` là một mảng kết hợp thì closure sẽ nhận các khóa được chọn, nếu không nó sẽ nhận các giá trị được chọn. Closure có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

<a name="suggest"></a>
### Suggest

Hàm `suggest` có thể được sử dụng để cung cấp tự động hoàn thành cho các lựa chọn có thể. Người dùng vẫn có thể cung cấp bất kỳ câu trả lời nào, bất kể các gợi ý tự động hoàn thành:

```php
use function Laravel\Prompts\suggest;

$name = suggest('What is your name?', ['Taylor', 'Dayle']);
```

Ngoài ra, bạn có thể truyền một closure làm đối số thứ hai cho hàm `suggest`. Closure sẽ được gọi mỗi khi người dùng gõ một ký tự input. Closure nên chấp nhận một tham số chuỗi chứa input của người dùng cho đến nay và trả về một mảng các tùy chọn để tự động hoàn thành:

```php
$name = suggest(
    label: 'What is your name?',
    options: fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

Bạn cũng có thể bao gồm văn bản placeholder, giá trị mặc định, và gợi ý thông tin:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="suggest-info"></a>
#### Thông tin Phụ

Đối số `info` có thể được sử dụng để hiển thị thông tin bổ sung về tùy chọn hiện đang được làm nổi bật. Khi một closure được cung cấp, nó sẽ nhận giá trị của tùy chọn hiện đang được làm nổi bật và nên trả về một chuỗi hoặc `null`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    info: fn (string $value) => match ($value) {
        'Taylor' => 'Administrator',
        'Dayle' => 'Contributor',
        default => null,
    }
);
```

<a name="suggest-required"></a>
#### Giá trị Bắt buộc

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền đối số `required`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể truyền một chuỗi:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: 'Your name is required.'
);
```

<a name="suggest-validation"></a>
#### Validation Bổ sung

Cuối cùng, nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Closure sẽ nhận giá trị đã được nhập và có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

Ngoài ra, bạn có thể tận dụng sức mạnh của [validator](/docs/{{version}}/validation) của Laravel. Để làm điều này, hãy cung cấp một mảng chứa tên của thuộc tính và các quy tắc validation mong muốn cho đối số `validate`:

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: ['name' => 'required|min:3|max:255']
);
```

<a name="search"></a>
### Search

Nếu bạn có nhiều tùy chọn để người dùng chọn, hàm `search` cho phép người dùng gõ một truy vấn tìm kiếm để lọc kết quả trước khi sử dụng các phím mũi tên để chọn một tùy chọn:

```php
use function Laravel\Prompts\search;

$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Closure sẽ nhận văn bản đã được người dùng gõ cho đến nay và phải trả về một mảng các tùy chọn. Nếu bạn trả về một mảng kết hợp thì khóa của tùy chọn được chọn sẽ được trả về, nếu không giá trị của nó sẽ được trả về thay thế.

Khi lọc một mảng mà bạn định trả về giá trị, bạn nên sử dụng hàm `array_values` hoặc phương thức `values` của Collection để đảm bảo mảng không trở thành kết hợp:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Bạn cũng có thể bao gồm văn bản placeholder và gợi ý thông tin:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Tối đa năm tùy chọn sẽ được hiển thị trước khi danh sách bắt đầu cuộn. Bạn có thể tùy chỉnh điều này bằng cách truyền đối số `scroll`:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="search-info"></a>
#### Thông tin Phụ

Đối số `info` có thể được sử dụng để hiển thị thông tin bổ sung về tùy chọn hiện đang được làm nổi bật. Khi một closure được cung cấp, nó sẽ nhận giá trị của tùy chọn hiện đang được làm nổi bật và nên trả về một chuỗi hoặc `null`:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

<a name="search-validation"></a>
#### Validation Bổ sung

Nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'This user has opted-out of receiving mail.';
        }
    }
);
```

Nếu closure `options` trả về một mảng kết hợp, thì closure sẽ nhận khóa được chọn, nếu không, nó sẽ nhận giá trị được chọn. Closure có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

<a name="multisearch"></a>
### Multi-search

Nếu bạn có nhiều tùy chọn có thể tìm kiếm và cần người dùng có thể chọn nhiều mục, hàm `multisearch` cho phép người dùng gõ một truy vấn tìm kiếm để lọc kết quả trước khi sử dụng các phím mũi tên và thanh khoảng trắng để chọn các tùy chọn:

```php
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

Closure sẽ nhận văn bản đã được người dùng gõ cho đến nay và phải trả về một mảng các tùy chọn. Nếu bạn trả về một mảng kết hợp thì các khóa của các tùy chọn được chọn sẽ được trả về; nếu không, giá trị của chúng sẽ được trả về thay thế.

Khi lọc một mảng mà bạn định trả về giá trị, bạn nên sử dụng hàm `array_values` hoặc phương thức `values` của Collection để đảm bảo mảng không trở thành kết hợp:

```php
$names = collect(['Taylor', 'Abigail']);

$selected = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

Bạn cũng có thể bao gồm văn bản placeholder và gợi ý thông tin:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Tối đa năm tùy chọn sẽ được hiển thị trước khi danh sách bắt đầu cuộn. Bạn có thể tùy chỉnh điều này bằng cách cung cấp đối số `scroll`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="multisearch-info"></a>
#### Thông tin Phụ

Đối số `info` có thể được sử dụng để hiển thị thông tin bổ sung về tùy chọn hiện đang được làm nổi bật. Khi một closure được cung cấp, nó sẽ nhận giá trị của tùy chọn hiện đang được làm nổi bật và nên trả về một chuỗi hoặc `null`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

<a name="multisearch-required"></a>
#### Yêu cầu một Giá trị

Theo mặc định, người dùng có thể chọn không hoặc nhiều tùy chọn. Bạn có thể truyền đối số `required` để thực thi một hoặc nhiều tùy chọn thay thế:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể cung cấp một chuỗi cho đối số `required`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: 'You must select at least one user.'
);
```

<a name="multisearch-validation"></a>
#### Validation Bổ sung

Nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::whereLike('name', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';
        }
    }
);
```

Nếu closure `options` trả về một mảng kết hợp, thì closure sẽ nhận các khóa được chọn; nếu không, nó sẽ nhận các giá trị được chọn. Closure có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

<a name="pause"></a>
### Pause

Hàm `pause` có thể được sử dụng để hiển thị văn bản thông tin cho người dùng và chờ họ xác nhận mong muốn tiếp tục bằng cách nhấn phím Enter / Return:

```php
use function Laravel\Prompts\pause;

pause('Press ENTER to continue.');
```

<a name="autocomplete"></a>
### Autocomplete

Hàm `autocomplete` có thể được sử dụng để cung cấp tự động hoàn thành nội tuyến cho các lựa chọn có thể. Khi người dùng gõ, các gợi ý khớp với input của họ sẽ xuất hiện dưới dạng văn bản ma có thể được chấp nhận bằng cách nhấn `Tab` hoặc phím mũi tên phải:

```php
use function Laravel\Prompts\autocomplete;

$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim']
);
```

Bạn cũng có thể bao gồm văn bản placeholder, giá trị mặc định, và gợi ý thông tin:

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'Use tab to accept, up/down to cycle.'
);
```

<a name="autocomplete-closure"></a>
#### Tùy chọn Động

Bạn cũng có thể truyền một closure để tạo tùy chọn động dựa trên input của người dùng. Closure sẽ được gọi mỗi khi người dùng gõ một ký tự và nên trả về một mảng các tùy chọn để tự động hoàn thành:

```php
$file = autocomplete(
    label: 'Which file?',
    options: fn (string $value) => collect($files)
        ->filter(fn ($file) => str_starts_with(strtolower($file), strtolower($value)))
        ->values()
        ->all(),
);
```

<a name="autocomplete-required"></a>
#### Giá trị Bắt buộc

Nếu bạn yêu cầu một giá trị phải được nhập, bạn có thể truyền đối số `required`:

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: true
);
```

Nếu bạn muốn tùy chỉnh thông báo validation, bạn cũng có thể truyền một chuỗi:

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: 'Your name is required.'
);
```

<a name="autocomplete-validation"></a>
#### Validation Bổ sung

Cuối cùng, nếu bạn muốn thực hiện logic validation bổ sung, bạn có thể truyền một closure cho đối số `validate`:

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

Closure sẽ nhận giá trị đã được nhập và có thể trả về thông báo lỗi, hoặc `null` nếu validation vượt qua.

<a name="transforming-input-before-validation"></a>
## Chuyển đổi Input Trước Khi Validate

Đôi khi bạn có thể muốn chuyển đổi input của prompt trước khi validation diễn ra. Ví dụ, bạn có thể muốn xóa khoảng trắng từ bất kỳ chuỗi nào được cung cấp. Để thực hiện điều này, nhiều hàm prompt cung cấp đối số `transform`, chấp nhận một closure:

```php
$name = text(
    label: 'What is your name?',
    transform: fn (string $value) => trim($value),
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

<a name="forms"></a>
## Forms

Thường thì, bạn sẽ có nhiều prompt sẽ được hiển thị theo trình tự để thu thập thông tin trước khi thực hiện các hành động bổ sung. Bạn có thể sử dụng hàm `form` để tạo một tập hợp các prompt được nhóm lại để người dùng hoàn thành:

```php
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true)
    ->password('What is your password?', validate: ['password' => 'min:8'])
    ->confirm('Do you accept the terms?')
    ->submit();
```

Phương thức `submit` sẽ trả về một mảng được lập chỉ mục số chứa tất cả các phản hồi từ các prompt của form. Tuy nhiên, bạn có thể cung cấp một tên cho mỗi prompt thông qua đối số `name`. Khi một tên được cung cấp, phản hồi của prompt được đặt tên có thể được truy cập thông qua tên đó:

```php
use App\Models\User;
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->password(
        label: 'What is your password?',
        validate: ['password' => 'min:8'],
        name: 'password'
    )
    ->confirm('Do you accept the terms?')
    ->submit();

User::create([
    'name' => $responses['name'],
    'password' => $responses['password'],
]);
```

Lợi ích chính của việc sử dụng hàm `form` là khả năng cho phép người dùng quay lại các prompt trước trong form bằng cách sử dụng `CTRL + U`. Điều này cho phép người dùng sửa lỗi hoặc thay đổi lựa chọn mà không cần hủy và khởi động lại toàn bộ form.

Nếu bạn cần kiểm soát chi tiết hơn một prompt trong form, bạn có thể gọi phương thức `add` thay vì gọi trực tiếp một trong các hàm prompt. Phương thức `add` được truyền tất cả các phản hồi trước đó được cung cấp bởi người dùng:

```php
use function Laravel\Prompts\form;
use function Laravel\Prompts\outro;
use function Laravel\Prompts\text;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->add(function ($responses) {
        return text("How old are you, {$responses['name']}?");
    }, name: 'age')
    ->submit();

outro("Your name is {$responses['name']} and you are {$responses['age']} years old.");
```

<a name="informational-messages"></a>
## Thông báo Thông tin

Các hàm `note`, `info`, `warning`, `error`, và `alert` có thể được sử dụng để hiển thị các thông báo thông tin:

```php
use function Laravel\Prompts\info;

info('Package installed successfully.');
```

<a name="tables"></a>
## Bảng

Hàm `table` giúp dễ dàng hiển thị nhiều hàng và cột dữ liệu. Tất cả những gì bạn cần làm là cung cấp tên cột và dữ liệu cho bảng:

```php
use function Laravel\Prompts\table;

table(
    headers: ['Name', 'Email'],
    rows: User::all(['name', 'email'])->toArray()
);
```

<a name="spin"></a>
## Spin

Hàm `spin` hiển thị một spinner cùng với một thông báo tùy chọn trong khi thực hiện một callback được chỉ định. Nó phục vụ để chỉ ra các quá trình đang diễn ra và trả về kết quả của callback khi hoàn thành:

```php
use function Laravel\Prompts\spin;

$response = spin(
    callback: fn () => Http::get('http://example.com'),
    message: 'Fetching response...'
);
```

> [!WARNING]
> Hàm `spin` yêu cầu phần mở rộng PHP [PCNTL](https://www.php.net/manual/en/book.pcntl.php) để tạo hoạt ảnh cho spinner. Khi phần mở rộng này không khả dụng, một phiên bản tĩnh của spinner sẽ xuất hiện thay thế.

<a name="progress"></a>
## Thanh Tiến độ

Đối với các tác vụ chạy lâu, có thể hữu ích khi hiển thị thanh tiến độ thông báo cho người dùng mức độ hoàn thành của tác vụ. Sử dụng hàm `progress`, Laravel sẽ hiển thị thanh tiến độ và tăng tiến độ của nó cho mỗi lần lặp qua một giá trị có thể lặp được:

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user)
);
```

Hàm `progress` hoạt động giống như một hàm map và sẽ trả về một mảng chứa giá trị trả về của mỗi lần lặp của callback của bạn.

Callback cũng có thể chấp nhận instance `Laravel\Prompts\Progress`, cho phép bạn sửa đổi nhãn và gợi ý trên mỗi lần lặp:

```php
$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: function ($user, $progress) {
        $progress
            ->label("Updating {$user->name}")
            ->hint("Created on {$user->created_at}");

        return $this->performTask($user);
    },
    hint: 'This may take some time.'
);
```

Đôi khi, bạn có thể cần kiểm soát thủ công nhiều hơn về cách thanh tiến độ được tăng. Đầu tiên, xác định tổng số bước mà quá trình sẽ lặp qua. Sau đó, tăng thanh tiến độ thông qua phương thức `advance` sau khi xử lý mỗi mục:

```php
$progress = progress(label: 'Updating users', steps: 10);

$users = User::all();

$progress->start();

foreach ($users as $user) {
    $this->performTask($user);

    $progress->advance();
}

$progress->finish();
```

<a name="task"></a>
## Task

Hàm `task` hiển thị một tác vụ được gắn nhãn với một spinner và một vùng output trực tiếp cuộn trong khi một callback được chỉ định đang thực thi. Nó lý tưởng để bao bọc các quá trình chạy lâu như cài đặt dependency hoặc script triển khai, cung cấp khả năng hiển thị thời gian thực về những gì đang xảy ra:

```php
use function Laravel\Prompts\task;

task(
    label: 'Installing dependencies',
    callback: function ($logger) {
        // Long-running process...
    }
);
```

Callback nhận một instance `Logger` mà bạn có thể sử dụng để hiển thị các dòng log, thông báo trạng thái, và văn bản được truyền trong vùng output của tác vụ.

> [!WARNING]
> Hàm `task` yêu cầu phần mở rộng PHP [PCNTL](https://www.php.net/manual/en/book.pcntl.php) để tạo hoạt ảnh cho spinner. Khi phần mở rộng này không khả dụng, một phiên bản tĩnh của tác vụ sẽ xuất hiện thay thế.

<a name="task-logging"></a>
#### Ghi log Các dòng

Phương thức `line` ghi một dòng log đơn vào vùng output cuộn của tác vụ:

```php
task(
    label: 'Installing dependencies',
    callback: function ($logger) {
        $logger->line('Resolving packages...');
        // ...
        $logger->line('Downloading laravel/framework');
        // ...
    }
);
```

<a name="task-status-messages"></a>
#### Thông báo Trạng thái

Bạn có thể sử dụng các phương thức `success`, `warning`, và `error` để hiển thị thông báo trạng thái. Các thông báo này xuất hiện dưới dạng các thông báo được làm nổi bật, ổn định ở trên vùng log cuộn:

```php
task(
    label: 'Deploying application',
    callback: function ($logger) {
        $logger->line('Pulling latest changes...');
        // ...
        $logger->success('Changes pulled!');

        $logger->line('Running migrations...');
        // ...
        $logger->warning('No new migrations to run.');

        $logger->line('Clearing cache...');
        // ...
        $logger->success('Cache cleared!');
    }
);
```

<a name="task-label"></a>
#### Cập nhật Nhãn

Phương thức `label` cho phép bạn cập nhật nhãn của tác vụ trong khi nó đang chạy:

```php
task(
    label: 'Starting deployment...',
    callback: function ($logger) {
        $logger->label('Pulling latest changes...');
        // ...
        $logger->label('Running migrations...');
        // ...
        $logger->label('Clearing cache...');
        // ...
    }
);
```

<a name="task-sub-label"></a>
#### Hiển thị Sub-Label

Phương thức `subLabel` hiển thị một dòng mờ bên dưới nhãn chính của tác vụ, hữu ích để giao tiếp trạng thái tạm thời như bước hiện đang trong tiến trình. Truyền một chuỗi rỗng để xóa sub-label:

```php
task(
    label: 'Deploying',
    callback: function ($logger) {
        $logger->subLabel('Building assets...');
        // ...
        $logger->subLabel('Running migrations...');
        // ...
        $logger->subLabel('');
    }
);
```

Bạn cũng có thể cung cấp một sub-label ban đầu thông qua đối số `subLabel`:

```php
task(
    label: 'Deploying',
    callback: function ($logger) {
        // ...
    },
    subLabel: 'Preparing...'
);
```

<a name="task-streaming"></a>
#### Truyền Văn bản

Đối với các quá trình tạo output dần dần, chẳng hạn như phản hồi được tạo bởi AI, phương thức `partial` cho phép bạn truyền văn bản từng từ hoặc từng đoạn. Khi luồng hoàn tất, gọi `commitPartial` để hoàn tất output:

```php
task(
    label: 'Generating response...',
    callback: function ($logger) {
        foreach ($words as $word) {
            $logger->partial($word . ' ');
        }

        $logger->commitPartial();
    }
);
```

<a name="task-limit"></a>
#### Tùy chỉnh Giới hạn Output

Theo mặc định, tác vụ hiển thị tối đa 10 dòng output cuộn. Bạn có thể tùy chỉnh điều này thông qua đối số `limit`:

```php
task(
    label: 'Installing dependencies',
    callback: function ($logger) {
        // ...
    },
    limit: 20
);
```

<a name="task-keep-summary"></a>
#### Giữ lại Tóm tắt

Theo mặc định, output của tác vụ bị xóa khi callback hoàn tất. Nếu bạn muốn giữ các thông báo trạng thái trên màn hình sau khi tác vụ đã hoàn thành, bạn có thể truyền đối số `keepSummary`:

```php
task(
    label: 'Deploying',
    callback: function ($logger) {
        $logger->success('Assets built');
        // ...
        $logger->success('Migrations complete');
    },
    keepSummary: true,
);
```

<a name="stream"></a>
## Stream

Hàm `stream` hiển thị văn bản được truyền vào terminal, lý tưởng để hiển thị nội dung được tạo bởi AI hoặc bất kỳ văn bản nào đến dần dần:

```php
use function Laravel\Prompts\stream;

$stream = stream();

foreach ($words as $word) {
    $stream->append($word . ' ');
    usleep(25_000); // Simulate delay between chunks...
}

$stream->close();
```

Phương thức `append` thêm văn bản vào luồng, hiển thị nó với hiệu ứng fade-in dần. Khi tất cả nội dung đã được truyền, gọi phương thức `close` để hoàn tất output và khôi phục con trỏ.

<a name="terminal-title"></a>
## Tiêu đề Terminal

Hàm `title` cập nhật tiêu đề của cửa sổ hoặc tab terminal của người dùng:

```php
use function Laravel\Prompts\title;

title('Installing Dependencies');
```

Để đặt lại tiêu đề terminal về mặc định, hãy truyền một chuỗi rỗng:

```php
title('');
```

<a name="clear"></a>
## Xóa Terminal

Hàm `clear` có thể được sử dụng để xóa terminal của người dùng:

```php
use function Laravel\Prompts\clear;

clear();
```

<a name="terminal-considerations"></a>
## Các Lưu ý về Terminal

<a name="terminal-width"></a>
#### Chiều rộng Terminal

Nếu độ dài của bất kỳ nhãn, tùy chọn, hoặc thông báo validation vượt quá số lượng "cột" trong terminal của người dùng, nó sẽ tự động bị cắt ngắn để vừa. Hãy xem xét giảm thiểu độ dài của các chuỗi này nếu người dùng của bạn có thể sử dụng các terminal hẹp hơn. Độ dài tối đa an toàn thường là 74 ký tự để hỗ trợ terminal 80 ký tự.

<a name="terminal-height"></a>
#### Chiều cao Terminal

Đối với bất kỳ prompt nào chấp nhận đối số `scroll`, giá trị được cấu hình sẽ tự động được giảm để vừa với chiều cao của terminal của người dùng, bao gồm không gian cho thông báo validation.

<a name="fallbacks"></a>
## Môi trường Không được Hỗ trợ và Fallbacks

Laravel Prompts hỗ trợ macOS, Linux và Windows với WSL. Do các hạn chế trong phiên bản Windows của PHP, hiện không thể sử dụng Laravel Prompts trên Windows ngoài WSL.

Vì lý do này, Laravel Prompts hỗ trợ chuyển sang một triển khai thay thế như [Symfony Console Question Helper](https://symfony.com/doc/current/components/console/helpers/questionhelper.html).

> [!NOTE]
> Khi sử dụng Laravel Prompts với framework Laravel, các fallback cho mỗi prompt đã được cấu hình cho bạn và sẽ tự động được kích hoạt trong các môi trường không được hỗ trợ.

<a name="fallback-conditions"></a>
#### Điều kiện Fallback

Nếu bạn không sử dụng Laravel hoặc cần tùy chỉnh khi hành vi fallback được sử dụng, bạn có thể truyền một boolean cho phương thức tĩnh `fallbackWhen` trên lớp `Prompt`:

```php
use Laravel\Prompts\Prompt;

Prompt::fallbackWhen(
    ! $input->isInteractive() || windows_os() || app()->runningUnitTests()
);
```

<a name="fallback-behavior"></a>
#### Hành vi Fallback

Nếu bạn không sử dụng Laravel hoặc cần tùy chỉnh hành vi fallback, bạn có thể truyền một closure cho phương thức tĩnh `fallbackUsing` trên mỗi lớp prompt:

```php
use Laravel\Prompts\TextPrompt;
use Symfony\Component\Console\Question\Question;
use Symfony\Component\Console\Style\SymfonyStyle;

TextPrompt::fallbackUsing(function (TextPrompt $prompt) use ($input, $output) {
    $question = (new Question($prompt->label, $prompt->default ?: null))
        ->setValidator(function ($answer) use ($prompt) {
            if ($prompt->required && $answer === null) {
                throw new \RuntimeException(
                    is_string($prompt->required) ? $prompt->required : 'Required.'
                );
            }

            if ($prompt->validate) {
                $error = ($prompt->validate)($answer ?? '');

                if ($error) {
                    throw new \RuntimeException($error);
                }
            }

            return $answer;
        });

    return (new SymfonyStyle($input, $output))
        ->askQuestion($question);
});
```

Fallbacks phải được cấu hình riêng cho mỗi lớp prompt. Closure sẽ nhận một instance của lớp prompt và phải trả về một loại phù hợp cho prompt.

<a name="testing"></a>
## Testing

Laravel cung cấp nhiều phương thức để kiểm tra rằng lệnh của bạn hiển thị các thông báo Prompt mong đợi:

```php tab=Pest
test('report generation', function () {
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
public function test_report_generation(): void
{
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
}
```

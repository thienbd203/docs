# Xác thực

- [Giới thiệu](#introduction)
- [Bắt đầu nhanh với Xác thực](#validation-quickstart)
    - [Định nghĩa các Route](#quick-defining-the-routes)
    - [Tạo Controller](#quick-creating-the-controller)
    - [Viết Logic Xác thực](#quick-writing-the-validation-logic)
    - [Hiển thị Lỗi Xác thực](#quick-displaying-the-validation-errors)
    - [Điền lại Form](#repopulating-forms)
    - [Lưu ý về các Trường Tùy chọn](#a-note-on-optional-fields)
    - [Định dạng Phản hồi Lỗi Xác thực](#validation-error-response-format)
- [Xác thực Form Request](#form-request-validation)
    - [Tạo Form Requests](#creating-form-requests)
    - [Phân quyền Form Requests](#authorizing-form-requests)
    - [Tùy chỉnh Thông báo Lỗi](#customizing-the-error-messages)
    - [Chuẩn bị Input cho Xác thực](#preparing-input-for-validation)
- [Tạo Validators Thủ công](#manually-creating-validators)
    - [Tự động Chuyển hướng](#automatic-redirection)
    - [Error Bags Được đặt tên](#named-error-bags)
    - [Tùy chỉnh Thông báo Lỗi](#manual-customizing-the-error-messages)
    - [Thực hiện Xác thực Bổ sung](#performing-additional-validation)
- [Làm việc với Input Đã xác thực](#working-with-validated-input)
- [Làm việc với Thông báo Lỗi](#working-with-error-messages)
    - [Chỉ định Thông báo Tùy chỉnh trong Tệp Ngôn ngữ](#specifying-custom-messages-in-language-files)
    - [Chỉ định Attributes trong Tệp Ngôn ngữ](#specifying-attribute-in-language-files)
    - [Chỉ định Giá trị trong Tệp Ngôn ngữ](#specifying-values-in-language-files)
- [Các Quy tắc Xác thực Có sẵn](#available-validation-rules)
- [Thêm Quy tắc Có điều kiện](#conditionally-adding-rules)
- [Xác thực Mảng](#validating-arrays)
    - [Xác thực Input Mảng Lồng nhau](#validating-nested-array-input)
    - [Chỉ số và Vị trí Thông báo Lỗi](#error-message-indexes-and-positions)
- [Xác thực Tệp](#validating-files)
- [Xác thực Mật khẩu](#validating-passwords)
- [Quy tắc Xác thực Tùy chỉnh](#custom-validation-rules)
    - [Sử dụng Đối tượng Quy tắc](#using-rule-objects)
    - [Sử dụng Closures](#using-closures)
    - [Quy tắc Ngầm định](#implicit-rules)

<a name="introduction"></a>
## Giới thiệu

Laravel cung cấp một số cách khác nhau để xác thực dữ liệu đầu vào của ứng dụng. Cách phổ biến nhất là sử dụng phương thức `validate` có sẵn trên tất cả các yêu cầu HTTP đến. Tuy nhiên, chúng ta cũng sẽ thảo luận về các phương pháp xác thực khác.

Laravel bao gồm nhiều quy tắc xác thực tiện lợi mà bạn có thể áp dụng cho dữ liệu, thậm chí cung cấp khả năng xác thực xem các giá trị có duy nhất trong một bảng cơ sở dữ liệu cụ thể hay không. Chúng ta sẽ đề cập chi tiết từng quy tắc xác thực này để bạn làm quen với tất cả các tính năng xác thực của Laravel.

<a name="validation-quickstart"></a>
## Bắt đầu nhanh với Xác thực

Để tìm hiểu về các tính năng xác thực mạnh mẽ của Laravel, hãy xem một ví dụ hoàn chỉnh về xác thực một form và hiển thị các thông báo lỗi trở lại cho người dùng. Bằng cách đọc tổng quan cấp cao này, bạn sẽ có thể hiểu tốt về cách xác thực dữ liệu yêu cầu đến bằng Laravel:

<a name="quick-defining-the-routes"></a>
### Định nghĩa các Route

Đầu tiên, giả sử chúng ta có các route sau được định nghĩa trong tệp `routes/web.php`:

```php
use App\Http\Controllers\PostController;

Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

Route `GET` sẽ hiển thị một form cho người dùng để tạo bài đăng blog mới, trong khi route `POST` sẽ lưu bài đăng blog mới vào cơ sở dữ liệu.

<a name="quick-creating-the-controller"></a>
### Tạo Controller

Tiếp theo, hãy xem một controller đơn giản xử lý các yêu cầu đến cho các route này. Chúng ta sẽ để phương thức `store` trống trong lúc này:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

class PostController extends Controller
{
    /**
     * Hiển thị form để tạo bài đăng blog mới.
     */
    public function create(): View
    {
        return view('post.create');
    }

    /**
     * Lưu bài đăng blog mới.
     */
    public function store(Request $request): RedirectResponse
    {
        // Xác thực và lưu bài đăng blog...

        $post = /** ... */

        return to_route('post.show', ['post' => $post->id]);
    }
}
```

<a name="quick-writing-the-validation-logic"></a>
### Viết Logic Xác thực

Bây giờ chúng ta đã sẵn sàng để điền vào phương thức `store` với logic để xác thực bài đăng blog mới. Để làm điều này, chúng ta sẽ sử dụng phương thức `validate` được cung cấp bởi đối tượng `Illuminate\Http\Request`. Nếu các quy tắc xác thực vượt qua, mã của bạn sẽ tiếp tục thực thi bình thường; tuy nhiên, nếu xác thực thất bại, một ngoại lệ `Illuminate\Validation\ValidationException` sẽ được ném ra và phản hồi lỗi thích hợp sẽ tự động được gửi trở lại cho người dùng.

Nếu xác thực thất bại trong một yêu cầu HTTP truyền thống, một phản hồi chuyển hướng đến URL trước đó sẽ được tạo ra. Nếu yêu cầu đến là một yêu cầu XHR, một [phản hồi JSON chứa các thông báo lỗi xác thực](#validation-error-response-format) sẽ được trả về.

Để hiểu rõ hơn về phương thức `validate`, hãy quay lại phương thức `store`:

```php
/**
 * Lưu bài đăng blog mới.
 */
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

    // Bài đăng blog hợp lệ...

    return redirect('/posts');
}
```

Như bạn có thể thấy, các quy tắc xác thực được chuyển vào phương thức `validate`. Đừng lo - tất cả các quy tắc xác thực có sẵn đều được [tài liệu hóa](#available-validation-rules). Một lần nữa, nếu xác thực thất bại, phản hồi thích hợp sẽ tự động được tạo ra. Nếu xác thực vượt qua, controller của chúng ta sẽ tiếp tục thực thi bình thường.

Ngoài ra, bạn có thể sử dụng phương thức `validateWithBag` để xác thực một yêu cầu và lưu trữ bất kỳ thông báo lỗi nào trong một [error bag được đặt tên](#named-error-bags):

```php
$validated = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

<a name="stopping-on-first-validation-failure"></a>
#### Dừng khi Lỗi Xác thực Đầu tiên

Đôi khi bạn có thể muốn dừng chạy các quy tắc xác thực trên một thuộc tính sau khi lỗi xác thực đầu tiên xảy ra. Để làm điều này, gán quy tắc `bail` cho thuộc tính:

```php
$request->validate([
    'title' => ['bail', 'required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

Trong ví dụ này, nếu quy tắc `unique` trên thuộc tính `title` thất bại, quy tắc `max` sẽ không được kiểm tra. Các quy tắc sẽ được xác thực theo thứ tự chúng được gán.

<a name="a-note-on-nested-attributes"></a>
#### Lưu ý về Các thuộc tính Lồng nhau

Nếu yêu cầu HTTP đến chứa dữ liệu trường "lồng nhau", bạn có thể chỉ định các trường này trong quy tắc xác thực của mình bằng cú pháp "dấu chấm":

```php
$request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'author.name' => ['required'],
    'author.description' => ['required'],
]);
```

Mặt khác, nếu tên trường của bạn chứa một dấu chấm theo nghĩa đen, bạn có thể ngăn chặn rõ ràng việc này được hiểu là cú pháp "dấu chấm" bằng cách thoát dấu chấm bằng dấu gạch chéo ngược:

```php
$request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'v1\.0' => ['required'],
]);
```

<a name="quick-displaying-the-validation-errors"></a>
### Hiển thị Lỗi Xác thực

Vậy, nếu các trường yêu cầu đến không vượt qua các quy tắc xác thực đã cho thì sao? Như đã đề cập trước đó, Laravel sẽ tự động chuyển hướng người dùng trở lại vị trí trước đó của họ. Ngoài ra, tất cả các lỗi xác thực và [input yêu cầu](/docs/{{version}}/requests#retrieving-old-input) sẽ tự động được [flash vào session](/docs/{{version}}/session#flash-data).

Một biến `$errors` được chia sẻ với tất cả các view của ứng dụng bạn bởi middleware `Illuminate\View\Middleware\ShareErrorsFromSession`, được cung cấp bởi nhóm middleware `web`. Khi middleware này được áp dụng, biến `$errors` sẽ luôn có sẵn trong các view của bạn, cho phép bạn giả định một cách thuận tiện rằng biến `$errors` luôn được định nghĩa và có thể được sử dụng an toàn. Biến `$errors` sẽ là một thể hiện của `Illuminate\Support\MessageBag`. Để biết thêm thông tin về cách làm việc với đối tượng này, [hãy xem tài liệu của nó](#working-with-error-messages).

Vì vậy, trong ví dụ của chúng ta, người dùng sẽ được chuyển hướng đến phương thức `create` của controller khi xác thực thất bại, cho phép chúng ta hiển thị các thông báo lỗi trong view:

```blade
<!-- /resources/views/post/create.blade.php -->

<h1>Tạo Bài đăng</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Tạo Form Bài đăng -->
```

<a name="quick-customizing-the-error-messages"></a>
#### Tùy chỉnh Thông báo Lỗi

Mỗi quy tắc xác thực tích hợp của Laravel đều có một thông báo lỗi nằm trong tệp `lang/en/validation.php` của ứng dụng bạn. Nếu ứng dụng của bạn không có thư mục `lang`, bạn có thể hướng dẫn Laravel tạo nó bằng lệnh Artisan `lang:publish`.

Trong tệp `lang/en/validation.php`, bạn sẽ tìm thấy một mục dịch cho mỗi quy tắc xác thực. Bạn có thể thay đổi hoặc sửa đổi các thông báo này dựa trên nhu cầu của ứng dụng bạn.

Ngoài ra, bạn có thể sao chép tệp này sang một thư mục ngôn ngữ khác để dịch các thông báo cho ngôn ngữ của ứng dụng bạn. Để tìm hiểu thêm về bản địa hóa Laravel, hãy xem tài liệu [bản địa hóa](/docs/{{version}}/localization) hoàn chỉnh.

> [!WARNING]
> Theo mặc định, khung ứng dụng Laravel không bao gồm thư mục `lang`. Nếu bạn muốn tùy chỉnh các tệp ngôn ngữ của Laravel, bạn có thể xuất bản chúng thông qua lệnh Artisan `lang:publish`.

<a name="quick-xhr-requests-and-validation"></a>
#### Yêu cầu XHR và Xác thực

Trong ví dụ này, chúng ta đã sử dụng một form truyền thống để gửi dữ liệu đến ứng dụng. Tuy nhiên, nhiều ứng dụng nhận các yêu cầu XHR từ một frontend được hỗ trợ bởi JavaScript. Khi sử dụng phương thức `validate` trong một yêu cầu XHR, Laravel sẽ không tạo phản hồi chuyển hướng. Thay vào đó, Laravel tạo một [phản hồi JSON chứa tất cả các lỗi xác thực](#validation-error-response-format). Phản hồi JSON này sẽ được gửi với mã trạng thái HTTP 422.

<a name="the-at-error-directive"></a>
#### Directive `@error`

Bạn có thể sử dụng directive [Blade](/docs/{{version}}/blade) `@error` để nhanh chóng xác định xem có thông báo lỗi xác thực cho một thuộc tính cụ thể hay không. Trong một directive `@error`, bạn có thể echo biến `$message` để hiển thị thông báo lỗi:

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Tiêu đề Bài đăng</label>

<input
    id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Nếu bạn đang sử dụng [error bags được đặt tên](#named-error-bags), bạn có thể chuyển tên của error bag làm đối số thứ hai cho directive `@error`:

```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

<a name="repopulating-forms"></a>
### Điền lại Form

Khi Laravel tạo phản hồi chuyển hướng do lỗi xác thực, framework sẽ tự động [flash tất cả input của yêu cầu vào session](/docs/{{version}}/session#flash-data). Điều này được thực hiện để bạn có thể truy cập input một cách thuận tiện trong yêu cầu tiếp theo và điền lại form mà người dùng đã cố gắng gửi.

Để truy xuất input đã flash từ yêu cầu trước đó, hãy gọi phương thức `old` trên một thể hiện của `Illuminate\Http\Request`. Phương thức `old` sẽ kéo dữ liệu input đã flash trước đó từ [session](/docs/{{version}}/session):

```php
$title = $request->old('title');
```

Laravel cũng cung cấp một helper `old` toàn cục. Nếu bạn đang hiển thị input cũ trong một [template Blade](/docs/{{version}}/blade), sẽ thuận tiện hơn khi sử dụng helper `old` để điền lại form. Nếu không có input cũ nào tồn tại cho trường đã cho, `null` sẽ được trả về:

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

<a name="a-note-on-optional-fields"></a>
### Lưu ý về Các trường Tùy chọn

Theo mặc định, Laravel bao gồm middleware `TrimStrings` và `ConvertEmptyStringsToNull` trong stack middleware toàn cục của ứng dụng bạn. Vì lý do này, bạn thường sẽ cần đánh dấu các trường yêu cầu "tùy chọn" của mình là `nullable` nếu bạn không muốn validator coi các giá trị `null` là không hợp lệ. Ví dụ:

```php
$request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
    'publish_at' => ['nullable', 'date'],
]);
```

Trong ví dụ này, chúng ta đang chỉ định rằng trường `publish_at` có thể là `null` hoặc một biểu diễn ngày hợp lệ. Nếu bộ sửa đổi `nullable` không được thêm vào định nghĩa quy tắc, validator sẽ coi `null` là một ngày không hợp lệ.

<a name="validation-error-response-format"></a>
### Định dạng Phản hồi Lỗi Xác thực

Khi ứng dụng của bạn ném một ngoại lệ `Illuminate\Validation\ValidationException` và yêu cầu HTTP đến mong đợi một phản hồi JSON, Laravel sẽ tự động định dạng các thông báo lỗi cho bạn và trả về một phản hồi HTTP `422 Unprocessable Entity`.

Dưới đây, bạn có thể xem xét một ví dụ về định dạng phản hồi JSON cho các lỗi xác thực. Lưu ý rằng các khóa lỗi lồng nhau được làm phẳng thành định dạng ký hiệu "dấu chấm":

```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

<a name="form-request-validation"></a>
## Xác thực Form Request

<a name="creating-form-requests"></a>
### Tạo Form Requests

Đối với các tình huống xác thực phức tạp hơn, bạn có thể muốn tạo một "form request". Form requests là các lớp yêu cầu tùy chỉnh đóng gói logic xác thực và phân quyền của riêng chúng. Để tạo một lớp form request, bạn có thể sử dụng lệnh CLI Artisan `make:request`:

```shell
php artisan make:request StorePostRequest
```

Lớp form request được tạo sẽ được đặt trong thư mục `app/Http/Requests`. Nếu thư mục này không tồn tại, nó sẽ được tạo khi bạn chạy lệnh `make:request`. Mỗi form request được tạo bởi Laravel có hai phương thức: `authorize` và `rules`.

Như bạn có thể đã đoán, phương thức `authorize` chịu trách nhiệm xác định xem người dùng đã xác thực hiện tại có thể thực hiện hành động được đại diện bởi yêu cầu hay không, trong khi phương thức `rules` trả về các quy tắc xác thực nên áp dụng cho dữ liệu của yêu cầu:

```php
/**
 * Lấy các quy tắc xác thực áp dụng cho yêu cầu.
 *
 * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
 */
public function rules(): array
{
    return [
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ];
}
```

> [!NOTE]
> Bạn có thể type-hint bất kỳ dependency nào bạn cần trong chữ ký của phương thức `rules`. Chúng sẽ tự động được giải quyết thông qua [service container](/docs/{{version}}/container) của Laravel.

Vậy, các quy tắc xác thực được đánh giá như thế nào? Tất cả những gì bạn cần làm là type-hint yêu cầu trên phương thức controller của bạn. Yêu cầu form đến được xác thực trước khi phương thức controller được gọi, nghĩa là bạn không cần làm rối controller của mình với bất kỳ logic xác thực nào:

```php
/**
 * Lưu bài đăng blog mới.
 */
public function store(StorePostRequest $request): RedirectResponse
{
    // Yêu cầu đến hợp lệ...

    // Truy xuất dữ liệu input đã xác thực...
    $validated = $request->validated();

    // Truy xuất một phần dữ liệu input đã xác thực...
    $validated = $request->safe()->only(['name', 'email']);
    $validated = $request->safe()->except(['name', 'email']);

    // Lưu bài đăng blog...

    return redirect('/posts');
}
```

Nếu xác thực thất bại, một phản hồi chuyển hướng sẽ được tạo để gửi người dùng trở lại vị trí trước đó của họ. Các lỗi cũng sẽ được flash vào session để chúng có sẵn để hiển thị. Nếu yêu cầu là một yêu cầu XHR, một phản hồi HTTP với mã trạng thái 422 sẽ được trả về cho người dùng bao gồm một [biểu diễn JSON của các lỗi xác thực](#validation-error-response-format).

> [!NOTE]
> Cần thêm xác thực form request thời gian thực vào frontend Laravel được hỗ trợ bởi Inertia? Hãy xem [Laravel Precognition](/docs/{{version}}/precognition).

<a name="performing-additional-validation-on-form-requests"></a>
#### Thực hiện Xác thực Bổ sung

Đôi khi bạn cần thực hiện xác thực bổ sung sau khi xác thực ban đầu hoàn tất. Bạn có thể thực hiện điều này bằng phương thức `after` của form request.

Phương thức `after` nên trả về một mảng các callable hoặc closure sẽ được gọi sau khi xác thực hoàn tất. Các callable đã cho sẽ nhận một thể hiện `Illuminate\Validation\Validator`, cho phép bạn nâng cao các thông báo lỗi bổ sung nếu cần:

```php
use Illuminate\Validation\Validator;

/**
 * Lấy các callable xác thực "after" cho yêu cầu.
 */
public function after(): array
{
    return [
        function (Validator $validator) {
            if ($this->somethingElseIsInvalid()) {
                $validator->errors()->add(
                    'field',
                    'Something is wrong with this field!'
                );
            }
        }
    ];
}
```

Như đã lưu ý, mảng được trả về bởi phương thức `after` cũng có thể chứa các lớp có thể gọi. Phương thức `__invoke` của các lớp này sẽ nhận một thể hiện `Illuminate\Validation\Validator`:

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;
use Illuminate\Validation\Validator;

/**
 * Lấy các callable xác thực "after" cho yêu cầu.
 */
public function after(): array
{
    return [
        new ValidateUserStatus,
        new ValidateShippingTime,
        function (Validator $validator) {
            //
        }
    ];
}
```

<a name="request-stopping-on-first-validation-rule-failure"></a>
#### Dừng khi Lỗi Xác thực Đầu tiên

Bằng cách thêm thuộc tính `StopOnFirstFailure` vào lớp yêu cầu của bạn, bạn có thể thông báo cho validator rằng nó nên dừng xác thực tất cả các thuộc tính sau khi một lỗi xác thực đơn lẻ đã xảy ra:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\Attributes\StopOnFirstFailure;
use Illuminate\Foundation\Http\FormRequest;

#[StopOnFirstFailure]
class StorePostRequest extends FormRequest
{
    // ...
}
```

<a name="request-failing-on-unknown-fields"></a>
#### Thất bại khi Các trường Không xác định

Bằng cách thêm thuộc tính `FailOnUnknownFields` vào lớp yêu cầu của bạn, bạn có thể hướng dẫn Laravel từ chối bất kỳ trường đến nào không được định nghĩa bởi các quy tắc xác thực của yêu cầu:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\Attributes\FailOnUnknownFields;
use Illuminate\Foundation\Http\FormRequest;

#[FailOnUnknownFields]
class StorePostRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'title' => ['required', 'string'],
            'body' => ['required', 'string'],
        ];
    }
}
```

Bạn cũng có thể bật hành vi này toàn cục cho tất cả các form request từ `AppServiceProvider` của bạn:

```php
use Illuminate\Foundation\Http\FormRequest;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    FormRequest::failOnUnknownFields();
}
```

Nếu cần, bạn có thể tắt hành vi này cho một yêu cầu cụ thể bằng cách chuyển `false` cho thuộc tính:

```php
#[FailOnUnknownFields(false)]
class PublicWebhookRequest extends FormRequest
{
    // ...
}
```

Từ chối các trường không xác định có thể cung cấp bảo vệ bổ sung chống lại các vấn đề kiểu mass-assignment bằng cách ngăn chặn các khóa input không mong muốn chảy sâu hơn vào ứng dụng của bạn. Tuy nhiên, bạn vẫn nên cấu hình các thuộc tính `$fillable` / `$guarded` của mô hình và chỉ lưu trữ input đã xác thực đáng tin cậy.

<a name="customizing-the-redirect-location"></a>
#### Tùy chỉnh Vị trí Chuyển hướng

Khi xác thực form request thất bại, một phản hồi chuyển hướng sẽ được tạo để gửi người dùng trở lại vị trí trước đó của họ. Tuy nhiên, bạn có thể tùy chỉnh hành vi này. Để làm điều này, bạn có thể sử dụng thuộc tính `RedirectTo` trên form request của bạn:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\Attributes\RedirectTo;
use Illuminate\Foundation\Http\FormRequest;

#[RedirectTo('/dashboard')]
class StorePostRequest extends FormRequest
{
    // ...
}
```

Hoặc, nếu bạn muốn chuyển hướng người dùng đến một route được đặt tên, bạn có thể sử dụng thuộc tính `RedirectToRoute` thay thế:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\Attributes\RedirectToRoute;
use Illuminate\Foundation\Http\FormRequest;

#[RedirectToRoute('dashboard')]
class StorePostRequest extends FormRequest
{
    // ...
}
```

<a name="customizing-the-error-bag"></a>
#### Tùy chỉnh Error Bag

Khi xác thực form request thất bại, các lỗi được flash vào error bag `default`. Nếu bạn cần lưu trữ các lỗi trong một [error bag được đặt tên](#named-error-bags) khác, bạn có thể sử dụng thuộc tính `ErrorBag` trên form request của bạn:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\Attributes\ErrorBag;
use Illuminate\Foundation\Http\FormRequest;

#[ErrorBag('login')]
class LoginRequest extends FormRequest
{
    // ...
}
```

<a name="authorizing-form-requests"></a>
### Phân quyền Form Requests

Lớp form request cũng chứa một phương thức `authorize`. Trong phương thức này, bạn có thể xác định xem người dùng đã xác thực thực sự có quyền cập nhật một tài nguyên cụ thể hay không. Ví dụ, bạn có thể xác định xem người dùng thực sự sở hữu một bình luận blog mà họ đang cố gắng cập nhật. Rất có thể, bạn sẽ tương tác với các [cổng và chính sách phân quyền](/docs/{{version}}/authorization) của bạn trong phương thức này:

```php
use App\Models\Comment;

/**
 * Xác định xem người dùng có được phép thực hiện yêu cầu này hay không.
 */
public function authorize(): bool
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

Vì tất cả các form request đều mở rộng lớp yêu cầu cơ bản của Laravel, chúng ta có thể sử dụng phương thức `user` để truy cập người dùng đã xác thực hiện tại. Ngoài ra, lưu ý cuộc gọi đến phương thức `route` trong ví dụ trên. Phương thức này cấp cho bạn quyền truy cập các tham số URI được định nghĩa trên route đang được gọi, chẳng hạn như tham số `{comment}` trong ví dụ dưới đây:

```php
Route::post('/comment/{comment}');
```

Do đó, nếu ứng dụng của bạn đang tận dụng [ràng buộc mô hình route](/docs/{{version}}/routing#route-model-binding), mã của bạn có thể được làm ngắn gọn hơn bằng cách truy cập mô hình đã giải quyết như một thuộc tính của yêu cầu:

```php
return $this->user()->can('update', $this->comment);
```

Nếu phương thức `authorize` trả về `false`, một phản hồi HTTP với mã trạng thái 403 sẽ tự động được trả về và phương thức controller của bạn sẽ không thực thi.

Nếu bạn có kế hoạch xử lý logic phân quyền cho yêu cầu trong một phần khác của ứng dụng, bạn có thể xóa hoàn toàn phương thức `authorize`, hoặc đơn giản là trả về `true`:

```php
/**
 * Xác định xem người dùng có được phép thực hiện yêu cầu này hay không.
 */
public function authorize(): bool
{
    return true;
}
```

> [!NOTE]
> Bạn có thể type-hint bất kỳ dependency nào bạn cần trong chữ ký của phương thức `authorize`. Chúng sẽ tự động được giải quyết thông qua [service container](/docs/{{version}}/container) của Laravel.

<a name="customizing-the-error-messages"></a>
### Tùy chỉnh Thông báo Lỗi

Bạn có thể tùy chỉnh các thông báo lỗi được sử dụng bởi form request bằng cách ghi đè phương thức `messages`. Phương thức này nên trả về một mảng các cặp thuộc tính / quy tắc và các thông báo lỗi tương ứng của chúng:

```php
/**
 * Lấy các thông báo lỗi cho các quy tắc xác thực đã định nghĩa.
 *
 * @return array<string, string>
 */
public function messages(): array
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```

<a name="customizing-the-validation-attributes"></a>
#### Tùy chỉnh Các thuộc tính Xác thực

Nhiều thông báo lỗi quy tắc xác thực tích hợp của Laravel chứa một placeholder `:attribute`. Nếu bạn muốn placeholder `:attribute` của thông báo xác thực của mình được thay thế bằng một tên thuộc tính tùy chỉnh, bạn có thể chỉ định các tên tùy chỉnh bằng cách ghi đè phương thức `attributes`. Phương thức này nên trả về một mảng các cặp thuộc tính / tên:

```php
/**
 * Lấy các thuộc tính tùy chỉnh cho lỗi validator.
 *
 * @return array<string, string>
 */
public function attributes(): array
{
    return [
        'email' => 'email address',
    ];
}
```

<a name="preparing-input-for-validation"></a>
### Chuẩn bị Input cho Xác thực

Nếu bạn cần chuẩn bị hoặc làm sạch bất kỳ dữ liệu nào từ yêu cầu trước khi bạn áp dụng các quy tắc xác thực, bạn có thể sử dụng phương thức `prepareForValidation`:

```php
use Illuminate\Support\Str;

/**
 * Chuẩn bị dữ liệu để xác thực.
 */
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

Tương tự, nếu bạn cần chuẩn hóa bất kỳ dữ liệu yêu cầu nào sau khi xác thực hoàn tất, bạn có thể sử dụng phương thức `passedValidation`:

```php
/**
 * Xử lý một nỗ lực xác thực đã vượt qua.
 */
protected function passedValidation(): void
{
    $this->replace(['name' => 'Taylor']);
}
```

<a name="manually-creating-validators"></a>
## Tạo Validators Thủ công

Nếu bạn không muốn sử dụng phương thức `validate` trên yêu cầu, bạn có thể tạo một thể hiện validator thủ công bằng cách sử dụng [facade](/docs/{{version}}/facades) `Validator`. Phương thức `make` trên facade tạo ra một thể hiện validator mới:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    /**
     * Lưu bài đăng blog mới.
     */
    public function store(Request $request): RedirectResponse
    {
        $validator = Validator::make($request->all(), [
            'title' => ['required', 'unique:posts', 'max:255'],
            'body' => ['required'],
        ]);

        if ($validator->fails()) {
            return redirect('/post/create')
                ->withErrors($validator)
                ->withInput();
        }

        // Truy xuất input đã xác thực...
        $validated = $validator->validated();

        // Truy xuất một phần input đã xác thực...
        $validated = $validator->safe()->only(['name', 'email']);
        $validated = $validator->safe()->except(['name', 'email']);

        // Lưu bài đăng blog...

        return redirect('/posts');
    }
}
```

Đối số đầu tiên được chuyển đến phương thức `make` là dữ liệu đang được xác thực. Đối số thứ hai là một mảng các quy tắc xác thực nên được áp dụng cho dữ liệu.

Sau khi xác định xem xác thực yêu cầu có thất bại hay không, bạn có thể sử dụng phương thức `withErrors` để flash các thông báo lỗi vào session. Khi sử dụng phương thức này, biến `$errors` sẽ tự động được chia sẻ với các view của bạn sau khi chuyển hướng, cho phép bạn dễ dàng hiển thị chúng trở lại cho người dùng. Phương thức `withErrors` chấp nhận một validator, một `MessageBag`, hoặc một mảng PHP.

#### Dừng khi Lỗi Xác thực Đầu tiên

Phương thức `stopOnFirstFailure` sẽ thông báo cho validator rằng nó nên dừng xác thực tất cả các thuộc tính sau khi một lỗi xác thực đơn lẻ đã xảy ra:

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

<a name="automatic-redirection"></a>
### Tự động Chuyển hướng

Nếu bạn muốn tạo một thể hiện validator thủ công nhưng vẫn tận dụng lợi ích của chuyển hướng tự động được cung cấp bởi phương thức `validate` của yêu cầu HTTP, bạn có thể gọi phương thức `validate` trên một thể hiện validator hiện có. Nếu xác thực thất bại, người dùng sẽ tự động được chuyển hướng hoặc, trong trường hợp yêu cầu XHR, một [phản hồi JSON sẽ được trả về](#validation-error-response-format):

```php
Validator::make($request->all(), [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
])->validate();
```

Bạn có thể sử dụng phương thức `validateWithBag` để lưu trữ các thông báo lỗi trong một [error bag được đặt tên](#named-error-bags) nếu xác thực thất bại:

```php
Validator::make($request->all(), [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
])->validateWithBag('post');
```

<a name="named-error-bags"></a>
### Error Bags Được đặt tên

Nếu bạn có nhiều form trên một trang, bạn có thể muốn đặt tên cho `MessageBag` chứa các lỗi xác thực, cho phép bạn truy xuất các thông báo lỗi cho một form cụ thể. Để đạt được điều này, chuyển một tên làm đối số thứ hai cho `withErrors`:

```php
return redirect('/register')->withErrors($validator, 'login');
```

Sau đó, bạn có thể truy cập thể hiện `MessageBag` được đặt tên từ biến `$errors`:

```blade
{{ $errors->login->first('email') }}
```

<a name="manual-customizing-the-error-messages"></a>
### Tùy chỉnh Thông báo Lỗi

Nếu cần, bạn có thể cung cấp các thông báo lỗi tùy chỉnh mà một thể hiện validator nên sử dụng thay vì các thông báo lỗi mặc định được cung cấp bởi Laravel. Có một số cách để chỉ định các thông báo tùy chỉnh. Đầu tiên, bạn có thể chuyển các thông báo tùy chỉnh làm đối số thứ ba cho phương thức `Validator::make`:

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'The :attribute field is required.',
]);
```

Trong ví dụ này, placeholder `:attribute` sẽ được thay thế bằng tên thực tế của trường đang được xác thực. Bạn cũng có thể sử dụng các placeholder khác trong các thông báo xác thực. Ví dụ:

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

<a name="specifying-a-custom-message-for-a-given-attribute"></a>
#### Chỉ định Thông báo Tùy chỉnh cho Một thuộc tính Cụ thể

Đôi khi bạn có thể muốn chỉ định một thông báo lỗi tùy chỉnh chỉ cho một thuộc tính cụ thể. Bạn có thể làm điều này bằng cách sử dụng ký hiệu "dấu chấm". Chỉ định tên thuộc tính trước, sau đó là quy tắc:

```php
$messages = [
    'email.required' => 'We need to know your email address!',
];
```

<a name="specifying-custom-attribute-values"></a>
#### Chỉ định Giá trị Thuộc tính Tùy chỉnh

Nhiều thông báo lỗi tích hợp của Laravel bao gồm một placeholder `:attribute` được thay thế bằng tên của trường hoặc thuộc tính đang được xác thực. Để tùy chỉnh các giá trị được sử dụng để thay thế các placeholder này cho các trường cụ thể, bạn có thể chuyển một mảng các thuộc tính tùy chỉnh làm đối số thứ tư cho phương thức `Validator::make`:
```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'email address',
]);
```

<a name="performing-additional-validation"></a>
### Thực hiện Xác thực Bổ sung

Đôi khi bạn cần thực hiện xác thực bổ sung sau khi xác thực ban đầu hoàn tất. Bạn có thể thực hiện việc này bằng cách sử dụng phương thức `after` của validator. Phương thức `after` chấp nhận một closure hoặc một mảng các callable sẽ được gọi sau khi xác thực hoàn tất. Các callable được cung cấp sẽ nhận một instance của `Illuminate\Validation\Validator`, cho phép bạn thêm thông báo lỗi bổ sung nếu cần:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make(/* ... */);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add(
            'field', 'Something is wrong with this field!'
        );
    }
});

if ($validator->fails()) {
    // ...
}
```

Như đã lưu ý, phương thức `after` cũng chấp nhận một mảng các callable, điều này đặc biệt thuận tiện nếu logic "xác thực sau" của bạn được đóng gói trong các lớp có thể gọi (invokable classes), các lớp này sẽ nhận một instance của `Illuminate\Validation\Validator` thông qua phương thức `__invoke` của chúng:

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;

$validator->after([
    new ValidateUserStatus,
    new ValidateShippingTime,
    function ($validator) {
        // ...
    },
]);
```

<a name="working-with-validated-input"></a>
## Làm việc với Dữ liệu Đã Xác thực

Sau khi xác thực dữ liệu request đến bằng form request hoặc một instance validator được tạo thủ công, bạn có thể muốn lấy dữ liệu request đến thực sự đã trải qua xác thực. Điều này có thể được thực hiện theo một số cách. Đầu tiên, bạn có thể gọi phương thức `validated` trên form request hoặc instance validator. Phương thức này trả về một mảng dữ liệu đã được xác thực:

```php
$validated = $request->validated();

$validated = $validator->validated();
```

Ngoài ra, bạn có thể gọi phương thức `safe` trên form request hoặc instance validator. Phương thức này trả về một instance của `Illuminate\Support\ValidatedInput`. Đối tượng này cung cấp các phương thức `only`, `except`, và `all` để lấy một tập con của dữ liệu đã xác thực hoặc toàn bộ mảng dữ liệu đã xác thực:

```php
$validated = $request->safe()->only(['name', 'email']);

$validated = $request->safe()->except(['name', 'email']);

$validated = $request->safe()->all();
```

Ngoài ra, instance của `Illuminate\Support\ValidatedInput` có thể được lặp qua và truy cập như một mảng:

```php
// Dữ liệu đã xác thực có thể được lặp qua...
foreach ($request->safe() as $key => $value) {
    // ...
}

// Dữ liệu đã xác thực có thể được truy cập như một mảng...
$validated = $request->safe();

$email = $validated['email'];
```

Nếu bạn muốn thêm các trường bổ sung vào dữ liệu đã xác thực, bạn có thể gọi phương thức `merge`:

```php
$validated = $request->safe()->merge(['name' => 'Taylor Otwell']);
```

Nếu bạn muốn lấy dữ liệu đã xác thực dưới dạng một instance [collection](/docs/{{version}}/collections), bạn có thể gọi phương thức `collect`:

```php
$collection = $request->safe()->collect();
```

<a name="working-with-error-messages"></a>
## Làm việc với Thông báo Lỗi

Sau khi gọi phương thức `errors` trên một instance của `Validator`, bạn sẽ nhận được một instance của `Illuminate\Support\MessageBag`, có nhiều phương thức thuận tiện để làm việc với các thông báo lỗi. Biến `$errors` được tự động cung cấp cho tất cả các view cũng là một instance của lớp `MessageBag`.

<a name="retrieving-the-first-error-message-for-a-field"></a>
#### Lấy Thông báo Lỗi Đầu tiên cho một Trường

Để lấy thông báo lỗi đầu tiên cho một trường nhất định, sử dụng phương thức `first`:

```php
$errors = $validator->errors();

echo $errors->first('email');
```

<a name="retrieving-all-error-messages-for-a-field"></a>
#### Lấy Tất cả Thông báo Lỗi cho một Trường

Nếu bạn cần lấy một mảng tất cả các thông báo cho một trường nhất định, sử dụng phương thức `get`:

```php
foreach ($errors->get('email') as $message) {
    // ...
}
```

Nếu bạn đang xác thực một trường form mảng, bạn có thể lấy tất cả các thông báo cho từng phần tử của mảng bằng cách sử dụng ký tự `*`:

```php
foreach ($errors->get('attachments.*') as $message) {
    // ...
}
```

<a name="retrieving-all-error-messages-for-all-fields"></a>
#### Lấy Tất cả Thông báo Lỗi cho Tất cả các Trường

Để lấy một mảng tất cả các thông báo cho tất cả các trường, sử dụng phương thức `all`:

```php
foreach ($errors->all() as $message) {
    // ...
}
```

<a name="determining-if-messages-exist-for-a-field"></a>
#### Xác định xem Thông báo Có Tồn tại cho một Trường hay Không

Phương thức `has` có thể được sử dụng để xác định xem có bất kỳ thông báo lỗi nào tồn tại cho một trường nhất định hay không:

```php
if ($errors->has('email')) {
    // ...
}
```

<a name="specifying-custom-messages-in-language-files"></a>
### Chỉ định Thông báo Tùy chỉnh trong Tệp Ngôn ngữ

Mỗi quy tắc xác thực tích hợp sẵn của Laravel đều có một thông báo lỗi nằm trong tệp `lang/en/validation.php` của ứng dụng. Nếu ứng dụng của bạn không có thư mục `lang`, bạn có thể hướng dẫn Laravel tạo nó bằng lệnh Artisan `lang:publish`.

Trong tệp `lang/en/validation.php`, bạn sẽ tìm thấy một mục dịch cho mỗi quy tắc xác thực. Bạn có thể thay đổi hoặc sửa đổi các thông báo này dựa trên nhu cầu của ứng dụng.

Ngoài ra, bạn có thể sao chép tệp này sang một thư mục ngôn ngữ khác để dịch các thông báo cho ngôn ngữ của ứng dụng. Để tìm hiểu thêm về bản địa hóa Laravel, hãy xem tài liệu [localization](/docs/{{version}}/localization) đầy đủ.

> [!WARNING]
> Theo mặc định, khung ứng dụng Laravel không bao gồm thư mục `lang`. Nếu bạn muốn tùy chỉnh các tệp ngôn ngữ của Laravel, bạn có thể xuất bản chúng thông qua lệnh Artisan `lang:publish`.

<a name="custom-messages-for-specific-attributes"></a>
#### Thông báo Tùy chỉnh cho Các Thuộc tính Cụ thể

Bạn có thể tùy chỉnh các thông báo lỗi được sử dụng cho các kết hợp thuộc tính và quy tắc được chỉ định trong các tệp ngôn ngữ xác thực của ứng dụng. Để làm điều này, hãy thêm các tùy chỉnh thông báo của bạn vào mảng `custom` của tệp ngôn ngữ `lang/xx/validation.php` của ứng dụng:

```php
'custom' => [
    'email' => [
        'required' => 'We need to know your email address!',
        'max' => 'Your email address is too long!'
    ],
],
```

<a name="specifying-attribute-in-language-files"></a>
### Chỉ định Các Thuộc tính trong Tệp Ngôn ngữ

Nhiều thông báo lỗi tích hợp sẵn của Laravel bao gồm một placeholder `:attribute` được thay thế bằng tên của trường hoặc thuộc tính đang được xác thực. Nếu bạn muốn phần `:attribute` của thông báo xác thực của mình được thay thế bằng một giá trị tùy chỉnh, bạn có thể chỉ định tên thuộc tính tùy chỉnh trong mảng `attributes` của tệp ngôn ngữ `lang/xx/validation.php` của mình:

```php
'attributes' => [
    'email' => 'email address',
],
```

> [!WARNING]
> Theo mặc định, khung ứng dụng Laravel không bao gồm thư mục `lang`. Nếu bạn muốn tùy chỉnh các tệp ngôn ngữ của Laravel, bạn có thể xuất bản chúng thông qua lệnh Artisan `lang:publish`.

<a name="specifying-values-in-language-files"></a>
### Chỉ định Các Giá trị trong Tệp Ngôn ngữ

Một số thông báo lỗi quy tắc xác thực tích hợp sẵn của Laravel chứa một placeholder `:value` được thay thế bằng giá trị hiện tại của thuộc tính request. Tuy nhiên, đôi khi bạn có thể cần phần `:value` của thông báo xác thực của mình được thay thế bằng một biểu diễn tùy chỉnh của giá trị. Ví dụ, hãy xem xét quy tắc sau chỉ định rằng số thẻ tín dụng là bắt buộc nếu `payment_type` có giá trị là `cc`:

```php
Validator::make($request->all(), [
    'credit_card_number' => ['required_if:payment_type,cc']
]);
```

Nếu quy tắc xác thực này thất bại, nó sẽ tạo ra thông báo lỗi sau:

```text
The credit card number field is required when payment type is cc.
```

Thay vì hiển thị `cc` làm giá trị loại thanh toán, bạn có thể chỉ định một biểu diễn giá trị thân thiện hơn với người dùng trong tệp ngôn ngữ `lang/xx/validation.php` của mình bằng cách định nghĩa một mảng `values`:

```php
'values' => [
    'payment_type' => [
        'cc' => 'credit card'
    ],
],
```

> [!WARNING]
> Theo mặc định, khung ứng dụng Laravel không bao gồm thư mục `lang`. Nếu bạn muốn tùy chỉnh các tệp ngôn ngữ của Laravel, bạn có thể xuất bản chúng thông qua lệnh Artisan `lang:publish`.

Sau khi định nghĩa giá trị này, quy tắc xác thực sẽ tạo ra thông báo lỗi sau:

```text
The credit card number field is required when payment type is credit card.
```

<a name="available-validation-rules"></a>
## Các Quy tắc Xác thực Có sẵn

Dưới đây là danh sách tất cả các quy tắc xác thực có sẵn và chức năng của chúng:

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

#### Booleans

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Accepted If](#rule-accepted-if)
[Boolean](#rule-boolean)
[Declined](#rule-declined)
[Declined If](#rule-declined-if)

</div>

#### Strings

<div class="collection-method-list" markdown="1">

[Active URL](#rule-active-url)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Ascii](#rule-ascii)
[Confirmed](#rule-confirmed)
[Current Password](#rule-current-password)
[Different](#rule-different)
[Doesnt Start With](#rule-doesnt-start-with)
[Doesnt End With](#rule-doesnt-end-with)
[Email](#rule-email)
[Ends With](#rule-ends-with)
[Enum](#rule-enum)
[Hex Color](#rule-hex-color)
[In](#rule-in)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Lowercase](#rule-lowercase)
[MAC Address](#rule-mac)
[Max](#rule-max)
[Min](#rule-min)
[Not In](#rule-not-in)
[Regular Expression](#rule-regex)
[Not Regular Expression](#rule-not-regex)
[Same](#rule-same)
[Size](#rule-size)
[Starts With](#rule-starts-with)
[String](#rule-string)
[Uppercase](#rule-uppercase)
[URL](#rule-url)
[ULID](#rule-ulid)
[UUID](#rule-uuid)

</div>

#### Numbers

<div class="collection-method-list" markdown="1">

[Between](#rule-between)
[Decimal](#rule-decimal)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Integer](#rule-integer)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Max](#rule-max)
[Max Digits](#rule-max-digits)
[Min](#rule-min)
[Min Digits](#rule-min-digits)
[Multiple Of](#rule-multiple-of)
[Numeric](#rule-numeric)
[Same](#rule-same)
[Size](#rule-size)

</div>

#### Arrays

<div class="collection-method-list" markdown="1">

[Array](#rule-array)
[Between](#rule-between)
[Contains](#rule-contains)
[Doesnt Contain](#rule-doesnt-contain)
[Distinct](#rule-distinct)
[In Array](#rule-in-array)
[In Array Keys](#rule-in-array-keys)
[List](#rule-list)
[Max](#rule-max)
[Min](#rule-min)
[Size](#rule-size)

</div>

#### Dates

<div class="collection-method-list" markdown="1">

[After](#rule-after)
[After Or Equal](#rule-after-or-equal)
[Before](#rule-before)
[Before Or Equal](#rule-before-or-equal)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Timezone](#rule-timezone)

</div>

#### Files

<div class="collection-method-list" markdown="1">

[Between](#rule-between)
[Dimensions](#rule-dimensions)
[Encoding](#rule-encoding)
[Extensions](#rule-extensions)
[File](#rule-file)
[Image](#rule-image)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Size](#rule-size)

</div>

#### Database

<div class="collection-method-list" markdown="1">

[Exists](#rule-exists)
[Unique](#rule-unique)

</div>

#### Utilities

<div class="collection-method-list" markdown="1">

[Any Of](#rule-anyof)
[Bail](#rule-bail)
[Exclude](#rule-exclude)
[Exclude If](#rule-exclude-if)
[Exclude Unless](#rule-exclude-unless)
[Exclude With](#rule-exclude-with)
[Exclude Without](#rule-exclude-without)
[Filled](#rule-filled)
[Missing](#rule-missing)
[Missing If](#rule-missing-if)
[Missing Unless](#rule-missing-unless)
[Missing With](#rule-missing-with)
[Missing With All](#rule-missing-with-all)
[Nullable](#rule-nullable)
[Present](#rule-present)
[Present If](#rule-present-if)
[Present Unless](#rule-present-unless)
[Present With](#rule-present-with)
[Present With All](#rule-present-with-all)
[Prohibited](#rule-prohibited)
[Prohibited If](#rule-prohibited-if)
[Prohibited If Accepted](#rule-prohibited-if-accepted)
[Prohibited If Declined](#rule-prohibited-if-declined)
[Prohibited Unless](#rule-prohibited-unless)
[Prohibits](#rule-prohibits)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required If Accepted](#rule-required-if-accepted)
[Required If Declined](#rule-required-if-declined)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Required Array Keys](#rule-required-array-keys)
[Sometimes](#validating-when-present)

</div>

<a name="rule-accepted"></a>
#### accepted

Trường đang được xác thực phải là `"yes"`, `"on"`, `1`, `"1"`, `true`, hoặc `"true"`. Điều này hữu ích để xác nhận việc chấp nhận "Điều khoản Dịch vụ" hoặc các trường tương tự.

<a name="rule-accepted-if"></a>
#### accepted_if:anotherfield,value,...

Trường đang được xác thực phải là `"yes"`, `"on"`, `1`, `"1"`, `true`, hoặc `"true"` nếu một trường khác đang được xác thực bằng một giá trị được chỉ định. Điều này hữu ích để xác nhận việc chấp nhận "Điều khoản Dịch vụ" hoặc các trường tương tự.

<a name="rule-active-url"></a>
#### active_url

Trường đang được xác thực phải có bản ghi A hoặc AAAA hợp lệ theo hàm PHP `dns_get_record`. Tên máy chủ của URL được cung cấp được trích xuất bằng hàm PHP `parse_url` trước khi được chuyển đến `dns_get_record`.

<a name="rule-after"></a>
#### after:_date_

Trường đang được xác thực phải là một giá trị sau một ngày nhất định. Các ngày sẽ được chuyển vào hàm PHP `strtotime` để được chuyển đổi thành một instance `DateTime` hợp lệ:

```php
'start_date' => ['required', 'date', 'after:tomorrow']
```

Thay vì chuyển một chuỗi ngày để được đánh giá bởi `strtotime`, bạn có thể chỉ định một trường khác để so sánh với ngày:

```php
'finish_date' => ['required', 'date', 'after:start_date']
```

Để thuận tiện, các quy tắc dựa trên ngày có thể được xây dựng bằng trình xây dựng quy tắc `date` fluent:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->after(today()->addDays(7)),
],
```

Các phương thức `afterToday` và `todayOrAfter` có thể được sử dụng để diễn đạt fluent rằng ngày phải sau hôm nay, hoặc hôm nay hoặc sau, tương ứng:

```php
'start_date' => [
    'required',
    Rule::date()->afterToday(),
],
```

<a name="rule-after-or-equal"></a>
#### after\_or\_equal:_date_

Trường đang được xác thực phải là một giá trị sau hoặc bằng ngày nhất định. Để biết thêm thông tin, hãy xem quy tắc [after](#rule-after).

Để thuận tiện, các quy tắc dựa trên ngày có thể được xây dựng bằng trình xây dựng quy tắc `date` fluent:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->afterOrEqual(today()->addDays(7)),
],
```

<a name="rule-anyof"></a>
#### anyOf

Quy tắc xác thực `Rule::anyOf` cho phép bạn chỉ định rằng trường đang được xác thực phải thỏa mãn bất kỳ tập hợp quy tắc xác thực nào. Ví dụ, quy tắc sau sẽ xác thực rằng trường `username` là một địa chỉ email hoặc một chuỗi alpha-numeric (bao gồm dấu gạch ngang) có độ dài ít nhất 6 ký tự:

```php
use Illuminate\Validation\Rule;

'username' => [
    'required',
    Rule::anyOf([
        ['string', 'email'],
        ['string', 'alpha_dash', 'min:6'],
    ]),
],
```

<a name="rule-alpha"></a>
#### alpha

Trường đang được xác thực phải hoàn toàn là các ký tự chữ cái Unicode nằm trong [\p{L}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=) và [\p{M}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=).

Để hạn chế quy tắc xác thực này cho các ký tự trong phạm vi ASCII (`a-z` và `A-Z`), bạn có thể cung cấp tùy chọn `ascii` cho quy tắc xác thực:

```php
'username' => ['alpha:ascii'],
```

<a name="rule-alpha-dash"></a>
#### alpha_dash

Trường đang được xác thực phải hoàn toàn là các ký tự alpha-numeric Unicode nằm trong [\p{L}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=), [\p{M}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=), [\p{N}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=), cũng như dấu gạch ngang ASCII (`-`) và dấu gạch dưới ASCII (`_`).

Để hạn chế quy tắc xác thực này cho các ký tự trong phạm vi ASCII (`a-z`, `A-Z`, và `0-9`), bạn có thể cung cấp tùy chọn `ascii` cho quy tắc xác thực:

```php
'username' => ['alpha_dash:ascii'],
```

<a name="rule-alpha-num"></a>
#### alpha_num

Trường đang được xác thực phải hoàn toàn là các ký tự alpha-numeric Unicode nằm trong [\p{L}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=), [\p{M}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=), và [\p{N}](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=).

Để hạn chế quy tắc xác thực này cho các ký tự trong phạm vi ASCII (`a-z`, `A-Z`, và `0-9`), bạn có thể cung cấp tùy chọn `ascii` cho quy tắc xác thực:

```php
'username' => ['alpha_num:ascii'],
```

<a name="rule-array"></a>
#### array

Trường đang được xác thực phải là một `array` PHP.

Khi các giá trị bổ sung được cung cấp cho quy tắc `array`, mỗi khóa trong mảng đầu vào phải có mặt trong danh sách các giá trị được cung cấp cho quy tắc. Trong ví dụ sau, khóa `admin` trong mảng đầu vào không hợp lệ vì nó không nằm trong danh sách các giá trị được cung cấp cho quy tắc `array`:

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => ['array:name,username'],
]);
```

Nói chung, bạn nên luôn chỉ định các khóa mảng được phép có mặt trong mảng của mình.

<a name="rule-ascii"></a>
#### ascii

Trường đang được xác thực phải hoàn toàn là các ký tự ASCII 7-bit.

<a name="rule-bail"></a>
#### bail

Dừng chạy các quy tắc xác thực cho trường sau khi thất bại xác thực đầu tiên.

Mặc dù quy tắc `bail` sẽ chỉ dừng xác thực một trường cụ thể khi gặp thất bại xác thực, phương thức `stopOnFirstFailure` sẽ thông báo cho validator rằng nó nên dừng xác thực tất cả các thuộc tính sau khi một thất bại xác thực đã xảy ra:

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

<a name="rule-before"></a>
#### before:_date_

Trường đang được xác thực phải là một giá trị trước ngày nhất định. Các ngày sẽ được chuyển vào hàm PHP `strtotime` để được chuyển đổi thành một instance `DateTime` hợp lệ. Ngoài ra, giống như quy tắc [after](#rule-after), tên của một trường khác đang được xác thực có thể được cung cấp làm giá trị của `date`.

Để thuận tiện, các quy tắc dựa trên ngày cũng có thể được xây dựng bằng trình xây dựng quy tắc `date` fluent:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->before(today()->subDays(7)),
],
```

Các phương thức `beforeToday` và `todayOrBefore` có thể được sử dụng để diễn đạt fluent rằng ngày phải trước hôm nay, hoặc hôm nay hoặc trước, tương ứng:

```php
'start_date' => [
    'required',
    Rule::date()->beforeToday(),
],
```

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

Trường đang được xác thực phải là một giá trị trước hoặc bằng ngày nhất định. Các ngày sẽ được chuyển vào hàm PHP `strtotime` để được chuyển đổi thành một instance `DateTime` hợp lệ. Ngoài ra, giống như quy tắc [after](#rule-after), tên của một trường khác đang được xác thực có thể được cung cấp làm giá trị của `date`.

Để thuận tiện, các quy tắc dựa trên ngày cũng có thể được xây dựng bằng trình xây dựng quy tắc `date` fluent:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->beforeOrEqual(today()->subDays(7)),
],
```

<a name="rule-between"></a>
#### between:_min_,_max_

Trường đang được xác thực phải có kích thước giữa _min_ và _max_ nhất định (bao gồm). Chuỗi, số, mảng và tệp được đánh giá theo cùng cách như quy tắc [size](#rule-size).

<a name="rule-boolean"></a>
#### boolean

Trường đang được xác thực phải có thể được chuyển đổi thành boolean. Đầu vào được chấp nhận là `true`, `false`, `1`, `0`, `"1"`, và `"0"`.

Bạn có thể sử dụng tham số `strict` để chỉ coi trường hợp hợp lệ nếu giá trị của nó là `true` hoặc `false`:

```php
'foo' => ['boolean:strict']
```

<a name="rule-confirmed"></a>
#### confirmed

Trường đang được xác thực phải có một trường khớp của `{field}_confirmation`. Ví dụ, nếu trường đang được xác thực là `password`, một trường `password_confirmation` khớp phải có mặt trong đầu vào.

Bạn cũng có thể chuyển một tên trường xác nhận tùy chỉnh. Ví dụ, `confirmed:repeat_username` sẽ mong đợi trường `repeat_username` khớp với trường đang được xác thực.

<a name="rule-contains"></a>
#### contains:_foo_,_bar_,...

Trường đang được xác thực phải là một mảng chứa tất cả các giá trị tham số nhất định. Vì quy tắc này thường yêu cầu bạn `implode` một mảng, phương thức `Rule::contains` có thể được sử dụng để xây dựng quy tắc một cách fluent:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'roles' => [
        'required',
        'array',
        Rule::contains(['admin', 'editor']),
    ],
]);
```

<a name="rule-doesnt-contain"></a>
#### doesnt_contain:_foo_,_bar_,...

Trường đang được xác thực phải là một mảng không chứa bất kỳ giá trị tham số nào. Vì quy tắc này thường yêu cầu bạn `implode` một mảng, phương thức `Rule::doesntContain` có thể được sử dụng để xây dựng quy tắc một cách fluent:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'roles' => [
        'required',
        'array',
        Rule::doesntContain(['admin', 'editor']),
    ],
]);
```

<a name="rule-current-password"></a>
#### current_password

Trường đang được xác thực phải khớp với mật khẩu của người dùng đã xác thực. Bạn có thể chỉ định một [authentication guard](/docs/{{version}}/authentication) bằng tham số đầu tiên của quy tắc:

```php
'password' => ['current_password:api']
```

<a name="rule-date"></a>
#### date

Trường đang được xác thực phải là một ngày hợp lệ, không tương đối theo hàm PHP `strtotime`.

<a name="rule-date-equals"></a>
#### date_equals:_date_

Trường đang được xác thực phải bằng ngày nhất định. Các ngày sẽ được chuyển vào hàm PHP `strtotime` để được chuyển đổi thành một instance `DateTime` hợp lệ.

<a name="rule-date-format"></a>
#### date_format:_format_,...

Trường đang được xác thực phải khớp với một trong các _định dạng_ nhất định. Bạn nên sử dụng **hoặc** `date` hoặc `date_format` khi xác thực một trường, không phải cả hai. Quy tắc xác thực này hỗ trợ tất cả các định dạng được hỗ trợ bởi lớp [DateTime](https://www.php.net/manual/en/class.datetime.php) của PHP.

Để thuận tiện, các quy tắc dựa trên ngày có thể được xây dựng bằng trình xây dựng quy tắc `date` fluent:

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->format('Y-m-d'),
],
```

<a name="rule-decimal"></a>
#### decimal:_min_,_max_

Trường đang được xác thực phải là số và phải chứa số lượng vị trí thập phân được chỉ định:

```php
// Phải có chính xác hai vị trí thập phân (9.99)...
'price' => ['decimal:2']

// Phải có từ 2 đến 4 vị trí thập phân...
'price' => ['decimal:2,4']
```

<a name="rule-declined"></a>
#### declined

Trường đang được xác thực phải là `"no"`, `"off"`, `0`, `"0"`, `false`, hoặc `"false"`.

<a name="rule-declined-if"></a>
#### declined_if:anotherfield,value,...

Trường đang được xác thực phải là `"no"`, `"off"`, `0`, `"0"`, `false`, hoặc `"false"` nếu một trường khác đang được xác thực bằng một giá trị được chỉ định.

<a name="rule-different"></a>
#### different:_field_

Trường đang được xác thực phải có một giá trị khác với _field_.

<a name="rule-digits"></a>
#### digits:_value_

Số nguyên đang được xác thực phải có độ dài chính xác là _value_.

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

Số nguyên đang được xác thực phải có độ dài giữa _min_ và _max_ nhất định.

<a name="rule-dimensions"></a>
#### dimensions

Tệp đang được xác thực phải là một hình ảnh đáp ứng các ràng buộc kích thước như được chỉ định bởi các tham số của quy tắc:

```php
'avatar' => ['dimensions:min_width=100,min_height=200']
```

Các ràng buộc có sẵn là: _min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_, _min\_ratio_, _max\_ratio_.

Một ràng buộc _ratio_ nên được biểu diễn là chiều rộng chia cho chiều cao. Điều này có thể được chỉ định bằng một phân số như `3/2` hoặc một số thực như `1.5`:

```php
'avatar' => ['dimensions:ratio=3/2']
```

Các ràng buộc _min\_ratio_ và _max\_ratio_ có thể được sử dụng để định nghĩa một phạm vi các tỷ lệ khung hình chấp nhận được:

```php
'avatar' => ['dimensions:min_ratio=1/2,max_ratio=3/2']
```

Vì quy tắc này yêu cầu một số đối số, nó thường thuận tiện hơn để sử dụng phương thức `Rule::dimensions` để xây dựng quy tắc một cách fluent:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()
            ->maxWidth(1000)
            ->maxHeight(500)
            ->ratio(3 / 2),
    ],
]);
```

Bạn cũng có thể sử dụng các phương thức `minRatio`, `maxRatio`, và `ratioBetween` để định nghĩa các ràng buộc tỷ lệ một cách fluent:

```php
Rule::dimensions()->ratioBetween(min: 1 / 2, max: 3 / 2)
```
#### distinct

Khi xác thực mảng, trường đang được xác thực không được có bất kỳ giá trị trùng lặp nào:

```php
'foo.*.id' => ['distinct']
```

Distinct sử dụng so sánh biến lỏng lẻo theo mặc định. Để sử dụng so sánh nghiêm ngặt, bạn có thể thêm tham số `strict` vào định nghĩa quy tắc xác thực:

```php
'foo.*.id' => ['distinct:strict']
```

Bạn có thể thêm `ignore_case` vào các đối số của quy tắc xác thực để quy tắc bỏ qua sự khác biệt về chữ hoa/thường:

```php
'foo.*.id' => ['distinct:ignore_case']
```

<a name="rule-doesnt-start-with"></a>
#### doesnt_start_with:_foo_,_bar_,...

Trường đang được xác thực không được bắt đầu bằng một trong các giá trị đã cho.

<a name="rule-doesnt-end-with"></a>
#### doesnt_end_with:_foo_,_bar_,...

Trường đang được xác thực không được kết thúc bằng một trong các giá trị đã cho.

<a name="rule-email"></a>
#### email

Trường đang được xác thực phải được định dạng dưới dạng địa chỉ email. Quy tắc xác thực này sử dụng gói [egulias/email-validator](https://github.com/egulias/EmailValidator) để xác thực địa chỉ email. Theo mặc định, bộ xác thực `RFCValidation` được áp dụng, nhưng bạn cũng có thể áp dụng các kiểu xác thực khác:

```php
'email' => ['email:rfc,dns']
```

Ví dụ trên sẽ áp dụng các xác thực `RFCValidation` và `DNSCheckValidation`. Dưới đây là danh sách đầy đủ các kiểu xác thực bạn có thể áp dụng:

<div class="content-list" markdown="1">

- `rfc`: `RFCValidation` - Xác thực địa chỉ email theo [RFC được hỗ trợ](https://github.com/egulias/EmailValidator?tab=readme-ov-file#supported-rfcs).
- `strict`: `NoRFCWarningsValidation` - Xác thực email theo [RFC được hỗ trợ](https://github.com/egulias/EmailValidator?tab=readme-ov-file#supported-rfcs), thất bại khi tìm thấy cảnh báo (ví dụ: dấu chấm ở cuối và nhiều dấu chấm liên tiếp).
- `dns`: `DNSCheckValidation` - Đảm bảo miền của địa chỉ email có bản ghi MX hợp lệ.
- `spoof`: `SpoofCheckValidation` - Đảm bảo địa chỉ email không chứa ký tự Unicode đồng hình hoặc gây nhầm lẫn.
- `filter`: `FilterEmailValidation` - Đảm bảo địa chỉ email hợp lệ theo hàm `filter_var` của PHP.
- `filter_unicode`: `FilterEmailValidation::unicode()` - Đảm bảo địa chỉ email hợp lệ theo hàm `filter_var` của PHP, cho phép một số ký tự Unicode.

</div>

Để thuận tiện, các quy tắc xác thực email có thể được xây dựng bằng trình xây dựng quy tắc trôi chảy:

```php
use Illuminate\Validation\Rule;

$request->validate([
    'email' => [
        'required',
        Rule::email()
            ->rfcCompliant(strict: false)
            ->validateMxRecord()
            ->preventSpoofing()
    ],
]);
```

> [!WARNING]
> Các bộ xác thực `dns` và `spoof` yêu cầu phần mở rộng PHP `intl`.

<a name="rule-encoding"></a>
#### encoding:*encoding_type*

Trường đang được xác thực phải khớp với mã hóa ký tự được chỉ định. Quy tắc này sử dụng hàm `mb_check_encoding` của PHP để xác minh mã hóa của tệp hoặc giá trị chuỗi đã cho. Để thuận tiện, quy tắc `encoding` có thể được xây dựng bằng trình xây dựng quy tắc tệp trôi chảy của Laravel:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'attachment' => [
        'required',
        File::types(['csv'])
            ->encoding('utf-8'),
    ],
]);
```

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

Trường đang được xác thực phải kết thúc bằng một trong các giá trị đã cho.

<a name="rule-enum"></a>
#### enum

Quy tắc `Enum` là một quy tắc dựa trên lớp xác thực xem trường đang được xác thực có chứa giá trị enum hợp lệ hay không. Quy tắc `Enum` chấp nhận tên của enum làm đối số constructor duy nhất. Khi xác thực các giá trị nguyên thủy, một backed Enum nên được cung cấp cho quy tắc `Enum`:

```php
use App\Enums\ServerStatus;
use Illuminate\Validation\Rule;

$request->validate([
    'status' => [Rule::enum(ServerStatus::class)],
]);
```

Các phương thức `only` và `except` của quy tắc `Enum` có thể được sử dụng để giới hạn các trường hợp enum nào nên được coi là hợp lệ:

```php
Rule::enum(ServerStatus::class)
    ->only([ServerStatus::Pending, ServerStatus::Active]);

Rule::enum(ServerStatus::class)
    ->except([ServerStatus::Pending, ServerStatus::Active]);
```

Phương thức `when` có thể được sử dụng để sửa đổi có điều kiện quy tắc `Enum`:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rule;

Rule::enum(ServerStatus::class)
    ->when(
        Auth::user()->isAdmin(),
        fn ($rule) => $rule->only(...),
        fn ($rule) => $rule->only(...),
    );
```

<a name="rule-exclude"></a>
#### exclude

Trường đang được xác thực sẽ bị loại trừ khỏi dữ liệu yêu cầu được trả về bởi các phương thức `validate` và `validated`.

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

Trường đang được xác thực sẽ bị loại trừ khỏi dữ liệu yêu cầu được trả về bởi các phương thức `validate` và `validated` nếu trường _anotherfield_ bằng _value_.

Nếu cần logic loại trừ có điều kiện phức tạp, bạn có thể sử dụng phương thức `Rule::excludeIf`. Phương thức này chấp nhận một boolean hoặc một closure. Khi được cung cấp một closure, closure nên trả về `true` hoặc `false` để chỉ ra liệu trường đang được xác thực có nên bị loại trừ hay không:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => [Rule::excludeIf($request->user()->is_admin)],
]);

Validator::make($request->all(), [
    'role_id' => [Rule::excludeIf(fn () => $request->user()->is_admin)],
]);
```

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

Trường đang được xác thực sẽ bị loại trừ khỏi dữ liệu yêu cầu được trả về bởi các phương thức `validate` và `validated` trừ khi trường của _anotherfield_ bằng _value_. Nếu _value_ là `null` (`exclude_unless:name,null`), trường đang được xác thực sẽ bị loại trừ trừ khi trường so sánh là `null` hoặc trường so sánh bị thiếu trong dữ liệu yêu cầu.

Nếu cần logic loại trừ có điều kiện phức tạp, bạn có thể sử dụng phương thức `Rule::excludeUnless`. Phương thức này chấp nhận một boolean hoặc một closure. Khi được cung cấp một closure, closure nên trả về `true` hoặc `false` để chỉ ra liệu trường đang được xác thực có nên không bị loại trừ hay không:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => [Rule::excludeUnless($request->user()->is_admin)],
]);

Validator::make($request->all(), [
    'role_id' => [Rule::excludeUnless(fn () => $request->user()->is_admin)],
]);
```

<a name="rule-exclude-with"></a>
#### exclude_with:_anotherfield_

Trường đang được xác thực sẽ bị loại trừ khỏi dữ liệu yêu cầu được trả về bởi các phương thức `validate` và `validated` nếu trường _anotherfield_ có mặt.

<a name="rule-exclude-without"></a>
#### exclude_without:_anotherfield_

Trường đang được xác thực sẽ bị loại trừ khỏi dữ liệu yêu cầu được trả về bởi các phương thức `validate` và `validated` nếu trường _anotherfield_ không có mặt.

<a name="rule-exists"></a>
#### exists:_table_,_column_

Trường đang được xác thực phải tồn tại trong một bảng cơ sở dữ liệu đã cho.

<a name="basic-usage-of-exists-rule"></a>
#### Basic Usage of Exists Rule

```php
'state' => ['exists:states']
```

Nếu tùy chọn `column` không được chỉ định, tên trường sẽ được sử dụng. Vì vậy, trong trường hợp này, quy tắc sẽ xác thực rằng bảng cơ sở dữ liệu `states` chứa một bản ghi có giá trị cột `state` khớp với giá trị thuộc tính `state` của yêu cầu.

<a name="specifying-a-custom-column-name"></a>
#### Specifying a Custom Column Name

Bạn có thể chỉ định rõ tên cột cơ sở dữ liệu nên được sử dụng bởi quy tắc xác thực bằng cách đặt nó sau tên bảng cơ sở dữ liệu:

```php
'state' => ['exists:states,abbreviation']
```

Thỉnh thoảng, bạn có thể cần chỉ định một kết nối cơ sở dữ liệu cụ thể để sử dụng cho truy vấn `exists`. Bạn có thể thực hiện việc này bằng cách thêm tên kết nối vào tên bảng:

```php
'email' => ['exists:connection.staff,email']
```

Thay vì chỉ định tên bảng trực tiếp, bạn có thể chỉ định mô hình Eloquent nên được sử dụng để xác định tên bảng:

```php
'user_id' => ['exists:App\Models\User,id']
```

Nếu bạn muốn tùy chỉnh truy vấn được thực thi bởi quy tắc xác thực, bạn có thể sử dụng lớp `Rule` để xác định quy tắc một cách trôi chảy.

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function (Builder $query) {
            $query->where('account_id', 1);
        }),
    ],
]);
```

Bạn có thể chỉ định rõ tên cột cơ sở dữ liệu nên được sử dụng bởi quy tắc `exists` được tạo bởi phương thức `Rule::exists` bằng cách cung cấp tên cột làm đối số thứ hai cho phương thức `exists`:

```php
'state' => [Rule::exists('states', 'abbreviation')],
```

Đôi khi, bạn có thể muốn xác thực xem một mảng giá trị có tồn tại trong cơ sở dữ liệu hay không. Bạn có thể thực hiện việc này bằng cách thêm cả quy tắc `exists` và quy tắc [array](#rule-array) vào trường đang được xác thực:

```php
'states' => ['array', Rule::exists('states', 'abbreviation')],
```

Khi cả hai quy tắc này được gán cho một trường, Laravel sẽ tự động xây dựng một truy vấn duy nhất để xác định xem tất cả các giá trị đã cho có tồn tại trong bảng được chỉ định hay không.

<a name="rule-extensions"></a>
#### extensions:_foo_,_bar_,...

Tệp đang được xác thực phải có phần mở rộng do người dùng gán tương ứng với một trong các phần mở rộng được liệt kê:

```php
'photo' => ['required', 'extensions:jpg,png'],
```

> [!WARNING]
> Bạn không bao giờ nên dựa vào việc xác thực tệp chỉ bằng phần mở rộng do người dùng gán. Quy tắc này thường nên luôn được sử dụng kết hợp với các quy tắc [mimes](#rule-mimes) hoặc [mimetypes](#rule-mimetypes).

<a name="rule-file"></a>
#### file

Trường đang được xác thực phải là một tệp đã được tải lên thành công.

<a name="rule-filled"></a>
#### filled

Trường đang được xác thực không được để trống khi nó có mặt.

<a name="rule-gt"></a>
#### gt:_field_

Trường đang được xác thực phải lớn hơn _field_ hoặc _value_ đã cho. Hai trường phải cùng loại. Chuỗi, số, mảng và tệp được đánh giá theo các quy ước giống như quy tắc [size](#rule-size).

<a name="rule-gte"></a>
#### gte:_field_

Trường đang được xác thực phải lớn hơn hoặc bằng _field_ hoặc _value_ đã cho. Hai trường phải cùng loại. Chuỗi, số, mảng và tệp được đánh giá theo các quy ước giống như quy tắc [size](#rule-size).

<a name="rule-hex-color"></a>
#### hex_color

Trường đang được xác thực phải chứa một giá trị màu hợp lệ ở định dạng [hexadecimal](https://developer.mozilla.org/en-US/docs/Web/CSS/hex-color).

<a name="rule-image"></a>
#### image

Tệp đang được xác thực phải là một hình ảnh (jpg, jpeg, png, bmp, gif, hoặc webp).

> [!WARNING]
> Theo mặc định, quy tắc image không cho phép tệp SVG do khả năng lỗ hổng XSS. Nếu bạn cần cho phép tệp SVG, bạn có thể cung cấp chỉ thị `allow_svg` cho quy tắc `image` (`image:allow_svg`).

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

Trường đang được xác thực phải được bao gồm trong danh sách giá trị đã cho. Vì quy tắc này thường yêu cầu bạn `implode` một mảng, phương thức `Rule::in` có thể được sử dụng để xây dựng quy tắc một cách trôi chảy:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

Khi quy tắc `in` được kết hợp với quy tắc `array`, mỗi giá trị trong mảng đầu vào phải có mặt trong danh sách giá trị được cung cấp cho quy tắc `in`. Trong ví dụ sau, mã sân bay `LAS` trong mảng đầu vào không hợp lệ vì nó không được chứa trong danh sách sân bay được cung cấp cho quy tắc `in`:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$input = [
    'airports' => ['NYC', 'LAS'],
];

Validator::make($input, [
    'airports' => [
        'required',
        'array',
    ],
    'airports.*' => Rule::in(['NYC', 'LIT']),
]);
```

<a name="rule-in-array"></a>
#### in_array:_anotherfield_.*

Trường đang được xác thực phải tồn tại trong các giá trị của _anotherfield_.

<a name="rule-in-array-keys"></a>
#### in_array_keys:_value_.*

Trường đang được xác thực phải là một mảng có ít nhất một trong các _values_ đã cho làm khóa trong mảng:

```php
'config' => ['array', 'in_array_keys:timezone']
```

<a name="rule-integer"></a>
#### integer

Trường đang được xác thực phải là một số nguyên.

Bạn có thể sử dụng tham số `strict` để chỉ coi trường hợp hợp lệ nếu loại của nó là `integer`. Chuỗi có giá trị số nguyên sẽ được coi là không hợp lệ:

```php
'age' => ['integer:strict']
```

> [!WARNING]
> Quy tắc xác thực này không xác minh rằng đầu vào là loại biến "integer", chỉ xác nhận rằng đầu vào là một loại được chấp nhận bởi quy tắc `FILTER_VALIDATE_INT` của PHP. Nếu bạn cần xác thực đầu vào là một số, vui lòng sử dụng quy tắc này kết hợp với [quy tắc xác thực `numeric`](#rule-numeric).

<a name="rule-ip"></a>
#### ip

Trường đang được xác thực phải là một địa chỉ IP.

<a name="ipv4"></a>
#### ipv4

Trường đang được xác thực phải là một địa chỉ IPv4.

<a name="ipv6"></a>
#### ipv6

Trường đang được xác thực phải là một địa chỉ IPv6.

<a name="rule-json"></a>
#### json

Trường đang được xác thực phải là một chuỗi JSON hợp lệ.

<a name="rule-lt"></a>
#### lt:_field_

Trường đang được xác thực phải nhỏ hơn _field_ đã cho. Hai trường phải cùng loại. Chuỗi, số, mảng và tệp được đánh giá theo các quy ước giống như quy tắc [size](#rule-size).

<a name="rule-lte"></a>
#### lte:_field_

Trường đang được xác thực phải nhỏ hơn hoặc bằng _field_ đã cho. Hai trường phải cùng loại. Chuỗi, số, mảng và tệp được đánh giá theo các quy ước giống như quy tắc [size](#rule-size).

<a name="rule-lowercase"></a>
#### lowercase

Trường đang được xác thực phải là chữ thường.

<a name="rule-list"></a>
#### list

Trường đang được xác thực phải là một mảng là một danh sách. Một mảng được coi là danh sách nếu các khóa của nó bao gồm các số liên tiếp từ 0 đến `count($array) - 1`.

<a name="rule-mac"></a>
#### mac_address

Trường đang được xác thực phải là một địa chỉ MAC.

<a name="rule-max"></a>
#### max:_value_

Trường đang được xác thực phải nhỏ hơn hoặc bằng một _value_ tối đa. Chuỗi, số, mảng và tệp được đánh giá theo cùng cách như quy tắc [size](#rule-size).

<a name="rule-max-digits"></a>
#### max_digits:_value_

Số nguyên đang được xác thực phải có độ dài tối đa là _value_.

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

Tệp đang được xác thực phải khớp với một trong các loại MIME đã cho:

```php
'video' => ['mimetypes:video/avi,video/mpeg,video/quicktime'],

'media' => ['mimetypes:image/*,video/*'],
```

Để xác định loại MIME của tệp đã tải lên, nội dung của tệp sẽ được đọc và framework sẽ cố gắng đoán loại MIME, có thể khác với loại MIME do máy khách cung cấp.

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

Tệp đang được xác thực phải có loại MIME tương ứng với một trong các phần mở rộng được liệt kê:

```php
'photo' => ['mimes:jpg,bmp,png']
```

Mặc dù bạn chỉ cần chỉ định các phần mở rộng, quy tắc này thực sự xác thực loại MIME của tệp bằng cách đọc nội dung của tệp và đoán loại MIME của nó. Danh sách đầy đủ các loại MIME và các phần mở rộng tương ứng của chúng có thể được tìm thấy tại vị trí sau:

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="mime-types-and-extensions"></a>
#### MIME Types and Extensions

Quy tắc xác thực này không xác minh sự đồng thuận giữa loại MIME và phần mở rộng mà người dùng gán cho tệp. Ví dụ, quy tắc xác thực `mimes:png` sẽ coi một tệp chứa nội dung PNG hợp lệ là một hình ảnh PNG hợp lệ, ngay cả khi tệp được đặt tên là `photo.txt`. Nếu bạn muốn xác thực phần mở rộng do người dùng gán của tệp, bạn có thể sử dụng quy tắc [extensions](#rule-extensions).

<a name="rule-min"></a>
#### min:_value_

Trường đang được xác thực phải có một _value_ tối thiểu. Chuỗi, số, mảng và tệp được đánh giá theo cùng cách như quy tắc [size](#rule-size).

<a name="rule-min-digits"></a>
#### min_digits:_value_

Số nguyên đang được xác thực phải có độ dài tối thiểu là _value_.

<a name="rule-multiple-of"></a>
#### multiple_of:_value_

Trường đang được xác thực phải là bội số của _value_.

<a name="rule-missing"></a>
#### missing

Trường đang được xác thực không được có mặt trong dữ liệu đầu vào.

<a name="rule-missing-if"></a>
#### missing_if:_anotherfield_,_value_,...

Trường đang được xác thực không được có mặt nếu trường _anotherfield_ bằng bất kỳ _value_ nào.

<a name="rule-missing-unless"></a>
#### missing_unless:_anotherfield_,_value_

Trường đang được xác thực không được có mặt trừ khi trường _anotherfield_ bằng bất kỳ _value_ nào.

<a name="rule-missing-with"></a>
#### missing_with:_foo_,_bar_,...

Trường đang được xác thực không được có mặt _chỉ nếu_ bất kỳ trường nào khác được chỉ định có mặt.

<a name="rule-missing-with-all"></a>
#### missing_with_all:_foo_,_bar_,...

Trường đang được xác thực không được có mặt _chỉ nếu_ tất cả các trường khác được chỉ định có mặt.

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

Trường đang được xác thực không được bao gồm trong danh sách giá trị đã cho. Phương thức `Rule::notIn` có thể được sử dụng để xây dựng quy tắc một cách trôi chảy:

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'toppings' => [
        'required',
        Rule::notIn(['sprinkles', 'cherries']),
    ],
]);
```

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

Trường đang được xác thực không được khớp với biểu thức chính quy đã cho.

Bên trong, quy tắc này sử dụng hàm `preg_match` của PHP. Mẫu được chỉ định nên tuân theo cùng định dạng được yêu cầu bởi `preg_match` và do đó cũng bao gồm các dấu phân cách hợp lệ. Ví dụ: `'email' => ['not_regex:/^.+$/i']`.

<a name="rule-nullable"></a>
#### nullable

Trường đang được xác thực có thể là `null`.

<a name="rule-numeric"></a>
#### numeric

Trường đang được xác thực phải là [numeric](https://www.php.net/manual/en/function.is-numeric.php).

Bạn có thể sử dụng tham số `strict` để chỉ coi trường hợp hợp lệ nếu giá trị của nó là loại số nguyên hoặc float. Chuỗi số sẽ được coi là không hợp lệ:

```php
'amount' => ['numeric:strict']
```

<a name="rule-present"></a>
#### present

Trường đang được xác thực phải tồn tại trong dữ liệu đầu vào.

<a name="rule-present-if"></a>
#### present_if:_anotherfield_,_value_,...

Trường đang được xác thực phải có mặt nếu trường _anotherfield_ bằng bất kỳ _value_ nào.

<a name="rule-present-unless"></a>
#### present_unless:_anotherfield_,_value_

Trường đang được xác thực phải có mặt trừ khi trường _anotherfield_ bằng bất kỳ _value_ nào.

<a name="rule-present-with"></a>
#### present_with:_foo_,_bar_,...

Trường đang được xác thực phải có mặt _chỉ nếu_ bất kỳ trường nào khác được chỉ định có mặt.

<a name="rule-present-with-all"></a>
#### present_with_all:_foo_,_bar_,...

Trường đang được xác thực phải có mặt _chỉ nếu_ tất cả các trường khác được chỉ định có mặt.

<a name="rule-prohibited"></a>
#### prohibited

Trường đang được xác thực phải bị thiếu hoặc để trống. Một trường được coi là "để trống" nếu nó đáp ứng một trong các tiêu chí sau:

<div class="content-list" markdown="1">

- Giá trị là `null`.
- Giá trị là một chuỗi rỗng.
- Giá trị là một mảng rỗng hoặc đối tượng `Countable` rỗng.
- Giá trị là một tệp đã tải lên có đường dẫn rỗng.

</div>

<a name="rule-prohibited-if"></a>
#### prohibited_if:_anotherfield_,_value_,...

Trường đang được xác thực phải bị thiếu hoặc để trống nếu trường _anotherfield_ bằng bất kỳ _value_ nào. Một trường được coi là "để trống" nếu nó đáp ứng một trong các tiêu chí sau:

<div class="content-list" markdown="1">

- Giá trị là `null`.
- Giá trị là một chuỗi rỗng.
- Giá trị là một mảng rỗng hoặc đối tượng `Countable` rỗng.
- Giá trị là một tệp đã tải lên có đường dẫn rỗng.

</div>

Nếu cần logic cấm có điều kiện phức tạp, bạn có thể sử dụng phương thức `Rule::prohibitedIf`. Phương thức này chấp nhận một boolean hoặc một closure. Khi được cung cấp một closure, closure nên trả về `true` hoặc `false` để chỉ ra liệu trường đang được xác thực có nên bị cấm hay không:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => [Rule::prohibitedIf($request->user()->is_admin)],
]);

Validator::make($request->all(), [
    'role_id' => [Rule::prohibitedIf(fn () => $request->user()->is_admin)],
]);
```
<a name="rule-prohibited-if-accepted"></a>
#### prohibited_if_accepted:_anotherfield_,...

Trường đang được xác thực phải bị thiếu hoặc để trống nếu trường _anotherfield_ bằng `"yes"`, `"on"`, `1`, `"1"`, `true`, hoặc `"true"`.

<a name="rule-prohibited-if-declined"></a>
#### prohibited_if_declined:_anotherfield_,...

Trường đang được xác thực phải bị thiếu hoặc để trống nếu trường _anotherfield_ bằng `"no"`, `"off"`, `0`, `"0"`, `false`, hoặc `"false"`.

<a name="rule-prohibited-unless"></a>
#### prohibited_unless:_anotherfield_,_value_,...

Trường đang được xác thực phải bị thiếu hoặc để trống trừ khi trường _anotherfield_ bằng bất kỳ _value_ nào. Một trường được coi là "để trống" nếu nó đáp ứng một trong các tiêu chí sau:

<div class="content-list" markdown="1">

- Giá trị là `null`.
- Giá trị là một chuỗi rỗng.
- Giá trị là một mảng rỗng hoặc đối tượng `Countable` rỗng.
- Giá trị là một tệp đã tải lên có đường dẫn rỗng.

</div>

Nếu cần logic cấm có điều kiện phức tạp, bạn có thể sử dụng phương thức `Rule::prohibitedUnless`. Phương thức này chấp nhận một boolean hoặc một closure. Khi được cung cấp một closure, closure nên trả về `true` hoặc `false` để chỉ ra liệu trường đang được xác thực có nên không bị cấm hay không:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => [Rule::prohibitedUnless($request->user()->is_admin)],
]);

Validator::make($request->all(), [
    'role_id' => [Rule::prohibitedUnless(fn () => $request->user()->is_admin)],
]);
```

<a name="rule-prohibits"></a>
#### prohibits:_anotherfield_,...

Nếu trường đang được xác thực không bị thiếu hoặc để trống, tất cả các trường trong _anotherfield_ phải bị thiếu hoặc để trống. Một trường được coi là "để trống" nếu nó đáp ứng một trong các tiêu chí sau:

<div class="content-list" markdown="1">

- Giá trị là `null`.
- Giá trị là một chuỗi rỗng.
- Giá trị là một mảng rỗng hoặc đối tượng `Countable` rỗng.
- Giá trị là một tệp đã tải lên có đường dẫn rỗng.

</div>

<a name="rule-regex"></a>
#### regex:_pattern_

Trường đang được xác thực phải khớp với biểu thức chính quy đã cho.

Bên trong, quy tắc này sử dụng hàm `preg_match` của PHP. Mẫu được chỉ định nên tuân theo cùng định dạng được yêu cầu bởi `preg_match` và do đó cũng bao gồm các dấu phân cách hợp lệ. Ví dụ: `'email' => ['regex:/^.+@.+$/i']`.

<a name="rule-required"></a>
#### required

Trường đang được xác thực phải có mặt trong dữ liệu đầu vào và không được để trống. Một trường được coi là "để trống" nếu nó đáp ứng một trong các tiêu chí sau:

<div class="content-list" markdown="1">

- Giá trị là `null`.
- Giá trị là một chuỗi rỗng.
- Giá trị là một mảng rỗng hoặc đối tượng `Countable` rỗng.
- Giá trị là một tệp đã tải lên không có đường dẫn.

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

Trường đang được xác thực phải có mặt và không được để trống nếu trường _anotherfield_ bằng bất kỳ _value_ nào.

Nếu bạn muốn xây dựng một điều kiện phức tạp hơn cho quy tắc `required_if`, bạn có thể sử dụng phương thức `Rule::requiredIf`. Phương thức này chấp nhận một boolean hoặc một closure. Khi được truyền một closure, closure nên trả về `true` hoặc `false` để chỉ ra liệu trường đang được xác thực có được yêu cầu hay không:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => [Rule::requiredIf($request->user()->is_admin)],
]);

Validator::make($request->all(), [
    'role_id' => [Rule::requiredIf(fn () => $request->user()->is_admin)],
]);
```

<a name="rule-required-if-accepted"></a>
#### required_if_accepted:_anotherfield_,...

Trường đang được xác thực phải có mặt và không được để trống nếu trường _anotherfield_ bằng `"yes"`, `"on"`, `1`, `"1"`, `true`, hoặc `"true"`.

<a name="rule-required-if-declined"></a>
#### required_if_declined:_anotherfield_,...

Trường đang được xác thực phải có mặt và không được để trống nếu trường _anotherfield_ bằng `"no"`, `"off"`, `0`, `"0"`, `false`, hoặc `"false"`.

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

Trường đang được xác thực phải có mặt và không được để trống trừ khi trường _anotherfield_ bằng bất kỳ _value_ nào. Điều này cũng có nghĩa là _anotherfield_ phải có mặt trong dữ liệu yêu cầu trừ khi _value_ là `null`. Nếu _value_ là `null` (`required_unless:name,null`), trường đang được xác thực sẽ được yêu cầu trừ khi trường so sánh là `null` hoặc trường so sánh bị thiếu trong dữ liệu yêu cầu.

Nếu bạn muốn xây dựng một điều kiện phức tạp hơn cho quy tắc `required_unless`, bạn có thể sử dụng phương thức `Rule::requiredUnless`. Phương thức này chấp nhận một boolean hoặc một closure. Khi được truyền một closure, closure nên trả về `true` hoặc `false` để chỉ ra liệu trường đang được xác thực có không được yêu cầu hay không:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => [Rule::requiredUnless($request->user()->is_admin)],
]);

Validator::make($request->all(), [
    'role_id' => [Rule::requiredUnless(fn () => $request->user()->is_admin)],
]);
```

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

Trường đang được xác thực phải có mặt và không được để trống _chỉ nếu_ bất kỳ trường nào khác được chỉ định có mặt và không để trống.

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

Trường đang được xác thực phải có mặt và không được để trống _chỉ nếu_ tất cả các trường khác được chỉ định có mặt và không để trống.

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

Trường đang được xác thực phải có mặt và không được để trống _chỉ khi_ bất kỳ trường nào khác được chỉ định để trống hoặc không có mặt.

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

Trường đang được xác thực phải có mặt và không được để trống _chỉ khi_ tất cả các trường khác được chỉ định để trống hoặc không có mặt.

<a name="rule-required-array-keys"></a>
#### required_array_keys:_foo_,_bar_,...

Trường đang được xác thực phải là một mảng và phải chứa ít nhất các khóa được chỉ định.

<a name="rule-same"></a>
#### same:_field_

_field_ đã cho phải khớp với trường đang được xác thực.

<a name="rule-size"></a>
#### size:_value_

Trường đang được xác thực phải có kích thước khớp với _value_ đã cho. Đối với dữ liệu chuỗi, _value_ tương ứng với số ký tự. Đối với dữ liệu số, _value_ tương ứng với một giá trị số nguyên đã cho (thuộc tính cũng phải có quy tắc `numeric` hoặc `integer`). Đối với một mảng, _size_ tương ứng với `count` của mảng. Đối với tệp, _size_ tương ứng với kích thước tệp tính bằng kilobyte. Hãy xem một số ví dụ:

```php
// Xác thực rằng một chuỗi có chính xác 12 ký tự...
'title' => ['size:12'];

// Xác thực rằng một số nguyên được cung cấp bằng 10...
'seats' => ['integer', 'size:10'];

// Xác thực rằng một mảng có chính xác 5 phần tử...
'tags' => ['array', 'size:5'];

// Xác thực rằng một tệp đã tải lên chính xác là 512 kilobyte...
'image' => ['file', 'size:512'];
```

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

Trường đang được xác thực phải bắt đầu bằng một trong các giá trị đã cho.

<a name="rule-string"></a>
#### string

Trường đang được xác thực phải là một chuỗi. Nếu bạn muốn cho phép trường cũng là `null`, bạn nên gán quy tắc `nullable` cho trường.

Để thuận tiện, các quy tắc xác thực chuỗi cũng có thể được xây dựng bằng trình xây dựng quy tắc trôi chảy `Rule::string()`:

```php
use Illuminate\Validation\Rule;

'title' => [
    'required',
    Rule::string()
        ->min(3)
        ->max(255)
        ->alphaDash(ascii: true),
],
```

Trình xây dựng quy tắc chuỗi cung cấp các phương thức cho các ràng buộc chuỗi phổ biến, bao gồm `alpha`, `alphaDash`, `alphaNumeric`, `ascii`, `between`, `doesntEndWith`, `doesntStartWith`, `endsWith`, `exactly`, `lowercase`, `max`, `min`, `startsWith`, và `uppercase`. Vì trình xây dựng quy tắc có thể điều kiện, bạn cũng có thể sử dụng các phương thức `when` và `unless` để áp dụng có điều kiện các ràng buộc.

<a name="rule-timezone"></a>
#### timezone

Trường đang được xác thực phải là một định danh múi giờ hợp lệ theo phương thức `DateTimeZone::listIdentifiers`.

Các đối số [được chấp nhận bởi phương thức `DateTimeZone::listIdentifiers`](https://www.php.net/manual/en/datetimezone.listidentifiers.php) cũng có thể được cung cấp cho quy tắc xác thực này:
```php
'timezone' => ['required', 'timezone:all'];

'timezone' => ['required', 'timezone:Africa'];

'timezone' => ['required', 'timezone:per_country,US'];
```

<a name="rule-unique"></a>
#### unique:_table_,_column_

Trường đang được xác thực không được tồn tại trong bảng cơ sở dữ liệu đã cho.

**Chỉ định Tên Bảng / Cột Tùy chỉnh:**

Thay vì chỉ định tên bảng trực tiếp, bạn có thể chỉ định model Eloquent nên được sử dụng để xác định tên bảng:

```php
'email' => ['unique:App\Models\User,email_address']
```

Tùy chọn `column` có thể được sử dụng để chỉ định cột cơ sở dữ liệu tương ứng của trường. Nếu tùy chọn `column` không được chỉ định, tên của trường đang được xác thực sẽ được sử dụng.

```php
'email' => ['unique:users,email_address']
```

**Chỉ định Kết nối Cơ sở Dữ Liệu Tùy chỉnh**

Thỉnh thoảng, bạn có thể cần thiết lập kết nối tùy chỉnh cho các truy vấn cơ sở dữ liệu được thực hiện bởi Validator. Để thực hiện việc này, bạn có thể thêm tiền tố tên kết nối vào tên bảng:

```php
'email' => ['unique:connection.users,email_address']
```

**Buộc Quy tắc Unique Bỏ qua Một ID Cho trước:**

Đôi khi, bạn có thể muốn bỏ qua một ID nhất định trong quá trình xác thực unique. Ví dụ, hãy xem xét màn hình "cập nhật hồ sơ" bao gồm tên, địa chỉ email và vị trí của người dùng. Bạn có thể muốn xác minh rằng địa chỉ email là duy nhất. Tuy nhiên, nếu người dùng chỉ thay đổi trường tên và không thay đổi trường email, bạn không muốn một lỗi xác thực được ném ra vì người dùng đã là chủ sở hữu của địa chỉ email đó.

Để hướng dẫn validator bỏ qua ID của người dùng, chúng ta sẽ sử dụng class `Rule` để định nghĩa quy tắc một cách trôi chảy.

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

> [!WARNING]
> Bạn không bao giờ nên truyền bất kỳ đầu vào yêu cầu do người dùng kiểm soát vào phương thức `ignore`. Thay vào đó, bạn chỉ nên truyền một ID duy nhất được tạo bởi hệ thống như ID tự tăng hoặc UUID từ instance model Eloquent. Nếu không, ứng dụng của bạn sẽ dễ bị tấn công SQL injection.

Thay vì truyền giá trị khóa model vào phương thức `ignore`, bạn cũng có thể truyền toàn bộ instance model. Laravel sẽ tự động trích xuất khóa từ model:

```php
Rule::unique('users')->ignore($user)
```

Nếu bảng của bạn sử dụng tên cột khóa chính khác `id`, bạn có thể chỉ định tên cột khi gọi phương thức `ignore`:

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

Theo mặc định, quy tắc `unique` sẽ kiểm tra tính duy nhất của cột khớp với tên của thuộc tính đang được xác thực. Tuy nhiên, bạn có thể truyền một tên cột khác làm đối số thứ hai cho phương thức `unique`:

```php
Rule::unique('users', 'email_address')->ignore($user->id)
```

**Thêm Các Mệnh đề Where Bổ sung:**

Bạn có thể chỉ định các điều kiện truy vấn bổ sung bằng cách tùy chỉnh truy vấn sử dụng phương thức `where`. Ví dụ, hãy thêm điều kiện truy vấn giới hạn truy vấn chỉ tìm kiếm các bản ghi có giá trị cột `account_id` là `1`:

```php
'email' => Rule::unique('users')->where(fn (Builder $query) => $query->where('account_id', 1))
```

**Bỏ qua Các Bản Ghi Soft Deleted trong Kiểm tra Unique:**

Theo mặc định, quy tắc unique bao gồm các bản ghi soft deleted khi xác định tính duy nhất. Để loại bỏ các bản ghi soft deleted khỏi kiểm tra tính duy nhất, bạn có thể gọi phương thức `withoutTrashed`:

```php
Rule::unique('users')->withoutTrashed();
```

Nếu model của bạn sử dụng tên cột khác `deleted_at` cho các bản ghi soft deleted, bạn có thể cung cấp tên cột khi gọi phương thức `withoutTrashed`:

```php
Rule::unique('users')->withoutTrashed('was_deleted_at');
```

<a name="rule-uppercase"></a>
#### uppercase

Trường đang được xác thực phải là chữ hoa.

<a name="rule-url"></a>
#### url

Trường đang được xác thực phải là một URL hợp lệ.

Nếu bạn muốn chỉ định các giao thức URL nên được coi là hợp lệ, bạn có thể truyền các giao thức dưới dạng tham số quy tắc xác thực:

```php
'url' => ['url:http,https'],

'game' => ['url:minecraft,steam'],
```

<a name="rule-ulid"></a>
#### ulid

Trường đang được xác thực phải là một [Mã định danh duy nhất toàn cầu có thể sắp xếp theo từ điển](https://github.com/ulid/spec) (ULID) hợp lệ.

<a name="rule-uuid"></a>
#### uuid

Trường đang được xác thực phải là một mã định danh duy nhất toàn cầu (UUID) RFC 9562 (phiên bản 1, 3, 4, 5, 6, 7, hoặc 8) hợp lệ.

Bạn cũng có thể xác thực rằng UUID đã cho khớp với đặc tả UUID theo phiên bản:

```php
'uuid' => ['uuid:4']
```

<a name="conditionally-adding-rules"></a>
## Điều kiện Thêm Quy tắc

<a name="skipping-validation-when-fields-have-certain-values"></a>
#### Bỏ qua Xác thực Khi Các Trường Có Giá Trị Nhất định

Đôi khi bạn có thể muốn không xác thực một trường nhất định nếu một trường khác có một giá trị nhất định. Bạn có thể thực hiện việc này sử dụng quy tắc xác thực `exclude_if`. Trong ví dụ này, các trường `appointment_date` và `doctor_name` sẽ không được xác thực nếu trường `has_appointment` có giá trị là `false`:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, [
    'has_appointment' => ['required', 'boolean'],
    'appointment_date' => ['exclude_if:has_appointment,false', 'required', 'date'],
    'doctor_name' => ['exclude_if:has_appointment,false', 'required', 'string'],
]);
```

Ngoài ra, bạn có thể sử dụng quy tắc `exclude_unless` để không xác thực một trường nhất định trừ khi một trường khác có một giá trị nhất định:

```php
$validator = Validator::make($data, [
    'has_appointment' => ['required', 'boolean'],
    'appointment_date' => ['exclude_unless:has_appointment,true', 'required', 'date'],
    'doctor_name' => ['exclude_unless:has_appointment,true', 'required', 'string'],
]);
```

<a name="validating-when-present"></a>
#### Xác thực Khi Có Mặt

Trong một số tình huống, bạn có thể muốn chạy các kiểm tra xác thực đối với một trường **chỉ** nếu trường đó có mặt trong dữ liệu đang được xác thực. Để thực hiện nhanh việc này, hãy thêm quy tắc `sometimes` vào danh sách quy tắc của bạn:

```php
$validator = Validator::make($data, [
    'email' => ['sometimes', 'required', 'email'],
]);
```

Trong ví dụ trên, trường `email` sẽ chỉ được xác thực nếu nó có mặt trong mảng `$data`.

> [!NOTE]
> Nếu bạn đang cố gắng xác thực một trường nên luôn có mặt nhưng có thể trống, hãy xem [ghi chú này về các trường tùy chọn](#a-note-on-optional-fields).

<a name="complex-conditional-validation"></a>
#### Xác thực Điều kiện Phức tạp

Đôi khi bạn có thể muốn thêm các quy tắc xác thực dựa trên logic điều kiện phức tạp hơn. Ví dụ, bạn có thể muốn yêu cầu một trường nhất định chỉ khi một trường khác có giá trị lớn hơn 100. Hoặc, bạn có thể cần hai trường có một giá trị nhất định chỉ khi một trường khác có mặt. Thêm các quy tắc xác thực này không cần phải đau đầu. Đầu tiên, tạo một instance `Validator` với các _quy tắc tĩnh_ của bạn không bao giờ thay đổi:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'email' => ['required', 'email'],
    'games' => ['required', 'integer', 'min:0'],
]);
```

Hãy giả định ứng dụng web của chúng ta dành cho những người sưu tập game. Nếu một người sưu tập game đăng ký với ứng dụng của chúng ta và họ sở hữu hơn 100 game, chúng ta muốn họ giải thích lý do tại sao họ sở hữu nhiều game như vậy. Ví dụ, có thể họ điều hành một cửa hàng bán lại game, hoặc có thể họ chỉ thích sưu tập game. Để thêm yêu cầu này có điều kiện, chúng ta có thể sử dụng phương thức `sometimes` trên instance `Validator`.

```php
use Illuminate\Support\Fluent;

$validator->sometimes('reason', ['required', 'max:500'], function (Fluent $input) {
    return $input->games >= 100;
});
```

Đối số đầu tiên được truyền cho phương thức `sometimes` là tên của trường chúng ta đang xác thực có điều kiện. Đối số thứ hai là danh sách các quy tắc chúng ta muốn thêm. Nếu closure được truyền làm đối số thứ ba trả về `true`, các quy tắc sẽ được thêm. Phương thức này giúp việc xây dựng các xác thực điều kiện phức tạp trở nên dễ dàng. Bạn thậm chí có thể thêm các xác thực điều kiện cho nhiều trường cùng một lúc:

```php
$validator->sometimes(['reason', 'cost'], 'required', function (Fluent $input) {
    return $input->games >= 100;
});
```

> [!NOTE]
> Tham số `$input` được truyền cho closure của bạn sẽ là một instance của `Illuminate\Support\Fluent` và có thể được sử dụng để truy cập đầu vào và tệp của bạn đang được xác thực.

<a name="complex-conditional-array-validation"></a>
#### Xác thực Mảng Điều kiện Phức tạp

Đôi khi bạn có thể muốn xác thực một trường dựa trên một trường khác trong cùng một mảng lồng nhau mà bạn không biết chỉ mục của nó. Trong những tình huống này, bạn có thể cho phép closure của bạn nhận một đối số thứ hai sẽ là mục cá nhân hiện tại trong mảng đang được xác thực:

```php
$input = [
    'channels' => [
        [
            'type' => 'email',
            'address' => 'abigail@example.com',
        ],
        [
            'type' => 'url',
            'address' => 'https://example.com',
        ],
    ],
];

$validator->sometimes('channels.*.address', 'email', function (Fluent $input, Fluent $item) {
    return $item->type === 'email';
});

$validator->sometimes('channels.*.address', 'url', function (Fluent $input, Fluent $item) {
    return $item->type !== 'email';
});
```

Giống như tham số `$input` được truyền cho closure, tham số `$item` là một instance của `Illuminate\Support\Fluent` khi dữ liệu thuộc tính là một mảng; nếu không, nó là một chuỗi.

<a name="validating-arrays"></a>
## Xác thực Mảng

Như đã thảo luận trong [tài liệu quy tắc xác thực mảng](#rule-array), quy tắc `array` chấp nhận danh sách các khóa mảng được phép. Nếu bất kỳ khóa bổ sung nào có mặt trong mảng, xác thực sẽ thất bại:

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => ['array:name,username'],
]);
```

Nói chung, bạn nên luôn chỉ định các khóa mảng được phép có mặt trong mảng của bạn. Nếu không, các phương thức `validate` và `validated` của validator sẽ trả về tất cả dữ liệu đã xác thực, bao gồm mảng và tất cả các khóa của nó, ngay cả khi những khóa đó không được xác thực bởi các quy tắc xác thực mảng lồng nhau khác.

<a name="validating-nested-array-input"></a>
### Xác thực Đầu vào Mảng Lồng nhau

Xác thực các trường đầu vào biểu mẫu dựa trên mảng lồng nhau không cần phải đau đầu. Bạn có thể sử dụng "ký hiệu dấu chấm" để xác thực các thuộc tính trong một mảng. Ví dụ, nếu yêu cầu HTTP đến chứa một trường `photos[profile]`, bạn có thể xác thực nó như sau:

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.profile' => ['required', 'image'],
]);
```

Bạn cũng có thể xác thực từng phần tử của một mảng. Ví dụ, để xác thực rằng mỗi email trong một trường đầu vào mảng đã cho là duy nhất, bạn có thể làm như sau:

```php
$validator = Validator::make($request->all(), [
    'users.*.email' => ['email', 'unique:users'],
    'users.*.first_name' => ['required_with:users.*.last_name'],
]);
```

Tương tự, bạn có thể sử dụng ký tự `*` khi chỉ định [thông báo xác thực tùy chỉnh trong tệp ngôn ngữ của bạn](#custom-messages-for-specific-attributes), giúp việc sử dụng một thông báo xác thực duy nhất cho các trường dựa trên mảng trở nên dễ dàng:

```php
'custom' => [
    'users.*.email' => [
        'unique' => 'Each user must have a unique email address',
    ]
],
```

<a name="accessing-nested-array-data"></a>
#### Truy cập Dữ liệu Mảng Lồng nhau

Đôi khi bạn có thể cần truy cập giá trị cho một phần tử mảng lồng nhau nhất định khi gán các quy tắc xác thực cho thuộc tính. Bạn có thể thực hiện việc này sử dụng phương thức `Rule::forEach`. Phương thức `forEach` chấp nhận một closure sẽ được gọi cho mỗi lần lặp của thuộc tính mảng đang được xác thực và sẽ nhận giá trị của thuộc tính và tên thuộc tính rõ ràng, được mở rộng đầy đủ. Closure nên trả về một mảng các quy tắc để gán cho phần tử mảng:

```php
use App\Rules\HasPermission;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$validator = Validator::make($request->all(), [
    'companies.*.id' => Rule::forEach(function (string|null $value, string $attribute) {
        return [
            Rule::exists(Company::class, 'id'),
            new HasPermission('manage-company', $value),
        ];
    }),
]);
```

<a name="error-message-indexes-and-positions"></a>
### Chỉ mục và Vị trí Thông báo Lỗi

Khi xác thực mảng, bạn có thể muốn tham chiếu chỉ mục hoặc vị trí của một mục cụ thể thất bại trong xác thực trong thông báo lỗi được hiển thị bởi ứng dụng của bạn. Để thực hiện việc này, bạn có thể bao gồm các placeholder `:index` (bắt đầu từ `0`), `:position` (bắt đầu từ `1`), hoặc `:ordinal-position` (bắt đầu từ `1st`) trong [thông báo xác thực tùy chỉnh](#manual-customizing-the-error-messages) của bạn:

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'photos' => [
        [
            'name' => 'BeachVacation.jpg',
            'description' => 'A photo of my beach vacation!',
        ],
        [
            'name' => 'GrandCanyon.jpg',
            'description' => '',
        ],
    ],
];

Validator::validate($input, [
    'photos.*.description' => ['required'],
], [
    'photos.*.description.required' => 'Please describe photo #:position.',
]);
```

Với ví dụ trên, xác thực sẽ thất bại và người dùng sẽ được hiển thị lỗi sau: _"Please describe photo #2."_

Nếu cần thiết, bạn có thể tham chiếu các chỉ mục và vị trí lồng nhau sâu hơn thông qua `second-index`, `second-position`, `third-index`, `third-position`, v.v.

```php
'photos.*.attributes.*.string' => 'Invalid attribute for photo #:second-position.',
```

<a name="validating-files"></a>
## Xác thực Tệp

Laravel cung cấp nhiều quy tắc xác thực có thể được sử dụng để xác thực các tệp đã tải lên, chẳng hạn như `mimes`, `image`, `min`, và `max`. Trong khi bạn tự do chỉ định các quy tắc này riêng lẻ khi xác thực tệp, Laravel cũng cung cấp một trình xây dựng quy tắc xác thực tệp trôi chảy mà bạn có thể thấy tiện lợi:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'attachment' => [
        'required',
        File::types(['mp3', 'wav'])
            ->min(1024)
            ->max(12 * 1024),
    ],
]);
```

<a name="validating-files-file-types"></a>
#### Xác thực Loại Tệp

Mặc dù bạn chỉ cần chỉ định các phần mở rộng khi gọi phương thức `types`, phương thức này thực sự xác thực loại MIME của tệp bằng cách đọc nội dung của tệp và đoán loại MIME của nó. Danh sách đầy đủ các loại MIME và các phần mở rộng tương ứng của chúng có thể được tìm thấy tại vị trí sau:

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="validating-files-file-sizes"></a>
#### Xác thực Kích thước Tệp

Để thuận tiện, kích thước tệp tối thiểu và tối đa có thể được chỉ định dưới dạng chuỗi với hậu tố chỉ định đơn vị kích thước tệp. Các hậu tố `kb`, `mb`, `gb`, và `tb` được hỗ trợ:

```php
File::types(['mp3', 'wav'])
    ->min('1kb')
    ->max('10mb');
```

<a name="validating-files-image-files"></a>
#### Xác thực Tệp Hình ảnh

Nếu ứng dụng của bạn chấp nhận hình ảnh được tải lên bởi người dùng, bạn có thể sử dụng phương thức constructor `image` của quy tắc `File` để đảm bảo rằng tệp đang được xác thực là một hình ảnh (jpg, jpeg, png, bmp, gif, hoặc webp).

Ngoài ra, quy tắc `dimensions` có thể được sử dụng để giới hạn kích thước của hình ảnh:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'photo' => [
        'required',
        File::image()
            ->min(1024)
            ->max(12 * 1024)
            ->dimensions(Rule::dimensions()->maxWidth(1000)->maxHeight(500)),
    ],
]);
```

> [!NOTE]
> Thêm thông tin về việc xác thực kích thước hình ảnh có thể được tìm thấy trong [tài liệu quy tắc kích thước](#rule-dimensions).

> [!WARNING]
> Theo mặc định, quy tắc `image` không cho phép các tệp SVG do khả năng lỗ hổng XSS. Nếu bạn cần cho phép các tệp SVG, bạn có thể truyền `allowSvg: true` cho quy tắc `image`: `File::image(allowSvg: true)`.

<a name="validating-files-image-dimensions"></a>
#### Xác thực Kích thước Hình ảnh

Bạn cũng có thể xác thực kích thước của một hình ảnh. Ví dụ, để xác thực rằng một hình ảnh được tải lên có chiều rộng ít nhất 1000 pixel và chiều cao 500 pixel, bạn có thể sử dụng quy tắc `dimensions`:

```php
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

File::image()->dimensions(
    Rule::dimensions()
        ->maxWidth(1000)
        ->maxHeight(500)
)
```

> [!NOTE]
> Thêm thông tin về việc xác thực kích thước hình ảnh có thể được tìm thấy trong [tài liệu quy tắc kích thước](#rule-dimensions).

<a name="validating-passwords"></a>
## Xác thực Mật khẩu

Để đảm bảo rằng mật khẩu có mức độ phức tạp đầy đủ, bạn có thể sử dụng đối tượng quy tắc `Password` của Laravel:

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\Password;

$validator = Validator::make($request->all(), [
    'password' => ['required', 'confirmed', Password::min(8)],
]);
```

Đối tượng quy tắc `Password` cho phép bạn dễ dàng tùy chỉnh các yêu cầu phức tạp của mật khẩu cho ứng dụng của bạn, chẳng hạn như chỉ định rằng mật khẩu yêu cầu ít nhất một chữ cái, số, ký hiệu, hoặc ký tự với chữ hoa thường xen kẽ:

```php
// Yêu cầu ít nhất 8 ký tự...
Password::min(8)

// Yêu cầu ít nhất một chữ cái...
Password::min(8)->letters()

// Yêu cầu ít nhất một chữ cái hoa và một chữ cái thường...
Password::min(8)->mixedCase()

// Yêu cầu ít nhất một số...
Password::min(8)->numbers()

// Yêu cầu ít nhất một ký hiệu...
Password::min(8)->symbols()
```

Ngoài ra, bạn có thể đảm bảo rằng một mật khẩu không bị xâm phạm trong một rò rỉ dữ liệu mật khẩu công khai sử dụng phương thức `uncompromised`:

```php
Password::min(8)->uncompromised()
```

Nội bộ, đối tượng quy tắc `Password` sử dụng mô hình [k-Anonymity](https://en.wikipedia.org/wiki/K-anonymity) để xác định xem một mật khẩu có bị rò rỉ thông qua dịch vụ [haveibeenpwned.com](https://haveibeenpwned.com) hay không mà không hy sinh quyền riêng tư hoặc bảo mật của người dùng.

Theo mặc định, nếu một mật khẩu xuất hiện ít nhất một lần trong một rò rỉ dữ liệu, nó sẽ được coi là bị xâm phạm. Bạn có thể tùy chỉnh ngưỡng này sử dụng đối số đầu tiên của phương thức `uncompromised`:

```php
// Đảm bảo mật khẩu xuất hiện ít hơn 3 lần trong cùng một rò rỉ dữ liệu...
Password::min(8)->uncompromised(3);
```

Tất nhiên, bạn có thể chuỗi tất cả các phương thức trong các ví dụ trên:

```php
Password::min(8)
    ->letters()
    ->mixedCase()
    ->numbers()
    ->symbols()
    ->uncompromised()
```

Bạn có thể chuyển đổi một đối tượng quy tắc `Password` thành một chuỗi phù hợp cho thuộc tính HTML `passwordrules` sử dụng phương thức `toPasswordRulesString`:

```blade
<input
    type="password"
    name="password"
    autocomplete="new-password"
    passwordrules="{{ Password::defaults()->toPasswordRulesString() }}"
/>
```

<a name="defining-default-password-rules"></a>
#### Định nghĩa Quy tắc Mật khẩu Mặc định

Bạn có thể thấy thuận tiện khi chỉ định các quy tắc xác thực mặc định cho mật khẩu trong một vị trí duy nhất của ứng dụng của bạn. Bạn có thể dễ dàng thực hiện việc này sử dụng phương thức `Password::defaults`, chấp nhận một closure. Closure được cung cấp cho phương thức `defaults` nên trả về cấu hình mặc định của quy tắc Password. Thông thường, quy tắc `defaults` nên được gọi trong phương thức `boot` của một trong các nhà cung cấp dịch vụ của ứng dụng:

```php
use Illuminate\Validation\Rules\Password;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
            ? $rule->mixedCase()->uncompromised()
            : $rule;
    });
}
```

Sau đó, khi bạn muốn áp dụng các quy tắc mặc định cho một mật khẩu cụ thể đang được xác thực, bạn có thể gọi phương thức `defaults` không có đối số:

```php
'password' => ['required', Password::defaults()],
```

Thỉnh thoảng, bạn có thể muốn gắn thêm các quy tắc xác thực vào các quy tắc xác thực mật khẩu mặc định của mình. Bạn có thể sử dụng phương thức `rules` để thực hiện việc này:

```php
use App\Rules\ZxcvbnRule;

Password::defaults(function () {
    $rule = Password::min(8)->rules([new ZxcvbnRule]);

    // ...
});
```

<a name="custom-validation-rules"></a>
## Quy tắc Xác thực Tùy chỉnh

<a name="using-rule-objects"></a>
### Sử dụng Đối tượng Quy tắc

Laravel cung cấp nhiều quy tắc xác thực hữu ích; tuy nhiên, bạn có thể muốn chỉ định một số quy tắc của riêng mình. Một phương thức đăng ký các quy tắc xác thực tùy chỉnh là sử dụng các đối tượng quy tắc. Để tạo một đối tượng quy tắc mới, bạn có thể sử dụng lệnh Artisan `make:rule`. Hãy sử dụng lệnh này để tạo một quy tắc xác minh một chuỗi là chữ hoa. Laravel sẽ đặt quy tắc mới trong thư mục `app/Rules`. Nếu thư mục này không tồn tại, Laravel sẽ tạo nó khi bạn thực thi lệnh Artisan để tạo quy tắc của bạn:

```shell
php artisan make:rule Uppercase
```

Khi quy tắc đã được tạo, chúng ta sẵn sàng định nghĩa hành vi của nó. Một đối tượng quy tắc chứa một phương thức duy nhất: `validate`. Phương thức này nhận tên thuộc tính, giá trị của nó, và một callback nên được gọi khi thất bại với thông báo lỗi xác thực:

```php
<?php

namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements ValidationRule
{
    /**
     * Run the validation rule.
     */
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail('The :attribute must be uppercase.');
        }
    }
}
```

Khi quy tắc đã được định nghĩa, bạn có thể gắn nó vào một validator bằng cách truyền một instance của đối tượng quy tắc với các quy tắc xác thực khác của bạn:

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

#### Dịch Thông báo Xác thực

Thay vì cung cấp một thông báo lỗi theo nghĩa đen cho closure `$fail`, bạn cũng có thể cung cấp một [khóa chuỗi dịch](/docs/{{version}}/localization) và hướng dẫn Laravel dịch thông báo lỗi:

```php
if (strtoupper($value) !== $value) {
    $fail('validation.uppercase')->translate();
}
```

Nếu cần thiết, bạn có thể cung cấp các thay thế placeholder và ngôn ngữ ưu tiên làm đối số thứ nhất và thứ hai cho phương thức `translate`:

```php
$fail('validation.location')->translate([
    'value' => $this->value,
], 'fr');
```

#### Truy cập Dữ liệu Bổ sung

Nếu class quy tắc xác thực tùy chỉnh của bạn cần truy cập tất cả dữ liệu khác đang được xác thực, class quy tắc của bạn có thể triển khai interface `Illuminate\Contracts\Validation\DataAwareRule`. Interface này yêu cầu class của bạn định nghĩa một phương thức `setData`. Phương thức này sẽ được gọi tự động bởi Laravel (trước khi xác thực tiếp tục) với tất cả dữ liệu đang được xác thực:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\DataAwareRule;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements DataAwareRule, ValidationRule
{
    /**
     * All of the data under validation.
     *
     * @var array<string, mixed>
     */
    protected $data = [];

    // ...

    /**
     * Set the data under validation.
     *
     * @param  array<string, mixed>  $data
     */
    public function setData(array $data): static
    {
        $this->data = $data;

        return $this;
    }
}
```

Hoặc, nếu quy tắc xác thực của bạn yêu cầu truy cập vào instance validator đang thực hiện xác thực, bạn có thể triển khai interface `ValidatorAwareRule`:

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\ValidationRule;
use Illuminate\Contracts\Validation\ValidatorAwareRule;
use Illuminate\Validation\Validator;

class Uppercase implements ValidationRule, ValidatorAwareRule
{
    /**
     * The validator instance.
     *
     * @var \Illuminate\Validation\Validator
     */
    protected $validator;

    // ...

    /**
     * Set the current validator.
     */
    public function setValidator(Validator $validator): static
    {
        $this->validator = $validator;

        return $this;
    }
}
```

<a name="using-closures"></a>
### Sử dụng Closures

Nếu bạn chỉ cần chức năng của một quy tắc tùy chỉnh một lần trong ứng dụng của mình, bạn có thể sử dụng một closure thay vì một đối tượng quy tắc. Closure nhận tên thuộc tính, giá trị thuộc tính, và một callback `$fail` nên được gọi nếu xác thực thất bại:

```php
use Illuminate\Support\Facades\Validator;
use Closure;

$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function (string $attribute, mixed $value, Closure $fail) {
            if ($value === 'foo') {
                $fail("The {$attribute} is invalid.");
            }
        },
    ],
]);
```

<a name="implicit-rules"></a>
### Quy tắc Ngầm định

Theo mặc định, khi một thuộc tính đang được xác thực không có mặt hoặc chứa một chuỗi rỗng, các quy tắc xác thực bình thường, bao gồm các quy tắc tùy chỉnh, không được chạy. Ví dụ, quy tắc [unique](#rule-unique) sẽ không được chạy đối với một chuỗi rỗng:

```php
use Illuminate\Support\Facades\Validator;

$rules = ['name' => ['unique:users,name']];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

Để một quy tắc tùy chỉnh chạy ngay cả khi một thuộc tính trống, quy tắc phải ngụ ý rằng thuộc tính là bắt buộc. Để tạo nhanh một đối tượng quy tắc ngầm định mới, bạn có thể sử dụng lệnh Artisan `make:rule` với tùy chọn `--implicit`:

```shell
php artisan make:rule Uppercase --implicit
```

> [!WARNING]
> Một quy tắc "ngầm định" chỉ _ngụ ý_ rằng thuộc tính là bắt buộc. Việc nó thực sự làm vô hiệu một thuộc tính bị thiếu hoặc trống là tùy thuộc vào bạn.

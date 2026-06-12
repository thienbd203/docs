# Precognition

- [Introduction](#introduction)
- [Live Validation](#live-validation)
    - [Using Vue](#using-vue)
    - [Using React](#using-react)
    - [Using Alpine and Blade](#using-alpine)
    - [Configuring Axios](#configuring-axios)
- [Validating Arrays](#validating-arrays)
- [Customizing Validation Rules](#customizing-validation-rules)
- [Handling File Uploads](#handling-file-uploads)
- [Managing Side-Effects](#managing-side-effects)
- [Testing](#testing)

<a name="introduction"></a>
## Introduction

Laravel Precognition cho phép bạn dự đoán kết quả của một request HTTP trong tương lai. Một trong các trường hợp sử dụng chính của Precognition là khả năng cung cấp validation "live" cho ứng dụng JavaScript frontend của bạn mà không cần phải nhân đôi các quy tắc validation backend của ứng dụng.

Khi Laravel nhận được một "precognitive request", nó sẽ thực thi tất cả middleware của route và giải quyết các dependencies controller của route, bao gồm validating [form requests](/docs/{{version}}/validation#form-request-validation) - nhưng nó sẽ không thực sự thực thi phương thức controller của route.

> [!NOTE]
> Kể từ Inertia 2.3, hỗ trợ Precognition được tích hợp sẵn. Vui lòng tham khảo [tài liệu Inertia Forms](https://inertiajs.com/forms) để biết thêm thông tin. Các phiên bản Inertia trước đó yêu cầu Precognition 0.x.

<a name="live-validation"></a>
## Live Validation

<a name="using-vue"></a>
### Using Vue

Sử dụng Laravel Precognition, bạn có thể cung cấp các trải nghiệm validation live cho người dùng mà không cần nhân đôi các quy tắc validation trong ứng dụng Vue frontend của bạn. Để minh họa cách nó hoạt động, hãy xây dựng một form để tạo người dùng mới trong ứng dụng của chúng ta.

Trước hết, để bật Precognition cho một route, middleware `HandlePrecognitiveRequests` nên được thêm vào định nghĩa route. Bạn cũng nên tạo một [form request](/docs/{{version}}/validation#form-request-validation) để chứa các quy tắc validation của route:

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Tiếp theo, bạn nên cài đặt các helpers frontend Laravel Precognition cho Vue thông qua NPM:

```shell
npm install laravel-precognition-vue
```

Với package Laravel Precognition được cài đặt, bạn hiện có thể tạo một object form bằng cách sử dụng hàm `useForm` của Precognition, cung cấp phương thức HTTP (`post`), URL mục tiêu (`/users`), và dữ liệu form ban đầu.

Sau đó, để bật validation live, gọi phương thức `validate` của form trên sự kiện `change` của mỗi input, cung cấp tên của input:

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <label for="name">Name</label>
        <input
            id="name"
            v-model="form.name"
            @change="form.validate('name')"
        />
        <div v-if="form.invalid('name')">
            {{ form.errors.name }}
        </div>

        <label for="email">Email</label>
        <input
            id="email"
            type="email"
            v-model="form.email"
            @change="form.validate('email')"
        />
        <div v-if="form.invalid('email')">
            {{ form.errors.email }}
        </div>

        <button :disabled="form.processing">
            Create User
        </button>
    </form>
</template>
```

Bây giờ, khi form được điền bởi người dùng, Precognition sẽ cung cấp output validation live được hỗ trợ bởi các quy tắc validation trong form request của route. Khi các inputs của form thay đổi, một request validation "precognitive" được debounce sẽ được gửi đến ứng dụng Laravel của bạn. Bạn có thể cấu hình timeout debounce bằng cách gọi hàm `setValidationTimeout` của form:

```js
form.setValidationTimeout(3000);
```

Khi một request validation đang trong quá trình, thuộc tính `validating` của form sẽ là `true`:

```html
<div v-if="form.validating">
    Validating...
</div>
```

Bất kỳ lỗi validation nào được trả về trong quá trình request validation hoặc gửi form sẽ tự động điền vào object `errors` của form:

```html
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

Bạn có thể xác định xem form có bất kỳ lỗi nào bằng cách sử dụng thuộc tính `hasErrors` của form:

```html
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

Bạn cũng có thể xác định xem một input đã vượt qua hay thất bại validation bằng cách chuyển tên của input cho các hàm `valid` và `invalid` của form, tương ứng:

```html
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

> [!WARNING]
> Một input form sẽ chỉ xuất hiện là hợp lệ hoặc không hợp lệ sau khi nó đã thay đổi và một phản hồi validation đã được nhận.

Nếu bạn đang xác thực một tập hợp con của các inputs của form với Precognition, có thể hữu ích để xóa lỗi thủ công. Bạn có thể sử dụng hàm `forgetError` của form để đạt được điều này:

```html
<input
    id="avatar"
    type="file"
    @change="(e) => {
        form.avatar = e.target.files[0]

        form.forgetError('avatar')
    }"
>
```

Như chúng ta đã thấy, bạn có thể hook vào sự kiện `change` của một input và xác thực các inputs riêng lẻ khi người dùng tương tác với chúng; tuy nhiên, bạn có thể cần xác thực các inputs mà người dùng chưa tương tác. Điều này thường gặp khi xây dựng một "wizard", nơi bạn muốn xác thực tất cả các inputs hiển thị, bất kể người dùng đã tương tác với chúng hay không, trước khi chuyển sang bước tiếp theo.

Để thực hiện điều này với Precognition, bạn nên gọi phương thức `validate` chuyển tên các trường bạn muốn xác thực cho key cấu hình `only`. Bạn có thể xử lý kết quả validation với các callbacks `onSuccess` hoặc `onValidationError`:

```html
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

Tất nhiên, bạn cũng có thể thực thi mã để phản ứng với phản hồi gửi form. Hàm `submit` của form trả về một promise request Axios. Điều này cung cấp một cách thuận tiện để truy xuất payload phản hồi, reset các inputs của form khi gửi thành công, hoặc xử lý một request thất bại:

```js
const submit = () => form.submit()
    .then(response => {
        form.reset();

        alert('User created.');
    })
    .catch(error => {
        alert('An error occurred.');
    });
```

Bạn có thể xác định xem một request gửi form đang trong quá trình bằng cách kiểm tra thuộc tính `processing` của form:

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="using-react"></a>
### Using React

Sử dụng Laravel Precognition, bạn có thể cung cấp các trải nghiệm validation live cho người dùng mà không cần nhân đôi các quy tắc validation trong ứng dụng React frontend của bạn. Để minh họa cách nó hoạt động, hãy xây dựng một form để tạo người dùng mới trong ứng dụng của chúng ta.

Trước hết, để bật Precognition cho một route, middleware `HandlePrecognitiveRequests` nên được thêm vào định nghĩa route. Bạn cũng nên tạo một [form request](/docs/{{version}}/validation#form-request-validation) để chứa các quy tắc validation của route:

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Tiếp theo, bạn nên cài đặt các helpers frontend Laravel Precognition cho React thông qua NPM:

```shell
npm install laravel-precognition-react
```

Với package Laravel Precognition được cài đặt, bạn hiện có thể tạo một object form bằng cách sử dụng hàm `useForm` của Precognition, cung cấp phương thức HTTP (`post`), URL mục tiêu (`/users`), và dữ liệu form ban đầu.

Để bật validation live, bạn nên lắng nghe sự kiện `change` và `blur` của mỗi input. Trong trình xử lý sự kiện `change`, bạn nên đặt dữ liệu của form với hàm `setData`, chuyển tên của input và giá trị mới. Sau đó, trong trình xử lý sự kiện `blur`, gọi phương thức `validate` của form, cung cấp tên của input:

```jsx
import { useForm } from 'laravel-precognition-react';

export default function Form() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    const submit = (e) => {
        e.preventDefault();

        form.submit();
    };

    return (
        <form onSubmit={submit}>
            <label htmlFor="name">Name</label>
            <input
                id="name"
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label htmlFor="email">Email</label>
            <input
                id="email"
                value={form.data.email}
                onChange={(e) => form.setData('email', e.target.value)}
                onBlur={() => form.validate('email')}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button disabled={form.processing}>
                Create User
            </button>
        </form>
    );
};
```

Bây giờ, khi form được điền bởi người dùng, Precognition sẽ cung cấp output validation live được hỗ trợ bởi các quy tắc validation trong form request của route. Khi các inputs của form thay đổi, một request validation "precognitive" được debounce sẽ được gửi đến ứng dụng Laravel của bạn. Bạn có thể cấu hình timeout debounce bằng cách gọi hàm `setValidationTimeout` của form:

```js
form.setValidationTimeout(3000);
```

Khi một request validation đang trong quá trình, thuộc tính `validating` của form sẽ là `true`:

```jsx
{form.validating && <div>Validating...</div>}
```

Bất kỳ lỗi validation nào được trả về trong quá trình request validation hoặc gửi form sẽ tự động điền vào object `errors` của form:

```jsx
{form.invalid('email') && <div>{form.errors.email}</div>}
```

Bạn có thể xác định xem form có bất kỳ lỗi nào bằng cách sử dụng thuộc tính `hasErrors` của form:

```jsx
{form.hasErrors && <div><!-- ... --></div>}
```

Bạn cũng có thể xác định xem một input đã vượt qua hay thất bại validation bằng cách chuyển tên của input cho các hàm `valid` và `invalid` của form, tương ứng:

```jsx
{form.valid('email') && <span>✅</span>}

{form.invalid('email') && <span>❌</span>}
```

> [!WARNING]
> Một input form sẽ chỉ xuất hiện là hợp lệ hoặc không hợp lệ sau khi nó đã thay đổi và một phản hồi validation đã được nhận.

Nếu bạn đang xác thực một tập hợp con của các inputs của form với Precognition, có thể hữu ích để xóa lỗi thủ công. Bạn có thể sử dụng hàm `forgetError` của form để đạt được điều này:

```jsx
<input
    id="avatar"
    type="file"
    onChange={(e) => {
        form.setData('avatar', e.target.files[0]);

        form.forgetError('avatar');
    }}
>
```

Như chúng ta đã thấy, bạn có thể hook vào sự kiện `blur` của một input và xác thực các inputs riêng lẻ khi người dùng tương tác với chúng; tuy nhiên, bạn có thể cần xác thực các inputs mà người dùng chưa tương tác. Điều này thường gặp khi xây dựng một "wizard", nơi bạn muốn xác thực tất cả các inputs hiển thị, bất kể người dùng đã tương tác với chúng hay không, trước khi chuyển sang bước tiếp theo.

Để thực hiện điều này với Precognition, bạn nên gọi phương thức `validate` chuyển tên các trường bạn muốn xác thực cho key cấu hình `only`. Bạn có thể xử lý kết quả validation với các callbacks `onSuccess` hoặc `onValidationError`:

```jsx
<button
    type="button"
    onClick={() => form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })}
>Next Step</button>
```

Tất nhiên, bạn cũng có thể thực thi mã để phản ứng với phản hồi gửi form. Hàm `submit` của form trả về một promise request Axios. Điều này cung cấp một cách thuận tiện để truy xuất payload phản hồi, reset các inputs của form khi gửi form thành công, hoặc xử lý một request thất bại:

```js
const submit = (e) => {
    e.preventDefault();

    form.submit()
        .then(response => {
            form.reset();

            alert('User created.');
        })
        .catch(error => {
            alert('An error occurred.');
        });
};
```

Bạn có thể xác định xem một request gửi form đang trong quá trình bằng cách kiểm tra thuộc tính `processing` của form:

```html
<button disabled={form.processing}>
    Submit
</button>
```

<a name="using-alpine"></a>
### Using Alpine and Blade

Sử dụng Laravel Precognition, bạn có thể cung cấp các trải nghiệm validation live cho người dùng mà không cần nhân đôi các quy tắc validation trong ứng dụng Alpine frontend của bạn. Để minh họa cách nó hoạt động, hãy xây dựng một form để tạo người dùng mới trong ứng dụng của chúng ta.

Trước hết, để bật Precognition cho một route, middleware `HandlePrecognitiveRequests` nên được thêm vào định nghĩa route. Bạn cũng nên tạo một [form request](/docs/{{version}}/validation#form-request-validation) để chứa các quy tắc validation của route:

```php
use App\Http\Requests\CreateUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (CreateUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Tiếp theo, bạn nên cài đặt các helpers frontend Laravel Precognition cho Alpine thông qua NPM:

```shell
npm install laravel-precognition-alpine
```

Sau đó, đăng ký plugin Precognition với Alpine trong file `resources/js/app.js` của bạn:

```js
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

window.Alpine = Alpine;

Alpine.plugin(Precognition);
Alpine.start();
```

Với package Laravel Precognition được cài đặt và đăng ký, bạn hiện có thể tạo một object form bằng cách sử dụng "magic" `$form` của Precognition, cung cấp phương thức HTTP (`post`), URL mục tiêu (`/users`), và dữ liệu form ban đầu.

Để bật validation live, bạn nên bind dữ liệu của form với input liên quan của nó và sau đó lắng nghe sự kiện `change` của mỗi input. Trong trình xử lý sự kiện `change`, bạn nên gọi phương thức `validate` của form, cung cấp tên của input:

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    @csrf
    <label for="name">Name</label>
    <input
        id="name"
        name="name"
        x-model="form.name"
        @change="form.validate('name')"
    />
    <template x-if="form.invalid('name')">
        <div x-text="form.errors.name"></div>
    </template>

    <label for="email">Email</label>
    <input
        id="email"
        name="email"
        x-model="form.email"
        @change="form.validate('email')"
    />
    <template x-if="form.invalid('email')">
        <div x-text="form.errors.email"></div>
    </template>

    <button :disabled="form.processing">
        Create User
    </button>
</form>
```

Bây giờ, khi form được điền bởi người dùng, Precognition sẽ cung cấp output validation live được hỗ trợ bởi các quy tắc validation trong form request của route. Khi các inputs của form thay đổi, một request validation "precognitive" được debounce sẽ được gửi đến ứng dụng Laravel của bạn. Bạn có thể cấu hình timeout debounce bằng cách gọi hàm `setValidationTimeout` của form:

```js
form.setValidationTimeout(3000);
```

Khi một request validation đang trong quá trình, thuộc tính `validating` của form sẽ là `true`:

```html
<template x-if="form.validating">
    <div>Validating...</div>
</template>
```

Bất kỳ lỗi validation nào được trả về trong quá trình request validation hoặc gửi form sẽ tự động điền vào object `errors` của form:

```html
<template x-if="form.invalid('email')">
    <div x-text="form.errors.email"></div>
</template>
```

Bạn có thể xác định xem form có bất kỳ lỗi nào bằng cách sử dụng thuộc tính `hasErrors` của form:

```html
<template x-if="form.hasErrors">
    <div><!-- ... --></div>
</template>
```

Bạn cũng có thể xác định xem một input đã vượt qua hay thất bại validation bằng cách chuyển tên của input cho các hàm `valid` và `invalid` của form, tương ứng:

```html
<template x-if="form.valid('email')">
    <span>✅</span>
</template>

<template x-if="form.invalid('email')">
    <span>❌</span>
</template>
```

> [!WARNING]
> Một input form sẽ chỉ xuất hiện là hợp lệ hoặc không hợp lệ sau khi nó đã thay đổi và một phản hồi validation đã được nhận.

Như chúng ta đã thấy, bạn có thể hook vào sự kiện `change` của một input và xác thực các inputs riêng lẻ khi người dùng tương tác với chúng; tuy nhiên, bạn có thể cần xác thực các inputs mà người dùng chưa tương tác. Điều này thường gặp khi xây dựng một "wizard", nơi bạn muốn xác thực tất cả các inputs hiển thị, bất kể người dùng đã tương tác với chúng hay không, trước khi chuyển sang bước tiếp theo.

Để thực hiện điều này với Precognition, bạn nên gọi phương thức `validate` chuyển tên các trường bạn muốn xác thực cho key cấu hình `only`. Bạn có thể xử lý kết quả validation với các callbacks `onSuccess` hoặc `onValidationError`:

```html
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

Bạn có thể xác định xem một request gửi form đang trong quá trình bằng cách kiểm tra thuộc tính `processing` của form:

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="repopulating-old-form-data"></a>
#### Repopulating Old Form Data

Trong ví dụ tạo người dùng được thảo luận ở trên, chúng ta đang sử dụng Precognition để thực hiện validation live; tuy nhiên, chúng ta đang thực hiện một gửi form phía server truyền thống để gửi form. Vì vậy, form nên được điền với bất kỳ input "old" và lỗi validation nào được trả về từ gửi form phía server:

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

Ngoài ra, nếu bạn muốn gửi form thông qua XHR bạn có thể sử dụng hàm `submit` của form, trả về một promise request Axios:

```html
<form
    x-data="{
        form: $form('post', '/register', {
            name: '',
            email: '',
        }),
        submit() {
            this.form.submit()
                .then(response => {
                    this.form.reset();

                    alert('User created.')
                })
                .catch(error => {
                    alert('An error occurred.');
                });
        },
    }"
    @submit.prevent="submit"
>
```

<a name="configuring-axios"></a>
### Configuring Axios

Các thư viện validation Precognition sử dụng [Axios](https://github.com/axios/axios) HTTP client để gửi các requests đến backend ứng dụng của bạn. Để thuận tiện, instance Axios có thể được tùy chỉnh nếu được yêu cầu bởi ứng dụng của bạn. Ví dụ, khi sử dụng thư viện `laravel-precognition-vue`, bạn có thể thêm các headers request bổ sung cho mỗi request đi ra trong file `resources/js/app.js` của ứng dụng:

```js
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

Hoặc, nếu bạn đã có một instance Axios được cấu hình cho ứng dụng của bạn, bạn có thể nói với Precognition để sử dụng instance đó thay thế:

```js
import Axios from 'axios';
import { client } from 'laravel-precognition-vue';

window.axios = Axios.create()
window.axios.defaults.headers.common['Authorization'] = authToken;

client.use(window.axios)
```

<a name="validating-arrays"></a>
## Validating Arrays

Bạn có thể sử dụng các ký tự đại diện để xác thực các trường trong arrays hoặc các objects lồng nhau. Mỗi `*` khớp một đoạn đường dẫn duy nhất:

```js
// Validate email for all users in an array...
form.validate('users.*.email');

// Validate all fields in a profile object...
form.validate('profile.*');

// Validate all fields for all users...
form.validate('users.*.*');
```

<a name="customizing-validation-rules"></a>
## Customizing Validation Rules

Có thể tùy chỉnh các quy tắc validation được thực thi trong quá trình một request precognitive bằng cách sử dụng phương thức `isPrecognitive` của request.

Ví dụ, trên một form tạo người dùng, chúng ta có thể muốn xác thực rằng một mật khẩu là "uncompromised" chỉ trên lần gửi form cuối cùng. Đối với các requests validation precognitive, chúng ta sẽ chỉ xác thực rằng mật khẩu là bắt buộc và có tối thiểu 8 ký tự. Sử dụng phương thức `isPrecognitive`, chúng ta có thể tùy chỉnh các quy tắc được định nghĩa bởi form request của chúng ta:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    protected function rules()
    {
        return [
            'password' => [
                'required',
                $this->isPrecognitive()
                    ? Password::min(8)
                    : Password::min(8)->uncompromised(),
            ],
            // ...
        ];
    }
}
```

<a name="handling-file-uploads"></a>
## Handling File Uploads

Theo mặc định, Laravel Precognition không tải lên hoặc xác thực các files trong quá trình request validation precognitive. Điều này đảm bảo rằng các files lớn không được tải lên không cần thiết nhiều lần.

Do hành vi này, bạn nên đảm bảo rằng ứng dụng của bạn [tùy chỉnh các quy tắc validation của form request tương ứng](#customizing-validation-rules) để chỉ định trường chỉ được yêu cầu cho các lần gửi form đầy đủ:

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png',
            'dimensions:ratio=3/2',
        ],
        // ...
    ];
}
```

Nếu bạn muốn bao gồm các files trong mỗi request validation, bạn có thể gọi hàm `validateFiles` trên instance form phía client của bạn:

```js
form.validateFiles();
```

<a name="managing-side-effects"></a>
## Managing Side-Effects

Khi thêm middleware `HandlePrecognitiveRequests` vào một route, bạn nên xem xét xem có bất kỳ side-effects nào trong middleware _khác_ nên được bỏ qua trong quá trình request precognitive hay không.

Ví dụ, bạn có thể có một middleware tăng tổng số "interactions" mỗi người dùng có với ứng dụng của bạn, nhưng bạn có thể không muốn các requests precognitive được tính là một interaction. Để thực hiện điều này, chúng ta có thể kiểm tra phương thức `isPrecognitive` của request trước khi tăng số lượng interaction:

```php
<?php

namespace App\Http\Middleware;

use App\Facades\Interaction;
use Closure;
use Illuminate\Http\Request;

class InteractionMiddleware
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): mixed
    {
        if (! $request->isPrecognitive()) {
            Interaction::incrementFor($request->user());
        }

        return $next($request);
    }
}
```

<a name="testing"></a>
## Testing

Nếu bạn muốn thực hiện các requests precognitive trong các tests của bạn, `TestCase` của Laravel bao gồm một helper `withPrecognition` sẽ thêm header request `Precognition`.

Ngoài ra, nếu bạn muốn assert rằng một request precognitive thành công, ví dụ, không trả về bất kỳ lỗi validation nào, bạn có thể sử dụng phương thức `assertSuccessfulPrecognition` trên phản hồi:

```php tab=Pest
it('validates registration form with precognition', function () {
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();

    expect(User::count())->toBe(0);
});
```

```php tab=PHPUnit
public function test_it_validates_registration_form_with_precognition()
{
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();
    $this->assertSame(0, User::count());
}
```

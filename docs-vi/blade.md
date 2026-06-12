# Blade Templates

- [Giới thiệu](#introduction)
    - [Tăng cường Blade với Livewire](#supercharging-blade-with-livewire)
- [Hiển thị Dữ liệu](#displaying-data)
    - [Mã hóa HTML Entity](#html-entity-encoding)
    - [Blade và các Framework JavaScript](#blade-and-javascript-frameworks)
- [Chỉ thị Blade](#blade-directives)
    - [Câu lệnh If](#if-statements)
    - [Câu lệnh Switch](#switch-statements)
    - [Vòng lặp](#loops)
    - [Biến Vòng lặp](#the-loop-variable)
    - [Lớp Điều kiện](#conditional-classes)
    - [Thuộc tính Bổ sung](#additional-attributes)
    - [Bao gồm Subviews](#including-subviews)
    - [Chỉ thị `@once`](#the-once-directive)
    - [PHP Thô](#raw-php)
    - [Phông chữ](#fonts)
    - [Chú thích](#comments)
- [Components](#components)
    - [Render Components](#rendering-components)
    - [Index Components](#index-components)
    - [Truyền Dữ liệu cho Components](#passing-data-to-components)
    - [Thuộc tính Component](#component-attributes)
    - [Từ khóa Dành riêng](#reserved-keywords)
    - [Slots](#slots)
    - [Inline Component Views](#inline-component-views)
    - [Dynamic Components](#dynamic-components)
    - [Đăng ký Components Thủ công](#manually-registering-components)
- [Anonymous Components](#anonymous-components)
    - [Anonymous Index Components](#anonymous-index-components)
    - [Thuộc tính Dữ liệu / Thuộc tính](#data-properties-attributes)
    - [Truy cập Dữ liệu Cha](#accessing-parent-data)
    - [Đường dẫn Anonymous Components](#anonymous-component-paths)
- [Xây dựng Layouts](#building-layouts)
    - [Layouts Sử dụng Components](#layouts-using-components)
    - [Layouts Sử dụng Kế thừa Template](#layouts-using-template-inheritance)
- [Forms](#forms)
    - [Trường CSRF](#csrf-field)
    - [Trường Method](#method-field)
    - [Lỗi Validation](#validation-errors)
- [Stacks](#stacks)
- [Service Injection](#service-injection)
- [Render Blade Templates Inline](#rendering-inline-blade-templates)
- [Render Blade Fragments](#rendering-blade-fragments)
- [Mở rộng Blade](#extending-blade)
    - [Custom Echo Handlers](#custom-echo-handlers)
    - [Custom If Statements](#custom-if-statements)

<a name="introduction"></a>
## Giới thiệu

Blade là công cụ tạo mẫu (templating engine) đơn giản nhưng mạnh mẽ được tích hợp sẵn với Laravel. Khác với một số công cụ tạo mẫu PHP khác, Blade không hạn chế bạn sử dụng mã PHP thuần trong các mẫu của mình. Trên thực tế, tất cả các mẫu Blade đều được biên dịch thành mã PHP thuần và được cache cho đến khi chúng được sửa đổi, có nghĩa là Blade thêm về cơ bản không có chi phí nào cho ứng dụng của bạn. Các file mẫu Blade sử dụng đuôi file `.blade.php` và thường được lưu trữ trong thư mục `resources/views`.

Các view Blade có thể được trả về từ routes hoặc controllers bằng cách sử dụng helper `view` toàn cục. Tất nhiên, như đã đề cập trong tài liệu về [views](/docs/{{version}}/views), dữ liệu có thể được truyền cho view Blade bằng cách sử dụng đối số thứ hai của helper `view`:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

<a name="supercharging-blade-with-livewire"></a>
### Tăng cường Blade với Livewire

Bạn muốn đưa các mẫu Blade của mình lên cấp độ tiếp theo và xây dựng các giao diện động một cách dễ dàng? Hãy xem [Laravel Livewire](https://livewire.laravel.com). Livewire cho phép bạn viết các component Blade được tăng cường với chức năng động mà trước đây thường chỉ có thể thực hiện thông qua các framework frontend như React, Svelte, hoặc Vue, cung cấp một cách tiếp cận tuyệt vời để xây dựng các frontend hiện đại, phản hồi mà không có sự phức tạp, rendering phía client, hoặc các bước build của nhiều framework JavaScript.

<a name="displaying-data"></a>
## Hiển thị Dữ liệu

Bạn có thể hiển thị dữ liệu được truyền cho các view Blade của mình bằng cách bao bọc biến trong dấu ngoặc nhọn. Ví dụ, với route sau:

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

Bạn có thể hiển thị nội dung của biến `name` như sau:

```blade
Hello, {{ $name }}.
```

> [!NOTE]
> Các câu lệnh echo `{{ }}` của Blade tự động được gửi qua hàm `htmlspecialchars` của PHP để ngăn chặn các cuộc tấn công XSS.

Bạn không bị giới hạn trong việc hiển thị nội dung của các biến được truyền cho view. Bạn cũng có thể echo kết quả của bất kỳ hàm PHP nào. Trên thực tế, bạn có thể đặt bất kỳ mã PHP nào bạn muốn bên trong một câu lệnh echo Blade:

```blade
The current UNIX timestamp is {{ time() }}.
```

<a name="html-entity-encoding"></a>
### Mã hóa HTML Entity

Theo mặc định, Blade (và hàm `e` của Laravel) sẽ mã hóa kép các HTML entity. Nếu bạn muốn tắt mã hóa kép, hãy gọi phương thức `Blade::withoutDoubleEncoding` từ phương thức `boot` của `AppServiceProvider` của bạn:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::withoutDoubleEncoding();
    }
}
```

<a name="displaying-unescaped-data"></a>
#### Hiển thị Dữ liệu Không được Escape

Theo mặc định, các câu lệnh `{{ }}` của Blade tự động được gửi qua hàm `htmlspecialchars` của PHP để ngăn chặn các cuộc tấn công XSS. Nếu bạn không muốn dữ liệu của mình được escape, bạn có thể sử dụng cú pháp sau:

```blade
Hello, {!! $name !!}.
```

> [!WARNING]
> Hãy rất cẩn thận khi echo nội dung được cung cấp bởi người dùng của ứng dụng của bạn. Bạn thường nên sử dụng cú pháp dấu ngoặc nhọn kép đã escape để ngăn chặn các cuộc tấn công XSS khi hiển thị dữ liệu do người dùng cung cấp.

<a name="blade-and-javascript-frameworks"></a>
### Blade và các Framework JavaScript

Vì nhiều framework JavaScript cũng sử dụng dấu ngoặc nhọn "curly" để chỉ ra một biểu thức nhất định nên nên được hiển thị trong trình duyệt, bạn có thể sử dụng ký hiệu `@` để thông báo cho công cụ render Blade rằng một biểu thức nên giữ nguyên không thay đổi. Ví dụ:

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

Trong ví dụ này, ký hiệu `@` sẽ bị Blade xóa; tuy nhiên, biểu thức `{{ name }}` sẽ giữ nguyên không bị Blade thay đổi, cho phép nó được render bởi framework JavaScript của bạn.

Ký hiệu `@` cũng có thể được sử dụng để escape các chỉ thị Blade:

```blade
{{-- Blade template --}}
@@if()

<!-- HTML output -->
@if()
```

<a name="rendering-json"></a>
#### Render JSON

Đôi khi bạn có thể truyền một mảng cho view của mình với ý định render nó dưới dạng JSON để khởi tạo một biến JavaScript. Ví dụ:

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

Tuy nhiên, thay vì gọi `json_encode` thủ công, bạn có thể sử dụng phương thức `Illuminate\Support\Js::from`. Phương thức `from` chấp nhận các đối số giống như hàm `json_encode` của PHP; tuy nhiên, nó sẽ đảm bảo rằng JSON kết quả đã được escape đúng cách để bao gồm trong các trích dẫn HTML. Phương thức `from` sẽ trả về một câu lệnh JavaScript `JSON.parse` dạng chuỗi sẽ chuyển đổi đối tượng hoặc mảng đã cho thành một đối tượng JavaScript hợp lệ:

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

Các phiên bản mới nhất của khung ứng dụng Laravel bao gồm một facade `Js`, cung cấp quyền truy cập thuận tiện vào chức năng này trong các mẫu Blade của bạn:

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> [!WARNING]
> Bạn chỉ nên sử dụng phương thức `Js::from` để render các biến hiện có dưới dạng JSON. Việc tạo mẫu Blade dựa trên các biểu thức chính quy và việc cố gắng truyền một biểu thức phức tạp cho chỉ thị có thể gây ra các lỗi không mong muốn.

<a name="the-at-verbatim-directive"></a>
#### Chỉ thị `@verbatim`

Nếu bạn đang hiển thị các biến JavaScript trong một phần lớn của mẫu, bạn có thể bao bọc HTML trong chỉ thị `@verbatim` để bạn không cần phải thêm tiền tố ký hiệu `@` cho mỗi câu lệnh echo Blade:

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

<a name="blade-directives"></a>
## Chỉ thị Blade

Ngoài việc kế thừa mẫu và hiển thị dữ liệu, Blade cũng cung cấp các phím tắt thuận tiện cho các cấu trúc điều khiển PHP phổ biến, chẳng hạn như câu lệnh điều kiện và vòng lặp. Các phím tắt này cung cấp một cách rất sạch sẽ, ngắn gọn để làm việc với các cấu trúc điều khiển PHP trong khi vẫn giữ quen thuộc với các đối tác PHP của chúng.

<a name="if-statements"></a>
### Câu lệnh If

Bạn có thể xây dựng các câu lệnh `if` bằng cách sử dụng các chỉ thị `@if`, `@elseif`, `@else`, và `@endif`. Các chỉ thị này hoạt động giống hệt với các đối tác PHP của chúng:

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

Để thuận tiện, Blade cũng cung cấp một chỉ thị `@unless`:

```blade
@unless (Auth::check())
    You are not signed in.
@endunless
```

Ngoài các chỉ thị điều kiện đã thảo luận, các chỉ thị `@isset` và `@empty` có thể được sử dụng như các phím tắt thuận tiện cho các hàm PHP tương ứng của chúng:

```blade
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

<a name="authentication-directives"></a>
#### Chỉ thị Xác thực

Các chỉ thị `@auth` và `@guest` có thể được sử dụng để xác định nhanh xem người dùng hiện tại có được [xác thực](/docs/{{version}}/authentication) hay không hoặc là khách:

```blade
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

Nếu cần, bạn có thể chỉ định guard xác thực nên được kiểm tra khi sử dụng các chỉ thị `@auth` và `@guest`:

```blade
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

<a name="environment-directives"></a>
#### Chỉ thị Môi trường

Bạn có thể kiểm tra xem ứng dụng có đang chạy trong môi trường production hay không bằng cách sử dụng chỉ thị `@production`:

```blade
@production
    // Production specific content...
@endproduction
```

Hoặc, bạn có thể xác định xem ứng dụng có đang chạy trong một môi trường cụ thể hay không bằng cách sử dụng chỉ thị `@env`:

```blade
@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
```

<a name="section-directives"></a>
#### Chỉ thị Section

Bạn có thể xác định xem một section kế thừa mẫu có nội dung hay không bằng cách sử dụng chỉ thị `@hasSection`:

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

Bạn có thể sử dụng chỉ thị `sectionMissing` để xác định xem một section không có nội dung:

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

<a name="session-directives"></a>
#### Chỉ thị Session

Chỉ thị `@session` có thể được sử dụng để xác định xem một giá trị [session](/docs/{{version}}/session) có tồn tại hay không. Nếu giá trị session tồn tại, nội dung mẫu trong các chỉ thị `@session` và `@endsession` sẽ được đánh giá. Trong nội dung của chỉ thị `@session`, bạn có thể echo biến `$value` để hiển thị giá trị session:

```blade
@session('status')
    <div class="p-4 bg-green-100">
        {{ $value }}
    </div>
@endsession
```

<a name="context-directives"></a>
#### Chỉ thị Context

Chỉ thị `@context` có thể được sử dụng để xác định xem một giá trị [context](/docs/{{version}}/context) có tồn tại hay không. Nếu giá trị context tồn tại, nội dung mẫu trong các chỉ thị `@context` và `@endcontext` sẽ được đánh giá. Trong nội dung của chỉ thị `@context`, bạn có thể echo biến `$value` để hiển thị giá trị context:

```blade
@context('canonical')
    <link href="{{ $value }}" rel="canonical">
@endcontext
```

<a name="switch-statements"></a>
### Câu lệnh Switch

Các câu lệnh switch có thể được xây dựng bằng cách sử dụng các chỉ thị `@switch`, `@case`, `@break`, `@default` và `@endswitch`:

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

<a name="loops"></a>
### Vòng lặp

Ngoài các câu lệnh điều kiện, Blade cung cấp các chỉ thị đơn giản để làm việc với các cấu trúc vòng lặp của PHP. Một lần nữa, mỗi chỉ thị này hoạt động giống hệt với các đối tác PHP của chúng:

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

> [!NOTE]
> Khi lặp qua một vòng lặp `foreach`, bạn có thể sử dụng [biến vòng lặp](#the-loop-variable) để thu thập thông tin có giá trị về vòng lặp, chẳng hạn như xem bạn có đang ở lần lặp đầu tiên hay cuối cùng hay không.

Khi sử dụng vòng lặp, bạn cũng có thể bỏ qua lần lặp hiện tại hoặc kết thúc vòng lặp bằng cách sử dụng các chỉ thị `@continue` và `@break`:

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

Bạn cũng có thể bao gồm điều kiện tiếp tục hoặc ngắt trong khai báo chỉ thị:

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

<a name="the-loop-variable"></a>
### Biến Vòng lặp

Khi lặp qua một vòng lặp `foreach`, một biến `$loop` sẽ có sẵn bên trong vòng lặp của bạn. Biến này cung cấp quyền truy cập vào một số thông tin hữu ích như chỉ mục vòng lặp hiện tại và xem đây có phải là lần lặp đầu tiên hay cuối cùng hay không:

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

Nếu bạn đang ở trong một vòng lặp lồng nhau, bạn có thể truy cập biến `$loop` của vòng lặp cha thông qua thuộc tính `parent`:

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

Biến `$loop` cũng chứa nhiều thuộc tính hữu ích khác:

<div class="overflow-auto">

|| Thuộc tính           | Mô tả                                            |
|| ------------------ | ------------------------------------------------------ |
|| `$loop->index`     | Chỉ mục của lần lặp vòng lặp hiện tại (bắt đầu từ 0). |
|| `$loop->iteration` | Lần lặp vòng lặp hiện tại (bắt đầu từ 1).              |
|| `$loop->remaining` | Các lần lặp còn lại trong vòng lặp.                  |
|| `$loop->count`     | Tổng số mục trong mảng đang được lặp.                 |
|| `$loop->first`     | Liệu đây có phải là lần lặp đầu tiên qua vòng lặp hay không.  |
|| `$loop->last`      | Liệu đây có phải là lần lặp cuối cùng qua vòng lặp hay không.   |
|| `$loop->even`      | Liệu đây có phải là lần lặp chẵn qua vòng lặp hay không.    |
|| `$loop->odd`       | Liệu đây có phải là lần lặp lẻ qua vòng lặp hay không.     |
|| `$loop->depth`     | Cấp độ lồng nhau của vòng lặp hiện tại.                 |
|| `$loop->parent`    | Khi ở trong vòng lặp lồng nhau, biến vòng lặp của cha.     |

</div>

<a name="conditional-classes"></a>
### Lớp Điều kiện & Styles

Chỉ thị `@class` biên dịch có điều kiện một chuỗi lớp CSS. Chỉ thị chấp nhận một mảng các lớp trong đó khóa mảng chứa lớp hoặc các lớp bạn muốn thêm, trong khi giá trị là một biểu thức boolean. Nếu phần tử mảng có khóa số, nó sẽ luôn được bao gồm trong danh sách lớp được render:

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

Tương tự, chỉ thị `@style` có thể được sử dụng để thêm có điều kiện các kiểu CSS inline vào một phần tử HTML:

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

<a name="additional-attributes"></a>
### Thuộc tính Bổ sung

Để thuận tiện, bạn có thể sử dụng chỉ thị `@checked` để dễ dàng chỉ ra xem một ô đầu vào checkbox HTML nhất định có được "checked" hay không. Chỉ thị này sẽ echo `checked` nếu điều kiện được cung cấp đánh giá là `true`:

```blade
<input
    type="checkbox"
    name="active"
    value="active"
    @checked(old('active', $user->active))
/>
```

Tương tự, chỉ thị `@selected` có thể được sử dụng để chỉ ra xem một tùy chọn select nhất định có nên được "selected" hay không:

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

Ngoài ra, chỉ thị `@disabled` có thể được sử dụng để chỉ ra xem một phần tử nhất định có nên được "disabled" hay không:

```blade
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

Hơn nữa, chỉ thị `@readonly` có thể được sử dụng để chỉ ra xem một phần tử nhất định có nên được "readonly" hay không:

```blade
<input
    type="email"
    name="email"
    value="email@laravel.com"
    @readonly($user->isNotAdmin())
/>
```

Ngoài ra, chỉ thị `@required` có thể được sử dụng để chỉ ra xem một phần tử nhất định có nên được "required" hay không:

```blade
<input
    type="text"
    name="title"
    value="title"
    @required($user->isAdmin())
/>
```

<a name="including-subviews"></a>
### Bao gồm Subviews

> [!NOTE]
> Mặc dù bạn có thể tự do sử dụng chỉ thị `@include`, [components](#components) của Blade cung cấp chức năng tương tự và mang lại một số lợi ích so với chỉ thị `@include` chẳng hạn như liên kết dữ liệu và thuộc tính.

Chỉ thị `@include` của Blade cho phép bạn bao gồm một view Blade từ trong một view khác. Tất cả các biến có sẵn cho view cha sẽ được cung cấp cho view được bao gồm:

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Mặc dù view được bao gồm sẽ kế thừa tất cả dữ liệu có sẵn trong view cha, bạn cũng có thể truyền một mảng dữ liệu bổ sung nên được cung cấp cho view được bao gồm:

```blade
@include('view.name', ['status' => 'complete'])
```

Nếu bạn cố gắng `@include` một view không tồn tại, Laravel sẽ ném ra một lỗi. Nếu bạn muốn bao gồm một view có thể có hoặc không, bạn nên sử dụng chỉ thị `@includeIf`:

```blade
@includeIf('view.name', ['status' => 'complete'])
```

Nếu bạn muốn `@include` một view nếu một biểu thức boolean nhất định đánh giá là `true` hoặc `false`, bạn có thể sử dụng các chỉ thị `@includeWhen` và `@includeUnless`:

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

Để bao gồm view đầu tiên tồn tại từ một mảng các view nhất định, bạn có thể sử dụng chỉ thị `includeFirst`:

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

Nếu bạn muốn bao gồm một view mà không kế thừa bất kỳ biến nào từ view cha, bạn có thể sử dụng chỉ thị `@includeIsolated`. View được bao gồm sẽ chỉ có quyền truy cập vào các biến bạn truyền rõ ràng:

```blade
@includeIsolated('view.name', ['user' => $user])
```
> [!WARNING]
> Bạn nên tránh sử dụng các hằng số `__DIR__` và `__FILE__` trong các view Blade của mình, vì chúng sẽ tham chiếu đến vị trí của view đã được cache và biên dịch.

<a name="rendering-views-for-collections"></a>
#### Rendering Views for Collections

Bạn có thể kết hợp vòng lặp và includes thành một dòng với directive `@each` của Blade:

```blade
@each('view.name', $jobs, 'job')
```

Đối số đầu tiên của directive `@each` là view để render cho từng phần tử trong mảng hoặc collection. Đối số thứ hai là mảng hoặc collection mà bạn muốn lặp qua, trong khi đối số thứ ba là tên biến sẽ được gán cho lần lặp hiện tại trong view. Vì vậy, ví dụ, nếu bạn đang lặp qua một mảng `jobs`, thường thì bạn sẽ muốn truy cập từng job dưới dạng biến `job` trong view. Khóa mảng cho lần lặp hiện tại sẽ có sẵn dưới dạng biến `key` trong view.

Bạn cũng có thể truyền đối số thứ tư cho directive `@each`. Đối số này xác định view sẽ được render nếu mảng đã cho trống.

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> [!WARNING]
> Các view được render thông qua `@each` không kế thừa các biến từ view cha. Nếu view con yêu cầu các biến này, bạn nên sử dụng các directive `@foreach` và `@include` thay thế.

<a name="the-once-directive"></a>
### The `@once` Directive

Directive `@once` cho phép bạn định nghĩa một phần của template sẽ chỉ được đánh giá một lần cho mỗi chu kỳ render. Điều này có thể hữu ích để đẩy một đoạn JavaScript nhất định vào header của trang bằng cách sử dụng [stacks](#stacks). Ví dụ, nếu bạn đang render một [component](#components) nhất định trong một vòng lặp, bạn có thể chỉ muốn đẩy JavaScript vào header lần đầu tiên component được render:

```blade
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

Vì directive `@once` thường được sử dụng cùng với các directive `@push` hoặc `@prepend`, các directive `@pushOnce` và `@prependOnce` có sẵn để thuận tiện cho bạn:

```blade
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

Nếu bạn đang đẩy nội dung trùng lặp từ hai template Blade riêng biệt, bạn nên cung cấp một định danh duy nhất làm đối số thứ hai cho directive `@pushOnce` để đảm bảo nội dung chỉ được render một lần:

```blade
<!-- pie-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce

<!-- line-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce
```

<a name="raw-php"></a>
### Raw PHP

Trong một số tình huống, việc nhúng mã PHP vào các view của bạn rất hữu ích. Bạn có thể sử dụng directive `@php` của Blade để thực thi một khối PHP thuần túy trong template của mình:

```blade
@php
    $counter = 1;
@endphp
```

Hoặc, nếu bạn chỉ cần sử dụng PHP để import một class, bạn có thể sử dụng directive `@use`:

```blade
@use('App\Models\Flight')
```

Một đối số thứ hai có thể được cung cấp cho directive `@use` để đặt alias cho class đã import:

```blade
@use('App\Models\Flight', 'FlightModel')
```

Nếu bạn có nhiều class trong cùng một namespace, bạn có thể nhóm các import của các class đó:

```blade
@use('App\Models\{Flight, Airport}')
```

Directive `@use` cũng hỗ trợ import các hàm và hằng số PHP bằng cách thêm tiền tố `function` hoặc `const` vào đường dẫn import:

```blade
@use(function App\Helpers\format_currency)
@use(const App\Constants\MAX_ATTEMPTS)
```

Giống như import class, alias cũng được hỗ trợ cho hàm và hằng số:

```blade
@use(function App\Helpers\format_currency, 'formatMoney')
@use(const App\Constants\MAX_ATTEMPTS, 'MAX_TRIES')
```

Import nhóm cũng được hỗ trợ với cả hai modifier function và const, cho phép bạn import nhiều ký hiệu từ cùng một namespace trong một directive duy nhất:

```blade
@use(function App\Helpers\{format_currency, format_date})
@use(const App\Constants\{MAX_ATTEMPTS, DEFAULT_TIMEOUT})
```

<a name="fonts"></a>
### Fonts

Khi sử dụng [tối ưu hóa font của Laravel Vite](/docs/{{version}}/vite#working-with-fonts), bạn có thể sử dụng directive `@fonts` để render các liên kết preload font đã cấu hình và CSS font inline trong layout ứng dụng của mình:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @fonts
    @vite('resources/js/app.js')
</head>
```

Directive `@fonts` render tất cả các font family được cấu hình trong file `vite.config.js` của bạn. Directive thường nên được đặt trong `<head>` của layout gốc ứng dụng của bạn trước bất kỳ nội dung nào sử dụng các font đó.

Nếu một trang chỉ cần một số font đã cấu hình của bạn, bạn có thể truyền một hoặc nhiều alias font cho directive:

```blade
{{-- Load a single font alias... --}}
@fonts('sans')

{{-- Load multiple font aliases... --}}
@fonts(['sans', 'mono'])
```

Alias font được cấu hình bằng tùy chọn `alias` khi định nghĩa font trong cấu hình Vite của bạn. Directive `@fonts` gọi phương thức `fonts` được cung cấp bởi facade `Vite`, phương thức này cũng có thể được gọi trực tiếp:

```blade
{{ Vite::fonts(['sans', 'mono']) }}
```

<a name="comments"></a>
### Comments

Blade cũng cho phép bạn định nghĩa các comment trong các view của mình. Tuy nhiên, không giống như HTML comment, các comment Blade không được bao gồm trong HTML được trả về bởi ứng dụng của bạn:

```blade
{{-- This comment will not be present in the rendered HTML --}}
```

<a name="components"></a>
## Components

Components và slots cung cấp các lợi ích tương tự như sections, layouts, và includes; tuy nhiên, một số người có thể thấy mô hình tư duy của components và slots dễ hiểu hơn. Có hai cách tiếp cận để viết components: components dựa trên class và components ẩn danh.

Để tạo một component dựa trên class, bạn có thể sử dụng lệnh Artisan `make:component`. Để minh họa cách sử dụng components, chúng ta sẽ tạo một component `Alert` đơn giản. Lệnh `make:component` sẽ đặt component trong thư mục `app/View/Components`:

```shell
php artisan make:component Alert
```

Lệnh `make:component` cũng sẽ tạo một template view cho component. View sẽ được đặt trong thư mục `resources/views/components`. Khi viết components cho ứng dụng của riêng bạn, components được tự động phát hiện trong thư mục `app/View/Components` và thư mục `resources/views/components`, vì vậy thường không cần đăng ký component thêm.

Bạn cũng có thể tạo components trong các thư mục con:

```shell
php artisan make:component Forms/Input
```

Lệnh trên sẽ tạo một component `Input` trong thư mục `app/View/Components/Forms` và view sẽ được đặt trong thư mục `resources/views/components/forms`.

<a name="manually-registering-package-components"></a>
#### Manually Registering Package Components

Khi viết components cho ứng dụng của riêng bạn, components được tự động phát hiện trong thư mục `app/View/Components` và thư mục `resources/views/components`.

Tuy nhiên, nếu bạn đang xây dựng một package sử dụng Blade components, bạn sẽ cần đăng ký thủ công class component của mình và alias thẻ HTML của nó. Bạn thường nên đăng ký các component của mình trong phương thức `boot` của service provider của package:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', Alert::class);
}
```

Sau khi component của bạn đã được đăng ký, nó có thể được render bằng cách sử dụng alias thẻ của nó:

```blade
<x-package-alert/>
```

Ngoài ra, bạn có thể sử dụng phương thức `componentNamespace` để tự động tải các class component theo quy ước. Ví dụ, một package `Nightshade` có thể có các component `Calendar` và `ColorPicker` nằm trong namespace `Package\Views\Components`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

Điều này sẽ cho phép sử dụng các component của package bằng namespace vendor của chúng bằng cách sử dụng cú pháp `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade sẽ tự động phát hiện class được liên kết với component này bằng cách chuyển đổi tên component sang Pascal case. Các thư mục con cũng được hỗ trợ bằng cách sử dụng ký hiệu "dot".

<a name="rendering-components"></a>
### Rendering Components

Để hiển thị một component, bạn có thể sử dụng thẻ component Blade trong một trong các template Blade của mình. Các thẻ component Blade bắt đầu bằng chuỗi `x-` theo sau là tên kebab-case của class component:

```blade
<x-alert/>

<x-user-profile/>
```

Nếu class component được lồng sâu hơn trong thư mục `app/View/Components`, bạn có thể sử dụng ký tự `.` để chỉ ra việc lồng thư mục. Ví dụ, nếu chúng ta giả định một component nằm tại `app/View/Components/Inputs/Button.php`, chúng ta có thể render nó như sau:

```blade
<x-inputs.button/>
```

Nếu bạn muốn render component có điều kiện, bạn có thể định nghĩa một phương thức `shouldRender` trên class component của mình. Nếu phương thức `shouldRender` trả về `false`, component sẽ không được render:

```php
use Illuminate\Support\Str;

/**
 * Whether the component should be rendered
 */
public function shouldRender(): bool
{
    return Str::length($this->message) > 0;
}
```

<a name="index-components"></a>
### Index Components

Đôi khi components là một phần của một nhóm component và bạn có thể muốn nhóm các component liên quan trong một thư mục duy nhất. Ví dụ, hãy tưởng tượng một component "card" với cấu trúc class sau:

```text
App\Views\Components\Card\Card
App\Views\Components\Card\Header
App\Views\Components\Card\Body
```

Vì component `Card` gốc được lồng trong một thư mục `Card`, bạn có thể mong đợi rằng bạn sẽ cần render component thông qua `<x-card.card>`. Tuy nhiên, khi tên file của component khớp với tên thư mục của component, Laravel tự động giả định rằng component đó là component "gốc" và cho phép bạn render component mà không cần lặp lại tên thư mục:

```blade
<x-card>
    <x-card.header>...</x-card.header>
    <x-card.body>...</x-card.body>
</x-card>
```

<a name="passing-data-to-components"></a>
### Passing Data to Components

Bạn có thể truyền dữ liệu cho các Blade components bằng cách sử dụng các thuộc tính HTML. Các giá trị nguyên thủy được hard-coding có thể được truyền cho component bằng cách sử dụng các chuỗi thuộc tính HTML đơn giản. Các biểu thức và biến PHP nên được truyền cho component thông qua các thuộc tính sử dụng ký tự `:` làm tiền tố:

```blade
<x-alert type="error" :message="$message"/>
```

Bạn nên định nghĩa tất cả các thuộc tính dữ liệu của component trong constructor của class. Tất cả các thuộc tính public trên một component sẽ tự động được cung cấp cho view của component. Không cần thiết phải truyền dữ liệu cho view từ phương thức `render` của component:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class Alert extends Component
{
    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
        public string $message,
    ) {}

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View
    {
        return view('components.alert');
    }
}
```

Khi component của bạn được render, bạn có thể hiển thị nội dung của các biến public của component bằng cách echo các biến theo tên:

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

<a name="casing"></a>
#### Casing

Các đối số constructor của component nên được chỉ định bằng cách sử dụng `camelCase`, trong khi `kebab-case` nên được sử dụng khi tham chiếu tên đối số trong các thuộc tính HTML của bạn. Ví dụ, với constructor component sau:

```php
/**
 * Create the component instance.
 */
public function __construct(
    public string $alertType,
) {}
```

Đối số `$alertType` có thể được cung cấp cho component như sau:

```blade
<x-alert alert-type="danger" />
```

<a name="short-attribute-syntax"></a>
#### Short Attribute Syntax

Khi truyền thuộc tính cho các components, bạn cũng có thể sử dụng cú pháp "thuộc tính ngắn gọn". Điều này thường thuận tiện vì tên thuộc tính thường khớp với tên biến mà chúng tương ứng:

```blade
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />

{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

<a name="escaping-attribute-rendering"></a>
#### Escaping Attribute Rendering

Vì một số framework JavaScript như Alpine.js cũng sử dụng các thuộc tính có tiền tố dấu hai chấm, bạn có thể sử dụng tiền tố dấu hai chấm kép (`::`) để thông báo cho Blade rằng thuộc tính không phải là biểu thức PHP. Ví dụ, với component sau:

```blade
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

HTML sau sẽ được render bởi Blade:

```blade
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

<a name="component-methods"></a>
#### Component Methods

Ngoài các biến public có sẵn cho template component của bạn, bất kỳ phương thức public nào trên component cũng có thể được gọi. Ví dụ, hãy tưởng tượng một component có phương thức `isSelected`:

```php
/**
 * Determine if the given option is the currently selected option.
 */
public function isSelected(string $option): bool
{
    return $option === $this->selected;
}
```

Bạn có thể thực thi phương thức này từ template component của mình bằng cách gọi biến khớp với tên phương thức:

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

<a name="using-attributes-slots-within-component-class"></a>
#### Accessing Attributes and Slots Within Component Classes

Các Blade component cũng cho phép bạn truy cập tên component, thuộc tính, và slot trong phương thức render của class. Tuy nhiên, để truy cập dữ liệu này, bạn nên trả về một closure từ phương thức `render` của component:

```php
use Closure;

/**
 * Get the view / contents that represent the component.
 */
public function render(): Closure
{
    return function () {
        return '<div {{ $attributes }}>Components content</div>';
    };
}
```

Closure được trả về bởi phương thức `render` của component cũng có thể nhận một mảng `$data` làm đối số duy nhất của nó. Mảng này sẽ chứa một số phần tử cung cấp thông tin về component:

```php
return function (array $data) {
    // $data['componentName'];
    // $data['attributes'];
    // $data['slot'];

    return '<div {{ $attributes }}>Components content</div>';
}
```

> [!WARNING]
> Các phần tử trong mảng `$data` không bao giờ nên được nhúng trực tiếp vào chuỗi Blade được trả về bởi phương thức `render` của bạn, vì việc làm như vậy có thể cho phép thực thi mã từ xa thông qua nội dung thuộc tính độc hại.

`componentName` bằng với tên được sử dụng trong thẻ HTML sau tiền tố `x-`. Vì vậy `componentName` của `<x-alert />` sẽ là `alert`. Phần tử `attributes` sẽ chứa tất cả các thuộc tính có trên thẻ HTML. Phần tử `slot` là một instance `Illuminate\Support\HtmlString` với nội dung của slot của component.

Closure nên trả về một chuỗi. Nếu chuỗi được trả về tương ứng với một view hiện có, view đó sẽ được render; nếu không, chuỗi được trả về sẽ được đánh giá như một view Blade inline.

<a name="additional-dependencies"></a>
#### Additional Dependencies

Nếu component của bạn yêu cầu các dependency từ [service container](/docs/{{version}}/container) của Laravel, bạn có thể liệt kê chúng trước bất kỳ thuộc tính dữ liệu nào của component và chúng sẽ tự động được inject bởi container:

```php
use App\Services\AlertCreator;

/**
 * Create the component instance.
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```

<a name="hiding-attributes-and-methods"></a>
#### Hiding Attributes / Methods

Nếu bạn muốn ngăn một số phương thức hoặc thuộc tính public bị lộ dưới dạng biến cho template component của mình, bạn có thể thêm chúng vào một thuộc tính mảng `$except` trên component của mình:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var array
     */
    protected $except = ['type'];

    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
    ) {}
}
```

<a name="component-attributes"></a>
### Component Attributes

Chúng ta đã xem xét cách truyền thuộc tính dữ liệu cho một component; tuy nhiên, đôi khi bạn có thể cần chỉ định các thuộc tính HTML bổ sung, chẳng hạn như `class`, không phải là một phần của dữ liệu cần thiết để component hoạt động. Thông thường, bạn muốn chuyển các thuộc tính bổ sung này xuống phần tử gốc của template component. Ví dụ, hãy tưởng tượng chúng ta muốn render một component `alert` như sau:

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

Tất cả các thuộc tính không phải là một phần của constructor của component sẽ tự động được thêm vào "attribute bag" của component. Attribute bag này tự động được cung cấp cho component thông qua biến `$attributes`. Tất cả các thuộc tính có thể được render trong component bằng cách echo biến này:

```blade
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

> [!WARNING]
> Việc sử dụng các directive như `@env` trong các thẻ component hiện không được hỗ trợ. Ví dụ, `<x-alert :live="@env('production')"/>` sẽ không được biên dịch.

<a name="default-merged-attributes"></a>
#### Default / Merged Attributes

Đôi khi bạn có thể cần chỉ định các giá trị mặc định cho các thuộc tính hoặc hợp nhất các giá trị bổ sung vào một số thuộc tính của component. Để thực hiện điều này, bạn có thể sử dụng phương thức `merge` của attribute bag. Phương thức này đặc biệt hữu ích để định nghĩa một tập hợp các class CSS mặc định nên luôn được áp dụng cho một component:

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Nếu chúng ta giả định component này được sử dụng như sau:

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

HTML cuối cùng được render của component sẽ xuất hiện như sau:

```blade
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

<a name="conditionally-merge-classes"></a>
#### Conditionally Merge Classes

Đôi khi bạn có thể muốn hợp nhất các class nếu một điều kiện nhất định là `true`. Bạn có thể thực hiện điều này thông qua phương thức `class`, phương thức này chấp nhận một mảng các class trong đó khóa mảng chứa class hoặc các class bạn muốn thêm, trong khi giá trị là một biểu thức boolean. Nếu phần tử mảng có khóa số, nó sẽ luôn được bao gồm trong danh sách class được render:

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

Nếu bạn cần hợp nhất các thuộc tính khác vào component của mình, bạn có thể xâu chuỗi phương thức `merge` vào phương thức `class`:

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> [!NOTE]
> Nếu bạn cần biên dịch có điều kiện các class trên các phần tử HTML khác không nên nhận các thuộc tính đã hợp nhất, bạn có thể sử dụng [directive @class](#conditional-classes).

<a name="non-class-attribute-merging"></a>
#### Non-Class Attribute Merging

Khi hợp nhất các thuộc tính không phải là thuộc tính `class`, các giá trị được cung cấp cho phương thức `merge` sẽ được coi là các giá trị "mặc định" của thuộc tính. Tuy nhiên, không giống như thuộc tính `class`, các thuộc tính này sẽ không được hợp nhất với các giá trị thuộc tính được inject. Thay vào đó, chúng sẽ bị ghi đè. Ví dụ, việc triển khai một component `button` có thể trông như sau:

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```
Ngược lại, phương thức `whereDoesntStartWith` có thể được sử dụng để loại bỏ tất cả các thuộc tính có khóa bắt đầu bằng một chuỗi nhất định:

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

Sử dụng phương thức `first`, bạn có thể hiển thị thuộc tính đầu tiên trong một gói thuộc tính nhất định:

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

Nếu bạn muốn kiểm tra xem một thuộc tính có tồn tại trên component hay không, bạn có thể sử dụng phương thức `has`. Phương thức này chấp nhận tên thuộc tính làm đối số duy nhất và trả về một giá trị boolean cho biết thuộc tính có tồn tại hay không:

```blade
@if ($attributes->has('class'))
    <div>Thuộc tính class có mặt</div>
@endif
```

Nếu một mảng được truyền vào phương thức `has`, phương thức sẽ xác định xem tất cả các thuộc tính đã cho có tồn tại trên component hay không:

```blade
@if ($attributes->has(['name', 'class']))
    <div>Tất cả các thuộc tính đều có mặt</div>
@endif
```

Phương thức `hasAny` có thể được sử dụng để xác định xem bất kỳ thuộc tính nào đã cho có tồn tại trên component hay không:

```blade
@if ($attributes->hasAny(['href', ':href', 'v-bind:href']))
    <div>Một trong các thuộc tính có mặt</div>
@endif
```

Bạn có thể lấy giá trị của một thuộc tính cụ thể bằng phương thức `get`:

```blade
{{ $attributes->get('class') }}
```

Phương thức `only` có thể được sử dụng để chỉ lấy các thuộc tính với các khóa đã cho:

```blade
{{ $attributes->only(['class']) }}
```

Phương thức `except` có thể được sử dụng để lấy tất cả các thuộc tính ngoại trừ những thuộc tính có các khóa đã cho:

```blade
{{ $attributes->except(['class']) }}
```

<a name="reserved-keywords"></a>
### Từ Khóa Dành Riêng

Theo mặc định, một số từ khóa được dành riêng cho việc sử dụng nội bộ của Blade để hiển thị các component. Các từ khóa sau không thể được định nghĩa là thuộc tính công khai hoặc tên phương thức trong các component của bạn:

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolve`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

<a name="slots"></a>
### Slots

Bạn thường sẽ cần truyền thêm nội dung vào component của mình thông qua "slots". Các slot của component được hiển thị bằng cách echo biến `$slot`. Để khám phá khái niệm này, hãy tưởng tượng rằng một component `alert` có đánh dấu sau:

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Chúng ta có thể truyền nội dung vào `slot` bằng cách chèn nội dung vào component:

```blade
<x-alert>
    <strong>Whoops!</strong> Có gì đó sai sót!
</x-alert>
```

Đôi khi một component có thể cần hiển thị nhiều slot khác nhau ở các vị trí khác nhau trong component. Hãy sửa đổi component alert của chúng ta để cho phép chèn một slot "title":

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

Bạn có thể định nghĩa nội dung của slot được đặt tên bằng thẻ `x-slot`. Bất kỳ nội dung nào không nằm trong thẻ `x-slot` rõ ràng sẽ được truyền vào component trong biến `$slot`:

```xml
<x-alert>
    <x-slot:title>
        Lỗi Máy Chủ
    </x-slot>

    <strong>Whoops!</strong> Có gì đó sai sót!
</x-alert>
```

Bạn có thể gọi phương thức `isEmpty` của slot để xác định xem slot có chứa nội dung hay không:

```blade
<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    @if ($slot->isEmpty())
        Đây là nội dung mặc định nếu slot trống.
    @else
        {{ $slot }}
    @endif
</div>
```

Ngoài ra, phương thức `hasActualContent` có thể được sử dụng để xác định xem slot có chứa bất kỳ nội dung "thực tế" nào không phải là chú thích HTML hay không:

```blade
@if ($slot->hasActualContent())
    Scope có nội dung không phải chú thích.
@endif
```

<a name="scoped-slots"></a>
#### Scoped Slots

Nếu bạn đã sử dụng một framework JavaScript như Vue, bạn có thể quen thuộc với "scoped slots", cho phép bạn truy cập dữ liệu hoặc phương thức từ component trong slot của mình. Bạn có thể đạt được hành vi tương tự trong Laravel bằng cách định nghĩa các phương thức hoặc thuộc tính công khai trên component của mình và truy cập component trong slot của mình thông qua biến `$component`. Trong ví dụ này, chúng ta sẽ giả định rằng component `x-alert` có một phương thức công khai `formatAlert` được định nghĩa trên lớp component của nó:

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Lỗi Máy Chủ') }}
    </x-slot>

    <strong>Whoops!</strong> Có gì đó sai sót!
</x-alert>
```

<a name="slot-attributes"></a>
#### Thuộc Tính Slot

Giống như các component Blade, bạn có thể gán thêm [thuộc tính](#component-attributes) cho các slot như tên class CSS:

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Tiêu đề
    </x-slot>

    Nội dung

    <x-slot:footer class="text-sm">
        Chân trang
    </x-slot>
</x-card>
```

Để tương tác với các thuộc tính của slot, bạn có thể truy cập thuộc tính `attributes` của biến của slot. Để biết thêm thông tin về cách tương tác với các thuộc tính, vui lòng tham khảo tài liệu về [thuộc tính component](#component-attributes):

```blade
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

<a name="inline-component-views"></a>
### Component View Inline

Đối với các component rất nhỏ, việc quản lý cả lớp component và mẫu view của component có thể cảm thấy phức tạp. Vì lý do này, bạn có thể trả về đánh dấu của component trực tiếp từ phương thức `render`:

```php
/**
 * Lấy view / nội dung đại diện cho component.
 */
public function render(): string
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

<a name="generating-inline-view-components"></a>
#### Tạo Component View Inline

Để tạo một component hiển thị view inline, bạn có thể sử dụng tùy chọn `inline` khi thực thi lệnh `make:component`:

```shell
php artisan make:component Alert --inline
```

<a name="dynamic-components"></a>
### Component Động

Đôi khi bạn có thể cần hiển thị một component nhưng không biết component nào nên được hiển thị cho đến thời điểm chạy. Trong tình huống này, bạn có thể sử dụng component `dynamic-component` tích hợp sẵn của Laravel để hiển thị component dựa trên một giá trị thời gian chạy hoặc biến:

```blade
// $componentName = "secondary-button";

<x-dynamic-component :component="$componentName" class="mt-4" />
```

<a name="manually-registering-components"></a>
### Đăng Ký Component Thủ Công

> [!WARNING]
> Tài liệu sau đây về đăng ký component thủ công chủ yếu áp dụng cho những người đang viết các gói Laravel bao gồm các component view. Nếu bạn không viết gói, phần này của tài liệu component có thể không liên quan đến bạn.

Khi viết component cho ứng dụng của riêng bạn, các component được tự động phát hiện trong thư mục `app/View/Components` và thư mục `resources/views/components`.

Tuy nhiên, nếu bạn đang xây dựng một gói sử dụng các component Blade hoặc đặt các component trong các thư mục không theo quy ước, bạn sẽ cần đăng ký thủ công lớp component của mình và bí danh thẻ HTML của nó để Laravel biết nơi tìm component. Bạn thường nên đăng ký các component của mình trong phương thức `boot` của nhà cung cấp dịch vụ của gói:

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Khởi tạo các dịch vụ của gói.
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

Sau khi component của bạn đã được đăng ký, nó có thể được hiển thị bằng bí danh thẻ của nó:

```blade
<x-package-alert/>
```

#### Tự Động Tải Component Gói

Ngoài ra, bạn có thể sử dụng phương thức `componentNamespace` để tự động tải các lớp component theo quy ước. Ví dụ, một gói `Nightshade` có thể có các component `Calendar` và `ColorPicker` nằm trong không gian tên `Package\Views\Components`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Khởi tạo các dịch vụ của gói.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

Điều này sẽ cho phép sử dụng các component gói theo không gian tên nhà cung cấp của chúng bằng cú pháp `package-name::`:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade sẽ tự động phát hiện lớp được liên kết với component này bằng cách chuyển đổi tên component thành PascalCase. Các thư mục con cũng được hỗ trợ bằng cách sử dụng ký hiệu "dot".

<a name="anonymous-components"></a>
## Component Ẩn Danh

Tương tự như các component inline, các component ẩn danh cung cấp một cơ chế để quản lý một component thông qua một tệp duy nhất. Tuy nhiên, các component ẩn danh sử dụng một tệp view duy nhất và không có lớp liên kết. Để định nghĩa một component ẩn danh, bạn chỉ cần đặt một mẫu Blade trong thư mục `resources/views/components` của mình. Ví dụ, giả sử bạn đã định nghĩa một component tại `resources/views/components/alert.blade.php`, bạn có thể chỉ cần hiển thị nó như sau:

```blade
<x-alert/>
```

Bạn có thể sử dụng ký tự `.` để chỉ ra nếu một component được lồng sâu hơn trong thư mục `components`. Ví dụ, giả sử component được định nghĩa tại `resources/views/components/inputs/button.blade.php`, bạn có thể hiển thị nó như sau:

```blade
<x-inputs.button/>
```

Để tạo một component ẩn danh thông qua Artisan, bạn có thể sử dụng cờ `--view` khi gọi lệnh `make:component`:

```shell
php artisan make:component forms.input --view
```

Lệnh trên sẽ tạo một tệp Blade tại `resources/views/components/forms/input.blade.php` có thể được hiển thị như một component thông qua `<x-forms.input />`.

<a name="anonymous-index-components"></a>
### Component Index Ẩn Danh

Đôi khi, khi một component được tạo thành từ nhiều mẫu Blade, bạn có thể muốn nhóm các mẫu của component đã cho trong một thư mục duy nhất. Ví dụ, hãy tưởng tượng một component "accordion" với cấu trúc thư mục sau:

```text
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

Cấu trúc thư mục này cho phép bạn hiển thị component accordion và mục của nó như sau:

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

Tuy nhiên, để hiển thị component accordion thông qua `x-accordion`, chúng ta buộc phải đặt mẫu component accordion "index" trong thư mục `resources/views/components` thay vì lồng nó trong thư mục `accordion` với các mẫu liên quan đến accordion khác.

May mắn thay, Blade cho phép bạn đặt một tệp khớp với tên thư mục của component trong chính thư mục của component. Khi mẫu này tồn tại, nó có thể được hiển thị như phần tử "root" của component mặc dù nó được lồng trong một thư mục. Vì vậy, chúng ta có thể tiếp tục sử dụng cùng một cú pháp Blade được đưa ra trong ví dụ trên; tuy nhiên, chúng ta sẽ điều chỉnh cấu trúc thư mục của mình như sau:

```text
/resources/views/components/accordion/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

<a name="data-properties-attributes"></a>
### Thuộc Tính Dữ Liệu / Thuộc Tính

Vì các component ẩn danh không có bất kỳ lớp liên kết nào, bạn có thể tự hỏi làm thế nào để phân biệt dữ liệu nào nên được truyền vào component như biến và thuộc tính nào nên được đặt trong [gói thuộc tính](#component-attributes) của component.

Bạn có thể chỉ định các thuộc tính nào nên được coi là biến dữ liệu bằng directive `@props` ở đầu mẫu Blade của component. Tất cả các thuộc tính khác trên component sẽ có sẵn thông qua gói thuộc tính của component. Nếu bạn muốn cung cấp cho một biến dữ liệu một giá trị mặc định, bạn có thể chỉ định tên biến làm khóa mảng và giá trị mặc định làm giá trị mảng:

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

Với định nghĩa component ở trên, chúng ta có thể hiển thị component như sau:

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### Truy Cập Dữ Liệu Cha

Đôi khi bạn có thể muốn truy cập dữ liệu từ một component cha trong một component con. Trong những trường hợp này, bạn có thể sử dụng directive `@aware`. Ví dụ, hãy tưởng tượng chúng ta đang xây dựng một component menu phức tạp bao gồm một component cha `<x-menu>` và component con `<x-menu.item>`:

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

Component `<x-menu>` có thể có một triển khai như sau:

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

Vì prop `color` chỉ được truyền vào component cha (`<x-menu>`), nó sẽ không có sẵn trong `<x-menu.item>`. Tuy nhiên, nếu chúng ta sử dụng directive `@aware`, chúng ta có thể làm cho nó có sẵn trong `<x-menu.item>` cũng như:

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> [!WARNING]
> Directive `@aware` không thể truy cập dữ liệu cha không được truyền rõ ràng vào component cha thông qua các thuộc tính HTML. Giá trị `@props` mặc định không được truyền rõ ràng vào component cha không thể được truy cập bởi directive `@aware`.

<a name="anonymous-component-paths"></a>
### Đường Dẫn Component Ẩn Danh

Như đã thảo luận trước đó, các component ẩn danh thường được định nghĩa bằng cách đặt một mẫu Blade trong thư mục `resources/views/components` của bạn. Tuy nhiên, đôi khi bạn có thể muốn đăng ký các đường dẫn component ẩn danh khác với Laravel ngoài đường dẫn mặc định.

Phương thức `anonymousComponentPath` chấp nhận "đường dẫn" đến vị trí component ẩn danh làm đối số đầu tiên và một "namespace" tùy chọn mà các component nên được đặt dưới làm đối số thứ hai. Thông thường, phương thức này nên được gọi từ phương thức `boot` của một trong các [nhà cung cấp dịch vụ](/docs/{{version}}/providers) của ứng dụng:

```php
/**
 * Khởi tạo bất kỳ dịch vụ ứng dụng nào.
 */
public function boot(): void
{
    Blade::anonymousComponentPath(__DIR__.'/../components');
}
```

Khi các đường dẫn component được đăng ký mà không có tiền tố được chỉ định như trong ví dụ trên, chúng có thể được hiển thị trong các component Blade của bạn mà không có tiền tố tương ứng cũng như. Ví dụ, nếu một component `panel.blade.php` tồn tại trong đường dẫn đã đăng ký ở trên, nó có thể được hiển thị như sau:

```blade
<x-panel />
```

Tiền tố "namespace" có thể được cung cấp làm đối số thứ hai cho phương thức `anonymousComponentPath`:

```php
Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');
```

Khi một tiền tố được cung cấp, các component trong "namespace" đó có thể được hiển thị bằng cách thêm tiền tố namespace của component vào tên component khi component được hiển thị:

```blade
<x-dashboard::panel />
```

<a name="building-layouts"></a>
## Xây Dựng Layout

<a name="layouts-using-components"></a>
### Layout Sử Dụng Component

Hầu hết các ứng dụng web duy trì cùng một layout chung trên các trang khác nhau. Nó sẽ cực kỳ phức tạp và khó duy trì ứng dụng của chúng ta nếu chúng ta phải lặp lại toàn bộ HTML layout trong mọi view mà chúng ta tạo. May mắn thay, rất thuận tiện để định nghĩa layout này như một [component Blade](#components) duy nhất và sau đó sử dụng nó trong toàn bộ ứng dụng của chúng ta.

<a name="defining-the-layout-component"></a>
#### Định Nghĩa Component Layout

Ví dụ, hãy tưởng tượng chúng ta đang xây dựng một ứng dụng danh sách "todo". Chúng ta có thể định nghĩa một component `layout` trông như sau:

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

<a name="applying-the-layout-component"></a>
#### Áp Dụng Component Layout

Sau khi component `layout` đã được định nghĩa, chúng ta có thể tạo một view Blade sử dụng component. Trong ví dụ này, chúng ta sẽ định nghĩa một view đơn giản hiển thị danh sách nhiệm vụ của chúng ta:

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

Nhớ rằng, nội dung được chèn vào một component sẽ được cung cấp cho biến `$slot` mặc định trong component `layout` của chúng ta. Như bạn có thể đã nhận thấy, `layout` của chúng ta cũng tôn trọng một slot `$title` nếu một slot được cung cấp; nếu không, một tiêu đề mặc định được hiển thị. Chúng ta có thể chèn một tiêu đề tùy chỉnh từ view danh sách nhiệm vụ của chúng ta bằng cú pháp slot tiêu chuẩn được thảo luận trong [tài liệu component](#components):

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Tiêu đề Tùy Chỉnh
    </x-slot>

    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

Bây giờ chúng ta đã định nghĩa layout và view danh sách nhiệm vụ của mình, chúng ta chỉ cần trả về view `task` từ một route:

```php
use App\Models\Task;

Route::get('/tasks', function () {
    return view('tasks', ['tasks' => Task::all()]);
});
```

<a name="layouts-using-template-inheritance"></a>
### Layout Sử Dụng Kế Thừa Mẫu

<a name="defining-a-layout"></a>
#### Định Nghĩa Layout

Layout cũng có thể được tạo thông qua "kế thừa mẫu". Đây là cách chính để xây dựng các ứng dụng trước khi giới thiệu [các component](#components).

Để bắt đầu, hãy xem một ví dụ đơn giản. Đầu tiên, chúng ta sẽ xem xét một layout trang. Vì hầu hết các ứng dụng web duy trì cùng một layout chung trên các trang khác nhau, rất thuận tiện để định nghĩa layout này như một view Blade duy nhất:

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            Đây là sidebar chính.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

Như bạn có thể thấy, tệp này chứa đánh dấu HTML điển hình. Tuy nhiên, hãy lưu ý các directive `@section` và `@yield`. Directive `@section`, như tên gọi, định nghĩa một phần nội dung, trong khi directive `@yield` được sử dụng để hiển thị nội dung của một phần nhất định.

Bây giờ chúng ta đã định nghĩa một layout cho ứng dụng của mình, hãy định nghĩa một trang con kế thừa layout.

<a name="extending-a-layout"></a>
#### Kế Thừa Layout

Khi định nghĩa một view con, sử dụng directive Blade `@extends` để chỉ định layout nào mà view con nên "kế thừa". Các view kế thừa một layout Blade có thể chèn nội dung vào các phần của layout bằng các directive `@section`. Nhớ rằng, như thấy trong ví dụ trên, nội dung của các phần này sẽ được hiển thị trong layout bằng `@yield`:

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Tiêu đề Trang')

@section('sidebar')
    @@parent

    <p>Đây được thêm vào sidebar chính.</p>
@endsection

@section('content')
    <p>Đây là nội dung thân của tôi.</p>
@endsection
```

Trong ví dụ này, phần `sidebar` đang sử dụng directive `@@parent` để thêm (thay vì ghi đè) nội dung vào sidebar của layout. Directive `@@parent` sẽ được thay thế bằng nội dung của layout khi view được hiển thị.

> [!NOTE]
> Trái ngược với ví dụ trước, phần `sidebar` này kết thúc bằng `@endsection` thay vì `@show`. Directive `@endsection` sẽ chỉ định nghĩa một phần trong khi `@show` sẽ định nghĩa và **hiển thị ngay lập tức** phần đó.

Directive `@yield` cũng chấp nhận một giá trị mặc định làm tham số thứ hai. Giá trị này sẽ được hiển thị nếu phần được hiển thị không được định nghĩa:

```blade
@yield('content', 'Nội dung mặc định')
```

<a name="forms"></a>
## Form

<a name="csrf-field"></a>
### Trường CSRF

Bất cứ khi nào bạn định nghĩa một form HTML trong ứng dụng của mình, bạn nên bao gồm một trường token CSRF ẩn trong form để [middleware bảo vệ CSRF](/docs/{{version}}/csrf) có thể xác thực yêu cầu. Bạn có thể sử dụng directive Blade `@csrf` để tạo trường token:

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

<a name="method-field"></a>
### Trường Phương Thức

Vì các form HTML không thể thực hiện các yêu cầu `PUT`, `PATCH`, hoặc `DELETE`, bạn sẽ cần thêm một trường ẩn `_method` để giả mạo các động từ HTTP này. Directive Blade `@method` có thể tạo trường này cho bạn:

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```<a name="validation-errors"></a>
### Lỗi Xác Thực

Directive `@error` có thể được sử dụng để kiểm tra nhanh xem [thông báo lỗi xác thực](/docs/{{version}}/validation#quick-displaying-the-validation-errors) có tồn tại cho một thuộc tính nhất định hay không. Trong directive `@error`, bạn có thể hiển thị biến `$message` để hiển thị thông báo lỗi:

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

Vì directive `@error` được biên dịch thành câu lệnh "if", bạn có thể sử dụng directive `@else` để hiển thị nội dung khi không có lỗi cho một thuộc tính:

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror"
/>
```

Bạn có thể truyền [tên của một error bag cụ thể](/docs/{{version}}/validation#named-error-bags) làm tham số thứ hai cho directive `@error` để lấy thông báo lỗi xác thực trên các trang chứa nhiều form:

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror"
/>

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

<a name="stacks"></a>
## Stacks

Blade cho phép bạn đẩy vào các stack có tên có thể được hiển thị ở một nơi khác trong view hoặc layout khác. Điều này có thể đặc biệt hữu ích để chỉ định bất kỳ thư viện JavaScript nào được yêu cầu bởi các view con của bạn:

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

Nếu bạn muốn `@push` nội dung nếu một biểu thức boolean nhất định đánh giá là `true`, bạn có thể sử dụng directive `@pushIf`:

```blade
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

Bạn có thể đẩy vào một stack nhiều lần khi cần. Để hiển thị toàn bộ nội dung của stack, hãy truyền tên của stack cho directive `@stack`:

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

Nếu bạn muốn thêm nội dung vào đầu một stack, bạn nên sử dụng directive `@prepend`:

```blade
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

Directive `@hasstack` có thể được sử dụng để xác định xem một stack có trống hay không:

```blade
@hasstack('list')
    <ul>
        @stack('list')
    </ul>
@endif
```

<a name="service-injection"></a>
## Service Injection

Directive `@inject` có thể được sử dụng để lấy một service từ [service container](/docs/{{version}}/container) của Laravel. Tham số đầu tiên được truyền cho `@inject` là tên của biến mà service sẽ được đặt vào, trong khi tham số thứ hai là tên class hoặc interface của service bạn muốn giải quyết:

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="rendering-inline-blade-templates"></a>
## Rendering Inline Blade Templates

Đôi khi bạn có thể cần chuyển đổi một chuỗi template Blade thô thành HTML hợp lệ. Bạn có thể thực hiện việc này bằng cách sử dụng phương thức `render` được cung cấp bởi facade `Blade`. Phương thức `render` chấp nhận chuỗi template Blade và một mảng dữ liệu tùy chọn để cung cấp cho template:

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel render các template Blade inline bằng cách ghi chúng vào thư mục `storage/framework/views`. Nếu bạn muốn Laravel xóa các tệp tạm thời này sau khi render template Blade, bạn có thể cung cấp đối số `deleteCachedView` cho phương thức:

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

<a name="rendering-blade-fragments"></a>
## Rendering Blade Fragments

Khi sử dụng các framework frontend như [Turbo](https://turbo.hotwired.dev/) và [htmx](https://htmx.org/), đôi khi bạn có thể cần chỉ trả về một phần của template Blade trong phản hồi HTTP của mình. Blade "fragments" cho phép bạn làm điều đó. Để bắt đầu, hãy đặt một phần của template Blade của bạn trong các directive `@fragment` và `@endfragment`:

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

Sau đó, khi render view sử dụng template này, bạn có thể gọi phương thức `fragment` để chỉ định rằng chỉ fragment được chỉ định nên được bao gồm trong phản hồi HTTP đi ra:

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

Phương thức `fragmentIf` cho phép bạn có điều kiện trả về một fragment của view dựa trên một điều kiện nhất định. Nếu không, toàn bộ view sẽ được trả về:

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

Các phương thức `fragments` và `fragmentsIf` cho phép bạn trả về nhiều fragment view trong phản hồi. Các fragment sẽ được nối lại với nhau:

```php
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

<a name="extending-blade"></a>
## Extending Blade

Blade cho phép bạn định nghĩa các directive tùy chỉnh của riêng mình bằng phương thức `directive`. Khi trình biên dịch Blade gặp directive tùy chỉnh, nó sẽ gọi callback được cung cấp với biểu thức mà directive chứa.

Ví dụ sau tạo một directive `@datetime($var)` định dạng một `$var` nhất định, phải là một thể hiện của `DateTime`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
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
        Blade::directive('datetime', function (string $expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

Như bạn có thể thấy, chúng ta sẽ nối phương thức `format` vào bất kỳ biểu thức nào được truyền vào directive. Vì vậy, trong ví dụ này, PHP cuối cùng được tạo ra bởi directive này sẽ là:

```php
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

> [!WARNING]
> Sau khi cập nhật logic của một directive Blade, bạn sẽ cần xóa tất cả các view Blade đã được cache. Các view Blade đã được cache có thể được xóa bằng lệnh Artisan `view:clear`.

<a name="custom-echo-handlers"></a>
### Custom Echo Handlers

Nếu bạn cố gắng "echo" một đối tượng bằng Blade, phương thức `__toString` của đối tượng sẽ được gọi. Phương thức [__toString](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) là một trong các "magic methods" tích hợp của PHP. Tuy nhiên, đôi khi bạn có thể không kiểm soát được phương thức `__toString` của một class nhất định, chẳng hạn như khi class mà bạn đang tương tác thuộc về một thư viện bên thứ ba.

Trong những trường hợp này, Blade cho phép bạn đăng ký một trình xử lý echo tùy chỉnh cho loại đối tượng cụ thể đó. Để thực hiện việc này, bạn nên gọi phương thức `stringable` của Blade. Phương thức `stringable` chấp nhận một closure. Closure này nên type-hint loại đối tượng mà nó chịu trách nhiệm render. Thông thường, phương thức `stringable` nên được gọi trong phương thức `boot` của class `AppServiceProvider` của ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Blade;
use Money\Money;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

Sau khi trình xử lý echo tùy chỉnh của bạn đã được định nghĩa, bạn có thể đơn giản echo đối tượng trong template Blade của mình:

```blade
Cost: {{ $money }}
```

<a name="custom-if-statements"></a>
### Custom If Statements

Lập trình một directive tùy chỉnh đôi khi phức tạp hơn mức cần thiết khi định nghĩa các câu lệnh điều kiện tùy chỉnh đơn giản. Vì lý do đó, Blade cung cấp phương thức `Blade::if` cho phép bạn định nghĩa nhanh các directive điều kiện tùy chỉnh bằng cách sử dụng closures. Ví dụ, hãy định nghĩa một điều kiện tùy chỉnh kiểm tra "disk" mặc định đã cấu hình cho ứng dụng. Chúng ta có thể làm điều này trong phương thức `boot` của `AppServiceProvider` của chúng ta:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::if('disk', function (string $value) {
        return config('filesystems.default') === $value;
    });
}
```

Sau khi điều kiện tùy chỉnh đã được định nghĩa, bạn có thể sử dụng nó trong các template của mình:

```blade
@disk('local')
    <!-- The application is using the local disk... -->
@elsedisk('s3')
    <!-- The application is using the s3 disk... -->
@else
    <!-- The application is using some other disk... -->
@enddisk

@unlessdisk('local')
    <!-- The application is not using the local disk... -->
@enddisk
```

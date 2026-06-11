# Localization

- [Introduction](#introduction)
    - [Publishing the Language Files](#publishing-the-language-files)
    - [Configuring the Locale](#configuring-the-locale)
    - [Pluralization Language](#pluralization-language)
- [Defining Translation Strings](#defining-translation-strings)
    - [Using Short Keys](#using-short-keys)
    - [Using Translation Strings as Keys](#using-translation-strings-as-keys)
- [Retrieving Translation Strings](#retrieving-translation-strings)
    - [Replacing Parameters in Translation Strings](#replacing-parameters-in-translation-strings)
    - [Pluralization](#pluralization)
- [Overriding Package Language Files](#overriding-package-language-files)

<a name="introduction"></a>
## Introduction

> [!NOTE]
> Theo mặc định, skeleton ứng dụng Laravel không bao gồm thư mục `lang`. Nếu bạn muốn tùy chỉnh các file ngôn ngữ của Laravel, bạn có thể publish chúng thông qua lệnh Artisan `lang:publish`.

Các tính năng localization của Laravel cung cấp một cách thuận tiện để truy xuất các chuỗi trong nhiều ngôn ngữ khác nhau, cho phép bạn dễ dàng hỗ trợ nhiều ngôn ngữ trong ứng dụng của mình.

Laravel cung cấp hai cách để quản lý các chuỗi dịch. Đầu tiên, các chuỗi ngôn ngữ có thể được lưu trữ trong các file trong thư mục `lang` của ứng dụng. Trong thư mục này, có thể có các thư mục con cho mỗi ngôn ngữ được ứng dụng hỗ trợ. Đây là cách tiếp cận mà Laravel sử dụng để quản lý các chuỗi dịch cho các tính năng tích hợp sẵn của Laravel như thông báo lỗi xác thực:

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

Hoặc, các chuỗi dịch có thể được định nghĩa trong các file JSON được đặt trong thư mục `lang`. Khi sử dụng cách tiếp cận này, mỗi ngôn ngữ được ứng dụng hỗ trợ sẽ có một file JSON tương ứng trong thư mục này. Cách tiếp cận này được khuyến nghị cho các ứng dụng có số lượng lớn chuỗi có thể dịch:

```text
/lang
    en.json
    es.json
```

Chúng tôi sẽ thảo luận về từng cách tiếp cận để quản lý các chuỗi dịch trong tài liệu này.

<a name="publishing-the-language-files"></a>
### Publishing the Language Files

Theo mặc định, skeleton ứng dụng Laravel không bao gồm thư mục `lang`. Nếu bạn muốn tùy chỉnh các file ngôn ngữ của Laravel hoặc tạo file của riêng bạn, bạn nên scaffold thư mục `lang` thông qua lệnh Artisan `lang:publish`. Lệnh `lang:publish` sẽ tạo thư mục `lang` trong ứng dụng của bạn và publish bộ mặc định các file ngôn ngữ được sử dụng bởi Laravel:

```shell
php artisan lang:publish
```

<a name="configuring-the-locale"></a>
### Configuring the Locale

Ngôn ngữ mặc định cho ứng dụng của bạn được lưu trữ trong tùy chọn cấu hình `locale` của file cấu hình `config/app.php`, thường được đặt bằng biến môi trường `APP_LOCALE`. Bạn tự do sửa đổi giá trị này để phù hợp với nhu cầu của ứng dụng của bạn.

Bạn cũng có thể cấu hình một "fallback language", sẽ được sử dụng khi ngôn ngữ mặc định không chứa một chuỗi dịch nhất định. Giống như ngôn ngữ mặc định, fallback language cũng được cấu hình trong file cấu hình `config/app.php`, và giá trị của nó thường được đặt bằng biến môi trường `APP_FALLBACK_LOCALE`.

Bạn có thể sửa đổi ngôn ngữ mặc định cho một HTTP request duy nhất tại runtime bằng cách sử dụng phương thức `setLocale` được cung cấp bởi facade `App`:

```php
use Illuminate\Support\Facades\App;

Route::get('/greeting/{locale}', function (string $locale) {
    if (! in_array($locale, ['en', 'es', 'fr'])) {
        abort(400);
    }

    App::setLocale($locale);

    // ...
});
```

<a name="determining-the-current-locale"></a>
#### Determining the Current Locale

Bạn có thể sử dụng các phương thức `currentLocale` và `isLocale` trên facade `App` để xác định locale hiện tại hoặc kiểm tra xem locale có phải là một giá trị nhất định hay không:

```php
use Illuminate\Support\Facades\App;

$locale = App::currentLocale();

if (App::isLocale('en')) {
    // ...
}
```

<a name="pluralization-language"></a>
### Pluralization Language

<style>
.code-list-no-flex-break code {
    display: contents !important;
}
</style>

<div class="code-list-no-flex-break">

Bạn có thể hướng dẫn "pluralizer" của Laravel, được sử dụng bởi Eloquent và các phần khác của framework để chuyển đổi các chuỗi số ít thành các chuỗi số nhiều, để sử dụng một ngôn ngữ khác ngoài tiếng Anh. Điều này có thể được thực hiện bằng cách gọi phương thức `useLanguage` trong phương thức `boot` của một trong các service providers của ứng dụng. Các ngôn ngữ hiện được hỗ trợ bởi pluralizer là: `french`, `norwegian-bokmal`, `portuguese`, `spanish`, và `turkish`:

</div>

```php
use Illuminate\Support\Pluralizer;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pluralizer::useLanguage('spanish');

    // ...
}
```

> [!WARNING]
> Nếu bạn tùy chỉnh ngôn ngữ của pluralizer, bạn nên định nghĩa rõ ràng [tên bảng](/docs/{{version}}/eloquent#table-names) của Eloquent model của mình.

<a name="defining-translation-strings"></a>
## Defining Translation Strings

<a name="using-short-keys"></a>
### Using Short Keys

Thông thường, các chuỗi dịch được lưu trữ trong các file trong thư mục `lang`. Trong thư mục này, nên có một thư mục con cho mỗi ngôn ngữ được ứng dụng hỗ trợ. Đây là cách tiếp cận mà Laravel sử dụng để quản lý các chuỗi dịch cho các tính năng tích hợp sẵn của Laravel như thông báo lỗi xác thực:

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

Tất cả các file ngôn ngữ trả về một array của các chuỗi được khóa. Ví dụ:

```php
<?php

// lang/en/messages.php

return [
    'welcome' => 'Welcome to our application!',
];
```

> [!WARNING]
> Đối với các ngôn ngữ khác nhau theo lãnh thổ, bạn nên đặt tên các thư mục ngôn ngữ theo ISO 15897. Ví dụ, "en_GB" nên được sử dụng cho tiếng Anh Anh thay vì "en-gb".

<a name="using-translation-strings-as-keys"></a>
### Using Translation Strings as Keys

Đối với các ứng dụng có số lượng lớn chuỗi có thể dịch, việc định nghĩa mỗi chuỗi bằng một "short key" có thể trở nên khó hiểu khi tham chiếu các keys trong các views của bạn và việc liên tục phát minh các keys cho mỗi chuỗi dịch được ứng dụng hỗ trợ là cồng kềnh.

Vì lý do này, Laravel cũng cung cấp hỗ trợ để định nghĩa các chuỗi dịch bằng cách sử dụng "dịch mặc định" của chuỗi làm key. Các file ngôn ngữ sử dụng các chuỗi dịch làm keys được lưu trữ dưới dạng các file JSON trong thư mục `lang`. Ví dụ, nếu ứng dụng của bạn có một bản dịch tiếng Tây Ban Nha, bạn nên tạo một file `lang/es.json`:

```json
{
    "I love programming.": "Me encanta programar."
}
```

#### Key / File Conflicts

Bạn không nên định nghĩa các key chuỗi dịch xung đột với các tên file dịch khác. Ví dụ, dịch `__('Action')` cho locale "NL" trong khi một file `nl/action.php` tồn tại nhưng một file `nl.json` không tồn tại sẽ dẫn đến việc translator trả về toàn bộ nội dung của `nl/action.php`.

<a name="retrieving-translation-strings"></a>
## Retrieving Translation Strings

Bạn có thể truy xuất các chuỗi dịch từ các file ngôn ngữ của mình bằng cách sử dụng hàm helper `__`. Nếu bạn đang sử dụng "short keys" để định nghĩa các chuỗi dịch của mình, bạn nên chuyển file chứa key và chính key đó cho hàm `__` bằng cách sử dụng cú pháp "dot". Ví dụ, hãy truy xuất chuỗi dịch `welcome` từ file ngôn ngữ `lang/en/messages.php`:

```php
echo __('messages.welcome');
```

Nếu chuỗi dịch được chỉ định không tồn tại, hàm `__` sẽ trả về key chuỗi dịch. Vì vậy, sử dụng ví dụ trên, hàm `__` sẽ trả về `messages.welcome` nếu chuỗi dịch không tồn tại.

Nếu bạn đang sử dụng [các chuỗi dịch mặc định của mình làm các key dịch của mình](#using-translation-strings-as-keys), bạn nên chuyển dịch mặc định của chuỗi của mình cho hàm `__`;

```php
echo __('I love programming.');
```

Một lần nữa, nếu chuỗi dịch không tồn tại, hàm `__` sẽ trả về key chuỗi dịch mà nó đã được đưa.

Nếu bạn đang sử dụng [Blade templating engine](/docs/{{version}}/blade), bạn có thể sử dụng cú pháp echo `{{ }}` để hiển thị chuỗi dịch:

```blade
{{ __('messages.welcome') }}
```

<a name="replacing-parameters-in-translation-strings"></a>
### Replacing Parameters in Translation Strings

Nếu bạn muốn, bạn có thể định nghĩa các placeholders trong các chuỗi dịch của mình. Tất cả các placeholders đều có tiền tố là `:`. Ví dụ, bạn có thể định nghĩa một thông báo chào mừng với một tên placeholder:

```php
'welcome' => 'Welcome, :name',
```

Để thay thế các placeholders khi truy xuất một chuỗi dịch, bạn có thể chuyển một array các thay thế làm đối số thứ hai cho hàm `__`:

```php
echo __('messages.welcome', ['name' => 'dayle']);
```

Nếu placeholder của bạn chứa tất cả các chữ cái viết hoa, hoặc chỉ có chữ cái đầu tiên được viết hoa, giá trị đã dịch sẽ được viết hoa tương ứng:

```php
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
```

<a name="object-replacement-formatting"></a>
#### Object Replacement Formatting

Nếu bạn cố gắng cung cấp một đối tượng làm placeholder dịch, phương thức `__toString` của đối tượng sẽ được gọi. Phương thức [__toString](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) là một trong các "magic methods" tích hợp sẵn của PHP. Tuy nhiên, đôi khi bạn có thể không kiểm soát được phương thức `__toString` của một lớp nhất định, chẳng hạn như khi lớp mà bạn đang tương tác thuộc về một thư viện bên thứ ba.

Trong những trường hợp này, Laravel cho phép bạn đăng ký một trình xử lý định dạng tùy chỉnh cho loại đối tượng cụ thể đó. Để thực hiện điều này, bạn nên gọi phương thức `stringable` của translator. Phương thức `stringable` chấp nhận một closure, nên type-hint loại đối tượng mà nó chịu trách nhiệm định dạng. Thông thường, phương thức `stringable` nên được gọi trong phương thức `boot` của lớp `AppServiceProvider` của ứng dụng của bạn:

```php
use Illuminate\Support\Facades\Lang;
use Money\Money;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Lang::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

<a name="pluralization"></a>
### Pluralization

Số nhiều là một vấn đề phức tạp, vì các ngôn ngữ khác nhau có nhiều quy tắc phức tạp cho số nhiều; tuy nhiên, Laravel có thể giúp bạn dịch các chuỗi khác nhau dựa trên các quy tắc số nhiều mà bạn định nghĩa. Sử dụng một ký tự `|`, bạn có thể phân biệt các dạng số ít và số nhiều của một chuỗi:

```php
'apples' => 'There is one apple|There are many apples',
```

Tất nhiên, số nhiều cũng được hỗ trợ khi sử dụng [các chuỗi dịch làm keys](#using-translation-strings-as-keys):

```json
{
    "There is one apple|There are many apples": "Hay una manzana|Hay muchas manzanas"
}
```

Bạn thậm chí có thể tạo các quy tắc số nhiều phức tạp hơn chỉ định các chuỗi dịch cho nhiều phạm vi giá trị:

```php
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
```

Sau khi định nghĩa một chuỗi dịch có các tùy chọn số nhiều, bạn có thể sử dụng hàm `trans_choice` để truy xuất dòng cho một "count" nhất định. Trong ví dụ này, vì count lớn hơn một, dạng số nhiều của chuỗi dịch được trả về:

```php
echo trans_choice('messages.apples', 10);
```

Bạn cũng có thể định nghĩa các thuộc tính placeholder trong các chuỗi số nhiều. Các placeholders này có thể được thay thế bằng cách chuyển một array làm đối số thứ ba cho hàm `trans_choice`:

```php
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

Nếu bạn muốn hiển thị giá trị số nguyên được chuyển cho hàm `trans_choice`, bạn có thể sử dụng placeholder tích hợp sẵn `:count`:

```php
'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',
```

<a name="overriding-package-language-files"></a>
## Overriding Package Language Files

Một số packages có thể được gửi kèm với các file ngôn ngữ của riêng chúng. Thay vì thay đổi các file cốt lõi của package để điều chỉnh các dòng này, bạn có thể ghi đè chúng bằng cách đặt các file trong thư mục `lang/vendor/{package}/{locale}`.

Vì vậy, ví dụ, nếu bạn cần ghi đè các chuỗi dịch tiếng Anh trong `messages.php` cho một package có tên `skyrim/hearthfire`, bạn nên đặt một file ngôn ngữ tại: `lang/vendor/hearthfire/en/messages.php`. Trong file này, bạn chỉ nên định nghĩa các chuỗi dịch bạn muốn ghi đè. Bất kỳ chuỗi dịch nào bạn không ghi đè vẫn sẽ được tải từ các file ngôn ngữ gốc của package.

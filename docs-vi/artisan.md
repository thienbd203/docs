# Artisan Console

- [Giới thiệu](#introduction)
    - [Tinker (REPL)](#tinker)
- [Viết Lệnh](#writing-commands)
    - [Tạo Lệnh](#generating-commands)
    - [Cấu Trúc Lệnh](#command-structure)
    - [Lệnh Closure](#closure-commands)
    - [Lệnh Có Thể Cách Ly](#isolatable-commands)
- [Định Nghĩa Kỳ Vọng Đầu Vào](#defining-input-expectations)
    - [Đối Số](#arguments)
    - [Tùy Chọn](#options)
    - [Mảng Đầu Vào](#input-arrays)
    - [Mô Tả Đầu Vào](#input-descriptions)
    - [Nhắc Nhở Khi Thiếu Đầu Vào](#prompting-for-missing-input)
- [I/O Lệnh](#command-io)
    - [Lấy Đầu Vào](#retrieving-input)
    - [Nhắc Nhở Đầu Vào](#prompting-for-input)
    - [Ghi Đầu Ra](#writing-output)
- [Đăng Ký Lệnh](#registering-commands)
- [Thực Thi Lệnh Theo Chương Trình](#programmatically-executing-commands)
    - [Gọi Lệnh Từ Các Lệnh Khác](#calling-commands-from-other-commands)
- [Xử Lý Tín Hiệu](#signal-handling)
- [Tùy Chỉnh Stub](#stub-customization)
- [Sự Kiện](#events)

<a name="introduction"></a>
## Giới thiệu

Artisan là giao diện dòng lệnh được bao gồm với Laravel. Artisan tồn tại ở thư mục gốc của ứng dụng của bạn dưới dạng script `artisan` và cung cấp một số lệnh hữu ích có thể hỗ trợ bạn trong khi xây dựng ứng dụng. Để xem danh sách tất cả các lệnh Artisan có sẵn, bạn có thể sử dụng lệnh `list`:

```shell
php artisan list
```

Mỗi lệnh cũng bao gồm màn hình "help" hiển thị và mô tả các đối số và tùy chọn có sẵn của lệnh. Để xem màn hình help, hãy thêm `help` trước tên lệnh:

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail) làm môi trường phát triển cục bộ của mình, hãy nhớ sử dụng dòng lệnh `sail` để gọi các lệnh Artisan. Sail sẽ thực thi các lệnh Artisan của bạn trong các container Docker của ứng dụng:

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

[Laravel Tinker](https://github.com/laravel/tinker) là một REPL mạnh mẽ cho framework Laravel, được hỗ trợ bởi gói [PsySH](https://github.com/bobthecow/psysh).

<a name="installation"></a>
#### Cài đặt

Tất cả các ứng dụng Laravel đều bao gồm Tinker theo mặc định. Tuy nhiên, bạn có thể cài đặt Tinker bằng Composer nếu bạn đã xóa nó khỏi ứng dụng của mình:

```shell
composer require laravel/tinker
```

> [!NOTE]
> Bạn đang tìm kiếm hot reloading, chỉnh sửa code nhiều dòng và autocompletion khi tương tác với ứng dụng Laravel của mình? Hãy xem [Tinkerwell](https://tinkerwell.app)!

<a name="usage"></a>
#### Sử dụng

Tinker cho phép bạn tương tác với toàn bộ ứng dụng Laravel của mình trên dòng lệnh, bao gồm các Eloquent model, job, sự kiện và nhiều hơn nữa. Để vào môi trường Tinker, hãy chạy lệnh Artisan `tinker`:

```shell
php artisan tinker
```

Bạn có thể xuất bản file cấu hình của Tinker bằng lệnh `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> [!WARNING]
> Hàm trợ giúp `dispatch` và phương thức `dispatch` trên class `Dispatchable` phụ thuộc vào garbage collection để đặt job vào hàng đợi. Do đó, khi sử dụng Tinker, bạn nên sử dụng `Bus::dispatch` hoặc `Queue::push` để dispatch job.

<a name="command-allow-list"></a>
#### Danh Sách Cho Phép Lệnh

Tinker sử dụng danh sách "allow" để xác định các lệnh Artisan nào được phép chạy trong shell của nó. Theo mặc định, bạn có thể chạy các lệnh `clear-compiled`, `down`, `env`, `inspire`, `migrate`, `migrate:install`, `up` và `optimize`. Nếu bạn muốn cho phép nhiều lệnh hơn, bạn có thể thêm chúng vào mảng `commands` trong file cấu hình `tinker.php` của mình:

```php
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],
```

<a name="classes-that-should-not-be-aliased"></a>
#### Các Class Không Nên Được Đặt Bí Danh

Thông thường, Tinker tự động đặt bí danh cho các class khi bạn tương tác với chúng trong Tinker. Tuy nhiên, bạn có thể không bao giờ muốn đặt bí danh cho một số class. Bạn có thể thực hiện điều này bằng cách liệt kê các class trong mảng `dont_alias` của file cấu hình `tinker.php` của mình:

```php
'dont_alias' => [
    App\Models\User::class,
],
```

<a name="writing-commands"></a>
## Viết Lệnh

Ngoài các lệnh được cung cấp với Artisan, bạn có thể xây dựng các lệnh tùy chỉnh của riêng mình. Các lệnh thường được lưu trữ trong thư mục `app/Console/Commands`; tuy nhiên, bạn có thể tự do chọn vị trí lưu trữ của mình miễn là bạn hướng dẫn Laravel [quét các thư mục khác để tìm lệnh Artisan](#registering-commands).

<a name="generating-commands"></a>
### Tạo Lệnh

Để tạo một lệnh mới, bạn có thể sử dụng lệnh Artisan `make:command`. Lệnh này sẽ tạo một class lệnh mới trong thư mục `app/Console/Commands`. Đừng lo lắng nếu thư mục này không tồn tại trong ứng dụng của bạn - nó sẽ được tạo lần đầu tiên bạn chạy lệnh Artisan `make:command`:

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### Cấu Trúc Lệnh

Sau khi tạo lệnh của mình, bạn nên định nghĩa chữ ký và mô tả của lệnh bằng các thuộc tính `Signature` và `Description`. Thuộc tính `Signature` cũng cho phép bạn định nghĩa [kỳ vọng đầu vào của lệnh](#defining-input-expectations). Phương thức `handle` sẽ được gọi khi lệnh của bạn được thực thi. Bạn có thể đặt logic lệnh của mình trong phương thức này.

Hãy xem một ví dụ về lệnh. Lưu ý rằng chúng ta có thể yêu cầu bất kỳ dependency nào chúng ta cần thông qua phương thức `handle` của lệnh. [service container](/docs/{{version}}/container) của Laravel sẽ tự động inject tất cả các dependency được type-hint trong chữ ký của phương thức này:

```php
<?php

namespace App\Console\Commands;

use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Console\Attributes\Description;
use Illuminate\Console\Attributes\Signature;
use Illuminate\Console\Command;

#[Signature('mail:send {user}')]
#[Description('Send a marketing email to a user')]
class SendEmails extends Command
{
    /**
     * Execute the console command.
     */
    public function handle(DripEmailer $drip): void
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```

> [!NOTE]
> Để tái sử dụng code tốt hơn, thực hành tốt là giữ các lệnh console nhẹ nhàng và để chúng ủy quyền cho các dịch vụ ứng dụng để hoàn thành nhiệm vụ của chúng. Trong ví dụ trên, lưu ý rằng chúng ta inject một class dịch vụ để thực hiện "công việc nặng" khi gửi email.

<a name="exit-codes"></a>
#### Mã Thoát

Nếu không có gì được trả về từ phương thức `handle` và lệnh thực thi thành công, lệnh sẽ thoát với mã thoát `0`, cho biết thành công. Tuy nhiên, phương thức `handle` có thể tùy ý trả về một số nguyên để chỉ định thủ công mã thoát của lệnh:

```php
$this->error('Something went wrong.');

return 1;
```

Nếu bạn muốn "thất bại" lệnh từ bất kỳ phương thức nào trong lệnh, bạn có thể sử dụng phương thức `fail`. Phương thức `fail` sẽ ngay lập tức chấm dứt thực thi của lệnh và trả về mã thoát `1`:

```php
$this->fail('Something went wrong.');
```

<a name="closure-commands"></a>
### Lệnh Closure

Lệnh dựa trên closure cung cấp một giải pháp thay thế cho việc định nghĩa các lệnh console dưới dạng class. Cũng giống như route closure là giải pháp thay thế cho controller, hãy coi lệnh closure là giải pháp thay thế cho class lệnh.

Mặc dù file `routes/console.php` không định nghĩa các route HTTP, nó định nghĩa các điểm nhập console (route) vào ứng dụng của bạn. Trong file này, bạn có thể định nghĩa tất cả các lệnh console dựa trên closure của mình bằng phương thức `Artisan::command`. Phương thức `command` chấp nhận hai đối số: [chữ ký lệnh](#defining-input-expectations) và một closure nhận các đối số và tùy chọn của lệnh:

```php
Artisan::command('mail:send {user}', function (string $user) {
    $this->info("Sending email to: {$user}!");
});
```

Closure được liên kết với instance lệnh cơ bản, vì vậy bạn có quyền truy cập đầy đủ vào tất cả các phương thức trợ giúp mà bạn thường có thể truy cập trên một class lệnh đầy đủ.

<a name="type-hinting-dependencies"></a>
#### Type-Hinting Dependencies

Ngoài việc nhận các đối số và tùy chọn của lệnh, lệnh closure cũng có thể type-hint các dependency bổ sung mà bạn muốn được giải quyết từ [service container](/docs/{{version}}/container):

```php
use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Support\Facades\Artisan;

Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
    $drip->send(User::find($user));
});
```

<a name="closure-command-descriptions"></a>
#### Mô Tả Lệnh Closure

Khi định nghĩa một lệnh dựa trên closure, bạn có thể sử dụng phương thức `purpose` để thêm mô tả cho lệnh. Mô tả này sẽ được hiển thị khi bạn chạy các lệnh `php artisan list` hoặc `php artisan help`:

```php
Artisan::command('mail:send {user}', function (string $user) {
    // ...
})->purpose('Send a marketing email to a user');
```

<a name="isolatable-commands"></a>
### Lệnh Có Thể Cách Ly

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng driver cache `memcached`, `redis`, `dynamodb`, `database`, `file` hoặc `array` làm driver cache mặc định của ứng dụng. Ngoài ra, tất cả các server phải giao tiếp với cùng một server cache trung tâm.

Đôi khi bạn có thể muốn đảm bảo rằng chỉ có một instance của một lệnh có thể chạy tại một thời điểm. Để thực hiện điều này, bạn có thể triển khai interface `Illuminate\Contracts\Console\Isolatable` trên class lệnh của mình:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;

class SendEmails extends Command implements Isolatable
{
    // ...
}
```

Khi bạn đánh dấu một lệnh là `Isolatable`, Laravel tự động làm cho tùy chọn `--isolated` có sẵn cho lệnh mà không cần định nghĩa rõ ràng nó trong các tùy chọn của lệnh. Khi lệnh được gọi với tùy chọn đó, Laravel sẽ đảm bảo rằng không có instance nào khác của lệnh đó đang chạy. Laravel thực hiện điều này bằng cách cố gắng lấy một khóa nguyên tử bằng driver cache mặc định của ứng dụng. Nếu các instance khác của lệnh đang chạy, lệnh sẽ không thực thi; tuy nhiên, lệnh vẫn sẽ thoát với mã trạng thái thoát thành công:

```shell
php artisan mail:send 1 --isolated
```

Nếu bạn muốn chỉ định mã trạng thái thoát mà lệnh nên trả về nếu nó không thể thực thi, bạn có thể cung cấp mã trạng thái mong muốn thông qua tùy chọn `isolated`:

```shell
php artisan mail:send 1 --isolated=12
```

<a name="lock-id"></a>
#### ID Khóa

Theo mặc định, Laravel sẽ sử dụng tên của lệnh để tạo khóa chuỗi được sử dụng để lấy khóa nguyên tử trong cache của ứng dụng. Tuy nhiên, bạn có thể tùy chỉnh khóa này bằng cách định nghĩa một phương thức `isolatableId` trên class lệnh Artisan của mình, cho phép bạn tích hợp các đối số hoặc tùy chọn của lệnh vào khóa:

```php
/**
 * Get the isolatable ID for the command.
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

<a name="lock-expiration-time"></a>
#### Thời Gian Hết Hạn Khóa

Theo mặc định, các khóa cách ly hết hạn sau khi lệnh hoàn thành. Hoặc, nếu lệnh bị gián đoạn và không thể hoàn thành, khóa sẽ hết hạn sau một giờ. Tuy nhiên, bạn có thể điều chỉnh thời gian hết hạn khóa bằng cách định nghĩa một phương thức `isolationLockExpiresAt` trên lệnh của mình:

```php
use DateTimeInterface;
use DateInterval;

/**
 * Determine when an isolation lock expires for the command.
 */
public function isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->plus(minutes: 5);
}
```

<a name="defining-input-expectations"></a>
## Định Nghĩa Kỳ Vọng Đầu Vào

Khi viết các lệnh console, việc thu thập đầu vào từ người dùng thông qua đối số hoặc tùy chọn là rất phổ biến. Laravel làm cho việc định nghĩa đầu vào bạn mong đợi từ người dùng trở nên rất thuận tiện bằng cách sử dụng thuộc tính `signature` trên các lệnh của bạn. Thuộc tính `signature` cho phép bạn định nghĩa tên, đối số và tùy chọn cho lệnh trong một cú pháp giống như route duy nhất và biểu đạt.

<a name="arguments"></a>
### Đối Số

Tất cả các đối số và tùy chọn do người dùng cung cấp đều được bao bọc trong dấu ngoặc nhọn. Trong ví dụ sau, lệnh định nghĩa một đối số bắt buộc: `user`:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user}';
```

Bạn cũng có thể làm cho đối số trở nên tùy chọn hoặc định nghĩa các giá trị mặc định cho đối số:

```php
// Optional argument...
'mail:send {user?}'

// Optional argument with default value...
'mail:send {user=foo}'
```

<a name="options"></a>
### Tùy Chọn

Tùy chọn, giống như đối số, là một dạng khác của đầu vào người dùng. Tùy chọn được thêm tiền tố bằng hai dấu gạch ngang (`--`) khi chúng được cung cấp qua dòng lệnh. Có hai loại tùy chọn: những tùy chọn nhận giá trị và những tùy chọn không nhận. Các tùy chọn không nhận giá trị đóng vai trò như một "switch" boolean. Hãy xem một ví dụ về loại tùy chọn này:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue}';
```

Trong ví dụ này, switch `--queue` có thể được chỉ định khi gọi lệnh Artisan. Nếu switch `--queue` được truyền, giá trị của tùy chọn sẽ là `true`. Nếu không, giá trị sẽ là `false`:

```shell
php artisan mail:send 1 --queue
```

<a name="options-with-values"></a>
#### Tùy Chọn Có Giá Trị

Tiếp theo, hãy xem một tùy chọn mong đợi một giá trị. Nếu người dùng phải chỉ định một giá trị cho một tùy chọn, bạn nên thêm hậu tố tên tùy chọn bằng dấu `=`:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue=}';
```

Trong ví dụ này, người dùng có thể truyền một giá trị cho tùy chọn như sau. Nếu tùy chọn không được chỉ định khi gọi lệnh, giá trị của nó sẽ là `null`:

```shell
php artisan mail:send 1 --queue=default
```

Bạn có thể gán giá trị mặc định cho tùy chọn bằng cách chỉ định giá trị mặc định sau tên tùy chọn. Nếu không có giá trị tùy chọn nào được người dùng truyền, giá trị mặc định sẽ được sử dụng:

```php
'mail:send {user} {--queue=default}'
```

<a name="option-shortcuts"></a>
#### Phím Tắt Tùy Chọn

Để gán một phím tắt khi định nghĩa một tùy chọn, bạn có thể chỉ định nó trước tên tùy chọn và sử dụng ký tự `|` làm dấu phân tách để tách phím tắt khỏi tên tùy chọn đầy đủ:

```php
'mail:send {user} {--Q|queue=}'
```

Khi gọi lệnh trên terminal của bạn, các phím tắt tùy chọn nên được thêm tiền tố bằng một dấu gạch ngang duy nhất và không nên bao gồm ký tự `=` khi chỉ định một giá trị cho tùy chọn:

```shell
php artisan mail:send 1 -Qdefault
```

<a name="input-arrays"></a>
### Mảng Đầu Vào

Nếu bạn muốn định nghĩa đối số hoặc tùy chọn để mong đợi nhiều giá trị đầu vào, bạn có thể sử dụng ký tự `*`. Đầu tiên, hãy xem một ví dụ chỉ định một đối số như vậy:

```php
'mail:send {user*}'
```

Khi chạy lệnh này, các đối số `user` có thể được truyền theo thứ tự vào dòng lệnh. Ví dụ, lệnh sau sẽ đặt giá trị của `user` thành một mảng với `1` và `2` làm giá trị của nó:

```shell
php artisan mail:send 1 2
```

Ký tự `*` này có thể được kết hợp với định nghĩa đối số tùy chọn để cho phép không hoặc nhiều instance của một đối số:

```php
'mail:send {user?*}'
```

<a name="option-arrays"></a>
#### Mảng Tùy Chọn

Khi định nghĩa một tùy chọn mong đợi nhiều giá trị đầu vào, mỗi giá trị tùy chọn được truyền cho lệnh nên được thêm tiền tố bằng tên tùy chọn:

```php
'mail:send {--id=*}'
```

Một lệnh như vậy có thể được gọi bằng cách truyền nhiều đối số `--id`:

```shell
php artisan mail:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### Mô Tả Đầu Vào

Bạn có thể gán mô tả cho các đối số và tùy chọn đầu vào bằng cách tách tên đối số khỏi mô tả bằng dấu hai chấm. Nếu bạn cần thêm một chút không gian để định nghĩa lệnh của mình, hãy thoải mái phân tách định nghĩa trên nhiều dòng:

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send
                        {user : The ID of the user}
                        {--queue : Whether the job should be queued}';
```

<a name="prompting-for-missing-input"></a>
### Nhắc Nhở Khi Thiếu Đầu Vào

Nếu lệnh của bạn chứa các đối số bắt buộc, người dùng sẽ nhận được thông báo lỗi khi chúng không được cung cấp. Ngoài ra, bạn có thể cấu hình lệnh của mình để tự động nhắc người dùng khi các đối số bắt buộc bị thiếu bằng cách triển khai interface `PromptsForMissingInput`:

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\PromptsForMissingInput;

class SendEmails extends Command implements PromptsForMissingInput
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    // ...
}
```

Nếu Laravel cần thu thập một đối số bắt buộc từ người dùng, nó sẽ tự động hỏi người dùng về đối số đó bằng cách thông minh đặt câu hỏi bằng cách sử dụng tên hoặc mô tả đối số. Nếu bạn muốn tùy chỉnh câu hỏi được sử dụng để thu thập đối số bắt buộc, bạn có thể triển khai phương thức `promptForMissingArgumentsUsing`, trả về một mảng các câu hỏi được khóa theo tên đối số:

```php
/**
 * Prompt for missing input arguments using the returned questions.
 *
 * @return array<string, string>
 */
protected function promptForMissingArgumentsUsing(): array
{
    return [
        'user' => 'Which user ID should receive the mail?',
    ];
}
```

Bạn cũng có thể cung cấp văn bản placeholder bằng cách sử dụng một tuple chứa câu hỏi và placeholder:

```php
return [
    'user' => ['Which user ID should receive the mail?', 'E.g. 123'],
];
```

Nếu bạn muốn kiểm soát hoàn toàn lời nhắc, bạn có thể cung cấp một closure nên nhắc người dùng và trả về câu trả lời của họ:

```php
use App\Models\User;
use function Laravel\Prompts\search;

// ...

return [
    'user' => fn () => search(
        label: 'Search for a user:',
        placeholder: 'E.g. Taylor Otwell',
        options: fn ($value) => strlen($value) > 0
            ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
            : []
    ),
];
```

> [!NOTE]
Tài liệu [Laravel Prompts](/docs/{{version}}/prompts) toàn diện bao gồm thêm thông tin về các lời nhắc có sẵn và cách sử dụng của chúng.

Nếu bạn muốn nhắc người dùng chọn hoặc nhập [tùy chọn](#options), bạn có thể bao gồm các lời nhắc trong phương thức `handle` của lệnh. Tuy nhiên, nếu bạn chỉ muốn nhắc người dùng khi họ cũng đã được tự động nhắc về các đối số bị thiếu, thì bạn có thể triển khai phương thức `afterPromptingForMissingArguments`:

```php
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use function Laravel\Prompts\confirm;

// ...

/**
 * Perform actions after the user was prompted for missing arguments.
 */
protected function afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output): void
{
    $input->setOption('queue', confirm(
        label: 'Would you like to queue the mail?',
        default: $this->option('queue')
    ));
}
```

<a name="command-io"></a>
## I/O Lệnh

<a name="retrieving-input"></a>
### Lấy Đầu Vào

Trong khi lệnh của bạn đang thực thi, bạn có thể sẽ cần truy cập các giá trị cho các đối số và tùy chọn được chấp nhận bởi lệnh của bạn. Để làm điều này, bạn có thể sử dụng các phương thức `argument` và `option`. Nếu một đối số hoặc tùy chọn không tồn tại, `null` sẽ được trả về:

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $userId = $this->argument('user');
}
```

Nếu bạn cần lấy tất cả các đối số dưới dạng một `array`, hãy gọi phương thức `arguments`:

```php
$arguments = $this->arguments();
```

Các tùy chọn có thể được lấy dễ dàng như các đối số bằng phương thức `option`. Để lấy tất cả các tùy chọn dưới dạng một mảng, hãy gọi phương thức `options`:

```php
// Retrieve a specific option...
$queueName = $this->option('queue');

// Retrieve all options as an array...
$options = $this->options();
```

<a name="prompting-for-input"></a>
### Nhắc Nhở Đầu Vào

> [!NOTE]
> [Laravel Prompts](/docs/{{version}}/prompts) là một gói PHP để thêm các biểu mẫu đẹp và thân thiện với người dùng vào các ứng dụng dòng lệnh của bạn, với các tính năng giống như trình duyệt bao gồm văn bản placeholder và xác thực.

Ngoài việc hiển thị đầu ra, bạn cũng có thể yêu cầu người dùng cung cấp đầu vào trong quá trình thực thi lệnh của bạn. Phương thức `ask` sẽ nhắc người dùng với câu hỏi đã cho, chấp nhận đầu vào của họ, và sau đó trả về đầu vào của người dùng trở lại lệnh của bạn:

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $name = $this->ask('What is your name?');

    // ...
}
```

Phương thức `ask` cũng chấp nhận một đối số thứ hai tùy chọn chỉ định giá trị mặc định nên được trả về nếu không có đầu vào người dùng nào được cung cấp:

```php
$name = $this->ask('What is your name?', 'Taylor');
```

Phương thức `secret` tương tự như `ask`, nhưng đầu vào của người dùng sẽ không hiển thị cho họ khi họ nhập vào console. Phương thức này hữu ích khi yêu cầu thông tin nhạy cảm như mật khẩu:

```php
$password = $this->secret('What is the password?');
```

<a name="asking-for-confirmation"></a>
#### Yêu Cầu Xác Nhận

Nếu bạn cần yêu cầu người dùng xác nhận "có hoặc không" đơn giản, bạn có thể sử dụng phương thức `confirm`. Theo mặc định, phương thức này sẽ trả về `false`. Tuy nhiên, nếu người dùng nhập `y` hoặc `yes` để trả lời lời nhắc, phương thức sẽ trả về `true`.

```php
if ($this->confirm('Do you wish to continue?')) {
    // ...
}
```

Nếu cần thiết, bạn có thể chỉ định rằng lời nhắc xác nhận nên trả về `true` theo mặc định bằng cách truyền `true` làm đối số thứ hai cho phương thức `confirm`:

```php
if ($this->confirm('Do you wish to continue?', true)) {
    // ...
}
```

<a name="auto-completion"></a>
#### Tự Động Hoàn Thành

Phương thức `anticipate` có thể được sử dụng để cung cấp tự động hoàn thành cho các lựa chọn có thể. Người dùng vẫn có thể cung cấp bất kỳ câu trả lời nào, bất kể các gợi ý tự động hoàn thành:

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

Ngoài ra, bạn có thể truyền một closure làm đối số thứ hai cho phương thức `anticipate`. Closure sẽ được gọi mỗi lần người dùng nhập một ký tự đầu vào. Closure nên chấp nhận một tham số chuỗi chứa đầu vào của người dùng cho đến nay và trả về một mảng các tùy chọn để tự động hoàn thành:

```php
use App\Models\Address;

$name = $this->anticipate('What is your address?', function (string $input) {
    return Address::whereLike('name', "{$input}%")
        ->limit(5)
        ->pluck('name')
        ->all();
});
```

<a name="multiple-choice-questions"></a>
#### Câu Hỏi Nhiều Lựa Chọn

Nếu bạn cần cung cấp cho người dùng một tập hợp các lựa chọn được định nghĩa trước khi hỏi một câu hỏi, bạn có thể sử dụng phương thức `choice`. Bạn có thể đặt chỉ mục mảng của giá trị mặc định sẽ được trả về nếu không có tùy chọn nào được chọn bằng cách chuyển chỉ mục làm đối số thứ ba cho phương thức:

```php
$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex
);
```

Ngoài ra, phương thức `choice` chấp nhận các đối số thứ tư và thứ năm tùy chọn để xác định số lần thử tối đa để chọn một phản hồi hợp lệ và liệu nhiều lựa chọn có được phép hay không:

```php
$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex,
    $maxAttempts = null,
    $allowMultipleSelections = false
);
```

<a name="writing-output"></a>
### Ghi Đầu Ra

Để gửi đầu ra đến console, bạn có thể sử dụng các phương thức `line`, `newLine`, `info`, `comment`, `question`, `warn`, `alert` và `error`. Mỗi phương thức này sẽ sử dụng màu ANSI phù hợp cho mục đích của chúng. Ví dụ, hãy hiển thị một số thông tin chung cho người dùng. Thông thường, phương thức `info` sẽ hiển thị trong console dưới dạng văn bản màu xanh lá cây:

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    // ...

    $this->info('The command was successful!');
}
```

Để hiển thị thông báo lỗi, hãy sử dụng phương thức `error`. Văn bản thông báo lỗi thường được hiển thị màu đỏ:

```php
$this->error('Something went wrong!');
```

Bạn có thể sử dụng phương thức `line` để hiển thị văn bản đơn giản, không có màu:

```php
$this->line('Display this on the screen');
```

Bạn có thể sử dụng phương thức `newLine` để hiển thị một dòng trống:

```php
// Write a single blank line...
$this->newLine();

// Write three blank lines...
$this->newLine(3);
```

<a name="tables"></a>
#### Bảng

Phương thức `table` giúp dễ dàng định dạng đúng nhiều hàng / cột dữ liệu. Tất cả những gì bạn cần làm là cung cấp tên cột và dữ liệu cho bảng và Laravel sẽ tự động tính toán chiều rộng và chiều cao phù hợp của bảng cho bạn:

```php
use App\Models\User;

$this->table(
    ['Name', 'Email'],
    User::all(['name', 'email'])->toArray()
);
```

<a name="progress-bars"></a>
#### Thanh Tiến Độ

Đối với các tác vụ chạy lâu, việc hiển thị thanh tiến độ thông báo cho người dùng mức độ hoàn thành của tác vụ có thể hữu ích. Sử dụng phương thức `withProgressBar`, Laravel sẽ hiển thị một thanh tiến độ và tăng tiến độ của nó cho mỗi lần lặp qua một giá trị có thể lặp lại:

```php
use App\Models\User;

$users = $this->withProgressBar(User::all(), function (User $user) {
    $this->performTask($user);
});
```

Đôi khi, bạn có thể cần kiểm soát thủ công nhiều hơn về cách thanh tiến độ được tăng. Đầu tiên, định nghĩa tổng số bước mà quá trình sẽ lặp qua. Sau đó, tăng thanh tiến độ sau khi xử lý từng mục:

```php
$users = App\Models\User::all();

$bar = $this->output->createProgressBar(count($users));

$bar->start();

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

> [!NOTE]
> Để biết thêm các tùy chọn nâng cao, hãy xem [tài liệu thành phần Thanh Tiến Độ Symfony](https://symfony.com/doc/current/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## Đăng Ký Lệnh

Theo mặc định, Laravel tự động đăng ký tất cả các lệnh trong thư mục `app/Console/Commands`. Tuy nhiên, bạn có thể hướng dẫn Laravel quét các thư mục khác để tìm lệnh Artisan bằng phương thức `withCommands` trong file `bootstrap/app.php` của ứng dụng:

```php
->withCommands([
    __DIR__.'/../app/Domain/Orders/Commands',
])
```

Nếu cần thiết, bạn cũng có thể đăng ký thủ công các lệnh bằng cách cung cấp tên class của lệnh cho phương thức `withCommands`:

```php
use App\Domain\Orders\Commands\SendEmails;

->withCommands([
    SendEmails::class,
])
```

Khi Artisan khởi động, tất cả các lệnh trong ứng dụng của bạn sẽ được giải quyết bởi [service container](/docs/{{version}}/container) và đăng ký với Artisan.

<a name="programmatically-executing-commands"></a>
## Thực Thi Lệnh Theo Chương Trình

Đôi khi bạn có thể muốn thực thi một lệnh Artisan bên ngoài CLI. Ví dụ, bạn có thể muốn thực thi một lệnh Artisan từ một route hoặc controller. Bạn có thể sử dụng phương thức `call` trên facade `Artisan` để thực hiện điều này. Phương thức `call` chấp nhận tên chữ ký hoặc tên class của lệnh làm đối số đầu tiên và một mảng tham số lệnh làm đối số thứ hai. Mã thoát sẽ được trả về:

```php
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Route;

Route::post('/user/{user}/mail', function (string $user) {
    $exitCode = Artisan::call('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);

    // ...
});
```

Ngoài ra, bạn có thể chuyển toàn bộ lệnh Artisan cho phương thức `call` dưới dạng chuỗi:

```php
Artisan::call('mail:send 1 --queue=default');
```

<a name="passing-array-values"></a>
#### Truyền Giá Trị Mảng

Nếu lệnh của bạn định nghĩa một tùy chọn chấp nhận một mảng, bạn có thể truyền một mảng giá trị cho tùy chọn đó:

```php
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Route;

Route::post('/mail', function () {
    $exitCode = Artisan::call('mail:send', [
        '--id' => [5, 13]
    ]);
});
```

<a name="passing-boolean-values"></a>
#### Truyền Giá Trị Boolean

Nếu bạn cần chỉ định giá trị của một tùy chọn không chấp nhận giá trị chuỗi, chẳng hạn như cờ `--force` trên lệnh `migrate:refresh`, bạn nên truyền `true` hoặc `false` làm giá trị của tùy chọn:

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

<a name="queueing-artisan-commands"></a>
#### Đặt Lệnh Artisan Vào Hàng Đợi

Sử dụng phương thức `queue` trên facade `Artisan`, bạn thậm chí có thể đặt các lệnh Artisan vào hàng đợi để chúng được xử lý trong nền bởi [queue workers](/docs/{{version}}/queues) của bạn. Trước khi sử dụng phương thức này, hãy đảm bảo bạn đã cấu hình hàng đợi của mình và đang chạy một queue listener:

```php
use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\Route;

Route::post('/user/{user}/mail', function (string $user) {
    Artisan::queue('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);

    // ...
});
```

Sử dụng các phương thức `onConnection` và `onQueue`, bạn có thể chỉ định kết nối hoặc hàng đợi mà lệnh Artisan nên được dispatch đến:

```php
Artisan::queue('mail:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

<a name="calling-commands-from-other-commands"></a>
### Gọi Lệnh Từ Các Lệnh Khác

Đôi khi bạn có thể muốn gọi các lệnh khác từ một lệnh Artisan hiện có. Bạn có thể làm điều này bằng phương thức `call`. Phương thức `call` này chấp nhận tên lệnh và một mảng đối số / tùy chọn lệnh:

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $this->call('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    // ...
}
```

Nếu bạn muốn gọi một lệnh console khác và chặn tất cả đầu ra của nó, bạn có thể sử dụng phương thức `callSilently`. Phương thức `callSilently` có cùng chữ ký với phương thức `call`:

```php
$this->callSilently('mail:send', [
    'user' => 1, '--queue' => 'default'
]);
```

<a name="signal-handling"></a>
## Xử Lý Tín Hiệu

Như bạn có thể biết, các hệ điều hành cho phép gửi tín hiệu đến các quá trình đang chạy. Ví dụ, tín hiệu `SIGTERM` là cách hệ điều hành yêu cầu một chương trình chấm dứt một cách êm đẹp. Nếu bạn muốn lắng nghe các tín hiệu trong các lệnh console Artisan của mình và thực thi code khi chúng xảy ra, bạn có thể sử dụng phương thức `trap`:

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);

    while ($this->shouldKeepRunning) {
        // ...
    }
}
```

Để lắng nghe nhiều tín hiệu cùng một lúc, bạn có thể cung cấp một mảng tín hiệu cho phương thức `trap`:

```php
$this->trap([SIGTERM, SIGQUIT], function (int $signal) {
    $this->shouldKeepRunning = false;

    dump($signal); // SIGTERM / SIGQUIT
});
```

<a name="stub-customization"></a>
## Tùy Chỉnh Stub

Các lệnh `make` của console Artisan được sử dụng để tạo nhiều loại class khác nhau, chẳng hạn như controller, job, migration và test. Các class này được tạo bằng cách sử dụng các file "stub" được điền với các giá trị dựa trên đầu vào của bạn. Tuy nhiên, bạn có thể muốn thực hiện các thay đổi nhỏ đối với các file được tạo bởi Artisan. Để thực hiện điều này, bạn có thể sử dụng lệnh `stub:publish` để xuất bản các stub phổ biến nhất đến ứng dụng của mình để bạn có thể tùy chỉnh chúng:

```shell
php artisan stub:publish
```

Các stub được xuất bản sẽ nằm trong thư mục `stubs` ở thư mục gốc của ứng dụng. Bất kỳ thay đổi nào bạn thực hiện đối với các stub này sẽ được phản ánh khi bạn tạo các class tương ứng của chúng bằng các lệnh `make` của Artisan.

<a name="events"></a>
## Sự Kiện

Artisan dispatch ba sự kiện khi chạy các lệnh: `Illuminate\Console\Events\ArtisanStarting`, `Illuminate\Console\Events\CommandStarting` và `Illuminate\Console\Events\CommandFinished`. Sự kiện `ArtisanStarting` được dispatch ngay lập tức khi Artisan bắt đầu chạy. Tiếp theo, sự kiện `CommandStarting` được dispatch ngay lập tức trước khi một lệnh chạy. Cuối cùng, sự kiện `CommandFinished` được dispatch khi một lệnh hoàn thành thực thi.

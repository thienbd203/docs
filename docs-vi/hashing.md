# Hashing

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Basic Usage](#basic-usage)
    - [Hashing Passwords](#hashing-passwords)
    - [Verifying That a Password Matches a Hash](#verifying-that-a-password-matches-a-hash)
    - [Determining if a Password Needs to be Rehashed](#determining-if-a-password-needs-to-be-rehashed)
- [Hash Algorithm Verification](#hash-algorithm-verification)

<a name="introduction"></a>
## Introduction

Laravel `Hash` [facade](/docs/{{version}}/facades) cung cấp hashing Bcrypt và Argon2 an toàn để lưu trữ password của người dùng. Nếu bạn đang sử dụng một trong các [Laravel application starter kits](/docs/{{version}}/starter-kits), Bcrypt sẽ được sử dụng mặc định cho đăng ký và authentication.

Bcrypt là lựa chọn tuyệt vời để hash password vì "work factor" của nó có thể điều chỉnh, có nghĩa là thời gian để tạo ra hash có thể tăng lên khi sức mạnh phần cứng tăng lên. Khi hash password, chậm là tốt. Thuật toán càng mất nhiều thời gian để hash password, thì người dùng độc hại càng mất nhiều thời gian để tạo ra "rainbow tables" của tất cả các giá trị hash chuỗi có thể có có thể được sử dụng trong các cuộc tấn công brute force chống lại ứng dụng.

<a name="configuration"></a>
## Configuration

Theo mặc định, Laravel sử dụng driver hashing `bcrypt` khi hash dữ liệu. Tuy nhiên, một số driver hashing khác cũng được hỗ trợ, bao gồm [argon](https://en.wikipedia.org/wiki/Argon2) và [argon2id](https://en.wikipedia.org/wiki/Argon2).

Bạn có thể chỉ định driver hashing của ứng dụng bằng cách sử dụng biến môi trường `HASH_DRIVER`. Tuy nhiên, nếu bạn muốn tùy chỉnh tất cả các tùy chọn driver hashing của Laravel, bạn nên publish file cấu hình `hashing` hoàn chỉnh bằng lệnh Artisan `config:publish`:

```shell
php artisan config:publish hashing
```

<a name="basic-usage"></a>
## Basic Usage

<a name="hashing-passwords"></a>
### Hashing Passwords

Bạn có thể hash một password bằng cách gọi method `make` trên `Hash` facade:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class PasswordController extends Controller
{
    /**
     * Update the password for the user.
     */
    public function update(Request $request): RedirectResponse
    {
        // Validate the new password length...

        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Adjusting The Bcrypt Work Factor

Nếu bạn đang sử dụng thuật toán Bcrypt, method `make` cho phép bạn quản lý work factor của thuật toán bằng tùy chọn `rounds`; tuy nhiên, work factor mặc định được quản lý bởi Laravel là chấp nhận được cho hầu hết các ứng dụng:

```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

<a name="adjusting-the-argon2-work-factor"></a>
#### Adjusting The Argon2 Work Factor

Nếu bạn đang sử dụng thuật toán Argon2, method `make` cho phép bạn quản lý work factor của thuật toán bằng các tùy chọn `memory`, `time`, và `threads`; tuy nhiên, các giá trị mặc định được quản lý bởi Laravel là chấp nhận được cho hầu hết các ứng dụng:

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

> [!NOTE]
> Để biết thêm thông tin về các tùy chọn này, vui lòng tham khảo [tài liệu PHP chính thức về Argon hashing](https://secure.php.net/manual/en/function.password-hash.php).

<a name="verifying-that-a-password-matches-a-hash"></a>
### Verifying That a Password Matches a Hash

Method `check` được cung cấp bởi `Hash` facade cho phép bạn xác minh rằng một chuỗi plain-text nhất định tương ứng với một hash nhất định:

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // The passwords match...
}
```

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### Determining if a Password Needs to be Rehashed

Method `needsRehash` được cung cấp bởi `Hash` facade cho phép bạn xác định xem work factor được sử dụng bởi hasher đã thay đổi kể từ khi password được hash hay chưa. Một số ứng dụng chọn thực hiện kiểm tra này trong quá trình authentication của ứng dụng:

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

<a name="hash-algorithm-verification"></a>
## Hash Algorithm Verification

Để ngăn chặn việc thao túng thuật toán hash, method `Hash::check` của Laravel sẽ trước tiên xác minh rằng hash đã cho được tạo bằng thuật toán hashing được chọn của ứng dụng. Nếu các thuật toán khác nhau, một exception `RuntimeException` sẽ được ném ra.

Đây là hành vi mong đợi cho hầu hết các ứng dụng, nơi thuật toán hashing không được mong đợi thay đổi và các thuật toán khác nhau có thể là dấu hiệu của một cuộc tấn công độc hại. Tuy nhiên, nếu bạn cần hỗ trợ nhiều thuật toán hashing trong ứng dụng của mình, chẳng hạn như khi di chuyển từ thuật toán này sang thuật toán khác, bạn có thể tắt xác minh thuật toán hash bằng cách đặt biến môi trường `HASH_VERIFY` thành `false`:

```ini
HASH_VERIFY=false
```

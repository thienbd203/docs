# Encryption

- [Giới thiệu](#introduction)
- [Cấu hình](#configuration)
    - [Gracefully Rotating Encryption Keys](#gracefully-rotating-encryption-keys)
- [Sử dụng Encrypter](#using-the-encrypter)

<a name="introduction"></a>
## Giới thiệu

Các dịch vụ encryption của Laravel cung cấp một giao diện đơn giản, thuận tiện để mã hóa và giải mã văn bản thông qua OpenSSL bằng cách sử dụng encryption AES-256 và AES-128. Tất cả các giá trị được mã hóa của Laravel đều được ký bằng một message authentication code (MAC) để giá trị cơ bản của chúng không thể được sửa đổi hoặc can thiệp sau khi được mã hóa.

<a name="configuration"></a>
## Cấu hình

Trước khi sử dụng encrypter của Laravel, bạn phải đặt tùy chọn cấu hình `key` trong file cấu hình `config/app.php` của bạn. Giá trị cấu hình này được điều khiển bởi biến môi trường `APP_KEY`. Bạn nên sử dụng lệnh `php artisan key:generate` để tạo giá trị cho biến này vì lệnh `key:generate` sẽ sử dụng generator random bytes an toàn của PHP để xây dựng một key an toàn về mặt mật mã cho ứng dụng của bạn. Thông thường, giá trị của biến môi trường `APP_KEY` sẽ được tạo cho bạn trong quá trình [cài đặt Laravel](/docs/{{version}}/installation).

<a name="gracefully-rotating-encryption-keys"></a>
### Gracefully Rotating Encryption Keys

Nếu bạn thay đổi encryption key của ứng dụng, tất cả các user session đã xác thực sẽ bị đăng xuất khỏi ứng dụng của bạn. Điều này là do mọi cookie, bao gồm cả session cookies, đều được mã hóa bởi Laravel. Ngoài ra, sẽ không còn có thể giải mã bất kỳ dữ liệu nào đã được mã hóa với encryption key trước đó của bạn.

Để giảm thiểu vấn đề này, Laravel cho phép bạn liệt kê các encryption key trước đó của bạn trong biến môi trường `APP_PREVIOUS_KEYS` của ứng dụng. Biến này có thể chứa một danh sách được phân tách bằng dấu phẩy của tất cả các encryption key trước đó của bạn:

```ini
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```

Khi bạn đặt biến môi trường này, Laravel sẽ luôn sử dụng encryption key "hiện tại" khi mã hóa các giá trị. Tuy nhiên, khi giải mã các giá trị, Laravel sẽ trước hết thử key hiện tại, và nếu giải mã thất bại bằng cách sử dụng key hiện tại, Laravel sẽ thử tất cả các key trước đó cho đến khi một trong các key có thể giải mã giá trị.

Cách tiếp cận này đối với giải mã graceful cho phép người dùng tiếp tục sử dụng ứng dụng của bạn không bị gián đoạn ngay cả khi encryption key của bạn được rotate.

<a name="using-the-encrypter"></a>
## Sử dụng Encrypter

<a name="encrypting-a-value"></a>
#### Mã hóa một Giá trị

Bạn có thể mã hóa một giá trị bằng cách sử dụng phương thức `encryptString` được cung cấp bởi facade `Crypt`. Tất cả các giá trị được mã hóa đều được mã hóa bằng cách sử dụng OpenSSL và cipher AES-256-CBC. Hơn nữa, tất cả các giá trị được mã hóa đều được ký bằng một message authentication code (MAC). Message authentication code tích hợp sẽ ngăn chặn việc giải mã bất kỳ giá trị nào đã bị can thiệp bởi người dùng độc hại:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Crypt;

class DigitalOceanTokenController extends Controller
{
    /**
     * Store a DigitalOcean API token for the user.
     */
    public function store(Request $request): RedirectResponse
    {
        $request->user()->fill([
            'token' => Crypt::encryptString($request->token),
        ])->save();

        return redirect('/secrets');
    }
}
```

<a name="decrypting-a-value"></a>
#### Giải mã một Giá trị

Bạn có thể giải mã các giá trị bằng cách sử dụng phương thức `decryptString` được cung cấp bởi facade `Crypt`. Nếu giá trị không thể được giải mã đúng cách, chẳng hạn như khi message authentication code không hợp lệ, một `Illuminate\Contracts\Encryption\DecryptException` sẽ được throw:

```php
use Illuminate\Contracts\Encryption\DecryptException;
use Illuminate\Support\Facades\Crypt;

try {
    $decrypted = Crypt::decryptString($encryptedValue);
} catch (DecryptException $e) {
    // ...
}
```

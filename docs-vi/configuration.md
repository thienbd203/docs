# Configuration

- [Introduction](#introduction)
- [Environment Configuration](#environment-configuration)
    - [Environment Variable Types](#environment-variable-types)
    - [Retrieving Environment Configuration](#retrieving-environment-configuration)
    - [Determining the Current Environment](#determining-the-current-environment)
    - [Encrypting Environment Files](#encrypting-environment-files)
- [Accessing Configuration Values](#accessing-configuration-values)
- [Configuration Caching](#configuration-caching)
- [Configuration Publishing](#configuration-publishing)
- [Debug Mode](#debug-mode)
- [Maintenance Mode](#maintenance-mode)

<a name="introduction"></a>
## Introduction

Tất cả các file cấu hình của Laravel framework đều được lưu trữ trong thư mục `config`. Mỗi tùy chọn đều có tài liệu, vì vậy hãy thoải mái xem qua các file và làm quen với các tùy chọn có sẵn cho bạn.

Các file cấu hình này cho phép bạn cấu hình những thứ như thông tin kết nối database, thông tin mail server, cũng như các giá trị cấu hình cốt lõi khác như URL ứng dụng và encryption key.

<a name="the-about-command"></a>
#### The `about` Command

Laravel có thể hiển thị tổng quan về cấu hình, drivers và môi trường của ứng dụng thông qua command `about` của Artisan.

```shell
php artisan about
```

Nếu bạn chỉ quan tâm đến một phần cụ thể của đầu ra tổng quan ứng dụng, bạn có thể lọc cho phần đó bằng tùy chọn `--only`:

```shell
php artisan about --only=environment
```

Hoặc, để khám phá chi tiết các giá trị của một file cấu hình cụ thể, bạn có thể sử dụng command `config:show` của Artisan:

```shell
php artisan config:show database
```

<a name="environment-configuration"></a>
## Environment Configuration

Thường rất hữu ích khi có các giá trị cấu hình khác nhau dựa trên môi trường nơi ứng dụng đang chạy. Ví dụ, bạn có thể muốn sử dụng cache driver khác ở local so với production server.

Để làm điều này trở nên dễ dàng, Laravel sử dụng thư viện PHP [DotEnv](https://github.com/vlucas/phpdotenv). Trong một cài đặt Laravel mới, thư mục gốc của ứng dụng sẽ chứa file `.env.example` định nghĩa nhiều biến môi trường phổ biến. Trong quá trình cài đặt Laravel, file này sẽ tự động được sao chép thành `.env`.

File `.env` mặc định của Laravel chứa một số giá trị cấu hình phổ biến có thể khác nhau tùy thuộc vào việc ứng dụng của bạn đang chạy local hay trên production web server. Các giá trị này sau đó được đọc bởi các file cấu hình trong thư mục `config` bằng function `env` của Laravel.

Nếu bạn đang phát triển với một team, bạn có thể muốn tiếp tục bao gồm và cập nhật file `.env.example` với ứng dụng của mình. Bằng cách đặt các giá trị placeholder trong file cấu hình mẫu, các developer khác trong team của bạn có thể thấy rõ các biến môi trường nào cần thiết để chạy ứng dụng của bạn.

> [!NOTE]
> Bất kỳ biến nào trong file `.env` của bạn đều có thể được ghi đè bởi các biến môi trường bên ngoài như biến môi trường ở cấp server hoặc cấp hệ thống.

<a name="environment-file-security"></a>
#### Environment File Security

File `.env` của bạn không nên được commit vào source control của ứng dụng, vì mỗi developer / server sử dụng ứng dụng của bạn có thể yêu cầu cấu hình môi trường khác nhau. Hơn nữa, điều này sẽ là rủi ro bảo mật trong trường hợp kẻ xâm nhập truy cập vào repository source control của bạn, vì bất kỳ thông tin xác thực nhạy cảm nào đều sẽ bị lộ.

Tuy nhiên, có thể mã hóa file môi trường của bạn bằng [environment encryption](#encrypting-environment-files) tích hợp sẵn của Laravel. Các file môi trường đã mã hóa có thể được đặt trong source control một cách an toàn.

<a name="additional-environment-files"></a>
#### Additional Environment Files

Trước khi tải các biến môi trường của ứng dụng, Laravel xác định xem biến môi trường `APP_ENV` đã được cung cấp bên ngoài hay chưa hoặc xem argument `--env` CLI đã được chỉ định hay chưa. Nếu có, Laravel sẽ cố gắng tải file `.env.[APP_ENV]` nếu nó tồn tại. Nếu nó không tồn tại, file `.env` mặc định sẽ được tải.

<a name="environment-variable-types"></a>
### Environment Variable Types

Tất cả các biến trong file `.env` của bạn thường được phân tích cú pháp như chuỗi, vì vậy một số giá trị reserved đã được tạo để cho phép bạn trả về phạm vi rộng hơn các loại từ function `env()`:

<div class="overflow-auto">

| `.env` Value | `env()` Value |
| ------------ | ------------- |
| true         | (bool) true   |
| (true)       | (bool) true   |
| false        | (bool) false  |
| (false)      | (bool) false  |
| empty        | (string) ''   |
| (empty)      | (string) ''   |
| null         | (null) null   |
| (null)       | (null) null   |

</div>

Nếu bạn cần định nghĩa một biến môi trường với giá trị chứa khoảng trắng, bạn có thể làm điều đó bằng cách đóng gói giá trị trong dấu ngoặc kép:

```ini
APP_NAME="My Application"
```

<a name="retrieving-environment-configuration"></a>
### Retrieving Environment Configuration

Tất cả các biến được liệt kê trong file `.env` sẽ được tải vào PHP super-global `$_ENV` khi ứng dụng của bạn nhận được request. Tuy nhiên, bạn có thể sử dụng function `env` để truy xuất giá trị từ các biến này trong file cấu hình của bạn. Thực tế, nếu bạn xem xét các file cấu hình của Laravel, bạn sẽ thấy nhiều tùy chọn đã sử dụng function này:

```php
'debug' => (bool) env('APP_DEBUG', false),
```

Giá trị thứ hai được truyền cho function `env` là "giá trị mặc định". Giá trị này sẽ được trả về nếu không có biến môi trường nào tồn tại cho key đã cho.

<a name="determining-the-current-environment"></a>
### Determining the Current Environment

Môi trường ứng dụng hiện tại được xác định thông qua biến `APP_ENV` từ file `.env` của bạn. Bạn có thể truy cập giá trị này thông qua method `environment` trên [facade](/docs/{{version}}/facades) `App`:

```php
use Illuminate\Support\Facades\App;

$environment = App::environment();
```

Bạn cũng có thể truyền arguments cho method `environment` để xác định xem môi trường có khớp với một giá trị đã cho hay không. Method sẽ trả về `true` nếu môi trường khớp với bất kỳ giá trị nào đã cho:

```php
if (App::environment('local')) {
    // The environment is local
}

if (App::environment(['local', 'staging'])) {
    // The environment is either local OR staging...
}
```

> [!NOTE]
> Việc phát hiện môi trường ứng dụng hiện tại có thể được ghi đè bằng cách định nghĩa biến môi trường `APP_ENV` ở cấp server.

<a name="encrypting-environment-files"></a>
### Encrypting Environment Files

Các file môi trường chưa mã hóa không bao giờ nên được lưu trữ trong source control. Tuy nhiên, Laravel cho phép bạn mã hóa các file môi trường của mình để chúng có thể được thêm vào source control một cách an toàn với phần còn lại của ứng dụng.

<a name="encryption"></a>
#### Encryption

Để mã hóa một file môi trường, bạn có thể sử dụng command `env:encrypt`:

```shell
php artisan env:encrypt
```

Chạy command `env:encrypt` sẽ mã hóa file `.env` của bạn và đặt nội dung đã mã hóa trong file `.env.encrypted`. Khóa giải mã được hiển thị trong đầu ra của command và nên được lưu trữ trong trình quản lý mật khẩu an toàn. Nếu bạn muốn cung cấp khóa mã hóa của riêng mình, bạn có thể sử dụng tùy chọn `--key` khi gọi command:

```shell
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

> [!NOTE]
> Độ dài của khóa được cung cấp nên khớp với độ dài khóa cần thiết bởi cipher mã hóa đang được sử dụng. Theo mặc định, Laravel sẽ sử dụng cipher `AES-256-CBC` yêu cầu khóa 32 ký tự. Bạn có thể tự do sử dụng bất kỳ cipher nào được hỗ trợ bởi [encrypter](/docs/{{version}}/encryption) của Laravel bằng cách truyền tùy chọn `--cipher` khi gọi command.

Nếu ứng dụng của bạn có nhiều file môi trường, chẳng hạn như `.env` và `.env.staging`, bạn có thể chỉ định file môi trường nên được mã hóa bằng cách cung cấp tên môi trường thông qua tùy chọn `--env`:

```shell
php artisan env:encrypt --env=staging
```

<a name="readable-variable-names"></a>
#### Readable Variable Names

Khi mã hóa file môi trường của bạn, bạn có thể sử dụng tùy chọn `--readable` để giữ lại tên biến có thể nhìn thấy được trong khi mã hóa các giá trị của chúng:

```shell
php artisan env:encrypt --readable
```

Điều này sẽ tạo ra một file đã mã hóa với định dạng sau:

```ini
APP_NAME=eyJpdiI6...
APP_ENV=eyJpdiI6...
APP_KEY=eyJpdiI6...
APP_DEBUG=eyJpdiI6...
APP_URL=eyJpdiI6...
```

Sử dụng định dạng có thể đọc được cho phép bạn xem các biến môi trường nào tồn tại mà không cần lộ dữ liệu nhạy cảm. Nó cũng giúp việc review pull request dễ dàng hơn nhiều vì bạn có thể xem các biến nào đã được thêm, xóa hoặc đổi tên mà không cần giải mã file.

Khi giải mã các file môi trường, Laravel tự động phát hiện định dạng nào đã được sử dụng, vì vậy không cần tùy chọn bổ sung cho command `env:decrypt`.

> [!NOTE]
> Khi sử dụng tùy chọn `--readable`, các comment và dòng trống từ file môi trường gốc sẽ không được bao gồm trong đầu ra đã mã hóa.

<a name="decryption"></a>
#### Decryption

Để giải mã một file môi trường, bạn có thể sử dụng command `env:decrypt`. Command này yêu cầu khóa giải mã, mà Laravel sẽ truy xuất từ biến môi trường `LARAVEL_ENV_ENCRYPTION_KEY`:

```shell
php artisan env:decrypt
```

Hoặc, khóa có thể được cung cấp trực tiếp cho command thông qua tùy chọn `--key`:

```shell
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

Khi command `env:decrypt` được gọi, Laravel sẽ giải mã nội dung của file `.env.encrypted` và đặt nội dung đã giải mã trong file `.env`.

Tùy chọn `--cipher` có thể được cung cấp cho command `env:decrypt` để sử dụng cipher mã hóa tùy chỉnh:

```shell
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

Nếu ứng dụng của bạn có nhiều file môi trường, chẳng hạn như `.env` và `.env.staging`, bạn có thể chỉ định file môi trường nên được giải mã bằng cách cung cấp tên môi trường thông qua tùy chọn `--env`:

```shell
php artisan env:decrypt --env=staging
```

Để ghi đè một file môi trường hiện có, bạn có thể cung cấp tùy chọn `--force` cho command `env:decrypt`:

```shell
php artisan env:decrypt --force
```

<a name="accessing-configuration-values"></a>
## Accessing Configuration Values

Bạn có thể dễ dàng truy cập các giá trị cấu hình của mình bằng facade `Config` hoặc function `config` toàn cục từ bất kỳ đâu trong ứng dụng của bạn. Các giá trị cấu hình có thể được truy cập bằng cú pháp "dot", bao gồm tên file và tùy chọn bạn muốn truy cập. Một giá trị mặc định cũng có thể được chỉ định và sẽ được trả về nếu tùy chọn cấu hình không tồn tại:

```php
use Illuminate\Support\Facades\Config;

$value = Config::get('app.timezone');

$value = config('app.timezone');

// Retrieve a default value if the configuration value does not exist...
$value = config('app.timezone', 'Asia/Seoul');
```

Để đặt giá trị cấu hình tại runtime, bạn có thể gọi method `set` của facade `Config` hoặc truyền một mảng cho function `config`:

```php
Config::set('app.timezone', 'America/Chicago');

config(['app.timezone' => 'America/Chicago']);
```

Để hỗ trợ phân tích tĩnh, facade `Config` cũng cung cấp các method truy xuất cấu hình có kiểu. Nếu giá trị cấu hình được truy xuất không khớp với kiểu mong đợi, một exception sẽ được ném:

```php
Config::string('config-key');
Config::integer('config-key');
Config::float('config-key');
Config::boolean('config-key');
Config::array('config-key');
Config::collection('config-key');
```

<a name="configuration-caching"></a>
## Configuration Caching

Để tăng tốc ứng dụng của bạn, bạn nên cache tất cả các file cấu hình thành một file duy nhất bằng command `config:cache` của Artisan. Điều này sẽ kết hợp tất cả các tùy chọn cấu hình cho ứng dụng của bạn thành một file duy nhất có thể được tải nhanh bởi framework.

Bạn thường nên chạy command `php artisan config:cache` như một phần của quá trình deployment production. Command không nên chạy trong quá trình phát triển local vì các tùy chọn cấu hình sẽ thường xuyên cần được thay đổi trong quá trình phát triển ứng dụng.

Khi cấu hình đã được cache, file `.env` của ứng dụng sẽ không được tải bởi framework trong các request hoặc commands Artisan; do đó, function `env` sẽ chỉ trả về các biến môi trường ở cấp hệ thống bên ngoài.

Vì lý do này, bạn nên đảm bảo rằng bạn chỉ gọi function `env` từ trong các file cấu hình (`config`) của ứng dụng. Bạn có thể xem nhiều ví dụ về điều này bằng cách xem xét các file cấu hình mặc định của Laravel. Các giá trị cấu hình có thể được truy cập từ bất kỳ đâu trong ứng dụng của bạn bằng function `config` [được mô tả ở trên](#accessing-configuration-values).

Command `config:clear` có thể được sử dụng để xóa cấu hình đã cache:

```shell
php artisan config:clear
```

> [!WARNING]
> Nếu bạn thực thi command `config:cache` trong quá trình deployment, bạn nên đảm bảo rằng bạn chỉ gọi function `env` từ trong các file cấu hình của mình. Khi cấu hình đã được cache, file `.env` sẽ không được tải; do đó, function `env` sẽ chỉ trả về các biến môi trường ở cấp hệ thống bên ngoài.

<a name="configuration-publishing"></a>
## Configuration Publishing

Hầu hết các file cấu hình của Laravel đã được xuất bản trong thư mục `config` của ứng dụng; tuy nhiên, một số file cấu hình như `cors.php` và `view.php` không được xuất bản theo mặc định, vì hầu hết các ứng dụng sẽ không bao giờ cần sửa đổi chúng.

Tuy nhiên, bạn có thể sử dụng command `config:publish` của Artisan để xuất bản bất kỳ file cấu hình nào không được xuất bản theo mặc định:

```shell
php artisan config:publish

php artisan config:publish --all
```

<a name="debug-mode"></a>
## Debug Mode

Tùy chọn `debug` trong file cấu hình `config/app.php` của bạn xác định bao nhiêu thông tin về lỗi thực sự được hiển thị cho người dùng. Theo mặc định, tùy chọn này được đặt để tôn trọng giá trị của biến môi trường `APP_DEBUG`, được lưu trữ trong file `.env` của bạn.

> [!WARNING]
> Đối với phát triển local, bạn nên đặt biến môi trường `APP_DEBUG` thành `true`. **Trong môi trường production của bạn, giá trị này luôn phải là `false`. Nếu biến được đặt thành `true` trong production, bạn có nguy cơ lộ các giá trị cấu hình nhạy cảm cho người dùng cuối của ứng dụng.**

<a name="maintenance-mode"></a>
## Maintenance Mode

Khi ứng dụng của bạn ở trong maintenance mode, một view tùy chỉnh sẽ được hiển thị cho tất cả các request vào ứng dụng của bạn. Điều này giúp "vô hiệu hóa" ứng dụng của bạn một cách dễ dàng trong khi nó đang cập nhật hoặc khi bạn đang thực hiện bảo trì. Một kiểm tra maintenance mode được bao gồm trong middleware stack mặc định cho ứng dụng của bạn. Nếu ứng dụng ở trong maintenance mode, một instance `Symfony\Component\HttpKernel\Exception\HttpException` sẽ được ném với mã trạng thái 503.

Để bật maintenance mode, thực thi command `down` của Artisan:

```shell
php artisan down
```

Nếu bạn muốn HTTP header `Refresh` được gửi với tất cả các response maintenance mode, bạn có thể cung cấp tùy chọn `refresh` khi gọi command `down`. Header `Refresh` sẽ hướng dẫn trình duyệt tự động làm mới trang sau số giây đã chỉ định:

```shell
php artisan down --refresh=15
```

Bạn cũng có thể cung cấp tùy chọn `retry` cho command `down`, sẽ được đặt làm giá trị của HTTP header `Retry-After`, mặc dù trình duyệt thường bỏ qua header này:

```shell
php artisan down --retry=60
```

<a name="bypassing-maintenance-mode"></a>
#### Bypassing Maintenance Mode

Để cho phép maintenance mode được bỏ qua bằng một secret token, bạn có thể sử dụng tùy chọn `secret` để chỉ định một token bỏ qua maintenance mode:

```shell
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

Sau khi đặt ứng dụng trong maintenance mode, bạn có thể điều hướng đến URL ứng dụng khớp với token này và Laravel sẽ phát hành một cookie bỏ qua maintenance mode cho trình duyệt của bạn:

```shell
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

Nếu bạn muốn Laravel tạo secret token cho bạn, bạn có thể sử dụng tùy chọn `with-secret`. Secret sẽ được hiển thị cho bạn khi ứng dụng ở trong maintenance mode:

```shell
php artisan down --with-secret
```

Khi truy cập route ẩn này, bạn sau đó sẽ được chuyển hướng đến route `/` của ứng dụng. Khi cookie đã được phát hành cho trình duyệt của bạn, bạn sẽ có thể duyệt ứng dụng bình thường như thể nó không ở trong maintenance mode.

> [!NOTE]
> Secret maintenance mode của bạn thường nên bao gồm các ký tự chữ số và, tùy chọn, dấu gạch ngang. Bạn nên tránh sử dụng các ký tự có ý nghĩa đặc biệt trong URL như `?` hoặc `&`.

<a name="maintenance-mode-on-multiple-servers"></a>
#### Maintenance Mode on Multiple Servers

Theo mặc định, Laravel xác định xem ứng dụng của bạn có ở trong maintenance mode hay không bằng hệ thống dựa trên file. Điều này có nghĩa là để kích hoạt maintenance mode, command `php artisan down` phải được thực thi trên mỗi server lưu trữ ứng dụng của bạn.

Ngoài ra, Laravel cung cấp phương pháp dựa trên cache để xử lý maintenance mode. Phương pháp này yêu cầu chạy command `php artisan down` chỉ trên một server. Để sử dụng phương pháp này, sửa đổi các biến maintenance mode trong file `.env` của ứng dụng. Bạn nên chọn cache `store` có thể truy cập được bởi tất cả các server của bạn. Điều này đảm bảo trạng thái maintenance mode được duy trì nhất quán trên mọi server:

```ini
APP_MAINTENANCE_DRIVER=cache
APP_MAINTENANCE_STORE=database
```

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### Pre-Rendering the Maintenance Mode View

Nếu bạn sử dụng command `php artisan down` trong quá trình deployment, người dùng của bạn vẫn có thể thỉnh thoảng gặp lỗi nếu họ truy cập ứng dụng trong khi các Composer dependencies hoặc các thành phần cơ sở hạ tầng khác đang cập nhật. Điều này xảy ra vì một phần đáng kể của Laravel framework phải khởi động để xác định ứng dụng của bạn ở trong maintenance mode và hiển thị maintenance mode view bằng công cụ tạo mẫu.

Vì lý do này, Laravel cho phép bạn pre-render một maintenance mode view sẽ được trả về ngay từ đầu của request cycle. View này được render trước khi bất kỳ dependencies nào của ứng dụng đã tải. Bạn có thể pre-render một template lựa chọn của mình bằng tùy chọn `render` của command `down`:

```shell
php artisan down --render="errors::503"
```

<a name="redirecting-maintenance-mode-requests"></a>
#### Redirecting Maintenance Mode Requests

Trong khi ở maintenance mode, Laravel sẽ hiển thị maintenance mode view cho tất cả các URL ứng dụng mà người dùng cố gắng truy cập. Nếu bạn muốn, bạn có thể hướng dẫn Laravel chuyển hướng tất cả các request đến một URL cụ thể. Điều này có thể được thực hiện bằng tùy chọn `redirect`. Ví dụ, bạn có thể muốn chuyển hướng tất cả các request đến URI `/`:

```shell
php artisan down --redirect=/
```

<a name="disabling-maintenance-mode"></a>
#### Disabling Maintenance Mode

Để tắt maintenance mode, sử dụng command `up`:

```shell
php artisan up
```

> [!NOTE]
> Bạn có thể tùy chỉnh template maintenance mode mặc định bằng cách định nghĩa template của riêng mình tại `resources/views/errors/503.blade.php`.

<a name="maintenance-mode-queues"></a>
#### Maintenance Mode and Queues

Trong khi ứng dụng của bạn ở trong maintenance mode, không có [queued jobs](/docs/{{version}}/queues) nào sẽ được xử lý. Các jobs sẽ tiếp tục được xử lý bình thường khi ứng dụng ra khỏi maintenance mode.

<a name="alternatives-to-maintenance-mode"></a>
#### Alternatives to Maintenance Mode

Vì maintenance mode yêu cầu ứng dụng của bạn có vài giây downtime, hãy cân nhắc chạy ứng dụng của bạn trên một nền tảng được quản lý hoàn toàn như [Laravel Cloud](https://cloud.laravel.com) để thực hiện deployment zero-downtime với Laravel.

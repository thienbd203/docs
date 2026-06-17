# File Storage

- [Introduction](#introduction)
- [Configuration](#configuration)
  - [The Local Driver](#the-local-driver)
  - [The Public Disk](#the-public-disk)
  - [Driver Prerequisites](#driver-prerequisites)
  - [Scoped and Read-Only Filesystems](#scoped-and-read-only-filesystems)
  - [Amazon S3 Compatible Filesystems](#amazon-s3-compatible-filesystems)
- [Obtaining Disk Instances](#obtaining-disk-instances)
  - [On-Demand Disks](#on-demand-disks)
- [Retrieving Files](#retrieving-files)
  - [Downloading Files](#downloading-files)
  - [File URLs](#file-urls)
  - [Temporary URLs](#temporary-urls)
  - [File Metadata](#file-metadata)
- [Storing Files](#storing-files)
  - [Prepending and Appending To Files](#prepending-appending-to-files)
  - [Copying and Moving Files](#copying-moving-files)
  - [Automatic Streaming](#automatic-streaming)
  - [File Uploads](#file-uploads)
  - [File Visibility](#file-visibility)
- [Deleting Files](#deleting-files)
- [Directories](#directories)
- [Testing](#testing)
- [Custom Filesystems](#custom-filesystems)

<a name="introduction"></a>

## Introduction

Laravel cung cấp một filesystem abstraction mạnh mẽ nhờ vào [Flysystem](https://github.com/thephpleague/flysystem) PHP package tuyệt vời của Frank de Jonge. Laravel Flysystem integration cung cấp các drivers đơn giản để làm việc với local filesystems, SFTP, và Amazon S3. Hơn nữa, việc chuyển đổi giữa các tùy chọn lưu trữ này giữa local development machine và production server cực kỳ đơn giản vì API vẫn giữ nguyên cho mỗi hệ thống.

<a name="configuration"></a>

## Configuration

File cấu hình filesystem của Laravel nằm tại `config/filesystems.php`. Trong file này, bạn có thể cấu hình tất cả các filesystem "disks" của mình. Mỗi disk đại diện cho một storage driver và storage location cụ thể. Các cấu hình ví dụ cho mỗi driver được hỗ trợ được bao gồm trong file cấu hình để bạn có thể sửa đổi cấu hình để phản ánh preferences và credentials của mình.

Driver `local` tương tác với các files được lưu trữ locally trên server chạy ứng dụng Laravel, trong khi storage driver `sftp` được sử dụng cho FTP dựa trên SSH key. Driver `s3` được sử dụng để viết vào Amazon's S3 cloud storage service.

> [!NOTE]
> Bạn có thể cấu hình bao nhiêu disks tùy thích và thậm chí có thể có nhiều disks sử dụng cùng một driver.

<a name="the-local-driver"></a>

### The Local Driver

Khi sử dụng driver `local`, tất cả các file operations đều tương đối với directory `root` được định nghĩa trong file cấu hình `filesystems` của bạn. Theo mặc định, giá trị này được đặt thành directory `storage/app/private`. Do đó, method sau sẽ viết vào `storage/app/private/example.txt`:

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', 'Contents');
```

<a name="the-public-disk"></a>

### The Public Disk

Disk `public` được bao gồm trong file cấu hình `filesystems` của ứng dụng được dành cho các files sẽ được publicly accessible. Theo mặc định, disk `public` sử dụng driver `local` và lưu trữ các files của nó trong `storage/app/public`.

Nếu disk `public` của bạn sử dụng driver `local` và bạn muốn làm cho các files này có thể truy cập được từ web, bạn nên tạo một symbolic link từ source directory `storage/app/public` đến target directory `public/storage`:

Để tạo symbolic link, bạn có thể sử dụng command Artisan `storage:link`:

```shell
php artisan storage:link
```

Khi một file đã được lưu trữ và symbolic link đã được tạo, bạn có thể tạo một URL đến các files bằng cách sử dụng helper `asset`:

```php
echo asset('storage/file.txt');
```

Bạn có thể cấu hình các symbolic links bổ sung trong file cấu hình `filesystems` của mình. Mỗi link được cấu hình sẽ được tạo khi bạn chạy command `storage:link`:

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```

Command `storage:unlink` có thể được sử dụng để destroy các symbolic links được cấu hình của bạn:

```shell
php artisan storage:unlink
```

<a name="driver-prerequisites"></a>

### Driver Prerequisites

<a name="s3-driver-configuration"></a>

#### S3 Driver Configuration

Trước khi sử dụng driver S3, bạn sẽ cần cài đặt Flysystem S3 package thông qua trình quản lý gói Composer:

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

Một mảng cấu hình disk S3 nằm trong file cấu hình `config/filesystems.php` của bạn. Thông thường, bạn nên cấu hình thông tin và credentials S3 của mình bằng cách sử dụng các biến môi trường sau được tham chiếu bởi file cấu hình `config/filesystems.php`:

```ini
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
AWS_USE_PATH_STYLE_ENDPOINT=false
```

Để thuận tiện, các biến môi trường này khớp với quy ước đặt tên được sử dụng bởi AWS CLI.

<a name="ftp-driver-configuration"></a>

#### FTP Driver Configuration

Trước khi sử dụng driver FTP, bạn sẽ cần cài đặt Flysystem FTP package thông qua trình quản lý gói Composer:

```shell
composer require league/flysystem-ftp "^3.0"
```

Laravel Flysystem integrations hoạt động rất tốt với FTP; tuy nhiên, một cấu hình mẫu không được bao gồm với file cấu hình mặc định `config/filesystems.php` của framework. Nếu bạn cần cấu hình một FTP filesystem, bạn có thể sử dụng ví dụ cấu hình dưới đây:

```php
'ftp' => [
    'driver' => 'ftp',
    'host' => env('FTP_HOST'),
    'username' => env('FTP_USERNAME'),
    'password' => env('FTP_PASSWORD'),

    // Optional FTP Settings...
    // 'port' => env('FTP_PORT', 21),
    // 'root' => env('FTP_ROOT'),
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],
```

<a name="sftp-driver-configuration"></a>

#### SFTP Driver Configuration

Trước khi sử dụng driver SFTP, bạn sẽ cần cài đặt Flysystem SFTP package thông qua trình quản lý gói Composer:

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Laravel Flysystem integrations hoạt động rất tốt với SFTP; tuy nhiên, một cấu hình mẫu không được bao gồm với file cấu hình mặc định `config/filesystems.php` của framework. Nếu bạn cần cấu hình một SFTP filesystem, bạn có thể sử dụng ví dụ cấu hình dưới đây:

```php
'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'),

    // Settings for basic authentication...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),

    // Settings for SSH key-based authentication with encryption password...
    'privateKey' => env('SFTP_PRIVATE_KEY'),
    'passphrase' => env('SFTP_PASSPHRASE'),

    // Settings for file / directory permissions...
    'visibility' => 'private', // `private` = 0600, `public` = 0644
    'directory_visibility' => 'private', // `private` = 0700, `public` = 0755

    // Optional SFTP Settings...
    // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
    // 'maxTries' => 4,
    // 'passphrase' => env('SFTP_PASSPHRASE'),
    // 'port' => env('SFTP_PORT', 22),
    // 'root' => env('SFTP_ROOT', ''),
    // 'timeout' => 30,
    // 'useAgent' => true,
],
```

<a name="scoped-and-read-only-filesystems"></a>

### Scoped and Read-Only Filesystems

Scoped disks cho phép bạn định nghĩa một filesystem nơi tất cả các paths được tự động prefix với một path prefix nhất định. Trước khi tạo một scoped filesystem disk, bạn sẽ cần cài đặt một Flysystem package bổ sung thông qua trình quản lý gói Composer:

```shell
composer require league/flysystem-path-prefixing "^3.0"
```

Bạn có thể tạo một instance path scoped của bất kỳ filesystem disk hiện có nào bằng cách định nghĩa một disk sử dụng driver `scoped`. Ví dụ, bạn có thể tạo một disk scope disk `s3` hiện có của bạn thành một path prefix cụ thể, và sau đó mọi file operation sử dụng scoped disk của bạn sẽ sử dụng prefix được chỉ định:

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

"Read-only" disks cho phép bạn tạo các filesystem disks không cho phép write operations. Trước khi sử dụng tùy chọn cấu hình `read-only`, bạn sẽ cần cài đặt một Flysystem package bổ sung thông qua trình quản lý gói Composer:

```shell
composer require league/flysystem-read-only "^3.0"
```

Tiếp theo, bạn có thể bao gồm tùy chọn cấu hình `read-only` trong một hoặc nhiều mảng cấu hình disk của mình:

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

<a name="amazon-s3-compatible-filesystems"></a>

### Amazon S3 Compatible Filesystems

Theo mặc định, file cấu hình `filesystems` của ứng dụng chứa một cấu hình disk cho disk `s3`. Ngoài việc sử dụng disk này để tương tác với [Amazon S3](https://aws.amazon.com/s3/), bạn có thể sử dụng nó để tương tác với bất kỳ S3-compatible file storage service nào như [RustFS](https://github.com/rustfs/rustfs), [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces/), [Vultr Object Storage](https://www.vultr.com/products/object-storage/), [Cloudflare R2](https://www.cloudflare.com/developer-platform/products/r2/), hoặc [Hetzner Cloud Storage](https://www.hetzner.com/storage/object-storage/).

Thông thường, sau khi cập nhật credentials của disk để khớp với credentials của service bạn đang định sử dụng, bạn chỉ cần cập nhật giá trị của tùy chọn cấu hình `endpoint`. Giá trị của tùy chọn này thường được định nghĩa qua biến môi trường `AWS_ENDPOINT`:

```php
'endpoint' => env('AWS_ENDPOINT', 'https://rustfs:9000'),
```

<a name="obtaining-disk-instances"></a>

## Obtaining Disk Instances

Facade `Storage` có thể được sử dụng để tương tác với bất kỳ disks được cấu hình nào của bạn. Ví dụ, bạn có thể sử dụng method `put` trên facade để lưu trữ một avatar trên default disk. Nếu bạn gọi các methods trên facade `Storage` mà không gọi method `disk` trước, method sẽ tự động được chuyển đến default disk:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $content);
```

Nếu ứng dụng của bạn tương tác với nhiều disks, bạn có thể sử dụng method `disk` trên facade `Storage` để làm việc với các files trên một disk cụ thể:

```php
Storage::disk('s3')->put('avatars/1', $content);
```

<a name="on-demand-disks"></a>

### On-Demand Disks

Đôi khi bạn có thể muốn tạo một disk tại runtime bằng cách sử dụng một cấu hình nhất định mà cấu hình đó thực sự không có trong file cấu hình `filesystems` của ứng dụng. Để thực hiện điều này, bạn có thể chuyển một mảng cấu hình cho method `build` của facade `Storage`:

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

<a name="retrieving-files"></a>

## Retrieving Files

Method `get` có thể được sử dụng để truy xuất nội dung của một file. Nội dung string raw của file sẽ được trả về bởi method. Hãy nhớ rằng, tất cả các file paths nên được chỉ định tương đối với location "root" của disk:

```php
$contents = Storage::get('file.jpg');
```

Nếu file bạn đang truy xuất chứa JSON, bạn có thể sử dụng method `json` để truy xuất file và decode nội dung của nó:

```php
$orders = Storage::json('orders.json');
```

Method `exists` có thể được sử dụng để xác định xem một file có tồn tại trên disk hay không:

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```

Method `missing` có thể được sử dụng để xác định xem một file có bị thiếu khỏi disk hay không:

```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```

<a name="downloading-files"></a>

### Downloading Files

Method `download` có thể được sử dụng để tạo một response buộc browser của người dùng tải xuống file tại path đã cho. Method `download` chấp nhận một filename làm đối số thứ hai cho method, sẽ xác định filename được nhìn thấy bởi người dùng tải xuống file. Cuối cùng, bạn có thể chuyển một mảng HTTP headers làm đối số thứ ba cho method:

```php
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```

<a name="file-urls"></a>

### File URLs

Bạn có thể sử dụng method `url` để lấy URL cho một file nhất định. Nếu bạn đang sử dụng driver `local`, điều này thường chỉ prepend `/storage` vào path đã cho và trả về một relative URL đến file. Nếu bạn đang sử dụng driver `s3`, fully qualified remote URL sẽ được trả về:

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

Khi sử dụng driver `local`, tất cả các files nên publicly accessible nên được đặt trong directory `storage/app/public`. Hơn nữa, bạn nên [create a symbolic link](#the-public-disk) tại `public/storage` trỏ đến directory `storage/app/public`.

> [!WARNING]
> Khi sử dụng driver `local`, giá trị trả về của `url` không được URL encode. Vì lý do này, chúng tôi khuyến nghị luôn lưu trữ các files của bạn bằng cách sử dụng các tên sẽ tạo ra các URLs hợp lệ.

<a name="url-host-customization"></a>

#### URL Host Customization

Nếu bạn muốn sửa đổi host cho các URLs được tạo bằng cách sử dụng facade `Storage`, bạn có thể thêm hoặc thay đổi tùy chọn `url` trong mảng cấu hình của disk:

```php
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
    'throw' => false,
],
```

<a name="temporary-urls"></a>

### Temporary URLs

Sử dụng method `temporaryUrl`, bạn có thể tạo các temporary URLs đến các files được lưu trữ bằng cách sử dụng các drivers `local` và `s3`. Method này chấp nhận một path và một instance `DateTime` chỉ định khi nào URL nên hết hạn:

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::temporaryUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

<a name="enabling-local-temporary-urls"></a>

#### Enabling Local Temporary URLs

Nếu bạn bắt đầu phát triển ứng dụng của mình trước khi hỗ trợ cho temporary URLs được giới thiệu đến driver `local`, bạn có thể cần bật local temporary URLs. Để làm như vậy, thêm tùy chọn `serve` vào mảng cấu hình disk `local` của bạn trong file cấu hình `config/filesystems.php`:

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app/private'),
    'serve' => true, // [tl! add]
    'throw' => false,
],
```

<a name="s3-request-parameters"></a>

#### S3 Request Parameters

Nếu bạn cần chỉ định các [S3 request parameters](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests) bổ sung, bạn có thể chuyển mảng các request parameters làm đối số thứ ba cho method `temporaryUrl`:

```php
$url = Storage::temporaryUrl(
    'file.jpg',
    now()->plus(minutes: 5),
    [
        'ResponseContentType' => 'application/octet-stream',
        'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
    ]
);
```

<a name="customizing-temporary-urls"></a>

#### Customizing Temporary URLs

Nếu bạn cần tùy chỉnh cách temporary URLs được tạo cho một storage disk cụ thể, bạn có thể sử dụng method `buildTemporaryUrlsUsing`. Ví dụ, điều này có thể hữu ích nếu bạn có một controller cho phép bạn tải xuống các files được lưu trữ qua một disk thường không hỗ trợ temporary URLs. Thông thường, method này nên được gọi từ method `boot` của một service provider:

```php
<?php

namespace App\Providers;

use DateTime;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Storage::disk('local')->buildTemporaryUrlsUsing(
            function (string $path, DateTime $expiration, array $options) {
                return URL::temporarySignedRoute(
                    'files.download',
                    $expiration,
                    array_merge($options, ['path' => $path])
                );
            }
        );
    }
}
```

<a name="temporary-upload-urls"></a>

#### Temporary Upload URLs

> [!WARNING]
> Khả năng tạo temporary upload URLs chỉ được hỗ trợ bởi các drivers `s3` và `local`.

Nếu bạn cần tạo một temporary URL có thể được sử dụng để tải lên một file trực tiếp từ client-side application của bạn, bạn có thể sử dụng method `temporaryUploadUrl`. Method này chấp nhận một path và một instance `DateTime` chỉ định khi nào URL nên hết hạn. Method `temporaryUploadUrl` trả về một associative array có thể được destructured thành upload URL và các headers nên được bao gồm với upload request:

```php
use Illuminate\Support\Facades\Storage;

['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

Method này chủ yếu hữu ích trong các môi trường serverless yêu cầu client-side application tải lên files trực tiếp đến một cloud storage system như Amazon S3.

<a name="file-metadata"></a>

### File Metadata

Ngoài việc đọc và viết files, Laravel cũng có thể cung cấp thông tin về chính các files. Ví dụ, method `size` có thể được sử dụng để lấy kích thước của một file tính bằng bytes:

```php
use Illuminate\Support\Facades\Storage;

$size = Storage::size('file.jpg');
```

Method `lastModified` trả về UNIX timestamp của lần cuối file được sửa đổi:

```php
$time = Storage::lastModified('file.jpg');
```

MIME type của một file nhất định có thể được lấy qua method `mimeType`:

```php
$mime = Storage::mimeType('file.jpg');
```

<a name="file-paths"></a>

#### File Paths

Bạn có thể sử dụng method `path` để lấy path cho một file nhất định. Nếu bạn đang sử dụng driver `local`, điều này sẽ trả về absolute path đến file. Nếu bạn đang sử dụng driver `s3`, method này sẽ trả về relative path đến file trong S3 bucket:

```php
use Illuminate\Support\Facades\Storage;

$path = Storage::path('file.jpg');
```

<a name="storing-files"></a>

## Storing Files

Method `put` có thể được sử dụng để lưu trữ nội dung file trên một disk. Bạn cũng có thể chuyển một PHP `resource` cho method `put`, sẽ sử dụng stream support cơ bản của Flysystem. Hãy nhớ rằng, tất cả các file paths nên được chỉ định tương đối với location "root" được cấu hình cho disk:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

<a name="failed-writes"></a>

#### Failed Writes

Nếu method `put` (hoặc các operations "write" khác) không thể viết file vào disk, `false` sẽ được trả về:

```php
if (! Storage::put('file.jpg', $contents)) {
    // The file could not be written to disk...
}
```

Nếu bạn muốn, bạn có thể định nghĩa tùy chọn `throw` trong mảng cấu hình filesystem disk của mình. Khi tùy chọn này được định nghĩa là `true`, các methods "write" như `put` sẽ ném một instance của `League\Flysystem\UnableToWriteFile` khi write operations fail:

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

<a name="prepending-appending-to-files"></a>

### Prepending and Appending To Files

Các methods `prepend` và `append` cho phép bạn viết vào đầu hoặc cuối một file:

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

<a name="copying-moving-files"></a>

### Copying and Moving Files

Method `copy` có thể được sử dụng để copy một file hiện có đến một location mới trên disk, trong khi method `move` có thể được sử dụng để đổi tên hoặc di chuyển một file hiện có đến một location mới:

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');
```

<a name="automatic-streaming"></a>

### Automatic Streaming

Streaming files đến storage cung cấp giảm sử dụng bộ nhớ đáng kể. Nếu bạn muốn Laravel tự động quản lý streaming một file nhất định đến storage location của bạn, bạn có thể sử dụng method `putFile` hoặc `putFileAs`. Method này chấp nhận một instance `Illuminate\Http\File` hoặc `Illuminate\Http\UploadedFile` và sẽ tự động stream file đến location mong muốn của bạn:

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// Automatically generate a unique ID for filename...
$path = Storage::putFile('photos', new File('/path/to/photo'));

// Manually specify a filename...
$path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

Có một số điều quan trọng cần lưu ý về method `putFile`. Lưu ý rằng chúng ta chỉ chỉ định một tên directory chứ không phải filename. Theo mặc định, method `putFile` sẽ tạo một unique ID để phục vụ như filename. Extension của file sẽ được xác định bằng cách kiểm tra MIME type của file. Path đến file sẽ được trả về bởi method `putFile` để bạn có thể lưu trữ path, bao gồm filename được tạo, trong database của bạn.

Các methods `putFile` và `putFileAs` cũng chấp nhận một đối số để chỉ định "visibility" của file được lưu trữ. Điều này đặc biệt hữu ích nếu bạn đang lưu trữ file trên một cloud disk như Amazon S3 và muốn file có thể truy cập được công khai qua các URLs được tạo:

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

<a name="file-uploads"></a>

### File Uploads

Trong các ứng dụng web, một trong các use-cases phổ biến nhất để lưu trữ files là lưu trữ các files được tải lên bởi người dùng như photos và documents. Laravel làm cho việc lưu trữ uploaded files rất dễ dàng bằng cách sử dụng method `store` trên một uploaded file instance. Gọi method `store` với path mà bạn muốn lưu trữ uploaded file:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserAvatarController extends Controller
{
    /**
     * Update the avatar for the user.
     */
    public function update(Request $request): string
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```

Có một số điều quan trọng cần lưu ý về ví dụ này. Lưu ý rằng chúng ta chỉ chỉ định một tên directory, không phải filename. Theo mặc định, method `store` sẽ tạo một unique ID để phục vụ như filename. Extension của file sẽ được xác định bằng cách kiểm tra MIME type của file. Path đến file sẽ được trả về bởi method `store` để bạn có thể lưu trữ path, bao gồm filename được tạo, trong database của bạn.

Bạn cũng có thể gọi method `putFile` trên facade `Storage` để thực hiện cùng một file storage operation như ví dụ trên:

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```

<a name="specifying-a-file-name"></a>

#### Specifying a File Name

Nếu bạn không muốn một filename được gán tự động cho file được lưu trữ của mình, bạn có thể sử dụng method `storeAs`, nhận path, filename, và (tùy chọn) disk làm các đối số của nó:

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

Bạn cũng có thể sử dụng method `putFileAs` trên facade `Storage`, sẽ thực hiện cùng một file storage operation như ví dụ trên:

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

> [!WARNING]
> Các ký tự unprintable và unicode không hợp lệ sẽ tự động bị xóa khỏi file paths. Do đó, bạn có thể muốn sanitize file paths của mình trước khi chuyển chúng đến các methods file storage của Laravel. File paths được normalize bằng cách sử dụng method `League\Flysystem\WhitespacePathNormalizer::normalizePath`.

<a name="specifying-a-disk"></a>

#### Specifying a Disk

Theo mặc định, method `store` của uploaded file này sẽ sử dụng default disk của bạn. Nếu bạn muốn chỉ định một disk khác, chuyển tên disk làm đối số thứ hai cho method `store`:

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

Nếu bạn đang sử dụng method `storeAs`, bạn có thể chuyển tên disk làm đối số thứ ba cho method:

```php
$path = $request->file('avatar')->storeAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="other-uploaded-file-information"></a>

#### Other Uploaded File Information

Nếu bạn muốn lấy tên gốc và extension của uploaded file, bạn có thể làm như vậy bằng cách sử dụng các methods `getClientOriginalName` và `getClientOriginalExtension`:

```php
$file = $request->file('avatar');

$name = $file->getClientOriginalName();
$extension = $file->getClientOriginalExtension();
```

Tuy nhiên, hãy nhớ rằng các methods `getClientOriginalName` và `getClientOriginalExtension` được coi là unsafe, vì tên file và extension có thể bị can thiệp bởi một người dùng độc hại. Vì lý do này, bạn thường nên ưu tiên các methods `hashName` và `extension` để lấy tên và extension cho một file upload nhất định:

```php
$file = $request->file('avatar');

$name = $file->hashName(); // Generate a unique, random name...
$extension = $file->extension(); // Determine the file's extension based on the file's MIME type...
```

<a name="file-visibility"></a>

### File Visibility

Trong Laravel Flysystem integration, "visibility" là một abstraction của file permissions trên nhiều platforms. Files có thể được khai báo `public` hoặc `private`. Khi một file được khai báo `public`, bạn đang chỉ định rằng file nên có thể truy cập được cho người khác nói chung. Ví dụ, khi sử dụng driver S3, bạn có thể truy xuất các URLs cho các files `public`.

Bạn có thể đặt visibility khi viết file thông qua method `put`:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents, 'public');
```

Nếu file đã được lưu trữ, visibility của nó có thể được truy xuất và đặt qua các methods `getVisibility` và `setVisibility`:

```php
$visibility = Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public');
```

Khi tương tác với uploaded files, bạn có thể sử dụng các methods `storePublicly` và `storePubliclyAs` để lưu trữ uploaded file với visibility `public`:

```php
$path = $request->file('avatar')->storePublicly('avatars', 's3');

$path = $request->file('avatar')->storePubliclyAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="local-files-and-visibility"></a>

#### Local Files and Visibility

Khi sử dụng driver `local`, visibility `public` [visibility](#file-visibility) chuyển thành permissions `0755` cho directories và permissions `0644` cho files. Bạn có thể sửa đổi các mappings permissions trong file cấu hình `filesystems` của ứng dụng:

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app'),
    'permissions' => [
        'file' => [
            'public' => 0644,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0755,
            'private' => 0700,
        ],
    ],
    'throw' => false,
],
```

<a name="deleting-files"></a>

## Deleting Files

Method `delete` chấp nhận một filename đơn lẻ hoặc một mảng các files để xóa:

```php
use Illuminate\Support\Facades\Storage;

Storage::delete('file.jpg');

Storage::delete(['file.jpg', 'file2.jpg']);
```

Nếu cần thiết, bạn có thể chỉ định disk mà file nên được xóa từ:

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('s3')->delete('path/file.jpg');
```

<a name="directories"></a>

## Directories

<a name="get-all-files-within-a-directory"></a>

#### Get All Files Within a Directory

Method `files` trả về một mảng tất cả các files trong một directory nhất định. Nếu bạn muốn truy xuất một danh sách tất cả các files trong một directory nhất định bao gồm subdirectories, bạn có thể sử dụng method `allFiles`:

```php
use Illuminate\Support\Facades\Storage;

$files = Storage::files($directory);

$files = Storage::allFiles($directory);
```

<a name="get-all-directories-within-a-directory"></a>

#### Get All Directories Within a Directory

Method `directories` trả về một mảng tất cả các directories trong một directory nhất định. Nếu bạn muốn truy xuất một danh sách tất cả các directories trong một directory nhất định bao gồm subdirectories, bạn có thể sử dụng method `allDirectories`:

```php
$directories = Storage::directories($directory);

$directories = Storage::allDirectories($directory);
```

<a name="create-a-directory"></a>

#### Create a Directory

Method `makeDirectory` sẽ tạo directory đã cho, bao gồm bất kỳ subdirectories cần thiết nào:

```php
Storage::makeDirectory($directory);
```

<a name="delete-a-directory"></a>

#### Delete a Directory

Cuối cùng, method `deleteDirectory` có thể được sử dụng để xóa một directory và tất cả các files của nó:

```php
Storage::deleteDirectory($directory);
```

<a name="testing"></a>

## Testing

Method `fake` của facade `Storage` cho phép bạn dễ dàng tạo một fake disk mà, kết hợp với các utilities tạo file của class `Illuminate\Http\UploadedFile`, đơn giản hóa đáng kể việc testing file uploads. Ví dụ:

```php
<?php

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('albums can be uploaded', function () {
    Storage::fake('photos');

    $response = $this->json('POST', '/photos', [
        UploadedFile::fake()->image('photo1.jpg'),
        UploadedFile::fake()->image('photo2.jpg')
    ]);

    // Assert one or more files were stored...
    Storage::disk('photos')->assertExists('photo1.jpg');
    Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

    // Assert one or more files were not stored...
    Storage::disk('photos')->assertMissing('missing.jpg');
    Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

    // Assert that the number of files in a given directory matches the expected count...
    Storage::disk('photos')->assertCount('/wallpapers', 2);

    // Assert that a given directory is empty...
    Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
});
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_albums_can_be_uploaded(): void
    {
        Storage::fake('photos');

        $response = $this->json('POST', '/photos', [
            UploadedFile::fake()->image('photo1.jpg'),
            UploadedFile::fake()->image('photo2.jpg')
        ]);

        // Assert one or more files were stored...
        Storage::disk('photos')->assertExists('photo1.jpg');
        Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

        // Assert one or more files were not stored...
        Storage::disk('photos')->assertMissing('missing.jpg');
        Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

        // Assert that the number of files in a given directory matches the expected count...
        Storage::disk('photos')->assertCount('/wallpapers', 2);

        // Assert that a given directory is empty...
        Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
    }
}
```

Theo mặc định, method `fake` sẽ xóa tất cả các files trong temporary directory của nó. Nếu bạn muốn giữ các files này, bạn có thể sử dụng method "persistentFake" thay thế. Để biết thêm thông tin về testing file uploads, bạn có thể tham khảo [thông tin về file uploads trong tài liệu HTTP testing](/docs/{{version}}/http-tests#testing-file-uploads).

> [!WARNING]
> Method `image` yêu cầu [GD extension](https://www.php.net/manual/en/book.image.php).

<a name="custom-filesystems"></a>

## Custom Filesystems

Laravel Flysystem integration cung cấp hỗ trợ cho một số "drivers" out of the box; tuy nhiên, Flysystem không giới hạn ở những cái này và có adapters cho nhiều storage systems khác. Bạn có thể tạo một custom driver nếu bạn muốn sử dụng một trong các adapters bổ sung này trong ứng dụng Laravel của mình.

Để định nghĩa một custom filesystem, bạn sẽ cần một Flysystem adapter. Hãy thêm một Dropbox adapter được duy trì bởi cộng đồng vào project của chúng ta:

```shell
composer require spatie/flysystem-dropbox
```

Tiếp theo, bạn có thể đăng ký driver trong method `boot` của một trong các [service providers](/docs/{{version}}/providers) của ứng dụng. Để thực hiện điều này, bạn nên sử dụng method `extend` của facade `Storage`:

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Foundation\Application;
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

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
        Storage::extend('dropbox', function (Application $app, array $config) {
            $adapter = new DropboxAdapter(new DropboxClient(
                $config['authorization_token']
            ));

            return new FilesystemAdapter(
                new Filesystem($adapter, $config),
                $adapter,
                $config
            );
        });
    }
}
```

Đối số đầu tiên của method `extend` là tên của driver và đối số thứ hai là một closure nhận các biến `$app` và `$config`. Closure phải trả về một instance của `Illuminate\Filesystem\FilesystemAdapter`. Biến `$config` chứa các giá trị được định nghĩa trong `config/filesystems.php` cho disk được chỉ định.

Khi bạn đã tạo và đăng ký service provider của extension, bạn có thể sử dụng driver `dropbox` trong file cấu hình `config/filesystems.php` của mình.

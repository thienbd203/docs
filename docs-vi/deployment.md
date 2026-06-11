# Deployment

- [Introduction](#introduction)
- [Server Requirements](#server-requirements)
- [Server Configuration](#server-configuration)
    - [Nginx](#nginx)
    - [FrankenPHP](#frankenphp)
    - [Directory Permissions](#directory-permissions)
- [Optimization](#optimization)
    - [Caching Configuration](#optimizing-configuration-loading)
    - [Caching Events](#caching-events)
    - [Caching Routes](#optimizing-route-loading)
    - [Caching Views](#optimizing-view-loading)
- [Reloading Services](#reloading-services)
- [Debug Mode](#debug-mode)
- [The Health Route](#the-health-route)
- [Deploying With Laravel Cloud or Forge](#deploying-with-cloud-or-forge)

<a name="introduction"></a>
## Introduction

Khi bạn sẵn sàng deploy ứng dụng Laravel của mình lên production, có một số điều quan trọng bạn có thể làm để đảm bảo ứng dụng của bạn chạy càng hiệu quả càng tốt. Trong tài liệu này, chúng tôi sẽ đề cập đến một số điểm khởi đầu tuyệt vời để đảm bảo ứng dụng Laravel của bạn được deploy đúng cách.

<a name="server-requirements"></a>
## Server Requirements

Framework Laravel có một số yêu cầu hệ thống. Bạn nên đảm bảo rằng web server của bạn có phiên bản PHP và các extension tối thiểu sau:

<div class="content-list" markdown="1">

- PHP >= 8.3
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

</div>

<a name="server-configuration"></a>
## Server Configuration

<a name="nginx"></a>
### Nginx

Nếu bạn đang deploy ứng dụng của mình lên một server chạy Nginx, bạn có thể sử dụng file cấu hình sau làm điểm khởi đầu để cấu hình web server của mình. Rất có thể, file này sẽ cần được tùy chỉnh tùy thuộc vào cấu hình server của bạn. **Nếu bạn muốn hỗ trợ quản lý server của mình, hãy cân nhắc sử dụng một nền tảng Laravel được quản lý hoàn toàn như [Laravel Cloud](https://cloud.laravel.com).**

Hãy đảm bảo, giống như cấu hình dưới đây, web server của bạn chuyển hướng tất cả các request đến file `public/index.php` của ứng dụng. Bạn không bao giờ nên cố gắng di chuyển file `index.php` đến thư mục gốc của dự án, vì phục vụ ứng dụng từ thư mục gốc dự án sẽ hiển thị nhiều file cấu hình nhạy cảm cho công chúng Internet:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev/) cũng có thể được sử dụng để phục vụ các ứng dụng Laravel của bạn. FrankenPHP là một application server PHP hiện đại được viết bằng Go. Để phục vụ một ứng dụng PHP Laravel bằng FrankenPHP, bạn có thể chỉ cần gọi lệnh `php-server` của nó:

```shell
frankenphp php-server -r public/
```

Để tận dụng các tính năng mạnh mẽ hơn được hỗ trợ bởi FrankenPHP, chẳng hạn như tích hợp [Laravel Octane](/docs/{{version}}/octane), HTTP/3, nén hiện đại, hoặc khả năng đóng gói các ứng dụng Laravel dưới dạng binaries độc lập, hãy tham khảo tài liệu [Laravel của FrankenPHP](https://frankenphp.dev/docs/laravel/).

<a name="directory-permissions"></a>
### Directory Permissions

Laravel sẽ cần ghi vào các thư mục `bootstrap/cache` và `storage`, vì vậy bạn nên đảm bảo chủ sở hữu quy trình web server có quyền ghi vào các thư mục này.

<a name="optimization"></a>
## Optimization

Khi deploy ứng dụng của bạn lên production, có nhiều file nên được cache, bao gồm cấu hình, events, routes, và views của bạn. Laravel cung cấp một lệnh Artisan `optimize` duy nhất, thuận tiện sẽ cache tất cả các file này. Lệnh này thường nên được gọi như một phần của quy trình deploy ứng dụng của bạn:

```shell
php artisan optimize
```

Phương thức `optimize:clear` có thể được sử dụng để xóa tất cả các file cache được tạo bởi lệnh `optimize` cũng như tất cả các keys trong driver cache mặc định:

```shell
php artisan optimize:clear
```

Trong tài liệu sau, chúng tôi sẽ thảo luận về từng lệnh tối ưu hóa chi tiết được thực hiện bởi lệnh `optimize`.

<a name="optimizing-configuration-loading"></a>
### Caching Configuration

Khi deploy ứng dụng của bạn lên production, bạn nên đảm bảo rằng bạn chạy lệnh Artisan `config:cache` trong quá trình deploy của mình:

```shell
php artisan config:cache
```

Lệnh này sẽ kết hợp tất cả các file cấu hình của Laravel thành một file cache duy nhất, giúp giảm đáng kể số lần framework phải truy cập filesystem khi tải các giá trị cấu hình của bạn.

> [!WARNING]
> Nếu bạn thực thi lệnh `config:cache` trong quá trình deploy của mình, bạn nên đảm bảo rằng bạn chỉ gọi hàm `env` từ trong các file cấu hình của mình. Sau khi cấu hình đã được cache, file `.env` sẽ không được tải và tất cả các cuộc gọi đến hàm `env` cho các biến `.env` sẽ trả về `null`.

<a name="caching-events"></a>
### Caching Events

Bạn nên cache các ánh xạ event-to listener được tự động khám phá của ứng dụng trong quá trình deploy của mình. Điều này có thể được thực hiện bằng cách gọi lệnh Artisan `event:cache` trong quá trình deploy:

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### Caching Routes

Nếu bạn đang xây dựng một ứng dụng lớn với nhiều routes, bạn nên đảm bảo rằng bạn đang chạy lệnh Artisan `route:cache` trong quá trình deploy của mình:

```shell
php artisan route:cache
```

Lệnh này giảm tất cả các đăng ký route của bạn thành một cuộc gọi phương thức duy nhất trong một file cache, cải thiện hiệu suất của đăng ký route khi đăng ký hàng trăm routes.

<a name="optimizing-view-loading"></a>
### Caching Views

Khi deploy ứng dụng của bạn lên production, bạn nên đảm bảo rằng bạn chạy lệnh Artisan `view:cache` trong quá trình deploy của mình:

```shell
php artisan view:cache
```

Lệnh này biên dịch trước tất cả các Blade views của bạn để chúng không được biên dịch theo yêu cầu, cải thiện hiệu suất của mỗi request trả về một view.

<a name="reloading-services"></a>
## Reloading Services

> [!NOTE]
> Khi deploy lên [Laravel Cloud](https://cloud.laravel.com), không cần thiết phải sử dụng lệnh `reload`, vì việc tải lại nhẹ nhàng tất cả các dịch vụ được xử lý tự động.

Sau khi deploy một phiên bản mới của ứng dụng của bạn, bất kỳ dịch vụ chạy dài nào như queue workers, Laravel Reverb, hoặc Laravel Octane nên được tải lại / khởi động lại để sử dụng code mới. Laravel cung cấp một lệnh Artisan `reload` duy nhất sẽ chấm dứt các dịch vụ này:

```shell
php artisan reload
```

Nếu bạn không sử dụng [Laravel Cloud](https://cloud.laravel.com), bạn nên cấu hình thủ công một process monitor có thể phát hiện khi các quy trình có thể tải lại của bạn thoát và tự động khởi động lại chúng.

<a name="debug-mode"></a>
## Debug Mode

Tùy chọn debug trong file cấu hình `config/app.php` của bạn xác định bao nhiêu thông tin về một lỗi thực sự được hiển thị cho người dùng. Theo mặc định, tùy chọn này được đặt để tôn trọng giá trị của biến môi trường `APP_DEBUG`, được lưu trữ trong file `.env` của ứng dụng của bạn.

> [!WARNING]
> **Trong môi trường production của bạn, giá trị này luôn phải là `false`. Nếu biến `APP_DEBUG` được đặt thành `true` trong production, bạn có nguy cơ hiển thị các giá trị cấu hình nhạy cảm cho người dùng cuối của ứng dụng.**

<a name="the-health-route"></a>
## The Health Route

Laravel bao gồm một route health check tích hợp có thể được sử dụng để giám sát trạng thái của ứng dụng của bạn. Trong production, route này có thể được sử dụng để báo cáo trạng thái của ứng dụng cho một uptime monitor, load balancer, hoặc hệ thống orchestration như Kubernetes.

Theo mặc định, route health check được phục vụ tại `/up` và sẽ trả về một phản hồi HTTP 200 nếu ứng dụng đã khởi động mà không có ngoại lệ. Nếu không, một phản hồi HTTP 500 sẽ được trả về. Bạn có thể cấu hình URI cho route này trong file `bootstrap/app` của ứng dụng của bạn:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up', // [tl! remove]
    health: '/status', // [tl! add]
)
```

Khi các yêu cầu HTTP được thực hiện đến route này, Laravel cũng sẽ dispatch một event `Illuminate\Foundation\Events\DiagnosingHealth`, cho phép bạn thực hiện các health checks bổ sung liên quan đến ứng dụng của bạn. Trong một [listener](/docs/{{version}}/events) cho event này, bạn có thể kiểm tra trạng thái database hoặc cache của ứng dụng của bạn. Nếu bạn phát hiện một vấn đề với ứng dụng của mình, bạn có thể chỉ cần throw một exception từ listener.

<a name="deploying-with-cloud-or-forge"></a>
## Deploying With Laravel Cloud or Forge

<a name="laravel-cloud"></a>
#### Laravel Cloud

Nếu bạn muốn một nền tảng deployment được quản lý hoàn toàn, tự động mở rộng được tinh chỉnh cho Laravel, hãy xem [Laravel Cloud](https://cloud.laravel.com). Laravel Cloud là một nền tảng deployment mạnh mẽ cho Laravel, cung cấp compute, databases, caches, và object storage được quản lý.

Khởi chạy ứng dụng Laravel của bạn trên Cloud và yêu thích sự đơn giản có thể mở rộng. Laravel Cloud được tinh chỉnh bởi các nhà sáng tạo của Laravel để hoạt động liền mạch với framework để bạn có thể tiếp tục viết các ứng dụng Laravel của mình chính xác như bạn đã quen.

<a name="laravel-forge"></a>
#### Laravel Forge

Nếu bạn thích quản lý các server của riêng mình nhưng không thoải mái khi cấu hình tất cả các dịch vụ khác nhau cần thiết để chạy một ứng dụng Laravel mạnh mẽ, [Laravel Forge](https://forge.laravel.com) là một nền tảng quản lý server VPS cho các ứng dụng Laravel.

Laravel Forge có thể tạo server trên nhiều nhà cung cấp hạ tầng như DigitalOcean, Linode, AWS, và nhiều hơn nữa. Ngoài ra, Forge cài đặt và quản lý tất cả các công cụ cần thiết để xây dựng các ứng dụng Laravel mạnh mẽ, chẳng hạn như Nginx, MySQL, Redis, Memcached, Beanstalk, và nhiều hơn nữa.

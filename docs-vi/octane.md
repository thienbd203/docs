# Laravel Octane

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Điều kiện tiên quyết của Server](#server-prerequisites)
    - [FrankenPHP](#frankenphp)
    - [RoadRunner](#roadrunner)
    - [Swoole](#swoole)
- [Phục vụ Ứng dụng của Bạn](#serving-your-application)
    - [Phục vụ Ứng dụng của Bạn qua HTTPS](#serving-your-application-via-https)
    - [Phục vụ Ứng dụng của Bạn qua Nginx](#serving-your-application-via-nginx)
    - [Theo dõi Thay đổi File](#watching-for-file-changes)
    - [Chỉ định Số lượng Worker](#specifying-the-worker-count)
    - [Chỉ định Số lượng Request Tối đa](#specifying-the-max-request-count)
    - [Chỉ định Thời gian Thực thi Tối đa](#specifying-the-max-execution-time)
    - [Tải lại các Worker](#reloading-the-workers)
    - [Dừng Server](#stopping-the-server)
- [Dependency Injection và Octane](#dependency-injection-and-octane)
    - [Container Injection](#container-injection)
    - [Request Injection](#request-injection)
    - [Configuration Repository Injection](#configuration-repository-injection)
- [Quản lý Memory Leaks](#managing-memory-leaks)
- [Concurrent Tasks](#concurrent-tasks)
- [Ticks và Intervals](#ticks-and-intervals)
- [Octane Cache](#the-octane-cache)
- [Tables](#tables)

<a name="introduction"></a>
## Giới thiệu

[Laravel Octane](https://github.com/laravel/octane) tăng cường hiệu suất của ứng dụng của bạn bằng cách phục vụ ứng dụng sử dụng các application server mạnh mẽ, bao gồm [FrankenPHP](https://frankenphp.dev/), [Open Swoole](https://openswoole.com/), [Swoole](https://github.com/swoole/swoole-src), và [RoadRunner](https://roadrunner.dev). Octane khởi động ứng dụng của bạn một lần, giữ nó trong memory, và sau đó xử lý các request với tốc độ siêu nhanh.

<a name="installation"></a>
## Cài đặt

Octane có thể được cài đặt thông qua Composer package manager:

```shell
composer require laravel/octane
```

Sau khi cài đặt Octane, bạn có thể thực thi command `octane:install` của Artisan, command này sẽ cài đặt file cấu hình của Octane vào ứng dụng của bạn:

```shell
php artisan octane:install
```

<a name="server-prerequisites"></a>
## Điều kiện tiên quyết của Server

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev) là một PHP application server, được viết bằng Go, hỗ trợ các tính năng web hiện đại như early hints, Brotli, và nén Zstandard. Khi bạn cài đặt Octane và chọn FrankenPHP làm server của bạn, Octane sẽ tự động tải xuống và cài đặt FrankenPHP binary cho bạn.

<a name="frankenphp-via-laravel-sail"></a>
#### FrankenPHP qua Laravel Sail

Nếu bạn dự định phát triển ứng dụng của mình sử dụng [Laravel Sail](/docs/{{version}}/sail), bạn nên chạy các command sau để cài đặt Octane và FrankenPHP:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane
```

Tiếp theo, bạn nên sử dụng command `octane:install` của Artisan để cài đặt FrankenPHP binary:

```shell
./vendor/bin/sail artisan octane:install --server=frankenphp
```

Cuối cùng, thêm một environment variable `SUPERVISOR_PHP_COMMAND` vào định nghĩa service `laravel.test` trong file `docker-compose.yml` của ứng dụng của bạn. Environment variable này sẽ chứa command mà Sail sẽ sử dụng để phục vụ ứng dụng của bạn sử dụng Octane thay vì PHP development server:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port='${APP_PORT:-80}'" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

Để bật HTTPS, HTTP/2, và HTTP/3, hãy áp dụng các thay đổi sau:

```yaml
services:
  laravel.test:
    ports:
        - '${APP_PORT:-80}:80'
        - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        - '443:443' # [tl! add]
        - '443:443/udp' # [tl! add]
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --host=localhost --port=443 --admin-port=2019 --https" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

Thông thường, bạn nên truy cập ứng dụng FrankenPHP Sail của mình qua `https://localhost`, vì việc sử dụng `https://127.0.0.1` yêu cầu cấu hình bổ sung và được [không khuyến khích](https://frankenphp.dev/docs/known-issues/#using-https127001-with-docker).

<a name="frankenphp-via-docker"></a>
#### FrankenPHP qua Docker

Sử dụng Docker images chính thức của FrankenPHP có thể cung cấp hiệu suất cải thiện và sử dụng các extensions bổ sung không được bao gồm trong các cài đặt tĩnh của FrankenPHP. Ngoài ra, Docker images chính thức cung cấp hỗ trợ để chạy FrankenPHP trên các nền tảng mà nó không hỗ trợ sẵn, chẳng hạn như Windows. Docker images chính thức của FrankenPHP phù hợp cho cả phát triển cục bộ và sử dụng production.

Bạn có thể sử dụng Dockerfile sau làm điểm bắt đầu để containerize ứng dụng Laravel chạy bằng FrankenPHP của mình:

```dockerfile
FROM dunglas/frankenphp

RUN install-php-extensions \
    pcntl
    # Add other PHP extensions here...

COPY . /app

ENTRYPOINT ["php", "artisan", "octane:frankenphp"]
```

Sau đó, trong quá trình phát triển, bạn có thể sử dụng Docker Compose file sau để chạy ứng dụng của mình:

```yaml
# compose.yaml
services:
  frankenphp:
    build:
      context: .
    entrypoint: php artisan octane:frankenphp --workers=1 --max-requests=1
    ports:
      - "8000:8000"
    volumes:
      - .:/app
```

Nếu option `--log-level` được truyền rõ ràng vào command `php artisan octane:start`, Octane sẽ sử dụng logger gốc của FrankenPHP và, trừ khi được cấu hình khác, sẽ tạo ra các log JSON có cấu trúc.

Bạn có thể tham khảo [tài liệu FrankenPHP chính thức](https://frankenphp.dev/docs/docker/) để biết thêm thông tin về việc chạy FrankenPHP với Docker.

<a name="frankenphp-caddyfile"></a>
#### Cấu hình Caddyfile Tùy chỉnh

Khi sử dụng FrankenPHP, bạn có thể chỉ định một Caddyfile tùy chỉnh sử dụng option `--caddyfile` khi khởi động Octane:

```shell
php artisan octane:start --server=frankenphp --caddyfile=/path/to/your/Caddyfile
```

Điều này cho phép bạn tùy chỉnh cấu hình của FrankenPHP vượt quá các cài đặt mặc định, chẳng hạn như thêm middleware tùy chỉnh, cấu hình routing nâng cao, hoặc thiết lập các directives tùy chỉnh. Bạn có thể tham khảo [tài liệu Caddy chính thức](https://caddyserver.com/docs/caddyfile) để biết thêm thông tin về cú pháp và tùy chọn cấu hình Caddyfile.

<a name="roadrunner"></a>
### RoadRunner

[RoadRunner](https://roadrunner.dev) được cung cấp bởi RoadRunner binary, được xây dựng sử dụng Go. Lần đầu tiên bạn khởi động một Octane server dựa trên RoadRunner, Octane sẽ đề nghị tải xuống và cài đặt RoadRunner binary cho bạn.

<a name="roadrunner-via-laravel-sail"></a>
#### RoadRunner qua Laravel Sail

Nếu bạn dự định phát triển ứng dụng của mình sử dụng [Laravel Sail](/docs/{{version}}/sail), bạn nên chạy các command sau để cài đặt Octane và RoadRunner:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http
```

Tiếp theo, bạn nên khởi động một Sail shell và sử dụng executable `rr` để lấy bản build Linux mới nhất của RoadRunner binary:

```shell
./vendor/bin/sail shell

# Within the Sail shell...
./vendor/bin/rr get-binary
```

Sau đó, thêm một environment variable `SUPERVISOR_PHP_COMMAND` vào định nghĩa service `laravel.test` trong file `docker-compose.yml` của ứng dụng của bạn. Environment variable này sẽ chứa command mà Sail sẽ sử dụng để phục vụ ứng dụng của bạn sử dụng Octane thay vì PHP development server:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port='${APP_PORT:-80}'" # [tl! add]
```

Cuối cùng, đảm bảo binary `rr` có thể thực thi được và xây dựng Sail images của bạn:

```shell
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

<a name="swoole"></a>
### Swoole

Nếu bạn dự định sử dụng Swoole application server để phục vụ ứng dụng Laravel Octane của mình, bạn phải cài đặt Swoole PHP extension. Thông thường, điều này có thể được thực hiện thông qua PECL:

```shell
pecl install swoole
```

<a name="openswoole"></a>
#### Open Swoole

Nếu bạn muốn sử dụng Open Swoole application server để phục vụ ứng dụng Laravel Octane của mình, bạn phải cài đặt Open Swoole PHP extension. Thông thường, điều này có thể được thực hiện thông qua PECL:

```shell
pecl install openswoole
```

Sử dụng Laravel Octane với Open Swoole cung cấp cùng chức năng được cung cấp bởi Swoole, chẳng hạn như concurrent tasks, ticks, và intervals.

<a name="swoole-via-laravel-sail"></a>
#### Swoole qua Laravel Sail

> [!WARNING]
> Trước khi phục vụ một ứng dụng Octane qua Sail, đảm bảo bạn có phiên bản Laravel Sail mới nhất và thực thi `./vendor/bin/sail build --no-cache` trong thư mục gốc của ứng dụng của bạn.

Ngoài ra, bạn có thể phát triển ứng dụng Octane dựa trên Swoole của mình sử dụng [Laravel Sail](/docs/{{version}}/sail), môi trường phát triển dựa trên Docker chính thức cho Laravel. Laravel Sail bao gồm Swoole extension theo mặc định. Tuy nhiên, bạn vẫn cần điều chỉnh file `docker-compose.yml` được sử dụng bởi Sail.

Để bắt đầu, thêm một environment variable `SUPERVISOR_PHP_COMMAND` vào định nghĩa service `laravel.test` trong file `docker-compose.yml` của ứng dụng của bạn. Environment variable này sẽ chứa command mà Sail sẽ sử dụng để phục vụ ứng dụng của bạn sử dụng Octane thay vì PHP development server:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port='${APP_PORT:-80}'" # [tl! add]
```

Cuối cùng, xây dựng Sail images của bạn:

```shell
./vendor/bin/sail build --no-cache
```

<a name="swoole-configuration"></a>
#### Cấu hình Swoole

Swoole hỗ trợ một số tùy chọn cấu hình bổ sung mà bạn có thể thêm vào file cấu hình `octane` của mình nếu cần. Vì chúng hiếm khi cần được sửa đổi, các tùy chọn này không được bao gồm trong file cấu hình mặc định:

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

<a name="serving-your-application"></a>
## Phục vụ Ứng dụng của Bạn

Octane server có thể được khởi động thông qua command `octane:start` của Artisan. Theo mặc định, command này sẽ sử dụng server được chỉ định bởi tùy chọn cấu hình `server` của file cấu hình `octane` của ứng dụng của bạn:

```shell
php artisan octane:start
```

Theo mặc định, Octane sẽ khởi động server trên port 8000, vì vậy bạn có thể truy cập ứng dụng của mình trong trình duyệt web qua `http://localhost:8000`.

<a name="keeping-octane-running-in-production"></a>
#### Giữ Octane Chạy trong Production

Nếu bạn đang triển khai ứng dụng Octane của mình lên production, bạn nên sử dụng một process monitor như Supervisor để đảm bảo Octane server vẫn chạy. Một file cấu hình Supervisor mẫu cho Octane có thể trông như sau:

```ini
[program:octane]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/example.com/artisan octane:start --server=frankenphp --host=127.0.0.1 --port=8000
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/storage/logs/octane.log
stopwaitsecs=3600
```

<a name="serving-your-application-via-https"></a>
### Phục vụ Ứng dụng của Bạn qua HTTPS

Theo mặc định, các ứng dụng chạy qua Octane tạo ra các link có tiền tố `http://`. Environment variable `OCTANE_HTTPS`, được sử dụng trong file cấu hình `config/octane.php` của ứng dụng của bạn, có thể được đặt thành `true` khi phục vụ ứng dụng của bạn qua HTTPS. Khi giá trị cấu hình này được đặt thành `true`, Octane sẽ hướng dẫn Laravel thêm tiền tố `https://` vào tất cả các link được tạo:

```php
'https' => env('OCTANE_HTTPS', false),
```

<a name="serving-your-application-via-nginx"></a>
### Phục vụ Ứng dụng của Bạn qua Nginx

> [!NOTE]
> Nếu bạn chưa sẵn sàng để quản lý cấu hình server của riêng mình hoặc không thoải mái khi cấu hình tất cả các dịch vụ khác nhau cần thiết để chạy một ứng dụng Laravel Octane mạnh mẽ, hãy xem [Laravel Cloud](https://cloud.laravel.com), cung cấp hỗ trợ Laravel Octane được quản lý hoàn toàn.

Trong môi trường production, bạn nên phục vụ ứng dụng Octane của mình đằng sau một web server truyền thống như Nginx hoặc Apache. Việc làm như vậy sẽ cho phép web server phục vụ các static assets của bạn như hình ảnh và stylesheets, cũng như quản lý việc chấm dứt SSL certificate của bạn.

Trong ví dụ cấu hình Nginx dưới đây, Nginx sẽ phục vụ các static assets của trang web và proxy các request đến Octane server đang chạy trên port 8000:

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

<a name="watching-for-file-changes"></a>
### Theo dõi Thay đổi File

Vì ứng dụng của bạn được tải vào memory một lần khi Octane server khởi động, bất kỳ thay đổi nào đối với các file của ứng dụng của bạn sẽ không được phản ánh khi bạn refresh trình duyệt của mình. Ví dụ, các định nghĩa route được thêm vào file `routes/web.php` của bạn sẽ không được phản ánh cho đến khi server được khởi động lại. Để thuận tiện, bạn có thể sử dụng flag `--watch` để hướng dẫn Octane tự động khởi động lại server khi có bất kỳ thay đổi file nào trong ứng dụng của bạn:

```shell
php artisan octane:start --watch
```

Trước khi sử dụng tính năng này, bạn nên đảm bảo rằng [Node](https://nodejs.org) được cài đặt trong môi trường phát triển cục bộ của bạn. Ngoài ra, bạn nên cài đặt [Chokidar](https://github.com/paulmillr/chokidar) library theo dõi file trong dự án của mình:

```shell
npm install --save-dev chokidar
```

Bạn có thể cấu hình các thư mục và file nên được theo dõi sử dụng tùy chọn cấu hình `watch` trong file cấu hình `config/octane.php` của ứng dụng của bạn.

<a name="specifying-the-worker-count"></a>
### Chỉ định Số lượng Worker

Theo mặc định, Octane sẽ khởi động một application request worker cho mỗi CPU core được cung cấp bởi máy của bạn. Các worker này sau đó sẽ được sử dụng để phục vụ các HTTP request đến khi chúng vào ứng dụng của bạn. Bạn có thể chỉ định thủ công bao nhiêu worker bạn muốn khởi động sử dụng option `--workers` khi gọi command `octane:start`:

```shell
php artisan octane:start --workers=4
```

Nếu bạn đang sử dụng Swoole application server, bạn cũng có thể chỉ định bao nhiêu ["task workers"](#concurrent-tasks) bạn muốn khởi động:

```shell
php artisan octane:start --workers=4 --task-workers=6
```

<a name="specifying-the-max-request-count"></a>
### Chỉ định Số lượng Request Tối đa

Để giúp ngăn chặn memory leaks bất ngờ, Octane sẽ khởi động lại một cách nhẹ nhàng bất kỳ worker nào sau khi nó đã xử lý 500 request. Để điều chỉnh số này, bạn có thể sử dụng option `--max-requests`:

```shell
php artisan octane:start --max-requests=250
```

<a name="specifying-the-max-execution-time"></a>
### Chỉ định Thời gian Thực thi Tối đa

Theo mặc định, Laravel Octane đặt thời gian thực thi tối đa là 30 giây cho các request đến thông qua option `max_execution_time` trong file cấu hình `config/octane.php` của ứng dụng của bạn:

```php
'max_execution_time' => 30,
```

Cài đặt này định nghĩa số giây tối đa mà một request đến được phép thực thi trước khi bị chấm dứt. Đặt giá trị này thành `0` sẽ vô hiệu hóa hoàn toàn giới hạn thời gian thực thi. Tùy chọn cấu hình này đặc biệt hữu ích cho các ứng dụng xử lý các request chạy dài, chẳng hạn như tải lên file, xử lý dữ liệu, hoặc các API calls đến các dịch vụ bên ngoài.

> [!WARNING]
> Khi bạn sửa đổi cấu hình `max_execution_time`, bạn phải khởi động lại Octane server để các thay đổi có hiệu lực.

<a name="reloading-the-workers"></a>
### Tải lại các Worker

Bạn có thể khởi động lại một cách nhẹ nhàng các application worker của Octane server sử dụng command `octane:reload`. Thông thường, điều này nên được thực hiện sau khi triển khai để code mới được triển khai của bạn được tải vào memory và được sử dụng để phục vụ các request tiếp theo:

```shell
php artisan octane:reload
```

<a name="stopping-the-server"></a>
### Dừng Server

Bạn có thể dừng Octane server sử dụng command `octane:stop` của Artisan:

```shell
php artisan octane:stop
```

<a name="checking-the-server-status"></a>
#### Kiểm tra Trạng thái Server

Bạn có thể kiểm tra trạng thái hiện tại của Octane server sử dụng command `octane:status` của Artisan:

```shell
php artisan octane:status
```

<a name="dependency-injection-and-octane"></a>
## Dependency Injection và Octane

Vì Octane khởi động ứng dụng của bạn một lần và giữ nó trong memory trong khi phục vụ các request, có một số lưu ý bạn nên xem xét khi xây dựng ứng dụng của mình. Ví dụ, các phương thức `register` và `boot` của service providers của ứng dụng của bạn sẽ chỉ được thực thi một lần khi request worker khởi động ban đầu. Trong các request tiếp theo, cùng một instance ứng dụng sẽ được tái sử dụng.

Với điều này, bạn nên đặc biệt chú ý khi injecting application service container hoặc request vào constructor của bất kỳ đối tượng nào. Bằng cách làm như vậy, đối tượng đó có thể có phiên bản cũ của container hoặc request trong các request tiếp theo.

Octane sẽ tự động xử lý việc reset bất kỳ trạng thái framework chính thức nào giữa các request. Tuy nhiên, Octane không luôn biết cách reset trạng thái toàn cầu được tạo bởi ứng dụng của bạn. Do đó, bạn nên biết cách xây dựng ứng dụng của mình theo cách thân thiện với Octane. Dưới đây, chúng ta sẽ thảo luận về các tình huống phổ biến nhất có thể gây ra vấn đề khi sử dụng Octane.

<a name="container-injection"></a>
### Container Injection

Nói chung, bạn nên tránh injecting application service container hoặc HTTP request instance vào constructor của các đối tượng khác. Ví dụ, binding sau inject toàn bộ application service container vào một đối tượng được bound như một singleton:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

Trong ví dụ này, nếu instance `Service` được giải quyết trong quá trình khởi động ứng dụng, container sẽ được inject vào service và cùng container đó sẽ được giữ bởi instance `Service` trong các request tiếp theo. Điều này **có thể** không phải là vấn đề cho ứng dụng cụ thể của bạn; tuy nhiên, nó có thể dẫn đến việc container thiếu các binding một cách bất ngờ được thêm sau đó trong chu kỳ khởi động hoặc bởi một request tiếp theo.

Là một giải pháp thay thế, bạn có thể ngừng đăng ký binding như một singleton, hoặc bạn có thể inject một closure giải quyết container vào service luôn giải quyết instance container hiện tại:

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

Helper toàn cầu `app` và phương thức `Container::getInstance()` sẽ luôn trả về phiên bản mới nhất của application container.

<a name="request-injection"></a>
### Request Injection

Nói chung, bạn nên tránh injecting application service container hoặc HTTP request instance vào constructor của các đối tượng khác. Ví dụ, binding sau inject toàn bộ request instance vào một đối tượng được bound như một singleton:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app['request']);
    });
}
```

Trong ví dụ này, nếu instance `Service` được giải quyết trong quá trình khởi động ứng dụng, HTTP request sẽ được inject vào service và cùng request đó sẽ được giữ bởi instance `Service` trong các request tiếp theo. Do đó, tất cả headers, input, và query string data sẽ không chính xác, cũng như tất cả các request data khác.

Là một giải pháp thay thế, bạn có thể ngừng đăng ký binding như một singleton, hoặc bạn có thể inject một closure giải quyết request vào service luôn giải quyết instance request hiện tại. Hoặc, cách tiếp cận được khuyến nghị nhất là đơn giản chuyển thông tin request cụ thể mà đối tượng của bạn cần đến một trong các phương thức của đối tượng tại runtime:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// Or...

$service->method($request->input('name'));
```

Helper toàn cầu `request` sẽ luôn trả về request mà ứng dụng hiện đang xử lý và do đó an toàn để sử dụng trong ứng dụng của bạn.

> [!WARNING]
> Việc type-hint instance `Illuminate\Http\Request` trên các phương thức controller và route closures của bạn là chấp nhận được.

<a name="configuration-repository-injection"></a>
### Configuration Repository Injection

Nói chung, bạn nên tránh injecting configuration repository instance vào constructor của các đối tượng khác. Ví dụ, binding sau inject configuration repository vào một đối tượng được bound như một singleton:

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

Trong ví dụ này, nếu các giá trị cấu hình thay đổi giữa các request, service đó sẽ không có quyền truy cập vào các giá trị mới vì nó phụ thuộc vào instance repository gốc.

Là một giải pháp thay thế, bạn có thể ngừng đăng ký binding như một singleton, hoặc bạn có thể inject một closure giải quyết configuration repository vào class:

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

Helper toàn cầu `config` sẽ luôn trả về phiên bản mới nhất của configuration repository và do đó an toàn để sử dụng trong ứng dụng của bạn.

<a name="managing-memory-leaks"></a>
### Quản lý Memory Leaks

Hãy nhớ, Octane giữ ứng dụng của bạn trong memory giữa các request; do đó, thêm dữ liệu vào một mảng được duy trì tĩnh sẽ dẫn đến memory leak. Ví dụ, controller sau có memory leak vì mỗi request đến ứng dụng sẽ tiếp tục thêm dữ liệu vào mảng tĩnh `$data`:

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * Handle an incoming request.
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

Trong khi xây dựng ứng dụng của mình, bạn nên đặc biệt chú ý để tránh tạo ra các loại memory leak này. Được khuyến nghị rằng bạn nên giám sát việc sử dụng memory của ứng dụng của mình trong quá trình phát triển cục bộ để đảm bảo bạn không đang giới thiệu các memory leak mới vào ứng dụng của mình.

<a name="concurrent-tasks"></a>
## Concurrent Tasks

> [!WARNING]
> Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể thực thi các hoạt động đồng thời thông qua các background task nhẹ. Bạn có thể thực hiện điều này sử dụng phương thức `concurrently` của Octane. Bạn có thể kết hợp phương thức này với array destructuring của PHP để lấy kết quả của mỗi hoạt động:

```php
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

Các concurrent task được xử lý bởi Octane sử dụng "task workers" của Swoole, và thực thi trong một process hoàn toàn khác với request đến. Số lượng worker có sẵn để xử lý các concurrent task được xác định bởi directive `--task-workers` trên command `octane:start`:

```shell
php artisan octane:start --workers=4 --task-workers=6
```

Khi gọi phương thức `concurrently`, bạn không nên cung cấp nhiều hơn 1024 task do các giới hạn được áp đặt bởi hệ thống task của Swoole.

<a name="ticks-and-intervals"></a>
## Ticks và Intervals

> [!WARNING]
> Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể đăng ký các hoạt động "tick" sẽ được thực thi mỗi số giây được chỉ định. Bạn có thể đăng ký các callback "tick" thông qua phương thức `tick`. Đối số đầu tiên được cung cấp cho phương thức `tick` nên là một chuỗi đại diện cho tên của ticker. Đối số thứ hai nên là một callable sẽ được gọi tại khoảng thời gian được chỉ định.

Trong ví dụ này, chúng ta sẽ đăng ký một closure để được gọi mỗi 10 giây. Thông thường, phương thức `tick` nên được gọi trong phương thức `boot` của một trong các service providers của ứng dụng của bạn:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10);
```

Sử dụng phương thức `immediate`, bạn có thể hướng dẫn Octane gọi ngay lập tức tick callback khi Octane server khởi động ban đầu, và mỗi N giây sau đó:

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10)
    ->immediate();
```

<a name="the-octane-cache"></a>
## Octane Cache

> [!WARNING]
> Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể tận dụng Octane cache driver, cung cấp tốc độ đọc và ghi lên đến 2 triệu hoạt động mỗi giây. Do đó, cache driver này là một lựa chọn tuyệt vời cho các ứng dụng cần tốc độ đọc/ghi cực cao từ caching layer của họ.

Cache driver này được cung cấp bởi [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table). Tất cả dữ liệu được lưu trữ trong cache có sẵn cho tất cả worker trên server. Tuy nhiên, dữ liệu được cache sẽ bị xóa khi server được khởi động lại:

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

> [!NOTE]
> Số lượng mục tối đa được phép trong Octane cache có thể được định nghĩa trong file cấu hình `octane` của ứng dụng của bạn.

<a name="cache-intervals"></a>
### Cache Intervals

Ngoài các phương thức điển hình được cung cấp bởi hệ thống cache của Laravel, Octane cache driver có các cache dựa trên interval. Các cache này được tự động làm mới tại khoảng thời gian được chỉ định và nên được đăng ký trong phương thức `boot` của một trong các service providers của ứng dụng của bạn. Ví dụ, cache sau sẽ được làm mới mỗi năm giây:

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

<a name="tables"></a>
## Tables

> [!WARNING]
> Tính năng này yêu cầu [Swoole](#swoole).

Khi sử dụng Swoole, bạn có thể định nghĩa và tương tác với các [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table) tùy ý của riêng bạn. Swoole tables cung cấp throughput hiệu suất cực cao và dữ liệu trong các bảng này có thể được truy cập bởi tất cả worker trên server. Tuy nhiên, dữ liệu trong chúng sẽ bị mất khi server được khởi động lại.

Tables nên được định nghĩa trong mảng cấu hình `tables` của file cấu hình `octane` của ứng dụng của bạn. Một bảng ví dụ cho phép tối đa 1000 hàng đã được cấu hình cho bạn. Kích thước tối đa của các cột chuỗi có thể được cấu hình bằng cách chỉ định kích thước cột sau loại cột như được thấy dưới đây:

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

Để truy cập một bảng, bạn có thể sử dụng phương thức `Octane::table`:

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> [!WARNING]
> Các loại cột được hỗ trợ bởi Swoole tables là: `string`, `int`, và `float`.

# Laravel Reverb

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
    - [Application Credentials](#application-credentials)
    - [Allowed Origins](#allowed-origins)
    - [Additional Applications](#additional-applications)
    - [SSL](#ssl)
- [Chạy Server](#running-server)
    - [Debugging](#debugging)
    - [Restarting](#restarting)
- [Monitoring](#monitoring)
- [Chạy Reverb trong Production](#production)
    - [Open Files](#open-files)
    - [Event Loop](#event-loop)
    - [Web Server](#web-server)
    - [Ports](#ports)
    - [Process Management](#process-management)
    - [Scaling](#scaling)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

[Laravel Reverb](https://github.com/laravel/reverb) mang lại khả năng giao tiếp WebSocket real-time nhanh và có khả năng mở rộng trực tiếp đến ứng dụng Laravel của bạn, và cung cấp tích hợp liền mạch với bộ công cụ [event broadcasting](/docs/{{version}}/broadcasting) hiện có của Laravel.

<a name="installation"></a>
## Cài đặt

Bạn có thể cài đặt Reverb bằng cách sử dụng command Artisan `install:broadcasting`:

```shell
php artisan install:broadcasting
```

<a name="configuration"></a>
## Cấu hình

Ngầm bên dưới, command Artisan `install:broadcasting` sẽ chạy command `reverb:install`, sẽ cài đặt Reverb với một tập hợp các tùy chọn cấu hình mặc định hợp lý. Nếu bạn muốn thực hiện bất kỳ thay đổi cấu hình nào, bạn có thể làm điều đó bằng cách cập nhật các biến môi trường của Reverb hoặc bằng cách cập nhật file cấu hình `config/reverb.php`.

<a name="application-credentials"></a>
### Application Credentials

Để thiết lập kết nối đến Reverb, một tập hợp credentials "application" của Reverb phải được trao đổi giữa client và server. Các credentials này được cấu hình trên server và được sử dụng để xác minh request từ client. Bạn có thể định nghĩa các credentials này bằng cách sử dụng các biến môi trường sau:

```ini
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

<a name="allowed-origins"></a>
### Allowed Origins

Bạn cũng có thể định nghĩa các origins mà từ đó các request client có thể xuất phát bằng cách cập nhật giá trị cấu hình `allowed_origins` trong phần `apps` của file cấu hình `config/reverb.php`. Bất kỳ request nào từ một origin không được liệt kê trong allowed origins của bạn sẽ bị từ chối. Bạn có thể cho phép tất cả các origins bằng cách sử dụng `*`:

```php
'apps' => [
    [
        'app_id' => 'my-app-id',
        'allowed_origins' => ['laravel.com'],
        // ...
    ]
]
```

<a name="additional-applications"></a>
### Additional Applications

Thông thường, Reverb cung cấp một WebSocket server cho ứng dụng trong đó nó được cài đặt. Tuy nhiên, có thể phục vụ nhiều hơn một ứng dụng bằng cách sử dụng một cài đặt Reverb duy nhất.

Ví dụ, bạn có thể muốn duy trì một ứng dụng Laravel duy nhất mà, thông qua Reverb, cung cấp khả năng kết nối WebSocket cho nhiều ứng dụng. Điều này có thể đạt được bằng cách định nghĩa nhiều `apps` trong file cấu hình `config/reverb.php` của ứng dụng:

```php
'apps' => [
    [
        'app_id' => 'my-app-one',
        // ...
    ],
    [
        'app_id' => 'my-app-two',
        // ...
    ],
],
```

<a name="ssl"></a>
### SSL

Trong hầu hết các trường hợp, các kết nối WebSocket an toàn được xử lý bởi web server upstream (Nginx, v.v) trước khi request được proxy đến Reverb server của bạn.

Tuy nhiên, đôi khi có thể hữu ích, chẳng hạn như trong quá trình phát triển cục bộ, để Reverb server xử lý các kết nối an toàn trực tiếp. Nếu bạn đang sử dụng tính năng secure site của [Laravel Herd](https://herd.laravel.com) hoặc bạn đang sử dụng [Laravel Valet](/docs/{{version}}/valet) và đã chạy [secure command](/docs/{{version}}/valet#securing-sites) đối với ứng dụng của bạn, bạn có thể sử dụng certificate Herd / Valet được tạo cho site của bạn để bảo mật các kết nối Reverb của bạn. Để làm điều đó, đặt biến môi trường `REVERB_HOST` thành hostname của site hoặc truyền tùy chọn hostname một cách rõ ràng khi khởi động Reverb server:

```shell
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

Vì các domain Herd và Valet giải quyết đến `localhost`, chạy command ở trên sẽ dẫn đến Reverb server của bạn có thể truy cập thông qua giao thức WebSocket an toàn (`wss`) tại `wss://laravel.test:8080`.

Bạn cũng có thể chọn thủ công một certificate bằng cách định nghĩa các tùy chọn `tls` trong file cấu hình `config/reverb.php` của ứng dụng. Trong mảng các tùy chọn `tls`, bạn có thể cung cấp bất kỳ tùy chọn nào được hỗ trợ bởi [SSL context options của PHP](https://www.php.net/manual/en/context.ssl.php):

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## Chạy Server

Reverb server có thể được khởi động bằng cách sử dụng command Artisan `reverb:start`:

```shell
php artisan reverb:start
```

Theo mặc định, Reverb server sẽ được khởi động tại `0.0.0.0:8080`, làm cho nó có thể truy cập từ tất cả các network interfaces.

Nếu bạn cần chỉ định một host hoặc port tùy chỉnh, bạn có thể làm điều đó thông qua các tùy chọn `--host` và `--port` khi khởi động server:

```shell
php artisan reverb:start --host=127.0.0.1 --port=9000
```

Ngoài ra, bạn có thể định nghĩa các biến môi trường `REVERB_SERVER_HOST` và `REVERB_SERVER_PORT` trong file cấu hình `.env` của ứng dụng.

Các biến môi trường `REVERB_SERVER_HOST` và `REVERB_SERVER_PORT` không nên bị nhầm lẫn với `REVERB_HOST` và `REVERB_PORT`. Cái trước chỉ định host và port trên đó để chạy Reverb server thực tế, trong khi cặp sau hướng dẫn Laravel nơi để gửi broadcast messages. Ví dụ, trong môi trường production, bạn có thể route các request từ Reverb hostname công khai của bạn trên port `443` đến một Reverb server hoạt động trên `0.0.0.0:8080`. Trong kịch bản này, các biến môi trường của bạn sẽ được định nghĩa như sau:

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### Debugging

Để cải thiện hiệu suất, Reverb không xuất bất kỳ thông tin debug nào theo mặc định. Nếu bạn muốn xem luồng dữ liệu đi qua Reverb server của bạn, bạn có thể cung cấp tùy chọn `--debug` cho command `reverb:start`:

```shell
php artisan reverb:start --debug
```

<a name="restarting"></a>
### Restarting

Vì Reverb là một long-running process, các thay đổi đối với code của bạn sẽ không được phản ánh mà không khởi động lại server thông qua command Artisan `reverb:restart`.

Command `reverb:restart` đảm bảo tất cả các kết nối được terminated một cách graceful trước khi dừng server. Nếu bạn đang chạy Reverb với một process manager như Supervisor, server sẽ được khởi động lại tự động bởi process manager sau khi tất cả các kết nối đã được terminated:

```shell
php artisan reverb:restart
```

<a name="monitoring"></a>
## Monitoring

Reverb có thể được monitored thông qua tích hợp với [Laravel Pulse](/docs/{{version}}/pulse). Bằng cách bật tích hợp Pulse của Reverb, bạn có thể theo dõi số lượng kết nối và messages đang được xử lý bởi server của bạn.

Để bật tích hợp, trước tiên bạn nên đảm bảo bạn đã [cài đặt Pulse](/docs/{{version}}/pulse#installation). Sau đó, thêm bất kỳ recorders nào của Reverb vào file cấu hình `config/pulse.php` của ứng dụng:

```php
use Laravel\Reverb\Pulse\Recorders\ReverbConnections;
use Laravel\Reverb\Pulse\Recorders\ReverbMessages;

'recorders' => [
    ReverbConnections::class => [
        'sample_rate' => 1,
    ],

    ReverbMessages::class => [
        'sample_rate' => 1,
    ],

    // ...
],
```

Tiếp theo, thêm các Pulse cards cho mỗi recorder vào [Pulse dashboard](/docs/{{version}}/pulse#dashboard-customization) của bạn:

```blade
<x-pulse>
    <livewire:reverb.connections cols="full" />
    <livewire:reverb.messages cols="full" />
    ...
</x-pulse>
```

Hoạt động kết nối được ghi lại bằng cách polling cho các cập nhật mới trên cơ sở định kỳ. Để đảm bảo thông tin này được hiển thị chính xác trên Pulse dashboard, bạn phải chạy daemon `pulse:check` trên Reverb server của bạn. Nếu bạn đang chạy Reverb trong cấu hình [horizontally scaled](#scaling), bạn chỉ nên chạy daemon này trên một trong các server của bạn.

<a name="production"></a>
## Chạy Reverb trong Production

Do bản chất long-running của WebSocket servers, bạn có thể cần thực hiện một số tối ưu hóa cho server và môi trường hosting của bạn để đảm bảo Reverb server của bạn có thể xử lý hiệu quả số lượng kết nối tối ưu cho các tài nguyên có sẵn trên server của bạn.

> [!NOTE]
> [Laravel Cloud](https://cloud.laravel.com) cung cấp cơ sở hạ tầng WebSocket được quản lý hoàn toàn được hỗ trợ bởi các cluster Laravel Reverb, cho phép bạn scale và ship các ứng dụng có Reverb mà không cần quản lý cơ sở hạ tầng.

<a name="open-files"></a>
### Open Files

Mỗi kết nối WebSocket được giữ trong bộ nhớ cho đến khi client hoặc server ngắt kết nối. Trong các môi trường Unix và Unix-like, mỗi kết nối được đại diện bởi một file. Tuy nhiên, thường có các giới hạn về số lượng file mở được cho phép ở cả cấp độ hệ điều hành và ứng dụng.

<a name="operating-system"></a>
#### Operating System

Trên hệ điều hành dựa trên Unix, bạn có thể xác định số lượng file mở được cho phép bằng cách sử dụng command `ulimit`:

```shell
ulimit -n
```

Command này sẽ hiển thị các giới hạn file mở được cho phép cho các user khác nhau. Bạn có thể cập nhật các giá trị này bằng cách chỉnh sửa file `/etc/security/limits.conf`. Ví dụ, cập nhật số lượng file mở tối đa lên 10,000 cho user `forge` sẽ trông như sau:

```ini
# /etc/security/limits.conf
forge        soft  nofile  10000
forge        hard  nofile  10000
```

<a name="event-loop"></a>
### Event Loop

Ngầm bên dưới, Reverb sử dụng event loop ReactPHP để quản lý các kết nối WebSocket trên server. Theo mặc định, event loop này được hỗ trợ bởi `stream_select`, không yêu cầu bất kỳ extensions bổ sung nào. Tuy nhiên, `stream_select` thường bị giới hạn ở 1,024 file mở. Do đó, nếu bạn dự định xử lý hơn 1,000 kết nối đồng thời, bạn sẽ cần sử dụng một event loop thay thế không bị ràng buộc bởi cùng các giới hạn đó.

Reverb sẽ tự động chuyển sang một loop được hỗ trợ bởi `ext-uv` khi có sẵn. PHP extension này có sẵn để cài đặt thông qua PECL:

```shell
pecl install uv
```

<a name="web-server"></a>
### Web Server

Trong hầu hết các trường hợp, Reverb chạy trên một port không web-facing trên server của bạn. Vì vậy, để route traffic đến Reverb, bạn nên cấu hình một reverse proxy. Giả sử Reverb đang chạy trên host `0.0.0.0` và port `8080` và server của bạn sử dụng web server Nginx, một reverse proxy có thể được định nghĩa cho Reverb server của bạn bằng cách sử dụng cấu hình site Nginx sau:

```nginx
server {
    ...

    location / {
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";

        proxy_pass http://0.0.0.0:8080;
    }

    ...
}
```

> [!WARNING]
> Reverb lắng nghe các kết nối WebSocket tại `/app` và xử lý các API requests tại `/apps`. Bạn nên đảm bảo web server xử lý các request Reverb có thể phục vụ cả hai URI này. Nếu bạn đang sử dụng [Laravel Forge](https://forge.laravel.com) để quản lý các server của bạn, Reverb server của bạn sẽ được cấu hình chính xác theo mặc định.

Thông thường, các web servers được cấu hình để giới hạn số lượng kết nối được cho phép để ngăn chặn việc quá tải server. Để tăng số lượng kết nối được cho phép trên web server Nginx lên 10,000, các giá trị `worker_rlimit_nofile` và `worker_connections` của file `nginx.conf` nên được cập nhật:

```nginx
user forge;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
worker_rlimit_nofile 10000;

events {
  worker_connections 10000;
  multi_accept on;
}
```

Cấu hình ở trên sẽ cho phép lên đến 10,000 Nginx workers mỗi process được spawn. Ngoài ra, cấu hình này đặt giới hạn file mở của Nginx lên 10,000.

<a name="ports"></a>
### Ports

Các hệ điều hành dựa trên Unix thường giới hạn số lượng ports có thể được mở trên server. Bạn có thể xem phạm vi được cho phép hiện tại thông qua command sau:

```shell
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

Output ở trên cho thấy server có thể xử lý tối đa 28,231 (60,999 - 32,768) kết nối vì mỗi kết nối yêu cầu một port trống. Mặc dù chúng tôi khuyên dùng [horizontal scaling](#scaling) để tăng số lượng kết nối được cho phép, bạn có thể tăng số lượng ports mở có sẵn bằng cách cập nhật phạm vi port được cho phép trong file cấu hình `/etc/sysctl.conf` của server.

<a name="process-management"></a>
### Process Management

Trong hầu hết các trường hợp, bạn nên sử dụng một process manager như Supervisor để đảm bảo Reverb server liên tục chạy. Nếu bạn đang sử dụng Supervisor để chạy Reverb, bạn nên cập nhật cài đặt `minfds` của file `supervisor.conf` của server để đảm bảo Supervisor có thể mở các file cần thiết để xử lý các kết nối đến Reverb server của bạn:

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### Scaling

Nếu bạn cần xử lý nhiều kết nối hơn một server sẽ cho phép, bạn có thể scale Reverb server của bạn theo chiều ngang. Sử dụng các khả năng publish / subscribe của Redis, Reverb có thể quản lý các kết nối trên nhiều server. Khi một message được nhận bởi một trong các Reverb servers của ứng dụng, server sẽ sử dụng Redis để publish message đến tất cả các server khác.

Để bật horizontal scaling, bạn nên đặt biến môi trường `REVERB_SCALING_ENABLED` thành `true` trong file cấu hình `.env` của ứng dụng:

```env
REVERB_SCALING_ENABLED=true
```

Tiếp theo, bạn nên có một Redis server trung tâm, chuyên dụng mà tất cả các Reverb servers sẽ giao tiếp. Reverb sẽ sử dụng [Redis connection mặc định được cấu hình cho ứng dụng của bạn](/docs/{{version}}/redis#configuration) để publish messages đến tất cả các Reverb servers của bạn.

Khi bạn đã bật tùy chọn scaling của Reverb và cấu hình một Redis server, bạn có thể chỉ cần gọi command `reverb:start` trên nhiều server có thể giao tiếp với Redis server của bạn. Các Reverb servers này nên được đặt sau một load balancer phân phối các requests đến đều nhau giữa các server.

<a name="events"></a>
## Events

Reverb dispatches các internal events trong vòng đời của một kết nối và xử lý message. Bạn có thể [lắng nghe các events này](/docs/{{version}}/events) để thực hiện các hành động khi các kết nối được quản lý hoặc các messages được trao đổi.

Các events sau được dispatch bởi Reverb:

#### `Laravel\Reverb\Events\ChannelCreated`

Được dispatch khi một channel được tạo. Điều này thường xảy ra khi kết nối đầu tiên subscribe đến một channel cụ thể. Event nhận instance `Laravel\Reverb\Protocols\Pusher\Channel`.

#### `Laravel\Reverb\Events\ChannelRemoved`

Được dispatch khi một channel bị xóa. Điều này thường xảy ra khi kết nối cuối cùng unsubscribe từ một channel. Event nhận instance `Laravel\Reverb\Protocols\Pusher\Channel`.

#### `Laravel\Reverb\Events\ConnectionPruned`

Được dispatch khi một kết nối cũ bị pruned bởi server. Event nhận instance `Laravel\Reverb\Contracts\Connection`.

#### `Laravel\Reverb\Events\MessageReceived`

Được dispatch khi một message được nhận từ một kết nối client. Event nhận instance `Laravel\Reverb\Contracts\Connection` và chuỗi thô `$message`.

#### `Laravel\Reverb\Events\MessageSent`

Được dispatch khi một message được gửi đến một kết nối client. Event nhận instance `Laravel\Reverb\Contracts\Connection` và chuỗi thô `$message`.

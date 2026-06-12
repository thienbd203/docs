# Laravel Reverb

- [Introduction](#introduction)
- [Installation](#installation)
- [Configuration](#configuration)
    - [Application Credentials](#application-credentials)
    - [Allowed Origins](#allowed-origins)
    - [Additional Applications](#additional-applications)
    - [SSL](#ssl)
- [Running the Server](#running-server)
    - [Debugging](#debugging)
    - [Restarting](#restarting)
- [Monitoring](#monitoring)
- [Running Reverb in Production](#production)
    - [Open Files](#open-files)
    - [Event Loop](#event-loop)
    - [Web Server](#web-server)
    - [Ports](#ports)
    - [Process Management](#process-management)
    - [Scaling](#scaling)
- [Events](#events)

<a name="introduction"></a>
## Introduction

[Laravel Reverb](https://github.com/laravel/reverb) mang lại giao tiếp WebSocket thời gian thực cực nhanh và có thể mở rộng trực tiếp đến ứng dụng Laravel của bạn, và cung cấp tích hợp liền mạch với bộ công cụ [event broadcasting](/docs/{{version}}/broadcasting) hiện có của Laravel.

<a name="installation"></a>
## Installation

Bạn có thể cài đặt Reverb bằng cách sử dụng lệnh Artisan `install:broadcasting`:

```shell
php artisan install:broadcasting
```

<a name="configuration"></a>
## Configuration

Ngầm hậu, lệnh Artisan `install:broadcasting` sẽ chạy lệnh `reverb:install`, sẽ cài đặt Reverb với một tập hợp các tùy chọn cấu hình mặc định hợp lý. Nếu bạn muốn thực hiện bất kỳ thay đổi cấu hình nào, bạn có thể làm như vậy bằng cách cập nhật các biến môi trường Reverb hoặc bằng cách cập nhật file cấu hình `config/reverb.php`.

<a name="application-credentials"></a>
### Application Credentials

Để thiết lập kết nối đến Reverb, một bộ thông tin xác thực "application" Reverb phải được trao đổi giữa client và server. Các thông tin xác thực này được cấu hình trên server và được sử dụng để xác minh request từ client. Bạn có thể định nghĩa các thông tin xác thực này bằng cách sử dụng các biến môi trường sau:

```ini
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

<a name="allowed-origins"></a>
### Allowed Origins

Bạn cũng có thể định nghĩa các origins mà từ đó các requests client có thể xuất phát bằng cách cập nhật giá trị cấu hình `allowed_origins` trong phần `apps` của file cấu hình `config/reverb.php`. Bất kỳ request nào từ một origin không được liệt kê trong các origins được phép của bạn sẽ bị từ chối. Bạn có thể cho phép tất cả các origins bằng cách sử dụng `*`:

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

Thông thường, Reverb cung cấp một server WebSocket cho ứng dụng trong đó nó được cài đặt. Tuy nhiên, có thể phục vụ nhiều hơn một ứng dụng bằng cách sử dụng một cài đặt Reverb duy nhất.

Ví dụ, bạn có thể muốn duy trì một ứng dụng Laravel duy nhất mà, thông qua Reverb, cung cấp kết nối WebSocket cho nhiều ứng dụng. Điều này có thể đạt được bằng cách định nghĩa nhiều `apps` trong file cấu hình `config/reverb.php` của ứng dụng:

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

Trong hầu hết các trường hợp, các kết nối WebSocket an toàn được xử lý bởi web server ngược dòng (Nginx, v.v.) trước khi request được proxy đến server Reverb của bạn.

Tuy nhiên, đôi khi có thể hữu ích, chẳng hạn như trong quá trình phát triển cục bộ, để server Reverb xử lý các kết nối an toàn trực tiếp. Nếu bạn đang sử dụng tính năng site an toàn của [Laravel Herd's](https://herd.laravel.com) hoặc bạn đang sử dụng [Laravel Valet](/docs/{{version}}/valet) và đã chạy [lệnh secure](/docs/{{version}}/valet#securing-sites) đối với ứng dụng của bạn, bạn có thể sử dụng chứng chỉ Herd / Valet được tạo cho site của bạn để bảo mật các kết nối Reverb của bạn. Để thực hiện điều này, đặt biến môi trường `REVERB_HOST` thành hostname của site hoặc chuyển rõ ràng tùy chọn hostname khi bắt đầu server Reverb:

```shell
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

Vì các domain Herd và Valet giải quyết đến `localhost`, chạy lệnh trên sẽ dẫn đến server Reverb của bạn có thể truy cập thông qua giao thức WebSocket an toàn (`wss`) tại `wss://laravel.test:8080`.

Bạn cũng có thể chọn thủ công một chứng chỉ bằng cách định nghĩa các tùy chọn `tls` trong file cấu hình `config/reverb.php` của ứng dụng. Trong array các tùy chọn `tls`, bạn có thể cung cấp bất kỳ tùy chọn nào được hỗ trợ bởi [tùy chọn ngữ cảnh SSL của PHP](https://www.php.net/manual/en/context.ssl.php):

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## Running the Server

Server Reverb có thể được bắt đầu bằng cách sử dụng lệnh Artisan `reverb:start`:

```shell
php artisan reverb:start
```

Theo mặc định, server Reverb sẽ được bắt đầu tại `0.0.0.0:8080`, làm cho nó có thể truy cập từ tất cả các giao diện mạng.

Nếu bạn cần chỉ định một host hoặc port tùy chỉnh, bạn có thể làm như vậy thông qua các tùy chọn `--host` và `--port` khi bắt đầu server:

```shell
php artisan reverb:start --host=127.0.0.1 --port=9000
```

Ngoài ra, bạn có thể định nghĩa các biến môi trường `REVERB_SERVER_HOST` và `REVERB_SERVER_PORT` trong file cấu hình `.env` của ứng dụng.

Các biến môi trường `REVERB_SERVER_HOST` và `REVERB_SERVER_PORT` không nên bị nhầm lẫn với `REVERB_HOST` và `REVERB_PORT`. Cặp trước chỉ định host và port để chạy chính server Reverb, trong khi cặp sau hướng dẫn Laravel nơi gửi các tin nhắn broadcast. Ví dụ, trong môi trường production, bạn có thể định tuyến các requests từ hostname Reverb công khai trên port `443` đến một server Reverb hoạt động trên `0.0.0.0:8080`. Trong tình huống này, các biến môi trường của bạn sẽ được định nghĩa như sau:

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### Debugging

Để cải thiện hiệu suất, Reverb không xuất bất kỳ thông tin debug nào theo mặc định. Nếu bạn muốn xem luồng dữ liệu đi qua server Reverb của bạn, bạn có thể cung cấp tùy chọn `--debug` cho lệnh `reverb:start`:

```shell
php artisan reverb:start --debug
```

<a name="restarting"></a>
### Restarting

Vì Reverb là một quá trình chạy dài, các thay đổi trong mã của bạn sẽ không được phản ánh mà không khởi động lại server thông qua lệnh Artisan `reverb:restart`.

Lệnh `reverb:restart` đảm bảo tất cả các kết nối được chấm dứt một cách nhẹ nhàng trước khi dừng server. Nếu bạn đang chạy Reverb với một process manager như Supervisor, server sẽ tự động được khởi động lại bởi process manager sau khi tất cả các kết nối đã được chấm dứt:

```shell
php artisan reverb:restart
```

<a name="monitoring"></a>
## Monitoring

Reverb có thể được giám sát thông qua tích hợp với [Laravel Pulse](/docs/{{version}}/pulse). Bằng cách bật tích hợp Pulse của Reverb, bạn có thể theo dõi số lượng kết nối và tin nhắn được xử lý bởi server của bạn.

Để bật tích hợp, trước hết bạn nên đảm bảo bạn đã [cài đặt Pulse](/docs/{{version}}/pulse#installation). Sau đó, thêm bất kỳ recorders Reverb nào vào file cấu hình `config/pulse.php` của ứng dụng:

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

Tiếp theo, thêm các card Pulse cho mỗi recorder vào [dashboard Pulse](/docs/{{version}}/pulse#dashboard-customization) của bạn:

```blade
<x-pulse>
    <livewire:reverb.connections cols="full" />
    <livewire:reverb.messages cols="full" />
    ...
</x-pulse>
```

Hoạt động kết nối được ghi lại bằng cách thăm dò các cập nhật mới trên cơ sở định kỳ. Để đảm bảo thông tin này được hiển thị đúng trên dashboard Pulse, bạn phải chạy daemon `pulse:check` trên server Reverb của bạn. Nếu bạn đang chạy Reverb trong cấu hình [scale ngang](#scaling), bạn chỉ nên chạy daemon này trên một trong các servers của bạn.

<a name="production"></a>
## Running Reverb in Production

Do tính chất chạy dài của các servers WebSocket, bạn có thể cần thực hiện một số tối ưu hóa cho server và môi trường hosting của bạn để đảm bảo server Reverb của bạn có thể xử lý hiệu quả số lượng kết nối tối ưu cho các tài nguyên có sẵn trên server của bạn.

> [!NOTE]
> [Laravel Cloud](https://cloud.laravel.com) cung cấp hạ tầng WebSocket được quản lý hoàn toàn được hỗ trợ bởi các cụm Reverb Laravel, cho phép bạn scale và vận chuyển các ứng dụng được bật Reverb mà không cần quản lý hạ tầng.

<a name="open-files"></a>
### Open Files

Mỗi kết nối WebSocket được giữ trong bộ nhớ cho đến khi client hoặc server ngắt kết nối. Trong các môi trường Unix và giống Unix, mỗi kết nối được đại diện bởi một file. Tuy nhiên, thường có các giới hạn về số lượng file được phép mở ở cả cấp hệ điều hành và cấp ứng dụng.

<a name="operating-system"></a>
#### Operating System

Trên hệ điều hành dựa trên Unix, bạn có thể xác định số lượng file được phép mở bằng cách sử dụng lệnh `ulimit`:

```shell
ulimit -n
```

Lệnh này sẽ hiển thị các giới hạn file mở được phép cho các người dùng khác nhau. Bạn có thể cập nhật các giá trị này bằng cách chỉnh sửa file `/etc/security/limits.conf`. Ví dụ, cập nhật số lượng file mở tối đa lên 10,000 cho người dùng `forge` sẽ trông như sau:

```ini
# /etc/security/limits.conf
forge        soft  nofile 10000
forge        hard  nofile 10000
```

<a name="event-loop"></a>
### Event Loop

Ngầm hậu, Reverb sử dụng event loop ReactPHP để quản lý các kết nối WebSocket trên server. Theo mặc định, event loop này được hỗ trợ bởi `stream_select`, không yêu cầu bất kỳ extension bổ sung nào. Tuy nhiên, `stream_select` thường bị giới hạn ở 1,024 file mở. Như vậy, nếu bạn dự định xử lý hơn 1,000 kết nối đồng thời, bạn sẽ cần sử dụng một event loop thay thế không bị ràng buộc bởi cùng các giới hạn.

Reverb sẽ tự động chuyển sang một loop được hỗ trợ bởi `ext-uv` khi có sẵn. Extension PHP này có sẵn để cài đặt thông qua PECL:

```shell
pecl install uv
```

<a name="web-server"></a>
### Web Server

Trong hầu hết các trường hợp, Reverb chạy trên một port không hướng web trên server của bạn. Vì vậy, để định tuyến traffic đến Reverb, bạn nên cấu hình một reverse proxy. Giả sử Reverb đang chạy trên host `0.0.0.0` và port `8080` và server của bạn sử dụng web server Nginx, một reverse proxy có thể được định nghĩa cho server Reverb của bạn bằng cách sử dụng cấu hình site Nginx sau:

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
> Reverb lắng nghe các kết nối WebSocket tại `/app` và xử lý các requests API tại `/apps`. Bạn nên đảm bảo web server xử lý các requests Reverb có thể phục vụ cả hai URIs này. Nếu bạn đang sử dụng [Laravel Forge](https://forge.laravel.com) để quản lý các servers của bạn, server Reverb của bạn sẽ được cấu hình đúng theo mặc định.

Thông thường, các web servers được cấu hình để giới hạn số lượng kết nối được phép để ngăn chặn quá tải server. Để tăng số lượng kết nối được phép trên web server Nginx lên 10,000, các giá trị `worker_rlimit_nofile` và `worker_connections` của file `nginx.conf` nên được cập nhật:

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

Cấu hình ở trên sẽ cho phép tối đa 10,000 workers Nginx mỗi process được tạo ra. Ngoài ra, cấu hình này đặt giới hạn file mở của Nginx lên 10,000.

<a name="ports"></a>
### Ports

Các hệ điều hành dựa trên Unix thường giới hạn số lượng ports có thể được mở trên server. Bạn có thể xem phạm vi được phép hiện tại thông qua lệnh sau:

```shell
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

Đầu ra ở trên cho thấy server có thể xử lý tối đa 28,231 (60,999 - 32,768) kết nối vì mỗi kết nối yêu cầu một port trống. Mặc dù chúng tôi khuyến nghị [scale ngang](#scaling) để tăng số lượng kết nối được phép, bạn có thể tăng số lượng port mở có sẵn bằng cách cập nhật phạm vi port được phép trong file cấu hình `/etc/sysctl.conf` của server.

<a name="process-management"></a>
### Process Management

Trong hầu hết các trường hợp, bạn nên sử dụng một process manager như Supervisor để đảm bảo server Reverb liên tục chạy. Nếu bạn đang sử dụng Supervisor để chạy Reverb, bạn nên cập nhật cài đặt `minfds` của file `supervisor.conf` của server để đảm bảo Supervisor có thể mở các file cần thiết để xử lý các kết nối đến server Reverb của bạn:

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### Scaling

Nếu bạn cần xử lý nhiều kết nối hơn mức một server cho phép, bạn có thể scale server Reverb của mình theo chiều ngang. Sử dụng các khả năng publish / subscribe của Redis, Reverb có thể quản lý các kết nối trên nhiều servers. Khi một tin nhắn được nhận bởi một trong các servers Reverb của ứng dụng, server sẽ sử dụng Redis để publish tin nhắn đến tất cả các servers khác.

Để bật scale ngang, bạn nên đặt biến môi trường `REVERB_SCALING_ENABLED` thành `true` trong file cấu hình `.env` của ứng dụng:

```env
REVERB_SCALING_ENABLED=true
```

Tiếp theo, bạn nên có một server Redis chuyên dụng, trung tâm mà tất cả các servers Reverb sẽ giao tiếp. Reverb sẽ sử dụng [kết nối Redis mặc định được cấu hình cho ứng dụng](/docs/{{version}}/redis#configuration) của bạn để publish tin nhắn đến tất cả các servers Reverb của bạn.

Sau khi bạn đã bật tùy chọn scaling của Reverb và cấu hình một server Redis, bạn có thể chỉ cần gọi lệnh `reverb:start` trên nhiều servers có thể giao tiếp với server Redis của bạn. Các servers Reverb này nên được đặt sau một load balancer phân phối các requests đến đều nhau giữa các servers.

<a name="events"></a>
## Events

Reverb dispatch các sự kiện nội bộ trong vòng đời của kết nối và xử lý tin nhắn. Bạn có thể [lắng nghe các sự kiện này](/docs/{{version}}/events) để thực hiện các hành động khi các kết nối được quản lý hoặc tin nhắn được trao đổi.

Các sự kiện sau được dispatch bởi Reverb:

#### `Laravel\Reverb\Events\ChannelCreated`

Dispatched khi một channel được tạo. Điều này thường xảy ra khi kết nối đầu tiên subscribe vào một channel cụ thể. Sự kiện nhận instance `Laravel\Reverb\Protocols\Pusher\Channel`.

#### `Laravel\Reverb\Events\ChannelRemoved`

Dispatched khi một channel bị xóa. Điều này thường xảy ra khi kết nối cuối cùng unsubscribe từ một channel. Sự kiện nhận instance `Laravel\Reverb\Protocols\Pusher\Channel`.

#### `Laravel\Reverb\Events\ConnectionPruned`

Dispatched khi một kết nối cũ được cắt bỏ bởi server. Sự kiện nhận instance `Laravel\Reverb\Contracts\Connection`.

#### `Laravel\Reverb\Events\MessageReceived`

Dispatched khi một tin nhắn được nhận từ kết nối client. Sự kiện nhận instance `Laravel\Reverb\Contracts\Connection` và chuỗi thô `$message`.

#### `Laravel\Reverb\Events\MessageSent`

Dispatched khi một tin nhắn được gửi đến kết nối client. Sự kiện nhận instance `Laravel\Reverb\Contracts\Connection` và chuỗi thô `$message`.

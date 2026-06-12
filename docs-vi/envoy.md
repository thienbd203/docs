# Laravel Envoy

- [Introduction](#introduction)
- [Installation](#installation)
- [Writing Tasks](#writing-tasks)
    - [Defining Tasks](#defining-tasks)
    - [Multiple Servers](#multiple-servers)
    - [Setup](#setup)
    - [Variables](#variables)
    - [Stories](#stories)
    - [Hooks](#completion-hooks)
- [Running Tasks](#running-tasks)
    - [Confirming Task Execution](#confirming-task-execution)
- [Notifications](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## Introduction

[Laravel Envoy](https://github.com/laravel/envoy) là một công cụ để thực hiện các tác vụ phổ biến bạn chạy trên các servers từ xa của bạn. Sử dụng cú pháp phong cách [Blade](/docs/{{version}}/blade), bạn có thể dễ dàng thiết lập các tác vụ cho deployment, các lệnh Artisan, và hơn thế nữa. Hiện tại, Envoy chỉ hỗ trợ các hệ điều hành Mac và Linux. Tuy nhiên, hỗ trợ Windows có thể đạt được bằng cách sử dụng [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

<a name="installation"></a>
## Installation

Trước hết, cài đặt Envoy vào dự án của bạn bằng cách sử dụng trình quản lý package Composer:

```shell
composer require laravel/envoy --dev
```

Sau khi Envoy đã được cài đặt, binary Envoy sẽ có sẵn trong thư mục `vendor/bin` của ứng dụng:

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>
## Writing Tasks

<a name="defining-tasks"></a>
### Defining Tasks

Tasks là khối xây dựng cơ bản của Envoy. Tasks định nghĩa các lệnh shell nên thực thi trên các servers từ xa của bạn khi task được gọi. Ví dụ, bạn có thể định nghĩa một task thực thi lệnh `php artisan queue:restart` trên tất cả các servers queue worker của ứng dụng.

Tất cả các tasks Envoy của bạn nên được định nghĩa trong một file `Envoy.blade.php` ở gốc của ứng dụng. Đây là một ví dụ để giúp bạn bắt đầu:

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

Như bạn có thể thấy, một array `@servers` được định nghĩa ở đầu file, cho phép bạn tham chiếu các servers này thông qua tùy chọn `on` của các khai báo task của bạn. Khai báo `@servers` nên luôn được đặt trên một dòng duy nhất. Trong các khai báo `@task` của bạn, bạn nên đặt các lệnh shell nên thực thi trên các servers của bạn khi task được gọi.

<a name="local-tasks"></a>
#### Local Tasks

Bạn có thể buộc một script chạy trên máy tính cục bộ của bạn bằng cách chỉ định địa chỉ IP của server là `127.0.0.1`:

```blade
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>
#### Importing Envoy Tasks

Sử dụng directive `@import`, bạn có thể import các file Envoy khác để các stories và tasks của chúng được thêm vào của bạn. Sau khi các file đã được import, bạn có thể thực thi các tasks chúng chứa như thể chúng được định nghĩa trong file Envoy của chính bạn:

```blade
@import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>
### Multiple Servers

Envoy cho phép bạn dễ dàng chạy một task trên nhiều servers. Trước hết, thêm các servers bổ sung vào khai báo `@servers` của bạn. Mỗi server nên được gán một tên duy nhất. Sau khi bạn đã định nghĩa các servers bổ sung, bạn có thể liệt kê từng server trong array `on` của task:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="parallel-execution"></a>
#### Parallel Execution

Theo mặc định, tasks sẽ được thực thi trên mỗi server tuần tự. Nói cách khác, một task sẽ hoàn thành chạy trên server đầu tiên trước khi tiếp tục thực thi trên server thứ hai. Nếu bạn muốn chạy một task trên nhiều servers song song, thêm tùy chọn `parallel` vào khai báo task của bạn:

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="setup"></a>
### Setup

Đôi khi, bạn có thể cần thực thi mã PHP tùy ý trước khi chạy các tasks Envoy của bạn. Bạn có thể sử dụng directive `@setup` để định nghĩa một block mã PHP nên thực thi trước các tasks của bạn:

```php
@setup
    $now = new DateTime;
@endsetup
```

Nếu bạn cần require các file PHP khác trước khi task của bạn được thực thi, bạn có thể sử dụng directive `@include` ở đầu file `Envoy.blade.php` của bạn:

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

<a name="variables"></a>
### Variables

Nếu cần, bạn có thể chuyển các đối số đến các tasks Envoy bằng cách chỉ định chúng trên dòng lệnh khi gọi Envoy:

```shell
php vendor/bin/envoy run deploy --branch=master
```

Bạn có thể truy cập các tùy chọn trong các tasks của bạn bằng cách sử dụng cú pháp "echo" của Blade. Bạn cũng có thể định nghĩa các câu lệnh `if` và vòng lặp Blade trong các tasks của bạn. Ví dụ, hãy xác minh sự hiện diện của biến `$branch` trước khi thực thi lệnh `git pull`:

```blade
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

<a name="stories"></a>
### Stories

Stories nhóm một tập hợp các tasks dưới một tên duy nhất, thuận tiện. Ví dụ, một story `deploy` có thể chạy các tasks `update-code` và `install-dependencies` bằng cách liệt kê tên tasks trong định nghĩa của nó:

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

Sau khi story đã được viết, bạn có thể gọi nó theo cùng cách bạn sẽ gọi một task:

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>
### Hooks

Khi các tasks và stories chạy, một số hooks được thực thi. Các loại hook được hỗ trợ bởi Envoy là `@before`, `@after`, `@error`, `@success`, và `@finished`. Tất cả mã trong các hooks này được diễn giải là PHP và thực thi cục bộ, không phải trên các servers từ xa mà các tasks của bạn tương tác.

Bạn có thể định nghĩa bao nhiêu hook của mỗi loại tùy thích. Chúng sẽ được thực thi theo thứ tự chúng xuất hiện trong script Envoy của bạn.

<a name="hook-before"></a>
#### `@before`

Trước mỗi lần thực thi task, tất cả các hooks `@before` được đăng ký trong script Envoy của bạn sẽ thực thi. Các hooks `@before` nhận tên của task sẽ được thực thi:

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>
#### `@after`

Sau mỗi lần thực thi task, tất cả các hooks `@after` được đăng ký trong script Envoy của bạn sẽ thực thi. Các hooks `@after` nhận tên của task đã được thực thi:

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>
#### `@error`

Sau mỗi lần thất bại task (thoát với mã trạng thái lớn hơn `0`), tất cả các hooks `@error` được đăng ký trong script Envoy của bạn sẽ thực thi. Các hooks `@error` nhận tên của task đã được thực thi:

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>
#### `@success`

Nếu tất cả các tasks đã thực thi mà không có lỗi, tất cả các hooks `@success` được đăng ký trong script Envoy của bạn sẽ thực thi:

```blade
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>
#### `@finished`

Sau khi tất cả các tasks đã được thực thi (bất kể trạng thái thoát), tất cả các hooks `@finished` sẽ được thực thi. Các hooks `@finished` nhận mã trạng thái của task đã hoàn thành, có thể là `null` hoặc một `integer` lớn hơn hoặc bằng `0`:

```blade
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

<a name="running-tasks"></a>
## Running Tasks

Để chạy một task hoặc story được định nghĩa trong file `Envoy.blade.php` của ứng dụng, thực thi lệnh `run` của Envoy, chuyển tên của task hoặc story bạn muốn thực thi. Envoy sẽ thực thi task và hiển thị đầu ra từ các servers từ xa của bạn khi task đang chạy:

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>
### Confirming Task Execution

Nếu bạn muốn được nhắc xác nhận trước khi chạy một task nhất định trên các servers của bạn, bạn nên thêm directive `confirm` vào khai báo task của bạn. Tùy chọn này đặc biệt hữu ích cho các thao tác hủy diệt:

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>
## Notifications

<a name="slack"></a>
### Slack

Envoy hỗ trợ gửi thông báo đến [Slack](https://slack.com) sau khi mỗi task được thực thi. Directive `@slack` chấp nhận một URL hook Slack và một tên channel / người dùng. Bạn có thể truy xuất URL webhook của bạn bằng cách tạo một tích hợp "Incoming WebHooks" trong bảng điều khiển Slack của bạn.

Bạn nên chuyển toàn bộ URL webhook làm đối số đầu tiên được đưa cho directive `@slack`. Đối số thứ hai được đưa cho directive `@slack` nên là một tên channel (`#channel`) hoặc một tên người dùng (`@user`):

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

Theo mặc định, các thông báo Envoy sẽ gửi một tin nhắn đến channel thông báo mô tả task đã được thực thi. Tuy nhiên, bạn có thể ghi đè tin nhắn này bằng tin nhắn tùy chỉnh của chính bạn bằng cách chuyển đối số thứ ba cho directive `@slack`:

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>
### Discord

Envoy cũng hỗ trợ gửi thông báo đến [Discord](https://discord.com) sau khi mỗi task được thực thi. Directive `@discord` chấp nhận một URL hook Discord và một tin nhắn. Bạn có thể truy xuất URL webhook của bạn bằng cách tạo một "Webhook" trong Cài đặt Server và chọn channel nào webhook nên đăng lên. Bạn nên chuyển toàn bộ URL Webhook vào directive `@discord`:

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>
### Telegram

Envoy cũng hỗ trợ gửi thông báo đến [Telegram](https://telegram.org) sau khi mỗi task được thực thi. Directive `@telegram` chấp nhận một Bot ID Telegram và một Chat ID. Bạn có thể truy xuất Bot ID của bạn bằng cách tạo một bot mới bằng cách sử dụng [BotFather](https://t.me/botfather). Bạn có thể truy xuất một Chat ID hợp lệ bằng cách sử dụng [@username_to_id_bot](https://t.me/username_to_id_bot). Bạn nên chuyển toàn bộ Bot ID và Chat ID vào directive `@telegram`:

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>
### Microsoft Teams

Envoy cũng hỗ trợ gửi thông báo đến [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) sau khi mỗi task được thực thi. Directive `@microsoftTeams` chấp nhận một Teams Webhook (bắt buộc), một tin nhắn, màu chủ đề (success, info, warning, error), và một array các tùy chọn. Bạn có thể truy xuất Teams Webhook của bạn bằng cách tạo một [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) mới. API Teams có nhiều thuộc tính khác để tùy chỉnh hộp tin nhắn của bạn như tiêu đề, tóm tắt, và các phần. Bạn có thể tìm thêm thông tin trên [tài liệu Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message). Bạn nên chuyển toàn bộ URL Webhook vào directive `@microsoftTeams`:

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```

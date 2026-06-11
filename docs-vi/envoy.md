# Laravel Envoy

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Viết Tasks](#writing-tasks)
    - [Định nghĩa Tasks](#defining-tasks)
    - [Multiple Servers](#multiple-servers)
    - [Setup](#setup)
    - [Variables](#variables)
    - [Stories](#stories)
    - [Hooks](#completion-hooks)
- [Chạy Tasks](#running-tasks)
    - [Xác nhận Task Execution](#confirming-task-execution)
- [Notifications](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## Giới thiệu

[Laravel Envoy](https://github.com/laravel/envoy) là một công cụ để thực hiện các tasks phổ biến mà bạn chạy trên các remote servers của mình. Sử dụng cú pháp kiểu [Blade](/docs/{{version}}/blade), bạn có thể dễ dàng setup các tasks cho deployment, Artisan commands, và nhiều hơn nữa. Hiện tại, Envoy chỉ hỗ trợ các hệ điều hành Mac và Linux. Tuy nhiên, hỗ trợ Windows có thể đạt được bằng cách sử dụng [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10).

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt Envoy vào dự án của bạn bằng cách sử dụng Composer package manager:

```shell
composer require laravel/envoy --dev
```

Khi Envoy đã được cài đặt, binary Envoy sẽ có sẵn trong thư mục `vendor/bin` của ứng dụng:

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>
## Viết Tasks

<a name="defining-tasks"></a>
### Định nghĩa Tasks

Tasks là khối xây dựng cơ bản của Envoy. Tasks định nghĩa các shell commands nên thực thi trên các remote servers của bạn khi task được gọi. Ví dụ, bạn có thể định nghĩa một task thực thi command `php artisan queue:restart` trên tất cả các queue worker servers của ứng dụng.

Tất cả các Envoy tasks của bạn nên được định nghĩa trong file `Envoy.blade.php` tại thư mục gốc của ứng dụng. Đây là một ví dụ để bạn bắt đầu:

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

Như bạn có thể thấy, một mảng `@servers` được định nghĩa ở đầu file, cho phép bạn tham chiếu các servers này thông qua tùy chọn `on` của các khai báo task của bạn. Khai báo `@servers` nên luôn được đặt trên một dòng duy nhất. Trong các khai báo `@task` của bạn, bạn nên đặt các shell commands nên thực thi trên các servers của bạn khi task được gọi.

<a name="local-tasks"></a>
#### Local Tasks

Bạn có thể buộc một script chạy trên máy tính cục bộ của mình bằng cách chỉ định địa chỉ IP của server là `127.0.0.1`:

```blade
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>
#### Importing Envoy Tasks

Sử dụng directive `@import`, bạn có thể import các file Envoy khác để các stories và tasks của chúng được thêm vào của bạn. Sau khi các file đã được import, bạn có thể thực thi các tasks mà chúng chứa như thể chúng được định nghĩa trong file Envoy của chính bạn:

```blade
@import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>
### Multiple Servers

Envoy cho phép bạn dễ dàng chạy một task trên nhiều servers. Đầu tiên, thêm các servers bổ sung vào khai báo `@servers` của bạn. Mỗi server nên được gán một tên duy nhất. Khi bạn đã định nghĩa các servers bổ sung của mình, bạn có thể liệt kê từng server trong mảng `on` của task:

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

Theo mặc định, tasks sẽ được thực thi trên từng server một cách tuần tự. Nói cách khác, một task sẽ hoàn thành chạy trên server đầu tiên trước khi tiếp tục thực thi trên server thứ hai. Nếu bạn muốn chạy một task trên nhiều servers song song, thêm tùy chọn `parallel` vào khai báo task của bạn:

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

Đôi khi, bạn có thể cần thực thi code PHP tùy ý trước khi chạy các Envoy tasks của mình. Bạn có thể sử dụng directive `@setup` để định nghĩa một khối code PHP nên thực thi trước các tasks của bạn:

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

Nếu cần, bạn có thể truyền arguments cho Envoy tasks bằng cách chỉ định chúng trên command line khi gọi Envoy:

```shell
php vendor/bin/envoy run deploy --branch=master
```

Bạn có thể truy cập các options trong các tasks của mình bằng cách sử dụng cú pháp "echo" của Blade. Bạn cũng có thể định nghĩa các câu lệnh `if` và loops của Blade trong các tasks của mình. Ví dụ, hãy xác nhận sự hiện diện của biến `$branch` trước khi thực thi command `git pull`:

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

Stories nhóm một tập hợp các tasks dưới một tên duy nhất, thuận tiện. Ví dụ, một story `deploy` có thể chạy các tasks `update-code` và `install-dependencies` bằng cách liệt kê các tên task trong định nghĩa của nó:

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

Khi story đã được viết, bạn có thể gọi nó theo cùng cách bạn sẽ gọi một task:

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>
### Hooks

Khi tasks và stories chạy, một số hooks được thực thi. Các loại hook được hỗ trợ bởi Envoy là `@before`, `@after`, `@error`, `@success`, và `@finished`. Tất cả code trong các hooks này được hiểu là PHP và thực thi cục bộ, không phải trên các remote servers mà các tasks của bạn tương tác.

Bạn có thể định nghĩa bao nhiêu hook mỗi loại tùy thích. Chúng sẽ được thực thi theo thứ tự mà chúng xuất hiện trong script Envoy của bạn.

<a name="hook-before"></a>
#### `@before`

Trước khi mỗi task execution, tất cả các hooks `@before` được đăng ký trong script Envoy của bạn sẽ thực thi. Các hooks `@before` nhận tên của task sẽ được thực thi:

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>
#### `@after`

Sau khi mỗi task execution, tất cả các hooks `@after` được đăng ký trong script Envoy của bạn sẽ thực thi. Các hooks `@after` nhận tên của task đã được thực thi:

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>
#### `@error`

Sau khi mỗi task failure (thoát với status code lớn hơn `0`), tất cả các hooks `@error` được đăng ký trong script Envoy của bạn sẽ thực thi. Các hooks `@error` nhận tên của task đã được thực thi:

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

Sau khi tất cả các tasks đã được thực thi (bất kể exit status), tất cả các hooks `@finished` sẽ được thực thi. Các hooks `@finished` nhận status code của task đã hoàn thành, có thể là `null` hoặc một `integer` lớn hơn hoặc bằng `0`:

```blade
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

<a name="running-tasks"></a>
## Chạy Tasks

Để chạy một task hoặc story được định nghĩa trong file `Envoy.blade.php` của ứng dụng, thực thi command `run` của Envoy, truyền tên của task hoặc story bạn muốn thực thi. Envoy sẽ thực thi task và hiển thị output từ các remote servers của bạn khi task đang chạy:

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>
### Xác nhận Task Execution

Nếu bạn muốn được nhắc xác nhận trước khi chạy một task cụ thể trên các servers của mình, bạn nên thêm directive `confirm` vào khai báo task của bạn. Tùy chọn này đặc biệt hữu ích cho các operations phá hủy:

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

Envoy hỗ trợ gửi notifications đến [Slack](https://slack.com) sau khi mỗi task được thực thi. Directive `@slack` chấp nhận một Slack hook URL và một channel / user name. Bạn có thể lấy webhook URL của mình bằng cách tạo một integration "Incoming WebHooks" trong Slack control panel của bạn.

Bạn nên truyền toàn bộ webhook URL làm argument đầu tiên được đưa cho directive `@slack`. Argument thứ hai được đưa cho directive `@slack` nên là một tên channel (`#channel`) hoặc một tên user (`@user`):

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

Theo mặc định, Envoy notifications sẽ gửi một message đến notification channel mô tả task đã được thực thi. Tuy nhiên, bạn có thể ghi đè message này bằng custom message của riêng bạn bằng cách truyền argument thứ ba cho directive `@slack`:

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>
### Discord

Envoy cũng hỗ trợ gửi notifications đến [Discord](https://discord.com) sau khi mỗi task được thực thi. Directive `@discord` chấp nhận một Discord hook URL và một message. Bạn có thể lấy webhook URL của mình bằng cách tạo một "Webhook" trong Server Settings của bạn và chọn channel mà webhook nên post đến. Bạn nên truyền toàn bộ Webhook URL vào directive `@discord`:

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>
### Telegram

Envoy cũng hỗ trợ gửi notifications đến [Telegram](https://telegram.org) sau khi mỗi task được thực thi. Directive `@telegram` chấp nhận một Telegram Bot ID và một Chat ID. Bạn có thể lấy Bot ID của mình bằng cách tạo một bot mới sử dụng [BotFather](https://t.me/botfather). Bạn có thể lấy một Chat ID hợp lệ sử dụng [@username_to_id_bot](https://t.me/username_to_id_bot). Bạn nên truyền toàn bộ Bot ID và Chat ID vào directive `@telegram`:

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>
### Microsoft Teams

Envoy cũng hỗ trợ gửi notifications đến [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) sau khi mỗi task được thực thi. Directive `@microsoftTeams` chấp nhận một Teams Webhook (bắt buộc), một message, theme color (success, info, warning, error), và một mảng các options. Bạn có thể lấy Teams Webhook của mình bằng cách tạo một [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) mới. Teams API có nhiều thuộc tính khác để tùy chỉnh message box của bạn như title, summary, và sections. Bạn có thể tìm thêm thông tin trên [Microsoft Teams documentation](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message). Bạn nên truyền toàn bộ Webhook URL vào directive `@microsoftTeams`:

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```

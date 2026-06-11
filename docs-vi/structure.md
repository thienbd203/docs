# Directory Structure

- [Introduction](#introduction)
- [The Root Directory](#the-root-directory)
    - [The `app` Directory](#the-root-app-directory)
    - [The `bootstrap` Directory](#the-bootstrap-directory)
    - [The `config` Directory](#the-config-directory)
    - [The `database` Directory](#the-database-directory)
    - [The `public` Directory](#the-public-directory)
    - [The `resources` Directory](#the-resources-directory)
    - [The `routes` Directory](#the-routes-directory)
    - [The `storage` Directory](#the-storage-directory)
    - [The `tests` Directory](#the-tests-directory)
    - [The `vendor` Directory](#the-vendor-directory)
- [The App Directory](#the-app-directory)
    - [The `Broadcasting` Directory](#the-broadcasting-directory)
    - [The `Console` Directory](#the-console-directory)
    - [The `Events` Directory](#the-events-directory)
    - [The `Exceptions` Directory](#the-exceptions-directory)
    - [The `Http` Directory](#the-http-directory)
    - [The `Jobs` Directory](#the-jobs-directory)
    - [The `Listeners` Directory](#the-listeners-directory)
    - [The `Mail` Directory](#the-mail-directory)
    - [The `Models` Directory](#the-models-directory)
    - [The `Notifications` Directory](#the-notifications-directory)
    - [The `Policies` Directory](#the-policies-directory)
    - [The `Providers` Directory](#the-providers-directory)
    - [The `Rules` Directory](#the-rules-directory)

<a name="introduction"></a>
## Introduction

Cấu trúc ứng dụng Laravel mặc định được thiết kế để cung cấp một điểm khởi đầu tuyệt vời cho cả ứng dụng lớn và nhỏ. Nhưng bạn có thể tự do tổ chức ứng dụng của mình theo cách bạn thích. Laravel áp dụng gần như không có hạn chế nào về vị trí của bất kỳ class nào - miễn là Composer có thể autoload class đó.

<a name="the-root-directory"></a>
## The Root Directory

<a name="the-root-app-directory"></a>
### The App Directory

Thư mục `app` chứa code cốt lõi của ứng dụng. Chúng ta sẽ khám phá thư mục này chi tiết hơn sau; tuy nhiên, hầu hết tất cả các class trong ứng dụng của bạn sẽ nằm trong thư mục này.

<a name="the-bootstrap-directory"></a>
### The Bootstrap Directory

Thư mục `bootstrap` chứa file `app.php` giúp khởi động framework. Thư mục này cũng chứa thư mục `cache` chứa các file được tạo bởi framework để tối ưu hóa hiệu suất như file cache route và services.

<a name="the-config-directory"></a>
### The Config Directory

Thư mục `config`, như tên gọi, chứa tất cả các file cấu hình của ứng dụng. Ý tưởng tốt là đọc qua tất cả các file này và làm quen với tất cả các tùy chọn có sẵn cho bạn.

<a name="the-database-directory"></a>
### The Database Directory

Thư mục `database` chứa database migrations, model factories, và seeds của bạn. Nếu bạn muốn, bạn cũng có thể sử dụng thư mục này để giữ một SQLite database.

<a name="the-public-directory"></a>
### The Public Directory

Thư mục `public` chứa file `index.php`, là entry point cho tất cả các request đi vào ứng dụng của bạn và cấu hình autoloading. Thư mục này cũng chứa các assets của bạn như hình ảnh, JavaScript, và CSS.

<a name="the-resources-directory"></a>
### The Resources Directory

Thư mục `resources` chứa [views](/docs/{{version}}/views) của bạn cũng như các assets thô, chưa biên dịch như CSS hoặc JavaScript.

<a name="the-routes-directory"></a>
### The Routes Directory

Thư mục `routes` chứa tất cả các định nghĩa route cho ứng dụng của bạn. Theo mặc định, hai file route được bao gồm với Laravel: `web.php` và `console.php`.

File `web.php` chứa các route mà Laravel đặt trong middleware group `web`, cung cấp session state, CSRF protection, và cookie encryption. Nếu ứng dụng của bạn không cung cấp một RESTful API stateless thì tất cả các route của bạn có thể sẽ được định nghĩa trong file `web.php`.

File `console.php` là nơi bạn có thể định nghĩa tất cả các console commands dựa trên closure. Mỗi closure được gắn với một command instance cho phép một cách tiếp cận đơn giản để tương tác với các method IO của mỗi command. Mặc dù file này không định nghĩa HTTP routes, nó định nghĩa các entry point (routes) dựa trên console vào ứng dụng của bạn. Bạn cũng có thể [schedule](/docs/{{version}}/scheduling) các tasks trong file `console.php`.

Tùy chọn, bạn có thể cài đặt các file route bổ sung cho API routes (`api.php`) và broadcasting channels (`channels.php`), thông qua các command Artisan `install:api` và `install:broadcasting`.

File `api.php` chứa các route được thiết kế để stateless, vì vậy các request đi vào ứng dụng thông qua các route này được thiết kế để được xác thực [qua tokens](/docs/{{version}}/sanctum) và sẽ không có quyền truy cập vào session state.

File `channels.php` là nơi bạn có thể đăng ký tất cả các [event broadcasting](/docs/{{version}}/broadcasting) channels mà ứng dụng của bạn hỗ trợ.

<a name="the-storage-directory"></a>
### The Storage Directory

Thư mục `storage` chứa logs, Blade templates đã biên dịch, sessions dựa trên file, file caches, và các file khác được tạo bởi framework. Thư mục này được chia thành các thư mục `app`, `framework`, và `logs`. Thư mục `app` có thể được sử dụng để lưu trữ bất kỳ file nào được tạo bởi ứng dụng của bạn. Thư mục `framework` được sử dụng để lưu trữ các file và cache được tạo bởi framework. Cuối cùng, thư mục `logs` chứa các file log của ứng dụng.

Thư mục `storage/app/public` có thể được sử dụng để lưu trữ các file được tạo bởi người dùng, chẳng hạn như profile avatars, nên có thể truy cập công khai. Bạn nên tạo một symbolic link tại `public/storage` trỏ đến thư mục này. Bạn có thể tạo link bằng command Artisan `php artisan storage:link`.

<a name="the-tests-directory"></a>
### The Tests Directory

Thư mục `tests` chứa các automated tests của bạn. Ví dụ [Pest](https://pestphp.com) hoặc [PHPUnit](https://phpunit.de/) unit tests và feature tests được cung cấp sẵn. Mỗi test class nên có hậu tố là `Test`. Bạn có thể chạy tests của mình bằng các command `/vendor/bin/pest` hoặc `/vendor/bin/phpunit`. Hoặc, nếu bạn muốn một đại diện chi tiết và đẹp hơn của kết quả test, bạn có thể chạy tests bằng command Artisan `php artisan test`.

<a name="the-vendor-directory"></a>
### The Vendor Directory

Thư mục `vendor` chứa các dependencies [Composer](https://getcomposer.org) của bạn.

<a name="the-app-directory"></a>
## The App Directory

Phần lớn ứng dụng của bạn nằm trong thư mục `app`. Theo mặc định, thư mục này được đặt namespace dưới `App` và được autoload bởi Composer sử dụng [PSR-4 autoloading standard](https://www.php-fig.org/psr/psr-4/).

Theo mặc định, thư mục `app` chứa các thư mục `Http`, `Models`, và `Providers`. Tuy nhiên, theo thời gian, nhiều thư mục khác sẽ được tạo bên trong thư mục app khi bạn sử dụng các command make Artisan để tạo classes. Ví dụ, thư mục `app/Console` sẽ không tồn tại cho đến khi bạn thực thi command Artisan `make:command` để tạo một command class.

Cả thư mục `Console` và `Http` đều được giải thích thêm trong các phần tương ứng của chúng bên dưới, nhưng hãy coi thư mục `Console` và `Http` như cung cấp một API vào cốt lõi của ứng dụng. Giao thức HTTP và CLI đều là các cơ chế để tương tác với ứng dụng, nhưng không thực sự chứa logic ứng dụng. Nói cách khác, chúng là hai cách để phát hành commands cho ứng dụng. Thư mục `Console` chứa tất cả các Artisan commands của bạn, trong khi thư mục `Http` chứa controllers, middleware, và requests của bạn.

> [!NOTE]
> Nhiều class trong thư mục `app` có thể được tạo bởi Artisan thông qua các commands. Để xem xét các commands có sẵn, hãy chạy command `php artisan list make` trong terminal của bạn.

<a name="the-broadcasting-directory"></a>
### The Broadcasting Directory

Thư mục `Broadcasting` chứa tất cả các broadcast channel classes cho ứng dụng của bạn. Các class này được tạo bằng command `make:channel`. Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn khi bạn tạo channel đầu tiên. Để tìm hiểu thêm về channels, hãy xem tài liệu về [event broadcasting](/docs/{{version}}/broadcasting).

<a name="the-console-directory"></a>
### The Console Directory

Thư mục `Console` chứa tất cả các Artisan commands tùy chỉnh cho ứng dụng của bạn. Các commands này có thể được tạo bằng command `make:command`.

<a name="the-events-directory"></a>
### The Events Directory

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn bởi các command Artisan `event:generate` và `make:event`. Thư mục `Events` chứa [event classes](/docs/{{version}}/events). Events có thể được sử dụng để thông báo cho các phần khác của ứng dụng rằng một hành động đã xảy ra, cung cấp một sự linh hoạt và decoupling lớn.

<a name="the-exceptions-directory"></a>
### The Exceptions Directory

Thư mục `Exceptions` chứa tất cả các exceptions tùy chỉnh cho ứng dụng của bạn. Các exceptions này có thể được tạo bằng command `make:exception`.

<a name="the-http-directory"></a>
### The Http Directory

Thư mục `Http` chứa controllers, middleware, và form requests của bạn. Hầu hết tất cả logic để xử lý các request đi vào ứng dụng sẽ được đặt trong thư mục này.

<a name="the-jobs-directory"></a>
### The Jobs Directory

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn nếu bạn thực thi command Artisan `make:job`. Thư mục `Jobs` chứa [queueable jobs](/docs/{{version}}/queues) cho ứng dụng của bạn. Jobs có thể được queued bởi ứng dụng của bạn hoặc chạy đồng bộ trong request lifecycle hiện tại. Jobs chạy đồng bộ trong request hiện tại đôi khi được gọi là "commands" vì chúng là một triển khai của [command pattern](https://en.wikipedia.org/wiki/Command_pattern).

<a name="the-listeners-directory"></a>
### The Listeners Directory

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn nếu bạn thực thi command Artisan `event:generate` hoặc `make:listener`. Thư mục `Listeners` chứa các class xử lý [events](/docs/{{version}}/events) của bạn. Event listeners nhận một event instance và thực hiện logic để phản ứng với event được kích hoạt. Ví dụ, một event `UserRegistered` có thể được xử lý bởi một listener `SendWelcomeEmail`.

<a name="the-mail-directory"></a>
### The Mail Directory

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn nếu bạn thực thi command Artisan `make:mail`. Thư mục `Mail` chứa tất cả các [classes đại diện cho emails](/docs/{{version}}/mail) được gửi bởi ứng dụng của bạn. Mail objects cho phép bạn đóng gói tất cả logic của việc xây dựng một email trong một class đơn giản, đơn lẻ có thể được gửi bằng method `Mail::send`.

<a name="the-models-directory"></a>
### The Models Directory

Thư mục `Models` chứa tất cả các [Eloquent model classes](/docs/{{version}}/eloquent) của bạn. Eloquent ORM được bao gồm với Laravel cung cấp một triển khai ActiveRecord đẹp và đơn giản để làm việc với database. Mỗi database table có một "Model" tương ứng được sử dụng để tương tác với table đó. Models cho phép bạn query dữ liệu trong các table của bạn, cũng như chèn các bản ghi mới vào table.

<a name="the-notifications-directory"></a>
### The Notifications Directory

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn nếu bạn thực thi command Artisan `make:notification`. Thư mục `Notifications` chứa tất cả các [notifications](/docs/{{version}}/notifications) "transactional" được gửi bởi ứng dụng, chẳng hạn như các notifications đơn giản về các sự kiện xảy ra trong ứng dụng. Tính năng notification của Laravel trừu tượng hóa việc gửi notifications qua nhiều drivers khác nhau như email, Slack, SMS, hoặc được lưu trữ trong database.

<a name="the-policies-directory"></a>
### The Policies Directory

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn nếu bạn thực thi command Artisan `make:policy`. Thư mục `Policies` chứa các [authorization policy classes](/docs/{{version}}/authorization) cho ứng dụng của bạn. Policies được sử dụng để xác định xem người dùng có thể thực hiện một hành động đã cho đối với một resource hay không.

<a name="the-providers-directory"></a>
### The Providers Directory

Thư mục `Providers` chứa tất cả các [service providers](/docs/{{version}}/providers) cho ứng dụng của bạn. Service providers khởi động ứng dụng của bạn bằng cách binding services trong service container, đăng ký events, hoặc thực hiện bất kỳ tasks nào khác để chuẩn bị ứng dụng cho các request sắp tới.

Trong một ứng dụng Laravel mới, thư mục này sẽ đã chứa `AppServiceProvider`. Bạn có thể tự do thêm các providers của riêng mình vào thư mục này khi cần.

<a name="the-rules-directory"></a>
### The Rules Directory

Thư mục này không tồn tại theo mặc định, nhưng sẽ được tạo cho bạn nếu bạn thực thi command Artisan `make:rule`. Thư mục `Rules` chứa các custom validation rule objects cho ứng dụng của bạn. Rules được sử dụng để đóng gói logic validation phức tạp trong một object đơn giản. Để biết thêm thông tin, hãy xem [validation documentation](/docs/{{version}}/validation).

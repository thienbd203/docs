# Cài đặt

- [Giới thiệu Laravel](#meet-laravel)
    - [Tại sao chọn Laravel?](#why-laravel)
- [Tạo Ứng Dụng Laravel](#creating-a-laravel-project)
    - [Bắt Đầu Bằng AI](#getting-started-using-ai)
    - [Cài Đặt PHP và Laravel Installer](#installing-php)
    - [Tạo Ứng Dụng](#creating-an-application)
- [Cấu Hình Ban Đầu](#initial-configuration)
    - [Cấu Hình Dựa Trên Environment](#environment-based-configuration)
    - [Database và Migrations](#databases-and-migrations)
    - [Cấu Hình Thư Mục](#directory-configuration)
- [Cài Đặt Sử Dụng Herd](#installation-using-herd)
    - [Herd trên macOS](#herd-on-macos)
    - [Herd trên Windows](#herd-on-windows)
- [Hỗ Trợ IDE](#ide-support)
- [Laravel và AI](#laravel-and-ai)
    - [Cài Đặt Laravel Boost](#installing-laravel-boost)
- [Bước Tiếp Theo](#next-steps)
    - [Laravel Framework Full Stack](#laravel-the-fullstack-framework)
    - [Laravel API Backend](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## Giới thiệu Laravel

Laravel là một web application framework với cú pháp rõ ràng và tinh tế. Một web framework cung cấp cấu trúc và điểm bắt đầu để tạo ứng dụng của bạn, cho phép bạn tập trung vào việc tạo ra những điều tuyệt vời trong khi chúng tôi lo chi tiết kỹ thuật.

Laravel nỗ lực cung cấp trải nghiệm developer tuyệt vời với các tính năng mạnh mẽ như dependency injection toàn diện, database abstraction layer expressive, queues và scheduled jobs, unit và integration testing, và nhiều hơn nữa.

Dù bạn mới làm quen với PHP web frameworks hay đã có nhiều năm kinh nghiệm, Laravel là một framework có thể phát triển cùng bạn. Chúng tôi sẽ giúp bạn thực hiện những bước đầu tiên như một web developer hoặc thúc đẩy chuyên môn của bạn lên tầm cao mới. Chúng tôi không thể chờ đợi để xem bạn xây dựng những gì.

<a name="why-laravel"></a>
### Tại sao chọn Laravel?

Có nhiều công cụ và framework khác nhau có sẵn để bạn sử dụng khi xây dựng web application. Tuy nhiên, chúng tôi tin rằng Laravel là lựa chọn tốt nhất để xây dựng các modern, full-stack web applications.

#### Một Framework Tiến bộ

Chúng tôi thích gọi Laravel là một "progressive" framework. Điều đó có nghĩa là Laravel phát triển cùng bạn. Nếu bạn vừa bắt đầu bước vào thế giới web development, thư viện tài liệu khổng lồ, guides, và [video tutorials](https://laracasts.com) của Laravel sẽ giúp bạn học hỏi mà không bị quá tải.

Nếu bạn là một senior developer, Laravel cung cấp các công cụ mạnh mẽ cho [dependency injection](/docs/{{version}}/container), [unit testing](/docs/{{version}}/testing), [queues](/docs/{{version}}/queues), [real-time events](/docs/{{version}}/broadcasting), và nhiều hơn nữa. Laravel được tinh chỉnh để xây dựng các professional web applications và sẵn sàng xử lý enterprise workloads.

#### Một Framework Có Khả Năng Mở Rộng

Laravel có khả năng mở rộng tuyệt vời. Nhờ tính chất scaling-friendly của PHP và hỗ trợ tích hợp sẵn của Laravel cho các hệ thống cache phân tán nhanh như Redis, horizontal scaling với Laravel trở nên dễ dàng. Thực tế, các ứng dụng Laravel đã được mở rộng dễ dàng để xử lý hàng trăm triệu requests mỗi tháng.

Cần mở rộng cực độ? Các nền tảng như [Laravel Cloud](https://cloud.laravel.com) cho phép bạn chạy Laravel application của mình ở quy trình gần như vô hạn.

#### Framework Sẵn Sàng Cho AI Agent

Các conventions có chủ kiến và cấu trúc được định nghĩa rõ ràng của Laravel làm cho nó trở thành một framework lý tưởng cho [AI assisted development](/docs/{{version}}/ai) sử dụng các công cụ như Cursor và Claude Code. Khi bạn yêu cầu AI agent thêm một controller, nó biết chính xác nơi để đặt nó. Khi bạn cần một migration mới, các naming conventions và vị trí file có thể dự đoán được. Sự nhất quán này loại bỏ sự đoán thường làm khó khăn các công cụ AI trong các framework linh hoạt hơn.

Ngoài việc tổ chức file, cú pháp expressive và tài liệu toàn diện của Laravel cung cấp cho AI agents context họ cần để tạo ra code chính xác và idiomatic. Các tính năng như Eloquent relationships, form requests, và middleware tuân theo các patterns mà agents có thể hiểu và replicate một cách đáng tin cậy. Kết quả là AI-generated code trông giống như được viết bởi một Laravel developer dày dạn kinh nghiệm, không phải được ghép lại từ các generic PHP snippets.

Để tìm hiểu thêm về lý do Laravel là lựa chọn hoàn hảo cho AI assisted development, hãy xem tài liệu của chúng tôi về [agentic development](/docs/{{version}}/ai).

#### Một Framework Cộng Đồng

Laravel kết hợp các packages tốt nhất trong PHP ecosystem để cung cấp framework mạnh mẽ và thân thiện với developer nhất hiện có. Ngoài ra, hàng ngàn developer tài năng từ khắp nơi trên thế giới đã [đóng góp vào framework](https://github.com/laravel/framework). Biết đâu, có thể bạn sẽ trở thành một Laravel contributor.

<a name="creating-a-laravel-project"></a>
## Tạo Ứng Dụng Laravel

<a name="getting-started-using-ai"></a>
### Bắt Đầu Bằng AI

Nếu bạn đang sử dụng AI coding agent như [Claude Code](https://docs.anthropic.com/en/docs/claude-code) hoặc [OpenCode](https://opencode.ai), bạn có thể bắt đầu với một prompt cung cấp cho agent một Laravel-specific playbook trước khi nó chạm vào project của bạn.

Prompt dưới đây cho agent biết nơi tìm hướng dẫn cài đặt của Laravel, điều gì cần ưu tiên, và cách tạo sensible defaults khi bạn chưa đưa ra lựa chọn. Paste prompt này vào agent của bạn để bắt đầu:

```text
I'm building a new Laravel application.

Fetch and follow the instructions from https://laravel.com/for/agents. Treat the returned Markdown as the source of truth for how to install and set up Laravel in this session.
```

Sau khi agent đọc hướng dẫn, nó sẽ hướng dẫn bạn từng bước và giữ setup phù hợp với defaults của Laravel.

<a name="installing-php"></a>
### Cài Đặt PHP và Laravel Installer

Trước khi tạo Laravel application đầu tiên của bạn, hãy đảm bảo rằng local machine của bạn đã cài đặt [PHP](https://php.net), [Composer](https://getcomposer.org), và [Laravel installer](https://github.com/laravel/installer). Ngoài ra, bạn nên cài đặt [Node và NPM](https://nodejs.org) hoặc [Bun](https://bun.sh/) để bạn có thể compile frontend assets của application.

Nếu bạn chưa cài đặt PHP và Composer trên local machine, các commands sau sẽ cài đặt PHP, Composer, và Laravel installer trên macOS, Windows, hoặc Linux:

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.5)"
```

```shell tab=Windows PowerShell
# Run as administrator...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.5'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.5)"
```

Sau khi chạy một trong các commands trên, bạn nên restart terminal session của mình. Để update PHP, Composer, và Laravel installer sau khi cài đặt chúng qua `php.new`, bạn có thể chạy lại command trong terminal của mình.

Nếu bạn đã cài đặt PHP và Composer, bạn có thể cài đặt Laravel installer qua Composer:

```shell
composer global require laravel/installer
```

> [!NOTE]
> Để có trải nghiệm cài đặt và quản lý PHP đồ họa đầy đủ tính năng, hãy xem [Laravel Herd](#installation-using-herd).

<a name="creating-an-application"></a>
### Tạo Ứng Dụng

Sau khi bạn đã cài đặt PHP, Composer, và Laravel installer, bạn đã sẵn sàng tạo một Laravel application mới. Laravel installer sẽ nhắc bạn chọn testing framework, database, và starter kit ưa thích:

```shell
laravel new example-app
```

Sau khi application đã được tạo, bạn có thể bắt đầu local development server, queue worker, và Vite development server của Laravel sử dụng `dev` Composer script:

```shell
cd example-app
npm install && npm run build
composer run dev
```

Sau khi bạn đã bắt đầu development server, application của bạn sẽ có thể truy cập trong web browser tại [http://localhost:8000](http://localhost:8000). Tiếp theo, bạn đã sẵn sàng [bắt đầu các bước tiếp theo vào Laravel ecosystem](#next-steps). Tất nhiên, bạn cũng có thể muốn [configure a database](#databases-and-migrations).

> [!NOTE]
> Nếu bạn muốn có một khởi đầu thuận lợi khi phát triển Laravel application của mình, hãy cân nhắc sử dụng một trong [starter kits](/docs/{{version}}/starter-kits) của chúng tôi. Starter kits của Laravel cung cấp backend và frontend authentication scaffolding cho Laravel application mới của bạn.

<a name="initial-configuration"></a>
## Cấu Hình Ban Đầu

Tất cả các configuration files cho Laravel framework được lưu trữ trong thư mục `config`. Mỗi option đều được tài liệu hóa, vì vậy hãy tự do xem qua các files và làm quen với các options có sẵn cho bạn.

Laravel cần gần như không có additional configuration out of the box. Bạn được tự do bắt đầu phát triển! Tuy nhiên, bạn có thể muốn xem lại file `config/app.php` và tài liệu của nó. Nó chứa một số options như `url` và `locale` mà bạn có thể muốn thay đổi theo application của mình.

<a name="environment-based-configuration"></a>
### Cấu Hình Dựa Trên Environment

Vì nhiều giá trị configuration option của Laravel có thể khác nhau tùy thuộc vào việc application của bạn đang chạy trên local machine hay trên production web server, nhiều giá trị configuration quan trọng được định nghĩa sử dụng file `.env` tồn tại ở root của application.

File `.env` của bạn không nên được commit vào source control của application, vì mỗi developer / server sử dụng application của bạn có thể yêu cầu một environment configuration khác nhau. Hơn nữa, điều này sẽ là một security risk trong trường hợp kẻ xâm nhập có được quyền truy cập vào source control repository của bạn, vì bất kỳ sensitive credentials nào sẽ bị expose.

> [!NOTE]
> Để biết thêm thông tin về file `.env` và environment based configuration, hãy xem full [configuration documentation](/docs/{{version}}/configuration#environment-configuration).

<a name="databases-and-migrations"></a>
### Database và Migrations

Bây giờ bạn đã tạo Laravel application của mình, có lẽ bạn muốn lưu trữ một số dữ liệu trong database. Theo mặc định, file configuration `.env` của application chỉ định rằng Laravel sẽ tương tác với SQLite database.

Trong quá trình tạo application, Laravel đã tạo file `database/database.sqlite` cho bạn, và chạy các migrations cần thiết để tạo các database tables của application.

Nếu bạn thích sử dụng database driver khác như MySQL hoặc PostgreSQL, bạn có thể update file configuration `.env` của mình để sử dụng database phù hợp. Ví dụ, nếu bạn muốn sử dụng MySQL, update các biến `DB_*` trong file configuration `.env` của bạn như sau:

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

Nếu bạn chọn sử dụng database khác ngoài SQLite, bạn sẽ cần tạo database và chạy [database migrations](/docs/{{version}}/migrations) của application:

```shell
php artisan migrate
```

> [!NOTE]
> Nếu bạn đang phát triển trên macOS hoặc Windows và cần cài đặt MySQL, PostgreSQL, hoặc Redis local, hãy cân nhắc sử dụng [Herd Pro](https://herd.laravel.com/#plans) hoặc [DBngin](https://dbngin.com/).

<a name="directory-configuration"></a>
### Directory Configuration

Laravel should always be served out of the root of the "web directory" configured for your web server. You should not attempt to serve a Laravel application out of a subdirectory of the "web directory". Attempting to do so could expose sensitive files present within your application.

<a name="installation-using-herd"></a>
## Installation Using Herd

[Laravel Herd](https://herd.laravel.com) is a blazing fast, native Laravel and PHP development environment for macOS and Windows. Herd includes everything you need to get started with Laravel development, including PHP and Nginx.

Once you install Herd, you're ready to start developing with Laravel. Herd includes command line tools for `php`, `composer`, `laravel`, `expose`, `node`, `npm`, and `nvm`.

> [!NOTE]
> [Herd Pro](https://herd.laravel.com/#plans) augments Herd with additional powerful features, such as the ability to create and manage local MySQL, Postgres, and Redis databases, as well as local mail viewing and log monitoring.

<a name="herd-on-macos"></a>
### Herd on macOS

If you develop on macOS, you can download the Herd installer from the [Herd website](https://herd.laravel.com). The installer automatically downloads the latest version of PHP and configures your Mac to always run [Nginx](https://www.nginx.com/) in the background.

Herd for macOS uses [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) to support "parked" directories. Any Laravel application in a parked directory will automatically be served by Herd. By default, Herd creates a parked directory at `~/Herd` and you can access any Laravel application in this directory on the `.test` domain using its directory name.

After installing Herd, the fastest way to create a new Laravel application is using the Laravel CLI, which is bundled with Herd:

```shell
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

Of course, you can always manage your parked directories and other PHP settings via Herd's UI, which can be opened from the Herd menu in your system tray.

You can learn more about Herd by checking out the [Herd documentation](https://herd.laravel.com/docs).

<a name="herd-on-windows"></a>
### Herd on Windows

You can download the Windows installer for Herd on the [Herd website](https://herd.laravel.com/windows). After the installation finishes, you can start Herd to complete the onboarding process and access the Herd UI for the first time.

The Herd UI is accessible by left-clicking on Herd's system tray icon. A right-click opens the quick menu with access to all tools that you need on a daily basis.

During installation, Herd creates a "parked" directory in your home directory at `%USERPROFILE%\Herd`. Any Laravel application in a parked directory will automatically be served by Herd, and you can access any Laravel application in this directory on the `.test` domain using its directory name.

After installing Herd, the fastest way to create a new Laravel application is using the Laravel CLI, which is bundled with Herd. To get started, open Powershell and run the following commands:

```shell
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

You can learn more about Herd by checking out the [Herd documentation for Windows](https://herd.laravel.com/docs/windows).

<a name="ide-support"></a>
## IDE Support

You are free to use any code editor you wish when developing Laravel applications. If you're looking for lightweight and extensible editors, [VS Code](https://code.visualstudio.com) or [Cursor](https://cursor.com) combined with the official [Laravel VS Code Extension](https://marketplace.visualstudio.com/items?itemName=laravel.vscode-laravel) offers excellent Laravel support with features like syntax highlighting, snippets, artisan command integration, and smart autocompletion for Eloquent models, routes, middleware, assets, config, and Inertia.js.

For extensive and robust support of Laravel, take a look at [PhpStorm](https://www.jetbrains.com/phpstorm/laravel/?utm_source=laravel.com&utm_medium=link&utm_campaign=laravel-2025&utm_content=partner&ref=laravel-2025), a JetBrains IDE. PhpStorm's built-in Laravel framework support includes Blade templates, smart autocompletion for Eloquent models, routes, views, translations, and components, along with powerful code generation and navigation across Laravel projects.

For those seeking a cloud-based development experience, [Firebase Studio](https://firebase.studio/) provides instant access to building with Laravel directly in your browser. With zero setup required, Firebase Studio makes it easy to start building Laravel applications from any device.

<a name="laravel-and-ai"></a>
## Laravel and AI

[Laravel Boost](https://github.com/laravel/boost) is a powerful tool that bridges the gap between AI coding agents and Laravel applications. Boost provides AI agents with Laravel-specific context, tools, and guidelines so they can generate more accurate, version-specific code that follows Laravel conventions.

When you install Boost in your Laravel application, AI agents gain access to over 15 specialized tools including the ability to know which packages you are using, query your database, search the Laravel documentation, read browser logs, generate tests, and execute code via Tinker.

In addition, Boost gives AI agents access to over 17,000 pieces of vectorized Laravel ecosystem documentation, specific to your installed package versions. This means agents can provide guidance targeted to the exact versions your project uses.

Boost also includes Laravel-maintained AI guidelines that help agents to follow framework conventions, write appropriate tests, and avoid common pitfalls when generating Laravel code.

<a name="installing-laravel-boost"></a>
### Installing Laravel Boost

Boost can be installed in Laravel 10, 11, 12, and 13 applications running PHP 8.1 or higher. To get started, install Boost as a development dependency:

```shell
composer require laravel/boost --dev
```

Once installed, run the interactive installer:

```shell
php artisan boost:install
```

The installer will auto-detect your IDE and AI agents, allowing you to opt into the features that make sense for your project. Boost respects existing project conventions and doesn't force opinionated style rules by default.

> [!NOTE]
> To learn more about Boost, check out the [Laravel Boost repository on GitHub](https://github.com/laravel/boost).

<a name="adding-custom-ai-guidelines"></a>
#### Adding Custom AI Guidelines

To augment Laravel Boost with your own custom AI guidelines, add `.blade.php` or `.md` files to your application's `.ai/guidelines/*` directory. These files will automatically be included with Laravel Boost's guidelines when you run `boost:install`.

<a name="next-steps"></a>
## Bước Tiếp Theo

Bây giờ bạn đã tạo Laravel application của mình, có lẽ bạn đang thắc mắc nên học gì tiếp theo. Trước hết, chúng tôi khuyến nghị mạnh mẽ việc làm quen với cách Laravel hoạt động bằng cách đọc tài liệu sau:

<div class="content-list" markdown="1">

- [Request Lifecycle](/docs/{{version}}/lifecycle)
- [Configuration](/docs/{{version}}/configuration)
- [Directory Structure](/docs/{{version}}/structure)
- [Frontend](/docs/{{version}}/frontend)
- [Service Container](/docs/{{version}}/container)
- [Facades](/docs/{{version}}/facades)

</div>

Cách bạn muốn sử dụng Laravel cũng sẽ quyết định các bước tiếp theo trong hành trình của bạn. Có nhiều cách để sử dụng Laravel, và chúng tôi sẽ khám phá hai primary use cases cho framework dưới đây.

<a name="laravel-the-fullstack-framework"></a>
### Laravel Framework Full Stack

Laravel có thể phục vụ như một full stack framework. Theo "full stack" framework chúng tôi có nghĩa là bạn sẽ sử dụng Laravel để route requests đến application của bạn và render frontend của bạn qua [Blade templates](/docs/{{version}}/blade) hoặc một single-page application hybrid technology như [Inertia](https://inertiajs.com). Đây là cách phổ biến nhất để sử dụng Laravel framework, và theo ý kiến của chúng tôi, cách productive nhất để sử dụng Laravel.

Nếu đây là cách bạn kế hoạch sử dụng Laravel, bạn có thể muốn xem tài liệu của chúng tôi về [frontend development](/docs/{{version}}/frontend), [routing](/docs/{{version}}/routing), [views](/docs/{{version}}/views), hoặc [Eloquent ORM](/docs/{{version}}/eloquent). Ngoài ra, bạn có thể quan tâm đến việc học về community packages như [Livewire](https://livewire.laravel.com) và [Inertia](https://inertiajs.com). Các packages này cho phép bạn sử dụng Laravel như một full-stack framework trong khi tận hưởng nhiều UI benefits được cung cấp bởi single-page JavaScript applications.

Nếu bạn đang sử dụng Laravel như một full stack framework, chúng tôi cũng khuyến nghị mạnh mẽ bạn học cách compile CSS và JavaScript của application bằng [Vite](/docs/{{version}}/vite).

> [!NOTE]
> Nếu bạn muốn có một khởi đầu thuận lợi khi xây dựng application, hãy xem một trong [application starter kits](/docs/{{version}}/starter-kits) chính thức của chúng tôi.

<a name="laravel-the-api-backend"></a>
### Laravel API Backend

Laravel cũng có thể phục vụ như một API backend cho JavaScript single-page application hoặc mobile application. Ví dụ, bạn có thể sử dụng Laravel như một API backend cho [Next.js](https://nextjs.org) application của bạn. Trong context này, bạn có thể sử dụng Laravel để cung cấp [authentication](/docs/{{version}}/sanctum) và data storage / retrieval cho application của bạn, trong khi cũng tận dụng các powerful services của Laravel như queues, emails, notifications, và nhiều hơn nữa.

Nếu đây là cách bạn kế hoạch sử dụng Laravel, bạn có thể muốn xem tài liệu của chúng tôi về [routing](/docs/{{version}}/routing), [Laravel Sanctum](/docs/{{version}}/sanctum), và [Eloquent ORM](/docs/{{version}}/eloquent).

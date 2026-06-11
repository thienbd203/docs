# Laravel Valet

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Nâng cấp Valet](#upgrading-valet)
- [Serving Sites](#serving-sites)
    - [Lệnh "Park"](#the-park-command)
    - [Lệnh "Link"](#the-link-command)
    - [Bảo vệ Sites Với TLS](#securing-sites)
    - [Serving một Default Site](#serving-a-default-site)
    - [Per-Site PHP Versions](#per-site-php-versions)
- [Chia sẻ Sites](#sharing-sites)
    - [Chia sẻ Sites trên Local Network của bạn](#sharing-sites-on-your-local-network)
- [Site Specific Environment Variables](#site-specific-environment-variables)
- [Proxying Services](#proxying-services)
- [Custom Valet Drivers](#custom-valet-drivers)
    - [Local Drivers](#local-drivers)
- [Các Lệnh Valet Khác](#other-valet-commands)
- [Valet Directories và Files](#valet-directories-and-files)
    - [Disk Access](#disk-access)

<a name="introduction"></a>
## Giới thiệu

> [!NOTE]
> Tìm kiếm một cách dễ dàng hơn để phát triển các ứng dụng Laravel trên macOS hoặc Windows? Hãy xem [Laravel Herd](https://herd.laravel.com). Herd bao gồm mọi thứ bạn cần để bắt đầu phát triển Laravel, bao gồm Valet, PHP, và Composer.

[Laravel Valet](https://github.com/laravel/valet) là một môi trường phát triển cho macOS minimalists. Laravel Valet cấu hình Mac của bạn để luôn chạy [Nginx](https://www.nginx.com/) trong nền khi máy của bạn khởi động. Sau đó, sử dụng [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet proxy tất cả các requests trên domain `*.test` để trỏ đến các sites được cài đặt trên máy local của bạn.

Nói cách khác, Valet là một môi trường phát triển Laravel cực nhanh sử dụng khoảng 7 MB RAM. Valet không phải là một thay thế hoàn toàn cho [Sail](/docs/{{version}}/sail) hoặc [Homestead](/docs/{{version}}/homestead), nhưng cung cấp một lựa chọn tuyệt vời nếu bạn muốn cơ bản linh hoạt, thích tốc độ cực cao, hoặc đang làm việc trên một máy có lượng RAM hạn chế.

Out of the box, hỗ trợ Valet bao gồm, nhưng không giới hạn:

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">

- [Laravel](https://laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [ConcreteCMS](https://www.concretecms.com/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- Static HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)

</div>

Tuy nhiên, bạn có thể mở rộng Valet với [custom drivers](#custom-valet-drivers) của riêng bạn.

<a name="installation"></a>
## Cài đặt

> [!WARNING]
> Valet yêu cầu macOS và [Homebrew](https://brew.sh/). Trước khi cài đặt, bạn nên đảm bảo rằng không có chương trình nào khác như Apache hoặc Nginx đang bind đến port 80 của máy local của bạn.

Để bắt đầu, bạn trước tiên cần đảm bảo rằng Homebrew được cập nhật bằng lệnh `update`:

```shell
brew update
```

Tiếp theo, bạn nên sử dụng Homebrew để cài đặt PHP:

```shell
brew install php
```

Sau khi cài đặt PHP, bạn đã sẵn sàng để cài đặt [Composer package manager](https://getcomposer.org). Ngoài ra, bạn nên đảm bảo rằng thư mục `$HOME/.composer/vendor/bin` nằm trong "PATH" của hệ thống. Sau khi Composer đã được cài đặt, bạn có thể cài đặt Laravel Valet như một global Composer package:

```shell
composer global require laravel/valet
```

Cuối cùng, bạn có thể thực thi lệnh `install` của Valet. Điều này sẽ cấu hình và cài đặt Valet và DnsMasq. Ngoài ra, các daemons mà Valet phụ thuộc sẽ được cấu hình để khởi động khi hệ thống của bạn khởi động:

```shell
valet install
```

Sau khi Valet được cài đặt, hãy thử ping bất kỳ domain `*.test` nào trên terminal của bạn bằng một lệnh như `ping foobar.test`. Nếu Valet được cài đặt đúng, bạn sẽ thấy domain này phản hồi trên `127.0.0.1`.

Valet sẽ tự động khởi động các services cần thiết mỗi khi máy của bạn khởi động.

<a name="php-versions"></a>
#### PHP Versions

> [!NOTE]
> Thay vì sửa đổi phiên bản PHP global của bạn, bạn có thể hướng dẫn Valet sử dụng per-site PHP versions qua lệnh `isolate` [command](#per-site-php-versions).

Valet cho phép bạn chuyển đổi PHP versions bằng lệnh `valet use php@version`. Valet sẽ cài đặt phiên bản PHP được chỉ định qua Homebrew nếu nó chưa được cài đặt:

```shell
valet use php@8.2

valet use php
```

Bạn cũng có thể tạo một file `.valetrc` trong root của dự án. File `.valetrc` nên chứa phiên bản PHP mà site nên sử dụng:

```shell
php=php@8.2
```

Sau khi file này đã được tạo, bạn có thể đơn giản thực thi lệnh `valet use` và lệnh sẽ xác định phiên bản PHP ưu tiên của site bằng cách đọc file.

> [!WARNING]
> Valet chỉ phục vụ một phiên bản PHP tại một thời điểm, ngay cả khi bạn có nhiều phiên bản PHP được cài đặt.

<a name="database"></a>
#### Database

Nếu ứng dụng của bạn cần một database, hãy xem [DBngin](https://dbngin.com), cung cấp một công cụ quản lý database all-in-one miễn phí bao gồm MySQL, PostgreSQL, và Redis. Sau khi DBngin đã được cài đặt, bạn có thể kết nối với database tại `127.0.0.1` sử dụng username `root` và một chuỗi rỗng cho password.

<a name="resetting-your-installation"></a>
#### Resetting Your Installation

Nếu bạn gặp khó khăn khi cài đặt Valet chạy đúng, thực thi lệnh `composer global require laravel/valet` theo sau là `valet install` sẽ reset cài đặt của bạn và có thể giải quyết nhiều vấn đề. Trong các trường hợp hiếm gặp, có thể cần "hard reset" Valet bằng cách thực thi `valet uninstall --force` theo sau là `valet install`.

<a name="upgrading-valet"></a>
### Nâng cấp Valet

Bạn có thể cập nhật cài đặt Valet của mình bằng cách thực thi lệnh `composer global require laravel/valet` trong terminal. Sau khi nâng cấp, thực hành tốt là chạy lệnh `valet install` để Valet có thể thực hiện các nâng cấp bổ sung cho các file cấu hình của bạn nếu cần thiết.

<a name="upgrading-to-valet-4"></a>
#### Nâng cấp lên Valet 4

Nếu bạn đang nâng cấp từ Valet 3 lên Valet 4, hãy thực hiện các bước sau để nâng cấp cài đặt Valet của bạn đúng cách:

<div class="content-list" markdown="1">

- Nếu bạn đã thêm các file `.valetphprc` để tùy chỉnh phiên bản PHP của site, hãy đổi tên mỗi file `.valetphprc` thành `.valetrc`. Sau đó, thêm tiền tố `php=` vào nội dung hiện có của file `.valetrc`.
- Cập nhật bất kỳ custom drivers nào để khớp với namespace, extension, type-hints, và return type-hints của hệ thống driver mới. Bạn có thể tham khảo [SampleValetDriver](https://github.com/laravel/valet/blob/d7787c025e60abc24a5195dc7d4c5c6f2d984339/cli/stubs/SampleValetDriver.php) của Valet làm ví dụ.
- Nếu bạn sử dụng PHP 7.1 - 7.4 để phục vụ các sites của bạn, hãy đảm bảo bạn vẫn sử dụng Homebrew để cài đặt một phiên bản PHP là 8.0 hoặc cao hơn, vì Valet sẽ sử dụng phiên bản này, ngay cả khi nó không phải là phiên bản linked chính của bạn, để chạy một số scripts của nó.

</div>

<a name="serving-sites"></a>
## Serving Sites

Sau khi Valet được cài đặt, bạn đã sẵn sàng để bắt đầu phục vụ các ứng dụng Laravel của mình. Valet cung cấp hai lệnh để giúp bạn phục vụ các ứng dụng: `park` và `link`.

<a name="the-park-command"></a>
### Lệnh `park`

Lệnh `park` đăng ký một thư mục trên máy của bạn chứa các ứng dụng của bạn. Sau khi thư mục đã được "parked" với Valet, tất cả các thư mục trong thư mục đó sẽ có thể truy cập trong trình duyệt web của bạn tại `http://<directory-name>.test`:

```shell
cd ~/Sites

valet park
```

Đó là tất cả những gì cần thiết. Bây giờ, bất kỳ ứng dụng nào bạn tạo trong thư mục "parked" sẽ tự động được phục vụ sử dụng quy ước `http://<directory-name>.test`. Vì vậy, nếu thư mục parked của bạn chứa một thư mục tên là "laravel", ứng dụng trong thư mục đó sẽ có thể truy cập tại `http://laravel.test`. Ngoài ra, Valet tự động cho phép bạn truy cập site sử dụng wildcard subdomains (`http://foo.laravel.test`).

<a name="the-link-command"></a>
### Lệnh `link`

Lệnh `link` cũng có thể được sử dụng để phục vụ các ứng dụng Laravel của bạn. Lệnh này hữu ích nếu bạn muốn phục vụ một site duy nhất trong một thư mục và không phải toàn bộ thư mục:

```shell
cd ~/Sites/laravel

valet link
```

Sau khi một ứng dụng đã được linked đến Valet bằng lệnh `link`, bạn có thể truy cập ứng dụng bằng tên thư mục của nó. Vì vậy, site được linked trong ví dụ trên có thể truy cập tại `http://laravel.test`. Ngoài ra, Valet tự động cho phép bạn truy cập site sử dụng wildcard sub-domains (`http://foo.laravel.test`).

Nếu bạn muốn phục vụ ứng dụng tại một hostname khác, bạn có thể chuyển hostname cho lệnh `link`. Ví dụ, bạn có thể chạy lệnh sau để làm cho một ứng dụng có sẵn tại `http://application.test`:

```shell
cd ~/Sites/laravel

valet link application
```

Tất nhiên, bạn cũng có thể phục vụ các ứng dụng trên subdomains bằng lệnh `link`:

```shell
valet link api.application
```

Bạn có thể thực thi lệnh `links` để hiển thị danh sách tất cả các thư mục linked của bạn:

```shell
valet links
```

Lệnh `unlink` có thể được sử dụng để destroy symbolic link cho một site:

```shell
cd ~/Sites/laravel

valet unlink
```

<a name="securing-sites"></a>
### Bảo vệ Sites Với TLS

Theo mặc định, Valet phục vụ sites qua HTTP. Tuy nhiên, nếu bạn muốn phục vụ một site qua TLS được mã hóa sử dụng HTTP/2, bạn có thể sử dụng lệnh `secure`. Ví dụ, nếu site của bạn đang được phục vụ bởi Valet trên domain `laravel.test`, bạn nên chạy lệnh sau để bảo vệ nó:

```shell
valet secure laravel
```

Để "unsecure" một site và revert lại phục vụ traffic của nó qua HTTP thông thường, sử dụng lệnh `unsecure`. Giống như lệnh `secure`, lệnh này chấp nhận hostname mà bạn muốn unsecure:

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### Serving a Default Site

Sometimes, you may wish to configure Valet to serve a "default" site instead of a `404` when visiting an unknown `test` domain. To accomplish this, you may add a `default` option to your `~/.config/valet/config.json` configuration file containing the path to the site that should serve as your default site:

    "default": "/Users/Sally/Sites/example-site",

<a name="per-site-php-versions"></a>
### Per-Site PHP Versions

By default, Valet uses your global PHP installation to serve your sites. However, if you need to support multiple PHP versions across various sites, you may use the `isolate` command to specify which PHP version a particular site should use. The `isolate` command configures Valet to use the specified PHP version for the site located in your current working directory:

```shell
cd ~/Sites/example-site

valet isolate php@8.0
```

If your site name does not match the name of the directory that contains it, you may specify the site name using the `--site` option:

```shell
valet isolate php@8.0 --site="site-name"
```

For convenience, you may use the `valet php`, `composer`, and `which-php` commands to proxy calls to the appropriate PHP CLI or tool based on the site's configured PHP version:

```shell
valet php
valet composer
valet which-php
```

You may execute the `isolated` command to display a list of all of your isolated sites and their PHP versions:

```shell
valet isolated
```

To revert a site back to Valet's globally installed PHP version, you may invoke the `unisolate` command from the site's root directory:

```shell
valet unisolate
```

<a name="sharing-sites"></a>
## Sharing Sites

Valet includes a command to share your local sites with the world, providing an easy way to test your site on mobile devices or share it with team members and clients.

Out of the box, Valet supports sharing your sites via ngrok or Expose. Before sharing a site, you should update your Valet configuration using the `share-tool` command, specifying `ngrok`, `expose`, or  `cloudflared`:

```shell
valet share-tool ngrok
```

If you choose a tool and don't have it installed via Homebrew (for ngrok and cloudflared) or Composer (for Expose), Valet will automatically prompt you to install it. Of course, both tools require you to authenticate your ngrok or Expose account before you can start sharing sites.

To share a site, navigate to the site's directory in your terminal and run Valet's `share` command. A publicly accessible URL will be placed into your clipboard and is ready to paste directly into your browser or to be shared with your team:

```shell
cd ~/Sites/laravel

valet share
```

To stop sharing your site, you may press `Control + C`.

> [!WARNING]
> If you're using a custom DNS server (like `1.1.1.1`), ngrok sharing may not work correctly. If this is the case on your machine, open your Mac's system settings, go to the Network settings, open the Advanced settings, then go the DNS tab and add `127.0.0.1` as your first DNS server.

<a name="sharing-sites-via-ngrok"></a>
#### Sharing Sites via Ngrok

Sharing your site using ngrok requires you to [create an ngrok account](https://dashboard.ngrok.com/signup) and [set up an authentication token](https://dashboard.ngrok.com/get-started/your-authtoken). Once you have an authentication token, you can update your Valet configuration with that token:

```shell
valet set-ngrok-token YOUR_TOKEN_HERE
```

> [!NOTE]
> You may pass additional ngrok parameters to the share command, such as `valet share --region=eu`. For more information, consult the [ngrok documentation](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>
#### Sharing Sites via Expose

Sharing your site using Expose requires you to [create an Expose account](https://expose.dev/register) and [authenticate with Expose via your authentication token](https://expose.dev/docs/getting-started/getting-your-token).

You may consult the [Expose documentation](https://expose.dev/docs) for information regarding the additional command-line parameters it supports.

<a name="sharing-sites-on-your-local-network"></a>
### Sharing Sites on Your Local Network

Valet restricts incoming traffic to the internal `127.0.0.1` interface by default so that your development machine isn't exposed to security risks from the Internet.

If you wish to allow other devices on your local network to access the Valet sites on your machine via your machine's IP address (eg: `192.168.1.10/application.test`), you will need to manually edit the appropriate Nginx configuration file for that site to remove the restriction on the `listen` directive. You should remove the `127.0.0.1:` prefix on the `listen` directive for ports 80 and 443.

If you have not run `valet secure` on the project, you can open up network access for all non-HTTPS sites by editing the `/usr/local/etc/nginx/valet/valet.conf` file. However, if you're serving the project site over HTTPS (you have run `valet secure` for the site) then you should edit the `~/.config/valet/Nginx/app-name.test` file.

Once you have updated your Nginx configuration, run the `valet restart` command to apply the configuration changes.

<a name="site-specific-environment-variables"></a>
## Site Specific Environment Variables

Some applications using other frameworks may depend on server environment variables but do not provide a way for those variables to be configured within your project. Valet allows you to configure site specific environment variables by adding a `.valet-env.php` file within the root of your project. This file should return an array of site / environment variable pairs which will be added to the global `$_SERVER` array for each site specified in the array:

```php
<?php

return [
    // Set $_SERVER['key'] to "value" for the laravel.test site...
    'laravel' => [
        'key' => 'value',
    ],

    // Set $_SERVER['key'] to "value" for all sites...
    '*' => [
        'key' => 'value',
    ],
];
```

<a name="proxying-services"></a>
## Proxying Services

Sometimes you may wish to proxy a Valet domain to another service on your local machine. For example, you may occasionally need to run Valet while also running a separate site in Docker; however, Valet and Docker can't both bind to port 80 at the same time.

To solve this, you may use the `proxy` command to generate a proxy. For example, you may proxy all traffic from `http://elasticsearch.test` to `http://127.0.0.1:9200`:

```shell
# Proxy over HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

# Proxy over TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

You may remove a proxy using the `unproxy` command:

```shell
valet unproxy elasticsearch
```

You may use the `proxies` command to list all site configurations that are proxied:

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## Custom Valet Drivers

You can write your own Valet "driver" to serve PHP applications running on a framework or CMS that is not natively supported by Valet. When you install Valet, a `~/.config/valet/Drivers` directory is created which contains a `SampleValetDriver.php` file. This file contains a sample driver implementation to demonstrate how to write a custom driver. Writing a driver only requires you to implement three methods: `serves`, `isStaticFile`, and `frontControllerPath`.

All three methods receive the `$sitePath`, `$siteName`, and `$uri` values as their arguments. The `$sitePath` is the fully qualified path to the site being served on your machine, such as `/Users/Lisa/Sites/my-project`. The `$siteName` is the "host" / "site name" portion of the domain (`my-project`). The `$uri` is the incoming request URI (`/foo/bar`).

Once you have completed your custom Valet driver, place it in the `~/.config/valet/Drivers` directory using the `FrameworkValetDriver.php` naming convention. For example, if you are writing a custom valet driver for WordPress, your filename should be `WordPressValetDriver.php`.

Let's take a look at a sample implementation of each method your custom Valet driver should implement.

<a name="the-serves-method"></a>
#### The `serves` Method

The `serves` method should return `true` if your driver should handle the incoming request. Otherwise, the method should return `false`. So, within this method, you should attempt to determine if the given `$sitePath` contains a project of the type you are trying to serve.

For example, let's imagine we are writing a `WordPressValetDriver`. Our `serves` method might look something like this:

```php
/**
 * Determine if the driver serves the request.
 */
public function serves(string $sitePath, string $siteName, string $uri): bool
{
    return is_dir($sitePath.'/wp-admin');
}
```

<a name="the-isstaticfile-method"></a>
#### The `isStaticFile` Method

The `isStaticFile` should determine if the incoming request is for a file that is "static", such as an image or a stylesheet. If the file is static, the method should return the fully qualified path to the static file on disk. If the incoming request is not for a static file, the method should return `false`:

```php
/**
 * Determine if the incoming request is for a static file.
 *
 * @return string|false
 */
public function isStaticFile(string $sitePath, string $siteName, string $uri)
{
    if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
        return $staticFilePath;
    }

    return false;
}
```

> [!WARNING]
> The `isStaticFile` method will only be called if the `serves` method returns `true` for the incoming request and the request URI is not `/`.

<a name="the-frontcontrollerpath-method"></a>
#### The `frontControllerPath` Method

The `frontControllerPath` method should return the fully qualified path to your application's "front controller", which is typically an "index.php" file or equivalent:

```php
/**
 * Get the fully resolved path to the application's front controller.
 */
public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
{
    return $sitePath.'/public/index.php';
}
```

<a name="local-drivers"></a>
### Local Drivers

If you would like to define a custom Valet driver for a single application, create a `LocalValetDriver.php` file in the application's root directory. Your custom driver may extend the base `ValetDriver` class or extend an existing application specific driver such as the `LaravelValetDriver`:

```php
use Valet\Drivers\LaravelValetDriver;

class LocalValetDriver extends LaravelValetDriver
{
    /**
     * Determine if the driver serves the request.
     */
    public function serves(string $sitePath, string $siteName, string $uri): bool
    {
        return true;
    }

    /**
     * Get the fully resolved path to the application's front controller.
     */
    public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
    {
        return $sitePath.'/public_html/index.php';
    }
}
```

<a name="other-valet-commands"></a>
## Other Valet Commands

<div class="overflow-auto">

| Command | Description |
| --- | --- |
| `valet list` | Display a list of all Valet commands. |
| `valet diagnose` | Output diagnostics to aid in debugging Valet. |
| `valet directory-listing` | Determine directory-listing behavior. Default is "off", which renders a 404 page for directories. |
| `valet forget` | Run this command from a "parked" directory to remove it from the parked directory list. |
| `valet log` | View a list of logs which are written by Valet's services. |
| `valet paths` | View all of your "parked" paths. |
| `valet restart` | Restart the Valet daemons. |
| `valet start` | Start the Valet daemons. |
| `valet stop` | Stop the Valet daemons. |
| `valet trust` | Add sudoers files for Brew and Valet to allow Valet commands to be run without prompting for your password. |
| `valet uninstall` | Uninstall Valet: shows instructions for manual uninstall. Pass the `--force` option to aggressively delete all of Valet's resources. |

</div>

<a name="valet-directories-and-files"></a>
## Valet Directories and Files

You may find the following directory and file information helpful while troubleshooting issues with your Valet environment:

#### `~/.config/valet`

Contains all of Valet's configuration. You may wish to maintain a backup of this directory.

#### `~/.config/valet/dnsmasq.d/`

This directory contains DNSMasq's configuration.

#### `~/.config/valet/Drivers/`

This directory contains Valet's drivers. Drivers determine how a particular framework / CMS is served.

#### `~/.config/valet/Nginx/`

This directory contains all of Valet's Nginx site configurations. These files are rebuilt when running the `install` and `secure` commands.

#### `~/.config/valet/Sites/`

This directory contains all of the symbolic links for your [linked projects](#the-link-command).

#### `~/.config/valet/config.json`

This file is Valet's master configuration file.

#### `~/.config/valet/valet.sock`

This file is the PHP-FPM socket used by Valet's Nginx installation. This will only exist if PHP is running properly.

#### `~/.config/valet/Log/fpm-php.www.log`

This file is the user log for PHP errors.

#### `~/.config/valet/Log/nginx-error.log`

This file is the user log for Nginx errors.

#### `/usr/local/var/log/php-fpm.log`

This file is the system log for PHP-FPM errors.

#### `/usr/local/var/log/nginx`

This directory contains the Nginx access and error logs.

#### `/usr/local/etc/php/X.X/conf.d`

This directory contains the `*.ini` files for various PHP configuration settings.

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

This file is the PHP-FPM pool configuration file.

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

This file is the default Nginx configuration used for building SSL certificates for your sites.

<a name="disk-access"></a>
### Disk Access

Since macOS 10.14, [access to some files and directories is restricted by default](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf). These restrictions include the Desktop, Documents, and Downloads directories. In addition, network volume and removable volume access is restricted. Therefore, Valet recommends your site folders are located outside of these protected locations.

However, if you wish to serve sites from within one of those locations, you will need to give Nginx "Full Disk Access". Otherwise, you may encounter server errors or other unpredictable behavior from Nginx, especially when serving static assets. Typically, macOS will automatically prompt you to grant Nginx full access to these locations. Or, you may do so manually via `System Preferences` > `Security & Privacy` > `Privacy` and selecting `Full Disk Access`. Next, enable any `nginx` entries in the main window pane.

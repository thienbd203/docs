# Laravel Valet

- [Introduction](#introduction)
- [Installation](#installation)
    - [Upgrading Valet](#upgrading-valet)
- [Serving Sites](#serving-sites)
    - [The "Park" Command](#the-park-command)
    - [The "Link" Command](#the-link-command)
    - [Securing Sites With TLS](#securing-sites)
    - [Serving a Default Site](#serving-a-default-site)
    - [Per-Site PHP Versions](#per-site-php-versions)
- [Sharing Sites](#sharing-sites)
    - [Sharing Sites on Your Local Network](#sharing-sites-on-your-local-network)
- [Site Specific Environment Variables](#site-specific-environment-variables)
- [Proxying Services](#proxying-services)
- [Custom Valet Drivers](#custom-valet-drivers)
    - [Local Drivers](#local-drivers)
- [Other Valet Commands](#other-valet-commands)
- [Valet Directories and Files](#valet-directories-and-files)
    - [Disk Access](#disk-access)

<a name="introduction"></a>
## Introduction

> [!NOTE]
> Tìm kiếm một cách dễ dàng hơn để phát triển các ứng dụng Laravel trên macOS hoặc Windows? Hãy xem [Laravel Herd](https://herd.laravel.com). Herd bao gồm mọi thứ bạn cần để bắt đầu phát triển Laravel, bao gồm Valet, PHP, và Composer.

[Laravel Valet](https://github.com/laravel/valet) là một môi trường phát triển cho những người tối giản macOS. Laravel Valet cấu hình Mac của bạn để luôn chạy [Nginx](https://www.nginx.com/) trong nền khi máy của bạn khởi động. Sau đó, sử dụng [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq), Valet proxy tất cả các requests trên domain `*.test` để trỏ đến các sites được cài đặt trên máy cục bộ của bạn.

Nói cách khác, Valet là một môi trường phát triển Laravel cực nhanh sử dụng khoảng 7 MB RAM. Valet không phải là thay thế hoàn toàn cho [Sail](/docs/{{version}}/sail) hoặc [Homestead](/docs/{{version}}/homestead), nhưng cung cấp một lựa chọn thay thế tuyệt vời nếu bạn muốn những điều cơ bản linh hoạt, thích tốc độ cực cao, hoặc đang làm việc trên một máy có lượng RAM hạn chế.

Sẵn sàng, hỗ trợ Valet bao gồm, nhưng không giới hạn:

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
## Installation

> [!WARNING]
> Valet yêu cầu macOS và [Homebrew](https://brew.sh/). Trước khi cài đặt, bạn nên đảm bảo rằng không có chương trình nào khác như Apache hoặc Nginx đang bind với port 80 của máy cục bộ của bạn.

Để bắt đầu, trước tiên bạn cần đảm bảo rằng Homebrew được cập nhật bằng cách sử dụng lệnh `update`:

```shell
brew update
```

Tiếp theo, bạn nên sử dụng Homebrew để cài đặt PHP:

```shell
brew install php
```

Sau khi cài đặt PHP, bạn đã sẵn sàng để cài đặt [trình quản lý package Composer](https://getcomposer.org). Ngoài ra, bạn nên đảm bảo rằng thư mục `$HOME/.composer/vendor/bin` nằm trong "PATH" của hệ thống. Sau khi Composer đã được cài đặt, bạn có thể cài đặt Laravel Valet như một global Composer package:

```shell
composer global require laravel/valet
```

Cuối cùng, bạn có thể thực thi lệnh `install` của Valet. Điều này sẽ cấu hình và cài đặt Valet và DnsMasq. Ngoài ra, các daemons mà Valet phụ thuộc sẽ được cấu hình để khởi động khi hệ thống của bạn khởi động:

```shell
valet install
```

Sau khi Valet được cài đặt, hãy thử ping bất kỳ domain `*.test` nào trên terminal của bạn bằng cách sử dụng một lệnh như `ping foobar.test`. Nếu Valet được cài đặt đúng, bạn sẽ thấy domain này phản hồi trên `127.0.0.1`.

Valet sẽ tự động khởi động các dịch vụ cần thiết của nó mỗi khi máy của bạn khởi động.

<a name="php-versions"></a>
#### PHP Versions

> [!NOTE]
> Thay vì sửa đổi phiên bản PHP toàn cầu của bạn, bạn có thể hướng dẫn Valet sử dụng các phiên bản PHP cho mỗi site thông qua lệnh `isolate` [command](#per-site-php-versions).

Valet cho phép bạn chuyển đổi các phiên bản PHP bằng cách sử dụng lệnh `valet use php@version`. Valet sẽ cài đặt phiên bản PHP được chỉ định thông qua Homebrew nếu nó chưa được cài đặt:

```shell
valet use php@8.2

valet use php
```

Bạn cũng có thể tạo một file `.valetrc` trong root của dự án. File `.valetrc` nên chứa phiên bản PHP mà site nên sử dụng:

```shell
php=php@8.2
```

Sau khi file này đã được tạo, bạn có thể chỉ cần thực thi lệnh `valet use` và lệnh sẽ xác định phiên bản PHP ưu tiên của site bằng cách đọc file.

> [!WARNING]
> Valet chỉ phục vụ một phiên bản PHP tại một thời điểm, ngay cả khi bạn có nhiều phiên bản PHP được cài đặt.

<a name="database"></a>
#### Database

Nếu ứng dụng của bạn cần một database, hãy xem [DBngin](https://dbngin.com), cung cấp một công cụ quản lý database miễn phí, tất cả trong một bao gồm MySQL, PostgreSQL, và Redis. Sau khi DBngin đã được cài đặt, bạn có thể kết nối với database của bạn tại `127.0.0.1` bằng cách sử dụng username `root` và một chuỗi rỗng cho mật khẩu.

<a name="resetting-your-installation"></a>
#### Resetting Your Installation

Nếu bạn gặp khó khăn để cài đặt Valet chạy đúng, thực thi lệnh `composer global require laravel/valet` theo sau là `valet install` sẽ reset cài đặt của bạn và có thể giải quyết nhiều vấn đề. Trong các trường hợp hiếm, có thể cần phải "hard reset" Valet bằng cách thực thi `valet uninstall --force` theo sau là `valet install`.

<a name="upgrading-valet"></a>
### Upgrading Valet

Bạn có thể cập nhật cài đặt Valet của bạn bằng cách thực thi lệnh `composer global require laravel/valet` trong terminal. Sau khi nâng cấp, đó là một thực hành tốt để chạy lệnh `valet install` để Valet có thể thực hiện các nâng cấp bổ sung cho các file cấu hình của bạn nếu cần.

<a name="upgrading-to-valet-4"></a>
#### Upgrading to Valet 4

Nếu bạn đang nâng cấp từ Valet 3 lên Valet 4, hãy thực hiện các bước sau để nâng cấp đúng cài đặt Valet của bạn:

<div class="content-list" markdown="1">

- Nếu bạn đã thêm các file `.valetphprc` để tùy chỉnh phiên bản PHP của site, hãy đổi tên mỗi file `.valetphprc` thành `.valetrc`. Sau đó, thêm tiền tố `php=` vào nội dung hiện có của file `.valetrc`.
- Cập nhật bất kỳ custom drivers nào để khớp với namespace, extension, type-hints, và return type-hints của hệ thống driver mới. Bạn có thể tham khảo [SampleValetDriver](https://github.com/laravel/valet/blob/d7787c025e60abc24a5195dc7d4c5c6f2d984339/cli/stubs/SampleValetDriver.php) của Valet như một ví dụ.
- Nếu bạn sử dụng PHP 7.1 - 7.4 để phục vụ các sites của bạn, hãy đảm bảo bạn vẫn sử dụng Homebrew để cài đặt một phiên bản PHP là 8.0 hoặc cao hơn, vì Valet sẽ sử dụng phiên bản này, ngay cả khi nó không phải là phiên bản được liên kết chính của bạn, để chạy một số script của nó.

</div>

<a name="serving-sites"></a>
## Serving Sites

Sau khi Valet được cài đặt, bạn đã sẵn sàng để bắt đầu phục vụ các ứng dụng Laravel của bạn. Valet cung cấp hai lệnh để giúp bạn phục vụ các ứng dụng: `park` và `link`.

<a name="the-park-command"></a>
### The `park` Command

Lệnh `park` đăng ký một thư mục trên máy của bạn chứa các ứng dụng của bạn. Sau khi thư mục đã được "parked" với Valet, tất cả các thư mục trong thư mục đó sẽ có thể truy cập trong trình duyệt web của bạn tại `http://<directory-name>.test`:

```shell
cd ~/Sites

valet park
```

Đó là tất cả những gì cần thiết. Bây giờ, bất kỳ ứng dụng nào bạn tạo trong thư mục "parked" của bạn sẽ tự động được phục vụ bằng cách sử dụng quy ước `http://<directory-name>.test`. Vì vậy, nếu thư mục parked của bạn chứa một thư mục có tên "laravel", ứng dụng trong thư mục đó sẽ có thể truy cập tại `http://laravel.test`. Ngoài ra, Valet tự động cho phép bạn truy cập site bằng cách sử dụng wildcard subdomains (`http://foo.laravel.test`).

<a name="the-link-command"></a>
### The `link` Command

Lệnh `link` cũng có thể được sử dụng để phục vụ các ứng dụng Laravel của bạn. Lệnh này hữu ích nếu bạn muốn phục vụ một site duy nhất trong một thư mục và không phải toàn bộ thư mục:

```shell
cd ~/Sites/laravel

valet link
```

Sau khi một ứng dụng đã được liên kết với Valet bằng cách sử dụng lệnh `link`, bạn có thể truy cập ứng dụng bằng cách sử dụng tên thư mục của nó. Vì vậy, site đã được liên kết trong ví dụ trên có thể được truy cập tại `http://laravel.test`. Ngoài ra, Valet tự động cho phép bạn truy cập site bằng cách sử dụng wildcard sub-domains (`http://foo.laravel.test`).

Nếu bạn muốn phục vụ ứng dụng tại một hostname khác, bạn có thể chuyển hostname cho lệnh `link`. Ví dụ, bạn có thể chạy lệnh sau để làm cho một ứng dụng có sẵn tại `http://application.test`:

```shell
cd ~/Sites/laravel

valet link application
```

Tất nhiên, bạn cũng có thể phục vụ các ứng dụng trên subdomains bằng cách sử dụng lệnh `link`:

```shell
valet link api.application
```

Bạn có thể thực thi lệnh `links` để hiển thị danh sách tất cả các thư mục được liên kết của bạn:

```shell
valet links
```

Lệnh `unlink` có thể được sử dụng để hủy symbolic link cho một site:

```shell
cd ~/Sites/laravel

valet unlink
```

<a name="securing-sites"></a>
### Securing Sites With TLS

Theo mặc định, Valet phục vụ các sites qua HTTP. Tuy nhiên, nếu bạn muốn phục vụ một site qua TLS được mã hóa bằng cách sử dụng HTTP/2, bạn có thể sử dụng lệnh `secure`. Ví dụ, nếu site của bạn đang được phục vụ bởi Valet trên domain `laravel.test`, bạn nên chạy lệnh sau để bảo mật nó:

```shell
valet secure laravel
```

Để "unsecure" một site và quay lại phục vụ traffic của nó qua HTTP đơn giản, hãy sử dụng lệnh `unsecure`. Như lệnh `secure`, lệnh này chấp nhận hostname mà bạn muốn unsecure:

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### Serving a Default Site

Đôi khi, bạn có thể muốn cấu hình Valet để phục vụ một site "default" thay vì một `404` khi truy cập một domain `test` không xác định. Để thực hiện điều này, bạn có thể thêm một tùy chọn `default` vào file cấu hình `~/.config/valet/config.json` của bạn chứa đường dẫn đến site nên phục vụ như site mặc định của bạn:

    "default": "/Users/Sally/Sites/example-site",

<a name="per-site-php-versions"></a>
### Per-Site PHP Versions

Theo mặc định, Valet sử dụng cài đặt PHP toàn cầu của bạn để phục vụ các sites của bạn. Tuy nhiên, nếu bạn cần hỗ trợ nhiều phiên bản PHP trên các sites khác nhau, bạn có thể sử dụng lệnh `isolate` để chỉ định phiên bản PHP mà một site cụ thể nên sử dụng. Lệnh `isolate` cấu hình Valet để sử dụng phiên bản PHP được chỉ định cho site nằm trong thư mục làm việc hiện tại của bạn:

```shell
cd ~/Sites/example-site

valet isolate php@8.0
```

Nếu tên site của bạn không khớp với tên thư mục chứa nó, bạn có thể chỉ định tên site bằng cách sử dụng tùy chọn `--site`:

```shell
valet isolate php@8.0 --site="site-name"
```

Để thuận tiện, bạn có thể sử dụng các lệnh `valet php`, `composer`, và `which-php` để proxy các calls đến PHP CLI hoặc công cụ thích hợp dựa trên phiên bản PHP được cấu hình của site:

```shell
valet php
valet composer
valet which-php
```

Bạn có thể thực thi lệnh `isolated` để hiển thị danh sách tất cả các sites được cô lập của bạn và các phiên bản PHP của chúng:

```shell
valet isolated
```

Để quay lại một site về phiên bản PHP được cài đặt toàn cầu của Valet, bạn có thể gọi lệnh `unisolate` từ thư mục root của site:

```shell
valet unisolate
```

<a name="sharing-sites"></a>
## Sharing Sites

Valet bao gồm một lệnh để chia sẻ các sites cục bộ của bạn với thế giới, cung cấp một cách dễ dàng để test site của bạn trên các thiết bị di động hoặc chia sẻ nó với các thành viên nhóm và khách hàng.

Sẵn sàng, Valet hỗ trợ chia sẻ các sites của bạn thông qua ngrok hoặc Expose. Trước khi chia sẻ một site, bạn nên cập nhật cấu hình Valet của bạn bằng cách sử dụng lệnh `share-tool`, chỉ định `ngrok`, `expose`, hoặc `cloudflared`:

```shell
valet share-tool ngrok
```

Nếu bạn chọn một công cụ và không có nó được cài đặt thông qua Homebrew (cho ngrok và cloudflared) hoặc Composer (cho Expose), Valet sẽ tự động nhắc bạn cài đặt nó. Tất nhiên, cả hai công cụ đều yêu cầu bạn xác thực tài khoản ngrok hoặc Expose của bạn trước khi bạn có thể bắt đầu chia sẻ các sites.

Để chia sẻ một site, điều hướng đến thư mục của site trong terminal và chạy lệnh `share` của Valet. Một URL có thể truy cập công khai sẽ được đặt vào clipboard của bạn và sẵn sàng để dán trực tiếp vào trình duyệt hoặc để chia sẻ với nhóm của bạn:

```shell
cd ~/Sites/laravel

valet share
```

Để ngừng chia sẻ site của bạn, bạn có thể nhấn `Control + C`.

> [!WARNING]
> Nếu bạn đang sử dụng một DNS server tùy chỉnh (như `1.1.1.1`), chia sẻ ngrok có thể không hoạt động đúng. Nếu đây là trường hợp trên máy của bạn, hãy mở cài đặt hệ thống của Mac, đi đến cài đặt Network, mở cài đặt Advanced, sau đó đi đến tab DNS và thêm `127.0.0.1` làm DNS server đầu tiên của bạn.

<a name="sharing-sites-via-ngrok"></a>
#### Sharing Sites via Ngrok

Chia sẻ site của bạn bằng cách sử dụng ngrok yêu cầu bạn [tạo một tài khoản ngrok](https://dashboard.ngrok.com/signup) và [thiết lập một authentication token](https://dashboard.ngrok.com/get-started/your-authtoken). Sau khi bạn có một authentication token, bạn có thể cập nhật cấu hình Valet của bạn với token đó:

```shell
valet set-ngrok-token YOUR_TOKEN_HERE
```

> [!NOTE]
> Bạn có thể chuyển các tham số ngrok bổ sung cho lệnh share, chẳng hạn như `valet share --region=eu`. Để biết thêm thông tin, hãy tham khảo [tài liệu ngrok](https://ngrok.com/docs).

<a name="sharing-sites-via-expose"></a>
#### Sharing Sites via Expose

Chia sẻ site của bạn bằng cách sử dụng Expose yêu cầu bạn [tạo một tài khoản Expose](https://expose.dev/register) và [xác thực với Expose thông qua authentication token của bạn](https://expose.dev/docs/getting-started/getting-your-token).

Bạn có thể tham khảo [tài liệu Expose](https://expose.dev/docs) để biết thông tin về các tham số dòng lệnh bổ sung mà nó hỗ trợ.

<a name="sharing-sites-on-your-local-network"></a>
### Sharing Sites on Your Local Network

Valet hạn chế traffic đến cho interface nội bộ `127.0.0.1` theo mặc định để máy phát triển của bạn không bị tiếp xúc với các rủi ro bảo mật từ Internet.

Nếu bạn muốn cho phép các thiết bị khác trên mạng cục bộ của bạn truy cập các sites Valet trên máy của bạn thông qua địa chỉ IP của máy (ví dụ: `192.168.1.10/application.test`), bạn sẽ cần chỉnh sửa thủ công file cấu hình Nginx thích hợp cho site đó để xóa hạn chế trên directive `listen`. Bạn nên xóa tiền tố `127.0.0.1:` trên directive `listen` cho các port 80 và 443.

Nếu bạn chưa chạy `valet secure` trên dự án, bạn có thể mở quyền truy cập mạng cho tất cả các sites không HTTPS bằng cách chỉnh sửa file `/usr/local/etc/nginx/valet/valet.conf`. Tuy nhiên, nếu bạn đang phục vụ site dự án qua HTTPS (bạn đã chạy `valet secure` cho site) thì bạn nên chỉnh sửa file `~/.config/valet/Nginx/app-name.test`.

Sau khi bạn đã cập nhật cấu hình Nginx của bạn, hãy chạy lệnh `valet restart` để áp dụng các thay đổi cấu hình.

<a name="site-specific-environment-variables"></a>
## Site Specific Environment Variables

Một số ứng dụng sử dụng các frameworks khác có thể phụ thuộc vào các biến môi trường server nhưng không cung cấp một cách để cấu hình các biến đó trong dự án của bạn. Valet cho phép bạn cấu hình các biến môi trường cụ thể cho site bằng cách thêm một file `.valet-env.php` trong root của dự án. File này nên trả về một array các cặp site / biến môi trường sẽ được thêm vào array toàn cầu `$_SERVER` cho mỗi site được chỉ định trong array:

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

Đôi khi bạn có thể muốn proxy một domain Valet đến một dịch vụ khác trên máy cục bộ của bạn. Ví dụ, bạn có thể thỉnh thoảng cần chạy Valet trong khi cũng chạy một site riêng biệt trong Docker; tuy nhiên, Valet và Docker không thể cùng bind với port 80 tại cùng một thời điểm.

Để giải quyết điều này, bạn có thể sử dụng lệnh `proxy` để tạo một proxy. Ví dụ, bạn có thể proxy tất cả traffic từ `http://elasticsearch.test` đến `http://127.0.0.1:9200`:

```shell
# Proxy over HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

# Proxy over TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

Bạn có thể xóa một proxy bằng cách sử dụng lệnh `unproxy`:

```shell
valet unproxy elasticsearch
```

Bạn có thể sử dụng lệnh `proxies` để liệt kê tất cả các cấu hình site được proxy:

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## Custom Valet Drivers

Bạn có thể viết "driver" Valet của riêng bạn để phục vụ các ứng dụng PHP chạy trên một framework hoặc CMS không được hỗ trợ nguyên bản bởi Valet. Khi bạn cài đặt Valet, một thư mục `~/.config/valet/Drivers` được tạo chứa một file `SampleValetDriver.php`. File này chứa một triển khai driver mẫu để minh họa cách viết một custom driver. Viết một driver chỉ yêu cầu bạn triển khai ba phương thức: `serves`, `isStaticFile`, và `frontControllerPath`.

Cả ba phương thức đều nhận các giá trị `$sitePath`, `$siteName`, và `$uri` làm đối số của chúng. `$sitePath` là đường dẫn đầy đủ đến site đang được phục vụ trên máy của bạn, chẳng hạn như `/Users/Lisa/Sites/my-project`. `$siteName` là phần "host" / "site name" của domain (`my-project`). `$uri` là URI request đến (`/foo/bar`).

Sau khi bạn đã hoàn thành custom Valet driver của bạn, hãy đặt nó trong thư mục `~/.config/valet/Drivers` bằng cách sử dụng quy ước đặt tên `FrameworkValetDriver.php`. Ví dụ, nếu bạn đang viết một custom valet driver cho WordPress, tên file của bạn nên là `WordPressValetDriver.php`.

Hãy xem một triển khai mẫu của mỗi phương thức mà custom Valet driver của bạn nên triển khai.

<a name="the-serves-method"></a>
#### The `serves` Method

Phương thức `serves` nên trả về `true` nếu driver của bạn nên xử lý request đến. Nếu không, phương thức nên trả về `false`. Vì vậy, trong phương thức này, bạn nên cố gắng xác định xem `$sitePath` đã cho có chứa một dự án của loại bạn đang cố gắng phục vụ hay không.

Ví dụ, hãy tưởng tượng chúng ta đang viết một `WordPressValetDriver`. Phương thức `serves` của chúng ta có thể trông giống như sau:

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

`isStaticFile` nên xác định xem request đến có phải cho một file "static" hay không, chẳng hạn như một hình ảnh hoặc một stylesheet. Nếu file là static, phương thức nên trả về đường dẫn đầy đủ đến file static trên đĩa. Nếu request đến không phải cho một file static, phương thức nên trả về `false`:

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
> Phương thức `isStaticFile` sẽ chỉ được gọi nếu phương thức `serves` trả về `true` cho request đến và URI request không phải là `/`.

<a name="the-frontcontrollerpath-method"></a>
#### The `frontControllerPath` Method

Phương thức `frontControllerPath` nên trả về đường dẫn đầy đủ đến "front controller" của ứng dụng, thường là một file "index.php" hoặc tương đương:

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

Nếu bạn muốn định nghĩa một custom Valet driver cho một ứng dụng duy nhất, hãy tạo một file `LocalValetDriver.php` trong thư mục root của ứng dụng. Custom driver của bạn có thể mở rộng class `ValetDriver` cơ sở hoặc mở rộng một driver cụ thể cho ứng dụng hiện có như `LaravelValetDriver`:

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

Bạn có thể thấy thông tin thư mục và file sau hữu ích trong khi khắc phục sự cố với môi trường Valet của bạn:

#### `~/.config/valet`

Chứa tất cả cấu hình của Valet. Bạn có thể muốn duy trì một bản sao lưu của thư mục này.

#### `~/.config/valet/dnsmasq.d/`

Thư mục này chứa cấu hình của DNSMasq.

#### `~/.config/valet/Drivers/`

Thư mục này chứa các drivers của Valet. Drivers xác định cách một framework / CMS cụ thể được phục vụ.

#### `~/.config/valet/Nginx/`

Thư mục này chứa tất cả các cấu hình site Nginx của Valet. Các file này được xây dựng lại khi chạy các lệnh `install` và `secure`.

#### `~/.config/valet/Sites/`

Thư mục này chứa tất cả các symbolic links cho các [dự án được liên kết](#the-link-command) của bạn.

#### `~/.config/valet/config.json`

File này là file cấu hình chính của Valet.

#### `~/.config/valet/valet.sock`

File này là socket PHP-FPM được sử dụng bởi cài đặt Nginx của Valet. Điều này sẽ chỉ tồn tại nếu PHP đang chạy đúng.

#### `~/.config/valet/Log/fpm-php.www.log`

File này là log người dùng cho các lỗi PHP.

#### `~/.config/valet/Log/nginx-error.log`

File này là log người dùng cho các lỗi Nginx.

#### `/usr/local/var/log/php-fpm.log`

File này là log hệ thống cho các lỗi PHP-FPM.

#### `/usr/local/var/log/nginx`

Thư mục này chứa các log truy cập và lỗi Nginx.

#### `/usr/local/etc/php/X.X/conf.d`

Thư mục này chứa các file `*.ini` cho các cài đặt cấu hình PHP khác nhau.

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

File này là file cấu hình pool PHP-FPM.

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

File này là cấu hình Nginx mặc định được sử dụng để xây dựng các chứng chỉ SSL cho các sites của bạn.

<a name="disk-access"></a>
### Disk Access

Kể từ macOS 10.14, [truy cập vào một số files và thư mục bị hạn chế theo mặc định](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf). Các hạn chế này bao gồm các thư mục Desktop, Documents, và Downloads. Ngoài ra, truy cập volume mạng và volume có thể tháo rời bị hạn chế. Do đó, Valet khuyến nghị các thư mục site của bạn nằm ngoài các vị trí được bảo vệ này.

Tuy nhiên, nếu bạn muốn phục vụ các sites từ trong một trong các vị trí đó, bạn sẽ cần cung cấp cho Nginx "Full Disk Access". Nếu không, bạn có thể gặp các lỗi server hoặc hành vi không thể đoán trước khác từ Nginx, đặc biệt là khi phục vụ các tài sản tĩnh. Thông thường, macOS sẽ tự động nhắc bạn cấp cho Nginx quyền truy cập đầy đủ vào các vị trí này. Hoặc, bạn có thể làm điều đó thủ công thông qua `System Preferences` > `Security & Privacy` > `Privacy` và chọn `Full Disk Access`. Tiếp theo, bật bất kỳ mục `nginx` nào trong cửa sổ chính.

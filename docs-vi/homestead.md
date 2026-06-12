# Laravel Homestead

- [Introduction](#introduction)
- [Installation and Setup](#installation-and-setup)
    - [First Steps](#first-steps)
    - [Configuring Homestead](#configuring-homestead)
    - [Configuring Nginx Sites](#configuring-nginx-sites)
    - [Configuring Services](#configuring-services)
    - [Launching the Vagrant Box](#launching-the-vagrant-box)
    - [Per Project Installation](#per-project-installation)
    - [Installing Optional Features](#installing-optional-features)
    - [Aliases](#aliases)
- [Updating Homestead](#updating-homestead)
- [Daily Usage](#daily-usage)
    - [Connecting via SSH](#connecting-via-ssh)
    - [Adding Additional Sites](#adding-additional-sites)
    - [Environment Variables](#environment-variables)
    - [Ports](#ports)
    - [PHP Versions](#php-versions)
    - [Connecting to Databases](#connecting-to-databases)
    - [Database Backups](#database-backups)
    - [Configuring Cron Schedules](#configuring-cron-schedules)
    - [Configuring Mailpit](#configuring-mailpit)
    - [Configuring Minio](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [Sharing Your Environment](#sharing-your-environment)
- [Debugging and Profiling](#debugging-and-profiling)
    - [Debugging Web Requests With Xdebug](#debugging-web-requests)
    - [Debugging CLI Applications](#debugging-cli-applications)
    - [Profiling Applications With Blackfire](#profiling-applications-with-blackfire)
- [Network Interfaces](#network-interfaces)
- [Extending Homestead](#extending-homestead)
- [Provider Specific Settings](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## Introduction

> [!WARNING]
> Laravel Homestead là một package cũ không còn được duy trì tích cực. [Laravel Sail](/docs/{{version}}/sail) có thể được sử dụng như một lựa chọn thay thế hiện đại.

Laravel nỗ lực làm cho toàn bộ trải nghiệm phát triển PHP trở nên thú vị, bao gồm cả môi trường phát triển cục bộ của bạn. [Laravel Homestead](https://github.com/laravel/homestead) là một Vagrant box chính thức, được đóng gói sẵn cung cấp cho bạn một môi trường phát triển tuyệt vời mà không yêu cầu bạn cài đặt PHP, web server, hoặc bất kỳ phần mềm server nào khác trên máy cục bộ của bạn.

[Vagrant](https://www.vagrantup.com) cung cấp một cách đơn giản, thanh lịch để quản lý và cung cấp các Máy Ảo. Vagrant boxes hoàn toàn có thể loại bỏ. Nếu có gì sai, bạn có thể hủy và tạo lại box trong vài phút!

Homestead chạy trên bất kỳ hệ thống Windows, macOS, hoặc Linux nào và bao gồm Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, và tất cả các phần mềm khác bạn cần để phát triển các ứng dụng Laravel tuyệt vời.

> [!WARNING]
> Nếu bạn đang sử dụng Windows, bạn có thể cần bật hardware virtualization (VT-x). Nó thường có thể được bật thông qua BIOS của bạn. Nếu bạn đang sử dụng Hyper-V trên hệ thống UEFI, bạn có thể cần tắt Hyper-V để truy cập VT-x.

<a name="included-software"></a>
### Included Software

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Ubuntu 22.04
- Git
- PHP 8.3
- PHP 8.2
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 15
- Composer
- Docker
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailpit
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

</div>

<a name="optional-software"></a>
### Optional Software

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Elasticsearch
- EventStoreDB
- Flyway
- Gearman
- Go
- Grafana
- InfluxDB
- Logstash
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- Rust
- RVM (Ruby Version Manager)
- Solr
- TimescaleDB
- Trader <small>(PHP extension)</small>
- Webdriver & Laravel Dusk Utilities

</div>

<a name="installation-and-setup"></a>
## Installation and Setup

<a name="first-steps"></a>
### First Steps

Trước khi khởi chạy môi trường Homestead của bạn, bạn phải cài đặt [Vagrant](https://developer.hashicorp.com/vagrant/downloads) cũng như một trong các providers được hỗ trợ sau:

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Download_Old_Builds_6_1)
- [Parallels](https://www.parallels.com/products/desktop/)

Tất cả các gói phần mềm này cung cấp các trình cài đặt trực quan dễ sử dụng cho tất cả các hệ điều hành phổ biến.

Để sử dụng provider Parallels, bạn sẽ cần cài đặt [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels). Nó miễn phí.

<a name="installing-homestead"></a>
#### Installing Homestead

Bạn có thể cài đặt Homestead bằng cách clone repository Homestead vào máy chủ của bạn. Hãy xem xét việc clone repository vào một thư mục `Homestead` trong thư mục "home" của bạn, vì máy ảo Homestead sẽ phục vụ như host cho tất cả các ứng dụng Laravel của bạn. Trong suốt tài liệu này, chúng tôi sẽ đề cập đến thư mục này là "thư mục Homestead" của bạn:

```shell
git clone https://github.com/laravel/homestead.git ~/Homestead
```

Sau khi clone repository Laravel Homestead, bạn nên checkout branch `release`. Branch này luôn chứa bản phát hành ổn định mới nhất của Homestead:

```shell
cd ~/Homestead

git checkout release
```

Tiếp theo, thực thi lệnh `bash init.sh` từ thư mục Homestead để tạo file cấu hình `Homestead.yaml`. File `Homestead.yaml` là nơi bạn sẽ cấu hình tất cả các cài đặt cho cài đặt Homestead của bạn. File này sẽ được đặt trong thư mục Homestead:

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

<a name="configuring-homestead"></a>
### Configuring Homestead

<a name="setting-your-provider"></a>
#### Setting Your Provider

Khóa `provider` trong file `Homestead.yaml` của bạn chỉ định Vagrant provider nào nên được sử dụng: `virtualbox` hoặc `parallels`:

    provider: virtualbox

> [!WARNING]
> Nếu bạn đang sử dụng Apple Silicon, provider Parallels là bắt buộc.

<a name="configuring-shared-folders"></a>
#### Configuring Shared Folders

Thuộc tính `folders` của file `Homestead.yaml` liệt kê tất cả các thư mục bạn muốn chia sẻ với môi trường Homestead của bạn. Khi các files trong các thư mục này thay đổi, chúng sẽ được giữ đồng bộ giữa máy cục bộ của bạn và môi trường ảo Homestead. Bạn có thể cấu hình bao nhiêu thư mục chia sẻ khi cần thiết:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> [!WARNING]
> Người dùng Windows không nên sử dụng cú pháp đường dẫn `~/` và thay vào đó nên sử dụng đường dẫn đầy đủ đến dự án của họ, chẳng hạn như `C:\Users\user\Code\project1`.

Bạn nên luôn map từng ứng dụng sang mapping thư mục riêng của nó thay vì map một thư mục lớn duy nhất chứa tất cả các ứng dụng của bạn. Khi bạn map một thư mục, máy ảo phải theo dõi tất cả disk IO cho *mọi* file trong thư mục. Bạn có thể trải nghiệm hiệu suất giảm nếu bạn có số lượng lớn files trong một thư mục:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> [!WARNING]
> Bạn không bao giờ nên mount `.` (thư mục hiện tại) khi sử dụng Homestead. Điều này khiến Vagrant không map thư mục hiện tại sang `/vagrant` và sẽ phá vỡ các tính năng tùy chọn và gây ra kết quả không mong đợi trong khi cung cấp.

Để bật [NFS](https://developer.hashicorp.com/vagrant/docs/synced-folders/nfs), bạn có thể thêm tùy chọn `type` vào mapping thư mục của bạn:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> [!WARNING]
> Khi sử dụng NFS trên Windows, bạn nên xem xét cài đặt plug-in [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd). Plug-in này sẽ duy trì quyền user / group chính xác cho các files và thư mục trong máy ảo Homestead.

Bạn cũng có thể chuyển bất kỳ tùy chọn nào được hỗ trợ bởi [Synced Folders](https://developer.hashicorp.com/vagrant/docs/synced-folders/basic_usage) của Vagrant bằng cách liệt kê chúng dưới khóa `options`:

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

<a name="configuring-nginx-sites"></a>
### Configuring Nginx Sites

Không quen với Nginx? Không vấn đề gì. Thuộc tính `sites` của file `Homestead.yaml` của bạn cho phép bạn dễ dàng map một "domain" đến một thư mục trong môi trường Homestead của bạn. Một cấu hình site mẫu được bao gồm trong file `Homestead.yaml`. Một lần nữa, bạn có thể thêm bao nhiêu site vào môi trường Homestead của bạn khi cần thiết. Homestead có thể phục vụ như một môi trường ảo hóa thuận tiện cho mọi ứng dụng Laravel bạn đang làm việc:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

Nếu bạn thay đổi thuộc tính `sites` sau khi cung cấp máy ảo Homestead, bạn nên thực thi lệnh `vagrant reload --provision` trong terminal để cập nhật cấu hình Nginx trên máy ảo.

> [!WARNING]
> Các script Homestead được xây dựng để càng idempotent càng tốt. Tuy nhiên, nếu bạn đang gặp vấn đề trong khi cung cấp, bạn nên hủy và xây dựng lại máy bằng cách thực thi lệnh `vagrant destroy && vagrant up`.

<a name="hostname-resolution"></a>
#### Hostname Resolution

Homestead xuất bản hostnames bằng cách sử dụng `mDNS` để phân giải hostname tự động. Nếu bạn đặt `hostname: homestead` trong file `Homestead.yaml` của bạn, host sẽ có sẵn tại `homestead.local`. macOS, iOS, và các bản phân phối Linux desktop bao gồm hỗ trợ `mDNS` theo mặc định. Nếu bạn đang sử dụng Windows, bạn phải cài đặt [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US).

Sử dụng hostnames tự động hoạt động tốt nhất cho các cài đặt Homestead [cho mỗi dự án](#per-project-installation). Nếu bạn lưu trữ nhiều sites trên một instance Homestead duy nhất, bạn có thể thêm "domains" cho các web sites của bạn vào file `hosts` trên máy của bạn. File `hosts` sẽ chuyển hướng các requests cho các sites Homestead của bạn vào máy ảo Homestead. Trên macOS và Linux, file này nằm tại `/etc/hosts`. Trên Windows, nó nằm tại `C:\Windows\System32\drivers\etc\hosts`. Các dòng bạn thêm vào file này sẽ trông như sau:

```text
192.168.56.56  homestead.test
```

Hãy đảm bảo địa chỉ IP được liệt kê là địa chỉ được đặt trong file `Homestead.yaml` của bạn. Sau khi bạn đã thêm domain vào file `hosts` của bạn và khởi chạy Vagrant box, bạn sẽ có thể truy cập site thông qua trình duyệt web:

```shell
http://homestead.test
```

<a name="configuring-services"></a>
### Configuring Services

Homestead khởi động một số dịch vụ theo mặc định; tuy nhiên, bạn có thể tùy chỉnh các dịch vụ nào được bật hoặc tắt trong quá trình cung cấp. Ví dụ, bạn có thể bật PostgreSQL và tắt MySQL bằng cách sửa đổi tùy chọn `services` trong file `Homestead.yaml` của bạn:

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

Các dịch vụ được chỉ định sẽ được khởi động hoặc dừng dựa trên thứ tự của chúng trong các chỉ thị `enabled` và `disabled`.

<a name="launching-the-vagrant-box"></a>
### Launching the Vagrant Box

Sau khi bạn đã chỉnh sửa `Homestead.yaml` theo ý mình, hãy chạy lệnh `vagrant up` từ thư mục Homestead của bạn. Vagrant sẽ khởi động máy ảo và tự động cấu hình các thư mục chia sẻ và các sites Nginx của bạn.

Để hủy máy, bạn có thể sử dụng lệnh `vagrant destroy`.

<a name="per-project-installation"></a>
### Per Project Installation

Thay vì cài đặt Homestead toàn cầu và chia sẻ cùng một máy ảo Homestead trên tất cả các dự án của bạn, bạn có thể thay vào đó cấu hình một instance Homestead cho mỗi dự án bạn quản lý. Cài đặt Homestead cho mỗi dự án có thể có lợi nếu bạn muốn gửi một `Vagrantfile` với dự án của bạn, cho phép những người khác làm việc trên dự án `vagrant up` ngay lập tức sau khi clone repository dự án.

Bạn có thể cài đặt Homestead vào dự án của bạn bằng cách sử dụng trình quản lý package Composer:

```shell
composer require laravel/homestead --dev
```

Sau khi Homestead đã được cài đặt, hãy gọi lệnh `make` của Homestead để tạo file `Vagrantfile` và `Homestead.yaml` cho dự án của bạn. Các file này sẽ được đặt trong root của dự án. Lệnh `make` sẽ tự động cấu hình các chỉ thị `sites` và `folders` trong file `Homestead.yaml`:

```shell
# macOS / Linux...
php vendor/bin/homestead make

# Windows...
vendor\\bin\\homestead make
```

Tiếp theo, chạy lệnh `vagrant up` trong terminal và truy cập dự án của bạn tại `http://homestead.test` trong trình duyệt. Hãy nhớ, bạn vẫn cần thêm một mục file `/etc/hosts` cho `homestead.test` hoặc domain bạn chọn nếu bạn không sử dụng [phân giải hostname](#hostname-resolution) tự động.

<a name="installing-optional-features"></a>
### Installing Optional Features

Phần mềm tùy chọn được cài đặt bằng cách sử dụng tùy chọn `features` trong file `Homestead.yaml` của bạn. Hầu hết các tính năng có thể được bật hoặc tắt với một giá trị boolean, trong khi một số tính năng cho phép nhiều tùy chọn cấu hình:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
    - cassandra: true
    - chronograf: true
    - couchdb: true
    - crystal: true
    - dragonflydb: true
    - elasticsearch:
        version: 7.9.0
    - eventstore: true
        version: 21.2.0
    - flyway: true
    - gearman: true
    - golang: true
    - grafana: true
    - influxdb: true
    - logstash: true
    - mariadb: true
    - meilisearch: true
    - minio: true
    - mongodb: true
    - neo4j: true
    - ohmyzsh: true
    - openresty: true
    - pm2: true
    - python: true
    - r-base: true
    - rabbitmq: true
    - rustc: true
    - rvm: true
    - solr: true
    - timescaledb: true
    - trader: true
    - webdriver: true
```

<a name="elasticsearch"></a>
#### Elasticsearch

Bạn có thể chỉ định một phiên bản được hỗ trợ của Elasticsearch, phải là một số phiên bản chính xác (major.minor.patch). Cài đặt mặc định sẽ tạo một cluster có tên 'homestead'. Bạn không bao giờ nên cung cấp cho Elasticsearch nhiều hơn một nửa bộ nhớ của hệ điều hành, vì vậy hãy đảm bảo máy ảo Homestead của bạn có ít nhất gấp đôi phân bổ Elasticsearch.

> [!NOTE]
> Hãy xem [tài liệu Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current) để tìm hiểu cách tùy chỉnh cấu hình của bạn.

<a name="mariadb"></a>
#### MariaDB

Bật MariaDB sẽ xóa MySQL và cài đặt MariaDB. MariaDB thường phục vụ như một thay thế trực tiếp cho MySQL, vì vậy bạn vẫn nên sử dụng driver database `mysql` trong cấu hình database của ứng dụng.

<a name="mongodb"></a>
#### MongoDB

Cài đặt MongoDB mặc định sẽ đặt tên người dùng database thành `homestead` và mật khẩu tương ứng thành `secret`.

<a name="neo4j"></a>
#### Neo4j

Cài đặt Neo4j mặc định sẽ đặt tên người dùng database thành `homestead` và mật khẩu tương ứng thành `secret`. Để truy cập trình duyệt Neo4j, hãy truy cập `http://homestead.test:7474` thông qua trình duyệt web của bạn. Các port `7687` (Bolt), `7474` (HTTP), và `7473` (HTTPS) đã sẵn sàng phục vụ các requests từ client Neo4j.

<a name="aliases"></a>
### Aliases

Bạn có thể thêm Bash aliases vào máy ảo Homestead của bạn bằng cách sửa đổi file `aliases` trong thư mục Homestead của bạn:

```shell
alias c='clear'
alias ..='cd ..'
```

Sau khi bạn đã cập nhật file `aliases`, bạn nên cung cấp lại máy ảo Homestead bằng cách sử dụng lệnh `vagrant reload --provision`. Điều này sẽ đảm bảo rằng các aliases mới của bạn có sẵn trên máy.

<a name="updating-homestead"></a>
## Updating Homestead

Trước khi bạn bắt đầu cập nhật Homestead, bạn nên đảm bảo bạn đã xóa máy ảo hiện tại của mình bằng cách chạy lệnh sau trong thư mục Homestead của bạn:

```shell
vagrant destroy
```

Tiếp theo, bạn cần cập nhật mã nguồn Homestead. Nếu bạn đã clone repository, bạn có thể thực thi các lệnh sau tại vị trí bạn đã clone repository ban đầu:

```shell
git fetch

git pull origin release
```

Các lệnh này kéo mã Homestead mới nhất từ repository GitHub, lấy các tags mới nhất, sau đó checkout bản phát hành được tag mới nhất. Bạn có thể tìm phiên bản phát hành ổn định mới nhất trên [trang phát hành GitHub](https://github.com/laravel/homestead/releases) của Homestead.

Nếu bạn đã cài đặt Homestead thông qua file `composer.json` của dự án, bạn nên đảm bảo file `composer.json` của bạn chứa `"laravel/homestead": "^12"` và cập nhật các dependencies của bạn:

```shell
composer update
```

Tiếp theo, bạn nên cập nhật Vagrant box bằng cách sử dụng lệnh `vagrant box update`:

```shell
vagrant box update
```

Sau khi cập nhật Vagrant box, bạn nên chạy lệnh `bash init.sh` từ thư mục Homestead để cập nhật các file cấu hình bổ sung của Homestead. Bạn sẽ được hỏi xem bạn có muốn ghi đè các file `Homestead.yaml`, `after.sh`, và `aliases` hiện có của mình hay không:

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

Cuối cùng, bạn sẽ cần tạo lại máy ảo Homestead để sử dụng cài đặt Vagrant mới nhất:

```shell
vagrant up
```

<a name="daily-usage"></a>
## Daily Usage

<a name="connecting-via-ssh"></a>
### Connecting via SSH

Bạn có thể SSH vào máy ảo của mình bằng cách thực thi lệnh terminal `vagrant ssh` từ thư mục Homestead của bạn.

<a name="adding-additional-sites"></a>
### Adding Additional Sites

Sau khi môi trường Homestead của bạn đã được cung cấp và chạy, bạn có thể muốn thêm các sites Nginx bổ sung cho các dự án Laravel khác của bạn. Bạn có thể chạy bao nhiêu dự án Laravel tùy ý trên một môi trường Homestead duy nhất. Để thêm một site bổ sung, hãy thêm site vào file `Homestead.yaml` của bạn.

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

> [!WARNING]
> Bạn nên đảm bảo rằng bạn đã cấu hình một [mapping thư mục](#configuring-shared-folders) cho thư mục dự án trước khi thêm site.

Nếu Vagrant không tự động quản lý file "hosts" của bạn, bạn có thể cần thêm site mới vào file đó cũng vậy. Trên macOS và Linux, file này nằm tại `/etc/hosts`. Trên Windows, nó nằm tại `C:\Windows\System32\drivers\etc\hosts`:

```text
192.168.56.56  homestead.test
192.168.56.56  another.test
```

Sau khi site đã được thêm, hãy thực thi lệnh terminal `vagrant reload --provision` từ thư mục Homestead của bạn.

<a name="site-types"></a>
#### Site Types

Homestead hỗ trợ một số "types" của sites cho phép bạn dễ dàng chạy các dự án không dựa trên Laravel. Ví dụ, chúng ta có thể dễ dàng thêm một ứng dụng Statamic vào Homestead bằng cách sử dụng type site `statamic`:

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

Các types site có sẵn là: `apache`, `apache-proxy`, `apigility`, `expressive`, `laravel` (mặc định), `proxy` (cho nginx), `silverstripe`, `statamic`, `symfony2`, `symfony4`, và `zf`.

<a name="site-parameters"></a>
#### Site Parameters

Bạn có thể thêm các giá trị Nginx `fastcgi_param` bổ sung vào site của bạn thông qua chỉ thị site `params`:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      params:
          - key: FOO
            value: BAR
```

<a name="environment-variables"></a>
### Environment Variables

Bạn có thể định nghĩa các biến môi trường toàn cầu bằng cách thêm chúng vào file `Homestead.yaml` của bạn:

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

Sau khi cập nhật file `Homestead.yaml`, hãy đảm bảo cung cấp lại máy bằng cách thực thi lệnh `vagrant reload --provision`. Điều này sẽ cập nhật cấu hình PHP-FPM cho tất cả các phiên bản PHP được cài đặt và cũng cập nhật môi trường cho user `vagrant`.

<a name="ports"></a>
### Ports

Theo mặc định, các port sau được chuyển tiếp đến môi trường Homestead của bạn:

<div class="content-list" markdown="1">

- **HTTP:** 8000 &rarr; Chuyển tiếp đến 80
- **HTTPS:** 44300 &rarr; Chuyển tiếp đến 443

</div>

<a name="forwarding-additional-ports"></a>
#### Forwarding Additional Ports

Nếu bạn muốn, bạn có thể chuyển tiếp các port bổ sung đến Vagrant box bằng cách định nghĩa một mục cấu hình `ports` trong file `Homestead.yaml` của bạn. Sau khi cập nhật file `Homestead.yaml`, hãy đảm bảo cung cấp lại máy bằng cách thực thi lệnh `vagrant reload --provision`:

```yaml
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

Dưới đây là danh sách các port dịch vụ Homestead bổ sung mà bạn có thể muốn map từ máy chủ của bạn đến Vagrant box:

<div class="content-list" markdown="1">

- **SSH:** 2222 &rarr; Đến 22
- **ngrok UI:** 4040 &rarr; Đến 4040
- **MySQL:** 33060 &rarr; Đến 3306
- **PostgreSQL:** 54320 &rarr; Đến 5432
- **MongoDB:** 27017 &rarr; Đến 27017
- **Mailpit:** 8025 &rarr; Đến 8025
- **Minio:** 9600 &rarr; Đến 9600

</div>

<a name="php-versions"></a>
### PHP Versions

Homestead hỗ trợ chạy nhiều phiên bản PHP trên cùng một máy ảo. Bạn có thể chỉ định phiên bản PHP nào để sử dụng cho một site nhất định trong file `Homestead.yaml` của bạn. Các phiên bản PHP có sẵn là: "5.6", "7.0", "7.1", "7.2", "7.3", "7.4", "8.0", "8.1", "8.2", và "8.3", (mặc định):

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

[Trong máy ảo Homestead của bạn](#connecting-via-ssh), bạn có thể sử dụng bất kỳ phiên bản PHP được hỗ trợ nào thông qua CLI:

```shell
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
php8.0 artisan list
php8.1 artisan list
php8.2 artisan list
php8.3 artisan list
```

Bạn có thể thay đổi phiên bản PHP mặc định được sử dụng bởi CLI bằng cách đưa ra các lệnh sau từ trong máy ảo Homestead của bạn:

```shell
php56
php70
php71
php72
php73
php74
php80
php81
php82
php83
```

<a name="connecting-to-databases"></a>
### Connecting to Databases

Một database `homestead` được cấu hình cho cả MySQL và PostgreSQL sẵn sàng. Để kết nối với database MySQL hoặc PostgreSQL của bạn từ client database máy chủ của bạn, bạn nên kết nối đến `127.0.0.1` trên port `33060` (MySQL) hoặc `54320` (PostgreSQL). Tên người dùng và mật khẩu cho cả hai database là `homestead` / `secret`.

> [!WARNING]
> Bạn chỉ nên sử dụng các port không chuẩn này khi kết nối đến các databases từ máy chủ của bạn. Bạn sẽ sử dụng các port mặc định 3306 và 5432 trong file cấu hình `database` của ứng dụng Laravel vì Laravel chạy _trong_ máy ảo.

<a name="database-backups"></a>
### Database Backups

Homestead có thể tự động backup database của bạn khi máy ảo Homestead của bạn bị hủy. Để sử dụng tính năng này, bạn phải sử dụng Vagrant 2.1.0 hoặc cao hơn. Hoặc, nếu bạn đang sử dụng phiên bản Vagrant cũ hơn, bạn phải cài đặt plug-in `vagrant-triggers`. Để bật các backup database tự động, hãy thêm dòng sau vào file `Homestead.yaml` của bạn:

```yaml
backup: true
```

Sau khi được cấu hình, Homestead sẽ xuất các databases của bạn sang các thư mục `.backup/mysql_backup` và `.backup/postgres_backup` khi lệnh `vagrant destroy` được thực thi. Các thư mục này có thể được tìm thấy trong thư mục nơi bạn đã cài đặt Homestead hoặc trong root của dự án nếu bạn đang sử dụng phương pháp [cài đặt cho mỗi dự án](#per-project-installation).

<a name="configuring-cron-schedules"></a>
### Configuring Cron Schedules

Laravel cung cấp một cách thuận tiện để [lên lịch cron jobs](/docs/{{version}}/scheduling) bằng cách lên lịch một lệnh Artisan `schedule:run` duy nhất để chạy mỗi phút. Lệnh `schedule:run` sẽ kiểm tra lịch công việc được định nghĩa trong file `routes/console.php` của bạn để xác định các tác vụ đã lên lịch nào để chạy.

Nếu bạn muốn lệnh `schedule:run` được chạy cho một site Homestead, bạn có thể đặt tùy chọn `schedule` thành `true` khi định nghĩa site:

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

Cron job cho site sẽ được định nghĩa trong thư mục `/etc/cron.d` của máy ảo Homestead.

<a name="configuring-mailpit"></a>
### Configuring Mailpit

[Mailpit](https://github.com/axllent/mailpit) cho phép bạn chặn email đi của mình và kiểm tra nó mà không thực sự gửi mail đến người nhận. Để bắt đầu, hãy cập nhật file `.env` của ứng dụng để sử dụng các cài đặt mail sau:

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

Sau khi Mailpit đã được cấu hình, bạn có thể truy cập dashboard Mailpit tại `http://localhost:8025`.

<a name="configuring-minio"></a>
### Configuring Minio

[Minio](https://github.com/minio/minio) là một server lưu trữ đối tượng mã nguồn mở với API tương thích Amazon S3. Để cài đặt Minio, hãy cập nhật file `Homestead.yaml` của bạn với tùy chọn cấu hình sau trong phần [features](#installing-optional-features):

    minio: true

Theo mặc định, Minio có sẵn trên port 9600. Bạn có thể truy cập bảng điều khiển Minio bằng cách truy cập `http://localhost:9600`. Khóa truy cập mặc định là `homestead`, trong khi khóa bí mật mặc định là `secretkey`. Khi truy cập Minio, bạn nên luôn sử dụng region `us-east-1`.

Để sử dụng Minio, hãy đảm bảo file `.env` của bạn có các tùy chọn sau:

```ini
AWS_USE_PATH_STYLE_ENDPOINT=true
AWS_ENDPOINT=http://localhost:9600
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
```

Để cung cấp các buckets "S3" được hỗ trợ bởi Minio, hãy thêm chỉ thị `buckets` vào file `Homestead.yaml` của bạn. Sau khi định nghĩa các buckets của bạn, bạn nên thực thi lệnh `vagrant reload --provision` trong terminal:

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

Các giá trị `policy` được hỗ trợ bao gồm: `none`, `download`, `upload`, và `public`.

<a name="laravel-dusk"></a>
### Laravel Dusk

Để chạy các tests [Laravel Dusk](/docs/{{version}}/dusk) trong Homestead, bạn nên bật [tính năng webdriver](#installing-optional-features) trong cấu hình Homestead của bạn:

```yaml
features:
    - webdriver: true
```

Sau khi bật tính năng `webdriver`, bạn nên thực thi lệnh `vagrant reload --provision` trong terminal.

<a name="sharing-your-environment"></a>
### Sharing Your Environment

Đôi khi bạn có thể muốn chia sẻ những gì bạn đang làm việc với đồng nghiệp hoặc khách hàng. Vagrant có hỗ trợ tích hợp cho điều này thông qua lệnh `vagrant share`; tuy nhiên, điều này sẽ không hoạt động nếu bạn có nhiều sites được cấu hình trong file `Homestead.yaml` của bạn.

Để giải quyết vấn đề này, Homestead bao gồm lệnh `share` riêng của nó. Để bắt đầu, [SSH vào máy ảo Homestead của bạn](#connecting-via-ssh) thông qua `vagrant ssh` và thực thi lệnh `share homestead.test`. Lệnh này sẽ chia sẻ site `homestead.test` từ file cấu hình `Homestead.yaml` của bạn. Bạn có thể thay thế bất kỳ sites nào khác được cấu hình của bạn cho `homestead.test`:

```shell
share homestead.test
```

Sau khi chạy lệnh, bạn sẽ thấy màn hình Ngrok xuất hiện chứa nhật ký hoạt động và các URL có thể truy cập công khai cho site được chia sẻ. Nếu bạn muốn chỉ định một region tùy chỉnh, subdomain, hoặc tùy chọn runtime Ngrok khác, bạn có thể thêm chúng vào lệnh `share` của bạn:

```shell
share homestead.test -region=eu -subdomain=laravel
```

Nếu bạn cần chia sẻ nội dung qua HTTPS thay vì HTTP, sử dụng lệnh `sshare` thay vì `share` sẽ cho phép bạn làm điều đó.

> [!WARNING]
> Hãy nhớ, Vagrant vốn dĩ không an toàn và bạn đang tiếp xúc máy ảo của mình với Internet khi chạy lệnh `share`.

<a name="debugging-and-profiling"></a>
## Debugging and Profiling

<a name="debugging-web-requests"></a>
### Debugging Web Requests With Xdebug

Homestead bao gồm hỗ trợ cho debugging từng bước bằng cách sử dụng [Xdebug](https://xdebug.org). Ví dụ, bạn có thể truy cập một trang trong trình duyệt và PHP sẽ kết nối với IDE của bạn để cho phép kiểm tra và sửa đổi mã đang chạy.

Theo mặc định, Xdebug đã chạy và sẵn sàng chấp nhận các kết nối. Nếu bạn cần bật Xdebug trên CLI, hãy thực thi lệnh `sudo phpenmod xdebug` trong máy ảo Homestead của bạn. Tiếp theo, hãy làm theo hướng dẫn của IDE để bật debugging. Cuối cùng, cấu hình trình duyệt của bạn để kích hoạt Xdebug với một extension hoặc [bookmarklet](https://www.jetbrains.com/phpstorm/marklets/).

> [!WARNING]
> Xdebug khiến PHP chạy chậm hơn đáng kể. Để tắt Xdebug, hãy chạy `sudo phpdismod xdebug` trong máy ảo Homestead của bạn và khởi động lại dịch vụ FPM.

<a name="autostarting-xdebug"></a>
#### Autostarting Xdebug

Khi debugging các tests chức năng thực hiện các requests đến web server, dễ dàng hơn để tự động bắt đầu debugging thay vì sửa đổi các tests để chuyển qua một header hoặc cookie tùy chỉnh để kích hoạt debugging. Để buộc Xdebug tự động bắt đầu, hãy sửa đổi file `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` trong máy ảo Homestead của bạn và thêm cấu hình sau:

```ini
; Nếu Homestead.yaml chứa một subnet khác cho địa chỉ IP, địa chỉ này có thể khác...
xdebug.client_host = 192.168.10.1
xdebug.mode = debug
xdebug.start_with_request = yes
```

<a name="debugging-cli-applications"></a>
### Debugging CLI Applications

Để debug một ứng dụng CLI PHP, hãy sử dụng alias shell `xphp` trong máy ảo Homestead của bạn:

```shell
xphp /path/to/script
```

<a name="profiling-applications-with-blackfire"></a>
### Profiling Applications With Blackfire

[Blackfire](https://blackfire.io/docs/introduction) là một dịch vụ để profiling các requests web và ứng dụng CLI. Nó cung cấp một giao diện người dùng tương tác hiển thị dữ liệu profile trong các biểu đồ gọi và dòng thời gian. Nó được xây dựng để sử dụng trong phát triển, staging, và sản xuất, không có chi phí cho người dùng cuối. Ngoài ra, Blackfire cung cấp các kiểm tra hiệu suất, chất lượng và bảo mật trên mã và cài đặt cấu hình `php.ini`.

[Blackfire Player](https://blackfire.io/docs/player/index) là một ứng dụng Web Crawling, Web Testing, và Web Scraping mã nguồn mở có thể hoạt động cùng với Blackfire để kịch bản các kịch bản profiling.

Để bật Blackfire, hãy sử dụng cài đặt "features" trong file cấu hình Homestead của bạn:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Thông tin đăng nhập server Blackfire và thông tin đăng nhập client [yêu cầu một tài khoản Blackfire](https://blackfire.io/signup). Blackfire cung cấp nhiều tùy chọn để profile một ứng dụng, bao gồm một công cụ CLI và extension trình duyệt. Vui lòng [xem tài liệu Blackfire để biết thêm chi tiết](https://blackfire.io/docs/php/integrations/laravel/index).

<a name="network-interfaces"></a>
## Network Interfaces

Thuộc tính `networks` của file `Homestead.yaml` cấu hình các giao diện mạng cho máy ảo Homestead của bạn. Bạn có thể cấu hình bao nhiêu giao diện khi cần thiết:

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

Để bật giao diện [bridged](https://developer.hashicorp.com/vagrant/docs/networking/public_network), hãy cấu hình cài đặt `bridge` cho mạng và thay đổi loại mạng thành `public_network`:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

Để bật [DHCP](https://developer.hashicorp.com/vagrant/docs/networking/public_network#dhcp), chỉ cần xóa tùy chọn `ip` khỏi cấu hình của bạn:

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

Để cập nhật thiết bị mạng đang sử dụng, bạn có thể thêm tùy chọn `dev` vào cấu hình mạng. Giá trị `dev` mặc định là `eth0`:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
      dev: "enp2s0"
```

<a name="extending-homestead"></a>
## Extending Homestead

Bạn có thể mở rộng Homestead bằng cách sử dụng script `after.sh` trong root của thư mục Homestead của bạn. Trong file này, bạn có thể thêm bất kỳ lệnh shell nào cần thiết để cấu hình và tùy chỉnh máy ảo của bạn đúng cách.

Khi tùy chỉnh Homestead, Ubuntu có thể hỏi bạn xem bạn có muốn giữ cấu hình gốc của một gói hoặc ghi đè nó bằng một file cấu hình mới. Để tránh điều này, bạn nên sử dụng lệnh sau khi cài đặt các gói để tránh ghi đè bất kỳ cấu hình nào đã được viết trước bởi Homestead:

```shell
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

<a name="user-customizations"></a>
### User Customizations

Khi sử dụng Homestead với nhóm của bạn, bạn có thể muốn tinh chỉnh Homestead để phù hợp hơn với phong cách phát triển cá nhân của bạn. Để thực hiện điều này, bạn có thể tạo một file `user-customizations.sh` trong root của thư mục Homestead của bạn (cùng thư mục chứa file `Homestead.yaml` của bạn). Trong file này, bạn có thể thực hiện bất kỳ tùy chỉnh nào bạn muốn; tuy nhiên, `user-customizations.sh` không nên được kiểm soát phiên bản.

<a name="provider-specific-settings"></a>
## Provider Specific Settings

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

Theo mặc định, Homestead cấu hình cài đặt `natdnshostresolver` thành `on`. Điều này cho phép Homestead sử dụng cài đặt DNS của hệ điều hành máy chủ của bạn. Nếu bạn muốn ghi đè hành vi này, hãy thêm các tùy chọn cấu hình sau vào file `Homestead.yaml` của bạn:

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```

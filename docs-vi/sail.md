# Laravel Sail

- [Giới thiệu](#introduction)
- [Cài đặt và Thiết lập](#installation)
    - [Rebuild Sail Images](#rebuilding-sail-images)
    - [Cấu hình Shell Alias](#configuring-a-shell-alias)
- [Khởi động và Dừng Sail](#starting-and-stopping-sail)
- [Thực thi Commands](#executing-sail-commands)
    - [Thực thi PHP Commands](#executing-php-commands)
    - [Thực thi Composer Commands](#executing-composer-commands)
    - [Thực thi Artisan Commands](#executing-artisan-commands)
    - [Thực thi Node / NPM Commands](#executing-node-npm-commands)
- [Tương tác với Databases](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [MongoDB](#mongodb)
    - [Redis](#redis)
    - [Valkey](#valkey)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [File Storage](#file-storage)
- [Chạy Tests](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [Preview Emails](#previewing-emails)
- [Container CLI](#sail-container-cli)
- [PHP Versions](#sail-php-versions)
- [Node Versions](#sail-node-versions)
- [Chia sẻ Site của bạn](#sharing-your-site)
- [Debugging với Xdebug](#debugging-with-xdebug)
  - [Xdebug CLI Usage](#xdebug-cli-usage)
  - [Xdebug Browser Usage](#xdebug-browser-usage)
- [Tùy chỉnh](#sail-customization)

<a name="introduction"></a>
## Giới thiệu

[Laravel Sail](https://github.com/laravel/sail) là một command-line interface nhẹ để tương tác với môi trường phát triển Docker mặc định của Laravel. Sail cung cấp một điểm khởi đầu tuyệt vời để xây dựng một ứng dụng Laravel sử dụng PHP, MySQL, và Redis mà không cần kinh nghiệm Docker trước đó.

Trên thực chất, Sail là file `compose.yaml` và script `sail` được lưu tại root của dự án của bạn. Script `sail` cung cấp một CLI với các phương thức tiện lợi để tương tác với các Docker containers được định nghĩa bởi file `compose.yaml`.

Laravel Sail được hỗ trợ trên macOS, Linux, và Windows (thông qua [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)).

<a name="installation"></a>
## Cài đặt và Thiết lập

Bạn có thể cài đặt Sail bằng Composer package manager:

```shell
composer require laravel/sail --dev
```

Sau khi Sail đã được cài đặt, bạn có thể chạy lệnh Artisan `sail:install`. Lệnh này sẽ publish file `compose.yaml` của Sail đến root của ứng dụng và sửa đổi file `.env` của bạn với các biến môi trường cần thiết để kết nối với các Docker services:

```shell
php artisan sail:install
```

Cuối cùng, bạn có thể khởi động Sail. Để tiếp tục học cách sử dụng Sail, hãy tiếp tục đọc phần còn lại của tài liệu này:

```shell
./vendor/bin/sail up
```

> [!WARNING]
> Nếu bạn đang sử dụng Docker Desktop cho Linux, bạn nên sử dụng Docker context `default` bằng cách thực thi lệnh sau: `docker context use default`. Ngoài ra, nếu bạn gặp lỗi quyền file trong containers, bạn có thể cần đặt biến môi trường `SUPERVISOR_PHP_USER` thành `root`.

<a name="adding-additional-services"></a>
#### Thêm Additional Services

Nếu bạn muốn thêm một service bổ sung vào cài đặt Sail hiện có của mình, bạn có thể chạy lệnh Artisan `sail:add`:

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### Sử dụng Devcontainers

Nếu bạn muốn phát triển trong một [Devcontainer](https://code.visualstudio.com/docs/remote/containers), bạn có thể cung cấp tùy chọn `--devcontainer` cho lệnh `sail:install`. Tùy chọn `--devcontainer` sẽ hướng dẫn lệnh `sail:install` để publish một file `.devcontainer/devcontainer.json` mặc định đến root của ứng dụng:

```shell
php artisan sail:install --devcontainer
```

<a name="rebuilding-sail-images"></a>
### Rebuild Sail Images

Đôi khi bạn có thể muốn rebuild hoàn toàn các Sail images của mình để đảm bảo tất cả các packages và phần mềm của image đều được cập nhật. Bạn có thể thực hiện điều này bằng lệnh `build`:

```shell
docker compose down -v

sail build --no-cache

sail up
```

<a name="configuring-a-shell-alias"></a>
### Cấu hình Shell Alias

Theo mặc định, các lệnh Sail được gọi bằng script `vendor/bin/sail` được bao gồm với tất cả các ứng dụng Laravel mới:

```shell
./vendor/bin/sail up
```

Tuy nhiên, thay vì gõ `vendor/bin/sail` lặp đi lặp lại để thực thi các lệnh Sail, bạn có thể muốn cấu hình một shell alias cho phép bạn thực thi các lệnh của Sail dễ dàng hơn:

```shell
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

Để đảm bảo điều này luôn có sẵn, bạn có thể thêm điều này vào file cấu hình shell trong thư mục home của bạn, chẳng hạn như `~/.zshrc` hoặc `~/.bashrc`, và sau đó khởi động lại shell của bạn.

Sau khi shell alias đã được cấu hình, bạn có thể thực thi các lệnh Sail bằng cách đơn giản gõ `sail`. Các ví dụ còn lại của tài liệu này sẽ giả định rằng bạn đã cấu hình alias này:

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Khởi động và Dừng Sail

File `compose.yaml` của Laravel Sail định nghĩa một loạt các Docker containers hoạt động cùng nhau để giúp bạn xây dựng các ứng dụng Laravel. Mỗi container này là một entry trong cấu hình `services` của file `compose.yaml` của bạn. Container `laravel.test` là container ứng dụng chính sẽ phục vụ ứng dụng của bạn.

Trước khi khởi động Sail, bạn nên đảm bảo rằng không có web servers hoặc databases nào khác đang chạy trên máy tính local của bạn. Để khởi động tất cả các Docker containers được định nghĩa trong file `compose.yaml` của ứng dụng, bạn nên thực thi lệnh `up`:

```shell
sail up
```

Để khởi động tất cả các Docker containers trong nền, bạn có thể khởi động Sail ở chế độ "detached":

```shell
sail up -d
```

Sau khi các containers của ứng dụng đã được khởi động, bạn có thể truy cập dự án trong trình duyệt web của mình tại: http://localhost.

Để dừng tất cả các containers, bạn có thể đơn giản nhấn Control + C để dừng thực thi của container. Hoặc, nếu các containers đang chạy trong nền, bạn có thể sử dụng lệnh `stop`:

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## Thực thi Commands

Khi sử dụng Laravel Sail, ứng dụng của bạn đang thực thi trong một Docker container và được cô lập khỏi máy tính local của bạn. Tuy nhiên, Sail cung cấp một cách tiện lợi để chạy các lệnh khác nhau đối với ứng dụng của bạn như các lệnh PHP tùy ý, lệnh Artisan, lệnh Composer, và lệnh Node / NPM.

**Khi đọc tài liệu Laravel, bạn sẽ thường thấy các tham chiếu đến các lệnh Composer, Artisan, và Node / NPM không tham chiếu đến Sail.** Các ví dụ đó giả định rằng các công cụ này được cài đặt trên máy tính local của bạn. Nếu bạn đang sử dụng Sail cho môi trường phát triển Laravel local của mình, bạn nên thực thi các lệnh đó bằng Sail:

```shell
# Chạy lệnh Artisan local...
php artisan queue:work

# Chạy lệnh Artisan trong Laravel Sail...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### Thực thi PHP Commands

Các lệnh PHP có thể được thực thi bằng lệnh `php`. Tất nhiên, các lệnh này sẽ thực thi bằng phiên bản PHP được cấu hình cho ứng dụng của bạn. Để tìm hiểu thêm về các phiên bản PHP có sẵn cho Laravel Sail, hãy tham khảo [tài liệu phiên bản PHP](#sail-php-versions):

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Thực thi Composer Commands

Các lệnh Composer có thể được thực thi bằng lệnh `composer`. Container ứng dụng của Laravel Sail bao gồm một cài đặt Composer:

```shell
sail composer require laravel/sanctum
```

<a name="executing-artisan-commands"></a>
### Thực thi Artisan Commands

Các lệnh Laravel Artisan có thể được thực thi bằng lệnh `artisan`:

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Thực thi Node / NPM Commands

Các lệnh Node có thể được thực thi bằng lệnh `node` trong khi các lệnh NPM có thể được thực thi bằng lệnh `npm`:

```shell
sail node --version

sail npm run dev
```

Nếu bạn muốn, bạn có thể sử dụng Yarn thay vì NPM:

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## Tương tác với Databases

<a name="mysql"></a>
### MySQL

Như bạn có thể đã nhận thấy, file `compose.yaml` của ứng dụng chứa một entry cho container MySQL. Container này sử dụng một [Docker volume](https://docs.docker.com/storage/volumes/) để dữ liệu được lưu trữ trong database của bạn được duy trì ngay cả khi dừng và khởi động lại các containers của bạn.

Ngoài ra, lần đầu tiên container MySQL khởi động, nó sẽ tạo hai databases cho bạn. Database đầu tiên được đặt tên bằng giá trị của biến môi trường `DB_DATABASE` của bạn và dành cho phát triển local của bạn. Database thứ hai là một database testing chuyên dụng tên là `testing` và sẽ đảm bảo rằng các tests của bạn không can thiệp vào dữ liệu phát triển của bạn.

Sau khi bạn đã khởi động các containers, bạn có thể kết nối với instance MySQL trong ứng dụng bằng cách đặt biến môi trường `DB_HOST` trong file `.env` của ứng dụng thành `mysql`.

Để kết nối với database MySQL của ứng dụng từ máy local của bạn, bạn có thể sử dụng một ứng dụng quản lý database đồ họa như [TablePlus](https://tableplus.com). Theo mặc định, database MySQL có thể truy cập tại `localhost` port 3306 và thông tin đăng nhập tương ứng với các giá trị của biến môi trường `DB_USERNAME` và `DB_PASSWORD` của bạn. Hoặc, bạn có thể kết nối với tư cách người dùng `root`, cũng sử dụng giá trị của biến môi trường `DB_PASSWORD` của bạn làm mật khẩu.

<a name="mongodb"></a>
### MongoDB

Nếu bạn chọn cài đặt service [MongoDB](https://www.mongodb.com/) khi cài đặt Sail, file `compose.yaml` của ứng dụng chứa một entry cho container [MongoDB Atlas Local](https://www.mongodb.com/docs/atlas/cli/current/atlas-cli-local-cloud/) cung cấp database document MongoDB với các tính năng Atlas như [Search Indexes](https://www.mongodb.com/docs/atlas/atlas-search/). Container này sử dụng một [Docker volume](https://docs.docker.com/storage/volumes/) để dữ liệu được lưu trữ trong database của bạn được duy trì ngay cả khi dừng và khởi động lại các containers của bạn.

Sau khi bạn đã khởi động các containers, bạn có thể kết nối với instance MongoDB trong ứng dụng bằng cách đặt biến môi trường `MONGODB_URI` trong file `.env` của ứng dụng thành `mongodb://mongodb:27017`. Authentication bị tắt theo mặc định, nhưng bạn có thể đặt các biến môi trường `MONGODB_USERNAME` và `MONGODB_PASSWORD` để bật authentication trước khi khởi động container `mongodb`. Sau đó, thêm thông tin đăng nhập vào connection string:

```ini
MONGODB_USERNAME=user
MONGODB_PASSWORD=laravel
MONGODB_URI=mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017
```

Để tích hợp liền mạch MongoDB với ứng dụng của bạn, bạn có thể cài đặt [package chính thức được duy trì bởi MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/).

Để kết nối với database MongoDB của ứng dụng từ máy local của bạn, bạn có thể sử dụng giao diện đồ họa như [Compass](https://www.mongodb.com/products/tools/compass). Theo mặc định, database MongoDB có thể truy cập tại `localhost` port `27017`.

<a name="redis"></a>
### Redis

File `compose.yaml` của ứng dụng cũng chứa một entry cho container [Redis](https://redis.io). Container này sử dụng một [Docker volume](https://docs.docker.com/storage/volumes/) để dữ liệu được lưu trữ trong instance Redis của bạn được duy trì ngay cả khi dừng và khởi động lại các containers của bạn. Sau khi bạn đã khởi động các containers, bạn có thể kết nối với instance Redis trong ứng dụng bằng cách đặt biến môi trường `REDIS_HOST` trong file `.env` của ứng dụng thành `redis`.

Để kết nối với database Redis của ứng dụng từ máy local của bạn, bạn có thể sử dụng một ứng dụng quản lý database đồ họa như [TablePlus](https://tableplus.com). Theo mặc định, database Redis có thể truy cập tại `localhost` port 6379.

<a name="valkey"></a>
### Valkey

Nếu bạn chọn cài đặt service Valkey khi cài đặt Sail, file `compose.yaml` của ứng dụng sẽ chứa một entry cho [Valkey](https://valkey.io/). Container này sử dụng một [Docker volume](https://docs.docker.com/storage/volumes/) để dữ liệu được lưu trữ trong instance Valkey của bạn được duy trì ngay cả khi dừng và khởi động lại các containers của bạn. Bạn có thể kết nối với container này trong ứng dụng bằng cách đặt biến môi trường `REDIS_HOST` trong file `.env` của ứng dụng thành `valkey`.

Để kết nối với database Valkey của ứng dụng từ máy local của bạn, bạn có thể sử dụng một ứng dụng quản lý database đồ họa như [TablePlus](https://tableplus.com). Theo mặc định, database Valkey có thể truy cập tại `localhost` port 6379.

<a name="meilisearch"></a>
### Meilisearch

Nếu bạn chọn cài đặt service [Meilisearch](https://www.meilisearch.com) khi cài đặt Sail, file `compose.yaml` của ứng dụng sẽ chứa một entry cho search engine mạnh mẽ này được tích hợp với [Laravel Scout](/docs/{{version}}/scout). Sau khi bạn đã khởi động các containers, bạn có thể kết nối với instance Meilisearch trong ứng dụng bằng cách đặt biến môi trường `MEILISEARCH_HOST` thành `http://meilisearch:7700`.

Từ máy local của bạn, bạn có thể truy cập panel quản lý dựa trên web của Meilisearch bằng cách điều hướng đến `http://localhost:7700` trong trình duyệt web của bạn.

<a name="typesense"></a>
### Typesense

Nếu bạn chọn cài đặt service [Typesense](https://typesense.org) khi cài đặt Sail, file `compose.yaml` của ứng dụng sẽ chứa một entry cho search engine mã nguồn mở cực nhanh này được tích hợp sẵn với [Laravel Scout](/docs/{{version}}/scout#typesense). Sau khi bạn đã khởi động các containers, bạn có thể kết nối với instance Typesense trong ứng dụng bằng cách đặt các biến môi trường sau:

```ini
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=xyz
```

Từ máy local của bạn, bạn có thể truy cập API của Typesense qua `http://localhost:8108`.

<a name="file-storage"></a>
## File Storage

Nếu bạn định sử dụng Amazon S3 để lưu trữ files trong khi chạy ứng dụng trong môi trường production, bạn có thể muốn cài đặt service [RustFS](https://rustfs.com) khi cài đặt Sail. RustFS cung cấp một API tương thích S3 mà bạn có thể sử dụng để phát triển local bằng driver lưu trữ file `s3` của Laravel mà không cần tạo "test" storage buckets trong môi trường S3 production của bạn. Nếu bạn chọn cài đặt RustFS trong khi cài đặt Sail, một phần cấu hình RustFS sẽ được thêm vào file `compose.yaml` của ứng dụng.

Theo mặc định, file cấu hình `filesystems` của ứng dụng đã chứa cấu hình disk cho disk `s3`. Ngoài việc sử dụng disk này để tương tác với Amazon S3, bạn có thể sử dụng nó để tương tác với bất kỳ service lưu trữ file tương thích S3 nào như RustFS bằng cách đơn giản sửa đổi các biến môi trường liên quan kiểm soát cấu hình của nó. Ví dụ, khi sử dụng RustFS, cấu hình biến môi trường filesystem của bạn nên được định nghĩa như sau:

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://rustfs:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

<a name="running-tests"></a>
## Chạy Tests

Laravel cung cấp hỗ trợ testing tuyệt vời out of the box, và bạn có thể sử dụng lệnh `test` của Sail để chạy [feature và unit tests](/docs/{{version}}/testing) của ứng dụng. Bất kỳ tùy chọn CLI nào được chấp nhận bởi Pest / PHPUnit cũng có thể được chuyển đến lệnh `test`:

```shell
sail test

sail test --group orders
```

Lệnh `test` của Sail tương đương với việc chạy lệnh Artisan `test`:

```shell
sail artisan test
```

Theo mặc định, Sail sẽ tạo một database `testing` chuyên dụng để các tests của bạn không can thiệp vào trạng thái hiện tại của database. Trong cài đặt Laravel mặc định, Sail cũng sẽ cấu hình file `phpunit.xml` của bạn để sử dụng database này khi thực thi các tests:

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) cung cấp một API tự động hóa và testing trình duyệt expressive, dễ sử dụng. Nhờ Sail, bạn có thể chạy các tests này mà không bao giờ cần cài đặt Selenium hoặc các công cụ khác trên máy tính local của bạn. Để bắt đầu, hãy bỏ comment service Selenium trong file `compose.yaml` của ứng dụng:

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

Tiếp theo, đảm bảo rằng service `laravel.test` trong file `compose.yaml` của ứng dụng có một entry `depends_on` cho `selenium`:

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

Cuối cùng, bạn có thể chạy bộ test Dusk của mình bằng cách khởi động Sail và chạy lệnh `dusk`:

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Selenium trên Apple Silicon

Nếu máy local của bạn chứa chip Apple Silicon, service `selenium` của bạn phải sử dụng image `selenium/standalone-chromium`:

```yaml
selenium:
    image: 'selenium/standalone-chromium'
    extra_hosts:
        - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## Preview Emails

File `compose.yaml` mặc định của Laravel Sail chứa một entry service cho [Mailpit](https://github.com/axllent/mailpit). Mailpit chặn các emails được gửi bởi ứng dụng của bạn trong quá trình phát triển local và cung cấp giao diện web tiện lợi để bạn có thể preview các tin nhắn email của mình trong trình duyệt. Khi sử dụng Sail, host mặc định của Mailpit là `mailpit` và có sẵn qua port 1025:

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Khi Sail đang chạy, bạn có thể truy cập giao diện web Mailpit tại: http://localhost:8025

<a name="sail-container-cli"></a>
## Container CLI

Đôi khi bạn có thể muốn bắt đầu một session Bash trong container ứng dụng của mình. Bạn có thể sử dụng lệnh `shell` để kết nối với container ứng dụng, cho phép bạn kiểm tra các files và services đã cài đặt cũng như thực thi các lệnh shell tùy ý trong container:

```shell
sail shell

sail root-shell
```

Để bắt đầu một session [Laravel Tinker](https://github.com/laravel/tinker) mới, bạn có thể thực thi lệnh `tinker`:

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## PHP Versions

Sail hiện hỗ trợ phục vụ ứng dụng của bạn qua PHP 8.5, 8.4, 8.3, 8.2, 8.1, hoặc PHP 8.0. Phiên bản PHP mặc định được sử dụng bởi Sail hiện là PHP 8.5. Để thay đổi phiên bản PHP được sử dụng để phục vụ ứng dụng của bạn, bạn nên cập nhật định nghĩa `build` của container `laravel.test` trong file `compose.yaml` của ứng dụng:

```yaml
# PHP 8.5
context: ./vendor/laravel/sail/runtimes/8.5

# PHP 8.4
context: ./vendor/laravel/sail/runtimes/8.4

# PHP 8.3
context: ./vendor/laravel/sail/runtimes/8.3

# PHP 8.2
context: ./vendor/laravel/sail/runtimes/8.2

# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0
```

Ngoài ra, bạn có thể muốn cập nhật tên `image` của mình để phản ánh phiên bản PHP đang được sử dụng bởi ứng dụng. Tùy chọn này cũng được định nghĩa trong file `compose.yaml` của ứng dụng:

```yaml
image: sail-8.2/app
```

Sau khi cập nhật file `compose.yaml` của ứng dụng, bạn nên rebuild các container images:

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Node Versions

Sail cài đặt Node 24 theo mặc định. Để thay đổi phiên bản Node được cài đặt khi build images, bạn có thể cập nhật định nghĩa `build.args` của service `laravel.test` trong file `compose.yaml` của ứng dụng:

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '18'
```

Sau khi cập nhật file `compose.yaml` của ứng dụng, bạn nên rebuild các container images:

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## Chia sẻ Site của bạn

Đôi khi bạn có thể cần chia sẻ site của mình công khai để preview site cho đồng nghiệp hoặc để test các tích hợp webhook với ứng dụng. Để chia sẻ site, bạn có thể sử dụng lệnh `share`. Sau khi thực thi lệnh này, bạn sẽ được cấp một URL `laravel-sail.site` ngẫu nhiên mà bạn có thể sử dụng để truy cập ứng dụng:

```shell
sail share
```

Khi chia sẻ site qua lệnh `share`, bạn nên cấu hình trusted proxies của ứng dụng bằng phương thức middleware `trustProxies` trong file `bootstrap/app.php` của ứng dụng. Nếu không, các helpers tạo URL như `url` và `route` sẽ không thể xác định HTTP host chính xác nên được sử dụng trong quá trình tạo URL:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: '*');
})
```

Nếu bạn muốn chọn subdomain cho site được chia sẻ, bạn có thể cung cấp tùy chọn `subdomain` khi thực thi lệnh `share`:

```shell
sail share --subdomain=my-sail-site
```

> [!NOTE]
> Lệnh `share` được hỗ trợ bởi [Expose](https://github.com/beyondcode/expose), một dịch vụ tunneling mã nguồn mở bởi [BeyondCode](https://beyondco.de).

<a name="debugging-with-xdebug"></a>
## Debugging với Xdebug

Cấu hình Docker của Laravel Sail bao gồm hỗ trợ cho [Xdebug](https://xdebug.org/), một debugger phổ biến và mạnh mẽ cho PHP. Để bật Xdebug, đảm bảo bạn đã [publish cấu hình Sail của mình](#sail-customization). Sau đó, thêm các biến sau vào file `.env` của ứng dụng để cấu hình Xdebug:

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

Tiếp theo, đảm bảo rằng file `php.ini` đã publish của bạn bao gồm cấu hình sau để Xdebug được kích hoạt trong các modes được chỉ định:

```ini
[xdebug]
xdebug.mode=${XDEBUG_MODE}
```

Sau khi sửa đổi file `php.ini`, hãy nhớ rebuild các Docker images để các thay đổi của bạn đối với file `php.ini` có hiệu lực:

```shell
sail build --no-cache
```

#### Cấu hình Linux Host IP

Nội bộ, biến môi trường `XDEBUG_CONFIG` được định nghĩa là `client_host=host.docker.internal` để Xdebug được cấu hình đúng cho Mac và Windows (WSL2). Nếu máy local của bạn đang chạy Linux và bạn đang sử dụng Docker 20.10+, `host.docker.internal` có sẵn và không cần cấu hình thủ công.

Đối với các phiên bản Docker cũ hơn 20.10, `host.docker.internal` không được hỗ trợ trên Linux và bạn sẽ cần định nghĩa thủ công host IP. Để làm điều này, cấu hình một IP tĩnh cho container bằng cách định nghĩa một mạng tùy chỉnh trong file `compose.yaml` của bạn:

```yaml
networks:
  custom_network:
    ipam:
      config:
        - subnet: 172.20.0.0/16

services:
  laravel.test:
    networks:
      custom_network:
        ipv4_address: 172.20.0.2
```

Sau khi bạn đã đặt IP tĩnh, định nghĩa biến SAIL_XDEBUG_CONFIG trong file .env của ứng dụng:

```ini
SAIL_XDEBUG_CONFIG="client_host=172.20.0.2"
```

<a name="xdebug-cli-usage"></a>
### Xdebug CLI Usage

Một lệnh `sail debug` có thể được sử dụng để bắt đầu một session debugging khi chạy một lệnh Artisan:

```shell
# Chạy một lệnh Artisan mà không có Xdebug...
sail artisan migrate

# Chạy một lệnh Artisan với Xdebug...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebug Browser Usage

Để debug ứng dụng trong khi tương tác với ứng dụng qua trình duyệt web, hãy làm theo [hướng dẫn được cung cấp bởi Xdebug](https://xdebug.org/docs/step_debug#web-application) để bắt đầu một session Xdebug từ trình duyệt web.

Nếu bạn đang sử dụng PhpStorm, hãy xem lại tài liệu của JetBrains về [zero-configuration debugging](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html).

> [!WARNING]
> Laravel Sail dựa vào `artisan serve` để phục vụ ứng dụng của bạn. Lệnh `artisan serve` chỉ chấp nhận các biến `XDEBUG_CONFIG` và `XDEBUG_MODE` từ phiên bản Laravel 8.53.0. Các phiên bản cũ hơn của Laravel (8.52.0 và dưới) không hỗ trợ các biến này và sẽ không chấp nhận các kết nối debug.

<a name="sail-customization"></a>
## Tùy chỉnh

Vì Sail chỉ là Docker, bạn có thể tùy chỉnh gần như mọi thứ về nó. Để publish các Dockerfiles của Sail, bạn có thể thực thi lệnh `sail:publish`:

```shell
sail artisan sail:publish
```

Sau khi chạy lệnh này, các Dockerfiles và các file cấu hình khác được sử dụng bởi Laravel Sail sẽ được đặt trong một thư mục `docker` trong thư mục root của ứng dụng. Sau khi tùy chỉnh cài đặt Sail của mình, bạn có thể muốn thay đổi tên image cho container ứng dụng trong file `compose.yaml` của ứng dụng. Sau khi làm như vậy, rebuild các containers của ứng dụng bằng lệnh `build`. Gán một tên duy nhất cho image ứng dụng đặc biệt quan trọng nếu bạn đang sử dụng Sail để phát triển nhiều ứng dụng Laravel trên một máy:

```shell
sail build --no-cache
```

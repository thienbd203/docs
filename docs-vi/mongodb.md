# MongoDB

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [MongoDB Driver](#mongodb-driver)
    - [Starting a MongoDB Server](#starting-a-mongodb-server)
    - [Install the Laravel MongoDB Package](#install-the-laravel-mongodb-package)
- [Cấu hình](#configuration)
- [Features](#features)

<a name="introduction"></a>
## Giới thiệu

[MongoDB](https://www.mongodb.com/resources/products/fundamentals/why-use-mongodb) là một trong những NoSQL document-oriented database phổ biến nhất, được sử dụng cho high write load (hữu ích cho analytics hoặc IoT) và high availability (dễ dàng set replica sets với automatic failover). Nó cũng có thể shard database dễ dàng cho horizontal scalability và có một powerful query language để thực hiện aggregation, text search hoặc geospatial queries.

Thay vì lưu trữ data trong tables của rows hoặc columns như SQL databases, mỗi record trong MongoDB database là một document được mô tả trong BSON, một binary representation của data. Applications có thể retrieve thông tin này trong JSON format. Nó hỗ trợ một wide variety của data types, bao gồm documents, arrays, embedded documents, và binary data.

Trước khi sử dụng MongoDB với Laravel, chúng tôi recommend cài đặt và sử dụng `mongodb/laravel-mongodb` package qua Composer. `laravel-mongodb` package được officially maintain bởi MongoDB, và mặc dù MongoDB được natively support bởi PHP qua MongoDB driver, [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) package cung cấp một richer integration với Eloquent và các Laravel features khác:

```shell
composer require mongodb/laravel-mongodb
```

<a name="installation"></a>
## Cài đặt

<a name="mongodb-driver"></a>
### MongoDB Driver

Để connect đến MongoDB database, `mongodb` PHP extension là bắt buộc. Nếu bạn đang phát triển local sử dụng [Laravel Herd](https://herd.laravel.com) hoặc đã cài đặt PHP qua `php.new`, bạn đã có extension này được cài đặt trên system của mình. Tuy nhiên, nếu bạn cần cài đặt extension thủ công, bạn có thể làm như vậy qua PECL:

```shell
pecl install mongodb
```

Để biết thêm thông tin về việc cài đặt MongoDB PHP extension, hãy xem [MongoDB PHP extension installation instructions](https://www.php.net/manual/en/mongodb.installation.php).

<a name="starting-a-mongodb-server"></a>
### Starting a MongoDB Server

MongoDB Community Server có thể được sử dụng để chạy MongoDB local và có sẵn để cài đặt trên Windows, macOS, Linux, hoặc như một Docker container. Để học cách cài đặt MongoDB, hãy tham khảo [official MongoDB Community installation guide](https://docs.mongodb.com/manual/administration/install-community/).

Connection string cho MongoDB server có thể được set trong file `.env` của bạn:

```ini
MONGODB_URI="mongodb://localhost:27017"
MONGODB_DATABASE="laravel_app"
```

Để hosting MongoDB trong cloud, hãy cân nhắc sử dụng [MongoDB Atlas](https://www.mongodb.com/cloud/atlas).
Để access MongoDB Atlas cluster local từ application của bạn, bạn sẽ cần [add IP address của chính bạn trong cluster's network settings](https://www.mongodb.com/docs/atlas/security/add-ip-address-to-list/) vào project's IP Access List.

Connection string cho MongoDB Atlas cũng có thể được set trong file `.env` của bạn:

```ini
MONGODB_URI="mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority"
MONGODB_DATABASE="laravel_app"
```

<a name="install-the-laravel-mongodb-package"></a>
### Install the Laravel MongoDB Package

Cuối cùng, sử dụng Composer để cài đặt Laravel MongoDB package:

```shell
composer require mongodb/laravel-mongodb
```

> [!NOTE]
> Installation của package này sẽ fail nếu `mongodb` PHP extension không được cài đặt. PHP configuration có thể khác nhau giữa CLI và web server, vì vậy hãy đảm bảo extension được enabled trong cả hai configurations.

<a name="configuration"></a>
## Cấu hình

Bạn có thể configure MongoDB connection của bạn qua file configuration `config/database.php` của application. Trong file này, thêm một `mongodb` connection sử dụng `mongodb` driver:

```php
'connections' => [
    'mongodb' => [
        'driver' => 'mongodb',
        'dsn' => env('MONGODB_URI', 'mongodb://localhost:27017'),
        'database' => env('MONGODB_DATABASE', 'laravel_app'),
    ],
],
```

<a name="features"></a>
## Features

Sau khi configuration của bạn hoàn thành, bạn có thể sử dụng `mongodb` package và database connection trong application của bạn để leverage một variety của powerful features:

- [Using Eloquent](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/eloquent-models/), models có thể được lưu trữ trong MongoDB collections. Ngoài các standard Eloquent features, Laravel MongoDB package cung cấp additional features như embedded relationships. Package cũng cung cấp direct access đến MongoDB driver, có thể được sử dụng để execute operations như raw queries và aggregation pipelines.
- [Write complex queries](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/query-builder/) sử dụng query builder.
- `mongodb` [cache driver](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/) được optimized để sử dụng MongoDB features như TTL indexes để tự động clear expired cache entries.
- [Dispatch và process queued jobs](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/) với `mongodb` queue driver.
- [Storing files trong GridFS](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/filesystems/), qua [GridFS Adapter cho Flysystem](https://flysystem.thephpleague.com/docs/adapter/gridfs/).
- Hầu hết third party packages sử dụng database connection hoặc Eloquent có thể được sử dụng với MongoDB.

Để tiếp tục học cách sử dụng MongoDB và Laravel, hãy tham khảo [Quick Start guide](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/quick-start/) của MongoDB.

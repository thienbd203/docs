# Laravel Scout

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Queueing](#queueing)
- [Driver Prerequisites](#driver-prerequisites)
- [Cấu hình](#configuration)
    - [Cấu hình Searchable Data](#configuring-searchable-data)
- [Database / Collection Engines](#database-and-collection-engines)
    - [Database Engine](#database-engine)
    - [Collection Engine](#collection-engine)
- [Cấu hình Engine Bên thứ Ba](#third-party-engine-configuration)
    - [Cấu hình Model Indexes](#configuring-model-indexes)
    - [Algolia](#algolia-configuration)
    - [Meilisearch](#meilisearch-configuration)
    - [Typesense](#typesense-configuration)
- [Indexing Engine Bên thứ Ba](#indexing)
    - [Batch Import](#batch-import)
    - [Thêm Records](#adding-records)
    - [Cập nhật Records](#updating-records)
    - [Xóa Records](#removing-records)
    - [Tạm dừng Indexing](#pausing-indexing)
    - [Model Instances Có Điều kiện Searchable](#conditionally-searchable-model-instances)
- [Searching](#searching)
    - [Where Clauses](#where-clauses)
    - [Pagination](#pagination)
    - [Soft Deleting](#soft-deleting)
    - [Tùy chỉnh Engine Searches](#customizing-engine-searches)
- [Custom Engines](#custom-engines)

<a name="introduction"></a>
## Giới thiệu

[Laravel Scout](https://github.com/laravel/scout) cung cấp một giải pháp dựa trên driver đơn giản để thêm full-text search vào [Eloquent models](/docs/{{version}}/eloquent) của bạn. Sử dụng model observers, Scout sẽ tự động giữ các search indexes của bạn đồng bộ với các Eloquent records của bạn.

Scout đi kèm với một `database` engine tích hợp sử dụng MySQL / PostgreSQL full-text indexes và các mệnh đề `LIKE` để tìm kiếm database hiện có của bạn — không cần dịch vụ bên ngoài. Đối với hầu hết các ứng dụng, đây là tất cả những gì bạn cần. Để biết tổng quan về tất cả các tùy chọn tìm kiếm có sẵn trong Laravel, hãy tham khảo [tài liệu tìm kiếm](/docs/{{version}}/search).

Scout cũng bao gồm các drivers cho [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com), và [Typesense](https://typesense.org) khi bạn cần các tính năng như typo tolerance, faceted filtering, hoặc geo-search ở quy mô lớn. Một driver "collection" cũng có sẵn cho phát triển cục bộ, và bạn có thể tự do viết [custom engines](#custom-engines) cũng như.

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt Scout thông qua Composer package manager:

```shell
composer require laravel/scout
```

Sau khi cài đặt Scout, bạn nên publish file cấu hình Scout sử dụng command `vendor:publish` của Artisan. Command này sẽ publish file cấu hình `scout.php` vào thư mục `config` của ứng dụng của bạn:

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

Cuối cùng, thêm trait `Laravel\Scout\Searchable` vào model bạn muốn làm searchable. Trait này sẽ đăng ký một model observer sẽ tự động giữ model đồng bộ với search driver của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

<a name="queueing"></a>
### Queueing

Khi sử dụng một engine không phải là `database` hoặc `collection` engine, bạn nên cân nhắc mạnh việc cấu hình một [queue driver](/docs/{{version}}/queues) trước khi sử dụng thư viện. Chạy một queue worker sẽ cho phép Scout queue tất cả các hoạt động đồng bộ thông tin model của bạn với các search indexes của bạn, cung cấp thời gian phản hồi tốt hơn nhiều cho giao diện web của ứng dụng của bạn.

Sau khi bạn đã cấu hình một queue driver, đặt giá trị của tùy chọn `queue` trong file cấu hình `config/scout.php` của bạn thành `true`:

```php
'queue' => true,
```

Ngay cả khi tùy chọn `queue` được đặt thành `false`, điều quan trọng cần nhớ là một số Scout drivers như Algolia và Meilisearch luôn index records bất đồng bộ. Nói cách khác, ngay cả khi hoạt động index đã hoàn thành trong ứng dụng Laravel của bạn, search engine本身 có thể không phản ánh các records mới và được cập nhật ngay lập tức.

Để chỉ định kết nối và queue mà các Scout jobs của bạn sử dụng, bạn có thể định nghĩa tùy chọn cấu hình `queue` như một mảng:

```php
'queue' => [
    'connection' => 'redis',
    'queue' => 'scout'
],
```

Tất nhiên, nếu bạn tùy chỉnh kết nối và queue mà các Scout jobs của bạn sử dụng, bạn nên chạy một queue worker để xử lý các jobs trên kết nối và queue đó:

```shell
php artisan queue:work redis --queue=scout
```

<a name="unique-jobs"></a>
#### Unique Jobs

Trong các ứng dụng có nhiều ghi, bạn có thể muốn ngăn Scout queue các jobs trùng lặp cho cùng các model records. Bạn có thể chọn vào unique indexing jobs bằng cách đăng ký các class job `MakeSearchableUniquely` và `RemoveFromSearchUniquely`, thường là trong phương thức `boot` của một service provider:

```php
use Laravel\Scout\Jobs\MakeSearchableUniquely;
use Laravel\Scout\Jobs\RemoveFromSearchUniquely;
use Laravel\Scout\Scout;

Scout::makeSearchableUsing(MakeSearchableUniquely::class);
Scout::removeFromSearchUsing(RemoveFromSearchUniquely::class);
```

Các jobs này sử dụng [unique job locks](/docs/{{version}}/queues#unique-jobs) của Laravel để tránh dispatch các hoạt động indexing queued trùng lặp cho cùng các model records searchable trong khi một job phù hợp đã được queued.

<a name="driver-prerequisites"></a>
## Driver Prerequisites

<a name="algolia"></a>
### Algolia

Khi sử dụng Algolia driver, bạn nên cấu hình thông tin xác thực Algolia `id` và `secret` của bạn trong file cấu hình `config/scout.php` của bạn. Sau khi thông tin xác thực của bạn đã được cấu hình, bạn cũng sẽ cần cài đặt Algolia PHP SDK thông qua Composer package manager:

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
### Meilisearch

[Meilisearch](https://www.meilisearch.com) là một search engine mã nguồn mở nhanh. Nếu bạn không chắc cách cài đặt Meilisearch trên máy cục bộ của mình, bạn có thể sử dụng [Laravel Sail](/docs/{{version}}/sail#meilisearch), môi trường phát triển Docker được hỗ trợ chính thức của Laravel.

Khi sử dụng Meilisearch driver, bạn sẽ cần cài đặt Meilisearch PHP SDK thông qua Composer package manager:

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

Sau đó, đặt environment variable `SCOUT_DRIVER` cũng như thông tin xác thực Meilisearch `host` và `key` của bạn trong file `.env` của ứng dụng của bạn:

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

Để biết thêm thông tin về Meilisearch, hãy tham khảo [tài liệu Meilisearch](https://docs.meilisearch.com/learn/getting_started/quick_start.html).

Ngoài ra, bạn nên đảm bảo rằng bạn cài đặt một phiên bản của `meilisearch/meilisearch-php` tương thích với phiên bản Meilisearch binary của bạn bằng cách xem xét [tài liệu Meilisearch về tính tương thích binary](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch).

> [!WARNING]
> Khi nâng cấp Scout trên một ứng dụng sử dụng Meilisearch, bạn nên luôn [xem xét bất kỳ breaking changes bổ sung nào](https://github.com/meilisearch/Meilisearch/releases) đến dịch vụ Meilisearch本身.

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) là một search engine mã nguồn mở cực nhanh và hỗ trợ keyword search, semantic search, geo search, và vector search.

Bạn có thể [self-host](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) Typesense hoặc sử dụng [Typesense Cloud](https://cloud.typesense.org).

Để bắt đầu sử dụng Typesense với Scout, cài đặt Typesense PHP SDK thông qua Composer package manager:

```shell
composer require typesense/typesense-php
```

Sau đó, đặt environment variable `SCOUT_DRIVER` cũng như thông tin xác thực host và API key Typesense của bạn trong file .env của ứng dụng của bạn:

```ini
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

Nếu bạn đang sử dụng [Laravel Sail](/docs/{{version}}/sail), bạn có thể cần điều chỉnh environment variable `TYPESENSE_HOST` để khớp với tên Docker container. Bạn cũng có thể tùy chọn chỉ định port, path, và protocol của cài đặt của bạn:

```ini
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

Các cài đặt bổ sung và định nghĩa schema cho các Typesense collections của bạn có thể được tìm thấy trong file cấu hình `config/scout.php` của ứng dụng của bạn. Để biết thêm thông tin về Typesense, hãy tham khảo [tài liệu Typesense](https://typesense.org/docs/guide/#quick-start).

<a name="configuration"></a>
## Cấu hình

<a name="configuring-searchable-data"></a>
### Cấu hình Searchable Data

Theo mặc định, toàn bộ form `toArray` của một model nhất định sẽ được lưu vào search index của nó. Nếu bạn muốn tùy chỉnh dữ liệu được đồng bộ hóa với search index, bạn có thể ghi đè phương thức `toSearchableArray` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Get the indexable data array for the model.
     *
     * @return array<string, mixed>
     */
    public function toSearchableArray(): array
    {
        $array = $this->toArray();

        // Customize the data array...

        return $array;
    }
}
```

<a name="configuring-search-engines-per-model"></a>
#### Cấu hình Model Engines

Khi tìm kiếm, Scout thường sẽ sử dụng search engine mặc định được chỉ định trong file cấu hình `scout` của ứng dụng của bạn. Tuy nhiên, search engine cho một model cụ thể có thể được thay đổi bằng cách ghi đè phương thức `searchableUsing` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Engines\Engine;
use Laravel\Scout\Scout;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the engine used to index the model.
     */
    public function searchableUsing(): Engine
    {
        return Scout::engine('meilisearch');
    }
}
```

<a name="database-and-collection-engines"></a>
## Database / Collection Engines

<a name="database-engine"></a>
### Database Engine

> [!WARNING]
> Database engine hiện tại hỗ trợ MySQL và PostgreSQL, cả hai đều cung cấp hỗ trợ cho full-text column indexing nhanh.

`database` engine sử dụng MySQL / PostgreSQL full-text indexes và các mệnh đề `LIKE` để tìm kiếm database hiện có của bạn trực tiếp. Đối với nhiều ứng dụng, đây là cách đơn giản và thực tế nhất để thêm tìm kiếm — không cần dịch vụ bên ngoài hoặc cơ sở hạ tầng bổ sung.

Để sử dụng database engine, đặt environment variable `SCOUT_DRIVER` thành `database`:

```ini
SCOUT_DRIVER=database
```

Sau khi cấu hình, bạn có thể [định nghĩa searchable data của bạn](#configuring-searchable-data) và bắt đầu [thực thi các search queries](#searching) đối với các models của bạn. Không giống như các engines bên thứ ba, database engine không yêu cầu bước indexing riêng biệt — nó tìm kiếm các bảng database của bạn trực tiếp.

#### Tùy chỉnh Database Searching Strategies

Theo mặc định, database engine sẽ thực thi một query `LIKE` đối với mọi model attribute mà bạn đã [cấu hình là searchable](#configuring-searchable-data). Tuy nhiên, bạn có thể gán các chiến lược tìm kiếm hiệu quả hơn cho các cột cụ thể. Attribute `SearchUsingFullText` sẽ sử dụng full-text index của database cho cột đó, trong khi `SearchUsingPrefix` sẽ chỉ khớp phần đầu của chuỗi (`example%`) thay vì tìm kiếm trong toàn bộ chuỗi (`%example%`).

Để định nghĩa hành vi này, gán các PHP attributes vào phương thức `toSearchableArray` của model. Bất kỳ cột nào không có attribute sẽ tiếp tục sử dụng chiến lược `LIKE` mặc định:

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> [!WARNING]
> Trước khi chỉ định rằng một cột nên sử dụng các ràng buộc full text query, hãy đảm bảo rằng cột đã được gán một [full text index](/docs/{{version}}/migrations#available-index-types).

<a name="collection-engine"></a>
### Collection Engine

"collection" engine được dự định cho các nguyên mẫu nhanh, các bộ dữ liệu cực nhỏ (vài trăm records), hoặc chạy tests. Nó truy xuất tất cả các records có thể từ database của bạn và sử dụng helper `Str::is` của Laravel để lọc chúng trong PHP, vì vậy nó không yêu cầu bất kỳ indexing hoặc tính năng cụ thể của database nào. Đối với bất kỳ điều gì ngoài các trường hợp sử dụng tầm thường, bạn nên sử dụng [database engine](#database-engine) thay thế.

Để sử dụng collection engine, bạn có thể đơn giản đặt giá trị của environment variable `SCOUT_DRIVER` thành `collection`, hoặc chỉ định driver `collection` trực tiếp trong file cấu hình `scout` của ứng dụng của bạn:

```ini
SCOUT_DRIVER=collection
```

Sau khi bạn đã chỉ định collection driver làm driver ưa thích của mình, bạn có thể bắt đầu [thực thi các search queries](#searching) đối với các models của bạn. Indexing search engine, chẳng hạn như indexing cần thiết để seed Algolia, Meilisearch, hoặc Typesense indexes, là không cần thiết khi sử dụng collection engine.

#### Sự Khác Biệt Từ Database Engine

Trong khi database engine sử dụng full-text indexes và các mệnh đề `LIKE` để tìm các records phù hợp hiệu quả, collection engine kéo tất cả các records và lọc chúng trong PHP. Collection engine là tùy chọn di động nhất vì nó hoạt động trên tất cả các relational databases được hỗ trợ bởi Laravel (bao gồm SQLite và SQL Server); tuy nhiên, nó kém hiệu quả hơn đáng kể so với database engine và không nên được sử dụng với các bộ dữ liệu lớn.

<a name="third-party-engine-configuration"></a>
## Cấu hình Engine Bên thứ Ba

Các tùy chọn cấu hình sau chỉ có liên quan khi sử dụng một search engine bên thứ ba như Algolia, Meilisearch, hoặc Typesense. Nếu bạn đang sử dụng [database engine](#database-engine), bạn có thể bỏ qua phần này.

<a name="configuring-model-indexes"></a>
### Cấu hình Model Indexes

Khi sử dụng một engine bên thứ ba, mỗi Eloquent model được đồng bộ với một search "index" nhất định, chứa tất cả các searchable records cho model đó. Theo mặc định, mỗi model sẽ được lưu vào một index khớp với tên "table" điển hình của model. Thông thường, đây là dạng số nhiều của tên model; tuy nhiên, bạn có thể tự do tùy chỉnh index của model bằng cách ghi đè phương thức `searchableAs` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Get the name of the index associated with the model.
     */
    public function searchableAs(): string
    {
        return 'posts_index';
    }
}
```

> [!NOTE]
> Phương thức `searchableAs` không có hiệu lực khi sử dụng database engine, luôn tìm kiếm bảng database của model trực tiếp.

<a name="configuring-the-model-id"></a>
#### Cấu hình Model ID

Theo mặc định, Scout sẽ sử dụng primary key của model làm unique ID / key của model được lưu trong search index. Nếu bạn cần tùy chỉnh hành vi này khi sử dụng một engine bên thứ ba, bạn có thể ghi đè các phương thức `getScoutKey` và `getScoutKeyName` trên model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the value used to index the model.
     */
    public function getScoutKey(): mixed
    {
        return $this->email;
    }

    /**
     * Get the key name used to index the model.
     */
    public function getScoutKeyName(): mixed
    {
        return 'email';
    }
}
```

> [!NOTE]
> Các phương thức `getScoutKey` và `getScoutKeyName` không có hiệu lực khi sử dụng database engine, luôn sử dụng primary key của model.

<a name="algolia-configuration"></a>
### Algolia

<a name="algolia-index-settings"></a>
#### Index Settings

Đôi khi bạn có thể muốn cấu hình các cài đặt bổ sung trên các Algolia indexes của bạn. Trong khi bạn có thể quản lý các cài đặt này thông qua Algolia UI, đôi khi hiệu quả hơn là quản lý trạng thái mong muốn của cấu hình index của bạn trực tiếp từ file cấu hình `config/scout.php` của ứng dụng của bạn.

Cách tiếp cận này cho phép bạn triển khai các cài đặt này thông qua pipeline triển khai tự động của ứng dụng của bạn, tránh cấu hình thủ công và đảm bảo tính nhất quán trên nhiều môi trường. Bạn có thể cấu hình các thuộc tính có thể lọc, xếp hạng, faceting, hoặc [bất kỳ cài đặt được hỗ trợ nào khác](https://www.algolia.com/doc/rest-api/search/#tag/Indices/operation/setSettings).

Để bắt đầu, thêm cài đặt cho mỗi index trong file cấu hình `config/scout.php` của ứng dụng của bạn:

```php
use App\Models\User;
use App\Models\Flight;

'algolia' => [
    'id' => env('ALGOLIA_APP_ID', ''),
    'secret' => env('ALGOLIA_SECRET', ''),
    'index-settings' => [
        User::class => [
            'searchableAttributes' => ['id', 'name', 'email'],
            'attributesForFaceting'=> ['filterOnly(email)'],
            // Other settings fields...
        ],
        Flight::class => [
            'searchableAttributes'=> ['id', 'destination'],
        ],
    ],
],
```

Nếu model cơ bản của một index nhất định có thể soft delete và được bao gồm trong mảng `index-settings`, Scout sẽ tự động bao gồm hỗ trợ cho faceting trên các soft deleted models trên index đó. Nếu bạn không có các thuộc tính faceting nào khác để định nghĩa cho một index model soft deletable, bạn có thể đơn giản thêm một mục trống vào mảng `index-settings` cho model đó:

```php
'index-settings' => [
    Flight::class => []
],
```

Sau khi cấu hình các cài đặt index của ứng dụng của bạn, bạn phải gọi command `scout:sync-index-settings` của Artisan. Command này sẽ thông báo cho Algolia về các cài đặt index hiện được cấu hình của bạn. Để thuận tiện, bạn có thể muốn làm cho command này trở thành một phần của quy trình triển khai của bạn:

```shell
php artisan scout:sync-index-settings
```

<a name="algolia-identifying-users"></a>
#### Identifying Users

Scout cho phép bạn tự động xác định người dùng khi sử dụng Algolia. Liên kết người dùng được xác thực với các hoạt động tìm kiếm có thể hữu ích khi xem analytics tìm kiếm của bạn trong dashboard của Algolia. Bạn có thể bật xác định người dùng bằng cách định nghĩa một environment variable `SCOUT_IDENTIFY` là `true` trong file `.env` của ứng dụng của bạn:

```ini
SCOUT_IDENTIFY=true
```

Bật tính năng này cũng sẽ chuyển địa chỉ IP của request và định danh chính của người dùng được xác thực của bạn đến Algolia để dữ liệu này được liên kết với bất kỳ search request nào được thực hiện bởi người dùng.

<a name="meilisearch-configuration"></a>
### Meilisearch

<a name="meilisearch-index-settings"></a>
#### Index Settings

Meilisearch yêu cầu bạn định nghĩa trước các cài đặt tìm kiếm index như các thuộc tính có thể lọc, các thuộc tính có thể sắp xếp, và [các trường cài đặt được hỗ trợ khác](https://docs.meilisearch.com/reference/api/settings.html).

Các thuộc tính có thể lọc là bất kỳ thuộc tính nào bạn dự định lọc khi gọi phương thức `where` của Scout, trong khi các thuộc tính có thể sắp xếp là bất kỳ thuộc tính nào bạn dự định sắp xếp khi gọi phương thức `orderBy` của Scout. Để định nghĩa các cài đặt index của bạn, điều chỉnh phần `index-settings` của mục cấu hình `meilisearch` trong file cấu hình `scout` của ứng dụng của bạn:

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // Other settings fields...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```

Nếu model cơ bản của một index nhất định có thể soft delete và được bao gồm trong mảng `index-settings`, Scout sẽ tự động bao gồm hỗ trợ cho lọc trên các soft deleted models trên index đó. Nếu bạn không có các thuộc tính có thể lọc hoặc có thể sắp xếp nào khác để định nghĩa cho một index model soft deletable, bạn có thể đơn giản thêm một mục trống vào mảng `index-settings` cho model đó:

```php
'index-settings' => [
    Flight::class => []
],
```

Sau khi cấu hình các cài đặt index của ứng dụng của bạn, bạn phải gọi command `scout:sync-index-settings` của Artisan. Command này sẽ thông báo cho Meilisearch về các cài đặt index hiện được cấu hình của bạn. Để thuận tiện, bạn có thể muốn làm cho command này trở thành một phần của quy trình triển khai của bạn:

```shell
php artisan scout:sync-index-settings
```

<a name="meilisearch-data-types"></a>
#### Searchable Data Types

Meilisearch sẽ chỉ thực hiện các hoạt động lọc (`>`, `<`, v.v.) trên dữ liệu của loại đúng. Khi tùy chỉnh searchable data của bạn, bạn nên đảm bảo rằng các giá trị số được cast đến loại đúng của chúng:

```php
public function toSearchableArray()
{
    return [
        'id' => (int) $this->id,
        'name' => $this->name,
        'price' => (float) $this->price,
    ];
}
```

<a name="typesense-configuration"></a>
### Typesense

<a name="typesense-searchable-data"></a>
#### Chuẩn bị Searchable Data

Khi sử dụng Typesense, các searchable models của bạn phải định nghĩa một phương thức `toSearchableArray` cast primary key của model thành một chuỗi và ngày tạo thành một UNIX timestamp:

```php
/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
public function toSearchableArray(): array
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

Bạn cũng nên định nghĩa các schema Typesense collections của bạn trong file `config/scout.php` của ứng dụng của bạn. Một schema collection mô tả các loại dữ liệu của mỗi trường có thể tìm kiếm qua Typesense. Để biết thêm thông tin về tất cả các tùy chọn schema có sẵn, hãy tham khảo [tài liệu Typesense](https://typesense.org/docs/latest/api/collections.html#schema-parameters).

Nếu bạn cần thay đổi schema Typesense collection của bạn sau khi nó đã được định nghĩa, bạn có thể chạy `scout:flush` và `scout:import`, sẽ xóa tất cả dữ liệu được index hiện có và tạo lại schema. Hoặc, bạn có thể sử dụng API của Typesense để sửa đổi schema của collection mà không xóa bất kỳ dữ liệu được index nào.

Nếu searchable model của bạn có thể soft delete, bạn nên định nghĩa một trường `__soft_deleted` trong schema Typesense tương ứng của model trong file cấu hình `config/scout.php` của ứng dụng của bạn:

```php
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```

<a name="typesense-dynamic-search-parameters"></a>
#### Dynamic Search Parameters

Typesense cho phép bạn sửa đổi các [search parameters](https://typesense.org/docs/latest/api/search.html#search-parameters) của bạn động khi thực hiện một hoạt động tìm kiếm thông qua phương thức `options`:

```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

<a name="indexing"></a>
## Indexing Engine Bên thứ Ba

> [!NOTE]
> Các tính năng indexing được mô tả trong phần này chủ yếu có liên quan khi sử dụng một engine bên thứ ba (Algolia, Meilisearch, hoặc Typesense). Database engine tìm kiếm các bảng database của bạn trực tiếp, vì vậy nó không yêu cầu quản lý index thủ công.

<a name="batch-import"></a>
### Batch Import

Nếu bạn đang cài đặt Scout vào một dự án hiện có, bạn có thể đã có các database records cần import vào các indexes của bạn. Scout cung cấp một command `scout:import` của Artisan mà bạn có thể sử dụng để import tất cả các records hiện có của bạn vào các search indexes của bạn:

```shell
php artisan scout:import "App\Models\Post"
```

Command `scout:queue-import` có thể được sử dụng để import tất cả các records hiện có của bạn sử dụng [queued jobs](/docs/{{version}}/queues):

```shell
php artisan scout:queue-import "App\Models\Post" --chunk=500
```

Command `flush` có thể được sử dụng để xóa tất cả các records của một model khỏi các search indexes của bạn:

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### Sửa đổi Import Query

Nếu bạn muốn sửa đổi query được sử dụng để truy xuất tất cả các models của bạn cho batch importing, bạn có thể định nghĩa một phương thức `makeAllSearchableUsing` trên model của bạn. Đây là một nơi tuyệt vời để thêm bất kỳ eager relationship loading nào có thể cần thiết trước khi import các models của bạn:

```php
use Illuminate\Database\Eloquent\Builder;

/**
 * Modify the query used to retrieve models when making all of the models searchable.
 */
protected function makeAllSearchableUsing(Builder $query): Builder
{
    return $query->with('author');
}
```

> [!WARNING]
> Phương thức `makeAllSearchableUsing` có thể không áp dụng khi sử dụng một queue để batch import các models. Các relationships [không được khôi phục](/docs/{{version}}/queues#handling-relationships) khi các bộ sưu tập model được xử lý bởi các jobs.

<a name="adding-records"></a>
### Thêm Records

Sau khi bạn đã thêm trait `Laravel\Scout\Searchable` vào một model, tất cả những gì bạn cần làm là `save` hoặc `create` một instance model và nó sẽ tự động được thêm vào search index của bạn. Nếu bạn đã cấu hình Scout để [sử dụng queues](#queueing) hoạt động này sẽ được thực hiện trong nền bởi queue worker của bạn:

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

<a name="adding-records-via-query"></a>
#### Thêm Records qua Query

Nếu bạn muốn thêm một bộ sưu tập các models vào search index của bạn thông qua một Eloquent query, bạn có thể chain phương thức `searchable` vào Eloquent query. Phương thức `searchable` sẽ [chunk các kết quả](/docs/{{version}}/eloquent#chunking-results) của query và thêm các records vào search index của bạn. Một lần nữa, nếu bạn đã cấu hình Scout để sử dụng queues, tất cả các chunks sẽ được import trong nền bởi các queue workers của bạn:

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

Bạn cũng có thể gọi phương thức `searchable` trên một instance Eloquent relationship:

```php
$user->orders()->searchable();
```

Hoặc, nếu bạn đã có một bộ sưu tập các Eloquent models trong memory, bạn có thể gọi phương thức `searchable` trên instance bộ sưu tập để thêm các instances model vào index tương ứng của chúng:

```php
$orders->searchable();
```

> [!NOTE]
> Phương thức `searchable` có thể được coi là một hoạt động "upsert". Nói cách khác, nếu model record đã có trong index của bạn, nó sẽ được cập nhật. Nếu nó không tồn tại trong search index, nó sẽ được thêm vào index.

<a name="updating-records"></a>
### Cập nhật Records

Để cập nhật một model searchable, bạn chỉ cần cập nhật các thuộc tính của instance model và `save` model vào database của bạn. Scout sẽ tự động lưu các thay đổi vào search index của bạn:

```php
use App\Models\Order;

$order = Order::find(1);

// Update the order...

$order->save();
```

Bạn cũng có thể gọi phương thức `searchable` trên một instance Eloquent query để cập nhật một bộ sưu tập các models. Nếu các models không tồn tại trong search index của bạn, chúng sẽ được tạo:

```php
Order::where('price', '>', 100)->searchable();
```

Nếu bạn muốn cập nhật các records search index cho tất cả các models trong một relationship, bạn có thể gọi `searchable` trên instance relationship:

```php
$user->orders()->searchable();
```

Hoặc, nếu bạn đã có một bộ sưu tập các Eloquent models trong memory, bạn có thể gọi phương thức `searchable` trên instance bộ sưu tập để cập nhật các instances model trong index tương ứng của chúng:

```php
$orders->searchable();
```

<a name="modifying-records-before-importing"></a>
#### Sửa đổi Records Trước khi Import

Đôi khi bạn có thể cần chuẩn bị bộ sưu tập các models trước khi chúng được làm searchable. Ví dụ, bạn có thể muốn eager load một relationship để dữ liệu relationship có thể được thêm hiệu quả vào search index của bạn. Để thực hiện điều này, định nghĩa một phương thức `makeSearchableUsing` trên model tương ứng:

```php
use Illuminate\Database\Eloquent\Collection;

/**
 * Modify the collection of models being made searchable.
 */
public function makeSearchableUsing(Collection $models): Collection
{
    return $models->load('author');
}
```

<a name="conditionally-updating-the-search-index"></a>
#### Cập nhật Search Index Có Điều kiện

Theo mặc định, Scout sẽ reindex một model được cập nhật bất kể các thuộc tính nào được sửa đổi. Nếu bạn muốn tùy chỉnh hành vi này, bạn có thể định nghĩa một phương thức `searchIndexShouldBeUpdated` trên model của bạn:

```php
/**
 * Determine if the search index should be updated.
 */
public function searchIndexShouldBeUpdated(): bool
{
    return $this->wasRecentlyCreated || $this->wasChanged(['title', 'body']);
}
```

<a name="removing-records"></a>
### Xóa Records

Để xóa một record khỏi index của bạn, bạn có thể đơn giản `delete` model khỏi database. Điều này có thể được thực hiện ngay cả khi bạn đang sử dụng các models [soft deleted](/docs/{{version}}/eloquent#soft-deleting):

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

Nếu bạn không muốn truy xuất model trước khi xóa record, bạn có thể sử dụng phương thức `unsearchable` trên một instance Eloquent query:

```php
Order::where('price', '>', 100)->unsearchable();
```

Nếu bạn muốn xóa các records search index cho tất cả các models trong một relationship, bạn có thể gọi `unsearchable` trên instance relationship:

```php
$user->orders()->unsearchable();
```

Hoặc, nếu bạn đã có một bộ sưu tập các Eloquent models trong memory, bạn có thể gọi phương thức `unsearchable` trên instance bộ sưu tập để xóa các instances model khỏi index tương ứng của chúng:

```php
$orders->unsearchable();
```

Để xóa tất cả các model records khỏi index tương ứng của chúng, bạn có thể gọi phương thức `removeAllFromSearch`:

```php
Order::removeAllFromSearch();
```

<a name="pausing-indexing"></a>
### Tạm dừng Indexing

Đôi khi bạn có thể cần thực hiện một loạt các hoạt động Eloquent trên một model mà không đồng bộ hóa dữ liệu model với search index của bạn. Bạn có thể làm điều này sử dụng phương thức `withoutSyncingToSearch`. Phương thức này chấp nhận một closure duy nhất sẽ được thực thi ngay lập tức. Bất kỳ hoạt động model nào xảy ra trong closure sẽ không được đồng bộ hóa với index của model:

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // Perform model actions...
});
```

<a name="conditionally-searchable-model-instances"></a>
### Model Instances Có Điều kiện Searchable

Đôi khi bạn có thể cần chỉ làm cho một model searchable trong các điều kiện nhất định. Ví dụ, hãy tưởng tượng bạn có model `App\Models\Post` có thể ở một trong hai trạng thái: "draft" và "published". Bạn có thể chỉ muốn cho phép các posts "published" có thể tìm kiếm. Để thực hiện điều này, bạn có thể định nghĩa một phương thức `shouldBeSearchable` trên model của bạn:

```php
/**
 * Determine if the model should be searchable.
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

Phương thức `shouldBeSearchable` chỉ được áp dụng khi thao tác các models thông qua các phương thức `save` và `create`, queries, hoặc relationships. Trực tiếp làm cho các models hoặc bộ sưu tập searchable sử dụng phương thức `searchable` sẽ ghi đè kết quả của phương thức `shouldBeSearchable`.

> [!WARNING]
> Phương thức `shouldBeSearchable` không áp dụng khi sử dụng "database" engine của Scout, vì tất cả searchable data luôn được lưu trữ trong database. Để đạt được hành vi tương tự khi sử dụng database engine, bạn nên sử dụng [where clauses](#where-clauses) thay thế.

<a name="searching"></a>
## Searching

Bạn có thể bắt đầu tìm kiếm một model sử dụng phương thức `search`. Phương thức search chấp nhận một chuỗi duy nhất sẽ được sử dụng để tìm kiếm các models của bạn. Sau đó, bạn nên chain phương thức `get` vào search query để truy xuất các Eloquent models khớp với search query đã cho:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

Vì các kết quả tìm kiếm của Scout trả về một bộ sưu tập các Eloquent models, bạn thậm chí có thể trả về kết quả trực tiếp từ một route hoặc controller và chúng sẽ tự động được chuyển đổi thành JSON:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

Nếu bạn muốn lấy các kết quả tìm kiếm thô trước khi chúng được chuyển đổi thành Eloquent models, bạn có thể sử dụng phương thức `raw`:

```php
$orders = Order::search('Star Trek')->raw();
```

<a name="custom-indexes"></a>
#### Custom Indexes

Khi tìm kiếm sử dụng các engines bên thứ ba, các search queries thường sẽ được thực hiện trên index được chỉ định bởi phương thức [searchableAs](#configuring-model-indexes) của model. Tuy nhiên, bạn có thể sử dụng phương thức `within` để chỉ định một custom index nên được tìm kiếm thay thế:

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

<a name="where-clauses"></a>
### Where Clauses

Scout cho phép bạn thêm các mệnh đề "where" vào các search queries của bạn. Ví dụ, các kiểm tra equality cơ bản hữu ích để giới hạn các search queries theo một owner ID:

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

Bạn cũng có thể sử dụng các toán tử so sánh `=`, `!=`, `<`, `>`, `>=`, `<=` để xây dựng các queries nâng cao hơn:

```php
Order::search('Star Trek')
  ->where('status', '=', 'completed')
  ->where('is_refunded', '!=', true)
  ->where('total_price', '>', 100)
  ->where('shipping_cost', '<', 20)
  ->where('discount_percent', '>=', 10)
  ->where('item_count', '<=', 5)
  ->get();
```

Ngoài ra, phương thức `whereIn` có thể được sử dụng để xác minh rằng giá trị của một cột nhất định được chứa trong mảng đã cho:

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

Phương thức `whereNotIn` xác minh rằng giá trị của cột đã cho không được chứa trong mảng đã cho:

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

> [!WARNING]
> Nếu ứng dụng của bạn đang sử dụng Meilisearch, bạn phải cấu hình [filterable attributes](#meilisearch-index-settings) của ứng dụng của bạn trước khi sử dụng các mệnh đề "where" của Scout.

<a name="customizing-the-eloquent-results-query"></a>
#### Tùy chỉnh Eloquent Results Query

Sau khi Scout truy xuất một danh sách các Eloquent models phù hợp từ search engine của ứng dụng của bạn, Eloquent được sử dụng để truy xuất tất cả các models phù hợp theo primary keys của chúng. Bạn có thể tùy chỉnh query này bằng cách gọi phương thức `query`. Phương thức `query` chấp nhận một closure sẽ nhận instance Eloquent query builder làm đối số:

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

Khi sử dụng một engine bên thứ ba, callback này được gọi sau khi các models liên quan đã được truy xuất từ search engine, vì vậy nó không nên được sử dụng để "lọc" kết quả — sử dụng [Scout where clauses](#where-clauses) thay thế. Tuy nhiên, khi sử dụng database engine, các ràng buộc của phương thức `query` được áp dụng trực tiếp vào database query, vì vậy bạn có thể sử dụng nó để lọc cũng như.

<a name="pagination"></a>
### Pagination

Ngoài việc truy xuất một bộ sưu tập các models, bạn có thể phân trang các kết quả tìm kiếm của bạn sử dụng phương thức `paginate`. Phương thức này sẽ trả về một instance `Illuminate\Pagination\LengthAwarePaginator` giống như nếu bạn đã [phân trang một Eloquent query truyền thống](/docs/{{version}}/pagination):

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

Bạn có thể chỉ định bao nhiêu models để truy xuất mỗi trang bằng cách chuyển số lượng làm đối số đầu tiên cho phương thức `paginate`:

```php
$orders = Order::search('Star Trek')->paginate(15);
```

Khi sử dụng database engine, bạn cũng có thể sử dụng phương thức `simplePaginate`. Không giống như `paginate`, truy xuất tổng số các records phù hợp để nó có thể hiển thị số trang, `simplePaginate` chỉ xác định xem có thêm kết quả nào ngoài trang hiện tại hay không — làm cho nó hiệu quả hơn cho các bộ dữ liệu lớn nơi bạn chỉ cần các liên kết "previous" và "next":

```php
$orders = Order::search('Star Trek')->simplePaginate(15);
```

Sau khi bạn đã truy xuất kết quả, bạn có thể hiển thị kết quả và render các liên kết trang sử dụng [Blade](/docs/{{version}}/blade) giống như nếu bạn đã phân trang một Eloquent query truyền thống:

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

Tất nhiên, nếu bạn muốn truy xuất kết quả phân trang dưới dạng JSON, bạn có thể trả về instance paginator trực tiếp từ một route hoặc controller:

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    return Order::search($request->input('query'))->paginate(15);
});
```

> [!WARNING]
> Vì các search engines không biết về các định nghĩa global scope của Eloquent model của bạn, bạn không nên sử dụng global scopes trong các ứng dụng sử dụng Scout pagination. Hoặc, bạn nên tạo lại các ràng buộc của global scope khi tìm kiếm qua Scout.

<a name="soft-deleting"></a>
### Soft Deleting

Nếu các models được index của bạn là [soft deleting](/docs/{{version}}/eloquent#soft-deleting) và bạn cần tìm kiếm các soft deleted models của bạn, đặt tùy chọn `soft_delete` của file cấu hình `config/scout.php` thành `true`:

```php
'soft_delete' => true,
```

Khi tùy chọn cấu hình này là `true`, Scout sẽ không xóa các soft deleted models khỏi search index. Thay vào đó, nó sẽ đặt một attribute ẩn `__soft_deleted` trên record được index. Sau đó, bạn có thể sử dụng các phương thức `withTrashed` hoặc `onlyTrashed` để truy xuất các soft deleted records khi tìm kiếm:

```php
use App\Models\Order;

// Include trashed records when retrieving results...
$orders = Order::search('Star Trek')->withTrashed()->get();

// Only include trashed records when retrieving results...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

> [!NOTE]
> Khi một model soft deleted được xóa vĩnh viễn sử dụng `forceDelete`, Scout sẽ tự động xóa nó khỏi search index.

<a name="customizing-engine-searches"></a>
### Tùy chỉnh Engine Searches

Nếu bạn cần thực hiện tùy chỉnh nâng cao của hành vi tìm kiếm của một engine, bạn có thể chuyển một closure làm đối số thứ hai cho phương thức `search`. Ví dụ, bạn có thể sử dụng callback này để thêm dữ liệu geo-location vào các tùy chọn tìm kiếm của bạn trước khi search query được chuyển đến Algolia:

```php
use Algolia\AlgoliaSearch\SearchIndex;
use App\Models\Order;

Order::search(
    'Star Trek',
    function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];

        return $algolia->search($query, $options);
    }
)->get();
```

<a name="custom-engines"></a>
## Custom Engines

<a name="writing-the-engine"></a>
#### Viết Engine

Nếu một trong các search engines tích hợp của Scout không phù hợp với nhu cầu của bạn, bạn có thể viết custom engine của riêng mình và đăng ký nó với Scout. Engine của bạn nên mở rộng class trừu tượng `Laravel\Scout\Engines\Engine`. Class trừu tượng này chứa tám phương thức mà custom engine của bạn phải triển khai:

```php
use Laravel\Scout\Builder;

abstract public function update($models);
abstract public function delete($models);
abstract public function search(Builder $builder);
abstract public function paginate(Builder $builder, $perPage, $page);
abstract public function mapIds($results);
abstract public function map(Builder $builder, $results, $model);
abstract public function getTotalCount($results);
abstract public function flush($model);
```

Bạn có thể thấy hữu ích khi xem xét các triển khai của các phương thức này trên class `Laravel\Scout\Engines\AlgoliaEngine`. Class này sẽ cung cấp cho bạn một điểm bắt đầu tốt để học cách triển khai từng phương thức này trong engine của riêng bạn.

<a name="registering-the-engine"></a>
#### Đăng ký Engine

Sau khi bạn đã viết custom engine của mình, bạn có thể đăng ký nó với Scout sử dụng phương thức `extend` của Scout engine manager. Scout engine manager có thể được giải quyết từ Laravel service container. Bạn nên gọi phương thức `extend` từ phương thức `boot` của class `App\Providers\AppServiceProvider` của bạn hoặc bất kỳ service provider nào khác được sử dụng bởi ứng dụng của bạn:

```php
use App\ScoutExtensions\MySqlSearchEngine;
use Laravel\Scout\EngineManager;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    resolve(EngineManager::class)->extend('mysql', function () {
        return new MySqlSearchEngine;
    });
}
```

Sau khi engine của bạn đã được đăng ký, bạn có thể chỉ định nó làm Scout `driver` mặc định của bạn trong file cấu hình `config/scout.php` của ứng dụng của bạn:

```php
'driver' => 'mysql',
```

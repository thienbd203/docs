# Database: Migrations

- [Introduction](#introduction)
- [Generating Migrations](#generating-migrations)
    - [Squashing Migrations](#squashing-migrations)
- [Migration Structure](#migration-structure)
- [Running Migrations](#running-migrations)
    - [Rolling Back Migrations](#rolling-back-migrations)
- [Tables](#tables)
    - [Creating Tables](#creating-tables)
    - [Updating Tables](#updating-tables)
    - [Renaming / Dropping Tables](#renaming-and-dropping-tables)
- [Columns](#columns)
    - [Creating Columns](#creating-columns)
    - [Available Column Types](#available-column-types)
    - [Column Modifiers](#column-modifiers)
    - [Modifying Columns](#modifying-columns)
    - [Renaming Columns](#renaming-columns)
    - [Dropping Columns](#dropping-columns)
- [Indexes](#indexes)
    - [Creating Indexes](#creating-indexes)
    - [Renaming Indexes](#renaming-indexes)
    - [Dropping Indexes](#dropping-indexes)
    - [Foreign Key Constraints](#foreign-key-constraints)
- [Events](#events)

<a name="introduction"></a>
## Introduction

Migrations giống như version control cho database của bạn, cho phép team của bạn định nghĩa và chia sẻ định nghĩa schema database của ứng dụng. Nếu bạn từng phải yêu cầu một teammate thêm thủ công một cột vào schema database local của họ sau khi pull các thay đổi của bạn từ source control, bạn đã gặp phải vấn đề mà database migrations giải quyết.

Laravel `Schema` [facade](/docs/{{version}}/facades) cung cấp hỗ trợ database agnostic để tạo và thao tác các bảng trên tất cả các database systems được Laravel hỗ trợ. Thông thường, migrations sẽ sử dụng facade này để tạo và sửa đổi các bảng và cột database.

<a name="generating-migrations"></a>
## Generating Migrations

Bạn có thể sử dụng [Artisan command](/docs/{{version}}/artisan) `make:migration` để tạo một database migration. Migration mới sẽ được đặt trong thư mục `database/migrations` của bạn. Mỗi tên file migration chứa một timestamp cho phép Laravel xác định thứ tự của các migrations:

```shell
php artisan make:migration create_flights_table
```

Laravel sẽ sử dụng tên của migration để cố gắng đoán tên của bảng và liệu migration có tạo một bảng mới hay không. Nếu Laravel có thể xác định tên bảng từ tên migration, Laravel sẽ pre-fill file migration được tạo với bảng đã chỉ định. Nếu không, bạn có thể chỉ định bảng trong file migration thủ công.

Nếu bạn muốn chỉ định một path tùy chỉnh cho migration được tạo, bạn có thể sử dụng tùy chọn `--path` khi thực thi command `make:migration`. Path đã cho nên tương đối với base path của ứng dụng của bạn.

> [!NOTE]
> Migration stubs có thể được tùy chỉnh bằng cách sử dụng [stub publishing](/docs/{{version}}/artisan#stub-customization).

<a name="squashing-migrations"></a>
### Squashing Migrations

Khi bạn xây dựng ứng dụng của mình, bạn có thể tích lũy ngày càng nhiều migrations theo thời gian. Điều này có thể dẫn đến việc thư mục `database/migrations` của bạn bị phình to với hàng trăm migrations. Nếu bạn muốn, bạn có thể "squash" các migrations của mình thành một file SQL duy nhất. Để bắt đầu, thực thi command `schema:dump`:

```shell
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

Khi bạn thực thi command này, Laravel sẽ viết một file "schema" vào thư mục `database/schema` của ứng dụng. Tên file schema sẽ tương ứng với kết nối database. Bây giờ, khi bạn cố gắng migrate database của mình và không có migrations nào khác đã được thực thi, Laravel sẽ thực thi các câu lệnh SQL trong file schema của kết nối database bạn đang sử dụng trước. Sau khi thực thi các câu lệnh SQL của file schema, Laravel sẽ thực thi bất kỳ migrations còn lại nào không phải là một phần của schema dump.

Nếu các tests của ứng dụng của bạn sử dụng một kết nối database khác với kết nối bạn thường sử dụng trong quá trình phát triển local, bạn nên đảm bảo bạn đã dump một file schema bằng cách sử dụng kết nối database đó để các tests của bạn có thể xây dựng database của bạn. Bạn có thể muốn làm điều này sau khi dump kết nối database bạn thường sử dụng trong quá trình phát triển local:

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

Bạn nên commit file schema database của bạn vào source control để các developers mới khác trong team của bạn có thể nhanh chóng tạo cấu trúc database ban đầu của ứng dụng.

> [!WARNING]
> Migration squashing chỉ có sẵn cho các database MariaDB, MySQL, PostgreSQL, và SQLite và sử dụng command-line client của database.

<a name="migration-structure"></a>
## Migration Structure

Một migration class chứa hai phương thức: `up` và `down`. Phương thức `up` được sử dụng để thêm các bảng, cột, hoặc indexes mới vào database, trong khi phương thức `down` nên đảo ngược các thao tác được thực hiện bởi phương thức `up`.

Trong cả hai phương thức này, bạn có thể sử dụng Laravel schema builder để tạo và sửa đổi các bảng một cách expressive. Để tìm hiểu về tất cả các phương thức có sẵn trên `Schema` builder, [xem tài liệu của nó](#creating-tables). Ví dụ, migration sau tạo một bảng `flights`:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

<a name="setting-the-migration-connection"></a>
#### Setting the Migration Connection

Nếu migration của bạn sẽ tương tác với một kết nối database khác với kết nối database mặc định của ứng dụng, bạn nên đặt property `$connection` của migration:

```php
/**
 * The database connection that should be used by the migration.
 *
 * @var string
 */
protected $connection = 'pgsql';

/**
 * Run the migrations.
 */
public function up(): void
{
    // ...
}
```

<a name="skipping-migrations"></a>
#### Skipping Migrations

Đôi khi một migration có thể được dự định để hỗ trợ một tính năng chưa hoạt động và bạn không muốn nó chạy ngay bây giờ. Trong trường hợp này, bạn có thể định nghĩa một phương thức `shouldRun` trên migration. Nếu phương thức `shouldRun` trả về `false`, migration sẽ bị bỏ qua:

```php
use App\Models\Flight;
use Laravel\Pennant\Feature;

/**
 * Determine if this migration should run.
 */
public function shouldRun(): bool
{
    return Feature::active(Flight::class);
}
```

<a name="running-migrations"></a>
## Running Migrations

Để chạy tất cả các migrations chưa chạy của bạn, thực thi command `migrate` của Artisan:

```shell
php artisan migrate
```

Nếu bạn muốn xem các migrations nào đã chạy và các migrations nào vẫn đang chờ, bạn có thể sử dụng command `migrate:status` của Artisan:

```shell
php artisan migrate:status
```

Nếu bạn cung cấp tùy chọn `--step` cho command `migrate`, command sẽ chạy mỗi migration như một batch riêng, cho phép bạn rollback các migrations riêng lẻ sau này bằng cách sử dụng command `migrate:rollback`:

```shell
php artisan migrate --step
```

Nếu bạn muốn xem các câu lệnh SQL sẽ được thực thi bởi các migrations mà không thực sự chạy chúng, bạn có thể cung cấp flag `--pretend` cho command `migrate`:

```shell
php artisan migrate --pretend
```

<a name="isolating-migration-execution"></a>
#### Isolating Migration Execution

Nếu bạn đang triển khai ứng dụng của mình trên nhiều servers và chạy migrations như một phần của quá trình triển khai, có thể bạn không muốn hai servers cố gắng migrate database cùng một lúc. Để tránh điều này, bạn có thể sử dụng tùy chọn `isolated` khi gọi command `migrate`.

Khi tùy chọn `isolated` được cung cấp, Laravel sẽ có được một atomic lock bằng cách sử dụng cache driver của ứng dụng trước khi cố gắng chạy các migrations của bạn. Tất cả các lần cố gắng khác để chạy command `migrate` trong khi lock đó được giữ sẽ không thực thi; tuy nhiên, command vẫn sẽ thoát với một mã trạng thái thoát thành công:

```shell
php artisan migrate --isolated
```

> [!WARNING]
> Để sử dụng tính năng này, ứng dụng của bạn phải sử dụng cache driver `memcached`, `redis`, `dynamodb`, `database`, `file`, hoặc `array` làm cache driver mặc định của ứng dụng. Ngoài ra, tất cả các servers phải giao tiếp với cùng một central cache server.

<a name="forcing-migrations-to-run-in-production"></a>
#### Forcing Migrations to Run in Production

Một số thao tác migration là destructive, điều này có nghĩa là chúng có thể khiến bạn mất dữ liệu. Để bảo vệ bạn khỏi việc chạy các commands này đối với database production của bạn, bạn sẽ được nhắc xác nhận trước khi các commands được thực thi. Để buộc các commands chạy mà không có prompt, sử dụng flag `--force`:

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### Rolling Back Migrations

Để rollback thao tác migration mới nhất, bạn có thể sử dụng command `rollback` của Artisan. Command này rollback "batch" migrations cuối cùng, có thể bao gồm nhiều file migration:

```shell
php artisan migrate:rollback
```

Bạn có thể rollback một số lượng migrations giới hạn bằng cách cung cấp tùy chọn `step` cho command `rollback`. Ví dụ, command sau sẽ rollback năm migrations cuối cùng:

```shell
php artisan migrate:rollback --step=5
```

Bạn có thể rollback một "batch" migrations cụ thể bằng cách cung cấp tùy chọn `batch` cho command `rollback`, trong đó tùy chọn `batch` tương ứng với một giá trị batch trong bảng `migrations` database của ứng dụng. Ví dụ, command sau sẽ rollback tất cả các migrations trong batch ba:

```shell
php artisan migrate:rollback --batch=3
```

Nếu bạn muốn xem các câu lệnh SQL sẽ được thực thi bởi các migrations mà không thực sự chạy chúng, bạn có thể cung cấp flag `--pretend` cho command `migrate:rollback`:

```shell
php artisan migrate:rollback --pretend
```

Command `migrate:reset` sẽ rollback tất cả các migrations của ứng dụng:

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### Roll Back and Migrate Using a Single Command

Command `migrate:refresh` sẽ rollback tất cả các migrations của bạn và sau đó thực thi command `migrate`. Command này thực sự tái tạo toàn bộ database của bạn:

```shell
php artisan migrate:refresh

# Refresh the database and run all database seeds...
php artisan migrate:refresh --seed
```

Bạn có thể rollback và re-migrate một số lượng migrations giới hạn bằng cách cung cấp tùy chọn `step` cho command `refresh`. Ví dụ, command sau sẽ rollback và re-migrate năm migrations cuối cùng:

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### Drop All Tables and Migrate

Command `migrate:fresh` sẽ drop tất cả các bảng khỏi database và sau đó thực thi command `migrate`:

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

Theo mặc định, command `migrate:fresh` chỉ drop các bảng từ kết nối database mặc định. Tuy nhiên, bạn có thể sử dụng tùy chọn `--database` để chỉ định kết nối database nên được migrate. Tên kết nối database nên tương ứng với một kết nối được định nghĩa trong file cấu hình `database` [configuration file](/docs/{{version}}/configuration) của ứng dụng:

```shell
php artisan migrate:fresh --database=admin
```

> [!WARNING]
> Command `migrate:fresh` sẽ drop tất cả các bảng database bất kể prefix của chúng. Command này nên được sử dụng thận trọng khi phát triển trên một database được chia sẻ với các ứng dụng khác.

<a name="tables"></a>
## Tables

<a name="creating-tables"></a>
### Creating Tables

Để tạo một database table mới, sử dụng phương thức `create` trên `Schema` facade. Phương thức `create` chấp nhận hai đối số: đối số đầu tiên là tên của bảng, trong khi đối số thứ hai là một closure nhận một `Blueprint` object có thể được sử dụng để định nghĩa bảng mới:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```

Khi tạo bảng, bạn có thể sử dụng bất kỳ [phương thức cột](#creating-columns) nào của schema builder để định nghĩa các cột của bảng.

<a name="determining-table-column-existence"></a>
#### Determining Table / Column Existence

Bạn có thể xác định sự tồn tại của một bảng, cột, hoặc index bằng cách sử dụng các phương thức `hasTable`, `hasColumn`, và `hasIndex`:

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}

if (Schema::hasColumn('users', 'email')) {
    // The "users" table exists and has an "email" column...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // The "users" table exists and has a unique index on the "email" column...
}
```

<a name="database-connection-table-options"></a>
#### Database Connection and Table Options

Nếu bạn muốn thực hiện một thao tác schema trên một kết nối database không phải là kết nối mặc định của ứng dụng, sử dụng phương thức `connection`:

```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```

Ngoài ra, một số properties và methods khác có thể được sử dụng để định nghĩa các khía cạnh khác của việc tạo bảng. Property `engine` có thể được sử dụng để chỉ định storage engine của bảng khi sử dụng MariaDB hoặc MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```

Các properties `charset` và `collation` có thể được sử dụng để chỉ định character set và collation cho bảng được tạo khi sử dụng MariaDB hoặc MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```

Phương thức `temporary` có thể được sử dụng để chỉ định rằng bảng nên là "temporary". Temporary tables chỉ hiển thị với session database của kết nối hiện tại và được drop tự động khi kết nối được đóng:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```

Nếu bạn muốn thêm một "comment" vào một database table, bạn có thể gọi phương thức `comment` trên table instance. Table comments hiện chỉ được hỗ trợ bởi MariaDB, MySQL, và PostgreSQL:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

<a name="updating-tables"></a>
### Updating Tables

Phương thức `table` trên `Schema` facade có thể được sử dụng để cập nhật các bảng hiện có. Giống như phương thức `create`, phương thức `table` chấp nhận hai đối số: tên của bảng và một closure nhận một `Blueprint` instance bạn có thể sử dụng để thêm các cột hoặc indexes vào bảng:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="renaming-and-dropping-tables"></a>
### Renaming / Dropping Tables

Để đổi tên một database table hiện có, sử dụng phương thức `rename`:

```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```

Để drop một bảng hiện có, bạn có thể sử dụng các phương thức `drop` hoặc `dropIfExists`:

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

<a name="renaming-tables-with-foreign-keys"></a>
#### Renaming Tables With Foreign Keys

Trước khi đổi tên một bảng, bạn nên xác minh rằng bất kỳ foreign key constraints nào trên bảng có một tên rõ ràng trong các file migration của bạn thay vì để Laravel gán một tên dựa trên convention. Nếu không, tên foreign key constraint sẽ tham chiếu đến tên bảng cũ.

<a name="columns"></a>
## Columns

<a name="creating-columns"></a>
### Creating Columns

Phương thức `table` trên `Schema` facade có thể được sử dụng để cập nhật các bảng hiện có. Giống như phương thức `create`, phương thức `table` chấp nhận hai đối số: tên của bảng và một closure nhận một `Illuminate\Database\Schema\Blueprint` instance bạn có thể sử dụng để thêm các cột vào bảng:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="available-column-types"></a>
### Available Column Types

Schema builder blueprint cung cấp nhiều phương thức tương ứng với các loại cột khác nhau bạn có thể thêm vào các bảng database của mình. Mỗi phương thức có sẵn được liệt kê trong bảng dưới đây:

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="booleans-method-list"></a>
#### Boolean Types

<div class="collection-method-list" markdown="1">

[boolean](#column-method-boolean)

</div>

<a name="strings-and-texts-method-list"></a>
#### String & Text Types

<div class="collection-method-list" markdown="1">

[char](#column-method-char)
[longText](#column-method-longText)
[mediumText](#column-method-mediumText)
[string](#column-method-string)
[text](#column-method-text)
[tinyText](#column-method-tinyText)

</div>

<a name="numbers--method-list"></a>
#### Numeric Types

<div class="collection-method-list" markdown="1">

[bigIncrements](#column-method-bigIncrements)
[bigInteger](#column-method-bigInteger)
[decimal](#column-method-decimal)
[double](#column-method-double)
[float](#column-method-float)
[id](#column-method-id)
[increments](#column-method-increments)
[integer](#column-method-integer)
[mediumIncrements](#column-method-mediumIncrements)
[mediumInteger](#column-method-mediumInteger)
[smallIncrements](#column-method-smallIncrements)
[smallInteger](#column-method-smallInteger)
[tinyIncrements](#column-method-tinyIncrements)
[tinyInteger](#column-method-tinyInteger)
[unsignedBigInteger](#column-method-unsignedBigInteger)
[unsignedInteger](#column-method-unsignedInteger)
[unsignedMediumInteger](#column-method-unsignedMediumInteger)
[unsignedSmallInteger](#column-method-unsignedSmallInteger)
[unsignedTinyInteger](#column-method-unsignedTinyInteger)

</div>

<a name="dates-and-times-method-list"></a>
#### Date & Time Types

<div class="collection-method-list" markdown="1">

[dateTime](#column-method-dateTime)
[dateTimeTz](#column-method-dateTimeTz)
[date](#column-method-date)
[time](#column-method-time)
[timeTz](#column-method-timeTz)
[timestamp](#column-method-timestamp)
[timestamps](#column-method-timestamps)
[timestampsTz](#column-method-timestampsTz)
[softDeletes](#column-method-softDeletes)
[softDeletesTz](#column-method-softDeletesTz)
[year](#column-method-year)

</div>

<a name="binaries-method-list"></a>
#### Binary Types

<div class="collection-method-list" markdown="1">

[binary](#column-method-binary)

</div>

<a name="object-and-jsons-method-list"></a>
#### Object & Json Types

<div class="collection-method-list" markdown="1">

[json](#column-method-json)
[jsonb](#column-method-jsonb)

</div>

<a name="uuids-and-ulids-method-list"></a>
#### UUID & ULID Types

<div class="collection-method-list" markdown="1">

[ulid](#column-method-ulid)
[ulidMorphs](#column-method-ulidMorphs)
[uuid](#column-method-uuid)
[uuidMorphs](#column-method-uuidMorphs)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)

</div>

<a name="spatials-method-list"></a>
#### Spatial Types

<div class="collection-method-list" markdown="1">

[geography](#column-method-geography)
[geometry](#column-method-geometry)

</div>

<a name="relationship-method-list"></a>
#### Relationship Types

<div class="collection-method-list" markdown="1">

[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[foreignUuidFor](#column-method-foreignUuidFor)
[morphs](#column-method-morphs)
[nullableMorphs](#column-method-nullableMorphs)

</div>

<a name="specifics-method-list"></a>
#### Specialty Types

<div class="collection-method-list" markdown="1">

[enum](#column-method-enum)
[set](#column-method-set)
[macAddress](#column-method-macAddress)
[ipAddress](#column-method-ipAddress)
[rememberToken](#column-method-rememberToken)
[vector](#column-method-vector)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

Phương thức `bigIncrements` tạo một cột `UNSIGNED BIGINT` (primary key) tương đương auto-incrementing:

```php
$table->bigIncrements('id');
```

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

Phương thức `bigInteger` tạo một cột `BIGINT` tương đương:

```php
$table->bigInteger('votes');
```

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

Phương thức `binary` tạo một cột `BLOB` tương đương:

```php
$table->binary('photo');
```

Khi sử dụng MySQL, MariaDB, hoặc SQL Server, bạn có thể truyền các đối số `length` và `fixed` để tạo cột `VARBINARY` hoặc `BINARY` tương đương:

```php
$table->binary('data', length: 16); // VARBINARY(16)

$table->binary('data', length: 16, fixed: true); // BINARY(16)
```

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

Phương thức `boolean` tạo một cột `BOOLEAN` tương đương:

```php
$table->boolean('confirmed');
```

<a name="column-method-char"></a>
#### `char()` {.collection-method}

Phương thức `char` tạo một cột `CHAR` tương đương với độ dài đã cho:

```php
$table->char('name', length: 100);
```

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

Phương thức `dateTimeTz` tạo một cột `DATETIME` (với timezone) tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->dateTimeTz('created_at', precision: 0);
```

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

Phương thức `dateTime` tạo một cột `DATETIME` tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->dateTime('created_at', precision: 0);
```

<a name="column-method-date"></a>
#### `date()` {.collection-method}

Phương thức `date` tạo một cột `DATE` tương đương:

```php
$table->date('created_at');
```

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

Phương thức `decimal` tạo một cột `DECIMAL` tương đương với độ chính xác (tổng số chữ số) và scale (số chữ số thập phân) đã cho:

```php
$table->decimal('amount', total: 8, places: 2);
```

<a name="column-method-double"></a>
#### `double()` {.collection-method}

Phương thức `double` tạo một cột `DOUBLE` tương đương:

```php
$table->double('amount');
```

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

Phương thức `enum` tạo một cột `ENUM` tương đương với các giá trị hợp lệ đã cho:

```php
$table->enum('difficulty', ['easy', 'hard']);
```

Tất nhiên, bạn có thể sử dụng phương thức `Enum::cases()` thay vì định nghĩa thủ công một mảng các giá trị được phép:

```php
use App\Enums\Difficulty;

$table->enum('difficulty', Difficulty::cases());
```

<a name="column-method-float"></a>
#### `float()` {.collection-method}

Phương thức `float` tạo một cột `FLOAT` tương đương với độ chính xác đã cho:

```php
$table->float('amount', precision: 53);
```

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

Phương thức `foreignId` tạo một cột `UNSIGNED BIGINT` tương đương:

```php
$table->foreignId('user_id');
```

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

Phương thức `foreignIdFor` thêm một cột `{column}_id` tương đương cho một model class đã cho. Loại cột sẽ là `UNSIGNED BIGINT`, `CHAR(36)`, hoặc `CHAR(26)` tùy thuộc vào loại key của model:

```php
$table->foreignIdFor(User::class);
```

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

Phương thức `foreignUlid` tạo một cột `ULID` tương đương:

```php
$table->foreignUlid('user_id');
```

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

Phương thức `foreignUuid` tạo một cột `UUID` tương đương:

```php
$table->foreignUuid('user_id');
```

<a name="column-method-foreignUuidFor"></a>
#### `foreignUuidFor()` {.collection-method}

Phương thức `foreignUuidFor` thêm một cột `{column}_id` UUID tương đương cho một model class đã cho:

```php
$table->foreignUuidFor(User::class);
```

<a name="column-method-geography"></a>
#### `geography()` {.collection-method}

Phương thức `geography` tạo một cột `GEOGRAPHY` tương đương với loại không gian và SRID (Spatial Reference System Identifier) đã cho:

```php
$table->geography('coordinates', subtype: 'point', srid: 4326);
```

> [!NOTE]
> Hỗ trợ cho các loại không gian phụ thuộc vào database driver của bạn. Vui lòng tham khảo tài liệu của database. Nếu ứng dụng của bạn sử dụng database PostgreSQL, bạn phải cài đặt extension [PostGIS](https://postgis.net) trước khi phương thức `geography` có thể được sử dụng.

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

Phương thức `geometry` tạo một cột `GEOMETRY` tương đương với loại không gian và SRID (Spatial Reference System Identifier) đã cho:

```php
$table->geometry('positions', subtype: 'point', srid: 0);
```

> [!NOTE]
> Hỗ trợ cho các loại không gian phụ thuộc vào database driver của bạn. Vui lòng tham khảo tài liệu của database. Nếu ứng dụng của bạn sử dụng database PostgreSQL, bạn phải cài đặt extension [PostGIS](https://postgis.net) trước khi phương thức `geometry` có thể được sử dụng.

<a name="column-method-id"></a>
#### `id()` {.collection-method}

Phương thức `id` là một alias của phương thức `bigIncrements`. Theo mặc định, phương thức sẽ tạo một cột `id`; tuy nhiên, bạn có thể truyền một tên cột nếu bạn muốn gán một tên khác cho cột:

```php
$table->id();
```

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

Phương thức `increments` tạo một cột `UNSIGNED INTEGER` tương đương auto-incrementing làm primary key:

```php
$table->increments('id');
```

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

Phương thức `integer` tạo một cột `INTEGER` tương đương:

```php
$table->integer('votes');
```

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

Phương thức `ipAddress` tạo một cột `VARCHAR` tương đương:

```php
$table->ipAddress('visitor');
```

Khi sử dụng PostgreSQL, một cột `INET` sẽ được tạo.

<a name="column-method-json"></a>
#### `json()` {.collection-method}

Phương thức `json` tạo một cột `JSON` tương đương:

```php
$table->json('options');
```

Khi sử dụng SQLite, một cột `TEXT` sẽ được tạo.

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

Phương thức `jsonb` tạo một cột `JSONB` tương đương:

```php
$table->jsonb('options');
```

Khi sử dụng SQLite, một cột `TEXT` sẽ được tạo.

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

Phương thức `longText` tạo một cột `LONGTEXT` tương đương:

```php
$table->longText('description');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng một character set `binary` cho cột để tạo một cột `LONGBLOB` tương đương:

```php
$table->longText('data')->charset('binary'); // LONGBLOB
```

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

Phương thức `macAddress` tạo một cột được dự định để giữ một MAC address. Một số database systems, như PostgreSQL, có một loại cột chuyên dụng cho loại dữ liệu này. Các database systems khác sẽ sử dụng một cột string tương đương:

```php
$table->macAddress('device');
```

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

Phương thức `mediumIncrements` tạo một cột `UNSIGNED MEDIUMINT` tương đương auto-incrementing làm primary key:

```php
$table->mediumIncrements('id');
```

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

Phương thức `mediumInteger` tạo một cột `MEDIUMINT` tương đương:

```php
$table->mediumInteger('votes');
```

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

Phương thức `mediumText` tạo một cột `MEDIUMTEXT` tương đương:

```php
$table->mediumText('description');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng một character set `binary` cho cột để tạo một cột `MEDIUMBLOB` tương đương:

```php
$table->mediumText('data')->charset('binary'); // MEDIUMBLOB
```

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

Phương thức `morphs` là một phương thức thuận tiện thêm một cột `{column}_type` `VARCHAR` tương đương và một cột `{column}_id` tương đương. Loại cột cho `{column}_id` sẽ là `UNSIGNED BIGINT`, `CHAR(36)`, hoặc `CHAR(26)` tùy thuộc vào loại key của model.

Phương thức này được dự định để sử dụng khi định nghĩa các cột cần thiết cho một [Eloquent relationship](/docs/{{version}}/eloquent-relationships) polymorphic. Trong ví dụ sau, các cột `taggable_type` và `taggable_id` sẽ được tạo:

```php
$table->morphs('taggable');
```

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [morphs](#column-method-morphs); tuy nhiên, các cột được tạo sẽ là "nullable":

```php
$table->nullableMorphs('taggable');
```

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [ulidMorphs](#column-method-ulidMorphs); tuy nhiên, các cột được tạo sẽ là "nullable":

```php
$table->nullableUlidMorphs('taggable');
```

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

Phương thức này tương tự như phương thức [uuidMorphs](#column-method-uuidMorphs); tuy nhiên, các cột được tạo sẽ là "nullable":

```php
$table->nullableUuidMorphs('taggable');
```

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

Phương thức `rememberToken` tạo một cột nullable, `VARCHAR(100)` tương đương được dự định để lưu trữ [authentication token](/docs/{{version}}/authentication#remembering-users) "remember me" hiện tại:

```php
$table->rememberToken();
```

<a name="column-method-set"></a>
#### `set()` {.collection-method}

Phương thức `set` tạo một cột `SET` tương đương với danh sách các giá trị hợp lệ đã cho:

```php
$table->set('flavors', ['strawberry', 'vanilla']);
```

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

Phương thức `smallIncrements` tạo một cột `UNSIGNED SMALLINT` tương đương auto-incrementing làm primary key:

```php
$table->smallIncrements('id');
```

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

Phương thức `smallInteger` tạo một cột `SMALLINT` tương đương:

```php
$table->smallInteger('votes');
```

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

Phương thức `softDeletesTz` thêm một cột `deleted_at` `TIMESTAMP` (với timezone) tương đương nullable với độ chính xác giây phân số tùy chọn. Cột này được dự định để lưu trữ timestamp `deleted_at` cần thiết cho chức năng "soft delete" của Eloquent:

```php
$table->softDeletesTz('deleted_at', precision: 0);
```

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

Phương thức `softDeletes` thêm một cột `deleted_at` `TIMESTAMP` tương đương nullable với độ chính xác giây phân số tùy chọn. Cột này được dự định để lưu trữ timestamp `deleted_at` cần thiết cho chức năng "soft delete" của Eloquent:

```php
$table->softDeletes('deleted_at', precision: 0);
```

<a name="column-method-string"></a>
#### `string()` {.collection-method}

Phương thức `string` tạo một cột `VARCHAR` tương đương với độ dài đã cho:

```php
$table->string('name', length: 100);
```

<a name="column-method-text"></a>
#### `text()` {.collection-method}

Phương thức `text` tạo một cột `TEXT` tương đương:

```php
$table->text('description');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng một character set `binary` cho cột để tạo một cột `BLOB` tương đương:

```php
$table->text('data')->charset('binary'); // BLOB
```

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

Phương thức `timeTz` tạo một cột `TIME` (với timezone) tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->timeTz('sunrise', precision: 0);
```

<a name="column-method-time"></a>
#### `time()` {.collection-method}

Phương thức `time` tạo một cột `TIME` tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->time('sunrise', precision: 0);
```

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

Phương thức `timestampTz` tạo một cột `TIMESTAMP` (với timezone) tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->timestampTz('added_at', precision: 0);
```

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

Phương thức `timestamp` tạo một cột `TIMESTAMP` tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->timestamp('added_at', precision: 0);
```

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

Phương thức `timestampsTz` tạo các cột `created_at` và `updated_at` `TIMESTAMP` (với timezone) tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->timestampsTz(precision: 0);
```

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

Phương thức `timestamps` tạo các cột `created_at` và `updated_at` `TIMESTAMP` tương đương với độ chính xác giây phân số tùy chọn:

```php
$table->timestamps(precision: 0);
```

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

Phương thức `tinyIncrements` tạo một cột `UNSIGNED TINYINT` tương đương auto-incrementing làm primary key:

```php
$table->tinyIncrements('id');
```

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

Phương thức `tinyInteger` tạo một cột `TINYINT` tương đương:

```php
$table->tinyInteger('votes');
```

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

Phương thức `tinyText` tạo một cột `TINYTEXT` tương đương:

```php
$table->tinyText('notes');
```

Khi sử dụng MySQL hoặc MariaDB, bạn có thể áp dụng một character set `binary` cho cột để tạo một cột `TINYBLOB` tương đương:

```php
$table->tinyText('data')->charset('binary'); // TINYBLOB
```

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

Phương thức `unsignedBigInteger` tạo một cột `UNSIGNED BIGINT` tương đương:

```php
$table->unsignedBigInteger('votes');
```

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

Phương thức `unsignedInteger` tạo một cột `UNSIGNED INTEGER` tương đương:

```php
$table->unsignedInteger('votes');
```

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

Phương thức `unsignedMediumInteger` tạo một cột `UNSIGNED MEDIUMINT` tương đương:

```php
$table->unsignedMediumInteger('votes');
```

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

Phương thức `unsignedSmallInteger` tạo một cột `UNSIGNED SMALLINT` tương đương:

```php
$table->unsignedSmallInteger('votes');
```

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

Phương thức `unsignedTinyInteger` tạo một cột `UNSIGNED TINYINT` tương đương:

```php
$table->unsignedTinyInteger('votes');
```

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

Phương thức `ulidMorphs` là một phương thức thuận tiện thêm một cột `{column}_type` `VARCHAR` tương đương và một cột `{column}_id` `CHAR(26)` tương đương.

Phương thức này được dự định để sử dụng khi định nghĩa các cột cần thiết cho một [Eloquent relationship](/docs/{{version}}/eloquent-relationships) polymorphic sử dụng các định danh ULID. Trong ví dụ sau, các cột `taggable_type` và `taggable_id` sẽ được tạo:

```php
$table->ulidMorphs('taggable');
```

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

Phương thức `uuidMorphs` là một phương thức thuận tiện thêm một cột `{column}_type` `VARCHAR` tương đương và một cột `{column}_id` `CHAR(36)` tương đương.

Phương thức này được dự định để sử dụng khi định nghĩa các cột cần thiết cho một [polymorphic Eloquent relationship](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) sử dụng các định danh UUID. Trong ví dụ sau, các cột `taggable_type` và `taggable_id` sẽ được tạo:

```php
$table->uuidMorphs('taggable');
```

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

Phương thức `ulid` tạo một cột `ULID` tương đương:

```php
$table->ulid('id');
```

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

Phương thức `uuid` tạo một cột `UUID` tương đương:

```php
$table->uuid('id');
```

<a name="column-method-vector"></a>
#### `vector()` {.collection-method}

Phương thức `vector` tạo một cột `vector` tương đương:

```php
$table->vector('embedding', dimensions: 100);
```

Khi sử dụng PostgreSQL, extension `pgvector` phải được tải trước khi các cột `vector` có thể được tạo:

```php
Schema::ensureVectorExtensionExists();
```

<a name="column-method-year"></a>
#### `year()` {.collection-method}

Phương thức `year` tạo một cột `YEAR` tương đương:

```php
$table->year('birth_year');
```

<a name="column-modifiers"></a>
### Column Modifiers

Ngoài các loại cột được liệt kê ở trên, có một số "column modifiers" bạn có thể sử dụng khi thêm một cột vào một database table. Ví dụ, để làm cho cột "nullable", bạn có thể sử dụng phương thức `nullable`:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

Bảng sau chứa tất cả các column modifiers có sẵn. Danh sách này không bao gồm [index modifiers](#creating-indexes):

<div class="overflow-auto">

|| Modifier                            | Description                                                                                    ||
|| ----------------------------------- | ---------------------------------------------------------------------------------------------- ||
|| `->after('column')`                 | Place the column "after" another column (MariaDB / MySQL).                                     ||
|| `->autoIncrement()`                 | Set `INTEGER` columns as auto-incrementing (primary key).                                      ||
|| `->charset('utf8mb4')`              | Specify a character set for the column (MariaDB / MySQL).                                      ||
|| `->collation('utf8mb4_unicode_ci')` | Specify a collation for the column.                                                            ||
|| `->comment('my comment')`           | Add a comment to a column (MariaDB / MySQL / PostgreSQL).                                      ||
|| `->default($value)`                 | Specify a "default" value for the column.                                                      ||
|| `->first()`                         | Place the column "first" in the table (MariaDB / MySQL).                                       ||
|| `->from($integer)`                  | Set the starting value of an auto-incrementing field (MariaDB / MySQL / PostgreSQL).           ||
|| `->instant()`                       | Add or modify the column using an instant operation (MySQL).                                   ||
|| `->invisible()`                     | Make the column "invisible" to `SELECT *` queries (MariaDB / MySQL).                           ||
|| `->lock($mode)`                     | Specify a lock mode for the column operation (MySQL).                                          ||
|| `->nullable($value = true)`         | Allow `NULL` values to be inserted into the column.                                            ||
|| `->storedAs($expression)`           | Create a stored generated column (MariaDB / MySQL / PostgreSQL / SQLite).                      ||
|| `->unsigned()`                      | Set `INTEGER` columns as `UNSIGNED` (MariaDB / MySQL).                                         ||
|| `->useCurrent()`                    | Set `TIMESTAMP` columns to use `CURRENT_TIMESTAMP` as default value.                           ||
|| `->useCurrentOnUpdate()`            | Set `TIMESTAMP` columns to use `CURRENT_TIMESTAMP` when a record is updated (MariaDB / MySQL). ||
|| `->virtualAs($expression)`          | Create a virtual generated column (MariaDB / MySQL / SQLite).                                  ||
|| `->generatedAs($expression)`        | Create an identity column with specified sequence options (PostgreSQL).                        ||
|| `->always()`                        | Defines the precedence of sequence values over input for an identity column (PostgreSQL).      ||

</div>

<a name="default-expressions"></a>
#### Default Expressions

Modifier `default` chấp nhận một giá trị hoặc một instance `Illuminate\Database\Query\Expression`. Sử dụng một instance `Expression` sẽ ngăn Laravel bọc giá trị trong quotes và cho phép bạn sử dụng các functions cụ thể của database. Một tình huống mà điều này đặc biệt hữu ích là khi bạn cần gán các giá trị mặc định cho các cột JSON:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Query\Expression;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
};
```

> [!WARNING]
> Hỗ trợ cho default expressions phụ thuộc vào database driver, phiên bản database, và loại field. Vui lòng tham khảo tài liệu của database.

<a name="column-order"></a>
#### Column Order

Khi sử dụng database MariaDB hoặc MySQL, phương thức `after` có thể được sử dụng để thêm các cột sau một cột hiện có trong schema:

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```

<a name="instant-column-operations"></a>
#### Instant Column Operations

Khi sử dụng MySQL, bạn có thể chain modifier `instant` vào một định nghĩa cột để chỉ định rằng cột nên được thêm hoặc sửa đổi bằng cách sử dụng thuật toán "instant" của MySQL. Thuật toán này cho phép một số thay đổi schema được thực hiện mà không cần rebuild bảng hoàn chỉnh, khiến chúng gần như tức thị bất kể kích thước bảng:

```php
$table->string('name')->nullable()->instant();
```

Việc thêm cột instant chỉ có thể thêm các cột vào cuối bảng, vì vậy modifier `instant` không thể được kết hợp với các modifiers `after` hoặc `first`. Ngoài ra, thuật toán không hỗ trợ tất cả các loại cột hoặc các thao tác. Nếu thao tác được yêu cầu không tương thích, MySQL sẽ raise một lỗi.

Vui lòng tham khảo [tài liệu của MySQL](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html) để xác định các thao tác nào tương thích với các sửa đổi cột instant.

<a name="ddl-locking"></a>
#### DDL Locking

Khi sử dụng MySQL, bạn có thể chain modifier `lock` vào các định nghĩa cột, index, hoặc foreign key để điều khiển locking bảng trong các thao tác schema. MySQL hỗ trợ một số chế độ lock: `none` cho phép đọc và viết đồng thời, `shared` cho phép đọc đồng thời nhưng chặn viết, `exclusive` chặn tất cả truy cập đồng thời, và `default` để MySQL chọn chế độ phù hợp nhất:

```php
$table->string('name')->lock('none');

$table->index('email')->lock('shared');
```

Nếu chế độ lock được yêu cầu không tương thích với thao tác, MySQL sẽ raise một lỗi. Modifier `lock` có thể được kết hợp với modifier `instant` để tối ưu hóa thêm các thay đổi schema:

```php
$table->string('name')->instant()->lock('none');
```

<a name="modifying-columns"></a>
### Modifying Columns

Phương thức `change` cho phép bạn sửa đổi loại và các thuộc tính của các cột hiện có. Ví dụ, bạn có thể muốn tăng kích thước của một cột `string`. Để xem phương thức `change` trong hành động, hãy tăng kích thước của cột `name` từ 25 lên 50. Để thực hiện việc này, chúng ta chỉ cần định nghĩa trạng thái mới của cột và sau đó gọi phương thức `change`:

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

Khi sửa đổi một cột, bạn phải bao gồm rõ ràng tất cả các modifiers bạn muốn giữ trên định nghĩa cột - bất kỳ thuộc tính nào bị thiếu sẽ bị drop. Ví dụ, để giữ các thuộc tính `unsigned`, `default`, và `comment`, bạn phải gọi mỗi modifier rõ ràng khi thay đổi cột:

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('my comment')->change();
});
```

Phương thức `change` không thay đổi các indexes của cột. Do đó, bạn có thể sử dụng các index modifiers để thêm hoặc drop một index rõ ràng khi sửa đổi cột:

```php
// Add an index...
$table->bigIncrements('id')->primary()->change();

// Drop an index...
$table->char('postal_code', 10)->unique(false)->change();
```

<a name="renaming-columns"></a>
### Renaming Columns

Để đổi tên một cột, bạn có thể sử dụng phương thức `renameColumn` được cung cấp bởi schema builder:

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

<a name="dropping-columns"></a>
### Dropping Columns

Để drop một cột, bạn có thể sử dụng phương thức `dropColumn` trên schema builder:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

Bạn có thể drop nhiều cột từ một bảng bằng cách truyền một mảng tên cột cho phương thức `dropColumn`:

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

<a name="available-command-aliases"></a>
#### Available Command Aliases

Laravel cung cấp một số phương thức thuận tiện liên quan đến việc drop các loại cột phổ biến. Mỗi phương thức này được mô tả trong bảng dưới đây:

<div class="overflow-auto">

|| Command                             | Description                                           ||
|| ----------------------------------- | ----------------------------------------------------- ||
|| `$table->dropMorphs('morphable');`  | Drop the `morphable_type` and `morphable_id` columns. ||
|| `$table->dropRememberToken();`      | Drop the `remember_token` column.                     ||
|| `$table->dropSoftDeletes();`        | Drop the `deleted_at` column.                         ||
|| `$table->dropSoftDeletesTz();`      | Alias of `dropSoftDeletes()` method.                  ||
|| `$table->dropTimestamps();`         | Drop the `created_at` and `updated_at` columns.       ||
|| `$table->dropTimestampsTz();`       | Alias of `dropTimestamps()` method.                   ||

</div>

<a name="indexes"></a>
## Indexes

<a name="creating-indexes"></a>
### Creating Indexes

Schema builder của Laravel hỗ trợ một số loại indexes. Ví dụ sau tạo một cột `email` mới và chỉ định rằng các giá trị của nó nên là duy nhất. Để tạo index, chúng ta có thể chain phương thức `unique` vào định nghĩa cột:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->unique();
});
```

Ngoài ra, bạn có thể tạo index sau khi định nghĩa cột. Để làm điều này, bạn nên gọi phương thức `unique` trên schema builder blueprint. Phương thức này chấp nhận tên của cột nên nhận một unique index:

```php
$table->unique('email');
```

Bạn thậm chí có thể truyền một mảng các cột cho một phương thức index để tạo một index compound (hoặc composite):

```php
$table->index(['account_id', 'created_at']);
```

Khi tạo một index, Laravel sẽ tự động tạo một tên index dựa trên bảng, tên cột, và loại index, nhưng bạn có thể truyền một đối số thứ hai cho phương thức để chỉ định tên index của chính mình:

```php
$table->unique('email', 'unique_email');
```

<a name="available-index-types"></a>
#### Available Index Types

Schema builder blueprint class của Laravel cung cấp các phương thức để tạo mỗi loại index được Laravel hỗ trợ. Mỗi phương thức index chấp nhận một đối số thứ hai tùy chọn để chỉ định tên của index. Nếu bị bỏ qua, tên sẽ được dẫn xuất từ tên của bảng và cột(s) được sử dụng cho index, cũng như loại index. Mỗi phương thức index có sẵn được mô tả trong bảng dưới đây:

<div class="overflow-auto">

|| Command                                          | Description                                                    ||
|| ------------------------------------------------ | -------------------------------------------------------------- ||
|| `$table->primary('id');`                         | Adds a primary key.                                            ||
|| `$table->primary(['id', 'parent_id']);`          | Adds composite keys.                                           ||
|| `$table->unique('email');`                       | Adds a unique index.                                           ||
|| `$table->index('state');`                        | Adds an index.                                                 ||
|| `$table->fullText('body');`                      | Adds a full text index (MariaDB / MySQL / PostgreSQL).         ||
|| `$table->fullText('body')->language('english');` | Adds a full text index of the specified language (PostgreSQL). ||
|| `$table->spatialIndex('location');`              | Adds a spatial index (except SQLite).                          ||

</div>

<a name="online-index-creation"></a>
#### Online Index Creation

Theo mặc định, việc tạo một index trên một bảng lớn có thể lock bảng và chặn đọc hoặc viết trong khi index đang được xây dựng. Khi sử dụng PostgreSQL hoặc SQL Server, bạn có thể chain phương thức `online` vào một định nghĩa index để tạo index mà không lock bảng, cho phép ứng dụng của bạn tiếp tục đọc và ghi dữ liệu trong quá trình tạo index:

```php
$table->string('email')->unique()->online();
```

Khi sử dụng PostgreSQL, điều này thêm tùy chọn `CONCURRENTLY` vào câu lệnh tạo index. Khi sử dụng SQL Server, điều này thêm tùy chọn `WITH (online = on)`.

<a name="renaming-indexes"></a>
### Renaming Indexes

Để đổi tên một index, bạn có thể sử dụng phương thức `renameIndex` được cung cấp bởi schema builder blueprint. Phương thức này chấp nhận tên index hiện tại làm đối số đầu tiên và tên mong muốn làm đối số thứ hai:

```php
$table->renameIndex('from', 'to')
```

<a name="dropping-indexes"></a>
### Dropping Indexes

Để drop một index, bạn phải chỉ định tên của index. Theo mặc định, Laravel tự động gán một tên index dựa trên tên bảng, tên cột được index, và loại index. Dưới đây là một số ví dụ:

<div class="overflow-auto">

|| Command                                                  | Description                                                 ||
|| -------------------------------------------------------- | ----------------------------------------------------------- ||
|| `$table->dropPrimary('users_id_primary');`               | Drop a primary key from the "users" table.                  ||
|| `$table->dropUnique('users_email_unique');`              | Drop a unique index from the "users" table.                 ||
|| `$table->dropIndex('geo_state_index');`                  | Drop a basic index from the "geo" table.                    ||
|| `$table->dropFullText('posts_body_fulltext');`           | Drop a full text index from the "posts" table.              ||
|| `$table->dropSpatialIndex('geo_location_spatialindex');` | Drop a spatial index from the "geo" table  (except SQLite). ||

</div>

Nếu bạn truyền một mảng các cột vào một phương thức drop indexes, tên index convention sẽ được tạo dựa trên tên bảng, các cột, và loại index:

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // Drops index 'geo_state_index'
});
```

<a name="foreign-key-constraints"></a>
### Foreign Key Constraints

Laravel cũng cung cấp hỗ trợ để tạo foreign key constraints, được sử dụng để buộc referential integrity ở mức database. Ví dụ, hãy định nghĩa một cột `user_id` trên bảng `posts` tham chiếu đến cột `id` trên bảng `users`:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');

    $table->foreign('user_id')->references('id')->on('users');
});
```

Vì cú pháp này khá dài dòng, Laravel cung cấp các phương thức terser bổ sung sử dụng conventions để cung cấp trải nghiệm developer tốt hơn. Khi sử dụng phương thức `foreignId` để tạo cột của bạn, ví dụ trên có thể được viết lại như sau:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

Phương thức `foreignId` tạo một cột `UNSIGNED BIGINT` tương đương, trong khi phương thức `constrained` sẽ sử dụng conventions để xác định bảng và cột được tham chiếu. Nếu tên bảng của bạn không khớp với conventions của Laravel, bạn có thể cung cấp thủ công cho phương thức `constrained`. Ngoài ra, tên nên được gán cho index được tạo cũng có thể được chỉ định:

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained(
        table: 'users', indexName: 'posts_user_id'
    );
});
```

Bạn cũng có thể chỉ định hành động mong muốn cho các thuộc tính "on delete" và "on update" của constraint:

```php
$table->foreignId('user_id')
    ->constrained()
    ->onUpdate('cascade')
    ->onDelete('cascade');
```

Một cú pháp expressive thay thế cũng được cung cấp cho các hành động này:

<div class="overflow-auto">

|| Method                        | Description                                       ||
|| ----------------------------- | ------------------------------------------------- ||
|| `$table->cascadeOnUpdate();`  | Updates should cascade.                           ||
|| `$table->restrictOnUpdate();` | Updates should be restricted.                     ||
|| `$table->nullOnUpdate();`     | Updates should set the foreign key value to null. ||
|| `$table->noActionOnUpdate();` | No action on updates.                             ||
|| `$table->cascadeOnDelete();`  | Deletes should cascade.                           ||
|| `$table->restrictOnDelete();` | Deletes should be restricted.                     ||
|| `$table->nullOnDelete();`     | Deletes should set the foreign key value to null. ||
|| `$table->noActionOnDelete();` | Prevents deletes if child records exist.          ||

</div>

Bất kỳ [column modifiers](#column-modifiers) bổ sung nào phải được gọi trước phương thức `constrained`:

```php
$table->foreignId('user_id')
    ->nullable()
    ->constrained();
```

<a name="dropping-foreign-keys"></a>
#### Dropping Foreign Keys

Để drop một foreign key, bạn có thể sử dụng phương thức `dropForeign`, truyền tên của foreign key constraint cần xóa làm đối số. Foreign key constraints sử dụng cùng naming convention với indexes. Nói cách khác, tên foreign key constraint dựa trên tên của bảng và các cột trong constraint, theo sau là một suffix "\_foreign":

```php
$table->dropForeign('posts_user_id_foreign');
```

Ngoài ra, bạn có thể truyền một mảng chứa tên cột giữ foreign key cho phương thức `dropForeign`. Mảng sẽ được chuyển đổi thành một tên foreign key constraint bằng cách sử dụng các naming conventions constraint của Laravel:

```php
$table->dropForeign(['user_id']);
```

<a name="toggling-foreign-key-constraints"></a>
#### Toggling Foreign Key Constraints

Bạn có thể bật hoặc tắt foreign key constraints trong các migrations của mình bằng cách sử dụng các phương thức sau:

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // Constraints disabled within this closure...
});
```

> [!WARNING]
> SQLite tắt foreign key constraints theo mặc định. Khi sử dụng SQLite, hãy đảm bảo [bật hỗ trợ foreign key](/docs/{{version}}/database#configuration) trong cấu hình database của bạn trước khi cố gắng tạo chúng trong các migrations.

<a name="events"></a>
## Events

Để thuận tiện, mỗi thao tác migration sẽ dispatch một [event](/docs/{{version}}/events). Tất cả các events sau đều mở rộng class cơ sở `Illuminate\Database\Events\MigrationEvent`:

<div class="overflow-auto">

|| Class                                            | Description                                      ||
|| ------------------------------------------------ | ------------------------------------------------ ||
|| `Illuminate\Database\Events\MigrationsStarted`   | A batch of migrations is about to be executed.   ||
|| `Illuminate\Database\Events\MigrationsEnded`     | A batch of migrations has finished executing.    ||
|| `Illuminate\Database\Events\MigrationStarted`    | A single migration is about to be executed.      ||
|| `Illuminate\Database\Events\MigrationEnded`      | A single migration has finished executing.       ||
|| `Illuminate\Database\Events\NoPendingMigrations` | A migration command found no pending migrations. ||
|| `Illuminate\Database\Events\SchemaDumped`        | A database schema dump has completed.            ||
|| `Illuminate\Database\Events\SchemaLoaded`        | An existing database schema dump has loaded.     ||

</div>

# Database: Getting Started

- [Introduction](#introduction)
    - [Configuration](#configuration)
    - [Read and Write Connections](#read-and-write-connections)
- [Running SQL Queries](#running-queries)
    - [Using Multiple Database Connections](#using-multiple-database-connections)
    - [Listening for Query Events](#listening-for-query-events)
    - [Monitoring Cumulative Query Time](#monitoring-cumulative-query-time)
- [Database Transactions](#database-transactions)
- [Connecting to the Database CLI](#connecting-to-the-database-cli)
- [Inspecting Your Databases](#inspecting-your-databases)
- [Monitoring Your Databases](#monitoring-your-databases)

<a name="introduction"></a>
## Introduction

Hầu hết mọi ứng dụng web hiện đại đều tương tác với database. Laravel làm cho việc tương tác với database trở nên cực kỳ đơn giản trên nhiều loại database được hỗ trợ khác nhau bằng cách sử dụng raw SQL, [fluent query builder](/docs/{{version}}/queries), và [Eloquent ORM](/docs/{{version}}/eloquent). Hiện tại, Laravel cung cấp hỗ trợ chính thức cho năm database:

<div class="content-list" markdown="1">

- MariaDB 10.3+ ([Version Policy](https://mariadb.org/about/#maintenance-policy))
- MySQL 5.7+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 10.0+ ([Version Policy](https://www.postgresql.org/support/versioning/))
- SQLite 3.26.0+
- SQL Server 2017+ ([Version Policy](https://docs.microsoft.com/en-us/lifecycle/products/?products=sql-server))

</div>

Ngoài ra, MongoDB được hỗ trợ thông qua package `mongodb/laravel-mongodb`, được chính thức duy trì bởi MongoDB. Xem tài liệu [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) để biết thêm thông tin.

<a name="configuration"></a>
### Configuration

Cấu hình cho các dịch vụ database của Laravel nằm trong file cấu hình `config/database.php` của ứng dụng. Trong file này, bạn có thể định nghĩa tất cả các kết nối database của mình, cũng như chỉ định kết nối nào nên được sử dụng mặc định. Hầu hết các tùy chọn cấu hình trong file này được điều khiển bởi các giá trị của biến môi trường của ứng dụng. Các ví dụ cho hầu hết các hệ thống database được Laravel hỗ trợ đều được cung cấp trong file này.

Theo mặc định, cấu hình môi trường mẫu của [Laravel](/docs/{{version}}/configuration#environment-configuration) đã sẵn sàng để sử dụng với [Laravel Sail](/docs/{{version}}/sail), là một cấu hình Docker để phát triển ứng dụng Laravel trên máy local của bạn. Tuy nhiên, bạn có thể tự do sửa đổi cấu hình database của mình theo nhu cầu cho database local của bạn.

<a name="sqlite-configuration"></a>
#### SQLite Configuration

Database SQLite được chứa trong một file duy nhất trên filesystem của bạn. Bạn có thể tạo một database SQLite mới bằng cách sử dụng command `touch` trong terminal: `touch database/database.sqlite`. Sau khi database đã được tạo, bạn có thể dễ dàng cấu hình các biến môi trường của mình để trỏ đến database này bằng cách đặt đường dẫn tuyệt đối đến database trong biến môi trường `DB_DATABASE`:

```ini
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

Theo mặc định, foreign key constraints được bật cho các kết nối SQLite. Nếu bạn muốn tắt chúng, bạn nên đặt biến môi trường `DB_FOREIGN_KEYS` thành `false`:

```ini
DB_FOREIGN_KEYS=false
```

> [!NOTE]
> Nếu bạn sử dụng [Laravel installer](/docs/{{version}}/installation#creating-a-laravel-project) để tạo ứng dụng Laravel và chọn SQLite làm database của bạn, Laravel sẽ tự động tạo file `database/database.sqlite` và chạy các [database migrations](/docs/{{version}}/migrations) mặc định cho bạn.

<a name="mssql-configuration"></a>
#### Microsoft SQL Server Configuration

Để sử dụng database Microsoft SQL Server, bạn nên đảm bảo rằng bạn đã cài đặt các PHP extensions `sqlsrv` và `pdo_sqlsrv` cũng như bất kỳ dependencies nào mà chúng có thể yêu cầu như Microsoft SQL ODBC driver.

<a name="configuration-using-urls"></a>
#### Configuration Using URLs

Thông thường, các kết nối database được cấu hình bằng cách sử dụng nhiều giá trị cấu hình như `host`, `database`, `username`, `password`, v.v. Mỗi giá trị cấu hình này có biến môi trường tương ứng của riêng nó. Điều này có nghĩa là khi cấu hình thông tin kết nối database của bạn trên production server, bạn cần quản lý nhiều biến môi trường.

Một số nhà cung cấp database được quản lý như AWS và Heroku cung cấp một "URL" database duy nhất chứa tất cả thông tin kết nối cho database trong một chuỗi duy nhất. Một ví dụ về database URL có thể trông giống như sau:

```html
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

Các URL này thường tuân theo một quy ước schema tiêu chuẩn:

```html
driver://username:password@host:port/database?options
```

Để thuận tiện, Laravel hỗ trợ các URL này như một thay thế cho việc cấu hình database của bạn với nhiều tùy chọn cấu hình. Nếu tùy chọn cấu hình `url` (hoặc biến môi trường `DB_URL` tương ứng) có mặt, nó sẽ được sử dụng để trích xuất thông tin kết nối và credential của database.

<a name="read-and-write-connections"></a>
### Read and Write Connections

Đôi khi bạn có thể muốn sử dụng một kết nối database cho các câu lệnh SELECT, và một kết nối khác cho các câu lệnh INSERT, UPDATE, và DELETE. Laravel làm cho việc này trở nên dễ dàng, và các kết nối thích hợp sẽ luôn được sử dụng cho dù bạn đang sử dụng raw queries, query builder, hay Eloquent ORM.

Để xem cách các kết nối read / write nên được cấu hình, hãy xem ví dụ này:

```php
'mysql' => [
    'driver' => 'mysql',
    
    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '192.168.1.3',
        ],
    ],
    'sticky' => true,
    
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        (PHP_VERSION_ID >= 80500 ? \Pdo\Mysql::ATTR_SSL_CA : \PDO::MYSQL_ATTR_SSL_CA) => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
```

Lưu ý rằng ba key đã được thêm vào mảng cấu hình: `read`, `write` và `sticky`. Các key `read` và `write` có giá trị mảng chứa một key duy nhất: `host`. Phần còn lại của các tùy chọn database cho các kết nối `read` và `write` sẽ được hợp nhất từ mảng cấu hình `mysql` chính.

Bạn chỉ cần đặt các mục trong các mảng `read` và `write` nếu bạn muốn ghi đè các giá trị từ mảng `mysql` chính. Vì vậy, trong trường hợp này, `192.168.1.1` sẽ được sử dụng làm host cho kết nối "read", trong khi `192.168.1.3` sẽ được sử dụng cho kết nối "write". Các credentials database, prefix, character set, và tất cả các tùy chọn khác trong mảng `mysql` chính sẽ được chia sẻ giữa cả hai kết nối. Khi có nhiều giá trị trong mảng cấu hình `host`, một database host sẽ được chọn ngẫu nhiên cho mỗi request.

<a name="the-sticky-option"></a>
#### The `sticky` Option

Tùy chọn `sticky` là một giá trị *tùy chọn* có thể được sử dụng để cho phép đọc ngay lập tức các bản ghi đã được ghi vào database trong chu kỳ request hiện tại. Nếu tùy chọn `sticky` được bật và một thao tác "write" đã được thực hiện đối với database trong chu kỳ request hiện tại, bất kỳ thao tác "read" nào tiếp theo sẽ sử dụng kết nối "write". Điều này đảm bảo rằng bất kỳ dữ liệu nào được ghi trong chu kỳ request có thể được đọc lại ngay lập tức từ database trong cùng request đó. Tùy thuộc vào bạn để quyết định xem đây có phải là hành vi mong muốn cho ứng dụng của bạn hay không.

<a name="running-queries"></a>
## Running SQL Queries

Sau khi bạn đã cấu hình kết nối database của mình, bạn có thể chạy các query bằng cách sử dụng `DB` facade. `DB` facade cung cấp các phương thức cho mỗi loại query: `select`, `update`, `insert`, `delete`, và `statement`.

<a name="running-a-select-query"></a>
#### Running a Select Query

Để chạy một query SELECT cơ bản, bạn có thể sử dụng phương thức `select` trên `DB` facade:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show a list of all of the application's users.
     */
    public function index(): View
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

Đối số đầu tiên được truyền cho phương thức `select` là SQL query, trong khi đối số thứ hai là bất kỳ parameter bindings nào cần được bind vào query. Thông thường, đây là các giá trị của các ràng buộc where clause. Parameter binding cung cấp bảo vệ chống lại SQL injection.

Phương thức `select` sẽ luôn trả về một `array` của kết quả. Mỗi kết quả trong mảng sẽ là một PHP `stdClass` object đại diện cho một bản ghi từ database:

```php
use Illuminate\Support\Facades\DB;

$users = DB::select('select * from users');

foreach ($users as $user) {
    echo $user->name;
}
```

<a name="selecting-scalar-values"></a>
#### Selecting Scalar Values

Đôi khi query database của bạn có thể dẫn đến một giá trị scalar duy nhất. Thay vì yêu cầu lấy kết quả scalar của query từ một record object, Laravel cho phép bạn lấy giá trị này trực tiếp bằng cách sử dụng phương thức `scalar`:

```php
$burgers = DB::scalar(
    "select count(case when food = 'burger' then 1 end) as burgers from menu"
);
```

<a name="selecting-multiple-result-sets"></a>
#### Selecting Multiple Result Sets

Nếu ứng dụng của bạn gọi các stored procedures trả về nhiều result sets, bạn có thể sử dụng phương thức `selectResultSets` để lấy tất cả các result sets được trả về bởi stored procedure:

```php
[$options, $notifications] = DB::selectResultSets(
    "CALL get_user_options_and_notifications(?)", $request->user()->id
);
```

<a name="using-named-bindings"></a>
#### Using Named Bindings

Thay vì sử dụng `?` để đại diện cho parameter bindings của bạn, bạn có thể thực thi một query bằng cách sử dụng named bindings:

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

<a name="running-an-insert-statement"></a>
#### Running an Insert Statement

Để thực thi một câu lệnh `insert`, bạn có thể sử dụng phương thức `insert` trên `DB` facade. Giống như `select`, phương thức này chấp nhận SQL query làm đối số đầu tiên và bindings làm đối số thứ hai:

```php
use Illuminate\Support\Facades\DB;

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);
```

<a name="running-an-update-statement"></a>
#### Running an Update Statement

Phương thức `update` nên được sử dụng để cập nhật các bản ghi hiện có trong database. Số lượng hàng bị ảnh hưởng bởi câu lệnh được trả về bởi phương thức:

```php
use Illuminate\Support\Facades\DB;

$affected = DB::update(
    'update users set votes = 100 where name = ?',
    ['Anita']
);
```

<a name="running-a-delete-statement"></a>
#### Running a Delete Statement

Phương thức `delete` nên được sử dụng để xóa các bản ghi khỏi database. Giống như `update`, số lượng hàng bị ảnh hưởng sẽ được trả về bởi phương thức:

```php
use Illuminate\Support\Facades\DB;

$deleted = DB::delete('delete from users');
```

<a name="running-a-general-statement"></a>
#### Running a General Statement

Một số câu lệnh database không trả về bất kỳ giá trị nào. Đối với các loại thao tác này, bạn có thể sử dụng phương thức `statement` trên `DB` facade:

```php
DB::statement('drop table users');
```

<a name="running-an-unprepared-statement"></a>
#### Running an Unprepared Statement

Đôi khi bạn có thể muốn thực thi một câu lệnh SQL mà không bind bất kỳ giá trị nào. Bạn có thể sử dụng phương thức `unprepared` của `DB` facade để thực hiện việc này:

```php
DB::unprepared('update users set votes = 100 where name = "Dries"');
```

> [!WARNING]
> Vì unprepared statements không bind parameters, chúng có thể dễ bị tấn công SQL injection. Bạn không bao giờ nên cho phép các giá trị do người dùng kiểm soát trong một unprepared statement.

<a name="implicit-commits-in-transactions"></a>
#### Implicit Commits

Khi sử dụng các phương thức `statement` và `unprepared` của `DB` facade trong transactions, bạn phải cẩn thận để tránh các câu lệnh gây ra [implicit commits](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html). Các câu lệnh này sẽ khiến database engine gián tiếp commit toàn bộ transaction, khiến Laravel không biết về mức độ transaction của database. Một ví dụ về câu lệnh như vậy là tạo một database table:

```php
DB::unprepared('create table a (col varchar(1) null)');
```

Vui lòng tham khảo manual của MySQL để biết [danh sách tất cả các câu lệnh](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html) kích hoạt implicit commits.

<a name="using-multiple-database-connections"></a>
### Using Multiple Database Connections

Nếu ứng dụng của bạn định nghĩa nhiều kết nối trong file cấu hình `config/database.php`, bạn có thể truy cập từng kết nối thông qua phương thức `connection` được cung cấp bởi `DB` facade. Tên kết nối được truyền cho phương thức `connection` nên tương ứng với một trong các kết nối được liệt kê trong file cấu hình `config/database.php` của bạn hoặc được cấu hình tại runtime bằng cách sử dụng helper `config`:

```php
use Illuminate\Support\Facades\DB;

$users = DB::connection('sqlite')->select(/* ... */);
```

Bạn có thể truy cập PDO instance gốc, bên dưới của một kết nối bằng cách sử dụng phương thức `getPdo` trên một connection instance:

```php
$pdo = DB::connection()->getPdo();
```

<a name="listening-for-query-events"></a>
### Listening for Query Events

Nếu bạn muốn chỉ định một closure được gọi cho mỗi SQL query được thực thi bởi ứng dụng của bạn, bạn có thể sử dụng phương thức `listen` của `DB` facade. Phương thức này có thể hữu ích cho việc logging queries hoặc debugging. Bạn có thể đăng ký query listener closure của mình trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        DB::listen(function (QueryExecuted $query) {
            // $query->sql;
            // $query->bindings;
            // $query->time;
            // $query->toRawSql();
        });
    }
}
```

<a name="monitoring-cumulative-query-time"></a>
### Monitoring Cumulative Query Time

Một nút thắt hiệu suất phổ biến của các ứng dụng web hiện đại là lượng thời gian chúng dành cho việc query database. May mắn thay, Laravel có thể gọi một closure hoặc callback của sự lựa chọn của bạn khi nó dành quá nhiều thời gian để query database trong một request duy nhất. Để bắt đầu, hãy cung cấp một ngưỡng thời gian query (tính bằng mili-giây) và closure cho phương thức `whenQueryingForLongerThan`. Bạn có thể gọi phương thức này trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Database\Connection;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Events\QueryExecuted;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
            // Notify development team...
        });
    }
}
```

<a name="database-transactions"></a>
## Database Transactions

Bạn có thể sử dụng phương thức `transaction` được cung cấp bởi `DB` facade để chạy một tập hợp các thao tác trong một database transaction. Nếu một exception được ném trong transaction closure, transaction sẽ tự động được rollback và exception được ném lại. Nếu closure thực thi thành công, transaction sẽ tự động được commit. Bạn không cần lo lắng về việc manual rollback hay commit trong khi sử dụng phương thức `transaction`:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
});
```

<a name="handling-deadlocks"></a>
#### Handling Deadlocks

Phương thức `transaction` chấp nhận một đối số thứ hai tùy chọn định nghĩa số lần một transaction nên được thử lại khi xảy ra deadlock. Sau khi các lần thử này đã được sử dụng hết, một exception sẽ được ném:

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
}, attempts: 5);
```

<a name="manually-using-transactions"></a>
#### Manually Using Transactions

Nếu bạn muốn bắt đầu một transaction manual và có kiểm soát hoàn toàn các rollbacks và commits, bạn có thể sử dụng phương thức `beginTransaction` được cung cấp bởi `DB` facade:

```php
use Illuminate\Support\Facades\DB;

DB::beginTransaction();
```

Bạn có thể rollback transaction thông qua phương thức `rollBack`:

```php
DB::rollBack();
```

Cuối cùng, bạn có thể commit một transaction thông qua phương thức `commit`:

```php
DB::commit();
```

> [!NOTE]
> Các phương thức transaction của `DB` facade điều khiển các transactions cho cả [query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent).

<a name="connecting-to-the-database-cli"></a>
## Connecting to the Database CLI

Nếu bạn muốn kết nối với CLI của database, bạn có thể sử dụng command `db` của Artisan:

```shell
php artisan db
```

Nếu cần, bạn có thể chỉ định tên kết nối database để kết nối với một kết nối database không phải là kết nối mặc định:

```shell
php artisan db mysql
```

<a name="inspecting-your-databases"></a>
## Inspecting Your Databases

Sử dụng các commands `db:show` và `db:table` của Artisan, bạn có thể có được thông tin có giá trị về database của bạn và các bảng liên quan của nó. Để xem tổng quan về database của bạn, bao gồm kích thước, loại, số lượng kết nối mở, và tóm tắt các bảng của nó, bạn có thể sử dụng command `db:show`:

```shell
php artisan db:show
```

Bạn có thể chỉ định kết nối database nào nên được kiểm tra bằng cách cung cấp tên kết nối database cho command thông qua tùy chọn `--database`:

```shell
php artisan db:show --database=pgsql
```

Nếu bạn muốn bao gồm số lượng hàng bảng và chi tiết database view trong output của command, bạn có thể cung cấp các tùy chọn `--counts` và `--views` tương ứng. Trên các database lớn, việc lấy số lượng hàng và chi tiết view có thể chậm:

```shell
php artisan db:show --counts --views
```

Ngoài ra, bạn có thể sử dụng các phương thức `Schema` sau để kiểm tra database của mình:

```php
use Illuminate\Support\Facades\Schema;

$tables = Schema::getTables();
$views = Schema::getViews();
$columns = Schema::getColumns('users');
$indexes = Schema::getIndexes('users');
$foreignKeys = Schema::getForeignKeys('users');
```

Nếu bạn muốn kiểm tra một kết nối database không phải là kết nối mặc định của ứng dụng, bạn có thể sử dụng phương thức `connection`:

```php
$columns = Schema::connection('sqlite')->getColumns('users');
```

<a name="table-overview"></a>
#### Table Overview

Nếu bạn muốn có tổng quan về một bảng riêng lẻ trong database của mình, bạn có thể thực thi command `db:table` của Artisan. Command này cung cấp tổng quan chung về một database table, bao gồm các cột, loại, thuộc tính, keys, và indexes của nó:

```shell
php artisan db:table users
```

<a name="monitoring-your-databases"></a>
## Monitoring Your Databases

Sử dụng command `db:monitor` của Artisan, bạn có thể chỉ đạo Laravel dispatch một event `Illuminate\Database\Events\DatabaseBusy` nếu database của bạn đang quản lý nhiều hơn một số lượng kết nối mở được chỉ định.

Để bắt đầu, bạn nên lên lịch cho command `db:monitor` để [chạy mỗi phút](/docs/{{version}}/scheduling). Command chấp nhận tên của các cấu hình kết nối database mà bạn muốn giám sát cũng như số lượng kết nối mở tối đa nên được dung chịu trước khi dispatch một event:

```shell
php artisan db:monitor --databases=mysql,pgsql --max=100
```

Việc lên lịch command này một mình là không đủ để kích hoạt một thông báo cảnh báo bạn về số lượng kết nối mở. Khi command gặp một database có số lượng kết nối mở vượt quá ngưỡng của bạn, một event `DatabaseBusy` sẽ được dispatch. Bạn nên lắng nghe event này trong `AppServiceProvider` của ứng dụng để gửi một thông báo cho bạn hoặc đội ngũ phát triển của bạn:

```php
use App\Notifications\DatabaseApproachingMaxConnections;
use Illuminate\Database\Events\DatabaseBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notifications;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (DatabaseBusy $event) {
        Notification::route('mail', 'dev@example.com')
            ->notify(new DatabaseApproachingMaxConnections(
                $event->connectionName,
                $event->connections
            ));
    });
}
```

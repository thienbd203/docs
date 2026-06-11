# Database: Query Builder

- [Introduction](#introduction)
- [Running Database Queries](#running-database-queries)
    - [Chunking Results](#chunking-results)
    - [Streaming Results Lazily](#streaming-results-lazily)
    - [Aggregates](#aggregates)
- [Select Statements](#select-statements)
- [Raw Expressions](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [Basic Where Clauses](#basic-where-clauses)
    - [Where Clauses](#where-clauses)
    - [Or Where Clauses](#or-where-clauses)
    - [Where Not Clauses](#where-not-clauses)
    - [Where Any / All / None Clauses](#where-any-all-none-clauses)
    - [JSON Where Clauses](#json-where-clauses)
    - [Additional Where Clauses](#additional-where-clauses)
    - [Logical Grouping](#logical-grouping)
- [Advanced Where Clauses](#advanced-where-clauses)
    - [Where Exists Clauses](#where-exists-clauses)
    - [Subquery Where Clauses](#subquery-where-clauses)
    - [Full Text Where Clauses](#full-text-where-clauses)
    - [Vector Similarity Clauses](#vector-similarity-clauses)
- [Ordering, Grouping, Limit and Offset](#ordering-grouping-limit-and-offset)
    - [Ordering](#ordering)
    - [Grouping](#grouping)
    - [Limit and Offset](#limit-and-offset)
- [Conditional Clauses](#conditional-clauses)
- [Insert Statements](#insert-statements)
    - [Upserts](#upserts)
- [Update Statements](#update-statements)
    - [Updating JSON Columns](#updating-json-columns)
    - [Increment and Decrement](#increment-and-decrement)
- [Delete Statements](#delete-statements)
- [Pessimistic Locking](#pessimistic-locking)
- [Reusable Query Components](#reusable-query-components)
- [Debugging](#debugging)

<a name="introduction"></a>
## Introduction

Database query builder của Laravel cung cấp một giao diện thuận tiện, fluent để tạo và chạy các database query. Nó có thể được sử dụng để thực hiện hầu hết các database operations trong ứng dụng của bạn và hoạt động hoàn hảo với tất cả các database systems được Laravel hỗ trợ.

Query builder của Laravel sử dụng PDO parameter binding để bảo vệ ứng dụng của bạn chống lại các cuộc tấn công SQL injection. Không cần làm sạch hoặc sanitize các chuỗi được truyền cho query builder làm query bindings.

> [!WARNING]
> PDO không hỗ trợ binding column names. Do đó, bạn không bao giờ nên cho phép user input để quyết định các column names được tham chiếu bởi các query của bạn, bao gồm cả các cột "order by".

<a name="running-database-queries"></a>
## Running Database Queries

<a name="retrieving-all-rows-from-a-table"></a>
#### Retrieving All Rows From a Table

Bạn có thể sử dụng phương thức `table` được cung cấp bởi `DB` facade để bắt đầu một query. Phương thức `table` trả về một query builder instance fluent cho bảng đã cho, cho phép bạn chain thêm các ràng buộc vào query và sau đó cuối cùng lấy kết quả của query bằng cách sử dụng phương thức `get`:

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
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

Phương thức `get` trả về một `Illuminate\Support\Collection` instance chứa kết quả của query trong đó mỗi kết quả là một instance của PHP `stdClass` object. Bạn có thể truy cập giá trị của mỗi cột bằng cách truy cập cột đó như một property của object:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```

> [!NOTE]
> Laravel collections cung cấp nhiều phương thức cực kỳ mạnh mẽ để map và reduce data. Để biết thêm thông tin về Laravel collections, hãy xem tài liệu [collection documentation](/docs/{{version}}/collections).

<a name="retrieving-a-single-row-column-from-a-table"></a>
#### Retrieving a Single Row / Column From a Table

Nếu bạn chỉ cần lấy một hàng duy nhất từ một database table, bạn có thể sử dụng phương thức `first` của `DB` facade. Phương thức này sẽ trả về một `stdClass` object duy nhất:

```php
$user = DB::table('users')->where('name', 'John')->first();

return $user->email;
```

Nếu bạn muốn lấy một hàng duy nhất từ một database table, nhưng ném một `Illuminate\Database\RecordNotFoundException` nếu không tìm thấy hàng phù hợp, bạn có thể sử dụng phương thức `firstOrFail`. Nếu `RecordNotFoundException` không được bắt, một HTTP response 404 sẽ tự động được gửi lại cho client:

```php
$user = DB::table('users')->where('name', 'John')->firstOrFail();
```

Nếu bạn không cần một hàng hoàn chỉnh, bạn có thể trích xuất một giá trị duy nhất từ một record bằng cách sử dụng phương thức `value`. Phương thức này sẽ trả về giá trị của cột trực tiếp:

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

Để lấy một hàng duy nhất theo giá trị cột `id` của nó, sử dụng phương thức `find`:

```php
$user = DB::table('users')->find(3);
```

<a name="retrieving-a-list-of-column-values"></a>
#### Retrieving a List of Column Values

Nếu bạn muốn lấy một `Illuminate\Support\Collection` instance chứa các giá trị của một cột duy nhất, bạn có thể sử dụng phương thức `pluck`. Trong ví dụ này, chúng ta sẽ lấy một collection của user titles:

```php
use Illuminate\Support\Facades\DB;

$titles = DB::table('users')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

Bạn có thể chỉ định cột mà collection kết quả nên sử dụng làm keys của nó bằng cách cung cấp một đối số thứ hai cho phương thức `pluck`:

```php
$titles = DB::table('users')->pluck('title', 'name');

foreach ($titles as $name => $title) {
    echo $title;
}
```

<a name="chunking-results"></a>
### Chunking Results

Nếu bạn cần làm việc với hàng nghìn database records, hãy cân nhắc sử dụng phương thức `chunk` được cung cấp bởi `DB` facade. Phương thức này lấy một chunk nhỏ kết quả tại một thời điểm và feed từng chunk vào một closure để xử lý. Ví dụ, hãy lấy toàn bộ bảng `users` trong các chunk gồm 100 records tại một thời điểm:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    foreach ($users as $user) {
        // ...
    }
});
```

Bạn có thể ngăn các chunk tiếp theo được xử lý bằng cách trả về `false` từ closure:

```php
DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    // Process the records...

    return false;
});
```

Nếu bạn đang cập nhật database records trong khi chunking kết quả, kết quả chunk của bạn có thể thay đổi theo cách không mong đợi. Nếu bạn có kế hoạch cập nhật các records được lấy trong khi chunking, luôn tốt nhất là sử dụng phương thức `chunkById` thay thế. Phương thức này sẽ tự động paginate kết quả dựa trên primary key của record:

```php
DB::table('users')->where('active', false)
    ->chunkById(100, function (Collection $users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

Vì các phương thức `chunkById` và `lazyById` thêm các điều kiện "where" của riêng chúng vào query đang được thực thi, bạn thường nên [logically group](#logical-grouping) các điều kiện của riêng bạn trong một closure:

```php
DB::table('users')->where(function ($query) {
    $query->where('credits', 1)->orWhere('credits', 2);
})->chunkById(100, function (Collection $users) {
    foreach ($users as $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['credits' => 3]);
    }
});
```

> [!WARNING]
> Khi cập nhật hoặc xóa các records bên trong chunk callback, bất kỳ thay đổi nào đối với primary key hoặc foreign keys có thể ảnh hưởng đến chunk query. Điều này có thể dẫn đến việc các records không được bao gồm trong kết quả chunked.

<a name="streaming-results-lazily"></a>
### Streaming Results Lazily

Phương thức `lazy` hoạt động tương tự như [phương thức chunk](#chunking-results) theo nghĩa là nó thực thi query trong các chunk. Tuy nhiên, thay vì truyền từng chunk vào một callback, phương thức `lazy()` trả về một [LazyCollection](/docs/{{version}}/collections#lazy-collections), cho phép bạn tương tác với kết quả như một stream duy nhất:

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function (object $user) {
    // ...
});
```

Một lần nữa, nếu bạn có kế hoạch cập nhật các records được lấy trong khi iterating qua chúng, tốt nhất là sử dụng các phương thức `lazyById` hoặc `lazyByIdDesc` thay thế. Các phương thức này sẽ tự động paginate kết quả dựa trên primary key của record:

```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function (object $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

> [!WARNING]
> Khi cập nhật hoặc xóa các records trong khi iterating qua chúng, bất kỳ thay đổi nào đối với primary key hoặc foreign keys có thể ảnh hưởng đến chunk query. Điều này có thể dẫn đến việc các records không được bao gồm trong kết quả.

<a name="aggregates"></a>
### Aggregates

Query builder cũng cung cấp nhiều phương thức để lấy các aggregate values như `count`, `max`, `min`, `avg`, và `sum`. Bạn có thể gọi bất kỳ phương thức nào trong số này sau khi xây dựng query của mình:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

Tất nhiên, bạn có thể kết hợp các phương thức này với các mệnh đề khác để tinh chỉnh cách tính toán aggregate value của bạn:

```php
$price = DB::table('orders')
    ->where('finalized', 1)
    ->avg('price');
```

<a name="determining-if-records-exist"></a>
#### Determining if Records Exist

Thay vì sử dụng phương thức `count` để xác định xem có bất kỳ records nào tồn tại phù hợp với các ràng buộc của query hay không, bạn có thể sử dụng các phương thức `exists` và `doesntExist`:

```php
if (DB::table('orders')->where('finalized', 1)->exists()) {
    // ...
}

if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

<a name="select-statements"></a>
## Select Statements

<a name="specifying-a-select-clause"></a>
#### Specifying a Select Clause

Bạn có thể không luôn muốn chọn tất cả các cột từ một database table. Sử dụng phương thức `select`, bạn có thể chỉ định một mệnh đề "select" tùy chỉnh cho query:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->select('name', 'email as user_email')
    ->get();
```

Phương thức `distinct` cho phép bạn buộc query trả về các kết quả riêng biệt:

```php
$users = DB::table('users')->distinct()->get();
```

Nếu bạn đã có một query builder instance và bạn muốn thêm một cột vào mệnh đề select hiện có của nó, bạn có thể sử dụng phương thức `addSelect`:

```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

<a name="raw-expressions"></a>
## Raw Expressions

Đôi khi bạn có thể cần chèn một chuỗi tùy ý vào một query. Để tạo một raw string expression, bạn có thể sử dụng phương thức `raw` được cung cấp bởi `DB` facade:

```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```

> [!WARNING]
> Raw statements sẽ được chèn vào query dưới dạng chuỗi, vì vậy bạn nên cực kỳ cẩn thận để tránh tạo ra các lỗ hổng SQL injection.

<a name="raw-methods"></a>
### Raw Methods

Thay vì sử dụng phương thức `DB::raw`, bạn cũng có thể sử dụng các phương thức sau để chèn một raw expression vào các phần khác nhau của query. **Hãy nhớ, Laravel không thể đảm bảo rằng bất kỳ query nào sử dụng raw expressions đều được bảo vệ chống lại các lỗ hổng SQL injection.**

<a name="selectraw"></a>
#### `selectRaw`

Phương thức `selectRaw` có thể được sử dụng thay cho `addSelect(DB::raw(/* ... */))`. Phương thức này chấp nhận một mảng bindings tùy chọn làm đối số thứ hai:

```php
$orders = DB::table('orders')
    ->selectRaw('price * ? as price_with_tax', [1.0825])
    ->get();
```

<a name="whereraw-orwhereraw"></a>
#### `whereRaw / orWhereRaw`

Các phương thức `whereRaw` và `orWhereRaw` có thể được sử dụng để chèn một mệnh đề "where" raw vào query của bạn. Các phương thức này chấp nhận một mảng bindings tùy chọn làm đối số thứ hai:

```php
$orders = DB::table('orders')
    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
    ->get();
```

<a name="havingraw-orhavingraw"></a>
#### `havingRaw / orHavingRaw`

Các phương thức `havingRaw` và `orHavingRaw` có thể được sử dụng để cung cấp một raw string làm giá trị của mệnh đề "having". Các phương thức này chấp nhận một mảng bindings tùy chọn làm đối số thứ hai:

```php
$orders = DB::table('orders')
    ->select('department', DB::raw('SUM(price) as total_sales'))
    ->groupBy('department')
    ->havingRaw('SUM(price) > ?', [2500])
    ->get();
```

<a name="orderbyraw"></a>
#### `orderByRaw`

Phương thức `orderByRaw` có thể được sử dụng để cung cấp một raw string làm giá trị của mệnh đề "order by":

```php
$orders = DB::table('orders')
    ->orderByRaw('updated_at - created_at DESC')
    ->get();
```

<a name="groupbyraw"></a>
### `groupByRaw`

Phương thức `groupByRaw` có thể được sử dụng để cung cấp một raw string làm giá trị của mệnh đề `group by`:

```php
$orders = DB::table('orders')
    ->select('city', 'state')
    ->groupByRaw('city, state')
    ->get();
```

<a name="joins"></a>
## Joins

<a name="inner-join-clause"></a>
#### Inner Join Clause

Query builder cũng có thể được sử dụng để thêm các mệnh đề join vào các query của bạn. Để thực hiện một "inner join" cơ bản, bạn có thể sử dụng phương thức `join` trên một query builder instance. Đối số đầu tiên được truyền cho phương thức `join` là tên của bảng bạn cần join, trong khi các đối số còn lại chỉ định các ràng buộc cột cho join. Bạn thậm chí có thể join nhiều bảng trong một query duy nhất:

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->get();
```

<a name="left-join-right-join-clause"></a>
#### Left Join / Right Join Clause

Nếu bạn muốn thực hiện một "left join" hoặc "right join" thay vì một "inner join", hãy sử dụng các phương thức `leftJoin` hoặc `rightJoin`. Các phương thức này có cùng signature với phương thức `join`:

```php
$users = DB::table('users')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();

$users = DB::table('users')
    ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

<a name="cross-join-clause"></a>
#### Cross Join Clause

Bạn có thể sử dụng phương thức `crossJoin` để thực hiện một "cross join". Cross joins tạo ra một cartesian product giữa bảng đầu tiên và bảng được join:

```php
$sizes = DB::table('sizes')
    ->crossJoin('colors')
    ->get();
```

<a name="advanced-join-clauses"></a>
#### Advanced Join Clauses

Bạn cũng có thể chỉ định các mệnh đề join nâng cao hơn. Để bắt đầu, hãy truyền một closure làm đối số thứ hai cho phương thức `join`. Closure sẽ nhận một `Illuminate\Database\Query\JoinClause` instance cho phép bạn chỉ định các ràng buộc trên mệnh đề "join":

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')->orOn(/* ... */);
    })
    ->get();
```

Nếu bạn muốn sử dụng một mệnh đề "where" trên các join của mình, bạn có thể sử dụng các phương thức `where` và `orWhere` được cung cấp bởi `JoinClause` instance. Thay vì so sánh hai cột, các phương thức này sẽ so sánh cột với một giá trị:

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->where('contacts.user_id', '>', 5);
    })
    ->get();
```

<a name="subquery-joins"></a>
#### Subquery Joins

Bạn có thể sử dụng các phương thức `joinSub`, `leftJoinSub`, và `rightJoinSub` để join một query với một subquery. Mỗi phương thức này nhận ba đối số: subquery, table alias của nó, và một closure định nghĩa các cột liên quan. Trong ví dụ này, chúng ta sẽ lấy một collection của users trong đó mỗi user record cũng chứa timestamp `created_at` của blog post gần nhất được xuất bản của user:

```php
$latestPosts = DB::table('posts')
    ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
    ->where('is_published', true)
    ->groupBy('user_id');

$users = DB::table('users')
    ->joinSub($latestPosts, 'latest_posts', function (JoinClause $join) {
        $join->on('users.id', '=', 'latest_posts.user_id');
    })->get();
```

<a name="lateral-joins"></a>
#### Lateral Joins

> [!WARNING]
> Lateral joins hiện được hỗ trợ bởi PostgreSQL, MySQL >= 8.0.14, và SQL Server.

Bạn có thể sử dụng các phương thức `joinLateral` và `leftJoinLateral` để thực hiện một "lateral join" với một subquery. Mỗi phương thức này nhận hai đối số: subquery và table alias của nó. Các điều kiện join nên được chỉ định trong mệnh đề `where` của subquery đã cho. Lateral joins được đánh giá cho mỗi hàng và có thể tham chiếu các cột bên ngoài subquery.

Trong ví dụ này, chúng ta sẽ lấy một collection của users cũng như ba blog posts gần nhất của user. Mỗi user có thể tạo ra tối đa ba hàng trong result set: một cho mỗi blog post gần nhất của họ. Điều kiện join được chỉ định với một mệnh đề `whereColumn` trong subquery, tham chiếu đến hàng user hiện tại:

```php
$latestPosts = DB::table('posts')
    ->select('id as post_id', 'title as post_title', 'created_at as post_created_at')
    ->whereColumn('user_id', 'users.id')
    ->orderBy('created_at', 'desc')
    ->limit(3);

$users = DB::table('users')
    ->joinLateral($latestPosts, 'latest_posts')
    ->get();
```

<a name="unions"></a>
## Unions

Query builder cũng cung cấp một phương thức thuận tiện để "union" hai hoặc nhiều query lại với nhau. Ví dụ, bạn có thể tạo một query ban đầu và sử dụng phương thức `union` để union nó với nhiều query khác:

```php
use Illuminate\Support\Facades\DB;

$usersWithoutFirstName = DB::table('users')
    ->whereNull('first_name');

$users = DB::table('users')
    ->whereNull('last_name')
    ->union($usersWithoutFirstName)
    ->get();
```

Ngoài phương thức `union`, query builder cung cấp phương thức `unionAll`. Các query được kết hợp bằng phương thức `unionAll` sẽ không có các kết quả trùng lặp của chúng bị loại bỏ. Phương thức `unionAll` có cùng method signature với phương thức `union`.

<a name="basic-where-clauses"></a>
## Basic Where Clauses

<a name="where-clauses"></a>
### Where Clauses

Bạn có thể sử dụng phương thức `where` của query builder để thêm các mệnh đề "where" vào query. Lời gọi cơ bản nhất đến phương thức `where` yêu cầu ba đối số. Đối số đầu tiên là tên của cột. Đối số thứ hai là một operator, có thể là bất kỳ operator nào được database hỗ trợ. Đối số thứ ba là giá trị để so sánh với giá trị của cột.

Ví dụ, query sau lấy users trong đó giá trị của cột `votes` bằng `100` và giá trị của cột `age` lớn hơn `35`:

```php
$users = DB::table('users')
    ->where('votes', '=', 100)
    ->where('age', '>', 35)
    ->get();
```

Để thuận tiện, nếu bạn muốn xác minh rằng một cột là `=` với một giá trị đã cho, bạn có thể truyền giá trị làm đối số thứ hai cho phương thức `where`. Laravel sẽ giả định bạn muốn sử dụng operator `=`:

```php
$users = DB::table('users')->where('votes', 100)->get();
```

Bạn cũng có thể cung cấp một associative array cho phương thức `where` để query nhanh chóng trên nhiều cột:

```php
$users = DB::table('users')->where([
    'first_name' => 'Jane',
    'last_name' => 'Doe',
])->get();
```

Như đã đề cập trước đó, bạn có thể sử dụng bất kỳ operator nào được database system của bạn hỗ trợ:

```php
$users = DB::table('users')
    ->where('votes', '>=', 100)
    ->get();

$users = DB::table('users')
    ->where('votes', '<>', 100)
    ->get();

$users = DB::table('users')
    ->where('name', 'like', 'T%')
    ->get();
```

Bạn cũng có thể truyền một mảng các điều kiện cho hàm `where`. Mỗi phần tử của mảng nên là một mảng chứa ba đối số thường được truyền cho phương thức `where`:

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

> [!WARNING]
> PDO không hỗ trợ binding column names. Do đó, bạn không bao giờ nên cho phép user input để quyết định các column names được tham chiếu bởi các query của bạn, bao gồm cả các cột "order by".

> [!WARNING]
> MySQL và MariaDB tự động typecast strings sang integers trong các so sánh string-number. Trong quá trình này, các chuỗi không phải số được chuyển đổi thành `0`, điều này có thể dẫn đến kết quả không mong đợi. Ví dụ, nếu bảng của bạn có một cột `secret` với giá trị `aaa` và bạn chạy `User::where('secret', 0)`, hàng đó sẽ được trả về. Để tránh điều này, hãy đảm bảo tất cả các giá trị được typecast sang các loại thích hợp của chúng trước khi sử dụng chúng trong các query.

<a name="or-where-clauses"></a>
### Or Where Clauses

Khi chain các lời gọi đến phương thức `where` của query builder, các mệnh đề "where" sẽ được join lại với nhau bằng operator `and`. Tuy nhiên, bạn có thể sử dụng phương thức `orWhere` để join một mệnh đề vào query bằng operator `or`. Phương thức `orWhere` chấp nhận các đối số giống như phương thức `where`:

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere('name', 'John')
    ->get();
```

Nếu bạn cần group một điều kiện "or" trong ngoặc đơn, bạn có thể truyền một closure làm đối số đầu tiên cho phương thức `orWhere`:

```php
use Illuminate\Database\Query\Builder; 

$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere(function (Builder $query) {
        $query->where('name', 'Abigail')
            ->where('votes', '>', 50);
        })
    ->get();
```

Ví dụ trên sẽ tạo ra SQL sau:

```sql
select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
```

> [!WARNING]
> Bạn nên luôn group các lời gọi `orWhere` để tránh hành vi không mong đợi khi global scopes được áp dụng.

<a name="where-not-clauses"></a>
### Where Not Clauses

Các phương thức `whereNot` và `orWhereNot` có thể được sử dụng để phủ định một nhóm ràng buộc query đã cho. Ví dụ, query sau loại trừ các sản phẩm đang được clearance hoặc có giá thấp hơn mười:

```php
$products = DB::table('products')
    ->whereNot(function (Builder $query) {
        $query->where('clearance', true)
            ->orWhere('price', '<', 10);
        })
    ->get();
```

<a name="where-any-all-none-clauses"></a>
### Where Any / All / None Clauses

Đôi khi bạn có thể cần áp dụng cùng một ràng buộc query cho nhiều cột. Ví dụ, bạn có thể muốn lấy tất cả các records trong đó bất kỳ cột nào trong một danh sách đã cho là `LIKE` một giá trị đã cho. Bạn có thể thực hiện việc này bằng cách sử dụng phương thức `whereAny`:

```php
$users = DB::table('users')
    ->where('active', true)
    ->whereAny([
        'name',
        'email',
        'phone',
    ], 'like', 'Example%')
    ->get();
```

Query trên sẽ tạo ra SQL sau:

```sql
SELECT *
FROM users
WHERE active = true AND (
    name LIKE 'Example%' OR
    email LIKE 'Example%' OR
    phone LIKE 'Example%'
)
```

Tương tự, phương thức `whereAll` có thể được sử dụng để lấy các records trong đó tất cả các cột đã cho phù hợp với một ràng buộc đã cho:

```php
$posts = DB::table('posts')
    ->where('published', true)
    ->whereAll([
        'title',
        'content',
    ], 'like', '%Laravel%')
    ->get();
```

Query trên sẽ tạo ra SQL sau:

```sql
SELECT *
FROM posts
WHERE published = true AND (
    title LIKE '%Laravel%' AND
    content LIKE '%Laravel%'
)
```

Phương thức `whereNone` có thể được sử dụng để lấy các records trong đó không có cột nào trong số các cột đã cho phù hợp với một ràng buộc đã cho:

```php
$albums = DB::table('albums')
    ->where('published', true)
    ->whereNone([
        'title',
        'lyrics',
        'tags',
    ], 'like', '%explicit%')
    ->get();
```

Query trên sẽ tạo ra SQL sau:

```sql
SELECT *
FROM albums
WHERE published = true AND NOT (
    title LIKE '%explicit%' OR
    lyrics LIKE '%explicit%' OR
    tags LIKE '%explicit%'
)
```

<a name="json-where-clauses"></a>
### JSON Where Clauses

Laravel cũng hỗ trợ query các loại cột JSON trên các database cung cấp hỗ trợ cho các loại cột JSON. Hiện tại, điều này bao gồm MariaDB 10.3+, MySQL 8.0+, PostgreSQL 12.0+, SQL Server 2017+, và SQLite 3.39.0+. Để query một cột JSON, sử dụng operator `->`:

```php
$users = DB::table('users')
    ->where('preferences->dining->meal', 'salad')
    ->get();

$users = DB::table('users')
    ->whereIn('preferences->dining->meal', ['pasta', 'salad', 'sandwiches'])
    ->get();
```

Bạn có thể sử dụng các phương thức `whereJsonContains` và `whereJsonDoesntContain` để query các mảng JSON:

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', 'en')
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContain('options->languages', 'en')
    ->get();
```

Nếu ứng dụng của bạn sử dụng các database MariaDB, MySQL, hoặc PostgreSQL, bạn có thể truyền một mảng các giá trị cho các phương thức `whereJsonContains` và `whereJsonDoesntContain`:

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', ['en', 'de'])
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContain('options->languages', ['en', 'de'])
    ->get();
```

Ngoài ra, bạn có thể sử dụng các phương thức `whereJsonContainsKey` hoặc `whereJsonDoesntContainKey` để lấy các kết quả bao gồm hoặc không bao gồm một JSON key:

```php
$users = DB::table('users')
    ->whereJsonContainsKey('preferences->dietary_requirements')
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContainKey('preferences->dietary_requirements')
    ->get();
```

Cuối cùng, bạn có thể sử dụng phương thức `whereJsonLength` để query các mảng JSON theo độ dài của chúng:

```php
$users = DB::table('users')
    ->whereJsonLength('options->languages', 0)
    ->get();

$users = DB::table('users')
    ->whereJsonLength('options->languages', '>', 1)
    ->get();
```

<a name="additional-where-clauses"></a>
### Additional Where Clauses

**whereLike / orWhereLike / whereNotLike / orWhereNotLike**

Phương thức `whereLike` cho phép bạn thêm các mệnh đề "LIKE" vào query của bạn để pattern matching. Các phương thức này cung cấp một cách database-agnostic để thực hiện các query string matching, với khả năng toggle case-sensitivity. Theo mặc định, string matching là case-insensitive:

```php
$users = DB::table('users')
    ->whereLike('name', '%John%')
    ->get();
```

Bạn có thể bật tìm kiếm case-sensitive thông qua đối số `caseSensitive`:

```php
$users = DB::table('users')
    ->whereLike('name', '%John%', caseSensitive: true)
    ->get();
```

Phương thức `orWhereLike` cho phép bạn thêm một mệnh đề "or" với điều kiện LIKE:

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereLike('name', '%John%')
    ->get();
```

Phương thức `whereNotLike` cho phép bạn thêm các mệnh đề "NOT LIKE" vào query của bạn:

```php
$users = DB::table('users')
    ->whereNotLike('name', '%John%')
    ->get();
```

Tương tự, bạn có thể sử dụng `orWhereNotLike` để thêm một mệnh đề "or" với điều kiện NOT LIKE:

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereNotLike('name', '%John%')
    ->get();
```

> [!WARNING]
> Tùy chọn tìm kiếm case-sensitive của `whereLike` hiện không được hỗ trợ trên SQL Server.

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

Phương thức `whereIn` xác minh rằng giá trị của một cột đã cho nằm trong mảng đã cho:

```php
$users = DB::table('users')
    ->whereIn('id', [1, 2, 3])
    ->get();
```

Phương thức `whereNotIn` xác minh rằng giá trị của cột đã cho không nằm trong mảng đã cho:

```php
$users = DB::table('users')
    ->whereNotIn('id', [1, 2, 3])
    ->get();
```

Bạn cũng có thể cung cấp một query object làm đối số thứ hai của phương thức `whereIn`:

```php
$activeUsers = DB::table('users')->select('id')->where('is_active', 1);

$comments = DB::table('comments')
    ->whereIn('user_id', $activeUsers)
    ->get();
```

Ví dụ trên sẽ tạo ra SQL sau:

```sql
select * from comments where user_id in (
    select id
    from users
    where is_active = 1
)
```

> [!WARNING]
> Nếu bạn đang thêm một mảng lớn các integer bindings vào query của mình, các phương thức `whereIntegerInRaw` hoặc `whereIntegerNotInRaw` có thể được sử dụng để giảm đáng kể việc sử dụng bộ nhớ của bạn.

**whereBetween / orWhereBetween**

Phương thức `whereBetween` xác minh rằng giá trị của một cột nằm giữa hai giá trị:

```php
$users = DB::table('users')
    ->whereBetween('votes', [1, 100])
    ->get();
```

**whereNotBetween / orWhereNotBetween**

Phương thức `whereNotBetween` xác minh rằng giá trị của một cột nằm ngoài hai giá trị:

```php
$users = DB::table('users')
    ->whereNotBetween('votes', [1, 100])
    ->get();
```

**whereBetweenColumns / whereNotBetweenColumns / orWhereBetweenColumns / orWhereNotBetweenColumns**

Phương thức `whereBetweenColumns` xác minh rằng giá trị của một cột nằm giữa hai giá trị của hai cột trong cùng một hàng bảng:

```php
$patients = DB::table('patients')
    ->whereBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

Phương thức `whereNotBetweenColumns` xác minh rằng giá trị của một cột nằm ngoài hai giá trị của hai cột trong cùng một hàng bảng:

```php
$patients = DB::table('patients')
    ->whereNotBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

**whereValueBetween / whereValueNotBetween / orWhereValueBetween / orWhereValueNotBetween**

Phương thức `whereValueBetween` xác minh rằng một giá trị đã cho nằm giữa các giá trị của hai cột cùng loại trong cùng một hàng bảng:

```php
$products = DB::table('products')
    ->whereValueBetween(100, ['min_price', 'max_price'])
    ->get();
```

Phương thức `whereValueNotBetween` xác minh rằng một giá trị nằm ngoài các giá trị của hai cột trong cùng một hàng bảng:

```php
$products = DB::table('products')
    ->whereValueNotBetween(100, ['min_price', 'max_price'])
    ->get();
```

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

Phương thức `whereNull` xác minh rằng giá trị của cột đã cho là `NULL`:

```php
$users = DB::table('users')
    ->whereNull('updated_at')
    ->get();
```

Phương thức `whereNotNull` xác minh rằng giá trị của cột không phải là `NULL`:

```php
$users = DB::table('users')
    ->whereNotNull('updated_at')
    ->get();
```

**whereNullSafeEquals / orWhereNullSafeEquals**

Các phương thức `whereNullSafeEquals` và `orWhereNullSafeEquals` có thể được sử dụng để so sánh giá trị của một cột với một giá trị đã given trong khi coi hai giá trị `NULL` là bằng nhau:

```php
$lastLoginIp = $request->input('last_login_ip');

$users = DB::table('users')
    ->whereNullSafeEquals('last_login_ip', $lastLoginIp)
    ->get();
```

**whereDate / whereMonth / whereDay / whereYear / whereTime**

Phương thức `whereDate` có thể được sử dụng để so sánh giá trị của một cột với một ngày:

```php
$users = DB::table('users')
    ->whereDate('created_at', '2016-12-31')
    ->get();
```

Phương thức `whereMonth` có thể được sử dụng để so sánh giá trị của một cột với một tháng cụ thể:

```php
$users = DB::table('users')
    ->whereMonth('created_at', '12')
    ->get();
```

Phương thức `whereDay` có thể được sử dụng để so sánh giá trị của một cột với một ngày cụ thể của tháng:

```php
$users = DB::table('users')
    ->whereDay('created_at', '31')
    ->get();
```

Phương thức `whereYear` có thể được sử dụng để so sánh giá trị của một cột với một năm cụ thể:

```php
$users = DB::table('users')
    ->whereYear('created_at', '2016')
    ->get();
```

Phương thức `whereTime` có thể được sử dụng để so sánh giá trị của một cột với một thời gian cụ thể:

```php
$users = DB::table('users')
    ->whereTime('created_at', '=', '11:20:45')
    ->get();
```

**wherePast / whereFuture / whereToday / whereBeforeToday / whereAfterToday**

Các phương thức `wherePast` và `whereFuture` có thể được sử dụng để xác định xem giá trị của một cột có ở trong quá khứ hay tương lai:

```php
$invoices = DB::table('invoices')
    ->wherePast('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereFuture('due_at')
    ->get();
```

Các phương thức `whereNowOrPast` và `whereNowOrFuture` có thể được sử dụng để xác định xem giá trị của một cột có ở trong quá khứ hay tương lai, bao gồm cả ngày và giờ hiện tại:

```php
$invoices = DB::table('invoices')
    ->whereNowOrPast('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereNowOrFuture('due_at')
    ->get();
```

Các phương thức `whereToday`, `whereBeforeToday`, và `whereAfterToday` có thể được sử dụng để xác định xem giá trị của một cột có phải là hôm nay, trước hôm nay, hay sau hôm nay, tương ứng:

```php
$invoices = DB::table('invoices')
    ->whereToday('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereBeforeToday('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereAfterToday('due_at')
    ->get();
```

Tương tự, các phương thức `whereTodayOrBefore` và `whereTodayOrAfter` có thể được sử dụng để xác định xem giá trị của một cột có phải là trước hôm nay hay sau hôm nay, bao gồm cả ngày hôm nay:

```php
$invoices = DB::table('invoices')
    ->whereTodayOrBefore('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereTodayOrAfter('due_at')
    ->get();
```

**whereColumn / orWhereColumn**

Phương thức `whereColumn` có thể được sử dụng để xác minh rằng hai cột bằng nhau:

```php
$users = DB::table('users')
    ->whereColumn('first_name', 'last_name')
    ->get();
```

Bạn cũng có thể truyền một comparison operator cho phương thức `whereColumn`:

```php
$users = DB::table('users')
    ->whereColumn('updated_at', '>', 'created_at')
    ->get();
```

Bạn cũng có thể truyền một mảng các so sánh cột cho phương thức `whereColumn`. Các điều kiện này sẽ được join bằng operator `and`:

```php
$users = DB::table('users')
    ->whereColumn([
        ['first_name', '=', 'last_name'],
        ['updated_at', '>', 'created_at'],
    ])->get();
```

<a name="logical-grouping"></a>
### Logical Grouping

Đôi khi bạn có thể cần group một số mệnh đề "where" trong ngoặc đơn để đạt được logical grouping mong muốn cho query của mình. Thực tế, bạn thường nên luôn group các lời gọi đến phương thức `orWhere` trong ngoặc đơn để tránh hành vi query không mong đợi. Để thực hiện việc này, bạn có thể truyền một closure cho phương thức `where`:

```php
$users = DB::table('users')
    ->where('name', '=', 'John')
    ->where(function (Builder $query) {
        $query->where('votes', '>', 100)
            ->orWhere('title', '=', 'Admin');
    })
    ->get();
```

Như bạn có thể thấy, việc truyền một closure vào phương thức `where` hướng dẫn query builder bắt đầu một nhóm ràng buộc. Closure sẽ nhận một query builder instance mà bạn có thể sử dụng để đặt các ràng buộc nên được chứa trong nhóm ngoặc đơn. Ví dụ trên sẽ tạo ra SQL sau:

```sql
select * from users where name = 'John' and (votes > 100 or title = 'Admin')
```

> [!WARNING]
> Bạn nên luôn group các lời gọi `orWhere` để tránh hành vi không mong đợi khi global scopes được áp dụng.

<a name="advanced-where-clauses"></a>
## Advanced Where Clauses

<a name="where-exists-clauses"></a>
### Where Exists Clauses

Phương thức `whereExists` cho phép bạn viết các mệnh đề "where exists" SQL. Phương thức `whereExists` chấp nhận một closure sẽ nhận một query builder instance, cho phép bạn định nghĩa query nên được đặt bên trong mệnh đề "exists":

```php
$users = DB::table('users')
    ->whereExists(function (Builder $query) {
        $query->select(DB::raw(1))
            ->from('orders')
            ->whereColumn('orders.user_id', 'users.id');
    })
    ->get();
```

Ngoài ra, bạn có thể cung cấp một query object cho phương thức `whereExists` thay vì một closure:

```php
$orders = DB::table('orders')
    ->select(DB::raw(1))
    ->whereColumn('orders.user_id', 'users.id');

$users = DB::table('users')
    ->whereExists($orders)
    ->get();
```

Cả hai ví dụ trên sẽ tạo ra SQL sau:

```sql
select * from users
where exists (
    select 1
    from orders
    where orders.user_id = users.id
)
```

<a name="subquery-where-clauses"></a>
### Subquery Where Clauses

Đôi khi bạn có thể cần xây dựng một mệnh đề "where" so sánh kết quả của một subquery với một giá trị đã cho. Bạn có thể thực hiện việc này bằng cách truyền một closure và một giá trị cho phương thức `where`. Ví dụ, query sau sẽ lấy tất cả những users có "membership" gần đây của một loại đã cho;

```php
use App\Models\User;
use Illuminate\Database\Query\Builder;

$users = User::where(function (Builder $query) {
    $query->select('type')
        ->from('membership')
        ->whereColumn('membership.user_id', 'users.id')
        ->orderByDesc('membership.start_date')
        ->limit(1);
}, 'Pro')->get();
```

Hoặc, bạn có thể cần xây dựng một mệnh đề "where" so sánh một cột với kết quả của một subquery. Bạn có thể thực hiện việc này bằng cách truyền một cột, operator, và closure cho phương thức `where`. Ví dụ, query sau sẽ lấy tất cả các income records trong đó amount nhỏ hơn trung bình;

```php
use App\Models\Income;
use Illuminate\Database\Query\Builder;

$incomes = Income::where('amount', '<', function (Builder $query) {
    $query->selectRaw('avg(i.amount)')->from('incomes as i');
})->get();
```

<a name="full-text-where-clauses"></a>
### Full Text Where Clauses

> [!WARNING]
> Full text where clauses hiện được hỗ trợ bởi MariaDB, MySQL, và PostgreSQL.

Các phương thức `whereFullText` và `orWhereFullText` có thể được sử dụng để thêm các mệnh đề "where" full text vào một query cho các cột có [full text indexes](/docs/{{version}}/migrations#available-index-types). Các phương thức này sẽ được chuyển đổi thành SQL thích hợp cho database system bên dưới bởi Laravel. Ví dụ, một mệnh đề `MATCH AGAINST` sẽ được tạo ra cho các ứng dụng sử dụng MariaDB hoặc MySQL:

```php
$users = DB::table('users')
    ->whereFullText('bio', 'web developer')
    ->get();
```

<a name="vector-similarity-clauses"></a>
### Vector Similarity Clauses

> [!NOTE]
> Vector similarity clauses hiện chỉ được hỗ trợ trên các kết nối PostgreSQL sử dụng extension `pgvector`. Để biết thông tin về việc định nghĩa các cột và indexes vector, hãy tham khảo tài liệu [migration documentation](/docs/{{version}}/migrations#available-column-types).

Phương thức `whereVectorSimilarTo` lọc kết quả theo cosine similarity với một vector đã cho và sắp xếp kết quả theo relevance. Ngưỡng `minSimilarity` nên là một giá trị giữa `0.0` và `1.0`, trong đó `1.0` là giống hệt nhau:

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

Khi một plain string được đưa làm đối số vector, Laravel sẽ tự động tạo embeddings cho nó bằng cách sử dụng [Laravel AI SDK](/docs/{{version}}/ai-sdk#embeddings):

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

Theo mặc định, `whereVectorSimilarTo` cũng sắp xếp kết quả theo distance (tương tự nhất trước). Bạn có thể tắt sắp xếp này bằng cách truyền `false` làm đối số `order`:

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4, order: false)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

Nếu bạn cần kiểm soát nhiều hơn, bạn có thể sử dụng các phương thức `selectVectorDistance`, `whereVectorDistanceLessThan`, và `orderByVectorDistance` một cách độc lập:

```php
$documents = DB::table('documents')
    ->select('*')
    ->selectVectorDistance('embedding', $queryEmbedding, as: 'distance')
    ->whereVectorDistanceLessThan('embedding', $queryEmbedding, maxDistance: 0.3)
    ->orderByVectorDistance('embedding', $queryEmbedding)
    ->limit(10)
    ->get();
```

Khi sử dụng PostgreSQL, extension `pgvector` phải được tải trước khi các cột `vector` có thể được tạo:

```php
Schema::ensureVectorExtensionExists();
```

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit and Offset

<a name="ordering"></a>
### Ordering

<a name="orderby"></a>
#### The `orderBy` Method

Phương thức `orderBy` cho phép bạn sắp xếp kết quả của query theo một cột đã cho. Đối số đầu tiên được chấp nhận bởi phương thức `orderBy` nên là cột bạn muốn sắp xếp theo, trong khi đối số thứ hai xác định hướng của sắp xếp và có thể là `asc` hoặc `desc`:

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->get();
```

Để sắp xếp theo nhiều cột, bạn có thể đơn giản gọi `orderBy` nhiều lần khi cần thiết:

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->orderBy('email', 'asc')
    ->get();
```

Hướng sắp xếp là tùy chọn và mặc định là tăng dần. Nếu bạn muốn sắp xếp theo thứ tự giảm dần, bạn có thể chỉ định tham số thứ hai cho phương thức `orderBy`, hoặc chỉ sử dụng `orderByDesc`:

```php
$users = DB::table('users')
    ->orderByDesc('verified_at')
    ->get();
```

Cuối cùng, sử dụng operator `->`, kết quả có thể được sắp xếp theo một giá trị trong một cột JSON:

```php
$corporations = DB::table('corporations')
    ->where('country', 'US')
    ->orderBy('location->state')
    ->get();
```

<a name="latest-oldest"></a>
#### The `latest` and `oldest` Methods

Các phương thức `latest` và `oldest` cho phép bạn dễ dàng sắp xếp kết quả theo ngày. Theo mặc định, kết quả sẽ được sắp xếp theo cột `created_at` của bảng. Hoặc, bạn có thể truyền tên cột mà bạn muốn sắp xếp theo:

```php
$user = DB::table('users')
    ->latest()
    ->first();
```

<a name="random-ordering"></a>
#### Random Ordering

Phương thức `inRandomOrder` có thể được sử dụng để sắp xếp kết quả query ngẫu nhiên. Ví dụ, bạn có thể sử dụng phương thức này để lấy một user ngẫu nhiên:

```php
$randomUser = DB::table('users')
    ->inRandomOrder()
    ->first();
```

<a name="removing-existing-orderings"></a>
#### Removing Existing Orderings

Phương thức `reorder` loại bỏ tất cả các mệnh đề "order by" đã được áp dụng trước đó cho query:

```php
$query = DB::table('users')->orderBy('name');

$unorderedUsers = $query->reorder()->get();
```

Bạn có thể truyền một cột và hướng khi gọi phương thức `reorder` để loại bỏ tất cả các mệnh đề "order by" hiện có và áp dụng một order hoàn toàn mới cho query:

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorder('email', 'desc')->get();
```

Để thuận tiện, bạn có thể sử dụng phương thức `reorderDesc` để reorder kết quả query theo thứ tự giảm dần:

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorderDesc('email')->get();
```

<a name="grouping"></a>
### Grouping

<a name="groupby-having"></a>
#### The `groupBy` and `having` Methods

Như bạn có thể mong đợi, các phương thức `groupBy` và `having` có thể được sử dụng để group kết quả query. Signature của phương thức `having` tương tự như của phương thức `where`:

```php
$users = DB::table('users')
    ->groupBy('account_id')
    ->having('account_id', '>', 100)
    ->get();
```

Bạn có thể sử dụng phương thức `havingBetween` để lọc kết quả trong một phạm vi đã cho:

```php
$report = DB::table('orders')
    ->selectRaw('count(id) as number_of_orders, customer_id')
    ->groupBy('customer_id')
    ->havingBetween('number_of_orders', [5, 15])
    ->get();
```

Bạn có thể truyền nhiều đối số cho phương thức `groupBy` để group theo nhiều cột:

```php
$users = DB::table('users')
    ->groupBy('first_name', 'status')
    ->having('account_id', '>', 100)
    ->get();
```

Để xây dựng các mệnh đề `having` nâng cao hơn, xem phương thức [havingRaw](#raw-methods).

<a name="limit-and-offset"></a>
### Limit and Offset

Bạn có thể sử dụng các phương thức `limit` và `offset` để giới hạn số lượng kết quả được trả về từ query hoặc để bỏ qua một số lượng kết quả đã cho trong query:

```php
$users = DB::table('users')
    ->offset(10)
    ->limit(5)
    ->get();
```

<a name="conditional-clauses"></a>
## Conditional Clauses

Đôi khi bạn có thể muốn một số query clauses áp dụng cho một query dựa trên một điều kiện khác. Ví dụ, bạn có thể chỉ muốn áp dụng một mệnh đề `where` nếu một giá trị input đã cho có mặt trên incoming HTTP request. Bạn có thể thực hiện việc này bằng cách sử dụng phương thức `when`:

```php
$role = $request->input('role');

$users = DB::table('users')
    ->when($role, function (Builder $query, string $role) {
        $query->where('role_id', $role);
    })
    ->get();
```

Phương thức `when` chỉ thực thi closure đã cho khi đối số đầu tiên là `true`. Nếu đối số đầu tiên là `false`, closure sẽ không được thực thi. Vì vậy, trong ví dụ trên, closure được đưa cho phương thức `when` sẽ chỉ được gọi nếu trường `role` có mặt trên incoming request và đánh giá là `true`.

Bạn có thể truyền một closure khác làm đối số thứ ba cho phương thức `when`. Closure này chỉ sẽ thực thi nếu đối số đầu tiên đánh giá là `false`. Để minh họa cách tính năng này có thể được sử dụng, chúng ta sẽ sử dụng nó để cấu hình ordering mặc định của một query:

```php
$sortByVotes = $request->boolean('sort_by_votes');

$users = DB::table('users')
    ->when($sortByVotes, function (Builder $query, bool $sortByVotes) {
        $query->orderBy('votes');
    }, function (Builder $query) {
        $query->orderBy('name');
    })
    ->get();
```

<a name="insert-statements"></a>
## Insert Statements

Query builder cũng cung cấp một phương thức `insert` có thể được sử dụng để chèn các records vào database table. Phương thức `insert` chấp nhận một mảng các tên cột và giá trị:

```php
DB::table('users')->insert([
    'email' => 'kayla@example.com',
    'votes' => 0
]);
```

Bạn có thể chèn nhiều records cùng một lúc bằng cách truyền một mảng của các mảng. Mỗi mảng đại diện cho một record nên được chèn vào bảng:

```php
DB::table('users')->insert([
    ['email' => 'picard@example.com', 'votes' => 0],
    ['email' => 'janeway@example.com', 'votes' => 0],
]);
```

Phương thức `insertOrIgnore` sẽ bỏ qua các lỗi trong khi chèn các records vào database. Khi sử dụng phương thức này, bạn nên biết rằng các lỗi duplicate record sẽ bị bỏ qua và các loại lỗi khác cũng có thể bị bỏ qua tùy thuộc vào database engine. Ví dụ, `insertOrIgnore` sẽ [bypass strict mode của MySQL](https://dev.mysql.com/doc/refman/en/sql-mode.html#ignore-effect-on-execution):

```php
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'sisko@example.com'],
    ['id' => 2, 'email' => 'archer@example.com'],
]);
```

Phương thức `insertUsing` sẽ chèn các records mới vào bảng trong khi sử dụng một subquery để xác định dữ liệu nên được chèn:

```php
DB::table('pruned_users')->insertUsing([
    'id', 'name', 'email', 'email_verified_at'
], DB::table('users')->select(
    'id', 'name', 'email', 'email_verified_at'
)->where('updated_at', '<=', now()->minus(months: 1)));
```

<a name="auto-incrementing-ids"></a>
#### Auto-Incrementing IDs

Nếu bảng có một id auto-incrementing, sử dụng phương thức `insertGetId` để chèn một record và sau đó lấy ID:

```php
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

> [!WARNING]
> Khi sử dụng PostgreSQL, phương thức `insertGetId` mong đợi cột auto-incrementing được đặt tên là `id`. Nếu bạn muốn lấy ID từ một "sequence" khác, bạn có thể truyền tên cột làm đối số thứ hai cho phương thức `insertGetId`.

<a name="upserts"></a>
### Upserts

Phương thức `upsert` sẽ chèn các records không tồn tại và cập nhật các records đã tồn tại với các giá trị mới mà bạn có thể chỉ định. Đối số đầu tiên của phương thức bao gồm các giá trị để chèn hoặc cập nhật, trong khi đối số thứ hai liệt kê các cột xác định duy nhất các records trong bảng liên quan. Đối số thứ ba và cuối cùng của phương thức là một mảng các cột nên được cập nhật nếu một record phù hợp đã tồn tại trong database:

```php
DB::table('flights')->upsert(
    [
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ],
    ['departure', 'destination'],
    ['price']
);
```

Trong ví dụ trên, Laravel sẽ cố gắng chèn hai records. Nếu một record đã tồn tại với cùng các giá trị cột `departure` và `destination`, Laravel sẽ cập nhật cột `price` của record đó.

> [!WARNING]
> Tất cả các database ngoại trừ SQL Server yêu cầu các cột trong đối số thứ hai của phương thức `upsert` phải có một "primary" hoặc "unique" index. Ngoài ra, các database drivers MariaDB và MySQL bỏ qua đối số thứ hai của phương thức `upsert` và luôn sử dụng các indexes "primary" và "unique" của bảng để phát hiện các records hiện có.

<a name="update-statements"></a>
## Update Statements

Ngoài việc chèn các records vào database, query builder cũng có thể cập nhật các records hiện có bằng cách sử dụng phương thức `update`. Phương thức `update`, giống như phương thức `insert`, chấp nhận một mảng các cặp cột và giá trị chỉ định các cột cần được cập nhật. Phương thức `update` trả về số lượng hàng bị ảnh hưởng. Bạn có thể ràng buộc query `update` bằng cách sử dụng các mệnh đề `where`:

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['votes' => 1]);
```

<a name="update-or-insert"></a>
#### Update or Insert

Đôi khi bạn có thể muốn cập nhật một record hiện có trong database hoặc tạo nó nếu không có record phù hợp nào tồn tại. Trong kịch bản này, phương thức `updateOrInsert` có thể được sử dụng. Phương thức `updateOrInsert` chấp nhận hai đối số: một mảng các điều kiện để tìm record, và một mảng các cặp cột và giá trị chỉ định các cột cần được cập nhật.

Phương thức `updateOrInsert` sẽ cố gắng định vị một database record phù hợp bằng cách sử dụng các cặp cột và giá trị của đối số đầu tiên. Nếu record tồn tại, nó sẽ được cập nhật với các giá trị trong đối số thứ hai. Nếu record không thể được tìm thấy, một record mới sẽ được chèn với các thuộc tính được hợp nhất của cả hai đối số:

```php
DB::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```

Bạn có thể cung cấp một closure cho phương thức `updateOrInsert` để tùy chỉnh các thuộc tính được cập nhật hoặc chèn vào database dựa trên sự tồn tại của một record phù hợp:

```php
DB::table('users')->updateOrInsert(
    ['user_id' => $user_id],
    fn ($exists) => $exists ? [
        'name' => $data['name'],
        'email' => $data['email'],
    ] : [
        'name' => $data['name'],
        'email' => $data['email'],
        'marketable' => true,
    ],
);
```

<a name="updating-json-columns"></a>
### Updating JSON Columns

Khi cập nhật một cột JSON, bạn nên sử dụng cú pháp `->` để cập nhật key thích hợp trong JSON object. Thao tác này được hỗ trợ trên MariaDB 10.3+, MySQL 5.7+, và PostgreSQL 9.5+:

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['options->enabled' => true]);
```

<a name="increment-and-decrement"></a>
### Increment and Decrement

Query builder cũng cung cấp các phương thức thuận tiện để increment hoặc decrement giá trị của một cột đã cho. Cả hai phương thức này chấp nhận ít nhất một đối số: cột để sửa đổi. Một đối số thứ hai có thể được cung cấp để chỉ định số lượng mà cột nên được increment hoặc decrement:

```php
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
```

Nếu cần, bạn cũng có thể chỉ định các cột bổ sung để cập nhật trong quá trình increment hoặc decrement:

```php
DB::table('users')->increment('votes', 1, ['name' => 'John']);
```

Ngoài ra, bạn có thể increment hoặc decrement nhiều cột cùng một lúc bằng cách sử dụng các phương thức `incrementEach` và `decrementEach`:

```php
DB::table('users')->incrementEach([
    'votes' => 5,
    'balance' => 100,
]);
```

<a name="delete-statements"></a>
## Delete Statements

Phương thức `delete` của query builder có thể được sử dụng để xóa các records khỏi bảng. Phương thức `delete` trả về số lượng hàng bị ảnh hưởng. Bạn có thể ràng buộc các câu lệnh `delete` bằng cách thêm các mệnh đề "where" trước khi gọi phương thức `delete`:

```php
$deleted = DB::table('users')->delete();

$deleted = DB::table('users')->where('votes', '>', 100)->delete();
```

<a name="pessimistic-locking"></a>
## Pessimistic Locking

Query builder cũng bao gồm một số hàm để giúp bạn đạt được "pessimistic locking" khi thực thi các câu lệnh `select` của mình. Để thực thi một câu lệnh với một "shared lock", bạn có thể gọi phương thức `sharedLock`. Một shared lock ngăn chặn các hàng được chọn bị sửa đổi cho đến khi transaction của bạn được commit:

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->sharedLock()
    ->get();
```

Ngoài ra, bạn có thể sử dụng phương thức `lockForUpdate`. Một "for update" lock ngăn chặn các records được chọn bị sửa đổi hoặc bị chọn với một shared lock khác:

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->lockForUpdate()
    ->get();
```

Mặc dù không bắt buộc, nên bọc pessimistic locks trong một [transaction](/docs/{{version}}/database#database-transactions). Điều này đảm bảo rằng dữ liệu được lấy vẫn không bị thay đổi trong database cho đến khi toàn bộ quá trình hoàn thành. Trong trường hợp thất bại, transaction sẽ rollback bất kỳ thay đổi nào và tự động giải phóng các locks:

```php
DB::transaction(function () {
    $sender = DB::table('users')
        ->lockForUpdate()
        ->find(1);

    $receiver = DB::table('users')
        ->lockForUpdate()
        ->find(2);

    if ($sender->balance < 100) {
        throw new RuntimeException('Balance too low.');
    }

    DB::table('users')
        ->where('id', $sender->id)
        ->update([
            'balance' => $sender->balance - 100
        ]);

    DB::table('users')
        ->where('id', $receiver->id)
        ->update([
            'balance' => $receiver->balance + 100
        ]);
});
```

<a name="reusable-query-components"></a>
## Reusable Query Components

Nếu bạn có logic query lặp lại trong suốt ứng dụng của mình, bạn có thể trích xuất logic thành các objects có thể tái sử dụng bằng cách sử dụng các phương thức `tap` và `pipe` của query builder. Hãy tưởng tượng bạn có hai query khác nhau trong ứng dụng của mình:

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\DB;

$destination = $request->query('destination');

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) {
        $query->where('destination', $destination);
    })
    ->orderByDesc('price')
    ->get();

// ...

$destination = $request->query('destination');

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) {
        $query->where('destination', $destination);
    })
    ->where('user', $request->user()->id)
    ->orderBy('destination')
    ->get();
```

Bạn có thể muốn trích xuất lọc destination chung giữa các queries thành một object có thể tái sử dụng:

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Query\Builder;

class DestinationFilter
{
    public function __construct(
        private ?string $destination,
    ) {
        //
    }

    public function __invoke(Builder $query): void
    {
        $query->when($this->destination, function (Builder $query) {
            $query->where('destination', $this->destination);
        });
    }
}
```

Sau đó, bạn có thể sử dụng phương thức `tap` của query builder để áp dụng logic của object cho query:

```php
use App\Scopes\DestinationFilter;
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\DB;

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) { // [tl! remove]
        $query->where('destination', $destination); // [tl! remove]
    }) // [tl! remove]
    ->tap(new DestinationFilter($destination)) // [tl! add]
    ->orderByDesc('price')
    ->get();

// ...

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) { // [tl! remove]
        $query->where('destination', $destination); // [tl! remove]
    }) // [tl! remove]
    ->tap(new DestinationFilter($destination)) // [tl! add]
    ->where('user', $request->user()->id)
    ->orderBy('destination')
    ->get();
```

<a name="query-pipes"></a>
#### Query Pipes

Phương thức `tap` sẽ luôn trả về query builder. Nếu bạn muốn trích xuất một object thực thi query và trả về một giá trị khác, bạn có thể sử dụng phương thức `pipe` thay thế.

Hãy xem xét query object sau chứa logic [pagination](/docs/{{version}}/pagination) chung được sử dụng trong suốt một ứng dụng. Không giống như `DestinationFilter`, áp dụng các điều kiện query cho query, object `Paginate` thực thi query và trả về một paginator instance:

```php
<?php

namespace App\Scopes;

use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Database\Query\Builder;

class Paginate
{
    public function __construct(
        private string $sortBy = 'timestamp',
        private string $sortDirection = 'desc',
        private int $perPage = 25,
    ) {
        //
    }

    public function __invoke(Builder $query): LengthAwarePaginator
    {
        return $query->orderBy($this->sortBy, $this->sortDirection)
            ->paginate($this->perPage, pageName: 'p');
    }
}
```

Sử dụng phương thức `pipe` của query builder, chúng ta có thể tận dụng object này để áp dụng logic pagination chung của chúng ta:

```php
$flights = DB::table('flights')
    ->tap(new DestinationFilter($destination))
    ->pipe(new Paginate);
```

<a name="debugging"></a>
## Debugging

Bạn có thể sử dụng các phương thức `dd` và `dump` trong khi xây dựng một query để dump các query bindings và SQL hiện tại. Phương thức `dd` sẽ hiển thị thông tin debug và sau đó dừng thực thi request. Phương thức `dump` sẽ hiển thị thông tin debug nhưng cho phép request tiếp tục thực thi:

```php
DB::table('users')->where('votes', '>', 100)->dd();

DB::table('users')->where('votes', '>', 100)->dump();
```

Các phương thức `dumpRawSql` và `ddRawSql` có thể được gọi trên một query để dump SQL của query với tất cả các parameter bindings được thay thế đúng cách:

```php
DB::table('users')->where('votes', '>', 100)->dumpRawSql();

DB::table('users')->where('votes', '>', 100)->ddRawSql();
```

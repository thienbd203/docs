# Database: Pagination

- [Giới thiệu](#introduction)
- [Cách sử dụng cơ bản](#basic-usage)
    - [Paginating Query Builder Results](#paginating-query-builder-results)
    - [Paginating Eloquent Results](#paginating-eloquent-results)
    - [Cursor Pagination](#cursor-pagination)
    - [Tạo thủ công Paginator](#manually-creating-a-paginator)
    - [Tùy chỉnh Pagination URLs](#customizing-pagination-urls)
- [Hiển thị Pagination Results](#displaying-pagination-results)
    - [Điều chỉnh Pagination Link Window](#adjusting-the-pagination-link-window)
    - [Chuyển đổi Results sang JSON](#converting-results-to-json)
- [Tùy chỉnh Pagination View](#customizing-the-pagination-view)
    - [Sử dụng Bootstrap](#using-bootstrap)
- [Paginator và LengthAwarePaginator Instance Methods](#paginator-instance-methods)
- [Cursor Paginator Instance Methods](#cursor-paginator-instance-methods)

<a name="introduction"></a>
## Giới thiệu

Trong các frameworks khác, pagination có thể rất đau đầu. Chúng tôi hy vọng cách tiếp cận của Laravel đối với pagination sẽ là một luồng gió mới. Paginator của Laravel được tích hợp với [query builder](/docs/{{version}}/queries) và [Eloquent ORM](/docs/{{version}}/eloquent) và cung cấp pagination của database records tiện lợi, dễ sử dụng với zero configuration.

Theo mặc định, HTML được generate bởi paginator tương thích với [Tailwind CSS framework](https://tailwindcss.com/); tuy nhiên, Bootstrap pagination support cũng có sẵn.

<a name="tailwind"></a>
#### Tailwind

Nếu bạn đang sử dụng Laravel default Tailwind pagination views với Tailwind 4.x, file `resources/css/app.css` của application đã được configure đúng để `@source` Laravel pagination views:

```css
@import 'tailwindcss';

@source '../../vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php';
```

<a name="basic-usage"></a>
## Cách sử dụng cơ bản

<a name="paginating-query-builder-results"></a>
### Paginating Query Builder Results

Có một số cách để paginate items. Cách đơn giản nhất là sử dụng method `paginate` trên [query builder](/docs/{{version}}/queries) hoặc một [Eloquent query](/docs/{{version}}/eloquent). Method `paginate` tự động xử lý việc setting query's "limit" và "offset" dựa trên current page đang được xem bởi user. Theo mặc định, current page được detect bởi giá trị của query string argument `page` trên HTTP request. Giá trị này được tự động detect bởi Laravel, và cũng được tự động insert vào links được generate bởi paginator.

Trong ví dụ này, argument duy nhất được truyền vào method `paginate` là số lượng items bạn muốn hiển thị "per page". Trong trường hợp này, hãy chỉ định rằng chúng ta muốn hiển thị `15` items per page:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show all application users.
     */
    public function index(): View
    {
        return view('user.index', [
            'users' => DB::table('users')->paginate(15)
        ]);
    }
}
```

<a name="simple-pagination"></a>
#### Simple Pagination

Method `paginate` count tổng số records được match bởi query trước khi retrieve records từ database. Điều này được thực hiện để paginator biết có bao nhiêu pages của records trong tổng. Tuy nhiên, nếu bạn không có kế hoạch hiển thị tổng số pages trong UI của application thì record count query là không cần thiết.

Do đó, nếu bạn chỉ cần hiển thị simple "Next" và "Previous" links trong UI của application, bạn có thể sử dụng method `simplePaginate` để perform một single, efficient query:

```php
$users = DB::table('users')->simplePaginate(15);
```

<a name="paginating-eloquent-results"></a>
### Paginating Eloquent Results

Bạn cũng có thể paginate [Eloquent](/docs/{{version}}/eloquent) queries. Trong ví dụ này, chúng ta sẽ paginate model `App\Models\User` và chỉ định rằng chúng ta có kế hoạch hiển thị 15 records per page. Như bạn có thể thấy, cú pháp gần như giống hệt với paginating query builder results:

```php
use App\Models\User;

$users = User::paginate(15);
```

Tất nhiên, bạn có thể gọi method `paginate` sau khi setting các constraints khác trên query, chẳng hạn như `where` clauses:

```php
$users = User::where('votes', '>', 100)->paginate(15);
```

Bạn cũng có thể sử dụng method `simplePaginate` khi paginating Eloquent models:

```php
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

Tương tự, bạn có thể sử dụng method `cursorPaginate` để cursor paginate Eloquent models:

```php
$users = User::where('votes', '>', 100)->cursorPaginate(15);
```

<a name="multiple-paginator-instances-per-page"></a>
#### Multiple Paginator Instances per Page

Đôi khi bạn cần render hai separate paginators trên một single screen được render bởi application của bạn. Tuy nhiên, nếu cả hai paginator instances đều sử dụng query string parameter `page` để store current page, hai paginators sẽ conflict. Để resolve conflict này, bạn có thể pass tên của query string parameter bạn muốn sử dụng để store paginator's current page thông qua argument thứ ba được cung cấp cho các methods `paginate`, `simplePaginate`, và `cursorPaginate`:

```php
use App\Models\User;

$users = User::where('votes', '>', 100)->paginate(
    $perPage = 15, $columns = ['*'], $pageName = 'users'
);
```

<a name="cursor-pagination"></a>
### Cursor Pagination

Trong khi `paginate` và `simplePaginate` tạo queries sử dụng SQL "offset" clause, cursor pagination hoạt động bằng cách constructing "where" clauses so sánh các giá trị của ordered columns chứa trong query, cung cấp database performance tốt nhất có sẵn trong tất cả Laravel pagination methods. Method pagination này đặc biệt phù hợp cho large data-sets và "infinite" scrolling user interfaces.

Khác với offset based pagination, bao gồm một page number trong query string của URLs được generate bởi paginator, cursor-based pagination đặt một "cursor" string trong query string. Cursor là một encoded string chứa location mà next paginated query nên bắt đầu paginate và direction mà nó nên paginate:

```text
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

Bạn có thể tạo một cursor-based paginator instance thông qua method `cursorPaginate` được cung cấp bởi query builder. Method này trả về một instance của `Illuminate\Pagination\CursorPaginator`:

```php
$users = DB::table('users')->orderBy('id')->cursorPaginate(15);
```

Khi bạn đã retrieve một cursor paginator instance, bạn có thể [display the pagination results](#displaying-pagination-results) như bạn thường làm khi sử dụng các methods `paginate` và `simplePaginate`. Để biết thêm thông tin về các instance methods được cung cấp bởi cursor paginator, hãy tham khảo [cursor paginator instance method documentation](#cursor-paginator-instance-methods).

> [!WARNING]
> Query của bạn phải chứa một "order by" clause để tận dụng cursor pagination. Ngoài ra, các columns mà query được order phải thuộc về table bạn đang paginate.

<a name="cursor-vs-offset-pagination"></a>
#### Cursor so với Offset Pagination

Để illustrate sự khác biệt giữa offset pagination và cursor pagination, hãy examine một số example SQL queries. Cả hai queries sau đây sẽ hiển thị "second page" của results cho một `users` table được order bởi `id`:

```sql
# Offset Pagination...
select * from users order by id asc limit 15 offset 15;

# Cursor Pagination...
select * from users where id > 15 order by id asc limit 15;
```

Cursor pagination query cung cấp các advantages sau so với offset pagination:

- Đối với large data-sets, cursor pagination sẽ cung cấp performance tốt hơn nếu "order by" columns được indexed. Điều này là do "offset" clause scan qua tất cả previously matched data.
- Đối với data-sets với frequent writes, offset pagination có thể skip records hoặc show duplicates nếu results đã được recently added vào hoặc deleted từ page mà user hiện đang xem.

Tuy nhiên, cursor pagination có các limitations sau:

- Giống như `simplePaginate`, cursor pagination chỉ có thể được sử dụng để hiển thị "Next" và "Previous" links và không hỗ trợ generating links với page numbers.
- Nó yêu cầu rằng ordering dựa trên ít nhất một unique column hoặc một combination của columns là unique. Columns với `null` values không được hỗ trợ.
- Query expressions trong "order by" clauses chỉ được hỗ trợ nếu chúng được aliased và thêm vào "select" clause.
- Query expressions với parameters không được hỗ trợ.

<a name="manually-creating-a-paginator"></a>
### Tạo thủ công Paginator

Đôi khi bạn có thể muốn tạo một pagination instance thủ công, truyền cho nó một array của items mà bạn đã có trong memory. Bạn có thể làm điều này bằng cách tạo một instance `Illuminate\Pagination\Paginator`, `Illuminate\Pagination\LengthAwarePaginator` hoặc `Illuminate\Pagination\CursorPaginator`, tùy thuộc vào nhu cầu của bạn.

Các classes `Paginator` và `CursorPaginator` không cần biết tổng số items trong result set; tuy nhiên, vì điều này, các classes này không có methods để retrieve index của last page. `LengthAwarePaginator` chấp nhận gần như cùng arguments với `Paginator`; tuy nhiên, nó yêu cầu một count của tổng số items trong result set.

Nói cách khác, `Paginator` tương ứng với method `simplePaginate` trên query builder, `CursorPaginator` tương ứng với method `cursorPaginate`, và `LengthAwarePaginator` tương ứng với method `paginate`.

> [!WARNING]
> Khi tạo thủ công một paginator instance, bạn nên thủ công "slice" array của results mà bạn truyền cho paginator. Nếu bạn không chắc cách làm điều này, hãy kiểm tra [array_slice](https://secure.php.net/manual/en/function.array-slice.php) PHP function.

<a name="customizing-pagination-urls"></a>
### Tùy chỉnh Pagination URLs

Theo mặc định, links được generate bởi paginator sẽ match URI của current request. Tuy nhiên, method `withPath` của paginator cho phép bạn tùy chỉnh URI được sử dụng bởi paginator khi generating links. Ví dụ, nếu bạn muốn paginator generate links như `http://example.com/admin/users?page=N`, bạn nên pass `/admin/users` vào method `withPath`:

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->withPath('/admin/users');

    // ...
});
```

<a name="appending-query-string-values"></a>
#### Appending Query String Values

Bạn có thể append vào query string của pagination links sử dụng method `appends`. Ví dụ, để append `sort=votes` vào mỗi pagination link, bạn nên thực hiện call sau đến `appends`:

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->appends(['sort' => 'votes']);

    // ...
});
```

Bạn có thể sử dụng method `withQueryString` nếu bạn muốn append tất cả query string values của current request vào pagination links:

```php
$users = User::paginate(15)->withQueryString();
```

<a name="appending-hash-fragments"></a>
#### Appending Hash Fragments

Nếu bạn cần append một "hash fragment" vào URLs được generate bởi paginator, bạn có thể sử dụng method `fragment`. Ví dụ, để append `#users` vào cuối mỗi pagination link, bạn nên invoke method `fragment` như sau:

```php
$users = User::paginate(15)->fragment('users');
```

<a name="displaying-pagination-results"></a>
## Hiển thị Pagination Results

Khi gọi method `paginate`, bạn sẽ nhận được một instance của `Illuminate\Pagination\LengthAwarePaginator`, trong khi gọi method `simplePaginate` trả về một instance của `Illuminate\Pagination\Paginator`. Và cuối cùng, gọi method `cursorPaginate` trả về một instance của `Illuminate\Pagination\CursorPaginator`.

Các objects này cung cấp một số methods mô tả result set. Ngoài các helper methods này, paginator instances là iterators và có thể được loop như một array. Vì vậy, khi bạn đã retrieve results, bạn có thể display results và render page links sử dụng [Blade](/docs/{{version}}/blade):

```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

Method `links` sẽ render links đến phần còn lại của pages trong result set. Mỗi link này sẽ đã chứa proper `page` query string variable. Hãy nhớ, HTML được generate bởi method `links` tương thích với [Tailwind CSS framework](https://tailwindcss.com).

<a name="adjusting-the-pagination-link-window"></a>
### Điều chỉnh Pagination Link Window

Khi paginator hiển thị pagination links, current page number được hiển thị cũng như links cho ba pages trước và sau current page. Sử dụng method `onEachSide`, bạn có thể control bao nhiêu additional links được hiển thị trên mỗi bên của current page trong middle, sliding window của links được generate bởi paginator:

```blade
{{ $users->onEachSide(5)->links() }}
```

<a name="converting-results-to-json"></a>
### Chuyển đổi Results sang JSON

Laravel paginator classes implement `Illuminate\Contracts\Support\Jsonable` Interface contract và expose method `toJson`, vì vậy rất dễ convert pagination results của bạn sang JSON. Bạn cũng có thể convert một paginator instance sang JSON bằng cách return nó từ một route hoặc controller action:

```php
use App\Models\User;

Route::get('/users', function () {
    return User::paginate();
});
```

JSON từ paginator sẽ bao gồm meta information như `total`, `current_page`, `last_page`, và nhiều hơn nữa. Result records có sẵn qua key `data` trong JSON array. Đây là một ví dụ về JSON được tạo bằng cách return một paginator instance từ một route:

```json
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "current_page_url": "http://laravel.app?page=1",
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // Record...
        },
        {
            // Record...
        }
   ]
}
```

<a name="customizing-the-pagination-view"></a>
## Tùy chỉnh Pagination View

Theo mặc định, views được render để display pagination links tương thích với [Tailwind CSS](https://tailwindcss.com) framework. Tuy nhiên, nếu bạn không sử dụng Tailwind, bạn tự do định nghĩa views của riêng bạn để render các links này. Khi gọi method `links` trên một paginator instance, bạn có thể pass view name là argument đầu tiên cho method:

```blade
{{ $paginator->links('view.name') }}

<!-- Passing additional data to the view... -->
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```

Tuy nhiên, cách dễ nhất để tùy chỉnh pagination views là export chúng đến directory `resources/views/vendor` của bạn sử dụng command `vendor:publish`:

```shell
php artisan vendor:publish --tag=laravel-pagination
```

Command này sẽ đặt views trong directory `resources/views/vendor/pagination` của application. File `tailwind.blade.php` trong directory này tương ứng với default pagination view. Bạn có thể edit file này để modify pagination HTML.

Nếu bạn muốn designate một file khác là default pagination view, bạn có thể invoke paginator's `defaultView` và `defaultSimpleView` methods trong `boot` method của class `App\Providers\AppServiceProvider` của bạn:

```php
<?php

namespace App\Providers;

use Illuminate\Pagination\Paginator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }
}
```

<a name="using-bootstrap"></a>
### Sử dụng Bootstrap

Laravel bao gồm pagination views được xây dựng sử dụng [Bootstrap CSS](https://getbootstrap.com/). Để sử dụng các views này thay vì default Tailwind views, bạn có thể gọi paginator's `useBootstrapFour` hoặc `useBootstrapFive` methods trong `boot` method của class `App\Providers\AppServiceProvider` của bạn:

```php
use Illuminate\Pagination\Paginator;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Paginator::useBootstrapFive();
    Paginator::useBootstrapFour();
}
```

<a name="paginator-instance-methods"></a>
## Paginator / LengthAwarePaginator Instance Methods

Mỗi paginator instance cung cấp additional pagination information thông qua các methods sau:

<div class="overflow-auto">

| Method                                  | Description                                                                                                  |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `$paginator->count()`                   | Get the number of items for the current page.                                                                |
| `$paginator->currentPage()`             | Get the current page number.                                                                                 |
| `$paginator->firstItem()`               | Get the result number of the first item in the results.                                                      |
| `$paginator->getOptions()`              | Get the paginator options.                                                                                   |
| `$paginator->getUrlRange($start, $end)` | Create a range of pagination URLs.                                                                           |
| `$paginator->hasPages()`                | Determine if there are enough items to split into multiple pages.                                            |
| `$paginator->hasMorePages()`            | Determine if there are more items in the data store.                                                         |
| `$paginator->items()`                   | Get the items for the current page.                                                                          |
| `$paginator->lastItem()`                | Get the result number of the last item in the results.                                                       |
| `$paginator->lastPage()`                | Get the page number of the last available page. (Not available when using `simplePaginate`).                 |
| `$paginator->nextPageUrl()`             | Get the URL for the next page.                                                                               |
| `$paginator->onFirstPage()`             | Determine if the paginator is on the first page.                                                             |
| `$paginator->onLastPage()`              | Determine if the paginator is on the last page.                                                              |
| `$paginator->perPage()`                 | The number of items to be shown per page.                                                                    |
| `$paginator->previousPageUrl()`         | Get the URL for the previous page.                                                                           |
| `$paginator->total()`                   | Determine the total number of matching items in the data store. (Not available when using `simplePaginate`). |
| `$paginator->url($page)`                | Get the URL for a given page number.                                                                         |
| `$paginator->getPageName()`             | Get the query string variable used to store the page.                                                        |
| `$paginator->setPageName($name)`        | Set the query string variable used to store the page.                                                        |
| `$paginator->through($callback)`        | Transform each item using a callback.                                                                        |

</div>

<a name="cursor-paginator-instance-methods"></a>
## Cursor Paginator Instance Methods

Mỗi cursor paginator instance cung cấp additional pagination information thông qua các methods sau:

<div class="overflow-auto">

| Method                          | Description                                                       |
| ------------------------------- | ----------------------------------------------------------------- |
| `$paginator->count()`           | Get the number of items for the current page.                     |
| `$paginator->cursor()`          | Get the current cursor instance.                                  |
| `$paginator->getOptions()`      | Get the paginator options.                                        |
| `$paginator->hasPages()`        | Determine if there are enough items to split into multiple pages. |
| `$paginator->hasMorePages()`    | Determine if there are more items in the data store.              |
| `$paginator->getCursorName()`   | Get the query string variable used to store the cursor.           |
| `$paginator->items()`           | Get the items for the current page.                               |
| `$paginator->nextCursor()`      | Get the cursor instance for the next set of items.                |
| `$paginator->nextPageUrl()`     | Get the URL for the next page.                                    |
| `$paginator->onFirstPage()`     | Determine if the paginator is on the first page.                  |
| `$paginator->onLastPage()`      | Determine if the paginator is on the last page.                   |
| `$paginator->perPage()`         | The number of items to be shown per page.                         |
| `$paginator->previousCursor()`  | Get the cursor instance for the previous set of items.            |
| `$paginator->previousPageUrl()` | Get the URL for the previous page.                                |
| `$paginator->setCursorName()`   | Set the query string variable used to store the cursor.           |
| `$paginator->url($cursor)`      | Get the URL for a given cursor instance.                          |

</div>

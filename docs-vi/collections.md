# Collections

- [Giới thiệu](#introduction)
    - [Tạo Collections](#creating-collections)
    - [Mở rộng Collections](#extending-collections)
- [Các Phương thức Có sẵn](#available-methods)
- [Thông điệp Cấp cao hơn](#higher-order-messages)
- [Lazy Collections](#lazy-collections)
    - [Giới thiệu](#lazy-collection-introduction)
    - [Tạo Lazy Collections](#creating-lazy-collections)
    - [Enumerable Contract](#the-enumerable-contract)
    - [Các Phương thức Lazy Collection](#lazy-collection-methods)

<a name="introduction"></a>
## Giới thiệu

Lớp `Illuminate\Support\Collection` cung cấp một wrapper trôi chảy, tiện lợi để làm việc với mảng dữ liệu. Ví dụ, hãy xem đoạn mã sau. Chúng ta sẽ sử dụng helper `collect` để tạo một thể hiện collection mới từ mảng, chạy hàm `strtoupper` trên mỗi phần tử, và sau đó xóa tất cả các phần tử rỗng:

```php
$collection = collect(['Taylor', 'Abigail', null])->map(function (?string $name) {
    return strtoupper($name);
})->reject(function (string $name) {
    return empty($name);
});
```

Như bạn có thể thấy, lớp `Collection` cho phép bạn xâu chuỗi các phương thức của nó để thực hiện ánh xạ và giảm trôi chảy của mảng cơ bản. Nói chung, collections là bất biến, nghĩa là mỗi phương thức `Collection` trả về một thể hiện `Collection` hoàn toàn mới.

<a name="creating-collections"></a>
### Tạo Collections

Như đã đề cập ở trên, helper `collect` trả về một thể hiện `Illuminate\Support\Collection` mới cho mảng đã cho. Vì vậy, tạo một collection đơn giản như:

```php
$collection = collect([1, 2, 3]);
```

Bạn cũng có thể tạo một collection bằng các phương thức [make](#method-make) và [fromJson](#method-fromjson).

> [!NOTE]
> Kết quả của các truy vấn [Eloquent](/docs/{{version}}/eloquent) luôn được trả về dưới dạng các thể hiện `Collection`.

<a name="extending-collections"></a>
### Mở rộng Collections

Collections là "macroable", cho phép bạn thêm các phương thức bổ sung vào lớp `Collection` tại thời điểm chạy. Phương thức `macro` của lớp `Illuminate\Support\Collection` chấp nhận một closure sẽ được thực thi khi macro của bạn được gọi. Closure macro có thể truy cập các phương thức khác của collection thông qua `$this`, giống như nó là một phương thức thực sự của lớp collection. Ví dụ, đoạn mã sau thêm phương thức `toUpper` vào lớp `Collection`:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function (string $value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

Thông thường, bạn nên khai báo các collection macro trong phương thức `boot` của một [service provider](/docs/{{version}}/providers).

<a name="macro-arguments"></a>
#### Đối số Macro

Nếu cần thiết, bạn có thể định nghĩa các macro chấp nhận các đối số bổ sung:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Lang;

Collection::macro('toLocale', function (string $locale) {
    return $this->map(function (string $value) use ($locale) {
        return Lang::get($value, [], $locale);
    });
});

$collection = collect(['first', 'second']);

$translated = $collection->toLocale('es');

// ['primero', 'segundo'];
```

<a name="available-methods"></a>
## Các Phương thức Có sẵn

Đối với phần lớn tài liệu collection còn lại, chúng ta sẽ thảo luận về từng phương thức có sẵn trên lớp `Collection`. Hãy nhớ rằng, tất cả các phương thức này có thể được xâu chuỗi để thao tác trôi chảy trên mảng cơ bản. Hơn nữa, hầu như mọi phương thức đều trả về một thể hiện `Collection` mới, cho phép bạn giữ nguyên bản sao gốc của collection khi cần thiết:

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
</style>

<div class="collection-method-list" markdown="1">

[after](#method-after)
[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[before](#method-before)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collapseWithKeys](#method-collapsewithkeys)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffAssocUsing](#method-diffassocusing)
[diffKeys](#method-diffkeys)
[doesntContain](#method-doesntcontain)
[doesntContainStrict](#method-doesntcontainstrict)
[dot](#method-dot)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[ensure](#method-ensure)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[fromJson](#method-fromjson)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[hasAny](#method-hasany)
[hasMany](#method-hasmany)
[hasSole](#method-hassole)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectUsing](#method-intersectusing)
[intersectAssoc](#method-intersectAssoc)
[intersectAssocUsing](#method-intersectassocusing)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[lazy](#method-lazy)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[multiply](#method-multiply)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[percentage](#method-percentage)
[pipe](#method-pipe)
[pipeInto](#method-pipeinto)
[pipeThrough](#method-pipethrough)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[range](#method-range)
[reduce](#method-reduce)
[reduceSpread](#method-reduce-spread)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[select](#method-select)
[shift](#method-shift)
[shuffle](#method-shuffle)
[skip](#method-skip)
[skipUntil](#method-skipuntil)
[skipWhile](#method-skipwhile)
[slice](#method-slice)
[sliding](#method-sliding)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortDesc](#method-sortdesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[sortKeysUsing](#method-sortkeysusing)
[splice](#method-splice)
[split](#method-split)
[splitIn](#method-splitin)
[sum](#method-sum)
[take](#method-take)
[takeUntil](#method-takeuntil)
[takeWhile](#method-takewhile)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[toPrettyJson](#method-to-pretty-json)
[transform](#method-transform)
[undot](#method-undot)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[value](#method-value)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[whereNotNull](#method-wherenotnull)
[whereNull](#method-wherenull)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

<a name="method-listing"></a>
## Danh sách Phương thức

<style>
    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-after"></a>
#### `after()` {.collection-method .first-collection-method}

Phương thức `after` trả về phần tử sau phần tử đã cho. `null` được trả về nếu phần tử đã cho không được tìm thấy hoặc là phần tử cuối cùng:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->after(3);

// 4

$collection->after(5);

// null
```

Phương thức này tìm kiếm phần tử đã cho bằng cách sử dụng so sánh "lỏng lẻo", nghĩa là một chuỗi chứa giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị. Để sử dụng so sánh "nghiêm ngặt", bạn có thể cung cấp đối số `strict` cho phương thức:

```php
collect([2, 4, 6, 8])->after('4', strict: true);

// null
```

Ngoài ra, bạn có thể cung cấp closure của riêng mình để tìm kiếm phần tử đầu tiên vượt qua một bài kiểm tra truth đã cho:

```php
collect([2, 4, 6, 8])->after(function (int $item, int $key) {
    return $item > 5;
});

// 8
```

<a name="method-all"></a>
#### `all()` {.collection-method}

Phương thức `all` trả về mảng cơ bản được đại diện bởi collection:

```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

<a name="method-average"></a>
#### `average()` {.collection-method}

Bí danh cho phương thức [avg](#method-avg).

<a name="method-avg"></a>
#### `avg()` {.collection-method}

Phương thức `avg` trả về [giá trị trung bình](https://en.wikipedia.org/wiki/Average) của một khóa đã cho:

```php
$average = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->avg('foo');

// 20

$average = collect([1, 1, 2, 4])->avg();

// 2
```

<a name="method-before"></a>
#### `before()` {.collection-method}

Phương thức `before` là đối lập của phương thức [after](#method-after). Nó trả về phần tử trước phần tử đã cho. `null` được trả về nếu phần tử đã cho không được tìm thấy hoặc là phần tử đầu tiên:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->before(3);

// 2

$collection->before(1);

// null

collect([2, 4, 6, 8])->before('4', strict: true);

// null

collect([2, 4, 6, 8])->before(function (int $item, int $key) {
    return $item > 5;
});

// 4
```

<a name="method-chunk"></a>
#### `chunk()` {.collection-method}

Phương thức `chunk` chia collection thành nhiều collection nhỏ hơn với kích thước đã cho:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->all();

// [[1, 2, 3, 4], [5, 6, 7]]
```

Phương thức này đặc biệt hữu ích trong [views](/docs/{{version}}/views) khi làm việc với hệ thống lưới như [Bootstrap](https://getbootstrap.com/docs/5.3/layout/grid/). Ví dụ, hãy tưởng tượng bạn có một collection các mô hình [Eloquent](/docs/{{version}}/eloquent) mà bạn muốn hiển thị trong một lưới:

```blade
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

<a name="method-chunkwhile"></a>
#### `chunkWhile()` {.collection-method}

Phương thức `chunkWhile` chia collection thành nhiều collection nhỏ hơn dựa trên việc đánh giá callback đã cho. Biến `$chunk` được truyền vào closure có thể được sử dụng để kiểm tra phần tử trước đó:

```php
$collection = collect(str_split('AABBCCCD'));

$chunks = $collection->chunkWhile(function (string $value, int $key, Collection $chunk) {
    return $value === $chunk->last();
});

$chunks->all();

// [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]
```

<a name="method-collapse"></a>
#### `collapse()` {.collection-method}

Phương thức `collapse` thu gọn một collection của các mảng hoặc collections thành một collection duy nhất, phẳng:

```php
$collection = collect([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

<a name="method-collapsewithkeys"></a>
#### `collapseWithKeys()` {.collection-method}

Phương thức `collapseWithKeys` làm phẳng một collection của các mảng hoặc collections thành một collection duy nhất, giữ nguyên các khóa gốc. Nếu collection đã phẳng, nó sẽ trả về một collection rỗng:

```php
$collection = collect([
    ['first'  => collect([1, 2, 3])],
    ['second' => [4, 5, 6]],
    ['third'  => collect([7, 8, 9])]
]);

$collapsed = $collection->collapseWithKeys();

$collapsed->all();

// [
//     'first'  => [1, 2, 3],
//     'second' => [4, 5, 6],
//     'third'  => [7, 8, 9],
// ]
```

<a name="method-collect"></a>
#### `collect()` {.collection-method}

Phương thức `collect` trả về một thể hiện `Collection` mới với các phần tử hiện có trong collection:

```php
$collectionA = collect([1, 2, 3]);

$collectionB = $collectionA->collect();

$collectionB->all();

// [1, 2, 3]
```

Phương thức `collect` chủ yếu hữu ích để chuyển đổi [lazy collections](#lazy-collections) thành các thể hiện `Collection` tiêu chuẩn:

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$collection = $lazyCollection->collect();

$collection::class;

// 'Illuminate\Support\Collection'

$collection->all();

// [1, 2, 3]
```

> [!NOTE]
> Phương thức `collect` đặc biệt hữu ích khi bạn có một thể hiện của `Enumerable` và cần một thể hiện collection không lazy. Vì `collect()` là một phần của contract `Enumerable`, bạn có thể sử dụng nó một cách an toàn để lấy một thể hiện `Collection`.

<a name="method-combine"></a>
#### `combine()` {.collection-method}

Phương thức `combine` kết hợp các giá trị của collection, dưới dạng khóa, với các giá trị của một mảng hoặc collection khác:

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

<a name="method-concat"></a>
#### `concat()` {.collection-method}

Phương thức `concat` nối các giá trị của mảng hoặc collection đã cho vào cuối một collection khác:

```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```

Phương thức `concat` đánh chỉ mục lại các khóa theo số cho các phần tử được nối vào collection gốc. Để duy trì các khóa trong các collections kết hợp, hãy xem phương thức [merge](#method-merge).

<a name="method-contains"></a>
#### `contains()` {.collection-method}

Phương thức `contains` xác định xem collection có chứa một phần tử đã cho hay không. Bạn có thể truyền một closure cho phương thức `contains` để xác định xem một phần tử có tồn tại trong collection khớp với một bài kiểm tra truth đã given hay không:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function (int $value, int $key) {
    return $value > 5;
});

// false
```

Ngoài ra, bạn có thể truyền một chuỗi cho phương thức `contains` để xác định xem collection có chứa một giá trị phần tử đã cho hay không:

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

Bạn cũng có thể truyền một cặp khóa / giá trị cho phương thức `contains`, sẽ xác định xem cặp đã cho có tồn tại trong collection hay không:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

Phương thức `contains` sử dụng so sánh "lỏng lẻo" khi kiểm tra các giá trị phần tử, nghĩa là một chuỗi có giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị. Sử dụng phương thức [containsStrict](#method-containsstrict) để lọc bằng cách sử dụng so sánh "nghiêm ngặt".

Để đảo ngược của `contains`, hãy xem phương thức [doesntContain](#method-doesntcontain).

<a name="method-containsstrict"></a>
#### `containsStrict()` {.collection-method}

Phương thức này có cùng chữ ký với phương thức [contains](#method-contains); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

> [!NOTE]
> Hành vi của phương thức này được sửa đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-contains).

<a name="method-count"></a>
#### `count()` {.collection-method}

Phương thức `count` trả về tổng số phần tử trong collection:

```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

<a name="method-countBy"></a>
#### `countBy()` {.collection-method}

Phương thức `countBy` đếm số lần xuất hiện của các giá trị trong collection. Theo mặc định, phương thức này đếm số lần xuất hiện của mọi phần tử, cho phép bạn đếm một số "loại" phần tử nhất định trong collection:

```php
$collection = collect([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]
```

Bạn có thể truyền một closure cho phương thức `countBy` để đếm tất cả các phần tử theo một giá trị tùy chỉnh:

```php
$collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

$counted = $collection->countBy(function (string $email) {
    return substr(strrchr($email, '@'), 1);
});

$counted->all();

// ['gmail.com' => 2, 'yahoo.com' => 1]
```

<a name="method-crossjoin"></a>
#### `crossJoin()` {.collection-method}

Phương thức `crossJoin` cross join các giá trị của collection với các mảng hoặc collections đã cho, trả về một tích Descartes với tất cả các hoán vị có thể:

```php
$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

<a name="method-dd"></a>
#### `dd()` {.collection-method}

Phương thức `dd` dump các phần tử của collection và kết thúc thực thi của script:

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dd();

/*
    array:2 [
        0 => "John Doe"
        1 => "Jane Doe"
    ]
*/
```

Nếu bạn không muốn dừng thực thi script, hãy sử dụng phương thức [dump](#method-dump) thay thế.

<a name="method-diff"></a>
#### `diff()` {.collection-method}

Phương thức `diff` so sánh collection với một collection khác hoặc một mảng PHP `array` đơn giản dựa trên các giá trị của nó. Phương thức này sẽ trả về các giá trị trong collection gốc không có trong collection đã cho:

```php
$collection = collect([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

> [!NOTE]
> Hành vi của phương thức này được sửa đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-diff).

<a name="method-diffassoc"></a>
#### `diffAssoc()` {.collection-method}

Phương thức `diffAssoc` so sánh collection với một collection khác hoặc một mảng PHP `array` đơn giản dựa trên các khóa và giá trị của nó. Phương thức này sẽ trả về các cặp khóa / giá trị trong collection gốc không có trong collection đã cho:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6,
]);

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

<a name="method-diffassocusing"></a>
#### `diffAssocUsing()` {.collection-method}

Khác với `diffAssoc`, `diffAssocUsing` chấp nhận một hàm callback do người dùng cung cấp cho so sánh chỉ mục:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssocUsing([
    'Color' => 'yellow',
    'Type' => 'fruit',
    'Remain' => 3,
], 'strnatcasecmp');

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

Callback phải là một hàm so sánh trả về một số nguyên nhỏ hơn, bằng, hoặc lớn hơn không. Để biết thêm thông tin, hãy tham khảo tài liệu PHP về [array_diff_uassoc](https://www.php.net/array_diff_uassoc#refsect1-function.array-diff-uassoc-parameters), là hàm PHP mà phương thức `diffAssocUsing` sử dụng nội bộ.

<a name="method-diffkeys"></a>
#### `diffKeys()` {.collection-method}

Phương thức `diffKeys` so sánh collection với một collection khác hoặc một mảng PHP `array` đơn giản dựa trên các khóa của nó. Phương thức này sẽ trả về các cặp khóa / giá trị trong collection gốc không có trong collection đã cho:

```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);

$diff->all();

// ['one' => 10, 'three' => 30, 'five' => 50]
```

<a name="method-doesntcontain"></a>
#### `doesntContain()` {.collection-method}

Phương thức `doesntContain` xác định xem collection không chứa một phần tử đã given hay không. Bạn có thể truyền một closure cho phương thức `doesntContain` để xác định xem một phần tử không tồn tại trong collection khớp với một bài kiểm tra truth đã given hay không:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->doesntContain(function (int $value, int $key) {
    return $value < 5;
});

// false
```

Ngoài ra, bạn có thể truyền một chuỗi cho phương thức `doesntContain` để xác định xem collection không chứa một giá trị phần tử đã given hay không:

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->doesntContain('Table');

// true

$collection->doesntContain('Desk');

// false
```

Bạn cũng có thể truyền một cặp khóa / giá trị cho phương thức `doesntContain`, sẽ xác định xem cặp đã given không tồn tại trong collection hay không:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->doesntContain('product', 'Bookcase');

// true
```

Phương thức `doesntContain` sử dụng so sánh "lỏng lẻo" khi kiểm tra các giá trị phần tử, nghĩa là một chuỗi có giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị.

<a name="method-doesntcontainstrict"></a>
#### `doesntContainStrict()` {.collection-method}

Phương thức này có cùng chữ ký với phương thức [doesntContain](#method-doesntcontain); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-dot"></a>
#### `dot()` {.collection-method}

Phương thức `dot` làm phẳng một collection đa chiều thành một collection cấp đơn sử dụng ký hiệu "dot" để chỉ định độ sâu:

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$flattened = $collection->dot();

$flattened->all();

// ['products.desk.price' => 100]
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}

Phương thức `dump` dump các phần tử của collection:

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dump();

/*
    array:2 [
        0 => "John Doe"
        1 => "Jane Doe"
    ]
*/
```

Nếu bạn muốn dừng thực thi script sau khi dump collection, hãy sử dụng phương thức [dd](#method-dd) thay thế.

<a name="method-duplicates"></a>
#### `duplicates()` {.collection-method}

Phương thức `duplicates` truy xuất và trả về các giá trị trùng lặp từ collection:

```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);

$collection->duplicates();

// [2 => 'a', 4 => 'b']
```

Nếu collection chứa các mảng hoặc đối tượng, bạn có thể truyền khóa của các thuộc tính mà bạn muốn kiểm tra các giá trị trùng lặp:

```php
$employees = collect([
    ['email' => 'abigail@example.com', 'position' => 'Developer'],
    ['email' => 'james@example.com', 'position' => 'Designer'],
    ['email' => 'victoria@example.com', 'position' => 'Developer'],
]);

$employees->duplicates('position');

// [2 => 'Developer']
```

<a name="method-duplicatesstrict"></a>
#### `duplicatesStrict()` {.collection-method}

Phương thức này có cùng chữ ký với phương thức [duplicates](#method-duplicates); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".
#### `each()` {.collection-method}

Phương thức `each` lặp qua các mục trong collection và chuyển từng mục đến một closure:

```php
$collection = collect([1, 2, 3, 4]);

$collection->each(function (int $item, int $key) {
    // ...
});
```

Nếu bạn muốn dừng lặp qua các mục, bạn có thể trả về `false` từ closure của mình:

```php
$collection->each(function (int $item, int $key) {
    if (/* condition */) {
        return false;
    }
});
```

<a name="method-eachspread"></a>
#### `eachSpread()` {.collection-method}

Phương thức `eachSpread` lặp qua các mục của collection, chuyển từng giá trị mục lồng nhau vào callback đã cho:

```php
$collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

$collection->eachSpread(function (string $name, int $age) {
    // ...
});
```

Bạn có thể dừng lặp qua các mục bằng cách trả về `false` từ callback:

```php
$collection->eachSpread(function (string $name, int $age) {
    return false;
});
```

<a name="method-ensure"></a>
#### `ensure()` {.collection-method}

Phương thức `ensure` có thể được sử dụng để xác minh rằng tất cả các phần tử của một collection thuộc một kiểu hoặc danh sách kiểu đã cho. Nếu không, một `UnexpectedValueException` sẽ được ném ra:

```php
return $collection->ensure(User::class);

return $collection->ensure([User::class, Customer::class]);
```

Các kiểu nguyên thủy như `string`, `int`, `float`, `bool`, và `array` cũng có thể được chỉ định:

```php
return $collection->ensure('int');
```

> [!WARNING]
> Phương thức `ensure` không đảm bảo rằng các phần tử của các kiểu khác nhau sẽ không được thêm vào collection vào một thời điểm sau.

<a name="method-every"></a>
#### `every()` {.collection-method}

Phương thức `every` có thể được sử dụng để xác minh rằng tất cả các phần tử của một collection vượt qua một bài kiểm tra sự thật đã cho:

```php
collect([1, 2, 3, 4])->every(function (int $value, int $key) {
    return $value > 2;
});

// false
```

Nếu collection rỗng, phương thức `every` sẽ trả về true:

```php
$collection = collect([]);

$collection->every(function (int $value, int $key) {
    return $value > 2;
});

// true
```

<a name="method-except"></a>
#### `except()` {.collection-method}

Phương thức `except` trả về tất cả các mục trong collection ngoại trừ những mục có các khóa được chỉ định:

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

$filtered = $collection->except(['price', 'discount']);

$filtered->all();

// ['product_id' => 1]
```

Để xem nghịch đảo của `except`, hãy xem phương thức [only](#method-only).

> [!NOTE]
> Hành vi của phương thức này được sửa đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-except).

<a name="method-filter"></a>
#### `filter()` {.collection-method}

Phương thức `filter` lọc collection bằng cách sử dụng callback đã cho, chỉ giữ lại những mục vượt qua một bài kiểm tra sự thật đã cho:

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->filter(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [3, 4]
```

Nếu không có callback nào được cung cấp, tất cả các mục của collection tương đương với `false` sẽ bị xóa:

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);

$collection->filter()->all();

// [1, 2, 3]
```

Để xem nghịch đảo của `filter`, hãy xem phương thức [reject](#method-reject).

<a name="method-first"></a>
#### `first()` {.collection-method}

Phương thức `first` trả về phần tử đầu tiên trong collection vượt qua một bài kiểm tra sự thật đã cho:

```php
collect([1, 2, 3, 4])->first(function (int $value, int $key) {
    return $value > 2;
});

// 3
```

Bạn cũng có thể gọi phương thức `first` không có đối số để lấy phần tử đầu tiên trong collection. Nếu collection rỗng, `null` sẽ được trả về:

```php
collect([1, 2, 3, 4])->first();

// 1
```

<a name="method-first-or-fail"></a>
#### `firstOrFail()` {.collection-method}

Phương thức `firstOrFail` giống hệt với phương thức `first`; tuy nhiên, nếu không tìm thấy kết quả nào, một ngoại lệ `Illuminate\Support\ItemNotFoundException` sẽ được ném ra:

```php
collect([1, 2, 3, 4])->firstOrFail(function (int $value, int $key) {
    return $value > 5;
});

// Throws ItemNotFoundException...
```

Bạn cũng có thể gọi phương thức `firstOrFail` không có đối số để lấy phần tử đầu tiên trong collection. Nếu collection rỗng, một ngoại lệ `Illuminate\Support\ItemNotFoundException` sẽ được ném ra:

```php
collect([])->firstOrFail();

// Throws ItemNotFoundException...
```

<a name="method-first-where"></a>
#### `firstWhere()` {.collection-method}

Phương thức `firstWhere` trả về phần tử đầu tiên trong collection với cặp khóa / giá trị đã cho:

```php
$collection = collect([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);

$collection->firstWhere('name', 'Linda');

// ['name' => 'Linda', 'age' => 14]
```

Bạn cũng có thể gọi phương thức `firstWhere` với một toán tử so sánh:

```php
$collection->firstWhere('age', '>=', 18);

// ['name' => 'Diego', 'age' => 23]
```

Giống như phương thức [where](#method-where), bạn có thể chuyển một đối số cho phương thức `firstWhere`. Trong trường hợp này, phương thức `firstWhere` sẽ trả về mục đầu tiên mà giá trị của khóa mục đã cho là "truthy":

```php
$collection->firstWhere('age');

// ['name' => 'Linda', 'age' => 14]
```

<a name="method-flatmap"></a>
#### `flatMap()` {.collection-method}

Phương thức `flatMap` lặp qua collection và chuyển từng giá trị đến closure đã cho. Closure có thể tự do sửa đổi mục và trả về nó, do đó tạo thành một collection mới của các mục đã sửa đổi. Sau đó, mảng được làm phẳng một cấp:

```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);

$flattened = $collection->flatMap(function (array $values) {
    return array_map('strtoupper', $values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

<a name="method-flatten"></a>
#### `flatten()` {.collection-method}

Phương thức `flatten` làm phẳng một collection đa chiều thành một chiều duy nhất:

```php
$collection = collect([
    'name' => 'Taylor',
    'languages' => [
        'PHP', 'JavaScript'
    ]
]);

$flattened = $collection->flatten();

$flattened->all();

// ['Taylor', 'PHP', 'JavaScript'];
```

Nếu cần thiết, bạn có thể chuyển đối số "depth" cho phương thức `flatten`:

```php
$collection = collect([
    'Apple' => [
        [
            'name' => 'iPhone 6S',
            'brand' => 'Apple'
        ],
    ],
    'Samsung' => [
        [
            'name' => 'Galaxy S7',
            'brand' => 'Samsung'
        ],
    ],
]);

$products = $collection->flatten(1);

$products->values()->all();

/*
    [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
    ]
*/
```

Trong ví dụ này, việc gọi `flatten` mà không cung cấp depth cũng sẽ làm phẳng các mảng lồng nhau, dẫn đến `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Việc cung cấp depth cho phép bạn chỉ định số cấp mà các mảng lồng nhau sẽ được làm phẳng.

<a name="method-flip"></a>
#### `flip()` {.collection-method}

Phương thức `flip` hoán đổi các khóa của collection với các giá trị tương ứng của chúng:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$flipped = $collection->flip();

$flipped->all();

// ['Taylor' => 'name', 'Laravel' => 'framework']
```

<a name="method-forget"></a>
#### `forget()` {.collection-method}

Phương thức `forget` xóa một mục khỏi collection bằng khóa của nó:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

// Forget a single key...
$collection->forget('name');

// ['framework' => 'Laravel']

// Forget multiple keys...
$collection->forget(['name', 'framework']);

// []
```

> [!WARNING]
> Khác với hầu hết các phương thức collection khác, `forget` không trả về một collection đã sửa đổi mới; nó sửa đổi và trả về collection mà nó được gọi trên đó.

<a name="method-forpage"></a>
#### `forPage()` {.collection-method}

Phương thức `forPage` trả về một collection mới chứa các mục sẽ có mặt trên một số trang đã cho. Phương thức chấp nhận số trang làm đối số đầu tiên và số lượng mục để hiển thị trên mỗi trang làm đối số thứ hai:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunk = $collection->forPage(2, 3);

$chunk->all();

// [4, 5, 6]
```

<a name="method-fromjson"></a>
#### `fromJson()` {.collection-method}

Phương thức tĩnh `fromJson` tạo một thể hiện collection mới bằng cách giải mã một chuỗi JSON đã cho bằng hàm PHP `json_decode`:

```php
use Illuminate\Support\Collection;

$json = json_encode([
    'name' => 'Taylor Otwell',
    'role' => 'Developer',
    'status' => 'Active',
]);

$collection = Collection::fromJson($json);
```

<a name="method-get"></a>
#### `get()` {.collection-method}

Phương thức `get` trả về mục tại một khóa đã cho. Nếu khóa không tồn tại, `null` sẽ được trả về:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$value = $collection->get('name');

// Taylor
```

Bạn có thể tùy chọn chuyển một giá trị mặc định làm đối số thứ hai:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$value = $collection->get('age', 34);

// 34
```

Bạn thậm chí có thể chuyển một callback làm giá trị mặc định của phương thức. Kết quả của callback sẽ được trả về nếu khóa được chỉ định không tồn tại:

```php
$collection->get('email', function () {
    return 'taylor@example.com';
});

// taylor@example.com
```

<a name="method-groupby"></a>
#### `groupBy()` {.collection-method}

Phương thức `groupBy` nhóm các mục của collection theo một khóa đã cho:

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->all();

/*
    [
        'account-x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'account-x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

Thay vì chuyển một chuỗi `key`, bạn có thể chuyển một callback. Callback nên trả về giá trị mà bạn muốn dùng làm khóa để nhóm:

```php
$grouped = $collection->groupBy(function (array $item, int $key) {
    return substr($item['account_id'], -3);
});

$grouped->all();

/*
    [
        'x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

Nhiều tiêu chí nhóm có thể được chuyển dưới dạng một mảng. Mỗi phần tử mảng sẽ được áp dụng cho cấp tương ứng trong một mảng đa chiều:

```php
$data = new Collection([
    10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
    20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
    30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
    40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
]);

$result = $data->groupBy(['skill', function (array $item) {
    return $item['roles'];
}], preserveKeys: true);

/*
[
    1 => [
        'Role_1' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_2' => [
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_3' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        ],
    ],
    2 => [
        'Role_1' => [
            30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        ],
        'Role_2' => [
            40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
        ],
    ],
];
*/
```

<a name="method-has"></a>
#### `has()` {.collection-method}

Phương thức `has` xác định xem một khóa đã cho có tồn tại trong collection hay không:

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

$collection->has('product');

// true

$collection->has(['product', 'amount']);

// true

$collection->has(['amount', 'price']);

// false
```

<a name="method-hasany"></a>
#### `hasAny()` {.collection-method}

Phương thức `hasAny` xác định xem bất kỳ khóa nào đã cho có tồn tại trong collection hay không:

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

$collection->hasAny(['product', 'price']);

// true

$collection->hasAny(['name', 'price']);

// false
```

<a name="method-hasmany"></a>
#### `hasMany()` {.collection-method}

Phương thức `hasMany` xác định xem collection có chứa nhiều mục hay không:

```php
collect([])->hasMany();

// false

collect(['1'])->hasMany();

// false

collect([1, 2, 3])->hasMany();

// true

collect([
    ['age' => 2],
    ['age' => 3],
])->hasMany(fn ($item) => $item['age'] === 2)

// false
```

<a name="method-hassole"></a>
#### `hasSole()` {.collection-method}

Phương thức `hasSole` xác định xem collection có chứa một mục duy nhất hay không, tùy chọn khớp với tiêu chí đã cho:

```php
collect([])->hasSole();

// false

collect(['1'])->hasSole();

// true

collect([1, 2, 3])->hasSole(fn (int $item) => $item === 2);

// true
```

<a name="method-implode"></a>
#### `implode()` {.collection-method}

Phương thức `implode` nối các mục trong một collection. Các đối số của nó phụ thuộc vào loại mục trong collection. Nếu collection chứa mảng hoặc đối tượng, bạn nên chuyển khóa của các thuộc tính mà bạn muốn nối, và chuỗi "glue" mà bạn muốn đặt giữa các giá trị:

```php
$collection = collect([
    ['account_id' => 1, 'product' => 'Desk'],
    ['account_id' => 2, 'product' => 'Chair'],
]);

$collection->implode('product', ', ');

// 'Desk, Chair'
```

Nếu collection chứa các chuỗi đơn giản hoặc giá trị số, bạn nên chuyển "glue" làm đối số duy nhất cho phương thức:

```php
collect([1, 2, 3, 4, 5])->implode('-');

// '1-2-3-4-5'
```

Bạn có thể chuyển một closure cho phương thức `implode` nếu bạn muốn định dạng các giá trị đang được nối:

```php
$collection->implode(function (array $item, int $key) {
    return strtoupper($item['product']);
}, ', ');

// 'DESK, CHAIR'
```

<a name="method-intersect"></a>
#### `intersect()` {.collection-method}

Phương thức `intersect` xóa bất kỳ giá trị nào khỏi collection gốc không có trong mảng hoặc collection đã cho. Collection kết quả sẽ giữ lại các khóa của collection gốc:

```php
$collection = collect(['Desk', 'Sofa', 'Chair']);

$intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

$intersect->all();

// [0 => 'Desk', 2 => 'Chair']
```

> [!NOTE]
> Hành vi của phương thức này được sửa đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-intersect).

<a name="method-intersectusing"></a>
#### `intersectUsing()` {.collection-method}

Phương thức `intersectUsing` xóa bất kỳ giá trị nào khỏi collection gốc không có trong mảng hoặc collection đã cho, sử dụng một callback tùy chỉnh để so sánh các giá trị. Collection kết quả sẽ giữ lại các khóa của collection gốc:

```php
$collection = collect(['Desk', 'Sofa', 'Chair']);

$intersect = $collection->intersectUsing(['desk', 'chair', 'bookcase'], function (string $a, string $b) {
    return strcasecmp($a, $b);
});

$intersect->all();

// [0 => 'Desk', 2 => 'Chair']
```

<a name="method-intersectAssoc"></a>
#### `intersectAssoc()` {.collection-method}

Phương thức `intersectAssoc` so sánh collection gốc với một collection hoặc mảng khác, trả về các cặp khóa / giá trị có trong tất cả các collection đã cho:

```php
$collection = collect([
    'color' => 'red',
    'size' => 'M',
    'material' => 'cotton'
]);

$intersect = $collection->intersectAssoc([
    'color' => 'blue',
    'size' => 'M',
    'material' => 'polyester'
]);

$intersect->all();

// ['size' => 'M']
```

<a name="method-intersectassocusing"></a>
#### `intersectAssocUsing()` {.collection-method}

Phương thức `intersectAssocUsing` so sánh collection gốc với một collection hoặc mảng khác, trả về các cặp khóa / giá trị có trong cả hai, sử dụng một callback so sánh tùy chỉnh để xác định sự bình đẳng cho cả khóa và giá trị:

```php
$collection = collect([
    'color' => 'red',
    'Size' => 'M',
    'material' => 'cotton',
]);

$intersect = $collection->intersectAssocUsing([
    'color' => 'blue',
    'size' => 'M',
    'material' => 'polyester',
], function (string $a, string $b) {
    return strcasecmp($a, $b);
});

$intersect->all();

// ['Size' => 'M']
```

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {.collection-method}

Phương thức `intersectByKeys` xóa bất kỳ khóa nào và các giá trị tương ứng của chúng khỏi collection gốc không có trong mảng hoặc collection đã cho:

```php
$collection = collect([
    'serial' => 'UX301', 'type' => 'screen', 'year' => 2009,
]);

$intersect = $collection->intersectByKeys([
    'reference' => 'UX404', 'type' => 'tab', 'year' => 2011,
]);

$intersect->all();

// ['type' => 'screen', 'year' => 2009]
```

<a name="method-isempty"></a>
#### `isEmpty()` {.collection-method}

Phương thức `isEmpty` trả về `true` nếu collection rỗng; nếu không, `false` sẽ được trả về:

```php
collect([])->isEmpty();

// true
```

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {.collection-method}

Phương thức `isNotEmpty` trả về `true` nếu collection không rỗng; nếu không, `false` sẽ được trả về:

```php
collect([])->isNotEmpty();

// false
```

<a name="method-join"></a>
#### `join()` {.collection-method}

Phương thức `join` nối các giá trị của collection với một chuỗi. Sử dụng đối số thứ hai của phương thức này, bạn cũng có thể chỉ định cách phần tử cuối cùng nên được nối vào chuỗi:

```php
collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
collect(['a'])->join(', ', ' and '); // 'a'
collect([])->join(', ', ' and '); // ''
```

<a name="method-keyby"></a>
#### `keyBy()` {.collection-method}

Phương thức `keyBy` khóa collection theo khóa đã cho. Nếu nhiều mục có cùng khóa, chỉ mục cuối cùng sẽ xuất hiện trong collection mới:

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

Bạn cũng có thể chuyển một callback cho phương thức. Callback nên trả về giá trị để khóa collection theo:

```php
$keyed = $collection->keyBy(function (array $item, int $key) {
    return strtoupper($item['product_id']);
});

$keyed->all();

/*
    [
        'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

<a name="method-keys"></a>
#### `keys()` {.collection-method}

Phương thức `keys` trả về tất cả các khóa của collection:

```php
$collection = collect([
    'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
    'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keys = $collection->keys();

$keys->all();

// ['prod-100', 'prod-200']
```

<a name="method-last"></a>
#### `last()` {.collection-method}

Phương thức `last` trả về phần tử cuối cùng trong collection vượt qua một bài kiểm tra sự thật đã cho:

```php
collect([1, 2, 3, 4])->last(function (int $value, int $key) {
    return $value < 3;
});

// 2
```

Bạn cũng có thể gọi phương thức `last` không có đối số để lấy phần tử cuối cùng trong collection. Nếu collection rỗng, `null` sẽ được trả về:

```php
collect([1, 2, 3, 4])->last();

// 4
```

<a name="method-lazy"></a>
#### `lazy()` {.collection-method}

Phương thức `lazy` trả về một thể hiện [LazyCollection](#lazy-collections) mới từ mảng cơ bản của các mục:

```php
$lazyCollection = collect([1, 2, 3, 4])->lazy();

$lazyCollection::class;

// Illuminate\Support\LazyCollection

$lazyCollection->all();

// [1, 2, 3, 4]
```

Điều này đặc biệt hữu ích khi bạn cần thực hiện các chuyển đổi trên một `Collection` khổng lồ chứa nhiều mục:

```php
$count = $hugeCollection
    ->lazy()
    ->where('country', 'FR')
    ->where('balance', '>', '100')
    ->count();
```

Bằng cách chuyển đổi collection thành một `LazyCollection`, chúng ta tránh phải phân bổ một lượng lớn bộ nhớ bổ sung. Mặc dù collection gốc vẫn giữ _các_ giá trị của nó trong bộ nhớ, các bộ lọc tiếp theo sẽ không. Do đó, hầu như không có bộ nhớ bổ sung nào được phân bổ khi lọc kết quả của collection.

<a name="method-macro"></a>
#### `macro()` {.collection-method}

Phương thức tĩnh `macro` cho phép bạn thêm phương thức vào lớp `Collection` tại thời điểm chạy. Tham khảo tài liệu về [mở rộng collections](#extending-collections) để biết thêm thông tin.

<a name="method-make"></a>
#### `make()` {.collection-method}

Phương thức tĩnh `make` tạo một thể hiện collection mới. Xem phần [Tạo Collections](#creating-collections).

```php
use Illuminate\Support\Collection;

$collection = Collection::make([1, 2, 3]);
```

<a name="method-map"></a>
#### `map()` {.collection-method}

Phương thức `map` lặp qua collection và chuyển từng giá trị đến callback đã cho. Callback có thể tự do sửa đổi mục và trả về nó, do đó tạo thành một collection mới của các mục đã sửa đổi:

```php
$collection = collect([1, 2, 3, 4, 5]);

$multiplied = $collection->map(function (int $item, int $key) {
    return $item * 2;
});

$multiplied->all();

// [2, 4, 6, 8, 10]
```

> [!WARNING]
> Giống như hầu hết các phương thức collection khác, `map` trả về một thể hiện collection mới; nó không sửa đổi collection mà nó được gọi trên đó. Nếu bạn muốn chuyển đổi collection gốc, hãy sử dụng phương thức [transform](#method-transform).

<a name="method-mapinto"></a>
#### `mapInto()` {.collection-method}

Phương thức `mapInto()` lặp qua collection, tạo một thể hiện mới của lớp đã cho bằng cách chuyển giá trị vào constructor:

```php
class Currency
{
    /**
     * Create a new currency instance.
     */
    function __construct(
        public string $code,
    ) {}
}

$collection = collect(['USD', 'EUR', 'GBP']);

$currencies = $collection->mapInto(Currency::class);

$currencies->all();

// [Currency('USD'), Currency('EUR'), Currency('GBP')]
```
```

<a name="method-mapspread"></a>
#### `mapSpread()` {.collection-method}

Phương thức `mapSpread` lặp qua các mục của collection, truyền từng giá trị mục lồng nhau vào closure đã cho. Closure có thể tự do sửa đổi mục và trả về nó, từ đó tạo thành một collection mới của các mục đã được sửa đổi:

```php
$collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunks = $collection->chunk(2);

$sequence = $chunks->mapSpread(function (int $even, int $odd) {
    return $even + $odd;
});

$sequence->all();

// [1, 5, 9, 13, 17]
```

<a name="method-maptogroups"></a>
#### `mapToGroups()` {.collection-method}

Phương thức `mapToGroups` nhóm các mục của collection theo closure đã cho. Closure nên trả về một mảng kết hợp chứa một cặp key/value duy nhất, từ đó tạo thành một collection mới của các giá trị đã được nhóm:

```php
$collection = collect([
    [
        'name' => 'John Doe',
        'department' => 'Sales',
    ],
    [
        'name' => 'Jane Doe',
        'department' => 'Sales',
    ],
    [
        'name' => 'Johnny Doe',
        'department' => 'Marketing',
    ]
]);

$grouped = $collection->mapToGroups(function (array $item, int $key) {
    return [$item['department'] => $item['name']];
});

$grouped->all();

/*
    [
        'Sales' => ['John Doe', 'Jane Doe'],
        'Marketing' => ['Johnny Doe'],
    ]
*/

$grouped->get('Sales')->all();

// ['John Doe', 'Jane Doe']
```

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {.collection-method}

Phương thức `mapWithKeys` lặp qua collection và truyền từng giá trị vào callback đã cho. Callback nên trả về một mảng kết hợp chứa một cặp key/value duy nhất:

```php
$collection = collect([
    [
        'name' => 'John',
        'department' => 'Sales',
        'email' => 'john@example.com',
    ],
    [
        'name' => 'Jane',
        'department' => 'Marketing',
        'email' => 'jane@example.com',
    ]
]);

$keyed = $collection->mapWithKeys(function (array $item, int $key) {
    return [$item['email'] => $item['name']];
});

$keyed->all();

/*
    [
        'john@example.com' => 'John',
        'jane@example.com' => 'Jane',
    ]
*/
```

<a name="method-max"></a>
#### `max()` {.collection-method}

Phương thức `max` trả về giá trị lớn nhất của một key đã cho:

```php
$max = collect([
    ['foo' => 10],
    ['foo' => 20]
])->max('foo');

// 20

$max = collect([1, 2, 3, 4, 5])->max();

// 5
```

<a name="method-median"></a>
#### `median()` {.collection-method}

Phương thức `median` trả về [giá trị trung vị](https://en.wikipedia.org/wiki/Median) của một key đã cho:

```php
$median = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->median('foo');

// 15

$median = collect([1, 1, 2, 4])->median();

// 1.5
```

<a name="method-merge"></a>
#### `merge()` {.collection-method}

Phương thức `merge` gộp mảng hoặc collection đã cho với collection gốc. Nếu một key dạng chuỗi trong các mục đã cho khớp với một key dạng chuỗi trong collection gốc, giá trị của mục đã cho sẽ ghi đè giá trị trong collection gốc:

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->merge(['price' => 200, 'discount' => false]);

$merged->all();

// ['product_id' => 1, 'price' => 200, 'discount' => false]
```

Nếu các key của mục đã cho là số, các giá trị sẽ được thêm vào cuối collection:

```php
$collection = collect(['Desk', 'Chair']);

$merged = $collection->merge(['Bookcase', 'Door']);

$merged->all();

// ['Desk', 'Chair', 'Bookcase', 'Door']
```

<a name="method-mergerecursive"></a>
#### `mergeRecursive()` {.collection-method}

Phương thức `mergeRecursive` gộp đệ quy mảng hoặc collection đã cho với collection gốc. Nếu một key dạng chuột trong các mục đã cho khớp với một key dạng chuỗi trong collection gốc, thì các giá trị cho các key này sẽ được gộp với nhau thành một mảng, và việc này được thực hiện đệ quy:

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->mergeRecursive([
    'product_id' => 2,
    'price' => 200,
    'discount' => false
]);

$merged->all();

// ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]
```

<a name="method-min"></a>
#### `min()` {.collection-method}

Phương thức `min` trả về giá trị nhỏ nhất của một key đã cho:

```php
$min = collect([
    ['foo' => 10],
    ['foo' => 20]
])->min('foo');

// 10

$min = collect([1, 2, 3, 4, 5])->min();

// 1
```

<a name="method-mode"></a>
#### `mode()` {.collection-method}

Phương thức `mode` trả về [giá trị mode](https://en.wikipedia.org/wiki/Mode_(statistics)) của một key đã cho:

```php
$mode = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->mode('foo');

// [10]

$mode = collect([1, 1, 2, 4])->mode();

// [1]

$mode = collect([1, 1, 2, 2])->mode();

// [1, 2]
```

<a name="method-multiply"></a>
#### `multiply()` {.collection-method}

Phương thức `multiply` tạo số lượng bản sao được chỉ định của tất cả các mục trong collection:

```php
$users = collect([
    ['name' => 'User #1', 'email' => 'user1@example.com'],
    ['name' => 'User #2', 'email' => 'user2@example.com'],
])->multiply(3);

/*
    [
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
    ]
*/
```

<a name="method-nth"></a>
#### `nth()` {.collection-method}

Phương thức `nth` tạo một collection mới bao gồm mỗi phần tử thứ n:

```php
$collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->nth(4);

// ['a', 'e']
```

Bạn có thể tùy chọn truyền một offset bắt đầu làm đối số thứ hai:

```php
$collection->nth(4, 1);

// ['b', 'f']
```

<a name="method-only"></a>
#### `only()` {.collection-method}

Phương thức `only` trả về các mục trong collection với các key được chỉ định:

```php
$collection = collect([
    'product_id' => 1,
    'name' => 'Desk',
    'price' => 100,
    'discount' => false
]);

$filtered = $collection->only(['product_id', 'name']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

Để xem phương thức ngược lại của `only`, hãy xem phương thức [except](#method-except).

> [!NOTE]
> Hành vi của phương thức này được sửa đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-only).

<a name="method-pad"></a>
#### `pad()` {.collection-method}

Phương thức `pad` sẽ điền mảng với giá trị đã cho cho đến khi mảng đạt kích thước được chỉ định. Phương thức này hoạt động giống như hàm PHP [array_pad](https://secure.php.net/manual/en/function.array-pad.php).

Để đệm sang trái, bạn nên chỉ định kích thước âm. Không có đệm nào sẽ diễn ra nếu giá trị tuyệt đối của kích thước đã cho nhỏ hơn hoặc bằng độ dài của mảng:

```php
$collection = collect(['A', 'B', 'C']);

$filtered = $collection->pad(5, 0);

$filtered->all();

// ['A', 'B', 'C', 0, 0]

$filtered = $collection->pad(-5, 0);

$filtered->all();

// [0, 0, 'A', 'B', 'C']
```

<a name="method-partition"></a>
#### `partition()` {.collection-method}

Phương thức `partition` có thể được kết hợp với destructuring mảng PHP để tách các phần tử vượt qua một bài kiểm tra truth đã cho với những phần tử không vượt qua:

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

[$underThree, $equalOrAboveThree] = $collection->partition(function (int $i) {
    return $i < 3;
});

$underThree->all();

// [1, 2]

$equalOrAboveThree->all();

// [3, 4, 5, 6]
```

> [!NOTE]
> Hành vi của phương thức này được sửa đổi khi tương tác với [Eloquent collections](/docs/{{version}}/eloquent-collections#method-partition).

<a name="method-percentage"></a>
#### `percentage()` {.collection-method}

Phương thức `percentage` có thể được sử dụng để nhanh chóng xác định phần trăm các mục trong collection vượt qua một bài kiểm tra truth đã cho:

```php
$collection = collect([1, 1, 2, 2, 2, 3]);

$percentage = $collection->percentage(fn (int $value) => $value === 1);

// 33.33
```

Theo mặc định, phần trăm sẽ được làm tròn đến hai chữ số thập phân. Tuy nhiên, bạn có thể tùy chỉnh hành vi này bằng cách cung cấp đối số thứ hai cho phương thức:

```php
$percentage = $collection->percentage(fn (int $value) => $value === 1, precision: 3);

// 33.333
```

<a name="method-pipe"></a>
#### `pipe()` {.collection-method}

Phương thức `pipe` truyền collection vào closure đã cho và trả về kết quả của closure đã thực thi:

```php
$collection = collect([1, 2, 3]);

$piped = $collection->pipe(function (Collection $collection) {
    return $collection->sum();
});

// 6
```

<a name="method-pipeinto"></a>
#### `pipeInto()` {.collection-method}

Phương thức `pipeInto` tạo một instance mới của class đã cho và truyền collection vào constructor:

```php
class ResourceCollection
{
    /**
     * Create a new ResourceCollection instance.
     */
    public function __construct(
        public Collection $collection,
    ) {}
}

$collection = collect([1, 2, 3]);

$resource = $collection->pipeInto(ResourceCollection::class);

$resource->collection->all();

// [1, 2, 3]
```

<a name="method-pipethrough"></a>
#### `pipeThrough()` {.collection-method}

Phương thức `pipeThrough` truyền collection vào mảng các closure đã cho và trả về kết quả của các closure đã thực thi:

```php
use Illuminate\Support\Collection;

$collection = collect([1, 2, 3]);

$result = $collection->pipeThrough([
    function (Collection $collection) {
        return $collection->merge([4, 5]);
    },
    function (Collection $collection) {
        return $collection->sum();
    },
]);

// 15
```

<a name="method-pluck"></a>
#### `pluck()` {.collection-method}

Phương thức `pluck` truy xuất tất cả các giá trị cho một key đã cho:

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name');

$plucked->all();

// ['Desk', 'Chair']
```

Bạn cũng có thể chỉ định cách bạn muốn collection kết quả được key:

```php
$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```

Phương thức `pluck` cũng hỗ trợ truy xuất các giá trị lồng nhau sử dụng ký hiệu "dot":

```php
$collection = collect([
    [
        'name' => 'Laracon',
        'speakers' => [
            'first_day' => ['Rosa', 'Judith'],
        ],
    ],
    [
        'name' => 'VueConf',
        'speakers' => [
            'first_day' => ['Abigail', 'Joey'],
        ],
    ],
]);

$plucked = $collection->pluck('speakers.first_day');

$plucked->all();

// [['Rosa', 'Judith'], ['Abigail', 'Joey']]
```

Nếu key trùng lặp tồn tại, phần tử khớp cuối cùng sẽ được chèn vào collection đã pluck:

```php
$collection = collect([
    ['brand' => 'Tesla',  'color' => 'red'],
    ['brand' => 'Pagani', 'color' => 'white'],
    ['brand' => 'Tesla',  'color' => 'black'],
    ['brand' => 'Pagani', 'color' => 'orange'],
]);

$plucked = $collection->pluck('color', 'brand');

$plucked->all();

// ['Tesla' => 'black', 'Pagani' => 'orange']
```

<a name="method-pop"></a>
#### `pop()` {.collection-method}

Phương thức `pop` xóa và trả về mục cuối cùng từ collection. Nếu collection rỗng, `null` sẽ được trả về:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop();

// 5

$collection->all();

// [1, 2, 3, 4]
```

Bạn có thể truyền một số nguyên vào phương thức `pop` để xóa và trả về nhiều mục từ cuối của một collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop(3);

// collect([5, 4, 3])

$collection->all();

// [1, 2]
```

<a name="method-prepend"></a>
#### `prepend()` {.collection-method}

Phương thức `prepend` thêm một mục vào đầu collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->prepend(0);

$collection->all();

// [0, 1, 2, 3, 4, 5]
```

Bạn cũng có thể truyền đối số thứ hai để chỉ định key của mục được thêm vào đầu:

```php
$collection = collect(['one' => 1, 'two' => 2]);

$collection->prepend(0, 'zero');

$collection->all();

// ['zero' => 0, 'one' => 1, 'two' => 2]
```

<a name="method-pull"></a>
#### `pull()` {.collection-method}

Phương thức `pull` xóa và trả về một mục từ collection theo key của nó:

```php
$collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

$collection->pull('name');

// 'Desk'

$collection->all();

// ['product_id' => 'prod-100']
```

<a name="method-push"></a>
#### `push()` {.collection-method}

Phương thức `push` thêm một mục vào cuối collection:

```php
$collection = collect([1, 2, 3, 4]);

$collection->push(5);

$collection->all();

// [1, 2, 3, 4, 5]
```

Bạn cũng có thể cung cấp nhiều mục để thêm vào cuối collection:

```php
$collection = collect([1, 2, 3, 4]);

$collection->push(5, 6, 7);

$collection->all();

// [1, 2, 3, 4, 5, 6, 7]
```

<a name="method-put"></a>
#### `put()` {.collection-method}

Phương thức `put` đặt key và giá trị đã cho trong collection:

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk']);

$collection->put('price', 100);

$collection->all();

// ['product_id' => 1, 'name' => 'Desk', 'price' => 100]
```

<a name="method-random"></a>
#### `random()` {.collection-method}

Phương thức `random` trả về một mục ngẫu nhiên từ collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->random();

// 4 - (retrieved randomly)
```

Bạn có thể truyền một số nguyên vào `random` để chỉ định bao nhiêu mục bạn muốn truy xuất ngẫu nhiên. Một collection của các mục luôn được trả về khi truyền rõ ràng số lượng mục bạn muốn nhận:

```php
$random = $collection->random(3);

$random->all();

// [2, 4, 5] - (retrieved randomly)
```

Nếu instance collection có ít mục hơn được yêu cầu, phương thức `random` sẽ ném một `InvalidArgumentException`.

Phương thức `random` cũng chấp nhận một closure, closure này sẽ nhận instance collection hiện tại:

```php
use Illuminate\Support\Collection;

$random = $collection->random(fn (Collection $items) => min(10, count($items)));

$random->all();

// [1, 2, 3, 4, 5] - (retrieved randomly)
```

<a name="method-range"></a>
#### `range()` {.collection-method}

Phương thức `range` trả về một collection chứa các số nguyên trong phạm vi được chỉ định:

```php
$collection = collect()->range(3, 6);

$collection->all();

// [3, 4, 5, 6]
```

<a name="method-reduce"></a>
#### `reduce()` {.collection-method}

Phương thức `reduce` giảm collection xuống một giá trị duy nhất, truyền kết quả của mỗi lần lặp vào lần lặp tiếp theo:

```php
$collection = collect([1, 2, 3]);

$total = $collection->reduce(function (?int $carry, int $item) {
    return $carry + $item;
});

// 6
```

Giá trị cho `$carry` trong lần lặp đầu tiên là `null`; tuy nhiên, bạn có thể chỉ định giá trị ban đầu của nó bằng cách truyền đối số thứ hai vào `reduce`:

```php
$collection->reduce(function (int $carry, int $item) {
    return $carry + $item;
}, 4);

// 10
```

Phương thức `reduce` cũng truyền các key mảng vào callback đã cho:

```php
$collection = collect([
    'usd' => 1400,
    'gbp' => 1200,
    'eur' => 1000,
]);

$ratio = [
    'usd' => 1,
    'gbp' => 1.37,
    'eur' => 1.22,
];

$collection->reduce(function (int $carry, int $value, string $key) use ($ratio) {
    return $carry + ($value * $ratio[$key]);
}, 0);

// 4264
```

<a name="method-reduce-spread"></a>
#### `reduceSpread()` {.collection-method}

Phương thức `reduceSpread` giảm collection xuống một mảng các giá trị, truyền kết quả của mỗi lần lặp vào lần lặp tiếp theo. Phương thức này tương tự như phương thức `reduce`; tuy nhiên, nó có thể chấp nhận nhiều giá trị ban đầu:

```php
[$creditsRemaining, $batch] = Image::where('status', 'unprocessed')
    ->get()
    ->reduceSpread(function (int $creditsRemaining, Collection $batch, Image $image) {
        if ($creditsRemaining >= $image->creditsRequired()) {
            $batch->push($image);

            $creditsRemaining -= $image->creditsRequired();
        }

        return [$creditsRemaining, $batch];
    }, $creditsAvailable, collect());
```

<a name="method-reject"></a>
#### `reject()` {.collection-method}

Phương thức `reject` lọc collection sử dụng closure đã cho. Closure nên trả về `true` nếu mục nên được xóa khỏi collection kết quả:

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->reject(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [1, 2]
```

Để xem phương thức ngược lại của phương thức `reject`, hãy xem phương thức [filter](#method-filter).

<a name="method-replace"></a>
#### `replace()` {.collection-method}

Phương thức `replace` hoạt động tương tự như `merge`; tuy nhiên, ngoài việc ghi đè các mục khớp có key dạng chuỗi, phương thức `replace` cũng sẽ ghi đè các mục trong collection có key dạng số khớp:

```php
$collection = collect(['Taylor', 'Abigail', 'James']);

$replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

$replaced->all();

// ['Taylor', 'Victoria', 'James', 'Finn']
```

<a name="method-replacerecursive"></a>
#### `replaceRecursive()` {.collection-method}

Phương thức `replaceRecursive` hoạt động tương tự như `replace`, nhưng nó sẽ đệ quy vào các mảng và áp dụng cùng một quy trình thay thế cho các giá trị bên trong:

```php
$collection = collect([
    'Taylor',
    'Abigail',
    [
        'James',
        'Victoria',
        'Finn'
    ]
]);

$replaced = $collection->replaceRecursive([
    'Charlie',
    2 => [1 => 'King']
]);

$replaced->all();

// ['Charlie', 'Abigail', ['James', 'King', 'Finn']]
```

<a name="method-reverse"></a>
#### `reverse()` {.collection-method}

Phương thức `reverse` đảo ngược thứ tự của các mục trong collection, giữ nguyên các key gốc:

```php
$collection = collect(['a', 'b', 'c', 'd', 'e']);

$reversed = $collection->reverse();

$reversed->all();

/*
    [
        4 => 'e',
        3 => 'd',
        2 => 'c',
        1 => 'b',
        0 => 'a',
    ]
*/
```

<a name="method-search"></a>
#### `search()` {.collection-method}

Phương thức `search` tìm kiếm collection cho giá trị đã cho và trả về key của nó nếu tìm thấy. Nếu mục không được tìm thấy, `false` sẽ được trả về:

```php
$collection = collect([2, 4, 6, 8]);

$collection->search(4);

// 1
```

Việc tìm kiếm được thực hiện sử dụng so sánh "loose", nghĩa là một chuỗi với giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị. Để sử dụng so sánh "strict", truyền `true` làm đối số thứ hai cho phương thức:

```php
collect([2, 4, 6, 8])->search('4', strict: true);

// false
```

Ngoài ra, bạn có thể cung cấp closure của riêng mình để tìm kiếm mục đầu tiên vượt qua một bài kiểm tra truth đã cho:

```php
collect([2, 4, 6, 8])->search(function (int $item, int $key) {
    return $item > 5;
});

// 2
```

<a name="method-select"></a>
#### `select()` {.collection-method}

Phương thức `select` chọn các key đã cho từ collection, tương tự như câu lệnh SQL `SELECT`:

```php
$users = collect([
    ['name' => 'Taylor Otwell', 'role' => 'Developer', 'status' => 'active'],
    ['name' => 'Victoria Faith', 'role' => 'Researcher', 'status' => 'active'],
]);

$users->select(['name', 'role']);

/*
    [
        ['name' => 'Taylor Otwell', 'role' => 'Developer'],
        ['name' => 'Victoria Faith', 'role' => 'Researcher'],
    ],
*/
```

<a name="method-shift"></a>
#### `shift()` {.collection-method}

Phương thức `shift` xóa và trả về mục đầu tiên từ collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->shift();

// 1

$collection->all();

// [2, 3, 4, 5]
```

Bạn có thể truyền một số nguyên vào phương thức `shift` để xóa và trả về nhiều mục từ đầu của một collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->shift(3);

// collect([1, 2, 3])

$collection->all();

// [4, 5]
```

<a name="method-shuffle"></a>
#### `shuffle()` {.collection-method}

Phương thức `shuffle` xáo trộn ngẫu nhiên các mục trong collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$shuffled = $collection->shuffle();

$shuffled->all();

// [3, 2, 5, 1, 4] - (generated randomly)
```
```

<a name="method-skip"></a>
#### `skip()` {.collection-method}

Phương thức `skip` trả về một collection mới, với số lượng phần tử được chỉ định được loại bỏ từ đầu collection:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$collection = $collection->skip(4);

$collection->all();

// [5, 6, 7, 8, 9, 10]
```

<a name="method-skipuntil"></a>
#### `skipUntil()` {.collection-method}

Phương thức `skipUntil` bỏ qua các phần tử từ collection trong khi callback được chỉ định trả về `false`. Khi callback trả về `true`, tất cả các phần tử còn lại trong collection sẽ được trả về như một collection mới:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(function (int $item) {
    return $item >= 3;
});

$subset->all();

// [3, 4]
```

Bạn cũng có thể truyền một giá trị đơn giản cho phương thức `skipUntil` để bỏ qua tất cả các phần tử cho đến khi tìm thấy giá trị được chỉ định:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(3);

$subset->all();

// [3, 4]
```

> [!WARNING]
> Nếu giá trị được chỉ định không được tìm thấy hoặc callback không bao giờ trả về `true`, phương thức `skipUntil` sẽ trả về một collection rỗng.

<a name="method-skipwhile"></a>
#### `skipWhile()` {.collection-method}

Phương thức `skipWhile` bỏ qua các phần tử từ collection trong khi callback được chỉ định trả về `true`. Khi callback trả về `false`, tất cả các phần tử còn lại trong collection sẽ được trả về như một collection mới:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipWhile(function (int $item) {
    return $item <= 3;
});

$subset->all();

// [4]
```

> [!WARNING]
> Nếu callback không bao giờ trả về `false`, phương thức `skipWhile` sẽ trả về một collection rỗng.

<a name="method-slice"></a>
#### `slice()` {.collection-method}

Phương thức `slice` trả về một phần của collection bắt đầu từ chỉ mục được chỉ định:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$slice = $collection->slice(4);

$slice->all();

// [5, 6, 7, 8, 9, 10]
```

Nếu bạn muốn giới hạn kích thước của phần được trả về, hãy truyền kích thước mong muốn làm đối số thứ hai cho phương thức:

```php
$slice = $collection->slice(4, 2);

$slice->all();

// [5, 6]
```

Phần được trả về sẽ giữ nguyên các khóa theo mặc định. Nếu bạn không muốn giữ nguyên các khóa gốc, bạn có thể sử dụng phương thức [values](#method-values) để đánh chỉ số lại cho chúng.

<a name="method-sliding"></a>
#### `sliding()` {.collection-method}

Phương thức `sliding` trả về một collection mới của các phần đại diện cho chế độ xem "cửa sổ trượt" của các phần tử trong collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunks = $collection->sliding(2);

$chunks->toArray();

// [[1, 2], [2, 3], [3, 4], [4, 5]]
```

Điều này đặc biệt hữu ích khi kết hợp với phương thức [eachSpread](#method-eachspread):

```php
$transactions->sliding(2)->eachSpread(function (Collection $previous, Collection $current) {
    $current->total = $previous->total + $current->amount;
});
```

Bạn có thể tùy chọn truyền một giá trị "bước" thứ hai, xác định khoảng cách giữa phần tử đầu tiên của mỗi phần:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunks = $collection->sliding(3, step: 2);

$chunks->toArray();

// [[1, 2, 3], [3, 4, 5]]
```

<a name="method-sole"></a>
#### `sole()` {.collection-method}

Phương thức `sole` trả về phần tử đầu tiên trong collection vượt qua một bài kiểm tra điều kiện được chỉ định, nhưng chỉ khi bài kiểm tra điều kiện khớp chính xác một phần tử:

```php
collect([1, 2, 3, 4])->sole(function (int $value, int $key) {
    return $value === 2;
});

// 2
```

Bạn cũng có thể truyền một cặp khóa / giá trị cho phương thức `sole`, sẽ trả về phần tử đầu tiên trong collection khớp với cặp được chỉ định, nhưng chỉ khi có chính xác một phần tử khớp:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->sole('product', 'Chair');

// ['product' => 'Chair', 'price' => 100]
```

Ngoài ra, bạn cũng có thể gọi phương thức `sole` không có đối số để lấy phần tử đầu tiên trong collection nếu chỉ có một phần tử:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
]);

$collection->sole();

// ['product' => 'Desk', 'price' => 200]
```

Nếu không có phần tử nào trong collection nên được trả về bởi phương thức `sole`, một ngoại lệ `\Illuminate\Collections\ItemNotFoundException` sẽ được ném. Nếu có nhiều hơn một phần tử nên được trả về, một `\Illuminate\Collections\MultipleItemsFoundException` sẽ được ném.

<a name="method-some"></a>
#### `some()` {.collection-method}

Bí danh cho phương thức [contains](#method-contains).

<a name="method-sort"></a>
#### `sort()` {.collection-method}

Phương thức `sort` sắp xếp collection. Collection đã sắp xếp giữ nguyên các khóa mảng gốc, vì vậy trong ví dụ sau chúng ta sẽ sử dụng phương thức [values](#method-values) để đặt lại các khóa thành các chỉ mục được đánh số liên tiếp:

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3, 4, 5]
```

Nếu nhu cầu sắp xếp của bạn phức tạp hơn, bạn có thể truyền một callback cho `sort` với thuật toán của riêng bạn. Tham khảo tài liệu PHP về [uasort](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters), là hàm mà phương thức `sort` của collection sử dụng nội bộ.

> [!NOTE]
> Nếu bạn cần sắp xếp một collection của các mảng hoặc đối tượng lồng nhau, hãy xem các phương thức [sortBy](#method-sortby) và [sortByDesc](#method-sortbydesc).

<a name="method-sortby"></a>
#### `sortBy()` {.collection-method}

Phương thức `sortBy` sắp xếp collection theo khóa được chỉ định. Collection đã sắp xếp giữ nguyên các khóa mảng gốc, vì vậy trong ví dụ sau chúng ta sẽ sử dụng phương thức [values](#method-values) để đặt lại các khóa thành các chỉ mục được đánh số liên tiếp:

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
]);

$sorted = $collection->sortBy('price');

$sorted->values()->all();

/*
    [
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

Phương thức `sortBy` chấp nhận [các cờ sắp xếp](https://www.php.net/manual/en/function.sort.php) làm đối số thứ hai:

```php
$collection = collect([
    ['title' => 'Item 1'],
    ['title' => 'Item 12'],
    ['title' => 'Item 3'],
]);

$sorted = $collection->sortBy('title', SORT_NATURAL);

$sorted->values()->all();

/*
    [
        ['title' => 'Item 1'],
        ['title' => 'Item 3'],
        ['title' => 'Item 12'],
    ]
*/
```

Ngoài ra, bạn có thể truyền closure của riêng mình để xác định cách sắp xếp các giá trị của collection:

```php
$collection = collect([
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$sorted = $collection->sortBy(function (array $product, int $key) {
    return count($product['colors']);
});

$sorted->values()->all();

/*
    [
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]
*/
```

Nếu bạn muốn sắp xếp collection của mình theo nhiều thuộc tính, bạn có thể truyền một mảng các thao tác sắp xếp cho phương thức `sortBy`. Mỗi thao tác sắp xếp nên là một mảng bao gồm thuộc tính mà bạn muốn sắp xếp theo và hướng của sắp xếp mong muốn:

```php
$collection = collect([
    ['name' => 'Taylor Otwell', 'age' => 34],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Taylor Otwell', 'age' => 36],
    ['name' => 'Abigail Otwell', 'age' => 32],
]);

$sorted = $collection->sortBy([
    ['name', 'asc'],
    ['age', 'desc'],
]);

$sorted->values()->all();

/*
    [
        ['name' => 'Abigail Otwell', 'age' => 32],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Taylor Otwell', 'age' => 34],
    ]
*/
```

Khi sắp xếp một collection theo nhiều thuộc tính, bạn cũng có thể cung cấp các closure xác định mỗi thao tác sắp xếp:

```php
$collection = collect([
    ['name' => 'Taylor Otwell', 'age' => 34],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Taylor Otwell', 'age' => 36],
    ['name' => 'Abigail Otwell', 'age' => 32],
]);

$sorted = $collection->sortBy([
    fn (array $a, array $b) => $a['name'] <=> $b['name'],
    fn (array $a, array $b) => $b['age'] <=> $a['age'],
]);

$sorted->values()->all();

/*
    [
        ['name' => 'Abigail Otwell', 'age' => 32],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Taylor Otwell', 'age' => 34],
    ]
*/
```

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {.collection-method}

Phương thức này có cùng chữ ký với phương thức [sortBy](#method-sortby), nhưng sẽ sắp xếp collection theo thứ tự ngược lại.

<a name="method-sortdesc"></a>
#### `sortDesc()` {.collection-method}

Phương thức này sẽ sắp xếp collection theo thứ tự ngược lại với phương thức [sort](#method-sort):

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sortDesc();

$sorted->values()->all();

// [5, 4, 3, 2, 1]
```

Khác với `sort`, bạn không thể truyền một closure cho `sortDesc`. Thay vào đó, bạn nên sử dụng phương thức [sort](#method-sort) và đảo ngược so sánh của mình.

<a name="method-sortkeys"></a>
#### `sortKeys()` {.collection-method}

Phương thức `sortKeys` sắp xếp collection theo các khóa của mảng kết hợp bên dưới:

```php
$collection = collect([
    'id' => 22345,
    'first' => 'John',
    'last' => 'Doe',
]);

$sorted = $collection->sortKeys();

$sorted->all();

/*
    [
        'first' => 'John',
        'id' => 22345,
        'last' => 'Doe',
    ]
*/
```

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()` {.collection-method}

Phương thức này có cùng chữ ký với phương thức [sortKeys](#method-sortkeys), nhưng sẽ sắp xếp collection theo thứ tự ngược lại.

<a name="method-sortkeysusing"></a>
#### `sortKeysUsing()` {.collection-method}

Phương thức `sortKeysUsing` sắp xếp collection theo các khóa của mảng kết hợp bên dưới bằng cách sử dụng một callback:

```php
$collection = collect([
    'ID' => 22345,
    'first' => 'John',
    'last' => 'Doe',
]);

$sorted = $collection->sortKeysUsing('strnatcasecmp');

$sorted->all();

/*
    [
        'first' => 'John',
        'ID' => 22345,
        'last' => 'Doe',
    ]
*/
```

Callback phải là một hàm so sánh trả về một số nguyên nhỏ hơn, bằng, hoặc lớn hơn không. Để biết thêm thông tin, hãy tham khảo tài liệu PHP về [uksort](https://www.php.net/manual/en/function.uksort.php#refsect1-function.uksort-parameters), là hàm PHP mà phương thức `sortKeysUsing` sử dụng nội bộ.

<a name="method-splice"></a>
#### `splice()` {.collection-method}

Phương thức `splice` loại bỏ và trả về một phần các phần tử bắt đầu từ chỉ mục được chỉ định:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2);

$chunk->all();

// [3, 4, 5]

$collection->all();

// [1, 2]
```

Bạn có thể truyền một đối số thứ hai để giới hạn kích thước của collection kết quả:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 4, 5]
```

Ngoài ra, bạn có thể truyền một đối số thứ ba chứa các phần tử mới để thay thế các phần tử bị loại bỏ khỏi collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1, [10, 11]);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 10, 11, 4, 5]
```

<a name="method-split"></a>
#### `split()` {.collection-method}

Phương thức `split` chia một collection thành số lượng nhóm được chỉ định:

```php
$collection = collect([1, 2, 3, 4, 5]);

$groups = $collection->split(3);

$groups->all();

// [[1, 2], [3, 4], [5]]
```

<a name="method-splitin"></a>
#### `splitIn()` {.collection-method}

Phương thức `splitIn` chia một collection thành số lượng nhóm được chỉ định, điền đầy các nhóm không phải cuối cùng hoàn toàn trước khi phân bổ phần còn lại cho nhóm cuối cùng:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$groups = $collection->splitIn(3);

$groups->all();

// [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]
```

<a name="method-sum"></a>
#### `sum()` {.collection-method}

Phương thức `sum` trả về tổng của tất cả các phần tử trong collection:

```php
collect([1, 2, 3, 4, 5])->sum();

// 15
```

Nếu collection chứa các mảng hoặc đối tượng lồng nhau, bạn nên truyền một khóa sẽ được sử dụng để xác định các giá trị cần tính tổng:

```php
$collection = collect([
    ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
    ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
]);

$collection->sum('pages');

// 1272
```

Ngoài ra, bạn có thể truyền closure của riêng mình để xác định các giá trị của collection cần tính tổng:

```php
$collection = collect([
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$collection->sum(function (array $product) {
    return count($product['colors']);
});

// 6
```

<a name="method-take"></a>
#### `take()` {.collection-method}

Phương thức `take` trả về một collection mới với số lượng phần tử được chỉ định:

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(3);

$chunk->all();

// [0, 1, 2]
```

Bạn cũng có thể truyền một số nguyên âm để lấy số lượng phần tử được chỉ định từ cuối collection:

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(-2);

$chunk->all();

// [4, 5]
```

<a name="method-takeuntil"></a>
#### `takeUntil()` {.collection-method}

Phương thức `takeUntil` trả về các phần tử trong collection cho đến khi callback được chỉ định trả về `true`:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->takeUntil(function (int $item) {
    return $item >= 3;
});

$subset->all();

// [1, 2]
```

Bạn cũng có thể truyền một giá trị đơn giản cho phương thức `takeUntil` để lấy các phần tử cho đến khi tìm thấy giá trị được chỉ định:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->takeUntil(3);

$subset->all();

// [1, 2]
```

> [!WARNING]
> Nếu giá trị được chỉ định không được tìm thấy hoặc callback không bao giờ trả về `true`, phương thức `takeUntil` sẽ trả về tất cả các phần tử trong collection.

<a name="method-takewhile"></a>
#### `takeWhile()` {.collection-method}

Phương thức `takeWhile` trả về các phần tử trong collection cho đến khi callback được chỉ định trả về `false`:

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->takeWhile(function (int $item) {
    return $item < 3;
});

$subset->all();

// [1, 2]
```

> [!WARNING]
> Nếu callback không bao giờ trả về `false`, phương thức `takeWhile` sẽ trả về tất cả các phần tử trong collection.

<a name="method-tap"></a>
#### `tap()` {.collection-method}

Phương thức `tap` truyền collection cho callback được chỉ định, cho phép bạn "chạm" vào collection tại một điểm cụ thể và làm điều gì đó với các phần tử trong khi không ảnh hưởng đến chính collection. Sau đó collection được trả về bởi phương thức `tap`:

```php
collect([2, 4, 3, 1, 5])
    ->sort()
    ->tap(function (Collection $collection) {
        Log::debug('Values after sorting', $collection->values()->all());
    })
    ->shift();

// 1
```

<a name="method-times"></a>
#### `times()` {.collection-method}

Phương thức tĩnh `times` tạo một collection mới bằng cách gọi closure được chỉ định một số lần được chỉ định:

```php
$collection = Collection::times(10, function (int $number) {
    return $number * 9;
});

$collection->all();

// [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
```

<a name="method-toarray"></a>
#### `toArray()` {.collection-method}

Phương thức `toArray` chuyển đổi collection thành một mảng PHP thuần túy. Nếu các giá trị của collection là các model [Eloquent](/docs/{{version}}/eloquent), các model cũng sẽ được chuyển đổi thành mảng:

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toArray();

/*
    [
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

> [!WARNING]
> `toArray` cũng chuyển đổi tất cả các đối tượng lồng nhau của collection là một instance của `Arrayable` thành mảng. Nếu bạn muốn lấy mảng thô bên dưới collection, hãy sử dụng phương thức [all](#method-all) thay thế.

<a name="method-tojson"></a>
#### `toJson()` {.collection-method}

Phương thức `toJson` chuyển đổi collection thành một chuỗi JSON được tuần tự hóa:

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toJson();

// '{"name":"Desk", "price":200}'
```

<a name="method-to-pretty-json"></a>
#### `toPrettyJson()` {.collection-method}

Phương thức `toPrettyJson` chuyển đổi collection thành một chuỗi JSON được định dạng bằng cách sử dụng tùy chọn `JSON_PRETTY_PRINT`:

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toPrettyJson();
```

<a name="method-transform"></a>
#### `transform()` {.collection-method}

Phương thức `transform` lặp qua collection và gọi callback được chỉ định với mỗi phần tử trong collection. Các phần tử trong collection sẽ được thay thế bằng các giá trị được trả về bởi callback:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->transform(function (int $item, int $key) {
    return $item * 2;
});

$collection->all();

// [2, 4, 6, 8, 10]
```

> [!WARNING]
> Khác với hầu hết các phương thức collection khác, `transform` sửa đổi chính collection. Nếu bạn muốn tạo một collection mới thay thế, hãy sử dụng phương thức [map](#method-map).

<a name="method-undot"></a>
#### `undot()` {.collection-method}

Phương thức `undot` mở rộng một collection một chiều sử dụng ký hiệu "dot" thành một collection đa chiều:

```php
$person = collect([
    'name.first_name' => 'Marie',
    'name.last_name' => 'Valentine',
    'address.line_1' => '2992 Eagle Drive',
    'address.line_2' => '',
    'address.suburb' => 'Detroit',
    'address.state' => 'MI',
    'address.postcode' => '48219'
]);

$person = $person->undot();

$person->toArray();

/*
    [
        "name" => [
            "first_name" => "Marie",
            "last_name" => "Valentine",
        ],
        "address" => [
            "line_1" => "2992 Eagle Drive",
            "line_2" => "",
            "suburb" => "Detroit",
            "state" => "MI",
            "postcode" => "48219",
        ],
    ]
*/
```

<a name="method-union"></a>
#### `union()` {.collection-method}

Phương thức `union` thêm mảng được chỉ định vào collection. Nếu mảng được chỉ định chứa các khóa đã có trong collection gốc, các giá trị của collection gốc sẽ được ưu tiên:

```php
$collection = collect([1 => ['a'], 2 => ['b']]);

$union = $collection->union([3 => ['c'], 1 => ['d']]);

$union->all();

// [1 => ['a'], 2 => ['b'], 3 => ['c']]
```

<a name="method-unique"></a>
#### `unique()` {.collection-method}

Phương thức `unique` trả về tất cả các phần tử duy nhất trong collection. Collection được trả về giữ nguyên các khóa mảng gốc, vì vậy trong ví dụ sau chúng ta sẽ sử dụng phương thức [values](#method-values) để đặt lại các khóa thành các chỉ mục được đánh số liên tiếp:

```php
$collection = collect([1, 1, 2, 2, 3, 4, 2]);

$unique = $collection->unique();

$unique->values()->all();

// [1, 2, 3, 4]
```

Khi xử lý các mảng hoặc đối tượng lồng nhau, bạn có thể chỉ định khóa được sử dụng để xác định tính duy nhất:

```php
$collection = collect([
    ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
    ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
    ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
    ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
    ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
]);

$unique = $collection->unique('brand');

$unique->values()->all();

/*
    [
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
    ]
*/
```

Cuối cùng, bạn cũng có thể truyền closure của riêng mình cho phương thức `unique` để chỉ định giá trị nào nên xác định tính duy nhất của một phần tử:

```php
$unique = $collection->unique(function (array $item) {
    return $item['brand'].$item['type'];
});

$unique->values()->all();

/*
    [
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]
*/
```

Phương thức `unique` sử dụng so sánh "lỏng" khi kiểm tra các giá trị phần tử, nghĩa là một chuỗi có giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị. Sử dụng phương thức [uniqueStrict](#method-uniquestrict) để lọc bằng cách sử dụng so sánh "nghiêm ngặt".

> [!NOTE]
> Hành vi của phương thức này được sửa đổi khi sử dụng [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-unique).

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {.collection-method}

Phương thức này có cùng chữ ký với phương thức [unique](#method-unique); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".

<a name="method-unless"></a>
#### `unless()` {.collection-method}

Phương thức `unless` sẽ thực thi callback được chỉ định trừ khi đối số đầu tiên được đưa cho phương thức được đánh giá là `true`. Instance collection và đối số đầu tiên được đưa cho phương thức `unless` sẽ được cung cấp cho closure:

```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function (Collection $collection, bool $value) {
    return $collection->push(4);
});

$collection->unless(false, function (Collection $collection, bool $value) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

Một callback thứ hai có thể được truyền cho phương thức `unless`. Callback thứ hai sẽ được thực thi khi đối số đầu tiên được đưa cho phương thức `unless` được đánh giá là `true`:

```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function (Collection $collection, bool $value) {
    return $collection->push(4);
}, function (Collection $collection, bool $value) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

Để biết đảo ngược của `unless`, hãy xem phương thức [when](#method-when).

<a name="method-unlessempty"></a>
#### `unlessEmpty()` {.collection-method}

Bí danh cho phương thức [whenNotEmpty](#method-whennotempty).

<a name="method-unlessnotempty"></a>
#### `unlessNotEmpty()` {.collection-method}

Bí danh cho phương thức [whenEmpty](#method-whenempty).

<a name="method-unwrap"></a>
#### `unwrap()` {.collection-method}

Phương thức tĩnh `unwrap` trả về các phần tử bên dưới của collection từ giá trị được chỉ định khi áp dụng:

```php
Collection::unwrap(collect('John Doe'));

// ['John Doe']

Collection::unwrap(['John Doe']);

// ['John Doe']

Collection::unwrap('John Doe');

// 'John Doe'
```

<a name="method-value"></a>
#### `value()` {.collection-method}

Phương thức `value` truy xuất một giá trị được chỉ định từ phần tử đầu tiên của collection:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Speaker', 'price' => 400],
]);

$value = $collection->value('price');

// 200
```

<a name="method-values"></a>
#### `values()` {.collection-method}
|
 2|Phương thức `values` trả về một collection mới với các key được đặt lại thành các số nguyên liên tiếp:
 3|
 4|```php
 5|$collection = collect([
 6|    10 => ['product' => 'Desk', 'price' => 200],
 7|    11 => ['product' => 'Speaker', 'price' => 400],
 8|]);
 9|
 10|$values = $collection->values();
 11|
 12|$values->all();
 13|
 14|/*
 15|    [
 16|        0 => ['product' => 'Desk', 'price' => 200],
 17|        1 => ['product' => 'Speaker', 'price' => 400],
 18|    ]
 19|*/
 20|```
 21|
 22|<a name="method-when"></a>
 23|#### `when()` {.collection-method}
 24|
 25|Phương thức `when` sẽ thực thi callback đã cho khi đối số đầu tiên được truyền vào phương thức đánh giá là `true`. Thể hiện của collection và đối số đầu tiên được truyền vào phương thức `when` sẽ được cung cấp cho closure:
 26|
 27|```php
 28|$collection = collect([1, 2, 3]);
 29|
 30|$collection->when(true, function (Collection $collection, bool $value) {
 31|    return $collection->push(4);
 32|});
 33|
 34|$collection->when(false, function (Collection $collection, bool $value) {
 35|    return $collection->push(5);
 36|});
 37|
 38|$collection->all();
 39|
 40|// [1, 2, 3, 4]
 41|```
 42|
 43|Một callback thứ hai có thể được truyền vào phương thức `when`. Callback thứ hai sẽ được thực thi khi đối số đầu tiên được truyền vào phương thức `when` đánh giá là `false`:
 44|
 45|```php
 46|$collection = collect([1, 2, 3]);
 47|
 48|$collection->when(false, function (Collection $collection, bool $value) {
 49|    return $collection->push(4);
 50|}, function (Collection $collection, bool $value) {
 51|    return $collection->push(5);
 52|});
 53|
 54|$collection->all();
 55|
 56|// [1, 2, 3, 5]
 57|```
 58|
 59|Để xem phương thức ngược lại của `when`, hãy xem phương thức [unless](#method-unless).
 60|
 61|<a name="method-whenempty"></a>
 62|#### `whenEmpty()` {.collection-method}
 63|
 64|Phương thức `whenEmpty` sẽ thực thi callback đã cho khi collection rỗng:
 65|
 66|```php
 67|$collection = collect(['Michael', 'Tom']);
 68|
 69|$collection->whenEmpty(function (Collection $collection) {
 70|    return $collection->push('Adam');
 71|});
 72|
 73|$collection->all();
 74|
 75|// ['Michael', 'Tom']
 76|
 77|$collection = collect();
 78|
 79|$collection->whenEmpty(function (Collection $collection) {
 80|    return $collection->push('Adam');
 81|});
 82|
 83|$collection->all();
 84|
 85|// ['Adam']
 86|```
 87|
 88|Một closure thứ hai có thể được truyền vào phương thức `whenEmpty` sẽ được thực thi khi collection không rỗng:
 89|
 90|```php
 91|$collection = collect(['Michael', 'Tom']);
 92|
 93|$collection->whenEmpty(function (Collection $collection) {
 94|    return $collection->push('Adam');
 95|}, function (Collection $collection) {
 96|    return $collection->push('Taylor');
 97|});
 98|
 99|$collection->all();
100|
101|// ['Michael', 'Tom', 'Taylor']
102|```
103|
104|Để xem phương thức ngược lại của `whenEmpty`, hãy xem phương thức [whenNotEmpty](#method-whennotempty).
105|
106|<a name="method-whennotempty"></a>
107|#### `whenNotEmpty()` {.collection-method}
108|
109|Phương thức `whenNotEmpty` sẽ thực thi callback đã cho khi collection không rỗng:
110|
111|```php
112|$collection = collect(['Michael', 'Tom']);
113|
114|$collection->whenNotEmpty(function (Collection $collection) {
115|    return $collection->push('Adam');
116|});
117|
118|$collection->all();
119|
120|// ['Michael', 'Tom', 'Adam']
121|
122|$collection = collect();
123|
124|$collection->whenNotEmpty(function (Collection $collection) {
125|    return $collection->push('Adam');
126|});
127|
128|$collection->all();
129|
130|// []
131|```
132|
133|Một closure thứ hai có thể được truyền vào phương thức `whenNotEmpty` sẽ được thực thi khi collection rỗng:
134|
135|```php
136|$collection = collect();
137|
138|$collection->whenNotEmpty(function (Collection $collection) {
139|    return $collection->push('Adam');
140|}, function (Collection $collection) {
141|    return $collection->push('Taylor');
142|});
143|
144|$collection->all();
145|
146|// ['Taylor']
147|```
148|
149|Để xem phương thức ngược lại của `whenNotEmpty`, hãy xem phương thức [whenEmpty](#method-whenempty).
150|
151|<a name="method-where"></a>
152|#### `where()` {.collection-method}
153|
154|Phương thức `where` lọc collection theo một cặp key / value đã cho:
155|
156|```php
157|$collection = collect([
158|    ['product' => 'Desk', 'price' => 200],
159|    ['product' => 'Chair', 'price' => 100],
160|    ['product' => 'Bookcase', 'price' => 150],
161|    ['product' => 'Door', 'price' => 100],
162|]);
163|
164|$filtered = $collection->where('price', 100);
165|
166|$filtered->all();
167|
168|/*
169|    [
170|        ['product' => 'Chair', 'price' => 100],
171|        ['product' => 'Door', 'price' => 100],
172|    ]
173|*/
174|```
175|
176|Phương thức `where` sử dụng so sánh "lỏng" khi kiểm tra giá trị của phần tử, nghĩa là một chuỗi có giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị. Sử dụng phương thức [whereStrict](#method-wherestrict) để lọc bằng cách sử dụng so sánh "nghiêm ngặt", hoặc các phương thức [whereNull](#method-wherenull) và [whereNotNull](#method-wherenotnull) để lọc các giá trị `null`.
177|
178|Tùy chọn, bạn có thể truyền một toán tử so sánh làm tham số thứ hai. Các toán tử được hỗ trợ là: '===', '!==', '!=', '==', '=', '<>', '>', '<', '>=', và '<=':
179|
180|```php
181|$collection = collect([
182|    ['name' => 'Jim', 'platform' => 'Mac'],
183|    ['name' => 'Sally', 'platform' => 'Mac'],
184|    ['name' => 'Sue', 'platform' => 'Linux'],
185|]);
186|
187|$filtered = $collection->where('platform', '!=', 'Linux');
188|
189|$filtered->all();
190|
191|/*
192|    [
193|        ['name' => 'Jim', 'platform' => 'Mac'],
194|        ['name' => 'Sally', 'platform' => 'Mac'],
195|    ]
196|*/
197|```
198|
199|<a name="method-wherestrict"></a>
200|#### `whereStrict()` {.collection-method}
201|
202|Phương thức này có cùng chữ ký với phương thức [where](#method-where); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".
203|
204|<a name="method-wherebetween"></a>
205|#### `whereBetween()` {.collection-method}
206|
207|Phương thức `whereBetween` lọc collection bằng cách xác định xem một giá trị phần tử được chỉ định có nằm trong một phạm vi đã cho hay không:
208|
209|```php
210|$collection = collect([
211|    ['product' => 'Desk', 'price' => 200],
212|    ['product' => 'Chair', 'price' => 80],
213|    ['product' => 'Bookcase', 'price' => 150],
214|    ['product' => 'Pencil', 'price' => 30],
215|    ['product' => 'Door', 'price' => 100],
216|]);
217|
218|$filtered = $collection->whereBetween('price', [100, 200]);
219|
220|$filtered->all();
221|
222|/*
223|    [
224|        ['product' => 'Desk', 'price' => 200],
225|        ['product' => 'Bookcase', 'price' => 150],
226|        ['product' => 'Door', 'price' => 100],
227|    ]
228|*/
229|```
230|
231|<a name="method-wherein"></a>
232|#### `whereIn()` {.collection-method}
233|
234|Phương thức `whereIn` loại bỏ các phần tử khỏi collection không có một giá trị phần tử được chỉ định nằm trong mảng đã cho:
235|
236|```php
237|$collection = collect([
238|    ['product' => 'Desk', 'price' => 200],
239|    ['product' => 'Chair', 'price' => 100],
240|    ['product' => 'Bookcase', 'price' => 150],
241|    ['product' => 'Door', 'price' => 100],
242|]);
243|
244|$filtered = $collection->whereIn('price', [150, 200]);
245|
246|$filtered->all();
247|
248|/*
249|    [
250|        ['product' => 'Desk', 'price' => 200],
251|        ['product' => 'Bookcase', 'price' => 150],
252|    ]
253|*/
254|```
255|
256|Phương thức `whereIn` sử dụng so sánh "lỏng" khi kiểm tra giá trị của phần tử, nghĩa là một chuỗi có giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị. Sử dụng phương thức [whereInStrict](#method-whereinstrict) để lọc bằng cách sử dụng so sánh "nghiêm ngặt".
257|
258|<a name="method-whereinstrict"></a>
259|#### `whereInStrict()` {.collection-method}
260|
261|Phương thức này có cùng chữ ký với phương thức [whereIn](#method-wherein); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".
262|
263|<a name="method-whereinstanceof"></a>
264|#### `whereInstanceOf()` {.collection-method}
265|
266|Phương thức `whereInstanceOf` lọc collection theo một kiểu class đã cho:
267|
268|```php
269|use App\Models\User;
270|use App\Models\Post;
271|
272|$collection = collect([
273|    new User,
274|    new User,
275|    new Post,
276|]);
277|
278|$filtered = $collection->whereInstanceOf(User::class);
279|
280|$filtered->all();
281|
282|// [App\Models\User, App\Models\User]
283|```
284|
285|<a name="method-wherenotbetween"></a>
286|#### `whereNotBetween()` {.collection-method}
287|
288|Phương thức `whereNotBetween` lọc collection bằng cách xác định xem một giá trị phần tử được chỉ định có nằm ngoài một phạm vi đã cho hay không:
289|
290|```php
291|$collection = collect([
292|    ['product' => 'Desk', 'price' => 200],
293|    ['product' => 'Chair', 'price' => 80],
294|    ['product' => 'Bookcase', 'price' => 150],
295|    ['product' => 'Pencil', 'price' => 30],
296|    ['product' => 'Door', 'price' => 100],
297|]);
298|
299|$filtered = $collection->whereNotBetween('price', [100, 200]);
300|
301|$filtered->all();
302|
303|/*
304|    [
305|        ['product' => 'Chair', 'price' => 80],
306|        ['product' => 'Pencil', 'price' => 30],
307|    ]
308|*/
309|```
310|
311|<a name="method-wherenotin"></a>
312|#### `whereNotIn()` {.collection-method}
313|
314|Phương thức `whereNotIn` loại bỏ các phần tử khỏi collection có một giá trị phần tử được chỉ định nằm trong mảng đã cho:
315|
316|```php
317|$collection = collect([
318|    ['product' => 'Desk', 'price' => 200],
319|    ['product' => 'Chair', 'price' => 100],
320|    ['product' => 'Bookcase', 'price' => 150],
321|    ['product' => 'Door', 'price' => 100],
322|]);
323|
324|$filtered = $collection->whereNotIn('price', [150, 200]);
325|
326|$filtered->all();
327|
328|/*
329|    [
330|        ['product' => 'Chair', 'price' => 100],
331|        ['product' => 'Door', 'price' => 100],
332|    ]
333|*/
334|```
335|
336|Phương thức `whereNotIn` sử dụng so sánh "lỏng" khi kiểm tra giá trị của phần tử, nghĩa là một chuỗi có giá trị số nguyên sẽ được coi là bằng với một số nguyên có cùng giá trị. Sử dụng phương thức [whereNotInStrict](#method-wherenotinstrict) để lọc bằng cách sử dụng so sánh "nghiêm ngặt".
337|
338|<a name="method-wherenotinstrict"></a>
339|#### `whereNotInStrict()` {.collection-method}
340|
341|Phương thức này có cùng chữ ký với phương thức [whereNotIn](#method-wherenotin); tuy nhiên, tất cả các giá trị được so sánh bằng cách sử dụng so sánh "nghiêm ngặt".
342|
343|<a name="method-wherenotnull"></a>
344|#### `whereNotNull()` {.collection-method}
345|
346|Phương thức `whereNotNull` trả về các phần tử từ collection trong đó key đã cho không phải là `null`:
347|
348|```php
349|$collection = collect([
350|    ['name' => 'Desk'],
351|    ['name' => null],
352|    ['name' => 'Bookcase'],
353|    ['name' => 0],
354|    ['name' => ''],
355|]);
356|
357|$filtered = $collection->whereNotNull('name');
358|
359|$filtered->all();
360|
361|/*
362|    [
363|        ['name' => 'Desk'],
364|        ['name' => 'Bookcase'],
365|        ['name' => 0],
366|        ['name' => ''],
367|    ]
368|*/
369|```
370|
371|<a name="method-wherenull"></a>
372|#### `whereNull()` {.collection-method}
373|
374|Phương thức `whereNull` trả về các phần tử từ collection trong đó key đã cho là `null`:
375|
376|```php
377|$collection = collect([
378|    ['name' => 'Desk'],
379|    ['name' => null],
380|    ['name' => 'Bookcase'],
381|    ['name' => 0],
382|    ['name' => ''],
383|]);
384|
385|$filtered = $collection->whereNull('name');
386|
387|$filtered->all();
388|
389|/*
390|    [
391|        ['name' => null],
392|    ]
393|*/
394|```
395|
396|<a name="method-wrap"></a>
397|#### `wrap()` {.collection-method}
398|
399|Phương thức tĩnh `wrap` bọc giá trị đã cho trong một collection khi có thể áp dụng:
400|
401|```php
402|use Illuminate\Support\Collection;
403|
404|$collection = Collection::wrap('John Doe');
405|
406|$collection->all();
407|
408|// ['John Doe']
409|
410|$collection = Collection::wrap(['John Doe']);
411|
412|$collection->all();
413|
414|// ['John Doe']
415|
416|$collection = Collection::wrap(collect('John Doe'));
417|
418|$collection->all();
419|
420|// ['John Doe']
421|```
422|
423|<a name="method-zip"></a>
424|#### `zip()` {.collection-method}
425|
426|Phương thức `zip` gộp các giá trị của mảng đã cho với các giá trị của collection gốc tại chỉ số tương ứng của chúng:
427|
428|```php
429|$collection = collect(['Chair', 'Desk']);
430|
431|$zipped = $collection->zip([100, 200]);
432|
433|$zipped->all();
434|
435|// [['Chair', 100], ['Desk', 200]]
436|```
437|
438|<a name="higher-order-messages"></a>
439|## Higher Order Messages
440|
441|Collections cũng cung cấp hỗ trợ cho "higher order messages", là các phím tắt để thực hiện các hành động phổ biến trên collections. Các phương thức collection cung cấp higher order messages là: [average](#method-average), [avg](#method-avg), [contains](#method-contains), [each](#method-each), [every](#method-every), [filter](#method-filter), [first](#method-first), [flatMap](#method-flatmap), [groupBy](#method-groupby), [keyBy](#method-keyby), [map](#method-map), [max](#method-max), [min](#method-min), [partition](#method-partition), [reject](#method-reject), [skipUntil](#method-skipuntil), [skipWhile](#method-skipwhile), [some](#method-some), [sortBy](#method-sortby), [sortByDesc](#method-sortbydesc), [sum](#method-sum), [takeUntil](#method-takeuntil), [takeWhile](#method-takewhile), và [unique](#method-unique).
442|
443|Mỗi higher order message có thể được truy cập như một thuộc tính động trên một thể hiện collection. Ví dụ, hãy sử dụng higher order message `each` để gọi một phương thức trên mỗi đối tượng trong một collection:
444|
445|```php
446|use App\Models\User;
447|
448|$users = User::where('votes', '>', 500)->get();
449|
450|$users->each->markAsVip();
451|```
452|
453|Tương tự, chúng ta có thể sử dụng higher order message `sum` để thu thập tổng số "votes" cho một collection người dùng:
454|
455|```php
456|$users = User::where('group', 'Development')->get();
457|
458|return $users->sum->votes;
459|```
460|
461|<a name="lazy-collections"></a>
462|## Lazy Collections
463|
464|<a name="lazy-collection-introduction"></a>
465|### Introduction
466|
467|> [!WARNING]
468|> Trước khi tìm hiểu thêm về lazy collections của Laravel, hãy dành thời gian để làm quen với [PHP generators](https://www.php.net/manual/en/language.generators.overview.php).
469|
470|Để bổ sung cho class `Collection` vốn đã mạnh mẽ, class `LazyCollection` tận dụng [generators](https://www.php.net/manual/en/language.generators.overview.php) của PHP để cho phép bạn làm việc với các bộ dữ liệu rất lớn trong khi giữ mức sử dụng bộ nhớ thấp.
471|
472|Ví dụ, hãy tưởng tượng ứng dụng của bạn cần xử lý một tệp log nhiều gigabyte trong khi tận dụng các phương thức collection của Laravel để phân tích các log. Thay vì đọc toàn bộ tệp vào bộ nhớ cùng một lúc, lazy collections có thể được sử dụng để chỉ giữ một phần nhỏ của tệp trong bộ nhớ tại một thời điểm nhất định:
473|
474|```php
475|use App\Models\LogEntry;
476|use Illuminate\Support\LazyCollection;
477|
478|LazyCollection::make(function () {
479|    $handle = fopen('log.txt', 'r');
480|
481|    while (($line = fgets($handle)) !== false) {
482|        yield $line;
483|    }
484|
485|    fclose($handle);
486|})->chunk(4)->map(function (array $lines) {
487|    return LogEntry::fromLines($lines);
488|})->each(function (LogEntry $logEntry) {
489|    // Xử lý log entry...
490|});
491|```
492|
493|Hoặc, hãy tưởng tượng bạn cần lặp qua 10,000 model Eloquent. Khi sử dụng collections truyền thống của Laravel, tất cả 10,000 model Eloquent phải được tải vào bộ nhớ cùng một lúc:
494|
495|```php
496|use App\Models\User;
497|
498|$users = User::all()->filter(function (User $user) {
499|    return $user->id > 500;
500|});
501|```
502|
503|Tuy nhiên, phương thức `cursor` của query builder trả về một thể hiện `LazyCollection`. Điều này cho phép bạn vẫn chỉ chạy một truy vấn duy nhất đối với cơ sở dữ liệu nhưng cũng chỉ giữ một model Eloquent được tải trong bộ nhớ tại một thời điểm. Trong ví dụ này, callback `filter` không được thực thi cho đến khi chúng ta thực sự lặp qua từng người dùng một cách riêng lẻ, cho phép giảm đáng kể mức sử dụng bộ nhớ:
504|
505|```php
506|use App\Models\User;
507|
508|$users = User::cursor()->filter(function (User $user) {
509|    return $user->id > 500;
510|});
511|
512|foreach ($users as $user) {
513|    echo $user->id;
514|}
515|```
516|
517|<a name="creating-lazy-collections"></a>
518|### Creating Lazy Collections
519|
520|Để tạo một thể hiện lazy collection, bạn nên truyền một hàm generator PHP vào phương thức `make` của collection:
521|
522|```php
523|use Illuminate\Support\LazyCollection;
524|
525|LazyCollection::make(function () {
526|    $handle = fopen('log.txt', 'r');
527|
528|    while (($line = fgets($handle)) !== false) {
529|        yield $line;
530|    }
531|
532|    fclose($handle);
533|});
534|```
535|
536|<a name="the-enumerable-contract"></a>
537|### The Enumerable Contract
538|
539|Hầu hết tất cả các phương thức có sẵn trên class `Collection` cũng có sẵn trên class `LazyCollection`. Cả hai class này đều triển khai contract `Illuminate\Support\Enumerable`, định nghĩa các phương thức sau:
540|
541|<style>
542|    .collection-method-list > p {
543|        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
544|    }
545|
546|    .collection-method-list a {
547|        display: block;
548|        overflow: hidden;
549|        text-overflow: ellipsis;
550|        white-space: nowrap;
551|    }
552|</style>
553|
554|<div class="collection-method-list" markdown="1">
555|
556|[all](#method-all)
557|[average](#method-average)
558|[avg](#method-avg)
559|[chunk](#method-chunk)
560|[chunkWhile](#method-chunkwhile)
561|[collapse](#method-collapse)
562|[collect](#method-collect)
563|[combine](#method-combine)
564|[concat](#method-concat)
565|[contains](#method-contains)
566|[containsStrict](#method-containsstrict)
567|[count](#method-count)
568|[countBy](#method-countBy)
569|[crossJoin](#method-crossjoin)
570|[dd](#method-dd)
571|[diff](#method-diff)
572|[diffAssoc](#method-diffassoc)
573|[diffKeys](#method-diffkeys)
574|[dump](#method-dump)
575|[duplicates](#method-duplicates)
576|[duplicatesStrict](#method-duplicatesstrict)
577|[each](#method-each)
578|[eachSpread](#method-eachspread)
579|[every](#method-every)
580|[except](#method-except)
581|[filter](#method-filter)
582|[first](#method-first)
583|[firstOrFail](#method-first-or-fail)
584|[firstWhere](#method-first-where)
585|[flatMap](#method-flatmap)
586|[flatten](#method-flatten)
587|[flip](#method-flip)
588|[forPage](#method-forpage)
589|[get](#method-get)
590|[groupBy](#method-groupby)
591|[has](#method-has)
592|[implode](#method-implode)
593|[intersect](#method-intersect)
594|[intersectAssoc](#method-intersectAssoc)
595|[intersectByKeys](#method-intersectbykeys)
596|[isEmpty](#method-isempty)
597|[isNotEmpty](#method-isnotempty)
598|[join](#method-join)
599|[keyBy](#method-keyby)
600|[keys](#method-keys)
601|[last](#method-last)
602|[macro](#method-macro)
603|[make](#method-make)
604|[map](#method-map)
605|[mapInto](#method-mapinto)
606|[mapSpread](#method-mapspread)
607|[mapToGroups](#method-maptogroups)
608|[mapWithKeys](#method-mapwithkeys)
609|[max](#method-max)
610|[median](#method-median)
611|[merge](#method-merge)
612|[mergeRecursive](#method-mergerecursive)
613|[min](#method-min)
614|[mode](#method-mode)
615|[nth](#method-nth)
616|[only](#method-only)
617|[pad](#method-pad)
618|[partition](#method-partition)
619|[pipe](#method-pipe)
620|[pluck](#method-pluck)
621|[random](#method-random)
622|[reduce](#method-reduce)
623|[reject](#method-reject)
624|[replace](#method-replace)
625|[replaceRecursive](#method-replacerecursive)
626|[reverse](#method-reverse)
627|[search](#method-search)
628|[shuffle](#method-shuffle)
629|[skip](#method-skip)
630|[slice](#method-slice)
631|[sole](#method-sole)
632|[some](#method-some)
633|[sort](#method-sort)
634|[sortBy](#method-sortby)
635|[sortByDesc](#method-sortbydesc)
636|[sortKeys](#method-sortkeys)
637|[sortKeysDesc](#method-sortkeysdesc)
638|[split](#method-split)
639|[sum](#method-sum)
640|[take](#method-take)
641|[tap](#method-tap)
642|[times](#method-times)
643|[toArray](#method-toarray)
644|[toJson](#method-tojson)
645|[union](#method-union)
646|[unique](#method-unique)
647|[uniqueStrict](#method-uniquestrict)
648|[unless](#method-unless)
649|[unlessEmpty](#method-unlessempty)
650|[unlessNotEmpty](#method-unlessnotempty)
651|[unwrap](#method-unwrap)
652|[values](#method-values)
653|[when](#method-when)
654|[whenEmpty](#method-whenempty)
655|[whenNotEmpty](#method-whennotempty)
656|[where](#method-where)
657|[whereStrict](#method-wherestrict)
658|[whereBetween](#method-wherebetween)
659|[whereIn](#method-wherein)
660|[whereInStrict](#method-whereinstrict)
661|[whereInstanceOf](#method-whereinstanceof)
662|[whereNotBetween](#method-wherenotbetween)
663|[whereNotIn](#method-wherenotin)
664|[whereNotInStrict](#method-wherenotinstrict)
665|[wrap](#method-wrap)
666|[zip](#method-zip)
667|
668|</div>
669|
670|> [!WARNING]
671|> Các phương thức thay đổi collection (như `shift`, `pop`, `prepend` v.v.) **không** có sẵn trên class `LazyCollection`.
672|
673|<a name="lazy-collection-methods"></a>
674|### Lazy Collection Methods
675|
676|Ngoài các phương thức được định nghĩa trong contract `Enumerable`, class `LazyCollection` chứa các phương thức sau:
677|
678|<a name="method-takeUntilTimeout"></a>
679|#### `takeUntilTimeout()` {.collection-method}
680|
681|Phương thức `takeUntilTimeout` trả về một lazy collection mới sẽ liệt kê các giá trị cho đến thời điểm được chỉ định. Sau thời điểm đó, collection sẽ ngừng liệt kê:
682|
683|```php
684|$lazyCollection = LazyCollection::times(INF)
685|    ->takeUntilTimeout(now()->plus(minutes: 1));
686|
687|$lazyCollection->each(function (int $number) {
688|    dump($number);
689|
690|    sleep(1);
691|});
692|
693|// 1
694|// 2
695|// ...
696|// 58
697|// 59
698|```
699|
700|Để minh họa cách sử dụng phương thức này, hãy tưởng tượng một ứng dụng gửi hóa đơn từ cơ sở dữ liệu bằng cách sử dụng cursor. Bạn có thể định nghĩa một [scheduled task](/docs/{{version}}/scheduling) chạy mỗi 15 phút và chỉ xử lý hóa đơn tối đa trong 14 phút:
701|
702|```php
703|use App\Models\Invoice;
704|use Illuminate\Support\Carbon;
705|
706|Invoice::pending()->cursor()
707|    ->takeUntilTimeout(
708|        Carbon::createFromTimestamp(LARAVEL_START)->add(14, 'minutes')
709|    )
710|    ->each(fn (Invoice $invoice) => $invoice->submit());
711|```
712|
713|<a name="method-tapEach"></a>
714|#### `tapEach()` {.collection-method}
715|Trong khi phương thức `each` gọi callback đã cho cho từng phần tử trong collection ngay lập tức, phương thức `tapEach` chỉ gọi callback đã cho khi các phần tử được lấy ra khỏi danh sách từng cái một:
716|
717|```php
718|// Chưa có gì được dump cho đến nay...
719|$lazyCollection = LazyCollection::times(INF)->tapEach(function (int $value) {
720|    dump($value);
721|});
722|
723|// Ba phần tử được dump...
724|$array = $lazyCollection->take(3)->all();
725|
726|// 1
727|// 2
728|// 3
729|```
730|
731|<a name="method-throttle"></a>
732|#### `throttle()` {.collection-method}
733|
734|Phương thức `throttle` sẽ điều tiết lazy collection sao cho mỗi giá trị được trả về sau số giây được chỉ định. Phương thức này đặc biệt hữu ích cho các tình huống bạn có thể tương tác với các API bên ngoài giới hạn tốc độ các yêu cầu đến:
735|
736|```php
737|use App\Models\User;
738|
739|User::where('vip', true)
740|    ->cursor()
741|    ->throttle(seconds: 1)
742|    ->each(function (User $user) {
743|        // Gọi API bên ngoài...
744|    });
745|```
746|
747|<a name="method-remember"></a>
748|#### `remember()` {.collection-method}
749|
750|Phương thức `remember` trả về một lazy collection mới sẽ ghi nhớ bất kỳ giá trị nào đã được liệt kê và sẽ không truy xuất chúng lại trong các lần liệt kê collection tiếp theo:
751|
752|```php
753|// Chưa có truy vấn nào được thực thi...
754|$users = User::cursor()->remember();
755|
756|// Truy vấn được thực thi...
757|// 5 người dùng đầu tiên được hydrate từ cơ sở dữ liệu...
758|$users->take(5)->all();
759|
760|// 5 người dùng đầu tiên đến từ cache của collection...
761|// Phần còn lại được hydrate từ cơ sở dữ liệu...
762|$users->take(20)->all();
763|```
764|
765|<a name="method-with-heartbeat"></a>
766|#### `withHeartbeat()` {.collection-method}
767|
768|Phương thức `withHeartbeat` cho phép bạn thực thi một callback tại các khoảng thời gian đều đặn trong khi một lazy collection đang được liệt kê. Điều này đặc biệt hữu ích cho các hoạt động chạy dài yêu cầu các nhiệm vụ bảo trì định kỳ, chẳng hạn như gia hạn khóa hoặc gửi cập nhật tiến độ:
769|
770|```php
771|use Carbon\CarbonInterval;
772|use Illuminate\Support\Facades\Cache;
773|
774|$lock = Cache::lock('generate-reports', seconds: 60 * 5);
775|
776|if ($lock->get()) {
777|    try {
778|        Report::where('status', 'pending')
779|            ->lazy()
780|            ->withHeartbeat(
781|                CarbonInterval::minutes(4),
782|                fn () => $lock->extend(CarbonInterval::minutes(5))
783|            )
784|            ->each(fn ($report) => $report->process());
785|    } finally {
786|        $lock->release();
787|    }
788|}
789|```

# Search

- [Introduction](#introduction)
    - [Full-Text Search](#introduction-full-text-search)
    - [Semantic / Vector Search](#introduction-semantic-vector-search)
    - [Reranking](#introduction-reranking)
    - [Scout Search Engines](#introduction-scout-search-engines)
- [Full-Text Search](#full-text-search)
    - [Adding Full-Text Indexes](#adding-full-text-indexes)
    - [Running Full-Text Queries](#running-full-text-queries)
- [Semantic / Vector Search](#semantic-vector-search)
    - [Generating Embeddings](#generating-embeddings)
    - [Storing and Indexing Vectors](#storing-and-indexing-vectors)
    - [Querying by Similarity](#querying-by-similarity)
- [Reranking Results](#reranking-results)
- [Laravel Scout](#laravel-scout)
    - [Database Engine](#database-engine)
    - [Third-Party Engines](#third-party-engines)
- [Combining Techniques](#combining-techniques)

<a name="introduction"></a>
## Introduction

Hầu như mọi ứng dụng đều cần search. Cho dù người dùng của bạn đang tìm kiếm một knowledge base cho các bài viết liên quan, khám phá một danh mục sản phẩm, hoặc đặt câu hỏi ngôn ngữ tự nhiên đối với một corpus tài liệu, Laravel cung cấp các công cụ tích hợp để xử lý từng trường hợp này — và bạn thường không cần bất kỳ dịch vụ bên ngoài nào để đạt được điều đó.

Hầu hết các ứng dụng sẽ thấy rằng các tùy chọn dựa trên database tích hợp được cung cấp bởi Laravel là quá đủ — các dịch vụ search bên ngoài chỉ cần thiết khi bạn cần các tính năng như dung sai lỗi đánh máy, lọc faceted, hoặc geo-search ở quy mô lớn.

<a name="introduction-full-text-search"></a>
#### Full-Text Search

Khi bạn cần xếp hạng relevance từ khóa — nơi database đánh điểm và sắp xếp kết quả dựa trên mức độ phù hợp với các từ khóa tìm kiếm — phương thức query builder `whereFullText` của Laravel tận dụng các full-text indexes tích hợp trên MariaDB, MySQL, và PostgreSQL. Full-text search hiểu ranh giới từ và stemming, vì vậy một tìm kiếm "running" có thể khớp với các bản ghi chứa "run". Không cần dịch vụ bên ngoài.

<a name="introduction-semantic-vector-search"></a>
#### Semantic / Vector Search

Để tìm kiếm ngữ nghĩa AI khớp kết quả theo *ý nghĩa* thay vì từ khóa chính xác, phương thức query builder `whereVectorSimilarTo` sử dụng vector embeddings được lưu trữ trong PostgreSQL với extension `pgvector`. Ví dụ, một tìm kiếm "best wineries in Napa Valley" có thể hiển thị một bài viết có tiêu đề "Top Vineyards to Visit" — ngay cả khi các từ không trùng nhau. Vector search yêu cầu PostgreSQL với extension `pgvector` và [Laravel AI SDK](/docs/{{version}}/ai-sdk).

<a name="introduction-reranking"></a>
#### Reranking

[Laravel AI SDK](/docs/{{version}}/ai-sdk) của Laravel cung cấp các khả năng reranking sử dụng các AI models để sắp xếp lại bất kỳ tập hợp kết quả nào theo relevance ngữ nghĩa với một query. Reranking đặc biệt mạnh mẽ như một giai đoạn thứ hai sau một bước truy xuất nhanh như full-text search — cung cấp cho bạn cả tốc độ và độ chính xác ngữ nghĩa.

<a name="introduction-scout-search-engines"></a>
#### Laravel Scout Search

Đối với các ứng dụng muốn một trait `Searchable` tự động giữ các search indexes đồng bộ với các Eloquent models, [Laravel Scout](/docs/{{version}}/scout) cung cấp cả một database engine tích hợp và các drivers cho các dịch vụ bên thứ ba như Algolia, Meilisearch, và Typesense.

<a name="full-text-search"></a>
## Full-Text Search

Trong khi các truy vấn `LIKE` hoạt động tốt cho việc khớp substring đơn giản, chúng không hiểu ngôn ngữ. Một tìm kiếm `LIKE` cho "running" sẽ không tìm thấy một bản ghi chứa "run", và kết quả không được xếp hạng theo relevance — chúng chỉ đơn giản được trả về theo bất kỳ thứ tự nào mà database tìm thấy chúng. Full-text search giải quyết cả hai vấn đề này bằng cách sử dụng các indexes chuyên biệt hiểu ranh giới từ, stemming, và xếp hạng relevance, cho phép database trả về các kết quả phù hợp nhất trước.

Full-text search nhanh được tích hợp vào MariaDB, MySQL, và PostgreSQL — không cần dịch vụ search bên ngoài. Bạn chỉ cần thêm một full-text index vào các cột bạn muốn tìm kiếm, và sau đó sử dụng phương thức query builder `whereFullText` để tìm kiếm chúng.

> [!WARNING]
> Full-text search hiện được hỗ trợ bởi MariaDB, MySQL, và PostgreSQL.

<a name="adding-full-text-indexes"></a>
### Adding Full-Text Indexes

Để sử dụng full-text search, trước tiên hãy thêm một full-text index vào các cột bạn muốn tìm kiếm. Bạn có thể thêm index vào một cột duy nhất, hoặc chuyển một array các cột để tạo một index composite tìm kiếm qua nhiều trường cùng một lúc:

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('body');
    $table->timestamps();

    $table->fullText(['title', 'body']);
});
```

Trên PostgreSQL, bạn có thể chỉ định một cấu hình ngôn ngữ cho index, điều này kiểm soát cách các từ được stemmed:

```php
$table->fullText('body')->language('english');
```

Để biết thêm thông tin về việc tạo indexes, hãy tham khảo tài liệu [migration](/docs/{{version}}/migrations#available-index-types).

<a name="running-full-text-queries"></a>
### Running Full-Text Queries

Sau khi index đã được đặt, sử dụng phương thức query builder `whereFullText` để tìm kiếm nó. Laravel sẽ tạo SQL thích hợp cho database driver của bạn — ví dụ, `MATCH(...) AGAINST(...)` trên MariaDB và MySQL, và `to_tsvector(...) @@ plainto_tsquery(...)` trên PostgreSQL:

```php
$articles = Article::whereFullText('body', 'web developer')->get();
```

Khi sử dụng MariaDB và MySQL, kết quả được tự động sắp xếp theo relevance score. Trên PostgreSQL, `whereFullText` lọc các bản ghi khớp nhưng không sắp xếp chúng theo relevance — nếu bạn cần sắp xếp relevance tự động trên PostgreSQL, hãy cân nhắc sử dụng [database engine của Scout](#database-engine), xử lý điều này cho bạn.

Nếu bạn đã tạo một full-text index composite qua nhiều cột, bạn có thể tìm kiếm tất cả chúng bằng cách chuyển cùng một array các cột cho `whereFullText`:

```php
$articles = Article::whereFullText(
    ['title', 'body'], 'web developer'
)->get();
```

Phương thức `orWhereFullText` có thể được sử dụng để thêm một mệnh đề full-text search như một điều kiện "or". Để biết chi tiết đầy đủ, hãy tham khảo tài liệu [query builder](/docs/{{version}}/queries#full-text-where-clauses).

<a name="semantic-vector-search"></a>
## Semantic / Vector Search

Full-text search dựa vào việc khớp từ khóa — các từ trong query phải xuất hiện (dưới một số dạng nào đó) trong dữ liệu. Semantic search tiếp cận một cách khác nhau: nó sử dụng vector embeddings được tạo bởi AI để đại diện cho *ý nghĩa* của văn bản dưới dạng các mảng số, và sau đó tìm các kết quả có ý nghĩa gần nhất với query. Ví dụ, một tìm kiếm "best wineries in Napa Valley" có thể hiển thị một bài viết có tiêu đề "Top Vineyards to Visit" — ngay cả khi các từ không trùng nhau chút nào.

Quy trình cơ bản cho vector search là: tạo một embedding (mảng số) cho mỗi phần nội dung và lưu trữ nó cùng với dữ liệu của bạn, sau đó tại thời điểm tìm kiếm, tạo một embedding cho query của người dùng và tìm các embeddings được lưu trữ gần nhất với nó trong không gian vector.

> [!NOTE]
> Vector search yêu cầu một database PostgreSQL với extension `pgvector` và [Laravel AI SDK](/docs/{{version}}/ai-sdk). Tất cả các databases Serverless Postgres của [Laravel Cloud](https://cloud.laravel.com) đã bao gồm `pgvector`.

<a name="generating-embeddings"></a>
### Generating Embeddings

Một embedding là một mảng số chiều cao (thường là hàng trăm hoặc hàng nghìn số) đại diện cho ý nghĩa ngữ nghĩa của một phần văn bản. Bạn có thể tạo embeddings cho một chuỗi bằng cách sử dụng phương thức `toEmbeddings` có sẵn trên lớp `Stringable` của Laravel:

```php
use Illuminate\Support\Str;

$embedding = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

Để tạo embeddings cho nhiều đầu vào cùng một lúc — hiệu quả hơn việc tạo chúng từng cái một vì nó chỉ cần một cuộc gọi API duy nhất đến nhà cung cấp embedding — sử dụng lớp `Embeddings`:

```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```

Để biết thêm chi tiết về việc cấu hình các nhà cung cấp embedding, tùy chỉnh kích thước, và caching, hãy tham khảo tài liệu [AI SDK](/docs/{{version}}/ai-sdk#embeddings).

<a name="storing-and-indexing-vectors"></a>
### Storing and Indexing Vectors

Để lưu trữ vector embeddings, định nghĩa một cột `vector` trong migration của bạn, chỉ định số chiều phù hợp với đầu ra của nhà cung cấp embedding của bạn (ví dụ, 1536 cho model `text-embedding-3-small` của OpenAI). Bạn cũng nên gọi `index` trên cột để tạo một index HNSW (Hierarchical Navigable Small World), tăng tốc đáng kể các tìm kiếm similarity trên các tập dữ liệu lớn:

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536)->index();
    $table->timestamps();
});
```

Phương thức `Schema::ensureVectorExtensionExists` đảm bảo extension `pgvector` được bật trên database PostgreSQL của bạn trước khi tạo table.

Trên Eloquent model của bạn, cast cột vector thành một `array` để Laravel tự động xử lý chuyển đổi giữa các PHP arrays và định dạng vector của database:

```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```

Để biết thêm chi tiết về các cột và indexes vector, hãy tham khảo tài liệu [migration](/docs/{{version}}/migrations#available-column-types).

<a name="querying-by-similarity"></a>
### Querying by Similarity

Sau khi bạn đã lưu trữ embeddings cho nội dung của mình, bạn có thể tìm kiếm các bản ghi tương tự bằng cách sử dụng phương thức `whereVectorSimilarTo`. Phương thức này so sánh embedding đã cho với các vectors được lưu trữ bằng cách sử dụng cosine similarity, lọc ra các kết quả dưới ngưỡng `minSimilarity`, và tự động sắp xếp kết quả theo relevance — với các bản ghi tương tự nhất trước. Ngưỡng nên là một giá trị giữa `0.0` và `1.0`, nơi `1.0` có nghĩa là các vectors giống hệt nhau:

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

Để thuận tiện, khi một chuỗi đơn giản được đưa thay vì một mảng embedding, Laravel sẽ tự động tạo embedding cho bạn bằng cách sử dụng nhà cung cấp embedding được cấu hình của bạn. Điều này có nghĩa là bạn có thể chuyển query tìm kiếm của người dùng trực tiếp mà không cần chuyển đổi thủ công thành embedding trước:

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

Để kiểm soát cấp độ thấp hơn các truy vấn vector, các phương thức `whereVectorDistanceLessThan`, `selectVectorDistance`, và `orderByVectorDistance` cũng có sẵn. Các phương thức này cho phép bạn làm việc trực tiếp với các giá trị khoảng cách thay vì điểm similarity, chọn khoảng cách được tính toán như một cột trong kết quả của bạn, hoặc kiểm soát thủ công việc sắp xếp. Để biết chi tiết đầy đủ, hãy tham khảo tài liệu [query builder](/docs/{{version}}/queries#vector-similarity-clauses) và tài liệu [AI SDK](/docs/{{version}}/ai-sdk#querying-embeddings).

<a name="reranking-results"></a>
## Reranking Results

Reranking là một kỹ thuật trong đó một AI model sắp xếp lại một tập hợp kết quả theo mức độ relevance ngữ nghĩa của mỗi kết quả với một query nhất định. Không giống như vector search, yêu cầu bạn tính toán trước và lưu trữ embeddings, reranking hoạt động trên bất kỳ tập hợp văn bản nào — nó lấy nội dung thô và query làm đầu vào và trả về các mục được sắp xếp theo relevance.

Reranking đặc biệt mạnh mẽ như một giai đoạn thứ hai sau một bước truy xuất nhanh. Ví dụ, bạn có thể sử dụng full-text search để nhanh chóng thu hẹp hàng nghìn bản ghi xuống 50 ứng viên hàng đầu, và sau đó sử dụng reranking để đưa các kết quả phù hợp nhất lên đầu. Mô hình "retrieve then rerank" này cung cấp cho bạn cả tốc độ và độ chính xác ngữ nghĩa.

Bạn có thể rerank một array các chuỗi bằng cách sử dụng lớp `Reranking`:

```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

$response->first()->document; // "Laravel is a PHP web application framework."
```

Các collections của Laravel cũng có một macro `rerank` chấp nhận một tên trường (hoặc closure) và một query, giúp dễ dàng rerank các kết quả Eloquent:

```php
$articles = Article::all()
    ->rerank('body', 'Laravel tutorials');
```

Để biết chi tiết đầy đủ về việc cấu hình các nhà cung cấp reranking và các tùy chọn có sẵn, hãy tham khảo tài liệu [AI SDK](/docs/{{version}}/ai-sdk#reranking).

<a name="laravel-scout"></a>
## Laravel Scout

Các kỹ thuật tìm kiếm được mô tả ở trên đều là các phương thức query builder mà bạn gọi trực tiếp trong code của mình. [Laravel Scout](/docs/{{version}}/scout) tiếp cận một cách khác: nó cung cấp một trait `Searchable` mà bạn thêm vào các Eloquent models của bạn, và Scout tự động giữ các search indexes của bạn đồng bộ khi các bản ghi được tạo, cập nhật, và xóa. Điều này đặc biệt thuận tiện khi bạn muốn các models của bạn luôn có thể tìm kiếm mà không cần quản lý thủ công các cập nhật index.

<a name="database-engine"></a>
### Database Engine

Database engine tích hợp của Scout thực hiện các tìm kiếm full-text và `LIKE` đối với database hiện có của bạn — không cần dịch vụ bên ngoài hoặc hạ tầng bổ sung. Chỉ cần thêm trait `Searchable` vào model của bạn và định nghĩa một phương thức `toSearchableArray` trả về các cột bạn muốn có thể tìm kiếm.

Bạn có thể sử dụng các PHP attributes để kiểm soát chiến lược tìm kiếm cho mỗi cột. `SearchUsingFullText` sẽ sử dụng full-text index của database, `SearchUsingPrefix` sẽ chỉ khớp từ đầu chuỗi (`example%`), và bất kỳ cột nào không có attribute sử dụng chiến lược `LIKE` mặc định với wildcards ở cả hai phía (`%example%`):

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;
use Laravel\Scout\Searchable;

class Article extends Model
{
    use Searchable;

    #[SearchUsingPrefix(['id'])]
    #[SearchUsingFullText(['title', 'body'])]
    public function toSearchableArray(): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
        ];
    }
}
```

> [!WARNING]
> Trước khi chỉ định rằng một cột nên sử dụng các ràng buộc truy vấn full-text, hãy đảm bảo rằng cột đã được gán một [full-text index](/docs/{{version}}/migrations#available-index-types).

Sau khi trait đã được thêm, bạn có thể tìm kiếm model của bạn bằng cách sử dụng phương thức `search` của Scout. Database engine của Scout sẽ tự động sắp xếp kết quả theo relevance, ngay cả trên PostgreSQL:

```php
$articles = Article::search('Laravel')->get();
```

Database engine là một lựa chọn tuyệt vời khi nhu cầu tìm kiếm của bạn vừa phải và bạn muốn sự thuận tiện của việc đồng bộ hóa index tự động của Scout mà không cần triển khai một dịch vụ bên ngoài. Nó xử lý hầu hết các trường hợp sử dụng tìm kiếm phổ biến tốt, bao gồm lọc, phân trang, và xử lý các bản ghi soft-deleted. Để biết chi tiết đầy đủ, hãy tham khảo tài liệu [Scout](/docs/{{version}}/scout#database-engine).

<a name="third-party-engines"></a>
### Third-Party Engines

Scout cũng hỗ trợ các search engines bên thứ ba như [Algolia](https://www.algolia.com/), [Meilisearch](https://www.meilisearch.com), và [Typesense](https://typesense.org). Các dịch vụ search chuyên dụng này cung cấp các tính năng nâng cao như dung sai lỗi đánh máy, lọc faceted, geo-search, và các quy tắc xếp hạng tùy chỉnh — các tính năng trở nên quan trọng ở quy mô rất lớn hoặc khi bạn cần trải nghiệm search-as-you-type được đánh bóng cao.

Vì Scout cung cấp một API thống nhất trên tất cả các drivers của nó, chuyển từ database engine sang một engine bên thứ ba sau này chỉ cần thay đổi code tối thiểu. Bạn có thể bắt đầu với database engine và chuyển sang một dịch vụ bên thứ ba chỉ khi nhu cầu của ứng dụng vượt quá những gì database có thể cung cấp.

Để biết chi tiết đầy đủ về việc cấu hình các engines bên thứ ba, hãy tham khảo tài liệu [Scout](/docs/{{version}}/scout).

> [!NOTE]
> Nhiều ứng dụng không bao giờ cần một search engine bên ngoài. Các kỹ thuật tích hợp được mô tả trên trang này bao phủ phần lớn các trường hợp sử dụng.

<a name="combining-techniques"></a>
## Combining Techniques

Các kỹ thuật tìm kiếm được mô tả trên trang này không loại trừ lẫn nhau — kết hợp chúng thường tạo ra kết quả tốt nhất. Dưới đây là hai mô hình phổ biến thể hiện cách các công cụ này hoạt động cùng nhau.

**Full-Text Retrieval + Reranking**

Sử dụng full-text search để nhanh chóng thu hẹp một tập dữ liệu lớn xuống một tập ứng viên, sau đó áp dụng reranking để sắp xếp các ứng viên đó theo relevance ngữ nghĩa. Điều này cung cấp cho bạn tốc độ của full-text search gốc database với độ chính xác của xếp hạng relevance dựa trên AI:

```php
$articles = Article::query()
    ->whereFullText('body', $request->input('query'))
    ->limit(50)
    ->get()
    ->rerank('body', $request->input('query'), limit: 10);
```

**Vector Search + Traditional Filters**

Kết hợp vector similarity với các mệnh đề `where` tiêu chuẩn để scope tìm kiếm ngữ nghĩa cho một tập hợp con của các bản ghi. Điều này hữu ích khi bạn muốn tìm kiếm dựa trên ý nghĩa nhưng cần giới hạn kết quả theo quyền sở hữu, danh mục, hoặc bất kỳ thuộc tính nào khác:

```php
$documents = Document::query()
    ->where('team_id', $user->team_id)
    ->whereVectorSimilarTo('embedding', $request->input('query'))
    ->limit(10)
    ->get();
```

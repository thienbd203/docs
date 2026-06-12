# Release Notes

- [Versioning Scheme](#versioning-scheme)
- [Support Policy](#support-policy)
- [Laravel 13](#laravel-13)

<a name="versioning-scheme"></a>
## Versioning Scheme

Laravel và các packages chính thức khác của nó tuân theo [Semantic Versioning](https://semver.org). Các bản phát hành framework chính được phát hành hàng năm (~Q1), trong khi các bản phát hành nhỏ và bản vá có thể được phát hành thường xuyên nhất là mỗi tuần. Các bản phát hành nhỏ và bản vá không bao giờ chứa các breaking changes.

Khi tham chiếu đến Laravel framework hoặc các thành phần của nó từ ứng dụng hoặc package của bạn, bạn nên luôn sử dụng một ràng buộc phiên bản như `^13.0`, vì các bản phát hành chính của Laravel bao gồm các breaking changes. Tuy nhiên, chúng tôi luôn nỗ lực đảm bảo bạn có thể cập nhật lên một bản phát hành chính mới trong một ngày hoặc ít hơn.

<a name="named-arguments"></a>
#### Named Arguments

[Named arguments](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) không được bao gồm trong các hướng dẫn tương thích ngược của Laravel. Chúng tôi có thể chọn đổi tên các đối số hàm khi cần thiết để cải thiện codebase của Laravel. Do đó, sử dụng named arguments khi gọi các phương thức Laravel nên được thực hiện một cách thận trọng và với sự hiểu biết rằng tên tham số có thể thay đổi trong tương lai.

<a name="support-policy"></a>
## Support Policy

Đối với tất cả các bản phát hành Laravel, các bản sửa lỗi được cung cấp trong 18 tháng và các bản sửa lỗi bảo mật được cung cấp trong 2 năm. Đối với tất cả các thư viện bổ sung, chỉ bản phát hành chính mới nhất nhận được các bản sửa lỗi. Ngoài ra, hãy xem xét các phiên bản database [được hỗ trợ bởi Laravel](/docs/{{version}}/database#introduction).

<div class="overflow-auto">

|| Version | PHP (*)   | Release             | Bug Fixes Until     | Security Fixes Until |
|| ------- |-----------| ------------------- | ------------------- | -------------------- |
|| 10      | 8.1 - 8.3 | February 14th, 2023 | August 6th, 2024    | February 4th, 2025   |
|| 11      | 8.2 - 8.4 | March 12th, 2024    | September 3rd, 2025 | March 12th, 2026     |
|| 12      | 8.2 - 8.5 | February 24th, 2025 | August 13th, 2026   | February 24th, 2027  |
|| 13      | 8.3 - 8.5 | March 17th, 2026    | Q3 2027             | March 17th, 2028     |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>End of life</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Security fixes only</div>
    </div>
</div>

(*) Supported PHP versions

<a name="laravel-13"></a>
## Laravel 13

Laravel 13 tiếp tục nhịp phát hành hàng năm của Laravel với trọng tâm vào các workflows AI-native, các mặc định mạnh hơn, và các APIs nhà phát triển biểu đạt hơn. Bản phát hành này bao gồm các primitives AI chính thức, resources JSON:API, các khả năng tìm kiếm ngữ nghĩa / vector, và các cải tiến gia tăng trên queues, cache, và bảo mật.

<a name="minimal-breaking-changes"></a>
### Minimal Breaking Changes

Phần lớn trọng tâm của chúng tôi trong chu kỳ phát hành này là giảm thiểu các breaking changes. Thay vào đó, chúng tôi đã cống hiến bản thân để vận chuyển các cải tiến chất lượng cuộc sống liên tục trong suốt năm không làm hỏng các ứng dụng hiện có.

Do đó, bản phát hành Laravel 13 là một nâng cấp tương đối nhỏ về mặt nỗ lực, trong khi vẫn cung cấp các khả năng mới đáng kể. Nhìn vào điều này, hầu hết các ứng dụng Laravel có thể nâng cấp lên Laravel 13 mà không cần thay đổi nhiều mã ứng dụng.

<a name="php-8"></a>
### PHP 8.3

Laravel 13.x yêu cầu phiên bản PHP tối thiểu là 8.3.

<a name="ai-sdk"></a>
### Laravel AI SDK

Laravel 13 giới thiệu [Laravel AI SDK](https://laravel.com/ai) chính thức, cung cấp một API thống nhất cho tạo văn bản, các agents tool-calling, embeddings, audio, hình ảnh, và các tích hợp vector-store.

Với AI SDK, bạn có thể xây dựng các tính năng AI không phụ thuộc nhà cung cấp trong khi giữ trải nghiệm nhà phát triển gốc của Laravel nhất quán.

Ví dụ, một agent cơ bản có thể được prompted với một cuộc gọi đơn:

```php
use App\Ai\Agents\SalesCoach;

$response = SalesCoach::make()->prompt('Analyze this sales transcript...');

return (string) $response;
```

Laravel AI SDK cũng có thể tạo hình ảnh, audio, và embeddings:

Đối với các trường hợp sử dụng tạo hình ảnh, SDK cung cấp một API sạch để tạo hình ảnh từ các prompts ngôn ngữ đơn giản:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

Đối với các trải nghiệm giọng nói, bạn có thể tổng hợp audio nghe tự nhiên từ văn bản cho các assistants, kể chuyện, và các tính năng khả năng truy cập:

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

Và cho các workflows tìm kiếm và truy xuất ngữ nghĩa, bạn có thể tạo embeddings trực tiếp từ các chuỗi:

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

<a name="json-api"></a>
### JSON:API Resources

Laravel hiện bao gồm [resources JSON:API](/docs/{{version}}/eloquent-resources#jsonapi-resources) chính thức, giúp dễ dàng trả về các phản hồi tuân theo specification JSON:API.

Resources JSON:API xử lý serialization đối tượng resource, bao gồm relationship, sparse fieldsets, links, và các headers phản hồi tuân theo JSON:API.

<a name="request-forgery-protection"></a>
### Request Forgery Protection

Để bảo mật, middleware [bảo vệ request forgery](/docs/{{version}}/csrf#preventing-csrf-requests) của Laravel đã được nâng cấp và chính thức hóa như `PreventRequestForgery`, thêm xác minh request có nhận biết origin trong khi giữ tương thích với bảo vệ CSRF dựa trên token.

<a name="queue-routing"></a>
### Queue Routing

Laravel 13 thêm [routing queue theo class](/docs/{{version}}/queues#queue-routing) thông qua `Queue::route(...)`, cho phép bạn định nghĩa các quy tắc routing queue / connection mặc định cho các jobs cụ thể ở một nơi trung tâm:

```php
Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
```

<a name="php-attributes"></a>
### Expanded PHP Attributes

Laravel 13 tiếp tục mở rộng hỗ trợ attribute PHP chính thức trên toàn bộ framework, làm cho các mối quan tâm cấu hình và hành vi phổ biến trở nên khai báo và colocated hơn với các classes và phương thức của bạn.

Các bổ sung đáng chú ý bao gồm các attributes controller và authorization như [`#[Middleware]`](/docs/{{version}}/controllers#controller-middleware) và [`#[Authorize]`](/docs/{{version}}/controllers#authorization-attributes), cũng như các điều khiển job hướng tới queue như [`#[Tries]`](/docs/{{version}}/queues#max-job-attempts-and-timeout), [`#[Backoff]`](/docs/{{version}}/queues#dealing-with-failed-jobs), [`#[Timeout]`](/docs/{{version}}/queues#max-job-attempts-and-timeout), và [`#[FailOnTimeout]`](/docs/{{version}}/queues#failing-on-timeout).

Ví dụ, middleware controller và kiểm tra policy hiện có thể được khai báo trực tiếp trên các classes và phương thức:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Routing\Attributes\Controllers\Authorize;
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class CommentController
{
    #[Middleware('subscribed')]
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post)
    {
        // ...
    }
}
```

Các attributes bổ sung cũng đã được giới thiệu trên các APIs serialization resource, validation, testing, và events, cung cấp cho bạn một tùy chọn ưu tiên attribute trong nhiều khu vực hơn của framework.

<a name="cache-touch"></a>
### Cache TTL Extension

Laravel hiện bao gồm [`Cache::touch(...)`](/docs/{{version}}/cache), cho phép bạn mở rộng TTL của một mục cache hiện có mà không cần truy xuất và lưu trữ lại giá trị của nó.

<a name="semantic-search"></a>
### Semantic / Vector Search

Laravel 13 làm sâu câu chuyện tìm kiếm ngữ nghĩa của mình với hỗ trợ truy vấn vector gốc, các workflows embedding, và các APIs liên quan được tài liệu hóa trên [search](/docs/{{version}}/search#semantic-vector-search), [queries](/docs/{{version}}/queries#vector-similarity-clauses), và [AI SDK](/docs/{{version}}/ai-sdk#embeddings).

Các tính năng này giúp dễ dàng xây dựng các trải nghiệm tìm kiếm được hỗ trợ bởi AI bằng cách sử dụng PostgreSQL + `pgvector`, bao gồm tìm kiếm tương tự đối với các embeddings được tạo trực tiếp từ các chuỗi.

Ví dụ, bạn có thể chạy các tìm kiếm tương tự ngữ nghĩa trực tiếp từ query builder:

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

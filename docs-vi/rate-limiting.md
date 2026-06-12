# Giới hạn tốc độ

- [Giới thiệu](#introduction)
    - [Cấu hình Cache](#cache-configuration)
- [Sử dụng cơ bản](#basic-usage)
    - [Tăng số lần thử thủ công](#manually-incrementing-attempts)
    - [Xóa số lần thử](#clearing-attempts)

<a name="introduction"></a>
## Giới thiệu

Laravel bao gồm một trừu tượng giới hạn tốc độ dễ sử dụng, kết hợp với [cache](cache) của ứng dụng, cung cấp một cách dễ dàng để giới hạn bất kỳ hành động nào trong một khoảng thời gian cụ thể.

> [!NOTE]
> Nếu bạn quan tâm đến việc giới hạn tốc độ cho các yêu cầu HTTP đến, vui lòng tham khảo [tài liệu middleware giới hạn tốc độ](/docs/{{version}}/routing#rate-limiting).

<a name="cache-configuration"></a>
### Cấu hình Cache

Thông thường, bộ giới hạn tốc độ sử dụng cache mặc định của ứng dụng như được định nghĩa bởi khóa `default` trong file cấu hình `cache` của ứng dụng. Tuy nhiên, bạn có thể chỉ định driver cache mà bộ giới hạn tốc độ nên sử dụng bằng cách định nghĩa một khóa `limiter` trong file cấu hình `cache` của ứng dụng:

```php
'default' => env('CACHE_STORE', 'database'),

'limiter' => 'redis', // [tl! add]
```

<a name="basic-usage"></a>
## Sử dụng cơ bản

Facade `Illuminate\Support\Facades\RateLimiter` có thể được sử dụng để tương tác với bộ giới hạn tốc độ. Phương thức đơn giản nhất được cung cấp bởi bộ giới hạn tốc độ là phương thức `attempt`, giới hạn tốc độ cho một callback cụ thể trong một số giây nhất định.

Phương thức `attempt` trả về `false` khi callback không còn số lần thử nào; ngược lại, phương thức `attempt` sẽ trả về kết quả của callback hoặc `true`. Đối số đầu tiên được chấp nhận bởi phương thức `attempt` là một "khóa" của bộ giới hạn tốc độ, có thể là bất kỳ chuỗi nào bạn chọn để đại diện cho hành động đang bị giới hạn tốc độ:

```php
use Illuminate\Support\Facades\RateLimiter;

$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perMinute = 5,
    function() {
        // Send message...
    }
);

if (! $executed) {
    return 'Too many messages sent!';
}
```

Nếu cần thiết, bạn có thể cung cấp đối số thứ tư cho phương thức `attempt`, đó là "tốc độ giảm", hoặc số giây cho đến khi các lần thử có sẵn được đặt lại. Ví dụ, chúng ta có thể sửa đổi ví dụ trên để cho phép năm lần thử mỗi hai phút:

```php
$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perTwoMinutes = 5,
    function() {
        // Send message...
    },
    $decayRate = 120,
);
```

<a name="manually-incrementing-attempts"></a>
### Tăng số lần thử thủ công

Nếu bạn muốn tương tác thủ công với bộ giới hạn tốc độ, có nhiều phương thức khác có sẵn. Ví dụ, bạn có thể gọi phương thức `tooManyAttempts` để xác định xem một khóa bộ giới hạn tốc độ cụ thể có vượt quá số lần thử tối đa được phép mỗi phút hay không:

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    return 'Too many attempts!';
}

RateLimiter::increment('send-message:'.$user->id);

// Send message...
```

Khi giới hạn tốc độ cho một endpoint có thể nhận nhiều yêu cầu đồng thời, bạn có thể muốn kiểm tra giá trị trả về bởi phương thức `increment` thay vì sử dụng `tooManyAttempts` và `increment` như các thao tác riêng biệt. Khi sử dụng các kho cache `redis`, `memcached`, hoặc `database`, giá trị này được tăng lên một cách nguyên tử, đảm bảo mỗi yêu cầu đồng thời nhận được một số đếm duy nhất:

```php
use Illuminate\Support\Facades\RateLimiter;

$perMinute = 5;

if (RateLimiter::increment('send-message:'.$user->id) > $perMinute) {
    return 'Too many attempts!';
}

// Send message...
```

Ngoài ra, bạn có thể sử dụng phương thức `remaining` để lấy số lần thử còn lại cho một khóa cụ thể. Nếu một khóa cụ thể còn có lần thử, bạn có thể gọi phương thức `increment` để tăng số lần thử tổng:

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
    RateLimiter::increment('send-message:'.$user->id);

    // Send message...
}
```

Nếu bạn muốn tăng giá trị cho một khóa bộ giới hạn tốc độ cụ thể lên nhiều hơn một, bạn có thể cung cấp số lượng mong muốn cho phương thức `increment`:

```php
RateLimiter::increment('send-message:'.$user->id, amount: 5);
```

<a name="determining-limiter-availability"></a>
#### Xác định tính sẵn có của bộ giới hạn

Khi một khóa không còn lần thử nào, phương thức `availableIn` trả về số giây còn lại cho đến khi có thêm lần thử:

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    $seconds = RateLimiter::availableIn('send-message:'.$user->id);

    return 'You may try again in '.$seconds.' seconds.';
}

RateLimiter::increment('send-message:'.$user->id);

// Send message...
```

<a name="clearing-attempts"></a>
### Xóa số lần thử

Bạn có thể đặt lại số lần thử cho một khóa bộ giới hạn tốc độ cụ thể bằng phương thức `clear`. Ví dụ, bạn có thể đặt lại số lần thử khi một tin nhắn cụ thể được đọc bởi người nhận:

```php
use App\Models\Message;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Mark the message as read.
 */
public function read(Message $message): Message
{
    $message->markAsRead();

    RateLimiter::clear('send-message:'.$message->user_id);

    return $message;
}
```

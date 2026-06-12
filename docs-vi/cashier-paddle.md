# Laravel Cashier (Paddle)

- [Giới thiệu](#introduction)
- [Nâng cấp Cashier](#upgrading-cashier)
- [Cài đặt](#installation)
    - [Paddle Sandbox](#paddle-sandbox)
- [Cấu hình](#configuration)
    - [Mô hình Billable](#billable-model)
    - [Khóa API](#api-keys)
    - [Paddle JS](#paddle-js)
    - [Cấu hình Tiền tệ](#currency-configuration)
    - [Ghi đè Mô hình Mặc định](#overriding-default-models)
- [Bắt đầu nhanh](#quickstart)
    - [Bán Sản phẩm](#quickstart-selling-products)
    - [Bán Đăng ký](#quickstart-selling-subscriptions)
- [Phiên Thanh toán](#checkout-sessions)
    - [Overlay Checkout](#overlay-checkout)
    - [Inline Checkout](#inline-checkout)
    - [Thanh toán Khách](#guest-checkouts)
- [Xem trước Giá](#price-previews)
    - [Xem trước Giá Khách hàng](#customer-price-previews)
    - [Giảm giá](#price-discounts)
- [Khách hàng](#customers)
    - [Mặc định Khách hàng](#customer-defaults)
    - [Lấy Khách hàng](#retrieving-customers)
    - [Tạo Khách hàng](#creating-customers)
- [Đăng ký](#subscriptions)
    - [Tạo Đăng ký](#creating-subscriptions)
    - [Kiểm tra Trạng thái Đăng ký](#checking-subscription-status)
    - [Phí Đơn lẻ Đăng ký](#subscription-single-charges)
    - [Cập nhật Thông tin Thanh toán](#updating-payment-information)
    - [Thay đổi Gói](#changing-plans)
    - [Số lượng Đăng ký](#subscription-quantity)
    - [Đăng ký Với Nhiều Sản phẩm](#subscriptions-with-multiple-products)
    - [Nhiều Đăng ký](#multiple-subscriptions)
    - [Tạm dừng Đăng ký](#pausing-subscriptions)
    - [Hủy Đăng ký](#canceling-subscriptions)
- [Thử nghiệm Đăng ký](#subscription-trials)
    - [Với Phương thức Thanh toán Trước](#with-payment-method-up-front)
    - [Không Phương thức Thanh toán Trước](#without-payment-method-up-front)
    - [Mở rộng hoặc Kích hoạt Thử nghiệm](#extend-or-activate-a-trial)
- [Xử lý Webhook Paddle](#handling-paddle-webhooks)
    - [Định nghĩa Xử lý sự kiện Webhook](#defining-webhook-event-handlers)
    - [Xác minh Chữ ký Webhook](#verifying-webhook-signatures)
- [Phí Đơn lẻ](#single-charges)
    - [Thu phí cho Sản phẩm](#charging-for-products)
    - [Hoàn tiền Giao dịch](#refunding-transactions)
    - [Tín dụng Giao dịch](#crediting-transactions)
- [Giao dịch](#transactions)
    - [Thanh toán Quá khứ và Sắp tới](#past-and-upcoming-payments)
- [Kiểm thử](#testing)

<a name="introduction"></a>
## Giới thiệu

> [!WARNING]
> Tài liệu này dành cho tích hợp Cashier Paddle 2.x với Paddle Billing. Nếu bạn vẫn đang sử dụng Paddle Classic, bạn nên sử dụng [Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x).

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) cung cấp một giao diện diễn đạt, trôi chảy cho các dịch vụ thanh toán đăng ký của [Paddle](https://paddle.com). Nó xử lý hầu hết tất cả các mã thanh toán đăng ký mẫu mà bạn đang lo ngại. Ngoài việc quản lý đăng ký cơ bản, Cashier có thể xử lý: đổi đăng ký, "số lượng" đăng ký, tạm dừng đăng ký, thời gian ân hạn hủy bỏ, và nhiều hơn nữa.

Trước khi đi sâu vào Cashier Paddle, chúng tôi khuyên bạn cũng nên xem lại các [hướng dẫn khái niệm](https://developer.paddle.com/concepts/overview) và [tài liệu API](https://developer.paddle.com/api-reference/overview) của Paddle.

<a name="upgrading-cashier"></a>
## Nâng cấp Cashier

Khi nâng cấp lên phiên bản mới của Cashier, điều quan trọng là bạn phải xem xét kỹ [hướng dẫn nâng cấp](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md).

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt gói Cashier cho Paddle bằng trình quản lý gói Composer:

```shell
composer require laravel/cashier-paddle
```

Tiếp theo, bạn nên xuất bản các tệp migration của Cashier bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

Sau đó, bạn nên chạy các migration cơ sở dữ liệu của ứng dụng. Các migration của Cashier sẽ tạo một bảng `customers` mới. Ngoài ra, các bảng `subscriptions` và `subscription_items` mới sẽ được tạo để lưu trữ tất cả các đăng ký của khách hàng. Cuối cùng, một bảng `transactions` mới sẽ được tạo để lưu trữ tất cả các giao dịch Paddle liên quan đến khách hàng của bạn:

```shell
php artisan migrate
```

> [!WARNING]
> Để đảm bảo Cashier xử lý đúng tất cả các sự kiện Paddle, hãy nhớ [thiết lập xử lý webhook của Cashier](#handling-paddle-webhooks).

<a name="paddle-sandbox"></a>
### Paddle Sandbox

Trong quá trình phát triển cục bộ và staging, bạn nên [đăng ký tài khoản Paddle Sandbox](https://sandbox-login.paddle.com/signup). Tài khoản này sẽ cung cấp cho bạn một môi trường sandbox để kiểm tra và phát triển ứng dụng của mình mà không cần thực hiện thanh toán thực tế. Bạn có thể sử dụng các [số thẻ kiểm tra](https://developer.paddle.com/concepts/payment-methods/credit-debit-card#test-payment-method) của Paddle để mô phỏng các tình huống thanh toán khác nhau.

Khi sử dụng môi trường Paddle Sandbox, bạn nên đặt biến môi trường `PADDLE_SANDBOX` thành `true` trong tệp `.env` của ứng dụng:

```ini
PADDLE_SANDBOX=true
```

Sau khi bạn đã hoàn thành việc phát triển ứng dụng, bạn có thể [đăng ký tài khoản nhà cung cấp Paddle](https://paddle.com). Trước khi ứng dụng của bạn được đưa vào sản xuất, Paddle sẽ cần phê duyệt tên miền của ứng dụng.

<a name="configuration"></a>
## Cấu hình

<a name="billable-model"></a>
### Mô hình Billable

Trước khi sử dụng Cashier, bạn phải thêm trait `Billable` vào định nghĩa mô hình người dùng của mình. Trait này cung cấp nhiều phương thức khác nhau để cho phép bạn thực hiện các tác vụ thanh toán phổ biến, chẳng hạn như tạo đăng ký và cập nhật thông tin phương thức thanh toán:

```php
use Laravel\Paddle\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Nếu bạn có các thực thể có thể tính phí không phải là người dùng, bạn cũng có thể thêm trait vào các lớp đó:

```php
use Illuminate\Database\Eloquent\Model;
use Laravel\Paddle\Billable;

class Team extends Model
{
    use Billable;
}
```

<a name="api-keys"></a>
### Khóa API

Tiếp theo, bạn nên cấu hình các khóa Paddle của mình trong tệp `.env` của ứng dụng. Bạn có thể lấy các khóa API Paddle của mình từ bảng điều khiển Paddle:

```ini
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

Biến môi trường `PADDLE_SANDBOX` nên được đặt thành `true` khi bạn đang sử dụng [môi trường Sandbox của Paddle](#paddle-sandbox). Biến `PADDLE_SANDBOX` nên được đặt thành `false` nếu bạn đang triển khai ứng dụng của mình lên sản xuất và đang sử dụng môi trường nhà cung cấp trực tiếp của Paddle.

`PADDLE_RETAIN_KEY` là tùy chọn và chỉ nên được đặt nếu bạn đang sử dụng Paddle với [Retain](https://developer.paddle.com/concepts/retain/overview).

<a name="paddle-js"></a>
### Paddle JS

Paddle dựa vào thư viện JavaScript của riêng mình để khởi tạo widget thanh toán Paddle. Bạn có thể tải thư viện JavaScript bằng cách đặt directive Blade `@paddleJS` ngay trước thẻ đóng `</head>` của bố cục ứng dụng:

```blade
<head>
    ...

    @paddleJS
</head>
```

<a name="currency-configuration"></a>
### Cấu hình Tiền tệ

Bạn có thể chỉ định một locale để sử dụng khi định dạng các giá trị tiền tệ để hiển thị trên hóa đơn. Nội bộ, Cashier sử dụng [lớp `NumberFormatter` của PHP](https://www.php.net/manual/en/class.numberformatter.php) để đặt locale tiền tệ:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]
> Để sử dụng các locale khác `en`, hãy đảm bảo rằng phần mở rộng PHP `ext-intl` được cài đặt và cấu hình trên máy chủ của bạn.

<a name="overriding-default-models"></a>
### Ghi đè Mô hình Mặc định

Bạn có thể mở rộng các mô hình được sử dụng nội bộ bởi Cashier bằng cách định nghĩa mô hình của riêng bạn và mở rộng mô hình Cashier tương ứng:

```php
use Laravel\Paddle\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

Sau khi định nghĩa mô hình của bạn, bạn có thể hướng dẫn Cashier sử dụng mô hình tùy chỉnh của bạn thông qua lớp `Laravel\Paddle\Cashier`. Thông thường, bạn nên thông báo cho Cashier về các mô hình tùy chỉnh của mình trong phương thức `boot` của lớp `App\Providers\AppServiceProvider` của ứng dụng:

```php
use App\Models\Cashier\Subscription;
use App\Models\Cashier\Transaction;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useTransactionModel(Transaction::class);
}
```

<a name="quickstart"></a>
## Bắt đầu nhanh

<a name="quickstart-selling-products"></a>
### Bán Sản phẩm

> [!NOTE]
> Trước khi sử dụng Paddle Checkout, bạn nên định nghĩa Sản phẩm với giá cố định trong bảng điều khiển Paddle của mình. Ngoài ra, bạn nên [cấu hình xử lý webhook của Paddle](#handling-paddle-webhooks).

Cung cấp thanh toán sản phẩm và đăng ký thông qua ứng dụng của bạn có thể đáng sợ. Tuy nhiên, nhờ Cashier và [Checkout Overlay của Paddle](https://developer.paddle.com/concepts/sell/overlay-checkout), bạn có thể dễ dàng xây dựng các tích hợp thanh toán hiện đại, mạnh mẽ.

Để tính phí cho khách hàng cho các sản phẩm phí đơn lẻ không định kỳ, chúng ta sẽ sử dụng Cashier để tính phí cho khách hàng với Checkout Overlay của Paddle, nơi họ sẽ cung cấp chi tiết thanh toán và xác nhận mua hàng của mình. Sau khi thanh toán đã được thực hiện thông qua Checkout Overlay, khách hàng sẽ được chuyển hướng đến URL thành công mà bạn chọn trong ứng dụng:

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout('pri_deluxe_album')
        ->returnTo(route('dashboard'));

    return view('buy', ['checkout' => $checkout]);
})->name('checkout');
```

Như bạn có thể thấy trong ví dụ trên, chúng ta sẽ sử dụng phương thức `checkout` được cung cấp bởi Cashier để tạo một đối tượng checkout để hiển thị cho khách hàng Checkout Overlay của Paddle cho một "giá định danh" nhất định. Khi sử dụng Paddle, "giá" đề cập đến [giá đã định nghĩa cho các sản phẩm cụ thể](https://developer.paddle.com/build/products/create-products-prices).

Nếu cần thiết, phương thức `checkout` sẽ tự động tạo một khách hàng trong Paddle và kết nối bản ghi khách hàng Paddle đó với người dùng tương ứng trong cơ sở dữ liệu của ứng dụng. Sau khi hoàn thành phiên checkout, khách hàng sẽ được chuyển hướng đến một trang thành công chuyên biệt nơi bạn có thể hiển thị thông tin cho khách hàng.

Trong view `buy`, chúng ta sẽ bao gồm một nút để hiển thị Checkout Overlay. Thành phần Blade `paddle-button` được bao gồm với Cashier Paddle; tuy nhiên, bạn cũng có thể [tự động hiển thị một overlay checkout](#manually-rendering-an-overlay-checkout):

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy Product
</x-paddle-button>
```

<a name="providing-meta-data-to-paddle-checkout"></a>
#### Cung cấp Meta Data cho Paddle Checkout

Khi bán sản phẩm, việc theo dõi các đơn hàng đã hoàn thành và sản phẩm đã mua thông qua các mô hình `Cart` và `Order` được định nghĩa bởi ứng dụng của bạn là phổ biến. Khi chuyển hướng khách hàng đến Checkout Overlay của Paddle để hoàn tất mua hàng, bạn có thể cần cung cấp một định danh đơn hàng hiện có để bạn có thể liên kết mua hàng đã hoàn thành với đơn hàng tương ứng khi khách hàng được chuyển hướng trở lại ứng dụng của bạn.

Để thực hiện điều này, bạn có thể cung cấp một mảng dữ liệu tùy chỉnh cho phương thức `checkout`. Hãy tưởng tượng rằng một `Order` đang chờ xử lý được tạo trong ứng dụng của chúng ta khi người dùng bắt đầu quy trình checkout. Hãy nhớ rằng, các mô hình `Cart` và `Order` trong ví dụ này mang tính minh họa và không được cung cấp bởi Cashier. Bạn có thể tự do thực hiện các khái niệm này dựa trên nhu cầu của ứng dụng riêng:

```php
use App\Models\Cart;
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/cart/{cart}/checkout', function (Request $request, Cart $cart) {
    $order = Order::create([
        'cart_id' => $cart->id,
        'price_ids' => $cart->price_ids,
        'status' => 'incomplete',
    ]);

    $checkout = $request->user()->checkout($order->price_ids)
        ->customData(['order_id' => $order->id]);

    return view('billing', ['checkout' => $checkout]);
})->name('checkout');
```

Như bạn có thể thấy trong ví dụ trên, khi người dùng bắt đầu quy trình checkout, chúng ta sẽ cung cấp tất cả các định danh giá Paddle liên quan đến giỏ hàng / đơn hàng cho phương thức `checkout`. Tất nhiên, ứng dụng của bạn có trách nhiệm liên kết các mục này với "giỏ hàng" hoặc đơn hàng khi khách hàng thêm chúng. Chúng ta cũng cung cấp ID của đơn hàng cho Checkout Overlay của Paddle thông qua phương thức `customData`.

Tất nhiên, bạn có thể muốn đánh dấu đơn hàng là "hoàn thành" sau khi khách hàng đã hoàn thành quy trình checkout. Để thực hiện điều này, bạn có thể lắng nghe các webhook được gửi bởi Paddle và được kích hoạt thông qua các sự kiện bởi Cashier để lưu trữ thông tin đơn hàng trong cơ sở dữ liệu của bạn.

Để bắt đầu, hãy lắng nghe sự kiện `TransactionCompleted` được gửi bởi Cashier. Thông thường, bạn nên đăng ký trình lắng nghe sự kiện trong phương thức `boot` của `AppServiceProvider` của ứng dụng:

```php
use App\Listeners\CompleteOrder;
use Illuminate\Support\Facades\Event;
use Laravel\Paddle\Events\TransactionCompleted;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(TransactionCompleted::class, CompleteOrder::class);
}
```

Trong ví dụ này, trình lắng nghe `CompleteOrder` có thể trông như sau:

```php
namespace App\Listeners;

use App\Models\Order;
use Laravel\Paddle\Cashier;
use Laravel\Paddle\Events\TransactionCompleted;

class CompleteOrder
{
    /**
     * Handle the incoming Cashier webhook event.
     */
    public function handle(TransactionCompleted $event): void
    {
        $orderId = $event->payload['data']['custom_data']['order_id'] ?? null;

        $order = Order::findOrFail($orderId);

        $order->update(['status' => 'completed']);
    }
}
```

Vui lòng tham khảo tài liệu của Paddle để biết thêm thông tin về [dữ liệu chứa trong sự kiện `transaction.completed`](https://developer.paddle.com/webhooks/transactions/transaction-completed).

<a name="quickstart-selling-subscriptions"></a>
### Bán Đăng ký

> [!NOTE]
> Trước khi sử dụng Paddle Checkout, bạn nên định nghĩa Sản phẩm với giá cố định trong bảng điều khiển Paddle của mình. Ngoài ra, bạn nên [cấu hình xử lý webhook của Paddle](#handling-paddle-webhooks).

Cung cấp thanh toán sản phẩm và đăng ký thông qua ứng dụng của bạn có thể đáng sợ. Tuy nhiên, nhờ Cashier và [Checkout Overlay của Paddle](https://developer.paddle.com/concepts/sell/overlay-checkout), bạn có thể dễ dàng xây dựng các tích hợp thanh toán hiện đại, mạnh mẽ.

Để tìm hiểu cách bán đăng ký sử dụng Cashier và Checkout Overlay của Paddle, let's consider the simple scenario of a subscription service with a basic monthly (`price_basic_monthly`) and yearly (`price_basic_yearly`) plan. These two prices could be grouped under a "Basic" product (`pro_basic`) in our Paddle dashboard. Ngoài ra, our subscription service might offer an "Expert" plan as `pro_expert`.

Đầu tiên, hãy khám phá cách một khách hàng có thể đăng ký dịch vụ của chúng ta. Tất nhiên, you can imagine the customer might click a "subscribe" button for the Basic plan on our application's pricing page. This button will invoke a Paddle Checkout Overlay for their chosen plan. Để bắt đầu, let's initiate a checkout session via the `checkout` method:

```php
use Illuminate\Http\Request;

Route::get('/subscribe', function (Request $request) {
    $checkout = $request->user()->checkout('price_basic_monthly')
        ->returnTo(route('dashboard'));

    return view('subscribe', ['checkout' => $checkout]);
})->name('subscribe');
```

Trong view `subscribe`, chúng ta sẽ bao gồm một nút để hiển thị Checkout Overlay. Thành phần Blade `paddle-button` được bao gồm với Cashier Paddle; tuy nhiên, bạn cũng có thể [tự động hiển thị một overlay checkout](#manually-rendering-an-overlay-checkout):

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Bây giờ, khi nút Đăng ký được nhấp, the customer will be able to enter their payment details and initiate their subscription. To know when their subscription has actually started (since some payment methods require a few seconds to process), you should also [configure Cashier's webhook handling](#handling-paddle-webhooks).

Bây giờ khách hàng có thể bắt đầu đăng ký, chúng ta cần hạn chế một số phần của ứng dụng so that only subscribed users can access them. Tất nhiên, we can always determine a user's current subscription status via the `subscribed` method provided by Cashier's `Billable` trait:

```blade
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

Chúng ta có thể even easily determine if a user is subscribed to specific product or price:

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### Building a Đăng kýd Middleware

Để thuận tiện, bạn có thể muốn tạo một [middleware](/docs/{{version}}/middleware) which determines if the incoming request is from a subscribed user. Sau khi this middleware has been defined, you may easily assign it to a route to prevent users that are not subscribed from accessing the route:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            // Chuyển hướng người dùng đến trang thanh toán và yêu cầu họ đăng ký...
            return redirect('/subscribe');
        }

        return $next($request);
    }
}
```

Sau khi middleware đã được định nghĩa, bạn có thể gán nó cho một route:

```php
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### Cho phép Khách hàng Quản lý Gói Thanh toán của họ

Tất nhiên, customers may want to change their subscription plan to another product or "tier". In our example from above, we'd want to allow the customer to change their plan from a monthly subscription to a yearly subscription. For this you'll need to implement something like a button that leads to the below route:

```php
use Illuminate\Http\Request;

Route::put('/subscription/{price}/swap', function (Request $request, $price) {
    $user->subscription()->swap($price); // Với "$price" là "price_basic_yearly" cho ví dụ này.

    return redirect()->route('dashboard');
})->name('subscription.swap');
```

Ngoài việc đổi gói, bạn cũng cần cho phép khách hàng của mình hủy đăng ký. Giống như đổi gói, hãy cung cấp một nút dẫn đến route sau:

```php
use Illuminate\Http\Request;

Route::put('/subscription/cancel', function (Request $request, $price) {
    $user->subscription()->cancel();

    return redirect()->route('dashboard');
})->name('subscription.cancel');
```

Và bây giờ đăng ký của bạn sẽ bị hủy vào cuối kỳ thanh toán.

> [!NOTE]
> Miễn là bạn đã cấu hình xử lý webhook của Cashier, Cashier sẽ tự động giữ cho các bảng cơ sở dữ liệu liên quan đến Cashier của ứng dụng được đồng bộ bằng cách kiểm tra các webhook đến từ Paddle. Vì vậy, ví dụ, khi bạn hủy đăng ký của khách hàng thông qua bảng điều khiển của Paddle, Cashier sẽ nhận webhook tương ứng và đánh dấu đăng ký là "đã hủy" trong cơ sở dữ liệu của ứng dụng.

<a name="checkout-sessions"></a>
## Phiên Thanh toán

Hầu hết các hoạt động để tính phí khách hàng được thực hiện bằng cách sử dụng "checkouts" thông qua [widget Checkout Overlay](https://developer.paddle.com/build/checkout/build-overlay-checkout) của Paddle hoặc bằng cách sử dụng [inline checkout](https://developer.paddle.com/build/checkout/build-branded-inline-checkout).

Trước processing checkout payments using Paddle, you should define your application's [default payment link](https://developer.paddle.com/build/transactions/default-payment-link#set-default-link) in your Paddle checkout settings dashboard.

<a name="overlay-checkout"></a>
### Overlay Checkout

Trước displaying the Checkout Overlay widget, you must generate a checkout session using Cashier. A checkout session will inform the checkout widget of the billing operation that should be performed:

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Cashier includes a `paddle-button` [Blade component](/docs/{{version}}/blade#components). Bạn có thể pass the checkout session to this component as a "prop". Sau đó, when this button is clicked, Paddle's checkout widget will be displayed:

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

By default, this will display the widget using Paddle's default styling. Bạn có thể customize the widget by adding [Paddle supported attributes](https://developer.paddle.com/paddlejs/html-data-attributes) like the  `data-theme='light'` attribute to the component:

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4" data-theme="light">
    Subscribe
</x-paddle-button>
```

The Paddle checkout widget is asynchronous. Sau khi the user creates a subscription within the widget, Paddle will send your application a webhook so that you may properly update the subscription state in your application's database. Do đó, it's important that you properly [set up webhooks](#handling-paddle-webhooks) to accommodate for state changes from Paddle.

> [!WARNING]
> Sau a subscription state change, the delay for receiving the corresponding webhook is typically minimal but you should account for this in your application by considering that your user's subscription might not be immediately available after completing the checkout.

<a name="manually-rendering-an-overlay-checkout"></a>
#### Tự động Hiển thị Overlay Checkout

Bạn có thể also manually render an overlay checkout without using Laravel's built-in Blade components. Để bắt đầu, generate the checkout session [as demonstrated in previous examples](#overlay-checkout):

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Tiếp theo, you may use Paddle.js to initialize the checkout. In this example, we will create a link that is assigned the `paddle_button` class. Paddle.js will detect this class and display the overlay checkout when the link is clicked:

```blade
<?php
$items = $checkout->getItems();
$customer = $checkout->getCustomer();
$custom = $checkout->getCustomData();
?>

<a
    href='#!'
    class='paddle_button'
    data-items='{!! json_encode($items) !!}'
    @if ($customer) data-customer-id='{{ $customer->paddle_id }}' @endif
    @if ($custom) data-custom-data='{{ json_encode($custom) }}' @endif
    @if ($returnUrl = $checkout->getReturnUrl()) data-success-url='{{ $returnUrl }}' @endif
>
    Buy Product
</a>
```

<a name="inline-checkout"></a>
### Inline Checkout

Nếu bạn không muốn sử dụng widget checkout kiểu "overlay" của Paddle, Paddle cũng cung cấp tùy chọn để hiển thị widget inline. Mặc dù cách tiếp cận này không cho phép bạn điều chỉnh bất kỳ trường HTML nào của checkout, nó cho phép bạn nhúng widget trong ứng dụng của mình.

To make it easy for you to get started with inline checkout, Cashier includes a `paddle-checkout` Blade component. Để bắt đầu, you should [generate a checkout session](#overlay-checkout):

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Sau đó, you may pass the checkout session to the component's `checkout` attribute:

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

Để điều chỉnh chiều cao của thành phần inline checkout, bạn có thể chuyển thuộc tính `height` cho thành phần Blade:

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" height="500" />
```

Vui lòng tham khảo [hướng dẫn của Paddle về Inline Checkout](https://developer.paddle.com/build/checkout/build-branded-inline-checkout) và [các cài đặt checkout có sẵn](https://developer.paddle.com/build/checkout/set-up-checkout-default-settings) để biết thêm chi tiết về các tùy chọn tùy chỉnh của inline checkout.

<a name="manually-rendering-an-inline-checkout"></a>
#### Tự động Hiển thị Inline Checkout

Bạn có thể also manually render an inline checkout without using Laravel's built-in Blade components. Để bắt đầu, generate the checkout session [as demonstrated in previous examples](#inline-checkout):

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Tiếp theo, you may use Paddle.js to initialize the checkout. In this example, we will demonstrate this using [Alpine.js](https://github.com/alpinejs/alpine); however, you are free to modify this example for your own frontend stack:

```blade
<?php
$options = $checkout->options();

$options['settings']['frameTarget'] = 'paddle-checkout';
$options['settings']['frameInitialHeight'] = 366;
?>

<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open(@json($options));
">
</div>
```

<a name="guest-checkouts"></a>
### Thanh toán Khách

Đôi khi, bạn có thể cần tạo một phiên checkout cho người dùng không cần tài khoản với ứng dụng của bạn. Để làm điều này, bạn có thể sử dụng phương thức `guest`:

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Checkout;

Route::get('/buy', function (Request $request) {
    $checkout = Checkout::guest(['pri_34567'])
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

Sau đó, you may provide the checkout session to the [Paddle button](#overlay-checkout) or [inline checkout](#inline-checkout) Blade components.

<a name="price-previews"></a>
## Xem trước Giá

Paddle allows you to customize prices per currency, essentially allowing you to configure different prices for different countries. Cashier Paddle allows you to retrieve all of these prices using the `previewPrices` method. Phương thức này accepts the price IDs you wish to retrieve prices for:

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456']);
```

Tiền tệ sẽ được xác định dựa trên địa chỉ IP của yêu cầu; tuy nhiên, bạn có thể tùy chọn cung cấp một quốc gia cụ thể để lấy giá:

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], ['address' => [
    'country_code' => 'BE',
    'postal_code' => '1234',
]]);
```

Sau retrieving the prices you may display them however you wish:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

Bạn có thể also display the subtotal price and tax amount separately:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->subtotal() }} (+ {{ $price->tax() }} tax)</li>
    @endforeach
</ul>
```

Để biết thêm thông tin, [checkout Paddle's API documentation regarding price previews](https://developer.paddle.com/api-reference/pricing-preview/preview-prices).

<a name="customer-price-previews"></a>
### Xem trước Giá Khách hàng

Nếu người dùng đã là khách hàng và bạn muốn hiển thị các giá áp dụng cho khách hàng đó, bạn có thể làm điều đó bằng cách lấy giá trực tiếp từ thể hiện khách hàng:

```php
use App\Models\User;

$prices = User::find(1)->previewPrices(['pri_123', 'pri_456']);
```

Internally, Cashier will use the user's customer ID to retrieve the prices in their currency. So, for example, a user living in the United States will see prices in US dollars while a user in Belgium will see prices in Euros. If no matching currency can be found, the default currency of the product will be used. Bạn có thể customize all prices of a product or subscription plan in the Paddle control panel.

<a name="price-discounts"></a>
### Giảm giá

Bạn có thể also choose to display prices after a discount. When calling the `previewPrices` method, you provide the discount ID via the `discount_id` option:

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], [
    'discount_id' => 'dsc_123'
]);
```

Sau đó, display the calculated prices:

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

<a name="customers"></a>
## Khách hàng

<a name="customer-defaults"></a>
### Mặc định Khách hàng

Cashier allows you to define some useful defaults for your customers when creating checkout sessions. Setting these defaults allow you to pre-fill a customer's email address and name so that they can immediately move on to the payment portion of the checkout widget. Bạn có thể set these defaults by overriding the following methods on your billable model:

```php
/**
 * Get the customer's name to associate with Paddle.
 */
public function paddleName(): string|null
{
    return $this->name;
}

/**
 * Get the customer's email address to associate with Paddle.
 */
public function paddleEmail(): string|null
{
    return $this->email;
}
```

Các mặc định này sẽ được sử dụng cho mọi hoạt động trong Cashier tạo [phiên checkout](#checkout-sessions).

<a name="retrieving-customers"></a>
### Lấy Khách hàng

Bạn có thể retrieve a customer by their Paddle Customer ID using the `Cashier::findBillable` method. Phương thức này will return an instance of the billable model:

```php
use Laravel\Paddle\Cashier;

$user = Cashier::findBillable($customerId);
```

<a name="creating-customers"></a>
### Tạo Khách hàng

Occasionally, you may wish to create a Paddle customer without beginning a subscription. Bạn có thể accomplish this using the `createAsCustomer` method:

```php
$customer = $user->createAsCustomer();
```

An instance of `Laravel\Paddle\Customer` is returned. Sau khi the customer has been created in Paddle, you may begin a subscription at a later date. Bạn có thể provide an optional `$options` array to pass in any additional [customer creation parameters that are supported by the Paddle API](https://developer.paddle.com/api-reference/customers/create-customer):

```php
$customer = $user->createAsCustomer($options);
```

<a name="subscriptions"></a>
## Đăng ký

<a name="creating-subscriptions"></a>
### Tạo Đăng ký

To create a subscription, first retrieve an instance of your billable model from your database, which will typically be an instance of `App\Models\User`. Sau khi you have retrieved the model instance, you may use the `subscribe` method to create the model's checkout session:

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

The first argument given to the `subscribe` method is the specific price the user is subscribing to. This value should correspond to the price's identifier in Paddle. The `returnTo` method accepts a URL that your user will be redirected to after they successfully complete the checkout. The second argument passed to the `subscribe` method should be the internal "type" of the subscription. If your application only offers a single subscription, you might call this `default` or `primary`. This subscription type is only for internal application usage and is not meant to be displayed to users. Ngoài ra, it should not contain spaces and it should never be changed after creating the subscription.

Bạn có thể also provide an array of custom metadata regarding the subscription using the `customData` method:

```php
$checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
    ->customData(['key' => 'value'])
    ->returnTo(route('home'));
```

Sau khi a subscription checkout session has been created, the checkout session may be provided to the `paddle-button` [Blade component](#overlay-checkout) that is included with Cashier Paddle:

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

Sau the user has finished their checkout, a `subscription_created` webhook will be dispatched from Paddle. Cashier will receive this webhook and setup the subscription for your customer. In order to make sure all webhooks are properly received and handled by your application, ensure you have properly [setup webhook handling](#handling-paddle-webhooks).

<a name="checking-subscription-status"></a>
### Kiểm tra Trạng thái Đăng ký

Sau khi a user is subscribed to your application, you may check their subscription status using a variety of convenient methods. Đầu tiên, the `subscribed` method returns `true` if the user has a valid subscription, even if the subscription is currently within its trial period:

```php
if ($user->subscribed()) {
    // ...
}
```

Nếu ứng dụng của bạn cung cấp nhiều đăng ký, bạn có thể chỉ định đăng ký khi gọi phương thức `subscribed`:

```php
if ($user->subscribed('default')) {
    // ...
}
```

Phương thức `subscribed` cũng là một ứng viên tuyệt vời cho một [middleware route](/docs/{{version}}/middleware), cho phép bạn lọc quyền truy cập vào các route và controller dựa trên trạng thái đăng ký của người dùng:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsSubscribed
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->subscribed()) {
            // Người dùng này không phải là khách hàng trả tiền...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

If you would like to determine if a user is still within their trial period, you may use the `onTrial` method. Phương thức này can be useful for determining if you should display a warning to the user that they are still on their trial period:

```php
if ($user->subscription()->onTrial()) {
    // ...
}
```

Phương thức `subscribedToPrice` có thể được sử dụng để xác định xem người dùng có đăng ký một gói nhất định dựa trên ID giá Paddle nhất định hay không. Trong ví dụ này, chúng ta sẽ xác định xem đăng ký `default` của người dùng có đang đăng ký giá hàng tháng hay không:

```php
if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
    // ...
}
```

Phương thức `recurring` có thể được sử dụng để xác định xem người dùng hiện có đang ở đăng ký hoạt động và không còn trong thời gian dùng thử hoặc thời gian ân hạn hay không:

```php
if ($user->subscription()->recurring()) {
    // ...
}
```

<a name="canceled-subscription-status"></a>
#### Trạng thái Đăng ký Đã hủy

Để xác định xem người dùng có từng là người đăng ký hoạt động nhưng đã hủy đăng ký của họ hay không, bạn có thể sử dụng phương thức `canceled`:

```php
if ($user->subscription()->canceled()) {
    // ...
}
```

Bạn có thể also determine if a user has canceled their subscription, but are still on their "grace period" until the subscription fully expires. Ví dụ, if a user cancels a subscription on March 5th that was originally scheduled to expire on March 10th, the user is on their "grace period" until March 10th. Ngoài ra, the `subscribed` method will still return `true` during this time:

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

<a name="past-due-status"></a>
#### Trạng thái Quá hạn

If a payment fails for a subscription, it will be marked as `past_due`. When your subscription is in this state it will not be active until the customer has updated their payment information. Bạn có thể determine if a subscription is past due using the `pastDue` method on the subscription instance:

```php
if ($user->subscription()->pastDue()) {
    // ...
}
```

Khi đăng ký quá hạn, bạn nên hướng dẫn người dùng [cập nhật thông tin thanh toán của họ](#updating-payment-information).

Nếu bạn muốn đăng ký vẫn được coi là hợp lệ khi chúng `past_due`, bạn có thể sử dụng phương thức `keepPastDueSubscriptionsActive` được cung cấp bởi Cashier. Thông thường, phương thức này nên được gọi trong phương thức `register` của `AppServiceProvider` của bạn:

```php
use Laravel\Paddle\Cashier;

/**
 * Register any application services.
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
}
```

> [!WARNING]
> When a subscription is in a `past_due` state it cannot be changed until payment information has been updated. Do đó, the `swap` and `updateQuantity` methods will throw an exception when the subscription is in a `past_due` state.

<a name="subscription-scopes"></a>
#### Phạm vi Đăng ký

Hầu hết các trạng thái đăng ký cũng có sẵn dưới dạng phạm vi truy vấn để bạn có thể dễ dàng truy vấn cơ sở dữ liệu của mình cho các đăng ký ở trạng thái nhất định:

```php
// Lấy tất cả các đăng ký hợp lệ...
$subscriptions = Subscription::query()->valid()->get();

// Lấy tất cả các đăng ký đã hủy cho một người dùng...
$subscriptions = $user->subscriptions()->canceled()->get();
```

Danh sách đầy đủ các phạm vi có sẵn có sẵn dưới đây:

```php
Subscription::query()->valid();
Subscription::query()->onTrial();
Subscription::query()->expiredTrial();
Subscription::query()->notOnTrial();
Subscription::query()->active();
Subscription::query()->recurring();
Subscription::query()->pastDue();
Subscription::query()->paused();
Subscription::query()->notPaused();
Subscription::query()->onPausedGracePeriod();
Subscription::query()->notOnPausedGracePeriod();
Subscription::query()->canceled();
Subscription::query()->notCanceled();
Subscription::query()->onGracePeriod();
Subscription::query()->notOnGracePeriod();
```

<a name="subscription-single-charges"></a>
### Phí Đơn lẻ Đăng ký

Subscription single charges allow you to charge subscribers with a one-time charge on top of their subscriptions. Bạn phải provide one or multiple price ID's when invoking the `charge` method:

```php
// Tính phí một giá...
$response = $user->subscription()->charge('pri_123');

// Tính phí nhiều giá cùng lúc...
$response = $user->subscription()->charge(['pri_123', 'pri_456']);
```

Phương thức `charge` sẽ không thực sự tính phí cho khách hàng cho đến kỳ thanh toán tiếp theo của đăng ký của họ. Nếu bạn muốn tính phí cho khách hàng ngay lập tức, bạn có thể sử dụng phương thức `chargeAndInvoice` thay thế:

```php
$response = $user->subscription()->chargeAndInvoice('pri_123');
```

<a name="updating-payment-information"></a>
### Cập nhật Thông tin Thanh toán

Paddle luôn lưu một phương thức thanh toán cho mỗi đăng ký. Nếu bạn muốn cập nhật phương thức thanh toán mặc định cho đăng ký, bạn nên chuyển hướng khách hàng của mình đến trang cập nhật phương thức thanh toán được lưu trữ của Paddle bằng phương thức `redirectToUpdatePaymentMethod` trên mô hình đăng ký:

```php
use Illuminate\Http\Request;

Route::get('/update-payment-method', function (Request $request) {
    $user = $request->user();

    return $user->subscription()->redirectToUpdatePaymentMethod();
});
```

Khi người dùng đã hoàn thành việc cập nhật thông tin của họ, một webhook `subscription_updated` sẽ được gửi bởi Paddle và chi tiết đăng ký sẽ được cập nhật trong cơ sở dữ liệu của ứng dụng.

<a name="changing-plans"></a>
### Thay đổi Gói

Sau a user has subscribed to your application, they may occasionally want to change to a new subscription plan. To update the subscription plan for a user, you should pass the Paddle price's identifier to the subscription's `swap` method:

```php
use App\Models\User;

$user = User::find(1);

$user->subscription()->swap($premium = 'pri_456');
```

Nếu bạn muốn đổi gói và tính hóa đơn cho người dùng ngay lập tức thay vì chờ chu kỳ thanh toán tiếp theo của họ, bạn có thể sử dụng phương thức `swapAndInvoice`:

```php
$user = User::find(1);

$user->subscription()->swapAndInvoice($premium = 'pri_456');
```

<a name="prorations"></a>
#### Tỷ lệ

Theo mặc định, Paddle tính phí theo tỷ lệ khi đổi giữa các gói. Phương thức `noProrate` có thể được sử dụng để cập nhật đăng ký mà không tính phí theo tỷ lệ:

```php
$user->subscription('default')->noProrate()->swap($premium = 'pri_456');
```

Nếu bạn muốn vô hiệu hóa tính phí theo tỷ lệ và tính hóa đơn cho khách hàng ngay lập tức, bạn có thể sử dụng phương thức `swapAndInvoice` kết hợp với `noProrate`:

```php
$user->subscription('default')->noProrate()->swapAndInvoice($premium = 'pri_456');
```

Hoặc, để không tính phí cho khách hàng cho thay đổi đăng ký, bạn có thể sử dụng phương thức `doNotBill`:

```php
$user->subscription('default')->doNotBill()->swap($premium = 'pri_456');
```

Để biết thêm thông tin về chính sách tính phí theo tỷ lệ của Paddle, vui lòng tham khảo [tài liệu tính phí theo tỷ lệ của Paddle](https://developer.paddle.com/concepts/subscriptions/proration).

<a name="subscription-quantity"></a>
### Số lượng Đăng ký

Sometimes subscriptions are affected by "quantity". Ví dụ, a project management application might charge $10 per month per project. To easily increment or decrement your subscription's quantity, use the `incrementQuantity` and `decrementQuantity` methods:

```php
$user = User::find(1);

$user->subscription()->incrementQuantity();

// Thêm năm vào số lượng hiện tại của đăng ký...
$user->subscription()->incrementQuantity(5);

$user->subscription()->decrementQuantity();

// Trừ năm từ số lượng hiện tại của đăng ký...
$user->subscription()->decrementQuantity(5);
```

Ngoài ra, bạn có thể đặt một số lượng cụ thể bằng phương thức `updateQuantity`:

```php
$user->subscription()->updateQuantity(10);
```

Phương thức `noProrate` có thể được sử dụng để cập nhật số lượng đăng ký mà không tính phí theo tỷ lệ:

```php
$user->subscription()->noProrate()->updateQuantity(10);
```

<a name="quantities-for-subscription-with-multiple-products"></a>
#### Số lượng cho Đăng ký Với Nhiều Sản phẩm

Nếu đăng ký của bạn là [đăng ký với nhiều sản phẩm](#subscriptions-with-multiple-products), bạn nên chuyển ID của giá có số lượng bạn muốn tăng hoặc giảm làm đối số thứ hai cho các phương thức tăng / giảm:

```php
$user->subscription()->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### Đăng ký With Multiple Products

[Subscription with multiple products](https://developer.paddle.com/build/subscriptions/add-remove-products-prices-addons) allow you to assign multiple billing products to a single subscription. Ví dụ, imagine you are building a customer service "helpdesk" application that has a base subscription price of $10 per month but offers a live chat add-on product for an additional $15 per month.

Khi tạo phiên checkout đăng ký, bạn có thể chỉ định nhiều sản phẩm cho một đăng ký nhất định bằng cách chuyển một mảng giá làm đối số đầu tiên cho phương thức `subscribe`:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe([
        'price_monthly',
        'price_chat',
    ]);

    return view('billing', ['checkout' => $checkout]);
});
```

Trong ví dụ trên, the customer will have two prices attached to their `default` subscription. Both prices will be charged on their respective billing intervals. If necessary, you may pass an associative array of key / value pairs to indicate a specific quantity for each price:

```php
$user = User::find(1);

$checkout = $user->subscribe('default', ['price_monthly', 'price_chat' => 5]);
```

Nếu bạn muốn thêm một giá khác vào một đăng ký hiện có, bạn phải sử dụng phương thức `swap` của đăng ký. Khi gọi phương thức `swap`, bạn cũng nên bao gồm các giá và số lượng hiện tại của đăng ký cũng như:

```php
$user = User::find(1);

$user->subscription()->swap(['price_chat', 'price_original' => 2]);
```

Ví dụ trên sẽ thêm giá mới, nhưng khách hàng sẽ không được tính phí cho nó cho đến chu kỳ thanh toán tiếp theo của họ. Nếu bạn muốn tính phí cho khách hàng ngay lập tức, bạn có thể sử dụng phương thức `swapAndInvoice`:

```php
$user->subscription()->swapAndInvoice(['price_chat', 'price_original' => 2]);
```

Bạn có thể remove prices from subscriptions using the `swap` method and omitting the price you want to remove:

```php
$user->subscription()->swap(['price_original' => 2]);
```

> [!WARNING]
> Bạn có thể not remove the last price on a subscription. Instead, you should simply cancel the subscription.

<a name="multiple-subscriptions"></a>
### Nhiều Đăng ký

Paddle allows your customers to have multiple subscriptions simultaneously. Ví dụ, you may run a gym that offers a swimming subscription and a weight-lifting subscription, and each subscription may have different pricing. Tất nhiên, customers should be able to subscribe to either or both plans.

Khi ứng dụng của bạn tạo đăng ký, bạn có thể cung cấp loại đăng ký cho phương thức `subscribe` làm đối số thứ hai. Loại có thể là bất kỳ chuỗi nào đại diện cho loại đăng ký mà người dùng đang bắt đầu:

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($swimmingMonthly = 'pri_123', 'swimming');

    return view('billing', ['checkout' => $checkout]);
});
```

In this example, we initiated a monthly swimming subscription for the customer. Tuy nhiên, they may want to swap to a yearly subscription at a later time. When adjusting the customer's subscription, we can simply swap the price on the `swimming` subscription:

```php
$user->subscription('swimming')->swap($swimmingYearly = 'pri_456');
```

Tất nhiên, you may also cancel the subscription entirely:

```php
$user->subscription('swimming')->cancel();
```

<a name="pausing-subscriptions"></a>
### Tạm dừng Đăng ký

Để tạm dừng đăng ký, hãy gọi phương thức `pause` trên đăng ký của người dùng:

```php
$user->subscription()->pause();
```

When a subscription is paused, Cashier will automatically set the `paused_at` column in your database. This column is used to determine when the `paused` method should begin returning `true`. Ví dụ, if a customer pauses a subscription on March 1st, but the subscription was not scheduled to recur until March 5th, the `paused` method will continue to return `false` until March 5th. This is because a user is typically allowed to continue using an application until the end of their billing cycle.

Theo mặc định, việc tạm dừng diễn ra vào kỳ thanh toán tiếp theo để khách hàng có thể sử dụng phần còn lại của kỳ họ đã trả tiền. Nếu bạn muốn tạm dừng đăng ký ngay lập tức, bạn có thể sử dụng phương thức `pauseNow`:

```php
$user->subscription()->pauseNow();
```

Sử dụng phương thức `pauseUntil`, bạn có thể tạm dừng đăng ký cho đến một thời điểm cụ thể:

```php
$user->subscription()->pauseUntil(now()->plus(months: 1));
```

Hoặc, bạn có thể sử dụng phương thức `pauseNowUntil` để tạm dừng ngay lập tức đăng ký cho đến một thời điểm nhất định:

```php
$user->subscription()->pauseNowUntil(now()->plus(months: 1));
```

Bạn có thể determine if a user has paused their subscription but are still on their "grace period" using the `onPausedGracePeriod` method:

```php
if ($user->subscription()->onPausedGracePeriod()) {
    // ...
}
```

Để tiếp tục một đăng ký đã tạm dừng, bạn có thể gọi phương thức `resume` trên đăng ký:

```php
$user->subscription()->resume();
```

> [!WARNING]
> Đăng ký không thể được sửa đổi trong khi bị tạm dừng. Nếu bạn muốn đổi sang một gói khác hoặc cập nhật số lượng, bạn phải tiếp tục đăng ký trước.

<a name="canceling-subscriptions"></a>
### Hủy Đăng ký

Để hủy đăng ký, hãy gọi phương thức `cancel` trên đăng ký của người dùng:

```php
$user->subscription()->cancel();
```

When a subscription is canceled, Cashier will automatically set the `ends_at` column in your database. This column is used to determine when the `subscribed` method should begin returning `false`. Ví dụ, if a customer cancels a subscription on March 1st, but the subscription was not scheduled to end until March 5th, the `subscribed` method will continue to return `true` until March 5th. This is done because a user is typically allowed to continue using an application until the end of their billing cycle.

Bạn có thể determine if a user has canceled their subscription but are still on their "grace period" using the `onGracePeriod` method:

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

Nếu bạn muốn hủy đăng ký ngay lập tức, bạn có thể gọi phương thức `cancelNow` trên đăng ký:

```php
$user->subscription()->cancelNow();
```

Để ngăn chặn một đăng ký trong thời gian ân hạn bị hủy, bạn có thể gọi phương thức `stopCancelation`:

```php
$user->subscription()->stopCancelation();
```

> [!WARNING]
> Đăng ký của Paddle không thể được tiếp tục sau khi hủy. Nếu khách hàng của bạn muốn tiếp tục đăng ký của họ, họ sẽ phải tạo một đăng ký mới.

<a name="subscription-trials"></a>
## Thử nghiệm Đăng ký

<a name="with-payment-method-up-front"></a>
### Với Phương thức Thanh toán Trước

If you would like to offer trial periods to your customers while still collecting payment method information up front, you should use set a trial time in the Paddle dashboard on the price your customer is subscribing to. Sau đó, initiate the checkout session as normal:

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

Khi ứng dụng của bạn nhận được sự kiện `subscription_created`, Cashier sẽ đặt ngày kết thúc thời gian dùng thử trên bản ghi đăng ký trong cơ sở dữ liệu của ứng dụng của bạn cũng như hướng dẫn Paddle không bắt đầu tính phí cho khách hàng cho đến sau ngày này.

> [!WARNING]
> Nếu đăng ký của khách hàng không bị hủy trước ngày kết thúc thời gian dùng thử, họ sẽ bị tính phí ngay khi thời gian dùng thử hết hạn, vì vậy bạn nên đảm bảo thông báo cho người dùng về ngày kết thúc thời gian dùng thử của họ.

Bạn có thể determine if the user is within their trial period using either the `onTrial` method of the user instance:

```php
if ($user->onTrial()) {
    // ...
}
```

Để xác định xem thời gian dùng thử hiện có đã hết hạn hay chưa, bạn có thể sử dụng phương thức `hasExpiredTrial`:

```php
if ($user->hasExpiredTrial()) {
    // ...
}
```

Để xác định xem người dùng có đang dùng thử cho một loại đăng ký cụ thể hay không, bạn có thể cung cấp loại cho phương thức `onTrial` hoặc `hasExpiredTrial`:

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->hasExpiredTrial('default')) {
    // ...
}
```

<a name="without-payment-method-up-front"></a>
### Không Phương thức Thanh toán Trước

Nếu bạn muốn cung cấp thời gian dùng thử mà không thu thập thông tin phương thức thanh toán của người dùng trước, bạn có thể đặt cột `trial_ends_at` trên bản ghi khách hàng được gắn vào người dùng của bạn thành ngày kết thúc thời gian dùng thử mong muốn. Điều này thường được thực hiện trong quá trình đăng ký người dùng:

```php
use App\Models\User;

$user = User::create([
    // ...
]);

$user->createAsCustomer([
    'trial_ends_at' => now()->plus(days: 10)
]);
```

Cashier gọi loại thời gian dùng thử này là "thử nghiệm chung", vì nó không được gắn vào bất kỳ đăng ký hiện có nào. Phương thức `onTrial` trên thể hiện `User` sẽ trả về `true` nếu ngày hiện tại không vượt quá giá trị của `trial_ends_at`:

```php
if ($user->onTrial()) {
    // Người dùng đang trong thời gian dùng thử của họ...
}
```

Sau khi you are ready to create an actual subscription for the user, you may use the `subscribe` method as usual:

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

To retrieve the user's trial ending date, you may use the `trialEndsAt` method. Phương thức này will return a Carbon date instance if a user is on a trial or `null` if they aren't. Bạn có thể also pass an optional subscription type parameter if you would like to get the trial ending date for a specific subscription other than the default one:

```php
if ($user->onTrial('default')) {
    $trialEndsAt = $user->trialEndsAt();
}
```

Bạn có thể use the `onGenericTrial` method if you wish to know specifically that the user is within their "generic" trial period and has not created an actual subscription yet:

```php
if ($user->onGenericTrial()) {
    // Người dùng đang trong thời gian dùng thử "chung" của họ...
}
```

<a name="extend-or-activate-a-trial"></a>
### Mở rộng hoặc Kích hoạt Thử nghiệm

Bạn có thể extend an existing trial period on a subscription by invoking the `extendTrial` method and specifying the moment in time that the trial should end:

```php
$user->subscription()->extendTrial(now()->plus(days: 5));
```

Hoặc, bạn có thể kích hoạt ngay lập tức một đăng ký bằng cách kết thúc thời gian dùng thử của nó bằng cách gọi phương thức `activate` trên đăng ký:

```php
$user->subscription()->activate();
```

<a name="handling-paddle-webhooks"></a>
## Xử lý Webhook Paddle

Paddle có thể thông báo cho ứng dụng của bạn về nhiều sự kiện khác nhau thông qua webhook. Theo mặc định, một route trỏ đến bộ điều khiển webhook của Cashier được đăng ký bởi nhà cung cấp dịch vụ Cashier. Bộ điều khiển này sẽ xử lý tất cả các yêu cầu webhook đến.

Theo mặc định, bộ điều khiển này sẽ tự động xử lý hủy đăng ký có quá nhiều khoản phí không thành công, cập nhật đăng ký và thay đổi phương thức thanh toán; tuy nhiên, như chúng ta sẽ sớm khám phá, bạn có thể mở rộng bộ điều khiển này để xử lý bất kỳ sự kiện webhook Paddle nào bạn thích.

Để đảm bảo your application can handle Paddle webhooks, be sure to [configure the webhook URL in the Paddle control panel](https://vendors.paddle.com/notifications-v2). By default, Cashier's webhook controller responds to the `/paddle/webhook` URL path. The full list of all webhooks you should enable in the Paddle control panel are:

- Customer Updated
- Transaction Completed
- Transaction Updated
- Subscription Created
- Subscription Updated
- Subscription Paused
- Subscription Canceled

> [!WARNING]
> Đảm bảo bạn bảo vệ các yêu cầu đến bằng middleware [xác minh chữ ký webhook](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures) được bao gồm với Cashier.

<a name="webhooks-csrf-protection"></a>
#### Webhooks và Bảo vệ CSRF

Since Paddle webhooks need to bypass Laravel's [CSRF protection](/docs/{{version}}/csrf), you should ensure that Laravel does not attempt to verify the CSRF token for incoming Paddle webhooks. Để thực hiện điều này, you should exclude `paddle/*` from CSRF protection in your application's `bootstrap/app.php` file:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(except: [
        'paddle/*',
    ]);
})
```

<a name="webhooks-local-development"></a>
#### Webhooks và Phát triển Cục bộ

Để Paddle có thể gửi webhook cho ứng dụng của bạn trong quá trình phát triển cục bộ, bạn sẽ cần hiển thị ứng dụng của mình thông qua dịch vụ chia sẻ trang web như [Ngrok](https://ngrok.com/) hoặc [Expose](https://expose.dev/docs/introduction). Nếu bạn đang phát triển ứng dụng của mình cục bộ bằng [Laravel Sail](/docs/{{version}}/sail), bạn có thể sử dụng [lệnh chia sẻ trang web](/docs/{{version}}/sail#sharing-your-site) của Sail.

<a name="defining-webhook-event-handlers"></a>
### Định nghĩa Xử lý sự kiện Webhook

Cashier automatically handles subscription cancelation on failed charges and other common Paddle webhooks. Tuy nhiên, if you have additional webhook events you would like to handle, you may do so by listening to the following events that are dispatched by Cashier:

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

Both events contain the full payload of the Paddle webhook. Ví dụ, if you wish to handle the `transaction.billed` webhook, you may register a [listener](/docs/{{version}}/events#defining-listeners) that will handle the event:

```php
<?php

namespace App\Listeners;

use Laravel\Paddle\Events\WebhookReceived;

class PaddleEventListener
{
    /**
     * Handle received Paddle webhooks.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['event_type'] === 'transaction.billed') {
            // Xử lý sự kiện đến...
        }
    }
}
```

Cashier cũng phát ra các sự kiện dành riêng cho loại webhook nhận được. Ngoài toàn bộ payload từ Paddle, chúng cũng chứa các mô hình liên quan được sử dụng để xử lý webhook như mô hình có thể tính phí, đăng ký hoặc biên lai:

<div class="content-list" markdown="1">

- `Laravel\Paddle\Events\CustomerUpdated`
- `Laravel\Paddle\Events\TransactionCompleted`
- `Laravel\Paddle\Events\TransactionUpdated`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionPaused`
- `Laravel\Paddle\Events\SubscriptionCanceled`

</div>

Bạn có thể also override the default, built-in webhook route by defining the `CASHIER_WEBHOOK` environment variable in your application's `.env` file. This value should be the full URL to your webhook route and needs to match the URL set in your Paddle control panel:

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### Xác minh Chữ ký Webhook

Để bảo mật webhook của bạn, bạn có thể sử dụng [chữ ký webhook của Paddle](https://developer.paddle.com/webhooks/signature-verification). Để thuận tiện, Cashier tự động bao gồm một middleware xác nhận rằng yêu cầu webhook Paddle đến là hợp lệ.

Để kích hoạt xác minh webhook, hãy đảm bảo rằng biến môi trường `PADDLE_WEBHOOK_SECRET` được định nghĩa trong tệp `.env` của ứng dụng. Bí mật webhook có thể được lấy từ bảng điều khiển tài khoản Paddle của bạn.

<a name="single-charges"></a>
## Phí Đơn lẻ

<a name="charging-for-products"></a>
### Thu phí cho Sản phẩm

Nếu bạn muốn bắt đầu mua sản phẩm cho khách hàng, bạn có thể sử dụng phương thức `checkout` trên một thể hiện mô hình có thể tính phí để tạo một phiên checkout cho mua hàng. Phương thức `checkout` chấp nhận một hoặc nhiều ID giá. Nếu cần thiết, một mảng kết hợp có thể được sử dụng để cung cấp số lượng của sản phẩm đang được mua:

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout(['pri_tshirt', 'pri_socks' => 5]);

    return view('buy', ['checkout' => $checkout]);
});
```

Sau generating the checkout session, you may use Cashier's provided `paddle-button` [Blade component](#overlay-checkout) to allow the user to view the Paddle checkout widget and complete the purchase:

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy
</x-paddle-button>
```

Một phiên checkout có phương thức `customData`, cho phép bạn chuyển bất kỳ dữ liệu tùy chỉnh nào bạn muốn cho việc tạo giao dịch cơ bản. Vui lòng tham khảo [tài liệu của Paddle](https://developer.paddle.com/build/transactions/custom-data) để tìm hiểu thêm về các tùy chọn có sẵn cho bạn khi chuyển dữ liệu tùy chỉnh:

```php
$checkout = $user->checkout('pri_tshirt')
    ->customData([
        'custom_option' => $value,
    ]);
```

<a name="refunding-transactions"></a>
### Hoàn tiền Giao dịch

Refunding transactions will return the refunded amount to your customer's payment method that was used at the time of purchase. If you need to refund a Paddle purchase, you may use the `refund` method on a `Cashier\Paddle\Transaction` model. Phương thức này accepts a reason as the first argument, one or more price ID's to refund with optional amounts as an associative array. Bạn có thể retrieve the transactions for a given billable model using the `transactions` method.

Ví dụ, imagine we want to refund a specific transaction for prices `pri_123` and `pri_456`. We want to fully refund `pri_123`, but only refund two dollars for `pri_456`:

```php
use App\Models\User;

$user = User::find(1);

$transaction = $user->transactions()->first();

$response = $transaction->refund('Accidental charge', [
    'pri_123', // Hoàn lại hoàn toàn giá này...
    'pri_456' => 200, // Chỉ hoàn lại một phần giá này...
]);
```

Ví dụ trên hoàn lại các mục dòng cụ thể trong một giao dịch. Nếu bạn muốn hoàn lại toàn bộ giao dịch, chỉ cần cung cấp một lý do:

```php
$response = $transaction->refund('Accidental charge');
```

Để biết thêm thông tin về hoàn tiền, vui lòng tham khảo [tài liệu hoàn tiền của Paddle](https://developer.paddle.com/build/transactions/create-transaction-adjustments).

> [!WARNING]
> Hoàn tiền phải luôn được phê duyệt bởi Paddle trước khi xử lý hoàn toàn.

<a name="crediting-transactions"></a>
### Tín dụng Giao dịch

Giống như hoàn tiền, bạn cũng có thể tín dụng giao dịch. Tín dụng giao dịch sẽ thêm tiền vào số dư của khách hàng để nó có thể được sử dụng cho các mua hàng trong tương lai. Tín dụng giao dịch chỉ có thể được thực hiện cho các giao dịch thu thập thủ công và không phải cho các giao dịch thu thập tự động (như đăng ký) vì Paddle xử lý tín dụng đăng ký tự động:

```php
$transaction = $user->transactions()->first();

// Tín dụng hoàn toàn một mục dòng cụ thể...
$response = $transaction->credit('Compensation', 'pri_123');
```

Để biết thêm thông tin, [xem tài liệu của Paddle về tín dụng](https://developer.paddle.com/build/transactions/create-transaction-adjustments).

> [!WARNING]
> Tín dụng chỉ có thể được áp dụng cho các giao dịch thu thập thủ công. Các giao dịch thu thập tự động được tín dụng bởi chính Paddle.

<a name="transactions"></a>
## Giao dịch

Bạn có thể easily retrieve an array of a billable model's transactions via the `transactions` property:

```php
use App\Models\User;

$user = User::find(1);

$transactions = $user->transactions;
```

Giao dịch đại diện cho thanh toán cho sản phẩm và mua hàng của bạn và được đi kèm với hóa đơn. Chỉ các giao dịch đã hoàn thành được lưu trữ trong cơ sở dữ liệu của ứng dụng.

When listing the transactions for a customer, you may use the transaction instance's methods to display the relevant payment information. Ví dụ, you may wish to list every transaction in a table, allowing the user to easily download any of the invoices:

```html
<table>
    @foreach ($transactions as $transaction)
        <tr>
            <td>{{ $transaction->billed_at->toFormattedDateString() }}</td>
            <td>{{ $transaction->total() }}</td>
            <td>{{ $transaction->tax() }}</td>
            <td><a href="{{ route('download-invoice', $transaction->id) }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

Route `download-invoice` có thể trông như sau:

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Transaction;

Route::get('/download-invoice/{transaction}', function (Request $request, Transaction $transaction) {
    return $transaction->redirectToInvoicePdf();
})->name('download-invoice');
```

<a name="past-and-upcoming-payments"></a>
### Thanh toán Quá khứ và Sắp tới

Bạn có thể use the `lastPayment` and `nextPayment` methods to retrieve and display a customer's past or upcoming payments for recurring subscriptions:

```php
use App\Models\User;

$user = User::find(1);

$subscription = $user->subscription();

$lastPayment = $subscription->lastPayment();
$nextPayment = $subscription->nextPayment();
```

Cả hai phương thức này sẽ trả về một thể hiện của `Laravel\Paddle\Payment`; tuy nhiên, `lastPayment` sẽ trả về `null` khi các giao dịch chưa được đồng bộ bởi webhook, trong khi `nextPayment` sẽ trả về `null` khi chu kỳ thanh toán đã kết thúc (như khi đăng ký đã bị hủy):

```blade
Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="testing"></a>
## Kiểm thử

Trong khi kiểm thử, bạn nên kiểm tra thủ công quy trình thanh toán của mình để đảm bảo tích hợp của bạn hoạt động như mong đợi.

Để kiểm thử tự động, bao gồm cả những cái được thực thi trong môi trường CI, bạn có thể sử dụng [HTTP Client của Laravel](/docs/{{version}}/http-client#testing) để giả mạo các cuộc gọi HTTP được thực hiện cho Paddle. Mặc dù điều này không kiểm tra các phản hồi thực tế từ Paddle, nó cung cấp một cách để kiểm tra ứng dụng của bạn mà không thực sự gọi API của Paddle.

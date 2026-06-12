# Laravel Cashier (Stripe)

- [Giới thiệu](#introduction)
- [Nâng cấp Cashier](#upgrading-cashier)
- [Cài đặt](#installation)
- [Cấu hình](#configuration)
    - [Model Có thể Thanh toán](#billable-model)
    - [Khóa API](#api-keys)
    - [Cấu hình Tiền tệ](#currency-configuration)
    - [Cấu hình Thuế](#tax-configuration)
    - [Ghi nhật ký](#logging)
    - [Sử dụng Model Tùy chỉnh](#using-custom-models)
- [Bắt đầu nhanh](#quickstart)
    - [Bán Sản phẩm](#quickstart-selling-products)
    - [Bán Đăng ký](#quickstart-selling-subscriptions)
- [Khách hàng](#customers)
    - [Lấy Khách hàng](#retrieving-customers)
    - [Tạo Khách hàng](#creating-customers)
    - [Cập nhật Khách hàng](#updating-customers)
    - [Số dư](#balances)
    - [Mã số thuế](#tax-ids)
    - [Đồng bộ Dữ liệu Khách hàng Với Stripe](#syncing-customer-data-with-stripe)
    - [Cổng Thanh toán](#billing-portal)
- [Phương thức Thanh toán](#payment-methods)
    - [Lưu trữ Phương thức Thanh toán](#storing-payment-methods)
    - [Lấy Phương thức Thanh toán](#retrieving-payment-methods)
    - [Sự hiện diện của Phương thức Thanh toán](#payment-method-presence)
    - [Cập nhật Phương thức Thanh toán Mặc định](#updating-the-default-payment-method)
    - [Thêm Phương thức Thanh toán](#adding-payment-methods)
    - [Xóa Phương thức Thanh toán](#deleting-payment-methods)
- [Đăng ký](#subscriptions)
    - [Tạo Đăng ký](#creating-subscriptions)
    - [Kiểm tra Trạng thái Đăng ký](#checking-subscription-status)
    - [Thay đổi Giá](#changing-prices)
    - [Số lượng Đăng ký](#subscription-quantity)
    - [Đăng ký Với Nhiều Sản phẩm](#subscriptions-with-multiple-products)
    - [Nhiều Đăng ký](#multiple-subscriptions)
    - [Thanh toán Dựa trên Sử dụng](#usage-based-billing)
    - [Thuế Đăng ký](#subscription-taxes)
    - [Ngày neo Đăng ký](#subscription-anchor-date)
    - [Hủy Đăng ký](#cancelling-subscriptions)
    - [Tiếp tục Đăng ký](#resuming-subscriptions)
- [Thử nghiệm Đăng ký](#subscription-trials)
    - [Với Phương thức Thanh toán Trước](#with-payment-method-up-front)
    - [Không có Phương thức Thanh toán Trước](#without-payment-method-up-front)
    - [Gia hạn Thử nghiệm](#extending-trials)
- [Xử lý Webhook Stripe](#handling-stripe-webhooks)
    - [Định nghĩa Xử lý sự kiện Webhook](#defining-webhook-event-handlers)
    - [Xác minh Chữ ký Webhook](#verifying-webhook-signatures)
- [Phí Đơn lẻ](#single-charges)
    - [Phí Đơn giản](#simple-charge)
    - [Phí Với Hóa đơn](#charge-with-invoice)
    - [Tạo Payment Intents](#creating-payment-intents)
    - [Hoàn tiền Phí](#refunding-charges)
- [Hóa đơn](#invoices)
    - [Lấy Hóa đơn](#retrieving-invoices)
    - [Hóa đơn Sắp tới](#upcoming-invoices)
    - [Xem trước Hóa đơn Đăng ký](#previewing-subscription-invoices)
    - [Tạo PDF Hóa đơn](#generating-invoice-pdfs)
- [Thanh toán](#checkout)
    - [Thanh toán Sản phẩm](#product-checkouts)
    - [Thanh toán Phí Đơn lẻ](#single-charge-checkouts)
    - [Thanh toán Đăng ký](#subscription-checkouts)
    - [Thu thập Mã số thuế](#collecting-tax-ids)
    - [Thanh toán Khách](#guest-checkouts)
- [Xử lý Thanh toán Thất bại](#handling-failed-payments)
    - [Xác nhận Thanh toán](#confirming-payments)
- [Xác thực Khách hàng Mạnh (SCA)](#strong-customer-authentication)
    - [Thanh toán Yêu cầu Xác nhận Bổ sung](#payments-requiring-additional-confirmation)
    - [Thông báo Thanh toán Ngoài Phiên](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)
- [Kiểm thử](#testing)

<a name="introduction"></a>
## Giới thiệu

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) cung cấp một giao diện biểu đạt, trôi chảy cho [dịch vụ thanh toán đăng ký của Stripe](https://stripe.com). Nó xử lý hầu hết tất cả mã thanh toán đăng ký chuẩn mà bạn đang ngại viết. Ngoài việc quản lý đăng ký cơ bản, Cashier có thể xử lý phiếu giảm giá, đổi đăng ký, "số lượng" đăng ký, thời gian ân hạn hủy, và thậm chí tạo PDF hóa đơn.

<a name="upgrading-cashier"></a>
## Nâng cấp Cashier

Khi nâng cấp lên phiên bản mới của Cashier, điều quan trọng là bạn phải xem xét kỹ [hướng dẫn nâng cấp](https://github.com/laravel/cashier-stripe/blob/16.x/UPGRADE.md).

> [!WARNING]
> Để ngăn chặn các thay đổi phá vỡ, Cashier sử dụng phiên bản API Stripe cố định. Cashier 16 sử dụng phiên bản API Stripe `2025-06-30.basil`. Phiên bản API Stripe sẽ được cập nhật trong các bản phát hành nhỏ để sử dụng các tính năng và cải tiến mới của Stripe.

<a name="installation"></a>
## Cài đặt

Đầu tiên, cài đặt gói Cashier cho Stripe bằng trình quản lý gói Composer:

```shell
composer require laravel/cashier
```

Sau khi cài đặt gói, hãy xuất bản các migration của Cashier bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

Sau đó, chạy migration cơ sở dữ liệu của bạn:

```shell
php artisan migrate
```

Các migration của Cashier sẽ thêm một số cột vào bảng `users` của bạn. Chúng cũng sẽ tạo một bảng `subscriptions` mới để lưu trữ tất cả đăng ký của khách hàng và một bảng `subscription_items` cho các đăng ký có nhiều giá.

Nếu bạn muốn, bạn cũng có thể xuất bản tệp cấu hình của Cashier bằng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --tag="cashier-config"
```

Cuối cùng, để đảm bảo Cashier xử lý đúng tất cả các sự kiện Stripe, hãy nhớ [cấu hình xử lý webhook của Cashier](#handling-stripe-webhooks).

> [!WARNING]
> Stripe khuyến nghị rằng bất kỳ cột nào được sử dụng để lưu trữ định danh Stripe nên phân biệt chữ hoa/thường. Do đó, bạn nên đảm bảo rằng collation của cột cho cột `stripe_id` được đặt thành `utf8_bin` khi sử dụng MySQL. Thông tin thêm về điều này có thể được tìm thấy trong [tài liệu Stripe](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible).

<a name="configuration"></a>
## Cấu hình

<a name="billable-model"></a>
### Model Có thể Thanh toán

Trước khi sử dụng Cashier, hãy thêm trait `Billable` vào định nghĩa model có thể thanh toán của bạn. Thông thường, đây sẽ là model `App\Models\User`. Trait này cung cấp nhiều phương thức khác nhau để cho phép bạn thực hiện các tác vụ thanh toán phổ biến, chẳng hạn như tạo đăng ký, áp dụng phiếu giảm giá, và cập nhật thông tin phương thức thanh toán:

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Cashier giả định rằng model có thể thanh toán của bạn sẽ là class `App\Models\User` đi kèm với Laravel. Nếu bạn muốn thay đổi điều này, bạn có thể chỉ định một model khác thông qua phương thức `useCustomerModel`. Phương thức này thường nên được gọi trong phương thức `boot` của class `AppServiceProvider` của bạn:

```php
use App\Models\Cashier\User;
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useCustomerModel(User::class);
}
```

> [!WARNING]
> Nếu bạn đang sử dụng một model khác với model `App\Models\User` được cung cấp bởi Laravel, bạn sẽ cần xuất bản và thay đổi [các migration Cashier](#installation) được cung cấp để khớp với tên bảng của model thay thế của bạn.

<a name="api-keys"></a>
### Khóa API

Tiếp theo, bạn nên cấu hình các khóa API Stripe của bạn trong tệp `.env` của ứng dụng. Bạn có thể lấy các khóa API Stripe của mình từ bảng điều khiển Stripe:

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

> [!WARNING]
> Bạn nên đảm bảo rằng biến môi trường `STRIPE_WEBHOOK_SECRET` được định nghĩa trong tệp `.env` của ứng dụng, vì biến này được sử dụng để đảm bảo rằng các webhook đến thực sự từ Stripe.

<a name="currency-configuration"></a>
### Cấu hình Tiền tệ

Tiền tệ mặc định của Cashier là Đô la Mỹ (USD). Bạn có thể thay đổi tiền tệ mặc định bằng cách đặt biến môi trường `CASHIER_CURRENCY` trong tệp `.env` của ứng dụng:

```ini
CASHIER_CURRENCY=eur
```

Ngoài việc cấu hình tiền tệ của Cashier, bạn cũng có thể chỉ định một locale để sử dụng khi định dạng các giá trị tiền tệ để hiển thị trên hóa đơn. Bên trong, Cashier sử dụng [class `NumberFormatter` của PHP](https://www.php.net/manual/en/class.numberformatter.php) để đặt locale tiền tệ:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]
> Để sử dụng các locale khác với `en`, hãy đảm bảo rằng phần mở rộng PHP `ext-intl` được cài đặt và cấu hình trên máy chủ của bạn.

<a name="tax-configuration"></a>
### Cấu hình Thuế

Nhờ [Stripe Tax](https://stripe.com/tax), có thể tự động tính toán thuế cho tất cả các hóa đơn được tạo bởi Stripe. Bạn có thể kích hoạt tính toán thuế tự động bằng cách gọi phương thức `calculateTaxes` trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::calculateTaxes();
}
```

Sau khi tính toán thuế đã được kích hoạt, bất kỳ đăng ký mới nào và bất kỳ hóa đơn một lần nào được tạo sẽ nhận được tính toán thuế tự động.

Để tính năng này hoạt động đúng, chi tiết thanh toán của khách hàng, chẳng hạn như tên, địa chỉ, và mã số thuế của khách hàng, cần được đồng bộ với Stripe. Bạn có thể sử dụng các phương thức [đồng bộ dữ liệu khách hàng](#syncing-customer-data-with-stripe) và [Mã số thuế](#tax-ids) được cung cấp bởi Cashier để thực hiện điều này.

<a name="logging"></a>
### Ghi nhật ký

Cashier cho phép bạn chỉ định kênh nhật ký để sử dụng khi ghi nhật ký các lỗi nghiêm trọng của Stripe. Bạn có thể chỉ định kênh nhật ký bằng cách định nghĩa biến môi trường `CASHIER_LOGGER` trong tệp `.env` của ứng dụng:

```ini
CASHIER_LOGGER=stack
```

Các ngoại lệ được tạo bởi các gọi API đến Stripe sẽ được ghi nhật ký thông qua kênh nhật ký mặc định của ứng dụng.

<a name="using-custom-models"></a>
### Sử dụng Model Tùy chỉnh

Bạn có thể mở rộng các model được sử dụng bên trong bởi Cashier bằng cách định nghĩa model của riêng bạn và mở rộng model Cashier tương ứng:

```php
use Laravel\Cashier\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

Sau khi định nghĩa model của bạn, bạn có thể hướng dẫn Cashier sử dụng model tùy chỉnh của bạn thông qua class `Laravel\Cashier\Cashier`. Thông thường, bạn nên thông báo cho Cashier về các model tùy chỉnh của bạn trong phương thức `boot` của class `App\Providers\AppServiceProvider` của ứng dụng:

```php
use App\Models\Cashier\Subscription;
use App\Models\Cashier\SubscriptionItem;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useSubscriptionItemModel(SubscriptionItem::class);
}
```

<a name="quickstart"></a>
## Bắt đầu nhanh

<a name="quickstart-selling-products"></a>
### Bán Sản phẩm

> [!NOTE]
> Trước khi sử dụng Stripe Checkout, bạn nên định nghĩa Sản phẩm với giá cố định trong bảng điều khiển Stripe của bạn. Ngoài ra, bạn nên [cấu hình xử lý webhook của Cashier](#handling-stripe-webhooks).

Cung cấp thanh toán sản phẩm và đăng ký thông qua ứng dụng của bạn có thể đáng sợ. Tuy nhiên, nhờ Cashier và [Stripe Checkout](https://stripe.com/payments/checkout), bạn có thể dễ dàng xây dựng các tích hợp thanh toán hiện đại, mạnh mẽ.

Để tính phí cho khách hàng đối với các sản phẩm phí đơn lẻ không định kỳ, chúng tôi sẽ sử dụng Cashier để hướng dẫn khách hàng đến Stripe Checkout, nơi họ sẽ cung cấp chi tiết thanh toán của họ và xác nhận mua hàng của họ. Sau khi thanh toán đã được thực hiện thông qua Checkout, khách hàng sẽ được chuyển hướng đến URL thành công theo lựa chọn của bạn trong ứng dụng:

```php
use Illuminate\Http\Request;

Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';

    $quantity = 1;

    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
    ]);
})->name('checkout');

Route::view('/checkout/success', 'checkout.success')->name('checkout-success');
Route::view('/checkout/cancel', 'checkout.cancel')->name('checkout-cancel');
```

Như bạn có thể thấy trong ví dụ trên, chúng tôi sẽ sử dụng phương thức `checkout` được cung cấp bởi Cashier để chuyển hướng khách hàng đến Stripe Checkout cho một "định danh giá" nhất định. Khi sử dụng Stripe, "giá" đề cập đến [giá được định nghĩa cho các sản phẩm cụ thể](https://stripe.com/docs/products-prices/how-products-and-prices-work).

Nếu cần thiết, phương thức `checkout` sẽ tự động tạo một khách hàng trong Stripe và kết nối bản ghi khách hàng Stripe đó với người dùng tương ứng trong cơ sở dữ liệu của ứng dụng. Sau khi hoàn tất phiên checkout, khách hàng sẽ được chuyển hướng đến một trang thành công hoặc hủy bỏ chuyên biệt nơi bạn có thể hiển thị thông tin cho khách hàng.

<a name="providing-meta-data-to-stripe-checkout"></a>
#### Cung cấp Meta Data cho Stripe Checkout

Khi bán sản phẩm, việc theo dõi các đơn hàng đã hoàn tất và sản phẩm đã mua thông qua các model `Cart` và `Order` được định nghĩa bởi ứng dụng của riêng bạn là phổ biến. Khi chuyển hướng khách hàng đến Stripe Checkout để hoàn tất mua hàng, bạn có thể cần cung cấp một định danh đơn hàng hiện có để bạn có thể liên kết mua hàng đã hoàn tất với đơn hàng tương ứng khi khách hàng được chuyển hướng trở lại ứng dụng của bạn.

Để thực hiện điều này, bạn có thể cung cấp một mảng `metadata` cho phương thức `checkout`. Hãy tưởng tượng rằng một `Order` đang chờ xử lý được tạo trong ứng dụng của chúng tôi khi người dùng bắt đầu quy trình checkout. Hãy nhớ rằng, các model `Cart` và `Order` trong ví dụ này mang tính minh họa và không được cung cấp bởi Cashier. Bạn có thể tự do triển khai các khái niệm này dựa trên nhu cầu của ứng dụng riêng của bạn:

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

    return $request->user()->checkout($order->price_ids, [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
        'metadata' => ['order_id' => $order->id],
    ]);
})->name('checkout');
```

Như bạn có thể thấy trong ví dụ trên, khi người dùng bắt đầu quy trình checkout, chúng tôi sẽ cung cấp tất cả các định danh giá Stripe liên quan của giỏ hàng / đơn hàng cho phương thức `checkout`. Tất nhiên, ứng dụng của bạn chịu trách nhiệm liên kết các mục này với "giỏ hàng" hoặc đơn hàng khi khách hàng thêm chúng. Chúng tôi cũng cung cấp ID của đơn hàng cho phiên Stripe Checkout thông qua mảng `metadata`. Cuối cùng, chúng tôi đã thêm biến mẫu `CHECKOUT_SESSION_ID` vào route thành công Checkout. Khi Stripe chuyển hướng khách hàng trở lại ứng dụng của bạn, biến mẫu này sẽ tự động được điền với ID phiên Checkout.

Tiếp theo, hãy xây dựng route thành công Checkout. Đây là route mà người dùng sẽ được chuyển hướng đến sau khi mua hàng của họ đã được hoàn tất thông qua Stripe Checkout. Trong route này, chúng tôi có thể lấy ID phiên Stripe Checkout và phiên Stripe Checkout liên quan để truy cập meta data đã cung cấp của chúng tôi và cập nhật đơn hàng của khách hàng tương ứng:

```php
use App\Models\Order;
use Illuminate\Http\Request;
use Laravel\Cashier\Cashier;

Route::get('/checkout/success', function (Request $request) {
    $sessionId = $request->get('session_id');

    if ($sessionId === null) {
        return;
    }

    $session = Cashier::stripe()->checkout->sessions->retrieve($sessionId);

    if ($session->payment_status !== 'paid') {
        return;
    }

    $orderId = $session['metadata']['order_id'] ?? null;

    $order = Order::findOrFail($orderId);

    $order->update(['status' => 'completed']);

    return view('checkout-success', ['order' => $order]);
})->name('checkout-success');
```

Vui lòng tham khảo tài liệu của Stripe để biết thêm thông tin về [dữ liệu được chứa bởi đối tượng phiên Checkout](https://stripe.com/docs/api/checkout/sessions/object).

<a name="quickstart-selling-subscriptions"></a>
### Bán Đăng ký

> [!NOTE]
> Trước khi sử dụng Stripe Checkout, bạn nên định nghĩa Sản phẩm với giá cố định trong bảng điều khiển Stripe của bạn. Ngoài ra, bạn nên [cấu hình xử lý webhook của Cashier](#handling-stripe-webhooks).

Cung cấp thanh toán sản phẩm và đăng ký thông qua ứng dụng của bạn có thể đáng sợ. Tuy nhiên, nhờ Cashier và [Stripe Checkout](https://stripe.com/payments/checkout), bạn có thể dễ dàng xây dựng các tích hợp thanh toán hiện đại, mạnh mẽ.

Để tìm hiểu cách bán đăng ký sử dụng Cashier và Stripe Checkout, hãy xem xét kịch bản đơn giản của một dịch vụ đăng ký với gói hàng tháng cơ bản (`price_basic_monthly`) và hàng năm (`price_basic_yearly`). Hai giá này có thể được nhóm dưới một sản phẩm "Basic" (`pro_basic`) trong bảng điều khiển Stripe của chúng tôi. Ngoài ra, dịch vụ đăng ký của chúng tôi có thể cung cấp gói Expert là `pro_expert`.

Đầu tiên, hãy khám phá cách một khách hàng có thể đăng ký dịch vụ của chúng tôi. Tất nhiên, bạn có thể tưởng tượng khách hàng có thể nhấp vào nút "đăng ký" cho gói Basic trên trang giá của ứng dụng. Nút hoặc liên kết này nên hướng người dùng đến một route Laravel tạo phiên Stripe Checkout cho gói đã chọn của họ:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_basic_monthly')
        ->trialDays(5)
        ->allowPromotionCodes()
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

Như bạn có thể thấy trong ví dụ trên, chúng tôi sẽ chuyển hướng khách hàng đến một phiên Stripe Checkout sẽ cho phép họ đăng ký gói Basic của chúng tôi. Sau khi checkout thành công hoặc hủy bỏ, khách hàng sẽ được chuyển hướng trở lại URL mà chúng tôi đã cung cấp cho phương thức `checkout`. Để biết khi nào đăng ký của họ thực sự đã bắt đầu (vì một số phương thức thanh toán cần vài giây để xử lý), chúng tôi cũng sẽ cần [cấu hình xử lý webhook của Cashier](#handling-stripe-webhooks).

Bây giờ khách hàng có thể bắt đầu đăng ký, chúng tôi cần hạn chế một số phần của ứng dụng để chỉ người dùng đã đăng ký mới có thể truy cập. Tất nhiên, chúng tôi luôn có thể xác định trạng thái đăng ký hiện tại của người dùng thông qua phương thức `subscribed` được cung cấp bởi trait `Billable` của Cashier:

```blade
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

Chúng tôi thậm chí có thể dễ dàng xác định xem người dùng có đăng ký một sản phẩm hoặc giá cụ thể hay không:

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### Xây dựng Middleware Đã đăng ký

Để thuận tiện, bạn có thể muốn tạo một [middleware](/docs/{{version}}/middleware) xác định xem yêu cầu đến có từ người dùng đã đăng ký hay không. Sau khi middleware này đã được định nghĩa, bạn có thể dễ dàng gán nó cho một route để ngăn người dùng chưa đăng ký truy cập route:

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
            // Redirect user to billing page and ask them to subscribe...
            return redirect('/billing');
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
#### Cho phép Khách hàng Quản lý Kế hoạch Thanh toán của họ

Tất nhiên, khách hàng có thể muốn thay đổi gói đăng ký của họ sang một sản phẩm hoặc "cấp độ" khác. Cách dễ nhất để cho phép điều này là hướng dẫn khách hàng đến [Cổng Thanh toán Khách hàng của Stripe](https://stripe.com/docs/no-code/customer-portal), cung cấp một giao diện người dùng được lưu trữ cho phép khách hàng tải xuống hóa đơn, cập nhật phương thức thanh toán của họ, và thay đổi gói đăng ký.

Đầu tiên, định nghĩa một liên kết hoặc nút trong ứng dụng của bạn hướng người dùng đến một route Laravel mà chúng tôi sẽ sử dụng để khởi tạo phiên Cổng Thanh toán:

```blade
<a href="{{ route('billing') }}">
    Billing
</a>
```

Tiếp theo, hãy định nghĩa route khởi tạo phiên Cổng Thanh toán Khách hàng Stripe và chuyển hướng người dùng đến Cổng. Phương thức `redirectToBillingPortal` chấp nhận URL mà người dùng nên được trả về khi thoát khỏi Cổng:

```php
use Illuminate\Http\Request;

Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
})->middleware(['auth'])->name('billing');
```

> [!NOTE]
> Miễn là bạn đã cấu hình xử lý webhook của Cashier, Cashier sẽ tự động giữ cho các bảng cơ sở dữ liệu liên quan đến Cashier của ứng dụng của bạn được đồng bộ bằng cách kiểm tra các webhook đến từ Stripe. Vì vậy, ví dụ, khi người dùng hủy đăng ký của họ thông qua Cổng Thanh toán Khách hàng của Stripe, Cashier sẽ nhận webhook tương ứng và đánh dấu đăng ký là "đã hủy" trong cơ sở dữ liệu của ứng dụng.

<a name="customers"></a>
## Khách hàng

<a name="retrieving-customers"></a>
### Lấy Khách hàng

Bạn có thể lấy một khách hàng bằng ID Stripe của họ sử dụng phương thức `Cashier::findBillable`. Phương thức này sẽ trả về một instance của model có thể thanh toán:

```php
use Laravel\Cashier\Cashier;

$user = Cashier::findBillable($stripeId);
```

<a name="creating-customers"></a>
### Tạo Khách hàng

Thỉnh thoảng, bạn có thể muốn tạo một khách hàng Stripe mà không bắt đầu đăng ký. Bạn có thể thực hiện điều này sử dụng phương thức `createAsStripeCustomer`:

```php
$stripeCustomer = $user->createAsStripeCustomer();
```

Sau khi khách hàng đã được tạo trong Stripe, bạn có thể bắt đầu đăng ký vào một ngày sau. Bạn có thể cung cấp một mảng `$options` tùy chọn để chuyển vào bất kỳ [tham số tạo khách hàng nào được hỗ trợ bởi API Stripe](https://stripe.com/docs/api/customers/create):

```php
$stripeCustomer = $user->createAsStripeCustomer($options);
```

Bạn có thể sử dụng phương thức `asStripeCustomer` nếu bạn muốn trả về đối tượng khách hàng Stripe cho một model có thể thanh toán:

```php
$stripeCustomer = $user->asStripeCustomer();
```

Phương thức `createOrGetStripeCustomer` có thể được sử dụng nếu bạn muốn lấy đối tượng khách hàng Stripe cho một model có thể thanh toán nhất định nhưng không chắc chắn liệu model có thể thanh toán đó đã là một khách hàng trong Stripe hay chưa. Phương thức này sẽ tạo một khách hàng mới trong Stripe nếu chưa tồn tại:

```php
$stripeCustomer = $user->createOrGetStripeCustomer();
```

<a name="updating-customers"></a>
### Cập nhật Khách hàng

Thỉnh thoảng, bạn có thể muốn cập nhật khách hàng Stripe trực tiếp với thông tin bổ sung. Bạn có thể thực hiện điều này sử dụng phương thức `updateStripeCustomer`. Phương thức này chấp nhận một mảng [tùy chọn cập nhật khách hàng được hỗ trợ bởi API Stripe](https://stripe.com/docs/api/customers/update):

```php
$stripeCustomer = $user->updateStripeCustomer($options);
```

<a name="balances"></a>
### Số dư

Stripe cho phép bạn ghi có hoặc ghi nợ "số dư" của khách hàng. Sau đó, số dư này sẽ được ghi có hoặc ghi nợ trên các hóa đơn mới. Để kiểm tra số dư tổng của khách hàng, bạn có thể sử dụng phương thức `balance` có sẵn trên model có thể thanh toán của bạn. Phương thức `balance` sẽ trả về một biểu diễn chuỗi được định dạng của số dư trong tiền tệ của khách hàng:

```php
$balance = $user->balance();
```

Để ghi có số dư của khách hàng, bạn có thể cung cấp một giá trị cho phương thức `creditBalance`. Nếu bạn muốn, bạn cũng có thể cung cấp một mô tả:

```php
$user->creditBalance(500, 'Premium customer top-up.');
```

Cung cấp một giá trị cho phương thức `debitBalance` sẽ ghi nợ số dư của khách hàng:

```php
$user->debitBalance(300, 'Bad usage penalty.');
```

Phương thức `applyBalance` sẽ tạo các giao dịch số dư khách hàng mới cho khách hàng. Bạn có thể lấy các bản ghi giao dịch này sử dụng phương thức `balanceTransactions`, có thể hữu ích để cung cấp nhật ký các ghi có và ghi nợ cho khách hàng xem xét:

```php
// Retrieve all transactions...
$transactions = $user->balanceTransactions();

foreach ($transactions as $transaction) {
    // Transaction amount...
    $amount = $transaction->amount(); // $2.31

    // Retrieve the related invoice when available...
    $invoice = $transaction->invoice();
}
```

<a name="tax-ids"></a>
### Mã số thuế

Cashier cung cấp một cách dễ dàng để quản lý mã số thuế của khách hàng. Ví dụ, phương thức `taxIds` có thể được sử dụng để lấy tất cả các [mã số thuế](https://stripe.com/docs/api/customer_tax_ids/object) được gán cho một khách hàng dưới dạng một collection:

```php
$taxIds = $user->taxIds();
```

Bạn cũng có thể lấy một mã số thuế cụ thể cho một khách hàng bằng định danh của nó:

```php
$taxId = $user->findTaxId('txi_belgium');
```

Bạn có thể tạo một Mã số thuế mới bằng cách cung cấp một [loại](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type) và giá trị hợp lệ cho phương thức `createTaxId`:

```php
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
```

Phương thức `createTaxId` sẽ ngay lập tức thêm mã số VAT vào tài khoản của khách hàng. [Xác minh mã số VAT cũng được thực hiện bởi Stripe](https://stripe.com/docs/invoicing/customer/tax-ids#validation); tuy nhiên, đây là một quy trình không đồng bộ. Bạn có thể được thông báo về các cập nhật xác minh bằng cách đăng ký sự kiện webhook `customer.tax_id.updated` và kiểm tra [tham số `verification` của mã số VAT](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification). Để biết thêm thông tin về xử lý webhook, vui lòng tham khảo [tài liệu về định nghĩa trình xử lý webhook](#handling-stripe-webhooks).

Bạn có thể xóa một mã số thuế sử dụng phương thức `deleteTaxId`:

```php
$user->deleteTaxId('txi_belgium');
```

<a name="syncing-customer-data-with-stripe"></a>
### Đồng bộ Dữ liệu Khách hàng Với Stripe

Thông thường, khi người dùng của ứng dụng cập nhật tên, địa chỉ email, hoặc thông tin khác cũng được lưu trữ bởi Stripe, bạn nên thông báo cho Stripe về các cập nhật. Bằng cách làm như vậy, bản sao của thông tin của Stripe sẽ được đồng bộ với ứng dụng của bạn.

Để tự động hóa điều này, bạn có thể định nghĩa một trình lắng nghe sự kiện trên model có thể thanh toán phản ứng với sự kiện `updated` của model. Sau đó, trong trình lắng nghe sự kiện của bạn, bạn có thể gọi phương thức `syncStripeCustomerDetails` trên model:

```php
use App\Models\User;
use function Illuminate\Events\queueable;

/**
 * The "booted" method of the model.
 */
protected static function booted(): void
{
    static::updated(queueable(function (User $customer) {
        if ($customer->hasStripeId()) {
            $customer->syncStripeCustomerDetails();
        }
    }));
}
```

Bây giờ, mỗi khi model khách hàng của bạn được cập nhật, thông tin của nó sẽ được đồng bộ với Stripe. Để thuận tiện, Cashier sẽ tự động đồng bộ thông tin khách hàng của bạn với Stripe khi tạo ban đầu khách hàng.

Bạn có thể tùy chỉnh các cột được sử dụng để đồng bộ thông tin khách hàng với Stripe bằng cách ghi đè nhiều phương thức khác nhau được cung cấp bởi Cashier. Ví dụ, bạn có thể ghi đè phương thức `stripeName` để tùy chỉnh thuộc tính nên được coi là "tên" của khách hàng khi Cashier đồng bộ thông tin khách hàng với Stripe:

```php
/**
 * Get the customer name that should be synced to Stripe.
 */
public function stripeName(): string|null
{
    return $this->company_name;
}
```

Tương tự, bạn có thể ghi đè các phương thức `stripeEmail`, `stripePhone` (tối đa 20 ký tự), `stripeAddress`, và `stripePreferredLocales`. Các phương thức này sẽ đồng bộ thông tin với các tham số khách hàng tương ứng của chúng khi [cập nhật đối tượng khách hàng Stripe](https://stripe.com/docs/api/customers/update). Nếu bạn muốn kiểm soát hoàn toàn quy trình đồng bộ thông tin khách hàng, bạn có thể ghi đè phương thức `syncStripeCustomerDetails`.

<a name="billing-portal"></a>
### Cổng Thanh toán

Stripe cung cấp [một cách dễ dàng để thiết lập cổng thanh toán](https://stripe.com/docs/billing/subscriptions/customer-portal) để khách hàng của bạn có thể quản lý đăng ký, phương thức thanh toán, và xem lịch sử thanh toán của họ. Bạn có thể chuyển hướng người dùng của mình đến cổng thanh toán bằng cách gọi phương thức `redirectToBillingPortal` trên model có thể thanh toán từ một controller hoặc route:

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal();
});
```

Theo mặc định, khi người dùng hoàn tất quản lý đăng ký của họ, họ sẽ có thể quay lại route `home` của ứng dụng của bạn thông qua một liên kết trong cổng thanh toán Stripe. Bạn có thể cung cấp một URL tùy chỉnh mà người dùng nên quay lại bằng cách chuyển URL làm đối số cho phương thức `redirectToBillingPortal`:

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('billing'));
});
```

Nếu bạn muốn tạo URL đến cổng thanh toán mà không tạo phản hồi chuyển hướng HTTP, bạn có thể gọi phương thức `billingPortalUrl`:

```php
$url = $request->user()->billingPortalUrl(route('billing'));
```

<a name="payment-methods"></a>
## Phương thức Thanh toán

<a name="storing-payment-methods"></a>
### Lưu trữ Phương thức Thanh toán

Để tạo đăng ký hoặc thực hiện các khoản phí "một lần" với Stripe, bạn sẽ cần lưu trữ một phương thức thanh toán và lấy định danh của nó từ Stripe. Cách tiếp cận được sử dụng để thực hiện điều này khác nhau tùy thuộc vào việc bạn có kế hoạch sử dụng phương thức thanh toán cho đăng ký hay phí đơn lẻ, vì vậy chúng tôi sẽ xem xét cả hai bên dưới.

<a name="payment-methods-for-subscriptions"></a>
#### Phương thức Thanh toán cho Đăng ký

Khi lưu trữ thông tin thẻ tín dụng của khách hàng để sử dụng trong tương lai bởi một đăng ký, API "Setup Intents" của Stripe phải được sử dụng để thu thập an toàn chi tiết phương thức thanh toán của khách hàng. "Setup Intent" chỉ định cho Stripe ý định tính phí phương thức thanh toán của khách hàng. Trait `Billable` của Cashier bao gồm phương thức `createSetupIntent` để dễ dàng tạo một Setup Intent mới. Bạn nên gọi phương thức này từ route hoặc controller sẽ hiển thị biểu mẫu thu thập chi tiết phương thức thanh toán của khách hàng:

```php
return view('update-payment-method', [
    'intent' => $user->createSetupIntent()
]);
```

Sau khi bạn đã tạo Setup Intent và chuyển nó đến view, bạn nên đính kèm bí mật của nó vào phần tử sẽ thu thập phương thức thanh toán. Ví dụ, hãy xem xét biểu mẫu "cập nhật phương thức thanh toán" này:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

Tiếp theo, thư viện Stripe.js có thể được sử dụng để đính kèm một [Stripe Element](https://stripe.com/docs/stripe-js) vào biểu mẫu và thu thập an toàn chi tiết thanh toán của khách hàng:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Tiếp theo, thẻ có thể được xác minh và một "định danh phương thức thanh toán" an toàn có thể được lấy từ Stripe sử dụng [phương thức `confirmCardSetup` của Stripe](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

Sau khi thẻ đã được xác minh bởi Stripe, bạn có thể chuyển định danh `setupIntent.payment_method` kết quả đến ứng dụng Laravel của bạn, nơi nó có thể được đính kèm vào khách hàng. Phương thức thanh toán có thể được [thêm như một phương thức thanh toán mới](#adding-payment-methods) hoặc [sử dụng để cập nhật phương thức thanh toán mặc định](#updating-the-default-payment-method). Bạn cũng có thể ngay lập tức sử dụng định danh phương thức thanh toán để [tạo một đăng ký mới](#creating-subscriptions).

> [!NOTE]
> Nếu bạn muốn thêm thông tin về Setup Intents và thu thập chi tiết thanh toán khách hàng, vui lòng [xem tổng quan này được cung cấp bởi Stripe](https://stripe.com/docs/payments/save-and-reuse#php).

<a name="payment-methods-for-single-charges"></a>
#### Phương thức Thanh toán cho Phí Đơn lẻ

Tất nhiên, khi tính phí đơn lẻ đối với phương thức thanh toán của khách hàng, chúng tôi sẽ chỉ cần sử dụng một định danh phương thức thanh toán một lần. Do giới hạn của Stripe, bạn có thể không sử dụng phương thức thanh toán mặc định được lưu trữ của khách hàng cho các phí đơn lẻ. Bạn phải cho phép khách hàng nhập chi tiết phương thức thanh toán của họ sử dụng thư viện Stripe.js. Ví dụ, hãy xem xét biểu mẫu sau:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

Sau khi định nghĩa một biểu mẫu như vậy, thư viện Stripe.js có thể được sử dụng để đính kèm một [Stripe Element](https://stripe.com/docs/stripe-js) vào biểu mẫu và thu thập an toàn chi tiết thanh toán của khách hàng:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Tiếp theo, thẻ có thể được xác minh và một "định danh phương thức thanh toán" an toàn có thể được lấy từ Stripe sử dụng [phương thức `createPaymentMethod` của Stripe](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

Nếu thẻ được xác minh thành công, bạn có thể chuyển `paymentMethod.id` đến ứng dụng Laravel của bạn và xử lý một [phí đơn lẻ](#simple-charge).

<a name="retrieving-payment-methods"></a>
### Lấy Phương thức Thanh toán

Phương thức `paymentMethods` trên instance model có thể thanh toán trả về một collection của các instance `Laravel\Cashier\PaymentMethod`:

```php
$paymentMethods = $user->paymentMethods();
```

Theo mặc định, phương thức này sẽ trả về các phương thức thanh toán của mọi loại. Để lấy các phương thức thanh toán của một loại cụ thể, bạn có thể chuyển `type` làm đối số cho phương thức:

```php
$paymentMethods = $user->paymentMethods('sepa_debit');
```

Để lấy phương thức thanh toán mặc định của khách hàng, phương thức `defaultPaymentMethod` có thể được sử dụng:

```php
$paymentMethod = $user->defaultPaymentMethod();
```

Bạn có thể lấy một phương thức thanh toán cụ thể được đính kèm vào model có thể thanh toán sử dụng phương thức `findPaymentMethod`:

```php
$paymentMethod = $user->findPaymentMethod($paymentMethodId);
```

<a name="payment-method-presence"></a>
### Payment Method Presence

Để xác định xem một model có khả năng thanh toán có phương thức thanh toán mặc định được gắn vào tài khoản của họ hay không, hãy gọi phương thức `hasDefaultPaymentMethod`:

```php
if ($user->hasDefaultPaymentMethod()) {
    // ...
}
```

Bạn có thể sử dụng phương thức `hasPaymentMethod` để xác định xem một model có khả năng thanh toán có ít nhất một phương thức thanh toán được gắn vào tài khoản của họ hay không:

```php
if ($user->hasPaymentMethod()) {
    // ...
}
```

Phương thức này sẽ xác định xem model có khả năng thanh toán có bất kỳ phương thức thanh toán nào hay không. Để xác định xem có phương thức thanh toán của một loại cụ thể tồn tại cho model hay không, bạn có thể truyền `type` làm đối số cho phương thức:

```php
if ($user->hasPaymentMethod('sepa_debit')) {
    // ...
}
```

<a name="updating-the-default-payment-method"></a>
### Updating the Default Payment Method

Phương thức `updateDefaultPaymentMethod` có thể được sử dụng để cập nhật thông tin phương thức thanh toán mặc định của khách hàng. Phương thức này chấp nhận một định danh phương thức thanh toán của Stripe và sẽ gán phương thức thanh toán mới làm phương thức thanh toán mặc định cho hóa đơn:

```php
$user->updateDefaultPaymentMethod($paymentMethod);
```

Để đồng bộ thông tin phương thức thanh toán mặc định của bạn với thông tin phương thức thanh toán mặc định của khách hàng trong Stripe, bạn có thể sử dụng phương thức `updateDefaultPaymentMethodFromStripe`:

```php
$user->updateDefaultPaymentMethodFromStripe();
```

> [!WARNING]
> Phương thức thanh toán mặc định trên một khách hàng chỉ có thể được sử dụng để xuất hóa đơn và tạo gói đăng ký mới. Do các hạn chế do Stripe áp đặt, nó có thể không được sử dụng cho các khoản phí đơn lẻ.

<a name="adding-payment-methods"></a>
### Adding Payment Methods

Để thêm một phương thức thanh toán mới, bạn có thể gọi phương thức `addPaymentMethod` trên model có khả năng thanh toán, truyền định danh phương thức thanh toán:

```php
$user->addPaymentMethod($paymentMethod);
```

> [!NOTE]
> Để tìm hiểu cách lấy định danh phương thức thanh toán, vui lòng xem lại [tài liệu lưu trữ phương thức thanh toán](#storing-payment-methods).

<a name="deleting-payment-methods"></a>
### Deleting Payment Methods

Để xóa một phương thức thanh toán, bạn có thể gọi phương thức `delete` trên instance `Laravel\Cashier\PaymentMethod` mà bạn muốn xóa:

```php
$paymentMethod->delete();
```

Phương thức `deletePaymentMethod` sẽ xóa một phương thức thanh toán cụ thể khỏi model có khả năng thanh toán:

```php
$user->deletePaymentMethod('pm_visa');
```

Phương thức `deletePaymentMethods` sẽ xóa tất cả thông tin phương thức thanh toán của model có khả năng thanh toán:

```php
$user->deletePaymentMethods();
```

Theo mặc định, phương thức này sẽ xóa phương thức thanh toán của mọi loại. Để xóa phương thức thanh toán của một loại cụ thể, bạn có thể truyền `type` làm đối số cho phương thức:

```php
$user->deletePaymentMethods('sepa_debit');
```

> [!WARNING]
> Nếu người dùng có gói đăng ký đang hoạt động, ứng dụng của bạn không nên cho phép họ xóa phương thức thanh toán mặc định của họ.

<a name="subscriptions"></a>
## Subscriptions

Gói đăng ký cung cấp một cách để thiết lập thanh toán định kỳ cho khách hàng của bạn. Gói đăng ký Stripe được quản lý bởi Cashier cung cấp hỗ trợ cho nhiều giá gói đăng ký, số lượng gói đăng ký, bản dùng thử, và nhiều hơn nữa.

<a name="creating-subscriptions"></a>
### Creating Subscriptions

Để tạo một gói đăng ký, trước tiên hãy lấy một instance của model có khả năng thanh toán, thường sẽ là một instance của `App\Models\User`. Sau khi bạn đã lấy được instance của model, bạn có thể sử dụng phương thức `newSubscription` để tạo gói đăng ký của model:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription(
        'default', 'price_monthly'
    )->create($request->paymentMethodId);

    // ...
});
```

Đối số đầu tiên được truyền cho phương thức `newSubscription` nên là loại nội bộ của gói đăng ký. Nếu ứng dụng của bạn chỉ cung cấp một gói đăng ký duy nhất, bạn có thể gọi nó là `default` hoặc `primary`. Loại gói đăng ký này chỉ dành cho việc sử dụng nội bộ trong ứng dụng và không có ý định được hiển thị cho người dùng. Ngoài ra, nó không nên chứa khoảng trắng và không bao giờ nên được thay đổi sau khi tạo gói đăng ký. Đối số thứ hai là giá cụ thể mà người dùng đang đăng ký. Giá trị này nên tương ứng với định danh của giá trong Stripe.

Phương thức `create`, chấp nhận [một định danh phương thức thanh toán của Stripe](#storing-payment-methods) hoặc đối tượng `PaymentMethod` của Stripe, sẽ bắt đầu gói đăng ký cũng như cập nhật cơ sở dữ liệu của bạn với ID khách hàng Stripe của model có khả năng thanh toán và các thông tin thanh toán liên quan khác.

> [!WARNING]
> Truyền trực tiếp một định danh phương thức thanh toán cho phương thức tạo gói đăng ký `create` cũng sẽ tự động thêm nó vào các phương thức thanh toán đã lưu của người dùng.

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### Collecting Recurring Payments via Invoice Emails

Thay vì tự động thu thập các khoản thanh toán định kỳ của khách hàng, bạn có thể hướng dẫn Stripe gửi email hóa đơn cho khách hàng mỗi khi khoản thanh toán định kỳ của họ đến hạn. Sau đó, khách hàng có thể thanh toán hóa đơn thủ công sau khi nhận được nó. Khách hàng không cần cung cấp phương thức thanh toán trước khi thu thập các khoản thanh toán định kỳ qua hóa đơn:

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice();
```

Thời gian khách hàng có để thanh toán hóa đơn trước khi gói đăng ký của họ bị hủy được xác định bởi tùy chọn `days_until_due`. Theo mặc định, đây là 30 ngày; tuy nhiên, bạn có thể cung cấp một giá trị cụ thể cho tùy chọn này nếu bạn muốn:

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
    'days_until_due' => 30
]);
```

<a name="subscription-quantities"></a>
#### Quantities

Nếu bạn muốn thiết lập một [số lượng](https://stripe.com/docs/billing/subscriptions/quantities) cụ thể cho giá khi tạo gói đăng ký, bạn nên gọi phương thức `quantity` trên trình tạo gói đăng ký trước khi tạo gói đăng ký:

```php
$user->newSubscription('default', 'price_monthly')
    ->quantity(5)
    ->create($paymentMethod);
```

<a name="additional-details"></a>
#### Additional Details

Nếu bạn muốn chỉ định các tùy chọn [khách hàng](https://stripe.com/docs/api/customers/create) hoặc [gói đăng ký](https://stripe.com/docs/api/subscriptions/create) bổ sung được Stripe hỗ trợ, bạn có thể làm như vậy bằng cách truyền chúng làm đối số thứ hai và thứ ba cho phương thức `create`:

```php
$user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
    'email' => $email,
], [
    'metadata' => ['note' => 'Some extra information.'],
]);
```

<a name="coupons"></a>
#### Coupons

Nếu bạn muốn áp dụng mã giảm giá khi tạo gói đăng ký, bạn có thể sử dụng phương thức `withCoupon`:

```php
$user->newSubscription('default', 'price_monthly')
    ->withCoupon('code')
    ->create($paymentMethod);
```

Hoặc, nếu bạn muốn áp dụng một [mã khuyến mãi của Stripe](https://stripe.com/docs/billing/subscriptions/discounts/codes), bạn có thể sử dụng phương thức `withPromotionCode`:

```php
$user->newSubscription('default', 'price_monthly')
    ->withPromotionCode('promo_code_id')
    ->create($paymentMethod);
```

ID mã khuyến mãi đã cung cấp nên là ID API Stripe được gán cho mã khuyến mãi và không phải mã khuyến mãi hướng tới khách hàng. Nếu bạn cần tìm ID mã khuyến mãi dựa trên một mã khuyến mãi hướng tới khách hàng đã cho, bạn có thể sử dụng phương thức `findPromotionCode`:

```php
// Find a promotion code ID by its customer facing code...
$promotionCode = $user->findPromotionCode('SUMMERSALE');

// Find an active promotion code ID by its customer facing code...
$promotionCode = $user->findActivePromotionCode('SUMMERSALE');
```

Trong ví dụ trên, đối tượng `$promotionCode` được trả về là một instance của `Laravel\Cashier\PromotionCode`. Lớp này trang trí một đối tượng `Stripe\PromotionCode` cơ bản. Bạn có thể lấy mã giảm giá liên quan đến mã khuyến mãi bằng cách gọi phương thức `coupon`:

```php
$coupon = $user->findPromotionCode('SUMMERSALE')->coupon();
```

Instance mã giảm giá cho phép bạn xác định số tiền giảm giá và liệu mã giảm giá có đại diện cho giảm giá cố định hay giảm giá dựa trên phần trăm hay không:

```php
if ($coupon->isPercentage()) {
    return $coupon->percentOff().'%'; // 21.5%
} else {
    return $coupon->amountOff(); // $5.99
}
```

Bạn cũng có thể lấy các giảm giá hiện đang được áp dụng cho khách hàng hoặc gói đăng ký:

```php
$discount = $billable->discount();

$discount = $subscription->discount();
```

Các instance `Laravel\Cashier\Discount` được trả về trang trí một instance đối tượng `Stripe\Discount` cơ bản. Bạn có thể lấy mã giảm giá liên quan đến giảm giá này bằng cách gọi phương thức `coupon`:

```php
$coupon = $subscription->discount()->coupon();
```

Nếu bạn muốn áp dụng một mã giảm giá hoặc mã khuyến mãi mới cho khách hàng hoặc gói đăng ký, bạn có thể làm như vậy thông qua các phương thức `applyCoupon` hoặc `applyPromotionCode`:

```php
$billable->applyCoupon('coupon_id');
$billable->applyPromotionCode('promotion_code_id');

$subscription->applyCoupon('coupon_id');
$subscription->applyPromotionCode('promotion_code_id');
```

Hãy nhớ rằng, bạn nên sử dụng ID API Stripe được gán cho mã khuyến mãi và không phải mã khuyến mãi hướng tới khách hàng. Chỉ một mã giảm giá hoặc mã khuyến mãi có thể được áp dụng cho khách hàng hoặc gói đăng ký tại một thời điểm nhất định.

Để biết thêm thông tin về chủ đề này, vui lòng tham khảo tài liệu Stripe về [mã giảm giá](https://stripe.com/docs/billing/subscriptions/coupons) và [mã khuyến mãi](https://stripe.com/docs/billing/subscriptions/coupons/codes).

<a name="adding-subscriptions"></a>
#### Adding Subscriptions

Nếu bạn muốn thêm một gói đăng ký cho một khách hàng đã có phương thức thanh toán mặc định, bạn có thể gọi phương thức `add` trên trình tạo gói đăng ký:

```php
use App\Models\User;

$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->add();
```

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### Creating Subscriptions From the Stripe Dashboard

Bạn cũng có thể tạo gói đăng ký từ chính dashboard Stripe. Khi làm như vậy, Cashier sẽ đồng bộ các gói đăng ký mới được thêm và gán cho chúng loại `default`. Để tùy chỉnh loại gói đăng ký được gán cho các gói đăng ký được tạo từ dashboard, [xác định các trình xử lý sự kiện webhook](#defining-webhook-event-handlers).

Ngoài ra, bạn chỉ có thể tạo một loại gói đăng ký thông qua dashboard Stripe. Nếu ứng dụng của bạn cung cấp nhiều gói đăng ký sử dụng các loại khác nhau, chỉ một loại gói đăng ký có thể được thêm thông qua dashboard Stripe.

Cuối cùng, bạn nên luôn đảm bảo chỉ thêm một gói đăng ký đang hoạt động cho mỗi loại gói đăng ký được cung cấp bởi ứng dụng của bạn. Nếu khách hàng có hai gói đăng ký `default`, chỉ gói đăng ký được thêm gần đây nhất sẽ được Cashier sử dụng mặc dù cả hai sẽ được đồng bộ với cơ sở dữ liệu của ứng dụng của bạn.

<a name="checking-subscription-status"></a>
### Checking Subscription Status

Sau khi khách hàng đăng ký vào ứng dụng của bạn, bạn có thể dễ dàng kiểm tra trạng thái gói đăng ký của họ bằng cách sử dụng nhiều phương thức tiện lợi. Đầu tiên, phương thức `subscribed` trả về `true` nếu khách hàng có gói đăng ký đang hoạt động, ngay cả khi gói đăng ký hiện đang trong giai đoạn dùng thử của nó. Phương thức `subscribed` chấp nhận loại của gói đăng ký làm đối số đầu tiên của nó:

```php
if ($user->subscribed('default')) {
    // ...
}
```

Phương thức `subscribed` cũng là một ứng cử viên tuyệt vời cho một [middleware tuyến](/docs/{{version}}/middleware), cho phép bạn lọc quyền truy cập vào các tuyến và bộ điều khiển dựa trên trạng thái gói đăng ký của người dùng:

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
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // This user is not a paying customer...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

Nếu bạn muốn xác định xem người dùng có còn trong giai đoạn dùng thử của họ hay không, bạn có thể sử dụng phương thức `onTrial`. Phương thức này có thể hữu ích để xác định xem bạn có nên hiển thị cảnh báo cho người dùng rằng họ vẫn đang trong giai đoạn dùng thử hay không:

```php
if ($user->subscription('default')->onTrial()) {
    // ...
}
```

Phương thức `subscribedToProduct` có thể được sử dụng để xác định xem người dùng có đăng ký vào một sản phẩm đã cho dựa trên định danh sản phẩm Stripe đã cho hay không. Trong Stripe, sản phẩm là tập hợp các giá. Trong ví dụ này, chúng ta sẽ xác định xem gói đăng ký `default` của người dùng có đang đăng ký tích cực vào sản phẩm "premium" của ứng dụng hay không. Định danh sản phẩm Stripe đã cho nên tương ứng với một trong các định danh sản phẩm của bạn trong dashboard Stripe:

```php
if ($user->subscribedToProduct('prod_premium', 'default')) {
    // ...
}
```

Bằng cách truyền một mảng cho phương thức `subscribedToProduct`, bạn có thể xác định xem gói đăng ký `default` của người dùng có đang đăng ký tích cực vào sản phẩm "basic" hoặc "premium" của ứng dụng hay không:

```php
if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
    // ...
}
```

Phương thức `subscribedToPrice` có thể được sử dụng để xác định xem gói đăng ký của khách hàng có tương ứng với một ID giá đã cho hay không:

```php
if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
    // ...
}
```

Phương thức `recurring` có thể được sử dụng để xác định xem người dùng hiện đang đăng ký và không còn trong giai đoạn dùng thử của họ hay không:

```php
if ($user->subscription('default')->recurring()) {
    // ...
}
```

> [!WARNING]
> Nếu người dùng có hai gói đăng ký cùng loại, gói đăng ký gần đây nhất sẽ luôn được trả về bởi phương thức `subscription`. Ví dụ, người dùng có thể có hai bản ghi gói đăng ký với loại `default`; tuy nhiên, một trong các gói đăng ký có thể là một gói đăng ký cũ, đã hết hạn, trong khi gói kia là gói đăng ký hiện tại, đang hoạt động. Gói đăng ký gần đây nhất sẽ luôn được trả về trong khi các gói đăng ký cũ hơn được giữ trong cơ sở dữ liệu để xem xét lịch sử.

<a name="cancelled-subscription-status"></a>
#### Canceled Subscription Status

Để xác định xem người dùng có từng là người đăng ký tích cực nhưng đã hủy gói đăng ký của họ hay không, bạn có thể sử dụng phương thức `canceled`:

```php
if ($user->subscription('default')->canceled()) {
    // ...
}
```

Bạn cũng có thể xác định xem người dùng đã hủy gói đăng ký của họ nhưng vẫn đang trong "giai đoạn ân hạn" cho đến khi gói đăng ký hết hạn hoàn toàn hay không. Ví dụ, nếu người dùng hủy một gói đăng ký vào ngày 5 tháng 3 được lên lịch ban đầu để hết hạn vào ngày 10 tháng 3, người dùng đang trong "giai đoạn ân hạn" của họ cho đến ngày 10 tháng 3. Lưu ý rằng phương thức `subscribed` vẫn trả về `true` trong thời gian này:

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

Để xác định xem người dùng đã hủy gói đăng ký của họ và không còn trong "giai đoạn ân hạn" hay không, bạn có thể sử dụng phương thức `ended`:

```php
if ($user->subscription('default')->ended()) {
    // ...
}
```

<a name="incomplete-and-past-due-status"></a>
#### Incomplete and Past Due Status

Nếu một gói đăng ký yêu cầu một hành động thanh toán thứ cấp sau khi tạo, gói đăng ký sẽ được đánh dấu là `incomplete`. Trạng thái gói đăng ký được lưu trữ trong cột `stripe_status` của bảng cơ sở dữ liệu `subscriptions` của Cashier.

Tương tự, nếu một hành động thanh toán thứ cấp được yêu cầu khi đổi giá, gói đăng ký sẽ được đánh dấu là `past_due`. Khi gói đăng ký của bạn ở trong một trong hai trạng thái này, nó sẽ không hoạt động cho đến khi khách hàng xác nhận thanh toán của họ. Việc xác định xem một gói đăng ký có thanh toán chưa hoàn thành hay không có thể được thực hiện bằng cách sử dụng phương thức `hasIncompletePayment` trên model có khả năng thanh toán hoặc một instance gói đăng ký:

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

Khi một gói đăng ký có thanh toán chưa hoàn thành, bạn nên hướng dẫn người dùng đến trang xác nhận thanh toán của Cashier, truyền định danh `latestPayment`. Bạn có thể sử dụng phương thức `latestPayment` có sẵn trên instance gói đăng ký để lấy định danh này:

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    Please confirm your payment.
</a>
```

Nếu bạn muốn gói đăng ký vẫn được coi là đang hoạt động khi nó ở trạng thái `past_due` hoặc `incomplete`, bạn có thể sử dụng các phương thức `keepPastDueSubscriptionsActive` và `keepIncompleteSubscriptionsActive` được cung cấp bởi Cashier. Thông thường, các phương thức này nên được gọi trong phương thức `register` của `App\Providers\AppServiceProvider` của bạn:

```php
use Laravel\Cashier\Cashier;

/**
 * Register any application services.
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
    Cashier::keepIncompleteSubscriptionsActive();
}
```

> [!WARNING]
> Khi một gói đăng ký ở trạng thái `incomplete`, nó không thể được thay đổi cho đến khi thanh toán được xác nhận. Do đó, các phương thức `swap` và `updateQuantity` sẽ ném ra một ngoại lệ khi gói đăng ký ở trạng thái `incomplete`.

<a name="subscription-scopes"></a>
#### Subscription Scopes

Hầu hết các trạng thái gói đăng ký cũng có sẵn dưới dạng phạm vi truy vấn để bạn có thể dễ dàng truy vấn cơ sở dữ liệu của mình cho các gói đăng ký ở một trạng thái đã cho:

```php
// Get all active subscriptions...
$subscriptions = Subscription::query()->active()->get();

// Get all of the canceled subscriptions for a user...
$subscriptions = $user->subscriptions()->canceled()->get();
```

Danh sách đầy đủ các phạm vi có sẵn có sẵn dưới đây:

```php
Subscription::query()->active();
Subscription::query()->canceled();
Subscription::query()->ended();
Subscription::query()->incomplete();
Subscription::query()->notCanceled();
Subscription::query()->notOnGracePeriod();
Subscription::query()->notOnTrial();
Subscription::query()->onGracePeriod();
Subscription::query()->onTrial();
Subscription::query()->pastDue();
Subscription::query()->recurring();
```

<a name="changing-prices"></a>
### Changing Prices

Sau khi khách hàng đăng ký vào ứng dụng của bạn, họ có thể thỉnh thoảng muốn chuyển sang một giá gói đăng ký mới. Để đổi khách hàng sang một giá mới, hãy truyền định danh giá Stripe cho phương thức `swap`. Khi đổi giá, người dùng được giả định là muốn kích hoạt lại gói đăng ký của họ nếu nó đã bị hủy trước đó. Định danh giá đã cho nên tương ứng với một định danh giá Stripe có sẵn trong dashboard Stripe:

```php
use App\Models\User;

$user = App\Models\User::find(1);

$user->subscription('default')->swap('price_yearly');
```

Nếu khách hàng đang dùng thử, giai đoạn dùng thử sẽ được duy trì. Ngoài ra, nếu một "số lượng" tồn tại cho gói đăng ký, số lượng đó cũng sẽ được duy trì.

Nếu bạn muốn đổi giá và hủy bất kỳ giai đoạn dùng thử nào mà khách hàng hiện đang có, bạn có thể gọi phương thức `skipTrial`:

```php
$user->subscription('default')
    ->skipTrial()
    ->swap('price_yearly');
```

Nếu bạn muốn đổi giá và xuất hóa đơn cho khách hàng ngay lập tức thay vì chờ chu kỳ thanh toán tiếp theo của họ, bạn có thể sử dụng phương thức `swapAndInvoice`:

```php
$user = User::find(1);

$user->subscription('default')->swapAndInvoice('price_yearly');
```

<a name="prorations"></a>
#### Prorations

Theo mặc định, Stripe tính toán tỷ lệ phí khi đổi giữa các giá. Phương thức `noProrate` có thể được sử dụng để cập nhật giá của gói đăng ký mà không tính toán tỷ lệ phí:

```php
$user->subscription('default')->noProrate()->swap('price_yearly');
```

Để biết thêm thông tin về tính toán tỷ lệ gói đăng ký, hãy tham khảo [tài liệu Stripe](https://stripe.com/docs/billing/subscriptions/prorations).

> [!WARNING]
> Thực hiện phương thức `noProrate` trước phương thức `swapAndInvoice` sẽ không có tác dụng nào đối với tính toán tỷ lệ. Một hóa đơn sẽ luôn được phát hành.

<a name="subscription-quantity"></a>
### Subscription Quantity

Đôi khi gói đăng ký bị ảnh hưởng bởi "số lượng". Ví dụ, một ứng dụng quản lý dự án có thể tính phí 10 đô la mỗi tháng cho mỗi dự án. Bạn có thể sử dụng các phương thức `incrementQuantity` và `decrementQuantity` để dễ dàng tăng hoặc giảm số lượng gói đăng ký của bạn:

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->incrementQuantity();

// Add five to the subscription's current quantity...
$user->subscription('default')->incrementQuantity(5);

$user->subscription('default')->decrementQuantity();

// Subtract five from the subscription's current quantity...
$user->subscription('default')->decrementQuantity(5);
```

Ngoài ra, bạn có thể thiết lập một số lượng cụ thể bằng cách sử dụng phương thức `updateQuantity`:

```php
$user->subscription('default')->updateQuantity(10);
```

Phương thức `noProrate` có thể được sử dụng để cập nhật số lượng của gói đăng ký mà không tính toán tỷ lệ phí:

```php
$user->subscription('default')->noProrate()->updateQuantity(10);
```

Để biết thêm thông tin về số lượng gói đăng ký, hãy tham khảo [tài liệu Stripe](https://stripe.com/docs/subscriptions/quantities).

<a name="quantities-for-subscription-with-multiple-products"></a>
#### Quantities for Subscriptions With Multiple Products

Nếu gói đăng ký của bạn là một [gói đăng ký với nhiều sản phẩm](#subscriptions-with-multiple-products), bạn nên truyền ID của giá có số lượng bạn muốn tăng hoặc giảm làm đối số thứ hai cho các phương thức tăng / giảm:

```php
$user->subscription('default')->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### Subscriptions With Multiple Products

[Gói đăng ký với nhiều sản phẩm](https://stripe.com/docs/billing/subscriptions/multiple-products) cho phép bạn gán nhiều sản phẩm thanh toán cho một gói đăng ký duy nhất. Ví dụ, hãy tưởng tượng bạn đang xây dựng một ứng dụng "helpdesk" dịch vụ khách hàng có giá gói đăng ký cơ bản là 10 đô la mỗi tháng nhưng cung cấp một sản phẩm bổ sung trò chuyện trực tiếp với giá thêm 15 đô la mỗi tháng. Thông tin cho các gói đăng ký với nhiều sản phẩm được lưu trữ trong bảng cơ sở dữ liệu `subscription_items` của Cashier.

Bạn có thể chỉ định nhiều sản phẩm cho một gói đăng ký đã cho bằng cách truyền một mảng giá làm đối số thứ hai cho phương thức `newSubscription`:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', [
        'price_monthly',
        'price_chat',
    ])->create($request->paymentMethodId);

    // ...
});
```

Trong ví dụ trên, khách hàng sẽ có hai giá được gắn vào gói đăng ký `default` của họ. Cả hai giá sẽ được tính phí theo các khoảng thời gian thanh toán tương ứng của chúng. Nếu cần thiết, bạn có thể sử dụng phương thức `quantity` để chỉ định một số lượng cụ thể cho mỗi giá:

```php
$user = User::find(1);

$user->newSubscription('default', ['price_monthly', 'price_chat'])
    ->quantity(5, 'price_chat')
    ->create($paymentMethod);
```

Nếu bạn muốn thêm một giá khác vào một gói đăng ký hiện có, bạn có thể gọi phương thức `addPrice` của gói đăng ký:

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat');
```

Ví dụ trên sẽ thêm giá mới và khách hàng sẽ bị tính phí cho nó vào chu kỳ thanh toán tiếp theo của họ. Nếu bạn muốn tính phí cho khách hàng ngay lập tức, bạn có thể sử dụng phương thức `addPriceAndInvoice`:

```php
$user->subscription('default')->addPriceAndInvoice('price_chat');
```

Nếu bạn muốn thêm một giá với một số lượng cụ thể, bạn có thể truyền số lượng làm đối số thứ hai của các phương thức `addPrice` hoặc `addPriceAndInvoice`:

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat', 5);
```

Bạn có thể xóa giá khỏi các gói đăng ký bằng cách sử dụng phương thức `removePrice`:

```php
$user->subscription('default')->removePrice('price_chat');
```

> [!WARNING]
> Bạn không thể xóa giá cuối cùng trên một gói đăng ký. Thay vào đó, bạn chỉ cần hủy gói đăng ký.

<a name="swapping-prices"></a>
#### Swapping Prices

Bạn cũng có thể thay đổi các giá được gắn vào một gói đăng ký với nhiều sản phẩm. Ví dụ, hãy tưởng tượng một khách hàng có gói đăng ký `price_basic` với một sản phẩm bổ sung `price_chat` và bạn muốn nâng cấp khách hàng từ `price_basic` sang giá `price_pro`:

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->swap(['price_pro', 'price_chat']);
```

Khi thực hiện ví dụ trên, mục gói đăng ký cơ bản với `price_basic` bị xóa và mục với `price_chat` được giữ lại. Ngoài ra, một mục gói đăng ký mới cho `price_pro` được tạo.

Bạn cũng có thể chỉ định các tùy chọn mục gói đăng ký bằng cách truyền một mảng các cặp khóa / giá trị cho phương thức `swap`. Ví dụ, bạn có thể cần chỉ định số lượng giá gói đăng ký:

```php
$user = User::find(1);

$user->subscription('default')->swap([
    'price_pro' => ['quantity' => 5],
    'price_chat'
]);
```

Nếu bạn muốn đổi một giá duy nhất trên một gói đăng ký, bạn có thể làm như vậy bằng cách sử dụng phương thức `swap` trên chính mục gói đăng ký. Cách tiếp cận này đặc biệt hữu ích nếu bạn muốn giữ lại tất cả metadata hiện có trên các giá khác của gói đăng ký:

```php
$user = User::find(1);

$user->subscription('default')
    ->findItemOrFail('price_basic')
    ->swap('price_pro');
```

<a name="proration"></a>
#### Proration

Theo mặc định, Stripe sẽ tính toán tỷ lệ phí khi thêm hoặc xóa giá khỏi một gói đăng ký với nhiều sản phẩm. Nếu bạn muốn thực hiện điều chỉnh giá mà không tính toán tỷ lệ, bạn nên xâu chuỗi phương thức `noProrate` vào hoạt động giá của bạn:

```php
$user->subscription('default')->noProrate()->removePrice('price_chat');
```

<a name="swapping-quantities"></a>
#### Quantities

Nếu bạn muốn cập nhật số lượng trên các giá gói đăng ký riêng lẻ, bạn có thể làm như vậy bằng cách sử dụng [các phương thức số lượng hiện có](#subscription-quantity) bằng cách truyền ID của giá làm đối số bổ sung cho phương thức:

```php
$user = User::find(1);

$user->subscription('default')->incrementQuantity(5, 'price_chat');

$user->subscription('default')->decrementQuantity(3, 'price_chat');

$user->subscription('default')->updateQuantity(10, 'price_chat');
```

> [!WARNING]
> Khi một gói đăng ký có nhiều giá, các thuộc tính `stripe_price` và `quantity` trên model `Subscription` sẽ là `null`. Để truy cập các thuộc tính giá riêng lẻ, bạn nên sử dụng mối quan hệ `items` có sẵn trên model `Subscription`.

<a name="subscription-items"></a>
#### Subscription Items

Khi một gói đăng ký có nhiều giá, nó sẽ có nhiều "mục" gói đăng ký được lưu trữ trong bảng `subscription_items` của cơ sở dữ liệu của bạn. Bạn có thể truy cập các mục này thông qua mối quan hệ `items` trên gói đăng ký:

```php
use App\Models\User;

$user = User::find(1);

$subscriptionItem = $user->subscription('default')->items->first();

// Retrieve the Stripe price and quantity for a specific item...
$stripePrice = $subscriptionItem->stripe_price;
$quantity = $subscriptionItem->quantity;
```

Bạn cũng có thể lấy một giá cụ thể bằng cách sử dụng phương thức `findItemOrFail`:

```php
$user = User::find(1);

$subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');
```

<a name="multiple-subscriptions"></a>
### Multiple Subscriptions

Stripe cho phép khách hàng của bạn có nhiều gói đăng ký đồng thời. Ví dụ, bạn có thể vận hành một phòng tập thể dục cung cấp gói đăng ký bơi lội và gói đăng ký cử tạ, và mỗi gói đăng ký có thể có giá cả khác nhau. Tất nhiên, khách hàng nên có thể đăng ký vào một hoặc cả hai gói.

Khi ứng dụng của bạn tạo gói đăng ký, bạn có thể cung cấp loại của gói đăng ký cho phương thức `newSubscription`. Loại có thể là bất kỳ chuỗi nào đại diện cho loại gói đăng ký mà người dùng đang bắt đầu:

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $request->user()->newSubscription('swimming')
        ->price('price_swimming_monthly')
        ->create($request->paymentMethodId);

    // ...
});
```

Trong ví dụ này, chúng ta đã bắt đầu một gói đăng ký bơi lội hàng tháng cho khách hàng. Tuy nhiên, họ có thể muốn đổi sang gói đăng ký hàng năm vào một thời điểm sau. Khi điều chỉnh gói đăng ký của khách hàng, chúng ta có thể chỉ cần đổi giá trên gói đăng ký `swimming`:

```php
$user->subscription('swimming')->swap('price_swimming_yearly');
```

Tất nhiên, bạn cũng có thể hủy hoàn toàn gói đăng ký:

```php
$user->subscription('swimming')->cancel();
```

<a name="usage-based-billing"></a>
### Usage Based Billing

[Thanh toán dựa trên mức sử dụng](https://stripe.com/docs/billing/subscriptions/metered-billing) cho phép bạn tính phí cho khách hàng dựa trên mức sử dụng sản phẩm của họ trong một chu kỳ thanh toán. Ví dụ, bạn có thể tính phí cho khách hàng dựa trên số lượng tin nhắn văn bản hoặc email họ gửi mỗi tháng.

Để bắt đầu sử dụng thanh toán dựa trên mức sử dụng, trước tiên bạn sẽ cần tạo một sản phẩm mới trong dashboard Stripe của bạn với một [mô hình thanh toán dựa trên mức sử dụng](https://docs.stripe.com/billing/subscriptions/usage-based/implementation-guide) và một [đồng hồ đo](https://docs.stripe.com/billing/subscriptions/usage-based/recording-usage#configure-meter). Sau khi tạo đồng hồ đo, hãy lưu trữ tên sự kiện liên quan và ID đồng hồ đo, mà bạn sẽ cần để báo cáo và truy xuất mức sử dụng. Sau đó, sử dụng phương thức `meteredPrice` để thêm ID giá đo lường vào gói đăng ký của khách hàng:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default')
        ->meteredPrice('price_metered')
        ->create($request->paymentMethodId);

    // ...
});
```

Bạn cũng có thể bắt đầu một gói đăng ký đo lường thông qua [Stripe Checkout](#checkout):

```php
$checkout = Auth::user()
    ->newSubscription('default', [])
    ->meteredPrice('price_metered')
    ->checkout();

return view('your-checkout-view', [
    'checkout' => $checkout,
]);
```

<a name="reporting-usage"></a>
#### Reporting Usage

Khi khách hàng của bạn sử dụng ứng dụng của bạn, bạn sẽ báo cáo mức sử dụng của họ cho Stripe để họ có thể được tính phí chính xác. Để báo cáo mức sử dụng của một sự kiện đo lường, bạn có thể sử dụng phương thức `reportMeterEvent` trên model `Billable` của bạn:

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent');
```

Theo mặc định, một "số lượng sử dụng" là 1 được thêm vào chu kỳ thanh toán. Ngoài ra, bạn có thể truyền một số lượng "sử dụng" cụ thể để thêm vào mức sử dụng của khách hàng cho chu kỳ thanh toán:

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent', quantity: 15);
```

Để truy xuất tóm tắt sự kiện của khách hàng cho một đồng hồ đo, bạn có thể sử dụng phương thức `meterEventSummaries` của instance `Billable`:

```php
$user = User::find(1);

$meterUsage = $user->meterEventSummaries($meterId);

$meterUsage->first()->aggregated_value // 10
```

Vui lòng tham khảo [tài liệu đối tượng tóm tắt sự kiện đồng hồ đo của Stripe](https://docs.stripe.com/api/billing/meter-event_summary/object) để biết thêm thông tin về tóm tắt sự kiện đồng hồ đo.

Để [liệt kê tất cả đồng hồ đo](https://docs.stripe.com/api/billing/meter/list), bạn có thể sử dụng phương thức `meters` của instance `Billable`:

```php
$user = User::find(1);

$user->meters();
```

<a name="subscription-taxes"></a>
### Subscription Taxes

> [!WARNING]
> Thay vì tính toán thuế suất thủ công, bạn có thể [tự động tính toán thuế bằng cách sử dụng Stripe Tax](#tax-configuration)

Để chỉ định các thuế suất mà người dùng trả cho một gói đăng ký, bạn nên triển khai phương thức `taxRates` trên model có khả năng thanh toán của bạn và trả về một mảng chứa các ID thuế suất Stripe. Bạn có thể xác định các thuế suất này trong [dashboard Stripe của bạn](https://dashboard.stripe.com/test/tax-rates):

```php
/**
 * The tax rates that should apply to the customer's subscriptions.
 *
 * @return array<int, string>
 */
public function taxRates(): array
{
    return ['txr_id'];
}
```

Phương thức `taxRates` cho phép bạn áp dụng một thuế suất trên cơ sở từng khách hàng, điều này có thể hữu ích cho một cơ sở người dùng trải dài nhiều quốc gia và thuế suất.
Nếu bạn đang cung cấp các gói đăng ký với nhiều sản phẩm, bạn có thể định nghĩa các mức thuế khác nhau cho mỗi giá bằng cách triển khai phương thức `priceTaxRates` trên mô hình có thể tính phí của mình:

```php
/**
 * Các mức thuế nên áp dụng cho gói đăng ký của khách hàng.
 *
 * @return array<string, array<int, string>>
 */
public function priceTaxRates(): array
{
    return [
        'price_monthly' => ['txr_id'],
    ];
}
```

> [!WARNING]
> Phương thức `taxRates` chỉ áp dụng cho các khoản phí đăng ký. Nếu bạn sử dụng Cashier để thực hiện các khoản phí "một lần", bạn sẽ cần chỉ định mức thuế thủ công tại thời điểm đó.

<a name="syncing-tax-rates"></a>
#### Đồng bộ hóa Mức Thuế

Khi thay đổi các ID mức thuế được mã hóa cứng được trả về bởi phương thức `taxRates`, cài đặt thuế trên bất kỳ gói đăng ký hiện có nào của người dùng sẽ vẫn giữ nguyên. Nếu bạn muốn cập nhật giá trị thuế cho các gói đăng ký hiện có với các giá trị `taxRates` mới, bạn nên gọi phương thức `syncTaxRates` trên phiên bản gói đăng ký của người dùng:

```php
$user->subscription('default')->syncTaxRates();
```

Điều này cũng sẽ đồng bộ hóa bất kỳ mức thuế mục nào cho gói đăng ký có nhiều sản phẩm. Nếu ứng dụng của bạn đang cung cấp các gói đăng ký với nhiều sản phẩm, bạn nên đảm bảo rằng mô hình có thể tính phí của bạn triển khai phương thức `priceTaxRates` [được thảo luận ở trên](#subscription-taxes).

<a name="tax-exemption"></a>
#### Miễn Thuế

Cashier cũng cung cấp các phương thức `isNotTaxExempt`, `isTaxExempt`, và `reverseChargeApplies` để xác định xem khách hàng có được miễn thuế hay không. Các phương thức này sẽ gọi API Stripe để xác định trạng thái miễn thuế của khách hàng:

```php
use App\Models\User;

$user = User::find(1);

$user->isTaxExempt();
$user->isNotTaxExempt();
$user->reverseChargeApplies();
```

> [!WARNING]
> Các phương thức này cũng có sẵn trên bất kỳ đối tượng `Laravel\Cashier\Invoice` nào. Tuy nhiên, khi được gọi trên đối tượng `Invoice`, các phương thức sẽ xác định trạng thái miễn thuế tại thời điểm hóa đơn được tạo.

<a name="subscription-anchor-date"></a>
### Ngày Neo Gói Đăng Ký

Theo mặc định, ngày neo chu kỳ thanh toán là ngày gói đăng ký được tạo hoặc, nếu sử dụng thời gian dùng thử, là ngày kết thúc thời gian dùng thử. Nếu bạn muốn sửa đổi ngày neo thanh toán, bạn có thể sử dụng phương thức `anchorBillingCycleOn`:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $anchor = Carbon::parse('first day of next month');

    $request->user()->newSubscription('default', 'price_monthly')
        ->anchorBillingCycleOn($anchor->startOfDay())
        ->create($request->paymentMethodId);

    // ...
});
```

Để biết thêm thông tin về quản lý chu kỳ thanh toán gói đăng ký, hãy tham khảo [tài liệu chu kỳ thanh toán của Stripe](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### Hủy Gói Đăng Ký

Để hủy một gói đăng ký, hãy gọi phương thức `cancel` trên gói đăng ký của người dùng:

```php
$user->subscription('default')->cancel();
```

Khi một gói đăng ký bị hủy, Cashier sẽ tự động đặt cột `ends_at` trong bảng cơ sở dữ liệu `subscriptions` của bạn. Cột này được sử dụng để biết khi nào phương thức `subscribed` nên bắt đầu trả về `false`.

Ví dụ, nếu khách hàng hủy gói đăng ký vào ngày 1 tháng 3, nhưng gói đăng ký không được lên lịch kết thúc cho đến ngày 5 tháng 3, phương thức `subscribed` sẽ tiếp tục trả về `true` cho đến ngày 5 tháng 3. Điều này được thực hiện vì người dùng thường được phép tiếp tục sử dụng ứng dụng cho đến hết chu kỳ thanh toán của họ.

Bạn có thể xác định xem người dùng đã hủy gói đăng ký của họ nhưng vẫn còn trong "thời gian ân hạn" hay không bằng phương thức `onGracePeriod`:

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

Nếu bạn muốn hủy gói đăng ký ngay lập tức, hãy gọi phương thức `cancelNow` trên gói đăng ký của người dùng:

```php
$user->subscription('default')->cancelNow();
```

Nếu bạn muốn hủy gói đăng ký ngay lập tức và xuất hóa đơn cho bất kỳ việc sử dụng theo đồng hồ chưa xuất hóa đơn còn lại hoặc các mục hóa đơn điều chỉnh mới / đang chờ xử lý, hãy gọi phương thức `cancelNowAndInvoice` trên gói đăng ký của người dùng:

```php
$user->subscription('default')->cancelNowAndInvoice();
```

Bạn cũng có thể chọn hủy gói đăng ký tại một thời điểm cụ thể:

```php
$user->subscription('default')->cancelAt(
    now()->plus(days: 10)
);
```

Cuối cùng, bạn nên luôn hủy gói đăng ký của người dùng trước khi xóa mô hình người dùng liên quan:

```php
$user->subscription('default')->cancelNow();

$user->delete();
```

<a name="resuming-subscriptions"></a>
### Tiếp Tục Gói Đăng Ký

Nếu khách hàng đã hủy gói đăng ký của họ và bạn muốn tiếp tục lại, bạn có thể gọi phương thức `resume` trên gói đăng ký. Khách hàng vẫn phải nằm trong "thời gian ân hạn" của họ để có thể tiếp tục gói đăng ký:

```php
$user->subscription('default')->resume();
```

Nếu khách hàng hủy gói đăng ký và sau đó tiếp tục lại gói đăng ký đó trước khi gói đăng ký hết hạn hoàn toàn, khách hàng sẽ không bị tính phí ngay lập tức. Thay vào đó, gói đăng ký của họ sẽ được kích hoạt lại và họ sẽ được tính phí theo chu kỳ thanh toán gốc.

<a name="subscription-trials"></a>
## Thời Gian Dùng Thử Gói Đăng Ký

<a name="with-payment-method-up-front"></a>
### Với Phương Thức Thanh Toán Trước

Nếu bạn muốn cung cấp thời gian dùng thử cho khách hàng của mình trong khi vẫn thu thập thông tin phương thức thanh toán trước, bạn nên sử dụng phương thức `trialDays` khi tạo gói đăng ký của mình:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', 'price_monthly')
        ->trialDays(10)
        ->create($request->paymentMethodId);

    // ...
});
```

Phương thức này sẽ đặt ngày kết thúc thời gian dùng thử trên bản ghi gói đăng ký trong cơ sở dữ liệu và hướng dẫn Stripe không bắt đầu tính phí cho khách hàng cho đến sau ngày này. Khi sử dụng phương thức `trialDays`, Cashier sẽ ghi đè bất kỳ thời gian dùng thử mặc định nào được cấu hình cho giá trong Stripe.

> [!WARNING]
> Nếu gói đăng ký của khách hàng không bị hủy trước ngày kết thúc thời gian dùng thử, họ sẽ bị tính phí ngay khi thời gian dùng thử hết hạn, vì vậy bạn nên đảm bảo thông báo cho người dùng của bạn về ngày kết thúc thời gian dùng thử của họ.

Phương thức `trialUntil` cho phép bạn cung cấp một phiên bản `DateTime` chỉ định khi nào thời gian dùng thử nên kết thúc:

```php
use Illuminate\Support\Carbon;

$user->newSubscription('default', 'price_monthly')
    ->trialUntil(Carbon::now()->plus(days: 10))
    ->create($paymentMethod);
```

Bạn có thể xác định xem người dùng có đang trong thời gian dùng thử của họ hay không bằng cách sử dụng phương thức `onTrial` của phiên bản người dùng hoặc phương thức `onTrial` của phiên bản gói đăng ký. Hai ví dụ dưới đây là tương đương:

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->subscription('default')->onTrial()) {
    // ...
}
```

Bạn có thể sử dụng phương thức `endTrial` để kết thúc ngay lập tức thời gian dùng thử của gói đăng ký:

```php
$user->subscription('default')->endTrial();
```

Để xác định xem một thời gian dùng thử hiện có đã hết hạn hay chưa, bạn có thể sử dụng các phương thức `hasExpiredTrial`:

```php
if ($user->hasExpiredTrial('default')) {
    // ...
}

if ($user->subscription('default')->hasExpiredTrial()) {
    // ...
}
```

<a name="defining-trial-days-in-stripe-cashier"></a>
#### Định Nghĩa Số Ngày Dùng Thử trong Stripe / Cashier

Bạn có thể chọn định nghĩa số ngày dùng thử mà giá của bạn nhận được trong bảng điều khiển Stripe hoặc luôn truyền chúng một cách rõ ràng bằng cách sử dụng Cashier. Nếu bạn chọn định nghĩa số ngày dùng thử của giá trong Stripe, bạn nên lưu ý rằng các gói đăng ký mới, bao gồm cả các gói đăng ký mới cho khách hàng đã từng có gói đăng ký trong quá khứ, sẽ luôn nhận được thời gian dùng thử trừ khi bạn gọi rõ ràng phương thức `skipTrial()`.

<a name="without-payment-method-up-front"></a>
### Không Có Phương Thức Thanh Toán Trước

Nếu bạn muốn cung cấp thời gian dùng thử mà không thu thập thông tin phương thức thanh toán của người dùng trước, bạn có thể đặt cột `trial_ends_at` trên bản ghi người dùng thành ngày kết thúc thời gian dùng thử mong muốn của bạn. Điều này thường được thực hiện trong quá trình đăng ký người dùng:

```php
use App\Models\User;

$user = User::create([
    // ...
    'trial_ends_at' => now()->plus(days: 10),
]);
```

> [!WARNING]
> Hãy đảm bảo thêm một [chuyển đổi ngày](/docs/{{version}}/eloquent-mutators#date-casting) cho thuộc tính `trial_ends_at` trong định nghĩa lớp của mô hình có thể tính phí của bạn.

Cashier gọi loại thời gian dùng thử này là "thời gian dùng thử chung", vì nó không được gắn với bất kỳ gói đăng ký hiện có nào. Phương thức `onTrial` trên phiên bản mô hình có thể tính phí sẽ trả về `true` nếu ngày hiện tại không vượt quá giá trị của `trial_ends_at`:

```php
if ($user->onTrial()) {
    // Người dùng đang trong thời gian dùng thử của họ...
}
```

Khi bạn đã sẵn sàng tạo một gói đăng ký thực tế cho người dùng, bạn có thể sử dụng phương thức `newSubscription` như bình thường:

```php
$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->create($paymentMethod);
```

Để lấy ngày kết thúc thời gian dùng thử của người dùng, bạn có thể sử dụng phương thức `trialEndsAt`. Phương thức này sẽ trả về một phiên bản ngày Carbon nếu người dùng đang trong thời gian dùng thử hoặc `null` nếu họ không. Bạn cũng có thể truyền một tham số loại gói đăng ký tùy chọn nếu bạn muốn lấy ngày kết thúc thời gian dùng thử cho một gói đăng ký cụ thể khác với gói mặc định:

```php
if ($user->onTrial()) {
    $trialEndsAt = $user->trialEndsAt('main');
}
```

Bạn cũng có thể sử dụng phương thức `onGenericTrial` nếu bạn muốn biết cụ thể rằng người dùng đang trong thời gian dùng thử "chung" của họ và chưa tạo một gói đăng ký thực tế:

```php
if ($user->onGenericTrial()) {
    // Người dùng đang trong thời gian dùng thử "chung" của họ...
}
```

<a name="extending-trials"></a>
### Mở Rộng Thời Gian Dùng Thử

Phương thức `extendTrial` cho phép bạn mở rộng thời gian dùng thử của gói đăng ký sau khi gói đăng ký đã được tạo. Nếu thời gian dùng thử đã hết hạn và khách hàng đã được tính phí cho gói đăng ký, bạn vẫn có thể cung cấp cho họ thời gian dùng thử mở rộng. Thời gian nằm trong thời gian dùng thử sẽ được trừ từ hóa đơn tiếp theo của khách hàng:

```php
use App\Models\User;

$subscription = User::find(1)->subscription('default');

// Kết thúc thời gian dùng thử 7 ngày kể từ bây giờ...
$subscription->extendTrial(
    now()->plus(days: 7)
);

// Thêm thêm 5 ngày vào thời gian dùng thử...
$subscription->extendTrial(
    $subscription->trial_ends_at->plus(days: 5)
);
```

<a name="handling-stripe-webhooks"></a>
## Xử Lý Webhook Stripe

> [!NOTE]
> Bạn có thể sử dụng [Stripe CLI](https://stripe.com/docs/stripe-cli) để giúp kiểm tra webhooks trong quá trình phát triển cục bộ.

Stripe có thể thông báo cho ứng dụng của bạn về nhiều sự kiện khác nhau thông qua webhooks. Theo mặc định, một route trỏ đến bộ điều khiển webhook của Cashier được tự động đăng ký bởi nhà cung cấp dịch vụ Cashier. Bộ điều khiển này sẽ xử lý tất cả các yêu cầu webhook đến.

Theo mặc định, bộ điều khiển webhook của Cashier sẽ tự động xử lý việc hủy các gói đăng ký có quá nhiều khoản phí thất bại (như được định nghĩa bởi cài đặt Stripe của bạn), cập nhật khách hàng, xóa khách hàng, cập nhật gói đăng ký và thay đổi phương thức thanh toán; tuy nhiên, như chúng ta sẽ sớm khám phá, bạn có thể mở rộng bộ điều khiển này để xử lý bất kỳ sự kiện webhook Stripe nào bạn thích.

Để đảm bảo ứng dụng của bạn có thể xử lý webhooks Stripe, hãy đảm bảo cấu hình URL webhook trong bảng điều khiển Stripe. Theo mặc định, bộ điều khiển webhook của Cashier phản hồi với đường dẫn URL `/stripe/webhook`. Danh sách đầy đủ tất cả các webhooks bạn nên kích hoạt trong bảng điều khiển Stripe là:

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

Để thuận tiện, Cashier bao gồm lệnh Artisan `cashier:webhook`. Lệnh này sẽ tạo một webhook trong Stripe lắng nghe tất cả các sự kiện được yêu cầu bởi Cashier:

```shell
php artisan cashier:webhook
```

Theo mặc định, webhook được tạo sẽ trỏ đến URL được định nghĩa bởi biến môi trường `APP_URL` và route `cashier.webhook` được bao gồm với Cashier. Bạn có thể cung cấp tùy chọn `--url` khi gọi lệnh nếu bạn muốn sử dụng một URL khác:

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

Webhook được tạo sẽ sử dụng phiên bản API Stripe mà phiên bản Cashier của bạn tương thích. Nếu bạn muốn sử dụng phiên bản Stripe khác, bạn có thể cung cấp tùy chọn `--api-version`:

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

Sau khi tạo, webhook sẽ được kích hoạt ngay lập tức. Nếu bạn muốn tạo webhook nhưng để nó bị vô hiệu hóa cho đến khi bạn sẵn sàng, bạn có thể cung cấp tùy chọn `--disabled` khi gọi lệnh:

```shell
php artisan cashier:webhook --disabled
```

> [!WARNING]
> Hãy đảm bảo bạn bảo vệ các yêu cầu webhook Stripe đến bằng middleware [xác minh chữ ký webhook](#verifying-webhook-signatures) được bao gồm trong Cashier.

<a name="webhooks-csrf-protection"></a>
#### Webhooks và Bảo Vệ CSRF

Vì webhooks Stripe cần bỏ qua [bảo vệ CSRF](/docs/{{version}}/csrf) của Laravel, bạn nên đảm bảo rằng Laravel không cố gắng xác thực mã CSRF cho các webhooks Stripe đến. Để thực hiện điều này, bạn nên loại trừ `stripe/*` khỏi bảo vệ CSRF trong tệp `bootstrap/app.php` của ứng dụng của bạn:

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(except: [
        'stripe/*',
    ]);
})
```

<a name="defining-webhook-event-handlers"></a>
### Định Nghĩa Bộ Xử Lý Sự Kiện Webhook

Cashier tự động xử lý việc hủy gói đăng ký cho các khoản phí thất bại và các sự kiện webhook Stripe phổ biến khác. Tuy nhiên, nếu bạn có các sự kiện webhook bổ sung mà bạn muốn xử lý, bạn có thể thực hiện điều này bằng cách lắng nghe các sự kiện sau được gửi đi bởi Cashier:

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

Cả hai sự kiện đều chứa tải đầy đủ của webhook Stripe. Ví dụ, nếu bạn muốn xử lý webhook `invoice.payment_succeeded`, bạn có thể đăng ký một [người lắng nghe](/docs/{{version}}/events#defining-listeners) sẽ xử lý sự kiện:

```php
<?php

namespace App\Listeners;

use Laravel\Cashier\Events\WebhookReceived;

class StripeEventListener
{
    /**
     * Xử lý các webhook Stripe nhận được.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['type'] === 'invoice.payment_succeeded') {
            // Xử lý sự kiện đến...
        }
    }
}
```

<a name="verifying-webhook-signatures"></a>
### Xác Minh Chữ Ký Webhook

Để bảo mật webhooks của bạn, bạn có thể sử dụng [chữ ký webhook của Stripe](https://stripe.com/docs/webhooks/signatures). Để thuận tiện, Cashier tự động bao gồm một middleware xác nhận rằng yêu cầu webhook Stripe đến là hợp lệ.

Để kích hoạt xác minh webhook, hãy đảm bảo rằng biến môi trường `STRIPE_WEBHOOK_SECRET` được đặt trong tệp `.env` của ứng dụng của bạn. `secret` của webhook có thể được lấy từ bảng điều khiển tài khoản Stripe của bạn.

<a name="single-charges"></a>
## Khoản Phí Đơn Lẻ

<a name="simple-charge"></a>
### Khoản Phí Đơn Giản

Nếu bạn muốn thực hiện khoản phí một lần đối với khách hàng, bạn có thể sử dụng phương thức `charge` trên phiên bản mô hình có thể tính phí. Bạn sẽ cần [cung cấp định danh phương thức thanh toán](#payment-methods-for-single-charges) làm đối số thứ hai cho phương thức `charge`:

```php
use Illuminate\Http\Request;

Route::post('/purchase', function (Request $request) {
    $stripeCharge = $request->user()->charge(
        100, $request->paymentMethodId
    );

    // ...
});
```

Phương thức `charge` chấp nhận một mảng làm đối số thứ ba, cho phép bạn chuyển bất kỳ tùy chọn nào bạn muốn cho việc tạo khoản phí Stripe bên dưới. Thông tin thêm về các tùy chọn có sẵn cho bạn khi tạo khoản phí có thể được tìm thấy trong [tài liệu Stripe](https://stripe.com/docs/api/charges/create):

```php
$user->charge(100, $paymentMethod, [
    'custom_option' => $value,
]);
```

Bạn cũng có thể sử dụng phương thức `charge` mà không có khách hàng hoặc người dùng bên dưới. Để thực hiện điều này, hãy gọi phương thức `charge` trên một phiên bản mới của mô hình có thể tính phí của ứng dụng của bạn:

```php
use App\Models\User;

$stripeCharge = (new User)->charge(100, $paymentMethod);
```

Phương thức `charge` sẽ ném một ngoại lệ nếu khoản phí thất bại. Nếu khoản phí thành công, một phiên bản của `Laravel\Cashier\Payment` sẽ được trả về từ phương thức:

```php
try {
    $payment = $user->charge(100, $paymentMethod);
} catch (Exception $e) {
    // ...
}
```

> [!WARNING]
> Phương thức `charge` chấp nhận số tiền thanh toán trong đơn vị thấp nhất của đồng tiền được sử dụng bởi ứng dụng của bạn. Ví dụ, nếu khách hàng đang thanh toán bằng Đô la Mỹ, số tiền nên được chỉ định bằng xu.

<a name="charge-with-invoice"></a>
### Khoản Phí Với Hóa Đơn

Đôi khi bạn có thể cần thực hiện khoản phí một lần và cung cấp hóa đơn PDF cho khách hàng của mình. Phương thức `invoicePrice` cho phép bạn làm điều đó. Ví dụ, hãy xuất hóa đơn cho khách hàng cho năm chiếc áo mới:

```php
$user->invoicePrice('price_tshirt', 5);
```

Hóa đơn sẽ được tính phí ngay lập tức đối với phương thức thanh toán mặc định của người dùng. Phương thức `invoicePrice` cũng chấp nhận một mảng làm đối số thứ ba. Mảng này chứa các tùy chọn thanh toán cho mục hóa đơn. Đối số thứ tư được chấp nhận bởi phương thức cũng là một mảng nên chứa các tùy chọn thanh toán cho chính hóa đơn:

```php
$user->invoicePrice('price_tshirt', 5, [
    'discounts' => [
        ['coupon' => 'SUMMER21SALE']
    ],
], [
    'default_tax_rates' => ['txr_id'],
]);
```

Tương tự như `invoicePrice`, bạn có thể sử dụng phương thức `tabPrice` để tạo khoản phí một lần cho nhiều mục (lên đến 250 mục mỗi hóa đơn) bằng cách thêm chúng vào "tab" của khách hàng và sau đó xuất hóa đơn cho khách hàng. Ví dụ, chúng ta có thể xuất hóa đơn cho khách hàng cho năm chiếc áo và hai cái cốc:

```php
$user->tabPrice('price_tshirt', 5);
$user->tabPrice('price_mug', 2);
$user->invoice();
```

Ngoài ra, bạn có thể sử dụng phương thức `invoiceFor` để thực hiện khoản phí "một lần" đối với phương thức thanh toán mặc định của khách hàng:

```php
$user->invoiceFor('One Time Fee', 500);
```

Mặc dù phương thức `invoiceFor` có sẵn để bạn sử dụng, nhưng được khuyến nghị rằng bạn sử dụng các phương thức `invoicePrice` và `tabPrice` với các giá được định nghĩa trước. Bằng cách làm như vậy, bạn sẽ có quyền truy cập vào phân tích và dữ liệu tốt hơn trong bảng điều khiển Stripe của bạn về doanh số của bạn trên cơ sở từng sản phẩm.

> [!WARNING]
> Các phương thức `invoice`, `invoicePrice`, và `invoiceFor` sẽ tạo một hóa đơn Stripe sẽ thử lại các nỗ lực thanh toán thất bại. Nếu bạn không muốn hóa đơn thử lại các khoản phí thất bại, bạn sẽ cần đóng chúng bằng cách sử dụng API Stripe sau khoản phí thất bại đầu tiên.

<a name="creating-payment-intents"></a>
### Tạo Payment Intents

Bạn có thể tạo một payment intent Stripe mới bằng cách gọi phương thức `pay` trên phiên bản mô hình có thể tính phí. Gọi phương thức này sẽ tạo một payment intent được bọc trong một phiên bản `Laravel\Cashier\Payment`:

```php
use Illuminate\Http\Request;

Route::post('/pay', function (Request $request) {
    $payment = $request->user()->pay(
        $request->get('amount')
    );

    return $payment->client_secret;
});
```

Sau khi tạo payment intent, bạn có thể trả về client secret cho frontend của ứng dụng của mình để người dùng có thể hoàn tất thanh toán trong trình duyệt của họ. Để đọc thêm về việc xây dựng các luồng thanh toán hoàn chỉnh bằng cách sử dụng payment intents Stripe, hãy tham khảo [tài liệu Stripe](https://stripe.com/docs/payments/accept-a-payment?platform=web).

Khi sử dụng phương thức `pay`, các phương thức thanh toán mặc định được kích hoạt trong bảng điều khiển Stripe của bạn sẽ có sẵn cho khách hàng. Ngoài ra, nếu bạn chỉ muốn cho phép sử dụng một số phương thức thanh toán cụ thể, bạn có thể sử dụng phương thức `payWith`:

```php
use Illuminate\Http\Request;

Route::post('/pay', function (Request $request) {
    $payment = $request->user()->payWith(
        $request->get('amount'), ['card', 'bancontact']
    );

    return $payment->client_secret;
});
```

> [!WARNING]
> Các phương thức `pay` và `payWith` chấp nhận số tiền thanh toán trong đơn vị thấp nhất của đồng tiền được sử dụng bởi ứng dụng của bạn. Ví dụ, nếu khách hàng đang thanh toán bằng Đô la Mỹ, số tiền nên được chỉ định bằng xu.

<a name="refunding-charges"></a>
### Hoàn Tiền Khoản Phí

Nếu bạn cần hoàn tiền cho một khoản phí Stripe, bạn có thể sử dụng phương thức `refund`. Phương thức này chấp nhận [ID payment intent](#payment-methods-for-single-charges) của Stripe làm đối số đầu tiên:

```php
$payment = $user->charge(100, $paymentMethodId);

$user->refund($payment->id);
```

<a name="invoices"></a>
## Hóa Đơn

<a name="retrieving-invoices"></a>
### Lấy Hóa Đơn

Bạn có thể dễ dàng lấy một mảng các hóa đơn của mô hình có thể tính phí bằng phương thức `invoices`. Phương thức `invoices` trả về một tập hợp các phiên bản `Laravel\Cashier\Invoice`:

```php
$invoices = $user->invoices();
```

Nếu bạn muốn bao gồm các hóa đơn đang chờ xử lý trong kết quả, bạn có thể sử dụng phương thức `invoicesIncludingPending`:

```php
$invoices = $user->invoicesIncludingPending();
```

Bạn có thể sử dụng phương thức `findInvoice` để lấy một hóa đơn cụ thể theo ID của nó:

```php
$invoice = $user->findInvoice($invoiceId);
```

<a name="displaying-invoice-information"></a>
#### Hiển Thị Thông Tin Hóa Đơn

Khi liệt kê các hóa đơn cho khách hàng, bạn có thể sử dụng các phương thức của hóa đơn để hiển thị thông tin hóa đơn liên quan. Ví dụ, bạn có thể muốn liệt kê mọi hóa đơn trong một bảng, cho phép người dùng dễ dàng tải xuống bất kỳ hóa đơn nào trong số đó:

```blade
<table>
    @foreach ($invoices as $invoice)
        <tr>
            <td>{{ $invoice->date()->toFormattedDateString() }}</td>
            <td>{{ $invoice->total() }}</td>
            <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
        </tr>
    @endforeach
</table>
```

<a name="upcoming-invoices"></a>
### Hóa Đơn Sắp Tới

Để lấy hóa đơn sắp tới cho khách hàng, bạn có thể sử dụng phương thức `upcomingInvoice`:

```php
$invoice = $user->upcomingInvoice();
```

Tương tự, nếu khách hàng có nhiều gói đăng ký, bạn cũng có thể lấy hóa đơn sắp tới cho một gói đăng ký cụ thể:

```php
$invoice = $user->subscription('default')->upcomingInvoice();
```

<a name="previewing-subscription-invoices"></a>
### Xem Trước Hóa Đơn Gói Đăng Ký

Sử dụng phương thức `previewInvoice`, bạn có thể xem trước một hóa đơn trước khi thực hiện thay đổi giá. Điều này sẽ cho phép bạn xác định hóa đơn của khách hàng sẽ trông như thế nào khi một thay đổi giá nhất định được thực hiện:

```php
$invoice = $user->subscription('default')->previewInvoice('price_yearly');
```

Bạn có thể chuyển một mảng giá cho phương thức `previewInvoice` để xem trước các hóa đơn với nhiều giá mới:

```php
$invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);
```

<a name="generating-invoice-pdfs"></a>
### Tạo PDF Hóa Đơn

Trước khi tạo PDF hóa đơn, bạn nên sử dụng Composer để cài đặt thư viện Dompdf, là trình kết xuất hóa đơn mặc định cho Cashier:

```shell
composer require dompdf/dompdf
```

Từ trong một route hoặc bộ điều khiển, bạn có thể sử dụng phương thức `downloadInvoice` để tạo tải xuống PDF của một hóa đơn nhất định. Phương thức này sẽ tự động tạo phản hồi HTTP thích hợp cần thiết để tải xuống hóa đơn:

```php
use Illuminate\Http\Request;

Route::get('/user/invoice/{invoice}', function (Request $request, string $invoiceId) {
    return $request->user()->downloadInvoice($invoiceId);
});
```

Theo mặc định, tất cả dữ liệu trên hóa đơn được lấy từ dữ liệu khách hàng và hóa đơn được lưu trữ trong Stripe. Tên tệp dựa trên giá trị cấu hình `app.name` của bạn. Tuy nhiên, bạn có thể tùy chỉnh một số dữ liệu này bằng cách cung cấp một mảng làm đối số thứ hai cho phương thức `downloadInvoice`. Mảng này cho phép bạn tùy chỉnh thông tin như chi tiết công ty và sản phẩm của bạn:

```php
return $request->user()->downloadInvoice($invoiceId, [
    'vendor' => 'Your Company',
    'product' => 'Your Product',
    'street' => 'Main Str. 1',
    'location' => '2000 Antwerp, Belgium',
    'phone' => '+32 499 00 00 00',
    'email' => 'info@example.com',
    'url' => 'https://example.com',
    'vendorVat' => 'BE123456789',
]);
```

Phương thức `downloadInvoice` cũng cho phép tên tệp tùy chỉnh thông qua đối số thứ ba của nó. Tên tệp này sẽ tự động được thêm hậu tố `.pdf`:

```php
return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');
```

<a name="custom-invoice-render"></a>
#### Trình Kết Xuất Hóa Đơn Tùy Chỉnh

Cashier cũng cho phép sử dụng trình kết xuất hóa đơn tùy chỉnh. Theo mặc định, Cashier sử dụng triển khai `DompdfInvoiceRenderer`, sử dụng thư viện PHP [dompdf](https://github.com/dompdf/dompdf) để tạo các hóa đơn của Cashier. Tuy nhiên, bạn có thể sử dụng bất kỳ trình kết xuất nào bạn muốn bằng cách triển khai giao diện `Laravel\Cashier\Contracts\InvoiceRenderer`. Ví dụ, bạn có thể muốn kết xuất một PDF hóa đơn bằng cách sử dụng cuộc gọi API đến dịch vụ kết xuất PDF bên thứ ba:

```php
use Illuminate\Support\Facades\Http;
use Laravel\Cashier\Contracts\InvoiceRenderer;
use Laravel\Cashier\Invoice;

class ApiInvoiceRenderer implements InvoiceRenderer
{
    /**
     * Kết xuất hóa đơn đã cho và trả về byte PDF thô.
     */
    public function render(Invoice $invoice, array $data = [], array $options = []): string
    {
        $html = $invoice->view($data)->render();

        return Http::get('https://example.com/html-to-pdf', ['html' => $html])->get()->body();
    }
}
```

Sau khi bạn đã triển khai hợp đồng trình kết xuất hóa đơn, bạn nên cập nhật giá trị cấu hình `cashier.invoices.renderer` trong tệp cấu hình `config/cashier.php` của ứng dụng của bạn. Giá trị cấu hình này nên được đặt thành tên lớp của triển khai trình kết xuất tùy chỉnh của bạn.

<a name="checkout"></a>
## Checkout

Cashier Stripe cũng cung cấp hỗ trợ cho [Stripe Checkout](https://stripe.com/payments/checkout). Stripe Checkout loại bỏ nỗi đau khi triển khai các trang tùy chỉnh để chấp nhận thanh toán bằng cách cung cấp một trang thanh toán được lưu trữ, được xây dựng sẵn.

Tài liệu sau chứa thông tin về cách bắt đầu sử dụng Stripe Checkout với Cashier. Để tìm hiểu thêm về Stripe Checkout, bạn cũng nên xem xét xem lại [tài liệu của Stripe về Checkout](https://stripe.com/docs/payments/checkout).

<a name="product-checkouts"></a>
### Checkout Sản Phẩm

Bạn có thể thực hiện checkout cho một sản phẩm hiện có đã được tạo trong bảng điều khiển Stripe của bạn bằng phương thức `checkout` trên mô hình có thể tính phí. Phương thức `checkout` sẽ khởi tạo một phiên Stripe Checkout mới. Theo mặc định, bạn được yêu cầu chuyển một ID Giá Stripe:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout('price_tshirt');
});
```

Nếu cần, bạn cũng có thể chỉ định số lượng sản phẩm:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 15]);
});
```

Khi khách hàng truy cập route này, họ sẽ được chuyển hướng đến trang Checkout của Stripe. Theo mặc định, khi người dùng hoàn tất thành công hoặc hủy một mua hàng, họ sẽ được chuyển hướng đến vị trí route `home` của bạn, nhưng bạn có thể chỉ định các URL gọi lại tùy chỉnh bằng cách sử dụng các tùy chọn `success_url` và `cancel_url`:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

Khi định nghĩa tùy chọn checkout `success_url` của bạn, bạn có thể hướng dẫn Stripe thêm ID phiên checkout làm tham số chuỗi truy vấn khi gọi URL của bạn. Để làm như vậy, hãy thêm chuỗi ký tự `{CHECKOUT_SESSION_ID}` vào chuỗi truy vấn `success_url` của bạn. Stripe sẽ thay thế trình giữ chỗ này bằng ID phiên checkout thực tế:

```php
use Illuminate\Http\Request;
use Stripe\Checkout\Session;
use Stripe\Customer;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
    ]);
});

Route::get('/checkout-success', function (Request $request) {
    $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

    return view('checkout.success', ['checkoutSession' => $checkoutSession]);
})->name('checkout-success');
```

<a name="checkout-promotion-codes"></a>
#### Mã Khuyến Mãi

Theo mặc định, Stripe Checkout không cho phép [mã khuyến mãi có thể dùng được bởi người dùng](https://stripe.com/docs/billing/subscriptions/discounts/codes). May mắn thay, có một cách dễ dàng để kích hoạt các mã này cho trang Checkout của bạn. Để làm như vậy, bạn có thể gọi phương thức `allowPromotionCodes`:

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()
        ->allowPromotionCodes()
        ->checkout('price_tshirt');
});
```

<a name="single-charge-checkouts"></a>
### Checkout Khoản Phí Đơn Lẻ

Bạn cũng có thể thực hiện một khoản phí đơn giản cho một sản phẩm tùy ý chưa được tạo trong bảng điều khiển Stripe của bạn. Để làm như vậy, bạn có thể sử dụng phương thức `checkoutCharge` trên mô hình có thể tính phí và chuyển cho nó một số tiền có thể tính phí, tên sản phẩm và số lượng tùy chọn. Khi khách hàng truy cập route này, họ sẽ được chuyển hướng đến trang Checkout của Stripe:

```php
use Illuminate\Http\Request;

Route::get('/charge-checkout', function (Request $request) {
    return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
});
```

> [!WARNING]
> Khi sử dụng phương thức `checkoutCharge`, Stripe sẽ luôn tạo một sản phẩm và giá mới trong bảng điều khiển Stripe của bạn. Do đó, chúng tôi khuyến nghị rằng bạn tạo các sản phẩm trước trong bảng điều khiển Stripe của bạn và sử dụng phương thức `checkout` thay thế.

<a name="subscription-checkouts"></a>
### Checkout Gói Đăng Ký

> [!WARNING]
> Sử dụng Stripe Checkout cho các gói đăng ký yêu cầu bạn kích hoạt webhook `customer.subscription.created` trong bảng điều khiển Stripe của bạn. Webhook này sẽ tạo bản ghi gói đăng ký trong cơ sở dữ liệu của bạn và lưu trữ tất cả các mục gói đăng ký liên quan.

Bạn cũng có thể sử dụng Stripe Checkout để khởi tạo các gói đăng ký. Sau khi định nghĩa gói đăng ký của bạn với các phương thức xây dựng gói đăng ký của Cashier, bạn có thể gọi phương thức `checkout`. Khi khách hàng truy cập route này, họ sẽ được chuyển hướng đến trang Checkout của Stripe:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout();
});
```

Giống như với checkout sản phẩm, bạn có thể tùy chỉnh các URL thành công và hủy:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

Tất nhiên, bạn cũng có thể kích hoạt mã khuyến mãi cho checkout gói đăng ký:

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->allowPromotionCodes()
        ->checkout();
});
```

> [!WARNING]
> Thật không may, Stripe Checkout không hỗ trợ tất cả các tùy chọn thanh toán gói đăng ký khi bắt đầu các gói đăng ký. Sử dụng phương thức `anchorBillingCycleOn` trên trình xây dựng gói đăng ký, đặt hành vi điều chỉnh, hoặc đặt hành vi thanh toán sẽ không có bất kỳ hiệu quả nào trong các phiên Stripe Checkout. Vui lòng tham khảo [tài liệu API Phiên Stripe Checkout](https://stripe.com/docs/api/checkout/sessions/create) để xem lại các tham số nào có sẵn.

<a name="stripe-checkout-trial-periods"></a>
#### Stripe Checkout và Thời Gian Dùng Thử

Tất nhiên, bạn có thể định nghĩa một thời gian dùng thử khi xây dựng một gói đăng ký sẽ được hoàn tất bằng cách sử dụng Stripe Checkout:

```php
$checkout = Auth::user()->newSubscription('default', 'price_monthly')
    ->trialDays(3)
    ->checkout();
```

Tuy nhiên, thời gian dùng thử phải ít nhất là 48 giờ, là thời gian dùng thử tối thiểu được hỗ trợ bởi Stripe Checkout.

<a name="stripe-checkout-subscriptions-and-webhooks"></a>
#### Gói Đăng Ký và Webhooks
Hãy nhớ rằng, Stripe và Cashier cập nhật trạng thái đăng ký thông qua webhooks, vì vậy có khả năng đăng ký có thể chưa được kích hoạt khi khách hàng quay lại ứng dụng sau khi nhập thông tin thanh toán của họ. Để xử lý tình huống này, bạn có thể muốn hiển thị thông báo cho người dùng biết rằng thanh toán hoặc đăng ký của họ đang chờ xử lý.

<a name="collecting-tax-ids"></a>
### Thu thập Mã Thuế

Checkout cũng hỗ trợ thu thập Mã Thuế của khách hàng. Để bật tính này cho một phiên checkout, hãy gọi phương thức `collectTaxIds` khi tạo phiên:

```php
$checkout = $user->collectTaxIds()->checkout('price_tshirt');
```

Khi phương thức này được gọi, một checkbox mới sẽ có sẵn cho khách hàng cho phép họ chỉ định xem họ có đang mua dưới tư cách công ty hay không. Nếu có, họ sẽ có cơ hội cung cấp số Mã Thuế của mình.

> [!WARNING]
> Nếu bạn đã cấu hình [tự động thu thuế](#tax-configuration) trong service provider của ứng dụng thì tính năng này sẽ được bật tự động và không cần gọi phương thức `collectTaxIds`.

<a name="guest-checkouts"></a>
### Checkout Khách

Sử dụng phương thức `Checkout::guest`, bạn có thể khởi tạo các phiên checkout cho khách của ứng dụng không có "tài khoản":

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()->create('price_tshirt', [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

Tương tự như khi tạo phiên checkout cho người dùng hiện có, bạn có thể sử dụng các phương thức bổ sung có sẵn trên instance `Laravel\Cashier\CheckoutBuilder` để tùy chỉnh phiên checkout khách:

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()
        ->withPromotionCode('promo-code')
        ->create('price_tshirt', [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

Sau khi checkout khách đã hoàn tất, Stripe có thể gửi sự kiện webhook `checkout.session.completed`, vì vậy hãy đảm bảo [cấu hình Stripe webhook của bạn](https://dashboard.stripe.com/webhooks) để thực sự gửi sự kiện này đến ứng dụng của bạn. Sau khi webhook đã được bật trong Stripe dashboard, bạn có thể [xử lý webhook với Cashier](#handling-stripe-webhooks). Object chứa trong payload webhook sẽ là một [checkout object](https://stripe.com/docs/api/checkout/sessions/object) mà bạn có thể kiểm tra để hoàn thành đơn hàng của khách hàng.

<a name="handling-failed-payments"></a>
## Xử lý Thanh toán Thất bại

Đôi khi, thanh toán cho đăng ký hoặc phí đơn lẻ có thể thất bại. Khi điều này xảy ra, Cashier sẽ ném ngoại lệ `Laravel\Cashier\Exceptions\IncompletePayment` thông báo cho bạn về việc này. Sau khi bắt ngoại lệ này, bạn có hai lựa chọn về cách tiếp tục.

Đầu tiên, bạn có thể chuyển hướng khách hàng của mình đến trang xác nhận thanh toán chuyên biệt được đi kèm với Cashier. Trang này đã có một named route liên kết được đăng ký qua service provider của Cashier. Vì vậy, bạn có thể bắt ngoại lệ `IncompletePayment` và chuyển hướng người dùng đến trang xác nhận thanh toán:

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $subscription = $user->newSubscription('default', 'price_monthly')
        ->create($paymentMethod);
} catch (IncompletePayment $exception) {
    return redirect()->route(
        'cashier.payment',
        [$exception->payment->id, 'redirect' => route('home')]
    );
}
```

Trên trang xác nhận thanh toán, khách hàng sẽ được nhắc nhập lại thông tin thẻ tín dụng của họ và thực hiện bất kỳ hành động bổ sung nào được yêu cầu bởi Stripe, chẳng hạn như xác nhận "3D Secure". Sau khi xác nhận thanh toán của họ, người dùng sẽ được chuyển hướng đến URL được cung cấp bởi tham số `redirect` được chỉ định ở trên. Khi chuyển hướng, các biến query string `message` (string) và `success` (integer) sẽ được thêm vào URL. Trang thanh toán hiện hỗ trợ các loại phương thức thanh toán sau:

<div class="content-list" markdown="1">

- Thẻ Tín dụng
- Alipay
- Bancontact
- BECS Direct Debit
- EPS
- Giropay
- iDEAL
- SEPA Direct Debit

</div>

Ngoài ra, bạn có thể cho phép Stripe xử lý xác nhận thanh toán cho bạn. Trong trường hợp này, thay vì chuyển hướng đến trang xác nhận thanh toán, bạn có thể [thiết lập email thanh toán tự động của Stripe](https://dashboard.stripe.com/account/billing/automatic) trong Stripe dashboard của bạn. Tuy nhiên, nếu ngoại lệ `IncompletePayment` được bắt, bạn vẫn nên thông báo cho người dùng rằng họ sẽ nhận được email với hướng dẫn xác nhận thanh toán thêm.

Ngoại lệ thanh toán có thể được ném cho các phương thức sau: `charge`, `invoiceFor`, và `invoice` trên các model sử dụng trait `Billable`. Khi tương tác với đăng ký, phương thức `create` trên `SubscriptionBuilder`, và các phương thức `incrementAndInvoice` và `swapAndInvoice` trên các model `Subscription` và `SubscriptionItem` có thể ném ngoại lệ thanh toán chưa hoàn thành.

Xác định xem một đăng ký hiện có có thanh toán chưa hoàn thành hay không có thể được thực hiện bằng cách sử dụng phương thức `hasIncompletePayment` trên model billable hoặc một instance đăng ký:

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

Bạn có thể lấy trạng thái cụ thể của một thanh toán chưa hoàn thành bằng cách kiểm tra thuộc tính `payment` trên instance ngoại lệ:

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $user->charge(1000, 'pm_card_threeDSecure2Required');
} catch (IncompletePayment $exception) {
    // Lấy trạng thái payment intent...
    $exception->payment->status;

    // Kiểm tra các điều kiện cụ thể...
    if ($exception->payment->requiresPaymentMethod()) {
        // ...
    } elseif ($exception->payment->requiresConfirmation()) {
        // ...
    }
}
```

<a name="confirming-payments"></a>
### Xác nhận Thanh toán

Một số phương thức thanh toán yêu cầu dữ liệu bổ sung để xác nhận thanh toán. Ví dụ, các phương thức thanh toán SEPA yêu cầu dữ liệu "mandate" bổ sung trong quá trình thanh toán. Bạn có thể cung cấp dữ liệu này cho Cashier bằng phương thức `withPaymentConfirmationOptions`:

```php
$subscription->withPaymentConfirmationOptions([
    'mandate_data' => '...',
])->swap('price_xxx');
```

Bạn có thể tham khảo [tài liệu API của Stripe](https://stripe.com/docs/api/payment_intents/confirm) để xem tất cả các tùy chọn được chấp nhận khi xác nhận thanh toán.

<a name="strong-customer-authentication"></a>
## Xác thực Khách hàng Mạnh (Strong Customer Authentication)

Nếu doanh nghiệp của bạn hoặc một trong số khách hàng của bạn có trụ sở tại Châu Âu, bạn sẽ cần tuân thủ các quy định Xác thực Khách hàng Mạnh (SCA) của EU. Các quy định này được áp đặt vào tháng 9 năm 2019 bởi Liên minh Châu Âu để ngăn chặn gian lận thanh toán. May mắn thay, Stripe và Cashier đã chuẩn bị để xây dựng các ứng dụng tuân thủ SCA.

> [!WARNING]
> Trước khi bắt đầu, hãy xem lại [hướng dẫn của Stripe về PSD2 và SCA](https://stripe.com/guides/strong-customer-authentication) cũng như [tài liệu của họ về các API SCA mới](https://stripe.com/docs/strong-customer-authentication).

<a name="payments-requiring-additional-confirmation"></a>
### Thanh toán Yêu cầu Xác nhận Bổ sung

Quy định SCA thường yêu cầu xác minh thêm để xác nhận và xử lý thanh toán. Khi điều này xảy ra, Cashier sẽ ném ngoại lệ `Laravel\Cashier\Exceptions\IncompletePayment` thông báo cho bạn rằng cần xác minh thêm. Thông tin thêm về cách xử lý các ngoại lệ này có thể được tìm thấy trong tài liệu về [xử lý thanh toán thất bại](#handling-failed-payments).

Màn hình xác nhận thanh toán được trình bày bởi Stripe hoặc Cashier có thể được tùy chỉnh cho quy trình thanh toán của một ngân hàng hoặc nhà phát hành thẻ cụ thể và có thể bao gồm xác nhận thẻ bổ sung, một khoản phí nhỏ tạm thời, xác nhận thiết bị riêng biệt, hoặc các hình thức xác minh khác.

<a name="incomplete-and-past-due-state"></a>
#### Trạng thái Chưa hoàn thành và Quá hạn

Khi thanh toán cần xác nhận thêm, đăng ký sẽ vẫn ở trạng thái `incomplete` hoặc `past_due` như được chỉ định bởi cột cơ sở dữ liệu `stripe_status` của nó. Cashier sẽ tự động kích hoạt đăng ký của khách hàng ngay khi xác nhận thanh toán hoàn tất và ứng dụng của bạn được thông báo bởi Stripe qua webhook về việc hoàn tất.

Để biết thêm thông tin về các trạng thái `incomplete` và `past_due`, vui lòng tham khảo [tài liệu bổ sung của chúng tôi về các trạng thái này](#incomplete-and-past-due-status).

<a name="off-session-payment-notifications"></a>
### Thông báo Thanh toán Off-Session

Vì quy định SCA yêu cầu khách hàng đôi khi phải xác minh chi tiết thanh toán của họ ngay cả khi đăng ký của họ đang hoạt động, Cashier có thể gửi thông báo cho khách hàng khi cần xác nhận thanh toán off-session. Ví dụ, điều này có thể xảy ra khi đăng ký đang gia hạn. Thông báo thanh toán của Cashier có thể được bật bằng cách đặt biến môi trường `CASHIER_PAYMENT_NOTIFICATION` thành một lớp thông báo. Theo mặc định, thông báo này bị tắt. Tất nhiên, Cashier bao gồm một lớp thông báo mà bạn có thể sử dụng cho mục đích này, nhưng bạn có thể tự cung cấp lớp thông báo của riêng mình nếu muốn:

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

Để đảm bảo rằng thông báo xác nhận thanh toán off-session được gửi đi, hãy xác minh rằng [Stripe webhooks được cấu hình](#handling-stripe-webhooks) cho ứng dụng của bạn và webhook `invoice.payment_action_required` được bật trong Stripe dashboard của bạn. Ngoài ra, model `Billable` của bạn cũng nên sử dụng trait `Illuminate\Notifications\Notifiable` của Laravel.

> [!WARNING]
> Thông báo sẽ được gửi ngay cả khi khách hàng đang thực hiện thanh toán thủ công yêu cầu xác nhận thêm. Thật không may, không có cách nào để Stripe biết rằng thanh toán được thực hiện thủ công hay "off-session". Tuy nhiên, khách hàng sẽ chỉ thấy thông báo "Thanh toán Thành công" nếu họ truy cập trang thanh toán sau khi đã xác nhận thanh toán của họ. Khách hàng sẽ không được phép vô tình xác nhận cùng một thanh toán hai lần và chịu một khoản phí thứ hai vô tình.

<a name="stripe-sdk"></a>
## Stripe SDK

Nhiều object của Cashier là các wrapper xung quanh các object Stripe SDK. Nếu bạn muốn tương tác trực tiếp với các object Stripe, bạn có thể lấy chúng một cách thuận tiện bằng phương thức `asStripe`:

```php
$stripeSubscription = $subscription->asStripeSubscription();

$stripeSubscription->application_fee_percent = 5;

$stripeSubscription->save();
```

Bạn cũng có thể sử dụng phương thức `updateStripeSubscription` để cập nhật trực tiếp một đăng ký Stripe:

```php
$subscription->updateStripeSubscription(['application_fee_percent' => 5]);
```

Bạn có thể gọi phương thức `stripe` trên lớp `Cashier` nếu bạn muốn sử dụng trực tiếp client `Stripe\StripeClient`. Ví dụ, bạn có thể sử dụng phương thức này để truy cập instance `StripeClient` và lấy danh sách giá từ tài khoản Stripe của bạn:

```php
use Laravel\Cashier\Cashier;

$prices = Cashier::stripe()->prices->all();
```

<a name="testing"></a>
## Kiểm thử (Testing)

Khi kiểm thử một ứng dụng sử dụng Cashier, bạn có thể mock các yêu cầu HTTP thực tế đến API Stripe; tuy nhiên, điều này yêu cầu bạn phải triển khai lại một phần hành vi của Cashier. Do đó, chúng tôi khuyên bạn nên cho phép các kiểm thử của bạn truy cập API Stripe thực tế. Mặc dù điều này chậm hơn, nó cung cấp sự tự tin hơn rằng ứng dụng của bạn đang hoạt động như mong đợi và bất kỳ kiểm thử chậm nào có thể được đặt trong nhóm kiểm thử Pest / PHPUnit riêng của chúng.

Khi kiểm thử, hãy nhớ rằng Cashier bản thân đã có một bộ kiểm thử tuyệt vời, vì vậy bạn chỉ nên tập trung vào kiểm thử quy trình đăng ký và thanh toán của ứng dụng riêng của bạn chứ không phải mọi hành vi Cashier cơ bản.

Để bắt đầu, hãy thêm phiên bản **testing** của Stripe secret của bạn vào file `phpunit.xml` của bạn:

```xml
<env name="STRIPE_SECRET" value="sk_test_<your-key>"/>
```

Bây giờ, bất cứ khi nào bạn tương tác với Cashier trong khi kiểm thử, nó sẽ gửi các yêu cầu API thực tế đến môi trường kiểm thử Stripe của bạn. Để thuận tiện, bạn nên điền trước tài khoản kiểm thử Stripe của mình với các đăng ký / giá mà bạn có thể sử dụng trong quá trình kiểm thử.

> [!NOTE]
> Để kiểm thử nhiều tình huống thanh toán khác nhau, chẳng hạn như từ chối thẻ tín dụng và thất bại, bạn có thể sử dụng phạm vi rộng lớn các [số thẻ và token kiểm thử](https://stripe.com/docs/testing) được cung cấp bởi Stripe.

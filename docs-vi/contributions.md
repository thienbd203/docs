# Contribution Guide

- [Bug Reports](#bug-reports)
- [Support Questions](#support-questions)
- [Core Development Discussion](#core-development-discussion)
- [Which Branch?](#which-branch)
- [Compiled Assets](#compiled-assets)
- [AI-Generated Contributions](#ai-generated-contributions)
- [Security Vulnerabilities](#security-vulnerabilities)
- [Coding Style](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)
- [Code of Conduct](#code-of-conduct)

<a name="bug-reports"></a>
## Bug Reports

Để khuyến khích hợp tác tích cực, Laravel khuyến khích mạnh các pull requests, không chỉ các báo cáo lỗi. Pull requests chỉ sẽ được xem xét khi được đánh dấu là "ready for review" (không ở trạng thái "draft") và tất cả các tests cho các tính năng mới đều vượt qua. Các pull request còn sót, không hoạt động ở trạng thái "draft" sẽ bị đóng sau vài ngày.

Tuy nhiên, nếu bạn gửi một báo cáo lỗi, vấn đề của bạn nên chứa một tiêu đề và mô tả rõ ràng về vấn đề. Bạn cũng nên bao gồm càng nhiều thông tin liên quan càng tốt và một mẫu mã thể hiện vấn đề. Mục tiêu của một báo cáo lỗi là làm cho chính bạn - và những người khác - dễ dàng tái tạo lỗi và phát triển một bản sửa.

Hãy nhớ rằng, các báo cáo lỗi được tạo với hy vọng những người khác có cùng vấn đề sẽ có thể hợp tác với bạn để giải quyết nó. Đừng mong đợi rằng báo cáo lỗi sẽ tự động thấy bất kỳ hoạt động nào hoặc những người khác sẽ nhảy vào để sửa nó. Tạo một báo cáo lỗi phục vụ để giúp chính bạn và những người khác bắt đầu trên con đường giải quyết vấn đề. Nếu bạn muốn đóng góp, bạn có thể giúp bằng cách sửa [bất kỳ lỗi nào được liệt kê trong các issue trackers của chúng tôi](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel). Bạn phải được xác thực với GitHub để xem tất cả các vấn đề của Laravel.

Nếu bạn nhận thấy DocBlock không chính xác, PHPStan, hoặc các cảnh báo IDE trong khi sử dụng Laravel, đừng tạo một GitHub issue. Thay vào đó, hãy gửi một pull request để sửa vấn đề.

Mã nguồn Laravel được quản lý trên GitHub, và có các repositories cho mỗi dự án Laravel:

<div class="content-list" markdown="1">

- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Boost](https://github.com/laravel/boost)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Folio](https://github.com/laravel/folio)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Pennant](https://github.com/laravel/pennant)
- [Laravel Pint](https://github.com/laravel/pint)
- [Laravel Prompts](https://github.com/laravel/prompts)
- [Laravel Reverb](https://github.com/laravel/reverb)
- [Laravel Sail](https://github.com/laravel/sail)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel Livewire Starter Kit](https://github.com/laravel/livewire-starter-kit)
- [Laravel React Starter Kit](https://github.com/laravel/react-starter-kit)
- [Laravel Svelte Starter Kit](https://github.com/laravel/svelte-starter-kit)
- [Laravel Vue Starter Kit](https://github.com/laravel/vue-starter-kit)

</div>

<a name="support-questions"></a>
## Support Questions

Các issue trackers GitHub của Laravel không được dự định để cung cấp trợ giúp hoặc hỗ trợ Laravel. Thay vào đó, sử dụng một trong các kênh sau:

<div class="content-list" markdown="1">

- [GitHub Discussions](https://github.com/laravel/framework/discussions)
- [Laracasts Forums](https://laracasts.com/discuss)
- [Laravel.io Forums](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

</div>

<a name="core-development-discussion"></a>
## Core Development Discussion

Bạn có thể đề xuất các tính năng mới hoặc cải tiến hành vi Laravel hiện có trong [GitHub discussion board](https://github.com/laravel/framework/discussions) của repository framework Laravel. Nếu bạn đề xuất một tính năng mới, hãy sẵn sàng thực hiện ít nhất một phần mã sẽ cần thiết để hoàn thành tính năng.

Thảo luận không chính thức về các lỗi, tính năng mới, và triển khai các tính năng hiện có diễn ra trong kênh `#internals` của [server Discord Laravel](https://discord.gg/laravel). Taylor Otwell, người duy trì Laravel, thường có mặt trong kênh vào các ngày làm việc từ 8am-5pm (UTC-06:00 hoặc America/Chicago), và thỉnh thoảng có mặt trong kênh vào các thời điểm khác.

<a name="which-branch"></a>
## Which Branch?

**Tất cả** các bản sửa lỗi nên được gửi đến phiên bản mới nhất hỗ trợ các bản sửa lỗi (hiện tại là `13.x`). Các bản sửa lỗi không bao giờ nên được gửi đến nhánh `master` trừ khi chúng sửa các tính năng chỉ tồn tại trong bản phát hành sắp tới.

Các tính năng **nhỏ** **hoàn toàn tương thích ngược** với bản phát hành hiện tại có thể được gửi đến nhánh ổn định mới nhất (hiện tại là `13.x`).

Các tính năng **lớn** mới hoặc các tính năng có breaking changes nên luôn được gửi đến nhánh `master`, chứa bản phát hành sắp tới.

<a name="compiled-assets"></a>
## Compiled Assets

Nếu bạn gửi một thay đổi sẽ ảnh hưởng đến một file được biên dịch, chẳng hạn như hầu hết các file trong `resources/css` hoặc `resources/js` của repository `laravel/laravel`, đừng commit các file được biên dịch. Do kích thước lớn của chúng, chúng không thể thực tế được xem xét bởi một người duy trì. Điều này có thể được khai thác như một cách để inject mã độc vào Laravel. Để phòng thủ ngăn chặn điều này, tất cả các file được biên dịch sẽ được tạo và commit bởi những người duy trì Laravel.

<a name="ai-generated-contributions"></a>
## AI-Generated Contributions

Chúng tôi trân trọng mọi pull request được gửi đến Laravel. Tuy nhiên, các đóng góp chủ yếu được tạo bởi AI mà không có sự xem xét và cân nhắc kỹ lưỡng của con người là không thể chấp nhận.

Nếu bạn chọn sử dụng các công cụ AI để hỗ trợ với đóng góp của mình, mã kết quả phải được xem xét, kiểm tra, và hiểu kỹ lưỡng bởi bạn trước khi gửi.

**Mass mở các issues hoặc pull requests hoàn toàn được tạo bởi AI sẽ không được dung thứ.** Các pull requests như vậy sẽ bị đóng mà không cần xem xét, và người đóng góp có thể bị chặn khỏi repository.

Chúng tôi khuyến khích những người đóng góp làm quen với codebase hiện có, tham gia với cộng đồng, và gửi các pull requests phản ánh sự hiểu biết và cân nhắc kỹ lưỡng của chính họ về vấn đề họ đang giải quyết.

<a name="security-vulnerabilities"></a>
## Security Vulnerabilities

Nếu bạn phát hiện một lỗ hổng bảo mật trong Laravel, hãy gửi email cho Taylor Otwell tại <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. Tất cả các lỗ hổng bảo mật sẽ được giải quyết kịp thời.

<a name="coding-style"></a>
## Coding Style

Laravel tuân theo tiêu chuẩn coding [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) và tiêu chuẩn autoloading [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md).

<a name="phpdoc"></a>
### PHPDoc

Dưới đây là một ví dụ về một block tài liệu Laravel hợp lệ. Lưu ý rằng attribute `@param` được theo sau bởi hai khoảng trắng, loại đối số, hai khoảng trắng nữa, và cuối cùng là tên biến:

```php
/**
 * Register a binding with the container.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */
public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```

Khi các attribute `@param` hoặc `@return` thừa thãi do việc sử dụng các types gốc, chúng có thể được xóa:

```php
/**
 * Execute the job.
 * [tl! remove]
 * @return void [tl! remove]
 */
public function handle(AudioProcessor $processor): void
{
    // ...
}
```

Tuy nhiên, khi type gốc là generic, hãy chỉ định type generic thông qua việc sử dụng các attribute `@param` hoặc `@return`:

```php
/**
 * Get the attachments for the message.
 * [tl! add]
 * @return array<int, \Illuminate\Mail\Mailables\Attachment> [tl! add]
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

<a name="styleci"></a>
### StyleCI

Đừng lo lắng nếu phong cách code của bạn không hoàn hảo! [StyleCI](https://styleci.io/) sẽ tự động hợp nhất bất kỳ bản sửa phong cách nào vào repository Laravel sau khi các pull requests được hợp nhất. Điều này cho phép chúng tôi tập trung vào nội dung của đóng góp và không phải phong cách code.

<a name="code-of-conduct"></a>
## Code of Conduct

Code of conduct của Laravel được dẫn xuất từ code of conduct của Ruby. Bất kỳ vi phạm nào của code of conduct có thể được báo cáo cho Taylor Otwell (taylor@laravel.com):

<div class="content-list" markdown="1">

- Người tham gia sẽ dung nạp các quan điểm đối lập.
- Người tham gia phải đảm bảo rằng ngôn ngữ và hành động của họ không có các cuộc tấn công cá nhân và các nhận xét cá nhân miệt thị.
- Khi diễn giải lời nói và hành động của người khác, người tham gia nên luôn giả định ý định tốt.
- Hành vi có thể được coi hợp lý là quấy rối sẽ không được dung thứ.

</div>

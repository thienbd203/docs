# Phát triển có hỗ trợ AI

 - [Giới thiệu](#introduction)
     - [Tại sao chọn Laravel cho phát triển AI?](#why-laravel-for-ai-development)
 - [Laravel Boost](#laravel-boost)
     - [Cài đặt](#installation)
     - [Các công cụ có sẵn](#available-tools)
     - [Hướng dẫn AI](#ai-guidelines)
     - [Kỹ năng Agent](#agent-skills)
     - [Tìm kiếm tài liệu](#documentation-search)
     - [Tích hợp Agents](#agents-integration)

<a name="introduction"></a>
## Giới thiệu

Laravel có vị thế độc đáo là framework tốt nhất cho phát triển có hỗ trợ AI và phát triển dạng agent. Sự trỗi dậy của các AI coding agent như [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [OpenCode](https://opencode.ai), [Cursor](https://cursor.com), và [GitHub Copilot](https://github.com/features/copilot) đã thay đổi cách developers viết code. Các công cụ này có thể tạo ra toàn bộ tính năng, debug các vấn đề phức tạp, và refactor code với tốc độ chưa từng có - nhưng hiệu quả của chúng phụ thuộc nhiều vào mức độ hiểu codebase của bạn.

<a name="why-laravel-for-ai-development"></a>
### Tại sao chọn Laravel cho phát triển AI?

Các quy ước có định hướng và cấu trúc được định nghĩa rõ ràng của Laravel làm cho nó trở thành framework lý tưởng cho phát triển có hỗ trợ AI. Khi bạn yêu cầu một AI agent thêm một controller, nó biết chính xác nơi cần đặt. Khi bạn cần một migration mới, các quy ước đặt tên và vị trí file có thể dự đoán được. Sự nhất quán này loại bỏ sự phỏng đoán thường khiến các công cụ AI gặp khó khăn trong các framework linh hoạt hơn.

Ngoài việc tổ chức file, cú pháp rõ ràng và tài liệu toàn diện của Laravel cung cấp cho AI agents ngữ cảnh cần thiết để tạo ra code chính xác và đúng chuẩn. Các tính năng như Eloquent relationships, form requests, và middleware tuân theo các pattern mà agents có thể hiểu và sao chép một cách đáng tin cậy. Kết quả là code được tạo bởi AI trông giống như được viết bởi một Laravel developer dày dạn kinh nghiệm, không phải được ghép lại từ các đoạn PHP snippet chung chung.

<a name="laravel-boost"></a>
## Laravel Boost

[Laravel Boost](https://github.com/laravel/boost) là cầu nối giữa AI coding agents và ứng dụng Laravel của bạn. Boost là một MCP (Model Context Protocol) server được trang bị hơn 15 công cụ chuyên biệt cung cấp cho AI agents cái nhìn sâu sắc về cấu trúc, database, routes, và nhiều hơn nữa của ứng dụng. Khi bạn cài đặt Boost, AI agent của bạn sẽ chuyển đổi từ một trợ lý code tổng dụng thành một Laravel expert hiểu ứng dụng cụ thể của bạn.

Boost cung cấp ba khả năng chính: một bộ công cụ MCP để kiểm tra và tương tác với ứng dụng, các hướng dẫn AI có thể kết hợp được tạo riêng cho hệ sinh thái Laravel, và một documentation API mạnh mẽ chứa hơn 17,000 mảnh kiến thức đặc thù về Laravel.

<a name="installation"></a>
### Cài đặt

Boost có thể được cài đặt trong các ứng dụng Laravel 10, 11, 12, và 13 chạy PHP 8.1 hoặc cao hơn. Để bắt đầu, cài đặt Boost như một development dependency:

```shell
composer require laravel/boost --dev
```

Sau khi cài đặt, chạy trình cài đặt tương tác:

```shell
php artisan boost:install
```

Trình cài đặt sẽ tự động phát hiện IDE và AI agents của bạn, cho phép bạn chọn các tích hợp phù hợp với dự án của bạn. Boost sẽ tạo ra các file cấu hình cần thiết, như `.mcp.json` cho các editor tương thích MCP và các file hướng dẫn cho ngữ cảnh AI.

> [!NOTE]
> Các file cấu hình được tạo như `.mcp.json`, `CLAUDE.md`, và `boost.json` có thể được thêm an toàn vào `.gitignore` của bạn nếu bạn muốn mỗi developer tự cấu hình môi trường của riêng họ.

<a name="available-tools"></a>
### Các công cụ có sẵn

Boost cung cấp một bộ công cụ toàn diện cho AI agents thông qua Model Context Protocol. Các công cụ này cho phép agents hiểu sâu và tương tác với ứng dụng Laravel của bạn:

<div class="content-list" markdown="1">

- **Application Introspection** - Truy vấn phiên bản PHP và Laravel của bạn, liệt kê các packages đã cài đặt, và kiểm tra cấu trúc ứng dụng và biến môi trường của bạn.
- **Database Tools** - Kiểm tra database schema, thực thi các truy vấn read-only, và hiểu cấu trúc dữ liệu của bạn mà không cần rời khỏi cuộc hội thoại.
- **Route Inspection** - Liệt kê tất cả các routes đã đăng ký với middleware, controllers, và parameters của chúng.
- **Artisan Commands** - Khám phá các Artisan commands có sẵn và arguments của chúng, cho phép agents đề xuất và thực thi các lệnh đúng cho nhiệm vụ của bạn.
- **Log Analysis** - Đọc và phân tích các file log của ứng dụng để giúp debug các vấn đề.
- **Browser Logs** - Truy cập các log và lỗi của browser console khi phát triển với các công cụ frontend của Laravel.
- **Tinker Integration** - Thực thi PHP code trong ngữ cảnh ứng dụng của bạn thông qua Laravel Tinker, cho phép agents kiểm tra các giả thuyết và xác minh hành vi.
- **Documentation Search** - Tìm kiếm tài liệu hệ sinh thái Laravel với kết quả được điều chỉnh theo phiên bản packages đã cài đặt của bạn.

</div>

<a name="ai-guidelines"></a>
### Hướng dẫn AI

Boost bao gồm một bộ hướng dẫn AI toàn diện được tạo riêng cho hệ sinh thái Laravel. Các hướng dẫn này dạy AI agents cách viết code đúng chuẩn Laravel, tuân theo các quy ước framework, và tránh các lỗi phổ biến. Các hướng dẫn có thể kết hợp và có nhận biết phiên bản, nghĩa là agents nhận được các hướng dẫn phù hợp với phiên bản packages chính xác của bạn.

Các hướng dẫn có sẵn cho Laravel chính nó và hơn 16 packages trong hệ sinh thái Laravel, bao gồm:

<div class="content-list" markdown="1">

- Livewire (2.x, 3.x, và 4.x)
- Inertia.js (các biến thể React, Svelte, và Vue)
- Tailwind CSS (3.x và 4.x)
- Filament (3.x và 4.x)
- PHPUnit
- Pest PHP
- Laravel Pint
- Và nhiều hơn nữa

</div>

Khi bạn chạy `boost:install`, Boost tự động phát hiện các packages mà ứng dụng của bạn sử dụng và tổng hợp các hướng dẫn liên quan vào các file ngữ cảnh AI của dự án.

<a name="agent-skills"></a>
### Kỹ năng Agent

[Agent Skills](https://agentskills.io/home) là các module kiến thức nhẹ và tập trung mà agents có thể kích hoạt theo yêu cầu khi làm việc trên các lĩnh vực cụ thể. Khác với các hướng dẫn được tải lên trước, skills cho phép các pattern và best practices chi tiết chỉ được tải khi có liên quan, giảm bloat ngữ cảnh và cải thiện tính liên quan của code được tạo bởi AI.

Skills có sẵn cho các Laravel packages phổ biến như Livewire, Inertia, Tailwind CSS, Pest, và nhiều hơn nữa. Khi bạn chạy `boost:install` và chọn skills như một tính năng, skills được cài đặt tự động dựa trên các packages được phát hiện trong `composer.json` của bạn.

<a name="documentation-search"></a>
### Tìm kiếm tài liệu

Boost bao gồm một documentation API mạnh mẽ cung cấp cho AI agents quyền truy cập vào hơn 17,000 mảnh tài liệu hệ sinh thái Laravel. Khác với các web tìm kiếm chung chung, tài liệu này được lập chỉ mục, vector hóa, và lọc để khớp với phiên bản packages chính xác của bạn.

Khi một agent cần hiểu cách một tính năng hoạt động, nó có thể tìm kiếm documentation API của Boost và nhận thông tin chính xác, cụ thể theo phiên bản. Điều này loại bỏ vấn đề phổ biến của AI agents đề xuất các phương thức hoặc cú pháp đã bị deprecated từ các phiên bản framework cũ hơn.

<a name="agent-integration"></a>
### Tích hợp Agents

Boost tích hợp với các IDE và AI tools phổ biến hỗ trợ Model Context Protocol. Để biết hướng dẫn thiết lập chi tiết cho Cursor, Claude Code, Codex, Gemini CLI, GitHub Copilot, và Junie, xem phần [Set Up Your Agents](/docs/{{version}}/boost#set-up-your-agents) của tài liệu Boost.

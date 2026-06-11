# Laravel Boost

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Thiết lập Agents của bạn](#set-up-your-agents)
    - [Giữ cập nhật Boost Resources](#keeping-boost-resources-updated)
- [MCP Server](#mcp-server)
    - [Các MCP Tools có sẵn](#available-mcp-tools)
    - [Đăng ký thủ công MCP Server](#manually-registering-the-mcp-server)
- [AI Guidelines](#ai-guidelines)
    - [Các AI Guidelines có sẵn](#available-ai-guidelines)
    - [Thêm Custom AI Guidelines](#adding-custom-ai-guidelines)
    - [Ghi đè Boost AI Guidelines](#overriding-boost-ai-guidelines)
    - [AI Guidelines cho Third-Party Package](#third-party-package-ai-guidelines)
- [Agent Skills](#agent-skills)
    - [Các Skills có sẵn](#available-skills)
    - [Custom Skills](#custom-skills)
    - [Ghi đè Skills](#overriding-skills)
    - [Skills cho Third-Party Package](#third-party-package-skills)
- [Guidelines vs. Skills](#guidelines-vs-skills)
- [Documentation API](#documentation-api)
- [Mở rộng Boost](#extending-boost)
    - [Thêm hỗ trợ cho các IDEs / AI Agents khác](#adding-support-for-other-ides-ai-agents)

<a name="introduction"></a>
## Giới thiệu

Laravel Boost tăng tốc phát triển có hỗ trợ AI bằng cách cung cấp các hướng dẫn và agent skills thiết yếu giúp AI agents viết các ứng dụng Laravel chất lượng cao tuân theo các best practices của Laravel.

Boost cũng cung cấp một documentation API hệ sinh thái Laravel mạnh mẽ kết hợp một MCP tool tích hợp với một knowledge base rộng lớn chứa hơn 17,000 mảnh thông tin đặc thù về Laravel, tất cả được nâng cao bởi các khả năng tìm kiếm ngữ nghĩa sử dụng embeddings để có kết quả chính xác và có ngữ cảnh. Boost hướng dẫn các AI agents như Claude Code và Cursor sử dụng API này để tìm hiểu về các tính năng và best practices mới nhất của Laravel.

<a name="installation"></a>
## Cài đặt

Laravel Boost có thể được cài đặt thông qua Composer:

```shell
composer require laravel/boost --dev
```

Tiếp theo, cài đặt MCP server và coding guidelines:

```shell
php artisan boost:install
```

Lệnh `boost:install` sẽ tạo ra các file guideline và skill agent tương ứng cho các coding agents bạn đã chọn trong quá trình cài đặt.

Sau khi Laravel Boost đã được cài đặt, bạn đã sẵn sàng để bắt đầu code với Cursor, Claude Code, hoặc AI agent lựa chọn của bạn.

> [!NOTE]
> Bạn có thể thêm file cấu hình MCP được tạo (`.mcp.json`), các file guideline (`CLAUDE.md`, `AGENTS.md`, `junie/`, v.v.), và file cấu hình `boost.json` vào `.gitignore` của ứng dụng, vì các file này được tự động tạo lại khi chạy `boost:install` và `boost:update`.

<a name="set-up-your-agents"></a>
### Thiết lập Agents của bạn

```text tab=Cursor
1. Mở command palette (`Cmd+Shift+P` hoặc `Ctrl+Shift+P`)
2. Nhấn `enter` trên "/open MCP Settings"
3. Bật toggle cho `laravel-boost`
```

```text tab=Claude Code
Claude Code support thường được kích hoạt tự động. Nếu bạn thấy nó không được kích hoạt, mở một shell trong thư mục dự án và chạy lệnh sau:

claude mcp add -s local -t stdio laravel-boost php artisan boost:mcp
```

```text tab=Codex
Codex support thường được kích hoạt tự động. Nếu bạn thấy nó không được kích hoạt, mở một shell trong thư mục dự án và chạy lệnh sau:

codex mcp add laravel-boost -- php "artisan" "boost:mcp"
```

```text tab=Gemini CLI
Gemini CLI support thường được kích hoạt tự động. Nếu bạn thấy nó không được kích hoạt, mở một shell trong thư mục dự án và chạy lệnh sau:

gemini mcp add -s project -t stdio laravel-boost php artisan boost:mcp
```

```text tab=GitHub Copilot (VS Code)
1. Mở command palette (`Cmd+Shift+P` hoặc `Ctrl+Shift+P`)
2. Nhấn `enter` trên "MCP: List Servers"
3. Mũi tên đến `laravel-boost` và nhấn `enter`
4. Chọn "Start server"
```

```text tab=Junie
1. Nhấn `shift` hai lần để mở command palette
2. Tìm kiếm "MCP Settings" và nhấn `enter`
3. Check box bên cạnh `laravel-boost`
4. Click "Apply" ở góc dưới bên phải
```

<a name="keeping-boost-resources-updated"></a>
### Giữ cập nhật Boost Resources

Bạn có thể muốn cập nhật định kỳ các Boost resources cục bộ của bạn (AI guidelines và skills) để đảm bảo chúng phản ánh các phiên bản mới nhất của các packages hệ sinh thái Laravel bạn đã cài đặt. Để làm điều này, bạn có thể sử dụng lệnh Artisan `boost:update`.

```shell
php artisan boost:update
```

Bạn cũng có thể tự động hóa quá trình này bằng cách thêm nó vào các scripts "post-update-cmd" của Composer:

```json
{
  "scripts": {
    "post-update-cmd": [
      "@php artisan boost:update --ansi"
    ]
  }
}
```

Theo mặc định, lệnh `boost:update` sẽ chỉ cập nhật các Boost resources hiện có đã được publish trong ứng dụng của bạn. Nếu bạn muốn Boost quét ứng dụng của bạn cho bất kỳ packages mới được cài đặt và đề xuất publish các guidelines và skills tương ứng của chúng, bạn có thể sử dụng tùy chọn `--discover`:

```shell
php artisan boost:update --discover
```

<a name="mcp-server"></a>
## MCP Server

Laravel Boost cung cấp một MCP (Model Context Protocol) server expose các tools cho AI agents tương tác với ứng dụng Laravel của bạn. Các tools này cung cấp cho agents khả năng kiểm tra cấu trúc ứng dụng, query database, thực thi code, và nhiều hơn nữa.

<a name="available-mcp-tools"></a>
### Các MCP Tools có sẵn

| Name                 | Notes                                                                                                       |
| -------------------- | ----------------------------------------------------------------------------------------------------------- |
| Application Info     | Đọc phiên bản PHP & Laravel, database engine, danh sách các packages hệ sinh thái với phiên bản, và Eloquent models |
| Browser Logs         | Đọc logs và errors từ browser                                                                       |
| Database Connections | Kiểm tra các database connections có sẵn, bao gồm connection mặc định                                    |
| Database Query       | Thực thi một query trên database                                                                        |
| Database Schema      | Đọc database schema                                                                                    |
| Get Absolute URL     | Chuyển đổi relative path URIs thành absolute để agents tạo valid URLs                                        |
| Last Error           | Đọc error cuối cùng từ các file log của ứng dụng                                                        |
| Read Log Entries     | Đọc N log entries cuối cùng                                                                                 |
| Search Docs          | Query service documentation API được host bởi Laravel để truy xuất documentation dựa trên các packages đã cài đặt    |

<a name="manually-registering-the-mcp-server"></a>
### Đăng ký thủ công MCP Server

Đôi khi bạn có thể cần đăng ký thủ công Laravel Boost MCP server với editor lựa chọn của bạn. Bạn nên đăng ký MCP server sử dụng các chi tiết sau:

<table>
<tr><td><strong>Command</strong></td><td><code>php</code></td></tr>
<tr><td><strong>Args</strong></td><td><code>artisan boost:mcp</code></td></tr>
</table>

Ví dụ JSON:

```json
{
    "mcpServers": {
        "laravel-boost": {
            "command": "php",
            "args": ["artisan", "boost:mcp"]
        }
    }
}
```

<a name="ai-guidelines"></a>
## AI Guidelines

AI guidelines là các file hướng dẫn có thể kết hợp được tải lên trước để cung cấp cho AI agents ngữ cảnh thiết yếu về các packages hệ sinh thái Laravel. Các guidelines này chứa các quy ước cốt lõi, best practices, và các pattern đặc thù framework giúp agents tạo ra code nhất quán và chất lượng cao.

<a name="available-ai-guidelines"></a>
### Các AI Guidelines có sẵn

Laravel Boost bao gồm AI guidelines cho các packages và frameworks sau. Các guidelines `core` cung cấp lời khuyên chung, tổng quát cho AI cho package đã cho áp dụng trên tất cả các phiên bản.

| Package           | Versions Supported     |
| ----------------- | ---------------------- |
| Core & Boost      | core                   |
| Laravel Framework | core, 10.x, 11.x, 12.x, 13.x |
| Livewire          | core, 2.x, 3.x, 4.x    |
| Flux UI           | core, free, pro        |
| Folio             | core                   |
| Herd              | core                   |
| Inertia Laravel   | core, 1.x, 2.x, 3.x    |
| Inertia React     | core, 1.x, 2.x, 3.x    |
| Inertia Vue       | core, 1.x, 2.x, 3.x    |
| Inertia Svelte    | core, 1.x, 2.x, 3.x    |
| MCP               | core                   |
| Pennant           | core                   |
| Pest              | core, 3.x, 4.x         |
| PHPUnit           | core                   |
| Pint              | core                   |
| Sail              | core                   |
| Tailwind CSS      | core, 3.x, 4.x         |
| Livewire Volt     | core                   |
| Wayfinder         | core                   |
| Enforce Tests     | conditional            |

> **Note:** Để giữ AI guidelines của bạn cập nhật, xem phần [Giữ cập nhật Boost Resources](#keeping-boost-resources-updated).

<a name="adding-custom-ai-guidelines"></a>
### Thêm Custom AI Guidelines

Để mở rộng Laravel Boost với các AI guidelines tùy chỉnh của riêng bạn, thêm các file `.blade.php` hoặc `.md` vào thư mục `.ai/guidelines/*` của ứng dụng. Các file này sẽ tự động được bao gồm với các guidelines của Laravel Boost khi bạn chạy `boost:install`.

<a name="overriding-boost-ai-guidelines"></a>
### Ghi đè Boost AI Guidelines

Bạn có thể ghi đè các AI guidelines tích hợp của Boost bằng cách tạo các guidelines tùy chỉnh của riêng bạn với các đường dẫn file khớp. Khi bạn tạo một guideline tùy chỉnh khớp với đường dẫn guideline Boost hiện có, Boost sẽ sử dụng phiên bản tùy chỉnh của bạn thay vì phiên bản tích hợp.

Ví dụ, để ghi đè các guidelines "Inertia React v2 Form Guidance" của Boost, tạo một file tại `.ai/guidelines/inertia-react/2/forms.blade.php`. Khi bạn chạy `boost:install`, Boost sẽ bao gồm guideline tùy chỉnh của bạn thay vì mặc định.

<a name="third-party-package-ai-guidelines"></a>
### AI Guidelines cho Third-Party Package

Nếu bạn duy trì một third-party package và muốn Boost bao gồm AI guidelines cho nó, bạn có thể làm điều này bằng cách thêm một file `resources/boost/guidelines/core.blade.php` vào package của bạn. Khi người dùng của package chạy `php artisan boost:install`, Boost sẽ tự động tải các guidelines của bạn.

AI guidelines nên cung cấp một tổng quan ngắn gọn về những gì package của bạn làm, phác thảo bất kỳ cấu trúc file hoặc quy ước cần thiết, và giải thích cách tạo hoặc sử dụng các tính năng chính của nó (với các lệnh ví dụ hoặc code snippets). Giữ chúng ngắn gọn, có thể thực hiện, và tập trung vào best practices để AI có thể tạo code chính xác cho người dùng của bạn. Đây là một ví dụ:

```php
## Package Name

Package này cung cấp [mô tả ngắn gọn về chức năng].

### Features

- Feature 1: [mô tả ngắn gọn & rõ ràng].
- Feature 2: [mô tả ngắn gọn & rõ ràng]. Ví dụ sử dụng:

@verbatim
<code-snippet name="How to use Feature 2" lang="php">
$result = PackageName::featureTwo($param1, $param2);
</code-snippet>
@endverbatim
```

<a name="agent-skills"></a>
## Agent Skills

[Agent Skills](https://agentskills.io/home) là các module kiến thức nhẹ và tập trung mà agents có thể kích hoạt theo yêu cầu khi làm việc trên các lĩnh vực cụ thể. Khác với guidelines được tải lên trước, skills cho phép các pattern và best practices chi tiết chỉ được tải khi có liên quan, giảm bloat ngữ cảnh và cải thiện tính liên quan của code được tạo bởi AI.

Khi bạn chạy `boost:install` và chọn skills như một tính năng, skills được cài đặt tự động dựa trên các packages được phát hiện trong `composer.json` của bạn. Ví dụ, nếu dự án của bạn bao gồm `livewire/livewire`, skill `livewire-development` sẽ được cài đặt tự động.

<a name="available-skills"></a>
### Các Skills có sẵn

| Skill                      | Package        |
| -------------------------- | -------------- |
| fluxui-development         | Flux UI        |
| folio-routing              | Folio          |
| inertia-react-development  | Inertia React  |
| inertia-svelte-development | Inertia Svelte |
| inertia-vue-development    | Inertia Vue    |
| livewire-development       | Livewire       |
| mcp-development            | MCP            |
| pennant-development        | Pennant        |
| pest-testing               | Pest           |
| tailwindcss-development    | Tailwind CSS   |
| volt-development           | Volt           |
| wayfinder-development      | Wayfinder      |

> **Note:** Để giữ skills của bạn cập nhật, xem phần [Giữ cập nhật Boost Resources](#keeping-boost-resources-updated).

<a name="custom-skills"></a>
### Custom Skills

Để tạo các skills tùy chỉnh của riêng bạn, thêm một file `SKILL.md` vào thư mục `.ai/skills/{skill-name}/` của ứng dụng. Khi bạn chạy `boost:update`, các skills tùy chỉnh của bạn sẽ được cài đặt cùng với các skills tích hợp của Boost.

Ví dụ, để tạo một skill tùy chỉnh cho domain logic của ứng dụng:

```
.ai/skills/creating-invoices/SKILL.md
```

<a name="overriding-skills"></a>
### Ghi đè Skills

Bạn có thể ghi đè các skills tích hợp của Boost bằng cách tạo các skills tùy chỉnh của riêng bạn với các tên khớp. Khi bạn tạo một skill tùy chỉnh khớp với tên skill Boost hiện có, Boost sẽ sử dụng phiên bản tùy chỉnh của bạn thay vì phiên bản tích hợp.

Ví dụ, để ghi đè skill `livewire-development` của Boost, tạo một file tại `.ai/skills/livewire-development/SKILL.md`. Khi bạn chạy `boost:update`, Boost sẽ bao gồm skill tùy chỉnh của bạn thay vì mặc định.

<a name="third-party-package-skills"></a>
### Skills cho Third-Party Package

Nếu bạn duy trì một third-party package và muốn Boost bao gồm skills cho nó, bạn có thể làm điều này bằng cách thêm một file `resources/boost/skills/{skill-name}/SKILL.md` vào package của bạn. Khi người dùng của package chạy `php artisan boost:install`, Boost sẽ tự động cài đặt các skills của bạn dựa trên preference của người dùng.

Boost Skills hỗ trợ [Agent Skills format](https://agentskills.io/what-are-skills) và nên được cấu trúc như một thư mục chứa một file `SKILL.md` với YAML frontmatter và các hướng dẫn Markdown. File `SKILL.md` phải bao gồm frontmatter cần thiết (`name` và `description`) và có thể tùy ý bao gồm scripts, templates, và tài liệu tham khảo.

Skills nên phác thảo bất kỳ cấu trúc file hoặc quy ước cần thiết, và giải thích cách tạo hoặc sử dụng các tính năng chính của nó (với các lệnh ví dụ hoặc code snippets). Giữ chúng ngắn gọn, có thể thực hiện, và tập trung vào best practices để AI có thể tạo code chính xác cho người dùng của bạn:

```markdown
---
name: package-name-development
description: Build and work with PackageName features, including components and workflows.
---

# Package Name Development

## When to use this skill
Use this skill when working with PackageName features...

## Features

- Feature 1: [mô tả ngắn gọn & rõ ràng].
- Feature 2: [mô tả ngắn gọn & rõ ràng]. Ví dụ sử dụng:

$result = PackageName::featureTwo($param1, $param2);
```

<a name="guidelines-vs-skills"></a>
## Guidelines vs. Skills

Laravel Boost cung cấp hai cách riêng biệt để cung cấp cho AI agents ngữ cảnh về ứng dụng của bạn: **guidelines** và **skills**.

**Guidelines** được tải lên trước khi AI agent bắt đầu, cung cấp ngữ cảnh thiết yếu về các quy ước và best practices của Laravel áp dụng rộng rãi trên codebase của bạn.

**Skills** được kích hoạt theo yêu cầu khi làm việc trên các tasks cụ thể, chứa các pattern chi tiết cho các lĩnh vực cụ thể (như Livewire components hoặc Pest tests). Chỉ tải skills khi có liên quan giảm bloat ngữ cảnh và cải thiện chất lượng code.

| Aspect      | Guidelines                        | Skills                           |
| ----------- | --------------------------------- | -------------------------------- |
| **Loaded**  | Upfront, luôn hiện diện           | On-demand, khi có liên quan         |
| **Scope**   | Rộng, foundational               | Tập trung, task-specific           |
| **Purpose** | Quy ước cốt lõi & best practices | Pattern implementation chi tiết |

<a name="documentation-api"></a>
## Documentation API

Laravel Boost bao gồm một Documentation API cung cấp cho AI agents quyền truy cập vào một knowledge base rộng lớn chứa hơn 17,000 mảnh thông tin đặc thù về Laravel. API sử dụng tìm kiếm ngữ nghĩa với embeddings để cung cấp kết quả chính xác và có ngữ cảnh.

MCP tool `Search Docs` cho phép agents query service documentation API được host bởi Laravel để truy xuất documentation dựa trên các packages đã cài đặt của bạn. Các AI guidelines và skills của Boost sẽ tự động hướng dẫn coding agent của bạn sử dụng API này.

| Package           | Versions Supported |
| ----------------- | ------------------ |
| Laravel Framework | 10.x, 11.x, 12.x, 13.x |
| Filament          | 2.x, 3.x, 4.x, 5.x |
| Flux UI           | 2.x Free, 2.x Pro  |
| Inertia           | 1.x, 2.x           |
| Livewire          | 1.x, 2.x, 3.x, 4.x |
| Nova              | 4.x, 5.x           |
| Pest              | 3.x, 4.x           |
| Tailwind CSS      | 3.x, 4.x           |

<a name="extending-boost"></a>
## Mở rộng Boost

Boost hoạt động với nhiều IDEs và AI agents phổ biến ngay lập tức. Nếu công cụ code của bạn chưa được hỗ trợ, bạn có thể tạo agent riêng của mình và tích hợp nó với Boost.

<a name="adding-support-for-other-ides-ai-agents"></a>
### Thêm hỗ trợ cho các IDEs / AI Agents khác

Để thêm hỗ trợ cho một IDE hoặc AI agent mới, tạo một class extends `Laravel\Boost\Install\Agents\Agent` và implement một hoặc nhiều contracts sau tùy thuộc vào những gì bạn cần:

- `Laravel\Boost\Contracts\SupportsGuidelines` - Thêm hỗ trợ cho AI guidelines.
- `Laravel\Boost\Contracts\SupportsMcp` - Thêm hỗ trợ cho MCP.
- `Laravel\Boost\Contracts\SupportsSkills` - Thêm hỗ trợ cho Agent Skills.

<a name="writing-the-agent"></a>
#### Viết Agent

```php
<?php

declare(strict_types=1);

namespace App;

use Laravel\Boost\Contracts\SupportsGuidelines;
use Laravel\Boost\Contracts\SupportsMcp;
use Laravel\Boost\Contracts\SupportsSkills;
use Laravel\Boost\Install\Agents\Agent;

class CustomAgent extends Agent implements SupportsGuidelines, SupportsMcp, SupportsSkills
{
    // Implementation của bạn...
}
```

Để biết ví dụ implementation, xem [ClaudeCode.php](https://github.com/laravel/boost/blob/main/src/Install/Agents/ClaudeCode.php).

<a name="registering-the-agent"></a>
#### Đăng ký Agent

Đăng ký custom agent của bạn trong method `boot` của `App\Providers\AppServiceProvider` của ứng dụng:

```php
use Laravel\Boost\Boost;

public function boot(): void
{
    Boost::registerAgent('customagent', CustomAgent::class);
}
```

Sau khi đăng ký, agent của bạn sẽ có sẵn để chọn khi chạy `php artisan boost:install`.

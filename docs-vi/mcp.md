# Laravel MCP

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Publishing Routes](#publishing-routes)
- [Tạo Servers](#creating-servers)
    - [Server Registration](#server-registration)
    - [Web Servers](#web-servers)
    - [Local Servers](#local-servers)
- [Tools](#tools)
    - [Creating Tools](#creating-tools)
    - [Tool Input Schemas](#tool-input-schemas)
    - [Tool Output Schemas](#tool-output-schemas)
    - [Validating Tool Arguments](#validating-tool-arguments)
    - [Tool Dependency Injection](#tool-dependency-injection)
    - [Tool Annotations](#tool-annotations)
    - [Conditional Tool Registration](#conditional-tool-registration)
    - [Tool Responses](#tool-responses)
- [Prompts](#prompts)
    - [Creating Prompts](#creating-prompts)
    - [Prompt Arguments](#prompt-arguments)
    - [Validating Prompt Arguments](#validating-prompt-arguments)
    - [Prompt Dependency Injection](#prompt-dependency-injection)
    - [Conditional Prompt Registration](#conditional-prompt-registration)
    - [Prompt Responses](#prompt-responses)
- [Resources](#resources)
    - [Creating Resources](#creating-resources)
    - [Resource Templates](#resource-templates)
    - [Resource URI and MIME Type](#resource-uri-and-mime-type)
    - [Resource Request](#resource-request)
    - [Resource Dependency Injection](#resource-dependency-injection)
    - [Resource Annotations](#resource-annotations)
    - [Conditional Resource Registration](#conditional-resource-registration)
    - [Resource Responses](#resource-responses)
- [Apps](#apps)
    - [Creating App Resources](#creating-app-resources)
    - [Rendering Apps From Tools](#rendering-apps-from-tools)
    - [App Tool Visibility](#app-tool-visibility)
    - [App Configuration](#app-configuration)
    - [Building Apps With Boost](#building-apps-with-boost)
- [Metadata](#metadata)
- [Authentication](#authentication)
    - [OAuth 2.1](#oauth)
    - [Sanctum](#sanctum)
- [Authorization](#authorization)
- [MCP Client](#client)
    - [Connecting to Servers](#client-connecting)
    - [Named Clients](#named-clients)
    - [Client Authentication](#client-authentication)
    - [Tools](#client-tools)
- [Testing Servers](#testing-servers)
    - [MCP Inspector](#mcp-inspector)
    - [Unit Tests](#unit-tests)

<a name="introduction"></a>
## Giới thiệu

[Laravel MCP](https://github.com/laravel/mcp) cung cấp một cách đơn giản và thanh lịch để AI clients tương tác với ứng dụng Laravel của bạn thông qua [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro). Nó cung cấp một interface expressive và fluent để định nghĩa servers, tools, resources, và prompts cho phép các tương tác AI-powered với ứng dụng của bạn.

<a name="installation"></a>
## Cài đặt

Để bắt đầu, cài đặt Laravel MCP vào dự án của bạn sử dụng Composer package manager:

```shell
composer require laravel/mcp
```

<a name="publishing-routes"></a>
### Publishing Routes

Sau khi cài đặt Laravel MCP, thực thi lệnh Artisan `vendor:publish` để publish file `routes/ai.php` nơi bạn sẽ định nghĩa MCP servers của mình:

```shell
php artisan vendor:publish --tag=ai-routes
```

Lệnh này tạo file `routes/ai.php` trong thư mục `routes` của ứng dụng, mà bạn sẽ sử dụng để đăng ký MCP servers của mình.

<a name="creating-servers"></a>
## Tạo Servers

Bạn có thể tạo một MCP server sử dụng lệnh Artisan `make:mcp-server`. Servers đóng vai trò điểm giao tiếp trung tâm expose các MCP capabilities như tools, resources, và prompts đến AI clients:

```shell
php artisan make:mcp-server WeatherServer
```

Lệnh này sẽ tạo một server class mới trong thư mục `app/Mcp/Servers`. Server class được tạo extends class base `Laravel\Mcp\Server` của Laravel MCP và cung cấp các attributes và properties để cấu hình server và đăng ký tools, resources, và prompts:

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server\Attributes\Instructions;
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Version;
use Laravel\Mcp\Server;

#[Name('Weather Server')]
#[Version('1.0.0')]
#[Instructions('This server provides weather information and forecasts.')]
class WeatherServer extends Server
{
    /**
     * The tools registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Tool>>
     */
    protected array $tools = [
        // GetCurrentWeatherTool::class,
    ];

    /**
     * The resources registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Resource>>
     */
    protected array $resources = [
        // WeatherGuidelinesResource::class,
    ];

    /**
     * The prompts registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Prompt>>
     */
    protected array $prompts = [
        // DescribeWeatherPrompt::class,
    ];
}
```

<a name="server-registration"></a>
### Server Registration

Sau khi bạn đã tạo một server, bạn phải đăng ký nó trong file `routes/ai.php` của mình để làm cho nó có thể truy cập được. Laravel MCP cung cấp hai methods để đăng ký servers: `web` cho servers có thể truy cập qua HTTP và `local` cho servers command-line.

<a name="web-servers"></a>
### Web Servers

Web servers là các loại servers phổ biến nhất và có thể truy cập qua HTTP POST requests, làm cho chúng lý tưởng cho remote AI clients hoặc web-based integrations. Đăng ký một web server sử dụng method `web`:

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::web('/mcp/weather', WeatherServer::class);
```

Giống như các routes thông thường, bạn có thể áp dụng middleware để bảo vệ web servers của mình:

```php
Mcp::web('/mcp/weather', WeatherServer::class)
    ->middleware(['throttle:mcp']);
```

<a name="local-servers"></a>
### Local Servers

Local servers chạy như các Artisan commands, hoàn hảo để xây dựng các local AI assistant integrations như [Laravel Boost](/docs/{{version}}/installation#installing-laravel-boost). Đăng ký một local server sử dụng method `local`:

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::local('weather', WeatherServer::class);
```

Sau khi đăng ký, bạn thường không cần phải tự chạy lệnh Artisan `mcp:start`. Thay vào đó, cấu hình MCP client (AI agent) của bạn để bắt đầu server hoặc sử dụng [MCP Inspector](#mcp-inspector).

<a name="tools"></a>
## Tools

Tools cho phép server của bạn expose functionality mà AI clients có thể gọi. Chúng cho phép language models thực hiện actions, chạy code, hoặc tương tác với các external systems:

```php
<?php

namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Tool;

#[Description('Fetches the current weather forecast for a specified location.')]
class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request): Response
    {
        $location = $request->get('location');

        // Get weather...

        return Response::text('The weather is...');
    }

    /**
     * Get the tool's input schema.
     *
     * @return array<string, \Illuminate\JsonSchema\Types\Type>
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'location' => $schema->string()
                ->description('The location to get the weather for.')
                ->required(),
        ];
    }
}
```

<a name="creating-tools"></a>
### Creating Tools

Để tạo một tool, chạy lệnh Artisan `make:mcp-tool`:

```shell
php artisan make:mcp-tool CurrentWeatherTool
```

Sau khi tạo một tool, đăng ký nó trong property `$tools` của server:

```php
<?php

namespace App\Mcp\Servers;

use App\Mcp\Tools\CurrentWeatherTool;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * The tools registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Tool>>
     */
    protected array $tools = [
        CurrentWeatherTool::class,
    ];
}
```

<a name="tool-name-title-description"></a>
#### Tool Name, Title, and Description

Theo mặc định, name và title của tool được lấy từ class name. Ví dụ, `CurrentWeatherTool` sẽ có name là `current-weather` và title là `Current Weather Tool`. Bạn có thể tùy chỉnh các giá trị này sử dụng các attributes `Name` và `Title`:

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('get-optimistic-weather')]
#[Title('Get Optimistic Weather Forecast')]
class CurrentWeatherTool extends Tool
{
    // ...
}
```

Tool descriptions không được tự động tạo. Bạn nên luôn cung cấp một description có ý nghĩa sử dụng attribute `Description`:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Fetches the current weather forecast for a specified location.')]
class CurrentWeatherTool extends Tool
{
    //
}
```

> [!NOTE]
> Description là một phần quan trọng của metadata của tool, vì nó giúp AI models hiểu khi và cách sử dụng tool một cách hiệu quả.

<a name="tool-input-schemas"></a>
### Tool Input Schemas

Tools có thể định nghĩa input schemas để chỉ định các arguments mà chúng chấp nhận từ AI clients. Sử dụng builder `Illuminate\Contracts\JsonSchema\JsonSchema` của Laravel để định nghĩa các input requirements của tool:

```php
<?php

namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Get the tool's input schema.
     *
     * @return array<string, \Illuminate\JsonSchema\Types\Type>
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'location' => $schema->string()
                ->description('The location to get the weather for.')
                ->required(),

            'units' => $schema->string()
                ->enum(['celsius', 'fahrenheit'])
                ->description('The temperature units to use.')
                ->default('celsius'),
        ];
    }
}
```

<a name="tool-output-schemas"></a>
### Tool Output Schemas

Tools có thể định nghĩa [output schemas](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema) để chỉ định cấu trúc của responses của chúng. Điều này cho phép integration tốt hơn với AI clients cần các tool results có thể parse được. Sử dụng method `outputSchema` để định nghĩa cấu trúc output của tool:

```php
<?php

namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Get the tool's output schema.
     *
     * @return array<string, \Illuminate\JsonSchema\Types\Type>
     */
    public function outputSchema(JsonSchema $schema): array
    {
        return [
            'temperature' => $schema->number()
                ->description('Temperature in Celsius')
                ->required(),

            'conditions' => $schema->string()
                ->description('Weather conditions')
                ->required(),

            'humidity' => $schema->integer()
                ->description('Humidity percentage')
                ->required(),
        ];
    }
}
```

<a name="validating-tool-arguments"></a>
### Validating Tool Arguments

JSON Schema definitions cung cấp một cấu trúc cơ bản cho tool arguments, nhưng bạn cũng có thể muốn enforce các validation rules phức tạp hơn.

Laravel MCP tích hợp seamlessly với các [validation features](/docs/{{version}}/validation) của Laravel. Bạn có thể validate incoming tool arguments trong method `handle` của tool:

```php
<?php

namespace App\Mcp\Tools;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request): Response
    {
        $validated = $request->validate([
            'location' => 'required|string|max:100',
            'units' => 'in:celsius,fahrenheit',
        ]);

        // Fetch weather data using the validated arguments...
    }
}
```

Khi validation thất bại, AI clients sẽ hành động dựa trên các error messages bạn cung cấp. Do đó, điều quan trọng là phải cung cấp các error messages rõ ràng và có thể hành động:

```php
$validated = $request->validate([
    'location' => ['required','string','max:100'],
    'units' => 'in:celsius,fahrenheit',
],[
    'location.required' => 'You must specify a location to get the weather for. For example, "New York City" or "Tokyo".',
    'units.in' => 'You must specify either "celsius" or "fahrenheit" for the units.',
]);
```

<a name="tool-dependency-injection"></a>
#### Tool Dependency Injection

[Service container](/docs/{{version}}/container) của Laravel được sử dụng để giải quyết tất cả các tools. Kết quả là, bạn có thể type-hint bất kỳ dependencies nào mà tool của bạn có thể cần trong constructor của nó. Các dependencies được khai báo sẽ được tự động giải quyết và inject vào tool instance:

```php
<?php

namespace App\Mcp\Tools;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Create a new tool instance.
     */
    public function __construct(
        protected WeatherRepository $weather,
    ) {}

    // ...
}
```

Ngoài constructor injection, bạn cũng có thể type-hint dependencies trong method `handle()` của tool. Service container sẽ tự động giải quyết và inject các dependencies khi method được gọi:

```php
<?php

namespace App\Mcp\Tools;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request, WeatherRepository $weather): Response
    {
        $location = $request->get('location');

        $forecast = $weather->getForecastFor($location);

        // ...
    }
}
```

<a name="tool-annotations"></a>
### Tool Annotations

Bạn có thể enhance tools của mình với [annotations](https://modelcontextprotocol.io/specification/2025-06-18/schema#toolannotations) để cung cấp thêm metadata đến AI clients. Các annotations này giúp AI models hiểu behavior và capabilities của tool. Annotations được thêm vào tools thông qua attributes:

```php
<?php

namespace App\Mcp\Tools;

use Laravel\Mcp\Server\Tools\Annotations\IsIdempotent;
use Laravel\Mcp\Server\Tools\Annotations\IsReadOnly;
use Laravel\Mcp\Server\Tool;

#[IsIdempotent]
#[IsReadOnly]
class CurrentWeatherTool extends Tool
{
    //
}
```

Các annotations có sẵn bao gồm:

|| Annotation         | Type    | Description                                                                                  |
|| ------------------ | ------- | -------------------------------------------------------------------------------------------- |
|| `#[IsReadOnly]`    | boolean | Indicates the tool does not modify its environment.                                          |
|| `#[IsDestructive]` | boolean | Indicates the tool may perform destructive updates (only meaningful when not read-only).     |
|| `#[IsIdempotent]`  | boolean | Indicates repeated calls with same arguments have no additional effect (when not read-only). |
|| `#[IsOpenWorld]`   | boolean | Indicates the tool may interact with external entities.                                      |

Annotation values có thể được set một cách rõ ràng sử dụng boolean arguments:

```php
use Laravel\Mcp\Server\Tools\Annotations\IsReadOnly;
use Laravel\Mcp\Server\Tools\Annotations\IsDestructive;
use Laravel\Mcp\Server\Tools\Annotations\IsOpenWorld;
use Laravel\Mcp\Server\Tools\Annotations\IsIdempotent;
use Laravel\Mcp\Server\Tool;

#[IsReadOnly(true)]
#[IsDestructive(false)]
#[IsOpenWorld(false)]
#[IsIdempotent(true)]
class CurrentWeatherTool extends Tool
{
    //
}
```

<a name="conditional-tool-registration"></a>
### Conditional Tool Registration

Bạn có thể đăng ký tools có điều kiện tại runtime bằng cách implement method `shouldRegister` trong class tool của bạn. Method này cho phép bạn xác định xem một tool có nên có sẵn dựa trên application state, configuration, hoặc request parameters:

```php
<?php

namespace App\Mcp\Tools;

use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Determine if the tool should be registered.
     */
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

Khi method `shouldRegister` của một tool trả về `false`, nó sẽ không xuất hiện trong danh sách các tools có sẵn và không thể được gọi bởi AI clients.

<a name="tool-responses"></a>
### Tool Responses

Tools phải trả về một instance của `Laravel\Mcp\Response`. Class Response cung cấp một số methods tiện lợi để tạo các loại responses khác nhau:

Đối với text responses đơn giản, sử dụng method `text`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

/**
 * Handle the tool request.
 */
public function handle(Request $request): Response
{
    // ...

    return Response::text('Weather Summary: Sunny, 72°F');
}
```

Để chỉ định một error xảy ra trong quá trình thực thi tool, sử dụng method `error`:

```php
return Response::error('Unable to fetch weather data. Please try again.');
```

Để trả về image hoặc audio content, sử dụng các methods `image` và `audio`:

```php
return Response::image(file_get_contents(storage_path('weather/radar.png')), 'image/png');

return Response::audio(file_get_contents(storage_path('weather/alert.mp3')), 'audio/mp3');
```

Bạn cũng có thể load image và audio content trực tiếp từ một Laravel filesystem disk sử dụng method `fromStorage`. MIME type sẽ được tự động detect từ file:

```php
return Response::fromStorage('weather/radar.png');
```

Nếu cần thiết, bạn có thể chỉ định một disk cụ thể hoặc override MIME type:

```php
return Response::fromStorage('weather/radar.png', disk: 's3');

return Response::fromStorage('weather/radar.png', mimeType: 'image/webp');
```

<a name="multiple-content-responses"></a>
#### Multiple Content Responses

Tools có thể trả về nhiều pieces of content bằng cách trả về một array của các instances `Response`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

/**
 * Handle the tool request.
 *
 * @return array<int, \Laravel\Mcp\Response>
 */
public function handle(Request $request): array
{
    // ...

    return [
        Response::text('Weather Summary: Sunny, 72°F'),
        Response::text("**Detailed Forecast**\n- Morning: 65°F\n- Afternoon: 78°F\n- Evening: 70°F")
    ];
}
```

<a name="structured-responses"></a>
#### Structured Responses

Tools có thể trả về [structured content](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#structured-content) sử dụng method `structured`. Điều này cung cấp dữ liệu có thể parse được cho AI clients trong khi duy trì backward compatibility với một text representation được JSON-encoded:

```php
return Response::structured([
    'temperature' => 22.5,
    'conditions' => 'Partly cloudy',
    'humidity' => 65,
]);
```

Nếu bạn cần cung cấp custom text cùng với structured content, sử dụng method `withStructuredContent` trên response factory:

```php
return Response::make(
    Response::text('Weather is 22.5°C and sunny')
)->withStructuredContent([
    'temperature' => 22.5,
    'conditions' => 'Sunny',
]);
```

<a name="streaming-responses"></a>
#### Streaming Responses

Đối với các operations chạy lâu hoặc real-time data streaming, tools có thể trả về một [generator](https://www.php.net/manual/en/language.generators.overview.php) từ method `handle` của chúng. Điều này cho phép gửi intermediate updates đến client trước khi final response:

```php
<?php

namespace App\Mcp\Tools;

use Generator;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     *
     * @return \Generator<int, \Laravel\Mcp\Response>
     */
    public function handle(Request $request): Generator
    {
        $locations = $request->array('locations');

        foreach ($locations as $index => $location) {
            yield Response::notification('processing/progress', [
                'current' => $index + 1,
                'total' => count($locations),
                'location' => $location,
            ]);

            yield Response::text($this->forecastFor($location));
        }
    }
}
```

Khi sử dụng web-based servers, streaming responses tự động mở một SSE (Server-Sent Events) stream, gửi mỗi yielded message như một event đến client.

<a name="prompts"></a>
## Prompts

[Prompts](https://modelcontextprotocol.io/specification/2025-06-18/server/prompts) cho phép server của bạn chia sẻ reusable prompt templates mà AI clients có thể sử dụng để tương tác với language models. Chúng cung cấp một cách standardized để cấu trúc các common queries và interactions.

<a name="creating-prompts"></a>
### Creating Prompts

Để tạo một prompt, chạy lệnh Artisan `make:mcp-prompt`:

```shell
php artisan make:mcp-prompt DescribeWeatherPrompt
```

Sau khi tạo một prompt, đăng ký nó trong property `$prompts` của server:

```php
<?php

namespace App\Mcp\Servers;

use App\Mcp\Prompts\DescribeWeatherPrompt;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * The prompts registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Prompt>>
     */
    protected array $prompts = [
        DescribeWeatherPrompt::class,
    ];
}
```

<a name="prompt-name-title-and-description"></a>
#### Prompt Name, Title, and Description

Theo mặc định, name và title của prompt được lấy từ class name. Ví dụ, `DescribeWeatherPrompt` sẽ có name là `describe-weather` và title là `Describe Weather Prompt`. Bạn có thể tùy chỉnh các giá trị này sử dụng các attributes `Name` và `Title`:

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('weather-assistant')]
#[Title('Weather Assistant Prompt')]
class DescribeWeatherPrompt extends Prompt
{
    // ...
}
```

Prompt descriptions không được tự động tạo. Bạn nên luôn cung cấp một description có ý nghĩa sử dụng attribute `Description`:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Generates a natural-language explanation of the weather for a given location.')]
class DescribeWeatherPrompt extends Prompt
{
    //
}
```

> [!NOTE]
> Description là một phần quan trọng của metadata của prompt, vì nó giúp AI models hiểu khi và cách sử dụng prompt một cách hiệu quả.

<a name="prompt-arguments"></a>
### Prompt Arguments

Prompts có thể định nghĩa arguments cho phép AI clients tùy chỉnh prompt template với các giá trị cụ thể. Sử dụng method `arguments` để định nghĩa các arguments mà prompt của bạn chấp nhận:

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Server\Prompt;
use Laravel\Mcp\Server\Prompts\Argument;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Get the prompt's arguments.
     *
     * @return array<int, \Laravel\Mcp\Server\Prompts\Argument>
     */
    public function arguments(): array
    {
        return [
            new Argument(
                name: 'tone',
                description: 'The tone to use in the weather description (e.g., formal, casual, humorous).',
                required: true,
            ),
        ];
    }
}
```

<a name="validating-prompt-arguments"></a>
### Validating Prompt Arguments

Prompt arguments được tự động validate dựa trên định nghĩa của chúng, nhưng bạn cũng có thể muốn enforce các validation rules phức tạp hơn.

Laravel MCP tích hợp seamlessly với các [validation features](/docs/{{version}}/validation) của Laravel. Bạn có thể validate incoming prompt arguments trong method `handle` của prompt:

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Handle the prompt request.
     */
    public function handle(Request $request): Response
    {
        $validated = $request->validate([
            'tone' => 'required|string|max:50',
        ]);

        $tone = $validated['tone'];

        // Generate the prompt response using the given tone...
    }
}
```

Khi validation thất bại, AI clients sẽ hành động dựa trên các error messages bạn cung cấp. Do đó, điều quan trọng là phải cung cấp các error messages rõ ràng và có thể hành động:

```php
$validated = $request->validate([
    'tone' => ['required','string','max:50'],
],[
    'tone.*' => 'You must specify a tone for the weather description. Examples include "formal", "casual", or "humorous".',
]);
```

<a name="prompt-dependency-injection"></a>
### Prompt Dependency Injection

[Service container](/docs/{{version}}/container) của Laravel được sử dụng để giải quyết tất cả các prompts. Kết quả là, bạn có thể type-hint bất kỳ dependencies nào mà prompt của bạn có thể cần trong constructor của nó. Các dependencies được khai báo sẽ được tự động giải quyết và inject vào prompt instance:

```php
<?php

namespace App\Mcp\Prompts;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Create a new prompt instance.
     */
    public function __construct(
        protected WeatherRepository $weather,
    ) {}

    //
}
```

Ngoài constructor injection, bạn cũng có thể type-hint dependencies trong method `handle` của prompt. Service container sẽ tự động giải quyết và inject các dependencies khi method được gọi:

```php
<?php

namespace App\Mcp\Prompts;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Handle the prompt request.
     */
    public function handle(Request $request, WeatherRepository $weather): Response
    {
        $isAvailable = $weather->isServiceAvailable();

        // ...
    }
}
```

<a name="conditional-prompt-registration"></a>
### Conditional Prompt Registration

Bạn có thể đăng ký prompts có điều kiện tại runtime bằng cách implement method `shouldRegister` trong class prompt của bạn. Method này cho phép bạn xác định xem một prompt có nên có sẵn dựa trên application state, configuration, hoặc request parameters:

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Prompt;

class CurrentWeatherPrompt extends Prompt
{
    /**
     * Determine if the prompt should be registered.
     */
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

Khi method `shouldRegister` của một prompt trả về `false`, nó sẽ không xuất hiện trong danh sách các prompts có sẵn và không thể được gọi bởi AI clients.

<a name="prompt-responses"></a>
### Prompt Responses

Prompts có thể trả về một `Laravel\Mcp\Response` đơn hoặc một iterable của các instances `Laravel\Mcp\Response`. Các responses này encapsulate content sẽ được gửi đến AI client:

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Handle the prompt request.
     *
     * @return array<int, \Laravel\Mcp\Response>
     */
    public function handle(Request $request): array
    {
        $tone = $request->string('tone');

        $systemMessage = "You are a helpful weather assistant. Please provide a weather description in a {$tone} tone.";

        $userMessage = "What is the current weather like in New York City?";

        return [
            Response::text($systemMessage)->asAssistant(),
            Response::text($userMessage),
        ];
    }
}
```

Bạn có thể sử dụng method `asAssistant()` để chỉ định rằng một response message nên được xử lý như đến từ AI assistant, trong khi regular messages được xử lý như user input.

<a name="resources"></a>
## Resources

[Resources](https://modelcontextprotocol.io/specification/2025-06-18/server/resources) cho phép server của bạn expose data và content mà AI clients có thể đọc và sử dụng như context khi tương tác với language models. Chúng cung cấp một cách để chia sẻ static hoặc dynamic information như documentation, configuration, hoặc bất kỳ data nào giúp inform AI responses.

<a name="creating-resources"></a>
### Creating Resources

Để tạo một resource, chạy lệnh Artisan `make:mcp-resource`:

```shell
php artisan make:mcp-resource WeatherGuidelinesResource
```

Sau khi tạo một resource, đăng ký nó trong property `$resources` của server:

```php
<?php

namespace App\Mcp\Servers;

use App\Mcp\Resources\WeatherGuidelinesResource;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * The resources registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Resource>>
     */
    protected array $resources = [
        WeatherGuidelinesResource::class,
    ];
}
```

<a name="resource-name-title-and-description"></a>
#### Resource Name, Title, and Description

Theo mặc định, name và title của resource được lấy từ class name. Ví dụ, `WeatherGuidelinesResource` sẽ có name là `weather-guidelines` và title là `Weather Guidelines Resource`. Bạn có thể tùy chỉnh các giá trị này sử dụng các attributes `Name` và `Title`:

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('weather-api-docs')]
#[Title('Weather API Documentation')]
class WeatherGuidelinesResource extends Resource
{
    // ...
}
```

Resource descriptions không được tự động tạo. Bạn nên luôn cung cấp một description có ý nghĩa sử dụng attribute `Description`:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Comprehensive guidelines for using the Weather API.')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

> [!NOTE]
> Description là một phần quan trọng của metadata của resource, vì nó giúp AI models hiểu khi và cách sử dụng resource một cách hiệu quả.

<a name="resource-templates"></a>
### Resource Templates

[Resource templates](https://modelcontextprotocol.io/specification/2025-06-18/server/resources#resource-templates) cho phép server của bạn expose dynamic resources match với URI patterns có variables. Thay vì định nghĩa một static URI cho mỗi resource, bạn có thể tạo một single resource xử lý nhiều URIs dựa trên một template pattern.

<a name="creating-resource-templates"></a>
#### Creating Resource Templates

Để tạo một resource template, implement interface `HasUriTemplate` trên class resource của bạn và định nghĩa một method `uriTemplate` trả về một instance `UriTemplate`:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Contracts\HasUriTemplate;
use Laravel\Mcp\Server\Resource;
use Laravel\Mcp\Support\UriTemplate;

#[Description('Access user files by ID')]
#[MimeType('text/plain')]
class UserFileResource extends Resource implements HasUriTemplate
{
    /**
     * Get the URI template for this resource.
     */
    public function uriTemplate(): UriTemplate
    {
        return new UriTemplate('file://users/{userId}/files/{fileId}');
    }

    /**
     * Handle the resource request.
     */
    public function handle(Request $request): Response
    {
        $userId = $request->get('userId');
        $fileId = $request->get('fileId');

        // Fetch and return the file content...

        return Response::text($content);
    }
}
```

Khi một resource implement interface `HasUriTemplate`, nó sẽ được đăng ký như một resource template thay vì một static resource. AI clients có thể request resources sử dụng URIs match với template pattern, và các variables từ URI sẽ được tự động extract và có sẵn trong method `handle` của resource.

<a name="uri-template-syntax"></a>
#### URI Template Syntax

URI templates sử dụng placeholders enclosed trong curly braces để định nghĩa variable segments trong URI:

```php
new UriTemplate('file://users/{userId}');
new UriTemplate('file://users/{userId}/files/{fileId}');
new UriTemplate('https://api.example.com/{version}/{resource}/{id}');
```

<a name="accessing-template-variables"></a>
#### Accessing Template Variables

Khi một URI match với resource template của bạn, các extracted variables được tự động merge vào request và có thể được truy cập sử dụng method `get`:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Contracts\HasUriTemplate;
use Laravel\Mcp\Server\Resource;
use Laravel\Mcp\Support\UriTemplate;

class UserProfileResource extends Resource implements HasUriTemplate
{
    public function uriTemplate(): UriTemplate
    {
        return new UriTemplate('file://users/{userId}/profile');
    }

    public function handle(Request $request): Response
    {
        // Access the extracted variable
        $userId = $request->get('userId');

        // Access the full URI if needed
        $uri = $request->uri();

        // Fetch user profile...

        return Response::text("Profile for user {$userId}");
    }
}
```

Object `Request` cung cấp cả extracted variables và original URI được request, cho phép bạn có full context để xử lý resource request.

<a name="resource-uri-and-mime-type"></a>
### Resource URI and MIME Type

Mỗi resource được xác định bởi một URI duy nhất và có một MIME type liên kết giúp AI clients hiểu format của resource.

Theo mặc định, URI của resource được tạo dựa trên name của resource, nên `WeatherGuidelinesResource` sẽ có URI là `weather://resources/weather-guidelines`. MIME type mặc định là `text/plain`.

Bạn có thể tùy chỉnh các giá trị này sử dụng các attributes `Uri` và `MimeType`:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Attributes\Uri;
use Laravel\Mcp\Server\Resource;

#[Uri('weather://resources/guidelines')]
#[MimeType('application/pdf')]
class WeatherGuidelinesResource extends Resource
{
}
```

URI và MIME type giúp AI clients xác định cách process và interpret resource content một cách phù hợp.

<a name="resource-request"></a>
### Resource Request

Khác với tools và prompts, resources không thể định nghĩa input schemas hoặc arguments. Tuy nhiên, bạn vẫn có thể tương tác với request object trong method `handle` của resource:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Handle the resource request.
     */
    public function handle(Request $request): Response
    {
        // ...
    }
}
```

<a name="resource-dependency-injection"></a>
### Resource Dependency Injection

[Service container](/docs/{{version}}/container) của Laravel được sử dụng để giải quyết tất cả các resources. Kết quả là, bạn có thể type-hint bất kỳ dependencies nào mà resource của bạn có thể cần trong constructor của nó. Các dependencies được khai báo sẽ được tự động giải quyết và inject vào resource instance:

```php
<?php

namespace App\Mcp\Resources;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Create a new resource instance.
     */
    public function __construct(
        protected WeatherRepository $weather,
    ) {}

    // ...
}
```

Ngoài constructor injection, bạn cũng có thể type-hint dependencies trong method `handle` của resource. Service container sẽ tự động giải quyết và inject các dependencies khi method được gọi:

```php
<?php

namespace App\Mcp\Resources;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Handle the resource request.
     */
    public function handle(WeatherRepository $weather): Response
    {
        $guidelines = $weather->guidelines();

        return Response::text($guidelines);
    }
}
```

<a name="resource-annotations"></a>
### Resource Annotations

Bạn có thể enhance resources của mình với [annotations](https://modelcontextprotocol.io/specification/2025-06-18/schema#resourceannotations) để cung cấp thêm metadata đến AI clients. Annotations được thêm vào resources thông qua attributes:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Enums\Role;
use Laravel\Mcp\Server\Annotations\Audience;
use Laravel\Mcp\Server\Annotations\LastModified;
use Laravel\Mcp\Server\Annotations\Priority;
use Laravel\Mcp\Server\Resource;

#[Audience(Role::User)]
#[LastModified('2025-01-12T15:00:58Z')]
#[Priority(0.9)]
class UserDashboardResource extends Resource
{
    //
}
```

Các annotations có sẵn bao gồm:

|| Annotation        | Type          | Description                                                                 |
|| ----------------- | ------------- | --------------------------------------------------------------------------- |
|| `#[Audience]`     | Role or array | Specifies the intended audience (`Role::User`, `Role::Assistant`, or both). |
|| `#[Priority]`     | float         | A numerical score between 0.0 and 1.0 indicating resource importance.       |
|| `#[LastModified]` | string        | An ISO 8601 timestamp showing when the resource was last updated.           |

<a name="conditional-resource-registration"></a>
### Conditional Resource Registration

Bạn có thể đăng ký resources có điều kiện tại runtime bằng cách implement method `shouldRegister` trong class resource của bạn. Method này cho phép bạn xác định xem một resource có nên có sẵn dựa trên application state, configuration, hoặc request parameters:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Determine if the resource should be registered.
     */
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

Khi method `shouldRegister` của một resource trả về `false`, nó sẽ không xuất hiện trong danh sách các resources có sẵn và không thể được truy cập bởi AI clients.

<a name="resource-responses"></a>
### Resource Responses

Resources phải trả về một instance của `Laravel\Mcp\Response`. Class Response cung cấp một số methods tiện lợi để tạo các loại responses khác nhau:

Đối với text content đơn giản, sử dụng method `text`:

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

/**
 * Handle the resource request.
 */
public function handle(Request $request): Response
{
    // ...

    return Response::text($weatherData);
}
```

<a name="resource-link-responses"></a>
#### Resource Link Responses

Để trả về một resource link, sử dụng method `resourceLink`, cung cấp URI và name. Khác với một embedded resource, một resource link trả về một URI pointer mà AI client fetch độc lập:

```php
return Response::resourceLink(
    uri: 'file:///data/report.json',
    name: 'monthly-report',
    mimeType: 'application/json',
);
```

Bạn cũng có thể truyền một registered resource class hoặc instance, sẽ tự động inherit URI, name, title, description, và MIME type của resource:

```php
return Response::resourceLink(new WeatherForecastResource);
```

<a name="resource-blob-responses"></a>
#### Blob Responses

Để trả về blob content, sử dụng method `blob`, cung cấp blob content:

```php
return Response::blob(file_get_contents(storage_path('weather/radar.png')));
```

Khi trả về blob content, MIME type sẽ được xác định bởi MIME type được cấu hình của resource:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Resource;

#[MimeType('image/png')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

<a name="resource-error-responses"></a>
#### Error Responses

Để chỉ định một error xảy ra trong quá trình resource retrieval, sử dụng method `error()`:

```php
return Response::error('Unable to fetch weather data for the specified location.');
```

<a name="apps"></a>
## Apps

Laravel MCP hỗ trợ [MCP Apps](https://modelcontextprotocol.io/extensions/apps/overview), một extension của Model Context Protocol cho phép tools render interactive HTML applications trong sandboxed iframes trong supported hosts. Điều này cho phép bạn xây dựng dashboards, forms, visualizations, và các rich experiences khác vượt qua plain text responses.

Một MCP app bao gồm hai phần hoạt động cùng nhau:

- Một **app resource** trả về self-contained HTML cho application của bạn.
- Một **tool** được link đến app resource sử dụng attribute `#[RendersApp]`. Khi tool được gọi, host fetch và render linked resource.

<a name="creating-app-resources"></a>
### Creating App Resources

Bạn có thể tạo một app resource sử dụng lệnh Artisan `make:mcp-app-resource`:

```shell
php artisan make:mcp-app-resource WeatherDashboardApp
```

Lệnh này tạo hai files: một PHP class trong `app/Mcp/Resources` và một Blade view trong `resources/views/mcp`. View name được tự động infer từ class name. Ví dụ, `WeatherDashboardApp` maps đến `mcp.weather-dashboard-app`:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\AppMeta;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\AppResource;

#[Description('An interactive weather dashboard.')]
#[AppMeta]
class WeatherDashboardApp extends AppResource
{
    /**
     * Handle the app resource request.
     */
    public function handle(Request $request): Response
    {
        return Response::view('mcp.weather-dashboard-app', [
            'title' => $this->title(),
        ]);
    }
}
```

`AppResource` extends base class `Resource` và tự động cấu hình URI scheme `ui://` và MIME type `text/html;profile=mcp-app` được yêu cầu bởi MCP Apps specification. Giống như bất kỳ resource nào khác, bạn phải đăng ký nó trong array `$resources` của server.

Blade view được tạo sử dụng component `<x-mcp::app>`, render một complete HTML document với client-side MCP SDK bundled và sẵn sàng sử dụng:

```blade
<x-mcp::app :title="$title">
    <x-slot:head>
        <script type="module">
        createMcpApp(async (app) => {
            document.getElementById('run-btn').addEventListener('click', async () => {
                const result = await app.callServerTool('get-weather-data', {});
                document.getElementById('output').textContent = result.content[0]?.text ?? '';
            });
        });
        </script>
    </x-slot:head>

    <div id="app">
        <button id="run-btn">Refresh</button>
        <p id="output"></p>
    </div>
</x-mcp::app>
```

Global `createMcpApp` được cung cấp bởi bundled SDK và xử lý việc kết nối iframe đến server, áp dụng host theming, và expose helpers như `callServerTool`, `sendMessage`, `openLink`, và event callbacks. Đối với full client-side API, tham khảo [MCP Apps specification](https://modelcontextprotocol.io/extensions/apps/overview).

<a name="rendering-apps-from-tools"></a>
### Rendering Apps From Tools

Để display một app resource, link một tool đến nó sử dụng attribute `#[RendersApp]`. Khi tool được gọi, Laravel MCP bao gồm URI của resource trong tool metadata để host có thể render app trong một sandboxed iframe:

```php
<?php

namespace App\Mcp\Tools;

use App\Mcp\Resources\WeatherDashboardApp;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\RendersApp;
use Laravel\Mcp\Server\Tool;

#[RendersApp(resource: WeatherDashboardApp::class)]
class ShowWeatherDashboard extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request): Response
    {
        return Response::text('Weather dashboard loaded.');
    }
}
```

Laravel MCP tự động advertise capability `io.modelcontextprotocol/ui` bất cứ khi nào bất kỳ `AppResource` nào được đăng ký, nên không cần additional server configuration.

<a name="app-tool-visibility"></a>
### App Tool Visibility

Mỗi tool `#[RendersApp]` có thể limit ai có thể invoke nó thông qua argument `visibility`. Điều này hữu ích để expose private, app-only tools mà UI gọi để load hoặc refresh data mà không làm cho những tools đó visible đến model:

```php
use Laravel\Mcp\Server\Attributes\RendersApp;
use Laravel\Mcp\Server\Ui\Enums\Visibility;

#[RendersApp(resource: WeatherDashboardApp::class, visibility: [Visibility::App])]
class GetWeatherData extends Tool
{
    // ...
}
```

Enum `Visibility` có hai cases, `Model` và `App`, và mặc định là cả hai. Sử dụng `[Visibility::App]` cho backend actions mà UI gọi trực tiếp, hoặc `[Visibility::Model]` để làm cho một tool unavailable đến UI.

<a name="app-configuration"></a>
### App Configuration

Attribute `#[AppMeta]` trên app resource của bạn cấu hình Content Security Policy của iframe, browser permissions, và bất kỳ library scripts nào nên được include trong `<head>` của view:

```php
use Laravel\Mcp\Server\Attributes\AppMeta;
use Laravel\Mcp\Server\Ui\Enums\Library;
use Laravel\Mcp\Server\Ui\Enums\Permission;

#[AppMeta(
    connectDomains: ['https://api.weather.com'],
    permissions: [Permission::Geolocation],
    libraries: [Library::Tailwind, Library::Alpine],
)]
class WeatherDashboardApp extends AppResource
{
    // ...
}
```

Enum `Library` bao gồm pre-configured CDN scripts cho các common front-end libraries, như `Library::Tailwind` và `Library::Alpine`, và CDN origins của chúng được tự động merge vào CSP. Enum `Permission` bao gồm browser permissions như `Camera`, `Microphone`, `Geolocation`, và `ClipboardWrite`.

Đối với computed hoặc dynamic configuration, override method `appMeta` trên resource của bạn sử dụng fluent builders `AppMeta`, `Csp`, và `Permissions` từ namespace `Laravel\Mcp\Server\Ui`.

<a name="building-apps-with-boost"></a>
### Building Apps With Boost

[Laravel Boost](/docs/{{version}}/installation#installing-laravel-boost) tích hợp sâu với MCP Apps, cho phép bạn xây dựng rich, interactive experiences cho local AI assistant của bạn. Boost tự động khám phá và render các MCP Apps được đăng ký trong application của bạn, cung cấp một seamless workflow để xây dựng và test apps của bạn.

Để bắt đầu xây dựng apps với Boost, đảm bảo bạn đã cài đặt Laravel Boost và đăng ký một local MCP server với các app resources của bạn. Boost sẽ tự động phát hiện và render các apps trong giao diện assistant của nó.

<a name="metadata"></a>
## Metadata

Bạn có thể cung cấp metadata bổ sung cho MCP server của bạn bằng cách implement interface `HasMetadata` trong class server của bạn:

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server\Contracts\HasMetadata;
use Laravel\Mcp\Server;

class WeatherServer extends Server implements HasMetadata
{
    /**
     * Get the server's metadata.
     */
    public function metadata(): array
    {
        return [
            'version' => '1.0.0',
            'author' => 'Your Name',
            'license' => 'MIT',
        ];
    }
}
```

Metadata này có thể được sử dụng bởi AI clients để hiểu thêm về server và capabilities của nó.

<a name="authentication"></a>
## Authentication

<a name="oauth"></a>
### OAuth 2.1

Laravel MCP hỗ trợ [OAuth 2.1](/docs/{{version}}/sanctum#oauth) authentication thông qua Laravel Sanctum. Để kích hoạt OAuth authentication cho MCP server của bạn, bạn cần:

1. Cài đặt và cấu hình Laravel Sanctum trong application của bạn.
2. Thêm middleware `auth:sanctum` vào web server của bạn.

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::web('/mcp/weather', WeatherServer::class)
    ->middleware(['auth:sanctum']);
```

Sau khi cấu hình, AI clients sẽ cần cung cấp một valid access token trong request headers của họ để truy cập server.

<a name="sanctum"></a>
### Sanctum

Ngoài OAuth, Laravel MCP cũng hỗ trợ [Laravel Sanctum](/docs/{{version}}/sanctum) token authentication. Điều này hữu ích cho các scenarios nơi bạn muốn sử dụng simple API tokens thay vì full OAuth flow:

```php
Mcp::web('/mcp/weather', WeatherServer::class)
    ->middleware(['auth:sanctum']);
```

<a name="authorization"></a>
## Authorization

Laravel MCP tích hợp với [authorization system](/docs/{{version}}/authorization) của Laravel, cho phép bạn kiểm soát ai có thể truy cập tools, resources, và prompts của bạn. Bạn có thể sử dụng gates và policies để enforce authorization rules:

```php
<?php

namespace App\Mcp\Tools;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request): Response
    {
        $this->authorize('view-weather', $request->user());

        // ...
    }
}
```

Bạn cũng có thể sử dụng middleware để enforce authorization:

```php
Mcp::web('/mcp/weather', WeatherServer::class)
    ->middleware(['auth:sanctum', 'can:view-weather']);
```

<a name="client"></a>
## MCP Client

Laravel MCP cũng cung cấp một client cho phép ứng dụng Laravel của bạn kết nối đến các MCP servers external. Điều này hữu ích khi bạn muốn consume tools, resources, và prompts từ các servers khác.

<a name="client-connecting"></a>
### Connecting to Servers

Để kết nối đến một MCP server, sử dụng class `Laravel\Mcp\Client`:

```php
use Laravel\Mcp\Client;

$client = Client::web('https://mcp.example.com');
```

Đối với local servers, sử dụng method `local`:

```php
$client = Client::local('php', ['artisan', 'mcp:start']);
```

<a name="named-clients"></a>
### Named Clients

Bạn có thể định nghĩa named clients trong file cấu hình `config/mcp.php` của bạn:

```php
'clients' => [
    'github' => [
        'url' => 'https://mcp.github.com',
        'token' => env('GITHUB_MCP_TOKEN'),
    ],
],
```

Sau đó, bạn có thể truy xuất client sử dụng facade:

```php
use Laravel\Mcp\Facades\Mcp;

$client = Mcp::client('github');
```

<a name="client-authentication"></a>
### Client Authentication

Để authenticate với một MCP server, bạn có thể sử dụng bearer tokens hoặc OAuth:

```php
$client = Client::web('https://mcp.example.com')
    ->withToken('your-api-token');
```

Đối với OAuth, sử dụng method `withOAuth`:

```php
$client = Client::web('https://mcp.example.com')
    ->withOAuth('client-id', 'client-secret', 'redirect-uri');
```

<a name="client-tools"></a>
### Tools

Sau khi kết nối đến một server, bạn có thể truy xuất các tools có sẵn:

```php
$tools = $client->tools();

foreach ($tools as $tool) {
    echo $tool->name;
}
```

Để gọi một tool, sử dụng method `callTool`:

```php
$response = $client->callTool('tool-name', [
    'argument' => 'value',
]);
```

<a name="testing-servers"></a>
## Testing Servers

<a name="mcp-inspector"></a>
### MCP Inspector

Laravel MCP bao gồm một MCP Inspector cho phép bạn test MCP servers của bạn một cách tương tác. Để bắt đầu inspector, chạy lệnh Artisan `mcp:inspect`:

```shell
php artisan mcp:inspect
```

Lệnh này sẽ khởi động một interactive session nơi bạn có thể khám phá tools, resources, và prompts của server của bạn.

<a name="unit-tests"></a>
### Unit Tests

Bạn có thể test MCP servers của bạn sử dụng Laravel's testing features. Để fake MCP server responses, sử dụng method `fake`:

```php
use Laravel\Mcp\Facades\Mcp;

Mcp::fake([
    'tool-name' => 'mocked response',
]);
```

Sau đó, bạn có thể thực hiện assertions về các requests đã được thực hiện:

```php
Mcp::assertToolCalled('tool-name');
Mcp::assertToolCalledWith('tool-name', ['argument' => 'value']);
```

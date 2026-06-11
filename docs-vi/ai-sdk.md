# Laravel AI SDK

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
    - [Cấu hình](#configuration)
    - [Custom Base URLs](#custom-base-urls)
    - [Hỗ trợ Provider](#provider-support)
- [Agents](#agents)
    - [Prompting](#prompting)
    - [Ngữ cảnh hội thoại](#conversation-context)
    - [Structured Output](#structured-output)
    - [Attachments](#attachments)
    - [Streaming](#streaming)
    - [Broadcasting](#broadcasting)
    - [Queueing](#queueing)
    - [Tools](#tools)
    - [MCP Tools](#mcp-tools)
    - [Provider Tools](#provider-tools)
    - [Sub-Agents](#sub-agents)
    - [Middleware](#middleware)
    - [Anonymous Agents](#anonymous-agents)
    - [Cấu hình Agent](#agent-configuration)
    - [Tùy chọn Provider](#provider-options)
- [Images](#images)
- [Audio (TTS)](#audio)
- [Transcription (STT)](#transcription)
- [Embeddings](#embeddings)
    - [Querying Embeddings](#querying-embeddings)
    - [Caching Embeddings](#caching-embeddings)
- [Reranking](#reranking)
- [Files](#files)
- [Vector Stores](#vector-stores)
    - [Adding Files to Stores](#adding-files-to-stores)
- [Failover](#failover)
- [Testing](#testing)
    - [Agents](#testing-agents)
    - [Images](#testing-images)
    - [Audio](#testing-audio)
    - [Transcriptions](#testing-transcriptions)
    - [Embeddings](#testing-embeddings)
    - [Reranking](#testing-reranking)
    - [Files](#testing-files)
    - [Vector Stores](#testing-vector-stores)
- [Events](#events)

<a name="introduction"></a>
## Giới thiệu

[Laravel AI SDK](https://github.com/laravel/ai) cung cấp một API thống nhất và rõ ràng để tương tác với các AI providers như OpenAI, Anthropic, Gemini, và nhiều hơn nữa. Với AI SDK, bạn có thể xây dựng các intelligent agents với tools và structured output, tạo images, tổng hợp và transcribe audio, tạo vector embeddings, và nhiều hơn nữa — tất cả sử dụng một interface nhất quán và thân thiện với Laravel.

<a name="installation"></a>
## Cài đặt

Bạn có thể cài đặt Laravel AI SDK thông qua Composer:

```shell
composer require laravel/ai
```

Tiếp theo, bạn nên publish file cấu hình và migration của AI SDK sử dụng lệnh Artisan `vendor:publish`:

```shell
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
```

Cuối cùng, bạn nên chạy database migrations của ứng dụng. Điều này sẽ tạo ra các bảng `agent_conversations` và `agent_conversation_messages` mà AI SDK sử dụng để hỗ trợ lưu trữ hội thoại:

```shell
php artisan migrate
```

<a name="configuration"></a>
### Cấu hình

Bạn có thể định nghĩa credentials của AI provider trong file cấu hình `config/ai.php` của ứng dụng hoặc dưới dạng biến môi trường trong file `.env` của ứng dụng:

```ini
ANTHROPIC_API_KEY=
AZURE_OPENAI_API_KEY=
COHERE_API_KEY=
DEEPSEEK_API_KEY=
ELEVENLABS_API_KEY=
GEMINI_API_KEY=
GROQ_API_KEY=
MISTRAL_API_KEY=
OLLAMA_API_KEY=
OPENAI_API_KEY=
OPENROUTER_API_KEY=
JINA_API_KEY=
VOYAGEAI_API_KEY=
XAI_API_KEY=
```

Các models mặc định được sử dụng cho text, images, audio, transcription, và embeddings cũng có thể được cấu hình trong file cấu hình `config/ai.php` của ứng dụng.

<a name="custom-base-urls"></a>
### Custom Base URLs

Theo mặc định, Laravel AI SDK kết nối trực tiếp với API endpoint công khai của từng provider. Tuy nhiên, bạn có thể cần route các requests qua một endpoint khác — ví dụ, khi sử dụng proxy service để tập trung quản lý API key, implement rate limiting, hoặc route traffic qua một corporate gateway.

Bạn có thể cấu hình custom base URLs bằng cách thêm tham số `url` vào cấu hình provider của bạn:

```php
'providers' => [
    'openai' => [
        'driver' => 'openai',
        'key' => env('OPENAI_API_KEY'),
        'url' => env('OPENAI_BASE_URL'),
    ],

    'anthropic' => [
        'driver' => 'anthropic',
        'key' => env('ANTHROPIC_API_KEY'),
        'url' => env('ANTHROPIC_BASE_URL'),
    ],
],
```

Điều này hữu ích khi route các requests qua proxy service (như LiteLLM hoặc Azure OpenAI Gateway) hoặc sử dụng alternative endpoints.

Custom base URLs được hỗ trợ cho các providers sau: OpenAI, Anthropic, Gemini, Groq, Cohere, DeepSeek, xAI, và OpenRouter.

<a name="provider-support"></a>
### Hỗ trợ Provider

AI SDK hỗ trợ nhiều providers khác nhau trên các tính năng của nó. Bảng sau tóm tắt các providers có sẵn cho từng tính năng:

| Tính năng | Providers |
|---|---|
| Text | OpenAI, Anthropic, Gemini, Azure, Bedrock, Groq, xAI, DeepSeek, Mistral, Ollama, OpenRouter |
| Images | OpenAI, Gemini, xAI, Azure, Bedrock, OpenRouter |
| TTS | OpenAI, ElevenLabs, Gemini |
| STT | OpenAI, ElevenLabs, Mistral, Gemini |
| Embeddings | OpenAI, Gemini, Azure, Bedrock, Cohere, Mistral, Jina, VoyageAI, Ollama, OpenRouter |
| Reranking | Cohere, Jina, VoyageAI |
| Files | OpenAI, Anthropic, Gemini |

Enum `Laravel\Ai\Enums\Lab` có thể được sử dụng để tham chiếu providers trong code của bạn thay vì sử dụng plain strings:

```php
use Laravel\Ai\Enums\Lab;

Lab::Anthropic;
Lab::OpenAI;
Lab::Gemini;
// ...
```

<a name="agents"></a>
## Agents

Agents là building block cơ bản để tương tác với AI providers trong Laravel AI SDK. Mỗi agent là một class PHP chuyên biệt đóng gói các instructions, ngữ cảnh hội thoại, tools, và output schema cần thiết để tương tác với một large language model. Hãy coi agent như một trợ lý chuyên biệt — một sales coach, một document analyzer, một support bot — mà bạn cấu hình một lần và prompt khi cần thiết trong ứng dụng của bạn.

Bạn có thể tạo một agent thông qua lệnh Artisan `make:agent`:

```shell
php artisan make:agent SalesCoach

php artisan make:agent SalesCoach --structured
```

Trong class agent được tạo, bạn có thể định nghĩa system prompt / instructions, ngữ cảnh message, các tools có sẵn, và output schema (nếu có):

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\RetrievePreviousTranscripts;
use App\Models\History;
use App\Models\User;
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Messages\Message;
use Laravel\Ai\Promptable;
use Stringable;

class SalesCoach implements Agent, Conversational, HasTools, HasStructuredOutput
{
    use Promptable;

    public function __construct(public User $user) {}

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): Stringable|string
    {
        return 'You are a sales coach, analyzing transcripts and providing feedback and an overall sales strength score.';
    }

    /**
     * Get the list of messages comprising the conversation so far.
     */
    public function messages(): iterable
    {
        return History::where('user_id', $this->user->id)
            ->latest()
            ->limit(50)
            ->get()
            ->reverse()
            ->map(function ($message) {
                return new Message($message->role, $message->content);
            })->all();
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new RetrievePreviousTranscripts,
        ];
    }

    /**
     * Get the agent's structured output schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'feedback' => $schema->string()->required(),
            'score' => $schema->integer()->min(1)->max(10)->required(),
        ];
    }
}
```

<a name="prompting"></a>
### Prompting

Để prompt một agent, trước tiên tạo một instance sử dụng method `make` hoặc instantiation tiêu chuẩn, sau đó gọi `prompt`:

```php
$response = (new SalesCoach)
    ->prompt('Analyze this sales transcript...');

return (string) $response;
```

Method `make` giải quyết agent của bạn từ container, cho phép automatic dependency injection. Bạn cũng có thể truyền arguments vào constructor của agent:

```php
$agent = SalesCoach::make(user: $user);
```

Bằng cách truyền thêm arguments vào method `prompt`, bạn có thể override provider, model, hoặc HTTP timeout mặc định khi prompting:

```php
$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: Lab::Anthropic,
    model: 'claude-haiku-4-5-20251001',
    timeout: 120,
);
```

<a name="conversation-context"></a>
### Ngữ cảnh hội thoại

Nếu agent của bạn implement interface `Conversational`, bạn có thể sử dụng method `messages` để trả về ngữ cảnh hội thoại trước đó, nếu có:

```php
use App\Models\History;
use Laravel\Ai\Messages\Message;

/**
 * Get the list of messages comprising the conversation so far.
 */
public function messages(): iterable
{
    return History::where('user_id', $this->user->id)
        ->latest()
        ->limit(50)
        ->get()
        ->reverse()
        ->map(function ($message) {
            return new Message($message->role, $message->content);
        })->all();
}
```

<a name="remembering-conversations"></a>
#### Remembering Conversations

> **Note:** Trước khi sử dụng trait `RemembersConversations`, bạn nên publish và chạy AI SDK migrations sử dụng lệnh Artisan `vendor:publish`. Các migrations này sẽ tạo ra các database tables cần thiết để lưu trữ hội thoại.

Nếu bạn muốn Laravel tự động lưu trữ và truy xuất lịch sử hội thoại cho agent của bạn, bạn có thể sử dụng trait `RemembersConversations`. Trait này cung cấp một cách đơn giản để persist các message hội thoại vào database mà không cần implement thủ công interface `Conversational`:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, Conversational
{
    use Promptable, RemembersConversations;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You are a sales coach...';
    }
}
```

Khi sử dụng trait `RemembersConversations`, không định nghĩa thủ công method `messages` trong class agent của bạn. Nếu method `messages` có mặt, nó sẽ được ưu tiên hơn implementation của trait và lịch sử hội thoại sẽ không được tải từ database.

Để bắt đầu một hội thoại mới cho một user, gọi method `forUser` trước khi prompting:

```php
$response = (new SalesCoach)->forUser($user)->prompt('Hello!');

$conversationId = $response->conversationId;
```

Conversation ID được trả về trên response và có thể được lưu trữ để tham khảo sau này. Nếu bạn muốn truy xuất tất cả các hội thoại của một user sử dụng Eloquent, bạn có thể thêm trait `HasConversations` vào user model của bạn:

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Ai\Concerns\HasConversations;

class User extends Authenticatable
{
    use HasConversations;
}
```

Sau khi trait đã được thêm vào model của bạn, bạn có thể truy xuất và query các hội thoại của user thông qua relationship `conversations`:

```php
$conversations = $user->conversations()
    ->latest('updated_at')
    ->paginate(20);
```

Để tiếp tục một hội thoại hiện có, sử dụng method `continue`:

```php
$response = (new SalesCoach)
    ->continue($conversationId, as: $user)
    ->prompt('Tell me more about that.');
```

Khi sử dụng trait `RemembersConversations`, các messages trước đó được tự động tải và bao gồm trong ngữ cảnh hội thoại khi prompting. Các messages mới (cả user và assistant) được tự động lưu trữ sau mỗi tương tác.

<a name="structured-output"></a>
### Structured Output

Nếu bạn muốn agent của bạn trả về structured output, implement interface `HasStructuredOutput`, yêu cầu agent của bạn định nghĩa method `schema`:

```php
<?php

namespace App\Ai\Agents;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    // ...

    /**
     * Get the agent's structured output schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
        ];
    }
}
```

Khi prompting một agent trả về structured output, bạn có thể truy cập `StructuredAgentResponse` được trả về như một array:

```php
$response = (new SalesCoach)->prompt('Analyze this sales transcript...');

return $response['score'];
```

<a name="structured-output-nested-objects"></a>
#### Nested Objects

Để định nghĩa structured output lồng nhau, sử dụng method `object` với một closure:

```php
<?php

namespace App\Ai\Agents;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    // ...

    /**
     * Get the agent's structured output schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
            'metadata' => $schema->object(fn ($schema) => [
                'confidence' => $schema->string()->enum(['low', 'medium', 'high'])->required(),
                'language' => $schema->string()->required(),
            ])->required(),
        ];
    }
}
```

<a name="structured-output-arrays-of-objects"></a>
#### Arrays of Objects

Nếu agent của bạn nên trả về một danh sách các items có cấu trúc, kết hợp các methods `array` và `object`:

```php
public function schema(JsonSchema $schema): array
{
    return [
        'feedback' => $schema->array()
            ->items(
                $schema->object(fn ($schema) => [
                    'comment' => $schema->string()->required(),
                    'score' => $schema->integer()->required(),
                ])
            )
            ->required(),
    ];
}
```

<a name="attachments"></a>
### Attachments

Khi prompting, bạn cũng có thể truyền attachments với prompt để cho phép model kiểm tra images và documents:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...',
    attachments: [
        Files\Document::fromStorage('transcript.pdf') // Attach a document from a filesystem disk...
        Files\Document::fromPath('/home/laravel/transcript.md') // Attach a document from a local path...
        $request->file('transcript'), // Attach an uploaded file...
    ]
);
```

Tương tự, class `Laravel\Ai\Files\Image` có thể được sử dụng để attach images vào một prompt:

```php
use App\Ai\Agents\ImageAnalyzer;
use Laravel\Ai\Files;

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Files\Image::fromStorage('photo.jpg') // Attach an image from a filesystem disk...
        Files\Image::fromPath('/home/laravel/photo.jpg') // Attach an image from a local path...
        $request->file('photo'), // Attach an uploaded file...
    ]
);
```

<a name="streaming"></a>
### Streaming

Bạn có thể stream response của một agent bằng cách gọi method `stream`. `StreamableAgentResponse` được trả về có thể được trả về từ một route để tự động gửi streaming response (SSE) đến client:

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)->stream('Analyze this sales transcript...');
});
```

Method `then` có thể được sử dụng để cung cấp một closure sẽ được gọi khi toàn bộ response đã được stream đến client:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Responses\StreamedAgentResponse;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->then(function (StreamedAgentResponse $response) {
            // $response->text, $response->events, $response->usage...
        });
});
```

Ngoài ra, bạn có thể iterate qua các streamed events thủ công:

```php
$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    // ...
}
```

<a name="streaming-using-the-vercel-ai-sdk-protocol"></a>
#### Streaming Using the Vercel AI SDK Protocol

Bạn có thể stream các events sử dụng [Vercel AI SDK stream protocol](https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol) bằng cách gọi method `usingVercelDataProtocol` trên streamable response:

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->usingVercelDataProtocol();
});
```

<a name="broadcasting"></a>
### Broadcasting

Bạn có thể broadcast các streamed events theo một vài cách khác nhau. Đầu tiên, bạn có thể đơn giản gọi method `broadcast` hoặc `broadcastNow` trên một streamed event:

```php
use App\Ai\Agents\SalesCoach;
use Illuminate\Broadcasting\Channel;

$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    $event->broadcast(new Channel('channel-name'));
}
```

Hoặc, bạn có thể gọi method `broadcastOnQueue` của một agent để queue agent operation và broadcast các streamed events khi chúng có sẵn:

```php
(new SalesCoach)->broadcastOnQueue(
    'Analyze this sales transcript...'
    new Channel('channel-name'),
);
```

<a name="queueing"></a>
### Queueing

Sử dụng method `queue` của một agent, bạn có thể prompt agent, nhưng cho phép nó xử lý response trong background, giữ cho ứng dụng của bạn cảm thấy nhanh và responsive. Các methods `then` và `catch` có thể được sử dụng để đăng ký các closures sẽ được gọi khi response có sẵn hoặc nếu một exception xảy ra:

```php
use Illuminate\Http\Request;
use Laravel\Ai\Responses\AgentResponse;
use Throwable;

Route::post('/coach', function (Request $request) {
    (new SalesCoach)
        ->queue($request->input('transcript'))
        ->then(function (AgentResponse $response) {
            // ...
        })
        ->catch(function (Throwable $e) {
            // ...
        });

    return back();
});
```

<a name="tools"></a>
### Tools

Tools có thể được sử dụng để cung cấp cho agents thêm chức năng mà họ có thể sử dụng khi phản hồi prompts. Tools có thể được tạo sử dụng lệnh Artisan `make:tool`:

```shell
php artisan make:tool RandomNumberGenerator
```

Tool được tạo sẽ được đặt trong thư mục `app/Ai/Tools` của ứng dụng. Mỗi tool chứa một method `handle` sẽ được gọi bởi agent khi nó cần sử dụng tool:

```php
<?php

namespace App\Ai\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Tool;
use Laravel\Ai\Tools\Request;
use Stringable;

class RandomNumberGenerator implements Tool
{
    /**
     * Get the description of the tool's purpose.
     */
    public function description(): Stringable|string
    {
        return 'This tool may be used to generate cryptographically secure random numbers.';
    }

    /**
     * Execute the tool.
     */
    public function handle(Request $request): Stringable|string
    {
        return (string) random_int($request['min'], $request['max']);
    }

    /**
     * Get the tool's schema definition.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'min' => $schema->integer()->min(0)->required(),
            'max' => $schema->integer()->required(),
        ];
    }
}
```

Sau khi bạn đã định nghĩa tool của mình, bạn có thể trả về nó từ method `tools` của bất kỳ agent nào của bạn:

```php
use App\Ai\Tools\RandomNumberGenerator;

/**
 * Get the tools available to the agent.
 *
 * @return Tool[]
 */
public function tools(): iterable
{
    return [
        new RandomNumberGenerator,
    ];
}
```

<a name="similarity-search"></a>
#### Similarity Search

Tool `SimilaritySearch` cho phép agents tìm kiếm các documents tương tự với một query nhất định sử dụng vector embeddings được lưu trữ trong database của bạn. Điều này hữu ích cho retrieval-augmented generation (RAG) khi bạn muốn cung cấp cho agents khả năng tìm kiếm dữ liệu ứng dụng của bạn.

Cách đơn giản nhất để tạo một similarity search tool là sử dụng method `usingModel` với một Eloquent model có vector embeddings:

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        SimilaritySearch::usingModel(Document::class, 'embedding'),
    ];
}
```

Argument đầu tiên là class Eloquent model, và argument thứ hai là column chứa vector embeddings.

Bạn cũng có thể cung cấp một minimum similarity threshold giữa `0.0` và `1.0` và một closure để tùy chỉnh query:

```php
SimilaritySearch::usingModel(
    model: Document::class,
    column: 'embedding',
    minSimilarity: 0.7,
    limit: 10,
    query: fn ($query) => $query->where('published', true),
),
```

Để có thêm control, bạn có thể tạo một similarity search tool với một custom closure trả về kết quả tìm kiếm:

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        new SimilaritySearch(using: function (string $query) {
            return Document::query()
                ->where('user_id', $this->user->id)
                ->whereVectorSimilarTo('embedding', $query)
                ->limit(10)
                ->get();
        }),
    ];
}
```

Bạn có thể tùy chỉnh description của tool sử dụng method `withDescription`:

```php
SimilaritySearch::usingModel(Document::class, 'embedding')
    ->withDescription('Search the knowledge base for relevant articles.'),
```

<a name="mcp-tools"></a>
### MCP Tools

Nếu ứng dụng của bạn sử dụng [Laravel MCP](/docs/{{version}}/mcp), bạn có thể cung cấp cho agents các tools được expose bởi [Model Context Protocol](https://modelcontextprotocol.io) servers. Sử dụng [Laravel MCP client](/docs/{{version}}/mcp#client), bạn có thể kết nối đến một MCP server remote hoặc local và truyền các tools của nó trực tiếp đến agent của bạn.

> [!NOTE]
> MCP tools yêu cầu package [Laravel MCP](/docs/{{version}}/mcp) được cài đặt trong ứng dụng của bạn.

Vì method `tools` của một MCP client trả về một collection, hãy spread nó vào array `tools` của agent của bạn sử dụng operator `...`:

```php
use App\Ai\Tools\RandomNumberGenerator;
use Laravel\Mcp\Client;

/**
 * Get the tools available to the agent.
 *
 * @return Tool[]
 */
public function tools(): iterable
{
    return [
        ...Client::web('https://mcp.example.com')
            ->withToken($token)
            ->tools(),

        new RandomNumberGenerator,
    ];
}
```

AI SDK tự động wrap mỗi MCP tool để agent có thể gọi nó như bất kỳ tool nào khác. Bạn cũng có thể sử dụng một [named MCP client](/docs/{{version}}/mcp#named-clients):

```php
use Laravel\Mcp\Facades\Mcp;

public function tools(): iterable
{
    return [
        ...Mcp::client('github')->tools(),
    ];
}
```

Hoặc kết nối đến một [local MCP server](/docs/{{version}}/mcp#client-connecting):

```php
use Laravel\Mcp\Client;

public function tools(): iterable
{
    return [
        ...Client::local('php', ['artisan', 'mcp:start'])->tools(),
    ];
}
```

Để biết thêm thông tin về việc tạo và xác thực MCP clients, bao gồm bearer tokens và OAuth, hãy tham khảo [MCP client documentation](/docs/{{version}}/mcp#client).

<a name="provider-tools"></a>
### Provider Tools

Provider tools là các tools đặc biệt được implement natively bởi AI providers, cung cấp các khả năng như web searching, URL fetching, và file searching. Khác với các tools thông thường, provider tools được thực thi bởi chính provider thay vì ứng dụng của bạn.

Provider tools có thể được trả về bởi method `tools` của agent của bạn.

<a name="web-search"></a>
#### Web Search

Tool provider `WebSearch` cho phép agents tìm kiếm web để lấy thông tin thời gian thực. Điều này hữu ích để trả lời các câu hỏi về sự kiện hiện tại, dữ liệu gần đây, hoặc các chủ đề có thể đã thay đổi kể từ training cutoff của model.

**Supported Providers:** Anthropic, OpenAI, Gemini

```php
use Laravel\Ai\Providers\Tools\WebSearch;

public function tools(): iterable
{
    return [
        new WebSearch,
    ];
}
```

Bạn có thể cấu hình web search tool để giới hạn số lượng tìm kiếm hoặc hạn chế kết quả đến các domains cụ thể:

```php
(new WebSearch)->max(5)->allow(['laravel.com', 'php.net']),
```

Để tinh chỉnh kết quả tìm kiếm dựa trên vị trí người dùng, sử dụng method `location`:

```php
(new WebSearch)->location(
    city: 'New York',
    region: 'NY',
    country: 'US'
);
```

<a name="web-fetch"></a>
#### Web Fetch

Tool provider `WebFetch` cho phép agents fetch và đọc nội dung của các web pages. Điều này hữu ích khi bạn cần agent phân tích các URLs cụ thể hoặc truy xuất thông tin chi tiết từ các web pages đã biết.

**Supported providers:** Anthropic, Gemini

```php
use Laravel\Ai\Providers\Tools\WebFetch;

public function tools(): iterable
{
    return [
        new WebFetch,
    ];
}
```

Bạn có thể cấu hình web fetch tool để giới hạn số lượng fetches hoặc hạn chế đến các domains cụ thể:

```php
(new WebFetch)->max(3)->allow(['docs.laravel.com']),
```

<a name="file-search"></a>
#### File Search

Tool provider `FileSearch` cho phép agents tìm kiếm qua các [files](#files) được lưu trữ trong [vector stores](#vector-stores). Điều này cho phép retrieval-augmented generation (RAG) bằng cách cho phép agent tìm kiếm các documents đã upload của bạn để lấy thông tin liên quan.

**Supported providers:** OpenAI, Gemini

```php
use Laravel\Ai\Providers\Tools\FileSearch;

public function tools(): iterable
{
    return [
        new FileSearch(stores: ['store_id']),
    ];
}
```

Bạn có thể cung cấp nhiều vector store IDs để tìm kiếm qua nhiều stores:

```php
new FileSearch(stores: ['store_1', 'store_2']);
```

Nếu files của bạn có [metadata](#adding-files-to-stores), bạn có thể lọc kết quả tìm kiếm bằng cách cung cấp argument `where`. Đối với các bộ lọc equality đơn giản, truyền một array:

```php
new FileSearch(stores: ['store_id'], where: [
    'author' => 'Taylor Otwell',
    'year' => 2026,
]);
```

Đối với các bộ lọc phức tạp hơn, bạn có thể truyền một closure nhận một instance `FileSearchQuery`:

```php
use Laravel\Ai\Providers\Tools\FileSearchQuery;

new FileSearch(stores: ['store_id'], where: fn (FileSearchQuery $query) =>
    $query->where('author', 'Taylor Otwell')
        ->whereNot('status', 'draft')
        ->whereIn('category', ['news', 'updates'])
);
```

<a name="sub-agents"></a>
### Sub-Agents

Agents cũng có thể được trả về từ method `tools` của một agent khác. Khi một agent được trả về như một tool, agent parent có thể delegate một task cụ thể đến sub-agent và sử dụng response của sub-agent trong khi trả lời prompt gốc. Điều này hữu ích khi một general-purpose agent cần truy cập đến các specialized agents với instructions, tools, cấu hình model, hoặc preferences provider riêng của họ.

Ví dụ, một customer support agent có thể delegate các câu hỏi về refund eligibility đến một refunds agent chuyên biệt:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Promptable;

class CustomerSupportAgent implements Agent, HasTools
{
    use Promptable;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You help customers with account, order, and billing questions. Delegate refund policy questions to the refunds specialist.';
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new RefundsAgent,
        ];
    }
}
```

Để tùy chỉnh cách sub-agent được expose đến agent parent, implement interface `CanActAsTool` trên sub-agent và định nghĩa một tool-facing name và description:

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\LookupOrder;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\CanActAsTool;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
class RefundsAgent implements Agent, CanActAsTool, HasTools
{
    use Promptable;

    /**
     * Get the instructions that the agent should follow.
     */
    public function instructions(): string
    {
        return 'You are a refunds specialist. Use order details and the refund policy to give concise eligibility guidance.';
    }

    /**
     * Get the agent's tool name.
     */
    public function name(): string
    {
        return 'refunds_specialist';
    }

    /**
     * Get the agent's tool description.
     */
    public function description(): string
    {
        return 'Determine whether an order is eligible for a refund and explain the next step.';
    }

    /**
     * Get the tools available to the agent.
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new LookupOrder,
        ];
    }
}
```

Nếu một sub-agent không implement `CanActAsTool`, Laravel sẽ sử dụng class basename của agent như tool name và một description generic yêu cầu agent parent truyền một task description rõ ràng và tự chứa. Mỗi lần gọi sub-agent chạy trong isolation và không nhận lịch sử hội thoại của agent parent.

<a name="middleware"></a>
### Middleware

Agents hỗ trợ middleware, cho phép bạn intercept và sửa đổi prompts trước khi chúng được gửi đến provider. Middleware có thể được tạo sử dụng lệnh Artisan `make:agent-middleware`:

```shell
php artisan make:agent-middleware LogPrompts
```

Middleware được tạo sẽ được đặt trong thư mục `app/Ai/Middleware` của ứng dụng. Để thêm middleware vào một agent, implement interface `HasMiddleware` và định nghĩa method `middleware` trả về một array của các class middleware:

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Middleware\LogPrompts;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasMiddleware;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasMiddleware
{
    use Promptable;

    // ...

    /**
     * Get the agent's middleware.
     */
    public function middleware(): array
    {
        return [
            new LogPrompts,
        ];
    }
}
```

Mỗi class middleware nên định nghĩa một method `handle` nhận `AgentPrompt` và một `Closure` để truyền prompt đến middleware tiếp theo:

```php
<?php

namespace App\Ai\Middleware;

use Closure;
use Laravel\Ai\Prompts\AgentPrompt;

class LogPrompts
{
    /**
     * Handle the incoming prompt.
     */
    public function handle(AgentPrompt $prompt, Closure $next)
    {
        Log::info('Prompting agent', ['prompt' => $prompt->prompt]);

        return $next($prompt);
    }
}
```

Bạn có thể sử dụng method `then` trên response để thực thi code sau khi agent đã hoàn thành xử lý. Điều này hoạt động cho cả synchronous và streaming responses:

```php
public function handle(AgentPrompt $prompt, Closure $next)
{
    return $next($prompt)->then(function (AgentResponse $response) {
        Log::info('Agent responded', ['text' => $response->text]);
    });
}
```

<a name="anonymous-agents"></a>
### Anonymous Agents

Đôi khi bạn có thể muốn tương tác nhanh với một model mà không cần tạo một class agent chuyên biệt. Bạn có thể tạo một agent ad-hoc, anonymous sử dụng function `agent`:

```php
use function Laravel\Ai\{agent};

$response = agent(
    instructions: 'You are an expert at software development.',
    messages: [],
    tools: [],
)->prompt('Tell me about Laravel')
```

Anonymous agents cũng có thể tạo ra structured output:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;

use function Laravel\Ai\{agent};

$response = agent(
    schema: fn (JsonSchema $schema) => [
        'number' => $schema->integer()->required(),
    ],
)->prompt('Generate a random number less than 100')
```

<a name="agent-configuration"></a>
### Cấu hình Agent

Bạn có thể cấu hình các tùy chọn tạo text cho một agent sử dụng PHP attributes. Các attributes sau có sẵn:

- `MaxSteps`: Số bước tối đa mà agent có thể thực hiện khi sử dụng tools.
- `MaxTokens`: Số token tối đa mà model có thể tạo.
- `Model`: Model mà agent nên sử dụng.
- `Provider`: AI provider (hoặc các providers cho failover) để sử dụng cho agent.
- `Temperature`: Sampling temperature để sử dụng cho generation (0.0 đến 1.0).
- `Timeout`: HTTP timeout tính bằng giây cho agent requests (default: 60).
- `TopP`: Nucleus sampling probability để sử dụng cho generation (0.0 đến 1.0).
- `UseCheapestModel`: Sử dụng text model rẻ nhất của provider để tối ưu hóa chi phí.
- `UseSmartestModel`: Sử dụng text model có khả năng nhất của provider cho các tasks phức tạp.

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Attributes\MaxSteps;
use Laravel\Ai\Attributes\MaxTokens;
use Laravel\Ai\Attributes\Model;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Attributes\Temperature;
use Laravel\Ai\Attributes\Timeout;
use Laravel\Ai\Attributes\TopP;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
#[Model('claude-haiku-4-5-20251001')]
#[MaxSteps(10)]
#[MaxTokens(4096)]
#[Temperature(0.7)]
#[Timeout(120)]
#[TopP(0.9)]
class SalesCoach implements Agent
{
    use Promptable;

    // ...
}
```

Các attributes `UseCheapestModel` và `UseSmartestModel` cho phép bạn tự động chọn model hiệu quả về chi phí nhất hoặc có khả năng nhất cho một provider nhất định mà không cần chỉ định tên model. Điều này hữu ích khi bạn muốn tối ưu hóa cho chi phí hoặc khả năng trên các providers khác nhau:

```php
use Laravel\Ai\Attributes\UseCheapestModel;
use Laravel\Ai\Attributes\UseSmartestModel;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Promptable;

#[UseCheapestModel]
class SimpleSummarizer implements Agent
{
    use Promptable;

    // Will use the cheapest model (e.g., Haiku)...
}

#[UseSmartestModel]
class ComplexReasoner implements Agent
{
    use Promptable;

    // Will use the most capable model (e.g., Opus)...
}
```

> [!NOTE]
> Model cơ bản được chọn bởi `UseCheapestModel` và `UseSmartestModel` có thể thay đổi giữa các releases của Laravel AI SDK khi các providers phát hành các models mới. Việc chuyển đổi models có thể giới thiệu các thay đổi hành vi, các parameters đã deprecated, và sự khác biệt chi phí đáng kể. Nếu bạn cần một model và pricing ổn định, có thể dự đoán, hãy chỉ định model một cách rõ ràng sử dụng attribute `Model`.

<a name="provider-options"></a>
### Tùy chọn Provider

Nếu agent của bạn cần truyền các tùy chọn cụ thể của provider (như OpenAI reasoning effort hoặc penalty settings), implement contract `HasProviderOptions` và định nghĩa method `providerOptions`:

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasProviderOptions;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasProviderOptions
{
    use Promptable;

    // ...

    /**
     * Get provider-specific generation options.
     */
    public function providerOptions(Lab|string $provider): array
    {
        return match ($provider) {
            Lab::OpenAI => [
                'reasoning' => ['effort' => 'low'],
                'frequency_penalty' => 0.5,
                'presence_penalty' => 0.3,
            ],
            Lab::Anthropic => [
                'thinking' => ['budget_tokens' => 1024],
                'cache_control' => ['type' => 'ephemeral'],
            ],
            default => [],
        };
    }
}
```

Method `providerOptions` nhận provider hiện đang được sử dụng (enum `Lab` hoặc string), cho phép bạn trả về các tùy chọn khác nhau cho mỗi provider. Điều này đặc biệt hữu ích khi sử dụng [failover](#failover), vì mỗi fallback provider có thể nhận cấu hình riêng của nó.

Ví dụ Anthropic ở trên cũng kích hoạt [prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) thông qua `cache_control`.

<a name="images"></a>
## Images

Class `Laravel\Ai\Image` có thể được sử dụng để tạo images sử dụng các providers `openai`, `gemini`, hoặc `xai`:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

Các methods `square`, `portrait`, và `landscape` có thể được sử dụng để kiểm soát aspect ratio của image, trong khi method `quality` có thể được sử dụng để hướng dẫn model về chất lượng image cuối cùng (`high`, `medium`, `low`). Method `timeout` có thể được sử dụng để chỉ định HTTP timeout tính bằng giây:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')
    ->quality('high')
    ->landscape()
    ->timeout(120)
    ->generate();
```

Bạn có thể attach reference images sử dụng method `attachments`:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Image;

$image = Image::of('Update this photo of me to be in the style of an impressionist painting.')
    ->attachments([
        Files\Image::fromStorage('photo.jpg'),
        // Files\Image::fromPath('/home/laravel/photo.jpg'),
        // Files\Image::fromUrl('https://example.com/photo.jpg'),
        // $request->file('photo'),
    ])
    ->landscape()
    ->generate();
```

Các images được tạo có thể được lưu trữ dễ dàng trên disk mặc định được cấu hình trong file cấu hình `config/filesystems.php` của ứng dụng:

```php
$image = Image::of('A donut sitting on the kitchen counter');

$path = $image->store();
$path = $image->storeAs('image.jpg');
$path = $image->storePublicly();
$path = $image->storePubliclyAs('image.jpg');
```

Tạo image cũng có thể được queued:

```php
use Laravel\Ai\Image;
use Laravel\Ai\Responses\ImageResponse;

Image::of('A donut sitting on the kitchen counter')
    ->portrait()
    ->queue()
    ->then(function (ImageResponse $image) {
        $path = $image->store();

        // ...
    });
```

<a name="audio"></a>
## Audio

Class `Laravel\Ai\Audio` có thể được sử dụng để tạo audio từ text đã cho:

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

Bạn cũng có thể tạo audio từ một string sử dụng method `toAudio` có sẵn thông qua class `Stringable` của Laravel:

```php
use Illuminate\Support\Str;

$audio = Str::of('I love coding with Laravel.')->toAudio();
```

Các methods `male`, `female`, và `voice` có thể được sử dụng để xác định voice của audio được tạo:

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->generate();

$audio = Audio::of('I love coding with Laravel.')
    ->voice('voice-id-or-name')
    ->generate();
```

Tương tự, method `instructions` có thể được sử dụng để dynamically hướng dẫn model về cách audio được tạo nên nghe như thế nào:

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->instructions('Said like a pirate')
    ->generate();
```

Audio được tạo có thể được lưu trữ dễ dàng trên disk mặc định được cấu hình trong file cấu hình `config/filesystems.php` của ứng dụng:

```php
$audio = Audio::of('I love coding with Laravel.')->generate();

$path = $audio->store();
$path = $audio->storeAs('audio.mp3');
$path = $audio->storePublicly();
$path = $audio->storePubliclyAs('audio.mp3');
```

Tạo audio cũng có thể được queued:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Responses\AudioResponse;

Audio::of('I love coding with Laravel.')
    ->queue()
    ->then(function (AudioResponse $audio) {
        $path = $audio->store();

        // ...
    });
```

<a name="transcription"></a>
## Transcriptions

Class `Laravel\Ai\Transcription` có thể được sử dụng để tạo một transcript của audio đã cho:

```php
use Laravel\Ai\Transcription;

$transcript = Transcription::fromPath('/home/laravel/audio.mp3')->generate();
$transcript = Transcription::fromStorage('audio.mp3')->generate();
$transcript = Transcription::fromUpload($request->file('audio'))->generate();

return (string) $transcript;
```

Method `diarize` có thể được sử dụng để chỉ định bạn muốn response bao gồm diarized transcript ngoài raw text transcript, cho phép bạn truy cập segmented transcript theo speaker:

```php
$transcript = Transcription::fromStorage('audio.mp3')
    ->diarize()
    ->generate();
```

Tạo transcription cũng có thể được queued:

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Responses\TranscriptionResponse;

Transcription::fromStorage('audio.mp3')
    ->queue()
    ->then(function (TranscriptionResponse $transcript) {
        // ...
    });
```

<a name="embeddings"></a>
## Embeddings

Bạn có thể dễ dàng tạo vector embeddings cho bất kỳ string nào sử dụng method `toEmbeddings` mới có sẵn thông qua class `Stringable` của Laravel:

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

Ngoài ra, bạn có thể sử dụng class `Embeddings` để tạo embeddings cho nhiều inputs cùng một lúc:

```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```

Bạn có thể chỉ định dimensions và provider cho embeddings:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->dimensions(1536)
    ->generate(Lab::OpenAI, 'text-embedding-3-small');
```

<a name="querying-embeddings"></a>
### Querying Embeddings

Sau khi bạn đã tạo embeddings, bạn thường sẽ lưu trữ chúng trong một column `vector` trong database để query sau này. Laravel cung cấp hỗ trợ native cho các vector columns trên PostgreSQL thông qua extension `pgvector`. Để bắt đầu, định nghĩa một column `vector` trong migration của bạn, chỉ định số lượng dimensions:

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536);
    $table->timestamps();
});
```

Bạn cũng có thể thêm một vector index để tăng tốc độ similarity searches. Khi gọi `index` trên một vector column, Laravel sẽ tự động tạo một HNSW index với cosine distance:

```php
$table->vector('embedding', dimensions: 1536)->index();
```

Trên Eloquent model của bạn, bạn nên cast vector column thành một `array`:

```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```

Để query cho các records tương tự, sử dụng method `whereVectorSimilarTo`. Method này lọc kết quả theo một minimum cosine similarity (giữa `0.0` và `1.0`, trong đó `1.0` là giống hệt) và sắp xếp kết quả theo similarity:

```php
use App\Models\Document;

$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

`$queryEmbedding` có thể là một array của floats hoặc một plain string. Khi một string được đưa ra, Laravel sẽ tự động tạo embeddings cho nó:

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

Nếu bạn cần thêm control, bạn có thể sử dụng các methods cấp thấp hơn `whereVectorDistanceLessThan`, `selectVectorDistance`, và `orderByVectorDistance` một cách độc lập:

```php
$documents = Document::query()
    ->select('*')
    ->selectVectorDistance('embedding', $queryEmbedding, as: 'distance')
    ->whereVectorDistanceLessThan('embedding', $queryEmbedding, maxDistance: 0.3)
    ->orderByVectorDistance('embedding', $queryEmbedding)
    ->limit(10)
    ->get();
```

Nếu bạn muốn cung cấp cho agent khả năng thực hiện similarity searches như một tool, hãy xem tài liệu tool [Similarity Search](#similarity-search).

> [!NOTE]
> Vector queries hiện chỉ được hỗ trợ trên các kết nối PostgreSQL sử dụng extension `pgvector`.

<a name="caching-embeddings"></a>
### Caching Embeddings

Tạo embedding có thể được cached để tránh các API calls dư thừa cho các inputs giống hệt nhau. Để kích hoạt động caching, đặt tùy chọn cấu hình `ai.caching.embeddings.cache` thành `true`:

```php
'caching' => [
    'embeddings' => [
        'cache' => true,
        'store' => env('CACHE_STORE', 'database'),
        // ...
    ],
],
```

Khi caching được kích hoạt, embeddings được cached trong 30 ngày. Cache key dựa trên provider, model, dimensions, và nội dung input, đảm bảo rằng các requests giống hệt nhau trả về kết quả cached trong khi các cấu hình khác nhau tạo ra embeddings mới.

Bạn cũng có thể kích hoạt động caching cho một request cụ thể sử dụng method `cache`, ngay cả khi global caching bị tắt:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache()
    ->generate();
```

Bạn có thể chỉ định một custom cache duration tính bằng giây:

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache(seconds: 3600) // Cache for 1 hour
    ->generate();
```

Method Stringable `toEmbeddings` cũng chấp nhận một argument `cache`:

```php
// Cache with default duration...
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: true);

// Cache for a specific duration...
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: 3600);
```

<a name="reranking"></a>
## Reranking

Reranking cho phép bạn sắp xếp lại một danh sách documents dựa trên relevance của chúng với một query nhất định. Điều này hữu ích để cải thiện kết quả tìm kiếm bằng cách sử dụng semantic understanding:

Class `Laravel\Ai\Reranking` có thể được sử dụng để rerank documents:

```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

// Access the top result...
$response->first()->document; // "Laravel is a PHP web application framework."
$response->first()->score;    // 0.95
$response->first()->index;    // 1 (original position)
```

Method `limit` có thể được sử dụng để hạn chế số lượng kết quả được trả về:

```php
$response = Reranking::of($documents)
    ->limit(5)
    ->rerank('search query');
```

<a name="reranking-collections"></a>
### Reranking Collections

Để thuận tiện, các Laravel collections có thể được rerank sử dụng macro `rerank`. Argument đầu tiên chỉ định field(s) nào để sử dụng cho reranking, và argument thứ hai là query:

```php
// Rerank by a single field...
$posts = Post::all()
    ->rerank('body', 'Laravel tutorials');

// Rerank by multiple fields (sent as JSON)...
$reranked = $posts->rerank(['title', 'body'], 'Laravel tutorials');

// Rerank using a closure to build the document...
$reranked = $posts->rerank(
    fn ($post) => $post->title.': '.$post->body,
    'Laravel tutorials'
);
```

Bạn cũng có thể hạn chế số lượng kết quả và chỉ định một provider:

```php
$reranked = $posts->rerank(
    by: 'content',
    query: 'Laravel tutorials',
    limit: 10,
    provider: Lab::Cohere
);
```

<a name="files"></a>
## Files

Class `Laravel\Ai\Files` hoặc các class file riêng lẻ có thể được sử dụng để lưu trữ files với AI provider của bạn để sử dụng sau này trong các cuộc hội thoại. Điều này hữu ích cho các documents hoặc files lớn mà bạn muốn tham chiếu nhiều lần mà không cần re-upload:

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Files\Image;

// Store a file from a local path...
$response = Document::fromPath('/home/laravel/document.pdf')->put();
$response = Image::fromPath('/home/laravel/photo.jpg')->put();

// Store a file that is stored on a filesystem disk...
$response = Document::fromStorage('document.pdf', disk: 'local')->put();
$response = Image::fromStorage('photo.jpg', disk: 'local')->put();

// Store a file that is stored on a remote URL...
$response = Document::fromUrl('https://example.com/document.pdf')->put();
$response = Image::fromUrl('https://example.com/photo.jpg')->put();

return $response->id;
```

Bạn cũng có thể lưu trữ raw content hoặc uploaded files:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Document;

// Store raw content...
$stored = Document::fromString('Hello, World!', 'text/plain')->put();

// Store an uploaded file...
$stored = Document::fromUpload($request->file('document'))->put();
```

Sau khi một file đã được lưu trữ, bạn có thể tham chiếu file khi tạo text thông qua agents thay vì re-upload file:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...'
    attachments: [
        Files\Document::fromId('file-id') // Attach a stored document...
    ]
);
```

Để truy xuất một file đã lưu trữ trước đó, sử dụng method `get` trên một file instance:

```php
use Laravel\Ai\Files\Document;

$file = Document::fromId('file-id')->get();

$file->id;
$file->mimeType();
```

Để xóa một file từ provider, sử dụng method `delete`:

```php
Document::fromId('file-id')->delete();
```

Theo mặc định, class `Files` sử dụng AI provider mặc định được cấu hình trong file cấu hình `config/ai.php` của ứng dụng. Đối với hầu hết các operations, bạn có thể chỉ định một provider khác sử dụng argument `provider`:

```php
$response = Document::fromPath(
    '/home/laravel/document.pdf'
)->put(provider: Lab::Anthropic);
```

<a name="using-stored-files-in-conversations"></a>
### Using Stored Files in Conversations

Sau khi một file đã được lưu trữ với một provider, bạn có thể tham chiếu nó trong các cuộc hội thoại agent sử dụng method `fromId` trên các class `Document` hoặc `Image`:

```php
use App\Ai\Agents\DocumentAnalyzer;
use Laravel\Ai\Files;
use Laravel\Ai\Files\Document;

$stored = Document::fromPath('/path/to/report.pdf')->put();

$response = (new DocumentAnalyzer)->prompt(
    'Summarize this document.',
    attachments: [
        Document::fromId($stored->id),
    ],
);
```

Tương tự, các images đã lưu trữ có thể được tham chiếu sử dụng class `Image`:

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Image;

$stored = Image::fromPath('/path/to/photo.jpg')->put();

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Image::fromId($stored->id),
    ],
);
```

<a name="vector-stores"></a>
## Vector Stores

Vector stores cho phép bạn tạo các collections có thể tìm kiếm của files có thể được sử dụng cho retrieval-augmented generation (RAG). Class `Laravel\Ai\Stores` cung cấp các methods để tạo, truy xuất, và xóa vector stores:

```php
use Laravel\Ai\Stores;

// Create a new vector store...
$store = Stores::create('Knowledge Base');

// Create a store with additional options...
$store = Stores::create(
    name: 'Knowledge Base',
    description: 'Documentation and reference materials.',
    expiresWhenIdleFor: days(30),
);

return $store->id;
```

Để truy xuất một vector store hiện có theo ID của nó, sử dụng method `get`:

```php
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

$store->id;
$store->name;
$store->fileCounts;
$store->ready;
```

Để xóa một vector store, sử dụng method `delete` trên class `Stores` hoặc store instance:

```php
use Laravel\Ai\Stores;

// Delete by ID...
Stores::delete('store_id');

// Or delete via a store instance...
$store = Stores::get('store_id');

$store->delete();
```

<a name="adding-files-to-stores"></a>
### Adding Files to Stores

Sau khi bạn có một vector store, bạn có thể thêm [files](#files) vào nó sử dụng method `add`. Các files được thêm vào một store được tự động indexed cho semantic searching sử dụng [file search provider tool](#file-search):

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

// Add a file that has already been stored with the provider...
$document = $store->add('file_id');
$document = $store->add(Document::fromId('file_id'));

// Or, store and add a file in one step...
$document = $store->add(Document::fromPath('/path/to/document.pdf'));
$document = $store->add(Document::fromStorage('manual.pdf'));
$document = $store->add($request->file('document'));

$document->id;
$document->fileId;
```

> **Note:** Thông thường, khi thêm các files đã lưu trữ trước đó vào vector stores, document ID được trả về sẽ khớp với ID đã gán trước đó của file; tuy nhiên, một số vector storage providers có thể trả về một "document ID" mới, khác nhau. Do đó, được khuyến khuyến rằng bạn luôn lưu trữ cả hai IDs trong database của bạn để tham khảo sau này.

Bạn có thể attach metadata vào files khi thêm chúng vào một store. Metadata này có thể được sử dụng sau này để lọc kết quả tìm kiếm khi sử dụng [file search provider tool](#file-search):

```php
$store->add(Document::fromPath('/path/to/document.pdf'), metadata: [
    'author' => 'Taylor Otwell',
    'department' => 'Engineering',
    'year' => 2026,
]);
```

Để xóa một file từ một store, sử dụng method `remove`:

```php
$store->remove('file_id');
```

Xóa một file từ một vector store không xóa nó từ [file storage](#files) của provider. Để xóa một file từ vector store và xóa nó vĩnh viễn từ file storage, sử dụng argument `deleteFile`:

```php
$store->remove('file_abc123', deleteFile: true);
```

<a name="failover"></a>
## Failover

Khi prompting hoặc tạo các media khác, bạn có thể cung cấp một array của providers / models để tự động failover đến một backup provider / model nếu một service interruption hoặc rate limit được gặp phải trên provider chính:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Image;

$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: [Lab::OpenAI, Lab::Anthropic],
);

$image = Image::of('A donut sitting on the kitchen counter')
    ->generate(provider: [Lab::Gemini, Lab::xAI]);
```

Failover chỉ xảy ra khi một `FailoverableException` được ném — chẳng hạn như một rate limit (`RateLimitedException`), một provider bị quá tải hoặc không khả dụng (`ProviderOverloadedException`), hoặc insufficient credits (`InsufficientCreditsException`). Các lỗi thông thường, như một validation hoặc bad request error, sẽ không kích hoạt failover.

Khi bạn truyền một plain list của providers, chẳng hạn như `[Lab::OpenAI, Lab::Anthropic]`, mỗi provider sử dụng model mặc định của nó. Để chỉ định một model cụ thể cho mỗi provider trong failover chain, truyền một associative array được key bởi provider, sử dụng `value` của enum `Lab` làm key (enum cases không thể được sử dụng trực tiếp như PHP array keys):

```php
use Laravel\Ai\Enums\Lab;

$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: [
        Lab::Gemini->value => 'gemini-3-flash-preview',
        Lab::DeepSeek->value => 'deepseek-v4-pro',
    ],
);
```

<a name="testing"></a>
## Testing

<a name="testing-agents"></a>
### Agents

Để fake responses của một agent trong các tests, gọi method `fake` trên class agent. Bạn có thể tùy chọn cung cấp một array của responses hoặc một closure:

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Prompts\AgentPrompt;

// Automatically generate a fixed response for every prompt...
SalesCoach::fake();

// Provide a list of prompt responses...
SalesCoach::fake([
    'First response',
    'Second response',
]);

// Dynamically handle prompt responses based on the incoming prompt...
SalesCoach::fake(function (AgentPrompt $prompt) {
    return 'Response for: '.$prompt->prompt;
});
```

> **Note:** Khi `Agent::fake()` được gọi trên một agent trả về structured output, Laravel sẽ tự động tạo fake data khớp với output schema được định nghĩa của agent.

Sau khi prompting agent, bạn có thể thực hiện các assertions về các prompts đã được nhận:

```php
use Laravel\Ai\Prompts\AgentPrompt;

SalesCoach::assertPrompted('Analyze this...');

SalesCoach::assertPrompted(function (AgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotPrompted('Missing prompt');

SalesCoach::assertNeverPrompted();
```

Đối với các queued agent invocations, sử dụng các queued assertion methods:

```php
use Laravel\Ai\QueuedAgentPrompt;

SalesCoach::assertQueued('Analyze this...');

SalesCoach::assertQueued(function (QueuedAgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotQueued('Missing prompt');

SalesCoach::assertNeverQueued();
```

Để đảm bảo tất cả các agent invocations có một fake response tương ứng, bạn có thể sử dụng `preventStrayPrompts`. Nếu một agent được gọi mà không có một fake response được định nghĩa, một exception sẽ được ném:

```php
SalesCoach::fake()->preventStrayPrompts();
```

<a name="testing-images"></a>
### Images

Tạo images có thể được fake bằng cách gọi method `fake` trên class `Image`. Sau khi image đã được fake, các assertions khác nhau có thể được thực hiện đối với các image generation prompts đã ghi lại:

```php
use Laravel\Ai\Image;
use Laravel\Ai\Prompts\ImagePrompt;
use Laravel\Ai\Prompts\QueuedImagePrompt;

// Automatically generate a fixed response for every prompt...
Image::fake();

// Provide a list of prompt responses...
Image::fake([
    base64_encode($firstImage),
    base64_encode($secondImage),
]);

// Dynamically handle prompt responses based on the incoming prompt...
Image::fake(function (ImagePrompt $prompt) {
    return base64_encode('...');
});
```

Sau khi tạo images, bạn có thể thực hiện các assertions về các prompts đã được nhận:

```php
Image::assertGenerated(function (ImagePrompt $prompt) {
    return $prompt->contains('sunset') && $prompt->isLandscape();
});

Image::assertNotGenerated('Missing prompt');

Image::assertNothingGenerated();
```

Đối với các queued image generations, sử dụng các queued assertion methods:

```php
Image::assertQueued(
    fn (QueuedImagePrompt $prompt) => $prompt->contains('sunset')
);

Image::assertNotQueued('Missing prompt');

Image::assertNothingQueued();
```

Để đảm bảo tất cả các image generations có một fake response tương ứng, bạn có thể sử dụng `preventStrayImages`. Nếu một image được tạo mà không có một fake response được định nghĩa, một exception sẽ được ném:

```php
Image::fake()->preventStrayImages();
```

<a name="testing-audio"></a>
### Audio

Tạo audio có thể được fake bằng cách gọi method `fake` trên class `Audio`. Sau khi audio đã được fake, các assertions khác nhau có thể được thực hiện đối với các audio generation prompts đã ghi lại:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Prompts\AudioPrompt;
use Laravel\Ai\Prompts\QueuedAudioPrompt;

// Automatically generate a fixed response for every prompt...
Audio::fake();

// Provide a list of prompt responses...
Audio::fake([
    base64_encode($firstAudio),
    base64_encode($secondAudio),
]);

// Dynamically handle prompt responses based on the incoming prompt...
Audio::fake(function (AudioPrompt $prompt) {
    return base64_encode('...');
});
```

Sau khi tạo audio, bạn có thể thực hiện các assertions về các prompts đã được nhận:

```php
Audio::assertGenerated(function (AudioPrompt $prompt) {
    return $prompt->contains('Hello') && $prompt->isFemale();
});

Audio::assertNotGenerated('Missing prompt');

Audio::assertNothingGenerated();
```

Đối với các queued audio generations, sử dụng các queued assertion methods:

```php
Audio::assertQueued(
    fn (QueuedAudioPrompt $prompt) => $prompt->contains('Hello')
);

Audio::assertNotQueued('Missing prompt');

Audio::assertNothingQueued();
```

Để đảm bảo tất cả các audio generations có một fake response tương ứng, bạn có thể sử dụng `preventStrayAudio`. Nếu audio được tạo mà không có một fake response được định nghĩa, một exception sẽ được ném:

```php
Audio::fake()->preventStrayAudio();
```

<a name="testing-transcriptions"></a>
### Transcriptions

Tạo transcription có thể được fake bằng cách gọi method `fake` trên class `Transcription`. Sau khi transcription đã được fake, các assertions khác nhau có thể được thực hiện đối với các transcription generation prompts đã ghi lại:

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Prompts\TranscriptionPrompt;
use Laravel\Ai\Prompts\QueuedTranscriptionPrompt;

// Automatically generate a fixed response for every prompt...
Transcription::fake();

// Provide a list of prompt responses...
Transcription::fake([
    'First transcription text.',
    'Second transcription text.',
]);

// Dynamically handle prompt responses based on the incoming prompt...
Transcription::fake(function (TranscriptionPrompt $prompt) {
    return 'Transcribed text...';
});
```

Sau khi tạo transcriptions, bạn có thể thực hiện các assertions về các prompts đã được nhận:

```php
Transcription::assertGenerated(function (TranscriptionPrompt $prompt) {
    return $prompt->language === 'en' && $prompt->isDiarized();
});

Transcription::assertNotGenerated(
    fn (TranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingGenerated();
```

Đối với các queued transcription generations, sử dụng các queued assertion methods:

```php
Transcription::assertQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->isDiarized()
);

Transcription::assertNotQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingQueued();
```

Để đảm bảo tất cả các transcription generations có một fake response tương ứng, bạn có thể sử dụng `preventStrayTranscriptions`. Nếu một transcription được tạo mà không có một fake response được định nghĩa, một exception sẽ được ném:

```php
Transcription::fake()->preventStrayTranscriptions();
```

<a name="testing-embeddings"></a>
### Embeddings

Tạo embeddings có thể được fake bằng cách gọi method `fake` trên class `Embeddings`. Sau khi embeddings đã được fake, các assertions khác nhau có thể được thực hiện đối với các embeddings generation prompts đã ghi lại:

```php
use Laravel\Ai\Embeddings;
use Laravel\Ai\Prompts\EmbeddingsPrompt;
use Laravel\Ai\Prompts\QueuedEmbeddingsPrompt;

// Automatically generate fake embeddings of the proper dimensions for every prompt...
Embeddings::fake();

// Provide a list of prompt responses...
Embeddings::fake([
    [$firstEmbeddingVector],
    [$secondEmbeddingVector],
]);

// Dynamically handle prompt responses based on the incoming prompt...
Embeddings::fake(function (EmbeddingsPrompt $prompt) {
    return array_map(
        fn () => Embeddings::fakeEmbedding($prompt->dimensions),
        $prompt->inputs
    );
});
```

Sau khi tạo embeddings, bạn có thể thực hiện các assertions về các prompts đã được nhận:

```php
Embeddings::assertGenerated(function (EmbeddingsPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->dimensions === 1536;
});

Embeddings::assertNotGenerated(
    fn (EmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingGenerated();
```

Đối với các queued embeddings generations, sử dụng các queued assertion methods:

```php
Embeddings::assertQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Laravel')
);

Embeddings::assertNotQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingQueued();
```

Để đảm bảo tất cả các embeddings generations có một fake response tương ứng, bạn có thể sử dụng `preventStrayEmbeddings`. Nếu embeddings được tạo mà không có một fake response được định nghĩa, một exception sẽ được ném:

```php
Embeddings::fake()->preventStrayEmbeddings();
```

<a name="testing-reranking"></a>
### Reranking

Các operations reranking có thể được fake bằng cách gọi method `fake` trên class `Reranking`:

```php
use Laravel\Ai\Reranking;
use Laravel\Ai\Prompts\RerankingPrompt;
use Laravel\Ai\Responses\Data\RankedDocument;

// Automatically generate a fake reranked responses...
Reranking::fake();

// Provide custom responses...
Reranking::fake([
    [
        new RankedDocument(index: 0, document: 'First', score: 0.95),
        new RankedDocument(index: 1, document: 'Second', score: 0.80),
    ],
]);
```

Sau khi reranking, bạn có thể thực hiện các assertions về các operations đã được thực hiện:

```php
Reranking::assertReranked(function (RerankingPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->limit === 5;
});

Reranking::assertNotReranked(
    fn (RerankingPrompt $prompt) => $prompt->contains('Django')
);

Reranking::assertNothingReranked();
```

<a name="testing-files"></a>
### Files

Các file operations có thể được fake bằng cách gọi method `fake` trên class `Files`:

```php
use Laravel\Ai\Files;

Files::fake();
```

Sau khi file operations đã được fake, bạn có thể thực hiện các assertions về các uploads và deletions đã xảy ra:

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

// Store files...
Document::fromString('Hello, Laravel!', mimeType: 'text/plain')
    ->as('hello.txt')
    ->put();

// Make assertions...
Files::assertStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, Laravel!' &&
        $file->mimeType() === 'text/plain';
);

Files::assertNotStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, World!'
);

Files::assertNothingStored();
```

Để asserting đối với file deletions, bạn có thể truyền một file ID:

```php
Files::assertDeleted('file-id');
Files::assertNotDeleted('file-id');
Files::assertNothingDeleted();
```

<a name="testing-vector-stores"></a>
### Vector Stores

Vector store operations có thể được fake bằng cách gọi method `fake` trên class `Stores`. Faking stores sẽ cũng fake [file operations](#files) tự động:

```php
use Laravel\Ai\Stores;

Stores::fake();
```

Sau khi store operations đã được fake, bạn có thể thực hiện các assertions về các stores đã được tạo hoặc xóa:

```php
use Laravel\Ai\Stores;

// Create store...
$store = Stores::create('Knowledge Base');

// Make assertions...
Stores::assertCreated('Knowledge Base');

Stores::assertCreated(fn (string $name, ?string $description) =>
    $name === 'Knowledge Base'
);

Stores::assertNotCreated('Other Store');

Stores::assertNothingCreated();
```

Để asserting đối với store deletions, bạn có thể cung cấp store ID:

```php
Stores::assertDeleted('store_id');
Stores::assertNotDeleted('other_store_id');
Stores::assertNothingDeleted();
```

Để assert files được thêm hoặc xóa từ một store, sử dụng các assertion methods trên một instance `Store`:

```php
Stores::fake();

$store = Stores::get('store_id');

// Add / remove files...
$store->add('added_id');
$store->remove('removed_id');

// Make assertions...
$store->assertAdded('added_id');
$store->assertRemoved('removed_id');

$store->assertNotAdded('other_file_id');
$store->assertNotRemoved('other_file_id');
```

Nếu một file được lưu trữ trong [file storage](#files) của provider và thêm vào một vector store trong cùng một request, bạn có thể không biết file's provider ID. Trong trường hợp này, bạn có thể truyền một closure đến method `assertAdded` để assert đối với nội dung của file được thêm:

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

$store->add(Document::fromString('Hello, World!', 'text/plain')->as('hello.txt'));

$store->assertAdded(fn (StorableFile $file) => $file->name() === 'hello.txt');
$store->assertAdded(fn (StorableFile $file) => $file->content() === 'Hello, World!');
```

<a name="events"></a>
## Events

Laravel AI SDK dispatches một variety của [events](/docs/{{version}}/events), bao gồm:

- `AddingFileToStore`
- `AgentPrompted`
- `AgentStreamed`
- `AudioGenerated`
- `CreatingStore`
- `EmbeddingsGenerated`
- `FileAddedToStore`
- `FileDeleted`
- `FileRemovedFromStore`
- `FileStored`
- `GeneratingAudio`
- `GeneratingEmbeddings`
- `GeneratingImage`
- `GeneratingTranscription`
- `ImageGenerated`
- `InvokingTool`
- `PromptingAgent`
- `RemovingFileFromStore`
- `Reranked`
- `Reranking`
- `StoreCreated`
- `StoringFile`
- `StreamingAgent`
- `ToolInvoked`
- `TranscriptionGenerated`

Bạn có thể lắng nghe bất kỳ events nào để log hoặc lưu trữ thông tin sử dụng AI SDK.

---
title: Laravel AI SDK
description: A unified, expressive API for interacting with AI providers with support for agents, images, audio, embeddings, and more.
url: https://laravel.com/docs/13.x/ai-sdk
tags: [ai]
cssclasses:
  - ai
  - color-purple
color: purple
---

# Laravel AI SDK

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Configuration](#configuration)
    -   [Custom Base URLs](#custom-base-urls)
    -   [Provider Support](#provider-support)
-   [Agents](#agents)
    -   [Prompting](#prompting)
    -   [Conversation Context](#conversation-context)
    -   [Structured Output](#structured-output)
    -   [Attachments](#attachments)
    -   [Streaming](#streaming)
    -   [Broadcasting](#broadcasting)
    -   [Queueing](#queueing)
    -   [Tools](#tools)
    -   [Provider Tools](#provider-tools)
    -   [Middleware](#middleware)
    -   [Anonymous Agents](#anonymous-agents)
    -   [Agent Configuration](#agent-configuration)
    -   [Provider Options](#provider-options)
-   [Images](#images)
-   [Audio (TTS)](#audio)
-   [Transcription (STT)](#transcription)
-   [Embeddings](#embeddings)
    -   [Querying Embeddings](#querying-embeddings)
    -   [Caching Embeddings](#caching-embeddings)
-   [Reranking](#reranking)
-   [Files](#files)
-   [Vector Stores](#vector-stores)
    -   [Adding Files to Stores](#adding-files-to-stores)
-   [Failover](#failover)
-   [Testing](#testing)
    -   [Agents](#testing-agents)
    -   [Images](#testing-images)
    -   [Audio](#testing-audio)
    -   [Transcriptions](#testing-transcriptions)
    -   [Embeddings](#testing-embeddings)
    -   [Reranking](#testing-reranking)
    -   [Files](#testing-files)
    -   [Vector Stores](#testing-vector-stores)
-   [Events](#events)

## [Introduction](#introduction)

The [Laravel AI SDK](https://github.com/laravel/ai) provides a unified, expressive API for interacting with AI providers such as OpenAI, Anthropic, Gemini, and more. With the AI SDK, you can build intelligent agents with tools and structured output, generate images, synthesize and transcribe audio, create vector embeddings, and much more — all using a consistent, Laravel-friendly interface.

## [Installation](#installation)

You can install the Laravel AI SDK via Composer:

```bash
composer require laravel/ai
```

Next, you should publish the AI SDK configuration and migration files using the `vendor:publish` Artisan command:

```bash
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
```

Finally, you should run your application's database migrations. This will create a `agent_conversations` and `agent_conversation_messages` table that the AI SDK uses to power its conversation storage:

```bash
php artisan migrate
```

### [Configuration](#configuration)

You may define your AI provider credentials in your application's `config/ai.php` configuration file or as environment variables in your application's `.env` file:

```env
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

The default models used for text, images, audio, transcription, and embeddings may also be configured in your application's `config/ai.php` configuration file.

### [Custom Base URLs](#custom-base-urls)

By default, the Laravel AI SDK connects directly to each provider's public API endpoint. However, you may need to route requests through a different endpoint - for example, when using a proxy service to centralize API key management, implement rate limiting, or route traffic through a corporate gateway.

You may configure custom base URLs by adding a `url` parameter to your provider configuration:

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

This is useful when routing requests through a proxy service (such as LiteLLM or Azure OpenAI Gateway) or using alternative endpoints.

Custom base URLs are supported for the following providers: OpenAI, Anthropic, Gemini, Groq, Cohere, DeepSeek, xAI, and OpenRouter.

### [Provider Support](#provider-support)

The AI SDK supports a variety of providers across its features. The following table summarizes which providers are available for each feature:

Feature

Providers

Text

OpenAI, Anthropic, Gemini, Azure, Groq, xAI, DeepSeek, Mistral, Ollama

Images

OpenAI, Gemini, xAI

TTS

OpenAI, ElevenLabs

STT

OpenAI, ElevenLabs, Mistral

Embeddings

OpenAI, Gemini, Azure, Cohere, Mistral, Jina, VoyageAI

Reranking

Cohere, Jina

Files

OpenAI, Anthropic, Gemini

The `Laravel\Ai\Enums\Lab` enum may be used to reference providers throughout your code instead of using plain strings:

```php
use Laravel\Ai\Enums\Lab;

Lab::Anthropic;
Lab::OpenAI;
Lab::Gemini;
// ...
```

## [Agents](#agents)

Agents are the fundamental building block for interacting with AI providers in the Laravel AI SDK. Each agent is a dedicated PHP class that encapsulates the instructions, conversation context, tools, and output schema needed to interact with a large language model. Think of an agent as a specialized assistant — a sales coach, a document analyzer, a support bot — that you configure once and prompt as needed throughout your application.

You can create an agent via the `make:agent` Artisan command:

```bash
php artisan make:agent SalesCoach
php artisan make:agent SalesCoach --structured
```

Within the generated agent class, you can define the system prompt / instructions, message context, available tools, and output schema (if applicable):

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

> [!TIP]
> **Laravel Expert Note: Agent Design Patterns**
> When designing Agents in Laravel 13.x, treat them like "Smart Actions". Use Constructor Property Promotion to inject the models or services the agent needs (like `User $user` in the example above). This allows the agent to build its own context via the `messages()` method, keeping your controllers clean.

### [Prompting](#prompting)

To prompt an agent, first create an instance using the `make` method or standard instantiation, then call `prompt`:

```php
$response = (new SalesCoach)
    ->prompt('Analyze this sales transcript...');

return (string) $response;
```

The `make` method resolves your agent from the container, allowing automatic dependency injection. You may also pass arguments to the agent's constructor:

```php
$agent = SalesCoach::make(user: $user);
```

By passing additional arguments to the `prompt` method, you may override the default provider, model, or HTTP timeout when prompting:

```php
$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: Lab::Anthropic,
    model: 'claude-haiku-4-5-20251001',
    timeout: 120,
);
```

### [Conversation Context](#conversation-context)

If your agent implements the `Conversational` interface, you may use the `messages` method to return the previous conversation context, if applicable:

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

#### [Remembering Conversations](#remembering-conversations)

Before using the `RemembersConversations` trait, you should publish and run the AI SDK migrations using the `vendor:publish` Artisan command. These migrations will create the necessary database tables to store conversations.

If you would like Laravel to automatically store and retrieve conversation history for your agent, you may use the `RemembersConversations` trait. This trait provides a simple way to persist conversation messages to the database without manually implementing the `Conversational` interface:

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

To start a new conversation for a user, call the `forUser` method before prompting:

```php
$response = (new SalesCoach)->forUser($user)->prompt('Hello!');

$conversationId = $response->conversationId;
```

The conversation ID is returned on the response and can be stored for future reference, or you can retrieve all of a user's conversations from the `agent_conversations` table directly.

To continue an existing conversation, use the `continue` method:

```php
$response = (new SalesCoach)
    ->continue($conversationId, as: $user)
    ->prompt('Tell me more about that.');
```

When using the `RemembersConversations` trait, previous messages are automatically loaded and included in the conversation context when prompting. New messages (both user and assistant) are automatically stored after each interaction.

### [Structured Output](#structured-output)

If you would like your agent to return structured output, implement the `HasStructuredOutput` interface, which requires that your agent define a `schema` method:

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

When prompting an agent that returns structured output, you can access the returned `StructuredAgentResponse` like an array:

```php
$response = (new SalesCoach)->prompt('Analyze this sales transcript...');

return $response['score'];
```

#### [Nested Objects](#structured-output-nested-objects)

To define nested structured output, use the `object` method with a closure:

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

#### [Arrays of Objects](#structured-output-arrays-of-objects)

If your agent should return a list of structured items, combine the `array` and `object` methods:

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

### [Attachments](#attachments)

When prompting, you may also pass attachments with the prompt to allow the model to inspect images and documents:

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

Likewise, the `Laravel\Ai\Files\Image` class may be used to attach images to a prompt:

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

### [Streaming](#streaming)

You may stream an agent's response by invoking the `stream` method. The returned `StreamableAgentResponse` may be returned from a route to automatically send a streaming response (SSE) to the client:

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)->stream('Analyze this sales transcript...');
});
```

The `then` method may be used to provide a closure that will be invoked when the entire response has been streamed to the client:

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

Alternatively, you may iterate through the streamed events manually:

```php
$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    // ...
}
```

#### [Streaming Using the Vercel AI SDK Protocol](#streaming-using-the-vercel-ai-sdk-protocol)

You may stream the events using the [Vercel AI SDK stream protocol](https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol) by invoking the `usingVercelDataProtocol` method on the streamable response:

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->usingVercelDataProtocol();
});
```

### [Broadcasting](#broadcasting)

You may broadcast streamed events in a few different ways. First, you can simply invoke the `broadcast` or `broadcastNow` method on a streamed event:

```php
use App\Ai\Agents\SalesCoach;
use Illuminate\Broadcasting\Channel;

$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    $event->broadcast(new Channel('channel-name'));
}
```

Or, you can invoke an agent's `broadcastOnQueue` method to queue the agent operation and broadcast the streamed events as they are available:

```php
(new SalesCoach)->broadcastOnQueue(
    'Analyze this sales transcript...'
    new Channel('channel-name'),
);
```

### [Queueing](#queueing)

Using an agent's `queue` method, you may prompt the agent, but allow it to process the response in the background, keeping your application feeling fast and responsive. The `then` and `catch` methods may be used to register closures that will be invoked when a response is available or if an exception occurs:

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

### [Tools](#tools)

Tools may be used to give agents additional functionality that they can utilize while responding to prompts. Tools can be created using the `make:tool` Artisan command:

```bash
php artisan make:tool RandomNumberGenerator
```

The generated tool will be placed in your application's `app/Ai/Tools` directory. Each tool contains a `handle` method that will be invoked by the agent when it needs to utilize the tool:

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

Once you have defined your tool, you may return it from the `tools` method of any of your agents:

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

#### [Similarity Search](#similarity-search)

The `SimilaritySearch` tool allows agents to search for documents similar to a given query using vector embeddings stored in your database. This is useful for retrieval-augmented generation (RAG) when you want to give agents access to search your application's data.

The simplest way to create a similarity search tool is using the `usingModel` method with an Eloquent model that has vector embeddings:

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

The first argument is the Eloquent model class, and the second argument is the column containing the vector embeddings.

You may also provide a minimum similarity threshold between `0.0` and `1.0` and a closure to customize the query:

```php
SimilaritySearch::usingModel(
    model: Document::class,
    column: 'embedding',
    minSimilarity: 0.7,
    limit: 10,
    query: fn ($query) => $query->where('published', true),
),
```

For more control, you may create a similarity search tool with a custom closure that returns the search results:

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

You may customize the tool's description using the `withDescription` method:

```php
SimilaritySearch::usingModel(Document::class, 'embedding')
    ->withDescription('Search the knowledge base for relevant articles.'),
```

### [Provider Tools](#provider-tools)

Provider tools are special tools implemented natively by AI providers, offering capabilities like web searching, URL fetching, and file searching. Unlike regular tools, provider tools are executed by the provider itself rather than your application.

Provider tools can be returned by your agent's `tools` method.

#### [Web Search](#web-search)

The `WebSearch` provider tool allows agents to search the web for real-time information. This is useful for answering questions about current events, recent data, or topics that may have changed since the model's training cutoff.

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

You may configure the web search tool to limit the number of searches or restrict results to specific domains:

```php
(new WebSearch)->max(5)->allow(['laravel.com', 'php.net']),
```

To refine search results based on user location, use the `location` method:

```php
(new WebSearch)->location(
    city: 'New York',
    region: 'NY',
    country: 'US'
);
```

#### [Web Fetch](#web-fetch)

The `WebFetch` provider tool allows agents to fetch and read the contents of web pages. This is useful when you need the agent to analyze specific URLs or retrieve detailed information from known web pages.

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

You may configure the web fetch tool to limit the number of fetches or restrict to specific domains:

```php
(new WebFetch)->max(3)->allow(['docs.laravel.com']),
```

#### [File Search](#file-search)

The `FileSearch` provider tool allows agents to search through [files](#files) stored in [vector stores](#vector-stores). This enables retrieval-augmented generation (RAG) by allowing the agent to search your uploaded documents for relevant information.

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

You may provide multiple vector store IDs to search across multiple stores:

```php
new FileSearch(stores: ['store_1', 'store_2']);
```

If your files have [metadata](#adding-files-to-stores), you may filter the search results by providing a `where` argument. For simple equality filters, pass an array:

```php
new FileSearch(stores: ['store_id'], where: [
    'author' => 'Taylor Otwell',
    'year' => 2026,
]);
```

For more complex filters, you may pass a closure that receives a `FileSearchQuery` instance:

```php
use Laravel\Ai\Providers\Tools\FileSearchQuery;

new FileSearch(stores: ['store_id'], where: fn (FileSearchQuery $query) =>
    $query->where('author', 'Taylor Otwell')
        ->whereNot('status', 'draft')
        ->whereIn('category', ['news', 'updates'])
);
```

### [Middleware](#middleware)

Agents support middleware, allowing you to intercept and modify prompts before they are sent to the provider. Middleware can be created using the `make:agent-middleware` Artisan command:

```bash
php artisan make:agent-middleware LogPrompts
```

The generated middleware will be placed in your application's `app/Ai/Middleware` directory. To add middleware to an agent, implement the `HasMiddleware` interface and define a `middleware` method that returns an array of middleware classes:

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

Each middleware class should define a `handle` method that receives the `AgentPrompt` and a `Closure` to pass the prompt to the next middleware:

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

You may use the `then` method on the response to execute code after the agent has finished processing. This works for both synchronous and streaming responses:

```php
public function handle(AgentPrompt $prompt, Closure $next)
{
    return $next($prompt)->then(function (AgentResponse $response) {
        Log::info('Agent responded', ['text' => $response->text]);
    });
}
```

### [Anonymous Agents](#anonymous-agents)

Sometimes you may want to quickly interact with a model without creating a dedicated agent class. You can create an ad-hoc, anonymous agent using the `agent` function:

```php
use function Laravel\Ai\{agent};

$response = agent(
    instructions: 'You are an expert at software development.',
    messages: [],
    tools: [],
)->prompt('Tell me about Laravel')
```

Anonymous agents may also produce structured output:

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;

use function Laravel\Ai\{agent};

$response = agent(
    schema: fn (JsonSchema $schema) => [
        'number' => $schema->integer()->required(),
    ],
)->prompt('Generate a random number less than 100')
```

### [Agent Configuration](#agent-configuration)

You may configure text generation options for an agent using PHP attributes. The following attributes are available:

-   `MaxSteps`: The maximum number of steps the agent may take when using tools.
-   `MaxTokens`: The maximum number of tokens the model may generate.
-   `Model`: The model the agent should use.
-   `Provider`: The AI provider (or providers for failover) to use for the agent.
-   `Temperature`: The sampling temperature to use for generation (0.0 to 1.0).
-   `Timeout`: The HTTP timeout in seconds for agent requests (default: 60).
-   `UseCheapestModel`: Use the provider's cheapest text model for cost optimization.
-   `UseSmartestModel`: Use the provider's most capable text model for complex tasks.

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Attributes\MaxSteps;
use Laravel\Ai\Attributes\MaxTokens;
use Laravel\Ai\Attributes\Model;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Attributes\Temperature;
use Laravel\Ai\Attributes\Timeout;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
#[Model('claude-haiku-4-5-20251001')]
#[MaxSteps(10)]
#[MaxTokens(4096)]
#[Temperature(0.7)]
#[Timeout(120)]
class SalesCoach implements Agent
{
    use Promptable;

    // ...
}
```

The `UseCheapestModel` and `UseSmartestModel` attributes allow you to automatically select the most cost-effective or most capable model for a given provider without specifying a model name. This is useful when you want to optimize for cost or capability across different providers:

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

### [Provider Options](#provider-options)

If your agent needs to pass provider-specific options (such as OpenAI reasoning effort or penalty settings), implement the `HasProviderOptions` contract and define a `providerOptions` method:

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
            ],
            default => [],
        };
    }
}
```

The `providerOptions` method receives the provider currently being used (`Lab` enum or string), allowing you to return different options per provider. This is especially useful when using [failover](#failover), since each fallback provider can receive its own configuration.

## [Images](#images)

The `Laravel\Ai\Image` class may be used to generate images using the `openai`, `gemini`, or `xai` providers:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

The `square`, `portrait`, and `landscape` methods may be used to control the aspect ratio of the image, while the `quality` method may be used to guide the model on final image quality (`high`, `medium`, `low`). The `timeout` method may be used to specify the HTTP timeout in seconds:

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')
    ->quality('high')
    ->landscape()
    ->timeout(120)
    ->generate();
```

You may attach reference images using the `attachments` method:

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

## [Audio (TTS)](#audio)

The `Laravel\Ai\Audio` class may be used to synthesize speech using the `openai` or `elevenlabs` providers:

```php
use Laravel\Ai\Audio;

$audio = Audio::of('Hello, welcome to Laravel!')->generate();

$content = (string) $audio;
```

You may customize the voice, model, and output format:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Enums\Lab;

$audio = Audio::of('Hello, welcome to Laravel!')
    ->voice('alloy')
    ->model('tts-1')
    ->format('mp3')
    ->provider(Lab::OpenAI)
    ->generate();
```

You may also use the `onProgress` method to stream audio chunks as they are generated:

```php
$audio = Audio::of('Hello, welcome to Laravel!')
    ->onProgress(function (string $chunk) {
        // Stream each chunk to the client...
    })
    ->generate();
```

### [Streaming Audio](#streaming-audio)

When using the `elevenlabs` provider, you may stream audio in real-time:

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Enums\Lab;

$audio = Audio::of('Hello, welcome to Laravel!')
    ->provider(Lab::ElevenLabs)
    ->stream();
```

## [Transcription (STT)](#transcription)

The `Laravel\Ai\Transcription` class may be used to transcribe audio using the `openai`, `elevenlabs`, or `mistral` providers:

```php
use Laravel\Ai\Transcription;

$transcription = Transcription::of('audio.mp3')->generate();

$text = (string) $transcription;
```

You may transcribe audio from a storage disk, a local path, or an uploaded file:

```php
use Laravel\Ai\Transcription;

// From a storage disk...
$transcription = Transcription::fromStorage('audio.mp3')->generate();

// From a local path...
$transcription = Transcription::fromPath('/path/to/audio.mp3')->generate();

// From an uploaded file...
$transcription = Transcription::fromRequest($request->file('audio'))->generate();
```

## [Embeddings](#embeddings)

The `Laravel\Ai\Embeddings` class may be used to create vector embeddings using the `openai`, `gemini`, `azure`, `cohere`, `mistral`, `jina`, or `voyageai` providers:

```php
use Laravel\Ai\Embeddings;

$embeddings = Embeddings::of('The quick brown fox jumps over the lazy dog')->generate();

$vector = $embeddings->toArray();
```

### [Querying Embeddings](#querying-embeddings)

The `Laravel\Ai\Embeddings` class can also be used to query for similar documents using vector embeddings stored in your database:

```php
use App\Models\Document;
use Laravel\Ai\Embeddings;

$results = Embeddings::query('search term')
    ->usingModel(Document::class, 'embedding')
    ->limit(10)
    ->get();
```

The first argument is the Eloquent model class, and the second argument is the column containing the vector embeddings. You may also specify the minimum similarity threshold:

```php
$results = Embeddings::query('search term')
    ->usingModel(Document::class, 'embedding')
    ->minSimilarity(0.7)
    ->limit(10)
    ->get();
```

If your model uses a [[07-database/02-query-builder.md#vector-similarity|vector index]] that has been explicitly created in your database, you do not need to provide the `minSimilarity` option:

```php
$results = Embeddings::query('search term')
    ->usingModel(Document::class, 'embedding')
    ->limit(10)
    ->get();
```

For more complex queries, you may provide a query builder closure:

```php
$results = Embeddings::query('search term')
    ->usingModel(Document::class, 'embedding', query: fn ($query) => $query->where('published', true))
    ->limit(10)
    ->get();
```

### [Caching Embeddings](#caching-embeddings)

To cache embeddings and avoid regenerating them for the same content, use the `cache` method:

```php
use Laravel\Ai\Embeddings;

$embeddings = Embeddings::of('The quick brown fox jumps over the lazy dog')
    ->cache()
    ->generate();
```

By default, embeddings are cached using the `embeddings` cache store. You may specify a different cache driver:

```php
$embeddings = Embeddings::of('The quick brown fox jumps over the lazy dog')
    ->cache(key: 'custom-cache-key')
    ->generate();
```

## [Reranking](#reranking)

The `Laravel\Ai\Reranking` class may be used to rerank search results using the `cohere` or `jina` providers:

```php
use Laravel\Ai\Reranking;

$results = Reranking::of(['doc1', 'doc2', 'doc3'], query: 'search term')
    ->generate();

$reranked = $results->toArray();
```

## [Files](#files)

The `Laravel\Ai\Files` class may be used to upload files to AI providers for processing. Files can be uploaded from a storage disk, a local path, or an uploaded file:

```php
use Laravel\Ai\Files;

// From a storage disk...
$file = Files::Document::fromStorage('transcript.pdf');

// From a local path...
$file = Files\Image::fromPath('/home/laravel/photo.jpg');

// From an uploaded file...
$file = Files::Document::fromRequest($request->file('document'));
```

## [Vector Stores](#vector-stores)

Vector stores allow you to store and search files using semantic search. The `Laravel\Ai\VectorStore` class may be used to manage vector stores:

```php
use Laravel\Ai\VectorStore;

$store = VectorStore::create('My Vector Store');
```

### [Adding Files to Stores](#adding-files-to-stores)

To add files to a vector store, first upload the file, then add it to the store:

```php
use Laravel\Ai\Files;
use Laravel\Ai\VectorStore;

$file = Files\Document::fromStorage('transcript.pdf');

$store = VectorStore::find('store_id');

$store->add($file);
```

You may also provide metadata to attach to the file:

```php
$store->add($file, metadata: [
    'author' => 'Taylor Otwell',
    'year' => 2026,
]);
```

## [Failover](#failover)

The AI SDK supports automatic failover between providers. To enable failover, assign multiple providers using an array when configuring your agent:

```php
use Laravel\Ai\Enums\Lab;

#[Provider([Lab::OpenAI, Lab::Anthropic])]
class SalesCoach implements Agent
{
    use Promptable;

    // ...
}
```

When using failover, providers will be tried in order until one succeeds. The AI SDK will automatically attempt to use the next provider if the current one fails.

## [Testing](#testing)

The Laravel AI SDK provides fake implementations for testing. To get started, publish the AI SDK's test resources:

```bash
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider" --tag="test-assets"
```

### [Testing Agents](#testing-agents)

To test agents, use the `Laravel\Ai\Testing\FakeAgent` class:

```php
use Laravel\Ai\Testing\FakeAgent;

$agent = new FakeAgent([
    'Hello! How can I help you today?',
]);

$response = (new SalesCoach)->prompt('Hello');

$this->assertSame('Hello! How can I help you today?', (string) $response);
```

For agents with tools, the fake will automatically return results when a tool is invoked:

```php
$agent = new FakeAgent([
    'Hello! How can I help you today?',
], tools: [
    'random_number' => '42',
]);

$response = (new SalesCoach)->prompt('Generate a random number');

$this->assertSame('42', (string) $response);
```

For structured output:

```php
$agent = new FakeAgent([
    ['score' => 8],
], structured: true);

$response = (new SalesCoach)->prompt('Analyze this sales transcript...');

$this->assertSame(8, $response['score']);
```

### [Testing Images](#testing-images)

To test image generation, use the `Laravel\Ai\Testing\FakeImage` class:

```php
use Laravel\Ai\Testing\FakeImage;

$image = FakeImage::fake()->generate();

$this->assertStringContainsString('fake-image-data', (string) $image);
```

### [Testing Audio](#testing-audio)

To test audio generation, use the `Laravel\Ai\Testing\FakeAudio` class:

```php
use Laravel\Ai\Testing\FakeAudio;

$audio = FakeAudio::fake()->generate();

$this->assertStringContainsString('fake-audio-data', (string) $audio);
```

### [Testing Transcriptions](#testing-transcriptions)

To test transcription, use the `Laravel\Ai\Testing\FakeTranscription` class:

```php
use Laravel\Ai\Testing\FakeTranscription;

$transcription = FakeTranscription::fake()->generate();

$this->assertSame('This is a transcript.', (string) $transcription);
```

### [Testing Embeddings](#testing-embeddings)

To test embeddings, use the `Laravel\Ai\Testing\FakeEmbeddings` class:

```php
use Laravel\Ai\Testing\FakeEmbeddings;

$embeddings = FakeEmbeddings::fake()->generate();

$this->assertCount(1536, $embeddings->toArray());
```

### [Testing Reranking](#testing-reranking)

To test reranking, use the `Laravel\Ai\Testing\FakeReranking` class:

```php
use Laravel\Ai\Testing\FakeReranking;

$results = FakeReranking::fake()->generate();

$this->assertCount(3, $results->toArray());
```

### [Testing Files](#testing-files)

To test files, use the `Laravel\Ai\Testing\FakeFile` class:

```php
use Laravel\Ai\Testing\FakeFile;

$file = FakeFile::fake();

$id = $file->id();
```

### [Testing Vector Stores](#testing-vector-stores)

To test vector stores, use the `Laravel\Ai\Testing\FakeVectorStore` class:

```php
use Laravel\Ai\Testing\FakeVectorStore;

$store = FakeVectorStore::fake();

$files = $store->files();
```

## [Events](#events)

The AI SDK fires a variety of events during its lifecycle. You may listen to these events to monitor the SDK's behavior:

Event

Description

`Laravel\Ai\Events\AiAgentPrompting`

Fired before an agent is prompted

`Laravel\Ai\Events\AiAgentResponded`

Fired after an agent responds

`Laravel\Ai\Events\AiToolInvoked`

Fired when a tool is invoked

`Laravel\Ai\Events\AiToolReturned`

Fired when a tool returns

`Laravel\Ai\Events\AiProviderChanged`

Fired when the provider changes (failover)
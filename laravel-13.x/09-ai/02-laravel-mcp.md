---
title: Laravel MCP
description: Provides a simple and elegant way for AI clients to interact with your Laravel application through the Model Context Protocol.
url: https://laravel.com/docs/13.x/mcp
tags: [ai]
---

# Laravel MCP

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Publishing Routes](#publishing-routes)
-   [Creating Servers](#creating-servers)
    -   [Server Registration](#server-registration)
    -   [Web Servers](#web-servers)
    -   [Local Servers](#local-servers)
-   [Tools](#tools)
    -   [Creating Tools](#creating-tools)
    -   [Tool Input Schemas](#tool-input-schemas)
    -   [Tool Output Schemas](#tool-output-schemas)
    -   [Validating Tool Arguments](#validating-tool-arguments)
    -   [Tool Dependency Injection](#tool-dependency-injection)
    -   [Tool Annotations](#tool-annotations)
    -   [Conditional Tool Registration](#conditional-tool-registration)
    -   [Tool Responses](#tool-responses)
-   [Prompts](#prompts)
    -   [Creating Prompts](#creating-prompts)
    -   [Prompt Arguments](#prompt-arguments)
    -   [Validating Prompt Arguments](#validating-prompt-arguments)
    -   [Prompt Dependency Injection](#prompt-dependency-injection)
    -   [Conditional Prompt Registration](#conditional-prompt-registration)
    -   [Prompt Responses](#prompt-responses)
-   [Resources](#resources)
    -   [Creating Resources](#creating-resources)
    -   [Resource Templates](#resource-templates)
    -   [Resource URI and MIME Type](#resource-uri-and-mime-type)
    -   [Resource Request](#resource-request)
    -   [Resource Dependency Injection](#resource-dependency-injection)
    -   [Resource Annotations](#resource-annotations)
    -   [Conditional Resource Registration](#conditional-resource-registration)
    -   [Resource Responses](#resource-responses)
-   [Apps](#apps)
    -   [Creating App Resources](#creating-app-resources)
    -   [Rendering Apps From Tools](#rendering-apps-from-tools)
    -   [App Tool Visibility](#app-tool-visibility)
    -   [App Configuration](#app-configuration)
    -   [Building Apps With Boost](#building-apps-with-boost)
-   [Metadata](#metadata)
-   [Authentication](#authentication)
    -   [OAuth 2.1](#oauth)
    -   [Sanctum](#sanctum)
-   [Authorization](#authorization)
-   [Testing Servers](#testing-servers)
    -   [MCP Inspector](#mcp-inspector)
    -   [Unit Tests](#unit-tests)

## [Introduction](#introduction)

[Laravel MCP](https://github.com/laravel/mcp) provides a simple and elegant way for AI clients to interact with your Laravel application through the [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro). It offers an expressive, fluent interface for defining servers, tools, resources, and prompts that enable AI-powered interactions with your application.

## [Installation](#installation)

To get started, install Laravel MCP into your project using the Composer package manager:

```bash
composer require laravel/mcp
```

### [Publishing Routes](#publishing-routes)

After installing Laravel MCP, execute the `vendor:publish` Artisan command to publish the `routes/ai.php` file where you will define your MCP servers:

```bash
php artisan vendor:publish --tag=ai-routes
```

This command creates the `routes/ai.php` file in your application's `routes` directory, which you will use to register your MCP servers.

## [Creating Servers](#creating-servers)

You can create an MCP server using the `make:mcp-server` Artisan command. Servers act as the central communication point that exposes MCP capabilities like tools, resources, and prompts to AI clients:

```bash
php artisan make:mcp-server WeatherServer
```

This command will create a new server class in the `app/Mcp/Servers` directory. The generated server class extends Laravel MCP's base `Laravel\Mcp\Server` class and provides attributes and properties for configuring the server and registering tools, resources, and prompts:

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

### [Server Registration](#server-registration)

Once you've created a server, you must register it in your `routes/ai.php` file to make it accessible. Laravel MCP provides two methods for registering servers: `web` for HTTP-accessible servers and `local` for command-line servers.

### [Web Servers](#web-servers)

Web servers are the most common types of servers and are accessible via HTTP POST requests, making them ideal for remote AI clients or web-based integrations. Register a web server using the `web` method:

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::web('/mcp/weather', WeatherServer::class);
```

Just like normal routes, you may apply middleware to protect your web servers:

```php
Mcp::web('/mcp/weather', WeatherServer::class)
    ->middleware(['throttle:mcp']);
```

### [Local Servers](#local-servers)

Local servers run as Artisan commands, perfect for building local AI assistant integrations like [[02-getting-started/01-installation.md#installing-laravel-boost|Laravel Boost]]. Register a local server using the `local` method:

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::local('weather', WeatherServer::class);
```

Once registered, you should not typically need to manually run the `mcp:start` Artisan command yourself. Instead, configure your MCP client (AI agent) to start the server or use the [MCP Inspector](#mcp-inspector).

## [Tools](#tools)

Tools enable your server to expose functionality that AI clients can call. They allow language models to perform actions, run code, or interact with external systems:

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

### [Creating Tools](#creating-tools)

To create a tool, run the `make:mcp-tool` Artisan command:

```bash
php artisan make:mcp-tool CurrentWeatherTool
```

After creating a tool, register it in your server's `$tools` property:

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

#### [Tool Name, Title, and Description](#tool-name-title-description)

By default, the tool's name and title are derived from the class name. For example, `CurrentWeatherTool` will have a name of `current-weather` and a title of `Current Weather Tool`. You may customize these values using the `Name` and `Title` attributes:

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

Tool descriptions are not automatically generated. You should always provide a meaningful description using the `Description` attribute:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Fetches the current weather forecast for a specified location.')]
class CurrentWeatherTool extends Tool
{
    //
}
```

The description is a critical part of the tool's metadata, as it helps AI models understand when and how to use the tool effectively.

### [Tool Input Schemas](#tool-input-schemas)

Tools can define input schemas to specify what arguments they accept from AI clients. Use Laravel's `Illuminate\Contracts\JsonSchema\JsonSchema` builder to define your tool's input requirements:

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

### [Tool Output Schemas](#tool-output-schemas)

Tools can define [output schemas](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema) to specify the structure of their responses. This enables better integration with AI clients that need parseable tool results. Use the `outputSchema` method to define your tool's output structure:

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

### [Validating Tool Arguments](#validating-tool-arguments)

JSON Schema definitions provide a basic structure for tool arguments, but you may also want to enforce more complex validation rules.

Laravel MCP integrates seamlessly with Laravel's [[04-the-basics/12-validation.md|validation features]]. You may validate incoming tool arguments within your tool's `handle` method:

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

On validation failure, AI clients will act based on the error messages you provide. As such, it is critical to provide clear and actionable error messages:

```php
$validated = $request->validate([
    'location' => ['required','string','max:100'],
    'units' => 'in:celsius,fahrenheit',
],[
    'location.required' => 'You must specify a location to get the weather for. For example, "New York City" or "Tokyo".',
    'units.in' => 'You must specify either "celsius" or "fahrenheit" for the units.',
]);
```

#### [Tool Dependency Injection](#tool-dependency-injection)

The Laravel [[03-architecture-concepts/02-service-container.md|service container]] is used to resolve all tools. As a result, you are able to type-hint any dependencies your tool may need in its constructor. The declared dependencies will automatically be resolved and injected into the tool instance:

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

In addition to constructor injection, you may also type-hint dependencies in your tool's `handle()` method. The service container will automatically resolve and inject the dependencies when the method is called:

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

### [Tool Annotations](#tool-annotations)

You may enhance your tools with [annotations](https://modelcontextprotocol.io/specification/2025-06-18/schema#toolannotations) to provide additional metadata to AI clients. These annotations help AI models understand the tool's behavior and capabilities. Annotations are added to tools via attributes:

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

Available annotations include:

Annotation

Type

Description

`#[IsReadOnly]`

boolean

Indicates the tool does not modify its environment.

`#[IsDestructive]`

boolean

Indicates the tool may perform destructive updates (only meaningful when not read-only).

`#[IsIdempotent]`

boolean

Indicates repeated calls with same arguments have no additional effect (when not read-only).

`#[IsOpenWorld]`

boolean

Indicates the tool may interact with external entities.

Annotation values can be explicitly set using boolean arguments:

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

### [Conditional Tool Registration](#conditional-tool-registration)

You may conditionally register tools at runtime by implementing the `shouldRegister` method in your tool class. This method allows you to determine whether a tool should be available based on application state, configuration, or request parameters:

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

When a tool's `shouldRegister` method returns `false`, it will not appear in the list of available tools and cannot be invoked by AI clients.

### [Tool Responses](#tool-responses)

Tools must return an instance of `Laravel\Mcp\Response`. The Response class provides several convenient methods for creating different types of responses:

For simple text responses, use the `text` method:

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

To indicate an error occurred during tool execution, use the `error` method:

```php
return Response::error('Unable to fetch weather data. Please try again.');
```

To return image or audio content, use the `image` and `audio` methods:

```php
return Response::image(file_get_contents(storage_path('weather/radar.png')), 'image/png');

return Response::audio(file_get_contents(storage_path('weather/alert.mp3')), 'audio/mp3');
```

You may also load image and audio content directly from a Laravel filesystem disk using the `fromStorage` method. The MIME type will be automatically detected from the file:

```php
return Response::fromStorage('weather/radar.png');
```

If needed, you may specify a particular disk or override the MIME type:

```php
return Response::fromStorage('weather/radar.png', disk: 's3');

return Response::fromStorage('weather/radar.png', mimeType: 'image/webp');
```

#### [Multiple Content Responses](#multiple-content-responses)

Tools can return multiple pieces of content by returning an array of `Response` instances:

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

#### [Structured Responses](#structured-responses)

Tools can return [structured content](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#structured-content) using the `structured` method. This provides parseable data for AI clients while maintaining backward compatibility with a JSON-encoded text representation:

```php
return Response::structured([
    'temperature' => 22.5,
    'conditions' => 'Partly cloudy',
    'humidity' => 65,
]);
```

If you need to provide custom text alongside structured content, use the `withStructuredContent` method on the response factory:

```php
return Response::make(
    Response::text('Weather is 22.5°C and sunny')
)->withStructuredContent([
    'temperature' => 22.5,
    'conditions' => 'Sunny',
]);
```

#### [Streaming Responses](#streaming-responses)

For long-running operations or real-time data streaming, tools can return a [generator](https://www.php.net/manual/en/language.generators.overview.php) from their `handle` method. This enables sending intermediate updates to the client before the final response:

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

When using web-based servers, streaming responses automatically open an SSE (Server-Sent Events) stream, sending each yielded message as an event to the client.

## [Prompts](#prompts)

[Prompts](https://modelcontextprotocol.io/specification/2025-06-18/server/prompts) enable your server to share reusable prompt templates that AI clients can use to interact with language models. They provide a standardized way to structure common queries and interactions.

### [Creating Prompts](#creating-prompts)

To create a prompt, run the `make:mcp-prompt` Artisan command:

```bash
php artisan make:mcp-prompt DescribeWeatherPrompt
```

After creating a prompt, register it in your server's `$prompts` property:

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

#### [Prompt Name, Title, and Description](#prompt-name-title-and-description)

By default, the prompt's name and title are derived from the class name. For example, `DescribeWeatherPrompt` will have a name of `describe-weather` and a title of `Describe Weather Prompt`. You may customize these values using the `Name` and `Title` attributes:

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

Prompt descriptions are not automatically generated. You should always provide a meaningful description using the `Description` attribute:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Generates a natural-language explanation of the weather for a given location.')]
class DescribeWeatherPrompt extends Prompt
{
    //
}
```

The description is a critical part of the prompt's metadata, as it helps AI models understand when and how to get the best use out of the prompt.

### [Prompt Arguments](#prompt-arguments)

Prompts can define arguments that allow AI clients to customize the prompt template with specific values. Use the `arguments` method to define what arguments your prompt accepts:

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

### [Validating Prompt Arguments](#validating-prompt-arguments)

Prompt arguments are automatically validated based on their definition, but you may also want to enforce more complex validation rules.

Laravel MCP integrates seamlessly with Laravel's [[04-the-basics/12-validation.md|validation features]]. You may validate incoming prompt arguments within your prompt's `handle` method:

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

On validation failure, AI clients will act based on the error messages you provide. As such, it is critical to provide clear and actionable error messages:

```php
$validated = $request->validate([
    'tone' => ['required','string','max:50'],
],[
    'tone.*' => 'You must specify a tone for the weather description. Examples include "formal", "casual", or "humorous".',
]);
```

### [Prompt Dependency Injection](#prompt-dependency-injection)

The Laravel [[03-architecture-concepts/02-service-container.md|service container]] is used to resolve all prompts. As a result, you are able to type-hint any dependencies your prompt may need in its constructor. The declared dependencies will automatically be resolved and injected into the prompt instance:

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

In addition to constructor injection, you may also type-hint dependencies in your prompt's `handle` method. The service container will automatically resolve and inject the dependencies when the method is called:

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

### [Conditional Prompt Registration](#conditional-prompt-registration)

You may conditionally register prompts at runtime by implementing the `shouldRegister` method in your prompt class. This method allows you to determine whether a prompt should be available based on application state, configuration, or request parameters:

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

When a prompt's `shouldRegister` method returns `false`, it will not appear in the list of available prompts and cannot be invoked by AI clients.

### [Prompt Responses](#prompt-responses)

Prompts may return a single `Laravel\Mcp\Response` or an iterable of `Laravel\Mcp\Response` instances. These responses encapsulate the content that will be sent to the AI client:

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

You can use the `asAssistant()` method to indicate that a response message should be treated as coming from the AI assistant, while regular messages are treated as user input.

## [Resources](#resources)

[Resources](https://modelcontextprotocol.io/specification/2025-06-18/server/resources) enable your server to expose data and content that AI clients can read and use as context when interacting with language models. They provide a way to share static or dynamic information like documentation, configuration, or any data that helps inform AI responses.

## [Creating Resources](#creating-resources)

To create a resource, run the `make:mcp-resource` Artisan command:

```bash
php artisan make:mcp-resource WeatherGuidelinesResource
```

After creating a resource, register it in your server's `$resources` property:

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

#### [Resource Name, Title, and Description](#resource-name-title-and-description)

By default, the resource's name and title are derived from the class name. For example, `WeatherGuidelinesResource` will have a name of `weather-guidelines` and a title of `Weather Guidelines Resource`. You may customize these values using the `Name` and `Title` attributes:

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

Resource descriptions are not automatically generated. You should always provide a meaningful description using the `Description` attribute:

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Comprehensive guidelines for using the Weather API.')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

The description is a critical part of the resource's metadata, as it helps AI models understand when and how to use the resource effectively.

### [Resource Templates](#resource-templates)

[Resource templates](https://modelcontextprotocol.io/specification/2025-06-18/server/resources#resource-templates) enable your server to expose dynamic resources that match URI patterns with variables. Instead of defining a static URI for each resource, you can create a single resource that handles multiple URIs based on a template pattern.

#### [Creating Resource Templates](#creating-resource-templates)

To create a resource template, implement the `HasUriTemplate` interface on your resource class and define a `uriTemplate` method that returns a `UriTemplate` instance:

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

### [Resource URI and MIME Type](#resource-uri-and-mime-type)

By default, the resource's URI is derived from the class name (e.g., `WeatherGuidelinesResource` becomes `weather-guidelines://weather-guidelines`). You may customize this using the `Uri` attribute:

```php
use Laravel\Mcp\Server\Attributes\Uri;

#[Uri('weather://guidelines')]
#[Description('Comprehensive guidelines for using the Weather API.')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

Similarly, you can set the MIME type using the `MimeType` attribute:

```php
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Attributes\MimeType;

#[Description('Access the documentation in PDF format')]
#[MimeType('application/pdf')]
class DocumentationResource extends Resource
{
    //
}
```

### [Resource Request](#resource-request)

The `handle` method receives a `Laravel\Mcp\Request` instance containing all URI template variables extracted from the requested resource URI:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

class UserFileResource extends Resource
{
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

### [Resource Dependency Injection](#resource-dependency-injection)

The Laravel [[03-architecture-concepts/02-service-container.md|service container]] is used to resolve all resources. As a result, you are able to type-hint any dependencies your resource may need in its constructor. The declared dependencies will automatically be resolved and injected into the resource instance:

```php
<?php

namespace App\Mcp\Resources;

use App\Repositories\FileRepository;
use Laravel\Mcp\Server\Resource;

class UserFileResource extends Resource
{
    /**
     * Create a new resource instance.
     */
    public function __construct(
        protected FileRepository $files,
    ) {}

    // ...
}
```

### [Resource Annotations](#resource-annotations)

You may enhance your resources with annotations to provide additional metadata to AI clients. These annotations help AI models understand the resource's behavior and capabilities.

Annotation

Type

Description

`#[IsVolatile]`

boolean

Indicates the resource may change at any time.

Annotation values can be explicitly set using boolean arguments

```php
use Illuminate\Http\File;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Resources\Annotations\IsVolatile;
use Laravel\Mcp\Server\Resource;

#[Description('Current system logs')]
#[IsVolatile(true)]
class LogsResource extends Resource
{
    //
}
```

### [Conditional Resource Registration](#conditional-resource-registration)

You may conditionally register resources at runtime by implementing the `shouldRegister` method in your resource class. This method allows you to determine whether a resource should be available based on application state, configuration, or request parameters:

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
        return $request?->user()?->hasFeature('weather_api') ?? false;
    }
}
```

When a resource's `shouldRegister` method returns `false`, it will not appear in the list of available resources and cannot be accessed by AI clients.

### [Resource Responses](#resource-responses)

Resources must return an instance of `Laravel\Mcp\Response`. The Response class provides several convenient methods for creating different types of responses:

For simple text responses, use the `text` method:

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

/**
 * Handle the resource request.
 */
public function handle(Request $request): Response
{
    // ...

    return Response::text('Here are the weather guidelines...');
}
```

To return binary content, such as images or PDFs, use the `binary` method:

```php
return Response::binary(file_get_contents(storage_path('docs/weather.pdf')), 'application/pdf');
```

You may also load binary content directly from a Laravel filesystem disk using the `fromStorage` method:

```php
return Response::fromStorage('docs/weather.pdf');
```

## [Apps](#apps)

[Apps](https://modelcontextprotocol.io/specification/2025-06-18/server/apps) are a special type of resource that allow your server to expose interactive UI components rendered within AI clients. This is useful when you need to provide rich, interactive experiences that go beyond static content.

### [Creating App Resources](#creating-app-resources)

To create an app resource, run the `make:mcp-app` Artisan command:

```bash
php artisan make:mcp-app WeatherWidget
```

After creating an app, register it in your server's `$resources` property (not `$tools`):

```php
<?php

namespace App\Mcp\Servers;

use App\Mcp\Apps\WeatherWidget;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * The resources registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Resource>>
     */
    protected array $resources = [
        WeatherWidget::class,
    ];
}
```

Within the generated app class, you can define the app's metadata and layout:

```php
<?php

namespace App\Mcp\Apps;

use Laravel\Mcp\Server\Apps\App;
use Laravel\Mcp\Server\Apps\Card;
use Laravel\Mcp\Server\Apps\Heading;
use Laravel\Mcp\Server\Apps\Text;

class WeatherWidget extends App
{
    /**
     * Get the title of the app.
     */
    public function title(): string
    {
        return 'Weather Widget';
    }

    /**
     * Get the description of the app.
     */
    public function description(): string
    {
        return 'A simple weather display widget.';
    }

    /**
     * Get the app's content.
     */
    public function content(): array
    {
        return [
            new Heading('Current Weather'),
            new Text('Sunny, 72°F'),
            new Card(
                new Heading('Forecast'),
                new Text('Partly cloudy with a chance of rain.')
            ),
        ];
    }
}
```

### [Rendering Apps From Tools](#rendering-apps-from-tools)

Within a tool's `handle` method, you may also return an app instead of a plain text response:

```php
<?php

namespace App\Mcp\Tools;

use App\Mcp\Apps\WeatherWidget;
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
        // Get weather data...

        return Response::app(new WeatherWidget());
    }
}
```

### [App Tool Visibility](#app-tool-visibility)

By default, apps are returned as resources (read-only). If you would like your app to also be callable as a tool, add it to your server's `$tools` array:

```php
protected array $tools = [
    CurrentWeatherTool::class,
    WeatherWidget::class, // Will be exposed as both a resource and a tool
];
```

### [App Configuration](#app-configuration)

Apps support several configuration options that you can define in your app class:

```php
<?php

namespace App\Mcp\Apps;

use Laravel\Mcp\Server\Apps\App;
use Laravel\Mcp\Server\Apps\Image;

class WeatherWidget extends App
{
    /**
     * Get the category of the app.
     */
    public function category(): string
    {
        return 'weather';
    }

    /**
     * Determine if the app supports streaming.
     */
    public function supportsStreaming(): bool
    {
        return true;
    }

    /**
     * Get the input schema for the app.
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'location' => $schema->string()->required(),
        ];
    }

    /**
     * Handle the resource request.
     */
    public function handle(Request $request): array
    {
        return [
            // ...
        ];
    }
}
```

### [Building Apps With Boost](#building-apps-with-boost)

To quickly scaffold a complete app with database integration, tools for reading and modifying data, and more, you can use Laravel Boost's built-in `boost:make app` command:

```bash
php artisan boost:make app WeatherWidget
```

This command will scaffold a complete app structure with a database migration, Eloquent model, resource, and tool.

## [Metadata](#metadata)

You may add custom metadata to your server that is transmitted during the handshake. Use the `metadata` method on your server:

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * Get the server metadata.
     */
    public function metadata(): array
    {
        return [
            'server_name' => config('app.name'),
            'version' => '1.0.0',
        ];
    }
}
```

## [Authentication](#authentication)

Laravel MCP supports authenticating incoming requests using OAuth 2.1 or Laravel Sanctum. This ensures that only authorized clients can access your server.

### [OAuth 2.1](#oauth)

To protect your server using OAuth 2.1, assign the `ProtectWithOauth` attribute to your server class:

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server\Attributes\ProtectWithOauth;
use Laravel\Mcp\Server;

#[ProtectWithOauth]
class WeatherServer extends Server
{
    // ...
}
```

When using OAuth, clients must obtain an access token from your authorization server before accessing your MCP server.

### [Sanctum](#sanctum)

To protect your server using Laravel Sanctum, assign the `ProtectWithSanctum` attribute to your server class:

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server\Attributes\ProtectWithSanctum;
use Laravel\Mcp\Server;

#[ProtectWithSanctum]
class WeatherServer extends Server
{
    // ...
}
```

When using Sanctum, incoming requests will be authenticated using the Sanctum guard, allowing clients to access your MCP server using a Sanctum token in the `Authorization` header.

## [Authorization](#authorization)

In addition to authentication, you may also want to implement authorization logic within your server to control what each authenticated user can access. You can override the `isAuthorized` method on your server:

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Request;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * Determine if the request is authorized.
     */
    public function isAuthorized(Request $request): bool
    {
        $user = $request->user();

        return $user && $user->can('access-weather-server');
    }
}
```

## [Testing Servers](#testing-servers)

### [MCP Inspector](#mcp-inspector)

The MCP Inspector is a developer tool that allows you to inspect and test your MCP server locally. To start the MCP Inspector, run the `mcp:inspect` Artisan command:

```bash
php artisan mcp:inspect
```

This command will start a local web server at `http://localhost:5173` that provides an interactive interface for testing your server's capabilities.

### [Unit Tests](#unit-tests)

Laravel MCP provides a `FakeServer` class that allows you to unit test your server's tools, resources, and prompts. To get started, create a test that uses the `FakeServer`:

```php
<?php

use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Testing\FakeServer;
use PHPUnit\Framework\TestCase;

class WeatherServerTest extends TestCase
{
    public function test_tool_returns_weather(): void
    {
        $server = new FakeServer(new WeatherServer);

        $response = $server->tool('current-weather')
            ->with(['location' => 'New York'])
            ->respond();

        $this->assertStringContainsString('New York', $response->content());
    }
}
```

You can also test prompts and resources using the same API:

```php
$server->prompt('describe-weather')
    ->with(['tone' => 'formal'])
    ->respond();

$server->resource('weather-guidelines')
    ->get();
```

The `FakeServer` class provides methods for interacting with each capability:

Method

Description

`tool($name)`

Get a tool by name

`prompt($name)`

Get a prompt by name

`resource($uri)`

Get a resource by URI

`app($name)`

Get an app by name

You may also inspect the capabilities of your server:

```php
$capabilities = $server->capabilities();

$this->assertTrue($capabilities->tools->list->ok);
$this->assertTrue($capabilities->prompts->list->ok);
$this->assertTrue($capabilities->resources->list->ok);
```
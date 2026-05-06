---
title: HTTP Client
description: Laravel HTTP Client documentation - expressive API around Guzzle HTTP client
url: https://laravel.com/docs/13.x/http-client
tags: [logic]
cssclasses:
  - logic
  - color-orange
color: orange
---

# HTTP Client

- [Introduction](#introduction)
- [Making Requests](#making-requests)
- [Concurrent Requests](#concurrent-requests)
- [Macros](#macros)
- [Testing](#testing)
- [Events](#events)

## Introduction

Laravel provides an expressive, minimal API around the Guzzle HTTP client, allowing you to quickly make outgoing HTTP requests to communicate with other web applications. Laravel's wrapper around Guzzle is focused on its most common use cases and a wonderful developer experience.

## Making Requests

To make requests, you may use the `head`, `get`, `post`, `put`, `patch`, and `delete` methods provided by the `Http` facade:

```php
use Illuminate\Support\Facades\Http;

$response = Http::get('http://example.com');
```

The `get` method returns an instance of `Illuminate\Http\Client\Response`, which provides a variety of methods:

```php
$response->body() : string;
$response->json($key = null, $default = null) : mixed;
$response->object() : object;
$response->collect($key = null) : Illuminate\Support\Collection;
$response->status() : int;
$response->successful() : bool;
$response->failed() : bool;
$response->clientError() : bool;
$response->serverError() : bool;
$response->header($header) : string;
$response->headers() : array;
```

### Request Data

Of course, it is common when making `POST`, `PUT`, and `PATCH` requests to send additional data:

```php
use Illuminate\Support\Facades\Http;

$response = Http::post('http://example.com/users', [
    'name' => 'Steve',
    'role' => 'Network Administrator',
]);
```

#### GET Request Query Parameters

When making `GET` requests, you may either append a query string to the URL directly or pass an array:

```php
$response = Http::get('http://example.com/users', [
    'name' => 'Taylor',
    'page' => 1,
]);
```

#### Sending Form URL Encoded Requests

If you would like to send data using `application/x-www-form-urlencoded`:

```php
$response = Http::asForm()->post('http://example.com/users', [
    'name' => 'Sara',
    'role' => 'Privacy Consultant',
]);
```

#### Sending a Raw Request Body

You may use the `withBody` method if you would like to provide a raw request body:

```php
$response = Http::withBody(
    base64_encode($photo), 'image/jpeg'
)->post('http://example.com/photo');
```

#### Multi-Part Requests

If you would like to send files as multi-part requests:

```php
$response = Http::attach(
    'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
)->post('http://example.com/attachments');
```

### Headers

Headers may be added to requests using the `withHeaders` method:

```php
$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

You may use the `accept` method to specify the content type:

```php
$response = Http::accept('application/json')->get('http://example.com/users');

$response = Http::acceptJson()->get('http://example.com/users');
```

### Authentication

You may specify basic and digest authentication credentials:

```php
// Basic authentication...
$response = Http::withBasicAuth('[email protected]', 'secret')->post(/* ... */);

// Digest authentication...
$response = Http::withDigestAuth('[email protected]', 'secret')->post(/* ... */);
```

#### Bearer Tokens

If you would like to quickly add a bearer token:

```php
$response = Http::withToken('token')->post(/* ... */);
```

### Timeout

The `timeout` method may be used to specify the maximum number of seconds to wait for a response:

```php
$response = Http::timeout(3)->get(/* ... */);
```

If the given timeout is exceeded, an instance of `Illuminate\Http\Client\ConnectionException` will be thrown.

You may specify the maximum number of seconds to wait while trying to connect:

```php
$response = Http::connectTimeout(3)->get(/* ... */);
```

### Retries

If you would like the HTTP client to automatically retry the request if a client or server error occurs:

```php
$response = Http::retry(3, 100)->post(/* ... */);
```

The `retry` method accepts the maximum number of times the request should be attempted and the number of milliseconds between attempts.

### Error Handling

Unlike Guzzle's default behavior, Laravel's HTTP client wrapper does not throw exceptions on client or server errors. You may determine if one of these errors was returned using the `successful`, `clientError`, or `serverError` methods:

```php
// Determine if the status code is >= 200 and < 300...
$response->successful();

// Determine if the status code is >= 400...
$response->failed();

// Determine if the response has a 400 level status code...
$response->clientError();

// Determine if the response has a 500 level status code...
$response->serverError();
```

#### Throwing Exceptions

If you have a response instance and would like to throw an exception:

```php
$response = Http::post(/* ... */);

// Throw an exception if a client or server error occurred...
$response->throw();

// Throw an exception if an error occurred and the given condition is true...
$response->throwIf($condition);

return $response['user']['id'];
```

### Guzzle Middleware

Since Laravel's HTTP client is powered by Guzzle, you may take advantage of Guzzle Middleware:

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\RequestInterface;

$response = Http::withRequestMiddleware(
    function (RequestInterface $request) {
        return $request->withHeader('X-Example', 'Value');
    }
)->get('http://example.com');
```

Likewise, you can inspect the incoming HTTP response:

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\ResponseInterface;

$response = Http::withResponseMiddleware(
    function (ResponseInterface $response) {
        $header = $response->getHeader('X-Example');

        // ...

        return $response;
    }
)->get('http://example.com');
```

### Guzzle Options

You may specify additional Guzzle request options:

```php
$response = Http::withOptions([
    'debug' => true,
])->get('http://example.com/users');
```

## Concurrent Requests

### Request Pooling

Sometimes, you may wish to make multiple HTTP requests concurrently:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->get('http://localhost/first'),
    $pool->get('http://localhost/second'),
    $pool->get('http://localhost/third'),
]);

return $responses[0]->ok() &&
       $responses[1]->ok() &&
       $responses[2]->ok();
```

As you can see, each response instance can be accessed based on the order it was added to the pool. If you wish, you can name the requests using the `as` method:

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->as('first')->get('http://localhost/first'),
    $pool->as('second')->get('http://localhost/second'),
    $pool->as('third')->get('http://localhost/third'),
]);

return $responses['first']->ok();
```

The maximum concurrency of the request pool may be controlled:

```php
$responses = Http::pool(fn (Pool $pool) => [
    // ...
], concurrency: 5);
```

### Request Batching

Another way of working with concurrent requests is to use the `batch` method:

```php
use Illuminate\Http\Client\Batch;
use Illuminate\Http\Client\Response;
use Illuminate\Support\Facades\Http;

$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
    $batch->get('http://localhost/third'),
])->then(function (Batch $batch, array $results) {
    // All requests completed successfully...
})->catch(function (Batch $batch, int|string $key, Response $response) {
    // Batch request failure detected...
})->finally(function (Batch $batch, array $results) {
    // The batch has finished executing...
})->send();
```

## Macros

The Laravel HTTP client allows you to define "macros":

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::macro('github', function () {
        return Http::withHeaders([
            'X-Example' => 'example',
        ])->baseUrl('https://github.com');
    });
}
```

Once your macro has been configured, you may invoke it:

```php
$response = Http::github()->get('/');
```

## Testing

### Faking Responses

The `Http` facade's `fake` method allows you to instruct the HTTP client to return stubbed responses when requests are made:

```php
use Illuminate\Support\Facades\Http;

Http::fake();

$response = Http::post(/* ... */);
```

#### Faking Specific URLs

Alternatively, you may pass an array to the `fake` method:

```php
Http::fake([
    'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),
    'google.com/*' => Http::response('Hello World', 200, $headers),
]);
```

Any requests made to URLs that have not been faked will actually be executed.

#### Faking Response Sequences

Sometimes you may need to specify that a single URL should return a series of fake responses:

```php
Http::fake([
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->pushStatus(404),
]);
```

### Inspecting Requests

When faking responses, you may occasionally wish to inspect the requests the client receives:

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::fake();

Http::withHeaders([
    'X-First' => 'foo',
])->post('http://example.com/users', [
    'name' => 'Taylor',
    'role' => 'Developer',
]);

Http::assertSent(function (Request $request) {
    return $request->hasHeader('X-First', 'foo') &&
           $request->url() == 'http://example.com/users' &&
           $request['name'] == 'Taylor' &&
           $request['role'] == 'Developer';
});
```

You may use the `assertSentCount` method to assert how many requests were "sent":

```php
Http::fake();
Http::assertSentCount(5);
```

Or, you may use the `assertNothingSent` method:

```php
Http::fake();
Http::assertNothingSent();
```

## Events

The HTTP client dispatches several events during the request lifecycle:
- `Illuminate\Http\Client\Events\RequestSending` - Before a request is sent
- `Illuminate\Http\Client\Events\ResponseReceived` - When a response is received
- `Illuminate\Http\Client\Events\ConnectionFailed` - When a connection fails
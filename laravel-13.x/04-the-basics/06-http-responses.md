---
title: HTTP Responses
description: Create and customize HTTP responses in Laravel
url: https://laravel.com/docs/13.x/responses
tags: [framework]
---

# HTTP Responses

-   [Creating Responses](#creating-responses)
    -   [Attaching Headers to Responses](#attaching-headers-to-responses)
    -   [Attaching Cookies to Responses](#attaching-cookies-to-responses)
    -   [Cookies and Encryption](#cookies-and-encryption)
-   [Redirects](#redirects)
    -   [Redirecting to Named Routes](#redirecting-named-routes)
    -   [Redirecting to Controller Actions](#redirecting-controller-actions)
    -   [Redirecting to External Domains](#redirecting-external-domains)
    -   [Redirecting With Flashed Session Data](#redirecting-with-flashed-session-data)
-   [Other Response Types](#other-response-types)
    -   [View Responses](#view-responses)
    -   [JSON Responses](#json-responses)
    -   [File Downloads](#file-downloads)
    -   [File Responses](#file-responses)
-   [Streamed Responses](#streamed-responses)
    -   [Consuming Streamed Responses](#consuming-streamed-responses)
    -   [Streamed JSON Responses](#streamed-json-responses)
    -   [Event Streams (SSE)](#event-streams)
    -   [Streamed Downloads](#streamed-downloads)
-   [Response Macros](#response-macros)

## [Creating Responses](#creating-responses)

#### [Strings and Arrays](#strings-arrays)

All routes and controllers should return a response to be sent back to the user's browser. Laravel provides several different ways to return responses. The most basic response is returning a string from a route or controller:

```
1Route::get('/', function () {2    return 'Hello World';3});
Route::get('/', function () {
    return 'Hello World';
});
```

In addition to returning strings from your routes and controllers, you may also return arrays. The framework will automatically convert the array into a JSON response:

```
1Route::get('/', function () {2    return [1, 2, 3];3});
Route::get('/', function () {
    return [1, 2, 3];
});
```

#### [Response Objects](#response-objects)

Typically, you won't just be returning simple strings or arrays from your route actions. Instead, you will be returning full `Illuminate\Http\Response` instances or [[04-the-basics/07-views.md|views]]:

```
1Route::get('/home', function () {2    return response('Hello World', 200)3        ->header('Content-Type', 'text/plain');4});
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

#### [Eloquent Models and Collections](#eloquent-models-and-collections)

You may also return [[08-eloquent-orm/01-eloquent-getting-started.md|Eloquent ORM]] models and collections directly from your routes and controllers:

```
1use App\Models\User;2 3Route::get('/user/{user}', function (User $user) {4    return $user;5});
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

### [Attaching Headers to Responses](#attaching-headers-to-responses)

Keep in mind that most response methods are chainable, allowing for the fluent construction of response instances:

```
1return response($content)2    ->header('Content-Type', $type)3    ->header('X-Header-One', 'Header Value')4    ->header('X-Header-Two', 'Header Value');
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

Or, you may use the `withHeaders` method to specify an array of headers to be added to the response:

```
1return response($content)2    ->withHeaders([3        'Content-Type' => $type,4        'X-Header-One' => 'Header Value',5        'X-Header-Two' => 'Header Value',6    ]);
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

You can remove specific headers from an outgoing response using the `withoutHeader` method:

```
1return response($content)->withoutHeader('X-Debug');2 3return response($content)->withoutHeader(['X-Debug', 'X-Powered-By']);
return response($content)->withoutHeader('X-Debug');

return response($content)->withoutHeader(['X-Debug', 'X-Powered-By']);
```

#### [Cache Control Middleware](#cache-control-middleware)

Laravel includes a `cache.headers` middleware, which may be used to quickly set the `Cache-Control` header for a group of routes:

```
1Route::middleware('cache.headers:public;max_age=30;s_maxage=300;stale_while_revalidate=600;etag')->group(function () {2    Route::get('/privacy', function () {3        // ...4    });5 6    Route::get('/terms', function () {7        // ...8    });9});
Route::middleware('cache.headers:public;max_age=30;s_maxage=300;stale_while_revalidate=600;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

### [Attaching Cookies to Responses](#attaching-cookies-to-responses)

You may attach a cookie to an outgoing `Illuminate\Http\Response` instance using the `cookie` method:

```
1return response('Hello World')->cookie(2    'name', 'value', $minutes3);
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

#### [Generating Cookie Instances](#generating-cookie-instances)

If you would like to generate a `Symfony\Component\HttpFoundation\Cookie` instance that can be attached to a response instance at a later time, you may use the global `cookie` helper:

```
1$cookie = cookie('name', 'value', $minutes);2 3return response('Hello World')->cookie($cookie);
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

#### [Expiring Cookies Early](#expiring-cookies-early)

You may remove a cookie by expiring it via the `withoutCookie` method of an outgoing response:

```
1return response('Hello World')->withoutCookie('name');
return response('Hello World')->withoutCookie('name');
```

### [Cookies and Encryption](#cookies-and-encryption)

By default, all cookies generated by Laravel are encrypted and signed so that they can't be modified or read by the client. If you would like to disable encryption for a subset of cookies generated by your application, you may use the `encryptCookies` method in your application's `bootstrap/app.php` file:

```
1->withMiddleware(function (Middleware $middleware): void {2    $middleware->encryptCookies(except: [3        'cookie_name',4    ]);5})
->withMiddleware(function (Middleware $middleware): void {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

## [Redirects](#redirects)

Redirect responses are instances of the `Illuminate\Http\RedirectResponse` class. There are several ways to generate a `RedirectResponse` instance:

```
1Route::get('/dashboard', function () {2    return redirect('/home/dashboard');3});
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

Sometimes you may wish to redirect the user to their previous location:

```
1Route::post('/user/profile', function () {2    // Validate the request...3 4    return back()->withInput();5});
Route::post('/user/profile', function () {
    // Validate the request...

    return back()->withInput();
});
```

### [Redirecting to Named Routes](#redirecting-named-routes)

When you call the `redirect` helper with no parameters, an instance of `Illuminate\Routing\Redirector` is returned:

```
1return redirect()->route('login');
return redirect()->route('login');
```

If your route has parameters, you may pass them as the second argument to the `route` method:

```
1// For a route with the following URI: /profile/{id}2 3return redirect()->route('profile', ['id' => 1]);
// For a route with the following URI: /profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

#### [Populating Parameters via Eloquent Models](#populating-parameters-via-eloquent-models)

If you are redirecting to a route with an "ID" parameter that is being populated from an Eloquent model, you may pass the model itself:

```
1// For a route with the following URI: /profile/{id}2 3return redirect()->route('profile', [$user]);
// For a route with the following URI: /profile/{id}

return redirect()->route('profile', [$user]);
```

### [Redirecting to Controller Actions](#redirecting-controller-actions)

You may also generate redirects to [[04-the-basics/04-controllers.md|controller actions]]:

```
1use App\Http\Controllers\UserController;2 3return redirect()->action([UserController::class, 'index']);
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

If your controller route requires parameters, you may pass them as the second argument to the `action` method:

```
1return redirect()->action(2    [UserController::class, 'profile'], ['id' => 1]3);
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

### [Redirecting to External Domains](#redirecting-external-domains)

Sometimes you may need to redirect to a domain outside of your application:

```
1return redirect()->away('https://www.google.com');
return redirect()->away('https://www.google.com');
```

### [Redirecting With Flashed Session Data](#redirecting-with-flashed-session-data)

Redirecting to a new URL and [[04-the-basics/11-http-session.md#flash-data|flashing data to the session]] are usually done at the same time:

```
1Route::post('/user/profile', function () {2    // ...3 4    return redirect('/dashboard')->with('status', 'Profile updated!');5});
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

After the user is redirected, you may display the flashed message from the [[04-the-basics/11-http-session.md|session]]:

```
1@if (session('status'))2    <div class="alert alert-success">3        {{ session('status') }}4    </div>5@endif
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

#### [Redirecting With Input](#redirecting-with-input)

You may use the `withInput` method provided by the `RedirectResponse` instance to flash the current request's input data to the session before redirecting:

```
1return back()->withInput();
return back()->withInput();
```

## [Other Response Types](#other-response-types)

The `response` helper may be used to generate other types of response instances.

### [View Responses](#view-responses)

If you need control over the response's status and headers but also need to return a [[04-the-basics/07-views.md|view]] as the response's content, you should use the `view` method:

```
1return response()2    ->view('hello', $data, 200)3    ->header('Content-Type', $type);
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

### [JSON Responses](#json-responses)

The `json` method will automatically set the `Content-Type` header to `application/json`:

```
1return response()->json([2    'name' => 'Abigail',3    'state' => 'CA',4]);
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

### [File Downloads](#file-downloads)

The `download` method may be used to generate a response that forces the user's browser to download the file at the given path:

```
1return response()->download($pathToFile);2 3return response()->download($pathToFile, $name, $headers);
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

### [File Responses](#file-responses)

The `file` method may be used to display a file, such as an image or PDF, directly in the user's browser:

```
1return response()->file($pathToFile);2 3return response()->file($pathToFile, $headers);
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

## [Streamed Responses](#streamed-responses)

By streaming data to the client as it is generated, you can significantly reduce memory usage and improve performance:

```
1Route::get('/stream', function () {2    return response()->stream(function (): void {3        foreach (['developer', 'admin'] as $string) {4            echo $string;5            ob_flush();6            flush();7            sleep(2); // Simulate delay between chunks...8        }9    }, 200, ['X-Accel-Buffering' => 'no']);10});
Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (['developer', 'admin'] as $string) {
            echo $string;
            ob_flush();
            flush();
            sleep(2); // Simulate delay between chunks...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

### [Streamed JSON Responses](#streamed-json-responses)

If you need to stream JSON data incrementally, you may utilize the `streamJson` method:

```
1use App\Models\User;2 3Route::get('/users.json', function () {4    return response()->streamJson([5        'users' => User::cursor(),6    ]);7});
use App\Models\User;

Route::get('/users.json', function () {
    return response()->streamJson([
        'users' => User::cursor(),
    ]);
});
```

### [Event Streams (SSE)](#event-streams)

The `eventStream` method may be used to return a server-sent events (SSE) streamed response:

```
1Route::get('/chat', function () {2    return response()->eventStream(function () {3        $stream = OpenAI::client()->chat()->createStreamed(...);4 5        foreach ($stream as $response) {6            yield $response->choices[0];7        }8    });9});
Route::get('/chat', function () {
    return response()->eventStream(function () {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

### [Streamed Downloads](#streamed-downloads)

Sometimes you may wish to turn the string response of a given operation into a downloadable response:

```
1use App\Services\GitHub;2 3return response()->streamDownload(function () {4    echo GitHub::api('repo')5        ->contents()6        ->readme('laravel', 'laravel')['contents'];7}, 'laravel-readme.md');
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
        ->contents()
        ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

## [Response Macros](#response-macros)

If you would like to define a custom response that you can re-use in a variety of your routes and controllers, you may use the `macro` method on the `Response` facade:

```
 1<?php 2  3namespace App\Providers; 4  5use Illuminate\Support\Facades\Response; 6use Illuminate\Support\ServiceProvider; 7  8class AppServiceProvider extends ServiceProvider 9{10    /**11     * Bootstrap any application services.12     */13    public function boot(): void14    {15        Response::macro('caps', function (string $value) {16            return Response::make(strtoupper($value));17        });18    }19}
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

The `macro` function accepts a name as its first argument and a closure as its second argument:

```
1return response()->caps('foo');
return response()->caps('foo');
```
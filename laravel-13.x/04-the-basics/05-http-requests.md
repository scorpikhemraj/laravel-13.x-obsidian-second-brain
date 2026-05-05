---
title: HTTP Requests
description: Access and handle incoming HTTP request data in Laravel
url: https://laravel.com/docs/13.x/requests
tags: [framework]
---

# HTTP Requests

-   [Introduction](#introduction)
-   [Interacting With The Request](#interacting-with-the-request)
    -   [Accessing the Request](#accessing-the-request)
    -   [Request Path, Host, and Method](#request-path-and-method)
    -   [Request Headers](#request-headers)
    -   [Request IP Address](#request-ip-address)
    -   [Content Negotiation](#content-negotiation)
    -   [PSR-7 Requests](#psr7-requests)
-   [Input](#input)
    -   [Retrieving Input](#retrieving-input)
    -   [Input Presence](#input-presence)
    -   [Merging Additional Input](#merging-additional-input)
    -   [Old Input](#old-input)
    -   [Cookies](#cookies)
    -   [Input Trimming and Normalization](#input-trimming-and-normalization)
-   [Files](#files)
    -   [Retrieving Uploaded Files](#retrieving-uploaded-files)
    -   [Storing Uploaded Files](#storing-uploaded-files)
-   [Configuring Trusted Proxies](#configuring-trusted-proxies)
-   [Configuring Trusted Hosts](#configuring-trusted-hosts)

## [Introduction](#introduction)

Laravel's `Illuminate\Http\Request` class provides an object-oriented way to interact with the current HTTP request being handled by your application as well as retrieve the input, cookies, and files that were submitted with the request.

## [Interacting With The Request](#interacting-with-the-request)

### [Accessing the Request](#accessing-the-request)

To obtain an instance of the current HTTP request via dependency injection, you should type-hint the `Illuminate\Http\Request` class on your route closure or controller method:

```
 1<?php 2  3namespace App\Http\Controllers; 4  5use Illuminate\Http\RedirectResponse; 6use Illuminate\Http\Request; 7  8class UserController extends Controller 9{10    /**11     * Store a new user.12     */13    public function store(Request $request): RedirectResponse14    {15        $name = $request->input('name');16 17        // Store the user...18 19        return redirect('/users');20    }21}
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Store the user...

        return redirect('/users');
    }
}
```

As mentioned, you may also type-hint the `Illuminate\Http\Request` class on a route closure:

```
1use Illuminate\Http\Request;2 3Route::get('/', function (Request $request) {4    // ...5});
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

#### [Dependency Injection and Route Parameters](#dependency-injection-route-parameters)

If your controller method is also expecting input from a route parameter you should list your route parameters after your other dependencies:

```
 1<?php 2  3namespace App\Http\Controllers; 4  5use Illuminate\Http\RedirectResponse; 6use Illuminate\Http\Request; 7  8class UserController extends Controller 9{10    /**11     * Update the specified user.12     */13    public function update(Request $request, string $id): RedirectResponse14    {15        // Update the user...16 17        return redirect('/users');18    }19}
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...

        return redirect('/users');
    }
}
```

### [Request Path, Host, and Method](#request-path-and-method)

The `Illuminate\Http\Request` instance provides a variety of methods for examining the incoming HTTP request.

#### [Retrieving the Request Path](#retrieving-the-request-path)

The `path` method returns the request's path information:

```
1$uri = $request->path();
$uri = $request->path();
```

#### [Inspecting the Request Path / Route](#inspecting-the-request-path)

The `is` method allows you to verify that the incoming request path matches a given pattern:

```
1if ($request->is('admin/*')) {2    // ...3}
if ($request->is('admin/*')) {
    // ...
}
```

Using the `routeIs` method, you may determine if the incoming request has matched a [[04-the-basics/01-routing.md#named-routes|named route]]:

```
1if ($request->routeIs('admin.*')) {2    // ...3}
if ($request->routeIs('admin.*')) {
    // ...
}
```

#### [Retrieving the Request URL](#retrieving-the-request-url)

To retrieve the full URL for the incoming request you may use the `url` or `fullUrl` methods:

```
1$url = $request->url();2 3$urlWithQueryString = $request->fullUrl();
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```

If you would like to append query string data to the current URL, you may call the `fullUrlWithQuery` method:

```
1$request->fullUrlWithQuery(['type' => 'phone']);
$request->fullUrlWithQuery(['type' => 'phone']);
```

#### [Retrieving the Request Host](#retrieving-the-request-host)

You may retrieve the "host" of the incoming request via the `host`, `httpHost`, and `schemeAndHttpHost` methods:

```
1// http://localhost:80002$request->host(); // localhost3$request->httpHost(); // localhost:80004$request->schemeAndHttpHost(); // http://localhost:8000
// http://localhost:8000
$request->host(); // localhost
$request->httpHost(); // localhost:8000
$request->schemeAndHttpHost(); // http://localhost:8000
```

#### [Retrieving the Request Method](#retrieving-the-request-method)

The `method` method will return the HTTP verb for the request:

```
1$method = $request->method();2 3if ($request->isMethod('post')) {4    // ...5}
$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}
```

### [Request Headers](#request-headers)

You may retrieve a request header from the `Illuminate\Http\Request` instance using the `header` method:

```
1$value = $request->header('X-Header-Name');2 3$value = $request->header('X-Header-Name', 'default');
$value = $request->header('X-Header-Name');

$value = $request->header('X-Header-Name', 'default');
```

The `hasHeader` method may be used to determine if the request contains a given header:

```
1if ($request->hasHeader('X-Header-Name')) {2    // ...3}
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

For convenience, the `bearerToken` method may be used to retrieve a bearer token from the `Authorization` header:

```
1$token = $request->bearerToken();
$token = $request->bearerToken();
```

### [Request IP Address](#request-ip-address)

The `ip` method may be used to retrieve the IP address of the client that made the request to your application:

```
1$ipAddress = $request->ip();
$ipAddress = $request->ip();
```

If you would like to retrieve an array of IP addresses, including all of the client IP addresses that were forwarded by proxies, you may use the `ips` method:

```
1$ipAddresses = $request->ips();
$ipAddresses = $request->ips();
```

### [Content Negotiation](#content-negotiation)

Laravel provides several methods for inspecting the incoming request's requested content types via the `Accept` header:

```
1$contentTypes = $request->getAcceptableContentTypes();
$contentTypes = $request->getAcceptableContentTypes();
```

The `accepts` method accepts an array of content types and returns `true` if any of the content types are accepted by the request:

```
1if ($request->accepts(['text/html', 'application/json'])) {2    // ...3}
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

You may use the `prefers` method to determine which content type out of a given array of content types is most preferred by the request:

```
1$preferred = $request->prefers(['text/html', 'application/json']);
$preferred = $request->prefers(['text/html', 'application/json']);
```

Since many applications only serve HTML or JSON, you may use the `expectsJson` method to quickly determine if the incoming request expects a JSON response:

```
1if ($request->expectsJson()) {2    // ...3}
if ($request->expectsJson()) {
    // ...
}
```

### [PSR-7 Requests](#psr7-requests)

The [PSR-7 standard](https://www.php-fig.org/psr/psr-7/) specifies interfaces for HTTP messages. If you would like to obtain an instance of a PSR-7 request instead of a Laravel request, you will first need to install a few libraries:

```
1composer require symfony/psr-http-message-bridge2composer require nyholm/psr7
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

## [Input](#input)

### [Retrieving Input](#retrieving-input)

#### [Retrieving All Input Data](#retrieving-all-input-data)

You may retrieve all of the incoming request's input data as an `array` using the `all` method:

```
1$input = $request->all();
$input = $request->all();
```

Using the `collect` method, you may retrieve all of the incoming request's input data as a [[05-digging-deeper/04-collections.md|collection]]:

```
1$input = $request->collect();
$input = $request->collect();
```

#### [Retrieving an Input Value](#retrieving-an-input-value)

Using a few simple methods, you may access all of the user input from your `Illuminate\Http\Request` instance:

```
1$name = $request->input('name');
$name = $request->input('name');
```

You may pass a default value as the second argument to the `input` method:

```
1$name = $request->input('name', 'Sally');
$name = $request->input('name', 'Sally');
```

When working with forms that contain array inputs, use "dot" notation to access the arrays:

```
1$name = $request->input('products.0.name');2 3$names = $request->input('products.*.name');
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

#### [Retrieving Input From the Query String](#retrieving-input-from-the-query-string)

While the `input` method retrieves values from the entire request payload (including the query string), the `query` method will only retrieve values from the query string:

```
1$name = $request->query('name');
$name = $request->query('name');
```

#### [Retrieving JSON Input Values](#retrieving-json-input-values)

When sending JSON requests to your application, you may access the JSON data via the `input` method:

```
1$name = $request->input('user.name');
$name = $request->input('user.name');
```

#### [Retrieving Stringable Input Values](#retrieving-stringable-input-values)

Instead of retrieving the request's input data as a primitive `string`, you may use the `string` method:

```
1$name = $request->string('name')->trim();
$name = $request->string('name')->trim();
```

#### [Retrieving Integer Input Values](#retrieving-integer-input-values)

To retrieve input values as integers, you may use the `integer` method:

```
1$perPage = $request->integer('per_page');
$perPage = $request->integer('per_page');
```

#### [Retrieving Boolean Input Values](#retrieving-boolean-input-values)

When dealing with HTML elements like checkboxes, you may use the `boolean` method to retrieve these values as booleans:

```
1$archived = $request->boolean('archived');
$archived = $request->boolean('archived');
```

#### [Retrieving Array Input Values](#retrieving-array-input-values)

Input values containing arrays may be retrieved using the `array` method:

```
1$versions = $request->array('versions');
$versions = $request->array('versions');
```

#### [Retrieving Date Input Values](#retrieving-date-input-values)

For convenience, input values containing dates / times may be retrieved as Carbon instances using the `date` method:

```
1$birthday = $request->date('birthday');
$birthday = $request->date('birthday');
```

#### [Retrieving Enum Input Values](#retrieving-enum-input-values)

Input values that correspond to [PHP enums](https://www.php.net/manual/en/language.types.enumerations.php) may also be retrieved from the request:

```
1use App\Enums\Status;2 3$status = $request->enum('status', Status::class);
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```

#### [Retrieving Input via Dynamic Properties](#retrieving-input-via-dynamic-properties)

You may also access user input using dynamic properties on the `Illuminate\Http\Request` instance:

```
1$name = $request->name;
$name = $request->name;
```

#### [Retrieving a Portion of the Input Data](#retrieving-a-portion-of-the-input-data)

If you need to retrieve a subset of the input data, you may use the `only` and `except` methods:

```
1$input = $request->only(['username', 'password']);2 3$input = $request->except(['credit_card']);
$input = $request->only(['username', 'password']);

$input = $request->except(['credit_card']);
```

### [Input Presence](#input-presence)

You may use the `has` method to determine if a value is present on the request:

```
1if ($request->has('name')) {2    // ...3}
if ($request->has('name')) {
    // ...
}
```

When given an array, the `has` method will determine if all of the specified values are present:

```
1if ($request->has(['name', 'email'])) {2    // ...3}
if ($request->has(['name', 'email'])) {
    // ...
}
```

The `hasAny` method returns `true` if any of the specified values are present:

```
1if ($request->hasAny(['name', 'email'])) {2    // ...3}
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

If you would like to determine if a value is present on the request and is not an empty string, you may use the `filled` method:

```
1if ($request->filled('name')) {2    // ...3}
if ($request->filled('name')) {
    // ...
}
```

If you would like to determine if a value is missing from the request or is an empty string, you may use the `isNotFilled` method:

```
1if ($request->isNotFilled('name')) {2    // ...3}
if ($request->isNotFilled('name')) {
    // ...
}
```

### [Merging Additional Input](#merging-additional-input)

Sometimes you may need to manually merge additional input into the request's existing input data. To accomplish this, you may use the `merge` method:

```
1$request->merge(['votes' => 0]);
$request->merge(['votes' => 0]);
```

The `mergeIfMissing` method may be used to merge input into the request if the corresponding keys do not already exist:

```
1$request->mergeIfMissing(['votes' => 0]);
$request->mergeIfMissing(['votes' => 0]);
```

### [Old Input](#old-input)

Laravel allows you to keep input from one request during the next request. This feature is particularly useful for re-populating forms after detecting validation errors.

#### [Flashing Input to the Session](#flashing-input-to-the-session)

The `flash` method on the `Illuminate\Http\Request` class will flash the current input to the [[04-the-basics/11-http-session.md|session]]:

```
1$request->flash();
$request->flash();
```

You may also use the `flashOnly` and `flashExcept` methods to flash a subset of the request data to the session:

```
1$request->flashOnly(['username', 'email']);2 3$request->flashExcept('password');
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

#### [Flashing Input Then Redirecting](#flashing-input-then-redirecting)

Since you often will want to flash input to the session and then redirect to the previous page, you may easily chain input flashing onto a redirect using the `withInput` method:

```
1return redirect('/form')->withInput();2 3return redirect()->route('user.create')->withInput();4 5return redirect('/form')->withInput(6    $request->except('password')7);
return redirect('/form')->withInput();

return redirect()->route('user.create')->withInput();

return redirect('/form')->withInput(
    $request->except('password')
);
```

#### [Retrieving Old Input](#retrieving-old-input)

To retrieve flashed input from the previous request, invoke the `old` method on an instance of `Illuminate\Http\Request`:

```
1$username = $request->old('username');
$username = $request->old('username');
```

Laravel also provides a global `old` helper:

```
1<input type="text" name="username" value="{{ old('username') }}">
<input type="text" name="username" value="{{ old('username') }}">
```

### [Cookies](#cookies)

#### [Retrieving Cookies From Requests](#retrieving-cookies-from-requests)

All cookies created by the Laravel framework are encrypted and signed. To retrieve a cookie value from the request, use the `cookie` method:

```
1$value = $request->cookie('name');
$value = $request->cookie('name');
```

## [Input Trimming and Normalization](#input-trimming-and-normalization)

By default, Laravel includes the `Illuminate\Foundation\Http\Middleware\TrimStrings` and `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull` middleware in your application's global middleware stack.

#### Disabling Input Normalization

If you would like to disable this behavior for all requests, you may remove the two middleware from your application's middleware stack:

```
1use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;2use Illuminate\Foundation\Http\Middleware\TrimStrings;3 4->withMiddleware(function (Middleware $middleware): void {5    $middleware->remove([6        ConvertEmptyStringsToNull::class,7        TrimStrings::class,8    ]);9})
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;
use Illuminate\Foundation\Http\Middleware\TrimStrings;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->remove([
        ConvertEmptyStringsToNull::class,
        TrimStrings::class,
    ]);
})
```

## [Files](#files)

### [Retrieving Uploaded Files](#retrieving-uploaded-files)

You may retrieve uploaded files from an `Illuminate\Http\Request` instance using the `file` method or using dynamic properties:

```
1$file = $request->file('photo');2 3$file = $request->photo;
$file = $request->file('photo');

$file = $request->photo;
```

You may determine if a file is present on the request using the `hasFile` method:

```
1if ($request->hasFile('photo')) {2    // ...3}
if ($request->hasFile('photo')) {
    // ...
}
```

#### [Validating Successful Uploads](#validating-successful-uploads)

In addition to checking if the file is present, you may verify that there were no problems uploading the file via the `isValid` method:

```
1if ($request->file('photo')->isValid()) {2    // ...3}
if ($request->file('photo')->isValid()) {
    // ...
}
```

#### [File Paths and Extensions](#file-paths-extensions)

The `UploadedFile` class also contains methods for accessing the file's fully-qualified path and its extension:

```
1$path = $request->photo->path();2 3$extension = $request->photo->extension();
$path = $request->photo->path();

$extension = $request->photo->extension();
```

### [Storing Uploaded Files](#storing-uploaded-files)

To store an uploaded file, you will typically use one of your configured [[05-digging-deeper/09-file-storage.md|filesystems]]:

```
1$path = $request->photo->store('images');2 3$path = $request->photo->store('images', 's3');
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

If you do not want a filename to be automatically generated, you may use the `storeAs` method:

```
1$path = $request->photo->storeAs('images', 'filename.jpg');2 3$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

## [Configuring Trusted Proxies](#configuring-trusted-proxies)

When running your applications behind a load balancer that terminates TLS / SSL certificates, you may enable the `Illuminate\Http\Middleware\TrustProxies` middleware:

```
1->withMiddleware(function (Middleware $middleware): void {2    $middleware->trustProxies(at: [3        '192.168.1.1',4        '10.0.0.0/8',5    ]);6})
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: [
        '192.168.1.1',
        '10.0.0.0/8',
    ]);
})
```

#### [Trusting All Proxies](#trusting-all-proxies)

If you are using Amazon AWS or another "cloud" load balancer provider, you may use `*` to trust all proxies:

```
1->withMiddleware(function (Middleware $middleware): void {2    $middleware->trustProxies(at: '*');3})
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: '*');
})
```

## [Configuring Trusted Hosts](#configuring-trusted-hosts)

By default, Laravel will respond to all requests it receives regardless of the content of the HTTP request's `Host` header. You may do so by enabling the `Illuminate\Http\Middleware\TrustHosts` middleware:

```
1->withMiddleware(function (Middleware $middleware): void {2    $middleware->trustHosts(at: ['^laravel\.test$']);3})
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$']);
})
```
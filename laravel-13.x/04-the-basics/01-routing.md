---
title: Routing
description: Define application routes and handle URL navigation in Laravel
url: https://laravel.com/docs/13.x/routing
tags: [framework]
cssclasses:
  - framework
  - color-blue
color: blue
---

# Routing

-   [Basic Routing](#basic-routing)
    -   [The Default Route Files](#the-default-route-files)
    -   [Redirect Routes](#redirect-routes)
    -   [View Routes](#view-routes)
    -   [Listing Your Routes](#listing-your-routes)
    -   [Routing Customization](#routing-customization)
-   [Route Parameters](#route-parameters)
    -   [Required Parameters](#required-parameters)
    -   [Optional Parameters](#parameters-optional-parameters)
    -   [Regular Expression Constraints](#parameters-regular-expression-constraints)
-   [Named Routes](#named-routes)
-   [Route Groups](#route-groups)
    -   [Middleware](#route-group-middleware)
    -   [Controllers](#route-group-controllers)
    -   [Subdomain Routing](#route-group-subdomain-routing)
    -   [Route Prefixes](#route-group-prefixes)
    -   [Route Name Prefixes](#route-group-name-prefixes)
-   [Route Model Binding](#route-model-binding)
    -   [Implicit Binding](#implicit-binding)
    -   [Implicit Enum Binding](#implicit-enum-binding)
    -   [Explicit Binding](#explicit-binding)
-   [Fallback Routes](#fallback-routes)
-   [Rate Limiting](#rate-limiting)
    -   [Defining Rate Limiters](#defining-rate-limiters)
    -   [Attaching Rate Limiters to Routes](#attaching-rate-limiters-to-routes)
-   [Form Method Spoofing](#form-method-spoofing)
-   [Accessing the Current Route](#accessing-the-current-route)
-   [Cross-Origin Resource Sharing (CORS)](#cors)
-   [Route Caching](#route-caching)

## [Basic Routing](#basic-routing)

The most basic Laravel routes accept a URI and a closure, providing a very simple and expressive method of defining routes and behavior without complicated routing configuration files:

```
1use Illuminate\Support\Facades\Route;2 3Route::get('/greeting', function () {4    return 'Hello World';5});
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

### [The Default Route Files](#the-default-route-files)

All Laravel routes are defined in your route files, which are located in the `routes` directory. These files are automatically loaded by Laravel using the configuration specified in your application's `bootstrap/app.php` file. The `routes/web.php` file defines routes that are for your web interface. These routes are assigned the `web` [[04-the-basics/02-middleware.md#laravels-default-middleware-groups|middleware group]], which provides features like session state and CSRF protection.

For most applications, you will begin by defining routes in your `routes/web.php` file. The routes defined in `routes/web.php` may be accessed by entering the defined route's URL in your browser. For example, you may access the following route by navigating to `http://example.com/user` in your browser:

```
1use App\Http\Controllers\UserController;2 3Route::get('/user', [UserController::class, 'index']);
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

#### [API Routes](#api-routes)

If your application will also offer a stateless API, you may enable API routing using the `install:api` Artisan command:

```
1php artisan install:api
php artisan install:api
```

The `install:api` command installs [Laravel Sanctum](/docs/13.x/sanctum), which provides a robust, yet simple API token authentication guard which can be used to authenticate third-party API consumers, SPAs, or mobile applications. In addition, the `install:api` command creates the `routes/api.php` file:

```
1Route::get('/user', function (Request $request) {2    return $request->user();3})->middleware('auth:sanctum');
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

Of course, you are free to omit the `auth:sanctum` middleware on routes that should be publicly accessible.

The routes in `routes/api.php` are stateless and are assigned to the `api` [[04-the-basics/02-middleware.md#laravels-default-middleware-groups|middleware group]]. Additionally, the `/api` URI prefix is automatically applied to these routes, so you do not need to manually apply it to every route in the file. You may change the prefix by modifying your application's `bootstrap/app.php` file:

```
1->withRouting(2    api: __DIR__.'/../routes/api.php',3    apiPrefix: 'api/admin',4    // ...5)
->withRouting(
    api: __DIR__.'/../routes/api.php',
    apiPrefix: 'api/admin',
    // ...
)
```

#### [Available Router Methods](#available-router-methods)

The router allows you to register routes that respond to any HTTP verb:

```
1Route::get($uri, $callback);2Route::post($uri, $callback);3Route::put($uri, $callback);4Route::patch($uri, $callback);5Route::delete($uri, $callback);6Route::options($uri, $callback);
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the `match` method. Or, you may even register a route that responds to all HTTP verbs using the `any` method:

```
1Route::match(['get', 'post'], '/', function () {2    // ...3});4 5Route::any('/', function () {6    // ...7});
Route::match(['get', 'post'], '/', function () {
    // ...
});

Route::any('/', function () {
    // ...
});
```

When defining multiple routes that share the same URI, routes using the `get`, `post`, `put`, `patch`, `delete`, and `options` methods should be defined before routes using the `any`, `match`, and `redirect` methods. This ensures the incoming request is matched with the correct route.

#### [Dependency Injection](#dependency-injection)

You may type-hint any dependencies required by your route in your route's callback signature. The declared dependencies will automatically be resolved and injected into the callback by the Laravel [[03-architecture-concepts/02-service-container.md|service container]]. For example, you may type-hint the `Illuminate\Http\Request` class to have the current HTTP request automatically injected into your route callback:

```
1use Illuminate\Http\Request;2 3Route::get('/users', function (Request $request) {4    // ...5});
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // ...
});
```

#### [CSRF Protection](#csrf-protection)

Remember, any HTML forms pointing to `POST`, `PUT`, `PATCH`, or `DELETE` routes that are defined in the `web` routes file should include a CSRF token field. Otherwise, the request will be rejected. You can read more about CSRF protection in the [[04-the-basics/03-csrf-protection.md|CSRF documentation]]:

```
1<form method="POST" action="/profile">2    @csrf3    ...4</form>
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

### [Redirect Routes](#redirect-routes)

If you are defining a route that redirects to another URI, you may use the `Route::redirect` method. This method provides a convenient shortcut so that you do not have to define a full route or controller for performing a simple redirect:

```
1Route::redirect('/here', '/there');
Route::redirect('/here', '/there');
```

By default, `Route::redirect` returns a `302` status code. You may customize the status code using the optional third parameter:

```
1Route::redirect('/here', '/there', 301);
Route::redirect('/here', '/there', 301);
```

Or, you may use the `Route::permanentRedirect` method to return a `301` status code:

```
1Route::permanentRedirect('/here', '/there');
Route::permanentRedirect('/here', '/there');
```

When using route parameters in redirect routes, the following parameters are reserved by Laravel and cannot be used: `destination` and `status`.

### [View Routes](#view-routes)

If your route only needs to return a [[04-the-basics/07-views.md|view]], you may use the `Route::view` method. Like the `redirect` method, this method provides a simple shortcut so that you do not have to define a full route or controller. The `view` method accepts a URI as its first argument and a view name as its second argument. In addition, you may provide an array of data to pass to the view as an optional third argument:

```
1Route::view('/welcome', 'welcome');2 3Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

When using route parameters in view routes, the following parameters are reserved by Laravel and cannot be used: `view`, `data`, `status`, and `headers`.

### [Listing Your Routes](#listing-your-routes)

The `route:list` Artisan command can easily provide an overview of all of the routes that are defined by your application:

```
1php artisan route:list
php artisan route:list
```

By default, the route middleware that are assigned to each route will not be displayed in the `route:list` output; however, you can instruct Laravel to display the route middleware and middleware group names by adding the `-v` option to the command:

```
1php artisan route:list -v2 3# Expand middleware groups...4php artisan route:list -vv
php artisan route:list -v

# Expand middleware groups...
php artisan route:list -vv
```

You may also instruct Laravel to only show routes that begin with a given URI:

```
1php artisan route:list --path=api
php artisan route:list --path=api
```

In addition, you may instruct Laravel to hide any routes that are defined by third-party packages by providing the `--except-vendor` option when executing the `route:list` command:

```
1php artisan route:list --except-vendor
php artisan route:list --except-vendor
```

Likewise, you may also instruct Laravel to only show routes that are defined by third-party packages by providing the `--only-vendor` option when executing the `route:list` command:

```
1php artisan route:list --only-vendor
php artisan route:list --only-vendor
```

### [Routing Customization](#routing-customization)

By default, your application's routes are configured and loaded by the `bootstrap/app.php` file:

```
 1<?php 2  3use Illuminate\Foundation\Application; 4  5return Application::configure(basePath: dirname(__DIR__)) 6    ->withRouting( 7        web: __DIR__.'/../routes/web.php', 8        commands: __DIR__.'/../routes/console.php', 9        health: '/up',10    )->create();
<?php

use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```

However, sometimes you may want to define an entirely new file to contain a subset of your application's routes. To accomplish this, you may provide a `then` closure to the `withRouting` method. Within this closure, you may register any additional routes that are necessary for your application:

```
 1use Illuminate\Support\Facades\Route; 2  3->withRouting( 4     web: __DIR__.'/../routes/web.php', 5     commands: __DIR__.'/../routes/console.php', 6     health: '/up', 7     then: function () { 8         Route::middleware('api') 9             ->prefix('webhooks')10             ->name('webhooks.')11             ->group(base_path('routes/webhooks.php'));12     },13)
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

Or, you may even take complete control over route registration by providing a `using` closure to the `withRouting` method. When this argument is passed, no HTTP routes will be registered by the framework and you are responsible for manually registering all routes:

```
 1use Illuminate\Support\Facades\Route; 2  3->withRouting( 4     commands: __DIR__.'/../routes/console.php', 5     using: function () { 6         Route::middleware('api') 7             ->prefix('api') 8             ->group(base_path('routes/api.php')); 9 10         Route::middleware('web')11             ->group(base_path('routes/web.php'));12     },13)
use Illuminate\Support\Facades\Route;

->withRouting(
    commands: __DIR__.'/../routes/console.php',
    using: function () {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->group(base_path('routes/web.php'));
    },
)
```

## [Route Parameters](#route-parameters)

### [Required Parameters](#required-parameters)

Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:

```
1Route::get('/user/{id}', function (string $id) {2    return 'User '.$id;3});
Route::get('/user/{id}', function (string $id) {
    return 'User '.$id;
});
```

You may define as many route parameters as required by your route:

```
1Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {2    // ...3});
Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```

Route parameters are always encased within `{}` braces and should consist of alphabetic characters. Underscores (`_`) are also acceptable within route parameter names. Route parameters are injected into route callbacks / controllers based on their order - the names of the route callback / controller arguments do not matter.

#### [Parameters and Dependency Injection](#parameters-and-dependency-injection)

If your route has dependencies that you would like the Laravel service container to automatically inject into your route's callback, you should list your route parameters after your dependencies:

```
1use Illuminate\Http\Request;2 3Route::get('/user/{id}', function (Request $request, string $id) {4    return 'User '.$id;5});
use Illuminate\Http\Request;

Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User '.$id;
});
```

### [Optional Parameters](#parameters-optional-parameters)

Occasionally you may need to specify a route parameter that may not always be present in the URI. You may do so by placing a `?` mark after the parameter name. Make sure to give the route's corresponding variable a default value:

```
1Route::get('/user/{name?}', function (?string $name = null) {2    return $name;3});4 5Route::get('/user/{name?}', function (?string $name = 'John') {6    return $name;7});
Route::get('/user/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

### [Regular Expression Constraints](#parameters-regular-expression-constraints)

You may constrain the format of your route parameters using the `where` method on a route instance. The `where` method accepts the name of the parameter and a regular expression defining how the parameter should be constrained:

```
 1Route::get('/user/{name}', function (string $name) { 2     // ... 3})->where('name', '[A-Za-z]+'); 4  5Route::get('/user/{id}', function (string $id) { 6     // ... 7})->where('id', '[0-9]+'); 8  9Route::get('/user/{id}/{name}', function (string $id, string $name) {10     // ...11})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

For convenience, some commonly used regular expression patterns have helper methods that allow you to quickly add pattern constraints to your routes:

```
 1Route::get('/user/{id}/{name}', function (string $id, string $name) { 2     // ... 3})->whereNumber('id')->whereAlpha('name'); 4  5Route::get('/user/{name}', function (string $name) { 6     // ... 7})->whereAlphaNumeric('name'); 8  9Route::get('/user/{id}', function (string $id) {10     // ...11})->whereUuid('id');12 13Route::get('/user/{id}', function (string $id) {14     // ...15})->whereUlid('id');16 17Route::get('/category/{category}', function (string $category) {18     // ...19})->whereIn('category', ['movie', 'song', 'painting']);20 21Route::get('/category/{category}', function (string $category) {22     // ...23})->whereIn('category', CategoryEnum::cases());
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', CategoryEnum::cases());
```

If the incoming request does not match the route pattern constraints, a 404 HTTP response will be returned.

#### [Global Constraints](#parameters-global-constraints)

If you would like a route parameter to always be constrained by a given regular expression, you may use the `pattern` method. You should define these patterns in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
1use Illuminate\Support\Facades\Route;2 3/**4 * Bootstrap any application services.5 */6public function boot(): void7{8    Route::pattern('id', '[0-9]+');9}
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}
```

Once the pattern has been defined, it is automatically applied to all routes using that parameter name:

```
1Route::get('/user/{id}', function (string $id) {2    // Only executed if {id} is numeric...3});
Route::get('/user/{id}', function (string $id) {
    // Only executed if {id} is numeric...
});
```

#### [Encoded Forward Slashes](#parameters-encoded-forward-slashes)

The Laravel routing component allows all characters except `/` to be present within route parameter values. You must explicitly allow `/` to be part of your placeholder using a `where` condition regular expression:

```
1Route::get('/search/{search}', function (string $search) {2    return $search;3})->where('search', '.*');
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

Encoded forward slashes are only supported within the last route segment.

## [Named Routes](#named-routes)

Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the `name` method onto the route definition:

```
1Route::get('/user/profile', function () {2    // ...3})->name('profile');
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

You may also specify route names for controller actions:

```
1Route::get(2    '/user/profile',3    [UserProfileController::class, 'show']4)->name('profile');
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```

Route names should always be unique.

#### [Generating URLs to Named Routes](#generating-urls-to-named-routes)

Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via Laravel's `route` and `redirect` helper functions:

```
1// Generating URLs...2$url = route('profile');3 4// Generating Redirects...5return redirect()->route('profile');6 7return to_route('profile');
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');

return to_route('profile');
```

If the named route defines parameters, you may pass the parameters as the second argument to the `route` function. The given parameters will automatically be inserted into the generated URL in their correct positions:

```
1Route::get('/user/{id}/profile', function (string $id) {2    // ...3})->name('profile');4 5$url = route('profile', ['id' => 1]);
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1]);
```

If you pass additional parameters in the array, those key / value pairs will automatically be added to the generated URL's query string:

```
1Route::get('/user/{id}/profile', function (string $id) {2    // ...3})->name('profile');4 5$url = route('profile', ['id' => 1, 'photos' => 'yes']);6 7// http://example.com/user/1/profile?photos=yes
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// http://example.com/user/1/profile?photos=yes
```

Sometimes, you may wish to specify request-wide default values for URL parameters, such as the current locale. To accomplish this, you may use the [[04-the-basics/10-url-generation.md#default-values|URL::defaults method]].

#### [Inspecting the Current Route](#inspecting-the-current-route)

If you would like to determine if the current request was routed to a given named route, you may use the `named` method on a Route instance. For example, you may check the current route name from a route middleware:

```
 1use Closure; 2use Illuminate\Http\Request; 3use Symfony\Component\HttpFoundation\Response; 4  5/** 6 * Handle an incoming request. 7 * 8 * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next 9 */10public function handle(Request $request, Closure $next): Response11{12    if ($request->route()->named('profile')) {13        // ...14    }15 16    return $next($request);17}
use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * Handle an incoming request.
 *
 * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
 */
public function handle(Request $request, Closure $next): Response
{
    if ($request->route()->named('profile')) {
        // ...
    }

    return $next($request);
}
```

## [Route Groups](#route-groups)

Route groups allow you to share route attributes, such as middleware, across a large number of routes without needing to define those attributes on each individual route.

Nested groups attempt to intelligently "merge" attributes with their parent group. Middleware and `where` conditions are merged while names and prefixes are appended. Namespace delimiters and slashes in URI prefixes are automatically added where appropriate.

### [Middleware](#route-group-middleware)

To assign [[04-the-basics/02-middleware.md|middleware]] to all routes within a group, you may use the `middleware` method before defining the group. Middleware are executed in the order they are listed in the array:

```
1Route::middleware(['first', 'second'])->group(function () {2    Route::get('/', function () {3        // Uses first & second middleware...4    });5 6    Route::get('/user/profile', function () {7        // Uses first & second middleware...8    });9});
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware...
    });

    Route::get('/user/profile', function () {
        // Uses first & second middleware...
    });
});
```

### [Controllers](#route-group-controllers)

If a group of routes all utilize the same [[04-the-basics/04-controllers.md|controller]], you may use the `controller` method to define the common controller for all of the routes within the group. Then, when defining the routes, you only need to provide the controller method that they invoke:

```
1use App\Http\Controllers\OrderController;2 3Route::controller(OrderController::class)->group(function () {4    Route::get('/orders/{id}', 'show');5    Route::post('/orders', 'store');6});
use App\Http\Controllers\OrderController;

Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```

### [Subdomain Routing](#route-group-subdomain-routing)

Route groups may also be used to handle subdomain routing. Subdomains may be assigned route parameters just like route URIs, allowing you to capture a portion of the subdomain for usage in your route or controller. The subdomain may be specified by calling the `domain` method before defining the group:

```
1Route::domain('{account}.example.com')->group(function () {2    Route::get('/user/{id}', function (string $account, string $id) {3        // ...4    });5});
Route::domain('{account}.example.com')->group(function () {
    Route::get('/user/{id}', function (string $account, string $id) {
        // ...
    });
});
```

### [Route Prefixes](#route-group-prefixes)

The `prefix` method may be used to prefix each route in the group with a given URI. For example, you may want to prefix all route URIs within the group with `admin`:

```
1Route::prefix('admin')->group(function () {2    Route::get('/users', function () {3        // Matches The "/admin/users" URL4    });5});
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // Matches The "/admin/users" URL
    });
});
```

### [Route Name Prefixes](#route-group-name-prefixes)

The `name` method may be used to prefix each route name in the group with a given string. For example, you may want to prefix the names of all of the routes in the group with `admin`. The given string is prefixed to the route name exactly as it is specified, so we will be sure to provide the trailing `.` character in the prefix:

```
1Route::name('admin.')->group(function () {2    Route::get('/users', function () {3        // Route assigned name "admin.users"...4    })->name('users');5});
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```

## [Route Model Binding](#route-model-binding)

When injecting a model ID to a route or controller action, you will often query the database to retrieve the model that corresponds to that ID. Laravel route model binding provides a convenient way to automatically inject the model instances directly into your routes. For example, instead of injecting a user's ID, you can inject the entire `User` model instance that matches the given ID.

### [Implicit Binding](#implicit-binding)

Laravel automatically resolves Eloquent models defined in routes or controller actions whose type-hinted variable names match a route segment name. For example:

```
1use App\Models\User;2 3Route::get('/users/{user}', function (User $user) {4    return $user->email;5});
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

Since the `$user` variable is type-hinted as the `App\Models\User` Eloquent model and the variable name matches the `{user}` URI segment, Laravel will automatically inject the model instance that has an ID matching the corresponding value from the request URI. If a matching model instance is not found in the database, a 404 HTTP response will automatically be generated.

Of course, implicit binding is also possible when using controller methods. Again, note the `{user}` URI segment matches the `$user` variable in the controller which contains an `App\Models\User` type-hint:

```
 1use App\Http\Controllers\UserController; 2use App\Models\User; 3  4// Route definition... 5Route::get('/users/{user}', [UserController::class, 'show']); 6  7// Controller method definition... 8public function show(User $user) 9{10    return view('user.profile', ['user' => $user]);11}
use App\Http\Controllers\UserController;
use App\Models\User;

// Route definition...
Route::get('/users/{user}', [UserController::class, 'show']);

// Controller method definition...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

#### [Soft Deleted Models](#implicit-soft-deleted-models)

Typically, implicit model binding will not retrieve models that have been [[08-eloquent-orm/01-eloquent-getting-started.md#soft-deleting|soft deleted]]. However, you may instruct the implicit binding to retrieve these models by chaining the `withTrashed` method onto your route's definition:

```
1use App\Models\User;2 3Route::get('/users/{user}', function (User $user) {4    return $user->email;5})->withTrashed();
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

#### [Customizing the Key](#customizing-the-default-key-name)

Sometimes you may wish to resolve Eloquent models using a column other than `id`. To do so, you may specify the column in the route parameter definition:

```
1use App\Models\Post;2 3Route::get('/posts/{post:slug}', function (Post $post) {4    return $post;5});
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

If you would like model binding to always use a database column other than `id` when retrieving a given model class, you may override the `getRouteKeyName` method on the Eloquent model:

```
1/**2 * Get the route key for the model.3 */4public function getRouteKeyName(): string5{6    return 'slug';7}
/**
 * Get the route key for the model.
 */
public function getRouteKeyName(): string
{
    return 'slug';
}
```

#### [Custom Keys and Scoping](#implicit-model-binding-scoping)

When implicitly binding multiple Eloquent models in a single route definition, you may wish to scope the second Eloquent model such that it must be a child of the previous Eloquent model. For example, consider this route definition that retrieves a blog post by slug for a specific user:

```
1use App\Models\Post;2use App\Models\User;3 4Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {5    return $post;6});
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

When using a custom keyed implicit binding as a nested route parameter, Laravel will automatically scope the query to retrieve the nested model by its parent using conventions to guess the relationship name on the parent. In this case, it will be assumed that the `User` model has a relationship named `posts` (the plural form of the route parameter name) which can be used to retrieve the `Post` model.

If you wish, you may instruct Laravel to scope "child" bindings even when a custom key is not provided. To do so, you may invoke the `scopeBindings` method when defining your route:

```
1use App\Models\Post;2use App\Models\User;3 4Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {5    return $post;6})->scopeBindings();
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

Or, you may instruct an entire group of route definitions to use scoped bindings:

```
1Route::scopeBindings()->group(function () {2    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {3        return $post;4    });5});
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```

Similarly, you may explicitly instruct Laravel to not scope bindings by invoking the `withoutScopedBindings` method:

```
1Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {2    return $post;3})->withoutScopedBindings();
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

#### [Customizing Missing Model Behavior](#customizing-missing-model-behavior)

Typically, a 404 HTTP response will be generated if an implicitly bound model is not found. However, you may customize this behavior by calling the `missing` method when defining your route. The `missing` method accepts a closure that will be invoked if an implicitly bound model cannot be found:

```
1use App\Http\Controllers\LocationsController;2use Illuminate\Http\Request;3use Illuminate\Support\Facades\Redirect;4 5Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])6    ->name('locations.view')7    ->missing(function (Request $request) {8        return Redirect::route('locations.index');9    });
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

### [Implicit Enum Binding](#implicit-enum-binding)

PHP 8.1 introduced support for [Enums](https://www.php.net/manual/en/language.enumerations.backed.php). To complement this feature, Laravel allows you to type-hint a [string-backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php) on your route definition and Laravel will only invoke the route if that route segment corresponds to a valid Enum value. Otherwise, a 404 HTTP response will be returned automatically. For example, given the following Enum:

```
1<?php2 3namespace App\Enums;4 5enum Category: string6{7    case Fruits = 'fruits';8    case People = 'people';9}
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

You may define a route that will only be invoked if the `{category}` route segment is `fruits` or `people`. Otherwise, Laravel will return a 404 HTTP response:

```
1use App\Enums\Category;2use Illuminate\Support\Facades\Route;3 4Route::get('/categories/{category}', function (Category $category) {5    return $category->value;6});
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

### [Explicit Binding](#explicit-binding)

You are not required to use Laravel's implicit, convention based model resolution in order to use model binding. You can also explicitly define how route parameters correspond to models. To register an explicit binding, use the router's `model` method to specify the class for a given parameter. You should define your explicit model bindings at the beginning of the `boot` method of your `AppServiceProvider` class:

```
 1use App\Models\User; 2use Illuminate\Support\Facades\Route; 3  4/** 5 * Bootstrap any application services. 6 */ 7public function boot(): void 8{ 9    Route::model('user', User::class);10}
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::model('user', User::class);
}
```

Next, define a route that contains a `{user}` parameter:

```
1use App\Models\User;2 3Route::get('/users/{user}', function (User $user) {4    // ...5});
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    // ...
});
```

Since we have bound all `{user}` parameters to the `App\Models\User` model, an instance of that class will be injected into the route. So, for example, a request to `users/1` will inject the `User` instance from the database which has an ID of `1`.

If a matching model instance is not found in the database, a 404 HTTP response will be automatically generated.

#### [Customizing the Resolution Logic](#customizing-the-resolution-logic)

If you wish to define your own model binding resolution logic, you may use the `Route::bind` method. The closure you pass to the `bind` method will receive the value of the URI segment and should return the instance of the class that should be injected into the route. Again, this customization should take place in the `boot` method of your application's `AppServiceProvider`:

```
 1use App\Models\User; 2use Illuminate\Support\Facades\Route; 3  4/** 5 * Bootstrap any application services. 6 */ 7public function boot(): void 8{ 9    Route::bind('user', function (string $value) {10        return User::where('name', $value)->firstOrFail();11    });12}
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });
}
```

Alternatively, you may override the `resolveRouteBinding` method on your Eloquent model. This method will receive the value of the URI segment and should return the instance of the class that should be injected into the route:

```
 1/** 2 * Retrieve the model for a bound value. 3 * 4 * @param  mixed  $value 5 * @param  string|null  $field 6 * @return \Illuminate\Database\Eloquent\Model|null 7 */ 8public function resolveRouteBinding($value, $field = null) 9{10    return $this->where('name', $value)->firstOrFail();11}
/**
 * Retrieve the model for a bound value.
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

If a route is utilizing [implicit binding scoping](#implicit-model-binding-scoping), the `resolveChildRouteBinding` method will be used to resolve the child binding of the parent model:

```
 1/** 2 * Retrieve the child model for a bound value. 3 * 4 * @param  string  $childType 5 * @param  mixed  $value 6 * @param  string|null  $field 7 * @return \Illuminate\Database\Eloquent\Model|null 8 */ 9public function resolveChildRouteBinding($childType, $value, $field)10{11    return parent::resolveChildRouteBinding($childType, $value, $field);12}
/**
 * Retrieve the child model for a bound value.
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

## [Fallback Routes](#fallback-routes)

Using the `Route::fallback` method, you may define a route that will be executed when no other route matches the incoming request. Typically, unhandled requests will automatically render a "404" page via your application's exception handler. However, since you would typically define the `fallback` route within your `routes/web.php` file, all middleware in the `web` middleware group will apply to the route. You are free to add additional middleware to this route as needed:

```
1Route::fallback(function () {2    // ...3});
Route::fallback(function () {
    // ...
});
```

## [Rate Limiting](#rate-limiting)

### [Defining Rate Limiters](#defining-rate-limiters)

Laravel includes powerful and customizable rate limiting services that you may utilize to restrict the amount of traffic for a given route or group of routes. To get started, you should define rate limiter configurations that meet your application's needs.

Rate limiters may be defined within the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
 1use Illuminate\Cache\RateLimiting\Limit; 2use Illuminate\Http\Request; 3use Illuminate\Support\Facades\RateLimiter; 4  5/** 6 * Bootstrap any application services. 7 */ 8public function boot(): void 9{10    RateLimiter::for('api', function (Request $request) {11        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());12    });13}
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

Rate limiters are defined using the `RateLimiter` facade's `for` method. The `for` method accepts a rate limiter name and a closure that returns the limit configuration that should apply to routes that are assigned to the rate limiter. Limit configuration are instances of the `Illuminate\Cache\RateLimiting\Limit` class. This class contains helpful "builder" methods so that you can quickly define your limit. The rate limiter name may be any string you wish:

```
 1use Illuminate\Cache\RateLimiting\Limit; 2use Illuminate\Http\Request; 3use Illuminate\Support\Facades\RateLimiter; 4  5/** 6 * Bootstrap any application services. 7 */ 8public function boot(): void 9{10    RateLimiter::for('global', function (Request $request) {11        return Limit::perMinute(1000);12    });13}
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

If the incoming request exceeds the specified rate limit, a response with a 429 HTTP status code will automatically be returned by Laravel. If you would like to define your own response that should be returned by a rate limit, you may use the `response` method:

```
1RateLimiter::for('global', function (Request $request) {2    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {3        return response('Custom response...', 429, $headers);4    });5});
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('Custom response...', 429, $headers);
    });
});
```

Since rate limiter callbacks receive the incoming HTTP request instance, you may build the appropriate rate limit dynamically based on the incoming request or authenticated user:

```
1RateLimiter::for('uploads', function (Request $request) {2    return $request->user()?->vipCustomer()3        ? Limit::none()4        : Limit::perHour(10);5});
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()?->vipCustomer()
        ? Limit::none()
        : Limit::perHour(10);
});
```

#### [Segmenting Rate Limits](#segmenting-rate-limits)

Sometimes you may wish to segment rate limits by some arbitrary value. For example, you may wish to allow users to access a given route 100 times per minute per IP address. To accomplish this, you may use the `by` method when building your rate limit:

```
1RateLimiter::for('uploads', function (Request $request) {2    return $request->user()->vipCustomer()3        ? Limit::none()4        : Limit::perMinute(100)->by($request->ip());5});
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
        ? Limit::none()
        : Limit::perMinute(100)->by($request->ip());
});
```

To illustrate this feature using another example, we can limit access to the route to 100 times per minute per authenticated user ID or 10 times per minute per IP address for guests:

```
1RateLimiter::for('uploads', function (Request $request) {2    return $request->user()3        ? Limit::perMinute(100)->by($request->user()->id)4        : Limit::perMinute(10)->by($request->ip());5});
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
        ? Limit::perMinute(100)->by($request->user()->id)
        : Limit::perMinute(10)->by($request->ip());
});
```

#### [Multiple Rate Limits](#multiple-rate-limits)

If needed, you may return an array of rate limits for a given rate limiter configuration. Each rate limit will be evaluated for the route based on the order they are placed within the array:

```
1RateLimiter::for('login', function (Request $request) {2    return [3        Limit::perMinute(500),4        Limit::perMinute(3)->by($request->input('email')),5    ];6});
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```

If you're assigning multiple rate limits segmented by identical `by` values, you should ensure that each `by` value is unique. The easiest way to achieve this is to prefix the values given to the `by` method:

```
1RateLimiter::for('uploads', function (Request $request) {2    return [3        Limit::perMinute(10)->by('minute:'.$request->user()->id),4        Limit::perDay(1000)->by('day:'.$request->user()->id),5    ];6});
RateLimiter::for('uploads', function (Request $request) {
    return [
        Limit::perMinute(10)->by('minute:'.$request->user()->id),
        Limit::perDay(1000)->by('day:'.$request->user()->id),
    ];
});
```

#### [Response-Based Rate Limiting](#response-base-rate-limiting)

In addition to rate limiting incoming requests, Laravel allows you to rate limit based on the response using the `after` method. This is useful when you only want to count certain responses toward the rate limit, such as validation errors, 404 responses, or other specific HTTP status codes.
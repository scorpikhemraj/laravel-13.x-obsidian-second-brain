---
title: Facades
description: Provide a static interface to classes available in the service container
url: https://laravel.com/docs/13.x/facades
tags: [framework]
cssclasses:
  - framework
  - color-blue
color: blue
---

# Facades

## Introduction

Throughout the Laravel documentation, you will see examples of code that interacts with Laravel's features via "facades". Facades provide a "static" interface to classes that are available in the application's [[03-architecture-concepts/02-service-container.md|service container]]. Laravel ships with many facades which provide access to almost all of Laravel's features.

Laravel facades serve as "static proxies" to underlying classes in the service container, providing the benefit of a terse, expressive syntax while maintaining more testability and flexibility than traditional static methods.

All of Laravel's facades are defined in the `Illuminate\Support\Facades` namespace:

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

#### Helper Functions

To complement facades, Laravel offers a variety of global "helper functions" that make it even easier to interact with common Laravel features. Some of the common helper functions you may interact with are `view`, `response`, `url`, `config`, and more.

For example, instead of using the `Illuminate\Support\Facades\Response` facade to generate a JSON response, we may simply use the `response` function:

```php
Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

## When to Utilize Facades

Facades have many benefits. They provide a terse, memorable syntax that allows you to use Laravel's features without remembering long class names that must be injected or configured manually.

However, some care must be taken when using facades. The primary danger of facades is class "scope creep". Since facades are so easy to use and do not require injection, it can be easy to let your classes continue to grow and use many facades in a single class.

### Facades vs. Dependency Injection

One of the primary benefits of dependency injection is the ability to swap implementations of the injected class. This is useful during testing since you can inject a mock or stub and assert that various methods were called on the stub.

Since facades use dynamic methods to proxy method calls to objects resolved from the service container, we actually can test facades just as we would test an injected class instance:

```php
use Illuminate\Support\Facades\Cache;

test('basic example', function () {
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```

### Facades vs. Helper Functions

In addition to facades, Laravel includes a variety of "helper" functions which can perform common tasks like generating views, firing events, dispatching jobs, or sending HTTP responses.

```php
return view('profile');

// is equivalent to

return Illuminate\Support\Facades\View::make('profile');
```

There is absolutely no practical difference between facades and helper functions. When using helper functions, you may still test them exactly as you would the corresponding facade.

## How Facades Work

In a Laravel application, a facade is a class that provides access to an object from the container. The machinery that makes this work is in the `Facade` class.

The `Facade` base class makes use of the `__callStatic()` magic-method to defer calls from your facade to an object resolved from the container:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

Notice that near the top of the file we are "importing" the `Cache` facade. This facade serves as a proxy for accessing the underlying implementation of the `Illuminate\Contracts\Cache\Factory` interface.

If we look at that `Illuminate\Support\Facades\Cache` class, you'll see that there is no static method `get`:

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

Instead, the `Cache` facade extends the base `Facade` class and defines the method `getFacadeAccessor()`. This method's job is to return the name of a service container binding. When a user references any static method on the `Cache` facade, Laravel resolves the `cache` binding from the [[03-architecture-concepts/02-service-container.md|service container]] and runs the requested method (in this case, `get`) against that object.

## Real-Time Facades

Using real-time facades, you may treat any class in your application as if it was a facade.

To illustrate how this can be used, let's first examine some code that does not use real-time facades. Assume our `Podcast` model has a `publish` method that requires injecting a `Publisher` instance:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

Using real-time facades, we can maintain the same testability while not being required to explicitly pass a `Publisher` instance. To generate a real-time facade, prefix the namespace of the imported class with `Facades`:

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Facades\App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(): void
    {
        $this->update(['publishing' => now()]);

        Publisher::publish($this);
    }
}
```

When the real-time facade is used, the publisher implementation will be resolved out of the service container. When testing, we can use Laravel's built-in facade testing helpers to mock this method call:

```php
use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

test('podcast can be published', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```

## Facade Class Reference

Below you will find every facade and its underlying class:

| Facade | Class | Service Container Binding |
|--------|-------|----------------------|
| App | Illuminate\\Foundation\\Application | `app` |
| Artisan | Illuminate\\Contracts\\Console\\Kernel | `artisan` |
| Auth | Illuminate\\Auth\\AuthManager | `auth` |
| Blade | Illuminate\\View\\Compilers\\BladeCompiler | `blade.compiler` |
| Cache | Illuminate\\Cache\\CacheManager | `cache` |
| Config | Illuminate\\Config\\Repository | `config` |
| Cookie | Illuminate\\Cookie\\CookieJar | `cookie` |
| Crypt | Illuminate\\Encryption\\Encrypter | `encrypter` |
| Date | Illuminate\\Support\\DateFactory | `date` |
| DB | Illuminate\\Database\\DatabaseManager | `db` |
| Event | Illuminate\\Events\\Dispatcher | `events` |
| File | Illuminate\\Filesystem\\Filesystem | `files` |
| Hash | Illuminate\\Contracts\\Hashing\\Hasher | `hash` |
| Lang | Illuminate\\Translation\\Translator | `translator` |
| Log | Illuminate\\Log\\LogManager | `log` |
| Mail | Illuminate\\Mail\\Mailer | `mailer` |
| Queue | Illuminate\\Queue\\QueueManager | `queue` |
| Redirect | Illuminate\\Routing\\Redirector | `redirect` |
| Redis | Illuminate\\Redis\\RedisManager | `redis` |
| Request | Illuminate\\Http\\Request | `request` |
| Route | Illuminate\\Routing\\Router | `router` |
| Schema | Illuminate\\Database\\Schema\\Builder | |
| Session | Illuminate\\Session\\SessionManager | `session` |
| Storage | Illuminate\\Filesystem\\FilesystemManager | `filesystem` |
| URL | Illuminate\\Routing\\UrlGenerator | `url` |
| Validator | Illuminate\\Validation\\Factory | `validator` |
| View | Illuminate\\View\\Factory | `view` |
| Vite | Illuminate\\Foundation\\Vite | |
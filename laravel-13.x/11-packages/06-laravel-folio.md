---
title: Laravel Folio
description: A powerful page based router designed to simplify routing in Laravel applications.
url: https://laravel.com/docs/13.x/folio
tags: [packages]
---

#Laravel Folio

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Page Paths / URIs](#page-paths-uris)
    -   [Subdomain Routing](#subdomain-routing)
-   [Creating Routes](#creating-routes)
    -   [Nested Routes](#nested-routes)
    -   [Index Routes](#index-routes)
-   [Route Parameters](#route-parameters)
-   [Route Model Binding](#route-model-binding)
    -   [Soft Deleted Models](#soft-deleted-models)
-   [Render Hooks](#render-hooks)
-   [Named Routes](#named-routes)
-   [Middleware](#middleware)
-   [Route Caching](#route-caching)

## [Introduction](#introduction)

[Laravel Folio](https://github.com/laravel/folio) is a powerful page based router designed to simplify routing in Laravel applications. With Laravel Folio, generating a route becomes as effortless as creating a Blade template within your application's `resources/views/pages` directory.

For example, to create a page that is accessible at the `/greeting` URL, just create a `greeting.blade.php` file in your application's `resources/views/pages` directory:

```
<div>
    Hello World
</div>
```

## [Installation](#installation)

To get started, install Folio into your project using the Composer package manager:

```
composer require laravel/folio
```

After installing Folio, you may execute the `folio:install` Artisan command:

```
php artisan folio:install
```

### [Page Paths / URIs](#page-paths-uris)

By default, Folio serves pages from your application's `resources/views/pages` directory. You may customize these directories in your Folio service provider's `boot` method:

```
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages/guest'))->uri('/');

Folio::path(resource_path('views/pages/admin'))
    ->uri('/admin')
    ->middleware([
        '*' => [
            'auth',
            'verified',
        ],
    ]);
```

### [Subdomain Routing](#subdomain-routing)

You may also route to pages based on the incoming request's subdomain:

```
use Laravel\Folio\Folio;

Folio::domain('admin.example.com')
    ->path(resource_path('views/pages/admin'));
```

## [Creating Routes](#creating-routes)

You may create a Folio route by placing a Blade template in any of your Folio mounted directories. By default, Folio mounts the `resources/views/pages` directory.

To quickly view a list of all of your Folio pages / routes:

```
php artisan folio:list
```

### [Nested Routes](#nested-routes)

You may create a nested route by creating one or more directories within one of Folio's directories:

```
php artisan folio:page user/profile

# pages/user/profile.blade.php → /user/profile
```

### [Index Routes](#index-routes)

Sometimes, you may wish to make a given page the "index" of a directory:

```
php artisan folio:page index
# pages/index.blade.php → /

php artisan folio:page users/index
# pages/users/index.blade.php → /users
```

## [Route Parameters](#route-parameters)

Often, you will need to have segments of the incoming request's URL injected into your page:

```
php artisan folio:page "users/[id]"

# pages/users/[id].blade.php → /users/1
```

Captured segments can be accessed as variables within your Blade template:

```
<div>
    User {{ $id }}
</div>
```

To capture multiple segments, you can prefix the encapsulated segment with three dots `...`:

```
php artisan folio:page "users/[...ids]"

# pages/users/[...ids].blade.php → /users/1/2/3
```

When capturing multiple segments, the captured segments will be injected into the page as an array:

```
<ul>
    @foreach ($ids as $id)
        <li>User {{ $id }}</li>
    @endforeach
</ul>
```

## [Route Model Binding](#route-model-binding)

If a wildcard segment of your page template's filename corresponds one of your application's Eloquent models, Folio will automatically take advantage of Laravel's route model binding:

```
php artisan folio:page "users/[User]"

# pages/users/[User].blade.php → /users/1
```

Captured models can be accessed as variables within your Blade template:

```
<div>
    User {{ $user->id }}
</div>
```

#### [Customizing the Key](#customizing-the-key)

Sometimes you may wish to resolve bound Eloquent models using a column other than `id`:

```
[Post:slug].blade.php
```

#### [Model Location](#model-location)

By default, Folio will search for your model within your application's `app/Models` directory:

```
php artisan folio:page "users/[.App.Models.User]"

# pages/users/[.App.Models.User].blade.php → /users/1
```

### [Soft Deleted Models](#soft-deleted-models)

By default, models that have been soft deleted are not retrieved when resolving implicit model bindings:

```
<?php

use function Laravel\Folio\{withTrashed};

withTrashed();

?>

<div>
    User {{ $user->id }}
</div>
```

## [Render Hooks](#render-hooks)

By default, Folio will return the content of the page's Blade template as the response to the incoming request:

```
<?php

use App\Models\Post;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

use function Laravel\Folio\render;

render(function (View $view, Post $post) {
    if (! Auth::user()->can('view', $post)) {
        return response('Unauthorized', 403);
    }

    return $view->with('photos', $post->author->photos);
}); ?>

<div>
    {{ $post->content }}
</div>
```

## [Named Routes](#named-routes)

You may specify a name for a given page's route using the `name` function:

```
<?php

use function Laravel\Folio\name;

name('users.index');
```

Just like Laravel's named routes, you may use the `route` function to generate URLs to Folio pages that have been assigned a name:

```
<a href="{{ route('users.index') }}">
    All Users
</a>
```

If the page has parameters, you may simply pass their values to the `route` function:

```
route('users.show', ['user' => $user]);
```

## [Middleware](#middleware)

You can apply middleware to a specific page by invoking the `middleware` function within the page's template:

```
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);

?>

<div>
    Dashboard
</div>
```

Or, to assign middleware to a group of pages, you may chain the `middleware` method after invoking the `Folio::path` method:

```
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',
    ],
]);
```

## [Route Caching](#route-caching)

When using Folio, you should always take advantage of Laravel's route caching capabilities. Folio listens for the `route:cache` Artisan command to ensure that Folio page definitions and route names are properly cached for maximum performance.
---
title: Views
description: Create and manage view templates in Laravel
url: https://laravel.com/docs/13.x/views
tags: [framework]
cssclasses:
  - framework
  - color-blue
color: blue
---

# Views

-   [Introduction](#introduction)
    -   [Writing Views in React / Svelte / Vue](#writing-views-in-react-svelte-or-vue)
-   [Creating and Rendering Views](#creating-and-rendering-views)
    -   [Nested View Directories](#nested-view-directories)
    -   [Creating the First Available View](#creating-the-first-available-view)
    -   [Determining if a View Exists](#determining-if-a-view-exists)
-   [Passing Data to Views](#passing-data-to-views)
    -   [Sharing Data With All Views](#sharing-data-with-all-views)
-   [View Composers](#view-composers)
    -   [View Creators](#view-creators)
-   [Optimizing Views](#optimizing-views)

## [Introduction](#introduction)

Of course, it's not practical to return entire HTML documents strings directly from your routes and controllers. Thankfully, views provide a convenient way to place all of our HTML in separate files.

Views separate your controller / application logic from your presentation logic and are stored in the `resources/views` directory. When using Laravel, view templates are usually written using the [[04-the-basics/08-blade-templates.md|Blade templating language]]. A simple view might look something like this:

```
1<!-- View stored in resources/views/greeting.blade.php -->2 3<html>4    <body>5        <h1>Hello, {{ $name }}</h1>6    </body>7</html>
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Since this view is stored at `resources/views/greeting.blade.php`, we may return it using the global `view` helper like so:

```
1Route::get('/', function () {2    return view('greeting', ['name' => 'James']);3});
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

Looking for more information on how to write Blade templates? Check out the full [[04-the-basics/08-blade-templates.md|Blade documentation]] to get started.

### [Writing Views in React / Svelte / Vue](#writing-views-in-react-svelte-or-vue)

Instead of writing their frontend templates in PHP via Blade, many developers have begun to prefer to write their templates using React, Svelte, or Vue. Laravel makes this painless thanks to [Inertia](https://inertiajs.com/), a library that makes it a cinch to tie your React / Svelte / Vue frontend to your Laravel backend without the typical complexities of building an SPA.

Our [[02-getting-started/06-starter-kits.md|React, Svelte, and Vue application starter kits]] give you a great starting point for your next Laravel application powered by Inertia.

## [Creating and Rendering Views](#creating-and-rendering-views)

You may create a view by placing a file with the `.blade.php` extension in your application's `resources/views` directory or by using the `make:view` Artisan command:

```
1php artisan make:view greeting
php artisan make:view greeting
```

The `.blade.php` extension informs the framework that the file contains a [[04-the-basics/08-blade-templates.md|Blade template]]. Blade templates contain HTML as well as Blade directives that allow you to easily echo values, create "if" statements, iterate over data, and more.

Once you have created a view, you may return it from one of your application's routes or controllers using the global `view` helper:

```
1Route::get('/', function () {2    return view('greeting', ['name' => 'James']);3});
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

Views may also be returned using the `View` facade:

```
1use Illuminate\Support\Facades\View;2 3return View::make('greeting', ['name' => 'James']);
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```

As you can see, the first argument passed to the `view` helper corresponds to the name of the view file in the `resources/views` directory. The second argument is an array of data that should be made available to the view.

### [Nested View Directories](#nested-view-directories)

Views may also be nested within subdirectories of the `resources/views` directory. "Dot" notation may be used to reference nested views:

```
1return view('admin.profile', $data);
return view('admin.profile', $data);
```

View directory names should not contain the `.` character.

### [Creating the First Available View](#creating-the-first-available-view)

Using the `View` facade's `first` method, you may create the first view that exists in a given array of views:

```
1use Illuminate\Support\Facades\View;2 3return View::first(['custom.admin', 'admin'], $data);
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

### [Determining if a View Exists](#determining-if-a-view-exists)

If you need to determine if a view exists, you may use the `View` facade:

```
1use Illuminate\Support\Facades\View;2 3if (View::exists('admin.profile')) {4    // ...5}
use Illuminate\Support\Facades\View;

if (View::exists('admin.profile')) {
    // ...
}
```

## [Passing Data to Views](#passing-data-to-views)

As you saw in the previous examples, you may pass an array of data to views to make that data available to the view:

```
1return view('greetings', ['name' => 'Victoria']);
return view('greetings', ['name' => 'Victoria']);
```

When passing information in this manner, the data should be an array with key / value pairs. After providing data to a view, you can then access each value within your view using the data's keys.

As an alternative to passing a complete array of data to the `view` helper function, you may use the `with` method to add individual pieces of data to the view:

```
1return view('greeting')2    ->with('name', 'Victoria')3    ->with('occupation', 'Astronaut');
return view('greeting')
    ->with('name', 'Victoria')
    ->with('occupation', 'Astronaut');
```

### [Sharing Data With All Views](#sharing-data-with-all-views)

Occasionally, you may need to share data with all views that are rendered by your application. You may do so using the `View` facade's `share` method:

```
 1<?php 2  3namespace App\Providers; 4  5use Illuminate\Support\Facades\View; 6  7class AppServiceProvider extends ServiceProvider 8{ 9    /**10     * Register any application services.11     */12    public function register(): void13    {14        // ...15    }16 17    /**18     * Bootstrap any application services.19     */20    public function boot(): void21    {22        View::share('key', 'value');23    }24}
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::share('key', 'value');
    }
}
```

## [View Composers](#view-composers)

View composers are callbacks or class methods that are called when a view is rendered. If you have data that you want to be bound to a view each time that view is rendered, a view composer can help you organize that logic into a single location.

Typically, view composers will be registered within one of your application's [[03-architecture-concepts/03-service-providers.md|service providers]]. In this example, we'll assume that the `App\Providers\AppServiceProvider` will house this logic.

We'll use the `View` facade's `composer` method to register the view composer:

```
 1<?php 2  3namespace App\Providers; 4  5use App\View\Composers\ProfileComposer; 6use Illuminate\Support\Facades; 7use Illuminate\Support\ServiceProvider; 8use Illuminate\View\View; 9 10class AppServiceProvider extends ServiceProvider11{12    /**13     * Register any application services.14     */15    public function register(): void16    {17        // ...18    }19 20    /**21     * Bootstrap any application services.22     */23    public function boot(): void24    {25        // Using class-based composers...26        Facades\View::composer('profile', ProfileComposer::class);27 28        // Using closure-based composers...29        Facades\View::composer('welcome', function (View $view) {30            // ...31        });32 33        Facades\View::composer('dashboard', function (View $view) {34            // ...35        });36    }37}
<?php

namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades;
use Illuminate\Support\ServiceProvider;
use Illuminate\View\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // Using class-based composers...
        Facades\View::composer('profile', ProfileComposer::class);

        // Using closure-based composers...
        Facades\View::composer('welcome', function (View $view) {
            // ...
        });

        Facades\View::composer('dashboard', function (View $view) {
            // ...
        });
    }
}
```

Now that we have registered the composer, the `compose` method of the `App\View\Composers\ProfileComposer` class will be executed each time the `profile` view is being rendered:

```
 1<?php 2  3namespace App\View\Composers; 4  5use App\Repositories\UserRepository; 6use Illuminate\View\View; 7  8class ProfileComposer 9{10    /**11     * Create a new profile composer.12     */13    public function __construct(14        protected UserRepository $users,15    ) {}16 17    /**18     * Bind data to the view.19     */20    public function compose(View $view): void21    {22        $view->with('count', $this->users->count());23    }24}
<?php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * Create a new profile composer.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * Bind data to the view.
     */
    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```

As you can see, all view composers are resolved via the [[03-architecture-concepts/02-service-container.md|service container]], so you may type-hint any dependencies you need within a composer's constructor.

#### [Attaching a Composer to Multiple Views](#attaching-a-composer-to-multiple-views)

You may attach a view composer to multiple views at once by passing an array of views as the first argument to the `composer` method:

```
1use App\Views\Composers\MultiComposer;2use Illuminate\Support\Facades\View;3 4View::composer(5    ['profile', 'dashboard'],6    MultiComposer::class7);
use App\Views\Composers\MultiComposer;
use Illuminate\Support\Facades\View;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

The `composer` method also accepts the `*` character as a wildcard, allowing you to attach a composer to all views:

```
1use Illuminate\Support\Facades;2use Illuminate\View\View;3 4Facades\View::composer('*', function (View $view) {5    // ...6});
use Illuminate\Support\Facades;
use Illuminate\View\View;

Facades\View::composer('*', function (View $view) {
    // ...
});
```

### [View Creators](#view-creators)

View "creators" are very similar to view composers; however, they are executed immediately after the view is instantiated instead of waiting until the view is about to render:

```
1use App\View\Creators\ProfileCreator;2use Illuminate\Support\Facades\View;3 4View::creator('profile', ProfileCreator::class);
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

## [Optimizing Views](#optimizing-views)

By default, Blade template views are compiled on demand. When a request is executed that renders a view, Laravel will determine if a compiled version of the view exists. If the file exists, Laravel will then determine if the uncompiled view has been modified more recently than the compiled view. If the compiled view either does not exist, or the uncompiled view has been modified, Laravel will recompile the view.

Compiling views during the request may have a small negative impact on performance, so Laravel provides the `view:cache` Artisan command to precompile all of the views utilized by your application:

```
1php artisan view:cache
php artisan view:cache
```

You may use the `view:clear` command to clear the view cache:

```
1php artisan view:clear
php artisan view:clear
```
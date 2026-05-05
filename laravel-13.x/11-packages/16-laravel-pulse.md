---
title: Laravel Pulse
description: Application performance monitoring and insights dashboard
url: https://laravel.com/docs/13.x/pulse
tags: [packages]
---

# Laravel Pulse

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Configuration](#configuration)
-   [Dashboard](#dashboard)
    -   [Authorization](#dashboard-authorization)
    -   [Customization](#dashboard-customization)
    -   [Resolving Users](#dashboard-resolving-users)
    -   [Cards](#dashboard-cards)
-   [Capturing Entries](#capturing-entries)
    -   [Recorders](#recorders)
    -   [Filtering](#filtering)
-   [Performance](#performance)
    -   [Using a Different Database](#using-a-different-database)
    -   [Redis Ingest](#ingest)
    -   [Sampling](#sampling)
    -   [Trimming](#trimming)
    -   [Handling Pulse Exceptions](#pulse-exceptions)
-   [Custom Cards](#custom-cards)
    -   [Card Components](#custom-card-components)
    -   [Styling](#custom-card-styling)
    -   [Data Capture and Aggregation](#custom-card-data)

## [Introduction](#introduction)

[Laravel Pulse](https://github.com/laravel/pulse) delivers at-a-glance insights into your application's performance and usage. With Pulse, you can track down bottlenecks like slow jobs and endpoints, find your most active users, and more.

For in-depth debugging of individual events, check out [Laravel Telescope](/docs/13.x/telescope).

## [Installation](#installation)

Pulse's first-party storage implementation currently requires a MySQL, MariaDB, or PostgreSQL database. If you are using a different database engine, you will need a separate MySQL, MariaDB, or PostgreSQL database for your Pulse data.

You may install Pulse using the Composer package manager:

```
1composer require laravel/pulse
composer require laravel/pulse
```

Next, you should publish the Pulse configuration and migration files using the `vendor:publish` Artisan command:

```
1php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
```

Finally, you should run the `migrate` command in order to create the tables needed to store Pulse's data:

```
1php artisan migrate
php artisan migrate
```

Once Pulse's database migrations have been run, you may access the Pulse dashboard via the `/pulse` route.

If you do not want to store Pulse data in your application's primary database, you may [specify a dedicated database connection](#using-a-different-database).

### [Configuration](#configuration)

Many of Pulse's configuration options can be controlled using environment variables. To see the available options, register new recorders, or configure advanced options, you may publish the `config/pulse.php` configuration file:

```
1php artisan vendor:publish --tag=pulse-config
php artisan vendor:publish --tag=pulse-config
```

## [Dashboard](#dashboard)

### [Authorization](#dashboard-authorization)

The Pulse dashboard may be accessed via the `/pulse` route. By default, you will only be able to access this dashboard in the `local` environment, so you will need to configure authorization for your production environments by customizing the `'viewPulse'` authorization gate. You can accomplish this within your application's `app/Providers/AppServiceProvider.php` file:

```
 1use App\Models\User; 2use Illuminate\Support\Facades\Gate; 3  4/** 5 * Bootstrap any application services. 6 */ 7public function boot(): void 8{ 9    Gate::define('viewPulse', function (User $user) { 10        return $user->isAdmin(); 11    }); 12  13    // ... 14}
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('viewPulse', function (User $user) {
        return $user->isAdmin();
    });

    // ...
}
```

### [Customization](#dashboard-customization)

The Pulse dashboard cards and layout may be configured by publishing the dashboard view. The dashboard view will be published to `resources/views/vendor/pulse/dashboard.blade.php`:

```
1php artisan vendor:publish --tag=pulse-dashboard
php artisan vendor:publish --tag=pulse-dashboard
```

The dashboard is powered by [Livewire](https://livewire.laravel.com/), and allows you to customize the cards and layout without needing to rebuild any JavaScript assets.

Within this file, the `<x-pulse>` component is responsible for rendering the dashboard and provides a grid layout for the cards. If you would like the dashboard to span the full width of the screen, you may provide the `full-width` prop to the component:

```
1<x-pulse full-width>2    ...3</x-pulse>
<x-pulse full-width>
    ...
</x-pulse>
```

By default, the `<x-pulse>` component will create a 12 column grid, but you may customize this using the `cols` prop:

```
1<x-pulse cols="16">2    ...3</x-pulse>
<x-pulse cols="16">
    ...
</x-pulse>
```

Each card accepts a `cols` and `rows` prop to control the space and positioning:

```
1<livewire:pulse.usage cols="4" rows="2" />
<livewire:pulse.usage cols="4" rows="2" />
```

Most cards also accept an `expand` prop to show the full card instead of scrolling:

```
1<livewire:pulse.slow-queries expand />
<livewire:pulse.slow-queries expand />
```

### [Resolving Users](#dashboard-resolving-users)

For cards that display information about your users, such as the Application Usage card, Pulse will only record the user's ID. When rendering the dashboard, Pulse will resolve the `name` and `email` fields from your default `Authenticatable` model and display avatars using the Gravatar web service.

You may customize the fields and avatar by invoking the `Pulse::user` method within your application's `App\Providers\AppServiceProvider` class.

The `user` method accepts a closure which will receive the `Authenticatable` model to be displayed and should return an array containing `name`, `extra`, and `avatar` information for the user:

```
 1use Laravel\Pulse\Facades\Pulse; 2  3/** 4 * Bootstrap any application services. 5 */ 6public function boot(): void 7{ 8    Pulse::user(fn ($user) => [ 9        'name' => $user->name, 10        'extra' => $user->email, 11        'avatar' => $user->avatar_url, 12    ]); 13  14    // ... 15}
use Laravel\Pulse\Facades\Pulse;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::user(fn ($user) => [
        'name' => $user->name,
        'extra' => $user->email,
        'avatar' => $user->avatar_url,
    ]);

    // ...
}
```

You may completely customize how the authenticated user is captured and retrieved by implementing the `Laravel\Pulse\Contracts\ResolvesUsers` contract and binding it in Laravel's [[03-architecture-concepts/02-service-container.md#binding-a-singleton|service container]].

### [Cards](#dashboard-cards)

#### [Servers](#servers-card)

The `<livewire:pulse.servers />` card displays system resource usage for all servers running the `pulse:check` command. Please refer to the documentation regarding the [servers recorder](#servers-recorder) for more information on system resource reporting.

If you replace a server in your infrastructure, you may wish to stop displaying the inactive server in the Pulse dashboard after a given duration. You may accomplish this using the `ignore-after` prop, which accepts the number of seconds after which inactive servers should be removed from the Pulse dashboard. Alternatively, you may provide a relative time formatted string, such as `1 hour` or `3 days and 1 hour`:

```
1<livewire:pulse.servers ignore-after="3 hours" />
<livewire:pulse.servers ignore-after="3 hours" />
```

#### [Application Usage](#application-usage-card)

The `<livewire:pulse.usage />` card displays the top 10 users making requests to your application, dispatching jobs, and experiencing slow requests.

If you wish to view all usage metrics on screen at the same time, you may include the card multiple times and specify the `type` attribute:

```
1<livewire:pulse.usage type="requests" />2<livewire:pulse.usage type="slow_requests" />3<livewire:pulse.usage type="jobs" />
<livewire:pulse.usage type="requests" />
<livewire:pulse.usage type="slow_requests" />
<livewire:pulse.usage type="jobs" />
```

To learn how to customize how Pulse retrieves and displays user information, consult our documentation on [resolving users](#dashboard-resolving-users).

If your application receives a lot of requests or dispatches a lot of jobs, you may wish to enable [sampling](#sampling). See the [user requests recorder](#user-requests-recorder), [user jobs recorder](#user-jobs-recorder), and [slow jobs recorder](#slow-jobs-recorder) documentation for more information.

#### [Exceptions](#exceptions-card)

The `<livewire:pulse.exceptions />` card shows the frequency and recency of exceptions occurring in your application. By default, exceptions are grouped based on the exception class and location where it occurred. See the [exceptions recorder](#exceptions-recorder) documentation for more information.

#### [Queues](#queues-card)

The `<livewire:pulse.queues />` card shows the throughput of the queues in your application, including the number of jobs queued, processing, processed, released, and failed. See the [queues recorder](#queues-recorder) documentation for more information.

#### [Slow Requests](#slow-requests-card)

The `<livewire:pulse.slow-requests />` card shows incoming requests to your application that exceed the configured threshold, which is 1,000ms by default. See the [slow requests recorder](#slow-requests-recorder) documentation for more information.

#### [Slow Jobs](#slow-jobs-card)

The `<livewire:pulse.slow-jobs />` card shows the queued jobs in your application that exceed the configured threshold, which is 1,000ms by default. See the [slow jobs recorder](#slow-jobs-recorder) documentation for more information.

#### [Slow Queries](#slow-queries-card)

The `<livewire:pulse.slow-queries />` card shows the database queries in your application that exceed the configured threshold, which is 1,000ms by default.

By default, slow queries are grouped based on the SQL query (without bindings) and the location where it occurred, but you may choose to not capture the location if you wish to group solely on the SQL query.

If you encounter rendering performance issues due to extremely large SQL queries receiving syntax highlighting, you may disable highlighting by adding the `without-highlighting` prop:

```
1<livewire:pulse.slow-queries without-highlighting />
<livewire:pulse.slow-queries without-highlighting />
```

See the [slow queries recorder](#slow-queries-recorder) documentation for more information.

#### [Slow Outgoing Requests](#slow-outgoing-requests-card)

The `<livewire:pulse.slow-outgoing-requests />` card shows outgoing requests made using Laravel's [[05-digging-deeper/11-http-client.md|HTTP client]] that exceed the configured threshold, which is 1,000ms by default.

By default, entries will be grouped by the full URL. However, you may wish to normalize or group similar outgoing requests using regular expressions. See the [slow outgoing requests recorder](#slow-outgoing-requests-recorder) documentation for more information.

#### [Cache](#cache-card)

The `<livewire:pulse.cache />` card shows the cache hit and miss statistics for your application, both globally and for individual keys.

By default, entries will be grouped by key. However, you may wish to normalize or group similar keys using regular expressions. See the [cache interactions recorder](#cache-interactions-recorder) documentation for more information.

## [Capturing Entries](#capturing-entries)

Most Pulse recorders will automatically capture entries based on framework events dispatched by Laravel. However, the [servers recorder](#servers-recorder) and some third-party cards must poll for information regularly. To use these cards, you must run the `pulse:check` daemon on all of your individual application servers:

```
1php artisan pulse:check
php artisan pulse:check
```

To keep the `pulse:check` process running permanently in the background, you should use a process monitor such as Supervisor to ensure that the command does not stop running.

As the `pulse:check` command is a long-lived process, it will not see changes to your codebase without being restarted. You should gracefully restart the command by calling the `pulse:restart` command during your application's deployment process:

```
1php artisan pulse:restart
php artisan pulse:restart
```

Pulse uses the [[05-digging-deeper/03-cache.md|cache]] to store restart signals, so you should verify that a cache driver is properly configured for your application before using this feature.

### [Recorders](#recorders)

Recorders are responsible for capturing entries from your application to be recorded in the Pulse database. Recorders are registered and configured in the `recorders` section of the [Pulse configuration file](#configuration).

#### [Cache Interactions](#cache-interactions-recorder)

The `CacheInteractions` recorder captures information about the [[05-digging-deeper/03-cache.md|cache]] hits and misses occurring in your application for display on the [Cache](#cache-card) card.

You may optionally adjust the [sample rate](#sampling) and ignored key patterns.

You may also configure key grouping so that similar keys are grouped as a single entry. For example, you may wish to remove unique IDs from keys caching the same type of information. Groups are configured using a regular expression to "find and replace" parts of the key. An example is included in the configuration file:

```
1Recorders\CacheInteractions::class => [2    // ...3    'groups' => [4        // '/:\d+/' => ':*',5    ],6],
Recorders\CacheInteractions::class => [
    // ...
    'groups' => [
        // '/:\d+/' => ':*',
    ],
],
```

The first pattern that matches will be used. If no patterns match, then the key will be captured as-is.

#### [Exceptions](#exceptions-recorder)

The `Exceptions` recorder captures information about reportable exceptions occurring in your application for display on the [Exceptions](#exceptions-card) card.

You may optionally adjust the [sample rate](#sampling) and ignored exception patterns. You may also configure whether to capture the location that the exception originated from. The captured location will be displayed on the Pulse dashboard which can help to track down the exception origin; however, if the same exception occurs in multiple locations then it will appear multiple times for each unique location.

#### [Queues](#queues-recorder)

The `Queues` recorder captures information about your application's queues for display on the [Queues](#queues-card).

You may optionally adjust the [sample rate](#sampling) and ignored jobs patterns.

#### [Slow Jobs](#slow-jobs-recorder)

The `SlowJobs` recorder captures information about slow jobs occurring in your application for display on the [Slow Jobs](#slow-jobs-recorder) card.

You may optionally adjust the slow job threshold, [sample rate](#sampling), and ignored job patterns.

You may have some jobs that you expect to take longer than others. In those cases, you may configure per-job thresholds:

```
1Recorders\SlowJobs::class => [2    // ...3    'threshold' => [4        '#^App\\Jobs\\GenerateYearlyReports$#' => 5000,5        'default' => env('PULSE_SLOW_JOBS_THRESHOLD', 1000),6    ],7],
Recorders\SlowJobs::class => [
    // ...
    'threshold' => [
        '#^App\\Jobs\\GenerateYearlyReports$#' => 5000,
        'default' => env('PULSE_SLOW_JOBS_THRESHOLD', 1000),
    ],
],
```

If no regular expression patterns match the job's classname, then the `'default'` value will be used.

#### [Slow Outgoing Requests](#slow-outgoing-requests-recorder)

The `SlowOutgoingRequests` recorder captures information about outgoing HTTP requests made using Laravel's [[05-digging-deeper/11-http-client.md|HTTP client]] that exceed the configured threshold for display on the [Slow Outgoing Requests](#slow-outgoing-requests-card) card.

You may optionally adjust the slow outgoing request threshold, [sample rate](#sampling), and ignored URL patterns.

You may have some outgoing requests that you expect to take longer than others. In those cases, you may configure per-request thresholds:

```
1Recorders\SlowOutgoingRequests::class => [2    // ...3    'threshold' => [4        '#backup.zip$#' => 5000,5        'default' => env('PULSE_SLOW_OUTGOING_REQUESTS_THRESHOLD', 1000),6    ],7],
Recorders\SlowOutgoingRequests::class => [
    // ...
    'threshold' => [
        '#backup.zip$#' => 5000,
        'default' => env('PULSE_SLOW_OUTGOING_REQUESTS_THRESHOLD', 1000),
    ],
],
```

If no regular expression patterns match the request's URL, then the `'default'` value will be used.

You may also configure URL grouping so that similar URLs are grouped as a single entry. For example, you may wish to remove unique IDs from URL paths or group by domain only. Groups are configured using a regular expression to "find and replace" parts of the URL. Some examples are included in the configuration file:

```
1Recorders\SlowOutgoingRequests::class => [2    // ...3    'groups' => [4        // '#^https://api\.github\.com/repos/.*$#' => 'api.github.com/repos/*',5        // '#^https?://([^/]*).*$#' => '\1',6        // '#/\d+#' => '/*',7    ],8],
Recorders\SlowOutgoingRequests::class => [
    // ...
    'groups' => [
        // '#^https://api\.github\.com/repos/.*$#' => 'api.github.com/repos/*',
        // '#^https?://([^/]*).*$#' => '\1',
        // '#/\d+#' => '/*',
    ],
],
```

The first pattern that matches will be used. If no patterns match, then the URL will be captured as-is.

#### [Slow Queries](#slow-queries-recorder)

The `SlowQueries` recorder captures any database queries in your application that exceed the configured threshold for display on the [Slow Queries](#slow-queries-card) card.

You may optionally adjust the slow query threshold, [sample rate](#sampling), and ignored query patterns. You may also configure whether to capture the query location. The captured location will be displayed on the Pulse dashboard which can help to track down the query origin; however, if the same query is made in multiple locations then it will appear multiple times for each unique location.

You may have some queries that you expect to take longer than others. In those cases, you may configure per-query thresholds:

```
1Recorders\SlowQueries::class => [2    // ...3    'threshold' => [4        '#^insert into `yearly_reports`#' => 5000,5        'default' => env('PULSE_SLOW_QUERIES_THRESHOLD', 1000),6    ],7],
Recorders\SlowQueries::class => [
    // ...
    'threshold' => [
        '#^insert into `yearly_reports`#' => 5000,
        'default' => env('PULSE_SLOW_QUERIES_THRESHOLD', 1000),
    ],
],
```

If no regular expression patterns match the query's SQL, then the `'default'` value will be used.

#### [Slow Requests](#slow-requests-recorder)

The `Requests` recorder captures information about requests made to your application for display on the [Slow Requests](#slow-requests-card) and [Application Usage](#application-usage-card) cards.

You may optionally adjust the slow route threshold, [sample rate](#sampling), and ignored paths.

You may have some requests that you expect to take longer than others. In those cases, you may configure per-request thresholds:

```
1Recorders\SlowRequests::class => [2    // ...3    'threshold' => [4        '#^/admin/#' => 5000,5        'default' => env('PULSE_SLOW_REQUESTS_THRESHOLD', 1000),6    ],7],
Recorders\SlowRequests::class => [
    // ...
    'threshold' => [
        '#^/admin/#' => 5000,
        'default' => env('PULSE_SLOW_REQUESTS_THRESHOLD', 1000),
    ],
],
```

If no regular expression patterns match the request's URL, then the `'default'` value will be used.

#### [Servers](#servers-recorder)

The `Servers` recorder captures CPU, memory, and storage usage of the servers that power your application for display on the [Servers](#servers-card) card. This recorder requires the [pulse:check command](#capturing-entries) to be running on each of the servers you wish to monitor.

Each reporting server must have a unique name. By default, Pulse will use the value returned by PHP's `gethostname` function. If you wish to customize this, you may set the `PULSE_SERVER_NAME` environment variable:

```
1PULSE_SERVER_NAME=load-balancer
PULSE_SERVER_NAME=load-balancer
```

The Pulse configuration file also allows you to customize the directories that are monitored.

#### [User Jobs](#user-jobs-recorder)

The `UserJobs` recorder captures information about the users dispatching jobs in your application for display on the [Application Usage](#application-usage-card) card.

You may optionally adjust the [sample rate](#sampling) and ignored job patterns.

#### [User Requests](#user-requests-recorder)

The `UserRequests` recorder captures information about the users making requests to your application for display on the [Application Usage](#application-usage-card) card.

You may optionally adjust the [sample rate](#sampling) and ignored URL patterns.

### [Filtering](#filtering)

As we have seen, many [recorders](#recorders) offer the ability to, via configuration, "ignore" incoming entries based on their value, such as a request's URL. But, sometimes it may be useful to filter out records based on other factors, such as the currently authenticated user. To filter out these records, you may pass a closure to Pulse's `filter` method. Typically, the `filter` method should be invoked within the `boot` method of your application's `AppServiceProvider`:

```
 1use Illuminate\Support\Facades\Auth; 2use Laravel\Pulse\Entry; 3use Laravel\Pulse\Facades\Pulse; 4use Laravel\Pulse\Value; 5  6/** 7 * Bootstrap any application services. 8 */ 9public function boot(): void10{11    Pulse::filter(function (Entry|Value $entry) {12        return Auth::user()->isNotAdmin();13    });14  15    // ...16}
use Illuminate\Support\Facades\Auth;
use Laravel\Pulse\Entry;
use Laravel\Pulse\Facades\Pulse;
use Laravel\Pulse\Value;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::filter(function (Entry|Value $entry) {
        return Auth::user()->isNotAdmin();
    });

    // ...
}
```

## [Performance](#performance)

Pulse has been designed to drop into an existing application without requiring any additional infrastructure. However, for high-traffic applications, there are several ways of removing any impact Pulse may have on your application's performance.

### [Using a Different Database](#using-a-different-database)

For high-traffic applications, you may prefer to use a dedicated database connection for Pulse to avoid impacting your application database.

You may customize the [[07-database/01-database-getting-started.md#configuration|database connection]] used by Pulse by setting the `PULSE_DB_CONNECTION` environment variable.

```
1PULSE_DB_CONNECTION=pulse
PULSE_DB_CONNECTION=pulse
```

### [Redis Ingest](#ingest)

The Redis Ingest requires Redis 6.2 or greater and `phpredis` or `predis` as the application's configured Redis client driver.

By default, Pulse will store entries directly to the [configured database connection](#using-a-different-database) after the HTTP response has been sent to the client or a job has been processed; however, you may use Pulse's Redis ingest driver to send entries to a Redis stream instead. This can be enabled by configuring the `PULSE_INGEST_DRIVER` environment variable:

```
1PULSE_INGEST_DRIVER=redis
PULSE_INGEST_DRIVER=redis
```

Pulse will use your default [[07-database/06-redis.md#configuration|Redis connection]] by default, but you may customize this via the `PULSE_REDIS_CONNECTION` environment variable:

```
1PULSE_REDIS_CONNECTION=pulse
PULSE_REDIS_CONNECTION=pulse
```

When using the Redis ingest driver, your Pulse installation should always use a different Redis connection than your Redis powered queue, if applicable.

When using the Redis ingest, you will need to run the `pulse:work` command to monitor the stream and move entries from Redis into Pulse's database tables.

```
1php artisan pulse:work
php artisan pulse:work
```

To keep the `pulse:work` process running permanently in the background, you should use a process monitor such as Supervisor to ensure that the Pulse worker does not stop running.

As the `pulse:work` command is a long-lived process, it will not see changes to your codebase without being restarted. You should gracefully restart the command by calling the `pulse:restart` command during your application's deployment process:

```
1php artisan pulse:restart
php artisan pulse:restart
```

Pulse uses the [[05-digging-deeper/03-cache.md|cache]] to store restart signals, so you should verify that a cache driver is properly configured for your application before using this feature.

### [Sampling](#sampling)

By default, Pulse will capture every relevant event that occurs in your application. For high-traffic applications, this can result in needing to aggregate millions of database rows in the dashboard, especially for longer time periods.

You may instead choose to enable "sampling" on certain Pulse data recorders. For example, setting the sample rate to `0.1` on the [User Requests](#user-requests-recorder) recorder will mean that you only record approximately 10% of the requests to your application. In the dashboard, the values will be scaled up and prefixed with a `~` to indicate that they are an approximation.

In general, the more entries you have for a particular metric, the lower you can safely set the sample rate without sacrificing too much accuracy.

### [Trimming](#trimming)

Pulse will automatically trim its stored entries once they are outside of the dashboard window. Trimming occurs when ingesting data using a lottery system which may be customized in the Pulse [configuration file](#configuration).

### [Handling Pulse Exceptions](#pulse-exceptions)

If an exception occurs while capturing Pulse data, such as being unable to connect to the storage database, Pulse will silently fail to avoid impacting your application.

If you wish to customize how these exceptions are handled, you may provide a closure to the `handleExceptionsUsing` method:

```
1use Laravel\Pulse\Facades\Pulse;2use Illuminate\Support\Facades\Log;3 4Pulse::handleExceptionsUsing(function ($e) {5    Log::debug('An exception happened in Pulse', [6        'message' => $e->getMessage(),7        'stack' => $e->getTraceAsString(),8    ]);9});
use Laravel\Pulse\Facades\Pulse;
use Illuminate\Support\Facades\Log;

Pulse::handleExceptionsUsing(function ($e) {
    Log::debug('An exception happened in Pulse', [
        'message' => $e->getMessage(),
        'stack' => $e->getTraceAsString(),
    ]);
});
```

## [Custom Cards](#custom-cards)

Pulse allows you to build custom cards to display data relevant to your application's specific needs. Pulse uses [Livewire](https://livewire.laravel.com), so you may want to [review its documentation](https://livewire.laravel.com/docs) before building your first custom card.

### [Card Components](#custom-card-components)

Creating a custom card in Laravel Pulse starts with extending the base `Card` Livewire component and defining a corresponding view:

```
 1namespace App\Livewire\Pulse; 2  3use Laravel\Pulse\Livewire\Card; 4use Livewire\Attributes\Lazy; 5  6#[Lazy] 7class TopSellers extends Card 8{ 9    public function render()10    {11        return view('livewire.pulse.top-sellers');12    }13}
namespace App\Livewire\Pulse;

use Laravel\Pulse\Livewire\Card;
use Livewire\Attributes\Lazy;

#[Lazy]
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers');
    }
}
```

When using Livewire's [lazy loading](https://livewire.laravel.com/docs/lazy) feature, The `Card` component will automatically provide a placeholder that respects the `cols` and `rows` attributes passed to your component.

When writing your Pulse card's corresponding view, you may leverage Pulse's Blade components for a consistent look and feel:

```
 1<x-pulse::card :cols="$cols" :rows="$rows" :class="$class" wire:poll.5s=""> 2    <x-pulse::card-header name="Top Sellers"> 3        <x-slot:icon> 4            ... 5        </x-slot:icon> 6    </x-pulse::card-header> 7  8    <x-pulse::scroll :expand="$expand"> 9        ...10    </x-pulse::scroll>11</x-pulse::card>
<x-pulse::card :cols="$cols" :rows="$rows" :class="$class" wire:poll.5s="">
    <x-pulse::card-header name="Top Sellers">
        <x-slot:icon>
            ...
        </x-slot:icon>
    </x-pulse::card-header>

    <x-pulse::scroll :expand="$expand">
        ...
    </x-pulse::scroll>
</x-pulse::card>
```

The `$cols`, `$rows`, `$class`, and `$expand` variables should be passed to their respective Blade components so the card layout may be customized from the dashboard view. You may also wish to include the `wire:poll.5s=""` attribute in your view to have the card automatically update.

Once you have defined your Livewire component and template, the card may be included in your [dashboard view](#dashboard-customization):

```
1<x-pulse>2    ...3 4    <livewire:pulse.top-sellers cols="4" />5</x-pulse>
<x-pulse>
    ...

    <livewire:pulse.top-sellers cols="4" />
</x-pulse>
```

If your card is included in a package, you will need to register the component with Livewire using the `Livewire::component` method.

### [Styling](#custom-card-styling)

If your card requires additional styling beyond the classes and components included with Pulse, there are a few options for including custom CSS for your cards.

#### [Laravel Vite Integration](#custom-card-styling-vite)

If your custom card lives within your application's code base and you are using Laravel's [[04-the-basics/09-asset-bundling-vite.md|Vite integration]], you may update your `vite.config.js` file to include a dedicated CSS entry point for your card:

```
1laravel({2    input: [3        'resources/css/pulse/top-sellers.css',4        // ...5    ],6}),
laravel({
    input: [
        'resources/css/pulse/top-sellers.css',
        // ...
    ],
}),
```

You may then use the `@vite` Blade directive in your [dashboard view](#dashboard-customization), specifying the CSS entrypoint for your card:

```
1<x-pulse>2    @vite('resources/css/pulse/top-sellers.css')3 4    ...5</x-pulse>
<x-pulse>
    @vite('resources/css/pulse/top-sellers.css')

    ...
</x-pulse>
```

#### [CSS Files](#custom-card-styling-css)

For other use cases, including Pulse cards contained within a package, you may instruct Pulse to load additional stylesheets by defining a `css` method on your Livewire component that returns the file path to your CSS file:

```
1class TopSellers extends Card2{3    // ...4 5    protected function css()6    {7        return __DIR__.'/../../dist/top-sellers.css';8    }9}
class TopSellers extends Card
{
    // ...

    protected function css()
    {
        return __DIR__.'/../../dist/top-sellers.css';
    }
}
```

When this card is included on the dashboard, Pulse will automatically include the contents of this file within a `<style>` tag so it does not need to be published to the `public` directory.

#### [Tailwind CSS](#custom-card-styling-tailwind)

When using Tailwind CSS, you should create a dedicated CSS entrypoint. The following example excludes Tailwind's [Preflight](https://tailwindcss.com/docs/preflight) base styles which are already included by Pulse, and scopes Tailwind using a CSS selector to avoid conflicts with Pulse's Tailwind classes:

```
 1@import "tailwindcss/theme.css"; 2  3@custom-variant dark (&:where(.dark, .dark *)); 4@source "./../../views/livewire/pulse/top-sellers.blade.php"; 5  6@theme { 7  /* ... */ 8} 9 10#top-sellers {11  @import "tailwindcss/utilities.css" source(none);12}
@import "tailwindcss/theme.css";

@custom-variant dark (&:where(.dark, .dark *));
@source "./../../views/livewire/pulse/top-sellers.blade.php";

@theme {
  /* ... */
}

#top-sellers {
  @import "tailwindcss/utilities.css" source(none);
}
```

You will also need to include an `id` or `class` attribute in your card's view that matches the CSS selector in your entrypoint:

```
1<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">2    ...3</x-pulse::card>
<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">
    ...
</x-pulse::card>
```

### [Data Capture and Aggregation](#custom-card-data)

Custom cards may fetch and display data from anywhere; however, you may wish to leverage Pulse's powerful and efficient data recording and aggregation system.

#### [Capturing Entries](#custom-card-data-capture)

Pulse allows you to record "entries" using the `Pulse::record` method:

```
1use Laravel\Pulse\Facades\Pulse;2 3Pulse::record('user_sale', $user->id, $sale->amount)4    ->sum()5    ->count();
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

The first argument provided to the `record` method is the `type` for the entry you are recording, while the second argument is the `key` that determines how the aggregated data should be grouped. For most aggregation methods you will also need to specify a `value` to be aggregated. In the example above, the value being aggregated is `$sale->amount`. You may then invoke one or more aggregation methods (such as `sum`) so that Pulse may capture pre-aggregated values into "buckets" for efficient retrieval later.

The available aggregation methods are:

-   `avg`
-   `count`
-   `max`
-   `min`
-   `sum`

When building a card package that captures the currently authenticated user ID, you should use the `Pulse::resolveAuthenticatedUserId()` method, which respects any [user resolver customizations](#dashboard-resolving-users) made to the application.

#### [Retrieving Aggregate Data](#custom-card-data-retrieval)

When extending Pulse's `Card` Livewire component, you may use the `aggregate` method to retrieve aggregated data for the period being viewed in the dashboard:

```
1class TopSellers extends Card2{3    public function render()4    {5        return view('livewire.pulse.top-sellers', [6            'topSellers' => $this->aggregate('user_sale', ['sum', 'count'])7        ]);8    }9}
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers', [
            'topSellers' => $this->aggregate('user_sale', ['sum', 'count'])
        ]);
    }
}
```

The `aggregate` method returns a collection of PHP `stdClass` objects. Each object will contain the `key` property captured earlier, along with keys for each of the requested aggregates:

```
1@foreach ($topSellers as $seller)2    {{ $seller->key }}3    {{ $seller->sum }}4    {{ $seller->count }}5@endforeach
@foreach ($topSellers as $seller)
    {{ $seller->key }}
    {{ $seller->sum }}
    {{ $seller->count }}
@endforeach
```

Pulse will primarily retrieve data from the pre-aggregated buckets; therefore, the specified aggregates must have been captured up-front using the `Pulse::record` method. The oldest bucket will typically fall partially outside the period, so Pulse will aggregate the oldest entries to fill the gap and give an accurate value for the entire period, without needing to aggregate the entire period on each poll request.

You may also retrieve a total value for a given type by using the `aggregateTotal` method. For example, the following method would retrieve the total of all user sales instead of grouping them by user.

```
1$total = $this->aggregateTotal('user_sale', 'sum');
$total = $this->aggregateTotal('user_sale', 'sum');
```

#### [Displaying Users](#custom-card-displaying-users)

When working with aggregates that record a user ID as the key, you may resolve the keys to user records using the `Pulse::resolveUsers` method:

```
 1$aggregates = $this->aggregate('user_sale', ['sum', 'count']); 2  3$users = Pulse::resolveUsers($aggregates->pluck('key')); 4  5return view('livewire.pulse.top-sellers', [ 6    'sellers' => $aggregates->map(fn ($aggregate) => (object) [ 7        'user' => $users->find($aggregate->key), 8        'sum' => $aggregate->sum, 9        'count' => $aggregate->count, 10    ]) 11]);
$aggregates = $this->aggregate('user_sale', ['sum', 'count']);

$users = Pulse::resolveUsers($aggregates->pluck('key'));

return view('livewire.pulse.top-sellers', [
    'sellers' => $aggregates->map(fn ($aggregate) => (object) [
        'user' => $users->find($aggregate->key),
        'sum' => $aggregate->sum,
        'count' => $aggregate->count,
    ])
]);
```

The `find` method returns an object containing `name`, `extra`, and `avatar` keys, which you may optionally pass directly to the `<x-pulse::user-card>` Blade component:

```
1<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
```

#### [Custom Recorders](#custom-recorders)

Package authors may wish to provide recorder classes to allow users to configure the capturing of data.

Recorders are registered in the `recorders` section of the application's `config/pulse.php` configuration file:

```
 1[ 2    // ... 3    'recorders' => [ 4        Acme\Recorders\Deployments::class => [ 5            // ... 6        ], 7  8        // ... 9    ], 10]
[
    // ...
    'recorders' => [
        Acme\Recorders\Deployments::class => [
            // ...
        ],

        // ...
    ],
]
```

Recorders may listen to events by specifying a `$listen` property. Pulse will automatically register the listeners and call the recorders `record` method:

```
 1<?php 2  3namespace Acme\Recorders; 4  5use Acme\Events\Deployment; 6use Illuminate\Support\Facades\Config; 7use Laravel\Pulse\Facades\Pulse; 8  9class Deployments10{11    /**12     * The events to listen for.13     *14     * @var array<int, class-string>15     */16    public array $listen = [17        Deployment::class,18    ];19 20    /**21     * Record the deployment.22     */23    public function record(Deployment $event): void24    {25        $config = Config::get('pulse.recorders.'.static::class);26 27        Pulse::record(28            // ...29        );30    }31}
<?php

namespace Acme\Recorders;

use Acme\Events\Deployment;
use Illuminate\Support\Facades\Config;
use Laravel\Pulse\Facades\Pulse;

class Deployments
{
    /**
     * The events to listen for.
     *
     * @var array<int, class-string>
     */
    public array $listen = [
        Deployment::class,
    ];

    /**
     * Record the deployment.
     */
    public function record(Deployment $event): void
    {
        $config = Config::get('pulse.recorders.'.static::class);

        Pulse::record(
            // ...
        );
    }
}
```

Copy as markdown

### On this page

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Configuration](#configuration)
-   [Dashboard](#dashboard)
    -   [Authorization](#dashboard-authorization)
    -   [Customization](#dashboard-customization)
    -   [Resolving Users](#dashboard-resolving-users)
    -   [Cards](#dashboard-cards)
-   [Capturing Entries](#capturing-entries)
    -   [Recorders](#recorders)
    -   [Filtering](#filtering)
-   [Performance](#performance)
    -   [Using a Different Database](#using-a-different-database)
    -   [Redis Ingest](#ingest)
    -   [Sampling](#sampling)
    -   [Trimming](#trimming)
    -   [Handling Pulse Exceptions](#pulse-exceptions)
-   [Custom Cards](#custom-cards)
    -   [Card Components](#custom-card-components)
    -   [Styling](#custom-card-styling)
    -   [Data Capture and Aggregation](#custom-card-data)
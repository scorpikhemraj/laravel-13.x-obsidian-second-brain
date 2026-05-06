---
title: Laravel Telescope
description: Debugging and monitoring tool for Laravel applications
url: https://laravel.com/docs/13.x/telescope
tags: [packages]
cssclasses:
  - ai
  - color-purple
color: purple
---

# Laravel Telescope

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Local Only Installation](#local-only-installation)
    -   [Configuration](#configuration)
    -   [Data Pruning](#data-pruning)
    -   [Dashboard Authorization](#dashboard-authorization)
-   [Upgrading Telescope](#upgrading-telescope)
-   [Filtering](#filtering)
    -   [Entries](#filtering-entries)
    -   [Batches](#filtering-batches)
-   [Tagging](#tagging)
-   [Available Watchers](#available-watchers)
    -   [Batch Watcher](#batch-watcher)
    -   [Cache Watcher](#cache-watcher)
    -   [Command Watcher](#command-watcher)
    -   [Dump Watcher](#dump-watcher)
    -   [Event Watcher](#event-watcher)
    -   [Exception Watcher](#exception-watcher)
    -   [Gate Watcher](#gate-watcher)
    -   [HTTP Client Watcher](#http-client-watcher)
    -   [Job Watcher](#job-watcher)
    -   [Log Watcher](#log-watcher)
    -   [Mail Watcher](#mail-watcher)
    -   [Model Watcher](#model-watcher)
    -   [Notification Watcher](#notification-watcher)
    -   [Query Watcher](#query-watcher)
    -   [Redis Watcher](#redis-watcher)
    -   [Request Watcher](#request-watcher)
    -   [Schedule Watcher](#schedule-watcher)
    -   [View Watcher](#view-watcher)
-   [Displaying User Avatars](#displaying-user-avatars)

## [Introduction](#introduction)

[Laravel Telescope](https://github.com/laravel/telescope) makes a wonderful companion to your local Laravel development environment. Telescope provides insight into the requests coming into your application, exceptions, log entries, database queries, queued jobs, mail, notifications, cache operations, scheduled tasks, variable dumps, and more.

## [Installation](#installation)

You may use the Composer package manager to install Telescope into your Laravel project:

```bash
composer require laravel/telescope
```

After installing Telescope, publish its assets and migrations using the `telescope:install` Artisan command. After installing Telescope, you should also run the `migrate` command in order to create the tables needed to store Telescope's data:

```bash
php artisan telescope:install

php artisan migrate
```

Finally, you may access the Telescope dashboard via the `/telescope` route.

### [Local Only Installation](#local-only-installation)

If you plan to only use Telescope to assist your local development, you may install Telescope using the `--dev` flag:

```bash
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

After running `telescope:install`, you should remove the `TelescopeServiceProvider` service provider registration from your application's `bootstrap/providers.php` configuration file. Instead, manually register Telescope's service providers in the `register` method of your `App\Providers\AppServiceProvider` class. We will ensure the current environment is `local` before registering the providers:

```php
/**
 * Register any application services.
 */
public function register(): void
{
    if ($this->app->environment('local') && class_exists(\Laravel\Telescope\TelescopeServiceProvider::class)) {
        $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
        $this->app->register(TelescopeServiceProvider::class);
    }
}
```

Finally, you should also prevent the Telescope package from being [[05-digging-deeper/15-package-development.md#package-discovery|auto-discovered]] by adding the following to your `composer.json` file:

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "laravel/telescope"
        ]
    }
}
```

### [Configuration](#configuration)

After publishing Telescope's assets, its primary configuration file will be located at `config/telescope.php`. This configuration file allows you to configure your [watcher options](#available-watchers). Each configuration option includes a description of its purpose, so be sure to thoroughly explore this file.

If desired, you may disable Telescope's data collection entirely using the `enabled` configuration option:

```php
'enabled' => env('TELESCOPE_ENABLED', true),
```

### [Data Pruning](#data-pruning)

Without pruning, the `telescope_entries` table can accumulate records very quickly. To mitigate this, you should [[05-digging-deeper/21-task-scheduling.md|schedule]] the `telescope:prune` Artisan command to run daily:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune')->daily();
```

By default, all entries older than 24 hours will be pruned. You may use the `hours` option when calling the command to determine how long to retain Telescope data. For example, the following command will delete all records created over 48 hours ago:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune --hours=48')->daily();
```

### [Dashboard Authorization](#dashboard-authorization)

The Telescope dashboard may be accessed via the `/telescope` route. By default, you will only be able to access this dashboard in the `local` environment. Within your `app/Providers/TelescopeServiceProvider.php` file, there is an [[06-security/02-authorization.md#gates|authorization gate]] definition. This authorization gate controls access to Telescope in **non-local** environments. You are free to modify this gate as needed to restrict access to your Telescope installation:

```php
use App\Models\User;

/**
 * Register the Telescope gate.
 *
 * This gate determines who can access Telescope in non-local environments.
 */
protected function gate(): void
{
    Gate::define('viewTelescope', function (User $user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

You should ensure you change your `APP_ENV` environment variable to `production` in your production environment. Otherwise, your Telescope installation will be publicly available.

## [Upgrading Telescope](#upgrading-telescope)

When upgrading to a new major version of Telescope, it's important that you carefully review [the upgrade guide](https://github.com/laravel/telescope/blob/master/UPGRADE.md).

In addition, when upgrading to any new Telescope version, you should re-publish Telescope's assets:

```bash
php artisan telescope:publish
```

To keep the assets up-to-date and avoid issues in future updates, you may add the `vendor:publish --tag=laravel-assets` command to the `post-update-cmd` scripts in your application's `composer.json` file:

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

## [Filtering](#filtering)

### [Entries](#filtering-entries)

You may filter the data that is recorded by Telescope via the `filter` closure that is defined in your `App\Providers\TelescopeServiceProvider` class. By default, this closure records all data in the `local` environment and exceptions, failed jobs, scheduled tasks, and data with monitored tags in all other environments:

```php
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filter(function (IncomingEntry $entry) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entry->isReportableException() ||
            $entry->isFailedJob() ||
            $entry->isScheduledTask() ||
            $entry->isSlowQuery() ||
            $entry->hasMonitoredTag();
    });
}
```

### [Batches](#filtering-batches)

While the `filter` closure filters data for individual entries, you may use the `filterBatch` method to register a closure that filters all data for a given request or console command. If the closure returns `true`, all of the entries are recorded by Telescope:

```php
use Illuminate\Support\Collection;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filterBatch(function (Collection $entries) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entries->contains(function (IncomingEntry $entry) {
            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->isSlowQuery() ||
                $entry->hasMonitoredTag();
            });
    });
}
```

## [Tagging](#tagging)

Telescope allows you to search entries by "tag". Often, tags are Eloquent model class names or authenticated user IDs which Telescope automatically adds to entries. Occasionally, you may want to attach your own custom tags to entries. To accomplish this, you may use the `Telescope::tag` method. The `tag` method accepts a closure which should return an array of tags. The tags returned by the closure will be merged with any tags Telescope would automatically attach to the entry. Typically, you should call the `tag` method within the `register` method of your `App\Providers\TelescopeServiceProvider` class:

```php
use Laravel\Telescope\EntryType;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::tag(function (IncomingEntry $entry) {
        return $entry->type === EntryType::REQUEST
            ? ['status:'.$entry->content['response_status']]
            : [];
    });
}
```

## [Available Watchers](#available-watchers)

Telescope "watchers" gather application data when a request or console command is executed. You may customize the list of watchers that you would like to enable within your `config/telescope.php` configuration file:

```php
'watchers' => [
    Watchers\CacheWatcher::class => true,
    Watchers\CommandWatcher::class => true,
    // ...
],
```

Some watchers also allow you to provide additional customization options:

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 100,
    ],
    // ...
],
```

### [Batch Watcher](#batch-watcher)

The batch watcher records information about queued [[05-digging-deeper/17-queues.md#job-batching|batches]], including the job and connection information.

### [Cache Watcher](#cache-watcher)

The cache watcher records data when a cache key is hit, missed, updated and forgotten.

### [Command Watcher](#command-watcher)

The command watcher records the arguments, options, exit code, and output whenever an Artisan command is executed. If you would like to exclude certain commands from being recorded by the watcher, you may specify the command in the `ignore` option within your `config/telescope.php` file:

```php
'watchers' => [
    Watchers\CommandWatcher::class => [
        'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
        'ignore' => ['key:generate'],
    ],
    // ...
],
```

### [Dump Watcher](#dump-watcher)

The dump watcher records and displays your variable dumps in Telescope. When using Laravel, variables may be dumped using the global `dump` function. The dump watcher tab must be open in a browser for the dump to be recorded, otherwise, the dumps will be ignored by the watcher.

### [Event Watcher](#event-watcher)

The event watcher records the payload, listeners, and broadcast data for any [[05-digging-deeper/08-events.md|events]] dispatched by your application. The Laravel framework's internal events are ignored by the Event watcher.

### [Exception Watcher](#exception-watcher)

The exception watcher records the data and stack trace for any reportable exceptions that are thrown by your application.

### [Gate Watcher](#gate-watcher)

The gate watcher records the data and result of [[06-security/02-authorization.md|gate and policy]] checks by your application. If you would like to exclude certain abilities from being recorded by the watcher, you may specify those in the `ignore_abilities` option in your `config/telescope.php` file:

```php
'watchers' => [
    Watchers\GateWatcher::class => [
        'enabled' => env('TELESCOPE_GATE_WATCHER', true),
        'ignore_abilities' => ['viewNova'],
    ],
    // ...
],
```

### [HTTP Client Watcher](#http-client-watcher)

The HTTP client watcher records outgoing [[05-digging-deeper/11-http-client.md|HTTP client requests]] made by your application.

### [Job Watcher](#job-watcher)

The job watcher records the data and status of any [[05-digging-deeper/17-queues.md|jobs]] dispatched by your application.

### [Log Watcher](#log-watcher)

The log watcher records the [[04-the-basics/14-logging.md|log data]] for any logs written by your application.

By default, Telescope will only record logs at the `error` level and above. However, you can modify the `level` option in your application's `config/telescope.php` configuration file to modify this behavior:

```php
'watchers' => [
    Watchers\LogWatcher::class => [
        'enabled' => env('TELESCOPE_LOG_WATCHER', true),
        'level' => 'debug',
    ],

    // ...
],
```

### [Mail Watcher](#mail-watcher)

The mail watcher allows you to view an in-browser preview of [[05-digging-deeper/13-mail.md|emails]] sent by your application along with their associated data. You may also download the email as an `.eml` file.

### [Model Watcher](#model-watcher)

The model watcher records model changes whenever an Eloquent [[08-eloquent-orm/01-eloquent-getting-started.md#events|model event]] is dispatched. You may specify which model events should be recorded via the watcher's `events` option:

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
    ],
    // ...
],
```

If you would like to record the number of models hydrated during a given request, enable the `hydrations` option:

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
        'hydrations' => true,
    ],
    // ...
],
```

### [Notification Watcher](#notification-watcher)

The notification watcher records all [[05-digging-deeper/14-notifications.md|notifications]] sent by your application. If the notification triggers an email and you have the mail watcher enabled, the email will also be available for preview on the mail watcher screen.

### [Query Watcher](#query-watcher)

The query watcher records the raw SQL, bindings, and execution time for all queries that are executed by your application. The watcher also tags any queries slower than 100 milliseconds as `slow`. You may customize the slow query threshold using the watcher's `slow` option:

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 50,
    ],
    // ...
],
```

### [Redis Watcher](#redis-watcher)

The Redis watcher records all [[07-database/06-redis.md|Redis]] commands executed by your application. If you are using Redis for caching, cache commands will also be recorded by the Redis watcher.

### [Request Watcher](#request-watcher)

The request watcher records the request, headers, session, and response data associated with any requests handled by the application. You may limit your recorded response data via the `size_limit` (in kilobytes) option:

```php
'watchers' => [
    Watchers\RequestWatcher::class => [
        'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
        'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
    ],
    // ...
],
```

### [Schedule Watcher](#schedule-watcher)

The schedule watcher records the command and output of any [[05-digging-deeper/21-task-scheduling.md|scheduled tasks]] run by your application.

### [View Watcher](#view-watcher)

The view watcher records the [[04-the-basics/07-views.md|view]] name, path, data, and "composers" used when rendering views.

## [Displaying User Avatars](#displaying-user-avatars)

The Telescope dashboard displays the user avatar for the user that was authenticated when a given entry was saved. By default, Telescope will retrieve avatars using the Gravatar web service. However, you may customize the avatar URL by registering a callback in your `App\Providers\TelescopeServiceProvider` class. The callback will receive the user's ID and email address and should return the user's avatar image URL:

```php
use App\Models\User;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    // ...

    Telescope::avatar(function (?string $id, ?string $email) {
        return ! is_null($id)
            ? '/avatars/'.User::find($id)->avatar_path
            : '/generic-avatar.jpg';
    });
}
```
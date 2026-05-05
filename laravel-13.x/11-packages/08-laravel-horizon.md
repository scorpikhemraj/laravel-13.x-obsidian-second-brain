---
title: Laravel Horizon
description: Provides a beautiful dashboard and code-driven configuration for Laravel Redis queues.
url: https://laravel.com/docs/13.x/horizon
tags: [packages]
---

#Laravel Horizon

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Configuration](#configuration)
    -   [Dashboard Authorization](#dashboard-authorization)
    -   [Max Job Attempts](#max-job-attempts)
    -   [Job Timeout](#job-timeout)
    -   [Job Backoff](#job-backoff)
    -   [Silenced Jobs](#silenced-jobs)
-   [Balancing Strategies](#balancing-strategies)
    -   [Auto Balancing](#auto-balancing)
    -   [Simple Balancing](#simple-balancing)
    -   [No Balancing](#no-balancing)
-   [Upgrading Horizon](#upgrading-horizon)
-   [Running Horizon](#running-horizon)
    -   [Deploying Horizon](#deploying-horizon)
-   [Tags](#tags)
-   [Notifications](#notifications)
-   [Metrics](#metrics)
-   [Deleting Failed Jobs](#deleting-failed-jobs)
-   [Clearing Jobs From Queues](#clearing-jobs-from-queues)

## [Introduction](#introduction)

Before digging into Laravel Horizon, you should familiarize yourself with Laravel's base queue services. Horizon augments Laravel's queue with additional features.

[Laravel Horizon](https://github.com/laravel/horizon) provides a beautiful dashboard and code-driven configuration for your Laravel powered Redis queues. Horizon allows you to easily monitor key metrics of your queue system such as job throughput, runtime, and job failures.

When using Horizon, all of your queue worker configuration is stored in a single, simple configuration file.

## [Installation](#installation)

Laravel Horizon requires that you use Redis to power your queue. Therefore, you should ensure that your queue connection is set to `redis` in your application's `config/queue.php` configuration file.

You may install Horizon into your project using the Composer package manager:

```
composer require laravel/horizon
```

After installing Horizon, publish its assets using the `horizon:install` Artisan command:

```
php artisan horizon:install
```

### [Configuration](#configuration)

After publishing Horizon's assets, its primary configuration file will be located at `config/horizon.php`.

#### [Environments](#environments)

After installation, the primary Horizon configuration option to familiarize yourself with is the `environments` configuration option:

```
'environments' => [
    'production' => [
        'supervisor-1' => [
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],

    'local' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

#### [Supervisors](#supervisors)

Each environment can contain one or more "supervisors". Each supervisor is essentially responsible for "supervising" a group of worker processes.

#### [Maintenance Mode](#maintenance-mode)

While your application is in maintenance mode, queued jobs will not be processed by Horizon unless the supervisor's `force` option is defined:

```
'environments' => [
    'production' => [
        'supervisor-1' => [
            'force' => true,
        ],
    ],
],
```

### [Dashboard Authorization](#dashboard-authorization)

The Horizon dashboard may be accessed via the `/horizon` route. By default, you will only be able to access this dashboard in the `local` environment:

```
Gate::define('viewHorizon', function (User $user) {
    return in_array($user->email, [
        '[email protected]',
    ]);
});
```

### [Max Job Attempts](#max-job-attempts)

You can define the maximum number of attempts a job can consume within a supervisor's configuration:

```
'environments' => [
    'production' => [
        'supervisor-1' => [
            'tries' => 10,
        ],
    ],
],
```

### [Job Timeout](#job-timeout)

Similarly, you can set a `timeout` value at the supervisor level:

```
'environments' => [
    'production' => [
        'supervisor-1' => [
            'timeout' => 60,
        ],
    ],
],
```

### [Job Backoff](#job-backoff)

You can define the `backoff` value at the supervisor level:

```
'environments' => [
    'production' => [
        'supervisor-1' => [
            'backoff' => 10,
        ],
    ],
],
```

### [Silenced Jobs](#silenced-jobs)

Sometimes, you may not be interested in viewing certain jobs dispatched by your application:

```
'silenced' => [
    App\Jobs\ProcessPodcast::class,
],
```

## [Balancing Strategies](#balancing-strategies)

Each supervisor can process one or more queues and Horizon allows you to choose from three worker balancing strategies: `auto`, `simple`, and `false`.

### [Auto Balancing](#auto-balancing)

The `auto` strategy adjusts the number of worker processes per queue based on the current workload:

```
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default', 'notifications'],
            'balance' => 'auto',
            'autoScalingStrategy' => 'time',
            'minProcesses' => 1,
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],
],
```

### [Simple Balancing](#simple-balancing)

The `simple` strategy distributes worker processes evenly across the specified queues.

### [No Balancing](#no-balancing)

When the `balance` option is set to `false`, Horizon processes queues strictly in the order they're listed.

## [Upgrading Horizon](#upgrading-horizon)

When upgrading to a new major version of Horizon, it's important that you carefully review the upgrade guide.

## [Running Horizon](#running-horizon)

Once you have configured your supervisors and workers, you may start Horizon using the `horizon` Artisan command:

```
php artisan horizon
```

You may pause the Horizon process:

```
php artisan horizon:pause
php artisan horizon:continue
```

You may gracefully terminate the Horizon process:

```
php artisan horizon:terminate
```

#### [Deploying Horizon](#deploying-horizon)

When you're ready to deploy Horizon to your application's actual server, you should configure a process monitor to monitor the `php artisan horizon` command:

```
php artisan horizon:terminate
```

To start Horizon in production, you would typically install Supervisor:

```
sudo apt-get install supervisor
```

Supervisor configuration:

```
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

## [Tags](#tags)

Horizon allows you to assign "tags" to jobs. Horizon will intelligently and automatically tag most jobs depending on the Eloquent models that are attached to the job.

If you would like to manually define the tags for one of your queueable objects, you may define a `tags` method on the class:

```
public function tags(): array
{
    return ['render', 'video:'.$this->video->id];
}
```

## [Notifications](#notifications)

When configuring Horizon to send Slack or SMS notifications, you should review the prerequisites for the relevant notification channel:

```
Horizon::routeSmsNotificationsTo('15556667777');
Horizon::routeMailNotificationsTo('[email protected]');
Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
```

## [Metrics](#metrics)

Horizon includes a metrics dashboard. To populate this dashboard, you should configure Horizon's `snapshot` Artisan command to run every five minutes:

```
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

## [Deleting Failed Jobs](#deleting-failed-jobs)

If you would like to delete a failed job:

```
php artisan horizon:forget 5
```

To delete all failed jobs:

```
php artisan horizon:forget --all
```

## [Clearing Jobs From Queues](#clearing-jobs-from-queues)

If you would like to delete all jobs from your application's default queue:

```
php artisan horizon:clear
```

You may provide the `queue` option to delete jobs from a specific queue:

```
php artisan horizon:clear --queue=emails
```
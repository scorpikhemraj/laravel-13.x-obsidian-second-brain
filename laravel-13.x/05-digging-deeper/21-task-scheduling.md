---
title: Task Scheduling
description: Guide to Laravel's task scheduler for recurring jobs
url: https://laravel.com/docs/13.x/scheduling
tags: [logic]
---

# Task Scheduling

-   [Introduction](#introduction)
-   [Defining Schedules](#defining-schedules)
    -   [Scheduling Artisan Commands](#scheduling-artisan-commands)
    -   [Scheduling Queued Jobs](#scheduling-queued-jobs)
    -   [Scheduling Shell Commands](#scheduling-shell-commands)
    -   [Schedule Frequency Options](#schedule-frequency-options)
    -   [Timezones](#timezones)
    -   [Preventing Task Overlaps](#preventing-task-overlaps)
    -   [Running Tasks on One Server](#running-tasks-on-one-server)
    -   [Background Tasks](#background-tasks)
    -   [Maintenance Mode](#maintenance-mode)
    -   [Pausing Scheduled Tasks](#pausing-scheduled-tasks)
    -   [Schedule Groups](#schedule-groups)
-   [Running the Scheduler](#running-the-scheduler)
    -   [Sub-Minute Scheduled Tasks](#sub-minute-scheduled-tasks)
    -   [Running the Scheduler Locally](#running-the-scheduler-locally)
-   [Task Output](#task-output)
-   [Task Hooks](#task-hooks)
-   [Events](#events)

## [Introduction](#introduction)

In the past, you may have written a cron configuration entry for each task you needed to schedule on your server. However, this can quickly become a pain because your task schedule is no longer in source control and you must SSH into your server to view your existing cron entries or add additional entries.

Laravel's command scheduler offers a fresh approach to managing scheduled tasks on your server. The scheduler allows you to fluently and expressively define your command schedule within your Laravel application itself. When using the scheduler, only a single cron entry is needed on your server. Your task schedule is typically defined in your application's `routes/console.php` file.

## [Defining Schedules](#defining-schedules)

You may define all of your scheduled tasks in your application's `routes/console.php` file. To get started, let's take a look at an example. In this example, we will schedule a closure to be called every day at midnight. Within the closure we will execute a database query to clear a table:

```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

In addition to scheduling using closures, you may also schedule [invokable objects](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke):

```php
Schedule::call(new DeleteRecentUsers)->daily();
```

If you prefer to reserve your `routes/console.php` file for command definitions only, you may use the `withSchedule` method in your application's `bootstrap/app.php` file to define your scheduled tasks:

```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

If you would like to view an overview of your scheduled tasks and the next time they are scheduled to run, you may use the `schedule:list` Artisan command:

```bash
php artisan schedule:list
```

### [Scheduling Artisan Commands](#scheduling-artisan-commands)

In addition to scheduling closures, you may also schedule [[05-digging-deeper/01-artisan-console.md|Artisan commands]] and system commands:

```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

#### [Scheduling Artisan Closure Commands](#scheduling-artisan-closure-commands)

If you want to schedule an Artisan command defined by a closure, you may chain the scheduling related methods after the command's definition:

```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Delete recent users')->daily();
```

If you need to pass arguments to the closure command, you may provide them to the `schedule` method:

```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();
```

### [Scheduling Queued Jobs](#scheduling-queued-jobs)

The `job` method may be used to schedule a [[05-digging-deeper/17-queues.md|queued job]]. This method provides a convenient way to schedule queued jobs without using the `call` method to define closures to queue the job:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```

Optional second and third arguments may be provided to the `job` method which specifies the queue name and queue connection that should be used to queue the job:

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// Dispatch the job to the "heartbeats" queue on the "sqs" connection...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

### [Scheduling Shell Commands](#scheduling-shell-commands)

The `exec` method may be used to issue a command to the operating system:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

### [Schedule Frequency Options](#schedule-frequency-options)

We've already seen a few examples of how you may configure a task to run at specified intervals. However, there are many more task schedule frequencies that you may assign to a task:

| Method | Description |
|--------|-----------|
| `->cron('* * * * *');` | Run the task on a custom cron schedule. |
| `->everyMinute();` | Run the task every minute. |
| `->everyFiveMinutes();` | Run the task every five minutes. |
| `->everyTenMinutes();` | Run the task every ten minutes. |
| `->everyFifteenMinutes();` | Run the task every fifteen minutes. |
| `->everyThirtyMinutes();` | Run the task every thirty minutes. |
| `->hourly();` | Run the task every hour. |
| `->hourlyAt(17);` | Run the task every hour at 17 minutes past the hour. |
| `->daily();` | Run the task every day at midnight. |
| `->dailyAt('13:00');` | Run the task every day at 13:00. |
| `->twiceDaily(1, 13);` | Run the task daily at 1:00 & 13:00. |
| `->weekly();` | Run the task every Sunday at 00:00. |
| `->weeklyOn(1, '8:00');` | Run the task every week on Monday at 8:00. |
| `->monthly();` | Run the task on the first day of every month at 00:00. |
| `->monthlyOn(4, '15:00');` | Run the task every month on the 4th at 15:00. |
| `->quarterly();` | Run the task on the first day of every quarter at 00:00. |
| `->yearly();` | Run the task on the first day of every year at 00:00. |
| `->timezone('America/New_York');` | Set the timezone for the task. |

These methods may be combined with additional constraints to create even more finely tuned schedules:

```php
use Illuminate\Support\Facades\Schedule;

// Run once per week on Monday at 1 PM...
Schedule::call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
Schedule::command('foo')
    ->weekdays()
    ->hourly()
    ->timezone('America/Chicago')
    ->between('8:00', '17:00');
```

A list of additional schedule constraints:

| Method | Description |
|--------|-----------|
| `->weekdays();` | Limit the task to weekdays. |
| `->weekends();` | Limit the task to weekends. |
| `->sundays();` | Limit the task to Sunday. |
| `->mondays();` | Limit the task to Monday. |
| `->tuesdays();` | Limit the task to Tuesday. |
| `->wednesdays();` | Limit the task to Wednesday. |
| `->thursdays();` | Limit the task to Thursday. |
| `->fridays();` | Limit the task to Friday. |
| `->saturdays();` | Limit the task to Saturday. |
| `->days(array|mixed);` | Limit the task to specific days. |
| `->between($startTime, $endTime);` | Limit the task to run between start and end times. |
| `->unlessBetween($startTime, $endTime);` | Limit the task to not run between start and end times. |
| `->when(Closure);` | Limit the task based on a truth test. |
| `->environments($env);` | Limit the task to specific environments. |

#### [Day Constraints](#day-constraints)

The `days` method may be used to limit the execution of a task to specific days of the week:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```

Alternatively, you may use the constants available on the `Illuminate\Console\Scheduling\Schedule` class:

```php
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Support\Facades;

Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

#### [Between Time Constraints](#between-time-constraints)

The `between` method may be used to limit the execution of a task based on the time of day:

```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```

Similarly, the `unlessBetween` method can be used to exclude the execution of a task for a period of time:

```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

#### [Truth Test Constraints](#truth-test-constraints)

The `when` method may be used to limit the execution of a task based on the result of a given truth test:

```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```

The `skip` method may be seen as the inverse of `when`:

```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```

When using chained `when` methods, the scheduled command will only execute if all `when` conditions return `true`.

#### [Environment Constraints](#environment-constraints)

The `environments` method may be used to execute tasks only on the given environments:

```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

### [Timezones](#timezones)

Using the `timezone` method, you may specify that a scheduled task's time should be interpreted within a given timezone:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```

If you are repeatedly assigning the same timezone to all of your scheduled tasks, you can specify which timezone should be assigned to all schedules by defining a `schedule_timezone` option within your application's `app` configuration file:

```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```

Remember that some timezones utilize daylight savings time. When daylight saving time changes occur, your scheduled task may run twice or even not run at all.

### [Preventing Task Overlaps](#preventing-task-overlaps)

By default, scheduled tasks will be run even if the previous instance of the task is still running. To prevent this, you may use the `withoutOverlapping` method:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```

In this example, the `emails:send` Artisan command will be run every minute if it is not already running. The `withoutOverlapping` method is especially useful if you have tasks that vary drastically in their execution time.

If needed, you may specify how many minutes must pass before the "without overlapping" lock expires. By default, the lock will expire after 24 hours:

```php
Schedule::command('emails:send')->withoutOverlapping(10);
```

Behind the scenes, the `withoutOverlapping` method utilizes your application's [[05-digging-deeper/03-cache.md|cache]] to obtain locks.

### [Running Tasks on One Server](#running-tasks-on-one-server)

To utilize this feature, your application must be using the `database`, `memcached`, `dynamodb`, or `redis` cache driver.

If your application's scheduler is running on multiple servers, you may limit a scheduled job to only execute on a single server using the `onOneServer` method:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

You may use the `useCache` method to customize the cache store used by the scheduler:

```php
Schedule::useCache('database');
```

#### [Naming Single Server Jobs](#naming-unique-jobs)

Sometimes you may need to schedule the same job to be dispatched with different parameters, while still instructing Laravel to run each permutation of the job on a single server:

```php
Schedule::job(new CheckUptime('https://laravel.com'))
    ->name('check_uptime:laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();
```

Similarly, scheduled closures must be assigned a name if they are intended to be run on one server:

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

### [Background Tasks](#background-tasks)

By default, multiple tasks scheduled at the same time will execute sequentially based on the order they are defined in your `schedule` method. If you would like to run tasks in the background so that they may all run simultaneously, you may use the `runInBackground` method:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

The `runInBackground` method may only be used when scheduling tasks via the `command` and `exec` methods.

### [Maintenance Mode](#maintenance-mode)

Your application's scheduled tasks will not run when the application is in [[02-getting-started/02-configuration.md#maintenance-mode|maintenance mode]]. However, if you would like to force a task to run even in maintenance mode, you may call the `evenInMaintenanceMode` method when defining the task:

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

### [Pausing Scheduled Tasks](#pausing-scheduled-tasks)

You may temporarily pause scheduled task processing without changing your deployed code by using the `schedule:pause` Artisan command:

```bash
php artisan schedule:pause
```

While the scheduler is paused, no scheduled tasks will run. You may resume scheduled task processing using the `schedule:continue` command:

```bash
php artisan schedule:continue
```

If a task should still run while the scheduler is paused, you may mark it with the `evenWhenPaused` method:

```php
Schedule::command('emails:send')->evenWhenPaused();
```

### [Schedule Groups](#schedule-groups)

When defining multiple scheduled tasks with similar configurations, you can use Laravel's task grouping feature to avoid repeating the same settings for each task:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::daily()
    ->onOneServer()
    ->timezone('America/New_York')
    ->group(function () {
        Schedule::command('emails:send --force');
        Schedule::command('emails:prune');
    });
```

## [Running the Scheduler](#running-the-scheduler)

Now that we have learned how to define scheduled tasks, let's discuss how to actually run them on our server. The `schedule:run` Artisan command will evaluate all of your scheduled tasks and determine if they need to run based on the server's current time:

```bash
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

### [Sub-Minute Scheduled Tasks](#sub-minute-scheduled-tasks)

On most operating systems, cron jobs are limited to running a maximum of once per minute. However, Laravel's scheduler allows you to schedule tasks to run at more frequent intervals, even as often as once per second:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

When sub-minute tasks are defined within your application, the `schedule:run` command will continue running until the end of the current minute.

Since sub-minute tasks that take longer than expected to run could delay the execution of later sub-minute tasks, it is recommended that all sub-minute tasks dispatch queued jobs or background commands:

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

#### [Interrupting Sub-Minute Tasks](#interrupting-sub-minute-tasks)

To interrupt in-progress `schedule:run` invocations, you may add the `schedule:interrupt` command to your application's deployment script:

```bash
php artisan schedule:interrupt
```

### [Running the Scheduler Locally](#running-the-scheduler-locally)

Typically, you would not add a scheduler cron entry to your local development machine. Instead, you may use the `schedule:work` Artisan command:

```bash
php artisan schedule:work
```

## [Task Output](#task-output)

The Laravel scheduler provides several convenient methods for working with the output generated by scheduled tasks. First, using the `sendOutputTo` method, you may send the output to a file:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);
```

If you would like to append the output to a given file, you may use the `appendOutputTo` method:

```php
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);
```

Using the `emailOutputTo` method, you may email the output to an email address of your choice:

```php
Schedule::command('report:generate')
    ->daily()
    ->sendOutputTo($filePath)
    ->emailOutputTo('[email protected]');
```

If you only want to email the output if the scheduled Artisan or system command terminates with a non-zero exit code, use the `emailOutputOnFailure` method:

```php
Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('[email protected]');
```

## [Task Hooks](#task-hooks)

Using the `before` and `after` methods, you may specify code to be executed before and after the scheduled task is executed:

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->before(function () {
        // The task is about to execute...
    })
    ->after(function () {
        // The task has executed...
    });
```

The `onSuccess` and `onFailure` methods allow you to specify code to be executed if the scheduled task succeeds or fails:

```php
Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function () {
        // The task succeeded...
    })
    ->onFailure(function () {
        // The task failed...
    });
```

If output is available from your command, you may access it in your hooks:

```php
use Illuminate\Support\Stringable;

Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function (Stringable $output) {
        // The task succeeded...
    })
    ->onFailure(function (Stringable $output) {
        // The task failed...
    });
```

#### [Pinging URLs](#pinging-urls)

Using the `pingBefore` and `thenPing` methods, the scheduler can automatically ping a given URL before or after a task is executed:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url);
```

The `pingOnSuccess` and `pingOnFailure` methods may be used to ping a given URL only if the task succeeds or fails:

```php
Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

## [Events](#events)

Laravel dispatches a variety of [[05-digging-deeper/08-events.md|events]] during the scheduling process:

| Event Name |
|----------|
| `Illuminate\Console\Events\ScheduledTaskStarting` |
| `Illuminate\Console\Events\ScheduledTaskFinished` |
| `Illuminate\Console\Events\ScheduledBackgroundTaskFinished` |
| `Illuminate\Console\Events\ScheduledTaskSkipped` |
| `Illuminate\Console\Events\ScheduledTaskFailed` |
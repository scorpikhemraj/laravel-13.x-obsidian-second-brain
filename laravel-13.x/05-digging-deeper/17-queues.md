---
title: Queues
description: Guide to Laravel queue system for processing background jobs
url: https://laravel.com/docs/13.x/queues
tags: [logic]
---

# Queues

-   [Introduction](#introduction)
    -   [Connections vs. Queues](#connections-vs-queues)
    -   [Driver Notes and Prerequisites](#driver-prerequisites)
-   [Creating Jobs](#creating-jobs)
    -   [Generating Job Classes](#generating-job-classes)
    -   [Class Structure](#class-structure)
    -   [Unique Jobs](#unique-jobs)
    -   [Debounced Jobs](#debounced-jobs)
    -   [Encrypted Jobs](#encrypted-jobs)
-   [Job Middleware](#job-middleware)
    -   [Rate Limiting](#rate-limiting)
    -   [Preventing Job Overlaps](#preventing-job-overlaps)
    -   [Throttling Exceptions](#throttling-exceptions)
    -   [Skipping Jobs](#skipping-jobs)
-   [Dispatching Jobs](#dispatching-jobs)
    -   [Delayed Dispatching](#delayed-dispatching)
    -   [Synchronous Dispatching](#synchronous-dispatching)
    -   [Jobs & Database Transactions](#jobs-and-database-transactions)
    -   [Job Chaining](#job-chaining)
    -   [Customizing The Queue and Connection](#customizing-the-queue-and-connection)
    -   [Specifying Max Job Attempts / Timeout Values](#max-job-attempts-and-timeout)
    -   [SQS FIFO and Fair Queues](#sqs-fifo-and-fair-queues)
    -   [Queue Failover](#queue-failover)
    -   [Error Handling](#error-handling)
-   [Job Batching](#job-batching)
    -   [Defining Batchable Jobs](#defining-batchable-jobs)
    -   [Dispatching Batches](#dispatching-batches)
    -   [Chains and Batches](#chains-and-batches)
    -   [Adding Jobs to Batches](#adding-jobs-to-batches)
    -   [Inspecting Batches](#inspecting-batches)
    -   [Cancelling Batches](#cancelling-batches)
    -   [Batch Failures](#batch-failures)
    -   [Pruning Batches](#pruning-batches)
    -   [Storing Batches in DynamoDB](#storing-batches-in-dynamodb)
-   [Queueing Closures](#queueing-closures)
-   [Running the Queue Worker](#running-the-queue-worker)
    -   [The `queue:work` Command](#the-queue-work-command)
    -   [Queue Priorities](#queue-priorities)
    -   [Queue Workers and Deployment](#queue-workers-and-deployment)
    -   [Reacting to Worker Signals](#reacting-to-worker-signals)
    -   [Job Expirations and Timeouts](#job-expirations-and-timeouts)
    -   [Pausing and Resuming Queue Workers](#pausing-and-resuming-queue-workers)
-   [Supervisor Configuration](#supervisor-configuration)
-   [Dealing With Failed Jobs](#dealing-with-failed-jobs)
    -   [Cleaning Up After Failed Jobs](#cleaning-up-after-failed-jobs)
    -   [Retrying Failed Jobs](#retrying-failed-jobs)
    -   [Ignoring Missing Models](#ignoring-missing-models)
    -   [Pruning Failed Jobs](#pruning-failed-jobs)
    -   [Storing Failed Jobs in DynamoDB](#storing-failed-jobs-in-dynamodb)
    -   [Disabling Failed Job Storage](#disabling-failed-job-storage)
    -   [Failed Job Events](#failed-job-events)
-   [Clearing Jobs From Queues](#clearing-jobs-from-queues)
-   [Monitoring Your Queues](#monitoring-your-queues)
-   [Testing](#testing)
    -   [Faking a Subset of Jobs](#faking-a-subset-of-jobs)
    -   [Testing Job Chains](#testing-job-chains)
    -   [Testing Job Batches](#testing-job-batches)
    -   [Testing Job / Queue Interactions](#testing-job-queue-interactions)
-   [Job Events](#job-events)

## [Introduction](#introduction)

While building your web application, you may have some tasks, such as parsing and storing an uploaded CSV file, that take too long to perform during a typical web request. Thankfully, Laravel allows you to easily create queued jobs that may be processed in the background. By moving time intensive tasks to a queue, your application can respond to web requests with blazing speed and provide a better user experience to your customers.

Laravel queues provide a unified queueing API across a variety of different queue backends, such as [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), or even a relational database.

Laravel's queue configuration options are stored in your application's `config/queue.php` configuration file. In this file, you will find connection configurations for each of the queue drivers that are included with the framework, including the database, [Amazon SQS](https://aws.amazon.com/sqs/), [Redis](https://redis.io), and [Beanstalkd](https://beanstalkd.github.io/) drivers, as well as a synchronous driver that will execute jobs immediately (for use during development or testing). A `null` queue driver is also included which discards queued jobs.

Laravel Horizon is a beautiful dashboard and configuration system for your Redis powered queues. Check out the full [Horizon documentation](/docs/13.x/horizon) for more information.

### [Connections vs. Queues](#connections-vs-queues)

Before getting started with Laravel queues, it is important to understand the distinction between "connections" and "queues". In your `config/queue.php` configuration file, there is a `connections` configuration array. This option defines the connections to backend queue services such as Amazon SQS, Beanstalk, or Redis. However, any given queue connection may have multiple "queues" which may be thought of as different stacks or piles of queued jobs.

Note that each connection configuration example in the `queue` configuration file contains a `queue` attribute. This is the default queue that jobs will be dispatched to when they are sent to a given connection. In other words, if you dispatch a job without explicitly defining which queue it should be dispatched to, the job will be placed on the queue that is defined in the `queue` attribute of the connection configuration:

```php
use App\Jobs\ProcessPodcast;

// This job is sent to the default connection's default queue...
ProcessPodcast::dispatch();

// This job is sent to the default connection's "emails" queue...
ProcessPodcast::dispatch()->onQueue('emails');
```

Some applications may not need to ever push jobs onto multiple queues, instead preferring to have one simple queue. However, pushing jobs to multiple queues can be especially useful for applications that wish to prioritize or segment how jobs are processed, since the Laravel queue worker allows you to specify which queues it should process by priority. For example, if you push jobs to a `high` queue, you may run a worker that gives them higher processing priority:

```bash
php artisan queue:work --queue=high,default
```

### [Driver Notes and Prerequisites](#driver-prerequisites)

#### [Database](#database)

In order to use the `database` queue driver, you will need a database table to hold the jobs. Typically, this is included in Laravel's default `0001_01_01_000002_create_jobs_table.php` [[07-database/04-migrations.md|database migration]]; however, if your application does not contain this migration, you may use the `make:queue-table` Artisan command to create it:

```bash
php artisan make:queue-table

php artisan migrate
```

#### [Redis](#redis)

In order to use the `redis` queue driver, you should configure a Redis database connection in your `config/database.php` configuration file.

The `serializer` and `compression` Redis options are not supported by the `redis` queue driver.

##### [Redis Cluster](#redis-cluster)

If your Redis queue connection uses a [Redis Cluster](https://redis.io/docs/latest/operate/rs/databases/durability-ha/clustering), your queue names must contain a [key hash tag](https://redis.io/docs/latest/develop/using-commands/keyspace/#hashtags). This is required in order to ensure all of the Redis keys for a given queue are placed into the same hash slot:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', '{default}'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => null,
    'after_commit' => false,
],
```

##### [Blocking](#blocking)

When using the Redis queue, you may use the `block_for` configuration option to specify how long the driver should wait for a job to become available before iterating through the worker loop and re-polling the Redis database.

Adjusting this value based on your queue load can be more efficient than continually polling the Redis database for new jobs. For instance, you may set the value to `5` to indicate that the driver should block for five seconds while waiting for a job to become available:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => 5,
    'after_commit' => false,
],
```

Setting `block_for` to `0` will cause queue workers to block indefinitely until a job is available. This will also prevent signals such as `SIGTERM` from being handled until the next job has been processed.

#### [Other Driver Prerequisites](#other-driver-prerequisites)

The following dependencies are needed for the listed queue drivers. These dependencies may be installed via the Composer package manager:

-   Amazon SQS: `aws/aws-sdk-php ~3.0`
-   Beanstalkd: `pda/pheanstalk ~5.0`
-   Redis: `predis/predis ~2.0` or phpredis PHP extension
-   [MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/): `mongodb/laravel-mongodb`

## [Creating Jobs](#creating-jobs)

### [Generating Job Classes](#generating-job-classes)

By default, all of the queueable jobs for your application are stored in the `app/Jobs` directory. If the `app/Jobs` directory doesn't exist, it will be created when you run the `make:job` Artisan command:

```bash
php artisan make:job ProcessPodcast
```

The generated class will implement the `Illuminate\Contracts\Queue\ShouldQueue` interface, indicating to Laravel that the job should be pushed onto the queue to run asynchronously.

Job stubs may be customized using [[05-digging-deeper/01-artisan-console.md#stub-customization|stub publishing]].

### [Class Structure](#class-structure)

Job classes are very simple, normally containing only a `handle` method that is invoked when the job is processed by the queue. To get started, let's take a look at an example job class. In this example, we'll pretend we manage a podcast publishing service and need to process the uploaded podcast files before they are published:

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }
}
```

In this example, note that we were able to pass an [[08-eloquent-orm/01-eloquent-getting-started.md|Eloquent model]] directly into the queued job's constructor. Because of the `Queueable` trait that the job is using, Eloquent models and their loaded relationships will be gracefully serialized and unserialized when the job is processing.

If your queued job accepts an Eloquent model in its constructor, only the identifier for the model will be serialized onto the queue. When the job is actually handled, the queue system will automatically re-retrieve the full model instance and its loaded relationships from the database. This approach to model serialization allows for much smaller job payloads to be sent to your queue driver.

#### [`handle` Method Dependency Injection](#handle-method-dependency-injection)

The `handle` method is invoked when the job is processed by the queue. Note that we are able to type-hint dependencies on the `handle` method of the job. The Laravel [[03-architecture-concepts/02-service-container.md|service container]] automatically injects these dependencies.

If you would like to take total control over how the container injects dependencies into the `handle` method, you may use the container's `bindMethod` method. The `bindMethod` method accepts a callback which receives the job and the container. Within the callback, you are free to invoke the `handle` method however you wish. Typically, you should call this method from the `boot` method of your `App\Providers\AppServiceProvider` [[03-architecture-concepts/03-service-providers.md|service provider]]:

```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

Binary data, such as raw image contents, should be passed through the `base64_encode` function before being passed to a queued job. Otherwise, the job may not properly serialize to JSON when being placed on the queue.

#### [Queued Relationships](#handling-relationships)

Because all loaded Eloquent model relationships also get serialized when a job is queued, the serialized job string can sometimes become quite large. Furthermore, when a job is deserialized and model relationships are re-retrieved from the database, they will be retrieved in their entirety. Any previous relationship constraints that were applied before the model was serialized during the job queueing process will not be applied when the job is deserialized. Therefore, if you wish to work with a subset of a given relationship, you should re-constrain that relationship within your queued job.

Or, to prevent relations from being serialized, you can call the `withoutRelations` method on the model when setting a property value. This method will return an instance of the model without its loaded relationships:

```php
/**
 * Create a new job instance.
 */
public function __construct(
    Podcast $podcast,
) {
    $this->podcast = $podcast->withoutRelations();
}
```

If you only need to remove specific relations while keeping the rest, you may use the `withoutRelation` method:

```php
$this->podcast = $podcast->withoutRelation('comments');
```

If you are using [PHP constructor property promotion](https://www.php.net/manual/en/language.oop5.decon.php#language.oop5.decon.constructor.promotion) and would like to indicate that an Eloquent model should not have its relations serialized, you may use the `WithoutRelations` attribute:

```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * Create a new job instance.
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast,
) {}
```

For convenience, if you wish to serialize all models without relationships, you may apply the `WithoutRelations` attribute to the entire class instead of applying the attribute to each model:

```php
<?php

namespace App\Jobs;

use App\Models\DistributionPlatform;
use App\Models\Podcast;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\WithoutRelations;

#[WithoutRelations]
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
        public DistributionPlatform $platform,
    ) {}
}
```

If a job receives a collection or array of Eloquent models instead of a single model, the models within that collection will not have their relationships restored when the job is deserialized and executed. This is to prevent excessive resource usage on jobs that deal with large numbers of models.

### [Unique Jobs](#unique-jobs)

Unique jobs require a cache driver that supports [[05-digging-deeper/03-cache.md#atomic-locks|locks]]. Currently, the `memcached`, `redis`, `dynamodb`, `database`, `file`, and `array` cache drivers support atomic locks.

Unique job constraints do not apply to jobs within batches.

Sometimes, you may want to ensure that only one instance of a specific job is on the queue at any point in time. You may do so by implementing the `ShouldBeUnique` interface on your job class. This interface does not require you to define any additional methods on your class:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...
}
```

In the example above, the `UpdateSearchIndex` job is unique. So, the job will not be dispatched if another instance of the job is already on the queue and has not finished processing.

In certain cases, you may want to define a specific "key" that makes the job unique or you may want to specify a timeout beyond which the job no longer stays unique. To accomplish this, you may use the `UniqueFor` attribute and define a `uniqueId` method on your job class:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Queue\Attributes\UniqueFor;

#[UniqueFor(3600)]
class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    /**
     * The product instance.
     *
     * @var \App\Models\Product
     */
    public $product;

    /**
     * Get the unique ID for the job.
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```

In the example above, the `UpdateSearchIndex` job is unique by a product ID. So, any new dispatches of the job with the same product ID will be ignored until the existing job has completed processing. In addition, if the existing job is not processed within one hour, the unique lock will be released and another job with the same unique key can be dispatched to the queue.

If your application dispatches jobs from multiple web servers or containers, you should ensure that all of your servers are communicating with the same central cache server so that Laravel can accurately determine if a job is unique.

#### [Keeping Jobs Unique Until Processing Begins](#keeping-jobs-unique-until-processing-begins)

By default, unique jobs are "unlocked" after a job completes processing or fails all of its retry attempts. However, there may be situations where you would like your job to unlock immediately before it is processed. To accomplish this, your job should implement the `ShouldBeUniqueUntilProcessing` contract instead of the `ShouldBeUnique` contract:

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

#### [Unique Job Locks](#unique-job-locks)

Behind the scenes, when a `ShouldBeUnique` job is dispatched, Laravel attempts to acquire a [[05-digging-deeper/03-cache.md#atomic-locks|lock]] with the `uniqueId` key. If the lock is already held, the job is not dispatched. This lock is released when the job completes processing or fails all of its retry attempts. By default, Laravel will use the default cache driver to obtain this lock. However, if you wish to use another driver for acquiring the lock, you may define a `uniqueVia` method that returns the cache driver that should be used:

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...

    /**
     * Get the cache driver for the unique job lock.
     */
    public function uniqueVia(): Repository
    {
        return Cache::driver('redis');
    }
}
```

If you only need to limit the concurrent processing of a job, use the [[05-digging-deeper/17-queues.md#preventing-job-overlaps|WithoutOverlapping]] job middleware instead.

### [Debounced Jobs](#debounced-jobs)

Sometimes, you may want to ensure that when the same job is dispatched many times in a short window, only the latest dispatch actually executes. You may do so by adding the `DebounceFor` attribute to your job:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\Attributes\DebounceFor;

#[DebounceFor(30)]
class UpdateSearchIndex implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(public int $productId)
    {
    }

    /**
     * Get the debounce ID for the job.
     */
    public function debounceId(): string
    {
        return (string) $this->productId;
    }
}
```

In the example above, repeatedly dispatching `UpdateSearchIndex` for the same product within `30` seconds will debounce the job so that only the latest dispatch runs.

If you would like to cap how long a frequently re-dispatched job can be deferred, you may provide the `maxWait` argument to the `DebounceFor` attribute:

```php
#[DebounceFor(30, maxWait: 120)]
class UpdateSearchIndex implements ShouldQueue
{
    use Queueable;

    // ...
}
```

You may customize the cache store used for debounce tracking by defining a `debounceVia` method on your job:

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

public function debounceVia(): Repository
{
    return Cache::driver('redis');
}
```

If a debounced job is superseded by a newer dispatch, Laravel will dispatch the `Illuminate\Queue\Events\JobDebounced` event and remove the superseded job from the queue.

Debounced jobs and unique jobs are mutually exclusive. A job using the `DebounceFor` attribute should not implement `ShouldBeUnique`.

If your application dispatches debounced jobs from multiple web servers or containers, you should ensure that all of your servers are communicating with the same central cache server.

### [Encrypted Jobs](#encrypted-jobs)

Laravel allows you to ensure the privacy and integrity of a job's data via [[06-security/04-encryption.md|encryption]]. To get started, simply add the `ShouldBeEncrypted` interface to the job class. Once this interface has been added to the class, Laravel will automatically encrypt your job before pushing it onto a queue:

```php
<?php

use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

## [Job Middleware](#job-middleware)

Job middleware allow you to wrap custom logic around the execution of queued jobs, reducing boilerplate in the jobs themselves. For example, consider the following `handle` method which leverages Laravel's Redis rate limiting features to allow only one job to process every five seconds:

```php
use Illuminate\Support\Facades\Redis;

/**
 * Execute the job.
 */
public function handle(): void
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('Lock obtained...');

        // Handle job...
    }, function () {
        // Could not obtain lock...

        return $this->release(5);
    });
}
```

While this code is valid, the implementation of the `handle` method becomes noisy since it is cluttered with Redis rate limiting logic. In addition, this rate limiting logic must be duplicated for any other jobs that we want to rate limit. Instead of rate limiting in the handle method, we could define a job middleware that handles rate limiting:

```php
<?php

namespace App\Jobs\Middleware;

use Closure;
use Illuminate\Support\Facades\Redis;

class RateLimited
{
    /**
     * Process the queued job.
     *
     * @param  \Closure(object): void  $next
     */
    public function handle(object $job, Closure $next): void
    {
        Redis::throttle('key')
            ->block(0)->allow(1)->every(5)
            ->then(function () use ($job, $next) {
                // Lock obtained...

                $next($job);
            }, function () use ($job) {
                // Could not obtain lock...

                $job->release(5);
            });
    }
}
```

As you can see, like [[04-the-basics/02-middleware.md|route middleware]], job middleware receive the job being processed and a callback that should be invoked to continue processing the job.

You can generate a new job middleware class using the `make:job-middleware` Artisan command. After creating job middleware, they may be attached to a job by returning them from the job's `middleware` method. This method does not exist on jobs scaffolded by the `make:job` Artisan command, so you will need to manually add it to your job class:

```php
use App\Jobs\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited];
}
```

Job middleware can also be assigned to [[05-digging-deeper/08-events.md#queued-event-listeners|queueable event listeners]], [[05-digging-deeper/13-mail.md#queueing-mail|mailables]], and [[05-digging-deeper/14-notifications.md#queueing-notifications|notifications]].

### [Rate Limiting](#rate-limiting)

Although we just demonstrated how to write your own rate limiting job middleware, Laravel actually includes a rate limiting middleware that you may utilize to rate limit jobs. Like [[04-the-basics/01-routing.md#defining-rate-limiters|route rate limiters]], job rate limiters are defined using the `RateLimiter` facade's `for` method.

For example, you may wish to allow users to backup their data once per hour while imposing no such limit on premium customers. To accomplish this, you may define a `RateLimiter` in the `boot` method of your `AppServiceProvider`:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    RateLimiter::for('backups', function (object $job) {
        return $job->user->vipCustomer()
            ? Limit::none()
            : Limit::perHour(1)->by($job->user->id);
    });
}
```

In the example above, we defined an hourly rate limit; however, you may easily define a rate limit based on minutes using the `perMinute` method. In addition, you may pass any value you wish to the `by` method of the rate limit; however, this value is most often used to segment rate limits by customer:

```php
return Limit::perMinute(50)->by($job->user->id);
```

Once you have defined your rate limit, you may attach the rate limiter to your job using the `Illuminate\Queue\Middleware\RateLimited` middleware. Each time the job exceeds the rate limit, this middleware will release the job back to the queue with an appropriate delay based on the rate limit duration:

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited('backups')];
}
```

Releasing a rate limited job back onto the queue will still increment the job's total number of `attempts`. You may wish to tune your `Tries` and `MaxExceptions` attributes on your job class accordingly. Or, you may wish to use the [retryUntil method](#time-based-attempts) to define the amount of time until the job should no longer be attempted.

Using the `releaseAfter` method, you may also specify the number of seconds that must elapse before the released job will be attempted again:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->releaseAfter(60)];
}
```

If you do not want a job to be retried when it is rate limited, you may use the `dontRelease` method:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->dontRelease()];
}
```

#### [Rate Limiting With Redis](#rate-limiting-with-redis)

If you are using Redis, you may use the `Illuminate\Queue\Middleware\RateLimitedWithRedis` middleware, which is fine-tuned for Redis and more efficient than the basic rate limiting middleware:

```php
use Illuminate\Queue\Middleware\RateLimitedWithRedis;

public function middleware(): array
{
    return [new RateLimitedWithRedis('backups')];
}
```

The `connection` method may be used to specify which Redis connection the middleware should use:

```php
return [(new RateLimitedWithRedis('backups'))->connection('limiter')];
```

### [Preventing Job Overlaps](#preventing-job-overlaps)

Laravel includes an `Illuminate\Queue\Middleware\WithoutOverlapping` middleware that allows you to prevent job overlaps based on an arbitrary key. This can be helpful when a queued job is modifying a resource that should only be modified by one job at a time.

For example, let's imagine you have a queued job that updates a user's credit score and you want to prevent credit score update job overlaps for the same user ID. To accomplish this, you can return the `WithoutOverlapping` middleware from your job's `middleware` method:

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

Releasing an overlapping job back onto the queue will still increment the job's total number of attempts. You may wish to tune your `Tries` and `MaxExceptions` attributes on your job class accordingly. For example, leaving `Tries` to 1 as it is by default would prevent any overlapping job from being retried later.

Any overlapping jobs of the same type will be released back to the queue. You may also specify the number of seconds that must elapse before the released job will be attempted again:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
}
```

If you wish to immediately delete any overlapping jobs so that they will not be retried, you may use the `dontRelease` method:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->dontRelease()];
}
```

The `WithoutOverlapping` middleware is powered by Laravel's atomic lock feature. Sometimes, your job may unexpectedly fail or timeout in such a way that the lock is not released. Therefore, you may explicitly define a lock expiration time using the `expireAfter` method. For example, the example below will instruct Laravel to release the `WithoutOverlapping` lock three minutes after the job has started processing:

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
}
```

The `WithoutOverlapping` middleware requires a cache driver that supports [[05-digging-deeper/03-cache.md#atomic-locks|locks]]. Currently, the `memcached`, `redis`, `dynamodb`, `database`, `file`, and `array` cache drivers support atomic locks.

#### [Sharing Lock Keys Across Job Classes](#sharing-lock-keys)

By default, the `WithoutOverlapping` middleware will only prevent overlapping jobs of the same class. So, although two different job classes may use the same lock key, they will not be prevented from overlapping. However, you can instruct Laravel to apply the key across job classes using the `shared` method:

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProviderIsDown
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}

class ProviderIsUp
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}
```

### [Throttling Exceptions](#throttling-exceptions)

Laravel includes a `Illuminate\Queue\Middleware\ThrottlesExceptions` middleware that allows you to throttle exceptions. Once the job throws a given number of exceptions, all further attempts to execute the job are delayed until a specified time interval lapses. This middleware is particularly useful for jobs that interact with third-party services that are unstable.

For example, let's imagine a queued job that interacts with a third-party API that begins throwing exceptions. To throttle exceptions, you can return the `ThrottlesExceptions` middleware from your job's `middleware` method. Typically, this middleware should be paired with a job that implements [time based attempts](#time-based-attempts):

```php
use DateTime;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new ThrottlesExceptions(10, 5 * 60)];
}

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 30);
}
```

The first constructor argument accepted by the middleware is the number of exceptions the job can throw before being throttled, while the second constructor argument is the number of seconds that should elapse before the job is attempted again once it has been throttled. In the code example above, if the job throws 10 consecutive exceptions, we will wait 5 minutes before attempting the job again, constrained by the 30-minute time limit.

When a job throws an exception but the exception threshold has not yet been reached, the job will typically be retried immediately. However, you may specify the number of minutes such a job should be delayed by calling the `backoff` method when attaching the middleware to the job:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5 * 60))->backoff(5)];
}
```

Internally, this middleware uses Laravel's cache system to implement rate limiting, and the job's class name is utilized as the cache "key". You may override this key by calling the `by` method when attaching the middleware to your job. This may be useful if you have multiple jobs interacting with the same third-party service and you would like them to share a common throttling "bucket" ensuring they respect a single shared limit:

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->by('key')];
}
```

By default, this middleware will throttle every exception. You can modify this behavior by invoking the `when` method when attaching the middleware to your job. The exception will then only be throttled if the closure provided to the `when` method returns `true`:

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->when(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

Unlike the `when` method, which releases the job back onto the queue or throws an exception, the `deleteWhen` method allows you to delete the job entirely when a given exception occurs:

```php
use App\Exceptions\CustomerDeletedException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(2, 10 * 60))->deleteWhen(CustomerDeletedException::class)];
}
```

### [Skipping Jobs](#skipping-jobs)

Sometimes you may wish to determine that a job should be skipped (deleted from the queue) based on some condition. You may accomplish this using the `Skip` middleware:

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [
        new Skip(fn () => $this->user->isBanned()),
    ];
}
```

If a job is skipped, it will not be retried. If you would like to skip a job but still count the attempt toward the job's `Tries` count, use the `SkipButCountAttempt` middleware instead:

```php
use Illuminate\Queue\Middleware\SkipButCountAttempt;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [
        new SkipButCountAttempt(fn () => $this->user->isBanned()),
    ];
}
```

## [Dispatching Jobs](#dispatching-jobs)

Once you have written your job class, you may dispatch it using the `dispatch` function or the `Dispatch` facade. The only argument the `dispatch` function accepts is an instance of the job:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast);
```

### [Delayed Dispatching](#delayed-dispatching)

If you would like to specify that the job should not be immediately available for processing, you may use the `delay` method when dispatching the job. For example, we will specify that a job should not be processed until 10 minutes after it has been queued:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)
    ->delay(now()->addMinutes(10));
```

### [Synchronous Dispatching](#synchronous-dispatching)

If you would like to dispatch a job immediately so that it runs synchronously (rather than being queued), you may use the `dispatchSync` method. When using this method, the job will not be queued and will instead be executed immediately by the queue worker:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatchSync($podcast);
```

### [Jobs & Database Transactions](#jobs-and-database-transactions)

When jobs are dispatched within database transactions, the jobs are typically processed by the queue worker once the database commits the transactions. This behavior is great because it ensures your dispatched jobs actually exist in the database. However, there may be scenarios where you need to ensure the job is processed before the database transaction is committed. To accomplish this, you may chain the `afterCommit` method when dispatching the job:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterCommit();
```

Alternatively, you may execute the callback provided to the `afterDispatch` method after all pending database transactions have been committed:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterDispatch(function () {
    // ...
});
```

If you are dispatching jobs within database transactions and you would like Laravel to always dispatch the job after all open database transactions have been committed, you may set the `after_commit` option to `true` in your `config/queue.php` configuration file:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'default',
    'retry_after' => 90,
    'after_commit' => true,
],
```

### [Job Chaining](#job-chaining)

Laravel's job chaining feature allows you to specify a list of queued jobs that should be run in sequence. If one job in the sequence fails, the rest of the jobs will not be executed:

```php
use App\Jobs\ProcessPodcast;
use App\Jobs\OptimizePodcast;
use App\Jobs\PublishPodcast;

ProcessPodcast::withChain([
    new OptimizePodcast($podcast),
    new PublishPodcast($podcast),
])->dispatch();
```

### [Customizing The Queue and Connection](#customizing-the-queue-and-connection)

#### [Dispatching To A Specific Queue](#dispatching-to-a-specific-queue)

You may also specify which queue a job should be dispatched to. Workers can be configured to process jobs from specific queues, allowing you to separate workers to define priority. This does not push jobs to different "queues" as workers are separate processes; instead, it simply tells workers to prioritize processing from the specified queue:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->onQueue('processing');
```

By default, queued jobs are dispatched to the `default` queue connection. If you would like to specify a different connection, you may invoke the `onConnection` method when dispatching the job:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)
    ->onConnection('redis')
    ->onQueue('processing');
```

### [Specifying Max Job Attempts / Timeout Values](#max-job-attempts-and-timeout)

#### [Max Attempts](#max-attempts)

If one of your queued jobs is encountering an error, you may want to specify how many times the job should be retried before it is recorded as a "failed" job. To accomplish this, you may define a `$tries` property on your job class:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     */
    public int $tries = 5;
}
```

Alternatively, you may define a `tries` method on your job class. This allows you to return dynamic values based on the job's payload or other conditions:

```php
/**
 * The number of times the job may be attempted.
 */
public function tries(): int
{
    return $this-> podcast->isPremium() ? 5 : 3;
}
```

#### [Time-Based Attempts](#time-based-attempts)

In addition to defining how many times a job can be retried before failing, you may define a time at which the job should timeout. This allows a job to be retried any time before a given duration has elapsed:

```php
use DateTime;

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(10);
}
```

When using the `throttleExceptions` job middleware, the job may be retried based on thrown exceptions, and this is often a better approach than time-based or attempt-based retries.

#### [Manually Releasing A Job](#manually-releasing-a-job)

You may manually release a job back onto the queue using the `release` method. The `release` method accepts the number of seconds delay before the job should be retried:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    if ($this->podcast->isProcessing()) {
        $this->release(now()->addMinutes(5));
    }
}
```

#### [Manually Failing A Job](#manually-failing-a-job)

You may manually mark a job as "failed" using the `fail` method:

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    try {
        // Process podcast...
    } catch (PodcastWasDeleted $e) {
        $this->fail($e);
    }
}
```

### [SQS FIFO and Fair Queues](#sqs-fifo-and-fair-queues)

If you use Amazon SQS FIFO queues, you need to ensure you only dispatch one job at a time to prevent duplicate processes. You may achieve this by using the `onQueue` method to set a FIFO "deduplication ID":

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)
    ->onQueue('podcast-broadcast')
    ->uniqueId($podcast->uuid);
```

By default, the unique ID is generated from the serializing the job's properties. If you would like to override this behavior, you may define a `uniqueId` method on your job class:

```php
/**
 * Get the unique ID for the job.
 */
public function uniqueId(): string
{
    return (string) $this->podcast->uuid;
}
```

The unique lock will be released when the job completes processing.

To ensure jobs are only processed once on a given SQS queue, you may use the `uniqueFor` method:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)
    ->onQueue('podcast-broadcast')
    ->uniqueId($podcast->uuid)
    ->uniqueFor(300);
```

Unique jobs require a cache driver that supports [[05-digging-deeper/03-cache.md#atomic-locks|locks]]. Currently, the `memcached`, `redis`, `dynamodb`, `database`, `file`, and `array` cache drivers support atomic locks.

### [Queue Failover](#queue-failover)

If you would like to specify an alternate queue connection to dispatch to if the primary queue connection fails, you may use the `onConnection` method with `fallbackOn`:

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)
    ->onConnection('sqs')
    ->fallbackOn('redis');
```

Laravel will attempt to dispatch to the `sqs` queue connection. If an `AwsException` occurs during dispatch, Laravel will attempt to dispatch to the `redis` queue connection instead.

This requires the retry after configuration to be set for both queue connections.

### [Error Handling](#error-handling)

If an exception is thrown while the job is being processed, the job will automatically be released back onto the queue so it can be retried again. The job will continue to be released until it has been attempted the maximum number of times defined by the `$tries` property. If no max attempts have been defined, the job will be released indefinitely.

#### [Cleaning Up After Failed Jobs](#cleaning-up-after-failed-jobs)

If you would like to run some logic when a job fails, you may define a `failed` method on your job class. This is the perfect location to send notifications or rollback any actions that were partially completed by the job:

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    public Podcast $podcast;

    /**
     * Create a new job instance.
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }

    /**
     * Handle a job failure.
     */
    public function failed(?Throwable $exception): void
    {
        // Called when the job fails...
    }
}
```

## [Job Batching](#job-batching)

Laravel's job batching feature allows you to easily batch up large numbers of queued jobs and track their progress and completion.

### [Defining Batchable Jobs](#defining-batchable-jobs)

To define a batchable job, you should add the `Illuminate\Contracts\Queue\ShouldBatch` interface to your job class. This interface tells Laravel that the job can be batched:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldBatch;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportCsv implements ShouldQueue, ShouldBatch
{
    use Queueable;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // Import CSV data...
    }
}
```

### [Dispatching Batches](#dispatching-batches)

To dispatch a batch, you may use the `batch` method from the `Bus` facade. First, gather the jobs you wish to batch in an array:

```php
use App\Jobs\ImportCsv;
use Illuminate\Support\Facades\Bus;

$jobs = [];

foreach (range(1, 100) as $id) {
    $jobs[] = new ImportCsv($id);
}

Bus::batch($jobs)->dispatch();
```

### [Chains and Batches](#chains-and-batches)

You may chain batch jobs using the `then` method. The callback will be invoked once every job in the batch has finished processing:

```php
use App\Jobs\ImportCsv;
use App\Jobs\NotifyUserOfImportCompletion;
use Illuminate\Support\Facades\Bus;

Bus::batch([
    new ImportCsv(1),
    new ImportCsv(2),
    new ImportCsv(3),
])->then(function (Batch $batch) {
    // All jobs completed...
})->dispatch();
```

### [Adding Jobs to Batches](#adding-jobs-to-batches)

Sometimes it may be useful to add additional jobs to a batch from within a batched job. This can be accomplished using the `batch` method on the `$this->batch` object:

```php
$batch = $this->batch();

$batch->add([
    new ImportCsv(4),
    new ImportCsv(5),
]);
```

### [Inspecting Batches](#inspecting-batches)

Laravel provides a variety of methods to inspect a batch. First, you may retrieve the total number of jobs in the batch using the `totalJobs` method:

```php
$batch = $this->batch();

return $batch->totalJobs();
```

You may also retrieve the jobs that have been processed, failed, or pending using the `processedJobs`, `failedJobs`, and `pendingJobs` methods:

```php
$batch->processedJobs();

$batch->failedJobs();

$batch->pendingJobs();
```

### [Cancelling Batches](#cancelling-batches)

You may cancel a batch by calling the `cancel` method on the `$batch` object:

```php
$batch->cancel();
```

### [Batch Failures](#batch-failures)

When a job within a batch fails, Laravel will automatically cancel future jobs in the batch unless explicitly instructed to ignore the failure. However, you may define a callback on the batch to take further action:

```php
use Illuminate\Support\Facades\Bus;

Bus::batch([
    new ImportCsv(1),
    new ImportCsv(2),
    new ImportCsv(3),
])
    ->then(function (Batch $batch) {
        // All jobs completed successfully...
    })
    ->catch(function (Batch $batch, Throwable $e) {
        // First failing job encountered...
        $batch->cancel();
    })
    ->finally(function (Batch $batch) {
        // Batch execution completed...
    })
    ->dispatch();
```

## [Queueing Closures](#queueing-closures)

Instead of defining a job class for a simple task, you may choose to queue a closure instead. This is useful for one-off tasks that would otherwise require creating an entire job class:

```php
use Illuminate\Support\Facades\Queue;

Queue::(function () {
    // Perform task...
})->dispatch();
```

## [Running the Queue Worker](#running-the-queue-worker)

### [The `queue:work` Command](#the-queue-work-command)

Laravel includes a `queue:work` Artisan command that will process new jobs as they are pushed onto the queue. By default, this command will process jobs from the default queue connection:

```bash
php artisan queue:work
```

### [Queue Priorities](#queue-priorities)

Sometimes you may wish to process jobs from a certain queue in priority order. If you are running multiple queue workers for a certain connection, you can use the `--queue` flag to indicate which queue prioritization:

```bash
php artisan queue:work --queue=high,default
```

### [Queue Workers and Deployment](#queue-workers-and-deployment)

The first thing you should know is that the queue workers are long-lived processes. The worker is started once when your application starts. Because of this, the application code is not reloaded between jobs. So, be sure to restart your queue workers whenever you deploy a new version of your application.

The simplest way to restart your queue workers is to restart the supervisor process:

```bash
php artisan queue:restart
```

### [Reacting to Worker Signals](#reacting-to-worker-signals)

The queue workers listen for the SIGTERM signal to gracefully quit. When this signal is received, the worker will finish processing the current job and then exit. Once exited, the process will restart if the queue worker is managed via a process monitor.

### [Job Expirations and Timeouts](#job-expirations-and-timeouts)

#### [Job Expiration](#job-expiration)

The `$expires` property allows you to specify how many seconds before the job should be considered expired. If a worker does not process the job before the expiration time elapses, the job will be released back onto the queue for reprocessing:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class RetryProcessingPodcast implements ShouldQueue
{
    /**
     * The number of seconds the job may run before timing out.
     */
    public int $timeout = 1200;

    /**
     * The number of times the job may be attempted.
     */
    public int $tries = 5;

    /**
     * The maximum number of unhandled exceptions to throw before failing.
     */
    public int $maxExceptions = 3;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
    }
}
```

#### [Worker Timeout](#worker-timeout)

The `--timeout` flag allows you to specify how long the queue worker should run before shutting down. If a job is being processed when the timeout ends, the worker will wait until the job completes before exiting:

```bash
php artisan queue:work --timeout=60
```

## [Supervisor Configuration](#supervisor-configuration)

Supervisor is a process monitor for Linux servers, and it will ensure your queue workers are restarted if they fail. To install Supervisor on Ubuntu, run:

```bash
sudo apt-get install supervisor
```

Write the configuration file in `/etc/supervisor/conf.d/`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=4
redirect_stderr=true
stdout_logfile=/home/forge/app.com/storage/logs/worker.log
stopwaitsecs=3600
```

## [Dealing With Failed Jobs](#dealing-with-failed-jobs)

When a job exceeds its number of retry attempts, Laravel's `queue:failed` command will report the job for review:

```bash
php artisan queue:failed
```

### [Retrying Failed Jobs](#retrying-failed-jobs)

To retry all failed jobs, you may execute the `queue:retry` command:

```bash
php artisan queue:retry all
```

To retry a failed job by its ID:

```bash
php artisan queue:retry 5
```

### [Ignoring Missing Models](#ignoring-missing-models)

The `Illuminate\Queue\InteractsWithQueue` trait includes a `retryUntil` method that allows you to specify when the job should stop being retried. However, this trait also provides the `$deleteWhenMissingModels` property. If a queued job accepting an Eloquent model is deleted while in queue, you may wish to have the job automatically deleted by setting this property to `true`:

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Delete the job if its model no longer exists.
     */
    public bool $deleteWhenMissingModels = true;
}
```

### [Pruning Failed Jobs](#pruning-failed-jobs)

If you would like to automatically delete failed jobs older than a specified number of minutes, you may invoke the `queue:prune-failed` Artisan command:

```bash
php artisan queue:prune-failed --hours=48
```

### [Storing Failed Jobs in DynamoDB](#storing-failed-jobs-in-dynamodb)

If you are using a cache driver other than `dynamodb` and would like to store your failed jobs in DynamoDB instead of your database, configure a `dynamodb` store:

```php
'failed' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],
```

### [Disabling Failed Job Storage](#disabling-failed-job-storage)

If you would like to disable the storage of failed jobs entirely, you may configure `false` as the queue connection driver. This allows you to skip the maintenance of a `failed_jobs` database table:

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => 'default',
    'retry_after' => 90,
    'block_for' => null,
    'after_commit' => false,
    'failed' => false,
],
```

### [Failed Job Events](#failed-job-events)

If you would like to register an event listener that will be called when a queued job fails, you may use the `Queue::failing` method. This event is a great place to notify your team:

```php
use Illuminate\Contracts\Queue\Queue;
use Illuminate\Queue\Events\JobFailed;

Queue::failing(function (JobFailed $event) {
    // $event->connectionName
    // $event->job
    // $event->exception
});
```

## [Clearing Jobs From Queues](#clearing-jobs-from-queues)

You may use the `queue:clear` command to clear jobs from a queue:

```bash
php artisan queue:clear redis
```

## [Monitoring Your Queues](#monitoring-your-queues)

You may monitor queue length using the `queue:monitor` command. This command will check your queue length and emit a log notification when a queue exceeds specified thresholds:

```bash
php artisan queue:monitor notifications:default --max=100
```

## [Testing](#testing)

Laravel provides a variety of helpful testing utilities to make it easier to test your queued jobs. Many Laravel services provide functionality to help you easily and expressively write tests, and Laravel's queue service is no exception.

### [Faking a Subset of Jobs](#faking-a-subset-of-jobs)

Sometimes you may need to prevent jobs from being dispatched to the actual queue while still allowing other jobs to run. You can achieve this by using the `except` method:

```php
Queue::fake()->except([
    App\Jobs\ArchivePodcast::class,
]);
```

### [Testing Job Chains](#testing-job-chains)

Job chains are one of the most powerful Laravel features. However, testing that they're actually constructed correctly requires a fair bit of care. Thankfully, Laravel provides the `assertChained` method to make this easy:

```php
use App\Jobs\ArchivePodcast;
use App\Jobs\EmailPodcast;
use App\Jobs\ProcessPodcast;

$podcast = Podcast::factory()->create();

Bus::assertChained([
    new ProcessPodcast($podcast),
    new EmailPodcast($podcast),
    new ArchivePodcast($podcast),
]);
```

### [Testing Job Batches](#testing-job-batches)

Laravel's batching capabilities also include testing support. You can verify that a job has been dispatched with the correct batch configuration using the `assertBatched` method:

```php
use App\Jobs\ProcessPodcast;
use Illuminate\Support\Facades\Bus;

Bus::assertBatched([
    new ProcessPodcast($this->podcast, 'batch-1'),
    new ProcessPodcast($this->podcast, 'batch-2'),
]);
```

### [Testing Job / Queue Interactions](#testing-job-queue-interactions)

To test that a job dispatches other jobs, you may use the `assertPushed` method. This method ensures a job was dispatched with the given arguments but does not ensure the job has already been processed:

```php
use App\Jobs\ProcessPodcast;
use Illuminate\Support\Facades\Queue;

Queue::assertPushed(ProcessPodcast::class, function (ProcessPodcast $job) {
    return $job->podcast->id === $this->podcast->id;
});
```

## [Job Events](#job-events)

Laravel dispatches a variety of [[05-digging-deeper/08-events.md|events]] during the queue process. You may [[05-digging-deeper/08-events.md|define listeners]] for any of the following events:

- `Illuminate\Queue\Events\JobQueued`
- `Illuminate\Queue\Events\JobProcessed`
- `Illuminate\Queue\Events\JobProcessing`
- `Illuminate\Queue\Events\Looping`
- `Illuminate\Queue\Events\WorkerStopping`
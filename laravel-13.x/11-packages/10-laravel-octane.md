---
title: Laravel Octane
description: High-performance application server supporting FrankenPHP, RoadRunner, and Swoole
url: https://laravel.com/docs/13.x/octane
tags: [packages]
---

# Laravel Octane

-   [Introduction](#introduction)
-   [Installation](#installation)
-   [Server Prerequisites](#server-prerequisites)
    -   [FrankenPHP](#frankenphp)
    -   [RoadRunner](#roadrunner)
    -   [Swoole](#swoole)
-   [Serving Your Application](#serving-your-application)
    -   [Serving Your Application via HTTPS](#serving-your-application-via-https)
    -   [Serving Your Application via Nginx](#serving-your-application-via-nginx)
    -   [Watching for File Changes](#watching-for-file-changes)
    -   [Specifying the Worker Count](#specifying-the-worker-count)
    -   [Specifying the Max Request Count](#specifying-the-max-request-count)
    -   [Specifying the Max Execution Time](#specifying-the-max-execution-time)
    -   [Reloading the Workers](#reloading-the-workers)
    -   [Stopping the Server](#stopping-the-server)
-   [Dependency Injection and Octane](#dependency-injection-and-octane)
    -   [Container Injection](#container-injection)
    -   [Request Injection](#request-injection)
    -   [Configuration Repository Injection](#configuration-repository-injection)
-   [Managing Memory Leaks](#managing-memory-leaks)
-   [Concurrent Tasks](#concurrent-tasks)
-   [Ticks and Intervals](#ticks-and-intervals)
-   [The Octane Cache](#the-octane-cache)
-   [Tables](#tables)

## [Introduction](#introduction)

[Laravel Octane](https://github.com/laravel/octane) supercharges your application's performance by serving your application using high-powered application servers, including [FrankenPHP](https://frankenphp.dev/), [Open Swoole](https://openswoole.com/), [Swoole](https://github.com/swoole/swoole-src), and [RoadRunner](https://roadrunner.dev). Octane boots your application once, keeps it in memory, and then feeds it requests at supersonic speeds.

## [Installation](#installation)

Octane may be installed via the Composer package manager:

```
1composer require laravel/octane
composer require laravel/octane
```

After installing Octane, you may execute the `octane:install` Artisan command, which will install Octane's configuration file into your application:

```
1php artisan octane:install
php artisan octane:install
```

## [Server Prerequisites](#server-prerequisites)

### [FrankenPHP](#frankenphp)

[FrankenPHP](https://frankenphp.dev) is a PHP application server, written in Go, that supports modern web features like early hints, Brotli, and Zstandard compression. When you install Octane and choose FrankenPHP as your server, Octane will automatically download and install the FrankenPHP binary for you.

#### [FrankenPHP via Laravel Sail](#frankenphp-via-laravel-sail)

If you plan to develop your application using [Laravel Sail](/docs/13.x/sail), you should run the following commands to install Octane and FrankenPHP:

```
1./vendor/bin/sail up2 3./vendor/bin/sail composer require laravel/octane
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane
```

Next, you should use the `octane:install` Artisan command to install the FrankenPHP binary:

```
1./vendor/bin/sail artisan octane:install --server=frankenphp
./vendor/bin/sail artisan octane:install --server=frankenphp
```

Finally, add a `SUPERVISOR_PHP_COMMAND` environment variable to the `laravel.test` service definition in your application's `docker-compose.yml` file. This environment variable will contain the command that Sail will use to serve your application using Octane instead of the PHP development server:

```
1services:2  laravel.test:3    environment:4      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port='${APP_PORT:-80}'" 5      XDG_CONFIG_HOME:  /var/www/html/config 6      XDG_DATA_HOME:  /var/www/html/data 
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port='${APP_PORT:-80}'"
      XDG_CONFIG_HOME:  /var/www/html/config
      XDG_DATA_HOME:  /var/www/html/data
```

To enable HTTPS, HTTP/2, and HTTP/3, apply these modifications instead:

```
 1services: 2  laravel.test: 3    ports: 4        - '${APP_PORT:-80}:80' 5        - '${VITE_PORT:-5173}:${VITE_PORT:-5173}' 6        - '443:443'  7        - '443:443/udp'  8    environment: 9      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --host=localhost --port=443 --admin-port=2019 --https" 10      XDG_CONFIG_HOME:  /var/www/html/config 11      XDG_DATA_HOME:  /var/www/html/data 
services:
  laravel.test:
    ports:
        - '${APP_PORT:-80}:80'
        - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        - '443:443'
        - '443:443/udp'
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --host=localhost --port=443 --admin-port=2019 --https"
      XDG_CONFIG_HOME:  /var/www/html/config
      XDG_DATA_HOME:  /var/www/html/data
```

Typically, you should access your FrankenPHP Sail application via `https://localhost`, as using `https://127.0.0.1` requires additional configuration and is [discouraged](https://frankenphp.dev/docs/known-issues/#using-https127001-with-docker).

#### [FrankenPHP via Docker](#frankenphp-via-docker)

Using FrankenPHP's official Docker images can offer improved performance and the use of additional extensions not included with static installations of FrankenPHP. In addition, the official Docker images provide support for running FrankenPHP on platforms it doesn't natively support, such as Windows. FrankenPHP's official Docker images are suitable for both local development and production usage.

You may use the following Dockerfile as a starting point for containerizing your FrankenPHP powered Laravel application:

```
1FROM dunglas/frankenphp2 3RUN install-php-extensions \4    pcntl5    # Add other PHP extensions here...6 7COPY . /app8 9ENTRYPOINT ["php", "artisan", "octane:frankenphp"]
FROM dunglas/frankenphp

RUN install-php-extensions \
    pcntl
    # Add other PHP extensions here...

COPY . /app

ENTRYPOINT ["php", "artisan", "octane:frankenphp"]
```

Then, during development, you may utilize the following Docker Compose file to run your application:

```
 1# compose.yaml 2services: 3  frankenphp: 4    build: 5      context: . 6    entrypoint: php artisan octane:frankenphp --workers=1 --max-requests=1 7    ports: 8      - "8000:8000" 9    volumes:10      - .:/app
# compose.yaml
services:
  frankenphp:
    build:
      context: .
    entrypoint: php artisan octane:frankenphp --workers=1 --max-requests=1
    ports:
      - "8000:8000"
    volumes:
      - .:/app
```

If the `--log-level` option is explicitly passed to the `php artisan octane:start` command, Octane will use FrankenPHP's native logger and, unless configured differently, will produce structured JSON logs.

You may consult [the official FrankenPHP documentation](https://frankenphp.dev/docs/docker/) for more information on running FrankenPHP with Docker.

#### [Custom Caddyfile Configuration](#frankenphp-caddyfile)

When using FrankenPHP, you may specify a custom Caddyfile using the `--caddyfile` option when starting Octane:

```
1php artisan octane:start --server=frankenphp --caddyfile=/path/to/your/Caddyfile
php artisan octane:start --server=frankenphp --caddyfile=/path/to/your/Caddyfile
```

This allows you to customize FrankenPHP's configuration beyond the default settings, such as adding custom middleware, configuring advanced routing, or setting up custom directives. You may consult the [official Caddy documentation](https://caddyserver.com/docs/caddyfile) for more information on Caddyfile syntax and configuration options.

### [RoadRunner](#roadrunner)

[RoadRunner](https://roadrunner.dev) is powered by the RoadRunner binary, which is built using Go. The first time you start a RoadRunner based Octane server, Octane will offer to download and install the RoadRunner binary for you.

#### [RoadRunner via Laravel Sail](#roadrunner-via-laravel-sail)

If you plan to develop your application using [Laravel Sail](/docs/13.x/sail), you should run the following commands to install Octane and RoadRunner:

```
1./vendor/bin/sail up2 3./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http
```

Next, you should start a Sail shell and use the `rr` executable to retrieve the latest Linux based build of the RoadRunner binary:

```
1./vendor/bin/sail shell2 3# Within the Sail shell...4./vendor/bin/rr get-binary
./vendor/bin/sail shell

# Within the Sail shell...
./vendor/bin/rr get-binary
```

Then, add a `SUPERVISOR_PHP_COMMAND` environment variable to the `laravel.test` service definition in your application's `docker-compose.yml` file. This environment variable will contain the command that Sail will use to serve your application using Octane instead of the PHP development server:

```
1services:2  laravel.test:3    environment:4      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port='${APP_PORT:-80}'" 
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port='${APP_PORT:-80}'"
```

Finally, ensure the `rr` binary is executable and build your Sail images:

```
1chmod +x ./rr2 3./vendor/bin/sail build --no-cache
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

### [Swoole](#swoole)

If you plan to use the Swoole application server to serve your Laravel Octane application, you must install the Swoole PHP extension. Typically, this can be done via PECL:

```
1pecl install swoole
pecl install swoole
```

#### [Open Swoole](#openswoole)

If you want to use the Open Swoole application server to serve your Laravel Octane application, you must install the Open Swoole PHP extension. Typically, this can be done via PECL:

```
1pecl install openswoole
pecl install openswoole
```

Using Laravel Octane with Open Swoole grants the same functionality provided by Swoole, such as concurrent tasks, ticks, and intervals.

#### [Swoole via Laravel Sail](#swoole-via-laravel-sail)

Before serving an Octane application via Sail, ensure you have the latest version of Laravel Sail and execute `./vendor/bin/sail build --no-cache` within your application's root directory.

Alternatively, you may develop your Swoole based Octane application using [Laravel Sail](/docs/13.x/sail), the official Docker based development environment for Laravel. Laravel Sail includes the Swoole extension by default. However, you will still need to adjust the `docker-compose.yml` file used by Sail.

To get started, add a `SUPERVISOR_PHP_COMMAND` environment variable to the `laravel.test` service definition in your application's `docker-compose.yml` file. This environment variable will contain the command that Sail will use to serve your application using Octane instead of the PHP development server:

```
1services:2  laravel.test:3    environment:4      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port='${APP_PORT:-80}'" 
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port='${APP_PORT:-80}'"
```

Finally, build your Sail images:

```
1./vendor/bin/sail build --no-cache
./vendor/bin/sail build --no-cache
```

#### [Swoole Configuration](#swoole-configuration)

Swoole supports a few additional configuration options that you may add to your `octane` configuration file if necessary. Because they rarely need to be modified, these options are not included in the default configuration file:

```
1'swoole' => [2    'options' => [3        'log_file' => storage_path('logs/swoole_http.log'),4        'package_max_length' => 10 * 1024 * 1024,5    ],6],
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

## [Serving Your Application](#serving-your-application)

The Octane server can be started via the `octane:start` Artisan command. By default, this command will utilize the server specified by the `server` configuration option of your application's `octane` configuration file:

```
1php artisan octane:start
php artisan octane:start
```

By default, Octane will start the server on port 8000, so you may access your application in a web browser via `http://localhost:8000`.

#### [Keeping Octane Running in Production](#keeping-octane-running-in-production)

If you are deploying your Octane application to production, you should use a process monitor such as Supervisor to ensure the Octane server stays running. A sample Supervisor configuration file for Octane might look like the following:

```
1[program:octane]2process_name=%(program_name)s_%(process_num)02d3command=php /home/forge/example.com/artisan octane:start --server=frankenphp --host=127.0.0.1 --port=80004autostart=true5autorestart=true6user=forge7redirect_stderr=true8stdout_logfile=/home/forge/example.com/storage/logs/octane.log9stopwaitsecs=3600
[program:octane]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/example.com/artisan octane:start --server=frankenphp --host=127.0.0.1 --port=8000
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/storage/logs/octane.log
stopwaitsecs=3600
```

### [Serving Your Application via HTTPS](#serving-your-application-via-https)

By default, applications running via Octane generate links prefixed with `http://`. The `OCTANE_HTTPS` environment variable, used within your application's `config/octane.php` configuration file, can be set to `true` when serving your application via HTTPS. When this configuration value is set to `true`, Octane will instruct Laravel to prefix all generated links with `https://`:

```
1'https' => env('OCTANE_HTTPS', false),
'https' => env('OCTANE_HTTPS', false),
```

### [Serving Your Application via Nginx](#serving-your-application-via-nginx)

If you aren't quite ready to manage your own server configuration or aren't comfortable configuring all of the various services needed to run a robust Laravel Octane application, check out [Laravel Cloud](https://cloud.laravel.com), which offers fully-managed Laravel Octane support.

In production environments, you should serve your Octane application behind a traditional web server such as Nginx or Apache. Doing so will allow the web server to serve your static assets such as images and stylesheets, as well as manage your SSL certificate termination.

In the Nginx configuration example below, Nginx will serve the site's static assets and proxy requests to the Octane server that is running on port 8000:

```
 1map $http_upgrade $connection_upgrade { 2    default upgrade; 3    ''      close; 4} 5  6server { 7    listen 80; 8    listen [::]:80; 9    server_name domain.com;10    server_tokens off;11    root /home/forge/domain.com/public;12 13    index index.php;14 15    charset utf-8;16 17    location /index.php {18        try_files /not_exists @octane;19    }20 21    location / {22        try_files $uri $uri/ @octane;23    }24 25    location = /favicon.ico { access_log off; log_not_found off; }26    location = /robots.txt  { access_log off; log_not_found off; }27 28    access_log off;29    error_log  /var/log/nginx/domain.com-error.log error;30 31    error_page 404 /index.php;32 33    location @octane {34        set $suffix "";35 36        if ($uri = /index.php) {37            set $suffix ?$query_string;38        }39 40        proxy_http_version 1.1;41        proxy_set_header Host $http_host;42        proxy_set_header Scheme $scheme;43        proxy_set_header SERVER_PORT $server_port;44        proxy_set_header REMOTE_ADDR $remote_addr;45        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;46        proxy_set_header Upgrade $http_upgrade;47        proxy_set_header Connection $connection_upgrade;48 49        proxy_pass http://127.0.0.1:8000$suffix;50    }51}
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

### [Watching for File Changes](#watching-for-file-changes)

Since your application is loaded in memory once when the Octane server starts, any changes to your application's files will not be reflected when you refresh your browser. For example, route definitions added to your `routes/web.php` file will not be reflected until the server is restarted. For convenience, you may use the `--watch` flag to instruct Octane to automatically restart the server on any file changes within your application:

```
1php artisan octane:start --watch
php artisan octane:start --watch
```

Before using this feature, you should ensure that [Node](https://nodejs.org) is installed within your local development environment. In addition, you should install the [Chokidar](https://github.com/paulmillr/chokidar) file-watching library within your project:

```
1npm install --save-dev chokidar
npm install --save-dev chokidar
```

You may configure the directories and files that should be watched using the `watch` configuration option within your application's `config/octane.php` configuration file.

### [Specifying the Worker Count](#specifying-the-worker-count)

By default, Octane will start an application request worker for each CPU core provided by your machine. These workers will then be used to serve incoming HTTP requests as they enter your application. You may manually specify how many workers you would like to start using the `--workers` option when invoking the `octane:start` command:

```
1php artisan octane:start --workers=4
php artisan octane:start --workers=4
```

If you are using the Swoole application server, you may also specify how many ["task workers"](#concurrent-tasks) you wish to start:

```
1php artisan octane:start --workers=4 --task-workers=6
php artisan octane:start --workers=4 --task-workers=6
```

### [Specifying the Max Request Count](#specifying-the-max-request-count)

To help prevent stray memory leaks, Octane gracefully restarts any worker once it has handled 500 requests. To adjust this number, you may use the `--max-requests` option:

```
1php artisan octane:start --max-requests=250
php artisan octane:start --max-requests=250
```

### [Specifying the Max Execution Time](#specifying-the-max-execution-time)

By default, Laravel Octane sets a maximum execution time of 30 seconds for incoming requests via the `max_execution_time` option in your application's `config/octane.php` configuration file:

```
1'max_execution_time' => 30,
'max_execution_time' => 30,
```

This setting defines the maximum number of seconds that an incoming request is allowed to execute before being terminated. Setting this value to `0` will disable the execution time limit entirely. This configuration option is particularly useful for applications that handle long-running requests, such as file uploads, data processing, or API calls to external services.

When you modify the `max_execution_time` configuration, you must restart the Octane server for the changes to take effect.

### [Reloading the Workers](#reloading-the-workers)

You may gracefully restart the Octane server's application workers using the `octane:reload` command. Typically, this should be done after deployment so that your newly deployed code is loaded into memory and is used to serve to subsequent requests:

```
1php artisan octane:reload
php artisan octane:reload
```

### [Stopping the Server](#stopping-the-server)

You may stop the Octane server using the `octane:stop` Artisan command:

```
1php artisan octane:stop
php artisan octane:stop
```

#### [Checking the Server Status](#checking-the-server-status)

You may check the current status of the Octane server using the `octane:status` Artisan command:

```
1php artisan octane:status
php artisan octane:status
```

## [Dependency Injection and Octane](#dependency-injection-and-octane)

Since Octane boots your application once and keeps it in memory while serving requests, there are a few caveats you should consider while building your application. For example, the `register` and `boot` methods of your application's service providers will only be executed once when the request worker initially boots. On subsequent requests, the same application instance will be reused.

In light of this, you should take special care when injecting the application service container or request into any object's constructor. By doing so, that object may have a stale version of the container or request on subsequent requests.

Octane will automatically handle resetting any first-party framework state between requests. However, Octane does not always know how to reset the global state created by your application. Therefore, you should be aware of how to build your application in a way that is Octane friendly. Below, we will discuss the most common situations that may cause problems while using Octane.

### [Container Injection](#container-injection)

In general, you should avoid injecting the application service container or HTTP request instance into the constructors of other objects. For example, the following binding injects the entire application service container into an object that is bound as a singleton:

```
 1use App\Service; 2use Illuminate\Contracts\Foundation\Application; 3  4/** 5 * Register any application services. 6 */ 7public function register(): void 8{ 9    $this->app->singleton(Service::class, function (Application $app) {10        return new Service($app);11    });12}
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

In this example, if the `Service` instance is resolved during the application boot process, the container will be injected into the service and that same container will be held by the `Service` instance on subsequent requests. This **may** not be a problem for your particular application; however, it can lead to the container unexpectedly missing bindings that were added later in the boot cycle or by a subsequent request.

As a work-around, you could either stop registering the binding as a singleton, or you could inject a container resolver closure into the service that always resolves the current container instance:

```
 1use App\Service; 2use Illuminate\Container\Container; 3use Illuminate\Contracts\Foundation\Application; 4  5$this->app->bind(Service::class, function (Application $app) { 6    return new Service($app); 7}); 8  9$this->app->singleton(Service::class, function () {10    return new Service(fn () => Container::getInstance());11});
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

The global `app` helper and the `Container::getInstance()` method will always return the latest version of the application container.

### [Request Injection](#request-injection)

In general, you should avoid injecting the application service container or HTTP request instance into the constructors of other objects. For example, the following binding injects the entire request instance into an object that is bound as a singleton:

```
 1use App\Service; 2use Illuminate\Contracts\Foundation\Application; 3  4/** 5 * Register any application services. 6 */ 7public function register(): void 8{ 9    $this->app->singleton(Service::class, function (Application $app) {10        return new Service($app['request']);11    });12}
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app['request']);
    });
}
```

In this example, if the `Service` instance is resolved during the application boot process, the HTTP request will be injected into the service and that same request will be held by the `Service` instance on subsequent requests. Therefore, all headers, input, and query string data will be incorrect, as well as all other request data.

As a work-around, you could either stop registering the binding as a singleton, or you could inject a request resolver closure into the service that always resolves the current request instance. Or, the most recommended approach is simply to pass the specific request information your object needs to one of the object's methods at runtime:

```
 1use App\Service; 2use Illuminate\Contracts\Foundation\Application; 3  4$this->app->bind(Service::class, function (Application $app) { 5    return new Service($app['request']); 6}); 7  8$this->app->singleton(Service::class, function (Application $app) { 9    return new Service(fn () => $app['request']);10});11 12// Or...13 14$service->method($request->input('name'));
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// Or...

$service->method($request->input('name'));
```

The global `request` helper will always return the request the application is currently handling and is therefore safe to use within your application.

It is acceptable to type-hint the `Illuminate\Http\Request` instance on your controller methods and route closures.

### [Configuration Repository Injection](#configuration-repository-injection)

In general, you should avoid injecting the configuration repository instance into the constructors of other objects. For example, the following binding injects the configuration repository into an object that is bound as a singleton:

```
 1use App\Service; 2use Illuminate\Contracts\Foundation\Application; 3  4/** 5 * Register any application services. 6 */ 7public function register(): void 8{ 9    $this->app->singleton(Service::class, function (Application $app) {10        return new Service($app->make('config'));11    });12}
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

In this example, if the configuration values change between requests, that service will not have access to the new values because it's depending on the original repository instance.

As a work-around, you could either stop registering the binding as a singleton, or you could inject a configuration repository resolver closure to the class:

```
 1use App\Service; 2use Illuminate\Container\Container; 3use Illuminate\Contracts\Foundation\Application; 4  5$this->app->bind(Service::class, function (Application $app) { 6    return new Service($app->make('config')); 7}); 8  9$this->app->singleton(Service::class, function () {10    return new Service(fn () => Container::getInstance()->make('config'));11});
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

The global `config` will always return the latest version of the configuration repository and is therefore safe to use within your application.

### [Managing Memory Leaks](#managing-memory-leaks)

Remember, Octane keeps your application in memory between requests; therefore, adding data to a statically maintained array will result in a memory leak. For example, the following controller has a memory leak since each request to the application will continue to add data to the static `$data` array:

```
 1use App\Service; 2use Illuminate\Http\Request; 3use Illuminate\Support\Str; 4  5/** 6 * Handle an incoming request. 7 */ 8public function index(Request $request): array 9{10    Service::$data[] = Str::random(10);11 12    return [13        // ...14    ];15}
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * Handle an incoming request.
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

While building your application, you should take special care to avoid creating these types of memory leaks. It is recommended that you monitor your application's memory usage during local development to ensure you are not introducing new memory leaks into your application.

## [Concurrent Tasks](#concurrent-tasks)

This feature requires [Swoole](#swoole).

When using Swoole, you may execute operations concurrently via light-weight background tasks. You may accomplish this using Octane's `concurrently` method. You may combine this method with PHP array destructuring to retrieve the results of each operation:

```
1use App\Models\User;2use App\Models\Server;3use Laravel\Octane\Facades\Octane;4 5[$users, $servers] = Octane::concurrently([6    fn () => User::all(),7    fn () => Server::all(),8]);
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

Concurrent tasks processed by Octane utilize Swoole's "task workers", and execute within an entirely different process than the incoming request. The amount of workers available to process concurrent tasks is determined by the `--task-workers` directive on the `octane:start` command:

```
1php artisan octane:start --workers=4 --task-workers=6
php artisan octane:start --workers=4 --task-workers=6
```

When invoking the `concurrently` method, you should not provide more than 1024 tasks due to limitations imposed by Swoole's task system.

## [Ticks and Intervals](#ticks-and-intervals)

This feature requires [Swoole](#swoole).

When using Swoole, you may register "tick" operations that will be executed every specified number of seconds. You may register "tick" callbacks via the `tick` method. The first argument provided to the `tick` method should be a string that represents the name of the ticker. The second argument should be a callable that will be invoked at the specified interval.

In this example, we will register a closure to be invoked every 10 seconds. Typically, the `tick` method should be called within the `boot` method of one of your application's service providers:

```
1Octane::tick('simple-ticker', fn () => ray('Ticking...'))2    ->seconds(10);
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10);
```

Using the `immediate` method, you may instruct Octane to immediately invoke the tick callback when the Octane server initially boots, and every N seconds thereafter:

```
1Octane::tick('simple-ticker', fn () => ray('Ticking...'))2    ->seconds(10)3    ->immediate();
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10)
    ->immediate();
```

## [The Octane Cache](#the-octane-cache)

This feature requires [Swoole](#swoole).

When using Swoole, you may leverage the Octane cache driver, which provides read and write speeds of up to 2 million operations per second. Therefore, this cache driver is an excellent choice for applications that need extreme read / write speeds from their caching layer.

This cache driver is powered by [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table). All data stored in the cache is available to all workers on the server. However, the cached data will be flushed when the server is restarted:

```
1Cache::store('octane')->put('framework', 'Laravel', 30);
Cache::store('octane')->put('framework', 'Laravel', 30);
```

The maximum number of entries allowed in the Octane cache may be defined in your application's `octane` configuration file.

### [Cache Intervals](#cache-intervals)

In addition to the typical methods provided by Laravel's cache system, the Octane cache driver features interval based caches. These caches are automatically refreshed at the specified interval and should be registered within the `boot` method of one of your application's service providers. For example, the following cache will be refreshed every five seconds:

```
1use Illuminate\Support\Str;2 3Cache::store('octane')->interval('random', function () {4    return Str::random(10);5}, seconds: 5);
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

## [Tables](#tables)

This feature requires [Swoole](#swoole).

When using Swoole, you may define and interact with your own arbitrary [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table). Swoole tables provide extreme performance throughput and the data in these tables can be accessed by all workers on the server. However, the data within them will be lost when the server is restarted.

Tables should be defined within the `tables` configuration array of your application's `octane` configuration file. An example table that allows a maximum of 1000 rows is already configured for you. The maximum size of string columns may be configured by specifying the column size after the column type as seen below:

```
1'tables' => [2    'example:1000' => [3        'name' => 'string:1000',4        'votes' => 'int',5    ],6],
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

To access a table, you may use the `Octane::table` method:

```
1use Laravel\Octane\Facades\Octane;2 3Octane::table('example')->set('uuid', [4    'name' => 'Nuno Maduro',5    'votes' => 1000,6]);7 8return Octane::table('example')->get('uuid');
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

The column types supported by Swoole tables are: `string`, `int`, and `float`.

Copy as markdown

### On this page

-   [Introduction](#introduction)
-   [Installation](#installation)
-   [Server Prerequisites](#server-prerequisites)
    -   [FrankenPHP](#frankenphp)
    -   [RoadRunner](#roadrunner)
    -   [Swoole](#swoole)
-   [Serving Your Application](#serving-your-application)
    -   [Serving Your Application via HTTPS](#serving-your-application-via-https)
    -   [Serving Your Application via Nginx](#serving-your-application-via-nginx)
    -   [Watching for File Changes](#watching-for-file-changes)
    -   [Specifying the Worker Count](#specifying-the-worker-count)
    -   [Specifying the Max Request Count](#specifying-the-max-request-count)
    -   [Specifying the Max Execution Time](#specifying-the-max-execution-time)
    -   [Reloading the Workers](#reloading-the-workers)
    -   [Stopping the Server](#stopping-the-server)
-   [Dependency Injection and Octane](#dependency-injection-and-octane)
    -   [Container Injection](#container-injection)
    -   [Request Injection](#request-injection)
    -   [Configuration Repository Injection](#configuration-repository-injection)
-   [Managing Memory Leaks](#managing-memory-leaks)
-   [Concurrent Tasks](#concurrent-tasks)
-   [Ticks and Intervals](#ticks-and-intervals)
-   [The Octane Cache](#the-octane-cache)
-   [Tables](#tables)
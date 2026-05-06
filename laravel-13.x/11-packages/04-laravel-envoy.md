---
title: Laravel Envoy
description: A tool for executing common tasks you run on your remote servers.
url: https://laravel.com/docs/13.x/envoy
tags: [packages]
cssclasses:
  - ai
  - color-purple
color: purple
---

#Laravel Envoy

-   [Introduction](#introduction)
-   [Installation](#installation)
-   [Writing Tasks](#writing-tasks)
    -   [Defining Tasks](#defining-tasks)
    -   [Multiple Servers](#multiple-servers)
    -   [Setup](#setup)
    -   [Variables](#variables)
    -   [Stories](#stories)
    -   [Hooks](#completion-hooks)
-   [Running Tasks](#running-tasks)
    -   [Confirming Task Execution](#confirming-task-execution)
-   [Notifications](#notifications)
    -   [Slack](#slack)
    -   [Discord](#discord)
    -   [Telegram](#telegram)
    -   [Microsoft Teams](#microsoft-teams)

## [Introduction](#introduction)

[Laravel Envoy](https://github.com/laravel/envoy) is a tool for executing common tasks you run on your remote servers. Using Blade style syntax, you can easily setup tasks for deployment, Artisan commands, and more. Currently, Envoy only supports the Mac and Linux operating systems.

## [Installation](#installation)

First, install Envoy into your project using the Composer package manager:

```
composer require laravel/envoy --dev
```

Once Envoy has been installed, the Envoy binary will be available in your application's `vendor/bin` directory:

```
php vendor/bin/envoy
```

## [Writing Tasks](#writing-tasks)

### [Defining Tasks](#defining-tasks)

Tasks are the basic building block of Envoy. Tasks define the shell commands that should execute on your remote servers when the task is invoked.

All of your Envoy tasks should be defined in an `Envoy.blade.php` file at the root of your application:

```
@servers(['web' => ['[email protected]'], 'workers' => ['[email protected]']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

#### [Local Tasks](#local-tasks)

You can force a script to run on your local computer by specifying the server's IP address as `127.0.0.1`:

```
@servers(['localhost' => '127.0.0.1'])
```

#### [Importing Envoy Tasks](#importing-envoy-tasks)

Using the `@import` directive, you may import other Envoy files:

```
@import('vendor/package/Envoy.blade.php')
```

### [Multiple Servers](#multiple-servers)

Envoy allows you to easily run a task across multiple servers:

```
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

#### [Parallel Execution](#parallel-execution)

By default, tasks will be executed on each server serially. If you would like to run a task across multiple servers in parallel, add the `parallel` option:

```
@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
```

### [Setup](#setup)

Sometimes, you may need to execute arbitrary PHP code before running your Envoy tasks:

```
@setup
    $now = new DateTime;
@endsetup
```

### [Variables](#variables)

If needed, you may pass arguments to Envoy tasks by specifying them on the command line:

```
php vendor/bin/envoy run deploy --branch=master
```

### [Stories](#stories)

Stories group a set of tasks under a single, convenient name:

```
@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

### [Hooks](#completion-hooks)

When tasks and stories run, a number of hooks are executed. The hook types supported by Envoy are `@before`, `@after`, `@error`, `@success`, and `@finished`:

```
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore

@after
    if ($task === 'deploy') {
        // ...
    }
@endafter

@error
    if ($task === 'deploy') {
        // ...
    }
@enderror

@success
    // ...
@endsuccess

@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

## [Running Tasks](#running-tasks)

To run a task or story:

```
php vendor/bin/envoy run deploy
```

### [Confirming Task Execution](#confirming-task-execution)

If you would like to be prompted for confirmation before running a given task:

```
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

## [Notifications](#notifications)

### [Slack](#slack)

Envoy supports sending notifications to Slack after each task is executed:

```
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

### [Discord](#discord)

Envoy also supports sending notifications to Discord:

```
@finished
    @discord('discord-webhook-url')
@endfinished
```

### [Telegram](#telegram)

Envoy also supports sending notifications to Telegram:

```
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

### [Microsoft Teams](#microsoft-teams)

Envoy also supports sending notifications to Microsoft Teams:

```
@finished
    @microsoftTeams('webhook-url')
@endfinished
```
---
title: Laravel Homestead
description: An official, pre-packaged Vagrant box that provides a wonderful development environment.
url: https://laravel.com/docs/13.x/homestead
tags: [packages]
cssclasses:
  - ai
  - color-purple
color: purple
---

#Laravel Homestead

-   [Introduction](#introduction)
-   [Installation and Setup](#installation-and-setup)
    -   [First Steps](#first-steps)
    -   [Configuring Homestead](#configuring-homestead)
    -   [Configuring Nginx Sites](#configuring-nginx-sites)
    -   [Configuring Services](#configuring-services)
    -   [Launching the Vagrant Box](#launching-the-vagrant-box)
    -   [Per Project Installation](#per-project-installation)
    -   [Installing Optional Features](#installing-optional-features)
    -   [Aliases](#aliases)
-   [Updating Homestead](#updating-homestead)
-   [Daily Usage](#daily-usage)
    -   [Connecting via SSH](#connecting-via-ssh)
    -   [Adding Additional Sites](#adding-additional-sites)
    -   [Environment Variables](#environment-variables)
    -   [Ports](#ports)
    -   [PHP Versions](#php-versions)
    -   [Connecting to Databases](#connecting-to-databases)
    -   [Database Backups](#database-backups)
    -   [Configuring Cron Schedules](#configuring-cron-schedules)
    -   [Configuring Mailpit](#configuring-mailpit)
    -   [Configuring Minio](#configuring-minio)
    -   [Laravel Dusk](#laravel-dusk)
    -   [Sharing Your Environment](#sharing-your-environment)
-   [Debugging and Profiling](#debugging-and-profiling)
    -   [Debugging Web Requests With Xdebug](#debugging-web-requests)
    -   [Debugging CLI Applications](#debugging-cli-applications)
    -   [Profiling Applications With Blackfire](#profiling-applications-with-blackfire)
-   [Network Interfaces](#network-interfaces)
-   [Extending Homestead](#extending-homestead)
-   [Provider Specific Settings](#provider-specific-settings)

## [Introduction](#introduction)

Laravel Homestead is a legacy package that is no longer actively maintained. [Laravel Sail](/docs/13.x/sail) may be used as a modern alternative.

Laravel strives to make the entire PHP development experience delightful, including your local development environment. [Laravel Homestead](https://github.com/laravel/homestead) is an official, pre-packaged Vagrant box that provides you a wonderful development environment without requiring you to install PHP, a web server, or any other server software on your local machine.

[Vagrant](https://www.vagrantup.com) provides a simple, elegant way to manage and provision Virtual Machines. Vagrant boxes are completely disposable. If something goes wrong, you can destroy and re-create the box in minutes!

Homestead runs on any Windows, macOS, or Linux system and includes Nginx, PHP, MySQL, PostgreSQL, Redis, Memcached, Node, and all of the other software you need to develop amazing Laravel applications.

### [Included Software](#included-software)

-   Ubuntu 22.04
-   Git
-   PHP 8.3, 8.2, 8.1, 8.0, 7.4, 7.3, 7.2, 7.1, 7.0, 5.6
-   Nginx
-   MySQL 8.0
-   SQLite3
-   PostgreSQL 15
-   Composer
-   Docker
-   Node (With Yarn, Bower, Grunt, and Gulp)
-   Redis
-   Memcached
-   Beanstalkd
-   Mailpit
-   ngrok
-   Xdebug
-   wp-cli

### [Optional Software](#optional-software)

-   Apache
-   Blackfire
-   Cassandra
-   CouchDB
-   Elasticsearch
-   EventStoreDB
-   Flyway
-   Gearman
-   Go
-   Grafana
-   InfluxDB
-   Logstash
-   MariaDB
-   Meilisearch
-   MinIO
-   MongoDB
-   Neo4j
-   Oh My Zsh
-   Open Resty
-   PM2
-   Python
-   RabbitMQ
-   Rust
-   Solr
-   TimescaleDB
-   Webdriver & Laravel Dusk Utilities

## [Installation and Setup](#installation-and-setup)

### [First Steps](#first-steps)

Before launching your Homestead environment, you must install [Vagrant](https://developer.hashicorp.com/vagrant/downloads) as well as one of the following supported providers:

-   [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Download_Old_Builds_6_1)
-   [Parallels](https://www.parallels.com/products/desktop/)

#### [Installing Homestead](#installing-homestead)

You may install Homestead by cloning the Homestead repository onto your host machine:

```
git clone https://github.com/laravel/homestead.git ~/Homestead
```

After cloning the Laravel Homestead repository, you should checkout the `release` branch:

```
cd ~/Homestead
git checkout release
```

Next, execute the `bash init.sh` command from the Homestead directory to create the `Homestead.yaml` configuration file:

```
bash init.sh
```

### [Configuring Homestead](#configuring-homestead)

#### [Setting Your Provider](#setting-your-provider)

The `provider` key in your `Homestead.yaml` file indicates which Vagrant provider should be used:

```
provider: virtualbox
```

#### [Configuring Shared Folders](#configuring-shared-folders)

The `folders` property of the `Homestead.yaml` file lists all of the folders you wish to share with your Homestead environment:

```
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

You should always map individual applications to their own folder mapping instead of mapping a single large directory.

### [Configuring Nginx Sites](#configuring-nginx-sites)

Your `Homestead.yaml` file's `sites` property allows you to easily map a "domain" to a folder on your Homestead environment:

```
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

#### [Hostname Resolution](#hostname-resolution)

Homestead publishes hostnames using `mDNS` for automatic host resolution.

### [Configuring Services](#configuring-services)

Homestead starts several services by default; however, you may customize which services are enabled or disabled during provisioning:

```
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

### [Launching the Vagrant Box](#launching-the-vagrant-box)

Once you have edited the `Homestead.yaml` to your liking, run the `vagrant up` command from your Homestead directory:

```
vagrant up
```

### [Per Project Installation](#per-project-installation)

Instead of installing Homestead globally and sharing the same Homestead virtual machine across all of your projects, you may instead configure a Homestead instance for each project:

```
composer require laravel/homestead --dev
php vendor/bin/homestead make
```

### [Installing Optional Features](#installing-optional-features)

Optional software is installed using the `features` option within your `Homestead.yaml` file:

```
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
    - cassandra: true
    - elasticsearch:
        version: 7.9.0
    - mongodb: true
```

### [Aliases](#aliases)

You may add Bash aliases to your Homestead virtual machine by modifying the `aliases` file:

```
alias c='clear'
alias ..='cd ..'
```

## [Updating Homestead](#updating-homestead)

Before you begin updating Homestead you should ensure you have removed your current virtual machine:

```
vagrant destroy
```

## [Daily Usage](#daily-usage)

### [Connecting via SSH](#connecting-via-ssh)

You can SSH into your virtual machine by executing the `vagrant ssh` terminal command:

```
vagrant ssh
```

### [Adding Additional Sites](#adding-additional-sites)

Once your Homestead environment is provisioned and running, you may want to add additional Nginx sites:

```
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

### [Environment Variables](#environment-variables)

You can define global environment variables by adding them to your `Homestead.yaml` file:

```
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

### [Ports](#ports)

By default, the following ports are forwarded to your Homestead environment:

-   HTTP: 8000 → Forwards To 80
-   HTTPS: 44300 → Forwards To 443

### [PHP Versions](#php-versions)

Homestead supports running multiple versions of PHP on the same virtual machine:

```
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.4"
```

### [Connecting to Databases](#connecting-to-databases)

A `homestead` database is configured for both MySQL and PostgreSQL. To connect from your host machine's database client:

```
Host: 127.0.0.1
Port: 33060 (MySQL) or 54320 (PostgreSQL)
Username: homestead
Password: secret
```

### [Database Backups](#database-backups)

Homestead can automatically backup your database when your Homestead virtual machine is destroyed:

```
backup: true
```

### [Configuring Cron Schedules](#configuring-cron-schedules)

To enable the scheduler for a site:

```
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

### [Configuring Mailpit](#configuring-mailpit)

Mailpit allows you to intercept your outgoing email:

```
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
```

### [Configuring Minio](#configuring-minio)

Minio is an open source object storage server with an Amazon S3 compatible API:

```
features:
    - minio: true
```

### [Laravel Dusk](#laravel-dusk)

To run Laravel Dusk tests within Homestead:

```
features:
    - webdriver: true
```

### [Sharing Your Environment](#sharing-your-environment)

Homestead includes its own `share` command:

```
share homestead.test
```

## [Debugging and Profiling](#debugging-and-profiling)

### [Debugging Web Requests With Xdebug](#debugging-web-requests)

Homestead includes support for step debugging using Xdebug.

To enable Xdebug on the CLI, execute the `sudo phpenmod xdebug` command:

```
sudo phpenmod xdebug
```

### [Debugging CLI Applications](#debugging-cli-applications)

To debug a PHP CLI application, use the `xphp` shell alias:

```
xphp /path/to/script
```

### [Profiling Applications With Blackfire](#profiling-applications-with-blackfire)

Blackfire is a service for profiling web requests and CLI applications:

```
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
```

## [Network Interfaces](#network-interfaces)

The `networks` property of the `Homestead.yaml` file configures network interfaces:

```
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

## [Extending Homestead](#extending-homestead)

You may extend Homestead using the `after.sh` script:

```
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

## [Provider Specific Settings](#provider-specific-settings)

### [VirtualBox](#virtualbox)

By default, Homestead configures the `natdnshostresolver` setting to `on`:

```
provider: virtualbox
natdnshostresolver: 'off'
```
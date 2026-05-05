---
title: Laravel Mix
description: Legacy package for webpack-based asset compilation, superseded by Vite
url: https://laravel.com/docs/13.x/mix
tags: [packages]
---

# Laravel Mix

-   [Introduction](#introduction)

## [Introduction](#introduction)

Laravel Mix is a legacy package that is no longer actively maintained. [[04-the-basics/09-asset-bundling-vite.md|Vite]] may be used as a modern alternative.

[Laravel Mix](https://github.com/laravel-mix/laravel-mix), a package developed by [Laracasts](https://laracasts.com) creator Jeffrey Way, provides a fluent API for defining [webpack](https://webpack.js.org) build steps for your Laravel application using several common CSS and JavaScript pre-processors.

In other words, Mix makes it a cinch to compile and minify your application's CSS and JavaScript files. Through simple method chaining, you can fluently define your asset pipeline. For example:

```
1mix.js('resources/js/app.js', 'public/js')2    .postCss('resources/css/app.css', 'public/css');
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

If you've ever been confused and overwhelmed about getting started with webpack and asset compilation, you will love Laravel Mix. However, you are not required to use it while developing your application; you are free to use any asset pipeline tool you wish, or even none at all.

Vite has replaced Laravel Mix in new Laravel installations. For Mix documentation, please visit the [official Laravel Mix](https://laravel-mix.com/) website. If you would like to switch to Vite, please see our [Vite migration guide](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite).

Copy as markdown

### On this page

-   [Introduction](#introduction)
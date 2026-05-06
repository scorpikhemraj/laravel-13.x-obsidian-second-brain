---
title: Laravel Pint
description: Opinionated PHP code style fixer built on PHP CS Fixer
url: https://laravel.com/docs/13.x/pint
tags: [packages]
cssclasses:
  - ai
  - color-purple
color: purple
---

# Laravel Pint

-   [Introduction](#introduction)
-   [Installation](#installation)
-   [Running Pint](#running-pint)
-   [Configuring Pint](#configuring-pint)
    -   [Presets](#presets)
    -   [Rules](#rules)
    -   [Excluding Files / Folders](#excluding-files-or-folders)
-   [Continuous Integration](#continuous-integration)
    -   [GitHub Actions](#running-tests-on-github-actions)

## [Introduction](#introduction)

[Laravel Pint](https://github.com/laravel/pint) is an opinionated PHP code style fixer for minimalists. Pint is built on top of [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) and makes it simple to ensure that your code style stays clean and consistent.

Pint is automatically installed with all new Laravel applications so you may start using it immediately. By default, Pint does not require any configuration and will fix code style issues in your code by following the opinionated coding style of Laravel.

## [Installation](#installation)

Pint is included in recent releases of the Laravel framework, so installation is typically unnecessary. However, for older applications, you may install Laravel Pint via Composer:

```
1composer require laravel/pint --dev
composer require laravel/pint --dev
```

## [Running Pint](#running-pint)

You can instruct Pint to fix code style issues by invoking the `pint` binary that is available in your project's `vendor/bin` directory:

```
1./vendor/bin/pint
./vendor/bin/pint
```

If you would like Pint to run in parallel mode (experimental) for improved performance, you may use the `--parallel` option:

```
1./vendor/bin/pint --parallel
./vendor/bin/pint --parallel
```

Parallel mode also allows you to specify the maximum number of processes to run via the `--max-processes` option. If this option is not provided, Pint will use every available core on your machine:

```
1./vendor/bin/pint --parallel --max-processes=4
./vendor/bin/pint --parallel --max-processes=4
```

You may also run Pint on specific files or directories:

```
1./vendor/bin/pint app/Models2 3./vendor/bin/pint app/Models/User.php
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pint will display a thorough list of all of the files that it updates. You can view even more detail about Pint's changes by providing the `-v` option when invoking Pint:

```
1./vendor/bin/pint -v
./vendor/bin/pint -v
```

If you would like Pint to simply inspect your code for style errors without actually changing the files, you may use the `--test` option. Pint will return a non-zero exit code if any code style errors are found:

```
1./vendor/bin/pint --test
./vendor/bin/pint --test
```

If you would like Pint to only modify the files that differ from the provided branch according to Git, you may use the `--diff=[branch]` option. This can be effectively used in your CI environment (like GitHub actions) to save time by only inspecting new or modified files:

```
1./vendor/bin/pint --diff=main
./vendor/bin/pint --diff=main
```

If you would like Pint to only modify the files that have uncommitted changes according to Git, you may use the `--dirty` option:

```
1./vendor/bin/pint --dirty
./vendor/bin/pint --dirty
```

If you would like Pint to fix any files with code style errors but also exit with a non-zero exit code if any errors were fixed, you may use the `--repair` option:

```
1./vendor/bin/pint --repair
./vendor/bin/pint --repair
```

## [Configuring Pint](#configuring-pint)

As previously mentioned, Pint does not require any configuration. However, if you wish to customize the presets, rules, or inspected folders, you may do so by creating a `pint.json` file in your project's root directory:

```
1{2    "preset": "laravel3}
{
    "preset": "laravel"
}
```

In addition, if you wish to use a `pint.json` from a specific directory, you may provide the `--config` option when invoking Pint:

```
1./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
```

### [Presets](#presets)

Presets define a set of rules that can be used to fix code style issues in your code. By default, Pint uses the `laravel` preset, which fixes issues by following the opinionated coding style of Laravel. However, you may specify a different preset by providing the `--preset` option to Pint:

```
1./vendor/bin/pint --preset psr12
./vendor/bin/pint --preset psr12
```

If you wish, you may also set the preset in your project's `pint.json` file:

```
1{2    "preset": "psr123}
{
    "preset": "psr12"
}
```

Pint's currently supported presets are: `laravel`, `per`, `psr12`, `symfony`, and `empty`.

### [Rules](#rules)

Rules are style guidelines that Pint will use to fix code style issues in your code. As mentioned above, presets are predefined groups of rules that should be perfect for most PHP projects, so you typically will not need to worry about the individual rules they contain.

However, if you wish, you may enable or disable specific rules in your `pint.json` file or use the `empty` preset and define the rules from scratch:

```
 1{ 2    "preset": "laravel", 3    "rules": { 4        "simplified_null_return": true, 5        "array_indentation": false, 6        "new_with_parentheses": { 7            "anonymous_class": true, 8            "named_class": true 9        }10    }11}
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "array_indentation": false,
        "new_with_parentheses": {
            "anonymous_class": true,
            "named_class": true
        }
    }
}
```

Pint is built on top of [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer). Therefore, you may use any of its rules to fix code style issues in your project: [PHP CS Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator).

#### [Custom Rules](#custom-rules)

In addition to PHP CS Fixer rules, Pint provides custom rules prefixed with `Pint/`. These rules are not enabled by default, but you may enable them in your `pint.json` file.

##### [`Pint/phpdoc_type_annotations_only`](#phpdoc-type-annotations-only)

This rule removes all comments and docblock prose from your code, keeping only lines that contain `@` annotations such as `@param`, `@return`, `@var`, `@phpstan-type`, etc:

```
1/**2 * Get the posts for the user. 3 * 4 * @return HasMany<Post, $this>5 */6public function posts(): HasMany
/**
 * Get the posts for the user.
 *
 * @return HasMany<Post, $this>
 */
public function posts(): HasMany
```

Single-line comments and block comments without `@` annotations are removed entirely. If you would like to keep a specific comment, you may prefix it with `@note`, `@warning`, or `@todo`:

```
1// @note This comment will be preserved.
// @note This comment will be preserved.
```

To enable this rule, add it to your `pint.json` file:

```
1{2    "preset": "laravel",3    "rules": {4        "Pint/phpdoc_type_annotations_only": true5    }6}
{
    "preset": "laravel",
    "rules": {
        "Pint/phpdoc_type_annotations_only": true
    }
}
```

This rule automatically skips files in the `config` directory, as configuration files typically rely on comments for documentation.

### [Excluding Files / Folders](#excluding-files-or-folders)

By default, Pint will inspect all `.php` files in your project except those in the `vendor` directory. If you wish to exclude more folders, you may do so using the `exclude` configuration option:

```
1{2    "exclude": [3        "my-specific/folder"4    ]5}
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

If you wish to exclude all files that contain a given name pattern, you may do so using the `notName` configuration option:

```
1{2    "notName": [3        "*-my-file.php"4    ]5}
{
    "notName": [
        "*-my-file.php"
    ]
}
```

If you would like to exclude a file by providing an exact path to the file, you may do so using the `notPath` configuration option:

```
1{2    "notPath": [3        "path/to/excluded-file.php"4    ]5}
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

## [Continuous Integration](#continuous-integration)

### [GitHub Actions](#running-tests-on-github-actions)

To automate linting your project with Laravel Pint, you can configure [GitHub Actions](https://github.com/features/actions) to run Pint whenever new code is pushed to GitHub. First, be sure to grant "Read and write permissions" to workflows within GitHub at **Settings > Actions > General > Workflow permissions**. Then, create a `.github/workflows/lint.yml` file with the following content:

```
 1name: Fix Code Style 2  3on: [push] 4  5jobs: 6  lint: 7    runs-on: ubuntu-latest 8    strategy: 9      fail-fast: true10      matrix:11        php: [8.4]12 13    steps:14      - name: Checkout code15        uses: actions/checkout@v516 17      - name: Setup PHP18        uses: shivammathur/setup-php@v219        with:20          php-version: ${{ matrix.php }}21          tools: pint22 23      - name: Run Pint24        run: pint25 26      - name: Commit linted files27        uses: stefanzweifel/git-auto-commit-action@v6
name: Fix Code Style

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        php: [8.4]

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          tools: pint

      - name: Run Pint
        run: pint

      - name: Commit linted files
        uses: stefanzweifel/git-auto-commit-action@v6
```

Copy as markdown

### On this page

-   [Introduction](#introduction)
-   [Installation](#installation)
-   [Running Pint](#running-pint)
-   [Configuring Pint](#configuring-pint)
    -   [Presets](#presets)
    -   [Rules](#rules)
    -   [Excluding Files / Folders](#excluding-files-or-folders)
-   [Continuous Integration](#continuous-integration)
    -   [GitHub Actions](#running-tests-on-github-actions)
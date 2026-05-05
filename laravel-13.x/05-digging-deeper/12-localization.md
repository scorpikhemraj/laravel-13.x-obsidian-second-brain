---
title: Localization
description: Laravel Localization documentation - retrieving strings in various languages
url: https://laravel.com/docs/13.x/localization
tags: [logic]
---

# Localization

- [Introduction](#introduction)
- [Defining Translation Strings](#defining-translation-strings)
- [Retrieving Translation Strings](#retrieving-translation-strings)
- [Overriding Package Language Files](#overriding-package-language-files)

## Introduction

By default, the Laravel application skeleton does not include the `lang` directory. If you would like to customize Laravel's language files, you may publish them via the `lang:publish` Artisan command.

Laravel's localization features provide a convenient way to retrieve strings in various languages, allowing you to easily support multiple languages within your application.

Laravel provides two ways to manage translation strings. First, language strings may be stored in files within the application's `lang` directory:

```
/lang
    /en
        messages.php
    /es
        messages.php
```

Or, translation strings may be defined within JSON files that are placed within the `lang` directory:

```
/lang
    en.json
    es.json
```

### Publishing the Language Files

By default, the Laravel application skeleton does not include the `lang` directory. If you would like to customize Laravel's language files or create your own, you should scaffold the `lang` directory:

```bash
php artisan lang:publish
```

### Configuring the Locale

The default language for your application is stored in the `config/app.php` configuration file's `locale` configuration option, which is typically set using the `APP_LOCALE` environment variable.

You may also configure a "fallback language", which will be used when the default language does not contain a given translation string.

You may modify the default language for a single HTTP request at runtime using the `setLocale` method:

```php
use Illuminate\Support\Facades\App;

Route::get('/greeting/{locale}', function (string $locale) {
    if (! in_array($locale, ['en', 'es', 'fr'])) {
        abort(400);
    }

    App::setLocale($locale);

    // ...
});
```

#### Determining the Current Locale

You may use the `currentLocale` and `isLocale` methods:

```php
use Illuminate\Support\Facades\App;

$locale = App::currentLocale();

if (App::isLocale('en')) {
    // ...
}
```

### Pluralization Language

You may instruct Laravel's "pluralizer" to use a language other than English:

```php
use Illuminate\Support\_pluralizer;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pluralizer::useLanguage('spanish');

    // ...
}
```

The pluralizer's currently supported languages are: `french`, `norwegian-bokmal`, `portuguese`, `spanish`, and `turkish`.

## Defining Translation Strings

### Using Short Keys

Typically, translation strings are stored in files within the `lang` directory. Within this directory, there should be a subdirectory for each language supported by your application:

```php
// lang/en/messages.php

return [
    'welcome' => 'Welcome to our application!',
];
```

For languages that differ by territory, you should name the language directories according to ISO 15897. For example, "en_GB" should be used for British English rather than "en-gb".

### Using Translation Strings as Keys

For applications with a large number of translatable strings, defining every string with a "short key" can become confusing. For this reason, Laravel also provides support for defining translation strings using the "default" translation of the string as the key. Language files that use translation strings as keys are stored as JSON files:

```json
{
    "I love programming.": "Me encanta programar."
}
```

## Retrieving Translation Strings

You may retrieve translation strings from your language files using the `__` helper function:

```php
echo __('messages.welcome');
```

If the specified translation string does not exist, the `__` function will return the translation string key.

If you are using your default translation strings as your translation keys:

```php
echo __('I love programming.');
```

If you are using the Blade templating engine, you may use the echo syntax:

```php
{{ __('messages.welcome') }}
```

### Replacing Parameters in Translation Strings

If you wish, you may define placeholders in your translation strings. All placeholders are prefixed with a `:`:

```php
'welcome' => 'Welcome, :name',
```

To replace the placeholders:

```php
echo __('messages.welcome', ['name' => 'dayle']);
```

If your placeholder contains all capital letters, or only has its first letter capitalized, the translated value will be capitalized accordingly:

```php
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle
```

### Pluralization

Pluralization is a complex problem, as different languages have a variety of complex rules for pluralization; however, Laravel can help you translate strings differently based on pluralization rules. Using a `|` character, you may distinguish singular and plural forms:

```php
'apples' => 'There is one apple|There are many apples',
```

Of course, pluralization is also supported when using translation strings as keys:

```json
{
    "There is one apple|There are many apples": "Hay una manzana|Hay Muchas manzanas"
}
```

You may even create more complex pluralization rules:

```php
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
```

After defining a translation string that has pluralization options, you may use the `trans_choice` function:

```php
echo trans_choice('messages.apples', 10);
```

You may also define placeholder attributes in pluralization strings:

```php
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

If you would like to display the integer value that was passed to the `trans_choice` function, you may use the built-in `:count` placeholder:

```php
'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',
```

## Overriding Package Language Files

Some packages may ship with their own language files. Instead of changing the package's core files to tweak these lines, you may override them by placing files in the `lang/vendor/{package}/{locale}` directory.

So, for example, if you need to override the English translation strings in `messages.php` for a package named `skyrim/hearthfire`, you should place a language file at: `lang/vendor/hearthfire/en/messages.php`.

Within this file, you should only define the translation strings you wish to override. Any translation strings you don't override will still be loaded from the package's original language files.
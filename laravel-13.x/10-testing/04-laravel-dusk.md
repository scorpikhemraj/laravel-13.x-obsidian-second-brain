---
title: Laravel Dusk
description: Browser automation and testing with Laravel Dusk using ChromeDriver
url: https://laravel.com/docs/13.x/dusk
tags: [testing]
cssclasses:
  - ai
  - color-purple
color: purple
---

# Laravel Dusk

- [Introduction](#introduction)
- [Installation](#installation)
    - [Managing ChromeDriver Installations](#managing-chromedriver-installations)
    - [Using Other Browsers](#using-other-browsers)
- [Getting Started](#getting-started)
    - [Generating Tests](#generating-tests)
    - [Resetting the Database After Each Test](#resetting-the-database-after-each-test)
    - [Running Tests](#running-tests)
    - [Environment Handling](#environment-handling)
- [Browser Basics](#browser-basics)
    - [Creating Browsers](#creating-browsers)
    - [Navigation](#navigation)
    - [Resizing Browser Windows](#resizing-browser-windows)
    - [Browser Macros](#browser-macros)
    - [Authentication](#authentication)
    - [Cookies](#cookies)
    - [Executing JavaScript](#executing-javascript)
    - [Taking a Screenshot](#taking-a-screenshot)
- [Interacting With Elements](#interacting-with-elements)
    - [Dusk Selectors](#dusk-selectors)
    - [Text, Values, and Attributes](#text-values-and-attributes)
    - [Interacting With Forms](#interacting-with-forms)
    - [Attaching Files](#attaching-files)
    - [Pressing Buttons](#pressing-buttons)
    - [Clicking Links](#clicking-links)
    - [Using the Keyboard](#using-the-keyboard)
    - [Using the Mouse](#using-the-mouse)
    - [JavaScript Dialogs](#javascript-dialogs)
    - [Waiting for Elements](#waiting-for-elements)
- [Available Assertions](#available-assertions)
- [Pages](#pages)
- [Components](#components)
- [Continuous Integration](#continuous-integration)

## Introduction

Pest 4 now includes automated browser testing which offers significant performance and usability improvements compared to Laravel Dusk. For new projects, we recommend using Pest for browser testing.

Laravel Dusk provides an expressive, easy-to-use browser automation and testing API. By default, Dusk does not require you to install JDK or Selenium on your local computer. Instead, Dusk uses a standalone ChromeDriver installation. However, you are free to utilize any other Selenium compatible driver you wish.

## Installation

To get started, you should install Google Chrome and add the `laravel/dusk` Composer dependency to your project:

```shell
composer require laravel/dusk --dev
```

If you are manually registering Dusk's service provider, you should **never** register it in your production environment, as doing so could lead to arbitrary users being able to authenticate with your application.

After installing the Dusk package, execute the `dusk:install` Artisan command. The `dusk:install` command will create a `tests/Browser` directory, an example Dusk test, and install the Chrome Driver binary for your operating system:

```shell
php artisan dusk:install
```

Next, set the `APP_URL` environment variable in your application's `.env` file. This value should match the URL you use to access your application in a browser.

### Managing ChromeDriver Installations

If you would like to install a different version of ChromeDriver than what is installed by Laravel Dusk via the `dusk:install` command, you may use the `dusk:chrome-driver` command:

```shell
# Install the latest version of ChromeDriver for your OS...
php artisan dusk:chrome-driver

# Install a given version of ChromeDriver for your OS...
php artisan dusk:chrome-driver 86

# Install a given version of ChromeDriver for all supported OSs...
php artisan dusk:chrome-driver --all

# Install the version of ChromeDriver that matches the detected version of Chrome / Chromium for your OS...
php artisan dusk:chrome-driver --detect
```

Dusk requires the `chromedriver` binaries to be executable. If you're having problems running Dusk, you should ensure the binaries are executable using the following command: `chmod -R 0755 vendor/laravel/dusk/bin/`.

### Using Other Browsers

By default, Dusk uses Google Chrome and a standalone ChromeDriver installation to run your browser tests. However, you may start your own Selenium server and run your tests against any browser you wish.

To get started, open your `tests/DuskTestCase.php` file, which is the base Dusk test case for your application. Within this file, you can remove the call to the `startChromeDriver` method. This will stop Dusk from automatically starting the ChromeDriver:

```php
/**
 * Prepare for Dusk test execution.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

Next, you may modify the `driver` method to connect to the URL and port of your choice:

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * Create the RemoteWebDriver instance.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

## Getting Started

### Generating Tests

To generate a Dusk test, use the `dusk:make` Artisan command. The generated test will be placed in the `tests/Browser` directory:

```shell
php artisan dusk:make LoginTest
```

### Resetting the Database After Each Test

Most of the tests you write will interact with pages that retrieve data from your application's database; however, your Dusk tests should never use the `RefreshDatabase` trait. The `RefreshDatabase` trait leverages database transactions which will not be applicable or available across HTTP requests. Instead, you have two options: the `DatabaseMigrations` trait and the `DatabaseTruncation` trait.

#### Using Database Migrations

The `DatabaseMigrations` trait will run your database migrations before each test. However, dropping and re-creating your database tables for each test is typically slower than truncating the tables:

```php
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

//
```

```php
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    //
}
```

SQLite in-memory databases may not be used when executing Dusk tests. Since the browser executes within its own process, it will not be able to access the in-memory databases of other processes.

#### Using Database Truncation

The `DatabaseTruncation` trait will migrate your database on the first test in order to ensure your database tables have been properly created. However, on subsequent tests, the database's tables will simply be truncated - providing a speed boost over re-running all of your database migrations:

```php
<?php

use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;

pest()->use(DatabaseTruncation::class);

//
```

By default, this trait will truncate all tables except the `migrations` table. If you would like to customize the tables that should be truncated, you may define a `$tablesToTruncate` property on your test class:

```php
protected $tablesToTruncate = ['users'];
```

Alternatively, you may define an `$exceptTables` property on your test class to specify which tables should be excluded from truncation:

```php
protected $exceptTables = ['users'];
```

### Running Tests

To run your browser tests, execute the `dusk` Artisan command:

```shell
php artisan dusk
```

If you had test failures the last time you ran the `dusk` command, you may save time by re-running the failing tests first using the `dusk:fails` command:

```shell
php artisan dusk:fails
```

The `dusk` command accepts any argument that is normally accepted by the Pest / PHPUnit test runner, such as allowing you to only run the tests for a given group:

```shell
php artisan dusk --group=foo
```

### Environment Handling

To force Dusk to use its own environment file when running tests, create a `.env.dusk.{environment}` file in the root of your project. For example, if you will be initiating the `dusk` command from your `local` environment, you should create a `.env.dusk.local` file.

When running tests, Dusk will back-up your `.env` file and rename your Dusk environment to `.env`. Once the tests have completed, your `.env` file will be restored.

## Browser Basics

### Creating Browsers

To get started, let's write a test that verifies we can log into our application. After generating a test, we can modify it to navigate to the login page, enter some credentials, and click the "Login" button. To create a browser instance, you may call the `browse` method from within your Dusk test:

```php
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

test('basic example', function () {
    $user = User::factory()->create([
        'email' => ' example@example.com',
    ]);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/home');
    });
});
```

```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    /**
     * A basic browser test example.
     */
    public function test_basic_example(): void
    {
        $user = User::factory()->create([
            'email' => ' example@example.com',
        ]);

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                ->type('email', $user->email)
                ->type('password', 'password')
                ->press('Login')
                ->assertPathIs('/home');
        });
    }
}
```

As you can see in the example above, the `browse` method accepts a closure. A browser instance will automatically be passed to this closure by Dusk and is the main object used to interact with and make assertions against your application.

#### Creating Multiple Browsers

Sometimes you may need multiple browsers in order to properly carry out a test. For example, multiple browsers may be needed to test a chat screen that interacts with websockets. To create multiple browsers, simply add more browser arguments to the signature of the closure given to the `browse` method:

```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))
        ->visit('/home')
        ->waitForText('Message');

    $second->loginAs(User::find(2))
        ->visit('/home')
        ->waitForText('Message')
        ->type('message', 'Hey Taylor')
        ->press('Send');

    $first->waitForText('Hey Taylor')
        ->assertSee('Jeffrey Way');
});
```

### Navigation

The `visit` method may be used to navigate to a given URI within your application:

```php
$browser->visit('/login');
```

You may use the `visitRoute` method to navigate to a named route:

```php
$browser->visitRoute($routeName, $parameters);
```

You may navigate "back" and "forward" using the `back` and `forward` methods:

```php
$browser->back();

$browser->forward();
```

You may use the `refresh` method to refresh the page:

```php
$browser->refresh();
```

### Resizing Browser Windows

You may use the `resize` method to adjust the size of the browser window:

```php
$browser->resize(1920, 1080);
```

The `maximize` method may be used to maximize the browser window:

```php
$browser->maximize();
```

The `fitContent` method will resize the browser window to match the size of its content:

```php
$browser->fitContent();
```

You may use the `move` method to move the browser window to a different position on your screen:

```php
$browser->move($x = 100, $y = 100);
```

### Browser Macros

If you would like to define a custom browser method that you can re-use in a variety of your tests, you may use the `macro` method on the `Browser` class:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Browser;

class DuskServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Browser::macro('scrollToElement', function (string $element = null) {
            $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

            return $this;
        });
    }
}
```

### Authentication

Often, you will be testing pages that require authentication. You can use Dusk's `loginAs` method in order to avoid interacting with your application's login screen during every test:

```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
        ->visit('/home');
});
```

After using the `loginAs` method, the user session will be maintained for all tests within the file.

### Cookies

You may use the `cookie` method to get or set an encrypted cookie's value:

```php
$browser->cookie('name');

$browser->cookie('name', 'Taylor');
```

You may use the `plainCookie` method to get or set an unencrypted cookie's value:

```php
$browser->plainCookie('name');

$browser->plainCookie('name', 'Taylor');
```

You may use the `deleteCookie` method to delete the given cookie:

```php
$browser->deleteCookie('name');
```

### Executing JavaScript

You may use the `script` method to execute arbitrary JavaScript statements within the browser:

```php
$browser->script('document.documentElement.scrollTop = 0');

$browser->script([
    'document.body.scrollTop = 0',
    'document.documentElement.scrollTop = 0',
]);

$output = $browser->script('return window.location.pathname');
```

### Taking a Screenshot

You may use the `screenshot` method to take a screenshot and store it with the given filename:

```php
$browser->screenshot('filename');
```

The `responsiveScreenshots` method may be used to take a series of screenshots at various breakpoints:

```php
$browser->responsiveScreenshots('filename');
```

The `screenshotElement` method may be used to take a screenshot of a specific element on the page:

```php
$browser->screenshotElement('#selector', 'filename');
```

## Interacting With Elements

### Dusk Selectors

Choosing good CSS selectors for interacting with elements is one of the hardest parts of writing Dusk tests. Dusk selectors allow you to focus on writing effective tests rather than remembering CSS selectors. To define a selector, add a `dusk` attribute to your HTML element:

```html
<button dusk="login-button">Login</button>
```

Then, when interacting with a Dusk browser, prefix the selector with `@`:

```php
$browser->click('@login-button');
```

If desired, you may customize the HTML attribute that the Dusk selector utilizes:

```php
use Laravel\Dusk\Dusk;

Dusk::selectorHtmlAttribute('data-dusk');
```

### Text, Values, and Attributes

Dusk provides several methods for interacting with the current value, display text, and attributes of elements on the page:

```php
// Retrieve the value...
$value = $browser->value('selector');

// Set the value...
$browser->value('selector', 'value');

// Retrieve text...
$text = $browser->text('selector');

// Retrieve attributes...
$attribute = $browser->attribute('selector', 'value');
```

### Interacting With Forms

Dusk provides a variety of methods for interacting with forms and input elements:

```php
$browser->type('email', 'example@example.com');

$browser->type('tags', 'foo')
    ->append('tags', ', bar, baz');

$browser->clear('email');

$browser->typeSlowly('mobile', '+1 (202) 555-5555');

$browser->type('tags', 'foo')
    ->appendSlowly('tags', ', bar, baz');
```

#### Dropdowns

```php
$browser->select('size', 'Large');

$browser->select('size');

$browser->select('categories', ['Art', 'Music']);
```

#### Checkboxes

```php
$browser->check('terms');

$browser->uncheck('terms');
```

#### Radio Buttons

```php
$browser->radio('size', 'large');
```

### Attaching Files

The `attach` method may be used to attach a file to a `file` input element:

```php
$browser->attach('photo', __DIR__.'/photos/mountains.png');
```

The attach function requires the `Zip` PHP extension to be installed and enabled on your server.

### Pressing Buttons

The `press` method may be used to click a button element on the page:

```php
$browser->press('Login');
```

When submitting forms, many applications disable the form's submission button after it is pressed. To press a button and wait for the button to be re-enabled:

```php
$browser->pressAndWaitFor('Save');

$browser->pressAndWaitFor('Save', 1);
```

### Clicking Links

To click a link, you may use the `clickLink` method:

```php
$browser->clickLink($linkText);
```

You may use the `seeLink` method to determine if a link with the given display text is visible on the page:

```php
if ($browser->seeLink($linkText)) {
    // ...
}
```

### Using the Keyboard

The `keys` method allows you to provide more complex input sequences:

```php
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');

$browser->keys('.app', ['{command}', 'j']);
```

### Using the Mouse

```php
$browser->click('.selector');

$browser->clickAtXPath('//div[@class = "selector"]');

$browser->clickAtPoint($x = 0, $y = 0);

$browser->doubleClick();

$browser->rightClick();

$browser->clickAndHold('.selector');

$browser->mouseover('.selector');

$browser->drag('.from-selector', '.to-selector');
```

### JavaScript Dialogs

```php
$browser->waitForDialog($seconds = null);

$browser->assertDialogOpened('Dialog message');

$browser->typeInDialog('Hello World');

$browser->acceptDialog();

$browser->dismissDialog();
```

### Waiting for Elements

Dusk provides several methods for waiting for elements:

```php
$browser->waitForText('Hello');

$browser->waitForLink('Login');

$browser->waitForElement('.selector');

$browser->waitUntilVisible('.selector');

$browser->waitForEnabled('input[type=submit]');
```

## Available Assertions

Dusk provides many assertion methods:

```php
$browser->assertTitle('Home');

$browser->assertTitleContains('Home');

$browser->assertPathIs('/home');

$browser->assertSee('Hello World');

$browser->assertSeeInOrder('First', 'Second');

$browser->assertDontSee('Hello');

$browser->assertQueryStringHas('name', 'value');

$browser->assertQueryStringMissing('name');

$browser->assertVisible('.selector');

$browser->assertHidden('.selector');

$browser->assertEnabled('input');

$browser->assertDisabled('input');

$browser->assertChecked('input[type=checkbox]');

$browser->assertNotChecked('input[type=checkbox]');

$browser->assertSelected('select', 'value');

$browser->assertNotSelected('select', 'value');
```

## Pages

Laravel Dusk also includes support for "pages". Pages allow you to define expressive actions that can be performed on a given page in your application.

### Generating Pages

To generate a page object, use the `dusk:page` Artisan command:

```shell
php artisan dusk:page Login
```

### Configuring Pages

Pages typically contain four methods: `url`, `assert`, `elements`, and `actions`.

### Navigating to Pages

You may use the `visit` method to navigate to a given page:

```php
$browser->visit(new LoginPage);
```

You may also use convenience methods:

```php
$browser->visitLoginPage()
    ->assertSee('Login');
```

### Shorthand Selectors

Pages also allow you to define shorthand selectors:

```php
protected function assert($browser)
{
    $browser->assertSee('Login');
}

protected function elements()
{
    return [
        '@email' => 'input[name=email]',
        '@password' => 'input[name=password]',
        '@submit' => 'button[type=submit]',
    ];
}
```

### Page Methods

You may define additional actions on the page:

```php
public function submitLogin($email, $password)
{
    return $this->type('@email', $email)
        ->type('@password', $password)
        ->press('@submit');
}
```

## Components

Components are similar to pages, but they are intended to represent reusable pieces of UI that may be scattered throughout your application.

### Generating Components

To generate a Dusk component, use the `dusk:component` Artisan command:

```shell
php artisan dusk:component DatePicker
```

## Continuous Integration

### GitHub Actions

```yaml
name: Browser Tests

on: [push, pull_request]

jobs:
  dusk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.2'
      - name: Install Dependencies
        run: composer install --no-interaction
      - name: Run Tests
        run: php artisan dusk
        env:
          APP_URL: http://localhost
```
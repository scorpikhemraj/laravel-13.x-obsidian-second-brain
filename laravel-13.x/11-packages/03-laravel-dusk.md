---
title: Laravel Dusk
description: Provides an expressive, easy-to-use browser automation and testing API.
url: https://laravel.com/docs/13.x/dusk
tags: [packages]
cssclasses:
  - ai
  - color-purple
color: purple
---

#Laravel Dusk

-   [Introduction](#introduction)
-   [Installation](#installation)
    -   [Managing ChromeDriver Installations](#managing-chromedriver-installations)
    -   [Using Other Browsers](#using-other-browsers)
-   [Getting Started](#getting-started)
    -   [Generating Tests](#generating-tests)
    -   [Resetting the Database After Each Test](#resetting-the-database-after-each-test)
    -   [Running Tests](#running-tests)
    -   [Environment Handling](#environment-handling)
-   [Browser Basics](#browser-basics)
    -   [Creating Browsers](#creating-browsers)
    -   [Navigation](#navigation)
    -   [Resizing Browser Windows](#resizing-browser-windows)
    -   [Browser Macros](#browser-macros)
    -   [Authentication](#authentication)
    -   [Cookies](#cookies)
    -   [Executing JavaScript](#executing-javascript)
    -   [Taking a Screenshot](#taking-a-screenshot)
-   [Interacting With Elements](#interacting-with-elements)
    -   [Dusk Selectors](#dusk-selectors)
    -   [Text, Values, and Attributes](#text-values-and-attributes)
    -   [Interacting With Forms](#interacting-with-forms)
    -   [Pressing Buttons](#pressing-buttons)
    -   [Clicking Links](#clicking-links)
    -   [Using the Keyboard](#using-the-keyboard)
    -   [Using the Mouse](#using-the-mouse)
-   [Available Assertions](#available-assertions)
-   [Pages](#pages)
    -   [Generating Pages](#generating-pages)
    -   [Configuring Pages](#configuring-pages)
    -   [Navigating to Pages](#navigating-to-pages)
-   [Components](#components)
    -   [Generating Components](#generating-components)
    -   [Using Components](#using-components)
-   [Continuous Integration](#continuous-integration)

## [Introduction](#introduction)

[Pest 4](https://pestphp.com/) now includes automated browser testing which offers significant performance and usability improvements compared to Laravel Dusk. For new projects, we recommend using Pest for browser testing.

[Laravel Dusk](https://github.com/laravel/dusk) provides an expressive, easy-to-use browser automation and testing API. By default, Dusk does not require you to install JDK or Selenium on your local computer. Instead, Dusk uses a standalone [ChromeDriver](https://sites.google.com/chromium.org/driver) installation.

## [Installation](#installation)

To get started, you should install [Google Chrome](https://www.google.com/chrome) and add the `laravel/dusk` Composer dependency to your project:

```
composer require laravel/dusk --dev
```

After installing the Dusk package, execute the `dusk:install` Artisan command:

```
php artisan dusk:install
```

Next, set the `APP_URL` environment variable in your application's `.env` file.

### [Managing ChromeDriver Installations](#managing-chromedriver-installations)

If you would like to install a different version of ChromeDriver than what is installed by Laravel Dusk via the `dusk:install` command, you may use the `dusk:chrome-driver` command:

```
php artisan dusk:chrome-driver
php artisan dusk:chrome-driver 86
php artisan dusk:chrome-driver --all
php artisan dusk:chrome-driver --detect
```

### [Using Other Browsers](#using-other-browsers)

By default, Dusk uses Google Chrome. However, you may start your own Selenium server and run your tests against any browser you wish.

## [Getting Started](#getting-started)

### [Generating Tests](#generating-tests)

To generate a Dusk test, use the `dusk:make` Artisan command:

```
php artisan dusk:make LoginTest
```

### [Resetting the Database After Each Test](#resetting-the-database-after-each-test)

Most of the tests you write will interact with pages that retrieve data from your application's database; however, your Dusk tests should never use the `RefreshDatabase` trait. You have two options: the `DatabaseMigrations` trait and the `DatabaseTruncation` trait.

### [Running Tests](#running-tests)

To run your browser tests, execute the `dusk` Artisan command:

```
php artisan dusk
```

If you had test failures the last time you ran the `dusk` command, you may save time by re-running the failing tests first using the `dusk:fails` command:

```
php artisan dusk:fails
```

### [Environment Handling](#environment-handling)

To force Dusk to use its own environment file when running tests, create a `.env.dusk.{environment}` file in the root of your project.

## [Browser Basics](#browser-basics)

### [Creating Browsers](#creating-browsers)

To create a browser instance, you may call the `browse` method from within your Dusk test:

```
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/login')
        ->type('email', $user->email)
        ->type('password', 'password')
        ->press('Login')
        ->assertPathIs('/home');
});
```

### [Navigation](#navigation)

The `visit` method may be used to navigate to a given URI within your application:

```
$browser->visit('/login');
$browser->visitRoute($routeName, $parameters);
$browser->back();
$browser->forward();
$browser->refresh();
```

### [Resizing Browser Windows](#resizing-browser-windows)

```
$browser->resize(1920, 1080);
$browser->maximize();
$browser->fitContent();
```

### [Browser Macros](#browser-macros)

If you would like to define a custom browser method, you may use the `macro` method on the `Browser` class:

```
Browser::macro('scrollToElement', function (string $element = null) {
    $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");
    return $this;
});
```

### [Authentication](#authentication)

You can use Dusk's `loginAs` method to avoid interacting with your application's login screen during every test:

```
$browser->loginAs(User::find(1))
    ->visit('/home');
```

### [Cookies](#cookies)

```
$browser->cookie('name');
$browser->cookie('name', 'Taylor');
$browser->plainCookie('name');
$browser->deleteCookie('name');
```

### [Executing JavaScript](#executing-javascript)

```
$browser->script('document.documentElement.scrollTop = 0');
$output = $browser->script('return window.location.pathname');
```

### [Taking a Screenshot](#taking-a-screenshot)

```
$browser->screenshot('filename');
$browser->responsiveScreenshots('filename');
$browser->screenshotElement('#selector', 'filename');
```

## [Interacting With Elements](#interacting-with-elements)

### [Dusk Selectors](#dusk-selectors)

Dusk selectors allow you to focus on writing effective tests rather than remembering CSS selectors:

```
<button dusk="login-button">Login</button>
$browser->click('@login-button');
```

### [Text, Values, and Attributes](#text-values-and-attributes)

```
$value = $browser->value('selector');
$text = $browser->text('selector');
$attribute = $browser->attribute('selector', 'value');
```

### [Interacting With Forms](#interacting-with-forms)

```
$browser->type('email', 'taylor@laravel.com');
$browser->typeSlowly('mobile', '+1 (202) 555-5555');
$browser->append('tags', ', bar, baz');
$browser->clear('email');
$browser->select('size', 'Large');
$browser->check('terms');
$browser->uncheck('terms');
$browser->radio('size', 'large');
```

### [Pressing Buttons](#pressing-buttons)

```
$browser->press('Login');
$browser->pressAndWaitFor('Save');
```

### [Clicking Links](#clicking-links)

```
$browser->clickLink($linkText);
$browser->seeLink($linkText);
```

### [Using the Keyboard](#using-the-keyboard)

```
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');
$browser->keys('.app', ['{command}', 'j']);
```

### [Using the Mouse](#using-the-mouse)

```
$browser->click('.selector');
$browser->doubleClick();
$browser->rightClick();
$browser->clickAndHold();
$browser->mouseover('.selector');
$browser->drag('.from-selector', '.to-selector');
```

## [Available Assertions](#available-assertions)

Dusk provides a variety of assertions. Some of the most common assertions include:

```
$browser->assertTitle('Laravel');
$browser->assertTitleContains('Laravel');
$browser->assertPathIs('/home');
$browser->assertQueryStringHas('name', 'value');
$browser->assertSee('text');
$browser->assertDontSee('text');
$browser->assertVisible('.selector');
$browser->assertMissing('.selector');
$browser->assertChecked('terms');
$browser->assertRadioSelected('size', 'large');
$browser->assertSelected('country', 'US');
$browser->assertValue('email', 'taylor@laravel.com');
```

## [Pages](#pages)

### [Generating Pages](#generating-pages)

To generate a Dusk page, use the `dusk:page` Artisan command:

```
php artisan dusk:page Login
```

### [Configuring Pages](#configuring-pages)

Pages allow you to define shorthand selectors and convenience methods for common tasks:

```
<?php

namespace Tests\Browser\Pages;

class Login extends Page
{
    public function url(): string
    {
        return '/login';
    }

    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url());
    }

    public function submitLogin(Browser $browser, string $email, string $password): void
    {
        $browser->type('email', $email)
            ->type('password', $password)
            ->press('Login');
    }
}
```

### [Navigating to Pages](#navigating-to-pages)

```
$browser->visit(new Login);
$browser->visit('/login');
```

## [Components](#components)

### [Generating Components](#generating-components)

To generate a Dusk component, use the `dusk:component` Artisan command:

```
php artisan dusk:component DatePicker
```

### [Using Components](#using-components)

Components allow you to create reusable groups of element interactions:

```
$browser->script('@datepicker->setDate', Carbon::now()->format('Y-m-d'));
```

## [Continuous Integration](#continuous-integration)

### [Heroku CI](#running-tests-on-heroku-ci)

### [Travis CI](#running-tests-on-travis-ci)

### [GitHub Actions](#running-tests-on-github-actions)

### [Chipper CI](#running-tests-on-chipper-ci)

Visit the Laravdusk documentation for more information on running Dusk in CI environments.
---
title: Laravel Fortify
description: A frontend agnostic authentication backend implementation for Laravel.
url: https://laravel.com/docs/13.x/fortify
tags: [packages]
---

#Laravel Fortify

-   [Introduction](#introduction)
    -   [What is Fortify?](#what-is-fortify)
    -   [When Should I Use Fortify?](#when-should-i-use-fortify)
-   [Installation](#installation)
    -   [Fortify Features](#fortify-features)
    -   [Disabling Views](#disabling-views)
-   [Authentication](#authentication)
    -   [Customizing User Authentication](#customizing-user-authentication)
    -   [Customizing the Authentication Pipeline](#customizing-the-authentication-pipeline)
    -   [Customizing Redirects](#customizing-authentication-redirects)
-   [Two-Factor Authentication](#two-factor-authentication)
    -   [Enabling Two-Factor Authentication](#enabling-two-factor-authentication)
    -   [Authenticating With Two-Factor Authentication](#authenticating-with-two-factor-authentication)
    -   [Disabling Two-Factor Authentication](#disabling-two-factor-authentication)
-   [Registration](#registration)
    -   [Customizing Registration](#customizing-registration)
-   [Password Reset](#password-reset)
    -   [Requesting a Password Reset Link](#requesting-a-password-reset-link)
    -   [Resetting the Password](#resetting-the-password)
    -   [Customizing Password Resets](#customizing-password-resets)
-   [Email Verification](#email-verification)
    -   [Protecting Routes](#protecting-routes)
-   [Password Confirmation](#password-confirmation)

## [Introduction](#introduction)

[Laravel Fortify](https://github.com/laravel/fortify) is a frontend agnostic authentication backend implementation for Laravel. Fortify registers the routes and controllers needed to implement all of Laravel's authentication features, including login, registration, password reset, email verification, and more.

### [What is Fortify?](#what-is-fortify)

As mentioned previously, Laravel Fortify is a frontend agnostic authentication backend implementation for Laravel. Fortify registers the routes and controllers needed to implement all of Laravel's authentication features.

### [When Should I Use Fortify?](#when-should-i-use-fortify)

You may be wondering when it is appropriate to use Laravel Fortify. First, if you are using one of Laravel's application starter kits, you do not need to install Laravel Fortify since all of Laravel's application starter kits use Fortify.

## [Installation](#installation)

To get started, install Fortify using the Composer package manager:

```
composer require laravel/fortify
```

Next, publish Fortify's resources using the `fortify:install` Artisan command:

```
php artisan fortify:install
```

This command will publish Fortify's actions to your `app/Actions` directory, which will be created if it does not exist. In addition, the `FortifyServiceProvider`, configuration file, and all necessary database migrations will be published.

Next, you should migrate your database:

```
php artisan migrate
```

### [Fortify Features](#fortify-features)

The `fortify` configuration file contains a `features` configuration array. This array defines which backend routes / features Fortify will expose by default:

```
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

### [Disabling Views](#disabling-views)

By default, Fortify defines routes that are intended to return views. However, if you are building a JavaScript driven single-page application, you may not need these routes:

```
'views' => false,
```

## [Authentication](#authentication)

To get started, we need to instruct Fortify how to return our "login" view. All of the authentication view's rendering logic may be customized using the `Laravel\Fortify\Fortify` class:

```
use Laravel\Fortify\Fortify;

public function boot(): void
{
    Fortify::loginView(function () {
        return view('auth.login');
    });

    // ...
}
```

### [Customizing User Authentication](#customizing-user-authentication)

Fortify will automatically retrieve and authenticate the user based on the provided credentials and the authentication guard that is configured for your application. However, you may sometimes wish to have full customization:

```
Fortify::authenticateUsing(function (Request $request) {
    $user = User::where('email', $request->email)->first();

    if ($user &&
        Hash::check($request->password, $user->password)) {
        return $user;
    }
});
```

### [Customizing the Authentication Pipeline](#customizing-the-authentication-pipeline)

Laravel Fortify authenticates login requests through a pipeline of invokable classes:

```
Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            config('fortify.lowercase_usernames') ? CanonicalizeUsername::class : null,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

### [Customizing Redirects](#customizing-authentication-redirects)

If the login attempt is successful, Fortify will redirect you to the URI configured via the `home` configuration option within your application's `fortify` configuration file.

## [Two-Factor Authentication](#two-factor-authentication)

When Fortify's two-factor authentication feature is enabled, the user is required to input a six digit numeric token during the authentication process.

Before getting started, you should first ensure that your application's `App\Models\User` model uses the `Laravel\Fortify\TwoFactorAuthenticatable` trait:

```
class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
```

### [Enabling Two-Factor Authentication](#enabling-two-factor-authentication)

To begin enabling two-factor authentication, your application should make a POST request to the `/user/two-factor-authentication` endpoint.

You may retrieve the QR code SVG using the `twoFactorQrCodeSvg` method:

```
$request->user()->twoFactorQrCodeSvg();
```

### [Authenticating With Two-Factor Authentication](#authenticating-with-two-factor-authentication)

During the authentication process, Fortify will automatically redirect the user to your application's two-factor authentication challenge screen:

```
Fortify::twoFactorChallengeView(function () {
    return view('auth.two-factor-challenge');
});
```

### [Disabling Two-Factor Authentication](#disabling-two-factor-authentication)

To disable two-factor authentication, your application should make a DELETE request to the `/user/two-factor-authentication` endpoint.

## [Registration](#registration)

To begin implementing our application's registration functionality:

```
Fortify::registerView(function () {
    return view('auth.register');
});
```

### [Customizing Registration](#customizing-registration)

The user validation and creation process may be customized by modifying the `App\Actions\Fortify\CreateNewUser` action.

## [Password Reset](#password-reset)

### [Requesting a Password Reset Link](#requesting-a-password-reset-link)

To begin implementing our application's password reset functionality:

```
Fortify::requestPasswordResetLinkView(function () {
    return view('auth.forgot-password');
});
```

### [Resetting the Password](#resetting-the-password)

To finish implementing our application's password reset functionality:

```
Fortify::resetPasswordView(function (Request $request) {
    return view('auth.reset-password', ['request' => $request]);
});
```

### [Customizing Password Resets](#customizing-password-resets)

The password reset process may be customized by modifying the `App\Actions\ResetUserPassword` action.

## [Email Verification](#email-verification)

After registration, you may wish for users to verify their email address before they continue accessing your application. Ensure the `emailVerification` feature is enabled in your `fortify` configuration file:

```
Fortify::verifyEmailView(function () {
    return view('auth.verify-email');
});
```

### [Protecting Routes](#protecting-routes)

To specify that a route requires that the user has verified their email address:

```
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

## [Password Confirmation](#password-confirmation)

While building your application, you may occasionally have actions that should require the user to confirm their password before the action is performed:

```
Fortify::confirmPasswordView(function () {
    return view('auth.confirm-password');
});
```
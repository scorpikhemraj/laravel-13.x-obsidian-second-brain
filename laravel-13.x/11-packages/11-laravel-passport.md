---
title: Laravel Passport
description: Full OAuth2 server implementation for Laravel applications
url: https://laravel.com/docs/13.x/passport
tags: [packages]
cssclasses:
  - ai
  - color-purple
color: purple
---

# Laravel Passport

-   [Introduction](#introduction)
    -   [Passport or Sanctum?](#passport-or-sanctum)
-   [Installation](#installation)
    -   [Deploying Passport](#deploying-passport)
    -   [Upgrading Passport](#upgrading-passport)
-   [Configuration](#configuration)
    -   [Token Lifetimes](#token-lifetimes)
    -   [Overriding Default Models](#overriding-default-models)
    -   [Overriding Routes](#overriding-routes)
-   [Authorization Code Grant](#authorization-code-grant)
    -   [Managing Clients](#managing-clients)
    -   [Requesting Tokens](#requesting-tokens)
    -   [Managing Tokens](#managing-tokens)
    -   [Refreshing Tokens](#refreshing-tokens)
    -   [Revoking Tokens](#revoking-tokens)
    -   [Purging Tokens](#purging-tokens)
-   [Authorization Code Grant With PKCE](#code-grant-pkce)
    -   [Creating the Client](#creating-a-auth-pkce-grant-client)
    -   [Requesting Tokens](#requesting-auth-pkce-grant-tokens)
-   [Device Authorization Grant](#device-authorization-grant)
    -   [Creating a Device Code Grant Client](#creating-a-device-authorization-grant-client)
    -   [Requesting Tokens](#requesting-device-authorization-grant-tokens)
-   [Password Grant](#password-grant)
    -   [Creating a Password Grant Client](#creating-a-password-grant-client)
    -   [Requesting Tokens](#requesting-password-grant-tokens)
    -   [Requesting All Scopes](#requesting-all-scopes)
    -   [Customizing the User Provider](#customizing-the-user-provider)
    -   [Customizing the Username Field](#customizing-the-username-field)
    -   [Customizing the Password Validation](#customizing-the-password-validation)
-   [Implicit Grant](#implicit-grant)
-   [Client Credentials Grant](#client-credentials-grant)
-   [Personal Access Tokens](#personal-access-tokens)
    -   [Creating a Personal Access Client](#creating-a-personal-access-client)
    -   [Customizing the User Provider](#customizing-the-user-provider-for-pat)
    -   [Managing Personal Access Tokens](#managing-personal-access-tokens)
-   [Protecting Routes](#protecting-routes)
    -   [Via Middleware](#via-middleware)
    -   [Passing the Access Token](#passing-the-access-token)
-   [Token Scopes](#token-scopes)
    -   [Defining Scopes](#defining-scopes)
    -   [Default Scope](#default-scope)
    -   [Assigning Scopes to Tokens](#assigning-scopes-to-tokens)
    -   [Checking Scopes](#checking-scopes)
-   [SPA Authentication](#spa-authentication)
-   [Events](#events)
-   [Testing](#testing)

## [Introduction](#introduction)

[Laravel Passport](https://github.com/laravel/passport) provides a full OAuth2 server implementation for your Laravel application in a matter of minutes. Passport is built on top of the [League OAuth2 server](https://github.com/thephpleague/oauth2-server) that is maintained by Andy Millington and Simon Hamp.

This documentation assumes you are already familiar with OAuth2. If you do not know anything about OAuth2, consider familiarizing yourself with the general [terminology](https://oauth2.thephpleague.com/terminology/) and features of OAuth2 before continuing.

### [Passport or Sanctum?](#passport-or-sanctum)

Before getting started, you may wish to determine if your application would be better served by Laravel Passport or [Laravel Sanctum](/docs/13.x/sanctum). If your application absolutely needs to support OAuth2, then you should use Laravel Passport.

However, if you are attempting to authenticate a single-page application, mobile application, or issue API tokens, you should use [Laravel Sanctum](/docs/13.x/sanctum). Laravel Sanctum does not support OAuth2; however, it provides a much simpler API authentication development experience.

## [Installation](#installation)

You may install Laravel Passport via the `install:api` Artisan command:

```
1php artisan install:api --passport
php artisan install:api --passport
```

This command will publish and run the database migrations necessary for creating the tables your application needs to store OAuth2 clients and access tokens. The command will also create the encryption keys required to generate secure access tokens.

After running the `install:api` command, add the `Laravel\Passport\HasApiTokens` trait and `Laravel\Passport\Contracts\OAuthenticatable` interface to your `App\Models\User` model. This trait will provide a few helper methods to your model which allow you to inspect the authenticated user's token and scopes:

```
 1<?php 2  3namespace App\Models; 4  5use Illuminate\Database\Eloquent\Factories\HasFactory; 6use Illuminate\Foundation\Auth\User as Authenticatable; 7use Illuminate\Notifications\Notifiable; 8use Laravel\Passport\Contracts\OAuthenticatable; 9use Laravel\Passport\HasApiTokens;10 11class User extends Authenticatable implements OAuthenticatable12{13    use HasApiTokens, HasFactory, Notifiable;14}
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

Finally, in your application's `config/auth.php` configuration file, you should define an `api` authentication guard and set the `driver` option to `passport`. This will instruct your application to use Passport's `TokenGuard` when authenticating incoming API requests:

```
 1'guards' => [ 2    'web' => [ 3        'driver' => 'session', 4        'provider' => 'users', 5    ], 6  7    'api' => [ 8        'driver' => 'passport', 9        'provider' => 'users', 10    ], 11],
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

### [Deploying Passport](#deploying-passport)

When deploying Passport to your application's servers for the first time, you will likely need to run the `passport:keys` command. This command generates the encryption keys Passport needs in order to generate access tokens. The generated keys are not typically kept in source control:

```
1php artisan passport:keys
php artisan passport:keys
```

If necessary, you may define the path where Passport's keys should be loaded from. You may use the `Passport::loadKeysFrom` method to accomplish this. Typically, this method should be called from the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
1/**2 * Bootstrap any application services.3 */4public function boot(): void5{6    Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');7}
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
}
```

#### [Loading Keys From the Environment](#loading-keys-from-the-environment)

Alternatively, you may publish Passport's configuration file using the `vendor:publish` Artisan command:

```
1php artisan vendor:publish --tag=passport-config
php artisan vendor:publish --tag=passport-config
```

After the configuration file has been published, you may load your application's encryption keys by defining them as environment variables:

```
1PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----2<private key here>3-----END RSA PRIVATE KEY-----"4 5PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----6<public key here>7-----END PUBLIC KEY-----"
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

### [Upgrading Passport](#upgrading-passport)

When upgrading to a new major version of Passport, it's important that you carefully review [the upgrade guide](https://github.com/laravel/passport/blob/master/UPGRADE.md).

## [Configuration](#configuration)

### [Token Lifetimes](#token-lifetimes)

By default, Passport issues long-lived access tokens that expire after one year. If you would like to configure a longer / shorter token lifetime, you may use the `tokensExpireIn`, `refreshTokensExpireIn`, and `personalAccessTokensExpireIn` methods. These methods should be called from the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
 1use Carbon\CarbonInterval; 2  3/** 4 * Bootstrap any application services. 5 */ 6public function boot(): void 7{ 8    Passport::tokensExpireIn(CarbonInterval::days(15)); 9    Passport::refreshTokensExpireIn(CarbonInterval::days(30)); 10    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6)); 11}
use Carbon\CarbonInterval;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensExpireIn(CarbonInterval::days(15));
    Passport::refreshTokensExpireIn(CarbonInterval::days(30));
    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
}
```

The `expires_at` columns on Passport's database tables are read-only and for display purposes only. When issuing tokens, Passport stores the expiration information within the signed and encrypted tokens. If you need to invalidate a token you should [revoke it](#revoking-tokens).

### [Overriding Default Models](#overriding-default-models)

You are free to extend the models used internally by Passport by defining your own model and extending the corresponding Passport model:

```
1use Laravel\Passport\Client as PassportClient;2 3class Client extends PassportClient4{5    // ...6}
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

After defining your model, you may instruct Passport to use your custom model via the `Laravel\Passport\Passport` class. Typically, you should inform Passport about your custom models in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
 1use App\Models\Passport\AuthCode; 2use App\Models\Passport\Client; 3use App\Models\Passport\DeviceCode; 4use App\Models\Passport\RefreshToken; 5use App\Models\Passport\Token; 6use Laravel\Passport\Passport; 7  8/** 9 * Bootstrap any application services. 10 */ 11public function boot(): void 12{ 13    Passport::useTokenModel(Token::class); 14    Passport::useRefreshTokenModel(RefreshToken::class); 15    Passport::useAuthCodeModel(AuthCode::class); 16    Passport::useClientModel(Client::class); 17    Passport::useDeviceCodeModel(DeviceCode::class); 18}
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\DeviceCode;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::useTokenModel(Token::class);
    Passport::useRefreshTokenModel(RefreshToken::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::useClientModel(Client::class);
    Passport::useDeviceCodeModel(DeviceCode::class);
}
```

### [Overriding Routes](#overriding-routes)

Sometimes you may wish to customize the routes defined by Passport. To achieve this, you first need to ignore the routes registered by Passport by adding `Passport::ignoreRoutes` to the `register` method of your application's `AppServiceProvider`:

```
1use Laravel\Passport\Passport;2 3/**4 * Register any application services.5 */6public function register(): void7{8    Passport::ignoreRoutes();9}
use Laravel\Passport\Passport;

/**
 * Register any application services.
 */
public function register(): void
{
    Passport::ignoreRoutes();
}
```

Then, you may copy the routes defined by Passport in [its routes file](https://github.com/laravel/passport/blob/master/routes/web.php) to your application's `routes/web.php` file and modify them to your liking:

```
1Route::group([2    'as' => 'passport.',3    'prefix' => config('passport.path', 'oauth'),4    'namespace' => '\Laravel\Passport\Http\Controllers',5], function () {6    // Passport routes...7});
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport routes...
});
```

## [Authorization Code Grant](#authorization-code-grant)

Using OAuth2 via authorization codes is how most developers are familiar with OAuth2. When using authorization codes, a client application will redirect a user to your server where they will either approve or deny the request to issue an access token to the client.

To get started, we need to instruct Passport how to return our "authorization" view.

All the authorization view's rendering logic may be customized using the appropriate methods available via the `Laravel\Passport\Passport` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
 1use Inertia\Inertia; 2use Laravel\Passport\Passport; 3  4/** 5 * Bootstrap any application services. 6 */ 7public function boot(): void 8{ 9    // By providing a view name... 10    Passport::authorizationView('auth.oauth.authorize'); 11  12    // By providing a closure... 13    Passport::authorizationView( 14        fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [ 15            'request' => $parameters['request'], 16            'authToken' => $parameters['authToken'], 17            'client' => $parameters['client'], 18            'user' => $parameters['user'], 19            'scopes' => $parameters['scopes'], 20        ]) 21    ); 22}
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::authorizationView('auth.oauth.authorize');

    // By providing a closure...
    Passport::authorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );
}
```

Passport will automatically define the `/oauth/authorize` route that returns this view. Your `auth.oauth.authorize` template should include a form that makes a POST request to the `passport.authorizations.approve` route to approve the authorization and a form that makes a DELETE request to the `passport.authorizations.deny` route to deny the authorization. The `passport.authorizations.approve` and `passport.authorizations.deny` routes expect `state`, `client_id`, and `auth_token` fields.

### [Managing Clients](#managing-clients)

Developers building applications that need to interact with your application's API will need to register their application with yours by creating a "client". Typically, this consists of providing the name of their application and a URI that your application can redirect to after users approve their request for authorization.

#### [First-Party Clients](#managing-first-party-clients)

The simplest way to create a client is using the `passport:client` Artisan command. This command may be used to create first-party clients or testing your OAuth2 functionality. When you run the `passport:client` command, Passport will prompt you for more information about your client and will provide you with a client ID and secret:

```
1php artisan passport:client
php artisan passport:client
```

If you would like to allow multiple redirect URIs for your client, you may specify them using a comma-delimited list when prompted for the URI by the `passport:client` command. Any URIs which contain commas should be URI encoded:

```
1https://third-party-app.com/callback,https://example.com/oauth/redirect
https://third-party-app.com/callback,https://example.com/oauth/redirect
```

#### [Third-Party Clients](#managing-third-party-clients)

Since your application's users will not be able to utilize the `passport:client` command, you may use `createAuthorizationCodeGrantClient` method of the `Laravel\Passport\ClientRepository` class to register a client for a given user:

```
 1use App\Models\User; 2use Laravel\Passport\ClientRepository; 3  4$user = User::find($userId); 5  6// Creating an OAuth app client that belongs to the given user... 7$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient( 8    user: $user, 9    name: 'Example App', 10    redirectUris: ['https://third-party-app.com/callback'], 11    confidential: false, 12    enableDeviceFlow: true 13); 14  15// Retrieving all the OAuth app clients that belong to the user... 16$clients = $user->oauthApps()->get();
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

// Creating an OAuth app client that belongs to the given user...
$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient(
    user: $user,
    name: 'Example App',
    redirectUris: ['https://third-party-app.com/callback'],
    confidential: false,
    enableDeviceFlow: true
);

// Retrieving all the OAuth app clients that belong to the user...
$clients = $user->oauthApps()->get();
```

The `createAuthorizationCodeGrantClient` method returns an instance of `Laravel\Passport\Client`. You may display the `$client->id` as the client ID and `$client->plainSecret` as the client secret to the user.

### [Requesting Tokens](#requesting-tokens)

#### [Redirecting for Authorization](#requesting-tokens-redirecting-for-authorization)

Once a client has been created, developers may use their client ID and secret to request an authorization code and access token from your application. First, the consuming application should make a redirect request to your application's `/oauth/authorize` route like so:

```
 1use Illuminate\Http\Request; 2use Illuminate\Support\Str; 3  4Route::get('/redirect', function (Request $request) { 5    $request->session()->put('state', $state = Str::random(40)); 6  7    $query = http_build_query([ 8        'client_id' => 'your-client-id', 9        'redirect_uri' => 'https://third-party-app.com/callback', 10        'response_type' => 'code', 11        'scope' => 'user:read orders:create', 12        'state' => $state, 13        // 'prompt' => '', // "none", "consent", or "login" 14    ]); 15  16    return redirect('https://passport-app.test/oauth/authorize?'.$query); 17});
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

The `prompt` parameter may be used to specify the authentication behavior of the Passport application.

If the `prompt` value is `none`, Passport will always throw an authentication error if the user is not already authenticated with the Passport application. If the value is `consent`, Passport will always display the authorization approval screen, even if all scopes were previously granted to the consuming application. When the value is `login`, the Passport application will always prompt the user to re-login to the application, even if they already have an existing session.

If no `prompt` value is provided, the user will be prompted for authorization only if they have not previously authorized access to the consuming application for the requested scopes.

Remember, the `/oauth/authorize` route is already defined by Passport. You do not need to manually define this route.

#### [Approving the Request](#approving-the-request)

When receiving authorization requests, Passport will automatically respond based on the value of `prompt` parameter (if present) and may display a template to the user allowing them to approve or deny the authorization request. If they approve the request, they will be redirected back to the `redirect_uri` that was specified by the consuming application. The `redirect_uri` must match the `redirect` URL that was specified when the client was created.

Sometimes you may wish to skip the authorization prompt, such as when authorizing a first-party client. You may accomplish this by [extending the `Client` model](#overriding-default-models) and defining a `skipsAuthorization` method. If `skipsAuthorization` returns `true` the client will be approved and the user will be redirected back to the `redirect_uri` immediately, unless the consuming application has explicitly set the `prompt` parameter when redirecting for authorization:

```
 1<?php 2  3namespace App\Models\Passport; 4  5use Illuminate\Contracts\Auth\Authenticatable; 6use Laravel\Passport\Client as BaseClient; 7  8class Client extends BaseClient 9{ 10    /** 11     * Determine if the client should skip the authorization prompt. 12     * 13     * @param  \Laravel\Passport\Scope[]  $scopes 14     */ 15    public function skipsAuthorization(Authenticatable $user, array $scopes): bool 16    { 17        return $this->firstParty(); 18    } 19}
<?php

namespace App\Models\Passport;

use Illuminate\Contracts\Auth\Authenticatable;
use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * Determine if the client should skip the authorization prompt.
     *
     * @param  \Laravel\Passport\Scope[]  $scopes
     */
    public function skipsAuthorization(Authenticatable $user, array $scopes): bool
    {
        return $this->firstParty();
    }
}
```

#### [Converting Authorization Codes to Access Tokens](#requesting-tokens-converting-authorization-codes-to-access-tokens)

If the user approves the authorization request, they will be redirected back to the consuming application. The consumer should first verify the `state` parameter against the value that was stored prior to the redirect. If the state parameter matches then the consumer should issue a `POST` request to your application to request an access token. The request should include the authorization code that was issued by your application when the user approved the authorization request:

```
 1use Illuminate\Http\Request; 2use Illuminate\Support\Facades\Http; 3  4Route::get('/callback', function (Request $request) { 5    $state = $request->session()->pull('state'); 6  7    throw_unless( 8        strlen($state) > 0 && $state === $request->state, 9        InvalidArgumentException::class, 10        'Invalid state value.' 11    ); 12  13    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [ 14        'grant_type' => 'authorization_code', 15        'client_id' => 'your-client-id', 16        'client_secret' => 'your-client-secret', 17        'redirect_uri' => 'https://third-party-app.com/callback', 18        'code' => $request->code, 19    ]); 20  21    return $response->json(); 22});
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

This `/oauth/token` route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

Like the `/oauth/authorize` route, the `/oauth/token` route is defined for you by Passport. There is no need to manually define this route.

### [Managing Tokens](#managing-tokens)

You may retrieve user's authorized tokens using the `tokens` method of the `Laravel\Passport\HasApiTokens` trait. For example, this may be used to offer your users a dashboard to keep track of their connections with third-party applications:

```
 1use App\Models\User; 2use Illuminate\Database\Eloquent\Collection; 3use Illuminate\Support\Facades\Date; 4use Laravel\Passport\Token; 5  6$user = User::find($userId); 7  8// Retrieving all of the valid tokens for the user... 9$tokens = $user->tokens() 10    ->where('revoked', false) 11    ->where('expires_at', '>', Date::now()) 12    ->get(); 13  14// Retrieving all the user's connections to third-party OAuth app clients... 15$connections = $tokens->load('client') 16    ->reject(fn (Token $token) => $token->client->firstParty()) 17    ->groupBy('client_id') 18    ->map(fn (Collection $tokens) => [ 19        'client' => $tokens->first()->client, 20        'scopes' => $tokens->pluck('scopes')->flatten()->unique()->values()->all(), 21        'tokens_count' => $tokens->count(), 22    ]) 23    ->values();
use App\Models\User;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// Retrieving all of the valid tokens for the user...
$tokens = $user->tokens()
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get();

// Retrieving all the user's connections to third-party OAuth app clients...
$connections = $tokens->load('client')
    ->reject(fn (Token $token) => $token->client->firstParty())
    ->groupBy('client_id')
    ->map(fn (Collection $tokens) => [
        'client' => $tokens->first()->client,
        'scopes' => $tokens->pluck('scopes')->flatten()->unique()->values()->all(),
        'tokens_count' => $tokens->count(),
    ])
    ->values();
```

### [Refreshing Tokens](#refreshing-tokens)

If your application issues short-lived access tokens, users will need to refresh their access tokens via the refresh token that was provided to them when the access token was issued:

```
1use Illuminate\Support\Facades\Http;2 3$response = Http::asForm()->post('https://passport-app.test/oauth/token', [4    'grant_type' => 'refresh_token',5    'refresh_token' => 'the-refresh-token',6    'client_id' => 'your-client-id',7    'client_secret' => 'your-client-secret', // Required for confidential clients only...8    'scope' => 'user:read orders:create',9]);10 11return $response->json();
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // Required for confidential clients only...
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

This `/oauth/token` route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

### [Revoking Tokens](#revoking-tokens)

You may revoke a token by using the `revoke` method on the `Laravel\Passport\Token` model. You may revoke a token's refresh token using the `revoke` method on the `Laravel\Passport\RefreshToken` model:

```
1use Laravel\Passport\Passport;2use Laravel\Passport\Token;3 4$token = Passport::token()->find($tokenId);5 6// Revoke an access token...7$token->revoke();8 9// Revoke the token's refresh token...10$token->refreshToken?->revoke();11 12// Revoke all of the user's tokens...13User::find($userId)->tokens()->each(function (Token $token) {14    $token->revoke();15    $token->refreshToken?->revoke();16});
use Laravel\Passport\Passport;
use Laravel\Passport\Token;

$token = Passport::token()->find($tokenId);

// Revoke an access token...
$token->revoke();

// Revoke the token's refresh token...
$token->refreshToken?->revoke();

// Revoke all of the user's tokens...
User::find($userId)->tokens()->each(function (Token $token) {
    $token->revoke();
    $token->refreshToken?->revoke();
});
```

### [Purging Tokens](#purging-tokens)

When tokens have been revoked or expired, you might want to purge them from the database. Passport's included `passport:purge` Artisan command can do this for you:

```
1# Purge revoked and expired tokens, auth codes, and device codes...2php artisan passport:purge3 4# Only purge tokens expired for more than 6 hours...5php artisan passport:purge --hours=66 7# Only purge revoked tokens, auth codes, and device codes...8php artisan passport:purge --revoked9 10# Only purge expired tokens, auth codes, and device codes...11php artisan passport:purge --expired
# Purge revoked and expired tokens, auth codes, and device codes...
php artisan passport:purge

# Only purge tokens expired for more than 6 hours...
php artisan passport:purge --hours=6

# Only purge revoked tokens, auth codes, and device codes...
php artisan passport:purge --revoked

# Only purge expired tokens, auth codes, and device codes...
php artisan passport:purge --expired
```

You may also configure a [[05-digging-deeper/21-task-scheduling.md|scheduled job]] in your application's `routes/console.php` file to automatically prune your tokens on a schedule:

```
1use Illuminate\Support\Facades\Schedule;2 3Schedule::command('passport:purge')->hourly();
use Illuminate\Support\Facades\Schedule;

Schedule::command('passport:purge')->hourly();
```

## [Authorization Code Grant With PKCE](#code-grant-pkce)

The Authorization Code grant with "Proof Key for Code Exchange" (PKCE) is a secure way to authenticate single page applications or mobile applications to access your API. This grant should be used when you can't guarantee that the client secret will be stored confidentially or in order to mitigate the threat of having the authorization code intercepted by an attacker. A combination of a "code verifier" and a "code challenge" replaces the client secret when exchanging the authorization code for an access token.

### [Creating the Client](#creating-a-auth-pkce-grant-client)

Before your application can issue tokens via the authorization code grant with PKCE, you will need to create a PKCE-enabled client. You may do this using the `passport:client` Artisan command with the `--public` option:

```
1php artisan passport:client --public
php artisan passport:client --public
```

### [Requesting Tokens](#requesting-auth-pkce-grant-tokens)

#### [Code Verifier and Code Challenge](#code-verifier-code-challenge)

As this authorization grant does not provide a client secret, developers will need to generate a combination of a code verifier and a code challenge in order to request a token.

The code verifier should be a random string of between 43 and 128 characters containing letters, numbers, and `"-"`, `"."`, `"_"`, `"~"` characters, as defined in the [RFC 7636 specification](https://tools.ietf.org/html/rfc7636).

The code challenge should be a Base64 encoded string with URL and filename-safe characters. The trailing `'='` characters should be removed and no line breaks, whitespace, or other additional characters should be present.

```
1$encoded = base64_encode(hash('sha256', $codeVerifier, true));2 3$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
$encoded = base64_encode(hash('sha256', $codeVerifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

#### [Redirecting for Authorization](#code-grant-pkce-redirecting-for-authorization)

Once a client has been created, you may use the client ID and the generated code verifier and code challenge to request an authorization code and access token from your application. First, the consuming application should make a redirect request to your application's `/oauth/authorize` route:

```
 1use Illuminate\Http\Request; 2use Illuminate\Support\Str; 3  4Route::get('/redirect', function (Request $request) { 5    $request->session()->put('state', $state = Str::random(40)); 6  7    $request->session()->put( 8        'code_verifier', $codeVerifier = Str::random(128) 9    ); 10  11    $codeChallenge = strtr(rtrim( 12        base64_encode(hash('sha256', $codeVerifier, true)) 13    , '='), '+/', '-_'); 14  15    $query = http_build_query([ 16        'client_id' => 'your-client-id', 17        'redirect_uri' => 'https://third-party-app.com/callback', 18        'response_type' => 'code', 19        'scope' => 'user:read orders:create', 20        'state' => $state, 21        'code_challenge' => $codeChallenge, 22        'code_challenge_method' => 'S256', 23        // 'prompt' => '', // "none", "consent", or "login" 24    ]); 25  26    return redirect('https://passport-app.test/oauth/authorize?'.$query); 27});
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put(
        'code_verifier', $codeVerifier = Str::random(128)
    );

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $codeVerifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

#### [Converting Authorization Codes to Access Tokens](#code-grant-pkce-converting-authorization-codes-to-access-tokens)

If the user approves the authorization request, they will be redirected back to the consuming application. The consumer should verify the `state` parameter against the value that was stored prior to the redirect, as in the standard Authorization Code Grant.

If the state parameter matches, the consumer should issue a `POST` request to your application to request an access token. The request should include the authorization code that was issued by your application when the user approved the authorization request along with the originally generated code verifier:

```
 1use Illuminate\Http\Request; 2use Illuminate\Support\Facades\Http; 3  4Route::get('/callback', function (Request $request) { 5    $state = $request->session()->pull('state'); 6  7    $codeVerifier = $request->session()->pull('code_verifier'); 8  9    throw_unless( 10        strlen($state) > 0 && $state === $request->state, 11        InvalidArgumentException::class 12    ); 13  14    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [ 15        'grant_type' => 'authorization_code', 16        'client_id' => 'your-client-id', 17        'redirect_uri' => 'https://third-party-app.com/callback', 18        'code_verifier' => $codeVerifier, 19        'code' => $request->code, 20    ]); 21  22    return $response->json(); 23});
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

## [Device Authorization Grant](#device-authorization-grant)

The OAuth2 device authorization grant allows browserless or limited input devices, such as TVs and game consoles, to obtain an access token by exchanging a "device code". When using device flow, the device client will instruct the user to use a secondary device, such as a computer or a smartphone and connect to your server where they will enter the provided "user code" and either approve or deny the access request.

To get started, we need to instruct Passport how to return our "user code" and "authorization" views.

All the authorization view's rendering logic may be customized using the appropriate methods available via the `Laravel\Passport\Passport` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\AppServiceProvider` class.

```
 1use Inertia\Inertia; 2use Laravel\Passport\Passport; 3  4/** 5 * Bootstrap any application services. 6 */ 7public function boot(): void 8{ 9    // By providing a view name... 10    Passport::deviceUserCodeView('auth.oauth.device.user-code'); 11    Passport::deviceAuthorizationView('auth.oauth.device.authorize'); 12  13    // By providing a closure... 14    Passport::deviceUserCodeView( 15        fn ($parameters) => Inertia::render('Auth/OAuth/Device/UserCode') 16    ); 17  18    Passport::deviceAuthorizationView( 19        fn ($parameters) => Inertia::render('Auth/OAuth/Device/Authorize', [ 20            'request' => $parameters['request'], 21            'authToken' => $parameters['authToken'], 22            'client' => $parameters['client'], 23            'user' => $parameters['user'], 24            'scopes' => $parameters['scopes'], 25        ]) 26    ); 27  28    // ... 29}
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // By providing a view name...
    Passport::deviceUserCodeView('auth.oauth.device.user-code');
    Passport::deviceAuthorizationView('auth.oauth.device.authorize');

    // By providing a closure...
    Passport::deviceUserCodeView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/UserCode')
    );

    Passport::deviceAuthorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );

    // ...
}
```

Passport will automatically define routes that return these views. Your `auth.oauth.device.user-code` template should include a form that makes a GET request to the `passport.device.authorizations.authorize` route. The `passport.device.authorizations.authorize` route expects a `user_code` query parameter.

Your `auth.oauth.device.authorize` template should include a form that makes a POST request to the `passport.device.authorizations.approve` route to approve the authorization and a form that makes a DELETE request to the `passport.device.authorizations.deny` route to deny the authorization. The `passport.device.authorizations.approve` and `passport.device.authorizations.deny` routes expect `state`, `client_id`, and `auth_token` fields.

### [Creating a Device Authorization Grant Client](#creating-a-device-authorization-grant-client)

Before your application can issue tokens via the device authorization grant, you will need to create a device flow enabled client. You may do this using the `passport:client` Artisan command with the `--device` option. This command will create a first-party device flow enabled client and provide you with a client ID and secret:

```
1php artisan passport:client --device
php artisan passport:client --device
```

Additionally, you may use `createDeviceAuthorizationGrantClient` method on the `ClientRepository` class to register a third-party client that belongs to the given user:

```
 1use App\Models\User; 2use Laravel\Passport\ClientRepository; 3  4$user = User::find($userId); 5  6$client = app(ClientRepository::class)->createDeviceAuthorizationGrantClient( 7    user: $user, 8    name: 'Example Device', 9    confidential: false, 10);
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

$client = app(ClientRepository::class)->createDeviceAuthorizationGrantClient(
    user: $user,
    name: 'Example Device',
    confidential: false,
);
```

### [Requesting Tokens](#requesting-device-authorization-grant-tokens)

#### [Requesting a Device Code](#device-code)

Once a client has been created, developers may use their client ID to request a device code from your application. First, the consuming device should make a `POST` request to your application's `/oauth/device/code` route to request a device code:

```
1use Illuminate\Support\Facades\Http;2 3$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [4    'client_id' => 'your-client-id',5    'scope' => 'user:read orders:create',6]);7 8return $response->json();
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

This will return a JSON response containing `device_code`, `user_code`, `verification_uri`, `interval`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the device code expires. The `interval` attribute contains the number of seconds the consuming device should wait between requests when polling `/oauth/token` route to avoid rate limit errors.

Remember, the `/oauth/device/code` route is already defined by Passport. You do not need to manually define this route.

#### [Displaying the Verification URI and User Code](#user-code)

Once a device code request has been obtained, the consuming device should instruct the user to use another device and visit the provided `verification_uri` and enter the `user_code` in order to approve the authorization request.

#### [Polling Token Request](#polling-token-request)

Since the user will be using a separate device to grant (or deny) access, the consuming device should poll your application's `/oauth/token` route to determine when the user has responded to the request. The consuming device should use the minimum polling `interval` provided in the JSON response when requesting device code to avoid rate limit errors:

```
 1use Illuminate\Support\Facades\Http; 2use Illuminate\Support\Sleep; 3  4$interval = 5; 5  6do { 7    Sleep::for($interval)->seconds(); 8  9    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [ 10        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code', 11        'client_id' => 'your-client-id', 12        'client_secret' => 'your-client-secret', // Required for confidential clients only... 13        'device_code' => 'the-device-code', 14    ]); 15  16    if ($response->json('error') === 'slow_down') { 17        $interval += 5; 18    } 19} while (in_array($response->json('error'), ['authorization_pending', 'slow_down'])); 20  21return $response->json();
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Sleep;

$interval = 5;

do {
    Sleep::for($interval)->seconds();

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret', // Required for confidential clients only...
        'device_code' => 'the-device-code',
    ]);

    if ($response->json('error') === 'slow_down') {
        $interval += 5;
    }
} while (in_array($response->json('error'), ['authorization_pending', 'slow_down']));

return $response->json();
```

If the user has approved the authorization request, this will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes. The `expires_in` attribute contains the number of seconds until the access token expires.

## [Password Grant](#password-grant)

We no longer recommend using password grant tokens. Instead, you should choose [a grant type that is currently recommended by OAuth2 Server](https://oauth2.thephpleague.com/authorization-server/which-grant/).

The OAuth2 password grant allows your other first-party clients, such as a mobile application, to obtain an access token using an email address / username and password. This allows you to issue access tokens securely to your first-party clients without requiring your users to go through the entire OAuth2 authorization code redirect flow.

To enable the password grant, call the `enablePasswordGrant` method in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
1/**2 * Bootstrap any application services.3 */4public function boot(): void5{6    Passport::enablePasswordGrant();7}
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enablePasswordGrant();
}
```

### [Creating a Password Grant Client](#creating-a-password-grant-client)

Before your application can issue tokens via the password grant, you will need to create a password grant client. You may do this using the `passport:client` Artisan command with the `--password` option.

```
1php artisan passport:client --password
php artisan passport:client --password
```

### [Requesting Tokens](#requesting-password-grant-tokens)

Once a password grant client has been created, you may request an access token by issuing a `POST` request to the `/oauth/token` route with the user's email address and password:

```
1use Illuminate\Support\Facades\Http;2 3$response = Http::asForm()->post('https://passport-app.test/oauth/token', [4    'grant_type' => 'password',5    'client_id' => 'your-client-id',6    'client_secret' => 'your-client-secret',7    'username' => 'taylor@example.com',8    'password' => 'my-password',9    'scope' => '',10]);11 12return $response->json();
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'username' => 'taylor@example.com',
    'password' => 'my-password',
    'scope' => '',
]);

return $response->json();
```

Remember, access tokens are long-lived. However, the user may revoke the token at any time using the `revoke` method on the `Laravel\Passport\Token` model.

#### [Requesting All Scopes](#requesting-all-scopes)

When using the password grant, you may wish to authorize the token for all of the scopes that the user currently has. You can accomplish this via the `scope` parameter:

```
1$response = Http::asForm()->post('https://passport-app.test/oauth/token', [2    // ...3    'scope' => '*',4]);
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    // ...
    'scope' => '*',
]);
```

#### [Customizing the User Provider](#customizing-the-user-provider)

If your application uses more than one user authentication provider, you will need to configure which user provider the password grant uses by passing the `provider` option to the `setClient` method:

```
1use Laravel\Passport\Passport;2 3/**4 * Bootstrap any application services.5 */6public function boot(): void7{8    Passport::setClientResolver(fn ($client) =>9        app(UserProvider::class)->retrieveById($client->user_id)10    );11}
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::setClientResolver(fn ($client) =>
        app(UserProvider::class)->retrieveById($client->user_id)
    );
}
```

Or you may use the `viaCredentials` method to retrieve a user when issuing a token:

```
1use Illuminate\Support\Facades\Hash;2use App\Models\User;3use Laravel\Passport\Passport;4 5/**6 * Bootstrap any application services.7 */8public function boot(): void9{10    Passport::viaCredentials(function ($client) {11        $user = User::where('email', request('username'))->first();12 13        if (! $user || ! Hash::check(request('password'), $user->password)) {14            return;15        }16 17        return $user;18    });19}
use Illuminate\Support\Facades\Hash;
use App\Models\User;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::viaCredentials(function ($client) {
        $user = User::where('email', request('username'))->first();

        if (! $user || ! Hash::check(request('password'), $user->password)) {
            return;
        }

        return $user;
    });
}
```

#### [Customizing the Username Field](#customizing-the-username-field)

When using the password grant, Passport will attempt to authenticate using the `email` field on the `users` database table by default. You may customize this behavior by defining a `findUserForPasswordGrant` method on your application's `User` model:

```
1<?php 2  3namespace App\Models; 4  5use Illuminate\Foundation\Auth\User as Authenticatable; 6use Laravel\Passport\HasApiTokens; 7  8class User extends Authenticatable 9{10    use HasApiTokens;11 12    /**13     * Find the user for the password grant.14     */15    public function findForPasswordGrant(string $username):16    {17        return $this->where('username', $username)->first();18    }19}
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens;

    /**
     * Find the user for the password grant.
     */
    public function findForPasswordGrant(string $username): self
    {
        return $this->where('username', $username)->first();
    }
}
```

#### [Customizing the Password Validation](#customizing-the-password-validation)

You may customize the password validation logic by defining a `validateForPasswordGrant` method on your application's `User` model:

```
1<?php 2  3namespace App\Models; 4  5use Illuminate\Foundation\Auth\User as Authenticatable; 6use Laravel\Passport\HasApiTokens; 7  8class User extends Authenticatable 9{10    use HasApiTokens;11 12    /**13     * Validate the password for the password grant.14     */15    public function validateForPasswordGrant(string $password): bool16    {17        return Hash::check($password, $this->password);18    }19}
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens;

    /**
     * Validate the password for the password grant.
     */
    public function validateForPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

## [Implicit Grant](#implicit-grant)

The implicit grant is a simplified OAuth2 flow that returns an access token to the client without requiring an authorization code exchange. This is typically used for JavaScript applications that are unable to securely store client secrets.

To enable the implicit grant, call the `enableImplicitGrant` method in the `boot` method of your `AppServiceProvider`:

```
1/**2 * Bootstrap any application services.3 */4public function boot(): void5{6    Passport::enableImplicitGrant();7}
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

Once the grant has been enabled, developers may use their client ID to request an access token from your application. The consuming application should make a redirect request to your application's `/oauth/authorize` route like so:

```
1$query = http_build_query([2    'client_id' => 'your-client-id',3    'redirect_uri' => 'https://third-party-app.com/callback',4    'response_type' => 'token',5    'scope' => 'user:read orders:create',6]);7 8return redirect('https://passport-app.test/oauth/authorize?'.$query);
$query = http_build_query([
    'client_id' => 'your-client-id',
    'redirect_uri' => 'https://third-party-app.com/callback',
    'response_type' => 'token',
    'scope' => 'user:read orders:create',
]);

return redirect('https://passport-app.test/oauth/authorize?'.$query);
```

Remember, the `/oauth/authorize` route is already defined by Passport. You do not need to manually define this route. After approving the authorization request, the user will be redirected back to the `redirect_uri` with a `#token=` in the URL fragment. The fragment will contain the `access_token`, `expires_in`, and `token_type` parameters:

```
1https://third-party-app.com/callback#access_token=eyJpdiI6IjNk...&expires_in=3600&token_type=Bearer&scope=user:read+orders:create
https://third-party-app.com/callback#access_token=eyJpdiI6IjNk...&expires_in=3600&token_type=Bearer&scope=user:read+orders:create
```

## [Client Credentials Grant](#client-credentials-grant)

The client credentials grant is suitable for machine-to-machine authentication. For example, you might use this in a scheduled job that performs maintenance tasks against an API.

Before using the client credentials grant, you will need to create a client for your application using the `passport:client` Artisan command:

```
1php artisan passport:client --client
php artisan passport:client --client
```

Then, to request an access token, your application should make a request to the `/oauth/token` endpoint using the client's ID and secret:

```
1use Illuminate\Support\Facades\Http;2 3$response = Http::asForm()->post('https://passport-app.test/oauth/token', [4    'grant_type' => 'client_credentials',5    'client_id' => 'client-id',6    'client_secret' => 'client-secret',7]);8 9return $response->json();
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
]);

return $response->json();
```

This route will return a JSON response containing `access_token`, `refresh_token`, and `expires_in` attributes.

The client's ID and secret can be stored in environment variables, or configured in the `services` configuration file:

```
1'passport' => [2    'client_id' => env('PASSPORT_CLIENT_ID'),3    'client_secret' => env('PASSPORT_CLIENT_SECRET'),4],
'passport' => [
    'client_id' => env('PASSPORT_CLIENT_ID'),
    'client_secret' => env('PASSPORT_CLIENT_SECRET'),
],
```

The client credentials grant does not provide a "user" for the token, so you can use Passport's scope system to define what API routes the token can access:

```
1$response = Http::asForm()->post('https://passport-app.test/oauth/token', [2    'grant_type' => 'client_credentials',3    'client_id' => 'client-id',4    'client_secret' => 'client-secret',5    'scope' => 'check-status',6]);7 8return $response->json();
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => 'check-status',
]);

return $response->json();
```

## [Personal Access Tokens](#personal-access-tokens)

Sometimes your users may wish to issue access tokens themselves without going through the typical authorization code redirect flow. You may allow your users to issue tokens for themselves by using Passport's "personal access" feature.

### [Creating a Personal Access Client](#creating-a-personal-access-client)

Before your users can issue personal access tokens, you will need to create a personal access client. You may do this using the `passport:client` Artisan command with the `--personal` option:

```
1php artisan passport:client --personal
php artisan passport:client --personal
```

If you have already run the `passport:install` command, you may run the `passport:client --personal` command to create your personal access client. If you would like to use a different name for the personal access client, you may use the `--name` option to provide a custom name:

```
1php artisan passport:client --personal --name="Personal Access Client"
php artisan passport:client --personal --name="Personal Access Client"
```

#### [Customizing the User Provider](#customizing-the-user-provider-for-pat)

If your application uses more than one user authentication provider, you may need to configure which user provider to use when creating personal access tokens by passing the `provider` option to the `setClient` method:

```
1use Laravel\Passport\Passport;2 3/**4 * Bootstrap any application services.5 */6public function boot(): void7{8    Passport::setClientResolver(fn ($client) =>9        app(UserProvider::class)->retrieveById($client->user_id)10    );11}
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::setClientResolver(fn ($client) =>
        app(UserProvider::class)->retrieveById($client->user_id)
    );
}
```

### [Managing Personal Access Tokens](#managing-personal-access-tokens)

Once a personal access client has been created, you may issue tokens for a user using the `createToken` method on the `User` model instance. The `createToken` method accepts the token name as its first argument and an optional array of [scopes](#token-scopes) as its second argument:

```
1$token = $user->createToken('my-token')->accessToken;
$token = $user->createToken('my-token')->accessToken;
```

You may also retrieve all of the user's tokens using the `tokens` relationship:

```
1$tokens = $user->tokens;
$tokens = $user->tokens;
```

You may revoke a token using the `revoke` method on the `Laravel\Passport\Token` model:

```
1$token->revoke();
$token->revoke();
```

## [Protecting Routes](#protecting-routes)

### [Via Middleware](#via-middleware)

Passport includes an `auth:api` guard that will validate the access token on incoming requests. Once you have configured your application's `config/auth.php` file to use the `passport` driver, you may use the `auth:api` middleware to protect routes:

```
1Route::get('/user', function () {2    // ...3})->middleware('auth:api');
Route::get('/user', function () {
    // ...
})->middleware('auth:api');
```

By default, Passport's token guard will use the `users` provider. You may customize which provider is used by setting the `provider` option in your `config/auth.php` configuration file:

```
1'guards' => [2    'api' => [3        'driver' => 'passport', 4        'provider' => 'users', // 'users' or 'admins'... 5    ], 6],
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users', // 'users' or 'admins'...
    ],
],
```

### [Passing the Access Token](#passing-the-access-token)

Routes protected by Passport expect access tokens to be included in the request as a "Bearer" token in the `Authorization` header. You may also pass the token as a query parameter with the key `access_token`:

```
1curl -X GET https://laravel.app/api/user -H "Authorization: Bearer {token}"
curl -X GET https://laravel.app/api/user -H "Authorization: Bearer {token}"
```

## [Token Scopes](#token-scopes)

### [Defining Scopes](#defining-scopes)

Token scopes allow your API clients to request a specific set of permissions when requesting an access token. To define the scopes that your API supports, you may use the ` Passport::tokensCan` method in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
1use Laravel\Passport\Passport;2 3/**4 * Bootstrap any application services.5 */6public function boot(): void7{8    Passport::tokensCan([9        'place-orders' => 'Place orders',10        'check-status' => 'Check order status',11    ]);12}
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);
}
```

### [Default Scope](#default-scope)

If you would like to provide a default scope to tokens, you may use the `setDefaultScopes` method. Typically, this method should be called in the `boot` method of your application's `AppServiceProvider`:

```
1use Laravel\Passport\Passport;2 3/**4 * Bootstrap any application services.5 */6public function boot(): void7{8    Passport::setDefaultScopes([9        'check-status',10    ]);11}
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::setDefaultScopes([
        'check-status',
    ]);
}
```

### [Assigning Scopes to Tokens](#assigning-scopes-to-tokens)

When requesting an access token using the authorization code grant, consumers should specify the desired scopes as the `scope` parameter:

```
1$query = http_build_query([2    'client_id' => 'client-id',3    'redirect_uri' => 'callback',4    'response_type' => 'code',5    'scope' => 'place-orders check-status',6    // ...7]);
$query = http_build_query([
    'client_id' => 'client-id',
    'redirect_uri' => 'callback',
    'response_type' => 'code',
    'scope' => 'place-orders check-status',
    // ...
]);
```

When using the password grant, you may specify the desired scopes using the `scope` parameter:

```
1$response = Http::asForm()->post('https://passport-app.test/oauth/token', [2    // ...3    'scope' => 'place-orders check-status',4]);
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    // ...
    'scope' => 'place-orders check-status',
]);
```

When using the client credentials grant, you may specify the desired scopes using the `scope` parameter:

```
1$response = Http::asForm()->post('https://passport-app.test/oauth/token', [2    'grant_type' => 'client_credentials',3    'client_id' => 'client-id',4    'client_secret' => 'client-secret',5    'scope' => 'place-orders',6]);7 8return $response->json();
$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => 'place-orders',
]);

return $response->json();
```

When issuing personal access tokens using the `createToken` method, you may pass an array of scopes as the second argument:

```
1$token = $user->createToken('my-token', ['place-orders', 'check-status'])->accessToken;
$token = $user->createToken('my-token', ['place-orders', 'check-status'])->accessToken;
```

### [Checking Scopes](#checking-scoutes)

Passport includes two middleware that can be used to verify that an incoming request has been authenticated with a token that has a given scope:

```
1Route::get('/orders', function () {2    // Access token has both "place-orders" and "check-status" scopes...3})->middleware('scopes:place-orders,check-status');
Route::get('/orders', function () {
    // Access token has both "place-orders" and "check-status" scopes...
})->middleware('scopes:place-orders,check-status');
```

```
1Route::get('/orders', function () {2    // Access token has either "place-orders" or "check-status" scope...3})->middleware('scope:place-orders,check-status');
Route::get('/orders', function () {
    // Access token has either "place-orders" or "check-status" scope...
})->middleware('scope:place-orders,check-status');
```

You may also check if a token has a given scope using the `can` method on the authenticated user instance:

```
1if ($request->user()->tokenCan('place-orders')) {2    // ...3}
if ($request->user()->tokenCan('place-orders')) {
    // ...
}
```

## [SPA Authentication](#spa-authentication)

If you have a JavaScript driven single-page application (SPA) that needs to communicate with an API, you will need to configure your application to support CORS requests. If you want to use Passport's built-in CORS support, you can publish the Passport configuration file using the `vendor:publish` Artisan command:

```
1php artisan vendor:publish --tag=passport-config
php artisan vendor:publish --tag=passport-config
```

After publishing the configuration, you should set the `stateful` domains in your `config/cors.php` file. These domains will be used to determine which domains can utilize stateful authentication using your application's session:

```
1'stateful' => explode(',', env('CORS_STATEFUL_DOMAINS', 'localhost,127.0.0.1')),
'stateful' => explode(',', env('CORS_STATEFUL_DOMAINS', 'localhost,127.0.0.1')),
```

To enable stateful CORS, the following environment variables should be configured in your application's `.env` file:

```
1CORS_STATEFUL_DOMAINS=localhost:3000,localhost:8080
CORS_STATEFUL_DOMAINS=localhost:3000,localhost:8080
```

If you are using a different domain in different environments, you may wish to use a different value for `CORS_STATEFUL_DOMAINS` in your `.env` file per environment.

After configuring your stateful domains, you will need to configure the authentication guard to use stateful session authentication:

```
1'guards' => [2    'web' => [3        'driver' => 'session', 4        'provider' => 'users', 5    ], 6],
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

Then, in your application's `bootstrap/app.php` file, you should configure the `authenticate` middleware to use the stateful cookie authentication:

```
1->withMiddleware(function (Middleware $middleware) {2    $middleware->statefulApi();3})
->withMiddleware(function (Middleware $middleware) {
    $middleware->statefulApi();
})
```

After configuring stateful API authentication, your SPA will be able to authenticate with your Laravel application using the standard Laravel session cookie.

For more information on how to use this authentication, please consult the documentation on [Sanctum](/docs/13.x/sanctum).

## [Events](#events)

Passport raises events when issuing access tokens and refresh tokens. You may use these events to modify or revoke access tokens. You may attach listeners to these events in your application's `EventServiceProvider`:

```
1use App\Listeners\RevokeOldTokens;2use Laravel\Passport\Events\AccessTokenCreated;3use Laravel\Passport\Events\RefreshTokenCreated;4 5/**6 * The event listener mappings for the application.7 *8 * @var array<class-string, array<class-string>>9 */10protected $listen = [11    AccessTokenCreated::class => [12        RevokeOldTokens::class,13    ],14    RefreshTokenCreated::class => [15        RevokeOldTokens::class,16    ],17];
use App\Listeners\RevokeOldTokens;
use Laravel\Passport\Events\AccessTokenCreated;
use Laravel\Passport\Events\RefreshTokenCreated;

/**
 * The event listener mappings for the application.
 *
 * @var array<class-string, array<class-string>>
 */
protected $listen = [
    AccessTokenCreated::class => [
        RevokeOldTokens::class,
    ],
    RefreshTokenCreated::class => [
        RevokeOldTokens::class,
    ],
];
```

## [Testing](#testing)

When testing, you may use the `actingAs` method to simulate a user authenticating with a token. The `actingAs` method accepts the user instance and an optional array of scopes to authorize for the token:

```
1use Laravel\Passport\Passport;2use App\Models\User;3 4public function test_server_status()5{6    Passport::actingAs(7        User::factory()->create(),8        ['check-status']9    );10 11    $response = $this->getJson('/api/status');12 13    $response->assertStatus(200);14}
use Laravel\Passport\Passport;
use App\Models\User;

public function test_server_status()
{
    Passport::actingAs(
        User::factory()->create(),
        ['check-status']
    );

    $response = $this->getJson('/api/status');

    $response->assertStatus(200);
}
```

If you are using [Pest PHP](https://pestphp.com), you may use the `actingAs` function:

```
1use Laravel\Passport\Passport;2 3it('server status', function () {4    Passport::actingAs(5        User::factory()->create(),6        ['check-status']7    );8 9    $response = $this->getJson('/api/status');10 11    $response->assertStatus(200);12});
use Laravel\Passport\Passport;

it('server status', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['check-status']
    );

    $response = $this->getJson('/api/status');

    $response->assertStatus(200);
});
```

You may also simulate a client authentication using the `actingAsClient` method:

```
1use Laravel\Passport\Passport;2use Laravel\Passport\Client;3 4public function test_create_order()5{6    Passport::actingAsClient(7        Client::factory()->create(),8        ['place-orders']9    );10 11    $response = $this->postJson('/api/orders');12 113    $response->assertStatus(201);14}
use Laravel\Passport\Passport;
use Laravel\Passport\Client;

public function test_create_order()
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['place-orders']
    );

    $response = $this->postJson('/api/orders');

    $response->assertStatus(201);
}
```

Copy as markdown

### On this page

-   [Introduction](#introduction)
    -   [Passport or Sanctum?](#passport-or-sanctum)
-   [Installation](#installation)
    -   [Deploying Passport](#deploying-passport)
    -   [Upgrading Passport](#upgrading-passport)
-   [Configuration](#configuration)
    -   [Token Lifetimes](#token-lifetimes)
    -   [Overriding Default Models](#overriding-default-models)
    -   [Overriding Routes](#overriding-routes)
-   [Authorization Code Grant](#authorization-code-grant)
    -   [Managing Clients](#managing-clients)
    -   [Requesting Tokens](#requesting-tokens)
    -   [Managing Tokens](#managing-tokens)
    -   [Refreshing Tokens](#refreshing-tokens)
    -   [Revoking Tokens](#revoking-tokens)
    -   [Purging Tokens](#purging-tokens)
-   [Authorization Code Grant With PKCE](#code-grant-pkce)
    -   [Creating the Client](#creating-a-auth-pkce-grant-client)
    -   [Requesting Tokens](#requesting-auth-pkce-grant-tokens)
-   [Device Authorization Grant](#device-authorization-grant)
    -   [Creating a Device Code Grant Client](#creating-a-device-authorization-grant-client)
    -   [Requesting Tokens](#requesting-device-authorization-grant-tokens)
-   [Password Grant](#password-grant)
    -   [Creating a Password Grant Client](#creating-a-password-grant-client)
    -   [Requesting Tokens](#requesting-password-grant-tokens)
    -   [Requesting All Scopes](#requesting-all-scopes)
    -   [Customizing the User Provider](#customizing-the-user-provider)
    -   [Customizing the Username Field](#customizing-the-username-field)
    -   [Customizing the Password Validation](#customizing-the-password-validation)
-   [Implicit Grant](#implicit-grant)
-   [Client Credentials Grant](#client-credentials-grant)
-   [Personal Access Tokens](#personal-access-tokens)
    -   [Creating a Personal Access Client](#creating-a-personal-access-client)
    -   [Customizing the User Provider](#customizing-the-user-provider-for-pat)
    -   [Managing Personal Access Tokens](#managing-personal-access-tokens)
-   [Protecting Routes](#protecting-routes)
    -   [Via Middleware](#via-middleware)
    -   [Passing the Access Token](#passing-the-access-token)
-   [Token Scopes](#token-scopes)
    -   [Defining Scopes](#defining-scopes)
    -   [Default Scope](#default-scope)
    -   [Assigning Scopes to Tokens](#assigning-scopes-to-tokens)
    -   [Checking Scopes](#checking-scopes)
-   [SPA Authentication](#spa-authentication)
-   [Events](#events)
-   [Testing](#testing)
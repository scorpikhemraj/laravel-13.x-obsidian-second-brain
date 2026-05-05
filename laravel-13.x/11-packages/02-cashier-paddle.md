---
title: Laravel Cashier (Paddle)
description: Provides an expressive, fluent interface to Paddle's subscription billing services.
url: https://laravel.com/docs/13.x/cashier-paddle
tags: [packages]
---

#Laravel Cashier (Paddle)

-   [Introduction](#introduction)
-   [Upgrading Cashier](#upgrading-cashier)
-   [Installation](#installation)
    -   [Paddle Sandbox](#paddle-sandbox)
-   [Configuration](#configuration)
    -   [Billable Model](#billable-model)
    -   [API Keys](#api-keys)
    -   [Paddle JS](#paddle-js)
    -   [Currency Configuration](#currency-configuration)
    -   [Overriding Default Models](#overriding-default-models)
-   [Quickstart](#quickstart)
    -   [Selling Products](#quickstart-selling-products)
    -   [Selling Subscriptions](#quickstart-selling-subscriptions)
-   [Checkout Sessions](#checkout-sessions)
    -   [Overlay Checkout](#overlay-checkout)
    -   [Inline Checkout](#inline-checkout)
    -   [Guest Checkouts](#guest-checkouts)
-   [Price Previews](#price-previews)
    -   [Customer Price Previews](#customer-price-previews)
    -   [Discounts](#price-discounts)
-   [Customers](#customers)
    -   [Customer Defaults](#customer-defaults)
    -   [Retrieving Customers](#retrieving-customers)
    -   [Creating Customers](#creating-customers)
-   [Subscriptions](#subscriptions)
    -   [Creating Subscriptions](#creating-subscriptions)
    -   [Checking Subscription Status](#checking-subscription-status)
    -   [Subscription Single Charges](#subscription-single-charges)
    -   [Updating Payment Information](#updating-payment-information)
    -   [Changing Plans](#changing-plans)
    -   [Subscription Quantity](#subscription-quantity)
    -   [Subscriptions With Multiple Products](#subscriptions-with-multiple-products)
    -   [Multiple Subscriptions](#multiple-subscriptions)
    -   [Pausing Subscriptions](#pausing-subscriptions)
    -   [Canceling Subscriptions](#canceling-subscriptions)
-   [Subscription Trials](#subscription-trials)
    -   [With Payment Method Up Front](#with-payment-method-up-front)
    -   [Without Payment Method Up Front](#without-payment-method-up-front)
    -   [Extend or Activate a Trial](#extend-or-activate-a-trial)
-   [Handling Paddle Webhooks](#handling-paddle-webhooks)
    -   [Defining Webhook Event Handlers](#defining-webhook-event-handlers)
    -   [Verifying Webhook Signatures](#verifying-webhook-signatures)
-   [Single Charges](#single-charges)
    -   [Charging for Products](#charging-for-products)
    -   [Refunding Transactions](#refunding-transactions)
    -   [Crediting Transactions](#crediting-transactions)
-   [Transactions](#transactions)
    -   [Past and Upcoming Payments](#past-and-upcoming-payments)
-   [Testing](#testing)

## [Introduction](#introduction)

This documentation is for Cashier Paddle 2.x's integration with Paddle Billing. If you're still using Paddle Classic, you should use [Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x).

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) provides an expressive, fluent interface to [Paddle's](https://paddle.com) subscription billing services. It handles almost all of the boilerplate subscription billing code you are dreading. In addition to basic subscription management, Cashier can handle: swapping subscriptions, subscription "quantities", subscription pausing, cancelation grace periods, and more.

Before digging into Cashier Paddle, we recommend you also review Paddle's [concept guides](https://developer.paddle.com/concepts/overview) and [API documentation](https://developer.paddle.com/api-reference/overview).

## [Upgrading Cashier](#upgrading-cashier)

When upgrading to a new version of Cashier, it's important that you carefully review [the upgrade guide](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md).

## [Installation](#installation)

First, install the Cashier package for Paddle using the Composer package manager:

```
composer require laravel/cashier-paddle
```

Next, you should publish the Cashier migration files using the `vendor:publish` Artisan command:

```
php artisan vendor:publish --tag="cashier-migrations"
```

Then, you should run your application's database migrations. The Cashier migrations will create a new `customers` table. In addition, new `subscriptions` and `subscription_items` tables will be created to store all of your customer's subscriptions. Lastly, a new `transactions` table will be created to store all of the Paddle transactions associated with your customers:

```
php artisan migrate
```

To ensure Cashier properly handles all Paddle events, remember to [set up Cashier's webhook handling](#handling-paddle-webhooks).

### [Paddle Sandbox](#paddle-sandbox)

During local and staging development, you should [register a Paddle Sandbox account](https://sandbox-login.paddle.com/signup). This account will give you a sandboxed environment to test and develop your applications without making actual payments. You may use Paddle's [test card numbers](https://developer.paddle.com/concepts/payment-methods/credit-debit-card#test-payment-method) to simulate various payment scenarios.

When using the Paddle Sandbox environment, you should set the `PADDLE_SANDBOX` environment variable to `true` within your application's `.env` file:

```
PADDLE_SANDBOX=true
```

After you have finished developing your application you may [apply for a Paddle vendor account](https://paddle.com). Before your application is placed into production, Paddle will need to approve your application's domain.

## [Configuration](#configuration)

### [Billable Model](#billable-model)

Before using Cashier, you must add the `Billable` trait to your user model definition. This trait provides various methods to allow you to perform common billing tasks, such as creating subscriptions and updating payment method information:

```
use Laravel\Paddle\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

If you have billable entities that are not users, you may also add the trait to those classes:

```
use Illuminate\Database\Eloquent\Model;
use Laravel\Paddle\Billable;

class Team extends Model
{
    use Billable;
}
```

### [API Keys](#api-keys)

Next, you should configure your Paddle keys in your application's `.env` file. You can retrieve your Paddle API keys from the Paddle control panel:

```
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

The `PADDLE_SANDBOX` environment variable should be set to `true` when you are using [Paddle's Sandbox environment](#paddle-sandbox). The `PADDLE_SANDBOX` variable should be set to `false` if you are deploying your application to production and are using Paddle's live vendor environment.

The `PADDLE_RETAIN_KEY` is optional and should only be set if you're using Paddle with [Retain](https://developer.paddle.com/concepts/retain/overview).

### [Paddle JS](#paddle-js)

Paddle relies on its own JavaScript library to initiate the Paddle checkout widget. You can load the JavaScript library by placing the `@paddleJS` Blade directive right before your application layout's closing `</head>` tag:

```
<head>
    ...

    @paddleJS
</head>
```

### [Currency Configuration](#currency-configuration)

You can specify a locale to be used when formatting money values for display on invoices. Internally, Cashier utilizes [PHP's `NumberFormatter` class](https://www.php.net/manual/en/class.numberformatter.php) to set the currency locale:

```
CASHIER_CURRENCY_LOCALE=nl_BE
```

In order to use locales other than `en`, ensure the `ext-intl` PHP extension is installed and configured on your server.

### [Overriding Default Models](#overriding-default-models)

You are free to extend the models used internally by Cashier by defining your own model and extending the corresponding Cashier model:

```
use Laravel\Paddle\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

After defining your model, you may instruct Cashier to use your custom model via the `Laravel\Paddle\Cashier` class. Typically, you should inform Cashier about your custom models in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
use App\Models\Cashier\Subscription;
use App\Models\Cashier\Transaction;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useTransactionModel(Transaction::class);
}
```

## [Quickstart](#quickstart)

### [Selling Products](#quickstart-selling-products)

Before utilizing Paddle Checkout, you should define Products with fixed prices in your Paddle dashboard. In addition, you should [configure Paddle's webhook handling](#handling-paddle-webhooks).

Offering product and subscription billing via your application can be intimidating. However, thanks to Cashier and [Paddle's Checkout Overlay](https://developer.paddle.com/concepts/sell/overlay-checkout), you can easily build modern, robust payment integrations.

To charge customers for non-recurring, single-charge products, we'll utilize Cashier to charge customers with Paddle's Checkout Overlay, where they will provide their payment details and confirm their purchase:

```
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout('pri_deluxe_album')
        ->returnTo(route('dashboard'));

    return view('buy', ['checkout' => $checkout]);
})->name('checkout');
```

### [Selling Subscriptions](#quickstart-selling-subscriptions)

To learn how to sell subscriptions using Cashier and Paddle's Checkout Overlay, let's consider the simple scenario of a subscription service with a basic monthly (`price_basic_monthly`) and yearly (`price_basic_yearly`) plan:

```
use Illuminate\Http\Request;

Route::get('/subscribe', function (Request $request) {
    $checkout = $request->user()->checkout('price_basic_monthly')
        ->returnTo(route('dashboard'));

    return view('subscribe', ['checkout' => $checkout]);
})->name('subscribe');
```

## [Checkout Sessions](#checkout-sessions)

### [Overlay Checkout](#overlay-checkout)

Before displaying the Checkout Overlay widget, you must generate a checkout session using Cashier:

```
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

### [Inline Checkout](#inline-checkout)

Paddle also provides the option to display the widget inline:

```
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

### [Guest Checkouts](#guest-checkouts)

Sometimes, you may need to create a checkout session for users that do not need an account with your application:

```
use Illuminate\Http\Request;
use Laravel\Paddle\Checkout;

Route::get('/buy', function (Request $request) {
    $checkout = Checkout::guest(['pri_34567'])
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

## [Price Previews](#price-previews)

Paddle allows you to customize prices per currency. Cashier Paddle allows you to retrieve all of these prices using the `previewPrices` method:

```
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456']);
```

## [Customers](#customers)

### [Customer Defaults](#customer-defaults)

Cashier allows you to define some useful defaults for your customers:

```
public function paddleName(): string|null
{
    return $this->name;
}

public function paddleEmail(): string|null
{
    return $this->email;
}
```

### [Retrieving Customers](#retrieving-customers)

You can retrieve a customer by their Paddle Customer ID using the `Cashier::findBillable` method:

```
use Laravel\Paddle\Cashier;

$user = Cashier::findBillable($customerId);
```

### [Creating Customers](#creating-customers)

Occasionally, you may wish to create a Paddle customer without beginning a subscription:

```
$customer = $user->createAsCustomer();
```

## [Subscriptions](#subscriptions)

### [Creating Subscriptions](#creating-subscriptions)

To create a subscription, use the `subscribe` method:

```
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

### [Checking Subscription Status](#checking-subscription-status)

Once a user is subscribed to your application, you may check their subscription status using a variety of convenient methods:

```
if ($user->subscribed()) {
    // ...
}

if ($user->subscription()->onTrial()) {
    // ...
}

if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
    // ...
}

if ($user->subscription()->recurring()) {
    // ...
}
```

### [Cancecing Subscriptions](#canceling-subscriptions)

To cancel a subscription, call the `cancel` method on the user's subscription:

```
$user->subscription('default')->cancel();
```

## [Handling Paddle Webhooks](#handling-paddle-webhooks)

Cashier automatically handles the most common Paddle webhook events. You may define webhook handlers in your application's `AppServiceProvider`:

```
use Illuminate\Support\Facades\Event;
use Laravel\Paddle\Events\TransactionCompleted;

Event::listen(TransactionCompleted::class, function (TransactionCompleted $event) {
    // Handle the event...
});
```

### [Verifying Webhook Signatures](#verifying-webhook-signatures)

Paddle verifies the webhook signature by verifying the signature header sent with the webhook request.

## [Testing](#testing)

When testing Cashier, you may wish to "mock" the HTTP requests to the Paddle API. To do so, you may use the `Laravel\Paddle\Cashier` class to set test API keys:

```
use Laravel\Paddle\Cashier;

Cashier::setTestApiKey('sk_test_...');
Cashier::setTestClientToken('client_token_...');
```
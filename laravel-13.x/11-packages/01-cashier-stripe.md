---
title: Laravel Cashier (Stripe)
description: Provides an expressive, fluent interface to Stripe's subscription billing services.
url: https://laravel.com/docs/13.x/billing
tags: [packages]
---

#Laravel Cashier (Stripe)

-   [Introduction](#introduction)
-   [Upgrading Cashier](#upgrading-cashier)
-   [Installation](#installation)
-   [Configuration](#configuration)
    -   [Billable Model](#billable-model)
    -   [API Keys](#api-keys)
    -   [Currency Configuration](#currency-configuration)
    -   [Tax Configuration](#tax-configuration)
    -   [Logging](#logging)
    -   [Using Custom Models](#using-custom-models)
-   [Quickstart](#quickstart)
    -   [Selling Products](#quickstart-selling-products)
    -   [Selling Subscriptions](#quickstart-selling-subscriptions)
-   [Customers](#customers)
    -   [Retrieving Customers](#retrieving-customers)
    -   [Creating Customers](#creating-customers)
    -   [Updating Customers](#updating-customers)
    -   [Balances](#balances)
    -   [Tax IDs](#tax-ids)
    -   [Syncing Customer Data With Stripe](#syncing-customer-data-with-stripe)
    -   [Billing Portal](#billing-portal)
-   [Payment Methods](#payment-methods)
    -   [Storing Payment Methods](#storing-payment-methods)
    -   [Retrieving Payment Methods](#retrieving-payment-methods)
    -   [Payment Method Presence](#payment-method-presence)
    -   [Updating the Default Payment Method](#updating-the-default-payment-method)
    -   [Adding Payment Methods](#adding-payment-methods)
    -   [Deleting Payment Methods](#deleting-payment-methods)
-   [Subscriptions](#subscriptions)
    -   [Creating Subscriptions](#creating-subscriptions)
    -   [Checking Subscription Status](#checking-subscription-status)
    -   [Changing Prices](#changing-prices)
    -   [Subscription Quantity](#subscription-quantity)
    -   [Subscriptions With Multiple Products](#subscriptions-with-multiple-products)
    -   [Multiple Subscriptions](#multiple-subscriptions)
    -   [Usage Based Billing](#usage-based-billing)
    -   [Subscription Taxes](#subscription-taxes)
    -   [Subscription Anchor Date](#subscription-anchor-date)
    -   [Canceling Subscriptions](#cancelling-subscriptions)
    -   [Resuming Subscriptions](#resuming-subscriptions)
-   [Subscription Trials](#subscription-trials)
    -   [With Payment Method Up Front](#with-payment-method-up-front)
    -   [Without Payment Method Up Front](#without-payment-method-up-front)
    -   [Extending Trials](#extending-trials)
-   [Handling Stripe Webhooks](#handling-stripe-webhooks)
    -   [Defining Webhook Event Handlers](#defining-webhook-event-handlers)
    -   [Verifying Webhook Signatures](#verifying-webhook-signatures)
-   [Single Charges](#single-charges)
    -   [Simple Charge](#simple-charge)
    -   [Charge With Invoice](#charge-with-invoice)
    -   [Creating Payment Intents](#creating-payment-intents)
    -   [Refunding Charges](#refunding-charges)
-   [Invoices](#invoices)
    -   [Retrieving Invoices](#retrieving-invoices)
    -   [Upcoming Invoices](#upcoming-invoices)
    -   [Previewing Subscription Invoices](#previewing-subscription-invoices)
    -   [Generating Invoice PDFs](#generating-invoice-pdfs)
-   [Checkout](#checkout)
    -   [Product Checkouts](#product-checkouts)
    -   [Single Charge Checkouts](#single-charge-checkouts)
    -   [Subscription Checkouts](#subscription-checkouts)
    -   [Collecting Tax IDs](#collecting-tax-ids)
    -   [Guest Checkouts](#guest-checkouts)
-   [Handling Failed Payments](#handling-failed-payments)
    -   [Confirming Payments](#confirming-payments)
-   [Strong Customer Authentication (SCA)](#strong-customer-authentication)
    -   [Payments Requiring Additional Confirmation](#payments-requiring-additional-confirmation)
    -   [Off-session Payment Notifications](#off-session-payment-notifications)
-   [Stripe SDK](#stripe-sdk)
-   [Testing](#testing)

## [Introduction](#introduction)

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) provides an expressive, fluent interface to [Stripe's](https://stripe.com) subscription billing services. It handles almost all of the boilerplate subscription billing code you are dreading writing. In addition to basic subscription management, Cashier can handle coupons, swapping subscription, subscription "quantities", cancellation grace periods, and even generate invoice PDFs.

## [Upgrading Cashier](#upgrading-cashier)

When upgrading to a new version of Cashier, it's important that you carefully review [the upgrade guide](https://github.com/laravel/cashier-stripe/blob/16.x/UPGRADE.md).

To prevent breaking changes, Cashier uses a fixed Stripe API version. Cashier 16 utilizes Stripe API version `2025-06-30.basil`. The Stripe API version will be updated on minor releases in order to make use of new Stripe features and improvements.

## [Installation](#installation)

First, install the Cashier package for Stripe using the Composer package manager:

```
composer require laravel/cashier
composer require laravel/cashier
```

After installing the package, publish Cashier's migrations using the `vendor:publish` Artisan command:

```
php artisan vendor:publish --tag="cashier-migrations"
php artisan vendor:publish --tag="cashier-migrations"
```

Then, migrate your database:

```
php artisan migrate
php artisan migrate
```

Cashier's migrations will add several columns to your `users` table. They will also create a new `subscriptions` table to hold all of your customer's subscriptions and a `subscription_items` table for subscriptions with multiple prices.

If you wish, you can also publish Cashier's configuration file using the `vendor:publish` Artisan command:

```
php artisan vendor:publish --tag="cashier-config"
php artisan vendor:publish --tag="cashier-config"
```

Lastly, to ensure Cashier properly handles all Stripe events, remember to [configure Cashier's webhook handling](#handling-stripe-webhooks).

Stripe recommends that any column used for storing Stripe identifiers should be case-sensitive. Therefore, you should ensure the column collation for the `stripe_id` column is set to `utf8_bin` when using MySQL. More information regarding this can be found in the [Stripe documentation](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible).

## [Configuration](#configuration)

### [Billable Model](#billable-model)

Before using Cashier, add the `Billable` trait to your billable model definition. Typically, this will be the `App\Models\User` model. This trait provides various methods to allow you to perform common billing tasks, such as creating subscriptions, applying coupons, and updating payment method information:

```
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Cashier assumes your billable model will be the `App\Models\User` class that ships with Laravel. If you wish to change this you may specify a different model via the `useCustomerModel` method. This method should typically be called in the `boot` method of your `AppServiceProvider` class:

```
use App\Models\Cashier\User;
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useCustomerModel(User::class);
}
```

If you're using a model other than Laravel's supplied `App\Models\User` model, you'll need to publish and alter the [Cashier migrations](#installation) provided to match your alternative model's table name.

### [API Keys](#api-keys)

Next, you should configure your Stripe API keys in your application's `.env` file. You can retrieve your Stripe API keys from the Stripe control panel:

```
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

You should ensure that the `STRIPE_WEBHOOK_SECRET` environment variable is defined in your application's `.env` file, as this variable is used to ensure that incoming webhooks are actually from Stripe.

### [Currency Configuration](#currency-configuration)

The default Cashier currency is United States Dollars (USD). You can change the default currency by setting the `CASHIER_CURRENCY` environment variable within your application's `.env` file:

```
CASHIER_CURRENCY=eur
```

In addition to configuring Cashier's currency, you may also specify a locale to be used when formatting money values for display on invoices. Internally, Cashier utilizes [PHP's `NumberFormatter` class](https://www.php.net/manual/en/class.numberformatter.php) to set the currency locale:

```
CASHIER_CURRENCY_LOCALE=nl_BE
```

In order to use locales other than `en`, ensure the `ext-intl` PHP extension is installed and configured on your server.

### [Tax Configuration](#tax-configuration)

Thanks to [Stripe Tax](https://stripe.com/tax), it's possible to automatically calculate taxes for all invoices generated by Stripe. You can enable automatic tax calculation by invoking the `calculateTaxes` method in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::calculateTaxes();
}
```

Once tax calculation has been enabled, any new subscriptions and any one-off invoices that are generated will receive automatic tax calculation.

For this feature to work properly, your customer's billing details, such as the customer's name, address, and tax ID, need to be synced to Stripe. You may use the [customer data synchronization](#syncing-customer-data-with-stripe) and [Tax ID](#tax-ids) methods offered by Cashier to accomplish this.

### [Logging](#logging)

Cashier allows you to specify the log channel to be used when logging fatal Stripe errors. You may specify the log channel by defining the `CASHIER_LOGGER` environment variable within your application's `.env` file:

```
CASHIER_LOGGER=stack
```

Exceptions that are generated by API calls to Stripe will be logged through your application's default log channel.

### [Using Custom Models](#using-custom-models)

You are free to extend the models used internally by Cashier by defining your own model and extending the corresponding Cashier model:

```
use Laravel\Cashier\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

After defining your model, you may instruct Cashier to use your custom model via the `Laravel\Cashier\Cashier` class. Typically, you should inform Cashier about your custom models in the `boot` method of your application's `App\Providers\AppServiceProvider` class:

```
use App\Models\Cashier\Subscription;
use App\Models\Cashier\SubscriptionItem;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useSubscriptionItemModel(SubscriptionItem::class);
}
```

## [Quickstart](#quickstart)

### [Selling Products](#quickstart-selling-products)

Before utilizing Stripe Checkout, you should define Products with fixed prices in your Stripe dashboard. In addition, you should [configure Cashier's webhook handling](#handling-stripe-webhooks).

Offering product and subscription billing via your application can be intimidating. However, thanks to Cashier and [Stripe Checkout](https://stripe.com/payments/checkout), you can easily build modern, robust payment integrations.

To charge customers for non-recurring, single-charge products, we'll utilize Cashier to direct customers to Stripe Checkout, where they will provide their payment details and confirm their purchase. Once the payment has been made via Checkout, the customer will be redirected to a success URL of your choosing within your application:

```
use Illuminate\Http\Request;

Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';

    $quantity = 1;

    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
    ]);
})->name('checkout');

Route::view('/checkout/success', 'checkout.success')->name('checkout-success');
Route::view('/checkout/cancel', 'checkout.cancel')->name('checkout-cancel');
```

As you can see in the example above, we will utilize Cashier's provided `checkout` method to redirect the customer to Stripe Checkout for a given "price identifier". When using Stripe, "prices" refer to [defined prices for specific products](https://stripe.com/docs/products-prices/how-products-and-prices-work).

If necessary, the `checkout` method will automatically create a customer in Stripe and connect that Stripe customer record to the corresponding user in your application's database. After completing the checkout session, the customer will be redirected to a dedicated success or cancellation page where you can display an informational message to the customer.

#### [Providing Meta Data to Stripe Checkout](#providing-meta-data-to-stripe-checkout)

When selling products, it's common to keep track of completed orders and purchased products via `Cart` and `Order` models defined by your own application. When redirecting customers to Stripe Checkout to complete a purchase, you may need to provide an existing order identifier so that you can associate the completed purchase with the corresponding order when the customer is redirected back to your application.

To accomplish this, you may provide an array of `metadata` to the `checkout` method. Let's imagine that a pending `Order` is created within our application when a user begins the checkout process. Remember, the `Cart` and `Order` models in this example are illustrative and not provided by Cashier. You are free to implement these concepts based on the needs of your own application:

```
use App\Models\Cart;
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/cart/{cart}/checkout', function (Request $request, Cart $cart) {
    $order = Order::create([
        'cart_id' => $cart->id,
        'price_ids' => $cart->price_ids,
        'status' => 'incomplete',
    ]);

    return $request->user()->checkout($order->price_ids, [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
        'metadata' => ['order_id' => $order->id],
    ]);
})->name('checkout');
```

As you can see in the example above, when a user begins the checkout process, we will provide all of the cart / order's associated Stripe price identifiers to the `checkout` method. Of course, your application is responsible for associating these items with the "shopping cart" or order as a customer adds them. We also provide the order's ID to the Stripe Checkout session via the `metadata` array. Finally, we have added the `CHECKOUT_SESSION_ID` template variable to the Checkout success route. When Stripe redirects customers back to your application, this template variable will automatically be populated with the Checkout session ID.

Next, let's build the Checkout success route. This is the route that users will be redirected to after their purchase has been completed via Stripe Checkout. Within this route, we can retrieve the Stripe Checkout session ID and the associated Stripe Checkout instance in order to access our provided meta data and update our customer's order accordingly:

```
use App\Models\Order;
use Illuminate\Http\Request;
use Laravel\Cashier\Cashier;

Route::get('/checkout/success', function (Request $request) {
    $sessionId = $request->get('session_id');

    if ($sessionId === null) {
        return;
    }

    $session = Cashier::stripe()->checkout->sessions->retrieve($sessionId);

    if ($session->payment_status !== 'paid') {
        return;
    }

    $orderId = $session['metadata']['order_id'] ?? null;

    $order = Order::findOrFail($orderId);

    $order->update(['status' => 'completed']);

    return view('checkout-success', ['order' => $order]);
})->name('checkout-success');
```

Please refer to Stripe's documentation for more information on the [data contained by the Checkout session object](https://stripe.com/docs/api/checkout/sessions/object).

### [Selling Subscriptions](#quickstart-selling-subscriptions)

Before utilizing Stripe Checkout, you should define Products with fixed prices in your Stripe dashboard. In addition, you should [configure Cashier's webhook handling](#handling-stripe-webhooks).

Offering product and subscription billing via your application can be intimidating. However, thanks to Cashier and [Stripe Checkout](https://stripe.com/payments/checkout), you can easily build modern, robust payment integrations.

To learn how to sell subscriptions using Cashier and Stripe Checkout, let's consider the simple scenario of a subscription service with a basic monthly (`price_basic_monthly`) and yearly (`price_basic_yearly`) plan. These two prices could be grouped under a "Basic" product (`pro_basic`) in our Stripe dashboard. In addition, our subscription service might offer an Expert plan as `pro_expert`.

First, let's discover how a customer can subscribe to our services. Of course, you can imagine the customer might click a "subscribe" button for the Basic plan on our application's pricing page. This button or link should direct the user to a Laravel route which creates the Stripe Checkout session for their chosen plan:

```
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_basic_monthly')
        ->trialDays(5)
        ->allowPromotionCodes()
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

As you can see in the example above, we will redirect the customer to a Stripe Checkout session which will allow them to subscribe to our Basic plan. After a successful checkout or cancellation, the customer will be redirected back to the URL we provided to the `checkout` method. To know when their subscription has actually started (since some payment methods require a few seconds to process), we'll also need to [configure Cashier's webhook handling](#handling-stripe-webhooks).

Now that customers can start subscriptions, we need to restrict certain portions of our application so that only subscribed users can access them. Of course, we can always determine a user's current subscription status via the `subscribed` method provided by Cashier's `Billable` trait:

```
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

We can even easily determine if a user is subscribed to specific product or price:

```
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

#### [Building a Subscribed Middleware](#quickstart-building-a-subscribed-middleware)

For convenience, you may wish to create a [[04-the-basics/02-middleware.md|middleware]] which determines if the incoming request is from a subscribed user. Once this middleware has been defined, you may easily assign it to a route to prevent users that are not subscribed from accessing the route:

```
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            // Redirect user to billing page and ask them to subscribe...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

Once the middleware has been defined, you may assign it to a route:

```
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

#### [Allowing Customers to Manage Their Billing Plan](#quickstart-allowing-customers-to-manage-their-billing-plan)

Of course, customers may want to change their subscription plan to another product or "tier". The easiest way to allow this is by directing customers to Stripe's [Customer Billing Portal](https://stripe.com/docs/no-code/customer-portal), which provides a hosted user interface that allows customers to download invoices, update their payment method, and change subscription plans.

First, define a link or button within your application that directs users to a Laravel route which we will utilize to initiate a Billing Portal session:

```
<a href="{{ route('billing') }}">
    Billing
</a>
```

Next, let's define the route that initiates a Stripe Customer Billing Portal session and redirects the user to the Portal. The `redirectToBillingPortal` method accepts the URL that users should be returned to when exiting the Portal:

```
use Illuminate\Http\Request;

Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
})->middleware(['auth'])->name('billing');
```

As long as you have configured Cashier's webhook handling, Cashier will automatically keep your application's Cashier-related database tables in sync by inspecting the incoming webhooks from Stripe. So, for example, when a user cancels their subscription via Stripe's Customer Billing Portal, Cashier will receive the corresponding webhook and mark the subscription as "canceled" in your application's database.

## [Customers](#customers)

### [Retrieving Customers](#retrieving-customers)

You can retrieve a customer by their Stripe ID using the `Cashier::findBillable` method. This method will return an instance of the billable model:

```
use Laravel\Cashier\Cashier;

$user = Cashier::findBillable($stripeId);
```

### [Creating Customers](#creating-customers)

Occasionally, you may wish to create a Stripe customer without beginning a subscription. You may accomplish this using the `createAsStripeCustomer` method:

```
$stripeCustomer = $user->createAsStripeCustomer();
```

Once the customer has been created in Stripe, you may begin a subscription at a later date. You may provide an optional `$options` array to pass in any additional [customer creation parameters that are supported by the Stripe API](https://stripe.com/docs/api/customers/create):

```
$stripeCustomer = $user->createAsStripeCustomer($options);
```

You may use the `asStripeCustomer` method if you want to return the Stripe customer object for a billable model:

```
$stripeCustomer = $user->asStripeCustomer();
```

The `createOrGetStripeCustomer` method may be used if you would like to retrieve the Stripe customer object for a given billable model but are not sure whether the billable model is already a customer within Stripe. This method will create a new customer in Stripe if one does not already exist:

```
$stripeCustomer = $user->createOrGetStripeCustomer();
```

### [Updating Customers](#updating-customers)

Occasionally, you may wish to update the Stripe customer directly with additional information. You may accomplish this using the `updateStripeCustomer` method. This method accepts an array of [customer update options supported by the Stripe API](https://stripe.com/docs/api/customers/update):

```
$stripeCustomer = $user->updateStripeCustomer($options);
```

### [Balances](#balances)

Stripe allows you to credit or debit a customer's "balance". Later, this balance will be credited or debited on new invoices. To check the customer's total balance you may use the `balance` method that is available on your billable model. The `balance` method will return a formatted string representation of the balance in the customer's currency:

```
$balance = $user->balance();
```

To credit a customer's balance, you may provide a value to the `creditBalance` method. If you wish, you may also provide a description:

```
$user->creditBalance(500, 'Premium customer top-up.');
```

Providing a value to the `debitBalance` method will debit the customer's balance:

```
$user->debitBalance(300, 'Bad usage penalty.');
```

The `applyBalance` method will create new customer balance transactions for the customer. You may retrieve these transaction records using the `balanceTransactions` method, which may be useful in order to provide a log of credits and debits for the customer to review:

```
// Retrieve all transactions...
$transactions = $user->balanceTransactions();

foreach ($transactions as $transaction) {
    // Transaction amount...
    $amount = $transaction->amount(); // $2.31

    // Retrieve the related invoice when available...
    $invoice = $transaction->invoice();
}
```

### [Tax IDs](#tax-ids)

Cashier offers an easy way to manage a customer's tax IDs. For example, the `taxIds` method may be used to retrieve all of the [tax IDs](https://stripe.com/docs/api/customer_tax_ids/object) that are assigned to a customer as a collection:

```
$taxIds = $user->taxIds();
```

You can also retrieve a specific tax ID for a customer by its identifier:

```
$taxId = $user->findTaxId('txi_belgium');
```

You may create a new Tax ID by providing a valid [type](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type) and value to the `createTaxId` method:

```
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
```

The `createTaxId` method will immediately add the VAT ID to the customer's account. [Verification of VAT IDs is also done by Stripe](https://stripe.com/docs/invoicing/customer/tax-ids#validation); however, this is an asynchronous process. You can be notified of verification updates by subscribing to the `customer.tax_id.updated` webhook event and inspecting [the VAT IDs `verification` parameter](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification). For more information on handling webhooks, please consult the [documentation on defining webhook handlers](#handling-stripe-webhooks).

You may delete a tax ID using the `deleteTaxId` method:

```
$user->deleteTaxId('txi_belgium');
```

### [Syncing Customer Data With Stripe](#syncing-customer-data-with-stripe)

Typically, when your application's users update their name, email address, or other information that is also stored by Stripe, you should inform Stripe of the updates. By doing so, Stripe's copy of the information will be in sync with your application's.

To automate this, you may define an event listener on your billable model that reacts to the model's `updated` event. Then, within your event listener, you may invoke the `syncStripeCustomerDetails` method on the model:

```
use App\Models\User;
use function Illuminate\Events\queueable;

/**
 * The "booted" method of the model.
 */
protected static function booted(): void
{
    static::updated(queueable(function (User $customer) {
        if ($customer->hasStripeId()) {
            $customer->syncStripeCustomerDetails();
        }
    }));
}
```

Now, every time your customer model is updated, its information will be synced with Stripe. For convenience, Cashier will automatically sync your customer's information with Stripe on the initial creation of the customer.

You may customize the columns used for syncing customer information to Stripe by overriding a variety of methods provided by Cashier. For example, you may override the `stripeName` method to customize the attribute that should be considered the customer's "name" when Cashier syncs customer information to Stripe:

```
/**
 * Get the customer name that should be synced to Stripe.
 */
public function stripeName(): string|null
{
    return $this->company_name;
}
```

Similarly, you may override the `stripeEmail`, `stripePhone` (20 character maximum), `stripeAddress`, and `stripePreferredLocales` methods. These methods will sync information to their corresponding customer parameters when [updating the Stripe customer object](https://stripe.com/docs/api/customers/update). If you wish to take total control over the customer information sync process, you may override the `syncStripeCustomerDetails` method.

### [Billing Portal](#billing-portal)

Stripe offers [an easy way to set up a billing portal](https://stripe.com/docs/billing/subscriptions/customer-portal) so that your customer can manage their subscription, payment methods, and view their billing history. You can redirect your users to the billing portal by invoking the `redirectToBillingPortal` method on the billable model from a controller or route:

```
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal();
});
```

By default, when the user is finished managing their subscription, they will be able to return to the `home` route of your application via a link within the Stripe billing portal. You may provide a custom URL that the user should return to by passing the URL as an argument to the `redirectToBillingPortal` method:

```
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('billing'));
});
```

If you would like to generate the URL to the billing portal without generating an HTTP redirect response, you may invoke the `billingPortalUrl` method:

```
$url = $request->user()->billingPortalUrl(route('billing'));
```

## [Payment Methods](#payment-methods)

### [Storing Payment Methods](#storing-payment-methods)

In order to create subscriptions or perform "one-off" charges with Stripe, you will need to store a payment method and retrieve its identifier from Stripe. The approach used to accomplish this differs based on whether you plan to use the payment method for subscriptions or single charges, so we will examine both below.

#### [Payment Methods for Subscriptions](#payment-methods-for-subscriptions)

When storing a customer's credit card information for future use by a subscription, the Stripe "Setup Intents" API must be used to securely gather the customer's payment method details. A "Setup Intent" indicates to Stripe the intention to charge a customer's payment method. Cashier's `Billable` trait includes the `createSetupIntent` method to easily create a new Setup Intent. You should invoke this method from the route or controller that will render the form which gathers your customer's payment method details:

```
return view('update-payment-method', [
    'intent' => $user->createSetupIntent()
]);
```

After you have created the Setup Intent and passed it to the view, you should attach its secret to the element that will gather the payment method. For example, consider this "update payment method" form:

```
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

Next, the Stripe.js library may be used to attach a [Stripe Element](https://stripe.com/docs/stripe-js) to the form and securely gather the customer's payment details:

```
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Next, the card can be verified and a secure "payment method identifier" can be retrieved from Stripe using [Stripe's `confirmCardSetup` method](https://stripe.com/docs/js/setup_intents/confirm_card_setup):

```
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

After the card has been verified by Stripe, you may pass the resulting `setupIntent.payment_method` identifier to your Laravel application, where it can be attached to the customer. The payment method can either be [added as a new payment method](#adding-payment-methods) or [used to update the default payment method](#updating-the-default-payment-method). You can also immediately use the payment method identifier to [create a new subscription](#creating-subscriptions).

If you would like more information about Setup Intents and gathering customer payment details please [review this overview provided by Stripe](https://stripe.com/docs/payments/save-and-reuse#php).

#### [Payment Methods for Single Charges](#payment-methods-for-single-charges)

Of course, when making a single charge against a customer's payment method, we will only need to use a payment method identifier once. Due to Stripe limitations, you may not use the stored default payment method of a customer for single charges. You must allow the customer to enter their payment method details using the Stripe.js library. For example, consider the following form:

```
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

After defining such a form, the Stripe.js library may be used to attach a [Stripe Element](https://stripe.com/docs/stripe-js) to the form and securely gather the customer's payment details:

```
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

Next, the card can be verified and a secure "payment method identifier" can be retrieved from Stripe using [Stripe's `createPaymentMethod` method](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method):

```
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

If the card is verified successfully, you may pass the `paymentMethod.id` to your Laravel application and process a [single charge](#simple-charge).

### [Retrieving Payment Methods](#retrieving-payment-methods)

The `paymentMethods` method on the billable model instance returns a collection of `Laravel\Cashier\PaymentMethod` instances:

```
$paymentMethods = $user->paymentMethods();
```

By default, this method will return payment methods of every type. To retrieve payment methods of a specific type, you may pass the `type` as an argument to the method:

```
$paymentMethods = $user->paymentMethods('sepa_debit');
```

To retrieve the customer's default payment method, the `defaultPaymentMethod` method may be used:

```
$paymentMethod = $user->defaultPaymentMethod();
```

You can retrieve a specific payment method that is attached to the billable model using the `findPaymentMethod` method:

```
$paymentMethod = $user->findPaymentMethod($paymentMethodId);
```

### [Payment Method Presence](#payment-method-presence)

To determine if a billable model has a default payment method attached to their account, invoke the `hasDefaultPaymentMethod` method:

```
if ($user->hasDefaultPaymentMethod()) {
    // ...
}
```

You may use the `hasPaymentMethod` method to determine if a billable model has at least one payment method attached to their account:

```
if ($user->hasPaymentMethod()) {
    // ...
}
```

This method will determine if the billable model has any payment method at all. To determine if a payment method of a specific type exists for the model, you may pass the `type` as an argument to the method:

```
if ($user->hasPaymentMethod('sepa_debit')) {
    // ...
}
```

### [Updating the Default Payment Method](#updating-the-default-payment-method)

The `updateDefaultPaymentMethod` method may be used to update a customer's default payment method information. This method accepts a Stripe payment method identifier and will assign the new payment method as the default billing payment method:

```
$user->updateDefaultPaymentMethod($paymentMethod);
```

To sync your default payment method information with the customer's default payment method information in Stripe, you may use the `updateDefaultPaymentMethodFromStripe` method:

```
$user->updateDefaultPaymentMethodFromStripe();
```

### [Adding Payment Methods](#adding-payment-methods)

The `addPaymentMethod` method may be used to add a new payment method to the billable model. This method accepts a Stripe payment method identifier and will add it to the customer's account:

```
$user->addPaymentMethod($paymentMethod);
```

### [Deleting Payment Methods](#deleting-payment-methods)

The `deletePaymentMethod` method may be used to delete a payment method from the billable model:

```
$user->deletePaymentMethod($paymentMethod);
```

## [Subscriptions](#subscriptions)

### [Creating Subscriptions](#creating-subscriptions)

To create a subscription, first retrieve an instance of your billable model from your database, which will typically be an instance of `App\Models\User`. Then, you may use the `newSubscription` method to create a subscription. This method accepts the name of the subscription as the first argument and the Stripe price / plan identifier as the second argument:

```
$user = User::find(1);

$user->newSubscription('default', 'price_basic_monthly')->create($paymentMethod);
```

### [Checking Subscription Status](#checking-subscription-status)

To determine if a user has an active subscription, you may use the `subscribed` method on your billable model:

```
if ($user->subscribed()) {
    // ...
}
```

The `subscribed` method also makes a great candidate for a [[04-the-basics/02-middleware.md|route middleware]], allowing you to filter routes based on a user's subscription status:

```
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsSubscribed
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->subscribed()) {
            // This user is not a paying customer...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

If you would like to determine if a user is still within their trial period, you may use the `onTrial` method:

```
if ($user->subscription()->onTrial()) {
    // ...
}
```

The `subscribedToPrice` method may be used to determine if the user is subscribed to a given plan based on a given Stripe price ID:

```
if ($user->subscribedToPrice('price_basic_monthly')) {
    // ...
}
```

The `subscribedToProduct` method may be used to determine if the user is subscribed to a given product based on a given Stripe product ID:

```
if ($user->subscribedToProduct('prod_basic')) {
    // ...
}
```

The `recurring` method may be used to determine if the user is currently on an active subscription and is no longer within their trial period:

```
if ($user->subscription()->recurring()) {
    // ...
}
```

#### [Canceled Subscription Status](#canceled-subscription-status)

To determine if the user was once an active subscriber but has canceled their subscription, you may use the `canceled` method:

```
if ($user->subscription()->canceled()) {
    // ...
}
```

You may also determine if a user has canceled their subscription, but are still on their "grace period" until the subscription fully expires:

```
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

#### [Past Due Status](#past-due-status)

If a payment fails for a subscription, it will be marked as `past_due`. When your subscription is in this state it will not be active until the customer has updated their payment information. You may determine if a subscription is past due using the `pastDue` method on the subscription instance:

```
if ($user->subscription()->pastDue()) {
    // ...
}
```

### [Changing Prices](#changing-prices)

Sometimes you may wish to change the price or plan that is associated with a subscription. To swap a subscription to a new price, use the `swap` method on the subscription:

```
$user->subscription('default')->swap('price_basic_yearly');
```

If you would like to force a specific invoice date when swapping plans, you may use the `swapAndInvoice` method:

```
$user->subscription('default')->swapAndInvoice('price_basic_yearly');
```

### [Subscription Quantity](#subscription-quantity)

Cashier also supports subscription "quantities". You may increment or decrement a subscription's quantity using the `incrementQuantity` and `decrementQuantity` methods:

```
$user->subscription('default')->incrementQuantity(5);

$user->subscription('default')->decrementQuantity(3);
```

Or, you may set a specific quantity using the `updateQuantity` method:

```
$user->subscription('default')->updateQuantity(10);
```

### [Subscriptions With Multiple Products](#subscriptions-with-multiple-products)

Sometimes your application may need to have a subscription that includes multiple products. For example, imagine a user has a subscription for $100 per month but they also want to add an additional product for $50 per month.

You can provide an array of prices when creating a subscription using the `newSubscription` method:

```
$user->newSubscription('default', ['price_basic_monthly', 'price_seat_1'])->create($paymentMethod);
```

When retrieving a subscription, you may access the prices that are subscribed to using the `items` method:

```
$items = $user->subscription('default')->items;
```

To change the prices that are subscribed to, you may use the `swap` method and pass an array containing the new price IDs:

```
$user->subscription('default')->swap(['price_basic_yearly', 'price_seat_2']);
```

### [Multiple Subscriptions](#multiple-subscriptions)

By default, each billable model can only have one subscription. However, you can allow for multiple subscriptions by defining a "type" when creating a subscription:

```
$user->newSubscription('default', 'price_basic_monthly')->create($paymentMethod);
$user->newSubscription('secondary', 'price_secondary_monthly')->create($paymentMethod);
```

In this example, we're creating two subscriptions. Each subscription will be considered a separate subscription type. To retrieve each subscription, use the `subscription` method:

```
$user->subscription('default');
$user->subscription('secondary');
```

Alternatively, you may retrieve all subscriptions using the `subscriptions` method:

```
$subscriptions = $user->subscriptions;
```

### [Usage Based Billing](#usage-based-billing)

For some products, prices are usage-based. Meaning, the customer's subscription can have a base price, but you need to track additional usage and charge the customer based on that usage. For example, your application might charge based on the number of "API Requests" sent.

To implement usage-based billing, you first need to configure a usage-based price in your Stripe dashboard. After creating the price, you should add it to the subscription using the `metered` method when creating the subscription:

```
$user->newSubscription('default', 'price_basic_monthly')
    ->metered('price_metered_requests')
    ->create($paymentMethod);
```

Next, after the customer has consumed some resources, you can report the usage to Stripe:

```
$user->subscription('default')->reportUsage(['record' => 100]);
```

If you didn't assign a usage-based price at subscription creation, you may add it to an existing subscription using the `addMeteredPrice` method:

```
$user->subscription('default')->addMeteredPrice('price_metered_requests');
```

To retrieve the usage summary, you may use the `usageSummaries` method:

```
$usageSummaries = $user->subscription('default')->usageSummaries();
```

### [Subscription Taxes](#subscription-taxes)

To set the tax percentage rate for a subscription, you should invoke the `taxPercentage` method when defining the subscription:

```
$user->newSubscription('default', 'price_basic_monthly')
    ->taxPercentage(20)
    ->create($paymentMethod);
```

You may allow Cashier to automatically calculate tax based on the customer's address information by invoking the `calculateTaxes` method:

```
$user->newSubscription('default', 'price_basic_monthly')
    ->calculateTaxes()
    ->create($paymentMethod);
```

### [Subscription Anchor Date](#subscription-anchor-date)

By default, the billing cycle anchor is the date the subscription was created or, when a trial is used, the date the trial ends. If you would like to manipulate the billing cycle anchor, you may use the `anchor` method:

```
$user->newSubscription('default', 'price_basic_monthly')
    ->anchor(15)
    ->create($paymentMethod);
```

This method sets the day of the month on which subscription invoices will be billed.

### [Canceling Subscriptions](#cancelling-subscriptions)

To cancel a subscription, call the `cancel` method on the user's subscription:

```
$user->subscription('default')->cancel();
```

You may also pass `true` to immediately cancel the subscription rather than at the end of the billing cycle:

```
$user->subscription('default')->cancel(true);
```

### [Resuming Subscriptions](#resuming-subscriptions)

If a user has canceled their subscription and you wish to resume it, use the `resume` method:

```
$user->subscription('default')->resume();
```

When resuming a subscription that was within a trial period, the trial will resume. When resuming a subscription that was canceled, the subscription will resume where it left off, and the customer will be billed immediately.

## [Subscription Trials](#subscription-trials)

### [With Payment Method Up Front](#with-payment-method-up-front)

If you would like to offer a trial period that requires a payment method up front, you should use the `trialDays` method when creating the subscription:

```
$user->newSubscription('default', 'price_basic_monthly')
    ->trialDays(10)
    ->create($paymentMethod);
```

### [Without Payment Method Up Front](#without-payment-method-up-front)

If you would like to offer a trial period without requiring the user to enter their payment method information up front, you may set the trial days to `0` and simply not provide a payment method when creating the subscription:

```
$user->newSubscription('default', 'price_basic_monthly')
    ->trialDays(10)
    ->create();
```

Or, you may use the ` trial` method to immediately create the subscription in trial mode:

```
$user->newSubscription('default', 'price_basic_monthly')
    ->trial()
    ->create();
```

### [Extending Trials](#extending-trials)

The `extendTrial` method may be used to extend an existing trial period:

```
$user->subscription('default')->extendTrial(now()->addDays(15));
```

## [Handling Stripe Webhooks](#handling-stripe-webhooks)

Cashier automatically handles the most common Stripe webhook events. However, if you have additional webhook events you would like to handle, you may use the `webhook` method provided by Cashier.

Cashier automatically handles the webhook signature verification to ensure the incoming webhook request is actually from Stripe.

To get started, create a route to handle webhooks. This route should point to a controller that extends `Laravel\Cashier\Http\Controllers\WebhookController`:

```
use Laravel\Cashier\Http\Controllers\WebhookController;

Route::post('/stripe/webhook', [WebhookController::class, 'handleWebhook']);
```

### [Defining Webhook Event Handlers](#defining-webhook-event-handlers)

By default, the `WebhookController` contains a mapping of Stripe webhook events to their corresponding handler methods. To define your own webhook event handlers, you may override the `handlers` method within your own controller which extends `Laravel\Cashier\Http\Controllers\WebhookController`:

```
protected function handlers(): array
{
    return array_merge(parent::handlers(), [
        'customer.subscription.created' => WebhookHandler::class,
    ]);
}
```

### [Verifying Webhook Signatures](#verifying-webhook-signatures)

Stripe verifies the webhook signature by verifying the signature header sent with the webhook request. Cashier automatically handles this verification using the `STRIPE_WEBHOOK_SECRET` in your `.env` file.

If you would like to disable webhook signature verification, you may override the `usesStripeWebhook()` method within your own controller to return `false`:

```
protected function usesStripeWebhook(): bool
{
    return false;
}
```

## [Single Charges](#single-charges)

### [Simple Charge](#simple-charge)

If you would like to make a "one-off" charge against a customer, you may use the `charge` method on a billable model:

```
$user->charge(100, 'price_basic_monthly');
```

The `charge` method accepts the amount in cents (or the lowest currency unit). Optionally, you may pass a second argument to provide a description for the charge:

```
$user->charge(100, 'price_basic_monthly', ['description' => 'One-time payment']);
```

### [Charge With Invoice](#charge-with-invoice)

Sometimes you may need to make a one-time charge but also generate an invoice for the customer so that they have a printable PDF receipt for the purchase. The `invoiceFor` method lets you do just that:

```
$user->invoiceFor('One-time fee', 500);
```

Alternatively, you can provide an array of options in addition to the amount:

```
$user->invoiceFor('One-time fee', 500, ['description' => 'One-time payment']);
```

### [Creating Payment Intents](#creating-payment-intents)

To create a Stripe payment intent, you may use the `createSetupIntent` method or create a payment intent directly using the Cashier Stripe SDK:

```
use Laravel\Cashier\Cashier;

$paymentIntent = Cashier::stripe()->paymentIntents->create([
    'amount' => 1000,
    'currency' => 'usd',
    'customer' => $user->stripe_customer_id,
]);
```

### [Refunding Charges](#refunding-charges)

To refund a Stripe charge, you may use the `refund` method on a billable model:

```
$user->refund($chargeId);
```

## [Invoices](#invoices)

### [Retrieving Invoices](#retrieving-invoices)

You may easily retrieve all of a customer's invoices using the `invoices` method:

```
$invoices = $user->invoices();
```

The `paidInvoices` method may be used to retrieve all of a customer's paid invoices:

```
$paidInvoices = $user->paidInvoices();
```

To retrieve the customer's open invoices, you may use the `openInvoices` method:

```
$openInvoices = $user->openInvoices();
```

### [Upcoming Invoices](#upcoming-invoices)

To retrieve an upcoming invoice, you may use the `upcomingInvoice` method:

```
$invoice = $user->upcomingInvoice();
```

### [Previewing Subscription Invoices](#previewing-subscription-invoices)

Using Cashier, you may also preview subscription invoices before making changes using the `previewInvoice` method:

```
$invoice = $user->subscription('default')->previewInvoice('price_basic_yearly');
```

### [Generating Invoice PDFs](#generating-invoice-pdfs)

To generate a PDF for an invoice, you may use the `pdf` method on the invoice:

```
$invoice->pdf();
```

## [Checkout](#checkout)

### [Product Checkouts](#product-checkouts)

Cashier provides built-in support for Stripe Checkout. To redirect customers to Stripe Checkout to purchase products, you may use the `checkout` method:

```
$user->checkout(['price_basic_monthly' => 1], [
    'success_url' => route('checkout-success'),
    'cancel_url' => route('checkout-cancel'),
]);
```

### [Single Charge Checkouts](#single-charge-checkouts)

For single charge checkouts, you can redirect customers to purchase specific products that have already been configured in Stripe:

```
$user->checkout('price_basic_monthly', [
    'success_url' => route('checkout-success'),
    'cancel_url' => route('checkout-cancel'),
]);
```

### [Subscription Checkouts](#subscription-checkouts)

You may also create subscription checkouts directly:

```
$user->checkout('price_basic_monthly', [
    'success_url' => route('checkout-success'),
    'cancel_url' => route('checkout-cancel'),
]);
```

However, this approach may not be as flexible as using the `newSubscription` method. We recommend using the `newSubscription` method for subscription checkouts.

### [Collecting Tax IDs](#collecting-tax-ids)

To collect a customer's tax ID during checkout, you may use the `collectTaxIds` method:

```
$user->checkout('price_basic_monthly', [
    'success_url' => route('checkout-success'),
    'cancel_url' => route('checkout-cancel'),
])->collectTaxIds();
```

### [Guest Checkouts](#guest-checkouts)

If you need to create a checkout session for a guest (unauthenticated user), you may use the `Checkout` facade:

```
use Laravel\Cashier\Checkout;

Checkout::guest()->create('price_basic_monthly', [
    'success_url' => route('checkout-success'),
    'cancel_url' => route('checkout-cancel'),
]);
```

## [Handling Failed Payments](#handling-failed-payments)

Occasionally, a customer's payment may fail when scheduled to renew. When this happens, Cashier will throw a `PaymentFailure` exception. You can catch this exception and redirect the user back to a page where they can update their payment information:

```
try {
    // Process the subscription...
} catch (\Laravel\Cashier\Exceptions\PaymentFailure $e) {
    return redirect('/billing')->with('error', $e->getMessage());
}
```

### [Confirming Payments](#confirming-payments)

Some payments may require additional confirmation or action from the customer. Stripe will call your webhook with a `payment_intent.succeeded` event when additional confirmation is required. In this case, you should display a message to the customer instructing them to take additional action.

## [Strong Customer Authentication (SCA)](#strong-customer-authentication)

Cashier is aware of and ready for Strong Customer Authentication (SCA) regulations in the EU. These regulations require customers to add additional verification to payments. Cashier will automatically build this into your checkout experience.

### [Payments Requiring Additional Confirmation](#payments-requiring-additional-confirmation)

If a payment requires additional confirmation, Cashier will throw a `PaymentConfirmationRequired` exception. Of course, you can catch this exception and redirect the customer to a dedicated payment confirmation page:

```
try {
    $user->charge(100);
} catch (\Laravel\Cashier\Exceptions\PaymentConfirmationRequired $e) {
    return redirect('/payment-confirmation');
}
```

Within your payment confirmation page, you can check if there is a pending payment intent that requires confirmation:

```
$pendingPaymentIntent = $user->pendingPaymentIntent();
```

### [Off-session Payment Notifications](#off-session-payment-notifications)

Because of the SCA regulations, you may need to notify the customer when an off-session payment attempt is made so they can verify the payment. You can send an email notification using the built-in `notifyoffSessionPaymentIntent` method:

```
$user->notifyOffSessionPaymentIntent($paymentIntent);
```

## [Stripe SDK](#stripe-sdk)

Cashier provides access to all of the underlying Stripe SDK through the `Cashier::stripe()` method:

```
$stripeCustomer = Cashier::stripe()->customers->retrieve($stripeId);
```

## [Testing](#testing)

When testing Cashier, you may wish to "mock" the HTTP requests to the Stripe API. To do so, you may use the `Laravel\Cashier\Cashier` class to set a test API key:

```
use Laravel\Cashier\Cashier;

Cashier::setTestApiKey('sk_test_...');
```
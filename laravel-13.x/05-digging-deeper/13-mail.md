---
title: Mail
description: Laravel Mail documentation - clean email API powered by Symfony Mailer
url: https://laravel.com/docs/13.x/mail
tags: [logic]
cssclasses:
  - logic
  - color-orange
color: orange
---

# Mail

- [Introduction](#introduction)
- [Generating Mailables](#generating-mailables)
- [Writing Mailables](#writing-mailables)
- [Sending Mail](#sending-mail)
- [Markdown Mailables](#markdown-mailables)
- [Testing](#testing)

## Introduction

Sending email doesn't have to be complicated. Laravel provides a clean, simple email API powered by the popular Symfony Mailer component. Laravel and Symfony Mailer provide drivers for sending email via SMTP, Cloudflare, Mailgun, Postmark, Resend, Amazon SES, and `sendmail`.

## Configuration

Laravel's email services may be configured via your application's `config/mail.php` configuration file. Each mailer configured within this file may have its own unique configuration and even its own unique "transport".

### Driver Prerequisites

#### Cloudflare Driver

To use the Cloudflare driver:

```bash
composer require symfony/http-client
```

Then configure your `config/mail.php`:

```php
'cloudflare' => [
    'transport' => 'cloudflare',
],
```

And add to `config/services.php`:

```php
'cloudflare' => [
    'account_id' => env('CLOUDFLARE_ACCOUNT_ID'),
    'key' => env('CLOUDFLARE_KEY'),
],
```

#### Mailgun Driver

To use the Mailgun driver:

```bash
composer require symfony/mailgun-mailer symfony/http-client
```

#### Postmark Driver

To use the Postmark driver:

```bash
composer require symfony/postmark-mailer symfony/http-client
```

#### Resend Driver

To use the Resend driver:

```bash
composer require resend/resend-php
```

#### SES Driver

To use the Amazon SES driver:

```bash
composer require aws/aws-sdk-php
```

### Failover Configuration

Sometimes, an external service you have configured to send your application's mail may be down. In these cases, it can be useful to define one or more backup mail delivery configurations:

```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

### Round Robin Configuration

The `roundrobin` transport allows you to distribute your mailing workload across multiple mailers:

```php
'mailers' => [
    'roundrobin' => [
        'transport' => 'roundrobin',
        'mailers' => [
            'ses',
            'postmark',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

## Generating Mailables

When building Laravel applications, each type of email sent by your application is represented as a "mailable" class. These classes are stored in the `app/Mail` directory:

```bash
php artisan make:mail OrderShipped
```

## Writing Mailables

Once you have generated a mailable class, configuration is done via several methods including `envelope()`, `content()`, and `attachments()`.

### Configuring the Sender

#### Using the Envelope

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * Get the message envelope.
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('[email protected]', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```

If you would like, you may also specify a `replyTo` address:

```php
return new Envelope(
    from: new Address('[email protected]', 'Jeffrey Way'),
    replyTo: [
        new Address('[email protected]', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);
```

#### Using a Global `from` Address

You may specify a global "from" address in your `config/mail.php` configuration file:

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', '[email protected]'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

### Configuring the View

Within a mailable class's `content` method, you may define the view:

```php
/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```

#### Plain Text Emails

If you would like to define a plain-text version of your email:

```php
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

### View Data

#### Via Public Properties

Any public property defined on your mailable class will automatically be made available to the view:

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * Create a new message instance.
     */
    public function __construct(
        public Order $order,
    ) {}

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```

#### Via the `with` Parameter

If you would like to customize the format of your email's data before it is sent to the template:

```php
return new Content(
    view: 'mail.orders.shipped',
    with: [
        'orderName' => $this->order->name,
        'orderPrice' => $this->order->price,
    ],
);
```

### Attachments

To add attachments to an email:

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file'),
    ];
}
```

When attaching files to a message, you may also specify the display name and/or MIME type:

```php
return [
    Attachment::fromPath('/path/to/file')
        ->as('name.pdf')
        ->withMime('application/pdf'),
];
```

#### Attaching Files From Disk

If you have stored a file on one of your filesystem disks:

```php
return [
    Attachment::fromStorage('/path/to/file'),
];
```

#### Raw Data Attachments

The `fromData` attachment method may be used to attach a raw string of bytes:

```php
return [
    Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
        ->withMime('application/pdf'),
];
```

### Inline Attachments

Embedding inline images into your emails:

```html
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

If you already have a raw image data string:

```html
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

### Attachable Objects

You may implement the `Illuminate\Contracts\Mail\Attachable` interface on models:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Mail\Attachable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Mail\Attachment;

class Photo extends Model implements Attachable
{
    /**
     * Get the attachable representation of the model.
     */
    public function toMailAttachment(): Attachment
    {
        return Attachment::fromPath('/path/to/file');
    }
}
```

Then use the model in attachments:

```php
public function attachments(): array
{
    return [$this->photo];
}
```

### Headers

Sometimes you may need to attach additional headers:

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * Get the message headers.
 */
public function headers(): Headers
{
    return new Headers(
        messageId: '[email protected]',
        references: ['[email protected]'],
        text: [
            'X-Custom-Header' => 'Custom Value',
        ],
    );
}
```

### Tags and Metadata

Some third-party email providers support message "tags" and "metadata":

```php
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

## Sending Mail

To send a message, use the `to` method on the `Mail` facade:

```php
<?php

namespace App\Http\Controllers;

use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Ship the order...

        Mail::to($request->user())->send(new OrderShipped($order));

        return redirect('/orders');
    }
}
```

You are not limited to just specifying the "to" recipients. You may set "to", "cc", and "bcc":

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

### Sending Mail via a Specific Mailer

By default, Laravel will send email using the default mailer, but you may use the `mailer` method:

```php
Mail::mailer('postmark')
    ->to($request->user())
    ->send(new OrderShipped($order));
```

### Queueing Mail

If you would like to queue the mail message instead of sending it synchronously, use the `queue` method:

```php
Mail::to($request->user())
    ->queue(new OrderShipped($order));
```

## Markdown Mailables

Markdown mailable messages allow you to take advantage of the pre-built templates.

### Generating Markdown Mailables

```bash
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

### Writing Markdown Messages

Markdown mailables use a combination of Blade components and Markdown syntax:

```html
<x-mail::message>
# Order Shipped

Your order has been shipped!

<x-mail::button :url="$url">
View Order
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

#### Button Component

```html
<x-mail::button :url="$url" color="success">
View Order
</x-mail::button>
```

#### Panel Component

```html
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

#### Table Component

```html
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

### Customizing the Components

You may export all of the Markdown mail components to your own application:

```bash
php artisan vendor:publish --tag=laravel-mail
```

This command will publish the Markdown mail components to the `resources/views/vendor/mail` directory.

## Testing

### Testing Mailable Content

You may use the ` Mail::fake()` method to fake mail sending:

```php
use Illuminate\Support\Facades\Mail;

Mail::fake();

$response = $this->post('/order', [
    'name' => 'Taylor',
]);

Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
    return $mail->order->id === $order->id;
});

Mail::assertNotSent(OrderShipped::class);
```

### Testing Mailable Sending

You may test that a mailable was sent to a particular recipient:

```php
Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasFrom('[email protected]') &&
           $mail->hasSubject('Order Shipped');
});
```

## Mail and Local Development

When developing locally, you may not want to actually send emails. There are several options:

### Log Driver

Set `MAIL_MAILER=log` in your `.env` file to write emails to your log files instead of actually sending them.

### Mailtrap

Use [Mailtrap](https://mailtrap.io) with the SMTP driver:

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your_username
MAIL_PASSWORD=your_password
MAIL_ENCRYPTION=tls
```

## Events

Laravel dispatches events during the mail sending process:
- `Illuminate\Mail\Events\MessageSending` - Before a message is sent
- `Illuminate\Mail\Events\MessageSent` - After a message has been sent
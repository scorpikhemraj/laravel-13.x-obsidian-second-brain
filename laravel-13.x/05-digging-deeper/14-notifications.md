---
title: Notifications
description: Laravel Notifications documentation - send notifications across various delivery channels
url: https://laravel.com/docs/13.x/notifications
tags: [logic]
cssclasses:
  - logic
  - color-orange
color: orange
---

# Notifications

- [Introduction](#introduction)
- [Generating Notifications](#generating-notifications)
- [Sending Notifications](#sending-notifications)
- [Mail Notifications](#mail-notifications)
- [Database Notifications](#database-notifications)
- [Broadcast Notifications](#broadcast-notifications)
- [SMS Notifications](#sms-notifications)
- [Slack Notifications](#slack-notifications)
- [Localizing Notifications](#localizing-notifications)
- [Testing](#testing)

## Introduction

In addition to support for sending email, Laravel provides support for sending notifications across a variety of delivery channels, including email, SMS (via Vonage), and Slack. Notifications may also be stored in a database so they may be displayed in your web interface.

Typically, notifications should be short, informational messages that notify users of something that occurred in your application.

## Generating Notifications

In Laravel, each notification is represented by a single class that is typically stored in the `app/Notifications` directory:

```bash
php artisan make:notification InvoicePaid
```

This command will place a fresh notification class in your `app/Notifications` directory. Each notification class contains a `via` method and a variable number of message building methods.

## Sending Notifications

### Using the Notifiable Trait

Notifications may be sent using the `notify` method of the `Notifiable` trait. The `Notifiable` trait is included on your application's `App\Models\User` model by default:

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

### Using the Notification Facade

Alternatively, you may send notifications via the `Notification` facade:

```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));
```

### Specifying Delivery Channels

Every notification class has a `via` method that determines on which channels the notification will be delivered:

```php
/**
 * Get the notification's delivery channels.
 *
 * @return array<int, string>
 */
public function via(object $notifiable): array
{
    return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
}
```

Notifications may be sent on the `mail`, `database`, `broadcast`, `vonage`, and `slack` channels.

### Queueing Notifications

Before queueing notifications, you should configure your queue and start a worker. Sending notifications can take time, so let your notification be queued by adding the `ShouldQueue` interface:

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

Once the `ShouldQueue` interface has been added, Laravel will detect it and automatically queue the delivery.

#### Delaying Notifications

If you would like to delay the delivery of the notification:

```php
$delay = now()->plus(minutes: 10);

$user->notify((new InvoicePaid($invoice))->delay($delay));
```

Or define a `withDelay` method on the notification class itself.

#### Customizing the Notification Queue Connection

If you would like to specify a different connection:

```php
public function __construct()
{
    $this->onConnection('redis');
}
```

### On-Demand Notifications

Sometimes you may need to send a notification to someone who is not stored as a "user" of your application:

```php
use Illuminate\Support\Facades\Notification;

Notification::route('mail', '[email protected]')
    ->route('vonage', '5555555555')
    ->notify(new InvoicePaid($invoice));
```

## Mail Notifications

### Formatting Mail Messages

If a notification supports being sent as an email, you should define a `toMail` method:

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
        ->greeting('Hello!')
        ->line('One of your invoices has been paid!')
        ->lineIf($this->amount > 0, "Amount paid: {$this->amount}")
        ->action('View Invoice', $url)
        ->line('Thank you for using our application!');
}
```

#### Customizing the Recipient

When sending notifications via the `mail` channel, the notification system will automatically look for an `email` property on your notifiable entity. You may customize which email address is used by defining a `routeNotificationForMail` method:

```php
public function routeNotificationForMail(Notification $notification): array|string
{
    // Return email address only...
    return $this->email_address;

    // Return email address and name...
    return [$this->email_address => $this->name];
}
```

#### Customizing the Subject

By default, the email's subject is the class name of the notification. If you would like to specify a different subject:

```php
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Notification Subject')
        ->line('...');
}
```

### Mail Attachments

To add attachments to an email notification:

```php
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attach('/path/to/file');
}
```

You may also specify the display name and MIME type:

```php
->attach('/path/to/file', [
    'as' => 'name.pdf',
    'mime' => 'application/pdf',
]);
```

## Database Notifications

### Prerequisites

To use the database notification channel, you need a table to store notifications. Laravel can create this table for you:

```bash
php artisan notifications:table

php artisan migrate
```

### Formatting Database Notifications

To send notifications via the database channel, define a `toDatabase` method:

```php
use Illuminate\Notifications\Messages\DatabaseMessage;

/**
 * Get the database representation of the notification.
 */
public function toDatabase(object $notifiable): array
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

### Accessing the Notifications

To access the notifications for a user:

```php
foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

Unread notifications:

```php
foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

### Marking Notifications as Read

```php
// Mark a single notification as read...
$user->notifications->markAsRead();

// Mark all notifications as read...
$user->unreadNotifications->markAsRead();
```

## Broadcast Notifications

### Prerequisites

Broadcast notifications use Laravel's broadcasting system and require配置 of a Pusher driver.

### Formatting Broadcast Notifications

To send broadcast notifications, define a `toBroadcast` method:

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the broadcast representation of the notification.
 */
public function toBroadcast(object $notifiable): BroadcastMessage
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

## SMS Notifications

### Prerequisites

To use the Vonage driver:

```bash
composer require laravel/vonage-notifications-channel guzzlehttp/guzzle
```

### Formatting SMS Notifications

To send notifications via SMS, define a `toVonage` method:

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage())
        ->content('Your invoice has been paid!');
}
```

### Customizing the "From" Number

```php
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage())
        ->from('15551234567')
        ->content('Your invoice has been paid!');
}
```

### Routing SMS Notifications

You may specify which phone number the notification should be sent to:

```php
public function routeNotificationForVonage(Notification $notification): string
{
    return $this->phone_number;
}
```

## Slack Notifications

### Prerequisites

To use the Slack driver:

```bash
composer require laravel/slack-notifications-channel guzzlehttp/guzzle
```

### Formatting Slack Notifications

To send notifications via Slack, define a `toSlack` method:

```php
use Illuminate\Notifications\Messages\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->content('One of your invoices has been paid!')
        ->attachment(function ($attachment) {
            $attachment->title('Invoice', url('/invoices/'.$this->invoice->id))
                       ->fields([
                           'Amount' => $this->invoice->amount,
                           'Due Date' => $this->invoice->due_date->format('M j, Y'),
                       ]);
        });
}
```

### Slack Interactivity

Slack messages can include interactive buttons configured via the `action` method:

```php
use Illuminate\Notifications\Messages\SlackMessage;

public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->content('Deployment completed!')
        ->action('Rollback', url('/rollback/'.$this->deployment->id))
        ->action('View Log', url('/deployments/'.$this->deployment->id));
}
```

### Routing Slack Notifications

You may specify which Slack channel the notification should be sent to:

```php
public function routeNotificationForSlack(Notification $notification): string
{
    return $this->slack_channel;
}
```

## Localizing Notifications

Laravel allows you to set the locale for notifications:

```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
```

Using the Notification facade:

```php
Notification::send($users, new InvoicePaid($invoice))->locale('es');
```

## Testing

### Testing Mail Notifications

You may use the `Mail::fake()` method to test mail notifications:

```php
use Illuminate\Support\Facades\Mail;

Mail::fake();

// Assert a mailable was sent...
Mail::assertSent(OrderShipped::class);

// Assert a mailable was not sent...
Mail::assertNotSent(OrderShipped::class);

// Assert how many mailables were sent...
Mail::assertSentCount(1);
```

### Testing Database Notifications

```php
use Illuminate\Support\Facades\DB;

DB::fake();

$response = $this->postJson('/orders');

DB::assertInserted('notifications', function ($record) {
    return $record['type'] === 'App\Notifications\OrderShipped';
});
```

### Testing Broadcast Notifications

You may use the `Broadcast::fake()` method to test broadcast notifications.

## Custom Channels

You can create custom notification channels by creating a class that implements the `Channel` interface. See Laravel's documentation for detailed instructions on creating custom channels.
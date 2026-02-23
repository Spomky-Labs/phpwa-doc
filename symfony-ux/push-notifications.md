# Web Push Notifications

The Web Push Notifications component integrates with the [Web Push Bundle](https://web-push.spomky-labs.com/) to enable push notifications in your Progressive Web App. This Stimulus controller manages push notification subscriptions and handles user permission requests.

## Introduction

Web Push Notifications allow you to send timely, relevant updates to users even when they're not actively using your application. The component provides a simple interface to:

* Subscribe users to push notifications
* Manage user permission states (granted, denied, default)
* Handle subscription lifecycle events
* Integrate with the Web Push Bundle for server-side notification delivery

{% hint style="info" %}
This component requires the [Web Push Bundle](https://web-push.spomky-labs.com/) to be installed and configured. The bundle handles server-side notification delivery using VAPID authentication.
{% endhint %}

## Browser Support

<table><thead><tr><th width="200">Browser</th><th width="150">Desktop</th><th>Mobile</th></tr></thead><tbody>
<tr><td>Chrome/Edge</td><td>✅ Full</td><td>✅ Full</td></tr>
<tr><td>Firefox</td><td>✅ Full</td><td>✅ Full</td></tr>
<tr><td>Safari</td><td>✅ 16.1+</td><td>✅ 16.4+</td></tr>
<tr><td>Opera</td><td>✅ Full</td><td>✅ Full</td></tr>
<tr><td>Samsung Internet</td><td>N/A</td><td>✅ Full</td></tr>
</tbody></table>

{% hint style="warning" %}
Safari on macOS requires version 16.1+ and Safari on iOS requires version 16.4+. Web Push is not available in Safari Private Browsing mode.
{% endhint %}

## Prerequisites

Before using Web Push Notifications, you need to:

1. **Install and configure the Web Push Bundle**:

```bash
composer require spomky-labs/web-push
```

2. **Configure VAPID keys** in your `config/packages/web_push.yaml`:

{% code title="config/packages/web_push.yaml" lineNumbers="true" %}
```yaml
web_push:
    vapid:
        subject: 'mailto:your-email@example.com'
        public_key: '%env(VAPID_PUBLIC_KEY)%'
        private_key: '%env(VAPID_PRIVATE_KEY)%'
```
{% endcode %}

3. **Generate VAPID keys** using the Web Push Bundle command:

```bash
php bin/console web-push:generate:keys
```

4. **Enable Service Worker** in your PWA configuration as Web Push requires an active service worker.

## Usage

### Basic Setup with Twig Live Component

To implement Web Push, you should create either a Stimulus Controller to intercept events or a Twig Live Component. Below is an example of the latter. Please note that you must hook into the Service Worker to show the notifications to the user and allow action interaction

{% code title="src/App/Twig/Component/WebPush.php" lineNumbers="true" %}
```php
<?php

namespace App\Twig\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\Attribute\LiveAction;
use Symfony\UX\LiveComponent\Attribute\LiveArg;
use Symfony\UX\LiveComponent\Attribute\LiveListener;
use Symfony\UX\LiveComponent\Attribute\LiveProp;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\ComponentToolsTrait;
use WebPush\Subscription;

#[AsLiveComponent('WebPush')]
class WebPush
{
    use DefaultActionTrait;
    use ComponentToolsTrait;

    public function __construct(
        private readonly \WebPush\WebPush $webpushService
    ) {
    }

    #[LiveProp]
    public string $status = 'unknown';

    /**
     * @param array{auth: string, p256dh: string} $keys
     * @param string[] $supportedContentEncodings
     */
    #[LiveListener('subscribed')]
    public function onSubscription(
        #[LiveArg] string $endpoint,
        #[LiveArg] array $keys,
        #[LiveArg] array $supportedContentEncodings
    ): void {
        $this->status = 'subscribed';
        $subscription = json_encode([
            'endpoint' => $endpoint,
            'keys' => $keys,
            'supportedContentEncodings' => $supportedContentEncodings,
        ]);
        
        // Store the $subscription (Filesystem, Databse...)

        // If the user is logged in, associate this subscription;
        // you will be able to send targeted notification to that 
    }

    #[LiveListener('unsubscribed')]
    public function onUnsubscription(
    ): void {
        $this->status = 'unsubscribed';
    }
}

```
{% endcode %}

{% code title="templates/components/WebPush.html.twig" lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/web-push', { applicationServerKey: 'YOUR APPLICATION SERVER PUBLICKEY HERE'})|stimulus_controller('live') }} {{ attributes }}>
    <h2>Web Push Notifications</h2>
    <div>
        {% if this.status == 'unsubscribed' %}
        <button {{ stimulus_action('@pwa/web-push', 'subscribe', 'click')}}>
            Subscribe
        </button>
        {% endif %}
        {% if this.status == 'subscribed' %}
        <button {{ stimulus_action('@pwa/web-push', 'unsubscribe', 'click')}}>
            Unsubscribe
        </button>
        <button {{ stimulus_action('live', 'action', 'click', {action: 'notify'})}}>
            Send Test Message
        </button>
        {% endif %}
    </div>
    {% if this.status == 'unknown' %}
    <div>
        Status: <span class="font-mono">Not initialized</span>
    </div>
    {% endif %}
</div>

```
{% endcode %}

### Service Worker Hooks

The service worker shall be enable when using web push notifications. Please make sure [the configuration](broken-reference) is set accordingly.

In your service wroker JS file (we consider it is in `assets/sw.js` in this example), you must at least enable the push notification support. There are two types of notifications:

* Simple: only a message. No payload.
* Structured: a message, its payload and optionnaly actions

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
// Use one of the method
registerPushTask(structuredPushNotificationSupport);
//registerPushTask(simplePushNotificationSupport);
```
{% endcode %}

Let say we send the following message. This is a structured notification with buttons `'action1'` and `'action2'` .

```php
use WebPush\Action;
use WebPush\Message;

 $message = Message::create('My super Application.', 'Hello World! Clic on the body to go to Facebook')
    ->vibrate(200, 300, 200, 300)
    ->withImage('https://picsum.photos/1024/512')
    ->withIcon('https://picsum.photos/512/512')
    ->withBadge('https://picsum.photos/256/256')
    ->withLang('en_US')
    ->withTimestamp(time()*1000)
    ->withData('{"action1Url":"https://example.com/foo/bar", "action2Url":"https://example.com/baz/qux", "defaultUrl":"https://example.com"}')
    ->addAction(Action::create('action1', 'To Foo'))
    ->addAction(Action::create('action2', 'To Bar'));
;
```

You can decide what to do when the user clicks on the button or the notification. In the example, we will use the data associated to the notification to decide where to redirect the user for `action1` , `action2` or a click on the notification itself.

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
// Button name "action1" is clicked
registerNotificationAction('action1', async (event) => {
  const data = JSON.parse(event.notification.data);
  await clients.openWindow(data.action1Url);
});

// Button name "action2" is clicked
registerNotificationAction('action2', async (event) => {
  const data = JSON.parse(event.notification.data);
  await clients.openWindow(data.action2Url);
});

// '' means the notification (popup) is clicked.
registerNotificationAction('', async (event) => {
  const data = JSON.parse(event.notification.data);
  await clients.openWindow(data.defaultUrl);
});

```
{% endcode %}

{% hint style="info" %}
The event is of type [NotificationEvent](https://developer.mozilla.org/en-US/docs/Web/API/NotificationEvent) and you can access the notification object. Read this page to know more.
{% endhint %}

As it can become difficult to manage several action names, there is a wildcard action `'*'` that can be declared. It is not recommended to declare any other handlers when the wildcard is set.

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
registerNotificationAction('*', async (event) => {
  const data = JSON.parse(event.notification.data);
  const action = event.action || 'default';
  await clients.openWindow(data[action]);
});
```
{% endcode %}

## Best Practices

### Permission Request Timing

Don't request notification permission immediately when the user visits your site. Instead:

```php
// ❌ Bad: Request permission on page load
#[Route('/')]
public function index(): Response
{
    // Immediately showing permission prompt
    return $this->render('home.html.twig');
}

// ✅ Good: Request permission contextually
#[Route('/subscribe-to-updates')]
public function subscribeToUpdates(): Response
{
    // User explicitly chose to enable notifications
    return $this->render('subscribe.html.twig');
}
```

### Handle Permission States

Always handle all three permission states:

1. **Default**: User hasn't decided yet
2. **Granted**: User allowed notifications
3. **Denied**: User blocked notifications

```twig
{% if this.status == 'denied' %}
<div class="alert alert-warning">
    Notifications are blocked. Please enable them in your browser settings.
</div>
{% endif %}
```

### Store Subscriptions Securely

Store subscription data securely and associate it with authenticated users:

```php
#[LiveListener('subscribed')]
public function onSubscription(
    #[LiveArg] string $endpoint,
    #[LiveArg] array $keys,
    #[LiveArg] array $supportedContentEncodings
): void {
    $subscription = Subscription::create($endpoint)
        ->setKey('auth', $keys['auth'])
        ->setKey('p256dh', $keys['p256dh'])
        ->withContentEncodings($supportedContentEncodings);

    // Store in database with user association
    $this->subscriptionRepository->save(
        $this->getUser(),
        $subscription
    );
}
```

### Notification Delivery

When sending notifications from the server, use the Web Push service:

```php
use WebPush\Bundle\Service\WebPush;
use WebPush\Message;
use WebPush\Action;
use WebPush\Notification;

class NotificationService
{
    public function __construct(
        private readonly WebPush $webPush
    ) {}

    public function sendNotification(Subscription $subscription): void
    {
        $message = Message::create('New Update Available', 'Click to view details')
            ->withIcon('/icon-192.png')
            ->withBadge('/badge-96.png')
            ->addAction(Action::create('view', 'View Now'))
            ->addAction(Action::create('dismiss', 'Later'));

        $notification = Notification::create()
            ->withPayload($message->toString())
            ->withTTL(3600) // 1 hour
            ->highUrgency();

        $statusReport = $this->webPush->send($notification, $subscription);

        // Check if subscription expired
        if ($statusReport->isSubscriptionExpired()) {
            // Remove subscription from database
            $this->subscriptionRepository->remove($subscription);
        }
    }
}
```

## Common Use Cases

### 1. News and Content Updates

Notify users when new content is published:

```php
// Send notification when article is published
class ArticlePublisher
{
    public function publish(Article $article): void
    {
        // ... publish article

        $message = Message::create(
            $article->getTitle(),
            substr($article->getContent(), 0, 100) . '...'
        )
        ->withImage($article->getFeaturedImage())
        ->withIcon('/icon-192.png')
        ->withData(json_encode([
            'url' => $this->urlGenerator->generate('article_view', [
                'slug' => $article->getSlug()
            ])
        ]));

        $notification = Notification::create()
            ->withPayload($message->toString())
            ->withTTL(86400) // 24 hours
            ->normalUrgency();

        foreach ($this->getSubscribedUsers($article->getCategory()) as $subscription) {
            $this->webPush->send($notification, $subscription);
        }
    }
}
```

### 2. Real-time Messaging

Send instant message notifications:

```php
class ChatNotifier
{
    public function notifyNewMessage(User $recipient, Message $message): void
    {
        $notification = Message::create(
            $message->getSender()->getName(),
            $message->getContent()
        )
        ->withIcon($message->getSender()->getAvatarUrl())
        ->withBadge('/badge-unread.png')
        ->renotify() // Always notify even if previous notification exists
        ->withTag('chat-' . $message->getConversationId())
        ->withData(json_encode([
            'conversationUrl' => $this->urlGenerator->generate('chat_conversation', [
                'id' => $message->getConversationId()
            ])
        ]));

        $pushNotification = Notification::create()
            ->withPayload($notification->toString())
            ->highUrgency()
            ->withTTL(3600);

        foreach ($recipient->getPushSubscriptions() as $subscription) {
            $this->webPush->send($pushNotification, $subscription);
        }
    }
}
```

### 3. E-commerce Order Updates

Keep customers informed about their orders:

```php
class OrderStatusNotifier
{
    public function notifyStatusChange(Order $order, string $newStatus): void
    {
        $statusMessages = [
            'confirmed' => 'Your order has been confirmed!',
            'shipped' => 'Your order has been shipped!',
            'delivered' => 'Your order has been delivered!',
        ];

        $message = Message::create(
            'Order #' . $order->getNumber(),
            $statusMessages[$newStatus] ?? 'Order status updated'
        )
        ->withIcon('/icon-192.png')
        ->withBadge('/badge-order.png')
        ->addAction(Action::create('view', 'View Order'))
        ->addAction(Action::create('track', 'Track Package'))
        ->withData(json_encode([
            'orderUrl' => $this->urlGenerator->generate('order_view', [
                'id' => $order->getId()
            ]),
            'trackingUrl' => $order->getTrackingUrl()
        ]));

        $notification = Notification::create()
            ->withPayload($message->toString())
            ->withTTL(604800) // 7 days
            ->highUrgency();

        foreach ($order->getCustomer()->getPushSubscriptions() as $subscription) {
            $this->webPush->send($notification, $subscription);
        }
    }
}
```

### 4. Reminder System

Send time-based reminders to users:

```php
class ReminderService
{
    public function sendReminder(Reminder $reminder): void
    {
        $message = Message::create(
            $reminder->getTitle(),
            $reminder->getDescription()
        )
        ->withIcon('/icon-reminder.png')
        ->withBadge('/badge-bell.png')
        ->vibrate(200, 100, 200)
        ->interactionRequired() // Keep notification visible until user interacts
        ->withTag('reminder-' . $reminder->getId())
        ->addAction(Action::create('complete', 'Mark Complete'))
        ->addAction(Action::create('snooze', 'Snooze'))
        ->withData(json_encode([
            'reminderId' => $reminder->getId(),
            'snoozeUrl' => $this->urlGenerator->generate('reminder_snooze', [
                'id' => $reminder->getId()
            ])
        ]));

        $notification = Notification::create()
            ->withPayload($message->toString())
            ->highUrgency()
            ->withTTL(86400);

        foreach ($reminder->getUser()->getPushSubscriptions() as $subscription) {
            $this->webPush->send($notification, $subscription);
        }
    }
}
```

## API Reference

### Values

#### `applicationServerKey`

**Type**: `String` (required)

The VAPID public key for authentication. This is your `VAPID_PUBLIC_KEY` from the Web Push Bundle configuration.

```twig
{{ stimulus_controller('@pwa/web-push', {
    applicationServerKey: vapid_public_key
}) }}
```

### Actions

#### `status()`

Checks the current push notification subscription status and dispatches an event with the result.

```twig
<button {{ stimulus_action('@pwa/web-push', 'status') }}>
    Check Status
</button>
```

**Dispatches**: `pwa--web-push:subscribed` or `pwa--web-push:unsubscribed`

#### `subscribe()`

Requests notification permission from the user and subscribes to push notifications if granted.

```twig
<button {{ stimulus_action('@pwa/web-push', 'subscribe') }}>
    Enable Notifications
</button>
```

**Dispatches**:
- `pwa--web-push:subscribed` on success
- `pwa--web-push:denied` if user denies permission
- `pwa--web-push:error` on error

#### `unsubscribe()`

Unsubscribes the current device from push notifications.

```twig
<button {{ stimulus_action('@pwa/web-push', 'unsubscribe') }}>
    Disable Notifications
</button>
```

**Dispatches**: `pwa--web-push:unsubscribed`

### Events

#### `pwa--web-push:subscribed`

Fired when a user successfully subscribes to push notifications or when checking status finds an active subscription.

**Event Details**:
```javascript
{
    endpoint: string,                    // Push service endpoint
    keys: {
        auth: string,                    // Authentication secret
        p256dh: string                   // Public key for encryption
    },
    supportedContentEncodings: string[]  // Supported encoding methods
}
```

**Example**:
```javascript
element.addEventListener('pwa--web-push:subscribed', (event) => {
    console.log('Subscription endpoint:', event.detail.endpoint);
    console.log('Auth key:', event.detail.keys.auth);
    console.log('P256dh key:', event.detail.keys.p256dh);
});
```

#### `pwa--web-push:unsubscribed`

Fired when a user unsubscribes or when checking status finds no active subscription.

**Example**:
```javascript
element.addEventListener('pwa--web-push:unsubscribed', () => {
    console.log('User is not subscribed to push notifications');
});
```

#### `pwa--web-push:denied`

Fired when the user denies notification permission.

**Example**:
```javascript
element.addEventListener('pwa--web-push:denied', () => {
    alert('Please enable notifications in your browser settings');
});
```

#### `pwa--web-push:error`

Fired when an error occurs during subscription or unsubscription.

**Event Details**:
```javascript
{
    error: Error  // The error object
}
```

**Example**:
```javascript
element.addEventListener('pwa--web-push:error', (event) => {
    console.error('Push notification error:', event.detail.error);
});
```

## Related Components

* [Service Worker](../the-service-worker/) - Required for Web Push functionality
* [Web Push Bundle Documentation](https://web-push.spomky-labs.com/) - Server-side notification delivery

## Additional Resources

* [Web Push API on MDN](https://developer.mozilla.org/en-US/docs/Web/API/Push_API)
* [Notifications API on MDN](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API)
* [VAPID Protocol Specification](https://datatracker.ietf.org/doc/html/rfc8292)

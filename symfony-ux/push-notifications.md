# Web Push Notifications

If you are interested in Web Push notifications, please read about the bundle we maintain: [https://web-push.spomky-labs.com/](https://web-push.spomky-labs.com/). This bundle provides a Stimulus Controller to ease the push notification subscription and clicks on the message or buttons (if any).

To implement this, you must create either a Stimulus Controller to intercept events or a Twig Live Component. Below is an example of the latter. Please note that you shall hook into the Service Worker to shwo the notifications to the user and allow action interaction

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

### Parameters

`applicationServerKey`: the public key for the cypher operations. This parameter is mandatory.

### Actions

`status`: Signals the Stimulus Component to verify subscription status and triggers an event.

`subscribe`: Asks the user to accept or decline web push notification on the current device.

`unsubscribe`: Unsubscribe the user from future web push notification on the current device.

### Targets

None

### Events

`pwa--web-push:unsubscribed`

`pwa--web-push:subscribed`

`pwa--web-push:denied`

`pwa--web-push:error`

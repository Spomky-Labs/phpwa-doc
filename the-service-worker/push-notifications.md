# Push Notifications

If you are interested in Web Push notifications from the server side, please read about the bundle we maintain: [https://web-push.spomky-labs.com/](https://web-push.spomky-labs.com/)

As handling Web Push notifications can differ from one application to another, this bundle does not provide easy way to manage them. You will hav to write your own logic into the Service Worker.

Hereafter an example where the received notifications are displayed.

```javascript
self.addEventListener('push', async function (event) {
    if (!(self.Notification && self.Notification.permission === 'granted')) {
        return;
    }
    const {title, options}= event.data.json();
    event.waitUntil(self.registration.showNotification(title, options));
});
```

{% hint style="info" %}
More examples to come
{% endhint %}

# Periodic Sync

Utilize the Periodic Sync API to efficiently retrieve updates when the app is closed. Contrary to the other features of this bundle, you will have to write few JS lines of code.

## Service Worker Functions

To implement periodic tasks in your service worker, create a function for operations like cache cleaning or blog post retrieval.

In this example, we will ping the `/ping` endpoint, cache the response, and notify clients. This task will be registered and linked to the `ping` tag. The tag will be needed on the client side.

{% code title="assets/sw.js" overflow="wrap" lineNumbers="true" %}
```javascript
const ping = async () => {
  const cache = await openCache('ping-cache');
  const res = await fetch('/ping');
  await cache.put('/ping', res.clone());

  notifyPeriodicSyncClients('ping', { updated: true });
};

registerPeriodicSyncTask('ping', ping);
```
{% endcode %}

{% hint style="success" %}
The functions `openCache` , `notifyPeriodicSyncClients` and `registerPeriodicSyncTask` are helpers provided by the Service Worker.
{% endhint %}

{% hint style="info" %}
&#x20;You can register multiple tasks under the same tag.
{% endhint %}

## Client Site Fucntions

On the client side, we will need to tell the client to periodically execute the tasks registered with the defined tasks.

Now we prompt the client to perform the tasks under the `ping` tag every 6 hours.

{% code title="assets/app.js" overflow="wrap" lineNumbers="true" %}
```javascript
//... shortened for brevity

import {registerPeriodicSync} from '@spomky-labs/pwa/helpers';

await registerPeriodicSync('ping', 6 * 60 * 60 * 1000); //Every 6 hours in milliseconds
```
{% endcode %}

{% hint style="warning" %}
The browser is not required to respect the declared interval (e.g. every 6 hours).

**The actual execution time** depends on several factors, such as

* whether the application is installed as a PWA,
* the site engagement score,
* battery and data-saver settings,
* and network availability.

In practice, short intervals (like a few seconds or minutes) are ignored. As a result, the periodic sync may be delayed or skipped entirely, especially during extended user inactivity or limited connectivity.
{% endhint %}

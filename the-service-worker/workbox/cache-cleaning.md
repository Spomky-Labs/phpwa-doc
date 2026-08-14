# Cache Cleaning

When the service worker changes and is activated, the cache is purged. This feature is made to avoid the cache to grow in size and prevent your application to reach the limit of the host system or browser.

You can disable the cache purge, however you have to make sure the cached data is managed by other means.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            clear_cache: false # Default to true
```
{% endcode %}

## What actually gets purged

{% hint style="warning" %}
Only the caches the bundle knows about are purged. A cache your own service worker source opens
with `caches.open('my-app-data')` is **not** deleted, and nothing warns you about it.
{% endhint %}

The bundle keeps a registry of the cache names it creates, filled by `registerCacheName()`. Every
strategy generated from your configuration goes through it, so the asset cache, the font cache,
the image cache, the resource caches and the offline fallback are all covered.

Caches you create yourself are not, for the simple reason that the bundle has no way of knowing
about them.

## Purging your own caches

Register a listener from your own service worker source. It receives every cache name currently
stored and returns the ones to delete:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
registerClearCacheListener((names) =>
    names.filter((name) => name.startsWith('my-app-'))
);
```
{% endcode %}

The callback shape is what makes versioned caches work. Keeping the current version while dropping
the previous ones is a filter, not a list:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const CURRENT = 'my-app-v3';

registerClearCacheListener((names) =>
    names.filter((name) => name.startsWith('my-app-') && name !== CURRENT)
);
```
{% endcode %}

Listeners may be async, which is useful when the decision depends on stored state:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
registerClearCacheListener(async (names) => {
    const keep = await readCurrentVersionFromIndexedDb();

    return names.filter((name) => name.startsWith('my-app-') && name !== keep);
});
```
{% endcode %}

You can register as many listeners as you need; their results are merged. A listener that throws
is logged to the console and skipped, so it neither stops the other listeners nor breaks the
install event.

{% hint style="info" %}
Listeners only run when cache clearing is enabled. With `clear_cache: false` nothing is purged at
all, yours included.
{% endhint %}

## The other way round

If you write a [custom service worker rule](../custom-service-worker-rule.md) that creates a
cache, pass its name through `registerCacheName()` instead. It adds the name to the registry and
returns it, so it drops straight into a strategy:

{% code title="src/ServiceWorker/MyRule.php" lineNumbers="true" %}
```php
$declaration = <<<'RULE'
const myStrategy = new workbox.strategies.CacheFirst({
  cacheName: registerCacheName('my-cache'),
});
RULE;
```
{% endcode %}

# Custom Service Worker Rule

If needed, you can define custom sections in the service worker appended by your own services and depending on your application configuration or requirements.

To do so, create a service that implements `SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRuleInterface`. It is auto-registered, nothing to declare.

{% hint style="info" %}
The `process()` method returns valid JavaScript as a string. It is appended to the generated
service worker and executed by browsers.
{% endhint %}

```php
<?php

declare(strict_types=1);

namespace App\ServiceWorker;

use SpomkyLabs\PwaBundle\Dto\Workbox;
use SpomkyLabs\PwaBundle\Service\ServiceWorkerBuilder;
use SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRuleInterface;

final readonly class MyCustomRule implements ServiceWorkerRuleInterface
{
    private Workbox $workbox;

    public function __construct(ServiceWorkerBuilder $serviceWorkerBuilder)
    {
        $this->workbox = $serviceWorkerBuilder->create()->workbox;
    }

    public function process(bool $debug = false): string
    {
        if ($this->workbox->enabled === false) {
            return '';
        }

        return <<<'HELLO_WORLD'
// This will be added to the Service Worker
console.log('FOO-BAR from the Service Worker!');
HELLO_WORLD;
    }
}
```

{% hint style="warning" %}
`SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRule` is deprecated since 1.2.0 and will be
removed in 2.0.0. Implement `ServiceWorkerRuleInterface` instead.
{% endhint %}

## Ordering

Rules run at the default priority unless you say otherwise. Use `#[AsTaggedItem]` when your rule
must come before or after another one:

```php
use Symfony\Component\DependencyInjection\Attribute\AsTaggedItem;

#[AsTaggedItem(priority: 500)]
final readonly class MyCustomRule implements ServiceWorkerRuleInterface
```

The higher the priority, the earlier the rule runs. The bundle reserves the top of the range:

| Priority | Rule | What it does |
|----------|------|--------------|
| 1024 | `WorkboxImport` | `importScripts()` of Workbox |
| 1023 | `WorkboxHelpers` | Declares the helper functions listed below |
| 1022 | `NavigationPreload` | Enables navigation preload |
| 0 | everything else | Cache strategies, offline fallback, cache clearing… |

Your own service worker source, the one configured through the `src` option, is appended **after**
every rule. It therefore sees everything the bundle declared, and it is the right place for
listeners and one-off logic. Use a rule when the JavaScript has to be generated from PHP, for
instance because it embeds routes, translations or configuration values.

## Available helpers

`WorkboxHelpers` declares a handful of functions your rule can rely on.

### `registerInstallTask(callback, priority = 100)`

Registers a task to run on the `install` event. Tasks run in ascending priority order, unlike
native `install` listeners which run concurrently. This is why the helper exists: cache clearing
runs at 0, `skipWaiting()` at 5, the offline fallback at 10 and the precaching at 100.

```js
registerInstallTask(() => doSomething(), 50);
```

### `precacheResources(strategy, urls, event)`

Warms a Workbox strategy with a list of URLs. Meant to be called from an install task.

### `registerCacheName(name)`

Adds a cache name to the registry and returns it, so it drops straight into a strategy:

```js
const myStrategy = new workbox.strategies.CacheFirst({
  cacheName: registerCacheName('my-cache'),
});
```

{% hint style="warning" %}
This matters more than it looks. [Cache cleaning](workbox/cache-cleaning.md) only purges the names
in that registry. A cache created with a plain string survives every install, silently.
{% endhint %}

### `openCache(name)`

Opens a cache and memoises the instance for the lifetime of the service worker. It does **not**
register the name; combine it with `registerCacheName()` if you want the cache purged.

### `registerClearCacheListener(callback)`

Lets you contribute cache names to delete when the cache is cleared. See
[Cache cleaning](workbox/cache-cleaning.md).

### `statusGuard(min, max)`

Returns a Workbox plugin that treats a response status range as a failure.

### `createBackgroundSyncPlugin(queueName, maxRetentionTime, forceSyncFallback)`

Returns a Workbox `BackgroundSyncPlugin`. The `createBackgroundSyncPluginWithBroadcast()` variant
takes an extra channel name and reports queue progress on a `BroadcastChannel`.

{% hint style="danger" %}
A number of other helpers, `registerPushTask()`, `registerMessageTask()`,
`registerNotificationAction()`, `registerPeriodicSyncTask()`, `registerBackgroundFetchTask()`,
`registerCacheFirst()` and `openBackgroundFetchDatabase()`, are deprecated since 1.6.0 and removed
in 2.0.0. They wrap events the bundle never handles, so a plain `self.addEventListener()` in your
own source does the same job. See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md).
{% endhint %}

# Custom Service Worker Rule

If needed, you can define custom sections in the service worker appended by your own services and depending on your application configuration or requirements.

To do so, you can create a service that implements `SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRuleInterface`.

{% hint style="info" %}
The method `process` shall return valid JavaScript as a string. This script will be executed by browsers.
{% endhint %}

## Basic Example

```php
<?php

declare(strict_types=1);

namespace App\ServiceWorkerRule;

use SpomkyLabs\PwaBundle\Dto\ServiceWorker;
use SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRuleInterface;

final readonly class MyCustomRule implements ServiceWorkerRuleInterface
{
    public function __construct(
        private ServiceWorker $serviceWorker,
    ) {}

    public function process(bool $debug = false): string
    {
        return <<<JS
// This will be added to the Service Worker
console.log('FOO-BAR from the Service Worker!');
JS;
    }
}
```

{% hint style="success" %}
When using Symfony's service autoconfiguration, your rule will be automatically registered. If autoconfiguration is disabled, tag your service with `spomky_labs_pwa.service_worker_rule`.
{% endhint %}

## Manual Service Registration

If you need to manually register your service:

{% code title="config/services.yaml" lineNumbers="true" %}
```yaml
services:
    App\ServiceWorkerRule\MyCustomRule:
        tags: ['spomky_labs_pwa.service_worker_rule']
```
{% endcode %}

## Advanced Example with Configuration

You can access the service worker configuration to conditionally generate JavaScript:

```php
<?php

declare(strict_types=1);

namespace App\ServiceWorkerRule;

use SpomkyLabs\PwaBundle\Dto\ServiceWorker;
use SpomkyLabs\PwaBundle\Dto\Workbox;
use SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRuleInterface;

final readonly class ConditionalAnalyticsRule implements ServiceWorkerRuleInterface
{
    public function __construct(
        private ServiceWorker $serviceWorker,
    ) {}

    public function process(bool $debug = false): string
    {
        // Access Workbox configuration
        $workbox = $this->serviceWorker->workbox;

        if (!$workbox->enabled) {
            return '';
        }

        $analyticsCode = $debug
            ? "console.log('Analytics tracking (debug mode)');"
            : "// Production analytics code";

        return <<<JS
// Custom Analytics Tracking
self.addEventListener('fetch', (event) => {
    if (event.request.mode === 'navigate') {
        {$analyticsCode}
    }
});
JS;
    }
}
```

## Rule Execution Order

Service worker rules are executed in a specific order based on their priority. The bundle's built-in rules have predefined priorities:

| Rule | Priority | Purpose |
|------|----------|---------|
| WorkboxImport | 1024 | Imports Workbox library |
| WorkboxHelpers | 1023 | Helper functions |
| Custom Rules | 0 (default) | Your custom rules |
| Cache Strategies | -512 | Cache configuration |
| Other Rules | Various | Specialized functionality |

To control when your rule executes, you can implement additional logic or structure your JavaScript to work regardless of execution order.

## Best Practices

1. **Return empty string for disabled features**: If your rule depends on configuration, return an empty string when it shouldn't be active
2. **Use heredoc for multi-line JavaScript**: Makes the code more readable
3. **Respect the debug flag**: Provide additional logging or different behavior when `$debug === true`
4. **Access configuration through the ServiceWorker DTO**: Don't rely on external configuration sources
5. **Test thoroughly**: Service workers run in a different context and can be hard to debug

## Example: Custom Cache Invalidation

```php
<?php

declare(strict_types=1);

namespace App\ServiceWorkerRule;

use SpomkyLabs\PwaBundle\Dto\ServiceWorker;
use SpomkyLabs\PwaBundle\ServiceWorkerRule\ServiceWorkerRuleInterface;

final readonly class CacheInvalidationRule implements ServiceWorkerRuleInterface
{
    public function __construct(
        private ServiceWorker $serviceWorker,
        private string $appVersion,
    ) {}

    public function process(bool $debug = false): string
    {
        $debugLog = $debug ? "console.log('Cache invalidation rule active');" : '';

        return <<<JS
// Cache Invalidation - Version: {$this->appVersion}
{$debugLog}

self.addEventListener('activate', (event) => {
    const currentVersion = '{$this->appVersion}';

    event.waitUntil(
        caches.keys().then((cacheNames) => {
            return Promise.all(
                cacheNames
                    .filter((cacheName) => !cacheName.includes(currentVersion))
                    .map((cacheName) => caches.delete(cacheName))
            );
        })
    );
});
JS;
    }
}
```

{% hint style="warning" %}
Be careful when manipulating the service worker lifecycle events (`install`, `activate`, `fetch`). Improper handling can break your application's offline functionality.
{% endhint %}

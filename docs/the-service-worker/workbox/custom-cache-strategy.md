# Custom cache strategy

The bundle provides a comprehensive set of cache strategies through configuration. However, for advanced use cases requiring specific behaviors not covered by the YAML configuration, you can create custom cache strategies programmatically.

## When to use custom strategies

Use custom cache strategies when you need:

- **Dynamic behavior**: Cache strategies that change based on runtime conditions
- **Complex matching logic**: URL patterns too complex for regex or simple callbacks
- **Custom plugins**: Workbox plugins with specific configuration
- **Conditional caching**: Different strategies based on user state, headers, or other factors
- **Business logic integration**: Caching tied to your application's domain logic

For most use cases, the [YAML configuration](../configuration.md) and [resource caching](resource-caching.md) are sufficient.

## Quick start: Using WorkboxCacheStrategy

The easiest way to create custom cache strategies is using the `WorkboxCacheStrategy` helper class:

```php title="src/Service/MyCustomCacheStrategies.php" linenums="1"
<?php

declare(strict_types=1);

namespace App\Service;

use SpomkyLabs\PwaBundle\CachingStrategy\CacheStrategyInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\HasCacheStrategiesInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\WorkboxCacheStrategy;
use SpomkyLabs\PwaBundle\WorkboxPlugin\ExpirationPlugin;

final readonly class MyCustomCacheStrategies implements HasCacheStrategiesInterface
{
    /**
     * @return array<CacheStrategyInterface>
     */
    public function getCacheStrategies(): array
    {
        return [
            WorkboxCacheStrategy::create(
                enabled: true,
                requireWorkbox: true,
                strategy: CacheStrategyInterface::STRATEGY_CACHE_FIRST,
                matchCallback: '({url}) => url.pathname.startsWith("/api/static/")',
            )
                ->withName('api-static-data')
                ->withPlugin(
                    ExpirationPlugin::create(
                        maxEntries: 50,
                        maxAgeSeconds: 86400, // 1 day
                    )
                ),
        ];
    }
}
```

!!! info

    With service autoconfiguration enabled (default in Symfony), the tag `spomky_labs_pwa.cache_strategy` is added automatically. Otherwise, you need to tag your service manually.

## Complete example: Multiple strategies

```php title="src/Service/AdvancedCacheStrategies.php" linenums="1"
<?php

declare(strict_types=1);

namespace App\Service;

use SpomkyLabs\PwaBundle\CachingStrategy\CacheStrategyInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\HasCacheStrategiesInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\WorkboxCacheStrategy;
use SpomkyLabs\PwaBundle\WorkboxPlugin\BroadcastUpdatePlugin;
use SpomkyLabs\PwaBundle\WorkboxPlugin\CacheableResponsePlugin;
use SpomkyLabs\PwaBundle\WorkboxPlugin\ExpirationPlugin;

final readonly class AdvancedCacheStrategies implements HasCacheStrategiesInterface
{
    /**
     * @return array<CacheStrategyInterface>
     */
    public function getCacheStrategies(): array
    {
        return [
            $this->createAdminApiStrategy(),
            $this->createUserContentStrategy(),
            $this->createCdnStrategy(),
        ];
    }

    private function createAdminApiStrategy(): CacheStrategyInterface
    {
        return WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
            matchCallback: '({url}) => url.pathname.startsWith("/admin/api/")',
        )
            ->withName('admin-api')
            ->withMethod('POST')
            ->withOptions(['networkTimeoutSeconds' => 2])
            ->withPlugin(
                CacheableResponsePlugin::create(statuses: [200]),
                ExpirationPlugin::create(
                    maxEntries: 20,
                    maxAgeSeconds: 300, // 5 minutes
                ),
            );
    }

    private function createUserContentStrategy(): CacheStrategyInterface
    {
        return WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_STALE_WHILE_REVALIDATE,
            matchCallback: 'regex: /\\/user\\/[0-9]+\\/profile/',
        )
            ->withName('user-profiles')
            ->withPlugin(
                BroadcastUpdatePlugin::create([
                    'Content-Type',
                    'ETag',
                    'Last-Modified',
                ]),
                CacheableResponsePlugin::create(statuses: [0, 200]),
                ExpirationPlugin::create(
                    maxEntries: 100,
                    maxAgeSeconds: 3600, // 1 hour
                ),
            );
    }

    private function createCdnStrategy(): CacheStrategyInterface
    {
        return WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_CACHE_FIRST,
            matchCallback: 'origin: https://cdn.example.com',
        )
            ->withName('external-cdn')
            ->withPlugin(
                CacheableResponsePlugin::create(statuses: [0, 200]),
                ExpirationPlugin::create(
                    maxEntries: 200,
                    maxAgeSeconds: 2592000, // 30 days
                ),
            );
    }
}
```

## WorkboxCacheStrategy API

### create()

Create a new strategy instance:

```php
WorkboxCacheStrategy::create(
    enabled: true,              // Enable/disable this strategy
    requireWorkbox: true,       // Whether Workbox is required
    strategy: string,           // One of the 5 Workbox strategies
    matchCallback: string,      // URL matching pattern or callback
)
```

**Available strategies:**
- `CacheStrategyInterface::STRATEGY_CACHE_FIRST` - Cache first, fallback to network
- `CacheStrategyInterface::STRATEGY_NETWORK_FIRST` - Network first, fallback to cache
- `CacheStrategyInterface::STRATEGY_STALE_WHILE_REVALIDATE` - Cache immediately, update in background
- `CacheStrategyInterface::STRATEGY_CACHE_ONLY` - Cache only, never network
- `CacheStrategyInterface::STRATEGY_NETWORK_ONLY` - Network only, never cache

### Fluent methods

| Method | Description |
|--------|-------------|
| `withName(string)` | Set a custom cache name |
| `withMethod(string)` | Specify HTTP method to intercept (e.g., `'POST'`, `'GET'`) |
| `withPlugin(CachePluginInterface, ...)` | Add one or more Workbox plugins |
| `withPreloadUrl(string, ...)` | Precache specific URLs on service worker install |
| `withOptions(array)` | Set additional Workbox options (e.g., `networkTimeoutSeconds`) |

## Available Workbox plugins

### ExpirationPlugin

Automatically remove old cache entries:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\ExpirationPlugin;

ExpirationPlugin::create(
    maxEntries: 50,
    maxAgeSeconds: 86400, // 1 day
)
```

### CacheableResponsePlugin

Control which responses get cached:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\CacheableResponsePlugin;

// Cache only successful responses
CacheableResponsePlugin::create(statuses: [200])

// Cache successful and opaque responses
CacheableResponsePlugin::create(statuses: [0, 200])
```

### BroadcastUpdatePlugin

Notify clients when cached data updates:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\BroadcastUpdatePlugin;

BroadcastUpdatePlugin::create([
    'Content-Type',
    'ETag',
    'Last-Modified',
])
```

Listen for updates in your application:

```javascript
const channel = new BroadcastChannel('workbox');
channel.addEventListener('message', (event) => {
    if (event.data.type === 'CACHE_UPDATED') {
        console.log('Cache updated:', event.data);
    }
});
```

### BackgroundSyncPlugin

Queue failed requests for retry when online:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\BackgroundSyncPlugin;

BackgroundSyncPlugin::create(
    queueName: 'api-queue',
    maxRetentionTime: 2880,           // 2 days in minutes
    forceSyncFallback: false,
    broadcastChannel: 'sync-updates',
)
```

### RangeRequestsPlugin

Support HTTP range requests for large files:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\RangeRequestsPlugin;

RangeRequestsPlugin::create()
```

## Match callback patterns

The `matchCallback` parameter supports several formats:

### Path prefix
```php
matchCallback: 'startsWith: /api/'
```

### Regular expression
```php
matchCallback: 'regex: /\\/product\\/[0-9]+/'
```

### Origin match
```php
matchCallback: 'origin: https://api.example.com'
```

### Custom JavaScript callback
```php
matchCallback: '({url, request}) => url.pathname.endsWith(".json") && request.method === "GET"'
```

### Navigation requests
```php
matchCallback: 'navigate'
```

## Conditional strategies

Create strategies that adapt based on conditions:

```php title="src/Service/ConditionalCacheStrategy.php" linenums="1"
<?php

declare(strict_types=1);

namespace App\Service;

use SpomkyLabs\PwaBundle\CachingStrategy\CacheStrategyInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\HasCacheStrategiesInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\WorkboxCacheStrategy;
use SpomkyLabs\PwaBundle\WorkboxPlugin\ExpirationPlugin;
use Symfony\Component\DependencyInjection\Attribute\Autowire;

final readonly class ConditionalCacheStrategy implements HasCacheStrategiesInterface
{
    public function __construct(
        #[Autowire(param: 'kernel.environment')]
        private string $environment,
    ) {
    }

    public function getCacheStrategies(): array
    {
        if ($this->environment === 'prod') {
            return [
                WorkboxCacheStrategy::create(
                    enabled: true,
                    requireWorkbox: true,
                    strategy: CacheStrategyInterface::STRATEGY_CACHE_FIRST,
                    matchCallback: '({url}) => url.pathname.startsWith("/static/")',
                )
                    ->withName('static-production')
                    ->withPlugin(
                        ExpirationPlugin::create(maxEntries: 200, maxAgeSeconds: 2592000)
                    ),
            ];
        }

        return [
            WorkboxCacheStrategy::create(
                enabled: true,
                requireWorkbox: true,
                strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
                matchCallback: '({url}) => url.pathname.startsWith("/static/")',
            )
                ->withName('static-development')
                ->withOptions(['networkTimeoutSeconds' => 1])
                ->withPlugin(
                    ExpirationPlugin::create(maxEntries: 10, maxAgeSeconds: 60)
                ),
        ];
    }
}
```

## Implementing CacheStrategyInterface directly

For complete control, implement `CacheStrategyInterface` directly to generate raw JavaScript:

```php title="src/Service/CustomJavaScriptStrategy.php" linenums="1"
<?php

declare(strict_types=1);

namespace App\Service;

use SpomkyLabs\PwaBundle\CachingStrategy\CacheStrategyInterface;

final class CustomJavaScriptStrategy implements CacheStrategyInterface
{
    public function getName(): ?string
    {
        return 'my-custom-strategy';
    }

    public function isEnabled(): bool
    {
        return true;
    }

    public function needsWorkbox(): bool
    {
        return false; // Standalone JavaScript
    }

    public function render(string $cacheObjectName, bool $debug = false): string
    {
        return <<<'JS'
self.addEventListener('fetch', (event) => {
    if (event.request.url.includes('/custom/')) {
        event.respondWith(
            caches.match(event.request).then((cachedResponse) => {
                if (cachedResponse) {
                    fetch(event.request).then((response) => {
                        caches.open('custom-cache').then((cache) => {
                            cache.put(event.request, response);
                        });
                    });
                    return cachedResponse;
                }
                return fetch(event.request);
            })
        );
    }
});
JS;
    }
}
```

!!! warning

    When implementing the full interface, you're responsible for all JavaScript generation. This approach is powerful but requires careful testing.

## Service registration

If service autoconfiguration is disabled, register your service manually:

```yaml title="config/services.yaml" linenums="1"
services:
    App\Service\MyCustomCacheStrategies:
        tags:
            - { name: 'spomky_labs_pwa.cache_strategy' }
```

## Listing cache strategies

Use the console command to list all registered cache strategies:

```bash
php bin/console pwa:cache:list
```

This displays all strategies (from YAML config and custom services) with their configuration details.

## Debugging

### Check generated service worker

View the compiled service worker to see the generated JavaScript:

```bash
php bin/console pwa:compile
# Then inspect the generated sw.js file
```

When debug is enabled, the generated code includes comments explaining each strategy.

### Chrome DevTools

1. Open DevTools (F12) → **Application** → **Service Workers**
2. Click "Update" to reload service worker
3. Check **Application** → **Cache Storage** to see your custom caches
4. Use **Network** tab with "Offline" to test cache behavior

## Best practices

1. **Start with YAML configuration**: Only use custom strategies when necessary
2. **Use descriptive names**: Make cache names clear (e.g., `api-user-data` not `cache1`)
3. **Set appropriate limits**: Configure `maxEntries` and `maxAgeSeconds` to prevent cache bloat
4. **Test offline**: Verify strategies work correctly without network
5. **Document complex logic**: Add comments explaining why custom strategies exist

## Related documentation

- [Service Worker Configuration](../configuration.md) - Standard YAML configuration
- [Resource Caching](resource-caching.md) - Configure caching via YAML
- [Custom Service Worker Rules](../custom-service-worker-rule.md) - Advanced service worker customization

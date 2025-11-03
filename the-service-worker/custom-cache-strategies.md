# Custom Cache Strategies

The bundle provides a comprehensive set of cache strategies through configuration. However, for advanced use cases requiring specific behaviors not covered by the YAML configuration, you can create custom cache strategies programmatically.

## When to Use Custom Strategies

Use custom cache strategies when you need:

- **Dynamic behavior**: Cache strategies that change based on runtime conditions
- **Complex matching logic**: URL patterns too complex for regex or simple callbacks
- **Custom plugins**: Workbox plugins with specific configuration
- **Conditional caching**: Different strategies based on user state, headers, or other factors
- **Business logic integration**: Caching tied to your application's domain logic

For most use cases, the [YAML configuration](configuration.md) and [resource caching](workbox/resource-caching.md) are sufficient.

## Quick Start: Using WorkboxCacheStrategy

The easiest way to create custom cache strategies is using the `WorkboxCacheStrategy` helper class:

{% code title="src/Service/MyCustomCacheStrategies.php" lineNumbers="true" %}
```php
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
            // CacheFirst strategy for API data
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
{% endcode %}

{% hint style="info" %}
With service autoconfiguration enabled (default in Symfony), the tag `spomky_labs_pwa.cache_strategy` is added automatically. Otherwise, you need to tag your service manually.
{% endhint %}

## Complete Example: Multiple Strategies

A comprehensive example showing various cache strategies with different configurations:

{% code title="src/Service/AdvancedCacheStrategies.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Service;

use SpomkyLabs\PwaBundle\CachingStrategy\CacheStrategyInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\HasCacheStrategiesInterface;
use SpomkyLabs\PwaBundle\CachingStrategy\WorkboxCacheStrategy;
use SpomkyLabs\PwaBundle\WorkboxPlugin\ExpirationPlugin;
use SpomkyLabs\PwaBundle\WorkboxPlugin\CacheableResponsePlugin;
use SpomkyLabs\PwaBundle\WorkboxPlugin\BroadcastUpdatePlugin;

final readonly class AdvancedCacheStrategies implements HasCacheStrategiesInterface
{
    /**
     * @return array<CacheStrategyInterface>
     */
    public function getCacheStrategies(): array
    {
        return [
            // 1. Admin API - NetworkFirst with short cache
            $this->createAdminApiStrategy(),

            // 2. User-generated content - StaleWhileRevalidate with broadcast
            $this->createUserContentStrategy(),

            // 3. External CDN resources - CacheFirst with long expiration
            $this->createCdnStrategy(),

            // 4. Search results - NetworkFirst with tight cache
            $this->createSearchStrategy(),
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
            ->withMethod('POST')  // Only intercept POST requests
            ->withOptions(['networkTimeoutSeconds' => 2])
            ->withPlugin(
                // Only cache successful responses
                CacheableResponsePlugin::create(statuses: [200]),
                // Short cache duration
                ExpirationPlugin::create(
                    maxEntries: 20,
                    maxAgeSeconds: 300, // 5 minutes
                )
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
                // Notify clients when cache updates
                BroadcastUpdatePlugin::create([
                    'Content-Type',
                    'ETag',
                    'Last-Modified',
                ]),
                // Cache valid responses
                CacheableResponsePlugin::create(statuses: [0, 200]),
                // Reasonable expiration
                ExpirationPlugin::create(
                    maxEntries: 100,
                    maxAgeSeconds: 3600, // 1 hour
                )
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
                )
            );
    }

    private function createSearchStrategy(): CacheStrategyInterface
    {
        return WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
            matchCallback: '({url}) => url.pathname === "/search" && url.searchParams.has("q")',
        )
            ->withName('search-results')
            ->withOptions(['networkTimeoutSeconds' => 3])
            ->withPlugin(
                CacheableResponsePlugin::create(statuses: [200]),
                ExpirationPlugin::create(
                    maxEntries: 30,
                    maxAgeSeconds: 600, // 10 minutes
                )
            );
    }
}
```
{% endcode %}

## WorkboxCacheStrategy API

The `WorkboxCacheStrategy` class provides a fluent interface for building cache strategies:

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

**Available strategies**:
- `CacheStrategyInterface::STRATEGY_CACHE_FIRST` - Cache first, fallback to network
- `CacheStrategyInterface::STRATEGY_NETWORK_FIRST` - Network first, fallback to cache
- `CacheStrategyInterface::STRATEGY_STALE_WHILE_REVALIDATE` - Cache immediately, update in background
- `CacheStrategyInterface::STRATEGY_CACHE_ONLY` - Cache only, never network
- `CacheStrategyInterface::STRATEGY_NETWORK_ONLY` - Network only, never cache

### withName()

Set a custom cache name:

```php
->withName('my-custom-cache')
```

### withMethod()

Specify HTTP method to intercept (default: all methods):

```php
->withMethod('POST')  // Only POST requests
->withMethod('GET')   // Only GET requests
```

### withPlugin()

Add Workbox plugins to customize behavior:

```php
->withPlugin(
    ExpirationPlugin::create(maxEntries: 50, maxAgeSeconds: 3600),
    CacheableResponsePlugin::create(statuses: [200])
)
```

### withPreloadUrl()

Precache specific URLs when service worker installs:

```php
->withPreloadUrl('/critical-page', '/important-data.json')
```

### withOptions()

Set additional Workbox options:

```php
->withOptions([
    'networkTimeoutSeconds' => 3,  // For NetworkFirst/NetworkOnly
])
```

## Available Workbox Plugins

### ExpirationPlugin

Automatically remove old cache entries:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\ExpirationPlugin;

ExpirationPlugin::create(
    maxEntries: 50,        // Max number of entries
    maxAgeSeconds: 86400,  // Max age (1 day)
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

// Cache based on headers
CacheableResponsePlugin::create(
    statuses: [200],
    headers: ['X-Is-Cacheable' => 'true']
)
```

### BroadcastUpdatePlugin

Notify clients when cached data updates:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\BroadcastUpdatePlugin;

// Default headers
BroadcastUpdatePlugin::create()

// Custom headers to check
BroadcastUpdatePlugin::create([
    'Content-Type',
    'ETag',
    'Last-Modified',
    'X-Custom-Version',
])
```

Listen for updates in your application:

```javascript
// In your frontend code
const channel = new BroadcastChannel('workbox');
channel.addEventListener('message', (event) => {
    if (event.data.type === 'CACHE_UPDATED') {
        console.log('Cache updated:', event.data);
        // Reload data or notify user
    }
});
```

### BackgroundSyncPlugin

Queue failed requests for retry when online:

```php
use SpomkyLabs\PwaBundle\WorkboxPlugin\BackgroundSyncPlugin;

BackgroundSyncPlugin::create(
    queueName: 'api-queue',
    maxRetentionTime: 2880,      // 2 days in minutes
    forceSyncFallback: false,
    broadcastChannel: 'sync-updates',
)
```

## Match Callback Patterns

The `matchCallback` parameter supports several formats:

### 1. Path Prefix

```php
matchCallback: 'startsWith: /api/'
```

### 2. Regular Expression

```php
matchCallback: 'regex: /\\/product\\/[0-9]+/'
```

### 3. Origin Match

```php
matchCallback: 'origin: https://api.example.com'
```

### 4. Custom JavaScript Callback

```php
matchCallback: '({url, request}) => url.pathname.endsWith(".json") && request.method === "GET"'
```

### 5. Special Keywords

```php
matchCallback: 'navigate'  // Navigation requests (page loads)
```

## Advanced Example: Conditional Strategy

Create strategies that adapt based on conditions:

{% code title="src/Service/ConditionalCacheStrategy.php" lineNumbers="true" %}
```php
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
        #[Autowire(param: 'kernel.debug')]
        private bool $debug,
    ) {
    }

    public function getCacheStrategies(): array
    {
        $strategies = [];

        // Different strategies for production vs development
        if ($this->environment === 'prod') {
            // Aggressive caching in production
            $strategies[] = WorkboxCacheStrategy::create(
                enabled: true,
                requireWorkbox: true,
                strategy: CacheStrategyInterface::STRATEGY_CACHE_FIRST,
                matchCallback: '({url}) => url.pathname.startsWith("/static/")',
            )
                ->withName('static-production')
                ->withPlugin(
                    ExpirationPlugin::create(
                        maxEntries: 200,
                        maxAgeSeconds: 2592000, // 30 days
                    )
                );
        } else {
            // Minimal caching in development
            $strategies[] = WorkboxCacheStrategy::create(
                enabled: true,
                requireWorkbox: true,
                strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
                matchCallback: '({url}) => url.pathname.startsWith("/static/")',
            )
                ->withName('static-development')
                ->withOptions(['networkTimeoutSeconds' => 1])
                ->withPlugin(
                    ExpirationPlugin::create(
                        maxEntries: 10,
                        maxAgeSeconds: 60, // 1 minute
                    )
                );
        }

        return $strategies;
    }
}
```
{% endcode %}

## Real-World Use Cases

### Use Case 1: Multi-tenant Application

Different caching for each tenant:

```php
public function getCacheStrategies(): array
{
    // Assuming tenant ID is in URL like /tenant/123/...
    return [
        WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
            matchCallback: '({url}) => /\/tenant\/[0-9]+\/data/.test(url.pathname)',
        )
            ->withName('tenant-data')
            ->withPlugin(
                ExpirationPlugin::create(
                    maxEntries: 100,
                    maxAgeSeconds: 1800, // 30 minutes
                )
            ),
    ];
}
```

### Use Case 2: Authenticated API with Token Refresh

Cache API calls but refresh when needed:

```php
public function getCacheStrategies(): array
{
    return [
        WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
            matchCallback: <<<'JS'
({url, request}) => {
    return url.pathname.startsWith('/api/') &&
           request.headers.has('Authorization');
}
JS,
        )
            ->withName('authenticated-api')
            ->withMethod('GET')
            ->withOptions(['networkTimeoutSeconds' => 2])
            ->withPlugin(
                // Only cache successful responses
                CacheableResponsePlugin::create(statuses: [200]),
                // Notify on updates for real-time sync
                BroadcastUpdatePlugin::create(),
                // Short cache duration
                ExpirationPlugin::create(
                    maxEntries: 50,
                    maxAgeSeconds: 300,
                )
            ),
    ];
}
```

### Use Case 3: Versioned API Endpoints

Different strategies for stable vs unstable API versions:

```php
public function getCacheStrategies(): array
{
    return [
        // Stable API (v1) - aggressive caching
        WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_CACHE_FIRST,
            matchCallback: '({url}) => url.pathname.startsWith("/api/v1/")',
        )
            ->withName('api-v1-stable')
            ->withPlugin(
                ExpirationPlugin::create(
                    maxEntries: 100,
                    maxAgeSeconds: 86400, // 1 day
                )
            ),

        // Beta API (v2) - fresh data
        WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
            matchCallback: '({url}) => url.pathname.startsWith("/api/v2/")',
        )
            ->withName('api-v2-beta')
            ->withOptions(['networkTimeoutSeconds' => 2])
            ->withPlugin(
                ExpirationPlugin::create(
                    maxEntries: 30,
                    maxAgeSeconds: 300, // 5 minutes
                )
            ),
    ];
}
```

## Implementing Custom Strategy Interface

For complete control, implement `CacheStrategyInterface` directly:

{% code title="src/Service/CustomJavaScriptStrategy.php" lineNumbers="true" %}
```php
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
        return false;  // Standalone JavaScript
    }

    public function render(string $cacheObjectName, bool $debug = false): string
    {
        // Return raw JavaScript that will be injected into service worker
        return <<<'JS'
// Custom cache strategy with pure JavaScript
self.addEventListener('fetch', (event) => {
    if (event.request.url.includes('/custom/')) {
        event.respondWith(
            caches.match(event.request).then((cachedResponse) => {
                if (cachedResponse) {
                    // Custom logic: fetch in background if cached
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
{% endcode %}

{% hint style="warning" %}
When implementing the full interface, you're responsible for all JavaScript generation. This approach is powerful but requires careful testing.
{% endhint %}

## Service Registration

If service autoconfiguration is disabled, register your service manually:

{% code title="config/services.yaml" lineNumbers="true" %}
```yaml
services:
    App\Service\MyCustomCacheStrategies:
        tags:
            - { name: 'spomky_labs_pwa.cache_strategy' }
```
{% endcode %}

## Debugging Custom Strategies

### 1. Check Generated Service Worker

View the compiled service worker at `/sw.js` (or your configured dest) to see the generated JavaScript:

```bash
curl https://your-app.com/sw.js
```

### 2. Enable Debug Mode

Set `kernel.debug: true` to get commented JavaScript output:

```yaml
# config/packages/pwa.yaml
pwa:
    serviceworker:
        workbox:
            enabled: true
```

When debug is enabled, the generated code includes comments:

```javascript
/**************************************************** CACHE STRATEGY ****************************************************/
// Strategy: NetworkFirst
// Match: ({url}) => url.pathname.startsWith("/api/")
// Cache Name: admin-api
// ...
```

### 3. Chrome DevTools

1. Open DevTools (F12)
2. Go to **Application** → **Service Workers**
3. Click "Update" to reload service worker
4. Check **Application** → **Cache Storage** to see your custom caches
5. Use **Network** tab with "Offline" to test cache behavior

## Best Practices

1. **Start with YAML configuration**: Only use custom strategies when necessary
2. **Use descriptive names**: Make cache names clear (e.g., `api-user-data` not `cache1`)
3. **Set appropriate limits**: Configure `maxEntries` and `maxAgeSeconds` to prevent cache bloat
4. **Test offline**: Verify strategies work correctly without network
5. **Monitor cache size**: Check DevTools → Storage to ensure caches don't grow too large
6. **Use typed callbacks**: TypeScript-style types help prevent errors in matchCallback
7. **Document complex logic**: Add comments explaining why custom strategies exist

## Related Documentation

- [Configuration Reference](configuration.md) - Standard YAML configuration
- [Resource Caching](workbox/resource-caching.md) - Configure caching via YAML
- [Workbox Strategies](https://developers.google.com/web/tools/workbox/modules/workbox-strategies) - Official Workbox docs
- [Custom Service Worker Rules](custom-service-worker-rule.md) - Advanced service worker customization

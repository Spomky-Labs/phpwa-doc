# Custom Cache Strategy

Custom cache strategies allow you to programmatically define Workbox caching rules beyond the standard configuration options. This provides maximum flexibility for complex caching scenarios.

## Overview

While the PWA Bundle provides convenient configuration options for common caching patterns (asset caching, image caching, etc.), some applications require custom caching logic that goes beyond standard configuration. The bundle supports programmatic cache strategy creation through the `WorkboxCacheStrategy` class.

## When to Use Custom Cache Strategies

Use custom cache strategies when you need to:

1. **Dynamic route matching** - Cache routes based on complex logic
2. **Custom plugins** - Apply specialized Workbox plugins (ExpirationPlugin, BackgroundSyncPlugin, etc.)
3. **Per-route configuration** - Different strategies for different URL patterns
4. **Advanced caching logic** - Multi-tenant applications, authenticated APIs, conditional caching
5. **Runtime flexibility** - Strategies that change based on application state

## Quick Example

{% code title="src/ServiceWorker/CacheStrategy/ApiCacheStrategy.php" lineNumbers="true" %}
```php
<?php

namespace App\ServiceWorker\CacheStrategy;

use SpomkyLabs\PwaBundle\CachingStrategy\WorkboxCacheStrategy;
use SpomkyLabs\PwaBundle\WorkboxPlugin\ExpirationPlugin;
use Symfony\Component\DependencyInjection\Attribute\AsTaggedItem;

#[AsTaggedItem('spomky_labs_pwa.cache_strategy')]
final readonly class ApiCacheStrategy extends WorkboxCacheStrategy
{
    public static function create(): self
    {
        return WorkboxCacheStrategy::create(
            enabled: true,
            requireWorkbox: true,
            strategy: CacheStrategyInterface::STRATEGY_NETWORK_FIRST,
            matchCallback: '({url}) => url.pathname.startsWith("/api/")',
        )
            ->withName('api-cache')
            ->withPlugin(
                ExpirationPlugin::create(
                    maxEntries: 50,
                    maxAgeSeconds: 3600,
                )
            );
    }
}
```
{% endcode %}

This creates a cache strategy that:
- Uses NetworkFirst strategy for all `/api/` routes
- Limits the cache to 50 entries
- Expires cached responses after 1 hour

## Complete Documentation

For comprehensive documentation on creating custom cache strategies, including:

- **WorkboxCacheStrategy API** - Complete method reference
- **All Workbox Plugins** - ExpirationPlugin, BackgroundSyncPlugin, CacheableResponsePlugin, etc.
- **Strategy Types** - CacheFirst, NetworkFirst, StaleWhileRevalidate, NetworkOnly, CacheOnly
- **Real-World Examples** - Multi-tenant apps, authenticated APIs, conditional caching
- **Advanced Patterns** - Per-user caching, geo-based caching, A/B testing
- **Testing & Debugging** - Service worker inspection, cache verification

See the complete **[Custom Cache Strategies](../custom-cache-strategies.md)** documentation.

## Standard Workbox Configuration

For most applications, the standard Workbox configuration options are sufficient:

- **[Asset Caching](asset-caching.md)** - Cache CSS, JavaScript, and static assets
- **[Image Caching](image-caching.md)** - Optimize image caching with size/age limits
- **[Font Caching](font-caching.md)** - Cache web fonts for offline use
- **[Background Sync](backgoundsync.md)** - Retry failed requests in the background
- **[Offline Fallback](offline-fallbacks.md)** - Provide fallback pages when offline

## Related Documentation

- **[Custom Cache Strategies](../custom-cache-strategies.md)** - Complete programmatic cache strategy guide
- **[Workbox Configuration](README.md)** - Overview of all Workbox options
- **[Service Worker Configuration](../configuration.md)** - Service worker setup

## Resources

- **Workbox Strategies**: https://developer.chrome.com/docs/workbox/modules/workbox-strategies/
- **Workbox Plugins**: https://developer.chrome.com/docs/workbox/reference/workbox-core/
- **Workbox Routing**: https://developer.chrome.com/docs/workbox/modules/workbox-routing/

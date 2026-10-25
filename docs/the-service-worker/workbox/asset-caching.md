# Asset Caching

{% hint style="danger" %}
Contains a breaking change compared to v1.0.x
{% endhint %}

Asset caching provides automatic precaching and efficient serving of your application's static assets managed by Symfony's AssetMapper component. This ensures your CSS, JavaScript, images, and other static files are available offline and load instantly from the cache.

## How It Works

The asset cache feature automatically:

1. **Discovers assets**: Scans all assets registered with Symfony AssetMapper
2. **Filters by regex**: Only includes assets matching the regex pattern
3. **Precaches on install**: Downloads and caches all matching assets when the service worker installs
4. **Serves from cache**: Uses CacheFirst strategy for optimal performance
5. **Manages expiration**: Automatically removes old cached assets based on your configuration

## Default Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            asset_cache:
                enabled: true  # Enable asset caching
                regex: '/\.(css|js|json|xml|txt|map|ico|png|jpe?g|gif|svg|webp|bmp)$/'  # Default pattern
                cache_name: null  # Auto-generated: 'asset-cache'
                max_age: null  # Default: 1 year (31536000 seconds)
                max_entries: 60  # Maximum number of cached assets
```
{% endcode %}

By default, the cache includes all common asset types:
- **Stylesheets**: `.css`
- **Scripts**: `.js`
- **Data**: `.json`, `.xml`, `.txt`
- **Source maps**: `.map`
- **Icons**: `.ico`
- **Images**: `.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.webp`, `.bmp`

## Configuration Options

### enabled

**Type**: `boolean`
**Default**: `true`

Enable or disable asset caching entirely.

```yaml
asset_cache:
    enabled: false  # Disable asset caching
```

### regex

**Type**: `string` (regex pattern)
**Default**: `/\.(css|js|json|xml|txt|map|ico|png|jpe?g|gif|svg|webp|bmp)$/`

Regular expression to filter which assets should be cached. Only assets whose public path matches this pattern will be included.

**Examples**:

Cache only CSS and JavaScript:
```yaml
asset_cache:
    regex: '/\.(css|jsx?)$/'
```

Cache only images:
```yaml
asset_cache:
    regex: '/\.(png|jpe?g|gif|svg|webp)$/'
```

Cache everything:
```yaml
asset_cache:
    regex: '/.*/'
```

Exclude source maps:
```yaml
asset_cache:
    regex: '/\.(css|js|json|xml|txt|ico|png|jpe?g|gif|svg|webp|bmp)$/'
```

### cache_name

**Type**: `string|null`
**Default**: `null` (auto-generates "asset-cache")

Custom name for the cache. Useful for debugging and cache management.

```yaml
asset_cache:
    cache_name: 'my-app-assets'
```

### max_age

**Type**: `string|integer|null`
**Default**: `null` (1 year)

Maximum age for cached assets. Accepts human-readable format or seconds.

**Examples**:
```yaml
asset_cache:
    max_age: '30 days'     # Human-readable
    max_age: 2592000       # 30 days in seconds
    max_age: null          # Use default (1 year)
```

### max_entries

**Type**: `integer`
**Default**: `60`

Maximum number of asset entries to keep in cache. When exceeded, oldest entries are removed first. The actual limit is `max_entries * 2` to account for growth between cleanups.

```yaml
asset_cache:
    max_entries: 100  # Allow up to 100 cached assets
```

## Integration with AssetMapper

The asset cache works seamlessly with Symfony's AssetMapper:

### What Gets Cached

All assets registered through AssetMapper that match the regex pattern:

```php
// These assets will be automatically discovered and cached
// if they match the regex pattern
#[AssetMapper]
class MyController
{
    // app.css will be cached (matches .css)
    // app.js will be cached (matches .js)
}
```

### Asset Versioning

AssetMapper automatically versions assets (e.g., `app.a1b2c3.css`). When you update an asset:

1. New version gets a new filename
2. Service worker detects the change
3. New version is precached on next service worker update
4. Old version is removed from cache

### Public Path Resolution

The cache automatically uses the correct public path configured in AssetMapper:

```yaml
# config/packages/asset_mapper.yaml
framework:
    asset_mapper:
        public_prefix: '/build'  # Assets served from /build/
```

The service worker will cache all assets under `/build/` that match the regex.

## Caching Strategy

Asset caching uses the **CacheFirst** strategy:

```javascript
// Generated service worker code
workbox.routing.registerRoute(
    ({url}) => url.pathname.startsWith('/assets'),
    new workbox.strategies.CacheFirst({
        cacheName: 'asset-cache',
        plugins: [
            new workbox.expiration.ExpirationPlugin({
                maxEntries: 120,  // max_entries * 2
                maxAgeSeconds: 31536000,  // 1 year
            })
        ]
    })
);
```

**Why CacheFirst?**
- Static assets rarely change
- Versioned filenames ensure new versions are fetched
- Maximum performance: no network delay
- Works offline immediately

## Complete Examples

### Minimal Configuration

Use defaults for most applications:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            asset_cache:
                enabled: true  # Everything else uses defaults
```
{% endcode %}

### CSS and JavaScript Only

Cache only stylesheets and scripts:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            asset_cache:
                enabled: true
                regex: '/\.(css|jsx?)$/'
                cache_name: 'app-code'
                max_age: '7 days'
                max_entries: 30
```
{% endcode %}

### Large Application

Handle many assets with extended expiration:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            asset_cache:
                enabled: true
                regex: '/\.(css|js|json|xml|txt|map|ico|png|jpe?g|gif|svg|webp|bmp|woff2?)$/'
                cache_name: 'production-assets'
                max_age: '90 days'
                max_entries: 200  # Allow many cached files
```
{% endcode %}

### Separate Image Cache

Cache assets but handle images separately:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            asset_cache:
                enabled: true
                regex: '/\.(css|js|json|xml|txt|map)$/'  # No images
                cache_name: 'app-assets'
                max_age: '30 days'
                max_entries: 50

            image_cache:
                enabled: true  # Handle images separately with different settings
```
{% endcode %}

## Precaching Behavior

All matching assets are **precached** during service worker installation:

### Installation Process

1. Service worker installs
2. Asset cache scans AssetMapper for all assets
3. Filters assets by regex
4. Downloads all matching assets
5. Stores in cache
6. Service worker activates

### Impact on Install Time

Precaching all assets can increase service worker installation time. For large applications:

**Optimize by**:
- Use stricter regex to cache fewer assets
- Reduce max_entries
- Consider caching only critical assets

**Example**: Cache only critical CSS/JS, not images:
```yaml
asset_cache:
    regex: '/\.(css|js)$/'  # Faster install, images loaded on-demand
```

## Debugging Asset Cache

### View Cached Assets

1. Open DevTools (F12)
2. Go to **Application** → **Cache Storage**
3. Look for "asset-cache" (or your custom cache name)
4. Inspect cached files and their versions

### Check What Will Be Cached

To see which assets match your regex before deploying:

```bash
# List all AssetMapper assets
php bin/console debug:asset-mapper

# Filter by your regex pattern manually
php bin/console debug:asset-mapper | grep -E "\.(css|js)$"
```

### Common Issues

**Assets not caching?**
- Verify regex pattern matches asset paths
- Check AssetMapper is configured correctly
- Ensure service worker is active (not waiting)

**Too many assets being cached?**
- Make regex more specific
- Reduce max_entries
- Use separate caches for different asset types

**Old assets not clearing?**
- Check max_age configuration
- Verify automatic cache clearing is enabled (`clear_cache: true`)

## Asset Cache vs Image Cache

Both cache assets, but serve different purposes:

| Feature | Asset Cache | Image Cache |
|---------|-------------|-------------|
| **Purpose** | Application assets (CSS, JS, etc.) | Images specifically |
| **Strategy** | CacheFirst | CacheFirst |
| **Precaching** | Yes - all matching assets | No - caches on first request |
| **AssetMapper** | Uses AssetMapper to find assets | Matches any image request |
| **Use Case** | App code and resources | User-generated or dynamic images |

**When to use both**:
```yaml
asset_cache:
    regex: '/\.(css|js|json|xml)$/'  # App code - precached

image_cache:
    enabled: true  # Images - cached on demand
```

## Best Practices

1. **Start with defaults**: Only customize if you have specific needs
2. **Monitor cache size**: Check DevTools during development
3. **Be selective**: Don't cache assets you don't need offline
4. **Version assets**: Use AssetMapper's versioning (automatic)
5. **Test offline**: Verify cached assets load without network
6. **Separate concerns**: Use image_cache for images, asset_cache for code
7. **Set appropriate max_age**: Balance freshness vs. cache hits

## Troubleshooting

**Service worker not installing?**
- Too many assets to precache
- Network timeout during install
- **Solution**: Reduce assets with stricter regex

**Assets loading old versions?**
- AssetMapper not versioning
- Cache not clearing
- **Solution**: Ensure AssetMapper versioning is enabled

**Cache growing too large?**
- max_entries too high
- max_age too long
- **Solution**: Reduce both values

## Related Documentation

- [Image Caching](image-caching.md) - Dedicated image caching
- [Font Caching](font-caching.md) - Font-specific caching
- [Cache Management](cache-names-and-purge.md) - Cache lifecycle
- [Resource Caching](resource-caching.md) - Custom caching for any resource

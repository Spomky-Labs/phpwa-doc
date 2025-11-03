# Cache Management

Proper cache management ensures your PWA doesn't consume excessive storage and old data doesn't persist after updates. The bundle provides automatic cache cleaning and allows custom cache naming for better organization.

## Automatic Cache Cleaning

### Default Behavior

When enabled (default), the service worker automatically purges old caches during activation. This happens when:
- A new service worker version is installed
- The service worker activates
- Cache keys don't match current cache names

**Benefits**:
- Prevents unlimited cache growth
- Removes outdated content
- Frees device storage
- Avoids browser storage limits

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            clear_cache: true  # Default - recommended
```
{% endcode %}

### How It Works

1. **Service worker updates**: New version detected
2. **Installation**: New service worker installs in background
3. **Activation**: Old service worker replaced
4. **Cache cleanup**: Old cache versions deleted automatically
5. **Fresh start**: Only current caches remain

### Disabling Cache Cleaning

You can disable automatic cleaning if you want manual control:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            clear_cache: false
```
{% endcode %}

{% hint style="warning" %}
**Important**: When `clear_cache: false`, you must implement your own cache management to prevent unlimited storage growth. This can lead to:
- Device storage exhaustion
- Browser cache limits being hit
- Application errors
- Poor performance
{% endhint %}

### When to Disable

Only disable automatic cleaning when:
- Implementing custom cache versioning
- Using manual cache management
- Testing cache behavior
- Specific use case requires persistent old caches

Most applications should keep `clear_cache: true`.

## Custom Cache Names

Every cache type in Workbox can have a custom name for better organization and debugging.

### Default Cache Names

When not specified, the bundle generates cache names automatically:

| Cache Type | Default Name Pattern |
|-----------|---------------------|
| Images | `image-cache` |
| Fonts | `font-cache` |
| Assets | `asset-cache` |
| Resources | `resource-cache-{index}` |
| Google Fonts CSS | `google-fonts-stylesheets` |
| Google Fonts Files | `google-fonts-webfonts` |
| Offline Fallbacks | `offline-fallback` |

### Why Use Custom Names?

- **Easier debugging**: Quickly identify caches in DevTools
- **Multiple caches**: Separate similar content (e.g., user photos vs product images)
- **Team coordination**: Standardized naming across projects
- **Monitoring**: Track cache usage by type

### Setting Custom Names

#### Image Cache

```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                cache_name: 'product-images'
```

#### Font Cache

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                cache_name: 'brand-fonts'
```

#### Resource Caches

```yaml
pwa:
    serviceworker:
        workbox:
            resource_caches:
                - match_callback: 'navigate'
                  cache_name: 'pages'
                  strategy: 'NetworkFirst'

                - match_callback: 'startsWith: /api/'
                  cache_name: 'api-responses'
                  strategy: 'NetworkFirst'
```

#### Offline Fallbacks

```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                cache_name: 'offline-content'
                page: 'app_offline'
```

#### Google Fonts

```yaml
pwa:
    serviceworker:
        workbox:
            google_fonts:
                cache_prefix: 'gfonts'
                # Creates: gfonts-stylesheets and gfonts-webfonts
```

## Complete Example

Organized cache structure with custom names:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            clear_cache: true  # Automatic cleanup

            # Asset caches
            image_cache:
                enabled: true
                cache_name: 'app-images'

            font_cache:
                enabled: true
                cache_name: 'app-fonts'

            # Google Fonts
            google_fonts:
                enabled: true
                cache_prefix: 'google'

            # Resource caches
            resource_caches:
                - match_callback: 'navigate'
                  cache_name: 'html-pages'
                  strategy: 'NetworkFirst'

                - match_callback: 'startsWith: /api/'
                  cache_name: 'api-data'
                  strategy: 'NetworkFirst'
                  max_age: 5 minutes

            # Offline fallback
            offline_fallback:
                cache_name: 'offline-fallbacks'
                page: 'app_offline'
                image: 'images/offline.svg'
```
{% endcode %}

## Cache Inspection

### Using DevTools

1. Open DevTools (F12)
2. Go to **Application** tab
3. Expand **Cache Storage** in left sidebar
4. View all caches and their contents

You'll see your custom cache names, making it easy to:
- Verify cache contents
- Check cache sizes
- Debug caching issues
- Delete specific caches for testing

### Cache Contents

Click a cache name to see:
- **Request URL**: What's cached
- **Response**: Cache entry details
- **Headers**: Cache metadata
- **Preview**: Content preview

### Manual Cache Management

For testing or debugging, you can manually clear caches:

```javascript
// Clear all caches
caches.keys().then(keys => {
    keys.forEach(key => caches.delete(key));
});

// Clear specific cache
caches.delete('app-images');

// Clear caches matching pattern
caches.keys().then(keys => {
    keys.filter(key => key.startsWith('api-'))
        .forEach(key => caches.delete(key));
});
```

## Best Practices

1. **Use descriptive names**: `product-images` > `cache-1`
2. **Group by purpose**: Separate user content from app assets
3. **Enable auto-cleanup**: Keep `clear_cache: true` for production
4. **Monitor size**: Check DevTools regularly during development
5. **Version caches**: Include version in name for migration (e.g., `images-v2`)
6. **Document structure**: Maintain a list of cache names and purposes

## Cache Size Limits

Browsers impose storage limits that vary by:
- Device type (mobile vs desktop)
- Available storage
- Browser implementation

**Typical limits**:
- **Chrome**: ~60% of free disk space
- **Firefox**: ~50% of free disk space
- **Safari**: ~1GB on mobile, more on desktop

When limits are reached:
- Browser may delete caches (least recently used first)
- New cache operations may fail
- Application may not work offline

## Storage Quota API

Check available storage in your application:

```javascript
if ('storage' in navigator && 'estimate' in navigator.storage) {
    navigator.storage.estimate().then(estimate => {
        const usedMB = (estimate.usage / 1024 / 1024).toFixed(2);
        const quotaMB = (estimate.quota / 1024 / 1024).toFixed(2);
        const percentUsed = ((estimate.usage / estimate.quota) * 100).toFixed(2);

        console.log(`Storage used: ${usedMB} MB of ${quotaMB} MB (${percentUsed}%)`);
    });
}
```

## Troubleshooting

**Caches not clearing on update?**
- Verify `clear_cache: true`
- Check service worker is activating (not waiting)
- Ensure cache names have changed if using versioned names
- Close all tabs and reopen to force activation

**Running out of storage?**
- Reduce `max_entries` for each cache type
- Decrease `max_age` to expire entries sooner
- Enable `clear_cache` if disabled
- Review what's being cached (too many large files?)

**Old caches persisting?**
- Service worker may not have activated
- Check for `skipWaiting` configuration
- Manually delete in DevTools
- Check for custom cache management logic preventing deletion

**Can't find cache in DevTools?**
- Verify cache name spelling in configuration
- Check service worker is registered and active
- Look in **Application** → **Cache Storage** (not **Storage**)
- Ensure service worker has run at least once

{% hint style="success" %}
**Recommended Setup**: Use automatic cache cleaning (`clear_cache: true`) with descriptive custom cache names. This provides the best balance of automatic management and debuggability.
{% endhint %}

## Migration Strategy

When changing cache structures:

1. **Update cache names**: Increment version or change name
2. **Deploy new service worker**: With updated names
3. **Old caches auto-deleted**: When new SW activates
4. **User re-downloads**: Content on next visit
5. **Smooth transition**: No manual intervention needed

Example version increment:
```yaml
# Old configuration
image_cache:
    cache_name: 'images-v1'

# New configuration
image_cache:
    cache_name: 'images-v2'
```

Workbox automatically handles the migration.

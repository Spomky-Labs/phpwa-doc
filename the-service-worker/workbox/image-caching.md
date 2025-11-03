# Image Caching

Image caching optimizes the delivery of images in your PWA by storing them locally in the browser. This feature uses the **CacheFirst** strategy, meaning images are served from the cache if available, with the network as a fallback.

## Why Cache Images?

Images are often the largest assets on web pages. Caching them provides:
- **Faster page loads**: Images load instantly from cache
- **Reduced bandwidth**: Same images aren't downloaded multiple times
- **Offline access**: Images remain available without network connection
- **Better user experience**: No loading spinners or broken images

## Default Configuration

By default, the bundle caches images with these settings:
- **Max entries**: 60 images
- **Max age**: 365 days (1 year)
- **Strategy**: CacheFirst
- **Supported formats**: `.ico`, `.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.webp`, `.bmp`

**Pattern matched**:
```regex
/\.(ico|png|jpe?g|gif|svg|webp|bmp)$/
```

## Asset Cache vs Image Cache

{% hint style="info" %}
**Important distinction**:
- **Asset Cache**: Images managed by Symfony's Asset Mapper (precached during service worker installation)
- **Image Cache**: Runtime caching for images NOT in Asset Mapper (external images, user uploads, CDN images)

Both caches work together - Asset Mapper images are precached, while other images are cached on first request.
{% endhint %}

## Configuration Options

### Enabling/Disabling

```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                enabled: true  # Default: true
```

### Custom Regex Pattern

Customize which image URLs are cached by modifying the regex pattern:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                enabled: true
                regex: '/\.(png|jpe?g|svg|webp)$/'  # Only these formats
```
{% endcode %}

**Use cases**:
- Limit to specific formats (e.g., only PNG and JPG)
- Match specific URL patterns (e.g., `/\/images\/.*\.(png|jpg)$/`)
- Exclude certain paths

### Max Entries

**Type**: `integer`
**Default**: `60`

Maximum number of images to store in the cache. When the limit is reached, the least recently used images are removed.

```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                max_entries: 200  # Cache up to 200 images
```

**Recommendations**:
- **Small apps**: 30-60 images
- **Medium apps**: 100-200 images
- **Large apps**: 200-500 images
- **Image-heavy apps**: 500+ images

Consider your users' storage and typical image count.

### Max Age

**Type**: `integer` (seconds) or `string` (human-readable)
**Default**: `31536000` (365 days)

How long images remain in the cache before being refreshed.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                max_age: 2592000      # 30 days in seconds
                # or
                max_age: 30 days      # Human-readable format
                # or
                max_age: 6 months     # Also supported
```
{% endcode %}

**Human-readable format examples**:
- `1 hour`, `2 hours`
- `1 day`, `7 days`, `30 days`
- `1 week`, `2 weeks`
- `1 month`, `6 months`
- `1 year`

**Choosing the right max age**:
- **Static branding images**: 1 year or more
- **Product photos**: 1-3 months
- **User avatars**: 1-7 days
- **Frequently changing content**: 1 day or less

### Cache Name

**Type**: `string`
**Default**: `null` (auto-generated)

Customize the cache storage name for organization and debugging.

```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                cache_name: 'product-images'
```

**Benefits of custom names**:
- Easier debugging in DevTools
- Separate caches for different image types
- Better cache management

## Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            image_cache:
                enabled: true
                cache_name: 'app-images'
                max_entries: 150
                max_age: 90 days
                regex: '/\.(png|jpe?g|svg|webp)$/'
```
{% endcode %}

## Use Cases

### E-commerce Site

Cache product images with moderate retention:

```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                enabled: true
                cache_name: 'product-images'
                max_entries: 300
                max_age: 30 days
```

### Image Gallery App

Prioritize capacity with shorter retention:

```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                enabled: true
                cache_name: 'gallery-images'
                max_entries: 500
                max_age: 7 days
```

### Blog with Static Images

Long retention for rarely changing images:

```yaml
pwa:
    serviceworker:
        workbox:
            image_cache:
                enabled: true
                max_entries: 100
                max_age: 1 year
```

## How It Works

1. **First request**: Image is fetched from network and stored in cache
2. **Subsequent requests**: Image is served instantly from cache (CacheFirst strategy)
3. **Cache miss**: If image not in cache or expired, network request is made
4. **Cache limit**: When `max_entries` is reached, oldest images are removed
5. **Expiration**: After `max_age`, images are refreshed on next request

## Debugging

To inspect the image cache in Chrome DevTools:

1. Open DevTools (F12)
2. Go to **Application** tab
3. Expand **Cache Storage**
4. Look for your cache (default or custom `cache_name`)
5. View cached images and their metadata

## Best Practices

1. **Balance capacity and storage**: Don't cache excessive images on mobile devices
2. **Use appropriate max_age**: Match your content update frequency
3. **Monitor cache size**: Check DevTools to ensure cache isn't growing too large
4. **Consider user storage**: Mobile users have limited space
5. **Use custom cache names**: Makes debugging and management easier

{% hint style="success" %}
The default configuration works well for most applications. Start with defaults and adjust based on your specific needs and user feedback.
{% endhint %}

# Workbox

[Workbox](https://developer.chrome.com/docs/workbox/) is a set of libraries and tools from Google that makes service worker creation easier and more reliable. It provides production-ready caching strategies, request routing, and offline support with minimal configuration.

## Why Workbox?

Writing service workers from scratch is complex and error-prone. Common challenges include:
- Managing multiple caching strategies
- Handling cache updates and versioning
- Implementing offline fallbacks
- Ensuring consistent behavior across browsers

Workbox solves these problems by providing:
- Battle-tested caching patterns
- Automatic cache management
- Simple, declarative configuration
- Extensive plugin ecosystem

## How This Bundle Uses Workbox

The PWA Bundle integrates Workbox deeply into Symfony, allowing you to:
- Configure caching strategies in YAML
- Use Symfony routes in cache patterns
- Generate service workers automatically
- Leverage Asset Mapper for asset precaching

You don't need to write JavaScript - just configure your needs in `pwa.yaml`.

## Core Concepts

### Precaching (Warm Cache)

Precaching loads resources during service worker installation, before users request them. This creates a "warm cache" with instant access.

**Benefits**:
* **Faster Load Times**: Assets load instantly from local cache
* **Offline Support**: Application works without network connection
* **Consistency**: All users get the same asset versions
* **Background Updates**: New versions update without interrupting users

**What to Precache**:
- Essential application shell (HTML, CSS, JavaScript)
- Critical images and fonts
- Static content users will definitely access
- Core functionality pages (homepage, pricing, about)

### Runtime Caching

Runtime caching handles requests as they occur, using different strategies based on content type and freshness requirements.

**Available Strategies**:
- **CacheFirst**: Use cache, fall back to network (static assets)
- **NetworkFirst**: Try network, fall back to cache (dynamic content)
- **StaleWhileRevalidate**: Serve cache immediately, update in background
- **NetworkOnly**: Always use network (user-specific data)
- **CacheOnly**: Only use cache (fully offline)

See [Resource Caching](resource-caching.md) for detailed strategy documentation.

## Configuration

### Basic Configuration

Workbox is enabled by default when you enable the service worker:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        # Workbox is automatically enabled
```
{% endcode %}

### Disabling Workbox

If you want to write a custom service worker without Workbox:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: false
```
{% endcode %}

{% hint style="warning" %}
Disabling Workbox removes all automatic caching, offline support, and related features. You'll need to implement everything manually in your service worker.
{% endhint %}

### CDN vs Local Installation

**Type**: `boolean` (`use_cdn`)
**Default**: `false`

Workbox can be loaded from a CDN or served locally via Asset Mapper.

#### Local Installation (Recommended)

**Benefits**:
- Faster loading (same origin)
- Works offline immediately
- No external dependencies
- Better privacy and security
- Controlled versioning

**Configuration**:
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            enabled: true
            use_cdn: false  # Default - recommended
```
{% endcode %}

#### CDN Installation

**When to use**:
- Quick prototyping
- Bandwidth concerns
- Don't want to manage Workbox updates

**Configuration**:
{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            enabled: true
            use_cdn: true
```
{% endcode %}

{% hint style="info" %}
When using local installation, Workbox files are automatically served by the bundle. You don't need to manually install or configure anything.
{% endhint %}

### Workbox Version

**Type**: `string`
**Default**: `"7.3.0"`

Specify which version of Workbox to use. This applies to both CDN and local installations.

```yaml
pwa:
    serviceworker:
        workbox:
            version: "7.3.0"  # Default
            # or
            version: "7.0.0"   # Specific version
```

{% hint style="warning" %}
Changing the Workbox version may introduce breaking changes. Refer to the [Workbox release notes](https://github.com/GoogleChrome/workbox/releases) before upgrading.
{% endhint %}

### Public URLs

**Workbox Public URL**
**Type**: `string`
**Default**: `"/workbox"`

URL path where Workbox files are served (when `use_cdn: false`).

**IndexDB Public URL**
**Type**: `string`
**Default**: `"/idb"`

URL path for IndexedDB helper files.

```yaml
pwa:
    serviceworker:
        workbox:
            workbox_public_url: "/assets/workbox"
            idb_public_url: "/assets/idb"
```

### Cache Manifest

**Type**: `boolean`
**Default**: `true`

Controls whether Asset Mapper assets are automatically precached.

```yaml
pwa:
    serviceworker:
        workbox:
            cache_manifest: true  # Precache all Asset Mapper assets
```

When enabled, all assets managed by Symfony's Asset Mapper are automatically added to the precache list. This typically includes:
- JavaScript files
- CSS stylesheets
- Images
- Fonts
- Other static assets

### Clear Cache

**Type**: `boolean`
**Default**: `true`

Automatically removes old cache versions on service worker activation.

```yaml
pwa:
    serviceworker:
        workbox:
            clear_cache: true  # Clean up old caches (recommended)
```

When enabled, outdated caches are deleted when a new service worker activates, preventing storage bloat.

## Complete Workbox Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: true
            use_cdn: false
            version: "7.3.0"
            workbox_public_url: "/workbox"
            idb_public_url: "/idb"
            cache_manifest: true
            clear_cache: true

            # Feature-specific configuration
            image_cache:
                enabled: true
            font_cache:
                enabled: true
            resource_caches:
                - match_callback: 'navigate'
                  strategy: 'NetworkFirst'
            # ... more features
```
{% endcode %}

## Available Features

The Workbox integration provides several specialized caching features:

- **[Asset Caching](asset-caching.md)**: Automatic Asset Mapper integration
- **[Resource Caching](resource-caching.md)**: Page and API caching with strategies
- **[Image Caching](image-caching.md)**: Optimized image caching
- **[Font Caching](font-caching.md)**: Font file caching
- **[Offline Fallback](offline-fallback.md)**: Fallback pages for offline scenarios
- **[Background Sync](backgoundsync.md)**: Retry failed requests
- **[Cache Management](cache-names-and-purge.md)**: Cache naming and cleanup
- **[Custom Strategies](custom-cache-strategy.md)**: Build your own caching logic

## Next Steps

1. **Learn the basics**: Read about [caching strategies](resource-caching.md#strategy-parameter)
2. **Configure assets**: Set up [asset caching](asset-caching.md) for your static files
3. **Add offline support**: Configure [offline fallbacks](offline-fallback.md)
4. **Optimize performance**: Implement [image](image-caching.md) and [font](font-caching.md) caching

{% hint style="success" %}
For most applications, the default Workbox configuration works great. Start simple and add features as needed.
{% endhint %}

# Complete Example

This page provides comprehensive examples of Service Worker configurations, from basic to advanced scenarios. Use these as starting points for your own PWA implementation.

## Quick Start: Minimal Configuration

The simplest configuration that enables basic offline functionality:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest: "/manifest.json"
    serviceworker: "sw.js"  # Create an empty assets/sw.js file
```
{% endcode %}

This gives you:
- Service worker registration
- Asset precaching (CSS, JS, images from AssetMapper)
- Image caching (on-demand)
- Font caching (on-demand)
- Page caching with NetworkFirst strategy
- Automatic cache management

{% hint style="info" %}
The `sw.js` file in your `/assets/` folder can be empty. The bundle generates all the necessary code automatically.
{% endhint %}

## Intermediate: Custom Cache Settings

Enhanced configuration with customized caching behavior:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest: "/manifest.json"

    serviceworker:
        enabled: true
        src: "sw.js"
        dest: "/sw.js"
        scope: "/"
        skip_waiting: true  # Auto-update service worker
        use_cache: true

        workbox:
            enabled: true
            use_cdn: false  # Use local Workbox files
            version: "7.3.0"

            # Clear old caches on activation
            clear_cache: true

            # Asset cache (CSS, JS, fonts from AssetMapper)
            asset_cache:
                enabled: true
                regex: '/\.(css|js|json|xml|txt|map)$/'  # No images
                cache_name: 'app-assets'
                max_age: '30 days'
                max_entries: 60

            # Image cache (all image requests)
            image_cache:
                enabled: true
                regex: '/\.(png|jpe?g|gif|svg|webp|bmp)$/'
                cache_name: 'images'
                max_age: '7 days'
                max_entries: 100

            # Font cache
            font_cache:
                enabled: true
                regex: '/\.(ttf|eot|otf|woff2?)$/'
                cache_name: 'fonts'
                max_age: '90 days'
                max_entries: 30

            # Offline fallback pages
            offline_fallback:
                cache_name: 'offline'
                page: '/offline'  # Route name
                image: '/images/offline.svg'
                font: false  # No font fallback
```
{% endcode %}

## Advanced: Full-Featured PWA

Complete configuration showcasing all major features:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest: "/manifest.json"

    serviceworker:
        enabled: true
        src: "sw.js"
        dest: "/sw.js"
        scope: "/"
        skip_waiting: true
        use_cache: true

        workbox:
            enabled: true
            use_cdn: false
            version: "7.3.0"
            workbox_public_url: "/workbox"
            idb_public_url: "/idb"
            clear_cache: true

            # ===== ASSET CACHING =====
            asset_cache:
                enabled: true
                regex: '/\.(css|js|json|xml|txt|map)$/'
                cache_name: 'production-assets'
                max_age: '30 days'
                max_entries: 100

            # ===== IMAGE CACHING =====
            image_cache:
                enabled: true
                regex: '/\.(ico|png|jpe?g|gif|svg|webp|bmp)$/'
                cache_name: 'production-images'
                max_age: '7 days'
                max_entries: 150

            # ===== FONT CACHING =====
            font_cache:
                enabled: true
                regex: '/\.(ttf|eot|otf|woff2?)$/'
                cache_name: 'production-fonts'
                max_age: '90 days'
                max_entries: 50

            # ===== GOOGLE FONTS =====
            google_fonts:
                enabled: true
                cache_prefix: 'google-fonts'
                max_age: '1 year'
                max_entries: 30

            # ===== OFFLINE FALLBACKS =====
            offline_fallback:
                cache_name: 'offline-fallbacks'
                page: '/offline'
                image: '/images/offline-placeholder.svg'
                font: false

            # ===== RESOURCE CACHING =====
            resource_caches:
                # Page navigation - NetworkFirst
                - match_callback: 'navigate'
                  strategy: 'NetworkFirst'
                  cache_name: 'pages'
                  network_timeout: 3
                  max_age: '1 hour'
                  max_entries: 50

                # API calls - NetworkFirst with short timeout
                - match_callback: 'startsWith: /api/'
                  strategy: 'NetworkFirst'
                  cache_name: 'api-calls'
                  network_timeout: 2
                  max_age: '5 minutes'
                  max_entries: 100

                # CDN resources - CacheFirst
                - match_callback: 'origin: https://cdn.example.com'
                  strategy: 'CacheFirst'
                  cache_name: 'cdn-resources'
                  max_age: '30 days'
                  max_entries: 200

                # Static data - StaleWhileRevalidate
                - match_callback: 'regex: /\/(data|static)\//'
                  strategy: 'StaleWhileRevalidate'
                  cache_name: 'static-data'
                  max_age: '1 day'
                  max_entries: 50

            # ===== BACKGROUND SYNC =====
            background_sync:
                # Form submissions
                - queue_name: 'form-submissions'
                  match_callback: 'startsWith: /submit/'
                  method: POST
                  max_retention_time: 2880  # 2 days (in minutes)
                  broadcast_channel: 'form-sync'

                # API mutations
                - queue_name: 'api-mutations'
                  match_callback: 'regex: /\/api\/(create|update|delete)/'
                  method: POST
                  max_retention_time: 4320  # 3 days
                  force_sync_fallback: true
                  broadcast_channel: 'api-sync'

                # Contact form
                - queue_name: 'contact'
                  match_callback: 'regex: /\/contact\//''
                  method: POST
                  max_retention_time: 1440  # 1 day
                  broadcast_channel: 'contact-sync'
```
{% endcode %}

{% hint style="success" %}
This configuration provides:
- Complete offline support
- Optimized caching for different resource types
- Network resilience with background sync
- Automatic updates with skip_waiting
- Multiple caching strategies for different use cases
{% endhint %}

## E-commerce Application Example

Optimized for online stores with product images and dynamic content:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest: "/manifest.json"

    serviceworker:
        enabled: true
        src: "sw.js"
        skip_waiting: true

        workbox:
            enabled: true

            # High capacity for product images
            image_cache:
                enabled: true
                cache_name: 'product-images'
                max_age: '3 days'
                max_entries: 300  # Many product images

            # Product pages - fresh content
            resource_caches:
                # Product pages
                - match_callback: 'regex: /\/product\//'
                  strategy: 'NetworkFirst'
                  cache_name: 'products'
                  network_timeout: 2
                  max_age: '1 hour'
                  max_entries: 100

                # Category pages
                - match_callback: 'regex: /\/category\//'
                  strategy: 'NetworkFirst'
                  cache_name: 'categories'
                  network_timeout: 2
                  max_age: '30 minutes'
                  max_entries: 50

                # Search results
                - match_callback: 'startsWith: /search'
                  strategy: 'NetworkFirst'
                  cache_name: 'search'
                  network_timeout: 3
                  max_age: '10 minutes'
                  max_entries: 30

            # Background sync for cart and checkout
            background_sync:
                - queue_name: 'cart-updates'
                  match_callback: 'startsWith: /cart/'
                  method: POST
                  max_retention_time: 1440  # 1 day
                  broadcast_channel: 'cart-sync'

                - queue_name: 'wishlist'
                  match_callback: 'regex: /\/wishlist\//'
                  method: POST
                  max_retention_time: 2880  # 2 days

            offline_fallback:
                page: '/offline-shop'
                image: '/images/product-placeholder.svg'
```
{% endcode %}

## Content/Blog Application Example

Optimized for content-heavy sites with articles and media:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest: "/manifest.json"

    serviceworker:
        enabled: true
        src: "sw.js"
        skip_waiting: true

        workbox:
            enabled: true

            # Asset caching with long expiration
            asset_cache:
                enabled: true
                cache_name: 'blog-assets'
                max_age: '60 days'
                max_entries: 100

            # Featured images
            image_cache:
                enabled: true
                cache_name: 'article-images'
                max_age: '14 days'
                max_entries: 200

            # Google Fonts for typography
            google_fonts:
                enabled: true
                cache_prefix: 'blog-fonts'
                max_age: '1 year'
                max_entries: 20

            resource_caches:
                # Article pages - StaleWhileRevalidate for best UX
                - match_callback: 'regex: /\/(article|post)\//'
                  strategy: 'StaleWhileRevalidate'
                  cache_name: 'articles'
                  max_age: '7 days'
                  max_entries: 100

                # Homepage and category listings
                - match_callback: 'regex: /\/(home|category)\//'
                  strategy: 'NetworkFirst'
                  cache_name: 'listings'
                  network_timeout: 3
                  max_age: '1 hour'
                  max_entries: 30

                # Comments API
                - match_callback: 'startsWith: /api/comments'
                  strategy: 'NetworkFirst'
                  cache_name: 'comments'
                  network_timeout: 2
                  max_age: '10 minutes'
                  max_entries: 50

            # Background sync for comments
            background_sync:
                - queue_name: 'comments'
                  match_callback: 'regex: /\/api\/comments\/create/'
                  method: POST
                  max_retention_time: 2880  # 2 days
                  broadcast_channel: 'comment-sync'

            offline_fallback:
                page: '/offline-articles'
                image: '/images/article-placeholder.svg'
```
{% endcode %}

## SaaS/Dashboard Application Example

Optimized for real-time data and user interactions:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest: "/manifest.json"

    serviceworker:
        enabled: true
        src: "sw.js"
        skip_waiting: true

        workbox:
            enabled: true

            # Minimal asset precaching
            asset_cache:
                enabled: true
                regex: '/\.(css|js)$/'  # Only critical assets
                cache_name: 'app-core'
                max_age: '7 days'
                max_entries: 50

            # Small image cache (UI icons only)
            image_cache:
                enabled: true
                regex: '/\/icons?\/.*\.(png|svg)$/'  # Only UI icons
                cache_name: 'ui-icons'
                max_age: '30 days'
                max_entries: 30

            resource_caches:
                # Dashboard data - short cache
                - match_callback: 'startsWith: /api/dashboard'
                  strategy: 'NetworkFirst'
                  cache_name: 'dashboard-data'
                  network_timeout: 2
                  max_age: '1 minute'  # Fresh data
                  max_entries: 10

                # User data - NetworkFirst
                - match_callback: 'regex: /\/api\/user\//'
                  strategy: 'NetworkFirst'
                  cache_name: 'user-data'
                  network_timeout: 2
                  max_age: '5 minutes'
                  max_entries: 20

                # Static reference data - CacheFirst
                - match_callback: 'startsWith: /api/reference'
                  strategy: 'CacheFirst'
                  cache_name: 'reference-data'
                  max_age: '1 day'
                  max_entries: 50

            # Background sync for all mutations
            background_sync:
                - queue_name: 'data-sync'
                  match_callback: 'regex: /\/api\/.*\/(create|update|delete)/'
                  method: POST
                  max_retention_time: 4320  # 3 days
                  force_sync_fallback: true
                  broadcast_channel: 'app-sync'

            offline_fallback:
                page: '/offline-dashboard'
```
{% endcode %}

## Testing Your Configuration

After implementing your service worker configuration, test it thoroughly:

### 1. Development Testing

```bash
# Clear cache before testing
php bin/console cache:clear

# Compile assets
php bin/console asset-map:compile

# Start development server
symfony serve
```

### 2. Chrome DevTools Inspection

1. Open DevTools (F12)
2. Go to **Application** tab
3. Check:
   - **Service Workers**: Should show "activated and is running"
   - **Cache Storage**: Verify all configured caches exist
   - **Network**: Set to Offline and test navigation

### 3. Lighthouse Audit

Run a PWA audit to verify your configuration:

1. Open DevTools → Lighthouse
2. Select "Progressive Web App"
3. Click "Generate report"
4. Verify:
   - Service worker registered
   - Offline support working
   - Caching configured properly

## Common Configuration Patterns

### Pattern 1: Aggressive Caching (Maximum Offline Support)

```yaml
workbox:
    asset_cache:
        max_age: '90 days'
        max_entries: 200

    image_cache:
        max_age: '30 days'
        max_entries: 300

    resource_caches:
        - match_callback: 'navigate'
          strategy: 'CacheFirst'  # Aggressive
```

**Use when**: Offline-first is critical, content updates infrequently

### Pattern 2: Fresh Content (Minimum Caching)

```yaml
workbox:
    asset_cache:
        max_age: '1 day'
        max_entries: 50

    resource_caches:
        - match_callback: 'navigate'
          strategy: 'NetworkFirst'
          network_timeout: 1  # Fast timeout
          max_age: '5 minutes'  # Short cache
```

**Use when**: Real-time data is essential, caching is backup only

### Pattern 3: Balanced (Recommended Default)

```yaml
workbox:
    asset_cache:
        max_age: '30 days'

    image_cache:
        max_age: '7 days'

    resource_caches:
        - match_callback: 'navigate'
          strategy: 'NetworkFirst'
          network_timeout: 3
          max_age: '1 hour'
```

**Use when**: Good offline UX + reasonably fresh content

## Next Steps

After configuring your service worker:

1. **Test offline functionality**: Disconnect network and verify app works
2. **Monitor cache size**: Check DevTools → Application → Storage
3. **Add custom logic**: Create [Custom Service Worker Rules](custom-service-worker-rule.md)
4. **Implement background sync handlers**: Use [Symfony UX components](../symfony-ux/)
5. **Configure push notifications**: Set up notification handling

## Related Documentation

- [Configuration Reference](configuration.md) - All configuration options explained
- [Workbox Documentation](workbox/) - Detailed Workbox feature guides
- [Custom Service Worker Rules](custom-service-worker-rule.md) - Add custom logic
- [Background Sync](workbox/backgoundsync.md) - Handle offline submissions
- [Offline Fallbacks](workbox/offline-fallback.md) - Create offline pages

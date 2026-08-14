# Complete Example

This page shows a comprehensive service worker configuration using all available features.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        dest: /sw.js
        scope: /
        skip_waiting: true
        config:
            use_cdn: false
            version: '7.3.0'
            workbox_public_url: '/workbox'

        workbox:
            # Asset caching (CSS, JS from Asset Mapper)
            asset_cache:
                enabled: true
                regex: '/\.(css|js|woff2?)$/'
                cache_name: 'assets-cache'
                max_entries: 100
                max_age: 31536000 # 1 year

            # Image caching
            image_cache:
                enabled: true
                regex: '/\.(png|jpg|jpeg|gif|svg|webp|avif)$/'
                cache_name: 'images-cache'
                max_entries: 60
                max_age: 2592000 # 30 days

            # Font caching
            font_cache:
                enabled: true
                regex: '/\.(woff|woff2|ttf|otf|eot)$/'
                cache_name: 'fonts-cache'
                max_entries: 30
                max_age: 31536000 # 1 year

            # Google Fonts caching
            google_font_cache:
                enabled: true
                cache_name: 'google-fonts'
                max_entries: 20
                max_age: 31536000

            # Resource caching (pages, API)
            resource_caches:
                - match_callback: 'navigate'
                  cache_name: 'pages-cache'
                  strategy: 'StaleWhileRevalidate'
                  broadcast: true
                  max_entries: 50
                  max_age: 86400 # 1 day

                - match_callback: 'regex: /\/api\//'
                  cache_name: 'api-cache'
                  strategy: 'NetworkFirst'
                  network_timeout: 3
                  max_entries: 100
                  max_age: 3600 # 1 hour
                  cacheable_response_statuses: [0, 200]

            # Offline fallback
            offline_fallback:
                page: '/offline'
                image: '/images/offline.svg'
                font: '/fonts/fallback.woff2'

            # Background sync
            background_sync:
                - queue_name: 'contact-form'
                  match_callback: 'regex: /\/contact/'
                  method: 'POST'
                  max_retention_time: 2880 # 2 days in minutes
                  broadcast_channel: 'form-sync'

            # Cache cleaning on install
            clear_cache: true

            # Navigation preload
            navigation_preload: true
```
{% endcode %}

## What This Configuration Does

### Asset, Image, and Font Caching

Static resources are cached with long expiration times for instant loading:
- **Assets** (CSS, JS): CacheFirst, up to 100 entries, 1 year max
- **Images**: CacheFirst, up to 60 entries, 30 days max
- **Fonts**: CacheFirst, up to 30 entries, 1 year max
- **Google Fonts**: Stylesheets via StaleWhileRevalidate, font files via CacheFirst

### Page and API Caching

- **Navigation requests**: StaleWhileRevalidate with broadcast updates, so users see cached pages instantly while fresh content loads in the background
- **API calls**: NetworkFirst with 3-second timeout, falling back to cache when offline

### Offline Support

- Offline fallback page shown when no cached page is available
- Fallback image and font for graceful degradation
- Background sync queues form submissions for retry when back online

### Performance

- **Navigation Preload**: Network requests start in parallel with service worker boot
- **Cache Cleaning**: Old caches are removed on service worker install

## Combining with Push Notifications

Add push notification support alongside caching:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
self.addEventListener('push', (event) => {
    const data = event.data?.json() ?? {};
    const title = data.title ?? 'New notification';
    const options = {
        body: data.body ?? '',
        icon: '/icons/notification-192.png',
        badge: '/icons/badge-72.png',
        data: { url: data.url ?? '/' },
    };

    event.waitUntil(self.registration.showNotification(title, options));
});

self.addEventListener('notificationclick', (event) => {
    event.notification.close();
    const url = event.notification.data.url;
    event.waitUntil(clients.openWindow(url));
});
```
{% endcode %}

## Combining with Periodic Sync

Add periodic background sync for content updates:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const syncChannel = new BroadcastChannel('my-app-sync');

const syncContent = async () => {
    const cache = await openCache('pages-cache');
    const response = await fetch('/api/content/latest');
    await cache.put('/api/content/latest', response.clone());

    syncChannel.postMessage({ tag: 'content-sync', updated: true, timestamp: Date.now() });
};

self.addEventListener('periodicsync', (event) => {
    if (event.tag === 'content-sync') {
        event.waitUntil(syncContent());
    }
});
```
{% endcode %}

## Related Documentation

- [Configuration](configuration.md) - Service worker configuration reference
- [Content Security Policy](content-security-policy.md) - CSP and nonce support
- [Workbox](workbox/) - All Workbox features
- [Push Notifications](push-notifications.md) - Web Push setup
- [Periodic Sync](periodic-sync.md) - Background periodic sync
- [Custom Cache Strategy](workbox/custom-cache-strategy.md) - Programmatic cache strategies

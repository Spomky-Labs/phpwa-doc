# Periodic Sync

The Periodic Background Sync API allows your PWA to fetch fresh content periodically in the background, even when the app is closed. This ensures users always have up-to-date data when they open your app.

## Overview

Periodic sync enables:
- **Background updates**: Fetch fresh content while app is closed
- **Improved UX**: Users see latest content immediately
- **Reduced data usage**: Sync only when on WiFi/charging
- **Battery efficiency**: Browser controls execution based on engagement

{% hint style="info" %}
**Important**: Unlike other PWA Bundle features, Periodic Sync requires writing JavaScript code in both the service worker and your application.
{% endhint %}

## How It Works

1. **Client registration**: App requests periodic sync with a tag and interval
2. **Browser scheduling**: Browser decides when to actually run sync (based on engagement, battery, etc.)
3. **Service worker execution**: Service worker performs background task
4. **Client notification**: Service worker notifies clients of updates

## Browser Support

{% hint style="warning" %}
**Limited Support**: Periodic Background Sync is currently only supported in Chromium-based browsers (Chrome, Edge) on Android and Desktop. Not available in Safari or Firefox.
{% endhint %}

## Service Worker Implementation

Periodic sync is entirely yours to write. Add a `periodicsync` listener to your own service worker
source, the file configured through the `src` option.

{% hint style="danger" %}
Earlier versions offered `registerPeriodicSyncTask()` and `notifyPeriodicSyncClients()` helpers.
Both are deprecated since 1.6.0 and removed in 2.0.0: they only wrapped a listener the bundle
never uses itself. See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md).
{% endhint %}

### Basic Task

{% code title="assets/sw.js" overflow="wrap" lineNumbers="true" %}
```javascript
const syncChannel = new BroadcastChannel('my-app-sync');

const ping = async () => {
    const cache = await openCache('ping-cache');
    const response = await fetch('/ping');
    await cache.put('/ping', response.clone());

    syncChannel.postMessage({ tag: 'ping', updated: true, timestamp: Date.now() });
};

self.addEventListener('periodicsync', (event) => {
    if (event.tag === 'ping') {
        event.waitUntil(ping());
    }
});
```
{% endcode %}

{% hint style="success" %}
`openCache(name)` is provided by the bundle: it opens a cache and memoises the instance for the
lifetime of the service worker. Pair it with `registerCacheName()` if you want that cache purged
on install, see [Cache cleaning](workbox/cache-cleaning.md).
{% endhint %}

The channel name is yours to pick. The bundle used to impose `periodic-sync`; anything the page
also listens to works.

### Several Tasks Under the Same Tag

Ordering is just the order of your `await` calls:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const updateNews = async () => {
    const cache = await openCache('news-cache');
    const response = await fetch('/api/news/latest');
    await cache.put('/api/news/latest', response.clone());
};

const cleanOldNews = async () => {
    const cache = await openCache('news-cache');
    const keys = await cache.keys();
    const now = Date.now();
    const maxAge = 7 * 24 * 60 * 60 * 1000; // 7 days

    for (const request of keys) {
        const response = await cache.match(request);
        const date = new Date(response.headers.get('date'));
        if (now - date.getTime() > maxAge) {
            await cache.delete(request);
        }
    }
};

self.addEventListener('periodicsync', (event) => {
    if (event.tag === 'news-sync') {
        event.waitUntil((async () => {
            await updateNews();
            await cleanOldNews();
        })());
    }
});
```
{% endcode %}

### Several Tags

A `switch` keeps things readable once you have more than one:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
self.addEventListener('periodicsync', (event) => {
    switch (event.tag) {
        case 'news-sync':
            event.waitUntil(syncNews());
            break;
        case 'user-sync':
            event.waitUntil(prefetchUserData());
            break;
    }
});
```
{% endcode %}

### Complete Example: Blog Post Sync

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const syncChannel = new BroadcastChannel('my-app-sync');

const syncBlogPosts = async () => {
    try {
        const response = await fetch('/api/blog/latest');
        if (!response.ok) {
            throw new Error('Failed to fetch blog posts');
        }

        const posts = await response.json();
        const cache = await openCache('blog-cache');
        await cache.put('/api/blog/latest', response.clone());

        for (const post of posts) {
            await cache.put(`/api/blog/post/${post.id}`, new Response(JSON.stringify(post)));
        }

        syncChannel.postMessage({
            tag: 'blog-sync',
            updated: true,
            postCount: posts.length,
            timestamp: Date.now(),
        });
    } catch (error) {
        console.error('Blog sync failed:', error);
        syncChannel.postMessage({
            tag: 'blog-sync',
            updated: false,
            error: error.message,
        });
    }
};

self.addEventListener('periodicsync', (event) => {
    if (event.tag === 'blog-sync') {
        event.waitUntil(syncBlogPosts());
    }
});
```
{% endcode %}

## Client-Side Registration

### Basic Registration

Request periodic sync from your application:

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
async function setupPeriodicSync() {
    if (!('periodicSync' in ServiceWorkerRegistration.prototype)) {
        console.warn('Periodic Sync not supported');
        return;
    }

    const registration = await navigator.serviceWorker.ready;
    const status = await navigator.permissions.query({ name: 'periodic-background-sync' });
    if (status.state !== 'granted') {
        return;
    }

    const tags = await registration.periodicSync.getTags();
    if (!tags.includes('content-sync')) {
        await registration.periodicSync.register('content-sync', {
            minInterval: 12 * 60 * 60 * 1000,
        });
    }
}

setupPeriodicSync();
```
{% endcode %}

### Listening for Sync Updates

Receive notifications when sync completes, on the channel your service worker posts to:

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
const channel = new BroadcastChannel('my-app-sync');
channel.addEventListener('message', (event) => {
    const { tag, timestamp, ...data } = event.data;

    console.log(`Periodic sync '${tag}' completed at ${new Date(timestamp)}:`, data);

    if (data.updated) {
        updateUIWithNewContent();
    }
});
```
{% endcode %}

{% hint style="warning" %}
`BroadcastChannel.postMessage()` cannot wake a sleeping service worker, and a message sent while
no page is listening is simply lost. If the outcome matters, persist it in IndexedDB from the
service worker and read it when the page connects.
{% endhint %}

## Common Use Cases

### News/Content Updates

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const syncNews = async () => {
    const cache = await openCache('news-cache');
    const response = await fetch('/api/news/latest?limit=20');
    await cache.put('/api/news/latest', response.clone());

    const articles = await response.json();
    syncChannel.postMessage({ tag: 'news-sync', updated: true, count: articles.length });
};
```
{% endcode %}

### User Data Prefetch

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const prefetchUserData = async () => {
    const endpoints = [
        '/api/user/profile',
        '/api/user/settings',
        '/api/user/notifications',
    ];

    const cache = await openCache('user-cache');

    for (const endpoint of endpoints) {
        try {
            const response = await fetch(endpoint);
            if (response.ok) {
                await cache.put(endpoint, response.clone());
            }
        } catch (error) {
            console.warn(`Failed to prefetch ${endpoint}:`, error);
        }
    }

    syncChannel.postMessage({ tag: 'user-sync', updated: true });
};
```
{% endcode %}

## Browser Execution Control

{% hint style="warning" %}
**Important**: The browser controls when periodic sync actually runs. The interval you specify is a **minimum**, not a guarantee.
{% endhint %}

### Factors Affecting Execution

- **Site Engagement Score**: High engagement = more frequent sync
- **PWA Installation**: Installed PWAs get higher priority
- **Device State**: Battery level, charging status, data saver mode
- **Network**: WiFi preferred over cellular
- **Interval Duration**: Very short intervals (< 12 hours) often ignored

### Recommended Intervals

```javascript
// Too short - likely ignored
registration.periodicSync.register('tag', { minInterval: 30 * 60 * 1000 }); // 30 min

// Reasonable
registration.periodicSync.register('tag', { minInterval: 12 * 60 * 60 * 1000 }); // 12h

// Ideal - most likely respected
registration.periodicSync.register('tag', { minInterval: 24 * 60 * 60 * 1000 }); // 24h
```

## Managing Periodic Sync

### Check Registered Tags

```javascript
const registration = await navigator.serviceWorker.ready;
const tags = await registration.periodicSync.getTags();
console.log('Registered periodic sync tags:', tags);
```

### Unregister Periodic Sync

```javascript
const registration = await navigator.serviceWorker.ready;
await registration.periodicSync.unregister('news-sync');
```

## Testing

### Force Sync in DevTools

1. Open DevTools (F12)
2. Go to **Application** → **Service Workers**
3. Find your service worker
4. Look for "Periodic Sync" section
5. Enter tag name and click "Start"

## Graceful Degradation

Handle unsupported browsers:

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
async function setupBackgroundSync() {
    if ('periodicSync' in ServiceWorkerRegistration.prototype) {
        const registration = await navigator.serviceWorker.ready;
        await registration.periodicSync.register('content-sync', {
            minInterval: 12 * 60 * 60 * 1000,
        });
    } else {
        // Fallback: manual polling when app is open
        setInterval(async () => {
            if (document.visibilityState === 'visible') {
                await fetchLatestContent();
            }
        }, 5 * 60 * 1000); // Check every 5 minutes
    }
}
```
{% endcode %}

## Limitations

- **Supported**: Chrome, Edge (Chromium-based) only
- **Not Supported**: Safari, Firefox, older browsers
- **Platform**: Android and Desktop only (not iOS)
- Limited execution time (typically < 30 seconds)
- Browser decides actual execution time

## Permissions

```javascript
const status = await navigator.permissions.query({
    name: 'periodic-background-sync'
});
console.log('Permission status:', status.state);
// 'granted', 'denied', or 'prompt'
```

## Best Practices

1. **Use reasonable intervals**: 12+ hours recommended
2. **Minimize network requests**: Batch operations into a single sync
3. **Provide user control**: Allow users to configure sync frequency
4. **Handle failures gracefully**: Return Promises, use try/catch
5. **Check network conditions**: Skip heavy sync on poor connections

## Related Documentation

- [Background Sync](workbox/background-sync.md) - One-time background sync
- [Service Worker Configuration](configuration.md) - Service worker setup
- [Workbox](workbox/) - Workbox features

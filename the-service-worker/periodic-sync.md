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

**Check support**:
```javascript
if ('periodicSync' in ServiceWorkerRegistration.prototype) {
    console.log('Periodic Background Sync supported!');
} else {
    console.log('Periodic Background Sync NOT supported');
}
```

## Service Worker Implementation

### Basic Task Registration

Create periodic tasks in your service worker:

{% code title="assets/sw.js" overflow="wrap" lineNumbers="true" %}
```javascript
const ping = async () => {
    const cache = await openCache('ping-cache');
    const res = await fetch('/ping');
    await cache.put('/ping', res.clone());

    notifyPeriodicSyncClients('ping', { updated: true });
};

registerPeriodicSyncTask('ping', ping);
```
{% endcode %}

{% hint style="success" %}
**Helper Functions**: The bundle provides `openCache`, `notifyPeriodicSyncClients`, and `registerPeriodicSyncTask` helpers in your service worker.
{% endhint %}

### Multiple Tasks Under Same Tag

Register multiple related tasks:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
// Task 1: Update news cache
const updateNews = async () => {
    const cache = await openCache('news-cache');
    const response = await fetch('/api/news/latest');
    const news = await response.json();
    await cache.put('/api/news/latest', new Response(JSON.stringify(news)));
};

// Task 2: Clean old news
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

// Register both under 'news-sync' tag
registerPeriodicSyncTask('news-sync', updateNews);
registerPeriodicSyncTask('news-sync', cleanOldNews);
```
{% endcode %}

### Complete Example: Blog Post Sync

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const syncBlogPosts = async () => {
    try {
        // Fetch latest posts
        const response = await fetch('/api/blog/latest');
        if (!response.ok) {
            throw new Error('Failed to fetch blog posts');
        }

        const posts = await response.json();

        // Cache the response
        const cache = await openCache('blog-cache');
        await cache.put('/api/blog/latest', response.clone());

        // Store individual posts
        for (const post of posts) {
            const postResponse = new Response(JSON.stringify(post));
            await cache.put(`/api/blog/post/${post.id}`, postResponse);
        }

        // Notify clients about update
        notifyPeriodicSyncClients('blog-sync', {
            updated: true,
            postCount: posts.length,
            timestamp: Date.now()
        });

        console.log(`Synced ${posts.length} blog posts`);
    } catch (error) {
        console.error('Blog sync failed:', error);

        // Notify clients about failure
        notifyPeriodicSyncClients('blog-sync', {
            updated: false,
            error: error.message
        });
    }
};

registerPeriodicSyncTask('blog-sync', syncBlogPosts);
```
{% endcode %}

## Client-Side Registration

### Basic Registration

Request periodic sync from your application:

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
import { registerPeriodicSync } from '@spomky-labs/pwa/helpers';

// Register periodic sync for 'ping' tag every 6 hours
await registerPeriodicSync('ping', 6 * 60 * 60 * 1000);
```
{% endcode %}

### With Error Handling

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
import { registerPeriodicSync } from '@spomky-labs/pwa/helpers';

async function setupPeriodicSync() {
    try {
        // Check if supported
        if (!('periodicSync' in ServiceWorkerRegistration.prototype)) {
            console.warn('Periodic Sync not supported');
            return;
        }

        // Register sync every 12 hours
        await registerPeriodicSync('content-sync', 12 * 60 * 60 * 1000);
        console.log('Periodic sync registered successfully');
    } catch (error) {
        console.error('Failed to register periodic sync:', error);
    }
}

setupPeriodicSync();
```
{% endcode %}

### Listening for Sync Updates

Receive notifications when sync completes:

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
// Listen for periodic sync updates
navigator.serviceWorker.addEventListener('message', (event) => {
    if (event.data.type === 'periodicSync') {
        const { tag, data } = event.data;

        console.log(`Periodic sync '${tag}' completed:`, data);

        if (data.updated) {
            // Update UI with fresh data
            updateUIWithNewContent();

            // Show notification to user
            showNotification('New content available!');
        }
    }
});

function updateUIWithNewContent() {
    // Reload data from cache
    fetch('/api/news/latest')
        .then(response => response.json())
        .then(data => {
            // Update UI components
            renderNewsList(data);
        });
}

function showNotification(message) {
    const notification = document.createElement('div');
    notification.className = 'notification';
    notification.textContent = message;
    document.body.appendChild(notification);

    setTimeout(() => notification.remove(), 3000);
}
```
{% endcode %}

## Common Use Cases

### 1. News/Content Updates

Keep articles fresh in the background:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const syncNews = async () => {
    const cache = await openCache('news-cache');
    const response = await fetch('/api/news/latest?limit=20');
    await cache.put('/api/news/latest', response.clone());

    const articles = await response.json();
    notifyPeriodicSyncClients('news-sync', {
        updated: true,
        count: articles.length
    });
};

registerPeriodicSyncTask('news-sync', syncNews);
```
{% endcode %}

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
// Sync news every 4 hours
await registerPeriodicSync('news-sync', 4 * 60 * 60 * 1000);
```
{% endcode %}

### 2. Social Media Feed

Update social feed periodically:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const syncFeed = async () => {
    const cache = await openCache('social-cache');
    const response = await fetch('/api/feed?since=' + Date.now());
    const feed = await response.json();

    if (feed.items.length > 0) {
        await cache.put('/api/feed', response.clone());

        notifyPeriodicSyncClients('feed-sync', {
            updated: true,
            newItems: feed.items.length
        });
    }
};

registerPeriodicSyncTask('feed-sync', syncFeed);
```
{% endcode %}

### 3. Weather Data

Keep weather information up-to-date:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const syncWeather = async () => {
    try {
        const response = await fetch('/api/weather/current');
        const weather = await response.json();

        const cache = await openCache('weather-cache');
        await cache.put('/api/weather/current', response.clone());

        notifyPeriodicSyncClients('weather-sync', {
            updated: true,
            temperature: weather.temperature,
            condition: weather.condition
        });
    } catch (error) {
        console.error('Weather sync failed:', error);
    }
};

registerPeriodicSyncTask('weather-sync', syncWeather);
```
{% endcode %}

### 4. Cache Cleanup

Periodically clean old cached data:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const cleanupCache = async () => {
    const cacheNames = await caches.keys();
    const now = Date.now();
    const maxAge = 30 * 24 * 60 * 60 * 1000; // 30 days

    for (const cacheName of cacheNames) {
        const cache = await caches.open(cacheName);
        const keys = await cache.keys();

        for (const request of keys) {
            const response = await cache.match(request);
            const dateHeader = response.headers.get('date');

            if (dateHeader) {
                const cacheDate = new Date(dateHeader).getTime();
                if (now - cacheDate > maxAge) {
                    await cache.delete(request);
                }
            }
        }
    }

    notifyPeriodicSyncClients('cleanup', { cleaned: true });
};

registerPeriodicSyncTask('cleanup', cleanupCache);
```
{% endcode %}

### 5. User Data Prefetch

Preload user-specific data:

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
const prefetchUserData = async () => {
    const endpoints = [
        '/api/user/profile',
        '/api/user/settings',
        '/api/user/notifications'
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

    notifyPeriodicSyncClients('user-sync', { updated: true });
};

registerPeriodicSyncTask('user-sync', prefetchUserData);
```
{% endcode %}

## Browser Execution Control

{% hint style="warning" %}
**Important**: The browser controls when periodic sync actually runs. The interval you specify is a **minimum**, not a guarantee.
{% endhint %}

### Factors Affecting Execution

**1. Site Engagement Score**
- High engagement = more frequent sync
- Low engagement = less frequent or no sync
- Based on user's interaction with your site

**2. PWA Installation**
- Installed PWAs get higher priority
- Non-installed sites may not sync at all

**3. Device State**
- Battery level (low battery = no sync)
- Charging status (more likely when charging)
- Data saver mode (disabled when enabled)
- Network type (WiFi preferred over cellular)

**4. User Activity**
- More likely during typical usage times
- Skipped during extended inactivity

**5. Interval Duration**
- Very short intervals (< 12 hours) often ignored
- Longer intervals more likely to be respected
- Practical minimum: 12-24 hours

### Recommended Intervals

```javascript
// Too short - likely ignored
await registerPeriodicSync('tag', 30 * 60 * 1000); // 30 minutes ✗

// Short - may be ignored
await registerPeriodicSync('tag', 2 * 60 * 60 * 1000); // 2 hours ✗

// Reasonable - good balance
await registerPeriodicSync('tag', 12 * 60 * 60 * 1000); // 12 hours ✓

// Ideal - most likely respected
await registerPeriodicSync('tag', 24 * 60 * 60 * 1000); // 24 hours ✓
```

## Managing Periodic Sync

### Check Registered Tags

Query which tags are registered:

```javascript
const registration = await navigator.serviceWorker.ready;
const tags = await registration.periodicSync.getTags();
console.log('Registered periodic sync tags:', tags);
```

### Unregister Periodic Sync

Stop periodic sync for a tag:

```javascript
const registration = await navigator.serviceWorker.ready;
await registration.periodicSync.unregister('news-sync');
console.log('Periodic sync unregistered');
```

### Re-register After Updates

Update periodic sync interval:

```javascript
async function updateSyncInterval(tag, newInterval) {
    const registration = await navigator.serviceWorker.ready;

    // Unregister old
    await registration.periodicSync.unregister(tag);

    // Register with new interval
    await registerPeriodicSync(tag, newInterval);
}

// Change news sync from 12h to 6h
await updateSyncInterval('news-sync', 6 * 60 * 60 * 1000);
```

## Testing Periodic Sync

### Force Sync in DevTools

Chrome DevTools allows manual triggering:

```bash
1. Open DevTools (F12)
2. Go to Application → Service Workers
3. Find your service worker
4. Look for "Periodic Sync" section
5. Enter tag name
6. Click "Start"
```

### Test in Code

Manually trigger for testing:

```javascript
// In your test file
async function testPeriodicSync() {
    const registration = await navigator.serviceWorker.ready;

    // Manually dispatch periodicsync event
    const event = new Event('periodicsync');
    event.tag = 'news-sync';

    registration.active.dispatchEvent(event);
}
```

### Debugging Tips

```javascript
// Add extensive logging
const syncNews = async () => {
    console.log('[Periodic Sync] Starting news sync...');

    try {
        const start = Date.now();
        const response = await fetch('/api/news/latest');

        console.log('[Periodic Sync] Fetch completed in', Date.now() - start, 'ms');
        console.log('[Periodic Sync] Status:', response.status);

        const cache = await openCache('news-cache');
        await cache.put('/api/news/latest', response.clone());

        console.log('[Periodic Sync] Cached successfully');

        notifyPeriodicSyncClients('news-sync', { updated: true });
    } catch (error) {
        console.error('[Periodic Sync] Failed:', error);
        throw error; // Re-throw to signal failure to browser
    }
};
```

## Error Handling

### Retry on Failure

The browser will retry if your sync task throws an error:

```javascript
const syncWithRetry = async () => {
    let retries = 3;

    while (retries > 0) {
        try {
            const response = await fetch('/api/data');
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            const cache = await openCache('data-cache');
            await cache.put('/api/data', response.clone());

            notifyPeriodicSyncClients('data-sync', { updated: true });
            return; // Success
        } catch (error) {
            retries--;
            console.error(`Sync failed, ${retries} retries left:`, error);

            if (retries === 0) {
                throw error; // Signal complete failure
            }

            // Wait before retry
            await new Promise(resolve => setTimeout(resolve, 1000));
        }
    }
};

registerPeriodicSyncTask('data-sync', syncWithRetry);
```

### Graceful Degradation

Handle unsupported browsers:

```javascript
import { registerPeriodicSync } from '@spomky-labs/pwa/helpers';

async function setupBackgroundSync() {
    if ('periodicSync' in ServiceWorkerRegistration.prototype) {
        // Periodic sync supported
        await registerPeriodicSync('content-sync', 12 * 60 * 60 * 1000);
        console.log('Using Periodic Background Sync');
    } else {
        // Fallback: manual polling when app is open
        setInterval(async () => {
            if (document.visibilityState === 'visible') {
                await fetchLatestContent();
            }
        }, 5 * 60 * 1000); // Check every 5 minutes
        console.log('Using manual polling (Periodic Sync not supported)');
    }
}
```

## Best Practices

### 1. Use Reasonable Intervals

```javascript
// ✗ Avoid very short intervals
await registerPeriodicSync('tag', 5 * 60 * 1000); // 5 minutes

// ✓ Use longer intervals
await registerPeriodicSync('tag', 12 * 60 * 60 * 1000); // 12 hours
```

### 2. Minimize Network Requests

```javascript
// ✓ Efficient - single request
const syncEfficient = async () => {
    const response = await fetch('/api/updates?all=true');
    // ... handle response
};

// ✗ Inefficient - multiple requests
const syncInefficient = async () => {
    await fetch('/api/news');
    await fetch('/api/weather');
    await fetch('/api/stocks');
};
```

### 3. Check Network Conditions

```javascript
const syncWithNetworkCheck = async () => {
    // Check connection type
    const connection = navigator.connection;

    if (connection && connection.effectiveType === '4g') {
        // Good connection - fetch everything
        await fetchAllUpdates();
    } else if (connection && connection.effectiveType === '3g') {
        // Moderate connection - fetch essentials only
        await fetchEssentialUpdates();
    } else {
        // Poor connection - skip
        console.log('Skipping sync due to poor connection');
        return;
    }
};
```

### 4. Provide User Control

Allow users to configure sync:

```javascript
// In your settings UI
async function updateSyncPreferences(enabled, interval) {
    if (enabled) {
        await registerPeriodicSync('content-sync', interval);
    } else {
        const registration = await navigator.serviceWorker.ready;
        await registration.periodicSync.unregister('content-sync');
    }

    // Save preference
    localStorage.setItem('syncEnabled', enabled);
    localStorage.setItem('syncInterval', interval);
}
```

### 5. Monitor Performance

Track sync performance:

```javascript
const syncWithMetrics = async () => {
    const start = performance.now();

    try {
        await fetch('/api/data');
        const duration = performance.now() - start;

        // Send metrics
        await fetch('/api/metrics', {
            method: 'POST',
            body: JSON.stringify({
                event: 'periodic_sync',
                duration,
                success: true
            })
        });
    } catch (error) {
        const duration = performance.now() - start;

        await fetch('/api/metrics', {
            method: 'POST',
            body: JSON.stringify({
                event: 'periodic_sync',
                duration,
                success: false,
                error: error.message
            })
        });

        throw error;
    }
};
```

## Limitations

### Browser Support

- **Supported**: Chrome, Edge (Chromium-based)
- **Not Supported**: Safari, Firefox, older browsers
- **Platform**: Android and Desktop only (not iOS)

### Execution Constraints

- Browser decides actual execution time
- May not run at all if engagement is low
- Requires PWA installation for best results
- Respects battery and data saver settings

### Resource Limits

- Limited execution time (typically < 30 seconds)
- Must complete quickly or risk termination
- Network-only (no intensive computations)

## Permissions

Periodic sync requires the same permissions as service workers:

```javascript
// Check permission status
const status = await navigator.permissions.query({
    name: 'periodic-background-sync'
});

console.log('Permission status:', status.state);
// 'granted', 'denied', or 'prompt'
```

## Related Documentation

- [Background Sync](workbox/backgoundsync.md) - One-time background sync
- [Service Worker Configuration](configuration.md) - Service worker setup
- [Workbox](workbox/) - Workbox features

## Resources

- **MDN Periodic Background Sync**: https://developer.mozilla.org/en-US/docs/Web/API/Web_Periodic_Background_Synchronization_API
- **Web.dev Article**: https://web.dev/periodic-background-sync/
- **Chrome Developers**: https://developer.chrome.com/docs/capabilities/periodic-background-sync
- **Can I Use**: https://caniuse.com/periodic-background-sync

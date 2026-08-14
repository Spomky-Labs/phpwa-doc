# Periodic Sync

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Periodic Sync component enables your Progressive Web App to synchronize data in the background at regular intervals, even when the app is closed. This feature leverages the Periodic Background Sync API to keep content fresh without requiring constant user interaction.

## Introduction

Periodic Background Sync allows your PWA to:

* Fetch fresh content at regular intervals in the background
* Update local caches with the latest data
* Synchronize data when the device is idle and on Wi-Fi
* Provide up-to-date content when users return to the app
* Reduce data usage by syncing only when conditions are optimal

{% hint style="info" %}
Periodic Sync requires an active service worker and is subject to browser-controlled intervals. The browser decides when to actually trigger syncs based on site engagement and device conditions.
{% endhint %}

## Browser Support

<table><thead><tr><th width="200">Browser</th><th width="150">Desktop</th><th>Mobile</th></tr></thead><tbody>
<tr><td>Chrome/Edge</td><td>✅ 80+</td><td>✅ 80+</td></tr>
<tr><td>Firefox</td><td>❌ Not supported</td><td>❌ Not supported</td></tr>
<tr><td>Safari</td><td>❌ Not supported</td><td>❌ Not supported</td></tr>
<tr><td>Opera</td><td>✅ Full</td><td>✅ Full</td></tr>
<tr><td>Samsung Internet</td><td>N/A</td><td>✅ 13+</td></tr>
</tbody></table>

{% hint style="warning" %}
Periodic Background Sync is currently only supported in Chromium-based browsers (Chrome, Edge, Opera). The API requires the `periodic-background-sync` permission, which is automatically granted on desktop but may require user engagement on mobile.
{% endhint %}

## Prerequisites

Before using Periodic Sync, ensure that:

1. **Service Worker is enabled** in your PWA configuration
2. **Workbox is enabled** as it provides the necessary helpers
3. The user has granted the **periodic-background-sync permission**

## Usage

### Basic Periodic Sync Registration

First, import the helpers in your JavaScript:

{% code lineNumbers="true" %}
```javascript
import { registerPeriodicSync, onPeriodicSync } from '@symfony/ux-pwa/helpers';

// Register a periodic sync that runs at least every 24 hours
await registerPeriodicSync('content-sync', 24 * 60 * 60 * 1000);

// Listen for sync events
onPeriodicSync('content-sync', (data) => {
    console.log('Content synced!', data);
    updateUI(data);
});
```
{% endcode %}

### News Feed Synchronization

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
import { registerPeriodicSync, onPeriodicSync } from '@symfony/ux-pwa/helpers';

// Register periodic sync for news updates (every 12 hours)
async function setupNewsFeedSync() {
    try {
        await registerPeriodicSync('news-feed-sync', 12 * 60 * 60 * 1000);
        console.log('News feed sync registered');

        // Listen for sync completion
        onPeriodicSync('news-feed-sync', (data) => {
            if (data.success) {
                console.log(`Synced ${data.articleCount} articles`);
                // Refresh the news feed UI
                refreshNewsFeed();
            }
        });
    } catch (error) {
        console.error('Failed to register periodic sync:', error);
    }
}

setupNewsFeedSync();
```
{% endcode %}

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
// Register the task that will execute during periodic sync
registerPeriodicSyncTask('news-feed-sync', async (event) => {
    console.log('Periodic sync triggered: news-feed-sync');

    try {
        // Fetch latest articles from API
        const response = await fetch('/api/articles/latest');
        const articles = await response.json();

        // Update cache with new articles
        const cache = await openCache('articles-cache');
        await cache.put('/api/articles/latest', new Response(JSON.stringify(articles)));

        // Notify clients about successful sync
        notifyPeriodicSyncClients('news-feed-sync', {
            success: true,
            articleCount: articles.length,
            timestamp: Date.now()
        });
    } catch (error) {
        console.error('Failed to sync news feed:', error);

        notifyPeriodicSyncClients('news-feed-sync', {
            success: false,
            error: error.message
        });
    }
});
```
{% endcode %}

### Weather Data Synchronization

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
import { registerPeriodicSync, onPeriodicSync } from '@symfony/ux-pwa/helpers';

async function setupWeatherSync() {
    // Register sync every 2 hours
    await registerPeriodicSync('weather-sync', 2 * 60 * 60 * 1000);

    // Update UI when weather data is synced
    onPeriodicSync('weather-sync', (data) => {
        if (data.weather) {
            updateWeatherWidget(data.weather);
            showNotification('Weather Updated', {
                body: `Current temperature: ${data.weather.temp}°C`,
                icon: '/weather-icon.png'
            });
        }
    });
}

setupWeatherSync();
```
{% endcode %}

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
registerPeriodicSyncTask('weather-sync', async (event) => {
    try {
        // Get user location from IndexedDB or use default
        const location = await getUserLocation() || { lat: 48.8566, lon: 2.3522 };

        // Fetch weather data
        const response = await fetch(
            `/api/weather?lat=${location.lat}&lon=${location.lon}`
        );
        const weather = await response.json();

        // Cache the weather data
        const cache = await openCache('weather-cache');
        await cache.put('/api/weather/current', new Response(JSON.stringify(weather)));

        // Notify clients
        notifyPeriodicSyncClients('weather-sync', {
            weather,
            location,
            timestamp: Date.now()
        });
    } catch (error) {
        console.error('Weather sync failed:', error);
        notifyPeriodicSyncClients('weather-sync', { error: error.message });
    }
});
```
{% endcode %}

### Social Media Timeline Sync

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
import { registerPeriodicSync, onPeriodicSync } from '@symfony/ux-pwa/helpers';

// Setup periodic sync for social media timeline
async function setupTimelineSync() {
    await registerPeriodicSync('timeline-sync', 30 * 60 * 1000); // Every 30 minutes

    onPeriodicSync('timeline-sync', (data) => {
        if (data.newPosts > 0) {
            // Show badge notification
            showBadge(data.newPosts);

            // Update timeline without page reload
            if (isTimelineVisible()) {
                prependNewPosts(data.posts);
            }
        }
    });
}

setupTimelineSync();
```
{% endcode %}

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
registerPeriodicSyncTask('timeline-sync', async (event) => {
    try {
        // Get the timestamp of the last sync
        const lastSync = await getLastSyncTimestamp('timeline');

        // Fetch new posts since last sync
        const response = await fetch(`/api/timeline?since=${lastSync}`);
        const data = await response.json();

        if (data.posts.length > 0) {
            // Store new posts in IndexedDB
            await storePostsInDB(data.posts);

            // Update cache
            const cache = await openCache('timeline-cache');
            await cache.put('/api/timeline/latest', new Response(JSON.stringify(data)));

            // Update last sync timestamp
            await updateLastSyncTimestamp('timeline', Date.now());

            // Notify clients
            notifyPeriodicSyncClients('timeline-sync', {
                newPosts: data.posts.length,
                posts: data.posts
            });
        }
    } catch (error) {
        console.error('Timeline sync failed:', error);
    }
});
```
{% endcode %}

### Podcast Episode Download

{% code title="assets/app.js" lineNumbers="true" %}
```javascript
import { registerPeriodicSync, onPeriodicSync } from '@symfony/ux-pwa/helpers';

async function setupPodcastSync() {
    // Check for new episodes every 6 hours
    await registerPeriodicSync('podcast-sync', 6 * 60 * 60 * 1000);

    onPeriodicSync('podcast-sync', (data) => {
        if (data.newEpisodes && data.newEpisodes.length > 0) {
            showNotification('New Episodes Available', {
                body: `${data.newEpisodes.length} new episodes ready to download`,
                badge: '/badge-podcast.png',
                tag: 'podcast-update'
            });

            // Update podcast list
            updatePodcastList(data.newEpisodes);
        }
    });
}

setupPodcastSync();
```
{% endcode %}

{% code title="assets/sw.js" lineNumbers="true" %}
```javascript
registerPeriodicSyncTask('podcast-sync', async (event) => {
    try {
        // Get user's subscribed podcasts
        const subscriptions = await getUserPodcastSubscriptions();
        const newEpisodes = [];

        // Check each podcast for new episodes
        for (const podcast of subscriptions) {
            const response = await fetch(`/api/podcasts/${podcast.id}/episodes/latest`);
            const episodes = await response.json();

            // Filter episodes that are newer than last check
            const unseenEpisodes = episodes.filter(ep =>
                new Date(ep.publishedAt) > new Date(podcast.lastChecked)
            );

            if (unseenEpisodes.length > 0) {
                newEpisodes.push(...unseenEpisodes);

                // Optionally pre-cache episode audio files
                const cache = await openCache('podcast-episodes');
                for (const episode of unseenEpisodes) {
                    if (podcast.autoDownload) {
                        await cache.add(episode.audioUrl);
                    }
                }
            }
        }

        // Update last checked timestamp
        await updatePodcastCheckTimestamp(subscriptions);

        // Notify clients
        notifyPeriodicSyncClients('podcast-sync', {
            newEpisodes,
            totalCount: newEpisodes.length
        });
    } catch (error) {
        console.error('Podcast sync failed:', error);
    }
});
```
{% endcode %}

## Best Practices

### 1. Request Minimal Sync Intervals

The `minInterval` parameter is a suggestion, not a guarantee. The browser may sync less frequently based on:

- Site engagement (how often user visits)
- Device battery level
- Network connectivity (prefers Wi-Fi)
- Data saver mode

```javascript
// ✅ Good: Reasonable interval (24 hours)
await registerPeriodicSync('daily-sync', 24 * 60 * 60 * 1000);

// ❌ Bad: Too frequent (may be ignored or throttled)
await registerPeriodicSync('frequent-sync', 60 * 1000); // Every minute
```

### 2. Check Permission Status

Always check if the periodic-background-sync permission is granted:

```javascript
async function checkPeriodicSyncPermission() {
    try {
        const status = await navigator.permissions.query({
            name: 'periodic-background-sync'
        });

        if (status.state === 'granted') {
            console.log('Periodic sync permission granted');
            return true;
        } else {
            console.log('Periodic sync permission denied or prompt');
            return false;
        }
    } catch (error) {
        console.log('Periodic sync not supported');
        return false;
    }
}

// Use before registering
const hasPermission = await checkPeriodicSyncPermission();
if (hasPermission) {
    await registerPeriodicSync('my-sync', 12 * 60 * 60 * 1000);
}
```

### 3. Handle Sync Failures Gracefully

Background syncs can fail for various reasons:

```javascript
registerPeriodicSyncTask('content-sync', async (event) => {
    let retryCount = 0;
    const maxRetries = 3;

    async function syncWithRetry() {
        try {
            const response = await fetch('/api/content/latest');

            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }

            const data = await response.json();

            // Success: cache data and notify clients
            await cacheData(data);
            notifyPeriodicSyncClients('content-sync', {
                success: true,
                itemCount: data.length
            });
        } catch (error) {
            retryCount++;

            if (retryCount < maxRetries) {
                // Wait before retry (exponential backoff)
                await new Promise(resolve =>
                    setTimeout(resolve, Math.pow(2, retryCount) * 1000)
                );
                return syncWithRetry();
            } else {
                // Max retries reached
                console.error('Sync failed after retries:', error);
                notifyPeriodicSyncClients('content-sync', {
                    success: false,
                    error: error.message
                });
            }
        }
    }

    await syncWithRetry();
});
```

### 4. Respect User Data Preferences

Only sync essential data automatically. For large downloads, ask for user permission:

```javascript
registerPeriodicSyncTask('media-sync', async (event) => {
    // Check user preferences
    const preferences = await getUserPreferences();

    if (preferences.dataSaver) {
        // Skip heavy data sync in data saver mode
        console.log('Skipping media sync: data saver enabled');
        return;
    }

    // Only download on Wi-Fi if user prefers
    if (preferences.wifiOnly) {
        const connection = navigator.connection;
        if (connection && connection.effectiveType !== 'wifi') {
            console.log('Skipping media sync: not on Wi-Fi');
            return;
        }
    }

    // Proceed with sync
    await syncMediaContent();
});
```

### 5. Provide User Control

Let users manage sync settings:

```twig
<div class="sync-settings">
    <h3>Background Sync Settings</h3>

    <label>
        <input type="checkbox" id="enable-sync" checked>
        Enable automatic content sync
    </label>

    <label>
        Sync frequency:
        <select id="sync-interval">
            <option value="3600000">Every hour</option>
            <option value="21600000">Every 6 hours</option>
            <option value="43200000" selected>Every 12 hours</option>
            <option value="86400000">Daily</option>
        </select>
    </label>

    <label>
        <input type="checkbox" id="wifi-only" checked>
        Sync only on Wi-Fi
    </label>
</div>

<script>
import { registerPeriodicSync } from '@symfony/ux-pwa/helpers';

document.getElementById('enable-sync').addEventListener('change', async (e) => {
    if (e.target.checked) {
        const interval = parseInt(document.getElementById('sync-interval').value);
        await registerPeriodicSync('user-content-sync', interval);
    } else {
        // Unregister sync
        const registration = await navigator.serviceWorker.ready;
        await registration.periodicSync.unregister('user-content-sync');
    }
});

document.getElementById('sync-interval').addEventListener('change', async (e) => {
    if (document.getElementById('enable-sync').checked) {
        const interval = parseInt(e.target.value);
        await registerPeriodicSync('user-content-sync', interval);
    }
});
</script>
```

## Common Use Cases

### 1. News and Content Aggregation

Keep news feeds fresh with periodic updates:

```php
class NewsController extends AbstractController
{
    #[Route('/api/articles/latest')]
    public function getLatest(Request $request): JsonResponse
    {
        $since = $request->query->get('since', time() - 3600);

        $articles = $this->articleRepository->findPublishedSince(
            new \DateTime('@' . $since)
        );

        return $this->json([
            'articles' => $articles,
            'count' => count($articles),
            'timestamp' => time()
        ]);
    }
}
```

### 2. Inventory and Pricing Updates

For e-commerce apps, keep product information current:

```javascript
// assets/sw.js
registerPeriodicSyncTask('inventory-sync', async (event) => {
    try {
        // Get products user is interested in
        const watchlist = await getProductWatchlist();

        if (watchlist.length === 0) return;

        // Fetch updated prices and availability
        const response = await fetch('/api/products/updates', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ products: watchlist })
        });

        const updates = await response.json();

        // Check for price drops or stock changes
        const priceDrops = updates.filter(p => p.priceChanged && p.newPrice < p.oldPrice);
        const backInStock = updates.filter(p => p.stockChanged && p.inStock);

        if (priceDrops.length > 0 || backInStock.length > 0) {
            // Show notification
            await self.registration.showNotification('Product Updates', {
                body: `${priceDrops.length} price drops, ${backInStock.length} back in stock`,
                badge: '/badge-shopping.png',
                data: { priceDrops, backInStock }
            });
        }

        // Update cache
        await updateProductCache(updates);

        notifyPeriodicSyncClients('inventory-sync', {
            updates,
            priceDrops: priceDrops.length,
            backInStock: backInStock.length
        });
    } catch (error) {
        console.error('Inventory sync failed:', error);
    }
});
```

### 3. Calendar and Event Synchronization

Keep calendar events synchronized:

```javascript
registerPeriodicSyncTask('calendar-sync', async (event) => {
    try {
        const lastSync = await getLastCalendarSync();

        // Fetch calendar updates
        const response = await fetch(`/api/calendar/events?since=${lastSync}`);
        const data = await response.json();

        if (data.events.length > 0) {
            // Store events in IndexedDB
            await storeCalendarEvents(data.events);

            // Check for upcoming events (within next hour)
            const upcomingEvents = data.events.filter(event => {
                const eventTime = new Date(event.startTime);
                const oneHourFromNow = Date.now() + (60 * 60 * 1000);
                return eventTime.getTime() <= oneHourFromNow;
            });

            // Notify about upcoming events
            for (const event of upcomingEvents) {
                await self.registration.showNotification(event.title, {
                    body: `Starting soon at ${new Date(event.startTime).toLocaleTimeString()}`,
                    badge: '/badge-calendar.png',
                    tag: `event-${event.id}`
                });
            }

            await updateLastCalendarSync(Date.now());

            notifyPeriodicSyncClients('calendar-sync', {
                newEvents: data.events.length,
                upcomingEvents: upcomingEvents.length
            });
        }
    } catch (error) {
        console.error('Calendar sync failed:', error);
    }
});
```

## API Reference

### Client-Side Helpers

#### `registerPeriodicSync(tag, minInterval, options)`

Registers a periodic background sync task with the service worker.

**Parameters**:
- `tag` (String, required): Unique identifier for this sync task
- `minInterval` (Number, required): Minimum interval in milliseconds between syncs
- `options` (Object, optional): Additional registration options

**Returns**: `Promise<void>`

**Example**:
```javascript
import { registerPeriodicSync } from '@symfony/ux-pwa/helpers';

// Register sync every 12 hours
await registerPeriodicSync('content-sync', 12 * 60 * 60 * 1000);

// With options
await registerPeriodicSync('backup-sync', 24 * 60 * 60 * 1000, {
    // Additional options can be passed here
});
```

**Notes**:
- Requires the `periodic-background-sync` permission
- The browser controls actual sync frequency based on user engagement
- Only works when the service worker is active

#### `onPeriodicSync(tag, callback)`

Listens for periodic sync completion events from the service worker.

**Parameters**:
- `tag` (String, required): The sync tag to listen for
- `callback` (Function, required): Function called when sync completes. Receives sync data as parameter.

**Returns**: `void`

**Example**:
```javascript
import { onPeriodicSync } from '@symfony/ux-pwa/helpers';

onPeriodicSync('content-sync', (data) => {
    console.log('Sync completed:', data);

    if (data.success) {
        updateUI(data);
    } else {
        showError(data.error);
    }
});
```

**Event Data Structure**:
```javascript
{
    type: 'periodic-sync-update',
    tag: 'content-sync',
    timestamp: 1234567890,
    ...payload  // Custom data from notifyPeriodicSyncClients()
}
```

### Service Worker Functions

#### `registerPeriodicSyncTask(tag, callback, priority)`

Registers a task to be executed when a periodic sync event occurs in the service worker.

**Parameters**:
- `tag` (String, required): Unique identifier matching the client-side registration
- `callback` (Function, required): Async function to execute during sync. Receives the event object.
- `priority` (Number, optional): Execution priority (lower numbers run first). Default: 100

**Returns**: `void`

**Example**:
```javascript
// In assets/sw.js
registerPeriodicSyncTask('content-sync', async (event) => {
    console.log('Periodic sync triggered:', event.tag);

    try {
        const response = await fetch('/api/content/latest');
        const data = await response.json();

        // Cache the data
        const cache = await openCache('content-cache');
        await cache.put('/api/content/latest', new Response(JSON.stringify(data)));

        // Notify clients
        notifyPeriodicSyncClients('content-sync', {
            success: true,
            itemCount: data.length
        });
    } catch (error) {
        notifyPeriodicSyncClients('content-sync', {
            success: false,
            error: error.message
        });
    }
}, 100);
```

#### `notifyPeriodicSyncClients(tag, payload)`

Sends a message to all clients listening for a specific periodic sync event.

**Parameters**:
- `tag` (String, required): The sync tag
- `payload` (Object, optional): Custom data to send to clients

**Returns**: `void`

**Example**:
```javascript
// In assets/sw.js
notifyPeriodicSyncClients('weather-sync', {
    success: true,
    temperature: 22,
    condition: 'sunny',
    lastUpdate: Date.now()
});
```

The payload will be merged with default properties:
```javascript
{
    type: 'periodic-sync-update',
    tag: 'weather-sync',
    timestamp: Date.now(),
    ...payload
}
```

## Unregistering Periodic Sync

To stop a periodic sync:

```javascript
async function unregisterPeriodicSync(tag) {
    try {
        const registration = await navigator.serviceWorker.ready;

        if ('periodicSync' in registration) {
            await registration.periodicSync.unregister(tag);
            console.log(`Unregistered periodic sync: ${tag}`);
        }
    } catch (error) {
        console.error('Failed to unregister periodic sync:', error);
    }
}

// Usage
await unregisterPeriodicSync('content-sync');
```

## Checking Registered Syncs

List all registered periodic syncs:

```javascript
async function listPeriodicSyncs() {
    try {
        const registration = await navigator.serviceWorker.ready;

        if ('periodicSync' in registration) {
            const tags = await registration.periodicSync.getTags();
            console.log('Registered syncs:', tags);
            return tags;
        }
    } catch (error) {
        console.error('Failed to list periodic syncs:', error);
        return [];
    }
}

// Usage
const syncs = await listPeriodicSyncs();
// Output: ['content-sync', 'weather-sync', 'calendar-sync']
```

## Related Components

* [BackgroundSync Form](backgroundsync-form.md) - One-time background synchronization for form submissions
* [BackgroundSync Queue](backgroundsync-queue.md) - Monitor background sync queue status
* [Service Worker](service-worker.md) - Manage service worker lifecycle

## Additional Resources

* [Periodic Background Sync API on MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_Periodic_Background_Synchronization_API)
* [Periodic Background Sync on web.dev](https://web.dev/periodic-background-sync/)
* [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)

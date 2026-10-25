# BackgroundSync Queue

The BackgroundSync Queue component provides real-time monitoring and manual control of background synchronization queues in your Progressive Web App. When network requests fail (due to being offline or network errors), they are automatically queued by the service worker and retried when connectivity is restored. This component displays the number of pending requests and allows users to manually trigger queue replay.

This component is particularly useful for:

* Showing users how many pending actions are waiting to sync
* Providing manual retry buttons for failed operations
* Building offline-first forms and data entry applications
* Creating reliable e-commerce checkout flows that work offline
* Implementing offline task management and todo applications
* Monitoring background sync status in real-time
* Giving users control over when to retry failed operations
* Building dashboards that display sync status across multiple queues

## Browser Support

The BackgroundSync Queue component relies on the Broadcast Channel API for communication with the service worker, which is widely supported in modern browsers.

**Support level:** Excellent - Works on all modern browsers that support service workers and the Broadcast Channel API (Chrome, Firefox, Safari 15.4+, Edge).

{% hint style="info" %}
This component requires:
- An active service worker with Workbox configured
- Background Sync enabled in your PWA configuration
- A broadcast channel configured for the queue
{% endhint %}

{% hint style="success" %}
Background Sync automatically retries failed requests when the device regains connectivity. This component simply provides visibility and manual control over that process.
{% endhint %}

## How It Works

1. **Service Worker Queue**: When requests fail, Workbox queues them in a background sync queue
2. **Broadcast Channel**: The service worker broadcasts queue status updates via a named channel
3. **Component Listening**: The BackgroundSync Queue component listens to the channel
4. **Status Display**: Component receives and displays the number of pending requests
5. **Manual Replay**: Users can trigger immediate queue replay via a button action

## Configuration

First, configure background sync with a broadcast channel in your PWA configuration:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            enabled: true
            background_sync:
                - queue_name: 'form-submissions'
                  broadcast_channel: 'form-queue-status'
                  max_retention_time: 10080  # 7 days in minutes
                - queue_name: 'api-requests'
                  broadcast_channel: 'api-queue-status'
                  max_retention_time: 4320   # 3 days in minutes
```
{% endcode %}

The `broadcast_channel` name must match the `channel` parameter in your component.

## Usage

### Basic Queue Status Display

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'form-queue-status'}) }}>
    <p>Pending form submissions: <strong id="queue-count">--</strong></p>
    <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>
        Retry Now
    </button>
</div>

<script type="module">
    const host = document.querySelector('[data-controller="pwa__backgroundsync-queue"]');
    const countEl = document.getElementById('queue-count');

    host.addEventListener('backgroundsync-queue:status', (e) => {
        if (Number.isInteger(e.detail.remaining)) {
            countEl.textContent = e.detail.remaining;

            if (e.detail.remaining === 0) {
                countEl.parentElement.classList.add('text-green-600');
                countEl.parentElement.classList.remove('text-yellow-600');
            } else {
                countEl.parentElement.classList.add('text-yellow-600');
                countEl.parentElement.classList.remove('text-green-600');
            }
        }
    });

    host.addEventListener('backgroundsync-queue:error', (e) => {
        console.error('Queue error:', e.detail.reason);
    });
</script>
```
{% endcode %}

### Visual Queue Status Indicator

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'form-queue-status'}) }}>
    <div id="sync-status" class="status-indicator hidden">
        <div class="status-icon">⏳</div>
        <div class="status-text">
            <strong id="queue-count">0</strong> item(s) waiting to sync
        </div>
        <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}
                class="retry-btn">
            Sync Now
        </button>
    </div>
</div>

<script type="module">
    const host = document.querySelector('[data-controller="pwa__backgroundsync-queue"]');
    const statusEl = document.getElementById('sync-status');
    const countEl = document.getElementById('queue-count');

    host.addEventListener('backgroundsync-queue:status', (e) => {
        const remaining = e.detail.remaining;

        if (Number.isInteger(remaining)) {
            countEl.textContent = remaining;

            if (remaining > 0) {
                statusEl.classList.remove('hidden');
            } else {
                statusEl.classList.add('hidden');
            }
        }
    });
</script>

<style>
    .status-indicator {
        display: flex;
        align-items: center;
        gap: 1rem;
        padding: 1rem;
        background: #fff3cd;
        border: 1px solid #ffc107;
        border-radius: 4px;
        margin: 1rem 0;
    }

    .status-icon {
        font-size: 2rem;
    }

    .retry-btn {
        margin-left: auto;
        padding: 0.5rem 1rem;
        background: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }

    .retry-btn:hover {
        background: #0056b3;
    }
</style>
```
{% endcode %}

### Multiple Queue Monitoring

{% code lineNumbers="true" %}
```twig
<div class="queue-dashboard">
    <h2>Sync Status</h2>

    {# Form submissions queue #}
    <div class="queue-item"
         {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'form-queue-status'}) }}
         data-queue="forms">
        <span class="queue-name">Form Submissions:</span>
        <span class="queue-count" data-target="count">--</span>
        <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>Retry</button>
    </div>

    {# API requests queue #}
    <div class="queue-item"
         {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'api-queue-status'}) }}
         data-queue="api">
        <span class="queue-name">API Requests:</span>
        <span class="queue-count" data-target="count">--</span>
        <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>Retry</button>
    </div>

    {# File uploads queue #}
    <div class="queue-item"
         {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'upload-queue-status'}) }}
         data-queue="uploads">
        <span class="queue-name">File Uploads:</span>
        <span class="queue-count" data-target="count">--</span>
        <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>Retry</button>
    </div>
</div>

<script type="module">
    document.querySelectorAll('.queue-item').forEach(queueEl => {
        const countEl = queueEl.querySelector('[data-target="count"]');

        queueEl.addEventListener('backgroundsync-queue:status', (e) => {
            const remaining = e.detail.remaining;
            if (Number.isInteger(remaining)) {
                countEl.textContent = remaining;

                if (remaining > 0) {
                    queueEl.classList.add('has-pending');
                } else {
                    queueEl.classList.remove('has-pending');
                }
            }
        });
    });
</script>

<style>
    .queue-dashboard {
        border: 1px solid #ddd;
        border-radius: 8px;
        padding: 1.5rem;
    }

    .queue-item {
        display: flex;
        align-items: center;
        gap: 1rem;
        padding: 1rem;
        margin: 0.5rem 0;
        background: #f8f9fa;
        border-radius: 4px;
    }

    .queue-item.has-pending {
        background: #fff3cd;
        border-left: 4px solid #ffc107;
    }

    .queue-name {
        flex: 1;
        font-weight: 500;
    }

    .queue-count {
        font-weight: bold;
        min-width: 3ch;
        text-align: center;
    }
</style>
```
{% endcode %}

### Automatic Polling for Status Updates

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'form-queue-status'}) }}
     id="sync-monitor">
    <div class="sync-header">
        <h3>Sync Queue</h3>
        <span id="last-update">Never</span>
    </div>
    <div class="sync-body">
        <p>Pending items: <strong id="pending-count">--</strong></p>
        <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>
            Sync Now
        </button>
    </div>
</div>

<script type="module">
    const host = document.querySelector('[data-controller="pwa__backgroundsync-queue"]');
    const countEl = document.getElementById('pending-count');
    const updateEl = document.getElementById('last-update');

    // Request status update every 30 seconds
    setInterval(() => {
        // The component automatically requests status on connect
        // For manual refresh, we can reconnect the controller
        if (host.controller && host.controller.bc) {
            host.controller.bc.postMessage({ type: 'status-request' });
        }
    }, 30000);

    host.addEventListener('backgroundsync-queue:status', (e) => {
        if (Number.isInteger(e.detail.remaining)) {
            countEl.textContent = e.detail.remaining;
            updateEl.textContent = new Date().toLocaleTimeString();
        }
    });
</script>
```
{% endcode %}

### Integration with Form Submission

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'form-queue-status'}) }}>
    {# Show queue status banner if items are pending #}
    <div id="sync-banner" class="hidden alert alert-warning">
        <p>You have <strong id="queue-size">0</strong> form(s) waiting to sync.</p>
        <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>
            Try Syncing Now
        </button>
    </div>

    {# Regular form #}
    <form action="/submit" method="POST">
        <input type="text" name="title" required>
        <textarea name="description" required></textarea>
        <button type="submit">Submit</button>
    </form>
</div>

<script type="module">
    const host = document.querySelector('[data-controller="pwa__backgroundsync-queue"]');
    const banner = document.getElementById('sync-banner');
    const sizeEl = document.getElementById('queue-size');

    host.addEventListener('backgroundsync-queue:status', (e) => {
        const remaining = e.detail.remaining;

        if (Number.isInteger(remaining)) {
            sizeEl.textContent = remaining;

            if (remaining > 0) {
                banner.classList.remove('hidden');
            } else {
                banner.classList.add('hidden');
            }
        }
    });

    // Show success message when queue is emptied
    let previousCount = null;
    host.addEventListener('backgroundsync-queue:status', (e) => {
        const current = e.detail.remaining;

        if (previousCount > 0 && current === 0) {
            alert('All pending forms have been synced successfully!');
        }

        previousCount = current;
    });
</script>
```
{% endcode %}

## Best Practices

### Always Configure Broadcast Channel

{% hint style="warning" %}
The broadcast channel must be configured in both the PWA config file and the component. Mismatched channel names will prevent communication between the service worker and the component.
{% endhint %}

{% code lineNumbers="true" %}
```yaml
# config/packages/pwa.yaml
pwa:
    serviceworker:
        workbox:
            background_sync:
                - queue_name: 'my-queue'
                  broadcast_channel: 'my-queue-status'  # Must match component
```
{% endcode %}

{% code lineNumbers="true" %}
```twig
{# Channel name must match configuration #}
<div {{ stimulus_controller('@pwa/backgroundsync-queue', {
    channel: 'my-queue-status'
}) }}>
```
{% endcode %}

### Handle Edge Cases

{% hint style="info" %}
Always check if the `remaining` value is an integer before displaying it. The initial value may be undefined until the first status update.
{% endhint %}

{% code lineNumbers="true" %}
```javascript
host.addEventListener('backgroundsync-queue:status', (e) => {
    // Always validate the data type
    if (Number.isInteger(e.detail.remaining)) {
        updateDisplay(e.detail.remaining);
    } else {
        // Show loading state
        updateDisplay('--');
    }
});
```
{% endcode %}

### Provide User Feedback

{% hint style="success" %}
Always give users feedback when they trigger a manual replay. Even if the sync fails, users should know something happened.
{% endhint %}

{% code lineNumbers="true" %}
```javascript
const replayBtn = document.querySelector('button');
let isReplaying = false;

replayBtn.addEventListener('click', () => {
    if (isReplaying) return;

    isReplaying = true;
    replayBtn.disabled = true;
    replayBtn.textContent = 'Syncing...';

    // Reset after a few seconds (service worker will handle the actual sync)
    setTimeout(() => {
        isReplaying = false;
        replayBtn.disabled = false;
        replayBtn.textContent = 'Sync Now';
    }, 3000);
});
```
{% endcode %}

### Monitor Multiple Queues Separately

{% hint style="info" %}
If your application has multiple background sync queues (forms, API calls, uploads), use separate component instances with different channel names for each queue.
{% endhint %}

## Common Use Cases

### 1. Offline Form Submission Tracker

Track pending form submissions and show users what's waiting to sync:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'form-submissions'}) }}>
    <div class="submission-tracker">
        <div id="offline-notice" class="hidden">
            <h4>⚠️ You're currently offline</h4>
            <p>Your form submissions will be sent automatically when you're back online.</p>
            <p>Pending submissions: <strong id="pending">0</strong></p>
            <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>
                Try Sending Now
            </button>
        </div>
    </div>
</div>

<script type="module">
    const host = document.querySelector('[data-controller="pwa__backgroundsync-queue"]');
    const notice = document.getElementById('offline-notice');
    const pending = document.getElementById('pending');

    host.addEventListener('backgroundsync-queue:status', (e) => {
        const count = e.detail.remaining;

        if (Number.isInteger(count) && count > 0) {
            pending.textContent = count;
            notice.classList.remove('hidden');
        } else {
            notice.classList.add('hidden');
        }
    });

    // Also check online status
    window.addEventListener('online', () => {
        notice.querySelector('h4').textContent = '✅ Back online!';
        setTimeout(() => {
            if (host.controller && host.controller.bc) {
                host.controller.bc.postMessage({ type: 'status-request' });
            }
        }, 1000);
    });
</script>
```
{% endcode %}

### 2. E-Commerce Offline Checkout

Show customers their pending orders waiting to process:

{% code lineNumbers="true" %}
```twig
<div class="checkout-status"
     {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'order-queue'}) }}>
    <div id="pending-orders" class="hidden">
        <div class="alert alert-info">
            <h5>Pending Orders</h5>
            <p>You have <strong id="order-count">0</strong> order(s) waiting to be processed.</p>
            <p class="text-sm">Orders will be submitted automatically when connection is restored.</p>
            <button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}
                    class="btn btn-primary btn-sm">
                Submit Orders Now
            </button>
        </div>
    </div>
</div>

<script type="module">
    const host = document.querySelector('[data-controller="pwa__backgroundsync-queue"]');
    const container = document.getElementById('pending-orders');
    const count = document.getElementById('order-count');

    host.addEventListener('backgroundsync-queue:status', (e) => {
        if (Number.isInteger(e.detail.remaining)) {
            count.textContent = e.detail.remaining;

            if (e.detail.remaining > 0) {
                container.classList.remove('hidden');
            } else {
                container.classList.add('hidden');
            }
        }
    });
</script>
```
{% endcode %}

### 3. Todo/Task Management Offline Sync

Display pending task changes waiting to sync with the server:

{% code lineNumbers="true" %}
```twig
<div class="task-manager">
    <div {{ stimulus_controller('@pwa/backgroundsync-queue', {channel: 'task-sync'}) }}
         class="sync-indicator">
        <span id="sync-status" class="badge badge-secondary">Synced</span>
    </div>

    <div class="task-list">
        {# Task items here #}
    </div>
</div>

<script type="module">
    const host = document.querySelector('[data-controller="pwa__backgroundsync-queue"]');
    const status = document.getElementById('sync-status');

    host.addEventListener('backgroundsync-queue:status', (e) => {
        const pending = e.detail.remaining;

        if (Number.isInteger(pending)) {
            if (pending > 0) {
                status.textContent = `Syncing... (${pending} pending)`;
                status.className = 'badge badge-warning';
            } else {
                status.textContent = 'Synced';
                status.className = 'badge badge-success';
            }
        }
    });
</script>
```
{% endcode %}

## API Reference

### Values

#### `channel`

**Type:** `String`
**Required:** Yes

The name of the broadcast channel to listen to. Must match the `broadcast_channel` configured in your PWA configuration file.

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/backgroundsync-queue', {
    channel: 'form-queue-status'
}) }}>
```
{% endcode %}

### Actions

#### `replay()`

Manually triggers replay of all pending requests in the queue. Sends a `replay-request` message to the service worker via the broadcast channel.

{% code lineNumbers="true" %}
```twig
<button {{ stimulus_action('@pwa/backgroundsync-queue', 'replay') }}>
    Retry Now
</button>
```
{% endcode %}

### Targets

None

### Events

#### `status`

Fired when a status update is received from the service worker containing the current queue state.

**Event detail:**
- `name` (string): Name of the queue
- `remaining` (number): Number of pending requests in the queue

{% code lineNumbers="true" %}
```javascript
host.addEventListener('backgroundsync-queue:status', (e) => {
    console.log(`Queue ${e.detail.name} has ${e.detail.remaining} pending requests`);
});
```
{% endcode %}

#### `error`

Fired when an error occurs (e.g., no channel provided).

**Event detail:**
- `reason` (string): Description of the error

{% code lineNumbers="true" %}
```javascript
host.addEventListener('backgroundsync-queue:error', (e) => {
    console.error('Queue error:', e.detail.reason);
});
```
{% endcode %}

## How Queue Updates Work

The component automatically:

1. **On Connect**: Opens a BroadcastChannel connection with the specified channel name
2. **Status Request**: Immediately sends a `status-request` message to the service worker
3. **Listen for Updates**: Listens for status messages from the service worker
4. **Dispatch Events**: Emits `status` events with queue information
5. **On Disconnect**: Closes the broadcast channel connection

The service worker broadcasts status updates:
- When requests are added to the queue
- When requests are successfully synced
- When a `status-request` is received
- When a `replay-request` is processed

## Related Components

- [BackgroundSync Form](backgroundsync-form.md) - Automatically queue form submissions when offline
- [Service Worker](service-worker.md) - Service worker configuration and setup
- [Connection Status](connection-status.md) - Monitor online/offline status
- [Sync Broadcast](sync-broadcast.md) - Generic broadcast channel communication

## Resources

- [MDN: Background Sync API](https://developer.mozilla.org/en-US/docs/Web/API/Background_Synchronization_API)
- [MDN: Broadcast Channel API](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)
- [Workbox: Background Sync](https://developer.chrome.com/docs/workbox/modules/workbox-background-sync/)
- [PWA Bundle Service Worker Configuration](../the-service-worker/workbox/backgoundsync.md)

# Service Worker

The Service Worker controller provides useful methods to facilitate interaction with the service worker lifecycle. It enables you to manually update the service worker and notify users when updates are available, particularly useful when the [`skipWaiting`](../the-service-worker/configuration.md#skip-waiting) option is set to `false`.

This component is essential for:

* Notifying users about available application updates
* Providing manual update controls
* Ensuring users get the latest features without forced reloads
* Creating a smooth update experience without interrupting user workflow

{% hint style="info" %}
When `skipWaiting` is enabled, the service worker automatically activates when updated. When disabled, you have more control over when the update is applied, which is useful to avoid interrupting user activities.
{% endhint %}

## Usage

### Basic Update Button

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/service-worker') }}>
    <button
        id="update-btn"
        {{ stimulus_action('@pwa/service-worker', 'update', 'click') }}
        style="display:none;"
    >
        Update Available - Click to Refresh
    </button>
</div>

<script>
    document.addEventListener('pwa--service-worker:update-available', () => {
        document.getElementById('update-btn').style.display = 'block';
    });
</script>
```
{% endcode %}

### Update Notification Banner

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/service-worker') }}>
    <div id="update-banner" class="update-notification" style="display:none;">
        <p>A new version of this app is available!</p>
        <button {{ stimulus_action('@pwa/service-worker', 'update', 'click') }}>
            Update Now
        </button>
        <button onclick="document.getElementById('update-banner').style.display='none'">
            Later
        </button>
    </div>
</div>

<script>
    document.addEventListener('pwa--service-worker:update-available', () => {
        document.getElementById('update-banner').style.display = 'block';
    });
</script>

<style>
    .update-notification {
        position: fixed;
        bottom: 20px;
        left: 50%;
        transform: translateX(-50%);
        background: #007bff;
        color: white;
        padding: 15px 20px;
        border-radius: 8px;
        box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        z-index: 1000;
    }

    .update-notification button {
        margin: 5px;
        padding: 8px 15px;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
</style>
```
{% endcode %}

### Toast Notification with Countdown

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/service-worker') }}>
    <div id="update-toast" class="toast" style="display:none;">
        <span id="toast-message">New version available!</span>
        <button {{ stimulus_action('@pwa/service-worker', 'update', 'click') }}>
            Update
        </button>
    </div>
</div>

<script>
    document.addEventListener('pwa--service-worker:update-available', () => {
        const toast = document.getElementById('update-toast');
        const message = document.getElementById('toast-message');

        toast.style.display = 'flex';

        // Optional: Auto-hide after 10 seconds
        let countdown = 10;
        const interval = setInterval(() => {
            countdown--;
            message.textContent = `New version available! Auto-dismissing in ${countdown}s`;

            if (countdown <= 0) {
                clearInterval(interval);
                toast.style.display = 'none';
            }
        }, 1000);
    });
</script>
```
{% endcode %}

### Integration with User Settings

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/service-worker') }}>
    <!-- Settings page -->
    <div class="settings">
        <h2>Application Settings</h2>

        <label>
            <input type="checkbox" id="auto-update" checked>
            Automatically install updates
        </label>

        <button id="manual-update-btn" style="display:none;">
            Update Available
        </button>
    </div>
</div>

<script>
    document.addEventListener('pwa--service-worker:update-available', () => {
        const autoUpdate = document.getElementById('auto-update').checked;
        const updateBtn = document.getElementById('manual-update-btn');

        if (autoUpdate) {
            // Auto-update: show brief notification and update
            console.log('Auto-updating to new version...');
            // Trigger update automatically
            document.querySelector('[data-action*="update"]').click();
        } else {
            // Manual update: show button
            updateBtn.style.display = 'block';
            updateBtn.textContent = 'Update Available - Click to Install';
        }
    });
</script>
```
{% endcode %}

## Parameters

None

## Actions

### `update`

Triggers the service worker update process. When called:
1. The new service worker becomes active
2. The page automatically refreshes to load the updated version
3. Users immediately see the latest features and fixes

```twig
{{ stimulus_action('@pwa/service-worker', 'update', 'click') }}
```

{% hint style="warning" %}
Calling this action will refresh the current page. Ensure users are aware of this behavior and have saved any unsaved work before triggering the update.
{% endhint %}

## Targets

None

## Events

### `pwa--service-worker:update-available`

Dispatched when a new version of the service worker is detected and ready to be installed.

**No payload**

This event is fired only when:
* A new service worker has finished installing
* The new service worker is waiting to activate
* The `skipWaiting` option is set to `false` in the [service worker configuration](../the-service-worker/configuration.md#skip-waiting)

Example usage:
```javascript
document.addEventListener('pwa--service-worker:update-available', () => {
    // Show update notification
    showUpdateBanner();

    // Log for debugging
    console.log('Service worker update available');

    // Optional: Ask user preference
    if (confirm('New version available. Update now?')) {
        // Trigger update
        document.querySelector('[data-action*="update"]').click();
    }
});
```

## Best Practices

1. **Clear communication**: Always inform users that updating will refresh the page
2. **Save user work**: If your app has forms or unsaved data, warn users before updating
3. **Provide options**: Let users choose to update now or later
4. **Visual feedback**: Use prominent but non-intrusive UI elements like banners or toasts
5. **Graceful timing**: Consider detecting idle time before prompting for updates
6. **Persistent notification**: Keep the update notification visible until user acts on it

## Configuration Reference

This component works in conjunction with the service worker configuration. To enable manual updates, ensure `skipWaiting` is set to `false`:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        skip_waiting: false  # Allows manual control over updates
```
{% endcode %}

For more information on service worker configuration, see the [Service Worker Configuration](../the-service-worker/configuration.md) page.

## Advanced Example: Update Manager

Here's a complete example of an update manager with user preferences and smart update timing:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/service-worker') }}>
    <div id="update-manager" style="display:none;">
        <div class="update-card">
            <h3>📦 Update Available</h3>
            <p>A new version of the app is ready to install.</p>

            <div class="update-options">
                <button
                    class="btn-primary"
                    {{ stimulus_action('@pwa/service-worker', 'update', 'click') }}
                >
                    Update Now
                </button>
                <button class="btn-secondary" onclick="snoozeUpdate()">
                    Remind Me in 1 Hour
                </button>
            </div>
        </div>
    </div>
</div>

<script>
    let updateSnoozed = false;

    document.addEventListener('pwa--service-worker:update-available', () => {
        if (updateSnoozed) return;

        // Check if user is idle (no interaction for 30 seconds)
        let idleTime = 0;
        const checkIdle = setInterval(() => {
            idleTime++;
            if (idleTime >= 30) {
                clearInterval(checkIdle);
                showUpdateManager();
            }
        }, 1000);

        // Reset idle time on user interaction
        ['mousedown', 'keypress', 'scroll', 'touchstart'].forEach(event => {
            document.addEventListener(event, () => {
                idleTime = 0;
            }, { once: true });
        });
    });

    function showUpdateManager() {
        document.getElementById('update-manager').style.display = 'block';
    }

    function snoozeUpdate() {
        document.getElementById('update-manager').style.display = 'none';
        updateSnoozed = true;

        // Re-enable after 1 hour
        setTimeout(() => {
            updateSnoozed = false;
            showUpdateManager();
        }, 3600000);
    }
</script>
```
{% endcode %}

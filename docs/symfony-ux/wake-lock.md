# Wake Lock

The Wake Lock component provides an interface to the Screen Wake Lock API, enabling your Progressive Web App to prevent devices from dimming or locking the screen when the app needs to keep running. This is particularly useful for applications that require continuous user attention without interaction.

This component is useful for applications such as:

* Recipe or cooking apps that need to keep instructions visible
* Presentation or slideshow applications
* Video or audio playback interfaces
* Reading apps for long-form content
* Fitness or workout apps displaying exercise instructions
* Navigation or map applications

## Browser Support

The Screen Wake Lock API is supported in most modern browsers including Chrome, Edge, and Safari. Always check browser compatibility and handle the case where the API is not available.

{% hint style="info" %}
Wake locks are automatically released when the page is not visible or when the user minimizes the browser.
{% endhint %}

## Usage

### Basic Wake Lock Toggle

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/wake-lock') }}>
    <button {{ stimulus_action('@pwa/wake-lock', 'toggle', 'click') }}>
        Toggle Screen Wake Lock
    </button>

    <p id="status">Wake lock status: <span>Not active</span></p>
</div>

<script>
    document.addEventListener('pwa--wake-lock:updated', (event) => {
        const status = event.detail.wakeLock;
        const statusText = status && !status.released ? 'Active' : 'Not active';
        document.querySelector('#status span').textContent = statusText;
    });
</script>
```
{% endcode %}

### Explicit Lock and Unlock

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/wake-lock') }}>
    <button {{ stimulus_action('@pwa/wake-lock', 'lock', 'click') }}>
        Keep Screen Awake
    </button>

    <button {{ stimulus_action('@pwa/wake-lock', 'unlock', 'click') }}>
        Release Wake Lock
    </button>

    <div id="wake-lock-status" class="status-indicator"></div>
</div>

<script>
    document.addEventListener('pwa--wake-lock:updated', (event) => {
        const statusDiv = document.getElementById('wake-lock-status');
        const wakeLock = event.detail.wakeLock;

        if (wakeLock && !wakeLock.released) {
            statusDiv.textContent = '🔒 Screen will stay awake';
            statusDiv.className = 'status-indicator active';
        } else {
            statusDiv.textContent = '💤 Screen can sleep normally';
            statusDiv.className = 'status-indicator inactive';
        }
    });
</script>
```
{% endcode %}

### Recipe App Example

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/wake-lock') }} class="recipe-viewer">
    <div class="recipe-header">
        <h1>Chocolate Chip Cookies</h1>
        <label class="keep-awake-toggle">
            <input type="checkbox" {{ stimulus_action('@pwa/wake-lock', 'toggle', 'change') }}>
            Keep screen awake while cooking
        </label>
    </div>

    <div class="recipe-content">
        <!-- Recipe steps here -->
    </div>
</div>

<script>
    document.addEventListener('pwa--wake-lock:updated', (event) => {
        const checkbox = document.querySelector('.keep-awake-toggle input');
        const wakeLock = event.detail.wakeLock;

        // Update checkbox state
        checkbox.checked = wakeLock && !wakeLock.released;

        // Show notification when wake lock is active
        if (wakeLock && !wakeLock.released) {
            console.log('Screen will stay awake while you cook!');
        }
    });
</script>
```
{% endcode %}

## Parameters

None

## Actions

### `lock`

Requests a screen wake lock to prevent the device from dimming or locking the screen.

```twig
{{ stimulus_action('@pwa/wake-lock', 'lock', 'click') }}
```

If the wake lock cannot be obtained (e.g., browser doesn't support the API or user denied permission), the action fails silently. Listen to the `updated` event to check the actual state.

### `unlock`

Releases the current wake lock, allowing the screen to dim and lock normally.

```twig
{{ stimulus_action('@pwa/wake-lock', 'unlock', 'click') }}
```

### `toggle`

Toggles the wake lock state. If currently locked, it unlocks. If unlocked, it locks.

```twig
{{ stimulus_action('@pwa/wake-lock', 'toggle', 'click') }}
```

This is convenient for checkbox or toggle button implementations.

## Targets

None

## Events

### `pwa--wake-lock:updated`

Dispatched whenever the wake lock state changes (locked, unlocked, or released by the system).

**Payload**: `{wakeLock}`
* `wakeLock` is `null` when no wake lock is active
* `wakeLock` is an object when active, containing:
  * `type` (string): The type of wake lock (usually `"screen"`)
  * `released` (boolean): Whether the wake lock has been released

Example:
```javascript
document.addEventListener('pwa--wake-lock:updated', (event) => {
    const { wakeLock } = event.detail;

    if (wakeLock === null) {
        console.log('No wake lock active');
    } else if (wakeLock.released) {
        console.log('Wake lock was released');
    } else {
        console.log('Wake lock is active, type:', wakeLock.type);
    }
});
```

## Best Practices

1. **Always provide UI feedback**: Show users when the wake lock is active so they understand why their screen isn't dimming
2. **Make it optional**: Allow users to enable/disable the wake lock based on their preference
3. **Handle visibility changes**: The browser automatically releases wake locks when the page is hidden, but you should handle this gracefully
4. **Don't abuse it**: Only use wake locks when truly necessary for the user experience
5. **Provide a toggle**: Use the `toggle` action with checkboxes or switches for better UX
6. **Consider battery impact**: Inform users that keeping the screen awake will consume more battery

## Common Pitfalls

* **Wake lock requires user interaction**: The lock can only be requested after a user gesture (click, touch, etc.)
* **Automatic release**: Wake locks are automatically released when the page becomes hidden (user switches tabs or minimizes browser)
* **No persistent wake locks**: Wake locks don't persist across page reloads
* **HTTPS required**: Like most modern web APIs, the Screen Wake Lock API requires a secure context (HTTPS)

## Complete Example

Here's a complete example for a meditation timer app:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/wake-lock') }} class="meditation-timer">
    <h2>Meditation Timer</h2>

    <div class="timer-display">
        <span id="time">10:00</span>
    </div>

    <div class="controls">
        <button id="start-btn">Start Meditation</button>
        <button id="stop-btn" style="display:none;">Stop</button>
    </div>

    <label class="wake-lock-option">
        <input
            type="checkbox"
            id="keep-awake"
            {{ stimulus_action('@pwa/wake-lock', 'toggle', 'change') }}
            checked
        >
        Keep screen awake during meditation
    </label>

    <div id="status"></div>
</div>

<script>
    let isActive = false;

    document.getElementById('start-btn').addEventListener('click', () => {
        isActive = true;
        document.getElementById('start-btn').style.display = 'none';
        document.getElementById('stop-btn').style.display = 'block';

        // Auto-enable wake lock when starting
        if (document.getElementById('keep-awake').checked) {
            // Wake lock will be triggered via the checkbox being checked
            console.log('Starting meditation with wake lock active');
        }
    });

    document.getElementById('stop-btn').addEventListener('click', () => {
        isActive = false;
        document.getElementById('stop-btn').style.display = 'none';
        document.getElementById('start-btn').style.display = 'block';
    });

    document.addEventListener('pwa--wake-lock:updated', (event) => {
        const statusDiv = document.getElementById('status');
        const wakeLock = event.detail.wakeLock;
        const checkbox = document.getElementById('keep-awake');

        checkbox.checked = wakeLock && !wakeLock.released;

        if (isActive && wakeLock && !wakeLock.released) {
            statusDiv.textContent = '🧘 Screen will stay on during your meditation';
            statusDiv.className = 'status-success';
        } else if (isActive) {
            statusDiv.textContent = '⚠️ Screen may turn off during meditation';
            statusDiv.className = 'status-warning';
        } else {
            statusDiv.textContent = '';
        }
    });
</script>
```
{% endcode %}

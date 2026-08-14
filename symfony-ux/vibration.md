# Vibration

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Vibration component provides an interface to the Vibration API, allowing your Progressive Web App to trigger haptic feedback on devices that support it. This creates tactile sensations that enhance user experience and provide physical feedback for actions.

This component is particularly useful for:

* Gaming applications requiring haptic feedback
* Notification and alert systems
* Form validation feedback
* Interactive buttons and controls
* Accessibility features for users with visual impairments
* Timers and alarm applications

## Browser Support

The Vibration API is supported on most Android devices in Chrome, Firefox, and Edge. iOS devices (iPhone, iPad) do not support the Vibration API due to platform restrictions.

{% hint style="info" %}
Always check for API support and provide visual alternatives for devices that don't support vibration.
{% endhint %}

## Usage

### Basic Vibration

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <button {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [200]}) }}>
        Short Vibration (200ms)
    </button>

    <button {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [500]}) }}>
        Long Vibration (500ms)
    </button>
</div>

<script>
    document.addEventListener('pwa--vibration:triggered', () => {
        console.log('Vibration started');
    });
</script>
```
{% endcode %}

### Vibration Patterns

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <h3>Vibration Patterns</h3>

    <!-- Simple pulse -->
    <button {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [100, 50, 100]}) }}>
        Double Tap
    </button>

    <!-- SOS pattern -->
    <button {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {
        pattern: [100, 100, 100, 100, 100, 200, 200, 100, 200, 100, 200, 200, 100, 100, 100, 100, 100]
    }) }}>
        SOS
    </button>

    <!-- Heartbeat pattern -->
    <button {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [100, 50, 100, 500]}) }}>
        Heartbeat
    </button>

    <!-- Notification pattern -->
    <button {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [50, 100, 50, 100, 50]}) }}>
        Notification
    </button>
</div>
```
{% endcode %}

### Persistent Vibration with Interval

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <h3>Repeating Vibration</h3>

    <!-- Start repeating vibration -->
    <button {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {
        pattern: [200, 100, 200],
        interval: 2000
    }) }}>
        Start Repeating (every 2s)
    </button>

    <!-- Stop vibration -->
    <button {{ stimulus_action('@pwa/vibration', 'stop', 'click') }}>
        Stop Vibration
    </button>
</div>

<script>
    document.addEventListener('pwa--vibration:triggered', () => {
        console.log('Vibration started');
    });

    document.addEventListener('pwa--vibration:stopped', () => {
        console.log('Vibration stopped');
    });
</script>
```
{% endcode %}

### Form Validation Feedback

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <form id="login-form">
        <input type="email" id="email" placeholder="Email" required>
        <input type="password" id="password" placeholder="Password" required>
        <button type="submit">Login</button>
    </form>
</div>

<script>
    const form = document.getElementById('login-form');
    const controller = document.querySelector('[data-controller="@pwa/vibration"]');

    form.addEventListener('submit', (e) => {
        e.preventDefault();

        const email = document.getElementById('email').value;
        const password = document.getElementById('password').value;

        if (!email || !password) {
            // Error vibration - three short pulses
            controller.dispatchEvent(new CustomEvent('vibrate', {
                detail: {
                    params: {
                        pattern: [50, 50, 50, 50, 50]
                    }
                }
            }));
            alert('Please fill in all fields');
        } else {
            // Success vibration - single smooth pulse
            controller.dispatchEvent(new CustomEvent('vibrate', {
                detail: {
                    params: {
                        pattern: [100]
                    }
                }
            }));
            // Process login...
        }
    });
</script>
```
{% endcode %}

### Gaming Controls

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <div class="game-container">
        <canvas id="game-canvas"></canvas>

        <div class="game-controls">
            <button id="shoot-btn">Shoot</button>
            <button id="jump-btn">Jump</button>
        </div>
    </div>
</div>

<script>
    const controller = document.querySelector('[data-controller="@pwa/vibration"]');

    function vibratePattern(pattern) {
        controller.dispatchEvent(new CustomEvent('vibrate', {
            detail: { params: { pattern } }
        }));
    }

    // Shoot action - quick sharp vibration
    document.getElementById('shoot-btn').addEventListener('click', () => {
        vibratePattern([30]);
        // Game logic for shooting
    });

    // Jump action - light bounce vibration
    document.getElementById('jump-btn').addEventListener('click', () => {
        vibratePattern([50, 50, 30]);
        // Game logic for jumping
    });

    // Collision event - strong impact vibration
    function onCollision() {
        vibratePattern([100, 50, 100]);
    }

    // Power-up collected - ascending vibration
    function onPowerUp() {
        vibratePattern([50, 30, 70, 30, 90]);
    }

    // Game over - descending vibration
    function onGameOver() {
        vibratePattern([200, 100, 150, 100, 100]);
    }
</script>
```
{% endcode %}

### Timer/Alarm Application

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <div class="timer-app">
        <h2>Pomodoro Timer</h2>
        <div id="timer-display">25:00</div>

        <button id="start-timer">Start</button>
        <button id="stop-timer">Stop</button>
    </div>
</div>

<script>
    const controller = document.querySelector('[data-controller="@pwa/vibration"]');
    let timerInterval;
    let timeLeft = 25 * 60; // 25 minutes in seconds

    document.getElementById('start-timer').addEventListener('click', () => {
        timerInterval = setInterval(() => {
            timeLeft--;
            updateDisplay();

            if (timeLeft <= 0) {
                clearInterval(timerInterval);
                onTimerComplete();
            }
        }, 1000);
    });

    document.getElementById('stop-timer').addEventListener('click', () => {
        clearInterval(timerInterval);
        controller.dispatchEvent(new CustomEvent('stop'));
    });

    function updateDisplay() {
        const mins = Math.floor(timeLeft / 60);
        const secs = timeLeft % 60;
        document.getElementById('timer-display').textContent =
            `${mins}:${secs.toString().padStart(2, '0')}`;
    }

    function onTimerComplete() {
        // Alarm vibration pattern - repeating pulses
        controller.dispatchEvent(new CustomEvent('vibrate', {
            detail: {
                params: {
                    pattern: [300, 200, 300, 200, 300],
                    interval: 2000  // Repeat every 2 seconds
                }
            }
        }));

        alert('Timer complete!');
    }
</script>
```
{% endcode %}

### Accessibility Features

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <div class="accessible-navigation">
        <button
            class="nav-item"
            {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [20]}) }}
        >
            Home
        </button>

        <button
            class="nav-item"
            {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [20]}) }}
        >
            Settings
        </button>

        <button
            class="nav-item active"
            {{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [40, 20, 40]}) }}
        >
            Current Page (double pulse)
        </button>
    </div>
</div>

<script>
    // Provide haptic feedback for important UI elements
    const importantButtons = document.querySelectorAll('.important-action');
    const controller = document.querySelector('[data-controller="@pwa/vibration"]');

    importantButtons.forEach(button => {
        button.addEventListener('click', () => {
            // Distinctive vibration for important actions
            controller.dispatchEvent(new CustomEvent('vibrate', {
                detail: {
                    params: {
                        pattern: [50, 30, 50, 30, 50]
                    }
                }
            }));
        });
    });
</script>
```
{% endcode %}

## Parameters

None

## Actions

### `vibrate`

Triggers a vibration pattern on the device.

**Parameters:**
* `pattern` (Array of numbers): Vibration pattern in milliseconds. Odd indices are vibration durations, even indices are pause durations.
  * Single value: `[200]` - vibrate for 200ms
  * Pattern: `[100, 50, 100]` - vibrate 100ms, pause 50ms, vibrate 100ms
* `interval` (number, optional): Interval in milliseconds to repeat the pattern. If specified, the vibration will repeat until stopped or the page is closed.

```twig
{{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [200, 100, 200]}) }}
```

With interval for repeating vibration:
```twig
{{ stimulus_action('@pwa/vibration', 'vibrate', 'click', {pattern: [100], interval: 1000}) }}
```

{% hint style="info" %}
Pattern values are specified in milliseconds. The first value is always a vibration duration. See [MDN documentation](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/vibrate#pattern) for more details.
{% endhint %}

### `stop`

Stops any ongoing vibration, including persistent vibrations started with an interval.

```twig
{{ stimulus_action('@pwa/vibration', 'stop', 'click') }}
```

## Targets

None

## Events

### `pwa--vibration:triggered`

Dispatched when a vibration pattern is triggered.

**No payload**

Example:
```javascript
document.addEventListener('pwa--vibration:triggered', () => {
    console.log('Device is vibrating');
    // Show visual indicator
    document.body.classList.add('vibrating');
    setTimeout(() => {
        document.body.classList.remove('vibrating');
    }, 500);
});
```

### `pwa--vibration:stopped`

Dispatched when vibration is explicitly stopped (not when a pattern naturally completes).

**No payload**

Example:
```javascript
document.addEventListener('pwa--vibration:stopped', () => {
    console.log('Vibration stopped');
    // Clear any visual indicators
    document.body.classList.remove('vibrating');
});
```

## Best Practices

1. **Be conservative**: Use vibration sparingly to avoid annoying users
2. **Keep it short**: Most vibration patterns should be under 500ms total
3. **Provide visual feedback**: Always accompany vibration with visual feedback
4. **Check support**: Detect if the API is available before relying on it
5. **Respect battery**: Excessive vibration drains battery quickly
6. **User preferences**: Consider providing an option to disable vibrations
7. **Context-appropriate**: Use different patterns for different types of feedback

## Common Vibration Patterns

Here are some commonly used vibration patterns:

```javascript
// Notification/Alert
const notification = [50, 50, 50, 50, 50];

// Success
const success = [100];

// Error
const error = [50, 50, 50, 50, 50, 50, 50];

// Warning
const warning = [200, 100, 200];

// Click/Tap
const tap = [20];

// Double-tap
const doubleTap = [30, 50, 30];

// Long press
const longPress = [100];

// Heartbeat
const heartbeat = [100, 50, 100, 450];

// SOS (... --- ...)
const sos = [
    100, 100, 100, 100, 100,  // ...
    200,
    200, 100, 200, 100, 200,  // ---
    200,
    100, 100, 100, 100, 100   // ...
];
```

## Device Considerations

### Android

* Full support in Chrome, Firefox, and Edge
* Vibration strength controlled by system settings
* Users can disable vibration in system settings

### iOS (iPhone/iPad)

* **Not supported** - iOS does not expose the Vibration API to web applications
* System haptics are only available to native apps
* Always provide visual alternatives

### Desktop

* Limited support - most laptops don't have vibration motors
* Some gaming devices with haptic feedback may work
* Not reliable for desktop PWAs

## Battery Impact

Vibration uses power from both the motor and the processor. Consider these guidelines:

* **Short vibrations (< 100ms)**: Minimal impact
* **Medium vibrations (100-500ms)**: Moderate impact
* **Long vibrations (> 500ms)**: Significant impact
* **Repeating vibrations**: High impact, avoid if possible

## Testing Vibration

```javascript
// Check if Vibration API is supported
if ('vibrate' in navigator) {
    console.log('Vibration API is supported');

    // Test vibration
    navigator.vibrate(200);
} else {
    console.log('Vibration API is not supported');
    // Provide visual-only feedback
}
```

## Complete Example: Interactive Notification System

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/vibration') }}>
    <div class="notification-system">
        <h2>Notification Center</h2>

        <div class="notification-types">
            <button id="info-notif">Info</button>
            <button id="success-notif">Success</button>
            <button id="warning-notif">Warning</button>
            <button id="error-notif">Error</button>
        </div>

        <div id="notification-display"></div>

        <label>
            <input type="checkbox" id="vibration-enabled" checked>
            Enable Vibration Feedback
        </label>
    </div>
</div>

<script>
    const controller = document.querySelector('[data-controller="@pwa/vibration"]');
    const display = document.getElementById('notification-display');
    const vibrationEnabled = document.getElementById('vibration-enabled');

    const patterns = {
        info: [50],
        success: [100],
        warning: [100, 50, 100],
        error: [50, 50, 50, 50, 50, 50, 50]
    };

    const colors = {
        info: '#3b82f6',
        success: '#10b981',
        warning: '#f59e0b',
        error: '#ef4444'
    };

    function showNotification(type, message) {
        // Visual feedback
        display.innerHTML = `
            <div class="notification ${type}" style="background: ${colors[type]}">
                <strong>${type.toUpperCase()}</strong>
                <p>${message}</p>
            </div>
        `;

        // Haptic feedback
        if (vibrationEnabled.checked && 'vibrate' in navigator) {
            controller.dispatchEvent(new CustomEvent('vibrate', {
                detail: {
                    params: {
                        pattern: patterns[type]
                    }
                }
            }));
        }

        // Auto-dismiss after 3 seconds
        setTimeout(() => {
            display.innerHTML = '';
        }, 3000);
    }

    document.getElementById('info-notif').addEventListener('click', () => {
        showNotification('info', 'New message received');
    });

    document.getElementById('success-notif').addEventListener('click', () => {
        showNotification('success', 'Operation completed successfully');
    });

    document.getElementById('warning-notif').addEventListener('click', () => {
        showNotification('warning', 'Please check your settings');
    });

    document.getElementById('error-notif').addEventListener('click', () => {
        showNotification('error', 'An error occurred');
    });

    document.addEventListener('pwa--vibration:triggered', () => {
        console.log('Haptic feedback provided');
    });
</script>

<style>
    .notification {
        padding: 15px;
        margin: 10px 0;
        border-radius: 8px;
        color: white;
        animation: slideIn 0.3s ease-out;
    }

    @keyframes slideIn {
        from {
            transform: translateX(-100%);
            opacity: 0;
        }
        to {
            transform: translateX(0);
            opacity: 1;
        }
    }

    .notification-types {
        display: flex;
        gap: 10px;
        margin: 20px 0;
    }

    .notification-types button {
        padding: 10px 20px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-weight: bold;
    }

    #info-notif { background: #3b82f6; color: white; }
    #success-notif { background: #10b981; color: white; }
    #warning-notif { background: #f59e0b; color: white; }
    #error-notif { background: #ef4444; color: white; }
</style>
```
{% endcode %}

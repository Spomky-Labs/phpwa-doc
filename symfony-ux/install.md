# Install

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Install controller handles the installation flow of a Progressive Web App (PWA). It listens for browser installation events, detects if the app is already installed, and provides an easy way to trigger the "Add to Home Screen" prompt when supported.

This component is essential for:

* Providing a custom install button or banner
* Detecting whether the app is already installed
* Tracking installation events and user behavior
* Creating a seamless installation experience
* Showing/hiding install prompts based on installation status

## Browser Support

The installation prompt API (`beforeinstallprompt` event) is supported in Chromium-based browsers (Chrome, Edge, Opera). Other browsers may use different installation mechanisms. The controller gracefully handles all scenarios.

{% hint style="info" %}
For the PWA to be installable, it must meet [PWA installation criteria](../how-to-install-remove-a-pwa.md): valid manifest, HTTPS, service worker, and appropriate icons.
{% endhint %}

## Usage

### Basic Install Button

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/install') }}>
    <button
        id="install-btn"
        {{ stimulus_action('@pwa/install', 'install', 'click') }}
        style="display:none;"
    >
        Install App
    </button>
</div>

<script>
    document.addEventListener('pwa--install:not-installed', () => {
        document.getElementById('install-btn').style.display = 'block';
    });

    document.addEventListener('pwa--install:installed', () => {
        document.getElementById('install-btn').style.display = 'none';
    });
</script>
```
{% endcode %}

### Install Banner with Animation

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/install') }}>
    <div id="install-banner" class="install-banner" style="display:none;">
        <div class="banner-content">
            <span class="app-icon">📱</span>
            <div class="banner-text">
                <strong>Install Our App</strong>
                <p>Get quick access and offline support!</p>
            </div>
            <button
                class="btn-install"
                {{ stimulus_action('@pwa/install', 'install', 'click') }}
            >
                Install
            </button>
            <button class="btn-close" onclick="document.getElementById('install-banner').style.display='none'">
                ✕
            </button>
        </div>
    </div>
</div>

<script>
    document.addEventListener('pwa--install:not-installed', () => {
        // Show banner after 3 seconds to avoid immediate interruption
        setTimeout(() => {
            const banner = document.getElementById('install-banner');
            banner.style.display = 'block';
            banner.classList.add('slide-in');
        }, 3000);
    });

    document.addEventListener('pwa--install:installed', () => {
        document.getElementById('install-banner').style.display = 'none';
        console.log('Thank you for installing our app!');
    });

    document.addEventListener('pwa--install:cancelled', () => {
        console.log('Installation was cancelled');
    });
</script>

<style>
    .install-banner {
        position: fixed;
        bottom: 0;
        left: 0;
        right: 0;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 15px;
        box-shadow: 0 -4px 6px rgba(0,0,0,0.1);
        z-index: 1000;
        transform: translateY(100%);
        transition: transform 0.3s ease-out;
    }

    .install-banner.slide-in {
        transform: translateY(0);
    }

    .banner-content {
        display: flex;
        align-items: center;
        gap: 15px;
        max-width: 1200px;
        margin: 0 auto;
    }

    .app-icon {
        font-size: 2em;
    }

    .banner-text {
        flex: 1;
    }

    .banner-text p {
        margin: 0;
        opacity: 0.9;
        font-size: 0.9em;
    }

    .btn-install, .btn-close {
        padding: 10px 20px;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-weight: bold;
    }

    .btn-install {
        background: white;
        color: #667eea;
    }

    .btn-close {
        background: transparent;
        color: white;
    }
</style>
```
{% endcode %}

### Install Card in App Settings

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/install') }}>
    <div class="settings-page">
        <h2>Settings</h2>

        <!-- Install card - only shown when not installed -->
        <div id="install-card" class="card" style="display:none;">
            <h3>📲 Install App</h3>
            <p>Install this app on your device for:</p>
            <ul>
                <li>Faster access from your home screen</li>
                <li>Offline functionality</li>
                <li>Better performance</li>
                <li>Native app experience</li>
            </ul>
            <button
                class="btn-primary"
                {{ stimulus_action('@pwa/install', 'install', 'click') }}
            >
                Install Now
            </button>
        </div>

        <!-- Already installed message -->
        <div id="installed-message" class="card success" style="display:none;">
            <h3>✅ App Installed</h3>
            <p>Thank you for installing our app!</p>
        </div>
    </div>
</div>

<script>
    document.addEventListener('pwa--install:not-installed', () => {
        document.getElementById('install-card').style.display = 'block';
        document.getElementById('installed-message').style.display = 'none';
    });

    document.addEventListener('pwa--install:installed', () => {
        document.getElementById('install-card').style.display = 'none';
        document.getElementById('installed-message').style.display = 'block';
    });

    document.addEventListener('pwa--install:installing', () => {
        console.log('Installation prompt shown to user');
    });
</script>
```
{% endcode %}

### Smart Install Prompt with User Tracking

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/install') }}>
    <button id="smart-install-btn" style="display:none;">
        Install App
    </button>
</div>

<script>
    // Track install prompt dismissals
    const PROMPT_DISMISSED_KEY = 'pwa_install_dismissed';
    const PROMPT_DISMISS_COUNT = 'pwa_install_dismiss_count';

    function canShowPrompt() {
        const dismissed = localStorage.getItem(PROMPT_DISMISSED_KEY);
        const dismissCount = parseInt(localStorage.getItem(PROMPT_DISMISS_COUNT) || '0');

        // Don't show if dismissed today or dismissed more than 3 times
        if (dismissed === new Date().toDateString() || dismissCount >= 3) {
            return false;
        }

        return true;
    }

    document.addEventListener('pwa--install:not-installed', () => {
        if (canShowPrompt()) {
            // Only show after user has interacted with the site
            let interactionCount = 0;
            ['click', 'scroll', 'keypress'].forEach(event => {
                document.addEventListener(event, () => {
                    interactionCount++;
                    if (interactionCount >= 3) {
                        document.getElementById('smart-install-btn').style.display = 'block';
                    }
                });
            });
        }
    });

    document.addEventListener('pwa--install:installed', () => {
        // Clear dismissal tracking on successful install
        localStorage.removeItem(PROMPT_DISMISSED_KEY);
        localStorage.removeItem(PROMPT_DISMISS_COUNT);
    });

    document.addEventListener('pwa--install:cancelled', () => {
        // Track dismissal
        localStorage.setItem(PROMPT_DISMISSED_KEY, new Date().toDateString());

        const count = parseInt(localStorage.getItem(PROMPT_DISMISS_COUNT) || '0');
        localStorage.setItem(PROMPT_DISMISS_COUNT, (count + 1).toString());

        console.log('User dismissed installation prompt');
    });
</script>
```
{% endcode %}

## Parameters

None

## Actions

### `install`

Triggers the PWA installation prompt. This action will:
1. Show the browser's native "Add to Home Screen" dialog
2. Emit the `installing` event when the prompt is shown
3. Emit either `installed` or `cancelled` based on user choice

```twig
{{ stimulus_action('@pwa/install', 'install', 'click') }}
```

{% hint style="warning" %}
This action only works if the browser supports installation prompts and the PWA meets all installation criteria.
{% endhint %}

## Targets

None

## Events

### `pwa--install:not-installed`

Dispatched when the PWA is not currently installed but can potentially be installed.

**No payload**

Use this event to show install buttons or banners:
```javascript
document.addEventListener('pwa--install:not-installed', () => {
    document.getElementById('install-button').style.display = 'block';
});
```

### `pwa--install:installed`

Dispatched when the PWA is detected as already installed or running in standalone mode.

**No payload**

This event is emitted:
* On page load if the app is running in standalone mode
* After a successful installation

Use this to hide install prompts and show thank you messages:
```javascript
document.addEventListener('pwa--install:installed', () => {
    document.getElementById('install-button').style.display = 'none';
    showThankYouMessage();
});
```

### `pwa--install:installing`

Dispatched when the user triggers the `install()` action and the installation prompt is shown.

**No payload**

Use this to track when users see the install prompt:
```javascript
document.addEventListener('pwa--install:installing', () => {
    console.log('Install prompt shown');
    // Track analytics
});
```

### `pwa--install:cancelled`

Dispatched when the user dismisses the install prompt or an error occurs during installation.

**No payload**

Use this to handle installation cancellation:
```javascript
document.addEventListener('pwa--install:cancelled', () => {
    console.log('Installation cancelled');
    // Maybe show a "Maybe later" message
    // Or track dismissal count
});
```

## Best Practices

1. **Don't be pushy**: Show install prompts after users have engaged with your app
2. **Provide value**: Explain benefits of installation (offline access, faster loading, etc.)
3. **Respect dismissals**: Track how many times users dismiss the prompt and back off
4. **Check installation status**: Always listen for the `installed` event to hide prompts
5. **Use appropriate timing**: Show install prompts at natural break points in the user journey
6. **Make it optional**: Never force installation; let users discover and install when ready
7. **Track metrics**: Monitor installation rates and prompt dismissals to optimize your approach

## Detection of Already Installed Apps

The controller automatically detects if your PWA is already installed by checking if the app is running in standalone mode. This detection happens immediately on page load, so you can hide install buttons for users who have already installed your app.

```javascript
// This event fires on page load if app is already installed
document.addEventListener('pwa--install:installed', () => {
    console.log('App is already installed!');
});
```

## Common Scenarios

### E-commerce Site

Show install prompt after user adds first item to cart:

```javascript
let itemsInCart = 0;

function addToCart(item) {
    itemsInCart++;
    // Show install prompt after first item
    if (itemsInCart === 1) {
        document.getElementById('install-banner').style.display = 'block';
    }
}
```

### Content Site

Show install prompt after user reads 3 articles:

```javascript
const articlesRead = parseInt(localStorage.getItem('articlesRead') || '0');

if (articlesRead >= 3) {
    document.addEventListener('pwa--install:not-installed', () => {
        showInstallPrompt();
    });
}
```

### Gaming App

Show install prompt after first game completed:

```javascript
function onGameComplete() {
    if (!localStorage.getItem('gameCompleted')) {
        localStorage.setItem('gameCompleted', 'true');
        showInstallPrompt();
    }
}
```

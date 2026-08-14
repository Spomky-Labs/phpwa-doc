# Picture In Picture

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Picture-in-Picture (PiP) component provides an interface to both classic video Picture-in-Picture and Document Picture-in-Picture APIs. It enables your Progressive Web App to display content in a floating window that stays on top of other windows, allowing users to multitask while keeping your content visible.

This component is particularly useful for:

* Video players and streaming applications
* Video conferencing and calls
* Live sports or event streaming
* Tutorial and educational content
* Dashboard monitoring and live data
* Multi-window workflows

## Browser Support

**Classic Picture-in-Picture (Video)**:
* Supported in Chrome, Edge, Safari, and Firefox
* Works with `<video>` elements

**Document Picture-in-Picture**:
* Supported in Chrome 116+ and Edge 116+
* Can display any DOM content (not just video)
* Currently experimental in other browsers

{% hint style="info" %}
The component automatically falls back to classic PiP for video elements when Document PiP is not available.
{% endhint %}

## Usage

### Basic Video Picture-in-Picture

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
    <div {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
        <video
            {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
            controls
            src="/videos/sample.mp4"
        >
        </video>
    </div>

    <button {{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}>
        Toggle Picture-in-Picture
    </button>
</div>

<script>
    document.addEventListener('pwa--picture-in-picture:enter', () => {
        console.log('Entered PiP mode');
    });

    document.addEventListener('pwa--picture-in-picture:exit', () => {
        console.log('Exited PiP mode');
    });
</script>
```
{% endcode %}

### Video Player with PiP Controls

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
    <div class="video-player" {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
        <video
            id="main-video"
            {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
            src="/videos/movie.mp4"
        ></video>

        <div class="player-controls">
            <button id="play-pause">▶️ Play</button>
            <button {{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}>
                📺 PiP Mode
            </button>
        </div>
    </div>
</div>

<script>
    const video = document.getElementById('main-video');
    const playPauseBtn = document.getElementById('play-pause');

    playPauseBtn.addEventListener('click', () => {
        if (video.paused) {
            video.play();
            playPauseBtn.textContent = '⏸️ Pause';
        } else {
            video.pause();
            playPauseBtn.textContent = '▶️ Play';
        }
    });

    document.addEventListener('pwa--picture-in-picture:enter', () => {
        // Auto-play when entering PiP
        video.play();
        // Update UI
        document.querySelector('.video-player').classList.add('in-pip');
    });

    document.addEventListener('pwa--picture-in-picture:exit', () => {
        // Update UI when exiting PiP
        document.querySelector('.video-player').classList.remove('in-pip');
    });

    document.addEventListener('pwa--picture-in-picture:error', (event) => {
        console.error('PiP error:', event.detail);
        alert('Picture-in-Picture is not available');
    });
</script>
```
{% endcode %}

### Document PiP (Any Content)

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
    <div {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
        <div
            {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
            class="dashboard-widget"
        >
            <h3>Live Dashboard</h3>
            <div class="stats">
                <div class="stat">
                    <span class="label">Users Online:</span>
                    <span class="value" id="users-count">1,234</span>
                </div>
                <div class="stat">
                    <span class="label">Revenue:</span>
                    <span class="value" id="revenue">$12,345</span>
                </div>
                <div class="stat">
                    <span class="label">Orders:</span>
                    <span class="value" id="orders">89</span>
                </div>
            </div>
        </div>
    </div>

    <button {{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}>
        Float Dashboard
    </button>
</div>

<script>
    // Simulate real-time updates
    setInterval(() => {
        document.getElementById('users-count').textContent =
            Math.floor(Math.random() * 2000);
        document.getElementById('revenue').textContent =
            '$' + Math.floor(Math.random() * 50000);
        document.getElementById('orders').textContent =
            Math.floor(Math.random() * 200);
    }, 2000);

    document.addEventListener('pwa--picture-in-picture:unsupported', () => {
        alert('Document Picture-in-Picture is not supported in this browser');
    });
</script>
```
{% endcode %}

### Video Conference with PiP

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
    <div class="video-conference" {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
        <div
            class="local-video-container"
            {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
        >
            <video id="local-video" autoplay muted></video>
            <div class="video-controls">
                <button id="mute-toggle">🎤 Mute</button>
                <button id="camera-toggle">📹 Camera</button>
            </div>
        </div>

        <div class="remote-videos">
            <video class="remote-video" autoplay></video>
            <video class="remote-video" autoplay></video>
        </div>
    </div>

    <button {{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}>
        Float My Video
    </button>
</div>

<script>
    // Initialize local video
    navigator.mediaDevices.getUserMedia({ video: true, audio: true })
        .then(stream => {
            document.getElementById('local-video').srcObject = stream;
        });

    document.addEventListener('pwa--picture-in-picture:enter', () => {
        console.log('Your video is now floating');
        // Continue seeing your video while browsing other tabs
    });

    document.addEventListener('pwa--picture-in-picture:exit', () => {
        console.log('Your video is back in the main window');
    });
</script>
```
{% endcode %}

### Live Stream Monitor

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
    <div class="stream-monitor" {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
        <div
            class="live-stream"
            {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
        >
            <video id="live-stream" autoplay>
                <source src="https://example.com/live.m3u8" type="application/x-mpegURL">
            </video>

            <div class="stream-info">
                <span class="live-indicator">🔴 LIVE</span>
                <span id="viewer-count">1.2K viewers</span>
            </div>
        </div>
    </div>

    <div class="chat-section">
        <h3>Live Chat</h3>
        <div id="chat-messages"></div>
        <input type="text" placeholder="Type a message...">
    </div>

    <button {{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}>
        Pop Out Stream
    </button>
</div>

<script>
    document.addEventListener('pwa--picture-in-picture:enter', () => {
        // Keep watching the stream while interacting with chat
        console.log('Stream popped out - continue chatting!');
    });
</script>
```
{% endcode %}

### Music Player with Lyrics

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
    <div class="music-player" {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
        <div
            class="player-widget"
            {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
        >
            <div class="album-art">
                <img src="/albums/cover.jpg" alt="Album Cover">
            </div>

            <div class="track-info">
                <h4 id="track-title">Song Title</h4>
                <p id="artist-name">Artist Name</p>
            </div>

            <div class="player-controls">
                <button>⏮️</button>
                <button id="play-pause">▶️</button>
                <button>⏭️</button>
            </div>

            <div class="progress-bar">
                <div class="progress" style="width: 45%"></div>
            </div>
        </div>
    </div>

    <div class="lyrics-section">
        <h3>Lyrics</h3>
        <div id="lyrics"></div>
    </div>

    <button {{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}>
        Float Player
    </button>
</div>

<script>
    document.addEventListener('pwa--picture-in-picture:enter', () => {
        // Keep controls accessible while reading lyrics
        console.log('Player floating - read lyrics below');
    });
</script>
```
{% endcode %}

## Parameters

None

## Actions

### `toggle`

Toggles Picture-in-Picture mode. If not currently in PiP, it enters PiP mode. If already in PiP, it exits.

```twig
{{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}
```

The action will:
1. Check if the `floating` target element is specified
2. For `<video>` elements: Use classic Video PiP API
3. For other elements: Use Document PiP API (where supported)
4. Automatically restore the element to its original position when exiting

## Targets

### `floating`

**Required**. The element to display in the Picture-in-Picture window.

* For video elements: Uses classic Video PiP
* For other elements: Uses Document PiP (Chromium browsers only)

```twig
{{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
```

### `container`

**Required**. The parent container where the floating element should be restored when exiting PiP.

```twig
{{ stimulus_target('@pwa/picture-in-picture', 'container') }}
```

The component uses this target to remember where to put the element back after PiP closes.

## Events

### `pwa--picture-in-picture:enter`

Dispatched when the element successfully enters Picture-in-Picture mode.

**No payload**

Example:
```javascript
document.addEventListener('pwa--picture-in-picture:enter', () => {
    console.log('Entered PiP mode');

    // Update UI
    document.querySelector('.pip-button').textContent = 'Exit PiP';

    // Start tracking
    analytics.track('pip_entered');
});
```

### `pwa--picture-in-picture:exit`

Dispatched when the Picture-in-Picture session ends.

**No payload**

This event fires when:
* User closes the PiP window
* `toggle()` action is called while in PiP mode
* Browser automatically closes PiP

Example:
```javascript
document.addEventListener('pwa--picture-in-picture:exit', () => {
    console.log('Exited PiP mode');

    // Update UI
    document.querySelector('.pip-button').textContent = 'Enter PiP';

    // The element is automatically restored to its container
});
```

### `pwa--picture-in-picture:unsupported`

Dispatched when Picture-in-Picture is not available.

**No payload**

Reasons for this event:
* Browser doesn't support PiP
* Feature is disabled by user/browser policy
* Platform restrictions (e.g., trying Document PiP on unsupported browser)

Example:
```javascript
document.addEventListener('pwa--picture-in-picture:unsupported', () => {
    console.warn('PiP not supported');

    // Hide PiP button
    document.querySelector('.pip-button').style.display = 'none';

    // Show message
    alert('Picture-in-Picture is not supported on this device');
});
```

### `pwa--picture-in-picture:error`

Dispatched when entering or exiting PiP fails.

**Payload**: Error details

Example:
```javascript
document.addEventListener('pwa--picture-in-picture:error', (event) => {
    console.error('PiP error:', event.detail);

    // Show user-friendly message
    const errorMsg = event.detail.message || 'Failed to activate Picture-in-Picture';
    alert(errorMsg);
});
```

## Best Practices

1. **Provide clear controls**: Make it obvious how to enter and exit PiP mode
2. **Preserve state**: Ensure playback state is maintained when entering/exiting PiP
3. **Responsive design**: PiP windows are typically small, design accordingly
4. **Handle errors gracefully**: Always listen for unsupported and error events
5. **User interaction required**: PiP can only be triggered by user actions
6. **Test on devices**: Behavior varies across browsers and platforms
7. **Provide fallbacks**: Not all browsers support Document PiP

## Classic vs Document PiP

| Feature | Classic PiP | Document PiP |
|---------|-------------|--------------|
| Content | `<video>` only | Any DOM elements |
| Browser Support | Wide (all modern browsers) | Chrome/Edge 116+ |
| Use Cases | Video streaming | Dashboards, widgets, any content |
| Styling | Limited | Full CSS control |
| Interactivity | Video controls only | Full DOM interaction |

## Browser-Specific Behavior

### Chrome/Edge

* Full support for both classic and Document PiP
* Document PiP available from version 116+
* Provides native PiP controls

### Safari

* Classic PiP supported for video
* No Document PiP support
* iOS Safari has platform-specific PiP UI

### Firefox

* Classic PiP supported
* No Document PiP support yet
* Provides built-in PiP button for videos

## Styling PiP Content

When using Document PiP, you can style the floating content:

```css
/* Target PiP window content */
@media (display-mode: picture-in-picture) {
    .dashboard-widget {
        padding: 10px;
        font-size: 14px;
    }

    /* Hide unnecessary elements in PiP */
    .sidebar,
    .footer {
        display: none;
    }
}
```

## Complete Example: Multi-Feature Video Player

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/picture-in-picture') }}>
    <div class="advanced-player" {{ stimulus_target('@pwa/picture-in-picture', 'container') }}>
        <video
            id="player-video"
            {{ stimulus_target('@pwa/picture-in-picture', 'floating') }}
            src="/videos/demo.mp4"
        ></video>

        <div class="player-ui">
            <div class="progress-container">
                <input type="range" id="progress" min="0" max="100" value="0">
                <span id="time">0:00 / 0:00</span>
            </div>

            <div class="controls">
                <button id="play-pause">▶️</button>
                <button id="mute">🔊</button>
                <input type="range" id="volume" min="0" max="100" value="100">
                <button id="fullscreen">⛶</button>
                <button
                    {{ stimulus_action('@pwa/picture-in-picture', 'toggle', 'click') }}
                    id="pip-btn"
                >
                    📺 PiP
                </button>
            </div>
        </div>
    </div>

    <div class="video-description">
        <h2>Video Title</h2>
        <p>Video description and details...</p>
    </div>
</div>

<script>
    const video = document.getElementById('player-video');
    const playPauseBtn = document.getElementById('play-pause');
    const muteBtn = document.getElementById('mute');
    const volumeSlider = document.getElementById('volume');
    const progressSlider = document.getElementById('progress');
    const timeDisplay = document.getElementById('time');
    const pipBtn = document.getElementById('pip-btn');

    // Play/Pause
    playPauseBtn.addEventListener('click', () => {
        if (video.paused) {
            video.play();
            playPauseBtn.textContent = '⏸️';
        } else {
            video.pause();
            playPauseBtn.textContent = '▶️';
        }
    });

    // Mute/Unmute
    muteBtn.addEventListener('click', () => {
        video.muted = !video.muted;
        muteBtn.textContent = video.muted ? '🔇' : '🔊';
    });

    // Volume
    volumeSlider.addEventListener('input', (e) => {
        video.volume = e.target.value / 100;
    });

    // Progress
    video.addEventListener('timeupdate', () => {
        const percent = (video.currentTime / video.duration) * 100;
        progressSlider.value = percent;

        const current = formatTime(video.currentTime);
        const total = formatTime(video.duration);
        timeDisplay.textContent = `${current} / ${total}`;
    });

    progressSlider.addEventListener('input', (e) => {
        const time = (e.target.value / 100) * video.duration;
        video.currentTime = time;
    });

    function formatTime(seconds) {
        const mins = Math.floor(seconds / 60);
        const secs = Math.floor(seconds % 60);
        return `${mins}:${secs.toString().padStart(2, '0')}`;
    }

    // PiP Events
    document.addEventListener('pwa--picture-in-picture:enter', () => {
        pipBtn.textContent = '⬛ Exit PiP';
        pipBtn.classList.add('active');

        // Auto-play when entering PiP
        video.play();

        console.log('Video is now floating - continue browsing!');
    });

    document.addEventListener('pwa--picture-in-picture:exit', () => {
        pipBtn.textContent = '📺 PiP';
        pipBtn.classList.remove('active');

        console.log('Video returned to main window');
    });

    document.addEventListener('pwa--picture-in-picture:unsupported', () => {
        pipBtn.style.display = 'none';
        console.warn('Picture-in-Picture not supported');
    });

    document.addEventListener('pwa--picture-in-picture:error', (event) => {
        console.error('PiP error:', event.detail);
        alert('Failed to activate Picture-in-Picture mode');
    });

    // Fullscreen
    document.getElementById('fullscreen').addEventListener('click', () => {
        if (video.requestFullscreen) {
            video.requestFullscreen();
        }
    });
</script>

<style>
    .advanced-player {
        position: relative;
        max-width: 800px;
        background: #000;
        border-radius: 8px;
        overflow: hidden;
    }

    video {
        width: 100%;
        display: block;
    }

    .player-ui {
        background: linear-gradient(transparent, rgba(0,0,0,0.8));
        padding: 10px;
    }

    .controls {
        display: flex;
        gap: 10px;
        align-items: center;
    }

    .controls button {
        background: transparent;
        border: none;
        color: white;
        font-size: 20px;
        cursor: pointer;
        padding: 5px 10px;
    }

    .controls button.active {
        background: rgba(255,255,255,0.2);
        border-radius: 4px;
    }

    .progress-container {
        margin-bottom: 10px;
    }

    #progress {
        width: 100%;
    }

    #time {
        color: white;
        font-size: 12px;
    }

    .video-description {
        padding: 20px;
    }

    /* Style for when video is in PiP */
    .advanced-player.in-pip {
        opacity: 0.5;
    }
</style>
```
{% endcode %}

# Fullscreen

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Fullscreen component provides an interface to the Fullscreen API, enabling your Progressive Web App to display content in fullscreen mode. This creates an immersive experience by removing browser chrome and utilizing the entire screen space.

This component is particularly useful for:

* Media players and video streaming applications
* Photo galleries and image viewers
* Presentation and slideshow applications
* Gaming applications
* Data visualization and dashboard applications
* Reading apps and e-books

## Browser Support

The Fullscreen API is widely supported in modern browsers. However, browser vendors may have different implementations and require user interaction to activate fullscreen mode.

{% hint style="info" %}
Fullscreen mode can only be triggered by user interaction (click, touch) for security reasons. Programmatic fullscreen requests without user gestures will be blocked.
{% endhint %}

## Usage

### Fullscreen for Entire Page

{% code lineNumbers="true" %}
```twig
<section {{ stimulus_controller('@pwa/fullscreen') }}>
    <h1>My Presentation</h1>
    <div class="content">
        <!-- Your content here -->
    </div>

    <button {{ stimulus_action('@pwa/fullscreen', 'request', 'click') }}>
        Enter Fullscreen
    </button>

    <button {{ stimulus_action('@pwa/fullscreen', 'exit', 'click') }}>
        Exit Fullscreen
    </button>
</section>

<script>
    document.addEventListener('pwa--fullscreen:change', (event) => {
        const { fullscreen, element } = event.detail;
        console.log('Fullscreen:', fullscreen ? 'active' : 'inactive');
    });
</script>
```
{% endcode %}

### Fullscreen for Specific Element

{% code lineNumbers="true" %}
```twig
<section {{ stimulus_controller('@pwa/fullscreen') }}>
    <img
        id="gallery-image"
        src="https://picsum.photos/800/600"
        alt="Sample Image"
        class="max-w-full"
    >

    <div class="controls">
        <button {{ stimulus_action('@pwa/fullscreen', 'request', 'click', {target: '#gallery-image'}) }}>
            View Image Fullscreen
        </button>

        <button {{ stimulus_action('@pwa/fullscreen', 'exit', 'click') }}>
            Exit Fullscreen
        </button>
    </div>
</section>
```
{% endcode %}

### Video Player with Fullscreen

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/fullscreen') }}>
    <div id="video-container" class="video-wrapper">
        <video id="main-video" controls>
            <source src="/videos/sample.mp4" type="video/mp4">
        </video>

        <div class="video-controls">
            <button {{ stimulus_action('@pwa/fullscreen', 'request', 'click', {target: '#video-container'}) }}>
                ⛶ Fullscreen
            </button>
        </div>
    </div>
</div>

<script>
    document.addEventListener('pwa--fullscreen:change', (event) => {
        const { fullscreen } = event.detail;
        const video = document.getElementById('main-video');

        if (fullscreen) {
            // Auto-play when entering fullscreen
            video.play();
        }
    });

    document.addEventListener('pwa--fullscreen:error', (event) => {
        console.error('Fullscreen error:', event.detail);
        alert('Unable to enter fullscreen mode');
    });
</script>
```
{% endcode %}

### Image Gallery with Fullscreen

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/fullscreen') }}>
    <div class="gallery">
        <div class="image-grid">
            <img id="img-1" src="https://picsum.photos/400/300?random=1" alt="Photo 1" class="gallery-item">
            <img id="img-2" src="https://picsum.photos/400/300?random=2" alt="Photo 2" class="gallery-item">
            <img id="img-3" src="https://picsum.photos/400/300?random=3" alt="Photo 3" class="gallery-item">
        </div>
    </div>
</div>

<script>
    document.querySelectorAll('.gallery-item').forEach(img => {
        img.addEventListener('click', function() {
            const controller = document.querySelector('[data-controller="@pwa/fullscreen"]');
            controller.dispatchEvent(new CustomEvent('request', {
                detail: {
                    params: {
                        target: `#${this.id}`
                    }
                }
            }));
        });
    });

    // Exit fullscreen on Escape key
    document.addEventListener('keydown', (e) => {
        if (e.key === 'Escape') {
            const controller = document.querySelector('[data-controller="@pwa/fullscreen"]');
            controller.dispatchEvent(new CustomEvent('exit'));
        }
    });
</script>
```
{% endcode %}

### Presentation Mode with Navigation

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/fullscreen') }}>
    <div id="presentation" class="slides-container">
        <div class="slide active">
            <h2>Slide 1</h2>
            <p>Introduction</p>
        </div>
        <div class="slide">
            <h2>Slide 2</h2>
            <p>Main Content</p>
        </div>
        <div class="slide">
            <h2>Slide 3</h2>
            <p>Conclusion</p>
        </div>
    </div>

    <div class="presentation-controls">
        <button {{ stimulus_action('@pwa/fullscreen', 'request', 'click', {target: '#presentation'}) }}>
            Start Presentation
        </button>
    </div>
</div>

<script>
    let currentSlide = 0;
    const slides = document.querySelectorAll('.slide');

    function showSlide(index) {
        slides.forEach(slide => slide.classList.remove('active'));
        slides[index].classList.add('active');
    }

    document.addEventListener('pwa--fullscreen:change', (event) => {
        const { fullscreen } = event.detail;

        if (fullscreen) {
            // Enable keyboard navigation in fullscreen
            document.addEventListener('keydown', navigateSlides);
        } else {
            // Disable keyboard navigation when exiting
            document.removeEventListener('keydown', navigateSlides);
        }
    });

    function navigateSlides(e) {
        if (e.key === 'ArrowRight' && currentSlide < slides.length - 1) {
            currentSlide++;
            showSlide(currentSlide);
        } else if (e.key === 'ArrowLeft' && currentSlide > 0) {
            currentSlide--;
            showSlide(currentSlide);
        }
    }
</script>

<style>
    .slide {
        display: none;
        padding: 2rem;
        height: 100vh;
        align-items: center;
        justify-content: center;
    }

    .slide.active {
        display: flex;
        flex-direction: column;
    }

    .presentation-controls {
        position: fixed;
        bottom: 20px;
        left: 50%;
        transform: translateX(-50%);
    }
</style>
```
{% endcode %}

## Parameters

None

## Actions

### `request`

Triggers fullscreen mode for the entire document or a specific element.

**Parameters:**
* `target` (string, optional): CSS selector for the element to display in fullscreen. If omitted, the entire document enters fullscreen.

```twig
{{ stimulus_action('@pwa/fullscreen', 'request', 'click') }}
```

Or for a specific element:
```twig
{{ stimulus_action('@pwa/fullscreen', 'request', 'click', {target: '#my-element'}) }}
```

{% hint style="warning" %}
Fullscreen requests must be triggered by user interaction. Programmatic calls without user gestures will be rejected by the browser.
{% endhint %}

### `exit`

Exits fullscreen mode and returns to normal display.

```twig
{{ stimulus_action('@pwa/fullscreen', 'exit', 'click') }}
```

Alternatively, users can press the `Escape` key to exit fullscreen mode (this is a browser feature, not controlled by the component).

## Targets

None

## Events

### `pwa--fullscreen:change`

Dispatched when fullscreen mode is toggled (entered or exited).

**Payload**: `{fullscreen, element}`
* `fullscreen` (boolean): `true` when entering fullscreen, `false` when exiting
* `element` (Element): The element that entered fullscreen

Example:
```javascript
document.addEventListener('pwa--fullscreen:change', (event) => {
    const { fullscreen, element } = event.detail;

    if (fullscreen) {
        console.log('Entered fullscreen for:', element);
        // Update UI for fullscreen mode
        document.body.classList.add('in-fullscreen');
    } else {
        console.log('Exited fullscreen');
        // Restore normal UI
        document.body.classList.remove('in-fullscreen');
    }
});
```

### `pwa--fullscreen:error`

Dispatched when an error occurs while entering or exiting fullscreen mode.

**Payload**: `{element, error}`
* `element` (Element): The element that failed to enter fullscreen
* `error` (Error): Error object containing details about the failure

Example:
```javascript
document.addEventListener('pwa--fullscreen:error', (event) => {
    const { element, error } = event.detail;
    console.error('Fullscreen error for', element, ':', error);

    // Show user-friendly error message
    alert('Unable to enter fullscreen mode. Please try again.');
});
```

## Best Practices

1. **User-initiated only**: Always trigger fullscreen from user interactions (clicks, touches)
2. **Provide exit option**: Always offer a visible way to exit fullscreen
3. **Handle Escape key**: Document that users can press Escape to exit
4. **Responsive design**: Ensure your fullscreen content adapts to different screen sizes
5. **Test on devices**: Fullscreen behavior may vary across browsers and devices
6. **Communicate state**: Clearly indicate when fullscreen mode is active
7. **Preserve content**: Ensure fullscreen elements are properly restored when exiting

## Common Use Cases

### Media Player Controls

```javascript
document.addEventListener('pwa--fullscreen:change', (event) => {
    const { fullscreen } = event.detail;
    const controls = document.querySelector('.media-controls');

    if (fullscreen) {
        // Show minimal controls in fullscreen
        controls.classList.add('minimal');
        // Auto-hide controls after 3 seconds
        setTimeout(() => {
            controls.classList.add('hidden');
        }, 3000);
    } else {
        // Show full controls in normal mode
        controls.classList.remove('minimal', 'hidden');
    }
});
```

### Dashboard Fullscreen

```javascript
function toggleDashboardFullscreen() {
    const controller = document.querySelector('[data-controller="@pwa/fullscreen"]');
    const dashboard = document.getElementById('dashboard');

    // Store current scroll position
    const scrollPos = window.scrollY;

    controller.dispatchEvent(new CustomEvent('request', {
        detail: { params: { target: '#dashboard' } }
    }));

    // Restore scroll position when exiting
    document.addEventListener('pwa--fullscreen:change', function handler(e) {
        if (!e.detail.fullscreen) {
            window.scrollTo(0, scrollPos);
            document.removeEventListener('pwa--fullscreen:change', handler);
        }
    });
}
```

### Photo Viewer with Gestures

```javascript
let touchStartX = 0;
let touchEndX = 0;

document.addEventListener('pwa--fullscreen:change', (event) => {
    const { fullscreen, element } = event.detail;

    if (fullscreen && element.tagName === 'IMG') {
        // Enable swipe gestures for navigation
        element.addEventListener('touchstart', e => {
            touchStartX = e.changedTouches[0].screenX;
        });

        element.addEventListener('touchend', e => {
            touchEndX = e.changedTouches[0].screenX;
            handleSwipe();
        });
    }
});

function handleSwipe() {
    if (touchEndX < touchStartX - 50) {
        // Swipe left - next image
        showNextImage();
    }
    if (touchEndX > touchStartX + 50) {
        // Swipe right - previous image
        showPreviousImage();
    }
}
```

## Browser-Specific Considerations

### Safari (iOS)

* On iOS, only video elements can enter fullscreen (not other elements)
* Fullscreen video is handled by the native player
* Document-level fullscreen is not supported

### Chrome/Edge

* Full support for element and document fullscreen
* Provides native fullscreen UI controls
* Supports custom fullscreen styling with CSS

### Firefox

* Good support with vendor prefixes in older versions
* Modern versions support standard Fullscreen API

## CSS for Fullscreen Mode

You can style elements differently when in fullscreen using CSS:

```css
/* Normal state */
.video-player {
    max-width: 800px;
}

/* Fullscreen state */
.video-player:fullscreen {
    width: 100vw;
    height: 100vh;
    background: black;
}

/* Vendor prefixes for older browsers */
.video-player:-webkit-full-screen {
    width: 100vw;
    height: 100vh;
}

.video-player:-moz-full-screen {
    width: 100vw;
    height: 100vh;
}
```

## Troubleshooting

### Fullscreen not working

**Common issues:**
1. **No user interaction**: Fullscreen must be triggered by user gesture
2. **Browser permissions**: Some browsers block fullscreen by default
3. **Iframe restrictions**: Fullscreen may not work in iframes without proper attributes
4. **Mobile Safari**: Only video elements support fullscreen

### Element not displaying correctly

**Solutions:**
1. **Check CSS**: Ensure fullscreen element has proper styling
2. **Z-index issues**: Fullscreen elements should have high z-index
3. **Viewport units**: Use `100vw` and `100vh` for full coverage
4. **Aspect ratio**: Maintain proper aspect ratios in fullscreen

## Complete Example: Video Player

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/fullscreen') }}>
    <div id="player-container" class="video-player">
        <video id="video" src="/videos/sample.mp4"></video>

        <div class="player-controls">
            <button id="play-pause">▶️ Play</button>
            <input type="range" id="progress" min="0" max="100" value="0">
            <span id="time">0:00 / 0:00</span>
            <button {{ stimulus_action('@pwa/fullscreen', 'request', 'click', {target: '#player-container'}) }}>
                ⛶ Fullscreen
            </button>
        </div>
    </div>
</div>

<script>
    const video = document.getElementById('video');
    const playPauseBtn = document.getElementById('play-pause');
    const progress = document.getElementById('progress');
    const timeDisplay = document.getElementById('time');

    playPauseBtn.addEventListener('click', () => {
        if (video.paused) {
            video.play();
            playPauseBtn.textContent = '⏸️ Pause';
        } else {
            video.pause();
            playPauseBtn.textContent = '▶️ Play';
        }
    });

    video.addEventListener('timeupdate', () => {
        const percent = (video.currentTime / video.duration) * 100;
        progress.value = percent;

        const current = formatTime(video.currentTime);
        const total = formatTime(video.duration);
        timeDisplay.textContent = `${current} / ${total}`;
    });

    progress.addEventListener('input', (e) => {
        const time = (e.target.value / 100) * video.duration;
        video.currentTime = time;
    });

    function formatTime(seconds) {
        const mins = Math.floor(seconds / 60);
        const secs = Math.floor(seconds % 60);
        return `${mins}:${secs.toString().padStart(2, '0')}`;
    }

    document.addEventListener('pwa--fullscreen:change', (event) => {
        const { fullscreen } = event.detail;
        const controls = document.querySelector('.player-controls');

        if (fullscreen) {
            controls.classList.add('fullscreen-controls');
            video.play();
        } else {
            controls.classList.remove('fullscreen-controls');
        }
    });
</script>

<style>
    .video-player {
        position: relative;
        max-width: 800px;
        margin: 0 auto;
    }

    .video-player:fullscreen {
        width: 100vw;
        height: 100vh;
        background: black;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }

    .video-player:fullscreen video {
        width: 100%;
        height: calc(100% - 60px);
    }

    .player-controls {
        display: flex;
        gap: 10px;
        padding: 10px;
        background: rgba(0, 0, 0, 0.8);
        color: white;
    }

    .fullscreen-controls {
        position: absolute;
        bottom: 0;
        width: 100%;
    }
</style>
```
{% endcode %}

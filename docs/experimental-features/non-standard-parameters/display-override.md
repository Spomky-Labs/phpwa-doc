# Display Override

{% hint style="warning" %}
**Experimental Feature**: `display_override` is part of the PWA manifest specification but has limited browser support. Check compatibility before using in production.
{% endhint %}

## Overview

The `display_override` property allows you to specify a preferred display mode with multiple fallbacks for cases where certain display modes are not supported by the platform or browser. This provides graceful degradation and ensures your PWA displays optimally across different environments.

## How It Works

The browser evaluates display modes in the `display_override` array **from left to right**. The first supported mode is used. If no mode in the array is supported, the browser falls back to the `display` property.

**Evaluation Order**:
1. Try first mode in `display_override`
2. If not supported, try next mode
3. Continue until a supported mode is found
4. If none supported, use `display` property
5. If `display` not supported, use `browser`

## Configuration

### Basic Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        display: "standalone"  # Fallback if display_override not supported
        display_override: ["fullscreen", "minimal-ui"]
```
{% endcode %}

**Fallback chain**: `fullscreen` → `minimal-ui` → `standalone` → `browser`

### Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "My Immersive App"
        display: "standalone"
        display_override:
            - "fullscreen"
            - "window-controls-overlay"
            - "minimal-ui"
```
{% endcode %}

**Evaluation**:
1. Try `fullscreen` (for media/games)
2. If not available, try `window-controls-overlay` (for desktop apps)
3. If not available, try `minimal-ui`
4. If not available, use `standalone` (from display property)

## Display Modes

### fullscreen

**Description**: Uses all available display area with no browser UI visible. Ideal for immersive experiences.

**Use Cases**:
- Games
- Video players
- Image galleries
- Presentation apps
- Immersive media experiences

**Platform Support**:
- ✅ Android Chrome (full support)
- ⚠️ Desktop (varies by browser)
- ❌ iOS Safari (not supported)

**Example**:
```yaml
display_override: ["fullscreen"]
display: "standalone"  # Fallback for iOS
```

**User Experience**:
- No address bar
- No browser controls
- No status bar
- Fullscreen content
- Exit typically via system gesture

### standalone

**Description**: The application looks and feels like a standalone native application with its own window and app icon.

**Use Cases**:
- Most PWAs
- Productivity apps
- Social media apps
- E-commerce apps
- Default recommended mode

**Platform Support**:
- ✅ Android Chrome
- ✅ iOS Safari
- ✅ Desktop browsers
- ✅ Windows (installed PWAs)

**Example**:
```yaml
display: "standalone"  # Most common, widely supported
```

**User Experience**:
- Separate window
- Own app icon
- System status bar (mobile)
- No address bar
- No browser back/forward buttons

### minimal-ui

**Description**: Looks like a standalone app but includes a minimal set of UI elements for navigation (browser-dependent).

**Use Cases**:
- Content-focused apps
- Reading apps
- News applications
- Blog platforms
- When you want minimal browser UI

**Platform Support**:
- ✅ Some Android browsers
- ⚠️ Implementation varies widely
- ❌ Many browsers treat as `standalone`

**Example**:
```yaml
display_override: ["minimal-ui"]
display: "standalone"  # Common fallback
```

**User Experience**:
- Similar to standalone
- May include back button
- May include reload button
- Varies by browser/platform

### browser

**Description**: Opens in a conventional browser tab or window with full browser UI.

**Use Cases**:
- Progressive enhancement
- Apps that benefit from browser context
- Default fallback
- Development/testing

**Platform Support**:
- ✅ All browsers (default behavior)

**Example**:
```yaml
display: "browser"  # Explicitly request browser mode
```

**User Experience**:
- Standard browser tab
- Address bar visible
- Browser navigation buttons
- Browser menu
- Standard web experience

### window-controls-overlay

**Description**: Desktop-only mode that provides maximum window surface area with window controls (minimize, maximize, close) as an overlay.

**Use Cases**:
- Desktop-focused apps
- Productivity tools
- Code editors
- Design tools
- Apps needing maximum screen space

**Platform Support**:
- ✅ Desktop Chrome/Edge (Windows, macOS, Linux)
- ❌ Mobile (ignored)
- ❌ iOS (not supported)

**Example**:
```yaml
display_override: ["window-controls-overlay", "standalone"]
```

**User Experience**:
- Full window surface for content
- Title bar controls as overlay
- Custom title bar possible
- Maximize content area
- Desktop-optimized

**JavaScript Integration**:
```javascript
if ('windowControlsOverlay' in navigator) {
    const controls = navigator.windowControlsOverlay;
    console.log('Overlay visible:', controls.visible);
    console.log('Title bar area:', controls.getTitlebarAreaRect());
}
```

## Browser Support

| Display Mode | Chrome | Edge | Safari | Firefox | Samsung Internet |
|--------------|--------|------|--------|---------|------------------|
| `fullscreen` | ✅ | ✅ | ❌ | ⚠️ | ✅ |
| `standalone` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `minimal-ui` | ⚠️ | ⚠️ | ❌ | ❌ | ⚠️ |
| `browser` | ✅ | ✅ | ✅ | ✅ | ✅ |
| `window-controls-overlay` | ✅* | ✅* | ❌ | ❌ | ❌ |

*Desktop only

{% hint style="info" %}
**Support Detection**: Always include a well-supported fallback in your `display` property.
{% endhint %}

## Use Cases by Application Type

### 1. Media/Gaming App

Prioritize immersive fullscreen experience:

```yaml
pwa:
    manifest:
        name: "Game Studio"
        display: "standalone"
        display_override:
            - "fullscreen"
```

**Reasoning**: Games benefit from fullscreen. Falls back to standalone on iOS.

### 2. Productivity App (Desktop-focused)

Maximize workspace on desktop:

```yaml
pwa:
    manifest:
        name: "Code Editor"
        display: "standalone"
        display_override:
            - "window-controls-overlay"
```

**Reasoning**: Desktop users get maximum screen space. Mobile users get standalone.

### 3. Content Reader

Minimal UI for focused reading:

```yaml
pwa:
    manifest:
        name: "News Reader"
        display: "standalone"
        display_override:
            - "minimal-ui"
```

**Reasoning**: Tries for minimal UI, falls back to well-supported standalone.

### 4. E-Commerce App

Standard app-like experience:

```yaml
pwa:
    manifest:
        name: "Shop App"
        display: "standalone"
```

**Reasoning**: Standalone is widely supported and provides app-like experience without experimental features.

### 5. Progressive Enhancement

Start with browser, offer better experiences:

```yaml
pwa:
    manifest:
        name: "Progressive App"
        display: "browser"
        display_override:
            - "standalone"
```

**Reasoning**: Works everywhere, enhances to standalone where supported.

### 6. Multi-Platform App

Different experiences for desktop and mobile:

```yaml
pwa:
    manifest:
        name: "Universal App"
        display: "standalone"
        display_override:
            - "window-controls-overlay"  # Desktop
            - "fullscreen"               # Mobile (games/media)
            - "minimal-ui"               # Fallback
```

**Reasoning**: Desktop gets window controls overlay, mobile might get fullscreen, everyone gets at least standalone.

## Testing Display Modes

### 1. Chrome DevTools

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "Display" and "Display Override" sections
4. See which mode is active
```

### 2. Install and Check

```bash
# Desktop
1. Install PWA from browser
2. Open as standalone app
3. Check window chrome/controls
4. Verify expected display mode

# Mobile
1. Add to home screen
2. Open from home screen
3. Check for address bar/browser UI
4. Verify fullscreen if configured
```

### 3. JavaScript Detection

```javascript
// Check current display mode
function getCurrentDisplayMode() {
    if (window.matchMedia('(display-mode: fullscreen)').matches) {
        return 'fullscreen';
    }
    if (window.matchMedia('(display-mode: standalone)').matches) {
        return 'standalone';
    }
    if (window.matchMedia('(display-mode: minimal-ui)').matches) {
        return 'minimal-ui';
    }
    return 'browser';
}

console.log('Display mode:', getCurrentDisplayMode());

// Listen for display mode changes
window.matchMedia('(display-mode: standalone)').addEventListener('change', (e) => {
    if (e.matches) {
        console.log('App is now in standalone mode');
    }
});
```

### 4. Test Across Platforms

```bash
# Test matrix
- ✓ Android Chrome (mobile)
- ✓ Desktop Chrome (Linux, Windows, macOS)
- ✓ Desktop Edge (Windows, macOS)
- ✓ iOS Safari (iPhone, iPad)
- ✓ Samsung Internet (Android)
```

## Window Controls Overlay (Advanced)

For desktop apps using `window-controls-overlay`, you can customize the title bar area.

### Detecting Window Controls Overlay

```javascript
if ('windowControlsOverlay' in navigator) {
    const wco = navigator.windowControlsOverlay;

    // Check if visible
    console.log('WCO visible:', wco.visible);

    // Get title bar dimensions
    const titleBarRect = wco.getTitlebarAreaRect();
    console.log('Title bar area:', {
        x: titleBarRect.x,
        y: titleBarRect.y,
        width: titleBarRect.width,
        height: titleBarRect.height
    });

    // Listen for changes (e.g., maximize/minimize)
    wco.addEventListener('geometrychange', () => {
        const rect = wco.getTitlebarAreaRect();
        console.log('Title bar resized:', rect);
    });
}
```

### Custom Title Bar with WCO

{% code title="public/css/window-controls.css" lineNumbers="true" %}
```css
/* Custom title bar styling */
body {
    margin: 0;
}

.title-bar {
    /* Reserve space for window controls */
    position: fixed;
    top: 0;
    left: 0;
    right: env(titlebar-area-x, 0);
    height: env(titlebar-area-height, 40px);

    background: var(--theme-color);
    color: white;
    display: flex;
    align-items: center;
    padding: 0 16px;

    /* Make draggable */
    -webkit-app-region: drag;
    app-region: drag;
}

.title-bar button {
    /* Buttons should not be draggable */
    -webkit-app-region: no-drag;
    app-region: no-drag;
}

/* Main content below title bar */
.main-content {
    margin-top: env(titlebar-area-height, 40px);
}
```
{% endcode %}

{% code title="templates/base.html.twig" lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html>
<head>
    <link rel="stylesheet" href="{{ asset('css/window-controls.css') }}">
</head>
<body>
    <div class="title-bar">
        <h1>{{ app_name }}</h1>
        <button>Settings</button>
    </div>
    <div class="main-content">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```
{% endcode %}

## Best Practices

### 1. Always Provide Fallbacks

```yaml
# ✗ Avoid - no fallback
display_override: ["fullscreen"]

# ✓ Good - has fallback
display_override: ["fullscreen"]
display: "standalone"
```

### 2. Order from Most to Least Specific

```yaml
# ✓ Good - specific to general
display_override:
    - "window-controls-overlay"  # Desktop-specific
    - "fullscreen"               # Platform-specific
    - "minimal-ui"               # Partially supported
display: "standalone"            # Widely supported
```

### 3. Consider Platform Differences

```yaml
# ✓ Good - works well across platforms
display_override:
    - "window-controls-overlay"  # Desktop benefit
display: "standalone"            # Mobile & desktop fallback
```

### 4. Test on Real Devices

Don't rely solely on DevTools:
- Install on actual Android device
- Install on actual iOS device
- Install on Windows/macOS/Linux desktop
- Test in different browsers

### 5. Provide CSS Fallbacks

```css
/* Adjust layout based on display mode */
@media (display-mode: fullscreen) {
    .app-header {
        display: none;  /* Hide header in fullscreen */
    }
}

@media (display-mode: standalone) {
    .app-header {
        position: sticky;
        top: 0;
    }
}

@media (display-mode: browser) {
    .app-header {
        position: relative;  /* Normal flow in browser */
    }
}
```

## Common Mistakes

### 1. No Fallback Display

**Problem**:
```yaml
display_override: ["fullscreen"]
# Missing display property
```

**Solution**:
```yaml
display_override: ["fullscreen"]
display: "standalone"  # Always include fallback
```

### 2. Wrong Order

**Problem**:
```yaml
display_override:
    - "standalone"          # Too generic first
    - "window-controls-overlay"  # Specific last
```

**Solution**:
```yaml
display_override:
    - "window-controls-overlay"  # Most specific first
    - "standalone"               # General last
```

### 3. Assuming Support

**Problem**:
```javascript
// Assuming window-controls-overlay works everywhere
const rect = navigator.windowControlsOverlay.getTitlebarAreaRect();
```

**Solution**:
```javascript
// Feature detection
if ('windowControlsOverlay' in navigator) {
    const rect = navigator.windowControlsOverlay.getTitlebarAreaRect();
    // Use rect
} else {
    // Fallback layout
}
```

### 4. Not Testing iOS

**Problem**: Assuming fullscreen works like on Android

**Solution**: Always test on iOS Safari - fullscreen is not supported, falls back to standalone

### 5. Ignoring Browser Tab Mode

**Problem**: Only designing for standalone mode

**Solution**: Ensure app works in browser tab mode too (progressive enhancement)

## Troubleshooting

### Display Mode Not Applied

**Problem**: App doesn't use expected display mode

**Checklist**:
- ✓ Check browser support for requested mode
- ✓ Verify manifest is valid JSON
- ✓ Ensure app is installed, not just bookmarked
- ✓ Check DevTools → Application → Manifest
- ✓ Try uninstalling and reinstalling PWA
- ✓ Clear browser cache

### Window Controls Overlay Not Working

**Problem**: Desktop app doesn't show window controls overlay

**Solutions**:
- Verify using desktop Chrome/Edge
- Check that PWA is installed (not running in tab)
- Ensure `display_override` includes `"window-controls-overlay"`
- Check DevTools console for errors
- Verify CSS uses correct environment variables

### Fullscreen Mode Issues on Mobile

**Problem**: App doesn't go fullscreen on mobile

**Solutions**:
- Check if platform supports fullscreen (Android: yes, iOS: no)
- Verify manifest has `display_override: ["fullscreen"]`
- Ensure app is launched from home screen
- Some browsers require user gesture for fullscreen

## Platform-Specific Behavior

### Android

- **fullscreen**: Fully supported, hides system UI
- **standalone**: Full support, standard app mode
- **minimal-ui**: May fallback to standalone
- **browser**: Standard browser tab

### iOS (Safari)

- **fullscreen**: Not supported, falls back to standalone
- **standalone**: Full support, hides Safari UI
- **minimal-ui**: Not supported, falls back to standalone
- **browser**: Standard Safari tab

### Desktop (Chrome/Edge)

- **window-controls-overlay**: Full support on Windows/macOS/Linux
- **fullscreen**: Supported (F11 equivalent)
- **standalone**: Creates separate window
- **minimal-ui**: Usually falls back to standalone

### Desktop (Firefox)

- Limited PWA support overall
- **standalone**: Some support
- Other modes: Limited or no support

## Related Documentation

- [Display Mode](../../the-manifest/application-information/display-mode.md) - Standard display property
- [Orientation](../../the-manifest/application-information/orientation.md) - Screen orientation control
- [Complete Example](../../the-manifest/complete-example.md) - Full manifest example

## Resources

- **MDN display_override**: https://developer.mozilla.org/en-US/docs/Web/Manifest/display_override
- **Window Controls Overlay**: https://developer.chrome.com/docs/web-platform/window-controls-overlay/
- **Display Modes Spec**: https://w3c.github.io/manifest/#display-modes
- **Can I Use**: https://caniuse.com/mdn-html_manifest_display_override

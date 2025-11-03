# EDGE Side Panel

{% hint style="warning" %}
**Microsoft Edge Only**: The `edge_side_panel` property is specific to Microsoft Edge and will be ignored by other browsers.
{% endhint %}

## Overview

Progressive Web Apps can opt-in to be pinned to the sidebar in Microsoft Edge. The sidebar allows users to easily access popular websites and utilities alongside their browser tabs, enabling side-by-side browsing without context switching.

**Key Features**:
- Always accessible alongside browser content
- Persistent across browser sessions
- Customizable width
- Perfect for utilities and reference apps
- Parallel browsing experience

## Use Cases

The Edge sidebar is ideal for applications that users want to access while browsing other content:

- **Productivity Tools**: Calculators, note-taking apps, task managers
- **Reference Apps**: Dictionary, translation tools, documentation viewers
- **Communication**: Chat apps, messaging platforms
- **Media**: Music players, podcast apps (can play while browsing)
- **Shopping**: Price comparison tools, shopping lists
- **Developer Tools**: API testers, JSON formatters, code snippets
- **Social Media**: Social network feeds, notifications

## Configuration

### Basic Configuration

Enable sidebar support without specifying width:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        edge_side_panel: ~
```
{% endcode %}

With this configuration, your PWA can be added to the Edge sidebar with default width.

### Specify Preferred Width

Set a preferred width for your sidebar panel:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        edge_side_panel:
            preferred_width: 400
```
{% endcode %}

**Width Guidelines**:
- **Minimum**: 200px (Edge enforced minimum)
- **Recommended**: 350-500px for most apps
- **Maximum**: No hard limit, but Edge may constrain to reasonable values

### Display Mode Considerations

For sidebar-only apps, use `browser` display mode:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        display: "browser"
        edge_side_panel:
            preferred_width: 400
```
{% endcode %}

Setting `display: "browser"` tells Edge that your app is designed for the sidebar and shouldn't be offered as a standalone installable app.

### Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "Calculator"
        short_name: "Calc"
        display: "browser"
        orientation: "portrait"
        edge_side_panel:
            preferred_width: 350
        icons:
            - src: "icons/icon-192.png"
              sizes: [192]
            - src: "icons/icon-512.png"
              sizes: [512]
```
{% endcode %}

## Width Recommendations by Application Type

### Calculator / Utilities

**Recommended**: 300-350px

```yaml
edge_side_panel:
    preferred_width: 320
```

**Reasoning**: Calculators don't need much horizontal space. Narrower width leaves more room for browser content.

### Note-Taking App

**Recommended**: 400-500px

```yaml
edge_side_panel:
    preferred_width: 450
```

**Reasoning**: Enough width for comfortable typing and reading notes.

### Chat / Messaging

**Recommended**: 350-400px

```yaml
edge_side_panel:
    preferred_width: 380
```

**Reasoning**: Standard chat interface width, similar to messaging apps.

### Documentation Viewer

**Recommended**: 500-600px

```yaml
edge_side_panel:
    preferred_width: 550
```

**Reasoning**: Documentation benefits from wider layout for code examples and text.

### Music Player

**Recommended**: 300-350px

```yaml
edge_side_panel:
    preferred_width: 320
```

**Reasoning**: Compact control interface, doesn't need much space.

### Translation Tool

**Recommended**: 400-450px

```yaml
edge_side_panel:
    preferred_width: 420
```

**Reasoning**: Needs space for source and translated text.

## Browser Support

| Browser | Support |
|---------|---------|
| Microsoft Edge | ✅ Full support |
| Chrome | ❌ Not supported (property ignored) |
| Safari | ❌ Not supported (property ignored) |
| Firefox | ❌ Not supported (property ignored) |

{% hint style="info" %}
**Graceful Degradation**: Other browsers will simply ignore the `edge_side_panel` property. Your app will work normally in those browsers.
{% endhint %}

## Responsive Design

Design your app to work both in the sidebar and as a standalone page:

### CSS Media Queries

{% code title="public/css/responsive.css" lineNumbers="true" %}
```css
/* Default layout */
.app-container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

/* Sidebar layout (narrow viewport) */
@media (max-width: 600px) {
    .app-container {
        padding: 10px;
    }

    /* Simplify layout for narrow space */
    .sidebar-only {
        display: none;
    }

    /* Stack elements vertically */
    .flex-row {
        flex-direction: column;
    }

    /* Reduce font sizes */
    h1 {
        font-size: 1.5rem;
    }

    /* Compact buttons */
    .btn {
        padding: 8px 12px;
        font-size: 14px;
    }
}

/* Specific styling for very narrow sidebar */
@media (max-width: 400px) {
    .app-container {
        padding: 5px;
    }

    .detailed-view {
        display: none;
    }

    .compact-view {
        display: block;
    }
}
```
{% endcode %}

### JavaScript Detection

Detect narrow viewport and adjust functionality:

{% code title="public/app.js" lineNumbers="true" %}
```javascript
// Detect if running in sidebar (narrow viewport)
function isSidebarMode() {
    return window.innerWidth < 600;
}

function adaptToViewport() {
    if (isSidebarMode()) {
        // Sidebar mode adjustments
        document.body.classList.add('sidebar-mode');

        // Simplify interface
        hideDetailedViews();
        enableCompactMode();

        console.log('Running in sidebar mode');
    } else {
        // Full app mode
        document.body.classList.remove('sidebar-mode');

        // Full interface
        showDetailedViews();
        disableCompactMode();

        console.log('Running in full mode');
    }
}

// Adapt on load
adaptToViewport();

// Adapt on resize (if user resizes sidebar)
window.addEventListener('resize', adaptToViewport);
```
{% endcode %}

## User Experience Best Practices

### 1. Design for Narrow Viewports

```css
/* Prioritize vertical scrolling over horizontal */
.content {
    overflow-x: hidden;
    overflow-y: auto;
}

/* Use full width available */
.panel {
    width: 100%;
    box-sizing: border-box;
}
```

### 2. Minimize Horizontal Elements

```html
<!-- ✗ Avoid wide horizontal layouts -->
<div class="horizontal-menu">
    <button>Button 1</button>
    <button>Button 2</button>
    <button>Button 3</button>
    <button>Button 4</button>
</div>

<!-- ✓ Use vertical or icon-based layouts -->
<nav class="vertical-menu">
    <button><icon>📊</icon></button>
    <button><icon>⚙️</icon></button>
    <button><icon>📁</icon></button>
    <button><icon>👤</icon></button>
</nav>
```

### 3. Use Collapsible Sections

```html
<details class="section">
    <summary>Settings</summary>
    <div class="section-content">
        <!-- Settings content -->
    </div>
</details>

<details class="section">
    <summary>History</summary>
    <div class="section-content">
        <!-- History content -->
    </div>
</details>
```

### 4. Optimize Text Readability

```css
/* Ensure readable text in narrow space */
body {
    font-size: 14px;
    line-height: 1.5;
}

/* Break long words */
.text-content {
    word-wrap: break-word;
    overflow-wrap: break-word;
}

/* Prevent text overflow */
.truncate {
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}
```

### 5. Provide Clear Actions

```html
<!-- Clear, tappable actions -->
<button class="action-btn" style="
    display: block;
    width: 100%;
    padding: 12px;
    margin: 8px 0;
    font-size: 16px;">
    Add New Item
</button>
```

## Testing in Edge

### 1. Enable Sidebar Testing

```bash
1. Open Microsoft Edge
2. Go to edge://flags
3. Search for "Desktop PWA side panel"
4. Enable the flag
5. Restart Edge
```

### 2. Install PWA to Sidebar

```bash
1. Navigate to your PWA
2. Click "..." menu → Apps → Install this site as an app
3. In the Edge sidebar, click "+" to add app
4. Select your PWA from the list
5. Your app appears in the sidebar
```

### 3. Test Width Adjustment

```bash
1. Open your app in sidebar
2. Drag the sidebar edge to resize
3. Observe how your app adapts
4. Test at minimum width (~200px)
5. Test at preferred width
6. Test at maximum sidebar width
```

### 4. Test Functionality

```bash
# Checklist
- ✓ All features accessible in narrow viewport
- ✓ No horizontal scrolling required
- ✓ Text is readable
- ✓ Buttons are tappable
- ✓ Forms are usable
- ✓ Images load and scale properly
- ✓ Navigation works smoothly
- ✓ App remembers state when reopened
```

## Complete Examples

### Example 1: Calculator App

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Quick Calculator"
        short_name: "Calculator"
        description: "Simple calculator for quick calculations"
        display: "browser"
        edge_side_panel:
            preferred_width: 300
        theme_color: "#2196F3"
        background_color: "#ffffff"
        icons:
            - src: "icons/calc-192.png"
              sizes: [192]
            - src: "icons/calc-512.png"
              sizes: [512]
```
{% endcode %}

### Example 2: Notes App

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Quick Notes"
        short_name: "Notes"
        description: "Take quick notes while browsing"
        display: "browser"
        edge_side_panel:
            preferred_width: 450
        theme_color: "#FFC107"
        icons:
            - src: "icons/notes-192.png"
              sizes: [192]
```
{% endcode %}

### Example 3: Music Player

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Music Player"
        short_name: "Music"
        description: "Listen to music while browsing"
        display: "browser"
        edge_side_panel:
            preferred_width: 350
        theme_color: "#E91E63"
        icons:
            - src: "icons/music-192.png"
              sizes: [192]
```
{% endcode %}

### Example 4: Translation Tool

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Quick Translate"
        short_name: "Translate"
        description: "Translate text while browsing"
        display: "browser"
        edge_side_panel:
            preferred_width: 420
        theme_color: "#4CAF50"
        icons:
            - src: "icons/translate-192.png"
              sizes: [192]
```
{% endcode %}

## Common Issues

### App Not Appearing in Sidebar

**Problem**: PWA doesn't show up in Edge sidebar app list

**Solutions**:
- Ensure you're using Microsoft Edge (not Chrome/other browsers)
- Verify PWA is installed (edge://apps)
- Check that `edge_side_panel` is configured in manifest
- Restart Edge browser
- Check Edge version supports sidebar PWAs

### Width Not Respected

**Problem**: Sidebar width doesn't match `preferred_width`

**Reasons**:
- Edge enforces minimum width (~200px)
- Edge may constrain width based on screen size
- User can manually resize sidebar
- `preferred_width` is a suggestion, not a requirement

**Solution**: Design app to work across range of widths (200-600px)

### Content Overflows

**Problem**: Content extends beyond sidebar viewport

**Solutions**:
```css
/* Prevent horizontal overflow */
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    overflow-x: hidden;
}

.container {
    max-width: 100%;
    padding: 10px;
}

img {
    max-width: 100%;
    height: auto;
}
```

### App Opens in New Window

**Problem**: Clicking links opens new windows instead of navigating in sidebar

**Solutions**:
```html
<!-- Use relative links -->
<a href="/page">Link</a>

<!-- Or use JavaScript navigation -->
<a href="#" onclick="navigate('/page'); return false;">Link</a>
```

```javascript
function navigate(url) {
    window.location.href = url;
    // Stays in sidebar
}
```

## Debugging

### Check Manifest in DevTools

```bash
1. Open your PWA in Edge
2. Press F12 (DevTools)
3. Go to Application tab
4. Click "Manifest" in sidebar
5. Look for "edge_side_panel" section
6. Verify preferred_width value
```

### Console Logging

```javascript
// Log viewport information
console.log('Viewport width:', window.innerWidth);
console.log('Viewport height:', window.innerHeight);
console.log('Is sidebar mode:', window.innerWidth < 600);

// Monitor resize events
window.addEventListener('resize', () => {
    console.log('Viewport resized to:', window.innerWidth, 'x', window.innerHeight);
});
```

### Test Across Widths

```javascript
// Simulate different widths (in DevTools)
const widths = [200, 300, 400, 500, 600];

widths.forEach(width => {
    // Set viewport width in DevTools
    console.log(`Testing at ${width}px width`);
    // Manually check layout at each width
});
```

## Best Practices Summary

1. **Set appropriate width**: 300-500px for most apps
2. **Use `display: "browser"`**: For sidebar-only apps
3. **Design responsive**: Support 200-600px widths
4. **Avoid horizontal scrolling**: Stack elements vertically
5. **Test in actual Edge sidebar**: Don't rely on DevTools alone
6. **Provide fallback**: App should work in other browsers too
7. **Optimize text**: Ensure readability in narrow space
8. **Use icons**: Save space with icon-based navigation
9. **Implement collapsible sections**: Maximize vertical space
10. **Remember state**: Persist user data across sessions

## Related Documentation

- [Display Mode](../../the-manifest/application-information/display-mode.md) - App display modes
- [Orientation](../../the-manifest/application-information/orientation.md) - Screen orientation
- [Icons](../../the-manifest/icons.md) - App icons configuration

## Resources

- **Microsoft Edge Sidebar**: https://learn.microsoft.com/en-us/microsoft-edge/progressive-web-apps-chromium/how-to/sidebar
- **Edge PWA Documentation**: https://learn.microsoft.com/en-us/microsoft-edge/progressive-web-apps-chromium/
- **Edge DevTools**: https://learn.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/

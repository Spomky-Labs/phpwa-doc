# Orientation

The `orientation` property controls the default screen orientation when your PWA is launched. This ensures your application displays optimally based on its content and intended user experience.

## Purpose

Screen orientation affects:
- **User interface layout**: How content is arranged on screen
- **User experience**: Natural interaction patterns for your app type
- **Media consumption**: Optimal viewing for videos, games, or documents
- **Device ergonomics**: How users hold and interact with their device

## Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        orientation: "portrait"  # or landscape, any, etc.
```
{% endcode %}

{% hint style="info" %}
If not specified, the orientation defaults to `any`, allowing the device to rotate freely.
{% endhint %}

## Available Orientations

### any (Default)

Allows the application to rotate freely based on device orientation.

```yaml
orientation: "any"
```

**Best for**:
- Multi-purpose applications
- Apps with responsive layouts
- Content that works well in both orientations
- Most general-purpose apps

**Examples**: Social media apps, news readers, general productivity tools

### portrait

Locks to portrait orientation (vertical), allowing both normal and inverted portrait.

```yaml
orientation: "portrait"
```

**Best for**:
- Reading-focused applications
- Social media feeds
- Chat applications
- Form-heavy applications
- One-handed mobile use

**Examples**:
- Instagram-like photo sharing
- Messaging apps
- News articles
- Shopping apps with vertical scrolling

### portrait-primary

Locks to the device's primary portrait orientation (typically upright, not inverted).

```yaml
orientation: "portrait-primary"
```

**Best for**:
- Applications requiring consistent UI position
- Apps using device sensors
- Camera-based applications
- Applications with fixed control positions

**Examples**:
- Camera apps
- QR code scanners
- Fitness tracking apps
- Applications with bottom navigation

### landscape

Locks to landscape orientation (horizontal), allowing both left and right landscape.

```yaml
orientation: "landscape"
```

**Best for**:
- Video playback applications
- Games
- Presentation tools
- Media consumption
- Spreadsheets or wide data views

**Examples**:
- Video streaming apps
- Gaming platforms
- Dashboard applications
- Photo galleries

### landscape-primary

Locks to the device's primary landscape orientation.

```yaml
orientation: "landscape-primary"
```

**Best for**:
- Games with specific control layouts
- Applications using device sensors in landscape
- Car dashboard apps
- Presentation apps

**Examples**:
- Racing games
- Flight simulators
- Car navigation apps
- DJ/Music production apps

## Use Cases by Application Type

### Social Media App

```yaml
pwa:
    manifest:
        name: "SocialConnect"
        orientation: "portrait"  # Vertical scrolling is natural
```

**Rationale**: Content feeds are designed for vertical scrolling, and users typically hold phones in portrait mode.

### Video Streaming App

```yaml
pwa:
    manifest:
        name: "StreamFlix"
        orientation: "any"  # Switch to landscape for fullscreen
```

**Rationale**: Browse in portrait, watch videos in landscape. Flexibility is key.

### Gaming App

```yaml
pwa:
    manifest:
        name: "SpaceRacer"
        orientation: "landscape-primary"  # Fixed game controls
```

**Rationale**: Game controls are designed for landscape, and consistency is crucial.

### E-book Reader

```yaml
pwa:
    manifest:
        name: "BookReader"
        orientation: "portrait-primary"  # Reading comfort
```

**Rationale**: Reading text is more comfortable in portrait mode, mimicking physical books.

### Dashboard App

```yaml
pwa:
    manifest:
        name: "Analytics Dashboard"
        orientation: "landscape"  # More horizontal space for charts
```

**Rationale**: Charts and data visualizations benefit from wider screens.

### Fitness Tracker

```yaml
pwa:
    manifest:
        name: "FitTrack"
        orientation: "portrait-primary"  # One-handed use during exercise
```

**Rationale**: Users often interact while exercising, requiring one-handed portrait mode.

## Complete Examples

### Portrait-Only Application

```yaml
pwa:
    manifest:
        name: "ChatApp"
        short_name: "Chat"
        orientation: "portrait"
        display: "standalone"
        theme_color: "#2196F3"
        description: "Stay connected with friends and family through instant messaging."
```

### Landscape-Only Application

```yaml
pwa:
    manifest:
        name: "VideoEditor"
        short_name: "Editor"
        orientation: "landscape"
        display: "fullscreen"
        theme_color: "#FF5722"
        description: "Create and edit videos with professional tools."
```

### Flexible Orientation Application

```yaml
pwa:
    manifest:
        name: "TaskManager"
        short_name: "Tasks"
        orientation: "any"  # Adapts to user preference
        display: "standalone"
        theme_color: "#4CAF50"
        description: "Organize your tasks and boost productivity."
```

## Platform Behavior

### iOS (iPhone/iPad)

- Respects orientation lock if device rotation is disabled in settings
- `portrait-primary` and `landscape-primary` may behave differently based on device
- iPad may ignore orientation locks in some contexts
- Supports all standard orientations

### Android

- Strongly respects manifest orientation
- Users can override with device settings
- Different behavior in tablet vs phone mode
- Chrome and other browsers may handle differently

### Desktop/Laptop

- Orientation setting is typically ignored
- Windows treated as landscape by default
- Application runs in window mode
- Resizing window doesn't trigger orientation changes

## Best Practices

### 1. Choose Based on Content

Match orientation to your primary use case:

```yaml
# Reading app - portrait is natural
orientation: "portrait"

# Video app - allow both
orientation: "any"

# Game - lock to landscape
orientation: "landscape-primary"
```

### 2. Consider User Context

Think about how users will hold their device:

- **One-handed use**: `portrait` or `portrait-primary`
- **Two-handed comfortable viewing**: `landscape`
- **Variable situations**: `any`

### 3. Test on Real Devices

Different devices handle orientation differently:

```bash
# Test on various devices
- iPhone (small)
- iPad (large, different aspect ratio)
- Android phone (various sizes)
- Android tablet
- Desktop browser
```

### 4. Provide Responsive Design Regardless

Even with orientation lock, design responsively:

```css
/* Support both orientations in CSS */
@media (orientation: portrait) {
    /* Portrait-specific styles */
}

@media (orientation: landscape) {
    /* Landscape-specific styles */
}
```

### 5. Don't Lock Unnecessarily

Only lock orientation when truly needed:

```yaml
# ✗ Poor - locks without reason
orientation: "portrait"  # For a general-purpose app

# ✓ Good - allows natural rotation
orientation: "any"  # For a general-purpose app
```

## Advanced: Handling Orientation Changes

Even with a locked orientation in the manifest, handle rotation in your JavaScript:

```javascript
// Listen for orientation changes
window.addEventListener('orientationchange', () => {
    const orientation = screen.orientation.type;
    console.log('Orientation changed to:', orientation);

    // Adjust UI accordingly
    if (orientation.includes('portrait')) {
        adjustForPortrait();
    } else {
        adjustForLandscape();
    }
});

// Or use matchMedia
const isPortrait = window.matchMedia('(orientation: portrait)');

isPortrait.addEventListener('change', (e) => {
    if (e.matches) {
        console.log('Now in portrait mode');
    } else {
        console.log('Now in landscape mode');
    }
});
```

## Accessibility Considerations

### 1. Don't Restrict User Choice

Some users have accessibility needs requiring specific orientations:

- **Motor disabilities**: May need landscape for larger touch targets
- **Visual impairments**: May prefer portrait for easier reading
- **Physical limitations**: May only be able to hold device one way

**Recommendation**: Use `any` unless orientation lock is essential.

### 2. Provide Clear Feedback

If your app requires a specific orientation, inform users:

```javascript
// Show message if user is in wrong orientation
if (requiredOrientation === 'landscape' && isPortrait()) {
    showRotationPrompt('Please rotate your device to landscape mode');
}
```

### 3. Test with Screen Readers

Ensure orientation changes don't disrupt screen reader navigation:

```html
<!-- Announce orientation changes -->
<div role="status" aria-live="polite" id="orientation-status"></div>

<script>
window.addEventListener('orientationchange', () => {
    document.getElementById('orientation-status').textContent =
        `Screen orientation changed to ${screen.orientation.type}`;
});
</script>
```

## Common Mistakes

### 1. Locking When Not Needed

```yaml
# ✗ Poor - unnecessary restriction
pwa:
    manifest:
        name: "GeneralApp"
        orientation: "portrait"  # No specific reason
```

**Impact**: Frustrates users who want to use landscape mode.

### 2. Wrong Orientation for Content

```yaml
# ✗ Poor - video app locked to portrait
pwa:
    manifest:
        name: "VideoApp"
        orientation: "portrait"  # Videos are better in landscape
```

**Impact**: Poor user experience for primary use case.

### 3. Not Testing Both Orientations

Even with a locked orientation, test your layout in both:

- Users may override in settings
- Tablets behave differently
- Desktop windows can be any size

### 4. Ignoring CSS Media Queries

Manifest orientation doesn't replace responsive CSS:

```css
/* ✓ Still implement responsive design */
@media (orientation: landscape) {
    .content { flex-direction: row; }
}

@media (orientation: portrait) {
    .content { flex-direction: column; }
}
```

## Debugging Orientation

### 1. Check Manifest in DevTools

```bash
# Chrome DevTools
1. Open DevTools (F12)
2. Go to Application tab
3. Select "Manifest" in sidebar
4. Check "orientation" value
```

### 2. Test Orientation Lock

```javascript
// Check current orientation
console.log('Type:', screen.orientation.type);
console.log('Angle:', screen.orientation.angle);

// Available types:
// - portrait-primary (0°)
// - portrait-secondary (180°)
// - landscape-primary (90° or -90°)
// - landscape-secondary (-90° or 90°)
```

### 3. Simulate on Desktop

Chrome DevTools device emulation:

```bash
1. Open DevTools
2. Click device toolbar icon (Ctrl+Shift+M)
3. Select device
4. Click rotation icon to toggle orientation
```

## Related Properties

- [Display Mode](../display.md) - How the app window is presented
- [Theme Color](colors-and-theme.md) - Visual appearance
- [Scope](scope.md) - URL scope for the app

## Summary

Orientation best practices:
- ✓ Use `any` unless you have a specific reason to lock
- ✓ Match orientation to your primary use case (video → landscape, reading → portrait)
- ✓ Test on real devices in both orientations
- ✓ Provide responsive CSS regardless of manifest setting
- ✓ Consider accessibility needs before locking orientation
- ✓ Inform users if a specific orientation is required
- ✗ Don't lock orientation unnecessarily
- ✗ Don't forget to test orientation changes even when locked

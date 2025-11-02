# Startup Images

Startup images (also called splash screens) are displayed when launching your PWA from the home screen. They provide visual feedback during app initialization, creating a smoother, more app-like experience.

## Why Startup Images Matter

**User Experience:**
- Smooth transition from home screen to app
- Professional first impression
- Reduces perceived loading time
- Masks initial page rendering

**Platform Expectations:**
- iOS users expect splash screens like native apps
- Android handles this automatically with icon + background
- Builds trust and credibility

**Branding:**
- Reinforces brand identity
- Consistent across app launches
- Matches app theme and design

## How It Works

The startup image generation differs by platform:

### Android

Android automatically generates splash screens using:
- Your app icon (from manifest)
- Background color (from manifest `background_color`)
- No additional configuration needed

**What Android shows:**
1. Centered app icon
2. Solid background color
3. Short duration (until app loads)

### iOS

iOS requires specific splash screen images for each device size and orientation:

**Device variations:**
- iPhone models (SE, 8, X, 11, 12, 13, 14, 15, etc.)
- iPad models (different sizes)
- Portrait and landscape orientations

**The challenge:** This traditionally requires dozens of different image files!

**Bundle solution:** Automatically generates all required sizes from your icon and background color.

## Automatic Generation

By default, when you configure favicons, startup images are automatically created:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        background_color: '#ffffff'  # Used for startup images
```
{% endcode %}

**What gets generated:**
- iOS portrait images (all device sizes)
- iOS landscape images (all device sizes)
- Correct media queries in HTML
- Dark mode variants (if configured)

{% hint style="success" %}
The bundle generates 30+ startup images automatically to cover all iOS devices and orientations!
{% endhint %}

## Configuration Options

### Basic Setup

The default configuration works for most cases:

```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        background_color: '#ffffff'
```

### Disable Startup Images

If you don't want startup images (not recommended for iOS):

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        use_start_image: false  # Disable startup image generation
```
{% endcode %}

**Why you might disable:**
- Very simple web apps
- Performance concerns (fewer files)
- Custom splash screen implementation

**Downsides:**
- iOS shows white screen during launch
- Less professional appearance
- Missing expected UX on iOS

### Dark Mode Startup Images

Dark mode icons automatically generate dark startup images:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true

        # Light mode
        default:
            src: assets/icon.svg
            background_color: '#ffffff'

        # Dark mode
        dark:
            src: assets/icon_dark.svg
            background_color: '#000000'
```
{% endcode %}

**Result:**
- Light startup images for light mode (from `default`)
- Dark startup images for dark mode (from `dark`)
- Automatic switching based on system theme
- All device sizes generated for both themes

### Custom Background Colors

Choose backgrounds that match your app's theme:

```yaml
pwa:
    favicons:
        enabled: true

        # Light theme with brand color
        default:
            src: assets/icon.svg
            background_color: '#2196f3'  # Material blue

        # Dark theme
        dark:
            src: assets/icon_dark.svg
            background_color: '#121212'  # Near black
```

**Color examples:**
- Light gray: `#f5f5f5`
- White: `#ffffff`
- Brand colors: `#2196f3`, `#ff5722`, etc.
- Dark gray: `#121212`
- Black: `#000000`

**Best practices:**
- Match your app's `theme_color` from manifest
- Ensure good contrast with icon
- Use colors that represent your brand
- Keep consistency between light and dark themes

## Generated Files

The bundle creates startup images for all iOS devices:

**iPhone Portrait:**
- iPhone SE (640x1136)
- iPhone 8 (750x1334)
- iPhone 8 Plus (1242x2208)
- iPhone X/11 Pro (1125x2436)
- iPhone 11/XR (828x1792)
- iPhone 11 Pro Max (1242x2688)
- iPhone 12/13 mini (1080x2340)
- iPhone 12/13/14 (1170x2532)
- iPhone 14 Plus (1284x2778)
- iPhone 15 Pro Max (1290x2796)

**iPhone Landscape:**
- All above devices in landscape orientation

**iPad Portrait & Landscape:**
- iPad (various sizes)
- iPad Pro 11"
- iPad Pro 12.9"

**Total:** 30+ images covering all combinations

## HTML Output

The bundle injects all necessary HTML tags:

```html
<!-- Light mode startup images -->
<link rel="apple-touch-startup-image"
      media="(device-width: 430px) and (device-height: 932px) and (orientation: portrait)"
      href="/startup-430x932-portrait.png">

<!-- Dark mode startup images -->
<link rel="apple-touch-startup-image"
      media="(prefers-color-scheme: dark) and (device-width: 430px) and (device-height: 932px) and (orientation: portrait)"
      href="/startup-430x932-portrait-dark.png">

<!-- ...30+ more variations... -->
```

All this is automatic - you don't write any HTML!

## Best Practices

1. **Always enable startup images**: Essential for iOS PWAs
2. **Match theme colors**: Use same `background_color` as manifest
3. **Provide dark mode**: Better user experience
4. **Keep icon centered**: Works best for startup screens
5. **Test on real devices**: Verify appearance on iPhone and iPad

## Troubleshooting

### Startup images not showing

**iOS checklist:**
1. PWA installed to home screen (not running in Safari)
2. `use_start_image` is not set to `false`
3. `background_color` is configured
4. Assets compiled (`pwa:compile`)
5. `{{ pwa() }}` function in HTML

### Wrong colors appearing

**Check:**
- `background_color` matches your intention
- Dark mode variant configured if needed
- Cache cleared after changes
- Recompile assets

### Images look pixelated

**Ensure:**
- Source icon is high resolution (512x512+)
- Using SVG (preferred) or large PNG
- `image_scale` not set too high

## Performance Considerations

**File size impact:**
- Each startup image: 10-50 KB
- Total for all images: ~500 KB - 1 MB
- Cached by browser after first launch

**Optimization:**
- Bundle automatically optimizes images
- PNG compression applied
- Only downloaded as needed (per device)

**When to disable:**
- Extremely bandwidth-constrained users
- Very simple utility apps
- Custom splash implementation

{% hint style="info" %}
For most PWAs, the user experience benefit of startup images far outweighs the minimal performance impact.
{% endhint %}

## Example Configurations

### Minimal (recommended)

```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        background_color: '#ffffff'
# use_start_image defaults to true
```

### With Dark Mode

```yaml
pwa:
    favicons:
        enabled: true
        default:
            src: assets/icon.svg
            background_color: '#ffffff'
        dark:
            src: assets/icon_dark.svg
            background_color: '#1a1a1a'
```

### Brand Colors

```yaml
pwa:
    favicons:
        enabled: true
        default:
            src: assets/logo.svg
            background_color: '#2196f3'  # Brand blue
            svg_attr:
                fill: '#ffffff'           # White logo
```

### Disabled (not recommended)

```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        use_start_image: false  # Only if you have a good reason
```

## Comparing to Native Apps

**Native iOS app:**
- Requires manual creation of 30+ splash images
- Must use Xcode and asset catalogs
- Updates require resubmission

**PWA with this bundle:**
- Automatic generation from one icon
- No manual image creation
- Updates deploy instantly

This is one of many ways PWAs simplify app development!


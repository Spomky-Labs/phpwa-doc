# Icon Management

{% hint style="warning" %}
**Deprecated**: The command `pwa:create:icons` is deprecated and has been removed. Use modern icon generation tools and configure icons directly in your manifest.
{% endhint %}

## Overview

PWA icons are configured in your manifest file and served through AssetMapper or absolute paths. The bundle no longer provides icon generation commands - instead, use dedicated tools or services.

## Icon Configuration

Icons are defined in your manifest configuration. See the complete [Icons documentation](../the-manifest/icons.md) for detailed configuration options.

### Basic Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        icons:
            - src: "icons/icon-192.png"
              sizes: [192]
            - src: "icons/icon-512.png"
              sizes: [512]
            - src: "icons/maskable-icon-512.png"
              sizes: [512]
              purpose: "maskable"
```
{% endcode %}

## Recommended Icon Generation Tools

### Online Tools

**1. PWA Asset Generator**
- https://www.pwabuilder.com/imageGenerator
- Generates all required icon sizes
- Creates maskable icons automatically
- Free and easy to use

**2. RealFaviconGenerator**
- https://realfavicongenerator.net/
- Comprehensive favicon and icon generation
- Tests icons on multiple platforms
- Generates HTML meta tags

**3. Maskable.app**
- https://maskable.app/
- Specialized for maskable icon creation
- Visual safe zone editor
- Preview on different devices

**4. Favicon.io**
- https://favicon.io/
- Simple and fast
- Generates from text, image, or emoji
- Free to use

### Command-Line Tools

**1. PWA Asset Generator (CLI)**
```bash
npx pwa-asset-generator source-icon.png public/icons \
    --background "#2196F3" \
    --icon-only \
    --maskable
```

**2. Sharp (Node.js)**
```javascript
const sharp = require('sharp');

// Generate icon sizes
const sizes = [48, 72, 96, 144, 192, 256, 512];

sizes.forEach(size => {
    sharp('source-icon.png')
        .resize(size, size)
        .toFile(`public/icons/icon-${size}.png`);
});
```

**3. ImageMagick**
```bash
# Generate multiple sizes
for size in 48 72 96 144 192 256 512; do
    convert source-icon.png -resize ${size}x${size} icons/icon-${size}.png
done
```

## Icon Requirements

### Required Sizes

At minimum, provide these sizes:

```yaml
icons:
    - src: "icons/icon-192.png"
      sizes: [192]              # Android Chrome
    - src: "icons/icon-512.png"
      sizes: [512]              # Android Chrome, install prompt
```

### Recommended Sizes

For comprehensive support:

```yaml
icons:
    - src: "icons/icon-48.png"
      sizes: [48]               # Windows taskbar
    - src: "icons/icon-72.png"
      sizes: [72]               # iOS, iPad
    - src: "icons/icon-96.png"
      sizes: [96]               # Android launcher
    - src: "icons/icon-144.png"
      sizes: [144]              # Windows desktop
    - src: "icons/icon-192.png"
      sizes: [192]              # Android Chrome
    - src: "icons/icon-256.png"
      sizes: [256]              # Windows Store
    - src: "icons/icon-512.png"
      sizes: [512]              # High-res displays
```

### Maskable Icons

Always include maskable icons for better Android integration:

```yaml
icons:
    - src: "icons/maskable-icon-192.png"
      sizes: [192]
      purpose: "maskable"
    - src: "icons/maskable-icon-512.png"
      sizes: [512]
      purpose: "maskable"
```

## Icon Design Guidelines

### Standard Icons

- **Format**: PNG (preferred) or JPEG
- **Background**: Solid color or transparent
- **Content**: Icon centered, using ~80% of canvas
- **Design**: Simple, recognizable at small sizes

### Maskable Icons

- **Safe zone**: Keep important content in center 80%
- **Background**: Must fill entire canvas (no transparency)
- **Bleeding**: Design can extend to edges
- **Testing**: Use https://maskable.app/ to verify

### Design Tips

1. **Simplicity**: Icons should be recognizable at 48x48
2. **Contrast**: Ensure good contrast with background
3. **Consistency**: Match your brand colors
4. **Scalability**: Design should work at all sizes
5. **Testing**: Test on actual devices

## File Organization

### Recommended Structure

```
assets/
└── icons/
    ├── icon-48.png
    ├── icon-72.png
    ├── icon-96.png
    ├── icon-144.png
    ├── icon-192.png
    ├── icon-256.png
    ├── icon-512.png
    ├── maskable-icon-192.png
    ├── maskable-icon-512.png
    └── icon.svg (optional, for 'any' size)
```

### Configuration Example

```yaml
pwa:
    manifest:
        icons:
            # Standard icons
            - src: "icons/icon-48.png"
              sizes: [48]
            - src: "icons/icon-72.png"
              sizes: [72]
            - src: "icons/icon-96.png"
              sizes: [96]
            - src: "icons/icon-144.png"
              sizes: [144]
            - src: "icons/icon-192.png"
              sizes: [192]
            - src: "icons/icon-256.png"
              sizes: [256]
            - src: "icons/icon-512.png"
              sizes: [512]

            # Maskable icons
            - src: "icons/maskable-icon-192.png"
              sizes: [192]
              purpose: "maskable"
            - src: "icons/maskable-icon-512.png"
              sizes: [512]
              purpose: "maskable"

            # Vector icon (optional)
            - src: "icons/icon.svg"
              sizes: 0  # 'any' size
```

## Using Symfony UX Icons

If you have `symfony/ux-icons` installed, you can use icon libraries:

```yaml
pwa:
    manifest:
        icons:
            - src: "bx:badge-check"
              sizes: [192]
            - src: "heroicons:check-circle"
              sizes: [512]
```

See [Symfony UX Icons](https://ux.symfony.com/icons) for available icons.

## Migration from Deprecated Command

### Old Approach (Deprecated)

```bash
# This no longer works
php bin/console pwa:create:icons
```

### New Approach

1. **Generate icons** using online tools or CLI tools
2. **Place icons** in your `assets/icons/` directory
3. **Configure** in `pwa.yaml`:

```yaml
pwa:
    manifest:
        icons:
            - src: "icons/icon-192.png"
              sizes: [192]
            - src: "icons/icon-512.png"
              sizes: [512]
```

4. **Compile assets**:
```bash
php bin/console asset-map:compile
```

## Testing Icons

### 1. Lighthouse PWA Audit

```bash
# Run Lighthouse
npx lighthouse https://your-app.com --view
```

Check the PWA section for icon requirements.

### 2. Chrome DevTools

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "Icons" section
4. Verify all sizes are present
```

### 3. Install on Device

- **Android**: Install PWA, check app drawer icon
- **iOS**: Add to home screen, check icon appearance
- **Desktop**: Install, check taskbar/dock icon

### 4. Maskable Icon Tester

Visit https://maskable.app/editor and upload your maskable icons to verify the safe zone.

## Common Issues

### Icons Not Appearing

**Problem**: Icons don't show after installation

**Solutions**:
- Clear browser cache
- Check icon paths are correct
- Verify icons are served (check network tab)
- Ensure AssetMapper compiled assets
- Check file permissions

### Wrong Icon Size

**Problem**: Icon appears pixelated or blurry

**Solutions**:
- Provide higher resolution icons (512px minimum)
- Include appropriate size for each platform
- Use vector SVG for scalability

### Maskable Icons Not Adapting

**Problem**: Maskable icons get cropped incorrectly

**Solutions**:
- Ensure safe zone is respected (center 80%)
- Test with https://maskable.app/
- Provide solid background (no transparency)
- Check purpose is set to "maskable"

## Platform-Specific Icons

### Android

**Requirements**:
- Minimum: 192x192 and 512x512
- Format: PNG with transparency or solid background
- Maskable: Highly recommended

```yaml
icons:
    - src: "icons/android-192.png"
      sizes: [192]
    - src: "icons/android-512.png"
      sizes: [512]
    - src: "icons/android-maskable-512.png"
      sizes: [512]
      purpose: "maskable"
```

### iOS

**Requirements**:
- Recommended: 180x180 for Apple Touch Icon
- Format: PNG, no transparency
- Additional meta tag may be needed

```html
<!-- In your HTML head -->
<link rel="apple-touch-icon" href="/icons/apple-touch-icon.png">
```

### Windows

**Requirements**:
- Recommended: 144x144, 256x256
- Format: PNG or ICO
- May require separate configuration

```yaml
icons:
    - src: "icons/windows-144.png"
      sizes: [144]
    - src: "icons/windows-256.png"
      sizes: [256]
```

## Advanced Icon Features

For advanced icon configuration including rounded corners, image scaling, and background colors, see the complete [Icons documentation](../the-manifest/icons.md).

## Related Documentation

- [Icons (Manifest)](../the-manifest/icons.md) - Complete icon configuration
- [Screenshots](screenshots.md) - App screenshot management
- [AssetMapper](https://symfony.com/doc/current/frontend/asset_mapper.html) - Asset management
- [Symfony UX Icons](https://ux.symfony.com/icons) - Icon library integration

## Resources

- **PWA Builder**: https://www.pwabuilder.com/
- **Maskable.app**: https://maskable.app/
- **Favicon Generator**: https://realfavicongenerator.net/
- **MDN Icons**: https://developer.mozilla.org/en-US/docs/Web/Manifest/icons
- **Web.dev PWA Icons**: https://web.dev/add-manifest/#icons

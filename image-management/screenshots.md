# Screenshot Management

{% hint style="warning" %}
**Deprecated**: The command `pwa:create:screenshot` is deprecated and has been removed. Create screenshots manually or use screen capture tools.
{% endhint %}

## Overview

Screenshots enhance your PWA's installation experience by showing users what your app looks like before they install it. They appear in app stores, installation prompts, and PWA directories.

## Screenshot Configuration

Screenshots are defined in your manifest configuration. See the complete [Screenshots documentation](../the-manifest/screenshots.md) for detailed configuration options.

### Basic Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        screenshots:
            - src: "screenshots/home.png"
              sizes: [1280, 720]
              form_factor: "wide"
            - src: "screenshots/mobile-home.png"
              sizes: [750, 1334]
              form_factor: "narrow"
```
{% endcode %}

## Creating Screenshots

### Manual Screenshots

**1. Browser DevTools (Recommended)**
```bash
# Chrome/Edge DevTools
1. Open your PWA (F12)
2. Click device toolbar (Ctrl+Shift+M)
3. Select device/resolution
4. Click "Capture screenshot" (in DevTools menu)
5. Or use Cmd+Shift+P → "Capture screenshot"
```

**2. OS Screen Capture**
- **Windows**: Win + Shift + S
- **macOS**: Cmd + Shift + 4
- **Linux**: PrtScn or Shift + PrtScn

**3. Browser Extensions**
- Awesome Screenshot
- FireShot
- Full Page Screen Capture

### Automated Screenshots

**Puppeteer (Node.js)**
```javascript
const puppeteer = require('puppeteer');

async function captureScreenshots() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();

    // Desktop screenshot
    await page.setViewport({ width: 1280, height: 720 });
    await page.goto('https://your-app.com');
    await page.screenshot({
        path: 'screenshots/desktop-home.png',
        fullPage: false
    });

    // Mobile screenshot
    await page.setViewport({ width: 375, height: 667 });
    await page.goto('https://your-app.com');
    await page.screenshot({
        path: 'screenshots/mobile-home.png',
        fullPage: false
    });

    await browser.close();
}

captureScreenshots();
```

**Playwright**
```javascript
const { chromium } = require('playwright');

async function captureScreenshots() {
    const browser = await chromium.launch();
    const page = await browser.newPage();

    // Desktop
    await page.setViewportSize({ width: 1280, height: 720 });
    await page.goto('https://your-app.com');
    await page.screenshot({ path: 'screenshots/desktop.png' });

    // Mobile
    await page.setViewportSize({ width: 375, height: 667 });
    await page.goto('https://your-app.com');
    await page.screenshot({ path: 'screenshots/mobile.png' });

    await browser.close();
}
```

## Screenshot Requirements

### Recommended Sizes

**Mobile (Narrow)**
```yaml
screenshots:
    - src: "screenshots/mobile-1.png"
      sizes: [750, 1334]        # iPhone 8
      form_factor: "narrow"
    - src: "screenshots/mobile-2.png"
      sizes: [1080, 1920]       # Android
      form_factor: "narrow"
```

**Desktop/Tablet (Wide)**
```yaml
screenshots:
    - src: "screenshots/desktop-1.png"
      sizes: [1280, 720]        # HD
      form_factor: "wide"
    - src: "screenshots/desktop-2.png"
      sizes: [1920, 1080]       # Full HD
      form_factor: "wide"
```

### Platform-Specific

```yaml
screenshots:
    # Android
    - src: "screenshots/android-1.png"
      sizes: [1080, 1920]
      platform: "android"
      form_factor: "narrow"

    # iOS
    - src: "screenshots/ios-1.png"
      sizes: [1284, 2778]       # iPhone 13 Pro Max
      platform: "ios"
      form_factor: "narrow"

    # Windows
    - src: "screenshots/windows-1.png"
      sizes: [1920, 1080]
      platform: "windows"
      form_factor: "wide"
```

## Screenshot Guidelines

### Content Best Practices

1. **Show Key Features**: Highlight main functionality
2. **Use Real Data**: Show actual app content, not placeholders
3. **Keep UI Clean**: Remove debug info, test data
4. **Consistent Branding**: Match your app's design
5. **Clear Text**: Ensure text is readable

### Number of Screenshots

- **Minimum**: 1-2 screenshots
- **Recommended**: 3-5 screenshots
- **Maximum**: 8 screenshots (platform limit)

**Rationale**:
- Too few: Doesn't showcase app properly
- Too many: Overwhelming, slow loading
- Sweet spot: 3-5 showing main features

### Screenshot Sequence

Order screenshots logically:

```yaml
screenshots:
    - "screenshots/01-splash.png"      # Welcome/landing
    - "screenshots/02-main.png"        # Main interface
    - "screenshots/03-feature1.png"    # Key feature #1
    - "screenshots/04-feature2.png"    # Key feature #2
    - "screenshots/05-settings.png"    # Configuration/settings
```

## Design Guidelines

### Visual Quality

- **Resolution**: High-DPI (2x or 3x)
- **Format**: PNG (preferred) or JPEG
- **Compression**: Optimize without quality loss
- **Aspect Ratio**: Match target device

### Composition

- **Framing**: Show full interface or focused feature
- **Spacing**: Leave some breathing room
- **Highlights**: Use subtle overlays to emphasize features
- **Annotations**: Add brief captions if helpful

### Consistency

- **Same viewport**: All screenshots same dimensions per form factor
- **Same theme**: Light/dark mode consistent across all
- **Same state**: Logged in, data loaded, no errors
- **Same branding**: Colors, fonts match throughout

## File Organization

### Recommended Structure

```
assets/
└── screenshots/
    ├── mobile/
    │   ├── 01-home.png
    │   ├── 02-feature1.png
    │   ├── 03-feature2.png
    │   └── 04-settings.png
    ├── desktop/
    │   ├── 01-home.png
    │   ├── 02-dashboard.png
    │   └── 03-reports.png
    └── tablet/
        ├── 01-home.png
        └── 02-main.png
```

### Configuration Example

```yaml
pwa:
    manifest:
        screenshots:
            # Mobile screenshots
            - src: "screenshots/mobile/01-home.png"
              sizes: [750, 1334]
              form_factor: "narrow"
              platform: "android"
              label: "Home screen with recent activity"

            - src: "screenshots/mobile/02-feature1.png"
              sizes: [750, 1334]
              form_factor: "narrow"
              platform: "android"
              label: "Task management view"

            - src: "screenshots/mobile/03-feature2.png"
              sizes: [750, 1334]
              form_factor: "narrow"
              platform: "android"
              label: "Analytics dashboard"

            # Desktop screenshots
            - src: "screenshots/desktop/01-home.png"
              sizes: [1280, 720]
              form_factor: "wide"
              label: "Main dashboard interface"

            - src: "screenshots/desktop/02-dashboard.png"
              sizes: [1280, 720]
              form_factor: "wide"
              label: "Advanced analytics view"
```

## Optimizing Screenshots

### Image Optimization

**1. Use Image Compression Tools**
```bash
# Using ImageOptim (macOS)
imageoptim screenshots/*.png

# Using TinyPNG (CLI)
npx tinypng-cli screenshots/*.png

# Using Sharp (Node.js)
npx sharp-cli -i screenshot.png -o optimized.png
```

**2. Convert to WebP** (if supported)
```bash
# Using cwebp
for file in screenshots/*.png; do
    cwebp -q 80 "$file" -o "${file%.png}.webp"
done
```

**3. Serve Appropriate Format**
```yaml
screenshots:
    - src: "screenshots/home.webp"
      format: "image/webp"
      sizes: [1280, 720]
    - src: "screenshots/home.png"
      format: "image/png"
      sizes: [1280, 720]
```

### File Size Guidelines

- **Target**: < 500 KB per screenshot
- **Maximum**: < 1 MB per screenshot
- **Total**: All screenshots < 5 MB combined

## Platform-Specific Requirements

### Android / Google Play

**Requirements**:
- Minimum: 2 screenshots
- Format: PNG or JPEG
- Dimensions: 320-3840 pixels
- Aspect ratio: 16:9 or 9:16

```yaml
screenshots:
    - src: "screenshots/android-1.png"
      sizes: [1080, 1920]
      platform: "play"
      form_factor: "narrow"
```

### iOS / App Store

**Requirements**:
- Screenshots for each device size
- Format: PNG or JPEG
- No transparency
- Actual device screenshots preferred

```yaml
screenshots:
    - src: "screenshots/iphone-14-pro.png"
      sizes: [1179, 2556]
      platform: "ios"
      form_factor: "narrow"
```

### Windows / Microsoft Store

**Requirements**:
- Minimum: 1 screenshot
- Format: PNG, JPEG, or GIF
- Dimensions: 1366 x 768 or larger

```yaml
screenshots:
    - src: "screenshots/windows.png"
      sizes: [1920, 1080]
      platform: "windows"
      form_factor: "wide"
```

## Testing Screenshots

### 1. Validate in Manifest

```javascript
// Check screenshots in manifest
fetch('/manifest.json')
    .then(r => r.json())
    .then(manifest => {
        console.log('Screenshots:', manifest.screenshots);

        manifest.screenshots?.forEach((screenshot, index) => {
            console.log(`Screenshot ${index + 1}:`, {
                src: screenshot.src,
                sizes: screenshot.sizes,
                form_factor: screenshot.form_factor,
                platform: screenshot.platform
            });
        });
    });
```

### 2. Check in DevTools

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "Screenshots" section
4. Verify images load correctly
5. Check dimensions and labels
```

### 3. Test Install Flow

- **Desktop**: Install PWA, check screenshots in install dialog
- **Mobile**: Add to home screen, check screenshots
- **App Stores**: Submit to test store, verify display

### 4. Preview Tools

- **Chrome**: chrome://apps (after installation)
- **PWA Builder**: https://www.pwabuilder.com/
- **Web.dev**: Use PWA analyzer tools

## Common Issues

### Screenshots Not Showing

**Problem**: Screenshots don't appear in install prompt

**Solutions**:
- Verify screenshot paths are correct
- Check images are accessible (CORS)
- Ensure proper sizes specified
- Clear browser cache
- Check manifest is valid JSON

### Screenshots Too Large

**Problem**: Installation slow due to large screenshots

**Solutions**:
- Compress images (TinyPNG, ImageOptim)
- Reduce dimensions if too large
- Use WebP format
- Lazy load screenshots if possible

### Wrong Aspect Ratio

**Problem**: Screenshots appear stretched or cropped

**Solutions**:
- Match device aspect ratios
- Use form_factor correctly (narrow/wide)
- Test on actual devices
- Create platform-specific screenshots

### Poor Quality

**Problem**: Screenshots look blurry or pixelated

**Solutions**:
- Use high-resolution source
- Capture at 2x or 3x scale
- Use lossless compression
- Don't upscale small images

## Advanced Features

### Adding Labels

Provide context with descriptive labels:

```yaml
screenshots:
    - src: "screenshots/dashboard.png"
      sizes: [1280, 720]
      label: "Real-time analytics dashboard with customizable widgets"
      form_factor: "wide"

    - src: "screenshots/reports.png"
      sizes: [1280, 720]
      label: "Detailed reports with export functionality"
      form_factor: "wide"
```

### Multiple Platforms

Serve different screenshots per platform:

```yaml
screenshots:
    # Android-specific
    - src: "screenshots/android-home.png"
      platform: "android"
      form_factor: "narrow"

    # iOS-specific
    - src: "screenshots/ios-home.png"
      platform: "ios"
      form_factor: "narrow"

    # Desktop-specific
    - src: "screenshots/desktop-home.png"
      platform: "windows"
      form_factor: "wide"
```

## Migration from Deprecated Command

### Old Approach (Deprecated)

```bash
# This no longer works
php bin/console pwa:create:screenshot
```

### New Approach

1. **Capture screenshots** using browser DevTools or automation
2. **Optimize images** using compression tools
3. **Place in assets** directory: `assets/screenshots/`
4. **Configure** in `pwa.yaml`:

```yaml
pwa:
    manifest:
        screenshots:
            - src: "screenshots/home.png"
              sizes: [1280, 720]
```

5. **Compile assets**:
```bash
php bin/console asset-map:compile
```

## Best Practices Checklist

Before publishing:

- [ ] Captured 3-5 key screenshots
- [ ] Used high resolution (2x or 3x)
- [ ] Optimized file sizes (< 500 KB each)
- [ ] Added descriptive labels
- [ ] Specified correct form_factor
- [ ] Created both narrow and wide versions
- [ ] Tested on actual devices
- [ ] Verified in install prompt
- [ ] Checked all images load
- [ ] Validated manifest JSON

## Related Documentation

- [Screenshots (Manifest)](../the-manifest/screenshots.md) - Complete configuration
- [Icons](icons.md) - Icon management
- [AssetMapper](https://symfony.com/doc/current/frontend/asset_mapper.html) - Asset management

## Resources

- **PWA Builder**: https://www.pwabuilder.com/
- **TinyPNG**: https://tinypng.com/
- **ImageOptim**: https://imageoptim.com/
- **MDN Screenshots**: https://developer.mozilla.org/en-US/docs/Web/Manifest/screenshots
- **Web.dev**: https://web.dev/add-manifest/#screenshots

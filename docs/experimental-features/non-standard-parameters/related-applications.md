# Related applications

## Overview

The `related_applications` property allows you to specify native applications available on different platforms (App Store, Play Store, Microsoft Store, etc.). Combined with `prefer_related_applications`, browsers can suggest installing the native app instead of or alongside the PWA.

**Use Cases**:
- You have both a PWA and native mobile apps
- You want to promote the native app for better platform integration
- You want users to discover all versions of your app
- You need platform-specific features only available in native apps

## Configuration

### Basic related applications

List related native apps without preferring them:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        prefer_related_applications: false  # Don't prefer native apps
        related_applications:
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.example.app"
              id: "com.example.app"
```

**Result**: Browser shows PWA install prompt. Native apps listed as alternatives.

### Prefer native applications

Suggest native apps over PWA installation:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        prefer_related_applications: true  # Prefer native apps
        related_applications:
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.example.app"
              id: "com.example.app"
            - platform: "itunes"
              url: "https://apps.apple.com/app/example-app/id123456789"
```

**Result**: Browser suggests installing native app first, PWA as fallback.

### Multiple platforms

List apps for all major platforms:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        prefer_related_applications: false
        related_applications:
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.example.app"
              id: "com.example.app"
            - platform: "itunes"
              url: "https://apps.apple.com/app/example-app/id123456789"
              id: "123456789"
            - platform: "windows"
              url: "https://apps.microsoft.com/store/detail/example-app/9NBLGGH4NNS1"
              id: "9NBLGGH4NNS1"
            - platform: "chrome_web_store"
              url: "https://chrome.google.com/webstore/detail/example-app/abcdefghijklmnopqrstu"
              id: "abcdefghijklmnopqrstu"
```

## Properties

### prefer_related_applications

**Type**: `boolean`
**Default**: `false`

Controls whether browsers should suggest native apps over the PWA.

| Value | Behavior |
|-------|----------|
| `false` | PWA is primary, native apps as alternatives |
| `true` | Native apps preferred, PWA as fallback |

**When to use `true`**:
- Native app provides significantly better experience
- Platform-specific features are critical
- Native app is your primary product
- PWA is mainly for web compatibility

**When to use `false`** (default):
- PWA is your primary product
- Want maximum reach across all platforms
- Native app is optional enhancement
- PWA feature parity with native apps

### platform

**Type**: `string` (required)

Identifier for the app platform.

## Supported platforms

### play (Google play store)

**For**: Android devices

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "play"
      url: "https://play.google.com/store/apps/details?id=com.example.myapp"
      id: "com.example.myapp"
```

**URL Format**: `https://play.google.com/store/apps/details?id={package_name}`

**ID Format**: Java package name (e.g., `com.company.app`)

**Finding your package name**:
1. Open Play Console
2. Go to your app
3. Check "Package name" field
4. Or find in `build.gradle`: `applicationId "com.example.app"`

### itunes (Apple app store)

**For**: iOS, iPadOS devices

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "itunes"
      url: "https://apps.apple.com/app/example-app/id123456789"
      id: "123456789"
```

**URL Format**: `https://apps.apple.com/app/{app-name}/id{app_id}`

**ID Format**: Numeric App ID (e.g., `123456789`)

**Finding your App ID**:
1. Open App Store Connect
2. Go to your app
3. Check "Apple ID" field
4. Or find in App Store URL after `/id`

### windows (Microsoft store)

**For**: Windows 10/11 devices

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "windows"
      url: "https://apps.microsoft.com/store/detail/9NBLGGH4NNS1"
      id: "9NBLGGH4NNS1"
```

**URL Format**: `https://apps.microsoft.com/store/detail/{store_id}`

**ID Format**: Store ID starting with 9 (e.g., `9NBLGGH4NNS1`)

**Finding your Store ID**:
1. Open Partner Center
2. Go to your app
3. Check "Store ID" field
4. Or find in Microsoft Store URL

### chrome_web_store (Chrome web store)

**For**: Chrome Extensions/Apps

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "chrome_web_store"
      url: "https://chrome.google.com/webstore/detail/abcdefghijklmnopqrst"
      id: "abcdefghijklmnopqrst"
```

**URL Format**: `https://chrome.google.com/webstore/detail/{extension_id}`

**ID Format**: 32-character extension ID

**Finding your Extension ID**:
1. Open Chrome Web Store Developer Dashboard
2. Go to your extension
3. Check "Item ID" field
4. Or find in extension URL

### chromeos_play (Chrome OS play store)

**For**: Chrome OS devices with Play Store support

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "chromeos_play"
      url: "https://play.google.com/store/apps/details?id=com.example.app"
      id: "com.example.app"
```

**Format**: Same as Google Play Store

### webapp (web apps)

**For**: Generic web applications

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "webapp"
      url: "https://app.example.com/install"
```

**URL Format**: Any URL to your web app

### f-droid (f-droid)

**For**: Android open-source app repository

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "f-droid"
      url: "https://f-droid.org/packages/com.example.app"
      id: "com.example.app"
```

**URL Format**: `https://f-droid.org/packages/{package_name}`

### amazon (amazon appstore)

**For**: Amazon Fire devices, Android

```yaml title="/config/packages/pwa.yaml" linenums="1"
related_applications:
    - platform: "amazon"
      url: "https://www.amazon.com/dp/B0123456789"
      id: "B0123456789"
```

**URL Format**: `https://www.amazon.com/dp/{asin}`

**ID Format**: ASIN (Amazon Standard Identification Number)

## Complete examples

### Example 1: Social media app

PWA primary, native apps as enhancements:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "SocialApp"
        description: "Connect with friends"
        prefer_related_applications: false  # PWA is primary
        related_applications:
            # Android native app
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.socialapp"
              id: "com.socialapp"

            # iOS native app
            - platform: "itunes"
              url: "https://apps.apple.com/app/socialapp/id987654321"
              id: "987654321"
```

### Example 2: Game

Native apps preferred for performance:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Epic Game"
        description: "Play the best game ever"
        prefer_related_applications: true  # Prefer native for performance
        related_applications:
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.epic.game"
              id: "com.epic.game"

            - platform: "itunes"
              url: "https://apps.apple.com/app/epic-game/id111222333"
              id: "111222333"

            - platform: "windows"
              url: "https://apps.microsoft.com/store/detail/9NBLGGH4GAME"
              id: "9NBLGGH4GAME"
```

### Example 3: Productivity tool

Available on all platforms:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "TaskMaster Pro"
        prefer_related_applications: false
        related_applications:
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.taskmaster.pro"
              id: "com.taskmaster.pro"

            - platform: "itunes"
              url: "https://apps.apple.com/app/taskmaster-pro/id555666777"
              id: "555666777"

            - platform: "windows"
              url: "https://apps.microsoft.com/store/detail/9TASKMASTER123"
              id: "9TASKMASTER123"

            - platform: "chrome_web_store"
              url: "https://chrome.google.com/webstore/detail/taskmaster/abcdefghijkl"
              id: "abcdefghijkl"

            - platform: "f-droid"
              url: "https://f-droid.org/packages/com.taskmaster.pro"
              id: "com.taskmaster.pro"
```

### Example 4: PWA-only with web app link

No native apps, just PWA:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        name: "Web-Only App"
        prefer_related_applications: false
        related_applications:
            - platform: "webapp"
              url: "https://webapp.example.com"
```

## Browser support

| Browser | Support | Behavior |
|---------|---------|----------|
| Chrome (Android) | ✅ | May show native app banner if prefer=true |
| Chrome (Desktop) | ⚠️ | Limited support |
| Edge | ⚠️ | Limited support |
| Safari | ❌ | Ignored |
| Firefox | ❌ | Ignored |

!!! info

    **Graceful Degradation**: Browsers that don't support this feature simply ignore it and show standard PWA install prompts.

## User experience flow

### When prefer_related_applications = false

```
User visits PWA
    ↓
Browser shows: "Add to Home Screen" (PWA)
    ↓
User installs PWA
    ↓
[Optional] "Also available on Google Play" link shown
```

### When prefer_related_applications = true

```
User visits PWA (on Android)
    ↓
Browser checks: Native app available on Play Store?
    ↓
Yes: Shows "Get the app" banner → Play Store
    ↓
No: Falls back to "Add to Home Screen" (PWA)
```

## Testing

### 1. Check manifest

```javascript title="JavaScript Console" linenums="1"
fetch('/manifest.json')
    .then(r => r.json())
    .then(manifest => {
        console.log('Prefer native:', manifest.prefer_related_applications);
        console.log('Related apps:', manifest.related_applications);

        manifest.related_applications?.forEach(app => {
            console.log(`Platform: ${app.platform}`);
            console.log(`URL: ${app.url}`);
            console.log(`ID: ${app.id}`);
        });
    });
```

### 2. Test on Android Chrome

```bash
1. Visit your PWA on Android Chrome
2. If prefer_related_applications = true:
   - Look for native app install banner
   - Banner should link to Play Store
3. If prefer_related_applications = false:
   - Look for PWA install prompt
   - Native app may be mentioned as alternative
```

### 3. Verify store links

```bash
# Test each URL
- ✓ Play Store URL opens correct app
- ✓ App Store URL opens correct app
- ✓ Microsoft Store URL opens correct app
- ✓ Chrome Web Store URL opens correct extension
- ✓ All IDs match actual apps
```

### 4. DevTools inspection

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "Related Applications" section
4. Verify:
   - prefer_related_applications value
   - All platforms listed
   - URLs are correct
   - IDs match
```

## Best practices

### 1. Keep URLs updated

```yaml
# ✓ Good - URLs point to active apps
related_applications:
    - platform: "play"
      url: "https://play.google.com/store/apps/details?id=com.active.app"

# ✗ Bad - URL points to removed app
related_applications:
    - platform: "play"
      url: "https://play.google.com/store/apps/details?id=com.old.removed.app"
```

### 2. Include app IDs

```yaml
# ✓ Good - includes ID for better matching
- platform: "play"
  url: "https://play.google.com/store/apps/details?id=com.example.app"
  id: "com.example.app"

# ⚠️ Acceptable but less ideal
- platform: "play"
  url: "https://play.google.com/store/apps/details?id=com.example.app"
```

### 3. List active platforms only

```yaml
# ✓ Good - only lists platforms where app exists
related_applications:
    - platform: "play"
      url: "..."

# ✗ Bad - lists platforms without apps
related_applications:
    - platform: "play"
      url: "..."
    - platform: "itunes"
      url: "..."  # But iOS app doesn't exist!
```

### 4. Use prefer_related_applications sparingly

```yaml
# Default (recommended for most PWAs)
prefer_related_applications: false

# Only use true if native app is significantly better
prefer_related_applications: true
```

### 5. Provide fallback

Always ensure your PWA works standalone, even when preferring native apps.

## Decision tree

**Should you prefer native applications?**

```
Does your native app provide significantly better UX?
├─ Yes → Does it have features PWA can't offer?
│         ├─ Yes → prefer_related_applications: true
│         └─ No → prefer_related_applications: false
└─ No → prefer_related_applications: false

Do you want maximum reach?
└─ Yes → prefer_related_applications: false (PWA primary)

Is installation friction a concern?
└─ Yes → prefer_related_applications: false (PWA easier to install)
```

## Common mistakes

### 1. Wrong ID format

**Problem**:
```yaml
- platform: "play"
  id: "https://play.google.com/..."  # Wrong! ID is not a URL
```

**Solution**:
```yaml
- platform: "play"
  id: "com.example.app"  # Correct: package name
```

### 2. Missing platform

**Problem**:
```yaml
related_applications:
    - url: "https://play.google.com/..."  # Missing platform!
```

**Solution**:
```yaml
related_applications:
    - platform: "play"  # Required!
      url: "https://play.google.com/..."
```

### 3. Prefer without related apps

**Problem**:
```yaml
prefer_related_applications: true
# But no related_applications listed!
```

**Solution**:
```yaml
prefer_related_applications: true
related_applications:
    - platform: "play"
      url: "..."  # Must provide apps to prefer
```

### 4. Outdated URLs

**Problem**: URLs point to removed or renamed apps

**Solution**: Regularly verify all store URLs still work

## Troubleshooting

### Native app banner not showing

**Problem**: Expected native app banner doesn't appear

**Checklist**:
- ✓ `prefer_related_applications: true` is set
- ✓ Testing on supported browser (Chrome Android)
- ✓ App actually exists on specified platform
- ✓ URL and ID are correct
- ✓ App is published (not draft)
- ✓ PWA manifest is valid

### Wrong app suggested

**Problem**: Browser suggests wrong native app

**Solutions**:
- Verify `id` matches the actual app
- Check `platform` is correct
- Ensure URL points to correct store listing
- Clear browser cache and try again

### PWA install blocked

**Problem**: Users can't install PWA when prefer=true

**Solutions**:
- Set `prefer_related_applications: false`
- Or ensure native app is available
- Provide manual "Install PWA" link in your app

## Related documentation

- [PWA Manifest](../../the-manifest/complete-example.md) - Complete manifest configuration
- [Installation](../../how-to-install-remove-a-pwa.md) - PWA installation guide

## Resources

- **MDN related_applications**: https://developer.mozilla.org/en-US/docs/Web/Manifest/related_applications
- **Web.dev**: https://web.dev/add-manifest/#related_applications
- **W3C Spec**: https://w3c.github.io/manifest/#related_applications-member

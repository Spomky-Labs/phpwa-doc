# Related Applications

## Overview

The `related_applications` property allows you to specify native applications available on different platforms (App Store, Play Store, Microsoft Store, etc.). Combined with `prefer_related_applications`, browsers can suggest installing the native app instead of or alongside the PWA.

**Use Cases**:
- You have both a PWA and native mobile apps
- You want to promote the native app for better platform integration
- You want users to discover all versions of your app
- You need platform-specific features only available in native apps

## Configuration

### Basic Related Applications

List related native apps without preferring them:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        prefer_related_applications: false  # Don't prefer native apps
        related_applications:
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.example.app"
              id: "com.example.app"
```
{% endcode %}

**Result**: Browser shows PWA install prompt. Native apps listed as alternatives.

### Prefer Native Applications

Suggest native apps over PWA installation:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
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
{% endcode %}

**Result**: Browser suggests installing native app first, PWA as fallback.

### Multiple Platforms

List apps for all major platforms:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
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
{% endcode %}

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

## Supported Platforms

### play (Google Play Store)

**For**: Android devices

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "play"
      url: "https://play.google.com/store/apps/details?id=com.example.myapp"
      id: "com.example.myapp"
```
{% endcode %}

**URL Format**: `https://play.google.com/store/apps/details?id={package_name}`

**ID Format**: Java package name (e.g., `com.company.app`)

**Finding your package name**:
1. Open Play Console
2. Go to your app
3. Check "Package name" field
4. Or find in `build.gradle`: `applicationId "com.example.app"`

### itunes (Apple App Store)

**For**: iOS, iPadOS devices

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "itunes"
      url: "https://apps.apple.com/app/example-app/id123456789"
      id: "123456789"
```
{% endcode %}

**URL Format**: `https://apps.apple.com/app/{app-name}/id{app_id}`

**ID Format**: Numeric App ID (e.g., `123456789`)

**Finding your App ID**:
1. Open App Store Connect
2. Go to your app
3. Check "Apple ID" field
4. Or find in App Store URL after `/id`

### windows (Microsoft Store)

**For**: Windows 10/11 devices

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "windows"
      url: "https://apps.microsoft.com/store/detail/9NBLGGH4NNS1"
      id: "9NBLGGH4NNS1"
```
{% endcode %}

**URL Format**: `https://apps.microsoft.com/store/detail/{store_id}`

**ID Format**: Store ID starting with 9 (e.g., `9NBLGGH4NNS1`)

**Finding your Store ID**:
1. Open Partner Center
2. Go to your app
3. Check "Store ID" field
4. Or find in Microsoft Store URL

### chrome_web_store (Chrome Web Store)

**For**: Chrome Extensions/Apps

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "chrome_web_store"
      url: "https://chrome.google.com/webstore/detail/abcdefghijklmnopqrst"
      id: "abcdefghijklmnopqrst"
```
{% endcode %}

**URL Format**: `https://chrome.google.com/webstore/detail/{extension_id}`

**ID Format**: 32-character extension ID

**Finding your Extension ID**:
1. Open Chrome Web Store Developer Dashboard
2. Go to your extension
3. Check "Item ID" field
4. Or find in extension URL

### chromeos_play (Chrome OS Play Store)

**For**: Chrome OS devices with Play Store support

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "chromeos_play"
      url: "https://play.google.com/store/apps/details?id=com.example.app"
      id: "com.example.app"
```
{% endcode %}

**Format**: Same as Google Play Store

### webapp (Web Apps)

**For**: Generic web applications

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "webapp"
      url: "https://app.example.com/install"
```
{% endcode %}

**URL Format**: Any URL to your web app

### f-droid (F-Droid)

**For**: Android open-source app repository

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "f-droid"
      url: "https://f-droid.org/packages/com.example.app"
      id: "com.example.app"
```
{% endcode %}

**URL Format**: `https://f-droid.org/packages/{package_name}`

### amazon (Amazon Appstore)

**For**: Amazon Fire devices, Android

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
related_applications:
    - platform: "amazon"
      url: "https://www.amazon.com/dp/B0123456789"
      id: "B0123456789"
```
{% endcode %}

**URL Format**: `https://www.amazon.com/dp/{asin}`

**ID Format**: ASIN (Amazon Standard Identification Number)

## Complete Examples

### Example 1: Social Media App

PWA primary, native apps as enhancements:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
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
{% endcode %}

### Example 2: Game

Native apps preferred for performance:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
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
{% endcode %}

### Example 3: Productivity Tool

Available on all platforms:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
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
{% endcode %}

### Example 4: PWA-Only with Web App Link

No native apps, just PWA:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Web-Only App"
        prefer_related_applications: false
        related_applications:
            - platform: "webapp"
              url: "https://webapp.example.com"
```
{% endcode %}

## Browser Support

| Browser | Support | Behavior |
|---------|---------|----------|
| Chrome (Android) | ✅ | May show native app banner if prefer=true |
| Chrome (Desktop) | ⚠️ | Limited support |
| Edge | ⚠️ | Limited support |
| Safari | ❌ | Ignored |
| Firefox | ❌ | Ignored |

{% hint style="info" %}
**Graceful Degradation**: Browsers that don't support this feature simply ignore it and show standard PWA install prompts.
{% endhint %}

## User Experience Flow

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

### 1. Check Manifest

{% code title="JavaScript Console" lineNumbers="true" %}
```javascript
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
{% endcode %}

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

### 3. Verify Store Links

```bash
# Test each URL
- ✓ Play Store URL opens correct app
- ✓ App Store URL opens correct app
- ✓ Microsoft Store URL opens correct app
- ✓ Chrome Web Store URL opens correct extension
- ✓ All IDs match actual apps
```

### 4. DevTools Inspection

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

## Best Practices

### 1. Keep URLs Updated

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

### 2. Include App IDs

```yaml
# ✓ Good - includes ID for better matching
- platform: "play"
  url: "https://play.google.com/store/apps/details?id=com.example.app"
  id: "com.example.app"

# ⚠️ Acceptable but less ideal
- platform: "play"
  url: "https://play.google.com/store/apps/details?id=com.example.app"
```

### 3. List Active Platforms Only

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

### 4. Use prefer_related_applications Sparingly

```yaml
# Default (recommended for most PWAs)
prefer_related_applications: false

# Only use true if native app is significantly better
prefer_related_applications: true
```

### 5. Provide Fallback

Always ensure your PWA works standalone, even when preferring native apps.

## Decision Tree

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

## Common Mistakes

### 1. Wrong ID Format

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

### 2. Missing Platform

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

### 3. Prefer Without Related Apps

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

### Native App Banner Not Showing

**Problem**: Expected native app banner doesn't appear

**Checklist**:
- ✓ `prefer_related_applications: true` is set
- ✓ Testing on supported browser (Chrome Android)
- ✓ App actually exists on specified platform
- ✓ URL and ID are correct
- ✓ App is published (not draft)
- ✓ PWA manifest is valid

### Wrong App Suggested

**Problem**: Browser suggests wrong native app

**Solutions**:
- Verify `id` matches the actual app
- Check `platform` is correct
- Ensure URL points to correct store listing
- Clear browser cache and try again

### PWA Install Blocked

**Problem**: Users can't install PWA when prefer=true

**Solutions**:
- Set `prefer_related_applications: false`
- Or ensure native app is available
- Provide manual "Install PWA" link in your app

## Related Documentation

- [PWA Manifest](../../the-manifest/complete-example.md) - Complete manifest configuration
- [Installation](../../how-to-install-remove-a-pwa.md) - PWA installation guide

## Resources

- **MDN related_applications**: https://developer.mozilla.org/en-US/docs/Web/Manifest/related_applications
- **Web.dev**: https://web.dev/add-manifest/#related_applications
- **W3C Spec**: https://w3c.github.io/manifest/#related_applications-member

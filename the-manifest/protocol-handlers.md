# Protocol Handlers

Progressive Web Apps (PWAs) have the ability to handle web protocols, which means they can respond to specific URL schemes. For example, a PWA can register to handle `mailto:` links, so that clicking on an email link can open a compose window in the PWA instead of opening the default mail client.

With protocol handlers, your PWA can provide a more integrated user experience, functioning more like a native application by becoming the default handler for specific protocols on the user's device.

## Overview

Protocol handlers allow your PWA to:
- **Handle standard protocols**: `mailto`, `tel`, `sms`, etc.
- **Register custom protocols**: Using the `web+` prefix
- **Intercept protocol links**: Capture clicks on protocol URLs
- **Provide integrated experiences**: Open your PWA instead of native apps
- **Deepen user engagement**: Make your PWA the default handler

**Real-world examples**:
- Email clients handling `mailto:` links
- Messaging apps handling `sms:` or `tel:` links
- Music apps handling `web+music:` links
- Recipe apps handling `web+recipe:` links
- Game launchers handling `web+game:` links

## Browser Support

| Browser | Support | Notes |
|---------|---------|-------|
| Chrome (Desktop) | ✅ Full | Since Chrome 96 |
| Chrome (Android) | ✅ Full | Since Chrome 96 |
| Edge (Desktop) | ✅ Full | Since Edge 96 |
| Safari | ❌ Not supported | - |
| Firefox | ❌ Not supported | - |

{% hint style="warning" %}
**Limited Browser Support**: Protocol handlers are currently only supported in Chromium-based browsers (Chrome, Edge). Always provide fallback experiences.
{% endhint %}

## Configuration

### Basic Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        protocol_handlers:
            - protocol: "mailto"
              url: "/compose?to=%s"
            - protocol: "web+custom"
              url:
                  path: "app_feature1"
                  params:
                      foo: "bar"
```
{% endcode %}

### Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "My Email App"
        protocol_handlers:
            # Standard mailto protocol
            - protocol: "mailto"
              url: "/mail/compose?to=%s"

            # Custom recipe protocol with route
            - protocol: "web+recipe"
              url:
                  path: "app_recipe_view"
                  params:
                      recipe_id: "%s"
                      utm_source: "protocol_handler"

            # Custom music protocol with placeholder
            - protocol: "web+music"
              placeholder: "track"
              url: "/player?track=%s"
```
{% endcode %}

## Protocol Parameter

The `protocol` parameter specifies the protocol scheme your PWA can handle. It must follow these rules:

### Standard Protocols

Standard web protocols can be registered without any prefix:

**`mailto` - Email addresses**
```yaml
protocol_handlers:
    - protocol: "mailto"
      url: "/compose?to=%s"
```

**Usage**: `<a href="mailto:user@example.com">Email Me</a>`

**`tel` - Telephone numbers**
```yaml
protocol_handlers:
    - protocol: "tel"
      url: "/call?number=%s"
```

**Usage**: `<a href="tel:+1234567890">Call Us</a>`

**`sms` - SMS messages**
```yaml
protocol_handlers:
    - protocol: "sms"
      url: "/message?to=%s"
```

**Usage**: `<a href="sms:+1234567890">Text Us</a>`

### Custom Protocols

Custom protocols **must** start with the `web+` prefix:

**Valid custom protocols**:
```yaml
protocol_handlers:
    - protocol: "web+jngl"      # ✓ Valid
    - protocol: "web+music"     # ✓ Valid
    - protocol: "web+recipe"    # ✓ Valid
    - protocol: "web+game"      # ✓ Valid
```

**Invalid custom protocols**:
```yaml
protocol_handlers:
    - protocol: "jngl"          # ✗ Invalid - missing web+ prefix
    - protocol: "custom"        # ✗ Invalid - missing web+ prefix
    - protocol: "myapp"         # ✗ Invalid - missing web+ prefix
```

{% hint style="warning" %}
**Custom Protocol Rules**: Custom protocols MUST use the `web+` prefix to be valid. Protocols without this prefix are reserved for standard web protocols.
{% endhint %}

## URL Parameter

The `url` parameter defines where to redirect when the protocol is triggered. The URL can include a placeholder `%s` that will be replaced with the protocol's argument.

### Simple String Format

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
protocol_handlers:
    - protocol: "mailto"
      url: "/mail/compose?to=%s"
```
{% endcode %}

**How it works**:
- User clicks: `mailto:user@example.com`
- PWA opens: `/mail/compose?to=user@example.com`

### Advanced Object Format

Just like with [shortcuts](shortcuts.md#url-parameter) and [start_url](application-information/start-url.md), you can use the full URL object configuration:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
protocol_handlers:
    - protocol: "web+recipe"
      url:
          path: "app_recipe_view"       # Symfony route name
          params:
              recipe_id: "%s"            # Protocol argument
              utm_source: "protocol_handler"
              utm_medium: "web+recipe"
          path_type_reference: 1         # Absolute path
```
{% endcode %}

**Corresponding Symfony route**:
```php
#[Route('/recipe/{recipe_id}', name: 'app_recipe_view')]
public function viewRecipe(string $recipe_id): Response
{
    // recipe_id will contain the protocol argument
    // Also receives utm_source and utm_medium parameters
}
```

### Path Type Reference

The `path_type_reference` option controls URL generation:

| Value | Type | Example Output |
|-------|------|----------------|
| `0` | Absolute URL | `https://app.com/foo/bar` |
| `1` | Absolute path (default) | `/foo/bar` |
| `2` | Relative path | `../bar` |
| `3` | Network path | `//app.com/foo/bar` |

**Example**:
```yaml
protocol_handlers:
    - protocol: "web+app"
      url:
          path: "app_feature"
          path_type_reference: 0  # Generates full URL with domain
```

## Placeholder Parameter

When you need more control over the placeholder format, you can use the `placeholder` parameter. The bundle will automatically format it as `placeholder=%s`:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
protocol_handlers:
    - protocol: "web+jngl"
      placeholder: "type"
      url: "/handle?type=%s"
```
{% endcode %}

**How it works**:
- User clicks: `web+jngl:example`
- PWA opens: `/handle?type=example`

**Without placeholder** (default):
- User clicks: `web+jngl:example`
- PWA opens: `/handle?example` (not ideal)

This allows you to create more semantic URL patterns for your protocol handlers.

## Use Cases

### 1. Email Client

Handle `mailto:` links to compose emails in your PWA:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "WebMail Pro"
        protocol_handlers:
            - protocol: "mailto"
              url: "/compose?to=%s"
```
{% endcode %}

{% code title="src/Controller/EmailController.php" lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class EmailController extends AbstractController
{
    #[Route('/compose', name: 'app_compose')]
    public function compose(Request $request): Response
    {
        $to = $request->query->get('to', '');

        return $this->render('email/compose.html.twig', [
            'to' => $to,
        ]);
    }
}
```
{% endcode %}

### 2. Phone Dialer

Handle `tel:` links for a calling app:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "WebPhone"
        protocol_handlers:
            - protocol: "tel"
              placeholder: "number"
              url: "/call?number=%s"
```
{% endcode %}

{% code title="src/Controller/PhoneController.php" lineNumbers="true" %}
```php
#[Route('/call', name: 'app_call')]
public function call(Request $request): Response
{
    $number = $request->query->get('number', '');

    // Clean and validate phone number
    $cleanNumber = preg_replace('/[^0-9+]/', '', $number);

    return $this->render('phone/call.html.twig', [
        'number' => $cleanNumber,
    ]);
}
```
{% endcode %}

### 3. Recipe App

Handle custom `web+recipe:` protocol:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "RecipeBook"
        protocol_handlers:
            - protocol: "web+recipe"
              url:
                  path: "app_recipe_view"
                  params:
                      id: "%s"
```
{% endcode %}

{% code title="src/Controller/RecipeController.php" lineNumbers="true" %}
```php
#[Route('/recipe/{id}', name: 'app_recipe_view')]
public function view(string $id, RecipeRepository $repository): Response
{
    $recipe = $repository->find($id);

    if (!$recipe) {
        throw $this->createNotFoundException('Recipe not found');
    }

    return $this->render('recipe/view.html.twig', [
        'recipe' => $recipe,
    ]);
}
```
{% endcode %}

**Sharing recipes**:
```html
<a href="web+recipe:chocolate-chip-cookies">
    Open in RecipeBook
</a>
```

### 4. Music Player

Handle `web+music:` protocol for streaming:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "MusicStream"
        protocol_handlers:
            - protocol: "web+music"
              placeholder: "track"
              url: "/player?track=%s"
```
{% endcode %}

{% code title="src/Controller/MusicController.php" lineNumbers="true" %}
```php
#[Route('/player', name: 'app_music_player')]
public function player(Request $request): Response
{
    $trackId = $request->query->get('track', '');

    // Validate and load track
    $track = $this->getTrackById($trackId);

    return $this->render('music/player.html.twig', [
        'track' => $track,
        'autoplay' => true,
    ]);
}
```
{% endcode %}

### 5. Multi-Protocol App

Support multiple protocols in one PWA:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        name: "Communication Hub"
        protocol_handlers:
            # Email
            - protocol: "mailto"
              url: "/mail/compose?to=%s"

            # Phone
            - protocol: "tel"
              url: "/phone/call?number=%s"

            # SMS
            - protocol: "sms"
              url: "/sms/compose?to=%s"

            # Video call
            - protocol: "web+videocall"
              placeholder: "room"
              url: "/video/join?room=%s"
```
{% endcode %}

## Registration Process

### 1. User Installs PWA

When a user installs your PWA, the browser detects protocol handlers in the manifest.

### 2. Permission Request

The browser prompts the user to allow the PWA to handle specified protocols.

**Chrome prompt example**:
```
Allow "My Email App" to open mailto links?
[Allow] [Block]
```

### 3. Default Handler

If the user allows, your PWA becomes an available handler for those protocols.

**User can**:
- Set your PWA as the default handler
- Choose on a case-by-case basis
- Revoke permissions later

## Testing

### 1. Test Protocol Links

Create test HTML page with protocol links:

{% code title="public/test-protocols.html" %}
```html
<!DOCTYPE html>
<html>
<head>
    <title>Protocol Handler Test</title>
</head>
<body>
    <h1>Test Protocol Handlers</h1>

    <h2>Standard Protocols</h2>
    <ul>
        <li><a href="mailto:test@example.com">Test mailto</a></li>
        <li><a href="tel:+1234567890">Test tel</a></li>
        <li><a href="sms:+1234567890">Test sms</a></li>
    </ul>

    <h2>Custom Protocols</h2>
    <ul>
        <li><a href="web+recipe:123">Test web+recipe</a></li>
        <li><a href="web+music:track-456">Test web+music</a></li>
        <li><a href="web+game:level-7">Test web+game</a></li>
    </ul>
</body>
</html>
```
{% endcode %}

### 2. Check Manifest in DevTools

```bash
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "Protocol Handlers" section
4. Verify all protocols are listed
5. Verify URLs are correct
```

### 3. Test Registration Flow

```bash
1. Install PWA (if not installed)
2. Click a protocol link
3. Observe permission prompt
4. Allow protocol handling
5. Verify PWA opens with correct URL
6. Check query parameters are populated
```

### 4. JavaScript Testing

{% code title="public/test-protocol.js" %}
```javascript
// Check if protocol handler is registered
function checkProtocolHandler(protocol) {
    if ('getInstalledRelatedApps' in navigator) {
        navigator.getInstalledRelatedApps()
            .then(apps => {
                console.log('Installed apps:', apps);
            });
    }
}

// Test protocol link
function testProtocol(protocol, value) {
    const link = `${protocol}:${value}`;
    console.log('Testing protocol:', link);
    window.location.href = link;
}

// Examples
testProtocol('mailto', 'user@example.com');
testProtocol('web+recipe', '123');
```
{% endcode %}

## User Management

### Checking Permissions

Users can manage protocol handlers in browser settings:

**Chrome**:
```
chrome://settings/handlers
```

**Edge**:
```
edge://settings/content/defaultProtocolHandlers
```

### Revoking Permission

Users can remove your PWA as a protocol handler:
1. Open browser settings
2. Go to Protocol handlers / Default apps
3. Find your PWA
4. Click "Remove" or "Block"

## Best Practices

### 1. Request Only Needed Protocols

```yaml
# ✓ Good - only essential protocols
protocol_handlers:
    - protocol: "mailto"
      url: "/compose?to=%s"

# ✗ Bad - too many protocols
protocol_handlers:
    - protocol: "mailto"
    - protocol: "tel"
    - protocol: "sms"
    - protocol: "web+custom1"
    - protocol: "web+custom2"
    # ... (requesting too many)
```

### 2. Provide Clear Value

Explain why users should allow protocol handling:

```html
<div class="protocol-prompt">
    <h2>Make [App Name] your default email client</h2>
    <p>Click "Allow" to open email links in [App Name] instead of your default mail client.</p>
    <button onclick="testMailto()">Try it now</button>
</div>
```

### 3. Handle Missing Parameters

```php
#[Route('/compose', name: 'app_compose')]
public function compose(Request $request): Response
{
    $to = $request->query->get('to');

    // Handle case where 'to' is missing
    if (!$to) {
        return $this->render('email/compose.html.twig', [
            'to' => '',
            'error' => 'No recipient specified',
        ]);
    }

    // Validate email
    if (!filter_var($to, FILTER_VALIDATE_EMAIL)) {
        return $this->render('email/compose.html.twig', [
            'to' => $to,
            'error' => 'Invalid email address',
        ]);
    }

    return $this->render('email/compose.html.twig', [
        'to' => $to,
    ]);
}
```

### 4. Sanitize Protocol Input

```php
#[Route('/call', name: 'app_call')]
public function call(Request $request): Response
{
    $number = $request->query->get('number', '');

    // Sanitize phone number
    $cleanNumber = preg_replace('/[^0-9+\-() ]/', '', $number);

    // Validate format
    if (!preg_match('/^[\d\s()+\-]+$/', $cleanNumber)) {
        throw $this->createNotFoundException('Invalid phone number');
    }

    return $this->render('phone/call.html.twig', [
        'number' => $cleanNumber,
    ]);
}
```

### 5. Provide Fallback

Always ensure your app works even if protocol handlers aren't available:

```javascript
// Feature detection
if ('launchQueue' in window) {
    // Protocol handlers supported
    console.log('Protocol handlers available');
} else {
    // Provide alternative navigation
    console.log('Protocol handlers not supported');
}
```

### 6. Test Across Browsers

```bash
# Test matrix
- ✓ Chrome (Desktop) - Full support
- ✓ Chrome (Android) - Full support
- ✓ Edge (Desktop) - Full support
- ✗ Safari - Not supported (provide fallback)
- ✗ Firefox - Not supported (provide fallback)
```

## Security Considerations

### 1. HTTPS Required

Protocol handlers only work over HTTPS:

```yaml
# ✓ Works - HTTPS
https://your-app.com

# ✗ Doesn't work - HTTP
http://your-app.com
```

### 2. Validate Input

Always validate and sanitize protocol arguments:

```php
public function handleProtocol(Request $request): Response
{
    $input = $request->query->get('data', '');

    // Validate and sanitize
    if (!$this->isValid($input)) {
        throw $this->createAccessDeniedException('Invalid input');
    }

    // Escape for output
    $safeInput = htmlspecialchars($input, ENT_QUOTES, 'UTF-8');

    return $this->render('template.html.twig', [
        'data' => $safeInput,
    ]);
}
```

### 3. Prevent Open Redirects

```php
public function handleUrl(Request $request): Response
{
    $url = $request->query->get('url', '');

    // Validate URL is from your domain
    if (!$this->isInternalUrl($url)) {
        throw $this->createAccessDeniedException('External URLs not allowed');
    }

    return $this->redirect($url);
}

private function isInternalUrl(string $url): bool
{
    $parsed = parse_url($url);
    $allowedHosts = ['yourdomain.com', 'www.yourdomain.com'];

    return isset($parsed['host']) && in_array($parsed['host'], $allowedHosts);
}
```

### 4. Respect User Choice

Always provide an easy way for users to opt-out:

```html
<div class="settings">
    <h3>Protocol Handlers</h3>
    <p>You can disable protocol handling in your browser settings.</p>
    <a href="chrome://settings/handlers" target="_blank">
        Manage Protocol Handlers
    </a>
</div>
```

## Common Issues

### Protocol Handler Not Appearing

**Problem**: PWA doesn't appear as protocol handler option

**Solutions**:
- Verify PWA is installed (not just bookmarked)
- Check browser supports protocol handlers (Chrome/Edge only)
- Ensure manifest includes `protocol_handlers`
- Clear browser cache and reinstall PWA
- Check DevTools → Application → Manifest

### Permission Not Requested

**Problem**: Browser doesn't prompt for protocol handler permission

**Solutions**:
- User must click a protocol link first
- Browser only prompts once per protocol
- Check if permission was previously denied
- Reset permissions in browser settings

### Wrong URL Generated

**Problem**: Protocol handler opens wrong URL

**Solutions**:
- Verify `%s` placeholder is in URL
- Check `path_type_reference` value
- Ensure Symfony route exists
- Check route parameters match configuration

### Protocol Not Working

**Problem**: Clicking protocol link doesn't open PWA

**Solutions**:
- Verify protocol format (must start with `web+` for custom)
- Check PWA is set as default handler
- Test in supported browser (Chrome/Edge)
- Verify PWA is still installed

## Troubleshooting

### Debug Protocol Handling

```javascript
// Monitor protocol launches
if ('launchQueue' in window) {
    window.launchQueue.setConsumer((launchParams) => {
        console.log('Launched with:', launchParams.targetURL);

        // Extract protocol
        const url = new URL(launchParams.targetURL);
        console.log('Protocol:', url.protocol);
        console.log('Full URL:', url.href);
    });
}
```

### Check Protocol Registration

```bash
1. Open chrome://settings/handlers
2. Look for your PWA
3. Verify protocols are listed
4. Check "Allow" is selected
```

### Test Without Installation

Protocol handlers only work for installed PWAs. To test:
1. Install PWA first
2. Then click protocol links
3. Or test from external page/app

## Related Documentation

- [Shortcuts](shortcuts.md) - App shortcuts configuration
- [Start URL](application-information/start-url.md) - URL configuration
- [File Handlers](file-handlers.md) - Handle file types

## Resources

- **MDN Protocol Handlers**: https://developer.mozilla.org/en-US/docs/Web/Manifest/protocol_handlers
- **Web.dev**: https://web.dev/url-protocol-handler/
- **Chrome Platform Status**: https://chromestatus.com/feature/5680742077038592
- **W3C Spec**: https://w3c.github.io/manifest-app-info/#protocol_handlers-member

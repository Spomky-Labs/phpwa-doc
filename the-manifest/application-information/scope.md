# Scope

The `scope` property defines the navigation boundary of your PWA. It determines which URLs are considered part of your application and which will open in a regular browser tab.

## Purpose

The scope controls:
- **App boundaries**: Which pages stay in the PWA window
- **Browser context**: When to open the system browser
- **Service worker registration**: What URLs the service worker controls
- **User experience**: Seamless navigation within your app vs external links

## Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        scope: "/"  # Controls all URLs under your domain
```
{% endcode %}

{% hint style="info" %}
If not specified, the scope defaults to the directory containing the manifest file.
{% endhint %}

## How Scope Works

### URLs Within Scope

URLs within the scope remain in your PWA window:

```yaml
scope: "/app/"
```

**Within scope** (stays in PWA):
- `https://example.com/app/` ✓
- `https://example.com/app/dashboard` ✓
- `https://example.com/app/settings/profile` ✓

**Outside scope** (opens in browser):
- `https://example.com/` ✗
- `https://example.com/blog/` ✗
- `https://example.com/about` ✗
- `https://other-domain.com/` ✗

### Relationship with start_url

The `start_url` **must** be within the scope:

```yaml
# ✓ Valid - start_url within scope
scope: "/app/"
start_url: "/app/dashboard"

# ✗ Invalid - start_url outside scope
scope: "/app/"
start_url: "/login"  # Error!
```

## Common Scope Patterns

### Pattern 1: Root Scope (Most Common)

Control the entire domain:

```yaml
pwa:
    manifest:
        scope: "/"
        start_url: "/"
```

**Use when**:
- Your PWA is the entire website
- You want all pages in the PWA experience
- Single-page applications

**Examples**: Gmail, Twitter PWA, most SPAs

### Pattern 2: Application Subfolder

Limit PWA to a specific section:

```yaml
pwa:
    manifest:
        scope: "/app/"
        start_url: "/app/"
```

**Use when**:
- PWA is part of a larger website
- Marketing pages are separate
- Multiple apps on same domain

**Examples**:
- `/app/` - Main application
- `/` - Marketing website
- `/blog/` - Blog (opens in browser)

### Pattern 3: Versioned Scope

Include version in scope:

```yaml
pwa:
    manifest:
        scope: "/v2/"
        start_url: "/v2/home"
```

**Use when**:
- Running multiple app versions
- Gradual migration strategies
- A/B testing different versions

### Pattern 4: Language-Specific Scope

Separate scopes per language:

```yaml
# English version
pwa:
    manifest:
        scope: "/en/"
        start_url: "/en/"

# French version
pwa:
    manifest:
        scope: "/fr/"
        start_url: "/fr/"
```

**Use when**:
- Different manifests per language
- Localized app experiences
- Regional variations

## Practical Examples

### Example 1: E-commerce Site

Marketing site + shop PWA:

```yaml
pwa:
    manifest:
        name: "ShopApp"
        scope: "/shop/"  # Only shop is PWA
        start_url: "/shop/"
```

**Result**:
- `/shop/products` - PWA window ✓
- `/shop/cart` - PWA window ✓
- `/` - Opens in browser (marketing)
- `/about` - Opens in browser (corporate)

### Example 2: SaaS Dashboard

App separate from landing pages:

```yaml
pwa:
    manifest:
        name: "Dashboard"
        scope: "/dashboard/"
        start_url: "/dashboard/overview"
```

**Result**:
- `/dashboard/*` - PWA ✓
- `/pricing` - Browser (public page)
- `/docs` - Browser (documentation)

### Example 3: Multi-tenant Application

Each tenant has own scope:

```yaml
pwa:
    manifest:
        name: "TenantApp"
        scope: "/tenant/acme-corp/"
        start_url: "/tenant/acme-corp/dashboard"
```

**Result**:
- `/tenant/acme-corp/*` - PWA for this tenant ✓
- `/tenant/other-corp/*` - Opens in browser
- `/admin/*` - Opens in browser

### Example 4: Full-Site PWA

Everything is the PWA:

```yaml
pwa:
    manifest:
        name: "MyApp"
        scope: "/"
        start_url: "/"
```

**Result**:
- All URLs on your domain stay in PWA ✓
- External links open in browser

## Advanced Configurations

### Multiple Manifests for Different Scopes

Serve different manifests for different sections:

```php
// In your Symfony controller
public function getManifest(Request $request): Response
{
    $path = $request->getPathInfo();

    if (str_starts_with($path, '/admin/')) {
        return $this->render('manifest/admin.yaml');
    }

    return $this->render('manifest/app.yaml');
}
```

```yaml
# config/packages/pwa.yaml - App manifest
pwa:
    manifest:
        scope: "/app/"
        start_url: "/app/"

# Could have separate config for admin
```

### Dynamic Scope Based on User

```yaml
# Authenticated users
scope: "/dashboard/"

# Public users
scope: "/public/"
```

## Scope and Service Workers

The scope in your manifest should align with your service worker scope:

```yaml
# Manifest scope
pwa:
    manifest:
        scope: "/app/"

    # Service worker scope should match
    serviceworker:
        scope: "/app/"
```

{% hint style="warning" %}
If manifest scope and service worker scope don't align, you may experience unexpected behavior with offline functionality.
{% endhint %}

## Testing Your Scope

### 1. Check Manifest in DevTools

```bash
# Chrome DevTools
1. Open DevTools (F12)
2. Go to Application → Manifest
3. Check "scope" value
4. Verify "start_url" is within scope
```

### 2. Test Navigation Behavior

Click links to verify:
- Internal links stay in PWA window
- External links open in browser
- Cross-scope links open in browser

### 3. Test with Different URLs

```javascript
// Test if URL is in scope
const manifestScope = '/app/';
const currentUrl = window.location.pathname;

if (currentUrl.startsWith(manifestScope)) {
    console.log('URL is within PWA scope');
} else {
    console.log('URL is outside PWA scope');
}
```

## Common Mistakes

### 1. start_url Outside Scope

```yaml
# ✗ Error - will fail validation
pwa:
    manifest:
        scope: "/app/"
        start_url: "/"  # Not in scope!
```

**Fix**:
```yaml
# ✓ Correct
pwa:
    manifest:
        scope: "/app/"
        start_url: "/app/"
```

### 2. Too Restrictive Scope

```yaml
# ✗ Too narrow - breaks navigation
pwa:
    manifest:
        scope: "/app/dashboard/home/"  # Too specific!
```

**Impact**: Users navigating to `/app/settings` will leave the PWA.

**Fix**:
```yaml
# ✓ Broader scope
pwa:
    manifest:
        scope: "/app/"
```

### 3. Forgetting Trailing Slash

```yaml
# These are different!
scope: "/app"   # Matches /app, /application, /appdata
scope: "/app/"  # Matches /app/, /app/page, but not /application
```

**Best practice**: Use trailing slash for directory-based scopes:

```yaml
# ✓ Recommended for directories
scope: "/app/"

# ✓ OK for root
scope: "/"
```

### 4. Mismatched Service Worker Scope

```yaml
# ✗ Misaligned scopes
pwa:
    manifest:
        scope: "/app/"

    serviceworker:
        scope: "/"  # Different from manifest scope
```

**Impact**: Service worker may not control all PWA pages.

**Fix**:
```yaml
# ✓ Aligned scopes
pwa:
    manifest:
        scope: "/app/"

    serviceworker:
        scope: "/app/"
```

## Scope Validation Errors

### Error: start_url not within scope

```
Error: The start_url "/login" is not within the scope "/app/"
```

**Solution**: Adjust start_url or scope:

```yaml
# Option 1: Change start_url
start_url: "/app/login"

# Option 2: Expand scope
scope: "/"
```

### Error: Invalid scope format

```
Error: Scope must be a valid URL path
```

**Solution**: Use proper path format:

```yaml
# ✗ Wrong
scope: "app"
scope: "www.example.com/app/"

# ✓ Correct
scope: "/app/"
scope: "/"
```

## Platform-Specific Behavior

### iOS/Safari

- Respects scope for navigation
- Service worker scope must match or be narrower
- May show URL bar for out-of-scope navigation

### Android/Chrome

- Strongly enforces scope boundaries
- Out-of-scope links open in Chrome Custom Tabs
- Clear visual indication when leaving PWA

### Desktop Browsers

- Opens new tab for out-of-scope URLs
- May show browser UI for external navigation
- Varies by browser implementation

## Best Practices

### 1. Start Broad, Refine Later

```yaml
# Start with broad scope
scope: "/"

# Later, if needed, narrow it down
scope: "/app/"
```

### 2. Align with Site Architecture

Match your URL structure:

```yaml
# If your app lives under /app/
scope: "/app/"

# If entire site is the app
scope: "/"
```

### 3. Consider Future Growth

Leave room for expansion:

```yaml
# ✓ Good - allows for /app/v2/, /app/beta/
scope: "/app/"

# ✗ Too specific
scope: "/app/v1/"
```

### 4. Document Your Scope Decision

```yaml
pwa:
    manifest:
        # Scope limited to /app/ because:
        # - Marketing pages should not be in PWA
        # - Blog is separate site section
        # - Admin uses different manifest
        scope: "/app/"
```

### 5. Test Across Platforms

Verify scope behavior on:
- iOS Safari
- Android Chrome
- Desktop Chrome
- Desktop Edge
- Other target browsers

## Debugging Scope Issues

### Check Current Scope

```javascript
// In your PWA
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.ready.then(registration => {
        console.log('SW scope:', registration.scope);
    });
}

// Compare with manifest scope
fetch('/manifest.json')
    .then(r => r.json())
    .then(manifest => {
        console.log('Manifest scope:', manifest.scope);
    });
```

### Monitor Navigation Events

```javascript
// Detect when leaving scope
window.addEventListener('beforeunload', (e) => {
    console.log('Navigating to:', document.activeElement.href);

    const manifestScope = '/app/';
    const targetUrl = new URL(document.activeElement.href);

    if (!targetUrl.pathname.startsWith(manifestScope)) {
        console.log('Leaving PWA scope - will open in browser');
    }
});
```

### Validate scope Configuration

```javascript
// Helper to validate scope/start_url relationship
function validateManifest(manifest) {
    const scope = new URL(manifest.scope, location.origin);
    const startUrl = new URL(manifest.start_url, location.origin);

    if (!startUrl.href.startsWith(scope.href)) {
        console.error('start_url must be within scope!');
        console.log('Scope:', scope.href);
        console.log('Start URL:', startUrl.href);
        return false;
    }

    return true;
}
```

## Related Properties

- [Start URL](start-url.md) - Where your PWA starts (must be within scope)
- [ID](id.md) - Unique identifier for your PWA
- [Display Mode](../display.md) - How the app window appears

## Summary

Scope best practices:
- ✓ Use `/` for full-site PWAs
- ✓ Use `/app/` to separate PWA from marketing
- ✓ Ensure `start_url` is always within `scope`
- ✓ Align manifest scope with service worker scope
- ✓ Use trailing slashes for directory-based scopes
- ✓ Test navigation behavior across scope boundaries
- ✓ Document why you chose a specific scope
- ✗ Don't make scope too restrictive
- ✗ Don't forget to test on real devices
- ✗ Don't change scope frequently (affects PWA identity)

# Advanced Properties

This page documents advanced manifest properties that provide fine-grained control over your PWA's behavior and integration with the operating system.

## Display Override

The `display_override` property allows you to specify a sequence of display modes that the browser will consider before falling back to the standard `display` property. This gives you more control over how your app is displayed on different platforms.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        display: "standalone"
        display_override: ['window-controls-overlay', 'minimal-ui']
```
{% endcode %}

### Available Display Modes

The browser will try each mode in order and use the first one it supports:

* `"fullscreen"` - Full screen without any browser UI
* `"standalone"` - Looks like a standalone app with no browser UI
* `"minimal-ui"` - Similar to standalone but with minimal browser UI (back/forward buttons)
* `"browser"` - Standard browser tab experience
* `"window-controls-overlay"` - Places window controls (minimize, maximize, close) in the title bar, giving your app more screen space
* `"tabbed"` - Multiple app windows appear as tabs in a single window (experimental)

### Example with Fallbacks

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        display: "standalone"
        display_override:
            - 'window-controls-overlay'
            - 'minimal-ui'
            - 'standalone'
```
{% endcode %}

In this example:
1. The browser first tries `window-controls-overlay` (modern desktop browsers)
2. If not supported, falls back to `minimal-ui`
3. If not supported, uses `standalone`
4. If none are supported, uses the standard `display` value

{% hint style="info" %}
The `display_override` is particularly useful for providing an enhanced experience on desktop browsers that support window controls overlay while maintaining compatibility with mobile browsers.
{% endhint %}

## Use Credentials

The `use_credentials` property controls whether the manifest is fetched with credentials (cookies, HTTP authentication).

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        use_credentials: true
```
{% endcode %}

**Default value:** `true`

When `use_credentials` is set to `true`:
* The browser includes cookies when fetching the manifest
* HTTP authentication headers are included
* CORS credentials mode is used

**Set to `true` when:**
* Your manifest is behind authentication
* You serve different manifests to different users

**Set to `false` when:**
* Your manifest is publicly accessible
* You want to avoid potential CORS issues
* You're serving the manifest from a CDN

## Handle Links

The `handle_links` property specifies how the PWA should handle links when it's already open.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        handle_links: "preferred"
```
{% endcode %}

### Available Values

* `"auto"` (default) - The browser decides based on its own heuristics
* `"preferred"` - Prefer opening links in the PWA when it's already open
* `"not-preferred"` - Prefer opening links in the browser

{% hint style="info" %}
The `handle_links` property works in conjunction with the `scope` property. Links outside the scope are not affected by this setting.
{% endhint %}

## Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        use_credentials: true

        # Core display settings
        display: "standalone"
        display_override:
            - 'window-controls-overlay'
            - 'minimal-ui'

        # Link handling
        handle_links: "preferred"

        # Other properties
        scope: "/"
        start_url: "app_homepage"
```
{% endcode %}

## Browser Support

{% hint style="warning" %}
These advanced properties have varying levels of browser support:

* **display_override**: Supported in Chromium-based browsers (Chrome, Edge)
* **use_credentials**: Widely supported across modern browsers
* **handle_links**: Limited support, primarily in Chromium-based browsers

Always test your PWA on target browsers and provide sensible fallbacks through the standard `display` property.
{% endhint %}

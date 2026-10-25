# Advanced properties

This page documents advanced manifest properties that provide fine-grained control over your PWA's behavior and integration with the operating system.

## Display override

The `display_override` property allows you to specify a sequence of display modes that the browser will consider before falling back to the standard `display` property. This gives you more control over how your app is displayed on different platforms and browsers.

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        display: "standalone"
        display_override: ['window-controls-overlay', 'minimal-ui']
```

### Available display modes

The browser will try each mode in order and use the first one it supports:

* `"fullscreen"` - Full screen without any browser UI
* `"standalone"` - Looks like a standalone app with no browser UI
* `"minimal-ui"` - Similar to standalone but with minimal browser UI (back/forward buttons)
* `"browser"` - Standard browser tab experience
* `"window-controls-overlay"` - Places window controls (minimize, maximize, close) in the title bar, giving your app more screen space
* `"tabbed"` - Multiple app windows appear as tabs in a single window (experimental)

### Example with fallbacks

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        display: "standalone"
        display_override:
            - 'window-controls-overlay'
            - 'minimal-ui'
            - 'standalone'
```

In this example:
1. The browser first tries `window-controls-overlay` (modern desktop browsers)
2. If not supported, falls back to `minimal-ui`
3. If not supported, uses `standalone`
4. If none are supported, uses the standard `display` value

!!! info

    The `display_override` is particularly useful for providing an enhanced experience on desktop browsers that support window controls overlay while maintaining compatibility with mobile browsers.

## Use credentials

The `use_credentials` property controls whether the manifest is fetched with credentials (cookies, HTTP authentication). This is useful when your manifest file is protected or when you need to serve different manifests to different users.

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        use_credentials: true
```

**Default value:** `true`

When `use_credentials` is set to `true`:
* The browser includes cookies when fetching the manifest
* HTTP authentication headers are included
* CORS credentials mode is used

When set to `false`:
* The manifest is fetched without credentials
* No cookies or authentication headers are sent

!!! warning

    If your manifest file is protected by authentication and `use_credentials` is `false`, the manifest fetch will fail, preventing PWA installation.

### When to use credentials

**Set to `true` when:**
* Your manifest is behind authentication
* You serve different manifests to different users
* Your manifest URL is on a protected subdomain

**Set to `false` when:**
* Your manifest is publicly accessible
* You want to avoid potential CORS issues
* You're serving the manifest from a CDN

## Handle links

The `handle_links` property specifies how the PWA should handle links when it's already open. This gives you control over whether links should open in the PWA window or in the browser.

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        handle_links: "preferred"
```

### Available values

* `"auto"` (default) - The browser decides based on its own heuristics
* `"preferred"` - Prefer opening links in the PWA when it's already open
* `"not-preferred"` - Prefer opening links in the browser

### Example use cases

**E-commerce app:**
```yaml
pwa:
    manifest:
        handle_links: "preferred"
```
Keep users in the PWA for a seamless shopping experience.

**Documentation site:**
```yaml
pwa:
    manifest:
        handle_links: "auto"
```
Let the browser decide based on user behavior and context.

**External link aggregator:**
```yaml
pwa:
    manifest:
        handle_links: "not-preferred"
```
Most links point to external sites, so prefer opening in the browser.

!!! info

    The `handle_links` property works in conjunction with the `scope` property. Links outside the scope are not affected by this setting.

## Complete example

Here's how these advanced properties work together:

```yaml title="/config/packages/pwa.yaml" linenums="1"
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

## Browser support

!!! warning

    These advanced properties have varying levels of browser support:

    * **display_override**: Supported in Chromium-based browsers (Chrome, Edge)
    * **use_credentials**: Widely supported across modern browsers
    * **handle_links**: Limited support, primarily in Chromium-based browsers

    Always test your PWA on target browsers and provide sensible fallbacks through the standard `display` property.

## Best practices

1. **Progressive Enhancement**: Use `display_override` to provide enhanced experiences while maintaining a solid baseline
2. **Test Thoroughly**: Test link handling behavior to ensure it matches user expectations
3. **Document Behavior**: Make it clear to users how your PWA handles links, especially for external URLs
4. **Consider Privacy**: Be mindful of credentials when fetching manifests, especially on shared devices

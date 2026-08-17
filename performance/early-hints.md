# Early Hints (HTTP 103)

Early Hints is a feature that allows the server to send preliminary HTTP headers before the main response is ready. This enables browsers to start loading critical resources even before the HTML document arrives.

## How It Works

1. Browser requests a page
2. Server sends **103 Early Hints** response with `Link` headers
3. Browser starts preloading resources immediately
4. Server sends **200 OK** with the actual HTML content
5. Resources are already loading (or loaded) when the browser parses the HTML

This can significantly improve **Largest Contentful Paint (LCP)** by several hundred milliseconds.

## Server Requirements

Early Hints requires a compatible server:

- **FrankenPHP** (recommended with Symfony)
- **Caddy**
- **Apache** (with mod_http2)
- **nginx** (version 1.25.3+)

{% hint style="warning" %}
If your server doesn't support Early Hints, the `Link` headers will still be sent with the main response. They won't provide the same performance benefit but won't cause any issues.
{% endhint %}

## Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
  early_hints:
    enabled: true
    preload_manifest: true        # Preload the PWA manifest
    preload_serviceworker: false  # Preload the service worker script
    preconnect_workbox_cdn: true  # Preconnect to Workbox CDN (if using CDN)
```
{% endcode %}

## Options

### preload_manifest

When enabled, sends a preload hint for the PWA manifest file:

```http
Link: </site.webmanifest>; rel="preload"; as="fetch"; crossorigin="anonymous"
```

This allows the browser to fetch the manifest in parallel with parsing the HTML, speeding up PWA installation prompts.

**Default:** `true`

### preload_serviceworker

When enabled, sends a preload hint for the service worker script:

```http
Link: </sw.js>; rel="preload"; as="script"
```

This can help the service worker register faster, but use with caution as it adds to the critical path.

**Default:** `false`

### preconnect_workbox_cdn

When using Workbox from CDN (`use_cdn: true`), sends a preconnect hint:

```http
Link: <https://storage.googleapis.com>; rel="preconnect"; crossorigin
```

This establishes an early connection to the CDN, reducing latency when loading Workbox modules.

{% hint style="warning" %}
`use_cdn` is deprecated since 1.6.0 and will be removed in 2.0.0, together with this hint: the bundled
Workbox files are served by your own server, which needs no preconnect. See
[Workbox Files](../the-service-worker/workbox/cdn-and-versions.md).
{% endhint %}

**Default:** `true`

## Usage with FrankenPHP

FrankenPHP automatically handles Early Hints when `Link` headers are present in the request attributes. No additional configuration is needed beyond enabling the feature in this bundle.

```php
// The bundle automatically adds Link headers to the request
// FrankenPHP detects these and sends 103 Early Hints
```

## Combining with Resource Hints

Early Hints works well with the [Resource Hints](resource-hints.md) feature. Both features add `Link` headers to the request, but they serve different purposes:

- **Early Hints**: Sends headers in a 103 response before the main content
- **Resource Hints**: Adds `<link>` tags in the HTML `<head>`

When both are enabled, resources get the benefit of:
1. Early Hints for compatible servers (103 response)
2. HTML link tags as fallback for all browsers

## Example Response Flow

### With Early Hints (FrankenPHP/Caddy)

```http
HTTP/1.1 103 Early Hints
Link: </site.webmanifest>; rel="preload"; as="fetch"; crossorigin="anonymous"
Link: <https://storage.googleapis.com>; rel="preconnect"; crossorigin

HTTP/1.1 200 OK
Content-Type: text/html
Link: </site.webmanifest>; rel="preload"; as="fetch"; crossorigin="anonymous"

<!DOCTYPE html>
...
```

### Without Early Hints (standard server)

```http
HTTP/1.1 200 OK
Content-Type: text/html
Link: </site.webmanifest>; rel="preload"; as="fetch"; crossorigin="anonymous"

<!DOCTYPE html>
...
```

## Performance Benefits

| Metric | Improvement |
|--------|-------------|
| LCP | 100-500ms faster |
| TTFB perceived | Improved (resources load during server processing) |
| PWA prompt | Faster appearance |

## Best Practices

1. **Enable for production**: Early Hints has minimal overhead and provides significant benefits on compatible servers.

2. **Keep preload list small**: Only preload truly critical resources. Too many hints can overwhelm the browser.

3. **Test with DevTools**: Use the Network panel to verify 103 responses are being sent (look for the "early hints" indicator).

4. **Monitor Core Web Vitals**: Track LCP improvements after enabling Early Hints.

## Further Reading

- [Symfony Early Hints Blog Post](https://symfony.com/blog/new-in-symfony-6-3-early-hints)
- [FrankenPHP Early Hints](https://frankenphp.dev/docs/early-hints/)
- [Chrome Developers: Early Hints](https://developer.chrome.com/docs/web-platform/early-hints)

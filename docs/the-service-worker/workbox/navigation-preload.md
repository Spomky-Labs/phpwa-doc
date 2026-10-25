# Navigation Preload

Navigation Preload is a feature that speeds up navigation requests by allowing the browser to make network requests in parallel with service worker boot-up. Without Navigation Preload, the browser must wait for the service worker to start before making network requests, which can introduce latency.

## How It Works

When a user navigates to a page:

1. **Without Navigation Preload**: Browser waits for the service worker to start → Service worker makes the network request → Response is returned
2. **With Navigation Preload**: Browser starts the service worker AND makes the network request simultaneously → Faster response

This can significantly improve perceived performance, especially on devices where service worker startup is slow.

## When to Use Navigation Preload

Navigation Preload is beneficial when:

- Your service worker uses a **NetworkFirst** or **NetworkOnly** strategy for navigation requests
- You are **not** precaching HTML pages
- You want to reduce the latency introduced by service worker startup

## When NOT to Use Navigation Preload

{% hint style="warning" %}
**Do not enable Navigation Preload if you are precaching HTML pages.** If you're using `offline_fallback` with a page fallback or `preload_urls` to warm cache HTML pages, Navigation Preload would be redundant and may cause unnecessary network requests.
{% endhint %}

This is why Navigation Preload is **disabled by default** in this bundle.

## Configuration

To enable Navigation Preload:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            navigation_preload: true
```
{% endcode %}

## Browser Support

Navigation Preload is supported in modern browsers including Chrome, Edge, and Opera. Firefox and Safari do not currently support this feature, but Workbox handles this gracefully by falling back to standard behavior.

## Further Reading

- [Workbox Navigation Preload Documentation](https://developer.chrome.com/docs/workbox/modules/workbox-navigation-preload/)
- [Speed up Service Worker with Navigation Preloads](https://web.dev/navigation-preload/)

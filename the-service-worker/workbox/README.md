# Workbox

This bundle provides a simple Workbox integration path. Service Workers are not that easy to write. It can become a nightmare when dealing with mutliple caching rules or offline capabilities.&#x20;

Workbox takes the pain out of service worker creation by providing a set of tools and best practices that can be used out of the box. It’s like having an expert by your side, guiding you through the complexities of browser caching and service worker logic.

This bundle builds on top of Workbox and includes several options so you don't need to write a line of Javascript to have a fully functional Service Worker.

## Key Features

* **Precaching**: Workbox can precache the assets in your web app and keep them up to date.
* **Runtime Caching**: Flexible strategies for handling runtime requests, such as stale-while-revalidate, cache first, and network first.
* **Request Routing**: Easily define how different types of requests are handled by your service worker.
* **Background Sync**: Integrate background sync to replay failed requests once connectivity is restored.
* **Offline Fallback**: allow your service worker to serve a web page, image, or font if there's a routing error for any of the three, for instance if a user is offline and there isn't a cache hit.

### Disabling Workbox

Workbox is enabled by default. You can disable it completely if you do not need it.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: false
```
{% endcode %}

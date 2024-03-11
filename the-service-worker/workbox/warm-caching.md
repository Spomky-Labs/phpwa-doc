# Warm Caching

Workbox provides various caching strategies to make it easy to manage how requests are handled by the service worker. One of the features is the ability to maintain a _warm cache_ for certain URLs.

A warm cache refers to pre-caching URLs during the service worker installation phase. This ensures that those URLs are cached and readily available even before they are requested by the user.

The URLs to be pre-cached are likely the pages that users will definitely navigate to. For example, the main functionality of your application, the pricing page or a page with the latest news that users will definitely read.

When the service worker is installed, Workbox will automatically cache the pages specified in the list. These files will remain cached and will be instantly available to the user, contributing to a swift user experience.

<mark style="color:green;">**This feature is enabled by default!**</mark> All assets (images, stylesheets or scripts) managed by Asset Mapper are cached.

Hereafter the main benefits of Precache:

* **Faster Load Times**: Since assets are stored locally, web applications can load faster, providing a better user experience.
* **Offline Support**: Precached assets ensure that the application is usable even without a network connection.
* **Consistency**: All users receive the same version of files, ensuring a consistent experience.
* **Background Updates**: Assets are updated in the background, preventing any interruption to the user experience.

### Page Caching

{% hint style="danger" %}
Contains a breaking change compared to v1.0.x
{% endhint %}

Developers can build more resilient and user-friendly web applications that perform reliably under various network conditions. Also, it is possible to warm cache a selection of pages. This is powerfull as it allows applications to partially work offline.

The strategy applied for page is Network First i.e. the page from the web server is fetched first. In case of failure, the cached data is served. By default, the service worker will wait 3 seconds before serving the cached version. This value can be configured using the `network_timeout_seconds` option.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            page_caches:
                - regex: '\/pages\/.*$'
                  cache_name: 'static-pages'
                  strategy: 'networkFirst'
                  network_timeout: 2 # Wait only 2 seconds (only when strategy = networkFirst
                  urls: # List of URLs to precache. The URL shall be comprised within the regex
                      - 'page_tos'
                      - 'page_legal'
                - regex: '\/articles\/.*$'
                  cache_name: 'articles'
                  strategy: 'staleWhileRevalidate'
                  broadcast: true # Broadcast changes only when strategy = staleWhileRevalidate
                  urls: # List of URLs to precache. The URL shall be comprised within the regex
                      - 'app_articles'
                      - 'app_top_articles'
                        params:
                            display: 5
```
{% endcode %}

{% hint style="info" %}
Please note that you can refer to any URLs, but only URLs served by your application will be cached.
{% endhint %}

#### Broadcast Parameter

When accessing a page with the `staleWhileRevalidate` strategy, the Service Worker will verify if a page update exists and will save it in the cache if any.

The broadcast parameter will tell the Service Worker to send a broadcast message in case of an update. This is usefull to warn the user it is reading an outdated version of the page and ask for a page reload.&#x20;

In the example below, the `message` is catched and, if it is of type "`workbox-broadcast-update`" and the URL matches with the current URL, a toast notification is displayed for 5 seconds.

```javascript
navigator.serviceWorker.addEventListener('message', async event => {
    if (event.data.meta === 'workbox-broadcast-update') {
        const {updatedURL} = event.data.payload;
        if (updatedURL === window.location.href) {
            const toast = document.getElementById('toast-refresh');
            toast.classList.remove('hidden');
            setTimeout(() => {
                toast.classList.add('hidden');
            }, 5000);
        }
    }
});
```

#### Broadcast Header

By default, the page header used to check if the page is outdated or not are `Content-Length`, `ETag`, `Last-Modified`. These headers can be changed.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            page_caches:
                - ...
                  broadcast_headers:
                      - 'X-App-Cache'
```
{% endcode %}

### Image Caching

By default, a maximum of 60 images are cached for 1 year. The supported image extensions extensions are as follows:&#x20;

```regex
/\.(ico|png|jpe?g|gif|svg|webp|bmp)$/
```

This can be changed using the next configuration options

<pre class="language-yaml" data-title="/config/packages/pwa.yaml" data-line-numbers><code class="lang-yaml">pwa:
<strong>    serviceworker:
</strong>        enabled: true
        src: "sw.js"
        workbox:
            image_cache:
                enabled: true
                max_age: 2_592_000 # 30 days
                max_entries: 200
                regex: '/\.(png|jpe?g|svg|webp)$/'
</code></pre>

### Asset caching

{% hint style="danger" %}
Contains a breaking change compared to v1.0.x
{% endhint %}

Similary to images, e.g. CSS, JSON, XML or JS files are cached. The supported extensions are as follows:&#x20;

```regex
/\.(css|js|json|xml|txt|map|ico|png|jpe?g|gif|svg|webp|bmp)$/
```

This can be changed using the next configuration options:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            asset_cache:
                enabled: true
                regex: '/\.(css|jsx?)$/'
```
{% endcode %}

### Font Caching

By default, 30 fonts can be cached for 1 year. This only comprises fonts served directly by your application. The supported fonts are as follows:

```regex
/\.(ttf|eot|otf|woff2)$/
```

This can be changed using the next configuration options:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            font_cache:
                enabled: true
                max_entries: 10
                max_age: 2_592_000 # 30 days
```
{% endcode %}

### Google Font Caching

Fonts served by Google Fonts have a dedicated rule. It is enabled by default and you can disable it or customize some parameters.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            google_fonts:
                enabled: true
                cache_prefix: 'goolge-fonts'
                max_entries: 20
                max_age: 86_400 # 1 day
```
{% endcode %}

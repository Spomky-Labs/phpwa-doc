# Warm Caching

Workbox provides various caching strategies to make it easy to manage how requests are handled by the service worker. One of the features is the ability to maintain a _warm cache_ for certain URLs.

A warm cache refers to pre-caching URLs during the service worker installation phase. This ensures that those URLs are cached and readily available even before they are requested by the user.

The URLs to be pre-cached are likely the pages that users will definitely navigate to. For example, the main functionality of your application, the pricing page or a page with the latest news that users will definitely read.

When the service worker is installed, Workbox will automatically cache the pages specified in the list. These files will remain cached and will be instantly available to the user, contributing to a swift user experience.

Workbox is enabled by default. You can disable it completely if you do not need it.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: false
                warm_cache_urls:
                    - 'app_homepage' # Simple route name
                    - path: 'app_feature1' # Route name without parameters
                    - path: 'app_feature2' # Route name with parameters
                      params:
                          foo: 'bar'
                          param1: 'value1'
```
{% endcode %}

{% hint style="info" %}
Please note that you can refer to any URLs, but only URLs served by your application will be cached.
{% endhint %}

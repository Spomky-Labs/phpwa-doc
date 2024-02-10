# Cache Names and Purge

You can customize the cache names if needed. The configuration options are as follows:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            image_cache_name: 'images'
            font_cache_name: 'fonts'
            page_cache_name: 'pages'
            asset_cache_name: 'assets'
```
{% endcode %}

When the service worker changes and is activated, the cache is purged. This feature is made to avoid the cache to grow in size and prevent your application to reach the limit of the host system or browser.

You can disable the cache purge, however you have to make sure the cached data is managed by other means.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            clear_cache: false # Default to true
```
{% endcode %}

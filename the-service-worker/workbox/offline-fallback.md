# Offline Fallbacks

Workbox makes it easy to create robust offline experiences in web applications. When a user navigates to a page without an internet connection, instead of showing an error, your application can serve a predefined fallback page, image or font.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            page_fallback: 'app_offline_page'
            image_fallback: 'images/offline.svg'
            font_fallback: 'fonts/normal.ttf'
```
{% endcode %}

`page_fallback` can be a relative URL, a route name or a route name with parameters. See [URL parameter](../../the-manifest/shortcuts.md#url-parameter) for more information.

`image_fallback` and `font_fallback` refer to assets delivered by Asset Mapper or an URL to the font file.

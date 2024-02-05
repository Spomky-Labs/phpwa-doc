# Offline Fallback

Workbox makes it easy to create robust offline experiences in web applications. When a user navigates to a page without an internet connection, instead of showing an error, your application can serve a predefined fallback page.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: false
            offline_fallback: 'app_offline_page'
```
{% endcode %}

offline\_fallback can be a relative URL, a route name or a route name with parameters. See [URL parameter](../../the-manifest/shortcuts.md#url-parameter) for more information.

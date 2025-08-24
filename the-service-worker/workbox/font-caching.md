# Font Caching

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
                max_age: 30 days
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
                max_age: 1 day
```
{% endcode %}

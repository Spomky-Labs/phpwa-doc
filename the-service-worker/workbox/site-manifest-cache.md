# Site Manifest Cache

The Site Manifest file can be cache by the bundle. This feature is enabled by default and mainly used to avoid error messages in the console and adds no significant benefits over the time.



{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            cache_manifest: true
```
{% endcode %}

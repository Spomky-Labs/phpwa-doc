# CDN and Versions

By default, the bundle uses Workbox `7.0.0` which was released end of May 2023. The files are provided by the bundle so that all files are served by your server.

If needed, you can use a CDN and a different version.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            use_cdn: true
            version: 6.5.4
```
{% endcode %}

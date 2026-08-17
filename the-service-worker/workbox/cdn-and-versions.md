# CDN and Versions

The bundle ships the Workbox libraries, so that everything is served by your own server: no third-party request, and a Content Security Policy that needs no exception.

**The bundled version is `7.4.1`**, and it is the default since 1.6.0. It is copied to your public folder by `php bin/console pwa:compile`.

## Using another version

Only the bundled version can be served locally. Any other version requires the CDN:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            config:
                use_cdn: true
                version: '7.3.0'
```
{% endcode %}

{% hint style="warning" %}
Setting `version` to anything other than `7.4.1` without `use_cdn: true` points the service worker at files that were never written to your public folder, and the import fails in the browser.
{% endhint %}

## Where the files are served from

With `use_cdn: false` — the default — the libraries are written under `workbox_public_url`:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            config:
                workbox_public_url: '/workbox'
```
{% endcode %}

{% hint style="info" %}
`use_cdn`, `version` and `workbox_public_url` moved under `workbox.config` in 1.5.0. Their old location still works but is deprecated, and will be removed in 2.0.0.
{% endhint %}

## Debug logging

Workbox logs what every route and strategy does in the browser console, and loads its development builds to do so. The `debug` option is `true` by default; setting it to `false` emits `workbox.setConfig({debug: false})` in the service worker, which silences the logs and switches to the production builds.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            config:
                debug: false
```
{% endcode %}

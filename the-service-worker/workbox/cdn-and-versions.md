# Workbox Files

The bundle ships the Workbox libraries and serves them from your own server: no third-party request, and a Content Security Policy that needs no exception.

**The bundled version is `7.4.1`.** It is copied to your public folder by `php bin/console pwa:compile`, and there is nothing to configure.

## Choosing the version is deprecated

{% hint style="warning" %}
`use_cdn` and `version` are deprecated since 1.6.0 and will be removed in 2.0.0.
{% endhint %}

Everything the bundle generates — the imports, the caching strategies, the plugins, the helpers — is written against the API of the Workbox version it ships. Pinning another version, or fetching one from a CDN, produced a service worker written for one version and running against another. It failed in the browser rather than at compile time, which is the worst place to find out.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            config:
                use_cdn: true       # deprecated since 1.6.0
                version: '7.3.0'    # deprecated since 1.6.0
```
{% endcode %}

**What to do:** remove both options. The bundled files are what the generated service worker expects.

{% hint style="info" %}
Both spellings are deprecated: the flat `workbox.use_cdn` and `workbox.version`, deprecated since 1.5.0, and their `workbox.config.*` counterparts. An application still using the flat ones is told twice — once for the option it wrote, once for the section the value is migrated into. Removing the option silences both.
{% endhint %}

### Cleaning up

Switching back from the CDN leaves nothing behind, but switching version does: `pwa:compile` writes the new files without removing the folder of the old one.

{% code title="terminal" %}
```bash
rm -rf public/workbox-v7.0.0 public/workbox-v7.3.0
php bin/console pwa:compile
```
{% endcode %}

## Where the files are served from

The libraries are written under `workbox_public_url`, which is **not** deprecated — where your own server exposes them is yours to decide.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            config:
                workbox_public_url: '/assets/workbox'
```
{% endcode %}

{% hint style="info" %}
`workbox_public_url` moved under `workbox.config` in 1.5.0. Its old location still works but is deprecated, and will be removed in 2.0.0.
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

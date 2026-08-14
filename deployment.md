# Deployment

As this bundle leverages Asset Mapper, the resources are served using this great component.

In the `dev` environment, the resources are automatically handled and returned by your Symfony app.

For the `prod` environment, before deploy, you should run:

```shell
symfony console asset-map:compile
```

This command also compiles the PWA files. If you disabled that integration with `asset_compiler: false`, run `pwa:compile` as well:

```shell
symfony console pwa:compile
```

## Applications Served From A Sub-Directory

If your application is not served from the root of its domain, e.g. `https://example.com/my-project/`, all the URLs the bundle generates — favicons, manifest, service worker, Workbox scripts, precached assets — are prefixed with the base path, exactly like the URLs returned by the `asset()` Twig function.

When a page is rendered, the base path is read from the current request and there is nothing to configure.

Compilation is different: it happens on the command line, where no request exists. The base path is then taken from the router request context, so `default_uri` must be set:

{% code title="/config/packages/routing.yaml" lineNumbers="true" %}
```yaml
framework:
    router:
        default_uri: 'https://example.com/my-project'
```
{% endcode %}

Without it, the compiled files point at the root of the domain and return a 404. The most visible symptom is the service worker failing to start, because its `importScripts()` calls resolve to `/workbox/workbox-sw.js` instead of `/my-project/workbox/workbox-sw.js`.

Should you need a base path for the assets different from the one of the router, override the `asset.request_context.base_path` parameter instead:

{% code title="/config/services.yaml" lineNumbers="true" %}
```yaml
parameters:
    asset.request_context.base_path: '/my-project'
```
{% endcode %}

{% hint style="warning" %}
The values you write yourself are never rewritten. The service worker [`scope`](the-service-worker/configuration.md), the `href` of a [resource hint](performance/resource-hints.md) and the [offline fallback](the-service-worker/workbox/offline-fallback.md) page are used as-is, so include the base path in them when your application needs one.
{% endhint %}

# Configuration

The Service Worker is a Javascript file you can declare in the configuration file. It will be automatically served by the bundle as long as your template file uses the [Twig `pwa` method](../installation.md).

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker: "sw.js"
```
{% endcode %}

The following example is exactly the same:

{% hint style="info" %}
`"sw.js"` is served by Asset Mapper and refers to the file in the /assets folder of your project. It can be stored elsewhere if needed.
{% endhint %}

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
```
{% endcode %}

By default, the public URL of the service worker will be /sw.js. You can change this URL using the dest configuration option.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        dest: "/foo/service-worker.js"
```
{% endcode %}

### Service Worker Initialization

The Service Worker initialization script uses either Workbox Window if enabled or a smiliar Vanilla JS script.

If Workbox is enabled, the initialization script is loaded from a distant URL. We recommend to install it using Asset Mapper and avoid distant loading.

```sh
symfony console importmap:require workbox-window
```

The two other options are also available:

### Other Options

The Service worker section has other options you may be interested in.

#### Scope

The `scope` parameter defaults to `/`. It is a string representing the service worker's registration scope. It should be aligned with the [Manifest scope](../the-manifest/application-information/scope.md).

#### Cache

The `use_cache` parameter is enable by default. It is a boolean that sets how the HTTP cache is used for service worker script resources during updates.

#### Skip Waiting

The `skip_waiting` parameter is disabled by default. It ensures that any new versions of a service worker will take over the page and become activated immediately. It is safe in general, but may create issues when the old service worker is handling events while it is updated.

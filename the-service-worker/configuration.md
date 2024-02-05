# Configuration

The Service Worker is a Javascript file you can declare in the configuration file. It will be automatically served by the bundle as long as your template file uses the [Twig `pwa` method](../installation.md).

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker: "sw.js"
```
{% endcode %}

The following example is exactly the same:

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

The two other options are also available:

* `scope` (default to `/`): a string representing the service worker's registration scope.
* use\_cache (default to `true`): a boolean that sets how the HTTP cache is used for service worker script resources during updates.

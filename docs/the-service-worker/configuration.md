# Configuration

The Service Worker is a Javascript file you can declare in the configuration file. It will be automatically served by the bundle as long as your template file uses the [Twig `pwa` method](../installation.md).

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker: "sw.js"
```
{% endcode %}

`"sw.js"` is served by Asset Mapper and refers to the file in `/assets/sw.js` folder of your project. It can be stored elsewhere if needed.

{% hint style="info" %}
To start, just put an empty file. It will be automatically populated by the bundle and will evolves depending on your application needs.
{% endhint %}

The following example is exactly the same:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
```
{% endcode %}

{% hint style="warning" %}
As it has an impact on the Twig pages, you may need to clear the cache when the service worker is enabled.
{% endhint %}

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

When Workbox is enabled, its initialization script typically loads from an external URL. However, for improved performance and security, we advise installing it via Asset Mapper instead of relying on remote loading.

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

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        dest: "/foo/service-worker.js"
        scope: "/"
        use_cache: true
        skip_waiting: false
```
{% endcode %}

### Compiling the Service Worker

The service worker is compiled using the `pwa:compile` command:

```sh
php bin/console pwa:compile
```

**Options:**
- `--context-only` - Compile only assets that depend on the application context (e.g., the manifest). Useful for locale-specific compilations.
- `--no-screenshots` - Skip screenshot generation during compilation.

### Background Fetch

The Background Fetch API allows your PWA to handle long-running downloads in the background. Configure it in the workbox section:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            background_fetch:
                enabled: true
                success_url: 'app_download_success'  # Route name or URL
                progress_url: 'app_download_progress'
                success_message: 'Download complete!'
                failure_message: 'Download failed.'
                db_name: 'bgfetch-completed'
```
{% endcode %}

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `success_url` | URL/route | - | URL to open when download completes |
| `progress_url` | URL/route | - | URL to display download progress |
| `success_message` | string | `null` | Success notification message (translatable) |
| `failure_message` | string | `null` | Failure notification message (translatable) |
| `db_name` | string | `bgfetch-completed` | IndexedDB name for download storage |

{% hint style="info" %}
Background Fetch is supported in Chromium-based browsers. It handles downloads that would otherwise be interrupted when the user closes the tab.
{% endhint %}

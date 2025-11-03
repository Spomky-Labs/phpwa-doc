# Configuration

The Service Worker is a JavaScript file that acts as a proxy between your web application and the network. It enables advanced features like offline functionality, background sync, and push notifications.

The PWA Bundle automatically generates and serves your service worker based on your configuration. You simply need to provide a source file (which can be empty to start) and configure the desired features.

## Basic Configuration

The simplest configuration requires only specifying the source file:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker: "sw.js"
```
{% endcode %}

This shorthand configuration:
- Enables the service worker
- Uses `sw.js` from your `/assets/` folder as the source
- Serves it publicly at `/sw.js`

{% hint style="info" %}
The `sw.js` file can start empty. The bundle will automatically populate it with functionality based on your configuration (Workbox, caching strategies, etc.).
{% endhint %}

### Expanded Configuration

The shorthand above is equivalent to:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        dest: "/sw.js"  # Public URL (default)
        scope: "/"      # Service worker scope (default)
```
{% endcode %}

{% hint style="warning" %}
Enabling or disabling the service worker affects Twig page rendering. Clear your cache after changing the `enabled` setting:
```bash
php bin/console cache:clear
```
{% endhint %}

## Configuration Options

### Enabled

**Type**: `boolean`
**Default**: `false` (or `true` when using shorthand)

Controls whether the service worker is active.

```yaml
pwa:
    serviceworker:
        enabled: true
```

### Source (src)

**Type**: `string`
**Required**: Yes

Path to your source service worker file. This file is processed by Asset Mapper and can contain custom JavaScript that will be merged with the bundle's generated code.

```yaml
pwa:
    serviceworker:
        src: "sw.js"              # From /assets/sw.js
        # or
        src: "custom/my-sw.js"    # From /assets/custom/my-sw.js
```

The source file can:
- Be completely empty (bundle generates everything)
- Contain custom service worker logic
- Import additional modules

### Destination (dest)

**Type**: `string`
**Default**: `"/sw.js"`

Public URL where the compiled service worker will be accessible. This is the URL browsers will use to register the service worker.

```yaml
pwa:
    serviceworker:
        dest: "/service-worker.js"
        # or
        dest: "/assets/sw.js"
```

{% hint style="info" %}
Service workers must be served from the same origin as your application. The bundle handles this automatically.
{% endhint %}

### Scope

**Type**: `string`
**Default**: `"/"`

Defines the service worker's registration scope - the subset of pages it can control. Only pages within this scope can use the service worker.

```yaml
pwa:
    serviceworker:
        scope: "/"           # Controls all pages
        # or
        scope: "/app/"       # Controls only pages under /app/
```

**Important**: The scope should match your [manifest scope](../the-manifest/application-information/scope.md) to ensure consistent PWA behavior.

**Scope Rules**:
- Service workers can only control pages at their scope level or deeper
- A service worker at `/app/` cannot control pages at `/admin/`
- Use `/` for a global service worker
- The scope must be at or above the service worker's location

### Use Cache

**Type**: `boolean`
**Default**: `true`

Controls how the browser's HTTP cache is used when checking for service worker updates.

```yaml
pwa:
    serviceworker:
        use_cache: true   # Use HTTP cache (default)
        # or
        use_cache: false  # Bypass cache on updates
```

**When `true`**:
- Browser may use cached version during update checks
- Faster checks, but updates may be delayed
- Respects standard HTTP caching headers

**When `false`**:
- Browser bypasses cache during update checks
- Ensures latest version is always fetched
- May increase server load

### Skip Waiting

**Type**: `boolean`
**Default**: `false`

Controls whether a new service worker version activates immediately or waits for all tabs using the old version to close.

```yaml
pwa:
    serviceworker:
        skip_waiting: false  # Wait for tab closure (default)
        # or
        skip_waiting: true   # Activate immediately
```

**When `false` (default - recommended)**:
- New service worker waits until all tabs close
- Prevents version conflicts
- Ensures consistent experience across tabs
- Safer for most applications

**When `true`**:
- New service worker activates immediately
- May cause version mismatches between tabs
- Old tabs might have old assets, new tabs new assets
- Can break applications expecting consistent versions

{% hint style="warning" %}
Enabling `skip_waiting: true` can cause issues if your application assumes all tabs use the same version. Use with caution and thoroughly test your application's behavior.
{% endhint %}

**Use case for `skip_waiting: true`**:
- Critical bug fixes that must deploy immediately
- Applications designed to handle version differences
- Single-page apps that can gracefully reload

## Complete Configuration Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        dest: "/service-worker.js"
        scope: "/"
        use_cache: true
        skip_waiting: false

        # Workbox configuration (see Workbox documentation)
        workbox:
            enabled: true
            # ... workbox options
```
{% endcode %}

## Service Worker Initialization

The service worker is automatically registered when you include the PWA Twig function in your template:

{% code title="templates/base.html.twig" lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html>
<head>
    {{ pwa() }}
</head>
<body>
    ...
</body>
</html>
```
{% endcode %}

The registration script:
- Uses Workbox Window if Workbox is enabled
- Falls back to vanilla JavaScript registration otherwise
- Handles update checks automatically
- Respects your configuration (scope, use_cache, skip_waiting)

### Manual Service Worker Updates

For advanced control over service worker updates, use the [Service Worker Stimulus Controller](../symfony-ux/service-worker.md):

```twig
<div {{ stimulus_controller('@pwa/service-worker') }}>
    <button data-action="pwa--service-worker#update">
        Check for updates
    </button>
</div>
```

## Next Steps

- **Configure Workbox**: See [Workbox documentation](workbox/README.md) for caching strategies
- **Custom Rules**: Create [custom service worker rules](custom-service-worker-rule.md) for advanced functionality
- **Cache Strategies**: Implement [custom cache strategies](custom-cache-strategies.md) for specific needs
- **Security**: Configure [Content Security Policy](content-security-policy.md) if needed

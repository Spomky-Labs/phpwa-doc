# From 1.4.x to 1.5.0

This page describes the changes you need to make when upgrading from version 1.4.x to 1.5.0.

## Breaking Changes

### Default Caching Strategy Changed

The default resource cache strategy has been changed from `NetworkFirst` to `StaleWhileRevalidate`.

**Impact**: Pages will now load instantly from cache while updating in the background, instead of waiting for the network first.

**If you rely on NetworkFirst behavior**, explicitly set it in your configuration:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            resource_caches:
                - match_callback: 'navigate'
                  strategy: 'NetworkFirst'
```
{% endcode %}

### Broadcast Default Changed

The `broadcast` option for `StaleWhileRevalidate` resource caches now defaults to `true` (was `false`).

**Impact**: Clients will be notified via BroadcastChannel when cached content is updated in the background. This is generally desirable for StaleWhileRevalidate but may trigger unexpected behavior if you listen to broadcast events.

## Deprecated Configuration Options

### Workbox Configuration Restructuring

Three Workbox configuration options have been moved under a new `config` node. The old options still work but trigger deprecation warnings and will be removed in 2.0.0.

{% code title="Before (1.4.x)" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            use_cdn: false
            version: '7.3.0'
            workbox_public_url: '/workbox'
```
{% endcode %}

{% code title="After (1.5.x)" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            config:
                use_cdn: false
                version: '7.3.0'
                workbox_public_url: '/workbox'
```
{% endcode %}

| Old Option | New Option | Since |
|------------|-----------|-------|
| `pwa.serviceworker.workbox.use_cdn` | `pwa.serviceworker.workbox.config.use_cdn` | 1.5.0 |
| `pwa.serviceworker.workbox.version` | `pwa.serviceworker.workbox.config.version` | 1.5.0 |
| `pwa.serviceworker.workbox.workbox_public_url` | `pwa.serviceworker.workbox.config.workbox_public_url` | 1.5.0 |

{% hint style="warning" %}
If you are reading this on your way to 1.6.0, do not move `use_cdn` and `version` into `config`: both are
deprecated there too since 1.6.0. Remove them instead — see
[From 1.5.x to 1.6.0](from-1.5.x-to-1.6.0.md). Only `workbox_public_url` is worth moving.
{% endhint %}

A new `debug` option has also been added under `config`:

```yaml
pwa:
    serviceworker:
        workbox:
            config:
                debug: true  # Controls Workbox debug logging (default: true)
```

## New Features

### Navigation Preload

Speeds up navigation requests by starting network requests in parallel with service worker boot-up.

```yaml
pwa:
    serviceworker:
        workbox:
            navigation_preload: true
```

{% hint style="warning" %}
Don't enable navigation preload if you precache HTML pages (via `offline_fallback` or `resource_caches` with `urls`), as the preloaded response would be ignored in favor of the cached version.
{% endhint %}

See [Navigation Preload](../the-service-worker/workbox/navigation-preload.md) for details.

### Resource Hints

Automatically inject `<link>` tags for preconnect, dns-prefetch, and preload resources.

```yaml
pwa:
    resource_hints:
        enabled: true
        auto_preconnect: true
        preconnect:
            - https://api.example.com
        dns_prefetch:
            - https://analytics.example.com
        preload:
            - href: /fonts/custom.woff2
              as: font
              type: font/woff2
              crossorigin: anonymous
```

See [Resource Hints](../performance/resource-hints.md) for details.

### Early Hints (HTTP 103)

Send HTTP 103 Early Hints responses for faster resource loading on supported servers (FrankenPHP, Caddy, etc.).

```yaml
pwa:
    early_hints:
        enabled: true
        preload_manifest: true
        preload_serviceworker: false
        preconnect_workbox_cdn: true
```

See [Early Hints](../performance/early-hints.md) for details.

### Speculation Rules

Configure the browser's Speculation Rules API for prefetching and prerendering pages.

```yaml
pwa:
    speculation_rules:
        enabled: true
        prefetch:
            - source: document
              eagerness: moderate
              selector_matches: 'a.prefetch'
        prerender:
            - source: list
              urls:
                  - /checkout
              eagerness: conservative
```

See [Speculation Rules](../performance/speculation-rules.md) for details.

### Screenshot Generation with PHP Attribute

Generate screenshots automatically from your routes using the `#[Screenshot]` PHP attribute:

```php
use SpomkyLabs\PwaBundle\Attribute\Screenshot;

#[Screenshot(
    sizes: ['fhd', 'ipad/LP', '1920x1080'],
    parameters: ['id' => 123],
    locales: ['fr', 'en'],
    platform: 'android',
)]
public function index(): Response
{
    // ...
}
```

See [Screenshots](../image-management/screenshots.md) for details.

### Mobile Web App Capable Meta Tag

The bundle now automatically injects the `<meta name="mobile-web-app-capable" content="yes">` tag when the manifest `display` mode is set to `standalone`, `fullscreen`, or `minimal-ui`. This improves iOS compatibility.

### Cache Strategy Listing Command

A new console command lists all registered cache strategies:

```bash
php bin/console pwa:cache:list
```

### PreloadUrl Attribute

The `#[PreloadUrl]` attribute allows you to declare URLs that should be precached by the service worker:

```php
use SpomkyLabs\PwaBundle\Attribute\PreloadUrl;

#[PreloadUrl(alias: 'critical-pages')]
#[Route('/dashboard', name: 'app_dashboard')]
public function dashboard(): Response
{
    // This URL will be precached by the service worker
}
```

See the [PreloadUrl documentation](../the-service-worker/workbox/resource-caching.md) for details.

## Migration Checklist

- [ ] Move `use_cdn`, `version`, and `workbox_public_url` under the `config` node
- [ ] Review default caching strategy change (NetworkFirst → StaleWhileRevalidate)
- [ ] Review broadcast default change (false → true for StaleWhileRevalidate)
- [ ] Clear Symfony cache after configuration changes
- [ ] Test deprecation warnings with `APP_ENV=dev`
- [ ] Evaluate new features: Navigation Preload, Resource Hints, Speculation Rules
- [ ] Run `php bin/console pwa:compile` to regenerate assets

## Still Deprecated (from earlier versions)

The following options were deprecated in earlier versions and will also be removed in 2.0.0:

| Option | Replacement | Since |
|--------|-------------|-------|
| `pwa.favicons.src` | `pwa.favicons.default.src` | 1.3.0 |
| `pwa.favicons.src_dark` | `pwa.favicons.dark.src` | 1.3.0 |
| `pwa.favicons.background_color` | `pwa.favicons.default.background_color` | 1.3.0 |
| `pwa.favicons.background_color_dark` | `pwa.favicons.dark.background_color` | 1.3.0 |
| `pwa.favicons.border_radius` | `pwa.favicons.default.border_radius` | 1.3.0 |
| `pwa.favicons.image_scale` | `pwa.favicons.default.image_scale` | 1.3.0 |
| `pwa.path_type_reference` | URL node `path_type_reference` parameter | 1.1.0 |

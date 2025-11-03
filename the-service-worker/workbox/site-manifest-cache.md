# Site Manifest Cache

{% hint style="warning" %}
**Deprecated Feature**: As of version 1.4, the `cache_manifest` configuration option is deprecated and no longer has any effect. The manifest caching strategy was removed to simplify the Workbox configuration. This page is maintained for reference purposes only.
{% endhint %}

## Background

The Site Manifest file (`manifest.json`) is automatically served by your application and is typically very small (a few kilobytes). Previously, the bundle provided an option to cache the manifest file using Workbox.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            cache_manifest: true  # DEPRECATED - No longer functional
```
{% endcode %}

## Why Was It Removed?

The manifest caching feature was removed in version 1.4 for several reasons:

1. **Minimal benefit**: The manifest file is small and loads quickly even without caching
2. **Browser caching**: Browsers already cache the manifest file using standard HTTP caching
3. **Rare access**: The manifest is only accessed during PWA installation and updates
4. **Console warnings**: The main purpose was to avoid console warnings, which are now handled differently
5. **Simplification**: Removing unnecessary features makes the bundle easier to maintain

## What It Used To Do

When enabled, this option would:
- Create a Workbox caching strategy for the manifest URL
- Use the StaleWhileRevalidate strategy
- Cache the manifest file in the service worker cache
- Serve cached manifest on subsequent loads

## Alternatives

If you need to cache your manifest file, you have several options:

### 1. Standard HTTP Caching (Recommended)

Configure your web server to cache the manifest file using HTTP headers:

**Nginx**:
```nginx
location /manifest.json {
    add_header Cache-Control "public, max-age=3600";
}
```

**Apache (.htaccess)**:
```apache
<Files "manifest.json">
    Header set Cache-Control "public, max-age=3600"
</Files>
```

### 2. Custom Resource Cache

Add the manifest to a custom resource cache strategy:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            resource_caches:
                - match_callback: '({url}) => url.pathname === "/manifest.json"'
                  strategy: 'StaleWhileRevalidate'
                  cache_name: 'manifest'
                  max_age: 1 hour
```
{% endcode %}

### 3. Browser Default Behavior

Simply rely on browser caching - modern browsers efficiently cache small JSON files like manifests without requiring service worker intervention.

## Migration Guide

If your configuration includes `cache_manifest`, you can:

1. **Remove the option** - It has no effect, but removing it keeps your configuration clean:
   ```yaml
   pwa:
       serviceworker:
           workbox:
               # cache_manifest: true  ← Remove this line
   ```

2. **Keep the option** - It won't cause errors, but it won't do anything either

3. **Use HTTP caching** - Configure your web server for better performance (recommended)

{% hint style="info" %}
**No Action Required**: Removing this option from your configuration has no impact on your PWA functionality. The manifest will continue to work exactly as before.
{% endhint %}

## Related Documentation

- [Resource Caching](resource-caching.md) - Custom caching strategies for any resource
- [Cache Management](cache-names-and-purge.md) - Managing service worker caches
- [Configuration](../configuration.md) - Complete service worker configuration reference

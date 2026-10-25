# CDN and Versions

{% hint style="info" %}
**Note**: This page covers advanced Workbox configuration. Most applications should use the [default configuration documented in the Workbox README](README.md#cdn-vs-local-installation).
{% endhint %}

## Default Configuration

The bundle ships with Workbox **7.3.0** (released October 2024) and serves all files locally from your application server. This provides:
- Faster loading (same-origin)
- Offline availability immediately
- No external dependencies
- Better privacy
- Controlled updates

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            use_cdn: false  # Default
            version: "7.3.0"  # Default
```
{% endcode %}

## Available Versions

The bundle includes these Workbox versions locally:
- **7.3.0** (default, recommended)
- **7.0.0** (for compatibility if needed)

## Using a CDN

For quick prototyping or bandwidth considerations, you can load Workbox from a CDN:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            use_cdn: true
            version: "7.3.0"
```
{% endcode %}

When `use_cdn: true`, Workbox files are loaded from `https://storage.googleapis.com/workbox-cdn/releases/{version}/`.

### CDN Pros

- **No local storage**: Workbox files not bundled with your app
- **Shared cache**: Users may already have Workbox cached from other sites
- **Always latest**: Can easily switch versions
- **Reduced deploy size**: Slightly smaller deployment

### CDN Cons

- **External dependency**: Requires Google's CDN to be available
- **Network request**: Adds latency on first load
- **Privacy**: Third-party request (though Google doesn't track via Workbox CDN)
- **Offline delay**: Service worker can't work until Workbox loads
- **Version risk**: CDN outages affect your app

{% hint style="warning" %}
**Production Recommendation**: Use local installation (`use_cdn: false`) for production applications. The benefits of local hosting far outweigh the small bandwidth savings of a CDN.
{% endhint %}

## Custom Versions

You can specify any Workbox version when using the CDN:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            use_cdn: true
            version: "6.5.4"  # Older version
```
{% endcode %}

### Version Selection Guidelines

**Use 7.3.0 (default)** when:
- Starting a new project
- No specific compatibility requirements
- Want the latest features and fixes

**Use 7.0.0** when:
- Need stability over new features
- Specific compatibility requirements
- Team has testing/docs for 7.0.0

**Use 6.x** when:
- Legacy project already using Workbox 6.x
- Breaking changes in 7.x affect your application
- Need to maintain compatibility with older code

{% hint style="danger" %}
**Major Version Upgrades**: Workbox 7.x includes breaking changes from 6.x. Review the [Workbox migration guide](https://developer.chrome.com/docs/workbox/migration/migrate-from-v6/) before upgrading.
{% endhint %}

## Complete Configuration Examples

### Local Installation (Recommended)

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: true
            use_cdn: false          # Local files (default)
            version: "7.3.0"        # Latest bundled version
            workbox_public_url: "/workbox"  # Where Workbox is served
            idb_public_url: "/idb"          # Where IndexedDB lib is served
```
{% endcode %}

### CDN Installation

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: true
            use_cdn: true           # Load from Google CDN
            version: "7.3.0"        # Version to load
```
{% endcode %}

### Legacy Version with CDN

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            enabled: true
            use_cdn: true
            version: "6.5.4"        # Older version for compatibility
```
{% endcode %}

## Custom Public URLs (Local Only)

When using local installation, customize where Workbox files are served:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            use_cdn: false
            workbox_public_url: "/assets/workbox"
            idb_public_url: "/assets/idb"
```
{% endcode %}

This is useful when:
- Organizing assets by type
- Complying with specific URL structures
- Integrating with CDN configurations

## Checking Your Configuration

To verify which Workbox version and source is being used:

1. Open DevTools (F12)
2. Go to **Application** → **Service Workers**
3. View service worker source
4. Look for `importScripts()` calls at the top

**Local installation**:
```javascript
importScripts('/workbox/workbox-sw.js');
```

**CDN installation**:
```javascript
importScripts('https://storage.googleapis.com/workbox-cdn/releases/7.3.0/workbox-sw.js');
```

## Version Update Process

When updating Workbox versions:

1. **Review changelog**: Check [Workbox releases](https://github.com/GoogleChrome/workbox/releases) for breaking changes
2. **Update configuration**: Change `version` in `pwa.yaml`
3. **Clear caches**: Clear browser caches during testing
4. **Test thoroughly**: Test all caching strategies and offline functionality
5. **Monitor**: Watch for errors after deployment

## Troubleshooting

**Workbox not loading?**
- Check CDN availability if `use_cdn: true`
- Verify `version` is correct and available
- Check browser console for import errors
- Ensure service worker is registered

**Using CDN but want to switch to local?**
```yaml
workbox:
    use_cdn: false  # Switch to local
    version: "7.3.0"
```
Then clear caches and re-register service worker.

**Using local but Workbox not found?**
- Verify bundle provides the version you specified
- Check `workbox_public_url` is accessible
- Look for 404 errors in Network tab

{% hint style="success" %}
**Best Practice**: Start with local installation using the default version. Only use CDN for quick prototyping or specific use cases where bandwidth is critical and reliability of an external CDN is acceptable.
{% endhint %}

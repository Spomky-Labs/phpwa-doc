# Installation

To install the bundle, the recommended way is using Composer:

```sh
composer require spomky-labs/pwa-bundle
```

No Flex recipe exists for this bundle, but you can create a configuration file that will be modified as you progress in the integration of the features.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa: ~
```
{% endcode %}

The integration in your application is very simple. You are only required to add a Twig function inside the end of the `<head>` tag of your HTML pages.

{% code lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html lang="en">
<head>
  {{ pwa() }}
</head>
<body>
  ...
</body>
</html>
```
{% endcode %}

{% hint style="info" %}
You may need to clear the cache after the bundle is installed
{% endhint %}

## Twig Function Options

The `pwa()` Twig function accepts several optional parameters to control what is injected:

```twig
{{ pwa(
    injectThemeColor: true,
    injectFavicons: true,
    injectSW: true,
    swAttributes: {},
    locale: null,
    injectResourceHints: true,
    injectSpeculationRules: true
) }}
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `injectThemeColor` | bool | `true` | Inject theme color meta tags |
| `injectFavicons` | bool | `true` | Inject favicon link tags |
| `injectSW` | bool | `true` | Inject service worker registration script |
| `swAttributes` | object | `{}` | Custom attributes for the service worker script tag |
| `locale` | string | `null` | Locale for manifest (auto-detected from request if null) |
| `injectResourceHints` | bool | `true` | Inject [resource hints](performance/resource-hints.md) (preconnect, dns-prefetch, preload) |
| `injectSpeculationRules` | bool | `true` | Inject [speculation rules](performance/speculation-rules.md) for prefetch/prerender |

### Example: Disable Resource Hints

```twig
{{ pwa(injectResourceHints: false) }}
```

### Example: Localized Manifest

```twig
{{ pwa(locale: app.request.locale) }}
```

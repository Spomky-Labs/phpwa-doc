# Start URL

The `start_url` parameter defines the URL that should be loaded when a user launches your PWA from their home screen or app launcher. This is typically the main entry point of your application.

## Basic Usage

The simplest way to define a start URL is to provide a Symfony route name:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        start_url: "app_homepage"
```
{% endcode %}

You can also use a simple path string:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        start_url: "/"
```
{% endcode %}

## Advanced Configuration

For more complex scenarios, you can use the full URL object configuration that allows you to specify route parameters and path types:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        start_url:
            path: "app_dashboard"
            params:
                utm_source: "pwa"
                utm_medium: "homescreen"
            path_type_reference: 1
```
{% endcode %}

### Path Parameter

The `path` parameter can be:
* A Symfony route name (e.g., `"app_homepage"`)
* An absolute URL (e.g., `"https://example.com/app"`)
* A relative path (e.g., `"/app"`)
* A network path (e.g., `"//example.com/app"`)

### Params Parameter

The `params` parameter allows you to pass route parameters or query string parameters:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        start_url:
            path: "app_article"
            params:
                id: 123
                slug: "welcome"
```
{% endcode %}

This is particularly useful for:
* Tracking PWA installations with analytics parameters
* Pre-loading specific content when the app launches
* Passing initial state to your application

### Path Type Reference

The `path_type_reference` option controls how the URL is generated:

* `0`: **Absolute URL** - Generates a full URL with protocol and domain (e.g., `https://app.com/foo/bar`)
* `1`: **Absolute Path** (default) - Generates a path starting from root (e.g., `/foo/bar`)
* `2`: **Relative Path** - Generates a relative path (e.g., `../bar`)
* `3`: **Network Path** - Generates a protocol-relative URL (e.g., `//app.com/foo/bar`)

{% hint style="warning" %}
When using absolute URLs (`path_type_reference: 0`), make sure the [Symfony Router Request Context](https://symfony.com/doc/current/routing.html#generating-urls-in-commands) is properly configured. This is especially important when generating the manifest outside of HTTP requests (e.g., in console commands).
{% endhint %}

## Best Practices

1. **Keep it within scope**: The `start_url` must be within the application's `scope`.
2. **Use route names**: Prefer Symfony route names over hardcoded paths for better maintainability.
3. **Analytics tracking**: Add tracking parameters to distinguish PWA launches from regular visits:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        start_url:
            path: "app_homepage"
            params:
                utm_source: "pwa"
                utm_medium: "homescreen"
```
{% endcode %}

4. **Consistency**: The `start_url` should always resolve to the same logical page, even if the underlying route changes.

{% hint style="info" %}
The `start_url` is not translatable since version 1.3.0. If you need localized start URLs, use the `{locale}` placeholder in the manifest public URL configuration. See [Translations](../translations.md) for more details.
{% endhint %}

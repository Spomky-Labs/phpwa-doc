# ID

One of the parameters included in this file is the `id` property. This parameter is crucial as it uniquely identifies the PWA across browsers and devices, enabling a consistent user experience.

This `id` parameter should be consistent and not change, even if other manifest properties are updated. It's important for maintaining the application's identity for things like saved user preferences and home screen shortcuts.

When absent, an ID is determined using the `start_url` parameter, the manifest location and its `scope`. Adding an `id` to the manifest allows to change the `start_url` and the manifest path. Note that <mark style="color:red;">the domain shall not change</mark>.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        id: "/?homescreen=1"
```
{% endcode %}

## Why Use an ID?

The `id` parameter provides several benefits:

1. **Stability**: Allows you to change the `start_url` or manifest location without affecting the app's identity.
2. **User Data Preservation**: Ensures that user preferences, permissions, and cached data are maintained even if URLs change.
3. **Consistent Updates**: Helps the browser recognize that a new manifest belongs to the same application.

## When to Set an ID

You should explicitly set an `id` when:

* Your application's `start_url` might change in the future
* You plan to move the manifest file to a different location
* You want explicit control over your app's identity
* You support multiple entry points but want them treated as the same app

## ID Format

The `id` should be a string that:

* Is relative to the manifest's origin (domain must remain the same)
* Is unique within your domain
* Remains constant throughout your app's lifetime

Common patterns include:
* `"/?homescreen=1"` - Query parameter approach
* `"/app"` - Path-based approach
* `"/"` - Root-level identification

{% hint style="warning" %}
Once you've set an `id` and users have installed your PWA, changing it will cause browsers to treat it as a completely different application. This means users would need to reinstall the app, and all saved data would be lost.
{% endhint %}

## Example with Routing

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        id: "/?homescreen=1"
        start_url: "app_homepage"
        scope: "/"
```
{% endcode %}

In this example, even if the `app_homepage` route URL changes, the application identity remains stable thanks to the explicit `id`.

{% hint style="info" %}
If you don't specify an `id`, browsers will automatically compute one based on the `start_url`, manifest location, and `scope`. This works fine for most applications, but explicit IDs provide more control and flexibility.
{% endhint %}

# Scope

The `scope` parameter in a Progressive Web App (PWA) manifest file is an important feature as it defines the set of URLs that the browser considers to be within your app. When a user navigates to a URL within the scope, they are kept within the full app experience.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        scope: "/"
```
{% endcode %}

#### Defining Scope:

* The scope can be a relative path or an absolute URL.
* If omitted, it defaults to the location of the manifest.
* The `start_url` must be within the scope.

#### Best Practices:

1. Keep the scope as restrictive as possible to maintain control over your app's user experience.
2. Set the `start_url` to a URL within the app's scope so it starts in the right context.
3. Test the scope to ensure it includes all the necessary pages and excludes any that should not be part of the PWA experience.

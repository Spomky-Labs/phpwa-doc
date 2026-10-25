# ID

One of the parameters included in this file is the `id` property. This parameter is crucial as it uniquely identifies the PWA across browsers and devices, enabling a consistent user experience.

This `id` parameter should be consistent and not change, even if other manifest properties are updated. It's important for maintaining the application's identity for things like saved user preferences and home screen shortcuts.

When absent, an ID is determined using the `start_url` parameter, the manifest location and its `scope`. Adding an `id` to the manifest allows to change the `start_url` and the manifest path. Note that **the domain shall not change**.

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        id: "/?homescreen=1"
```

!!! info

    To be written

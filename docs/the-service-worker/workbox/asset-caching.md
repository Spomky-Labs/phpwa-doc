# Asset caching

!!! danger

    Contains a breaking change compared to v1.0.x

The cache is populated with all assets managed by Asset Mapper. By default, it includes all types of well recognized assets such as images, stylesheets or scripts:

```regex
/\.(css|js|json|xml|txt|map|ico|png|jpe?g|gif|svg|webp|bmp)$/
```

This can be changed using the next configuration options:

```yaml title="/config/packages/pwa.yaml" linenums="1"
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            asset_cache:
                enabled: true
                regex: '/\.(css|jsx?)$/'
```

# Image caching

By default, a maximum of 60 images are cached for 1 year. The supported image extensions extensions are as follows:&#x20;

```regex
/\.(ico|png|jpe?g|gif|svg|webp|bmp)$/
```

This can be changed using the next configuration options

```yaml title="/config/packages/pwa.yaml" linenums="1" hl_lines="2"
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            image_cache:
                enabled: true
                max_age: 30 days
                max_entries: 200
                regex: '/\.(png|jpe?g|svg|webp)$/'
```

!!! info

    This cache is different from the Asset one as it corresponds to images that are not managed by Asset Mapper

# Complete Example

```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        skip_waiting: true
        workbox:
            page_fallback: 'app_homepage'
            warm_cache_urls:
                - 'app_homepage'
                - 'app_feature1'
                - 'app_feature2'
                - 'app_feature3'
```

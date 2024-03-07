# BackgoundSync

Background Sync is a feature provided by service workers that enables your Progressive Web App to defer actions until the user has stable connectivity. This is particularly useful for ensuring any requests or data submitted by the user are not lost if their internet connection is unreliable.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            background_sync:
                - queue_name: 'api'
                  regex: /\/api\// # All requests starting with /api/
                  method: POST
                  max_retention_time: 7_200 # 5 days in minutes
                  force_sync_fallback: true #Optional
                - queue_name: 'contact'
                  regex: /\/contact-form\// # All requests starting with /contact-form/
                  method: POST
                  max_retention_time: 120 # 2 hours in minutes
```
{% endcode %}

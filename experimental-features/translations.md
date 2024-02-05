# Translations

The bundle leverages on Symfony Translation component is available. The texts you pass for almost all names, short names, descriptions, labels... can be translation keys.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "app.name"
        short_name: "app.short_name"
        start_url: "/index.html"
        displa": "standalone"
        background_color: "#ffffff"
        theme_color: "#4285f4"
        shortcuts:
            - name: "app.feature1.shorcut.name"
              short_name: "app.feature1.shorcut.short_name"
              description: "app.feature1.shorcut.description"
              url: "/start-chat"
              icons":          
                - src: "icons/feature1-96x96.png"
                  sizes: [96]
```
{% endcode %}

{% hint style="warning" %}
This feature is in early development stage and may not work as expected. Use with caution
{% endhint %}

# Translations

The bundle can leverage the Symfony Translation component. The text you provide for almost all names, abbreviations, descriptions, labels, etc., can be used as translation keys.

These translations must be placed in a `pwa` domain (e.g. `pwa.en.yaml` or `pwa.fr.yaml`)

```yaml title="/config/packages/pwa.yaml" linenums="1"
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

```yaml title="/translations/pwa.en.yaml" linenums="1"
app:
    name: "My App"
    short_name: "My App"
    feature1:
        shortcut:
            name: "Start chat"
            short_name: "Start chat"
            description: "Start a chat with your friends"
```

!!! warning

    This feature is in early development stage and may not work as expected. Use with caution

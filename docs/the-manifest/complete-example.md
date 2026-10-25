# Complete Example

Here's a comprehensive manifest configuration showing all available features:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: 'My Awesome Application'
        short_name: 'awesome-app'
        description: 'This application will help you to change the world'
        id: "/"
        start_url:
            path: "app_homepage"
            params:
                utm_source: "pwa"
        scope: "/"
        orientation: "any"
        display: "standalone"
        display_override: ['fullscreen', 'minimal-ui', 'window-controls-overlay']
        background_color: "#ffffff"
        theme_color: "#212529"
        dark_theme_color: "#1a1a2e"
        use_credentials: true
        handle_links: "preferred"
        categories: ['utility', 'productivity']
        icons:
            - src: "images/favicon.ico"
              sizes: [48]
            - src: "images/favicon-512x512.png"
              sizes: [512]
            - src: "images/favicon.svg"
              sizes: [0]
            - src: "images/favicon.svg"
              purpose: 'maskable'
              sizes: [0]
        screenshots:
            - "images/screenshots/homepage-1303x1718.png"
            - src: "images/screenshots/feature1-1303x1718.png"
              label: "Feature 1 in action"
            - src: "images/screenshots/feature2-2056x1080.png"
              label: "Feature 2 and available options"
        shortcuts:
            - name: "Feature 1"
              short_name: "feature1"
              description: "See Feature #1 in live action"
              url: "app_feature1"
              icons:
                  - src: "images/feature1.svg"
                    sizes: [0]
                  - src: "images/feature1-96x96.png"
                    sizes: [96]
        scope_extensions:
            - origin: "*.example.com"
        note_taking:
            note_taking_url: "app_note_taking"
```
{% endcode %}

## Scope Extensions

The `scope_extensions` property allows your PWA to control multiple subdomains and top-level domains as a single entity:

```yaml
pwa:
    manifest:
        scope_extensions:
            - origin: "*.example.com"
            - origin: "*.example.co.uk"
```

## Note Taking

The `note_taking` property declares note-taking capabilities, allowing the application to be registered as a note-taking app:

```yaml
pwa:
    manifest:
        note_taking:
            note_taking_url: "app_notes"  # Symfony route name or URL
```

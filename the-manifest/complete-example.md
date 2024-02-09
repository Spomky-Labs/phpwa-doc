# Complete Example



```yaml
pwa:
    manifest:
        enabled: true
        background_color: "#ffffff"
        theme_color: "#212529"
        name: 'My Awesome Application'
        short_name: 'awesome-app'
        id: '/?manifest=1'
        description: 'With application will help you to change the world'
        orientation: "any"
        display: "standalone"
        scope: "/"
        display_override: ['fullscreen', 'minimal-ui', 'window-controls-overlay']
        id: "/"
        start_url: "./"
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
            - src: "images/screenshots/feature3-2056x1080.png"
              label: "Feature 3 at its best"
        categories: ['utility', 'productivity']
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

```

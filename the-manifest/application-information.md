# Application Information

For a Progressive Web App (PWA) to be installable, the manifest file must include certain mandatory information. These details help the browser understand how the application should be presented to the user and its behavior upon installation.

While not all of these properties are strictly mandatory, a manifest must at least contain the minimum information for the browser to prompt the user to install the application. In practice, it is recommended to include as much information as possible to enhance the user experience.

Here are the key properties of the manifest that are typically considered necessary for PWA installation:

1. **`name`:** The name of the application.
2. **`short_name`:** A short or abbreviated name for the application (used when space is limited, e.g., on a home screen).
3. **`start_url`:** The relative or absolute URL from which the application should launch when opened.
4. **`display`:** Determines how the application should be displayed. Common values include `fullscreen`, `standalone`,`minimal-ui` and `browser`.
5. **`background_color`:** The background color of the application.
6. **`theme_color`:** The color representing the main theme of the application.
7. **`icons`:** An array of objects describing the application's icons for different sizes and resolutions. ([see after](icons.md))

Minimal example of a manifest:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "My PWA"
        short_name: "PWA"
        start_url: "/index.html"
        displa": "standalone"
        background_color: "#ffffff"
        theme_color: "#4285f4"
```
{% endcode %}

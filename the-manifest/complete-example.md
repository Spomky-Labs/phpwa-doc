# Complete Example

This page provides a comprehensive example of a PWA manifest configuration showcasing most of the available features in the Spomky-Labs PWA Bundle.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        # Basic Configuration
        enabled: true

        # Application Identity & Naming
        name: 'My Awesome Application'
        short_name: 'Awesome App'
        description: 'This application will help you to change the world'
        id: "/?manifest=1"

        # Visual Appearance
        background_color: "#ffffff"
        theme_color: "#2196f3"
        dark_theme_color: "#1565c0"

        # Display Configuration
        display: "standalone"
        display_override: ['window-controls-overlay', 'minimal-ui']
        orientation: "any"

        # Navigation & Scope
        scope: "/"
        start_url:
            path: "app_homepage"
            params:
                utm_source: "pwa"
                utm_medium: "homescreen"

        # Localization
        dir: "ltr"
        lang: "en"

        # Categorization
        categories: ['productivity', 'utilities']
        iarc_rating_id: "e84b072d-71b3-4d3e-86ae-31a8ce4e53b7"

        # Icons
        icons:
            # PNG icons with multiple sizes
            - src: "images/icon.png"
              sizes: [48, 72, 96, 144, 192, 256, 512]
              purpose: "any"

            # Maskable icon with background
            - src: "images/icon-maskable.png"
              sizes: [192, 512]
              purpose: "maskable"
              background_color: "#2196f3"
              image_scale: 80
              border_radius: 20

            # SVG icon
            - src: "images/icon.svg"
              sizes: [0]
              svg_attr:
                  fill: '#2196f3'
                  color: '#ffffff'

        # Screenshots
        screenshots:
            # Mobile screenshot (narrow)
            - src: "images/screenshots/mobile-home.png"
              label: "Home screen on mobile"
              form_factor: "narrow"
              platform: "android"

            - src: "images/screenshots/mobile-feature1.png"
              label: "Feature 1 in action"
              form_factor: "narrow"

            # Desktop/tablet screenshots (wide)
            - src: "images/screenshots/desktop-dashboard.png"
              label: "Dashboard view on desktop"
              form_factor: "wide"
              platform: "windows"

            - src: "images/screenshots/desktop-analytics.png"
              label: "Analytics and reporting"
              form_factor: "wide"

        # Shortcuts
        shortcuts:
            - name: "New Document"
              short_name: "New Doc"
              description: "Create a new document"
              url:
                  path: "app_document_new"
                  params:
                      utm_source: "shortcut"
              icons:
                  - src: "images/shortcuts/new-doc.svg"
                    sizes: [96]

            - name: "Recent Items"
              short_name: "Recent"
              description: "View recently accessed items"
              url: "app_recent"
              icons:
                  - src: "images/shortcuts/recent.svg"
                    sizes: [96]

            - name: "Settings"
              short_name: "Settings"
              description: "Open application settings"
              url: "app_settings"
              icons:
                  - src: "images/shortcuts/settings.svg"
                    sizes: [96]

        # Share Target - Allow receiving content from other apps
        share_target:
            action: "app_share_receiver"
            method: "POST"
            enctype: "multipart/form-data"
            params:
                title: "share_title"
                text: "share_text"
                url: "share_url"
                files:
                    - name: "photos"
                      accept: ["image/png", "image/jpeg", "image/webp"]
                    - name: "documents"
                      accept: ["application/pdf", "text/plain"]

        # File Handlers - Register to open specific file types
        file_handlers:
            - action: "app_file_handler_image"
              accept:
                  "image/png": [".png"]
                  "image/jpeg": [".jpg", ".jpeg"]
                  "image/webp": [".webp"]

            - action: "app_file_handler_document"
              accept:
                  "text/plain": [".txt"]
                  "text/markdown": [".md"]

        # Protocol Handlers - Handle custom URL schemes
        protocol_handlers:
            - protocol: "mailto"
              url: "app_mail_compose?to=%s"

            - protocol: "web+myapp"
              url:
                  path: "app_protocol_handler"
                  params:
                      action: "%s"

```
{% endcode %}

## Key Sections Explained

### Identity & Display
The manifest starts with basic identity information (`name`, `short_name`, `description`) and display preferences that control how your PWA appears when installed.

### Theme Colors
Both light and dark theme colors are specified to ensure the app looks great in both color schemes.

### Icons
Multiple icon configurations are provided:
- Regular PNG icons in multiple sizes for compatibility
- Maskable icons with padding and rounded corners for modern Android devices
- SVG icons for scalability

### Screenshots
Screenshots are organized by form factor (narrow for mobile, wide for desktop) and can be targeted to specific platforms.

### Shortcuts
App shortcuts provide quick access to key features directly from the OS launcher.

### Advanced Features
- **Share Target**: Allows your PWA to receive content from other apps
- **File Handlers**: Registers your PWA as a handler for specific file types
- **Protocol Handlers**: Enables your PWA to handle custom URL protocols

{% hint style="info" %}
This example shows many features together. In practice, you should only include the features that your application actually supports to keep your manifest file focused and maintainable.
{% endhint %}

{% hint style="warning" %}
Some features like File Handlers and Protocol Handlers have limited browser support. Always check compatibility and provide fallback options for users on unsupported browsers.
{% endhint %}

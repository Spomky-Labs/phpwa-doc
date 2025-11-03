# Colors and Theme

The PWA manifest allows you to customize the visual appearance of your application through color properties. These colors define how your app looks both when launching and while running.

## Theme Color

The `theme_color` property defines the default color of the application. This color is used by the browser to customize the color of the UI around your app, such as the address bar, the taskbar on desktop, or the status bar on mobile devices.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        theme_color: "#4285f4"
```
{% endcode %}

## Dark Theme Color

The `dark_theme_color` property specifies an alternative theme color to use when the user's device is in dark mode. This allows your PWA to automatically adapt to the user's system preferences.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        theme_color: "#4285f4"
        dark_theme_color: "#1a73e8"
```
{% endcode %}

When the user switches between light and dark modes, the browser will automatically use the appropriate theme color.

{% hint style="info" %}
If `dark_theme_color` is not specified, the browser will use `theme_color` for both light and dark modes.
{% endhint %}

## Background Color

The `background_color` property defines the placeholder background color shown before your application's stylesheet is loaded. It should match the background color of your app's main content area to create a smooth loading experience.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        background_color: "#ffffff"
```
{% endcode %}

## Best Practices

* **Consistency**: Choose colors that match your app's branding and design system.
* **Contrast**: Ensure sufficient contrast between text and background colors for accessibility.
* **Dark Mode**: When providing a `dark_theme_color`, make sure it works well with your app's dark mode styles.
* **Testing**: Test your color choices on different devices and operating systems to ensure they display correctly.

## Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        theme_color: "#4285f4"
        dark_theme_color: "#1a73e8"
        background_color: "#ffffff"
```
{% endcode %}

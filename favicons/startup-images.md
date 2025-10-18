# Startup Images

On Android, a startup image is created using the app icon and a background. On iOS, specific media query images are necessary.

When setting the icon or dark icon, startup images are automatically generated for both landscape and portrait orientations. To disable this feature, you can turn it off.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        use_start_image: false
```
{% endcode %}


# Create Favicons

Providing favicons for a large set of devices and operating systems is painful. Indeed, ensuring your website has a favicon that looks good and is recognizable across all devices and operating systems can be a daunting task.

{% hint style="info" %}
Favicons are different from application icons declared on the Manifest. This will be simplified in the next versions of the bundle.
{% endhint %}

With this bundle, it takes few seconds to get it working. All you need is a square icon as a SVG or a large PNG image (512 pixels or more).

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
```
{% endcode %}

{% hint style="info" %}
It is highly recommended to have a transparent background.
{% endhint %}

**Done!**

Now that you've configured your YAML file to enable favicons, your website will automatically generate favicons suitable for a wide array of devices and operating systems. This eliminates the need to manually create and link multiple favicon sizes, ensuring your site's icon is always displayed optimally, whether viewed on a desktop browser, a tablet, or a mobile phone.

## Icon masks and safe zone

As mentioned in the [Web App Manifest](https://www.w3.org/TR/appmanifest/#icon-masks), _Some platforms have their own preferred icon shape_.

You can reduce the size of the image. A transparent background will be added to fulfil with this requirement.

<figure><img src="../.gitbook/assets/safe-zone.svg" alt=""><figcaption><p>Safe zone</p></figcaption></figure>

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        image_scale: 80 # = 80% of the original size
```
{% endcode %}

## Background and Rounded Corners

To add a background color and rounded corners to your app's icon, specify the background color and corner shape in your `pwa.yaml` configuration. This customization enhances the visual appeal of your icons on various platforms that support these features.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        background_color: "#ffffff" # Hex color for the icon background
        border_radius: 10 # = 10% of the size. 50% is a circle.
```
{% endcode %}

{% hint style="info" %}
border\_radius has no effect if no background is set.
{% endhint %}

## Safari Pinned Tab

By setting a `safari_pinned_tab_color` to a color value, the corresponding HTML tags will be added. The icon will be rendered as it is, even if you specified background color or border radius.

If you prefer a silhouette instead, you can set the option `use_silhouette` to `true`. Note that this option requires [potrace](https://en.wikipedia.org/wiki/Potrace) to be installed.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        safari_pinned_tab_color: "#f5ef06" # Safari only
        use_silhouette: true
        potrace: "/path/to/potrace" #Optional.
        # Only if potrace is installed in a non-conventional folder
```
{% endcode %}

## Windows 8 / 10 Tiles

You can enable Windows 8 / 10 tiles by setting a tile color.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        tile_color: "#f5ef06" # Windows 8 to 10 only
```
{% endcode %}

<figure><img src="../.gitbook/assets/c6f236b2-2093-4fdf-add1-303e4656f990.png" alt=""><figcaption><p>Example of Windows 8 tiles</p></figcaption></figure>

{% hint style="info" %}
Background color or border radius options have no effect on the windows tiles
{% endhint %}

## Low Resolution Icons

By default, favicons are created for modern platforms. It may be interesting to provide icons for old systems such as iOS6 or Chrome 20. To do so, you just need to enable the low\_resolution option and other icon sizes will be generated.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        src: assets/icon.svg
        low_resolution: true
```
{% endcode %}

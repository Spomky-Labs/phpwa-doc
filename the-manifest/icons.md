# Icons

To integrate the icon details into the Progressive Web App (PWA) manifest file, ensure that each icon listed is accompanied by its respective size. For example, `icon-256x256.png` is indicated as having a size of 256px by 256px. This is crucial for providing clear visual elements across different devices and resolutions.

The `sizes` attribute indicates the size of the icon to the browser. For PNG or JPEG icons, specify the dimensions (e.g., 48, 96, 256). For vector icons, you can use "any" as they are scalable without losing quality. The `format` attribute is also important as it tells the browser what the file format is, helping it to render the image correctly or the browser to select the most suitable format.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        icons:
            - src: "icons/icon.png"
              sizes: [192]
            - src: "icons/maskable-icon.png"
              sizes: [192]
              purpose: "maskable"
            - src: "icons/icon.svg"
              sizes: 0
```
{% endcode %}

### `src` Parameter

The `src` parameter is the path to the resource file. It can be:

* an [Asset Mapper resource](https://pwa.spomky-labs.com/),
* an absolute path to the resource,
* a [Symfony UX Icon](https://ux.symfony.com/icons) if the `symfony/ux-icons` bundle is installed.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        icons:
            - src: "icons/icon-48x48.png"
              sizes: [48]
            - src: "/home/project/foo/bar/icon-48x48.png"
              sizes: [48]
            - src: "bx:badge-check"
              sizes: [48]
```
{% endcode %}

### `sizes` Parameter

The sizes parameter indicates the suitable sizes for the icon. The expected value is an positive integer or a list of positive integers.

`0` means `any` size and is suitable only for vector images.

The recommended sizes for application icons are as 48, 72, 96, 144, 168, 192, 256 and 512 pixels.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        icons:
            - src: "icons/icon-192x192.png"
              sizes: [48, 96, 192]
            - src: "icons/icon.svg"
              sizes: 0
```
{% endcode %}

### `format` Parameter

When the `format` parameter is set, the bundle will try to save the image in the specified format. If the component `symfony/mime` is present, the bundle will guess the correct type.

In general, browsers can read `svg`, `png` and `jpg` types. Modern browsers may support `webp`.

{% hint style="info" %}
Conversion to SVG is not possible.
{% endhint %}

### `purpose` Parameter

The purpose `maskable` icons indicates the icon has a security margin and borders can be cropped on certain devices.

<figure><img src="../.gitbook/assets/maskable-icon-safe-area (1).png" alt=""><figcaption><p>Maskable image safe area</p></figcaption></figure>

### `svg_color` Parameter

Some SVG icons have a `currentColor` atribute or no `color` attribute and the bundle automatically sets `#000` (black color) as default color.

You can change it using the option `svg_color`:

{% code title="config/packages/pwa.yaml" overflow="wrap" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        icons:
            - src: "icons/icon.svg"
              sizes: 0
              svg_color: '#15fe68'
```
{% endcode %}

# Startup Images

Startup images — splash screens — are what iOS paints while your application boots from the home screen. Without them, an installed PWA shows a white screen until the first paint.

{% hint style="info" %}
Startup images are an iOS feature. Android builds its own splash screen from the manifest `icons` and `background_color`, and needs no configuration at all.
{% endhint %}

## Quick Start

Startup images have a section of their own since 1.6.0.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    startup_images:
        enabled: true
        default:
            src: 'icons/logo.svg'
            background_color: '#ffffff'
```
{% endcode %}

Nothing else is needed: run `php bin/console pwa:compile`, keep `{{ pwa() }}` in your `<head>`, and every `<link rel="apple-touch-startup-image">` tag is written for you.

### Inherited from the favicons

The section is optional. When you say nothing, the startup images keep borrowing everything from the favicons, the way they did before 1.6.0.

| `pwa.startup_images` … | falls back to … |
| --- | --- |
| `enabled` | `pwa.favicons.enabled` **and** `pwa.favicons.use_start_image` |
| `default.src` / `dark.src` | `pwa.favicons.default.src` / `pwa.favicons.dark.src` |
| `default.background_color` | `pwa.favicons.default.background_color`, then `pwa.manifest.background_color` |
| `default.border_radius` | `pwa.favicons.default.border_radius` |
| `default.svg_attr` | `pwa.favicons.default.svg_attr` |

`pwa.favicons.monochrome` is followed as well, and has no counterpart in this section: a monochrome
favicon means a monochrome startup image.

So this 1.5.x configuration still produces the very same images, byte for byte:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    favicons:
        enabled: true
        default:
            src: 'icons/logo.svg'
            background_color: '#ffffff'
```
{% endcode %}

{% hint style="warning" %}
`pwa.favicons.use_start_image` is deprecated since 1.6.0 and will be removed in 2.0.0. It now only seeds `pwa.startup_images.enabled`; set that one instead.
{% endhint %}

A color scheme nobody described an image for is simply not declined, and when no `default` source can be resolved at all, no startup image is generated.

## What Gets Generated

The bundle knows 21 iOS screen geometries, declined over both orientations — 42 images, doubled when a `dark` variant is configured.

| Family | Devices |
| --- | --- |
| iPad | Pro 13" (M4), Pro 11" (M4), Pro 12.9", Pro 11", Pro 10.5", Air 10.9", 10.2", mini 8.3", 9.7" |
| iPhone | 16 Pro Max, 16 Pro, 15 Pro Max, 15 Pro, 14 Plus, 14, 11 Pro Max, 11, 11 Pro, 8 Plus, 8, SE |

Two devices sharing a point size and a pixel ratio are one entry: their media queries would be identical, and the second image would never be picked.

Each image is written to `/pwa/start-image-{width}x{height}-{hash}.png`, where the hash covers everything the rendering is made of. Change the source image, the background color or the template, and the URL changes with it.

{% code title="the tags injected by {{ pwa() }}" lineNumbers="true" %}
```html
<link rel="apple-touch-startup-image" sizes="1290x2796" type="image/png"
      href="/pwa/start-image-1290x2796-8a3f….png"
      media="(device-width: 430px) and (device-height: 932px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)">
```
{% endcode %}

When a `dark` theme is configured, `and (prefers-color-scheme: light)` / `and (prefers-color-scheme: dark)` is appended to each query.

{% hint style="danger" %}
iOS only shows an image whose dimensions match the screen **exactly**. This is why the geometry is not negotiable, and why the bundle fails loudly rather than shipping an image of the wrong size.
{% endhint %}

## Two Ways Of Drawing The Image

### The plain image

This is the default, and what the bundle has always done: the source image, scaled down and centered over the background color. It asks nothing of your application beyond the image processor the favicons already need.

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    startup_images:
        enabled: true
        default:
            src: 'icons/logo.svg'
            background_color: '#ffffff'
            border_radius: 20
        dark:
            src: 'icons/logo.dark.svg'
            background_color: '#0b0b0b'
```
{% endcode %}

{% hint style="info" %}
The share of the screen the logo takes is derived from the screen diagonal: the bigger the screen, the smaller the share. A logo scaled to a third of an iPad screen reads as a poster, where the very same share of an iPhone SE screen reads as an icon. The `image_scale` option is therefore not applied to startup images.
{% endhint %}

### A Twig template

A logo centered over a color is all a startup image could ever be before 1.6.0. An application wanting its name, a baseline, or a layout of its own now describes the document iOS paints:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    startup_images:
        enabled: true
        template: 'pwa/startup_image.html.twig'
        context:
            subtitle: 'Identify every plant, even offline.'
        default:
            src: 'icons/logo.svg'
            background_color: '#ffffff'
        dark:
            src: 'icons/logo.dark.svg'
            background_color: '#0b0b0b'
            context:
                subtitle: 'The dark side of the application.'
```
{% endcode %}

The template is rendered by your own Twig environment: its globals, extensions and filters are the ones your application already knows. The document is then painted by a headless browser and captured at the exact pixel size of each screen.

{% hint style="warning" %}
Describing the images with a template is a **compile-time** requirement: `symfony/twig-bundle` and a headless browser must be available when `pwa:compile` runs. Asking for a template without the means to render it is reported, not worked around — quietly shipping the plain image would ship something you chose not to ask for.
{% endhint %}

#### Requirements

{% code title="terminal" %}
```bash
composer require --dev symfony/panther
vendor/bin/bdi detect drivers
```
{% endcode %}

A Chrome driver matching the installed Chrome is what actually paints the document. The following environment variables are honoured:

| Variable | Effect |
| --- | --- |
| `PANTHER_CHROME_DRIVER_BINARY` | Path to an existing driver, instead of the detected one |
| `PANTHER_NO_SANDBOX` | Adds `--no-sandbox`, needed in most containers |
| `PANTHER_NO_HEADLESS` | Runs the browser with a window — for debugging only, the capture then depends on your display |
| `PANTHER_CHROME_ARGUMENTS` | Extra arguments, space separated |

The browser is started once and reused for the whole compilation: there are more than eighty images to produce, and a browser start dwarfs the capture itself.

#### The variables handed to the template

| Variable | Description |
| --- | --- |
| `width`, `height` | The geometry of the image, in device pixels |
| `orientation` | `portrait` or `landscape` |
| `device_width`, `device_height` | The geometry of the device, in CSS pixels |
| `pixel_ratio` | The device pixel ratio, `2` or `3` |
| `device_name` | The device the image is produced for, e.g. `iPhone 16 Pro` |
| `color_scheme` | `light` or `dark` |
| `background_color` | The background color of the theme being rendered |
| `theme_color` | `pwa.manifest.theme_color`, or its dark counterpart |
| `app_name`, `short_name`, `description` | What the manifest declares |
| `lang`, `dir` | The language and writing direction of the manifest |
| `icon` | The source image, inlined as a `data:` URI |
| `icon_svg` | The source markup when it is an SVG, `null` otherwise |
| `context` | Whatever you declared under `context` |

{% hint style="info" %}
The manifest values are the raw configured ones. When they are translation keys, run them through the `trans` filter with the `pwa` domain: `{{ app_name|trans({}, 'pwa') }}`.
{% endhint %}

The document is painted from a temporary file with no application URL to resolve an asset against, which is why the source image is handed over inlined rather than as a path. For the same reason, any font, image or stylesheet the template pulls in must be inlined too.

#### Extending the shipped template

`@SpomkyLabsPwa/StartupImage/default.html.twig` is a complete starting point. Extend it rather than copy it when only a piece has to change:

{% code title="templates/pwa/startup_image.html.twig" lineNumbers="true" %}
```twig
{% extends '@SpomkyLabsPwa/StartupImage/default.html.twig' %}

{% block subtitle %}
    <p class="subtitle">{{ context.subtitle|trans({}, 'pwa') }}</p>
{% endblock %}
```
{% endcode %}

The blocks it exposes are `stylesheet`, `body`, `logo`, `name` and `subtitle`.

Everything in it is sized against `--unit`, a hundredth of the shorter side, so that a layout written once holds from an iPhone SE to an iPad Pro. Lengths are in device pixels: the document is painted at the exact pixel size of the screen, with no device pixel ratio applied on top.

{% hint style="warning" %}
The safe area of a notched screen is **not** honoured by a startup image. Keep your content away from the edges by hand — the shipped template reserves ten units of padding.
{% endhint %}

#### Per color scheme variables

`context` is declared once for every scheme, and refined per scheme. The two are merged, the more specific one winning:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    startup_images:
        template: 'pwa/startup_image.html.twig'
        context:
            baseline: 'Your daily companion'
            wordmark: 'ACME'
        dark:
            src: 'icons/logo.dark.svg'
            context:
                baseline: 'Your nightly companion'   # overrides the shared one
```
{% endcode %}

`border_radius` and `image_scale` are ignored when a template is used: the template is free to size and shape the image as it sees fit.

### Painting the document yourself

The headless browser sits behind `HtmlRendererInterface`. Register a service implementing it and it replaces the bundled one — useful when your CI has no Chrome, or when you would rather drive a rendering service.

{% code title="src/Pwa/MyHtmlRenderer.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Pwa;

use SpomkyLabs\PwaBundle\StartupImage\HtmlRendererInterface;

final class MyHtmlRenderer implements HtmlRendererInterface
{
    public function capture(string $html, int $width, int $height): string
    {
        // Return the PNG bytes of a viewport exactly $width x $height device pixels.
        // Throw rather than return an image of a different size: iOS would discard it.
    }
}
```
{% endcode %}

{% code title="config/services.yaml" lineNumbers="true" %}
```yaml
services:
    SpomkyLabs\PwaBundle\StartupImage\HtmlRendererInterface:
        alias: App\Pwa\MyHtmlRenderer
```
{% endcode %}

## Reference

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    startup_images:
        enabled: ~              # defaults to "favicons.use_start_image"
        template: ~             # a Twig template describing the image
        context: []             # variables handed to the template, whatever the color scheme
        default:
            src: ~              # Asset Mapper path, absolute path or UX Icon name
            background_color: ~
            border_radius: ~    # 1 to 50, ignored when a template is used
            image_scale: ~      # 1 to 100, not applied to startup images
            svg_attr: []        # attributes added to the SVG root
            context: []         # merged over the shared ones
        dark:
            # the same keys, for the dark color scheme
```
{% endcode %}

Turning them off entirely:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    startup_images:
        enabled: false
```
{% endcode %}

## Troubleshooting

### Nothing shows, or a white screen shows

* The application must be **installed** on the home screen; Safari itself never shows a startup image.
* Check that the `<link rel="apple-touch-startup-image">` tags are in the `<head>`, i.e. that `{{ pwa() }}` is rendered there.
* Check that the device geometry is one of the 21 the bundle knows about. A screen with no matching media query gets no image.
* Recompile with `php bin/console pwa:compile` after any change: the URLs carry a content hash.

### `Unable to paint a startup image with a headless browser`

The Chrome driver is missing or does not match the installed Chrome. Run `vendor/bin/bdi detect drivers`, or point `PANTHER_CHROME_DRIVER_BINARY` at an existing one. In a container, add `PANTHER_NO_SANDBOX=1`.

### `The browser produced a 1280x800 startup image where 1290x2796 was requested`

The window was capped by the display, which happens when the browser is not headless. Unset `PANTHER_NO_HEADLESS`.

### `The startup images are described by the template …, but Twig is not available`

Install `symfony/twig-bundle`, or drop the `template` option to go back to the plain image.

### `Unable to render the startup images: no image processor is available`

The plain rendering needs GD or Imagick. Set `pwa.image_processor`, or describe the images with a template.

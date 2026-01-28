# Screenshots

Taking screenshots of your application has great benefits. It may become difficult to manage screenshot updates as the application evolves, in particular if multiple sizes are managed.

This command requires dependencies and **should be executed in `dev` environment only**.

```sh
composer require --dev symfony/panther
composer require --dev dbrekelmans/bdi && vendor/bin/bdi detect drivers
```

## Image Processor

Please read the section on [the Icons page](icons.md#image-processor).

## Using PHP Attributes

Declare screenshots directly on your controller methods using the `#[Screenshot]` attribute. This provides automatic manifest integration.

```php
<?php

namespace App\Controller;

use SpomkyLabs\PwaBundle\Attribute\Screenshot;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class HomeController extends AbstractController
{
    #[Route('/', name: 'app_homepage')]
    #[Screenshot(
        sizes: ['fhd', 'ipad/LP', 'iphone-14'],
        name: 'homepage',
        label: 'Application homepage',
        platform: 'windows'
    )]
    public function index(): Response
    {
        return $this->render('home/index.html.twig');
    }
}
```

### Generating Screenshots

Screenshots can be generated in two ways:

#### With `pwa:compile` (Recommended)

The `pwa:compile` command automatically generates screenshots from attributes before compiling the manifest:

```sh
php bin/console pwa:compile
```

Use `--no-screenshots` to skip screenshot generation:

```sh
php bin/console pwa:compile --no-screenshots
```

#### Standalone Command

You can also generate screenshots independently:

```sh
php bin/console pwa:create:screenshot --from-attributes
# or short form:
php bin/console pwa:create:screenshot -a
```

The command will automatically discover all routes with `#[Screenshot]` attributes and generate the screenshots.

### Attribute Parameters

| Parameter    | Type   | Required | Description |
|--------------|--------|----------|-------------|
| `sizes`      | array  | Yes      | Array of size profiles or explicit dimensions with optional orientation selectors |
| `parameters` | array  | No       | Route parameters (e.g., `['id' => 123]`). Note: use `locales` parameter for `_locale` |
| `locales`    | array  | No       | Locales to generate screenshots for (e.g., `['fr', 'en']`). Each locale generates separate screenshots |
| `name`       | string | No       | Base filename (defaults to route name) |
| `label`      | string | No       | Label for the screenshot in the manifest (supports translation) |
| `platform`   | string | No       | Target platform (android, ios, windows, etc.) |
| `output`     | string | No       | Custom output directory (defaults to `assets/screenshots/`) |
| `format`     | string | No       | Output format: png, jpg, webp (defaults to png) |

### Automatic Manifest Integration

Screenshots declared with `#[Screenshot]` attributes are **automatically integrated into your PWA manifest**. The `ManifestScreenshotListener` discovers all controller methods with the attribute and adds the corresponding screenshot files to your manifest during compilation.

When a screenshot file exists, it's automatically added to the manifest with:
- `src`: The asset path (via AssetMapper if available)
- `width` and `height`: From the generated filename
- `form_factor`: Auto-calculated (wide/narrow based on dimensions)
- `platform`: From the attribute
- `type`: Based on format (e.g., "image/webp")
- `label`: From the attribute (supports translation via the `pwa` domain)
- `reference`: The generated URL for the route

**No manual configuration needed!**

### Multi-Locale Support

For internationalized applications, use the `locales` parameter to generate screenshots for multiple locales with a single attribute:

```php
#[Route('/{_locale}/dashboard', name: 'app_dashboard')]
#[Screenshot(
    sizes: ['desktop-lg', 'tablet/LP'],
    locales: ['fr', 'en', 'de'],
    label: 'Dashboard'
)]
public function dashboard(): Response
{
    return $this->render('dashboard/index.html.twig');
}
```

This generates screenshots for each locale with appropriate filenames (e.g., `app-dashboard-fr-1920x1080.png`, `app-dashboard-en-1920x1080.png`).

The manifest compiler filters screenshots by the current locale, ensuring only relevant screenshots are included in the localized manifest.

### Routes with Parameters

For routes with other parameters (like entity IDs), use the `parameters` attribute. Use multiple `#[Screenshot]` attributes for different parameter combinations:

```php
#[Route('/product/{id}', name: 'app_product')]
#[Screenshot(
    sizes: ['fhd', 'iphone-14'],
    parameters: ['id' => 1],
    name: 'product-featured',
    label: 'Featured Product'
)]
#[Screenshot(
    sizes: ['fhd', 'iphone-14'],
    parameters: ['id' => 42],
    name: 'product-popular',
    label: 'Popular Product'
)]
public function show(int $id): Response
{
    return $this->render('product/show.html.twig');
}
```

## Size Profiles

The `sizes` array accepts:
- **Predefined profile names**: `"fhd"`, `"iphone-14"`, `"ipad"`, `"desktop-lg"`, etc.
- **Explicit dimensions**: `"1920x1080"`, `"390x844"`, etc.
- **Orientation selectors**: `/L` (landscape), `/P` (portrait), `/LP` (both)

Example: `"ipad/LP"` generates both 768x1024 (portrait) and 1024x768 (landscape) screenshots.

### Available Size Profiles

#### Standard Resolutions (Landscape)

| Profile | Dimensions | Form Factor |
|---------|------------|-------------|
| `hd` | 1280×720 | wide |
| `fhd` | 1920×1080 | wide |
| `2k` | 2560×1440 | wide |
| `4k` | 3840×2160 | wide |
| `8k` | 7680×4320 | wide |

#### Portrait Resolutions

| Profile | Dimensions | Form Factor |
|---------|------------|-------------|
| `480p` | 640×480 | wide |
| `720p` | 720×1280 | narrow |
| `1080p` | 1080×1920 | narrow |
| `1440p` | 1440×2560 | narrow |

#### iPhone Models

| Profile | Dimensions | Form Factor |
|---------|------------|-------------|
| `iphone-se` | 375×667 | narrow |
| `iphone-8` | 375×667 | narrow |
| `iphone-12` | 390×844 | narrow |
| `iphone-13` | 390×844 | narrow |
| `iphone-14` | 390×844 | narrow |
| `iphone-14-plus` | 428×926 | narrow |
| `iphone-14-pro` | 393×852 | narrow |
| `iphone-14-pro-max` | 430×932 | narrow |
| `iphone-15` | 393×852 | narrow |
| `iphone-15-pro-max` | 430×932 | narrow |

#### Android Devices

| Profile | Dimensions | Form Factor |
|---------|------------|-------------|
| `pixel-5` | 393×851 | narrow |
| `pixel-6` | 412×915 | narrow |
| `pixel-7` | 412×915 | narrow |
| `pixel-7-pro` | 412×915 | narrow |
| `galaxy-s21` | 360×800 | narrow |
| `galaxy-s22` | 360×800 | narrow |
| `galaxy-s23` | 360×780 | narrow |

#### Tablets

| Profile | Dimensions | Form Factor |
|---------|------------|-------------|
| `ipad` | 768×1024 | narrow |
| `ipad-pro-11` | 834×1194 | narrow |
| `ipad-pro-129` | 1024×1366 | narrow |

{% hint style="info" %}
For tablets, use orientation selectors to generate both portrait and landscape: `ipad/LP`
{% endhint %}

#### Web Breakpoints

| Profile | Dimensions | Form Factor |
|---------|------------|-------------|
| `mobile-sm` | 320×568 | narrow |
| `mobile-md` | 375×667 | narrow |
| `mobile-lg` | 414×896 | narrow |
| `tablet` | 768×1024 | narrow |
| `desktop-sm` | 1024×768 | wide |
| `desktop-md` | 1366×768 | wide |
| `desktop-lg` | 1920×1080 | wide |
| `desktop-xl` | 2560×1440 | wide |

## Configuration

### User Agent

During screenshot generation, the bundle uses a special user agent (`PWAScreenshotBot` by default) to identify screenshot requests. When this user agent is detected, the Symfony profiler and debug toolbar are automatically disabled to ensure clean screenshots.

You can customize this user agent in your configuration:

```yaml
# config/packages/pwa.yaml
pwa:
    web_client:
        user_agent: 'MyCustomScreenshotBot'
```

### Custom Web Client

For advanced use cases, you can provide a custom Panther client:

```yaml
# config/packages/pwa.yaml
pwa:
    web_client:
        web_client: 'my_custom_panther_client'  # Service ID
```

## Legacy Command (Deprecated)

The simple URL-based command is still available but deprecated:

```sh
php bin/console pwa:create:screenshot https://example.com
```

Options:
- `--output` / `-o`: Output directory (default: `assets/screenshots/`)
- `--filename`: Base filename (default: `screenshot`)
- `--width`: Screenshot width
- `--height`: Screenshot height
- `--format` / `-f`: Output format (png, jpg, webp)

{% hint style="warning" %}
This legacy command does not support automatic manifest integration. Use the `#[Screenshot]` attribute approach instead.
{% endhint %}

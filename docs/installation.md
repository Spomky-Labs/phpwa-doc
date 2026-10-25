# Bundle installation

Transform your Symfony application into a Progressive Web App in just a few minutes.

## Step 1: Install the bundle

Install the bundle via Composer:

```bash
composer require spomky-labs/pwa-bundle
```

!!! info

    This bundle does not have a Symfony Flex recipe, so you'll need to create the configuration file manually in the next step.

## Step 2: Create configuration file

Create a configuration file to customize your PWA settings:

```yaml title="config/packages/pwa.yaml" linenums="1"
pwa:
    manifest:
        enabled: true
        name: 'My Application'
        short_name: 'MyApp'

    serviceworker:
        enabled: true
```

You can start with minimal configuration (`pwa: ~`) and add settings as you build your PWA.

## Step 3: Integrate in your templates

Add the PWA function to the `<head>` section of your base template:

```twig title="templates/base.html.twig" linenums="1"
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}My PWA{% endblock %}</title>

    {# PWA integration - add before closing </head> tag #}
    {{ pwa() }}
</head>
<body>
    {% block body %}{% endblock %}
</body>
</html>
```

!!! success

    The `{{ pwa() }}` function automatically injects:
    - The Web App Manifest link
    - Service worker registration script
    - Theme color meta tags
    - Favicon links

## Step 4: Clear cache

Clear your Symfony cache to activate the bundle:

```bash
php bin/console cache:clear
```

## Step 5: Verify installation

Visit your application in a browser and check:

1. **Manifest**: Open DevTools → Application tab → Manifest
2. **Service Worker**: Look in Application tab → Service Workers
3. **Install Prompt**: Visit your app on mobile or desktop - you should see an install option

!!! info

    In development mode, the service worker may show debugging information. This is normal and will be removed in production.

## What's next?

Your Symfony application is now a basic PWA! Enhance it further:

- **[Configure the manifest](the-manifest/application-information/README.md)** - Customize app name, icons, colors
- **[Set up caching](the-service-worker/workbox/README.md)** - Enable offline functionality
- **[Add icons](image-management/icons.md)** - Create app icons for all platforms
- **[Configure the service worker](the-service-worker/configuration.md)** - Control caching behavior
- **[Add UX components](symfony-ux/)** - Integrate device APIs

## Troubleshooting

### PWA function not found

If you see "Unknown function pwa", ensure:
1. The bundle is properly installed (`composer.json` should include it)
2. Cache is cleared (`php bin/console cache:clear`)
3. You're using Twig (not PHP templates)

### Service worker not registering

Check that:
1. Your site is served over HTTPS (or localhost for development)
2. The service worker is enabled in `pwa.yaml`
3. No JavaScript errors in the browser console

### No install prompt appearing

The install prompt requires:
1. Valid manifest with required properties (name, icons, start_url)
2. Service worker successfully registered
3. HTTPS connection (except localhost)
4. User engagement (varies by browser)

See [Application Information](the-manifest/application-information/README.md) for complete requirements.

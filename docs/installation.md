# Bundle Installation

Transform your Symfony application into a Progressive Web App in just a few minutes.

## Step 1: Install the Bundle

Install the bundle via Composer:

```bash
composer require spomky-labs/pwa-bundle
```

{% hint style="info" %}
This bundle does not have a Symfony Flex recipe, so you'll need to create the configuration file manually in the next step.
{% endhint %}

## Step 2: Create Configuration File

Create a configuration file to customize your PWA settings:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: 'My Application'
        short_name: 'MyApp'

    serviceworker:
        enabled: true
```
{% endcode %}

You can start with minimal configuration (`pwa: ~`) and add settings as you build your PWA.

## Step 3: Integrate in Your Templates

Add the PWA function to the `<head>` section of your base template:

{% code title="templates/base.html.twig" lineNumbers="true" %}
```twig
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
{% endcode %}

{% hint style="success" %}
The `{{ pwa() }}` function automatically injects:
- The Web App Manifest link
- Service worker registration script
- Theme color meta tags
- Favicon links
{% endhint %}

## Step 4: Clear Cache

Clear your Symfony cache to activate the bundle:

```bash
php bin/console cache:clear
```

## Step 5: Verify Installation

Visit your application in a browser and check:

1. **Manifest**: Open DevTools → Application tab → Manifest
2. **Service Worker**: Look in Application tab → Service Workers
3. **Install Prompt**: Visit your app on mobile or desktop - you should see an install option

{% hint style="info" %}
In development mode, the service worker may show debugging information. This is normal and will be removed in production.
{% endhint %}

## What's Next?

Your Symfony application is now a basic PWA! Enhance it further:

- **[Configure the manifest](the-manifest/application-information/)** - Customize app name, icons, colors
- **[Set up caching](the-service-worker/workbox/)** - Enable offline functionality
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

See [Application Information](the-manifest/application-information/) for complete requirements.

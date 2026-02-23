# Widgets (Win10+)

{% hint style="warning" %}
**Experimental Feature**: Windows Widgets for PWAs is currently an experimental feature primarily supported in Microsoft Edge on Windows 11+. The API and behavior may change.
{% endhint %}

Windows Widgets allow your PWA to provide mini-applications displayed on the Windows widgets dashboard. Users can interact with your app content without opening the full application.

## Overview

PWA Widgets use the [Adaptive Cards](https://adaptivecards.io/) template format to render content in the Windows widget panel. The bundle handles:

- **Manifest configuration**: Declaring widgets in the Web App Manifest
- **Service worker integration**: Automatic widget lifecycle management (install, update, uninstall)
- **Periodic updates**: Background data refresh via Periodic Sync

## Configuration

Widgets are configured in the manifest section:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        widgets:
            - name: "Weather Widget"
              short_name: "Weather"
              description: "Current weather conditions at a glance"
              tag: "weather"
              ms_ac_template: 'app_widget_weather_template'  # Symfony route name
              data: 'app_widget_weather_data'                 # Symfony route name
              type: "application/json"
              auth: false
              update: 3600          # Update every hour (in seconds)
              multiple: false       # Only one instance allowed
              screenshots:
                  - src: 'images/widget-weather.png'
                    label: 'Weather widget preview'
              icons:
                  - src: 'images/widget-weather-icon.svg'
```
{% endcode %}

## Configuration Reference

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `name` | string | Yes | Widget title displayed to users |
| `description` | string | Yes | Description of the widget |
| `tag` | string | Yes | Unique identifier for the widget |
| `ms_ac_template` | URL/route | Yes | URL or route name returning the Adaptive Cards template |
| `data` | URL/route | Yes | URL or route name returning widget data (JSON) |
| `screenshots` | array | Yes | At least one screenshot showing the widget |
| `short_name` | string | No | Shorter version of the widget name |
| `type` | string | No | MIME type for widget data (default: `application/json`) |
| `auth` | boolean | No | Whether the widget requires authentication |
| `update` | integer | No | Update frequency in seconds |
| `multiple` | boolean | No | Allow multiple widget instances (default: `true`) |
| `icons` | array | No | Widget-specific icons (falls back to manifest icons) |

## Server-Side Implementation

### Adaptive Cards Template

Create a controller that returns an Adaptive Cards template:

{% code title="src/Controller/WidgetController.php" lineNumbers="true" %}
```php
<?php

declare(strict_types=1);

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Attribute\Route;

final class WidgetController extends AbstractController
{
    #[Route('/widget/weather/template', name: 'app_widget_weather_template')]
    public function template(): JsonResponse
    {
        return $this->json([
            'type' => 'AdaptiveCard',
            '$schema' => 'http://adaptivecards.io/schemas/adaptive-card.json',
            'version' => '1.5',
            'body' => [
                [
                    'type' => 'TextBlock',
                    'text' => '${city}',
                    'size' => 'Large',
                    'weight' => 'Bolder',
                ],
                [
                    'type' => 'TextBlock',
                    'text' => '${temperature}°C - ${condition}',
                    'size' => 'Medium',
                ],
            ],
        ]);
    }

    #[Route('/widget/weather/data', name: 'app_widget_weather_data')]
    public function data(): JsonResponse
    {
        // Fetch actual weather data from your service
        return $this->json([
            'city' => 'Paris',
            'temperature' => 22,
            'condition' => 'Sunny',
        ]);
    }
}
```
{% endcode %}

## How the Service Worker Works

The bundle automatically generates service worker code that:

1. **On widget install** (`widgetinstall` event):
   - Fetches the Adaptive Cards template from `ms_ac_template`
   - Fetches data from the `data` URL
   - Renders the widget using `self.widgets.updateByTag()`
   - Registers periodic sync if `update` is configured

2. **On periodic sync** (`periodicsync` event):
   - Refreshes the template and data
   - Updates the widget with fresh content

3. **On widget uninstall** (`widgetuninstall` event):
   - Unregisters periodic sync when the last widget instance is removed

4. **On service worker activation**:
   - Updates all registered widgets with fresh data

## Multiple Widgets

You can configure multiple widgets:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        widgets:
            - name: "Weather Widget"
              tag: "weather"
              ms_ac_template: 'app_widget_weather_template'
              data: 'app_widget_weather_data'
              update: 3600
              screenshots:
                  - src: 'images/widget-weather.png'
                    label: 'Weather'

            - name: "News Headlines"
              tag: "news"
              ms_ac_template: 'app_widget_news_template'
              data: 'app_widget_news_data'
              update: 1800   # Every 30 minutes
              multiple: true
              screenshots:
                  - src: 'images/widget-news.png'
                    label: 'News headlines'
```
{% endcode %}

## Requirements and Limitations

- **Platform**: Microsoft Edge on Windows 11+ only
- **Template format**: Must use [Adaptive Cards](https://adaptivecards.io/) JSON format
- **Data format**: Widget data endpoints must return valid JSON
- **URLs**: Template and data URLs must be within the service worker scope
- **Update intervals**: Subject to browser/OS scheduling policies (minimum interval may be enforced)
- **Icon size**: Icons larger than 1024x1024 are ignored

## Resources

- [Microsoft PWA Widgets Documentation](https://learn.microsoft.com/en-us/microsoft-edge/progressive-web-apps-chromium/how-to/widgets)
- [Adaptive Cards Designer](https://adaptivecards.io/designer/)
- [Adaptive Cards Schema](https://adaptivecards.io/explorer/)

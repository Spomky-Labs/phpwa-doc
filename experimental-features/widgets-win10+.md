# Widgets (Win10+)

{% hint style="warning" %}
**Experimental Feature**: Windows widgets are an experimental feature currently only supported in Microsoft Edge on Windows 10 and later. The specification and implementation may change.
{% endhint %}

## Overview

Windows widgets allow your PWA to provide glanceable, at-a-glance information directly in the Windows widgets dashboard. Users can pin your app's widgets to their desktop for quick access to key information without opening the full application.

**Key Features**:
- Display real-time information in the Windows widgets panel
- Update automatically via service worker
- Support for multiple widget instances
- Customizable using Adaptive Cards templates
- Integrated with Windows notifications and actions

## Browser Support

| Platform | Browser | Support |
|----------|---------|---------|
| Windows 10+ | Microsoft Edge | ✅ Full support |
| Windows 10+ | Chrome | ❌ Not supported |
| macOS | All browsers | ❌ Not supported |
| Linux | All browsers | ❌ Not supported |
| Android | All browsers | ❌ Not supported |
| iOS | All browsers | ❌ Not supported |

{% hint style="info" %}
**Check before using**: Always feature-detect widget support in your service worker before attempting to update widgets.
{% endhint %}

## Configuration

### Basic Widget Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        widgets:
            - name: "Weather Widget"
              description: "Current weather conditions"
              tag: "weather"
              ms_ac_template: "widgets/weather-template.json"
              data: "api/widget/weather"
              screenshots:
                  - src: "screenshots/widget-weather.png"
                    sizes: [300, 200]
              update: 1800  # Update every 30 minutes
```
{% endcode %}

### Complete Widget Configuration

All available widget properties:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        widgets:
            - name: "Task Counter"
              short_name: "Tasks"
              description: "Number of pending tasks"
              tag: "task-counter"
              icons:
                  - src: "icons/widget-tasks.png"
                    sizes: [72]
              screenshots:
                  - src: "screenshots/widget-tasks.png"
                    sizes: [300, 200]
                    label: "Task counter widget preview"
              ms_ac_template: "widgets/tasks-template.json"
              data: "api/widgets/tasks"
              type: "application/json"
              auth: true
              update: 3600  # Update every hour
              multiple: true  # Allow multiple instances
```
{% endcode %}

## Configuration Properties

### Required Properties

| Property | Type | Description |
|----------|------|-------------|
| `name` | string | The title of the widget displayed to users |
| `description` | string | A description of what the widget shows |
| `tag` | string | Unique identifier used to reference the widget in the service worker |
| `ms_ac_template` | URL | URL to the Adaptive Cards template JSON file |
| `screenshots` | array | At least one screenshot showing the widget preview |

### Optional Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `short_name` | string | - | Short version of the name for small spaces |
| `icons` | array | Manifest icons | Custom icons for the widget (max 1024×1024) |
| `data` | URL | - | URL endpoint that returns JSON data for the template |
| `type` | string | - | MIME type for the widget data (e.g., `application/json`) |
| `auth` | boolean | false | Whether the widget requires authentication |
| `update` | integer | - | Update frequency in seconds for periodic refresh |
| `multiple` | boolean | true | Allow multiple instances of the same widget |

## Adaptive Cards Templates

Widgets use [Microsoft Adaptive Cards](https://adaptivecards.io/) for layout. The template defines the widget's visual structure.

### Example Template

{% code title="public/widgets/weather-template.json" lineNumbers="true" %}
```json
{
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [
        {
            "type": "TextBlock",
            "text": "${location}",
            "size": "Large",
            "weight": "Bolder"
        },
        {
            "type": "ColumnSet",
            "columns": [
                {
                    "type": "Column",
                    "width": "auto",
                    "items": [
                        {
                            "type": "Image",
                            "url": "${icon}",
                            "size": "Large"
                        }
                    ]
                },
                {
                    "type": "Column",
                    "width": "stretch",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "${temperature}°",
                            "size": "ExtraLarge",
                            "weight": "Bolder"
                        },
                        {
                            "type": "TextBlock",
                            "text": "${conditions}",
                            "size": "Medium"
                        }
                    ]
                }
            ]
        },
        {
            "type": "FactSet",
            "facts": [
                {
                    "title": "Humidity:",
                    "value": "${humidity}%"
                },
                {
                    "title": "Wind:",
                    "value": "${wind} km/h"
                }
            ]
        }
    ]
}
```
{% endcode %}

### Data Endpoint

The `data` URL should return JSON matching your template variables:

{% code title="src/Controller/WidgetController.php" lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\Routing\Attribute\Route;

class WidgetController extends AbstractController
{
    #[Route('/api/widgets/weather', name: 'widget_weather')]
    public function weather(): JsonResponse
    {
        // Fetch real weather data from your service
        $weatherData = [
            'location' => 'Paris, France',
            'temperature' => 22,
            'conditions' => 'Partly Cloudy',
            'icon' => 'https://example.com/weather-icons/partly-cloudy.png',
            'humidity' => 65,
            'wind' => 15,
        ];

        return new JsonResponse($weatherData);
    }

    #[Route('/api/widgets/tasks', name: 'widget_tasks')]
    public function tasks(TaskRepository $taskRepository): JsonResponse
    {
        $user = $this->getUser();

        $pendingCount = $taskRepository->count([
            'user' => $user,
            'status' => 'pending'
        ]);

        return new JsonResponse([
            'pending' => $pendingCount,
            'message' => "You have {$pendingCount} pending tasks",
            'updated' => (new \DateTime())->format('H:i'),
        ]);
    }
}
```
{% endcode %}

## Service Worker Integration

The bundle automatically generates service worker code to handle widget installation and updates. The generated code:

1. **Handles widget installation** (`widgetinstall` event)
2. **Handles widget uninstallation** (`widgetuninstall` event)
3. **Registers periodic sync** for automatic updates
4. **Updates widget data** by fetching from the data endpoint

### How It Works

```javascript
// Automatically generated by the bundle

// When a widget is installed
self.addEventListener("widgetinstall", event => {
    event.waitUntil(onWidgetInstall(event.widget));
});

async function onWidgetInstall(widget) {
    // Register periodic update if configured
    const tags = await self.registration.periodicSync.getTags();
    if (!tags.includes(widget.definition.tag)) {
        await self.registration.periodicSync.register(widget.definition.tag, {
            minInterval: widget.definition.update
        });
    }
    await renderWidget(widget);
}

// Render/update widget with latest data
async function renderWidget(widget) {
    const templateUrl = widget.definition.msAcTemplate;
    const dataUrl = widget.definition.data;
    const template = await (await fetch(templateUrl)).text();
    const data = await (await fetch(dataUrl)).text();
    await self.widgets.updateByTag(widget.definition.tag, {template, data});
}

// Periodic updates
self.addEventListener("periodicsync", async event => {
    const widget = await self.widgets.getByTag(event.tag);
    if (widget && "update" in widget.definition) {
        event.waitUntil(renderWidget(widget));
    }
});
```

## Use Cases

### 1. Weather Widget

Show current weather conditions:

```yaml
widgets:
    - name: "Weather"
      description: "Current weather and forecast"
      tag: "weather"
      ms_ac_template: "widgets/weather.json"
      data: "api/widgets/weather"
      update: 1800  # Update every 30 minutes
      screenshots:
          - src: "screenshots/widget-weather.png"
            sizes: [300, 200]
```

### 2. Task Counter

Display pending tasks count:

```yaml
widgets:
    - name: "Tasks"
      description: "Number of pending tasks"
      tag: "tasks"
      ms_ac_template: "widgets/tasks.json"
      data: "api/widgets/tasks"
      auth: true
      update: 300  # Update every 5 minutes
      multiple: false  # Only one instance
      screenshots:
          - src: "screenshots/widget-tasks.png"
            sizes: [300, 200]
```

### 3. Calendar Events

Show upcoming calendar events:

```yaml
widgets:
    - name: "Calendar"
      description: "Today's events"
      tag: "calendar"
      ms_ac_template: "widgets/calendar.json"
      data: "api/widgets/calendar/today"
      auth: true
      update: 900  # Update every 15 minutes
      screenshots:
          - src: "screenshots/widget-calendar.png"
            sizes: [300, 200]
```

### 4. Stock Ticker

Display stock prices:

```yaml
widgets:
    - name: "Stock Ticker"
      short_name: "Stocks"
      description: "Real-time stock prices"
      tag: "stocks"
      ms_ac_template: "widgets/stocks.json"
      data: "api/widgets/stocks"
      update: 60  # Update every minute
      multiple: true  # Users can add multiple instances
      screenshots:
          - src: "screenshots/widget-stocks.png"
            sizes: [300, 200]
```

### 5. News Headlines

Show latest news:

```yaml
widgets:
    - name: "News Headlines"
      short_name: "News"
      description: "Latest breaking news"
      tag: "news"
      ms_ac_template: "widgets/news.json"
      data: "api/widgets/news"
      update: 600  # Update every 10 minutes
      screenshots:
          - src: "screenshots/widget-news.png"
            sizes: [300, 200]
```

## Widget Screenshots

Widget screenshots are **required** and show users what the widget looks like before installation.

### Screenshot Requirements

- **Size**: Typically 300×200 or similar aspect ratio
- **Format**: PNG or JPEG
- **Content**: Show the actual widget appearance
- **At least one screenshot** is required per widget

## Authentication

If your widget requires user authentication, set `auth: true`:

```yaml
widgets:
    - name: "Private Widget"
      description: "User-specific data"
      tag: "private"
      auth: true  # Requires user to be logged in
      ms_ac_template: "widgets/private.json"
      data: "api/widgets/private"
```

**Important**: Ensure your data endpoint checks authentication:

```php
#[Route('/api/widgets/private', name: 'widget_private')]
public function privateWidget(): JsonResponse
{
    $this->denyAccessUnlessGranted('IS_AUTHENTICATED_FULLY');

    $user = $this->getUser();

    return new JsonResponse([
        'user' => $user->getName(),
        'data' => $this->getPrivateData($user),
    ]);
}
```

## Update Frequency

The `update` property controls how often the widget refreshes (in seconds):

| Frequency | Seconds | Use Case |
|-----------|---------|----------|
| 1 minute | 60 | Stock prices, live scores |
| 5 minutes | 300 | Task counts, notifications |
| 15 minutes | 900 | Calendar events, messages |
| 30 minutes | 1800 | Weather, news headlines |
| 1 hour | 3600 | Daily stats, summaries |

{% hint style="warning" %}
**Battery Consideration**: Frequent updates consume battery. Use the longest reasonable interval for your use case.
{% endhint %}

## Multiple Instances

Set `multiple: true` to allow users to install multiple instances of the same widget:

```yaml
widgets:
    - name: "Stock Ticker"
      tag: "stocks"
      multiple: true  # User can add multiple stock tickers
```

**Use cases for multiple instances**:
- Different stock symbols
- Multiple locations (weather)
- Different calendars
- Various project task boards

## Testing

### 1. Feature Detection

Always check if widgets are supported:

```javascript
if ('widgets' in self && self.widgets) {
    // Widgets are supported
    console.log('Widgets API available');
} else {
    console.log('Widgets not supported on this platform');
}
```

### 2. Test on Windows 10/11

1. Install Microsoft Edge (latest version)
2. Install your PWA
3. Open Windows widgets panel (Win + W)
4. Look for your PWA in the widget picker
5. Add widget and verify it displays correctly

### 3. Test Widget Updates

```javascript
// In service worker
async function testWidgetUpdate() {
    const widget = await self.widgets.getByTag('weather');
    if (widget) {
        console.log('Widget found:', widget);
        await renderWidget(widget);
        console.log('Widget updated successfully');
    }
}
```

### 4. DevTools Debugging

Monitor widget events in the service worker console:

```javascript
self.addEventListener("widgetinstall", event => {
    console.log('Widget installed:', event.widget.definition);
    event.waitUntil(onWidgetInstall(event.widget));
});

self.addEventListener("widgetuninstall", event => {
    console.log('Widget uninstalled:', event.widget.definition);
});
```

## Adaptive Cards Designer

Use Microsoft's Adaptive Cards Designer to create and test your templates:

**Designer URL**: https://adaptivecards.io/designer/

1. Design your widget layout
2. Add data binding variables (`${variableName}`)
3. Export the JSON template
4. Save to `public/widgets/your-template.json`
5. Reference in PWA configuration

## Best Practices

### 1. Keep Data Fresh

```php
#[Route('/api/widgets/stats')]
public function stats(): JsonResponse
{
    $response = new JsonResponse([
        'stats' => $this->getLatestStats(),
        'timestamp' => time(),
    ]);

    // Cache for short duration
    $response->setSharedMaxAge(60); // 1 minute

    return $response;
}
```

### 2. Handle Errors Gracefully

```javascript
async function renderWidget(widget) {
    try {
        const template = await (await fetch(widget.definition.msAcTemplate)).text();
        const data = await (await fetch(widget.definition.data)).text();
        await self.widgets.updateByTag(widget.definition.tag, {template, data});
    } catch (error) {
        console.error('Widget update failed:', error);
        // Widget will keep showing last successful data
    }
}
```

### 3. Optimize Template Size

- Keep templates under 50KB
- Minimize nested structures
- Use concise variable names
- Remove unnecessary whitespace in production

### 4. Test with Sample Data

Provide fallback data for development:

```json
{
    "location": "${location|Paris}",
    "temperature": "${temperature|20}",
    "conditions": "${conditions|Sunny}"
}
```

### 5. Localization

Widgets support translations via the bundle's translation system:

```yaml
widgets:
    - name: "widget.weather.name"
      description: "widget.weather.description"
      # ...
```

```yaml
# translations/messages.en.yaml
widget:
    weather:
        name: "Weather"
        description: "Current weather conditions"

# translations/messages.fr.yaml
widget:
    weather:
        name: "Météo"
        description: "Conditions météorologiques actuelles"
```

## Common Issues

### Widget Not Appearing

**Problem**: Widget doesn't show up in Windows widgets panel

**Solutions**:
- Verify you're using Microsoft Edge on Windows 10/11
- Ensure PWA is installed, not just running in browser tab
- Check that `screenshots` array has at least one entry
- Verify manifest is valid JSON

### Widget Not Updating

**Problem**: Widget shows stale data

**Solutions**:
- Check service worker is active (not waiting)
- Verify `update` interval is set
- Check data endpoint is returning fresh data
- Monitor service worker console for errors
- Ensure periodic sync is registered

### Template Not Rendering

**Problem**: Widget shows blank or error

**Solutions**:
- Validate Adaptive Card JSON at https://adaptivecards.io/designer/
- Check template URL is accessible
- Verify data variables match template placeholders
- Check data endpoint returns valid JSON
- Test template with sample data

### Authentication Issues

**Problem**: Widget fails to load user-specific data

**Solutions**:
- Ensure `auth: true` is set
- Check cookies/session is maintained
- Verify credentials are included in fetch
- Add authentication headers if needed

## Limitations

1. **Platform-specific**: Only works on Microsoft Edge on Windows 10+
2. **No interactivity**: Widgets are read-only, no buttons or input
3. **Size constraints**: Limited to predefined widget sizes
4. **Update frequency**: Minimum update interval enforced by OS
5. **Data size**: Keep data payloads small (< 100KB)

## Related Documentation

- [Periodic Background Sync](../the-service-worker/periodic-sync.md) - Powers widget updates
- [Screenshots](../the-manifest/screenshots.md) - Widget preview images
- [Icons](../the-manifest/icons.md) - Widget icons
- [Service Worker](../the-service-worker/configuration.md) - Service worker configuration

## Resources

- **Microsoft Docs**: https://learn.microsoft.com/en-us/microsoft-edge/progressive-web-apps-chromium/how-to/widgets
- **Adaptive Cards**: https://adaptivecards.io/
- **Adaptive Cards Designer**: https://adaptivecards.io/designer/
- **Adaptive Cards Schema**: https://adaptivecards.io/explorer/
- **Windows Widgets API**: https://learn.microsoft.com/en-us/microsoft-edge/progressive-web-apps-chromium/how-to/widgets

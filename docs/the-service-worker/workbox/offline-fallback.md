# Offline Fallbacks

Offline fallbacks provide graceful degradation when users lose network connectivity. Instead of showing browser error pages, your PWA can display custom fallback content for pages, images, and fonts.

## Why Use Offline Fallbacks?

When network requests fail and there's no cached content:
- **Pages**: Show a friendly "You're offline" page instead of a browser error
- **Images**: Display a placeholder image instead of broken image icons
- **Fonts**: Serve a fallback font instead of rendering with system fonts

This creates a professional offline experience that maintains your app's branding and provides clear feedback to users.

## How It Works

Workbox's offline fallback plugin:
1. Attempts to serve the requested resource from cache or network
2. If both fail (offline + no cache), serves the appropriate fallback
3. Fallbacks are precached during service worker installation
4. Works independently for pages, images, and fonts

## Configuration

### Basic Setup

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            offline_fallback:
                page: 'app_offline_page'
                image: 'images/offline.svg'
                font: 'fonts/fallback.woff2'
```
{% endcode %}

### Configuration Options

#### page (Page Fallback)

**Type**: `string` or `Url` object
**Default**: `null`

The page to display when a navigation request fails offline.

**Simple string (route name)**:
```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                page: 'app_offline'  # Route name
```

**Relative URL**:
```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                page: '/offline.html'
```

**Route with parameters**:
```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                page:
                    path: 'app_offline'
                    params:
                        utm_source: 'offline_fallback'
```

See [URL parameter documentation](../../the-manifest/shortcuts.md#url-parameter) for all URL configuration options.

#### image (Image Fallback)

**Type**: `string` (Asset path or URL)
**Default**: `null`

The image to display when an image request fails offline.

**Asset Mapper path**:
```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                image: 'images/offline-placeholder.svg'
```

**Absolute URL**:
```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                image: '/assets/images/no-connection.png'
```

The image file must be accessible via Asset Mapper or as a public URL.

#### font (Font Fallback)

**Type**: `string` (Asset path or URL)
**Default**: `null`

The font to serve when a font request fails offline.

**Asset Mapper path**:
```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                font: 'fonts/roboto-regular.woff2'
```

**Absolute URL**:
```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                font: '/assets/fonts/fallback.ttf'
```

#### cache_name (Cache Name)

**Type**: `string`
**Default**: `null` (auto-generated)

Custom name for the offline fallbacks cache.

```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                cache_name: 'offline-content'
                page: 'app_offline'
                image: 'images/offline.svg'
```

## Complete Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            offline_fallback:
                cache_name: 'offline-fallbacks'
                page:
                    path: 'app_offline_page'
                    params:
                        source: 'service_worker'
                image: 'images/offline-placeholder.svg'
                font: 'fonts/system-fallback.woff2'
```
{% endcode %}

## Creating Offline Fallback Pages

### Simple Offline Page

Create a Symfony route and template for the offline page:

**Route** (`config/routes.yaml`):
```yaml
app_offline:
    path: /offline
    controller: App\Controller\OfflineController::index
```

**Controller** (`src/Controller/OfflineController.php`):
```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class OfflineController extends AbstractController
{
    #[Route('/offline', name: 'app_offline')]
    public function index(): Response
    {
        return $this->render('offline/index.html.twig');
    }
}
```

**Template** (`templates/offline/index.html.twig`):
```twig
{% extends 'base.html.twig' %}

{% block title %}You're Offline{% endblock %}

{% block body %}
<div class="offline-page">
    <div class="offline-icon">
        <svg width="100" height="100" viewBox="0 0 100 100">
            <!-- Cloud icon with X -->
            <path d="M..." fill="#ccc"/>
        </svg>
    </div>

    <h1>You're Currently Offline</h1>

    <p>
        It looks like you've lost your internet connection.
        Don't worry, you can still access cached content.
    </p>

    <div class="offline-actions">
        <button onclick="location.reload()">Try Again</button>
        <a href="{{ path('app_homepage') }}">Go to Homepage</a>
    </div>
</div>

<style>
    .offline-page {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        min-height: 60vh;
        text-align: center;
        padding: 2rem;
    }

    .offline-icon {
        margin-bottom: 2rem;
        opacity: 0.5;
    }

    .offline-actions {
        margin-top: 2rem;
        display: flex;
        gap: 1rem;
    }

    .offline-actions button,
    .offline-actions a {
        padding: 0.75rem 1.5rem;
        border-radius: 0.5rem;
        text-decoration: none;
    }
</style>
{% endblock %}
```

### Advanced Offline Page with Cached Content List

Show users what content is available offline:

```twig
{% extends 'base.html.twig' %}

{% block body %}
<div class="offline-page">
    <h1>You're Offline</h1>
    <p>But you can still access these cached pages:</p>

    <div id="cached-pages-list">
        <p>Loading cached pages...</p>
    </div>

    <button onclick="location.reload()">Try to Reconnect</button>
</div>

<script>
// List cached pages
if ('caches' in window) {
    caches.keys().then(cacheNames => {
        return Promise.all(
            cacheNames.map(cacheName => caches.open(cacheName))
        );
    }).then(caches => {
        const allUrls = [];
        return Promise.all(
            caches.map(cache =>
                cache.keys().then(requests =>
                    requests.map(req => req.url)
                )
            )
        );
    }).then(urlArrays => {
        const urls = urlArrays.flat().filter(url =>
            url.includes(window.location.origin) && !url.includes('offline')
        );

        const list = document.getElementById('cached-pages-list');
        if (urls.length) {
            list.innerHTML = '<ul>' +
                urls.map(url => `<li><a href="${url}">${new URL(url).pathname}</a></li>`).join('') +
                '</ul>';
        } else {
            list.innerHTML = '<p>No cached pages available</p>';
        }
    });
}
</script>
{% endblock %}
```

## Creating Fallback Images

### SVG Offline Placeholder

Create a simple SVG offline image (`assets/images/offline.svg`):

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="400" height="300" viewBox="0 0 400 300">
    <rect width="400" height="300" fill="#f5f5f5"/>
    <text x="200" y="140" font-family="Arial" font-size="24" fill="#999" text-anchor="middle">
        Image Unavailable Offline
    </text>
    <path d="M180 160 L220 200 M220 160 L180 200" stroke="#999" stroke-width="3"/>
</svg>
```

### PNG Placeholder

For a more visual approach, create a branded offline placeholder image that matches your app's design.

## Use Cases

### Blog or News Site

```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                page: 'app_offline'
                image: 'images/article-placeholder.jpg'
```

### E-commerce Application

```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                page: 'app_offline_shop'
                image: 'images/product-placeholder.svg'
```

### Documentation Site

```yaml
pwa:
    serviceworker:
        workbox:
            offline_fallback:
                page: 'app_offline_docs'
                # No image fallback needed - docs don't rely on images
```

## Testing Offline Fallbacks

### In Chrome DevTools

1. Open DevTools (F12)
2. Go to **Application** tab
3. In **Service Workers** section, check **Offline**
4. Navigate to a page not in cache
5. Should see your offline fallback page

### Clear Cache to Test

```javascript
// In DevTools Console
caches.keys().then(keys => {
    keys.forEach(key => caches.delete(key));
});
```

Then test offline navigation.

## Best Practices

1. **Keep it simple**: Offline pages should have minimal dependencies
2. **Inline CSS**: Don't rely on external stylesheets that might not be cached
3. **Clear messaging**: Explain the offline state clearly to users
4. **Provide actions**: Include buttons to retry or navigate to cached content
5. **Match branding**: Keep the offline experience consistent with your app's design
6. **Test thoroughly**: Test various offline scenarios (page navigation, image loading, etc.)
7. **Monitor cache**: Ensure fallback resources are actually cached

## Fallback Priority

When a request fails:

1. **Try cache first**: Check if resource is in any cache
2. **Try network**: Attempt network request
3. **Use fallback**: If both fail, serve the appropriate fallback

This ensures fallbacks are truly a last resort.

{% hint style="info" %}
**Tip**: Combine offline fallbacks with [resource caching](resource-caching.md) and [precaching](asset-caching.md) strategies for the best offline experience.
{% endhint %}

## Troubleshooting

**Fallback not showing?**
- Verify fallback files exist and are accessible
- Check that files are being precached (DevTools → Application → Cache Storage)
- Ensure service worker is activated
- Test in incognito mode to rule out caching issues

**Fallback showing when it shouldn't?**
- Check your network conditions
- Verify cache strategies aren't too aggressive
- Look for JavaScript errors preventing network requests

{% hint style="success" %}
Start with just a page fallback. Once that works well, add image and font fallbacks as needed. Most apps only need a page fallback.
{% endhint %}

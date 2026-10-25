# How To Create A PWA?

Creating a Progressive Web App is a **progressive** process - you can start with basic features and enhance your application incrementally. Two core components make a web application "progressive": the **Web App Manifest** and the **Service Worker**.

These components work independently, meaning you can implement one without the other:
- **Manifest only**: Your app becomes installable but won't work offline
- **Service Worker only**: Your app works offline but may not be installable
- **Both together**: Full PWA experience with installation and offline capabilities

## The Two Pillars of PWAs

### 1. The Web App Manifest

The manifest is a JSON file that tells browsers how your application should behave when installed on a device. It contains metadata about your app:

**What it controls:**
- App name and description
- Icons for different platforms and sizes
- Theme and background colors
- Display mode (fullscreen, standalone, minimal UI)
- Screen orientation preferences
- Start URL when launched

**Why it matters:**
- Enables "Add to Home Screen" functionality
- Controls how your app appears when launched
- Provides app store-like information
- Creates a native app-like experience

**Example manifest file:**

{% code title="/site.webmanifest" lineNumbers="true" %}
```json
{
  "name": "My Progressive Web App",
  "short_name": "PWA",
  "description": "An example Progressive Web App",
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4285f4",
  "icons": [
    {
      "src": "/icons/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/icons/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ]
}
```
{% endcode %}

**Key properties explained:**
- `name` / `short_name`: Full and abbreviated app names
- `description`: Brief description of your app (shown during installation)
- `start_url`: URL to open when the app launches
- `display: "standalone"`: Hides browser UI for app-like experience
- `background_color`: Splash screen background during launch
- `theme_color`: Browser toolbar color
- `icons`: Different sizes for various platforms (192x192, 512x512 minimum required)

**Integrating the manifest:**

Add this link tag to your HTML `<head>` section:

{% code lineNumbers="true" %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
  ...
  <link rel="manifest" href="/manifest.json">
</head>
<body>
  ...
</body>
</html>

```
{% endcode %}

{% hint style="success" %}
With this bundle, the manifest is automatically generated and injected - you don't need to create or link it manually!
{% endhint %}

### 2. The Service Worker

A service worker is a JavaScript file that runs in the background, separate from your web page. It acts as a programmable proxy between your app and the network.

**What it does:**
- Intercepts network requests from your app
- Caches resources (HTML, CSS, JS, images, API responses)
- Serves cached content when offline
- Manages background synchronization
- Handles push notifications
- Updates content in the background

**Why it matters:**
- Enables offline functionality
- Dramatically improves loading speed
- Reduces server load through intelligent caching
- Provides reliable performance on slow networks
- Powers advanced features like background sync and push notifications

**Example service worker:**

{% code title="/service-worker.js" lineNumbers="true" %}
```javascript
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('my-cache').then((cache) => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles/main.css',
        '/scripts/main.js',
        '/images/logo.png'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
```
{% endcode %}

**How this service worker works:**

1. **Install event**: Runs when the service worker is first installed
   - Opens a cache named 'my-cache'
   - Stores essential files for offline use

2. **Fetch event**: Intercepts every network request
   - Checks if the request exists in the cache
   - Returns cached version if available
   - Falls back to network if not cached

**Registering the service worker:**

Add this script to your HTML to activate the service worker:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <script defer>
    if ('serviceWorker' in navigator) {
      navigator.serviceWorker.register('/service-worker.js')
        .then(function(registration) {
          console.log('Service Worker registered with scope:', registration.scope);
        })
        .catch(function(error) {
          console.error('Service Worker registration failed:', error);
        });
    }
  </script>
</head>
<body>
  ...
</body>
</html>

```

{% hint style="success" %}
This bundle automatically generates and registers the service worker for you, with advanced caching strategies powered by Google Workbox!
{% endhint %}

## How This Bundle Simplifies PWA Development

Instead of manually creating manifest files and service workers, this bundle:

1. **Generates the manifest** based on your configuration
2. **Creates the service worker** with optimized caching strategies
3. **Handles asset compilation** (icons, screenshots, favicons)
4. **Manages updates** automatically
5. **Provides debugging tools** via the Symfony profiler

**Your role:** Configure your PWA settings in `config/packages/pwa.yaml` and add `{{ pwa() }}` to your templates.

**The bundle handles:** Everything else - manifest generation, service worker creation, asset optimization, and deployment.

## Getting Started

Ready to create your PWA?

1. [Install the bundle](installation.md) - Quick setup guide
2. [Configure your manifest](the-manifest/application-information/) - Customize app metadata
3. [Set up caching](the-service-worker/workbox/) - Configure offline behavior
4. [Add PWA features](symfony-ux/) - Enhance with device APIs

Your Symfony application will be a fully functional Progressive Web App in minutes!

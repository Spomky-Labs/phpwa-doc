---
description: Transform your Symfony application into a Progressive Web App
---

# PWA Bundle

<figure><picture><source srcset=".gitbook/assets/pwalogo-inverse.svg" media="(prefers-color-scheme: dark)"><img src=".gitbook/assets/pwalogo.svg" alt="PWA Logo"></picture><figcaption></figcaption></figure>

Welcome! 👋

This bundle enables you to transform your Symfony application into a Progressive Web App (PWA) with minimal configuration. Whether you're new to PWAs or looking to enhance your existing Symfony app, you're in the right place.

## What is a Progressive Web App?

A Progressive Web App combines the best of web and mobile applications. It's a web application that behaves like a native mobile app while remaining accessible through a standard web browser.

Think of PWAs as websites that can:
- Be installed on devices like regular apps
- Work offline or with poor network conditions
- Send push notifications to users
- Access device features (camera, geolocation, etc.)
- Load instantly, even on slow networks

## Why Choose PWAs?

### Development Benefits

**Single Codebase**
- Write once, deploy everywhere (iOS, Android, desktop, web)
- Lower development and maintenance costs compared to native apps
- No need to learn platform-specific languages (Swift, Kotlin, etc.)

**Independent Distribution**
- Deploy directly to your web server
- No app store approval process or fees
- Update your app anytime without waiting for reviews
- Complete control over your release schedule

### User Experience Benefits

**App-Like Experience**
- Install from browser with one click
- Launch from home screen or app menu
- Fullscreen mode without browser UI
- Smooth animations and responsive interactions

**Always Available**
- Function offline or with poor connectivity
- Fast loading times with intelligent caching
- Automatic background updates
- Access to content even without internet

**Rich Engagement**
- Push notifications keep users informed
- Home screen icon for easy access
- Badge notifications for unread content
- Background synchronization when connectivity returns

### Business Benefits

**Better Discoverability**
- Fully indexed by search engines (SEO)
- Shareable via simple URL
- No installation barrier for first-time visitors
- Gradual feature adoption based on user engagement

**Performance & Reliability**
- Faster than traditional websites
- Resilient to network failures
- Reduced server load with caching
- Lower bandwidth consumption for users

## What Does This Bundle Do?

This Symfony bundle handles all the technical complexity of creating a PWA for you:

### Automatic Manifest Generation
- Creates and serves the Web App Manifest (JSON file)
- Manages app metadata (name, icons, theme colors, etc.)
- Handles multi-language support
- Configures installation behavior

### Service Worker Management
- Generates and compiles the service worker script
- Integrates Google Workbox for advanced caching strategies
- Manages offline fallbacks and resource caching
- Handles background synchronization

### Asset Optimization
- Generates favicons and app icons automatically
- Creates startup images for iOS
- Optimizes screenshots for app stores
- Manages cache versioning and updates

### Symfony UX Integration
- Provides ready-to-use Stimulus controllers for PWA features
- Easy integration with Symfony Live Components
- Access to device APIs (geolocation, camera, notifications, etc.)
- Pre-built components for common PWA patterns

### Developer Tools
- Debug toolbar integration for manifest inspection
- Service worker debugging support
- Event system for customization
- PSR-3 logging for troubleshooting

## Quick Start

Get your PWA up and running in minutes:

```bash
# Install the bundle
composer require spomky-labs/pwa-bundle

# Add one line to your base template
{{ pwa() }}

# Configure your app
# Edit config/packages/pwa.yaml
```

That's it! Your Symfony application is now a Progressive Web App.

## Real-World Use Cases

PWAs built with Symfony are perfect for:

**E-Commerce & Retail**
- Offline product browsing
- Shopping cart persistence
- Order status notifications
- Quick reordering from home screen

**Content & Media**
- Offline article reading
- Background content sync
- Push notification for new content
- Media file caching

**Business Applications**
- CRM and sales tools
- Field service apps
- Inventory management
- Expense reporting

**Social & Community**
- Social networks
- Forum and discussion platforms
- Event management
- Real-time messaging

**Productivity Tools**
- Todo lists and task managers
- Note-taking applications
- Time tracking
- Document editors

## Next Steps

Ready to transform your Symfony app into a PWA?

1. [Install the bundle](installation.md) - Get started in minutes
2. [Learn PWA concepts](how-to-create-a-pwa.md) - Understand the fundamentals
3. [Configure the manifest](the-manifest/application-information/) - Customize your app
4. [Set up the service worker](the-service-worker/configuration.md) - Enable offline mode
5. [Add UX components](symfony-ux/) - Enhance user experience

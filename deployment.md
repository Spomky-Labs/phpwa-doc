# Deployment

Deploying a PWA requires compiling assets to ensure optimal performance in production. This guide covers both deployment methods: with and without Symfony's Asset Mapper.

## Understanding PWA Asset Compilation

### Development vs Production

**Development (`APP_ENV=dev`):**
- Assets generated on-the-fly
- Request listener serves resources dynamically
- Debugging enabled (verbose logging, comments)
- No optimization or minification

**Production (`APP_ENV=prod`):**
- All assets pre-compiled and optimized
- Resources served as static files
- Debugging disabled
- Minified and optimized for performance

### What Gets Compiled

The compilation process generates:

1. **Web App Manifest** (`manifest.json`)
   - With correct production URLs
   - Optimized and minified

2. **Service Worker** (`sw.js`)
   - Complete with Workbox integration
   - Cache strategies configured
   - Optimized and minified

3. **Icons and Favicons**
   - All required sizes (16x16 to 512x512)
   - Platform-specific formats (PNG, ICO, SVG)
   - Optimized file sizes

4. **Screenshots** (if configured)
   - App store screenshots
   - Various device sizes

5. **Dependencies**
   - Workbox libraries
   - IndexedDB polyfills
   - Other runtime dependencies

## Deployment with Asset Mapper

If you're using Symfony's Asset Mapper (recommended), PWA assets are compiled together with your application assets.

### Step 1: Configure Asset Compilation

Ensure asset compilation is enabled (this is the default):

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    asset_compiler: true  # Default in 1.x
```
{% endcode %}

### Step 2: Compile Assets

Run the standard Asset Mapper compilation command before deployment:

```bash
php bin/console asset-map:compile
```

This single command:
- Compiles all application assets
- Generates PWA resources (manifest, service worker, icons)
- Optimizes and minifies everything
- Prepares assets for production

### Step 3: Deploy

Deploy your application with the compiled assets:

```bash
# Example deployment workflow
php bin/console asset-map:compile
git add public/assets
git commit -m "Compile assets for production"
git push production main
```

{% hint style="success" %}
Asset Mapper integration is the simplest approach - everything is handled in one command!
{% endhint %}

## Deployment without Asset Mapper

If you're not using Asset Mapper or prefer standalone PWA asset compilation, use the dedicated `pwa:compile` command.

### Step 1: Disable Asset Mapper Integration

Configure the bundle to handle compilation independently:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    asset_compiler: false
```
{% endcode %}

{% hint style="info" %}
In version 1.x, `asset_compiler` defaults to `true`. In version 2.x, it will default to `false`, requiring explicit Asset Mapper integration.
{% endhint %}

### Step 2: Compile PWA Assets

Run the PWA-specific compilation command:

```bash
php bin/console pwa:compile
```

This compiles all PWA resources exactly as Asset Mapper would, but independently.

### Step 3: Deploy

Deploy your application with the compiled PWA assets.

## Context-Aware Deployment

If your application URLs depend on the deployment environment (e.g., different domains for staging and production), you need context-aware compilation.

### The Problem

The manifest and service worker include absolute URLs. If you compile assets locally, they might reference `http://localhost` instead of your production domain `https://example.com`.

**Example issue:**
```json
{
  "start_url": "http://localhost/app",  // ❌ Wrong in production
  "scope": "http://localhost/"          // ❌ Wrong in production
}
```

### Solution 1: Compile in Production Environment

The best approach is to compile assets in the production environment where the correct context is available:

```bash
# During deployment on production server
APP_ENV=prod APP_DEFAULT_URI=https://example.com php bin/console pwa:compile
```

### Solution 2: Two-Step Compilation

If you must compile assets before deployment (e.g., in CI/CD), use a two-step process:

**Step 1: Initial compilation (local or CI)**
```bash
# Compile all assets (including time-consuming icon generation)
php bin/console pwa:compile
```

**Step 2: Context-only recompilation (on production)**
```bash
# Only recompile context-dependent files (manifest and service worker)
APP_ENV=prod php bin/console pwa:compile --context-only
```

The `--context-only` flag:
- Skips icon generation (already done in step 1)
- Only regenerates manifest and service worker
- Uses correct production URLs
- Completes in seconds instead of minutes

{% hint style="success" %}
This two-step approach balances build time (icons generated once) with URL correctness (manifest uses production URLs).
{% endhint %}

### Setting Application Context

Ensure your production environment has the correct context configured:

{% code title="config/packages/framework.yaml" lineNumbers="true" %}
```yaml
# config/packages/prod/framework.yaml
framework:
    router:
        default_uri: 'https://example.com'
```
{% endcode %}

Or set via environment variable:

```bash
# .env.prod
APP_DEFAULT_URI=https://example.com
```

## Deployment Checklist

Before deploying your PWA to production:

- [ ] Assets compiled (`asset-map:compile` or `pwa:compile`)
- [ ] Manifest URLs point to production domain
- [ ] Service worker registered correctly
- [ ] Icons generated for all required sizes
- [ ] HTTPS enabled (required for PWA features)
- [ ] Cache versioning strategy in place
- [ ] Offline fallback page configured
- [ ] Browser compatibility tested
- [ ] Installation tested on target devices

## Continuous Integration Example

Here's an example GitLab CI/CD configuration:

{% code title=".gitlab-ci.yml" lineNumbers="true" %}
```yaml
stages:
  - build
  - deploy

build:
  stage: build
  script:
    - composer install --no-dev --optimize-autoloader
    - php bin/console pwa:compile
  artifacts:
    paths:
      - public/

deploy:production:
  stage: deploy
  script:
    - rsync -avz public/ user@server:/var/www/html/
    - ssh user@server "cd /var/www/html && php bin/console pwa:compile --context-only"
  only:
    - main
```
{% endcode %}

## Verifying Deployment

After deployment, verify your PWA is working correctly:

### Check Manifest

```bash
curl https://example.com/manifest.json
```

Verify:
- All URLs use production domain
- Icons paths are correct
- No localhost references

### Check Service Worker

```bash
curl https://example.com/sw.js
```

Verify:
- File exists and loads
- No debugging code (in production)
- Minified and optimized

### Browser Testing

1. Open DevTools → Application tab
2. Check **Manifest** section - no errors
3. Check **Service Workers** - registered and active
4. Test **Install** prompt appears
5. Test **Offline** functionality

## Troubleshooting Deployment

### Assets Not Found (404)

**Problem:** PWA assets return 404 errors

**Solutions:**
1. Ensure compilation ran successfully
2. Check `public/` directory contains compiled assets
3. Verify web server serves static files from `public/`
4. Clear application cache: `php bin/console cache:clear`

### Wrong URLs in Manifest

**Problem:** Manifest contains localhost or incorrect URLs

**Solutions:**
1. Set `default_uri` in framework config
2. Use `--context-only` flag during production deployment
3. Compile assets in production environment

### Service Worker Not Updating

**Problem:** Old service worker version remains active

**Solutions:**
1. Update cache version in configuration
2. Change service worker file name/path
3. Implement `skipWaiting: true` for immediate updates
4. Clear browser service worker cache

### Icons Not Displaying

**Problem:** App icons don't appear or are broken

**Solutions:**
1. Verify icon files exist in `public/` directory
2. Check icon paths in manifest are absolute
3. Ensure icons are accessible (not blocked by firewall)
4. Regenerate icons: `php bin/console pwa:compile`

## Performance Optimization

For optimal PWA performance in production:

1. **Enable CDN** for static assets
2. **Configure caching headers** for PWA resources
3. **Implement versioning** for cache invalidation
4. **Compress assets** (gzip/brotli)
5. **Use HTTP/2** for faster resource loading

Example Nginx configuration:

```nginx
location /manifest.json {
    expires 1d;
    add_header Cache-Control "public, immutable";
}

location /sw.js {
    expires 1h;
    add_header Cache-Control "public";
    add_header Service-Worker-Allowed "/";
}
```

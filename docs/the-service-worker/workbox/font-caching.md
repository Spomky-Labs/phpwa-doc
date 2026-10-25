# Font Caching

Font caching ensures custom fonts load quickly and remain available offline. The bundle provides two separate caching mechanisms: one for self-hosted fonts and another optimized for Google Fonts.

## Why Cache Fonts?

Fonts significantly impact page rendering and user experience:
- **Prevent FOIT**: Font files in cache eliminate "Flash of Invisible Text"
- **Faster rendering**: No waiting for font downloads
- **Offline availability**: Fonts work without network connection
- **Reduced bandwidth**: Fonts downloaded once and reused
- **Better typography**: Consistent font rendering across sessions

## Self-Hosted Font Caching

### Default Configuration

The bundle automatically caches self-hosted fonts with these settings:
- **Max entries**: 30 fonts
- **Max age**: 365 days (1 year)
- **Strategy**: CacheFirst
- **Supported formats**: `.ttf`, `.eot`, `.otf`, `.woff`, `.woff2`

**Pattern matched**:
```regex
/\.(ttf|eot|otf|woff|woff2?)$/
```

{% hint style="info" %}
Font caching only applies to fonts served by your application, not external CDNs (except Google Fonts, which has dedicated handling).
{% endhint %}

### Configuration Options

#### Enabling/Disabling

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                enabled: true  # Default: true
```

#### Max Entries

**Type**: `integer`
**Default**: `30`

Maximum number of font files to cache.

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                max_entries: 50  # Cache up to 50 font files
```

**Recommendations**:
- **Small projects**: 5-10 fonts (1-2 font families)
- **Medium projects**: 20-30 fonts (multiple weights/styles)
- **Large projects**: 50+ fonts (many families or international support)

#### Max Age

**Type**: `integer` (seconds) or `string` (human-readable)
**Default**: `31536000` (365 days)

How long fonts remain in cache before refresh.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                enabled: true
                max_entries: 20
                max_age: 180 days  # 6 months
```
{% endcode %}

**Font-specific considerations**:
- **Custom brand fonts**: 1 year or more (rarely change)
- **Purchased fonts**: 6-12 months
- **Frequently updated fonts**: 1-3 months

#### Custom Regex

**Type**: `string`
**Default**: `/\.(ttf|eot|otf|woff|woff2?)$/`

Customize which font file patterns to cache.

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                regex: '/\.(woff2?)$/'  # Only WOFF and WOFF2
```

**Use cases**:
- Limit to modern formats only (WOFF2)
- Match specific paths (`/\/fonts\/.*\.woff2$/`)
- Include/exclude specific font directories

#### Cache Name

**Type**: `string`
**Default**: `null` (auto-generated)

Custom cache name for easier management.

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                cache_name: 'app-fonts'
```

### Complete Self-Hosted Example

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            font_cache:
                enabled: true
                cache_name: 'custom-fonts'
                max_entries: 25
                max_age: 180 days
                regex: '/\.(woff2?|ttf)$/'
```
{% endcode %}

## Google Fonts Caching

Google Fonts requires special handling because fonts are loaded from two sources:
1. **CSS file** from fonts.googleapis.com
2. **Font files** from fonts.gstatic.com

The bundle handles both automatically.

### Default Google Fonts Configuration

- **Enabled**: Yes (by default)
- **Cache prefix**: `google-fonts`
- **Max entries**: 20 font files
- **Max age**: 86400 seconds (1 day) for CSS, 31536000 seconds (1 year) for font files
- **Strategy**: StaleWhileRevalidate for CSS, CacheFirst for fonts

### Configuration Options

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            google_fonts:
                enabled: true
                cache_prefix: 'google-fonts'  # Cache name prefix
                max_entries: 30               # Font files to cache
                max_age: 2 days               # CSS cache duration
```
{% endcode %}

{% hint style="warning" %}
**Typo in original docs**: The cache_prefix was "goolge-fonts" but should be "google-fonts"
{% endhint %}

### Google Fonts Options Explained

#### cache_prefix

**Type**: `string`
**Default**: `'google-fonts'`

Prefix for the two Google Fonts caches:
- `{prefix}-stylesheets`: CSS files cache
- `{prefix}-webfonts`: Font files cache

```yaml
pwa:
    serviceworker:
        workbox:
            google_fonts:
                cache_prefix: 'gfonts'
                # Creates: gfonts-stylesheets and gfonts-webfonts
```

#### max_entries

**Type**: `integer`
**Default**: `20`

Maximum number of Google Font files to cache (not CSS files).

```yaml
pwa:
    serviceworker:
        workbox:
            google_fonts:
                max_entries: 40  # For apps using many Google Fonts
```

#### max_age

**Type**: `integer` (seconds) or `string` (human-readable)
**Default**: `86400` (1 day)

How long to cache the Google Fonts CSS files (font files cached for 1 year automatically).

```yaml
pwa:
    serviceworker:
        workbox:
            google_fonts:
                max_age: 7 days  # Cache CSS for a week
```

### Disabling Google Fonts Caching

If you don't use Google Fonts:

```yaml
pwa:
    serviceworker:
        workbox:
            google_fonts:
                enabled: false
```

## Complete Font Caching Example

Configuration for an app using both self-hosted and Google Fonts:

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            # Self-hosted fonts
            font_cache:
                enabled: true
                cache_name: 'brand-fonts'
                max_entries: 15
                max_age: 1 year
                regex: '/\/fonts\/.*\.(woff2|ttf)$/'

            # Google Fonts
            google_fonts:
                enabled: true
                cache_prefix: 'google-fonts'
                max_entries: 25
                max_age: 3 days
```
{% endcode %}

## Use Cases

### Modern Web Font Stack

Using only WOFF2 for modern browsers:

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                enabled: true
                regex: '/\.woff2$/'
                max_entries: 20
                max_age: 1 year
```

### International Application

Supporting many font families and weights:

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                max_entries: 100  # Many languages = many fonts
                max_age: 6 months
            google_fonts:
                enabled: true
                max_entries: 50
```

### Google Fonts Only

No self-hosted fonts:

```yaml
pwa:
    serviceworker:
        workbox:
            font_cache:
                enabled: false
            google_fonts:
                enabled: true
                max_entries: 30
```

## How It Works

### Self-Hosted Fonts Flow

1. **First request**: Font file downloaded and cached
2. **Subsequent requests**: Served instantly from cache (CacheFirst)
3. **Cache full**: Oldest fonts removed when `max_entries` reached
4. **Expiration**: After `max_age`, fonts refreshed on next request

### Google Fonts Flow

1. **CSS request**: StaleWhileRevalidate (serve cache, update in background)
2. **Font file request**: CacheFirst (cache permanently)
3. **Two separate caches**: One for CSS, one for font files
4. **Automatic updates**: CSS updates trigger font file updates

## Debugging

### Inspect Self-Hosted Fonts Cache

1. Open DevTools (F12)
2. **Application** tab → **Cache Storage**
3. Find cache (default or custom `cache_name`)
4. View cached font files

### Inspect Google Fonts Caches

Look for two caches:
- `{cache_prefix}-stylesheets`: CSS files
- `{cache_prefix}-webfonts`: Font files (.woff2, etc.)

## Best Practices

1. **Prefer WOFF2**: Modern, compressed format for smaller files
2. **Limit font weights**: Cache only the weights you actually use
3. **Use appropriate max_age**: Fonts rarely change, use long durations
4. **Custom cache names**: Makes debugging easier
5. **Monitor cache size**: Especially with international fonts
6. **Subset fonts**: Use font subsetting to reduce file sizes
7. **Consider font-display**: Use `font-display: swap` in CSS for better UX

## Font Loading Strategy

Combine PWA caching with CSS font-display for optimal performance:

```css
@font-face {
    font-family: 'CustomFont';
    src: url('/fonts/custom-font.woff2') format('woff2');
    font-display: swap;  /* Show fallback immediately */
}
```

This ensures:
- Fallback font shows instantly
- Custom font loads from cache (fast)
- No layout shift when font loads

{% hint style="success" %}
**Recommended setup**: Use WOFF2 self-hosted fonts with `font-display: swap` and cache them for 1 year. This provides the best performance and user experience.
{% endhint %}

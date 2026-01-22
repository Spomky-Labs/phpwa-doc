# Resource Hints

Resource Hints allow browsers to perform optimizations like DNS lookups, TCP connections, and resource fetching before they're actually needed. This bundle automatically generates and injects resource hints via `{{ pwa() }}`.

## Benefits

- **Faster page loads**: Preconnect to origins before requests are made
- **Reduced latency**: DNS prefetch eliminates DNS lookup time
- **Critical resource prioritization**: Preload important resources early
- **Automatic detection**: Auto-detects external origins from your PWA configuration

## Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
  resource_hints:
    enabled: true
    auto_preconnect: true  # Auto-detect origins from Workbox config
    preconnect:
      - 'https://api.example.com'
      - 'https://cdn.example.com'
    dns_prefetch:
      - 'https://analytics.example.com'
    preload:
      - href: '/fonts/custom.woff2'
        as: font
        type: 'font/woff2'
      - href: '/css/critical.css'
        as: style
        fetchpriority: high
      - href: '/images/hero.webp'
        as: image
        media: '(min-width: 768px)'
```
{% endcode %}

## Usage

Resource hints are automatically injected when using `{{ pwa() }}`:

```twig
<!DOCTYPE html>
<html>
<head>
    {{ pwa() }}
    {# Resource hints are automatically included #}
</head>
</html>
```

To disable resource hints:

```twig
{{ pwa(injectResourceHints: false) }}
```

## Hint Types

### Preconnect

Preconnect hints tell the browser to establish early connections to important origins. This includes DNS lookup, TCP handshake, and TLS negotiation.

```yaml
pwa:
  resource_hints:
    preconnect:
      - 'https://api.example.com'
      - 'https://fonts.googleapis.com'
```

Generated HTML:
```html
<link rel="preconnect" href="https://api.example.com" crossorigin>
```

### DNS Prefetch

DNS prefetch performs only the DNS lookup, which is less aggressive than preconnect but still helpful for origins you'll connect to later.

```yaml
pwa:
  resource_hints:
    dns_prefetch:
      - 'https://analytics.example.com'
      - 'https://tracking.example.com'
```

Generated HTML:
```html
<link rel="dns-prefetch" href="https://analytics.example.com">
```

### Preload

Preload hints fetch critical resources early. Use this for resources needed for the current page.

```yaml
pwa:
  resource_hints:
    preload:
      - href: '/fonts/custom.woff2'
        as: font
        type: 'font/woff2'
        crossorigin: anonymous
      - href: '/css/critical.css'
        as: style
        fetchpriority: high
      - href: '/images/hero.webp'
        as: image
        media: '(min-width: 768px)'
```

#### Preload Attributes

| Attribute | Description | Required |
|-----------|-------------|----------|
| `href` | URL or path to preload | Yes |
| `as` | Resource type: `script`, `style`, `font`, `image`, `fetch` | Yes |
| `type` | MIME type of the resource | No |
| `crossorigin` | CORS setting: `anonymous` or `use-credentials` | No (auto for fonts) |
| `fetchpriority` | Priority hint: `high`, `low`, or `auto` | No |
| `media` | Media query for responsive preloading | No |

{% hint style="info" %}
Fonts always require `crossorigin` attribute. If not specified, the bundle automatically adds `crossorigin="anonymous"` for font preloads.
{% endhint %}

#### Asset Mapper Support

The `href` attribute supports Asset Mapper logical paths. Paths without a leading `/` or `http(s)://` are automatically resolved to versioned URLs:

```yaml
pwa:
  resource_hints:
    preload:
      # Asset Mapper path - resolved to versioned URL
      - href: 'fonts/inter-var.woff2'
        as: font
        type: 'font/woff2'
      # Absolute path - kept as-is
      - href: '/fonts/custom.woff2'
        as: font
      # External URL - kept as-is
      - href: 'https://cdn.example.com/font.woff2'
        as: font
```

| Input | Output |
|-------|--------|
| `fonts/inter.woff2` | `/assets/fonts/inter-abc123.woff2` (versioned) |
| `/fonts/custom.woff2` | `/fonts/custom.woff2` (unchanged) |
| `https://cdn.example.com/font.woff2` | `https://cdn.example.com/font.woff2` (unchanged) |

{% hint style="tip" %}
Using Asset Mapper paths ensures your preload hints always point to the correct versioned assets, even after deployments.
{% endhint %}

## Auto-Detection

When `auto_preconnect` is enabled (default), the bundle automatically detects external origins from your PWA configuration:

### Workbox CDN

If you're using Workbox from CDN (`use_cdn: true`), preconnect hints are added for:
- `https://storage.googleapis.com`
- `https://cdn.jsdelivr.net`

### Google Fonts

If Google Fonts caching is enabled in Workbox, preconnect hints are added for:
- `https://fonts.googleapis.com`
- `https://fonts.gstatic.com`

## HTTP Link Headers

In addition to HTML link tags, the bundle automatically adds resource hints as HTTP `Link` headers on the response. This enables:

- **HTTP/2 Server Push** (if supported by your server)
- **HTTP/103 Early Hints** (when combined with the Early Hints feature)

The headers are added via Symfony's WebLink component for PSR-13 compliance.

## Best Practices

1. **Don't overuse preconnect**: Limit to 2-4 critical origins. Too many hints can delay more important resources.

2. **Use dns-prefetch for secondary origins**: For origins you'll connect to later in the page lifecycle.

3. **Preload critical fonts**: Fonts often block rendering, so preloading them can significantly improve perceived performance.

4. **Use fetchpriority wisely**: Set `high` only for above-the-fold critical resources.

5. **Consider media queries**: Use the `media` attribute to preload responsive images only when needed.

## Example: Complete Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
  resource_hints:
    enabled: true
    auto_preconnect: true
    preconnect:
      - 'https://api.myapp.com'
    dns_prefetch:
      - 'https://analytics.google.com'
    preload:
      # Critical font (Asset Mapper path - auto-versioned)
      - href: 'fonts/inter-var.woff2'
        as: font
        type: 'font/woff2'
      # Critical CSS (Asset Mapper path)
      - href: 'styles/critical.css'
        as: style
        fetchpriority: high
      # Hero image - desktop (Asset Mapper path)
      - href: 'images/hero-desktop.webp'
        as: image
        media: '(min-width: 1024px)'
      # Hero image - mobile (Asset Mapper path)
      - href: 'images/hero-mobile.webp'
        as: image
        media: '(max-width: 1023px)'
```
{% endcode %}

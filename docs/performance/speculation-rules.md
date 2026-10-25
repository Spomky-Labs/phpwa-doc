# Speculation Rules

Speculation Rules is a browser API that allows you to hint which pages the user is likely to navigate to next. The browser can then prefetch or prerender those pages in advance, making navigation feel instant.

## Benefits

- **Instant navigation**: Prerendered pages load instantly when clicked
- **Improved perceived performance**: Users experience near-zero navigation time
- **Smart prefetching**: Only fetch resources, not execute, for lighter optimization
- **Flexible rules**: Target specific links by URL patterns or CSS selectors

## Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
  speculation_rules:
    enabled: true
    prefetch:
      - source: document
        eagerness: moderate
        href_matches: '/products/*'
    prerender:
      - source: document
        eagerness: conservative
        selector_matches: '.prerender-link'
```
{% endcode %}

## Usage

Speculation rules are automatically injected when using `{{ pwa() }}`:

```twig
<!DOCTYPE html>
<html>
<head>
    {{ pwa() }}
    {# Speculation rules script is automatically included #}
</head>
</html>
```

To disable speculation rules injection:

```twig
{{ pwa(injectSpeculationRules: false) }}
```

## Rule Types

### Prefetch

Prefetch downloads resources in advance but doesn't execute them. This is lighter than prerendering and suitable for pages the user might navigate to.

```yaml
pwa:
  speculation_rules:
    enabled: true
    prefetch:
      - source: document
        href_matches: '/blog/*'
        eagerness: moderate
```

### Prerender

Prerender fully loads and renders the page in a hidden tab. When the user navigates, the page appears instantly. Use this for high-confidence navigation predictions.

```yaml
pwa:
  speculation_rules:
    enabled: true
    prerender:
      - source: document
        selector_matches: 'a.primary-cta'
        eagerness: moderate
```

{% hint style="warning" %}
Prerendering consumes more resources than prefetching. Use it sparingly for pages with high navigation probability.
{% endhint %}

## Source Types

### Document Source

Match links in the current document using CSS selectors or URL patterns:

```yaml
pwa:
  speculation_rules:
    prefetch:
      # Match by URL pattern
      - source: document
        href_matches: '/products/*'

      # Match by CSS selector
      - source: document
        selector_matches: 'a.prefetch-link'

      # Combine both (AND condition)
      - source: document
        href_matches: '/checkout/*'
        selector_matches: '.checkout-link'
```

### List Source

Specify explicit URLs to prefetch or prerender:

```yaml
pwa:
  speculation_rules:
    prefetch:
      - source: list
        urls:
          - '/popular-page'
          - '/frequently-visited'
```

## Eagerness Levels

The `eagerness` property controls when speculation happens:

| Level | Description |
|-------|-------------|
| `immediate` | Speculate as soon as the rules are observed |
| `eager` | Speculate on any slight indication of navigation |
| `moderate` | Speculate on hover (200ms) or pointer down |
| `conservative` | Speculate only on pointer/touch down |

```yaml
pwa:
  speculation_rules:
    prerender:
      # High confidence - prerender immediately
      - source: document
        selector_matches: '.hero-cta'
        eagerness: immediate

      # Medium confidence - wait for hover
      - source: document
        href_matches: '/products/*'
        eagerness: moderate

      # Low confidence - wait for click intent
      - source: document
        href_matches: '/external/*'
        eagerness: conservative
```

{% hint style="tip" %}
Start with `moderate` or `conservative` eagerness and increase based on analytics data showing actual navigation patterns.
{% endhint %}

## Advanced Options

### Referrer Policy

Control the referrer sent with speculative requests:

```yaml
pwa:
  speculation_rules:
    prefetch:
      - source: document
        href_matches: '/external/*'
        referrer_policy: 'no-referrer'
```

## Generated Output

The bundle generates a `<script type="speculationrules">` tag:

```html
<script type="speculationrules">
{
  "prefetch": [
    {
      "source": "document",
      "where": {"href_matches": "/products/*"},
      "eagerness": "moderate"
    }
  ],
  "prerender": [
    {
      "source": "document",
      "where": {"selector_matches": ".prerender-link"},
      "eagerness": "conservative"
    }
  ]
}
</script>
```

## Browser Support

Speculation Rules are supported in:
- Chrome 109+ (prefetch)
- Chrome 117+ (prerender)
- Edge 109+

For unsupported browsers, the script tag is simply ignored.

{% hint style="info" %}
Check [caniuse.com/speculation-rules](https://caniuse.com/speculation-rules) for current browser support.
{% endhint %}

## Best Practices

1. **Start conservative**: Begin with `conservative` or `moderate` eagerness and monitor performance impact.

2. **Use prefetch for uncertain navigations**: If navigation probability is below 80%, prefer prefetch over prerender.

3. **Prerender high-value pages**: Use prerender for pages where instant navigation provides significant UX benefit (checkout, product details).

4. **Avoid prerendering heavy pages**: Don't prerender pages with complex JavaScript, auto-playing media, or expensive API calls.

5. **Monitor resource usage**: Speculation consumes bandwidth and memory. Use browser DevTools to monitor impact.

6. **Combine with analytics**: Use navigation analytics to identify pages that benefit most from speculation.

## Example: E-commerce Site

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
  speculation_rules:
    enabled: true
    prefetch:
      # Prefetch product pages on hover
      - source: document
        href_matches: '/products/*'
        eagerness: moderate

      # Prefetch category pages
      - source: document
        selector_matches: '.category-link'
        eagerness: moderate

    prerender:
      # Prerender checkout - high value page
      - source: document
        selector_matches: '.checkout-button'
        eagerness: conservative

      # Prerender "Add to Cart" result page
      - source: document
        href_matches: '/cart'
        selector_matches: '.add-to-cart'
        eagerness: conservative
```
{% endcode %}

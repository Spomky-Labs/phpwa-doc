# Prefetch on Demand

The Prefetch on Demand component enables intelligent, user-driven page prefetching to dramatically improve navigation performance in your Progressive Web App. Instead of relying solely on static `<link rel="prefetch">` tags, this component allows you to dynamically prefetch pages based on user interactions like hovering over links, scrolling into view, or any custom trigger. Pages are cached by the service worker via Workbox, making subsequent navigation instantaneous.

This component is particularly useful for:

* Speeding up navigation in content-heavy websites (blogs, news sites, documentation)
* Preloading likely next pages in multi-step forms or checkout flows
* Improving perceived performance in single-page applications
* Caching related articles, author pages, or recommended content
* Prefetching resources when users hover over navigation menus
* Loading content as users scroll down infinite-scroll pages
* Preparing next chapters in e-learning platforms
* Optimizing user experience in catalog browsing and e-commerce

## Browser Support

The Prefetch on Demand component relies on service workers and the Cache API, which are widely supported across modern browsers.

**Support level:** Excellent - Works on all modern browsers that support service workers (Chrome, Firefox, Safari, Edge).

{% hint style="info" %}
This component requires an active service worker with Workbox configured. Ensure your PWA Bundle service worker is properly registered before using this component.
{% endhint %}

{% hint style="success" %}
Unlike native `<link rel="prefetch">`, which may be ignored by browsers under certain conditions (like when on cellular data), service worker-based prefetching gives you more control over caching behavior.
{% endhint %}

## Traditional Prefetching vs. On-Demand

### Static Prefetching

Traditional HTML-based prefetching uses link tags:

{% code lineNumbers="true" %}
```html
<link rel="prefetch" href="/article-2">
<link rel="prefetch" href="/article-3">
<link rel="prefetch" href="/author-18">
```
{% endcode %}

**Limitations:**
- Prefetching starts immediately on page load
- No control over timing or conditions
- Browser may ignore hints based on connection type or battery
- Cannot respond to user behavior

### Dynamic On-Demand Prefetching

The Prefetch on Demand component offers more flexibility:

- Trigger prefetching based on user interactions (hover, scroll, click)
- Programmatically control when and what to prefetch
- Cache resources through service worker for reliable offline access
- Respond to user behavior and intent

## Usage

### Basic Hover-Based Prefetching

{% code lineNumbers="true" %}
```twig
<section {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <main>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit.
        Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
    </main>
    <aside {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseover', {urls: ['/author-18', '/article-2', '/article-3']}) }}>
        Author: <a href="/author-18">John Doe</a>
        Other articles: <a href="/article-2">How to Foo</a>
        Other articles: <a href="/article-3">How to Bar</a>
    </aside>
</section>
```
{% endcode %}

When users hover over the aside section, the specified URLs are prefetched and cached.

### Individual Link Prefetching

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <h2>Related Articles</h2>
    <ul>
        <li>
            <a href="/articles/getting-started"
               {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                   urls: ['/articles/getting-started']
               }) }}>
                Getting Started Guide
            </a>
        </li>
        <li>
            <a href="/articles/advanced-topics"
               {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                   urls: ['/articles/advanced-topics']
               }) }}>
                Advanced Topics
            </a>
        </li>
    </ul>
</div>
```
{% endcode %}

### Intersection Observer - Prefetch on Scroll

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    {% for article in articles %}
        <article class="article-card"
                 data-prefetch-url="/articles/{{ article.slug }}"
                 data-controller="intersection-observer"
                 data-intersection-observer-root-margin-value="200px"
                 data-action="intersection-observer:appear->pwa--prefetch-on-demand#prefetchFromElement">
            <h3>{{ article.title }}</h3>
            <p>{{ article.excerpt }}</p>
        </article>
    {% endfor %}
</div>

<script type="module">
  // Add custom action to trigger from data attribute
  const controller = document.querySelector('[data-controller~="pwa__prefetch-on-demand"]').controller;
  controller.prefetchFromElement = function(event) {
    const url = event.currentTarget.dataset.prefetchUrl;
    if (url) {
      this.prefetch({ params: { urls: [url] } });
    }
  };
</script>
```
{% endcode %}

### Programmatic Prefetching

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <button id="load-next-chapter">Next Chapter</button>
    <div id="chapter-content"></div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__prefetch-on-demand"]');
  let currentChapter = 1;

  document.getElementById('load-next-chapter').addEventListener('click', async () => {
    // Prefetch next two chapters when user loads one
    const nextChapter = currentChapter + 1;
    const chapterAfterNext = currentChapter + 2;

    await host.controller.prefetch({
      params: {
        urls: [
          `/chapters/${nextChapter}`,
          `/chapters/${chapterAfterNext}`
        ]
      }
    });

    // Load current chapter
    loadChapter(currentChapter++);
  });

  // Listen for prefetch confirmation
  host.addEventListener('prefetch-on-demand:prefetched', (e) => {
    console.log('Prefetched URLs:', e.detail.params.urls);
  });

  host.addEventListener('prefetch-on-demand:error', (e) => {
    console.error('Prefetch failed:', e.detail);
  });
</script>
```
{% endcode %}

### Menu Navigation Prefetching

{% code lineNumbers="true" %}
```twig
<nav {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <ul class="main-menu">
        <li {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
            urls: ['/products', '/products/featured', '/products/new-arrivals']
        }) }}>
            <a href="/products">Products</a>
            <ul class="submenu hidden">
                <li><a href="/products/featured">Featured</a></li>
                <li><a href="/products/new-arrivals">New Arrivals</a></li>
            </ul>
        </li>
        <li {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
            urls: ['/about', '/about/team', '/about/contact']
        }) }}>
            <a href="/about">About</a>
            <ul class="submenu hidden">
                <li><a href="/about/team">Our Team</a></li>
                <li><a href="/about/contact">Contact Us</a></li>
            </ul>
        </li>
    </ul>
</nav>
```
{% endcode %}

## Best Practices

### Be Selective with Prefetching

{% hint style="warning" %}
Don't prefetch everything! Prefetching consumes bandwidth and cache storage. Only prefetch pages users are likely to visit based on:
- User behavior patterns
- Navigation flow analysis
- Content relationships
- User intent signals (hovering, scrolling direction)
{% endhint %}

{% code lineNumbers="true" %}
```twig
{# Good: Prefetch likely next pages #}
<div {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
    urls: ['/checkout/shipping']
}) }}>
    Continue to Shipping →
</div>

{# Bad: Prefetching unrelated pages wastes resources #}
<div {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
    urls: ['/about', '/contact', '/terms', '/privacy', '/faq', '/careers']
}) }}>
    Continue
</div>
```
{% endcode %}

### Use Hover Intent, Not Immediate Hover

{% hint style="success" %}
Consider using `mouseenter` instead of `mouseover`, or add a small delay before prefetching to avoid unnecessary prefetches when users move their mouse across the page.
{% endhint %}

{% code lineNumbers="true" %}
```javascript
// Add intelligent hover detection
let hoverTimeout;
element.addEventListener('mouseenter', () => {
  hoverTimeout = setTimeout(() => {
    // User has been hovering for 200ms - likely interested
    host.controller.prefetch({ params: { urls: ['/target-page'] } });
  }, 200);
});

element.addEventListener('mouseleave', () => {
  clearTimeout(hoverTimeout);
});
```
{% endcode %}

### Prioritize High-Value Pages

{% hint style="info" %}
Prefetch pages that provide the most value when loaded instantly:
- Next steps in user workflows
- High-traffic destination pages
- Content directly related to current page
- Pages that require authentication or API calls
{% endhint %}

### Monitor Cache Size

{% hint style="warning" %}
Service worker caches have storage limits. Implement cache management strategies to prevent filling up user storage:
- Set cache expiration policies in Workbox
- Limit the number of cached entries
- Clear old prefetched pages periodically
{% endhint %}

### Respect User Preferences

{% code lineNumbers="true" %}
```javascript
// Check for data saver mode before prefetching
const connection = navigator.connection;
if (connection && connection.saveData) {
  console.log('Data saver enabled - skipping prefetch');
} else {
  // Safe to prefetch
  host.controller.prefetch({ params: { urls: ['/next-page'] } });
}
```
{% endcode %}

## Common Use Cases

### 1. Multi-Step Form/Checkout Flow

{% code lineNumbers="true" %}
```twig
{# Step 1: Cart #}
<div {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <h2>Your Cart</h2>
    <div class="cart-items">
        {# Cart contents #}
    </div>

    <button {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
        urls: ['/checkout/shipping', '/checkout/payment']
    }) }}>
        Proceed to Checkout
    </button>
</div>
```
{% endcode %}

### 2. Blog Article Navigation

{% code lineNumbers="true" %}
```twig
<article {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <h1>{{ article.title }}</h1>
    <div class="article-content">
        {{ article.content }}
    </div>

    <footer>
        <div class="author-box"
             {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                 urls: ['/authors/' ~ article.author.slug]
             }) }}>
            <img src="{{ article.author.avatar }}" alt="{{ article.author.name }}">
            <span>By {{ article.author.name }}</span>
        </div>

        <div class="related-articles">
            <h3>You Might Also Like</h3>
            {% for related in article.relatedArticles %}
                <a href="{{ path('article_show', {slug: related.slug}) }}"
                   {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                       urls: [path('article_show', {slug: related.slug})]
                   }) }}>
                    {{ related.title }}
                </a>
            {% endfor %}
        </div>
    </footer>
</article>
```
{% endcode %}

### 3. E-Commerce Product Listing

{% code lineNumbers="true" %}
```twig
<div class="product-grid" {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    {% for product in products %}
        <div class="product-card"
             {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                 urls: ['/products/' ~ product.slug, '/api/products/' ~ product.id]
             }) }}>
            <img src="{{ product.image }}" alt="{{ product.name }}">
            <h3>{{ product.name }}</h3>
            <p class="price">{{ product.price|currency }}</p>
            <a href="/products/{{ product.slug }}">View Details</a>
        </div>
    {% endfor %}
</div>
```
{% endcode %}

### 4. Documentation Navigation

{% code lineNumbers="true" %}
```twig
<div class="docs-layout" {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <aside class="sidebar">
        <nav>
            {% for section in docSections %}
                <div class="section">
                    <h4>{{ section.title }}</h4>
                    <ul>
                        {% for page in section.pages %}
                            <li>
                                <a href="{{ path('docs_page', {slug: page.slug}) }}"
                                   {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                                       urls: [path('docs_page', {slug: page.slug})]
                                   }) }}>
                                    {{ page.title }}
                                </a>
                            </li>
                        {% endfor %}
                    </ul>
                </div>
            {% endfor %}
        </nav>
    </aside>

    <main class="docs-content">
        {{ currentPageContent }}

        {# Prefetch previous/next pages #}
        <div class="page-navigation">
            {% if previousPage %}
                <a href="{{ path('docs_page', {slug: previousPage.slug}) }}"
                   {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                       urls: [path('docs_page', {slug: previousPage.slug})]
                   }) }}>
                    ← {{ previousPage.title }}
                </a>
            {% endif %}

            {% if nextPage %}
                <a href="{{ path('docs_page', {slug: nextPage.slug}) }}"
                   {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                       urls: [path('docs_page', {slug: nextPage.slug})]
                   }) }}>
                    {{ nextPage.title }} →
                </a>
            {% endif %}
        </div>
    </main>
</div>
```
{% endcode %}

### 5. Infinite Scroll Feed

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/prefetch-on-demand') }} id="feed-container">
    {% for post in posts %}
        <article class="post">
            <h2>{{ post.title }}</h2>
            <p>{{ post.excerpt }}</p>
            <a href="/posts/{{ post.slug }}"
               {{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
                   urls: ['/posts/' ~ post.slug]
               }) }}>
                Read More
            </a>
        </article>
    {% endfor %}

    <div id="load-more-trigger"></div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__prefetch-on-demand"]');
  const loadMoreTrigger = document.getElementById('load-more-trigger');
  let currentPage = 1;

  // Observe when user scrolls near bottom
  const observer = new IntersectionObserver((entries) => {
    if (entries[0].isIntersecting) {
      loadNextPage();
    }
  }, { rootMargin: '400px' });

  observer.observe(loadMoreTrigger);

  async function loadNextPage() {
    currentPage++;

    // Prefetch next page of posts
    await host.controller.prefetch({
      params: {
        urls: [`/api/posts?page=${currentPage}`]
      }
    });

    // Also prefetch individual post pages
    const response = await fetch(`/api/posts?page=${currentPage}`);
    const posts = await response.json();

    const postUrls = posts.map(post => `/posts/${post.slug}`);
    await host.controller.prefetch({
      params: { urls: postUrls }
    });

    // Render posts...
  }
</script>
```
{% endcode %}

## API Reference

### Values

None

### Actions

#### `prefetch`

Prefetches and caches the specified URLs through the service worker.

**Parameters:**
- `urls` (array, required): Array of URL strings to prefetch and cache

{% code lineNumbers="true" %}
```twig
{{ stimulus_action('@pwa/prefetch-on-demand', 'prefetch', 'mouseenter', {
    urls: ['/page-1', '/page-2', '/api/data']
}) }}
```
{% endcode %}

**Example with custom event:**

{% code lineNumbers="true" %}
```javascript
host.controller.prefetch({
  params: {
    urls: ['/page-1', '/page-2']
  }
});
```
{% endcode %}

### Targets

None

### Events

#### `prefetched`

Fired when the service worker successfully receives and processes the prefetch request. Note that this indicates the command was received, not necessarily that all URLs are fully cached.

**Event detail:**
- `params.urls` (array): The URLs that were requested to be prefetched

{% code lineNumbers="true" %}
```javascript
host.addEventListener('prefetch-on-demand:prefetched', (e) => {
  console.log('Prefetch command processed:', e.detail.params.urls);
});
```
{% endcode %}

#### `error`

Fired when the prefetch operation fails (e.g., service worker not available, Workbox not configured).

**Event detail:**
- `params.urls` (array): The URLs that failed to prefetch

{% code lineNumbers="true" %}
```javascript
host.addEventListener('prefetch-on-demand:error', (e) => {
  console.error('Prefetch failed for:', e.detail.params.urls);
  // Optionally fall back to native prefetch
  e.detail.params.urls.forEach(url => {
    const link = document.createElement('link');
    link.rel = 'prefetch';
    link.href = url;
    document.head.appendChild(link);
  });
});
```
{% endcode %}

## How It Works

1. **User Interaction**: User triggers an event (hover, scroll, click)
2. **Controller Action**: The `prefetch` action is called with an array of URLs
3. **Service Worker Message**: Controller sends a `CACHE_URLS` message to the service worker via Workbox
4. **Caching**: Service worker fetches and caches the specified URLs
5. **Event Dispatch**: Component emits `prefetched` or `error` event
6. **Navigation**: When user navigates to prefetched URL, it loads instantly from cache

## Configuration

Ensure your service worker is configured to handle the `CACHE_URLS` message. The PWA Bundle automatically configures this when Workbox is enabled.

If you need custom cache configuration:

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        workbox:
            enabled: true
            # Configure resource caching for prefetched pages
            resource_caching:
                -
                    match: '/.*'
                    strategy: 'NetworkFirst'
                    cache_name: 'prefetched-pages'
                    options:
                        networkTimeoutSeconds: 3
                        expiration:
                            maxEntries: 50
                            maxAgeSeconds: 86400  # 1 day
```
{% endcode %}

## Related Components

- [Service Worker](service-worker.md) - Required for prefetching to work
- [Background Fetch](background-fetch.md) - For downloading large files in background
- [Connection Status](connection-status.md) - Check connection before prefetching
- [Network Information](network-information.md) - Adapt prefetching based on connection type

## Resources

- [MDN: Link types: prefetch](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/prefetch)
- [Web.dev: Resource Hints](https://web.dev/learn/performance/resource-hints)
- [Workbox: Cache URLs on demand](https://developer.chrome.com/docs/workbox/managing-fallback-responses/)
- [PWA Bundle Service Worker Documentation](../the-service-worker/workbox/)

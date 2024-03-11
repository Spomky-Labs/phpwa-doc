# Prefetch on demand

Modern browsers are able to prefetch pages. Let say an article is displayed to the user. This article has related articles or pages the user may read. These pages can be prefetched so that when the user will click on the related link, the page will be available instantly.

To acheive that, you can add the follwing HTML tag:

```html
<link rel="prefetch" href="/article-2">
<link rel="prefetch" href="/article-3">
<link rel="prefetch" href="/author-18">
```

Another approch could be the use of the provided Stimulus Component to prefetch on demand and depending on the user navigation on the page. For example, no link tags are set and mouseover or any other user action will trigger the prefetch of the related pages.

**Usage**

```twig
<section {{ stimulus_controller('@pwa/prefetch-on-demand') }}>
    <main>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit.
        Sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
        Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat.
        Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur.
        Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
    </main>
    <aside {{ stimulus_action('@pwa/prefetch-on-demand', 'mouseover', 'prefetch', {urls: ['/author-18', '/article-2', '/article-3']}) }}>
        Author: <a href="/author-18">John Doe</a>
        Other articles: <a href="/article-2">How to Foo</a>
        Other articles: <a href="/article-3">How to Bar</a>
    </aside>
</section>
```

### Parameters

None

### Actions

* prefetch: asks the Service Worker to prefetch a list of URLs set in the `urls` parameter.

### Targets

None

### Events

None

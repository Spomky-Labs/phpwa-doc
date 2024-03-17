# Image Caching

By default, a maximum of 60 images are cached for 1 year. The supported image extensions extensions are as follows:&#x20;

```regex
/\.(ico|png|jpe?g|gif|svg|webp|bmp)$/
```

This can be changed using the next configuration options

<pre class="language-yaml" data-title="/config/packages/pwa.yaml" data-line-numbers><code class="lang-yaml">pwa:
<strong>    serviceworker:
</strong>        enabled: true
        src: "sw.js"
        workbox:
            image_cache:
                enabled: true
                max_age: 30 days
                max_entries: 200
                regex: '/\.(png|jpe?g|svg|webp)$/'
</code></pre>

{% hint style="info" %}
This cache is different from the Asset one as it corresponds to images that are not managed by Asset Mapper
{% endhint %}

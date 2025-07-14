# Deployment

As this bundle leverages Asset Mapper, the resources are served using this great component.

In the `dev` environment, the resources are automactilly handled and returned by your Symfony app.

For the `prod` environment, before deploy, you should run:

<pre class="language-shell"><code class="lang-shell"><strong>symfony console asset-map:compile
</strong></code></pre>

If, for any reason, you want to manually trigger the asset compilation, please use the following command:

<pre><code><strong>symfony console pwa:compile
</strong></code></pre>

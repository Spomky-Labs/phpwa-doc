# Deployment

## With Asset Mapper

In the `dev` environment, the resources are automactilly handled and returned by a dedicated request listener.

For the `prod` environment all the assets shall be compiled. This means the favicons, the manifests, the service worker and all dependencoes (IndexDB, Workbox...) will be stored as pure assets.

Before deploy, you should run:

```sh
symfony console asset-map:compile
```

## Without Asset Mapper

If you do not want to compile PWA assets with asset mapper, set `false` to the configuration option `pwa.asset_compiler` .

{% code title="config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    asset_compiler: false
```
{% endcode %}

{% hint style="info" %}
Note that `true` is the default value in `1.x`, but will be set to `false` in `2.x`.
{% endhint %}

Then execute the following command to compile ALL assets exactly in the same manner as with Asset Mapper.

```sh
symfony console pwa:compile
```

If you have defined paths or URIs that depend on the application context, especially those relying on the `default_uri` parameter, make sure the context is set **before** running the command.\
Otherwise, generated URLs may default to something like `http://localhost/...` instead of the expected `https://acme.com/...`.

If you cannot set the context beforehand, you can proceed in two steps:

1. Run `symfony console pwa:compile` before deployment.
2. Deploy your application, then during the startup process, run the command again with the `--context-only` option.

This second execution will recompile only the context-dependent files (currently, the **manifests** and the **service worker**).\
The advantage is that the longer compilation tasks, such as icon generation, do not need to be repeated.

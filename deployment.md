# Deployment

As this bundle leverages Assert Mapper, the resources are served using this great component.

In the `dev` environment, the resources are automactilly handled and returned by your Symfony app.

For the `prod` environment, before deploy, you should run:

```shell
php bin/console asset-map:compile
```

# From 1.2.x to 1.3.0

## Backward Compatibility Breaks

Minor updates should not include backward compatibility breaks, except in specific circumstances.

### Start URL

Before 1.3.0, the Web Manifest start URL parameter was a translatable string. If you used it as an untranslated string, there is no change (e.g. `/en?pwa=1`).

However, if you set a translation key instead (e.g. `app.start_url`) the key will not be translated anymore. Please use a translatable route instead, e.g. `app_homepage` with parameters, that will be translated as any other route names.

```yaml
#Before
pwa:
    manifest:
        start_url: app.start_url

#After
pwa:
    manifest:
        start_url:
            path: 'app_homepage'
            params:
                pwa: 1
```

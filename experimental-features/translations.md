# Translations

The bundle can leverage the Symfony Translation component. The text you provide for almost all names, abbreviations, descriptions, labels, etc., can be used as translation keys.

These translations must be placed in a `pwa` domain (e.g. `pwa.en.yaml` or `pwa.fr.yaml`)

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "app.name"
        short_name: "app.short_name"
        start_url: "/index.html"
        display: "standalone"
        background_color: "#ffffff"
        theme_color: "#4285f4"
        shortcuts:
            - name: "app.feature1.shorcut.name"
              short_name: "app.feature1.shorcut.short_name"
              description: "app.feature1.shorcut.description"
              url: "/start-chat"
              icons:
                - src: "icons/feature1-96x96.png"
                  sizes: [96]
```
{% endcode %}

{% code title="/translations/pwa.en.yaml" lineNumbers="true" %}
```yaml
app:
    name: "My App"
    short_name: "My App"
    feature1:
        shortcut:
            name: "Start chat"
            short_name: "Start chat"
            description: "Start a chat with your friends"
```
{% endcode %}

{% hint style="warning" %}
This feature is in early development stage and may not work as expected. Use with caution
{% endhint %}

## Localization Strategies

The locales the bundle translates the manifest into are the ones you enabled in the framework configuration.

{% code title="/config/packages/framework.yaml" lineNumbers="true" %}
```yaml
framework:
    enabled_locales: ['en', 'fr', 'de', 'ar']
```
{% endcode %}

There are two ways to deliver these translations to the browsers, driven by the `localization_strategy` option.

| Strategy | Compiled files | `*_localized` members |
| --- | --- | --- |
| `files` (default) | one per enabled locale | no |
| `inline` | a single, locale agnostic one | yes |
| `both` | one per enabled locale | yes |

### `files`: one manifest per locale

This is the historical behaviour, and the default. The bundle compiles one manifest file per enabled locale, and the `{locale}` placeholder of the public URL is replaced by the locale of each of them.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: '/site.{locale}.webmanifest'
        name: "app.name"
```
{% endcode %}

```
public/site.en.webmanifest
public/site.fr.webmanifest
public/site.de.webmanifest
public/site.ar.webmanifest
```

Your template then has to render the URL matching the current request:

```twig
{{ pwa(locale: app.request.locale) }}
```

This is the fragile part of the approach: the `<link rel="manifest">` tag must carry the right locale, and the manifest is usually cached by the service worker.

### `inline`: a single manifest carrying every translation

The Web App Manifest specification supports [localized members](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Manifest/Reference/*_localized): `name_localized`, `short_name_localized`, `description_localized` and `icons_localized` hold the translations of their plain counterpart, and the browser picks the variant matching the user language settings. They are supported at the top level of the manifest and inside each shortcut.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        localization_strategy: 'inline'
        name: "app.name"
        short_name: "app.short_name"
        description: "app.description"
```
{% endcode %}

A single, locale agnostic file is compiled, so there is no locale to pass to the Twig function anymore:

```twig
{{ pwa() }}
```

{% code title="/public/site.webmanifest" lineNumbers="true" %}
```json
{
    "name": "The SuperSausage app",
    "name_localized": {
        "fr": "L'application SuperSausage",
        "de": "Die SuperWurst-App",
        "ar": {
            "value": "تطبيق سوبر سوسيج",
            "dir": "rtl"
        }
    },
    "short_name": "SuperSausage",
    "short_name_localized": {
        "fr": "SuperSaucisse",
        "de": "SuperWurst"
    }
}
```
{% endcode %}

{% hint style="warning" %}
The `{locale}` placeholder cannot be used in `public_url` with this strategy, as a single file is compiled. The bundle rejects the combination at configuration time.
{% endhint %}

### `both`: belt and braces

Localized members are still experimental and are not supported by every browser. With `both`, the bundle compiles one manifest per locale **and** adds the localized members to each of them. Browsers that do not understand them simply ignore them, and fall back on the file they were served.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        localization_strategy: 'both'
        public_url: '/site.{locale}.webmanifest'
        name: "app.name"
```
{% endcode %}

## What Ends Up In The Localized Members

The textual members need no additional configuration: their values already are translation keys of the `pwa` domain, so the localized map is derived from the enabled locales.

A locale is left out of the map when its translation resolves to the value already carried by the plain member, or to the key itself because nothing translates it. An application that translates nothing therefore gets no `*_localized` member at all, and with the `both` strategy the locale of a file never repeats itself inside its own map.

### Writing Direction

A locale whose writing direction differs from the one of the manifest is declined with the object form and an explicit `dir`, as `ar` is in the example above.

* the direction of the manifest is its `dir` member, or `ltr` when it declares none;
* the direction of a locale is detected from its script subtag (`uz-Arab` is right to left while `uz-Latn` is not), then from its primary language.

Should the detection be wrong for one of your locales, override it. The most granular tag wins, so `ar-DZ` can differ from `ar`.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        localization_strategy: 'inline'
        locale_directions:
            ku: 'rtl'
            ar-DZ: 'ltr'
```
{% endcode %}

### Icons

Icons cannot be derived from a translation catalogue, so they have their own option. It is available on the manifest and on each shortcut, and accepts the same shorthands as `icons`.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        localization_strategy: 'inline'
        icons:
            - src: 'icons/logo.svg'
              sizes: [48, 96, 128, 256]
        icons_localized:
            de:
                - src: 'icons/logo.de.svg'
                  sizes: [48, 96, 128, 256]
            ar:
                - src: 'icons/logo.ar.svg'
                  sizes: [48, 96, 128, 256]
        shortcuts:
            - name: "app.feature1.shorcut.name"
              url: "/start-chat"
              icons:
                - src: "icons/feature1-96x96.png"
                  sizes: [96]
              icons_localized:
                de:
                  - src: "icons/feature1-96x96.de.png"
                    sizes: [96]
```
{% endcode %}

The images they point at are compiled like any other application icon.

{% hint style="info" %}
`icons_localized` is only written to the manifest with the `inline` and `both` strategies. With `files`, each locale already gets a file of its own.
{% endhint %}

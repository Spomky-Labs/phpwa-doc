# Translations

The bundle leverages on Symfony Translation component if available. The texts you pass for almost all names, short names, descriptions, labels... are translation keys.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        name: "app.name"
        short_name: "app.short_name"
        start_url: "/index.html"
        displa": "standalone"
        background_color: "#ffffff"
        theme_color: "#4285f4"
        shortcuts:
            - name: "app.feature1.shorcut.name"
              short_name: "app.feature1.shorcut.short_name"
              description: "app.feature1.shorcut.description"
              url: "/start-chat"
              icons":          
                - src: "icons/feature1-96x96.png"
                  sizes: [96]
```
{% endcode %}

{% hint style="warning" %}
This feature is still in development stage and may not work as expected. Use with caution.

BC is not guaranteed.
{% endhint %}

## Translatable values

The following values are translation keys. The domain is `pwa`.

| Component    | Values                                                                                             |
| ------------ | -------------------------------------------------------------------------------------------------- |
| Manifest     | <ul><li>name</li><li>short_name</li><li>description</li><li>categories</li><li>start_url</li></ul> |
| Screenshot   | <ul><li>label</li></ul>                                                                            |
| Share Target | <ul><li>title</li><li>text</li></ul>                                                               |
| Shortcut     | <ul><li>name</li><li>short_name</li><li>description</li></ul>                                      |
| Widget       | <ul><li>name</li><li>short_name</li><li>description</li></ul>                                      |

## How To?

This feature relies on the `framework.enabled_locales` to generate static manifest files. Please refer to the [Symfony documentation](https://symfony.com/doc/7.1/reference/configuration/framework.html#reference-translator-enabled-locales) for more information.

To enable it, the manifest public URL shall contain the placeholder `{locale}`.

**Example:**

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        public_url: "/site.{locale}.webmanifest"
```
{% endcode %}

When done, the Twig function should get the locale to serve the translated version of the manifest.

{% code lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html lang="en">
<head>
  {{ pwa(locale=app.request.locale) }}
</head>
<body>
  ...
</body>
</html>
```
{% endcode %}

Now you can have translation files for each locale you support:

{% code title="translations/pwa+intl-icu.en_US.xlf" lineNumbers="true" %}
```xml
<?xml version="1.0" encoding="utf-8"?>
<xliff xmlns="urn:oasis:names:tc:xliff:document:1.2" version="1.2">
  <file source-language="en-US" target-language="en-US" datatype="plaintext" original="file.ext">
    <header>
      <tool tool-id="symfony" tool-name="Symfony"/>
    </header>
    <body>
      <trans-unit id="f3U2Gqw" resname="app.name">
        <source>app.name</source>
        <target>TODO!</target>
      </trans-unit>
      <trans-unit id="q2FYgid" resname="app.short_name">
        <source>app.short_name</source>
        <target>TODO!</target>
      </trans-unit>
      ...
    </body>
  </file>
</xliff>
```
{% endcode %}

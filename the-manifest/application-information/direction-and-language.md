# Direction and Language

The PWA dir and lang parameters.

Progressive Web Apps (PWAs) can be tailored for different languages and writing directions through the use of the `dir` and `lang` parameters in the HTML tag.

By properly setting the `dir` and `lang` parameters, you ensure better user experience, accessibility, and potentially improved SEO for your PWA.

Please note that those values should be in accordance with the ones set as `<html>` tag attributes.

```html
<html dir="ltr" lang="en">
</html>
```

### The `dir` Parameter

The `dir` parameter specifies the text directionality of the content in your PWA. It can have one of three values:

* `ltr`: Left-to-right, which is used for languages that are read from the left to the right (like English).
* `rtl`: Right-to-left, for languages read from the right to the left (like Arabic or Hebrew).
* `auto`: Automatically determines the direction based on the content.

Example usage in HTML:

### The `lang` Parameter

The `lang` parameter declares the default language of the text in the page. This is important for accessibility tools and search engines.

### Configuration

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        dir: "ltr"
        lang: "en"
```
{% endcode %}

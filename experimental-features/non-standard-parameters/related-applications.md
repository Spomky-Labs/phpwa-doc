# Related Applications

The `prefer_related_applications` member is a boolean value that specifies that applications listed in `related_applications` should be preferred over the web application.

If the `prefer_related_applications` member is set to `true`, the user agent might suggest installing one of the related applications instead of this web application.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        prefer_related_applications: true
        related_applications:
            - platform: "play"
              url: "https://play.google.com/store/apps/details?id=com.example.app1"
              id: "com.example.app1"
            - platform: "itunes"
              url: "https://itunes.apple.com/app/example-app1/id123456789"
            - platform: "windows"
              url: "https://apps.microsoft.com/store/detail/example-app1/id123456789"
```
{% endcode %}

Currently known possible values for the platform member are as follows;

* `"chrome_web_store"`: [Google Chrome Web Store](https://chrome.google.com/webstore).
* `"play"`: [Google Play Store](https://play.google.com/).
* `"chromeos_play"`: [Chrome OS Play](https://support.google.com/googleplay/answer/7021273).
* `"webapp"`: [Web apps](https://www.w3.org/TR/appmanifest/).
* `"windows"`: [Windows Store](https://www.microsoft.com/store/apps).
* `"f-droid"`: [F-droid](https://f-droid.org/).
* `"amazon"`: [Amazon App Store](https://www.amazon.com/gp/browse.html?node=2350149011).

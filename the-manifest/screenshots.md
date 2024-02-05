# Screenshots

Progressive Web App screenshots enable richer installation UI. As shown below, by default only the application name and short name are visible.

<figure><img src="../.gitbook/assets/Capture d&#x27;écran 2024-01-31 193350.png" alt=""><figcaption><p>GitHub installation UI</p></figcaption></figure>

However, by including screenshots in the web app manifest, the installation experience can be significantly improved. These screenshots give users a visual preview of the app, enhancing their understanding and increasing the likelihood of installation.

Screenshots should showcase the most important functionalities of your application and be of high quality to attract users.

In the example below, a selection of screenshots are visible and the user can navigate to show them all. On mobile devices, the interface is different, but shows the same information.

<figure><img src="../.gitbook/assets/Capture d&#x27;écran 2024-02-01 101647.png" alt=""><figcaption></figcaption></figure>

## Configuration

You can add as many screenshots as you need. But keep in mind that the host device or the platform may  show only a selection of them.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        screenshots:
            - "images/screenshot-feature1.png"
            - 
              src: "images/screenshot-feature2.png"
              platform: "android"
              type: "inmage/png"
            -
              src: "images/screenshot-feature3.png"
              label: "Feature #3 in action"
              form_factor: "narrow"
```
{% endcode %}

### `src` Parameter

See the description [on the `icon` page](icons.md#src-parameter).

Ensure that the screenshots you provide are of high quality. Crisp and clear images can make a significant difference in how users perceive your app. Always aim for the highest resolution possible, without compromising the load times or performance of the installation interface.

Remember, these screenshots are part of your app's first impression on potential users. Take the time to choose them wisely, ensuring they accurately represent your app and its key features.

### `type` Parameter

See the description [on the `icon` page](icons.md#src-parameter).

### `label` Parameter

The label parameter provides a way to give a brief description or caption for the screenshot. This helps users to understand what the feature or screen is about before they have installed the app. It assists in providing context and can be particularly useful when displaying a series of screenshots.

Example:

```yaml
label: "Main dashboard view"
```

### `platform` Parameter

The `platform` parameter can be used to specify which operating system the screenshot is intended for. This helps to display the appropriate screenshots for users on different devices. For instance, you might have specific screenshots for Android users versus those using a desktop browser.

Example:

```yaml
platform: "android"
```

Possible values are lisrted below. This list is not exhastive.

* Device platform identifiers:
  * `"android"`
  * `"chromeos"`
  * `"ipados"`
  * `"ios"`
  * `"kaios"`
  * `"macos"`
  * `"windows"`
  * `"xbox"`
* Distribution platform identifiers:
  * `"chrome_web_store"`
  * `"itunes"`
  * `"microsoft-inbox"`
  * `"microsoft-store"`
  * `"play"`

### `form_factor` Parameter

The `form_factor` parameter allows you to define the intended device form factor for your screenshots. This can help cater to different device types, such as mobile, tablet, or desktop, ensuring that the screenshots displayed are relevant to the user's device.

Possible values include:

* `"narrow"`: Suggests the screenshot is best suited for narrow screen devices, like phones.
* `"wide"`: Implies the screenshot is intended for wider screen devices, such as tablets or desktops.

Example:

```yaml
form_factor: "narrow"
```

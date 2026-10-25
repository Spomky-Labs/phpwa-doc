# Share Target

The Share Target feature enhances the functionality of Progressive Web Apps (PWAs) by seamlessly integrating them into the native operating system's sharing capabilities. This means that when users come across content that they want to share from any app or browser, your PWA can appear in the list of sharing options alongside other native applications.

In the example below, the PWA indicates it can receive image files. The files are sent to the application using a `POST` request to `/share-target` and `multipart/form-data` encoding. The shared file is in the request body `file` member.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
            share_target:
              action: "/share-target"
              method: "POST"
              enctype: "multipart/form-data"
              params:
                  files:
                      - name: "file",
                        accept: ["image/*"]
```
{% endcode %}

{% hint style="info" %}
The `POST` request is ideally replied with an _HTTP 303 See Other redirect_ to avoid multiple `POST` requests from being submitted if a page refresh was initiated by the user, for example.
{% endhint %}

### `action` Parameter

The URL of the share target. See the [URL parameter](shortcuts.md#url-parameter) for the details.

### `method` Parameter

The HTTP request method to use. Either `GET` or `POST`.

Use `POST` if the shared data includes binary data like image(s), or if it changes the target app, for example, if it creates a data point like a bookmark.

### `enctype` Parameter

The encoding of the share data when a `POST` request is used. Ignored with `GET` requests.

### `params` Parameter

An object to configure the share parameters.

Object values can be specified and will be used as query parameters. Unless otherwise noted, members are optional.

* `title`: Name of the query parameter to use for the title of the document being shared.
* `text`: Name of the query parameter for the text (or body) of the message being shared.
* `url` : Name of the query parameter for the URL to the resource being shared.
* `files`: a list defining which files are accepted by the share target. Each item requires the following properties:
  * `name`: Name of the form field used to share files.
  * `accept` : a list of accepted MIME types or file extensions.


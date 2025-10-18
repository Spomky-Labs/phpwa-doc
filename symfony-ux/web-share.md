# Web Share

## Introduction to the Web Share API

The Web Share API is a modern web technology that allows web applications to leverage the native share functionality of the user's device. By utilizing this API, developers can enhance the user experience by providing an easy and seamless way for users to share content directly from their web applications.

### Parameters

None

### Actions

`share` : This method requires a parameter `data` that contains at least one element of:

* `title`: The title of the data to be shared
* `url`: an URL to be shared
* `text`: a text to be shared
* `files`: a list of file URLs to be shared

{% hint style="info" %}
The `title` may be ignored by the client.
{% endhint %}

{% hint style="warning" %}
The list contains URLs that the client will download before sharing. **Ensure the files are small and shareable**. Refer to the list at [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/share#shareable_file_types) for acceptable file types.
{% endhint %}

### Targets

None

### Events

`pwa--web-share:success`&#x20;

`pwa--web-share:error`&#x20;

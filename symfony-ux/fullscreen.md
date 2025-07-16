# Fullscreen

Use the Fullscreen component to switch the entire page or a specific element to fullscreen mode. In the example below, the buttons toggle the mode for the page or the image respectively.

{% code lineNumbers="true" %}
```twig
<section data-controller="pwa--fullscreen">
    <img id="image1" src="https://picsum.photos/400/600" alt="Sample Image" class="h-auto max-w-full">
    <button {{ stimulus_action('pwa--fullscreen', 'request', 'click') }}>
        The page
    </button>
    <button {{ stimulus_action('pwa--fullscreen', 'request', 'click', {target: '#image1'}) }}>
        The image
    </button>
</section>
```
{% endcode %}

### Parameters

None

### Actions

`request`: this action triggers fullscreen mode. The `target` parameter specifies the element to be displayed in fullscreen.

`exit`: this action will exist the fullscreen mode.

### Targets

None

### Events

Upon status change, the event `pwa--battery:updated` is triggered. The payload includes the following data:

* `change`: indicates the mode changed. The event properties `fullscreen` (bool) and `element` (target) are available.
* `error`: in case of error during the application of the change. The event property `element` is available.

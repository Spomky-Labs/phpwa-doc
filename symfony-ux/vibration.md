# Vibration

### Overview of the Vibration API

The Vibration API is a powerful tool that allows web developers to enhance user experience on mobile devices by enabling vibration functionality. This API provides a means for web applications to programmatically trigger vibrations, thereby making web interactions more tactile and engaging.

#### Considerations

* **Device Support**: Ensure that the target mobile device supports vibration functionality, as some devices may not have this feature.
* **Battery Usage**: Frequent or lengthy vibrations may affect battery life, so it’s essential to use the Vibration API judiciously.

```html
<button
    data-controller="pwa--vibration"
    data-action="pwa--vibration#vibrate"
    data-pwa--vibration-pattern-param="[300,300,300,300,300,900,900,300,900,300,900,900,300,300,300,300,300]"
>
SOS
</button>
```

### Parameters

None

### Actions

`vibrate` : starts the vibration sequence given in the parameter `pattern`. See [https://developer.mozilla.org/en-US/docs/Web/API/Navigator/vibrate#pattern](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/vibrate#pattern). If the parameter `interval` (`integer`) is defined, the pattern will be repeated unless stopped or the page refreshed. The `interval` is in milliseconds).

`stop` : stop the persistent vibrations.

### Targets

None

### Events

`pwa--vibration:triggered`&#x20;

`pwa--vibration:stopped`&#x20;

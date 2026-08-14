# Capture

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Capture component provides a comprehensive interface for capturing and recording media streams in your Progressive Web App. It supports both user media (camera and microphone) and display media (screen sharing), with advanced features like region restriction and cropping.

This component is particularly useful for applications that need to:

* Capture video/audio from the user's camera and microphone
* Record screen content or specific application windows
* Crop or restrict the captured area to specific DOM elements
* Process media streams for video conferencing, tutorials, or content creation

## Browser Support

The Media Capture API and Screen Capture API are widely supported in modern browsers. However, some advanced features like `restrictToElement()` and `restrictToRegion()` may have limited support. Always check browser compatibility before implementing these features.

**Usage**

### Basic Camera and Microphone Capture

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/capture') }}>
    <video id="preview" autoplay playsinline></video>

    <button {{ stimulus_action('@pwa/capture', 'media', 'click', {constraints: {audio: true, video: true}}) }}>
        Start Camera
    </button>

    <button {{ stimulus_action('@pwa/capture', 'stop', 'click') }}>
        Stop Capture
    </button>
</div>

<script>
    document.addEventListener('pwa--capture:recorder:start', (event) => {
        document.getElementById('preview').srcObject = event.detail.stream;
    });
</script>
```
{% endcode %}

### Screen Capture with Recording

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/capture') }}>
    <video id="screen-preview" autoplay playsinline></video>

    <button {{ stimulus_action('@pwa/capture', 'capture', 'click', {
        videoConstraints: true,
        audioConstraints: true,
        preferCurrentTab: false,
        selfBrowserSurface: 'include'
    }) }}>
        Start Screen Capture
    </button>

    <button {{ stimulus_action('@pwa/capture', 'record', 'click', {timeSlice: 1000}) }}>
        Start Recording
    </button>

    <button {{ stimulus_action('@pwa/capture', 'stop', 'click') }}>
        Stop
    </button>
</div>

<script>
    const chunks = [];

    document.addEventListener('pwa--capture:recorder:start', (event) => {
        document.getElementById('screen-preview').srcObject = event.detail.stream;
    });

    document.addEventListener('pwa--capture:recorder:data', (event) => {
        chunks.push(event.detail.data);
    });

    document.addEventListener('pwa--capture:recorder:stop', () => {
        const blob = new Blob(chunks, { type: 'video/webm' });
        const url = URL.createObjectURL(blob);
        // Download or process the recorded video
    });
</script>
```
{% endcode %}

### Capture with Region Restriction

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/capture') }}>
    <div id="content-to-share" {{ stimulus_target('@pwa/capture', 'element') }}>
        This specific area will be captured
    </div>

    <video id="restricted-preview" autoplay playsinline></video>

    <button {{ stimulus_action('@pwa/capture', 'capture', 'click', {
        videoConstraints: true,
        audioConstraints: false
    }) }}>
        Capture This Region
    </button>
</div>

<script>
    document.addEventListener('pwa--capture:recorder:start', (event) => {
        document.getElementById('restricted-preview').srcObject = event.detail.stream;
    });
</script>
```
{% endcode %}

### Parameters

#### For the `media` Action

The `media` action captures audio and video from the user's camera and microphone.

* `constraints`: An object specifying media constraints. Default: `{audio: true, video: true}`
  * `audio`: Boolean or constraints object for audio capture
  * `video`: Boolean or constraints object for video capture (can specify resolution, frameRate, facingMode, etc.)

Example:
```javascript
{
    constraints: {
        audio: true,
        video: {
            width: { ideal: 1920 },
            height: { ideal: 1080 },
            facingMode: "user"
        }
    }
}
```

#### For the `capture` Action

The `capture` action initiates screen sharing with advanced options.

* `focusBehavior`: Controls focus behavior when sharing starts. Values: `"focus-captured-surface"` or `"no-focus-change"`
* `videoConstraints`: Video capture constraints. Default: `true`
* `audioConstraints`: Audio capture constraints. Default: `false`
* `monitorTypeSurfaces`: Include monitor-type surfaces. Values: `"include"` or `"exclude"`
* `preferCurrentTab`: Prefer sharing the current browser tab. Boolean value
* `selfBrowserSurface`: Include/exclude the current browser surface. Values: `"include"` or `"exclude"`
* `surfaceSwitching`: Allow switching between surfaces. Values: `"include"` or `"exclude"`
* `systemAudio`: Capture system audio. Values: `"include"` or `"exclude"`

#### For the `record` Action

* `timeSlice`: The number of milliseconds to record into each Blob. If not specified, the entire recording is in a single Blob. Default: `1000`

### Actions

* `media`: Captures audio/video from the user's camera and microphone using the `getUserMedia` API
* `capture`: Initiates screen sharing/capture using the `getDisplayMedia` API
* `record`: Starts recording the captured media stream. Must be called after `media` or `capture`
* `stop`: Stops the current capture session and recording
* `getSupportedConstraints`: Retrieves the media constraints supported by the browser
* `getSupportedDevices`: Lists all available media input devices (cameras, microphones)
* `restrictToElement`: Restricts screen capture to a specific DOM element (requires `element` target)
* `restrictToRegion`: Crops the captured video to a specific region (requires `region` target)

### Targets

* `element`: DOM element to which the screen capture should be restricted. When this target is set and the `capture` action is used, the capture will be automatically restricted to this element's bounds.
* `region`: DOM element defining the region to which the captured video should be cropped. When this target is set and the `capture` action is used, the video will be automatically cropped to this region.

{% hint style="info" %}
The `restrictToElement` and `restrictToRegion` features use the Region Capture and Element Capture APIs, which may not be supported in all browsers. Check compatibility before using these features.
{% endhint %}

### Events

* `pwa--capture:constraints`: Dispatched when supported media constraints are retrieved. Payload: `{constraints}`
* `pwa--capture:devices`: Dispatched when available media devices are enumerated. Payload: `{devices}`
* `pwa--capture:recorder:start`: Dispatched when recording starts. Payload: `{stream}` - the MediaStream object
* `pwa--capture:recorder:data`: Dispatched when recorded data is available. Payload: `{data}` - a Blob containing the recorded media
* `pwa--capture:recorder:stop`: Dispatched when the recording session ends
* `pwa--capture:track:ended`: Dispatched when a media track ends (e.g., user stops sharing)
* `pwa--capture:error`: Dispatched when an error occurs. Payload: `{error}`

### Best Practices

1. **Always handle the `error` event**: Media capture can fail for various reasons (permissions denied, unsupported constraints, etc.)
2. **Clean up resources**: Call the `stop` action when the capture is no longer needed to release camera/screen resources
3. **Request minimal permissions**: Only request the audio/video constraints you actually need
4. **Provide user feedback**: Let users know when recording is active using the `recorder:start` and `recorder:stop` events
5. **Handle track endings**: Listen for the `track:ended` event to detect when the user stops sharing from the browser UI


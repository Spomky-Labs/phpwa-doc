# Barcode Detection

The Barcode Detection component provides an interface to the Web Barcode Detection API, enabling your Progressive Web App to scan and decode various types of barcodes directly from images or video streams. This component supports multiple barcode formats including QR codes, EAN, UPC, and many others.

This component is particularly useful for applications that need to:

* Scan product barcodes for inventory or shopping apps
* Read QR codes for quick access to URLs or information
* Process shipping labels and tracking codes
* Enable contactless check-ins or ticket validation

## Browser Support

The Barcode Detection API is currently supported in Chromium-based browsers (Chrome, Edge, Opera). Support in other browsers may be limited. The component gracefully handles unsupported browsers by dispatching an `unsupported` event.

{% hint style="info" %}
You can check the current browser support status at [caniuse.com/barcode-detector](https://caniuse.com/barcode-detector)
{% endhint %}

## Usage

### Basic Barcode Detection from an Image

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/barcode-detection') }}>
    <img id="barcode-image" src="/path/to/barcode.jpg" alt="Barcode">

    <button {{ stimulus_action('@pwa/barcode-detection', 'detect', 'click', {target: '#barcode-image'}) }}>
        Scan Barcode
    </button>
</div>

<script>
    document.addEventListener('pwa--barcode-detection:detected', (event) => {
        const barcodes = event.detail;
        barcodes.forEach(barcode => {
            console.log('Format:', barcode.format);
            console.log('Value:', barcode.rawValue);
            console.log('Bounding box:', barcode.boundingBox);
        });
    });

    document.addEventListener('pwa--barcode-detection:unsupported', () => {
        alert('Barcode detection is not supported in this browser');
    });
</script>
```
{% endcode %}

### Detection with Specific Formats

You can limit detection to specific barcode formats for better performance:

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/barcode-detection', {formats: ['qr_code', 'ean_13']}) }}>
    <img id="qr-code" src="/path/to/qrcode.png" alt="QR Code">

    <button {{ stimulus_action('@pwa/barcode-detection', 'detect', 'click', {target: '#qr-code'}) }}>
        Scan QR Code
    </button>
</div>

<script>
    document.addEventListener('pwa--barcode-detection:detected', (event) => {
        const barcodes = event.detail;
        if (barcodes.length > 0) {
            const qrCode = barcodes[0];
            window.location.href = qrCode.rawValue; // Navigate to QR code URL
        }
    });
</script>
```
{% endcode %}

### Detection from Video Stream

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/barcode-detection') }}>
    <video id="camera-feed" autoplay playsinline></video>

    <button id="start-camera">Start Camera</button>
    <button {{ stimulus_action('@pwa/barcode-detection', 'detect', 'click', {target: '#camera-feed'}) }}>
        Scan from Video
    </button>
</div>

<script>
    // Start camera
    document.getElementById('start-camera').addEventListener('click', async () => {
        const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
        document.getElementById('camera-feed').srcObject = stream;
    });

    document.addEventListener('pwa--barcode-detection:detected', (event) => {
        const barcodes = event.detail;
        if (barcodes.length > 0) {
            console.log('Detected:', barcodes[0].rawValue);
        }
    });
</script>
```
{% endcode %}

## Parameters

### `formats` Value

Specifies which barcode formats to detect. If not specified or empty, all supported formats will be detected.

* **Type**: Array of strings
* **Default**: `[]` (all supported formats)

**Supported formats** (availability depends on browser):
* `aztec` - Aztec 2D barcode
* `code_39` - Code 39 1D barcode
* `code_93` - Code 93 1D barcode
* `code_128` - Code 128 1D barcode
* `data_matrix` - Data Matrix 2D barcode
* `ean_8` - EAN-8 1D barcode
* `ean_13` - EAN-13 1D barcode
* `itf` - ITF (Interleaved 2 of 5) 1D barcode
* `pdf417` - PDF417 2D barcode
* `qr_code` - QR Code 2D barcode
* `upc_a` - UPC-A 1D barcode
* `upc_e` - UPC-E 1D barcode

Example:
```twig
{{ stimulus_controller('@pwa/barcode-detection', {formats: ['qr_code', 'ean_13', 'upc_a']}) }}
```

## Actions

### `detect`

Scans for barcodes in the specified image or video element.

**Parameters:**
* `target`: CSS selector or DOM element reference containing the image or video to scan

The action can be triggered on any event (click, change, etc.):
```twig
{{ stimulus_action('@pwa/barcode-detection', 'detect', 'click', {target: '#my-image'}) }}
```

## Targets

None

## Events

### `pwa--barcode-detection:detected`

Dispatched when one or more barcodes are successfully detected.

**Payload**: Array of barcode objects, each containing:
* `format` (string): The barcode format (e.g., "qr_code", "ean_13")
* `rawValue` (string): The decoded barcode value
* `boundingBox` (DOMRectReadOnly): Position and dimensions of the barcode in the image
* `cornerPoints` (Array): Four corner points of the detected barcode

Example:
```javascript
document.addEventListener('pwa--barcode-detection:detected', (event) => {
    const barcodes = event.detail;
    barcodes.forEach(barcode => {
        console.log(`Found ${barcode.format}: ${barcode.rawValue}`);
    });
});
```

### `pwa--barcode-detection:unsupported`

Dispatched when the Barcode Detection API is not available in the browser or detector initialization failed.

No payload.

### `pwa--barcode-detection:error`

Dispatched when an error occurs during barcode detection (e.g., invalid target).

**Payload**: `{error}` - Error message or object

## Best Practices

1. **Check browser support**: Always listen for the `unsupported` event and provide a fallback option (e.g., manual entry or file upload)
2. **Limit formats when possible**: Specifying exact formats improves detection speed and accuracy
3. **Use appropriate image quality**: Ensure images are clear and well-lit for better detection rates
4. **Handle errors gracefully**: Listen for the `error` event and provide user-friendly feedback
5. **Camera optimization**: When using video streams, use the `environment` facing mode for rear camera access on mobile devices
6. **Performance consideration**: Avoid calling `detect` too frequently on video streams; consider throttling or using requestAnimationFrame

## Common Use Cases

### Product Scanner

```twig
<div {{ stimulus_controller('@pwa/barcode-detection', {formats: ['ean_13', 'upc_a']}) }}>
    <input type="file" id="product-image" accept="image/*">
</div>

<script>
    document.getElementById('product-image').addEventListener('change', (e) => {
        const img = new Image();
        img.onload = () => {
            const controller = document.querySelector('[data-controller="@pwa/barcode-detection"]');
            controller.dispatchEvent(new CustomEvent('detect', {
                detail: { params: { target: img } }
            }));
        };
        img.src = URL.createObjectURL(e.target.files[0]);
    });
</script>
```

### QR Code URL Handler

```javascript
document.addEventListener('pwa--barcode-detection:detected', (event) => {
    const barcodes = event.detail;
    const qrCodes = barcodes.filter(b => b.format === 'qr_code');

    if (qrCodes.length > 0) {
        const url = qrCodes[0].rawValue;
        if (url.startsWith('http://') || url.startsWith('https://')) {
            if (confirm(`Open ${url}?`)) {
                window.location.href = url;
            }
        }
    }
});
```

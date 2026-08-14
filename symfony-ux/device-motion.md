# Device Motion

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Device Motion component provides access to motion sensor data from your device's accelerometer and gyroscope. This component listens to devicemotion events from the Device Motion API and emits normalized update events containing acceleration, acceleration including gravity, rotation rate, and sampling interval. It automatically manages the permission flow when required by the device.

This component is particularly useful for:

* Creating motion-controlled games and interactive experiences
* Implementing shake-to-undo or shake-to-refresh features
* Building fitness and activity tracking applications
* Detecting device orientation changes for UI adaptation
* Creating augmented reality experiences
* Monitoring device movement for security purposes
* Building immersive storytelling and multimedia applications
* Implementing gesture-based navigation and controls

## Browser Support

The Device Motion API is part of the Generic Sensor API family and relies on the `devicemotion` event which is widely supported across modern browsers.

**Support level:** Good - Available on most modern browsers on mobile devices and laptops with accelerometers/gyroscopes.

{% hint style="warning" %}
On iOS 13+ and some Android browsers, accessing motion sensors requires explicit user permission. The component automatically handles the permission request flow when necessary.
{% endhint %}

{% hint style="info" %}
Desktop devices without built-in accelerometers or gyroscopes will not be able to use this API. The component will emit an `unavailable` event in such cases.
{% endhint %}

## Usage

### Basic Motion Detection

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion') }}>
  <p id="motion-status">Waiting for motion data…</p>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const out = document.getElementById('motion-status');

  host.addEventListener('device-motion:unavailable', () => {
    out.textContent = 'Device Motion API is not available on this device/browser.';
  });

  host.addEventListener('device-motion:permission-granted', () => {
    out.textContent = 'Permission granted. Receiving motion data…';
  });

  host.addEventListener('device-motion:permission-denied', () => {
    out.textContent = 'Permission denied. Cannot access motion sensors.';
  });

  host.addEventListener('device-motion:updated', (e) => {
    const { acceleration, rotationRate, interval } = e.detail;
    out.textContent =
      `Acc: x=${acceleration.x?.toFixed(2)} y=${acceleration.y?.toFixed(2)} z=${acceleration.z?.toFixed(2)} | ` +
      `Rot: α=${rotationRate.alpha?.toFixed(1)} β=${rotationRate.beta?.toFixed(1)} γ=${rotationRate.gamma?.toFixed(1)} | ` +
      `dt=${interval}ms`;
  });
</script>
```
{% endcode %}

### Shake Detection

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion') }}>
  <div id="shake-status" class="p-4 rounded">
    <p>Shake your device to trigger an action!</p>
    <p id="shake-count">Shakes detected: <strong>0</strong></p>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const countEl = document.querySelector('#shake-count strong');
  let shakeCount = 0;
  let lastShake = 0;
  const SHAKE_THRESHOLD = 15; // Acceleration threshold
  const SHAKE_COOLDOWN = 1000; // Minimum ms between shakes

  host.addEventListener('device-motion:updated', (e) => {
    const { accelerationIncludingGravity } = e.detail;
    const now = Date.now();

    // Calculate total acceleration magnitude
    const totalAcceleration = Math.abs(accelerationIncludingGravity.x || 0) +
                             Math.abs(accelerationIncludingGravity.y || 0) +
                             Math.abs(accelerationIncludingGravity.z || 0);

    // Detect shake (high acceleration + cooldown period)
    if (totalAcceleration > SHAKE_THRESHOLD && (now - lastShake) > SHAKE_COOLDOWN) {
      shakeCount++;
      lastShake = now;
      countEl.textContent = shakeCount;

      // Trigger your shake action here
      console.log('Shake detected!');

      // Visual feedback
      document.body.style.animation = 'shake 0.5s';
      setTimeout(() => document.body.style.animation = '', 500);
    }
  });
</script>

<style>
  @keyframes shake {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-10px); }
    75% { transform: translateX(10px); }
  }
</style>
```
{% endcode %}

### Tilt-Based UI Control

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 100 }) }}>
  <div id="tilt-display" class="w-full h-64 flex items-center justify-center">
    <div id="tilt-ball" class="w-16 h-16 bg-blue-500 rounded-full transition-transform"></div>
  </div>
  <p id="tilt-info" class="text-center mt-4"></p>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const ball = document.getElementById('tilt-ball');
  const info = document.getElementById('tilt-info');

  host.addEventListener('device-motion:updated', (e) => {
    const { accelerationIncludingGravity } = e.detail;

    // Use acceleration to control ball position
    const x = (accelerationIncludingGravity.x || 0) * 10;
    const y = (accelerationIncludingGravity.y || 0) * -10;

    ball.style.transform = `translate(${x}px, ${y}px)`;

    info.textContent = `Tilt: X=${x.toFixed(1)}px, Y=${y.toFixed(1)}px`;
  });

  host.addEventListener('device-motion:permission-denied', () => {
    info.textContent = 'Motion sensors permission denied';
  });
</script>
```
{% endcode %}

### Motion-Based Game Controller

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 50 }) }}>
  <canvas id="game-canvas" width="400" height="400"
          style="border: 2px solid #333; background: #f0f0f0;"></canvas>
  <p id="game-status">Tilt your device to move the ball!</p>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const canvas = document.getElementById('game-canvas');
  const ctx = canvas.getContext('2d');
  const status = document.getElementById('game-status');

  // Game state
  let ballX = 200;
  let ballY = 200;
  const ballRadius = 15;
  const friction = 0.95;
  let velocityX = 0;
  let velocityY = 0;

  host.addEventListener('device-motion:updated', (e) => {
    const { accelerationIncludingGravity } = e.detail;

    // Update velocity based on tilt
    velocityX += (accelerationIncludingGravity.x || 0) * 0.5;
    velocityY += (accelerationIncludingGravity.y || 0) * -0.5;

    // Apply friction
    velocityX *= friction;
    velocityY *= friction;

    // Update position
    ballX += velocityX;
    ballY += velocityY;

    // Boundary detection
    if (ballX < ballRadius || ballX > canvas.width - ballRadius) {
      velocityX *= -0.8;
      ballX = Math.max(ballRadius, Math.min(canvas.width - ballRadius, ballX));
    }
    if (ballY < ballRadius || ballY > canvas.height - ballRadius) {
      velocityY *= -0.8;
      ballY = Math.max(ballRadius, Math.min(canvas.height - ballRadius, ballY));
    }

    // Render
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.beginPath();
    ctx.arc(ballX, ballY, ballRadius, 0, Math.PI * 2);
    ctx.fillStyle = '#4A90E2';
    ctx.fill();
    ctx.strokeStyle = '#2C5AA0';
    ctx.lineWidth = 2;
    ctx.stroke();

    status.textContent = `Position: (${ballX.toFixed(0)}, ${ballY.toFixed(0)}) | ` +
                        `Velocity: (${velocityX.toFixed(2)}, ${velocityY.toFixed(2)})`;
  });

  host.addEventListener('device-motion:unavailable', () => {
    status.textContent = 'Device Motion not available - use a mobile device';
  });
</script>
```
{% endcode %}

## Best Practices

### Performance Optimization

{% hint style="success" %}
Always use the `throttleValue` parameter to limit the frequency of motion updates. Motion events can fire very frequently (up to 60Hz), which can impact performance if not throttled appropriately.
{% endhint %}

{% code lineNumbers="true" %}
```twig
{# Throttle to ~10 updates per second for most UI updates #}
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 100 }) }}>
  <!-- Your content -->
</div>

{# For games or high-frequency applications, use lower throttle values #}
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 16 }) }}>
  <!-- 60 FPS motion tracking -->
</div>
```
{% endcode %}

### Permission Handling

{% hint style="warning" %}
On iOS 13+ and some Android devices, motion sensor access requires user permission. Always handle the permission flow gracefully and provide feedback to users about why you need access to motion sensors.
{% endhint %}

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion') }}>
  <div id="motion-permission-status" class="p-4">
    <p>This feature requires motion sensor access.</p>
    <button id="request-btn" class="hidden">Enable Motion Sensors</button>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const status = document.getElementById('motion-permission-status');
  const button = document.getElementById('request-btn');

  host.addEventListener('device-motion:unavailable', () => {
    status.innerHTML = '<p class="text-red-600">Motion sensors are not available on this device.</p>';
  });

  host.addEventListener('device-motion:permission-denied', () => {
    status.innerHTML = `
      <p class="text-yellow-600">Motion sensor access was denied.</p>
      <p class="text-sm">You can enable it in your browser settings if you change your mind.</p>
    `;
  });

  host.addEventListener('device-motion:permission-granted', () => {
    status.innerHTML = '<p class="text-green-600">Motion sensors enabled! Move your device.</p>';
  });
</script>
```
{% endcode %}

### Coordinate System Understanding

{% hint style="info" %}
The Device Motion API uses a standard coordinate system:
- **X-axis**: Points to the right of the device
- **Y-axis**: Points to the top of the device
- **Z-axis**: Points up and out of the screen

Understanding this coordinate system is crucial for implementing motion-based features correctly.
{% endhint %}

### Battery Consideration

{% hint style="warning" %}
Continuous motion sensor monitoring can drain battery. Consider:
- Disabling motion tracking when the app is in the background
- Using appropriate throttle values
- Stopping tracking when not actively needed
{% endhint %}

### Null Value Handling

{% code lineNumbers="true" %}
```javascript
host.addEventListener('device-motion:updated', (e) => {
  const { acceleration, rotationRate } = e.detail;

  // Always check for null values - some devices don't provide all data
  const x = acceleration.x ?? 0;
  const y = acceleration.y ?? 0;
  const z = acceleration.z ?? 0;

  // Some devices might not have gyroscopes
  if (rotationRate.alpha !== null) {
    // Handle rotation data
  }
});
```
{% endcode %}

## Common Use Cases

### 1. Shake to Undo/Refresh

Implement a shake gesture to undo actions or refresh content, similar to many mobile apps.

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion') }} data-last-action="">
  <div id="content">
    <p>Make some changes, then shake to undo!</p>
    <button onclick="makeChange()">Do Something</button>
  </div>
  <div id="undo-toast" class="hidden fixed bottom-4 left-4 right-4 bg-gray-800 text-white p-4 rounded">
    Shake detected! Undo?
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const toast = document.getElementById('undo-toast');
  let lastShake = 0;

  host.addEventListener('device-motion:updated', (e) => {
    const { accelerationIncludingGravity } = e.detail;
    const magnitude = Math.sqrt(
      Math.pow(accelerationIncludingGravity.x || 0, 2) +
      Math.pow(accelerationIncludingGravity.y || 0, 2) +
      Math.pow(accelerationIncludingGravity.z || 0, 2)
    );

    if (magnitude > 20 && Date.now() - lastShake > 1000) {
      lastShake = Date.now();
      showUndoPrompt();
    }
  });

  function showUndoPrompt() {
    toast.classList.remove('hidden');
    setTimeout(() => toast.classList.add('hidden'), 3000);
  }
</script>
```
{% endcode %}

### 2. Step Counter

Track device movement to count steps (simplified implementation).

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 50 }) }}>
  <div class="text-center p-8">
    <h2 class="text-4xl font-bold" id="step-count">0</h2>
    <p class="text-gray-600">Steps Today</p>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const stepCount = document.getElementById('step-count');
  let steps = 0;
  let lastStepTime = 0;
  let previousMagnitude = 0;

  host.addEventListener('device-motion:updated', (e) => {
    const { accelerationIncludingGravity } = e.detail;
    const now = Date.now();

    // Calculate acceleration magnitude
    const magnitude = Math.sqrt(
      Math.pow(accelerationIncludingGravity.x || 0, 2) +
      Math.pow(accelerationIncludingGravity.y || 0, 2) +
      Math.pow(accelerationIncludingGravity.z || 0, 2)
    );

    // Detect step: magnitude crosses threshold with sufficient time gap
    const STEP_THRESHOLD = 12;
    const MIN_STEP_INTERVAL = 300; // ms

    if (magnitude > STEP_THRESHOLD &&
        previousMagnitude <= STEP_THRESHOLD &&
        (now - lastStepTime) > MIN_STEP_INTERVAL) {
      steps++;
      stepCount.textContent = steps;
      lastStepTime = now;
    }

    previousMagnitude = magnitude;
  });
</script>
```
{% endcode %}

### 3. Orientation-Adaptive Content

Display different content or adjust UI based on how the user is holding their device.

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 200 }) }}>
  <div id="orientation-content" class="p-8 text-center transition-all">
    <p id="orientation-text">Hold your device in different orientations</p>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const content = document.getElementById('orientation-content');
  const text = document.getElementById('orientation-text');

  host.addEventListener('device-motion:updated', (e) => {
    const { accelerationIncludingGravity } = e.detail;
    const x = accelerationIncludingGravity.x || 0;
    const y = accelerationIncludingGravity.y || 0;

    // Determine device orientation
    let orientation = 'unknown';
    if (Math.abs(y) > Math.abs(x)) {
      orientation = y > 0 ? 'top-up' : 'upside-down';
    } else {
      orientation = x > 0 ? 'right-side' : 'left-side';
    }

    // Update UI based on orientation
    content.className = `p-8 text-center transition-all orientation-${orientation}`;
    text.textContent = `Device orientation: ${orientation}`;
  });
</script>

<style>
  .orientation-top-up { background: linear-gradient(to bottom, #4A90E2, #fff); }
  .orientation-upside-down { background: linear-gradient(to top, #E24A4A, #fff); }
  .orientation-left-side { background: linear-gradient(to left, #4AE290, #fff); }
  .orientation-right-side { background: linear-gradient(to right, #E2904A, #fff); }
</style>
```
{% endcode %}

### 4. Gesture-Based Navigation

Navigate through content using tilt gestures.

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 100 }) }}>
  <div id="slider" class="overflow-hidden">
    <div id="slides" class="flex transition-transform duration-300">
      <div class="slide min-w-full p-8 bg-blue-100">Slide 1</div>
      <div class="slide min-w-full p-8 bg-green-100">Slide 2</div>
      <div class="slide min-w-full p-8 bg-yellow-100">Slide 3</div>
    </div>
  </div>
  <p id="nav-hint" class="text-center mt-4">Tilt left or right to navigate</p>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__device-motion"]');
  const slides = document.getElementById('slides');
  let currentSlide = 0;
  const totalSlides = 3;
  let lastGesture = 0;
  const GESTURE_COOLDOWN = 1000;
  const TILT_THRESHOLD = 5;

  host.addEventListener('device-motion:updated', (e) => {
    const { accelerationIncludingGravity } = e.detail;
    const tiltX = accelerationIncludingGravity.x || 0;
    const now = Date.now();

    if (now - lastGesture < GESTURE_COOLDOWN) return;

    // Navigate on strong tilt
    if (tiltX > TILT_THRESHOLD && currentSlide < totalSlides - 1) {
      currentSlide++;
      updateSlide();
      lastGesture = now;
    } else if (tiltX < -TILT_THRESHOLD && currentSlide > 0) {
      currentSlide--;
      updateSlide();
      lastGesture = now;
    }
  });

  function updateSlide() {
    slides.style.transform = `translateX(-${currentSlide * 100}%)`;
  }
</script>
```
{% endcode %}

## API Reference

### Parameters

`throttleValue`: Controls how often `updated` events are dispatched to avoid excessive UI updates. Value in milliseconds. Default varies by implementation.

{% code lineNumbers="true" %}
```twig
{# Throttle to 100ms (10 updates/second) #}
<div {{ stimulus_controller('@pwa/device-motion', { throttleValue: 100 }) }}>
</div>
```
{% endcode %}

### Actions

None

### Targets

None

### Events

#### `unavailable`

Fired when the Device Motion API is not supported by the device or browser.

{% code lineNumbers="true" %}
```javascript
host.addEventListener('device-motion:unavailable', () => {
  console.log('Device Motion API is not available');
});
```
{% endcode %}

#### `permission-granted`

Fired when the user grants permission to access motion sensors. The motion listener is now attached and `updated` events will start firing.

{% code lineNumbers="true" %}
```javascript
host.addEventListener('device-motion:permission-granted', () => {
  console.log('Permission granted, motion tracking active');
});
```
{% endcode %}

#### `permission-denied`

Fired when the user denies permission to access motion sensors, or an error occurs during the permission request.

{% code lineNumbers="true" %}
```javascript
host.addEventListener('device-motion:permission-denied', () => {
  console.log('Permission denied');
});
```
{% endcode %}

#### `updated`

Fired on every device motion event (as delivered by the browser and subject to throttling). Contains detailed motion data.

**Event detail structure:**

{% code lineNumbers="true" %}
```typescript
{
  acceleration: {
    x: number | null,  // m/s² (excludes gravity)
    y: number | null,
    z: number | null
  },
  accelerationIncludingGravity: {
    x: number | null,  // m/s² (includes gravity)
    y: number | null,
    z: number | null
  },
  rotationRate: {
    alpha: number | null, // deg/s around Z-axis
    beta: number | null,  // deg/s around X-axis
    gamma: number | null  // deg/s around Y-axis
  },
  interval: number // ms between samples
}
```
{% endcode %}

{% hint style="info" %}
**Coordinate System:**
- `acceleration`: Device acceleration without gravity influence
- `accelerationIncludingGravity`: Total acceleration including Earth's gravity (9.81 m/s²)
- `rotationRate`: Angular velocity around each axis in degrees per second
- `interval`: Time interval between motion samples in milliseconds
{% endhint %}

## Related Components

- [Device Orientation](device-orientation.md) - Access device orientation angles (alpha, beta, gamma)
- [Geolocation](geolocation.md) - Track device position and movement
- [Touch](touch.md) - Handle touch gestures and interactions

## Resources

- [MDN: DeviceMotionEvent](https://developer.mozilla.org/en-US/docs/Web/API/DeviceMotionEvent)
- [W3C Device Orientation Specification](https://www.w3.org/TR/orientation-event/)
- [Can I Use: DeviceMotion](https://caniuse.com/deviceorientation)

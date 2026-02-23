# Presentation and Receiver

The Presentation and Receiver components enable you to display web content on secondary screens (external displays, TVs, projectors, or wireless displays) using the Presentation API. The **Presentation** controller manages the connection from the primary device, while the **Receiver** controller handles incoming connections on the secondary display. Together, they enable multi-screen experiences for presentations, digital signage, gaming, and collaborative applications.

This component pair is particularly useful for:

* Delivering presentations on external displays or projectors
* Creating second-screen experiences for entertainment applications
* Building digital signage solutions
* Implementing collaborative whiteboarding and meeting tools
* Developing multi-display gaming experiences
* Showing media content on TVs via Chromecast or similar devices
* Creating kiosk applications with separate control and display screens
* Building remote display and monitoring dashboards

## Browser Support

The Presentation API has limited but growing support, primarily in Chromium-based browsers.

**Support level:** Limited - Supported in Chrome and Edge on desktop and Android. Requires compatible hardware (Chromecast, wireless displays, or other secondary screens). Not supported in Safari or Firefox as of early 2025.

{% hint style="warning" %}
The Presentation API requires compatible hardware to function. Users need access to:
- Chromecast devices
- Miracast-compatible displays
- Smart TVs with casting capabilities
- Or physical secondary displays connected to the device
{% endhint %}

{% hint style="info" %}
Always check for presentation availability before offering this functionality to users. The Presentation controller emits an `availability-changed` event you can use to show/hide presentation controls.
{% endhint %}

## Architecture

The Presentation API works with two parts:

1. **Presentation Controller (Primary Device)**: Runs on the user's main device (phone, laptop). Initiates and controls the presentation.
2. **Receiver (Secondary Display)**: Runs on the secondary screen. Receives and displays content.

Communication flows bidirectionally between controller and receiver through message passing.

## Usage

### Presentation Controller (Primary Device)

The presentation controller runs on the device that initiates the presentation.

#### Basic Presentation Setup

{% code lineNumbers="true" %}
```twig
{# presentation.html.twig - Controller page #}
<div {{ stimulus_controller('@pwa/presentation', {
  urls: ['/receiver.html']
}) }}>
  <div id="presentation-controls">
    <button id="start-btn" disabled>Present to Second Screen</button>
    <button id="disconnect-btn" class="hidden">Disconnect</button>
    <div id="status"></div>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__presentation"]');
  const startBtn = document.getElementById('start-btn');
  const disconnectBtn = document.getElementById('disconnect-btn');
  const status = document.getElementById('status');

  // Check if presentation displays are available
  host.addEventListener('presentation:availability-changed', (e) => {
    if (e.detail.availability.value) {
      startBtn.disabled = false;
      status.textContent = 'Presentation displays available';
    } else {
      startBtn.disabled = true;
      status.textContent = 'No presentation displays found';
    }
  });

  // Start presentation
  startBtn.addEventListener('click', async () => {
    await host.controller.start();
  });

  // Connection started
  host.addEventListener('presentation:started', (e) => {
    startBtn.classList.add('hidden');
    disconnectBtn.classList.remove('hidden');
    status.textContent = `Connected (ID: ${e.detail.id})`;
  });

  // Terminate presentation
  disconnectBtn.addEventListener('click', () => {
    host.controller.terminate();
  });

  // Connection terminated
  host.addEventListener('presentation:terminated', () => {
    startBtn.classList.remove('hidden');
    disconnectBtn.classList.add('hidden');
    status.textContent = 'Disconnected';
  });
</script>
```
{% endcode %}

#### Sending Messages to Receiver

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/presentation', {
  urls: ['/receiver.html']
}) }}>
  <button id="send-message-btn">Send Slide Update</button>
  <input type="number" id="slide-number" value="1" min="1">
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__presentation"]');
  const sendBtn = document.getElementById('send-message-btn');
  const slideInput = document.getElementById('slide-number');

  sendBtn.addEventListener('click', () => {
    // Send message to receiver
    host.controller.send({
      params: {
        action: 'changeSlide',
        slideNumber: parseInt(slideInput.value)
      }
    });
  });
</script>
```
{% endcode %}

#### Reconnecting to Existing Presentation

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/presentation', {
  urls: ['/receiver.html']
}) }}>
  <button id="reconnect-btn">Reconnect to Presentation</button>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__presentation"]');
  const reconnectBtn = document.getElementById('reconnect-btn');

  // Try to reconnect on page load
  window.addEventListener('load', async () => {
    await host.controller.reconnect();
  });

  reconnectBtn.addEventListener('click', async () => {
    await host.controller.reconnect();
  });

  host.addEventListener('presentation:started', (e) => {
    console.log('Reconnected to presentation:', e.detail.id);
  });
</script>
```
{% endcode %}

### Receiver (Secondary Display)

The receiver runs on the secondary display and handles incoming messages.

#### Basic Receiver Setup

{% code lineNumbers="true" %}
```twig
{# receiver.html.twig - Receiver page #}
<!DOCTYPE html>
<html>
<head>
  <title>Presentation Receiver</title>
  {{ pwa() }}
</head>
<body>
  <div {{ stimulus_controller('@pwa/receiver') }}>
    <div id="presentation-content">
      <h1 id="slide-title">Waiting for presentation...</h1>
      <div id="slide-content"></div>
    </div>
  </div>

  <script type="module">
    const host = document.querySelector('[data-controller="pwa__receiver"]');
    const slideTitle = document.getElementById('slide-title');
    const slideContent = document.getElementById('slide-content');

    // Handle incoming messages from controller
    host.addEventListener('receiver:message', (e) => {
      const { data } = e.detail;

      if (data.action === 'changeSlide') {
        slideTitle.textContent = `Slide ${data.slideNumber}`;
        loadSlideContent(data.slideNumber);
      }
    });

    // Handle connection close
    host.addEventListener('receiver:close', (e) => {
      slideTitle.textContent = 'Presentation ended';
      slideContent.innerHTML = '';
      console.log('Connection closed:', e.detail.reason);
    });

    function loadSlideContent(slideNumber) {
      // Load and display slide content
      fetch(`/api/slides/${slideNumber}`)
        .then(r => r.json())
        .then(slide => {
          slideContent.innerHTML = slide.content;
        });
    }
  </script>
</body>
</html>
```
{% endcode %}

#### Two-Way Communication

{% code lineNumbers="true" %}
```twig
{# Receiver can send messages back to controller #}
<div {{ stimulus_controller('@pwa/receiver') }}>
  <div id="interactive-content">
    <button id="next-slide-btn">Next Slide</button>
    <button id="prev-slide-btn">Previous Slide</button>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__receiver"]');
  let connection = null;

  // Store connection reference when message arrives
  host.addEventListener('receiver:message', (e) => {
    if (e.detail.connection) {
      connection = e.detail.connection;
    }
  });

  // Send messages back to controller
  document.getElementById('next-slide-btn').addEventListener('click', () => {
    if (connection) {
      connection.send(JSON.stringify({ action: 'next' }));
    }
  });

  document.getElementById('prev-slide-btn').addEventListener('click', () => {
    if (connection) {
      connection.send(JSON.stringify({ action: 'previous' }));
    }
  });
</script>
```
{% endcode %}

## Best Practices

### Check Availability First

{% hint style="success" %}
Always check for presentation display availability before showing presentation controls. This prevents showing features that won't work for the user.
{% endhint %}

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/presentation', { urls: ['/receiver.html'] }) }}>
  <div id="presentation-section" class="hidden">
    <button id="present-btn">Present</button>
  </div>
  <div id="no-display-message" class="hidden">
    No presentation displays available
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__presentation"]');

  host.addEventListener('presentation:availability-changed', (e) => {
    const section = document.getElementById('presentation-section');
    const message = document.getElementById('no-display-message');

    if (e.detail.availability.value) {
      section.classList.remove('hidden');
      message.classList.add('hidden');
    } else {
      section.classList.add('hidden');
      message.classList.remove('hidden');
    }
  });
</script>
```
{% endcode %}

### Handle Reconnection Gracefully

{% hint style="info" %}
Connection IDs are persisted in localStorage. Implement reconnection logic to allow users to resume presentations after page refreshes.
{% endhint %}

### Design for Large Screens

{% hint style="success" %}
The receiver typically runs on large displays (TVs, projectors). Design your receiver UI with larger fonts, higher contrast, and appropriate scaling for viewing from a distance.
{% endhint %}

{% code lineNumbers="true" %}
```css
/* receiver.css - Optimize for large displays */
body {
  font-size: 48px;
  line-height: 1.6;
  background: #000;
  color: #fff;
}

h1 {
  font-size: 96px;
  margin: 2rem 0;
}

.slide-content {
  max-width: 1400px;
  margin: 0 auto;
  padding: 4rem;
}
```
{% endcode %}

### Keep Controller and Receiver in Sync

{% hint style="warning" %}
Ensure your controller and receiver stay synchronized. Include state synchronization logic when connections are established or reconnected.
{% endhint %}

{% code lineNumbers="true" %}
```javascript
// Controller: Send current state when connection starts
host.addEventListener('presentation:started', () => {
  const currentState = {
    action: 'init',
    slideNumber: currentSlideNumber,
    timestamp: Date.now()
  };
  host.controller.send({ params: currentState });
});
```
{% endcode %}

### Error Handling

{% code lineNumbers="true" %}
```javascript
// Wrap presentation.start() in try-catch
startBtn.addEventListener('click', async () => {
  try {
    await host.controller.start();
  } catch (error) {
    if (error.name === 'NotAllowedError') {
      alert('User cancelled the presentation selection');
    } else if (error.name === 'NotFoundError') {
      alert('No presentation displays available');
    } else {
      console.error('Presentation error:', error);
      alert('Failed to start presentation');
    }
  }
});
```
{% endcode %}

## Common Use Cases

### 1. Slide Presentation System

{% code lineNumbers="true" %}
```twig
{# Controller #}
<div {{ stimulus_controller('@pwa/presentation', { urls: ['/slides/receiver.html'] }) }}>
  <div class="presentation-controls">
    <button id="start-presentation">Start Presentation</button>
    <div class="slide-nav hidden" id="slide-controls">
      <button id="prev-slide">← Previous</button>
      <span id="slide-counter">1 / 20</span>
      <button id="next-slide">Next →</button>
      <button id="end-presentation">End</button>
    </div>
  </div>

  <div class="slide-preview">
    <img id="preview-img" src="/slides/slide-1.jpg" alt="Current Slide">
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__presentation"]');
  let currentSlide = 1;
  const totalSlides = 20;

  document.getElementById('start-presentation').addEventListener('click', async () => {
    await host.controller.start();
  });

  host.addEventListener('presentation:started', () => {
    document.getElementById('slide-controls').classList.remove('hidden');
    sendSlideUpdate();
  });

  document.getElementById('next-slide').addEventListener('click', () => {
    if (currentSlide < totalSlides) {
      currentSlide++;
      sendSlideUpdate();
    }
  });

  document.getElementById('prev-slide').addEventListener('click', () => {
    if (currentSlide > 1) {
      currentSlide--;
      sendSlideUpdate();
    }
  });

  function sendSlideUpdate() {
    document.getElementById('slide-counter').textContent = `${currentSlide} / ${totalSlides}`;
    document.getElementById('preview-img').src = `/slides/slide-${currentSlide}.jpg`;

    host.controller.send({
      params: {
        action: 'showSlide',
        slideNumber: currentSlide,
        imageUrl: `/slides/slide-${currentSlide}.jpg`
      }
    });
  }

  document.getElementById('end-presentation').addEventListener('click', () => {
    host.controller.terminate();
  });
</script>
```
{% endcode %}

### 2. Digital Signage

{% code lineNumbers="true" %}
```twig
{# Receiver for digital signage #}
<div {{ stimulus_controller('@pwa/receiver') }}>
  <div id="signage-display" class="fullscreen">
    <div id="main-content"></div>
    <div id="ticker"></div>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__receiver"]');
  const mainContent = document.getElementById('main-content');
  const ticker = document.getElementById('ticker');

  host.addEventListener('receiver:message', (e) => {
    const { data } = e.detail;

    switch (data.action) {
      case 'updateContent':
        mainContent.innerHTML = data.html;
        break;

      case 'updateTicker':
        ticker.textContent = data.text;
        animateTicker();
        break;

      case 'showVideo':
        mainContent.innerHTML = `<video autoplay loop src="${data.videoUrl}"></video>`;
        break;
    }
  });

  function animateTicker() {
    ticker.style.animation = 'scroll-left 30s linear infinite';
  }
</script>

<style>
  .fullscreen {
    width: 100vw;
    height: 100vh;
    overflow: hidden;
  }

  #main-content {
    height: 90vh;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  #ticker {
    height: 10vh;
    background: #333;
    color: #fff;
    display: flex;
    align-items: center;
    white-space: nowrap;
  }

  @keyframes scroll-left {
    from { transform: translateX(100%); }
    to { transform: translateX(-100%); }
  }
</style>
```
{% endcode %}

### 3. Multi-Display Gaming

{% code lineNumbers="true" %}
```twig
{# Game controller on primary device #}
<div {{ stimulus_controller('@pwa/presentation', { urls: ['/game/display.html'] }) }}>
  <canvas id="game-controller" width="400" height="600"></canvas>
  <div id="game-controls">
    <button id="start-game">Start Game on TV</button>
  </div>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__presentation"]');
  const canvas = document.getElementById('game-controller');
  const ctx = canvas.getContext('2d');

  let gameState = {
    player: { x: 200, y: 500 },
    enemies: []
  };

  document.getElementById('start-game').addEventListener('click', async () => {
    await host.controller.start();
  });

  host.addEventListener('presentation:started', () => {
    startGameLoop();
  });

  function startGameLoop() {
    setInterval(() => {
      updateGame();
      sendGameState();
    }, 1000 / 60); // 60 FPS
  }

  function updateGame() {
    // Update game logic
    gameState.enemies.forEach(enemy => {
      enemy.y += enemy.speed;
    });

    // Draw on controller display
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawPlayer();
  }

  function sendGameState() {
    host.controller.send({
      params: {
        action: 'updateGame',
        state: gameState
      }
    });
  }

  function drawPlayer() {
    ctx.fillStyle = '#00ff00';
    ctx.fillRect(gameState.player.x - 20, gameState.player.y - 20, 40, 40);
  }

  // Touch controls on phone
  canvas.addEventListener('touchmove', (e) => {
    const touch = e.touches[0];
    const rect = canvas.getBoundingClientRect();
    gameState.player.x = touch.clientX - rect.left;
  });
</script>
```
{% endcode %}

### 4. Collaborative Whiteboard

{% code lineNumbers="true" %}
```twig
{# Shared whiteboard receiver #}
<div {{ stimulus_controller('@pwa/receiver') }}>
  <canvas id="whiteboard" width="1920" height="1080"></canvas>
</div>

<script type="module">
  const host = document.querySelector('[data-controller="pwa__receiver"]');
  const canvas = document.getElementById('whiteboard');
  const ctx = canvas.getContext('2d');

  host.addEventListener('receiver:message', (e) => {
    const { data } = e.detail;

    switch (data.action) {
      case 'draw':
        ctx.beginPath();
        ctx.moveTo(data.fromX, data.fromY);
        ctx.lineTo(data.toX, data.toY);
        ctx.strokeStyle = data.color;
        ctx.lineWidth = data.width;
        ctx.stroke();
        break;

      case 'clear':
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        break;

      case 'addText':
        ctx.font = data.font;
        ctx.fillStyle = data.color;
        ctx.fillText(data.text, data.x, data.y);
        break;
    }
  });
</script>
```
{% endcode %}

## API Reference

### Presentation Controller

#### Values

##### `urls`

**Type:** `Array`
**Required:** Yes

Array of URLs that can be presented on the secondary display. Typically contains a single URL pointing to your receiver page.

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/presentation', {
  urls: ['/receiver.html', '/receiver-alt.html']
}) }}>
```
{% endcode %}

#### Actions

##### `start()`

Initiates a new presentation. Opens a browser dialog for the user to select a display.

**Returns:** `Promise<void>`

{% code lineNumbers="true" %}
```javascript
await host.controller.start();
```
{% endcode %}

##### `reconnect()`

Attempts to reconnect to an existing presentation using the stored connection ID from localStorage.

**Returns:** `Promise<void>`

{% code lineNumbers="true" %}
```javascript
await host.controller.reconnect();
```
{% endcode %}

##### `send(params)`

Sends a message to the receiver.

**Parameters:**
- `params.params` (object): Data to send (will be JSON stringified)

{% code lineNumbers="true" %}
```javascript
host.controller.send({
  params: {
    action: 'update',
    data: { foo: 'bar' }
  }
});
```
{% endcode %}

##### `terminate()`

Closes the presentation connection.

{% code lineNumbers="true" %}
```javascript
host.controller.terminate();
```
{% endcode %}

#### Events

##### `availability-changed`

Fired when presentation display availability changes or on initial connection.

**Event detail:**
- `availability.value` (boolean): Whether presentation displays are available

{% code lineNumbers="true" %}
```javascript
host.addEventListener('presentation:availability-changed', (e) => {
  console.log('Displays available:', e.detail.availability.value);
});
```
{% endcode %}

##### `started`

Fired when a presentation connection is successfully established.

**Event detail:**
- `id` (string): Connection ID

{% code lineNumbers="true" %}
```javascript
host.addEventListener('presentation:started', (e) => {
  console.log('Connection ID:', e.detail.id);
});
```
{% endcode %}

##### `terminated`

Fired when the presentation connection is closed.

**Event detail:**
- `id` (string): Connection ID that was terminated

{% code lineNumbers="true" %}
```javascript
host.addEventListener('presentation:terminated', (e) => {
  console.log('Connection terminated:', e.detail.id);
});
```
{% endcode %}

### Receiver

#### Values

None

#### Actions

None (receiver is passive - it only listens for events)

#### Events

##### `message`

Fired when a message is received from the controller.

**Event detail:**
- `data` (object): Parsed JSON data from controller

{% code lineNumbers="true" %}
```javascript
host.addEventListener('receiver:message', (e) => {
  console.log('Received:', e.detail.data);
});
```
{% endcode %}

##### `close`

Fired when the connection is closed.

**Event detail:**
- `connectionId` (string): Connection ID
- `reason` (string): Close reason
- `message` (string): Close message

{% code lineNumbers="true" %}
```javascript
host.addEventListener('receiver:close', (e) => {
  console.log('Connection closed:', e.detail.reason);
});
```
{% endcode %}

## Related Components

- [Service Worker](service-worker.md) - May be used for offline receiver pages
- [Fullscreen](fullscreen.md) - Often used together for immersive presentation experiences
- [Wake Lock](wake-lock.md) - Keep screen on during presentations

## Resources

- [MDN: Presentation API](https://developer.mozilla.org/en-US/docs/Web/API/Presentation_API)
- [W3C Presentation API Specification](https://w3c.github.io/presentation-api/)
- [Google Developers: Presentation API](https://developers.google.com/cast/docs/web_sender)

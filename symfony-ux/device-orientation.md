# Device Orientation

The Device Orientation component provides access to the Device Orientation API, allowing your application to detect and respond to changes in the physical orientation of a mobile device or tablet. This enables creating immersive experiences that react to how users hold and move their devices.

This component is particularly useful for:

* Gaming experiences with tilt-based controls
* Augmented reality applications
* Compass and navigation apps
* 360° photo/video viewers
* Virtual reality experiences
* Interactive art installations
* Balance and leveling tools
* Educational simulations requiring device movement
* Gesture-based interfaces

## Browser Support

The Device Orientation API is primarily available on mobile devices (smartphones and tablets) with motion sensors. Desktop browsers typically don't support this API as most computers lack the necessary sensors.

**Supported Platforms:**
* iOS Safari: Full support (requires permission on iOS 13+)
* Android Chrome/Edge: Full support
* Android Firefox: Full support
* Desktop browsers: Limited support (requires device with sensors)

{% hint style="warning" %}
On iOS 13+, user permission is required and must be requested through a user gesture. The component handles this automatically but you need to trigger the initial action from a user interaction.
{% endhint %}

## Usage

### Basic Orientation Display

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-orientation', {
    throttle: 100
}) }}>
    <h2>Device Orientation</h2>

    <div id="orientation-display">
        <p>Waiting for orientation data...</p>
    </div>
</div>

<script>
    document.addEventListener('pwa--device-orientation:updated', (event) => {
        const { alpha, beta, gamma, absolute } = event.detail;

        const displayDiv = document.getElementById('orientation-display');

        displayDiv.innerHTML = `
            <div class="orientation-value">
                <strong>Alpha (Z-axis):</strong> ${alpha !== null ? alpha.toFixed(1) + '°' : 'N/A'}
                <div class="description">Rotation around Z-axis (0-360°)</div>
            </div>
            <div class="orientation-value">
                <strong>Beta (X-axis):</strong> ${beta !== null ? beta.toFixed(1) + '°' : 'N/A'}
                <div class="description">Front-to-back tilt (-180 to 180°)</div>
            </div>
            <div class="orientation-value">
                <strong>Gamma (Y-axis):</strong> ${gamma !== null ? gamma.toFixed(1) + '°' : 'N/A'}
                <div class="description">Left-to-right tilt (-90 to 90°)</div>
            </div>
            <div class="orientation-value">
                <strong>Absolute:</strong> ${absolute ? 'Yes' : 'No'}
            </div>
        `;
    });
</script>

<style>
    .orientation-value {
        padding: 15px;
        margin: 10px 0;
        background: #f9fafb;
        border-left: 4px solid #3b82f6;
        border-radius: 4px;
    }

    .orientation-value strong {
        display: block;
        margin-bottom: 5px;
        color: #1f2937;
        font-size: 18px;
    }

    .orientation-value .description {
        font-size: 14px;
        color: #6b7280;
        margin-top: 5px;
    }
</style>
```
{% endcode %}

### Visual Device Representation

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-orientation', {
    throttle: 50
}) }}>
    <h2>3D Device Visualization</h2>

    <div class="device-container">
        <div id="device-3d" class="device-3d">
            <div class="device-face front">Front</div>
            <div class="device-face back">Back</div>
            <div class="device-face top">Top</div>
            <div class="device-face bottom">Bottom</div>
            <div class="device-face left">Left</div>
            <div class="device-face right">Right</div>
        </div>
    </div>

    <div id="orientation-info" class="info-panel"></div>
</div>

<script>
    const device = document.getElementById('device-3d');
    const info = document.getElementById('orientation-info');

    document.addEventListener('pwa--device-orientation:updated', (event) => {
        const { alpha, beta, gamma } = event.detail;

        if (alpha !== null && beta !== null && gamma !== null) {
            // Apply 3D rotation
            device.style.transform = `
                rotateZ(${alpha}deg)
                rotateX(${beta}deg)
                rotateY(${gamma}deg)
            `;

            // Update info
            info.innerHTML = `
                <div>α: <strong>${Math.round(alpha)}°</strong></div>
                <div>β: <strong>${Math.round(beta)}°</strong></div>
                <div>γ: <strong>${Math.round(gamma)}°</strong></div>
            `;
        }
    });
</script>

<style>
    .device-container {
        width: 100%;
        height: 400px;
        perspective: 1000px;
        display: flex;
        align-items: center;
        justify-content: center;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        border-radius: 12px;
        margin-bottom: 20px;
    }

    .device-3d {
        width: 100px;
        height: 200px;
        position: relative;
        transform-style: preserve-3d;
        transition: transform 0.1s ease-out;
    }

    .device-face {
        position: absolute;
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: bold;
        color: white;
        font-size: 14px;
        opacity: 0.9;
    }

    .front {
        background: #3b82f6;
        transform: translateZ(20px);
    }

    .back {
        background: #1e40af;
        transform: translateZ(-20px) rotateY(180deg);
    }

    .top {
        background: #60a5fa;
        transform: rotateX(90deg) translateZ(100px);
    }

    .bottom {
        background: #2563eb;
        transform: rotateX(-90deg) translateZ(100px);
    }

    .left {
        background: #1d4ed8;
        transform: rotateY(-90deg) translateZ(50px);
    }

    .right {
        background: #3b82f6;
        transform: rotateY(90deg) translateZ(50px);
    }

    .info-panel {
        display: flex;
        justify-content: space-around;
        padding: 20px;
        background: #f9fafb;
        border-radius: 8px;
    }

    .info-panel div {
        font-size: 18px;
    }

    .info-panel strong {
        color: #3b82f6;
    }
</style>
```
{% endcode %}

### Compass Application

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-orientation', {
    throttle: 100
}) }}>
    <div class="compass-app">
        <h2>Digital Compass</h2>

        <div class="compass-container">
            <div id="compass" class="compass">
                <div class="compass-rose">
                    <div class="direction north">N</div>
                    <div class="direction east">E</div>
                    <div class="direction south">S</div>
                    <div class="direction west">W</div>
                </div>
                <div class="needle"></div>
            </div>
        </div>

        <div id="compass-reading" class="reading">
            <div class="heading">---°</div>
            <div class="cardinal">---</div>
        </div>
    </div>
</div>

<script>
    const compass = document.getElementById('compass');
    const headingEl = document.querySelector('.reading .heading');
    const cardinalEl = document.querySelector('.reading .cardinal');

    document.addEventListener('pwa--device-orientation:updated', (event) => {
        const { alpha } = event.detail;

        if (alpha !== null) {
            // Rotate compass (opposite direction)
            compass.style.transform = `rotate(${-alpha}deg)`;

            // Update heading
            const heading = Math.round(alpha);
            headingEl.textContent = `${heading}°`;

            // Determine cardinal direction
            cardinalEl.textContent = getCardinalDirection(alpha);
        }
    });

    function getCardinalDirection(degrees) {
        const directions = [
            'N', 'NNE', 'NE', 'ENE',
            'E', 'ESE', 'SE', 'SSE',
            'S', 'SSW', 'SW', 'WSW',
            'W', 'WNW', 'NW', 'NNW'
        ];

        const index = Math.round(degrees / 22.5) % 16;
        return directions[index];
    }
</script>

<style>
    .compass-app {
        max-width: 500px;
        margin: 0 auto;
        padding: 20px;
    }

    .compass-container {
        width: 300px;
        height: 300px;
        margin: 30px auto;
        position: relative;
    }

    .compass {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background: linear-gradient(135deg, #f3f4f6 0%, #e5e7eb 100%);
        border: 8px solid #1f2937;
        position: relative;
        transition: transform 0.1s ease-out;
        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
    }

    .compass-rose {
        width: 100%;
        height: 100%;
        position: relative;
    }

    .direction {
        position: absolute;
        font-weight: bold;
        font-size: 24px;
        width: 40px;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .north {
        top: 10px;
        left: 50%;
        transform: translateX(-50%);
        color: #ef4444;
        font-size: 32px;
    }

    .east {
        right: 10px;
        top: 50%;
        transform: translateY(-50%);
        color: #1f2937;
    }

    .south {
        bottom: 10px;
        left: 50%;
        transform: translateX(-50%);
        color: #1f2937;
    }

    .west {
        left: 10px;
        top: 50%;
        transform: translateY(-50%);
        color: #1f2937;
    }

    .needle {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 4px;
        height: 120px;
        margin-left: -2px;
        margin-top: -60px;
        background: linear-gradient(to bottom, #ef4444 0%, #ef4444 50%, #6b7280 50%, #6b7280 100%);
        transform-origin: center;
        border-radius: 2px;
    }

    .needle::before {
        content: '';
        position: absolute;
        top: -10px;
        left: 50%;
        transform: translateX(-50%);
        width: 0;
        height: 0;
        border-left: 10px solid transparent;
        border-right: 10px solid transparent;
        border-bottom: 20px solid #ef4444;
    }

    .reading {
        text-align: center;
        margin-top: 30px;
    }

    .reading .heading {
        font-size: 48px;
        font-weight: bold;
        color: #1f2937;
        margin-bottom: 10px;
    }

    .reading .cardinal {
        font-size: 32px;
        font-weight: bold;
        color: #3b82f6;
    }
</style>
```
{% endcode %}

### Spirit Level (Leveling Tool)

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-orientation', {
    throttle: 50
}) }}>
    <div class="spirit-level">
        <h2>Spirit Level</h2>

        <div class="level-container">
            <div class="level-horizontal">
                <div class="level-vial">
                    <div id="bubble-horizontal" class="bubble"></div>
                    <div class="markers">
                        <div class="marker center"></div>
                    </div>
                </div>
                <div class="label">Horizontal: <span id="angle-horizontal">0°</span></div>
            </div>

            <div class="level-vertical">
                <div class="level-vial vertical">
                    <div id="bubble-vertical" class="bubble"></div>
                    <div class="markers">
                        <div class="marker center"></div>
                    </div>
                </div>
                <div class="label">Vertical: <span id="angle-vertical">0°</span></div>
            </div>
        </div>

        <div id="level-status" class="status"></div>
    </div>
</div>

<script>
    const bubbleHorizontal = document.getElementById('bubble-horizontal');
    const bubbleVertical = document.getElementById('bubble-vertical');
    const angleHorizontal = document.getElementById('angle-horizontal');
    const angleVertical = document.getElementById('angle-vertical');
    const status = document.getElementById('level-status');

    document.addEventListener('pwa--device-orientation:updated', (event) => {
        const { beta, gamma } = event.detail;

        if (beta !== null && gamma !== null) {
            // Constrain angles to reasonable range
            const tiltX = Math.max(-15, Math.min(15, gamma));
            const tiltY = Math.max(-15, Math.min(15, beta - 90)); // Assuming portrait mode

            // Move bubbles (inverted)
            const bubbleXPos = (tiltX / 15) * 100; // -100 to 100
            const bubbleYPos = (tiltY / 15) * 100;

            bubbleHorizontal.style.transform = `translateX(${bubbleXPos}px)`;
            bubbleVertical.style.transform = `translateY(${-bubbleYPos}px)`;

            // Update angle displays
            angleHorizontal.textContent = `${tiltX.toFixed(1)}°`;
            angleVertical.textContent = `${tiltY.toFixed(1)}°`;

            // Check if level
            const isLevel = Math.abs(tiltX) < 1 && Math.abs(tiltY) < 1;

            if (isLevel) {
                status.textContent = '✓ Level!';
                status.className = 'status level';
            } else {
                status.textContent = 'Not level';
                status.className = 'status not-level';
            }
        }
    });
</script>

<style>
    .spirit-level {
        max-width: 600px;
        margin: 0 auto;
        padding: 20px;
    }

    .level-container {
        display: flex;
        flex-direction: column;
        gap: 40px;
        margin: 30px 0;
    }

    .level-horizontal,
    .level-vertical {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .level-vial {
        width: 400px;
        height: 80px;
        background: #f3f4f6;
        border: 3px solid #1f2937;
        border-radius: 40px;
        position: relative;
        overflow: hidden;
    }

    .level-vial.vertical {
        width: 80px;
        height: 400px;
    }

    .markers {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        pointer-events: none;
    }

    .marker.center {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 4px;
        height: 100%;
        background: #ef4444;
        transform: translate(-50%, -50%);
    }

    .vertical .marker.center {
        width: 100%;
        height: 4px;
    }

    .bubble {
        position: absolute;
        width: 60px;
        height: 60px;
        background: radial-gradient(circle, #fef3c7 0%, #fde047 100%);
        border-radius: 50%;
        border: 2px solid #ca8a04;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        transition: transform 0.1s ease-out;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    }

    .label {
        margin-top: 15px;
        font-size: 18px;
        font-weight: 500;
        color: #1f2937;
    }

    .label span {
        color: #3b82f6;
        font-weight: bold;
    }

    .status {
        text-align: center;
        padding: 20px;
        margin-top: 30px;
        border-radius: 8px;
        font-size: 24px;
        font-weight: bold;
    }

    .status.level {
        background: #d1fae5;
        color: #065f46;
    }

    .status.not-level {
        background: #fee2e2;
        color: #991b1b;
    }
</style>
```
{% endcode %}

### Tilt-Based Game

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-orientation', {
    throttle: 30
}) }}>
    <div class="tilt-game">
        <h2>Tilt Maze Game</h2>

        <div class="game-stats">
            <div>Time: <span id="game-time">0</span>s</div>
            <div>Moves: <span id="move-count">0</span></div>
        </div>

        <div id="game-canvas-container">
            <canvas id="game-canvas" width="400" height="400"></canvas>
        </div>

        <div class="game-controls">
            <button id="start-game-btn">Start Game</button>
            <button id="reset-game-btn">Reset</button>
        </div>
    </div>
</div>

<script>
    const canvas = document.getElementById('game-canvas');
    const ctx = canvas.getContext('2d');

    let gameActive = false;
    let startTime = 0;
    let moveCount = 0;
    let gameInterval = null;

    // Ball properties
    let ballX = 50;
    let ballY = 50;
    let ballVelX = 0;
    let ballVelY = 0;
    const ballRadius = 15;
    const friction = 0.95;

    // Goal properties
    const goalX = 350;
    const goalY = 350;
    const goalRadius = 20;

    // Obstacles
    const obstacles = [
        { x: 150, y: 100, width: 100, height: 20 },
        { x: 100, y: 200, width: 200, height: 20 },
        { x: 200, y: 300, width: 150, height: 20 }
    ];

    document.getElementById('start-game-btn').addEventListener('click', () => {
        startGame();
    });

    document.getElementById('reset-game-btn').addEventListener('click', () => {
        resetGame();
    });

    document.addEventListener('pwa--device-orientation:updated', (event) => {
        if (!gameActive) return;

        const { beta, gamma } = event.detail;

        if (beta !== null && gamma !== null) {
            // Convert orientation to acceleration
            // Adjust for portrait/landscape mode
            const accelX = gamma / 90 * 5; // -5 to 5
            const accelY = (beta - 90) / 90 * 5;

            ballVelX += accelX * 0.5;
            ballVelY += accelY * 0.5;

            // Apply friction
            ballVelX *= friction;
            ballVelY *= friction;

            // Update position
            ballX += ballVelX;
            ballY += ballVelY;

            // Collision with walls
            if (ballX - ballRadius < 0) {
                ballX = ballRadius;
                ballVelX *= -0.5;
            }
            if (ballX + ballRadius > canvas.width) {
                ballX = canvas.width - ballRadius;
                ballVelX *= -0.5;
            }
            if (ballY - ballRadius < 0) {
                ballY = ballRadius;
                ballVelY *= -0.5;
            }
            if (ballY + ballRadius > canvas.height) {
                ballY = canvas.height - ballRadius;
                ballVelY *= -0.5;
            }

            // Collision with obstacles
            obstacles.forEach(obs => {
                if (ballX + ballRadius > obs.x &&
                    ballX - ballRadius < obs.x + obs.width &&
                    ballY + ballRadius > obs.y &&
                    ballY - ballRadius < obs.y + obs.height) {

                    // Simple collision response
                    if (Math.abs(ballVelX) > Math.abs(ballVelY)) {
                        ballVelX *= -0.5;
                    } else {
                        ballVelY *= -0.5;
                    }
                }
            });

            // Check if reached goal
            const distToGoal = Math.sqrt(
                Math.pow(ballX - goalX, 2) + Math.pow(ballY - goalY, 2)
            );

            if (distToGoal < ballRadius + goalRadius) {
                winGame();
            }

            // Count significant moves
            if (Math.abs(ballVelX) > 0.5 || Math.abs(ballVelY) > 0.5) {
                moveCount++;
                document.getElementById('move-count').textContent = moveCount;
            }
        }

        draw();
    });

    function startGame() {
        gameActive = true;
        startTime = Date.now();
        moveCount = 0;

        gameInterval = setInterval(() => {
            const elapsed = Math.floor((Date.now() - startTime) / 1000);
            document.getElementById('game-time').textContent = elapsed;
        }, 1000);

        document.getElementById('start-game-btn').disabled = true;
    }

    function resetGame() {
        gameActive = false;
        ballX = 50;
        ballY = 50;
        ballVelX = 0;
        ballVelY = 0;
        moveCount = 0;

        clearInterval(gameInterval);
        document.getElementById('game-time').textContent = '0';
        document.getElementById('move-count').textContent = '0';
        document.getElementById('start-game-btn').disabled = false;

        draw();
    }

    function winGame() {
        gameActive = false;
        clearInterval(gameInterval);

        const elapsed = Math.floor((Date.now() - startTime) / 1000);
        alert(`You win! Time: ${elapsed}s, Moves: ${moveCount}`);

        document.getElementById('start-game-btn').disabled = false;
    }

    function draw() {
        // Clear canvas
        ctx.fillStyle = '#f0f9ff';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        // Draw obstacles
        ctx.fillStyle = '#1f2937';
        obstacles.forEach(obs => {
            ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
        });

        // Draw goal
        ctx.fillStyle = '#10b981';
        ctx.beginPath();
        ctx.arc(goalX, goalY, goalRadius, 0, Math.PI * 2);
        ctx.fill();

        ctx.fillStyle = '#ffffff';
        ctx.font = 'bold 14px sans-serif';
        ctx.textAlign = 'center';
        ctx.fillText('GOAL', goalX, goalY + 5);

        // Draw ball
        ctx.fillStyle = '#3b82f6';
        ctx.beginPath();
        ctx.arc(ballX, ballY, ballRadius, 0, Math.PI * 2);
        ctx.fill();

        // Draw shadow
        ctx.fillStyle = 'rgba(0, 0, 0, 0.2)';
        ctx.beginPath();
        ctx.arc(ballX + 2, ballY + 2, ballRadius, 0, Math.PI * 2);
        ctx.fill();
    }

    // Initial draw
    draw();
</script>

<style>
    .tilt-game {
        max-width: 500px;
        margin: 0 auto;
        padding: 20px;
    }

    .game-stats {
        display: flex;
        justify-content: space-around;
        padding: 15px;
        background: #f9fafb;
        border-radius: 8px;
        margin-bottom: 20px;
        font-size: 18px;
        font-weight: 500;
    }

    .game-stats span {
        color: #3b82f6;
        font-weight: bold;
    }

    #game-canvas-container {
        border: 3px solid #1f2937;
        border-radius: 8px;
        overflow: hidden;
        margin-bottom: 20px;
    }

    #game-canvas {
        display: block;
        width: 100%;
        height: auto;
    }

    .game-controls {
        display: flex;
        gap: 10px;
    }

    .game-controls button {
        flex: 1;
        padding: 12px;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
    }

    #start-game-btn {
        background: #10b981;
        color: white;
    }

    #start-game-btn:disabled {
        opacity: 0.5;
        cursor: not-allowed;
    }

    #reset-game-btn {
        background: #6b7280;
        color: white;
    }
</style>
```
{% endcode %}

### 360° Photo Viewer

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-orientation', {
    throttle: 50
}) }}>
    <div class="photo-viewer">
        <h2>360° Photo Viewer</h2>
        <p>Move your device to explore the photo</p>

        <div class="viewer-container">
            <div id="photo-viewport" class="viewport">
                <img id="panorama" src="/images/panorama.jpg" alt="360° panorama">
            </div>
        </div>

        <div class="viewer-controls">
            <button id="reset-view">Reset View</button>
        </div>
    </div>
</div>

<script>
    const panorama = document.getElementById('panorama');
    let initialAlpha = null;
    let initialBeta = null;
    let initialGamma = null;

    document.addEventListener('pwa--device-orientation:updated', (event) => {
        const { alpha, beta, gamma } = event.detail;

        if (alpha !== null && beta !== null && gamma !== null) {
            // Store initial orientation
            if (initialAlpha === null) {
                initialAlpha = alpha;
                initialBeta = beta;
                initialGamma = gamma;
            }

            // Calculate relative orientation
            const deltaAlpha = alpha - initialAlpha;
            const deltaBeta = beta - initialBeta;

            // Pan image based on orientation
            // Alpha controls horizontal pan (0-360°)
            const panX = (deltaAlpha / 360) * 100;

            // Beta controls vertical pan (-180 to 180°)
            const panY = (deltaBeta / 180) * 50;

            panorama.style.transform = `translate(${panX}%, ${panY}%) scale(1.5)`;
        }
    });

    document.getElementById('reset-view').addEventListener('click', () => {
        initialAlpha = null;
        initialBeta = null;
        initialGamma = null;
        panorama.style.transform = 'translate(0%, 0%) scale(1.5)';
    });
</script>

<style>
    .photo-viewer {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
    }

    .viewer-container {
        margin: 20px 0;
        border: 3px solid #1f2937;
        border-radius: 8px;
        overflow: hidden;
    }

    .viewport {
        width: 100%;
        height: 400px;
        overflow: hidden;
        position: relative;
        background: #000;
    }

    #panorama {
        width: 200%;
        height: auto;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%) scale(1.5);
        transition: transform 0.1s ease-out;
    }

    .viewer-controls {
        margin-top: 15px;
    }

    .viewer-controls button {
        width: 100%;
        padding: 12px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        font-weight: 500;
        cursor: pointer;
    }
</style>
```
{% endcode %}

## Parameters

### `throttle`

**Type:** `number` (milliseconds)
**Default:** Inherited from controller configuration

Controls how often the `updated` event is dispatched. This prevents overwhelming the UI with too many updates.

```twig
{{ stimulus_controller('@pwa/device-orientation', {
    throttle: 100
}) }}
```

**Recommended values:**
* Smooth visualization: `30-50ms`
* Normal updates: `100ms`
* Battery saving: `200-500ms`

## Actions

None - This component automatically monitors device orientation when connected.

## Targets

None

## Events

### `pwa--device-orientation:updated`

Dispatched when the device orientation changes (subject to throttle delay).

**Payload:**
* `alpha` (number|null): Rotation around Z-axis in degrees (0-360°)
  * Represents compass direction
  * 0° = North, 90° = East, 180° = South, 270° = West
* `beta` (number|null): Rotation around X-axis in degrees (-180 to 180°)
  * Represents front-to-back tilt
  * 0° = device lying flat
  * Positive = tilting forward
  * Negative = tilting backward
* `gamma` (number|null): Rotation around Y-axis in degrees (-90 to 90°)
  * Represents left-to-right tilt
  * 0° = device upright
  * Positive = tilting right
  * Negative = tilting left
* `absolute` (boolean): Whether the orientation is provided relative to Earth's coordinate system

Example:
```javascript
document.addEventListener('pwa--device-orientation:updated', (event) => {
    const { alpha, beta, gamma, absolute } = event.detail;

    console.log('Compass heading:', alpha); // 0-360°
    console.log('Forward tilt:', beta); // -180 to 180°
    console.log('Side tilt:', gamma); // -90 to 90°
    console.log('Uses Earth coords:', absolute);

    // Example: Detect if device is lying flat
    const isFlat = beta !== null && Math.abs(beta) < 10 &&
                   gamma !== null && Math.abs(gamma) < 10;

    if (isFlat) {
        console.log('Device is lying flat');
    }
});
```

## Best Practices

1. **Request permission on iOS**: On iOS 13+, trigger permission request from a user gesture
2. **Use appropriate throttle**: Balance between smoothness and performance
3. **Handle null values**: Orientation values can be null on unsupported devices
4. **Test on real devices**: Emulators don't accurately simulate device orientation
5. **Provide visual feedback**: Show users how to interact with orientation-based features
6. **Consider battery impact**: Continuous orientation monitoring drains battery
7. **Calibrate when needed**: Allow users to reset/calibrate the initial orientation
8. **Handle screen rotation**: Account for portrait/landscape mode changes
9. **Graceful degradation**: Provide alternative controls for devices without sensors
10. **Inform users**: Explain that orientation features require device movement

## Understanding Orientation Axes

### Coordinate System

```
Device in portrait mode (upright):

        α (alpha) - Z-axis
        ↻ Rotation (compass)

β (beta) - X-axis         γ (gamma) - Y-axis
↕ Tilt forward/back       ↔ Tilt left/right
```

### Practical Examples

```javascript
// Detect device lying flat on table
function isDeviceFlat(beta, gamma) {
    return Math.abs(beta) < 10 && Math.abs(gamma) < 10;
}

// Detect device held vertically (portrait)
function isDeviceVertical(beta) {
    return beta !== null && Math.abs(beta - 90) < 20;
}

// Detect device tilted significantly
function isDeviceTilted(beta, gamma, threshold = 30) {
    return Math.abs(beta - 90) > threshold || Math.abs(gamma) > threshold;
}

// Get tilt magnitude
function getTiltMagnitude(beta, gamma) {
    const tiltX = Math.abs(gamma);
    const tiltY = Math.abs(beta - 90);
    return Math.sqrt(tiltX * tiltX + tiltY * tiltY);
}
```

## iOS Permission Handling

On iOS 13+, you must request permission before accessing device orientation:

```javascript
// Check if permission is required
if (typeof DeviceOrientationEvent.requestPermission === 'function') {
    // Must be called from a user gesture
    document.getElementById('enable-orientation').addEventListener('click', async () => {
        try {
            const permission = await DeviceOrientationEvent.requestPermission();
            if (permission === 'granted') {
                // Permission granted - controller will now receive events
                console.log('Orientation permission granted');
            } else {
                console.log('Orientation permission denied');
            }
        } catch (error) {
            console.error('Error requesting orientation permission:', error);
        }
    });
}
```

## Complete Example: AR Pointer

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/device-orientation', {
    throttle: 30
}) }}>
    <div class="ar-pointer">
        <h1>AR Direction Pointer</h1>
        <p>Point your device at different locations</p>

        <div class="ar-view">
            <div id="ar-scene" class="scene">
                <div id="pointer" class="pointer">
                    <div class="arrow"></div>
                    <div class="target-info" id="target-info">---</div>
                </div>

                <div class="horizon-line"></div>

                <div class="poi" style="--angle: 0deg">
                    <div class="poi-marker"></div>
                    <div class="poi-label">North</div>
                </div>

                <div class="poi" style="--angle: 90deg">
                    <div class="poi-marker"></div>
                    <div class="poi-label">East</div>
                </div>

                <div class="poi" style="--angle: 180deg">
                    <div class="poi-marker"></div>
                    <div class="poi-label">South</div>
                </div>

                <div class="poi" style="--angle: 270deg">
                    <div class="poi-marker"></div>
                    <div class="poi-label">West</div>
                </div>
            </div>
        </div>

        <div class="orientation-data" id="orientation-data">
            <div>Heading: <strong>---°</strong></div>
            <div>Pitch: <strong>---°</strong></div>
            <div>Roll: <strong>---°</strong></div>
        </div>
    </div>
</div>

<script>
    const scene = document.getElementById('ar-scene');
    const pointer = document.getElementById('pointer');
    const targetInfo = document.getElementById('target-info');
    const orientationData = document.getElementById('orientation-data');

    // Define targets (in degrees)
    const targets = [
        { angle: 0, name: 'North', color: '#ef4444' },
        { angle: 45, name: 'NE', color: '#f59e0b' },
        { angle: 90, name: 'East', color: '#10b981' },
        { angle: 135, name: 'SE', color: '#3b82f6' },
        { angle: 180, name: 'South', color: '#8b5cf6' },
        { angle: 225, name: 'SW', color: '#ec4899' },
        { angle: 270, name: 'West', color: '#6366f1' },
        { angle: 315, name: 'NW', color: '#14b8a6' }
    ];

    document.addEventListener('pwa--device-orientation:updated', (event) => {
        const { alpha, beta, gamma } = event.detail;

        if (alpha !== null && beta !== null && gamma !== null) {
            // Rotate scene based on heading
            scene.style.transform = `rotateZ(${-alpha}deg) rotateX(${beta - 90}deg)`;

            // Update pointer
            pointer.style.transform = `rotateY(${gamma}deg)`;

            // Find nearest target
            const nearestTarget = findNearestTarget(alpha);
            const distance = Math.abs(normalizeAngle(alpha - nearestTarget.angle));

            if (distance < 15) {
                targetInfo.textContent = `→ ${nearestTarget.name}`;
                targetInfo.style.color = nearestTarget.color;
                targetInfo.style.opacity = '1';
            } else {
                targetInfo.style.opacity = '0.3';
                targetInfo.textContent = '---';
            }

            // Update data display
            orientationData.innerHTML = `
                <div>Heading: <strong>${Math.round(alpha)}°</strong></div>
                <div>Pitch: <strong>${Math.round(beta - 90)}°</strong></div>
                <div>Roll: <strong>${Math.round(gamma)}°</strong></div>
            `;
        }
    });

    function findNearestTarget(angle) {
        let nearest = targets[0];
        let minDiff = Math.abs(normalizeAngle(angle - nearest.angle));

        targets.forEach(target => {
            const diff = Math.abs(normalizeAngle(angle - target.angle));
            if (diff < minDiff) {
                minDiff = diff;
                nearest = target;
            }
        });

        return nearest;
    }

    function normalizeAngle(angle) {
        while (angle > 180) angle -= 360;
        while (angle < -180) angle += 360;
        return angle;
    }
</script>

<style>
    .ar-pointer {
        max-width: 900px;
        margin: 0 auto;
        padding: 20px;
    }

    .ar-view {
        width: 100%;
        height: 500px;
        background: linear-gradient(to bottom, #0ea5e9 0%, #38bdf8 50%, #7dd3fc 100%);
        border-radius: 12px;
        overflow: hidden;
        position: relative;
        margin: 20px 0;
        perspective: 1000px;
    }

    .scene {
        width: 100%;
        height: 100%;
        position: relative;
        transform-style: preserve-3d;
        transition: transform 0.05s ease-out;
    }

    .horizon-line {
        position: absolute;
        top: 50%;
        left: 0;
        right: 0;
        height: 2px;
        background: rgba(255, 255, 255, 0.5);
        transform: translateZ(0);
    }

    .pointer {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 100px;
        height: 100px;
        z-index: 10;
    }

    .arrow {
        width: 0;
        height: 0;
        border-left: 30px solid transparent;
        border-right: 30px solid transparent;
        border-bottom: 60px solid rgba(239, 68, 68, 0.8);
        position: absolute;
        top: 0;
        left: 50%;
        transform: translateX(-50%);
    }

    .target-info {
        position: absolute;
        bottom: -40px;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(0, 0, 0, 0.7);
        color: white;
        padding: 8px 16px;
        border-radius: 20px;
        font-weight: bold;
        white-space: nowrap;
        transition: opacity 0.3s;
    }

    .poi {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%) rotateZ(var(--angle)) translateY(-200px);
    }

    .poi-marker {
        width: 20px;
        height: 20px;
        background: rgba(255, 255, 255, 0.8);
        border: 3px solid #1f2937;
        border-radius: 50%;
        margin: 0 auto;
    }

    .poi-label {
        margin-top: 10px;
        background: rgba(0, 0, 0, 0.7);
        color: white;
        padding: 6px 12px;
        border-radius: 12px;
        font-size: 14px;
        font-weight: bold;
        text-align: center;
        white-space: nowrap;
    }

    .orientation-data {
        display: flex;
        justify-content: space-around;
        padding: 20px;
        background: #f9fafb;
        border-radius: 8px;
        font-size: 18px;
    }

    .orientation-data strong {
        color: #3b82f6;
    }
</style>
```
{% endcode %}

## Troubleshooting

### No orientation data on desktop

**Issue**: Events don't fire on desktop browsers

**Cause**: Most desktops lack motion sensors

**Solution**: Test on actual mobile devices

### Permission denied on iOS

**Issue**: iOS doesn't provide orientation data

**Cause**: Permission not granted or not requested from user gesture

**Solution**:
```javascript
// Must be in response to user click/tap
button.addEventListener('click', async () => {
    if (typeof DeviceOrientationEvent.requestPermission === 'function') {
        await DeviceOrientationEvent.requestPermission();
    }
});
```

### Erratic readings

**Issue**: Orientation values jump around

**Causes**:
- Magnetic interference
- Poor sensor calibration
- Too low throttle value

**Solutions**:
- Increase throttle value
- Apply smoothing/averaging
- Calibrate device sensors

### Wrong orientation in landscape

**Issue**: Values don't match device orientation in landscape mode

**Solution**: Adjust calculations based on screen orientation:
```javascript
const isLandscape = window.orientation === 90 || window.orientation === -90;
if (isLandscape) {
    // Swap beta and gamma
    [adjustedBeta, adjustedGamma] = [gamma, beta];
}
```

## Browser Compatibility

| Platform | Support |
|----------|---------|
| iOS Safari | ✓ Full (requires permission on iOS 13+) |
| Android Chrome | ✓ Full support |
| Android Firefox | ✓ Full support |
| Desktop Chrome | △ Limited (requires sensors) |
| Desktop Firefox | △ Limited (requires sensors) |
| Desktop Safari | △ Limited (requires sensors) |

{% hint style="info" %}
Device Orientation is primarily a mobile feature. Always test on real mobile devices with motion sensors for accurate results.
{% endhint %}

# Battery Status

The Battery Status component provides access to the Battery Status API, enabling your Progressive Web App to monitor the device's battery level, charging status, and remaining time. This allows you to create power-aware applications that can adapt their behavior based on the device's battery condition.

This component is particularly useful for:

* Power-saving modes that reduce functionality when battery is low
* Alerting users before battery-intensive operations
* Adjusting video/image quality based on battery level
* Deferring background tasks when battery is low
* Displaying battery status in system monitoring dashboards
* Warning users about low battery during important tasks
* Disabling power-intensive features when battery is critical
* Optimizing app performance based on charging status

## Browser Support

The Battery Status API has limited browser support and is primarily available on Chromium-based browsers. It requires HTTPS for security reasons.

**Supported Browsers:**
* Chrome/Edge (Desktop and Android): Full support
* Opera: Full support
* Firefox: Removed in version 52 (privacy concerns)
* Safari: Not supported

{% hint style="warning" %}
Always provide fallback behavior for browsers that don't support the Battery Status API. The component will dispatch an `unsupported` event when the API is not available.
{% endhint %}

## Usage

### Basic Battery Display

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery') }}>
    <div class="battery-widget">
        <h2>Battery Status</h2>

        <div id="battery-info">
            <p>Detecting battery status...</p>
        </div>
    </div>
</div>

<script>
    document.addEventListener('pwa--battery:updated', (event) => {
        const { charging, level, chargingTime, dischargingTime } = event.detail;

        const infoDiv = document.getElementById('battery-info');

        infoDiv.innerHTML = `
            <div class="battery-stat">
                <strong>Status:</strong> ${charging ? 'Charging ⚡' : 'Discharging'}
            </div>
            <div class="battery-stat">
                <strong>Level:</strong> ${Math.round(level * 100)}%
            </div>
            <div class="battery-stat">
                <strong>Charging Time:</strong> ${formatTime(chargingTime)}
            </div>
            <div class="battery-stat">
                <strong>Discharging Time:</strong> ${formatTime(dischargingTime)}
            </div>
        `;
    });

    document.addEventListener('pwa--battery:unsupported', () => {
        document.getElementById('battery-info').innerHTML =
            '<p>Battery Status API is not supported on this device.</p>';
    });

    function formatTime(seconds) {
        if (seconds === null || !isFinite(seconds)) {
            return '∞';
        }
        if (seconds === 0) {
            return 'Full';
        }

        const hours = Math.floor(seconds / 3600);
        const minutes = Math.floor((seconds % 3600) / 60);

        if (hours > 0) {
            return `${hours}h ${minutes}m`;
        }
        return `${minutes}m`;
    }
</script>

<style>
    .battery-widget {
        padding: 20px;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .battery-stat {
        padding: 10px 0;
        border-bottom: 1px solid #f3f4f6;
    }

    .battery-stat:last-child {
        border-bottom: none;
    }
</style>
```
{% endcode %}

### Visual Battery Indicator

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery') }}>
    <div class="battery-display">
        <div class="battery-icon">
            <div id="battery-fill" class="battery-fill"></div>
            <div id="battery-charging" class="charging-indicator" style="display: none;">⚡</div>
        </div>

        <div id="battery-percentage">--%</div>

        <div id="battery-details" class="battery-details">
            <small id="battery-time-remaining"></small>
        </div>
    </div>
</div>

<script>
    document.addEventListener('pwa--battery:updated', (event) => {
        const { charging, level, chargingTime, dischargingTime } = event.detail;

        // Update percentage
        const percentage = Math.round(level * 100);
        document.getElementById('battery-percentage').textContent = `${percentage}%`;

        // Update fill level
        const fillEl = document.getElementById('battery-fill');
        fillEl.style.width = `${percentage}%`;

        // Update color based on level
        if (level <= 0.15) {
            fillEl.style.background = '#ef4444';
        } else if (level <= 0.30) {
            fillEl.style.background = '#f59e0b';
        } else {
            fillEl.style.background = '#10b981';
        }

        // Show/hide charging indicator
        const chargingIndicator = document.getElementById('battery-charging');
        chargingIndicator.style.display = charging ? 'block' : 'none';

        // Update time remaining
        const timeEl = document.getElementById('battery-time-remaining');
        if (charging && chargingTime !== Infinity && chargingTime > 0) {
            const hours = Math.floor(chargingTime / 3600);
            const minutes = Math.floor((chargingTime % 3600) / 60);
            timeEl.textContent = `${hours}h ${minutes}m until full`;
        } else if (!charging && dischargingTime !== Infinity && dischargingTime > 0) {
            const hours = Math.floor(dischargingTime / 3600);
            const minutes = Math.floor((dischargingTime % 3600) / 60);
            timeEl.textContent = `${hours}h ${minutes}m remaining`;
        } else {
            timeEl.textContent = charging ? 'Charging...' : 'Calculating...';
        }
    });
</script>

<style>
    .battery-display {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 30px;
        gap: 15px;
    }

    .battery-icon {
        position: relative;
        width: 120px;
        height: 60px;
        border: 3px solid #1f2937;
        border-radius: 6px;
        background: #f9fafb;
        padding: 5px;
    }

    .battery-icon::after {
        content: '';
        position: absolute;
        right: -10px;
        top: 50%;
        transform: translateY(-50%);
        width: 8px;
        height: 20px;
        background: #1f2937;
        border-radius: 0 3px 3px 0;
    }

    .battery-fill {
        height: 100%;
        background: #10b981;
        border-radius: 2px;
        transition: width 0.3s ease, background 0.3s ease;
    }

    .charging-indicator {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: 28px;
        animation: pulse 1s infinite;
    }

    #battery-percentage {
        font-size: 32px;
        font-weight: bold;
        color: #1f2937;
    }

    .battery-details {
        text-align: center;
        color: #6b7280;
    }

    @keyframes pulse {
        0%, 100% {
            opacity: 1;
        }
        50% {
            opacity: 0.5;
        }
    }
</style>
```
{% endcode %}

### Power-Saving Mode

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery') }}>
    <div class="power-aware-app">
        <div id="power-status" class="power-status"></div>

        <div class="content-area">
            <h2>Media Gallery</h2>

            <div id="quality-notice" style="display: none;" class="notice">
                ⚡ Power-saving mode active - showing optimized content
            </div>

            <div id="gallery" class="gallery"></div>
        </div>
    </div>
</div>

<script>
    let powerSavingMode = false;

    document.addEventListener('pwa--battery:updated', (event) => {
        const { charging, level } = event.detail;
        const percentage = Math.round(level * 100);

        // Update power status
        const statusEl = document.getElementById('power-status');
        statusEl.innerHTML = `
            Battery: ${percentage}% ${charging ? '(Charging)' : ''}
        `;

        // Enable power-saving mode if battery is low and not charging
        const shouldEnablePowerSaving = !charging && level < 0.20;

        if (shouldEnablePowerSaving !== powerSavingMode) {
            powerSavingMode = shouldEnablePowerSaving;
            updateAppMode();
        }

        // Update status color
        if (level <= 0.15 && !charging) {
            statusEl.className = 'power-status critical';
        } else if (level <= 0.30 && !charging) {
            statusEl.className = 'power-status warning';
        } else {
            statusEl.className = 'power-status normal';
        }
    });

    function updateAppMode() {
        const notice = document.getElementById('quality-notice');
        const gallery = document.getElementById('gallery');

        notice.style.display = powerSavingMode ? 'block' : 'none';

        // Load images based on power mode
        const images = [
            { id: 1, title: 'Sunset' },
            { id: 2, title: 'Mountain' },
            { id: 3, title: 'Ocean' },
            { id: 4, title: 'Forest' }
        ];

        const quality = powerSavingMode ? 'low' : 'high';

        gallery.innerHTML = images.map(img => `
            <div class="gallery-item">
                <img src="/images/${img.id}-${quality}.jpg" alt="${img.title}">
                <p>${img.title}</p>
            </div>
        `).join('');

        console.log(`Power-saving mode: ${powerSavingMode ? 'ON' : 'OFF'}`);
    }

    // Initial load
    updateAppMode();
</script>

<style>
    .power-status {
        padding: 10px 20px;
        border-radius: 6px;
        font-weight: 500;
        margin-bottom: 20px;
    }

    .power-status.normal {
        background: #d1fae5;
        color: #065f46;
    }

    .power-status.warning {
        background: #fef3c7;
        color: #92400e;
    }

    .power-status.critical {
        background: #fee2e2;
        color: #991b1b;
    }

    .notice {
        padding: 15px;
        background: #fef3c7;
        border-left: 4px solid #f59e0b;
        border-radius: 6px;
        margin-bottom: 20px;
    }

    .gallery {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
        gap: 20px;
    }

    .gallery-item img {
        width: 100%;
        border-radius: 8px;
    }
</style>
```
{% endcode %}

### Low Battery Warning

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery') }}>
    <div class="task-app">
        <h2>Important Task</h2>

        <div id="battery-warning" class="battery-warning" style="display: none;">
            <strong>⚠️ Low Battery Warning</strong>
            <p>Your battery is running low. Consider connecting to a charger to avoid losing your work.</p>
        </div>

        <form id="task-form">
            <textarea id="task-content" rows="10" placeholder="Enter important information..."></textarea>
            <button type="submit" id="submit-btn">Save Task</button>
        </form>

        <div id="auto-save-status"></div>
    </div>
</div>

<script>
    let autoSaveInterval = null;
    let lastBatteryLevel = 1;

    document.addEventListener('pwa--battery:updated', (event) => {
        const { charging, level } = event.detail;

        // Show warning if battery drops below 15%
        const warningEl = document.getElementById('battery-warning');
        if (level < 0.15 && !charging) {
            warningEl.style.display = 'block';

            // Alert user if battery just dropped below 15%
            if (lastBatteryLevel >= 0.15) {
                alert('⚠️ Battery is low! Your work will be auto-saved more frequently.');
            }

            // Increase auto-save frequency
            startAutoSave(10000); // Every 10 seconds
        } else if (level < 0.30 && !charging) {
            warningEl.style.display = 'none';
            startAutoSave(30000); // Every 30 seconds
        } else {
            warningEl.style.display = 'none';
            startAutoSave(60000); // Every minute
        }

        lastBatteryLevel = level;
    });

    function startAutoSave(interval) {
        if (autoSaveInterval) {
            clearInterval(autoSaveInterval);
        }

        autoSaveInterval = setInterval(() => {
            saveContent();
        }, interval);

        updateAutoSaveStatus(interval);
    }

    function saveContent() {
        const content = document.getElementById('task-content').value;
        if (content) {
            localStorage.setItem('taskContent', content);
            localStorage.setItem('taskSavedAt', new Date().toISOString());
            console.log('Auto-saved at', new Date().toLocaleTimeString());
        }
    }

    function updateAutoSaveStatus(interval) {
        const statusEl = document.getElementById('auto-save-status');
        const seconds = interval / 1000;
        statusEl.textContent = `Auto-saving every ${seconds} seconds`;
    }

    document.getElementById('task-form').addEventListener('submit', (e) => {
        e.preventDefault();
        saveContent();
        alert('Task saved successfully!');
    });

    // Load saved content
    window.addEventListener('load', () => {
        const saved = localStorage.getItem('taskContent');
        if (saved) {
            document.getElementById('task-content').value = saved;
        }
    });
</script>

<style>
    .task-app {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
    }

    .battery-warning {
        padding: 15px;
        background: #fee2e2;
        border: 2px solid #ef4444;
        border-radius: 8px;
        margin-bottom: 20px;
    }

    .battery-warning strong {
        display: block;
        margin-bottom: 5px;
        color: #991b1b;
    }

    #task-content {
        width: 100%;
        padding: 15px;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        font-family: inherit;
        font-size: 16px;
    }

    #submit-btn {
        margin-top: 10px;
        padding: 10px 20px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-weight: 500;
    }

    #auto-save-status {
        margin-top: 10px;
        color: #6b7280;
        font-size: 14px;
    }
</style>
```
{% endcode %}

### Adaptive Video Quality

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery') }}>
    <div class="video-player">
        <h2>Video Player</h2>

        <video id="adaptive-video" controls width="800">
            <source id="video-source" src="" type="video/mp4">
        </video>

        <div id="video-quality-info" class="quality-info"></div>
    </div>
</div>

<script>
    const videoQualities = {
        high: '/videos/video-1080p.mp4',
        medium: '/videos/video-720p.mp4',
        low: '/videos/video-480p.mp4'
    };

    let currentQuality = 'medium';

    document.addEventListener('pwa--battery:updated', (event) => {
        const { charging, level } = event.detail;
        const percentage = Math.round(level * 100);

        let recommendedQuality;

        if (charging || level > 0.50) {
            recommendedQuality = 'high';
        } else if (level > 0.25) {
            recommendedQuality = 'medium';
        } else {
            recommendedQuality = 'low';
        }

        if (recommendedQuality !== currentQuality) {
            currentQuality = recommendedQuality;
            updateVideoQuality();
        }

        // Update info display
        const infoEl = document.getElementById('video-quality-info');
        infoEl.innerHTML = `
            <div>Battery: ${percentage}% ${charging ? '(Charging)' : ''}</div>
            <div>Video Quality: <strong>${currentQuality.toUpperCase()}</strong></div>
            ${!charging && level < 0.25 ? '<div class="warning">⚡ Quality reduced to save battery</div>' : ''}
        `;
    });

    function updateVideoQuality() {
        const video = document.getElementById('adaptive-video');
        const source = document.getElementById('video-source');

        const currentTime = video.currentTime;
        const wasPlaying = !video.paused;

        source.src = videoQualities[currentQuality];
        video.load();
        video.currentTime = currentTime;

        if (wasPlaying) {
            video.play();
        }

        console.log(`Video quality changed to: ${currentQuality}`);
    }

    // Initial quality
    updateVideoQuality();
</script>

<style>
    .video-player {
        max-width: 850px;
        margin: 0 auto;
        padding: 20px;
    }

    #adaptive-video {
        width: 100%;
        border-radius: 8px;
    }

    .quality-info {
        margin-top: 15px;
        padding: 15px;
        background: #f9fafb;
        border-radius: 6px;
    }

    .quality-info > div {
        padding: 5px 0;
    }

    .quality-info .warning {
        color: #f59e0b;
        font-weight: 500;
    }
</style>
```
{% endcode %}

### Battery Monitor Dashboard

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery') }}>
    <div class="battery-dashboard">
        <div class="dashboard-header">
            <h1>Battery Monitor</h1>
            <div id="last-updated">Never updated</div>
        </div>

        <div class="stats-grid">
            <div class="stat-card">
                <div class="stat-label">Battery Level</div>
                <div id="stat-level" class="stat-value">--%</div>
                <div id="stat-level-bar" class="stat-bar">
                    <div class="stat-bar-fill"></div>
                </div>
            </div>

            <div class="stat-card">
                <div class="stat-label">Status</div>
                <div id="stat-status" class="stat-value">--</div>
            </div>

            <div class="stat-card">
                <div class="stat-label">Time to Full</div>
                <div id="stat-charging-time" class="stat-value">--</div>
            </div>

            <div class="stat-card">
                <div class="stat-label">Time Remaining</div>
                <div id="stat-discharging-time" class="stat-value">--</div>
            </div>
        </div>

        <div class="chart-container">
            <h3>Battery Level History</h3>
            <canvas id="battery-chart" width="600" height="200"></canvas>
        </div>
    </div>
</div>

<script>
    const batteryHistory = [];
    const MAX_HISTORY_POINTS = 30;

    document.addEventListener('pwa--battery:updated', (event) => {
        const { charging, level, chargingTime, dischargingTime } = event.detail;
        const percentage = Math.round(level * 100);

        // Update stats
        document.getElementById('stat-level').textContent = `${percentage}%`;
        document.getElementById('stat-status').textContent = charging ? 'Charging ⚡' : 'Discharging';

        // Update level bar
        const barFill = document.querySelector('#stat-level-bar .stat-bar-fill');
        barFill.style.width = `${percentage}%`;

        if (level <= 0.15) {
            barFill.style.background = '#ef4444';
        } else if (level <= 0.30) {
            barFill.style.background = '#f59e0b';
        } else {
            barFill.style.background = '#10b981';
        }

        // Update times
        document.getElementById('stat-charging-time').textContent =
            formatTime(chargingTime);
        document.getElementById('stat-discharging-time').textContent =
            formatTime(dischargingTime);

        // Update last updated time
        document.getElementById('last-updated').textContent =
            `Last updated: ${new Date().toLocaleTimeString()}`;

        // Add to history
        batteryHistory.push({
            timestamp: Date.now(),
            level: percentage
        });

        if (batteryHistory.length > MAX_HISTORY_POINTS) {
            batteryHistory.shift();
        }

        // Update chart
        drawChart();
    });

    function formatTime(seconds) {
        if (seconds === null || !isFinite(seconds)) {
            return '∞';
        }
        if (seconds === 0) {
            return 'Full';
        }

        const hours = Math.floor(seconds / 3600);
        const minutes = Math.floor((seconds % 3600) / 60);

        if (hours > 0) {
            return `${hours}h ${minutes}m`;
        }
        return `${minutes}m`;
    }

    function drawChart() {
        const canvas = document.getElementById('battery-chart');
        const ctx = canvas.getContext('2d');

        // Clear canvas
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        if (batteryHistory.length < 2) return;

        // Draw grid
        ctx.strokeStyle = '#e5e7eb';
        ctx.lineWidth = 1;

        for (let i = 0; i <= 4; i++) {
            const y = (canvas.height * i) / 4;
            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(canvas.width, y);
            ctx.stroke();

            // Labels
            ctx.fillStyle = '#9ca3af';
            ctx.font = '12px sans-serif';
            ctx.fillText(`${100 - (i * 25)}%`, 5, y - 5);
        }

        // Draw line
        ctx.strokeStyle = '#3b82f6';
        ctx.lineWidth = 2;
        ctx.beginPath();

        batteryHistory.forEach((point, index) => {
            const x = (canvas.width * index) / (batteryHistory.length - 1);
            const y = canvas.height - (canvas.height * point.level / 100);

            if (index === 0) {
                ctx.moveTo(x, y);
            } else {
                ctx.lineTo(x, y);
            }
        });

        ctx.stroke();

        // Draw points
        ctx.fillStyle = '#3b82f6';
        batteryHistory.forEach((point, index) => {
            const x = (canvas.width * index) / (batteryHistory.length - 1);
            const y = canvas.height - (canvas.height * point.level / 100);

            ctx.beginPath();
            ctx.arc(x, y, 3, 0, 2 * Math.PI);
            ctx.fill();
        });
    }
</script>

<style>
    .battery-dashboard {
        max-width: 1000px;
        margin: 0 auto;
        padding: 20px;
    }

    .dashboard-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 30px;
    }

    #last-updated {
        color: #6b7280;
        font-size: 14px;
    }

    .stats-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 20px;
        margin-bottom: 30px;
    }

    .stat-card {
        padding: 20px;
        background: white;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .stat-label {
        font-size: 14px;
        color: #6b7280;
        margin-bottom: 8px;
    }

    .stat-value {
        font-size: 28px;
        font-weight: bold;
        color: #1f2937;
    }

    .stat-bar {
        margin-top: 10px;
        height: 8px;
        background: #e5e7eb;
        border-radius: 4px;
        overflow: hidden;
    }

    .stat-bar-fill {
        height: 100%;
        background: #10b981;
        transition: width 0.3s ease, background 0.3s ease;
    }

    .chart-container {
        padding: 20px;
        background: white;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .chart-container h3 {
        margin: 0 0 15px 0;
    }

    #battery-chart {
        width: 100%;
        height: auto;
    }
</style>
```
{% endcode %}

### Integration with Symfony Live Components

For applications using Symfony UX Live Components, here's an advanced integration example:

{% code title="src/Twig/Component/Battery.php" lineNumbers="true" %}
```php
<?php

namespace App\Twig\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\Attribute\LiveArg;
use Symfony\UX\LiveComponent\Attribute\LiveListener;
use Symfony\UX\LiveComponent\Attribute\LiveProp;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\ComponentToolsTrait;

#[AsLiveComponent('Battery')]
class Battery
{
    use DefaultActionTrait;
    use ComponentToolsTrait;

    #[LiveProp]
    public string $charging = '';

    #[LiveProp]
    public string $level = '';

    #[LiveProp]
    public string $chargingTime = '';

    #[LiveProp]
    public string $dischargingTime = '';

    #[LiveProp]
    public string $statusClass = 'normal';

    #[LiveListener('pwa--battery:updated')]
    public function onUpdate(
        #[LiveArg] bool $charging,
        #[LiveArg] float $level,
        #[LiveArg] ?int $chargingTime,
        #[LiveArg] ?int $dischargingTime,
    ): void {
        $this->charging = $charging ? 'Yes' : 'No';
        $this->level = ($level * 100) . '%';
        $this->chargingTime = $this->formatSeconds($chargingTime);
        $this->dischargingTime = $this->formatSeconds($dischargingTime);

        // Determine status class for styling
        if (!$charging && $level <= 0.15) {
            $this->statusClass = 'critical';
        } elseif (!$charging && $level <= 0.30) {
            $this->statusClass = 'warning';
        } else {
            $this->statusClass = 'normal';
        }
    }

    private function formatSeconds(?float $s): string
    {
        if ($s === null || !is_finite($s)) {
            return '∞';
        }

        if ($s === 0.0) {
            return 'Full';
        }

        $h = str_pad((string) floor($s / 3600), 2, '0', STR_PAD_LEFT);
        $m = str_pad((string) floor(($s % 3600) / 60), 2, '0', STR_PAD_LEFT);
        $sec = str_pad((string) floor($s % 60), 2, '0', STR_PAD_LEFT);

        return "$h:$m:$sec";
    }
}
```
{% endcode %}

{% code title="templates/components/Battery.html.twig" lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery', '@pwa/live') }} {{ attributes }}>
    <div class="battery-component battery-{{ this.statusClass }}">
        <div class="battery-header">
            <h3>Battery Status</h3>
            <span class="battery-badge">{{ this.charging == 'Yes' ? '⚡ Charging' : 'Discharging' }}</span>
        </div>

        <dl class="battery-stats">
            <div class="stat-row">
                <dt>Level</dt>
                <dd>{{ this.level }}</dd>
            </div>
            <div class="stat-row">
                <dt>Charging</dt>
                <dd>{{ this.charging }}</dd>
            </div>
            <div class="stat-row">
                <dt>Time to Full</dt>
                <dd>{{ this.chargingTime }}</dd>
            </div>
            <div class="stat-row">
                <dt>Time Remaining</dt>
                <dd>{{ this.dischargingTime }}</dd>
            </div>
        </dl>
    </div>
</div>

<style>
    .battery-component {
        padding: 20px;
        border-radius: 8px;
        border: 2px solid #e5e7eb;
    }

    .battery-component.battery-normal {
        border-color: #10b981;
        background: #f0fdf4;
    }

    .battery-component.battery-warning {
        border-color: #f59e0b;
        background: #fef3c7;
    }

    .battery-component.battery-critical {
        border-color: #ef4444;
        background: #fee2e2;
    }

    .battery-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 15px;
    }

    .battery-badge {
        padding: 4px 12px;
        background: white;
        border-radius: 12px;
        font-size: 14px;
        font-weight: 500;
    }

    .battery-stats {
        margin: 0;
    }

    .stat-row {
        display: flex;
        justify-content: space-between;
        padding: 10px 0;
        border-bottom: 1px solid rgba(0, 0, 0, 0.1);
    }

    .stat-row:last-child {
        border-bottom: none;
    }

    .stat-row dt {
        font-weight: 500;
        color: #6b7280;
    }

    .stat-row dd {
        margin: 0;
        font-weight: 600;
        color: #1f2937;
    }
</style>
```
{% endcode %}

Then use the component in your templates:

```twig
<twig:Battery />
```

## Parameters

None

## Actions

None - This component automatically monitors battery status and dispatches events when it changes.

## Targets

None

## Events

### `pwa--battery:updated`

Dispatched when the battery status changes or when the controller is initially connected.

**Payload:**
* `charging` (boolean): Whether the battery is currently charging
* `level` (number): Battery level from 0 to 1 (multiply by 100 for percentage)
* `chargingTime` (number|null): Seconds until battery is fully charged (Infinity if not charging or unknown)
* `dischargingTime` (number|null): Seconds until battery is fully discharged (Infinity if charging or unknown)

Example:
```javascript
document.addEventListener('pwa--battery:updated', (event) => {
    const { charging, level, chargingTime, dischargingTime } = event.detail;

    const percentage = Math.round(level * 100);
    console.log(`Battery: ${percentage}%`);
    console.log(`Charging: ${charging}`);

    if (charging) {
        console.log(`Time to full: ${chargingTime} seconds`);
    } else {
        console.log(`Time remaining: ${dischargingTime} seconds`);
    }

    // Adapt app behavior based on battery
    if (percentage < 15 && !charging) {
        enablePowerSavingMode();
    }
});
```

### `pwa--battery:unsupported`

Dispatched when the Battery Status API is not supported by the browser.

**No payload**

Example:
```javascript
document.addEventListener('pwa--battery:unsupported', () => {
    console.log('Battery Status API not supported');
    // Show fallback UI or disable battery-dependent features
    document.getElementById('battery-widget').style.display = 'none';
});
```

## Best Practices

1. **Always provide fallbacks**: Not all browsers support the Battery API
2. **Respect user privacy**: Don't share battery information with third parties
3. **Use sparingly**: Battery information is primarily for power optimization
4. **Test on devices**: Battery behavior varies significantly between devices
5. **Handle edge cases**: Battery values can be null or Infinity
6. **Don't be intrusive**: Subtle power-saving adjustments are better than aggressive changes
7. **Inform users**: Tell users when you're adapting behavior based on battery
8. **Combine with other signals**: Use battery status alongside other performance indicators
9. **Monitor updates**: Battery status can change frequently - be prepared
10. **Graceful degradation**: Core functionality should work regardless of battery level

## Common Use Cases

### Power-Saving Mode Activation

```javascript
let powerSavingEnabled = false;

document.addEventListener('pwa--battery:updated', (event) => {
    const { charging, level } = event.detail;

    const shouldEnablePowerSaving = !charging && level < 0.20;

    if (shouldEnablePowerSaving !== powerSavingEnabled) {
        powerSavingEnabled = shouldEnablePowerSaving;

        if (powerSavingEnabled) {
            // Reduce animations
            document.body.classList.add('reduce-motion');

            // Lower video quality
            adjustVideoQuality('low');

            // Disable background sync
            disableBackgroundSync();

            // Notify user
            showNotification('Power-saving mode enabled');
        } else {
            document.body.classList.remove('reduce-motion');
            adjustVideoQuality('auto');
            enableBackgroundSync();
        }
    }
});
```

### Background Task Scheduling

```javascript
document.addEventListener('pwa--battery:updated', (event) => {
    const { charging, level } = event.detail;

    // Only run background tasks when battery is good or charging
    if (charging || level > 0.30) {
        scheduleBackgroundTasks();
    } else {
        cancelBackgroundTasks();
    }
});

function scheduleBackgroundTasks() {
    // Schedule non-critical tasks like:
    // - Cache prefetching
    // - Analytics uploads
    // - Database cleanup
}
```

### Critical Operation Warning

```javascript
function performExpensiveOperation() {
    const batteryStatus = getBatteryStatus(); // Store battery status globally

    if (!batteryStatus.charging && batteryStatus.level < 0.20) {
        const confirmed = confirm(
            'Your battery is low. This operation may consume significant power. Continue?'
        );

        if (!confirmed) {
            return;
        }
    }

    // Proceed with operation
    doExpensiveWork();
}
```

## Complete Example: Adaptive Application

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/battery') }}>
    <div class="adaptive-app">
        <header class="app-header">
            <h1>Adaptive App</h1>
            <div id="battery-indicator" class="battery-indicator">
                <div id="battery-icon"></div>
                <span id="battery-percent">--%</span>
            </div>
        </header>

        <div id="power-mode-notice" class="notice" style="display: none;"></div>

        <main class="app-content">
            <section class="feature-section">
                <h2>Features</h2>

                <div class="feature-grid">
                    <div class="feature-card" id="feature-animations">
                        <h3>🎬 Animations</h3>
                        <p class="feature-status">Enabled</p>
                    </div>

                    <div class="feature-card" id="feature-hq-images">
                        <h3>🖼️ HQ Images</h3>
                        <p class="feature-status">Enabled</p>
                    </div>

                    <div class="feature-card" id="feature-background-sync">
                        <h3>🔄 Background Sync</h3>
                        <p class="feature-status">Enabled</p>
                    </div>

                    <div class="feature-card" id="feature-auto-update">
                        <h3>⚡ Auto Update</h3>
                        <p class="feature-status">Enabled</p>
                    </div>
                </div>
            </section>

            <section class="stats-section">
                <h2>Performance Metrics</h2>
                <div id="metrics"></div>
            </section>
        </main>
    </div>
</div>

<script>
    const features = {
        animations: true,
        hqImages: true,
        backgroundSync: true,
        autoUpdate: true
    };

    document.addEventListener('pwa--battery:updated', (event) => {
        const { charging, level } = event.detail;
        const percentage = Math.round(level * 100);

        // Update battery indicator
        updateBatteryIndicator(percentage, charging);

        // Determine power mode
        let powerMode = 'normal';
        if (charging || level > 0.50) {
            powerMode = 'normal';
        } else if (level > 0.20) {
            powerMode = 'balanced';
        } else {
            powerMode = 'powersaver';
        }

        // Adapt features based on power mode
        adaptFeatures(powerMode, charging);

        // Update metrics
        updateMetrics({ percentage, charging, powerMode });
    });

    function updateBatteryIndicator(percentage, charging) {
        const icon = document.getElementById('battery-icon');
        const percent = document.getElementById('battery-percent');

        percent.textContent = `${percentage}%`;

        // Update icon
        if (charging) {
            icon.textContent = '⚡';
        } else if (percentage <= 15) {
            icon.textContent = '🔴';
        } else if (percentage <= 30) {
            icon.textContent = '🟡';
        } else {
            icon.textContent = '🟢';
        }
    }

    function adaptFeatures(powerMode, charging) {
        const notice = document.getElementById('power-mode-notice');

        switch (powerMode) {
            case 'powersaver':
                features.animations = false;
                features.hqImages = false;
                features.backgroundSync = false;
                features.autoUpdate = false;

                notice.textContent = '⚡ Power Saver Mode: All non-essential features disabled';
                notice.style.display = 'block';
                notice.className = 'notice critical';
                break;

            case 'balanced':
                features.animations = false;
                features.hqImages = false;
                features.backgroundSync = true;
                features.autoUpdate = false;

                notice.textContent = '⚖️ Balanced Mode: Some features disabled to conserve battery';
                notice.style.display = 'block';
                notice.className = 'notice warning';
                break;

            default:
                features.animations = true;
                features.hqImages = true;
                features.backgroundSync = true;
                features.autoUpdate = true;

                notice.style.display = 'none';
                break;
        }

        updateFeatureCards();
        applyFeatureSettings();
    }

    function updateFeatureCards() {
        Object.keys(features).forEach(key => {
            const card = document.getElementById(`feature-${key.replace(/([A-Z])/g, '-$1').toLowerCase()}`);
            const status = card.querySelector('.feature-status');

            if (features[key]) {
                status.textContent = 'Enabled';
                status.style.color = '#10b981';
                card.style.opacity = '1';
            } else {
                status.textContent = 'Disabled';
                status.style.color = '#ef4444';
                card.style.opacity = '0.6';
            }
        });
    }

    function applyFeatureSettings() {
        // Apply actual feature settings
        if (features.animations) {
            document.body.classList.remove('reduce-motion');
        } else {
            document.body.classList.add('reduce-motion');
        }

        // In a real app, you would:
        // - Adjust image quality
        // - Enable/disable background sync
        // - Control auto-update behavior
    }

    function updateMetrics({ percentage, charging, powerMode }) {
        const metrics = document.getElementById('metrics');

        metrics.innerHTML = `
            <div class="metric">
                <span class="metric-label">Battery:</span>
                <span class="metric-value">${percentage}%</span>
            </div>
            <div class="metric">
                <span class="metric-label">Charging:</span>
                <span class="metric-value">${charging ? 'Yes' : 'No'}</span>
            </div>
            <div class="metric">
                <span class="metric-label">Power Mode:</span>
                <span class="metric-value">${powerMode}</span>
            </div>
            <div class="metric">
                <span class="metric-label">Active Features:</span>
                <span class="metric-value">${Object.values(features).filter(v => v).length}/4</span>
            </div>
        `;
    }
</script>

<style>
    .adaptive-app {
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
    }

    .app-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 30px;
    }

    .battery-indicator {
        display: flex;
        align-items: center;
        gap: 10px;
        padding: 10px 20px;
        background: #f9fafb;
        border: 1px solid #e5e7eb;
        border-radius: 20px;
    }

    #battery-icon {
        font-size: 24px;
    }

    #battery-percent {
        font-weight: 600;
    }

    .notice {
        padding: 15px 20px;
        border-radius: 8px;
        margin-bottom: 20px;
        font-weight: 500;
    }

    .notice.warning {
        background: #fef3c7;
        color: #92400e;
        border-left: 4px solid #f59e0b;
    }

    .notice.critical {
        background: #fee2e2;
        color: #991b1b;
        border-left: 4px solid #ef4444;
    }

    .feature-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 20px;
        margin-bottom: 40px;
    }

    .feature-card {
        padding: 20px;
        background: white;
        border: 2px solid #e5e7eb;
        border-radius: 8px;
        transition: opacity 0.3s;
    }

    .feature-card h3 {
        margin: 0 0 10px 0;
    }

    .feature-status {
        margin: 0;
        font-weight: 600;
    }

    .stats-section h2 {
        margin-bottom: 15px;
    }

    #metrics {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
    }

    .metric {
        padding: 15px;
        background: #f9fafb;
        border-radius: 6px;
        display: flex;
        justify-content: space-between;
    }

    .metric-label {
        color: #6b7280;
    }

    .metric-value {
        font-weight: 600;
        color: #1f2937;
    }

    body.reduce-motion * {
        animation: none !important;
        transition: none !important;
    }
</style>
```
{% endcode %}

## Troubleshooting

### API not supported

**Issue**: Battery Status API not available

**Solution**: Always handle the `unsupported` event and provide fallback behavior:
```javascript
document.addEventListener('pwa--battery:unsupported', () => {
    // Hide battery-dependent features
    // Use default power-neutral settings
});
```

### Values are null or undefined

**Issue**: Battery properties return null

**Cause**: Some properties may not be available on all devices

**Solution**: Always check for null/undefined:
```javascript
if (chargingTime !== null && isFinite(chargingTime)) {
    // Use the value
}
```

### Inaccurate remaining time

**Issue**: Charging/discharging time estimates are inaccurate

**Cause**: Time estimates are calculated by the OS and can be unreliable

**Solution**: Treat time values as estimates, not precise measurements

### HTTPS requirement

**Issue**: Battery API not working on HTTP

**Solution**: The Battery Status API requires a secure context (HTTPS or localhost)

## Browser Compatibility

| Browser | Support |
|---------|---------|
| Chrome/Edge (Desktop) | ✓ Full support |
| Chrome/Edge (Android) | ✓ Full support |
| Firefox | ✗ Removed (v52+) |
| Safari | ✗ Not supported |
| Opera | ✓ Full support |

{% hint style="danger" %}
Due to limited browser support and privacy concerns, always provide comprehensive fallback behavior when the Battery Status API is unavailable.
{% endhint %}

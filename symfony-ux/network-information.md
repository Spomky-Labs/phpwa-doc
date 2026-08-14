# Network Information

{% hint style="danger" %}
**Deprecated since 1.6.0, removed in 2.0.0.**

The Stimulus controllers are leaving the bundle. Copy the ones you use into your own application
and register them there: they are plain Stimulus controllers with no dependency on the bundle.

See the [upgrade guide](../upgrades/from-1.5.x-to-1.6.0.md) and
[the rationale](https://github.com/Spomky-Labs/pwa-bundle/issues/372#issuecomment-5295710299).
{% endhint %}

The Network Information component provides access to the Network Information API, allowing your Progressive Web App to detect and respond to changes in the user's network connection. This enables you to create adaptive experiences that optimize content delivery based on connection quality and type.

This component is particularly useful for:

* Adaptive content loading based on connection speed
* Video/image quality adjustment for bandwidth optimization
* Automatic quality degradation on slow connections
* Data saver mode detection and optimization
* Preloading strategies based on connection type
* Network-aware caching policies
* Offline/online state detection
* User experience optimization for different connection types
* Bandwidth-conscious feature toggling

## Browser Support

The Network Information API is currently supported in Chromium-based browsers (Chrome, Edge, Opera) and partially in Firefox. Safari does not support this API.

**Supported Properties:**
* `effectiveType`: Widely supported (Chrome, Edge, Opera, Firefox)
* `downlink`: Widely supported (Chrome, Edge, Opera)
* `rtt`: Widely supported (Chrome, Edge, Opera)
* `saveData`: Widely supported (Chrome, Edge, Opera, Firefox)
* `type`: Limited support (Chrome, Edge on some devices)

{% hint style="warning" %}
Always provide fallback behavior for browsers that don't support the Network Information API. The component will emit an `unsupported` event when the API is unavailable.
{% endhint %}

## Usage

### Basic Network Detection

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <h2>Network Status</h2>
    <div id="network-display">Detecting network...</div>
</div>

<script>
    document.addEventListener('pwa--network-information:change', (event) => {
        const connection = event.detail.connection;

        document.getElementById('network-display').innerHTML = `
            <p>Connection Type: ${connection.effectiveType || 'unknown'}</p>
            <p>Downlink: ${connection.downlink || 'N/A'} Mbps</p>
            <p>RTT: ${connection.rtt || 'N/A'} ms</p>
            <p>Data Saver: ${connection.saveData ? 'Enabled' : 'Disabled'}</p>
        `;
    });

    document.addEventListener('pwa--network-information:unsupported', () => {
        document.getElementById('network-display').innerHTML =
            '<p>Network Information API not supported</p>';
    });
</script>
```
{% endcode %}

### Adaptive Image Loading

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <h2>Photo Gallery</h2>
    <div id="gallery" class="image-grid"></div>
</div>

<script>
    const images = [
        { id: 1, title: 'Mountain View' },
        { id: 2, title: 'Ocean Sunset' },
        { id: 3, title: 'City Lights' },
        { id: 4, title: 'Forest Path' }
    ];

    let currentQuality = 'medium';

    document.addEventListener('pwa--network-information:change', (event) => {
        const connection = event.detail.connection;

        // Determine quality based on connection
        if (connection.saveData) {
            currentQuality = 'low';
        } else if (connection.effectiveType === '4g' && connection.downlink > 5) {
            currentQuality = 'high';
        } else if (connection.effectiveType === '3g' || connection.effectiveType === '4g') {
            currentQuality = 'medium';
        } else {
            currentQuality = 'low';
        }

        loadGallery();
    });

    function loadGallery() {
        const gallery = document.getElementById('gallery');

        const imageHtml = images.map(img => `
            <div class="image-card">
                <img src="/images/${img.id}-${currentQuality}.jpg"
                     alt="${img.title}"
                     loading="lazy">
                <p>${img.title}</p>
            </div>
        `).join('');

        gallery.innerHTML = imageHtml;

        // Show quality indicator
        console.log(`Gallery loaded with ${currentQuality} quality images`);
    }
</script>

<style>
    .image-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
        gap: 20px;
    }

    .image-card img {
        width: 100%;
        border-radius: 8px;
    }
</style>
```
{% endcode %}

### Video Quality Adjustment

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <h2>Adaptive Video Player</h2>

    <video id="adaptive-video" controls>
        <source id="video-source" src="" type="video/mp4">
    </video>

    <div id="quality-info"></div>
</div>

<script>
    const videoQualities = {
        high: '/videos/video-1080p.mp4',
        medium: '/videos/video-720p.mp4',
        low: '/videos/video-480p.mp4'
    };

    const videoElement = document.getElementById('adaptive-video');
    const videoSource = document.getElementById('video-source');
    const qualityInfo = document.getElementById('quality-info');

    document.addEventListener('pwa--network-information:change', (event) => {
        const connection = event.detail.connection;

        let selectedQuality = 'medium';

        // Determine optimal video quality
        if (connection.saveData) {
            selectedQuality = 'low';
        } else {
            const effectiveType = connection.effectiveType;

            if (effectiveType === '4g' && connection.downlink > 10) {
                selectedQuality = 'high';
            } else if (effectiveType === '4g' || effectiveType === '3g') {
                selectedQuality = 'medium';
            } else {
                selectedQuality = 'low';
            }
        }

        // Update video source
        const currentTime = videoElement.currentTime;
        const wasPlaying = !videoElement.paused;

        videoSource.src = videoQualities[selectedQuality];
        videoElement.load();
        videoElement.currentTime = currentTime;

        if (wasPlaying) {
            videoElement.play();
        }

        // Update UI
        qualityInfo.innerHTML = `
            <p>Video Quality: <strong>${selectedQuality}</strong></p>
            <p>Connection: ${connection.effectiveType} (${connection.downlink} Mbps)</p>
            ${connection.saveData ? '<p class="warning">⚠️ Data Saver Mode Active</p>' : ''}
        `;
    });
</script>

<style>
    #adaptive-video {
        width: 100%;
        max-width: 800px;
        border-radius: 8px;
    }

    #quality-info {
        margin-top: 15px;
        padding: 15px;
        background: #f3f4f6;
        border-radius: 8px;
    }

    .warning {
        color: #f59e0b;
        font-weight: bold;
    }
</style>
```
{% endcode %}

### Conditional Preloading

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <h2>News Articles</h2>

    <div id="articles-list">
        <article class="article-preview" data-article-id="1">
            <h3>Breaking News Story</h3>
            <p>Click to read more...</p>
        </article>
        <article class="article-preview" data-article-id="2">
            <h3>Tech Innovation</h3>
            <p>Click to read more...</p>
        </article>
        <article class="article-preview" data-article-id="3">
            <h3>Science Discovery</h3>
            <p>Click to read more...</p>
        </article>
    </div>

    <div id="preload-status"></div>
</div>

<script>
    let shouldPreload = false;

    document.addEventListener('pwa--network-information:change', (event) => {
        const connection = event.detail.connection;

        // Only preload on fast connections without data saver
        shouldPreload = !connection.saveData &&
                       (connection.effectiveType === '4g') &&
                       (connection.downlink > 5);

        const statusEl = document.getElementById('preload-status');

        if (shouldPreload) {
            statusEl.innerHTML = '✓ Preloading enabled (fast connection)';
            statusEl.style.color = '#10b981';
            preloadArticles();
        } else {
            statusEl.innerHTML = '○ Preloading disabled (slow connection or data saver)';
            statusEl.style.color = '#6b7280';
        }
    });

    function preloadArticles() {
        if (!shouldPreload) return;

        const articles = document.querySelectorAll('.article-preview');

        articles.forEach(article => {
            const articleId = article.dataset.articleId;

            // Preload article content
            fetch(`/api/articles/${articleId}`)
                .then(response => response.json())
                .then(data => {
                    // Cache article data
                    sessionStorage.setItem(`article-${articleId}`, JSON.stringify(data));
                    console.log(`Preloaded article ${articleId}`);
                })
                .catch(err => console.error('Preload failed:', err));
        });
    }

    // When user clicks an article
    document.getElementById('articles-list').addEventListener('click', (e) => {
        const article = e.target.closest('.article-preview');
        if (!article) return;

        const articleId = article.dataset.articleId;
        const cached = sessionStorage.getItem(`article-${articleId}`);

        if (cached) {
            console.log('Loaded from cache!');
            displayArticle(JSON.parse(cached));
        } else {
            console.log('Fetching article...');
            fetch(`/api/articles/${articleId}`)
                .then(response => response.json())
                .then(data => displayArticle(data));
        }
    });

    function displayArticle(data) {
        alert(`Article loaded: ${data.title}`);
    }
</script>

<style>
    .article-preview {
        padding: 20px;
        margin: 10px 0;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
        cursor: pointer;
        transition: all 0.2s;
    }

    .article-preview:hover {
        background: #f9fafb;
        border-color: #3b82f6;
    }

    #preload-status {
        margin-top: 20px;
        padding: 10px;
        font-weight: 500;
    }
</style>
```
{% endcode %}

### Data Saver Mode Detection

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <h2>Content Dashboard</h2>

    <div id="content-area">
        <div id="data-saver-notice" style="display: none;">
            <p>⚠️ Data Saver mode detected. Showing optimized content.</p>
        </div>

        <div id="main-content"></div>
    </div>
</div>

<script>
    document.addEventListener('pwa--network-information:change', (event) => {
        const connection = event.detail.connection;
        const dataSaverNotice = document.getElementById('data-saver-notice');
        const mainContent = document.getElementById('main-content');

        if (connection.saveData) {
            // Show data saver notice
            dataSaverNotice.style.display = 'block';

            // Load lightweight content
            mainContent.innerHTML = `
                <div class="text-content">
                    <h3>News Summary</h3>
                    <ul>
                        <li>Story 1 - <a href="/article/1">Read more</a></li>
                        <li>Story 2 - <a href="/article/2">Read more</a></li>
                        <li>Story 3 - <a href="/article/3">Read more</a></li>
                    </ul>
                </div>
            `;
        } else {
            // Hide data saver notice
            dataSaverNotice.style.display = 'none';

            // Load full content with images
            mainContent.innerHTML = `
                <div class="rich-content">
                    <div class="article">
                        <img src="/images/story1.jpg" alt="Story 1">
                        <h3>Full Story 1</h3>
                        <p>Complete article content with images...</p>
                    </div>
                    <div class="article">
                        <img src="/images/story2.jpg" alt="Story 2">
                        <h3>Full Story 2</h3>
                        <p>Complete article content with images...</p>
                    </div>
                </div>
            `;
        }
    });
</script>

<style>
    #data-saver-notice {
        background: #fef3c7;
        color: #92400e;
        padding: 15px;
        border-radius: 8px;
        margin-bottom: 20px;
    }

    .article {
        margin: 20px 0;
    }

    .article img {
        width: 100%;
        max-width: 600px;
        border-radius: 8px;
    }
</style>
```
{% endcode %}

### Network-Aware Feature Toggling

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <h2>Feature Dashboard</h2>

    <div id="features">
        <div class="feature" id="feature-analytics">
            <h3>📊 Analytics</h3>
            <p class="status">Checking connection...</p>
        </div>

        <div class="feature" id="feature-autosave">
            <h3>💾 Auto-save</h3>
            <p class="status">Checking connection...</p>
        </div>

        <div class="feature" id="feature-realtime">
            <h3>🔄 Real-time Updates</h3>
            <p class="status">Checking connection...</p>
        </div>

        <div class="feature" id="feature-hd-images">
            <h3>🖼️ HD Images</h3>
            <p class="status">Checking connection...</p>
        </div>
    </div>
</div>

<script>
    document.addEventListener('pwa--network-information:change', (event) => {
        const connection = event.detail.connection;

        const features = {
            analytics: false,
            autosave: false,
            realtime: false,
            hdImages: false
        };

        // Enable features based on connection quality
        const effectiveType = connection.effectiveType;
        const isFastConnection = effectiveType === '4g' && connection.downlink > 5;
        const isGoodConnection = effectiveType === '4g' || effectiveType === '3g';
        const isDataSaver = connection.saveData;

        if (isGoodConnection && !isDataSaver) {
            features.analytics = true;
            features.autosave = true;
        }

        if (isFastConnection && !isDataSaver) {
            features.realtime = true;
            features.hdImages = true;
        }

        // Update UI
        updateFeatureStatus('analytics', features.analytics);
        updateFeatureStatus('autosave', features.autosave);
        updateFeatureStatus('realtime', features.realtime);
        updateFeatureStatus('hd-images', features.hdImages);

        console.log('Features configuration:', features);
    });

    function updateFeatureStatus(featureId, enabled) {
        const featureEl = document.getElementById(`feature-${featureId}`);
        const statusEl = featureEl.querySelector('.status');

        if (enabled) {
            statusEl.textContent = '✓ Enabled';
            statusEl.style.color = '#10b981';
            featureEl.style.opacity = '1';
        } else {
            statusEl.textContent = '○ Disabled (limited connection)';
            statusEl.style.color = '#6b7280';
            featureEl.style.opacity = '0.6';
        }
    }
</script>

<style>
    #features {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 20px;
        margin-top: 20px;
    }

    .feature {
        padding: 20px;
        border: 2px solid #e5e7eb;
        border-radius: 8px;
        transition: all 0.3s;
    }

    .feature h3 {
        margin: 0 0 10px 0;
    }

    .feature .status {
        margin: 5px 0 0 0;
        font-weight: 500;
    }
</style>
```
{% endcode %}

### Connection Type Monitoring

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <h2>Network Monitor</h2>

    <div id="connection-details">
        <div class="detail-card">
            <h3>Connection Type</h3>
            <p id="connection-type" class="value">-</p>
        </div>

        <div class="detail-card">
            <h3>Effective Type</h3>
            <p id="effective-type" class="value">-</p>
        </div>

        <div class="detail-card">
            <h3>Download Speed</h3>
            <p id="downlink" class="value">-</p>
        </div>

        <div class="detail-card">
            <h3>Latency (RTT)</h3>
            <p id="rtt" class="value">-</p>
        </div>

        <div class="detail-card">
            <h3>Data Saver</h3>
            <p id="save-data" class="value">-</p>
        </div>

        <div class="detail-card full-width">
            <h3>Connection Quality</h3>
            <div id="quality-bar" class="quality-bar"></div>
        </div>
    </div>

    <div id="history">
        <h3>Connection History</h3>
        <ul id="history-list"></ul>
    </div>
</div>

<script>
    const historyList = document.getElementById('history-list');
    const maxHistoryItems = 10;

    document.addEventListener('pwa--network-information:change', (event) => {
        const connection = event.detail.connection;

        // Update connection details
        document.getElementById('connection-type').textContent =
            connection.type || 'unknown';

        document.getElementById('effective-type').textContent =
            connection.effectiveType || 'unknown';

        document.getElementById('downlink').textContent =
            connection.downlink ? `${connection.downlink} Mbps` : 'N/A';

        document.getElementById('rtt').textContent =
            connection.rtt ? `${connection.rtt} ms` : 'N/A';

        document.getElementById('save-data').textContent =
            connection.saveData ? 'Enabled' : 'Disabled';

        // Update quality bar
        updateQualityBar(connection);

        // Add to history
        addToHistory(connection);
    });

    function updateQualityBar(connection) {
        const qualityBar = document.getElementById('quality-bar');
        let quality = 0;
        let color = '#ef4444';
        let label = 'Poor';

        if (connection.effectiveType === '4g') {
            quality = connection.downlink > 10 ? 100 : 75;
            color = connection.downlink > 10 ? '#10b981' : '#3b82f6';
            label = connection.downlink > 10 ? 'Excellent' : 'Good';
        } else if (connection.effectiveType === '3g') {
            quality = 50;
            color = '#f59e0b';
            label = 'Fair';
        } else {
            quality = 25;
            color = '#ef4444';
            label = 'Poor';
        }

        qualityBar.innerHTML = `
            <div class="quality-fill" style="width: ${quality}%; background: ${color};"></div>
            <span class="quality-label">${label} (${quality}%)</span>
        `;
    }

    function addToHistory(connection) {
        const timestamp = new Date().toLocaleTimeString();

        const historyItem = document.createElement('li');
        historyItem.innerHTML = `
            <strong>${timestamp}</strong>: ${connection.effectiveType}
            (${connection.downlink || 'N/A'} Mbps, ${connection.rtt || 'N/A'} ms)
            ${connection.saveData ? '- Data Saver' : ''}
        `;

        historyList.insertBefore(historyItem, historyList.firstChild);

        // Keep only recent items
        while (historyList.children.length > maxHistoryItems) {
            historyList.removeChild(historyList.lastChild);
        }
    }
</script>

<style>
    #connection-details {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
        margin-bottom: 30px;
    }

    .detail-card {
        padding: 20px;
        background: #f9fafb;
        border-radius: 8px;
        border: 1px solid #e5e7eb;
    }

    .detail-card.full-width {
        grid-column: 1 / -1;
    }

    .detail-card h3 {
        margin: 0 0 10px 0;
        font-size: 14px;
        color: #6b7280;
        text-transform: uppercase;
    }

    .value {
        margin: 0;
        font-size: 24px;
        font-weight: bold;
        color: #1f2937;
    }

    .quality-bar {
        position: relative;
        height: 40px;
        background: #e5e7eb;
        border-radius: 20px;
        overflow: hidden;
    }

    .quality-fill {
        height: 100%;
        transition: width 0.3s ease;
    }

    .quality-label {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-weight: bold;
        color: #1f2937;
    }

    #history {
        margin-top: 30px;
    }

    #history h3 {
        margin-bottom: 15px;
    }

    #history-list {
        list-style: none;
        padding: 0;
    }

    #history-list li {
        padding: 10px;
        margin: 5px 0;
        background: #f9fafb;
        border-radius: 6px;
        font-size: 14px;
    }
</style>
```
{% endcode %}

## Parameters

None

## Actions

None - This component automatically monitors network changes and emits events.

## Targets

None

## Events

### `pwa--network-information:change`

Dispatched when the component is initialized and whenever the browser detects a change in network conditions (e.g., switching from Wi-Fi to cellular, connection quality changes).

**Payload**: `{connection}`

The `connection` object contains:

* `effectiveType` (string): The estimated effective connection type
  * Values: `'slow-2g'`, `'2g'`, `'3g'`, `'4g'`
  * Represents the effective network quality based on observed RTT and downlink values
* `downlink` (number): Estimated downlink speed in megabits per second (Mbps)
  * Example: `10.5` means 10.5 Mbps
* `rtt` (number): Estimated round-trip time in milliseconds
  * Lower values indicate better latency
  * Example: `50` means 50ms latency
* `saveData` (boolean): Whether the user has enabled data saver mode in their browser
  * `true`: User wants to minimize data usage
  * `false`: Normal data usage
* `type` (string, optional): The connection medium (when available)
  * Values: `'bluetooth'`, `'cellular'`, `'ethernet'`, `'wifi'`, `'wimax'`, `'none'`, `'other'`, `'unknown'`
  * Not widely supported on all devices/browsers

Example:
```javascript
document.addEventListener('pwa--network-information:change', (event) => {
    const { connection } = event.detail;

    console.log('Effective Type:', connection.effectiveType);
    console.log('Download Speed:', connection.downlink, 'Mbps');
    console.log('Latency:', connection.rtt, 'ms');
    console.log('Data Saver:', connection.saveData);

    if (connection.type) {
        console.log('Connection Type:', connection.type);
    }

    // Adapt application behavior
    if (connection.effectiveType === '4g' && !connection.saveData) {
        enableHighQualityContent();
    } else if (connection.effectiveType === 'slow-2g' || connection.saveData) {
        enableDataSaverMode();
    }
});
```

### `pwa--network-information:unsupported`

Dispatched when the Network Information API is not supported by the browser.

**No payload**

Example:
```javascript
document.addEventListener('pwa--network-information:unsupported', () => {
    console.log('Network Information API not supported');
    // Use default behavior or feature detection
    enableDefaultContent();
});
```

## Best Practices

1. **Always provide fallbacks**: Not all browsers support this API - design for graceful degradation
2. **Respect Data Saver mode**: When `saveData` is `true`, minimize data usage
3. **Don't rely on exact values**: Connection metrics are estimates, not precise measurements
4. **Combine with actual performance**: Use alongside Network Timing API for real measurements
5. **Test on real devices**: Emulated network conditions may not reflect real-world behavior
6. **Update gradually**: Don't make sudden quality changes that disrupt user experience
7. **Cache aggressively**: Use the information to inform caching strategies
8. **Monitor changes**: Connection can change frequently - handle multiple events
9. **Inform users**: Let users know when you're adapting content based on their connection
10. **Allow overrides**: Provide manual quality controls for user preference

## Connection Type Reference

### Effective Types

| Type | Minimum RTT | Maximum Downlink | Use Case |
|------|-------------|------------------|----------|
| `slow-2g` | 2000ms | 0.05 Mbps | Text-only, extreme optimization |
| `2g` | 1400ms | 0.07 Mbps | Lightweight content, minimal images |
| `3g` | 270ms | 0.7 Mbps | Standard content, compressed media |
| `4g` | 0ms | ∞ | Full quality, preloading enabled |

### Adapting Content by Connection

```javascript
function getContentStrategy(connection) {
    if (connection.saveData) {
        return {
            imageQuality: 'low',
            videoEnabled: false,
            preload: false,
            analytics: 'minimal'
        };
    }

    switch (connection.effectiveType) {
        case 'slow-2g':
        case '2g':
            return {
                imageQuality: 'low',
                videoEnabled: false,
                preload: false,
                analytics: 'minimal'
            };

        case '3g':
            return {
                imageQuality: 'medium',
                videoEnabled: true,
                preload: false,
                analytics: 'standard'
            };

        case '4g':
            const isFast = connection.downlink > 10;
            return {
                imageQuality: isFast ? 'high' : 'medium',
                videoEnabled: true,
                preload: isFast,
                analytics: 'full'
            };

        default:
            return {
                imageQuality: 'medium',
                videoEnabled: true,
                preload: false,
                analytics: 'standard'
            };
    }
}
```

## Complete Example: Adaptive Media Gallery

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/network-information') }}>
    <div class="media-gallery">
        <header class="gallery-header">
            <h1>Media Gallery</h1>

            <div class="network-indicator" id="network-indicator">
                <span class="indicator-dot"></span>
                <span class="indicator-text">Detecting...</span>
            </div>
        </header>

        <div class="controls">
            <button id="toggle-auto" class="btn-primary">Auto Quality: ON</button>
            <select id="manual-quality" disabled>
                <option value="low">Low Quality</option>
                <option value="medium">Medium Quality</option>
                <option value="high">High Quality</option>
            </select>
        </div>

        <div class="gallery-grid" id="gallery"></div>

        <div class="stats" id="stats"></div>
    </div>
</div>

<script>
    const mediaItems = [
        { id: 1, title: 'Sunset Beach', type: 'image' },
        { id: 2, title: 'Mountain Peak', type: 'image' },
        { id: 3, title: 'City Timelapse', type: 'video' },
        { id: 4, title: 'Forest Stream', type: 'image' },
        { id: 5, title: 'Desert Landscape', type: 'image' },
        { id: 6, title: 'Ocean Waves', type: 'video' }
    ];

    let autoQuality = true;
    let currentQuality = 'medium';
    let connectionInfo = null;

    // UI Elements
    const gallery = document.getElementById('gallery');
    const networkIndicator = document.getElementById('network-indicator');
    const statsEl = document.getElementById('stats');
    const toggleAutoBtn = document.getElementById('toggle-auto');
    const manualQualitySelect = document.getElementById('manual-quality');

    // Toggle auto quality
    toggleAutoBtn.addEventListener('click', () => {
        autoQuality = !autoQuality;
        toggleAutoBtn.textContent = `Auto Quality: ${autoQuality ? 'ON' : 'OFF'}`;
        manualQualitySelect.disabled = autoQuality;

        if (autoQuality && connectionInfo) {
            updateQualityFromConnection(connectionInfo);
        }
    });

    // Manual quality selection
    manualQualitySelect.addEventListener('change', () => {
        if (!autoQuality) {
            currentQuality = manualQualitySelect.value;
            loadGallery();
        }
    });

    // Network change handler
    document.addEventListener('pwa--network-information:change', (event) => {
        connectionInfo = event.detail.connection;

        updateNetworkIndicator(connectionInfo);

        if (autoQuality) {
            updateQualityFromConnection(connectionInfo);
        }

        updateStats(connectionInfo);
    });

    function updateQualityFromConnection(connection) {
        let newQuality = 'medium';

        if (connection.saveData) {
            newQuality = 'low';
        } else {
            const effectiveType = connection.effectiveType;
            const downlink = connection.downlink || 1;

            if (effectiveType === '4g' && downlink > 10) {
                newQuality = 'high';
            } else if (effectiveType === '4g' || effectiveType === '3g') {
                newQuality = 'medium';
            } else {
                newQuality = 'low';
            }
        }

        if (newQuality !== currentQuality) {
            currentQuality = newQuality;
            manualQualitySelect.value = currentQuality;
            loadGallery();
        }
    }

    function updateNetworkIndicator(connection) {
        const dot = networkIndicator.querySelector('.indicator-dot');
        const text = networkIndicator.querySelector('.indicator-text');

        let color = '#6b7280';
        let status = 'Unknown';

        if (connection.effectiveType === '4g' && connection.downlink > 10) {
            color = '#10b981';
            status = 'Excellent';
        } else if (connection.effectiveType === '4g' || connection.effectiveType === '3g') {
            color = '#3b82f6';
            status = 'Good';
        } else if (connection.effectiveType === '2g') {
            color = '#f59e0b';
            status = 'Slow';
        } else {
            color = '#ef4444';
            status = 'Very Slow';
        }

        if (connection.saveData) {
            status += ' (Data Saver)';
        }

        dot.style.background = color;
        text.textContent = status;
    }

    function loadGallery() {
        const items = mediaItems.map(item => {
            if (item.type === 'image') {
                return `
                    <div class="gallery-item">
                        <img src="/media/${item.id}-${currentQuality}.jpg"
                             alt="${item.title}"
                             loading="lazy">
                        <div class="item-title">${item.title}</div>
                        <div class="quality-badge">${currentQuality}</div>
                    </div>
                `;
            } else {
                const showVideo = currentQuality !== 'low';
                return `
                    <div class="gallery-item">
                        ${showVideo ? `
                            <video src="/media/${item.id}-${currentQuality}.mp4"
                                   controls
                                   preload="metadata">
                            </video>
                        ` : `
                            <div class="video-placeholder">
                                <p>📹 Video (disabled on slow connection)</p>
                                <button onclick="location.href='/media/${item.id}'">
                                    View anyway
                                </button>
                            </div>
                        `}
                        <div class="item-title">${item.title}</div>
                        <div class="quality-badge">${currentQuality}</div>
                    </div>
                `;
            }
        }).join('');

        gallery.innerHTML = items;
    }

    function updateStats(connection) {
        const estimatedPageSize = {
            low: 2.5,
            medium: 8.0,
            high: 25.0
        }[currentQuality];

        const loadTime = connection.downlink > 0
            ? (estimatedPageSize / connection.downlink).toFixed(1)
            : '?';

        statsEl.innerHTML = `
            <h3>Current Settings</h3>
            <div class="stats-grid">
                <div>
                    <strong>Quality:</strong> ${currentQuality}
                </div>
                <div>
                    <strong>Connection:</strong> ${connection.effectiveType}
                </div>
                <div>
                    <strong>Speed:</strong> ${connection.downlink || 'N/A'} Mbps
                </div>
                <div>
                    <strong>Latency:</strong> ${connection.rtt || 'N/A'} ms
                </div>
                <div>
                    <strong>Est. Page Size:</strong> ${estimatedPageSize} MB
                </div>
                <div>
                    <strong>Est. Load Time:</strong> ${loadTime}s
                </div>
            </div>
        `;
    }

    document.addEventListener('pwa--network-information:unsupported', () => {
        networkIndicator.querySelector('.indicator-text').textContent =
            'Network API not supported';
        currentQuality = 'medium';
        loadGallery();
    });
</script>

<style>
    .media-gallery {
        max-width: 1200px;
        margin: 0 auto;
        padding: 20px;
    }

    .gallery-header {
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 30px;
    }

    .network-indicator {
        display: flex;
        align-items: center;
        gap: 10px;
        padding: 10px 20px;
        background: #f9fafb;
        border-radius: 20px;
    }

    .indicator-dot {
        width: 12px;
        height: 12px;
        border-radius: 50%;
        background: #6b7280;
        animation: pulse 2s infinite;
    }

    @keyframes pulse {
        0%, 100% { opacity: 1; }
        50% { opacity: 0.5; }
    }

    .controls {
        display: flex;
        gap: 15px;
        margin-bottom: 30px;
    }

    .btn-primary {
        padding: 10px 20px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        font-weight: 500;
    }

    .btn-primary:hover {
        background: #2563eb;
    }

    select {
        padding: 10px;
        border: 1px solid #e5e7eb;
        border-radius: 6px;
        font-size: 14px;
    }

    .gallery-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
        gap: 20px;
        margin-bottom: 40px;
    }

    .gallery-item {
        position: relative;
        border-radius: 12px;
        overflow: hidden;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        transition: transform 0.2s;
    }

    .gallery-item:hover {
        transform: translateY(-4px);
    }

    .gallery-item img,
    .gallery-item video {
        width: 100%;
        display: block;
    }

    .video-placeholder {
        aspect-ratio: 16/9;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        background: #f3f4f6;
        padding: 40px;
        text-align: center;
    }

    .video-placeholder button {
        margin-top: 10px;
        padding: 8px 16px;
        background: #3b82f6;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
    }

    .item-title {
        padding: 15px;
        background: white;
        font-weight: 500;
    }

    .quality-badge {
        position: absolute;
        top: 10px;
        right: 10px;
        padding: 5px 10px;
        background: rgba(0, 0, 0, 0.7);
        color: white;
        border-radius: 4px;
        font-size: 12px;
        text-transform: uppercase;
    }

    .stats {
        padding: 20px;
        background: #f9fafb;
        border-radius: 12px;
    }

    .stats h3 {
        margin: 0 0 15px 0;
    }

    .stats-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
    }

    .stats-grid div {
        padding: 10px;
        background: white;
        border-radius: 6px;
    }
</style>
```
{% endcode %}

## Troubleshooting

### API not supported

**Browser**: Safari, older browsers

**Solution**: Always listen for the `unsupported` event and provide fallback behavior:
```javascript
document.addEventListener('pwa--network-information:unsupported', () => {
    // Use default quality settings
    useDefaultQuality();
});
```

### Inaccurate estimates

**Issue**: Connection metrics don't match actual performance

**Solutions**:
- Use metrics as estimates, not absolute values
- Combine with actual performance measurements (Resource Timing API)
- Track real download times and adjust accordingly
- Test on real devices and connections

### Connection type unavailable

**Issue**: `connection.type` returns `undefined`

**Cause**: Many devices don't expose the connection medium

**Solution**: Rely on `effectiveType`, `downlink`, and `rtt` instead

### Frequent quality changes

**Issue**: Content quality switches too often, disrupting user experience

**Solutions**:
- Implement debouncing to avoid rapid changes
- Set thresholds before switching quality
- Only change quality at natural breakpoints (page load, user action)

```javascript
let qualityChangeTimeout;
const QUALITY_CHANGE_DELAY = 5000; // 5 seconds

document.addEventListener('pwa--network-information:change', (event) => {
    clearTimeout(qualityChangeTimeout);

    qualityChangeTimeout = setTimeout(() => {
        updateQuality(event.detail.connection);
    }, QUALITY_CHANGE_DELAY);
});
```

### Data Saver not detected

**Issue**: `saveData` always returns `false`

**Cause**: User hasn't enabled Data Saver in browser settings

**Solution**: This is expected behavior - only optimize when explicitly requested by the user

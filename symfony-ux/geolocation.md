# Geolocation

The Geolocation component provides an interface to the Geolocation API, enabling your Progressive Web App to access the user's geographic location. This allows you to create location-aware applications that can provide personalized content, navigation, and services based on the user's position.

This component is particularly useful for:

* Location-based services and recommendations
* Mapping and navigation applications
* Delivery and transportation tracking
* Fitness and activity tracking apps
* Store locators and proximity searches
* Emergency services and safety features
* Weather and local information services
* Social check-ins and location sharing

## Browser Support

The Geolocation API is widely supported across all modern browsers on both desktop and mobile devices. However, for security and privacy reasons, browsers require:

1. **HTTPS**: Geolocation only works on secure origins (https:// or localhost)
2. **User Permission**: Users must explicitly grant permission to access their location
3. **User Gesture**: Initial requests should be triggered by user interaction

{% hint style="warning" %}
Always handle permission denials gracefully and provide clear explanations of why your app needs location access.
{% endhint %}

## Usage

### Basic Location Request

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <h2>Find Your Location</h2>

    <button {{ stimulus_action('@pwa/geolocation', 'locate', 'click') }}>
        Get My Location
    </button>

    <div id="location-display"></div>
</div>

<script>
    document.addEventListener('pwa--geolocation:position', (event) => {
        const { coords, timestamp } = event.detail;

        document.getElementById('location-display').innerHTML = `
            <p>Latitude: ${coords.latitude}</p>
            <p>Longitude: ${coords.longitude}</p>
            <p>Accuracy: ${coords.accuracy} meters</p>
        `;
    });

    document.addEventListener('pwa--geolocation:error', (event) => {
        const error = event.detail;
        alert(`Error: ${error.message}`);
    });
</script>
```
{% endcode %}

### Continuous Location Tracking

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <h2>Live Location Tracking</h2>

    <button {{ stimulus_action('@pwa/geolocation', 'watch', 'click', {
        enableHighAccuracy: true,
        maximumAge: 0,
        timeout: 5000
    }) }}>
        Start Tracking
    </button>

    <button {{ stimulus_action('@pwa/geolocation', 'clearWatch', 'click') }}>
        Stop Tracking
    </button>

    <div id="tracking-display">
        <p>Status: <span id="tracking-status">Not tracking</span></p>
        <p>Position: <span id="current-position">-</span></p>
        <p>Updates: <span id="update-count">0</span></p>
    </div>
</div>

<script>
    let updateCount = 0;

    document.addEventListener('pwa--geolocation:position', (event) => {
        const { coords } = event.detail;
        updateCount++;

        document.getElementById('tracking-status').textContent = 'Active';
        document.getElementById('current-position').textContent =
            `${coords.latitude.toFixed(6)}, ${coords.longitude.toFixed(6)}`;
        document.getElementById('update-count').textContent = updateCount;
    });

    document.addEventListener('pwa--geolocation:watch:cleared', () => {
        document.getElementById('tracking-status').textContent = 'Stopped';
    });
</script>
```
{% endcode %}

### Distance Calculator

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <h2>Distance to Destination</h2>

    <button {{ stimulus_action('@pwa/geolocation', 'locate', 'click', {
        enableHighAccuracy: true
    }) }}>
        Calculate Distance
    </button>

    <div id="distance-result"></div>
</div>

<script>
    // Destination coordinates (example: Eiffel Tower)
    const destination = {
        lat: 48.8584,
        lng: 2.2945,
        name: 'Eiffel Tower'
    };

    document.addEventListener('pwa--geolocation:position', (event) => {
        const { coords } = event.detail;
        const distance = calculateDistance(
            coords.latitude,
            coords.longitude,
            destination.lat,
            destination.lng
        );

        document.getElementById('distance-result').innerHTML = `
            <p>Your location: ${coords.latitude.toFixed(4)}, ${coords.longitude.toFixed(4)}</p>
            <p>Distance to ${destination.name}: <strong>${distance.toFixed(2)} km</strong></p>
        `;
    });

    // Haversine formula for calculating distance between two coordinates
    function calculateDistance(lat1, lon1, lat2, lon2) {
        const R = 6371; // Earth's radius in kilometers
        const dLat = toRadians(lat2 - lat1);
        const dLon = toRadians(lon2 - lon1);

        const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
                  Math.cos(toRadians(lat1)) * Math.cos(toRadians(lat2)) *
                  Math.sin(dLon / 2) * Math.sin(dLon / 2);

        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
        return R * c;
    }

    function toRadians(degrees) {
        return degrees * (Math.PI / 180);
    }
</script>
```
{% endcode %}

### Location-Based Content

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <h2>Local Weather</h2>

    <button {{ stimulus_action('@pwa/geolocation', 'locate', 'click') }}>
        Get Local Weather
    </button>

    <div id="weather-display"></div>
</div>

<script>
    document.addEventListener('pwa--geolocation:position', async (event) => {
        const { coords } = event.detail;

        // Show loading state
        document.getElementById('weather-display').innerHTML =
            '<p>Loading weather data...</p>';

        try {
            // Fetch weather based on coordinates
            const response = await fetch(
                `/api/weather?lat=${coords.latitude}&lng=${coords.longitude}`
            );
            const weather = await response.json();

            document.getElementById('weather-display').innerHTML = `
                <h3>${weather.location}</h3>
                <p>Temperature: ${weather.temperature}°C</p>
                <p>Condition: ${weather.condition}</p>
                <p>Humidity: ${weather.humidity}%</p>
            `;
        } catch (error) {
            document.getElementById('weather-display').innerHTML =
                '<p>Failed to load weather data</p>';
        }
    });
</script>
```
{% endcode %}

### Geofencing

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <h2>Location Alerts</h2>

    <button {{ stimulus_action('@pwa/geolocation', 'watch', 'click', {
        enableHighAccuracy: true,
        maximumAge: 0
    }) }}>
        Enable Location Alerts
    </button>

    <button {{ stimulus_action('@pwa/geolocation', 'clearWatch', 'click') }}>
        Disable Alerts
    </button>

    <div id="alert-display"></div>
</div>

<script>
    // Define geofence areas (example: important locations)
    const geofences = [
        {
            name: 'Home',
            lat: 48.8566,
            lng: 2.3522,
            radius: 100 // meters
        },
        {
            name: 'Office',
            lat: 48.8738,
            lng: 2.2950,
            radius: 50
        }
    ];

    let lastNotifiedZone = null;

    document.addEventListener('pwa--geolocation:position', (event) => {
        const { coords } = event.detail;

        geofences.forEach(fence => {
            const distance = calculateDistance(
                coords.latitude,
                coords.longitude,
                fence.lat,
                fence.lng
            ) * 1000; // Convert to meters

            if (distance <= fence.radius && lastNotifiedZone !== fence.name) {
                // Entered geofence
                lastNotifiedZone = fence.name;
                showAlert(`You've arrived at ${fence.name}`);
            } else if (distance > fence.radius && lastNotifiedZone === fence.name) {
                // Left geofence
                lastNotifiedZone = null;
                showAlert(`You've left ${fence.name}`);
            }
        });
    });

    function showAlert(message) {
        const alertDiv = document.getElementById('alert-display');
        alertDiv.innerHTML = `<div class="alert">${message}</div>`;
        setTimeout(() => alertDiv.innerHTML = '', 5000);
    }

    function calculateDistance(lat1, lon1, lat2, lon2) {
        const R = 6371;
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lon2 - lon1) * Math.PI / 180;
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                  Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                  Math.sin(dLon/2) * Math.sin(dLon/2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    }
</script>
```
{% endcode %}

### Fitness Tracker with Route Recording

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <div class="fitness-tracker">
        <h2>Run Tracker</h2>

        <div class="stats">
            <div>Distance: <span id="distance">0.00</span> km</div>
            <div>Duration: <span id="duration">00:00</span></div>
            <div>Speed: <span id="speed">0.0</span> km/h</div>
        </div>

        <button id="start-run" {{ stimulus_action('@pwa/geolocation', 'watch', 'click', {
            enableHighAccuracy: true,
            maximumAge: 0
        }) }}>
            Start Run
        </button>

        <button id="stop-run" {{ stimulus_action('@pwa/geolocation', 'clearWatch', 'click') }} disabled>
            Stop Run
        </button>

        <canvas id="route-map" width="400" height="300"></canvas>
    </div>
</div>

<script>
    let routePoints = [];
    let totalDistance = 0;
    let startTime = null;
    let durationInterval = null;

    document.getElementById('start-run').addEventListener('click', () => {
        routePoints = [];
        totalDistance = 0;
        startTime = Date.now();

        document.getElementById('start-run').disabled = true;
        document.getElementById('stop-run').disabled = false;

        durationInterval = setInterval(updateDuration, 1000);
    });

    document.getElementById('stop-run').addEventListener('click', () => {
        document.getElementById('start-run').disabled = false;
        document.getElementById('stop-run').disabled = true;

        clearInterval(durationInterval);
        saveRoute();
    });

    document.addEventListener('pwa--geolocation:position', (event) => {
        const { coords } = event.detail;
        const point = {
            lat: coords.latitude,
            lng: coords.longitude,
            timestamp: Date.now()
        };

        if (routePoints.length > 0) {
            const lastPoint = routePoints[routePoints.length - 1];
            const distance = calculateDistance(
                lastPoint.lat,
                lastPoint.lng,
                point.lat,
                point.lng
            );
            totalDistance += distance;

            // Update stats
            document.getElementById('distance').textContent = totalDistance.toFixed(2);

            // Calculate speed (km/h)
            if (coords.speed !== null) {
                document.getElementById('speed').textContent =
                    (coords.speed * 3.6).toFixed(1);
            }
        }

        routePoints.push(point);
        drawRoute();
    });

    function updateDuration() {
        const elapsed = Math.floor((Date.now() - startTime) / 1000);
        const minutes = Math.floor(elapsed / 60);
        const seconds = elapsed % 60;
        document.getElementById('duration').textContent =
            `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
    }

    function drawRoute() {
        const canvas = document.getElementById('route-map');
        const ctx = canvas.getContext('2d');

        if (routePoints.length === 0) return;

        // Clear canvas
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // Find bounds
        const lats = routePoints.map(p => p.lat);
        const lngs = routePoints.map(p => p.lng);
        const minLat = Math.min(...lats);
        const maxLat = Math.max(...lats);
        const minLng = Math.min(...lngs);
        const maxLng = Math.max(...lngs);

        // Draw route
        ctx.strokeStyle = '#3b82f6';
        ctx.lineWidth = 3;
        ctx.beginPath();

        routePoints.forEach((point, i) => {
            const x = ((point.lng - minLng) / (maxLng - minLng)) * (canvas.width - 20) + 10;
            const y = canvas.height - (((point.lat - minLat) / (maxLat - minLat)) * (canvas.height - 20) + 10);

            if (i === 0) {
                ctx.moveTo(x, y);
            } else {
                ctx.lineTo(x, y);
            }
        });

        ctx.stroke();

        // Draw start point
        const start = routePoints[0];
        const startX = ((start.lng - minLng) / (maxLng - minLng)) * (canvas.width - 20) + 10;
        const startY = canvas.height - (((start.lat - minLat) / (maxLat - minLat)) * (canvas.height - 20) + 10);
        ctx.fillStyle = '#10b981';
        ctx.beginPath();
        ctx.arc(startX, startY, 5, 0, 2 * Math.PI);
        ctx.fill();
    }

    function calculateDistance(lat1, lon1, lat2, lon2) {
        const R = 6371;
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lon2 - lon1) * Math.PI / 180;
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                  Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                  Math.sin(dLon/2) * Math.sin(dLon/2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    }

    function saveRoute() {
        const route = {
            points: routePoints,
            distance: totalDistance,
            duration: Date.now() - startTime,
            timestamp: new Date().toISOString()
        };

        // Save to localStorage or send to server
        localStorage.setItem('lastRoute', JSON.stringify(route));
        console.log('Route saved:', route);
    }
</script>

<style>
    .stats {
        display: flex;
        gap: 20px;
        margin: 20px 0;
        font-size: 18px;
    }

    .stats span {
        font-weight: bold;
        color: #3b82f6;
    }

    #route-map {
        border: 2px solid #e5e7eb;
        border-radius: 8px;
        margin-top: 20px;
    }
</style>
```
{% endcode %}

### Store Locator

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <h2>Find Nearest Store</h2>

    <button {{ stimulus_action('@pwa/geolocation', 'locate', 'click', {
        enableHighAccuracy: true
    }) }}>
        Find Stores Near Me
    </button>

    <div id="stores-list"></div>
</div>

<script>
    // Store locations
    const stores = [
        { name: 'Downtown Store', lat: 48.8566, lng: 2.3522, address: '123 Main St' },
        { name: 'North Branch', lat: 48.8738, lng: 2.2950, address: '456 North Ave' },
        { name: 'East Location', lat: 48.8606, lng: 2.3376, address: '789 East Blvd' },
        { name: 'West Shop', lat: 48.8584, lng: 2.2945, address: '321 West St' }
    ];

    document.addEventListener('pwa--geolocation:position', (event) => {
        const { coords } = event.detail;

        // Calculate distances and sort
        const storesWithDistance = stores.map(store => ({
            ...store,
            distance: calculateDistance(
                coords.latitude,
                coords.longitude,
                store.lat,
                store.lng
            )
        })).sort((a, b) => a.distance - b.distance);

        // Display stores
        const listHtml = storesWithDistance.map((store, index) => `
            <div class="store-item">
                <h3>${index + 1}. ${store.name}</h3>
                <p>${store.address}</p>
                <p><strong>${store.distance.toFixed(2)} km away</strong></p>
                <a href="https://www.google.com/maps/dir/?api=1&origin=${coords.latitude},${coords.longitude}&destination=${store.lat},${store.lng}" target="_blank">
                    Get Directions →
                </a>
            </div>
        `).join('');

        document.getElementById('stores-list').innerHTML = listHtml;
    });

    function calculateDistance(lat1, lon1, lat2, lon2) {
        const R = 6371;
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lon2 - lon1) * Math.PI / 180;
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                  Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                  Math.sin(dLon/2) * Math.sin(dLon/2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    }
</script>

<style>
    .store-item {
        padding: 15px;
        margin: 10px 0;
        border: 1px solid #e5e7eb;
        border-radius: 8px;
    }

    .store-item h3 {
        margin: 0 0 5px 0;
    }

    .store-item p {
        margin: 5px 0;
    }

    .store-item a {
        color: #3b82f6;
        text-decoration: none;
    }
</style>
```
{% endcode %}

## Parameters

None

## Actions

### `locate`

Retrieves the user's current position once. This is ideal for one-time location requests like "find stores near me" or "show local weather".

**Options:**
* `enableHighAccuracy` (boolean, optional): If `true`, requests the most accurate location available (may use GPS). Default: `false`
* `timeout` (number, optional): Maximum time (in milliseconds) to wait for a position. Default: `Infinity`
* `maximumAge` (number, optional): Maximum age (in milliseconds) of a cached position that is acceptable to return. Default: `0`

```twig
{{ stimulus_action('@pwa/geolocation', 'locate', 'click', {
    enableHighAccuracy: true,
    timeout: 10000,
    maximumAge: 0
}) }}
```

{% hint style="info" %}
`enableHighAccuracy: true` provides better accuracy but uses more battery and may take longer. Use it only when precision is important.
{% endhint %}

### `watch`

Starts continuous position tracking. The component will repeatedly update the position as the user moves. This is ideal for navigation, fitness tracking, or real-time location sharing.

**Options:**
* `enableHighAccuracy` (boolean, optional): If `true`, requests high-accuracy position updates. Default: `false`
* `timeout` (number, optional): Maximum time to wait for each position update. Default: `Infinity`
* `maximumAge` (number, optional): Maximum age of cached positions. Default: `0`

```twig
{{ stimulus_action('@pwa/geolocation', 'watch', 'click', {
    enableHighAccuracy: true,
    timeout: 5000,
    maximumAge: 0
}) }}
```

{% hint style="warning" %}
Continuous tracking consumes significant battery power. Always provide a way to stop tracking when it's no longer needed.
{% endhint %}

### `clearWatch`

Stops continuous position tracking that was started with the `watch` action.

```twig
{{ stimulus_action('@pwa/geolocation', 'clearWatch', 'click') }}
```

## Targets

None

## Events

### `pwa--geolocation:position`

Dispatched when a position update is received (from either `locate` or `watch` actions).

**Payload**: `{coords, timestamp}`
* `coords.latitude` (number): Latitude in decimal degrees
* `coords.longitude` (number): Longitude in decimal degrees
* `coords.accuracy` (number): Accuracy of the position in meters
* `coords.altitude` (number|null): Altitude in meters above sea level (if available)
* `coords.altitudeAccuracy` (number|null): Accuracy of altitude in meters (if available)
* `coords.heading` (number|null): Direction of travel in degrees (0-360) (if available)
* `coords.speed` (number|null): Speed in meters per second (if available)
* `timestamp` (number): Timestamp when the position was acquired

Example:
```javascript
document.addEventListener('pwa--geolocation:position', (event) => {
    const { coords, timestamp } = event.detail;

    console.log('Position:', coords.latitude, coords.longitude);
    console.log('Accuracy:', coords.accuracy, 'meters');

    if (coords.speed !== null) {
        console.log('Speed:', (coords.speed * 3.6).toFixed(1), 'km/h');
    }

    if (coords.heading !== null) {
        console.log('Heading:', coords.heading, 'degrees');
    }
});
```

### `pwa--geolocation:error`

Dispatched when an error occurs while retrieving the position (permission denied, timeout, position unavailable).

**Payload**: Error object with `code` and `message`
* `code` (number): Error code (1: PERMISSION_DENIED, 2: POSITION_UNAVAILABLE, 3: TIMEOUT)
* `message` (string): Human-readable error description

Example:
```javascript
document.addEventListener('pwa--geolocation:error', (event) => {
    const error = event.detail;

    switch (error.code) {
        case 1: // PERMISSION_DENIED
            alert('Please allow location access to use this feature');
            break;
        case 2: // POSITION_UNAVAILABLE
            alert('Location information is unavailable');
            break;
        case 3: // TIMEOUT
            alert('Location request timed out. Please try again');
            break;
    }

    console.error('Geolocation error:', error.message);
});
```

### `pwa--geolocation:unsupported`

Dispatched when the browser does not support the Geolocation API.

**No payload**

Example:
```javascript
document.addEventListener('pwa--geolocation:unsupported', () => {
    alert('Your browser does not support geolocation');
    // Show alternative content or functionality
});
```

### `pwa--geolocation:watch:cleared`

Dispatched when continuous position tracking is stopped (via `clearWatch` action).

**No payload**

Example:
```javascript
document.addEventListener('pwa--geolocation:watch:cleared', () => {
    console.log('Location tracking stopped');
    // Update UI to reflect tracking stopped
    document.getElementById('tracking-status').textContent = 'Inactive';
});
```

## Best Practices

1. **Request permission contextually**: Ask for location access when the user interacts with a location-based feature
2. **Explain why you need it**: Clearly communicate why your app needs location access
3. **Handle errors gracefully**: Provide fallback options when location access is denied or unavailable
4. **Use appropriate accuracy**: Don't request high accuracy unless you really need it
5. **Stop tracking when done**: Always clear watches to save battery
6. **Cache positions wisely**: Use `maximumAge` to avoid unnecessary GPS usage
7. **Respect privacy**: Don't share or store location data without explicit consent
8. **Provide visual feedback**: Show loading states while waiting for position
9. **Set reasonable timeouts**: Don't let requests hang indefinitely
10. **Test offline behavior**: Ensure your app handles lack of GPS signal gracefully

## Coordinate Calculations

### Distance Between Two Points (Haversine Formula)

```javascript
function calculateDistance(lat1, lon1, lat2, lon2) {
    const R = 6371; // Earth's radius in kilometers
    const dLat = toRadians(lat2 - lat1);
    const dLon = toRadians(lon2 - lon1);

    const a = Math.sin(dLat / 2) * Math.sin(dLat / 2) +
              Math.cos(toRadians(lat1)) * Math.cos(toRadians(lat2)) *
              Math.sin(dLon / 2) * Math.sin(dLon / 2);

    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
    return R * c; // Distance in kilometers
}

function toRadians(degrees) {
    return degrees * (Math.PI / 180);
}
```

### Check if Point is Within Radius

```javascript
function isWithinRadius(centerLat, centerLon, pointLat, pointLon, radiusKm) {
    const distance = calculateDistance(centerLat, centerLon, pointLat, pointLon);
    return distance <= radiusKm;
}
```

### Convert Coordinates to Compass Direction

```javascript
function getCompassDirection(heading) {
    const directions = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
    const index = Math.round(heading / 45) % 8;
    return directions[index];
}
```

## Error Handling

```javascript
function handleLocationError(error) {
    const errorMessages = {
        1: {
            title: 'Permission Denied',
            message: 'Please enable location access in your browser settings.',
            action: 'Show how to enable location access'
        },
        2: {
            title: 'Position Unavailable',
            message: 'Your location could not be determined. Please check your GPS and internet connection.',
            action: 'Retry'
        },
        3: {
            title: 'Timeout',
            message: 'Location request took too long. Please try again.',
            action: 'Retry'
        }
    };

    const errorInfo = errorMessages[error.code] || {
        title: 'Unknown Error',
        message: 'An error occurred while getting your location.',
        action: 'Try Again'
    };

    showErrorDialog(errorInfo);
}
```

## Complete Example: Real-Time Delivery Tracker

{% code lineNumbers="true" %}
```twig
<div {{ stimulus_controller('@pwa/geolocation') }}>
    <div class="delivery-tracker">
        <h1>Delivery Tracking</h1>

        <div class="order-info">
            <h2>Order #12345</h2>
            <p>Status: <span id="delivery-status">Preparing</span></p>
        </div>

        <div class="location-display">
            <h3>Your Location</h3>
            <p id="user-location">Waiting for location...</p>
            <p id="accuracy-info"></p>
        </div>

        <div class="delivery-info">
            <h3>Delivery Details</h3>
            <p>Distance to destination: <span id="distance-to-home">-</span></p>
            <p>Estimated arrival: <span id="eta">-</span></p>
        </div>

        <button id="start-tracking" {{ stimulus_action('@pwa/geolocation', 'watch', 'click', {
            enableHighAccuracy: true,
            maximumAge: 0,
            timeout: 10000
        }) }}>
            Start Tracking
        </button>

        <button id="stop-tracking" {{ stimulus_action('@pwa/geolocation', 'clearWatch', 'click') }} disabled>
            Stop Tracking
        </button>

        <div class="map-container">
            <canvas id="delivery-map" width="600" height="400"></canvas>
        </div>

        <div id="notifications"></div>
    </div>
</div>

<script>
    // Delivery destination (home address)
    const destination = {
        lat: 48.8566,
        lng: 2.3522,
        address: '123 Main Street, Paris'
    };

    let isTracking = false;
    let currentPosition = null;
    let locationHistory = [];
    let lastDistanceToHome = null;

    // UI Elements
    const startBtn = document.getElementById('start-tracking');
    const stopBtn = document.getElementById('stop-tracking');
    const statusEl = document.getElementById('delivery-status');
    const locationEl = document.getElementById('user-location');
    const accuracyEl = document.getElementById('accuracy-info');
    const distanceEl = document.getElementById('distance-to-home');
    const etaEl = document.getElementById('eta');
    const notificationsEl = document.getElementById('notifications');

    startBtn.addEventListener('click', () => {
        isTracking = true;
        startBtn.disabled = true;
        stopBtn.disabled = false;
        statusEl.textContent = 'Out for Delivery';
        showNotification('Tracking started', 'info');
    });

    stopBtn.addEventListener('click', () => {
        isTracking = false;
        startBtn.disabled = false;
        stopBtn.disabled = true;
        statusEl.textContent = 'Tracking Paused';
        showNotification('Tracking stopped', 'info');
    });

    document.addEventListener('pwa--geolocation:position', (event) => {
        if (!isTracking) return;

        const { coords, timestamp } = event.detail;
        currentPosition = coords;

        // Update location display
        locationEl.textContent =
            `${coords.latitude.toFixed(6)}, ${coords.longitude.toFixed(6)}`;
        accuracyEl.textContent =
            `Accuracy: ±${Math.round(coords.accuracy)}m`;

        // Calculate distance to destination
        const distanceToHome = calculateDistance(
            coords.latitude,
            coords.longitude,
            destination.lat,
            destination.lng
        );

        distanceEl.textContent = `${distanceToHome.toFixed(2)} km`;

        // Calculate ETA (assuming average speed from GPS)
        if (coords.speed && coords.speed > 0.5) { // Moving faster than 0.5 m/s
            const speedKmh = coords.speed * 3.6;
            const etaMinutes = (distanceToHome / speedKmh) * 60;
            etaEl.textContent = `${Math.round(etaMinutes)} minutes`;
        } else {
            etaEl.textContent = 'Calculating...';
        }

        // Check proximity alerts
        if (distanceToHome < 0.5 && (!lastDistanceToHome || lastDistanceToHome >= 0.5)) {
            showNotification('Almost home! Less than 500m away', 'success');
        }

        if (distanceToHome < 0.1) {
            showNotification('You have arrived!', 'success');
            statusEl.textContent = 'Delivered';
            stopBtn.click();
        }

        lastDistanceToHome = distanceToHome;

        // Add to history
        locationHistory.push({
            lat: coords.latitude,
            lng: coords.longitude,
            timestamp
        });

        // Keep only last 50 points
        if (locationHistory.length > 50) {
            locationHistory.shift();
        }

        // Draw map
        drawMap();
    });

    document.addEventListener('pwa--geolocation:error', (event) => {
        const error = event.detail;

        let message = 'Location error occurred';
        switch (error.code) {
            case 1:
                message = 'Location access denied. Please enable location permissions.';
                break;
            case 2:
                message = 'Location unavailable. Check your GPS and internet connection.';
                break;
            case 3:
                message = 'Location request timed out. Retrying...';
                break;
        }

        showNotification(message, 'error');
    });

    function drawMap() {
        const canvas = document.getElementById('delivery-map');
        const ctx = canvas.getContext('2d');

        // Clear canvas
        ctx.fillStyle = '#f3f4f6';
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        if (!currentPosition) return;

        // Calculate bounds
        const allPoints = [
            ...locationHistory.map(p => ({ lat: p.lat, lng: p.lng })),
            { lat: destination.lat, lng: destination.lng },
            { lat: currentPosition.latitude, lng: currentPosition.longitude }
        ];

        const lats = allPoints.map(p => p.lat);
        const lngs = allPoints.map(p => p.lng);
        const minLat = Math.min(...lats) - 0.001;
        const maxLat = Math.max(...lats) + 0.001;
        const minLng = Math.min(...lngs) - 0.001;
        const maxLng = Math.max(...lngs) + 0.001;

        const latRange = maxLat - minLat;
        const lngRange = maxLng - minLng;

        function projectPoint(lat, lng) {
            const x = ((lng - minLng) / lngRange) * (canvas.width - 40) + 20;
            const y = canvas.height - (((lat - minLat) / latRange) * (canvas.height - 40) + 20);
            return { x, y };
        }

        // Draw path
        if (locationHistory.length > 1) {
            ctx.strokeStyle = '#3b82f6';
            ctx.lineWidth = 3;
            ctx.beginPath();

            locationHistory.forEach((point, i) => {
                const { x, y } = projectPoint(point.lat, point.lng);
                if (i === 0) {
                    ctx.moveTo(x, y);
                } else {
                    ctx.lineTo(x, y);
                }
            });

            ctx.stroke();
        }

        // Draw destination
        const destPos = projectPoint(destination.lat, destination.lng);
        ctx.fillStyle = '#ef4444';
        ctx.beginPath();
        ctx.arc(destPos.x, destPos.y, 8, 0, 2 * Math.PI);
        ctx.fill();
        ctx.fillStyle = '#000';
        ctx.font = '12px sans-serif';
        ctx.fillText('Home', destPos.x + 12, destPos.y + 4);

        // Draw current position
        const currPos = projectPoint(currentPosition.latitude, currentPosition.longitude);
        ctx.fillStyle = '#10b981';
        ctx.beginPath();
        ctx.arc(currPos.x, currPos.y, 10, 0, 2 * Math.PI);
        ctx.fill();

        // Draw accuracy circle
        const metersPerPixel = (latRange * 111000) / (canvas.height - 40);
        const accuracyRadius = currentPosition.accuracy / metersPerPixel;
        ctx.strokeStyle = 'rgba(16, 185, 129, 0.3)';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(currPos.x, currPos.y, accuracyRadius, 0, 2 * Math.PI);
        ctx.stroke();
    }

    function calculateDistance(lat1, lon1, lat2, lon2) {
        const R = 6371;
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lon2 - lon1) * Math.PI / 180;
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                  Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) *
                  Math.sin(dLon/2) * Math.sin(dLon/2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    }

    function showNotification(message, type) {
        const notification = document.createElement('div');
        notification.className = `notification ${type}`;
        notification.textContent = message;
        notificationsEl.appendChild(notification);

        setTimeout(() => {
            notification.style.opacity = '0';
            setTimeout(() => notification.remove(), 300);
        }, 5000);
    }
</script>

<style>
    .delivery-tracker {
        max-width: 800px;
        margin: 0 auto;
        padding: 20px;
    }

    .order-info, .location-display, .delivery-info {
        background: #f9fafb;
        padding: 15px;
        margin: 15px 0;
        border-radius: 8px;
    }

    .order-info h2 {
        margin: 0 0 10px 0;
    }

    #delivery-status {
        font-weight: bold;
        color: #3b82f6;
    }

    button {
        padding: 12px 24px;
        margin: 10px 10px 10px 0;
        border: none;
        border-radius: 6px;
        font-size: 16px;
        cursor: pointer;
        transition: all 0.2s;
    }

    #start-tracking {
        background: #10b981;
        color: white;
    }

    #start-tracking:hover:not(:disabled) {
        background: #059669;
    }

    #stop-tracking {
        background: #ef4444;
        color: white;
    }

    #stop-tracking:hover:not(:disabled) {
        background: #dc2626;
    }

    button:disabled {
        opacity: 0.5;
        cursor: not-allowed;
    }

    .map-container {
        margin: 20px 0;
        border: 2px solid #e5e7eb;
        border-radius: 8px;
        overflow: hidden;
    }

    #delivery-map {
        display: block;
        width: 100%;
    }

    #notifications {
        position: fixed;
        top: 20px;
        right: 20px;
        z-index: 1000;
    }

    .notification {
        padding: 15px 20px;
        margin-bottom: 10px;
        border-radius: 6px;
        color: white;
        font-weight: 500;
        transition: opacity 0.3s;
        min-width: 250px;
    }

    .notification.info {
        background: #3b82f6;
    }

    .notification.success {
        background: #10b981;
    }

    .notification.error {
        background: #ef4444;
    }
</style>
```
{% endcode %}

## Troubleshooting

### Location access denied

**Solution**: Provide clear instructions on how to enable location permissions in browser settings. Consider showing a help modal with browser-specific instructions.

### Low accuracy

**Common causes**:
1. Indoor location requests (GPS signal blocked)
2. `enableHighAccuracy` set to `false`
3. Device limitations

**Solutions**:
- Use `enableHighAccuracy: true` for better precision
- Request location outdoors when possible
- Display accuracy information to users

### Timeout errors

**Solutions**:
- Increase timeout value: `timeout: 15000` (15 seconds)
- Show loading state to users
- Provide retry option
- Consider using cached positions with `maximumAge`

### Battery drain

**Solutions**:
- Use `watch` only when necessary
- Always call `clearWatch` when done
- Set reasonable `timeout` values
- Use `enableHighAccuracy: false` when precision isn't critical
- Consider updating less frequently

### HTTPS requirement

Geolocation only works on secure contexts (HTTPS or localhost). If testing on a local network, use:
- `localhost` instead of `127.0.0.1`
- Set up HTTPS for local development
- Use tools like ngrok for testing on mobile devices

# Network Information

This controller uses the Network Information API\
to detect changes in the user’s network connection.\
It emits an event whenever the connection type or state changes (for example, switching from Wi-Fi to 4G).

**Usage**

```twig
<div data-controller="network-info">
  <p id="network-status">Loading network info...</p>
</div>

<script type="module">
  const el = document.querySelector('[data-controller="network-info"]');

  el.addEventListener('network-info:change', (e) => {
    const c = e.detail.connection;
    document.getElementById('network-status').textContent =
      `Type: ${c.effectiveType}, Downlink: ${c.downlink} Mbps, RTT: ${c.rtt} ms`;
  });
</script>
```

### Parameters

None

### Actions

`None`

### Targets

None

### Events

change: Emitted on controller connection and every time the browser detects a change in network conditions. Contains the NetworkInformation object with&#x20;

* `effectiveType`: the estimated connection type (`'slow-2g'`, `'2g'`, `'3g'`, `'4g'`, etc.)
* `downlink`: estimated bandwidth in megabits per second
* `rtt`: estimated round-trip latency in milliseconds
* `saveData`: `true` if the user has enabled data-saver mode
* `type`: (optional) connection medium (`'wifi'`, `'cellular'`, `'ethernet'`, `'none'`, etc., when supported)


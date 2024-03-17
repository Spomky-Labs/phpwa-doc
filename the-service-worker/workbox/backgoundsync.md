# BackgoundSync

Background Sync is a feature provided by service workers that enables your Progressive Web App to defer actions until the user has stable connectivity. This is particularly useful for ensuring any requests or data submitted by the user are not lost if their internet connection is unreliable.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    serviceworker:
        enabled: true
        src: "sw.js"
        workbox:
            background_sync:
                - queue_name: 'api'
                  match_callback: startsWith: /api/ # All requests starting with /api/
                  method: POST
                  max_retention_time: 7_200 # 5 days in minutes
                  force_sync_fallback: true #Optional
                  broadcast_channel: 'api-list'
                - queue_name: 'contact'
                  regex: /\/contact-form\// # All requests starting with /contact-form/
                  method: POST
                  max_retention_time: 120 # 2 hours in minutes
```
{% endcode %}

* `queue_name`: Unique queue name for each queue to easily identify and manage separate background sync processes. This is essential for categorizing different types of requests such as API calls and form submissions.
* `match_callback`: see on this page.
* `method`: Specifies the HTTP method (e.g., POST) for the requests to be queued. This helps in filtering which request methods should be considered for background synchronization.
* `max_retention_time`: The maximum time (in minutes) that a request will be retried in the background. This is crucial for managing storage and ensuring outdated requests are not unnecessarily retried.
* `force_sync_fallback`: (Optional) A boolean value that, when set to true, forces the background sync to attempt a sync even under conditions where it might not normally do so. Useful in critical data submission scenarios.
* `broadcast_channel`: Specifies the name of the Broadcast Channel API channel used to communicate the status of the background sync process to the rest of the application. This enables real-time update capabilities on the user interface regarding the sync status. See[ the Stimulus Component](../../symfony-ux/sync-broadcast.md) for communication between the Service Worker and the frontend.

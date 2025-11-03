# BackgroundSync

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
                  match_callback: 'startsWith: /api/' # All requests starting with /api/
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

### Configuration Parameters

#### queue_name

**Type**: `string`
**Required**: Yes

Unique queue name to identify and manage separate background sync processes. Essential for categorizing different types of requests such as API calls and form submissions.

```yaml
queue_name: 'form-submissions'
```

#### match_callback

**Type**: `string`
**Required**: Yes

Defines which requests should be queued for background sync. Uses the same match callback system as [resource caching](resource-caching.md#match-callback).

**Supported patterns**:
- `startsWith: /path/` - Match URLs starting with a path
- `regex: /pattern/` - Match URLs with regex pattern
- `origin: https://api.example.com` - Match specific origin
- JavaScript callback - Custom matching logic

**Examples**:
```yaml
match_callback: 'startsWith: /api/'  # All API calls
match_callback: 'regex: /\/submit\//''  # Submission endpoints
match_callback: '({url}) => url.pathname.includes("/api/")'  # Custom
```

See [Resource Caching Match Callback documentation](resource-caching.md#match-callback) for complete details.

#### method

**Type**: `string`
**Default**: `POST`

HTTP method for requests to be queued. Typically POST for form submissions and data modifications.

```yaml
method: POST
# or
method: PUT
```

#### max_retention_time

**Type**: `integer` (minutes)
**Required**: Yes

Maximum time a request will be retried in the background. After this duration, failed requests are discarded.

```yaml
max_retention_time: 7200  # 5 days in minutes
# or
max_retention_time: 120   # 2 hours
```

**Recommendations**:
- **Critical data**: 7200 minutes (5 days)
- **User forms**: 2880 minutes (2 days)
- **Analytics**: 1440 minutes (1 day)
- **Comments/posts**: 720 minutes (12 hours)

#### force_sync_fallback

**Type**: `boolean`
**Default**: `false`
**Optional**: Yes

Forces background sync to attempt a sync even under conditions where it might not normally do so. Useful in critical data submission scenarios.

```yaml
force_sync_fallback: true
```

**When to use**:
- Critical business transactions
- Payment submissions
- User registration data
- Important form submissions

#### broadcast_channel

**Type**: `string`
**Optional**: Yes

Name of the Broadcast Channel API channel for communicating sync status to your application. Enables real-time UI updates about sync progress.

```yaml
broadcast_channel: 'form-sync-status'
```

See the [Sync Broadcast Stimulus Component](../../symfony-ux/sync-broadcast.md) for implementation details on receiving these updates in your frontend.

#### Additional Parameters

**error_on_4xx**: `boolean` (default: `false`)
- Treat 4xx status codes as errors requiring retry

**error_on_5xx**: `boolean` (default: `true`)
- Treat 5xx status codes as errors requiring retry

**expect_redirect**: `boolean` (default: `false`)
- Expect redirect responses as success

**expected_status_codes**: `array`
- List of specific status codes to treat as success

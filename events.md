# Events

The PWA Bundle uses Symfony's Event Dispatcher system to allow customization of the manifest generation process. You can listen to these events to modify the manifest before or after compilation.

## Available Events

The bundle dispatches two main events during manifest generation:

### `PreManifestCompileEvent`

Dispatched **before** the manifest is compiled. Use this event to:
- Add dynamic data to the manifest
- Modify manifest properties based on runtime conditions
- Inject user-specific or request-specific information

### `PostManifestCompileEvent`

Dispatched **after** the manifest is compiled. Use this event to:
- Validate the compiled manifest
- Add additional processing
- Log manifest details
- Perform cleanup operations

## Basic Event Listener

Here's how to create an event listener for both events:

{% code title="src/EventListener/ManifestListener.php" lineNumbers="true" %}
```php
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use SpomkyLabs\PwaBundle\Event\PostManifestCompileEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PreManifestCompileEvent::class, method: 'onPreCompile')]
#[AsEventListener(event: PostManifestCompileEvent::class, method: 'onPostCompile')]
final readonly class ManifestListener
{
    public function onPreCompile(PreManifestCompileEvent $event): void
    {
        // Modify manifest before compilation
        $manifest = $event->manifest;

        // Optionally check the locale
        $locale = $event->locale; // ?string

        // Your custom logic here
    }

    public function onPostCompile(PostManifestCompileEvent $event): void
    {
        // Access the compiled manifest data (JSON string)
        $jsonData = $event->data;

        // Access the manifest DTO
        $manifest = $event->manifest;

        // Your custom logic here
    }
}
```
{% endcode %}

{% hint style="info" %}
Event listeners are automatically registered when using the `#[AsEventListener]` attribute. No additional configuration needed!
{% endhint %}

## Use Cases

### Adding Dynamic Data

Inject dynamic information like user preferences or runtime configuration:

{% code title="src/EventListener/DynamicManifestListener.php" lineNumbers="true" %}
```php
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use Symfony\Bundle\SecurityBundle\Security;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PreManifestCompileEvent::class)]
final readonly class DynamicManifestListener
{
    public function __construct(
        private Security $security,
    ) {
    }

    public function __invoke(PreManifestCompileEvent $event): void
    {
        $manifest = $event->manifest;

        // Add user's preferred theme color
        $user = $this->security->getUser();
        if ($user !== null && method_exists($user, 'getThemeColor')) {
            $manifest->themeColor = $user->getThemeColor();
        }
    }
}
```
{% endcode %}

### Multi-Locale Support

Adapt the manifest for different locales:

{% code title="src/EventListener/LocalizedManifestListener.php" lineNumbers="true" %}
```php
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;
use Symfony\Component\HttpFoundation\RequestStack;

#[AsEventListener(event: PreManifestCompileEvent::class)]
final readonly class LocalizedManifestListener
{
    public function __construct(
        private RequestStack $requestStack,
    ) {
    }

    public function __invoke(PreManifestCompileEvent $event): void
    {
        $manifest = $event->manifest;
        $locale = $event->locale
            ?? $this->requestStack->getCurrentRequest()?->getLocale()
            ?? 'en';

        $names = [
            'en' => 'My Application',
            'fr' => 'Mon Application',
            'es' => 'Mi Aplicación',
            'de' => 'Meine Anwendung',
        ];

        $manifest->name = $names[$locale] ?? $names['en'];
        $manifest->lang = $locale;
    }
}
```
{% endcode %}

### Environment-Based Configuration

Modify manifest based on the environment (dev/staging/prod):

{% code title="src/EventListener/EnvironmentManifestListener.php" lineNumbers="true" %}
```php
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use Symfony\Component\DependencyInjection\Attribute\Autowire;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PreManifestCompileEvent::class)]
final readonly class EnvironmentManifestListener
{
    public function __construct(
        #[Autowire('%kernel.environment%')]
        private string $environment,
    ) {
    }

    public function __invoke(PreManifestCompileEvent $event): void
    {
        $manifest = $event->manifest;

        // Add environment indicator in dev/staging
        if ($this->environment !== 'prod') {
            $manifest->name .= sprintf(' [%s]', strtoupper($this->environment));
            $manifest->backgroundColor = '#ff9800'; // Orange background for non-prod
        }
    }
}
```
{% endcode %}

### Logging and Validation

Validate manifest data and log issues:

{% code title="src/EventListener/ManifestValidationListener.php" lineNumbers="true" %}
```php
namespace App\EventListener;

use Psr\Log\LoggerInterface;
use SpomkyLabs\PwaBundle\Event\PostManifestCompileEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PostManifestCompileEvent::class)]
final readonly class ManifestValidationListener
{
    public function __construct(
        private LoggerInterface $logger,
    ) {
    }

    public function __invoke(PostManifestCompileEvent $event): void
    {
        $data = json_decode($event->data, true, 512, JSON_THROW_ON_ERROR);

        // Validate required fields
        $required = ['name', 'start_url', 'display', 'icons'];
        $missing = [];

        foreach ($required as $field) {
            if (!isset($data[$field])) {
                $missing[] = $field;
            }
        }

        if ($missing !== []) {
            $this->logger->warning('Manifest missing required fields', [
                'missing_fields' => $missing,
            ]);
        }

        $this->logger->info('Manifest compiled successfully', [
            'name' => $data['name'] ?? 'Unknown',
            'icons_count' => count($data['icons'] ?? []),
        ]);
    }
}
```
{% endcode %}

## Event API Reference

### PreManifestCompileEvent

**Properties:**

| Property | Type | Access | Description |
|----------|------|--------|-------------|
| `manifest` | `Manifest` | read/write | The manifest DTO object to modify |
| `locale` | `?string` | readonly | The locale for this compilation (null for default) |

**Usage:**
```php
$manifest = $event->manifest;
$manifest->name = 'Custom Name';
$manifest->themeColor = '#ff0000';

// Check locale
if ($event->locale === 'fr') {
    $manifest->name = 'Nom personnalisé';
}
```

### PostManifestCompileEvent

**Properties:**

| Property | Type | Access | Description |
|----------|------|--------|-------------|
| `manifest` | `Manifest` | read/write | The manifest DTO object |
| `data` | `string` | read/write | The compiled manifest as a JSON string |

**Usage:**
```php
$jsonData = $event->data;
$decoded = json_decode($jsonData, true);
$appName = $decoded['name'];
```

## Best Practices

1. **Use PreCompile for modifications**: Modify the manifest object in `PreManifestCompileEvent`
2. **Use PostCompile for validation**: Check the compiled data in `PostManifestCompileEvent`
3. **Keep listeners focused**: Each listener should handle one specific concern
4. **Avoid heavy operations**: Events are dispatched on every manifest request
5. **Test thoroughly**: Ensure your modifications don't break manifest validity
6. **Handle errors gracefully**: Don't throw exceptions that could break manifest generation

# Events

The PWA Bundle uses Symfony's PSR-14 Event Dispatcher system to allow customization of the manifest generation process. You can listen to these events to modify the manifest before or after compilation.

## Available events

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

## Basic event listener

Here's how to create an event listener for both events:

```php title="src/EventListener/ManifestListener.php" linenums="1"
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
        $manifest = $event->getManifest();

        // Your custom logic here
    }

    public function onPostCompile(PostManifestCompileEvent $event): void
    {
        // Access the compiled manifest
        $data = $event->getData();

        // Your custom logic here
    }
}
```

!!! info

    Event listeners are automatically registered when using the `#[AsEventListener]` attribute. No additional configuration needed!

## Use cases

### Adding dynamic data

Inject dynamic information like user preferences or runtime configuration:

```php title="src/EventListener/DynamicManifestListener.php" linenums="1"
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;
use Symfony\Component\Security\Core\Security;

#[AsEventListener(event: PreManifestCompileEvent::class)]
final readonly class DynamicManifestListener
{
    public function __construct(
        private Security $security
    ) {}

    public function __invoke(PreManifestCompileEvent $event): void
    {
        $manifest = $event->getManifest();

        // Add user's preferred theme color
        $user = $this->security->getUser();
        if ($user && method_exists($user, 'getThemeColor')) {
            $manifest->themeColor = $user->getThemeColor();
        }

        // Add dynamic shortcuts based on user role
        if ($this->security->isGranted('ROLE_ADMIN')) {
            $manifest->addShortcut([
                'name' => 'Admin Panel',
                'url' => '/admin',
                'icons' => [
                    ['src' => '/icons/admin.png', 'sizes' => '96x96']
                ]
            ]);
        }
    }
}
```

### Multi-locale support

Adapt the manifest for different locales:

```php title="src/EventListener/LocalizedManifestListener.php" linenums="1"
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;
use Symfony\Component\HttpFoundation\RequestStack;

#[AsEventListener(event: PreManifestCompileEvent::class)]
final readonly class LocalizedManifestListener
{
    public function __construct(
        private RequestStack $requestStack
    ) {}

    public function __invoke(PreManifestCompileEvent $event): void
    {
        $manifest = $event->getManifest();
        $locale = $this->requestStack->getCurrentRequest()?->getLocale() ?? 'en';

        // Set localized app name
        $names = [
            'en' => 'My Application',
            'fr' => 'Mon Application',
            'es' => 'Mi Aplicación',
            'de' => 'Meine Anwendung'
        ];

        $manifest->name = $names[$locale] ?? $names['en'];
        $manifest->lang = $locale;
    }
}
```

### Environment-based configuration

Modify manifest based on the environment (dev/staging/prod):

```php title="src/EventListener/EnvironmentManifestListener.php" linenums="1"
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use Symfony\Component\DependencyInjection\Attribute\Autowire;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PreManifestCompileEvent::class)]
final readonly class EnvironmentManifestListener
{
    public function __construct(
        #[Autowire('%kernel.environment%')]
        private string $environment
    ) {}

    public function __invoke(PreManifestCompileEvent $event): void
    {
        $manifest = $event->getManifest();

        // Add environment indicator in dev/staging
        if ($this->environment !== 'prod') {
            $manifest->name .= sprintf(' [%s]', strtoupper($this->environment));
            $manifest->backgroundColor = '#ff9800'; // Orange background for non-prod
        }
    }
}
```

### Logging and validation

Validate manifest data and log issues:

```php title="src/EventListener/ManifestValidationListener.php" linenums="1"
namespace App\EventListener;

use Psr\Log\LoggerInterface;
use SpomkyLabs\PwaBundle\Event\PostManifestCompileEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PostManifestCompileEvent::class)]
final readonly class ManifestValidationListener
{
    public function __construct(
        private LoggerInterface $logger
    ) {}

    public function __invoke(PostManifestCompileEvent $event): void
    {
        $data = $event->getData();

        // Validate required fields
        $required = ['name', 'start_url', 'display', 'icons'];
        $missing = [];

        foreach ($required as $field) {
            if (!isset($data[$field])) {
                $missing[] = $field;
            }
        }

        if (!empty($missing)) {
            $this->logger->warning('Manifest missing required fields', [
                'missing_fields' => $missing
            ]);
        }

        // Log successful compilation
        $this->logger->info('Manifest compiled successfully', [
            'name' => $data['name'] ?? 'Unknown',
            'icons_count' => count($data['icons'] ?? [])
        ]);
    }
}
```

### Adding analytics tracking

Include analytics parameters in URLs:

```php title="src/EventListener/AnalyticsManifestListener.php" linenums="1"
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PreManifestCompileEvent::class)]
final readonly class AnalyticsManifestListener
{
    public function __invoke(PreManifestCompileEvent $event): void
    {
        $manifest = $event->getManifest();

        // Add UTM parameters to start URL
        $manifest->startUrl .= '?utm_source=pwa&utm_medium=app&utm_campaign=install';

        // Add tracking to shortcuts
        if (isset($manifest->shortcuts)) {
            foreach ($manifest->shortcuts as &$shortcut) {
                $shortcut['url'] .= '?utm_source=pwa&utm_medium=shortcut';
            }
        }
    }
}
```

## Event API reference

### PreManifestCompileEvent

**Methods:**

- `getManifest()`: Returns the `Manifest` DTO object (read/write)
- `getRequest()`: Returns the current `Request` object (read-only)

**Usage:**
```php
$manifest = $event->getManifest();
$manifest->name = 'Custom Name';
$manifest->themeColor = '#ff0000';
```

### PostManifestCompileEvent

**Methods:**

- `getData()`: Returns the compiled manifest as an array (read-only)
- `getRequest()`: Returns the current `Request` object (read-only)

**Usage:**
```php
$data = $event->getData();
$appName = $data['name'];
$iconCount = count($data['icons'] ?? []);
```

## Best practices

1. **Use PreCompile for modifications**: Modify the manifest object in `PreManifestCompileEvent`
2. **Use PostCompile for validation**: Check the compiled data in `PostManifestCompileEvent`
3. **Keep listeners focused**: Each listener should handle one specific concern
4. **Avoid heavy operations**: Events are dispatched on every manifest request
5. **Test thoroughly**: Ensure your modifications don't break manifest validity
6. **Handle errors gracefully**: Don't throw exceptions that could break manifest generation

## Debugging events

Enable logging to see when events are dispatched:

```yaml title="config/packages/monolog.yaml" linenums="1"
monolog:
    channels: ['event']

    handlers:
        event:
            type: stream
            path: '%kernel.logs_dir%/events.log'
            level: debug
            channels: ['event']
```

Then log in your listener:

```php
public function __invoke(PreManifestCompileEvent $event): void
{
    $this->logger->debug('PreManifestCompileEvent dispatched', [
        'manifest_name' => $event->getManifest()->name
    ]);
}
```

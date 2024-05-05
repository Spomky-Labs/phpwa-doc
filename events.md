# Events

Leveraging the PSR-14 Event Dispatcher within the bundle provides you an opportunity to fine-tune the manifest creation process by hooking into events dispatched before and after the manifest compilation. Below, you'll find how you can listen to these events and implement custom logic accordingly.

Events are:

* `SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent`
* `SpomkyLabs\PwaBundle\Event\PostManifestCompileEvent`

Here's a basic structure for listening to both events:

{% code title="src/EventListener\MyListener.php" lineNumbers="true" %}
```php
namespace App\EventListener;

use SpomkyLabs\PwaBundle\Event\PreManifestCompileEvent;
use SpomkyLabs\PwaBundle\Event\PostManifestCompileEvent;

use Symfony\Component\EventDispatcher\Attribute\AsEventListener;

#[AsEventListener(event: PreManifestCompileEvent ::class, method: 'preCompile')]
#[AsEventListener(event: PostManifestCompileEvent::class, method: 'postCompile')]
final readonly class MyListener
{
    public function preCompile(PreManifestCompileEvent $event): void
    {
        // ...
    }

    public function postCompile(PostManifestCompileEvent $event): void
    {
        // ...
    }
}
```
{% endcode %}

# Content Security Policy

Content Security Policy (CSP) is a security standard that helps prevent cross-site scripting (XSS), clickjacking, and other code injection attacks. When using CSP with PWAs, proper configuration is essential to allow service worker registration while maintaining security.

## Why CSP Matters for PWAs

CSP headers control which resources can be loaded and executed. Service workers require JavaScript execution, which CSP may block if not configured correctly.

**Common CSP issues with PWAs**:
- Service worker registration blocked
- Service worker scripts not loading
- Workbox scripts failing to load
- Background sync failures
- Cache API access denied

## Basic CSP Configuration

### Using Nonce (Recommended)

The PWA Bundle supports CSP nonces for service worker registration:

{% code title="templates/base.html.twig" lineNumbers="true" %}
```twig
<!DOCTYPE html>
<html lang="en">
<head>
    {{ pwa(swAttributes={nonce: csp_nonce()}) }}
</head>
<body>
    {% block body %}{% endblock %}
</body>
</html>
```
{% endcode %}

{% hint style="success" %}
**Nelmio Security Bundle**: If using [Nelmio Security Bundle](https://symfony.com/bundles/NelmioSecurityBundle/current/index.html#content-security-policy), nonces are automatically generated and applied. No manual configuration needed.
{% endhint %}

### Manual Nonce Generation

If not using Nelmio, generate nonces manually:

{% code title="src/Twig/CspExtension.php" lineNumbers="true" %}
```php
<?php

namespace App\Twig;

use Twig\Extension\AbstractExtension;
use Twig\TwigFunction;

class CspExtension extends AbstractExtension
{
    private ?string $nonce = null;

    public function getFunctions(): array
    {
        return [
            new TwigFunction('csp_nonce', [$this, 'getNonce']),
        ];
    }

    public function getNonce(): string
    {
        if ($this->nonce === null) {
            $this->nonce = base64_encode(random_bytes(16));
        }

        return $this->nonce;
    }
}
```
{% endcode %}

Then add the nonce to your CSP header in a middleware or response listener.

## CSP Headers for PWAs

### Minimal CSP Configuration

Allows service workers with nonce-based script execution:

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{NONCE}';
  worker-src 'self';
  manifest-src 'self';
```

### Complete PWA CSP Configuration

Comprehensive CSP for a full-featured PWA:

```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'nonce-{NONCE}';
  worker-src 'self' blob:;
  connect-src 'self' wss: https:;
  img-src 'self' data: blob: https:;
  style-src 'self' 'unsafe-inline';
  font-src 'self' data:;
  manifest-src 'self';
  frame-src 'self';
  media-src 'self' blob:;
```

**Directive explanations**:

- **default-src 'self'**: Default policy for all resources
- **script-src 'self' 'nonce-{NONCE}'**: Allow scripts from origin + nonce
- **worker-src 'self' blob:**: Allow service workers + blob workers
- **connect-src 'self' wss: https:**: Allow fetch/XHR + WebSockets
- **img-src 'self' data: blob: https:**: Allow images from multiple sources
- **style-src 'self' 'unsafe-inline'**: Allow inline styles (or use nonce)
- **font-src 'self' data:**: Allow fonts from origin + data URIs
- **manifest-src 'self'**: Allow manifest from origin
- **media-src 'self' blob:**: Allow media + blob URLs

### Using Workbox CDN

If using Workbox from CDN, add the CDN to your CSP:

```
Content-Security-Policy:
  script-src 'self' 'nonce-{NONCE}' https://storage.googleapis.com;
  connect-src 'self' https://storage.googleapis.com;
```

Or configure local Workbox to avoid CDN:

```yaml
pwa:
    serviceworker:
        workbox:
            use_cdn: false  # Use local Workbox files
```

## Symfony Configuration

### Using Nelmio Security Bundle

{% code title="config/packages/nelmio_security.yaml" lineNumbers="true" %}
```yaml
nelmio_security:
    content_security_policy:
        enabled: true

        # Use nonces for scripts
        script_nonce: true

        # CSP directives
        default-src:
            - 'self'

        script-src:
            - 'self'
            - 'nonce'  # Auto-generated nonce

        worker-src:
            - 'self'
            - 'blob:'

        connect-src:
            - 'self'
            - 'https:'
            - 'wss:'

        img-src:
            - 'self'
            - 'data:'
            - 'blob:'
            - 'https:'

        style-src:
            - 'self'
            - 'unsafe-inline'  # Or use style_nonce: true

        font-src:
            - 'self'
            - 'data:'

        manifest-src:
            - 'self'

        media-src:
            - 'self'
            - 'blob:'
```
{% endcode %}

### Custom Event Subscriber

For custom CSP header management:

{% code title="src/EventSubscriber/CspSubscriber.php" lineNumbers="true" %}
```php
<?php

namespace App\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\ResponseEvent;
use Symfony\Component\HttpKernel\KernelEvents;

class CspSubscriber implements EventSubscriberInterface
{
    public function __construct(
        private readonly string $nonce
    ) {
    }

    public static function getSubscribedEvents(): array
    {
        return [
            KernelEvents::RESPONSE => 'onKernelResponse',
        ];
    }

    public function onKernelResponse(ResponseEvent $event): void
    {
        if (!$event->isMainRequest()) {
            return;
        }

        $response = $event->getResponse();

        $csp = sprintf(
            "default-src 'self'; " .
            "script-src 'self' 'nonce-%s'; " .
            "worker-src 'self' blob:; " .
            "connect-src 'self' https: wss:; " .
            "img-src 'self' data: blob: https:; " .
            "style-src 'self' 'unsafe-inline'; " .
            "font-src 'self' data:; " .
            "manifest-src 'self';",
            $this->nonce
        );

        $response->headers->set('Content-Security-Policy', $csp);
    }
}
```
{% endcode %}

## Common CSP Issues

### 1. Service Worker Registration Fails

**Error**: `Refused to execute inline script because it violates CSP directive`

**Solution**: Add nonce to service worker registration script:

```twig
{{ pwa(swAttributes={nonce: csp_nonce()}) }}
```

### 2. Workbox Not Loading

**Error**: `Refused to load script from 'https://storage.googleapis.com/workbox-cdn/...'`

**Solution**: Add Workbox CDN to script-src or use local Workbox:

```yaml
# Option 1: Allow CDN
nelmio_security:
    content_security_policy:
        script-src:
            - 'https://storage.googleapis.com'

# Option 2: Use local Workbox (recommended)
pwa:
    serviceworker:
        workbox:
            use_cdn: false
```

### 3. fetch() Fails in Service Worker

**Error**: `Refused to connect to 'https://api.example.com'`

**Solution**: Add API domains to connect-src:

```yaml
nelmio_security:
    content_security_policy:
        connect-src:
            - 'self'
            - 'https://api.example.com'
```

### 4. Images Not Caching

**Error**: `Refused to load image from 'blob:...'`

**Solution**: Add blob: to img-src:

```yaml
nelmio_security:
    content_security_policy:
        img-src:
            - 'self'
            - 'blob:'
            - 'data:'
```

### 5. Background Sync Fails

**Error**: `Refused to create a worker from 'blob:...'`

**Solution**: Add blob: to worker-src:

```yaml
nelmio_security:
    content_security_policy:
        worker-src:
            - 'self'
            - 'blob:'
```

## Testing CSP Configuration

### 1. Check CSP Headers

```bash
# Using curl
curl -I https://your-app.com | grep -i content-security-policy

# Expected output
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-...'
```

### 2. Browser DevTools

```bash
1. Open DevTools (F12)
2. Go to Console tab
3. Look for CSP violations
4. Check Network tab for blocked requests
```

### 3. CSP Evaluator

Use Google's CSP Evaluator to validate your policy:
- https://csp-evaluator.withgoogle.com/

### 4. Report-Only Mode

Test CSP without blocking (reports violations only):

```yaml
nelmio_security:
    content_security_policy:
        report-only: true
        report-uri: '/csp-report'
```

## CSP Reporting

### Enable CSP Reports

Collect CSP violation reports to identify issues:

```yaml
nelmio_security:
    content_security_policy:
        report-uri: '/csp-violation-report'
```

### Create Report Endpoint

{% code title="src/Controller/CspController.php" lineNumbers="true" %}
```php
<?php

namespace App\Controller;

use Psr\Log\LoggerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class CspController extends AbstractController
{
    #[Route('/csp-violation-report', methods: ['POST'])]
    public function report(Request $request, LoggerInterface $logger): Response
    {
        $report = json_decode($request->getContent(), true);

        $logger->warning('CSP Violation', [
            'blocked_uri' => $report['csp-report']['blocked-uri'] ?? 'unknown',
            'violated_directive' => $report['csp-report']['violated-directive'] ?? 'unknown',
            'document_uri' => $report['csp-report']['document-uri'] ?? 'unknown',
        ]);

        return new Response('', Response::HTTP_NO_CONTENT);
    }
}
```
{% endcode %}

## Production vs Development

### Development Configuration

More permissive for easier debugging:

```yaml
# config/packages/dev/nelmio_security.yaml
nelmio_security:
    content_security_policy:
        enabled: true
        report-only: true  # Report violations without blocking

        script-src:
            - 'self'
            - 'unsafe-inline'  # Allow for hot reload
            - 'unsafe-eval'    # Allow for dev tools
```

### Production Configuration

Strict policy for maximum security:

```yaml
# config/packages/prod/nelmio_security.yaml
nelmio_security:
    content_security_policy:
        enabled: true
        report-only: false  # Enforce policy

        script-src:
            - 'self'
            - 'nonce'  # Only nonce-based scripts
```

## Best Practices

### 1. Use Nonces, Not unsafe-inline

```yaml
# ✗ Avoid
script-src:
    - 'self'
    - 'unsafe-inline'

# ✓ Prefer
script-src:
    - 'self'
    - 'nonce'
```

### 2. Use Local Resources

Avoid relying on external CDNs when possible:

```yaml
pwa:
    serviceworker:
        workbox:
            use_cdn: false  # Use local Workbox
```

### 3. Minimize 'unsafe-*' Directives

Only use when absolutely necessary:

```yaml
# ✓ Good - no unsafe directives
style-src:
    - 'self'
    - 'nonce'

# ✗ Avoid if possible
style-src:
    - 'self'
    - 'unsafe-inline'
```

### 4. Start Strict, Relax if Needed

Begin with restrictive policy and relax gradually:

```yaml
# Start here
default-src: ['self']

# Add as needed
connect-src: ['self', 'https://api.example.com']
```

### 5. Monitor Violations

Use report-uri to catch violations in production:

```yaml
content_security_policy:
    report-uri: '/csp-report'
```

## Platform-Specific Considerations

### iOS/Safari

Safari has strict CSP enforcement. Ensure:
- `manifest-src 'self'` is set
- `worker-src 'self'` allows service workers
- No unsafe-eval in production

### Android/Chrome

Chrome supports full CSP spec. Take advantage of:
- `'strict-dynamic'` for script loading
- `'unsafe-hashes'` for specific inline scripts
- Full nonce support

### Progressive Enhancement

Provide fallbacks for browsers without service worker support:

```javascript
if ('serviceWorker' in navigator) {
    // Register with CSP nonce
    navigator.serviceWorker.register('/sw.js');
} else {
    // Fallback for browsers without SW support
    console.warn('Service workers not supported');
}
```

## Troubleshooting Checklist

When CSP blocks your PWA:

- [ ] Check Console for CSP violations
- [ ] Verify nonce is correctly passed to pwa()
- [ ] Check worker-src includes 'self' and 'blob:'
- [ ] Verify connect-src allows your API domains
- [ ] Check img-src allows blob: and data: URIs
- [ ] Ensure manifest-src includes 'self'
- [ ] Test with report-only mode first
- [ ] Use CSP Evaluator to validate policy
- [ ] Check Workbox uses local files (use_cdn: false)
- [ ] Verify all external resources are whitelisted

## Related Documentation

- [Service Worker Configuration](configuration.md) - Service worker setup
- [Workbox](workbox/) - Workbox configuration options
- [Nelmio Security Bundle](https://symfony.com/bundles/NelmioSecurityBundle/current/index.html) - Symfony CSP configuration

## Resources

- **MDN CSP**: https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP
- **CSP Evaluator**: https://csp-evaluator.withgoogle.com/
- **CSP Cheat Sheet**: https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html
- **Can I Use CSP**: https://caniuse.com/contentsecuritypolicy

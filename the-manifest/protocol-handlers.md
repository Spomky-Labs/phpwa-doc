# Protocol Handlers

Progressive Web Apps (PWAs) have the ability to handle web protocols, which means they can respond to specific URL schemes. For example, a PWA can register to handle `mailto:` links, so that clicking on an e-mail link can open a compose window in the PWA instead of opening the default mail client.

The application can declare custom protocols using the prefix `web+`.

With protocol handlers, your PWA can provide a more integrated user experience, functioning more like a native application.

### Configuration

```yaml
pwa:
    manifest:
        protocol_handlers:
            - protocol: "mailto"
              url: "/compose?to=%s"
              title: "Compose Email"
            - protocol: "web+custom"
              url:
                  path: "app_feature1"
                  params:
                      foo: "bar"
              title: "Open with Feature #1"
```

### Considerations When Using Protocol Handlers

* Ensure that your PWA is served over HTTPS, as handling protocols can present security risks if not properly secured.
* Test the protocol handlers thoroughly on different browsers, as support for this feature can vary.
* Respect user choice. Always provide an easy way for users to opt-out of using the PWA as a default handler for specific protocols.

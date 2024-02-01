# Icons

To integrate the icon details into the Progressive Web App (PWA) manifest file, ensure that each icon listed is accompanied by its respective size. For example, `icon-256x256.png` is indicated as having a size of 256px by 256px. This is crucial for providing clear visual elements across different devices and resolutions.

The `sizes` attribute indicates the size of the icon to the browser. For PNG or JPEG icons, specify the dimensions (e.g., 48, 96, 256). For vector icons, you can use "any" as they are scalable without losing quality. The `type` attribute is also important as it tells the browser what the file format is, helping it to render the image correctly.

```yaml
pwa:
    manifest:
        icons:
            - src: "icons/icon-48x48.png"
              type: "image/png"
              sizes: [48]
            - src: "icons/icon-192x192.png"
              sizes: [192]
            - src: "icons/icon-192x192.png"
              sizes: [192]
              purpose: "maskable"
            - src: "icons/icon.svg"
              sizes: [0] # 0 means any and is suitable only for vector images
```

### `src` Parameter



### `sizes` Parameter



### `type` Parameter



### `purpose` Parameter

The purpose `maskable` icons indicates the icon has a security margin and borders can be cropped on certain devices.

<figure><img src="../.gitbook/assets/maskable-icon-safe-area (1).png" alt=""><figcaption><p>Maskable image safe area</p></figcaption></figure>

# Screenshots

Taking screenshots of your application has great benefits.It may become difficult to manage screenshot updates as the application evolves, in particular if multiple sizes are managed.

This command requires dependencies and **should be executed in `dev` environnement only**.

```sh
composer require --dev symfony/panther
composer require --dev dbrekelmans/bdi && vendor/bin/bdi detect drivers
```

### Image Processor

Please read the section on [the Icons page](screenshots.md#image-processor).

### Taking Screenshots

From your shell, you can use the following command line to take a screenshot of a given URL.

```sh
symfony console pwa:create:screenshot https://example.com
```

This will create a folder in `assets/screenshots` with screenshot. In addition, the console output will give you the best configuration to use these new image files by copy-pasting.

```yaml
pwa:
    manifest:
        screenshots:
            - src: screenshots/screenshot-1200x1100.png
              width: 1200
              height: 1100
              type: image/png
            - ...
```

### Formats And Sizes

You can change the format and sizes by setting the values you need as options. It is recommended to provide mobile and desktop views of your application.

```sh
symfony console pwa:create:screenshot https://example.com --format="webp" --width=750 --height=1334
```

### Output Folder

Similarily, if you want to change the output folder, you can set it as an option.

```sh
symfony console pwa:create:screenshot https://example.com --output="/foo/bar"
```

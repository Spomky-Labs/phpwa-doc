# Icons

Progressive Web Apps may have icons of multiple formats and sizes to be correctly shown on targeted plaforms. This task may become boring as your application evolves.

The bundle provides a simple console command to ease the creation of these icons. It requires an image processor to work to be declared in the configuration file. Two Image Processors are provided by the bundle.

* `pwa.image_processor.imagick`: requires the PHP Imagick extension
* `pwa.image_processor.gd`: requires the PHP GD extension

You can use a custom service if needed. It must implement the interface `SpomkyLabs\PwaBundle\ImageProcessor\ImageProcessor`.

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    image_processor: 'pwa.image_processor.imagick'
```
{% endcode %}

Then, from your shell you can use the following command line to convert an image to a preset of sizes and using the same format as the input.

{% hint style="warning" %}
Icons shall be square images
{% endhint %}

```sh
symfony console pwa:create:icons /path/to/the/image.png
```

This will create a folder in `assets/icons` with icons of sizes 16x16, 32x32, 48x48, 96x96, 144x144, 180x180, 256x256, 512x512 and 1024x1024. In addition, the console output will give you the best configuration to use these new image files by copy-pasting.

```yaml
pwa:
    manifest:
        icons:
            - src: icons/icon-16x16.jpg
              sizes:
                - 16
              type: image/png
            - ...
```

{% hint style="info" %}
This configuration is to be adapt depending on the icon type (application, shortcut, widget...)
{% endhint %}

You can change the format and sizes by setting the values you need as options

```sh
symfony console pwa:create:icons /path/to/the/image.png --format="jpg" 48 96 256
```

Similarily, if you want to change the output folder, you can set it as an option.

```sh
symfony console pwa:create:icons /path/to/the/image.png --output="/foo/bar"
```

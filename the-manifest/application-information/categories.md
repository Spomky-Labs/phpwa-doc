# Categories

The `categories` member is a string list that describes the application categories to which the web application belongs. It is meant as a hint to catalogs or stores listing web applications and it is expected that these will make a best effort to find appropriate categories (or category) under which to list the web application. Like search engines and meta keywords, catalogs and stores are not required to honor this hint.

{% hint style="info" %}
Manifest authors are encouraged to use lower-case.
{% endhint %}

{% code title="/config/packages/pwa.yaml" lineNumbers="true" %}
```yaml
pwa:
    manifest:
        enabled: true
        categories: ['games', 'kids']
```
{% endcode %}

List of known categories:

* beauty
* books
* books & reference
* business
* cars
* dating
* design
* developer
* developer tools
* development
* education
* entertainment
* events
* fashion
* finance
* fitness
* food
* fundraising
* games
* government
* graphics
* graphics & design
* health
* health & fitness
* kids
* lifestyle
* magazines
* medical
* multimedia
* multimedia design
* music
* navigation
* network
* networking
* news
* parenting
* personalization
* pets
* photo
* photo & video
* politics
* productivity
* reference
* security
* shopping
* social
* social networking
* sports
* transportation
* travel
* utilities
* video
* weather

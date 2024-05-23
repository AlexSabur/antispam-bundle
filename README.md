# The original author has deleted the package, but it was used in local project.

NucleosAntiSpamBundle
=====================
This bundle provides some basic features to reduce spam in Symfony. It is the successor of `core23/antispam-bundle`, but not related to `isometriks/spam-bundle`.

## Features

* **Honeypot protection for forms:** An additional "hidden" (i.e. made invisible with CSS) field will be added to your form. Whoever fills out this field, is considered to be a spam bot.

* **Time protection for forms:** The time between *displaying* the form and *submitting* the form is measured. Anybody who submits the form quicker than a certain number of seconds, is considered to be a spam bot. The timestamp is stored in the session.

* **Email address obfuscation filter for Twig:** To prevent spam harvest bots from detecting your email address, they are obfuscated by e.g. replacing `@` with `[AT]`. The filter will find email addresses automatically, so you can apply it to your entire text.

## Installation

Open a command console, enter your project directory and execute the following command to download the latest stable version of this bundle:

```
composer require alexsabur/antispam-bundle
```

### Enable the Bundle

In older versions of Symfony, you need to enable it manually:

```php
// config/bundles.php

return [
    // ...
    Nucleos\AntiSpamBundle\NucleosAntiSpamBundle::class => ['all' => true],
];
```

## Usage

### Form based protection

In a controller:

```php
$this->createForm(CustomFormType:class, null, [
    // Time protection
    'antispam_time'     => true,
    'antispam_time_min' => 10, // seconds
    'antispam_time_max' => 60,

    // Honeypot protection
    'antispam_honeypot'       => true,
    'antispam_honeypot_class' => 'hide-me',
    'antispam_honeypot_field' => 'email-repeat',
])
```

In a form class:

```php
class MyType extends AbstractType
{
    // ...

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            // ...
            'antispam_time'     => true,
            'antispam_time_min' => 10,
            // same as above
        ]);
    }
}
```

### Twig email address obfuscation

The Twig filter `antispam` replaces `@` by e.g. `[AT]`.

```twig
{# Replace plain text #}
{{ text|antispam }}

{# Replace rich text mails #}
{{ htmlText|antispam(true) }}
```

If you want a JavaScript decoding for the encoded email addresses, you should use the `AntiSpam.js` library:

```javascript
document.addEventListener('DOMContentLoaded', () => {
  new AntiSpam('.custom_class');
});
```

It is recommended to use [webpack](https://webpack.js.org/) / [webpack-encore](https://github.com/symfony/webpack-encore)
to include the JavaScript library in your page. This file is located in the `assets` folder.

### Configure the Bundle

Create a configuration file called `nucleos_antispam.yaml`:

```yaml
# config/packages/nucleos_antispam.yaml

nucleos_antispam:
    # Twig mail filter
    twig:
        mail:
            css_class: 'custom_class'
            at_text:   [ '[AT]', '(AT)', '[ÄT]' ]
            dot_text:  [ '[DOT]', '(DOT)', '[.]' ]

    # Time protection
    time:
        min: 5
        max: 3600
        global: true # This will add antispam to all forms

    # Honeypot protection
    honeypot:
        field: 'email_address'
        class: 'hidden'
        global: false
        provider: 'nucleos_antispam.provider.session'
```

## License

This bundle is under the [MIT license](LICENSE.md).

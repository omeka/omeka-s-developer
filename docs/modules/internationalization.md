# Internationalizing your module

The Omeka S core contains tools for internationalizing the strings in your modules to create `template.pot` files suitable for adding to Transifex or any other translating service.

## Preparation

### System requirements

Omeka S's tools for generating `template.pot` files use the [`gulp` javascript toolkit](https://gulpjs.com). You will need to install it on your system to use Omeka S's tools.

In turn, those tasks rely on the `gettext` package. It is also recommended to use `translate-toolkit` for debugging.


### Module preparation

To prepare your module for translation, there are two different additions to your code to add.

In view files, any strings to be translated should be wrapped in a call to `translate()`. Usually, this will look like:

```php
<?php echo $this->translate("The string to make translatable"); ?>
```

When variables need to be added to a translatable string directly, use `sprintf` outside the call to `translate()`.

```php
<?php echo sprintf($this->translate('version %s'), Omeka\Module::VERSION); ?>

```

In other files where the translation happens indirectly (e.g., configuration, controllers, forms, etc.), mark the string for translation with a comment:

```
{code} // @translate
```

For example, when creating a form you can mark the label as follows:

```php
        $generalFieldset->add([
            'name' => 'administrator_email',
            'type' => 'Email',
            'options' => [
                'label' => 'Administrator email', // @translate
            ],
        ]);
```

Any strings that have the `// @translate` comment will by picked up by Omeka S's tooling and added to the `template.pot` file.

The strings must be on a single line. Breaking them up across multiple lines of code with result in Zend's translation mechanism failing to see them as one string, and so they will not be processed successfuly.

When indirect translations also require string replacements as you would with `sprintf`, use an `Omeka\Stdlib\Message` object. The `sprintf` is not needed with this object.

```php
$message = new Message(
    'API key successfully created.<br><br>Here is your key ID and credential for access to the API. WARNING: "key_credential" will be unretrievable after you navigate away from this page.<br><br>key_identity: <code>%s</code><br>key_credential: <code>%s</code>', // @translate
$keyId, $keyCredential
);
$message->setEscapeHtml(false);
$this->messenger()->addWarning($message);

```

Finally, you should create a `/language` directory at the root of your module, and reference it in your module's config/module.config.php file:

```php
return [
// ...
    'translator' => [
        'translation_file_patterns' = [
            'type' => 'gettext',
            'base_dir' => dirname(__DIR__) . '/language',
            'pattern' => '%s.mo',
            'text_domain' => null,
        ],
    ],
// ...
];
```
## Creating and compiling the strings 

The `gulp i18n:module:template --module MyModule` task will generate the `template.pot` file in the `/language` directory. The `--module MyModule` parameter refers to the directory name of your module. The task can be run from any directory within your module.

When translations based on the `template.pot` file have been created, add their `*.po` files to the `/language` directory. All files should be named according to their correct localization code (e.g., `en-GB.po`).

Then, use the `gulp i18n:module:compile --module MyModule` task to compile the `*.po` files into their corresponding `*.mo` binaries.

Between those two steps, you will likely want to check that you have correctly marked all the appropriate strings for translation. The `translate-toolkit` Linux package provides the [`podebug`](http://docs.translatehouse.org/projects/translate-toolkit/en/latest/commands/podebug.html) command to create a testing translation. Running `podebug -i template.pot -o debug.po --rewrite=unicode` from the `/language` directory will create a `debug.po` file with pseudo-translations (other options are available).

After creating the `debug.po` file and compiling its `debug.mo` counterpart, in Omeka S's `/config/local.config.php` file set the translator locale value to `debug`:

```php
    'translator' => [
        'locale' => 'debug',
    ],

```

Then browse all your module's pages and update any untranslated strings, and repeat the process.

## Static translations

As of Omeka S version 1.2, you can also include a template file of static translations that will be included in the final template.
This can be useful if you have strings that need to be translated but can't easily be found by `xgettext` or marked with an `@translate`
comment.

The `gulp i18n:module:template` command will look for a static template at `language/template.static.pot`.

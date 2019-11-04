---
title: Internationalization for Modules
---

The Omeka S core contains tools for internationalizing the strings in your modules to create
`template.pot` files suitable for adding to Transifex or any other translating service, and
for handling translated `.po` files.

This page covers topics specific to internationalization in modules. For general information
about the Omeka S translation system, see the [main Internationalization page](../miscellaneous/internationalization.md).

## Module preparation

To prepare your module for translation, you need to prepare the strings themselves
throughout your code, and also create and configure a directory that will hold the
translation files.

For handling the strings, the same guidelines as given in the
[main documentation on making strings translatable](../miscellaneous/internationalization.md#making-strings-translatable) apply to modules as well.

For the directory, create a `/language` directory at the root of your module, and reference
it in your module's config/module.config.php file:

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

The `gulp i18n:module:template` task will generate the `template.pot` file in the `/language`
directory. If run from within the module, the task will automatically detect the correct
module to use, otherwise the `--module MyModule` parameter can be given, where the value
refers to the directory name of the module.

When translations based on the `template.pot` file have been created, add their `*.po` files
to the `/language` directory. All files should be named according to their correct
localization code (e.g., `en-GB.po`).

Then, use the `gulp i18n:module:compile` task to compile the `*.po` files into their
corresponding `*.mo` binaries.

Between those two steps, you will likely want to check that you have correctly marked all the
appropriate strings for translation. The `translate-toolkit` Linux package provides the
[`podebug`](http://docs.translatehouse.org/projects/translate-toolkit/en/latest/commands/podebug.html)
command to create a testing translation. Running
`podebug -i template.pot -o debug.po --rewrite=unicode` from the `/language` directory will
create a `debug.po` file with pseudo-translations (other options are available).

After creating the `debug.po` file and compiling its `debug.mo` counterpart, in Omeka S's
`/config/local.config.php` file set the translator locale value to `debug`:

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

---
title: Internationalization
---

Omeka S includes support for presenting interfaces to users in different languages.
The source strings for Omeka and its modules are written in English, and we rely on
volunteers to translate those strings.

## Basics

Omeka S stores translated strings in the [gettext](https://www.gnu.org/software/gettext/manual/gettext.html)
format. There are three basic types of file:

- a `.pot` file is a translation template: it contains only the English-language source
  strings.
- `.po` files are where translated strings are stored. They're the same as the `.pot` file,
  except the translations for each string are filled in. Each language has its own `.po`
  file.
- `.mo` files are compiled versions of `.po` files. The `.mo` file is what Omeka S actually
  uses to read and use the translated strings for a particular language.

All three of these types of file are found in the `languages` directory
(`application/languages` for the core). Generally, translators will deal with the first two
types through Transifex (see next section).

## Transifex

Omeka S uses the [Transifex](https://www.transifex.com/omeka/omeka-s/) platform to allow
a simpler way for people to contribute translations. The core and modules written by the
Omeka team are handled through the same project on Transifex. Through the Transifex site,
you can join existing translation teams, request new languages, and make changes or
contributions to the translations.

## Making strings translatable

When writing code in Omeka S, strings that will be presented to the user need some special
handling so that they can be translated.

### Direct translation: within a view

Instead of writing strings directly in the markup, use the `translate()` helper.

As a simple real example, see this line from the installer view:

```php
<h1><?php echo $this->translate('Install Omeka S'); ?></h1>
```

Using `translate()` will handle actually replacing the English source string with the
translated equivalent, and also serve as a marker that a string is translatable for
when we build the translation template that makes strings available to the translators.

### Indirect translation: within forms and other classes

Code that uses the `Zend\Form` classes will automatically translate strings that get
presented to the user like labels, legends, and descriptions.

Several other areas where user-facing strings are defined are also automatically translated,
such as labels for site blocks and media ingesters, labels for navigation entries defined in
config files, and error messages.

In these situations, all that's necessary is to comment the line to be translated with
`// @translate` at the end of the line. The translation itself will happen with or without
the comment, but the comment is needed so our automated template processing will pick up the
string to be translated and make it available to the translation teams.

The `// @translate` comment applies only to the immediately-previous string. If a single
line contains multiple strings that need to be marked as translatable, or has one
translatable string that appears in between untranslatable ones, it may be necessary to
split the original line into multiple lines.

Here's an example of using the `// @translate` comment in the site navigation section of
a module's `module.config.php` file:

```php
return [
    'navigation' => [
        'site' => [
            [
                'label' => 'Collecting', // @translate
                'route' => 'admin/site/slug/collecting',
                'action' => 'index',
                'useRouteMatch' => true,
                'pages' => [
                    // .....
                ],
            ],
        ]
    ]
];
```

And here's one from a form class that extends `Zend\Form\Form`:

```php
public function init()
{
    $this->add([
        'name' => 'o:email',
        'type' => 'Email',
        'options' => [
            'label' => 'Email', // @translate
        ],
        'attributes' => [
            'id' => 'email',
            'required' => true,
        ],
    ]);
};
```

### Handling strings with dynamic parts

The previous examples are focused on simple, static strings, but often it's
necessary to have strings with parts that change, like data that comes from users, or
counts of search results. Simply passing the existing string as-is to the translation
system won't work correctly: every change in the data will be treated as a "new" string
that won't get translated.

In those situations, the dynamic part of the string must be replaced by a "placeholder,"
leading to a translatable part that stays the same, which the dynamic parts are then
inserted into. To do this, Omeka S uses the PHP function [sprintf](https://www.php.net/manual/en/function.sprintf.php).

The basic placeholder for sprintf is `%s`. It's simplest to show an example of how this
gets used to make a dynamic string translatable. The following untranslated code:

```php
There are <?php echo $this->escapeHtml($results); ?> results.
```

becomes this when translated:

```php
<?php echo sprintf($this->translate('There are %s results.'), $this->escapeHtml($results)); ?>
```

Note that the call to `translate()` is *inside* the call to `sprintf`.

There's a slightly more complex style of placeholder that's used when one string contains
*multiple* dynamic parts. Some languages may need to reorder these parts in a different way
than the original English string, which means there needs to be something that differentiates
the first placeholder from the second, and so on. These placeholders look like this: `%1$s`,
`%2$s`, and so on.

There's a slightly different method used when dealing with these "dynamic" strings outside
of views (in places like class files, for example). Instead of using `sprintf` directly,
there's a class `Omeka\Stdlib\Message` that is used to pass the static and dynamic parts
of the string around together. 

```php
<?php
use Omeka\Stdlib\Message;

// ...
    function exampleAction()
    {
        $message = new Message('There are %s results.', $results); // @translate
        $this->messenger()->addSuccess($message);
    }
?>
```

The resulting Message object can be passed directly to `translate()` in a view. This
pattern is used often for error messages that can originate from deep within backend code
but still need to be presented to the user, and therefore translated.

## Updating translation files

Omeka S includes several Gulp tasks for working with the translation files. The tasks all
start with `i18n` (an abbreviation for "internationalization"). In addition to having Gulp
set up, these tasks require the `gettext` package, which provides commands such as `msgfmt`
and `xgettext` that are used to work with the translation files.

`gulp i18n:template` updates the core's `.pot` translation template. When strings are added
or changed, this task updates the template so it can be pushed to translators to in turn
update their translations. The Omeka team runs this command and then pushes the result to
Transifex.

`gulp i18n:compile` is run after updating `.po` files to compile them into `.mo` files. Just
updating the `.po` translation file isn't enough to get Omeka S to pick up the changes; the
`.mo` file is what's actually used. The Omeka team runs this command after pulling down
updates from Transifex.

For further commands used for working with modules rather than the core, see the
[documentation on internationalization in modules](../modules/internationalization.md).

## Internationalizing your module

The Omeka S core contains tools for internationalizing the strings in your modules to create
`template.pot` files suitable for adding to Transifex or any other translating service, and
for handling translated `.po` files.

For general information about the Omeka S translation system, see the
[main Internationalization page](../key_concepts/internationalization.md).

## Module preparation

To prepare your module for translation, you need to prepare the strings themselves
throughout your code, and also create and configure a directory that will hold the
translation files.

For handling the strings, the same guidelines as given in the
[main documentation on making strings translatable](../key_concepts/internationalization.md#making-strings-translatable) apply to modules as well.

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

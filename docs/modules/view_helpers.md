---
title: View Helpers
---

View helpers are used to consolidate code for creating or modifying HTML, simplifying
your views and minimizing redundancy. For example, tasks such as escaping HTML,
formatting dates, and creating commonly-used interface components are handled by
way of view helpers.

For a more complete introduction, read [Zend Framework's documentation](https://docs.zendframework.com/zend-view/helpers/intro/).
This page focuses on using Omeka S's view helpers, listed below. Note that Omeka
S also uses many of Zend Framework's native view helpers -- consult their documentation
for more information.

## Using A View Helper

Helpers are typically called from view scripts. A common example using one of Zend
Framework's view helpers is the `Url`, helper, which generates a URL for a route
and action:

```php
$this->url(null, ['action' => 'log'], true);
```

Note that while view helper names begin with an uppercase letter, the method invoked
from `$this` (the View object) begins with a lowercase letter. Each helper will
use its own signature, so consult each helper's documentation for its `__invoke`
method.

## Creating A View Helper In A Module

View Helpers should be placed here in the module's directory structure:

```
MyModule/
  src/
    View/
      Helper/
        MyHelper.php
```

View Helpers inherit from `Zend\View\Helper\AbstractHelper` and implement `Zend\View\Helper\HelperInterface`.

Thus, the basic structure is

```php
namespace MyModule\View\Helper;

use Zend\View\Helper\AbstractHelper;

class MyModuleViewHelper extends AbstractHelper
{
    public function __invoke()
    {
        $view = $this->getView(); // supplied by AbstractHelper
        //return HTML;
}

```

There are no restrictions on the signature for `__invoke`, so any necessary data
from the view can be added as needed, as in the first example of `Url` above. Its
`__invoke` method thus looks like:

```php


```


### Config file

For Omeka S to be aware of your View Helper, you must add it to your module's `config/module.config.php`
array. That file will contain a great deal more than this information, but this
is what will be relevant to the helper:

```php
return [
    'view_helpers' => [
        'invokables' => [
            'myModule' => 'MyModule\View\Helper\MyModuleViewHelper',
        ],
    ],
]
```

The `invokables` key signals that the View Helper class can be directly instantiated
(see below on invokables vs factories). Each value in the subsequent array refers
to the domain-specific class to refer to.

### Invokables vs Factories

Sometimes, a helper will need access to additional services, or data that is accessible
only via a service. In this case, the View Helper must be created via a factory,
rather than an invokable, as defined in the `config.php` file. (See also [Services and Factories](../configuration/services_and_factories.md)).

To create a factory for your View Helper, put the factory in the following directory:

```php
MyModule/
  src/
    Service/
      ViewHelper/
        MyModuleViewHelperFactory.php
```

Instead of declaring an `invokable` in `module.config.php`'s array, declare a factory:

```php
return [
    'view_helpers' => [
        'factories'  => [
            'myModule' => 'MyModule\Service\ViewHelper\MyModuleViewHelperFactory',
        ],
    ],
];
```

The file structure for the factory will be akin to:

```php
namespace MyModule\Service\ViewHelper;

use MyModule\View\Helper\ViewHelper;
use Zend\ServiceManager\Factory\FactoryInterface;
use Interop\Container\ContainerInterface;

class MyModuleViewHelperFactory implements FactoryInterface
{
    public function __invoke(ContainerInterface $services, $requestedName, array $options = null)
    {
        $config = $services->get('Config');
        $mediaAdapters = $config['my_module_media_adapters'];
        return new MyModuleViewHelper($services->get('Omeka\MediaIngesterManager'), $mediaAdapters);
    }
}
```
In this example, the View Helper needs information added to the config array, and
`Omeka\MediaIngesterManager` service. Those are not directly available within a
ViewHelper, and so the factory is used to inject them into the ViewHelper.

As such, the View Helper's `__construct` method must deal with the data

```php
namespace MyModule\View\Helper;

use Zend\View\Helper\AbstractHelper;

class ViewHelper extends AbstractHelper
{
    protected $mediaIngester;

    protected $mediaAdapters;

    public function __construct($mediaIngestManager, $mediaAdapters)
    {
        $this->mediaAdapters = $mediaAdapters;
        $this->mediaIngester = $mediaIngestManager;
    }

    public function __invoke()
    {
        $mediaForms = [];
        foreach ($this->mediaIngester->getRegisteredNames() as $ingester) {
            if (array_key_exists($ingester, $this->mediaAdapters)) {
                $mediaForms[$ingester] = [
                    'label' => $this->mediaIngester->get($ingester)->getLabel(),
                ];
            }
        }
}
```

### Using Partials Within View Helpers

A common tactic in Omeka S is to invoke a View Helper to create HTML content, but
also for that helper itself to use a view page. That is achieved by using a partial
within the helper.

These usualy appear in a `common` directory within the plugin:

```
MyModule/
  views/
    common/
      my-module-view-helper-partial.phtml
```

A shorter, more readable name is in order for the readability of a real module.

A View Helper might then create its HTML like so:

```php

<?php
namespace MyModule\View\Helper;

use Zend\View\Helper\AbstractHelper;

class MyModuleViewHelper extends AbstractHelper
{
    protected $user;

    public function __construct($user)
    {
        $this->user = $user;
    }

    public function __invoke()
    {
        $userRole = $this->user->getRole();
        return $this->getView()->partial(
            'common/my-module-view-helper-partial',
            [
                'userRole' => $userRole,
            ]
        );
    }
}
```

The `__invoke` method then depends upon the partial in `common` to produce the HTML.
The second parameter also passes along a `$userRole` variable to the partial for
it to use. Depending on the needs of the module, that might or might not be needed.

## Built-in View Helpers

Omeka S comes with quite a few view helpers:

- **api**: iew helper for direct access to API read and search operations.
- **assetUrl**: View helper for returning a path to an asset.
- **blockAttachmentsForm**: View helper for rendering a form for adding/editing block attachments.
- **blockLayout**: View helper for rendering block layouts.
- **blockShowTitleSelect**: View helper for rendering an attachment title display select element.
- **blockThumbnailTypeSelect**: View helper for rendering a thumbnail type select element.
- **cancelButton**: View helper for rendering a cancel button.
- **ckEditor**: View helper for loading scripts necessary to use CKEditor on a page.
- **dataType**: View helper for rendering data types.
- **deleteConfirm**: View helper for rendering the delete confirm partial.
- **filterSelector**: View helper for rendering a browse filtering form.
- **formAsset**: get the asset form element
- **formCkeditor**: get the Ckeditor form element
- **formCkeditorInline**: get the Ckeditor inline form element
- **formColorPicker**: get the color picker form element
- **formRecaptcha**: get the Recaptcha form elemnt
- **formRestoreTextarea**: get the restore textarea form element
- **htmlElement**: View helper for rendering a HTML element.
- **hyperlink**: View helper for rendering a HTML hyperlink.
- **i18n**: View helper for rendering localized data.
- **itemSetSelect**: View helper for rendering a select menu containing all item sets.
- **itemSetSelector**: View helper for rendering the item set selector form control.
- **jsTranslate**: View helper for rendering translations for JavaScript strings.
- **lang**: View helper for getting a BCP 47-compliant value for the lang attribute.
- **logger**: View helper for getting the Zend logger.
- **media**: View helper for rendering media.
- **messages**: View helper for proxing the messenger controller plugin.
- **navigationLink**: View helper for rendering a navigation links.
- **pageTitle**: View helper for rendering a title heading for a page.
- **pagination**: View helper for rendering pagination.
- **params**: View helper for getting params from the request.
- **passwordRequirements**: View helper for rendering the password requirements.
- **propertySelect**:  View helper for rendering a select menu containing all properties.
- **propertySelector**: View helper for rendering the property selector.
- **queryToHiddenInputs**: View helper for building a hidden form input for every query in the URL query string
- **resourceClassSelect**: View helper for rendering a select menu containing all resource classes.
- **resourceSelect**: View helper for rendering a select menu containing all of some resource.
- **resourceTemplateSelect**: View helper for rendering a select menu containing all resource templates.
- **roleSelect**: View helper for rendering a select menu containing all roles.
- **searchFilters**: View helper for rendering search filters.
- **searchUserFilters**: View helper for rendering search user filters.
- **sectionNav**: View helper for rendering section navigation.
- **setting**: View helper for getting settings.
- **sitePagePagination**: View helper for rendering site page pagination.
- **siteSelect**: View helper for rendering a select menu containing all sites.
- **siteSetting**: View helper for getting site settings.
- **sortLink**: View helper for rendering a sortable link.
- **sortSelector**: View helper for rendering a sorting form.
- **status**: View helper for getting MVC status.
- **themeSetting**: View helper for getting theme settings.
- **themeSettingAssetUrl**:  View helper for getting a path to a theme setting asset.
- **thumbnail**: View helper for rendering a thumbnail image.
- **trigger**: View helper for triggering a view event.
- **uploadLimit**: View helper for rendering the upload size limit.
- **userBar**: View helper for rendering the user bar.
- **userIsAllowed**: View helper for authorizing the current user.
- **userSelect**: View helper for rendering a select menu containing all users.
- **userSelector**: View helper for rendering the user selector.
- **userSetting**: View helper for getting user settings.

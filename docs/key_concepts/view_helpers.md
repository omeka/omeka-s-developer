# View Helpers

View helpers are used to consolidate code for creating or modifying HTML, simplifying your views and minimizing redundancy. For example, tasks such as escaping HTML, formatting dates, and creating commonly-used interface components are handled by way of view helpers. For a more complete introduction, read [Zend Framework's documentation](https://docs.zendframework.com/zend-view/helpers/intro/). This page focuses on using Omeka S's view helpers, listed below. Note that Omeka S also uses many of Zend Framework's native view helpers -- consult their documentation for more information.

## Using A View Helper

Helpers are typically called from view scripts. A common example using one of Zend Framework's view helpers is the `Url`, helper, which generates a URL for a route and action:

```php
$this->url(null, ['action' => 'log'], true);
```

Note that while view helper names begin with an uppercase letter, the method invoked from `$this` (the View object) begins with a lowercase letter. Each helper will use its own signature, so consult each helper's documentation for its `__invoke` method.

### From outside a view

Use the `getView()` method to find the view object,

```php
$view = $this->getView();
```

then proceed in the same way. Note that, depending on the class, the `getView` method might or might not be available.

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

### Config file

For Omeka S to be aware of your View Helper, you must add it to your module's `config/module.config.php` array. That file will contain a great deal more than this information, but this is what will be relevant to the helper:

```php
return [
    'view_helpers' => [
        'invokables' => [
            'myModule' => 'MyModule\View\Helper\MyModuleViewHelper',
        ],
    ],
]
```

The `invokables` key signals that the View Helper class can be directly instantiated (see below on invokables vs factories). Each value in the subsequent array refers to the domain-specific class to refer to.

### Invokables vs Factories

Sometimes, additional data beyond the View object must be passed to the helper. In this case, the View Helper must be created via a factory, rather than an invokable, as defined in the `config.php` file.

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
        $auth = $services->get('Omeka\AuthenticationService');
        $user = $auth->getIdentity();
        return new ViewHelper($user);
    }
}
```
In this example, the View Helper needs information about the currently logged in user, so it passes along that data to the View Helper.

As such, the View Helper's `__construct` method must deal with it:

```php
namespace MyModule\View\Helper;

use Zend\View\Helper\AbstractHelper;

class ViewHelper extends AbstractHelper
{
    protected $user;

    public function __construct($user)
    {
        $this->user = $user;
    }

    public function __invoke()
    {
        $userRole = $this->user->getRole();
        // return HTML based on the user
    }
}
```

### Using Partials Within View Helpers

A common tactic in Omeka S is to invoke a View Helper to create HTML content, but also for that helper itself to use a view page. That is achieved by using a partial within the helper.

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
            'my-module-view-helper-partial',
            [
                'userRole' => $userRole,
            ]
        );
    }
}
```

The `__invoke` method then depends upon the partial in `common` to produce the HTML. The second parameter also passes along a `$userRole` variable to the partial for it to use. Depending on the needs of the module, that might or might not be needed.


## Omeka S View Helpers

* [Api](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/Api.php)
Helper for direct access to API read and search operations.
* [AssetUrl](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/AssetUrl.php)
* [BlockAttachmentsForm](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/BlockAttachmentsForm.php)
* [BlockLayout](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/BlockLayout.php)
* [BlockShowTitleSelect](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/BlockShowTitleSelect.php) Render an attachment title display select element.
* [BlockThumbnailTypeSelect](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/BlockThumbnailTypeSelect.php)
* [CkEditor](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/CkEditor.php)
* [DataType](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/DataType.php)
* [DeleteConfirm](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/DeleteConfirm.php)
* [HtmlElement](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/HtmlElement.php)
* [Hyperlink](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/Hyperlink.php) Render an HTML hyperlink.
* [I18n](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/I18n.php)
* [ItemSetSelect](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/ItemSetSelect.php) A select menu containing all item sets.
* [ItemSetSelector](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/ItemSetSelector.php) Return the item set selector form control.
* [JsTranslate](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/JsTranslate.php) Provide translations for JavaScript strings.
* [Media](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/Media.php)
* [Messages](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/Messages.php) Helper to proxy the messenger controller plugin.
* [NavigationLink](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/NavigationLink.php)
* [PageTitle](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/PageTitle.php) Render a title heading for a page.
* [Pagination](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/Pagination.php)
* [Params](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/Params.php) Helper to get params from the request.
* [PropertySelector](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/PropertySelector.php) 
* [PropertySelect](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/PropertySelect.php) A select menu containing all properties.
* [QueryToHiddenInputs](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/QueryToHiddenInputs.php) Build a hidden form input for every query in the URL query string.
* [ResourceClassSelect](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/ResourceClassSelect.php) A select menu containing all resource classes.
* [ResourceSelect](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/ResourceSelect.php) A select menu containing all of some resource.
* [SearchFilters](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/SearchFilters.php)
* [SectionNav](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/SectionNav.php) Helper that renders section navigation.
* [Setting](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/Setting.php) Helper to get settings from the settings service.
* [SitePagePagination](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/SitePagePagination.php)
* [SortLink](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/SortLink.php)
* [SortSelector](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/SortSelector.php)
* [ThemeSettingAssetUrl](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/ThemeSettingAssetUrl.php) Return a path to a theme setting asset.
* [ThemeSetting](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/ThemeSetting.php) Helper to get theme settings.
* [Trigger](https://github.com/omeka/omeka-s/blob/develop/application/src/View/HeUlper/Trigger.php)
* [UploadLimit](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/UploadLimit.php)
* [UserIsAllowed](https://github.com/omeka/omeka-s/blob/develop/application/src/View/Helper/UserIsAllowed.php) Authorize the current user.

# View Helpers

View helpers are used to consolidate code for creating or modifying HTML, simplifying
your views and minimizing redundancy. For example, tasks such as escaping HTML,
formatting dates, and creating commonly-used interface components are handled by
way of view helpers.

For a more complete introduction, read [Laminas Framework's documentation](https://docs.laminas.dev/laminas-view/v2/helpers/intro/).
This page focuses on using Omeka S's view helpers, listed below. Note that Omeka
S also uses many of Laminas Framework's native view helpers -- consult their
documentation for more information.

## Using A View Helper

Helpers are typically called from view scripts. A common example using one of Laminas
Framework's view helpers is the `Url`, helper, which generates a URL for a route
and action:

```php-inline
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

View Helpers inherit from `Laminas\View\Helper\AbstractHelper` and implement `Laminas\View\Helper\HelperInterface`.

Thus, the basic structure is

```php-inline
namespace MyModule\View\Helper;

use Laminas\View\Helper\AbstractHelper;

class MyModuleViewHelper extends AbstractHelper
{
    public function __invoke()
    {
        $view = $this->getView(); // supplied by AbstractHelper
        ...
        //return HTML;
    }
}

```

There are no restrictions on the signature for `__invoke`, so any necessary data
from the view can be added as needed, as in the first example of `Url` above. Its
`__invoke` method thus looks like:

```php-inline
public function __invoke($name = null, $params = [], $options = [], $reuseMatchedParams = false)
```


### Config file

For Omeka S to be aware of your View Helper, you must add it to your module's `config/module.config.php`
array. That file will contain a great deal more than this information, but this
is what will be relevant to the helper:

```php-inline
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

```
MyModule/
  src/
    Service/
      ViewHelper/
        MyModuleViewHelperFactory.php
```

Instead of declaring an `invokable` in `module.config.php`'s array, declare a factory:

```php-inline
return [
    'view_helpers' => [
        'factories'  => [
            'myModule' => 'MyModule\Service\ViewHelper\MyModuleViewHelperFactory',
        ],
    ],
];
```

The file structure for the factory will be akin to:

```php-inline
namespace MyModule\Service\ViewHelper;

use MyModule\View\Helper\ViewHelper;
use Laminas\ServiceManager\Factory\FactoryInterface;
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

```php-inline
namespace MyModule\View\Helper;

use Laminas\View\Helper\AbstractHelper;

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

use Laminas\View\Helper\AbstractHelper;

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

### api

Direct access to API read and search operations.

The `api()` helper takes no arguments, but allows you to call several chained
methods which do:

- `$this->api()->read()`: Get a single resource by ID
- `$this->api()->search()`: Get an array of resources matching search parameters
- `$this->api()->searchOne()`: Get a single resource matching search parameters

See [the PHP API docs](../api/php_api.md) for more information about using the
API.

### assetUrl

Return a path to a static asset (CSS/JS/etc.).

The helper takes 5 arguments, but only the first is required:

- `$file`: (string) The filename of the asset to load (the part of the path
after the "asset" folder of the core or module).
- `$module`: (string) The module to load the asset from. If omitted or null,
loads from the current theme. The special module name `Omeka` refers to the
core.
- `$override`: (bool, default false) Whether to allow the theme to override or
replace this asset. If true is passed here and the theme contains an asset of
the same name, it will be used instead of the module or core's copy. This only
makes sense to pass if `$module` is passed.
- `$versioned`: (bool, default true) By default Omeka S will append a query
string `?v=` with the version number of the theme, module or core for
cache-busting purposes. To disable this, pass false here.
- `$absolute` (bool, default false) Pass true here to generate an absolute
instead of relative URL. (Added in 4.0.0)

### blockAttachmentsForm

Render a form for adding/editing block attachments.

This is intended for use only in implementing the admin form of site page
blocks. See [the page blocks documentation](page_blocks.md) for more
information.

Takes 3 arguments, the first is required:

- `$block`: (SitePageBlockRepresentation) The current block (can be null in the
case of blocks that haven't been added yet).
- `$itemOnly`: (bool, default false) Whether attachments are only items,
skipping the options for choosing media and captions.
- `$itemQuery`: (array, default empty) Search query to filter the items that
can be selected.

### blockLayout

Helper for rendering block layouts.

This is mostly used internally for the site page public and admin pages.

Functionality is present on chained method calls:

- `getLayouts()`: Get an array of all registered layout names
- `getLaboutLabel($layout)`: Get the label for a given layout
- `prepareForm()`: Run all layouts' `prepareForm` methods
- `forms($sitePage)`: Render all forms for blocks on a page
- `form()`: Render a block form. Either pass just a SitePageBlockRepresentation
to render a form for an existing block, or ($layout, $site, $page) to render
a form for a new block. Override the partial used to render the form by passing
a partial name as the 4th argument.
- `prepareRender($layout)`: Run a layout's `prepareRender` method
- `render($block)`: Render a block.

### blockShowTitleSelect

Render an attachment title display select element.

See [the page blocks documentation](page_blocks.md) for more
information.

Takes one argument:

- `$block`: (SitePageBlockRepresentation) The current block (can be null in the
case of blocks that haven't been added yet).

### blockThumbnailTypeSelect

Render a thumbnail type select element.

See [the page blocks documentation](page_blocks.md) for more
information.

Takes one argument:

- `$block`: (SitePageBlockRepresentation) The current block (can be null in the
case of blocks that haven't been added yet).

### browse

(Added in 4.0.0)

This helper collects methods useful for rendering browse pages, in particular
for using the admin browse page configuration added in Omeka S 4.0.0.

Functionality is available through chained method calls:

- `renderSortSelector($resourceTypeOrSortConfig)`: Render the sorting selection
form for a browse page. You can pass a string naming the resource type for the
current browse page (e.g., `'items'`), or an array of sorting options, as
`value => label` pairs where both are strings.

    This is the only method of this helper typically used on public pages.

    In 4.0+, it's generally recomended to use this method instead of the
    [sortSelector](#sortselector) helper.

- `renderHeaderRow($resourceType)`: Render the header of a browse page table
  according to the user's configured preferences.
- `renderContentRow($resourceType)`: Render a body row of a browse page table
  according to the user's configured preferences.

The other methods on the helper are used internally and to handle the interface
for users to configure their browse preferences, and won't commonly be used by
themes or modules.

### cancelButton

Render a cancel button.

```php
echo $this->cancelButton();
```

### ckEditor

Load scripts necessary to use CKEditor on a page.

```php
$this->ckEditor();
```

### currentSite

(Added in 4.0.0)

Get the SiteRepresentation object for the current site.

```php
$site = $this->currentSite();
```

### dataType

Helper for rendering data types.

### deleteConfirm

Render the delete confirm partial.

### filterSelector

Render a browse filtering form.

### htmlElement

Render a HTML element.

### hyperlink

Render a HTML hyperlink.

### i18n

Render localized data.

### iiifViewer

(Added in 4.0.0)

Render the Omeka S IIIF viewer in an `<iframe>`.

Takes 2 arguments:

- `$query` (array) The GET query to pass to the IIIF viewer page
- `$options`, (array, default empty) Options for the iframe. Currently accepts
  `width` and `height` keys.

### itemSetSelect

Render a select menu containing all item sets.

### itemSetSelector

Render the item set selector form control.

### jsTranslate

Render translations for JavaScript strings.

### lang

Get a BCP 47-compliant value for the lang attribute.

### lightGalleryOutput

(Added in 4.0.0)

Render a lightGallery viewer for presenting media.

The helper takes a single argument, a nested array of data used to render the
viewer.

For an item, this should usually be used in conjunction with the
[sortMedia](#sortmedia) helper, which will provide the proper array to pass to
this helper.


```php-inline
$itemMedia = $item->media();
$sortedMedia = $this->sortMedia($itemMedia);
if (isset($sortedMedia['lightMedia'])):
    echo $this->lightGalleryOutput($sortedMedia['lightMedia']);
endif;
```

To render a single media, it can be passed as `[['media' => $media]]`:

```php-inline
echo $this->lightGalleryOutput([['media' => $media]]);
```

### logger

Get the logger.

### media

Render media.

### messages

Render "flash" messages stored on the user session.

### navigationLink

Helper for rendering links for the site admin navigation form.

### pageTitle

Render a title heading for a page.

### pagination

Render pagination.

### params

Get params from the request.

### passwordRequirements

Render information about password requirements.

### propertySelect

Render a select menu containing all properties.

### propertySelector

Render the property selector.

### queryToHiddenInputs

Render a hidden form input for every query in the URL query string

### resourceClassSelect

Render a select menu containing all resource classes.

### resourcePageBlocks

(Added in 4.0.0)

This helper is used to work with the configurable resource page blocks feature
for themes added in Omeka S 4.0.0.

This is intended for use on the show pages for items, item sets, and media in
themes that support the resource page blocks feature.

The helper takes two arguments:

- `$resource`: (a Representation object for a resource) the resource that is
  being displayed. This must be an item, item set, or media currently.
- `$regionName`: (string, default `'main'`) The region of the page being
  rendered.

This helper's functionality is available through chained method calls:

- `getBlocks()`: get the configured content for the page region
- `hasBlocks()`: return whether the page region has any blocks configured in it

A typical theme would render the main region as a simple call to `getBlocks`:

```php-inline
echo $this->resourcePageBlocks($item)->getBlocks();
```

A region with wrapper markup that should only render if the region has content
would use `hasBlocks` as well:

```html+php
<?php if ($this->resourcePageBlocks($item, 'sidebar')->hasBlocks()): ?>
<div class="sidebar">
    <?php echo $this->resourcePageBlocks($item, 'sidebar')->getBlocks(); ?>
</div>
<?php endif; ?>
```

### resourceSelect

Render a select menu containing all of some resource.

### resourceTemplateSelect

Render a select menu containing all resource templates.

### roleSelect

Render a select menu containing all roles.

### searchFilters

Render a list of active search filters.

### searchUserFilters

Render a list of active search filters for users browse.

### sectionNav

Render section navigation.

### setting

Get global settings.

### sitePagePagination

Render site page pagination.

### siteSelect

Render a select menu containing all sites.

### siteSelector

Render the site selector.

### siteSetting

Get site settings.

### sortLink

Render a sortable link.

### sortMedia

(Added in 4.0.0)

Sort media between those usable in the lightGallery viewer and those not.

Takes a single argument, an array of media.

Returns an array with two top-level keys:

- `'lightMedia'` is an array of media supported by lightGallery, suitable to be
  passed to the [lightGalleryOutput](#lightgalleryoutput) helper.
- `'otherMedia'` is an array of media not supported by lightGallery. The intent
  is for themes to loop through this array and render the unsupported media in
  an alternative way, by listing them or rendering them directly, without the
  viewer.

### sortSelector

Render a sorting form.

### status

Get MVC status.

### themeSetting

Get theme settings.

### themeSettingAsset

(Added in 3.1.0)

Get the representation object for a theme setting asset.

### themeSettingAssetUrl

Get a URL to a theme setting asset.

### thumbnail

Render a thumbnail image.

### trigger

Trigger a view event.

### uploadLimit

Render the upload size limit.

### userBar

Render the public-side user bar.

### userIsAllowed

Check if the current user has a permission.

### userSelect

Render a select menu containing all users.

### userSelector

Rendering the user selector.

### userSetting

Get user settings.

## Form Element Helpers

These helpers are used in conjunction with form elements.

- **formAsset**: get the asset form element
- **formCkeditor**: get the Ckeditor form element
- **formCkeditorInline**: get the Ckeditor inline form element
- **formColorPicker**: get the color picker form element
- **formRecaptcha**: get the Recaptcha form element
- **formQuery**: get the form element for composing search queries (Added in 3.1.0)
- **formRestoreTextarea**: get the restore textarea form element

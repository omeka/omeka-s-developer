# Faceted Browse

## Custom facets

Modules can extend this module to add their own custom facets. To demonstrate this let's create a facet type named "My facet" that adds a way to filter by class. Filtering by class is already implemented in the "By class" facet type, but it serves as a good example of how to stitch the various parts together. First, register the facet type in your module's configuration.

```php

'faceted_browse_facet_types' => [
    'factories' => [
        'my_facet' => \MyModule\Service\FacetType\MyFacetFactory::class,
    ],
],
```

We are using a factory here because we will need to inject the form element manager into the facet type. This will make building form elements much easier.

```php
<?php
namespace MyModule\Service\FacetType;

use MyModule\FacetType\MyFacet;
use Laminas\ServiceManager\Factory\FactoryInterface;
use Interop\Container\ContainerInterface;

class MyFacetFactory implements FactoryInterface
{
    public function __invoke(ContainerInterface $services, $requestedName, array $options = null)
    {
        return new MyFacet($services->get('FormElementManager'));
    }
}
```

Then we make the facet type itself.

```php
<?php
namespace MyModule\FacetType;

use FacetedBrowse\Api\Representation\FacetedBrowseFacetRepresentation;
use FacetedBrowse\FacetType\FacetTypeInterface;
use Laminas\Form\Element;
use Laminas\ServiceManager\ServiceLocatorInterface;
use Laminas\View\Renderer\PhpRenderer;

class MyFacet implements FacetTypeInterface
{
    protected $formElements;

    public function __construct(ServiceLocatorInterface $formElements)
    {
        $this->formElements = $formElements;
    }

    /**
     * Get the label of this facet type.
     *
     * @return string
     */
    public function getLabel() : string
    {
        return 'My facet'; // @translate
    }

    /**
     * Get the resource types that can use this facet type.
     *
     * @return array
     */
    public function getResourceTypes() : array
    {
        return ['items', 'item_sets', 'media'];
    }

    /**
     * Get the maximum amount of this facet type for one category.
     *
     * @return ?int
     */
    public function getMaxFacets() : ?int
    {
        return null;
    }

    /**
     * Prepare the data form of this facet type.
     *
     * @param PhpRenderer $view
     */
    public function prepareDataForm(PhpRenderer $view) : void
    {
        $view->headScript()->appendFile($view->assetUrl('js/facet-data-form/my-facet.js', 'MyModule'));
    }

    /**
     * Render the data form of this facet type.
     *
     * @param PhpRenderer $view
     * @param array $data
     * @return string
     */
    public function renderDataForm(PhpRenderer $view, array $data) : string
    {
        // Class IDs
        $classIds = $this->formElements->get(OmekaElement\ResourceClassSelect::class);
        $classIds->setName('class_ids');
        $classIds->setValue($data['class_ids'] ?? []);
        $classIds->setOptions([
            'label' => 'Classes', // @translate
            'empty_option' => '',
        ]);
        $classIds->setAttributes([
            'id' => 'my-facet-class-ids',
            'data-placeholder' => 'Select classes…', // @translate
            'multiple' => true,
        ]);
        return $view->partial('common/faceted-browse/facet-data-form/my-facet', [
            'classIds' => $classIds,
        ]);
    }

    /**
     * Prepare the render of this facet type.
     *
     * @param PhpRenderer $view
     */
    public function prepareFacet(PhpRenderer $view) : void
    {
        $view->headScript()->appendFile($view->assetUrl('js/facet-render/my-facet.js', 'MyModule'));
    }

    /**
     * Render the markup for this facet type.
     *
     * @param PhpRenderer $view
     * @param FacetedBrowseFacetRepresentation $facet
     * @return string
     */
    public function renderFacet(PhpRenderer $view, FacetedBrowseFacetRepresentation $facet) : string
    {
        $classes = [];
        $classIds = $facet->data('class_ids', []);
        foreach ($classIds as $classId) {
            $class = $view->api()->read('resource_classes', $classId)->getContent();
            $classes[] = $class;
        }
        return $view->partial('common/faceted-browse/facet-render/my-facet', [
            'facet' => $facet,
            'classes' => $classes,
        ]);
    }
}
```

As you see, a facet type sets your facet's label, sets the types of resources that can use this facet type, sets the maximum number of facets allowed on one page, prepares and renders your facet type's administrative data form, and prepares and renders your facet type's public interface.

Note that facet types must correspond to an existing search query that the API recognizes. For example, the "Full-text" facet type corresponds to the `fulltext_search=foo` query, and the "By class" facet type corresponds to the `resource_class_id[]=123` query. If you want to create a facet type that does not have a corresponding query, you'll first have to implement that query using the `api.search.query` event (`Omeka\Api\Adapter\ItemAdapter` identifier).

The `getResourceTypes()` method defines the types of resources that can use the facet type. Omeks S has three resource types: Items, Item Sets, and Media. Some queries are specific to a particular resource type, so it wouldn't make sense to add their corresponding facet types to some pages. For example, the `item_set_id[]=123` query wouldn't make sense for a categoy belonging to an Item Set page, so excluding the `item_sets` resource type will exclude the ItemSet facet type for Item Set pages.

With `getMaxFacets()` the maximum number of facets per page will depend on your facet's corresponding query. If the query allows for only one search of the same kind, then return 1. If the query allows for one or more search of the same kind, then return null to signify there is no maximum. For example, `fulltext_search=foo` allows only one search; whereas `resource_class_id[]=123` allows any number of searches.

In `prepareDataForm()` you'll prepare the administrative data form of the facet type. All facet types will need to append a JavaScript file that does two things: 1) register a facet add/edit handler and 2) register a facet set handler.

```js
/**
 * Register a callback that handles facet add/edit.
 *
 * "Facet add/edit" happens when a user adds or edits a facet. Use this
 * handler to prepare the facet form for use. The handler will receive no
 * arguments. If needed, facet types should register a handler in a script
 * added in prepareDataForm().
 *
 * @param string facetType The facet type
 * @param function handler The callback that handles facet add/edit
 */
FacetedBrowse.registerFacetAddEditHandler('my_facet', function() {
    $('#my-facet-class-ids').chosen({
        include_group_label_in_selected: true
    });
});

/**
 * Register a callback that handles facet set.
 *
 * "Facet set" happens when a user finishes configuring the facet and sets
 * it. Use this handler to validate the facet form and, if it validates,
 * return the facet data object. If it does not validate, alert the user and
 * return nothing. The handler will receive no arguments. All facet types
 * should register a handler in a script added in prepareDataForm().
 *
 * @param string facetType The facet type
 * @param function handler The callback that handles facet set
 * @return object
 */
FacetedBrowse.registerFacetSetHandler('my_facet', function() {
    return {
        class_ids: $('#my-facet-class-ids').val()
    };
});
```

The idea here is that most facet types will need additional configuration options. (This is in addition to "Facet name," which the module applies automatically.) You'll build the part of the form that handles these options in `renderDataForm()` and register the form handlers via a script added in `prepareDataForm()`. Note that while `FacetedBrowse.registerFacetAddEditHandler()` is optional, `FacetedBrowse.registerFacetSetHandler()` is required, even if your facet type does not have additional configuration options. If you facet type has no options, simply return an empty object.

As mentioned above, in `renderDataForm()` you'll build and return the form markup needed to administer the data type. The markup should work in conjunction with with the handlers you added in `prepareDataForm()`. Make sure to use `id` or `class` attributes in your form markup so your handlers can identify the key components. We recommend building your form elements using the form element manager, and using a partial template to render the markup.

```php
<?php echo $this->formRow($classIds); ?>
```

Now that you have the code needed to administer the facet type, you'll need to write to code needed to render and enable the facet on the public faceted browse page. This is done using `prepareFacet()` and `renderFacet()`.

In `prepareFacet()`, you'll prepare the render of this facet type. All facet types will need to append a JavaScript file that does two things: 1) register a facet apply state handler and 2) detect a user interaction with the facet and act accordingly.

```js
/**
 * Register a callback that handles facet apply state.
 *
 * "Facet apply state" happens when the user navigates to a page, whether it has
 * been interacted with or not. Use this handler to apply a previously saved
 * state to a facet, and to generally prepare the facet for use. The handler
 * will receive a facet container element as the first argument and the facet's
 * state as the second argument. Note that the facet's state may be undefined if
 * the user has not interacted with the facet. All facet types should register a
 * handler in a script added in prepareFacet().
 *
 * @param string facetType The facet type
 * @param function handler The callback that handles facet apply state
 */
FacetedBrowse.registerFacetApplyStateHandler('my_facet', function(facet, facetState) {
    const thisFacet = $(facet);
    facetState = facetState ?? [];
    facetState.forEach(function(classId) {
        thisFacet.find(`input.my-facet[data-class-id="${classId}"]`)
            .prop('checked', true)
            .addClass('selected');
    });
});

$(document).ready(function() {

const container = $('#container');

container.on('click', '.resource-class', function(e) {
    const thisClass = $(this);
    const facet = thisClass.closest('.facet');
    const queries = [];
    const state = [];
    facet.find('.my-facet').not(thisClass).removeClass('selected');
    thisClass.prop('checked', !thisClass.hasClass('selected'));
    thisClass.toggleClass('selected');
    facet.find('.my-facet.selected').each(function() {
        const id = $(this).data('classId');
        queries.push(`resource_class_id=${id}`);
        state.push(id);
    });
    FacetedBrowse.setFacetState(facet.data('facetId'), state, queries.join('&'));
    FacetedBrowse.triggerStateChange();
});

});
```

The idea here is that all facet types need a facet control that looks and behaves in a certain way for the end user. You'll build the control in `renderFacet()` and control its behavior via a script added in `prepareFacet()`. Use `FacetedBrowse.registerFacetApplyStateHandler()` to apply a previously saved state to a facet, and to generally prepare the facet for use. This should make the facet "sticky" when a user navigates away from the page and returns via the back button. You must implemnt this handler or the user will lose the previous visual state of the facet.

In addition this this, and most importantly, your script should detect a user interaction, calculate the data needed to preserve the current state of the facet, calculate the API query needed to fetch the items, then set them using `FacetedBrowse.setFacetState(facetId, state, query)`. Afterwards, you must call `FacetedBrowse.triggerStateChange()` to trigger the state change. This will gather all queries from all facets, consolodate them, and update the items on the browse section of the page according to the new state.

As mentioned above, in `renderFacet()` you'll build and return the form markup needed to render the facet. The markup should work in conjunction with with the code you added in `prepareFacet()`. Make sure to use `id` or `class` attributes in your form markup so your script can identify the key components. When possible, we recommend building your form elements using the form element manager, and using a partial template to render the markup.

```php
<ul>
    <?php foreach ($classes as $class): ?>
    <li>
        <label>
            <input
                class="my-facet"
                type="radio"
                name="<?php echo sprintf('my_facet_%s', $facet->id()); ?>"
                data-class-id="<?php echo $this->escapeHtml($class->id()); ?>">
                <?php echo $class->label(); ?>
        </label>
    </li>
    <?php endforeach; ?>
</ul>
```

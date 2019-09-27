---
title: Event Reference
---

Below is a list of events that Omeka triggers during critical moments of operation. Modules may attach listeners to these events and extend Omeka's functionality. For example, in `Module::attachListeners()` you can run custom code after an item is created:

```php
$sharedEventManager->attach(
    'Omeka\Api\Adapter\ItemAdapter',
    'api.create.post',
    function (\Zend\EventManager\Event $event) {
        $response = $event->getParam('response');
        $item = $response->getContent();
        // do something
    }
);
```

We use Zend Framework's EventManager component, so many questions can be answered by reading [the documentation](http://framework.zend.com/manual/current/en/modules/zend.event-manager.event-manager.html).

## API Adapter Events

All classes that extend `Omeka\Api\Adapter\AbstractAdapter` trigger these events. Use the event's `getTarget()` method to get the adapter object, and the `getParam()` method to get the values of parameters, if there are any.

### api.execute.pre

* **request**: The API request.

Triggered before executing any API operation.

### api.execute.post

* **request**: The API request.
* **response**: The API response.

Triggered after executing any API operation. Does not trigger if a non-validation exception is thrown during execution.

### api.*.pre

* **request**: The API request.

Triggered before executing a particular API operation. Replace the asterisk with one of the following operations: `search`, `create`, `read`, `update`, `delete`, `batch_create`, `batch_update`, or `batch_delete`.

### api.*.post

* **request**: The API request.
* **response**: The API response.

Triggered after executing a particular API operation. Replace the asterisk with one of the following operations: `search`, `create`, `read`, `update`, `delete`, `batch_create`, `batch_update`, or `batch_delete`.

## API Entity Adapter Events

All classes that extend `Omeka\Api\Adapter\AbstractEntityAdapter` trigger these events. Use the event's `getTarget()` to get the adapter object.

### api.search.query

* **queryBuilder**: The Doctrine query builder.
* **request**: The API request.

Triggered during an API `search` operation, after the adapter builds the initial query.

### api.find.query

* **queryBuilder**: The Doctrine query builder.
* **request**: The API request.

Triggered anytime an adapter attempts to find an entity.

### api.find.post

* **entity**: The found entity.
* **request**: The API request.

Triggered during the API `read`, `delete`, and `batch_delete` operations, after an entity is found.

### api.hydrate.pre

* **entity**: The un-hydrated entity.
* **request**: The API request.
* **errorStore**: The error store.

Triggered during the API `create`, `update`, `batch_create`, and `batch_update` operations, before validating the request and before entity hydration.

### api.hydrate.post

* **entity**: The hydrated entity.
* **request**: The API request.
* **errorStore**: The error store.

Triggered during the API `create`, `update`, `batch_create`, and `batch_update` operations, after entity hydration and before validating the entity.

## Doctrine Lifecycle Events

All classes that implement `Omeka\Entity\EntityInterface` trigger these events. Use the event's `getTarget()` to get the entity object. These delegate selected [Doctrine lifecycle events](http://doctrine-orm.readthedocs.org/projects/doctrine-orm/en/latest/reference/events.html#lifecycle-events) to Omeka events.

### entity.remove.pre

See the `preRemove` lifecycle event.

### entity.remove.post

See the `postRemove` lifecycle event.

### entity.persist.pre

See the `prePersist` lifecycle event.

### entity.persist.post

See the `postPersist` lifecycle event.

### entity.update.pre

See the `preUpdate` lifecycle event.

### entity.update.post

See the `postUpdate` lifecycle event.

## API Representation Events

Use the event's `getTarget()` to get the representation object.

### rep.resource.json

* **jsonLd**: The JSON-LD array.

All classes that extend `Omeka\Api\Representation\AbstractRepresentation` trigger this event.

Triggered after the serialization of a representation's JSON-LD. To filter the JSON-LD, listeners may modify the `jsonLd` parameter and set it back to the event.

### rep.value.html

* **html**: The value text.

The `Omeka\Api\Representation\ValueRepresentation` class triggers this event. All information about the value is available in the event's target object, including target URL, target ID (for resource value), and label.

Triggered after getting a Value representation's text (for display on a webpage). To filter the text, listeners may modify the `html` parameter and set it back to the event.

### rep.resource.display_values

* **values**: The resource values

All classes that extend `Omeka\Api\Representation\AbstractResourceEntityRepresentation` trigger this event.

Triggered in method `displayValues` to modify the values passed to the partial.

## View Events

The `trigger` view helper triggers these events at strategic locations within view templates. Use the controller's invokable service name as the event identifier. Look for these under `application/config/module.config.php`, under the `controllers` key. It name might be listed under either its `invokable` or `factory` key. For example, instead of the name being `Omeka\Controller\Admin\ItemController`, it could be `Omeka\Controller\Admin\Item`. Use the event's `getTarget()` to get the view renderer. Any markup echoed in listeners will render on page.

### view.layout

Triggered within a view layout. Use this event to add to the HTML head on every page in installation.

### view.show.before

Triggered before show page markup.

### view.show.after

Triggered after show page markup.

### view.browse.before

Triggered before browse page markup.

### view.browse.after

Triggered after browse page markup.

### view.add.before

Triggered before add page markup.

### view.add.after

Triggered after add page markup.

### view.edit.before

Triggered before edit page markup.

### view.edit.after

Triggered after edit page markup.

### view.add.form.before

* **form**: The form being created.

Triggered before add page form markup, within the form.

### view.add.form.after

* **form**: The form being created.

Triggered after add page form markup, within the form.

### view.edit.form.before

* **form**: The form being created.

Triggered before edit page form markup, within the form.

### view.edit.form.after

* **form**: The form being created.

Triggered after edit page form markup, within the form.

### view.show.section_nav

* **section_nav**: An array of section navigation labels, keyed by section ID.

Triggered when creating the section navigation for show pages. To add a section navigation item, modify the **section_nav** parameter and set it back to the event using `setParam('section_nav', $sectionNav)`.

### view.add.section_nav

* **section_nav**: An array of section navigation labels, keyed by section ID.

Triggered when creating the section navigation for add pages. To add a section navigation item, modify the **section_nav** parameter and set it back to the event using `setParam('section_nav', $sectionNav)`.

### view.edit.section_nav

* **section_nav**: An array of section navigation labels, keyed by section ID.

Triggered when creating the section navigation for edit pages. To add a section navigation item, modify the **section_nav** parameter and set it back to the event using `setParam('section_nav', $sectionNav)`.

### view.advanced_search

* **query**: The current query array, if any
* **resourceType**: The type of resource being queried (`item`, `itemSet`, or `media`)
* **partials**: Array of partials representing the parts of the form to render

This event is intended to be used to filter the `partials` parameter: listeners can add, remove, or reorder the partials in the array
to change what is displayed on the form. Any added partial will be passed one parameter when it is rendered: `query`.

### view.manage_resources.before

Triggered before the "Manage resources" panel on the admin Dashboard

### view.manage_resources.after

Triggered after the "Manage resources" panel on the admin Dashboard

### view.manage_sites.before

Triggered before the "Manage sites" panel on the admin Dashboard

### view.manage_sites.after

Triggered after the "Manage sites" panel on the admin Dashboard

### view.details

* **entity** The entity whose details are being displayed

Triggered inside the details display on the admin side.

### view.sort-selector

* **sortBy**: Array of sorting options, each a sub-array with keys "label" and "value" (filterable)
* **sortByQuery**: Query string parameter for current active sort field
* **sortOrderQuery** Query string parameter for current active sort direction

Event for filtering options in the sorting `select` element on browse pages. Triggered in the sortSelector view helper. Added in Omeka S 1.3.0.

### view.search.filters

* **filters**: An array of filtered values, keyed by the filter label
* **query**: The search query

Triggered after creating the human-readable search params for search results.

## View Helper Events

### view_helper.media.render_options

* **options**: Array of options passed to the media renderer (filterable)
* **media**: MediaRepresentation for the media being rendered

Event for filtering options passed to media renderers. Triggered in the Media view helper. Added in Omeka S 1.3.0.

### view_helper.thumbnail.attribs

* **attribs**: Array of HTML attributes for thumbnail `img` tag (filterable)
* **thumbnail**: The URL for the thumbnail
* **primaryMedia**: MediaRepresentation for the media being thumbnailed
* **representation**: Representation object for the representation being thumbnailed (could be, e.g. an item or item set and therefore different from `primaryMedia`)
* **type**: Type of thumbnail being displayed

Event for filtering HTML attributes for thumbnails. Triggered in the Thumbnail view helper. Added in Omeka S 1.2.0.

## User Events

### user.login

Triggered when a user logs in. Target is the User entity that logged in. Added in Omeka S 1.2.0.

### user.logout

Triggered when a user logs out. Added in Omeka S 1.1.0.

## Miscellaneous Events

### js.translate_strings

* **strings**: An array of strings to be translated

Triggered in the invocation of the jsTranslate view helper.

### site_settings.form

* **form**: The site settings form.

Triggered in the site admin settings action. Use the event's `getTarget()` to get the controller object.

### form.add_elements

Triggered within Form objects (usually in forms that produce tabs). Used to directly add form elements to the form.

The target is the Form itself.

### form.add_input_filters

* **inputFilters**: The input filter object for the form.

Triggered with Form objects, after elements have been added, including from the `form.add_elements` event. Used to adjust filters, especially the `required` setting.

The target is the form itself.

### form.vocab_member_select.query

* **query**: The query to filter the select.

All classes that extend `Omeka\Form\Element\AbstractVocabularyMemberSelect` trigger this event, in particular `PropertySelect` and `ResourceClassSelect`.

Allow to filter the query used to limit the list of terms (properties or resources classes) displayed in a `select` html element.

### form.vocab_member_select.value_options

* **value_options**: The query to filter the select.

All classes that extend `Omeka\Form\Element\AbstractVocabularyMemberSelect` trigger this event, in particular `PropertySelect` and `ResourceClassSelect`.

Allow to filter the resulting list of terms (properties or resources classes) displayed in a `select` html element.

### service.registered_names

* **registered_names**: An array of registered names.

Triggered when getting the registered names of certain Omeka services (using `getRegisteredNames()`). Use the event's `getTarget()` to get the service plugin manager. To filter the names, listeners may modify the `registered_names` parameter and set it back to the event.

### api.context

* **context**: The JSON-LD context array

Triggered when visiting the `api-context` route. To add term definitions to the JSON-LD context, listeners may modify the `context` parameter and set it back to the event.

### htmlpurifier_config

* **config**: The HTMLPurifier_Config object (filterable).

Event for filtering the configuration of the HTMLPurifier library. Added in Omeka S 1.2.0.

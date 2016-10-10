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

# API Adapter Events

All classes that extend `Omeka\Api\Adapter\AbstractAdapter` trigger these events. Use the event's `getTarget()` to get the adapter object.

## api.execute.pre

* **request**: The API request.

Triggered before executing any API operation.

## api.execute.post

* **request**: The API request.
* **response**: The API response.

Triggered after executing any API operation. Does not trigger if a non-validation exception is thrown during execution.

## api.*.pre

* **request**: The API request.

Triggered before executing a particular API operation. Replace the asterisk with one of the following operations: `search`, `create`, `batch_create`, `read`, `update`, or `delete`.

## api.*.post

* **request**: The API request.
* **response**: The API response.

Triggered after executing a particular API operation. Replace the asterisk with one of the following operations: `search`, `create`, `batch_create`, `read`, `update`, or `delete`.

# API Entity Adapter Events

All classes that extend `Omeka\Api\Adapter\AbstractEntityAdapter` trigger these events. Use the event's `getTarget()` to get the adapter object.

## api.search.query

* **queryBuilder**: The Doctrine query builder.
* **request**: The API request.

Triggered during an API `search` operation, after the adapter builds the initial query.

## api.find.query

* **queryBuilder**: The Doctrine query builder.
* **request**: The API request.

Triggered anytime an adapter attempts to find an entity.

## api.find.post

* **entity**: The found entity.
* **request**: The API request.

Triggered during the API `read` and `delete` operations, after an entity is found.

## api.hydrate.pre

* **entity**: The un-hydrated entity.
* **request**: The API request.
* **errorStore**: The error store.

Triggered during the API `create`, `batch_create`, and `update` operations, before validating the request and before entity hydration.

## api.hydrate.post

* **entity**: The hydrated entity.
* **request**: The API request.
* **errorStore**: The error store.

Triggered during the API `create`, `batch_create`, and `update` operations, after entity hydration and before validating the entity.

# Doctrine Lifecycle Events

All classes that implement `Omeka\Entity\EntityInterface` trigger these events. Use the event's `getTarget()` to get the entity object. These delegate selected [Doctrine lifecycle events](http://doctrine-orm.readthedocs.org/projects/doctrine-orm/en/latest/reference/events.html#lifecycle-events) to Omeka events.

## entity.remove.pre

See the `preRemove` lifecycle event.

## entity.remove.post

See the `postRemove` lifecycle event.

## entity.persist.pre

See the `prePersist` lifecycle event.

## entity.persist.post

See the `postPersist` lifecycle event.

## entity.update.pre

See the `preUpdate` lifecycle event.

## entity.update.post

See the `postUpdate` lifecycle event.

# API Representation Events

Use the event's `getTarget()` to get the representation object.

## rep.resource.json

* **jsonLd**: The JSON-LD array.

All classes that extend `Omeka\Api\Representation\AbstractRepresentation` trigger this event.

Triggered after the serialization of a representation's JSON-LD. To filter the JSON-LD, listeners may modify the `jsonLd` parameter and set it back to the event.

## rep.value.html

* **html**: The value text.

The `Omeka\Api\Representation\ValueRepresentation` class triggers this event. All information about the value is available in the event's target object, including target URL, target ID (for resource value), and label.

Triggered after getting a Value representation's text (for display on a webpage). To filter the text, listeners may modify the `html` parameter and set it back to the event.

# View Events

The `trigger` view helper triggers these events at strategic locations within view templates. Use the controller's invokable service name as the event identifier. Use the event's `getTarget()` to get the view renderer. Any markup echoed in listeners will render on page.

## view.layout

Triggered within a view layout.

## view.show.after

Triggered after show page markup.

## view.browse.after

Triggered after browse page markup.

## view.add.after

Triggered after add page markup.

## view.edit.after

Triggered after edit page markup.

## view.add.form.after

Triggered after add page form markup, within the form.

## view.edit.form.after

Triggered after edit page form markup, within the form.

## view.show.section_nav

* **section_nav**: An array of section navigation labels, keyed by section ID.

Triggered when creating the section navigation for show pages. To add a section navigation item, modify the **section_nav** parameter and set it back to the event using `setParam('section_nav', $sectionNav)`.

## view.add.section_nav

* **section_nav**: An array of section navigation labels, keyed by section ID.

Triggered when creating the section navigation for add pages. To add a section navigation item, modify the **section_nav** parameter and set it back to the event using `setParam('section_nav', $sectionNav)`.

## view.edit.section_nav

* **section_nav**: An array of section navigation labels, keyed by section ID.

Triggered when creating the section navigation for edit pages. To add a section navigation item, modify the **section_nav** parameter and set it back to the event using `setParam('section_nav', $sectionNav)`.

## view.advanced_search

* **query**: The current query array, if any
* **resourceType**: The type of resource being queried (`item`, `itemSet`, or `media`)

Triggered after advanced search form markup, within the advanced search partial script.

# Miscellaneous Events

## js.translate_strings

* **strings**: An array of strings to be translated

Triggered in the invocation of the jsTranslate view helper.

## site_settings.form

* **form**: The site settings form.

Triggered in the site admin settings action. Use the event's `getTarget()` to get the controller object.

## service.registered_names

* **registered_names**: An array of registered names.

Triggered when getting the registered names of certain Omeka services (using `getRegisteredNames()`). Use the event's `getTarget()` to get the service plugin manager. To filter the names, listeners may modify the `registered_names` parameter and set it back to the event.

## api.context

* **context**: The JSON-LD context array

Triggered when visiting the `api-context` route. To add term definitions to the JSON-LD context, listeners may modify the `context` parameter and set it back to the event.
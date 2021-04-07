# Server Events

Server events (`Omeka\Event`) extend from `Zend\EventManager\Event`. They are triggered from Omeka S's PHP code in a variety of controllers, views and API actions to allow follow-up actions to be done within Omeka S itself or within modules. See [the reference page](server_event_reference.md) for a full list.

## Attaching a listener to an event

The main module class (Module.php) provides a method for easily attaching listeners to events: `attachListeners`.

Events are triggered based on the name of the event, and the class of the object that triggers it. A typical attachment of a listener in a module looks like:

```php
public function attachListeners(SharedEventManagerInterface $sharedEventManager) 
{
    $sharedEventManager->attach(
        'Omeka\Controller\Admin\Item',
        'view.show.after',
        [$this, 'showSource']
    );
}
```

The `attach` method used here takes three arguments:

1. The class identifier. In this case, it is `'Omeka\Controller\Admin\Item'`, meaning to only listen for events from the admin-side item controller.

2. The event identifier. This listener is attaching to the `view.show.after` event, which is fired within the "show" view.

3. The callback for the event. Any PHP callback can be given here, including closures, but the `[$this, 'methodName']` pattern is common for calling other methods in the same module class. Here, the module will use its `showSource` method to add content to the view. An `Event` object is passed to the callback. From that object, inspect the parameters with `$event->getParams()` or `$event->getParam('paramName')` to obtain the data you need.

## Further details

We use Zend Framework's EventManager component, so many questions can be answered by reading [the documentation](http://framework.zend.com/manual/current/en/modules/zend.event-manager.event-manager.html).

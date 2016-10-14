# Events

Omeka S Events (`Omeka\Event`) extend from `Zend\EventManager\Event`. They are triggered in a variety of controllers, views and API actions to allow follow-up actions to be done within Omeka S itself or within modules. See [Events Reference](../reference/events.md) for a full list.

## Attaching a listener to an event

Events are triggered based on the name of the event, and the class of the object that triggers it. A typical attachment of a listener in a module looks like:

```php
    public function attachListeners(SharedEventManagerInterface $sharedEventManager) 
    {
        $sharedEventManager->attach(
                'Omeka\Controller\Admin\Item',
                'view.show.after',
                array($this, 'showSource')
                );
    }

```

There are three parts:

1. The class identifier. In this case, it is `'Omeka\Controller\Admin\Item'`, meaning that the admin-side item controller will fire the event.

2. The event identifier. Here, the view action will fire the `view.show.after` event.

3. The callback for the event. The module will use its `showSource` method to add content to the view. An `Omeka\Event` object is passed along. From that object, inspect the parameters with `$event->getParams()` or `$event->getParam('paramName')` to obtain the data you need.


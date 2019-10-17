---
title: Server Events
---

Server events (`Omeka\Event`) extend from `Zend\EventManager\Event`. They are triggered from Omeka S's PHP code in a variety of controllers, views and API actions to allow follow-up actions to be done within Omeka S itself or within modules. See [the reference page](server_event_reference.md) for a full list.

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

# Event Reference

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

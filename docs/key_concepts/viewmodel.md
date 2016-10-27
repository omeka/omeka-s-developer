# ViewModel

The ViewModel is used to send data to views for display. It is typically created in a controller's action, and returned from there:

```php

namespace MyModule\Controller;

use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;

class IndexController extends AbstractActionController
{
    public function indexAction()
    {
        $view = new ViewModel();
        // set variables for the view
        $view->setVariable('mything', $mything);
        return $view;
    }
}
```

## From an Event

### Adding <head> content to a View

Views are made available via a number of `Event`s. This allows adding to the <head> element of a page in many ways.

For any `view.*` event, the view is the target. Thus, to get the view, a module needs to include code like this:

```php

    public function attachListeners(SharedEventManagerInterface $sharedEventManager)
    {
        $sharedEventManager->attach(
            $identifier,
            'view.show.after',
            array($this, 'addCSS')
        );
    }

    public function addCSS($event)
    {
        $view = $event->getTarget();
        $view->headLink()->appendStylesheet($view->assetUrl('css/mymodule.css', 'MyModule'));
    }
```

The first parameter for `$view->assetUrl()` is a path within the module's `/asset` directory, and the second parameter is the namespace for the module.

Javascript files are added similarly, using 

```php
    $view->headScript()->appendFile($view->assetUrl('js/mymodule.js', 'MyModule'));
```

## From a Module::getConfigForm() function

All of the above applies to working with a module's `getConfigForm` function, but the object passed in is a `Zend\View\Renderer\PhpRenderer` object. The same methods can be used.


## See Also

* [Omeka Events](omeka_events.md)

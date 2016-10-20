# Configuration

Following Zend Framework 3, the configuration file (`application/config/module.config.php`) defines many things that Omeka S depends on, including

* Routes and navigation
* Controllers
* Entities
  * API adapters
* Forms

Modules add their own configuration setting via their own `module.config.php` files. See [Modules](modules.md) for more information. These configuration settings add to, and possibly modify, the core configuration.

## Most important keys

The following keys in configuration are mostly likely to be of interest:

* `logger`
* `router`
* `navigation`

See also [Services and Factories](services_and_factories.md)


## Invocables vs Factories

You will often see subkeys of `invocables` and `factories` in the configuration file. These refer to how Omeka S creates new objects -- either directly ('invocable') or indirectly ('factory'). Factories are used to add data or classes into the object that would otherwise be directly created (i.e., invoced). See [Services and Factories](services_and_factories.md) for more details.

## Accessing the configuration

When a `ServiceLocator` object is available, do 

```php
$config = $this->getServiceLocator()->get('Config');

```

`$config` will be the array of information. In all cases, you should know the key of the information you need.


## See also

* [Services and Factories](services_and_factories.md)
* [Modules](modules.md)

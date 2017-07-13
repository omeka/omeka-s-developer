# Modules

You can extend the functionality of Omeka S by writing an add-on component called a *module.* Zend Framework 2 provides a substantial [framework for writing modules](http://framework.zend.com/manual/2.3/en/modules/zend.module-manager.intro.html), but Omeka S provides extra structure that makes the modules installable, upgradeable, and integratable.

## Directory Structure

At its most basic level, a module directory is structured like so:

```
MyModule/
    Module.php
    config/
        module.ini
        module.config.php
    src/
        <library-directories-and-files>
    assets/
        <asset-directories-and files>/
    view/
        <module-namespace>/
            <controller-directories>/
                <action-template-files>.phtml
        common/
          mymodule-<template-files>.phtml
```

The name of the module directory (`MyModule` above) is significant. It must be a reasonably unique, concise, and descriptive name of your module in CamelCase format.

The `asset` directory contains assets such as JavaScript, CSS, and image files, usually in their own subdirectories.

When using a shared directory in the `view` directory (such as `common` above) be sure to name the templates in such a way to avoid naming collisions. A good practice is to prefix the template names with the name of your plugin.

## config/module.ini

Every module must have an INI file, a file containing basic information about the module. The file must be named `module.ini` and be saved in your module's `config/` directory.

* name (required): The human-readable name of the module
* version (required): The version of the current code state
* author (optional): The author of the module
* configurable (optional): Whether the module is configurable, true or false
* description (optional): A description of the module
* module_link (optional): An absolute URL to a page about the module
* author_link (optional): An absolute URL to a page about the author
* omeka_version_constraint (optional): A [Composer version constraint](https://getcomposer.org/doc/articles/versions.md) for what versions of the Omeka S core this module works with

```ini
name         = "My Module"
version      = "1.0"
author       = "My Organization"
configurable = true ## whether there is a configuration for the module
description  = "Lorem ipsum dolor sit amet, consectetur adipiscing elit."
module_link  = "http://my-organization.com/my-module"
author_link  = "http://my-organization.com"
omeka_version_constraint = "^1.0.0"
```

## Module.php

The Module.php file is required. It provides the methods needed to integrate your custom functionality with Omeka S.

### Making a Module Configurable

To enable module configuration, remember to set `configurable = true` in config/module.ini and use the `getConfigForm()` and `handleConfigForm()` methods in your module class, like below:

```php
use Omeka\Module\AbstractModule;
use Zend\View\Model\ViewModel;
use Zend\Mvc\Controller\AbstractController;

class Module extends AbstractModule
{
    /** Module body **/

    /**
     * Get this module's configuration form.
     *
     * @param ViewModel $view
     * @return string
     */
    public function getConfigForm(ViewModel $view)
    {
        return '<input name="foo">';
    }

    /**
     * Handle this module's configuration form.
     *
     * @param AbstractController $controller
     * @return bool False if there was an error during handling
     */
    public function handleConfigForm(AbstractController $controller)
    {
        return true;
    }
}
```

### Module Installation

Omeka allows modules to execute code whenever a module is installed or uninstalled.  This allows modules to do some setup tasks, like creating new schemas or seeding the database
with data.  To enable this, override the `install()` and `uninstall()` methods defined in `AbstractModule`.

```php
use Omeka\Module\AbstractModule;
use Zend\ServiceManager\ServiceLocatorInterface;

class Module extends AbstractModule
{

    public function install(ServiceLocatorInterface $serviceLocator)
    {
        // create database schema
    }

    public function uninstall(ServiceLocatorInterface $serviceLocator)
    {
        // drop database schema
    }
}
```

### Attaching to Omeka Events

Extending functionality is largely a matter of attaching listeners to events that Omeka fires at critical times. A list of events and where they're fired is found [here](Events (Server)). Use the `attachListeners` method in your module class, like below:

```php
use Omeka\Module\AbstractModule;
use Zend\EventManager\Event;
use Zend\EventManager\SharedEventManagerInterface;

class Module extends AbstractModule
{
    /** Module body **/

    /**
     * Attach listeners to events.
     *
     * @param SharedEventManagerInterface $sharedEventManager
     */
    public function attachListeners(SharedEventManagerInterface $sharedEventManager)
    {
        $sharedEventManager->attach(
            'Omeka\Controller\Admin\Item', // identifier for event emitting component
            'view.show.after', // event name
            function (Event $event) { // any callback
                // do something during the `view.show.after` event for a `Omeka\Controller\Admin\Item`
            }
        );
    }
}
```
### Packaging a Module

* All of the above MUST be included in a module package
* It SHOULD be made available as a .zip file

#### Adding to omeka.org

TBD










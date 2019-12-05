---
title: Introduction to Modules
---

You can extend the functionality of Omeka S by writing an add-on component called
a *module*. Zend Framework provides a substantial [framework for writing modules](https://docs.zendframework.com/zend-modulemanager/intro/),
but Omeka S provides extra structure that makes the modules installable, upgradeable,
and integratable.

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
    asset/
        <asset-directories-and files>/
    view/
        <module-namespace>/
            <controller-directories>/
                <action-template-files>.phtml
        common/
          mymodule-<template-files>.phtml
```

The name of the module directory (`MyModule` above) is significant. It must be a
reasonably unique, concise, and descriptive name of your module in CamelCase format.

The `asset` directory contains assets such as JavaScript, CSS, and image files,
usually in their own subdirectories.

When using a shared directory in the `view` directory (such as `common` above) be
sure to name the templates in such a way to avoid naming collisions. A good practice
is to prefix the template names with the name of your plugin.

## config/module.ini

Every module must have an INI file, a file containing basic information about the
module. The file must be named `module.ini` and be saved in your module's `config/`
directory. This file must start with `[info]` on the first line.

### Required information

* `name`: The human-readable name of the module
* `version`: The current version of the module

### Optional information

* `author`: The author of the module
* `configurable`: Whether the module is configurable, true or false
* `description`: A description of the module
* `module_link`: An absolute URL to a page about the module
* `author_link`: An absolute URL to a page about the author
* `omeka_version_constraint`: A [Composer version constraint](https://getcomposer.org/doc/articles/versions.md) for what versions of the Omeka S core this module works with

For example:

```ini
[info]
name         = "My Module"
version      = "1.0"
author       = "My Organization"
configurable = true
description  = "Lorem ipsum dolor sit amet, consectetur adipiscing elit."
module_link  = "http://my-organization.com/my-module"
author_link  = "http://my-organization.com"
omeka_version_constraint = "^1.0.0"
```

## Module.php

The `Module.php` file (and the `Module` class) is required. It provides the methods
needed to integrate your custom functionality with Omeka S.

### Module configuration

Note that your module's configuration must be returned from your module's `getConfig()`
method. Even so, we recommend that you include a `config/module.config.php` file
to better organize your code.

```php
use Omeka\Module\AbstractModule;
use Zend\View\Model\ViewModel;
use Zend\Mvc\Controller\AbstractController;

class Module extends AbstractModule
{
    /** Module body **/

    /**
     * Get this module's configuration array.
     *
     * @return array
     */
    public function getConfig()
    {
        return include __DIR__ . '/config/module.config.php';
    }
}
```

This config file is where you take care of many required tasks, such as registering
controllers, entities, routes, and navigation. It is a keyed array that should be
returned. Here is an excerpt from the MetadataBrowse module as an example:

```php

return [
    'view_manager' => [
        'template_path_stack' => [
            OMEKA_PATH.'/modules/MetadataBrowse/view',
        ],
    ],
    'controllers' => [
        'invokables' => [
            'MetadataBrowse\Controller\Admin\Index' => 'MetadataBrowse\Controller\Admin\IndexController',
        ],
    ],
    'form_elements' => [
        'factories' => [
            'MetadataBrowse\Form\ConfigForm' => 'MetadataBrowse\Service\Form\ConfigFormFactory',
        ],
    ],
];
```

### User configuration

To enable your module's user configuration, remember to set `configurable = true`
in config/module.ini and use the `getConfigForm()` and `handleConfigForm()` methods
in your module class, like below:

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

### Attaching to Omeka Events

Extending functionality is largely a matter of attaching listeners to events that
Omeka triggers at critical times. Modules can attach to Omeka S's [server-side events](../events/server_events.md)
in the `Module.php` file. Use the `attachListeners` method in your module class,
like below:

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

## Packaging a Module

* At minimum, a module MUST contain a `Module.php` file and a `config/module.ini` file, following the requirements described above.
* It SHOULD be made available as a .zip file

## Adding to omeka.org

See [Register an addon](../register_an_addon.md).

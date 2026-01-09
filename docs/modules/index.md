# Introduction to Modules

You can extend the functionality of Omeka S by writing an add-on component called
a *module*. Laminas Framework provides a substantial [framework for writing modules](https://docs.laminas.dev/laminas-modulemanager/intro/),
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
        my-module/
            <controller-directories>/
                <action-template-files>.phtml
        common/
            mymodule-<template-files>.phtml
```

The name of the module directory (`MyModule` above) is significant. It must be a
unique, concise, and descriptive name of your module in CamelCase format.

The `asset` directory contains assets such as JavaScript, CSS, and image files,
usually in their own subdirectories.

View templates for pages are stored under the `view/my-module` folder. The name of
the folder under `view` is a lowercase, hyphen-separated version of your module's
name (so `MyModule` becomes `my-module`).

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

```php-inline
use Omeka\Module\AbstractModule;
use Laminas\View\Model\ViewModel;
use Laminas\Mvc\Controller\AbstractController;

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

```php-inline

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

```php-inline
namespace MyModule;

use Omeka\Module\AbstractModule;
use Laminas\View\Model\ViewModel;
use Laminas\Mvc\Controller\AbstractController;

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

```php-inline
namespace MyModule;

use Omeka\Module\AbstractModule;
use Laminas\EventManager\Event;
use Laminas\EventManager\SharedEventManagerInterface;

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

## MVC

Omeka S uses Laminas Framework's [MVC layer](https://docs.laminas.dev/laminas-mvc/)
to coordinate between its data, program logic, and presentation components. For
an introduction and detailed reference to the MVC layer, please read Laminas Framework's
[documentation](https://docs.laminas.dev/laminas-mvc/intro/). The [Quick Start](https://docs.laminas.dev/laminas-mvc/quick-start/)
guide is particularly helpful because it demonstrates how to do the following:

- Set up your view manager and controller configurations
- Create your controllers
- Create your view scripts
- Bring it all together by creating your routes

You can follow those directions as well as the directions we provide in this section
to create your own Omeka S module.

## Packaging a Module

* At minimum, a module MUST contain a `Module.php` file and a `config/module.ini` file, following the requirements described above.
* It SHOULD be made available as a .zip file

## Adding to omeka.org

See [Register an addon](../register_an_addon.md).

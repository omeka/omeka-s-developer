---
title: Introduction to modules
---

You can extend the functionality of Omeka S by writing an add-on component called a *module.* Zend Framework 2 provides a substantial [framework for writing modules](https://docs.zendframework.com/zend-modulemanager/intro/), but Omeka S provides extra structure that makes the modules installable, upgradeable, and integratable.

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

The name of the module directory (`MyModule` above) is significant. It must be a reasonably unique, concise, and descriptive name of your module in CamelCase format.

The `asset` directory contains assets such as JavaScript, CSS, and image files, usually in their own subdirectories.

When using a shared directory in the `view` directory (such as `common` above) be sure to name the templates in such a way to avoid naming collisions. A good practice is to prefix the template names with the name of your plugin.

## config/module.ini

Every module must have an INI file, a file containing basic information about the module. The file must be named `module.ini` and be saved in your module's `config/` directory. This file must start with `[info]` on the first line.

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

## config/module.config.php

This file is automatically added to Omeka S's configuration. This is where you take care of many required tasks, such as registering controllers, entities, routes, and navigation. It is a keyed array that should be returned. Here is an excerpt from the MetadataBrowse module as an example:

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
### Installing the Data Model

If your module needs to create database tables for Entities, follow this process after you have defined your Entity classes. See [Doctrine and Entities](../key_concepts/doctrine_and_entities.md) for an introduction.

Run

    $ php application/data/scripts/update-db-data-module.php <ModuleName>

and copy the statements it outputs. In your `Module.php` file's `install` function, copy it into the `$sql` variable with this pattern:

```php
    public function install(ServiceLocatorInterface $serviceLocator)
    {
        
        $connection = $serviceLocator->get('Omeka\Connection');
        $sql = $sqlDump; // whatever the result of the above gives you 
          
        $connection->exec($sql);
    }

```

This command will also create Doctrine "proxy" class files for your entities and place them in your module at "data/doctrine-proxies."


### Updating the Data Model

When you've made a change in the data model for a module, you need to follow mostly the same process as
getting the initial installation SQL from the previous section. As above, run the `update-db-data-module`
script to get the SQL statements, and update the module's `install` method to use the updated statements.

In addition, you'll need to handle updating the database for users who've already installed the old version
of your module (otherwise they'd be forced to uninstall and reinstall it, losing their data in the process).
The first step is to increase the version number of the module in its `config/module.ini` file. Omeka S can
only detect that an upgrade is necessary for a module if the version number increases. Version numbers should
follow [Semantic Versioning](http://semver.org/).

Exactly what needs to be done for an upgrade can vary based on the kinds of changes you've made, but usually
it's sufficient to just compare the previous installation SQL statements to the new ones, and create ALTER
statements as appropriate to add, remove, or change the necessary tables, columns, and indexes. More involved
changes may require you to also UPDATE or INSERT actual rows as well.

Just as the initial SQL goes in the Module class `install` method, upgrade SQL goes in the `upgrade` method.
Unlike, `install`, `upgrade` has some additional important arguments: `$oldVersion` and `$newVersion`,
representing the version of the plugin the user is upgrading from and the version they're upgrading to,
respectively. For most cases, `$oldVersion` is what's important here: you want to check if the previous version
is lower than the new version you just incremented to. If so, you need to run the update SQL.

Since the versions for modules follow Semantic Versioning, you should use the
[Comparator](https://github.com/composer/semver#comparator) class from Composer's "Semver" package
to compare version numbers. Omeka S already includes this as a dependency, so you simply need to `use`
it.

```php
use Composer\Semver\Comparator;

// ...

public function upgrade($oldVersion, $newVersion, ServiceLocatorInterface $serviceLocator)
{
    $connection = $serviceLocator->get('Omeka\Connection');
    
    if (Comparator::lessThan($oldVersion, '1.1.0')) { 
        $sql = $sqlDump; // your upgrade SQL statements 
        $connection->exec($sql);
    }
}
```

As you continue to make updates and add to the `upgrade` method, users will be able to upgrade many versions at once.

### Attaching to Omeka Events

Extending functionality is largely a matter of attaching listeners to events that Omeka fires at critical times. A list of events and where they're fired is found [here](../reference/events.md). Use the `attachListeners` method in your module class, like below:

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

* At minimum, a module MUST contain a `Module.php` file and a `config/module.ini` file, following the requirements described above.
* It SHOULD be made available as a .zip file

#### Adding to omeka.org

See [Register an addon](../register_an_addon.md).

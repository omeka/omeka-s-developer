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

```ini
name         = "My Module"
version      = "1.0"
author       = "My Organization"
configurable = true
description  = "Lorem ipsum dolor sit amet, consectetur adipiscing elit."
module_link  = "http://my-organization.com/my-module"
author_link  = "http://my-organization.com"
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

If your module needs to create database tables for Entities, follow this process after you have defined your Entity classes. See [Doctrine and Entities](doctrine_and_entities.md) for an introduction.

Run

    $ vendor/bin/doctrine orm:schema-tool:update --dump-sql

and copy the relevant statements. In your `Module.php` file's `install` function, copy it into the `$sql` variable with this pattern:

```php
    public function install(ServiceLocatorInterface $serviceLocator)
    {
        
        $connection = $serviceLocator->get('Omeka\Connection');
        $sql = $sqlDump; // whatever the result of the above gives you 
          
        $connection->exec($sql);
    }

```


### Updating the Data Model

In the course of developing Omeka you'll probably find reason to modify the data model. Whether you add or remove an entity, or add, remove, or modify a column, you'll need to reflect these changes in the database.

First you need to get the SQL necessary to update an existing, installed database:

    $ vendor/bin/doctrine orm:schema-tool:update --dump-sql

Copy the resulting statements. Then you need to make a migration for the changes:

    $ ant create-migration

This will prompt you to name the migration. Do so and press enter. Once the task is finished, open the newly created migration file in data/migration/ (it's now prefixed with a datestamp) and use the provided connection object to make whatever changes need to be made, using the SQL you copied previously.

Then you need to update the static database-related files:

    $ ant update-db-data

This will overwrite the existing installation schema with the most updated version of the data model, and re-generate Doctrine's proxy classes.

In order for Omeka to detect a new migration, you'll need to update the Omeka version number. Open `application/Module.php` and increment the `VERSION` constant.

Now stage and commit the resulting changes.


### Attaching to Omeka Events

Extending functionality is largely a matter of attaching listeners to events that Omeka fires at critical times. Use the `attachListeners` method in your module class, like below:

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

## See Also

[Omeka Events](omeka_events.md)

---
title: Doctrine and Entities
---

Omeka S uses [Doctrine](http://www.doctrine-project.org/) as its Object Relational Mapper. Here is a summary of how Omeka S uses it. Refer to their documentation for details.

## Defining an Entity

The `Omeka\Entity\AbstractEntity` class should be extended for any tasks that stores to the database. Doctrine uses code block annotations to define database relations, and can create database queries that create the necessary schema. `Omeka\Entity\Module` provides a basic example.


```php

<?php
namespace Omeka\Entity;

/**
 * @Entity
 */
class Module extends AbstractEntity
{
    /**
     * @Id
     * @Column(type="string", length=190)
     */
    protected $id;

    /**
     * @Column(type="boolean")
     */
    protected $isActive = false;

    /**
     * @Column
     */
    protected $version;
    
    // function definitions follow
    
}
```

The `@Entity` declaration in the top level comment block tells Doctrine that this class should be represented in the database. An instance of that class will be a row in the database. Each protected property that has a `@Column` declaration in its comment block tells Doctrine that the data in the property should be represented as a column in the row.

Thus, the `module` table has three columns: `id` of type string, `isActive` of type boolean, and `version`.

The Doctrine `orm:schema-tool:update --dump-sql` tool can be used to see the SQL query used to create a table in this way. Omeka S, of course, has integrated this process into the installation process. See [Modules](../modules/doctrine_and_entities.md) for how modules should implement this process.

## Updating the Data Model (Core)

In the course of developing Omeka you'll probably find reason to modify the data model. Whether you add or remove an entity, or add, remove, or modify a column, you'll need to reflect these changes in the database.

First you need to get the SQL necessary to update an existing, installed database:

    $ vendor/bin/doctrine orm:schema-tool:update --dump-sql

Copy the resulting statements. Then you need to make a migration for the changes:

    $ gulp db:create-migration

This will prompt you to name the migration. Do so and press enter. Once the task is finished, open the newly created migration file in data/migration/ (it's now prefixed with a datestamp) and use the provided connection object to make whatever changes need to be made, using the SQL you copied previously.

Then you need to update the static database-related files:

    $ gulp db

This will overwrite the existing installation schema with the most updated version of the data model, and re-generate Doctrine's proxy classes.

In order for Omeka to detect a new migration, you'll need to update the Omeka version number. Open `application/Module.php` and increment the `VERSION` constant.

Now stage and commit the resulting changes.

## Installing the Data Model (Module)

If your module needs to create database tables for Doctrine Entities, follow this
process after you have defined your Entity classes.

First you'll need to configure your module so Doctrine knows where your Entity and
proxy classes are. In `config/module.config.php`, add the following:

```php
    'entity_manager' => [
        'mapping_classes_paths' => [
            dirname(__DIR__) . '/src/Entity',
        ],
        'proxy_paths' => [
            dirname(__DIR__) . '/data/doctrine-proxies',
        ],
    ],
```

Then, run the following command in your terminal:

    $ php application/data/scripts/update-db-data-module.php <ModuleName>

This will create Doctrine "proxy" class files for your entities and place them in
your module at `data/doctrine-proxies`. It will also output the SQL statements needed
to install your data model. Copy the statements, then, in your `Module.php` file's
`install()` function, use the database connection service to execute them:

```php
    public function install(ServiceLocatorInterface $services)
    {
        $connection = $services->get('Omeka\Connection');
        $connection->exec('CREATE TABLE my_entity ...');
        $connection->exec('CREATE TABLE my_other_entity ...');
        $connection->exec('ALTER TABLE my_entity ...');
        $connection->exec('ALTER TABLE my_other_entity ...');
    }
```

Note that, even though `exec()` is capable of executing more than one statement
at a time, you should run it only once for every statement. This is to avoid SQL
errors that could be hidden by multi-line input.

## Updating the Data Model (Module)

When you've made a change in the data model for a module, you need to follow mostly
the same process as getting the initial installation SQL from the previous section.
As above, run the `update-db-data-module` script to get the SQL statements, and
update the module's `install` method to use the updated statements.

In addition, you'll need to handle updating the database for users who've already
installed the old version of your module (otherwise they'd be forced to uninstall
and reinstall it, losing their data in the process). The first step is to increase
the version number of the module in its `config/module.ini` file. Omeka S can only
detect that an upgrade is necessary for a module if the version number increases.
Version numbers should follow [Semantic Versioning](http://semver.org/).

Exactly what needs to be done for an upgrade can vary based on the kinds of changes
you've made, but usually it's sufficient to just compare the previous installation
SQL statements to the new ones, and create ALTER statements as appropriate to add,
remove, or change the necessary tables, columns, and indexes. More involved changes
may require you to also UPDATE or INSERT actual rows as well.

Just as the initial SQL goes in the Module class `install` method, upgrade SQL goes
in the `upgrade` method. Unlike, `install`, `upgrade` has some additional important
arguments: `$oldVersion` and `$newVersion`, representing the version of the plugin
the user is upgrading from and the version they're upgrading to, respectively. For
most cases, `$oldVersion` is what's important here: you want to check if the previous
version is lower than the new version you just incremented to. If so, you need to
run the update SQL.

Since the versions for modules follow Semantic Versioning, you should use the [Comparator](https://github.com/composer/semver#comparator)
class from Composer's "Semver" package to compare version numbers. Omeka S already
includes this as a dependency, so you simply need to `use` it.

```php
use Composer\Semver\Comparator;

// ...

public function upgrade($oldVersion, $newVersion, ServiceLocatorInterface $services)
{
    $connection = $services->get('Omeka\Connection');
    if (Comparator::lessThan($oldVersion, '1.1.0')) { 
        $sql = $sqlDump; // your upgrade SQL statements 
        $connection->exec($sql);
    }
}
```

As you continue to make updates and add to the `upgrade` method, users will be able
to upgrade many versions at once.

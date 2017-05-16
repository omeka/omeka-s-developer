# Doctrine and Entities

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

The Doctrine `orm:schema-tool:update --dump-sql` tool can be used to see the SQL query used to create a table in this way. Omeka S, of course, has integrated this process into the installation process. See [Modules](modules.md) for how modules should implement this process.

## Updating the Data Model

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


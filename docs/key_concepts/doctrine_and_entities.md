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


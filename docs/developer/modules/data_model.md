# Data Model

## Installing the Data Model

If your module needs to create database tables for Doctrine Entities, follow this
process after you have defined your Entity classes.

First you'll need to configure your module so Doctrine knows where your Entity and
proxy classes are. In `config/module.config.php`, add the following:

```php-inline
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

```php-inline
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

## Updating the Data Model

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

```php-inline
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

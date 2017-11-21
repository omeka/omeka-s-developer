# Debugging

## Configuration

To configure Omeka S for a development server, you will need to make these changes / additions.

In `.htaccess` change

```
SetEnv APPLICATION_ENV "production"
```
to
```
SetEnv APPLICATION_ENV "development"
```

In `config/local.config.php`, add

```php
    'logger' => [
        'log' => true,
    ],
```
Log messages are written to `logs/application.log`

To use internal assests, such as a local copy of jQuery, add this to `local.config.php`

```php
    'assets' => [
        'use_externals' => false,
    ],
```



## Logging within Controllers

Omeka S provides a `logger()` plugin withing Controllers. This provides access to the Omeka S logging system. You can write messages to the log with, e.g., `$this->logger()->info("Status: good");`

The `Omeka\Mvc\Controller\Plugin\Logger` object uses methods from `Zend\Log\LoggerInterface`, which makes it easy to give messages at different log levels:

* emerg()
* alert()
* crit()
* err()
* warn()
* notice()
* info()
* debug()

## Logging within Jobs

Jobs that run in the background do not have access to the `logger()` plugin. Instead, you can get the logger from the ServiceManager anywhere inside the Job class:

```php
$logger = $this->getServiceLocator()->get('Omeka\Logger');
```

Then use as above, e.g. `$logger()->info('Importing page 1')`

## Logging within Views

## Logging elsewhere

If you need to do some debugging work elsewhere, such as within an Entity, you need to inject the Logger via a factory for the object. See [Services and Factories](services_and_factories.md) for details.


Within the `__invoke()` method of your factory, add

```php
$logger = $serviceLocator->get('Omeka\Logger');
```

and add it as a property for your class.


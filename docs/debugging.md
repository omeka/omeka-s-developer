# Debugging Omeka S

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

Log messages are not written to `application.log` from inside a Job. Instead, they are written to the Job's record. You can view the information by look at the Job record in the admin screen.

## Logging within Views

## Logging elsewhere

If you need to do some debugging work elsewhere, such as within an Entity, you need to inject the Logger via a factory for the object. See [Services and Factories](configuration/services_and_factories.md) for details.


Within the `__invoke()` method of your factory, add

```php
$logger = $serviceLocator->get('Omeka\Logger');
```

and add it as a property for your class.

## Logging to stderr

When you want to temporarily log something where you don't have easy access to
the service locator, as a last resort you can use
['plain' Laminas logging](https://docs.laminas.dev/laminas-log/intro/) to
log to stderr. This is only suitable for temporary debug logging, not to be
committed to version control:

```php
use Laminas\Log\Logger;
use Laminas\Log\Writer;

...

	$logger = new Logger;
	$writer = new Writer\Stream('php://stderr');
	$logger->addWriter($writer);

	$logger->warn($result);
```

The logging written to stderr can typically be found in your webserver's error log.

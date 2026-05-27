# Debugging

## Configuration

To configure Omeka S for a development server, you will need to make these changes / additions.

To enable error display, in the file  `.htaccess` change

```
SetEnv APPLICATION_ENV "production"
```
to
```
SetEnv APPLICATION_ENV "development"
```

To enable logging, in `config/local.config.php`, set

```php-inline
    'logger' => [
        'log' => true,
    ],
```

Log messages are written by default to `logs/application.log`.

To use internal assests, such as a local copy of jQuery, add this to `local.config.php`

```php-inline
    'assets' => [
        'use_externals' => false,
    ],
```

### More log configuration

The `logger` section of `config/local.config.php` also has a `priority` setting that controls the lowest level of messages that will be logged. The default
setting is `\Laminas\Log\Logger::NOTICE`, meaning that "info" and "debug" messages will not be logged. Changing the setting to `\Laminas\Log\Logger::DEBUG`
will log all messages.

In the same section you can also set `path` to change where the log file is stored.

## Logging within Controllers and Views

Omeka S provides a `logger()` controller plugin and view helper. Theses provides access to the Omeka S logging system. You can write messages to the log with, e.g., `$this->logger()->notice("Status: good");`

`$this->logger()` returns an object that implements the Laminas log interface, which has convenience methods for logging different levels of messages:

* emerg()
* alert()
* crit()
* err()
* warn()
* notice()
* info()
* debug()

Also, a generic `log()` method allows specifying the level as a parameter instead.

## Logging within Jobs

Jobs that run in the background do not have access to the `logger()` plugin. Instead, you can get the logger from the ServiceManager anywhere inside the Job class:

```php-inline
$logger = $this->getServiceLocator()->get('Omeka\Logger');
```

Then use as above, e.g. `$logger->notice('Importing page 1');`

During a job, logs are written to the Job's database record in addition to  `logs/application.log`. You can view the information by looking at the Job record in the admin screen.

## Logging elsewhere

If you need to do some debugging work elsewhere, such as within an Entity, you need to inject the Logger via a factory for the object. See [Services and Factories](configuration/services_and_factories.md) for details.

Within the `__invoke()` method of your factory, add

```php-inline
$logger = $serviceLocator->get('Omeka\Logger');
```

and add it as a property for your class.

## PSR-3 logging

The log options mentioned above use the Laminas-specific log interface. As of Omeka S 4.2.0, developers can instead use the PSR-3 log interface standard.

The controller helper is `$this->psrLogger()` instead of `$this->logger()`, and the service for getting the logger directly is `Omeka\PsrLogger` instead of `Omeka\Logger`.

Usage is very similar to the Laminas logger, except the log levels are named slightly differently:

* emergency()
* alert()
* critical()
* error()
* warning()
* notice()
* info()
* debug()

PSR-3 also specifies a placeholder system for writing error messages with dynamic portions. Where code using the Laminas logger might construct an error message with `sprintf`,
a PSR-3 log message might use this placeholder system instead:

```php-inline
$this->psrLogger()->notice('Item {id} created', ['id' => $itemId]);
```

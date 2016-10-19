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
Log messages are written to `data/logs/application.log`

## Logging within Controllers

Omeka S provides a `logger()` plugin for Controllers. This provides access to the Omeka S logging system. You can write messages to the log with, e.g., `$this->logger()->info("Status: good");`

The `Omeka\Mvc\Controller\Plugin\Logger` object uses methods from `Zend\Log\LoggerInterface`, which makes it easy to give messages at different log levels:

* emerg()
* alert()
* crit()
* err()
* warn()
* notice()
* info()
* debug()



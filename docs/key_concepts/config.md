# Configuration

You can configure Omeka S by editing `config/local.config.php`. Options from the
application and module configuration files are merged into local configuration
and used by Omeka S services to operate the application. See the following files
to reference default options and their format (do not modify these files):

- `application/config/module.config.php`: read-only default configurations (see [Configuration Reference](../reference/configuration.md))
- `application/config/navigation.config.php`: read-only navigation configuration (see [zend-navigation docs](https://docs.zendframework.com/zend-navigation/pages/#mvc-pages))
- `application/config/routes.config.php`: read-only routing configuration (see [zend-mvc docs](http://zendframework.github.io/zend-mvc/routing/))

Local configuration comes with options that are most likely to be changed from
one installation to another. Feel free to add more depending on your system's
requirements.

## Invocables and Factories

You will often see subkeys of `invocables`, `factories`, etc. in the configuration
file. These refer to how the various Omeka S service managers create new services.
See the [zend-servicemanager docs](https://docs.zendframework.com/zend-servicemanager/configuring-the-service-manager/)
to learn how to configure your services.

## See also

- [Configuration Reference](../reference/configuration.md)
- [Services and Factories](services_and_factories.md)
- [Modules](modules.md)

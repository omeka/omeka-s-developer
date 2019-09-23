---
title: Configuration
---

You can configure Omeka S by editing `config/local.config.php`. Settings from
the application and module configuration files are merged into local
configuration and used by Omeka S services to operate the application. See the
following files to reference default settings and their format (do not modify
these files):

- `application/config/module.config.php`: read-only default settings (see [Configuration Settings](../reference/configuration.md))
- `application/config/navigation.config.php`: read-only navigation settings (see [zend-navigation docs](https://docs.zendframework.com/zend-navigation/pages/#mvc-pages))
- `application/config/routes.config.php`: read-only routing settings (see [zend-mvc docs](http://zendframework.github.io/zend-mvc/routing/))

Local configuration comes with settings that are most likely to be changed from
one installation to another. Feel free to add more depending on your system's
requirements.

Modules add their own configuration settings via their `config/module.config.php`
files. Note that these configuration settings are merged into local
configuration, and can extend, and possibly modify, the core configuration. See
[Modules](../modules/index.md) for more information.

## Invokables and Factories

You will often see subkeys of `invokables`, `factories`, etc. in the
configuration file. These refer to how the various Omeka S service managers
create new services. See [Services and Factories](services_and_factories.md) and
the [zend-servicemanager docs](https://docs.zendframework.com/zend-servicemanager/configuring-the-service-manager/)
to learn how to configure your services.

## See also

- [Configuration Settings](../reference/configuration.md)
- [Services and Factories](services_and_factories.md)
- [Modules](../modules/index.md)

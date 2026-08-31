# Services and Factories

## Services

Following Laminas Framework, services, taken together, comprise most the the core functionality of Omeka S. For example, to execute a database query, you need first to get the `Omeka\Connection` service:

```php-inline
$connection = $serviceLocator->get('Omeka\Connection');

$sql = ''; // whatever SQL you need to execute
$connection->exec($sql);
```

For many common tasks, Omeka S provides helpers for obtaining the necessary service. Thus, when you need to use the `Omeka\Logger` service, you need not get it via the `$serviceLocator` (indeed, often you cannot -- more on that below). Instead, within a controller you can simply do:

```php-inline
$this->logger()->warn('Something bad is happening.');
```

The `service_manager` config key provides global services like `Omeka\Connection` and `Omeka\Logger` mentioned above. The same "services" pattern and configuration is also used with many other keys like `controllers`, `api_adapters`, and more. These configurations are often recognizable by their having subkeys `invokables` or `factories`, which represent the two common ways of registering services.

This page focuses on these commonly-used cases, but there are more that are used in more specific situations. For more information about all the configuration options for service managers, see [the Laminas documentation](https://docs.laminas.dev/laminas-servicemanager/v3/configuring-the-service-manager/).

## Invokables

The `invokables` subkey is used for simple services that have no dependencies. An invokable links a key with a class name, and when that service is needed, the class is simply instantiated.

A good rule of thumb is that `invokables` is usually suitable for services whose class either has no constructor, or has a constructor that takes no arguments. If the class needs to take arguments, or otherwise needs to be set up beyond simple instantiation, then a factory is probably needed (see the next section).

For an invokable, all that's necessary is one line that maps the key to the class name. For example, the Mapping module registers two API adapters to add resources to the Omeka S API:

```php-inline
    'api_adapters' => [
        'invokables' => [
            'mappings' => Api\Adapter\MappingAdapter::class,
            'mapping_features' => Api\Adapter\MappingFeatureAdapter::class,
        ],
    ],
```

The key needs to be unique; reusing a key means you'll be overriding an existing service rather than registering a new one.

## Factories

Factories are used to instantiate a class and inject other related data and classes into it. Often, a factory gets used when a class has a dependency on one or more other services.

Take, for example, the Omeka2Importer plugin. Its first job is to retrieve data from an existing Omeka Classic site. A client for interacting with Omeka Classic's API had already been developed to handle the tasks of requesting and processing data. That client just needed to be included into the Omeka S module to make it available. That happens by using a Factory to inject the Service into the Controller.

```php-inline
namespace Omeka2Importer\Service\Controller;

use Omeka2Importer\Controller\IndexController;
use Laminas\ServiceManager\Factory\FactoryInterface;
use Interop\Container\ContainerInterface;

class IndexControllerFactory implements FactoryInterface
{
    public function __invoke(ContainerInterface $container, $requestedName, array $options = null)
    {
        $client = $container->get('Omeka2Importer\Omeka2Client');
        $indexController = new IndexController($client);
        return $indexController;
    }
}
```
The constructor for `Omeka2Importer\Controller\IndexController\IndexController` then assigns the client to its corresponding property, and uses it as needed.

Another common Factory task is to inject needed services into Forms. For example, if you need the `Laminas\Event\EventManager` to trigger an event or access to Site setting, you will need to create the Form via a Factory that injects it:

```php-inline
namespace Omeka\Service\Form;

use Omeka\Form\SiteSettingsForm;
use Laminas\ServiceManager\Factory\FactoryInterface;
use Interop\Container\ContainerInterface;

class SiteSettingsFormFactory implements FactoryInterface
{
    public function __invoke(ContainerInterface $services, $requestedName, array $options = null)
    {
        $form = new SiteSettingsForm;
        $form->setSiteSettings($services->get('Omeka\SiteSettings'));
        $form->setEventManager($services->get('EventManager'));
        return $form;
    }
}

```

The Form itself must have getters and setters:

```php-inline
    /**
     * @param SiteSettings $siteSettings
     */
    public function setSiteSettings(SiteSettings $siteSettings)
    {
        $this->siteSettings = $siteSettings;
    }

    /**
     * @return SiteSettings
     */
    public function getSiteSettings()
    {
        return $this->siteSettings;
    }

```

Those services will now be available within the form.

### Factory Configuration

Factories use the `factories` subkey. The key of the entry for a service works the same way as for `invokables`, but the value for a factory is instead the name of the *factory* class.

Taking the Omeka2Importer example, in `module.config.php` the index controller is registered using the `factories` subkey:

```php-inline
    'controllers' => [
        'factories' => [
            'Omeka2Importer\Controller\Index' => 'Omeka2Importer\Service\Controller\IndexControllerFactory',
        ],
    ],
```

## See also

[Configuration files](index.md)

# Services and Factories

See config file. // to write

## Services

Following Zend Framework 3, services, taken together, comprise most the the core functionality of Omeka S. For example, to execute a database query, you need first to get the `Omeka\Connection` service:

```php
$connection = $serviceLocator->get('Omeka\Connection');

$sql = ''; // whatever SQL you need to execute
$connection->exec($sql);
```

For many common tasks, Omeka S provides helpers for obtaining the necessary service. Thus, when you need to use the `Omeka\Logger` service, you need not get it via the `$serviceLocator` (indeed, often you cannot -- more on that below). Instead, within a controller you can simply do:

```php
$this->logger()->warn('Something bad is happening.');
```




## Factories

Among other things, factories are used to instantiate a class and inject other related data and classes into it. In Omeka S, this commonly means making a service available to address a special condition in which the service serves a special need.

Take, for example, the Omeka2Importer plugin. Its first job is to retrieve data from an existing Omeka Classic site. A client for interacting with Omeka Classic's API had already been developed to handle the tasks of requesting and processing data. That client just needed to be included into the Omeka S module to make it available. That happens by using a Factory to inject the Service into the Controller.

```php

namespace Omeka2Importer\Service\Controller;

use Omeka2Importer\Controller\IndexController;
use Zend\ServiceManager\Factory\FactoryInterface;
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


### Factory Configuration

See config file. // to write

In `module.config.php`, you will need to assert that the class in question is produced by a factory:

```php
    'controllers' => array(
        'factories' => array(
            'Omeka2Importer\Controller\Index' =>                'Omeka2Importer\Service\Controller\IndexControllerFactory',
        ),
    ),

```

## See also

Configuration // to write




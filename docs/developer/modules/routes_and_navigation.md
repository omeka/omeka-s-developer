# Routes and Navigation

## Routes

Routes define the connections between URL paths and the controllers and actions
that handle them. Thus, for a module to display a page at, e.g. `/yourmodule/controller/action`,
you need to define that route in `module.config.php`:

```php-inline
    'router' => [
        'routes' => [
                'your-module' => [
                    'type' => 'Segment',
                    'options' => [
                        'route' => '/yourmodule/controller/action',
                        'defaults' => [
                            '__NAMESPACE__' => 'Yourmodule\Controller',
                            'controller' => 'Controller',
                            'action' => 'action',
                        ],
                    ],
                ],
            ],
        ],
```

There are many options for defining the route. See the [Laminas Framework documentation](https://docs.laminas.dev/laminas-router/routing/)
for details.

## Navigation

Adding navigation for your module depends on the route being defined, then adding
the route to the navigation system. For a link on the left sidebar on admin pages,
it looks like this, taken from the Omeka2Importer module. First, the route is added
as a child of the main admin router. There will be two valid routes for the module,
it defines two child routes of its own under the `omeka2importer` route, `past-imports`
and `map-elements`.

```php-inline
    'router' => [
        'routes' => [
            'admin' => [
                'child_routes' => [
                    'omeka2importer' => [
                        'type' => 'Literal',
                        'options' => [
                            'route' => '/omeka2importer',
                            'defaults' => [
                                '__NAMESPACE__' => 'Omeka2Importer\Controller',
                                'controller' => 'Index',
                                'action' => 'index',
                            ],
                        ],
                        'may_terminate' => true,
                        'child_routes' => [
                            'past-imports' => [
                                'type' => 'Literal',
                                'options' => [
                                    'route' => '/past-imports',
                                    'defaults' => [
                                        '__NAMESPACE__' => 'Omeka2Importer\Controller',
                                        'controller' => 'Index',
                                        'action' => 'past-imports',
                                    ],
                                ],
                            ],
                            'map-elements' => [
                                'type' => 'Literal',
                                'options' => [
                                    'route' => '/map-elements',
                                    'defaults' => [
                                        '__NAMESPACE__' => 'Omeka2Importer\Controller',
                                        'controller' => 'Index',
                                        'action' => 'map-elements',
                                    ],
                                ],
                            ],
                        ],
                    ],
                ],
            ],
        ],
    ],
```

Then, the navigation key adds to the Admin navigation, defined by `AdminModule`.
As will all settings in `module.config.php`, Omeka merges the array values for each
key.

```php-inline
    'navigation' => [
        'AdminModule' => [
            [
                'label' => 'Omeka 2 Importer',
                'route' => 'admin/omeka2importer',
                'resource' => 'Omeka2Importer\Controller\Index',
                'pages' => [
                    [
                        'label' => 'Import',
                        'route' => 'admin/omeka2importer',
                        'resource' => 'Omeka2Importer\Controller\Index',
                    ],
                    [
                        'label' => 'Import',
                        'route' => 'admin/omeka2importer/map-elements',
                        'resource' => 'Omeka2Importer\Controller\Index',
                        'visible' => false,
                    ],
                    [
                        'label' => 'Past Imports',
                        'route' => 'admin/omeka2importer/past-imports',
                        'controller' => 'Index',
                        'action' => 'past-imports',
                        'resource' => 'Omeka2Importer\Controller\Index',
                    ],
                ],
            ],
        ],
    ],
```

The links to pages correspond to the routes defined in the router.

The pattern for adding to a Site's navigation is similar, but adds to the `admin/slug`
route and the `site` navigation, as shown in this example taken from the MetadataBrowse
module:

```php-inline
    'navigation' => [
        'site' => [
            [
                'label' => 'Metadata Browse', // @translate
                'route' => 'admin/site/slug/metadata-browse/default',
                'action' => 'index',
                'useRouteMatch' => true,
                'pages' => [
                    [
                        'route' => 'admin/site/slug/metadata-browse/default',
                        'visible' => false,
                    ],
                ],
            ],
        ],
    ],
    'router' => [
        'routes' => [
            'admin' => [
                'child_routes' => [
                    'site' => [
                        'child_routes' => [
                            'slug' => [
                                'child_routes' => [
                                    'metadata-browse' => [
                                        'type' => 'Literal',
                                        'options' => [
                                            'route' => '/metadata-browse',
                                            'defaults' => [
                                                '__NAMESPACE__' => 'MetadataBrowse\Controller\Admin',
                                                'controller' => 'index',
                                                'action' => 'index',
                                            ],
                                        ],
                                        'may_terminate' => true,
                                        'child_routes' => [
                                            'default' => [
                                                'type' => 'Segment',
                                                'options' => [
                                                    'route' => '/:action',
                                                    'constraints' => [
                                                        'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                                                    ],
                                                ],
                                            ],
                                        ],
                                    ],
                                ],
                            ],
                        ],
                    ],
                ],
            ],
        ],
    ],
```

## See also

[Controllers](index.md)
[Configuration](../configuration/index.md)

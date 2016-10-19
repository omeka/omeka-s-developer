# Internationalization

Roy Rosezweig Center for History and New Media is commited to making Omeka S available in as many people as possible.

Here's how we are doing that in the core. The same principles apply to themes and modules.

how will translations actually be produced, and will they be open for contribution for beta release?

## Within a view

Use the `translate()` helper.

### Examples

#### From `application/view-shared/omeka/install/index/index.phtml`

```php
<h1><?php echo $this->translate('Install Omeka S'); ?></h1>
```

## Within config or Form files

Comment the line to be translated with `// @translate`. Our processing will pick up the string to be translated and make it available to the translation teams.

### Examples

#### From a config file

This example applies internationalization to the site navigation in a module's `module.config.php` file

```php
return [
    'navigation' => [
        'site' => [
            [
                'label' => 'Collecting', // @translate
                'route' => 'admin/site/slug/collecting',
                'action' => 'index',
                'useRouteMatch' => true,
                'pages' => [
                    // .....
                ],
            ],
        ];
```

#### From a Form that extends `Zend\Form\Form`

```php
    public function init()
    {
        $this->add([
            'name' => 'o:email',
            'type' => 'Email',
            'options' => [
                'label' => 'Email', // @translate
            ],
            'attributes' => [
                'id' => 'email',
                'required' => true,
            ],
        ]);
    };
```


## See also

* [Config File](config.md)
* [Forms](forms.md)
* [Routes and Navigation](routes_and_navigation.md)
* [Translate Helper]()
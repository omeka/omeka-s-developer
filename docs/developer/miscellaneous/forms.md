# Forms

Omeka S often extends `Laminas\Form\Form` in the namespace `Omeka\Form`, and in some cases includes events to allow modules to add more fields to the form.

Adding a fieldset is the general pattern for modules to follow. In your `Module.php` class add the following to add a text area. 

This applies to these forms

* `Omeka\Form\SettingForm`
* `Omeka\Form\SiteSettingsForm`


Use the `Event::ADD_ELEMENTS` and `Event::ADD_INPUT_FILTERS` events, respectively.

See [Modules](../modules/index.md) for the broader structure of modules.

```php-inline

    public function addElements(Event $event)
    {
        $form = $event->getParam('form');
        $fieldset = new Fieldset('example');
        $fieldset->setLabel('Example Fieldset');
        
        $fieldset->add([
                'name' => 'example',
                'type' => 'text',
                'options' => [
                    'label' => 'Example text input', // @translate
                ],
            ]);
            
        $form->add($fieldset);
    }

```

## Validation

Validation is handled via input filters on the elements. Thus, to modify the server-side validation of forms, use the `Event::SITE_SETTINGS_ADD_INPUT_FILTERS` and `Event::GLOBAL_SETTINGS_ADD_INPUT_FILTERS` events.

```php-inline

    public function addFilters($event)
    {
        $inputFilter = $event->getParam('inputFilter');
        $inputFilter->get('example')->add([
                    'name' => 'example',
                    'required' => false,
                ]);
    }
```


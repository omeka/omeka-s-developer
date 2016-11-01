# Configuration keys

The following keys are used in `module.config.php` files. They are all available via the Config Service. Modules can add their own data. Listed here are the most commonly used keys, and is not an exhaustive list.

Note that many also extensive sublayers of keyed arrays, e.g. [Routes and Navigation](../key_concepts/routes_and_navigation.md). Only the most common and most shallowly layers are represented here.

Factories are typically used when a service needs to be injected into the class being created. If no service needs to be injected, an invokable is used.


* **router** array to merge into existing routes
* **navigation** array to merge into existing navigation
* **controllers** controllers to register
    * factories
    * invokables
* **form_elements**
    * factories
    * invokables
* **view_manager**
    * template_stack_path array of paths to view files, e.g. `[OMEKA_PATH . '/modules/Yourmodule/view']`
* **view_helpers**
    * factories
    * invokables 
* **controller_plugins**
* **entity_manager**
    * mapping_classes_paths array of paths to classes defining Entities, e.g. `[OMEKA_PATH . '/modules/Yourmodule/src/Entity']`
* **api_adapters**
    * factories
    * invokables
* **service_manager**
    * factories
    * invokables
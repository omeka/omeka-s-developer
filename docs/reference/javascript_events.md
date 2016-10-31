# Javascript Events

In the admin interface, the following events use jQuery's trigger function.

## o:sidebar-content-loaded

Triggered in global.js when sidebar content is loaded

## o:resource-selected

Triggered in resource-select.js when a resource is selected from the sidebar.

## o:block-added

Triggered in site-page-edit.js when a block is added to the page.

## o:form-loaded

Triggered in global.js when a form is loaded.

## o:before-form-unload

Triggered in global.js when a form is about to be unloaded, i.e. when moving away from a page.

## o:expanded

Triggered in global.js after toggling from `.collapse` to `.expand`.

## o:collapsed

Triggered in global.js after toggling from `.expand` to `.collapse`.

## o:section-closed

Triggered in global.js after closing a section via `Omeka.switchActiveSection:`.

## o:section-opened

Triggered in global.js after opening a section via `Omeka.switchActiveSection:`.

## o:prepare-value

* **type**: The value's data type
* **value**: The jQuery value object
* **valueObj**: An object containing value data, if any
* **namePrefix**: An indexed prefix that should be used to prefix form element names

Triggered in resource-form.js after making a value on a resource add/edit page. Use to populate the value node for a custom data type.
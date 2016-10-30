# Javascript Events

## o:sidebar-content-loaded

## o:resource-selected

## o:block-added

## o:form-loaded

## o:before-form-unload

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
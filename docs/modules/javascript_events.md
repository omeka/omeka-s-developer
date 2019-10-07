---
title: Javascript Events
---

Omeka S makes extensive use of jQuery, particularly on admin pages. Many user interactions require additional actions to respond to the changes on the page. For this, Omeka S uses the `trigger()` function so other parts of javascript code can follow up.

For example, when the sidebar is opened, a module might need to respond in some way, so opening the sidebar fires a `sidebar-opened` event:

```js
    openSidebar : function(sidebar) {
        sidebar.addClass('active');
        this.reserveSidebarSpace();
        sidebar.trigger('o:sidebar-opened');
    },
```

# Events

---
title: Javascript Events
---

In the admin interface, the following events use jQuery's trigger function.

## Generic Events

*global.js*

### `o:expanded`

Triggered on `.expand` links when they are expanded

### `o:collapsed`

Triggered on `.collapse` links when they are collapsed

## Section (Tab) Events

*global.js*

These events fire when switching between tabs on a multi-tab page. They are triggered on the `.section` element.
They are triggered by `Omeka.switchActiveSection()` and will fire on both user-initiated and programmatic 
section switches. "Open" sections have the class `.active` while closed ones do not.

### `o:section-opened`

Triggered on a `.section` after it is opened

### `o:section-closed`

Triggered on a `.section` after it is closed

## Sidebar Events

*global.js*

All sidebar events are triggered on the sidebar container itself (i.e., the `div.sidebar`).

### `o:sidebar-opened`

Triggered after the sidebar is opened (by `Omeka.openSidebar()`)

### `o:sidebar-closed`

Triggered after the sidebar is closed (by `Omeka.closeSidebar()`)

### `o:sidebar-content-loaded`

Triggered after content is successfully loaded via AJAX by `Omeka.populateSidebarContent()`

## Multi-value Form Field Events

*global.js*

This event applies to `.multi-value` form fields (as seen in the advanced search pages and the batch edit page).

### `o:value-created`

Triggered on a new field after it is created and inserted. 
  
## Resource Form Events

*resource-form.js*

These events apply to the add/edit form for resources (items, item sets, media).

### `o:prepare-value`

* **type**: The value's data type
* **value**: The jQuery object for the `.value` container
* **valueObj**: An object containing value data, if any
* **namePrefix**: An indexed prefix that should be used to prefix form element names

Triggered in resource-form.js after a value input is created or replaced. Use to populate the value node for a custom data type.

## Resource Select Sidebar Events

*resource-selector.js*

These events apply to the resource selection sidebar used on resource forms and in site page edit.

### `o:resource-selected`

Triggered on the `#select-item a` confirmation link when a resource selection is confirmed by the user

Also, triggered on the `.select-resource` link when a resource is initially selected from the list of results

### `o:resources-selected`

Triggered on the `.select-resources-button` link when a multiple-resource "quick" selection is confirmed by the user

## Site Page Edit Form Events

*site-page-edit.js*

This event applies only on the page edit form within a site.

### `o:block-added`

Triggered on a new block form (`.block` container) after it is added to the page.

## Form Change Detection Events

These events are used by the change detection code (to handle warning that their changes have not been saved).
The events only fire on forms with `method` set to `POST`, and will not fire on any form that has the class
`.disable-unsaved-warning` (opting them out of change detection).

### `o:form-loaded`

Triggered on page load on each POST-method `form` present. Triggering this manually resets the saved state of the form
(meaning the current state at the time of the trigger will be considered the "unchanged" state).

### `o:before-form-unload`

Triggered on each form on the page in a handler for the window's `beforeunload`. Calling `Omeka.markDirty()` and passing the form will
trigger the change warning. This is useful for forms that only update the actual on-page form just before submission.

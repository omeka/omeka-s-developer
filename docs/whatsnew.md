# What's New in Omeka S

## Changes in Version 4.0.0

### Configurable resource page blocks

Themes gain a major new feature in Omeka S 4.0.0: configurable resource page
blocks. The content of the resource "show" pages for items, item sets, and
media is no longer monolithically set by the theme author (or the default core
view files), and can be configured by the user. The page can also be split into
multiple regions, enabling sidebars and other structures without hardcoding what
goes where.

Modules can add new blocks to this system, so extra content that previously
would have to be added in a blunt manner at the top or bottom of the page only
with the "before" or "after" events can now be configured by the user and be
placed specifically where they want it (or removed entirely).

More information, particularly geared to theme writers, is available on the
[Configurable Resource Page Blocks page](themes/theme_use_resource_page_blocks.md).

### API updates

The API has several improvements and new features in 4.0.0.

#### New search query options

The `id` query accepted by most resources now allows IDs to be passed a single
comma-delimited string (`1,2,3`) rather than only supporting multiple values as
a PHP array (`id[]=1&id[]=2` in a query string for a REST API request, for
example).

Search for items, media and item sets now accepts:

- `modified_before`, `modified_after`, `created_before`, and `created_after` to
   search on modified and created dates
- For the `property` query, new types `sw` (starts with) and `ew` (ends with)

Search for items now accepts:

- `not_item_set_id` to return only items that are not in the provided set(s)
- `has_media` to return only items that have at least one Media

Search for properties and resource classes now accepts:

- `site_id` to return only those used in at least one item attached to the given
   site

#### Auto property IDs for resource values

When creating Omeka S Resources (items, item sets, media) through the API, the
`property_id` for a value can now be set to the string `auto`, and Omeka S
will create the value by looking up the term from the JSON key the value was
submitted under. Earlier versions of the API always ignored these keys on
input.

For example, in a payload such as

```
{
    "dcterms:title": [
        {
            "type": "literal",
            "property_id": "auto",
            "property_label": "Title",
            "is_public": true,
            "@value": "Example First Title"
        },
    ]
}
```

the `property_id` of `"auto"` will look up the appropriate property ID for the
term `dcterms:title`. This allows creating values without having to first know
in advance or submit an extra API query to retrieve the list of properties and
their IDs.

### Element groups for forms

Omeka S 4.0.0 includes a new "element groups" system for visually grouping and
labeling sets of form elements without using the Laminas "fieldset" element.
It's often more convenient to maintain a "flat" structure to the form, but
still allow for a logical grouping of elements to be presented to the user.

The admin global settings, site settings, and user settings forms have changed
to use this system, so modules that add elements to any of those forms and were
previously using fieldsets will need to update their code.

In a form, the list of all groups is stored in the form option
`element_groups`. The value of the option is an array where the keys are
internal identifiers, and the values are human-readable labels for the groups
(these are translatable strings). Each element can have an `element_group`
option referring to the internal identifier of the group the element should
belong to. These options can be set and modified using the existing events
used for adding elements to forms.

Module authors wanting to use the element groups system in their own forms
must render the form using the helper `formCollectionElementGroups`.

Theme settings as provided in theme INI files also gained access to this
grouping feature, meaning that theme authors can now provide groups for the
options they present to administrators for configuring the theme.

### View helpers

Omeka S 4.0.0 adds several view helpers:

- [browse](modules/view_helpers.md#browse), for handling aspects of rendering
   browse pages that are now configurable, like sorting, and the actual content
   of bowse tables on admin pages
- [currentSite](modules/view_helpers.md#currentsite), to return the
   representation object for the current site
- [iiifViewer](modules/view_helpers.md#iiifviewer), to render an embedded
   Mirador viewer for IIIF
- [lightGalleryOutput](modules/view_helpers.md#lightgalleryoutput), to render a
   gallery of media using lightGallery. Several themes provided by the Omeka
   Team separately included this feature before 4.0.0, and it's now integrated
   in the core and available using the resource page blocks system.
- [resourcePageBlocks](modules/view_helpers.md#resourcepageblocks), to render
   and work with configurable resource page blocks in themes
- [sortMedia](modules/view_helpers.md#sortmedia), to sort a set of media into
   subsets: one set that is supported by lightGallery display, and one that is
   not

### PHP events

#### New events

- `iiif_viewer.mirador_config` allows altering the configuration data passed to
   Mirador when viewing IIIF Presentation API media
- `sort-config` allows altering the set of sorting options that are used to
   present sorting selectors using the new Browse helper and the default
   sorting options available in user and site configuration

#### Changed events

- `view.sort-selector` now passes a `sortConfig` argument instead of `sortBy`,
   and the structure of the array is changed to be a simple one-dimensional
   array with keys being the internal sorting keys used in the API, and values
   being human-readable labels.

## Changes in Version 3.0.0

### Switch from Zend Framework to Laminas

The Zend Framework library that underlies much of Omeka S has changed its name
going forward to [Laminas](https://getlaminas.org/). This change affects the
names of all Zend Framework classes, and so most modules will need to be
updated accordingly.

For most modules, the process is simple: just a find-and-replace for `Zend\`
with `Laminas\` across the module's code. You should find that this
replacement nearly exclusively affects `use` statements, leaving your actual
code mostly unchanged. Configuration keys that referenced "Zend" names are
another common place that would be changed.

After making this change, a module will only be compatible with Omeka S 3.0.0
and up, and not any older versions. Accordingly, the `omeka_version_constraint`
in config/module.ini should be set to `^3.0.0`.

### Resource templates store multiple data types per property

Version 3.0.0 adds a feature that allows resource templates to configure a
property with multiple data types, letting the user choose between them when
adding or editing resources.

This means that the `data_type` column and `dataType` property for the
ResourceTemplateProperty entity are now arrays instead of simple strings, and
the ResourceTemplatePropertyRepresentation class now has new methods
`dataTypes` and `dataTypeLabels` to handle multiple types. The prior "singular"
versions of those methods are still available, but are deprecated: they will
simply return the first data type if multiple are set.

### Event changes

The server-side event `site_settings.form` is removed. The normal
`form.add_elements` and `form.add_input_filters` events work with the site
settings form and were already the preferred option.

A new set of server-side events is added to make it easier for modules to
add elements directly to the new "Advanced" tab for resource forms.
`view.$action.form.advanced` (where `$action` is `add` or `edit` is available
on the item, item set, and media forms.

A new client-side event for resource forms triggers whenever a new property
is added to the form. `o:property-added` will fire when properties are added
on initial load, by the selection of a template, or when a user explicitly
selects a property to add.

### New Gulp tasks

To make it easier to align with the project's code standards, a new Gulp task
is added. `test:module:cs` checks against the code standards for a module,
joining the existing `test:cs` that was used to check the core. Like other
`:module` tasks, you can run this task from within the module's folder or
explicitly pass a module name with `--module-name`.

For both the core and modules, new `fix:cs` and `fix:module:cs` tasks are
also added. Instead of just checking against the code standards, these tasks
make the necessary changes to the codebase to bring it into alignment with
the standards.

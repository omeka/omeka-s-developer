---
title: What's New in Omeka S
---

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

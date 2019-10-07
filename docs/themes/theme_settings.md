---
title: Theme Settings
---

You can provide users with theme settings, configurable parts of your theme that help users customize the theme to their projects. Examples of theme settings include configurable element colors, image asset uploads, footer content.

You create the form fields for your theme settings in `config/theme.ini`. Start with `[config]` as the heading under the `[info]` metadata. The `[config]` heading begins a list of theme settings; you do not need to include it over each individual setting.

```
[info]
name = "Theme Name"

#...

[config]
```

Each theme setting is a form element, so you begin by labeling your setting, providing it with a unique lowercase name, and adding it to the list of elements.

```
[config]
elements.my_theme_setting.name = "my_theme_setting"
elements.my_theme_setting.options.label = "My Theme Setting"
```

Next you need to define your theme setting's form element type.

The available form element types from Zend Framework are found in [their documentation](https://docs.zendframework.com/zend-form/element/intro/). In the config file, reference the element type with the path `Zend\Form\Element\[Type]`. The following example defines the theme setting as a checkbox.

```
elements.my_theme_setting.type = "Zend\Form\Element\Checkbox"
```

Available form elements created for Omeka sit in `application\src\Form\Element`. In the config file, reference the element type with the path `Omeka\Form\Element\[Type]`. The following example defines the theme setting as a select element containing resource templates accessible by the user.

```
elements.my_theme_setting.type = "Omeka\Form\Element\ResourceTemplateSelect"
```

This example demonstrates how to define value options for an element, in this case a radio input.

```
elements.my_theme_setting.type = "Zend\Form\Radio"
elements.my_theme_setting.options.value_options.first = "First Option"
elements.my_theme_setting.options.value_options.second = "Second Option"
elements.my_theme_setting.options.value_options.third = "Third Option"
```

You can also define html attributes for your setting's form element. This example defines the element's id.

```
elements.my_theme_setting.attributes.id = "my_theme_settings"
```

This is an example of a complete theme setting using a radio input.

```
elements.my_theme_setting.name = "my_theme_setting"
elements.my_theme_setting.attributes.id = "my_theme_setting"
elements.my_theme_setting.options.label = "My Theme Setting"
elements.my_theme_setting.type = "Zend\Form\Radio"
elements.my_theme_setting.options.value_options.first = "First Option"
elements.my_theme_setting.options.value_options.second = "Second Option"
elements.my_theme_setting.options.value_options.third = "Third Option"
```

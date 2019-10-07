---
title: Introduction to Themes
---

Omeka S supports themes, a way to customize the presentation of Omeka S sites. Omeka S comes with a default theme, and officially supports the [Center Row](http://github.com/omeka-s-themes/centerrow), [Cozy](http://github.com/omeka-s-themes/cozy), and [The Daily](http://github.com/omeka-s-themes/thedaily) themes. The following is a guide for modifying existing themes as well as starting a theme from scratch.

## Theme Basics

Themes for Omeka S live within the `themes` directory within the root directory of an Omeka S installation. They typically include the following directories and files:

* `config/theme.ini`: This is a **required** file for a theme to be recognized by Omeka S. It includes the theme name, author, support information, theme version, Omeka S version requirements, and theme configuration options for users.
* `asset`: Theme creators use this directory to house assets such as CSS, Javascript, images, and more. It mimics the directory structure of `application/asset` within an Omeka S installation.
* `view`: Files within this directory are the meat of a theme. These customized theme files override Omeka S default view template files located in `application/view/`.

## Exploring Existing Themes

One of the best ways to start working with themes is to experiment with existing themes. Omeka S does not use a consistent pool of CSS selectors across our officially supported themes, but most web browsers provide beginner themers tools for exploring the HTML and CSS of a given site.

[Google Chrome - Inspect and Edit Pages and Styles](https://developers.google.com/web/tools/chrome-devtools/inspect-styles/)

[Mozilla Firefox - Examine and Edit CSS](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_and_edit_CSS)

[Apple Safari - Inspecting Overview](https://support.apple.com/guide/safari-developer/inspecting-overview-dev1a8227029/mac)

## Sass

Themes officially supported by the Omeka team ([the Omeka S default theme](http://github.com/omeka-s-themes/default), [Center Row](http://github.com/omeka-s-themes/centerrow), [Cozy](http://github.com/omeka-s-themes/cozy), and [The Daily](http://github.com/omeka-s-themes/thedaily)), use SASS to generate their CSS. For users who want to start using SASS, here are recommended tutorials for installation. 

* [Sass Basics](http://sass-lang.com/guide)
* [Working with SASS and CSS](../key_concepts/working_with_Sass_and_CSS.md)

Those who simply want to edit the CSS without getting into preprocessors can ignore the 'asset/sass' folder completely and focus on .css files. **Note:** if you edit a .css file but later decide to use Sass, you should back up and make note of your changes before compiling for the first time. Changes made in .scss files overwrite any changes made to .css files.

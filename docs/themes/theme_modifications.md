---
title: Theme Modifications
---

## Overriding default templates

Omeka S has a set of default template files that all themes use, and override when the desired page structure is different from the default.

The default template files are in the folder `/application/view` in your Omeka S installation within the following subfolders: `/application/view/common`, `/application/view/error`, `/application/view/layout`, and `/application/view/omeka/site`. Most views that theme builders need will be in `/application/view/layout` and `/application/view/omeka/site`. Subfolders correspond to the pages that are seen along url patterns. For example, the page displayed at `{YourOmeka SSite}/item/show` is produced by the file in `/application/view/omeka/site/item/show.phtml`.

Themes might or might not override these files. The default theme, for example, has a `layout` directory that overrides one of the default templates: `layout.phtml`.

```
- default/
    - view/
        - layout/
            - layout.phtml
```

If you want to modify a file in a theme, the first place to look is in the theme's own directories. But notice that that will only work if the theme has overridden the default template. In many cases, then, you will need to copy the default template files into the theme, taking care to maintain the directory structure.

So, for example, imagine wanting to modify the show page for items and the browse page for collections in the default theme.

For the show page for items, we need to copy `/application/view/omeka/site/item/show.phtml` 
to `/default/view/omeka/site/item/show.phtml`

For the browse page for item sets, we need to first create the directory: `/default/view/omeka/site/item-set`

Then we can copy `/application/view/omeka/site/item-set/browse.phtml` 
to `/default/view/omeka/site/item-set/browse.phtml`

The result in the default theme will look like::

```
- default/
      - view/
        - layout/
            - layout.phtml
        - omeka/
            - site/
                - item/
                    - show.phtml
                - item-set/
                    - browse.phtml
---
```

## Theme Assets

Non-view theme files live in the theme's `asset/` directory, organized into subdirectories by type. A typical `asset/` directory may look like the following:

  
```
-   asset/
    - css/
    - sass/
    - js/
    - img/
```

  
To refer to a file from the `assets/` directory within a view, use `$this->assetUrl('[subdirectory]/[filename.ext]')`. For example, if you wanted to display a theme's logo image within a view, it might look like this:

`<img src=”<?php echo $this->assetUrl('img/logo.jpg'); ?>” title=”Logo”>`

### Using Custom CSS and Javascript

Every theme has its own version of layout.phtml, which provides the structure for each view. It typically includes:

-   the header,
-   the footer,
-   the call for the page content (`<?php echo $this->content; ?>`),
-   calls to fonts, css, and javascript files.

Themes load their CSS and Javascript files using Zend's view helper functions ([`headLink()`](https://docs.zendframework.com/zend-view/helpers/head-link/) and [`headScript()`](https://docs.zendframework.com/zend-view/helpers/head-script/)), which help control the order in which these files load alongside files from Omeka S's pages and  modules. These functions should appear before the `<head>` of layout.phtml.

Here are examples for including these files in the `<head>` of a theme.

For CSS:

`$this->headLink()->prependStylesheet($this->assetUrl('css/style.css'));`

The function `prependStylesheet()` will place the stylesheet at the top of the queue of loaded css files.

`$this->headLink()->appendStylesheet($this->assetUrl('css/style.css'));`

The function `appendStylesheet()` will place the stylesheet at the bottom of the queue of loaded CSS files.

For Javascript:

`$this->headScript()->prependFile($this->assetUrl('js/thedaily.js'));`

The function `prependFile()` will place the stylesheet at the top of the queue of loaded javascript files.

`$this->headScript()->appendFile($this->assetUrl('js/thedaily.js'));`

The function `appendFile()` will place the javascript file at the bottom of the queue of loaded javascript files.

### Using Webfonts

Themers have access to a greater variety of fonts to use via hosted webfont libraries. Common examples include free resources like [Google Fonts](https://fonts.google.com) and [Adobe Edge Web Fonts](https://edgewebfonts.adobe.com/), as well as paid subscription sites such as [Fonts by Hoefler & Co](https://www.typography.com/webfonts/) and [Mosaic](https://www.monotype.com/fonts/mosaic). Hosted webfont usage often looks like the following:

`<link href="//fonts.googleapis.com/css?family=Lato" rel="stylesheet">`

This line sources the font family “Lato” hosted by Google Fonts. To start using a library's webfont, look for where they provide a similar markup line. This line essentially calls a stylesheet that imports the hosted font files to a page. As such, themers will use the functionality to include a CSS file for the purposes of including a webfont in layout.phtml. For example:

`<?php $this->headLink()->prependStylesheet('//fonts.googleapis.com/css?family=Lato); ?>`

## Working with Omeka S Content

Omeka S's sites are primarily concerned with the presentation of Omeka S resources: items, item sets, and media. Themes control how these resources' associated metadata and files are displayed via methods defined in the resources' representations. The following section highlights some common tasks using these methods, and a more in-depth guide can be found in [Representations](../views/representations.md).

#### Displaying Resource Images

When an Omeka S resource can be visually represented by an associated image, it has a selection of usable thumbnails. Accessing a resource's thumbnail looks like this:

`<?php echo $this->thumbnail($resource, 'thumbnailSize', [/* array of attributes */]); ?>`

This produces HTML to display the primary media's default thumbnail:

`<img src="http://yourdomain.com/omeka-s/files/medium/file.jpg" alt="Description of image.">`

The default thumbnail, "medium", is one of the file derivatives Omeka S generates when files are uploaded to be associated with resources. These file derivatives are the square thumbnail, medium, and large. You can define your preferred file derivative, or even the original file, for the render() function to use. In the following example, the thumbnail type is set to the square thumbnail.

`<?php echo $this->thumbnail($resource, ['thumbnailType' => 'square']); ?>`

The widths of these file derivatives are defined within `application/config/module.config.php`. If an image within your theme appears at a different size than expected, it is most likely due to the theme's CSS.

[Explore more media-specific methods here.](../views/representations.md#media-specific-methods)

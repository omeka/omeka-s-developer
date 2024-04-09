# Page and Block Templates

As a theme developer, you can make customizations more accessible to your users via page and block templates. These templates allow you to override Omeka S's default markup for pages and blocks, and have those appear within the Omeka S admin's page builder interface.

There are two parts required for Omeka S to recognize a theme's template: the theme file must exist within `view/common/page_templates` or `view/common/block_templates` in the theme directory, and the template must be registered in the `theme.ini`.

In the `theme.ini`, the format for registering a block template looks like this:

```
block_templates.[blockName].[template-filename] = "[Template display name]"
```

This is a list of the available core blocks and how they should be referenced in the ini file. In the above example, these would be used for `[blockName]`.

* Asset - `asset`
* Browse preview - `browsePreview`
* HTML - `html`
* Item showcase - `itemShowcase`
* Line break - `lineBreak`
* List of pages - `listOfPages`
* List of sites - `listOfSites`
* Media embed - `media`
* Page date/time- `pageDateTime`
* Page title - `pageTitle`
* Table of contents - `tableOfContents`

The `[template-filename]` is the name of the template in `view/common/block_templates` without the `.phtml` file extension. The `[Template display name]` is what appears in the page builder interface where the user selects the block template.

!!! Example
    For a block template that presents the media embed block as a gallery, you could create a file at `view/common/block_templates/gallery.phtml`. In the `theme.ini`, you would register it by including `block_templates.media.gallery = "Gallery"`.

In the `theme.ini`, the format for registering a page template looks like this:

```
page_templates.[template-filename] = "[Template display name]"
```

Similar to block templates, `[template-filename]` is the name of the template in `view/common/page_templates` without the `.phtml` file extension, while `[Template display name]` is what appears in the page builder interface where the user selects the page template.

!!! Example
    For a page template that provides custom markup for a javascript timeline, you could create a file at `view/common/page_templates/js-timeline.phtml`. In the `theme.ini`, you would register it by including `page_templates.js-timeline = "Javascript timeline"`.

# Modifying Themes

## Overriding default templates

Omeka S has a set of default template files that all themes use, and override when the desired page structure is different from the default.

The default template files are in the folder `/application/views` in your Omeka S installation within the following subfolders: `/application/views/common`, `/application/views/error`, `/application/views/layout`, and `/application/views/omeka/site`. Most views that theme builders need will be in `/application/views/layout` and `/application/views/omeka/site`. Subfolders correspond to the pages that are seen along url patterns. For example, the page displayed at `{YourOmeka SSite}/item/show` is produced by the file in `/application/views/omeka/site/item/show.phtml`.

Themes might or might not override these files. The default theme, for example, has a `layout` directory that overrides one of the default templates: `layout.phtml`.

```
- default/
    - views/
        - layout/
            - layout.phtml
```

If you want to modify a file in a theme, the first place to look is in the theme's own directories. But notice that that will only work if the theme has overridden the default template. In many cases, then, you will need to copy the default template files into the theme, taking care to maintain the directory structure.

So, for example, imagine wanting to modify the show page for items and the browse page for collections in the default theme.

For the show page for items, we need to copy `/application/views/omeka/site/item/show.phtml` 
to `/default/views/omeka/site/item/show.phtml`

For the browse page for collections, we need to first create the directory: `/default/views/omeka/site/collection`

Then we can copy `/application/views/omeka/site/collection/browse.phtml` 
to `/default/collections/browse.phtml`

The result in the default theme will look like::

```
- default/
      - views/
        - layout/
            - layout.phtml
        - omeka/
            - site/
                - item/
                    - show.phtml
                - collection/
                    - browse.phtml
---
```
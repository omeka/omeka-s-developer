---
title: Page Blocks
---

Pages in Omeka S are made up of blocks. All content on a page, from attached
items and media that embed content, to simple plain text, to more complex and
dynamic things, is placed on the page as a block.

Every block has a layout, which controls both what the user creating or editing
the page sees as a user interface and what visitors viewing the public page
see.

Omeka S ships with several block layouts, and modules can also add their own
blocks to give users more options for what kinds of content they can include on
their pages.

## What data is stored in blocks

Blocks, and therefore layouts, deal with two basic types of data that users can
set when building a page.

**Attachments** allow a block to use existing items and/or media from the Omeka
S installation. There's a standardized interface used across different layouts
that allows users to search the item pool and select an item (and possibly a
specific media from that item).

Beyond attachments, blocks can also store **arbitrary data**. Anything and
everything a block needs to store that's not an attachment (like user-provided
HTML, settings or options for the block, or any other kind of data that needs
to get saved) is stored in a single "data" property. Behind the scenes, this
data property is saved as JSON, so layouts can choose whatever keys or other
structures they want to to store that data. Typically, layouts store this data
as a simple string-keyed array.

## Adding a new layout

A block layout takes the form of a single class, which must implement the
`BlockLayoutInterface`. To make things simpler, most blocks will actually
extend from the `AbstractBlockLayout` class instead, which allows you to more
skip implementing some "optional" methods.

### Required methods

#### `getLabel`

`getLabel()` simply returns the human-readable name for the block; it's what
will be shown to the user in the admin interface for a page. The string
returned here gets automatically translated by Omeka S, so you should mark it
with an `// @translate` comment.

#### `form`

`form()` returns the HTML markup for the admin-side form for a block.

Omeka S provides several helpers for common form tasks for attachments
so each new layout doesn't have to reimplement things fron scratch:

- `blockAttachmentsForm` is used to let users select media or items to
  attach. to the block. Layouts that display media content will almost
  always use this helper in their `form()` method. This helper is how a block
  can use the standard attachment UI.

- `blockShowTitleSelect` presents a simple `<select>` control for users
  to choose what text should be displayed as a heading when displaying
  an attachment: the title of the item, the title of the media, or
  nothing. The value selected here is stored in the "data" property
  under the key `'show_title_option'`.

- `blockThumbnailTypeSelect` presents a `<select>` control for users to
  to choose what kind of thumbnail to use when embedding an image for an
  attached piece of media. The value selected here is stored in the "data"
  property under the key "`thumbnail_type'`.

Beyond the attachments, blocks can provide whatever other inputs they want in
this method. Omeka S will automatically handle saving inputs into the "data"
property for the block, as long as they have a `name` attribute that follows
a particular pattern: `o:block[__blockIndex__][o:data][KEY]`, where `KEY` is
replaced with the desired key for the specific piece of data being saved.

Data saved in this way can be accessed from the block using the `dataValue`
method, passing the name of the key to retrieve.

For example, the very simple form for the built-in HTML block layout saves
just one piece of data: the markup input by the user, under the key `html`:

```php
public function form(PhpRenderer $view, SiteRepresentation $site,
    SitePageRepresentation $page = null, SitePageBlockRepresentation $block = null
) {
    $textarea = new Textarea("o:block[__blockIndex__][o:data][html]");
    $textarea->setAttribute('class', 'block-html full wysiwyg');
    if ($block) {
        $textarea->setAttribute('value', $block->dataValue('html'));
    }
    return $view->formRow($textarea);
}
```

Notice that the `<textarea>` input uses the name `o:block[__blockIndex__][o:data][html]`,
and it reads the data to populate the form from the block using
`$block->dataValue('html')`. If you follow this pattern, no additional code is neccesary
to save and retrieve any data value.

#### `render`

`render()` displays a block on the public page. The method recieves `$view` for
accessing the view and using helpers, and the block representation `$block` for
accessing the attachments and other data stored for the block.

Best practice is to write the actual code for displaying the HTML in a
separate view partial, and use the `$view->partial()` helper to render it.
This makes it easy for themes to override the markup for the layout. By
convention, this partial file for a layout is stored at the path
`common/block-layouts/layout-name.phtml`, where `layout-name` is the name
of the layout, in lowercase with words separated by hyphens.

`$block` has two main methods useful for use in a `render()` method:

- `$block->attachments()` returns an array of the attachments for the block
- `$block->dataValue($key, [$default])` returns a piece of arbitrary data with the
  given key. The optional parameter `$default` can be passed to provide a fallback
  value if the data key is not set.

  The entire set of data can be also accessed all at once by calling `$block->data()`.

### Optional methods

Several less-commonly used methods are pre-implemented by the `AbstractBlockLayout`
as blank methods. Layouts can provide these if their particular needs require
them, but you can simply omit them if you're extending from the abstract class.

#### `prepareForm`

This method fires when rendering the page form, just once
for each layout. The typical use for this method is to load assets onto the
page, in particular Javascript files that are used by the layout's form.

Omeka S provides a [Javascript event](../reference/javascript_events.md)
to help run code when a form for a new block is added to the page form,
`o:block-added`.

This method recieves a parameter `$view` to allow the code to easily call
view helpers.

#### `prepareRender`

The equivalent to `prepareForm()` but for the public-side
rendering, this method fires just once for each layout when rendering a page.
Again, its typical use is for loading things like Javascript that's needed
to display the block.

Like `prepareForm()`, this method recieves a `$view` parameter to allow use
of view helpers.

#### `onHydrate`

This method is fired when saving the block, just after the data
is set to the block object. `onHydrate()` can be used to validate or filter the
data. For example, the HTML block uses this method to run the HTML Purifier
filtering library agaisnt the user-provided markup.

When using the provided helpers and following the guidelines given in the
discussion of the `form()` method above, the process of saving the data and
attachments is handled automatically by Omeka S, so blocks don't need to use
this method unless they have particular requirements for validation or
filtering.

It recieves `$block`, the block Entity, to allow reading and setting of the
block's data, and `$errorStore`, for setting errors.

#### `getFulltextText`

Omeka S 2.0.0 added a new "fulltext" search index that
includes content from pages. Since all page content is stored in blocks,
the block layout must determine what data, if any, should be searchable.

Returning text from this method will include it in the index for any page
that includes a block that uses the layout.

User-provided text is the usual type of data that should be returned from
this method; for example, the HTML block returns the text input by the user:

```php
public function getFulltextText(PhpRenderer $view, SitePageBlockRepresentation $block)
{
    return strip_tags($this->render($view, $block));
}
```

### Configuration

A block layout must be registered in the [configuration](config.md) to be
available for use. The configuration is a standard service manager config
section, under the key `block_layouts`. Most blocks don't have any
dependencies on other services, so can simply be registered as invokables:

```php
<?php
namespace MyModule;

use Omeka\Module\AbstractModule;

class Module extends AbstractModule
{
    public function getConfig()
    {
        return [
            'block_layouts' => [
                'invokables' => [
                    'myModuleLayout' => Site\BlockLayout\MyModuleLayout::class,
                ],
            ],
        ];
    }
}
```

The service name (`myModuleLayout` in this example) is used internally to
connect blocks with the layouts they use, and it must be unique across the
entire Omeka S installation and all modules. Best practice is to include the
name of your module at the front of the service name, to avoid possible
conflicts with other modules.

# Resource Page Blocks

Omeka S 4.0 added a new system for users to configure the display of public
resource show pages, the pages used for displaying a single item, item set
or media. Administrators can add, remove, move, and reorder the parts of these
pages. See [the theme documentation](../themes/theme_use_resource_page_blocks.md)
for more information about the default blocks provided by the core and how
themes use them.

Modules can provide these blocks, giving site administrators more options for
content that appears on the resource pages. Omeka S has previously allowed
modules to modify pages with server events like `view.show.before` and
`view.show.after`, but these are not very flexible: when using the events
all module-added content could only appear in one or two places on the page,
typically at the top or bottom.

By comparison, when a module provides its content with a resource page block
layout, a site administrator can choose where the content will appear,
including by placing it above, below, or between core-provided parts of the
page and blocks added by other modules or the theme. Modules using this system
also don't need to create their own settings to enable or disable content
display, as the resource page block system itself allows administrators to
control which blocks they want to use.

## The Layout Class

A module adds a layout by providing a class that implements
`Omeka\Site\ResourcePageBlockLayout\ResourcePageBlockLayoutInterface`. The
interface requires only three methods:

- `getLabel` returns the human-readable label for the block, what the
  administrator will see when configuring blocks in the admin interface
- `getCompatibleResourceNames` returns an array of resources the block is
  compatible with
- `render` returns the markup to display on the page, and it gets passed the
  view object and the representation object for the resource being viewed

Having the `render` method work by rendering a partial is a good practice which
allows a theme to easily modify or override the block.

## Registering the Layout

The config key `resource_page_block_layouts` is used for registering the
available layouts. This is a
[service manager](../configuration/services_and_factories.md) config.

Layouts often have no other dependencies so they use the `invokables` subkey,
but `factories` can also be used if the layout needs other services.

## Example: Mapping Module

The Mapping module uses a resource page block layout for displaying the map
showing the location data assigned to an item or item set.

The layout class is simple and typical of a layout:

```php
<?php
namespace Mapping\Site\ResourcePageBlockLayout;

use Omeka\Api\Representation\AbstractResourceEntityRepresentation;
use Omeka\Site\ResourcePageBlockLayout\ResourcePageBlockLayoutInterface;
use Laminas\View\Renderer\PhpRenderer;

class Mapping implements ResourcePageBlockLayoutInterface
{
    public function getLabel(): string
    {
        return 'Mapping'; // @translate
    }

    public function getCompatibleResourceNames(): array
    {
        return ['items', 'item_sets'];
    }

    public function render(PhpRenderer $view, AbstractResourceEntityRepresentation $resource): string
    {
        return $view->partial('common/mapping-resource-map', ['resource' => $resource]);
    }
}
```

Since the block works for items and item sets, those are both returned in the
array from `getCompatibleResourceNames`. `render` uses a partial rather than
having markup directly in the class.

The layout is registered by a few lines in the module's module.config.php file:

```php-inline
    'resource_page_block_layouts' => [
        'invokables' => [
            'mapping' => Site\ResourcePageBlockLayout\Mapping::class,
        ],
    ],
```

Since the class does not have any dependencies on other services, it uses the
`invokables` subkey.

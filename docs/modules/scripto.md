# Technical Documentation

This documentation is geared to web designers and developers seeking to modify or enhance the Scripto module for their own Omeka S projects.

## Customizing public styles

When using the Scripto public interface in the context of a site, you may want to modify the native styles to better compliment the styles of your site. To do this, save a custom stylesheet using the site's theme name (as it appears in Omeka's `themes/` directory) to the module's `asset/css/site-themes/` directory.

[@todo: document the stylesheet]

## Customizing synced media

When syncing a Scripto project with its corresponding item set, Scripto creates a Scripto item for every Omeka item in the set. By default, Scripto assigns every child media of the Omeka item as the corresponding Scripto item's media. Implementations may want to modify the default behavior. To do this, open `src/Job/SyncProject.php` and replace the contents of `getAllItemMedia()` with custom code that returns an iterable of media entities.

For example, if you only want to assign the first media of the item:

```php
public function getAllItemMedia(Item $item, ScriptoProject $project) {
    $media = $item->getMedia();
    return isset($media[0]) ? $media[0] : [];
}
```

Or, if your media are assigned to another item:

```php
public function getAllItemMedia(Item $item, ScriptoProject $project) {
    $anotherItemId = $this->myGetItemId($item); // custom method
    $anotherItem = $this->getServiceLocator()
        ->get('Omeka\EntityManager')
        ->find('Omeka\Entity\Item', $anotherItemId);
    return $anotherItem->getMedia();
}
```

Note that these changes will apply to every project, so you may need to hard-code conditions depending on your project requirements. For example:

```php
public function getAllItemMedia(Item $item, ScriptoProject $project) {
    if (3 === $project->getId()) {
        // Use custom behavior if the project ID is 3.
        return $this->myGetMedia(); // custom method
    }
    // Otherwise, use default behavior.
    return $item->getMedia();
}
```

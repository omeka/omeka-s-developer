# Copy Resources

## Events

This module provides events where other modules may do something after copies have
been made.

- `copy_resources.*.pre`: Do something before copying a resource. Replace `*` with the resource name. Intended to allow modules to modify the JSON-LD before the resource has been copied. Params:
    - `copy_resources`: The CopyResources service object
    - `resource`: The original resource
    - `json_ld`: The JSON-LD used to copy the resource. Modules may modify the JSON-LD and send it back using `$event->setParam('json_ld', $jsonLd)`
- `copy_resources.*.post`: Do something after copying a resource. Replace `*` with the resource name. Intended to allow modules to copy their data after the resource has been copied. Params:
    - `copy_resources`: The CopyResources service object
    - `resource`: The original resource
    - `resource_copy`: The item set copy

## Copying Sites

Copying sites is more involved than copying other resources because modules may
add data to sites that must also be copied. These modules will need to listen to
the `copy_resources.sites.post` event and make adjustments so their data is correctly
copied over.

Modules that add block layouts and navigation links to sites via the `block_layouts`
and `navigation_links` configuration will need to **revert** the copied layouts and
links to their original state. Thankfully, the `CopyResources` service object has
convenience methods for this. For example:

```php-inline
$sharedEventManager->attach(
    '*',
    'copy_resources.sites.post',
    function (Event $event) {
        $copyResources = $event->getParam('copy_resources');
        $siteCopy = $event->getParam('resource_copy');

        // Revert block layout and link types.
        $copyResources->revertSiteBlockLayouts($siteCopy->id(), 'my_block_layout');
        $copyResources->revertSiteNavigationLinkTypes($siteCopy->id(), 'my_link_type');
    }
);
```

Modules that add API resources that are assigned to sites will need to copy the
resources and assign them to the new site. Again, the `CopyResources` service object
has convenience methods for this. For example:

```php-inline
$sharedEventManager->attach(
    '*',
    'copy_resources.sites.post',
    function (Event $event) {
        $api = $this->getServiceLocator()->get('Omeka\ApiManager');
        $site = $event->getParam('resource');
        $siteCopy = $event->getParam('resource_copy');
        $copyResources = $event->getParam('copy_resources');

        // Create resource copies.
        $myApiResources = $api->search('my_api_resource', ['site_id' => $site->id()])->getContent();
        foreach ($myApiResources as $myApiResource) {
            $preCallback = function (&$jsonLd) use ($siteCopy){
                unset($jsonLd['o:owner']);
                $jsonLd['o:site']['o:id'] = $siteCopy->id();
            };
            $copyResources->createResourceCopy('my_api_resource', $myApiResource, $preCallback);
        }
    }
);
```

Modules may need to modify block layout data and navigation link data to update
for the site copy. Again, the `CopyResources` service object has convenience methods
for this. For example:

```php-inline
$sharedEventManager->attach(
    '*',
    'copy_resources.sites.post',
    function (Event $event) {
        $copyResources = $event->getParam('copy_resources');
        $siteCopy = $event->getParam('resource_copy');

        // Modify block data.
        $callback = function (&$data) {
            $data['foo'] = 'bar';
        };
        $copyResources->modifySiteBlockData($siteCopy->id(), 'my_block_layout', $callback);

        // Modify site navigation.
        $callback = function (&$link) use ($visualizationmMap) {
            $data['baz'] = 'bat';
        };
        $copyResources->modifySiteNavigation($siteCopy->id(), 'my_link_type', $callback);

    }
);
```

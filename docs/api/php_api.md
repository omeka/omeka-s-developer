---
title: PHP API
---

From within Omeka's PHP environment there are three ways to make API requests: directly,
using the API manager service, and indirectly, using the API controller plugin or
the API view helper.

## API Manager

You can perform API operations directly using Omeka's API manager. The API manager
is responsible for authorizing the user, initializing the API request, performing
the operation on the resource, and finalizing the API response. (Note that not all
resources implement every API operation.)

You can access the API manager directly using the service locator (should you have
access to the service locator, of course). Once you have the manager you can perform
the following API operations:

- `search($resource, $data, $options)`: search resources by query
- `read($resource, $id, $data, $options)`: read a resource by ID
- `create($resource, $data, $fileData, $options)`: create a resource
- `batchCreate($resource, $data, $fileData, $options)`: batch create resources
- `update($resource, $id, $data, $fileData, $options)`: update a resource by ID
- `batchUpdate($resource, $ids, $data, $fileData, $options)`: batch update resources by IDs
- `delete($resource, $id, $data, $options)`: delete a resource by ID
- `batchDelete($resource, $ids, $data, $options)`: batch delete resources by IDs

The `$resource` argument is the resource name. For read, update, and delete operations,
the `$id` argument is the resource ID. The `$data` argument is an optional array
of API request parameters. For create and update operations, the `$fileData` argument
is an optional array of file data. The `$options` argument is an optional array
of API request options.

Each operation will return an API response object containing the requested data
and other relavent information. You can get the resultant representation by calling
`getContent()` on the response. For example, to get an item with an ID of 123:

```php
// Where $services is Omeka's service locator object.
$api = $services->get('Omeka\ApiManager');
$response = $api->read('items', 123);
$itemRepresentation = $response->getContent();
```

## Controller plugin

You can access the API in your controllers using the API controller plugin. The plugin
has every operation that the API manager has, plus an additional one
for convenience:

- `searchOne($resource, $data, $options)`: search for one resource by query

For example, to get an item representation via a route parameter in a typical show
action:

```php
public function showAction()
{
    $itemRepresentation = $this->api()->read('items', $this->params('id'))->getContent();
    $view = new ViewModel;
    $view->setVariable('item', $item);
    return $view;
}
```

## View helper

You can access the API manager in your views using the API view helper. The helper
only has the `search()`, `searchOne()`, and `read()` operations. This is prevent
operations that change state from being executed in the view layer.

For example, to get an item in a view template:

```php
<?php $itemRepresentation = $this->api()->read('items', 123); ?>
```

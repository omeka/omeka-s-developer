# API

Omeka S executes most operations through API calls. Let's say you want to get item \#100 from the database:

```php
// Compose the request object
$request = new Request(Request::READ, 'items');
$request->setId(100);
// Execute the request
$response = $apiManager->execute($request);
// Get the representation
$item = $response->getContent();

// The above could be written more concisely (recommended usage)
$item = $apiManager->read('items', 100)->getContent();

// Do something with the representation.
echo json_encode($item);
```

For HTTP clients we provide a RESTful interface on top of the API. An HTTP request for the above example looks something like this:

```
http://your-domain.com/api/items/100
```

The HTTP response will be formatted in [JSON-LD](http://json-ld.org/), a method of transporting Linked Data using JSON. [Example JSON-LD](../reference/json_ld.md)

## API Resources

Resources are registered in configuration and implemented with adapter and representation classes. The adapter is responsible for performing [SCRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations on a resource. A representation is a read-only, serializable object that represents a resource.

Note: API use the plural names of the resource (`items` and not ~~`item`~~) as the first parameter of its methods (or as first component of the URL after API path in RESTful interface).

Omeka S core integrates three RDF resource kinds:
* `items` to CRUD an item, its properties and property values, its class, its template and its media.
* `item-sets` to CRUD an item set, its properties and property values, its class, its template and its attached items.
* `media` to CRUD a medium, its properties and property values, its class, its template and its content file (or value).
* `resources` to perform generic SCRUD operations on resources.

Omeka S API integrates ten other resource adapter:
* `resource_classes` to add, list and look for RDF classes.
* `resource_templates` to add, list and look for resource templates.
* `jobs` to list jobs and their states.
* `modules` to list and look for modules.
* `users` to add and get users.
* `sites` to add, list and look for sites.
* `site_pages` to add and list site pages and managing their blocks.
* `assets` to list and add (~~and [remove](https://github.com/omeka/omeka-s/issues/937)~~) files attached to a site theme.
* `properties` to add, list and look for RDF properties.
* `vocabularies` to add, list and look for RDF ontologies.

## API Operations

### Read

Get one resource.

```php
$response = $apiManager->read('your_resource', 100);
```

```
GET /api/your_resource/100
```

### Search

Search for resources by criteria.

```php
$response = $apiManager->search('your_resource', array('foo' => 'bar'));
```

```
GET /api/your_resource?foo=bar
```

### Create

Create a resource.

```php
$response = $apiManager->create('your_resource', array('foo' => 'bar'));
```

```
POST /api/your_resource
Content-Type: application/json
{"foo":"bar"}
```

### Batch Create

Create resources in batch.

```php
$response = $apiManager->batchCreate('your_resource', array(
  array('foo' => 'bar'),
  array('baz' => 'bat'),
));
```

### Update

Update a resource.

```php
$response = $apiManager->update('your_resource', 100, array('foo' => 'bar'));
```

```
PUT /api/your_resource/100
Content-Type: application/json
{"foo":"bar"}
```

### Delete

Delete a resource.

```php
$response = $apiManager->delete('your_resource', 100);
```

```
DELETE /api/your_resource/100
```

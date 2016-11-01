# API

Omeka S executes most operations through API calls. Let's say you want to get item \#100 from the database:

```php
// Compose the request object
$request = new Request(Request::READ, 'item');
$request->setId(100);
// Execute the request
$response = $apiManager->execute($request);
// Get the representation
$item = $response->getContent();

// The above could be written more concisely (recommended usage)
$item = $apiManager->read('item', 100)->getContent();

// Do something with the representation.
echo $item->jsonSerialize();
```

For HTTP clients we provide a RESTful interface on top of the API. An HTTP request for the above example looks something like this:

```
http://your-domain.com/api/items/100
```

The HTTP response will be formatted in [JSON-LD](http://json-ld.org/), a method of transporting Linked Data using JSON. [Example JSON-LD](../reference/json_ld.md)

## API Resources

Resources are registered in configuration and implemented with adapter and representation classes. The adapter is responsible for performing [SCRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations on a resource. A representation is a read-only, serializable object that represents a resource.

## API Operations

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

### Read

Get one resource.

```php
$response = $apiManager->read('your_resource', 100);
```

```
GET /api/your_resource/100
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

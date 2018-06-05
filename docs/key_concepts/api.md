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

A full list of available resources is available at `api/api_resources`.

## API Response

Each API operation return a Response object which has three properties: `request` (Request object created by the API), `totalResults` (total result number) and `content` (RepresentationInterface object or array of ResourceReference for batch operations). Each one has a public getter.

RESTful interface response is a JSON serialization of the got representation, in the same manner as this code:
```php
//$response is a Response object (return value of an API operation)
$representation = $response->getContent();
echo json_encode($representation);
```

Instead of a representation, you can get an entity or a reference, passing `'responseContent'` option. This can have three different values: `'resource'` (entity), `'reference'` (references only have their ID hydrated) or `'representation'` (by default).

## API Operations

`api.execute.pre` and `api.{operation}.pre` events are triggered before each opertion, and after each one, `api.{operation}.post` and `api.execute.post` are triggered (where {operation} is the API used-method name).

You can disable these events triggering by passing boolean options `initialize` and `finalize` respectively for pre events and post events, with `false` as value.

### Operation options

Each operation can be configured with several options. Actually, API allow to pass an option array as the **last method parameter**.

Note that some operations (like [create](#create) and [read](#read)) can have a third param (file data), even a fourth parameter for [update](#update) operation, so not needed parameters should be empty arrays to ensure the option array is the last paramater.

To read item #100 with `'initialize'` option set to `false`, use:
```php
$response = $apiManager->read('items', 100, [], ['initialize' => false]);
```

Operation(s)	| Option	| Value type	| Default value	| Description	|
---------	| --------	| ----------	| ---------	| ----------	|
all		| initialize	| boolean	| `true`	| Set whether to trigger API-pre events |
all		| finalize	| boolean	| `true`	| Set whether to trigger API-post events and transform response content according to the "responseContent" option |
all		| responseContent| string	| `'representation'`| Set the type of content the API response should contain. The types are:<ul><li>`'representation'`: an API resource representation (implements `Omeka\Api\Representation\RepresentationInterface`)</li><li>`'reference'`: an API resource reference (instance of `Omeka\Api\Representation\ResourceReference`)</li><li>`'resource'`: an API resource (implements `Omeka\Api\ResourceInterface`)</li></ul> |
[search](#search)| returnScalar	| string	| `false`	| Set which field/column to return as an array of scalars. The request will not finalize when this option is set. |
[update](#update)| isPartial	| boolean	| `false`	| Set whether this is a partial UPDATE request (aka PATCH). |
[update](#update)| collectionAction| string	| `'replace'`	| Set which action to take on certain collections during a partial UPDATE request. The actions are: <ul><li>`'replace'`: the passed data replaces the collection</li><li>`'append'`: append passed data to collections</li><li>`'remove'`: remove passed data from collections</li></ul>
batch ([create](#batch-create), [update](#batch-update), [delete](#batch-delete))| continueOnError| boolean| `false`| Set whether a batch operation should continue processing on error. |
[create](#create), [update](#update), [delete](#delete)| flushEntityManager| boolean| `true`| Set whether to flush the entity manager.


### Read

Get one resource. The second paramater is the wanted-resource Id. 

For example, to get medium #101 representation:

```php
$response = $apiManager->read('media', 101);
```

```
GET /api/media/101
```

Note: `api.find.post` event is triggered.

### Search

Search for resources by criteria. The second parameter is an associative array where keys are critera and values are criterion values.

For example, to look for items which have `Person` RDF class (in any vocabulary):
```php
$personResources = $apiManager->search('items', array('resource_class_label' => 'Person'))->getContent();
```

```
GET /api/items?resource_class_label=Person
```
#### Search criteria

##### Pagination controls
Search results are paginated in pages of 25 results by default.

Links to additional pages are given in the `Link` header. The link with `rel="next"` is the next page in the sequence.
When appropriate, links with a `rel` value of `prev`, `first`, and `last` are also provided.

Pagination parameters are passed like any other criteria
* `page`: Page number of the result set to return
* `per_page`: Number of results per page

##### Common criteria
* `owner_id`: ID of the user who is the owner of the wanted resources.

##### Common criteria for RDF resources
* `search`: value to search in any RDF properties.
* `resource_class_label`: label of RDF class (do not prefix the vocaluary).
* `resource_class_id`: ID of RDF class.
* `resource_template_id`: ID of a resource template.
* `is_public`: whether the resource is public or private (boolean value).

###### RDF property criteria
Your request data can specify the `property` key. It should be associated to a list (array) of criteria on RDF properties.

Each critrion is an associative array with at least `property` and `type` keys (respectively the RDF property ID and the type of operator). 

Possible values for `'type'` are:
* `eq`: is exactly
* `neq`: is not exactly
* `in`: contains
* `nin`: does not contain
* `ex`: has any value
* `nex`: has no value

A `text` key is associated to criterion value.

When you use several criteria, the `joiner` key should be set to `'and'` or `'or'`.

##### Criteria for items
* `id`: item ID
* `item_set_id`: ID of an item set which contains the item, or list (array) of item set IDs.
* `site_id`: ID of a site which has the searched item is in its item pool.

##### Criteria for media
* `id`: medium ID.
* `site_id`: ID of a site where a page contains the medium.

##### Criteria for item sets
* `is_open`: whether the current user can assign new items to the item set.
* `site_id`: ID of a site where the searched item set is in its item set pool.

##### Criteria for vocabularies
* `namespace_uri`: namespace URI of the vocabulary.
* `prefix`: prefix of the vocabulary.

##### Criteria for properties and resource classes
* `vocabulary_id`: ID of the vocabulary which contains the wanted properties.
* `vocabulary_namespace_uri`: URI of the namespace URI of the vocabulary which contains the wanted properties.
* `vocabulary_prefix`: prefix of the vocabulary which contains the wanted properties.
* `local_name`: label of the property (without the vocabulary prefix).
* `term`: full label (`vocabulary_prefix:local_name`)

##### Criteria for users
* `email`: email of the user

##### Criteria for resource templates
* `label`: Given name of the resource template.

##### Criteria for jobs
* `class`

### Create

Create a resource. The second parameter is the resource-to-create representation (PHP associative array in JSON-LD style).

For example, to create an item called “My item title”:
```php
$data = [
	"dcterms:title" => [[
		"type"=> "literal",
		"property_id"=> 1,
		"@value"=> "My item title"
	]],
];
$response = $apiManager->create('items', $data);
```

```
POST /api/items
Content-Type: application/json
{
  "dcterms:title":[
    {
      "type": "literal",
      "property_id": 1,
      "@value": "My item title"
    }
  ]
}
```
#### Attach a file

You can upload a file and attach it to a new media resource using the third parameter of a create request.

A `'o:ingester'` key is required in resource data (second parameter) with `'upload'` as value. You must set a `'file_index'` key too with an arbitrary value (e.g. `0`), indexing upload data in file data (third parameter).

```php
$fileIndex=0;
$data=[ //Representation of the medium which will be created
	"o:ingester" => "upload",
	"file_index" => $fileIndex,
	"o:item" => [
		"o:id" => 100 //Id of the item to which attach the medium
	],
];
$fileData = [
	'file'=>[
		$fileIndex => $_FILES['fileInputName'],
	],
];
$response = $this->api->create('media', $data, $fileData);
```

Note: you need to post the file (e.g. using a form) to allow the script to get it using `$_FILES`. Note that you should use Zend <code>[Request](https://docs.zendframework.com/zend-http/request/)->getFiles()->toArray()</code> instead of directly `$_FILES`.

### Batch Create

Create resources in batch. The second paramater is a list (PHP array with numerical indexes) of representations of resources to create.

You can replace this code:

```php
$representations = [
	$this->api->create('your_resource', $data1)->getContent(),
	$this->api->create('your_resource', $data2)->getContent(),
];
```

by the following:

```php
$references = $apiManager->batchCreate('your_resource', [$data1, $data2] )->getContent();
```

However, batch operations return resource references instead of resource representations. From a reference you can only get entity ID.

In Response content, the reference keys are the same as their representations in request content.

### Update

Update a resource. The second parameter is the ID of the resource which will be updated. The third parameter is the new representation of the resource.

For example, to turn item #100 title into “My new item title”:
```php
$data = [
	"dcterms:title" => [[
		"type"=> "literal",
		"property_id"=> 1,
		"@value"=> "My new item title"
	]],
];
$response = $apiManager->update('items', 100, $data);
```

```
PUT /api/items/100
Content-Type: application/json
{
  "dcterms:title":[
    {
      "type": "literal",
      "property_id": 1,
      "@value": "My new item title"
    }
  ]
}
```

By default, all previous properties are removed. In the previous example, all properties of item #100 will be removed (except dcterms:title). To preserve previous properties and to append new ones from given representation, you must set `'isPartial'` option to `true` and `'collectionAction'` to `'append'`.

If you pass `['isPartial' => true, 'collectionAction' => 'remove']` as options, the data from the given represenation are removed from the updated item instead of to be appended.

### Delete

Delete a resource. The second parameter is the resource-to-delete ID.

```php
$response = $apiManager->delete('your_resource', 100);
```

```
DELETE /api/your_resource/100
```

`api.find.query` and `api.find.post` events are triggered.

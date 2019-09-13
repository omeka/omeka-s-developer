---
title: REST API
---

Omeka S provides an API for external clients to query and modify Omeka S installations.

## Endpoint

The API is located at `/api` under the root of the Omeka S installation.

## Format

HTTP responses will be formatted in [JSON-LD](http://json-ld.org/), a method of transporting Linked Data using JSON.

Payloads for requests should also be JSON-LD, but Omeka S sometimes requires clients to follow a
particular structure, even if the same data could be represented by alternate valid JSON-LD structures.
The Content-Type of a request with a payload must be `application/json` (or `multipart/form-data` if sending a multipart request). Format specifers (after `+`, i.e., `application/json+ld`, are also allowed).

## Authentication

The API permits anonymous access to public resources (i.e., reading non-private data).
To perform actions or view data that only logged-in users can access,
you must authenticate your requests.

Authentication requires every request to include two additional GET parameters: `key_identity` and `key_credential`.
An Omeka S user can create API keys in the "API keys" tab of their user edit page.

## API Resources

API resources listed as allowing anonymous access allow the search and read
operations. Some results may not be available to anonymous clients (e.g.,
resources marked private).

URL | Anonymous Access
--- | ---
`items` | Yes
`item_sets` | Yes
`media` | Yes
`resources` | Yes
`properties` | Yes
`vocabularies` | Yes
`resource_classes` | Yes
`resource_templates` | Yes
`sites` | Yes
`site_pages` | Yes
`assets` | Yes
`jobs` | No
`modules` | No
`users` | No

The create, update, and delete operations always require authentication.

Modules may add their own resources. A full list of available resources for an installation
is available at its own resource: `api_resources`.

## API Operations

### Read

```http
GET /api/:api_resource/:id
```

Get one resource by a known ID. The `id` parameter is mandatory.

### Search

```http
GET /api/:api_resource
```

Search or list resources, optionally specifying criteria.

Parameters to filter, limit, or alter the search results should be passed as GET parameters
in the query string.

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

```http
POST /api/:api_resource
```

Create a resource.

A JSON payload is required.

#### Upload files

For create operations that invlove uploading files, multipart requests (Content-Type
`multipart/form-data`) are supported.

In a multipart request, the normal JSON request body should be specified as a "field" with the name
`data`.

The `asset` API resource expects the associated file to be named `file`.

To upload media (as part of a `media` create operation or `item` create or update operation),
files are expected to be named as an "array" (i.e., `file[0]`, `file[1]`). The JSON body for the
media or item will specify the index into this array with the `file_index` key.

```
"o:ingester": "upload",
"file_index": 0,
...
```

### Update

```http
PUT /api/:api_resource/:id
```

Update a resource. The `id` URL parameter is required.

A JSON payload is required.

The payload completely updates/replaces the existing resource with the given ID. Therefore, if
making changes, it's typically wise to read the current state of the resource, alter the returned
JSON, and submit that as the update. Alternatively, you can perform a "partial" update.

### Partial Update (Patch)

```http
PATCH /api/:api_resource/:id
```

Update a resource, preserving data in non-specified keys. The `id` URL parameter is required.

A JSON payload is required.

Note that specifying _any_ values for RDF properties (the "values" for items, item sets, and media)
will be treated as an "update" to the values generally (meaning that specifying any value for any
RDF property will mean the removal of all other values that aren't passed). In other words, the values
are treated as if they're one big "key" for the purposes of patching.

### Delete

```http
DELETE /api/:api_resource/:id
```

Delete a resource. The `id` URL parameter is required.

---
title: API Reference
---

## API search parameters
---

You can filter a search and control its output by passing search parameters in the
`$data` argument (PHP API) or in the URL query string (REST API).

### Common parameters

These are the search parameters that are common to almost all API resources:

| Parameter | Description | Type |
| --- | --- | --- |
| sort_by | Sort the result set by this field | string | 
| sort_order | Sort the result set in this order, ascending ("asc") or descending ("desc") | string |
| page | The page number of the result set to return | integer |
| per_page |  The number of results per page | integer |
| limit | The number of results to return | integer |
| offset | The number offset of results to return | integer |

### Parameters for RDF resources

These are the search parameters that are common to all RDF resources (items, media,
item_sets):

| Parameter | Description | Type |
| --- | --- | --- |
| fulltext_search | Get RDF resources where there's a match in the fulltext index | string |
| search | Get RDF resources where there's an exact match in a value | string |
| id | Get the RDF resource that has this unique identifier | integer |
| owner_id | Get RDF resources that belong to this owner | integer |
| resource_class_label | Get RDF resources with a class that has this unique label | string |
| resource_class_id | Get RDF resources with a class that has this unique identifier | integer |
| resource_template_id | Get RDF resources with a template that has this unique identifier | integer |
| is_public | Get RDF resources that are public OR private | boolean |

RDF resources also feature a fine-tuned value search using this query format:

- `property[{index}][joiner]`: "and" OR "or" joiner with previous query
- `property[{index}][property]`: property ID
- `property[{index}][text]`: search text
- `property[{index}][type]`: search type
    - eq: is exactly
    - neq: is not exactly
    - in: contains
    - nin: does not contain
    - ex: has any value
    - nex: has no value
    - res: is this RDF resource (by ID)
    - nres: is not this RDF resource (by ID)

### Parameters for items

| Parameter | Description | Type |
| --- | --- | --- |
| item_set_id | Get items assigned to this item set | integer |
| site_id | Get items in this site's item pool | integer |
| site_attachments_only | When using site_id, whether items must be attached to a site page block | bool |

### Parameters for media

| Parameter | Description | Type |
| --- | --- | --- |
| item_id | Get media assigned to this item | integer |
| media_type | Get media of this media type | string |
| site_id | Get media in this site's item pool AND are attached to a block | integer |

### Parameters for item_sets

| Parameter | Description | Type |
| --- | --- | --- |
| is_open | Get item sets that are open or closed | bool |
| site_id | Get item sets in this site's item set pool | integer |

### Parameters for vocabularies

| Parameter | Description | Type |
| --- | --- | --- |
| owner_id | Get vocabularies belonging to this owner | integer |
| namespace_uri | Get a vocabulary that has this unique namespace URI (e.g. "http://purl.org/dc/terms/") | string |
| prefix | Get a vocabulary that has this unique namespace prefix (e.g. "dcterms") | string |

### Parameters for resource_classes

| Parameter | Description | Type |
| --- | --- | --- |
| owner_id | Get classes that belong to this owner | integer |
| vocabulary_id | Get classes that belong to a vocabulary that has this unique identifier | integer |
| vocabulary_namespace_uri | Get classes that belong to a vocabulary that has this unique namespace URI (e.g. "http://purl.org/dc/dcmitype/") | string |
| vocabulary_prefix | Get classes that belong to a vocabulary that has this unique namespace prefix (e.g. "dcmitype") | string |
| local_name | Get classes with this local name (e.g. "Image") | string |
| term | Get a class with this unique term (e.g. "dcmitype:Image") | string |

### Parameters for properties

| Parameter | Description | Type |
| --- | --- | --- |
| owner_id | Get properties that belong to this owner | integer |
| vocabulary_id | Get properties that belong to a vocabulary that has this unique identifier | integer |
| vocabulary_namespace_uri | Get properties that belong to a vocabulary that has this unique namespace URI (e.g. "http://purl.org/dc/terms/") | string |
| vocabulary_prefix | Get properties that belong to a vocabulary that has this unique namespace prefix (e.g. "dcterms") | string |
| local_name | Get properties with this local name (e.g. "title") | string |
| term | Get a property with this unique term (e.g. "dcterms:title") | string |

### Parameters for users

| Parameter | Description | Type |
| --- | --- | --- |
| email | Get a user that has this unique email | string |
| name | Get users that have this name | string |
| role | Get users that have this role (choices are: global_admin, site_admin, editor, reviewer, author, researcher) | string |
| is_active | Get users that are active or inactive | bool |
| site_permission_site_id | Get users that have site permissions | integer |

## API request options
---

For most resources, you can pass options that affect the execution of a request
in the `$options` argument. These options are only available when using the PHP
API, and are not available in the REST API.

| Option | Type | Description | Default |
| --- | --- | --- | --- |
| initialize | bool | Set whether to initialize the request during execute() (e.g. trigger API-pre events). | `true` |
| finalize | bool | Set whether to finalize the request during execute() (e.g. trigger API-post events and transform response content according to the "responseContent" option). | `true` |
| returnScalar | string | Set which field/column to return as an array of scalars during a SEARCH request. The request will not finalize when this option is set. | `false` |
| isPartial | bool | Set whether this is a partial UPDATE request (aka PATCH). | `false` |
| collectionAction | string | Set which action to take on certain collections during a partial UPDATE request:<ul><li>`"replace"`: the passed data replaces the collection</li><li>`"append"`: append passed data to collections</li><li>`"remove"`: remove passed data from collections</li></ul> | `"replace"` |
| continueOnError | bool | Set whether a BATCH_CREATE operation should continue processing on error. | `false` |
| flushEntityManager | bool | Set whether to flush the entity manager during CREATE, UPDATE, and DELETE. | `true` |
| responseContent | string | Set the type of content the API response should contain. Default is "representation". The types are:<ul><li>`"representation"`: an API resource representation</li><li>`"reference"`: an API resource reference</li><li>`"resource"`: an API resource</li></ul> | `"representation"` |


---
title: API Reference
---

Pagination parameters are passed like any other criteria.

* `page`: Page number of the result set to return
* `per_page`: Number of results per page

## Common criteria
* `owner_id`: ID of the user who is the owner of the wanted resources.

## Common criteria for RDF resources
* `search`: value to search in any RDF properties.
* `resource_class_label`: label of RDF class (do not prefix the vocaluary).
* `resource_class_id`: ID of RDF class.
* `resource_template_id`: ID of a resource template.
* `is_public`: whether the resource is public or private (boolean value).

### RDF property criteria
Your request data can specify the `property` key. It should be associated to a list (array) of criteria on RDF properties.

Each criterion is an associative array with at least `property` and `type` keys (respectively the RDF property ID and the type of operator). 

Possible values for `'type'` are:

* `eq`: is exactly
* `neq`: is not exactly
* `in`: contains
* `nin`: does not contain
* `ex`: has any value
* `nex`: has no value

A `text` key is associated to criterion value.

When you use several criteria, the `joiner` key should be set to `'and'` or `'or'`.

## Criteria for items
* `id`: item ID
* `item_set_id`: ID of an item set which contains the item, or list (array) of item set IDs.
* `site_id`: ID of a site which has the searched item is in its item pool.

## Criteria for media
* `id`: medium ID.
* `site_id`: ID of a site where a page contains the medium.

## Criteria for item sets
* `is_open`: whether the current user can assign new items to the item set.
* `site_id`: ID of a site where the searched item set is in its item set pool.

## Criteria for vocabularies
* `namespace_uri`: namespace URI of the vocabulary.
* `prefix`: prefix of the vocabulary.

## Criteria for properties and resource classes
* `vocabulary_id`: ID of the vocabulary which contains the wanted properties.
* `vocabulary_namespace_uri`: URI of the namespace URI of the vocabulary which contains the wanted properties.
* `vocabulary_prefix`: prefix of the vocabulary which contains the wanted properties.
* `local_name`: label of the property (without the vocabulary prefix).
* `term`: full label (`vocabulary_prefix:local_name`)

## Criteria for users
* `email`: email of the user

## Criteria for resource templates
* `label`: Given name of the resource template.

## Criteria for jobs
* `class`


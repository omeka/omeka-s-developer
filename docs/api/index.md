---
title: Introduction to the API
---

Omeka S provides an application programming interface (API) that enables [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)
operations on its resources. API resources are objects with a type, associated data,
relationships to other resources, and sets of methods that operate on them. There
are two ways to access Omeka's API:

- Programmatically, from within Omeka's PHP environment (see the [PHP API documentation](php_api))
- Using Omeka's [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) web service (see the [REST API documentation](rest_api))

## Operations

You can perform these API operations on almost every API resource:

| Operation | Description |
| --- | --- |
| search | Get a list of resources given query parameters |
| read | Get an individual resource given a unique identifier |
| create | Create a new resource |
| update | Update an existing resource given a unique identifier |
| delete | Delete an existing resource given a unique identifier |

## Resources

Omeka comes with the following API resources:

| Resource | Description |
| --- | --- |
| users | Users who have login credentials |
| vocabularies | RDF vocabularies imported into Omeka |
| resource_classes | RDF classes that belong to vocabularies |
| properties | RDF properties that belong to vocabularies |
| items | Item RDF resources, the building blocks of Omeka |
| media | Media RDF resources that belong to items |
| item_sets | Item set RDF resources, inclusive set of items |
| resource_templates | Templates that define how to describe RDF resources |
| sites | Omeka sites, the public components of Omeka |
| site_pages | Pages within sites |
| modules | Modules that extend Omeka fuctionality (search and read access only) |
| api_resources | API resources available on this install (search and read access only) |

Do not confuse "API resources" and "RDF resources." RDF resources are resources
that can be described and semantically linked using RDF vocabularies. The only RDF
resources in Omeka are items, media, and item sets. (See [Resource Description Framework](https://en.wikipedia.org/wiki/Resource_Description_Framework).)

# REST API Reference

## Resource Values

The main "resource" types (item, item set, media) have store their user-editable
metadata as "resource values." 

Every value has an associated [data type](../modules/data_types.md) that governs
how the value is displayed in the web UI, how it is stored in the database,
and how it is rendered to and ingested from JSON-LD.

In the Omeka S API's JSON-LD, resource values are represented as "top-level" arrays
of objects, where each object represents a single value:

```json
{
    "dcterms:title": [
        {
            "type": "literal",
            "property_id": 1,
            "property_label": "Title",
            "is_public": true,
            "@value": "Example First Title"
        },
        {
            "type": "literal",
            "property_id": 1,
            "property_label": "Title",
            "is_public": true,
            "@value": "Example Second Title"
        }
    ]
}
```

The values output by the REST API are grouped by property
(note the top-level `dcterms:title` key in the example above).

### Common Value Properties

Several object properties are common to all data types:

- `type`: (**required**, string) Declares the data type of the value. 
  Built-in types are `literal`, `uri`, `resource`, `resource:item`,
  `resource:itemset`, `resource:media`, and others can be added by modules.
- `property_id`: (**required**, number or string) Internal Omeka S ID for the property
  this value is associated with. Property IDs can be reused from API output or
  discovered via the `api/properties` endpoint. Since Omeka S 4.0.0, passing the string
  `auto` here will use the property matching the term used in the containing array's key
  (i.e, in the example above, it will look up and use the correct property ID for
  `dcterms:title`)
- `is_public`: (boolean) Whether this value is marked as private (only visible
  to the resource's creator and to users with high-level permissions). Values
  will be marked public if this property is not provided on input.
- `property_label`: (read-only, string) The human-readable name of the associated property.
  This is included in the JSON output for the convenience of consumers but is
  ignored completely on incoming data.

### Literal Value Properties

Literal values (type `literal`)  are plain text.

- `@value`: (**required**, string) The text of the value.
- `@language`: (string) The language of this text.
  Languages should be specified as [BCP 47](https://www.w3.org/International/articles/language-tags/) codes.

### URI Value Properties

URI values (type `uri`) repesent links to other resources external to the Omeka S system.

- `@id` (**required**, string) The URI being linked to.
- `o:label`: (string) A human-readable label for the link, which will be
  displayed as the link text in the web UI.

### Resource Value Properties

Resource values represent links to other resources within the same Omeka S
installation. The main type `resource` can be a link to an item or a set,
while `resource:item`, `resource:itemset`, and `resource:media` constrain the
value to linking to only items, item sets, or media, respectively.

- `value_resource_id` (**required**, number) The internal Omeka S ID of the
  resource being linked to. Resource IDs are exposed as `o:id` in the API
  output for all resources.

Only `value_resource_id` is used as input to the API, but several more
properties are included in the output for consumer convenience:

- `@id` (read-only, string) The API URL for the linked resource.
- `value_resource_name` (read-only, string) The name of the kind of linked resource.
- `display_title` (read-only, string) The title of the linked resource.
- `thumbnail_url`, `thumbnail_title`, `thumbnail_type` (read-only, string)
  Only present when the linked resource has an associated piece of media. The URL
  for the square thumbnail, the title of the media for the thumbnail, and the
  type of that media, respectively.
- `url` (read-only, string) The URL of the resource. This value is `null` in
  API output; an actual URL is only provided when JSON output is embedded in
  a web page view.

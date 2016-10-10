Request:

```
http://your-domain.com/api/items/3
```

Response:

    {
      "@context": {
        "o": "http://omeka.org/s/vocabulary#",
        "foaf": {
          "@id": "http://xmlns.com/foaf/0.1/",
          "vocabulary_id": 4,
          "vocabulary_label": "Friend of a Friend"
        },
        "dcterms": {
          "@id": "http://purl.org/dc/terms/",
          "vocabulary_id": 1,
          "vocabulary_label": "Dublin Core"
        }
      },
      "@id": "http://your-domain.com/api/items/3/",
      "o:id": 3,
      "@type": "foaf:Person",
      "o:owner": {
        "@id": "http://your-domain.com/api/users/1/",
        "o:id": 1
      },
      "o:resource_class": {
        "@id": "http://your-domain.com/api/resource_classes/94/",
        "o:id": 94
      },
      "o:resource_template": {
        "@id": "http://your-domain.com/api/resource_templates/1/",
        "o:id": 1
      },
      "o:created": {
        "@value": "2015-01-16T08:09:09-05:00",
        "@type": "http://www.w3.org/2001/XMLSchema#dateTime"
      },
      "o:modified": {
        "@value": "2015-01-21T16:16:56-05:00",
        "@type": "http://www.w3.org/2001/XMLSchema#dateTime"
      },
      "o:media": [
        
      ],
      "o:item_set": [
        {
          "@id": "http://your-domain.com/api/item_sets/1/",
          "o:id": 1
        }
      ],
      "foaf:name": [
        {
          "@value": "Joan Of Arc",
          "value_is_html": false,
          "value_id": 3,
          "property_id": 138,
          "property_label": "name"
        },
        {
          "@value": "Jeanne d'Arc",
          "value_is_html": false,
          "value_id": 15,
          "property_id": 138,
          "property_label": "name"
        }
      ],
      "dcterms:description": [
        {
          "@value": "Joan of Arc has become a world famous icon from 1412 France. While living she was instrumental in the Hundred Years War and after she passed she became a Saint in 1920.",
          "value_is_html": false,
          "value_id": 4,
          "property_id": 4,
          "property_label": "Description"
        }
      ]
    }

# Meta Group

Meta groups are typically entries within a metadata set. This can be properties within resource metadata, meta for a site, most scenarios where a heading with values are being displayed.

![A basic grouping of metadata, with "Class" as the label and "Person" as the value.](../img/meta_group.jpg)

```
<div class="meta-group">
    <dt>Class</dt>
    <dd class="value">Person</dd>
</div>
```

With resource properties, include CSS classes for the value type. The default in the core include "literal," "resource", and "uri". Include the language code in a `lang` attribute if applicable.

```
<div class="property">
    <dt>Title</dt>
    <dd class="value literal" lang="en">Georgia Gazette</dd>
</div>
```
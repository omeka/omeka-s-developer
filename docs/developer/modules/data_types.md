# Data Types

Data types are classes that extend the ways users can describe resources. They define
the entire round trip of a resource value: how it's entered and validated, saved
to the database, and rendered to the screen. It's important to know that *every*
value has a data type, and that setting the data type for a property in a resource
template only affects *newly* created values.

## Built-in Data Types

Omeka S comes with several commonly-used data types:

- **Literal**: Describe a resource using a literal string;
- **URI**: Describe a resource using a [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier);
- **Resource**:
    - **All**: Describe a resource using any Omeka S resource;
    - **Item**: Describe a resource using an Omeka S item;
    - **Item Set**: Describe a resource using an Omeka S item set;
    - **Media**: Describe a resource using an Omeka S media.

With these data types, you could describe a book like this:

|Subject|Predicate|Object|Data Type|
|---|---|---|---|
|This book|`dcterms:title`|"I Know Why the Caged Bird Sings"|Literal|
|This book|`bibo:uri`|[https://www.wikidata.org/wiki/Q3163506](https://www.wikidata.org/wiki/Q3163506)|URI|
|This book|`dcterms:creator`|["Maya Angelou"]() (link to item)|Item|

Or, when viewing the JSON-LD representation of the example above:

```json
{
    "@context": "http:\/\/example.com\/api-context",
    "@id": "http:\/\/example.com\/api\/items\/1",
    "@type": "o:Item",
    "dcterms:title": [
        {
            "type": "literal",
            "@value": "I Know Why the Caged Bird Sings",
            "@language": "en"
        }
    ],
    "bibo:uri": [
        {
            "type": "uri",
            "@id": "https:\/\/www.wikidata.org\/wiki\/Q3163506",
            "o:label": "Wikidata"
        }
    ],
    "dcterms:creator": [
        {
            "type": "resource",
            "value_resource_id": 1,
        }
    ]
}
```

Note the keys `@value`, `@language`, `@id`, `o:label`, and `value_resource_id`.
They will be helpful when writing your data type's value markup (see below).

## Interface

Every data type class must implement `DataTypeInterface`. The required methods are:

|Method|Purpose|Comment|
|---|---|---|
|`getName()`|Returns the name of the data type.|It must be unique across all data types. It's best to prefix the name with the name of your module.|
|`getLabel()`|Returns a human-readable label for the data type.|It should be unique across all data types.|
|`getOptgroupLabel()`|Returns a human-readable optgroup label for the data type, if any.|Typically used when a module has more than one data type.|
|`prepareForm()`|Prepares the view to enable the data types.|Typically used to add stylesheets and scripts needed to handle the form.|
|`form()`|Returns the value markup used to render the value in the resource form.|(See "Value Markup" section below.)|
|`isValid()`|Returns whether the value object is valid.||
|`hydrate()`|Hydrates the value entity using the value object.||
|`render()`|Returns the markup used to render the value.|Typically used to transform the stored value into human-readable markup.|
|`toString()`|Returns the value as a simple string.||
|`getJsonLd()`|Returns an array representation of the value using JSON-LD notation.||
|`getFulltextText()`|Returns the the fulltext of the value.|Typically used to transform the stored value into a searchable string.|

You should look at the built-in data types for a better idea of how to use the passed
arguments. You should extend off one of the built-in data types or `AbstractDataType`
when possible. Your data type may share more functionality with them than you realize.

## Value Markup

The markup returned from `DataTypeInterface::form()` makes it possible for Omeka S
to dynamically build values for the resource form. You can format the markup any
way that makes sense for your data type and fits within the form.

While custom styles (CSS) and behavior (JS) are possible using `DataTypeInterface::prepareForm()`
or from within partials returned from `DataTypeInterface::form()`, Omeka S automates
much of the requisite behavior. It automatically populates the inputs' `name` attributes
when it loads the value, acting on special attributes that you define in the markup.

### Special Attributes

The inputs don't include the `name` or `value` attributes because they are populated
dynamically, during page load and resource template selection. Omeka S detects inputs
with special attributes and acts on them accordingly. The special attributes are:

|Attribute|Maps to value key|Description|
|---|---|---|
|`data-value-key="@value"`|`@value`|A string, structured or unstructured|
|`data-value-key="@language"`|`@language`|A language tag|
|`data-value-key="@id"`|`@id`|A URI|
|`data-value-key="o:label"`|`o:label`|A URI label|
|`data-value-key="value_resource_id"`|`value_resource_id`| A resource ID|

### Required Values

Since resource template properties can be marked as required, Omeka S automatically
performs client-side validation by detecting a `to-require`class on inputs. Any
input(s) that will be submitted with the form should include this class.

## Adding a Custom Data Type

If the built-in data types aren't adequate, Omeka S makes it relatively easy to
add custom ones. To demonstrate, we're going to build a module that adds a simple
"Date" data type.

First let's create the module's INI file at `/modules/MyModule/config/module.ini`:

```ini
[info]
version = "1.0.0-alpha"
omeka_version_constraint = "^1.0.0 || ^2.0.0"
name = "My Module"
description = "Adds a simple Date data type"
```

Then let's create the module file at `/moudules/MyModule/Module.php` and register
the data type in the config:

```php
<?php
namespace MyModule;

use Omeka\Module\AbstractModule;

class Module extends AbstractModule
{
    public function getConfig()
    {
        return [
            'data_types' => [
                'invokables' => [
                    'mymodule:date' => DataType\Date::class,
                ],
            ],
        ];
    }
}
```

Note that you must register a data type in the module config for Omeka S to detect it.

Next, let's create the "Date" data type class at `/modules/MyModule/src/DataType/Date.php`:

```php
<?php
namespace MyModule\DataType;

use Omeka\Api\Adapter\AbstractEntityAdapter;
use Omeka\Api\Representation\ValueRepresentation;
use Omeka\DataType\Literal;
use Laminas\View\Renderer\PhpRenderer;

class Date extends Literal
{
    public function getName()
    {
        return 'mymodule:date';
    }
    public function getLabel()
    {
        return 'Date'; // @translate
    }
    public function form(PhpRenderer $view)
    {
        return '<input type="date" class="to-require" data-value-key="@value">';
    }
    public function isValid(array $valueObject)
    {
        return (bool) preg_match('/^\d{4}-\d{2}-\d{2}$/', $valueObject['@value']);
    }
    public function render(PhpRenderer $view, ValueRepresentation $value)
    {
        return date('l, j F Y', strtotime($value->value()));
    }
    public function getFulltextText(PhpRenderer $view, ValueRepresentation $value)
    {
        return $this->render($view, $value);
    }
}
```

Note that we're extending off the built-in `Literal` data type because we're sharing
some functionality with it.

Let's take a closer look at the the markup returned from `Date::form()`:

```html
<input name="" type="date" class="to-require" data-value-key="@value">
```

Note the `data-value-key="@value"` attribute. Omeka S will detect this and automatically
populate the `name` attribute to include the `@value` key. If the template contained,
say, an additional language input with a `data-value-key="@language"` attribute,
then it would populate that `name` attribute to include the `@language` key.

After installing the module, set the new "Date" data type in a resource template
for a property. Next add an item and set it to this template. The property value
should change to a date input. Enter a date, save the form, and you should see a
formatted date describing your resource. Edit the item and the date input should
contain the date you entered.

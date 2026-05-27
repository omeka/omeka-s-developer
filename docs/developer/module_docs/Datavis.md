# Data Visualization

## REST API endpoints

- `/api/datavis_visualizations`
- `/api/datavis_visualizations/:id`

## Custom visualizations

Modules can extend this module to add their own custom visualizations. Note that a **visualization** consists of two parts:

1. A **dataset** that represents some portion of your data;
1. A **diagram** that visually represents the dataset.

To demonstrate this let's create a new dataset type named "My dataset" that adds a way to count items with certain classes. This dataset is already implemented in the "Count of items with classes" dataset type, but it serves as a good example of how to stitch the various parts together. First, register the type in your module's configuration.

```php-inline
'datavis_dataset_types' => [
    'invokables' => [
        'my_dataset_type' => \MyModule\Datavis\DatasetType\MyDatasetType::class,
    ],
],
```
Then, make the dataset type itself.

```php
<?php
namespace MyModule\Datavis\DatasetType;

use Datavis\Api\Representation\DatavisVisRepresentation;
use Datavis\DatasetType\AbstractDatasetType;
use Laminas\Form\Fieldset;
use Laminas\ServiceManager\ServiceManager;
use Omeka\Api\Representation\SiteRepresentation;
use Omeka\Form\Element\ResourceClassSelect;

class MyDataset extends AbstractDatasetType
{
    /**
     * Get the label of this dataset type.
     *
     * @return string
     */
    public function getLabel() : string
    {
        return 'My dataset'; // @translate
    }

    /**
     * Get the description of this dataset type.
     *
     * @return string
     */
    public function getDescription() : ?string
    {
        return 'Visualize the count of items that are instances of selected resource classes.'; // @translate
    }

    /**
     * Get the names of the diagram types that are compatible with this dataset.
     *
     * @return array
     */
    public function getDiagramTypeNames() : array
    {
        return ['my_diagram_type'];
    }

    /**
     * Add the form elements used for the dataset data.
     *
     * @param SiteRepresentation $site
     * @param Fieldset $fieldset
     */
    public function addElements(SiteRepresentation $site, Fieldset $fieldset) : void
    {
        $fieldset->add([
            'type' => ResourceClassSelect::class,
            'name' => 'class_ids',
            'options' => [
                'label' => 'Classes', // @translate
                'show_required' => true,
            ],
            'attributes' => [
                'multiple' => true,
                'required' => false,
                'class' => 'chosen-select',
                'data-placeholder' => 'Select classes…', // @translate
            ],
        ]);
    }

    /**
     * Generate and return the JSON dataset given a visualiation.
     *
     * @param ServiceManager $services
     * @param DatavisVisRepresentation $vis
     * @return array
     */
    public function getDataset(ServiceManager $services, DatavisVisRepresentation $vis) : array
    {
        $em = $services->get('Omeka\EntityManager');

        // Prepare the query to get the count of items with a class.
        $dql = '
        SELECT COUNT(i.id)
        FROM Omeka\Entity\Item AS i
        JOIN i.resourceClass AS rc
        WHERE i.id IN (:item_ids)
        AND rc.id = :class_id';
        $query = $em->createQuery($dql);
        $query->setParameter('item_ids', $this->getItemIds($services, $vis));

        $dataset = [];
        $classIds = $vis->datasetData()['class_ids'] ?? [];
        foreach ($classIds as $classId) {
            $class = $em->find('Omeka\Entity\ResourceClass', $classId);
            if (null === $class) {
                // This class does not exist.
                continue;
            }
            $vocab = $class->getVocabulary();
            $query->setParameter('class_id', $class->getId());
            $dataset[] = [
                'id' => $class->getId(),
                'label' => $class->getLabel(),
                'label_long' => sprintf('%s (%s)', $class->getLabel(), $vocab->getLabel()),
                'value' => (int) $query->getSingleScalarResult(),
            ];
        }
        return $dataset;
    }
}
```

For the second part of your visualization, let's create a new diagram type named "My diagram" that adds a way to render a dataset using a bar chart. This diagram is already implemented in the "Bar chart" disgram type, but it serves as a good example of how to stitch the various parts together. First, register the type in your module's configuration.

```php-inline
'datavis_diagram_types' => [
    'invokables' => [
        'my_diagram_type' => \MyModule\Datavis\DiagramType\MyDiagramType::class,
    ],
],
```

Then, make the diagram type itself.

```php
<?php
namespace MyModule\Datavis\DiagramType;

use Datavis\DiagramType\DiagramTypeInterface;
use Laminas\Form\Element;
use Laminas\Form\Fieldset;
use Laminas\View\Renderer\PhpRenderer;
use Omeka\Api\Representation\SiteRepresentation;

class MyDiagram implements DiagramTypeInterface
{
    /**
     * Get the label of this diagram type.
     *
     * @return string
     */
    public function getLabel() : string
    {
        return 'My diagram'; // @translate
    }

    /**
     * Add the form elements used for the diagram data.
     *
     * @param SiteRepresentation $site
     * @param Fieldset $fieldset
     */
    public function addElements(SiteRepresentation $site, Fieldset $fieldset) : void
    {
        $defaults = [
            'width' => 700,
            'height' => 700,
            'margin_top' => 30,
            'margin_right' => 30,
            'margin_bottom' => 60,
            'margin_left' => 100,
            'order' => 'value_asc',
        ];

        $fieldset->add([
            'type' => Element\Number::class,
            'name' => 'width',
            'options' => [
                'label' => 'Width', // @translate
            ],
            'attributes' => [
                'min' => 0,
                'value' => $defaults['width'],
                'placeholder' => $defaults['width'],
                'required' => true,
            ],
        ]);
        $fieldset->add([
            'type' => Element\Number::class,
            'name' => 'height',
            'options' => [
                'label' => 'Height', // @translate
            ],
            'attributes' => [
                'min' => 0,
                'value' => $defaults['height'],
                'placeholder' => $defaults['height'],
                'required' => true,
            ],
        ]);
        $fieldset->add([
            'type' => Element\Number::class,
            'name' => 'margin_top',
            'options' => [
                'label' => 'Margin top', // @translate
            ],
            'attributes' => [
                'min' => 0,
                'value' => $defaults['margin_top'],
                'placeholder' => $defaults['margin_top'],
                'required' => true,
            ],
        ]);
        $fieldset->add([
            'type' => Element\Number::class,
            'name' => 'margin_right',
            'options' => [
                'label' => 'Margin right', // @translate
            ],
            'attributes' => [
                'min' => 0,
                'value' => $defaults['margin_right'],
                'placeholder' => $defaults['margin_right'],
                'required' => true,
            ],
        ]);
        $fieldset->add([
            'type' => Element\Number::class,
            'name' => 'margin_bottom',
            'options' => [
                'label' => 'Margin bottom', // @translate
            ],
            'attributes' => [
                'min' => 0,
                'value' => $defaults['margin_bottom'],
                'placeholder' => $defaults['margin_bottom'],
                'required' => true,
            ],
        ]);
        $fieldset->add([
            'type' => Element\Number::class,
            'name' => 'margin_left',
            'options' => [
                'label' => 'Margin left', // @translate
            ],
            'attributes' => [
                'min' => 0,
                'value' => $defaults['margin_left'],
                'placeholder' => $defaults['margin_left'],
                'required' => true,
            ],
        ]);
        $fieldset->add([
            'type' => Element\Select::class,
            'name' => 'order',
            'options' => [
                'label' => 'Order', // @translate
                'value_options' => [
                    'value_asc' => 'By value (ascending)', // @translate
                    'value_desc' => 'By value (descending)', // @translate
                    'label_asc' => 'By label (ascending)', // @translate
                    'label_desc' => 'By label (descending)', // @translate
                ],
            ],
            'attributes' => [
                'value' => $defaults['order'],
                'required' => true,
            ],
        ]);
    }

    /**
     * Prepare the render of this diagram type.
     *
     * @param PhpRenderer $view
     */
    public function prepareRender(PhpRenderer $view) : void
    {
        $view->headScript()->appendFile('https://d3js.org/d3.v6.js');
        $view->headScript()->appendFile($view->assetUrl('js/diagram-render/bar_chart.js', 'Datavis'));
        $view->headLink()->appendStylesheet($view->assetUrl('css/diagram-render/bar_chart.css', 'Datavis'));
    }
}
```

Then, the last things to do is write the Javascript this is responsible for rendering the diagram. Do this by registering a callback to `Datavis.addDiagramType()` and using [D3](https://d3js.org/) (or your of choice of visualization library) to draw the diagram on the page. The passed arguments are:

- `div`: Draw the diagram within this DOM element;
- `dataset`: The dataset (in JSON format) to represent using the diagram;
- `datasetData`: The configured dataset-specific data;
- `diagramData`: The configured diagram-specific data;
- `blockData`: The configured block-specific data.

```js
/**
 * This diagram type will consume a dataset in the following format:
 * [{label: {string}, value: {int}}]
 */
Datavis.addDiagramType('my_diagram_type', (div, dataset, datasetData, diagramData, blockData) => {

    // Set the dimensions and margins of the graph.
    let width = diagramData.width ? parseInt(diagramData.width) : 700;
    let height = diagramData.height ? parseInt(diagramData.height) : 700;
    const margin = {
        top: diagramData.margin_top ? parseInt(diagramData.margin_top) : 30,
        right: diagramData.margin_right ? parseInt(diagramData.margin_right) : 30,
        bottom: diagramData.margin_bottom ? parseInt(diagramData.margin_bottom) : 100,
        left: diagramData.margin_left ? parseInt(diagramData.margin_left) : 60
    };
    width = width - margin.left - margin.right;
    height = height - margin.top - margin.bottom;

    // Add the svg.
    div.style.maxWidth = `${width + margin.left + margin.right}px`
    const svg = d3.select(div)
        .append('svg')
            .attr('viewBox', `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`)
            .append('g')
                .attr('transform', `translate(${margin.left},${margin.top})`);

    // Sort the dataset.
    dataset.sort((b, a) => {
        switch (diagramData.order) {
            case 'label_desc':
                return a.label.localeCompare(b.label);
                break;
            case 'label_asc':
                return b.label.localeCompare(a.label);
                break;
            case 'value_desc':
                return a.value - b.value;
                break;
            case 'value_asc':
            default:
                return b.value - a.value;
        }
    });

    // Set the x and y scales.
    const x = d3.scaleLinear()
        .range([0, width])
        .domain([0, Math.max(...dataset.map(d => d.value))]);
    const y = d3.scaleBand()
        .range([0, height]).padding(0.2)
        .domain(dataset.map(d => d.label));

    // Add the X axis.
    const xGroup = svg.append('g')
        .attr('transform', `translate(0, ${height})`)
        .style('font-size', '14px')
        .call(d3.axisBottom(x));
    xGroup.selectAll('text')
        .attr('transform', 'translate(-10,0)rotate(-45)')
        .style('text-anchor', 'end');

    // Add the Y axis.
    const yGroup = svg.append('g')
        .style('font-size', '14px')
        .call(d3.axisLeft(y));
    const labels = yGroup.selectAll('text').data(dataset);

    // Add the tooltip div.
    const tooltip = d3.select(div)
        .append('div')
        .attr('class', 'tooltip');

    // Add the bars.
    svg.selectAll('bar')
        .data(dataset)
        .enter()
        .append('rect')
            .attr('x', x(0))
            .attr('y', d => y(d.label))
            .attr('width', d => x(d.value))
            .attr('height', y.bandwidth())
            .attr('fill', '#69b3a2')
            .style('cursor', 'crosshair')
            .on('mousemove', (e, d) => {
                tooltip.style('display', 'inline-block')
                    .style('left', `${e.pageX + 2}px`)
                    .style('top', `${e.pageY + 2}px`)
                    .html(`${d.label_long ? d.label_long : d.label}: ${Number(d.value).toLocaleString()}`);
            })
            .on('mouseout', (e, d) => {
                tooltip.style('display', 'none');
            });

    // Enable label links. Note that the dataset must include a "url" key.
    labels.on('click', (e, d) => {
        if (d.url) {
            window.location.href = d.url;
        }
    });
});
```

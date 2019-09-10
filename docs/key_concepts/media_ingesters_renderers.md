# Media Ingesters and Renderers

Media ingesters and renderers are classes that expand the kinds of media users can
add, update, and display. Together, they define the entire round trip of a media:
how it's added to an item, updated, and rendered to the screen. It's important to
know that *every* media has a ingester and renderer and that they are able to work
both with uploaded files and with remote content.

## Built-in Ingesters and Renderers

Omeka S comes with several commonly-used media ingesters and renderers:

### Ingesters

- **html**: Add and edit HTML media
- **iiif**: Add a media by IIIF image URL
- **oembed**: Add a media by oEmbed URL
- **upload**: Add a media by file upload
- **url**: Add a media by URL
- **youtube**: Add a media by YouTube URL

### Renderers

- **file**: Display a file
- **html**: Display HTML
- **iiif**: Display an IIIF image
- **oembed**: Display an oEmbed image or HTML
- **youtube**: Display a YouTube iframe

Note that the "upload" and "url" ingesters don't have a correspondingly-named renderer.
This is because they both use the generic "file" renderer, which renders files according
to their media type or extension (configurable in the `file_renderers` config).
So, when writing a custom ingester, you may find that using an already-registered
renderer will suffice.

## Interfaces

### Ingester

Every ingester class must implement `IngesterInterface`. The required methods are:

|Method|Purpose|Comment|
|---|---|---|
|`getLabel()`|Returns a human-readable label for the ingester.|It should be unique across all media ingesters.|
|`getRenderer()`|Returns the name of the renderer for media ingested by the ingester.|It should be the name of a registered media renderer.|
|`form()`|Returns the markup used to render the controls needed to add the media.||
|`ingest()`|Processes an ingest (create) request.|Typically used to validate form data and fetch files or thumbnails|

If the media can be updated then it must implement `MutableIngesterInterface`. The
required methods are:

|Method|Purpose|Comment|
|---|---|---|
|`updateForm()`|Returns the markup used to render the controls needed to edit the media.||
|`update()`|Processes an update request.||

You should look at the built-in ingesters for a better idea of how to use the passed
arguments. 

### Renderer

Every renderer class must implement `RendererInterface`. The required method is:

|Method|Purpose|Comment|
|---|---|---|
|`render()`|Returns the markup used to render the media.||

You should look at the built-in renderers for a better idea of how to use the passed
arguments.

## File Downloaders and Uploaders

You'll find that, for some media, you'll need to upload or download files during
ingest, typically to fetch files and do something with them, like store them or
generate thumbnail images. For these purposes, Omeka S provides two services:

- `Omeka\File\Downloader`: Download a file from a remote URI
- `Omeka\File\Uploader`: Upload a file from a local client

To inject these services into your ingester you'd follow Zend's service factory
pattern.

For example, after injecting the `Omeka\File\Downloader` service into your ingester,
you could fetch a thumbnail image for your media in `IngesterInterface::ingest()`
by passing it a URL:

```php
$url = sprintf('http://example.com/%s.jpg', $imagePath);
$tempFile = $this->downloader->download($url);
if ($tempFile) {
    $tempFile->mediaIngestFile($media, $request, $errorStore, false);
}
```

Both the downloader and uploader will return a temporary file object that has a
handy `mediaIngestFile()` method designed specifically for ingesting a media file.
It will automatically validate the file and set the storage ID to the media, among
other things. The method signature, in order:

|Argument|Type|Default|Definition|
|---|---|---|---|
|`$media`|`Media`|n/a|The media object|
|`$request`|`Request`|n/a|The request object|
|`$errorStore`|`ErrorStore`|n/a|The error store object|
|`$storeOriginal`|`bool`|true|Do you want to store the original file?|
|`$storeThumbnails`|`bool`|true|Do you want to store thumbnail images?|
|`$deleteTempFile`|`bool`|true|Do you want to delete the temporary file after ingest?|
|`$hydrateFileMetadataOnStoreOriginalFalse`|`bool`|false|Do you want to hydrate the file metadata when `$storeOriginal = false`?|

Note that in the above example we're passing `false` to `$storeOriginal` and relying
on the default `$storeThumbnails = true` to only create thumbnail images, discarding
the downloaded temporary file.

## Adding a Custom Ingester and Renderer

If the built-in ingesters and renderers aren't adequate, Omeka S makes it relatively
easy to add custom ones. To demonstrate, we're going to build a module that adds
a simple "Tweet" media.

First let's create the module's INI file at `/modules/MyModule/config/module.ini`:

```ini
[info]
version = "1.0.0-alpha"
omeka_version_constraint = "^1.0.0 || ^2.0.0"
name = "My Module"
description = "Adds a simple Tweet media"
```

Then let's create the module file at `/moudules/MyModule/Module.php` and register
the media in the config:

```php
<?php
namespace MyModule;

use Omeka\Module\AbstractModule;

class Module extends AbstractModule
{
    public function getConfig()
    {
        return [
            'media_ingesters' => [
                'invokables' => [
                    'mymodule_tweet' => Media\Ingester\Tweet::class,
                ],
            ],
            'media_renderers' => [
                'invokables' => [
                    'mymodule_tweet' => Media\Renderer\Tweet::class,
                ],
            ],
        ];
    }
}
```

Note that you must register the ingester and renderer in the module config for Omeka
S to detect them.

Next, since our ingester needs the `Omeka\HttpClient` service, let's create the
"Tweet" ingester factory at `/modules/MyModule/src/Service/Ingester/TweetFactory.php`:

```php
<?php
namespace MyModule\Service\Media\Ingester;

use MyModule\Media\Ingester\Tweet;
use Zend\ServiceManager\Factory\FactoryInterface;
use Interop\Container\ContainerInterface;

class TweetFactory implements FactoryInterface
{
    public function __invoke(ContainerInterface $services, $requestedName, array $options = null)
    {
        return new Tweet($services->get('Omeka\HttpClient'));
    }
}
```

Then, let's create the "Tweet" ingester class at `/modules/MyModule/src/Media/Ingester/Tweet.php`:

```php
<?php
namespace MyModule\Media\Ingester;

use Omeka\Api\Request;
use Omeka\Entity\Media;
use Omeka\Media\Ingester\IngesterInterface;
use Omeka\Stdlib\ErrorStore;
use Zend\Form\Element\Text;
use Zend\Http\Client;
use Zend\View\Renderer\PhpRenderer;

class Tweet implements IngesterInterface
{
    protected $client;
    public function __construct(Client $client)
    {
        $this->client = $client;
    }
    public function getLabel()
    {
        return 'Tweet'; // @translate
    }
    public function getRenderer()
    {
        return 'mymodule_tweet';
    }
    public function form(PhpRenderer $view, array $options = [])
    {
        $input = new Text('o:media[__index__][o:source]');
        $input->setOptions([
            'label' => 'Tweet URL', // @translate
            'info' => 'URL for the tweet to embed.', // @translate
        ]);
        $input->setAttributes([
            'required' => true,
        ]);
        return $view->formRow($input);
    }
    public function ingest(Media $media, Request $request, ErrorStore $errorStore)
    {
        $data = $request->getContent();
        if (!isset($data['o:source'])) {
            $errorStore->addError('o:source', 'No tweet URL specified');
            return;
        }
        $isMatch = preg_match('/^https:\/\/twitter\.com\/[\w]+\/status\/[\d]+$/', $data['o:source']);
        if (!$isMatch) {
            $errorStore->addError('o:source', sprintf(
                'Invalid tweet URL: %s',
                $data['o:source']
            ));
            return;
        }
        $url = sprintf('https://publish.twitter.com/oembed?url=%s', urlencode($data['o:source']));
        $response = $this->client->setUri($url)->send();
        if (!$response->isOk()) {
            $errorStore->addError('o:source', sprintf(
                'Error reading tweet: %s (%s)',
                $response->getReasonPhrase(),
                $response->getStatusCode()
            ));
            return false;
        }
        $media->setSource($data['o:source']);
        $media->setData(json_decode($response->getBody(), true));
    }
}
```

Finally, let's create the "Tweet" renderer class at `/modules/MyModule/src/Media/Renderer/Tweet.php`:

```php
<?php
namespace MyModule\Media\Renderer;

use Omeka\Api\Representation\MediaRepresentation;
use Omeka\Media\Renderer\RendererInterface;
use Zend\View\Renderer\PhpRenderer;

class Tweet implements RendererInterface
{
    public function render(PhpRenderer $view, MediaRepresentation $media, array $options = [])
    {
        return $media->mediaData()['html'];
    }
}
```

After installing the module, you'll be able to add an embedded tweet to an item
by adding a "Tweet" media to an item and entering a URL to a single tweet. (Note
that this fuctionality is generally covered by the built-in "oembed" ingester, so
you don't actually need a new ingester for this. It's a helpful example nonetheless.)

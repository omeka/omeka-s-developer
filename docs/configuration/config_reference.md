# Configuration Reference

See [Configuring Omeka S](index.md) for a general
configuration overview and how to use `/config/local.config.php` and your
module's `config/module.config.php` to customize your installation's
configuration settings.

The following configuration settings are available (more may be added by modules):

## Api Adapters

- `api_adapters`: A list of API adapters for Omeka S resources (see [Omeka S API docs](../api/index.md))

## Assets

Configuration for asset files (JavaScript, CSS, fonts, etc.).

- `assets`
    - `use_externals`: Use external assets? (`true` (default) or `false`)
    - `externals`: Lists of external asset URLs, keyed by local asset path.

## Block Layouts

- `block_layouts`: a list of block layouts for Omeka S sites

## CLI

Configuration for executing PHP-CLI.

- `cli`
    - `execute_strategy`: The execution strategy allowed by your server ("exec" (default) or "proc_open");
    - `phpcli_path`: The server path of the PHP version used to run Omeka S (auto-detected by default; e.g. "/usr/bin/php55").

## Controllers

- `controllers`: A list of MVC controllers (see [laminas-mvc docs](https://docs.laminas.dev/laminas-mvc/quick-start/#create-a-controller))

## Controller Plugins

- `controller_plugins`: A list of MVC controller plugins (see [laminas-mvc docs](https://docs.laminas.dev/laminas-mvc/plugins/))

## Data Types

- `data_types`: A list of data types for Omeka S resource templates

## Entity Manager

Doctrine entity manager settings.

- `entity_manager`:
    - `is_dev_mode`: Whether to run the entity manager in development mode (see [Doctrine docs](http://doctrine-orm.readthedocs.io/projects/doctrine-orm/en/latest/reference/configuration.html#obtaining-an-entitymanager))
    - `mapping_classes_paths`: A list of paths containing Doctrine entity classes
    - `resource_discriminator_map`: A list of entities that are mapped to Omeka's `Resource` entity (for first-class resources that can be described using values)
    - `filters`: A list of SQL filters (see [Doctrine docs](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/filters.html))
    - `functions`: A list of user defined numeric, string, and datetime DQL functions (see [Doctrine docs](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/dql-user-defined-functions.html))
    - `proxy_paths`: A list of server paths to directories containing Doctrine proxies

## File Renderers

- `file_renderers`: A list of renderers for Omeka S file media

## File Store

Configuration for file storage.

- `file_store`:
    - `local`: Configuration for local store
        - `base_path`: Base path to local files directory
        - `base_uri`: Base URI to local files directory

Set the file store by setting `['service_manager']['aliases']['Omeka\File\Store']`
to one of the following:

- `Omeka\File\Store\Local` (default): Store files on the local server
- (Modules may include additional file store adapters and configuration)

## Form Elements

- `form_elements`: A list of form elements (see [laminas-form docs](https://docs.laminas.dev/laminas-form/v3/advanced/#creating-custom-elements))

## HTTP Client

Configuration for the HTTP client.

- `http_client`
    - `adapter`: the connection adapter
    - `sslcapath`: the server path to the SSL certificate directory
    - `sslcafile`: the server path to the SSL certificate file

In case your Omeka S instance requires a proxy server for outgoing HTTP traffic, you should set `'adapter' => \Laminas\Http\Client\Adapter\Proxy::class,` and use these additional configuration parameters:
- `http_client`:
    - `proxy_host`: the hostname or FQDN of the proxy server
    - `proxy_port`: the proxy server's TCP port
    - `proxy_user`: user name for proxy server, if required
    - `proxy_pass`: password for proxy server, if required


See [Laminas http docs](https://docs.laminas.dev/laminas-http/client/adapters/) for configuration options.

## Installer

Configuration for Omeka S installation.

- `installer`:
    - `pre_tasks`: A list of tasks to run before installation, usually to check environment
    - `tasks`: A list of tasks to run during installation

## JavaScript Translate Strings

- `js_translate_strings`: A list of messages rendered by JavaScript that need translation

## Listeners

- `listeners`: A list of MVC listeners to load during runtime (see [laminas-mvc docs](https://docs.laminas.dev/laminas-mvc/mvc-event/))

## Logger

Logging settings for application-level messages.

- `logger`
    - `log`: Log errors? (`false` (default) or `true`)
    - `priority`: The priority level at which to start logging (default `\Laminas\Log\Logger::NOTICE`; see [laminas-log docs](https://docs.laminas.dev/laminas-log/intro/#using-built-in-priorities))
    - `path`: The server path to the log file

## Mail

Configuration for mail transport and message options.

- `mail`
    - `transport`: mail transport configuration (see [laminas-mail docs](https://docs.laminas.dev/laminas-mail/transport/intro/))
    - `default_message_options`: message options (one common option is `from` to set the From address; see setters listed in [laminas-mail docs](https://docs.laminas.dev/laminas-mail/message/intro/#available-methods) for all possible options)

The default transport is Sendmail, which is set up in `application/config/module.config.php`).
If using SMTP, use this example configuration, added to the end of `local.config.php`
(see [laminas-mail docs](https://docs.laminas.dev/laminas-mail/transport/smtp-options/)):

```php-inline
'mail' => [
    'transport' => [
        'type' => 'smtp',
        'options' => [
            'name' => 'localhost',
            'host' => '127.0.0.1',
            'port' => 25, // 465 for 'ssl', and 587 for 'tls'
            'connection_class' => 'smtp', // 'plain', 'login', or 'crammd5'
            'connection_config' => [
                'username' => null,
                'password' => null,
                'ssl' => null, // 'ssl' or 'tls'
                'use_complete_quit' => true,
            ],
        ],
    ],
],
```

## Media Ingesters

- `media_ingesters`: A list of ingesters for Omeka S media

## Media Renderers

- `media_renderers`: A list of renderers for Omeka S media

## Navigation Links

- `navigation_links`: A list of navigation links for Omeka S site pages

## oEmbed

- `oembed`: A whitelist of allowable URL patterns for ingesting oEmbed media

## Password

- `password`: A list of configurable password restrictions
    - `min_length`: The minimum length
    - `min_lowercase`: The minimum number of lowercase letters
    - `min_uppercase`: The minimum number of uppercase letters
    - `min_number`: The minimum number of numbers
    - `min_symbol`: The minimum number of symbols
    - `symbol_list`: The list of symbols allowed by `min_symbol`

## Permissions

Configuration for the access control list (ACL).

- `permissions`:
    - `acl_resources`: A list of resources to load into the access control list. API adapters, Doctrine entities, and controllers that are registered in configuration are automatically loaded.

## Service Manager

- `service_manager`: A list of services (see [laminas-servicemanager docs](https://docs.laminas.dev/laminas-servicemanager/v4/configuring-the-service-manager/))

## Session

Configuration for saving state between requests.

- `session`:
    - `config`: Session configuration options (see [laminas-session docs](https://docs.laminas.dev/laminas-session/config/))
    - `save_handler`: Session save handler (leave `null` to use default database handler; see [laminas-session docs](https://docs.laminas.dev/laminas-session/save-handler/)))

## Temporary Directory

- `temp_dir`: The path to the temporary directory

## Thumbnails

Configuration for creating thumbnails (file derivatives).

- `thumbnails`
    - `thumbnailer_options`:
        - `imagemagick_dir`: For the `ImageMagick` thumbnailer, the directory to the ImageMagick command
        - `page`: For multi-page files, which page to create the thumbnails
    - `types` (large, medium, and square types):
        - `strategy`:  "default" or "square" thumbnails
        - `constraint`: Pixel constraint for thumbnail width
        - `options`: Options according to type (e.g. `'gravity' => 'center'`

Set the thumbnailer by setting `['service_manager']['aliases']['Omeka\File\Thumbnailer']`
to one of the following:

- `Omeka\File\Thumbnailer\ImageMagick` (default): Use ImageMagick directly
- `Omeka\File\Thumbnailer\Imagick`: Use PHP's ImageMagick extension
- `Omeka\File\Thumbnailer\Gd`: Use PHP's GD extension

## Translator

Translator settings.

- `translator`
    - `locale`: The language code for your locale
    - `translation_file_patterns`: Options for translation

## View Helpers

- `view_helpers`: A list of view helpers (see [laminas-view docs](https://docs.laminas.dev/laminas-view/v2/helpers/advanced-usage/))

## View Manager

- `view_manager`: View manager settings (see [laminas-view docs](https://docs.laminas.dev/laminas-view/v2/quick-start/#configuration))


# Configuration Reference

See [Configuring Omeka S](index.md) for a general
configuration overview and how to use `/config/local.config.php` and your
module's `config/module.config.php` to customize your installation's
configuration settings.

The following configuration settings are available (more may be added by modules):

## API Adapters

- `api_adapters`: A list of API adapters for Omeka S resources (see [Omeka S API docs](../api/index.md))

## API Assets

Configuration for Assets (user-added files like custom logos, thumbnails, etc.)

- `api_assets`
    - `allowed_media_types`: (Since 4.0.2) An array of media (MIME) types that are allowed to be uploaded as assets.
    - `allowed_extensions`: (Since 4.0.3) An array of file extensions that are allowed to be uploaded as assets.

## Assets

Configuration for asset files (JavaScript, CSS, fonts, etc.).

- `assets`
    - `use_externals`: Use external assets? (`true` (default) or `false`)
    - `externals`: Lists of external asset URLs, keyed by local asset path.

## Block Layouts

- `block_layouts`: a list of block layouts for Omeka S sites

## Block Templates

- `block_templates`: (Since 4.2) An array of [block templates](../themes/theme_templates.md), used to allow modules to add their own templates.

## Browse Defaults

- `browse_defaults`: (Since 4.0) Default sorting settings for controllers, keyed by interface (admin vs. public), then controller name. For each controller `sort_by` and `sort_order` can be set.

```php-inline
'browse_defaults' => [
    'admin' => [
        'items' => [
            'sort_by' => 'created',
            'sort_order' => 'desc',
        ],
    ],
],
```

## CLI

Configuration for executing PHP-CLI.

- `cli`
    - `execute_strategy`: The execution strategy allowed by your server ("exec" (default) or "proc_open");
    - `phpcli_path`: The server path of the PHP version used to run Omeka S (auto-detected by default; e.g. "/usr/bin/php55").

## Columns

(Since 4.0) Configuration for column settings in the admin interface.

- `column_types`: Available column types for use in the interface (this is a Laminas plugin manager config)
- `column_defaults`: Default column config for views, keyed by interface (admin vs. public), then view.

## Controllers

- `controllers`: A list of MVC controllers (see [laminas-mvc docs](https://docs.laminas.dev/laminas-mvc/quick-start/#create-a-controller))

## Controller Plugins

- `controller_plugins`: A list of MVC controller plugins (see [laminas-mvc docs](https://docs.laminas.dev/laminas-mvc/plugins/))

## Data Types

- `data_types`: A list of data types for Omeka S resource templates

(Since 3.1) In addition to the usual Laminas plugin manager keys, this additionally takes the key `value_annotating`, an array of data type names that are available for value annotations).

## Entity Manager

Doctrine entity manager settings.

- `entity_manager`:
    - `is_dev_mode`: Whether to run the entity manager in development mode (see [Doctrine docs](http://doctrine-orm.readthedocs.io/projects/doctrine-orm/en/latest/reference/configuration.html#obtaining-an-entitymanager))
    - `mapping_classes_paths`: A list of paths containing Doctrine entity classes
    - `resource_discriminator_map`: A list of entities that are mapped to Omeka's `Resource` entity (for first-class resources that can be described using values)
    - `filters`: A list of SQL filters (see [Doctrine docs](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/filters.html))
    - `functions`: A list of user defined numeric, string, and datetime DQL functions (see [Doctrine docs](http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/cookbook/dql-user-defined-functions.html))
    - `proxy_paths`: A list of server paths to directories containing Doctrine proxies
    - `data_types` (Since 4.1) An array of Doctrine DBAL custom data types to register

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

Logging settings for application-level messages. These settings are commonly overridden in the `local.config.php`, particularly the `log` setting to turn on logging.

- `logger`
    - `log`: Log errors? (boolean, default `false`)
    - `priority`: The priority level at which to start logging (default `\Laminas\Log\Logger::NOTICE`; see [laminas-log docs](https://docs.laminas.dev/laminas-log/intro/#using-built-in-priorities))
    - `path`: The server path to the log file (the preconfigured path is the default log path within the site, but this can be set to any server-writable path to change where the log is stored)

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

- `oembed`:
    - `whitelist`: An array of allowable URL patterns for ingesting oEmbed media
        - Before version 4.2, the values here are just string regexes for allowed URLs. In 4.2 each value can also be a two-element array where the first element is the regex as before, and the second is the URL to the oEmbed endpoint for that site.

## Page Templates

- `page_templates`: (Since 4.2) An array of [page templates](../themes/theme_templates.md), used to allow modules to add their own templates.

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
 
## Resource Page Block Layouts

(Since 4.0) Configuration for [Resource Page Block Layouts](../themes/theme_use_resource_page_blocks.md) for themes.

- `resource_page_block_layouts`: Available layouts. (this is a Laminas plugin manager config)
- `resource_page_blocks_default`: Default theme config for resource page blocks, per resource. This is blank in the default configuration, meaning that only the themes' default settings are used.

## Service Manager

- `service_manager`: A list of services (see [laminas-servicemanager docs](https://docs.laminas.dev/laminas-servicemanager/v4/configuring-the-service-manager/))

## Session

Configuration for saving state between requests.

- `session`:
    - `config`: Session configuration options (see [laminas-session docs](https://docs.laminas.dev/laminas-session/config/))
    - `save_handler`: Session save handler (leave `null` to use default database handler; see [laminas-session docs](https://docs.laminas.dev/laminas-session/save-handler/)))

## Sort Defaults

- `sort_defaults` (Since 4.0) Default available choices for the sort selector on browse pages. Keyed by interface (admin vs. public), then controller name.

```php-inline
'sort_defaults' => [
    'admin' => [
        'items' => [
            'title' => 'Title', // @translate
            'resource_class_label' => 'Resource class', // @translate
            'owner_name' => 'Owner', // @translate
            'created' => 'Created', // @translate
        ],
    ],
],
```

Note: since the values in the innermost array here are user-facing, they're typically marked with the `// @translate` comment to mark them as translatable strings.

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


# Core Documentation

Here you will find a guide to the generated core documentation for Omeka S's core classes. Each link refers to a representation of the docblock documentation for top namespaces within Omeka S, found in `application/src`.

## [Api](apigendocs/current/Api/index.html)

Essential classes for using and understanding the Omeka S API, including Adapters (for interacting with the database) and Representations (for reading data for display). Many of the classes here will match with classes under Entity and Controller.

## [Authentication](apigendocs/current/Authentication/index.html)

Classes for authenticating users and establishing permissions to the storage. See also [Zend\Authentication](https://docs.zendframework.com/zend-authentication/)

## [Controller](apigendocs/current/Controller/index.html)

Essential classes negotiating between requests, the database, and passing objects on to Views. Many of the classes here will match with classes under Api and Entity.

## [DataType](apigendocs/current/DataType/index.html)

Classes defining Omeka S datatypes (literals, URIs, and internal references), and handling their form elements and display. 

## [Db](apigendocs/current/Db/index.html)

Classes defining how Omeka S interacts with the database.

## [Entity](apigendocs/current/Entity/index.html)

Essential classes defining Omeka S objects (Items, Item Sets, etc) and how they interact via Doctrine.

## [File](apigendocs/current/File/index.html)

Classes defining File data, including derivatives and their storage data.

## [Form](apigendocs/current/Form/index.html)

Subclasses of [Zend\Form\Form](https://docs.zendframework.com/zend-form) used in Views for entering data.

## [I18n](apigendocs/current/I18n/index.html)

Class for creating and displaying translations of admin view content.

## [Installation](apigendocs/current/Installation/index.html)

Classes that handle the installation of Omeka S.

## [Job](apigendocs/current/Job/index.html)

Classes that handle dispatching background jobs, such as importers or other long-running processes.

## [Log](apigendocs/current/Log/index.html)

Class to handle logging for background jobs (Zend handles the logging to the usual location in `logs/application.log`).

## [Media](apigendocs/current/Media/index.html)

Classes defining how Media (images, YouTube videos, HTML, etc.) are created and displayed.

## [Module](apigendocs/current/Module/index.html)

Classes for the installation, status, reading .ini files, etc. for addon modules for Omeka S.

## [Mvc](apigendocs/current/Mvc/index.html)

Top level classes for routing and handling requests. See also [Zend\Mvc](https://docs.zendframework.com/zend-mvc/)

## [Permissions](apigendocs/current/Permissions/index.html)

Classes defining the Access Control List behavior for Omeka S. See also [Zend\Permissions\(https://docs.zendframework.com/zend-permissions-acl/).

## [Service](apigendocs/current/Service/index.html)

Classes for creating Omeka S objects and injecting the appropriate dependencies. See also [Zend\ServiceManager](https://docs.zendframework.com/zend-servicemanager/)

## [ServiceManager](apigendocs/current/ServiceManager/index.html)

Class to manage the creation of Services. See also [Zend\ServiceManager](https://docs.zendframework.com/zend-servicemanager/)

## [Session](apigendocs/current/Session/index.html)

Class to handle browser sessions.

## [Settings](apigendocs/current/Settings/index.html)

Classes to handle basic settings for Omeka S, such as site settings, installation settings, and user settings.

## [Site](apigendocs/current/Site/index.html)

Essential classes for defining things like Site structure and navigation.

## [Stdlib](apigendocs/current/Stdlib/index.html)

Utility classes used throughout Omeka S for basic tasks, like sending email, handling error messages, pagination, etc.

## [Test](apigendocs/current/Test/index.html)

Classes for the core test suite, found in `/test`.

## [View](apigendocs/current/View/index.html)

Mostly helper classes used to create content display, but also some classes for the JSON-LD output from the API. See also [Zend\View](https://docs.zendframework.com/zend-view/)



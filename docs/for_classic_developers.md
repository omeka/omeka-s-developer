# For Omeka Classic Developers

Omeka S follows the same spirit and principles of Omeka Classic of using open source software to help cultural heritage institutions make their holdings available on the web. The codebase, however, is completely different. The following is a quick guide to the differences in broad strokes. You'll want to read through the Key Concepts sections for details.

## Underlying Technologies

### Zend Framework

Omeka Classic was built on Zend Framework 1, which has now reach end of life. Omeka S is built on Zend Framework 3, which is a completely different architecture. Some essential points that Omeka S now takes advantage of are

* a heavy reliance on services
* extensive use of factories
* configuration via an array in `module.config.php` files
* use of true PHP namespaces

### Doctrine ORM

Omeka S has adopted the Doctrine Object Relation Mapper for database queries and to define records. Thus, records no longer extend from an `Omeka_Record_AbstractRecord` class. Instead, they will extend from `Omeka\Entity\Resource`.

This also lets Omeka S separate concerns between `Entities`, `Api Adapters`, and `Representations` This replaces the Omeka Classic system of defining a record and optional table class as the model.

## Terminology

Based on the above, some common terms from Omeka Classic have new analogs in Omeka S

| Omeka Classic | Omeka S |
|---------------|---------|
|Hooks and Filters | Events |
|Plugins           | Modules |
| 


Omeka S uses an access control list for privilege management.

## User Roles

There are six discrete user roles, each in a large part having greater access than the last:

* researcher: `researcher`
* author: `author`
* reviewer: `reviewer`
* editor: `editor`
* site administrator: `site_admin`
* global administrator: `global_admin`

## Checking for Permission

There are three ways to run a permission check. 

### ACL Service
Where the ACL service is available, there are two methods: `userIsAllowed()` and `isAllowed()`. See Services and Factories for more information.

```php
// Get the ACL service:
$acl = $this->getServiceLocator()->get('Omeka\Acl');
if ($acl->userIsAllowed($resource, $privilege)) {
    // current user is allowed
}
if ($acl->isAllowed($user, $resource, $privilege)) {
    // passed user is allowed
}
```

### From within a Resource Representation

When you have a resource representation, use `userIsAllowed()` to check for privileges on it. See Api for information on how and where to obtain a resource representation.

```php
// Get a resource representation via the API manager:
$api = $this->getServiceLocator()->get('Omeka\ApiManager');
$item = $api->read('items', 1)->getContent();
if ($item->userIsAllowed($privilege)) {
    // current user is allowed
}
```

### From within a View or Controller

From within a view or a controller, the `userIsAllowed()` helper is available:

```php
// In a view script:
if ($this->userIsAllowed($resource, $privilege)) {
    // current user is allowed
}
```
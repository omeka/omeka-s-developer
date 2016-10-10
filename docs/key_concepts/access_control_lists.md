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

### Resource Representation
```php
// Get a resource representation via the API manager:
$api = $this->getServiceLocator()->get('Omeka\ApiManager');
$item = $api->read('items', 1)->getContent();
if ($item->userIsAllowed($privilege)) {
    // current user is allowed
}
```

### View Helper
```php
// In a view script:
if ($this->userIsAllowed($resource, $privilege)) {
    // current user is allowed
}
```
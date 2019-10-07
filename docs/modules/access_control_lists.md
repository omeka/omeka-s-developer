---
title: Access Control Lists (ACL)
---

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
Where the ACL service is available, there are three methods: `userIsAllowed()`, `isAdmin()`, and `isAllowed()`. See [Services and Factories](services_and_factories.md) for more information.

`userIsAllowed($resource, $privilege)` checks whether the current user has access to a resource and privilege.

`isAllowed($user, $resource, $privilege)` can be used to check the same access for any user.

`isAdminRole($role)` checks whether a user role is among the ones with admin privileges (i.e., `site_admin` or `global_admin`).

```php
// Get the ACL service:
$acl = $this->getServiceLocator()->get('Omeka\Acl');
if ($acl->userIsAllowed($resource, $privilege)) {
    // current user is allowed
}

if ($acl->isAllowed($user, $resource, $privilege)) {
    // passed user is allowed
}


$role = $user->getRole();
if ($acl->isAdminRole($role) {
    // allow admin access
}

```

### From within a Resource Representation

When you have a resource representation, use `userIsAllowed()` to check for privileges on it.

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

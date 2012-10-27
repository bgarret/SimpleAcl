Simple Access Control List (ACL) for PHP.

[![Build Status](https://secure.travis-ci.org/alexshelkov/SimpleAcl.png)](http://travis-ci.org/alexshelkov/SimpleAcl)
_____________________________________________________________________________________________________________________
#### Install
##### Using composer
Add following in your composer.json:
```json
{
    "require": {
        "alexshelkov/simpleacl": "1.*"
    }
}
```
##### Manual
Download library and register PSR-0 compatible autoloader.
________________________________________________________
#### Usage
##### Basic usage
###### Theory
There is 4 kind of objects: *Rules*, *Roles*, *Resources* and *Acl* which holds list of *Rules*. Some *Rule* can grant access for some *Role* to some *Resource*.

###### Create rules
Lets create "View" Rule, and with with it grant access for "User" to "Page" (note: all names are case sensitive):
```php
$view = new Rule('View');
$view->setRole(new Role('User'));
$view->setResource(new Resource('Page'));
$view->setAction(true); // true means that we allow access

var_dump($view->isAllowed('User', 'Page')); // true
```

###### Add rules
There is not much sense in rules without Acl. So we need to add rules in it. In next example we add few rules in Acl and see whats happens.
```php
$acl = new Acl();

$user = new Role('User');
$admin = new Role('Admin');

$siteFrontend = new Resource('SiteFrontend');
$siteBackend = new Resource('SiteBackend');

$acl->addRule($user, $siteFrontend, new Rule('View'), true);
$acl->addRule($admin, $siteFrontend, 'View', true); // you can use string as rule
$acl->addRule($admin, $siteBackend, 'View', true);

var_dump($acl->isAllowed('User', 'SiteFrontend', 'View')); // true
var_dump($acl->isAllowed('User', 'SiteBackend', 'View')); // false
var_dump($acl->isAllowed('Admin', 'SiteFrontend', 'View')); // true
var_dump($acl->isAllowed('Admin', 'SiteBackend', 'View')); // true
```
###### Roles inheritance
As you maybe notice in previous example we have some duplication of code, because both "User" and "Admin" was allowed to "View" "SiteFrontend" we added 2 rules. But it is possible to avoid this using roles inheritance.
```php
$acl = new Acl();

$user = new Role('User');
$admin = new Role('Admin');
$user->setParent($admin); // add user's parent

$siteFrontend = new Resource('SiteFrontend');
$siteBackend = new Resource('SiteBackend');

$acl->addRule($user, $siteFrontend, 'View', true);
$acl->addRule($admin, $siteBackend, 'View', true);

var_dump($acl->isAllowed('User', 'SiteFrontend', 'View')); // true
var_dump($acl->isAllowed('User', 'SiteBackend', 'View')); // false
var_dump($acl->isAllowed('Admin', 'SiteFrontend', 'View')); // true
var_dump($acl->isAllowed('Admin', 'SiteBackend', 'View')); // true
```
Inheritance works for resources too.

##### Advanced usage
It is possible to check access not for particular Role or Resource, but for objects which aggregate them. These kind of objects must implement, respectively, SimpleAcl\Role\RoleAggregateInterface and SimpleAcl\Role\ResourceAggregateInterface.

You can use SimpleAcl\Role\RoleAggregate and SimpleAcl\Role\ResourceAggregate as object which allow aggregation.
```php
$acl = new Acl();

$user = new Role('User');
$admin = new Role('Admin');

$all = new RoleAggregate;
$all->addRole($user);
$all->addRole($admin);

$siteFrontend = new Resource('SiteFrontend');
$siteBackend = new Resource('SiteBackend');

$acl->addRule($user, $siteFrontend, 'View', true);
$acl->addRule($admin, $siteBackend, 'View', true);

var_dump($acl->isAllowed($all, 'SiteFrontend', 'View')); // true
var_dump($acl->isAllowed($all, 'SiteBackend', 'View')); // true
```
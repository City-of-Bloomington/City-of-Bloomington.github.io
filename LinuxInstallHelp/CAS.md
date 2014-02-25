---
layout: default
title: CAS Install
---
[[CAS|http://www.jasig.org/cas]] (Single Sign On) is supported,
but is commented out in the `configuration.inc` by default. If you have a CAS
server set up, uncomment the CAS settings and fill in the values for your CAS
server. You will need to download the [[phpCAS|https://wiki.jasig.org/display/CASC/phpCAS]]
library and put it somewhere accessible by PHP.

```php
define('CAS',APPLICATION_HOME.'/libraries/phpCAS');
define('CAS_SERVER','cas.somewhere.org');
define('CAS_URI','cas');
```

CAS authentication takes place on a seperate website and the user never types
in a username and password in CRM. Currently, when CAS authentication is enabled,
any "local" or "Employee" authentication is bypassed, by default.

However, If you have configured an Employee LDAP directory, the LDAP directory
will still be used for importing user information during account editing.

## Implementation Notes:
CAS authentication is attempted in `/login`.
Local and Employee authentication is attemped in `/login/login`.
If CAS is unavailable, `/login` just redirects to `/login/login`.

The only link to the login is in `templates/html/partials/banner.inc`.
Depending on your situation, you can edit the links in the banner to point to
whatever authentication systems you want.

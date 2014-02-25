
LDAP (and ADS) can be used for authentication and for importing user details
during account creation. To enable LDAP support, you'll need to edit
`configuration.inc`

Uncomment the Employee values in the `$DIRECTORY_CONFIG` array and provide
values for your LDAP or ADS directory. The Employee class is written to expect
the inetOrgPerson schema. If you are using a different schema in your LDAP
directory, you may need to customize the code in
`libraries/framework/class/Employee.php`.

## Employee LDAP configuration

`DIRECTORY_USER_BINDING` will substitute {username} with the username the user
provides. For authentication, ADS servers typically need to bind as
username@domain.name, whereas LDAP servers typically only need to bind using the
raw username.

LDAP will bind as `DIRECTORY_ADMIN_BINDING` when doing lookups of user
information during user account creation and editing.

### LDAP-style configuration example
```php
$DIRECTORY_CONFIG = array(
	'Employee'=>array(
		'DIRECTORY_SERVER'=>'ldaps://example.org:636',
		'DIRECTORY_BASE_DN'=>'ou=people,o=some.ldap.domain',
		'DIRECTORY_USERNAME_ATTRIBUTE'=>'uid',
		'DIRECTORY_USER_BINDING'=>'uid={username},'.DIRECTORY_BASE_DN,

'DIRECTORY_ADMIN_BINDING'=>'uid=admin@bloomington.in.gov,'.DIRECTORY_BASE_DN,
		'DIRECTORY_ADMIN_PASS'=>'password'
	)
);
```

### ADS-style configuration example
```php
$DIRECTORY_CONFIG = array(
	'Employee'=>array(
		'DIRECTORY_SERVER'=>'ldaps://example.org:636',
		'DIRECTORY_BASE_DN'=>'OU=Department,DC=example,DC=org',
		'DIRECTORY_USERNAME_ATTRIBUTE'=>'CN',
		'DIRECTORY_USER_BINDING'=>'{username}@bloomington.in.gov',
		'DIRECTORY_ADMIN_BINDING'=>'admin@bloomington.in.gov',
		'DIRECTORY_ADMIN_PASS'=>'password'
	)
);
```

## Public LDAP configuration

To Be Implemented...

## Authentication

User accounts have a property called "authenticationMethod". The only method
enabled by default is "local". Local users are authenticated by comparing the
password they give with the SHA encrypted password in the database. When a user
attempts to login with an LDAP authentication method, such as Employee, CRM will
attempt to bind to the directory as the user to verify their password.

To add custom LDAP authentication schemes, you add entries to the
`$DIRECTORY_CONFIG` variable in `configuration.inc`. Employee authentication is
included, by default, but is commented out in configuration.inc. The idea is to
support separate LDAP directories for city employees and members of the public.
However; at the time of this writing (2012-01-23), public user accounts are not
yet fully implemented.

## Account Creation

Only Administrators are allowed to create or edit user accounts for people. When
creating or editing a user account, you can choose from the available
authentication methods. Local as well as any custom methods will be listed in
the drown.

When a user is saved with a custom `authenticationMethod`, CRM will import any
missing personal information from the LDAP directory. This means, once you have
an Employee LDAP set up, you can add users to the system by just providing their
username and any role you want to assign them.

**Employee user accounts are not created automatically on login**.
Employees must be added manually by an Administrator.

In future development, Public user accounts will probably be created
automatically on first login.

---
autotoc: true
---

## External user authentication

By default, Galaxy manages its own users.  However, it may be more useful at your site to tie into a local authentication system.

Galaxy does now do this by itself (Before we needed to use [Nginx](/Admin/Config/NginxExternalUserAuth) or [Apache](/Admin/Config/ApacheExternalUserAuth) to handle this).

### Activate authentication through LDAP

To be able to authenticate your users through the LDAP, we are going to use a configuration file to enter all the required informations.

#### Tell Galaxy to use auth_conf file

In `config/galaxy.ini`, uncomment the line `auth_config_file = config/auth_conf.xml`:

```
# XML config file that allows the use of different authentication providers
# (e.g. LDAP) instead or in addition to local authentication (.sample is used
# if default does not exist).
auth_config_file = config/auth_conf.xml
```


#### Configure the auth_conf file

Copy the `config/auth_conf.xml.sample` and name it `config/auth_conf.xml`:

```
cp config/auth_conf.xml.sample config/auth_conf.xml
```


Then configure it appropriately to your LDAP (the documentation in the sample file should be enough).

#### Special Case: AD in CRUK

In CRUK, the Active Directory does not allow to get `sAMAccountName`.

We had to find another solution to get the Authentication working, register properly and get the username.

##### Modifications in auth_conf file

Here are the modifications we had to do in the `config/auth_conf.xml`:

```
<bind-user>{email}</bind-user>
<bind-password>{password}</bind-password>
<auto-register-username>{usernameFromWhoami}</auto-register-username>
<auto-register-email>{email}</auto-register-email>
```


We can notice a new variable: `usernameFromWhoami`

Then, we had to modify the `lib/galaxy/auth/providers/ldap_ad_py` file to add this variable:

After the line: `import logging`, we imported the regexp python library:

```
import re
```


Then, we fetched the username through the `whoami_s` ldap-python library:

After the line `whoami = l.whoami_s()`:

```
p = re.compile(ur'[^\\]*$')

username_from_whoami = re.search(p, whoami).group()
params['usernameFromWhoami'] = username_from_whoami
```


Launch Galaxy, and try to login :).
---
title: usergroups
ordersectionsby: title
---

Manage users, groups, userpasswords and groups membership.

Available clients are:

- [usersgroups_adpypsrp](./usersgroups_adpypsrp/): store data into an Active Directory through Powershell commands across `pypsrp`.
- [usersgroups_adwinrm](./usersgroups_adwinrm/): stores data into an Active Directory through Powershell commands across `pywinrm`.
- [usersgroups_flatfiles_emails_of_groups](./usersgroups_flatfiles_emails_of_groups/): generates a flat txt file by `Groups`, containing the e-mail addresses of its members (one by line).
- [usersgroups_kadmin_heimdal](./usersgroups_kadmin_heimdal/): stores data in an Heimdal Kerberos server.
- [usersgroups_ldap](./usersgroups_ldap/): stores data in an LDAP directory.
- [usersgroups_null](./usersgroups_null/): does nothing but logging.

---
title: usergroups
ordersectionsby: title
---

Manage users, groups, userpasswords and groups membership.

Available clients are

- [usersgroups_adpypsrp](usersgroups_adpypsrp) : store data into an Active Directory through Powershell commands across `pypsrp`.
- [usersgroups_adwinrm](usersgroups_adwinrm) : stores data into an Active Directory through Powershell commands across `pywinrm`.
- [usersgroups_flatfiles_emails_of_groups](usersgroups_flatfiles_emails_of_groups) : generates a flat txt file by `Groups`, containing the e-mail adresses of its members (one by line).
- [usersgroups_ldap](usersgroups_ldap) : stores data in a LDAP directory.
- [usersgroups_null](usersgroups_null) : does nothing but logging.

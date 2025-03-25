---
title: usergroups
ordersectionsby: title
---

Manage users, groups, userpasswords and groups membership.

Available clients are:

- [usersgroups_adpypsrp](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_adpypsrp/): store data into an Active Directory through Powershell commands across `pypsrp`.
- [usersgroups_bsspartage](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_bsspartage/): store data into the [PARTAGE](https://www.renater.fr/services/collaborer-simplement/partage/) dashboard through its API, handled by [libPythonBssApi](https://github.com/dsi-univ-rennes1/libPythonBssApi).
- [usersgroups_flatfiles_emails_of_groups](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_flatfiles_emails_of_groups/): generates a flat txt file by `Groups`, containing the e-mail addresses of its members (one by line).
- [usersgroups_kadmin_heimdal](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_kadmin_heimdal/): stores data in an Heimdal Kerberos server.
- [usersgroups_ldap](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_ldap/): stores data in an LDAP directory.
- [usersgroups_null](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_null/): does nothing but logging.

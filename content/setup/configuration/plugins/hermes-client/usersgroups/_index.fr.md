---
title: usergroups
ordersectionsby: title
---

Gère les utilisateurs, les groupes, les mots de passe des utilisateurs et l'appartenance des utilisateurs aux groupes.

Les clients disponibles sont :

- [usersgroups_adpypsrp](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_adpypsrp/) : stocke les données dans un Active Directory via des commandes Powershell via `pypsrp`.
- [usersgroups_bsspartage](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_bsspartage/): stocke les données dans le tableau de bord de [PARTAGE](https://www.renater.fr/services/collaborer-simplement/partage/) via son API, gérée par [libPythonBssApi](https://github.com/dsi-univ-rennes1/libPythonBssApi).
- [usersgroups_flatfiles_emails_of_groups](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_flatfiles_emails_of_groups/) : génére un fichier txt plat par `Groups`, contenant les adresses e-mail de ses membres (une par ligne).
- [usersgroups_kadmin_heimdal](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_kadmin_heimdal/) : stocke les données sur un serveur Kerberos Heimdal.
- [usersgroups_ldap](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_ldap/) : stocke les données dans un annuaire LDAP.
- [usersgroups_null](/setup/configuration/plugins/hermes-client/usergroups/usersgroups_null/) : ne fait rien d'autre que de générer des logs.

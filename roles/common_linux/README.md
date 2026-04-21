# Rôle Ansible : common_linux

## Description

Ce rôle configure les éléments de base communs à tous les serveurs Linux de l'entreprise ATLOS. Il est conçu pour être appliqué sur tous les serveurs Linux pour établir une configuration de base cohérente.

## Fonctionnalités

- Installation des paquets de base (curl, vim, git, htop, etc.)
- Configuration du MOTD (Message Of The Day)
- Configuration de la timezone système
- Configuration de la locale système
- Configuration des limites système (ulimit)
- Création du répertoire de logs applicatifs
- Configuration des alias bash globaux

## Variables

### Variables principales

| Variable | Description | Défaut |
|----------|-------------|--------|
| `common_linux_install_packages` | Activer l'installation des paquets | `true` |
| `common_linux_packages` | Liste des paquets à installer | curl, wget, vim, git, etc. |
| `common_linux_motd_enabled` | Activer la configuration du MOTD | `false` (défini dans group_vars) |
| `common_linux_motd_message` | Message MOTD personnalisé | (défini dans group_vars) |
| `common_linux_timezone_enabled` | Activer la configuration de la timezone | `true` |
| `common_linux_timezone` | Timezone à configurer | `Europe/Paris` |
| `common_linux_locale_enabled` | Activer la configuration de la locale | `true` |
| `common_linux_locale` | Locale à configurer | `fr_FR.UTF-8` |
| `common_linux_ulimit_enabled` | Activer la configuration des ulimits | `false` |
| `common_linux_ulimit_nofile` | Limite de fichiers ouverts | `65536` |
| `common_linux_ulimit_nproc` | Limite de processus | `4096` |
| `common_linux_log_directory_enabled` | Activer la création du répertoire de logs | `true` |
| `common_linux_log_directory` | Répertoire de logs | `/var/log/apps` |
| `common_linux_bash_aliases_enabled` | Activer les alias bash | `true` |
| `common_linux_bash_aliases` | Dictionnaire des alias | ll, la, l, grep, etc. |

## Dépendances

Aucune dépendance externe.

## Exemple d'utilisation

### Dans un playbook

```yaml
- name: Appliquer la configuration de base Linux
  hosts: linux_servers
  become: yes
  roles:
    - common_linux
```

### Surcharge des variables

Vous pouvez surcharger les variables dans `group_vars` ou `host_vars` :

```yaml
# inventories/dev/group_vars/linux_servers.yml
common_linux_packages:
  - curl
  - wget
  - vim
  - htop
  - docker.io  # Ajout de Docker en dev
```

## Compatibilité

Ce rôle est conçu pour les distributions Linux basées sur Debian/Ubuntu. Pour une compatibilité avec RHEL/CentOS, il faudrait adapter les tâches pour utiliser `yum` ou `dnf` au lieu de `apt`.

## Notes de sécurité

- Ce rôle ne gère pas de secrets
- Les ulimits sont configurés pour root uniquement (à adapter pour d'autres utilisateurs en production)
- Le MOTD peut contenir des informations sensibles selon votre environnement

## Maintenance

Ce rôle est maintenu par l'équipe DevOps d'ATLOS. Pour toute question ou demande de modification, contactez : devops@atlos.local

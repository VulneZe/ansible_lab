# Rôle Ansible : nginx_reverse_proxy

## Description

Ce rôle installe et configure nginx comme serveur web simple. Il déploie une page d'accueil personnalisée via un template Jinja2, adaptée selon l'environnement (dev, staging, prod).

## Fonctionnalités

- Installation du paquet nginx
- Configuration du répertoire racine
- Déploiement d'une page d'accueil HTML personnalisée
- Gestion du service nginx (démarrage, activation)
- Vérification de la configuration nginx
- Redémarrage automatique uniquement si nécessaire (handler)

## Variables

### Variables principales

| Variable | Description | Défaut |
|----------|-------------|--------|
| `nginx_package_name` | Nom du paquet nginx | `nginx` |
| `nginx_service_name` | Nom du service nginx | `nginx` |
| `nginx_listen_port` | Port d'écoute HTTP | `80` |
| `nginx_https_port` | Port d'écoute HTTPS | `443` |
| `nginx_ssl_enabled` | Activer SSL/TLS | `false` |
| `nginx_root` | Répertoire racine nginx | `/var/www/html` |
| `nginx_user` | Utilisateur nginx | `www-data` |
| `nginx_group` | Groupe nginx | `www-data` |
| `nginx_index_title` | Titre de la page d'accueil | (défini dans group_vars) |
| `nginx_index_message` | Message de la page d'accueil | (défini dans group_vars) |
| `nginx_company_name` | Nom de l'entreprise | (défini dans group_vars) |
| `nginx_environment_name` | Nom de l'environnement | (défini dans group_vars) |
| `nginx_server_fqdn` | FQDN du serveur | `{{ inventory_hostname }}` |
| `nginx_index_dest` | Chemin de la page index.html | `/var/www/html/index.html` |
| `nginx_service_state` | État du service | `started` |
| `nginx_service_enabled` | Activer au démarrage | `true` |

## Dépendances

Aucune dépendance externe.

## Exemple d'utilisation

### Dans un playbook

```yaml
- name: Déployer nginx sur les serveurs Wazuh
  hosts: wazuh_managers
  become: yes
  roles:
    - nginx_reverse_proxy
```

### Surcharge des variables

Vous pouvez surcharger les variables dans `group_vars` ou `host_vars` :

```yaml
# inventories/dev/group_vars/wazuh_managers.yml
nginx_index_title: "ATLOS - Serveur de Développement Wazuh"
nginx_index_message: "Ce serveur héberge Wazuh Manager dans l'environnement de développement ATLOS."
nginx_ssl_enabled: false
```

## Template

Le template `index.html.j2` utilise les variables suivantes pour personnaliser la page :
- `nginx_index_title` : Titre principal
- `nginx_index_message` : Message d'accueil
- `nginx_company_name` : Nom de l'entreprise
- `nginx_environment_name` : Nom de l'environnement
- `nginx_server_fqdn` : FQDN du serveur
- `ansible_default_ipv4.address` : Adresse IP du serveur
- `ansible_date_time.iso8601` : Date et heure de déploiement

La page inclut un badge de couleur selon l'environnement :
- Vert pour dev
- Jaune pour staging
- Rouge pour prod
- Gris pour unknown

## Compatibilité

Ce rôle est conçu pour les distributions Linux basées sur Debian/Ubuntu. Pour une compatibilité avec RHEL/CentOS, il faudrait adapter les tâches pour utiliser `yum` ou `dnf` au lieu de `apt`.

## Handlers

### Redémarrer nginx

Ce handler est notifié par la tâche de déploiement de la page d'accueil. Le service nginx n'est redémarré que si le fichier `index.html` a été modifié, ce qui évite des redémarrages inutiles.

## Notes de sécurité

- SSL/TLS n'est pas activé par défaut (à configurer en production)
- Le rôle ne gère pas la configuration SSL (certificats, clés)
- En production, il est recommandé d'activer SSL/TLS avec des certificats valides

## Maintenance

Ce rôle est maintenu par l'équipe DevOps d'ATLOS. Pour toute question ou demande de modification, contactez : devops@atlos.local

# Rôle Ansible : wazuh_agent

## Description

**IMPORTANT : Ce rôle est une SIMULATION PÉDAGOGIQUE.**

Ce rôle simule l'installation et la configuration d'un agent Wazuh sur des serveurs Linux. Il est conçu pour l'apprentissage et montre la structure d'un rôle Ansible pour un déploiement Wazuh, mais n'installe PAS l'agent Wazuh réel.

Pour une installation réelle de Wazuh Agent en production, consultez la documentation officielle : https://documentation.wazuh.com/current/deployment-options/deploy-wazuh-agent/index.html

## Fonctionnalités (simulation)

- Installation des paquets de base nécessaires pour Wazuh
- Création du répertoire d'installation Wazuh
- Création du répertoire de logs
- Création d'un fichier de configuration exemple (ossec.conf)
- Structure prête pour être adaptée à une installation réelle

## Variables

### Variables principales

| Variable | Description | Défaut |
|----------|-------------|--------|
| `wazuh_agent_enabled` | Activer le rôle wazuh_agent | `true` |
| `wazuh_agent_mode` | Mode d'exécution (simulation/real) | `simulation` |
| `wazuh_agent_version` | Version de Wazuh Agent (simulation) | `4.7.0` |
| `wazuh_manager_address` | Adresse IP/FQDN du Wazuh Manager | `wazuh-manager.atlos.local` |
| `wazuh_manager_port` | Port de communication | `1514` |
| `wazuh_registration_mode` | Mode d'enregistrement | `auto` |
| `wazuh_registration_password` | Mot de passe d'enregistrement | (défini dans vault) |
| `wazuh_agent_install_dir` | Répertoire d'installation | `/var/ossec` |
| `wazuh_agent_config_file` | Fichier de configuration | `/var/ossec/etc/ossec.conf` |
| `wazuh_agent_service_name` | Nom du service | `wazuh-agent` |
| `wazuh_agent_service_state` | État du service | `started` |
| `wazuh_agent_service_enabled` | Activer au démarrage | `true` |

### Variables de surveillance (simulation)

| Variable | Description | Défaut |
|----------|-------------|--------|
| `wazuh_agent_file_monitoring_enabled` | Surveillance des fichiers | `true` |
| `wazuh_agent_process_monitoring_enabled` | Surveillance des processus | `true` |
| `wazuh_agent_network_monitoring_enabled` | Surveillance réseau | `true` |

## Dépendances

Aucune dépendance externe.

## Exemple d'utilisation

### Dans un playbook (mode simulation)

```yaml
- name: Déployer l'agent Wazuh (simulation)
  hosts: linux_servers
  become: yes
  roles:
    - wazuh_agent
```

### Surcharge des variables

Vous pouvez surcharger les variables dans `group_vars` ou `host_vars` :

```yaml
# inventories/dev/group_vars/linux_servers.yml
wazuh_manager_address: "172.20.10.4"
wazuh_manager_port: 1514
wazuh_registration_mode: "auto"
```

## Installation réelle (à adapter)

Pour passer en mode réel et installer Wazuh Agent effectivement :

1. Changez le mode dans les variables :
   ```yaml
   wazuh_agent_mode: real
   ```

2. Adaptez les tâches dans `tasks/main.yml` selon la documentation Wazuh :
   - Ajout du dépôt Wazuh
   - Import de la clé GPG
   - Installation du paquet wazuh-agent
   - Configuration de ossec.conf
   - Enregistrement auprès du manager
   - Gestion du service

3. Consultez la documentation officielle pour les détails :
   https://documentation.wazuh.com/current/deployment-options/deploy-wazuh-agent/index.html

## Compatibilité

Ce rôle est conçu pour les distributions Linux basées sur Debian/Ubuntu. Pour une compatibilité avec RHEL/CentOS, il faudrait adapter les tâches pour utiliser `yum` ou `dnf` au lieu de `apt`.

## Notes importantes

- **Ce rôle est une simulation pédagogique** et n'installe PAS l'agent Wazuh réel
- Le fichier de configuration créé est un exemple et ne doit PAS être utilisé en production
- Pour une installation réelle, vous devez adapter le rôle selon la documentation Wazuh
- Les secrets (mot de passe d'enregistrement) doivent être gérés avec Ansible Vault

## Sécurité

- En production, utilisez toujours Ansible Vault pour les secrets
- Configurez correctement le pare-feu pour la communication avec le Wazuh Manager
- Validez la configuration de l'agent avant déploiement en production
- Surveillez les logs de l'agent pour détecter les anomalies

## Maintenance

Ce rôle est maintenu par l'équipe DevOps d'ATLOS. Pour toute question ou demande de modification, contactez : devops@atlos.local

## Ressources

- Documentation Wazuh : https://documentation.wazuh.com/
- Référentiel Wazuh : https://github.com/wazuh/wazuh
- Communauté Wazuh : https://wazuh.com/community/

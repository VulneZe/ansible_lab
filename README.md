# ATLOS Ansible Lab

Projet Ansible complet, propre et pédagogique pour l'entreprise fictive **ATLOS**. Ce projet est conçu pour l'apprentissage d'Ansible tout en respectant les bonnes pratiques d'entreprise.

## 📋 Table des matières

- [Présentation du projet](#présentation-du-projet)
- [Architecture du lab](#architecture-du-lab)
- [Rôles des machines](#rôles-des-machines)
- [Structure des dossiers](#structure-des-dossiers)
- [Notion de multi-environnement](#notion-de-multi-environnement)
- [Comment lancer les playbooks](#comment-lancer-les-playbooks)
- [Bonnes pratiques SSH / user ansible](#bonnes-pratiques-ssh--user-ansible)
- [Notion de Vault](#notion-de-vault)
- [Notion de rôles et handlers](#notion-de-rôles-et-handlers)
- [Exemples de commandes utiles](#exemples-de-commandes-utiles)
- [Logique de nommage des machines](#logique-de-nommage-des-machines)

## Présentation du projet

Ce projet est un laboratoire Ansible orienté apprentissage mais basé sur des bonnes pratiques d'entreprise. Il permet de :

- Apprendre Ansible proprement avec une structure réaliste
- Comprendre l'architecture multi-environnement (dev, staging, prod)
- Maîtriser les rôles, handlers, templates et variables
- Découvrir Ansible Vault pour la gestion des secrets
- Se préparer à des déploiements en production

Le projet est conçu pour être :
- **Pédagogique** : Tout est commenté pour l'apprentissage
- **Propre** : Structure claire et organisée
- **Réaliste** : Architecture multi-environnement comme en entreprise
- **Sécurisé** : Utilisation de Vault pour les secrets

## Architecture du lab

### Machines du lab

Nous avons actuellement 3 machines dans le lab :

1. **Controller Ansible** (`atlos-mgmt-ans-01`)
   - Rôle : Nœud de contrôle Ansible
   - Fonction : Lance les playbooks vers les cibles
   - Ne doit pas être géré comme cible principale

2. **Serveur Linux dev** (`atlos-dev-waz-01`)
   - IP : `172.20.10.4`
   - Rôle : Serveur de développement / sécurité
   - Environnement : `dev`
   - Héberge : Wazuh Manager, nginx

3. **Machine de simulation staging** (`atlos-staging-kali-01`)
   - IP : `172.20.10.6`
   - Rôle : Machine de simulation / validation
   - Environnement : `staging`
   - Utilisation : Tests de sécurité, validation de configurations

### Environnement prod

L'environnement `prod` est prêt mais ne contient aucune machine pour l'instant. La structure est en place pour accueillir des serveurs de production.

## Rôles des machines

| Machine | IP | Rôle | Environnement | Description |
|---------|-----|-----|---------------|-------------|
| `atlos-mgmt-ans-01` | - | Controller | - | Nœud de contrôle Ansible |
| `atlos-dev-waz-01` | 172.20.10.4 | Wazuh Manager | dev | Serveur de développement |
| `atlos-staging-kali-01` | 172.20.10.6 | Simulation | staging | Tests de sécurité |

## Structure des dossiers

```
ansible-lab/
├── ansible.cfg                    # Configuration Ansible
├── README.md                      # Ce fichier
├── .gitignore                     # Fichiers ignorés par Git
├── inventories/                    # Inventaires multi-environnement
│   ├── dev/
│   │   ├── hosts.ini              # Inventaire dev
│   │   └── group_vars/
│   │       ├── all.yml            # Variables communes dev
│   │       ├── linux_servers.yml  # Variables serveurs Linux dev
│   │       ├── wazuh_managers.yml # Variables Wazuh dev
│   │       └── vault.yml.example  # Exemple Vault dev
│   ├── staging/
│   │   ├── hosts.ini              # Inventaire staging
│   │   └── group_vars/
│   │       ├── all.yml            # Variables communes staging
│   │       ├── linux_servers.yml  # Variables serveurs Linux staging
│   │       ├── linux_simulation.yml # Variables simulation staging
│   │       └── vault.yml.example  # Exemple Vault staging
│   └── prod/
│       ├── hosts.ini              # Inventaire prod (vide)
│       └── group_vars/
│           ├── all.yml            # Variables communes prod
│           ├── linux_servers.yml  # Variables serveurs Linux prod
│           └── vault.yml.example  # Exemple Vault prod
├── playbooks/                     # Playbooks Ansible
│   ├── validate_linux_access.yml  # Validation de l'accès SSH
│   ├── baseline_linux_servers.yml # Configuration de base Linux
│   ├── deploy_nginx_reverse_proxy.yml # Déploiement nginx
│   ├── deploy_wazuh_agent.yml    # Déploiement agent Wazuh (simulation)
│   └── site.yml                  # Orchestration globale
└── roles/                         # Rôles Ansible
    ├── common_linux/              # Configuration de base Linux
    │   ├── defaults/main.yml
    │   ├── handlers/main.yml
    │   ├── tasks/main.yml
    │   └── README.md
    ├── nginx_reverse_proxy/       # Serveur web nginx
    │   ├── defaults/main.yml
    │   ├── handlers/main.yml
    │   ├── tasks/main.yml
    │   ├── templates/index.html.j2
    │   └── README.md
    └── wazuh_agent/               # Agent Wazuh (simulation)
        ├── defaults/main.yml
        ├── handlers/main.yml
        ├── tasks/main.yml
        └── README.md
```

## Notion de multi-environnement

Ce projet utilise une architecture multi-environnement typique des entreprises :

### Dev (Développement)
- Objectif : Développement et tests rapides
- Configuration : Permissive (authentification par mot de passe activée)
- Données : Données de test, pas de production
- Monitoring : Désactivé
- Sauvegardes : Désactivées

### Staging (Pré-production)
- Objectif : Validation avant production
- Configuration : Proche de la production (authentification par clé SSH)
- Données : Données de test
- Monitoring : Activé
- Sauvegardes : Activées

### Prod (Production)
- Objectif : Environnement de production
- Configuration : Sécurisée (authentification par clé SSH uniquement)
- Données : Données réelles
- Monitoring : Activé avec alertes
- Sauvegardes : Activées avec rétention longue

**Pourquoi cette séparation ?**
- Évite de casser la production pendant les tests
- Permet de valider les changements avant déploiement
- Sépare clairement les données de test et de production
- Applique des niveaux de sécurité différents selon l'environnement

## Comment lancer les playbooks

### Prérequis

1. Ansible installé sur le nœud de contrôle
2. Accès SSH fonctionnel vers les hôtes de l'inventaire
3. Utilisateur distant autorisé à utiliser `become`
4. Clés SSH configurées pour l'authentification

### Lancer un playbook sur un environnement spécifique

```bash
# Environnement dev
ansible-playbook -i inventories/dev playbooks/validate_linux_access.yml

# Environnement staging
ansible-playbook -i inventories/staging playbooks/validate_linux_access.yml

# Environnement prod
ansible-playbook -i inventories/prod playbooks/validate_linux_access.yml
```

### Lancer avec Vault (si des secrets sont chiffrés)

```bash
ansible-playbook -i inventories/dev playbooks/site.yml --ask-vault-pass
```

### Playbooks disponibles

| Playbook | Description | Cible |
|----------|-------------|--------|
| `validate_linux_access.yml` | Valide l'accès SSH et affiche les infos système | `linux_servers` |
| `baseline_linux_servers.yml` | Applique la configuration de base Linux | `linux_servers` |
| `deploy_nginx_reverse_proxy.yml` | Déploie nginx sur les serveurs Wazuh | `wazuh_managers` |
| `deploy_wazuh_agent.yml` | Déploie l'agent Wazuh (simulation) | `linux_servers` |
| `site.yml` | Orchestration globale (tous les rôles) | `linux_servers` |

### Mode check (simulation sans modification)

```bash
ansible-playbook -i inventories/dev playbooks/site.yml --check --diff
```

### Mode verbose (pour le dépannage)

```bash
ansible-playbook -i inventories/dev playbooks/site.yml -vvv
```

## Bonnes pratiques SSH / user ansible

### Utilisateur dédié Ansible

Ce projet utilise un utilisateur dédié `ansible` pour les connexions SSH. C'est une bonne pratique de sécurité car :

- Sépare les comptes humains des comptes d'automatisation
- Permet de tracer les actions d'automatisation
- Facilite la gestion des permissions

### Configuration SSH recommandée

1. **Authentification par clé SSH uniquement**
   ```bash
   # Sur les serveurs cibles, désactiver l'authentification par mot de passe
   ssh_password_authentication: false  # Dans group_vars
   ```

2. **Interdire la connexion root directe**
   ```bash
   ssh_permit_root_login: false  # Dans group_vars
   ```

3. **Utiliser ~/.ssh/config pour la gestion des clés**
   ```sshconfig
   Host github.com
       HostName github.com
       User git
       IdentityFile /home/ansible/.ssh/id_ed25519
       IdentitiesOnly yes
   ```

4. **Permissions strictes sur les clés SSH**
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_ed25519
   chmod 644 ~/.ssh/id_ed25519.pub
   ```

### Distribution de clés avec ssh-copy-id

```bash
# Depuis le contrôleur Ansible
ssh-copy-id -i ~/.ssh/id_ed25519.pub ansible@172.20.10.4
```

## Notion de Vault

Ansible Vault est un outil intégré à Ansible pour chiffrer des données sensibles (mots de passe, clés API, etc.).

### Pourquoi utiliser Ansible Vault ?

- Sécurité : Les secrets ne sont pas en clair dans le dépôt Git
- Contrôle : Les secrets sont chiffrés avec un mot de passe maître
- Flexibilité : Les secrets peuvent être utilisés dans les playbooks comme des variables normales

### Créer un fichier Vault

```bash
# 1. Copier l'exemple
cp inventories/dev/group_vars/vault.yml.example inventories/dev/group_vars/vault.yml

# 2. Éditer avec vos vrais secrets
nano inventories/dev/group_vars/vault.yml

# 3. Chiffrer le fichier
ansible-vault encrypt inventories/dev/group_vars/vault.yml

# Vous serez invité à entrer un mot de passe pour le Vault
# CONSERVEZ CE MOT DE PASSE EN SÉCURITÉ
```

### Utiliser un fichier Vault dans un playbook

```bash
ansible-playbook -i inventories/dev playbooks/site.yml --ask-vault-pass
```

Ansible vous demandera le mot de passe Vault pour déchiffrer les variables.

### Modifier un fichier Vault chiffré

```bash
ansible-vault edit inventories/dev/group_vars/vault.yml
```

### Vérifier le contenu d'un fichier Vault

```bash
ansible-vault view inventories/dev/group_vars/vault.yml
```

### Déchiffrer un fichier Vault (non recommandé)

```bash
ansible-vault decrypt inventories/dev/group_vars/vault.yml
```

### Bonnes pratiques Vault

- Utilisez un mot de passe fort et unique pour le Vault
- Stockez le mot de passe Vault dans un gestionnaire de secrets sécurisé
- Ne commitez JAMAIS de secrets en clair dans Git
- Utilisez des fichiers `.example` pour montrer la structure
- Faites des sauvegardes des fichiers Vault

## Notion de rôles et handlers

### Qu'est-ce qu'un rôle Ansible ?

Un rôle est une unité d'organisation qui regroupe :
- Des tâches (`tasks/main.yml`)
- Des handlers (`handlers/main.yml`)
- Des variables par défaut (`defaults/main.yml`)
- Des fichiers (`files/`)
- Des templates (`templates/`)
- Des métadonnées (`meta/main.yml`)

Les rôles permettent de réutiliser du code Ansible et de le partager entre différents playbooks.

### Structure d'un rôle

```
role_name/
├── defaults/     # Variables par défaut (faible précédence)
│   └── main.yml
├── files/        # Fichiers statiques
├── handlers/     # Handlers (actions déclenchées par notify)
│   └── main.yml
├── meta/         # Métadonnées du rôle
│   └── main.yml
├── tasks/        # Tâches du rôle
│   └── main.yml
├── templates/    # Templates Jinja2
├── tests/        # Tests du rôle
├── vars/         # Variables (haute précédence)
│   └── main.yml
└── README.md     # Documentation du rôle
```

### Qu'est-ce qu'un handler ?

Un handler est une action qui ne s'exécute que lorsqu'une tâche le notifie via le mot-clé `notify`. Les handlers sont typiquement utilisés pour redémarrer des services après une modification de configuration.

**Pourquoi utiliser des handlers ?**
- Évite les redémarrages inutiles si la configuration n'a pas changé
- Centralise les actions de redémarrage
- Améliore la performance des playbooks

**Exemple de handler :**
```yaml
# Dans handlers/main.yml
- name: Redémarrer nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
  become: yes

# Dans tasks/main.yml
- name: Déploiement de la configuration nginx
  ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  become: yes
  notify: Redémarrer nginx  # Notifie le handler seulement si le fichier change
```

### Rôles de ce projet

#### common_linux
- Installe les paquets de base Linux
- Configure le MOTD
- Configure la timezone et la locale
- Configure les ulimits
- Crée le répertoire de logs
- Configure les alias bash

#### nginx_reverse_proxy
- Installe nginx
- Déploie une page d'accueil personnalisée (template Jinja2)
- Gère le service nginx
- Redémarre nginx uniquement si nécessaire (handler)

#### wazuh_agent
- **SIMULATION PÉDAGOGIQUE**
- Simule l'installation de l'agent Wazuh
- Crée une structure de fichiers exemple
- Pour une installation réelle, adapter selon la documentation Wazuh

## Exemples de commandes utiles

### Commandes Ansible

```bash
# Vérifier la connectivité
ansible -i inventories/dev linux_servers -m ping

# Collecter les facts
ansible -i inventories/dev linux_servers -m setup

# Exécuter une commande sur tous les hôtes
ansible -i inventories/dev linux_servers -m shell -a "uptime"

# Lister les hôtes d'un inventaire
ansible-inventory -i inventories/dev --list

# Vérifier la syntaxe d'un playbook
ansible-playbook --syntax-check playbooks/site.yml

# Lister les tâches d'un playbook
ansible-playbook playbooks/site.yml --list-tasks
```

### Commandes Git

```bash
# Initialiser le dépôt
git init

# Ajouter tous les fichiers
git add .

# Commiter
git commit -m "Initial commit"

# Ajouter une remote
git remote add origin git@github.com:VulneZe/ansible_lab.git

# Pousser
git push -u origin main

# Vérifier le statut
git status

# Voir les différences
git diff
```

### Commandes SSH

```bash
# Tester la connexion SSH
ssh ansible@172.20.10.4

# Copier une clé SSH
ssh-copy-id -i ~/.ssh/id_ed25519.pub ansible@172.20.10.4

# Générer une nouvelle clé SSH
ssh-keygen -t ed25519 -C "ansible@atlos"

# Vérifier les clés SSH
ls -la ~/.ssh/
```

## Logique de nommage des machines

Ce projet utilise une convention de nommage structurée pour les machines, typique des entreprises de taille moyenne à grande.

### Format du nommage

```
atlos-{environnement}-{rôle}-{index}
```

### Exemples de noms

| Nom | Environnement | Rôle | Index | Description |
|-----|---------------|------|-------|-------------|
| `atlos-dev-pc-01` | dev | pc | 01 | Poste de travail dev n°1 |
| `atlos-dev-pc-02` | dev | pc | 02 | Poste de travail dev n°2 |
| `atlos-dev-lnx-01` | dev | lnx | 01 | Serveur Linux dev n°1 |
| `atlos-prod-web-01` | prod | web | 01 | Serveur web prod n°1 |
| `atlos-prod-db-01` | prod | db | 01 | Serveur de base de données prod n°1 |
| `atlos-prod-app-01` | prod | app | 01 | Serveur applicatif prod n°1 |
| `atlos-staging-app-01` | staging | app | 01 | Serveur applicatif staging n°1 |

### Logique de nommage

1. **Entreprise** (`atlos`) : Identifie l'entreprise ou l'organisation
2. **Environnement** (`dev`, `staging`, `prod`) : Indique l'environnement
3. **Rôle** (`pc`, `lnx`, `web`, `db`, `app`) : Indique la fonction de la machine
4. **Index** (`01`, `02`, `03`) : Numéro séquentiel pour distinguer les machines similaires

### Codes de rôle courants

| Code | Description |
|------|-------------|
| `pc` | Poste de travail / Desktop |
| `lnx` | Serveur Linux générique |
| `web` | Serveur web |
| `db` | Serveur de base de données |
| `app` | Serveur applicatif |
| `mgmt` | Serveur de management |
| `sec` | Serveur de sécurité |
| `mon` | Serveur de monitoring |
| `bak` | Serveur de sauvegarde |

### Avantages de cette convention

- **Clarté** : Le nom de la machine indique son rôle et son environnement
- **Organisation** : Facilite la gestion des inventaires
- **Automatisation** : Permet de générer automatiquement des configurations
- **Recherche** : Facilite la recherche de machines spécifiques

## Conclusion

Ce projet est une base solide pour apprendre Ansible avec des bonnes pratiques d'entreprise. N'hésitez pas à :

- Explorer les rôles et les comprendre
- Modifier les variables dans les inventaires
- Ajouter de nouveaux rôles pour vos besoins
- Adapter le projet à votre infrastructure réelle
- Contribuer avec vos améliorations

Pour toute question ou suggestion, n'hésitez pas à contacter l'équipe DevOps ATLOS : devops@atlos.local

---

**Bonne découverte d'Ansible ! 🚀**

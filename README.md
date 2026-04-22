# Cynto - Déploiement Active Directory avec Ansible
## Présentation

Ce projet permet le déploiement automatisé d’une infrastructure Active Directory sur des serveurs Windows à l’aide d’Ansible.

Il couvre notamment :

- La préparation des hôtes Windows
- La configuration réseau
- L’installation et la configuration d’Active Directory
- La gestion sécurisée des secrets

L’objectif est de fournir une base fiable, reproductible et maintenable pour le déploiement d’un environnement Active Directory.

## Prérequis

- Ansible installé sur la machine de contrôle
- Accès réseau aux serveurs Windows cibles
- Comptes administrateurs sur les machines Windows
- Python non requis sur les hôtes Windows (utilisation de WinRM)

## Préparation des serveurs Windows

Avant toute exécution des playbooks, chaque serveur Windows doit être configuré pour accepter les connexions Ansible via WinRM.

Sur chaque serveur Windows, ouvrir PowerShell en administrateur et exécuter :
### Activation de WinRM

```PowerShell
winrm quickconfig
```

Confirmer avec `y` si demandé.
### Activation du remoting PowerShell

```Powershell
Enable-PSRemoting -Force
```
### Configuration de l’authentification (environnement de lab)

```PowerShell
Set-Item WSMan:\localhost\Service\AllowUnencrypted -Value $true    
Set-Item WSMan:\localhost\Service\Auth\Basic -Value $true
```
### Ouverture du pare-feu

```PowerShell
Enable-NetFirewallRule -Name *WinRM*
```

Important : cette configuration est adaptée à un environnement de test. En production, privilégier WinRM en HTTPS et l’authentification Kerberos.

## Structure du projet

```bash
Cynto-Ansible-AD/  
├── inventories/  
│   └── prod/  
│       ├── group_vars/  
│       │   └── all/  
│       │       └── vault.yml  
├── playbooks/  
├── roles/  
├── host_vars/  
└── README.md
```
## Gestion des secrets avec Ansible Vault

Les informations sensibles (mots de passe, comptes, etc.) sont stockées dans un fichier chiffré avec Ansible Vault.
### Création du fichier Vault

Créer le fichier Vault :

```bash
ansible-vault create inventories/prod/group_vars/all/vault.yml
```

Puis y ajouter les variables suivantes :

```YAML
vault_ad_safe_mode_password: "xxx"  
vault_domain_admin_password: "xxx"  
vault_default_user_password: "xxx"  
vault_windows_admin_password: "xxx"  
vault_service_account_password: "xxx"
```
### Consultation du fichier Vault

```bash
ansible-vault view inventories/prod/group_vars/all/vault.yml
```

Un mot de passe Vault sera demandé.
### Modification du fichier Vault

```
ansible-vault edit inventories/prod/group_vars/all/vault.yml
```
## Configuration de l’inventaire

Exemple d’inventaire :

```INI
[windows]  
srv-ad-01 ansible_host=10.8.40.10  
srv-ad-02 ansible_host=10.8.40.11
```

---
## Vérification de la connectivité

Avant le déploiement, tester la communication avec les serveurs :

```bash
ansible windows -m win_ping
```

## Exécution des playbooks

Lancer le déploiement complet :

```bash
ansible-playbook playbooks/00-site.yml --ask-vault-pass
```

## Sécurité

- Éviter l’utilisation de `AllowUnencrypted` en production
- Utiliser WinRM en HTTPS
- Privilégier l’authentification Kerberos
- Protéger les secrets avec Ansible Vault
- Restreindre les accès réseau aux ports nécessaires

## Maintenabilité

- Structurer les rôles de manière modulaire
- Centraliser les variables dans `group_vars` et `host_vars`
- Documenter chaque rôle
- Versionner les configurations

## Auteur

Nicolas Pelissier

# 3. Durcissement d'image et gestion des secrets

**Objectif :** deux sujets clés en sécurité cloud — préparer une **image durcie** réutilisable, et **ne jamais stocker un mot de passe en clair** grâce à Azure Key Vault.

> Toutes les valeurs sensibles (secrets, IDs, IP) sont masquées dans les extraits.

## A. Image de base durcie

Le principe : partir d'une VM, la nettoyer et la généraliser, puis en faire un **template** à partir duquel on lance des VM propres et identiques.

Étapes de nettoyage avant de figer l'image :

```
# Réinitialiser cloud-init
$ sudo rm -rf /var/lib/cloud/*

# Nettoyer les logs et l'historique
$ sudo rm -rf /var/log/*.log
$ history -c

# Déprovisionner l'agent Azure (supprime clés SSH host, baux DHCP, user)
$ sudo waagent -deprovision+user -force
```

Puis création du template et lancement d'une nouvelle VM à partir de lui :

```
PS> az vm deallocate --resource-group test --name tp3b
PS> az vm generalize --resource-group test --name tp3b
PS> az image create --resource-group test --name tp3 --source tp3b
PS> az vm create -g test -n tp3 --image tp3 --admin-username <USER> ...
```

Vérification que la nouvelle VM boote correctement (`cloud-init status: done`).

## B. Gestion des secrets avec Azure Key Vault

**Le problème :** une application a besoin du mot de passe de sa base de données. L'écrire en clair dans un fichier `.env` ou dans le code est une faute de sécurité classique.

**La solution :** stocker le secret dans un coffre **Azure Key Vault**, et laisser la VM le récupérer toute seule grâce à son **identité managée** (Managed Identity), sans aucun mot de passe stocké nulle part.

Création du coffre et d'un secret :

```
PS> az keyvault create --name <VAULT> --resource-group test --location francecentral
PS> az keyvault secret set --vault-name "<VAULT>" --name "db-password" --value "<SECRET>"
```

Autorisation donnée à l'identité de la VM (lecture seule des secrets) :

```
PS> az keyvault set-policy --name "<VAULT>" --object-id <ID_VM> --secret-permissions get list
```

Script qui tourne au démarrage de l'application pour injecter le secret dans le `.env` :

```bash
#!/bin/bash
# get_secrets.sh — récupère le mot de passe depuis Key Vault via l'identité de la VM
az login --identity --allow-no-subscriptions

DB_PASSWORD=$(az keyvault secret show \
    --vault-name "<VAULT>" --name "DBPASSWORD" --query "value" -o tsv)

sed -i "s/^DB_PASSWORD=.*/DB_PASSWORD=$DB_PASSWORD/" /opt/app/.env
```

Ce script est branché en `ExecStartPre=` du service systemd de l'application : le secret est récupéré **juste avant** chaque démarrage, et jamais stocké durablement en clair.

## Ce que j'en retiens

Un secret ne doit jamais vivre dans le code ni dans un fichier versionné. Un coffre + une identité managée permettent à une machine de prouver son identité et de lire ses secrets, sans qu'un mot de passe maître traîne quelque part.

## Outils

Azure CLI · Azure Key Vault · Managed Identity · cloud-init · systemd · Bash

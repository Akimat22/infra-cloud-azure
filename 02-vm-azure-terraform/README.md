# 2. Machines virtuelles Azure (CLI, SSH, Terraform)

**Objectif :** créer et administrer des machines virtuelles dans le cloud **Microsoft Azure**, de trois façons : à la main (Azure CLI), via un modèle réutilisable, et enfin en **Infrastructure as Code** avec **Terraform**.

> Les identifiants réels (subscription, tenant, IP publiques, clés) ont été remplacés par des valeurs génériques du type `<VOTRE_SUBSCRIPTION_ID>`.

## Prérequis : clés SSH et agent

Génération d'une paire de clés **ed25519** (algorithme moderne, recommandé pour SSH) et configuration de l'agent SSH pour ne pas retaper la passphrase à chaque connexion.

```
$ ssh-keygen -t ed25519
$ eval "$(ssh-agent -s)"
$ ssh-add ~/.ssh/id_ed25519
```

## Créer une VM depuis le Azure CLI

```
PS> az vm create -g test -n super_vm --size Standard_B2ats_v2 \
    --image almalinux:almalinux-x86_64:10-gen2:latest \
    --admin-username <USER> --ssh-key-values "<CLE_PUBLIQUE_SSH>"

{
  "location": "switzerlandnorth",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "<IP_PUBLIQUE_VM>",
  "resourceGroup": "test"
}
```

Connexion SSH sur l'IP publique, puis vérification que les agents Azure tournent bien :

```
$ systemctl status waagent.service    # Azure Linux Agent
$ systemctl status cloud-init.service # Cloud-init
```

## Infrastructure as Code avec Terraform

Plutôt que de cliquer ou de taper des commandes une à une, je décris toute l'infrastructure dans des fichiers. Terraform crée alors le groupe de ressources, le réseau virtuel, le sous-réseau, l'IP publique, la carte réseau et la VM, en une seule commande `terraform apply`.

Fichiers de ce dossier :
- [`main.tf`](main.tf) — les ressources Azure (réseau, VM AlmaLinux…)
- [`variables.tf`](variables.tf) — les variables (nom du groupe, région, user…)
- [`terraform.tfvars.example`](terraform.tfvars.example) — exemple de valeurs à copier et remplir

```
$ terraform init
$ terraform plan
$ terraform apply
azurerm_resource_group.main: Creation complete after 26s
azurerm_virtual_network.main: Creation complete after 6s
azurerm_linux_virtual_machine.main: Creating...
```

## Ce que j'en retiens

L'IaC change tout : l'infrastructure devient un code versionné, relisable et reproductible. On peut détruire et recréer un environnement identique en quelques minutes, au lieu de refaire des dizaines de clics sans trace.

## Outils

Azure CLI · Terraform · AlmaLinux · SSH (ed25519) · cloud-init · waagent

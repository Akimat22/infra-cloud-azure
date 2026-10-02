# Infrastructure cloud : Docker, Azure & Terraform

Labs d'administration cloud et conteneurs, du Bachelor Cybersécurité & Ethical Hacking (EFREI, 2026). On y déploie une application web complète, d'abord en conteneurs, puis sur des machines virtuelles Azure créées en Infrastructure as Code, avec une vraie attention portée à la sécurité (images durcies, scan de vulnérabilités, gestion des secrets).

> **Note sécurité :** tous les identifiants réels (subscription Azure, tenant, IP publiques, clés SSH, secrets) ont été remplacés par des valeurs génériques. Le vrai `terraform.tfvars` est exclu par le [`.gitignore`](.gitignore).

## Sommaire

1. [Conteneurs Docker](01-conteneurs-docker/) — Dockerfile, docker-compose, scan Trivy, benchmark CIS
2. [Machines virtuelles Azure](02-vm-azure-terraform/) — Azure CLI, SSH, et Infrastructure as Code avec Terraform
3. [Durcissement et secrets](03-durcissement-et-secrets/) — image durcie réutilisable, Azure Key Vault + identité managée

## Compétences

| Domaine | Technologies |
|---|---|
| Conteneurs | Docker, docker-compose, Trivy |
| Cloud | Microsoft Azure, Azure CLI, Key Vault |
| Infrastructure as Code | Terraform |
| Systèmes | AlmaLinux, Ubuntu, systemd, cloud-init |
| Sécurité | durcissement d'image, gestion de secrets, scan de vulnérabilités |

## Fil rouge

Les labs tournent autour d'une même petite application web Flask (fournie en cours) avec sa base de données. L'idée est de la déployer de plus en plus proprement : conteneurisée, puis sur des VM reproductibles via Terraform, puis avec ses secrets sortis du code et placés dans un coffre.

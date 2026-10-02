# 1. Conteneurs Docker

**Objectif :** créer mes propres images Docker, orchestrer plusieurs conteneurs avec `docker-compose`, puis regarder la sécurité des conteneurs.

## Image Apache maison (Dockerfile)

Une image Debian minimale qui sert une page web avec Apache :

```dockerfile
# Image de base Debian (minimale)
FROM debian:12-slim

# Mise à jour du système et installation d'Apache
RUN apt-get update && \
    apt-get install -y apache2 && \
    apt-get clean

# Copier la page HTML personnalisée
COPY index.html /var/www/html/index.html

# Exposer le port 80
EXPOSE 80

# Lancer Apache au premier plan
CMD ["apache2ctl", "-D", "FOREGROUND"]
```

Construction et lancement :

```
$ docker build -t my_apache_image .
$ docker run -d -p 8080:80 my_apache_image
```

## Orchestration avec docker-compose

J'ai déployé un **WikiJS** (wiki moderne) avec sa base PostgreSQL, puis conteneurisé une **application Flask** fournie en cours, avec sa base **Redis**, le tout décrit dans un seul `docker-compose.yml`. Les conteneurs se joignent par leur nom de service sur le réseau Docker (`db`, par exemple).

```
$ docker compose up -d
 ✔ Container wikijs-db    Started
 ✔ Container wikijs-app   Started
```

## Sécurité des conteneurs

Trois points étudiés :

1. **Le groupe `docker` = root.** Un utilisateur membre du groupe `docker` peut devenir root sur l'hôte, sans mot de passe, en montant le système de fichiers dans un conteneur privilégié. Démonstration faite en lisant `/etc/shadow` depuis un conteneur lancé en `--user root`.

2. **Scan de vulnérabilités avec [Trivy](https://github.com/aquasecurity/trivy).** J'ai scanné les images `postgres:15`, `requarks/wiki:2` et `nginx:latest` pour repérer les CVE connues des paquets qu'elles embarquent.

   ```
   $ trivy image postgres:15
   INFO  Detected OS  family="debian" version="13.4"
   INFO  [debian] Detecting vulnerabilities...  pkg_num=142
   ```

3. **Benchmark CIS avec [docker-bench-security](https://github.com/docker/docker-bench-security).** Cet outil vérifie la configuration de Docker par rapport aux recommandations du CIS (Center for Internet Security).

## Ce que j'en retiens

Un conteneur n'est pas une boîte étanche par défaut : les droits sur le socket Docker, les images de base non à jour et les privilèges excessifs sont autant de portes d'entrée. Scanner ses images et suivre un référentiel comme le CIS fait partie du travail.

## Outils

Docker · docker-compose · Debian · Trivy · docker-bench-security

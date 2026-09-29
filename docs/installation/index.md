---
title: Installation
description: Installer et vérifier AWS CLI v2 sur Linux et macOS
icon: material/download
---

# Installation d'AWS CLI

AWS CLI v2 est l'outil officiel pour interagir avec S3 et les autres services AWS
depuis le terminal. Cette section couvre l'installation sur Linux et macOS.

## Vérifier une installation existante

```bash
aws --version
```

Résultat attendu (version 2.x) :

```
aws-cli/2.x.y Python/3.x.y Linux/... botocore/2.x.y
```

Si la commande est introuvable ou affiche une version 1.x, suivre la procédure ci-dessous.

---

## Installation sur macOS

=== "Homebrew (recommandé)"

    ```bash
    brew install awscli
    aws --version
    ```

=== "Installateur officiel PKG"

    ```bash
    curl -fsSLO "https://awscli.amazonaws.com/AWSCLIV2.pkg"
    sudo installer -pkg AWSCLIV2.pkg -target /
    aws --version
    rm AWSCLIV2.pkg
    ```

---

## Installation sur Linux

=== "x86_64"

    ```bash
    curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
      -o awscli.zip
    unzip awscli.zip
    sudo ./aws/install
    rm -rf awscli.zip aws/
    aws --version
    ```

=== "ARM64 (aarch64)"

    ```bash
    curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" \
      -o awscli.zip
    unzip awscli.zip
    sudo ./aws/install
    rm -rf awscli.zip aws/
    aws --version
    ```

=== "Mise à jour"

    Si AWS CLI est déjà installé via l'installateur officiel :

    ```bash
    curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
      -o awscli.zip
    unzip awscli.zip
    sudo ./aws/install --update
    rm -rf awscli.zip aws/
    ```

!!! note "Gestionnaire de paquets système"
    Les versions packagées (`apt`, `dnf`, `yum`) sont souvent en retard.
    L'installateur officiel est recommandé pour disposer de la dernière version.

---

## Fichiers de configuration

AWS CLI utilise deux fichiers dans `~/.aws/` :

| Fichier | Contenu |
|---------|----------|
| `~/.aws/config` | Paramètres : région, format de sortie, rôles à assumer, SSO |
| `~/.aws/credentials` | Clés d'accès IAM (access key + secret key) |

Les variables d'environnement ont priorité sur les fichiers, elles-mêmes
ayant priorité sur les profils.

```
Priorité (haute → basse)
  1. Variables d'environnement  (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY…)
  2. Profil CLI (~/.aws/config + ~/.aws/credentials)
  3. Metadata d'instance EC2 / ECS task role
```

!!! tip "Emplacements personnalisés"
    Les variables `AWS_CONFIG_FILE` et `AWS_SHARED_CREDENTIALS_FILE`
    permettent de déplacer ces fichiers (utile pour des scripts d'automatisation
    ou des environnements multi-comptes isolés).

---

## Vérifier la connexion

Une fois un profil configuré (cf. [Authentification](../authentification/index.md)),
tester l'accès avec :

```bash
aws sts get-caller-identity --profile <MON_PROFIL>
```

Résultat attendu :

```json
{
    "UserId": "AIDA...",
    "Account": "<ACCOUNT_ID>",
    "Arn": "arn:aws:iam::<ACCOUNT_ID>:user/<MON_UTILISATEUR>"
}
```

Cette commande ne consomme pas de ressources et constitue le test de connectivité
de référence.

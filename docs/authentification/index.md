---
title: Authentification
description: Configurer les profils AWS CLI, SSO, rôles IAM et multi-profils
icon: material/shield-key
---

# Authentification

AWS CLI repose sur le concept de **profil** : un ensemble nommé de paramètres
de connexion stocké dans `~/.aws/config`. Plusieurs profils peuvent coexister,
chacun correspondant à un compte, un rôle ou un environnement distinct.

---

## Profil de base — clés IAM longue durée

!!! warning "Usage limité"
    Les clés IAM permanentes sont déconseillées pour un usage humain quotidien.
    Réserver ce mode aux comptes de service sans accès SSO disponible.

```bash
aws configure --profile <MON_PROFIL>
```

Répondre aux invites :

```
AWS Access Key ID [None]: <ACCESS_KEY_ID>
AWS Secret Access Key [None]: <SECRET_ACCESS_KEY>
Default region name [None]: <S3_REGION>
Default output format [None]: json
```

Les valeurs sont stockées dans :

- `~/.aws/credentials` : les clés
- `~/.aws/config` : région et format

Pour un service S3 privé (compatible S3, non AWS), ajouter l'endpoint :

```bash
aws configure set endpoint_url <S3_ENDPOINT_URL> \
  --profile <MON_PROFIL>
```

---

## Profil SSO — IAM Identity Center (recommandé)

L'authentification SSO délivre des credentials temporaires via le navigateur
et ne stocke aucune clé permanente sur le poste.

### Configuration initiale

```bash
aws configure sso --profile <MON_PROFIL>
```

Exemple de valeurs :

```
SSO session name (Recommended): sso-session-prod
SSO start URL [None]: https://mon-organisation.awsapps.com/start
SSO region [None]: <S3_REGION>
SSO registration scopes [sso:account:access]: sso:account:access
```

La section correspondante dans `~/.aws/config` :

```ini
[sso-session sso-session-prod]
sso_start_url = https://mon-organisation.awsapps.com/start
sso_region = <S3_REGION>
sso_registration_scopes = sso:account:access

[profile <MON_PROFIL>]
sso_session = sso-session-prod
sso_account_id = <ACCOUNT_ID>
sso_role_name = AdministrateurS3
region = <S3_REGION>
output = json
```

### Connexion / renouvellement

```bash
aws sso login --profile <MON_PROFIL>
```

Le navigateur s'ouvre pour valider l'authentification. Les credentials
temporaires sont ensuite mis en cache dans `~/.aws/sso/cache/`.

### Déconnexion

```bash
aws sso logout
```

---

## Profil avec rôle IAM (AssumeRole)

Permet d'élever les droits ou d'accéder à un compte différent depuis
un profil source qui dispose de la permission `sts:AssumeRole`.

```ini
# ~/.aws/config

# Profil source (identité de base, peut être SSO ou clés IAM)
[profile base-<MON_PROFIL>]
sso_session = sso-session-prod
sso_account_id = <ACCOUNT_ID>
sso_role_name = LectureSeule
region = <S3_REGION>
output = json

# Profil cible : assume un rôle dans un autre compte
[profile prod-admin]
source_profile = base-<MON_PROFIL>
role_arn = <ROLE_ARN>
role_session_name = session-<MON_UTILISATEUR>
region = <S3_REGION>
output = json
duration_seconds = 3600
```

L'appel STS est effectué automatiquement par la CLI lors du premier usage :

```bash
aws sts get-caller-identity --profile prod-admin
```

---

## Multi-profils : organisation et bonnes pratiques

### Exemple de `~/.aws/config` multi-environnements

```ini
# ─── Sessions SSO ────────────────────────────────────────────────────────────

[sso-session corp]
sso_start_url = https://mon-organisation.awsapps.com/start
sso_region = eu-west-3
sso_registration_scopes = sso:account:access

# ─── Environnement développement ─────────────────────────────────────────────

[profile dev]
sso_session = corp
sso_account_id = 111111111111
sso_role_name = DeveloppeurS3
region = eu-west-3
output = json
endpoint_url = <S3_ENDPOINT_URL>     # optionnel si service privé

# ─── Environnement staging ────────────────────────────────────────────────────

[profile staging]
sso_session = corp
sso_account_id = 222222222222
sso_role_name = DeveloppeurS3
region = eu-west-3
output = json

# ─── Environnement production (via rôle) ─────────────────────────────────────

[profile prod]
source_profile = staging
role_arn = arn:aws:iam::333333333333:role/AdministrateurS3Production
role_session_name = session-<MON_UTILISATEUR>
region = eu-west-3
output = json
duration_seconds = 3600
```

### Sélectionner un profil

=== "Option --profile"

    ```bash
    aws s3 ls --profile dev
    aws s3 ls --profile staging
    aws s3 ls --profile prod
    ```

=== "Variable d'environnement"

    La variable `AWS_PROFILE` s'applique à toutes les commandes de la session courante :

    ```bash
    export AWS_PROFILE=prod
    aws s3 ls s3://<MON_BUCKET>/
    aws sts get-caller-identity
    unset AWS_PROFILE
    ```

=== "Script shell"

    Pour un script qui doit cibler un profil précis sans polluer le shell courant :

    ```bash
    #!/usr/bin/env bash
    set -euo pipefail

    PROFIL="prod"

    aws s3 ls s3://<MON_BUCKET>/ --profile "${PROFIL}"
    ```

### Lister les profils configurés

```bash
aws configure list-profiles
```

### Vérifier le profil actif

```bash
aws sts get-caller-identity --profile <MON_PROFIL>
```

---

## Variables d'environnement de référence

| Variable | Rôle |
|----------|------|
| `AWS_PROFILE` | Nom du profil à utiliser |
| `AWS_ACCESS_KEY_ID` | Clé d'accès IAM ou STS |
| `AWS_SECRET_ACCESS_KEY` | Clé secrète IAM ou STS |
| `AWS_SESSION_TOKEN` | Token de session STS (obligatoire avec credentials temporaires) |
| `AWS_DEFAULT_REGION` | Région par défaut |
| `AWS_ENDPOINT_URL` | Endpoint S3 personnalisé (service privé) |
| `AWS_CONFIG_FILE` | Chemin alternatif vers `~/.aws/config` |
| `AWS_SHARED_CREDENTIALS_FILE` | Chemin alternatif vers `~/.aws/credentials` |

!!! danger "Sécurité"
    Ne jamais exporter `AWS_SECRET_ACCESS_KEY` ou `AWS_SESSION_TOKEN`
    dans un fichier `.env` versionné. Utiliser un gestionnaire de secrets
    (Vault, AWS Secrets Manager, SOPS…) pour les environnements partagés.

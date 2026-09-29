---
title: Documentation S3
description: Guide utilisateur complet pour l'accès et la gestion des données S3
icon: material/home
---

# Documentation S3

Bienvenue dans la documentation utilisateur du service de stockage objet S3.

Ce guide couvre l'installation et la configuration de l'outil en ligne de commande `aws`,
la gestion des identités et des accès, ainsi que l'ensemble des opérations courantes
sur les buckets et les objets.

## Organisation de la documentation

| Section | Contenu |
|---------|----------|
| [Installation](installation/index.md) | Installation d'AWS CLI et configuration initiale |
| [Concepts](concepts/index.md) | Buckets, objets, IAM, STS — comprendre le modèle |
| [Authentification](authentification/index.md) | Profils, SSO, rôles IAM et multi-profils |
| [Buckets](buckets/index.md) | Lister, créer, naviguer dans les buckets |
| [Transfert](transfert/index.md) | Envoyer, télécharger, synchroniser des données |
| [Recherche](recherche/index.md) | Trouver des objets par nom, préfixe ou filtre |

## Placeholders utilisés dans cette documentation

Les valeurs entre chevrons `<...>` sont des **placeholders** à remplacer par vos propres valeurs :

| Placeholder | Signification |
|-------------|---------------|
| `<S3_ENDPOINT_URL>` | URL du point d'accès S3 (ex: `https://s3.example.com`) |
| `<S3_REGION>` | Région AWS ou région du service privé (ex: `eu-west-3`, `us-east-1`) |
| `<MON_BUCKET>` | Nom de votre bucket S3 |
| `<MON_PROFIL>` | Nom du profil AWS CLI configuré localement |
| `<ACCOUNT_ID>` | Identifiant numérique du compte AWS (12 chiffres) |
| `<ROLE_ARN>` | ARN complet du rôle IAM à assumer |
| `<ACCESS_KEY_ID>` | Identifiant de la clé d'accès IAM |
| `<SECRET_ACCESS_KEY>` | Clé secrète associée (ne jamais partager) |
| `<SESSION_TOKEN>` | Token de session STS (credentials temporaires) |
| `<MON_UTILISATEUR>` | Nom d'utilisateur IAM |
| `<MON_GROUPE>` | Nom du groupe IAM |

!!! info "Service S3 compatible"
    Cette documentation s'applique aussi bien à AWS S3 qu'à tout service compatible
    S3 (MinIO, Ceph RGW, Scality, OVHcloud Object Storage…).
    Dans ce cas, précisez systématiquement `--endpoint-url <S3_ENDPOINT_URL>` dans vos commandes
    ou configurez-le dans votre profil (cf. [Authentification](authentification/index.md)).

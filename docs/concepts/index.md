---
title: Concepts
description: Comprendre les buckets, objets, IAM users, groupes et STS
icon: material/book-open-variant
---

# Concepts fondamentaux

Avant d'utiliser le CLI, il est essentiel de comprendre le modèle de stockage S3
et le modèle de contrôle d'accès IAM/STS.

## Stockage objet S3

S3 (**Simple Storage Service**) est un stockage objet accessible via HTTP/HTTPS.
Contrairement à un système de fichiers, il n'y a pas de répertoires réels :
les objets sont simplement identifiés par une **clé** (leur chemin complet).

### Bucket

Un **bucket** est un conteneur racine, globalement nommé dans une région :

- Le nom est **unique à l'échelle mondiale** (ou du service privé).
- Il contient des **objets**, chacun identifié par une **clé**.
- Il porte les politiques d'accès, le versionnement et les règles de cycle de vie.
- URI de référence : `s3://<MON_BUCKET>/`

### Objet

Un **objet** est la donnée stockée. Il est composé de :

- Une **clé** (chemin logique) : `backups/postgres/2026-09-29.sql`
- Un **corps** (le contenu binaire du fichier)
- Des **métadonnées** (type MIME, checksum, tags, etc.)
- Une **version** si le versionnement est activé sur le bucket

Les « dossiers » affichés dans les interfaces ne sont que des préfixes de clés.
Un objet nommé `logs/app/2026/access.log` donne l'illusion d'une arborescence
`logs/app/2026/` mais ce chemin n'a pas d'existence propre.

```
s3://<MON_BUCKET>/
├── logs/
│   └── app/
│       └── 2026/
│           └── access.log      ← clé : logs/app/2026/access.log
├── backups/
│   └── postgres/
│       └── 2026-09-29.sql      ← clé : backups/postgres/2026-09-29.sql
└── README.txt                  ← clé : README.txt
```

---

## Contrôle d'accès : IAM

**IAM** (**Identity and Access Management**) est le service qui gère
les identités et leurs permissions sur les ressources AWS.

### User IAM

Un **IAM user** est une identité persistante attachée à un compte AWS :

- Il peut avoir un mot de passe Console et/ou des **clés d'accès** (Access Key + Secret Key).
- Il reçoit des permissions via des **politiques** (policies) attachées directement ou via des groupes.
- Les clés d'accès longue durée sont déconseillées pour les humains ; préférer SSO ou les rôles.

### Group IAM

Un **IAM group** est un regroupement logique d'utilisateurs :

- Il permet d'attacher des politiques à plusieurs utilisateurs en une seule opération.
- Il **ne constitue pas** une identité de connexion : on ne peut pas assumer un groupe.
- Un utilisateur peut appartenir à plusieurs groupes.

```
Groupe : s3-readers
  └── Politique : AmazonS3ReadOnlyAccess
        ├── user : alice
        ├── user : bob
        └── user : <MON_UTILISATEUR>
```

### Role IAM

Un **IAM role** est une identité sans clés permanentes :

- Il définit des permissions et une **politique de confiance** (qui peut l'assumer).
- Une entité autorisée (user, service, instance EC2, CI/CD…) peut l'« assumer » via STS.
- Mécanisme recommandé pour les accès inter-comptes, les services AWS et les pipelines.

---

## STS — Credentials temporaires

**STS** (**Security Token Service**) délivre des credentials à durée limitée
en échange de l'identité actuelle et d'un rôle cible.

Un appel `sts:AssumeRole` retourne un triplet :

| Champ | Description |
|-------|-------------|
| `AccessKeyId` | Identifiant de la clé temporaire |
| `SecretAccessKey` | Secret temporaire |
| `SessionToken` | Token de session (obligatoire avec les deux précédents) |
| `Expiration` | Date d'expiration (par défaut 1h, configurable jusqu'à 12h) |

Ces trois valeurs sont transmises ensemble dans chaque requête signée.
Après expiration, elles sont inutilisables : il faut les renouveler.

```
alice (IAM user)
  │  sts:AssumeRole
  ▼
arn:aws:sts::<ACCOUNT_ID>:assumed-role/MonRole/session-alice
  │  credentials temporaires (1h)
  ▼
s3://production-bucket/  ← accès autorisé par la politique du rôle
```

!!! warning "Ne jamais stocker de credentials en clair"
    Les clés IAM permanentes et les tokens STS ne doivent jamais apparaître
    dans des fichiers versionnés, des variables d'environnement de CI visibles,
    ou des logs applicatifs.

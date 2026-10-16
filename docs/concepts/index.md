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

## ARN : adresser une ressource AWS

Un **ARN** (*Amazon Resource Name*) est l'identifiant unique qui permet de
référencer une ressource AWS dans une politique IAM, une commande `aws` ou la
configuration d'un service. Il ne donne pas de permission à lui seul : les
permissions viennent des policies IAM et des contrôles propres au service.

### Syntaxe générale

```text
arn:<partition>:<service>:<region>:<account-id>:<resource>
```

| Élément | Rôle | Exemple |
|---|---|---|
| `partition` | Partition AWS (`aws`, `aws-us-gov`, `aws-cn`…) | `aws` |
| `service` | Namespace du service ciblé | `iam`, `s3` |
| `region` | Région de la ressource ; parfois vide ou non applicable | `eu-west-3` |
| `account-id` | Compte propriétaire ; parfois vide ou non applicable | `123456789012` |
| `resource` | Type et nom/chemin de la ressource | `role/ReadOnly` |

Les ARN ne sont pas tous structurés de la même manière : le séparateur de la
partie `resource` peut être `/`, `:` ou un autre format documenté par le
service. Les caractères génériques `*` et `?` sont utilisables dans certains
éléments d'une policy IAM, mais pas comme règle universelle dans tous les
contextes.

### ARN IAM

IAM est un service global : ses ARN n'ont généralement ni région ni identifiant
de compte dans le champ `region`.

```text
arn:aws:iam::<ACCOUNT_ID>:user/<USER_NAME>
arn:aws:iam::<ACCOUNT_ID>:group/<GROUP_NAME>
arn:aws:iam::<ACCOUNT_ID>:role/<ROLE_NAME>
arn:aws:iam::<ACCOUNT_ID>:policy/<POLICY_NAME>
```

Par exemple, la valeur de `role_arn` dans un profil `AssumeRole` est un ARN IAM
complet :

```ini
role_arn = arn:aws:iam::123456789012:role/AdministrateurS3Production
```

À ne pas confondre avec l'ARN STS d'une session temporaire, qui représente
l'identité assumée et non le rôle permanent :

```text
arn:aws:sts::<ACCOUNT_ID>:assumed-role/<ROLE_NAME>/<SESSION_NAME>
```

### ARN S3

S3 est global pour le nom du bucket, tandis que les objets sont adressés dans
le bucket par leur clé. Les ARN S3 usuels sont donc :

```text
# Bucket entier
arn:aws:s3:::<BUCKET_NAME>

# Tous les objets du bucket
arn:aws:s3:::<BUCKET_NAME>/*

# Un objet ou un préfixe précis
arn:aws:s3:::<BUCKET_NAME>/<OBJECT_KEY>
arn:aws:s3:::<BUCKET_NAME>/backups/*
```

Le bucket ARN ne se termine pas par `/`. Pour les actions sur les objets
(`s3:GetObject`, `s3:PutObject`, `s3:DeleteObject`), utiliser l'ARN objet avec
`/*` ou le préfixe approprié. Pour les actions sur le bucket (`s3:ListBucket`,
`s3:GetBucketLocation`), utiliser l'ARN du bucket sans `/*`.

!!! note "ARN, URI et URL"
    `s3://<BUCKET_NAME>/<OBJECT_KEY>` est une URI de navigation/CLI ;
    `arn:aws:s3:::<BUCKET_NAME>/<OBJECT_KEY>` est un identifiant de policy.
    Ce ne sont pas des formats interchangeables.

Exemple minimal de policy IAM donnant le listing d'un bucket et la lecture de
ses objets :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::<BUCKET_NAME>"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<BUCKET_NAME>/*"
    }
  ]
}
```

---

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

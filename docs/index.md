---
title: Documentation S3
description: Parcours guidé pour comprendre et utiliser S3
icon: material/home
---

# Documentation S3

Bienvenue dans un parcours progressif pour utiliser S3 avec AWS CLI, sur AWS ou avec un service compatible S3.

## Parcours recommandé

1. [Démarrer](01-demarrer/index.md) — installer AWS CLI et préparer un profil.
2. [Comprendre S3](02-comprendre-s3/index.md) — buckets, objets, clés et protocole.
3. [Requêtes et protocole](03-requetes-protocole/index.md) — HTTP, signature et pagination.
4. [IAM et authentification](04-iam-authentification/index.md) — profils, SSO, rôles et STS.
5. [Contrôle d’accès](05-controle-acces/index.md) — policies, ARN et évaluation des droits.
6. [AWS CLI centralisé](06-cli/index.md) — paramètres communs et commandes sans répétition.
7. [Opérations courantes](07-operations/index.md) — buckets, transferts et recherche.
8. [Dépannage et sécurité](08-depannage-securite/index.md) — diagnostiquer sans exposer de secrets.

Les anciennes pages thématiques restent disponibles depuis les guides concernés afin de conserver les explications et exemples détaillés.

!!! info "AWS et services compatibles S3"
    Remplacez les valeurs entre chevrons (`<S3_ENDPOINT_URL>`, `<S3_REGION>`, `<MON_BUCKET>`, `<MON_PROFIL>`). Pour un service tiers, utilisez son endpoint et sa région de signature ; ne désactivez pas TLS.

---
title: Comprendre S3
icon: material/book-open-variant
---

# Comprendre S3

S3 est un stockage objet : un **bucket** contient des objets identifiés par une **clé**. Les dossiers affichés sont des préfixes de clés, pas des répertoires.

- [Concepts fondamentaux : buckets, objets, IAM et STS](../concepts/index.md)
- [Opérations et navigation dans les buckets](../buckets/index.md)
- [Requêtes et protocole S3](../03-requetes-protocole/index.md)

```text
s3://<MON_BUCKET>/logs/app/2026/access.log
       bucket       clé (préfixe + nom de l’objet)
```

Une requête peut être autorisée sur le bucket (lister) mais refusée sur l’objet (lire). Cette distinction explique de nombreux `AccessDenied`.

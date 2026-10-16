---
title: Recherche d'objets
description: Trouver des fichiers dans les buckets S3 par préfixe, nom ou filtre
icon: material/magnify
---

# Recherche d'objets

S3 ne propose pas de recherche plein texte sur les clés : listez les objets puis filtrez côté client. Réduire le préfixe améliore les performances.

```bash
aws s3 ls s3://<MON_BUCKET>/logs/app/ \
  --recursive --profile <MON_PROFIL> | grep -F "rapport"
```

## Recherche avec `s3api` et `--query`

```bash
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> --prefix backups/ \
  --query 'Contents[].Key' --output text --profile <MON_PROFIL>
```

Les listings sont paginés automatiquement par AWS CLI. Pour agréger toutes les pages, conservez ce comportement par défaut. **N’utilisez pas `--no-paginate`** : cette option limite volontairement la commande au premier appel.

Pour contrôler la taille d’une page, utilisez `--max-keys`; pour reprendre une requête manuellement, utilisez le `NextContinuationToken` renvoyé par l’API.

- [Requêtes et protocole](../03-requetes-protocole/index.md)
- [Opérations courantes](../07-operations/index.md)

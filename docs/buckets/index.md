---
title: Buckets
description: Lister, naviguer et inspecter les buckets S3
icon: material/bucket
---

# Gestion des buckets

Cette section couvre les opérations de navigation et d'inspection des buckets
et de leur contenu depuis la CLI.

---

## Lister les buckets accessibles

```bash
# Tous les buckets du compte
aws s3 ls --profile <MON_PROFIL>
```

Résultat :

```
2026-01-15 10:00:00 mon-bucket-prod
2026-03-01 08:30:00 mon-bucket-staging
2026-05-10 14:22:00 <MON_BUCKET>
```

!!! note "Service S3 privé"
    Pour un service compatible S3 (MinIO, Ceph…), les buckets affichés sont ceux
    accessibles avec les credentials du profil, pas tous les buckets du service.

---

## Lister le contenu d'un bucket

### Racine du bucket

```bash
aws s3 ls s3://<MON_BUCKET>/ --profile <MON_PROFIL>
```

### Préfixe spécifique (sous-dossier logique)

```bash
aws s3 ls s3://<MON_BUCKET>/backups/ --profile <MON_PROFIL>
```

### Listing récursif avec résumé

```bash
aws s3 ls s3://<MON_BUCKET>/backups/ \
  --recursive \
  --human-readable \
  --summarize \
  --profile <MON_PROFIL>
```

Résultat :

```
2026-09-29 02:00:05    1.2 MiB backups/postgres/2026-09-29.sql
2026-09-28 02:00:03    1.1 MiB backups/postgres/2026-09-28.sql
...

Total Objects: 30
   Total Size: 35.4 MiB
```

---

## Inspecter les métadonnées d'un objet

```bash
aws s3api head-object \
  --bucket <MON_BUCKET> \
  --key backups/postgres/2026-09-29.sql \
  --profile <MON_PROFIL>
```

Résultat :

```json
{
    "ContentLength": 1258496,
    "ContentType": "binary/octet-stream",
    "ETag": "\"d41d8cd98f00b204e9800998ecf8427e\"",
    "LastModified": "2026-09-29T02:00:05+00:00",
    "Metadata": {}
}
```

---

## Inspecter la configuration d'un bucket

Ces commandes utilisent `s3api` (interface bas niveau) :

=== "Politique d'accès"

    ```bash
    aws s3api get-bucket-policy \
      --bucket <MON_BUCKET> \
      --profile <MON_PROFIL> \
      --query Policy \
      --output text | python3 -m json.tool
    ```

=== "Versionnement"

    ```bash
    aws s3api get-bucket-versioning \
      --bucket <MON_BUCKET> \
      --profile <MON_PROFIL>
    ```

=== "Règles de cycle de vie"

    ```bash
    aws s3api get-bucket-lifecycle-configuration \
      --bucket <MON_BUCKET> \
      --profile <MON_PROFIL>
    ```

=== "ACL"

    ```bash
    aws s3api get-bucket-acl \
      --bucket <MON_BUCKET> \
      --profile <MON_PROFIL>
    ```

---

## Filtrer la sortie avec `--query`

AWS CLI supporte les expressions **JMESPath** via `--query` pour filtrer
les résultats JSON :

```bash
# Lister uniquement les clés d'objets d'un préfixe
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --prefix backups/ \
  --query 'Contents[].Key' \
  --output text \
  --profile <MON_PROFIL>
```

```bash
# Objets modifiés après une date
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --prefix logs/ \
  --query 'Contents[?LastModified>=`2026-09-01`].{Key: Key, Size: Size}' \
  --output table \
  --profile <MON_PROFIL>
```

---

## Changer le format de sortie

Le format par défaut du profil peut être surchargé avec `--output` :

| Format | Usage |
|--------|-------|
| `json` | Traitement programmatique (jq, Python) |
| `table` | Lecture humaine dans le terminal |
| `text` | Parsing shell simple (awk, cut) |
| `yaml` | Lisibilité YAML |

```bash
aws s3 ls s3://<MON_BUCKET>/ --profile <MON_PROFIL> --output table
```

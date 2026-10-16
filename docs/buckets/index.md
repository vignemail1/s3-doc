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

## Commandes S3 fréquemment utilisées

La commande `aws s3` fournit les opérations de haut niveau les plus courantes.
Utiliser `--dryrun` avant toute opération qui supprime ou remplace des données.

| Commande | Usage |
|---|---|
| `aws s3 ls` | Lister des buckets ou un préfixe |
| `aws s3 mb` | Créer un bucket |
| `aws s3 rb` | Supprimer un bucket vide (ou son contenu avec `--force`) |
| `aws s3 cp` | Copier un fichier ou un préfixe |
| `aws s3 mv` | Déplacer/renommer un objet |
| `aws s3 sync` | Synchroniser deux emplacements |
| `aws s3 rm` | Supprimer un objet ou un préfixe |
| `aws s3 presign` | Générer une URL temporaire |

### Créer ou supprimer un bucket

```bash
# La région doit être cohérente avec la configuration du bucket
aws s3 mb s3://<NOUVEAU_BUCKET> \\
  --region <REGION> \\
  --profile <MON_PROFIL>

# Prévisualiser puis supprimer un bucket et son contenu
aws s3 rb s3://<MON_BUCKET> --force --dryrun --profile <MON_PROFIL>
aws s3 rb s3://<MON_BUCKET> --force --profile <MON_PROFIL>
```

`--force` supprime d'abord les objets ; l'utiliser uniquement après avoir
vérifié le bucket ciblé et, si nécessaire, le versionnement et les sauvegardes.

### Générer une URL présignée

Une URL présignée permet un accès temporaire sans distribuer les credentials
AWS. La durée maximale dépend de la méthode d'authentification utilisée ; la
valeur par défaut est d'une heure.

```bash
aws s3 presign s3://<MON_BUCKET>/partage/rapport.pdf \\
  --expires-in 3600 \\
  --profile <MON_PROFIL>
```

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
aws s3 ls s3://<MON_BUCKET>/backups/ \\
  --recursive \\
  --human-readable \\
  --summarize \\
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
aws s3api head-object \\
  --bucket <MON_BUCKET> \\
  --key backups/postgres/2026-09-29.sql \\
  --profile <MON_PROFIL>
```

Résultat :

```json
{
    "ContentLength": 1258496,
    "ContentType": "binary/octet-stream",
    "ETag": "\\\"d41d8cd98f00b204e9800998ecf8427e\\\"",
    "LastModified": "2026-09-29T02:00:05+00:00",
    "Metadata": {}
}
```

---

## Inspecter la configuration d'un bucket

Ces commandes utilisent `s3api` (interface bas niveau) :

=== "Politique d'accès"

    ```bash
    aws s3api get-bucket-policy \\
      --bucket <MON_BUCKET> \\
      --profile <MON_PROFIL> \\
      --query Policy \\
      --output text | python3 -m json.tool
    ```

=== "Versionnement"

    ```bash
    aws s3api get-bucket-versioning \\
      --bucket <MON_BUCKET> \\
      --profile <MON_PROFIL>
    ```

=== "Règles de cycle de vie"

    ```bash
    aws s3api get-bucket-lifecycle-configuration \\
      --bucket <MON_BUCKET> \\
      --profile <MON_PROFIL>
    ```

=== "ACL"

    ```bash
    aws s3api get-bucket-acl \\
      --bucket <MON_BUCKET> \\
      --profile <MON_PROFIL>
    ```

---

## Filtrer la sortie avec `--query`

AWS CLI supporte les expressions **JMESPath** via `--query` pour filtrer
les résultats JSON :

```bash
# Lister uniquement les clés d'objets d'un préfixe
aws s3api list-objects-v2 \\
  --bucket <MON_BUCKET> \\
  --prefix backups/ \\
  --query 'Contents[].Key' \\
  --output text \\
  --profile <MON_PROFIL>
```

```bash
# Objets modifiés après une date
aws s3api list-objects-v2 \\
  --bucket <MON_BUCKET> \\
  --prefix logs/ \\
  --query 'Contents[?LastModified>=`2026-09-01`].{Key: Key, Size: Size}' \\
  --output table \\
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

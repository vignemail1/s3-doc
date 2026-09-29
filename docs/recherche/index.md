---
title: Recherche d'objets
description: Trouver des fichiers dans les buckets S3 par préfixe, nom ou filtre
icon: material/magnify
---

# Recherche d'objets

S3 ne propose pas de recherche plein texte sur les clés : il faut lister
les objets puis filtrer côté client. Cette section présente les patterns
les plus courants.

---

## Principes de base

`aws s3 ls` affiche les objets du niveau immédiatement sous le préfixe demandé.
`--recursive` traverse toute l'arborescence de clés sous ce préfixe.

```bash
# Un seul niveau (comme ls sans -R)
aws s3 ls s3://<MON_BUCKET>/backups/ --profile <MON_PROFIL>

# Toute l'arborescence sous backups/
aws s3 ls s3://<MON_BUCKET>/backups/ --recursive --profile <MON_PROFIL>
```

---

## Recherche par nom de fichier

```bash
# Trouver tous les objets dont la clé contient "rapport-2026"
aws s3 ls s3://<MON_BUCKET>/ \
  --recursive \
  --profile <MON_PROFIL> \
  | grep -F "rapport-2026"
```

### Insensible à la casse

```bash
aws s3 ls s3://<MON_BUCKET>/ \
  --recursive \
  --profile <MON_PROFIL> \
  | grep -iF "rapport"
```

---

## Recherche par extension

```bash
# Uniquement les fichiers .sql
aws s3 ls s3://<MON_BUCKET>/backups/ \
  --recursive \
  --profile <MON_PROFIL> \
  | awk '$4 ~ /\.sql$/ {print $4}'
```

```bash
# Plusieurs extensions : .gz et .tar.gz
aws s3 ls s3://<MON_BUCKET>/ \
  --recursive \
  --profile <MON_PROFIL> \
  | grep -E '\.(gz|tar\.gz)$'
```

---

## Recherche dans un préfixe précis

Réduire le périmètre au préfixe le plus précis possible améliore les performances
sur les buckets de grande taille (S3 liste jusqu'à 1 000 objets par requête API
et pagine automatiquement).

```bash
# Fichiers de log du mois de septembre 2026 sous logs/app/
aws s3 ls s3://<MON_BUCKET>/logs/app/ \
  --recursive \
  --profile <MON_PROFIL> \
  | grep "2026-09-"
```

---

## Recherche avec `s3api` et `--query` JMESPath

`list-objects-v2` offre plus de contrôle que `s3 ls` :

### Lister toutes les clés d'un préfixe

```bash
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --prefix backups/ \
  --query 'Contents[].Key' \
  --output text \
  --profile <MON_PROFIL>
```

### Objets modifiés après une date

```bash
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --prefix logs/ \
  --query 'Contents[?LastModified>=`2026-09-01T00:00:00`].{Key: Key, Taille: Size, Modifie: LastModified}' \
  --output table \
  --profile <MON_PROFIL>
```

### Objets de plus de 100 Mo

```bash
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --query 'Contents[?Size>`104857600`].{Key: Key, Mo: Size}' \
  --output table \
  --profile <MON_PROFIL>
```

---

## Pagination sur les grands buckets

`s3 ls --recursive` et `s3api list-objects-v2` paginent automatiquement.
Pour un contrôle explicite avec `s3api` :

```bash
# Premier appel — note le NextContinuationToken si IsTruncated=true
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --prefix logs/ \
  --max-keys 1000 \
  --profile <MON_PROFIL> \
  > /tmp/page1.json

# Page suivante
NEXT_TOKEN=$(jq -r '.NextContinuationToken' /tmp/page1.json)

aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --prefix logs/ \
  --max-keys 1000 \
  --continuation-token "${NEXT_TOKEN}" \
  --profile <MON_PROFIL>
```

!!! tip "Script de collecte complète"
    Pour agréger toutes les pages, utiliser l'option `--no-paginate`
    d'AWS CLI qui gère la pagination automatiquement et retourne
    l'ensemble des résultats en un seul flux :

    ```bash
    aws s3api list-objects-v2 \
      --bucket <MON_BUCKET> \
      --prefix logs/ \
      --no-paginate \
      --output json \
      --profile <MON_PROFIL> \
      | jq '.Contents[].Key'
    ```

---

## Compter les objets et calculer la taille totale

```bash
# Nombre d'objets et taille totale d'un préfixe
aws s3 ls s3://<MON_BUCKET>/backups/ \
  --recursive \
  --human-readable \
  --summarize \
  --profile <MON_PROFIL> \
  | tail -3
```

```bash
# Via s3api avec jq
aws s3api list-objects-v2 \
  --bucket <MON_BUCKET> \
  --prefix backups/ \
  --no-paginate \
  --profile <MON_PROFIL> \
  | jq '[.Contents[].Size] | {total_objets: length, taille_totale_octets: add}'
```

---

## Recherche cross-buckets

S3 ne permet pas nativement de chercher dans plusieurs buckets en une commande.
Un pattern shell simple :

```bash
#!/usr/bin/env bash
set -euo pipefail

PROFIL="<MON_PROFIL>"
PATTERN="rapport-annuel"
BUCKETS=("bucket-prod" "bucket-staging" "bucket-archives")

for bucket in "${BUCKETS[@]}"; do
  echo "=== ${bucket} ==="
  aws s3 ls "s3://${bucket}/" \
    --recursive \
    --profile "${PROFIL}" \
    | grep -F "${PATTERN}" || true
done
```

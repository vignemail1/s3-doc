---
title: Transfert de données
description: Envoyer, télécharger, déplacer et synchroniser des objets S3
icon: material/transfer
---

# Transfert de données

Les commandes `cp`, `mv` et `sync` couvrent l'ensemble des cas d'usage
de transfert entre le système de fichiers local et S3.

---

## Référence rapide des commandes

| Commande | Action |
|----------|--------|
| `aws s3 cp` | Copier un objet ou une arborescence |
| `aws s3 mv` | Déplacer (copier + supprimer la source) |
| `aws s3 sync` | Synchroniser deux emplacements (local↔S3 ou S3↔S3) |
| `aws s3 rm` | Supprimer un objet ou un préfixe |

---

## Envoyer des données vers S3

### Fichier unique

```bash
aws s3 cp /chemin/local/fichier.txt \
  s3://<MON_BUCKET>/destination/fichier.txt \
  --profile <MON_PROFIL>
```

### Dossier complet (récursif)

```bash
aws s3 cp /chemin/local/dossier/ \
  s3://<MON_BUCKET>/destination/dossier/ \
  --recursive \
  --profile <MON_PROFIL>
```

### Avec type MIME explicite

```bash
aws s3 cp rapport.pdf \
  s3://<MON_BUCKET>/rapports/rapport.pdf \
  --content-type application/pdf \
  --profile <MON_PROFIL>
```

### Avec métadonnées personnalisées

```bash
aws s3 cp dump.sql \
  s3://<MON_BUCKET>/backups/dump.sql \
  --metadata "env=production,created-by=<MON_UTILISATEUR>" \
  --profile <MON_PROFIL>
```

---

## Télécharger des données depuis S3

### Fichier unique

```bash
aws s3 cp s3://<MON_BUCKET>/backups/dump.sql \
  /tmp/restore/dump.sql \
  --profile <MON_PROFIL>
```

### Arborescence complète

```bash
aws s3 cp s3://<MON_BUCKET>/exports/ \
  /tmp/exports/ \
  --recursive \
  --profile <MON_PROFIL>
```

### Téléchargement vers stdout (pipe)

```bash
aws s3 cp s3://<MON_BUCKET>/config/app.json - \
  --profile <MON_PROFIL> | jq .
```

---

## Synchronisation

`sync` compare les sources et la cible, et transfère **uniquement les objets
absents ou modifiés** (basé sur la taille et la date de dernière modification).
C'est la commande recommandée pour les sauvegardes et les déploiements.

### Local → S3

```bash
aws s3 sync /chemin/local/site/ \
  s3://<MON_BUCKET>/site/ \
  --profile <MON_PROFIL>
```

### S3 → Local

```bash
aws s3 sync s3://<MON_BUCKET>/exports/ \
  /chemin/local/exports/ \
  --profile <MON_PROFIL>
```

### S3 → S3 (entre buckets ou préfixes)

```bash
aws s3 sync s3://<MON_BUCKET>/source/ \
  s3://<MON_BUCKET>/destination/ \
  --profile <MON_PROFIL>

# Entre deux comptes (profils différents) : non supporté nativement,
# passer par le local ou un rôle avec accès aux deux buckets.
```

### Synchronisation avec suppression côté cible

```bash
# Prévisualiser d'abord
aws s3 sync /chemin/local/site/ \
  s3://<MON_BUCKET>/site/ \
  --delete \
  --dryrun \
  --profile <MON_PROFIL>

# Exécuter après validation
aws s3 sync /chemin/local/site/ \
  s3://<MON_BUCKET>/site/ \
  --delete \
  --profile <MON_PROFIL>
```

!!! danger "`--delete` est destructif"
    Avec `--delete`, tout objet présent dans la cible mais absent de la source
    est supprimé définitivement (sans versionnement, sans corbeille).
    Toujours lancer `--dryrun` avant la première exécution.

---

## Filtrer les transferts avec `--include` / `--exclude`

Les filtres opèrent sur la clé complète de l'objet. Ils sont évalués dans
l'ordre de déclaration : la dernière règle correspondante l'emporte.

### Pattern de base : exclure tout puis inclure un type

```bash
aws s3 sync /chemin/local/exports/ \
  s3://<MON_BUCKET>/exports/ \
  --exclude "*" \
  --include "*.json" \
  --profile <MON_PROFIL>
```

### Exclure un répertoire spécifique

```bash
aws s3 sync /chemin/local/projet/ \
  s3://<MON_BUCKET>/projet/ \
  --exclude ".git/*" \
  --exclude "node_modules/*" \
  --exclude "*.log" \
  --profile <MON_PROFIL>
```

### Copier uniquement les fichiers d'un certain mois

```bash
aws s3 cp s3://<MON_BUCKET>/logs/ /tmp/logs/ \
  --recursive \
  --exclude "*" \
  --include "2026-09-*.log" \
  --profile <MON_PROFIL>
```

!!! note "Ordre des filtres"
    `--exclude "*"` doit toujours précéder les `--include` pour que le pattern
    fonctionne comme un filtre inclusif. Sans cela, aucun objet n'est exclu par défaut.

---

## Supprimer des objets

### Objet unique

```bash
aws s3 rm s3://<MON_BUCKET>/tmp/fichier.txt \
  --profile <MON_PROFIL>
```

### Préfixe complet (récursif)

```bash
# Prévisualiser
aws s3 rm s3://<MON_BUCKET>/tmp/ \
  --recursive \
  --dryrun \
  --profile <MON_PROFIL>

# Exécuter
aws s3 rm s3://<MON_BUCKET>/tmp/ \
  --recursive \
  --profile <MON_PROFIL>
```

### Suppression filtrée

```bash
aws s3 rm s3://<MON_BUCKET>/logs/ \
  --recursive \
  --exclude "*" \
  --include "*.tmp" \
  --dryrun \
  --profile <MON_PROFIL>
```

---

## Options de transfert avancées

| Option | Description |
|--------|-------------|
| `--dryrun` | Afficher les actions sans les exécuter |
| `--no-progress` | Désactiver la barre de progression (utile pour les logs) |
| `--only-show-errors` | N'afficher que les erreurs |
| `--quiet` | Sortie silencieuse |
| `--sse aws:kms` | Chiffrement côté serveur avec KMS |
| `--storage-class` | Classe de stockage (`STANDARD`, `INTELLIGENT_TIERING`, `GLACIER`…) |
| `--acl` | ACL de l'objet (`private`, `public-read`…) |

### Exemple avec chiffrement KMS

```bash
aws s3 cp données-sensibles.tar.gz \
  s3://<MON_BUCKET>/archives/ \
  --sse aws:kms \
  --sse-kms-key-id arn:aws:kms:<S3_REGION>:<ACCOUNT_ID>:key/MON_KEY_ID \
  --profile <MON_PROFIL>
```

---

## Déplacer des objets

`mv` copie l'objet puis supprime la source. Utiliser avec la même
précaution que `rm`.

```bash
# Renommer un objet dans le même bucket
aws s3 mv s3://<MON_BUCKET>/ancien-nom.txt \
  s3://<MON_BUCKET>/nouveau-nom.txt \
  --profile <MON_PROFIL>

# Déplacer un préfixe entier
aws s3 mv s3://<MON_BUCKET>/staging/ \
  s3://<MON_BUCKET>/archive/ \
  --recursive \
  --profile <MON_PROFIL>
```

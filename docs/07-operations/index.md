---
title: Opérations courantes
icon: material/arrow-split
---

# Opérations courantes

Les pages existantes conservent les exemples détaillés ; cette page les relie selon la tâche recherchée.

- [Lister et inspecter les buckets](../buckets/index.md)
- [Transférer, supprimer et synchroniser](../transfert/index.md)
- [Commandes AWS CLI génériques](../transfert/commandes-cli.md)
- [Rechercher des objets](../recherche/index.md)

!!! warning "Actions destructives"
    Vérifiez le profil, le bucket et la clé. Utilisez `--dryrun` quand il est disponible avant `rm`, `rb`, `sync --delete` ou une opération équivalente.

La permission de lister un bucket n’implique pas celle de lire, écrire ou supprimer ses objets.

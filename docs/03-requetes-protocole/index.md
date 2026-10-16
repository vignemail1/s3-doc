---
title: Requêtes et protocole
icon: material/lan-connect
---

# Requêtes et protocole S3

Les commandes S3 deviennent des requêtes HTTP signées vers un endpoint. Le client doit connaître le bucket, la clé, la région de signature et les identifiants temporaires éventuels.

## Signature et transport

Utilisez HTTPS et laissez la validation TLS active. Une signature invalide, une mauvaise région ou une horloge système décalée peuvent produire `SignatureDoesNotMatch` ou `RequestTimeTooSkewed`.

## Pagination

Les listings sont paginés par l’API. AWS CLI pagine automatiquement par défaut. N’utilisez `--no-paginate` que si vous voulez volontairement limiter la sortie au premier appel ; ce n’est pas l’option à utiliser pour agréger toutes les pages.

Pour contrôler une page, utilisez plutôt `--max-keys` et, si nécessaire, le `NextContinuationToken` renvoyé par l’API. Voir [Recherche d’objets](../recherche/index.md).

## Style d’adressage

Selon le fournisseur, l’URL peut être en style virtual-hosted ou path-style. Suivez sa documentation et centralisez l’endpoint dans la configuration plutôt que de le recopier dans chaque exemple.

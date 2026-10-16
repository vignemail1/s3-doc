---
title: Dépannage et sécurité
icon: material/bug-check
---

# Dépannage et sécurité

## Diagnostic rapide

1. Vérifier `aws --version` et l’identité avec `aws sts get-caller-identity`.
2. Vérifier le profil, la région de signature et l’endpoint.
3. Distinguer un refus de listing (`s3:ListBucket`) d’un refus objet (`s3:GetObject`).
4. Contrôler l’horloge système, DNS, certificat TLS et connectivité.
5. Sur un grand listing, laisser la pagination automatique activée par défaut.

| Symptôme | Pistes |
|---|---|
| `AccessDenied` | identité, policy, ressource bucket/objet, préfixe |
| `SignatureDoesNotMatch` | région, endpoint, horloge, style d’adressage |
| `NoSuchBucket` | nom, endpoint et compte ciblé |
| expiration STS | renouveler la session, vérifier l’heure système |

## Bonnes pratiques

- Préférer SSO, rôles et credentials temporaires.
- Ne jamais versionner `~/.aws/credentials`, tokens, logs ou URLs présignées.
- Ne pas désactiver TLS pour contourner une erreur de certificat.
- Masquer les secrets avant de partager une sortie de commande.

Voir aussi [IAM et authentification](../04-iam-authentification/index.md) et [Contrôle d’accès](../05-controle-acces/index.md).

---
title: IAM et authentification
icon: material/shield-key
---

# IAM et authentification

Les profils AWS CLI isolent les identités et les environnements. Préférez SSO ou des rôles et limitez les clés longue durée aux cas nécessaires.

- [Profils, SSO, AssumeRole et multi-profils](../authentification/index.md)
- [Concepts IAM et STS](../concepts/index.md)
- [Contrôle d’accès et policies](../05-controle-acces/index.md)

```bash
aws configure sso --profile <MON_PROFIL>
aws sso login --profile <MON_PROFIL>
aws sts get-caller-identity --profile <MON_PROFIL>
```

Un accès STS comprend toujours `AccessKeyId`, `SecretAccessKey` et `SessionToken`, avec une expiration. Ne publiez aucune de ces valeurs.

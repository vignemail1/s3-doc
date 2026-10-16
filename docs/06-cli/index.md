---
title: AWS CLI centralisée
icon: material/console
---

# AWS CLI centralisée

Cette page définit les paramètres communs. Les pages d’opérations peuvent donc se concentrer sur l’action et éviter de répéter la configuration.

```bash
aws s3api <OPERATION> \
  --endpoint-url <S3_ENDPOINT_URL> \
  --region <S3_REGION> \
  --profile <MON_PROFIL>
```

Pour AWS, `--endpoint-url` est généralement omis. Pour un service compatible S3, utilisez l’endpoint fourni et ne remplacez pas une URL HTTPS par une URL non sécurisée.

## Choisir un profil

```bash
export AWS_PROFILE=<MON_PROFIL>
# ou ajoutez --profile <MON_PROFIL> à une commande précise
aws configure list-profiles
aws sts get-caller-identity
unset AWS_PROFILE
```

- [Configuration complète des profils](../authentification/index.md)
- [Commandes AWS CLI pour services compatibles S3](../transfert/commandes-cli.md)
- [Opérations courantes](../07-operations/index.md)

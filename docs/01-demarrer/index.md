---
title: Démarrer
icon: material/rocket-launch
---

# Démarrer

Installez AWS CLI v2, puis configurez un profil nommé plutôt que de placer des clés dans les commandes.

- [Installer AWS CLI v2](../installation/index.md)
- [Configurer un profil et vérifier l’identité](../authentification/index.md)
- [Paramètres communs de la CLI](../06-cli/index.md)

```bash
aws --version
aws configure --profile <MON_PROFIL>
aws sts get-caller-identity --profile <MON_PROFIL>
```

!!! tip "Service compatible S3"
    Conservez l’endpoint dans le profil ou utilisez `--endpoint-url <S3_ENDPOINT_URL>` sur les commandes S3. Vérifiez le comportement documenté par le fournisseur.

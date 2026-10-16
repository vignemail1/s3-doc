---
title: Contrôle d’accès
icon: material/key-chain
---

# Contrôle d’accès

L’autorisation dépend de l’identité appelante, de l’action, de la ressource et des politiques applicables. Les permissions de bucket et d’objet sont distinctes.

## ARN et ressources

Un ARN de bucket et un ARN d’objet ne ciblent pas la même ressource. Une policy qui autorise le bucket ne suffit pas forcément à lire `arn:aws:s3:::<MON_BUCKET>/*`.

## Examiner sans modifier

```bash
aws s3api get-bucket-policy --bucket <MON_BUCKET> --profile <MON_PROFIL>
aws s3api get-bucket-acl --bucket <MON_BUCKET> --profile <MON_PROFIL>
aws s3api head-object --bucket <MON_BUCKET> --key <CHEMIN_OBJET> --profile <MON_PROFIL>
```

Les noms d’actions de policy (`s3:GetObject`, `s3:ListBucket`, etc.) ne sont pas des commandes CLI. Consultez [Buckets](../buckets/index.md) et [Concepts](../concepts/index.md) avant de modifier une policy.

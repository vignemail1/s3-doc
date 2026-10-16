# Utiliser un service compatible S3 avec AWS CLI

AWS CLI sait communiquer avec de nombreux services compatibles S3. Les commandes ci-dessous visent une API S3 fournie par un service tiers : elles ne nécessitent pas un compte AWS. La compatibilité dépend du fournisseur.

## Paramètres communs

Préparez l’endpoint, la région de signature, le profil et les droits. Les valeurs entre chevrons sont des placeholders. Utilisez HTTPS et ne désactivez pas TLS.

```bash
aws s3api <OPERATION> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Pour éviter de répéter ces options, voir [AWS CLI centralisée](../06-cli/index.md). Pour les profils et STS, voir [IAM et authentification](../04-iam-authentification/index.md).

## Vérifier l’accès

```bash
aws s3api list-buckets \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Cette opération peut être refusée même si un bucket précis est accessible.

## Opérations courantes

```bash
aws s3api list-objects-v2 --bucket <BUCKET> --prefix <PREFIXE>/ \
  --endpoint-url <ENDPOINT_URL> --region <REGION_SIGNATURE> --profile <NOM_PROFIL>

aws s3 cp s3://<BUCKET>/<CHEMIN_OBJET> <CHEMIN_LOCAL> \
  --endpoint-url <ENDPOINT_URL> --region <REGION_SIGNATURE> --profile <NOM_PROFIL>

aws s3 cp <CHEMIN_LOCAL> s3://<BUCKET>/<CHEMIN_OBJET> \
  --endpoint-url <ENDPOINT_URL> --region <REGION_SIGNATURE> --profile <NOM_PROFIL>
```

Consultez [Opérations courantes](../07-operations/index.md) pour les transferts, suppressions et synchronisations. Les opérations de suppression sont destructives ; vérifiez toujours les chemins.

## Liens utiles

- [Contrôle d’accès](../05-controle-acces/index.md)
- [Requêtes et protocole](../03-requetes-protocole/index.md)
- [Documentation AWS sur les endpoints](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-endpoints.html)

# Utiliser un service compatible S3 avec AWS CLI

AWS CLI sait communiquer avec de nombreux services compatibles S3. L’outil est celui d’AWS, mais les commandes ci-dessous visent une API S3 fournie par un service tiers : elles ne nécessitent pas un compte AWS. La compatibilité dépend toutefois du fournisseur, de sa version d’API et des fonctionnalités activées.

## 1. Préparer les informations

Demandez à l’administrateur ou au fournisseur :

- l’URL de l’endpoint S3, par exemple `https://s3.<domaine-fournisseur>` ;
- la région à utiliser pour la signature (certains services imposent une valeur, d’autres ignorent ce paramètre) ;
- l’identifiant et la clé secrète, ou les identifiants temporaires STS ;
- les buckets auxquels vous avez accès et les opérations autorisées ;
- si le fournisseur requiert le style d’adressage `path` ou prend en charge le style `virtual`.

Dans les exemples, remplacez les valeurs entre chevrons, par exemple `<ENDPOINT_URL>`, `<BUCKET>` et `<PREFIXE>`. N’incluez pas les chevrons lors de l’exécution. Les commandes utilisent une URL HTTPS et ne désactivent pas la validation TLS.

Installez AWS CLI v2, puis vérifiez son installation :

```bash
aws --version
```

## 2. Configurer les identifiants

### Profil dédié avec identifiants permanents

Il est préférable d’utiliser un profil nommé plutôt que de mettre les clés dans une commande ou dans un script :

```bash
aws configure --profile <NOM_PROFIL>
```

Saisissez les valeurs demandées. Pour un service tiers, indiquez la région de signature communiquée par le fournisseur. Cette commande stocke les identifiants dans les fichiers de configuration locaux de l’utilisateur ; protégez ces fichiers et ne les ajoutez jamais au contrôle de version.

AWS CLI ne propose pas de champ standard `endpoint_url` dans `aws configure`. Passez donc `--endpoint-url` à chaque commande, ou configurez un endpoint par service dans `~/.aws/config` si la version et le comportement du fournisseur le permettent.

### Identifiants temporaires STS

Un accès temporaire comprend généralement un identifiant, une clé secrète et un jeton de session, tous trois fournis par l’émetteur STS. Configurez un profil temporaire :

```bash
aws configure set aws_access_key_id <ACCESS_KEY_ID> --profile <NOM_PROFIL_STS>
aws configure set aws_secret_access_key <SECRET_ACCESS_KEY> --profile <NOM_PROFIL_STS>
aws configure set aws_session_token <SESSION_TOKEN> --profile <NOM_PROFIL_STS>
aws configure set region <REGION_SIGNATURE> --profile <NOM_PROFIL_STS>
```

Ces valeurs expirent. Ne réutilisez pas un ancien jeton après expiration et ne le publiez pas dans un ticket, un dépôt ou un journal de commandes. Certains fournisseurs proposent un mécanisme de renouvellement différent : suivez leurs instructions.

> **Important :** la création ou l’échange d’identifiants STS (par exemple `AssumeRole`) se fait auprès de l’endpoint STS du fournisseur, avec des paramètres, une version et des permissions qui peuvent varier. Ne supposez pas qu’un endpoint AWS STS ou une commande AWS particulière est universelle. Utilisez la procédure documentée par le fournisseur pour obtenir les trois valeurs temporaires, puis configurez-les comme ci-dessus.

## 3. Paramètres communs

Dans chaque exemple, fournissez explicitement l’endpoint du service S3 :

```bash
aws s3api <OPERATION> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

`<ENDPOINT_URL>` est l’endpoint S3, pas l’URL d’un bucket. Avec AWS CLI v2, `--endpoint-url` s’applique à la commande concernée. Le profil détermine les identifiants et la région par défaut ; les options explicites rendent l’exemple plus lisible.

Pour tester l’authentification et l’accès à la liste des buckets :

```bash
aws s3api list-buckets \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Cette opération peut être refusée même si l’accès à un bucket précis est autorisé. L’absence de droit de lister tous les buckets ne signifie donc pas nécessairement que les opérations sur un bucket échoueront.

## 4. Actions courantes sur les buckets et les objets

Les exemples supposent que les droits nécessaires sont accordés par le fournisseur. Les noms d’actions de policy S3 ne sont pas des commandes CLI : la policy doit autoriser l’opération sous-jacente, éventuellement avec des droits distincts sur le bucket et les objets.

### Lister le contenu d’un bucket

Lister la racine :

```bash
aws s3api list-objects-v2 \
  --bucket <BUCKET> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Limiter la liste à un préfixe :

```bash
aws s3api list-objects-v2 \
  --bucket <BUCKET> \
  --prefix <PREFIXE>/ \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

La permission de lister porte généralement sur le bucket et peut être restreinte à un préfixe. Elle est distincte de la permission de lire les objets.

### Télécharger un objet

```bash
aws s3 cp s3://<BUCKET>/<CHEMIN_OBJET> <CHEMIN_LOCAL> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Variante bas niveau :

```bash
aws s3api get-object \
  --bucket <BUCKET> \
  --key <CHEMIN_OBJET> \
  <CHEMIN_LOCAL> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

### Envoyer un objet

```bash
aws s3 cp <CHEMIN_LOCAL> s3://<BUCKET>/<CHEMIN_OBJET> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

L’équivalent bas niveau pour un objet simple :

```bash
aws s3api put-object \
  --bucket <BUCKET> \
  --key <CHEMIN_OBJET> \
  --body <CHEMIN_LOCAL> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Les transferts `aws s3 cp` peuvent utiliser des transferts multipart pour les gros fichiers. Vérifiez que le fournisseur les prend en charge et que les permissions nécessaires sont accordées.

### Supprimer un objet

```bash
aws s3 rm s3://<BUCKET>/<CHEMIN_OBJET> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Ou avec l’API bas niveau :

```bash
aws s3api delete-object \
  --bucket <BUCKET> \
  --key <CHEMIN_OBJET> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

La suppression est une action destructive. Vérifiez le bucket et la clé avant de l’exécuter. La gestion des versions peut rendre nécessaire une suppression de version spécifique.

### Créer un bucket

À utiliser uniquement si le fournisseur autorise la création de buckets avec ces identifiants :

```bash
aws s3api create-bucket \
  --bucket <NOM_BUCKET_UNIQUE> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

La création de bucket, les contraintes de nommage, les paramètres de région et les options de configuration diffèrent selon les services. Consultez le fournisseur avant d’ajouter des options propres à AWS.

### Supprimer un bucket

Un bucket doit en général être vide avant sa suppression :

```bash
aws s3api delete-bucket \
  --bucket <BUCKET> \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Cette action est destructive et peut être irréversible. Ne l’exécutez que si vous êtes autorisé et certain du bucket visé.

## 5. Utiliser des commandes `aws s3` en lot

Copier un répertoire local dans un préfixe distant :

```bash
aws s3 cp <REPERTOIRE_LOCAL>/ s3://<BUCKET>/<PREFIXE>/ \
  --recursive \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

Synchroniser un répertoire local vers un préfixe distant :

```bash
aws s3 sync <REPERTOIRE_LOCAL>/ s3://<BUCKET>/<PREFIXE>/ \
  --endpoint-url <ENDPOINT_URL> \
  --region <REGION_SIGNATURE> \
  --profile <NOM_PROFIL>
```

`sync` compare les contenus et transfère les différences. L’option `--delete`, si ajoutée, supprime à destination les objets absents de la source : examinez soigneusement les chemins avant de l’utiliser.

## 6. Définir une policy d’accès (administrateurs)

La création et l’attachement de policies IAM ne sont pas des opérations de l’API S3 standard. AWS CLI expose des commandes IAM (`aws iam ...`), mais un fournisseur tiers peut avoir une API d’administration différente, des actions différentes ou ne pas exposer IAM via AWS CLI. Dans ce cas, appliquez la policy au moyen du portail ou des outils documentés par ce fournisseur.

Exemple indicatif de policy S3 en lecture sur un préfixe, à adapter et faire valider par le fournisseur :

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListerUnPrefixe",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::<BUCKET>",
      "Condition": {
        "StringLike": {
          "s3:prefix": ["<PREFIXE>", "<PREFIXE>/*"]
        }
      }
    },
    {
      "Sid": "LireLesObjetsDuPrefixe",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<BUCKET>/<PREFIXE>/*"
    }
  ]
}
```

Les ARN, actions, conditions et opérateurs supportés varient entre fournisseurs. Une policy générique ne peut pas être garantie compatible partout. Accordez uniquement les permissions nécessaires ; pour l’envoi, la suppression ou les opérations multipart, demandez les actions exactes requises au fournisseur.

## 7. Diagnostiquer les erreurs

- **`AccessDenied`** : vérifiez les permissions, le bucket, le préfixe, les actions autorisées et, pour STS, le jeton de session.
- **Erreur de signature ou `SignatureDoesNotMatch`** : vérifiez la région de signature, l’heure système, les identifiants, l’endpoint et la méthode d’adressage attendue.
- **Erreur de connexion ou certificat** : vérifiez l’URL de l’endpoint, le réseau et la chaîne TLS du fournisseur. N’utilisez pas `--no-verify-ssl` comme solution permanente.
- **Bucket introuvable** : vérifiez le nom, le compte ou tenant, l’endpoint et si l’opération est autorisée. Certains services masquent l’existence des ressources non autorisées.
- **Commande non prise en charge** : le fournisseur peut ne pas implémenter l’opération S3, une condition IAM ou l’option utilisée. Consultez sa matrice de compatibilité.

Pour examiner les paramètres effectifs sans afficher les secrets :

```bash
aws configure list --profile <NOM_PROFIL>
```

Évitez le mode `--debug` dans les journaux partagés : il peut révéler des informations sensibles.

## Références

- [Guide IAM et policies](../authentification/iam.md)
- [Accès temporaires STS](../authentification/sts.md)
- [AWS CLI — compatibilité des services tiers et endpoint URL](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-endpoints.html)

# IAM : identités et permissions

IAM (*Identity and Access Management*) désigne les mécanismes qui servent à gérer **qui peut accéder à quoi et avec quelles actions**. Les services compatibles S3 ne proposent pas tous les mêmes fonctions IAM : les noms des menus, les opérations disponibles et le format des policies peuvent varier.

> Cette page présente les concepts communs, pas une procédure propre à un fournisseur. Suivez la documentation de votre service pour confirmer les fonctionnalités prises en charge.

## Les concepts essentiels

| Concept | Rôle | Exemple |
|---|---|---|
| **Compte / tenant** | Espace qui regroupe les ressources et les utilisateurs | L’espace de votre organisation chez le fournisseur |
| **Utilisateur** | Identité durable qui peut s’authentifier | Un compte technique pour une application |
| **Groupe** | Regroupement d’utilisateurs pour appliquer des permissions communes | Équipe qui consulte les archives |
| **Rôle** | Identité ou ensemble de permissions assumé temporairement par un utilisateur, un service ou une application | Rôle de transfert utilisé pendant une heure |
| **Policy** | Document qui décrit les permissions accordées ou refusées | Autoriser la lecture d’un bucket précis |
| **Clé d’accès** | Identifiant et secret utilisés par certains clients S3 pour signer les requêtes | Access key ID et secret access key |
| **Ressource** | Élément auquel une action s’applique | Un bucket ou un objet dans un bucket |

Un utilisateur, une clé d’accès et un rôle ne sont pas synonymes. Une clé est un **moyen d’authentification** associé à une identité ; une policy décrit ses **permissions**. STS permet, quand le service le prend en charge, d’obtenir des identifiants temporaires pour un rôle ou une session.

## Comment une policy est évaluée

Une requête S3 est autorisée seulement si l’identité possède les droits nécessaires et si les règles applicables ne la refusent pas. En pratique :

1. Le client s’authentifie auprès du service.
2. Il demande une action, par exemple `s3:GetObject`.
3. Le service vérifie les policies associées à l’identité, au rôle et éventuellement à la ressource.
4. Un refus explicite prévaut généralement sur une autorisation. Les règles exactes dépendent du fournisseur.

N’accordez que les actions et ressources nécessaires. Évitez les accès globaux ou administrateur pour un usage courant.

## Anatomie d’une policy de type AWS

De nombreux services compatibles S3 acceptent tout ou partie du format de policy JSON popularisé par AWS. La compatibilité n’est pas garantie : vérifiez la documentation de votre fournisseur avant de réutiliser un exemple.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListerLeBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": ["arn:aws:s3:::mon-bucket"]
    },
    {
      "Sid": "LireLesObjets",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::mon-bucket/documents/*"]
    }
  ]
}
```

Les champs usuels sont :

- `Version` : version du langage de policy comprise par le service — ce n’est pas la version de S3.
- `Statement` : une ou plusieurs règles.
- `Sid` : nom facultatif, utile pour identifier une règle.
- `Effect` : `Allow` pour autoriser ou `Deny` pour refuser.
- `Action` : opérations concernées.
- `Resource` : ressources concernées.
- `Condition` : conditions facultatives, si elles sont prises en charge, par exemple une restriction de date ou de réseau.

Dans un ARN de style AWS, `arn:aws:s3:::mon-bucket` désigne le bucket ; `arn:aws:s3:::mon-bucket/documents/*` désigne les objets sous le préfixe `documents/`. Un préfixe ressemble à un dossier, mais les objets S3 sont identifiés par leur clé. Certains fournisseurs utilisent un autre format de ressource.

### Attention aux actions de bucket et d’objet

` s3:ListBucket` agit sur le **bucket**. Les actions telles que `s3:GetObject`, `s3:PutObject` et `s3:DeleteObject` agissent sur les **objets**. Une policy qui autorise la lecture d’objets ne permet pas nécessairement de lister le bucket ; une policy qui permet de lister ne permet pas de lire leur contenu.

Exemples d’actions fréquentes (liste indicative) :

| Besoin | Actions courantes |
|---|---|
| Lister le contenu d’un bucket | `s3:ListBucket` |
| Lire ou télécharger un objet | `s3:GetObject` |
| Déposer ou remplacer un objet | `s3:PutObject` |
| Supprimer un objet | `s3:DeleteObject` |
| Créer un bucket | `s3:CreateBucket` (si pris en charge) |

Les opérations multipart, le versioning, les ACL, le chiffrement et les fonctions de console peuvent nécessiter d’autres actions. Ne devinez pas les droits : vérifiez les erreurs et la matrice de permissions du service.

## Écrire une policy adaptée à un besoin

Avant de rédiger une policy, répondez à ces questions :

1. **Quelle identité ?** Utilisateur, groupe, application ou rôle.
2. **Quelles opérations ?** Lecture, dépôt, suppression, listage…
3. **Sur quelles ressources ?** Un bucket, un préfixe ou quelques objets.
4. **Pour combien de temps ?** Accès permanent ou session temporaire.
5. **Quelles restrictions ?** Réseau, durée, chiffrement ou autres conditions supportées.

Procédez ensuite progressivement :

- commencez par `Allow` sur les seules actions nécessaires ;
- ciblez le bucket et le préfixe utiles, au lieu de `*` ;
- séparez les permissions de bucket des permissions d’objet ;
- ajoutez des conditions uniquement après avoir confirmé leur syntaxe et leur prise en charge ;
- validez la syntaxe avec la console ou l’outil du fournisseur ;
- testez avec l’identité concernée et un objet non sensible.

### Exemple : dépôt et lecture dans un préfixe

L’exemple ci-dessous autorise le listage du bucket et la lecture/écriture sous `equipe-a/`. Il n’autorise pas la suppression. Adaptez le nom du bucket, le préfixe, le format ARN et les actions selon votre fournisseur.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListerBucket",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::mon-bucket"
    },
    {
      "Sid": "LireEtDeposerDansLePrefixe",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::mon-bucket/equipe-a/*"
    }
  ]
}
```

> Certaines API permettent de restreindre aussi le listage à un préfixe avec une condition. La clé de condition et sa compatibilité varient : ne copiez pas une condition AWS sans la vérifier.

## Où associer la policy ?

Selon le service, une policy peut être attachée à un utilisateur, un groupe, un rôle, un bucket ou une combinaison de ces éléments. D’autres services proposent des profils ou des règles d’accès avec une interface différente.

Pour configurer un accès :

1. Créez ou sélectionnez l’identité prévue pour l’usage (préférez un compte technique dédié pour une application).
2. Créez ou choisissez une policy minimale.
3. Associez-la à l’identité ou au rôle selon le modèle du fournisseur.
4. Générez des identifiants si nécessaire et notez l’endpoint S3 ainsi que la région exigée.
5. Testez uniquement les opérations prévues.
6. Révoquez les clés ou sessions devenues inutiles.

Une policy IAM n’est pas forcément la seule couche de contrôle : une policy de bucket, une règle d’organisation ou une ACL peut également intervenir. Le modèle de combinaison dépend du service.

## Bonnes pratiques

- Utilisez une identité par application ou par usage afin de pouvoir révoquer un accès sans interrompre les autres.
- Évitez les comptes root/administrateur pour les transferts courants.
- N’inscrivez jamais un secret dans le code, un dépôt Git, une capture d’écran ou une commande enregistrée dans l’historique partagé.
- Stockez les secrets dans un gestionnaire de secrets ou le mécanisme sécurisé de votre environnement.
- Faites tourner et révoquez les clés selon la politique de votre organisation.
- Commencez avec des permissions minimales et élargissez-les uniquement sur la base d’un besoin identifié.

## Dépannage des permissions

- **`AccessDenied`** : vérifiez l’identité réellement utilisée, l’action demandée, la ressource ciblée et les éventuels refus explicites ou contrôles au niveau du bucket.
- **Le bucket est accessible mais pas listable** : il manque peut-être la permission de listage (`s3:ListBucket`).
- **Le listing fonctionne mais le téléchargement échoue** : vérifiez l’autorisation de lecture de l’objet (`s3:GetObject`) et le chemin exact.
- **La policy est rejetée** : contrôlez le JSON, les actions, les ARN et les fonctionnalités supportées par le fournisseur.
- **Le client ne se connecte pas** : une erreur d’authentification, d’endpoint, de signature ou de région n’est pas nécessairement une erreur IAM.

Pour diagnostiquer, testez une action à la fois et ne partagez jamais vos clés secrètes dans une demande d’assistance.

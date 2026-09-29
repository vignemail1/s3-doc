# Authentification

Cette page explique comment préparer votre accès au stockage S3 et protéger vos identifiants. Les écrans et commandes exacts dépendent de l’outil choisi ; suivez les consignes fournies par votre administrateur pour les paramètres de votre service.

## Avant de commencer

Demandez ou vérifiez les éléments suivants :

- l’identifiant ou la clé d’accès ;
- la clé secrète correspondante ;
- l’adresse de l’endpoint S3, si votre service en utilise une spécifique ;
- la région, si elle est requise ;
- le bucket auquel vous avez accès et les opérations autorisées ;
- la méthode d’authentification attendue par votre outil.

> **Important :** les valeurs de connexion sont propres à votre compte et à votre service. Ne les devinez pas et ne copiez pas celles d’un exemple trouvé en ligne.

## Configurer l’accès

1. Ouvrez le gestionnaire d’identifiants ou la configuration de votre outil S3.
2. Saisissez les valeurs reçues de votre administrateur dans les champs correspondants.
3. Renseignez l’endpoint et la région uniquement selon les consignes de votre service.
4. Enregistrez la configuration dans l’emplacement prévu par l’outil.
5. Vérifiez la connexion avec une opération de lecture autorisée, par exemple l’affichage des buckets accessibles si votre outil le permet.

Les instructions propres à chaque outil sont indiquées dans [Installation](../installation/index.md) et [Transfert](../transfert/index.md). Ne placez pas un véritable secret dans une commande copiée dans un terminal, un script, une capture d’écran ou un dépôt de code.

## Protéger vos identifiants

- Ne partagez pas vos clés secrètes, mots de passe ou jetons, même pour une demande d’assistance.
- N’ajoutez jamais d’identifiants dans un dépôt Git, un ticket public ou un document partagé.
- Préférez le gestionnaire d’identifiants recommandé par votre outil plutôt qu’un fichier en clair.
- N’accordez que les droits nécessaires à votre tâche.
- Si vous pensez qu’un secret a été exposé, prévenez immédiatement l’administrateur afin qu’il puisse le révoquer ou le remplacer.

> **À ne pas confondre :** une clé d’accès identifie le compte ; une clé secrète sert à signer les requêtes. Les deux doivent rester confidentielles.

## En cas d’échec de connexion

1. Vérifiez que vous avez choisi le bon profil ou compte dans l’outil.
2. Contrôlez l’endpoint et la région à partir des informations officielles reçues.
3. Vérifiez que les identifiants n’ont pas été copiés avec un espace ou un caractère manquant.
4. Assurez-vous que votre compte est actif et que votre réseau peut joindre le service.
5. Demandez à l’administrateur de confirmer vos droits et l’état des identifiants.

Un message **Accès refusé** peut indiquer des droits insuffisants, même si l’authentification a réussi. Pour obtenir de l’aide, transmettez le message d’erreur sans inclure de secret.

# Transférer des fichiers

Cette page présente un parcours simple pour envoyer ou récupérer un objet. Les commandes précises varient selon l’outil ; utilisez les exemples de la section correspondant à votre client S3 et les paramètres communiqués par votre administrateur.

## Avant le transfert

Vérifiez les points suivants :

- vous êtes connecté avec le bon compte ou profil ;
- le bucket et le préfixe de destination sont corrects ;
- vous avez les droits de lecture ou d’écriture nécessaires ;
- le fichier source existe et vous pouvez y accéder ;
- le nom du fichier et son emplacement respectent les consignes de votre équipe ;
- la taille et le type de fichier sont acceptés par le service.

N’inventez pas le nom du bucket, l’endpoint ou la région : utilisez les paramètres reçus de votre administrateur.

## Envoyer un fichier

1. Dans votre outil, sélectionnez l’action d’envoi ou de téléversement.
2. Choisissez le fichier local à envoyer.
3. Indiquez le bucket et, si nécessaire, le préfixe de destination.
4. Vérifiez le chemin complet avant de confirmer l’envoi.
5. Attendez la fin de l’opération et examinez le résultat affiché par l’outil.
6. Parcourez le bucket ou recherchez la clé pour vérifier que l’objet est présent.

Exemple de destination à adapter aux noms que votre équipe vous a communiqués :

```text
s3://<nom-du-bucket>/<préfixe>/<nom-du-fichier>
```

Les chevrons désignent des valeurs à remplacer ; ils ne font pas partie de la destination. Ne mettez jamais un secret dans cette adresse.

## Récupérer un fichier

1. Vérifiez le bucket et la clé de l’objet à récupérer.
2. Choisissez l’action de téléchargement de votre outil.
3. Sélectionnez un emplacement local où vous avez les droits d’écriture.
4. Vérifiez que le téléchargement s’est terminé sans erreur et que le fichier est lisible.
5. Comparez le nom et, si votre outil le permet, la taille ou la somme de contrôle avec les informations fournies.

## Transfert interrompu ou en échec

- **Accès refusé** : vérifiez le compte sélectionné et demandez la confirmation des droits sur le bucket et l’objet.
- **Bucket ou objet introuvable** : vérifiez l’orthographe, le bucket, la clé et le préfixe ; les noms peuvent être sensibles à la casse.
- **Connexion impossible** : vérifiez l’endpoint, le réseau et la disponibilité du service auprès de l’administrateur.
- **Transfert interrompu** : vérifiez la connexion et l’espace disque, puis suivez les consignes de reprise propres à votre outil. Évitez de relancer une opération destructive sans vérifier son effet.
- **Fichier trop volumineux ou refusé** : demandez les limites applicables à votre service.

Si le transfert semble réussi mais que l’objet n’apparaît pas, vérifiez le compte utilisé, le bucket et la clé complète. N’envoyez pas vos identifiants avec une demande d’assistance.

## Sécurité et bonnes pratiques

- Vérifiez la destination avant d’envoyer des données sensibles.
- N’envoyez que les données nécessaires et respectez les règles de votre organisation.
- Ne placez pas de clés secrètes ou de mots de passe dans le nom des fichiers ou leurs métadonnées.
- Vérifiez les permissions de partage avant de communiquer l’emplacement d’un objet.
- Pour les fichiers importants, confirmez le transfert avec une vérification de présence ou d’intégrité adaptée à votre outil.

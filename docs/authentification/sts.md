# Accès temporaires avec STS

STS (*Security Token Service*) est le nom couramment donné à un service qui émet des **identifiants temporaires**. Lorsqu’un fournisseur compatible S3 propose une API STS, un utilisateur ou une application peut obtenir des identifiants de courte durée au lieu d’utiliser une clé permanente.

> Tous les services compatibles S3 ne proposent pas STS, ni les mêmes opérations STS. Vérifiez le nom de l’endpoint, les mécanismes d’authentification, la durée maximale, les paramètres et la compatibilité de votre fournisseur.

## Pourquoi utiliser des identifiants temporaires ?

Une paire de clés permanente reste valide jusqu’à sa rotation ou sa révocation. Des identifiants temporaires expirent automatiquement et sont donc préférables pour les sessions, les tâches ponctuelles, certains pipelines ou les accès délégués.

Ils se composent généralement de :

- un identifiant de clé d’accès temporaire ;
- un secret associé ;
- un **token de session** ;
- une heure d’expiration.

Les trois valeurs d’identification sont souvent nécessaires pour signer les requêtes. Oublier le token de session entraîne fréquemment des erreurs d’authentification. Ne confondez pas ce token avec le secret d’accès.

## Délégation par rôle : le principe

Dans un modèle fréquent, une identité déjà authentifiée demande à **assumer un rôle**. Le service vérifie que cette identité est autorisée à le faire, puis retourne des identifiants limités dans le temps. Les permissions de la session découlent du rôle et, selon l’implémentation, peuvent être réduites par une policy de session ou d’autres restrictions.

Le parcours est généralement le suivant :

1. L’utilisateur ou l’application s’authentifie avec une identité de départ autorisée à demander une session.
2. Le client contacte l’endpoint STS configuré par le fournisseur.
3. Il demande une session, souvent en nommant un rôle et une durée.
4. STS retourne des identifiants temporaires, un token de session et une expiration.
5. Le client signe les requêtes S3 avec ces valeurs avant leur expiration.
6. À l’expiration, le client renouvelle la session si le mécanisme le permet ; sinon, il faut en demander une nouvelle.

Le rôle définit les permissions disponibles pour la session. Les identifiants temporaires ne donnent pas automatiquement un accès à toutes les ressources.

## Ce qu’il faut obtenir de l’administrateur

Avant de configurer STS, demandez les informations suivantes :

- si STS est activé et quelle opération ou méthode d’obtention de session utiliser ;
- l’endpoint STS et l’endpoint S3 ;
- le protocole ou la version d’API attendue par le client ;
- le rôle à assumer et la durée de session autorisée ;
- les identifiants de départ et les droits nécessaires pour assumer le rôle ;
- la région ou les paramètres de signature obligatoires ;
- les limites propres au service, notamment le renouvellement et les restrictions de réseau.

Ces détails ne sont pas uniformes entre fournisseurs. Un endpoint STS AWS ne doit pas être supposé fonctionner avec un service tiers.

## Utiliser la session avec un outil compatible S3

Configurez le client pour utiliser les trois valeurs temporaires, l’endpoint S3, et toute région ou option de signature requise. La méthode de configuration varie selon le client : variables d’environnement, fichier de credentials, profil, SDK ou intégration au fournisseur.

Exemple de variables **indicatives** — les noms réellement pris en charge dépendent de l’outil :

```sh
export AWS_ACCESS_KEY_ID="<identifiant-temporaire>"
export AWS_SECRET_ACCESS_KEY="<secret-temporaire>"
export AWS_SESSION_TOKEN="<token-de-session>"
```

Un client peut également exiger une région ou un endpoint explicite. N’inscrivez pas de vraies valeurs dans un script conservé ou un dépôt. Les variables ci-dessus illustrent une convention répandue dans les outils AWS ; un service tiers ou un client peut utiliser une autre interface.

Pour un test, utilisez une commande de lecture non destructive prise en charge par votre outil, par exemple le listage d’un bucket. Ajoutez l’endpoint documenté par votre fournisseur et vérifiez que le résultat correspond aux permissions du rôle. Les commandes exactes dépendent du client et ne sont pas identiques pour tous les services S3.

## Renouvellement et expiration

Une session cesse de fonctionner à son expiration. Les clients disposant d’une intégration STS peuvent renouveler automatiquement les identifiants ; d’autres nécessitent une nouvelle demande de session ou une nouvelle connexion.

- Configurez le renouvellement avant l’expiration si l’outil le supporte.
- Vérifiez que le processus peut encore s’authentifier pour obtenir une nouvelle session.
- Ne prolongez pas inutilement la durée : utilisez la durée minimale compatible avec la tâche.
- Supprimez les identifiants expirés des fichiers temporaires et évitez de les afficher dans les journaux.

## Sécurité

- Traitez le token de session, le secret et l’identifiant comme des secrets pendant leur période de validité.
- Ne les partagez pas dans une URL, un ticket, un dépôt ou une sortie de débogage.
- Ne confondez pas une session temporaire avec un lien présigné : ce dernier autorise une requête définie et se transmet différemment.
- En cas de fuite, révoquez la session ou le rôle si le fournisseur le permet, et révoquez également les identifiants de départ compromis.
- Préférez une identité de départ à privilèges restreints, autorisée à assumer uniquement le rôle nécessaire.

## Dépannage

| Symptôme | Vérifications |
|---|---|
| Authentification refusée immédiatement | Présence des trois valeurs, token complet, endpoint STS/S3, région et méthode de signature. |
| Session expirée | Heure d’expiration et mécanisme de renouvellement ; obtenez une nouvelle session. |
| STS refuse la demande | Droit de l’identité de départ à demander la session, nom/ARN du rôle, durée et paramètres acceptés. |
| Session active mais `AccessDenied` | Permissions du rôle, restrictions de session, policy du bucket et ressource/chemin visé. |
| Le client ne sait pas utiliser le token | Vérifiez la documentation du client : certains outils exigent un profil ou une configuration explicite pour les sessions temporaires. |

Les réponses d’erreur et les détails d’implémentation varient selon le service. Pour une investigation, fournissez le code d’erreur et la commande expurgée, jamais les identifiants ni le token.
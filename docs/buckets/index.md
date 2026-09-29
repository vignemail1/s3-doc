# Buckets

Un **bucket** est un espace de stockage qui contient des objets. Il est généralement créé et configuré par l’administrateur du service. Selon vos droits, vous pouvez consulter un bucket, y déposer des fichiers ou les récupérer.

## Bucket, objet et « dossier »

Un objet est identifié par une clé à l’intérieur d’un bucket. Par exemple :

```text
Bucket : equipe-projets
Clé    : rapports/2026/bilan.pdf
```

Dans de nombreuses interfaces, les `/` donnent l’impression d’une arborescence de dossiers. Ils font en réalité partie de la clé de l’objet : les « dossiers » peuvent être une représentation de préfixes, et non des répertoires traditionnels.

## Trouver le bon bucket

1. Choisissez le bucket indiqué par votre équipe ou votre administrateur.
2. Vérifiez que vous avez les droits requis pour l’action envisagée.
3. Si plusieurs buckets ont des noms proches, confirmez lequel utiliser avant d’y déposer des données.

Ne créez pas de bucket et ne modifiez pas ses paramètres sans consigne : ces actions peuvent être réservées à l’administration.

## Parcourir les objets

Dans l’outil utilisé, ouvrez le bucket puis parcourez les objets ou recherchez un préfixe. La présentation et les commandes diffèrent selon l’outil. Si vous ne voyez pas un objet, vérifiez le bucket, le préfixe et les droits d’accès ; consultez aussi [Recherche](../recherche/index.md).

Exemple de clés partageant un préfixe :

```text
rapports/2026/bilan.pdf
rapports/2026/budget.csv
rapports/2025/bilan.pdf
```

Le préfixe `rapports/2026/` permet de regrouper les deux premiers objets.

## Avant de supprimer ou déplacer un objet

- Confirmez le bucket et la clé complets : des noms proches sont faciles à confondre.
- Vérifiez que la suppression ou le déplacement est autorisé.
- Assurez-vous que l’objet n’est plus nécessaire et qu’une copie existe si elle est requise.
- Tenez compte des règles de conservation ou de versioning définies par votre service.

La suppression peut être irréversible ou soumise aux règles de conservation de l’organisation. En cas de doute, demandez confirmation à l’administrateur.

## Limites propres au service

La disponibilité de la création de buckets, du versioning, du chiffrement, des règles de conservation et des opérations de suppression dépend de votre service et de vos droits. Référez-vous aux consignes de votre organisation plutôt qu’à une fonctionnalité générale de S3.

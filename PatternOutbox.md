# PatternOutbox 

Le pattern Outbox est une façon de s'assurer que les modifications apportées à une base de données sont finalement consistantes avec les systèmes externes, même en cas de défaillances. L'idée de base est d'écrire les modifications dans la base de données et dans la Outbox (c'est-à-dire une table spéciale dans la base de données) dans une seule transaction, puis d'avoir un worker indépendant qui lit la outbox et publie les modifications aux systèmes externes.

Avec Kafka, vous pouvez mettre en place le pattern Outbox de la façon suivante :

- Lorsqu'une modification est apportée à la base de données, insérez un nouvel enregistrement dans la table de la outbox avec les détails de la modification.

- Un background worker écrit en C# (ou autre langage) peut lire la table de la outbox et envoyer les modifications aux systèmes externes en produisant des messages sur un sujet Kafka.

- Les systèmes externes peuvent ensuite consommer les messages depuis le sujet, appliquer les modifications et mettre à jour leur propre état.

- Une fois qu'un message est consommé par le système externe, vous pouvez marquer ce message comme consommé dans la outbox de sorte qu'il ne soit plus produit et archivé.

![outbox-pattern](outbox-diagram.jpg)

L'utilisation de Kafka pour la communication entre la outbox et les systèmes externes présente l'avantage d'être hautement évolutif, tolérant aux pannes et de fournir de bonnes performances.

## Quand l'utiliser ?

Lorsque vous avez une application qui modifie des données dans une base de données, et que ces modifications doivent être reflétées dans un ou plusieurs systèmes externes.

Par exemple, imaginez une application de commerce électronique qui modifie les informations de stock d'un produit dans sa base de données. Les modifications doivent également être reflétées dans le système de gestion de stock externe pour s'assurer que les informations de stock sont synchronisées.

Avec la pattern Outbox, lorsque l'application modifie les informations de stock d'un produit dans sa base de données, elle insère également un enregistrement dans la outbox contenant les détails de la modification. Ensuite, un processus séparé lit les enregistrements de la outbox et envoie les modifications aux systèmes externes via Kafka. Le système de gestion de stock externe peut alors consommer les messages de Kafka et mettre à jour son propre état pour refléter les modifications apportées.

## Combiner le pattern outbox et le pattern d'état pour un traitement différé

Dans certain cas fonctionnel vous pouvez avoir besoin de différer votre traitement. 

Imaginez que votre client à souscrit à un abonnement de fidélité de l'application de commerce. L'application possède une fonctionnalité qui permet aux utilisateurs de soumettre des demandes, telles que des demandes de résiliation, qui doi être appliquée à une date future spécifique.

Le pattern Outbox peut être combiné avec le pattern d'état pour gérer une date d'effet sur une demande de manière efficace.

Avec le pattern d'état, l'application peut stocker l'état de chaque demande dans la base de données, y compris la date à laquelle la demande doit être appliquée.

En utilisant le pattern Outbox, l'application peut différer le traitement des demandes jusqu'à ce que la date d'effet soit atteinte ou dépassé.

Un processus séparé peut alors récupérer les entrées de la boîte de sortie, vérifier si la date d'effet est atteinte et traiter la demande en conséquence.

Cela permet de gérer les demandes avec une date d'effet de manière efficace, en évitant le traitement inutile des demandes avant que la date d'effet ne soit atteinte.



## Conclusion

Il existe de nombreux cas d'utilisation pour la pattern Outbox dans différents contextes.

L'idée est que la pattern Outbox offre une méthode pour gérer la synchronisation des données entre plusieurs systèmes en utilisant une technique de messages asynchrones, qui est fiable et évolutive.
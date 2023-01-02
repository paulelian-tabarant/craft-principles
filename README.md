# Design applicatif

## Notions essentielles

### Dépendance

Utilisation d'un bout de code (classe, fonction) provenant d'un autre fichier

But ultime de toute architecture applicative : isoler le code "métier" dans changements d'infra (code "technique")

### Revue de code

Ego-less programming

Conventions d'équipe pour éviter de débattre de l'utilisation d'un nommage / pattern

Rôles : auteur, modérateur, scribe, reviewer, manager

À l'issue de la revue, c'est l'auteur qui corrige le code

Bonnes dimensions pour une revue : entre 200 et 400 lignes de code

Checklist de revue de code : what should we put inside ? (*TO BE COMPLETED*)

### Dette technique

Concessions faites sur les standards de qualité de l'équipe lors du développement d'une feature, qu'il faudra "rembourser" plus tard car elles détériorent la maintenabilité du code

## Tester son code

### FIRST

- **Fast** : unitaires = ultra-rapides, temps de run ne doit pas être un souci
- **Isolated** : doit être indépendant, ne pas dépendre d'un contexte ou d'autre chose initialisé dans un autre test
- **Repeatable** : doit toujours donner le même résultat avec les mêmes paramètres d'entrée, peu importe l'environnement
- **Self-validating**: pas d'intervention manuelle pour savoir si le comportement est validé ou non
- **Timely**: écrire ces tests avant (TDD) ou en même temps qu'on implem le code de prod a fait ses preuves pour assurer leur qualité
- **Thorough**: cas passants + cas d'erreur (exhaustifs)

### Couverture de tests 

Attention : code coverage est pas forcément une bonne métrique. On peut avoir un coverage de 100% en ne testant qu'un scénario complètement idiot.

ex.
~~~kotlin
fun multiply(a: Int, b: Int) = a * b

assertThat(multiply(1, 1)).isEqualTo(1)
~~~

### Mock/Stub

- Stub override directement une classe qui contient des dépendances dont on veut s'isoler pour le test, en définissant des retours "en dur"
- Mock est une lib qui permet de juste spécifier la valeur retournée par l'objet dans le contexte sans avoir à override / rajouter du code pour spécifier le comportement

Stubs largement abandonnés en faveur des mocks aujourd'hui

### Test-driven development

Méthode de développement de features qui consiste à décrire l'intention dans un test avant de démarrer l'implémentation.

Cycle TDD
- RED : le test doit décrire le comportement souhaité, être lancé et échouer (être rouge dans la plupart des IDE)
- GREEN : on fait passer le test au vert avec l'implémentation la plus simpliste possible (KISS)
- REFACTOR : on refactorise le code de test (nom, méthodes utilitaires), puis l'implémentation si besoin

### Mutation testing 

Tester la pertinence des tests, en créant des "mutants" de notre implem. Selon si les mutants sont rejetés ou non (comportements différents devraient être rejetés si le test en bon), on peut évaluer la qualité d'un test

## Principes de design

### Acronymes

**YAGNI** : "you ain't gonna need it"

**DRY/WET** : "don't repeat yourself" / "write everything twice"

**KISS** : "keep it simple and stupid"

### Step down rule

Dans le corps d'une méthode, tout le code doit être au même niveau d'abstraction. Pareil lorsqu'on descend dans le corps d'une méthode appelée dans la méthode courante. Le code doit pouvoir se lire comme un journal, avec des sections de plus en plus spécifiques selon le niveau de profondeur.

### Extract till you drop

Extraire le code dans des méthodes de plus haute abstraction, jusqu'à estimer qu'extraire davantage nuirait à la compréhension. L'idée est de trouver le "bon niveau d'abstraction", celui le plus adapté à la compréhension des autres développeurs. Bien que ce soit subjectif.

### SOLID

Notice: difficile de tous les respecter en même temps. C'est un idéal.

#### S : Single responsibility principle

Une classe / un module ne doit avoir qu'une seule raison de changer

> "if a class does too many things, you have to change it every time one of these things changes. While doing that, you’re risking breaking other parts of the class which you didn’t even intend to change"

L'intérêt de respecter ce principe est multiple :
- Éviter les effets de bord d'un changement dans le code
- Pouvoir conceptuellement mieux visualiser l'architecture de l'application

Exemple de violation du principe

Une classe qui modélise un livre et qui gère à la fois les stocks (utilisé dans le contexte d'un inventaire) et son affichage (utilisé pour les clients finaux, pour avoir un visu sur les infos du livre)

Une façon de respecter le SRP sur cet exemple serait d'avoir une classe Book, contenant un référence BookInformation (titre, date de publi, ISBN) et un ensemble de champs / méthodes pour gérer son stock

#### O : Open for extension, closed for modification

L'idée derrière l'Open Closed principle est de privilégier l'extension à la modification quand on rajoute une fonctionnalité. De cette manière, on évite de "toucher" au code initial et de casser le comportement de base qu'on veut conserver. On fait en sorte de l'isoler, pour pouvoir rajouter notre feature sereinement. Dans les langages orientés objet, on utilise souvent l'extension de classe via des interfaces pour garder le code initial générique et avoir nos comportements distincts dans les implémentations séparées de cette interface

Exemple de violation

On a envie de rajouter un nouveau mode d'affichage "BD" sur le livre. On crée un paramètre dans la méthode display(asBD: true/false). Pour prendre en compte le nouvel affichage, on va devoir aller modifier le corps de cette méthode et potentiellement entraîner des régressions sur le comportement de la méthode avant modification, qu'on a envie de conserver. Une manière de respecter ce principe serait de rendre la méthode display() abstraite et de surcharger cette méthode dans une classe BandeDessinée.

#### L : Liskov substitution principle

On doit pouvoir manipuler les objets d'une classe indépendamment du fait que l'on manipule, sous le capot, des sous-classes de cette classe
i.e. les sous-classes doivent avoir un comportement compatible avec celui de la classe mère

Exemple de violation du principe

On a une classe Bibliothèque qui manipule des Livre. Livre contient les infos d'un livre (date de publication, titre, ISBN...) et gère les stocks (un entier) via une méthode publique `setStock(int)`. Un jour, on a envie de gérer des livres numériques dans notre application. On pourrait penser qu'un livre numérique est un type particulier de Livre, et donc déclarer une classe LivreNumérique qui hérite de Livre. Le problème, c'est qu'à chaque endroit du code où l'on croit avoir affaire à des Livre, et donc potentiellement gérer leur stock, on pourra aussi avoir affaire à des LivreNumérique qui, eux, ont un stock illimité. Dans le cas de livres numériques, aucun besoin de manipuler des stocks. Ce que répresente LivreNumérique n'est pas totalement compatible avec le comportement de base de Livre.

Pour respecter le principe de Liskov, on peut mutualiser le comportement commun aux livres papier et numérique dans la classe Livre (infos), et ensuite créer une sous-classe LivrePapier qui contient la logique spécifique aux livres papier, i.e. la gestion des stocks dans ce cas.

#### I : Interface segregation principle

Le code appelant ne doit pas dépendre de méthodes qu'il n'utilise pas.

Avoir une interface avec trop de méthodes est un bad smell de design.

**Exemple de violation**



#### D : Dependency inversion principle

Des classes / modules de de haut niveau d'abstraction ne doivent pas dépendre de modules de plus bas niveau. Les deux doivent être dépendants d'une interface qui les relie.

Les abstractions ne sont pas censées être dépendantes de détails d'implémentation. Ce sont les implémentations qui doivent dépendre des abstractions.

**Exemple de violation**

On dispose d'une base de données pour stocker les livres disponibles à la location. On souhaite exposer un endpoint pour accéder à ces livres. Pour cela, on dispose d'une classe `SqlBookRepository` qui vient requêter la base de données mise en place via un ORM. En supposant que la requête `GET /books` sollicite un `BookService`, avec une méthode `findAllBooks()`, on crée une dépendance forte entre des règles applicatives (`BookService`) et l'infrastructure externe de persistance (`SqlBookRepository`).

Le jour où l'on décide non plus de changer de moyen de persistance ou simplement de modifier la classe `SqlBookRepository`, on risque de devoir adapter le code appelant (`BookService` en l'occurrence) en fonction alors que son rôle est d'encapsuler des règles applicatives.

Pour casser la dépendance au `SqlBookRepository`, on peut créer une interface`BookRepository` avec une méthode `findAll` sur laquelle le `SqlBookRepository` viendra se brancher. De cette manière, nos deux classes `SqlBookRepository` et `BookService` dépendent maintenant de cette interface.

### Loi de Demeter

Le code appelant n'est pas censé être au courant des dépendances de ses dépendances.

“when one wants a dog to walk, one does not command the dog's legs to walk directly; instead one commands the dog which then commands its own legs.”

Violation :
~~~java
object.getA().getB().getC().doSomething();
~~~

### Object calisthenics

> https://williamdurand.fr/2013/06/03/object-calisthenics/

Listing de principes par Jeff Bay, censé garantir les principes faisant du code orienté objet maintenable et lisible (encapsulation, polymorphisme, nommage) par des règles pratiques "bas niveau".

1. Un seul niveau d'indentation par méthode
2. Ne jamais utiliser de `else`
3. Encapsuler toutes les types primitifs
4. "First-class collections" : Toutes les classes contenant des collections ne doivent pas avoir d'autre attribut : ces classes doivent être dédiées à la manipulation de la collection en question
5. "One dot per line" : ne pas chaîner des appels de méthodes
6. "Don't abbreviate" : commencer à checher des abbréviations est le signe d'un mauvais design quelque part (duplication ? trop de responsabilités ?)
7. "Keep all entities small" : pas de classe de plus de 50 lignes (un peu extrême ?)
8. Pas de classe avec plus de 2 attributs d'instance 
9. Pas de getters / setters : utiliser les getters que pour lire et retransmettre l'état d'un objet, mais pas pour prendre des décisions en fonction de cet état. "Tell, dont ask" : c'est à l'objet lui-même de prendre des décisions quant à son état courant, et pas au code appelant. 

## Architectures applicatives

Objectif principal : separation of concerns, en divisant le code par couches (comme les autres types d'architecture), dont une avec les règles métier, isolée des autres

### Clean architecture

Présente plusieurs avantages

- indépendance aux éléments externes (frameworks, UI, DB, services externes)
- règles métier testable unitairement sans tirer aucune autre dépendance que le framework de test

**Dependency Rule** : respecter l'ordre des dépendances. Une couche ne peut dépendre que de couche(s) plus basse(s). Plus on va au centre de l'oignon, plus le niveau d'abstraction augmente

*TO BE COMPLETED* : "what data crosses the boundaries"

#### Couches

- **Entities : "enterprise-wide business rules"** - Utilisable dans plusieurs applications de l'entreprise. "they encapsulate the most general and high-level rules". Raison de changer : changement dans les règles métier
- **Use cases : "application-specific business rules"** - Orchestre le flot de données depuis et vers les entités du domaine, et manipule les entités pour remplir les objectifs du usecase (en utilant les règles métier des entités). Raison de changer : cas d'usage change, dans l'orchestration du flot de données
- **Interface adapters** - Responsable de la conversion entre format d'utilisation des données dans les usecases et format de l'élément externe branché sur l'interface, dans les deux sens. Controllers : infra API rest vers objets du domaine. Objets du domaine vers format d'infra (GTS, DB postgres...). Raison de changer : format des données à renvoyer / envoyer à l'élément externe - Manière d'interagir avec cet élément externe
- **Frameworks and drivers** - Devices, Web, DB, UI, external interfaces

**Thought** : pas sûr de voir où l'"inward" se situe (notre code au niveau du Controller utilise des choses du framework, typiquement
Nos repos de persistance dépendent de la DB, non ?

Différence enterprise <-> application specific business rules :
Un SI (global) + des microservices qui répondent à un besoin métier via un usecase

### Architecture hexagonale

*TO BE COMPLETED*

### Lecture

https://blog.octo.com/egoless-programming/

https://springframework.guru/gang-of-four-design-patterns/

https://www.martinfowler.com/articles/designDead.html

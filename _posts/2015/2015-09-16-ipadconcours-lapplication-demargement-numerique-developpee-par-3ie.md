---
title: "iPadConcours - l'application d'émargement numérique développée par 3IE"
date: "2015-09-16"
categories: 
  - "nos-projets"
---

## Le contexte

Le concours Advance est le concours APB (Admission Post-Bac) des écoles d'ingénieurs du groupe IONIS. Il s'agit donc d'un concours destiné principalement aux lycéens en classe de Terminale et représente un concours d'entrée pour les écoles d'ingénieurs du groupe IONIS, à savoir l'EPITA, l'ESME Sudria et l'IPSA.

Sur l'édition 2015 du concours, plus de 1600 candidats se sont donc retrouvés à la maison des examens d'Arcueil afin de passer ce concours. Afin de s'assurer de la présence de tout le monde, chaque candidat doit émarger pour chaque épreuve.

## Le besoin

Lors de la première édition du concours, cet émargement s'est fait de manière classique, via des feuilles d'émargement, chaque candidat signant dans la case correspondant à son nom. Il est également important de préciser que le jour du concours, les candidats tirent au sort leur place et leur salles et il est donc impossible de savoir avant le jour J où va se positionner chaque étudiant.

Avec des feuilles d'émargement pour 1600 candidats, on comprend facilement que l'opération aie pris énormément de temps. C'est pourquoi pour la seconde édition du concours, il a été décidé de moderniser la technique d'émargement utilisée.

## Le scénario mis en place

Ainsi, il a été décidé d'ajouter un QRCode sur la convocation de l'étudiant. Le jour du concours, un iPad scanne le QRCode du candidat afin de l'identifier. Le surveillant peut alors vérifier si les informations sont correctes. Si tel est le cas, le candidat peut alors signer directement sur l'iPad. Dans le cas d'un QRCode qui ne serait pas reconnu, le surveillant a toujours la possibilité de rechercher le candidat via son nom de famille.

\[caption id="attachment\_881" align="aligncenter" width="827"\][![IMG_0083 - copie](/assets/images/IMG_0083-copie.jpg)](/assets/images/IMG_0083-copie.jpg) Interface en mode "reconnaissance de QRCode"\[/caption\]

 

\[caption id="attachment\_882" align="aligncenter" width="738"\][![IMG_0081 - copie](/assets/images/IMG_0081-copie.png)](/assets/images/IMG_0081-copie.png) Interface d'émargement\[/caption\]

A la fin de l'émargement, les iPads sont regroupés dans une salle dans laquelle se trouve un serveur et envoient alors leurs informations de signature sur ce serveur grâce à un réseau local. Le serveur est alors capable de calculer des statistiques sur le taux de présence au concours.

Mais le point le plus intéressant de ce serveur est qu'il est capable d'imprimer les feuilles d'émargement avec les signatures des étudiants. Ces feuilles sont alors des feuilles d'émargement par demi-salle d'examen, soit 100 candidats.

Les candidats revenant l'après-midi pour la suite des épreuves (dans la même salle) se voient alors présenter une feuille d'émargement comportant 100 noms, ce qui est bien moins important que les 1600 de la première édition du concours.

 

\[caption id="attachment\_892" align="aligncenter" width="1279"\][![Scenario iPadConcours](/assets/images/Scenario-iPadConcours1.png)](/assets/images/Scenario-iPadConcours1.png) Scénario mis en place grâce à iPadConcours\[/caption\]

Ainsi, cette application permet depuis la seconde édition du concours de gagner énormément de temps à l'émargement le matin, mais aussi l'après-midi. Elle permet de plus d'éviter un certain nombre d'erreurs de saisie si un candidat se trompe de case etc, ce qui peut s'avérer réellement problématique sur un concours comme le concours Advance puisqu'une erreur invaliderait le concours.

## Et derrière, comment ça fonctionne ?

L'application iOS a été réécrite pour [iOS 9](http://www.apple.com/fr/ios/whats-new/) en [Swift 2](https://developer.apple.com/swift/blog/?id=29) pour la prochaine édition du concours. Lors du concours, les iPads n'ont pas d'accès réseau. Ainsi, une synchronisation est faite avant le concours pour que l'iPad sauvegarde toutes les données dans une base de données locale utilisant [CoreData](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html) (l'ORM d'Apple permettant d'avoir une base de données locale). Chaque fois qu'un étudiant signe, l'image est sauvegardée en local sur l'iPad et une entité signature est créée dans la base de données afin de sauvegarder un certain nombre d'informations.

Une fois le concours terminé et les vérifications effectuées, les données sont effacées.

De son côté, le serveur fournissant les APIs est écrit en [Node.js](https://nodejs.org) tout en utilisant le framework express.js dans le but de nous simplifier l'écriture de webservices. Il fonctionne pour sa part avec une base de données [PostgreSQL](http://www.postgresql.org) et est relié à celle-ci via l'ORM [Bookshelf.js](http://bookshelfjs.org). Cela permet d'effectuer un certain métier grâce à cet ORM en manipulant directement des objets. Toutefois, les ORMs n'étant la plupart du temps pas optimisés pour réaliser des actions complexes, celles-ci sont déplacées vers la base de données au travers de procédures stockées. Ainsi, la base de données peut exécuter plus vite son travail tout en déchargeant le serveur Node.js.

Enfin, le back-office utilise quant à lui la technologie [Angular.js](https://angularjs.org), ce qui permet de faire le rendu des pages web sur le client et non sur le serveur, ainsi , à l'instar des procédures stockées, le serveur est déchargé d'une partie du travail qui est à effectuer.

En combinant les efforts pour décaler une partie du travail vers la base de données et une autre partie du travail vers le client (rendu des pages), nous obtenons alors un serveur d'API très optimisé et performant écrit dans un langage lui même déjà performant.

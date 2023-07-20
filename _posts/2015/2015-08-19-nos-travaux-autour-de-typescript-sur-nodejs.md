---
title: "Nos travaux autour de typescript sur NodeJs"
date: "2015-08-19"
categories: 
  - "technical"
tags: 
  - "angularjs-2-0"
  - "express"
  - "framework-express"
  - "framework-mvc"
  - "github"
  - "microsoft"
  - "nodejs"
  - "npm"
  - "orm"
  - "raspberry-2"
  - "sails"
  - "typefx"
  - "typescript-1-5"
  - "visual-studio-2015"
  - "windows-10-iot-core"
---

Récemment Microsoft a publié sa dernière mouture de visual studio 2015, cette sortie fut également l'occasion pour l'éditeur d'annoncer [typescript 1.5](http://www.typescriptlang.org/) . Pour rappel, Typescript est une surcouche au javascript permettant d'apporter entre autre le support du typage au JS ainsi que la notion d'orienté objet.

3IE développe certains de ses projets en NodeJS avec le framework express. Bien qu'ayant une structuration de projet, il n'y a pas de type checking ce qui ne facilite pas la lecture de code. Pour pallier à ce problème et gagner en rapidité sur le développement de nos projets, nous nous sommes intéressés à recréer une surcouche typescript sur Express. Afin de gagner du temps nous sommes partis d'un projet existant réalisé sous typescript 1.0.

Nous venons donc de publier une première version de ce framework (TypeFx) que vous pouvez retrouver sur [GitHub](https://github.com/3IE/typeframework) et installable à travers le gestionnaire de package [NPM](https://www.npmjs.com/package/typefx).

##### Pourquoi s'intéresser à typescript ?

Phénomène de mode ou bien cette surcouche répond-elle à un réel besoin ? Dans tous les cas l'intérêt pour cette technologie ne fait que croître ([google trends](https://www.google.fr/trends/explore#q=typescript)). Le typescript devrait permettre de structurer beaucoup plus les projets autour du Javascript. La structuration du javascript à déjà fait un bond en avant avec l'apparition des différents frameworks comme Durendal, AngularJS ou encore backboneJS, ... Depuis la création de typescript certains de ces frameworks ont commencé à développer une version s'appuyant sur Typescript. C'est le cas de la version d'[AngularJS 2.0](http://angular.io) ou de Durandal connu sous le nom d'[Aurelia](http://aurelia.io/). Ces frameworks sont essentiellement là pour réaliser la partie UI des applications. Si l'on souhaite garder une cohérence des langages, il faudrait également développer la partie serveur en Javascript ce qui éviterait la démultiplication des compétences au sein d'une équipe. L'utilisation de [Node.js](https://nodejs.org/)  ou de son fork [io.js](https://iojs.org/en/index.html) s'impose donc.

##### NodeJS ?

NodeJS est une plateforme construite autour du moteur javascript de chrome permettant de construire des applications légères avec une forte capacité à monter en charge. De grands noms l'utilisent déjà au sein de leur SI comme Paypal (sujet abordé sur le [blog de Paypal](http://www.paypal-engineering.com/2013/11/22/node-js-at-paypal/)). De plus, de par sa légèreté NodeJS devient également un choix judicieux dans le développement d'application IOT. Microsoft a également annoncé une intégration de NodeJs sur sa plateforme [windows 10 iot core](http://blogs.windows.com/buildingapps/2015/05/12/bringing-node-js-to-windows-10-iot-core/)  fonctionnant sur Raspberry 2. De nombreux autres projets existent autour de l'IOT et de NodeJs : - [nodered](http://nodered.org/) - [cyclonJs](http://cylonjs.com/)

Node.js, s'appuyant sur Javascript, offre beaucoup de flexibilité, mais sans rigueur un projet peut vite devenir difficile à maintenir. Bien que certains projets aident à la structuration comme par exemple [Express](http://expressjs.com/) ou [Sails](http://sailsjs.org/), aucun n'apporte le support du typage ou de l'orienté objet.

##### TypeFx ?

L'utilisation de typescript permet d'apporter ce support. Bien entendu nous pouvons utiliser typescript juste comme une surcouche à Node.js, mais le fait ici de le combiner au-dessus d'un framework comme express permet de gagner du temps sur le développement. L'utilisation de TypeFx est très proche du framework MVC de Microsoft et enlève toute la partie initialisation d'express, tout en s'appuyant sur l'ensemble de ses concepts. Ainsi nous retrouvons bien le router, le moteur de vue EJS ...

Notre travail sur ce framework a consisté dans un premier temps à le remettre à jour avec les dernières versions des outils de compilation mais aussi avec la dernière version de typescript. Afin d'illustrer notre travail nous avons publié un [projet d'exemple](https://github.com/3IE/typefx-sample) sur GitHub.

##### Le futur ?

Notre contribution ne s'arrêtera pas à cette remise à jour. D'autres nouveautés sont à venir, notamment l'ajout d'annotations permisses par la version 1.5 de typescript, une modularité au niveau de l'ORM utilisé.

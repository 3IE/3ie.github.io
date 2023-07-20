---
title: "Un projet AngularJS 1.4 avec Typescript"
date: "2015-10-30"
categories: 
  - "technical"
tags: 
  - "angularjs"
  - "grunt"
  - "js"
  - "typescript"
  - "vscode"
---

Faisant suite à notre [précédent article](https://blog.3ie.fr/nos-travaux-autour-de-typescript-sur-nodejs/) sur Typescript, nous allons maintenant créer un projet AngularJS  avec TypeScript (en attendant la version 2.0 d'AngularJS qui est en cours de développement)

#### Quels sont les avantages à utiliser cette surcouche ?

Bien qu'angular soit déjà un framework permettant de structurer son architecture projet, il devient vite compliqué de maintenir un code dans le temps sans une certaine rigueur. Prenons un exemple, nous pouvons ajouter des propriétés sur le $scope dans une fonction du controller sans les déclarer ailleurs. Cependant, lors d'une refactorisation, nous pouvons passer à coté de cette déclaration à l'intérieur d'une fonction et causer une régression. C'est ici que nous pouvons voir l'un des avantages de Typescript en utilisant les interfaces.

Cet article se décomposera en plusieurs parties :

- Installation de l'environnement de développement
- Initialisation du projet
- Construction du gruntfile

#### Installation de l'environnement de développement

#### Initialisation du projet

Afin d'avoir une base pour commencer notre projet voici la structure de la solution :

├───app (Répertoire contenant l'application)
│ ├───controllers (Répertoire contenant les controllers angularjs)
│ ├───css (Répertoire de destination après compilation des fichiers less)
│ ├───directives (Répertoire contenant les directives angularjs)
│ ├───img (images de l'application)
│ ├───less (Répertoire contenant les fichiers less)
│ └───views (Répertoire contenant les fichiers html)
│		└───directive\_templates (Répertoire contenant les fichiers html des directives)
├───app\_engine (Répertoire permettant d'héberger la couche de logique métier et d'accès aux données)
│      └───common
│ 		 ├───business (Répertoire contenant la couche de logique métier)
│ 		 └───data (Répertoire contenant la couche d'accès aux données, par exemple les requêtes vers les API)
├───bower\_components
├───models (Répertoire contenant les fichiers des objets)

 

Pour initaliser la solution nous allons télécharger la bibliothèque angularJS

c:\\myproject> bower install angular angular-bootstrap angular-ui-router angular-ui-validate bootstrap less --save

L'option --save permet d'enregistrer les dépendances dans le fichier bower.json

##### tsd

La première étape de configuration d'un projet Angular en Typescript est de récupérer les fichiers de définition, ayant pour extension \*.d.ts. Dès que vous souhaitez utiliser une bibliothèque externe n'étant pas écrite en Typescript, il vous faudra récupérer ces fichiers afin de permettre l'autocomplétion. Ces fichiers sont disponibles grâce au projet [tsd manager](http://definitelytyped.org/tsd/). Pour l'utiliser il faut l'installer grâce à NPM.

C:\\myproject> npm install tsd -g

Une fois tsd installé, il faut l'initialiser. Pour cela, tapez la commande suivante :

C:\\myproject> tsd init

-> written tsd.json
-> written typings\\tsd.d.ts

Vous venez d'ajouter un fichier tsd.json qui permet de configurer l'ensemble des fichiers de définitions que vous allez installer pour votre projet. C'est grâce à  ce fichier que les autres développeurs récupéreront exactement les mêmes versions que vous. Le fichier tsd.d.ts permet quand à lui de recenser tous les fichiers de définitions que vous avez configuré. C'est ce fichier que vous référencerai afin d'obtenir l'auto complétion sur vos bibliothèques externes.

De base nous installons 2 fichiers de définition :

- angular
- angular-ui-router

N'oubliez pas de rajouter l'option --save pour que les fichiers \*.d.ts soit inscrit dans le fichier typings\\tsd.d.ts

C:\\myproject> tsd install angular angular-ui-router --save

 - angular-ui-router / angular-ui-router
   -> angularjs      > angular
   -> jquery         > jquery
 - angularjs         / angular
   -> jquery         > jquery

>> running install..

>> written 3 files:

    - angular-ui-router/angular-ui-router.d.ts
    - angularjs/angular.d.ts
    - jquery/jquery.d.ts

##### Grunt

Afin de pouvoir compiler nos fichiers typescript nous devons également installer quelques dépendances npm :

- grunt-ts
- load-grunt-tasks
- typescript

Afin que ces dépendances soient sauvegardées sur le projet (package.json), n'oubliez pas de rajouter --save sur la commande

C:\\myproject> npm install grunt grunt-develop grunt-ts load-grunt-tasks typescript grunt-http-server --save

Nous allons maintenant modifier le Gruntfile pour réaliser la compilation des fichiers TS. A cette étape le Gruntfile ne servira que pour la compilation. Nous verrons lors d'un prochain article comment l'enrichir pour générer les fichiers CSS, faire les tests et effectuer un déploiement du site.

module.exports = function (grunt) {

  require('load-grunt-tasks')(grunt);
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),

    ts: {
      default: {
        src: \["\*\*/\*.ts", "!node\_modules/\*\*/\*.ts", "!bower\_components/\*\*/\*.ts", "app\_engine/\*\*/\*.ts"\],
        options: {
          target: 'es5',
          module: 'commonjs',
          noEmitOnError: true,
          fast: 'never'
        }
      }
    }, 
    clean: { 
      // remove old javascript files 
      public: \["app/\*.js", "app/\*\*/\*.js", "app\_engine/\*\*/\*.js"\]
    },
    'http-server':{
      'dev':{
        port:5000,
        openBrowser :true
      }
    }
  }); 

  grunt.loadNpmTasks('grunt-develop')
  grunt.loadNpmTasks('grunt-contrib-clean');
  grunt.loadNpmTasks("grunt-ts");

  grunt.loadNpmTasks('grunt-http-server');

  // Default task(s).
  grunt.registerTask('web', \['http-server:dev'\]);
  grunt.registerTask('default', \['clean','ts'\]);
  
};

Ici, la tache par défaut du Gruntfile permet de nettoyer le répertoire des fichiers JS avant de compiler les fichiers TS.

Ensuite, ouvrez ce répertoire avec un éditeur de code (par exemple [VSCode](https://code.visualstudio.com/), IDE disponible sur plusieurs plateformes).

La première étape est de configurer son environnement pour ne voir uniquement que les fichiers TS. En effet nous n'aurons plus besoin de manipuler les fichiers JS puisqu'ils vont être générés automatiquement. Pour faire cela dans VsCode : File > Preferences > Workspace Settings

Rajoutez les extension js et js.map dans la liste des extensions à exclure. Afin d'être restrictif nous allons appliquer cette modification uniquement sur les répertoires app et app\_engine.

\[caption id="attachment\_922" align="aligncenter" width="676"\][![settingsVsCode](/assets/images/settingsVsCode-1024x404.png)](/assets/images/settingsVsCode.png) Visual Studio Code settings\[/caption\]

Maintenant que l'environnement est prêt nous allons pouvoir entamer le développement du projet.

#### Développement

Avant toute chose nous devons ajouter un fichier reference.ts à la racine du site. Ce fichier nous permettra de referencer l'ensemble des fichiers TS pour faciliter l'auto-complétion (il ne devra donc contenir aucun code). Rajoutez la ligne suivante (qui permet de référencer les fichiers de définition que nous avons intégré avec tsd manager):

/// <reference path="./typings/tsd.d.ts"/>

La 1ere étape du développement consiste à déclarer notre application Angular. Pour cela créez un fichier app.startup.ts pour initialiser les différentes routes et configurer l'application (Vous pouvez vous inspirer du fichier présent dans le [starter](https://github.com/3IE/TypescriptAngularStarter/blob/fd622603c3bf224b8bedc26a7cfa84c204eeb0ce/app/app.startup.ts)).

##### **Création d'un controller**

Pour créer un controller, il y a deux étapes. la première est de recenser les variables que l'on veut exposer à la vue pour les ajouter à une interface. Ce travail permet de typer l'utilisation du $scope et d'éviter les déclarations de variables n'importe où dans le code. Pour cela on étend ng.IScope et on définit nos variables. La déclaration du namespace est laissée à votre discrétion bien entendu.

namespace app {
	'use strict';

	interface INavBarControllerScope extends ng.IScope{
		isCollapsed:boolean;
		pseudo:string;
	}
}

La seconde étape est de la déclarer la classe du controller dans laquelle on va injecter le $scope typé :

namespace app{
	'use strict';
	
	interface INavBarControllerScope extends ng.IScope{
		isCollapsed:boolean;
		pseudo:string;
	}
	
	class NavBarController{
		
		constructor(private $scope:INavBarControllerScope) {
			this.$scope.isCollapsed = true;
			this.$scope.pseudo = "BeHappy";
		}
	}
}

L'enregistrement du controller ce fait comme d'habitude grâce à angular.module("...").controller("...")

namespace app{
	'use strict';
	
	interface INavBarControllerScope extends ng.IScope{
		isCollapsed:boolean;
		pseudo:string;
	}
	
	class NavBarController{
		
		constructor(private $scope:INavBarControllerScope) {
			this.$scope.isCollapsed = true;
			this.$scope.pseudo = "BeHappy";
		}
	}
	
	angular.module('starterKit').controller("NavBarController", \["$scope", NavBarController\]);
}

##### Ajout d'une classe model

Afin d'avoir un projet typé, les classes Model doivent elles aussi être déclarées. Pour cela, créez un répertoire Models à la racine du projet, puis ajoutez un fichier ts pour référencer votre nouvelle classe :

namespace app.models{
	'use strict';
	
	export class School{
		name:string;
		constructor(name:string) {
			this.name = name;
		}
	}
}

N'oubliez pas d'ajouter la référence vers ce fichier ts dans le fichier reference.ts. De cette manière la déclaration de cette classe pourra être utilisée dans tout le reste du projet. Il faut également ajouter la référence vers le fichier js qui sera généré dans l'index.html.

/// <reference path="./typings/tsd.d.ts"/>
/// <reference path="./models/school.ts"/>

utilisation de la classe school dans un controller :

/// <reference path="../../reference.ts"/>

namespace app {
	'use strict';

	interface IProfileControllerScope extends ng.IScope {
		schools: models.School\[\];
	}

	class ProfileController {

		constructor(private $scope: IProfileControllerScope) {
			this.$scope.schools = \[new models.School("school 1"), new models.School("school 2")\];
		}
	}

	angular.module('starterKit').controller("ProfileController", \["$scope", ProfileController\]);
}

##### Déclaration d'un service

Dans notre cas nous nous servons des services pour séparer les couches au niveau de notre app\_engine (par exemple pour faire les appels vers les API). Afin de garder une réactivité sur l'appel de nos services, nous utilisons également les _promises_.

Dans cette classe d'accès aux données nous allons utiliser ng.IHttpService pour accéder aux API et ng.IQService pour les _promises_.

/// <reference path="../../../reference.ts"/>

namespace engine.common.data {
	'use strict';

	export class User {
		/\*\*
		 \*
		 \*/
		constructor(private $http: ng.IHttpService, private $q: ng.IQService) {

		}

		getUsers(): ng.IPromise<app.models.User\[\]> {
			var deferred = this.$q.defer();

			this.$http({
				url: './users.json',
				method: 'GET'
			}).success(result=> {
				deferred.resolve(result);
			}).error(e=> {
				deferred.reject(e);
			});

			return deferred.promise;
		}
	}

	angular.module('common.data').service("data.user", \["$http", "$q", User\]);
}

Lorsque vous rajoutez des services ou tout fichier TS devant être utilisé dans le reste de la solution n'oubliez pas de rajouter la référence dans le fichier reference.ts à la racine du projet. Sinon vous n'obtiendrez pas l'auto-complétion.

##### Cas de la directive

Pour réaliser une directive il faut implémenter ng.IDirective. La particularité de la directive est le fait de définir une méthode (Factory) permettant de retourner une instance de la directive. Cela est du à Angular qui demande toujours une fonction en paramètre lors la déclaration d'une directive. Si vous devez récupérer des paramètres de la directive vous pouvez enrichir ng.IAttributes, par exemple pour exécuter une fonction du _controller_.

/// <reference path="../../reference.ts"/>

module app.directive {
	"use strict";

	interface INgEnterDirectiveScope extends ng.IScope { }

	interface INgEnterDirectiveAttribute extends ng.IAttributes {
		ngEnter: () => void;
	}

	export class NgEnter implements ng.IDirective {

		constructor() {	}

		link = (scope: INgEnterDirectiveScope, element: ng.IAugmentedJQuery, attrs: INgEnterDirectiveAttribute) => {
			element.bind("keydown keypress", function(event) {
				if (event.which === 13) {
					scope.$apply(function() {
						scope.$eval(attrs.ngEnter);
					});

					event.preventDefault();
				}
			});
		}

		public static Factory() {
			var directive = () => {
				return new NgEnter();
			}
			return directive;
		}
	}

	angular.module('starterKit').directive("ngEnter", \[NgEnter.Factory()\]);
}

##### Dernière étape la compilation

Etant donné que nous avons créé un gruntfile au début du projet nous allons pouvoir automatiser cette partie. Si vous êtes sous VS Code vous pouvez lancer une tache grunt avec la commande : crl + maj + p Sélectionnez launch task, puis le nom de la tache à lancer (dans notre cas Default).

\[caption id="attachment\_924" align="aligncenter" width="676"\][![Lancement d'une task](/assets/images/taskRunner-1024x395.png)](/assets/images/taskRunner.png) Lancement d'une task\[/caption\]

Si on souhaite gagner un peu de temps et lancer directement une tache de compilation nous pouvons configurer le task runner. Si le fichier tasks.json n'est pas encore créé vous pouvez faire une première fois ctrl + maj + b et VS Code va vous proposer de rajouter ce fichier.

\[caption id="attachment\_925" align="aligncenter" width="676"\][![Configuration de la tache de build](/assets/images/configTaskRunner-1024x196.png)](/assets/images/configTaskRunner.png) Configuration de la tache de build\[/caption\]

Il faut ensuite compléter le fichier avec le nom de la tache que l'on souhaite exécuter (dans notre cas default).

{
    "version": "0.1.0",
    "command": "grunt",
    "isShellCommand": true,
    "tasks": \[{
        "isBuildCommand": true,
        "taskName": "default"
    }\]
}

Maintenant lorsque l'on refait ctrl + maj + b nous pouvons lancer directement notre tache de compilation.

L'interet d'utiliser ce type de technologie c'est que nous ne sommes pas lié à un IDE. Ainsi si nous ouvrons un invite de commande nous pouvons également executer notre tache grunt.

C:\\myproject>grunt default

Running "ts:default" (ts) task
Compiling...
Using tsc v1.6.2

TypeScript compilation complete: 2.27s for 17 TypeScript files.

Done, without errors.

C:\\myproject>

##### Tester le site

Pour tester le site, vous pouvez soit configurez le serveur web pour pointer sur le répertoire, soit vous pouvez lancer la commande grunt web. Cette commande dépend du plugin grunt-http-server

#### Conclusion

Nous avons pu voir ici l'utilisation de TypeScript dans un projet AngularJS. Vous pouvez télécharger le projet exemple sur notre [github](https://github.com/3IE/TypescriptAngularStarter).

Dans le prochain article nous verrons comment intégrer toutes les étapes du gruntfile afin de pouvoir [déployer notre site](https://blog.3ie.fr/optimisez-les-releases-de-vos-projets-typescriptangular-avec-grunt/) et [exécuter un suite de tests unitaires](https://blog.3ie.fr/automatiser-les-tests-sur-votre-projet-typescript-avec-grunt/).

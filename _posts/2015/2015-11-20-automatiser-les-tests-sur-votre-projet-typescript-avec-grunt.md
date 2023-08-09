---
title: "Automatisez les tests sur votre projet Typescript avec Grunt"
date: "2015-11-20"
categories: 
  - "technical"
tags: 
  - "angularjs"
  - "grunt"
  - "karma"
  - "typescript"
  - "vscode"
---

Pour faire suite à notre précédent article sur un projet [AngularJS en typescript](https://blog.3ie.fr/un-projet-angularjs-avec-typescript/), nous allons maintenant voir comment automatiser les tests à l'aide de [Grunt](http://gruntjs.com/). Dans le prochain article, nous verrons [comment optimiser notre application pour la distribution](https://blog.3ie.fr/optimisez-les-releases-de-vos-projets-typescriptangular-avec-grunt/).

Le but n'est pas ici de faire un scénario ultra complexe avec divers environnements (dev, VS, prod), mais d'avoir un Gruntfile concis et puissant.

# Présentation des étapes

Nous souhaitons mettre en place les processus suivants :

- vérification de la propreté du code avec [tslint](https://github.com/palantir/tslint)
- exécution de tests unitaires avec [Karma](http://karma-runner.github.io/) (qui se charge de lancer les tests) et [Jasmine](http://jasmine.github.io/) (un framework de tests unitaires)

## tslint

Tout d'abord nous allons exécuter 'tslint' sur tous nos fichiers ts. 'tslint' est un outil qui vérifie un _coding style_ pour garantir une uniformité dans le code et éviter certaines mauvaises pratiques.

### Installation et paramètrage de tslint

Plutôt que partir de zéro, vous pouvez récupérer un fichier [tslint.json d’example](https://github.com/palantir/tslint/blob/master/docs/sample.tslint.json) sur github (le lien pointe sur la master, il peut contenir des options qui ne sont pas dans la dernière version stable). Le fichier est à renommer en tslint.json et à sauvegarder à la racine du projet. La configuration par défaut est vraiment très stricte, notamment sur le typage des variables où elle n'autorise pas d'inférence de type, mais pour l'exemple nous allons garder ce fichier quasiment inchangé.

Pour éviter que votre GIT soit souillé par le refactoring des différents devs, définissez une politique d'indentation ! Ici, nous avons choisi les tabulations. Nous avons choisi de n’utiliser que des tabs.

Dans le fichier _tslint.json,_ passez la propriété _indent_ à _tabs_ pour que tslint le vérifie:

```js
"indent": [ true, "tabs" ]
```

 

Une deuxième modification intéressante dans tslint est l'option _quotemark_ pour forcer une cohérence dans la saisie des chaines de caractères. Comme on est dans un univers proche du JS, on gardera les _string_ en _single quote_, ce qui permet de manipuler facilement du code html avec des _double quotes_. Passez la propriété _quotemark_ à “single”

```js
"quotemark": [
	true,
	"single",
	"avoid-escape"
]
```

 

Vous pouvez aussi augmenter le nombre de caractères maximum autorisé sur une ligne. De base la limite est à 140 mais vous pouvez le passer à 220 car ça se lit sans problème sur un grand écran.

```js
"max-line-length": [
	true,
	220
]
```

 

Nous allons garder l’option _no-var-keyword_ dans le tslint.conf ce qui permet d'interdire l’utilisation du mot clé “var”. Il faut alors uniquement passer par des _let_ (pour déclarer des variables qu'on peut ré-affecter) et des _const_ (pour des variables qu'on ne peut affecter qu'une fois). L'intérêt, en plus de permettre d'aider à la compréhension du code, est que ce sont des variables dont la visibilité est limitée au scope où elles sont déclarées. Cette approche a été adoptée dans [l'ECMAScript6](http://www.ecma-international.org/ecma-262/6.0/#sec-let-and-const-declarations) et on retrouve le même concept chez Apple avec le [Swift](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html) (mais avec les mots clés _var_ et _let_).

### Intégration de tslint dans VSCode

VSCode supporte depuis la version v0.10 les extensions, vous pouvez donc intégrer la vérification directement dans l'IDE grâce au plugin [tslint pour VSCode](https://marketplace.visualstudio.com/items/eg2.tslint).

Pour garantir une bonne indentation, demandez à VSCode de le faire pour vous. Pour cela, modifiez vos préférences (Preferences->User settings) et passez la propriété "editor.insertSpaces” à false :

```js
"editor.insertSpaces": false
```

 

### Intégration de tslint dans grunt

Nous n'allons pas couvrir les bases de Grunt dans cet article, je vous invite donc à lire [la présentation de grunt](http://gruntjs.com/getting-started) sur le site officiel.

#### Ajout des dépendances Grunt

Ajoutez dans votre 'package.json' la dépendance vers 'tslint'. Ajoutez aussi la dépendances ['load-grunt-tasks'](https://github.com/sindresorhus/load-grunt-tasks), cela permet de charger automatiquement les taches grunt prédéfinies. Vous pouvez alors remplacer

```js
grunt.loadNpmTasks('grunt-contrib-clean');
grunt.loadNpmTasks("grunt-ts");
grunt.loadNpmTasks('grunt-http-server');
```

par cette ligne

```js
 require('load-grunt-tasks')(grunt);
```

#### Modification du Gruntfile

Créez une tache 'testing' dans votre grunt. C'est à travers elle que l'on exécutera le 'tslint' et ensuite les tests unitaires.

```js
grunt.registerTask('testing', [
	'clean',
	'tslint'
]);
```

Dans votre bloc de 'grunt.initConfig', ajoutez la configuration pour votre tache 'tslint'.

```js
tslint: {
	options: {
		configuration: grunt.file.readJSON("tslint.json")
	},
	files: {
		//all your .ts files
		[
			'app/**/*.ts',
			'app_engine/**/*.ts',
			'models/**/*.ts',
			'test/**/*.ts',
		]
	}
}
```

L'attribut 'files' contient la liste de tous les fichiers _ts_ à vérifier, c'est à dire tous les fichiers _*.ts_ contenu dans les dossiers app, app_engine, models, et test (dossier qui contiendra tous les tests unitaires).

Comme vous pouvez le voir, c'est un peu verbeux et on l'impression de se répéter. Heureusement Grunt permet d'utiliser les même règles de _[brace expansion](https://www.gnu.org/software/bash/manual/html_node/Brace-Expansion.html)_ que Bash, c'est à dire utiliser des accolades pour générer des combinaisons de chaine de caractères.

On peut donc écrire la règle suivante qui sera équivalente

```js
tsFiles: ['{app,app_engine,models,test}/**/*.ts']
```

Pour garder le code Grunt propre, nous allons utiliser une variable globale pour stocker nos tableaux de fichiers. Ce code est à placer au début de votre Grunt, au dessus de votre 'grunt.initConfig'.

```js
var globalCfg = {
	src: {
		tsFiles: ['{app,app_engine,models,test}/**/*.ts']
	}
};
```

Notre configuration de _tslint_ ressemble donc à cela maintenant

```js
tslint: {
	options: {
		configuration: grunt.file.readJSON("tslint.json")
	},
	files: {
		src: '<%= globalCfg.src.tsFiles %>'
	}
}
```

 

# Les tests unitaires

[Karma](http://karma-runner.github.io) est ce qu'on appelle un _test runner_, c'est à dire une application qui va s'occuper de faire tourner une série de tests. Pour bien fonctionner, on lui adjoint [Jasmine](http://jasmine.github.io/), un framework de tests unitaires, et [PhantomJS](http://phantomjs.org/), un _headless browser_ (un navigateur web qui fonctionne sans interface graphique).

La création de tests unitaires n'étant pas le sujet de cet article, on ne détaillera pas ce processus ici.

### Installation de Karma et ses dépendances

Ajoutez les dépendances suivantes à votre fichier _package.json_

- grunt-karma
- jasmine
- jasmine-core
- karma
- karma-jasmine
- karma-phantomjs-launcher
- phantomjs

Pour fonctionner, Karma a besoin d'un fichier de configuration. Vous pouvez récupérer un fichier [karma.conf d'example](https://github.com/karma-runner/karma/blob/master/test/client/karma.conf.js) sur le git officiel.

Pensez à mettre à jour vos définitions tsd pour Jasmine avec la commande suivante :

```sh
$ tsd install jasmine -s
```

### Modification du Gruntfile

Dans le fichier Gruntfile.js, modifiez la tache grunt 'testing' pour que les tests unitaires s'effectuent après _tslint_ et après la compilation des fichiers _ts_.

```js
grunt.registerTask('testing', [
	'clean',
	'tslint',
	'ts',
	'karma:continuous',
]);
```

Il ne nous reste plus qu'a créer la tâche _karma_. On crée une target 'continuous' dans laquelle Karma est configuré pour lancer chaque test dans PhantomJS; l'option _singleRun_ fait que Karma ferme PhantomJS à la fin de chaque test. Ajoutez ce code à coté de la tache _tslint_ précédemment créée:

```js
karma: {
	//continuous integration mode: run tests once in PhantomJS browser.
	continuous: {
		configFile: 'karma.conf.js',
		singleRun: true,
		browsers: ['PhantomJS']
	}
}
```

# Conclusion

Voila le Gruntfile final que l'on obtient.

```js
module.exports = function (grunt) {
	// load all grunt tasks without explicitly referecing them
	require('load-grunt-tasks')(grunt);

	var globalCfg = {
		src: {
			tsFiles: ['{app,app_engine,models,test}/**/*.ts']
		}
	};
		
	grunt.initConfig({
		pkg: grunt.file.readJSON('package.json'),
		globalCfg: globalCfg,
		ts: {
			default: {
				src: '<%= globalCfg.src.tsFiles %>',
				options: {
					target: 'es5',
					module: 'commonjs',
					noEmitOnError: true,
					fast: 'never'
				}
			}
		},
		tslint: {
			options: {
				configuration: grunt.file.readJSON("tslint.json")
			},
			files: {
				src: '<%= globalCfg.src.tsFiles %>'
			}
		},
		karma: {
			//continuous integration mode: run tests once in PhantomJS browser.
			continuous: {
				configFile: 'karma.conf.js',
				singleRun: true,
				browsers: ['PhantomJS']
			},
		},
		clean: {
			public: [
				'{app,app_engine,models}/**/*.{js,js.map}',
				'reference.{js,js.map}'
			]
		},
		'http-server': {
			'dev': {
				root: '<%= globalCfg.distDir %>',
				port: 5000,
				openBrowser: true
			}
		}
	});

	grunt.registerTask('testing', [
		'clean',
		'tslint',
		'ts',
		'karma:continuous',
	]);
	grunt.registerTask('web', ['http-server:dev']);
};

```

Vous pouvez télécharger le projet complet sur le git du [TypescriptAngularStarter v0.2.3](https://github.com/3IE/TypescriptAngularStarter/tree/v0.2.3)
<br>
<br>

---------------------------------------
<br>
Auteur: **benoit.verdier**

---
title: "Optimisez les \"releases\" de vos projets Typescript/Angular avec Grunt"
date: "2015-11-24"
categories: 
  - "technical"
tags: 
  - "angularjs"
  - "grunt"
  - "less"
  - "minify"
  - "typescript"
  - "uglify"
  - "usemin"
---

Pour faire suite à notre [précédent article](https://blog.3ie.fr/automatiser-les-tests-sur-votre-projet-typescript-avec-grunt/) sur la mise en place de votre chaîne de test avec _Grunt_, nous allons aborder la "génération" de votre application avec un code optimisé.

 

# Présentation de usemin

_Usemin_ est un outil qui s'occupe de remplacer, dans votre code html, les références à des fichiers _CSS_ et _JS_ non optimisés. Il s'occupe de générer la configuration de différents plugins Grunt et de déplacer/modifier les fichiers pour les traiter.

Pour faire notre version, nous allons donc passer par les étapes suivantes:

- concaténation et minification des fichiers **CSS** et **JS**
- compilation des fichiers **less** en CSS
- copie des fichiers "statiques" (pages html, fonts, images)
- génération d'un nom unique pour les images, CSS et JS

## Tâches principales

_Usemin_ déclare 2 taches :

- _useminPreprare_ qui s'occupe de générer la configuration des différents plugins Grunt; pour cela il y a une notion de répertoire source et répertoire destination, ce qui permet de ne pas modifier directement son code source
- _usemin_ qui remplace les références vers les ressources dans les fichiers html

Pour chaque plugin, _Usemin_ va générer une _Target_ 'generated' à l'intérieur de chaque tâche, qu'il faudra ensuite exécuter entre l'étape _useminPreprare_ et _usemin_.

## Plugins supportés par Usemin

Usemin est capable d'utiliser et de configurer les plugins Grunt suivants:

- [`concat`](https://github.com/gruntjs/grunt-contrib-concat) qui concatène des fichiers JS et CSS
- [`uglify`](https://github.com/gruntjs/grunt-contrib-uglify) qui minifie les fichiers JS.
- [`cssmin`](https://github.com/gruntjs/grunt-contrib-cssmin) qui minifie les fichier CSS.
- [`filerev`](https://github.com/yeoman/grunt-filerev) qui génère des noms de fichier unique pour les fichiers de ressources

Pour la compilation des fichiers less, nous ajouterons une tâche de configuration dans _Usemin_, et pour la copie des fichiers statiques, nous configurerons la tâche de manière statique.

## Installation des packages npm

Il faut que vous installiez les packages suivants:

- less-plugin-clean-css
- less-plugin-autoprefix
- grunt-usemin
- grunt-contrib-concat
- grunt-contrib-cssmin
- grunt-contrib-uglify
- grunt-contrib-copy
- grunt-filerev
- grunt-contrib-less
- grunt-contrib-clean

Pour rappel, cela peut se faire directement dans le fichier 'package.json' (suivi d'un 'npm install') ou bien en passant par la commande ''npm install PACKAGE\_NAME --save" .

 

# Mise en place de Usemin

### Etapes préparatoires

##### Mise à jour des variables globales

Il nous faut tout d'abord ajouter les variables que l'on va utiliser dans le reste du Gruntfile :

```js
var globalCfg = {
	src: {
		tsFiles: \['{app,app\_engine,models,test}/**/*.ts','reference.ts'\],
		generatedJSFiles: \['{app,app\_engine,models,test}/**/*.{js,js.map}', 'reference.{js,js.map}'\],
		staticMiscFiles: \['index.html', 'app/**/*.{html,json}', 'app/img/*\*'\],
		staticFontFiles: \['bower\_components/bootstrap/dist/fonts/*\*'\]
	},
	distDir: 'dist/'
};
```

##### Modification de la tâche 'build'

Modifiez votre tâche _build_ de la façon suivante :

```js
grunt.registerTask('build', \[
	'testing',
	'copy',
	'useminPrepare',
	'less:generated',
	'concat:generated',
	'cssmin:generated',
	'uglify:generated',
	'filerev',
	'usemin'
\]);
```

Nous avons maintenant le déroulé suivant :

1. testing: validation du code Typescript et compilation en Javascript
2. copy: copie des ressources statiques dans le dossier de destination
3. useminPrepare: génération de la configuration des tâches
4. \*:generated: exécution des différentes tâches via la _Target_ 'generated'
5. filerev: renommage des ressources avec un nom unique basé sur le contenu du fichier (basé sur le md5)
6. usemin: remplacement des références vers les ressources dans les fichiers html

## Tache 'copy'

Cette tâche est simple. Nous voulons copier nos fichiers statiques vers le répertoire de distrib. Nous avons une règle spéciale pour les _fonts_ fournies par Bootstrap car ce dernier les référence depuis ses fichiers CSS, que nous concaténons. Il faut donc bien penser à copier ces ressources car Usemin ne le fera pas pour vous.

Ajoutez la tâche suivante dans votre Gruntfile.

```js
copy: {
	main: {
		files: \[
			{ expand: true, src: '<%= globalCfg.src.staticMiscFiles %>', dest: '<%= globalCfg.distDir %>', filter: 'isFile' },
			{ expand: true, src: '<%= globalCfg.src.staticFontFiles %>', dest: '<%= globalCfg.distDir %>/fonts/', flatten: true, filter: 'isFile' }
		\],
	},
}
```

## Tache 'useminPrepare'

Voila le code à ajouter dans votre _Gruntfile_, nous allons l'expliquer juste après.

```js
useminPrepare: {
	//html file that usemin is going to analyse to find the files to process
	html: 'app/index.html',
	options: {
		//destination folder that usemin is going to use to output the processed files (for .js and .css for example)
		dest: '<%= globalCfg.distDir %>/app/',
		//this custom flow allows usemin to support less
		flow: {
			steps: {
				'js': \['concat', 'uglify'\],
				'css': \['concat', 'cssmin'\],
				'less': \[{
					name: 'less',
					createConfig: lessCreateConfig
				}\]
			},
			post: {}
		},
	}
}
```

##### Détection des fichiers à traiter

On peut spécifier soit même le nom des fichiers _css_ et _js_ à traiter par _Usemin_ mais le plus intéressant est de le laisser explorer votre code html pour les trouver. Cela se fait avec la _target_ 'html'.

Pour que usemin sache quelles références traiter dans votre html, il faut définir ce qu'on appelle des _Blocks à l'aide_ de commentaires dans votre fichier html. Le format est le suivant :

```xhtml
<!-- build:<type> <path> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

Le _type_ correspond au type de fichier configuré dans le flux de traitement de usemin (nous en parlerons plus en détail juste après). Le _path_ correspond au fichier de destination.

A titre d'exemple, voila 2 _blocks_ tirés de notre fichier index.html

```xhtml
<!-- build:css css/globals.css -->
<link rel="stylesheet" href="../bower\_components/bootstrap/dist/css/bootstrap.css">
<link href="css/4-col-portfolio.css" rel="stylesheet">
<!-- endbuild -->
```

```xhtml
<!-- build:js js/app.js -->
<script src="../bower\_components/angular/angular.js"></script>
<script src="app.startup.js"></script>
<!-- endbuild -->
```

##### Répertoire de destination

La configuration du répertoire de destination se fait par l'option 'dest'. C'est là que _Usemin_ placera les différents fichiers générés, même si pour certaines étapes intermédiaires, _Usemin_ utilise un dossier '.tmp'.

##### Redéfinition du flux de traitement

Par défaut, _Usemin_ sait traiter les _JS_ et les _CSS_. Pour gérer les fichiers _less_, on pourrait simplement créer une tâche 'less' qui cible les fichiers en question, mais c'est beaucoup plus puissant et souple de configurer usemin pour qu'il analyse le _html_ pour en extraire la liste de fichiers à traiter.

Pour cela, la première étape est de redéfinir le _flow_ de _Usemin_ (on ne peut pas simplement étendre le flux existant). Les étapes 'js' et 'css' peuvent référencer les tâches pré-existantes, 'concat', 'uglify' et 'cssmin', par contre pour l'étape 'less' il nous faut expliquer à _Usemin_ quoi faire. C'est ce que nous avons fait avec l'option 'flow'.

```js
flow: {
	steps: {
		'js': \['concat', 'uglify'\],
		'css': \['concat', 'cssmin'\],
		'less': \[{
			name: 'less',
			createConfig: lessCreateConfig
		}\]
	},
	post: {}
}
```

Le paramètre 'name' permet de spécifier quelle tâche Grunt exécuter et le paramètre 'createConfig' est une fonction qui va s'occuper de créer la configuration de la tâche Grunt (ici la tache 'less'). Pour une question de lisibilité,nous vous recommandons de sortir cette fonction du 'flow' de usemin.

Déclarez cette fonction en haut de votre Gruntfile, à coté de votre variable globale 'globalCfg' :

```js
//less config function for usemin
var lessCreateConfig = function (context, block) {
	var cfg = { files: \[\] },
		//the destination file is the one referenced in the html and it's to be placed in the context.outDir folder 
		outfile = path.join(context.outDir, block.dest),
		filesDef = {};

	filesDef.dest = outfile;
	filesDef.src = \[\];
	//we have to process each 'less' file referenced in the html, and they are in the 'inDir' folder 
	context.inFiles.forEach(function (inFile) {
		filesDef.src.push(path.join(context.inDir, inFile));
	});

	cfg.files.push(filesDef);
	context.outFiles = \[block.dest\];
	return cfg;
};
```

Cette fonction utilise 2 paramètres fournis par _usemin._

- context: un objet qui contient le contexte d'exécution de l'étape que _usemin_ est entrain de traiter
- block: un objet qui contient le _block_ que usemin a extrait du fichier html (celui contenu entre <!-- build --> et <!-- endbuild --> )

Pour en savoir plus sur le sujet, vous pouvez vous référez à la doc usemin sur le [createConfig](https://github.com/yeoman/grunt-usemin#createconfig).

## Tâche concat

Ce travail se fait avec le plugin `[grunt-contrib-concat](https://github.com/gruntjs/grunt-contrib-concat)`. Le but est de générer un fichier unique à télécharger par le navigateur pour limiter le nombre de requêtes réseau du browser web.

La configuration générée par _usemin_ étant suffisante, la tâche n'est pas présente dans le Gruntfile. Elle est quand même utilisable par notre tâche de 'build' car une fois que _Usemin_ a été exécuté, _Grunt_ peut bien appeler 'concat:generated'.

## Tâche cssmin

Ce travail se fait avec le plugin [grunt-contrib-cssmin](https://github.com/gruntjs/grunt-contrib-cssmin) (qui se charge d'appeler [clean-css](https://github.com/jakubpawlowicz/clean-css)). Ce plugin va retirer tout ce qui est inutile dans le CSS pour réduire sa taille.

La configuration générée par _Usemin_ étant suffisante, la tâche n'est pas présente dans le Gruntfile. Elle est quand même utilisable par notre tache de 'build' car une fois que _Usemin_ a été exécuté, _Grunt_ peut bien appeler 'cssmin:generated'.

## Tache uglify

L'objectif est de rendre votre code Javascript moche pour compliquer sa compréhension par les curieux, et au passage diminuer la taille du fichier en retirant les informations inutiles (commentaires, nom de variables plus court, formatage de code, ...). Cela se fait avec le plugin [grunt-contrib-uglify](http://grunt-contrib-uglify).

La configuration générée par _Usemin_ étant suffisante, la tâche n'est pas présente dans le Gruntfile. Elle est quand même utilisable par notre tache de 'build' car une fois que _Usemin_ a été exécuté, _Grunt_ peut bien appeler 'uglify:generated'.

## Tache less

La compilation des fichiers _less_ se fait avec le plugin [grunt-contrib-less](https://github.com/gruntjs/grunt-contrib-less). Nous allons aussi activer 2 plugins spécifiques :

- [less-plugin-autoprefix](https://github.com/less/less-plugin-autoprefix) qui exécute [autoprefixer](http://autoprefixer.github.io/)
- [less-plugin-clean-css](https://github.com/less/less-plugin-clean-css) qui exécute [clean-css](https://github.com/jakubpawlowicz/clean-css), qui était déjà utilisé par la tache _cssmin_

##### Autoprefixer

Autoprefixer s'occupe d'ajouter les préfixes des différents vendor pour vos règles CSS. Il permet donc de garder un code _Less_ plus propre et d'éviter d'avoir à vérifier chaque commande CSS sur un site comme [caniuse](http://caniuse.com/). Attention par contre, si vous utilisez une règle CSS incompatible avec une partie des navigateurs que vous ciblez, il n'y aura aucune alerte.

Par exemple, pour le code suivant :

```css
.example {
    display: flex;
}
```

on obtient cette adaptation :

```css
.example {
    display: -webkit-flex;
    display: -ms-flexbox;
    display: flex;
}
```

##### Configuration Grunt

Créez la tâche suivante votre Gruntfile:

```js
less: {
	options: {
		plugins: \[
			//plugin that automatically prefixes the css rules with vendor prefixes when needed 
			new (require('less-plugin-autoprefix'))({ browsers: \["last 2 versions"\] }),
			//plugin that minifies the css
			new (require('less-plugin-clean-css'))()
		\]
	}
}
```

## Tache filerev

Le plugin [grunt-filerev](https://github.com/yeoman/grunt-filerev) s'occupe de renommer les images, _css_ et _js_ pour éviter les problèmes de cache. Il génère un nom unique de fichier basé sur le nom original du fichier et sur son md5.

Il est intéressant de déclarer plusieurs _Target_ dans la tache Grunt, une par dossiers à traiter, car cela permet d'avoir un feedback plus riche lors de l'exécution du 'grunt build'.

Quand _filerev_ s'exécute, il génère un dictionnaire de tous les fichiers qu'il a remplacé. Cela est stocké dans une variable 'grunt.filerev.summary'. De cette façon, quand _Usemin_ est appelé, il sait comment remplacer les références (désormais invalides) vers les ressources.

Ajoutez la tâche suivante dans votre Gruntfile :

```js
filerev: {
	options: {
		algorithm: 'md5',
		length: 8
	},
	images: { src: '<%= globalCfg.distDir %>/app/img/**/*' },
	js: { src: '<%= globalCfg.distDir %>/app/js/**/*' },
	css: { src: '<%= globalCfg.distDir %>/app/css/**/*' }
},
```

## Tâche usemin

C'est la tâche finale pour packager notre release. Usemin va parcourir votre html présent dans le répertoire de distrib pour remplacer les différentes références aux ressources.

```js
usemin: {
	//html file within which usemin is going to replace the resource references  
	html: \['<%= globalCfg.distDir %>/app/**/*.html'\],
	options: {
		assetsDirs: '<%= globalCfg.distDir %>/app',
		blockReplacements: {
			less: function (block) {
				return '<link rel="stylesheet" href="' + block.dest + '">';
			}
		}
	}
}
```

Pour les fichiers _css_ et _jss_, _usemin_ va remplacer tout le _Block_ par sa version concaténée et minifiée. Pour les fichiers _Less_, qui ne sont pas gérés nativement par _Usemin_, nous devons préciser comment remplacer les _Blocks_. Vous devez fournir une fonction de traitement par l'option 'blockReplacements'. Pour les autres ressources, _Usemin_ va vérifier dans l'objet 'grunt.filerev.summary' si cette dernière n'a pas été renommée. Si c'est le cas, _Usemin_ vérifie dans la liste de dossiers de ressources (configuré par l'option 'assetsDirs') si le fichier existe.

 

# Conlusion

Vous pouvez maintenant lancer la commande 'grunt build' pour packager votre application vers le dossier 'dist' et ensuite utiliser la commande 'grunt web' pour tester le résultat.

Vous pouvez retrouver le projet complet sur le git [TypescriptAngularStarter](https://github.com/3IE/TypescriptAngularStarter/tree/v0.2.3) et le détail du [fichier gruntfile](https://github.com/3IE/TypescriptAngularStarter/blob/v0.2.3/Gruntfile.js).

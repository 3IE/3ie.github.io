---
title: "Adoptez l'intégration et le déploiement continus avec Travis et Cloud Foundry"
date: "2016-02-26"
categories: 
  - "technical"
tags: 
  - "bluemix"
  - "cloud-foundry"
  - "grunt"
  - "travis"
---

Nous allons voir comment ajouter facilement à un projet web de l'intégration continue et du déploiement continu. Pour ce faire, nous allons étudier le cas de notre [starter Typescript + Angular](https://github.com/3IE/TypescriptAngularStarter)

# Pourquoi est-ce intéressant ?

Tout d'abord, pourquoi vouloir mettre en place ce type de process ?

[![](/assets/images/GithubTravisCF-300x250.png)](/assets/images/GithubTravisCF.png)

#### L'intégration continue

Elle permet de vérifier à chaque push que notre build se génère toujours correctement et que nos tests unitaires passent toujours. On peut consulter l'historique des builds, sur chaque branche, et en cas d'erreur, un mail est envoyé.

[Travis CI](http://travis-ci.org)  est une solution SaaS qui vous permet de ne pas avoir à gérer les différents serveurs pour mettre en place ce type de solution tout en facilitant la configuration par projet. Il est compatible avec beaucoup d'environnements, y compris mac pour les projets Xcode.

#### Le déploiement continu

Il permet de tester le dernier build généré avec succès sans avoir besoin qu'un admin sys mette les mains dans le cambouis.

[Cloud Foundry](https://www.cloudfoundry.org/) est une solution PaaS Open Source sur qui permet d'héberger facilement vos projets. Il n'y a pas à se soucier de la maintenance de la plateforme et vous pouvez générer facilement diverses url suivant la branche sur laquelle les équipes travaillent (parfait pour une approche DevOps).

Il existe 2 acteurs principaux qui vendent des services basé sur Cloud Foundry : [Pivotal Web Service](https://run.pivotal.io/) et [Bluemix](https://console.ng.bluemix.net/) avec les "Instant Runtimes". Dans cette article, nous allons manipuler Pivotal.

[Heroku](https://www.heroku.com/) utilise lui aussi des _buildpacks_ (même s'il ne se base pas sur Cloud Foundry), et il possède un workflow intéressant qui permet de promouvoir une release en production. Il n'y a malheureusement pour le moment pas de gestion de droits utilisateurs, ce qui est vraiment bloquant pour une entreprise (je mets de coté Heroku Entreprise qui demande plusieurs milliers de dollars de budget par an). C'est pourquoi nous avons choisi Pivotal.

 

# Intégration continue avec Travis CI

## Configuration du compte

Travis propose une offre gratuite pour tous les projets open source, ce qui vous permet de tester leur solution facilement. Pas besoin de gérer manuellement des webhooks ni de maintenir à jour vos droits d'accès sur les projets, Travis s'occupe de tout pour vous, il suffit  simplement de se connecter via votre compte _github_ sur le site de [Travis](http://travis-ci.org).

## Logique de fonctionnement

La configuration d'un projet pour Travis passe par un fichier '.travis.yml' dans lequel il faut déclarer le langage à utiliser (pour que Travis sache quelle image système charger) et ses différents paramètres. A chaque commit, Travis va injecter votre code dans une image système et lancer un jeu de commandes (basé sur le type de projet) mais qui se rapportera toujours au ['build lifecycle'](https://docs.travis-ci.com/user/customizing-the-build/#The-Build-Lifecycle) global.

Nous ne sommes pas vraiment sur un projet NodeJS mais nous utilisons fortement _npm_, c'est pourquoi cette configuration est adaptée.

Comme aucun package n'est pré-installé sur l'image Travis, il est important de mettre toutes ses dépendances dans le 'package.json', sans utiliser de package installé globalement sur le système. La seule exception est le _CLI_ de _grunt_ que nous allons devoir installer avant que Travis n'exécute son scénario complet.

## Configuration d'un projet

Pour rendre notre projet compatible, un fichier '.travis.yml' très simple est suffisant :

```yaml
language: node\_js
node\_js:
  - "stable"
before\_script:
  - npm install grunt-cli -g
```

Le paramètre 'language' sert à configurer l'image système à utiliser (NodeJS dans notre cas). Le paramètre 'before\_script' (qui se rapporte au lifecycle) nous permet d'installer _grunt_ avant que Travis n'exécute d'autres commandes.

 

Comme nous utilisons _bower_, il faut que ce dernier puisse installer ses dépendances. Pour cela nous avons modifié la propriété 'script' du 'package.json' (pour rappel, le fichier de configuration de _npm_) pour exécuter un 'bower install' (pour installer les packages _bower_) et un 'typings install' (pour installer les définitions _Typescript_). Cette configuration est à faire dans le script 'postinstall' qui est exécuté une fois que _npm_ a fini l'installation de ses packages.

Dans le 'package.json', nous avons donc la configuration suivante :

```js
"scripts": {
	"postinstall": "bower install && typings install"
}
```

 

Ensuite, conformément à la [documentation de Travis pour NodeJS](https://docs.travis-ci.com/user/languages/javascript-with-nodejs#Default-Test-Script), c'est la commande 'npm test' qui va être exécutée. Nous avons donc aussi reconfiguré cette étape pour exécuter 'grunt test', ce qui nous donne :

```js
"scripts": {
	"postinstall": "bower install && typings install,
	"test": "grunt test"
}
```

# Déploiement Continu avec Cloud Foundry

## Logique de fonctionnement

Les interactions avec Pivotal se font via un CLI, qui s’appelle ‘cf’ pour CloudFoundry, à installer depuis la [section tools de Pivotal](https://console.run.pivotal.io/tools). L'idée est que l'on va push notre code vers Pivotal qui va ensuite instancier notre application web. La configuration se fait via un fichier 'manifest.yml'.

Cloud Foundry utilise des images systèmes que l'on appelle _buildpacks_ et qui contiennent les différents binaires et dépendances nécessaires à votre projet. On peut spécifier le _buildpack_ à utiliser, mais Cloud Foundry sait détecter le type de projet. Dans notre cas, Cloud Foundry détecte un fichier 'package.json' dans le dossier racine, ce qui indique un projet NodeJS.

Il y a sans doute plus adapté pour servir un site web statique, d'autant plus que nous ne somme pas vraiment entrain de faire du NodeJS, on utilise simplement _npm_ pour gérer nos dépendances et avoir une sorte de makefile. On pourrait par exemple utiliser un _buildpack_ qui exécute un _nginx_, mais pour rester simple, nous allons continuer sur le scénario NodeJS.

## Configuration

#### manifest.yml

Regardons le contenu de notre ‘manifest.yml’ :

```yaml
applications:
- name: dev-angulartypescriptstarter
  memory: 128M
```

Il est très simple et contient simplement le nom du projet et la quantité de mémoire que l’on veut utiliser.

 

Dans le cadre d'un _buildpack_ NodeJS, il y a 2 étapes qui nous intéressent

- le [build](https://devcenter.heroku.com/articles/nodejs-support#build-behavior) qui exécute un $ npm install
- le [runtime](https://devcenter.heroku.com/articles/nodejs-support#default-web-process-type) qui exécute un $ npm start

Nous avons donc reconfiguré, via le fichier 'package.json', le script 'postinstall' (pour que le projet soit automatiquement _build_ une fois les packages _npm_ et _bower_ installés) ansi que le script 'start' (pour lancer un serveur web).

Cela nous donne donc la config suivante :

```js
"scripts": {
	"postinstall": "bower install && typings install && grunt build",
	"test": "grunt test",
	"start": "grunt connect:cloudfoundry"
}
```

#### Gruntfile.js

Intéressons nous un instant à la tache 'grunt connect' à laquelle nous avons ajouté la _target_ 'cloudfoundry'.

```js
connect: {
	dev: {
		options: {
			port: 5000,
			base: '<%= globalCfg.distDir %>',
			open: true,
			debug: true,
			keepalive: true
		}
	},
	cloudfoundry: {
		options: {
			port: process.env.PORT || 5000,
			base: '<%= globalCfg.distDir %>',
			keepalive: true
		}
	}
}
```

Les 2 _targets_ ont le même objectif : lancer un serveur web (en l'occurence, [connect](https://www.npmjs.com/package/connect)) qui va _host_ votre site

- la _target_ 'dev' va servir votre site sur le port 5000, depuis le dossier qui a été build par grunt; en plus elle va ouvrir votre browser (open: true) tout en affichant dans votre terminal les différentes requêtes (debug: true), ce qui vous permet de tester que le site s'affiche bien
- la _target_ 'cloudfoundry' lance le server web, mais en utilisant le port renseigné dans les variables d'environnement par Cloud Foundry (port: process.env.PORT || 5000)

## Déploiement

Avant de push le site sur Cloud Foundry, assurez vous qu'il s'affiche bien en local avec la commande  $ grunt build && grunt connect:dev.

Pour éviter de transférer inutilement tous les artefacts de _npm_ et _bower_ (les dossiers 'node\_modules' et 'bower\_components'), nous avons créé un fichier '.cfignore' qui fonctionne sur le même principe que le .'gitignore'. Vous pouvez y ajouter tous les fichiers ou dossiers que vous ne souhaitez pas transférer à Cloud Foundry.

Maintenant, tapez la commande suivante pour vous connecter à l’endpoint de Pivotal; en effet, comme le CLI est compatible avec toutes les solutions Cloud Foundry, il nous faut préciser laquelle utiliser :

$ cf login -a api.run.pivotal.io

```batch  

Nous pouvons maintenant push notre projet sur Pivotal avec la commande $ cf push qui va utiliser les paramètres de notre 'manifest.yml' pour finaliser le déploiement.

# Interconnexion entre Travis et Cloud Foundry

## Logique de fonctionnement

La dernière étape est de connecter les 2 services pour que Travis push automatiquement le projet à Cloud Foundry, mais seulement si les tests se passent bien. Cela se fait avec le fichier '.travis.yml' via la section _deploy_. Cette dernière permet de renseigner tous les _provider_ que vous souhaitez utiliser.

Pour Cloud Foundry, le plus simple est d'utiliser le [CLI de Travis](https://github.com/travis-ci/travis.rb#installation) qui va s'occuper de générer la bonne configuration basée sur vos informations. Cependant, pour faire un déploiement vers Cloud Foundry, il est nécessaire de fournir votre mot de passe, qui va être stocké dans le fichier '.travis.yml' sur votre git, à minima partagé avec plusieurs utilisateurs, au pire public. Heureusement Travis a prévu ce cas, et il est possible d'encoder ces infos à partir du CLI.

L'encryption de Travis se base sur le principe de cryptographie à clé publique :

- pour chaque git, Travis stocke une clé publique et une clé privée
- l'utilisateur peut récupérer la clé publique du git et encoder des données
- seul Travis pourra décoder les données cryptées qui nécessitent la clé privée

## Déploiement automatique

Une fois le CLI Travis installé, tapez la commande suivante pour créer la section _deploy_ dans le fichier '.travis.yml'

$ travis setup cloudfoundry
```

En réponse, Travis nous a généré la configuration suivante :

```yaml
deploy:
  - provider: cloudfoundry
    api: https://api.run.pivotal.io
    username: #Pivotal account
    password:
      secure: #Pivotal encrypted password
    organization: 3ie
    space: development
```

## Améliorations possibles

#### Déploiement conditionnel

Il est possible d'ajouter des conditions rattachées à chaque provider. Il est par exemple intéressant de déployer une branche 'release' sur un serveur à part. Dans ce cas, il faut dupliquer notre provider 'cloudfoundry' et ajouter des conditions sur chacun à l'aide de la commande 'on'. Chaque _provider_ peut référencer un fichier 'manifest.yml', ce qui nous permet bien d'avoir des noms de sites différents suivant la branche. Comme Pivotal ne supporte pas les _wildcards_ quand on cible une 'branch', il faut passer par une 'condition' pour matcher les branches de type _release_.

Cela nous donne la config suivante dans le fichier '.travis.yml' :

```yaml
deploy:
  - provider: cloudfoundry
    api: https://api.run.pivotal.io
    username: #Pivotal account
    password:
      secure: #Pivotal encrypted password
    organization: 3ie
    space: development
    manifest: manifest.yml
    on:
      repo: 3IE/TypescriptAngularStarter
      branch: develop
  - provider: cloudfoundry
    api: https://api.run.pivotal.io
    username: #Pivotal account
    password:
      secure: #Pivotal encrypted password
    organization: 3ie
    space: pre-production
    manifest: manifest-preprod.yml
    on:
      repo: 3IE/TypescriptAngularStarter
      all\_branches: true
      condition: ${TRAVIS\_BRANCH%%/*} == release #we check if the start of the branch name contains 'release'

```

#### Buildpack plus optimisé

Nous en parlions plus haut, pour avoir un hébergement plus léger et plus adapté à notre site statique, il est possible d'utiliser un '[staticfile-buildpack](https://github.com/cloudfoundry/staticfile-buildpack)' basé sur Nginx. De cette façon on consomme beaucoup moins d'espace de stockage (on stocke seulement le site, pas les dépendances) et on peut réduire la mémoire vive de l'application à 64mo (largement suffisant pour les besoins d'un site en dev).

Cette solution a été implémentée sur la [V0.4 de notre projet AngularTypescriptStarer](https://github.com/3IE/TypescriptAngularStarter/releases/tag/v0.4) que je vous invite à consulter pour en savoir plus.

# Conclusion

Ce type de scénario peut facilement s'appliquer à des projets dans d'autres languages, Travis et Pivotal supportant bon nombre d'environnements.

Vous pouvez télécharger un exemple fonctionnel dans la [V0.3 de notre projet AngularTypescriptStarer](https://github.com/3IE/TypescriptAngularStarter/releases/tag/v0.3)

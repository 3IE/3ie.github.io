---
title: "Un peu de 3D dans une application AngularJS"
date: "2016-04-15"
categories: 
  - "technical"
tags: 
  - "3d"
  - "angularjs"
  - "babylonjs"
  - "curl"
  - "philips-hue"
  - "typescript"
---

Les interfaces 3D deviennent de plus en plus présentes sur nos projets. Tout d'abord réservées aux jeux, ces interfaces sont de plus en plus utilisées au sein d'applications ou de serious games. Nous dépassons donc le cadre des jeux et nous devons coupler ces nouvelles interfaces et ces nouveaux usages avec de plus en plus de logique métier. Dans cet exemple, nous utiliserons BabylonJS et AngularJS afin de créer une interface 3D permettant d'allumer nos lampes Philips HUE et d'avoir une interaction avec l'application AngularJS.

# La stack technique

Notre application repose sur le serveur que nous avons développé dans la première partie. Vous pouvez retrouver également le code sur notre [Github](https://github.com/3IE/demo-client-hue/tree/v1.0). Notre logique d'application pourra être embarquée dans l'application AngularJs. L’intérêt ici sera de coupler AngularJS et BabylonJS pour les faire fonctionner de concert. On partira de notre [Starter Typescript AngularJS](https://github.com/3IE/TypescriptAngularStarter). Celui-ci est basé sur la version 1.5 d'AngularJs mais ce que nous présentons ici pourrait fonctionner également avec la version 2.0

Qu'est ce que BabylonJS ? BabylonJS est un framework WebGL initié notamment par deux français (David Catuhe & David Rousset) et maintenant supporté par une communauté. BabylonJs permet d'interagir avec l'éco-système 3D (3dsMax, Blender, ...) grâce à de nombreux plugins. Nous vous invitons à aller voir le site [babylonjs.com](http://www.babylonjs.com/) afin d'avoir une meilleure vision des possibilités de ce framework et de naviguer à travers les différentes démos.

Ci dessous le schéma de la solution que nous allons mettre en oeuvre :

[![Architecture](/assets/images/Architecture-1-1024x532.png)](https://blog.3ie.fr/wp-content/uploads/2016/03/Architecture-1.png)

# Création de la scène

Avant de commencer à utiliser BabylonJs, nous allons créer une scène sous 3ds max. Nous ne ferons pas un cours sur la modélisation dans cet exemple. En 3D, vous avez soit la possibilité de générer l'ensemble de votre scène en code (on appelle cela la génération procédurale) ou d'importer des modèles 3D directement dans votre scène (utiliser les deux méthodes en parallèle permet d'avoir un rendu travaillé sur les objets et d'avoir une scène infinie). La scène dans notre cas représente une lampe que nous allons exporter grâce au plugin d'export.

[![Exemple de scène 3D](/assets/images/SceneSample-259x300.png)](https://blog.3ie.fr/wp-content/uploads/2016/03/SceneSample.png)

 

Avant toute chose il faut [installer le plugin](https://github.com/BabylonJS/Babylon.js/tree/master/Exporters/3ds%20Max) permettant l'export de la scène 3D. Cet export nous permettra d'avoir un fichier _.babylon_ que nous pourrons alors importer dans notre scène WebGL. Le fichier de type _.babylon_ nous permet ici de faire la transition entre le monde BabylonJs et les différents format 3D qui peuvent exister, issus des logiciels de modelage (3ds Max, blender, ...).

Sous 3dsMax, un menu Babylon apparaît lorsque le plugin a bien été installé.

[![Menu Babylon dans 3dsMax](/assets/images/MenuExporterBabylon-300x70.png)](https://blog.3ie.fr/wp-content/uploads/2016/03/MenuExporterBabylon.png)

 

Vous pouvez maintenant exporter votre scène au format BabylonJs. Le fichier _.babylon.manifest,_ qui peut être généré en même temps, permet d'indiquer si vous souhaitez mettre en cache navigateur votre modèle une fois celui-ci chargé une première fois. Ce comportement peut être source de bug quand vous développez car si vous regénérez votre fichier _.babylon,_ il faudra bien penser à nettoyer votre cache navigateur pour voir apparaître vos modifications.

[![Exporter Babylon](/assets/images/ExporterBabylon-1024x696.png)](https://blog.3ie.fr/wp-content/uploads/2016/03/ExporterBabylon.png)

# Création de la directive

Maintenant que votre scène est prête, il faut créer une directive AngularJs permettant d'insérer le canvas pour le rendu de la scène. Dans la philosophie AngularJs la directive existe pour séparer la manipulation du DOM HTML de la logique contenu dans le controller. Comme nous allons devoir créer un canvas dans notre page, nous laisserons donc cette tâche à notre directive. De plus la séparation en directive nous permettra également d'isoler le code gérant notre scène 3D.

Dans le fichier [babylonSurface.Directive.ts](https://github.com/3IE/demo-client-hue/blob/v1.0/app/directives/babylonSurface.directive.ts) nous allons donc créer la directive. La propriété template contiendra notre Canvas dans lequel le rendu BabylonJs s’exécutera. La méthode _unboundLink_ nous servira dans l'étape suivante pour déclarer le moteur BabylonJs. L'interface _IBabylonSurfaceDirectiveScope_ contient la variable _newColor_ pour faire transiter la couleur entre le controller AngularJs et notre scène 3D.

/// <reference path='../../reference.ts'/>

namespace app.directive {
	'use strict';

	interface IBabylonSurfaceDirectiveScope extends ng.IScope {
		newColor: app.models.Light;
	}

	interface IBabylonSurfaceDirectiveAttribute extends ng.IAttributes { }

	export class BabylonSurface implements ng.IDirective {
		constructor() {
			this.template = '<canvas id="renderCanvas" style="width:800px;height:500px"> </canvas>';
			this.link = this.unboundLink.bind(this);
		}

		public template: any;
		public link: ng.IDirectiveLinkFn;

		private canvasRender: HTMLCanvasElement;

		/\* tslint:disable:typedef \*/
		public static Factory() {
			const directive = () => {
				return new BabylonSurface();
			};
			return directive;
		};
		/\* tslint:enable:typedef \*/

		private unboundLink(scope: IBabylonSurfaceDirectiveScope, element: ng.IAugmentedJQuery, attrs: IBabylonSurfaceDirectiveAttribute): void {
			let that: BabylonSurface = this;
			that.canvasRender = element.children()\[0\] as HTMLCanvasElement;			
		}

	}
	angular.module('clilentHUE').directive('BabylonSurface', \[BabylonSurface.Factory()\]);
}

 

Toujours dans cette directive, nous allons déclarer le moteur BabylonJs afin de pouvoir effectuer le rendu de la scène. Nous injecterons également dans la directive notre service ('business.light') permettant de faire le lien avec notre API. Pour pouvoir instancier le moteur BabylonJs (_BABYLON.Engine_), nous devons lui passer en paramètre la surface de rendu, dans notre cas le canvas de la directive _(this.canvasRender_). Pour charger une scène issue de 3dsMax, il faut utiliser la méthode _BABYLON.SceneLoader.Load_ en lui passant en paramètre le fichier \*.babylon. Celle-ci prend également en paramètre une callBack permettant d'être notifiée quand le fichier a fini d'être chargé. Sur une scène plus compliquée, nous pouvons également utiliser l'_[AssetManager](http://doc.babylonjs.com/tutorials/how_to_use_assetsmanager)_.

namespace app.directive {
	'use strict';

	interface IBabylonSurfaceDirectiveScope extends ng.IScope {
		newColor: app.models.Light;
	}

	interface IBabylonSurfaceDirectiveAttribute extends ng.IAttributes { }

	export class BabylonSurface implements ng.IDirective {
		constructor(businessLight: engine.common.business.Light) {
			this.template = '<canvas id="renderCanvas" style="width:800px;height:500px"> </canvas>';
			this.link = this.unboundLink.bind(this);
			this.businessLight = businessLight;
		}

		public template: any;
		public link: ng.IDirectiveLinkFn;

		private businessLight: engine.common.business.Light;
		private canvasRender: HTMLCanvasElement;
		private engine: BABYLON.Engine;

		/\* tslint:disable:typedef \*/
		public static Factory() {
			const directive = (businessLight: engine.common.business.Light) => {
				return new BabylonSurface(businessLight);
			};
			return directive;
		};
		/\* tslint:enable:typedef \*/

		private unboundLink(scope: IBabylonSurfaceDirectiveScope, element: ng.IAugmentedJQuery, attrs: IBabylonSurfaceDirectiveAttribute): void {
			let that: BabylonSurface = this;
			// on récupère l’élément canvas du template
			that.canvasRender = element.children()\[0\] as HTMLCanvasElement;
			// on instancie le moteur avec le canvas
			that.engine = new BABYLON.Engine(this.canvasRender, true);

			// on charge notre scène 3dsMax dans babylonJS
			BABYLON.SceneLoader.Load('', 'lamp.babylon', that.engine, function(scene: BABYLON.Scene): void {
				scene.executeWhenReady(() => {
					scene.activeCamera.attachControl(that.canvasRender);
					// boucle de rendu de notre scéne 3D
					that.engine.runRenderLoop(function(): void {
						scene.render();
					});
				});
			});
		}

	}
	angular.module('clientHUE').directive('BabylonSurface', \['business.light', BabylonSurface.Factory()\]);
}

 

# Création du controller

Le but de cet article est de montrer l'interaction entre l'application AngularJS et la scène 3D. Cette interaction va être assez simple, le principe est de sélectionner une couleur dans l'application AngularJS puis de la transférer à la scène qui l'enverra au travers de l'API jusqu'à la lampe. Nous aurions pu réaliser ce picker directement dans la scène 3D mais nous n'aurions pas pu tester l'interaction entre les deux frameworks.

Dans le controller ([HomeController](https://github.com/3IE/demo-client-hue/blob/v1.0/app/controllers/home.controller.ts)) attaché à la page qui hébergera notre directive, nous définissons le scope AngularJS suivant :

export interface IDebugControllerScope extends ng.IScope {
	/\*\*
	 \* méthode pour faire la transformation du colorPicker vers
	 \* la couleur typée 
	 \*/
	changeColor: () => void;
	/\*\*
	 \* représentation de la couleur typée
	 \*/
	newColor: app.models.Light;
	/\*\*
	 \* variable de la couleur binder sur le colorPicker 
	 \*/
	myColor: string;
}

 

Dans le constructeur du controller nous implémentons la méthode _changeColor_. Celle ci sera appelée pour valider la couleur du picker, la convertir en notre modèle et la transférer à notre directive. (Pour alléger le code nous aurions pu mettre aussi cette transformation directement dans un filter).

this.$scope.changeColor = () => {
	const re: RegExp = /rgba?\\(\\s\*(\\d{1,3})\\s\*,\\s\*(\\d{1,3})\\s\*,\\s\*(\\d{1,3})\\s\*(?:,\\s\*(\\d+(?:\\.\\d+)?)\\s\*)?\\)/;
	const resultatParse: RegExpExecArray = re.exec(this.$scope.myColor);
	that.$scope.newColor.color.r = parseInt(resultatParse\[1\], 10) / 255;
	that.$scope.newColor.color.g = parseInt(resultatParse\[2\], 10) / 255;
	that.$scope.newColor.color.b = parseInt(resultatParse\[3\], 10) / 255;
};

 

La variable this.$scope.myColor est bindée sur un [color picker](http://myplanet.github.io/angular-color-picker/) (package bower angular-color-picker). Grâce à ce color picker nous allons pouvoir changer la couleur de notre lampe.

Ensuite au niveau de la partie html, nous passons notre variable de scope à notre directive :

<color-picker 
	ng-model="myColor" 
	color-picker-format="'rgb'"
	 color-picker-swatch-only="true"
		></color-picker>
<button ng-click="changeColor()">change color</button>
<babylon-surface newColor="newColor"></babylon-surface>

 

Nous pouvons accéder à la variable _newColor_ dans la directive car nous l'avons déclarée dans le scope de celle-ci lors de sa création :

interface IBabylonSurfaceDirectiveScope extends ng.IScope {
	newColor: app.models.Light;
}

 

A l'heure actuelle nous pouvons faire passer la couleur de notre controller AngularJS à notre directive (et donc changer la couleur dans notre scène 3d) cependant il nous manque encore la liaison vers la lampe physique. Pour réaliser cette logique nous allons architecturer un peu notre code en séparant la déclaration du moteur et la logique d'affichage de la scène en la mettant dans une nouvelle classe nommée _WorldManager_.

La première étape dans notre directive est donc de passer à notre classe _[WorldManager](https://github.com/3IE/demo-client-hue/blob/v1.0/app/game_engine/WorldManager.ts)_ la variable de scope _newColor_. Celle-ci étant passée par référence nous aurons bien tous les changements d'état  (ex: Changement de couleur à travers le color picker). La classe _[WorldManager](https://github.com/3IE/demo-client-hue/blob/v1.0/app/game_engine/WorldManager.ts)_ exposera deux méthodes createScene (qui permettra d'initialiser notre scène en configurant les interactions avec les objets, les caméras, ...) et la méthode renderLoop (qui mettra à jour notre scène en fonction de l'état de la variable newColor)

private unboundLink(scope: IBabylonSurfaceDirectiveScope, element: ng.IAugmentedJQuery, attrs: IBabylonSurfaceDirectiveAttribute): void {
	let that: BabylonSurfaceNew = this;
	that.canvasRender = element.children()\[0\] as HTMLCanvasElement;
	that.engine = new BABYLON.Engine(this.canvasRender, true);
	let worldManager: app.game\_engine.WorldManager = new app.game\_engine.WorldManager(scope.newColor, that.businessLight);

	window.addEventListener('resize', function(): any {
		this.engine.resize();
	});
        // on charge notre scène 3dsMax dans babylonJS
	BABYLON.SceneLoader.Load('', 'lamp.babylon', that.engine, function(scene: BABYLON.Scene): void {
                // configuration de notre scène
		worldManager.CreateScene(scene);

		scene.executeWhenReady(() => {
			scene.activeCamera.attachControl(that.canvasRender);
			that.engine.runRenderLoop(function(): void {
                                // boucle de rendu
				worldManager.RenderLoop(scene);
			});
		});
	});
}

 

La seconde étape est de compléter la méthode createScene. Comme indiqué plus haut, elle permet de configurer notre scène. La première action est d'éteindre la lumière de la lampe. Puis nous allons rendre pickable (pouvoir interagir avec un mesh et notre souris) l'interrupteur de la lampe. Pour réaliser cela, BabylonJS projète le point d'impact de notre souris  et vérifie s'il y a une collision avec un objet dit pickable. Pour simplifier le code nous utiliserons l'[ActionManager](http://doc.babylonjs.com/tutorials/How_to_use_Actions) de BabylonJS que nous attachons à l'interrupteur. L'ActionManager permet d'attacher une réponse à un event se produisant sur l'objet sur lequel il est attaché. Dans notre cas lors d'un clic gauche sur l'interrupteur, nous allons lancer une animation actionnant l'interrupteur ainsi que la gestion de la lumière avec la méthode ManageLight (savoir si nous devons allumer ou éteindre la lumière de la scène et de quelle couleur nous devons l'afficher)

public createScene(scene: BABYLON.Scene): void {
	let that: WorldManager = this;
	scene.lights\[1\].setEnabled(false);
	scene.meshes.forEach((mesh: BABYLON.Mesh) => {
		mesh.isPickable = false;
		if (mesh.name === 'Box001') {
			// on a trouvé l'interupteur
			mesh.isPickable = true;
			mesh.actionManager = new BABYLON.ActionManager(scene);
			// lorsqu'on déclenche un clic gauche sur l'interrupteur
			mesh.actionManager.registerAction(new BABYLON.ExecuteCodeAction(BABYLON.ActionManager.OnLeftPickTrigger, function(evt: BABYLON.ActionEvent): void {
				// on lance l'animation de basculement de l'interrupteur
				scene.beginAnimation(mesh, 0, 100, false, 10, () => {
					// A la fin de l'animation on active ou non la lumière
					that.activeLight = !that.activeLight;
					that.ManageLight(scene);
				});
			}));
		}
	});
}

# Communication bidirectionnelle

Notre serveur étant capable de pousser de la données vers les clients pour les avertir lors d'un changement d'état de la part d'un autre client, nous allons maintenant doter notre scène 3D de cette fonctionnalité.

Pour gérer la partie WebSocket, nous ajoutons une méthode _realData_ dans la couche d'accès aux données dans le fichier [data.light.ts](https://github.com/3IE/demo-client-hue/blob/v1.0/app_engine/common/data/data.light.ts). Nous allons utiliser la bibliothèque [socket.io](http://socket.io/) disponible à travers [npm socket.io-client](https://www.npmjs.com/package/socket.io-client). Il faut d'abord créer le WebSocket grâce à la méthode _io.connect_ à laquelle il faut passer l'url du serveur créé dans la première partie (Par défaut localhost:3000). Sur la réception du nom de l'event (_eventName_) nous appliquerons la fonction passée en paramètre sous forme de callback.

public realData(eventName: string, callback: Function): void {
	this.socket = io.connect('http://localhost:3000/');
	const that: Light = this;

	this.socket.on(eventName, function() {
		var args = arguments;
		that.$rootScope.$apply(function() {
			callback.apply(that.socket, args);
		});
	});
}

 

Pour utiliser cette communication bidirectionnelle, nous allons ajouter le support de cette méthode dans le constructeur de la classe WorldManager. En paramètre nous lui passerons la callBack à exécuter en réponse à l'event nommé 'stateLight' lorsque celui ci passera à travers la WebSocket.

La Callback permettra de mettre à jour la couleur de la lampe, ou bien de l'éteindre en fonction du message qui sera passé à la WebSocket.

this.businessLight.realData('stateLight', function(data: models.Light): void {
	that.isRemoteChange = true;
	that.newColor.color.r = data.color.r;
	that.newColor.color.g = data.color.g;
	that.newColor.color.b = data.color.b;
	that.newColor.lightId = data.lightId;
	that.newColor.state = data.state;
	that.activeLight = data.state;

});

 

Voici le rendu final de la scène

[![rendu final](/assets/images/render-1024x633.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/render.png)

 

# Conclusion

Grâce à cet exemple nous avons pu mettre en place un client 3D basé sur babylonJS de façon assez simple. Bien que ce framework soit très bien pour les jeux videos, il peut tout a fait être couplé avec une application de type AngularJS afin d'avoir une interaction de l'application vers la scène mais également du serveur d'API vers la scène 3D.

---
title: "Un exemple de projet IoT"
date: "2016-04-06"
categories: 
  - "technical"
tags: 
  - "curl"
  - "iot"
  - "philips-hue"
  - "typefx"
  - "typescript"
---

L'IoT est devenu un Buzzword, mais derrière ce mot se cache du matériel. Nous nous sommes intéressés ici aux lampes Philips HUE. Le modèle que nous utilisons est livré avec une application (sur iPhone) qui permet de changer la couleur et l'ambiance lumineuse. En tant que bidouilleur, nous pouvons détourner l'utilisation de ces lampes pour l'intégrer dans un système d'intégration continue afin d'allumer les lumières en rouge lorsqu'une compilation échoue.

Vous pouvez trouver une version de ces lampes sur [Amazon](http://www.amazon.fr/Philips-Hue-connect%C3%A9es-smartphone-compatible/dp/B016151J3E/ref=sr_1_2?ie=UTF8&qid=1459332870&sr=8-2&keywords=philips+hue) au prix de 80 €. Pour voir l'offre complète vous pouvez également visiter le site de [Philips](http://www2.meethue.com/fr-fr/).

# Description de l'architecture

Les lampes Philips HUE fonctionnent avec un bridge branché sur le réseau (afin d'avoir une adresse IP) et ce dernier communique avec le reste des lampes grâce à un protocole ZigBee.

[![Architecture Philips HUE](/assets/images/Architecture-300x215.png)](/assets/images/Architecture.png)

# Modification de l'architecture

Pour contrôler nos lampes, nous devons soit utiliser l'application de base soit développer la nôtre. Ici nous irons un cran plus loin en développant notre propre serveur (afin de ne pas avoir à utiliser différent SDK pour piloter nos lampes en fonction des technologies). De plus le fait d'utiliser un serveur intermédiaire, nous permettra de piloter nos lampes à travers internet. Cette dernière fonctionnalité n'est pas disponible de base pour des raisons de sécurité bien entendu. (Nous n'aborderons pas ici cette problématique) Du coté matériel, notre serveur sera un Raspberry pi mais vous pouvez le remplacer par toute autre plateforme.

[![Architecture avec le serveur](/assets/images/ArchitectureFinale-300x224.png)](/assets/images/ArchitectureFinale.png)

Ce serveur s’appuie sur un package npm "[node-hue-api](https://www.npmjs.com/package/node-hue-api)": "^2.2.0". Il existe bien d'autres packages mais celui ci répond bien à notre problématique. Afin de gagner du temps, nous allons nous servir de notre framework TypeFx. Celui est basé sur le framework Express, mais apporte le support de Typescript. Vous pouvez avoir une introduction à ce framework [sur notre blog](https://blog.3ie.fr/nos-travaux-autour-de-typescript-sur-nodejs/).

# Implémentation du serveur

Avant de commencer le développement à proprement parler, il nous faut identifier le bridge sur le réseau, pour cela trois solutions :

- regarder sur votre routeur le dernier périphérique qui vient de se connecter
- se connecter directement avec l'adresse suivante : [https://www.meethue.com/api/nupnp](https://www.meethue.com/api/nupnp) cela vous liste l'ensemble des bridges sur votre réseau sous forme de JSON
- utiliser le protocole de découverte du bridge à travers l'api

Pour le dernier point,  vous pouvez utiliser le package npm cité plus haut avec le code suivant :

var hue = require("node-hue-api");

var displayBridges = function(bridge) {
    console.log("Hue Bridges Found: " + JSON.stringify(bridge));
};

// --------------------------
// Using a promise
hue.nupnpSearch().then(displayBridges).done();

L'affichage dans la console doit vous donner une sortie de ce type : Hue Bridges Found: 192.168.2.17

Une fois l'IP trouvée, vous pouvez vous connecter sur l'adresse http://192.168.2.17/debug/clip.html pour déclarer votre 'application' sur le bridge. Sur cette interface vous devez saisir la commande suivante en mode POST tout en appuyant sur le bouton de synchronisation du bridge  :

[![enrollement d'une nouvelle application](/assets/images/RegisterServer-300x228.png)](/assets/images/RegisterServer.png)

 

En réponse vous obtiendrez un GUID qui correspondra à votre username à passer en paramètre pour chaque appel futur sur l'API du bridge.

Maintenant que nous sommes capables d'appeler notre bridge, nous allons pouvoir construire notre API sur notre serveur. Celui-ci exposera une API relativement simple que nous enrichirons par la suite. Vous pouvez retrouver le code complet sur notre [github](https://github.com/3IE/demo-server-hue).

La première étape est d'établir une connexion avec le bridge, pour cela nous allons créer un controller dans notre cas [_LightController_](https://github.com/3IE/demo-server-hue/blob/v1.0/app/controllers/LightController.ts) dans lequel nous rajouterons le code suivant :

private bridgeConnection(): any {
	var HueApi = require("node-hue-api").HueApi;

	return new HueApi(this.hostname, this.username);
}

 

Afin de simplifier la lecture de notre code nous créons une fonction qui construit un _state_.

private createState(): any {
	var lightState = require("node-hue-api").lightState;
	return lightState.create();
}

 

Les states permettrons de changer la couleur ou d'allumer / éteindre nos lampes.

Notre méthode On() sera une méthode en POST qui prendra en paramètre une structure d'objet représentant l'état d'une lampe avec son Id. Cette structure d'objet sera créée dans le répertoire model de notre solution, dans les fichiers [Light.ts](https://github.com/3IE/demo-server-hue/blob/v1.0/app/models/light.ts) et [Color.ts](https://github.com/3IE/demo-server-hue/blob/v1.0/app/models/color.ts).

 class Light{
	public lightId : number;
	public color : Color;
}

class Color{
	public r:number;
	public g:number;
	public b:number;
}

 

L'implémentation de la méthode dans le controller _[LightController](https://github.com/3IE/demo-server-hue/blob/v1.0/app/controllers/LightController.ts)_ sera donc la suivante :

on() {
	var api = this.bridgeConnection();
	var light : Light = <Light> this.request.body;

	api.setLightState(light.lightId, this.createState().on().rgb(light.color.r, light.color.g, light.color.b))
		.then((result) => {
			return this.json({ result: true });
		})
		.fail((error) => {
			return this.json({ result: false });
		})
		.done();
}

# Pour aller plus loin

Notre serveur va être le point central d'une constellation de clients. Afin que chaque client puisse être notifié d'un changement d'état déclenché par un autre client, il faut que nous établissions une communication bidirectionnelle. Pour cela nous allons utiliser les WebSockets à travers la bibliothèque socket.io. Le support de socket.io est encore en beta sur [TypeFx](https://github.com/3IE/typeframework) pour cela nous devons utiliser une version spécifique : _"typefx": "^0.4.0-alpha"_

Il faut initier la communication bidirectionnelle dans le fichier [app.ts](https://github.com/3IE/demo-server-hue/blob/v1.0/app.ts) :

var io = require('socket.io')(app.serverInstance);
// event généré dès qu'un client se connecte
io.on('connection', function(client) {
    console.log('a user connected');
    // on affecte la socket directement dans l'app pour y avoir accès partout
    // a changer lors du prochaine update de typefx
    app.clientSocket = client;
});

 

On stocke dans app la webSocket. Ensuite sur chacune des méthodes où on souhaite notifier un autre client il faut appeler une méthode 'emit'. Dans cet exemple nous compléterons le controller _[LightController](https://github.com/3IE/demo-server-hue/blob/v1.0/app/controllers/LightController.ts)_ :

on() {
    var api = this.bridgeConnection();
    var light: Light = <Light>this.request.body;

    api.setLightState(light.lightId, this.createState().on().rgb(light.color.r, light.color.g, light.color.b))
        .then((result) => {
            
            let stateLight : Light = new Light();
            stateLight.lightId = light.lightId;
            stateLight.state = true;
            stateLight.color = light.color;
            // on recupére les sockets et on emet le message
            app.clientSocket.emit('stateLight', stateLight);
            
            return this.json({ result: true });
        })
        .fail((error) => {
            return this.json({ result: false });
        })
        .done();
}

Maintenant chaque client qui souhaite être notifié doit s'abonner sur l'event 'StateLight'. Nous verrons cela dans les prochains articles.

# Test de la solution

Pour tester notre solution, nous devons compiler notre serveur. Nous pouvons nous servir de la commande _grunt_ (relié au fichier [Gruntfile.js](https://github.com/3IE/demo-server-hue/blob/v1.0/Gruntfile.js)). Cette action aura pour but de générer un fichier app.js dans le répertoire _.build_ de la solution.

PS C:\\Github\\demo-server-hue> grunt
Running "ts:build" (ts) task
Compiling...
Cleared fast compile cache for target: build
Fast compile will not work when --out is specified. Ignoring fast compilation
WARNING: TypeScript does not allow external modules to be concatenated with -
See TypeScript issue #1544 for more details.
Using tsc v1.8.9

TypeScript compilation complete: 3.53s for 1 TypeScript files.

Running "file\_append:app" (file\_append) task

Done, without errors.
PS C:\\Github\\demo-server-hue>

 

Pour lancer notre serveur, nous utiliserons donc le binaire node, par défaut le serveur écoutera sur le port 3000 :

PS C:\\Github\\demo-server-hue> node .\\.build\\app.js
Listening on port: 3000

 

Pour tester nos APIs, nous pouvons allumer/éteindre une lumière à l'aide de [Postman](https://www.getpostman.com/) par exemple :

[![Postman Server Hue](/assets/images/Postman-1024x421.png)](/assets/images/Postman.png)

# Conclusion

Avec très peu de ligne de code (vous pouvez retrouver le code complet sur notre [github](https://github.com/3IE/demo-server-hue)), nous venons de créer une API pour gérer nos lampes en local. Ce serveur va nous être utile pour illustrer l'utilisation de différents clients (interface vocale avec Cortana, Client 3D en WebGL basé sur BabylonJS, utilisation du bracelet Myo) que nous verrons dans les prochaines parties.

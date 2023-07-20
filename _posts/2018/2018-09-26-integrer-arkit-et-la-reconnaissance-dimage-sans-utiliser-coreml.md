---
title: "Intégrer ARKit et la reconnaissance d'image sans utiliser CoreML"
date: "2018-09-26"
categories: 
  - "technical"
tags: 
  - "apple"
  - "arkit"
  - "ios"
  - "swift"
---

En Septembre 2017, Apple lançait ARKit (Augmented Reality Kit), un kit de développement permettant aux développeurs de réaliser des applications à l’aide de la réalité augmentée.

Il faudra au minimum iOS 11 et un iPhone 6S pour utiliser ARKit.

Ce kit a rencontré un grand succès au près des développeurs et du public, et Apple a annoncé ARKit 2.0 en Septembre dernier.

Il est vrai que la réalité augmentée fait fort effet, ce qui rend ce kit aussi populaire, cependant, trouver un usage pertinent à l’AR n’est pas évident, ce qui fait que la majorité des applications d’AR de l’app store sont plus des démonstrations technologiques et visuellement bluffantes que des applications utiles.

Nous nous sommes donc posés la question à 3IE sur comment concevoir une application utile tout en s’appuyant sur la réalité augmentée.

### L’application

Nous avons trouvé notre réponse dans la création d’une application qui serait utilisée lors des JPOs (Journées Portes Ouvertes).

Effectivement, lors d’une JPO, une visite de l’école est effectuée, ce qui est propice à l’affichage d’éléments en réalité augmentée venant enrichir la visite de base, tout en embellissant l’école.

Cette article ne parlera pas de la conception de l’application JPO de manière générale, mais se concentrera sur l’un des modules de cette application qui est la détection d’images clés et l’affichage d’éléments en réalité augmentée suite à cette détection.

Toute l’application est développée en Swift 4.2

### La base technique d’ARKit

ARKit marche sur un système de scènes visuelles qui décrivent l’environnement local.

On peux ajouter à ces scènes des nodes (noeud) 2D grâce à des SKNodes, ou bien des nodes 3D grâce à des SCNodes.

Ces nodes seront donc des images, des objets 3D, du texte, etc, qui apparaîtront dans l’environnement réel une fois ajoutés à la scène principale.

Il est possible (et fréquent) d’imbriquer des scènes dans des scènes pour plus de précision et de contrôle.

### Spécificité de l'application

Pour détecter des objets et ajouter des éléments en réalité augmentée dessus, l'approche généralement proposée est de combiner ARKit et CoreML, qui est le module de machine learning d'Apple.

Cette approche est efficace mais très lourde et sa mise en place dans l'application n'est pas des plus évidente non plus.

Nous avons donc opté pour essayer une nouvelle approche de détection d'images et d'affichage d'éléments AR, qui serait plus légère et plus adaptée à notre cas de figure.

La technique consistant à détecter des images ARKit, grace à des **ARReferenceImages**, puis d'afficher des éléments AR par dessus.

### La détection d’image avec ARKit

La détection d’image avec ARKit est une fonctionnalité additionnelle préexistante du module.

La première étape consiste à importer des images qu’on souhaite détecter.

Dans notre cas, on prendra comme exemple l’entrée d’un amphithéâtre dans la cour de l’école.

[![](/assets/images/IMG_0861-min-300x225.jpg)](/assets/images/IMG_0861-min.jpg)

On peut ensuite dans le code créer une instance de classe **ARReferenceImage** qui va venir charger une ou plusieurs images contenues dans notre catalogue d’assets

guard let referenceImages = ARReferenceImage.referenceImages(inGroupNamed: ressourceFolder, bundle: nil)
else {
            fatalError("Missing expected asset catalog resources")
     }

Une fois cette image créée, on peut exploiter la fonction renderer qui est appelée à chaque fois qu’une image de référence est détectée.

func renderer(\_ renderer: SCNSceneRenderer, didAdd node: SCNNode, for anchor: ARAnchor) {
guard let imageAnchor = anchor as? ARImageAnchor else { return }
let ref = imageAnchor.referenceImage
updateQueue.async {
	//Action à faire lorsqu'une image de référence a été détéctée
}

Ici, nous nous sommes amusés à afficher le nom de l’amphithéâtre en réalité augmentée une fois l’image détectée (nous reparlerons plus en détails de comment faire cela dans le prochain paragraphe).

[![](/assets/images/Screen-Shot-2018-07-24-at-17.28.27-300x138.png)](/assets/images/Screen-Shot-2018-07-24-at-17.28.27.png)

Pour optimiser le résultat, nous avons décidé de prendre de multiples photos de l'amphithéâtre, et d'afficher le titre en AR une fois une de ces images détéctées.

Cette technique est très efficace dans un cas de figure où il y a relativement peu d'images à reconnaitre mais passe mal à grande échelle à cause de la complexité linéaire du process de reconnaissance d'images en fonction du nombre d'images.

Apple précise dans sa documentation qu'il est conseillé de ne pas dépasser 25 images pour garder une performance optimale.

Ici, cette technique atypique est parfaitement adaptée, toutefois, tout dépend du cas de figure.

Il est conseillé d'adopter cette méthode uniquement dans un cadre restreint, comme ici, l'enceinte d'une école, ou pour une salle de musée par exemple.

On peux retrouver un tutoriel complet mis en place par Apple qui détaille la conception d’un système de reconnaissance d’images [ici.](https://developer.apple.com/documentation/arkit/recognizing_images_in_an_ar_experience)

### Ajouter du contenu en réalité augmentée

Une fois qu’on a détecté une surface qui nous intéresse, on peut ajouter toute une variété d’éléments à l’écran : des objets 3D, du texte 3D, du text 2D et des images.

Ici nous allons nous intéresser à l’ajout d’un object 3D qui sera un XWing (vaisseau spatial présent dans Star Wars)

[![](/assets/images/Screen-Shot-2018-07-12-at-12.24.43-212x300.png)](/assets/images/Screen-Shot-2018-07-12-at-12.24.43.png)

Pour ce faire, nous allons créer une fonction **createSCObjectWithVector** qui prend en paramètre le nom de l’objet 3D, le node à afficher dans l’objet 3D, l’ARSCNView sur laquelle on affichera notre object 3D, et un vecteur pour orienter notre objet.

func createSCObjectWithVector(name: String, rootname: String, sceneView: ARSCNView, vect: SCNVector3) {
        //Ici, nous créons une SCNScene à partir de notre objet 3D.
        guard let paperPlaneScene = SCNScene(named: name),
        //On crée ensuite une SCNode à partir de la SCNScene précédente, qui va venir charger l’objet principal de la scène 
        let paperPlaneNode = paperPlaneScene.rootNode.childNode(withName rootname,
        recursively: true)
        else {
           return
        }
        //Finalement, on positionne correctement notre SCNode grâce au vecteur passé en paramètre.
        paperPlaneNode.position = vect
        //On appelle la fonction animateObject afin d’animer notre object si nécessaire.
        animateObject(object: paperPlaneNode, objectName: rootname)
        //Finalement, on ajoute notre node à notre SceneView passé en paramètre !
        sceneView.scene.rootNode.addChildNode(paperPlaneNode)
    }

 

En combinant la détection d’images et l’ajout d’objet 3D, on peut alors commencer à créer des scènes dynamiques et sympathiques dans un environnement quelconque.

Par exemple sur ces 2 gifs, on se trouve dans la cour de l'école Epita et en fonction des bâtiments détectés, différent types d'animations et d'éléments AR sont ajoutés.

[Lien 1](https://media.giphy.com/media/2j07MIdRQ6zZM8Jygz/giphy.gif)

[Lien 2](https://media.giphy.com/media/9GIS1n5ySXi6RgtGdO/giphy.gif)

### Structurer le projet

Maintenant que nous pouvons détecter des patterns prédéfinis et que nous savons comment ajouter des éléments 3D par dessus, une bonne structure du code est primordiale !

C’est pour cela que j’ai découpé le code de cette manière :

- Une classe "NodeHandler" s’occupant de l’affichage de nodes (objets 3D, image 2D, etc) dans la scène
- Une classe "ViewController" s’occupant de la détection d’images
- Une classe scénario "ScenaryHandler" ayant pour but de construire un scénario prédéfini pour chaque environnement. Des classes héritées de la classe scénario permettent donc de faire plusieurs scripts en fonction de l’environnement dans lequel on se trouve.

L’avantage de cette structure de projet est double, tout d’abord, une fois créée, la classe de détection d’objets et la classe de reconnaissance d’images n’ont jamais à être modifiées.

De plus, on peut créer des scripts à la volée à partir de la classe scénario sans avoir à se préoccuper du fonctionnement de l’application.

Plus de détails sur les autres fichiers peuvent être trouvés directement sur le README du repo Github.

[![](/assets/images/Screen-Shot-2018-07-24-at-17.47.04-206x300.png)](/assets/images/Screen-Shot-2018-07-24-at-17.47.04.png)

### Conclusion

Avec une utilisation adaptée d’ARKit, une application de réalité augmentée a vraiment la capacité de rajouter de la magie à un environnement anodin !

Notre technique atypique et novatrice nous permet d'obtenir un projet très rapidement adaptable, léger et fluide !

Ceci dit, cette technique présente également des limites et ne doit pas être appliquée partout.

Finalement, Apple frappe encore très fort avec ce module relativement simple à prendre en main et tout à fait optimisé pour des appareils iOS.

Il est possible de retrouver le code complet du projet sur [ce repo github](https://github.com/3IE/JPO-ImageDetection).

### Références

- [ARKit and Augmented Reality](https://www.raywenderlich.com/172543/augmented-reality-and-arkit-tutorial)
- [ARKit by Apple](https://developer.apple.com/documentation/arkit)
- [Recognising images in AR Experiences by Apple](https://developer.apple.com/documentation/arkit/recognizing_images_in_an_ar_experience)
- [For 3D Models](https://poly.google.com)

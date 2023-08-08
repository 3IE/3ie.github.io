---
title: "Lire sur les lèvres depuis un iPhone, avec CoreML et ARKit"
date: "2018-09-11"
categories: 
  - "technical"
tags: 
  - "arkit"
  - "core-ml"
  - "create-ml"
---

Voyant qu'Xcode 10 arrive avec la possibilité d’entraîner facilement des modèles de Machine Learning, je me suis demandé s'il était possible d'utiliser les données de face tracking retournées par ARKit pour lire sur les lèvres. Je me suis fixé comme objectif de rester dans l'éco-système d'Apple, je n'utiliserai donc pas d'outils externes comme TensorFlow.

En premier lieu nous allons regarder ce qu'Apple nous fournit comme données liées au visage. Ensuite nous allons regarder comment reconnaître une voyelle (une expression à un moment donné) et enfin comment reconnaître un mot (une suite d'expressions).

Vu que nous allons couvrir beaucoup de sujets, l'article ne détaillera pas toutes les étapes mais vous pouvez retrouver le code complet sur le github de l'application de [reconnaissance](https://github.com/3IE/WordRecognition-iOS) sur iPhone et celui de l'application d'[entrainement des modèles](https://github.com/3IE/WordRecognitionModelTraining-MacOS) sur MacOS.

# Le face tracking d'ARKit

Apple utilise ARKit pour ses Animoji malheureusement ça ne fonctionne que sur iPhone X. L'analyse se fait environ 60 fois par secondes. Apple retourne une liste de [_blend shapes_](https://developer.apple.com/documentation/arkit/arfaceanchor/blendshapelocation) correspondant aux traits du visage, avec un coefficient variant entre 0 (pour un trait ne s'exprimant pas du tout) et 1(pour un trait s'exprimant complètement). Ex : la mâchoire ouverte ([jawOpen](https://developer.apple.com/documentation/arkit/arfaceanchor/blendshapelocation/2928236-jawopen))

[![](/assets/images/ed3838e6-65fa-4be2-8e86-638d97a3b2b2-300x215.png)](/assets/images/ed3838e6-65fa-4be2-8e86-638d97a3b2b2.png)

 

Pour mieux visualiser les variations de valeurs et comprendre ce qu'il est intéressant d'apprendre, nous allons d'abord réaliser une page de statistiques permettant d'afficher la valeur des _blend shapes_. En effet, pour faire un bon apprentissage , il est important de sélectionner les _features_ les plus pertinentes pour aider le modèle prédictif à se construire; c'est ce qu'on appelle le [_feature selection_](https://fr.wikipedia.org/wiki/Sélection_de_caractéristique).

Voilà le type de vue que l'on veut obtenir :

[![](/assets/images/facestats-473x1024.png)](/assets/images/facestats.png)

### Initialisation du face tracking

Dans votre storyboard, ajoutez un ViewController dans lequel vous placerez un SCNView et une tableView. Coté code, ajoutez un ViewController (nommez le FaceStatsTVC), en ajoutant les outlets et les delegates nécessaires, notamment le delegate pour la SCNView. Vous devez donc avoir la déclaration suivante :

FaceStatsTVC: UIViewController, UITableViewDelegate, ARSCNViewDelegate, UITableViewDataSource

 

La fonction suivante va initialiser le tracking. Il faudra l'appeler depuis la méthode "viewWillAppear" dans votre [FaceStatsTVC.swift](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/FaceStatsTVC.swift)

```swift
func setupTracking() {
	guard ARFaceTrackingConfiguration.isSupported else { return }
	let configuration = ARFaceTrackingConfiguration()
	configuration.isLightEstimationEnabled = false
	session.run(configuration, options: \[.resetTracking, .removeExistingAnchors\])
}
```

Nous n'avons pas besoin d'avoir d'estimation de la lumière car nous n'allons pas afficher d'informations en réalité augmentée sur le visage de l'utilisateur (ce qui est normalement prévu par Apple vu que ces fonctions font parties de ARKit).

### Récupération des blend shapes

La fonction renderer(\_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor)  est appelée à chaque fois que les valeurs du visage sont mises à jour et c'est à partir d'ici que l'on pourra récupérer les 'blend shapes' (pour rappel, les traits du visage).

Comme nous voulons lire sur les lèvres, intuitivement on sait que l'on peux se concentrer sur la région de la bouche. J'ai donc décidé de ne garder que les traits de la bouche, de la mâchoire et des pommettes. On enregistre aussi les valeurs min et max pour faciliter l'interprétation ensuite. Code à ajouter dans [FaceStatsTVC.swift](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/FaceStatsTVC.swift) :

```swift
func renderer(\_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
	guard let faceAnchor = anchor as? ARFaceAnchor else { return }
	let regionsToDisplay = \["mouth", "jaw", "cheek"\]
	//since apple used an enum, we can use the raw value to get the name of the blend shape and filter it based on what we want to keep
	blendShapes = faceAnchor.blendShapes.filter { regionsToDisplay.contains(where: $0.key.rawValue.contains)  }
	//we initialize our dictionaries of min and max values if it's the first time we get an expression
	if (sortedShapeLocations.count == 0) {
		sortedShapeLocations = blendShapes.keys.sorted{ $0.rawValue > $1.rawValue }
		blendShapesMinValue = blendShapes
		blendShapesMaxValue = blendShapes
	}
	else {
		for elt in blendShapes {
			guard let maxValue = blendShapesMaxValue\[elt.key\] else { continue }
			guard let minValue = blendShapesMinValue\[elt.key\] else { continue }
			if (elt.value.floatValue > maxValue.floatValue) { blendShapesMaxValue\[elt.key\] = elt.value }
			if (elt.value.floatValue < minValue.floatValue) { blendShapesMinValue\[elt.key\] = elt.value }
		}
	}
}
```

Vous pouvez afficher les valeurs dans votre tableView avec le formatage de votre choix.

### Analyse des valeurs et avis sur le Face Tracking

Il suffit de parler quelques dizaine de secondes pour avoir des valeurs à analyser.

On remarque que plusieurs critères ne nous intéressent pas:

- les mouvements des pommettes (tout ce qui préfixé par 'cheek') sont minimes et beaucoup plus liés au sourire qu'à la parole
- les mouvements latéraux de de la mâchoire ('jawLeft' et 'jawRight') et de la bouche ('mouthLeft' et 'mouthRight')
- les traits sont détectés à gauche et à droite, mais l'information est redondante, on peut prendre la moyenne des 2 valeurs (et dans le cas d'une paralysie faciale on pourrait prendre la valeur max).

Il est important de noter que si la détection d'Apple marche relativement bien, elle demande une lumière suffisante et relativement douce pour avoir des valeurs stables et cohérentes. En effet, même si Apple initialise le système avec la caméra True Depth, c'est ensuite une analyse d'images qui est effectuée, une lumière dure creusera ainsi vos traits et accentuera certaines expressions. Pour vous en convaincre, prenez une expression fixe et faites variez la lumière autour de vous, en bougeant la tête ou en vous déplaçant prêt d'une fenêtre, vous verrez que les valeurs oscilleront.

Pour l'enregistrement de vos expressions, il faudra donc penser à variez les prises de vue et la lumière, en tournant/penchant légèrement la tête pour bien capter les variations.

# Reconnaissance d'une voyelle

Ici nous utiliserons les voyelles 'a', 'e', 'i', 'o', 'u', mais pas le 'y'. En effet, ce dernier étant une demi-voyelle, sa prononciation change suivant le mot, ex 'c**y**cle' ou '**y**eux', on ne peut donc pas le reconnaître avec un simple instantané du visage. En plus nous utiliserons un "neutre" qui permettra d'avoir une réponse pertinente quand l'utilisateur ne prononce aucune voyelle.

La méthodologie que je vais présenter ici demande d'avoir Xcode 10 et d'être sous macOS Mojave.

### Récolte des données pour le machine learning

Avant de pouvoir reconnaitre une voyelle, il va falloir entraîner notre modèle et par conséquent enregistrer des expressions pour avoir des exemples sur lesquelles l'entraîner.

##### Sauvegarde des exemples

J'ai choisi de stocker les différents enregistrements du visage dans un fichier json pour ensuite entraîner mon modèle depuis Xcode. Voila le format que nous allons utiliser :

```js
\[
  {
    "mouthFrown" : 0.025898713676724583,
    "mouthLowerDown" : 0.21620424091815948,
    "mouthFunnel" : 0.15177972614765167,
    "mouthRollUpper" : 0.080305777490139008,
    "mouthShrugLower" : 0.037564069032669067,
    "mouthClose" : 0.085225477814674377,
    "mouthDimple" : 0.022052989341318607,
    "mouthPucker" : 0.17479763925075531,
    "jawOpen" : 0.14299187064170837,
    "mouthUpperUp" : 0.036568604409694672,
    "mouthRollLower" : 0.021438051015138626,
    "mouthPress" : 0.024457765743136406,
    "mouthSmile" : 0.031267983147676426,
    "mouthShrugUpper" : 0.14486999809741974,
    "vowel" : "a",
    "mouthStretch" : 0.12150635197758675
  }
\]
```

 

Créez un nouveau ViewController nommé "RecordVowelVC", ajoutez un ViewController dans le storyboard et placez-y un composant SCNView que vous connecterez et initialiserez comme précédemment. En plus, ajoutez un PickerView avec les différentes voyelles (et le neutre) ainsi qui bouton "record" pour enregistrer votre expression.

Notre fonction "renderer" va cette fois simplement stocker les 'blend shapes' qu'Apple nous fournit et c'est notre bouton qui va déclencher l'enregistrement. Nous avons donc cette méthode dans le fichier [RecordVowelVC.swift](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/RecordVowelVC.swift) :

```swift
func renderer(\_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
	guard let faceAnchor = anchor as? ARFaceAnchor else { return }
	latestBlendShapes = faceAnchor.blendShapes
	if (learningMode == .recognize) {
		detectVowel(latestBlendShapes)
	}
}
```

 

Toujours dans [RecordVowelVC.swift](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/RecordVowelVC.swift), ajoutez l'IBAction de votre "record" button

```swift
@IBAction func recordFaceAction(\_ sender: Any) {
	var encodablesShapes: \[String: Encodable\] = FaceProcessing.simplifyRecord(latestBlendShapes)
	encodablesShapes\["vowel"\] = selectedVowel
	recordedExpressions.append(encodablesShapes)
	
	do {
		let jsonData = try JSONSerialization.data(withJSONObject: recordedExpressions, options: .prettyPrinted)
		if let str = String(data: jsonData, encoding: .utf8) {
			let filename = "vowelTrainingData.json"
			let fileUrl = Helper.getDocumentsDirectory().appendingPathComponent(filename)
			try str.write(to: fileUrl, atomically: true, encoding: String.Encoding.utf8)
		}
		Helper.displayFlashSubview(inView: self.view, withDuration: 0.3)
	}
	catch {
		print("unable to save json")
	}
}
```

J'ai implémenté une méthode de simplification de mes 'blend shapes' que j'ai déclarée dans une classe [FaceProcessing](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/BusinessLogic/FaceProcessing.swift). Elle retire les zones qui ne m'intéressent pas, récupère le nom du trait, fusionne les valeurs gauche et droit, et me donne un dictionnaire prêt à être transformé en json. Il ne me reste qu'à ajouter la voyelle sélectionnée avant d'écrire le json.

### Apprentissage du model

Nous laissons un instant de coté l'application iOS et passons sous mac. Nous allons utiliser les outils du framework [CreateML](https://developer.apple.com/documentation/createml/), qui est une surcouche de [CoreML](https://developer.apple.com/documentation/coreml) et qui est arrivé avec macOS 10.14 Mojave.

Commencons par créer un 'playground' dans lequel nous allons importer les données et entraîner notre modèle.

##### Récupération du json sous macOS

Le plus simple est d'activer le partage de fichiers avec iTunes. Pour cela il faut rajouter la clé UIFileSharingEnabled dans votre fichier plist, comme spécifié dans la doc Apple sur les [clés de plist iOS](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Articles/iPhoneOSKeys.html). Vous pouvez ensuite copier/coller le fichier de l'iPhone vers votre mac.

##### Importation des données

Le formalisme de données choisi est semblable à un tableau de données dans un tableur. Notre json contient un tableau d'objets. Chacun de ces objets correspond à une ligne du tableau. Les attributs des objets (les traits de notre visage) correspondent aux colonnes du tableau, à l'exception de la 'vowel' qui sera ce que notre modèle doit prédire.

[![](/assets/images/f94a686b-8ea3-460d-a1b2-224d6481e2bd-1024x351.png)](/assets/images/f94a686b-8ea3-460d-a1b2-224d6481e2bd.png)

De cette manière nous allons pouvoir utiliser un [MLDataTable](https://developer.apple.com/documentation/createml/mldatatable) pour importer les données et les découper en "données d'apprentissage" (pour entraîner le modèle) et "données de test" (pour tester le modèle sur un jeu de données complètement indépendant).

##### Entraînement

En machine learning, vouloir déterminer une catégorie parmi un ensemble fini (nos 5 voyelles et le neutre) en se basant sur des données d'entrée est une tache d'apprentissage supervisé, et plus précisément de classification. Nous allons pouvoir utiliser les [MLClassifier](https://developer.apple.com/documentation/createml/mlclassifier) qui vont entraîner un modèle avec les données précédemment importées.

MLClassifier va essayer plusieurs algorithmes de classifications et va garder celui qui a le meilleur taux de réussite, basé sur un jeu de données de validation choisies au hasard parmi les données d'apprentissage. Après avoir testé, j'ai vu qu'Apple choisissait un boosted tree classifier, j'ai donc directement choisi cet algorithme dans mon code ce qui me permet de définir des paramètres en plus, notamment le nombre d'itérations maximum qui a un impact direct sur la qualité du modèle généré.

##### Le code

Votre trouverez sur github le json avec mes données ainsi que le modèle "VowelOnFace.mlmodel" basé sur ces données. Il est basé sur mon visage mais il marche plutôt bien sur un visage d'homme. Je recommande quand même d'entraîner avec vos données.

Ajoutez le code suivant dans votre [playground](https://github.com/3IE/WordRecognitionModelTraining-MacOS/blob/v1.0/wordRecoTraining.playground/Contents.swift), il affichera aussi des stats en fin d'entraînement :

```swift
func vowelTraining() {
    do {
        let data = try MLDataTable(contentsOf: URL(fileURLWithPath: "/Users/verdie\_b/Desktop/coreml/vowelTraining/vowelTrainingData-20180823-17h50.json"))
		let seed = Int((Date().timeIntervalSince1970 - Date().timeIntervalSince1970.rounded()) \* 1000)
        let (trainingData, testingData) = data.randomSplit(by: 0.90, seed: seed)
        
        //let reco = try MLClassifier(trainingData: trainingData, targetColumn: "vowel")
        let boostedTreeParams = MLBoostedTreeClassifier.ModelParameters(maxIterations: 40)
        let reco = try MLBoostedTreeClassifier(trainingData: trainingData, targetColumn: "vowel", featureColumns: nil, parameters: boostedTreeParams)
        
        let evaluationMetrics = reco.evaluation(on: testingData)
        print("Classification error = \\(evaluationMetrics.classificationError)")
        print("Confusion: \\(evaluationMetrics.confusion)")
        //
        let metadata = MLModelMetadata(author: "Benoit Verdier", shortDescription: "Vowel reco on face expression", version: "1.0")
        let url = URL(fileURLWithPath: "/Users/verdie\_b/Desktop/coreml/vowelTraining/VowelOnFace.mlmodel")
        try reco.write(to: url, metadata: metadata)
    }
    catch {
        print("unable to generate")
    }
}
```

### Jusqu'où entraîner un modèle ?

Il est important de ne pas trop entraîner le modèle, cela s'appelle du [surapprentissage](https://fr.wikipedia.org/wiki/Surapprentissage), et ça correspond à un modèle tellement entraîné qu'il reconnait parfaitement les données sur lesquelles il s'est entraîné, mais qui a du mal à reconnaître les autres données. On peut surveiller que le taux d'erreur de classification en fin d'apprentissage n'est pas trop éloigné du taux de validation (qui est, pour rappel, basé sur une partie des données d'apprentissage).

### Prédiction des voyelles

Maintenant que nous avons généré un modèle, nous allons pouvoir l'utiliser et afficher notre prédiction à l'utilisateur.

##### Interprétation simple des résultats

Comme précédemment, il nous faut à nouveau un controller qui utilise une SCNView, vous pouvez soit reprendre votre "RecordViewVC" et lui donner la possibilité de basculer entre apprentissage et reconnaissance, soit créer un nouveau controller.

Pour importer votre modèle "VowelOnFace.mlmodel" dans le projet, il suffit de faire un glisser/déposer dans Xcode. Automatiquement, Apple va générer une classe "VowelOnFace" qui nous permettra de lancer notre reconnaissance, et une classe "VowelOnFaceInput" qui contiendra les données sur lesquelles lancer la prédiction.

Déclarez votre modèle dans la classe "RecordVowelVC" : let vowelModel = VowelOnFace()

La détection en elle même est assez simple, il faut juste appeler la méthode "prediction" de notre modèle. Dans notre implémentation la plus naïve, nous allons simplement afficher les probabilités à l'utilisateur, sans chercher à lisser la valeur.

Pour créer une instance de 'VowelOnFaceInput', j'ai rajouté une extension à la classe générée; elle ajoute un constructeur qui prend en paramètre les blend shapes retournées lors du tracking du visage. Vous pouvez retrouver ce code dans le fichier "FaceInputExtensions.swift".

Maintenant, ajoutez la méthode suivante dans la classe [RecordVowelVC](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/RecordVowelVC.swift) pour lancer la prédiction :

```swift
func detectVowel(\_ blendshapes: \[ARFaceAnchor.BlendShapeLocation: NSNumber\]) {
	let probabilities: \[String:Double\]
	do {
		guard let input = VowelOnFaceInput(blendshapes: blendshapes) else { return }
		let predictions = try vowelModel.prediction(input: input)
		probabilities = predictions.vowelProbability
	}
	catch {
		print("prediction failure")
		return
	}
	
	// TODO : display in your UI your probabilities on the main thread
}
```

 

Nous voulons lancer la détection de voyelle à chaque update du visage, il faut donc appeler notre méthode dans la fonction "renderer" de [RecordVowelVC](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/RecordVowelVC.swift) :

```swift
func renderer(\_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
	guard let faceAnchor = anchor as? ARFaceAnchor else { return }
	latestBlendShapes = faceAnchor.blendShapes
	if (learningMode == .recognize) {
		detectVowel(latestBlendShapes)
	}
}
```

##### Amélioration de la prédiction

L'affichage direct de la prédiction était la première étape pour vérifier que notre modèle fonctionne bien. On peut améliorer la qualité de notre prédiction en lissant les valeurs sur une douzaine d'échantillons (cela représente environ 200ms à raison de 60 échantillons/sec). On peut ensuite faire la moyenne des prédictions mais le mieux est de coefficienter les valeurs de prédictions suivant leur âge pour réaliser une [moyenne mobile](https://fr.wikipedia.org/wiki/Moyenne_mobile).

J'ai créé une classe [HistorizedProbabilities](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/BusinessLogic/HistorizedProbabilities.swift) qui va stocker les N dernières prédictions dans un tableau (pos=0 pour la plus vieille et pos=N-1 pour la plus récente) et qui peut ensuite me fournir les probabilités moyennes pour les voyelles.

Je calcule un coefficient compris entre 0 et 1 qui me donne une importance proportionnelle à l'âge : coeff = (1 + pos) / N Ensuite j'élève ce coefficient à la puissance 0.3 ce qui a pour effet de donner plus d'importance aux valeurs récentes, comme on peut le voir sur le graphique de 'x^0.3' :

\[caption id="attachment\_2077" align="aligncenter" width="300"\][![](/assets/images/Screenshot-2018-09-06-at-18.41.17-300x279.png)](/assets/images/Screenshot-2018-09-06-at-18.41.17.png) x^0.3\[/caption\]

Vous pouvez mettre à jour votre méthode DetectVowel en ajouter le code suivant après calcul des probabilités :

```swift
vowelHistory.appendNewProbability(probabilities)
if let best = vowelHistory.averagedSortedDesc.first {
	DispatchQueue.main.async {
		//TODO display best.key and best.value
	}
}
```

# Reconnaissance d'un mot

### Format des données à reconnaître

Nous avons ici un signal continu que nous souhaitons reconnaître. Le transformation en signal discret est fait par le téléphone et ARKit : on reçoit les blend shapes à intervalle régulier, environ 60 fois par second.

##### Approche 1 : concaténation

On pourrait ajouter en postfix le numéro de l'échantillon, pour obtenir un json de ce type :

```js
\[
  {
    "mouthFrown\_t1" : 0.025898713676724583,
    "mouthFrown\_t2" : 0.21620424091815948,
    "mouthFunnel\_t1" : 0.15177972614765167,
    "mouthFunnel\_t2" : 0.15984314654
    //and so on
  }
\]
```

Mais cette approche n'est pas valable car les algorithmes de machine learning ont besoin d'un nombre constant de 'feature' à analyser, or la durée d'un mot est variable.

On pourrait tricher en ajoutant des données en padding, pour toujours avoir N échantillons, mais de toute façon on se retrouverait à créer un 'input' avec nos N échantillons \* M traits. En enregistrant pendant 2 sec avec les 15 traits du visage, on se retrouverait à devoir appeler notre constructeur avec 1800 paramètres, ce qui est complètement délirant.

##### Approche 2 : extraction

On pourrait utiliser une approche de 'feature extractation', c'est à dire un reformatage des données. L'idée est de stocker pour chaque trait du visage un 'array' de valeurs correspondant à son historique. Cette approche est crédible car Apple précise dans sa documentation que les [MLDataValue peuvent stocker des Array](https://developer.apple.com/documentation/createml/mldatavalue) (à un détail prêt, il faut des tableaux d'entiers, pas de nombres flottants). On aurait alors le json suivant :

```js
\[
  {
    "mouthFrown" : \[123, 201, 254\]
    "mouthFunnel" : \[51, 36, 32\]
    //and so on
  }
\]
```

Malheureusement, même si l'import des données par MLDataTable fonctionne, que le classifier s'entraîne bien, Xcode ne sait pas l'exporter. On récupère l'erreur suivante "Only string, numerical, or dictionary types allowed in exported model."

##### Approche 3 : image

La dernière approche, qui est celle que nous allons ensuite détailler, est de transformer notre enregistrement en une image avec niveau de gris, sur le principe des spectrogrammes. Chaque ligne correspondra à une expression à un moment donné, et chaque pixel stocke une valeur d'un 'blend shape' (pixel blanc pour la valeur max, pixel noir pour la valeur min). On utilisera ensuite des algorithmes de reconnaissance d'images à la place de notre 'classifier'.

Pour le mot "maison", on obtient l'image suivante (qui a été grossie pour l'affichage, elle fait en réalité 15x29 pixel) :

[![](/assets/images/maison-spectro-155x300.png)](/assets/images/maison-spectro.png)

### Quand enregistrer ?

Pour avoir un apprentissage et une reconnaissance de qualité, il est important d'avoir des données les plus cohérentes et constantes possibles. On pourrait déclencher l'enregistrement manuellement avec un bouton "record" mais c'est difficile d'être constant, on risque d'être décalé par rapport au mot. Pour cette raison, j'ai décidé de détourner un peu ma détection de voyelle et de la transformer en détection de neutre. On peut reprendre les enregistrements existants mais il faut capturer d'autres expressions pour bien différencier le neutre du reste.

Je vous recommande de bien enregistrer de nombreux échantillons sur des syllabes comme "vo-va-vi-vu", "fo-fa-fi-fu", "cho-cha-chi-chu", "so-sa-si-su", "to-ta-ti-tu", etc... . En prononçant lentement, on peut mitrailler le bouton d'enregistrement pour bien capter les différentes étapes du son. Il faut par contre éviter les "m" et "p" car on passe par une étape où la bouche est fermée, ce qui se rapproche trop de la position neutre que l'on veut détecter.

Comme on n'est lié à aucun son en particulier, on peut nommer nos 2 catégories "neutral" et "something". Une fois vos enregistrements finis, générez votre modèle sous MacOS, sauvegardez le dans un fichier "NeutralFace.mlmodel", et ajoutez le à votre projet iOS.

### Enregistrement du mot

##### Storyboard

Créez un controller dans votre storyboard et un [RecordWordVC](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/RecordWordVC.swift) dans votre code puis connectez un SCNView, comme précédemment. Cette fois ci, le pickerView nous permettra de choisir le mot à apprendre. J'ai décidé d'ajouter un 2ème pickerView pour sélectionner l'orateur, une imageView pour afficher mon 'wordImage' (le spectrogramme de mon mot), un label que j'affiche quand je détecte que l'utilisateur parle, un "performance counter", un switch pour désactiver l'enregistrement, un switch pour activer une alertView.

Voila le controller que j'ai dans mon storyboard :[![](/assets/images/Screenshot-2018-09-07-at-16.10.04-453x1024.png)](/assets/images/Screenshot-2018-09-07-at-16.10.04.png)

##### Détection de la parole

Nous allons utiliser la détection de neutre avec la classe [HistorizedProbabilities](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/BusinessLogic/HistorizedProbabilities.swift) pour lisser les valeurs, ce qui évite qu'une valeur parasite n'ait trop d'impact, mais il reste toujours le risque que l'on bascule de manière un peu erratique entre l'envie de lancer l'enregistrement et celle de l'arrêter. Cela peut se produire quand les probabilités tournent autour de 50%. Pour éviter ça, on peut introduire une notion de palier de confiance à atteindre pour prendre une décision, de la même manière qu'une boite de vitesse auto qui vient de passer au rapport supérieur ne va pas revenir au rapport inférieur parce qu'on lève légèrement le pied.

L'enregistrement des expressions et la prise de décision ont été regroupés dans une classe "WordImage". Pour prendre la décision d'enregistrer je vérifie si la probabilité de "something" est 20% supérieur à celle de "neutral", et pour arrêter, je demande 40% de plus. Ce code se trouve dans le fichier [WordImage.swift](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/BusinessLogic/WordImage.swift) :

```swift
func predictNeedToRecord(isCurrentlyRecording: Bool) -> Bool {
	guard neutralHistory.history.count > 0 else {
		print("unable to predict with an empty history")
		return false
	}
	let averagedProba = neutralHistory.averagedSortedDesc
	if let bestProba = averagedProba.first, let secondProba = averagedProba.last {
		let shouldRecordNow = (bestProba.key != "neutral")
		let certainty = bestProba.value / secondProba.value
		// we only take into account the recording decision if we are above a certain threshold
		if (shouldRecordNow != isCurrentlyRecording && (shouldRecordNow && certainty > WordImage.kNeedToRecordMeaningThreshold || shouldRecordNow == false && certainty > WordImage.kNeedToRecordNeutralThreshold)) {
			return shouldRecordNow
		}
	}
	return isCurrentlyRecording
}
```

 

L'appel à cette méthode est encapsulé dans la méthode recordOnNeutralDetection(blendShapes:)  qui coordonne la prédiction du neutre et la prise de décision d'enregistrer. C'est cette méthode qui est appelée à chaque mise à jour du 'face tracking'. Ce code est dans le fichier [WordImage.swift](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/BusinessLogic/WordImage.swift) :

```swift
func recordOnNeutralDetection(blendShapes: \[ARFaceAnchor.BlendShapeLocation : NSNumber\]) {
	neutralPredictionQueue.async {
		guard let input = NeutralFaceInput(blendshapes: blendShapes) else { return }
		let prediction: NeutralFaceOutput
		do { prediction = try self.mlNeutralModel.prediction(input: input) }
		catch { return }
		self.neutralHistory.appendNewProbability(prediction.vowelProbability)
		self.needToRecord = self.wordImage.predictNeedToRecord(isCurrentlyRecording: self.needToRecord)
	}
}
```

 

Et enfin, on peut appeler notre code depuis le controller [RecordWordVC](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/Controller/RecordWordVC.swift). La classe "WordImage" s'occupe aussi de stocker un buffer des dernières expressions détectées, buffer dans lequel on ira piocher les blend shapes qui correspondent au mot que l'utilisateur vient de prononcer :

```swift
func renderer(\_ renderer: SCNSceneRenderer, didUpdate node: SCNNode, for anchor: ARAnchor) {
	guard let faceAnchor = anchor as? ARFaceAnchor else { return }
	wordImage.appendNewExpression(faceAnchor.blendShapes)
	computePerformance()
	if (!waitingForAlertAnswer) {
		recordOnNeutralDetection(blendShapes: faceAnchor.blendShapes)
	}
}
```

 

##### Génération des WordImage

Une fois la fin de mot détectée, on peut lancer la génération du spectrogramme.

Pour améliorer l'image, j'ai décidé de multiplier toutes les valeurs des 'blend shapes' par 2. En effet, on peut voir dans notre vue de stats que les valeurs dépassent très rarement 0.5 lorsqu'on parle. En traitement d'image cela correspondrait à augmenter l'exposition et contraste, ce qui va aider notre algorithme de machine learning. Ensuite on multiplie notre valeur par 256 avant de la convertir en octet; cette valeur va devenir notre pixel. Il faut penser à vérifier les valeurs limites des nombres qu'on manipule car en partant d'un flottant que l'on multiple par 2 puis par 256 on peut se retrouver avec une valeur inférieure à 0 ou supérieure à 255. Ensuite on peut faire notre conversion en image niveau de gris.

La classe [WordImage](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/BusinessLogic/WordImage.swift) contient une méthode wordImageFromExpressions() qui s'occupe de réaliser tout ces traitements. Comme l'implémentation des différentes étapes est panachée de gestion bas niveau des images et que ce n'est pas le sujet de cet article, je vous laisse regarder le code plus en détail sur github si vous le souhaitez. On peut voir les étapes de traitement ci dessous :

```swift
func wordImageFromExpressions() -> UIImage? {
	guard neutralHistory.history.count > 0 else {
		print("unable to determine the right window of expressions containing the word because neutral has not been monitored")
		return nil
	}
	// we want to process the samples that led to the detection (and a few extra ones before)
	let samplesCountToProcess = min(recordedExpressionsCount + neutralHistory.probabilitiesHistoryMaxLength, WordImage.kMaxSamplesCountPerRecording, expressionsBuffer.count)
	// we also remove part of the sample that led to stopping the recording
	let recordingToProcess = expressionsBuffer.suffix(from: expressionsBuffer.count - samplesCountToProcess).prefix(samplesCountToProcess - neutralHistory.probabilitiesHistoryMaxLength / 3)
	let simplifiedRecording = FaceProcessing.simplifyRecording(Array(recordingToProcess))
	let enhancedRecording = WordImage.enhanceRecording(simplifiedRecording, factor: 2.0)
	
	return WordImage.imageFrom(enhancedRecording, addPadding: false)
}
```

Il faut ensuite sauvegarder les images sur le téléphone avec un dossier par mot, et on pourra les récupérer sur macOS depuis iTunes.

### Apprentissage du modèle

Nous allons utiliser les MLImageClassifier qui permettent d'entraîner un modèle à partir d'un jeu d'images. Il suffit de ranger ses images dans un dossier du nom de la catégorie. On peut fournir n'importe quelle taille d'image et laisser Apple faire le resize/crop nécessaire, mais il vaut mieux faire soi-même le traitement pour avoir plus de contrôle sur le processus.

Concernant la taille du jeu de données, j'ai au départ eu des résultats très décevants car j'utilisais une vingtaine d'enregistrements par mot. En prenant le temps de faire 200 enregistrements par mot, je commence à avoir un modèle assez efficace, même si les résultats sont assez aléatoires avec un autre orateur.

##### Redimensionnement des spectrogrammes

L'apprentissage d'Apple travaille en 299x299 pixel et c'est comme ça, on n'a pas le choix, il va donc falloir redimensionner à ces dimensions. C'est un peu dommage car on a des spectrogrammes bien plus petit, au maximum 15x120 pixel (15 traits du visage sur 120 échantillons), mais on va se retrouver à entraîner notre modèle sur une information redondante et dupliquée à cause du 'resize'.

Avant de redimensionner, on a un choix à faire :

- soit on ajoute du 'padding', c'est à dire de la donnée de bourrage, pour simuler un enregistrement qui fait toujours la même durée (120 échantillons)
- soit on étire notre image, sans se soucier de la durée d'enregistrement des mots, au risque d'avoir 1 échantillon représenté par une hauteur variable de notre image finale (de 2 à 10 pixels vu qu'un mot dure de 30 à 120 échantillons)

J'ai choisi d'étirer mon spectrogramme au maximum, ce qui me donne une méthode de reconnaissance plus robuste face à des personnes qui parlent à des vitesse différentes.

Comme nous avons des images très petites, une mise à l'échelle standard à tendance à flouter énormément l'image car presque tous les logiciels fonctionnent en [bilinear filtering](https://fr.wikipedia.org/wiki/Filtrage_bilin%C3%A9aire). J'ai donc décidé d'utiliser un redimensionnement en "nearest pixel" qui conserve le très fort contraste entre chaque pixel initial. En pratique j'ai pu constater que j'avais de meilleurs apprentissages avec les spectrogrammes traités de cette manière. Voila une comparaison entre les 2 méthodes de filtrage sur le mot "maison":

\[caption id="attachment\_2094" align="aligncenter" width="300"\][![](/assets/images/Benoit-maison-20180817-134016-300x145.png)](/assets/images/Benoit-maison-20180817-134016.png) nearest pixel VS bilinear filtering\[/caption\]

Par exemple les 4 spectrogrammes ci dessous proviennent de 4 enregistrements différents du mot "maison". Même si la la durée d'enregistrement varie de 29 à 34 échantillons, on peut constater que cela ne se voit plus une fois l'image redimensionnée :

 [](/assets/images/bateau1.png) [![](/assets/images/bateau1-150x150.png)](/assets/images/bateau1.png)  [](/assets/images/bateau2.png) [![](/assets/images/bateau2-150x150.png)](/assets/images/bateau2.png) [](/assets/images/bateau3.png)  [![](/assets/images/bateau3-150x150.png)](/assets/images/bateau3.png) [![](/assets/images/bateau4-150x150.png)](/assets/images/bateau4.png)

Pour traiter vos images, vous pouvez par exemple utiliser le traitement par lot de Photoshop ou bien faire du scripting avec ImageMagick.

##### Apprentissage des mots

Cette fois-ci, au lieu de taper plusieurs lignes de code dans notre playground, on va utiliser l'assistant graphique que propose Apple : le [MLImageClassifierBuilder](https://developer.apple.com/documentation/createml/mlimageclassifierbuilder). Il a l'avantage de laisser le développeur sélectionner très facilement son jeu de données, il permet de configurer les paramètres principaux depuis l'interface graphique, et il affiche les images au fur et à mesure qu'il les traite.

[![](/assets/images/5b286036-05a0-45ec-bbbf-8dfda5abf35b-249x300.png)](/assets/images/5b286036-05a0-45ec-bbbf-8dfda5abf35b.png)

 

Pour le mettre en place, ajoutez simplement les lignes suivantes dans votre [playground](https://github.com/3IE/WordRecognitionModelTraining-MacOS/blob/v1.0/wordRecoTraining.playground/Contents.swift) :

```swift
func wordTraining() {    
    let builder = MLImageClassifierBuilder()	
    builder.showInLiveView()
}
```

Avec une trentaine d'itérations, vous devriez obtenir des résultats satisfaisants.

### Prédiction d'un mot

Pour intégrer votre modèle prédictif de mots, procédez de la même manière que le modèle prédictif de voyelle, en réutilisant le RecordWordVC existant. Par contre, une fois le spectrogramme généré, il faut le redimensionner directement sur le téléphone avant de le donner au modèle coreML. Pour cela j'ai créé une extension de UIImage qui me permet de redimensionner une image avec la qualité d'interpolation de mon choix. Dans le cas présent, on utilisera CGInterpolationQuality.none qui correspond à du 'nearest pixel'. Vous pouvez trouver ce code dans le fichier [Helper.swift](https://github.com/3IE/WordRecognition-iOS/blob/v1.0/faceextract/BusinessLogic/Helper.swift) :

```swift
extension UIImage {
    func resizedImage(\_ newSize: CGSize, interpolationQuality: CGInterpolationQuality = .default) -> UIImage {
        guard self.size != newSize else { return self }
		
        UIGraphicsBeginImageContextWithOptions(newSize, true, 1);
		
        guard let context = UIGraphicsGetCurrentContext() else { return self }
        context.interpolationQuality = interpolationQuality
		
        self.draw(in: CGRect(x: 0, y: 0, width: newSize.width, height: newSize.height))
        let newImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()
        return newImage!
    }
}
```

On peut ensuite lancer la prédiction.

# Analyse des résultats

### Apprentissage dans Xcode

Avec cette première version de CreateML, Apple fourni des outils faciles d'accès et rapide à générer le modèle, même s'ils manquent quand même d'options et de fonctionnalités comparés à des outils comme TensorFlow. Sur les cas d'usages les plus simples, il y a quand même une vraie facilité d'accès. Il est dommage que MLImageClassifierBuilder ne permette pas de dissocier l'extraction de features des images de l'entraînement du modèle. En effet, dès qu'on veut modifier un paramètre, il faut relancer le processus complet ce qui devient assez vite laborieux, même sur un petit ensemble de 2000 images.

A surveiller dans les prochaines versions.

### Reconnaissance des voyelles

La reconnaissance des voyelles est un peu capricieuse. J'ai un taux d'erreur d'environ 8% mais ça cache de grosses disparités. Je n'ai pas le même nombre d'enregistrement par voyelles et toutes ne sont pas aussi faciles à reconnaître. Le 'e' et le 'o' peuvent se prononcer avec une forme de bouche proche, ce qui perturbe le modèle. Le 'i' fonctionne bien par contre. En forçant un peu les traits, 'o' bien rond avec mâchoire bien ouvert, 'u' avec les lèvres bien en avant, 'i' en tirant bien le coin des lèvres, on peut faciliter la reconnaissance, mais ce n'est pas très réaliste.

### Reconnaissance du neutre

La reconnaissance du neutre marche vraiment bien, surtout avec la moyenne mobile. Comme c'est très facile et rapide d'enregistrer des expressions, on peut rapidement obtenir un bon modèle.

### Reconnaissance des mots

La reconnaissance des mots fonctionne à 65% sur mes fichiers de tests. C'est un résultat mitigé. Il faudrait augmenter le nombre d'enregistrements et le nombre d'orateurs pour se faire une idée plus précise.

Il y a des mots qui à l'oral n'ont rien à voir mais qui peuvent se ressembler au niveau des mouvements de lèvre, comme "chien" et "vert". C'est ici qu'on a des erreurs qui nous semblent aberrantes d'un point de vue humain.

Si on pouvait garder ce taux sur un nombre plus important de mots cela serait intéressant, mais en augmentant la taille du dictionnaire il va avoir de plus en plus de mots qui se ressemblent et je crains que les résultats se dégradent.

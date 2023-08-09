---
title: "Migrer vers Swift 3"
date: "2016-11-16"
categories: 
  - "technical"
tags: 
  - "apple"
  - "ios"
  - "swift"
  - "swift-2"
  - "swift-3"
---

[**Swift 3.0**](https://swift.org/blog/) change tout, encore une fois.

Si vous pensiez que passer de Swift **1.2** à **2.0** était un grand changement, préparez vous à une refonte du langage phare d'Apple !

Heureusement, l'outil de migration de versions de Swift, qui est mis à disposition par Apple à chaque _nouvelle version_ du langage, nous facilite, une fois de plus, la mise à niveau de nos projets.

Disparition de certaines notations jugées antiques, simplification de la syntaxe et uniformisation de certaines normes font partie des nombreux changements que vous découvrirez.

### La fin de l'héritage C++/Objective C

Swift 2.1 avait déjà préparé le terrain en indiquant des Warnings lors de l'utilisation de boucles "à l'ancienne" de style C   **`for (int i = 0; i < taille; i++)`**.

Swift possède suffisamment de types de boucles pour pouvoir se passer de ces notations, et rendre ainsi le langage moins obscur au débutant. On peut aimer ou pas, mais Chris Lattner, le créateur du Swift [explique pourquoi il est sage](https://github.com/apple/swift-evolution/blob/master/proposals/0004-remove-pre-post-inc-decrement.md) de retirer les opérateurs \--  et ++  du lexique du Swift. Leur utilisation en pre-incrémentation ou post-incrémentation n'est pas toujours très claire, leur intérêt est minime par rapport à utiliser += 1  ou` -= 1` , et enfin, leur utilisation était assez limitée car le langage propose de nombreuses notations permettant de s'en passer.

Encore une fois, Apple fait tout pour faire disparaître les fonctions statiques de style C au profit de méthode Swift. Ainsi les célèbres fonctions du Grand Central Dispatch sont remplacées.

 

```swift
dispatch_async(dispatch_get_main_queue()) {
// Code à exécuter dans le main thread
}
```

 

Elles sont remplacées par les méthodes statiques d'un nouvel objet, la  `DispatchQueue` , ce qui est une approche plus Swift car orientée objet.

 

```swift
DispatchQueue.main.async {
// Code à exécuter dans le main thread
}
```

 

### Des effets positifs sur les API

["The Grand Renaming"](https://developer.apple.com/videos/play/wwdc2016/403/), c'est ainsi qu'Apple a présenté un des aspects le plus important de la mise à jour. Le nom de la plupart des méthodes et arguments changent pour les raisons suivantes.

#### La simplicité avant toute chose

Depuis l'Objective C, le premier paramètre d'une méthode était contenu dans le nom de cette dernière. Lors du passage à Swift, Apple a conservé cette habitude.

```swift
// Swift 2

override func numberOfSectionsInTableView(tableView: UITableView) -> Int

names.indexOf("3ie")
```

 

Les nouvelles méthodes voient leur nom se raccourcir et le label du premier argument devenir obligatoire, et les fonctions ci-dessus sont renommées ainsi :

```swift
// Swift 3

override func numberOfSections(in tableView: UITableView) -> Int

names.index(of: "3ie")
```

 

Et c'est bien plus lisible !

#### Vous avez dit redondance?

```swift
// Swift 2

pantalon.color = UIColor.blueColor()
tshirt.color = UIColor.whiteColor()
chapeau.color = UIColor.redColor()
```

 

Tout développeur avec un minimum d'éxperience est agaçé par ce défaut qui touche sa corde la plus sensible. Ce défaut est la redondance, qui était encore très présent dans l'API Cocoa.

Pourquoi répeter "Color" après chaque couleur alors que c'est une fonction de UIColor ? C'est le genre de question sans réponses que se posait la communauté Swift.

Et cette fois-ci, Apple y a répondu :

```swift
// Swift 3

pantalon.color = UIColor.blue()
tshirt.color = UIColor.white()
chapeau.color = UIColor.red()
```

 

#### Règles grammaticales de la langue anglaise

Lors de la WWDC 2016, les développeurs de chez Apple ont particulièrement insisté sur un point: le nouveau Swift doit être le plus grammaticalement correct, et doit se lire comme de l'anglais. Ces nouvelles "règles grammaticales" deviennent la nouvelle ligne directrice de la guideline Swift 3.

Par exemple, les méthodes avec [effet de bord](https://fr.wikipedia.org/wiki/Effet_de_bord_(informatique)) (c'est à dire qui modifient l'objet) ont un nom différent de celle qui ne modifie pas l'objet.

```swift
x.reverse()

```

Ici la méthode  reverse désigne l'action qu'elle aura sur  **`x`** . Elle modifie l'objet. Si on veut avoir un nouvel objet à partir de  `x` sans modifier , il faut conjuguer le nom de la méthode en fonction. Elle devient alors:

```swift
let y = x.reversed()

```

 

On comprend alors tout de suite que  `reversed` retourne quelque chose.

Pour plus d'information sur l'API, consultez la vidéo de la WWDC 2016 "[Swift API Design Guidelines](https://www.youtube.com/watch?v=UU2fNq35xkM)".

### La migration d'un projet Swift 2.2 vers 3.0

Afin de tester au mieux les nouveautés et mieux s'en rendre compte, j'ai passé un projet assez conséquent initialement écrit en Swift 1.0 puis mis à jour jusqu'en 2.2.

#### La préparation

Avant de procéder à la migration, il est préférable de vérifier les dépendances du projet en s'assurant que les bibliothèques et frameworks utilisés par le projet possèdent une version compatible avec Swift 3.

Pour les dépendances écrites en Objective-C, la seule modification que vous aurez à faire est à l'appel des fonctions et méthodes de celle-ci dans le projet. En revanche, pour vos projets vous devrez sans doute changer de version de release.

Une fois la vérification faite, vous n'aurez plus qu'à modifier le fichier de configuration (le Cartfile dans le cas de Carthage ou alors le Podfile sur Cocoapoad) en y renseignant les bons numéros de version.

#### L'outil de migration

Xcode 8 fraichement installé, j'ouvre mon projet quand l'IDE me propose de mettre à jour le code.

[![converttocurrentsyntax](/assets/images/convertToCurrentSyntax.png)](/assets/images/convertToCurrentSyntax.png)

J'ai le choix entre passer à Swift 3 ou bien utiliser la syntaxe Swift 2.3. Cette version est une mise à jour mineur de 2.2 qui permet au projet d'être compilé sous Xcode 8 sans avoir à tout changer comme Swift 3.

Cette version intermédiaire permet aux développeurs de déployer sous iOS 10, macOS 10.12 et Watch OS 3 sans avoir à modifier le langage.

Je prends mon courage à deux mains, et je choisis de convertir le projet dans la 3ème version du langage.

#### L'impact global se confirme

La quarantaine de fichiers Swift qui composent le projet apparaissent alors dans l'interface du gestionnaire de migration. Comme attendu, tout le projet est impacté par la mise à jour, qui est tout sauf mineure.

La modification qui revient le plus souvent est l'ajout de l'underscore avant le nom du premier argument d'une fonction qui servira de label lorsqu'on l'appelle. Il reste alors du travail à faire.

 

[![underscore](/assets/images/underscore.png)](/assets/images/underscore.png)

Comme on peut voir dans cette méthode saveObject , la logique Swift 3 veut qu'on renomme en  save et qu'on rajoute le label  `object` au premier argument.

J'ai dû alors répéter cette opération une centaine de fois pour avoir un code swifty. Finalement, l'outil de migration n'a fait que mettre des underscore là où il faut changer le code afin de respecter au mieux la nouvelle guideline.

Ce n'est pas la première fois que l'assistant de migration propose des modifications mystiques, mais cette fois-ci, je suis tombé sur des ajouts de code défiant toute logique.

Comme cette fonction de comparaison templatées ajoutée en `fileprivate` (un nouveau niveau d'accessibilité, rendant une fonction accessible uniquement dans le fichier où elle est déclarée) au début de ce fichier ex nihilo, dont je cherche toujours l'origine et qui ne sera jamais utilisée.

[![bizarre](/assets/images/bizarre-1024x202.png)](/assets/images/bizarre.png)

 

### Finalement

Finalement, la mise à jour rend le code fonctionnel et compilable par Xcode 8 et le compilateur Swift 3, malgré quelques petites incohérences. En revanche, il reste un travail de relecture profonde pour les développeurs s'ils souhaitent rendre conforme leur code avec les [guidelines de design d'API Swift](https://swift.org/documentation/api-design-guidelines/).

Enfin, disparue d'Xcode en lors du passage d'Objective C à Swift,  l'option de compilation "treat warning as errors", qui permet d'avoir des erreurs à la place des Warnings, fait son retour dans Xcode 8.

Pour plus de détails sur les points abordés dans l'article, voici quelques liens :

[Le guide de migration Apple](https://swift.org/migration-guide/)

[Le nouveau GCD](http://rolling-rabbits.com/2016/07/21/grand-central-dispatch-in-swift-3/)
<br>
<br>

---------------------------------------
<br>
Auteur: **serge.panev**

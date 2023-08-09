---
title: "La UI d'Instagram en SwiftUI"
date: "2019-10-23"
categories: 
  - "technical"
  - "ux"
tags: 
  - "apple"
  - "ios13"
  - "swift5"
  - "swiftui"
  - "xcode11"
---

Lors de la WWDC 2019 qui s'est tenue le 3 juin, Apple a dévoilé son nouveau framework [**SwiftUI**](https://developer.apple.com/xcode/swiftui/) permettant d'implémenter des interfaces en Swift de façon déclarative respectant les "[_Human Interface Guidelines_](https://developer.apple.com/design/human-interface-guidelines/)", simplifiant tous les patterns récurrents dans les applis iOS.

Le nouveau framework nécessitera donc iOS 13+ pour tourner. Il faudra donc attendre au moins 2 ans pour que ce dernier soit adopté par les développeurs puisqu'en moyenne une application iOS supporte les 2 dernières versions de l'OS.

En quelques semaines, beaucoup de développeurs se sont intéressés à SwiftUI et donc plein de projets sont déjà disponibles en Open Source. Voici un repository répertoriant les plus intéressants : [About-SwiftUI](https://github.com/Juanpe/About-SwiftUI)

# Objectif :

Nous vous proposons dans cet article de voir quelques unes des possibilités offertes par SwiftUI en implémentant la page d'accueil d'Instagram.

[![](/assets/images/Instagram-UI-2018-iPhone-X.png)](/assets/images/Instagram-UI-2018-iPhone-X.png)

# Pré-requis :

- macOS Mojave + ou macOS Catalina.
- Xcode 11.

que vous pouvez télécharger **[ici](https://developer.apple.com/download/)**.

_**NB:** le live preview sur Xcode 11 n'est disponible que sur MacOS Catalina !_

Vous pouvez également télécharger l'ensemble des Assets utilisés dans ce tutoriel [**ici**](https://github.com/yassram/InstaUI/blob/master/Assets.xcassets.zip).

 

# L' application :

Commençons par créer un projet Xcode. Assurez vous que la case "**_Use SwiftUI_**" est bien cochée.

[![](/assets/images/Capture-d’écran-2019-07-22-à-10.04.55.png)](/assets/images/Capture-d’écran-2019-07-22-à-10.04.55.png)

Une fois votre projet créé, vous devez avoir 3 fichiers Swift :

- **AppDelegate.swift**
- **SceneDelegate.swift**
- **ContentView.swift**

Vous remarquerez la création du nouveau Scene Delegate avec l'arrivée du [**Scene-Based Life-Cycle**](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle).

En ce qui nous concerne, nous allons nous intéresser au fichier **ContentView.swift** qui contiendra un Hello World en SwiftUI :

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello World")
    }
}
```

quant à :

```swift
#if DEBUG
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
#endif

```

C'est ce qui permet d'avoir un live preview de votre code sur Xcode (_si vous utilisez MacOS Catalina_).

[![](/assets/images/Capture-d’écran-2019-07-22-à-10.29.34-522x1024.png)](/assets/images/Capture-d’écran-2019-07-22-à-10.29.34.png)

Commençons par créer une TableView. En SwiftUI cela se fait en une ligne :

```swift
struct ContentView: View {
    var body: some View {
        List {
            Text("Hello World")
        }
    }
}
```

Construisons notre **_PostCell_** qui contiendra un _post._ La _cell_ peut être subdivisée en :

- **Header :** contenant l'utilisateur qui a posté.
- **Post :** l'image.
- **Barre horizontale :** contenant les boutons de like, comment, share et save.
- **Le nombre de Likes.**
- **La description.**

Si on décompose le **Header** on aura : [![](/assets/images/Capture-d’écran-2019-07-22-à-10.48.17.png)](/assets/images/Capture-d’écran-2019-07-22-à-10.48.17.png)

Avec en vert une **HStack** (Horizontal Stack) et en bleu une **VStack** (Vertical Stack)

En SwiftUI ça donne :

```swift
struct PostCell: View {
    var body: some View {
        HStack {
            Image("Avatar")
            VStack(alignment: .leading){
                Text("pieroborgo")
                Text("Milan, Italy")
            }
            Spacer()
            Image("More")
        }
    }
}
```

(N'oubliez pas de mettre à jour votre **ContentView** en remplaçant _Text("Hello World")_ par une instance _PostCell()_ )

[![](/assets/images/Capture-d’écran-2019-07-22-à-11.01.49.png)](/assets/images/Capture-d’écran-2019-07-22-à-11.01.49.png)

On ajustera la taille du texte :

```swift
struct PostCell: View {
    var body: some View {
        HStack {
            Image("Avatar")
            VStack(alignment: .leading){
                Text("pieroborgo")
                    .font(Font.system(size: 13.5))
                Text("Milan, Italy")
                    .font(Font.system(size: 11.5))
            }
            Spacer()
            Image("More")
        }
    }
}
```

 

Passons à la suite de la **PostCell.** Le concept reste le même,  subdiviser la view en **HStack** et en **VStack**.

[![](/assets/images/Capture-d’écran-2019-07-22-à-11.09.24-757x1024.png)](/assets/images/Capture-d’écran-2019-07-22-à-11.09.24.png)

Ce qui donne en SwiftUI:

```swift
struct PostCell: View {
    var body: some View {
        VStack {
            // Header
            HStack {
                Image("Avatar")
                VStack(alignment: .leading){
                    Text("pieroborgo")
                        .font(Font.system(size: 13.5))
                    Text("Milan, Italy")
                        .font(Font.system(size: 11.5))
                }
                Spacer()
                Image("More")
            }
            
            // Post
            Image("Photo")
                .resizable()
            
            // Barre horizontale
            HStack(alignment: .center) {
                Image("Like")
                Image("Comment")
                Image("Send")
                Spacer()
                Image("Collect")
            }
            
            // Le nombre de Likes
            Text("Liked by leeviahq and 621 others")
                .font(Font.system(size: 13.5))
            
            // La description
            Text("pieroborgo Thanks for dowloading this freebie ❤️ #freebie #instagram #sketch")
                            .lineLimit(4)
                            .font(Font.system(size: 12.5))
                            .foregroundColor(.init(white: 0.2))
        }
    }
}
```

_**NB :**_ .lineLimit( Int? ) : permet de préciser le nombre maximal de lignes pour un Text. (nil pour un nombre illimité de lignes)

[![](/assets/images/Capture-d’écran-2019-07-22-à-11.26.51-860x1024.png)](/assets/images/Capture-d’écran-2019-07-22-à-11.26.51.png)

On s'approche du résultat souhaité mais on a encore quelques petits problèmes :

1. L'image ne prend pas toute la largeur de l'écran.
2. Le nombre de likes est centré.

Le comportement de l'image est voulu par Apple puisque ça respecte les Guidelines concernant la **SafeArea**. Mais dans notre cas cela ne nous convient pas. Il suffit donc de donner à l'image un _**leading**_ et un _**trailing**_ **padd****ing** négatifs correspondant à la taille de la **SafeArea** (20 px).

```swift
Image("Photo")
     .resizable()
     .scaledToFit()
     .padding(.leading, -20)
     .padding(.trailing, -20)
```

En ce qui concerne le second problème, il faut savoir que les **HStack**, **VStack** et **ZSatck** prennent un paramètre optionnel pour **l'alignement**. Il suffit donc de spécifier un alignement **.leading** pour la **VStack** englobant toutes nos Views:

```swift
VStack(alignment: .leading){
  // code
}
```

[![](/assets/images/Capture-d’écran-2019-07-22-à-11.55.14-784x1024.png)](/assets/images/Capture-d’écran-2019-07-22-à-11.55.14.png)

Ajoutons maintenant une barre de navigation et quelques cellules :

```swift
struct ContentView: View {
    var body: some View {
        NavigationView {
            List {
                PostCell()
                PostCell()
                PostCell()
            }.navigationBarTitle("InstaUI", displayMode: .inline)
        }
    }
}

```

[![](/assets/images/Capture-d’écran-2019-07-22-à-12.01.14-512x1024.png)](/assets/images/Capture-d’écran-2019-07-22-à-12.01.14.png)

Passons aux Stories. Il s'agit tout simplement d'une **ScrollView** horizontale contenant une **HStack.**

On garde la même notation :  **HStack** en vert et **VStack** en bleu, avec une **ScrollView** en rouge :

[![](/assets/images/Capture-d’écran-2019-07-22-à-12.10.53.png)](/assets/images/Capture-d’écran-2019-07-22-à-12.10.53.png)

On crée une nouvelle struct dédiée au Stories :

```swift
struct StoriesView: View {
    var body: some View {
        VStack(alignment: .leading) {
            
            // Stories text + Watch all
            HStack {
               Text("Stories")
               Spacer()
               Image("Watch")
               Text("Watch all")
            }
            
            // Stories Circles
            ScrollView([.horizontal], showsIndicators: false){
                HStack {
                   Image("AvatarBig1")
                   Image("AvatarBig1")
                }
            }

        }
    }
}
```

Les Stories sont désormais **_Scrollable_** horizontalement.

(N'oubliez pas d'ajouter une instance _StoriesView()_ dans _List_ de votre _ContentView_)

['](/assets/images/Capture-d’écran-2019-07-22-à-12.35.47.png)[![](/assets/images/Capture-d’écran-2019-07-22-à-12.35.47.png)](/assets/images/Capture-d’écran-2019-07-22-à-12.35.47.png)

On veut maintenant ajouter l'image de bordure en arrière plan. On utilisera donc une **ZStack** :

```swift
ZStack {
    Image("Border")
    Image("AvatarBig1")
}
```

Mettons cela dans une **VStack** afin d'ajouter le nom de l'utilisateur en dessous de sa photo de profil :

```swift
VStack {

     ZStack {
          Image("Border")
          Image("AvatarBig1")
     }
     
     Text("beardguy")
         .font(Font.system(size: 13.5))
}
```

[![](/assets/images/Capture-d’écran-2019-07-22-à-13.39.45-273x300.png)](/assets/images/Capture-d’écran-2019-07-22-à-13.39.45.png)

On s'approche du résultat final. Toutefois, à cause de la **SafeArea**, la ScrollView ne prend pas toute la largeur de l'écran (comme on a vu précédemment pour l'image de la **PostCell**). On ajoute donc un **leading** et un **trailing** **padding** avec des valeurs négatives à la **ScrollView** et un petit **padding** à la **HStack** pour compenser.

```swift
ScrollView {
     HStack {

         VStack {

         }.padding(.trailing, 12)    // espace entre cellules

     }.padding(.leading, 16)   // espace au début de la ligne

}.padding(.leading, -14)
 .padding(.trailing, -14)
 // prendre toute la largeur de l'écran
```

[![](/assets/images/Capture-d’écran-2019-07-22-à-14.02.22-268x300.png)](/assets/images/Capture-d’écran-2019-07-22-à-14.02.22.png)

 

Pour finir la partie Stories, il ne manque plus que le "_Your Story_" avec le petit plus bleu. Il ne s'agit que d'une **ZStack** avec **bottomTrailing** comme **Alignment**.

[![](/assets/images/Capture-d’écran-2019-07-22-à-16.05.11.png)](/assets/images/Capture-d’écran-2019-07-22-à-16.05.11.png)

On ajoute donc au début de notre **HStack** qui contient les cellules de nos Stories :

```swift
VStack {
     ZStack(alignment: .bottomTrailing) {
          Image("AvatarBig")
          Image("Add")
     }
     Text("Your Story")
         .font(Font.system(size: 13.5))
}.padding(.trailing, 12)
```

Et on obtient :

[![](/assets/images/Capture-d’écran-2019-07-22-à-17.00.09-255x300.png)](/assets/images/Capture-d’écran-2019-07-22-à-17.00.09.png)

 

Ajoutons maintenant les deux boutons dans la N_avigation Bar_ :

```swift
List {
     StoriesView()
     PostCell()
     PostCell()
     PostCell()
}.navigationBarTitle("InstaUI", displayMode: .inline)
 .navigationBarItems(leading: Image("Camera"), trailing: Image("Direct"))
```

[![](/assets/images/Capture-d’écran-2019-07-22-à-17.26.45-500x1024.png)](/assets/images/Capture-d’écran-2019-07-22-à-17.26.45.png)

L'application est disponible en Open Source sur [ce dépôt](https://github.com/3IE/InstaUI) GitHub et pourra évoluer.

# Conclusion :

Pour l'avoir testé pendant quelques jours SwiftUI est un Framework assez puissant et efficace surtout pour des UI simples. Plus besoin de UITableViewDelegates, ViewController management... Cependant UIKit restera le framework phare d'Apple (pour le moment). Sachez qu'il est possible d'utiliser les deux Frameworks UIKit et SwiftUI au sein du même projet.

N'hésitez pas à jeter un coup d'oeil sur quelques projets intéressants réalisés en SwiftUI : [About-SwiftUI](https://github.com/Juanpe/About-SwiftUI)
<br>
<br>

---------------------------------------
<br>
Auteur: **yassir.ramdani**

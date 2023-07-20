---
title: "Introduction à l'ARCore de Google"
date: "2018-09-04"
categories: 
  - "technical"
---

En août 2017 , google mettait à disposition son SDK de réalité augmentée : L' **ARCore**. Il succède au Projet Tango, lancé en 2014 et qui visait à répandre la réalité augmentée, grâce à des smartphones pourvus de plusieurs capteurs spéciaux. 

Pour le moment seulement certains appareils Android supportent cet outil, parmi eux on retrouve notamment le **Google Pixel** ou encore le **Samsung Galaxy S8****.**

La liste exhaustive de devices compatible est publiée sur [la doc ARCore](https://developers.google.com/ar/discover/supported-devices)

L'outil équivalent chez Apple est L'ARKit. A l'heure actuelle, les deux kits de réalité augmentée se valent.

* * *

### **L’AR qu’est-ce que c’est ?**

Avant de rentrer plus profondément dans les détails techniques, prenons le temps de rappeler ce qu’est la **réalité augmentée**.

Souvent confondue avec la réalité virtuelle (Virtual Reality en anglais : **VR**) , qui consiste, elle, a créer un univers entièrement virtuel dans lequel l’utilisateur va évoluer (oculus, htc vive, google cardboard...), la réalité augmentée (Augmented Reality en anglais : **AR**) quant à elle, permet d'intégrer des éléments virtuels dans le monde réel. On ajoute des éléments, on augmente le réel, d’où son nom : la r_éalité augmentée_.

[![](/assets/images/arvr.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/arvr.png)

* * *

 

### **Qu’en est il de l’AR aujourd’hui ?**

A l’heure actuelle, la réalité augmentée est encore **peu répandue**. Un des leaders du moment est Microsoft avec ses _Hololens (ci-dessous)_, sorte de lunettes de réalité augmentée.

[![](/assets/images/hololens.jpg)](https://blog.3ie.fr/wp-content/uploads/2018/07/hololens.jpg)

Du coté des applications mobile on a par exemple IKEA, qui propose une application permettant de visualiser des meubles chez sois grâce à L’AR, afin d’aider le client dans sa décision. [![](/assets/images/ikea.jpg)](https://blog.3ie.fr/wp-content/uploads/2018/07/ikea.jpg)

Dans le domaine médical on retrouve également quelques exemples d’opérations dans lesquelles la réalité augmentée est utilisée. Le chirurgien (munis d'Hololens pour les exemples actuels), opère son patient tout en profitant de nombreuses informations affichées en réalité augmentée.[![](/assets/images/31501281374_81b237b65b_b.jpg)](https://blog.3ie.fr/wp-content/uploads/2018/07/31501281374_81b237b65b_b.jpg)

D’autres domaines commencent a intégrer cette technologie (automobile, publicité ...) , mais à l’heure actuelle, nous sommes plus dans une phase  d'**expérimentations** que dans une réelle phase adoption.

* * *

### Comment fonctionne L'ARCore ?

Afin d’être à même d’intégrer des éléments virtuels dans le monde réel, l'ARCore doit pouvoir au maximum "comprendre" son environnement. Il va notamment, repérer les surfaces planes (horizontales pour le moment) grâce à des "feature points" qui sont des points de l'environnement qui se différencient des autres points qui les entourent.

[![](/assets/images/feature-points.jpg)](https://blog.3ie.fr/wp-content/uploads/2018/07/feature-points.jpg)

Sur l'image ci-dessus, on peut voir les feature points en bleu et le plan détecté par L'ARCore en vert.

Pour pouvoir faire tout cela, l'ARCore utilise les capteurs de notre téléphone (accéléromètres notamment) pour pouvoir avoir les informations de position, rotations et autres déplacement dans la réalité, pour ainsi les transposer dans la scène "virtuelle". Il utilise également la caméra pour avoir la partie réelle de la scène et ainsi pouvoir travailler dessus (recherche de feature points, calcul de surfaces planes ...). La partie calculatoire etant très importante et complexe, il faut à l'ARCore un matériel suffisament puissant pour pouvoir assumer ces calculs. Cela explique pourquoi seulement certains terminaux sont compatibles.

### Un peu de pratique ...

 

Afin d'aller un peu plus loin et de rentrer dans les détails techniques, voici un petit tutoriel pour faire **vos premiers pas dans la réalité augmentée** avec L'ARCore. Pour suivre ce tutoriel, il est utile d'avoir quelque notions de programmation en **java** ainsi qu'une certaine aisance avec **Android Studio**.

#### Création du Projet :

Commençons par créer un **nouveau projet** sur Android Studio :

[![](/assets/images/create_project.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/create_project.png)

Choisissez l’**API** **26** comme API minimum :

[![](/assets/images/api26.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/api26.png)

Sélectionnez l’**Empty Activity** et terminez la création du  nouveau projet :

[![](/assets/images/empty_activity.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/empty_activity.png)

#### Paramétrage :

Nous devons maintenant paramétrer notre projet de sorte qu’il puisse utiliser l'ARCore. Commencez par aller dans le **AndroidManifest.xml** et ajoutez les lignes suivantes :

<uses-permission android:name="android.permission.CAMERA"/>
<uses-feature android:name="android.hardware.camera.ar" android:required="true"/>

 

Ajoutez également la ligne suivante, mais ce coup-ci, à l’intérieur des balises application :

<meta-data android:name="com.google.ar.core" android:value="required"/>

 

Ensuite, rendez vous dans le **gradle.build** du projet, ici (Project: tuto\_ar) , et ajoutez la ligne ci-dessous dans le champ "dependencies" :

classpath 'com.google.ar.sceneform:plugin:1.3.0'

 

Enfin, pour terminer notre paramétrage, ajoutez les dépendances suivantes dans le **gradle.build** de l’application (Module : app) :

Ces deux lignes dans "dependencies" :

implementation "com.google.ar.sceneform:core:1.3.0"
implementation "com.google.ar.sceneform.ux:sceneform-ux:1.3.0"

 

Et celle-ci juste dans le "build.gradle" :

apply plugin: 'com.google.ar.sceneform.plugin'

#### Création de l'ArFragment :

Maintenant que toutes les **autorisations** et **dépendances** ont été ajoutées, nous pouvons passer à la partie intéressante : la création de notre application.

Tout d’abord, occupons nous du layout. Dans **activity\_main.xml**, supprimez la textView et remplacez la par un **ArFragment** (élément qui va nous permettre d’afficher notre scène augmentée) :

<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout\_width="match\_parent"
    android:layout\_height="match\_parent"
    tools:context=".MainActivity">

    <fragment android:name="com.google.ar.sceneform.ux.ArFragment"
        android:id="@+id/ux\_fragment"
        android:layout\_width="match\_parent"
        android:layout\_height="match\_parent" />

</android.support.constraint.ConstraintLayout>

 

Vous pouvez lui donner le nom et les dimensions que vous souhaitez. Ici pour l’exemple, l’ArFragment est nommé ux\_fragment et il recouvre tout l’écran.

Nous pouvons maintenant réellement commencer à coder. Dans la **MainActivity**, créons tout d’abord un objet **ArFragment** et lions-le à l’ArFragment de notre layout :

public class MainActivity extends AppCompatActivity {

    private ArFragment arFragment;
   
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity\_main);

        arFragment = (ArFragment)getSupportFragmentManager().findFragmentById(R.id.ux\_fragment);
  }
}

#### Intégration d'un objet virtuel :

Il nous faut maintenant quelque chose à afficher dans notre scène, sans quoi la réalité augmentée perdrait tout son charme (et son intérêt aussi). Pour commencer, nous allons juste **créer un cube rouge**. Libre à vous de changer la couleur ou même la forme si l’idée d’afficher un cube rouge vous dérange.

Voici le code qui nous permet de créer le-dit cube :

public class MainActivity extends AppCompatActivity {

    private ArFragment arFragment;
    private ModelRenderable redCubeRenderable;
  
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity\_main);

        arFragment = (ArFragment)getSupportFragmentManager().findFragmentById(R.id.ux\_fragment);
        
        MaterialFactory.makeOpaqueWithColor(this, new Color(android.graphics.Color.RED))
        .thenAccept(
            material-> {
                  redCubeRenderable = 
                          ShapeFactory.makeCube(new Vector3(0.2f,0.2f,0.2f), new Vector3(0.0f,0.15f,0.0f), material);});
                    //si vous rencontrez un problème ici, il est possible que vous ne soyez pas en java 8 (nécessaire pour les lambdas)

 

Récapitulons :

Jusqu’ici nous avons une scène AR contenue dans notre ArFragment et une forme 3D. C’est un bon début mais, comme vous vous en doutez,  il va falloir relier tout ça à un moment. Cependant, avant d’aller plus loin, plusieurs notions sont à comprendre.

Tout d’abord l’organisation des différents modèles affichés dans notre scène. On pourrait représenter leurs différentes relations par **un arbre**. Chaque objet est **un nœud** de l’arbre et peut avoir **un parent** (_au maximum_) et de **0 à X enfants**. Un enfant “suis” son parent. Si l’on déplace le parent, l’enfant bouge avec lui. Pour visualiser, on peut prendre l’exemple suivant : nos bras sont enfants de notre corps. Si notre corps bouge, nos bras, _sauf cas exceptionnellement douloureux_, suivent.

Autre notion importante, chaque objet de la scène AR à besoin d’un **point d’ancrage** (**anchor** en anglais). Ainsi, la plupart du temps, on va créer notre anchor et lui donner comme enfant notre objet 3D.

Enfin, dernière chose à prendre en compte, et pas des moindres. Nous souhaitons pouvoir placer notre modèle 3D dans notre scène au moment et à l’endroit où nous toucherons l’écran. Ainsi, il nous faut un **listener** pour “écouter” nos actions et agir en conséquence.

 

Maintenant que nous avons vu toutes ces choses, voyons comment cela se traduit au niveau du code :

public class MainActivity extends AppCompatActivity {

    private ArFragment arFragment;
    private ModelRenderable redCubeRenderable;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity\_main);

        arFragment = (ArFragment)getSupportFragmentManager().findFragmentById(R.id.ux\_fragment);

        MaterialFactory.makeOpaqueWithColor(this, new Color(android.graphics.Color.RED)) 
        .thenAccept( 
             material-> { 
                  redCubeRenderable = 
                           ShapeFactory.makeCube(new Vector3(0.2f,0.2f,0.2f), new Vector3(0.0f,0.15f,0.0f), material);}); 
       //si vous rencontrez un problème ici, il est possible que vous ne soyez pas en java 8 (nécessaire pour les lambdas)

        arFragment.setOnTapArPlaneListener(
                (HitResult hitResult, Plane plane, MotionEvent motionEvent) -> {
                    if (redCubeRenderable == null) {
                        return;
                    }
                    if (plane.getType() != Plane.Type.HORIZONTAL\_UPWARD\_FACING) {
                        return;
                    }

                    //On créé le point d'encrage du modèle 3d
                    Anchor anchor = hitResult.createAnchor();
                    AnchorNode anchorNode = new AnchorNode(anchor);
                    anchorNode.setParent(arFragment.getArSceneView().getScene());

                    //On attache ensuite notre modèle au point d'encrage
                    TransformableNode Node = new TransformableNode(arFragment.getTransformationSystem());
                    Node.setParent(anchorNode);
                    Node.setRenderable(redCubeRenderable);
                    Node.select();

                }
        );

    }
}

 

On remarque qu’on attache un **OnTapArPlaneListener** à notre arFragment  qui, comme son nom l’indique, va **s'éxecuter** à chaque appuis de l’utilisateur sur un plan détecté par l’ARCore. On crée ensuite les éléments (nœud et point d'ancrage) évoqués plus haut à l'intérieur de ce listener. Ainsi, à chaque fois que nous toucherons un endroit d’une surface détectée par ARCore, un magnifique cube rouge sera placé.

 

Vous pouvez alors **compiler** et **lancer** votre première application de réalité augmentée (sur un modèle compatible évidemment).

\[youtube https://www.youtube.com/watch?v=jP\_I9S2XLfY&w=560&h=315\]

* * *

### Pour aller plus loin ...

Faire apparaître un cube rouge dans notre environnement proche est certes satisfaisant, mais seulement un temps. En effet, cette forme basique ne présente pas grand intérêt, elle ne contient pas d'information ou de message particulier et  n'est pas particulièrement esthétique. Voyons donc comment remplacer ce cube par le **modèle 3D** de notre choix.

Pour ce faire, il vous faut en premier lieu un modèle 3D qui vous convienne. Vous avez le choix, soit de **le créer** vous même avec votre logiciel de prédilection (Blender, Maya, SolidWorks ...), ou bien d'en **télécharger** sur une des nombreuses **plateformes en ligne** mettant à disposition un grand nombre de modèles 3D (Poly,  Sketchfab...).

Le modèle doit nécessairement être dans l'un des formats suivants : **OBJ**, **FBX** ou **glTF** (les seuls supportés par ARCore pour le moment).

Une fois pourvu de votre modèle, passons à la **partie technique**.

#### Mise en place de Sceneform :

Tout d'abord il vous faut installer l'outil **Sceneform** de Google. Pour ce faire rendez vous dans **File>Settings>Plugins>Browse Repositories** pour Windows et dans **Android Studio>Preferences>Plugins** pour Mac.

Cliquez ensuite sur **Browse Repositories** et installez **Google Sceneform Tools**.

[![](/assets/images/sceneform-plugin.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/sceneform-plugin.png)

Après cette étape, un **redémarrage** du studio est nécessaire.

#### Intégration du modèle 3D :

L'essentiel du paramétrage étant fait, **importons** notre modèle 3D dans Android Studio.

Tout d'abord, créez un dossier **sampledata** à la racine du projet en faisant un clic droit sur app **new>sampledata directory**. Ce dossier permet tout simplement de ne pas encombrer l'APK qui sera créée , avec les ressources que l'on va stocker dedans. On gagne donc en taille ainsi qu'en performances.

 

Créons ensuite un **nouveau dossier** dans notre sample data et donnons lui un nom. C'est ici que l'on va stocker notre modèle 3D :

[![](/assets/images/mon_modele_3d.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/mon_modele_3d.png)

Glissez-déposez votre objet dans ce dossier.

Faites ensuite un clic droit sur votre objet et sélectionnez **"Import Sceneform Asset"** :

[![](/assets/images/import-sceneform.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/import-sceneform.png)

Une nouvelle fenêtre s'ouvre  :

[![](/assets/images/fenetre-import.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/fenetre-import.png)

 

Vous pouvez changer les chemins de sorties des fichiers **.sfa** (qui permet de créer le .sfb) et **.sfb** (qui est un format binaire optimisé de votre objet), mais ce n'est pas nécessaire ni conseillé. Cliquez donc sur finish : vos fichiers .sfa et .sfb on été créés !

Le **build.gradle** de l'application a aussi été **modifié**, il vous sera d'ailleurs certainement demandé de **synchroniser** les modifications, auquel cas faites-le.

Maintenant que nous avons tous nos éléments, nous pouvons retourner dans **MainActivity**. Supprimez le code créant le cube, et **remplacez** le par celui ci :

ModelRenderable.builder()
                .setSource(this, Uri.parse("3ie.sfb"))
                .build()
                .thenAccept(renderable -> mon\_modele = renderable);

 

Ici "3ie" est le nom de mon modèle 3D.

Opérez ensuite tout les changements découlant de cette modification. Voici ce que vous devriez obtenir :

public class MainActivity extends AppCompatActivity {

    private ArFragment arFragment;
    private ModelRenderable mon\_modele;
    

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity\_main);

        arFragment = (ArFragment)getSupportFragmentManager().findFragmentById(R.id.ux\_fragment);

        ModelRenderable.builder()
                .setSource(this, Uri.parse("3ie.sfb"))
                .build()
                .thenAccept(renderable -> mon\_modele = renderable);

        arFragment.setOnTapArPlaneListener(
                (HitResult hitResult, Plane plane, MotionEvent motionEvent) -> {
                    if (mon\_modele == null) {
                        return;
                    }
                    if (plane.getType() != Plane.Type.HORIZONTAL\_UPWARD\_FACING) {
                        return;
                    }

                    //On créé le point d'encrage du modèle 3d
                    Anchor anchor = hitResult.createAnchor();
                    AnchorNode anchorNode = new AnchorNode(anchor);
                    anchorNode.setParent(arFragment.getArSceneView().getScene());

                    //On attache ensuite notre modèle au point d'encrage
                    TransformableNode Node = new TransformableNode(arFragment.getTransformationSystem());
                    Node.setParent(anchorNode);
                    Node.setRenderable(mon\_modele);
                    Node.select();

                }
        );

    }

 

Vous pouvez **compiler** et **tester**, c'est maintenant **votre modèle** que vous intégrez à la scène.

\[youtube https://www.youtube.com/watch?v=0pYYCKIbg7c&w=560&h=315\]

 

Le code est disponible sur le GitHub de 3ie :

[https://github.com/3IE/tutoriel\_arcore](https://github.com/3IE/tutoriel_arcore)

* * *

### Pour conclure :

L'ARCore est un outils pratique est relativement facile a prendre en main. Cependant, l'ARCore seul est pour le moment assez limité. Pour pouvoir élargir massivement le champ des possibles, l'utilisation d'un logiciel supplémentaire, comme Unity par exemple est nécessaire. Ainsi il est possible d'aller beaucoup plus loin et

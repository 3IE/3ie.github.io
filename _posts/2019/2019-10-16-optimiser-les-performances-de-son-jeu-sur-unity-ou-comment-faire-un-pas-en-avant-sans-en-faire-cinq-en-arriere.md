---
title: "Optimiser les performances de son jeu sur Unity (ou comment faire un pas en avant sans en faire cinq en arrière)"
date: "2019-10-16"
categories: 
  - "technical"
tags: 
  - "optimisation"
  - "performance"
  - "unity"
---

Les développeurs de jeux vidéo se heurtent à de nombreux problèmes. Et celui de l’optimisation est sans aucun doute l’un des plus prévalents. En effet, sans optimisation, un jeu tel que The Last of Us (un tour de force d’optimisation sur la PS3) ne tournerait qu’à quelques frames par seconde (dans le cas où le jeu arriverait à tourner). Mais cette question se pose alors : “Comment optimiser mon jeu ? Et comment identifier les potentiels bouts de code le ralentissant ?”.

Eh bien on va voir tout ça ensemble (la majorité des parties de cet article concernent les jeux Unity sur PC, sauf pour la partie GearVR)

## **Let’s go on an Update**

La méthode Update est l’une des méthodes spécifiques à Unity les plus utiles : faire bouger un personnage ? Update peut le faire. Vérifier qui passe à travers un Raycast ? Update s’en charge. Guérir les blessures et assurer le retour de l’être aimé ? Update est là pour vous !

Mais malgré tout cela, cette méthode est coûteuse, elle est appelée à chaque frame (60 FPS = 60 appels d’Update par seconde). Mais ce n’est pas tout ! Lorsque Update est appelée, Unity lance plusieurs vérifications sur l’état du GameObject (détruit, inactif, etc.). Et bien que ces vérifications ne sont pas très coûteuses en elles-même, le fait d’avoir une centaine de GameObjects qui appellent tous la méthode Update à chaque frame devient vite gourmand en performances…

Il y a plusieurs méthodes pour limiter la perte de performances liée à la mauvaise utilisation d’Updates. La plus évidente est de limiter leur utilisation, mais on doit aussi s’assurer d’éviter de lancer du code inutile (faire des checks pour s’assurer qu’on n’exécute pas du code qui ne fera rien). De plus, il faut absolument ne pas laisser des méthodes Update vides dans les scripts ! Comme dit plus haut, le simple fait d’appeler la méthode Update sous-entend qu’Unity va faire plusieurs vérifications sur l’état actuel du GameObject sur lequel le script est attaché.

\[caption id="attachment\_2280" align="alignnone" width="1084"\][![](/assets/images/Update.png)](shorturl.at/euEGQ) Manager killed the Update star.  
Source : shorturl.at/euEGQ\[/caption\]

Ci-dessus on peut voir la différence de performances entre 10000 instances d'un script appelant chacune la méthode Update. Et 10000 instances d'un script contenant une méthode appelée "UpdateMe" et un script qui appelle cette méthode pour chaque instance via la méthode Update. Comme on peut le voir, le gain de performances est flagrant.

## **Tell, don’t show**

Il est pratique courante dans le développement de jeux vidéo de ne pas “render” les éléments qui ne sont pas visibles par le joueur, mais qu’est-ce que le “rendering” ?

Le rendering sert à afficher un élément du jeu. Par exemple, un monstre aura un mesh (sa forme) et une texture (sa couleur, détails, etc.), ainsi qu’une lumière qui agira sur lui. Le rendering récupère toutes ces informations et affiche cet élément pour le joueur. Mais cela a un coût, et sur des gros environnements avec de nombreux éléments de décor et PNJs, ça peut vite ralentir les performances.

\[caption id="attachment\_2290" align="alignnone" width="480"\][![](/assets/images/HZD_Opti.gif)](https://blog.3ie.fr/wp-content/uploads/2019/07/HZD_Opti.gif) La méthode de rendering d'Horizon Zero Dawn  
Source: shorturl.at/loDSW\[/caption\]

Ici on peut voir le système de rendering du jeu Horizon Zero Dawn. On se rend compte que tout ce qui n’est pas dans le champ de vision du joueur (avec un peu de marge) n’est pas affiché pour améliorer la vitesse de calcul.

Il est important de préciser que bien que les éléments ne soient pas affichés, cela ne veut pas pour autant dire qu’ils ne peuvent plus interagir entre eux. Après tout, si on pouvait échapper à une créature simplement en ne la regardant plus, le jeu serait bien simple…

\[caption id="attachment\_2282" align="alignnone" width="850"\][![](/assets/images/OcclusionCullingOverdrawNoCulling.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/OcclusionCullingOverdrawNoCulling.jpg) Tout est render.\[/caption\]

\[caption id="attachment\_2283" align="alignnone" width="850"\][![](/assets/images/OcclusionCullingOverdrawReducedNew.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/OcclusionCullingOverdrawReducedNew.jpg) Seulement les éléments visibles sont render.  
Source: shorturl.at/eCET5\[/caption\]

Comme on peut le voir, en ajoutant de l'Occlusion Culling (le nom de la technique qui ne render que ce qui est visible par la caméra), on observe une nette réduction des triangles ainsi que du nombre de batches.

## **Oh my LOD**

De nombreux jeux AAA (l’équivalent d’un blockbuster dans le monde du développement de jeux vidéo) se targuent d’avoir un niveau de détail jamais vu auparavant.

\[caption id="attachment\_2284" align="alignnone" width="1038"\][![](/assets/images/GOW_Details_Captain_Eggcellent.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/GOW_Details_Captain_Eggcellent.jpg) On pourrait presque sentir le musc de Kratos™  
Source: shorturl.at/nDQS5\[/caption\]

Et en effet, lorsqu’un joueur décide de s’attarder sur un élément du décor, on veut que le joueur puisse avoir la version la plus attirante (oserai-je dire “sexy”) de cet élément. Cependant lorsque le joueur regarde un arbre se situant à 50 mètres, il est peu probable qu’il puisse apprécier tous les détails les plus fins de son écorce (à ma connaissance, les hybrides humains/aigles n’ont pas encore étés créés). Il est donc malin de faire en sorte d’afficher une version moins détaillée de cet arbre.

C’est à ce moment-là que la technique des LODs entre en jeu.

\[caption id="attachment\_2285" align="alignnone" width="1046"\][![](/assets/images/LOD_Brackeys.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/LOD_Brackeys.jpg) Les singes de la sagesse ont bien changés...  
Source: shorturl.at/lEPWZ\[/caption\]

LOD veut dire Level Of Detail. Cette technique prend en compte la distance du joueur (la caméra principale) par rapport aux éléments autour de lui. Plus l’élément est loin, moins il a besoin d’être détaillé, moins le modèle contiendra de triangles (l’unité de mesure du niveau de détail d’un objet 3d). Cette technique est largement utilisée dans n’importe quel jeu comprenant des graphismes en 3d ainsi que des environnements ouverts.

## **Finders Loaders**

Dans le même genre que la méthode Update qui peut être coûteuse si mal utilisée. Voici un florilège de méthodes qui sont coûteuses dans tous les cas.

#### Find()

La première suspecte est la méthode Find(), pour des raisons évidentes. En effet, cette méthode implique de parcourir l’entièreté des GameObjects et des Components présents dans la scène. Sur une petite scène, le problème n’est pas forcément visible. Mais sur une scène contenant plusieurs centaines de GameObjects, et si la méthode est appelée fréquemment, les performances vont en souffrir.

Malgré tout, cette méthode est très utile dans certains cas. Mais plutôt que de l’appeler trop souvent, il vaut mieux stocker le résultat dans une variable pour la réutiliser plus tard si possible.

#### Resources.Load()

Ensuite nous avons la méthode Resources.Load(), permettant d’aller directement chercher un asset dans le dossier “Resources”. On y spécifie le chemin d’accès de l’asset (relatif au dossier “Resources” bien sûr) et la magie opère ! Mais comme toute magie qui se respecte, il faut des ingrédients. Dans ce cas-là, le seul ingrédient nécessaire, c’est votre puissance de calcul !

Comme Find(), cette méthode est puissante et très utile. Cependant il vaut mieux créer des variables pour stocker les assets. Ou bien créer une sorte de base de données d’assets grâce aux Scriptable Objects.

#### Autres

Il existe aussi bien d’autres méthodes coûteuses. On peut par exemple citer Camera.main, qui sert à récupérer la caméra principale de la scène (ce qui est basiquement un Find()). Les variables Vector2 et Vector3 sont aussi assez gourmandes, mais uniquement si elles sont utilisées de manières vraiment abusives. En effet, il est important de garder en tête le fait qu’un Vector3 est bien plus complexe qu’un simple float ou int.

Transform est aussi coupable d’être coûteux dans certaines circonstances, notamment lorsqu’on essaye de récupérer la valeur contenue dans Transform.position. Cet accessor calcule la “world position” du Transform. Pour éviter d’avoir trop de calculs inutiles, il vaut mieux utiliser la méthode Transform.localPosition qui est simplement stockée directement dans le Component.

En conclusion, ces méthodes ne sont pas à interdire à proprement parler. Mais il faut cependant faire attention à ne pas en abuser.

## **Let there be light !**

On va maintenant passer à un côté assez sous-estimé du jeu vidéo : la lumière. Lorsqu’on leur demande de citer les éléments les plus importants d’un jeu, une grande majorité des joueurs oublient de mentionner la lumière. Et pourtant une bonne gestion de la lumière dans un jeu peut faire la différence entre un bon jeu et un chef-d’oeuvre.

\[video width="640" height="360" mp4="https://blog.3ie.fr/wp-content/uploads/2019/07/FluffyAlienatedDodobird-mobile-1.mp4" loop="true"\]\[/video\]

[Source: Vicious Attack Llama Apocalypse](https://gfycat.com/fluffyalienateddodobird-unity3d-gaming)

Prenons par exemple un jeu qui a particulièrement marqué les joueurs par sa direction artistique : Ori and the Blind Forest.

\[caption id="attachment\_2286" align="alignnone" width="1031"\][![](/assets/images/Ori_Lighting_Steam.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/Ori_Lighting_Steam.jpg) C'est beauuuu...  
Source: shorturl.at/nCVY5\[/caption\]

Sur cette image, on peut voir Ori devant un autel entouré de flammes. Maintenant imaginez cette scène avec une lumière assez moyenne, c’est moins attirant tout de suite. Peu importe si vous avez les meilleurs modèles 3D pour vos personnages. Peu importe si vos textures sont le travail de littéraux dieux. Si votre lumière est mal foutue voire carrément incohérente, votre jeu semblera laid.

Enfin, assez d’introductions, parlons donc de la lumière sur Unity et de son optimisation !

Unity permet d’utiliser 6 variétés de lumières : Directional, Point, Spot, Area, Ambient et les Emissive Materials. Chaque type de lumière a sa spécificité, ses avantages et ses inconvénients. Mais ce qui nous intéresse c’est les setups de lumière.

Unity permet de rendre une lumière statique (c’est-à-dire qu’elle ne bougera pas) ou dynamique (inverse de statique, je ne pense pas qu’il y ait besoin de plus expliquer). Les lumières dynamiques sont très utiles pour faire une lampe torche ou n’importe quel objet diffusant de la lumière que le joueur peut prendre. Mais cela implique que le moteur de jeu doit calculer la lumière et les ombres qui sont générées par le biais de cette lumière à chaque frame, c’est coûteux…

#### Lightly baked

C’est pourquoi si une lumière ne bougera pas (un lampadaire ou autres), on indique que cette lumière est statique et on spécifie que son mode de rendu est du type Baked, ce qui empêche au moteur de faire trop calculs. On indique aussi tous les éléments sur lesquels on générera des ombres en tant qu’objets statiques. Après avoir fait le “bake” de la lumière, on aura une jolie scène illuminée. Tout cela coûtera assez peu en performances par rapport à la version dynamique.

Le bake est une technique de rendu de lumière. Elle permet de précalculer la lumière d’une scène en la stockant dans une Lightmap. Lightmap qui sera chargée lors de l'exécution du jeu.

Malheureusement, si un objet dynamique se trouve dans la scène, il ne pourra pas recevoir les lumières du type Baked. Mais il existe un moyen de combiner les deux modes : le mode de rendu Mixed. Ce mode permet de bake la scène tout en permettant aussi à la lumière directe d’illuminer les objets dynamiques. Le tout en restant plus performant que le pur dynamique Realtime qui lui calcule aussi la lumière indirecte en temps réel.

Un autre moyen de résoudre le problème des objets dynamiques sur une lumière statique est de placer des Light Probes dans la scène. Ces Light Probes vont stocker l’information de la lumière passante et vont donc permettre de répliquer la lumière statique en runtime.

#### Tout n'est pas si lumineux...

Mais mettre ses lumières en Baked n’est pas pour autant la solution miracle. Si la performance sera meilleure pour le joueur car elle aura été calculée en amont, il faut tout de même générer les Lightmaps. Ce calcul peut prendre beaucoup de temps suivant la taille de votre scène (un bake de 20-30 minutes est courant dans les scènes les plus grandes). C’est donc à vous de décider si cette méthode vous convient.

\[caption id="attachment\_2293" align="alignnone" width="660"\][![](/assets/images/Bake_Opti.png)](https://blog.3ie.fr/wp-content/uploads/2019/07/Bake_Opti.png) En haut: Le temps de précalcul d'une scène sans optimisation de bake  
En bas: Le temps de précalcul avec optimisation de bake  
Source: shorturl.at/hBJYZ\[/caption\]

Pour plus d'informations sur les techniques de lumière, [voyez ici](http://shorturl.at/dksx3).

La lumière est un outil puissant pour la fidélité graphique ainsi que l’immersion. Mais elle a tendance à être assez gourmande en performances si on ne fait pas assez attention.

## **Réalité Virtuelle, problèmes réels**

Plus que les jeux classiques, la VR demande que les jeux et expériences qu’elle propose soient très bien optimisés. En effet, pour arriver à faire croire au cerveau que ce qu’il expérience est réel, il faut que la framerate soit haute et assez constante (90 FPS est considéré comme le standard). Si elle est trop faible, alors les joueurs risquent d’être dérangés, voire d’avoir des maux de tête.

\[caption id="attachment\_2288" align="alignnone" width="800"\][![](/assets/images/headache-man-senior-002.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/headache-man-senior-002.jpg) Quelqu'un jouant à un jeu VR à 30 FPS  
Source: Harvard Health Publishing\[/caption\]

C’est donc pourquoi les développeurs VR tiennent tout particulièrement à bien optimiser leurs jeux. Car la seule chose plus triste que de créer un mauvais jeu, est que ce jeu donne envie de rendre son petit-déjeuner.

Fort heureusement, la plupart des jeux VR sont développés sur Oculus, Vive et autres casques spécifiquement dédiés au PC, plateforme réputée pour sa puissance. Jeu trop gourmand ? Hop ! On augmente les prérequis recommandés pour faire tourner le jeu, et tout est bien dans le monde (jusqu’à un certain point, on ne peut pas tous avoir deux cartes graphiques hi-tech, un watercooling et assez de puissance de calcul pour faire fonctionner l’intégralité de la NASA).

Cependant, pour ce qui est du Gear VR, un système de réalité virtuelle spécifique aux mobiles, c’est une autre paire de manches. Il faut faire tourner un jeu VR (donc en 3d avec des graphismes un minimum sympa) sur, au mieux, un Galaxy S9+. Autant dire que c’est l’équivalent de faire un portage de The Witcher 3 sur DS (un peu exagéré, mais pas tant que ça).

Comment on se débrouille alors ?

#### VR mobile, problèmes horribles

Eh bien, à part le fait de respecter les conseils d’optimisation indiqués plus haut, il faut faire des sacrifices. Oui, votre jeu ne contiendra pas des graphismes dignes d’un Red Dead Redemption 2. Oui, il n’aura peut-être pas la richesse et la variété de gameplay d’un GTA V. Mais au moins, il tournera au-dessus de 60 FPS (mettons 50, pour être gentils). Et il empêchera une large partie de votre audience de maudire votre famille et votre chien sur cinq générations.

Privilégiez les modèles simples ou low-poly (moins de triangles = meilleures performances), ou alors réduisez leur nombre dans vos scènes. Limitez les effets visuels et suivez les conseils, et votre jeu devrait être fluide.

Il est aussi important de noter que la transparence en 3d sur des mobiles est très coûteuse. Il vaut donc mieux l’utiliser avec parcimonie, voire pas du tout. Si jamais vous avez de nombreuses textures à charger, il est probable que le GPU d’un mobile ne supporte pas le trop grand nombre de drawcalls (le nombre d’objets qui sont “dessinés” dans la scène). Il peut donc être sage de combiner toutes ces textures dans ce qui est appelé un atlas, une sorte de grosse texture qui sert pour toute la scène. Gardez aussi en tête que chaque matériau créé un drawcall en plus, il vaut mieux les garder au minimum.

Malgré tout ça, votre jeu ne tournera pas sur tous les mobiles. En effet, le marché mobile étant toujours en mouvement, les smartphones deviennent vite obsolètes. Certains seront plus prompts à la surchauffe (particulièrement fréquente avec la VR), sans parler des niveaux d’API minimum requis pour faire fonctionner certains jeux. Vous n’aurez donc jamais une couverture totale des appareils existants.

## **En bref**

En bref, l’optimisation est pratiquement inévitable lorsqu’on crée un jeu. Que ce soit un AAA qui nous fera questionner la qualité des détails du monde réel. Ou bien un jeu indé fait par une jeune équipe ayant travaillée d’arrache-pied. Une mauvaise optimisation peut vite enrager les joueurs et empêcher ce qui aurait pu être un excellent jeu de montrer tout son potentiel.

Il est aussi important de préciser qu’un jeu mal optimisé pourra tout de même tourner correctement sur certaines machines. Mais les prérequis pour qu’il soit jouable seront bien plus élevés qu’un jeu correctement optimisé, ce qui risque d’aliéner une partie des joueurs.

#### Optimisation : The Good, The Bad, and The Ugly

\[caption id="attachment\_2297" align="alignnone" width="960"\][![](/assets/images/Doom-2016.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/Doom-2016.jpg) Good boy  
Source: shorturl.at/fFVZ9\[/caption\]

\[caption id="attachment\_2296" align="alignnone" width="963"\][![](/assets/images/ARK.jpg)](https://blog.3ie.fr/wp-content/uploads/2019/07/ARK.jpg) Bad boy  
Source: shorturl.at/exFGY\[/caption\]

Voici quelques exemples de jeux dont l’optimisation pouvait laisser à désirer :

- ARK (PC) est un exemple qui continue de faire parler de lui dans la communauté du jeu. Le fait que les développeurs ne soient pas réputés pour corriger leurs problèmes n’arrange pas le mécontentement des joueurs non plus. Atlas, un autre jeu créé par le studio ayant fait ARK, souffre lui aussi des mêmes tares. ([https://www.pcgamer.com/ark-survival-evolved-is-the-new-crysis-of-pc-hardware/](https://www.pcgamer.com/ark-survival-evolved-is-the-new-crysis-of-pc-hardware/))
- Kingdom Come: Deliverance (PC), le jeu ultra-réaliste se déroulant durant le Moyen-Âge. A peut-être poussé le réalisme un peu trop loin lorsqu’on tente de passer dans les paramètres de qualité “High” et “Ultra”. Représentant de manière précise le ralentissement d’un esprit humain lorsque le flux d’informations est trop élevé. ([https://www.pcgamer.com/kingdom-come-deliverance-performance-analysis/](https://www.pcgamer.com/kingdom-come-deliverance-performance-analysis/))

Et certains exemples de jeux dont l’optimisation a été faite avec attention et amour :

- God of War (PS4). Monde ouvert. Magnifiques textures. Des graphismes à couper le souffle. Le tout en 30 FPS constants sur un hardware datant de 2013. Honnêtement, je suspecte que toute l’équipe de Sony Santa Monica a fait un pacte avec un démon, car pour moi, c’est de la magie noire.
- Doom 2016 (PC). Visuellement beau (et inspiré). Et le jeu tourne sans aucuns problèmes sur des PC middle end (même sur une Switch). ([https://www.pcgamer.com/doom-benchmarks-return-vulkan-vs-opengl/2/](https://www.pcgamer.com/doom-benchmarks-return-vulkan-vs-opengl/2/))

Et ce ne sont que quelques exemples. Il existe de nombreux jeux extrêmement bien optimisés (et l’inverse est vrai aussi). Il suffit juste d’un peu de réflexion avant de produire du code, des modèles ou des textures. Réfléchissez au scope de votre jeu, aux plateformes sur lesquelles il sera présent. Et n’oubliez pas qu’un jeu souffrira d’une mauvaise optimisation, peu importe ses autres qualités.

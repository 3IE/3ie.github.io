---
title: "Unity NavMesh"
date: "2019-01-18"
categories: 
  - "technical"
tags: 
  - "unity"
---

Depuis Unity 5, implémenter un algorithme de recherche de plus court chemin n’est plus nécessaire grâce à l’ajout des composantes NavMesh. Ces composantes permettent de configurer notre agent afin qu’il trouve le meilleur chemin de sa position à une destination donnée.

Quatre de ces composantes ([NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html), [NavMeshModifier](https://docs.unity3d.com/Manual/class-NavMeshModifier.html), [NavMeshModifierVolume](https://docs.unity3d.com/Manual/class-NavMesh-ModifierVolume.html) et [NavMeshLink](https://docs.unity3d.com/Manual/class-NavMeshLink.html)) ne sont pas inclues dans l’installateur Unity3D mais vous pouvez les trouver sur le [GitHub officiel d'Unity](https://github.com/Unity-Technologies/NavMeshComponents).

L'utilisation de ces composantes pour de la recherche de chemin repose sur la structure de donnée [mesh de navigation](https://en.wikipedia.org/wiki/Navigation_mesh). Cette méthode est répandue dans l'industrie du jeu vidéo. Elle permet de trouver le chemin le plus court réel (pas celui d'un graphe), réduit le temps de calcul et l'empreinte mémoire dans les environnements ouverts. Elle permet de gérer plus facilement les différents gabarits des agents et les obstacles dynamiques.

Nous allons aborder quelques bases pour l'utilisation des composantes NavMesh dynamiquement.

### Vue d'ensemble:

Nous allons commencer par une présentation rapide des deux composantes essentielles à l'utilisation des composantes de navigation.

#### NavMeshSurface:

C'est cette composante qui définit les endroits accessibles et inaccessibles dans notre environnement. Elle est essentielle car sans cela l'agent ne peut savoir par où passer. L'environnement se configure très facilement depuis l'éditeur grâce à cette composante. Pour ce faire, il suffit d'appuyer sur le bouton Bake après avoir configuré les paramètres comme souhaité. Les paramètres sont expliqués dans la [documentation](https://docs.unity3d.com/Manual/class-NavMeshSurface.html). Je tiens tout de même à parler de 2 de ses paramètres:

- Collect Objects: Ce paramètre permet de choisir les GameObjects à utiliser pour configurer l'environnement. On peut notamment lui attribuer la valeur Children qui permet de n'utiliser que les GameObject étant enfant du GameObject portant la composante
- Include Layers: Ce paramètre permet de choisir les couches dans lesquelles on va choisir les GameObject pour la configuration.

L'association de ces deux paramètres permet une configuration précise de l'environnement.

\[gallery columns="1" size="full" ids="1926"\]

#### NavMeshAgent:

Cette composante va permettre à l'agent de se déplacer. Un agent ne peut se déplacer si l'environnement n'a pas été configuré grâce à la composante [NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html). Elle possède différents paramètres définis dans la [documentation](https://docs.unity3d.com/560/Documentation/Manual/class-NavMeshAgent.html) et est facilement manipulable à travers des scripts.

[![](/assets/images/NavMeshAgent-236x300.png)](https://blog.3ie.fr/wp-content/uploads/2018/07/NavMeshAgent.png)

 

#### NavMeshModifier:

Cette composante permet d'ajuster le comportement d'un agent. Imaginons que vous souhaitiez configurer la navigation d'un ennemi et qu'il y a des zones que vous souhaitez qu'ils évitent mais peuvent utiliser en dernier recours. Cette composante peut permettre de définir la zone à éviter, l'agent l'évitera donc sauf s'il n'a pas le choix. Pour en savoir plus sur la composante et ses paramètres vous pouvez vous référer à la [documentation](https://docs.unity3d.com/Manual/class-NavMeshModifier.html). Cette composante se base sur la hiérarchie des composantes Transform.

\[gallery columns="1" size="full" ids="1954"\]

#### NavMeshModifierVolume:

Cette composante a le même but que celle du dessus sauf qu'elle se base sur le volume. Vous pouvez vous référer à sa [documentation](https://docs.unity3d.com/Manual/class-NavMesh-ModifierVolume.html) pour une étude plus approfondie.

\[gallery columns="1" size="full" ids="1955"\]

#### NavMeshLink:

Cette composante permet de lier deux endroits qui utilisent la composante [NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html). Elle permet de lier différentes surfaces ou une surface à elle-même. Comme montré dans la [documentation](https://docs.unity3d.com/Manual/class-NavMeshLink.html), elle est utile pour l'utilisation de sauts par exemple.

\[gallery columns="2" size="full" ids="1950,1951"\]

### Utilisation dynamique:

Imaginons que nous souhaitions utiliser ces composantes sur un environnement généré dynamiquement. Le problème survient au niveau de la composante [NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html) car nous devons configurer l'environnement à travers un script. Heureusement nous avons accès à la méthode BuildNavMesh() qui appartient à la composante [NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html). Cette méthode reproduit le comportement du bouton Bake.

Après différents tests, j'en ai conclu que :

- Le script interagissant avec la composante [NavMeshAgent](https://docs.unity3d.com/560/Documentation/Manual/class-NavMeshAgent.html) doit être lancé après que l'environnement soit prêt, c'est-à-dire après que la méthode BuildNavMesh() est été utilisée. Je vous conseille de créer votre propre méthode pour remplacer la méthode standard Start() dans le script de votre agent. En effet lorsqu'on instancie un GameObject, s'il a un script attaché, la méthode standard Start(), usuellement utilisée pour initialiser le script, va être appelée. Voici un exemple de substitution:
    
    public Camera cam;
    
    public NavMeshAgent agent;
    
    /\*
      La méthode Init() remplace la méthode standard Start(), elle est en public pour qu'elle
      être appelée à partir d'un autre script.
    \*/
    
    public void Init()
    {
        cam = Camera.main;
        agent = GetComponent<NavMeshAgent>();
        agent.enabled = false;
    }
    
- L'environnement peut être configuré dynamiquement qu'une seule fois par scène. C'est-à-dire que la méthode BuildNavMesh() ne sera ré-utilisable seulement s'il y a un changement de scène.

Pour illustrer le tout, j'ai réalisé un projet mettant en scène l'utilisation des composantes NavMesh dynamiquement. J'utilise uniquement les composantes [NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html) et [NavMeshAgent](https://docs.unity3d.com/560/Documentation/Manual/class-NavMeshAgent.html). Mon environnement est créé sous un GameObject parent qui possède le [NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html). Ci-dessous la configuration de mon [NavMeshSurface](https://docs.unity3d.com/Manual/class-NavMeshSurface.html) et de mon [NavMeshAgent](https://docs.unity3d.com/560/Documentation/Manual/class-NavMeshAgent.html):

\[gallery columns="2" size="full" ids="2022,2023"\]

[Mon script](https://github.com/Pleuvens/NavBoy/blob/develop/Assets/Scripts/Maze/Maze.cs) s’exécute comme suit:

1. Je génère mon environnement. Pour ma part, c'est un labyrinthe, j'instancie donc le sol et le murs.
2. Mon algorithme de génération aléatoire de labyrinthe me permet de connaître la position de départ et la position d'arrivée. J'instancie donc l'agent à la position de départ et le portail à la position d'arrivée. L'agent est instancié mais aucun code ne s'exécute car, comme dis ci-dessus, [je n'ai pas utilisé la méthode standard Start()](https://github.com/Pleuvens/NavBoy/blob/develop/Assets/Scripts/Boy/BoyBehaviour.cs).
3. Je fais appelle à la méthode BuildNavMesh() équivalente à Bake.
4. J'appelle ma méthode remplaçant la méthode standard Start() qui me permet entre autres d'assigner des valeurs à différents attributs dont l'attribut de type [NavMeshAgent](https://docs.unity3d.com/560/Documentation/Manual/class-NavMeshAgent.html).
5. Enfin j'indique la position d'arrivée à l'agent grâce à la méthode SetDestination() définie dans la composante [NavMeshAgent](https://docs.unity3d.com/560/Documentation/Manual/class-NavMeshAgent.html)
6. Lorsque l'agent atteint l'arrivée, [je relance la scène](https://github.com/Pleuvens/NavBoy/blob/develop/Assets/Scripts/Manager/FlowManager.cs) au lieu de reconstruire l'environnement afin de pouvoir réutiliser la méthode BuildNavMesh(). Si je ne relance pas la scène, malgré que l'environnement soit différent, il gardera la configuration précédente.

Voici les lignes importantes et l'ordre dans lesquels elles doivent être exécutées

/\*
  Les attributs surface et player sont définies plus haut et représentent respectivement
    un NavMeshSurface et un GameObject.
  BoyBehaviour est le script s'occupant du comportement de l'agent.
  La méthode Init() remplace la méthode standard Start() et MoveToDestination 
    indique la destination au composant NavMeshAgent de l'agent.
\*/

surface.BuildNavMesh();
player.GetComponent<BoyBehaviour>().Init();
player.GetComponent<BoyBehaviour>().MoveToDestination(destination.transform.position);

Si vous souhaitez voir le code source du projet dans son intégralité vous pouvez regarder mon [dépôt GitHub](https://github.com/Pleuvens/NavBoy/tree/develop).

Voici le résultat:

\[gallery columns="1" size="full" ids="1920"\]

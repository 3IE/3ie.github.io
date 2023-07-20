---
title: "Projet étudiant : Wheelchair-VR"
date: "2021-12-01"
categories: 
  - "nos-projets"
tags: 
  - "vr"
  - "wheelchair-vr"
---

La VR est un outil parfait pour se plonger dans des aventures extraordinaires ou pour se former professionnellement. Wheelchair-VR est un projet différent, pas d'aventure extraordinaire ou d'apprentissage de prévu. C'est une courte aventure banale comme nous pourrions en vivre tous les jours. Le twist, c'est que nous la vivons du point de vue d'une personne se déplaçant en fauteuil roulant.

### **Raison du projet** 

L'objectif principal de ce projet est de mettre en avant les problèmes d'accessibilité que peuvent rencontrer les personnes utilisant un fauteuil roulant. En mettant le joueur dans la peau d'une personne qui rencontre ces problèmes, nous souhaitons qu'il se rende compte de l'omniprésence d'obstacles dans la vie de tous les jours des personnes à mobilité réduite.

Le scénario est découpé en deux parties. Le joueur commence dans sa maison, un environnement adapté à son fauteuil. Il doit réaliser des tâches simples avant d’aller au travail (manger, prendre son sac à dos, etc.). Une fois au bureau, il doit suivre les ordres de son boss et ramener rapidement un dossier important sous peine de sanction. L’environnement de travail comporte des obstacles pour un usager en fauteuil (porte qui s’ouvre vers lui, objets en hauteur à attraper, etc.). L’intérêt de ces deux scènes est de mettre en avant les différences entre un environnement adapté et un qui ne l’est pas. Cela montre au joueur que des éléments simples et anodins comme une porte peuvent vraiment changer l’accessibilité d’un lieu.

### **Challenges techniques**

Lors du développement, notre première tâche fut de mettre en place des déplacements en fauteuil roulant. Nous voulions quelque chose de réaliste physiquement pour vraiment immerger le joueur. Rapidement, nous nous sommes heurtés à un problème : la physique dans Unity.  
  
Unity propose un système de physique poussé mais qui possède ses limites. Une d’entre elles est le comportement de _rigidbodies_ (objet qui suit les lois de la physique) lorsqu'ils sont enfants d'autres _rigidbodies_. La vélocité du parent va influencer la vélocité de l'enfant qui elle-même est influencée par des éléments extérieurs (gravité, contacts, etc.). Cela peut rapidement créer des comportements imprévisibles. C’est une pratique qu'il est fortement déconseillé d’utiliser, la solution pour s’en passer est d’utiliser des _joints._

Les _joints_ permettent de lier deux objets physiques entre eux d'une certaine façon (fixe, avec un effet ressort, en rotation libre, etc.). Nous avons utilisé les _joints_ pour lier les roues au châssis du fauteuil. Nous nous sommes alors retrouvés avec plusieurs éléments qui s’imposent les uns les autres des contraintes de rotation, de position et de vélocité. Cela est vite devenu très complexe à gérer, et cela a créé des bugs que nous n’avons pas totalement réussis à résoudre. Les roues ont notamment parfois des mouvements illogiques, imprédictibles, ou tremblent sur place sans raison apparente.

Nous avons essayé de contrer ces problèmes avec des méthodes différentes à chaque itération. Le système a entièrement été revu plusieurs fois. La version finale a toujours des défauts, mais elle est convenable pour l'expérience que nous avons développée. Une solution optimale, mais qui peut aussi prendre beaucoup plus de temps, aurait été de se passer de la physique d'Unity pour coder soi-même les interactions entre les différentes parties du fauteuil.

![Wheelchair-VR - Vue FPS, un téléphone flottant affiche des objectifs](/assets/images/wheelchair_vr_phone-1024x575.png)

_Téléphone donnant les objectifs et servant de repère visuel pour limiter la motion sickness._

La physique n'a pas été notre seul défi technique, éviter la _motion sickness_ en fut un autre. La _motion sickness_ apparait lorsque notre oreille interne est en désaccord avec ce que nous voyons. Notre cerveau voit que l'on se déplace alors que notre corps ne le ressent pas. Habituellement, pour éviter cette sensation, les jeux utilisent la téléportation.

Dans notre cas, la téléportation aurait rapidement fait perdre son sens au jeu. Nous avons donc dû trouver un moyen de réduire la _motion sickness_ produite par les déplacements du fauteuil. Nous avons ajouté un repère visuel statique, un téléphone, face au joueur. Nous avons aussi fait en sorte que le fauteuil avance uniquement lorsque le joueur tourne les roues. L'inconvénient de cette méthode est la vitesse des déplacements qui se retrouve limitée par la vitesse des mouvements du joueur sur les roues.

### **Ressenti personnel**

Wheelchair-VR est mon premier projet VR. J’avais déjà l’habitude d’utiliser Unity pour des projets personnels, mais il m’a permis d’apprendre beaucoup, notamment sur la physique et le développement pour la réalité virtuelle. La VR demande de concevoir une expérience très différemment d’un jeu « classique ». Les joueurs ont plus de libertés et les actions qu’ils peuvent faire sont souvent plus complexes. De plus, la vitesse et les moyens de déplacement sont fortement limités à cause de la _motion sickness_.  

Grâce à la conception du fauteuil, j’ai appris à connaître la physique de Unity. La conception des déplacements a changé à plusieurs reprises, et s’est adaptée aux plugins dont nous disposions. Au départ plutôt arcade (_grab_ dans la zone de la roue avec un mouvement pour la faire tourner), les mouvements sont progressivement devenus plus réalistes en partie grâce à [Auto Hand](https://assetstore.unity.com/packages/tools/physics/auto-hand-vr-physics-interaction-165323) qui permet un placement simplifié des mains.

Pour mieux débuter Wheelchair-VR, j’aurais dû prendre le temps de m’entrainer sur un projet en VR plus court et moins technique. En prenant du recul, je me rends aussi compte que j’ai passé trop de temps sur le fauteuil. Entre chaque itération, je n’ai pas pris assez de temps pour poser toutes les solutions possibles de façon clair, pour choisir celle qui semblait la plus adaptée.

En dehors du fauteuil, l’implémentation de nouvelles mécaniques de jeu et la mise en place du scénario étaient un vrai plaisir à réaliser. Les échanges constructifs, et l’aide que m’ont apporté les membres de 3IE pour la conception ou pour résoudre des bugs, ont participé à rendre ce projet agréable. Sans eux, Wheelchair-VR ne serait pas ce qu’il est aujourd’hui.

![Wheelchair-VR - Un npc ordonne au joueur de lui ramener un document](/assets/images/wheelchair_vr_boss-1024x574.png)

_Le patron donne l'objectif de la seconde scène_.

### **Conclusion**

Wheelchair-VR est un projet qui tire profit de la VR pour sensibiliser les joueurs sur des problèmes importants d’accessibilité. Il fut très formateur, et m’a permis de découvrir le développement de jeu en VR avec Unity. Le résultat final est améliorable sur plusieurs aspects, autant techniques (fonctionnement du fauteuil) que ludiques (conception des niveaux et mécaniques de jeu). Nous avons eu de nombreuses idées durant son développement que nous n’avons pas eu le temps d’explorer. Il aurait notamment pu être intéressant de garder des contrôles arcade, et de réaliser des niveaux courts qui mettent en avant un problème d’accessibilité précis.

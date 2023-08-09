---
title: "Cas pratique d'adaptation d'une application VR vers la MR"
date: "2023-04-18"
categories: 
  - "non-classe"
tags: 
  - "realite-augmentee"
  - "realite-mix"
  - "realite-virtuelle"
---

La réalité virtuelle et la réalité mixte sont des domaines encore très jeunes. Certains de leurs aspects similaires peuvent naïvement donner l’impression qu’il s’agit grossièrement de la même chose. Dans cet article, nous verrons à travers un premier exemple concret, basé sur un projet réalisé durant mon stage, les détails cruciaux à garder en tête au moment de réaliser une application MR. Puis, un exemple plus complexe pour explorer quelques spécificités supplémentaires.  
Pour préciser, la réalité mixte peut être expérimenté avec différents paramètres en fonction de l’appareil utilisé (L’expérience en MR sur le casque Meta Quest 2 est très différente de celle sur un Hololens de Microsoft). On se concentrera sur l’expérience donnée par le casque Hololens 2.

![](https://lh3.googleusercontent.com/c93hl9Juo05TGQyeKdC-ZnexqGbpKhIYEuwiBfxZqCR9-IFtLlnS5jDyhuKHAmik_eFpVClWk5wFKPBTr_vp4NNII0qqK7otSoM7NB222bg5Sa3aOLT8kMUPpYP70eO-_0YUQE3mHR79lUxcNkYdEw)

## I. Virtual Meeting

#### A. Le projet

Il s’agit d’un projet réalisé au cours de mon stage de quatrième année ayant pour but de développer une expérience de partage entre la Réalité Virtuelle et la Réalité Mixte. Nous avons donc réalisé une application pouvant tourner sur les casques Quest 2 et Hololens 2. Les possibilités offertes sur les deux appareils étaient symétriques et les fonctionnalités étaient donc réfléchies à la base pour fonctionner sur les deux plateformes. Cela n’a pas manqué de nous faire voir des différences et nouvelles problématiques lors de l’implémentation de ces fonctionnalités.

![](https://lh5.googleusercontent.com/pJYZF2Dc-jY6iuvZaVmIVRt274ZUUaN6HyXbAsy8ZFQOxFU6BEpQ1nMjCLud1usHVEOXXl25aR-A4MdgU5C5TdIF_2i3sdGBVAzrDyKrgy-HQzc7TGUJJsHdIfzcuzDTpHk0NIi6dnzcAu9zoLke8Q)

#### B. Problématique

Un aspect spécifique à la Réalité Augmentée et Mixte qu’il était nécessaire d’adresser rapidement est la position de la dimension virtuelle par rapport à la dimension réelle. En Réalité Virtuelle, cette position est décidée à l’avance par l’utilisateur et reste fixe. Il n’y a pas besoin de la modifier étant donné que la dimension virtuelle obstrue la dimension réelle et l’utilisateur ne va normalement pas sortir du cadre préétabli. 

La Réalité Augmentée et Mixte existent avec comme principale différence que le cadre de la dimension virtuelle y est bien plus large est souple : l’utilisateur peut ainsi changer entièrement de lieu sans jamais en sortir. Il a donc fallu trouver un moyen de pouvoir aisément pouvoir définir et repositionner l’origine de cette dimension. Il fut donc décidé que toute instance de l’application comporterait un premier tableau à son origine et que modifier la position de celui-ci localement sur un des appareils revenait à redéfinir l’origine de sa dimension dans l’espace réel de l’utilisateur concerné.

Un autre aspect qui saute aux yeux est le champ de vision limité et une certaine difficulté pour l’utilisateur à évaluer la profondeur des objets 3D en vue. Le premier rend fréquent les cas ou un objet suffisamment large remplit tout le champ de vision de l’utilisateur et lui fait perdre ses repères dans l’application.

Une solution fut d’utiliser des textures à paterne géométrique (une grille par exemple) ainsi que des animations sur les doigts (un cercle qui se réduit au contact d’une surface) et les éléments interactifs, afin de mettre en valeur cette profondeur et l’évolution de la position des éléments entre eux. 

Un dernier détail à adresser durant ce projet est l’instabilité du traçage des mains, lié à la difficulté pour les caméras de l’Hololens à suivre les mouvements brusques, mais aussi leur champ de vision moyen les perdant fréquemment de vue. Les autres joueurs d’une instance multijoueur pouvaient voir les mains se figer de façon inopiné, brisant l’immersion.

Une partie de la solution se trouvait déjà dans le ToolKit de l’appareil, cela consistait à concevoir des animations latentes poussant le joueur à ralentir son mouvement pour accompagner l’objet. La seconde, au niveau multijoueur, consistait à simplement détecter les moments durant lesquels une main n’était plus tracée et envoyer une position par défaut immersive (le long du corps) sur le réseau.

## **II. Virtual Piano**

#### A. Le projet

Ce projet a été réalisé pour le casque Valve Index afin d’exploiter spécifiquement le traçage des doigts par ses manettes. Sur un piano virtuel, le joueur peut placer ses manettes au niveau du clavier pour que celle-ci s’aligne automatiquement le long des touches. Il peut ensuite abaisser chacun de ses doigts pour “appuyer” sur des touches et jouer ainsi. Dans mon précédent article à son sujet, j’ai évoqué comme piste alternative intéressante d’utiliser le traçage des doigts. Notamment parce que les Index Controllers ne traquent que le repliement de chaque doigt et non leur espacement sur chaque main par exemple.

![](https://lh6.googleusercontent.com/K_mIndvzCDluzwiVXAb0-cAkypdX2apY_MxlvTVEqjgE2XaFPPQhzBgOG2g34HxnO4ulpZYl6gZr8Qk-OVymXssFelFtMgJZydE6NZzsM2tTaAPYBbotjKMoVXG3kVnVYP2DUa-_77CaWbwn4Ygmlg)

#### B. Problématique

Une difficulté présente sur la version VR et qu’il faut probablement revoir en MR est la façon pour l’utilisateur de poser ses doigts sur le clavier du piano virtuel. Avec les manettes, je verrouille toutes les rotations de la main ainsi que ses translations verticales et en profondeur. Il n’est pas certain que de telle restriction sur les mains visualisée par le casque soit agréable: les rotations sur les axes Y et Z sont peut-être à déverrouiller au moins partiellement.

L’aperçu de l’espacement variable des doigts permet une meilleure souplesse dans l’appui des touches, mais il affecte aussi la précision du toucher. Si l’on ne compte pas sur l’apprentissage de l’utilisateur (ce problème n’est pas spécifique à un piano virtuel, essayer de jouer du véritable instrument), une solution peut être de traquer le regard de l’utilisateur afin de corriger les appuis lorsque l’utilisateur regarde où il souhaite jouer.

Une fonctionnalité importante de la VR pour permettre à l’utilisateur de bien se repérer lorsqu’il interagit avec des éléments virtuels est la capacité de retour haptique de chaque manette. Pouvoir remarquer l’appuie des touches et le déplacement de la main sur le clavier autrement que par la vision est très important pour améliorer la présence et permettre une meilleure maîtrise de l‘utilisateur à long terme, en lui permettant notamment de pouvoir plus facilement interagir avec des éléments en dehors de son champ de vision en se s’aidant du “touché”.

Cette fonctionnalité est absente en MR à la base, il est donc nécessaire de trouver autre chose, d’utiliser un autre sens que le toucher. Microsoft dans les menus de son casque Hololens utilise des sons stéréo très détaillés pour pallier cela. 

Pour aller plus loin, il serait judicieux d'exploiter les possibilités exclusives à la MR. Notamment la possibilité d'exploiter l'environnement réel de l'utilisateur. On peut ainsi imaginer pouvoir situer le piano virtuel avec une surface plate horizontale comme une table. Cela pourrait compenser l'absence de retour haptique native, voire même de dépasser de certaine manière celle des manettes VR: tapoter sur une table offre une sensation plus proche d'un appui de touches que le repliement des doigts sur la manette. Cela peut aussi compenser le manque de bouton accessible comparé à des manettes.

## III. Conclusion

Bien qu’il soit possible de réaliser une application fonctionnant à travers les deux plateformes avec une approche utilisateur similaire, les interactions doivent être réfléchies à l’avance avec les deux appareils en tête. La réalité mixte, telle que proposée par l’Hololens 2, se distingue de la réalité virtuelle par une interactivité réduite, mais une plus grande souplesse par la possibilité d’exploiter l’environnement réel, ainsi qu’une plus grande expressivité avec les gestes des mains et la direction du regard tracé.

Noé Topeza
<br>
<br>

---------------------------------------
<br>
Auteur: **owen.duffourg**

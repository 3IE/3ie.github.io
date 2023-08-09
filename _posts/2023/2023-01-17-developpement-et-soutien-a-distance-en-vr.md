---
title: "Développement et soutien à distance en VR"
date: "2023-01-17"
categories: 
  - "non-classe"
  - "nos-projets"
  - "technical"
tags: 
  - "unity"
  - "vr"
---

Nous nous sommes demandé s'il était possible et efficace, en réalité virtuelle (VR), de coder et de se faire aider par un encadrant. Les intérêts peuvent être multiples, comme pour un étudiant qui ne pourrait pas se rendre en cours parce qu'il est malade (la pandémie nous l'a bien montré) ou bien améliorer l'accessibilité à ce genre d'exercice.

Dans cet article nous allons aborder la démarche que nous avons eue et le prototype que nous avons réalisé au laboratoire MetaLab.

Nous prendrons en exemple un contexte connu à Epita, la Piscine, ce qui nous donnera un cadre pour cette expérimentation. La Piscine Epita reste une expérience humaine à vivre, une expérience qui se partage dans la réalité !

## Qu'est-ce qu'est la Piscine EPITA ?

La **Piscine** de l’Epita est une période intense de plusieurs semaines lors de laquelle la motivation, l’endurance, le goût pour l’informatique et la logique des étudiants sont testés. Les étudiants montent en niveau dans les matières concernées et c’est là aussi que se créent les plus belles solidarités.

Les étudiants résoudront individuellement des exercices dans différents langages. Ils pourront s’appuyer sur des assistants durant cette période.

![](/assets/images/piscine-1024x473.jpg)

Piscine EPITA 2022

## Les objectifs de l'expérimentation

Nous avons choisi d’expérimenter avec le casque de réalité virtuelle Oculus Meta Quest 2, car c’est le plus répandu sur le marché public. Il offre plusieurs avantages tels que l’autonomie (pas besoin d’être branché à un ordinateur), le hand tracking (suivie des mains, plus besoin de controllers) et keyboard tracking (suivi et appairage à un clavier).

Les objectifs sont : 

- Mettre en réseau plusieurs utilisateurs en VR et qu’ils puissent communiquer. 
- Pouvoir taper et visualiser du code.
- Partager son code à un assistant.

![](/assets/images/metaquest-1024x998.png)

Meta Quest 2 avec les controlles et un clavier Apple

## Networking

Pour permettre aux utilisateurs de l’application de rejoindre une salle machine virtuelle et de pouvoir se voir et communiquer, il faut mettre en place un système de Networking (mise en réseau). Il existe plusieurs solutions de networking pour Unity, ici nous avons utilisé Photon PUN 2, une solution clé en main, facile et rapide à implémenter.

![](/assets/images/header.png)

Grâce à cette mise réseau les utilisateurs peuvent se rejoindre dans une ‘Room’ commune. On observe les avatars en mouvement des autres utilisateurs (la position et rotation de la tête grâce au casque, et des mains grâce aux controllers). On communique vocalement via Photon Voice et le microphone intégré du casque VR, l’ensemble des audio de la scène sont spatialisés (en 3D).

![](/assets/images/room-1024x577.png)

## Les interactions

L’utilisateur à la possibilité de se déplacer dans la salle machine virtuelle via la téléportation (un des modes de déplacement VR les plus simple et des moins sujet au motion sickness (le mal des transports, nausées, mal de tête).

Lors de ses téléportations l’utilisateur peut choisir de s’installer sur un des postes de travail (qui simule un ordinateur et écran d’une salle machine). Une fois sur le poste il doit pouvoir taper du code et suivre ses exercices, mais comment taper du code en VR ? 

Nous avons le choix d’utiliser les controllers du Quest 2 (les manettes)  ou le hand tracking  (suivie des mains).Les controllers sont précis et permettent un retour haptique. Par contre, ils ne permettent pas de faire plusieurs tâches rapidement facilement (par exemple taper au clavier).Le hand tracking est possible sur le Quest 2 grâce à  ses quatres caméras réparties sur le casque. Il apporte de l’immersion et une manière plus intuitive d'aborder les tâches. Par contre, il manque de précision pour les actions fines et de retour haptique.

#### Clavier physique (virtuelle)

On reproduit un clavier virtuel avec de la physique. L’utilisateur pourrait alors venir taper sur chaque touche du clavier virtuel dans l’application. 

L’ avantage de cette solution est sa modulabilité. Nous pouvons adapter le clavier selon nos préférences. Nous pouvons interagir soit avec les controllers, soit avec le hand tracking. 

Les inconvénients de cette solution avec les controllers sont le manque de fluidité dans l’action. On perd du temps à regarder le clavier et taper correctement la bonne touche.  Le hand tracking n’est pas aussi pointu pour taper de manière précise sur une touche d’un clavier classique. Nous pourrions agrandir l’espacement des touches sur le clavier mais l’action devient plus contraignante, l’utilisateur utilise davantage ses bras et n’a plus de repère sur le clavier.

![](/assets/images/keyboardphysic-1024x444.png)

Clavier physique dans Unity

#### Clavier UI

On utilise le clavier intégré au Quest 2. Un clavier 2D en GUI (graphical user interface). L’utilisateur clique sur chaque caractère à l’aide d’un rayon prolongeant un de ses controllers. 

Les avantages de cette solution sont sa modulabilité et sa précision sur le clavier, nous pouvons y ajouter des mots clé pour éviter de taper chaque caractère.

L’inconvénient est sur la rapidité d’action, l’utilisateur doit constamment regarder son clavier et perd un temps précieux.

![](/assets/images/keyboard2d-1024x463.png)

Clavier UI system Unity

#### Clavier réel

On utilise un clavier réel et on saisie avec nos mains (comme en condition réelle). Lorsque l’utilisateur est installé à son poste il peut déposer ses controllers VR et passer en mode hand tracking automatiquement. Le Quest 2 permet d'appairer en Bluetooth certains claviers, le casque reçoit donc les inputs des touches. Les caméras détectent également la position du clavier, ce qui permet d’afficher le flux vidéo des caméras sur l’emplacement du clavier réel. 

Les avantages de cette solution sont que l’utilisateur voit ses mains sur son clavier réel et peut taper du code de façon intuitive. Elle permet d’avoir un retour haptique connu et une action familière. 

L’ inconvénient est que la résolution des caméras du Quest 2 ne sont pas entièrement satisfaisantes à ce jour ce qui rend la lecture du clavier et la position des doigts pas si simple. Le Quest Pro pourrait résoudre cette lacune.

![](/assets/images/HandTracking.gif)

Hand Tracking

![](/assets/images/Transitions.gif)

Transition hand tracking to passtrough

![](/assets/images/passthrough-1024x363.png)

Passthrough mains et clavier

L’utilisateur doit pouvoir appeler un assistant et communiquer avec lui. Cela se fait à l’aide d’un bouton mis à disposition sur les postes de travail. A l’appuie du bouton cela lance une demande d’aide aux assistants qui pourront alors le rejoindre en VR ou sur Ordinateur (l’avatar se contrôlera donc au clavier souris).

## Pouvoir exécuter du code et des outils Linux à distance

Pour permette à l’utilisateur de taper directement du code depuis son casque, il faut :

- soit pouvoir embarquer directement dans le casque un Shell et un compilateur pour les langages à utiliser 
- Soit pouvoir se connecter à une machine distante et se contenter d’envoyer les inputs de l’utilisateur et de récupérer les résultats

Embarquer tous les outils directement sur le casque semble compliqué. Nous avons testé des solution du WASM directement dans le browser du casque mais les performances et fonctionnalités n’étaient pas toujours au rendez vous. De plus, il est parfois nécessaire d’avoir une connexion internet car les composants se téléchargent au fur et à mesure des commandes.

Nous avons donc décidé d’utiliser une machine Linux distante.

Pour profiter des standards déjà présents dans Linux, nous avons décidé d'utiliser le plus possible de composants existant et de ne pas développer d’agents sur mesure pour communiquer avec le casque. Nous sommes donc partis sur une connexion SSH standard.

Comme Unity permet d’utiliser du code .Net, nous avons d’abord testé la bibliothèque [SSH.NET](https://github.com/sshnet/SSH.NET) pour créer un une connexion SSH. La lib fonctionne bien quand il s’agit d'exécuter une commande simple, avec réponse qui arrive en temps fini. Mais quand il s’agit de garder la connexion ouverte et d’émuler un terminal, c’est beaucoup moins facile et bien moins documenté. 

De plus, pour avoir dans le casque un terminal standard, type [VT-100](https://en.wikipedia.org/wiki/VT100), il aurait fallu implémenter les [ANSI escape code](https://en.wikipedia.org/wiki/ANSI_escape_code) pour la couleur et le déplacement du curseur.

Nous avons finalement décidé d’utiliser l’asset Unity [SSH Terminal Emulator](https://assetstore.unity.com/packages/tools/gui/ssh-terminal-emulator-185376) qui permet d’avoir un terminal SSH (et l’asset utilise justement la lib SSH.NET).  
Pour faciliter la mise en place de la solution, la machine Linux est simplement un container Docker basé sur une image [openssh-server](https://hub.docker.com/r/linuxserver/openssh-server).

## Aide à distance

L’utilisateur doit pouvoir partager son terminal aux assistants pour avoir relecture ou pour qu’ils y apportent des corrections.

Ayant accès à un vrai Linux en SSH, nous avons utilisé la commande screen qui permet de partager un terminal entre plusieurs processus. Ici cela permet à l’utilisateur Unity d’initier un terminal virtuel et ensuite à un assistant de s’y connecter, et de l’aider en temps réel.

## Quelles sont les conclusions observées ?

Le prototype de l’application fonctionne techniquement, un utilisateur peut coder et partager son code en VR. Cependant la technologie n’est pas encore au point pour le Keyboard tracking (suivie de clavier), la résolution de caméras n’aide pas à taper au clavier aussi fluidement qu’en condition réelle.  
  
Ce qu’on a pu en conclure est que la VR est un bon outil pour de la formation, dans le cas où les exercices sont adaptés pour de la VR (ne pas essayer de reproduire la réalité à tout prix). Cela ouvre une nouvelle possibilités pour une expérience où l’on découvre la programmation, des concepts informatiques, où même découvrir les lieux de l’école, les salles machines, etc… en l’adaptant à la VR (par exemple drag and drop des textes à trou pour de la programmation).

La VR peut être aussi un bon outil pour les étudiants en période de Piscine mais pour les divertir, permettre un lieu d’échange, regrouper les régions et la métropole.
<br>
<br>

---------------------------------------
<br>
Auteur: **owen.duffourg**

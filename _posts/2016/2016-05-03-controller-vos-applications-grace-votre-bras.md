---
title: "Contrôler vos applications grâce à votre bras"
date: "2016-05-03"
categories: 
  - "technical"
tags: 
  - "curl"
  - "myo"
  - "nui"
  - "philips-hue"
  - "uwp"
---

Pour compléter nos tests sur les nouveaux usages autour des NUI (Natural User Interface), nous nous sommes intéressés au [bracelet Myo](https://www.myo.com/). Cet article fait suite au précédent où nous traitions de la [reconnaissance vocale](https://blog.3ie.fr/commander-votre-domotique-a-la-voix-grace-a-cortana/).

# Présentation du bracelet

La technologie du bracelet MYO repose sur l'interception de l'activité électrique de vos muscles. Lorsque vous réalisez un mouvement avec votre bras, votre main, vos muscles réagissent et les impulsions sont captées par le dispositif.

![muscle](/assets/images/muscle.jpg)

La bracelet est compatible avec les différents systèmes d'exploitations bureautiques et mobiles. La communication entre le périphérique et le bracelet s'effectue avec un dongle bluetooth. Celui-ci est fourni dans le kit de base. Vous devez ensuite installer les drivers, à la suite de quoi vous devez suivre un petit didacticiel afin de maîtriser les différents gestes. Le bracelet apprendra ainsi comment vos muscles réagissent en fonction des gestes et améliora sa reconnaissance pour votre prochain utilisation.

Parmi les gestes qui sont reconnus par le bracelet nous trouvons la liste ci dessous

![myo-gestures](/assets/images/myo-gestures.png)

 

Le Myo étant également équipé d'un gyroscope nous sommes également capable de repérer quand le bras se lève, tourne, ...

![myo-gesture-move](/assets/images/myo-gesture-move-2-300x136.png)

Quelques usages de base sont indiqués sur le site du constructeur; nous pouvons citer le contrôle d'une présentation PowerPoint, le pilotage de la souris, le contrôle de lecteur vidéo, ... Bien sûr nous pouvons utiliser le bracelet pour tout autre usage car le constructeur fournit un [SDK](https://developer.thalmic.com/downloads) compatible avec de nombreuses [plateformes](https://developer.thalmic.com/start/) (Windows, Mac, iOs, Android, Unity).

Dans cet article, nous illustrerons l'utilisation du bracelet MYO en venant compléter l'application [UniversalManagerLight](https://github.com/3IE/universal-manager-light/tree/v2.0.0) afin d'allumer et éteindre notre [lampe Philips HUE](https://blog.3ie.fr/un-exemple-de-projet-iot/).

# Utilisation du SDK

Pour Windows nous pouvons récupérer le SDK directement sur [leur site](https://s3.amazonaws.com/thalmicdownloads/windows/SDK/myo-sdk-win-0.9.0.zip). La version du SDK est une DLL implémentée en C++. Dans notre cas nous souhaitons l'utiliser à travers une application UWP développé en C#. Il faut donc référencer un wrapper que vous pouvez trouver sur ce [github MyoSharp](https://github.com/tayfuzun/MyoSharp).

La 1ère étape est de copier la DLL officielle (en fonction de votre plateforme il faudra prendre soit le x86 ou la x64), la copier dans votre projet et ne pas oubliez de la copier à chaque fois dans votre répertoire de build. Pour cela modifier dans les propriétés de la DLL l'option _Copy to output directory_ avec la valeur _Copy always._ 

![myo_dll](/assets/images/myo_dll-1024x681.png)

Il faut ensuite référencer notre wrapper c# dans les références de notre projet.

![myosharp](/assets/images/myosharp-1024x709.png)

 

Maintenant, nous devons nous connecter au Myo pour recevoir les différents événements du bracelet. Dans cet article nous allons ajouter cette fonctionnalité dans le viewModel '[LampViewModel.cs](https://github.com/3IE/universal-manager-light/blob/v2.0.0/UniversalManagerLight/ViewModel/LampViewModel.cs)', mais ce code peut se mettre à n'importe quel endroit où vous souhaitez déclencher la surveillance des événements du Myo. Pour cela il faut ouvrir un canal de communication :

var channel = Channel.Create(
       ChannelDriver.Create(ChannelBridge.Create(),
       MyoErrorHandlerDriver.Create(MyoErrorHandlerBridge.Create()))
);

Puis nous allons gérer les événements de connexion et déconnexion du bracelet grâce à un HUB.

Nous pouvons également interagir avec le Myo depuis le code par exemple en le faisant vibrer avec la méthode _Vibrate._

var hub = MyoSharp.Device.Hub.Create(channel);

// listen for when the Myo connects
hub.MyoConnected += (sender, e1) =>
{
    Debug.WriteLine("Myo {0} has connected!", e1.Myo.Handle);
    e1.Myo.Vibrate(VibrationType.Short);
    e1.Myo.Unlock(UnlockType.Hold);

    e1.Myo.PoseChanged += Myo\_PoseChanged;
};

// listen for when the Myo disconnects
hub.MyoDisconnected += (sender, e1) =>
{
    Debug.WriteLine("It looks like {0} arm Myo has disconnected!", e1.Myo.Arm);
    e1.Myo.PoseChanged -= Myo\_PoseChanged;
};

Le cœur de l'algorithme va maintenant se situer dans la réponse à l’événement _PoseChanged._ C'est dans cet événement que nous allons être notifié de chaque position détectée par le MYO. Dans notre cas nous allons filtrer uniquement l'action _double tap_ pour allumer ou éteindre notre lampe. Lorsque l'événement _DoubleTap_ sera détecté, on utilisera notre dataAccess pour envoyer un ordre à notre serveur d'API.

Dans l'argument de l’événement (_PoseEventArgs_) nous pouvons récupérer d'autres informations grâce à la propriété _Myo._ Vous pouvez filtrer les gestes grâce à l'enum _Pose,_ mais également récupérer la direction du bras, ou encore les données de l'accéléromètre_._

private async void Myo\_PoseChanged(object sender, PoseEventArgs e)
{
    switch (e.Myo.Pose)
    {
        case Pose.DoubleTap:
            if (\_lightState)
            {
                await \_dataAccess.On(new Models.Light() { State = true, LightId = 1, Color = new Models.Color() { R = 1, G = 1, B = 1 } });
            }
            else
            {
                await \_dataAccess.Off(1);
            }
            \_lightState = !\_lightState;
            break;
        case Pose.Unknown:
            break;
        default:
            break;
    }
}

Nous intégrons un simple bouton dans notre vue XAML pour lancer l'action. Cette action est basée sur une commande (_StartListeningMyo_) contenu dans le viewModel :

<Button  RequestedTheme="Dark" Command="{Binding StartListeningMyo}" IsEnabled="{Binding IsEnabledMyo}">Activer</Button>

 

Au final nous obtenons une application qui est capable de réagir au double tap du bracelet MYO pour allumer une lampe à distance et sans aucune autre interaction avec l'ordinateur.

[![UML-screen](/assets/images/UML-screen-1024x603.png)](/assets/images/UML-screen.png)

# Conclusion

Les NUI apportent de nouveaux usages à intégrer dans nos applications. Ici grâce au bracelet MYO, nous pouvons nous éloigner un peu plus des interfaces classiques clavier / souris. Bien sûr l'exemple que nous avons présenté est relativement simple dans sa mise en oeuvre, mais cela nous laisse entrevoir la facilité avec laquelle nous pouvons rajouter ce type d'interaction dans nos applications.

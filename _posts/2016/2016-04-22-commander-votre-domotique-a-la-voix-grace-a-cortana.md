---
title: "Commander votre domotique à la voix grâce à Cortana"
date: "2016-04-22"
categories: 
  - "technical"
tags: 
  - "assistant-vocal"
  - "cortana"
  - "curl"
  - "philips-hue"
  - "uwp"
---

Dans cette partie, nous allons nous intéresser à une NUI (Natural User Interface) : la reconnaissance vocale à travers _Cortana_ qui est l’assistant vocal de Windows10. Les commandes vocales dans les applications windows sont disponibles depuis déjà un moment, l’intérêt de _Cortana_ est de pouvoir établir un semblant d'interactivité. On parle ici d'un semblant car il faut programmer le comportement, et c'est que nous allons voir à travers cet article. Nous allons étudier une interaction directe avec l'application et une autre sous forme de dialogue directement dans _Cortana_ afin de contrôler notre lampe Philips HUE, pour laquelle nous avions développé un serveur lors d'une [partie précédente](https://blog.3ie.fr/un-exemple-de-projet-iot/). Le traitement du langage humain peut mener à certaines ambiguïtés. Cette problématique peut être levée avec [LUIS](https://www.microsoft.com/cognitive-services/en-us/language-understanding-intelligent-service-luis) que nous traiterons dans un prochain article. L'ensemble du code de cet article est disponible sur notre [Github](https://github.com/3IE/universal-manager-light/tree/v1.0.0).

# Objectifs

## Architecture

Pour interagir avec _Cortana_ il faut une application sous Windows10, et si on souhaite la diffuser également sous Windows Phone, une application universelle (UWP) est tout indiquée. Nous suivrons une logique de programmation de type MVVM tout au long de cet article. Pour raison de simplicité nous avons utilisé le framework [MVVM Light](http://www.mvvmlight.net/), mais tout le code peut être compatible avec d'autres frameworks tel que [PRISM](https://github.com/PrismLibrary/Prism). Il existe deux interactions avec _Cortana_ : la première est de lancer des ordres lorsque l'application est en cours d’exécution, et la seconde est d'inscrire des commandes auprès de _Cortana_ (sous forme de composant).

[![Architecture Cortana](/assets/images/Architecture-300x178.png)](/assets/images/Architecture.png) Les commandes vocales sont décrites dans un fichier XML qui suit la norme [VCD](https://msdn.microsoft.com/fr-fr/library/windows/apps/dn722331). Ci dessous un exemple de fichier de définition (celui que nous utiliserons dans cet article est disponible sur notre [github](https://github.com/3IE/universal-manager-light/blob/v1.0.0/UniversalManagerLight/VoiceCommandDefinition.xml)) :

<?xml version="1.0" encoding="utf-8"?>
<VoiceCommands xmlns="http://schemas.microsoft.com/voicecommands/1.2">
  <CommandSet xml:lang="fr-fr" Name="UniversalAppCommandSet\_fr-fr">
    <AppName>génie</AppName>
    <Example> génie </Example>
    <Command Name="offLight">
      <Example>  éteind la lampe  </Example>
      <ListenFor RequireAppName="BeforeOrAfterPhrase"> éteins la lampe </ListenFor>
      <Feedback> La lampe est entrain de s'éteindre </Feedback>
      <Navigate/>
    </Command>   
  </CommandSet>
</VoiceCommands>

Le nœud VoiceCommands peut contenir plusieurs CommandSet, ce qui est utile lorsque vous souhaitez utiliser votre reconnaissance vocale dans plusieurs langues. Vous devez renseigner la langue du CommandSet (dans notre cas 'fr-fr').

Le nœud _CommandSet_ contient le nom de l'application (_AppName_) qui indiquera à _Cortana_ à quelle application fait référence la suite de commandes.

Le nœud _ListenFor_ correspond à la phrase que devra comprendre _Cortana_ et le noeud _FeedBack_ permettra à _Cortana_ de faire un retour utilisateur si la correspondance a été trouvée pour cette commande_._

En prenant la grammaire définie ci dessus, nous pouvons donc éteindre la commande avec la commande suivante : "Génie, éteins la lampe"

## Interface

Pour réaliser cette application, nous n'allons pas avoir besoin d'une interface complexe. Nous aurons une simple zone d'interaction avec un label qui sera bindé sur notre ViewModel.

[![ScreenShot](/assets/images/ScreenShot-1024x600.png)](/assets/images/ScreenShot.png)

Le fichier d'interface sera la vue [lamp.xaml](https://github.com/3IE/universal-manager-light/blob/v1.0.0/UniversalManagerLight/View/Lamp.xaml).

# Ordre direct

Afin de développer cette application nous allons créer un projet BlankApp (Windows Universel).

[![Creations du projet sous visual studio 2015](/assets/images/CreateProject-1024x322.png)](/assets/images/CreateProject.png)

 

Avant toute chose nous allons créer un viewModel ([LampViewModel.cs](https://github.com/3IE/universal-manager-light/blob/v1.0.0/UniversalManagerLight/ViewModel/LampViewModel.cs)) qui permettra d'instancier une dataAccess pour faire les appels aux API.

Ce viewModel contiendra les commandes (OnLight, OffLight, ChangeColor) à exécuter pour piloter notre lampe. Afin de garder notre UI réactive, nous utiliserons ici le couple async / await pour nous assurer de l'asynchronisme de nos commandes. La variable _Message_ est bindée sur notre interface permettant d'avoir un feedback pour notre utilisateur.

OnLight = new RelayCommand(async (param) =>
{
    bool res = await dataAccess.On(new Models.Light() { State = true, LightId = 1, Color = new Models.Color() });
    if (res)
    {
        Message = "Lampe Allumée";
    }
    else
    {
        Message = "Erreur durant l'execution";
    }
});

 

Pour faire la liaison entre notre viewModel et notre serveur d'API pilotant les lampes Philips HUE, nous passerons par notre classe [Light.cs](https://github.com/3IE/universal-manager-light/blob/v1.0.0/DataAccess/Light.cs) située dans la dataAccess. Pour faire transiter notre structure d'objet vers notre serveur, nous sérialisons avec la méthode _JsonConvert.SerializeObject_ 

private const string BASE\_URL = "http://localhost:3000/";
public async Task<bool> On(Models.Light light)
{
    try
    {
        using (HttpClient webClient = new HttpClient())
        {
            string json = JsonConvert.SerializeObject(light);
            HttpContent content = new StringContent(json, Encoding.UTF8, "application/json");
            HttpResponseMessage response = await webClient.PostAsync($"{BASE\_URL}light/on", content);
            return true;
        }
    }
    catch (Exception ex)
    {
        return false;
    }
}

 

Maintenant l'ensemble du travail va s'effectuer dans l'[App.xaml.cs](https://github.com/3IE/universal-manager-light/blob/v1.0.0/UniversalManagerLight/App.xaml.cs)

Il faut tout d'abord charger le fichier de définition des commandes vocales (_[VoiceCommandDefinition.xml](https://github.com/3IE/universal-manager-light/blob/v1.0.0/UniversalManagerLight/VoiceCommandDefinition.xml)_) situé à la racine du projet. Ce chargement doit avoir lieu au moment du lancement de l'application, sur l'event _OnLaunched_.

var storageFile = await Windows.Storage.StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appx:///VoiceCommandDefinition.xml"));
await Windows.ApplicationModel.VoiceCommands.VoiceCommandDefinitionManager.InstallCommandDefinitionsFromStorageFileAsync(storageFile);

 

Puis sur la réponse à l'event _OnActivated_, nous allons filtrer le résultat de la reconnaissance vocale afin d'instancier le ViewModel avec les bons paramètres. Le filtre s'effectue avec l'enum _ActivationKind_ et la valeur _VoiceCommand_. Cette condition nous permet de pouvoir caster en toute sécurité l'argument _commandArgs_ en _VoiceCommandActivatedEventArgs._ Ce cast nous permet de récupérer le nom de la commande ainsi que le texte qui a été prononcé pour activer cette commande. Ensuite, il ne reste qu'à filtrer le nom de la commande pour configurer le contexte d’exécution. Dans un exemple plus complet nous pourrions récupérer par exemple l'id de la lampe sur laquelle on souhaite agir.

 if (args.Kind == ActivationKind.VoiceCommand)
{
    var commandArgs = args as VoiceCommandActivatedEventArgs;

    Windows.Media.SpeechRecognition.SpeechRecognitionResult speechRecognitionResult = commandArgs.Result;

    string voiceCommandName = speechRecognitionResult.RulePath\[0\];
    string textSpoken = speechRecognitionResult.Text;

    // The commandMode is either "voice" or "text", and it indictes how the voice command
    // was entered by the user.
    // Apps should respect "text" mode by providing feedback in silent form.
    string commandMode = this.SemanticInterpretation("commandMode", speechRecognitionResult);

    switch (voiceCommandName)
    {
        case "offLight":
        case "onLight":
           
            // Create a navigation command object to pass to the page. Any object can be passed in,
            // here we're using a simple struct.
            navigationCommand = new ViewModel.LampVoiceCommand() { 
               VoiceCommand = voiceCommandName,
               CommandMode = commandMode,
               TextSpoken = textSpoken
            };
            // Set the page to navigate to for this voice command.
            navigationToPageType = typeof(View.Lamp);
            break;
        default:
            // If we can't determine what page to launch, go to the default entry point.
            navigationToPageType = typeof(View.Lamp);
            break;
    }
}

 

Le cas du changement de couleur pour une lampe va nous permettre d'introduire le concept de désambiguïsation. Typiquement pour le changement de couleur nous avons introduit dans le fichier [VoiceCommandDefinition.xml](https://github.com/3IE/universal-manager-light/blob/v1.0.0/UniversalManagerLight/VoiceCommandDefinition.xml) une possibilité de choix au niveau des couleurs avec la variable _color._ La liste de choix pour cette variable se trouve dans le noeud _PhraseList_ ayant pour label le nom de la variable à savoir dans notre cas _color._

<Command Name="changeColor">
    <Example>  change la couleur en rouge  </Example>
    <ListenFor RequireAppName="BeforeOrAfterPhrase"> change la couleur en {color} </ListenFor>
    <Feedback> La couleur devient {color} </Feedback>
    <Navigate/>
</Command>
<PhraseList Label="color">
    <Item>rouge</Item>
    <Item>bleu</Item>
    <Item>vert</Item>
</PhraseList>

 

Pour obtenir ce choix et pouvoir envoyer les bons paramètres à notre viewModel, nous utiliserons la propriété _SemanticInterpretation_ du résultat de la commande vocale. Pour factoriser le code nous utiliserons la méthode :

private string SemanticInterpretation(string interpretationKey, SpeechRecognitionResult speechRecognitionResult)
{
    return speechRecognitionResult.SemanticInterpretation.Properties\[interpretationKey\].FirstOrDefault();
}

 

Maintenant nous pouvons compléter le switch case pour différencier les différentes commandes :

switch (voiceCommandName)
{
    case "offLight":
    case "onLight":
       
        // Create a navigation command object to pass to the page. Any object can be passed in,
        // here we're using a simple struct.
        navigationCommand = new ViewModel.LampVoiceCommand() { 
           VoiceCommand = voiceCommandName,
           CommandMode = commandMode,
           TextSpoken = textSpoken
        };

        // Set the page to navigate to for this voice command.
        navigationToPageType = typeof(View.Lamp);
        break;
    case "changeColor":
        // Access the value of the {destination} phrase in the voice command
        string color = this.SemanticInterpretation("color", speechRecognitionResult);

        // Create a navigation command object to pass to the page. Any object can be passed in,
        // here we're using a simple struct.
        navigationCommand = new ViewModel.LampVoiceCommand()
        {
            VoiceCommand = voiceCommandName,
            CommandMode = commandMode,
            TextSpoken = textSpoken,
            Color = color
        };

        // Set the page to navigate to for this voice command.
        navigationToPageType = typeof(View.Lamp);

        break;
    default:
        // If we can't determine what page to launch, go to the default entry point.
        navigationToPageType = typeof(View.Lamp);
        break;
}

 

Ensuite nous pouvons lancer la navigation vers la page qui est associée au viewModel avec la méthode _Navigate_ 

rootFrame.Navigate(navigationToPageType, navigationCommand);

 

La récupération des paramètres _navigationCommand_ s'effectuera dans la méthode _OnNavigatedTo_ de la [page](https://github.com/3IE/universal-manager-light/blob/v1.0.0/UniversalManagerLight/View/Lamp.xaml.cs)_._ Dans la logique du MVVM, le _DataContext_ est bindé sur le ViewModel de la page, ce qui nous permet de pouvoir récupérer les commandes du ViewModel et de pouvoir passer les paramètres de la commande vocale.

protected override void OnNavigatedTo(NavigationEventArgs e)
{
    var vm = this.DataContext as LampViewModel;
    if (e.Parameter is LampVoiceCommand)
    {
        var voiceCommand = e.Parameter as LampVoiceCommand;
        switch (voiceCommand.VoiceCommand)
        {
            case "offLight":
                vm.OffLight.Execute(null);
                break;
            case "onLight":
                vm.OnLight.Execute(null);
                break;
            case "changeColor":
                vm.ChangeColor.Execute(voiceCommand.Color);
                break;
            default:
                break;
        }
    }

    base.OnNavigatedTo(e);

}

 

Maintenant nous avons le chemin complet pour lancer une commande à travers Cortana et allumer notre lampe.

[![OrdreDirectCortana](/assets/images/OrdreDirectCortana-1024x596.png)](/assets/images/OrdreDirectCortana.png)

# Intégration plus poussée dans Cortana

Jusqu'à présent nous devions lancer l'application pour interagir avec elle grâce à notre voix. Pour un usage plus efficient il ne faudrait plus lancer l'application. Nous allons donc réaliser toutes les actions à travers _Cortana_.

Pour cela il faut rajouter un composant à notre solution, pour que celui-ci puisse s'inscrire dans la logique de _Cortana_. Quand l'assistant vocal détectera que notre interaction correspond à notre logiciel, il viendra utiliser notre composant pour interagir avec nous et nous guider dans le flux d’exécution.

[![Add Runtime Component VS2015](/assets/images/AddComponent-1024x325.png)](/assets/images/AddComponent.png)

 

Nous devons ensuite créer une [classe CortanaDialogFlow](https://github.com/3IE/universal-manager-light/blob/v1.0.0/RuntimeComponentCortana/CortanaDialogFlow.cs) qui implémente l'interface _IBackgroundTask._ Cette interface définie une méthode _Run_ dans laquelle nous allons définir notre flux_._

La première étape est de récupérer le _AppServiceTriggerDetails_ qui nous permettra de vérifier si le déclenchement correspond bien à notre composant_._

La deuxième étape est de récupérer la commande vocale. Ensuite nous retrouverons la même logique pour dispatcher les ordres vers notre dataAccess grâce à un _switch case_ sur le nom de la commande_._ Si aucune commande ne correspond, nous lancerons l'application.

var triggerDetails = taskInstance.TriggerDetails as AppServiceTriggerDetails;

voiceServiceConnection =
    VoiceCommandServiceConnection.FromAppServiceTriggerDetails(
        triggerDetails);

VoiceCommand voiceCommand = await voiceServiceConnection.GetVoiceCommandAsync();

switch (voiceCommand.CommandName)
{
    case "changeAmbiance":
        await SendCompletionMessageForAmbiance();
        break;
    default:
        LaunchAppInForeground();
        break;
}

 

La méthode _SendCompletionMessageForAmbiance permet_ d'établir un dialogue avec notre utilisateur pour lui indiquer les différents choix s'offrant à lui. Dans le but d'apporter un retour à notre utilisateur nous commençons par lui afficher un message à travers notre méthode _ShowProgressScreen._ Pour afficher une réponse à notre utilisateur il faut utiliser la méthode _CreateResponse_ de la classe statique _VoiceCommandResponse_ qui prend en paramètre un type _VoiceCommandUserMessage._ Ce type est intéressant car il va nous permettre d'indiquer ce que _Cortana_ devra afficher mais également prononcer à travers les propriétés respectives _DisplayMessage_ et _SpokenMessage._

private async Task ShowProgressScreen(string message)
{
    var userProgressMessage = new VoiceCommandUserMessage();
    userProgressMessage.DisplayMessage = userProgressMessage.SpokenMessage = message;

    VoiceCommandResponse response = VoiceCommandResponse.CreateResponse(userProgressMessage);
    await voiceServiceConnection.ReportProgressAsync(response);
}

 

Pour constituer une liste de choix il faut créer une liste de _VoiceCommandContentTile._ Dans notre cas nous allons utiliser uniquement un contenu textuel pour interagir.

var ambianceContentTiles = new List<VoiceCommandContentTile>();

var ambianceWork = new VoiceCommandContentTile();
ambianceWork.ContentTileType = VoiceCommandContentTileType.TitleWithText;
ambianceWork.Title = "Ambiance de travail";
ambianceWork.TextLine1 = "Permet de mettre les lampes en vert";
ambianceWork.AppContext = "work";

ambianceContentTiles.Add(ambianceWork);

 

Lorsque la liste de choix est réalisée nous pouvons l'afficher à l'utilisateur avec la fonction _CreateResponseForPrompt._ Celle-ci, en plus de prendre en paramètre notre liste de choix, doit également prendre les messages à afficher à l'utilisateur. Lorsque l'utilisateur a répondu nous devons analyser sa réponse pour récupérer le nom de l'ambiance.

var response = VoiceCommandResponse.CreateResponseForPrompt(userPrompt, userReprompt, ambianceContentTiles);
var voiceCommandDisambiguationResult = await voiceServiceConnection.RequestDisambiguationAsync(response);
string ambiance = voiceCommandDisambiguationResult.SelectedItem.AppContext as string;

 

Vous pouvez également demander une confirmation à votre utilisateur; la réponse négative ou positive sera alors contenu dans la réponse.

userPrompt.DisplayMessage = userPrompt.SpokenMessage = "Activer l'ambiance  " + ambiance;
userReprompt.DisplayMessage = userReprompt.DisplayMessage = "Voulez vous activer l'ambiance " + ambiance + "?";
response = VoiceCommandResponse.CreateResponseForPrompt(userPrompt, userReprompt);

var voiceCommandConfirmation = await voiceServiceConnection.RequestConfirmationAsync(response);
if (voiceCommandConfirmation.Confirmed)
{

}

 

Selon la réponse de l'utilisateur, nous pouvons lancer notre ordre grâce à la dataAccess.

Une fois l'interaction terminée il faut notifier l'utilisateur que le flux est terminé.

var userMessage = new VoiceCommandUserMessage();

userMessage.DisplayMessage = userMessage.SpokenMessage = "L'ambiance " + ambiance + " a été activée";
response = VoiceCommandResponse.CreateResponse(userMessage);
await voiceServiceConnection.ReportSuccessAsync(response);

 

Ci-dessous vous pouvez voir l’enchaînement des différentes étapes. Je vous invite à regarder le fichier sur notre Github afin d'avoir l'implémentation complète de cet exemple.

Afin que le composant puisse s'enregistrer une première fois au lancement de l'application, il faut le déclarer dans le manifest de l'application. Pour cela on référence le composant dans le projet MVVM, puis on édite le fichier Package.appx.manifest. Dans l'onglet _Declarations_ on ajoute un _App Service._ On renseigne un nom pour notre service permettant de l'identifier ensuite dans notre fichier VoiceCommandDefinition.xml à travers le nœud  _<VoiceCommandService Target="CortanaDialogFlow"/>_ . Dans notre exemple nous utiliserons le nom de la classe _CortanaDialogFlow_, puis on lui donne le point d'accès à cette méthode en renseignant _Entry Point._ Le nom du point d'accès correspond au namespace + le nom de la classe pour nous _RuntimeComponentCortana.CortanaDialogFlow._

 

[![AppManifest](/assets/images/AppManifest_2-300x164.png)](/assets/images/AppManifest_2.png)

 

Pour tester notre code il faut donc démarrer une première fois l'application, afin que le composant puisse s'enregistrer grâce au contrat. Pour le debug, on peut laisser l'application ouverte, ce qui nous permet de mettre des _breakpoints_ dans le composant, mais en toute logique nous pouvons la fermer.

Pour lancer une commande, on peut maintenant s'adresser à _Cortana_ avec la phrase suivante : "Genie change d'ambiance"

[![CortanaFlow](/assets/images/CortanaFlow.png)](/assets/images/CortanaFlow.png)

# Conclusion

Cet article a été écrit un peu avant l'annonce de Microsoft à la [//Build 2016](http://news.microsoft.com/build2016/) où Satya Nadella a annoncé que l'avenir des applications allait passer par l’avènement des bots. Pour accompagner cette annonce, Microsoft a sorti un [framework](https://dev.botframework.com/) permettant de programmer des bots qui seront mis en relation avec des logiciels, sites ou des assistants vocales tel que _Cortana_.

Avec ce type d'application, nous pouvons changer les usages des différents logiciels que nous utilisons. Nous vivons dans un monde de plus en plus multi-taches. Avec les assistants vocaux nous libérons nos mains pour réaliser d'autres tâches en parallèle.

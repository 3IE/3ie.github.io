---
title: "Discussion avec notre lampe"
date: "2016-05-25"
categories: 
  - "technical"
tags: 
  - "bot-framework"
  - "cortana"
  - "curl"
  - "luis"
  - "philips-hue"
---

Un sujet qui est sur toutes les bouches à l'heure actuelle est l'utilisation de Bots. Que ce soit Facebook ou Microsoft, les éditeurs proposent leur solution lors de leur manifestation respective. Dans cet article nous nous intéresserons à [BotFramework](https://dev.botframework.com/) de Microsoft, afin de compléter les interfaces de commandes de notre lampe Philips Hue. Nous couplerons l'utilisation de notre bot avec [LUIS](https://www.luis.ai/).

# Présentation de BotFramework

BotFramework permet de construire son propre robot de discussion à partir de templates C# ou node.JS. La création de robot permet une interaction naturelle sous forme de dialogue. Ces interactions peuvent être réalisées par sms, skype, web chat, Slack, Office 365, ... Lorsque votre bot est créé, vous pouvez le rendre public à travers le répertoire disponible sur le site.

Mais quel peut être les cas d'usage de ces robots ?  Par exemple nous pouvons exposer un robot pour faire une demande de devis auprès d'une entreprise. Le robot se chargerait de demander l'ensemble des informations nécessaires à la réalisation du document ou de prendre les coordonnées de la personne pour qu'elle puisse être rappelée. Un autre exemple peut être de donner des informations sur un contact que nous avons sur une base de données de CRM.

Le BotFramework est constitué de :

- un connecteur qui va permettre d’établir l'interaction entre votre robot et les autres applications
- un SDK disponible pour C# et NodeJS en OpenSource sous github. Dans ce SDK vous aurez également accès à un émulateur.
- un bot directory, pas encore disponible à l'heure où nous écrivons ces lignes.

# Installation de l'environnement

Avant toute chose il faut configurer votre environnement. Commencer par télécharger [l'émulateur Bot Framework](https://aka.ms/bf-bc-emulator). Lorsque celui est installé, il se lance comme une application de bureau.

[![EmulatorBotFramework](/assets/images/EmulatorBotFramework-1024x673.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/EmulatorBotFramework.png)

 

Ensuite vous devez télécharger et installer [les templates pour visual studio 2015](http://docs.botframework.com/connector/vs-template/). Téléchargez le fichier et décompressez le zip dans un dossier nommé "Bot Framework" dans le répertoire template de visual studio. Celui-ci se situe sous **%userprofile%\\documents\\Visual Studio 2015\\Templates\\ProjectTemplates\\Visual C#.**

# Premier bot

Pour la création de notre Bot, nous allons démarrer un nouveau projet, en sélectionnant le template Bot Framework.

[![CreateProject](/assets/images/CreateProject-1-1024x654.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/CreateProject-1.png)

 

De base vous pouvez compiler le projet et effectuer une interaction avec l'émulateur. Le seul et unique point d'entrée pour le dialogue avec votre Bot se situe dans la méthode Post du controller [_MessageController_](https://github.com/3IE/LightManagerBot/blob/v1.0.0/Controllers/MessagesController.cs). C'est ici que vous allez recevoir le message que les utilisateurs vont envoyer.

Afin d'interagir avec eux vous pouvez également leur répondre grâce à la méthode _message.CreateReplyMessage._ Après à vous d'implémenter votre logique d'interaction avec votre interlocuteur.

public async Task<Message> Post(\[FromBody\]Message message)
{
    if (message.Type == "Message")
    {
            return message.CreateReplyMessage("Message recu");
    }
    else
    {
        return HandleSystemMessage(message);
    }
}

 

Comme vous pouvez le voir la variable message comporte un champ _Type._ Nous pouvons filtrer ce champ afin d'avoir accès à des événements d'interaction et donc d'effectuer certaines actions. Parmi les types, nous avons la liste suivante :

- Message
- Ping
- DeleteUserData
- BotAddedToConversation
- BotRemovedFromConversation
- UserAddedToConversation
- UserRemovedFromConversation
- EndOfConversation

Lorsque votre implémentation est terminée vous pouvez déployer votre Bot sur un serveur (par exemple azure) et l'enregistrer sur la plateforme de bot. Cet enregistrement permettra l'interaction avec les autres solutions grâce au bot connector.

[![connectorDiagram](/assets/images/connectorDiagram-1024x576.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/connectorDiagram.png)

# Complément avec LUIS

Réaliser un bot qui répond à des questions prédéfinies reste assez simple comme nous avons pu le voir. Le problème est que pour répondre à ces interactions, il faut faire des égalités strictes entre les phrases saisies par l'utilisateur et les réponses aux actions implémentées de notre coté. Ceci va vite limiter les cas d'usages et nous n'aurons pas une interaction très naturelle. C'est ici qu'intervient _[LUIS](https://www.luis.ai/)_. _LUIS_ signifie _Language Understanding Intelligent Service,_ c'est grâce à ce service que nous allons pouvoir définir un modèle pour reconnaître nos phrases, l’entraîner et exposer cette reconnaissance à travers une API. L’intérêt de cette plateforme est de pouvoir mettre à disposition de tout développeur un système de machine learning sans pour autant avoir à le développer soi même.

Lorsque vous vous êtes connecté, vous pouvez commencer à réaliser votre premier modèle d'apprentissage. Pour cela, créez votre application :

[![CreateLUISApp](/assets/images/CreateLUISApp-1024x434.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/CreateLUISApp.png)

 

Puis vous pouvez configurer votre application (Lui donner un nom, choisir la langue à reconnaître).

[![CreateLUISAppConfig](/assets/images/CreateLUISAppConfig-300x273.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/CreateLUISAppConfig.png)

Attention au choix de la langue, certaines fonctionnalités ne sont disponibles que pour l'anglais.

Pour réaliser votre modèle d'apprentissage, vous devez dans un premier temps définir quelles vont être les entités dont vous allez avoir besoin. Une entité correspond à un morceau de phrase qui représente un concept dans votre phrase. Dans notre exemple nous allons avoir besoin de reconnaître une localisation et une couleur pour savoir comment et où allumer notre lampe. Il existe également des entités pré-entraînées  (localisation géographique, date, nombre, ...)

[![AddEntities](/assets/images/AddEntities-1024x419.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/AddEntities.png) Lorsque nous avons défini toutes nos entités, nous devons créer nos _intents_. Les _intents_ sont les actions qui vont être reconnues suivant le modèle de phrase que nous allons lui fournir. Pour cela il faut passer en mode preview (appuyer sur _Go To Preview_) afin d'avoir un peu plus d'options lors de la création de nos _intents._ Ajouter une nouvelle _intent :_

[![AddIntent](/assets/images/AddIntent-1024x551.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/AddIntent.png)

Le premier paramètre permet de donner un nom à cette action. Ce nom nous sera transmis lorsque le service reconnaîtra cette action. Ensuite l'exemple de la phrase permet de donner un 1er élément pour l'apprentissage. Une action peut avoir besoin, ou non, de paramètres potentiellement obligatoires. C'est grâce à ces paramètres que nous allons indiquer quelle entité reconnaître dans la phrase. Lorsque que l'_intent_ a bien été reconnue, si un paramètre obligatoire est absent de la phrase, alors le service nous demandera l'entité manquante en indiquant la phrase présente dans le paramètre _Prompt._ Lorsque l'intent est créée, nous pouvons passer à la phase d'apprentissage.

Pour cela il faut rentrer de nouvelles variantes de phrases grâces à l'onglet _New Utterance._ Lorsque vous validez cette variante, l'algorithme va tenter de déterminer de quelle action on parle et de trouver les paramètres correspondant à l'action. Si la prédiction est mauvaise, vous pouvez choisir de modifier l'action. Et si les paramètres ne sont pas trouvés, en cliquant sur un mot vous pouvez indiquer à quelle entité il appartient.

[![NewUtterance](/assets/images/NewUtterance-1024x381.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/NewUtterance.png)

 

Réalisez cet apprentissage avec beaucoup de variante afin d'avoir un résultat le plus déterminant possible.

Maintenant que nous avons un modèle entraîné, cliquez sur le bouton _Publish_ pour le rendre accessible grâce au service de publication. En fonction de vos usages, vous pouvez directement le lier à un botFramework ou un autre service. Dans notre cas nous avons juste besoin des _IDs_ qui vont référencer notre modèle.

[![Publish](/assets/images/Publish-1024x514.png)](https://blog.3ie.fr/wp-content/uploads/2016/04/Publish.png)

 

Le cadre rouge correspond à l'Id de votre modèle, et le cadre vert à votre clé d'abonnement. Avec cette paire vous allez pouvoir compléter les informations dans les classes que nous allons maintenant développer.

Pour interagir avec notre modèle LUIS, nous devons rajouter le package _nuget Microsoft.Bot.Builder._ Ensuite, créez une classe '[ManageLightDialog](https://github.com/3IE/LightManagerBot/blob/v1.0.0/ManageLightDialog.cs)' héritant de _LuisDialog_ afin de permettre l'interaction entre notre Bot et le modèle LUIS que nous avons réalisé précédemment. Afin de faire le lien entre notre classe et le modèle de LUIS nous allons utiliser l'attribut de classe _LuisModel_  qui prend en paramètre l'id de notre modèle (cadre rouge) et la clé d'abonnement (cadre vert).

\[Serializable\]
\[LuisModel("eb99623f-9e64-4990-bff3-93d47ee13cd7", "0acaa8df6dc84425a998728c9fa3a9aa")\]
public class ManageLightDialog : LuisDialog<object>
{
  

}

 

Lorsque LUIS reconnait une _intent_, il faut qu'il puisse appeler une méthode pour la traiter. Pour cela nous allons tagger une méthode avec l'attribut _intent_ qui prendra en paramètre  le nom de l'_intent_ défini dans le modèle _Luis._ La première _intent_ à définir est celle qui est vide, et qui sera appelée quand _Luis_ ne reconnaîtra pas d'action.

\[Serializable\]
\[LuisModel("eb99623f-9e64-4990-bff3-93d47ee13cd7", "0acaa8df6dc84425a998728c9fa3a9aa")\]
public class ManageLightDialog : LuisDialog<object>
{
    \[LuisIntent("")\]
    public async Task noIntent(IDialogContext context, LuisResult result)
    {
        string text = "Sorry, didn't understand.  Try again";

        await context.PostAsync(text); // permet d'afficher un texte à l'utilisateur
        context.Wait(MessageReceived); // attente de la prochaine entrée utilisateur
    }
}

 

L'action pour allumer la lumière est un peu plus complexe car nous devons vérifier avant toute chose la présence de deux entités (color & location). Nous allons donc faire une méthode pour extraire ces deux entités à partir de l'analyse de _Luis_ (_LuisResult_):

private bool TryFindType(LuisResult result, out string location, out string color)
{
    location = null;
    color = null;

    EntityRecommendation LocationEntity, ColorEntity;
    if (result.TryFindEntity("Location", out LocationEntity))
        location = LocationEntity.Entity;

    if (result.TryFindEntity("Color", out ColorEntity))
        color = ColorEntity.Entity;

    if (location != null && color != null)
        return true;

    return false;
}

 

Lorsque nous avons nos deux entités, nous pouvons retourner l'information au Bot afin qu'il puisse réaliser une action. Pour passer ces informations nous utiliserons la méthode _context.ConversationData.SetValue_ qui prend en template un objet défini par nous. Dans notre cas l'objet [_QueryItem_](https://github.com/3IE/LightManagerBot/blob/v1.0.0/QueryItem.cs) permettra de remonter les informations _Location_ et _Color_ utiles pour envoyer notre ordre d'allumage.

\[LuisIntent("OnLight")\]
public async Task Onlight(IDialogContext context, LuisResult result)
{
    string location;
    string color;

    if (TryFindType(result, out location, out color))
    {
        string message = $"Turn on the light in {location} with {color} color ....";
        context.ConversationData.SetValue<QueryItem>("userQuery", new QueryItem() { location = location, color = color });
        await context.PostAsync(message);
        context.Wait(MessageReceived);
    }
}

 

Lors de ce dialogue, il faut également prendre en compte que l'utilisateur ne donnera pas toutes les informations. Bien sûr le service de _Luis_ nous l'indiquera mais nous devons faire remonter l'information à l'utilisateur pour récupérer les données manquantes. Dans le cas où il nous manque soit la localisation soit la couleur, nous utiliserons la méthode suivante :

public async Task GetOtherField(IDialogContext context, IAwaitable<Message> argument)
{
    var incomingMessage = await argument;
    string location = null;
    string color = null;
    if (context.ConversationData.TryGetValue("location", out location))
        color = incomingMessage.Text;
    else if (context.ConversationData.TryGetValue("color", out color))
        location = incomingMessage.Text;

    string message = $"Switch on light in the {location} with {color} color ....";

    context.ConversationData.SetValue<QueryItem>("userQuery", new QueryItem() { location = location, color = color });

    await context.PostAsync(message);
    context.Wait(MessageReceived);
}

 

Avec cette méthode nous avons le flux complet ci-dessous prenant en compte les différents cas de figure :

\[LuisIntent("OnLight")\]
public async Task Onlight(IDialogContext context, LuisResult result)
{
    string location;
    string color;

    if (TryFindType(result, out location, out color))
    {
        string message = $"Searching for {location} in {color}....";

        context.ConversationData.SetValue<QueryItem>("userQuery", new QueryItem() { location = location, color = color });

        await context.PostAsync(message);
        context.Wait(MessageReceived);
    }
    else if (location == null && color == null)
    {
        string message = $"I think you want to switch on, please try again.\\n\\n" +
            "Try a phrase like \\"Switch on light in the kitchen with red color\\"";
        await context.PostAsync(message);
        context.Wait(MessageReceived);
    }
    else if (location == null)
    {
        string message = $"Ok, you want to light on with {color} color - But what location?";
        context.ConversationData.SetValue("color", color);
        await context.PostAsync(message);
        context.Wait(GetOtherField);
    }
    else
    {
        string message = $"Ok, you want to light on the light in the {location} - but what color ?";
        context.ConversationData.SetValue("location", location);
        await context.PostAsync(message);
        context.Wait(GetOtherField);
    }
}

 

Pour utiliser l'interaction avec Luis et le code de notre bot nous allons reprendre le code du controller [_MessageController_](https://github.com/3IE/LightManagerBot/blob/v1.0.0/Controllers/MessagesController.cs) et modifier la méthode _Post._ Lorsque la classe représentant notre modèle _Luis_ est instanciée nous pouvons la passer en paramètre à _Conversation.SendAsync._ La récupération des informations pour envoyer un ordre à notre serveur s'effectue avec la méthode _reply.GetBotConversationData_ que nous allons templater avec la classe que nous souhaitons récupérer et que nous avons définie dans la classe utilisant notre modèle _Luis_.

public async Task<Message> Post(\[FromBody\]Message message)
{
    if (message.Type == "Message")
    {
        IDialog<object> form = new ManageLightDialog();
        var reply = await Conversation.SendAsync(message, () => form);

        var userQuery = reply.GetBotConversationData<QueryItem>("userQuery");

        if (userQuery != null)
        {
            // Do Job with userQuery.color & userQuery.location
            return message.CreateReplyMessage("Light on !!");
        }

        return reply;
    }
    else
    {
        return HandleSystemMessage(message);
    }
}

 

# Conclusion

Cet article nous a permis d'aborder un thème qui, il y a encore quelques années, était réservé aux entreprises spécialisées. Avec les efforts des grands acteurs du monde IT (Microsoft, Amazon, ...), nous pouvons maintenant ajouter très facilement de l'intelligence à nos applications. Ces possibilités vont permettre de réaliser des applications toujours plus proches de nos utilisateurs et surtout qui vont pouvoir anticiper leurs besoins. Vous pouvez retrouver les sources sur notre [Gitbub](https://github.com/3IE/LightManagerBot/tree/v1.0.0).

---
title: "Création d'une enquête grâce au chatbot"
date: "2018-12-20"
categories: 
  - "technical"
tags: 
  - "bot-framework"
---

Il y a maintenant quelque temps, nous avions abordé un framework permettant de faire facilement des bots. Depuis ce temps le framework a bien évolué et supporte maintenant de nouvelles fonctionnalités dont une que nous allons explorer à travers cet article.

Nous commençons également à avoir des retours d’expérience  sur l’utilisation des bots notamment sur les premiers contacts avec les consommateurs (effectuer des achats, des réservations, …), mais également sur la réalisation de sondage et c’est ce point que nous allons donc aborder aujourd’hui.

# FormFlow

Cette fonctionnalité a été introduite pour gérer les dialogues dans une conversation guidée. Avant cette évolution il fallait utiliser _Dialogs_ qui est plus complexe à mettre en place avec l’ensemble des branches d’une conversation.

# Cas d’usage

Nous allons développer une application permettant de réaliser un sondage. Le bot doit donc être capable de poser les questions, mais également de restituer les réponses et de permettre la modification des réponses par l’utilisateur.

# Création de l’application

Bien que la toute dernière version de BotFramework (v4) puisse fonctionner sous .NET Core, nous utiliserons dans cet article la version 3 fonctionnant sous .NET.

Il faut partir du template BotApplication.

[![](/assets/images/BF-Template-1024x464.png)](/assets/images/BF-Template.png)

Ps : si vous n’avez pas le template d’installé, assurez-vous d’avoir installé l’extension « Bot Builder Template for Visual Studio ». Pour cela il faut utiliser « Extension et mise à jour » à partir du menu outils de visual studio et rajouter l’extension « Bot Builder Template for Visual Studio ». ([https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3))

[![](/assets/images/BF-Extensions-1024x408.png)](/assets/images/BF-Extensions.png)

Cette extension va vous permettre d’utiliser les templates ainsi que l’émulateur bot Framework.

## **Déroulement du dialogue**

Pour cet exemple nous allons réaliser le dialogue suivant :

[![](/assets/images/BF-DialogFlow.png)](/assets/images/BF-DialogFlow.png)

## Initialisation de la classe

Pour construire notre dialogue nous allons créer une classe permettant la génération de notre _DialogForm_.

Créer une classe qui devra être _Serializable_ et ayant une méthode retournant _IForm<T>_ où T correspond à cette même classe. Cette classe a toute son importance car c’est celle-ci qui nous permettra de créer le dialogue mais également de récupérer les résultats de celui-ci.

Chaque résultat doit être du type d’un _enum_ et exposé sous la forme d’un champ public de la classe.

Une des contraintes concernant l’_Enum_ est de la faire commencer à 1, le 0 étant réservé pour le non choix.

## Poser une question

Le but de _DialogForm_ étant de nous simplifier la vie, il est capable de générer une question cohérente par rapport au nom du champ. Cependant nous pouvons remplacer cette génération en utilisant l’attribut _Prompt_.

```c#
[Prompt("Voulez-vous recevoir notre newsletter ? {||}")]
public bool Newsletter;

```

## Génération des choix pour les réponses

Les réponses aux questions, sont générées de manière automatique également en se basant sur la valeur des _enums_. Ainsi pour une valeur d’_enum_ « _ScienceFiction_ » la génération affichera « Science Fiction » il en aurait été de même pour une valeur d’_enum_ « Science_Fiction » Si nous voulons afficher « Science-fiction » Nous devons utiliser l’attribut _Describe_.

```c#
[Describe("Science-fiction")]
ScienceFiction = 1,
```

L’utilisateur pour répondre, peut également taper des synonymes comme « SF » par exemple. Pour que le programme puisse interpréter convenablement la réponse nous devons lui spécifier les termes qu’il peut accepter en utilisant l’attribut _Term_ :

```c#
[Describe("Science-fiction")]
[Terms("SF", "Science fiction", "Science-fiction")]
ScienceFiction = 1,
```

L’attribut _Describe_ peut également prendre en compte des images :

```c#
[Describe(image: "https://www.staples.fr/content//assets/images/product/76170-00H_1_xnl.jpg")]
Eau = 1,
```

## Validation des réponses

Il existe également d’autres attributs permettant de conditionner la réponse ainsi sur un champ entier nous pouvons lui spécifier un intervalle :

```c#
[Numeric(10, 100)]
public int Age;
```

Si la réponse n’est pas dans cet intervalle, le dialogue demandera une nouvelle réponse.

La validation peut également être plus complexe en utilisant un pattern de validation de type expression régulière.

```c#
[Pattern("(@)(.+)$")]
public string Email;
```

## Configuration de l’enchaînement des questions

Pour instancier notre dialogue, nous allons implémenter une méthode renvoyant un _IForm<NomDeLaClasse>_ . Le corps de base de cette méthode générant le _Dialog_ se trouve ci dessous.

```c#
public static IForm<MovieForm> BuildForm()
{
    return new FormBuilder<MovieForm>()
           .Message("Merci de prendre quelques minutes pour répondre aux questions.")
	   .Confirm("Est-ce votre selection ? {*}")

           .Build();

}
```

La fonction message permet de lancer la série de questions. La méthode _Confirm_ permet une fois l’ensemble du questionnaire réalisé de demander une confirmation à l’utilisateur ainsi si celui-ci souhaite changer une réponse, il peut revenir à la question pour changer sa réponse.

En exécutant tel quel cette méthode, c'est le framework qui choisira l'ordre des questions. Dans notre cas nous souhaitons rajouter un peu de logique, nous allons donc modifier cette méthode afin d’être conforme au schéma présenté en introduction.

### L'ordre des questions

Pour définir l’ordre des questions il faut utiliser l’attribut _Field_, en indiquant le nom du champ

```c#
public static IForm<MovieForm> BuildForm()
{
    return new FormBuilder<MovieForm>()
           .Message("Merci de prendre quelques minutes pour répondre aux questions.")
           .Field(nameof(LocationTheater))
           .Field(nameof(Drink))
           // ....
           .Build();

}
```

Nous pouvons également décider de modifier la séquence des questions, et ainsi avoir une méthode nous indiquant la prochaine étape en fonction d’une valeur. Pour cela nous pouvons écrire une méthode qui retournera la prochaine étape, si nous souhaitons continuer l'ordre défini par les méthodes _Field("..")_, nous devrons juste appeler _NextStep()_ :

```c#
private static NextStep SetNextAfterLocation(object value, MovieForm state)
{

    switch ((LocationTheaterOptions)value)
    {
        case LocationTheaterOptions.Paris13:
            return new NextStep(new[] { nameof(Drink) });
        case LocationTheaterOptions.Paris12:
            return new NextStep();

        case LocationTheaterOptions.Autre:
            return new NextStep(StepDirection.Quit);

        default:
            return new NextStep();

    }
}
```

Dans le cas d’une localisation « Autre » nous avons utilisé une _Enum_ _StepDirection_ qui permet de terminer directement le dialogue.

Pour utiliser cette méthode il faut la déclarer à la suite de _Field_.

```c#
.Field(new FieldReflector<MovieForm>(nameof(LocationTheater))
    .SetNext(SetNextAfterLocation))
```

Nous pouvons également directement désactiver une question en fonction de la réponse d’une autre question :

```c#
.Field(nameof(Drink), state => state.LocationTheater == LocationTheaterOptions.Paris13)
```

Ainsi la question _Drink_ ne sera posée que si _LocationTheater_ est égale à _Paris13_.

Dans le paragraphe précédent nous avons pu voir que nous pouvions valider une réponse en fonction d’attributs (Numeric, Pattern, …). Bien que ces attributs répondent à une grande partie des problématiques, ils ne peuvent gérer des cas complexes. Dans cette situation nous pouvons utiliser une autre surcharge de _Field_ pour spécifier nous même une méthode de validation.

```c#
.Field(nameof(MovieCategory), validate: async (state, value) =>
{
    var category = (MovieCategoryOptions)value;

    var result = new ValidateResult() { IsValid = true, Value = category };

    if (state.Age <= 12 && category == MovieCategoryOptions.Horror)
    {
        result.IsValid = false;
        result.Feedback = "Vous n'êtes pas assez agé pour voir ce type de film";
    }

    return result;
  })
```

La validation d’une question se fait en retournant un objet _ValidateResult_. Si la validation est conforme vous devez setter l’attribut _IsValid_ à _true_ et indiquer la _Value_ pour cette question. Dans le cas où la réponse est non valide, _IsValid_ doit être renseigné à _false_ et vous devez surtout indiquer un feedback pour votre utilisateur.

Maintenant que nous avons vu les différentes étapes pour constituer notre dialogue, il nous faut encore l’intégrer dans notre _controller_.

## Intégration dans le controller

Nous  allons donc modifier la classe servant de point d’entrée du _dialog « MessageController »_ et plus précisément la méthode Post et dans le cas où le message type correspond à une conversation nous allons appeler notre méthode (dans notre cas _MakeRootDialog_) qui instanciera notre dialogue.

La construction du dialogue passe par la méthode suivante :

```swift
Chain.From(() => FormDialog.FromForm(MovieForm.BuildForm))
```

Que nous chaînons avec la méthode _Do_ permettant d’exécuter le dialogue. C’est dans cette action que nous pourrons récupérer les éléments de réponse du dialogue.

```swift
var completed = await survey;
```

La variable _completed_ contient les champs public de notre classe _MovieForm_. Il ne faut pas oublier non plus de gérer les cas d’erreur notamment lorsque nous avions retourné _NextStep(StepDirection.Quit);_ Cette instruction provoque une exception dans l’exécution que nous devons _catcher_. Nous pourrions laisser notre utilisateur sans information, mais il s’agirait ici d’une mauvaise expérience. A travers _e.LastForm_ vous pouvez accéder au résultat du dialogue et ainsi déterminer pourquoi vous avez quitté le dialogue et informer convenablement votre utilisateur de la raison.

# Conclusion

Grâce à _FormFlow_, nous avons pu mettre en place un dialogue qui suit un certains scénario pour guider notre utilisateur à répondre à un certain nombre de questions. Cette fonctionnalité permet également une flexibilité dans le choix des questions qui permet d'adapter son questionnaire à chaque utilisateur. Cette adaptation fait la différence dans ce type d’expérience car cela permet à notre bot de tenir une conversation plus agréable.

L'utilisation de _FormFlow_ pour construire un questionnaire est une bonne alternative à _GoogleForm_ et qui permet de recueillir de l'information à travers les systèmes de messagerie (_Slack, Skype, ..._), dans une page web, et même par sms ...

Vous pouvez retrouver l'ensemble du code source sur notre [Github](https://github.com/3IE/bot-survey).
<br>
<br>

---------------------------------------
<br>
Auteur: **arnaud.lemettre**

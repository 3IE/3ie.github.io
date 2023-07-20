---
title: "Communication entre une application unity et une application native"
date: "2017-05-22"
categories: 
  - "nos-projets"
  - "technical"
---

Dans le cadre d'un projet sous unity, nous sommes amenés à envoyer les téléphones exploitant l'application à divers endroits de la France. Une des informations changeante est donc la localisation de l'événement alors que l'expérience sous Unity reste la même. L'objectif est donc de personnaliser nos statistiques par rapport à chacun des différents sites où s'exécute l'application VR.

Le problème avec une application unity fonctionnant  sous téléphone android est qu'il n'existe pas de solution pour modifier des paramètres de l'application lorsque celle ci a été installée. Le seul moyen est de recompiler l'application en changeant les informations à chaque fois. Ceci occasionne beaucoup de travail et de vérifications pour chaque événement.

L'une des solutions pour contourner ce problème et améliorer notre processus de déploiement a été de créer une application native de configuration et de la faire communiquer avec l'application Unity.

# Mise en place de la solution

D'un point de vue architecture, la solution s'implémente de cette manière :

# [![](/assets/images/ArchiTechnique-1024x380.png)](https://blog.3ie.fr/wp-content/uploads/2017/03/ArchiTechnique.png)

# Création d'un plugin unity

La première étape est de créer un plugin unity permettant d'avoir un Broadcast Receiver.

1. Sous Android Studio, vous pouvez créer un projet avec une activity basic; nous reviendrons sur cette activity dans la prochaine partie. Elle correspondra à notre application de configuration. [![](/assets/images/1_CreateBasicActivity-1024x711.png)](https://blog.3ie.fr/wp-content/uploads/2017/03/1_CreateBasicActivity.png)
2. Ouvre le menu module settings [![](/assets/images/2_OpenModuleSettings.png)](https://blog.3ie.fr/wp-content/uploads/2017/03/2_OpenModuleSettings.png)
3. Rajouter un module (le + en haut à gauche) de type Android Library nommé  "PluginCommunication"
4. Puis créer une classe"ConfigurationReceiver" dérivant de BroadcastReceiver, il faudra implémenter la méthode OnReceive. C'est dans cette méthode que nous allons indiquer "le protocol" de communication, en fait il s'agit juste de dire dans quelle variable de l'intent nous allons rechercher l'information. Dans notre cas nous avons choisi la variable EXTRA\_TEXT de cette manière nous ferons véhiculer l'information dans une chaine Json que nous serons capables de traiter coté Unity.
    
    public class ConfigurationReceiver extends BroadcastReceiver {
       
        public static String text = "";
    
        @Override
        public void onReceive(Context context, Intent intent) {
            String sentIntent = intent.getStringExtra(Intent.EXTRA\_TEXT);
    
            if (sentIntent != null){
                text = sentIntent;
    
            }
        }
    }
    
     
5. Enfin, pour rendre accessible l'information que nous venons de récupérer, nous allons créer un singleton qui sera appelé par notre application Unity.
    
    public class ConfigurationReceiver extends BroadcastReceiver {
        private static ConfigurationReceiver instance;
        public static String text = "";
    
        @Override
        public void onReceive(Context context, Intent intent) {
            String sentIntent = intent.getStringExtra(Intent.EXTRA\_TEXT);
    
            if (sentIntent != null){
                text = sentIntent;
    
            }
        }
    
        public static void createInstance(){
            if (instance == null){
                instance = new ConfigurationReceiver();
            }
        }
    }
    
     
6. Il faut maintenant configurer le manifest Android (manifests/AndroidManifests.xml) pour indiquer que la bibliothèque que nous venons de développer est apte à recevoir des intents.
    
    <receiver android:name="fr.iiie.plugincommunication.ConfigurationReceiver">
    </receiver>
    
7. Un plugin sous unity est disponible sous forme d'une bibliothèque au format JAR. Pour obtenir ce jar nous devons intégrer une task gradle dans le fichier gradle de plugincommunication :
    
    task CreateJar(type: Copy){
        from('build/intermediates/bundles/release/')
        into('libs')
        include('classes.jar')
        rename('classes.jar', 'PluginCommunication.jar')
    }
    
    CreateJar.dependsOn(build);
    
8. Exécuter la task gradle et vous allez pouvoir récupérer votre plugin, dans le répertoire PluginCommunication/libs/[![](/assets/images/4_TaskGradle-1024x341.png)](https://blog.3ie.fr/wp-content/uploads/2017/03/4_TaskGradle.png)

# Installation du Plugin sous unity

Le plugin (PluginCommunication.jar) se copie dans le répertoire Asset/Plugins/Android/ de votre projet Unity. Afin que l'écoute de l'intent soit également pris en compte au niveau de l'application unity, vous devez rajouter des informations dans le ManifestAndroid de l'application Android. S'il n'est pas déjà présent vous pouvez générer une première fois l'application unity en targettant la plateforme android, et vous allez pouvoir récupérer le manifest dans le dossier temp/StagingArea/AndroidManifest.xml de votre projet unity. Copier ce manifest sous Asset/Plugins/Android/ en ajoutant les informations suivantes sur la réception de l'intent :

<receiver android:name="fr.iiie.plugincommunication.ConfigurationReceiver">
	<intent-filter>
		<action android:name="fr.iiie.unityConfiguration"></action>
	</intent-filter>
</receiver>

 

Maintenant que notre plugin unity est en place nous allons pouvoir l'utiliser. Pour cela positionnez vous dans une scène où vous allez avoir besoin de cette information et rajouter le code suivant :

AndroidJavaClass jc = new AndroidJavaClass("fr.iiie.plugincommunication.ConfigurationReceiver");

jc.CallStatic("createInstance");
string externalConfigurationRaw = jc.GetStatic<string>("text");

la 1ere ligne permet d'instancier la classe contenue dans notre plugin, il faut donc être rigoureux sur le nom du package et le nom de la classe.

La deuxième ligne permet d'appeler la méthode "createInstance" qui nous retourne le singleton de notre plugin, ensuite nous pouvons appeler notre propriété "text" qui contiendra la configuration.

Si vous souhaitez mettre en cache ces informations pour éviter de toujours lancer l'application de configuration, vous pouvez implémenter une base de données SQLite ou stocker ces informations dans un fichier dans le cache local de l'application unity.

# Création de l'application de configuration

Nous allons maintenant nous servir de notre projet "app" que nous avions créer dans la première partie. Sur la vue de l'activity (app/res/layout/content\_main.xml) , vous pouvez ajouter un textbox et un bouton.

[![](/assets/images/5_InterfaceContentMain-1024x516.png)](https://blog.3ie.fr/wp-content/uploads/2017/03/5_InterfaceContentMain.png)

L'essentiel du code se fera dans l'appel du bouton, où nous allons déclarer un intent et l'envoyer. Il faut faire attention dans la déclaration de l'intent car c'est ici que nous allons lui spécifier son type. Ce type devra être le même que celui auquel s'attend le broadcast receiver. Pour envoyer notre intent nous pouvons soit envoyer un sendBroadcast ou un sendOrderedBroadcast. la première méthode sera plus à considérer comme un fire and forget alors que pour la deuxième nous pourrons avoir un callback pour savoir si l'intent à bien été reçu par quelqu'un.

Button myButton = (Button)  findViewById(R.id.button);
final EditText editTextLocation = (EditText) findViewById(R.id.Location);
myButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        JSONObject applicationInformation = new JSONObject();
        try {
            applicationInformation.put("Location", editTextLocation.getText());
        } catch (JSONException e) {
            e.printStackTrace();
        }
        Intent myIntent = new Intent();
        myIntent.setAction("fr.iiie.unityConfiguration");
        myIntent.putExtra(Intent.EXTRA\_TEXT, applicationInformation.toString() );
        sendOrderedBroadcast(myIntent, null, new BroadcastReceiver() {
            @Override
            public void onReceive(Context context, Intent intent) {
                Bundle results = getResultExtras(true);
                Log.i("SendOrderedBroadc", "Final Result Receiver = " + results.getString("Breadcrumb", "nil"));
            }
        }, null, Activity.RESULT\_OK, null, null);
    }
});

 

# Conclusion

Une fois l'application Unity installée, ainsi que l'application de configuration nous pouvons renseigner les informations dans l'application native et s'assurer que ces informations sont bien récupérées sur l'application unity.

Vous pouvez retrouver les sources de l'application et du plugin sur notre [github](https://github.com/3IE/UnityConfigurator).

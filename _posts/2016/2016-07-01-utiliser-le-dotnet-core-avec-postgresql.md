---
title: "Faire une Web API avec .Net Core et PostgreSQL"
date: "2016-07-01"
categories: 
  - "technical"
  - "tendances-web"
tags: 
  - "net"
  - "net-core"
  - "cross-platform"
  - "postgresql"
---

Maintenant que Microsoft a officialisé la RC2 du [.Net Core](https://dotnet.github.io/), nous disposons d'une vraie " release candidate"  qui va permettre de se lancer dans le .Net multi-plateforme (après une RC1 qui se retrouve finalement non compatible avec la version finale).

Il est dès maintenant possible d'utiliser PostgreSQL avec Entity Framework car les [drivers font partie de EF Core 1.0](https://docs.efproject.net/en/latest/providers/index.html) via [npgsql](http://www.npgsql.org/). Nous allons voir dans cet article comment rapidement mettre cela en place.

## Installation et configuration

Pour le .Net Core, vous pouvez télécharger la dernière release sur le [site officiel](https://www.microsoft.com/net/core).

Si vous utilisez Visual Studio 2015, installez l'update 3 et mettez bien à jour vos extensions pour être compatible avec le .Net Core RC2 et profiter des templates projets.

Pour PostgreSQL, vous pouvez l'installer en local ou bien depuis l'image [Docker Postgres](https://hub.docker.com/_/postgres/)

Pour faire notre test, initialisez votre base avec les données suivantes :

CREATE TABLE public.persons
(
    id SERIAL PRIMARY KEY NOT NULL,
    name VARCHAR(100) NOT NULL
);
INSERT INTO persons(name) VALUES ('john'),('frank'),('fred');

## Création du projet

Si vous êtes sous Visual Studio, créez un nouveau projet de type "ASP.NET Core Web Application (.Net Core) -> Web API".

Si vous êtes sous VS Code, vous pouvez télécharger le template depuis le [github aspnet](https://github.com/aspnet/Templates/tree/release/src/BaseTemplates/WebAPI) mais il faudra le modifier un peu car il y a des directives qui sont normalement interprétées par Visual Studio.

Pour tester que tout fonctionne bien,  nous allons lancer notre projet. Le template contient une solution complète ainsi qu'un projet. Depuis un terminal, déplacez vous dans le dossier de ce dernier. Restaurez les packages avec $ dotnet restore et ensuite exécutez le projet avec $ dotnet run. La compilation est automatiquement lancée lors du 'run' si 'dotnet' détecte que des fichiers ont été modifiés depuis la dernière compilation.

Vous devriez obtenir le résultat suivant :

$ dotnet run
Project WebApplication1 (.NETCoreApp,Version=v1.0) was previously compiled. Skipping compilation.
Hosting environment: Production
Content root path: /Users/verdie\_b/Documents/versioning/DotnetCoreAndPostgres/src/WebApplication1
Now listening on: http://\*:5000
Application started. Press Ctrl+C to shut down.

Enfin, vérifiez que l'url suivante [http://localhost:5000/api/values](http://localhost:5000/api/values) s'affiche bien.

## Modifications du projet

#### Ajout du provider Postgres

Tout d'abord, pour pouvoir se connecter à une base Postgres, il faut que nous ajoutions une nouvelle dépendance à notre projet. Pour cela, ouvrez le fichier 'project.json'. Le fichier contient une zone 'dependencies' à laquelle nous allons ajouter notre _provider_ Entity Framework pour Postgres.

Ajoutez "Npgsql.EntityFrameworkCore.PostgreSQL" à vos dépendances :

"dependencies": {
    "Microsoft.NETCore.App": {
      "version": "1.0.0",
      "type": "platform"
    },
    "Microsoft.AspNetCore.Mvc": "1.0.0",
    "Microsoft.AspNetCore.Server.IISIntegration": "1.0.0",
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.0",
    "Microsoft.Extensions.Configuration.EnvironmentVariables": "1.0.0",
    "Microsoft.Extensions.Configuration.FileExtensions": "1.0.0",
    "Microsoft.Extensions.Configuration.Json": "1.0.0",
    "Microsoft.Extensions.Logging": "1.0.0",
    "Microsoft.Extensions.Logging.Console": "1.0.0",
    "Microsoft.Extensions.Logging.Debug": "1.0.0",
    "Microsoft.Extensions.Options.ConfigurationExtensions": "1.0.0",
    "Npgsql.EntityFrameworkCore.PostgreSQL": "1.0.0"
  }

Ensuite faites un $ dotnet restore pour vérifier que la config est valide.

#### Création du modèle

Avec EF Core, comme nous avons déjà notre base, il faut décrire dans le code les tables. Comme la structure est très simple, cela va être rapide.

Créez un dossier 'Models' dans votre dossier 'WebApplication1' et ajoutez un fichier 'PersonsContext.cs' :

using System.ComponentModel.DataAnnotations.Schema;
using Microsoft.EntityFrameworkCore;

namespace WebApplication1.Models
{
    public class PersonsContext : DbContext
	
    {
        public DbSet<Person> persons { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
			optionsBuilder.UseNpgsql(@"Server=localhost;Port=5432;Database=postgres;User Id=postgres;Password=toto;");
        }
    }

    \[Table("persons")\]
    public class Person
    {
        \[Column("id")\]
        public int Id { get; set; }

        \[Column("name")\]
        public string Name { get; set; }
    }

}

Notre base possède des identifiants en minuscule mais la _naming conventions_ en C# dicte que l'on utilise du Upper CamelCase. Ce problème se résout grace aux annotations 'Table' et 'Column' qui permettent de spécifier le nom des colonnes et des tables indépendamment des noms utilisés dans le code.

#### Création du controller

Pour l'exemple, nous allons  ajouter un web service très simple qui retournera l'intégralité des personnes.

Dans le dossier 'WebApplication1/Controllers', ajoutez un fichier 'DBController' :

using System.Collections.Generic;
using System.Linq;
using Microsoft.AspNetCore.Mvc;
using WebApplication1.Models;

namespace WebApplication1.Controllers
{
    \[Route("api/\[controller\]")\]
    public class DBController : Controller
    {
        // GET api/db
        \[HttpGet\]
        public IEnumerable<Person> Get()
        {
            using (var db = new PersonsContext())
            {
                var persons = db.persons.OrderBy(p => p.Id);
                return persons.ToList();
            }
        }
    }
}

En .Net Core, on n'hérite plus de la classe APIController mais de Controller. Pour déterminer la route qui sera utilisée, il faut regarder l'annotation. Comme nous référençons "api/\[controller\]", le .Net va prendre le nom de la classe et retirer le postfix "Controller", ce qui nous donne la route "api/db"

Vous pouvez maintenant consulter votre web service à l'url suivante : [http://localhost:5000/api/db](http://localhost:5000/api/db)

## Conclusion

La sortie du .Net Core est une très bonne nouvelle car cela va permettre de mixer beaucoup plus facilement les technos sur un petit serveur Linux. Plus de raison d'hésiter, le développement est en plus facilité avec la mise à dispo de VSCode sur les principaux OS.

Vous pouvez retrouver le code de projet sur le [github 3IE](https://github.com/3IE/DotnetCoreAndPostgres/)

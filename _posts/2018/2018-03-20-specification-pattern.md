---
title: "Specification Pattern"
date: "2018-03-20"
categories: 
  - "technical"
tags: 
  - "pattern"
---

Le design pattern specification fait partie du DDD (Domain Driven Design). Son nom complet est Query Specification. Ce pattern permet de décrire une requête sous la forme d’un objet. Bien qu’il fasse partie du DDD, ce pattern peut être appliqué tout à fait dans un autre design (N-Tier, …).

# Pour faire quoi ?

Par exemple pour encapsuler une requête qui recherche certains produits, vous pouvez créer une Specification ProductSpec qui prend les paramètres d’entrée nécessaires (conditions de la clause Where par exemple). Ensuite dans la dataAccess vous aurez une méthode (List, find, …) qui acceptera une ISpecification en paramètre et exécutera la requête attendue basée sur cette spécification.

De cette manière plutôt que d’écrire :

public IReadOnlyList<Order> GetOrderById(int id) { ... }
public IReadOnlyList<Order> GetOrderByDate(DateTime date) { ... }
public IReadOnlyList<Order> GetOrderBetweenDate(DateTime startDate, DateTime endDate) { ... }

Votre code devient :

public IReadOnlyList<Order> GetOrder(Specification<Order> specification) { ... }

Ce qui permet d’avoir des couches d’accès aux données beaucoup plus simples et surtout moins de code à écrire. L’autre avantage permet d’offrir plus de souplesse sur la récupération des données en permettant de mixer les Specifications, tout en gardant une lisibilité sur la condition. Si nous prenons, par exemple, le corps d’une méthode permettant de remonter une collection de données filtrées :

var tv = products.Where(x => x.Category == 1 && x.Brand == "samsung");

Dans une architecture en couche, on peut créer une fonction qui prendrait en paramètre Category et Brand afin de mutualiser le code, mais que se passe-t’il quand on souhaite rajouter une condition ou que l’on souhaite retirer une clause sur notre filtre ? Nous sommes obligés de réécrire la méthode ou de faire une surcharge.

Pour pallier à ce problème, nous allons utiliser le pattern Specification.

# Un peu de pratique

Ce pattern est assez simple à comprendre puisqu’il se compose d’une seule méthode IsSatisfiedBy qui prend en paramètre l’objet à vérifier. Afin d’éviter la duplication du code, nous allons créer une classe générique.

public abstract class Specification<T>
{
    public abstract Expression<Func<T, bool>> ToExpression();
 
    public bool IsSatisfiedBy(T entity)
    {
        Func<T, bool> predicate = ToExpression().Compile();
        return predicate(entity);
    }
}

Le type de retour de ToExpression() est Expression<Func<T, bool>>, celui-ci  pourrait être simplement Func<T, bool> mais comme nous souhaitons appliquer l’expression sur une collection IQueryable l’utilisation d’Expression<T> est _donc_ obligatoire. Avec seulement Func<,> nous pourrions l’appliquer que sur une liste de type IEnumerable, nous devrions donc récupérer l’ensemble des données en mémoire plutôt que de pouvoir filtrer directement sur la base de données.

Pour utiliser cette classe, nous devons implémenter une spécification typée :

public class CitySpecification : Specification<Order>
{
    private string \_city;
    public CitySpecification(string city)
    {
        \_city = city;
    }
    public override Expression<Func<Order, bool>> ToExpression()
    {
        return order => order.Address.City == \_city;
    }
}

Lors du développement, un besoin peut apparaître lorsque nous allons vouloir combiner plusieurs spécifications. Bien entendu, on pourrait recréer une nouvelle Specification combinant les deux conditions, mais nous ne respecterions pas le principe DRY.

Afin de respecter ce dernier, nous allons modifier légèrement notre classe de base en ajoutant deux méthodes And() et Or()

public abstract class Specification<T>
{

    public abstract Expression<Func<T, bool>> ToExpression();
    public bool IsSatisfiedBy(T entity)
    {
        Func<T, bool> predicate = ToExpression().Compile();
        return predicate(entity);
    }
    public Specification<T> And(Specification<T> specification)
    {
        return new AndSpecification<T>(this, specification);
    }
    public Specification<T> Or(Specification<T> specification)
    {
        return new OrSpecification<T>(this, specification);
    }
}

Pour l’implémentation d’un AndSpecification :

public class AndSpecification<T> : Specification<T>
   {
       private readonly Specification<T> \_left;
       private readonly Specification<T> \_right;
 
       public AndSpecification(Specification<T> left, Specification<T> right)
       {
           \_right = right;
           \_left = left;
       }
 
 
       public override Expression<Func<T, bool>> ToExpression()
       {
           Expression<Func<T, bool>> leftExpression = \_left.ToExpression();
           Expression<Func<T, bool>> rightExpression = \_right.ToExpression();
           
           var paramExpr = Expression.Parameter(typeof(T));
           var exprBody = Expression.AndAlso(leftExpression.Body, rightExpression.Body);
           exprBody = (BinaryExpression)new ParameterReplacer(paramExpr).Visit(exprBody);
           var finalExpr = Expression.Lambda<Func<T, bool>>(exprBody, paramExpr);
 
           return finalExpr;
       }
   }

Pour voir l’implémentation de la partie Or() je vous invite à voir les sources sur [Github](https://github.com/3IE/SpecificationPattern/blob/master/Domain/Specifications/OrSpecification.cs).

Avec les dernières modifications, nous pouvons utiliser les classes de cette manière :

var limitDateSpec = new LimitDateSpecification(DateTime.Now);
var citySpec = new CitySpecification(city);
var res = await \_query.FindOrderAsync(citySpec.And(limitDateSpec));

// ou encore
bool isInCity = citySpec.IsSatisfiedBy(myOrder);

# Conclusion

Cette approche nous permet de réutiliser facilement les conditions, facilite les tests, certes nous avons plus de classes à tester mais les méthodes sont plus simples. Nous gagnons également en temps de développement sur nos accès aux données en fournissant une seule méthode prenant en paramètre nos Specifications.

Les sources de cet article sont disponibles sur notre [github](https://github.com/3IE/SpecificationPattern).

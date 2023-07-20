---
title: "Responsive Web Design"
date: "2014-08-21"
categories: 
  - "tendances-web"
tags: 
  - "m-commerce"
  - "media-queries-css"
  - "solar-decathlon"
  - "web-responsive-design"
---

_Avec la multiplication des écrans les internautes ont eu besoin d'accéder aux contenus depuis n'importe quel terminal et n'importe où. La première solution a été de créer un site dédié pour chaque appareil. Cependant les écrans se fond de plus en plus nombreux avec l’arrivée des tablettes, des smartphones nouvelle génération et qui sait encore…_

Résultat : plusieurs adresses web pour un même contenu ce qui engendre un mauvais référencement sans parler du fait d'avoir plusieurs sites à gérer. La solution qui répond à ces nouveaux besoin est une approche introduite par Ethan Marcotte dans un article de [A List Appart](http://alistapart.com/article/responsive-web-design) publié en 2010 : le Responsive Web Design. Cette conception web vise à créer un site qui offre une navigation optimale pour l'utilisateur. Concrètement le site s’adapte automatiquement à l’espace disponible sur l’écran.  Ainsi le menu, la slidebar, les images et même sa structure sont visible convenablement et sans zoom sur mobile. Les colonnes, par exemple peuvent s’ajuster, se déplacer, voire disparaître. Les images se redimensionnent, se replacent et il en va de même pour de nombreuses choses. Voici quelques exemples, n’hésitez pas à réduire la largeur de la fenêtre afin de voir les éléments se réorganiser tous seuls.

[![Responsive web design 4](/assets/images/Responsive-web-design-4.png)](http://nanoc.ws/ "nanoc.ws") [![Responsive web design 5](/assets/images/Responsive-web-design-5.png)](http://hirondelleusa.org/ "hirondelleusa") Le principe est de concevoir une infinité de design en imaginant les éléments de la page s’organiser selon toutes les résolutions. Une fois que nous savons comment notre design va s’adapter en fonction de la largeur l'intégrateur web prend le relais. Celui-ci va rédiger des règles via Média Queries CSS qui permettent d'adapter le design en fonction de certains critères comme la largeur de l'écran.

La tendance actuelle consiste à penser la navigation, l’ergonomie et le contenu du site pour les mobiles, pour ensuite les adapter aux navigateurs web sur PC : c’est le mobile first. En commençant par la version mobile on évite toute les règles CSS destinés à la navigation desktop et on obtient un site plus rapide à télécharger et afficher sur mobile.

[![responsive vs mobile first](/assets/images/responsive-vs-mobile-first.png)](https://blog.3ie.fr/wp-content/uploads/2014/08/responsive-vs-mobile-first.png) Cependant le RWD n’est pas une solution miracle et ne convient pas à tous les projets. Selon vos besoins et ceux de vos internautes, le responsive web design ne sera pas toujours la solution à privilégier. Si vos mobinautes recherchent un accès rapide et simple à l’information et que le contenu de votre site web est dense et indispensable, le responsive design n’est pas la meilleure solution. En revanche si votre contenu est facilement réorganisable cette technique à de fortes chance de séduire votre public. Il faut donc clairement définir ses cibles et ses objectifs afin de voir si le RWD peut être la meilleure solution à mettre en œuvre.

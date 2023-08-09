---
title: "WebAssembly et Angular le combo parfait ?"
date: "2018-11-16"
categories: 
  - "technical"
---

Le web gagne de plus en plus en complexité, en effet aujourd’hui la plupart des sites web tendent à utiliser des technologies coûteuses en ressources : réalité virtuelle, intégration 3D, algorithmes de recherche, etc.

## Genèse

Pour répondre à cette demande, en 2015, Google et asm.js annoncèrent l’arrivée prochaine d’une technologie innovante capable de s’allier à _Javascript_ pour optimiser la vitesse d’exécution de certains processus coûteux, je parle bien entendu de WebAssembly. WebAssembly est un nouveau langage de bas niveau de type assembleur, formaté en bytecode (instructions proches du langage machine), exécutable dans les navigateurs modernes le supportant (Chrome, Edge, Mozilla Firefox and Safari) et se voulant aussi performants que des langages de bas niveaux tels que C, C++ ou Rust. 

## En quoi ça consiste ?

Le développeur écrit son sous-programme en C au lieu de l'écrire directement en Javascript, puis le compilateur va se charger de transformer directement son code C en bytecode WebAssembly et de générer le code Javascript permettant d'appeler l'API WebAssembly / JS.

[![](/assets/images/emscripten-diagram.png)](/assets/images/emscripten-diagram.png)

L'API WebAssembly de Javascript va permettre à Javascript de charger les modules WebAssembly et inversement. La machine virtuelle de votre application web qui exécutait auparavant seulement du code Javascript va aujourd'hui également pouvoir charger et exécuter du code WebAssembly.

## Pourquoi s’en soucier ?

Il est vrai que _Javascript_ est facile d’utilisation et très “malléable” car c’est un langage de très haut niveau, de plus il est déjà le socle principal de la quasi totalité du web d’aujourd’hui… sauf qu’il est lent. Mais très lent. Le niveau d’abstraction que nous offre _Javascript_, son garbage collector, ainsi que bon nombre de fonctionnalités, le ralentissent et c’est pour cela que parfois, sur certains programmes, on aurait envie de se passer de _Javascript_ et d’avoir sous la main un autre langage, plus bas niveau, sans toute cette couche abstraite qui nous est cachée et qui nous freine tellement.

C'est justement ce que nous offre WebAssembly : garder la puissance d’abstraction de _Javascript_ et l’incontournable lien qu’il permet d’établir entre une page HTML / CSS et du code source, et déléguer des tâches coûteuses en ressources à des sous programmes écrits dans un langage de plus bas niveau, comme le _C_, le _C++_ ou le _Rust_ (seuls ces trois là sont proposés pour l’instant) afin d’optimiser les performances de calcul à des endroits ciblés.

## Intégrer WebAssembly dans Angular

Passons aux choses sérieuses, WebAssembly n'en est encore qu'à ses balbutiements et l'intégrer à _Angular_ n'est pas une mince affaire.

Ici nous créons un nouveau projet Angular grâce à CLI

```ps
ng new angular-wasm
```

Nous avons envie que TypeScript puisse reconnaître des modules provenant de WebAssembly

```swift
npm install @types/webassembly-js-api --dev --save
```

Vous devez maintenant installer **emsdk**, **emcc**, et **emscripten**, voici quelques liens qui devraient vous être utiles :

- [Guide d'installation officiel](https://webassembly.org/getting-started/developers-guide/)
- [emsdk](https://github.com/juj/emsdk)

L'installation terminée, nous allons pouvoir créer nos programmes en _C_ qui seront ensuite compilés en WebAssembly par _**emcc**_, ce dernier va également générer le fichier Javascript associé au fichier WASM qui servira de lien entre le code source WASM et le TypeScript final afin de pouvoir être importé dans notre projet Angular.

Créez un dossier wasm dans app/

```ps
mkdir src/app/wasm
```

```c
#include <emscripten.h>
#include <string.h>
#include <stdlib.h>

// Implémentation itérative de fibonacci
int EMSCRIPTEN_KEEPALIVE fibo(int n)
{
  int first = 0, second = 1;

  int tmp;
  while (n--) {
    tmp = first+second;
    first = second;
    second = tmp;
  }

  return first;
}

// Ici on alloue petit à petit un tableau
// dans lequel on insert des caractères puis on le retourne
// Taille du tableau : 10 000 000 d'éléments
char* EMSCRIPTEN_KEEPALIVE play_with_memory()
{
	char* str = malloc(2);
	str[0] = 'a';
	str[1] = '\\0';
	for (size_t i = 0; i < 10000000; i++)
	{
		str = realloc(str, strlen(str) + 1);
		str[strlen(str) - 1] = (i % 128) + '0';
		str[strlen(str)] = '\\0';
	}
	return str;
}
```

Dans ce fichier, nous avons deux fonctions qui sont là pour tester les performances de WebAssembly. Ces fonctions seront réécrites en TypeScript par la suite afin d'évaluer les différences de performances entre les deux technologies.

Nous allons à présent devoir compiler séparément _evaluator.c,_ en effet je n'ai pas trouvé le moyen d'automatiser cette étape et elle est nécessaire pour obtenir notre .wasm et notre .js

```ps
cd app/src/wasm
emcc ./evaluator.c -Os -s WASM=1 -s MODULARIZE=1 -o ./evaluator.js
```

Nous obtenons ainsi notre fichier binaire _evaluator.wasm_ et notre fichier Javascript _evaluator.js._

_On remarquera l'utilisation de l'option MODULARIZE qui permet de rendre nos fonctions modulaires, ce qui facilite leur intégration dans Angular._

Maintenant que tout est fin prêt, créons un dossier service dans lequel nous allons demander à CLI de nous générer un nouveau service qui nous permettra de nous servir de nos toutes nouvelles fonctions.

```swift
mkdir src/app/services
cd src/app/services
ng generate service wasm
```

```js
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs/Observable';
import { BehaviorSubject } from 'rxjs/BehaviorSubject';
import { fromPromise } from 'rxjs/observable/fromPromise';
import { Subject } from 'rxjs/Subject';
import { filter, take, mergeMap } from 'rxjs/operators';

import * as Module from './../wasm/evaluator.js';
import '!!file-loader?name=wasm/evaluator.wasm!./../wasm/evaluator.wasm';
import { resolve } from 'url';

declare var WebAssembly;

@Injectable()
export class WasmService {
  module: any;

  wasmReady = new BehaviorSubject<boolean>(false);

  constructor() {
    this.instantiateWasm('wasm/evaluator.wasm');
  }

  private async instantiateWasm(path: string) {
    // charge le fichier .wasm
    const wasmFile = await fetch(path);

    // le convertit en un buffer binaire
    const buffer = await wasmFile.arrayBuffer();
    const binary = new Uint8Array(buffer);

    const moduleArgs = {
      wasmBinary: binary,
      onRuntimeInitialized: () => {
        this.wasmReady.next(true);
      }
    };

    // instantie le module
    this.module = Module(moduleArgs);
  }

  public fibonacci(input: number): Observable<number> {
    return this.wasmReady.pipe(filter(value => value === true)).pipe(
      mergeMap(() => {
        return fromPromise(
          new Promise<number>((resolve, reject) => {
            setTimeout(() => {
              const result = this.module._fibo(input);
              resolve(result);
            });
          })
        );
      }),
      take(1)
    );
  }

  public playWithMemory(): Observable<number> {
    return this.wasmReady.pipe(filter(value => value === true)).pipe(
      mergeMap(() => {
        return fromPromise(
          new Promise<number>((resolve, reject) => {
            setTimeout(() => {
              const result = this.module._play_with_memory();
              resolve(result);
            });
          })
        );
      }),
      take(1)
    );
  }
}
```

Il faut également l'ajouter aux providers de l'app

```js
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { NgModule } from '@angular/core';

import { AppComponent } from './app.component';
import { WasmService } from './services/wasm.service';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule
  ],
  providers: [WasmService],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

Il ne vous reste qu'une chose à faire, appeler ce service qui appellera pour vous les fonctions Web Assembly. Pour ce faire vous pouvez, dans n'importe quel component de votre app, instancier une instance WasmService et appeler l'une de ses méthodes.

```js
import { Component, OnInit } from '@angular/core';
import { WasmService } from './services/wasm.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})

export class AppComponent implements OnInit {
  private wasmService: WasmService;

  public constructor() {}

  public ngOnInit() {
    this.wasmService = new WasmService();
    this.wasmService.fibonacci(42).subscribe(res => {
      console.log(res); // On a bien 165580141 qui s'affiche dans la console
    });
  }
}
```

_Architecture finale :_

[![](/assets/images/architecture-article-wasm.png)](/assets/images/architecture-article-wasm.png)

## Et les performances dans tout ça ?

On aurait tendance à penser que toute cette couche ajoutée qui nous permet d'intégrer correctement notre code WebAssembly en Angular ralentirait considérablement l'exécution de nos fonctions, et... c'est en partie vrai.

 

[![](/assets/images/fibonacci-table-2.png)](/assets/images/fibonacci-table-2.png)  
**Résultats pour fibonacci**

 

En effet, pour appeler nos fonctions, on est obligé d'utiliser des requêtes asynchrones, ce qui implique une gestion d’événements, ce que la bibliothèque _**rxjs**_ fait très bien à notre place, mais cela ralentit le code en appel et en sortie de programme.

Ce qui fait que sur de petits programmes, comme c'est le cas de Fibonacci, l'exécution du programme est trop courte pour compenser la perte de performances due à l'appel et à la sortie d'exécution. C'est pourquoi, il n'est pas judicieux d'intégrer WebAssembly à tout projet Angular, dans de nombreux cas, cela peut même s'avérer contre-productif.

En revanche, si vos programmes demandent un temps d'exécution assez conséquent, WebAssembly intégré à Angular peut s'avérer très performant :

 

[![](/assets/images/play-with-memory-table.png)](/assets/images/play-with-memory-table.png)  
**Résultats pour playWithMemory**

## Conclusion

WebAssembly n'en est aujourd'hui qu'à ses débuts, les tests effectués dans le cadre de cet article restent minimes et des performances réellements impressionnantes peuvent êtres atteintes en calcul 3D ([voir l'article de Kamaron Peterson](https://www.lucidchart.com/techblog/2017/05/16/webassembly-overview-so-fast-so-fun-sorta-difficult/)). Son intégration à Angular est plutôt complexe et peu performante dans le cadre d'un développement front peu coûteux en ressources, donc à moins de devoir ré-afficher la fractale de Mandelbrot à chaque page de votre app, vous pouvez passer votre chemin.

 

_Notes:_ _vous pouvez cloner le projet de benchmark réalisé [ici](https://github.com/3IE/article-webassembly) pour avoir une meilleure idée des performances entre WebAssembly et TypeScript ou simplement démarrer sur un petit projet clefs en main._
<br>
<br>

---------------------------------------
<br>
Auteur: **leo.stephan**

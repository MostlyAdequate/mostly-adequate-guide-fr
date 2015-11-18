# Chapitre 5: Composer, c'est coder !

## L'élevage de fonctions

Voici `compose`:

```js
var compose = function(f,g) {
  return function(x) {
    return f(g(x));
  };
};
```

`f` et `g` sont des fonctions et `x` la valeur qu'elles se partagent.

La composition, c'est un peu de l'élevage de fonctions. Vous, fier éleveur de fonctions,
sélectionnez deux caractéristiques que vous souhaitez combiner et les mixer ensemble afin d'en
créer une d'un nouveau genre. Voici l'idée:

```js
var toUpperCase = function(x) { return x.toUpperCase(); };
var exclaim = function(x) { return x + '!'; };
var shout = compose(exclaim, toUpperCase);

shout("send in the clowns");
//=> "SEND IN THE CLOWNS!"
```

La composition de deux fonctions est une fonction et c'est tout à fait logique: composer deux
entités d'un même type (des fonctions dans notre cas) doit donner naissance à une nouvelle
entité de ce type très précisément. Ce n'est pas en assemblant deux légos que vous obtiendrez
des playmobiles. Ceci repose bien entendu sur un théorie et un ensemble de lois que nous
découvrirons en temps voulus.

Telle que nous l'avons définie `compose` applique `g` avant `f` et se faisant, créé un flot de
donnée se propageant de la droite vers la gauche. C'est bien plus lisible que d'imbriquer toute
une ribambelle d'appels. Sans composition, l'exemple précédent ressemblerait à ceci:

```js
var shout = function(x){
  return exclaim(toUpperCase(x));
};
```

Avec la composition, nous lisons de droite a gauche et non de l'intérieur vers l'extérieur.
Voyons voir un exemple où l'ordre à une importance:

```js
var head = function(x) { return x[0]; };
var reverse = reduce(function(acc, x){ return [x].concat(acc); }, []);
var last = compose(head, reverse);

last(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'uppercut'
```

`reverse` retourne l'ordre de la liste et `head` nous permet de récupérer le premier élément de
celle-ci. Il en résulte une fonction `last` efficace bien qu'inefficiente. L'enchaînement des
fonctions de la composition doit apparaître clairement. En outre, en composant de droite à
gauche l'on reflète le comportement de la composition d'un point de vue mathématique. Hé oui,
la composition est un concept tout droit sorti des livres de maths. Peut-être serait-il
intéressant de jetez un oeil à une propriété vérifiée par n'importe quelle composition.

```js
// associativité
var associative = compose(f, compose(g, h)) == compose(compose(f, g), h);
// true
```

La composition est associative: la façon de grouper deux d'entre-elles ne change pas la
fonction composée.  De fait, on peut de façon équivalente composer `toUpperCase` des deux
façons suivantes:

```js
compose(toUpperCase, compose(head, reverse));

// ou
compose(compose(toUpperCase, head), reverse);
```

Parce que la façon de grouper nos appels n'importe pas, le résultat sera le même. Cette
propriété intéressante nous permet ainsi d'écrire une version variadique de la fonction de
composition et de l'utiliser comme ceci:

```js
// auparavant, il aurait fallu effectuer deux appels à compose, mais grâce à l'associativité,
// il nous est possible de spécifier autant de fonctions que nous le souhaitons.
var lastUpper = compose(toUpperCase, head, reverse);

lastUpper(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'UPPERCUT'


var loudLastUpper = compose(exclaim, toUpperCase, head, reverse)

loudLastUpper(['jumpkick', 'roundhouse', 'uppercut']);
//=> 'UPPERCUT!'
```

L'associativité nous assure flexibilité et séreinité quant au résultat de la composition. La
version variadique étendue est inclue avec les différentes bibliothèques annexes de ce livre.
C'est par ailleurs la définition classique que vous pourriez trouver dans des bibliothèques
telles que [loadash][lodash-website], [underscore][underscore-website] et
[ramda][ramda-website]

L'un des attraits de l'associativité est de pouvoir créer de nouvelles fonctions simplement en
utilisant et composant n'importe quelles autres fonctions existantes. Modifions un peu notre
exemple précédent.

```js
var loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

// ou
var last = compose(head, reverse);
var loudLastUpper = compose(exclaim, toUpperCase, last);

// ou
var last = compose(head, reverse);
var angry = compose(exclaim, toUpperCase);
var loudLastUpper = compose(angry, last);

// et plus encore...
```

Il n'y a pas de bonne ou de mauvaises réponses - simplement des légos à assembler selon notre
bon-vouloir. De façon générale il est préférable de grouper des méthodes de telles sortes que
le résultat puisse être aisément réutiliser (comme `last` et `angry`). Ceux qui sont familiers
du livre de Fowler à propos du "[Refactoring][refactoring-book]" reconnaîtrons la [méthode
d'extraction][extract-method-refactor] ... cela dit sans toute la gestion des états des objets
à considérer.

## Pointfree

Le style *pointfree* (littéralement, "sans point") decrit un style où les données ne sont jamais
mentionnées; ou plutôt devrais-je dire que les fonctions n'ont jamais à explicitement faire
référence aux données qu'elles manipulent. Les fonctions de premier ordre, la curryfication et
la composition interagissent merveilleusement bien afin de procéder ainsi, *pointfree*.

```js
//Pas Pointfree car on se réfère explicitement à `word`
var snakeCase = function (word) {
  return word.toLowerCase().replace(/\s+/ig, '_');
};

//Pointfree
var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```

Voyez comment nous avons partiellement appliqué `replace`. Les données sont canalisées d'une
fonction à l'autre. La curryfication nous permet de mettre en forme chaque fonction de façon à
ce qu'elle n'opère que sur ses données avant de les passer à la suivante. Par ailleurs,
soulignons qu'il n'y a nullement besoin des données pour construire la fonction composée dans
cette version *pointfree* alors que dans la version avec point, il faut faire explicitement
référence au `word` afin de construire la fonction.

Jetons un oeil à un autre exemple.

```js
//Pas Pointfree car on se réfère explicitement à `name`
var initials = function (name) {
  return name.split(' ').map(compose(toUpperCase, head)).join('. ');
};

//Pointfree
var initials = compose(join('. '), map(compose(toUpperCase, head)), split(' '));

initials("hunter stockton thompson");
// 'H. S. T'
```

Un code *pointfree* sert également à retirer les noms inutiles pour ne garder qu'une essence
concise et générique de ce dernier. En pratique, un code *pointfree* se teste facilement en ce
qu'il s'exprime comme de petites fonctions transformant une entrée en une sortie. Difficile de
composer une boucle *while* par exemple. Gardez toutefois une chose à l'esprit, le style
*pointfree* est à double tranchant et peut parfois obfusquer les réelles intentions du code. Un
code 100% fonctionnel n'est pas a priori 100% *pointfree* - et ça ne pose aucun problème. Nous
verrons des cas où il est pertinent de conserver un style conventionnel.

## Débuggage

Une erreur classique consiste à composer une fonction comme `map` d'arité 2 sans partiellement
l'appliquer à un premier argument au préalable.

```js
//incorrect - `angry` se voit appliqué à une liste d'élément et `map` partiellement appliqué à on ne sait trop quoi
var latin = compose(map, angry, reverse);

latin(["frog", "eyes"]);
// erreur


// mieux - chaque fonction n'attend qu'un seul argument
var latin = compose(map(angry), reverse);

latin(["frog", "eyes"]);
// ["EYES!", "FROG!"])
```

En cas de problème pour débugguer une composition, il est possible d'avoir recours à cette
fonction utile bien qu'impure pour afficher quelques traces:

```js
var trace = curry(function(tag, x){
  console.log(tag, x);
  return x;
});

var dasherize = compose(join('-'), toLower, split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');
// TypeError: Cannot read property 'apply' of undefined
```

Il y a un soucis ici, affichons la trace.

```js
var dasherize = compose(join('-'), toLower, trace("after split"), split(' '), replace(/\s{2,}/ig, ' '));
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]
```

Ah! Il faut *mapper* sur `toLower` étant donné qu'elle reçoit une liste

```js
var dasherize = compose(join('-'), map(toLower), split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');

// 'the-world-is-a-vampire'
```

La fonction `trace` nous permet de visualiser les données à un certain point précis pour
déceler un éventuel problème. Des langages comme Haskell et PureScript proposent des fonctions
similaires afin de faciliter le développement.

La composition sera notre outil phare pour élaborer des programmes. De plus, il s'inscrit au
sein d'une puissante théorie justifiant de la validité de nos propositions. Examinons ladite
théorie.

## La théorie des Catégories

La théorie des Catégories est une branche des mathématiques qui formalise différents concepts
provenant d'autres branches et théories telles que la théorie des ensembles, la théorie des
types, la théories des groupes, la logique et bien plus encore. Elle manipule des objets,
des morphismes and des transformations qui reflètent les besoins de la programmation assez
fidèlement. Ci-après un petit tableau comparatif de la perception de ces concepts au sein des
différentes théories énoncées.

<img src="images/cat_theory.png" />

Si c'est le cas, je suis navré de vous avoir effrayé. Je n'attends pas de vous que vous soyez
intiment familier à ces concepts. Je souhaite simplement vous faire sentir l'objectif que tente
de réaliser la théorie des Catégories en unifiant les concepts présents dans chacune des
autres.

En théorie des catégories, nous allons être amené a nous référer à... des catégories. Elles
peuvent se voir comme des collections composées des éléments suivants:

  - Une collection d'objets
  - Une collection de morphismes
  - Une notion de composition de ces morphismes
  - Un morphisme particulier appelé identité.

La théorie des catégories est suffisament abstraite pour modéliser à peu près n'importe quoi.
Appliquons-là toutefois aux types et aux fonctions car c'est pour le moment, tout ce qui nous
intéresse.

**Une collection d'objets**
Les objets représentent nos différents type de données. Par exemple, ``String``, ``Boolean``,
``Number``, ``Object``, etc. Il est courant de parler des types comme de l'ensemble des valeurs
qu'ils permettent de représenter. Considérer les types comme des ensembles est fort pratique
lorsqu'on se réfère à la théorie des ensembles pour les manipuler.

**Une collection de morphismes** 
Les morphismes vont consituer nos fonctions pures telles que nous les avons définies.

**Une notion de composition de ces morphismes**
Ici, vous l'aurez compris, il s'agit de notre nouveau jouet fétiche - `compose`. Nous savons de
plus déjà que la fonction `compose` est associative ce qui, non fortuitement, est une propriété
que doit posséder la composition en théorie des catégories.

Voici un petit schéma mettant en avant la composition:

<img src="images/cat_comp1.png" />
<img src="images/cat_comp2.png" />

Et ci-dessous, un petit exemple concret:

```js
var g = function(x){ return x.length; };
var f = function(x){ return x === 4; };
var isFourLetterWord = compose(f, g);
```

**Un morphisme particulier appelé identité**

Faisons la connaissance d'une fonction bien pratique que nous appelerons `id`. Cette fonction
prend simplement des entrées, et vous les recrache telles quelles. Voyez plutôt:

```js
var id = function(x){ return x; };
```

Vous vous demandez sans doute "En quoi cette stupide fonction m'avance à quelque chose ?". Nous
ferons un usage extensif de cette fonction dans les chapitres à venir mais pour l'instant
contentez vous de voir cette fonction comme une valeur que l'on peut passer. 
En outre, `id` a quelques propriété intéressante face à la composition. Voici une propriété
toujours vraie quelquesoit la fonction f d'arité un (arité un: fonction d'un seul argument):

```js
// identité
compose(id, f) == compose(f, id) == f;
// true
```

Hey, c'est exactement comme la propriété d'identité sur les nombres ! Si c'est encore un peu
flou, laissez fermenter un peu dans votre esprit. Comprenez l'essentiel. Nous allons bientôt
utiliser `id` un petit peu partout, mais pour l'instant, voyons cela comme une simple fonction
qui retourne son entrée. C'est utile afin d'écrire du code *pointfree* nous le verrons.

Avec cela néanmoins vous avez dorénavant une catégorie de types de fonctions. Si c'est votre
première introduction de ce type, vous êtes sans doute encore un peu déconcerté face au concept
de catégorie et son utilité. Tout au long de ce livre, nous nous efforcerons d'en améliorer
votre intuition. Pour le moment, alors même que vous lisez ces quelques lignes, dites-vous que
c'est une façon de donner quelques solides justifications à l'utilisation de la composition - à
savoir, l'associativité et les propriétés d'identité.

What are some other categories, you ask? Well, we can define one for directed graphs with nodes being objects, edges being morphisms, and composition just being path concatenation. We can define with Numbers as objects and `>=` as morphisms(actually any partial or total order can be a category). There are heaps of categories, but for the purposes of this book, we'll only concern ourselves with the one defined above. We have sufficiently skimmed the surface and must move on.


## In Summary
Composition connects our functions together like a series of pipes. Data will flow through our application as it must - pure functions are input to output after all so breaking this chain would disregard output, rendering our software useless.

We hold composition as a design principle above all others. This is because it keeps our app simple and reasonable. Category theory will play a big part in app architecture, modelling side effects, and ensuring correctness.

We are now at a point where it would serve us well to see some of this in practice. Let's make an example application.

[Chapter 6: Example Application](ch6.md)

## Exercises

```js
var _ = require('ramda');
var accounting = require('accounting');
  
// Example Data
var CARS = [
    {name: "Ferrari FF", horsepower: 660, dollar_value: 700000, in_stock: true},
    {name: "Spyker C12 Zagato", horsepower: 650, dollar_value: 648000, in_stock: false},
    {name: "Jaguar XKR-S", horsepower: 550, dollar_value: 132000, in_stock: false},
    {name: "Audi R8", horsepower: 525, dollar_value: 114200, in_stock: false},
    {name: "Aston Martin One-77", horsepower: 750, dollar_value: 1850000, in_stock: true},
    {name: "Pagani Huayra", horsepower: 700, dollar_value: 1300000, in_stock: false}
  ];

// Exercise 1:
// ============
// use _.compose() to rewrite the function below. Hint: _.prop() is curried.
var isLastInStock = function(cars) {
  var last_car = _.last(cars);
  return _.prop('in_stock', last_car);
};

// Exercise 2:
// ============
// use _.compose(), _.prop() and _.head() to retrieve the name of the first car
var nameOfFirstCar = undefined;


// Exercise 3:
// ============
// Use the helper function _average to refactor averageDollarValue as a composition
var _average = function(xs) { return _.reduce(_.add, 0, xs) / xs.length; }; // <- leave be

var averageDollarValue = function(cars) {
  var dollar_values = _.map(function(c) { return c.dollar_value; }, cars);
  return _average(dollar_values);
};


// Exercise 4:
// ============
// Write a function: sanitizeNames() using compose that returns a list of lowercase and underscored car's names: e.g: sanitizeNames([{name: "Ferrari FF", horsepower: 660, dollar_value: 700000, in_stock: true}]) //=> ["ferrari_ff"].

var _underscore = _.replace(/\W+/g, '_'); //<-- leave this alone and use to sanitize

var sanitizeNames = undefined;


// Bonus 1:
// ============
// Refactor availablePrices with compose.

var availablePrices = function(cars) {
  var available_cars = _.filter(_.prop('in_stock'), cars);
  return available_cars.map(function(x){
    return accounting.formatMoney(x.dollar_value);
  }).join(', ');
};


// Bonus 2:
// ============
// Refactor to pointfree. Hint: you can use _.flip()

var fastestCar = function(cars) {
  var sorted = _.sortBy(function(car){ return car.horsepower }, cars);
  var fastest = _.last(sorted);
  return fastest.name + ' is the fastest';
};
```

[lodash-website]: https://lodash.com/
[underscore-website]: http://underscorejs.org/
[ramda-website]: http://ramdajs.com/
[refactoring-book]: http://martinfowler.com/books/refactoring.html
[extract-method-refactor]: http://refactoring.com/catalog/extractMethod.html

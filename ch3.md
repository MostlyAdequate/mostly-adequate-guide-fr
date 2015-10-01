# Chapitre 3 : Du pur bonheur avec du pur fonctionnel

## Soyez pur à nouveau

S'il y a une chose qu'il faut avoir bien compris, c'est le concept de fonction pure.

> Une fonction pure est une fonction qui au regard des mêmes entrées fournit toujours la même
> sortie et le fait sans aucun effet de bord visible. 

Prenez `slice` et `splice`. Ces deux fonctions font exactement la même chose - d'une façon bien
différente, je vous l'accorde, mais néanmoins la même chose. On dit que `slice` est pure car
elle retourne toujours la même sortie pour une même entrée, c'est garanti. `splice` en revanche
grignotera volontiers une partie de la liste d'entrée et l'altérera à jamais; c'est un effet de
bord observable. 

```js
var xs = [1,2,3,4,5];

// pure
xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]

xs.slice(0,3);
//=> [1,2,3]


// impure
xs.splice(0,3);
//=> [1,2,3]

xs.splice(0,3);
//=> [4,5]

xs.splice(0,3);
//=> []
```

En programmation fonctionnelle, nous n'aimons pas les fonctions complexes telles que `splice`
qui modifient (*mutate*) les entrées. Cela ne convient pas; nous recherchons des fonctions de
confiance qui nous retournerons toujours le même résultat, pas des fonctions qui laisseront
derrière elles un fatras d'effets peu désirables (comme `splice`).

Jetons un oeil à l'exemple suivant.

```js
// impure
var minimum = 21;

var checkAge = function(age) {
  return age >= minimum;
};


// pure
var checkAge = function(age) {
  var minimum = 21;
  return age >= minimum;
};
```

Dans la portion impure, `checkAge` dépend d'une variable mutable `minimum` afin de déterminer
le résultat. En d'autres termes, la fonction dépend de l'état courant du système et introduit
dans le même temps un environnement externe difficile à considérer.

Cela peut sembler anodin dans cet exemple mais pouvoir se fier ainsi à l'état des variables est
une condition nécessaire à l'élaboration de systèmes complexes
[^http://www.curtclifton.net/storage/papers/MoseleyMarks06a.pdf]. Selon ses entrées la fonction
`checkAge` peut retourner des résultats différents ce qui d'une part n'est pas conforme à la
notion de pureté mais qui d'autre part, force votre esprit à être en alerte dès lors qu'une
modification minime doit s'opérer sur le programme. 

Dans sa forme pure en revanche, la fonction est totalement autonome et hermétique. Il est aussi
possible de rendre `minimum` immutable, renforçant dans le même temps la pureté de la fonction.
Pour ce faire, créons un objet à geler. 

```js
var immutableState = Object.freeze({
  minimum: 21
});
```

## Les effets de bords c'est aussi...

Jetons un oeil plus attentif à ces "effets de bords". Quels sont donc ces abominables *effets
de bord* que l'on mentionne dans la définition de *fonction pure* ? Nous désignerons par effet
tout ce qui peut arriver au cours d'une exécution en dehors du calcul d'un résultat. 

Il n'y a rien d'intrinsèquement mauvais dans les effets si bien que nous les
utiliserons bientôt à tout va dans les prochains chapitres. C'est la partie sur le *bord* qui a
mauvaise réputation. L'eau seule n'a rien d'un incubateur à larves, ce sont les parties
*stagnantes* qui créent des marais répugnants, et je vous l'assure, les effets de *bord* sont
en tout point similaire pour votre programme.

> Un *effet de bord* est un changement de l'état du système ou une *interaction visible* avec le
> monde extérieur qui se produit lors du calcul d'un résultat.

Ceci inclut mais n'est pas limité à / aux :

- Modifier un fichier du système
- Ajouter une entrée à une base de données
- Effectuer une requête http
- Assignations et changements d'état de variables
- Afficher à l'écran ou dans la console
- Demander une entrée utilisateur
- Accéder au DOM
- Accéder à une information de l'environnement système

La liste continue ainsi de suite. Toutes interactions avec le monde en dehors d'une fonction
est un effet de bord, ce qui vous laisse entrevoir la commodité qu'ils constituent. Toutefois,
l'un des postulats de la programmation fonctionnelle stipule que les effets de bords sont
généralement la cause de comportements hasardeux. 

Il ne s'agit pas de ne pas les utiliser, mais plutôt, d'apprendre à les contenir et à s'en
servir de façon contrôlée. Nous aborderons les monades et foncteurs au cours des prochains
chapitres cependant pour l'instant, veillons à bien séparer ces fonctions piégeuses des autres
plus pures.

Les effets de bord empêchent une fonction d'être pure, ça tombe sous le sens : les fonctions
pures par définition doivent nécessairement retourner la même sortie par rapport à des entrées
données ce qui ne peut être garanti dès lors que la fonction se repose sur autre chose que son
corps local. 

Analysons plus attentivement pourquoi nous insistons autant sur cette histoire de même sortie
pour une même entrée. Rhabillez-vous, nous partons faire des Mathématiques de haut vol. 

## BAC+8 de Maths

Tiré et traduit de mathisfun.com:

> Une fonction est une relation privilégiée entre deux valeurs :
> à chacune de ses valeurs d'entrée est associée une unique valeur de sortie.

En d'autre terme, c'est simplement une relation entre deux valeurs : l'entrée et la sortie.
Néanmoins bien qu'à chaque entrée soit associée une et une seule sortie, cette sortie n'est pas
forcément unique à une entrée donnée. Vous trouverez ci-après un diagramme
d'une fonction tout à fait valide qui à `x` associe `y`.

<img src="images/function-sets.gif" />[^http://www.mathsisfun.com/sets/function.html]

En comparaison, le diagramme suivant montre une relation qui n'est *pas* une fonction en ce que
la valeur d'entrée `5` pointe sur plusieurs sorties.

<img src="images/relation-not-function.gif" />[^http://www.mathsisfun.com/sets/function.html]

Les fonctions peuvent être décrites comme un ensemble de paires (ou couples) de la forme:
(input, output) : `[(1,2), (3, 6), (5,10)]`[^Vous noterez que cette fonction semble doubler la
valeur de chaque entrée]

Ou encore sous forme de tableau :
<table> <tr> <th>Input</th> <th>Output</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

Mais aussi en tant que courbe avec `x` comme entrée et `y` comme sortie:

<img src="images/fn_graph.png" width="300" height="300" />

Lorsque les entrées impliquent directement les sorties, il n'y a pas lieu de considérer les
détails d'implémentation d'une fonction. N'étant alors qu'un ensemble d'associations d'une
entrée à une sortie, on pourrait imaginer représenter les fonctions à l'aide d'une objet
littéral et les appeler avec `[]` en lieu et place des habituelles `()`.

```js
var toLowerCase = {"A":"a", "B": "b", "C": "c", "D": "d", "E": "e", "D": "d"};

toLowerCase["C"];
//=> "c"

var isPrime = {1:false, 2: true, 3: true, 4: false, 5: true, 6:false};

isPrime[3];
//=> true
```

Bien entendu, on préfèrera calculer plutôt que d'écrire à la main chacune des valeurs de
sortie. Ceci illustre toutefois une façon connexe de considérer les fonctions. [^Vous vous
posez peut-être la question des fonctions de plusieurs arguments. En effets, c'est à première
vue plutôt déroutant d'un point de vue mathématique. Pour l'heure, nous pouvons simplement
considérer un tableau ou une structure similaire au pseudo-objet `arguments` comme notre
entrée. Lorsqu'on nous apprendrons à propos de la *curryfication*, nous verrons comment
refléter au mieux la définition mathématique d'une fonction.]

Maintenant le fin mot de l'histoire : les fonctions pures *sont* des fonctions au sens
mathématique et c'est ce sur quoi se fonde fondamentalement la programmation fonctionnelle.
Programmer à l'aide de ces petites bêtes sages apporte d'énormes avantages. Regardons quelques
raisons qui justifient notre irrémédiable envie de préserver la pureté de nos fonctions.

## The case for purity

### Cacheable

For starters, pure functions can always be cached by input. This is typically done using a technique called memoization:

```js
var squareNumber  = memoize(function(x){ return x*x; });

squareNumber(4);
//=> 16

squareNumber(4); // returns cache for input 4
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // returns cache for input 5
//=> 25
```

Here is a simplified implementation, though there are plenty of more robust versions available.

```js
var memoize = function(f) {
  var cache = {};

  return function() {
    var arg_str = JSON.stringify(arguments);
    cache[arg_str] = cache[arg_str] || f.apply(f, arguments);
    return cache[arg_str];
  };
};
```

Something to note is that you can transform some impure functions into pure ones by delaying evaluation:

```js
var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

The interesting thing here is that we don't actually make the http call - we instead return a function that will do so when called. This function is pure because it will always return the same output given the same input: the function that will make the particular http call given the `url` and `params`.

Our `memoize` function works just fine, though it doesn't cache the results of the http call, rather it caches the generated function.

This is not very useful yet, but we'll soon learn some tricks that will make it so. The takeaway is that we can cache every function no matter how destructive they seem.

### Portable / Self-Documenting

Pure functions are completely self contained. Everything the function needs is handed to it on a silver platter. Ponder this for a moment... How might this be beneficial? For starters, a function's dependencies are explicit and therefore easier to see and understand - no funny business going on under the hood.

```js
//impure
var signUp = function(attrs) {
  var user = saveUser(attrs);
  welcomeUser(user);
};

//pure
var signUp = function(Db, Email, attrs) {
  return function() {
    var user = saveUser(Db, attrs);
    welcomeUser(Email, user);
  };
};
```

The example here demonstrates that the pure function must be honest about its dependencies and, as such, tell us exactly what it's up to. Just from its signature, we know that it will use a `Db`, `Email`, and `attrs` which should be telling to say the least.

We'll learn how to make functions like this pure without merely deferring evaluation, but the point should be clear that the pure form is much more informative than its sneaky impure counterpart which is up to God knows what.

Something else to notice is that we're forced to "inject" dependencies, or pass them in as arguments, which makes our app much more flexible because we've parameterized our database or mail client or what have you[^Don't worry, we'll see a way to make this less tedious than it sounds]. Should we choose to use a different Db we need only to call our function with it. Should we find ourselves writing a new application in which we'd like to reuse this reliable function, we simply give this function whatever `Db` and `Email` we have at the time.

In a JavaScript setting, portability could mean serializing and sending functions over a socket. It could mean running all our app code in web workers. Portability is a powerful trait.

Contrary to "typical" methods and procedures in imperative programming rooted deep in their environment via state, dependencies, and available effects, pure functions can be run anywhere our hearts desire.

When was the last time you copied a method into a new app? One of my favorite quotes comes from Erlang creator, Joe Armstrong: "The problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana... and the entire jungle".

### Testable

Next, we come to realize pure functions make testing much easier. We don't have to mock a "real" payment gateway or setup and assert the state of the world after each test. We simply give the function input and assert output.

In fact, we find the functional community pioneering new test tools that can blast our functions with generated input and assert that properties hold on the output. It's beyond the scope of this book, but I strongly encourage you to search for and try *Quickcheck* - a testing tool that is tailored for a purely functional environment.

### Reasonable

Many believe the biggest win when working with pure functions is *referential transparency*. A spot of code is referentially transparent when it can be substituted for its evaluated value without changing the behavior of the program.

Since pure functions always return the same output given the same input, we can rely on them to always return the same results and thus preserve referential transparency. Let's see an example.

```js

var Immutable = require("immutable");

var decrementHP = function(player) {
  return player.set("hp", player.get("hp")-1);
};

var isSameTeam = function(player1, player2) {
  return player1.get("team") === player2.get("team");
};

var punch = function(player, target) {
  if(isSameTeam(player, target)) {
    return target;
  } else {
    return decrementHP(target);
  }
};

var jobe = Immutable.Map({name:"Jobe", hp:20, team: "red"});
var michael = Immutable.Map({name:"Michael", hp:20, team: "green"});

punch(jobe, michael);
//=> Immutable.Map({name:"Michael", hp:19, team: "green"})
```

`decrementHP`, `isSameTeam` and `punch` are all pure and therefore referentially transparent. We can use a technique called *equational reasoning* wherein one substitutes "equals for equals" to reason about code. It's a bit like manually evaluating the code without taking into account the quirks of programmatic evaluation. Using referential transparency, let's play with this code a bit.

First we'll inline the function `isSameTeam`.

```js
var punch = function(player, target) {
  if(player.get("team") === target.get("team")) {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Since our data is immutable, we can simply replace the teams with their actual value

```js
var punch = function(player, target) {
  if("red" === "green") {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

We see that it is false in this case so we can remove the entire if branch

```js
var punch = function(player, target) {
  return decrementHP(target);
};

```

And if we inline `decrementHP`, we see that, in this case, punch becomes a call to decrement the `hp` by 1.

```js
var punch = function(player, target) {
  return target.set("hp", target.get("hp")-1);
};
```

This ability to reason about code is terrific for refactoring and understanding code in general. In fact, we used this technique to refactor our flock of seagulls program. We used equational reasoning to harness the properties of addition and multiplication. Indeed, we'll be using these techniques throughout the book.

### Parallel Code

Finally, and here's the coup de grâce, we can run any pure function in parallel since it does not need access to shared memory and it cannot, by definition, have a race condition due to some side effect.

This is very much possible in a server side js environment with threads as well as in the browser with web workers though current culture seems to avoid it due to complexity when dealing with impure functions.


## In Summary

We've seen what pure functions are and why we, as functional programmers, believe they are the cat's evening wear. From this point on, we'll strive to write all our functions in a pure way. We'll require some extra tools to help us do so, but in the meantime, we'll try to separate the impure functions from the rest of the pure code.

Writing programs with pure functions is a tad laborious without some extra tools in our belt. We have to juggle data by passing arguments all over the place, we're forbidden to use state, not to mention effects. How does one go about writing these masochistic programs? Let's acquire a new tool called curry.

[Chapter 4: Currying](ch4.md)

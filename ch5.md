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

Pointfree style means never having to say your data. Excuse me. It means functions that never mention the data upon which they operate. First class functions, currying, and composition all play well together to create this style.

```js
//not pointfree because we mention the data: word
var snakeCase = function (word) {
  return word.toLowerCase().replace(/\s+/ig, '_');
};

//pointfree
var snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```

See how we partially applied `replace`? What we're doing is piping our data through each function of 1 argument. Currying allows us to prepare each function to just take its data, operate on it, and pass it along. Something else to notice is how we don't need the data to construct our function in the pointfree version, whereas in the pointful one, we must have our `word` available before anything else.

Let's look at another example.

```js
//not pointfree because we mention the data: name
var initials = function (name) {
  return name.split(' ').map(compose(toUpperCase, head)).join('. ');
};

//pointfree
var initials = compose(join('. '), map(compose(toUpperCase, head)), split(' '));

initials("hunter stockton thompson");
// 'H. S. T'
```

Pointfree code can again, help us remove needless names and keep us concise and generic. Pointfree is a good litmus test for functional code as it let's us know we've got small functions that take input to output. One can't compose a while loop, for instance. Be warned, however, pointfree is a double edge sword and can sometimes obfuscate intention. Not all functional code is pointfree and that is O.K. We'll shoot for it where we can and stick with normal functions otherwise.

## Debugging
A common mistake is to compose something like `map`, a function of two arguments, without first partially applying it.

```js
//wrong - we end up giving angry an array and we partially applied map with god knows what.
var latin = compose(map, angry, reverse);

latin(["frog", "eyes"]);
// error


// right - each function expects 1 argument.
var latin = compose(map(angry), reverse);

latin(["frog", "eyes"]);
// ["EYES!", "FROG!"])
```

If you are having trouble debugging a composition, we can use this helpful, but impure trace function to see what's going on.

```js
var trace = curry(function(tag, x){
  console.log(tag, x);
  return x;
});

var dasherize = compose(join('-'), toLower, split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');
// TypeError: Cannot read property 'apply' of undefined
```

Something is wrong here, let's `trace`

```js
var dasherize = compose(join('-'), toLower, trace("after split"), split(' '), replace(/\s{2,}/ig, ' '));
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]
```

Ah! We need to `map` this `toLower` since it's working on an array.

```js
var dasherize = compose(join('-'), map(toLower), split(' '), replace(/\s{2,}/ig, ' '));

dasherize('The world is a vampire');

// 'the-world-is-a-vampire'
```

The `trace` function allows us to view the data at a certain point for debugging purposes. Languages like haskell and purescript have similar functions for ease of development.

Composition will be our tool for constructing programs and, as luck would have it, is backed by a powerful theory that ensures things will work out for us. Let's examine this theory.


## Category theory

Category theory is an abstract branch of mathematics that can formalize concepts from several different branches such as set theory, type theory, group theory, logic, and more. It primarily deals with objects, morphisms, and transformations, which mirrors programming quite closely. Here is a chart of the same concepts as viewed from each separate theory.

<img src="images/cat_theory.png" />

Sorry, I didn't mean to frighten you. I don't expect you to be intimately familiar with all these concepts. My point is to show you how much duplication we have so you can see why category theory aims to unify these things.

In category theory, we have something called... a category. It is defined as a collection with the following components:

  * A collection of objects
  * A collection of morphisms
  * A notion of composition on the morphisms
  * A distinguished morphism called identity

Category theory is abstract enough to model many things, but let's apply this to types and functions, which is what we care about at the moment.

**A collection of objects**
The objects will be data types. For instance, ``String``, ``Boolean``, ``Number``, ``Object``, etc. We often view data types as sets of all the possible values. One could look at ``Boolean`` as the set of `[true, false]` and ``Number`` as the set of all possible numeric values. Treating types as sets is useful because we can use set theory to work with them. 


**A collection of morphisms**
The morphisms will be our standard every day pure functions.

**A notion of composition on the morphisms**
This, as you may have guessed, is our brand new toy - `compose`. We've discussed that our `compose` function is associative which is no coincidence as it is a property that must hold for any composition in category theory.

Here is an image demonstrating composition:

<img src="images/cat_comp1.png" />
<img src="images/cat_comp2.png" />

Here is a concrete example in code:

```js
var g = function(x){ return x.length; };
var f = function(x){ return x === 4; };
var isFourLetterWord = compose(f, g);
```

**A distinguished morphism called identity**
Let's introduce a useful function called `id`. This function simply takes some input and spits it back at you. Take a look:

```js
var id = function(x){ return x; };
```

You might ask yourself "What in the bloody hell is that useful for?". We'll make extensive use of this function in the following chapters, but for now think of it as a function that can stand in for our value - a function masquerading as every day data.

`id` must play nicely with compose. Here is a property that always holds for every unary(unary: a one argument function) function f:

```js
// identity
compose(id, f) == compose(f, id) == f;
// true
```

Hey, it's just like the identity property on numbers! If that's not immediately clear, take some time with it. Understand the futility. We'll be seeing `id` used all over the place soon, but for now we see it's a function that acts as a stand in for a given value. This is quite useful when writing pointfree code.

So there you have it, a category of types and functions. If this is your first introduction, I imagine you're still a little fuzzy on what a category is and why it's useful. We will build upon this knowledge throughout the book. As of right now, in this chapter, on this line, you can at least see it as providing us with some wisdom regarding composition - namely, the associativity and identity properties.

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

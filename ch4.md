# Chapter 4: Curryfication

NdT : Appelé *currying* en anglais.

## Vivre sans toi n'est pas vivre

Mon père me disait qu'il y a des choses dont on peut se passer, jusqu'au jour où on les obtient. Les fours micro-onde en font partie. Les smartphones aussi. Les plus vieux d'entre nous se rappelleront d'une vie épanouie sans Internet. Pour moi, la curryfication est une de ces choses.

Le concept est simple : il est possible d'appeler une fonction avec moins d'arguments qu'elle n'en attend. Est alors retournée une fonction qui attend les arguments restants.

On peut choisir d'appeler la fonction en fournissant tous ses arguments d'un coup, ou bien petit à petit.

```js
var add = function(x) {
  return function(y) {
    return x + y;
  };
};

var increment = add(1);
var addTen = add(10);

increment(2);
// 3

addTen(2);
// 12
```

Ici, nous créons une fonction `add` qui prend un argument et retourne une fonction. Appeler `add` retourne une fonction qui se souvient, par closure, de l'argument `x` alors passé. Appeler `add` avec les deux arguments d'un coup est un peu pénible cependant, nous pouvons alors utiliser une fonction auxiliaire nommée `curry` pour faciliter l'écriture et l'appel de ce genre de fonction.

Mettons en place quelques fonctions currifiées pour notre propre satisfaction.

```js
var curry = require('lodash.curry');

var match = curry(function(what, str) {
  return str.match(what);
});

var replace = curry(function(what, replacement, str) {
  return str.replace(what, replacement);
});

var filter = curry(function(f, ary) {
  return ary.filter(f);
});

var map = curry(function(f, ary) {
  return ary.map(f);
});
```

La manière dont j'ai écrit ces fonctions est simple, mais importante. J'ai savamment placé les données manipulées (String, Array) en derniers dans la liste d'arguments. Les raisons de cette pratique apparaîtront à l'utilisation.

```js
match(/\s+/g, "hello world");
// [ ' ' ]

match(/\s+/g)("hello world");
// [ ' ' ]

var hasSpaces = match(/\s+/g);
// function(x) { return x.match(/\s+/g) }

hasSpaces("hello world");
// [ ' ' ]

hasSpaces("spaceless");
// null

filter(hasSpaces, ["piaf_orthographe", "Edith Piaf"]);
// ["Edith Piaf"]

var findSpaces = filter(hasSpaces);
// function(xs) { return xs.filter(function(x) { return x.match(/\s+/g) }) }

findSpaces(["edith", "Edith Piaf"]);
// ["Edith Piaf"]

var noVowels = replace(/[aeiou]/ig);
// function(replacement, x) { return x.replace(/[aeiou]/ig, replacement) }

var censored = noVowels("*");
// function(x) { return x.replace(/[aeiou]/ig, "*") }

censored("Le temps des cerises");
// 'L* t*mps d*s c*r*s*s'
```

Ce qui est démontré ici est la possibilité de "pré-charger" une fonction avec un argument ou deux afin d'obtenir une nouvelle fonction, plus spécifique, qui se souvient des arguments passés.

Je vous encourage à lancer `npm install lodash`, copier le code ci-dessus et l'essayer dans le REPL. Vous pouvez aussi faire ça dans un navigateur où lodash et ramda sont disponibles.

## Plus qu'un jeu de mot ou une sauce délicieuse

La curryfication est utile pour beaucoup de choses. Nous pouvons créer de nouvelles fonctions juste en donnant à nos fonctions de bases quelques arguments comme il a été fait pour `hasSpaces`, `findSpaces`, et `censored`.

Nous avons aussi la possibilité de transformer n'importe quelle fonction qui marche sur un seul élément en une fonction qui marche sur des tableaux simplement en la donnant à `map` :

```js
var getChildren = function(x) {
  return x.childNodes;
};

var allTheChildren = map(getChildren);
```

Ne fournir à une fonction qu'une partie de ses arguments est ce que l'on appelle faire une *application partielle* (*partial application* en anglais). Appliquer partiellement une fonction peut éliminer beaucoup de verbosité. Voyez donc ce à quoi ressemblerait la fonction `allTheChildren` précédente avec la version lodash non curryfiée de `map`[^notez que l'ordre des arguments change] :

```js
var allTheChildren = function(elements) {
  return _.map(elements, getChildren);
};
```

Il est rare que l'on définisse des fonctions qui marchent sur des tableaux, car il suffit d'appeler `map(getChildren)` en une seule ligne. Pareil pour `sort`, `filter`, et d'autres fonctions d'ordre supérieur.[^Fonction d'ordre supérieur: Une fonction qui prend ou retourne une fonction].

Lorsque l'on parle de *fonctions pures*, on dit qu'elles prennent une entrée et produisent une sortie. Curryfier fait exactement cela : chaque argument à son tour retourne une nouvelle fonction qui attend les arguments restants.

Peu importe si la sortie est une nouvelle fonction, elle est elle-même pure. Il est autorisé de donner plus d'un argument à la fois, mais ce n'est finalement que retirer les parenthèses `()` supplémentaires à des fins pratiques.

## En résumé

La curryfication est commode et j'apprécie hautement travailler au quotidien avec des fonctions curryfiées. C'est une nouvelle corde à son arc qui rend la programmation fonctionnelle moins verbeuse et plus agréable.

Nous pouvons créer de nouvelles fonctions à la volée, simplement en passant une poignée d'arguments et, en bonus, nous avons conservé la définition mathématique d'une fonction malgré plusieurs arguments.

Emparons nous maintenant d'un autre outil essentiel appelé la *composition* : la fonction `compose`.

[Chapitre 5: Programmer par composition](ch5.md)

## Exercices

Un rapide mot avant de commencer. Nous utiliserons une librairie nommée *ramda* qui curryfie chaque fonction par défaut. Alternativement, vous pouvez choisir d'utiliser *lodash-fp* qui fait la même chose et est écrite/maintenue par l'auteur de lodash. N'importe laquelle convient, ce n'est qu'affaire de préférence.

[ramda](http://ramdajs.com)
[lodash-fp](https://github.com/lodash/lodash-fp)

Des [test unitaires](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises) sont disponibles pour y confronter vos solutions à mesure que vous les rédigez. Vous pouvez aussi simplement copier-coller dans un REPL JavaScript pour les premiers exercices.

Les solutions et leur codes sont fournis dans le [dépôt de ce livre](https://github.com/DrBoolean/mostly-adequate-guide/tree/master/code/part1_exercises/answers)

```js
var _ = require('ramda');


// Exercice 1
//==============
// Réécrire en appliquant partiellement la fonction afin de ne plus avoir d'argument

var words = function(str) {
  return _.split(' ', str);
};

// Exercice 1a
//==============
// Utiliser map pour écrire une nouvelle fonction words qui fonctionne sur un tableau de chaînes de caractères 

var sentences = undefined;


// Exercice 2
//==============
// Réécrire en appliquant partiellement les fonctions afin de ne plus avoir d'argument

var filterQs = function(xs) {
  return _.filter(function(x){ return match(/q/i, x);  }, xs);
};


// Exercice 3
//==============
// Utiliser la fonction auxiliaire _keepHighest pour que max ne se réfère à plus aucun argument

// NE PAS TOUCHER:
var _keepHighest = function(x,y){ return x >= y ? x : y; };

// RÉÉCRIRE CELLE-CI:
var max = function(xs) {
  return _.reduce(function(acc, x){
    return _keepHighest(acc, x);
  }, -Infinity, xs);
};


// Bonus 1:
// ============
// Enrober la fonction slice des tableaux en la rendant fonctionnelle et curryfiée.
// //[1,2,3].slice(0, 2)
var slice = undefined;


// Bonus 2:
// ============
// Utiliser slice pour définir une fonction "take" qui prend les n premiers éléments d'une chaîne de caractères. Elle doit être curryfiée.
// // Le résultat pour "Quelque chose" avec n=7 devrait être "Quelque"
var take = undefined;
```


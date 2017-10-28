# Les oignons monadiques

## Fabrique à Foncteurs pointus

Avant d'aller plus loin, j'aimerais vous faire un aveu: je n'ai pas été des plus honnêtes au
sujet de cette méthode `of` que nous avons définie sur chacun de nos types. En réalité, il ne
s'agit pas tant d'éviter l'emploi du mot-clé `new` mais surtout de placer nos valeurs dans ce
que l'on appelle un contexte minimal de défaut. Ainsi, `of` ne prend pas simplement la place du
constructeur - il fait parti d'une interface bien plus importante qui répond au nom de
*Pointée*.  

> Un foncteur pointée est un foncteur muni d'une méthode `of`.

Ce qu'il faut retenir c'est que cette interface nous permet d'encapsuler n'importe quelle
valeur dans notre type pour ensuite y appliquer toute sorte de morphisme. 

```js
IO.of('tetris').map(concat(' master'));
// IO('tetris master')

Maybe.of(1336).map(add(1));
// Maybe(1337)

Task.of({
  id: 2,
}).map(_.prop('id'));
// Task(2)

Either.of('The past, present and future walk into a bar...').map(
  concat('it was tense.')
);
// Right('The past, present and future walk into a bar...it was tense.')
```

Souvenez-vous! Les constructeurs de `IO` et `Task` tout deux requièrent une fonction
contrairement aux constructeurs de `Maybe` et `Either` qui sont plus flexibles. On saisit là
toute la beauté de cette interface qui offre une même façon de venir placer des valeurs dans
notre foncteur en s'émancipant de la compléxité éventuelle d'un constructeur. C'est aussi ce
que l'on entend par "contexte minimal de défaut": fournir le strict minimum pour permettre
l'utilisation de `map` sur notre foncteur. 

Au risque de paraître pointilleux (sans mauvais jeu de mots), notez que définir `Left.of` ne
fait aucun sens. Il ne doit exister qu'une seule et unique manière de définir un context
minimal de défaut, et dans le cas de `Either` il s'agit de `Either.of` qui s'appuie sur `new
Right(x)`. Ici, `of` est définie à partir du constructeur de `Right` car son utilisation
sous-entend que l'on souhaite après pouvoir *mapper* et que donc, il ne s'agit pas de *mapper*
dans le vide! Normalement, avec les examples ci-dessus, vous devriez être en mesure d'avoir
construit votre propre intuition quant à l'utilisation de `of`.

Si vous n'être pas totalement étranger au jargon fonctionnelle, vous avez peut-être déjà
entendu parler de fonctions comme `pure`, `point`, `unit` ou `return`. Ce sont des cousins de
notre fonction `of` avec également leur part de mystère. `of` va s'avérer des plus utiles
lorsque nous jouerons avec les Monades et qu'il sera de notre devoir de replacer les valeurs au
sein du monade manuellement. 

Afin d'éviter l'utilisation du mot-clé `new` il existe différentes bibliothèques JavaScript que
je vous propose d'utiliser comme de vrais adultes responsables. Notament, je vous recommande
l'utilisation de `folktale`, `ramda` ou `fantasy-land` qui fournissent des définitions
correctes de `of` implémentées élégamment sans même faire appel a `new`. 

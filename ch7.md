# Hindley-Milner et moi

## Quel est ton type ?
Si vous êtes encore jeune et un peu étranger au paradigme fonctionnel, il ne faudra pas
longtemps pour que vous vous enlisiez jusqu'au cou, submergé par les signatures de types. Les
types forment un meta-langage qui permet aux développeurs de différents horizons de communiquer
promptement et efficacement. Dans la plupart des cas, c'est le système "Hindley-Milner" qui est
utilisé pour traduire ces types; ce chapitre en fera l'étude. 

Lorsqu'on travaille avec des fonctions pures, les signatures de types ont un pouvoir expressif
bien au delà de n'importe quel langage courant. Ces signatures vous murmurent au creu de
l'oreille les secrets les plus intimes d'une fonction. En une simple et concise ligne, elles
en exposent le comportement et les intentions. Il est aisé d'en déduire des théorèmes. On peut
inférer les types et il n'y a donc nullement besoin d'annotations contingentes. Une signature
peut être rendue plus ou moins précise, plus ou moins abstraite. Elles ne sont pas seulement
utiles au compilateur mais constituent vraisembablement la meilleure documentation disponible.
De fait, les signatures de types jouent un rôle plus qu'important dans le monde fonctionnel -
bien au delà de ce que vous pourriez imaginer de prime abord. 

Le JavaScript est un langage dynamique, cela ne signifie pas pour autant que l'on s'affranchit
totalement du typage. Les "strings", "numbers", "booleans" et autres types sont bien présents.
Ces informations ne sont simplement pas explicitement relayé par le langage mais sont par
ailleurs confinées dans un coin de notre esprit. Pas d'inquiétude à avoir, il suffit d'avoir
recours à d'astucieux commentaires de documentation pour servir ces signatures au besoin. 

Il existe de nombreux outils de vérification de types en JavaScript.
[Flow](http://flowtype.org/) par exemple en est un. On trouve aussi des dialectes du JavaScript
fortement typés comme [TypeScript](http://www.typescriptlang.org/). Dans ce livre toutefois,
nous nous efforcerons de fournir les outils nécessaires à l'écriture de code fonctionnel et par
conséquent, nous nous en tiendrons à l'utilisation du système de typage standard utilisé au
sein du monde fonctionnel.

## Récit d'un mystérieux monde

Des pages poussiéreuses d'un livre de Maths au vaste océan de livres blancs en passant par les
articles de blogs fortuits et même jusqu'au profondeur d'un code source, les signatures de
types de Hindley-Milner sont partout. Le système est en lui-même assez simple mais mérite
néanmoins quelques explications et un peu de pratique pour en maitriser toute l'essence. 

```js
//  capitalize :: String -> String
var capitalize = function(s){
  return toUpperCase(head(s)) + toLowerCase(tail(s));
}

capitalize("smurf");
//=> "Smurf"
```

Ici, `capitalize` prend une `String` et retourne une `String`. Ne regardez pas
l'implémentation, c'est la signature de type qui nous intéresse ici. 

Au sein de HM, les fonctions sont écrites comme `a -> b` avec `a` et `b` étant des variables de
n'importe quel type. Ainsi, la signature pour `capitalize` peut se lire "une fonction de
`String` vers `String`. En d'autres termes, elle prend une `String` en entrée, et retourne une
`String` en sortie. 

Regardons de plus près quelques autres signatures:

```js
//  strLength :: String -> Number
var strLength = function(s){
  return s.length;
}

//  join :: String -> [String] -> String
var join = curry(function(what, xs){
  return xs.join(what);
});

//  match :: Regex -> String -> [String]
var match = curry(function(reg, s){
  return s.match(reg);
});

//  replace :: Regex -> String -> String -> String
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

`strLength` est similaire à ce que nous avons vu précedemment: `String` vers `Number`. 

Les autres vous ont peut-être laissé un tantinet perplexe. Sans saisir les détails dans leur
globalité, vous pouvez déjà considérer que le dernier type représente le type de retour de la
fonction. De fait, `match` peut s'interpréter comme: prend une `Regex` et une `String` et
retourne une `[String]`. Il y a néanmoins une petite subtilité que j'aimerais détailler un
petit peu avec vous. 

Dans le cas de `match`, il est tout a fait légitime de grouper la signature de la sorte:

```js
//  match :: Regex -> (String -> [String])
var match = curry(function(reg, s){
  return s.match(reg);
});
```

Intéressant ! Ce groupement nous révèle maintenant des informations supplémentaires sur la
fonction. On peut voir la fonction comme une fonction qui prend une `Regex` et nous retourne
une fonction d'une `String` vers une `[String]`. C'est effectivement le cas car la fonction est
curryfiée: donnez lui une `Regex` et elle recrachera une nouvelle fonction prête à accueillir
sa `String` en argument. Bien entendu, il n'est pas nécessaire de voir les choses comme cela,
mais c'est une bonne intuition que de considérer le dernier type comme le type de retour.

```js
//  match :: Regex -> (String -> [String])

//  onHoliday :: String -> [String]
var onHoliday = match(/holiday/ig);
```

Chaque argument éjecte un type du début de la signature. `onHoliday` est `match` qui est
possède déjà une `Regex`.

```js
//  replace :: Regex -> (String -> (String -> String))
var replace = curry(function(reg, sub, s){
  return s.replace(reg, sub);
});
```

Comme vous pouvez le constater lorsqu'on indique explicitement toutes les parenthèses de la
signature de `replace`, la notation devient un petit peu floue et redondante. C'est pourquoi
nous les omettons tout simplement. On peut très bien donner tous les arguments en une fois
auquel cas il est plus simple de voir la fonction comme: `replace` prend une `Regex`, une
`String` et une autre `String` puis retourne une `String`.

Encore un petit quelque chose ici :

```js
//  id :: a -> a
var id = function(x){ return x; }

//  map :: (a -> b) -> [a] -> [b]
var map = curry(function(f, xs){
  return xs.map(f);
});
```

La fonction `id` prend en argument n'importe quel type `a` et retourne quelque chose du même
type `a`. Il est possible d'utiliser des variables au sein des types, comme on le ferait au
sein du code. Des noms tels que `a` et `b` sont d'usage courants mais peuvent être
arbitrairement remplacé par ce que qui vous semblera opportun. Une règle importante toutefois:
si les variables sont les mêmes alors il doit s'agir du même type. Ainsi `a-> b` traduit une
fonction de n'importe quel type `a` vers n'importe quel type `b` alors que, `a -> a` signifie
qu'il s'agit du même type, en entrée comme en sortie. Par exemple, `id` peut être
`String -> String` ou `Number -> Number` mais certainement pas `String -> Bool`.

`map` utilise également des variables de type, avec cette fois-ci l'introduction de `b` qui
peut être ou ne pas être du même type que `a`. On peut lire cette signature comme: `map` prend
une fonction de n'importe quel type `a` vers le même type ou un type différent `b`, puis prend
une liste de `a`, et retourne une liste de `b`.

Vous voilà à présent subjugué par la beauté de l'expressivité de cette signature de type. Elle
décrit littéralement ce que fait la fonction presque mot pour mot. Elle reçoit une fonction de
`a` dans `b`, une liste de `a` et rend une liste de `b`. De toute évidence, la seule chose que
`map` peut bien faire est d'appeler cette fichue fonction sur chaque élément `a`. Tout autre
comportement serait complètement incongru. 

Être capable de raisonner à propos des types et de leur impact est un savoir-faire qui vous
mènvera loin dans le monde fonctionnel. Non seulement les articles, documentations et papiers
scientifiques deviendront plus digeste, mais la signature elle-même vous donnera quasiment les
fonctionnalités réalisées. Devenir un lecteur assuré vous demandera du temps, mais si vous
prenez la peine de vous y appliquer, c'est un vaste monde de ressources qui vous deviendra
accessible. 

En voici quelques autres histoire que vous les décryptiez par vous-même.

```js
//  head :: [a] -> a
var head = function(xs){ return xs[0]; }

//  filter :: (a -> Bool) -> [a] -> [a]
var filter = curry(function(f, xs){
  return xs.filter(f);
});

//  reduce :: (b -> a -> b) -> b -> [a] -> b
var reduce = curry(function(f, x, xs){
  return xs.reduce(f, x);
});
```
`reduce` est sans doute la plus expressive de toutes. Elle est cependant complexe, ne vous
sentez pas désemparé si elle vous donne du fil à retordre. Pour les plus curieux, je vais
tâcher de l'expliquer en Français bien que faire travailler votre esprit sur la signature vous
sera bien plus bénéfique. 

Bien... un coup d'oeil à la signature nous dit que la fontion attend en premier argument une
fonction prenant un `b`, un `a` et produisant un `b`. D'où peuvent bien provenir ces `a` et `b`
? Et bien, la suite de la signature fait référence à un `b` et une liste de `a`; on en déduit
donc qu'ils serviront sans doute d'entrées à la première fonction. On voit aussi que le type de
sortie est `b`, exactement comme celui de la fonction passée en paramètre. Sachant ce que fait
en réalité la fonction `reduce`, l'explication précédente est relativement légitime. 

## Restreindre les possibilités

Lorsqu'on introduit les variables de type, on introduit également une curieuse propriété
appelée *paramétricité*(http://en.wikipedia.org/wiki/Parametricity). Cette dernière stipule que
la fonction concernée *doit se comportement de façon cohérente et uniforme pour n'importe quel
type*. Investiguons:

```js
// head :: [a] -> a
```

Regardez `head`, elle prend `[a]` et retourne `a`. Derrière le type concrèt de liste, il n'y a
aucune autre information disponible et de fait, sa portée se limite à la manipulation de la
liste en question. Que serait-il possible de faire avec une variable de type `a` si l'on ne
sait absolument rien de sa nature ? En d'autres termes, `a` stipule qu'il ne peux pas s'agir
d'un type *spécifique* mais qu'il peut néanmoins s'agir de *n'importe quel* type. Il est
résulte une fonction qui doit traiter de façon uniforme *chaque* type imaginable. C'est là le
coeur de la *paramétricité*. Si l'on se penche sur l'implémentation ici, la seule hypothèse
valable est que notre fonction doit sans doute récupérer le premier, le dernier ou un élément
quelconque de la liste. Le nom `head` nous met toutefois sur une bonne piste.

Voici une autre fonction:

```js
// reverse :: [a] -> [a]
```

En s'appuyant à nouveau sur la signature, qu'est-il possible de déduire ? Encore une fois, il
ne s'agit pas d'un traitement spécifique à un type `a`. La fonction ne peut pas changer `a` en
un type différent - auquel cas l'on aurait introduit un `b`. Peut-elle ordonner ? Et bien, pas
vraiment. Elle manquerait d'information essentiel pour ce faire. Peut-elle réorganiser ? Oui,
on peut le supposer à condition qu'elle le fasse d'une façon hautement prédictible.
Éventuellement, elle peut aussi retirer ou dupliquer des éléments. Dans tous les cas, ce qu'il
faut percevoir, c'est que les possibles comportements sont contraints par le polymorphisme
qu'impose le type.

La restriction des possibilités nous permet d'utiliser des moteurs de recherches dédiés aux
signatures comme [Hoogle](https://www.haskell.org/hoogle) afin de rechercher ce qui nous
intéresse. Le condensé d'informations offert par une signature est tout à fait brillant.

## Des théorèmes à la pelle

Au delà de simplement déduire des possibles implémentations, ce genre de raisonnement nous
offrent des *théorèmes implicits*.  Ce qui suit constituent deux examples choisis au hasard et
issues de [l'essai de Philip Wadler à ce sujet](http://ttic.uchicago.edu/~dreyer/course/papers/wadler.pdf).

```js
// head :: [a] -> a
compose(f, head) == compose(head, map(f));

// filter :: (a -> Bool) -> [a] -> [a]
compose(map(f), filter(compose(p, f))) == compose(filter(p), map(f));
```

Il n'y a pas besoin de coder quoi que ce soit pour obtenir ces théorèmes, ils se déduisent des
types directement. Le premier stipule que si l'on récupère l'élément de tête de notre liste et
qu'on lui applique une fonction `f`, c'est tout autant équivalent à, et soit dit en passant
bien plus rapide que, appliquer `f` via `map` sur à chaque élément puis de prendre l'élément de
tête de la liste résultante.

Sans doute vous dites vous qu'il ne s'agit que de bon sens. Néanmoins, pour autant que je
sache, les ordinateurs n'ont pas de notion du bon sens. De fait, ils ont besoin d'une façon
formelle d'automatiser ce genre d'optimisation. Fort heureusement les Mathématiques possèdent
de nombreuses façon de formaliser ces intuitions ce qui se révèle particulièrement utile à la
logique un tant soit peu rigide de nos machines.

Le théorème concernant `filter` est similaire. Il indique que composer `f` et `p` pour chercher
quels éléments doivent être filtrés, puis appliquer `f` via `map` (rappelez-vous que filter ne
transform pas les éléments - la signature assure que `a` restera inchangé), sera toujours
équivalent à mapper notre fonction `f` puis à filtrer le ŕesultat à l'aide d'un prédicat `p`.

Ces deux théorèmes sont deux exemples parmi tant d'autres. Vous pouvez toutefois appliquer ce
raisonnement à n'importe quelle signature polymorphique et il demeure valable. En JavaScript,
il existe quelques outils qui permettent de d'obtenir ces règles. On peut aussi simplement
arriver à ces résultats via les propriétés de la fonction `compose` elles-mêmes. L'effort à
faire n'est pas excessif et les bénéfices sont sans limite. 

## Constraints

Une dernière petite chose à remarquer: on peut contraindre les types à un certaine interface.

```js
// sort :: Ord a => [a] -> [a]
```

Ce que l'on apperçoit du côté gauche de notre grosse flèche est une déclaration de nature: `a`
doit être un `Ord`. En d'autres termes, `a` doit implémenter l'interface `Ord`. Qu'est-ce qu'un
`Ord` et d'où sort-il ? Dans un langage typé, elle représenterait une interface impliquant que
des éléments puissent être ordonnés. Non seulement nous donne-t'elle plus d'information sur
notre type `a` et ce à quoi la fonction `sort` se prédestine, mais aussi restreint-elle leur
domaine. On appelle ce genre de déclarations d'interfaces des *contraintes de type*.

```js
// assertEqual :: (Eq a, Show a) => a -> a -> Assertion
```

Dans cet exemple, il y a deux contraintes: `Eq` et `Show`. La première nous assure que l'on
peut tester une égalité de deux `a` alors que la seconde assure qu'il est possible d'imprimer
la différence en cas de non-égalité.

Nous verrons plus d'exemples de contraintes et vous vous ferez davantage à l'idée dans les
chapitres à venir.

## En bref

Les signatures de types d'Hindley-Milner sont omniprésentes dans le monde fonctionnel. Bien
qu'elles soient simple à lire et à écrire, il faut de la pratique pour déduire de leur
seule interprétation le fonctionnement d'un programme. À partir de maintenant, nous les
ajouterons à chaque ligne de code que nous écrirons.

[Chapitre 8: Tupperware](ch8.md)

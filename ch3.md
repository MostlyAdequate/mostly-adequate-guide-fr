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

## Plaidoyer en en faveur de la pureté

### Mise en cache

En entrée de matière, soulignons que les fonctions pures peuvent mettre en cache un résultat
associé à une certaine valeur. Cela se réalise typiquement grace à une technique appelée
*memoization*.

```js
var squareNumber  = memoize(function(x){ return x*x; });

squareNumber(4);
//=> 16

squareNumber(4); // retourne la valeur cachée pour l'entrée 4
//=> 16

squareNumber(5);
//=> 25

squareNumber(5); // retourne la valeur cachée pour l'entrée 4
//=> 25
```

Bien qu'il existe de nombreuses autres implémentation plus robuste, en voici une relativement
modeste :

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
Un autre point intéressant à souligner est qu'il est possible de transformer des fonctions
impures en fonctions pures en retardant le moment de leur évaluation :

```js
var pureHttpCall = memoize(function(url, params){
  return function() { return $.getJSON(url, params); }
});
```

Ce qui est intéressant ici c'est que nous ne faisons pas réellement l'appel http - nous
retournons à la place une fonction qui sera à-même de le faire une fois appelée. La fonction
englobante est pure car elle retournera toujours la même fonction d'exécution pour une même
entrée : c'est cette fonction retournée qui s'occupera de realiser concrètement l'appel http
correspondant à un `url` et `params` donnés. 

Notre fonction `memoize` est tout à fair correcte et de fait, ne met pas en cache le résultat
de l'appel http, mais plutôt la fonction générée. 

C'est pour l'instant fort peu utile mais nous verrons bientôt quelques astuces de grand-mère
qui changeront cela. Ce qu'il faut en retenir, c'est qu'il nous est possible de mettre en cache
n'importe quelle fonction, peu importe leur niveau d'impureté. 

### Portatives et auto-documentées

Les fonctions pures sont totalement autonomes. Tout ce dont la fonction a besoin lui est servi
sur un plateau. Arrêtons-nous un instant... En quoi cela est-il favorable ? Premièrement, les
dépendances d'une fonction sont explicites et par conséquent simple à voir et comprendre -
nullement besoin de regarder la machinerie sous le capot. 

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

L'exemple ci-dessus illutre en quoi une fonction pure se doit d'être honnête au sujet de ses
dépendances de telle façon qu'elles nous apparaissent clairement. Au regard seulement de la
signature, on sait que l'on aura besoin d'une `Db`, d'un `Email` et d'`attrs`.

Nous verrons comment rendre des fonctions de la sorte pure sans pour autant grandemment
retarder leur évaluation; gardez néanmoins bien à l'esprit que la forme pure est bien plus
claire et parlante que son insidieuse homologue impure.  

Force est de constater qu'il nous faut également sinon "injecter" les dépendances du moins les
passer en paramètres ce qui rend notre application nettement plus flexible : nous avons rendu
paramétrables notre base de données et le client mail[^Pas d'inquiétude, je vous montrerez
comment rendre cela moins fastidieux qu'il n'y paraît]. Qu'il s'agisse de facilement changer
notre système de base de données pour un autre ou encore de réutiliser cette fonction dans une
application différente, cette façon de faire y répond sans problème.

Dans le monde du JavaScript, la portabilité peut ne signifier rien de plus que serialiser et
envoyer nos fonctions à travers des sockets. Autrement dit, faire tourner tout le code d'une
application dans des workers web. La portabilité est un atout puissant.

Contrairement aux méthodes et procédures "typiques" en progrmmation fonctionnelle profondément
ancrées dans leur environnement d'exécution via des états, dépendances et divers effets, les
fonctions pures peuvent s'exécuter partout où le vent nous porte.

A quand remonte la dernière fois que vous ayez copié une méthode d'une application à une autre
? L'une de mes citations favorites de l'inventeur d'Erlang, Joe Armstrong est celle-ci: "Le
problème avec les langages orienté objets c'est qu'il transporte tout un environnement
implicite avec eux. Vous vouliez une banane mais vous obtenez un gorille qui tient cette
banane... ainsi que tout la jungle". 

### Testable

En outre, on en vient à réaliser que les fonctions pures rendent le test largement plus facile.
Pas besoin de simuler une plateforme de paiement et de présumer de l'état du monde entier après
chaque test. On donne juste à la fonction des entrées, et on vérifie sa sortie.

En réalité, de nombreux pionniers de la communauté mettent au point des outils complexes
capables de générer des entrées et d'inférer des propriétés que doivent posséder la sortie.
Tout ceci est bien au delà de la portée de ce livre mais je vous encourage fortement à
rechercher et essayer *Quickcheck* - un outil de test taillé pour les environnement purement
fonctionnels.

### Raisonnable

Beaucoup pensent que l'un des plus gros points forts des fonctions pures est la *transparence
référentielle*. Un bout de code qui est référentiellement transparent peut se substituer à sa
valeur de sortie sans avoir aucun impact sur le comportement du programme.

Comme les fonctions pures retournent toujours la même sortie selon une entrée donnée, on peut
compter sur elles pour obtenir un resultat cohérent qui préserve la transparence référentielle.
Voyez plutôt:

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

`decrementHP`, `isSameTeam` et `punch` sont toutes trois pures et de fait référentiellement
transparentes. On peut utiliser une technique appelée *raisonnement équationnel* afin de
substituer des parties équivalentes du code et ainsi raisonner plus facilement sur ce dernier.
C'est comme procéder à une évaluation manuelle du code sans considérer les fioritures
contingentes d'une analyse programmatique. En utilisant la transparence référentielle, jouons
un peu avec le code précédent.

Tout d'abord, développons l'appel a la fonction `isSameTeam`.

```js
var punch = function(player, target) {
  if(player.get("team") === target.get("team")) {
    return target;
  } else {
    return decrementHP(target);
  }
};
```

Notre structure de données étant immutable, nous pouvons remplacer les équipes par leur valeur
courante.

```js
var punch = function(player, target) {
  if("red" === "green") {
    return target;
  } else {
    return decrementHP(target);
  }
};
```
En évaluant la condition, on se rend compte que la branche `if` est inutile.

```js
var punch = function(player, target) {
  return decrementHP(target);
};

```

En développant `decrementHP`, on fait apparaître que dans ce cas précis, un coup de point n'est
seulement qu'un appel visant à décrémenter les `hp` de 1.

```js
var punch = function(player, target) {
  return target.set("hp", target.get("hp")-1);
};
```

Cette aptitude à raisonner sur le code est redoutablement efficace pour refactorer et
comprendre le code en général. En fait nous avons déjà utiliser cette technique pour revoir
notre programme avec les mouettes. Nous avons utilisé un raisonnement équationnel afin de tirer
partie des propriétés de l'addition et du produit. Ainsi, nous serons amené à utiliser
davantage ces techniques tout au long du livre.

### Parallel Code

Finally, and here's the coup de grâce, we can run any pure function in parallel since it does not need access to shared memory and it cannot, by definition, have a race condition due to some side effect.

This is very much possible in a server side js environment with threads as well as in the browser with web workers though current culture seems to avoid it due to complexity when dealing with impure functions.


## In Summary

We've seen what pure functions are and why we, as functional programmers, believe they are the cat's evening wear. From this point on, we'll strive to write all our functions in a pure way. We'll require some extra tools to help us do so, but in the meantime, we'll try to separate the impure functions from the rest of the pure code.

Writing programs with pure functions is a tad laborious without some extra tools in our belt. We have to juggle data by passing arguments all over the place, we're forbidden to use state, not to mention effects. How does one go about writing these masochistic programs? Let's acquire a new tool called curry.

[Chapter 4: Currying](ch4.md)

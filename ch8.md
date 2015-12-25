# Tupperware

## La boîte qui déboîte

<img src="images/jar.jpg" alt="http://blog.dwinegar.com/2011/06/another-jar.html" />

Nous avons vu comment écrire des programmes qui acheminent des données au travers de fonctions
pures. Ils consituent une forme déclarative des spécifications de comportements attendus. Qu'en
est-il de la gestion des erreurs, du contrôle de l'exécution, des actions asynchrones, des états
et, allons jusque là, des effets ?! Dans ce chapitre, nous bâtirons les fondations au dessus
desquelles sont construites toutes ces abstractions importantes. 

En premier lieu, nous allons créer un contenant. Il doit pouvoir contenir n'importe quel type
de valeur; Il y a fort à parier qu'un bocal qui ne peut contenir que du pâté serve énormément.
Il s'agira d'un objet bien que nous nous refuserons à lui affecter des propriétés et des
méthodes au sens de l'orientée-objet. Non, nous le traiterons comme un coffre au trésor - une
boîte singulière qui renferme notre bien-aimée donnée. 

```js
var Container = function(x) {
  this.__value = x;
}

Container.of = function(x) { return new Container(x); };
```

Voilà notre premier contenant. Nous l'avons en toute connaissance de causes nommé `Container`.
Nous utiliserons `Container.of` comme constructeur en lieu et place de l'horrible mot-clé
`new`. Il y a plus à raconter sur cette fonction `of` qu'il n'y paraît, pour le moment
ceci-dit, elle n'est qu'une façon propre de placer une valeur au sein du contenant.

Jouons un peu avec notre nouvelle boîte flambant neuve...

```js
Container.of(3)
//=> Container(3)


Container.of("hotdogs")
//=> Container("hotdogs")


Container.of(Container.of({name: "yoda"}))
//=> Container(Container({name: "yoda" }))
```

Si vous utilisez *node*, vous verrez `{__value: x}` bien qu'il s'agissent d'un `Container(x)`.
Chrome affichera le type correctement mais peu importe tant que nous comprenons ce à quoi
ressemble un `Container`. Au sein de certain environnement, vous avez la possiblité de réecrire
la methode `inspect` mais nous n'irons pas jusque là. Pour les besoins du livre ceci dit, nous
présenterons la valeur de sortie théorique comme si nous avions modifié la méthode `inspect` en
conséquence; ce sera au moins plus intuitif que `{__value: x}` sinon à la fois plus esthétique
et pédagogique.

Clarifions quelques points avant de creuser encore le sujet:

* `Container` est un objet avec une unique propriété. De nombreux contenants ne contiennent
  qu'une seule chose mais n'en faites pas un état de fait. Nous avons de plus complétement
  arbitrairement appelé cette propriété `__value`.

* La valeur `__value` ne peut être restreinte à un type spécifique. Le cas échéant, nous
  aurions défini un bien piètre contenant.

* Une fois capturée par le `Container`, la donnée y reste. Il serait *hypothétiquement
  envisageable* d'y accéder via `__value` mais ce serait violer l'essence même du contenant.

Les motivations qui nous animent ici deviendront bientôt claires comme de l'eau de roche. Pour
l'heure je vous demande de me faire confiance.

## Mon premier foncteur

Quelque soit notre valeur, une fois dans son contenant, il nous faut lui appliquer des
fonctions.

```js
// (a -> b) -> Container a -> Container b
Container.prototype.map = function(f){
  return Container.of(f(this.__value))
}
```

Grandemment inspirée de la fameuse méthode `map` propre aux listes, cette méthode fonctionne de
façon similaire mais s'applique cependant à un `Container a` plutôt qu'à un `[a]`.

```js
Container.of(2).map(function(two){ return two + 2 })
//=> Container(4)


Container.of("flamethrowers").map(function(s){ return s.toUpperCase() })
//=> Container("FLAMETHROWERS")


Container.of("bombs").map(_.concat(' away')).map(_.prop('length'))
//=> Container(10)
```

Il est donc possible de travailler avec notre valeur sans jamais avoir à quitter le contenant.
C'est tout à fait prodigieux. La valeur au sein du contenant est simplement transmise à la
fonction `map` de telle sorte que l'on puisse la manipuler à notre guise avant de la replacer
au chaud dans sa boîte. Conserver la valeur au sein de son contenant a plusieurs avantages.
Entre autre, il nous est possible de continuer à appliquer des fonctions à l'aide de `map`
autant que nous le voulons. Il n'y a aucun problème non plus à changer le type de la valeur à
l'intérieur du contenant comme l'ont montré les trois exemples précédents. 

Mais au fait, si l'on appelle `map`, c'est que quelque part sous le capot on fait appel à de la
composition ! Quelle sorcellerie mathématiques est à l'oeuvre ici ? Bravo, nous venons de
découvrir les *Foncteurs*.

> Un Foncteur est un type qui implémente `map` et obéit à quelques lois

Oui, un *Foncteur* n'est rien de plus qu'une interface avec un contrat. On pourrait tout aussi
bien l'avoir appelé `Mappable` mais *Foncteur* rime avec *bonheur*, n'est-ce pas ? Les
Foncteurs proviennent tout droit de la théorie des catégories et bien entendu, nous aborderons
les détails techniques mathématiques à ce sujet d'ici la fin du chapitre. Pour l'instant,
tâchons d'en développer une intuition pratique et de comprendre ce qu'il se cache derrière
cette interface un tant soit peu étrange.

Quelle raison saugrenue peut bien nous pousser à encapsuler une valeur de la sorte pour ensuite
intéragir avec elle via `map` ? La réponse est dans la question pour peu qu'on prenne la peine
de la reformuler: Que gagnons-nous à demander au contenant d'appliquer les fonctions à notre
place ? Eh bien, c'est un niveau d'abstraction supplémentaire dans l'application de fonctions.
Il nous suffit d'appliquer une fonction et le contenant se charge de l'appliquer pour nous en
prenant soin de gérer le type. C'est un concept puissant, le voyez vous ? 

## Schrödinger, une histoire de Maybe

<img src="images/cat.png" alt="cool cat, need reference" />

`Container` est relativement morne. En réalité, on le présente d'ordinaire comme `Identity` et
agit de façon similaire à la fonction `id` (au risque de me répéter, sachez qu'il y a également
un lien mathématique entre ces deux entités, lien que nous expliciterons en temps voulus). Il
existe néanmoins bien d'autres foncteurs c'est à dire, des contenants typés qui possède leur
propre implémentation de la fonction `map` laquelle leur confère des propriétés intéressantes.
Voyons-en un nouveau dès à présent:

```js
var Maybe = function(x) {
  this.__value = x;
}

Maybe.of = function(x) {
  return new Maybe(x);
}

Maybe.prototype.isNothing = function() {
  return (this.__value === null || this.__value === undefined);
}

Maybe.prototype.map = function(f) {
  return this.isNothing() ? Maybe.of(null) : Maybe.of(f(this.__value));
}
```

Now, `Maybe` looks a lot like `Container` with one minor change: it will first check to see if it has a value before calling the supplied function. This has the effect of side stepping those pesky nulls as we `map`(Note that this implementation is simplied for teaching).

```js
Maybe.of("Malkovich Malkovich").map(match(/a/ig));
//=> Maybe(['a', 'a'])

Maybe.of(null).map(match(/a/ig));
//=> Maybe(null)

Maybe.of({name: "Boris"}).map(_.prop("age")).map(add(10));
//=> Maybe(null)

Maybe.of({name: "Dinah", age: 14}).map(_.prop("age")).map(add(10));
//=> Maybe(24)
```

Vous remarquerez que notre application ne s'effondre pas dès lors que l'on applique nos
fonctions sur des valeurs nulles via `map`. C'est en effet le rôle du `Maybe` de vérifier la
présence ou non d'une valeur au sein du contenant avant d'y appliquer la fonction passée en
argument. 

En outre, cette syntaxe n'est pas des plus fines et fonctionnelles; par conséquent et en accord
avec ce qui fut mentionné en première partie de ce livre, il serait bon d'encourager une
écriture *pointfree*. Comme par hasard, `map` est parfaitement équipée pour pallier ce
contretemps et déléguer le travail au foncteur qu'elle reçoit:

```js
//  map :: Functor f => (a -> b) -> f a -> f b
var map = curry(function(f, any_functor_at_all) {
  return any_functor_at_all.map(f);
});
```

C'est fantastque car nous pouvons désormais à nouveau procéder à des compositions et `map`
fonctionnera comme prévue. C'est par ailleurs aussi le cas avec la méthode `map` fournie par
ramda. Nous utiliserons la notation "classique" avec point lorsqu'elle est forte de sens, et la
notation *pointfree* le reste du temps car plus pratique. Au fait, avez-vous remarqué
quelque chose de singulier ? J'ai introduit en douce une petite notation dans la signature de
type. La partie `Functor f =>` indique que `f` désigne une foncteur dans le reste de la
signature. Intuitif mais une petite explication ne fait pas de mal. 

## Cas d'utilisation

Dans la nature, on utilise essentiellement `Maybe` avec des fonctions dont l'issue n'est pas a
priori garantie. 

```js
//  safeHead :: [a] -> Maybe(a)
var safeHead = function(xs) {
  return Maybe.of(xs[0]);
};

var streetName = compose(map(_.prop('street')), safeHead, _.prop('addresses'));

streetName({addresses: []});
// Maybe(null)

streetName({addresses: [{street: "Shady Ln.", number: 4201}]});
// Maybe("Shady Ln.")
```

`safeHead` agit comme notre classique `_.head`, sauf que la valeur de retour est encapsulée
dans un type plus sûr. L'introduction du `Maybe` dans notre code fait curieusement apparaître
quelque chose; il nous faut désormais traiter les valeurs nulles pourvant surgir inopinément.
La fonction `safeHead` est transparente et nous affiche clairement sa propension à l'erreur en
retournant un `Maybe`. D'ailleurs, non seulement nous informe-t-elle d'un retour
potentiellement erroné mais aussi nous oblige-t-elle à utiliser `map` pour accéder à la valeur
retournée emprisonnée dans son contenant. Intuitivement, il s'agit d'une vérification de la
présence de la valeur dictée par la fonction `safeHead` elle-même. On peut ainsi dormir sur nos
deux oreilles sachant qu'il ne risque pas de surgir de valeur nulle comme par magie. Ce genre
de méthodes peut significativement améliorer une application faite de bric et de broc en
quelque chose de nettement plus robuste. Par ce procédé, on garantit un programme plus sûr.

Parfois une fonction retournera un `Maybe(null)` afin de signaler explicitement un cas
d'erreur. Par exemple:

```js
//  withdraw :: Number -> Account -> Maybe(Account)
var withdraw = curry(function(amount, account) {
  return account.balance >= amount ?
    Maybe.of({balance: account.balance - amount}) :  
     Maybe.of(null);
});

//  finishTransaction :: Account -> String
var finishTransaction = compose(remainingBalance, updateLedger);  // <- these composed functions are hypothetical, not implemented here...

//  getTwenty :: Account -> Maybe(String)
var getTwenty = compose(map(finishTransaction), withdraw(20));



getTwenty({ balance: 200.00});
// Maybe("Your balance is $180.00")

getTwenty({ balance: 10.00});
// Maybe(null)
```

Si nous somme à court d'argent, `withdraw` nous le fera savoir en remontant un `Maybe(null)`.
Elle le fera de telle sorte que son caprice sera entendu jusqu'en bout de chaîne en nous
obligeant à appliquer `map` pour chaque fonction en suivant. À la fin, nous rendons
effectivement compte de l'erreur et c'est notre application qui s'arrête correctement. S'il y a
bien quelque chose à comprendre de cet exemple, c'est que si `withdraw` échoue, alors les
appels à `map` servent de bouclier et empêche l'exécution de tout autre fonction (comme
`finishTransaction`). C'est précisément le comportement que nous espérons. Mettre à jour le
solde de notre compte alors que nous possédons des fonds insuffisants n'est pas du meilleur
effet. 

## Libérez le Kraken

Ce que les gens oublient parfois c'est qu'à un moment donné, on atteint le bout de la chaîne;
des fonctions effectives qui transmettent du JSON, affichent à l'écran ou encore altère le
système de fichiers. À ce moment précis, il nous est impossible de retourner un résultat, il
nous faut exécuter des fonctions qui pourront communiquer avec le monde extérieur. Tel un sage
Buddhiste, nous pouvons le formuler comme le Kôan Zen suivant: "Si un programme ne possède
aucun effet observable, s'exécute-t-il seulement ? ". S'exécute-t-il pour selon son propre
désir ? Sans effet observable, il consomme au mieux quelques cycles CPU avant de retourner au
repos. 

Le rôle de notre application est de récupérer, transformer et d'amener un ensemble de données
jusqu'à ce qu'il soit l'heure de leur dire au revoir. Ce faisant, nous appliquons
successivement des fonctions grâce à `map` de telle sorte que les données n'ont guère besoin de
quitter leur chaleureux contenant. Une erreur assez commune consiste à essayer de retirer la
valeur du 'Maybe' d'une façon ou d'une autre en espérant que celle-ci se matérialise comme par
enchantement. Il faut bien comprendre qu'il existe potentiellement une exécution du programme
dans laquelle la valeur n'ira pas vivre sa destinée. Notre code est comme le chat de
Schrödinger, simultanément dans deux états et nous nous devons de maintenir cette distinction
jusqu'à l'exécution de la fonction finale. De cette façon le code demeure linéaire et l'on
s'évite des branchements logiques.

Il existe toutefois une échappatoire. Si l'on souhaite effectivement retourner une valeur
particulière et continuer, on peut le faire grâce à la petite fonction suivante:

```js
//  maybe :: b -> (a -> b) -> Maybe a -> b
var maybe = curry(function(x, f, m) {
  return m.isNothing() ? x : f(m.__value);
});

//  getTwenty :: Account -> String
var getTwenty = compose(
  maybe("You're broke!", finishTransaction), withdraw(20)
);


getTwenty({ balance: 200.00});
// "Your balance is $180.00"

getTwenty({ balance: 10.00});
// "You're broke!"
```

Dorénavant, nous retournons ou bien une valeur statique (néanmoins du même type que celle
retournée par `finishTransaction`) ou bien, nous menons la transaction à bien cette fois-ci
sans `Maybe`. Avec `Maybe`, nous reflétons un branchement équivalent à un `if/else` pour lequel
`map` correspond de façon moins impérative à: `if (x !== null) { return f(x) }`.

L'introduction de `Maybe` peut être dure à appréhender de premier abord. Les utilisateurs de
Swift et Scala savent bien de quoi je parle vu que le même concept est présent via les
bibliothèques native au travers des `Option(al)`. Lorsque l'on en vient à devoir traiter un
ensemble de vérification à `null` à la suite (même lorsque l'on sait parfois que la valeur ne
peut simplement pas être nulle), la plupart des gens n'ont aucune solution mais ressentent
toutefois bien la pénibilité de l'écriture d'un tel code. Avec `Maybe`, vous prendez vite
l'habitude à tel point que c'en deviendra une seconde nature. Après tout, la plupart du temps
ce sera un bien pour un moindre mal.

Développer un programme non fiable, c'est tout aussi stupide que de prendre le temps de décorer
des oeufs avec de la peinture avant de les jetez au milieu de la route. C'est une bonne chose
que de vouloir apporter un peu de robustesse à nos fonctions et `Maybe` est la pour ça.

Je manquerais à mes responsabilité si je ne vous précisais pas que la "réelle" implémentation
de `Maybe` séparera effectivement la valeur selon deux types: un pour ladite valeur, et l'autre
pour l'absence de cette valeur. Ceci nous offre une plus grande souplesse dans l'application
dans l'utilisation de `map` en permettant à des valeurs nulles comme `null` ou `undefined`
d'être également *mappées*. Ainsi, la dénomination universelle de foncteur est respectée. Vous
rencontrerez plus fréquemment des types tels que `Some(x) / None` ou `Just(x) / Nothing` plutôt
qu'un `Maybe` qui effectue une bête comparaison à `null`.

## Gestion d'erreur pure

<img src="images/fists.jpg" alt="pick a hand... need a reference" />

Ce peut être surprenant mais, `throw/catch` n'est pas vraiment pur. Lorsqu'une erreur surgit,
nous tirons la sonnette d'alarme plutôt que de retourner une valeur de sortie ! La fonction
agresse d'une millier de 1 et de 0 notre application en réponse à l'entrée qu'on lui a fourni.
Avec l'aide de notre nouvel ami `Either`, nous pouvons faire mieux que de déclarer la guerre à
nos entrées. Nous pouvons répondre un message à la fois poli et plus adéquat. Regardons cela de
plus près.

```js
var Left = function(x) {
  this.__value = x;
}

Left.of = function(x) {
  return new Left(x);
}

Left.prototype.map = function(f) {
  return this;
}

var Right = function(x) {
  this.__value = x;
}

Right.of = function(x) {
  return new Right(x);
}

Right.prototype.map = function(f) {
  return Right.of(f(this.__value));
}
```

`Left` et `Right` sont deux sous-classes d'un type plus abstrait nommé `Either`. Je vous
épargne la création de la classe mère `Either` étant donné que nous n'allons que peu l'utiliser
en tant que telle; n'hésitez pas à la regarder pour vous faire une idée cela dit. Ceci étant
dit, vous constaterez qu'il n'y a hormis les types que peu de nouveautés. Jetons-y un oeil:

```js
Right.of("rain").map(function(str){ return "b"+str; });
// Right("brain")

Left.of("rain").map(function(str){ return "b"+str; });
// Left("rain")

Right.of({host: 'localhost', port: 80}).map(_.prop('host'));
// Right('localhost')

Left.of("rolls eyes...").map(_.prop("host"));
// Left('rolls eyes...')
```
`Left` matérialise l'adolescent en crise et refuse toute communication et requête via `map`.
`Right` fonctionne de façon similaire à `Container` (a.k.a `Identity). L'intérêt ici vient de
la capacité de `Left` à stocker un message d'erreur. 

Faisons l'hypothèse d'une fonction qui a des chances d'échouer à l'exécution. Considérons le
calcul d'un âge depuis une date de naissance. On peut utiliser `Maybe(null)` afin de signaler
une erreur et aiguiller la suite de notre programme en conséquence mais l'information eset
maigre. En connaître davantage sur la nature de l'erreur, voilà ce qui nous intéresse. Voyons
ce que cela donne en utilisant `Either`.

```js
var moment = require('moment');

//  getAge :: Date -> User -> Either(String, Number)
var getAge = curry(function(now, user) {
  var birthdate = moment(user.birthdate, 'YYYY-MM-DD');
  if (!birthdate.isValid()) return Left.of("Birth date could not be parsed");
  return Right.of(now.diff(birthdate, 'years'));
});

getAge(moment(), {birthdate: '2005-12-12'});
// Right(9)

getAge(moment(), {birthdate: '20010704'});
// Left("Birth date could not be parsed")
```

De la même façon qu'avec `Maybe(null)` nous court-circuitons l'application en retournant un
`Left`. Une petite différence, forte de sens néanmoins, c'est que l'on peut maintenant
caractériser avec davantage de précision la nature de l'interruption. Ainsi, vous remarquerez
que l'on indique à présent dans la signature `Either(String, Number)` indiquant que le foncteur
renferme un `String` en partie gauche, et un `Number` en partie droite. Cette signature peut
sembler légérement informel vu que nous n'avons pas présenté la classe mère `Either`. C'est
pourtant assez transparent et l'on comprend aisément ce dont il s'agit. 

```js
//  fortune :: Number -> String
var fortune  = compose(concat("If you survive, you will be "), add(1));

//  zoltar :: User -> Either(String, _)
var zoltar = compose(map(console.log), map(fortune), getAge(moment()));

zoltar({birthdate: '2005-12-12'});
// "If you survive, you will be 10"
// Right(undefined)

zoltar({birthdate: 'balloons!'});
// Left("Birth date could not be parsed")
```

Lorsque `birthdate` est valide, le programme nous dévoile sa mystérieuse bonne fortune. Sinon,
nous récupérons un bon vieux message d'erreur cependant encore dans son contenant. D'une
certaine façon, c'est un peu comme avant lorsqu'une erreur était levée sauf qu'à présent elle
l'est calmement, poliement et sans excès là oú autrefois elle nous hurlait à la figure sans
aucun respect à la manière d'une enfant turbulent. 

Dans cet exemple, nous effectuons un branchement logique selon la potentielle validité de la
date de naissance. Il se lit toutefois d'un trait, linéairement sans avoir à visuellement
jongler entre les accolades d'une structure conditionnelle. On préférera usuellement déporter
l'appel à `console.log` en dehors de notre fonction `zoltar` pour l'appliquer via `map` au
niveau de chaque appel. C'est toutefois intéressant de rendre compte de la divergence de la
branche droite `Right`. Dans la signature de type de cette même branche, nous utilisons `\_`
pour désigner un type qui n'a aucune importance car ignoré. (Dans certains navigateurs, vous
aurez à utiliser `console.log.bind(console)` afin de l'utiliser en first-class). 

J'aimerai prendre quelques mots pour mettre l'accent sur quelque chose qui vous a probablement
échappé: La fonction `fortune` ici, bien qu'utilisée de paire avec `Either`, n'a aucune
connaissance a priori d'une quelconque association avec un foncteur. C'était aussi le cas avec
`finishTransaction` dans l'exemple précédent. On peut dire sans trop de formalisme qu'une
fonction classique peut être grâce à `map` transformée` en une fonction de foncteur. On dénomme
ce procédé `lifting`. Il est souvent plus simple pour des fonctions de travailler avec des
types normaux plutôt que des contenants, puis, d'être *liftées* vers un contenant adéquat. Il
s'en suit une plus grande simplicité et réutilisabilité dans le code.

En somme, `Either` est approprié pour la gestion des erreurs communes comme les résultats d'une
validation, tout autant qu'il l'est dans la gestion d'erreurs complexes qui conduisent
normalement à une interruption de l'exécution (un fichier manquant ou une socket inaccessible
par exemple). Essayez de remplacer quelques uns des `Maybe` précédents par `Either` pour en
améliorer le retour d'information. 

En outre, j'ai l'impression d'avoir présenté `Either` d'une façon un peu réductrice. Non
seulement nous sert-il à mieux gérer les erreurs, mais aussi représente-il une disjonction
logique  (a.k.a `||`) au sein d'un type. il matérialise également l'idée de *Coproduit* en
théorie des catégories (point que nous n'aborderons pas dans ce livre mais qu'il reste
intéressant de connaître ne serait-ce que pour explorer les propriétés à en exploiter). C'est
une représentation canonique de la somme (ou de l'union disjointe de deux ensembles) en tant
qu'union des quantités représentées par l'association des deux types contenus (je me doute que
ceci doit vous paraître obscure, n'hésitez pas à jetez un oeil à cet [excellent
article](https://www.fpcomplete.com/school/to-infinity-and-beyond/pick-of-the-week/sum-types)).
`Either` peut donc représenter de nombreuse chose mais en tant que foncteur, on l'associe
naturellement à la gestion d'erreur. 

Enfin, tout comme avec `Maybe`, nous considérons la petite `either` qui agit similairement mais
cette fois-ci à l'aide de deux fonctions et d'une valeur statique. Chaque fonction doit
bien-entendu avoir un même type de retour. 

```js
//  either :: (a -> c) -> (b -> c) -> Either a b -> c
var either = curry(function(f, g, e) {
  switch(e.constructor) {
    case Left: return f(e.__value);
    case Right: return g(e.__value);
  }
});

//  zoltar :: User -> _
var zoltar = compose(console.log, either(id, fortune), getAge(moment()));

zoltar({birthdate: '2005-12-12'});
// "If you survive, you will be 10"
// undefined

zoltar({birthdate: 'balloons!'});
// "Birth date could not be parsed"
// undefined
```

Finalement une utilisation de cette mystérieuse fonction `id`. Rien de plus qu'un simple
perroquet qui retransmet la valeur contenue dans `Left` afin d'être affichée via `console.log`.
En renforçant la gestion de nos erreurs depuis `getAge`, nous avons rendu notre diseuse de
bonne fortune plus robuste. Ou bien nous mettons l'utilisateur devant le fait accompli en lui
révélant la dure vérité liée à son erreur, ou bien nous exécutons la suite d'actions attendues.
À présent, nous voilà prêt à migrer vers une nouvelle famille de foncteurs.

## Old McDonald had Effects...

<img src="images/dominoes.jpg" alt="dominoes.. need a reference" />

In our chapter about purity we saw a peculiar example of a pure function. This function contained a side-effect, but we dubbed it pure by wrapping its action in another function. Here's another example of this:

```js
//  getFromStorage :: String -> (_ -> String)
var getFromStorage = function(key) {
  return function() {
    return localStorage[key];
  }
}
```

Had we not surrounded its guts in another function, `getFromStorage` would vary its output depending on external circumstance. With the sturdy wrapper in place, we will always get the same output per input: a function that, when called, will retrieve a particular item from `localStorage`. And just like that (maybe throw in a few Hail Mary's) we've cleared our conscience and all is forgiven.

Except, this isn't particularly useful now is it. Like a collectable action figure in its original packaging, we can't actually play with it. If only there were a way to reach inside of the container and get at its contents... Enter `IO`.

```js
var IO = function(f) {
  this.__value = f;
}

IO.of = function(x) {
  return new IO(function() {
    return x;
  });
}

IO.prototype.map = function(f) {
  return new IO(_.compose(f, this.__value));
}
```

`IO` differs from the previous functors in that the `__value` is always a function. We don't think of its `__value` as a function, however - that is an implementation detail and we best ignore it. What is happening is exactly what we saw with the `getFromStorage` example: `IO` delays the impure action by capturing it in a function wrapper. As such, we think of `IO` as containing the return value of the wrapped action and not the wrapper itself. This is apparent in the `of` function: we have an `IO(x)`, the `IO(function(){ return x })` is just necessary to avoid evaluation.

Let's see it in use:

```js
//  io_window :: IO Window
var io_window = new IO(function(){ return window; });

io_window.map(function(win){ return win.innerWidth });
// IO(1430)

io_window.map(_.prop('location')).map(_.prop('href')).map(split('/'));
// IO(["http:", "", "localhost:8000", "blog", "posts"])


//  $ :: String -> IO [DOM]
var $ = function(selector) {
  return new IO(function(){ return document.querySelectorAll(selector); });
}

$('#myDiv').map(head).map(function(div){ return div.innerHTML; });
// IO('I am some inner html')
```

Here, `io_window` is an actual `IO` that we can `map` over straight away, whereas `$` is a function that returns an `IO` after its called. I've written out the *conceptual* return values to better express the `IO`, though, in reality, it will always be `{ __value: [Function] }`. When we `map` over our `IO`, we stick that function at the end of a composition which, in turn, becomes the new `__value` and so on. Our mapped functions do not run, they get tacked on the end of a computation we're building up, function by function, like carefully placing dominoes that we don't dare tip over. The result is reminiscent of Gang of Four's command pattern or a queue.

Take a moment to channel your functor intuition. If we see past the implementation details, we should feel right at home mapping over any container no matter its quirks or idiosyncrasies. We have the functor laws, which we will explore toward the end of the chapter, to thank for this pseudo-psychic power. At any rate, we can finally play with impure values without sacrificing our precious purity.

Now, we've caged the beast, but we'll still have to set it free at some point. Mapping over our `IO` has built up a mighty impure computation and running it is surely going to disturb the peace. So where and when can we pull the trigger? Is it even possible to run our `IO` and still wear white at our wedding? The answer is yes, if we put the onus on the calling code. Our pure code, despite the nefarious plotting and scheming, maintains its innocence and it's the caller who gets burdened with the responsibility of actually running the effects. Let's see an example to make this concrete.

```js

////// Our pure library: lib/params.js ///////

//  url :: IO String
var url = new IO(function() { return window.location.href; });

//  toPairs =  String -> [[String]]
var toPairs = compose(map(split('=')), split('&'));

//  params :: String -> [[String]]
var params = compose(toPairs, last, split('?'));

//  findParam :: String -> IO Maybe [String]
var findParam = function(key) {
  return map(compose(Maybe.of, filter(compose(eq(key), head)), params), url);
};

////// Impure calling code: main.js ///////

// run it by calling __value()!
findParam("searchTerm").__value();
// Maybe([['searchTerm', 'wafflehouse']])
```

Our library keeps its hands clean by wrapping `url` in an `IO` and passing the buck to the caller. You might have also noticed that we have stacked our containers; it's perfectly reasonable to have a `IO(Maybe([x]))`, which is three functors deep(`Array` is most definitely a mappable container type) and exceptionally expressive.

There's something that's been bothering me and we should rectify it immediately: `IO`'s `__value` isn't really its contained value, nor is it a private property as the underscore prefix suggests. It is the pin in the grenade and it is meant to be pulled by a caller in the most public of ways. Let's rename this property to `unsafePerformIO` to remind our users of its volatility.

```js
var IO = function(f) {
  this.unsafePerformIO = f;
}

IO.prototype.map = function(f) {
  return new IO(_.compose(f, this.unsafePerformIO));
}
```

There, much better. Now our calling code becomes `findParam("searchTerm").unsafePerformIO()`, which is clear as day to users (and readers) of the application.

`IO` will be a loyal companion, helping us tame those feral impure actions. Next, we'll see a type similar in spirit, but has a drastically different use case.


## Asynchronous Tasks

Callbacks are the narrowing spiral staircase to hell. They are control flow as designed by M.C. Escher. With each nested callback squeezed in between the jungle gym of curly braces and parenthesis, they feel like limbo in an oubliette(how low can we go!). I'm getting claustrophobic chills just thinking about them. Not to worry, we have a much better way of dealing with asynchronous code and it starts with an "F".

The internals are a bit too complicated to spill out all over the page here so we will use `Data.Task` (previously `Data.Future`) from Quildreen Motta's fantastic [Folktale](http://folktalejs.org/). Behold some example usage:

```js
// Node readfile example:
//=======================

var fs = require('fs');

//  readFile :: String -> Task Error String
var readFile = function(filename) {
  return new Task(function(reject, result) {
    fs.readFile(filename, 'utf-8', function(err, data) {
      err ? reject(err) : result(data);
    });
  });
};

readFile("metamorphosis").map(split('\n')).map(head);
// Task("One morning, as Gregor Samsa was waking up from anxious dreams, he discovered that
// in bed he had been changed into a monstrous verminous bug.")


// jQuery getJSON example:
//========================

//  getJSON :: String -> {} -> Task Error JSON
var getJSON = curry(function(url, params) {
  return new Task(function(reject, result) {
    $.getJSON(url, params, result).fail(reject);
  });
});

getJSON('/video', {id: 10}).map(_.prop('title'));
// Task("Family Matters ep 15")

// We can put normal, non futuristic values inside as well
Task.of(3).map(function(three){ return three + 1 });
// Task(4)
```

The functions I'm calling `reject` and `result` are our error and success callbacks, respectively. As you can see, we simply `map` over the `Task` to work on the future value as if it was right there in our grasp. By now `map` should be old hat.

If you're familiar with promises, you might recognize the function `map` as `then` with `Task` playing the role of our promise. Don't fret if you aren't familiar with promises, we won't be using them anyhow because they are not pure, but the analogy holds nonetheless.

Like `IO`, `Task` will patiently wait for us to give it the green light before running. In fact, because it waits for our command, `IO` is effectively subsumed by `Task` for all things asynchronous; `readFile` and `getJSON` don't require an extra `IO` container to be pure. What's more, `Task` works in a similar fashion when we `map` over it: we're placing instructions for the future like a chore chart in a time capsule - an act of sophisticated technological procrastination.

To run our `Task`, we must call the method `fork`. This works like `unsafePerformIO`, but as the name suggests, it will fork our process and evaluation continues on without blocking our thread. This can be implemented in numerous ways with threads and such, but here it acts as a normal async call would and the big wheel of the event loop keeps on turning. Let's look at `fork`:

```js
// Pure application
//=====================
// blogTemplate :: String

//  blogPage :: Posts -> HTML
var blogPage = Handlebars.compile(blogTemplate);

//  renderPage :: Posts -> HTML
var renderPage = compose(blogPage, sortBy('date'));

//  blog :: Params -> Task Error HTML
var blog = compose(map(renderPage), getJSON('/posts'));


// Impure calling code
//=====================
blog({}).fork(
  function(error){ $("#error").html(error.message); },
  function(page){ $("#main").html(page); }
);

$('#spinner').show();
```

Upon calling `fork`, the `Task` hurries off to find some posts and render the page. Meanwhile, we show a spinner since `fork` does not wait for a response. Finally, we will either display an error or render the page onto the screen depending if the `getJSON` call succeeded or not.

Take a moment to consider how linear the control flow is here. We just read bottom to top, right to left even though the program will actually jump around a bit during execution. This makes reading and reasoning about our application simpler than having to bounce between callbacks and error handling blocks.

Goodness, would you look at that, `Task` has also swallowed up `Either`! It must do so in order to handle futuristic failures since our normal control flow does not apply in the async world. This is all well and good as it provides sufficient and pure error handling out of the box.

Even with `Task`, our `IO` and `Either` functors are not out of a job. Bear with me on a quick example that leans toward the more complex and hypothetical side, but is useful for illustrative purposes.

```js
// Postgres.connect :: Url -> IO DbConnection
// runQuery :: DbConnection -> ResultSet
// readFile :: String -> Task Error String

// Pure application
//=====================

//  dbUrl :: Config -> Either Error Url
var dbUrl = function(c) {
  return (c.uname && c.pass && c.host && c.db)
    ? Right.of("db:pg://"+c.uname+":"+c.pass+"@"+c.host+"5432/"+c.db)
    : Left.of(Error("Invalid config!"));
}

//  connectDb :: Config -> Either Error (IO DbConnection)
var connectDb = compose(map(Postgres.connect), dbUrl);

//  getConfig :: Filename -> Task Error (Either Error (IO DbConnection))
var getConfig = compose(map(compose(connectDb, JSON.parse)), readFile);


// Impure calling code
//=====================
getConfig("db.json").fork(
  logErr("couldn't read file"), either(console.log, map(runQuery))
);
```

In this example, we still make use of `Either` and `IO` from within the success branch of `readFile`. `Task` takes care of the impurities of reading a file asynchronously, but we still deal with validating the config with `Either` and wrangling the db connection with `IO`. So you see, we're still in business for all things synchronous.

I could go on, but that's all there is to it. Simple as `map`.

In practice, you'll likely have multiple asynchronous tasks in one workflow and we haven't yet acquired the full container apis to tackle this scenario. Not to worry, we'll look at monads and such soon, but first, we must examine the maths that make this all possible.


## A Spot of Theory

As mentioned before, functors come from category theory and satisfy a few laws. Let's first explore these useful properties.

```js
// identity
map(id) === id;

// composition
compose(map(f), map(g)) === map(compose(f, g));
```

The *identity* law is simple, but important. These laws are runnable bits of code so we can try them on our own functors to validate their legitimacy.

```js
var idLaw1 = map(id);
var idLaw2 = id;

idLaw1(Container.of(2));
//=> Container(2)

idLaw2(Container.of(2));
//=> Container(2)
```

You see, they are equal. Next let's look at composition.

```js
var compLaw1 = compose(map(concat(" world")), map(concat(" cruel")));
var compLaw2 = map(compose(concat(" world"), concat(" cruel")));

compLaw1(Container.of("Goodbye"));
//=> Container(' world cruelGoodbye')

compLaw2(Container.of("Goodbye"));
//=> Container(' world cruelGoodbye')
```

In category theory, functors take the objects and morphisms of a category and map them to a different category. By definition, this new category must have an identity and the ability to compose morphisms, but we needn't check because the aforementioned laws ensure these are preserved.

Perhaps our definition of a category is still a bit fuzzy. You can think of a category as a network of objects with morphisms that connect them. So a functor would map the one category to the other without breaking the network. If an object `a` is in our source category `C`, when we map it to category `D` with functor `F`, we refer to that object as `F a` (If you put it together what does that spell?!). Perhaps, it's better to look at a diagram:

<img src="images/catmap.png" alt="Categories mapped" />

For instance, `Maybe` maps our category of types and functions to a category where each object may not exist and each morphism has a `null` check. We accomplish this in code by surrounding each function with `map` and each type with our functor. We know that each of our normal types and functions will continue to compose in this new world. Technically, each functor in our code maps to a sub category of types and functions which makes all functors a particular brand called endofunctors, but for our purposes, we'll think of it as a different category.

We can also visualize the mapping of a morphism and its corresponding objects with this diagram:

<img src="images/functormap.png" alt="functor diagram" />

In addition to visualizing the mapped morphism from one category to another under the functor `F`, we see that the diagram commutes, which is to say, if you follow the arrows each route produces the same result. The different routes means different behavior, but we always end at the same type. This formalism gives us principled ways to reason about our code - we can boldly apply formulas without having to parse and examine each individual scenario. Let's take a concrete example.

```js
//  topRoute :: String -> Maybe String
var topRoute = compose(Maybe.of, reverse);

//  bottomRoute :: String -> Maybe String
var bottomRoute = compose(map(reverse), Maybe.of);


topRoute("hi");
// Maybe("ih")

bottomRoute("hi");
// Maybe("ih")
```

Or visually:

<img src="images/functormapmaybe.png" alt="functor diagram 2" />

We can instantly see and refactor code based on properties held by all functors.

Functors can stack:

```js
var nested = Task.of([Right.of("pillows"), Left.of("no sleep for you")]);

map(map(map(toUpperCase)), nested);
// Task([Right("PILLOWS"), Left("no sleep for you")])
```

What we have here with `nested` is a future array of elements that might be errors. We `map` to peel back each layer and run our function on the elements. We see no callbacks, if/else's, or for loops; just an explicit context. We do, however, have to `map(map(map(f)))`. We can instead compose functors. You heard me correctly:

```js
var Compose = function(f_g_x){
  this.getCompose = f_g_x;
}

Compose.prototype.map = function(f){
  return new Compose(map(map(f), this.getCompose));
}

var tmd = Task.of(Maybe.of("Rock over London"))

var ctmd = new Compose(tmd);

map(concat(", rock on, Chicago"), ctmd);
// Compose(Task(Maybe("Rock over London, rock on, Chicago")))

ctmd.getCompose;
// Task(Maybe("Rock over London, rock on, Chicago"))
```

There, one `map`. Functor composition is associative and earlier, we defined `Container`, which is actually called the `Identity` functor. If we have identity and associative composition we have a category. This particular category has categories as objects and functors as morphisms, which is enough to make one's brain perspire. We won't delve too far into this, but it's nice to appreciate the architectural implications or even just the simple abstract beauty in the pattern.


## In Summary

We've seen a few different functors, but there are infinitely many. Some notable omissions are iterable data structures like trees, lists, maps, pairs, you name it. eventstreams and observables are both functors. Others can be for encapsulation or even just type modelling. Functors are all around us and we'll use them extensively throughout the book.

What about calling a function with multiple functor arguments? How about working with an order sequence of impure or async actions? We haven't yet acquired the full tool set for working in this boxed up world. Next, we'll cut right to the chase and look at monads.

[Chapter 9: Monadic Onions](ch9.md)

## Exercises

```js
require('../../support');
var Task = require('data.task');
var _ = require('ramda');

// Exercise 1
// ==========
// Use _.add(x,y) and _.map(f,x) to make a function that increments a value
// inside a functor

var ex1 = undefined



//Exercise 2
// ==========
// Use _.head to get the first element of the list
var xs = Identity.of(['do', 'ray', 'me', 'fa', 'so', 'la', 'ti', 'do']);

var ex2 = undefined



// Exercise 3
// ==========
// Use safeProp and _.head to find the first initial of the user
var safeProp = _.curry(function (x, o) { return Maybe.of(o[x]); });

var user = { id: 2, name: "Albert" };

var ex3 = undefined


// Exercise 4
// ==========
// Use Maybe to rewrite ex4 without an if statement

var ex4 = function (n) {
  if (n) { return parseInt(n); }
};

var ex4 = undefined



// Exercise 5
// ==========
// Write a function that will getPost then toUpperCase the post's title

// getPost :: Int -> Future({id: Int, title: String})
var getPost = function (i) {
  return new Task(function(rej, res) {
    setTimeout(function(){
      res({id: i, title: 'Love them futures'})  
    }, 300)
  });
}

var ex5 = undefined



// Exercise 6
// ==========
// Write a function that uses checkActive() and showWelcome() to grant access
// or return the error

var showWelcome = _.compose(_.add( "Welcome "), _.prop('name'))

var checkActive = function(user) {
 return user.active ? Right.of(user) : Left.of('Your account is not active')
}

var ex6 = undefined



// Exercise 7
// ==========
// Write a validation function that checks for a length > 3. It should return
// Right(x) if it is greater than 3 and Left("You need > 3") otherwise

var ex7 = function(x) {
  return undefined // <--- write me. (don't be pointfree)
}



// Exercise 8
// ==========
// Use ex7 above and Either as a functor to save the user if they are valid or
// return the error message string. Remember either's two arguments must return
// the same type.

var save = function(x){
  return new IO(function(){
    console.log("SAVED USER!");
    return x + '-saved';
  });
}

var ex8 = undefined
```

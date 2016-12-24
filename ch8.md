# Tupperware

## La boîte qui déboîte

<img src="images/jar.jpg" alt="http://blog.dwinegar.com/2011/06/another-jar.html" />

Nous avons vu comment écrire des programmes qui acheminent des données au travers de fonctions
pures. Ils constituent une forme déclarative des spécifications de comportements attendus. Qu'en
est-il de la gestion des erreurs, du contrôle de l'exécution, des actions asynchrones, des états
et, allons jusque-là, des effets ?! Dans ce chapitre, nous bâtirons les fondations au-dessus
desquelles sont construites toutes ces abstractions importantes. 

En premier lieu, nous allons créer un contenant. Il doit pouvoir contenir n'importe quel type
de valeur; Il y a fort à parier qu'un bocal qui ne peut contenir que du pâté serve énormément.
Il s'agira d'un objet bien que nous nous refuserons à lui affecter des propriétés et des
méthodes au sens de l'orientée-objet. Non, nous le traiterons comme un coffre-fort - une
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

Si vous utilisez *node*, vous verrez `{__value: x}` bien qu'il s'agisse d'un `Container(x)`.
*Chrome* affichera le type correctement mais peu importe tant que nous comprenons ce à quoi
ressemble un `Container`. Au sein de certains environnements, vous avez la possibilité de réécrire
la méthode `inspect` mais nous n'irons pas jusque-là. Pour les besoins du livre ceci dit, nous
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

Grandement inspirée de la fameuse méthode `map` propre aux listes, cette méthode fonctionne de
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
Entre autres, il nous est possible de continuer à appliquer des fonctions à l'aide de `map`
autant que nous le voulons. Il n'y a aucun problème non plus à changer le type de la valeur à
l'intérieur du contenant comme l'ont montré les trois exemples précédents. 

Mais au fait, si l'on appelle `map`, c'est que quelque part sous le capot on fait appel à de la
composition ! Quelle sorcellerie mathématique est à l'oeuvre ici ? Bravo, nous venons de
découvrir les *Foncteurs*.

> Un Foncteur est un type qui implémente `map` et obéit à quelques lois

Oui, un *Foncteur* n'est rien de plus qu'une interface avec un contrat. On pourrait tout aussi
bien l'avoir appelé `Mappable` mais *Foncteur* rime avec *bonheur*, n'est-ce pas ? Les
Foncteurs proviennent tout droit de la théorie des catégories et bien entendu, nous aborderons
les détails techniques mathématiques à ce sujet d'ici la fin du chapitre. Pour l'instant,
tâchons d'en développer une intuition pratique et de comprendre ce qu'il se cache derrière
cette interface un tant soit peu étrange.

Quelle raison saugrenue peut bien nous pousser à encapsuler une valeur de la sorte pour ensuite
interagir avec elle via `map` ? La réponse est dans la question pour peu qu'on prenne la peine
de la reformuler: Que gagnons-nous à demander au contenant d'appliquer les fonctions à notre
place ? Eh bien, c'est un niveau d'abstraction supplémentaire dans l'application de fonctions.
Il nous suffit d'appliquer une fonction et le contenant se charge de l'appliquer pour nous en
prenant soin de gérer le type. C'est un concept puissant, le voyez-vous ? 

## Schrödinger, une histoire de Maybe

<img src="images/cat.png" alt="cool cat, need reference" />

`Container` est relativement morne. En réalité, on le présente d'ordinaire comme `Identity` et
agit de façon similaire à la fonction `id` (au risque de me répéter, sachez qu'il y a également
un lien mathématique entre ces deux entités, lien que nous expliciterons en temps voulus). Il
existe néanmoins bien d'autres foncteurs c'est-à-dire, des contenants typés qui possèdent leur
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

`Maybe` ressemble presque trait pour trait à `Container` avec pour seule petite différence
qu'il vérifie la présence d'une valeur avant d'y appliquer la fonction fournie. De cette façon,
les valeurs `null` ou `undefined` sont écartées du traitement (en outre, cette implémentation
est une version simplifié pour les besoins de l'apprentissage).

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
fonctions sur des valeurs nulles via `map`. C'est en effet le rôle du `Maybe` que de vérifier la
présence d'une valeur au sein du contenant avant d'y appliquer la fonction passée en
argument. 

Par ailleurs, cette syntaxe n'est pas des plus fines et fonctionnelles; par conséquent et en accord
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
fonctionnera comme prévue. C'est d'ailleurs aussi le cas avec la méthode `map` fournie par
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
quelque chose; il nous faut désormais traiter les valeurs nulles pouvant surgir inopinément.
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
var finishTransaction = compose(remainingBalance, updateLedger);  // <- ces fonctions composées sont hypothétiques et non-encore implémentées.

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
appels à `map` servent de bouclier et empêchent l'exécution de tout autre fonction (comme
`finishTransaction`). C'est précisément le comportement que nous espérons. Mettre à jour le
solde de notre compte alors que nous possédons des fonds insuffisants n'est pas du meilleur
effet. 

## Libérer la valeur

Ce que les gens oublient parfois c'est qu'à un moment donné, on atteint le bout de la chaîne;
des fonctions effectives qui transmettent du JSON, affichent à l'écran ou encore altèrent le
système de fichiers. À ce moment précis, il nous est impossible de retourner un résultat, il
nous faut exécuter des fonctions qui pourront communiquer avec le monde extérieur. Tel un sage
Buddhiste, nous pouvons le formuler comme le Kôan Zen suivant: "Si un programme ne possède
aucun effet observable, s'exécute-t-il seulement ? ". S'exécute-t-il selon son propre désir ?
Sans effet observable, il consomme au mieux quelques cycles CPU avant de retourner au repos. 

Le rôle de notre application est de récupérer, transformer et d'amener un ensemble de données
jusqu'à ce qu'il soit l'heure de leur dire au revoir. Ce faisant, nous appliquons
successivement des fonctions grâce à `map` de telle sorte que les données n'ont guère besoin de
quitter leur chaleureux contenant. Une erreur assez commune consiste à essayer de retirer la
valeur du `Maybe` d'une façon ou d'une autre en espérant que celle-ci se matérialise comme par
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
bibliothèques natives au travers des `Option(al)`. Lorsque l'on en vient à devoir traiter un
ensemble de vérifications à `null` à la suite (même lorsque l'on sait parfois que la valeur ne
peut simplement pas être nulle), la plupart des gens n'ont aucune solution mais ressentent
toutefois bien la pénibilité de l'écriture d'un tel code. Avec `Maybe`, vous prendrez vite
l'habitude à tel point que c'en deviendra une seconde nature. Après tout, la plupart du temps
ce sera un bien pour un moindre mal.

Développer un programme non fiable, c'est tout aussi stupide que de prendre le temps de décorer
des oeufs avec de la peinture avant de les jetez au milieu de la route. C'est une bonne chose
que de vouloir apporter un peu de robustesse à nos fonctions et `Maybe` est la pour ça.

Je manquerais à mes responsabilités si je ne vous précisais pas que la "réelle" implémentation
de `Maybe` séparera effectivement la valeur selon deux types: un pour ladite valeur, et l'autre
pour l'absence de cette valeur. Ceci nous offre une plus grande souplesse dans l'application
dans l'utilisation de `map` en permettant à des valeurs nulles comme `null` ou `undefined`
d'être également *mappées*. Ainsi, la dénomination universelle de foncteur est respectée. Vous
rencontrerez plus fréquemment des types tels que `Some(x) / None` ou `Just(x) / Nothing` plutôt
qu'un `Maybe` qui effectue une bête comparaison à `null`.

## Gestion d'erreur pure

<img src="images/fists.jpg" alt="pick a hand... need a reference" />

Ce peut être surprenant mais, `throw/catch` n'est pas vraiment pure. Lorsqu'une erreur surgit,
nous tirons la sonnette d'alarme plutôt que de retourner une valeur de sortie ! La fonction
agresse d'un millier de 1 et de 0 notre application en réponse à l'entrée qu'on lui a fourni.
Avec l'aide de notre nouvel ami `Either`, nous pouvons faire mieux que de déclarer la guerre à
nos entrées. Nous pouvons répondre un message à la fois plus poli et plus adéquat. Regardons
cela de plus près.

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
en tant que telle; n'hésitez pas à la regarder pour vous faire une idée néanmoins. Ceci étant
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
`Left` matérialise l'adolescent en crise qui refuse toute communication et requête via `map`.
`Right` fonctionne de façon similaire à `Container` (a.k.a `Identity). L'intérêt ici vient de
la capacité de `Left` à stocker un message d'erreur. 

Faisons l'hypothèse d'une fonction qui a des chances d'échouer à l'exécution. Considérons le
calcul d'un âge depuis une date de naissance. On peut utiliser `Maybe(null)` afin de signaler
une erreur et aiguiller la suite de notre programme en conséquence mais l'information est
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
sembler légérement informelle vu que nous n'avons pas présenté la classe mère `Either`. C'est
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
s'ensuit une plus grande simplicité et réutilisabilité dans le code.

En somme, `Either` est approprié pour la gestion des erreurs communes comme les résultats d'une
validation, tout autant qu'il l'est dans la gestion d'erreurs complexes qui conduisent
normalement à une interruption de l'exécution (un fichier manquant ou une socket inaccessible
par exemple). Essayez de remplacer quelques-uns des `Maybe` précédents par `Either` pour en
améliorer le retour d'information. 

En outre, j'ai l'impression d'avoir présenté `Either` d'une façon un peu réductrice. Non
seulement nous sert-il à mieux gérer les erreurs, mais aussi représente-t-il une disjonction
logique  (a.k.a `||`) au sein d'un type. Il matérialise également l'idée de *Coproduit* en
théorie des catégories (point que nous n'aborderons pas dans ce livre mais qu'il reste
intéressant de connaître ne serait-ce que pour explorer les propriétés à en exploiter). C'est
une représentation canonique de la somme (ou de l'union disjointe de deux ensembles) en tant
qu'union des quantités représentées par l'association des deux types contenus (je me doute que
ceci doive vous paraître obscure, n'hésitez pas à jeter un oeil à cet [excellent
article](https://www.fpcomplete.com/school/to-infinity-and-beyond/pick-of-the-week/sum-types)).
`Either` peut donc représenter de nombreuses choses mais en tant que foncteur, on l'associe
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

## Le vieux McDonald a des effets

<img src="images/dominoes.jpg" alt="dominoes.. need a reference" />

Dans le chapitre à propos de la pureté nous avons écrit une fonction relativement étrange. La
fonction original possédait un effet de bord et nous l'avons encapsulée dans une autre fonction
qui retournait la première. Dans la même idée, nous avons:

```js
//  getFromStorage :: String -> (_ -> String)
var getFromStorage = function(key) {
  return function() {
    return localStorage[key];
  }
}
```

Si l'on n'avait pas emprisonné les entrailles de notre fonction dans une autre, sa réponse
pourrait vraisemblablement varier selon le contexte. En revanche, notre petit mécanisme nous
assure une réponse toujours identique (pour une même entrée bien-entendu) qui n'est ni plus ni
moins qu'une fonction qui ne fois appelée nous retournera une entrée contenue dans notre
`localStorage`. Grâce à cela, nous continuons l'esprit tranquille le reste de notre application.

Toutefois il faut bien avouer que telle quelle la fonction ne nous sert pas à grand-chose. Tout
comme la figurine maintenue dans son emballage il nous est impossible de jouer avec. Si
seulement il nous était possible d'atteindre le contenu de ce paquet sans attendre la fin pour
tout déballer... C'est ici que `IO` entre en jeu.

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

`IO` se distingue des précédents foncteurs de par la nature de sa valeur `\_\_value`, toujours
une fonction. C'est toutefois davantage un détail d'implémentation et l'on s'y réfère comme à
une fonction. Ce que l'on observe réellement c'est que tout comme `getFromStorage`, `IO`
retarde l'exécution de l'action impure en la capturant au sein d'une fonction. Par conséquent
on perçoit `IO` comme contenant le résultat de l'action encapsulée plutôt que la fonction
englobante. Cela saute aux yeux avec la méthode `of`: nous obtenons un `IO(x)` même si sous le
capot, le mécanisme `IO(function(){ return x })` est nécessaire pour éviter l'évaluation
précoce.

Regardons tout ceci en action.

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

Ici, `io\_window` est bel et bien un `IO` sur lequel on peut *mapper* directement alors que `$`
est une fonction qui retourne un `IO` une fois invoquée. Remarquez que j'ai exposé les valeurs
*conceptuelles* afin de rendre les expressions plus explicites bien qu'en réalité les `IO`
s'exprimeront toujours comme `{ \_\_value: [Function] }`. Lorsqu'on `map` sur notre `IO`, on ne
fait qu'agréger une fonction en bout d'une composition qui devient la valeur d'un nouvel `IO`.
Notre fonction n'est pas exécutée, elle prend simplement sa place au milieu d'une chaîne de
calculs que l'on construit, fonction après fonction à l'image d'un ensemble de dominos que l'on
place bout à bout sans les renverser. Le résultat rappelle le *pattern* commande du Gang of Four.

Prenez une minute et faites fonctionner votre intuition sur les foncteurs. Si l'on regarde nos
expériences passées, on se sent bien confortable à pouvoir *mapper* sur des contenants dont les
bizarreries intrinsèques ne nous sont bien que contingentes. 

En outre, il est bien beau d'avoir emprisonnée la bête, il faudra tôt ou tard la libérer. En
composant notre `IO` avec de nouvelles fonctions, nous avons créé une bien puissante
quoiqu'impure fonction dont l'invocation risque fort de troubler l'ordre établi. À quel moment
est-il donc adéquat de libérer le Kraken ? Est-il ne serait-ce que possible d'exécuter
notre `IO` sans provoquer la fin du monde ? La réponse est oui à partir du moment où le code en
charge de l'appel en prend la responsabilité. Il faut que notre code pur en dépit de sa
fourberie soit préservé et maintenu innocent. C'est l'appelant qui doit également accepter le
fardeau et les responsabilités qui viennent avec les effets de bords liés à l'exécution.
Illustrons concrètement cela par un exemple:

```js

////// Notre ressource de code pure: lib/params.js ///////

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

////// La partie de code impure: main.js ///////

// run it by calling __value()!
findParam("searchTerm").__value();
// Maybe([['searchTerm', 'wafflehouse']])
```
Notre bibliothèque se dédouane de toute responsabilité et conserve sa pureté en encapsulant
`url` au sein d'un `IO` avant de passer le relais. Vous avez sans doute remarqué que nous avons
comme qui dirait empilé nos contenants; c'est toutefois tout à fait raisonnable d'avoir un
`IO(Maybe([x]))` qui se révèle être un foncteur à trois niveaux riches de sens (`Array`
peut somme toute s'interpréter comme un contenant sur lequel `map` s'applique).

Il demeure néanmoins un aspect dangereux qui me démange de rectifier dans la précédente
notation. La valeur contenue dans `IO` n'est pas vraiment une valeur, ni un quelconque attribut
privé comme le suggèrerait le préfixe devant son nom `\_\_value`. En effet, elle représente
sinon une grenade qui attend d'être dégoupillée du moins une portion de code qui mérite d'être
désignée en conséquence. Renommons celle-ci en `unsafePerformIO` afin de ne pas perdre de vue
son potentiel destructeur. 

```js
var IO = function(f) {
  this.unsafePerformIO = f;
}

IO.prototype.map = function(f) {
  return new IO(_.compose(f, this.unsafePerformIO));
}
```

Voilà qui est bien mieux. Notre code appelant se transforme en
`findParam("searchTerm").unsafePerformIO()` ce qui est de prime abord bien plus alertant. 

`IO` sera notre fidèle compagnon qui nous aidera à apprivoiser toutes ces féroces actions
impures à venir. Maintenant, nous armons nous d'un autre type poursuit un but bien différent
mais dans un esprit similaire. 

## Les tâches asynchrones

Emprunter la voie des callbacks c'est prendre un aller simple vers l'Enfer. Tels qu'imaginés
par M.C. Escher, ils contrôlent le flot d'exécution de l'application. Chacun d'eux tentant de
se dépatouiller parmi une jungle hostile d'accolades et de parenthèses. Je deviens malade rien
qu'à y penser. Pas d'inquiétude à avoir cela dit, nous avons dans nos rangs des structures bien
plus appropriés pour gérer du code asynchrones, et cela commence par un "F". 

Vous exposer dès à présent à la machinerie sur laquelle tout ceci repose serait un tantinet
rude. Ainsi nous utiliserons pour l'instant les `Data.Task` (autrefois `Data.Future`) de
Quildreen Motta et de sa fantastique bibliothèque [Folktale](http://folktalejs.org/). En voici
quelques usages:

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

// Exemple avec getJSON de JQuery:
//================================

//  getJSON :: String -> {} -> Task Error JSON
var getJSON = curry(function(url, params) {
  return new Task(function(reject, result) {
    $.getJSON(url, params, result).fail(reject);
  });
});

getJSON('/video', {id: 10}).map(_.prop('title'));
// Task("Family Matters ep 15")

// Fonctionne aussi avec des valeurs non asynchrones et complètement déterministes
Task.of(3).map(function(three){ return three + 1 });
// Task(4)
```

Les fonctions que je désigne par `reject` et `result` matérialisent respectivement nos
*callbacks* d'erreur et de succès. Comme vous pouvez le constater, nous appliquons nos
fonctions sur la `Task` en travaillant avec les valeurs à venir comme si elles étaient déjà à
portée de main. `map` doit à présent vous paraître bien familier.

Si vous êtes de plus déjà accoutumé des Promises vous ferez assez vite l'analogie entre `map`
et `then` avec `Task` tenant ici le rôle de Promise. Ne vous prenez toutefois pas le chou avec
les Promises, nous n'en utiliserons pas pour la simple et bonne raison qu'elles ne sont pas
pures mais l'analogie reste valable. 

À l'instar d'`IO`, `Task` attendra patiemment notre feu vert avec de s'exécuter conrètement. En
fait, en raison de cette attente, `Task` peut se substituer assez facilement à `IO` pour
n'importe quel travail asynchrone; `readfile` et `getJSON` ne requièrent aucun `IO` superflu
afin d'être purs. Cerise sur le gâteau, `Task` fonctionne parfaitement bien avec `map`; on y
dépose des directives futures comme une liste de tâches dans une capsule temporelle - il s'agit
là d'un exemple manifeste de procrastination technologiquement sophistiquée.  

Afin de démarrer le processus il nous faut appeler `fork` qui agit pareillement à
`unsafePerformIO`. En outre, comme son nom le suggère, cette méthode va exécuter la tâche
sans interrompre le processus courant. De fait on peut voir ici de nombreuses façons
d'implémenter ce comportement, notamment à l'aide de Threads, mais il ne s'agira ici que d'un
appel asynchrone classique qui trouvera sa place au milieu de l'Even-Loop déjà en marche.
Penchons-nous sur `fork` un peu plus:

```js
// Application pure
//=====================
// blogTemplate :: String

//  blogPage :: Posts -> HTML
var blogPage = Handlebars.compile(blogTemplate);

//  renderPage :: Posts -> HTML
var renderPage = compose(blogPage, sortBy('date'));

//  blog :: Params -> Task Error HTML
var blog = compose(map(renderPage), getJSON('/posts'));


// Exécutions impures
//=====================
blog({}).fork(
  function(error){ $("#error").html(error.message); },
  function(page){ $("#main").html(page); }
);

$('#spinner').show();
```

À l'appel de `fork`, notre tâche s'active et s'efforce à nous ramener dans les plus brefs
délais quelques articles à afficher sur notre page. Entre parenthèses, il est tout à fait
bienvenu d'afficher une bête animation de chargement pendant ce temps là; nous sommes libre de
le faire étant donné que l'appel à `fork` n'est pas bloquant. À la suite de ce petit temps
d'attente, à moins de voir surgir une erreur inopinée,  nous afficherons la page. 

Arrêtez-vous quelques instants à présent et prenez le temps d'admirer ô combien le flot
d'exécution paraît linéaire ici. Tout se lit de haut en bas, de gauche à droite même si
techniquement l'exécution n'est pas vraiment séquentielle en fin de compte. Laissons ces sauts
être exécutés par le programme et gardons pour nous cette lecture qui nous offre une force de
réflexion et de raisonnement immense. 

Doux Jésus ! Avez-vous vu cela également ? `Task` rend tout simplement `Either` caduque dans ce
cas-ci. Et il le faut car notre précédent flot de contrôle ne s'applique plus vraiment dans
le monde asynchrone où l'on doit traiter avec des erreurs potentielles cependant incertaines.
Par chance (si tant est que la chance est à voir là-dedans), les tâches ou futures nous
fournissent d'ores et déjà des mécanismes de gestion d'erreurs.

Loin de moi l'idée de mettre à la retraite nos `IO` et `Either` après de si courtes carrières
toutefois. Je vous prie d'accepter ce petit exemple qui atténue certains aspects complexes pour
mettre l'accent sur les propos précédents:

```js
// Postgres.connect :: Url -> IO DbConnection
// runQuery :: DbConnection -> ResultSet
// readFile :: String -> Task Error String

// Application pure
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


// Exécution impure
//=====================
getConfig("db.json").fork(
  logErr("couldn't read file"), either(console.log, map(runQuery))
);
```

Ce petit exemple illustre bien l'utilisation concomitante de `Either` et `IO` pour gérer la
partie en succès de notre tâche. `Task` s'occupe de gérer les impuretés liées à la lecture d'un
fichier de façon asynchrone tandis que nous devons toujours gérer la cohérence de la
configuration à l'aide d'`Either` ainsi que la connexion chancelante avec la base de données
via un `IO`. De fait, toutes ces structures sont toujours en course lorsque l'on a affaire à
du code synchrone. 

On pourrait continuer encore longtemps mais il n'y a plus tellement à raconter à présent. 

En pratique vous en conviendrez, il faudra gérer de multiples tâches asynchrones en même temps
et nous ne possédons pour l'heure aucun outil nous permettant de faire face à un tel scénario.
Cela viendra très prochainement lorsque nous attaquerons les Monades et leurs amis, mais pour
l'instant, il nous faut regarder d'un peu plus près les Maths qui rendent tout cela possible.

## Un brin de théorie

Nous l'avons évoqué précédemment, les foncteurs proviennent de la théorie des catégories et
par conséquent répondent à des lois particulières. Dans un premier temps, explorons ces
propriétés:

```js
// identity
map(id) === id;

// composition
compose(map(f), map(g)) === map(compose(f, g));
```

La loi d'identité est immédiate mais importante. Vous remarquerez que ces lois sont des
portions de code que l'on peut exécuter afin d'asseoir leur légitimité.

```js
var idLaw1 = map(id);
var idLaw2 = id;

idLaw1(Container.of(2));
//=> Container(2)

idLaw2(Container.of(2));
//=> Container(2)
```

Ça fonctionne à merveille ! Passons à la composition:

```js
var compLaw1 = compose(map(concat(" world")), map(concat(" cruel")));
var compLaw2 = map(compose(concat(" world"), concat(" cruel")));

compLaw1(Container.of("Goodbye"));
//=> Container(' world cruelGoodbye')

compLaw2(Container.of("Goodbye"));
//=> Container(' world cruelGoodbye')
```

En théorie des catégories les foncteurs prennent des éléments et des morphismes d'une catégorie
et leur associent une catégorie différente. Par définition, cette nouvelle catégorie se doit de
posséder une loi d'identité et de composition pour les morphismes mais ce sont deux propriétés
assurées de par les deux lois que l'on vient de voir. 

Notre définition de catégorie est sans doute quelque peu vague. Sommairement, une catégorie est
une sorte de réseau d'éléments interconnectés par des morphismes. Ainsi, un foncteur associera
les éléments d'une catégorie à une autre mais en conservant la structure de ce réseau. Soit un
élément `a` d'une catégorie source `C` que l'on associe à une catégorie `D` par le foncteur
`F`; on se réfère alors au nouvel élément comme `F a` (que retrouve-t-on mis bout-à-bout ?).
C'est peut-être plus clair avec un dessin:

<img src="images/catmap.png" alt="Association de catégories" />

Par exemple, `Maybe` associe notre catégorie de types et de fonctions vers une catégorie où
chaque élément peut ne pas exister et où chaque morphisme effectue une vérification à null. Du
point de vue du code, cela se traduit par deux choses: l'utilisation de `map` pour passer des
fonctions et l'encapsulation de nos types à l'intérieur de foncteurs. Au sein de ce nouveau
monde, les lois précédentes nous assurent que nos types classiques et nos fonctions seront
encore composables. Techniquement, chaque foncteur de notre code nous amène vers une
sous-catégorie de types et de fonctions de telle façon que l'on nomme ces foncteurs des
endofoncteurs. Afin de conserver les choses aussi simples que possible nous considérerons ces
sous-catégories comme de nouvelles catégories à parts entières.

Il est possible de visualiser les associations d'un morphisme et de ses éléments correspondant
au travers du schéma suivant:

<img src="images/functormap.png" alt="Schéma d'un foncteur" />

Non seulement nous visualisons l'association d'un morphisme vers une nouvelle catégorie au
moyen du foncteur `F`, mais nous pouvons aussi constater que le diagramme commute, c'est-à-dire
que l'ordre dans lequel on emprunte les flèches n'a pas d'importance, on arrive au même
résultat. Des chemins différents reflètent des comportements différents mais aboutissent en fin
de compte à un même type. Ce formalisme nous offre par ailleurs une certaine aptitude à
raisonner sur notre code sans avoir à réellement examiner tous les scénarios possibles vu
qu'ils conduisent tous au même résultat. Prenons un petit exemple:

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

Ou encore plus visuellement:

<img src="images/functormapmaybe.png" alt="Schéma d'un foncteur #2" />

Les foncteurs peuvent aussi s'accumuler ainsi:

```js
var nested = Task.of([Right.of("pillows"), Left.of("no sleep for you")]);

map(map(map(toUpperCase)), nested);
// Task([Right("PILLOWS"), Left("no sleep for you")])
```

Ce que nous avons ici est une liste d'éléments potentiellement en erreur imbriquée dans une
future. On utilise `map` chaque fois pour pénétrer l'une des structures afin d'appliquer notre
fonction sur les éléments en racine. Il n'y a aucun callback, ni if/else, ni boucle
d'ailleurs; ni plus ni moins qu'un contexte explicite. Il nous faut toutefois appliquer trois
fois l'opérateur `map` ce qui, je vous l'accorde, est un peu exagéré. On peut en revanche
composer des foncteurs pour y remédier. Oui, vous m'avez bien entendu:

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

Il n'y a ici plus qu'un seul appel à `map`. La composition sur les foncteurs est de plus
associative. Souvenez-vous plus tôt, nous avions défini un foncteur particulier que nous avions
appelé `Container` mais qui n'est en realité rien de plus que le foncteur `Identité`.
Normalement, cela sonne une petite cloche en vous: si nous avons l'identité et l'associativité
sur la composition alors nous avons de quoi définir une catégorie. Et cette catégorie
particulière possède des catégories comme éléments et des foncteurs comme morphismes; il n'en
faudra guère plus pour faire exploser notre cerveau. Nous ne creuserons pas plus loin de ce
côté là mais fort est de constater la puissance et la beauté de ce que toute cette théorie
implique et permet d'exprimer.

## En bref

Nous avons vu quelques foncteurs mais vous vous doutez bien qu'il en existe une infinité
d'autres. Parmi les omissions notables on retrouve les structures itérables comme les arbres,
les listes, les tableaux associatifs, les couples et qu'on se le dise, les streams d'événements
et les observables ont aussi leur place chez les foncteurs. Quelques autres servent à
l'encapsulation ou expriment tout simplement des types particuliers. Les foncteurs sont
omniprésents et nous en ferons désormais une utilisation extensive tout au long de ce livre.

Quelques questions se posent maintenant: 

- Peut-on appeler une fonction avec de multiples foncteurs en arguments ? 
- Comment gérer une liste ordonnée d'actions asynchrones impures ? 

Nous n'avons pas encore toutes les clés en main pour nous aventurer dans ce monde là. Il nous
faut à présent porter notre attention sur les monades. 

[Chapter 9: Monade ou Oignon ?](ch9.md)

## Exercices

```js
require('../../support');
var Task = require('data.task');
var _ = require('ramda');

// Exercice 1
// ==========
// Utiliser _.add(x,y) et _.map(f,x) pour créer une fonction qui incrémente une valeur à
// l'intérieur d'un foncteur

var ex1 = undefined



//Exercice 2
// ==========
// Utiliser _.head pour récupérer le premier élément de la liste
var xs = Identity.of(['do', 'ray', 'me', 'fa', 'so', 'la', 'ti', 'do']);

var ex2 = undefined



// Exercice 3
// ==========
// Utiliser safeProp et _.head afin de récupérer la première lettre du prénom de l'utilisateur.
var safeProp = _.curry(function (x, o) { return Maybe.of(o[x]); });

var user = { id: 2, name: "Albert" };

var ex3 = undefined


// Exercice 4
// ==========
// Récrire l'exercice 4 à l'aide de Maybe afin d'éviter l'emploi d'une structure conditionnelle

var ex4 = function (n) {
  if (n) { return parseInt(n); }
};

var ex4 = undefined



// Exercice 5
// ==========
// Écrire une fonction qui récupérera un article (getPost) et mettra en majuscule le title de l'article

// getPost :: Int -> Future({id: Int, title: String})
var getPost = function (i) {
  return new Task(function(rej, res) {
    setTimeout(function(){
      res({id: i, title: 'Love them futures'})  
    }, 300)
  });
}

var ex5 = undefined



// Exercice 6
// ==========

// En utilisant checkActive() et showWelcome(), écrire une fonction qui débloque un accès ou
// retourne une erreur.

var showWelcome = _.compose(_.add( "Welcome "), _.prop('name'))

var checkActive = function(user) {
 return user.active ? Right.of(user) : Left.of('Your account is not active')
}

var ex6 = undefined



// Exercice 7
// ==========
// Écrire une fonction de validation qui vérifie que length > 3. La fonction doit retourner un
// Right(x) le cas échéant ou un Left("You need > 3") sinon.

var ex7 = function(x) {
  return undefined // <--- write me. (don't be pointfree)
}



// Exercice 8
// ==========
// Utiliser l'ex7 précédent et Either en tant que foncteur afin d'enregistrer l'utilisateur
// s'il est valide ou échouer en fournissant un message d'erreur. Gardez à l'esprit que les deux
// arguments d'either doivent retourner le même type.

var save = function(x){
  return new IO(function(){
    console.log("SAVED USER!");
    return x + '-saved';
  });
}

var ex8 = undefined
```

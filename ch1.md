# Chapitre 1: De quoi s'agit-il exactement ?

## Introductions

Bonjour et ravi de faire votre connaissance, appelez-moi professeur Franklin Risby. Étant
supposé vous apprendre un peu de programmation fonctionnelle, nous allons être amené à passer
quelques temps ensemble. Assez parlé de moi, qu'en est-il de vous ? J'espère que vous êtes
relativement à l'aise avec le JavaScript, que vous avez ne serait-ce que de vagues notions sur
l'orienté objet, et que vous aspirez d'ores-et-déjà à devenir un meilleur développeur. Un
doctorat en Entomologie n'est pas nécessaire, toutefois il vaut mieux que vous sachiez comment
dénicher et exterminer des bugs.

Je ne ferai aucune supposition a priori sur vos éventuelles expériences passées en
programmation fonctionnelle car nous savons tout deux ce qu'il advient en pareil cas.
Néanmoins, j'espére que vous vous êtes déjà heurtés aux problèmes qui surgissent lorsque l'on
joue avec des objets mutables, des effets de bords non contrôlés et une archictecture
chancelante. Présentations faites, rentrons maintenant dans le vif du sujet.

L'objectif de ce chapitre est de vous faire toucher du doigt ce que l'on recherche au travers
de la programmation fonctionnelle. Nous devons nous faire une idée de ce qu'un programme
*fonctionel* représente afin de ne pas errer sans but, évitant les obstacles tel un zombie -
une bien triste fin me direz vous. Ce qu'il nous faut, c'est un objectif clair vers lequel
s'orienter, sorte de compas céleste nous guidant à travers une mer agitée. 

De fait, il existe des préceptes généraux, divers astuces de grands sages qui nous aideront á
travers les couloirs parfois sombres de n'importe quelle application: DRY (don't repeat
yourself, littéralement "ne vous répétez pas"), découplage et cohésion, YAGNI (ya ain't gonna
need it, littéralement "pas b'soin d'ça"), principe de la moindre surprise, responsabilité
unique, etc.

Je ne cherche pas à faire l'étalage de mes dernières découvertes... la plupart sont étroitement
liées aux concepts que nous allons manipuler et nous aurons fatalement à les aborder. En
revanche, ce que j'aimerais que vous ressentiez avant d'aller plus loin, c'est la démarche qui
sera la notre, notre utopie fonctionnelle.

<!--BREAK-->

## Première confrontation

Démarrons avec un petit grain de folie. Voici une application de mouettes. Lorsque deux volées
fusionnent (`conjoin`) elles deviennent un plus grand groupe (somme des tailles des groupes)
et, lorsqu'elles se reproduisent (`breed`), le groupe s'en voit grossi conformément (produit
des tailles des groupes). Gardez à l'esprit que le code suivant n'illustre en rien de bonnes
pratiques d'orienté objet. Il met en avant les risques que tente de prévenir de notre nouvelle
approche basée sur l'assignation. Voyez plutôt:

```js 
var Flock = function(n) { 
    this.seagulls = n; 
};

Flock.prototype.conjoin = function(other) { 
    this.seagulls += other.seagulls; 
    return this; 
};

Flock.prototype.breed = function(other) { 
    this.seagulls = this.seagulls * other.seagulls;
    return this; 
};

var flock_a = new Flock(4); 
var flock_b = new Flock(2); 
var flock_c = new Flock(0);

var result = flock_a.conjoin(flock_c) 
    .breed(flock_b).conjoin(flock_a.breed(flock_b)).seagulls;
//=> 32 
```

Qui donc sur Terre aurait envie d'écrire une telle abomination ? Il est incroyablement
difficile de garder la trace de l'état interne des objets en raison de sa mutabilité. De plus ô
Seigneur, le résultat est incorrect ! Cela devrait être `16` mais l'appel de méthode sur
`flock_a` à deux reprises a complètement altéré le processus. Pauvre `flock_a`. C'est
l'anarchie dans le code ! Rien de plus qu'une bête sauvage arithmétique. 

Si vous ne comprenez pas ce bout de programme, c'est pardonné car moi non plus. Le principal
soucis c'est que les états et les valeurs mutables sont durs à suivre même sur un exemple aussi
trivial. 

Essayons de nouveau avec cette fois, une approche plus fonctionnelle:

```js 
var conjoin = function(flock_x, flock_y) { 
    return flock_x + flock_y 
}; 
var breed = function(flock_x, flock_y) { 
    return flock_x * flock_y 
};

var flock_a = 4; 
var flock_b = 2; 
var flock_c = 0;

var result = conjoin(
    breed(flock_b, conjoin(flock_a, flock_c)), breed(flock_a, flock_b)
);
//=>16 
```

Parfait, nous obtenons un résultat cohérent cette fois-ci. Il y a moins de code. L'imbrication
des appels est un poil moins confuse...[^Nous remédierons complétement à cela au cours du ch5].
C'est mieux, mais on peut encore aller plus loin. Il y des avantages à appeler un chat un chat.
En réalité, nous ne travaillons qu'avec de simples additions (`conjoin`) et produits (`breed`). 

Il n'y a rien de réellement particulier à propos de ces deux fonctions hormis peut-être leur
nom. Changeons cela et dévoilons leur réelle identité.

```js 
var add = function(x, y) { 
    return x + y 
}; 
var multiply = function(x, y) { 
    return x * y
};

var flock_a = 4; 
var flock_b = 2; 
var flock_c = 0;

var result = add(
    multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a, flock_b)
); 
//=>16
``` 

Et grâce à cela, la sagesse des anciens nous apparaît: 

```js 
// associativité 
add(add(x, y), z) == add(x, add(y, z));

// commutativité 
add(x, y) == add(y, x);

// identité 
add(x, 0) == x;

// distributivité 
multiply(x, add(y,z)) == add(multiply(x, y), multiply(x, z)); 
```

Ah oui au fait, ces vieilles mais fidèles propriétés mathématiques vont s'avérer fort utiles à
partir de maintenant. Ne vous inquiétez pas si vous ne les connaissez pas sur le bout des
doigts. Pour la plupart d'entre-nous, il s'agit là d'un lointain souvenir. Voyons toutefois en
quoi peuvent-elles rendre nos histoires de mouettes plus simple.

```js 
// Ligne d'origine
add(multiply(flock_b, add(flock_a, flock_c)), multiply(flock_a,
flock_b));

// Appliquer la propriété d'identité pour retirer la fonction inutile
// (add(flock_a, flock_c) == flock_a)
add(multiply(flock_b, flock_a), multiply(flock_a, flock_b));

// Utiliser la distributivité pour simplifier notre résultat
multiply(flock_b, add(flock_a, flock_a));
```

Magnifique ! Nullement besoin d'autre chose que nos propres fonctions. Ajoutons les definitions
de `add` et `multiply` pour l'intégrité et on obtient une petite bibliothèque de code. C'est
toutefois inutile ici, nous trouverons sûrement des méthodes `add` et `multiply` dans une
bibliothèque existante. 

Vous devez sans doute vous dire "belle arnaque ton exemple purement mathématique" ou encore
"les vrais programmes ne sont jamais aussi simples et ce raisonnement ne tient pas debout".
J'ai choisi précisément cet exemple parce que la plupart d'entre-vous connaissent l'addition
et la multiplication; c'est donc facile d'y voir l'utilité des Mathématiques. 

Ne désespérez pas, ce livre sera parsemé de théorie des catégories, théories des ensembles et
de lambda calcul. De fait, nous viendrons à bout d'exemples réels avec la même simplicité que
dans ce cas sur les mouettes. Vous n'avez pas néanmoins besoin d'être un mathématicien; ce sera
comme utiliser n'importe quel framework ou api.

C'est peut-être suprenant de se dire que l'on peut écrire des applications "normales" avec les
même principes fonctionnels vus plus haut. Des applications pointues. Des programmes concis,
cependant peu évidents à se représenter. Ou encore de simples applications qui ne réinventent
pas la roue. L'anarchie a son intérêt si vous êtes un criminel, toutefois dans ce livre, nous
souhaitons embrasser et obéir les lois des Mathématiques. 

Ce que nous voulons, c'est une théorie au sein de laquelle chaque pièce semble s'emboîter
parfaitement. Nous souhaitons représenter des problèmes particuliers à partir de méthodes
génériques pour ensuite les exploiter pour notre propre dessein. Cela demandera plus de rigueur
que l'approche naïve et précipitée qu'offre la programmation impérative[^Nous donnerons une
définition précise de la programmation impérative plus tard; pour l'instant, considérez qu'il
s'agit de tout ce qui n'est pas de la programmation fonctionnelle], mais les bénéfices d'une
approche fonctionnelle au sein d'une théorie mathématique vous laisserons sans voix.  

Nous avons légèrement fait briller notre étoile dans un univers fonctionnel, mais il nous reste
quelques concepts clés à aborder avant de réellement entâmer notre aventure. 

[Chapitre 2: Les fonctions dites First-Class](ch2.md)

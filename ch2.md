# Chapitre 2: Les fonctions dites First-Class

## Bref résumé
Lorsque l'on dit que les fonctions sont "first class" (littéralement, de premier ordre), on
veut simplement dire qu'elles sont comme tout le monde... c'est-à-dire normales [^prof ?]. On
peut traiter les fonctions comme n'importe quel autre type de données et elles n'ont pas de
comportement particulier - faites-en ce que vous voulez, stockez-les dans une liste, passez-les
en arguments, assignez-les à des variables.

Il s'agit là des bases du JavaScript mais force est de constater l'ignorance ou bien simplement
le rejet du concept par bon nombre de sources visibles sur GitHub. Devrions-nous fournir
un petit exemple ?   
Je le pense oui.

```js
var hi = function(name){
  return "Hi " + name;
};

var greeting = function(name) {
  return hi(name);
};
```

Ici, la fonction englobant le `hi` assignée à `greeting` est complètement redondante. Pourquoi
? Parce que JavaScript vous laisse le choix d'appeler ou non vos fonctions. Si l'on munit `hi`
de parenthèses `()`, la fonction est appelée et retourne une valeur. Sans cela, `hi` retourne
simplement le contenu de la variable. Juste pour être sûr, jetons un coup d'oeil rapide à
l'exemple suivant:

```js
hi;
// function(name){
//  return "Hi " + name
// }

hi("jonas");
// "Hi jonas"
```

Et parce que `greeting` ne fait rien de plus qu'appeler `hi` avec très exactement le même
argument, nous pouvons simplement écrire:

```js
var greeting = hi;


greeting("times");
// "Hi times"
```

En d'autres termes, `hi` est déjà une fonction qui attend un unique argument, pourquoi diable
la placer au sein d'une autre fonction appelant `hi` et prenant le même fichu argument ? Cela
ne fait aucune espèce de sens sinon tout autant qu'enfiler votre plus grosse doudoune en plein
mois de Juillet pour simplement aller vous chercher une crème glacée.

En outre, il existe cette détestable pratique verbeuse qui consiste à envelopper une fonction
avec une autre dans le simple but d'en retarder l'évaluation. (Nous verrons pourquoi dans
quelques temps, mais il s'agit là de problèmes de maintenabilité).

Soyez-bien certain d'avoir saisi toute l'ampleur du propos avant d'aller plus loin; regardons
quelques exemples supplémentaires tirés tout droit de modules npm.

```js
// ignorant
var getServerStuff = function(callback){
  return ajaxCall(function(json){
    return callback(json);
  });
};

// avisé
var getServerStuff = ajaxCall;
```

Le monde est truffé de gestion d'appels ajax dans ce genre. Voyez ci-dessous en quoi les deux
propositions sont équivalentes:

```js
// cette ligne
return ajaxCall(function(json){
  return callback(json);
});

// est identique à celle-ci
return ajaxCall(callback);

// remanions getServerStuff
var getServerStuff = function(callback){
  return ajaxCall(callback);
};

// ...lequel est équivalent à:
var getServerStuff = ajaxCall; // <-- regarde maman, aucune ()
```

Ceci chers amis, est la bonne façon de faire. Un petit dernier avant que j'explique les raisons de
cette insistance.

```js
var BlogController = (function() {
  var index = function(posts) {
    return Views.index(posts);
  };

  var show = function(post) {
    return Views.show(post);
  };

  var create = function(attrs) {
    return Db.create(attrs);
  };

  var update = function(post, attrs) {
    return Db.update(post, attrs);
  };

  var destroy = function(post) {
    return Db.destroy(post);
  };

  return {
    index: index, show: show, create: create, update: update, destroy: destroy
  };
})();
```

Ce ridicule contrôleur est composé à 90% de vent. On peut tout à fait le réécrire comme ceci:

```js
var BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy
};
```

...on peut tout aussi bien le mettre à la poubelle vu qu'il ne fait rien de plus que grouper
naïvement nos vues et notre base de données.

## De l'importance du premier ordre

Bien, abordons maintenant quelques points qui font des fonctions de premier ordre un choix
privilégié. Comme nous venons de le voir dans les exemples `getServerStuff` et
`BlogController`, il est très facile d'ajouter des niveaux d'indirections dépourvus de toute
valeur sémantique et qui rendent tout bonnement le code plus complexe.

De plus, si l'une des fonctions encapsulées est amenée à changer, c'est toute la fonction
d'encapsulation qu'il faut également modifier. 

```js
httpGet('/post/2', function(json){
  return renderPost(json);
});
```
Par exemple, si `httpGet` venait à changer pour également retourner une possible `err`, il
faudrait impacter ce changement partout ailleurs où cette fonction est utilisée.

```js
// revenir sur nos pas et modifier chaque appel pour passer explicitement le paramètre err
httpGet('/post/2', function(json, err){
  return renderPost(json, err);
});
```

L'écrire sous forme de fonction first-class aurait largement simplifié la tâche:

```js
// renderPost est appelé par httpGet indépendamment du nombre d'arguments passés
httpGet('/post/2', renderPost);  
```

Au delà d'en retirer les fonctions superflues, cette méthode nous évite également d'avoir à
choisir des noms pour nos arguments. C'est une potentielle source de problèmes - d'autant plus
lorsque le code est amené à vieillir et que les spécifications changent.

Attribuer un même nom à différent concepts est bien souvent source de confusion dans un projet.
Le problème de la réutilisabilité se pose aussi. Par exemple, les deux fonctions suivantes font
exactement le même travail, cependant l'une d'elle semble infiniement plus générique et
réutilisable:

```js
// spécifique à notre blog
var validArticles = function(articles) {
  return articles.filter(function(article){
    return article !== null && article !== undefined;
  });
};

// largement plus pertinente pour de futurs projets 
var compact = function(xs) {
  return xs.filter(function(x) {
    return x !== null && x !== undefined;
  });
};
```

En désignant des choses par un nom nous nous lions implicitement à un certain type de données
(`articles` dans ce cas). Cela arrive très souvent et est une source de répétitions. 

Je souhaite aussi vous avertir qu'en raison du caractère orienté-objet du JavaScript, vous
devez préter attention à `this` qui pourrait bien vous sauter à la gorge en moment innoportun.
Si une fonction fait référence à `this` et qu'on l'utilise comme une fonction de premier ordre,
on est sujet à de sévères déconvenues.

```js
var fs = require('fs');

// effrayant
fs.readFile('freaky_friday.txt', Db.save);

// un peu moins
fs.readFile('freaky_friday.txt', Db.save.bind(Db));

```

En associant le contexte d'exécution à `Db` lui-même, on permet à la fonction d'accéder à son
prototype. J'évite au possible l'utilisation de `this`. Il n'y en a pas besoin en programmation
fonctionnelle. Cependant, en intéragissant avec des ressources externes vous vous y
confronterez de toutes les manières les plus farfelues.

Ceci étant dit, nous pouvons désormais passer au niveau suivant.

[Chapitre 3: Du pur bonheur avec du pur fonctionnel](ch3.md)

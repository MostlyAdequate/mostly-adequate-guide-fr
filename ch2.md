# Chapitre 2: Les fonctions dites First-Class

## Bref résumé
Lorsque l'on dit que les fonctions sont "first class" (littéralement, de premier ordre), on
veut simplement dire qu'elles sont comme tout le monde... normale[^prof ?]. On peut traiter les
fonctions comme n'importe quel autre type de données et elles n'ont pas de comportement
particulier - faites-en ce que vous voulez, stockez-les dans une liste, passez-les en
arguments, assignez-les à des variables.

Il s'agit là des bases du JavaScript mais force est de constater l'ignorance ou bien simplement
le rejet du concept par bon nombre de codes source visibles sur GitHub. Devrions-nous fournir
un petit exemple ? Je le pense oui.

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
simplement le contenu de la variable. Juste pour être sur, jetons un oeil rapide à l'exemple
suvant:

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

En d'autres termes, `hi` est déjà une fonction qui attends un unique argument, pourquoi diable
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

Le monde est truffé de gestion d'appel ajax dans ce genre. Voyez ci-dessous en quoi les deux
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

Ceci chers amis, est la bonne façon de faire. Un dernier avant que j'explique les raisons de
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

Okay, let's get down to the reasons to favor first class functions. As we saw in the `getServerStuff` and `BlogController` examples, it's easy to add layers of indirection that have no actual value and only increase the amount of code to maintain and search through.

In addition, if a function we are needlessly wrapping does change, we must also change our wrapper function.

```js
httpGet('/post/2', function(json){
  return renderPost(json);
});
```

If `httpGet` were to change to send a possible `err`, we would need to go back and change the "glue".

```js
// go back to every httpGet call in the application and explicitly pass err
// along.
httpGet('/post/2', function(json, err){
  return renderPost(json, err);
});
```

Had we written it as a first class function, much less would need to change:

```js
// renderPost is called from within httpGet with however many arguments it wants
httpGet('/post/2', renderPost);  
```

Besides the removal of unnecessary functions, we must name and reference arguments. Names are a bit of an issue, you see. We have potential misnomers - especially as the codebase ages and requirements change.

Having multiple names for the same concept is a common source of confusion in projects. There is also the issue of generic code. For instance, these two functions do exactly the same thing, but one feels infinitely more general and reusable:

```js
// specific to our current blog
var validArticles = function(articles) {
  return articles.filter(function(article){
    return article !== null && article !== undefined;
  });
};

// vastly more relevant for future projects
var compact = function(xs) {
  return xs.filter(function(x) {
    return x !== null && x !== undefined;
  });
};
```

By naming things, we've seemingly tied ourselves to specific data (in this case `articles`). This happens quite a bit and is a source of much reinvention.

I must mention that, just like with Object-Oriented code, you must be aware of `this` coming to bite you in the jugular. If an underlying function uses `this` and we call it first class, we are subject to this leaky abstraction's wrath.

```js
var fs = require('fs');

// scary
fs.readFile('freaky_friday.txt', Db.save);

// less so
fs.readFile('freaky_friday.txt', Db.save.bind(Db));

```

Having been bound to itself, the `Db` is free to access its prototypical garbage code. I avoid using `this` like a dirty nappy. There's really no need when writing functional code. However, when interfacing with other libraries, you'll have to acquiesce to the mad world around us.

Some will argue `this` is necessary for speed. If you are the micro-optimization sort, please close this book. If you cannot get your money back, perhaps you can exchange it for something more fiddly.

And with that, we're ready to move on.

[Chapter 3: Pure Happiness with Pure Functions](ch3.md)

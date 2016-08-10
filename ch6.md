# Chapitre 6: Exemple d'application #

## Un code déclaratif ##

Nous allons adopter une nouvelle attitude. À partir de maintenant, nous cesserons de dire à
l'ordinateur ce qu'il doit faire, et spécifierons à la place le résultat que nous aimerions
avoir. Je suis sûr que vous trouverez ça bien plus reposant que d'essayer de tout micro-gérer
tout le temps.

En déclaratif, contrairement à l'impératif, nous écrivons des expressions, au lieu
d'instructions étape par étape.

Pensez à SQL où il n'y a pas de "Fais d'abord ceci, puis fais cela". Il y a une expression qui
spécifie ce que l'on attend de la base de données. On ne décide pas la manière dont est fait le
travail, _elle_ le décide. Quand la base de données est mise à jour et que le moteur SQL est
optimisé, nous n'avons pas besoin de changer nos requêtes. Ceci parce qu'il existe de multiple
façons d'interpréter nos spécifications et d'atteindre un même résultat.

Pour certaines personnes, moi inclus, il est difficile de saisir du premier coup le concept de
la programmation déclarative, alors familiarisons-nous avec quelques exemples.

```js 
// impératif 
var makes = []; 
for (i = 0; i < cars.length; i++) {
    makes.push(cars[i].make); 
}


// déclaratif 
var makes = cars.map(function(car){ return car.make; }); 
```

La boucle impérative doit premièrement instancier le tableau. L'interpréteur doit évaluer cette
instruction avant de continuer. Il itère ensuite directement sur la liste des voitures,
incrémentant manuellement un compteur et faisant l'exposition vulgaire et explicite de ses
mécanismes d'itération.

La version `map` est une seule expression. Elle n'impose pas d'ordre d'évaluation. Elle donne
beaucoup de liberté à la fonction map pour itérer et assembler le tableau à retourner. Elle
spécifie le *quoi* et non le *comment*. Elle arbore ainsi fièrement l'insigne du code
déclaratif.

En plus d'être plus claires et concises, les entrailles de la fonction map peuvent être optimisées
à souhait sans que jamais notre précieux code applicatif ne doive changer.

Pour ceux parmi vous qui pensent "Certes, mais la boucle impérative est bien plus rapide", je
suggère de vous renseigner sur la manière dont le JIT optimise votre code. Voici une [vidéo
géniale qui pourrait vous éclairer](https://www.youtube.com/watch?v=65-RbBwZQdU) (en anglais).

Voici un autre exemple :

```js 
// impératif 
var authenticate = function(form) { 
    var user = toUser(form); 
    return logIn(user); 
};

// déclaratif 
var authenticate = compose(logIn, toUser); 
```

Bien que la version impérative ne soit pas nécessairement mauvaise, elle garde une évaluation
étape par étape inscrite dans son code. L'expression `compose` décrit simplement un fait :
l'authentification est la composition de `toUser` et `logIn`. Encore une fois, ça laisse une
marge de manoeuvre pour ajuster le code utilitaire, et résume notre code applicatif à une
spécification de haut niveau.

Parce que nulle part n'est écrit l'ordre d'évaluation, la programmation déclarative se prête
bien aux calculs parallèles. Ceci, couplé aux fonctions pures, fait de la programmation
fonctionnelle une bonne option pour une future parallélisation. Créer des systèmes
concurrents/parallèles ne requiert pas d'action spéciale de notre part.

## Flickr en programmation fonctionnelle ##

Nous allons maintenant construire un exemple d'application dans un style déclaratif et
composable. Nous tricherons encore un peu et utiliserons des effets de bords, mais ils seront
rares et séparés de notre code pur. Nous allons construire un widget pour navigateur qui
récupère des images flickr et les affiche. Commençons par la structure de l'application. Voici
le code HTML :

```html 
<!DOCTYPE html> 
<html> 
<head> 
    <script src="https://cdnjs.cloudflare.com/ajax/libs/require.js/2.1.11/require.min.js"></script> 
    <script src="flickr.js"></script> 
</head> 
<body></body> 
</html> 
```

Et voici la structure du fichier `flickr.js` :

```js 
requirejs.config({ 
    paths: { 
        ramda: 'https://cdnjs.cloudflare.com/ajax/libs/ramda/0.13.0/ramda.min', 
        jquery: 'https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min' 
    } 
});

require([ 'ramda', 'jquery' ], function (_, $) { 
    var trace = _.curry(function(tag, x) {
        console.log(tag, x);
        return x; 
    }); 
    // Code de l'application...  
}); 
```

On utilisera [ramda](http://ramdajs.com) au lieu de lodash, une autre bibliothèque utilitaire.
Elle inclut `compose`, `curry`, et d'autres. J'ai utilisé `requirejs`, ce qui peut sembler
exagéré, mais nous l'utiliserons tout au long de ce livre, et l'uniformité est cruciale. Aussi,
j'ai introduit directement notre belle fonction `trace` pour un débogage facile.

Maintenant que ceci est bouclé, attaquons les spécifications. Notre application fera 4 choses :

1. Construire une URL pour notre terme de recherche
2. Faire l'appel à l'API flickr
3. Transformer le JSON de sortie en images HTML
4. Afficher les images à l'écran

Deux des actions mentionnées sont impures. Vous les voyez ? Ces bouts concernant la
récupération de données depuis flickr et l'affichage à l'écran. Définissons-les en premier,
qu'on puisse ensuite les isoler.

```js 
var Impure = { 
    getJSON: _.curry(function(callback, url) { 
        $.getJSON(url, callback); 
    }),
    setHtml: _.curry(function(sel, html) { 
        $(sel).html(html); 
    }) 
}; 
```

Nous avons simplement enrobé les méthodes jQuery pour les curryfier et nous avons interverti
les arguments pour avoir une configuration plus favorable. Elles sont préfixées par `Impure`
pour nous rappeler que ce sont des fonctions dangereuses. Dans un futur exemple, nous rendrons
ces deux fonctions pures.

Ensuite, nous devons construire une URL à passer à notre fonction `Impure.getJSON`.

```js 
var url = function (term) { 
    return 'https://api.flickr.com/services/feeds/photos_public.gne?tags=' + term + '&format=json&jsoncallback=?'; 
}; 
```

Il y a des manières élégantes mais inutilement complexes d'écrire la fonction `url` dans un
style pointfree en utilisant les monoïdes (nous en apprendrons plus à ce propos plus tard) ou
les combinateurs. Nous avons opté pour une version lisible où nous assemblons la chaîne
classiquement.

Écrivons une fonction `app` qui effectue l'appel et positionne le contenu à l'écran.

```js 
var app = _.compose(Impure.getJSON(trace("response")), url);

app("cats"); 
```

Cela appelle notre fonction `url`, puis passe la chaîne à notre fonction `getJSON`, qui a été
partiellement appliquée avec `trace`. Au chargement de l'application, la réponse de l'API sera
affichée dans la console.

<img src="images/console_ss.png"/>

Nous voulons obtenir des images à partir de ce JSON. On dirait que les sources `src` sont
enfouies dans `items` puis dans la propriété `m` de chaque `media`

Dans tous les cas, récupérer ces propriétés enfouies, on peut utiliser une fonction _getter_
universelle de ramda appelée `_.prop()`. En voici une version maison, pour que vous voyiez de
quoi il en retourne :

```js 
var prop = _.curry(function(property, object){ 
    return object[property]; 
}); 
```

Ce n'est pas bien compliqué. Nous utilisons juste la syntaxe `[]` pour accéder à la propriété
d'un objet quelconque. Mettons cette fonction à l'oeuvre pour obtenir les `src` :

```js 
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var srcs = _.compose(_.map(mediaUrl), _.prop('items')); 
```

Une fois les `item` rassemblés, nous devons les _mapper_ pour en extraire les URL `media`. Le
résultat est un joli tableau de `src`. Incluons ça dans notre application et affichons-les à
l'écran.

```js 
var renderImages = _.compose(Impure.setHtml("body"), srcs); 
var app = _.compose(Impure.getJSON(renderImages), url); 
```

Nous n'avons fait que créer une nouvelle composition qui va appeler les `srcs` et les faire
occuper le corps du HTML. Maintenant que nous avons autre chose à afficher qu'un JSON brut,
nous avons remplacé l'appel à `trace` par `renderImages`. Les sources des images seront jetées
négligemment dans le `body`.

L'étape finale est de transformer ces sources en jolies images. Dans une application plus
grosse, nous utiliserions une bibliothèque de template/DOM telle que Handlebars ou React.
Cependant, ici, nous n'avons besoin que d'un tag `img` alors gardons jQuery.

```js 
var img = function (url) { return $('<img />', { src: url }); }; 
```

La méthode `html()` de jQuery accepte une liste de balises. Nous avons juste besoin de
transformer nos srcs en images et de les envoyer à travers `setHtml`.

```js 
var images = _.compose(_.map(img), srcs); 
var renderImages = _.compose(Impure.setHtml("body"), images);
var app = _.compose(Impure.getJSON(renderImages), url); 
```

Et voilà !

<img src="images/cats_ss.png" />

Voici le script terminé : 
```js 
requirejs.config({ 
    paths: { 
        ramda: 'https://cdnjs.cloudflare.com/ajax/libs/ramda/0.13.0/ramda.min', 
        jquery: 'https://ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min' 
    } 
});

require([ 'ramda', 'jquery' ], function (_, $) { 
    //////////////////////////////////
    // Utils

    var Impure = { 
        getJSON: _.curry(function(callback, url) { 
            $.getJSON(url, callback); 
        }),

        setHtml: _.curry(function(sel, html) {
            $(sel).html(html); }) 
        };

    var img = function (url) { 
        return $('<img />', { 
            src: url 
        }); 
    };

    var trace = _.curry(function(tag, x) { 
        console.log(tag, x);
        return x; 
    });

    ////////////////////////////////////////////

    var url = function (t) { 
        return 'http://api.flickr.com/services/feeds/photos_public.gne?tags=' + 
            t + '&format=json&jsoncallback=?'; };

    var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

    var srcs = _.compose(_.map(mediaUrl), _.prop('items'));

    var images = _.compose(_.map(img), srcs);

    var renderImages = _.compose(Impure.setHtml("body"), images);

    var app = _.compose(Impure.getJSON(renderImages), url);

    app("cats"); 
}); 
```

Regardez-moi ça. Une magnifique spécification déclarative de ce que les choses sont, et non
comment elles deviennent ce qu'elles sont. On peut maintenant lire chaque ligne comme une
équation dont les propriétés sont respectées. On peut se servir de ces propriétés pour
raisonner à propos de notre application et la refondre.

## Un refactor s'impose ##

Une optimisation existe quelque part : nous mappons les items pour les convertir en URLs, puis
nous mappons encore une fois pour obtenir des balises img. Il existe une loi concernant map et
la composition :

```js 
// Loi de la composition de map 
var loi = compose(map(f), map(g)) == map(compose(f, g));
```

Nous pouvons utiliser cette propriété pour optimiser notre code. Lançons-nous dans un refactor
raisonné.

```js 
// Code original 
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var srcs = _.compose(_.map(mediaUrl), _.prop('items'));

var images = _.compose(_.map(img), srcs);
```

Alignons nos maps. On peut remplacer `srcs` dans `images` par son équivalent, grâce à la pureté
des fonctions et au raisonnement basé sur les équations.

```js 
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var images = _.compose(_.map(img), _.map(mediaUrl), _.prop('items')); 
```

Maintenant que les maps sont chaînés, on peut appliquer la loi de composition.

```js 
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var images = _.compose(_.map(_.compose(img, mediaUrl)), _.prop('items')); 
```

À présent, une seule itération suffit pour convertir chaque item en une img. Améliorons juste
la lisibilité en extrayant la fonction.

```js 
var mediaUrl = _.compose(_.prop('m'), _.prop('media'));

var mediaToImg = _.compose(img, mediaUrl);

var images = _.compose(_.map(mediaToImg), _.prop('items')); 
```

## En bref ##

Nous avons vu comment mettre nos compétences en oeuvre avec une petite, mais réelle,
application. Nous avons utiliser notre cadre mathématique pour raisonner et refactorer notre
code. Mais qu'en est-il de la gestion d'erreur et du contrôle de flux ? Comment peut-on rendre
pure l'application entière au lieu de simplement cacher les fonction destructrices derrière des
espaces de noms ? Comment faire pour rendre notre application plus fiable et expressive ? Ce
sont des questions que nous aborderons en deuxième partie.

[Chapter 7: Hindley-Milner et Moi](ch7.md)

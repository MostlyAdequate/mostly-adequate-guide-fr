![cover](images/cover.png)

## Avant-propos

Ce livre traite de programmation fonctionnelle de manière générale. Afin d'appuyer le propos,
nous utiliserons le plus populaire des langages fonctionnels: JavaScript. Certains d'entre-vous
jugeront le choix douteux en ce qu'il s'oppose au courant actuel qui prône un JavaScript
davantage impératif.  Toutefois, je crois sincèrement que l'apprentissage de la programmation
fonctionnelle par ce biais présente de nombreux avantages:

- **Vous l'utilisez quotidiennement au boulot**
    La mise en application des connaissances théoriques que vous allez acquérir peut être
    faites sur des projets réels et complets plutôt que sur de petites séances nocturnes au
    moyen d'un quelconque langage fonctionnel plus ou moins ésotérique. 

- **Nullement besoin de se familiariser avec les bases du langage pour commencer**
    Avec un langage fonctionnel pur, vous ne pouvez espérer afficher une variable ou lire un
    élément du DOM sans avoir recours aux monads. Ici, on triche légèrement; on apprend à
    purifier notre code au fur et à mesure. De plus, il sera toujours plus facile de retomber
    sur ses pattes en cas de nécessité dans la mesure où nous évoluerons dans un paradigme
    mixte. 

- **Ce langage est clairement apte à écrire du code fonctionnel de haute qualité**

    Toutes les fonctionnalités du Scala ou de l'Haskell peuvent être amenées avec l'aide d'une
    ou deux petites bibliothèques. La programmation orientée objet 'classique' domine
    majoritairement l'industrie mais convient assez mal au JavaScript. Autant camper au
    milieu d'une autoroute ou tenter de faire des claquettes en ballerines. Il faut avoir
    recourt à des `bind` intempestifs afin de garder `this` sous contrôle; il n'y a pour
    l'instant pas de notion de classes, et quand bien même, il nous faut mettre en oeuvre des
    astuces pour pallier aux conséquences subtilement suprenantes d'un éventuel oubli du
    mot-clé `this`. De plus, la notion d'attributs privés passe par l'utilisation de
    *closures*. Voilà pourquoi pour bon nombre d'entre-nous, le paradigme fonctionnel apparaît
    comme bien plus naturel. 

    Ceci étant dit, les langages fonctionnels typés statiquement reflètent sans aucun doute
    encore mieux les concepts présentés dans ce livre. Le JavaScript est ici un outil
    d'apprentissage; appliquez ensuite ces notions où bon vous semblera. Par chance, la
    machinerie repose sur des théories mathématiques et demeure par conséquent universelle.
    Vous vous sentirez à l'aise avec Swift, Scala, Haskell, Purescript et tout autre
    environnement où les Mathématiques prédominent.

## Gitbook (Pour plus de confort)

- Lire en ligne *à venir*
- Télécharger EPUB *à venir*
- Télécharger Mobi (Kindle) *à venir*

## Soyez grand et faites-le vous-même

```
git clone https://github.com/MostlyAdequate/mostly-adequate-guide-fr

cd mostly-adequate-guide-fr/
npm install gitbook-cli -g
gitbook init

brew update
brew cask install calibre

gitbook mobi . ./functional.mobi
```

## Sommaire

[SUMMARY.md](SUMMARY.md)

## Comment contribuer ?

[CONTRIBUTING.md](CONTRIBUTING.md)

## Traductions

[TRANSLATIONS.md](TRANSLATIONS.md)

## Planification

- **La première partie** introduit les notions de base. Elle est mise à jour fréquemment
  lorsque des erreurs sont trouvées. Toute aide est la bienvenue !

- **La seconde partie** expose des types plus complets tels que les Foncteurs ou les Monades.
  J'espère trouver le temps de parler des transformeurs et de présenter une pure application
  concrète.

- **La dernière partie** sera à cheval entre le savoir pratique et les absurdités académiques.
  Nous traiterons des comonades, des f-algebres, des monades libres, du lemme de Yoneda et
  d'autres éléments propres à la théorie des catégories.

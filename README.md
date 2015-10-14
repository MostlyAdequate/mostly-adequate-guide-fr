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

# Autres langues

- [English (Anglais)](https://github.com/MostlyAdequate/mostly-adequate-guide) version originale par Brian Lonsdorf @DrBoolean
- [中文版 (Chinois)](https://github.com/llh911001/mostly-adequate-guide-chinese)  par Linghao
Li @llh911001
- [Русский (Russe)](https://github.com/MostlyAdequate/mostly-adequate-guide-ru) par Maksim
Filippov @maksimf
- [Português (Portuguais)](https://github.com/MostlyAdequate/mostly-adequate-guide-pt-BR) par
Palmer Oliveira @expalmer

# Sommaire

## Partie 1

* [Chapitre 1: De quoi s'agit-il exactement ?](ch1.md)
  * [Introductions](ch1.md#introductions)
  * [Première confrontation](ch1.md#première-confrontation)
* [Chapitre 2: Les fonctions dites First-Class](ch2.md)
  * [Bref résumé](ch2.md#bref-résumé)
  * [De l'importance du premier ordre](ch2.md#de-limportance-du-premier-ordre)
* [Chapitre 3: Du pur bonheur avec du pur fonctionnel](ch3.md)
  * [Soyez pur à nouveau](ch3.md#soyez-pur-à-nouveau)
  * [Les effets de bord c'est aussi...](ch3.md#les-effets-de-bord-cest-aussi)
  * [BAC + 8 de Maths](ch3.md#bac8-de-maths)
  * [Plaidoyer en faveur de la pureté](ch3.md#plaidoyer-en-faveur-de-la-pureté)
  * [En bref](ch3.md#en-bref)
* [Chapitre 4: Curryfication](ch4.md)
  * [Can't live if livin' is without you](ch4.md#cant-live-if-livin-is-without-you)
  * [Plus qu'un jeu de mots ou une sauce délicieuse](ch4.md#plus-quun-jeu-de-mots-ou-une-sauce-délicieuse)
  * [En bref](ch4.md#en-bref)
* [Chapitre 5: Composer, c'est coder](ch5.md)
  * [L'élevage de fonctions](ch5.md#functional-husbandry)
  * [Pointfree](ch5.md#pointfree)
  * [Débuggage](ch5.md#debugging)
  * [La théorie des Catégories](ch5.md#category-theory)
  * [En bref](ch5.md#in-summary)
* [Chapitre 6: Exemple d'application](ch6.md)
  * [Un code déclaratif](ch6.md#declarative-coding)
  * [Flickr en programmation fonctionnelle](ch6.md#a-flickr-of-functional-programming)
  * [Un refactor s'impose](ch6.md#a-principled-refactor)
  * [En bref](ch6.md#in-summary)

## Partie 2

* [Chapitre 7: Hindley-Milner et Moi](ch7.md)
  * [Quel est ton type ?](ch7.md#whats-your-type)
  * [Récit d'un mystérieux monde](ch7.md#tales-from-the-cryptic)
  * [Restreindre les possibilités](ch7.md#narrowing-the-possibility)
  * [Des théorèmes à la pelle](ch7.md#free-as-in-theorem)
  * [En bref](ch7.md#in-summary)
* [Chapitre 8: Tupperware](ch8.md)
  * [La boîte qui déboîte](ch8.md#the-mighty-container)
  * [Mon premier Foncteur](ch8.md#my-first-functor)
  * [Schrödinger, une histoire de Maybe](ch8.md#schrodingers-maybe)
  * [Gestion d'erreur pure](ch8.md#pure-error-handling)
  * [Le vieux McDonald à des effets](ch8.md#old-mcdonald-had-effects)
  * [Les tâches asynchrones](ch8.md#asynchronous-tasks)
  * [Un brin de théorie](ch8.md#a-spot-of-theory)
  * [En bref](ch8.md#in-summary)
* [Chapitre 9: Monade ou Oignon ?](ch9.md)
  * [La fabrique à Foncteurs pointés](ch9.md#pointy-functor-factory)
  * [Mélangeons les métaphores](ch9.md#mixing-metaphors)
  * [My chain hits my chest](ch9.md#my-chain-hits-my-chest)
  * [Théorie](ch9.md#theory)
  * [En bref](ch9.md#in-summary)
* [Chapitre 10: Foncteur applicatif](ch10.md)
  * [Appliquer des applicatives](ch10.md#applying-applicatives)
  * [Des navires en bouteille](ch10.md#ships-in-bottles)
  * [Coordination et motivation](ch10.md#coordination-motivation)
  * [Bro, do you even lift?](ch10.md#bro-do-you-even-lift)
  * [Ouvre-bouteille gratuit](ch10.md#free-can-openers)
  * [Lois](ch10.md#laws)
  * [En bref](ch10.md#in-summary)


# Planification

- **La première partie** introduit les notions de base. Elle est mise à jour fréquemment
  lorsque des erreurs sont trouvées. Toute aide est la bienvenue !

- **La seconde partie** expose des types plus complets tels que les Foncteurs ou les Monades.
  J'espère trouver le temps de parler des transformeurs et de présenter une pure application
  concrète.

- **La dernière partie** sera à cheval entre le savoir pratique et les absurdités académiques.
  Nous traiterons des comonades, des f-algebres, des monades libres, du lemme de Yoneda et
  d'autres éléments propres à la théorie des catégories.

# Comment contribuer ?

Pour contribuer, je vous demande de procéder comme suis:

- Forkez ce repository
- Regardez parmi les issues existantes ce qui se fait
- Rejoignez une tâche déjà en cours ou ouvrez une issue pour signalez le début d'une
  nouvelle en décrivant votre objectif
- Pour les traductions pensez à vérifier le sommaire sur le `README` d'accueil et
  surtout, renseignez le `CHANGELOG` en précisant le hash du commit du repository anglais
  source utilisé pour la traduction ainsi que sa date
- Une fois votre travail accompli, effectuez une pull request et indiquez quelles issues sont
  concernées

Un grand merci !

# M2 MALIA - Big Data Management - Rendu TP1

Rendu par Angelo Lamure-Fontanini. Le document est constitué de 3 partie :

* Partie 1 : Premières requêtes
* Partie 2 : Prise en main de quelques étapes d'agrégation
* Partie 3 : Pipelines d'agrégation


## Partie 1 : Premières requêtes

1. `db.grades.findOne()` et `db.zips.findOne()`:
```json
{
  _id: ObjectId('50b59cd75bed76f46522c34e'),
  student_id: 0,
  class_id: 2,
  scores: [
    {
      type: 'exam',
      score: 57.92947112575566
    },
    {
      type: 'quiz',
      score: 21.24542588206755
    },
    {
      type: 'homework',
      score: 68.1956781058743
    },
    {
      type: 'homework',
      score: 67.95019716560351
    },
    {
      type: 'homework',
      score: 18.81037253352722
    }
  ]
}
```
et
```json
{
  _id: '01001',
  city: 'AGAWAM',
  loc: [
    -72.622739,
    42.070206
  ],
  pop: 15338,
  state: 'MA'
}
```
> Dans le cas de la BDD **grades**, on a une structure avec un id *_id*, un id d'étudiant *studient_id*, une idée de classe *class_id* et une liste de notes dans l'argument *scores*. C'est une table qui peut contenir des notes, avec une clé *type* et la note dans la clé *score*.
>
> * *_id* : ObjectId
> * *studient_id* : int
> * *class_id* : int
> * *score* : array avec des objets tel que :
>   * *type* : string, 
>   * *score* : int
>
> Dans le cas de la BDD **zips**, on a une structure plus simple, un id *_id*, une ville *city*, la localisation gps *loc*, qui contient dans cet ordre la longitude et la latitude, la population *pop* ainsi que l'état de la ville *state*.
>
> * *_id* : string
> * *city* : string
> * *loc* : array de deux float
> * *pop* : int
> * *state* : string

2. `db.grades.find({})` 

```json
{
  _id: ObjectId('50b59cd75bed76f46522c351'),
  student_id: 0,
  class_id: 16,
  scores: [
    {
      type: 'exam',
      score: 59.1805667559299
    },
    {
      type: 'quiz',
      score: 47.58960202938239
    },
    {
      type: 'homework',
      score: 6.48470951607214
    },
    {
      type: 'homework',
      score: 68.33519637418685
    },
    {
      type: 'homework',
      score: 78.53068038180965
    }
  ]
}

{
  _id: ObjectId('50b59cd75bed76f46522c361'),
  student_id: 3,
  class_id: 10,
  scores: [
    {
      type: 'exam',
      score: 35.47946463550763
    },
    {
      type: 'quiz',
      score: 94.14222652833352
    },
    {
      type: 'homework',
      score: 58.43343860077279
    },
    {
      type: 'homework',
      score: 66.83562834109681
    }
  ]
}
```

> On ici la même structure décrite dans la question précédente : *_id*, *student_id*, *class_id* et une liste *scores*. 

3. `db.grades.find({}).itcount()`

> Permet de compter le nombre d'éléments pour la commande `db.grades.find({})` (280). On utilisera cette commande à chaque fois que sera indiqué le nombre de résultats.

4. `db.grades.find({ class_id: 20})` : 7 résultats, dernière correspondance :

```json
{
  _id: ObjectId('50b59cd75bed76f46522c45e'),
  student_id: 46,
  class_id: 20,
  scores: [
    {
      type: 'exam',
      score: 27.79390777259294
    },
    {
      type: 'quiz',
      score: 29.18509374453628
    },
    {
      type: 'homework',
      score: 59.51122184508205
    },
    {
      type: 'homework',
      score: 74.05819778852317
    },
    {
      type: 'homework',
      score: 86.91457200027877
    }
  ]
}
```

5. `db.grades.find({ class_id: { $lte: 20 }})` : 188 résultats, dernière correspondance : 

```json
{
  _id: ObjectId('50b59cd75bed76f46522c36d'),
  student_id: 4,
  class_id: 8,
  scores: [
    {
      type: 'exam',
      score: 60.13912489935064
    },
    {
      type: 'quiz',
      score: 16.49453097445802
    },
    {
      type: 'homework',
      score: 20.64686745448644
    },
    {
      type: 'homework',
      score: 72.03200074025955
    },
    {
      type: 'homework',
      score: 23.97256438707867
    }
  ]
}
```

6. `db.grades.find({$expr: { $gte: [ "$student_id", "$class_id" ] }})` : 188 résultats, dernière correspondance :

```json
{
  _id: ObjectId('50b59cd75bed76f46522c3a6'),
  student_id: 15,
  class_id: 0,
  scores: [
    {
      type: 'exam',
      score: 84.22482529114681
    },
    {
      type: 'quiz',
      score: 68.51195197931716
    },
    {
      type: 'homework',
      score: 48.135589104224
    }
  ]
}
```

7. a. `db.grades.find({ class_id: { $gte: 10, $lte: 20}})` : 100 résultats, dernière correspondance : 

```json
{
  _id: ObjectId('50b59cd75bed76f46522c383'),
  student_id: 8,
  class_id: 11,
  scores: [
    {
      type: 'exam',
      score: 89.12537830331416
    },
    {
      type: 'quiz',
      score: 11.48939533818351
    },
    {
      type: 'homework',
      score: 16.96585149869495
    },
    {
      type: 'homework',
      score: 17.8760904208101
    },
    {
      type: 'homework',
      score: 80.93286662099982
    },
    {
      type: 'homework',
      score: 28.93004389653184
    }
  ]
}
```

7. b. Deuxième version de la commande : 

   `db.grades.find({ $expr: { $and: [ { $gte: ["$class_id", 10] }, { $lte: ["$class_id", 20] } ]}})`

8. Requête : 

```shell
db.grades.find(
    {},
    {
        _id: 0,
        ue: "$class_id",
        etu: "$student_id",
        scores: 1
    }
)
```

On obtient 280 résultats. La dernière correspondance est : 

```json
{
  _id: ObjectId('50b59cd75bed76f46522c361'),
  scores: [
    {
      type: 'exam',
      score: 35.47946463550763
    },
    {
      type: 'quiz',
      score: 94.14222652833352
    },
    {
      type: 'homework',
      score: 58.43343860077279
    },
    {
      type: 'homework',
      score: 66.83562834109681
    }
  ],
  ue: 10,
  etu: 3
}
```



## Partie 2 : Prise en main de quelques étapes d'agrégation

1. On ajoute un champ *somme*, qui est la somme de la *class_id* et de la *student_id*.

```shell
db.grades.aggregate([
    {
        $project: {
            student_id: 1,
            class_id: 1,
            scores: 1,
            somme: { $add: ["$class_id", "$student_id"] }
        }
    }
])
```

On a comme dernier résultat : (au total 280)

```json
{
  _id: ObjectId('50b59cd75bed76f46522c361'),
  student_id: 3,
  class_id: 10,
  scores: [
    {
      type: 'exam',
      score: 35.47946463550763
    },
    {
      type: 'quiz',
      score: 94.14222652833352
    },
    {
      type: 'homework',
      score: 58.43343860077279
    },
    {
      type: 'homework',
      score: 66.83562834109681
    }
  ],
  somme: 13
}
```

2. On recherche dans **zips** les documents avec une population supérieure à 10000.

```shell
db.zips.aggregate([
    {
        $match: {pop: { $gte: 10000 }}
    }
])
```

On a 7634 résultats. Le dernier étant : 

```json
{
  _id: '01109',
  city: 'SPRINGFIELD',
  loc: [
    -72.554349,
    42.114455
  ],
  pop: 32635,
  state: 'MA'
}
```

3. On veut trier les document de **grades** en fonction de *class_id* puis *student_id* (en ordre croissant) :

```shell
db.grades.aggregate([
    {
        $sort: {class_id: 1, student_id: 1}
    }
])
```

On a 280 résultats (ce qui est logique, on a encore tous les documents). On montre ici le dernier (afficher, pas le dernier de la liste, car mongoDB coupe avant la fin le résultat de la requête) : 

```json
{
  _id: ObjectId('50b59cd75bed76f46522c425'),
  student_id: 38,
  class_id: 17,
  scores: [
    {
      type: 'exam',
      score: 47.26471309500098
    },
    {
      type: 'quiz',
      score: 25.33754474182506
    },
    {
      type: 'homework',
      score: 5.284458287115489
    },
    {
      type: 'homework',
      score: 41.30099854002849
    },
    {
      type: 'homework',
      score: 98.88182867118168
    },
    {
      type: 'homework',
      score: 60.91939590823482
    }
  ]
}
```

4. Dans la collection **grades**, on produit un document pour chaque élément du tableau contenu dans le champ `scores` :

```shell
db.grades.aggregate([
    {
        $unwind: "$scores"
    }
])
```

On a 1241 éléments, dont le dernier afficher : 

```json
{
  _id: ObjectId('50b59cd75bed76f46522c352'),
  student_id: 0,
  class_id: 24,
  scores: {
    type: 'homework',
    score: 86.79352850434199
  }
}
```

5. Dans la collection **zips**, on donne pour chaque ville la population minimum (dans le champs *min_pop*).

```shell
db.zips.aggregate([
    {
        $group: {_id: "$city", min_pop: {$min: "$pop"}}
    }
])
```

On a 16584 résultats. Donc le dernier affiché : 

```json
{
  _id: 'NORA',
  min_pop: 75
}
```

6. On va ici effectuer une jointure entre **grades** et **zips**, sur les champs respectifs *student_id* et *pop*. 

```shell
db.grades.aggregate([
    {
        $lookup: {
            from: "zips",
            localField: "student_id",
            foreignField: "pop",
            as: "jointure_grades_zips"
        }
    }
])
```

On obtient 280 résultats (ce qui est rassurant, on a fait une jointure sur **grades**, on a donc autant de résultats que de documents dans celui-ci). Le dernier affiché est : 

```json
{
  _id: ObjectId('50b59cd75bed76f46522c361'),
  student_id: 3,
  class_id: 10,
  scores: [
    {
      type: 'exam',
      score: 35.47946463550763
    },
    {
      type: 'quiz',
      score: 94.14222652833352
    },
    {
      type: 'homework',
      score: 58.43343860077279
    },
    {
      type: 'homework',
      score: 66.83562834109681
    }
  ],
  jointure_grades_zips: [
    {
      _id: '04570',
      city: 'SQUIRREL ISLAND',
      loc: [
        -69.630974,
        43.809031
      ],
      pop: 3,
      state: 'ME'
    },
    {
      _id: '60604',
      city: 'CHICAGO',
      loc: [
        -87.632999,
        41.87845
      ],
      pop: 3,
      state: 'IL'
    },
    {
      _id: '99692',
      city: 'DUTCH HARBOR',
      loc: [
        -167.510656,
        53.362757
      ],
      pop: 3,
      state: 'AK'
    }
  ]
}
```

On voit que l'on a, pour ce document avec *student_id=3*, toutes les villes avec une population de 3 (*pop=3*). On a donc la structure d'un document **grades** décrite dans la partie 1, avec en ajout une table dans le champ *jointure_grades_zips* avec les documents **zips**, où *student_id=pop*.  

## Partie 3 : Pipelines d'agrégation

1. Dans la collection **grades**, on veut calculer le nombre de notes de chaque type.

```shell
db.grades.aggregate([
    {$unwind: "$scores"},
    {$group: {_id: "$scores.type", count: {$sum: 1}}}
])
```

On obtient :

```json
{
  _id: 'exam',
  count: 280
}
{
  _id: 'homework',
  count: 681
}
{
  _id: 'quiz',
  count: 280
}
```

2. On veut, pour chaque matière, la meilleure note de l'examen.

```shell
db.grades.aggregate([
    {$unwind: "$scores"},
    {$match: {"scores.type": "exam"}},
    {$group: {_id: "$class_id", best_mark: {$max: "$scores.score"}}}
])
```

On obtient 31 résultats (matières). Les deux derniers affichés sont :

```json 
{
  _id: 12,
  best_mark: 95.99100704101919
}
{
  _id: 15,
  best_mark: 96.96878730277227
}
```

3. Dans **zips**, on veut donner les dix villes les plus peuplées. On crée le champ *tot_pop* qui contiendra la population de chaque ville. 

```shell
db.zips.aggregate([
    {$group: {_id: "$city", tot_pop: {$sum: "$pop"}}},
    {$sort: {tot_pop: -1}},
    {$limit: 10 }
])
```

On a bien 10 résultats. Les 3 premiers étant : 

```json
{
  _id: 'CHICAGO',
  tot_pop: 2452177
}
{
  _id: 'BROOKLYN',
  tot_pop: 2341387
}
{
  _id: 'HOUSTON',
  tot_pop: 2123053
}
```

On peut remarquer une chose : on a New York un peu plus loin avec seulement 1476790 habitants, ce qui est très en dessous de la réalité (c'est la ville la plus peuplée, avec plus de 8 millions d'habitants, dans New York directement, et 20 millions en aire urbaine). On peut donc se poser la question de la fiabilité de cette base de données. Ça pourrait être la population dans une autre époque, mais Chicago n'a jamais été la ville la plus peuplée, et encore moins la plus peuplée que New York. 

4. On aimerai obtenir la moyenne de la population (*pop*) des ville (*city*) groupé par état (*state*).

```shell
db.zips.aggregate([
    {$group: {_id: "$state", avg_city_pop: {$avg: "$pop"}}}
])
```

On obtient 51 résultats : les 50 états des États-Unis + le district de Colombia. On a les derniers résultats :

```json
{
  _id: 'DC',
  avg_city_pop: 25287.5
}
{
  _id: 'WV',
  avg_city_pop: 2733.454268292683
}
{
  _id: 'WY',
  avg_city_pop: 3239.4857142857145
}
```

5. On veut donner pour chaque matière, les étudiants ayant une note d'examen supérieure à 50.

```shell
db.grades.aggregate([
    {$unwind: "$scores" },
    {$match: {"scores.type": "exam", "scores.score": {$gte: 50}}},
    {$group: {_id: "$class_id", students: {$push: "$student_id"}}},
])
```

On obtient 31 résultats (comme pour les scores max de chaque matière question 2). On a le dernier résultat :

```json
{
  _id: 24,
  students: [
    2,
    15,
    40
  ]
}
```

6. On veut donner pour chaque étudiant et chaque matière sa note générale. Avec :
   * La note générale est la moyenne de chaque type de note. 
   * S'il y a plusieurs notes d'un même type, on prendra d'abord la moyenne de ce type avant de l'utiliser pour calculer la moyenne avec les autres types

```shell
db.grades.aggregate([
    {$unwind: "$scores" },
    {$group: {_id: {student_id: "$student_id", class_id: "$class_id", type: "$scores.type"}, avg: { $avg: "$scores.score"}}},
    {$group: {_id: {student_id: "$_id.student_id", class_id: "$_id.class_id"}, avg: { $avg: "$avg"}}}
])
```

On obtient 280 résultats. Le dernier étant :

```json
{
  _id: {
    student_id: 48,
    class_id: 14
  },
  avg: 52.55030646333287
}
```

7. On veut donner, pour chaque matière le type d'épreuve la mieux réussie. Le type d'épreuve la mieux réussie est celui ayant la meilleure moyenne, calculée sur l'ensemble des notes de ce type pour cette matière.

```shell
db.grades.aggregate([
    {$unwind: "$scores" },
    {$group: {_id: {class_id: "$class_id", type: "$scores.type"}, avg: {$avg: "$scores.score"}}},
    {$group: {_id: "$_id.class_id", avg: {$push: {type: "$_id.type", avg: "$avg"}}, best_avg: {$max: "$avg"}}},
    {$project: {type_with_best_mark: {$arrayElemAt: ["$avg", { $indexOfArray: ["$avg.avg", "$best_avg"]}]}}}
])
```

On obtient 31 résultats (les 31 matières comme vue précédement). On a le dernier résultat affiché :

```json
{
  _id: 25,
  type_with_best_mark: {
    type: 'quiz',
    avg: 48.595580262242
  }
}
```




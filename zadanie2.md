### Zadanie 2
W zadaniu skorzystałem z bazy GetGlue zamieszczonej na stronie prowadzącego zajęcia. Pierwszym krokiem było skonstruowanie zapytań w MongoDB. Drugim krokiem przepisanie ich do PyMongo.

Aby zainicjować bazę w PyMongo dopisujemy na początku 2 linie do kodu:
```sh
from pymongo import MongoClient
db = MongoClient().test
```

##Agregacja 1 -> 7 najbardziej aktywnych użytkowników

Wynik:
```sh
{ "_id" : "LukeWilliamss", "count" : 696782 }
{ "_id" : "demi_konti", "count" : 68137 }
{ "_id" : "bangwid", "count" : 59261 }
{ "_id" : "zenofmac", "count" : 56233 }
{ "_id" : "agentdunham", "count" : 55740 }
{ "_id" : "cillax", "count" : 43161 }
{ "_id" : "tamtomo", "count" : 42378 }
```

![Alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/activeusers.png)

MongoDB:
```sh
db.movies.aggregate(
    {$group:{_id: "$userId", count:{$sum: 1}}},
    {$sort:{count: -1}},{$limit: 7});
```
PyMongo:
```sh
db.movies.aggregate(
    {"$group":{"_id": "$userId", "count":{"$sum": 1}}},
    {"$sort":{"count": -1}},{"$limit": 7});
```

## Agregacja 2 -> 7 reżyserów z największą liczbą nakręconych filmów

Wynik:
```sh
{ "_id" : "alfred hitchcock", "count" : 50 }
{ "_id" : "michael curtiz", "count" : 48 }
{ "_id" : "woody allen", "count" : 47 }
{ "_id" : "takashi miike", "count" : 43 }
{ "_id" : "jesus franco", "count" : 43 }
{ "_id" : "ingmar bergman", "count" : 42 }
{ "_id" : "john ford", "count" : 42 }
```

![Alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/directors.png)

MongoDB:
```sh
db.movies.aggregate(
    { $match: {"modelName": "movies" || "tv_shows"  } },
    { $group: {_id: {"dir": "$director", id: "$title"}, count: {$sum: 1}} },
    { $group: {_id: "$_id.dir" , count: {$sum: 1}} },
    { $sort: {count: -1} },
    { $skip: 2},
    { $limit : 8}
    );
```
PyMongo:
```sh
db.movies.aggregate(
    { "$match": {"modelName": "movies" || "tv_shows"  } },
    { "$group": {"_id": {"dir": "$director", "id": "$title"}, "count": {"$sum": 1}} },
    { "$group": {"_id": "$_id.dir" , "count": {"$sum": 1}} },
    { "$sort": {"count": -1} },
    { "$skip" : 2},
    { "$limit" : 8}
    );
```
Pomijamy pierwsze dwa wyniki ponieważ są to filmy z nieokreślonym reżyserem albo grupa reżyserów (poniżej):
```sh
{ "_id" : "not available", "count" : 1474 }
{ "_id" : "various directors", "count" : 54 }
```

## Agregacja 3 -> 7 najpopularniejszych seriali

Wynik:
```sh
{ "_id" : "The Big Bang Theory", "count" : 260686 }
{ "_id" : "Fringe", "count" : 187910 }
{ "_id" : "Nikita", "count" : 150683 }
{ "_id" : "Glee", "count" : 146799 }
{ "_id" : "Supernatural", "count" : 130454 }
{ "_id" : "True Blood", "count" : 122913 }
{ "_id" : "The Walking Dead", "count" : 119369 }
```

![Alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/tvshows.png)

MongoDB:
```sh
db.collection.aggregate(     { $match: {"modelName": "tv_shows"} },
    { $group: {_id: "$title", count: {$sum: 1}} },
    { $sort: {count: -1} },
    { $limit : 7}     );
```
PyMongo:
```sh
db.movies.aggregate(
    { "$match": {"modelName": "tv_shows"} },
    { "$group": {"_id": "$title", "count": {"$sum": 1}} },
    { "$sort": {"count": -1} },
    { "$limit" : 7}
    );
```

## Agregacja 4 -> 7 filmów z największą liczbą polubień

Wynik:
```sh
{ "_id" : "Lord of the Rings: The Return of the King", "count" : 815 }
{ "_id" : "Fight Club", "count" : 725 }
{ "_id" : "Iron Man", "count" : 722 }
{ "_id" : "Pulp Fiction", "count" : 721 }
{ "_id" : "The Hangover", "count" : 691 }
{ "_id" : "X-Men", "count" : 667 }
{ "_id" : "Monsters, Inc.", "count" : 644 }
```

![Alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/likes.png)

MongoDB:
```sh
db.movies.aggregate( 
    { $match: { "action": "Liked" }},
    { $match: { "comment": {$ne: ""} } },
    { $group: { _id: "$title", count: {$sum: 1} } }, 
    { $sort: { count: -1 } }, { $limit: 7 } );
```
PyMongo:
```sh
db.movies.aggregate( 
    { "$match": { "action": "Liked" }},
    { "$match": { "comment": {"$ne": ""} } }, 
    { "$group": { "_id": "$title", "count": {"$sum": 1} } }, 
    { "$sort": { "count": -1 } }, { "$limit": 7 } );
```

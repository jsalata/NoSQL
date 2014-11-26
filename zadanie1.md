ZADANIE 1
=====


|Specyfikacja komputera |                                         |
|-----------------------|-----------------------------------------|
| System operacyjny     | Linux Mint 17 XFCE (64-bit)             |
| Procesor              | Intel(R) Core(TM) i3-2370M CPU @ 2.40GHz|
| Liczba rdzeni         | 4                                       |
| Pamięć Ram            | 4 GB                                    |
| Dysk twardy           | 320 GB HDD                              |


###Zadanie 1a
#####Przygotowanie pliku Train.csv
Po pobraniu pliku z serwera trzeba przekonwertować go tak, aby możliwy był import do MongoDB.
W tym celu skorzystałem ze skryptu, który uzyskałem z repozytorium prowadzącego.
```sh
#! /bin/bash

if [ -z "$1" ] ; then
echo "First replaces all the \\n with spaces then replaces all the \\r with \\n"
echo "Replace header line with: _id,title,body,tags (for mongoDB)"
echo "usage: $0 input.csv output.csv"
exit 1;
fi

tr '\n' ' ' < "$1" | tr '\r' '\n' > "$2"

sed -i '$ d' "$2"

sed -i '1 c "_id","title","body","tags"' "$2" 
```
Czas przygotowywania pliku:

real	16m45.896s
user	0m42.775s
sys	1m23.773s

#####Import przygotowanego pliku do bazy MongoDB
Zaimportowałem plik korzystając z poniższej komendy:
```sh
time mongoimport -c Topics --type csv --file Train.csv --headerline
```
#####Import przygotowanego pliku do bazy PostgreSQL
Aby zaimportować plik do PostgreSQL konieczne było utworzenie tabeli:
```sh
CREATE TABLE Topics(
ID INT PRIMARY KEY NOT NULL,
TITLE CHAR(128) NOT NULL,
BODY CHAR(128) NOT NULL,
TAGS CHAR(128) NOT NULL,
);
```
I wypełnienie jej danymi z pliku:
```sh
COPY Topics FROM '/home/jsalata/NoSQL/Train/TrainFixed.csv' DELIMITER ',' CSV;
```
Aby zmierzyć czas w PostgreSQL wykorzystałem komendę:
```sh
\timing
```
##Zestawienie czasów importu

| Baza Danych |                    Niemonitorowany             |                    Monitorowany          |
|-------------|:----------------------------------------------:|:----------------------------------------:|
|   MongoDB (2.6.4)   | real.10m19.423s  user.1m55.413s  sys.0m14.548s | real.8m21.330s  user.1m56.758s  sys.0m13.011s|
| PostgreSQL  |  764281.623ms (12 min) |   766421.893ms (12 min)   |   
| Mongo 2.8.0 RC z WiredTiger  |   -     |  785426.593ms (13 min) | 

###MongoDB (2.6.4)

![Alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/mongo264.png)

###PostgreSQL

![Alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/psql.png)

###MongoDB (2.8.0 RC z WiredTiger)

![Alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/mongo280.png)

### Zadanie 1b
W celu policzenia liczby dokumentów w kolekcji Mongo wpisałem komendę:
```sh
db.Topics.count()
```
Wynik:
```sh
6.034.195
```
### Zadanie 1c
```sh
var database = db.Train.find();

var tags = {};
var records = [];

var tagsCounter = 0;
var uniqueTagsCounter = 0;
var docsCounter = 0;

database.forEach(function(document) {
    tagsCounter++;
    var arrayOfTags = [];
    if(document.tags.constructor === String) {
        arrayOfTags = document.tags.split(" ");
    } 
    else { 
        arrayOfTags.push(document.tags);
    }
document.tags = arrayOfTags;
tagsCounter = arrayOfTags.length + tagsCounter;

for(var i=0; i<arrayOfTags.length; i++) {
    if(tags[arrayOfTags[i]] === undefined) {
        uniqueTagsCounter++;  
        tags[arrayOfTags[i]] = 1; 
    }
}

records.push(document);
if (docsCounter % 10000 === 0 || docsCounter === 6034195) {
    db.TrainFixed.insert(records);
    records = [];
    }
});

print("Number of tags:" + tagsCounter);
print("Number of unique tags:" + uniqueTagsCounter);
```
### Zadanie 1d
Importujemy do Mongo bazę z ważniejszymi miastami Polski za pomocą polecenia: 
```sh
mongoimport -c places < polskiemiasta.json
```
Dodajemy geoindeks do kolekcji places:
```sh

db.places.ensureIndex({loc : "2dsphere"})
```
Zapytanie 1: Miasta oddalone od Gdańska o maksymalnie 300 km:
```sh
db.places.find({loc: {$near: {$geometry: {type: "Point", coordinates: [18.62938, 54.35403]}, $maxDistance: 300000}}}).skip(1)
```
[Geojson1](https://github.com/jsalata/NoSQL/blob/master/zapytania/zapytanie1.geojson)

Zapytanie 2: Toruń oraz 2 najbliższe mu miasta:
```sh
db.places.find({loc: {$near: {$geometry: {type: "Point", coordinates: [18.613415, 53.018262]}}}}).limit(3)
```
[Geojson2](https://github.com/jsalata/NoSQL/blob/master/zapytania/zapytanie2.geojson)

Zapytanie 3. Miasta leżące w obrębie tzw. Polski B:
```sh
var polygon = {
  "type": "Polygon",
  "coordinates": [[
  [18.94043,54.342749],
  [18.71109,53.448551],
  [18.141174,53.103404],
  [19.109344,52.659466],
  [19.451294,51.821137],
  [19.264526,50.742321],
  [18.660278,49.765965],
  [22.873535,48.860198],
  [24.510498,51.066859],
  [23.708496,54.235538],
  [21.154175,54.500951],
  [18.94043,54.342749]
  ]]
}
db.places.find({loc: {$geoWithin: {$geometry: polygon}}})
```
[Geojson3](https://github.com/jsalata/NoSQL/blob/master/zapytania/zapytanie3.geojson)

Zapytanie 4. Miasta na linii Gdańsk-Warszawa:
```sh
var Gdansk = db.places.findOne({ _id: "Gdansk" })
var Warszawa = db.places.findOne({ _id: "Warszawa" })
db.places.find({loc: {$geoIntersects: {$geometry: {"type": "LineString", "coordinates": [Gdansk.loc.coordinates,Warszawa.loc.coordinates]}}}})
```
[Geojson4](https://github.com/jsalata/NoSQL/blob/master/zapytania/zapytanie4.geojson)

Zapytanie 5. Miasta położone na południku 19.021797
```sh
db.places.find({loc: {$geoIntersects: {$geometry: {type: "LineString", coordinates: [[19.021797, -90],[19.021797, 90]]}}}})
```
[Geojson5](https://github.com/jsalata/NoSQL/blob/master/zapytania/zapytanie5.geojson) 

Zapytanie 6. Miasto leżące najbliżej Poznania:
```sh
var Poznan = db.places.findOne({ _id: "Poznan" })
db.places.find({loc: {$near: {$geometry: Poznan.loc, $maxDistance: 600000}}}).skip(1).limit(1)
```
[Geojson6](https://github.com/jsalata/NoSQL/blob/master/zapytania/zapytanie6.geojson)

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

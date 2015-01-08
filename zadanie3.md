ZADANIE 3
=====


|Specyfikacja komputera |                                         |
|-----------------------|-----------------------------------------|
| System operacyjny     | Linux Mint 17 XFCE (64-bit)             |
| Procesor              | Intel(R) Core(TM) i3-2370M CPU @ 2.40GHz|
| Liczba rdzeni         | 4                                       |
| Pamięć Ram            | 4 GB                                    |
| Dysk twardy           | 320 GB HDD                              |


### Anagramy

Plik [word_list.txt](http://wbzyl.inf.ug.edu.pl/nosql/doc/data/word_list.txt) zaimportowałem do bazy MongoDB.

W celu importu skorzystałem z polecenia:
```sh
mongoimport -d anagramy -c anagramy --type csv --file /home/jsalata/Pobrane/word_list.txt -f "words"
```
Zaimportowano:
```sh
> db.anagramy.count()
8199
```


Anagramy uzyskałem z użyciem mapreduce:
```sh
db.anagramy.mapReduce(
  function(){emit(Array.sum(this.words.split("").sort()), this.words);},
  function(key, values) {return values.toString()},
  {
    query: {},
    out: "anagrams"
  }
)
```
W ten sposób uzyskałem pary posortowanych słów oraz ciągów znaków. 




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
mongoimport -d anagramy -c anagramy --type csv --file /home/jsalata/Pobrane/word_list.txt -f "word"
```

Czas importu:
```sh
real	0m1.879s
user	0m0.069s
sys	0m0.044s
```


Zaimportowano:
```sh
> db.anagramy.count()
8199
```

Anagramy uzyskałem z użyciem mapreduce:
```sh
var m = function() {
    emit(Array.sum(this.word.split("").sort()), this.word);
};
var r = function(key, values) {
    return values.toString();
};

db.anagramy.mapReduce(
  m,
  r,
  {
    query: {},
    out: "anagramy"
  }
)
```
W ten sposób uzyskałem pary posortowanych słów oraz ciągów znaków. 

```sh
{ "_id" : "aaabcl", "value" : "cabala" }
{ "_id" : "aaabcn", "value" : "cabana" }
{ "_id" : "aaabcs", "value" : "casaba" }
{ "_id" : "aaabnn", "value" : "banana" }
{ "_id" : "aaabrz", "value" : "bazaar" }
{ "_id" : "aaacci", "value" : "acacia" }
{ "_id" : "aaaclp", "value" : "alpaca" }
{ "_id" : "aaacmr", "value" : "maraca" }
{ "_id" : "aaadmr", "value" : "armada" }
{ "_id" : "aaaelz", "value" : "azalea" }
{ "_id" : "aaaitx", "value" : "ataxia" }
{ "_id" : "aaajmp", "value" : "pajama" }
{ "_id" : "aaalms", "value" : "salaam" }
{ "_id" : "aaamnn", "value" : "manana" }
{ "_id" : "aaamnp", "value" : "panama" }
{ "_id" : "aaappy", "value" : "papaya" }
{ "_id" : "aaartv", "value" : "avatar" }
{ "_id" : "aabbbo", "value" : "baobab" }
{ "_id" : "aabblo", "value" : "balboa" }
{ "_id" : "aabcls", "value" : "cabals" }
Type "it" for more
```

### Wikipedia

Pobrałem [plik z artykułami z wikipedii](http://dumps.wikimedia.org/plwiki/latest/plwiki-latest-pages-articles-multistream.xml.bz2), a następnie wykorzystując [XML2CSVGenericConverter](http://sourceforge.net/projects/xml2csvgenericconverter/files/?source=navbar) przekonwertowałem go do formatu csv w celu zaimportowania do MongoDB.

Aby przekonwertować plik użyłem polecenia:
```sh
time java -jar XML2CSVGenericConverter_V1.0.0.jar -v -i /home/jsalata/Pobrane/wikipedia.xml -o /home/
```

Czas konwersji:
```sh
real	15m3.784s
user	11m11.028s
sys	  0m44.588s
```
![alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/konwersja%20wiki.png "")

W celu zaimportowania posłużyłem się komendą:
```sh
time mongoimport -d test -c test --type csv --file wikipedia.csv --headerline --ignoreBlanks
```

Czas importu:
```sh
real	42m28.185s
user	7m34.223s
sys	  1m24.641s
```
![alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/import%20wiki.png "")

Na koniec skorzystałem z mapreduce:
```sh
var map = function() {  
    var id = this.id;
    if (id) { 
        id = id.split(";"); 
        for (var i = id.length - 1; i >= 0; i--) {
            if (id[i])  {    
               emit(id[i], 1
            }
        }
    }
};
var reduce = function( key, values ) {    
    var count = 0;    
    values.forEach(function(v) {            
        count +=v;    
    });
    return count;
}
db.test.mapReduce(map, reduce, {out: "word_count"})
```

Czas wykonywania oscylował w granicach 120 minut.

![alt text](https://raw.githubusercontent.com/jsalata/NoSQL/master/images/wiki-wykres.png "")



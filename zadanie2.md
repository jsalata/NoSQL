### Zadanie 2
W zadaniu skorzystałem z bazy GetGlue zamieszczonej na stronie prowadzącego zajęcia. Pierwszym krokiem było skonstruowanie zapytań w MongoDB. Drugim krokiem przepisanie ich do PyMongo.

Aby zainicjować bazę w PyMongo dopisałem na początku 2 linie do kodu:
```sh
from pymongo import MongoClient
db = MongoClient().test
```

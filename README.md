# Запуск mongodb

```
docker-compose -f mongobasic/docker-compose.yaml 
```

# Импорта данных

Выполняем следующие команды

```
docker cp path/to/dataset.json mongodb:/
docker exec -it mongodb bash
mongoimport --db hw --collection restaurants --drop --file /restaurants.json
```

Как результат видим следующий вывод

```
2021-10-13T16:11:20.541+0000    connected to: mongodb://localhost/
2021-10-13T16:11:20.541+0000    dropping: hw.restaurants
2021-10-13T16:11:20.971+0000    2548 document(s) imported successfully. 0 document(s) failed to import.
```

# Поиск и обновление документов

Далее выполняем поиск 5 ресторанов на улице `67 High Street` с рейтингом выше 5

```
mongo hw
> db.restaurants.find({"address": "67 High Street", "rating": {$gte: 5}}).limit(5)

{ "_id" : ObjectId("55f14312c7447c3da7051dca"), "URL" : "http://www.just-eat.co.uk/restaurants-allinonejohnstone-pa5/menu", "address" : "67 High Street", "address line 2" : "Johnstone", "name" : "All In One Johnstone", "outcode" : "PA5", "postcode" : "8QG", "rating" : 5, "type_of_food" : "Kebab" }
{ "_id" : ObjectId("55f14312c7447c3da7052029"), "URL" : "http://www.just-eat.co.uk/restaurants-aspendos-tn29/menu", "address" : "67 High Street", "address line 2" : "Dymchurch", "name" : "Aspendos", "outcode" : "TN29", "postcode" : "0NH", "rating" : 6, "type_of_food" : "Pizza" }
```

Далее найдем 10 ресторанов в которых готовят китайскую еду или пиццу

```
db.restaurants.find({"type_of_food": {$in: ["Pizza", "Chinese"]}}).limit(10)
```

Обновим рейтинг ресторана с id `55f14312c7447c3da7051dca`

```
db.restaurants.updateOne({ _id: ObjectId("55f14312c7447c3da7051dca") }, {$set: {"rating": 6}})

{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }

db.restaurants.find({_id: ObjectId("55f14312c7447c3da7051dca") }, {"rating": 1})

{ "_id" : ObjectId("55f14312c7447c3da7051dca"), "rating" : 6 }
```

# Индексы

Далее найдем 5 ресторанов с рейтингом выше 7

```
db.restaurants.find({"rating": {$gt: 7}}).sort({"rating": 1}).limit(5).explain("executionStats")

{
        "explainVersion" : "1",
        ...
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 5,
                "executionTimeMillis" : 4,
                "totalKeysExamined" : 0,
                "totalDocsExamined" : 2548,
                "executionStages" : {
                    ...
                }
        }
    ...
}
```

Далее создадим индекс для поля `rating` и запустим запрос еще раз

```
db.restaurants.createIndex({rating: 1})
db.restaurants.find({"rating": {$gt: 4}}).sort({"rating": 1}).limit(5).explain("executionStats")

{
        "explainVersion" : "1",
        ...
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 5,
                "executionTimeMillis" : 3,
                "totalKeysExamined" : 5,
                "totalDocsExamined" : 5,
                "executionStages" : {
                    ...
                }
        },
        ...
}
```

Как видно в первом случае было просмотрено 2548 документов, тогда как после создание индекса только 5.

Проверим поиск по тексту. Найдем 5 ресторанов содержащий в своем названии слово `Wembley`

```
db.restaurants.createIndex({"name": "text"})
db.restaurants.find({$text: {$search: "Wembley"}}).limit(5).explain("executionStats")

{
        "explainVersion" : "1",
        ...
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 2,
                "executionTimeMillis" : 0,
                "totalKeysExamined" : 2,
                "totalDocsExamined" : 2,
                "executionStages" : {
                    ...
                }
        },
        ...
}
```

Как видно понадобилось проверить только два документа при поиске с использованием текстового индекса.
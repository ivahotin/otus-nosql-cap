# Запуск mongodb

```
docker-compose -f mongobasic/docker-compose.yaml 
```

# Процесс импорта данных

Заходим в контейнер и создаем БД в которую будем выполнять импорт данных.

```
docker exec -it mongodb bash
mongo
> use restaurants
> exit
exit
```

Далее копируем dataset внутрь контейнера

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
                        "stage" : "SORT",
                        "nReturned" : 5,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 2556,
                        "advanced" : 5,
                        "needTime" : 2550,
                        "needYield" : 0,
                        "saveState" : 2,
                        "restoreState" : 2,
                        "isEOF" : 1,
                        "sortPattern" : {
                                "rating" : 1
                        },
                        "memLimit" : 104857600,
                        "limitAmount" : 5,
                        "type" : "simple",
                        "totalDataSizeSorted" : 2329,
                        "usedDisk" : false,
                        "inputStage" : {
                                "stage" : "COLLSCAN",
                                "filter" : {
                                        "rating" : {
                                                "$gt" : 4
                                        }
                                },
                                "nReturned" : 2228,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 2550,
                                "advanced" : 2228,
                                "needTime" : 321,
                                "needYield" : 0,
                                "saveState" : 2,
                                "restoreState" : 2,
                                "isEOF" : 1,
                                "direction" : "forward",
                                "docsExamined" : 2548
                        }
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
                        "stage" : "LIMIT",
                        "nReturned" : 5,
                        "executionTimeMillisEstimate" : 0,
                        "works" : 6,
                        "advanced" : 5,
                        "needTime" : 0,
                        "needYield" : 0,
                        "saveState" : 0,
                        "restoreState" : 0,
                        "isEOF" : 1,
                        "limitAmount" : 5,
                        "inputStage" : {
                                "stage" : "FETCH",
                                "nReturned" : 5,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 5,
                                "advanced" : 5,
                                "needTime" : 0,
                                "needYield" : 0,
                                "saveState" : 0,
                                "restoreState" : 0,
                                "isEOF" : 0,
                                "docsExamined" : 5,
                                "alreadyHasObj" : 0,
                                "inputStage" : {
                                        "stage" : "IXSCAN",
                                        "nReturned" : 5,
                                        "executionTimeMillisEstimate" : 0,
                                        "works" : 5,
                                        "advanced" : 5,
                                        "needTime" : 0,
                                        "needYield" : 0,
                                        "saveState" : 0,
                                        "restoreState" : 0,
                                        "isEOF" : 0,
                                        "keyPattern" : {
                                                "rating" : 1
                                        },
                                        "indexName" : "rating_1",
                                        "isMultiKey" : false,
                                        "multiKeyPaths" : {
                                                "rating" : [ ]
                                        },
                                        "isUnique" : false,
                                        "isSparse" : false,
                                        "isPartial" : false,
                                        "indexVersion" : 2,
                                        "direction" : "forward",
                                        "indexBounds" : {
                                                "rating" : [
                                                        "(4.0, inf.0]"
                                                ]
                                        },
                                        "keysExamined" : 5,
                                        "seeks" : 1,
                                        "dupsTested" : 0,
                                        "dupsDropped" : 0
                                }
                        }
                }
        },
        ...
}
```

Как видно в первом случае было просмотрено 2548 документов, тогда как после создание индекса только 5.
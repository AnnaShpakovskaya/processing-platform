# Processing platform

Платформа позволяет выполнять обработку изображений и последующий поиск информации на изображениях. Пользуясь API системы можно выполнять поиск изображений, удовлетворяющих некоторым критериям, либо получить полные результаты анализа для какого-то конкретного изображения.

Система анализирует получаемые изображения и сохраняет результаты для дальнейшего поиска или использования. Вся обработка и анализ изображений выполняются асинхронно, а обработчики могут работать как независимо друг от друга, так и совместно. Такой подход позволяет реализовывать практически любые варианты обработки данных, начиная от простейшего поиска объектов на изображении, и заканчивая сложными схемами улучшения изображений.

В публичном доступе сейчас доступны базовые обработчики изображений, выполяющие разметку лиц и номерных знаков автомобилей. Несмотря на довольно скудный функционал, этого достаточно чтобы получить представление о принципах работы системы.

Адрес сервера `http://45.135.265.211:9991`

## API

Система предоставляет REST интерфейс для загрузки изображений и последующео поиска объектов на загруженных изображениях.

### Загрузка изображений

POST `/v1/processor/sink` 

Параметры

Name | Type | Description
--------------- | --------------- | ---------------
data | MultipartFile | Изображение для выполнения анализа
processors | String | Перечисление процессоров для выполнения обработок. Процессоры должны быть разделены ",". Возможны делующие значения: `fast-faces-detector`, `fast-plates-detector`.

В ответе будет возвращен уникальный ID объекта и перечислены вызываемые обработчики.

### Получение загруженного изображения

Можно получить исходное изображение по его ID.

GET `/v1/processor/sink/{id}`

Параметры

Name | Type | Description
--------------- | --------------- | ---------------
id | String | Идентификатор объекта

### Поиск изображений с объектами

Выполняет поиск изображений, содержащих объекты c указанными параметрами.

POST `/v1/search/objects`

Параметры

Name | Type | Description
--------------- | --------------- | ---------------
label | String | Метка объекта на изображении. Возможны следующие значения: `face`, `plate`.

В ответе прийдет список объектов удовлетворяющих параметрам запроса.

Пример ответа:

```json
[
  {
    "imageId": "ed4bcf88-f321-3757-ac22-362b967081a8",
    "label": "face",
    "geometry": "{335, 896, 164x164}",
    "timestamp": "2022-10-01T21:53:03.302+00:00"
  }
]
```

Поля

Name | Type | Description
--------------- | --------------- | ---------------
imageId | String | Идентификатор изображения
label | String | Метка объекта
geometry | String | Геометрия области в формате { TOP, LEFT, WIDTHxHEIGHT }
timestamp | Timestamp with timezone | Время создания
### Обнаруженные объекты

Можно получить список объектов для конкретного изображения по его ID.

GET `/v1/search/objects/{id}`

Параметры

Name | Type | Description
--------------- | --------------- | ---------------
id | String | Идентификатор объекта

Ответ будет содержать список локализованных объектов. 

Пример ответа:

```json
[
  {
    "imageId": "ed4bcf88-f321-3757-ac22-362b967081a8",
    "label": "face",
    "geometry": "{335, 896, 164x164}",
    "timestamp": "2022-10-01T21:53:03.302+00:00"
  },
  {
    "imageId": "ed4bcf88-f321-3757-ac22-362b967081a8",
    "label": "plate",
    "geometry": "{232, 301, 264x216}",
    "timestamp": "2022-10-01T21:53:03.809+00:00"
  }
]
```

### Отрисовка объектов

Можно выполнить примитивную отрисовку обнаруженных объектов. Будет возвращено изображение с отрисованными рамками вокруг обнаруженных объектов.

POST `/v1/search/display`

Параметры

Name | Type | Description
--------------- | --------------- | ---------------
label | String | Метка объекта на изображении. Возможны следующие значения: `face`, `plate`.
id | String | Идентификатор объекта


## Ограничения

В настоящее время существуют сделующие ограничения:
- Максимальный размер изображения 5МБ
- Доступны базовые обработчики
- Ограничены вычислительные ресурсы
- Данные хранятся не более 1 часа

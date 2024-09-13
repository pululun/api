# Написание контроллеров OpenApi
## Общая информация написания
1. Описание ендпоинта сопровождается стандартом OpenApi, где:
    - Пример написания ендпоинта
    ```
      /api/v1/order/{id}:
         post:
            tags:
               - OrderController
            operationId: createOrder
            requestBody:
               content:
                  application/json:
                     schema:
                        $ref: '../dto/OrderRq.yml#/OrderRq'
            responses:
               "200":
                  description: OK
                  content:
                     application/json:
                        schema:
                        $ref: '../dto/OrderRq.yml#/OrderRq'
               "400":
                  $ref: '../dto/error/400.yml#/400'
               "403":
                  $ref: '../dto/error/403.yml#/403'
               "404":
                  $ref: '../dto/error/404.yml#/404'
         get:
         ...
    ```
   - Строка `/api/v1/order/{id}` отвечает за сам путь ендпоинта
   - Строка `post` отвечает за метод запроса, если методов запроса у ендпоинта несколько, то следующие ендпоинты указываются в данном блоке,
     но начиная с самого метода запроса и с соблюдением табуляции. Как пример предпоследняя строка `get`
   - Строка `tags` отвечает за наименование интерфейса контроллера, который будет создан для реализации в подключаемом классе
   - Строка `- OrderController` само наименование интерфейса контроллера через тире
   - Строка `operationId` отвечает за наименование самого метода для реализации
   - Строка `requestBody` отвечает за принимаемые аргументы в метод, `$ref: '../dto/OrderRq.yml#/OrderRq'` ссылается на файл с описанием дто
   - Строка `responses` отвечает за варианты ответа ендпоинта, в ней описывается все ответы данного ендпоинта, где:
     - `200` - код ответа, 
     - `description` - описание ответа, 
     - `content` - возвращаемый контент, выступающий в качестве возвращающего параметра метода
     - `application/json` тип ответа, 
     - `$ref: '../dto/OrderRq.yml#/OrderRq'` ссылка до дто (вернет саму дто), которую должен вернуть по запросу ендпоинта
2. После написания всех енддпоинтов в классе контроллера, надо указать все пути ендпоинтов в файле `spec.yaml` данного модуля
   - В файле `spec.yaml` в параметрах `paths` указывается путь ендпоинтов и ссылка на контроллер с данными ендпоинтами и указание пути - / через ~1, как на примере ниже
   ```
      /api/v1/order/{id}:
         $ref: "controller/OrderController.yml#/~1api~1v1~1order~1{id}"
   ```
   - Данная запись заходит в описание и собирает все методы запроса данного ендпоинта

# Написание дтошек OpenApi
## Общая информация написания
1. При написании дто следует помнить, что будут сгенерированы все дто участвующие в цепочке, которые присутствуют в дто, которая была указана в контроллере
2. Дто которая не участвует в созданом контроллере для кодогенерации можно добавить в следующее место `spec.yaml`:
   ```
      components:
         schemas:
            OrderRs:
               $ref: "dto/OrderRs.yml#/OrderRs"
            OrderRq:
               $ref: "dto/OrderRq.yml#/OrderRq"
   ```
3. Описание переменных дто сопровождается стандартом OpenApi, где:
   ```
      OrderRs:
         type: object
         properties:
            fieldString:
               type: string
               maxLength: 50
            fieldUUID:
               type: string
               format: uuid
            fieldInteger:
               type: integer
               format: int32
               minimum: 0
               maximum: 10
               default: 1
            fieldLong:
               type: integer
               format: int64
            fieldBoolean:
               type: boolean
            fieldDate:
               type: string
               format: date
            fieldLocalDate:
               type: string
               format: date
            fieldOffsetDateTime:
               type: string
               format: date-time
            fieldListString:
               type: array
               maxItems: 999
               items:
                  type: string (либо указание любого типа или $ref 'ссылка на любую дто', но без type)
            fieldSetString:
               type: array
               uniqueItems: true
               items:
                  type: string
            fieldBigDecimal:
               type: number
   ```
   - Строка `OrderRs` напзвание дто
   - Строка `type: object` отвечает за тип объекта
   - Строка `properties` отвечает за сами переменные в дто
   - Строка `fieldString` и ниже это сами названия переменных
   - Строка `type` в переменных отвечает за установку типа переменной в некоторых случая требуется указать еще `format`
   - Переменным может потребоваться проставить значения по умолчанию, для этого используется `default: "text" / true / 20`
     остальные валидационные параметры рекомендуется указывать через вставку ниже (так как в данном случае возможно полностью настроить все параметры аннотации)
   - Для валидации могут потребоваться некоторые аннотации, ниже представлен способ добавления 1 или более аннотаций для переменных
     `x-field-extra-annotation: " 
     @lombok.Builder.Default @javax.validation.constraints.Pattern(regexp = \**(?i)(asc|desc)&\*)"`
   - Когда создается list/set нужно обязательно данную переменную указывать в теге `required`, в ином случае переменная не будет иметь реализацию ArrayList или HashSet. Пример ниже
   ```
      OrderRq:
         type: object
         required:
            - name
            - deliveryType
            - count
            - messages
         properties:
            name:
               type: string
               maxLength: 50
            deliveryType:
               $ref: 'enum/DeliveryType.yml#/DeliveryType'
            count:
               type: integer
               minimum: 0
               maximum: 10
               default: 1
            messages:
               type: array
               items:
                  type: string
     ```
   - Тэг `required` нужен для создания конструктора с указанными переменными
4. Фрагмент наследования приведен ниже, обязательно нужно соблюдать все табуляции:
    ```
        ExtendsExample:
           allOf:
              - $ref: 'OrderRs.yml#/OrderRs'
              - type: object
                properties:
                   newProperty:
                      type: string
    ```
5. Создание енамок можно осуществить двумя способами:
   - первый способ создания констант в енамке без значений
   ```
      DeliveryType:
         type: string
         enum:
            - PICKUP
            - COURIER_DELIVERY
            - PEDESTALS
   ```
   - второй способ создания констант в енамке с значениями
   ```
      DeliveryType:
         type: string
         enum:
            - САМОВЫВОЗ
            - КУРЬЕРСКАЯ ДОСТАВКА
            - ПОСТАМАТЫ
         x-enum-varnames:
            - PICKUP
            - COURIER_DELIVERY
            - PEDESTALS
   ```
   - Во втором случае будут созданы константы `PICKUP, COURIER_DELIVERY, PEDESTALS` с их указанными значениями в `enum`
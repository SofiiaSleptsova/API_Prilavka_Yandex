# <a name="up" />Проект Яндекс Прилавка (API)

Яндекс Прилавок - это сервис, который предоставляет предпринимателям и компаниям возможность создавать собственные виртуальные магазины в онлайн-пространстве. Этот инструмент облегчает процесс создания и настройки интернет-магазинов, управления продажами и обработки заказов. Он также обеспечивает интеграцию с онлайн-платежами, что делает его полезным для развития электронной коммерции и совершения успешных онлайн-сделок.

## Содержание
- [Задачи тестировщика](#задачи-тестировщика)
- [Требования по проекту](#требования-по-проекту)
- [Инструменты](#инструменты)
- [Проектирование тестовой документации](#проектирование-тестовой-документации)
- [Выполнение тестов](#выполнение-тестов)

## Задачи тестировщика

<details>
<summary> Задачи </summary> 

1. Проанализировать требования к новой функциональности бэкенда Яндекс.Прилавка  
2. Изучить документацию к API в Apidoc  
3. Спроектировать тесты в виде чек-листа, чтобы покрыть новую функциональность  
-Работа с наборами: возможность добавлять продукты в набор    
Ручка **POST /api/v1/kits/:id/products**;   
-Работа с курьерами: возможность проверить, есть ли доставка курьерской службой «Привезём быстро» и сколько она стоит  
Ручка **POST /fast-delivery/v3.1.1/calculate-delivery.xml**;   
-Работа с корзиной:  
возможность получить список продуктов, которые добавили в корзину    
Ручка **GET /api/v1/orders/:id**;  
возможность добавлять продукты в корзину  
Ручка **PUT /api/v1/orders/:id**;  
возможность удалять корзину  
Ручка **DELETE /api/v1/orders/:id**.  
4. Протестировать API через Postman и завести баг-репорты в YouTrack
   
***

</details>

## Требования по проекту

<details>
<summary> Требования к бэкенду приложения Яндекс.Прилавка </summary>

### Описание общей логики
#### Авторизация и данные для заказа
Пользователь может зарегистрироваться. Если пользователь не зарегистрировался, то форма заполнения: имя, e-mail, телефон, адрес, комментарий — появляется, когда пользователь уже сформировал корзину и хочет оформить заказ. Пользователь не может сделать заказ, если не ввёл обязательные поля. Если пользователь зарегистрировался, то ему не нужно вновь вводить данные, однако он может их изменить. Пользователь может оформить несколько заказов.
Для заказа нужно ввести:  
- имя;  
- телефон;  
- адрес;  
- e-mail (необязательно);  
- комментарий к заказу (необязательно).

![iScreen Shoter - Safari - 231105164522](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/e6c1539a-3656-4934-bcb1-787f04cb7a25)

#### Главное меню заказа
На выбор даётся 3 карточки:  
- «Под ситуацию» (под кино и сериалы, на пикник, вкусы Парижа);  
- «Приготовь блюдо» (сырники, борщ, карбонара, штрудель);  
- «Создать свой набор» (пользователь сам называет и добавляет туда продукты).  
Переходишь в карточку — видишь варианты наборов.   
Переходишь в набор — видишь перечень возможных продуктов. Каждый продукт относится к определённой категории (например, «Напитки»). В перечне пользователь видит название продукта, его массу, цену. Когда клиент нажимает на продукт, ему даётся возможность выбрать количество продуктов. При этом появляется кнопка «Корзина», которая отображает сумму выбранных товаров и время доставки.
При нажатии на кнопку пользователь может посмотреть свою корзину.

#### Корзина
Отображает наименование продукта, его количество, цену для этого продукта с учётом количества, итог. Если доставка платная, то отображает сумму доставки и итоговую сумму заказа. Пользователь может удалить корзину, добавить новые продукты, убрать выбранные продукты.
Создание корзины:  
- При создании корзины должна быть возможность указать время доставки продуктов  
- При создании корзины проверять, что все службы доставки могут обработать заказ в указанное время  
- Если время доставки не указано - брать, как текущее время с сервера.  
При удалении и просмотре корзины время доставки не учитывается.  

#### Создание своего набора  
Пользователь может создать свой набор и выбрать продукты. Он обязательно даёт имя набору и выбирает продукты.   Пользователь может изменить название набора, удалить набор, удалить выбранные продукты, добавить новые. Если данные при создании или изменении набора введены неверно — выводится сообщение об ошибке.  
![iScreen Shoter - Safari - 231105164751](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/071491a7-497d-4dd1-bbe8-0354f5542835)

#### Работа с курьерами  
Работа с курьерами предполагает два режима работы:  
1. При оформлении заказа: URL
`POST /api/v1/orders`  
Логика выбора курьерской службы при оформлении заказа пользователем: служба должна работать в указанное в заказе время и должна быть самой дешёвой.   
Пользователю отображается время доставки в зависимости от выбранной службы.  
В заказе доставка становится платной, если соблюдается хотя бы одно условие:   
- превышено максимальное количество товаров;  
- превышен максимальный вес;  
- сумма заказа меньше 150 рублей.  
Логика расчёта стоимости доставки заказа для пользователя:  
- Если вес или количество превысили максимальное значение, стоимость доставки для пользователя становится 99 рублей.  
- Если вес или количество в заказе не превышают максимального, берётся
`price`
этих продуктов, и если их сумма меньше 150 рублей, то стоимость доставки для пользователя также становится 99 рублей.  
Стоимость доставки прибавляется в итоговую сумму заказа.  
Если ни одно из обозначенных условий не соблюдается, то цена доставки курьерской службы не включается в итоговую сумму заказа.   

2. Узнать возможность доставки продуктов и цену для отдельной курьерской службы: у каждой курьерской службы свой URL.   
Цена доставки курьерской службой рассчитывается по количеству и весу указанных продуктов.  
Детальные требования к расчёту стоимости доставки курьерскими службами можно посмотреть [тут](https://code.s3.yandex.net/qa/files/delivery_requirements.pdf).

![iScreen Shoter - Safari - 231105165024](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/0350b548-f19f-4c53-9c17-08ad8d9236a4)

#### Работа со складом
Имеется 4 складских отделения.   
У каждого склада своя ручка. 
У каждого свой ограниченный набор продуктов. Когда пользователь сделал заказ, ручка уточняет, какой склад сформирует заказ.  
Логика выбора склада: есть продукты на складе, должен работать во время заказа и самый дешёвый.   
Пользователь может заказать только те продукты и их количество, которые есть в полной мере хотя бы на одном складе (то есть ситуации, где он набрал корзину, а ему пишут «Не привезём» — нет).  

![iScreen Shoter - Safari - 231105165121](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/2cc5b6a3-6afc-4717-abf3-16f23a1fea93)

#### Список URL реализованных в API
Подробнее о самих URL и параметрах смотри в документации к API

#### URL для авторизации
- `POST /api/v1/users - создать пользователя`

#### URL для наборов
- `POST /api/v1/kits - создать набор`
- `GET /api/v1/kits - получить список наборов`
- `DELETE /api/v1/kits - удалить набор`
- `PUT /api/v1/kits - переименовать набор, изменить список продуктов в наборе`
- `GET /api/v1/kits/search - получить список продуктов в наборе`

#### URL для продуктов
- `POST /api/v1/products/kits - получить список наборов по продуктам`
- `POST /api/v1/kits/{id}/products - добавить продукты в набор`
- `PUT /api/v1/products/:id - изменить цену продукта`
- `POST /api/v1/orders - посчитать сумму продуктов`
- `POST /api/v1/warehouse/check - проверить наличие продуктов на складах`

#### URL для складов
- `GET /api/v1/warehouses - получить список складов`
- `POST /api/v1/warehouses/amount - получить информацию о количестве продуктов на складах`
- `/api/wsdl - получить информацию о количестве продуктов на складах`
- `POST /api/v1/orders - получить информацию, какой склад возьмет заказ`

#### URL для курьерских служб
- `GET /api/v1/couriers - получить список курьерских служб`
- `POST /api/v1/couriers/check - узнать информацию доступна ли курьерская служба для доставки заказа`
- `POST /api/v1/orders - узнать информацию, какой курьер возьмет заказ`
- `POST /moscow-delivery/v1/calculate - возможность доставки и её стоимость курьерской «Доставка Москва»`
- `POST /on-a-broomstick/v1/delivery - возможность доставки и её стоимость курьерской службой «На метле уюта»`
- `POST /fast-delivery/v3.1.1/calculate-delivery.xml - возможность доставки и её стоимость` `курьерской службой «Привезём быстро»`
- `POST /train-noises/wsdl - возможность доставки и её стоимость курьерской службой «Чух-чух и уже у вас»`

#### URL для заказов
- `POST /api/v1/orders - посчитать итоговую сумму заказа (вместе с доставкой)`
- `POST /api/v1/orders - посчитать стоимость доставки с учётом различных курьерских служб`
- `GET /api/v1/orders - получить список заказов`

#### URL для корзины
- `GET /api/v1/orders/id - получить список продуктов в корзине`
- `POST /api/v1/orders - создать корзину`
- `PUT /api/v1/orders/:id - добавить продукты в корзину`
- `DELETE/api/v1/orders/:id - удалить корзину`

#### Описание содержимого базы данных

![iScreen Shoter - Safari - 231105165304](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/4f6bae4e-5441-4085-a3b9-159cde3ba1ef)

![iScreen Shoter - Safari - 231105165323](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/b4fc893b-7071-4f9e-8bb2-ab2560b849db)

![iScreen Shoter - Safari - 231105165343](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/82537bfe-da23-4fe0-a9fe-af60e707bd0f)

![iScreen Shoter - Safari - 231105165400](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/3d9abe46-712f-4248-9b2c-e8ed030cba1c)

![iScreen Shoter - Safari - 231105165417](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/a0937e31-10f9-415c-88ac-455f2178aac2)

![iScreen Shoter - Safari - 231105165435](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/8dc37c42-48c9-41fc-9f01-409b93e9ac7d)

![iScreen Shoter - Safari - 231105165450](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/fd201b63-e4b1-46b6-be01-630cbbe360b7)

***

</details>

<details>
<summary> API Яндекс.Прилавка </summary>

### API Яндекс.Прилавок 3.1.1 

### Warehouses
**Warehouses - [SOAP] Проверить количество товаров на складах**
**POST**
`/api/wsdl`  
Параметр:  
| Название | Тип | Описание |
| ------------------- | ------------------- | ------------------- |
| ids       | number[]      | Массив идентификаторов продуктов (после id в таблице product_model).       |

[XML] Проверяем количество товаров
```
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="WebServices.MainWsdl">
    <soap:Body>
        <Request xmlns="WebServices.MainWsdl">
            <ids>1</ids>
            <ids>4</ids>
            <ids>44</ids>
        </Request>
    </soap:Body>
</soap:Envelope>
```

Ответ: На каком складе, что есть и сколько
```
    HTTP/1.1 200 OK
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="WebServices.MainWsdl">
    <soap:Body>
        <Response>
            <name>Имеется всё</name>
            <products>
                <name>Sprite классический</name>
                <quantity>9</quantity>
            </products>
            <products>
                <name>Чипсы кукурузные классик Salto Nachos</name>
                <quantity>6</quantity>
            </products>
        </Response>
        <Response>
            <name>Чердак</name>
            <products>
                <name>Сок Jumex апельсин без сахара</name>
                <quantity>3</quantity>
            </products>
            <products>
                <name>Sprite классический</name>
                <quantity>12</quantity>
            </products>
        </Response>
        <Response>
            <name>Большой мир</name>
            <products>
                <name>Сок Jumex апельсин без сахара</name>
                <quantity>1</quantity>
            </products>
        </Response>
        <Response>
            <name>Шведский дом</name>
            <products>
                <name>Сок Jumex апельсин без сахара</name>
                <quantity>3</quantity>
            </products>
            <products>
                <name>Sprite классический</name>
                <quantity>12</quantity>
            </products>
        </Response>
    </soap:Body>
</soap:Envelope>
```

**Warehouses - Получить список складов**
**GET**
`/api/v1/warehouses`  

Ответ: Успешное получение складов
```
HTTP/1.1 200 OK
[
    {
           "name": "Имеется всё",
           "workingHours": {
               "start": 7,
               "end": 23
           }
       },
    {
           "name": "Шведский дом",
           "workingHours": {
               "start": 8,
               "end": 23
           }
       },
    {
           "name": "Чердак",
           "workingHours": {
               "start": 8,
               "end": 21
           }
       },
    {
           "name": "Большой мир",
           "workingHours": {
               "start": 5,
               "end": 20
           }
       }
    ]
```

**Warehouses - Проверить количество товаров на складах**  
Версия этого эндпоинта для SOAP называется - [SOAP] Проверить количество товаров на складах  
**POST**
`/api/v1/warehouses/amount`  

[JSON] Пример заголовков
```
{
    "Content-Type": "application/json"
}
```
[XML] Пример заголовков
```
{
    "Content-Type": "application/xml"
}
```

Параметр:  
| Название | Тип | Описание |
| ------------------- | ------------------- | ------------------- |
| ids       | number[]      | Массив идентификаторов продуктов (после id в таблице product_model).       |
| dataType      | string      | Формат входных данных. Может принимать значения: "json" - тело запроса ожидается в JSON-формате "xml" - тело запроса ожидается в XML-формате По умолчанию: json     |

[JSON] Проверяем количество товаров
```
{
    "ids": [
        1,
        4,
        44
    ]
}
```
[XML] Проверяем количество товаров
```
<root>
    <id>1</id>
    <id>4</id>
    <id>44</id>
</root>
```

Ответ: На каком складе, что есть и сколько
```
HTTP/1.1 200 OK
{
       "Имеется всё": {
           "Sprite классический": 9,
           "Чипсы кукурузные классик Salto Nachos": 6
       },
       "Чердак": {
           "Сок Jumex апельсин без сахара": 3,
           "Sprite классический": 12
       },
       "Большой мир": {
           "Сок Jumex апельсин без сахара": 1
       },
       "Шведский дом": {
           "Сок Jumex апельсин без сахара": 3,
           "Sprite классический": 12
       }
   }
```





























***

</details>

## Инструменты
<p align="left"> 
  <a href="https://docs.google.com/" target="_blank" rel="noreferrer"><img src="https://w7.pngwing.com/pngs/240/1015/png-transparent-g-suite-google-docs-google-angle-rectangle-logo.png" width="36" height="36" alt="Google Sheets" /></a>
  <a href="https://www.figma.com/" target="_blank" rel="noreferrer"><img src="https://raw.githubusercontent.com/danielcranney/readme-generator/main/public/icons/skills/figma-colored.svg" width="36" height="36" alt="Figma" /></a>
  <a href="https://www.jetbrains.com/youtrack/" target="_blank" rel="noreferrer"><img src="https://upload.wikimedia.org/wikipedia/commons/9/95/YouTrack_Icon.png" width="36" height="36" alt="Youtrack" /></a>
 <a href="https://www.postman.com/" target="_blank" rel="noreferrer"><img src="https://seeklogo.com/images/P/postman-logo-0087CA0D15-seeklogo.com.png" title="postman" width="36" height="36" alt="Postman" /></a>
</p> 


## Процесс работы
### Проектирование тестовой документации
![Чек-лист_ API](https://github.com/SofiiaSleptsova/Yandex_Prilavka/assets/147629405/09c1aadc-212d-4f6b-9164-6c0c1ead372f)

### Выполнение тестов

[Тестовая документация с кликабельными ссылками на баг-репорты](https://docs.google.com/spreadsheets/d/1y_dVZCaKWYKP17JVRHCYdO9HxVyHSlECJB6-uux_4oA/edit?usp=sharing)

<details>
<summary> Баг-репорты </summary>

#### Работа с наборами

<details>
<summary>ID: 683-86 </summary>

### При добавлении продукта в набор, введение id набора в дробном типе - продукт добавлен, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-86](https://slepsovasonya.youtrack.cloud/issue/683-86/Pri-dobavlenii-produkta-v-nabor-vvedenie-id-nabora-v-drobnom-tipe-produkt-dobavlen-kod-i-status-200-OK-ruchka-POST-api-v1-kits)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать id набора в дробном типе  
**Path Variables=8.0**  
В теле указать:  
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8.0/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и стаутус 404 Not Found, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, в набор продукт добавлен
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "id": 7,
            "name": "Чипсы Lay's картофельные Лобстер рифленые",
            "price": 119,
            "weight": 150,
            "units": "г",
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-65 </summary>

###  При добавлении продукта в набор, введение id набора в вещественном типе - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-65](https://slepsovasonya.youtrack.cloud/issue/683-65)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать id набора в дробном типе  
**Path Variables=8.5**  
В теле указать:  
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8.5/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и стаутус 404 Not Found, в набор продукт не добавлен    
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-102 </summary>

###  При добавлении продукта в набор, изменение типа данных productlist на словарь - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-102](https://slepsovasonya.youtrack.cloud/issue/683-102)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле поменять тип данных на словарь "productlist":  
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и стаутус 404 Not Found, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-103 </summary>

###  При добавлении продукта в набор, если убрать значение и ключ productlist - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-103](https://slepsovasonya.youtrack.cloud/issue/683-103)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле убрать значение и ключ "productlist":    
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-104 </summary>

###  При добавлении продукта в набор, если productlist пустой - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-104](https://slepsovasonya.youtrack.cloud/issue/683-104)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле оставить пустым "productlist":     
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": []
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-106 </summary>

###  При добавлении продукта в набор, если productlist пустой (null) - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-106](https://slepsovasonya.youtrack.cloud/issue/683-106/Pri-dobavlenii-produkta-v-nabor-esli-productlist-pustoj-null-oshibka-na-storone-servera-500-Internal-Server-Error-ruchka-POST)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле оставить пустым "productlist": null:      
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": null
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-10 </summary>

###  При добавлении в набор продукта, с НЕсуществующим id продукта - продукт добавляется без id, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-10](https://slepsovasonya.youtrack.cloud/issue/683-10/Pri-dobavlenii-v-nabor-produkta-s-NEsushestvuyushim-id-produkta-produkt-dobavlyaetsya-bez-id-kod-i-status-200-OK-ruchka-POST-api)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id НЕсуществующего продукта:       
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 170,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, несуществующий продукт добавлен в набор без id, только с количеством  
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-57 </summary>

###  При добавлении в набор продукта, введение id продукта как строки - продукт добавляется без id, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-57](https://slepsovasonya.youtrack.cloud/issue/683-57/Pri-dobavlenii-v-nabor-produkta-vvedenie-id-produkta-kak-stroki-produkt-dobavlyaetsya-bez-id-kod-i-status-200-OK-ruchka-POST-api)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id продукта как строку:       
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": "7",
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**   
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, несуществующий продукт добавлен в набор без id, только с количеством  
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-87 </summary>

###  При добавлении в набор продукта, введение id продукта в дробном типе - продукт добавлен в набор, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-87](https://slepsovasonya.youtrack.cloud/issue/683-87)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id продукта в дробном типе:        
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7.0,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, несуществующий продукт добавлен в набор без id, только с количеством  
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "id": 7,
            "name": "Чипсы Lay's картофельные Лобстер рифленые",
            "price": 119,
            "weight": 150,
            "units": "г",
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Серьезная

***
</details>

<details>
<summary>ID: 683-56 </summary>

###  При добавлении в набор продукта, введение id продукта в вещественном типе - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-56](https://slepsovasonya.youtrack.cloud/issue/683-56/Pri-dobavlenii-v-nabor-produkta-vvedenie-id-produkta-v-veshestvennom-tipe-oshibka-na-storone-servera-500-Internal-Server-Error)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id продукта в вещественном типе:          
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7.5,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Серьезная

***
</details>

<details>
<summary>ID: 683-58 </summary>

###  При добавлении в набор продукта, введение id продукта в отрицательном типе - продукт добавляется без id, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-58](https://slepsovasonya.youtrack.cloud/issue/683-58/Pri-dobavlenii-v-nabor-produkta-vvedenie-id-produkta-v-otricatelnom-tipe-produkt-dobavlyaetsya-bez-id-kod-i-status-200-OK-ruchka)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id продукта в отрицательном типе:           
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": -7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в набор без id, только с количеством
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-12 </summary>

###  При добавлении в набор продукта, введение id продукта в не числовом типе - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-12](https://slepsovasonya.youtrack.cloud/issue/683-12/Pri-dobavlenii-v-nabor-produkta-vvedenie-id-produkta-v-ne-chislovom-tipe-oshibka-na-storone-servera-500-Internal-Server-Error)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id продукта в не числовом типе:            
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": "%",
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-55 </summary>

###  При добавлении в набор продукта, отсутствие ключа и значения id продукта - продукт добавляется без id, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-55](https://slepsovasonya.youtrack.cloud/issue/683-55/Pri-dobavlenii-v-nabor-produkta-otsutstvie-klyucha-i-znacheniya-id-produkta-produkt-dobavlyaetsya-bez-id-kod-i-status-200-OK)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле не указать ключ и значение id продукта:             
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в набор без id, только с количеством
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-14 </summary>

### При добавлении в набор продукта, пустой id продукта - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-14](https://slepsovasonya.youtrack.cloud/issue/683-14/Pri-dobavlenii-v-nabor-produkta-pustoj-id-produkta-oshibka-na-storone-servera-500-Internal-Server-Error-ruchka-POST-api-v1-kits)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле не указывать id продукта:              
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": "",
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-59 </summary>

### При добавлении в набор продукта, введение id=null продукта - продукт добавляется без id, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-59](https://slepsovasonya.youtrack.cloud/issue/683-59/Pri-dobavlenii-v-nabor-produkta-vvedenie-idnull-produkta-produkt-dobavlyaetsya-bez-id-kod-i-status-200-OK-ruchka-POST-api-v1)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id продукта null:                
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": null,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в набор без id, только с количеством
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Критическая

***
</details>


<details>
<summary>ID: 683-15 </summary>

### При добавлении в набор продукта, введение quantity=0 продукта - продукт добавляется в кол-ве 0 штук, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-15](https://slepsovasonya.youtrack.cloud/issue/683-15/Pri-dobavlenii-v-nabor-produkta-vvedenie-quantity0-produkta-produkt-dobavlyaetsya-v-kol-ve-0-shtuk-kod-i-status-200-OK-ruchka)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать quantity=0:               
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 0
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, продукт добавлен в набор в количестве 0 штук  
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "id": 7,
            "name": "Чипсы Lay's картофельные Лобстер рифленые",
            "price": 119,
            "weight": 150,
            "units": "г",
            "quantity": 0
        }
    ],
    "productsCount": 0
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-60 </summary>

### При добавлении в набор продукта, введение quantity в в иде строки - продукт добавляется в строчном значении, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-60](https://slepsovasonya.youtrack.cloud/issue/683-60/Pri-dobavlenii-v-nabor-produkta-vvedenie-quantity-v-v-ide-stroki-produkt-dobavlyaetsya-v-strochnom-znachenii-kod-i-status-200-OK)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать quantity в виде строки:                 
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": "3"
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, продукт в набор добавляется в строчном значении
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "id": 7,
            "name": "Чипсы Lay's картофельные Лобстер рифленые",
            "price": 119,
            "weight": 150,
            "units": "г",
            "quantity": "3"
        }
    ],
    "productsCount": "03"
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-61 </summary>

### При добавлении продукта в набор, введение quantity в дробном типе - продукт добавлен, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-61](https://slepsovasonya.youtrack.cloud/issue/683-61/Pri-dobavlenii-produkta-v-nabor-vvedenie-quantity-v-drobnom-tipe-produkt-dobavlen-kod-i-status-200-OK-ruchka-POST-api-v1-kits-id)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать quantity в дробном типе:                 
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3.0
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, в набор продукт добавлен  
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "id": 7,
            "name": "Чипсы Lay's картофельные Лобстер рифленые",
            "price": 119,
            "weight": 150,
            "units": "г",
            "quantity": 3
        }
    ],
    "productsCount": 3
}
```

**Приоритет:**   
Серьезная

***
</details>

<details>
<summary>ID: 683-88 </summary>

### При добавлении продукта в набор, введение quantity в вещественном типе - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-88](https://slepsovasonya.youtrack.cloud/issue/683-88/Pri-dobavlenii-produkta-v-nabor-vvedenie-quantity-v-veshestvennom-tipe-oshibka-na-storone-servera-500-Internal-Server-Error)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать quantity в дробном типе:                   
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3.5
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
ФР: В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-62 </summary>

### При добавлении продукта в набор, quantity в отрицательном типе - кол-во продуктов стремится к отрицательным значениям, код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-62](https://slepsovasonya.youtrack.cloud/issue/683-62/Pri-dobavlenii-produkta-v-nabor-quantity-v-otricatelnom-tipe-kol-vo-produktov-stremitsya-k-otricatelnym-znacheniyam-kod-i-status)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать id в отрицательном типе:                   
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": -3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, количество продуктов уменьшается на значение quantity
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "id": 7,
            "name": "Чипсы Lay's картофельные Лобстер рифленые",
            "price": 119,
            "weight": 150,
            "units": "г",
            "quantity": -3
        }
    ],
    "productsCount": -3
}
```

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-16 </summary>

### При добавлении в набор продукта, введение quantity в не числовом типе - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-16](https://slepsovasonya.youtrack.cloud/issue/683-16/Pri-dobavlenii-v-nabor-produkta-vvedenie-quantity-v-ne-chislovom-tipe-oshibka-na-storone-servera-500-Internal-Server-Error)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать quantity продукта в не числовом типе:                    
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": "%"
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-63 </summary>

### При добавлении продукта в набор, отсутствие значение и ключа quantity - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-63](https://slepsovasonya.youtrack.cloud/issue/683-63/Pri-dobavlenii-produkta-v-nabor-otsutstvie-znachenie-i-klyucha-quantity-oshibka-na-storone-servera-500-Internal-Server-Error)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле убрать значение и ключ qunatity:                     
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-18 </summary>

### При добавлении в набор продукта, с пустым quantity - продукт добавляется с quantity= "", код и статус 200 ОК (ручка POST /api/v1/kits/:id/products) [683-18](https://slepsovasonya.youtrack.cloud/issue/683-18/Pri-dobavlenii-v-nabor-produkta-s-pustym-quantity-produkt-dobavlyaetsya-s-quantity-kod-i-status-200-OK-ruchka-POST-api-v1-kits)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать не указывать quantity:                     
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": ""
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, quantity указан как ""
```
{
    "id": 8,
    "name": "Мой набор",
    "productsList": [
        {
            "id": 7,
            "name": "Чипсы Lay's картофельные Лобстер рифленые",
            "price": 119,
            "weight": 150,
            "units": "г",
            "quantity": ""
        }
    ],
    "productsCount": "0"
}

```

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-64 </summary>

### При добавлении продукта в набор, отсутствие quantity=null - ошибка на стороне сервера 500 Internal Server Error (ручка POST /api/v1/kits/:id/products) [683-64](https://slepsovasonya.youtrack.cloud/issue/683-64)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на добавление продуктов в набор **/api/v1/kits/:id/products**  
В URL указать существующий id набора  
В теле указать quantity=null:                      
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/kits/8/products' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": null
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в набор продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Неотложная  

***
</details>

**Работа с курьерами**

<details>
<summary>ID: 683-22 </summary>

### При запросе информации о доставке, с количеством продукта 0 - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-22](https://slepsovasonya.youtrack.cloud/issue/683-22/Pri-zaprose-informacii-o-dostavke-s-kolichestvom-produkta-0-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka-POST-fast)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**   
В теле указать productsCount=0:                      
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>0</productsCount>
    <productsWeight>1</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе код и стаус 200 ОК, доставка рассчитана  
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-66 </summary>

### При запросе информации о доставке, с количеством продукта в дробном типе - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-66](https://slepsovasonya.youtrack.cloud/issue/683-66)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать productsCount=3.5:                   
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3.5</productsCount>
    <productsWeight>1</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе код и стаус 200 ОК, доставка рассчитана  
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-67 </summary>

### При запросе информации о доставке, с количеством продукта в отрицательном типе - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-67](https://slepsovasonya.youtrack.cloud/issue/683-67)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать productsCount=3:                   
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>-3</productsCount>
    <productsWeight>1</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе код и стаус 200 ОК, доставка рассчитана  
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-23 </summary>

### При запросе информации о доставке, с не числовыми типом в количестве продуктов - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-23](https://slepsovasonya.youtrack.cloud/issue/683-23/Pri-zaprose-informacii-o-dostavke-s-ne-chislovymi-tipom-v-kolichestve-produktov-dostavka-rasschityvaetsya-kod-i-status-200-OK)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать:                    
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>%</productsCount>
    <productsWeight>1</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе код и стаус 200 ОК, доставка рассчитана  
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-68 </summary>

### При запросе информации о доставке, если убрать <productsCount> - ошибка на стороне сервера 500 Internal Server Error (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-68](https://slepsovasonya.youtrack.cloud/issue/683-68)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле убрать ключ и значение productsCount:                     
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsWeight>1</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-25 </summary>

### При запросе информации о доставке, с пустым в количеством продуктов - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-25](https://slepsovasonya.youtrack.cloud/issue/683-25/Pri-zaprose-informacii-o-dostavke-s-pustym-v-kolichestvom-produktov-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka-POST)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле оставить пустым productsCount:                      
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount/>
    <productsWeight>1</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе код и статус 200 ОК, доставка рассчитана
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-26 </summary>

### При запросе информации о доставке, со значением 0 в весе продуктов - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-26](https://slepsovasonya.youtrack.cloud/issue/683-26/Pri-zaprose-informacii-o-dostavke-so-znacheniem-0-v-vese-produktov-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka-POST)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать productsWeight=0:                        
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>0</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается    
**Фактический результат:**       
В ответе код и статус 200 ОК, доставка рассчитана
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-69 </summary>

### При запросе информации о доставке, с отрицательным типом в весе продуктов - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-69](https://slepsovasonya.youtrack.cloud/issue/683-69)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать productsWeight -3:                         
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>1</productsCount>
    <productsWeight>-3</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается    
**Фактический результат:**       
В ответе код и статус 200 ОК, доставка рассчитана
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-27 </summary>

### При запросе информации о доставке, с не числовыми типом в весе продуктов - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-27](https://slepsovasonya.youtrack.cloud/issue/683-27/Pri-zaprose-informacii-o-dostavke-s-ne-chislovymi-tipom-v-vese-produktov-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать productsWeight=%:                          
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>%</productsWeight>
    <deliveryTime>19</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается    
**Фактический результат:**       
В ответе код и статус 200 ОК, доставка рассчитана
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-70 </summary>

### При запросе информации о доставке, если убрать ключ и значение веса продуктов - ошибка на стороне сервера 500 Internal Server Error (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml [683-70](https://slepsovasonya.youtrack.cloud/issue/683-70)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле убрать ключ и значение productsWeight:                           
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <deliveryTime>19</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается    
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-29 </summary>

### При запросе информации о доставке, с пустым весом продуктов - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-29](https://slepsovasonya.youtrack.cloud/issue/683-29/Pri-zaprose-informacii-o-dostavke-s-pustym-vesom-produktov-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka-POST-fast)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать productsWeight оставить пустым:                          
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight/>
    <deliveryTime>19</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается    
**Фактический результат:**       
В ответе код 200 ОК, доставка рассчитана    
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="0">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-30 </summary>

### При запросе информации о доставке, в нерабочее время - в ответе нет информации "false" (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-30](https://slepsovasonya.youtrack.cloud/issue/683-30/Pri-zaprose-informacii-o-dostavke-v-nerabochee-vremya-v-otvete-net-informacii-false-ruchka-POST-fast-delivery-v3.1.1-calculate)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать deliveryTime=23:                          
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>5</productsWeight>
    <deliveryTime>23</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код 200 ОК, информация о невозможности доставки  
```
<response name="Привезём быстро" isItPossibleToDeliver="false" 
```
**Фактический результат:**       
В ответе код 200 ОК, информация о невозможности доставки отстутвует  
```
<response name="Привезём быстро" 
```

**Приоритет:**   
Серьезная

***
</details>

<details>
<summary>ID: 683-71 </summary>

### При запросе информации о доставке, если число не входит в диапазон часов - приходит название доставки, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-71](https://slepsovasonya.youtrack.cloud/issue/683-71)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать deliveryTime=26:                           
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>5</productsWeight>
    <deliveryTime>26</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается  
**Фактический результат:**       
В ответе код и статус 200 ОК  
```
<response name="Привезём быстро" 
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-72 </summary>

### При запросе информации о доставке, если отрицательный тип во времени - приходит название доставки, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-72](https://slepsovasonya.youtrack.cloud/issue/683-72)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать:                             
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>5</productsWeight>
    <deliveryTime>-19</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается  
**Фактический результат:**       
В ответе код и статус 200 ОК  
```
<response name="Привезём быстро" 
```

**Приоритет:**   
Серьезная

***
</details>

<details>
<summary>ID: 683-31 </summary>

### При запросе информации о доставке, с не числовыми типом во времени - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-31](https://slepsovasonya.youtrack.cloud/issue/683-31/Pri-zaprose-informacii-o-dostavke-s-ne-chislovymi-tipom-vo-vremeni-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka-POST)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать:                             
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>5</productsWeight>
    <deliveryTime>one</deliveryTime>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена   
**Фактический результат:**       
В ответе код 200 ОК, доставка рассчитана  
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="23" clientDeliveryCost="00">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-73 </summary>

### При запросе информации о доставке, в если убрать ключ и значение о времени - ошибка на стороне сервера 500 Internal Server Error (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-73](https://slepsovasonya.youtrack.cloud/issue/683-73)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле указать убрать ключ и значение deliveryTime:                              
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/fast-delivery/v3.1.1/calculate-delivery.xml' \
--header 'Content-Type: application/xml' \
--data '<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>5</productsWeight>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, доставка не рассчитывается  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-33 </summary>

### При запросе информации о доставке, с пустым временем - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-33](https://slepsovasonya.youtrack.cloud/issue/683-33/Pri-zaprose-informacii-o-dostavke-s-pustym-vremenem-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka-POST-fast-delivery)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**  
В теле deliveryTime указать пустым:                               
```
<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>5</productsWeight>
    <deliveryTime/>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе код и статус 200 ОК, доставка рассчитана  
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="43" clientDeliveryCost="99">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-33 </summary>

### При запросе информации о доставке, с пустым временем - доставка рассчитывается, код и статус 200 ОК (ручка POST /fast-delivery/v3.1.1/calculate-delivery.xml) [683-33](https://slepsovasonya.youtrack.cloud/issue/683-33/Pri-zaprose-informacii-o-dostavke-s-pustym-vremenem-dostavka-rasschityvaetsya-kod-i-status-200-OK-ruchka-POST-fast-delivery)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **POST** на проверку, наличия доставки курьерской службой «Привезём быстро» и получения информации о стоимости **/fast-delivery/v3.1.1/calculate-delivery.xml**    
В теле deliveryTime указать пустым:                               
```
<InputModel>
    <productsCount>3</productsCount>
    <productsWeight>5</productsWeight>
    <deliveryTime/>
</InputModel>
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, информация не получена  
**Фактический результат:**       
В ответе код и статус 200 ОК, доставка рассчитана  
```
<response name="Привезём быстро" isItPossibleToDeliver="true" hostDeliveryCost="43" clientDeliveryCost="99">
    <toBeDeliveredTime>
        <min>25</min>
        <max>30</max>
    </toBeDeliveredTime>
</response>
```

**Приоритет:**   
Критическая

***
</details>

**Работа с корзиной**

<details>
<summary>ID: 683-90 </summary>

### При запросе списка продуктов в корзине, введение id корзины в дробном типе - список получен, код и статус 200 ОК (ручка GET /api/v1/orders/:id) [683-90](https://slepsovasonya.youtrack.cloud/issue/683-90)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **GET** для получение списка продуктов в корзине **/api/v1/orders/:id**  
В URL указать id корзины в дробном типе:  
Path Variables=2.0                             
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2.0'
```

**Ожидаемый результат:**  
В ответе код и статус 404 Not Found, список продуктов в корзине не получен  
**Фактический результат:**       
В ответе код и статус 200 ОК, список продуктов в корзине получен  
```
[
    {
        "id": 1,
        "name": "Сок Jumex апельсин без сахара",
        "price": 149,
        "weight": 473,
        "units": "мл",
        "quantity": 3
    }
]
```

**Приоритет:**   
Серьезная

***
</details>

<details>
<summary>ID: 683-74 </summary>

### При запросе списка продуктов в корзине, введение id корзины в вещественном типе - ошибка на стороне сервера 500 Internal Server Error (ручка GET /api/v1/orders/:id) [683-74](https://slepsovasonya.youtrack.cloud/issue/683-74)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **GET** для получение списка продуктов в корзине **/api/v1/orders/:id**  
В URL указать id корзины в дробном типе:  
Path Variables=2.5                               
```
curl --location 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2.5'
```

**Ожидаемый результат:**  
В ответе код и статус 404 Not Found, список продуктов в корзине не получен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-93 </summary>

### При добавлении продукта в корзину, при введении корректных значений - нарушена структура ответа, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-93](https://slepsovasonya.youtrack.cloud/issue/683-93)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id***  
В URL указать   
Path Variables=2  
В теле указать:                                
```
curl --location --request PUT 'https://cf40f600-71ef-4f12-956f-9a4d03059467.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 200 ОК, тело ответа выглядит:  
```
{
"productsList": [
    {
        "id": 1,
        "quantity": 3
    },
    {
        "id": 7,
        "quantity": 13
    }
],
        "status": 0,
        "deliveryPriceOur": 30,
        "deliveryTime": "25~30",
        "courierService": "Привезём быстро",
        "deliveryPrice": 0,
        "wareHouse": "Шведский дом",
        "userId": null,
        "id": 5,
        "productsCost": 804,
        "finalCCost": 804
    }
```
**Фактический результат:**       
В ответе код и статус 200 ОК, тело ответа выглядит:
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 1,
            "quantity": 3
        },
        {
            "id": 7,
            "quantity": 3
        }
    ],
    "productsCost": 804,
    "finalCost": 804
}
```

**Приоритет:**   
Обычная

***
</details>

<details>
<summary>ID: 683-75 </summary>

### При добавлении продукта в корзину, введение id корзины в дробном типе - продукт добавлен в корзину, код и статус 200 ОК (запрос PUT /api/v1/orders/:id) [683-75](https://slepsovasonya.youtrack.cloud/issue/683-75/Pri-dobavlenii-produkta-v-korzinu-vvedenie-id-korziny-v-drobnom-tipe-produkt-dobavlen-v-korzinu-kod-i-status-200-OK-zapros-PUT)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**
В URL указать  
Path Variables=2.0    
В теле указать:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2.0' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 404 Not Found, список продуктов в корзине не получен  
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в корзину  
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 1,
            "quantity": 3
        },
        {
            "id": 7,
            "quantity": 3
        }
    ],
    "productsCost": 804,
    "finalCost": 804
}
```

**Приоритет:**   
Критическая  

***
</details>

<details>
<summary>ID: 683-94 </summary>

### При добавлении продукта в корзину, введение id корзины в вещественном типе - ошибка на стороне сервера 500 Internal Server Error (запрос PUT /api/v1/orders/:id) [683-94](https://slepsovasonya.youtrack.cloud/issue/683-94)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**
В URL указать  
Path Variables=2.5      
В теле указать:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2.5' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 404 Not Found, список продуктов в корзине не получен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-105 </summary>

### При добавлении продукта в корзину, изменение типа данных productlist на словарь - ошибка на стороне сервера 500 Internal Server Error (ручка PUT /api/v1/orders/:id) [683-105](https://slepsovasonya.youtrack.cloud/issue/683-105)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать существующий id корзины  
В теле поменять тип данных "productlist" на словарь:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": {
       "productsList1": {
            "id": 7,
            "quantity": 3
        }
    }
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-107 </summary>

### При добавлении продукта в корзину, если убрать ключ и значение productlist - приходит список корзины, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-107](https://slepsovasonya.youtrack.cloud/issue/683-107)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать существующий id корзины  
В теле убрать ключ и значение productlist :                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, приходит список корзины
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 5,
            "quantity": 1
        }
    ],
    "productsCost": 349,
    "finalCost": 349
}
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-108 </summary>

### При добавлении продукта в корзину, если productlist пустой - приходит список корзины, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-108](https://slepsovasonya.youtrack.cloud/issue/683-108)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать существующий id корзины  
В теле productlist оставить пустым :                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": []
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, приходит список корзины  
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 5,
            "quantity": 1
        }
    ],
    "productsCost": 349,
    "finalCost": 349
 }
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-109 </summary>

### При добавлении продукта в корзину, если productlist пустой (null) - приходит список корзины, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-109](https://slepsovasonya.youtrack.cloud/issue/683-109)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать существующий id корзины  
В теле productlist оставить пустым (null) :                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": null
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, приходит список корзины  
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 5,
            "quantity": 1
        }
    ],
    "productsCost": 349,
    "finalCost": 349
 }
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-93 </summary>

### При добавлении продукта в корзину, при введении корректных значений - нарушена структура ответа, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-93](https://slepsovasonya.youtrack.cloud/issue/683-93)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id корзины  
Path Variables=2  
В теле указать:                                 
```
curl --location --request PUT 'https://cf40f600-71ef-4f12-956f-9a4d03059467.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 200 ОК, тело ответа выглядит:  
```
{
"productsList": [
    {
        "id": 1,
        "quantity": 3
    },
    {
        "id": 7,
        "quantity": 13
    }
],
        "status": 0,
        "deliveryPriceOur": 30,
        "deliveryTime": "25~30",
        "courierService": "Привезём быстро",
        "deliveryPrice": 0,
        "wareHouse": "Шведский дом",
        "userId": null,
        "id": 5,
        "productsCost": 804,
        "finalCCost": 804
    }
```
**Фактический результат:**       
В ответе код и статус 200 ОК, тело ответа выглядит:    
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 1,
            "quantity": 3
        },
        {
            "id": 7,
            "quantity": 3
        }
    ],
    "productsCost": 804,
    "finalCost": 804
}
```

**Приоритет:**   
Обычная

***
</details>

<details>
<summary>ID: 683-76 </summary>

### При добавлении продуктов в корзину, введение id продукта в виде строки - продукт добавляется в строчном значении (ручка PUT /api/v1/orders/:id) [683-76](https://slepsovasonya.youtrack.cloud/issue/683-76/Pri-dobavlenii-produktov-v-korzinu-vvedenie-id-produkta-v-vide-stroki-produkt-dobavlyaetsya-v-strochnom-znachenii-ruchka-PUT-api)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": "7",
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, id продукта добавлен в строчном виде  
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 31,
    "deliveryTime": "25~30",
    "wareHouse": "Чердак",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": "7",
            "quantity": 3
        }
    ],
    "productsCost": 0,
    "finalCost": 99
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-91 </summary>

### При добавлении продуктов в корзину, введение id продукта в дробном типе - продукт добавлен в корзину, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-91](https://slepsovasonya.youtrack.cloud/issue/683-91)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                    
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7.0,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен   
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в корзину   
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 1,
            "quantity": 3
        },
        {
            "id": 7,
            "quantity": 3
        }
    ],
    "productsCost": 804,
    "finalCost": 804
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-77 </summary>

### При добавлении продуктов в корзину, значение id продукта в вещественном типе - ошибка на стороне сервера 500 Internal Server Error (ручка PUT /api/v1/orders/:id) [683-77](https://slepsovasonya.youtrack.cloud/issue/683-77/Pri-dobavlenii-produktov-v-korzinu-znachenie-id-produkta-v-veshestvennom-tipe-oshibka-na-storone-servera-500-Internal-Server)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7.5,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен   
**Фактический результат:**       
В ответе код и статус 500 Internal Server Error  

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-78 </summary>

### При добавлении продуктов в корзину, значение id продукта в отрицательном типе - ответ об отсутствии продукта, код и статус 409 Conflict (ручка PUT /api/v1/orders/:id) [683-78](https://slepsovasonya.youtrack.cloud/issue/683-78)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": -7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен   
**Фактический результат:**       
В ответе код и статус 409 Conflict
```
{
    "code": 409,
    "message": "Нет склада, способного обработать Ваш заказ"
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-40 </summary>

### При добавлении продуктов в корзину, введение id продукта в не числовом типе - ошибка на стороне сервера 500 Internal Server Error (ручка PUT /api/v1/orders/:id) (ручка PUT /api/v1/orders/:id) [683-40](https://slepsovasonya.youtrack.cloud/issue/683-40/Pri-dobavlenii-produktov-v-korzinu-vvedenie-id-produkta-v-ne-chislovom-tipe-oshibka-na-storone-servera-500-Internal-Server-Error)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": "%",
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен   
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error  

**Приоритет:**   
Неотложная

***
</details>

<details>
<summary>ID: 683-79 </summary>

### При добавлении продуктов в корзину, если убрать ключ и значение id продукта - ответ об отсутствии продукта, код и статус 409 Conflict (ручка PUT /api/v1/orders/:id) [683-79](https://slepsovasonya.youtrack.cloud/issue/683-79/Pri-dobavlenii-produktov-v-korzinu-esli-ubrat-klyuch-i-znachenie-id-produkta-otvet-ob-otsutstvii-produkta-kod-i-status-409)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен   
**Фактический результат:**       
В ответе код и статус 409 Conflict  
```
{
    "code": 409,
    "message": "Нет склада, способного обработать Ваш заказ"
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-42 </summary>

### При добавлении продуктов в корзину с пустым id продукта - ошибка на стороне сервера сервера 500 Internal Server Error (ручка PUT /api/v1/orders/:id) [683-42](https://slepsovasonya.youtrack.cloud/issue/683-42/Pri-dobavlenii-produktov-v-korzinu-s-pustym-id-produkta-oshibka-na-storone-servera-servera-500-Internal-Server-Error-ruchka-PUT)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": "",
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен   
**Фактический результат:**       
В ответе ошибка на стороне сервера 500 Internal Server Error и корректный ответ  

**Приоритет:**   
Неотложная  

***
</details>

<details>
<summary>ID: 683-80 </summary>

### При добавлении продуктов в корзину, значение id=null продукта - ответ об отсутствии продукта, код и статус 409 Conflict (ручка PUT /api/v1/orders/:id) [683-80](https://slepsovasonya.youtrack.cloud/issue/683-80)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины:                                
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": null,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен   
**Фактический результат:**       
В ответе код и статус 409 Conflict  
```
{
    "code": 409,
    "message": "Нет склада, способного обработать Ваш заказ"
}
```

**Приоритет:**   
Критическая 

***
</details>

<details>
<summary>ID: 683-93 </summary>

### При добавлении продукта в корзину, при введении корректных значений - нарушена структура ответа, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-93](https://slepsovasonya.youtrack.cloud/issue/683-93)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id корзины  
Path Variables=2   
В теле указать:                                  
```
curl --location --request PUT 'https://cf40f600-71ef-4f12-956f-9a4d03059467.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 7,
            "quantity": 3
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 200 ОК, тело ответа выглядит:  
```
{
"productsList": [
    {
        "id": 1,
        "quantity": 3
    },
    {
        "id": 7,
        "quantity": 13
    }
],
        "status": 0,
        "deliveryPriceOur": 30,
        "deliveryTime": "25~30",
        "courierService": "Привезём быстро",
        "deliveryPrice": 0,
        "wareHouse": "Шведский дом",
        "userId": null,
        "id": 5,
        "productsCost": 804,
        "finalCCost": 804
    }
```
**Фактический результат:**       
В ответе код и статус 200 ОК, тело ответа выглядит:  
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 1,
            "quantity": 3
        },
        {
            "id": 7,
            "quantity": 3
        }
    ],
    "productsCost": 804,
    "finalCost": 804
}
```

**Приоритет:**   
Обычная

***
</details>

<details>
<summary>ID: 683-43 </summary>

### При добавлении продукта в корзину со значением quantity=0 - продукт добавляется в кол-ве 0 шт, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-43](https://slepsovasonya.youtrack.cloud/issue/683-43/Pri-dobavlenii-produkta-v-korzinu-so-znacheniem-quantity0-produkt-dobavlyaetsya-v-kol-ve-0-sht-kod-i-status-200-OK-ruchka-PUT)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 4,
            "quantity": 0
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в корзину в кол-ве 0 штук   
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 4,
            "quantity": 0
        }
    ],
    "productsCost": 0,
    "finalCost": 99
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-81 </summary>

### При добавлении продукта в корзину, введение quantity в виде строки - продукт добавляется в строчном значении, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-81](https://slepsovasonya.youtrack.cloud/issue/683-81)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 4,
            "quantity": "1"
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, продукт добавляется в корзину в строчном значении   
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 36,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Чух-чух и уже у вас",
    "status": 0,
    "productsList": [
        {
            "id": 4,
            "quantity": "1"
        }
    ],
    "productsCost": 869,
    "finalCost": 869
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-92 </summary>

### При добавлении продукта в корзину, введение quantity в дробном типе - продукт в корзину добавлен, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-92](https://slepsovasonya.youtrack.cloud/issue/683-92)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 4,
            "quantity": 2.0
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в корзину в количестве как целое число   
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 1,
            "quantity": 3
        },
        {
            "id": 4,
            "quantity": 2
        }
    ],
    "productsCost": 605,
    "finalCost": 605
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-82 </summary>

### При добавлении продукта в корзину, введение quantity продукта в вещественном типе - продукт добавляется в дробном значении (ручка PUT /api/v1/orders/:id) [683-82](https://slepsovasonya.youtrack.cloud/issue/683-82)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2.5' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 4,
            "quantity": 2.5
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код и статус 200 ОК, продукт добавлен в корзину в количестве дробного типа 
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 1,
            "quantity": 3
        },
        {
            "id": 4,
            "quantity": 2.5
        }
    ],
    "productsCost": 644.5,
    "finalCost": 644.5
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-44 </summary>

### При добавлении продукта в корзину, введение quantity в не числовом типе - продукт добавляется с не числовыми значениями в кол-во, ответ 200 ОК (ручка PUT /api/v1/orders/:id) [683-44](https://slepsovasonya.youtrack.cloud/issue/683-44/Pri-dobavlenii-produkta-v-korzinu-vvedenie-quantity-v-ne-chislovom-tipe-produkt-dobavlyaetsya-s-ne-chislovymi-znacheniyami-v-kol)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2.5' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 4,
            "quantity": "%"
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, продукт добавляется в корзину с количеством, где указаны не числовые значения
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 33,
    "deliveryTime": "20~25",
    "wareHouse": "Чердак",
    "courierService": "На метле уюта",
    "status": 0,
    "productsList": [
        {
            "id": 4,
            "quantity": "11%"
        }
    ],
    "productsCost": 0,
    "finalCost": 99
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-83 </summary>

### При добавлении продукта в корзину, если убрать ключ и значение quantity - продукт добавляется в корзину в количестве=undefianed (ручка PUT /api/v1/orders/:id) [683-83](https://slepsovasonya.youtrack.cloud/issue/683-83)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT ' https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2.5' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 4,
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, продукт добавляется в корзину в количестве=undefined  
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 33,
    "deliveryTime": "20~25",
    "wareHouse": "Чердак",
    "courierService": "На метле уюта",
    "status": 0,
    "productsList": [
        {
            "id": 4,
            "quantity": "undefined"
        }
    ],
    "productsCost": 0,
    "finalCost": 99
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-46 </summary>

### При добавлении продукта в корзину с пустым quantity - продукт добавляется, quantity строчным типом, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [683-46](https://slepsovasonya.youtrack.cloud/issue/683-46/Pri-dobavlenii-produkta-v-korzinu-s-pustym-quantity-produkt-dobavlyaetsya-quantity-strochnym-tipom-kod-i-status-200-OK-ruchka)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 8,
            "quantity": ""
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, продукт добавлен, quantity = ""  
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 5,
            "quantity": 1
        },
        {
            "id": 8,
            "quantity": ""
        }
    ],
    "productsCost": 349,
    "finalCost": 349
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 83-84 </summary>

### При добавлении продукта в корзину, с пустым значением quantity=null - продукт добавляется в количестве null, код и статус 200 ОК (ручка PUT /api/v1/orders/:id) [83-84](https://slepsovasonya.youtrack.cloud/issue/683-84)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **PUT** на добавление продуктов в корзину **/api/v1/orders/:id**  
В URL указать id существующей корзины  
В теле указать:                                  
```
curl --location --request PUT 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2' \
--header 'Content-Type: application/json' \
--data '{
    "productsList": [
        {
            "id": 4,
            "quantity": null
        }
    ]
}'
```

**Ожидаемый результат:**  
В ответе код и статус 400 Bad request, в корзину продукт не добавлен  
**Фактический результат:**       
В ответе код 200 ОК, продукт добавляется в корзину в количестве = null    
```
{
    "id": 2,
    "userId": null,
    "deliveryPrice": 0,
    "deliveryPriceOur": 30,
    "deliveryTime": "25~30",
    "wareHouse": "Шведский дом",
    "courierService": "Привезём быстро",
    "status": 0,
    "productsList": [
        {
            "id": 5,
            "quantity": 1
        },
        {
            "id": 4,
            "quantity": null
        }
    ],
    "productsCost": 349,
    "finalCost": 349
}
```

**Приоритет:**   
Критическая

***
</details>

<details>
<summary>ID: 683-47 </summary>

### При удалении корзины, с существующей id корзины - корзину не находит, код и статус - 404 Not Found (ручка DELETE /api/v1/orders/:id) [683-47](https://slepsovasonya.youtrack.cloud/issue/683-47/Pri-udalenii-korziny-s-sushestvuyushej-id-korziny-korzinu-ne-nahodit-kod-i-status-404-Not-Found-ruchka-DELETE-api-v1-orders-id)
 
**Окружение:**  
Адрес сервера:  
Тестовый стенд "Яндекс.Прилавка"  
Адрес стенда на момент тестирования: https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru  

**Шаги для воспроизведения:**    
Отправить **DELETE** на удаление корзины **/api/v1/orders/:id**   
В URL указать id существующей корзины  
Path Variables=2                                 
```
curl --location --request DELETE 'https://3d2f8dd4-7d09-4143-bbcd-f54e2d3a8ae2.serverhub.praktikum-services.ru/api/v1/orders/2'
```

**Ожидаемый результат:**  
В ответе код и статус 200 OK, в корзина удалена  
```
{
       "ok": true
}
```
**Фактический результат:**       
В ответе код и статус 404 Not Found, корзина НЕ удалилась    
```
{
    "code": 404,
    "message": "Корзина с id=2 не найдена"
}
```

**Приоритет:**   
Неотложная  

***
</details>

</details>

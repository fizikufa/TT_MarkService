# TT_MarkService

# API-документация: Сервис маркировки (Honest Signs / Честный Знак)

## Окружения

| Окружение | Base URL |
|-----------|----------|
| Тестовый контур | `https://dev1c.tom-tailor.team/mark/v3` |
| Продуктовый | `https://api.tom-tailor.team/mark/v3` |

> **Проблема тестового контура:** заказы на эмиссию (`/order_create`) создаются, но не отправляются в Честный Знак (ЧЗ).

---

## Общие параметры запросов

Все методы принимают и возвращают JSON. Аутентификация выполняется по схеме **HTTP Basic Auth** с учётными данными, выданными для доступа к сервису.

| Параметр | Значение |
|----------|----------|
| Content-Type | `application/json` |
| Accept | `application/json` |
| Authorization | `Basic <base64(login:password)>` |

---

## Методы API

### POST /mark_free — Проверка свободных кодов маркировки

Проверяет наличие свободных (незарезервированных) кодов маркировки в пуле для указанных штрихкодов товаров.

**Метод:** `POST`  
**URL:** `/mark_free`

#### Request Body

```json
{
    "FreeHonestSigns": [
        {
            "Barcode": "4067261336381",
            "Quantity": 1
        },
        {
            "Barcode": "4067261336383",
            "Quantity": 5
        }
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `FreeHonestSigns` | `array` | Список запрашиваемых позиций |
| `FreeHonestSigns[].Barcode` | `string` | EAN/GTIN штрихкод товара |
| `FreeHonestSigns[].Quantity` | `integer` | Требуемое количество кодов |

#### Response Body — HTTP 200 OK

```json
{
    "FreeHonestSigns": [
        {
            "barcode": "4067261336381",
            "Base64Marks": [
                "MDEwNDA2NzI2MTMzNjM4MTIxNVVoVWlpSUZ6aHJwVx05MUVFMTAdOTJCcGY5aERqVno0c0VEcmdCUG81aC9DMW5aRDJCem0xL212T1BHTHFlQUpJPQ==",
                "MDEwNDA2NzI2MTMzNjM4MTIxNXZKVlI+Zlh3ci9WbB05MUVFMTAdOTJlc2JGM0ZQYWNPOE45L3NOR3VNY2ZLRTl6ODA1Qk00ZGpKTzFsMkNVbXQ0PQ=="
            ]
        },
        {
            "barcode": "4067261336383",
            "Base64Marks": []
        }
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `FreeHonestSigns` | `array` | Список результатов по каждому штрихкоду |
| `FreeHonestSigns[].barcode` | `string` | Штрихкод товара |
| `FreeHonestSigns[].Base64Marks` | `array<string>` | Коды маркировки в формате Base64; пустой массив — коды отсутствуют |

> **Логика работы:** если `Base64Marks` пустой — пул исчерпан, необходимо создать запрос на эмиссию через `/order_create`.

---

### POST /mark_commit — Подтверждение печати кода маркировки

Отправляет отчёт о факте печати кодов маркировки. Вызывается **после** успешной печати.

**Метод:** `POST`  
**URL:** `/mark_commit`

#### Request Body — вариант 1 (Base64)

```json
{
    "Base64Marks": [
        "MDEwNDA2NzI2MTMzNjM4MTIxNVVoVWlpSUZ6aHJwVx05MUVFMTAdOTJCcGY5aERqVno0c0VEcmdCUG81aC9DMW5aRDJCem0xL212T1BHTHFlQUpJPQ==",
        "MDEwNDA2NzI2MTMzNjM4MTIxNXZKVlI+Zlh3ci9WbB05MUVFMTAdOTJlc2JGM0ZQYWNPOE45L3NOR3VNY2ZLRTl6ODA1Qk00ZGpKTzFsMkNVbXQ0PQ=="
    ]
}
```

#### Request Body — вариант 2 (HonestSigns строки)

```json
{
    "HonestSigns": [
        "010000200000000221TESTSIGN00002",
        "010000200000000221TESTSIGN00002"
    ]
}
```


| Поле | Тип | Описание |
|------|-----|----------|
| `Base64Marks` | `array<string>` | Коды маркировки в формате Base64 (взаимоисключающе с `HonestSigns`) |
| `HonestSigns` | `array<string>` | Коды маркировки в виде строк ЧЗ (взаимоисключающе с `Base64Marks`) |

#### Response Body — HTTP 200 OK

```json
{
    "result": "Статусы установлены"
}
```

---

### POST /mark_release — Возврат кода маркировки в пул

Освобождает неиспользованные коды маркировки обратно в пул.

**Метод:** `POST`  
**URL:** `/mark_release`

#### Request Body — вариант 1 (Base64)

```json
{
    "Base64Marks": [
        "MDEwNDA2NzI2MTMzNjM4MTIxNVVoVWlpSUZ6aHJwVx05MUVFMTAdOTJCcGY5aERqVno0c0VEcmdCUG81aC9DMW5aRDJCem0xL212T1BHTHFlQUpJPQ==",
        "MDEwNDA2NzI2MTMzNjM4MTIxNXZKVlI+Zlh3ci9WbB05MUVFMTAdOTJlc2JGM0ZQYWNPOE45L3NOR3VNY2ZLRTl6ODA1Qk00ZGpKTzFsMkNVbXQ0PQ=="
    ]
}
```

#### Request Body — вариант 2 (HonestSigns строки)

```json
{
    "HonestSigns": [
        "010000200000000221TESTSIGN00002",
        "010000200000000221TESTSIGN00002"
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `Base64Marks` | `array<string>` | Коды маркировки в формате Base64 (взаимоисключающе с `HonestSigns`) |
| `HonestSigns` | `array<string>` | Коды маркировки в виде строк ЧЗ (взаимоисключающе с `Base64Marks`) |

#### Response Body — HTTP 200 OK

```json
{
    "result": "Статусы установлены"
}
```

---

### POST /order_create — Создание запроса на эмиссию

Создаёт заявку на выпуск новых кодов маркировки в ЧЗ. Вызывается, когда пул пуст (`Base64Marks: []` по запросу `/mark_free`). После создания заказа необходимо повторно проверять пул через `/mark_free` до появления кодов.

**Метод:** `POST`  
**URL:** `/order_create`

#### Request Body

```json
{
    "CreateEmissionRequest": [
        {
            "Barcode": "4067261267432",
            "Quantity": 1
        },
        {
            "Barcode": "2000000002",
            "Quantity": 5
        }
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `CreateEmissionRequest` | `array` | Список позиций для эмиссии |
| `CreateEmissionRequest[].Barcode` | `string` | EAN/GTIN штрихкод товара |
| `CreateEmissionRequest[].Quantity` | `integer` | Запрашиваемое количество кодов |

#### Response Body — HTTP 200 OK

```json
{
    "orders": [
        {
            "order_id": "f56970e2-7ed2-407b-b2c1-a51e3836af41",
            "items": [
                {
                    "gtin": "04067261267432",
                    "quantity": 1
                }
            ]
        },
        {
            "order_id": "f56970e2-7ed2-407b-b2c1-a51e3gsdfg",
            "items": [
                {
                    "gtin": "2000000002",
                    "quantity": 5
                }
            ]
        }
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `orders` | `array` | Список созданных заказов (по одному на штрихкод) |
| `orders[].order_id` | `string (UUID)` | Идентификатор заказа на эмиссию в ЧЗ |
| `orders[].items` | `array` | Позиции заказа |
| `orders[].items[].gtin` | `string` | GTIN товара (14 знаков с ведущим нулём) |
| `orders[].items[].quantity` | `integer` | Количество заказанных кодов |

---

### POST /order_status — Запрос состояния документов эмиссии

Возвращает текущий статус обработки документов эмиссии по их идентификаторам. Используется для отслеживания готовности заказов после вызова `/order_create`.

**Метод:** `POST`  
**URL:** `/order_status`

#### Request Body

```json
{
    "EmissionStatusRequest": [
        "4EC137D0-5FC4-44A6-A4FE-38410B36FBBC",
        "9D5225A8-CD9E-44C0-9652-A7F102BE26B0",
        "432D6B4E-44A8-45EB-BC3D-6E89CEB8EF74",
        "50FACA94-9376-497D-8C18-97662D62C386"
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `EmissionStatusRequest` | `array<string (UUID)>` | Список идентификаторов документов эмиссии |

#### Response Body — HTTP 200 OK

```json
{
    "EmissionDocumentStatus": [
        {
            "DocumentId": "4EC137D0-5FC4-44A6-A4FE-38410B36FBBC",
            "Status": "InProgress",
            "Error": ""
        },
        {
            "DocumentId": "432D6B4E-44A8-45EB-BC3D-6E89CEB8EF74",
            "Status": "InProgress"
        },
        {
            "DocumentId": "9D5225A8-CD9E-44C0-9652-A7F102BE26B0",
            "Status": "Ready"
        },
        {
            "DocumentId": "50FACA94-9376-497D-8C18-97662D62C386",
            "Status": "Ready"
        }
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `EmissionDocumentStatus` | `array` | Список статусов по каждому документу |
| `EmissionDocumentStatus[].DocumentId` | `string (UUID)` | Идентификатор документа эмиссии |
| `EmissionDocumentStatus[].Status` | `string` | Текущий статус документа (см. таблицу статусов ниже) |
| `EmissionDocumentStatus[].Error` | `string` | Описание ошибки; присутствует только при наличии ошибки, может отсутствовать в ответе |

#### Значения поля `Status`

| Статус | Описание |
|--------|----------|
| `InProgress` | Документ эмиссии обрабатывается; коды маркировки ещё не готовы |
| `Ready` | Документ обработан; коды маркировки доступны для получения через `/mark_get` |

> **Логика работы:** после создания заказа через `/order_create` необходимо периодически опрашивать `/order_status` до перехода документов в статус `Ready`, после чего можно запрашивать коды через `/mark_get`.

---

### POST /mark_get — Получение кодов маркировки по документам эмиссии

Возвращает коды маркировки (в формате Base64), сгруппированные по документам эмиссии. Вызывается после того, как `/order_status` вернул статус `Ready`.

**Метод:** `POST`  
**URL:** `/mark_get`

#### Request Body

```json
{
    "EmissionDocuments": [
        "57512593-a4ed-4410-a5dc-4721c1396463",
        "97d16f3f-be16-4a1f-80ba-bf1c32a2c990",
        "6dc3bb5a-5b7c-4c12-a427-dad1996ac19e"
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `EmissionDocuments` | `array<string (UUID)>` | Список идентификаторов документов эмиссии со статусом `Ready` |

#### Response Body — HTTP 200 OK

```json
{
    "EmissionDocuments": [
        {
            "EmissionDocument": "57512593-a4ed-4410-a5dc-4721c1396463",
            "HonestSigns": [
                {
                    "Barcode": "4011694590223",
                    "Base64Mark": "MDEwNDAxMTY5NDU5MDIyMzIxNT0pRmVKJTNTPi9uZx05MUVFMTEdOTJJWG51VWVza0pQYmFmZVc2RWVpR2JTZ2tUT3M5Q3VxS2RLWGc5bGlWWnBzPQ=="
                },
                {
                    "Barcode": "4607940693451",
                    "Base64Mark": "MDEwNDYwNzk0MDY5MzQ1MTIxNVJqZVBNZDZZVkkrSR05MUVFMTEdOTJyeGRtS0tHclNMNXpMaXVHQ3dscHkwcGY3MzRHTUF2Q2ZRV21kYVNQWjRZPQ=="
                },
                {
                    "Barcode": "4607940693468",
                    "Base64Mark": "MDEwNDYwNzk0MDY5MzQ2ODIxNWZhLmZYJSdMWXMxRR05MUVFMTEdOTJzc0puVnlucFVacUxvSlZDVXJkMmh0eXpqK1NoUkc1RXdoczlmOHlUdFZFPQ=="
                },
                {
                    "Barcode": "4607940693475",
                    "Base64Mark": "MDEwNDYwNzk0MDY5MzQ3NTIxNWNveCg8RGdPYWpTeh05MUVFMTEdOTJ6OFJUTW8vb21xSHowcmtvbzUvVU1kamJzMlRPd3hzMXA3VjJVT09Ndk9JPQ=="
                }
            ]
        },
        {
            "EmissionDocument": "97d16f3f-be16-4a1f-80ba-bf1c32a2c990",
            "HonestSigns": [
                {
                    "Barcode": "4059995291371",
                    "Base64Mark": "MDEwNDA1OTk5NTI5MTM3MTIxNUdyP0k/VXNsUk9UPB05MUVFMTEdOTJMeFdNb1hPMmw3VHhMejJpTXpLRnBCN25IVEhZQWhwajAzcWpyeWhCNjRJPQ=="
                },
                {
                    "Barcode": "4607940695592",
                    "Base64Mark": "MDEwNDYwNzk0MDY5NTU5MjIxNW5IWlBLaDlRS3NwRh05MUVFMTEdOTJOOUprbEtOK2dXQkdPOVJ6d283T0ZTeCttNllNQjh0dkJwTThyRGlnUFFjPQ=="
                },
                {
                    "Barcode": "4607940695608",
                    "Base64Mark": "MDEwNDYwNzk0MDY5NTYwODIxNVJlaXVUUTQ7VVhiNx05MUVFMTEdOTJmdnFHRURUbGYvZ0ZmNmNaNHJsZHNsYmx2QTFsYlg0Q1NLTlVMQVRDQU00PQ=="
                }
            ]
        },
        {
            "EmissionDocument": "6dc3bb5a-5b7c-4c12-a427-dad1996ac19e",
            "HonestSigns": [
                {
                    "Barcode": "4067261242644",
                    "Base64Mark": "MDEwNDA2NzI2MTI0MjY0NDIxNW5UO0RBUTZhRyJTKB05MUVFMTEdOTIrczk0bFR4eWR0RXVSSzhzRlpZZm1GVXZuaWNOWEgxeEh2bjRnem9RUFlnPQ=="
                }
            ]
        }
    ]
}
```

| Поле | Тип | Описание |
|------|-----|----------|
| `EmissionDocuments` | `array` | Список документов с кодами маркировки |
| `EmissionDocuments[].EmissionDocument` | `string (UUID)` | Идентификатор документа эмиссии |
| `EmissionDocuments[].HonestSigns` | `array` | Список кодов маркировки для данного документа |
| `EmissionDocuments[].HonestSigns[].Barcode` | `string` | EAN/GTIN штрихкод товара |
| `EmissionDocuments[].HonestSigns[].Base64Mark` | `string` | Код маркировки в формате Base64 |

> **Важно:** полученные коды необходимо подтвердить через `/mark_commit` после успешной печати. Неиспользованные коды следует вернуть в пул через `/mark_release`.

---

## Общий сценарий работы

```
1. Запрос на печать → /mark_free
       │
       ├─ Base64Marks не пустой → /mark_commit (подтверждение печати)
       │
       └─ Base64Marks пустой   → /order_create (заказ эмиссии)
                                        │
                                        └─ Polling /order_status
                                               │
                                               ├─ Status: InProgress → повторить запрос
                                               │
                                               └─ Status: Ready → /mark_get (получение кодов)
                                                                        │
                                                                        └─ /mark_commit (подтверждение печати)

Если код маркировки не нужен → /mark_release (возврат в пул)
```

---

## Примечания

- Аутентификация: схема **HTTP Basic Auth** (`Authorization: Basic <base64(login:password)>`). Используются учётные данные, выданные ранее для доступа к сервису 1C.

- В тестовом контуре заказы на эмиссию не передаются в ЧЗ; для полного тестирования цикла эмиссии следует использовать продуктовое окружение.

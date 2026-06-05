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

## Общий сценарий работы

```
1. Запрос на печать → /mark_free
       │
       ├─ Base64Marks не пустой → /mark_commit (подтверждение печати)
       │
       └─ Base64Marks пустой   → /order_create (заказ эмиссии)
                                        │
                                        └─ Polling /mark_free
                                               │
                                               └─ Коды появились → /mark_commit

Если код маркировки не нужен → /mark_release (возврат в пул)
```

---

## Примечания

- Аутентификация: схема **HTTP Basic Auth** (`Authorization: Basic <base64(login:password)>`). Используются учётные данные, выданные ранее для доступа к сервису маркировки.
- GTIN в ответе `/order_create` возвращается в 14-значном формате с ведущим `0`, тогда как в запросе передаётся без него — это ожидаемое поведение.
- В тестовом контуре заказы на эмиссию не передаются в ЧЗ; для полного тестирования цикла эмиссии следует использовать продуктовое окружение.

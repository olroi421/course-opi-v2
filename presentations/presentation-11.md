# RESTful API: принципи проєктування, тестування та документування

## План лекції

1. Що таке REST та API
2. Принципи REST архітектури
3. Проєктування RESTful API
4. HTTP методи та коди статусу
5. Безпека API
6. Тестування API
7. Документування з OpenAPI

## Основні поняття

- **API** — «контракт» між програмами: які запити, який формат, яка відповідь
- **REST** — архітектурний стиль на основі ресурсів і HTTP
- **Ресурс та URI** — сутність предметної області та її адреса
- **Безпечний / ідемпотентний метод** — не змінює стан / повтор дає той самий результат
- **Stateless** — кожен запит містить усе потрібне для обробки
- **Автентифікація / авторизація** — «хто ви?» / «що вам можна?»
- **OpenAPI** — машинозчитуваний опис HTTP API

## 1. Що таке REST та API

## API — Application Programming Interface

### 🔌 Інтерфейс для взаємодії програм:

```mermaid
graph LR
    A["Мобільний застосунок"] -->|API-запити| B["Сервер"]
    C["Вебсайт"] -->|API-запити| B
    D["Інший сервіс"] -->|API-запити| B
    E["ШІ-агент"] -->|API-запити| B

    B -->|JSON-відповіді| A
    B -->|JSON-відповіді| C
    B -->|JSON-відповіді| D
    B -->|JSON-відповіді| E
```

### 🎯 Навіщо потрібен API:

- Розділення клієнтської та серверної частин
- Мобільні застосунки
- Інтеграція між сервісами
- Мікросервісна архітектура
- Доступ для ШІ-агентів — потрібен чіткий опис

## REST — Representational State Transfer

### 📜 Історія:

Архітектурний стиль, запропонований **Роєм Філдінгом у 2000** році

Аналіз причин успіху архітектури вебу

### 🔑 Ключова ідея:

Взаємодія з **ресурсами** через стандартизований інтерфейс

Ресурс = будь-яка інформація (користувач, товар, замовлення)

### 🌐 REST використовує HTTP:

- Методи (GET, POST, PUT, PATCH, DELETE)
- URI для ідентифікації ресурсів
- Коди статусу для результатів
- Заголовки для метаданих

## Модель зрілості Річардсона

| Рівень | Опис |
|--------|------|
| **0** | Один URL, один метод (POST) для всього |
| **1** | Окремі ресурси зі своїми URI |
| **2** | Правильні HTTP-методи та коди статусу |
| **3** | Гіпермедіа (HATEOAS): відповіді містять посилання |

### 🎯 На практиці:

Більшість «RESTful» API — це **рівень 2**

Ми проєктуватимемо API рівня 2

## REST серед інших підходів

| Підхід | Коли доречний |
|--------|---------------|
| **REST** | API загального призначення — вибір за замовчуванням |
| **GraphQL** | Клієнт сам обирає поля; складні клієнтські застосунки |
| **gRPC** | Швидка взаємодія між внутрішніми сервісами |
| **WebSocket / SSE** | Чати, сповіщення, потокові дані |
| **Вебхуки** | Сервер сам повідомляє про подію |

### 🤖 ШІ-застосунки: протоколи на кшталт MCP часто «обгортають» наявні REST API

## 2. Принципи REST

## 6 обмежень REST архітектури

```mermaid
graph TB
    A["REST-обмеження"] --> B["Client-Server"]
    A --> C["Stateless"]
    A --> D["Cacheable"]
    A --> E["Uniform Interface"]
    A --> F["Layered System"]
    A --> G["Code on Demand<br/>необов'язкове"]
```

## Client-Server

### 🔄 Розділення відповідальності:

**Клієнт:**
- Інтерфейс користувача
- Подання даних

**Сервер:**
- Бізнес-логіка
- Зберігання даних

### ✅ Переваги:

Незалежна еволюція компонентів

Краща масштабованість

Спрощення серверних компонентів

## Stateless — без стану

```mermaid
sequenceDiagram
    participant C as Клієнт
    participant S as Сервер

    C->>S: GET /api/products (+ Auth Token)
    S->>C: 200 OK + список товарів

    C->>S: POST /api/orders (+ Auth Token + дані)
    S->>C: 201 Created + нове замовлення

    Note over C,S: Кожен запит містить УСЮ необхідну інформацію
```

### 🎯 Кожен запит самодостатній:

Сервер не зберігає стан клієнта

Уся інформація сесії — на клієнті

### ✅ Переваги:

Проста масштабованість, надійність, кешування

## Uniform Interface

### 🎯 Уніфікований інтерфейс має 4 аспекти:

**1. Ідентифікація ресурсів через URI**
```
/api/products/123
/api/users/456/orders
```

**2. Маніпуляція через представлення**
- JSON, XML для передачі стану ресурсу

**3. Самоописові повідомлення**
- HTTP-методи, коди статусу, заголовки

**4. HATEOAS** (Hypermedia as Engine of Application State)
```json
{
  "id": 123,
  "_links": {
    "self": { "href": "/api/v1/products/123" },
    "reviews": { "href": "/api/v1/products/123/reviews" }
  }
}
```

## Cacheable, Layered System, Code on Demand

### 🗄️ Cacheable:
Відповіді явно вказують, чи можна їх кешувати

### 🧱 Layered System:
Проміжні шари (балансувальники, кеші, шлюзи) невидимі для клієнта

### 📜 Code on Demand (необов'язкове):
Сервер може передавати клієнтові виконуваний код

## 3. Проєктування API

## Ресурси та URI

### 🎯 Проєктуйте навколо ресурсів (іменників):

```
✅ Добре:
GET    /api/products              # Колекція товарів
GET    /api/products/123          # Окремий товар
GET    /api/products/123/reviews  # Відгуки товару
POST   /api/products              # Створення
PUT    /api/products/123          # Оновлення
DELETE /api/products/123          # Видалення

❌ Погано (дієслова в URI):
GET    /api/getProduct?id=123
POST   /api/createProduct
POST   /api/deleteProduct?id=123
```

## Іменування URI

### ✅ Найкращі практики:

- 📝 Використовуйте **множину** для колекцій
- 🔡 **Малі літери** з дефісами: `/product-categories`
- 🌲 **Ієрархія** відображає зв'язки (2–3 рівні)
- 🔢 **Версіонування**: `/api/v1/products`
- 🚫 Уникайте **дієслів** та операцій в URI

```
# Ієрархія ресурсів
/products                      # Усі товари
/products/123                  # Конкретний товар
/products/123/reviews          # Відгуки товару
/products/123/reviews/456      # Конкретний відгук

/customers/789/orders          # Замовлення клієнта
/orders/321/items              # Товари в замовленні
```

## 4. HTTP методи

## CRUD → HTTP методи

```mermaid
graph LR
    A["CRUD-операції"] --> B["Create → POST"]
    A --> C["Read → GET"]
    A --> D["Update → PUT/PATCH"]
    A --> E["Delete → DELETE"]
```

| Метод | Призначення | Безпечний | Ідемпотентний |
|-------|-------------|:---------:|:-------------:|
| GET | Отримати ресурс | так | так |
| POST | Створити / виконати операцію | ні | ні |
| PUT | Повна заміна | ні | так |
| PATCH | Часткове оновлення | ні | не гарантовано |
| DELETE | Видалити | ні | так |
| QUERY | Запит із тілом | так | так |

## GET — читання даних

```http
GET /api/products HTTP/1.1
Host: example.com
Authorization: Bearer token123
```

### ✅ Властивості:

- **Безпечний** — не змінює стан сервера
- **Ідемпотентний** — множинні запити = один результат
- **Кешується** — можна кешувати відповіді

### 🎯 Використання:

- Отримання списку ресурсів
- Отримання окремого ресурсу
- З параметрами запиту для фільтрації

## POST — створення ресурсу

```http
POST /api/products HTTP/1.1
Content-Type: application/json

{
  "name": "Ноутбук Dell XPS 15",
  "price": 45999,
  "category": "Electronics"
}
```

### ✅ Властивості:

- **НЕ ідемпотентний** — повторний запит створить новий ресурс
- Повертає **201 Created** при успіху
- Заголовок **Location** з URI нового ресурсу
- Для безпечних повторів — заголовок **Idempotency-Key**

```http
HTTP/1.1 201 Created
Location: /api/products/789
```

## PUT vs PATCH

### PUT — повне оновлення:

```http
PUT /api/products/123 HTTP/1.1
Content-Type: application/json

{
  "name": "Оновлена назва",
  "price": 42999,
  "category": "Electronics",
  "in_stock": true
}
```

Потрібно надіслати **всі поля**

**Ідемпотентний**

### PATCH — часткове оновлення:

```http
PATCH /api/products/123 HTTP/1.1
Content-Type: application/json

{
  "price": 42999
}
```

Надсилаємо **лише змінені поля** (JSON Merge Patch)

## DELETE — видалення

```http
DELETE /api/products/123 HTTP/1.1
Authorization: Bearer token123
```

### ✅ Властивості:

- **Ідемпотентний** — повторне видалення = той самий стан
- Повертає **204 No Content** або **200 OK**
- Видалення неіснуючого → **404 Not Found**

### ⚠️ Безпека:

Завжди вимагайте авторизацію!

## QUERY — новий метод (RFC 10008, червень 2026)

```http
QUERY /api/products HTTP/1.1
Content-Type: application/json

{
  "category": "electronics",
  "price": { "min": 1000, "max": 5000 },
  "sort": "-price"
}
```

### 🎯 Навіщо:
- **GET**: фільтри мусять бути в URL — довжина, журнали, складні структури
- **POST**: тіло є, але не безпечний і не ідемпотентний
- **QUERY** = тіло як у POST + безпечність як у GET

### ⏳ Підтримка в інструментах ще з'являється

## HTTP коди статусу

## 2xx — Успіх

- **200 OK** — успішний GET, PUT, PATCH
- **201 Created** — успішний POST, ресурс створено
- **204 No Content** — успішно, але без тіла відповіді (DELETE)

## 4xx — Помилки клієнта

- **400 Bad Request** — некоректний синтаксис запиту
- **401 Unauthorized** — потрібна автентифікація («хто ви?»)
- **403 Forbidden** — доступ заборонено («вам не можна»)
- **404 Not Found** — ресурс не знайдено
- **409 Conflict** — конфлікт зі станом ресурсу
- **422 Unprocessable Entity** — помилки валідації
- **429 Too Many Requests** — перевищено ліміт запитів

## 5xx — Помилки сервера

- **500 Internal Server Error** — загальна помилка сервера
- **502 / 504** — проблема з проміжним чи залежним сервісом
- **503 Service Unavailable** — сервіс тимчасово недоступний

### 🎯 За кодом клієнт розуміє, **чия це помилка** і чи має сенс повторювати запит

## Фільтрація, сортування, пагінація

### 🔍 Фільтрація через параметри запиту:

```
GET /api/products?category=electronics&price_min=1000&price_max=5000
GET /api/products?in_stock=true&brand=Apple
GET /api/orders?status=pending&created_after=2026-01-01
```

### 📊 Сортування:

```
GET /api/products?sort=price          # за зростанням
GET /api/products?sort=-price         # за спаданням
GET /api/products?sort=category,price # множинне
```

### 📄 Пагінація:

```
GET /api/products?page=2&limit=20      # зсувна
GET /api/products?after=abc123&limit=20 # курсорна (швидша для великих даних)
```

## Відповідь з пагінацією

```json
{
  "data": [
    {
      "id": 21,
      "name": "Товар 21",
      "price": 999
    }
    // ... ще 19 товарів
  ],
  "pagination": {
    "total": 150,
    "page": 2,
    "limit": 20,
    "pages": 8,
    "next": "/api/products?page=3&limit=20",
    "prev": "/api/products?page=1&limit=20"
  }
}
```

## Обробка помилок

### 📋 Стандарт RFC 9457 — Problem Details:

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Помилка валідації даних",
  "status": 422,
  "detail": "Запит містить некоректні дані",
  "instance": "/api/v1/users",
  "errors": [
    {
      "field": "email",
      "code": "INVALID_FORMAT",
      "message": "Email має невалідний формат"
    },
    {
      "field": "age",
      "code": "OUT_OF_RANGE",
      "message": "Вік має бути між 18 та 120"
    }
  ]
}
```

- `Content-Type: application/problem+json`
- Повертайте **всі** помилки валідації одразу
- **Не розкривайте** внутрішні деталі (стек, SQL)

## Приклад реалізації: FastAPI

```python
from fastapi import FastAPI, HTTPException, Response
from pydantic import BaseModel, Field

app = FastAPI(title="Products API", version="1.0.0")

class ProductIn(BaseModel):
    name: str = Field(min_length=1, max_length=200)
    price: float = Field(ge=0)
    category: str

class Product(ProductIn):
    id: int

@app.post("/api/v1/products", response_model=Product,
          status_code=201)
def create_product(data: ProductIn, response: Response):
    product = Product(id=next(_ids), **data.model_dump())
    products[product.id] = product
    response.headers["Location"] = f"/api/v1/products/{product.id}"
    return product

@app.get("/api/v1/products/{product_id}", response_model=Product)
def get_product(product_id: int):
    if product_id not in products:
        raise HTTPException(status_code=404, detail="Не знайдено")
    return products[product_id]
```

- `fastapi dev main.py` → **`/docs`** (Swagger UI), **`/openapi.json`**
- Некоректні дані → автоматично **422**

## Версіонування

### 🔢 Стратегії:

```
/api/v1/products        # у URI (найпоширеніше)
Accept: application/vnd.company.v2+json   # у заголовку
```

### ✅ Правила:

- Найкраща версія — та, яка не знадобилась: **додавайте**, а не змінюйте
- Нова версія — лише для несумісних змін
- У URI — лише мажорна версія

## 5. Безпека API

## Автентифікація vs Авторизація

```mermaid
graph TB
    A["Безпека API"] --> B["Автентифікація<br/>Хто ти?"]
    A --> C["Авторизація<br/>Що тобі дозволено?"]

    B --> D["API-ключі"]
    B --> E["OAuth 2.0 / OIDC"]
    B --> F["JWT-токени"]

    C --> G["Ролі користувачів"]
    C --> H["Права доступу"]
```

### 🔒 Завжди **HTTPS** (+ HSTS)

## JWT — JSON Web Tokens

### 🔑 Структура JWT:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IklWYW4gUGV0cmVua28iLCJpYXQiOjE1MTYyMzkwMjJ9.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Header.Payload.Signature
```

### ✅ Переваги:

- **Stateless** — вся інформація в токені
- **Самодостатній** — не потребує БД для перевірки
- **Компактний** — передається в заголовку

### ⚠️ Підводні камені:
- Підпис **не шифрує** — не кладіть секрети в payload
- Завжди перевіряйте підпис, алгоритм, термін дії
- Робіть термін дії **коротким** (відкликати JWT складно)

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## OAuth 2.0 Flow

```mermaid
sequenceDiagram
    participant U as Користувач
    participant C as Клієнт
    participant A as Auth Server
    participant R as API Server

    U->>C: Запит доступу
    C->>C: code_verifier і code_challenge (PKCE)
    C->>A: Перенаправлення (+ code_challenge)
    U->>A: Логін + дозвіл
    A->>C: Authorization Code
    C->>A: Обмін code на token (+ code_verifier)
    A->>C: Access Token
    C->>R: API-запит + Token
    R->>C: Дані
```

- Для користувачів: **Authorization Code + PKCE**
- Для сервер-сервер: **Client Credentials**
- Implicit і «пароль власника» — **не використовувати**
- **OAuth 2.1** — чернетка, але вимоги вже стали практикою

## Авторизація на рівні об'єкта

### ❌ Найпоширеніша вразливість API:

`/orders/123` → змінили на `/orders/456` → бачимо чуже замовлення

### ✅ Перевіряйте належність об'єкта:

```python
order = orders.get(order_id)
if order is None or order.owner_id != current_user.id:
    raise HTTPException(status_code=404)
```

### 📋 Перелік типових ризиків: **OWASP API Security Top 10**

## Обмеження частоти запитів (Rate Limiting)

### ⚠️ Захист від зловживань і перевантаження:

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1634567890
```

### 🚫 Перевищення ліміту:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 60
Content-Type: application/problem+json
```

- Заголовки `RateLimit` / `RateLimit-Policy` — чернетка IETF
- Клієнт повторює запит із **наростаючою паузою**

## Захист від атак

### 🛡️ Основні заходи:

**SQL-ін'єкції:**
```python
# ❌ НІКОЛИ
query = f"SELECT * FROM users WHERE email = '{email}'"

# ✅ ЗАВЖДИ параметризовані запити
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (email,))
```

**CORS — Cross-Origin Resource Sharing:**
```http
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

- CORS — механізм **браузера**, не захист від інших програм
- `Allow-Origin: *` — лише для публічних даних

**HTTPS завжди в production!**

## 6. Тестування API

## Типи тестів

```mermaid
graph TB
    A["Тестування API"] --> B["Unit-тести<br/>Окремі функції"]
    A --> C["Інтеграційні тести<br/>Endpoints з БД"]
    A --> D["E2E-тести<br/>Повні сценарії"]

    B --> E["Швидкі<br/>Багато"]
    C --> F["Середні<br/>Помірно"]
    D --> G["Повільні<br/>Мало"]
```

## Інтеграційні тести з Python

```python
import requests
import unittest

class TestProductAPI(unittest.TestCase):
    BASE_URL = "http://localhost:8000/api/v1"

    def test_create_product(self):
        product_data = {
            "name": "Тестовий товар",
            "price": 999,
            "category": "Electronics"
        }

        response = requests.post(
            f"{self.BASE_URL}/products",
            json=product_data,
            headers={"Authorization": f"Bearer {self.token}"},
            timeout=5,
        )

        self.assertEqual(response.status_code, 201)
        data = response.json()
        self.assertEqual(data["name"], product_data["name"])
        self.assertIn("id", data)

    def test_unauthorized_access(self):
        response = requests.post(
            f"{self.BASE_URL}/products",
            json={"name": "Test"}, timeout=5,
        )
        self.assertEqual(response.status_code, 401)
```

⚠️ Потрібен запущений сервер

## Тести без запуску сервера: pytest

```python
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_create_product_returns_201_and_location():
    response = client.post("/api/v1/products", json={
        "name": "Ноутбук", "price": 999, "category": "Electronics"
    })
    assert response.status_code == 201
    assert response.headers["Location"].startswith("/api/v1/products/")

def test_get_missing_product_returns_404():
    assert client.get("/api/v1/products/999999").status_code == 404

def test_invalid_data_returns_422():
    response = client.post("/api/v1/products", json={
        "name": "", "price": -5, "category": "Electronics"
    })
    assert response.status_code == 422
```

### ✅ Переваги:
- Виконується за мілісекунди, без мережі
- Найцінніші — **«негативні» тести**: некоректні й неавторизовані запити

## Postman та альтернативи

### 🧪 Скрипти перевірок у Postman:

```javascript
// Перевірка статус коду
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Перевірка структури відповіді
pm.test("Response has products array", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('products');
    pm.expect(jsonData.products).to.be.an('array');
});

// Перевірка даних
pm.test("Product has required fields", function () {
    const product = pm.response.json().products[0];
    pm.expect(product).to.have.property('id');
    pm.expect(product).to.have.property('name');
    pm.expect(product).to.have.property('price');
});
```

### 🧰 Також:
- **Bruno**, **Hoppscotch** — колекції як файли (зручно для Git)
- **Newman** — запуск колекцій Postman у конвеєрі CI/CD
- **Schemathesis** — тести, згенеровані зі специфікації OpenAPI
- **Locust**, **k6** — навантажувальні тести

## 7. Документування API

## Чому документація критична?

### 📚 Хороша документація:

- Прискорює інтеграцію розробників
- Зменшує кількість питань до підтримки
- Слугує контрактом між клієнтською та серверною частинами
- Допомагає під час введення в проєкт нових розробників
- Потрібна й ШІ-агентам, які викликають API

### 📋 Що має містити:

- Опис кожної кінцевої точки (endpoint)
- Параметри запитів
- Формати відповідей
- Коди помилок
- Приклади використання
- Потоки автентифікації

### 💡 Одне джерело істини: специфікація → документація, тести, клієнти

## OpenAPI Specification (Swagger)

### 📄 Стандарт опису REST API:

```yaml
openapi: 3.1.0
info:
  title: Products API
  version: 1.0.0
  description: API для керування товарами

servers:
  - url: https://api.example.com/v1
    description: Робочий сервер

paths:
  /products:
    get:
      summary: Отримати список товарів
      parameters:
        - name: category
          in: query
          schema:
            type: string
        - name: page
          in: query
          schema:
            type: integer
            default: 1
      responses:
        '200':
          description: Успішна відповідь
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Product'
```

### 🆕 Версії: **3.1** (узгоджена з JSON Schema), **3.2** (вересень 2025: метод QUERY, потокові відповіді)

## OpenAPI: схеми даних

```yaml
components:
  schemas:
    Product:
      type: object
      required:
        - name
        - price
      properties:
        id:
          type: integer
          readOnly: true
        name:
          type: string
          minLength: 1
          maxLength: 200
        price:
          type: number
          format: double
          minimum: 0
        category:
          type: string
        in_stock:
          type: boolean
          default: true
        created_at:
          type: string
          format: date-time
          readOnly: true
```

- У 3.1 замість `nullable: true` — `type: [string, "null"]`

## Swagger UI

### 🎨 Інтерактивна документація:

```mermaid
graph LR
    A["OpenAPI Spec<br/>YAML/JSON"] --> B["Swagger UI"]
    B --> C["Інтерактивна<br/>документація"]
    B --> D["Можливість<br/>тестування"]
    B --> E["Автогенерація<br/>клієнтів"]
```

### ✅ Переваги:

- Візуальна документація
- Тестування API в браузері
- Специфікація з коду (FastAPI) або код зі специфікації
- Генерація клієнтських бібліотек
- Схожі інструменти: **Redoc**, **Scalar**; лінтер — **Spectral**
- Для подій — **AsyncAPI**

## Найкращі практики

### ✅ Консистентність:

- Єдине іменування для всього API
- Єдиний формат відповідей і дат (ISO 8601)
- Консистентна обробка помилок

### ✅ Версіонування та застарівання:

```
/api/v1/products
/api/v2/products
```

```http
Deprecation: @1798761600
Sunset: Thu, 01 Jul 2027 00:00:00 GMT
Link: <https://api.example.com/v2/products>; rel="successor-version"
```

Підтримка старих версій, завчасні попередження

### ✅ Продуктивність:

- Стискання (gzip, Brotli)
- `ETag` + `If-None-Match` → **304**
- Вибір полів: `?fields=id,name,price`
- Пагінація за замовчуванням

## Моніторинг та журналювання

### 📊 Що відстежувати:

```mermaid
graph TB
    A["Метрики API"] --> B["Час відповіді<br/>Затримка"]
    A --> C["Частка помилок"]
    A --> D["Пропускна здатність<br/>Запитів/сек"]
    A --> E["Популярні endpoints"]
```

### 🔍 Наскрізне трасування:

```http
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
```

Стандарт **W3C Trace Context** (OpenTelemetry) замінює власні `X-Correlation-ID`

## Висновки

### 🎯 Ключові моменти:

1. **REST принципи** — stateless, уніфікований інтерфейс, ресурси
2. **HTTP правильно** — методи за семантикою, коди статусу, стандартні помилки
3. **Проєктування навколо ресурсів** — іменники, не дієслова
4. **Безпека багатошарова** — HTTPS, автентифікація й авторизація на рівні об'єктів, обмеження запитів
5. **Тестування на всіх рівнях** — unit, інтеграційні, E2E; тести без запуску сервера
6. **Документація критична** — OpenAPI як єдине джерело істини
7. **Найкращі практики** — консистентність, версіонування, моніторинг

### 💡 Головна думка:

RESTful API — стандарт для створення зрозумілих, масштабованих та надійних вебсервісів

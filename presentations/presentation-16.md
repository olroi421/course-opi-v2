# Масштабування та оптимізація продуктивності застосунків

## План лекції

1. Основи продуктивності
2. Масштабування систем
3. Оптимізація бази даних
4. Кешування
5. Асинхронна обробка та черги
6. Оптимізація коду
7. Оптимізація клієнтської частини
8. Моніторинг та аналіз
9. Практичні рекомендації
10. Підсумок курсу

## Основні поняття

- **Продуктивність** — час відповіді, пропускна здатність, використання ресурсів
- **Латентність / throughput** — час одного запиту / запитів за секунду
- **Перцентилі (p50, p95, p99)** — реальний досвід користувачів краще за середнє
- **Вертикальне / горизонтальне масштабування** — потужніший сервер / більше серверів
- **Вузьке місце** — компонент, що обмежує продуктивність усієї системи
- **Кешування** — збереження результатів дорогих операцій для повторного використання
- **Профілювання та навантажувальне тестування** — вимірюємо, де витрачається час і що система витримає

## 1. Основи продуктивності

## Чому продуктивність важлива?

### 📊 Що показують дослідження:

- Соті частки секунди впливають на конверсію (Google / Deloitte, 2020)
- Значна частина мобільних відвідувачів залишає сторінку, що вантажиться понад 3 секунди (Google)
- Швидкість враховується в пошуковому ранжуванні (Core Web Vitals)

### ⚠️ Числа залежать від галузі та методики — це тенденція, а не константа

### 💰 Продуктивність = гроші

- ✅ Вища конверсія
- ✅ Кращий SEO
- ✅ Менше серверів — менші витрати

## Розуміння вузьких місць

```mermaid
graph TB
    A[Запит користувача] --> B{Час відповіді}
    B --> C[Мережа<br/>10-100ms]
    B --> D[Обробка на сервері<br/>50-500ms]
    B --> E[База даних<br/>10-1000ms]
    B --> F[Зовнішні API<br/>100-5000ms]
```

### ⚠️ Правило:

**Не здогадуйтесь — вимірюйте!**

## Закон Амдала

### Межа прискорення від паралелізації:

```
Прискорення = 1 / ((1 - P) + P / N)
```

- **P** — частка програми, що виконується паралельно
- **N** — кількість процесорів
- **(1 − P)** — послідовна частина: саме вона обмежує прискорення

### Приклади (N → ∞):

- P = 0,9 → максимум **10×**
- P = 0,1 → максимум лише **≈ 1,11×**

**Зменшуйте послідовну частину, а не лише додавайте ядра**

## Вимірювання продуктивності

### Ключові метрики:

- ⏱️ **Час відповіді** — латентність запиту
- 🔄 **Throughput** — запитів за секунду
- 💾 **Використання ресурсів** — CPU, RAM, диск
- 📊 **Перцентилі** — p50, p95, p99

### 🎯 Перцентилі > середні значення

```
p50 = 100ms  (медіана)
p95 = 500ms  (95% запитів швидші)
p99 = 2000ms (найгірший досвід)
```

## Core Web Vitals

### 🌐 Що відчуває користувач у браузері:

**LCP** — найбільший елемент з'явився: ≤ 2,5 с

**INP** — реакція інтерфейсу на дії: ≤ 200 мс (замінив FID у 2024)

**CLS** — «стрибки» розмітки: ≤ 0,1

### 🛠️ Інструменти:

Lighthouse, PageSpeed Insights, бібліотека `web-vitals`

Лабораторні вимірювання — для налагодження, польові (реальні користувачі) — для істини

## Профілювання коду

```python
import cProfile
import pstats

def complex_function():
    result = []
    for i in range(10000):
        result.append(sum(range(i)))
    return result

# Профілювання
profiler = cProfile.Profile()
profiler.enable()
complex_function()
profiler.disable()

# Аналіз результатів
stats = pstats.Stats(profiler)
stats.sort_stats('cumulative')
stats.print_stats(10)  # Топ-10 функцій
```

**Ще:** py-spy (без зупинки застосунку), Scalene, line_profiler, memory_profiler

## 2. Масштабування систем

## Вертикальне vs горизонтальне

### 📈 Вертикальне (Scale Up):

**Переваги:**
- ✅ Просто реалізувати
- ✅ Не потрібно змінювати код
- ✅ Менше складності

**Недоліки:**
- ❌ Фізична межа потужності
- ❌ Нелінійне зростання вартості
- ❌ Єдина точка відмови

## Горизонтальне масштабування

### 📊 Горизонтальне (Scale Out):

**Переваги:**
- ✅ Практично необмежене
- ✅ Відмовостійкість
- ✅ Приблизно лінійна вартість

**Недоліки:**
- ❌ Складніша архітектура
- ❌ Потрібні застосунки без збереження стану (stateless)
- ❌ Розподілена система: узгодженість даних

## Архітектура масштабування

```mermaid
graph TB
    A[Балансувальник навантаження] --> B[Сервер 1]
    A --> C[Сервер 2]
    A --> D[Сервер 3]
    A --> E[Сервер N]

    B --> F[(База даних)]
    C --> F
    D --> F
    E --> F

    G[Кеш] --> A
```

**Стан сесій — у зовнішньому сховищі (Redis) або в токенах**

## Балансування навантаження

### Стратегії розподілу:

1. **Round Robin** — по черзі
2. **Least Connections** — найменше з'єднань
3. **IP Hash** — на основі IP клієнта
4. **Weighted** — з вагами серверів

### Приклад Nginx:

```nginx
upstream backend {
    least_conn;
    server backend1:8000 weight=3;
    server backend2:8000 weight=2;
    server backend3:8000 backup;
}
```

## Health Checks

### Моніторинг доступності серверів:

```nginx
server {
    location / {
        proxy_pass http://backend;

        # Автоматичне виключення збійних серверів
        proxy_next_upstream error timeout
                           invalid_header http_500;
    }
}
```

**Якщо сервер не відповідає → виключається з пулу**

## Автоматичне масштабування

### ☁️ Autoscaling:

- Навантаження зросло → додаються екземпляри
- Навантаження спало → зайві видаляються
- Kubernetes: Horizontal Pod Autoscaler

### ⚠️ Умови та ризики:

- Працює для застосунків **без стану**
- Потрібні health checks і швидкий запуск
- Задавайте **верхні межі** й сповіщення про витрати

## 3. Оптимізація бази даних

## Індексація

### Без індексу — повільно:

```sql
SELECT * FROM users WHERE email = 'user@example.com';
-- Час: 450ms на 1,000,000 записів
```

### З індексом — швидко:

```sql
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'user@example.com';
-- Час: 2ms
```

**Різниця в порядки величин!** 🚀 (значення ілюстративні)

## Коли використовувати індекси

### ✅ Індексуйте:

- Стовпці в умовах WHERE
- Стовпці в операціях JOIN
- Стовпці в ORDER BY
- Зовнішні ключі

### ❌ Не індексуйте:

- Рідко використовувані стовпці
- Малі таблиці (< 1000 записів)
- Стовпці з частими оновленнями

**Індекси прискорюють читання, але сповільнюють запис!**

## Складені індекси

```sql
-- Індекс на кілька стовпців
CREATE INDEX idx_orders_customer_date
ON orders(customer_id, created_at);

-- ✅ Використає індекс
SELECT * FROM orders
WHERE customer_id = 123
  AND created_at > '2024-01-01';

-- ✅ Використає індекс
SELECT * FROM orders
WHERE customer_id = 123;

-- ❌ Зазвичай НЕ використає індекс ефективно
SELECT * FROM orders
WHERE created_at > '2024-01-01';
```

**Порядок стовпців важливий!** Перевіряйте план виконання

## N+1 проблема

### ❌ Погано — N+1 запитів:

```python
# 1 запит для користувачів
users = session.scalars(select(User)).all()

# N запитів для замовлень (по 1 для кожного)
for user in users:
    orders = user.orders  # Новий запит!
    print(f"{user.name}: {len(orders)} orders")
```

**100 користувачів = 101 запит до БД!** 😱

## Вирішення N+1

### ✅ Добре — жадібне завантаження:

```python
from sqlalchemy.orm import selectinload

users = session.scalars(
    select(User).options(selectinload(User.orders))
).all()

for user in users:
    # Дані вже завантажені!
    print(f"{user.name}: {len(user.orders)} orders")
```

**100 користувачів = 2 запити!** ✨

- `selectinload` — для колекцій; `joinedload` — для зв'язків «багато до одного»
- Потрібна лише кількість? `COUNT` + `GROUP BY` у SQL

## EXPLAIN ANALYZE

### Аналіз плану виконання:

```sql
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id)
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;

-- Результат покаже:
-- ✅ Чи використовуються індекси
-- ✅ Скільки рядків сканується
-- ✅ Час виконання кожного кроку
```

## З'єднання та масштабування бази

### 🔌 Пул з'єднань:

```python
engine = create_engine(
    "postgresql+psycopg://user:password@localhost/mydb",
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,
)
```

**PgBouncer** — проміжний пул, коли екземплярів застосунку багато

### 🗄️ Коли одна база не справляється:

1. Індекси, кеш, вертикальне масштабування
2. **Реплікація** — читання з реплік
3. **Шардування** — останній засіб

## 4. Кешування

## Рівні кешування

```mermaid
graph TB
    A[Користувач] --> B[Кеш браузера<br/>HTML, CSS, зображення]
    B --> C[CDN<br/>Статичні ресурси]
    C --> D[Кеш застосунку<br/>Redis / Memcached]
    D --> E[Кеш бази даних<br/>Результати запитів]
    E --> F[(База даних)]
```

## Кешування в браузері

### HTTP-заголовки для кешування:

```python
@app.route('/static/<path:filename>')
def serve_static(filename):
    response = make_response(
        send_from_directory('static', filename)
    )

    # Кешувати на 1 рік
    response.headers['Cache-Control'] = \
        'public, max-age=31536000, immutable'

    return response
```

### Лише для файлів із хешем вмісту:

```html
<link rel="stylesheet" href="style.3f9a1c.css">
<script src="app.8b21de.js"></script>
```

## Кешування на рівні застосунку

```python
import redis

redis_client = redis.Redis()

def get_popular_products():
    # Перевірити кеш
    cached = redis_client.get('popular_products')
    if cached:
        return json.loads(cached)

    # Запит до БД
    products = db.query(
        "SELECT * FROM products ORDER BY views DESC LIMIT 10"
    )

    # Зберегти в кеші на 10 хвилин
    redis_client.setex(
        'popular_products',
        600,
        json.dumps(products)
    )

    return products
```

**Redis / Valkey** — перевіряйте ліцензію

## Стратегії кешування

### 1️⃣ Cache-Aside (Lazy Loading):

1. Перевірити кеш
2. Якщо є → повернути
3. Якщо немає → завантажити з БД
4. Зберегти в кеші

### 2️⃣ Write-Through:

Записує в кеш та БД одночасно

### 3️⃣ Write-Behind:

Записує в кеш, потім асинхронно в БД (ризик втрати даних)

## Ефект табуна (cache stampede)

### 🐘 Проблема:

Популярний запис «помер» → тисячі запитів одночасно йдуть у БД

### ✅ Захист:

- **Jitter** — випадкове відхилення TTL
- **Блокування** — кеш перебудовує один запит
- **Фонове оновлення** до закінчення терміну дії

```python
ttl = 600 + random.randint(0, 60)
redis_client.setex("popular_products", ttl, data)
```

## Інвалідація кешу

### «Дві найскладніші речі в CS» (жарт):

1. Іменування змінних
2. **Інвалідація кешу**
3. Помилки на одиницю

### Стратегії інвалідації:

- ⏰ **За часом** — через певний час
- 🔔 **За подіями** — при зміні даних
- 🖐️ **Вручну** — явний виклик

## 5. Асинхронна обробка та черги

## Фонова обробка

### ⏳ Не все треба робити до відповіді користувачу

```mermaid
graph LR
    A[Запит] --> B[Веб-застосунок]
    B --> C[Черга повідомлень]
    B --> D[Відповідь 202 Accepted]
    C --> E[Виконавець 1]
    C --> F[Виконавець 2]
```

**Інструменти:** Celery, RQ, Dramatiq; брокери: RabbitMQ, Redis, Kafka

## Приклад: Celery

```python
@celery_app.task(bind=True, max_retries=3)
def send_order_confirmation(self, order_id):
    try:
        send_email(load_order(order_id).customer_email, "...")
    except ConnectionError as exc:
        raise self.retry(exc=exc, countdown=30)

@app.post("/api/orders")
def create_order():
    order = save_order(request.json)
    send_order_confirmation.delay(order.id)
    return jsonify({"id": order.id}), 202
```

### ⚠️ Вимоги:

Ідемпотентність завдань, повторні спроби, повідомлення про результат

## 6. Оптимізація коду

## Алгоритмічна складність

### Big O:

- **O(1)** — константна — найкраще! 🟢
- **O(log n)** — логарифмічна — добре 🟢
- **O(n)** — лінійна — прийнятно 🟡
- **O(n²)** — квадратична — погано 🔴
- **O(2ⁿ)** — експоненційна — жахливо 💀

### Приклад для n = 10 000:

- O(1) = 1 операція
- O(log n) ≈ 14 операцій
- O(n) = 10 000 операцій
- O(n²) = 100 000 000 операцій

## Порівняння алгоритмів

```python
# ❌ O(n²) — повільно
def find_duplicates_slow(items):
    duplicates = []
    for i in range(len(items)):
        for j in range(i + 1, len(items)):
            if items[i] == items[j]:
                duplicates.append(items[i])
    return duplicates
# 10 000 елементів: ≈ 2,5 секунди

# ✅ O(n) — швидко
def find_duplicates_fast(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        seen.add(item)
    return list(duplicates)
# 10 000 елементів: ≈ 0,002 секунди
```

## Вибір структури даних

### Швидкість операцій (середня):

| Структура | Доступ | Пошук | Вставка | Видалення |
|-----------|--------|-------|---------|-----------|
| List      | O(1)   | O(n)  | O(n)    | O(n)      |
| Set       | -      | O(1)  | O(1)    | O(1)      |
| Dict      | O(1)   | O(1)  | O(1)    | O(1)      |

**Правильна структура = швидкий код!**

## GIL: потоки, процеси, asyncio

### 🐍 CPython має GIL:

- **I/O-задачі** (мережа, БД) → потоки або `asyncio`
- **Обчислення** (CPU-bound) → процеси (`ProcessPoolExecutor`)
- Python 3.13: експериментальна збірка без GIL; **3.14 — офіційно підтримувана** (необов'язкова)

### 💡 Правило:

Спершу визначте, що вас обмежує — введення-виведення чи процесор

## Паралелізм: потоки та асинхронність

```python
import concurrent.futures

# ✅ Потоки для I/O
with concurrent.futures.ThreadPoolExecutor(10) as executor:
    results = list(executor.map(fetch, urls))
# 100 запитів: ≈ 4 с замість ≈ 30 с
```

```python
import asyncio, httpx

async def fetch_all(urls):
    async with httpx.AsyncClient() as client:
        return await asyncio.gather(
            *(client.get(url) for url in urls)
        )
```

**Час залежить від мережі; async вимагає асинхронного ланцюжка викликів**

## Генератори для пам'яті

```python
# ❌ Завантажує весь файл у пам'ять
def read_large_file_bad(filename):
    with open(filename) as f:
        lines = f.readlines()  # Усе в RAM!
    return [process(line) for line in lines]
# Файл 1 ГБ = ≈ 1 ГБ RAM

# ✅ Обробляє по одному рядку
def read_large_file_good(filename):
    with open(filename) as f:
        for line in f:  # Ліниве читання
            yield process(line)
# Файл 1 ГБ = кілька КБ RAM
```

## 7. Оптимізація клієнтської частини

## Мінімізація та стиснення

### Розмір файлів (приклад):

```
Оригінал JavaScript: 500 KB
├─ Мінімізований:    300 KB (-40%)
├─ Gzip:              90 KB (-82%)
└─ Brotli:            75 KB (-85%)
```

### Nginx-конфігурація:

```nginx
gzip on;
gzip_types text/plain text/css application/json
           application/javascript text/xml;
gzip_min_length 1000;
gzip_comp_level 6;
```

**Ще:** HTTP/2 і HTTP/3, WebP/AVIF, CDN

## Lazy Loading

### Відкладене завантаження зображень:

```html
<img src="image.jpg" loading="lazy"
     width="800" height="600" alt="Опис">
```

- ✅ Нативна підтримка в усіх сучасних браузерах
- ✅ `width` + `height` — менший CLS
- ❌ Не відкладайте головне зображення першого екрана → `fetchpriority="high"`

`IntersectionObserver` — для нескінченних списків, аналітики, анімацій

## Code Splitting

### Розділення JavaScript-коду:

```javascript
// ❌ Один великий файл
import HeavyComponent from './HeavyComponent';
// Завантажується завжди, навіть якщо не потрібен

// ✅ Динамічний імпорт
import React, { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() =>
    import('./HeavyComponent')
);

function App() {
    return (
        <Suspense fallback={<div>Завантаження...</div>}>
            <HeavyComponent />
        </Suspense>
    );
}
```

## Critical Rendering Path

```html
<!-- ❌ CSS блокує рендеринг -->
<head>
    <link rel="stylesheet" href="styles.css">
    <link rel="stylesheet" href="heavy.css">
</head>

<!-- ✅ Критичні стилі вбудовані -->
<head>
    <style>
        /* Критичні стилі для першого екрана */
        body { margin: 0; font-family: sans-serif; }
        .header { background: #333; color: white; }
    </style>

    <!-- Некритичні завантажуються асинхронно -->
    <link rel="preload" href="styles.css" as="style"
          onload="this.rel='stylesheet'">
</head>
```

**У проєктах це роблять збірники та фреймворки (SSR)**

## Debouncing та Throttling

```javascript
// Debouncing — викликає після паузи
function debounce(func, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() =>
            func.apply(this, args), delay
        );
    };
}

// Використання для пошуку
const search = debounce((query) => {
    console.log('Пошук:', query);
}, 300);

input.addEventListener('input', (e) =>
    search(e.target.value)
);
```

## Throttling: приклад

```javascript
// Throttling — обмежує частоту викликів
function throttle(func, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() =>
                inThrottle = false, limit
            );
        }
    };
}

// Для події прокручування
const handleScroll = throttle(() => {
    console.log('Прокручування:', window.scrollY);
}, 100);

window.addEventListener('scroll', handleScroll);
```

**Debounce** — після паузи; **throttle** — не частіше за інтервал

## 8. Моніторинг та аналіз

## Application Performance Monitoring

### Ключові метрики:

- ⏱️ **Час відповіді** — латентність
- 🔄 **Throughput** — запитів за секунду
- ❌ **Частота помилок** — відсоток помилок
- 💻 **CPU / пам'ять** — використання ресурсів
- 📊 **Запити до БД** — час і кількість

**Принципи моніторингу — лекція 13**

## Prometheus-метрики

```python
from prometheus_client import Counter, Histogram

# Лічильник
request_count = Counter(
    'http_requests_total',
    'Total HTTP requests'
)

# Гістограма часу (→ перцентилі в Prometheus)
request_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration'
)

@app.route('/api/data')
def get_data():
    request_count.inc()

    with request_duration.time():
        # Обробка запиту
        return jsonify(data)
```

## Розподілений трейсинг

```mermaid
sequenceDiagram
    participant Client as Клієнт
    participant API
    participant Database as БД
    participant Cache as Кеш

    Client->>API: GET /users/123
    Note over API: Span 1: 450ms
    API->>Cache: Перевірка кешу
    Note over Cache: Span 2: 5ms
    Cache-->>API: Немає в кеші
    API->>Database: Запит користувача
    Note over Database: Span 3: 200ms
    Database-->>API: Дані користувача
    API-->>Client: Відповідь
```

**Trace ID об'єднує всі spans; стандарт — OpenTelemetry (OTLP)**

## Трейсинг з OpenTelemetry

```python
provider = TracerProvider(
    resource=Resource.create({"service.name": "shop-api"})
)
provider.add_span_processor(
    BatchSpanProcessor(
        OTLPSpanExporter(endpoint="http://localhost:4317")
    )
)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("database_query"):
    user = database.get_user(user_id)
```

**Застарілий експортер Jaeger не використовуйте — Jaeger приймає OTLP**

## Навантажувальне тестування

```python
from locust import HttpUser, task, between

class ShopUser(HttpUser):
    wait_time = between(1, 3)

    @task(3)
    def view_products(self):
        self.client.get("/api/products")

    @task(1)
    def view_product(self):
        self.client.get("/api/products/1")
```

- **Інструменти:** k6, Locust, JMeter
- Збільшуємо користувачів → дивимось p95/p99 і частку помилок
- Шукаємо «точку зламу»; середовище — схоже на продакшн

## 9. Практичні рекомендації

## Правило оптимізації

### 🎯 Три кроки:

1. **Вимірюйте** — знайдіть вузьке місце
2. **Оптимізуйте** — покращте проблемне місце
3. **Вимірюйте знову** — перевірте результат

### ⚠️ Не оптимізуйте навмання!

> «Передчасна оптимізація — корінь усього зла»
>
> — **Дональд Кнут**

## Пріоритети оптимізації

### Зосередьтесь на:

1. **Найповільнішому** — зазвичай більшість часу витрачається в малій частці коду
2. **Найчастішому** — критичні шляхи
3. **Користувацькому досвіді** — те, що бачить користувач

### Ігноруйте:

- ❌ Мікрооптимізації без вимірювань
- ❌ Код, що виконується рідко
- ❌ Оптимізації з мінімальним ефектом

## Checklist продуктивності

### Серверна частина:

- ✅ Індекси на важливих стовпцях
- ✅ N+1 запити виправлені
- ✅ Кешування частих запитів
- ✅ Пул з'єднань з БД
- ✅ Фонові завдання — у чергах
- ✅ Асинхронність там, де вона виправдана

### Клієнтська частина:

- ✅ Мінімізація та стиснення
- ✅ Lazy loading зображень
- ✅ Code splitting
- ✅ CDN для статики
- ✅ Кешування в браузері (файли з хешем)
- ✅ Core Web Vitals у нормі

## Інструменти

### Профілювання:

- **Python:** cProfile, py-spy, Scalene, memory_profiler
- **JavaScript:** Chrome DevTools, Lighthouse
- **Java:** JProfiler, VisualVM

### Моніторинг:

- **OpenTelemetry** — збір телеметрії
- **Prometheus** + Grafana — метрики
- **Jaeger** — розподілений трейсинг
- **ELK / Loki** — логування

### Навантажувальне тестування:

- **k6**, **Locust**, **Apache JMeter**

## Етапи зростання системи

### 📈 Орієнтовні етапи:

**Десятки — сотні користувачів:**
- Один сервер, усе на ньому

**Сотні — тисяча:**
- Окремий сервер БД, індекси, базове кешування

**Тисячі — десятки тисяч:**
- Балансувальник + кілька серверів
- Redis / Memcached, CDN, черги

**Далі — за потреби:**
- Реплікація БД, сервіси, шардування

**Кожен крок додає складність — робіть його за вимірюваннями, а не «про запас»**

## 10. Підсумок курсу

## Від коду до надійної системи

```mermaid
graph LR
    A[Чистий код і рефакторинг] --> B[Тести]
    B --> C[CI/CD]
    C --> D[Безпека в конвеєрі]
    D --> E[Розгортання]
    E --> F[Моніторинг і продуктивність]
    F -->|метрики, інциденти, навантаження| A
```

### 🔁 Лекції 12–16:

Тести → CI/CD → безпека → чистий код → продуктивність — одне коло

## Ключові висновки

### 🎯 Основні принципи:

1. **Вимірюйте перед оптимізацією** — не здогадуйтесь
2. **Фокус на вузьких місцях** — принцип Парето
3. **Індекси й усунення N+1** — найдешевші виграші для БД
4. **Кешування = масштабованість** — на всіх рівнях, з інвалідацією
5. **Черги** — швидка відповідь і стійкість до піків
6. **Моніторинг і навантажувальні тести** — постійний контроль

### 💡 Головна думка:

Автоматизуйте перевірки, вимірюйте замість здогадів, покращуйте малими кроками

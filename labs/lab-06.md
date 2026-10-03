# Лабораторна робота 06 Інтеграція клієнтської частини з API та обробка помилок

## 🎯 Мета роботи

Навчитись підключати клієнтську частину (frontend) до створеного API, виконувати базові операції отримання та відправки даних, розуміти як працює асинхронний JavaScript.

## ✅ Завдання

Створити просту вебсторінку, яка:

- показує список ресурсів з вашого API;
- дозволяє додати новий ресурс через форму;
- показує повідомлення про успіх або помилку;
- має базове оформлення.

> [!IMPORTANT]
> Ця лабораторна використовує API, створений у лабораторній роботі 5. Переконайтесь, що ваш API запущений і працює.

## 🖥️ Програмне забезпечення

- Git [git-scm.com](https://git-scm.com) - розподілена система контролю версій;
- GitHub [github.com](https://github.com) - хмарна платформа для хостингу Git репозиторіїв;
- Visual Studio Code [code.visualstudio.com](https://code.visualstudio.com) - редактор коду з підтримкою Git;
- реляційна СКБД SQLite [sqlite.org](https://sqlite.org/)
- GitHub Desktop [desktop.github.com](https://desktop.github.com) - графічний клієнт Git (опціонально);
- мова програмування Python [https://www.python.org/](https://www.python.org/);
- вебфреймворк Flask [https://flask.palletsprojects.com](https://flask.palletsprojects.com);
- розширення Flask-CORS [flask-cors.readthedocs.io](https://flask-cors.readthedocs.io) (потрібне лише якщо сторінка відкривається не з того самого Flask-сервера, див. крок 3);
- сучасний браузер з інструментами розробника (Chrome, Firefox, Edge).

## 👥 Форма виконання роботи

Форма виконання роботи **групова** (3-4 особи в команді).

## 📝 Критерії оцінювання

- оцінка "задовільно" (4-6) - створено базову сторінку, що показує дані з API. Можуть бути помилки в обробці;
- оцінка "добре" (7-9) - сторінка працює коректно, реалізовано показ списку та додавання нових записів, є базова обробка помилок;
- оцінка "відмінно" (10-12) - все працює стабільно, код охайний і зрозумілий, є валідація форми, повідомлення користувачу зрозумілі, застосунок має гарний зовнішній вигляд.

## ⏰ Політика щодо дедлайнів

При порушенні встановленого терміну здачі лабораторної роботи максимальна можлива оцінка становить 9 балів ("добре"), незалежно від якості виконаної роботи. Винятки можливі лише за поважних причин, підтверджених документально.

## 📚 Теоретичні відомості

### Що таке асинхронність у JavaScript

Коли ви відкриваєте вебсторінку, браузер завантажує HTML, CSS та JavaScript. JavaScript може виконувати різні дії: змінювати текст, додавати елементи, відправляти дані на сервер.

Деякі операції виконуються миттєво, наприклад зміна тексту на сторінці. Але коли потрібно отримати дані з сервера через інтернет, це займає час. Якби JavaScript чекав відповіді від сервера, вся сторінка б "зависла" і користувач не міг би нічого робити.

**Асинхронність** означає, що JavaScript може відправити запит на сервер і продовжити працювати, не чекаючи відповіді. Коли дані прийдуть, спеціальна функція їх обробить.

### Проміси (Promises)

Promise — це обіцянка, що дані прийдуть у майбутньому. У промісу є три стани:

- **очікування** (pending) — запит відправлено, чекаємо відповіді;
- **виконано** (fulfilled) — дані успішно отримано;
- **відхилено** (rejected) — сталася помилка.

```javascript
// Приклад промісу
fetch('https://api.example.com/data')
    .then(response => response.json())  // Якщо успішно
    .then(data => console.log(data))    // Виводимо дані
    .catch(error => console.log(error)); // Якщо помилка
```

### Async/Await — простіший спосіб

Замість `.then()` можна використовувати `async` та `await`. Це робить код схожим на звичайний, хоча він все одно асинхронний.

```javascript
async function getData() {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    console.log(data);
}
```

Слово `await` означає "почекай, поки виконається". Але воно працює тільки всередині функції з `async`.

### Fetch API — отримання даних

`fetch()` — це функція для відправки HTTP запитів. Найпростіший варіант:

```javascript
fetch('/api/items')
```

Це відправить GET запит (запит на отримання даних) на вказану адресу. Якщо сторінку віддає той самий Flask-сервер, що й API, достатньо відносної адреси (`/api/items`); повна адреса (`http://127.0.0.1:5000/api/items`) потрібна лише тоді, коли сторінка відкривається з іншого джерела.

> [!WARNING]
> **Важливо: fetch() не вважає код 404 чи 500 помилкою**
>
> Проміс від `fetch()` відхиляється лише тоді, коли сервер взагалі недоступний (немає мережі, сервер не запущено). Якщо сервер відповів з кодом 400, 404 або 500, `fetch()` вважає це успіхом. Тому завжди перевіряйте властивість `response.ok` (вона дорівнює `true` для кодів 200–299).

### HTTP методи

- **GET** — отримати дані (як прочитати книгу);
- **POST** — створити щось нове (як написати нову сторінку);
- **PUT** — оновити існуюче (як виправити текст);
- **DELETE** — видалити (як викинути сторінку).

Для GET не потрібно нічого додаткового. Для POST потрібно вказати, які дані відправляємо:

```javascript
fetch('/api/items', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ name: 'Назва', description: 'Опис' })
})
```

### JSON — формат даних

JSON (JavaScript Object Notation) — це спосіб записати дані у вигляді тексту. Виглядає як об'єкт JavaScript:

```json
{
    "name": "Книга",
    "description": "Цікава книга",
    "id": 1
}
```

Щоб перетворити JavaScript об'єкт у JSON:

```javascript
const obj = { name: 'Книга' };
const json = JSON.stringify(obj);
```

Щоб перетворити JSON у JavaScript об'єкт:

```javascript
const json = '{"name":"Книга"}';
const obj = JSON.parse(json);
```

### CORS — політика спільного доступу до ресурсів

Браузер з міркувань безпеки дозволяє JavaScript-коду сторінки звертатися лише до того самого **джерела** (origin), з якого завантажено сторінку. Джерело складається з протоколу, домену та порту: `http://127.0.0.1:5000` і `http://localhost:5000` — це різні джерела, так само як `http://127.0.0.1:5000` і `http://127.0.0.1:5500`.

Якщо сторінка і API мають різні джерела, сервер має явно дозволити такий доступ спеціальними заголовками — це і є механізм CORS (Cross-Origin Resource Sharing). У Flask ці заголовки додає розширення Flask-CORS.

## 🟣 Ресурси

- [Проміси, async/await — Сучасний підручник з JavaScript](https://uk.javascript.info/async)
- [Fetch — Сучасний підручник з JavaScript](https://uk.javascript.info/fetch)
- [Using the Fetch API — MDN](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)
- [Cross-Origin Resource Sharing (CORS) — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)
- [Flask-CORS — документація](https://flask-cors.readthedocs.io/)
- [Inspect network activity — Chrome DevTools](https://developer.chrome.com/docs/devtools/network)
- [JavaScript fetch www.youtube.com](https://www.youtube.com/results?search_query=javascript+fetch+async+await)

## ▶️ Хід роботи

Приклад виконання лабораторної роботи ([📦 навчальний додаток](assets/lab06-flaskProject2API-frontend.zip))


### 1. Підготовка проєкту

1. **Переконайтесь що ваш API працює**
   - Запустіть ваш Flask додаток з лабораторної роботи 5
   - Перевірте в браузері або Postman, що ендпоінти відповідають
   - Запишіть базову адресу API (наприклад: `http://127.0.0.1:5000/api`)

2. **Визначте структуру вашого API**
   - Які ендпоінти у вас є? (наприклад: `/api/products`, `/api/orders`, `/api/feedback`)
   - Які поля мають ваші об'єкти? (наприклад: `name`, `price`, `email`, `message`)
   - Які операції підтримуються? (GET, POST, PUT, DELETE)


### 2. Створіть HTML структуру та JavaScript логіку для клієнтської частини

Створіть демонстраційний шаблон `templates/api-demo.html` на основі шаблону `base.html` та маршрут, який його показує:

```python
@app.route('/api-demo')
def api_demo():
    return render_template('api-demo.html')
```

**Важливо:** Адаптуйте структуру сторінки під ваші дані. Наприклад:

- Для продуктів: `name`, `price`, `description`
- Для відгуків: `name`, `email`, `message`
- Для замовлень: `customer_name`, `address`, `phone`

Мінімальна HTML-структура сторінки (елементи з цими `id` використовує JavaScript-код нижче):

```html
{% extends "base.html" %}
{% block content %}
<h1>Демонстрація роботи API</h1>

<div id="message"></div>
<div id="loading">⏳ Завантаження...</div>

<form id="addForm">
    <input id="field1" placeholder="Назва" required>
    <input id="field2" placeholder="Опис">
    <button type="submit">Додати</button>
</form>

<div id="dataList"></div>

<style>
    #loading, #message { display: none; }
    #loading.show, #message.show { display: block; }
    #message.success { color: green; }
    #message.error { color: red; }
</style>

<script src="{{ url_for('static', filename='js/api-demo.js') }}"></script>
{% endblock %}
```

Тег `<script>` розміщено **після** HTML-елементів, тому на момент виконання скрипта всі елементи вже існують на сторінці.

Для створення повноцінної клієнтської частини додайте JavaScript логіку для роботи з API у файл `static/js/api-demo.js`. Наприклад:

```javascript
// ============================================
// НАЛАШТУВАННЯ - ЗМІНІТЬ ПІД СВІЙ ПРОЄКТ
// ============================================

// ВАЖЛИВО: Вкажіть адресу ВАШОГО API.
// Якщо сторінку віддає той самий Flask-сервер, достатньо відносної адреси.
const API_URL = '/api/your-endpoint';

// ============================================
// ДОПОМІЖНІ ФУНКЦІЇ
// ============================================

function showLoading(show) {
    const loading = document.getElementById('loading');
    loading.classList.toggle('show', show);
}

function showMessage(text, type) {
    const messageBox = document.getElementById('message');
    messageBox.textContent = text;
    messageBox.className = type + ' show';
    setTimeout(() => messageBox.classList.remove('show'), 5000);
}

/**
 * Отримує текст помилки з відповіді сервера (якщо API повертає {"error": "..."})
 */
async function getErrorText(response) {
    try {
        const body = await response.json();
        return body.error || `HTTP помилка! Статус: ${response.status}`;
    } catch {
        return `HTTP помилка! Статус: ${response.status}`;
    }
}

// ============================================
// РОБОТА З API
// ============================================

/**
 * Завантажує всі записи з API
 */
async function loadData() {
    try {
        showLoading(true);

        const response = await fetch(API_URL);

        if (!response.ok) {
            throw new Error(await getErrorText(response));
        }

        const data = await response.json();
        displayData(data);

    } catch (error) {
        showMessage('❌ Помилка завантаження: ' + error.message, 'error');
        console.error('Помилка:', error);
    } finally {
        showLoading(false);
    }
}

/**
 * Додає новий запис
 * АДАПТУЙТЕ itemData під структуру вашого API
 */
async function addItem(itemData) {
    try {
        showLoading(true);

        const response = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify(itemData)
        });

        if (!response.ok) {
            throw new Error(await getErrorText(response));
        }

        showMessage('✅ Запис успішно додано!', 'success');
        await loadData();

    } catch (error) {
        showMessage('❌ Помилка додавання: ' + error.message, 'error');
        console.error('Помилка:', error);
    } finally {
        showLoading(false);
    }
}

// ============================================
// ВІДОБРАЖЕННЯ ДАНИХ
// ============================================

/**
 * Показує дані на сторінці
 * АДАПТУЙТЕ під структуру ваших даних
 */
function displayData(items) {
    const container = document.getElementById('dataList');
    container.innerHTML = '';

    if (!items || items.length === 0) {
        container.innerHTML = `
            <div class="empty-state">
                📭 Даних поки немає. Додайте перший запис!
            </div>
        `;
        return;
    }

    items.forEach(item => {
        const itemDiv = document.createElement('div');
        itemDiv.className = 'item';

        // ВАЖЛИВО: Змініть item.field1, item.field2 на реальні поля вашого API.
        // Дані від користувача вставляємо через textContent, а не innerHTML:
        // так введений у форму HTML-код не виконається на сторінці.
        const title = document.createElement('h3');
        title.textContent = item.field1 || 'Без назви';

        const text = document.createElement('p');
        text.textContent = item.field2 || 'Без опису';

        itemDiv.append(title, text);
        container.appendChild(itemDiv);
    });
}

// ============================================
// ОБРОБКА ФОРМИ
// ============================================

document.getElementById('addForm').addEventListener('submit', async (event) => {
    event.preventDefault();

    // АДАПТУЙТЕ: отримайте значення з ВАШИХ полів вводу
    const field1 = document.getElementById('field1').value.trim();
    const field2 = document.getElementById('field2').value.trim();

    // АДАПТУЙТЕ: додайте валідацію відповідно до ваших вимог
    if (!field1 || field1.length < 2) {
        showMessage('❌ Перше поле повинно містити мінімум 2 символи', 'error');
        return;
    }

    // АДАПТУЙТЕ: структура об'єкта має відповідати вашому API
    const itemData = {
        field1: field1,
        field2: field2
        // Додайте інші поля за потреби
    };

    await addItem(itemData);

    // Очищуємо форму
    event.target.reset();
});

// ============================================
// ІНІЦІАЛІЗАЦІЯ
// ============================================

loadData();
```

### 3. Налаштуйте CORS у вашому API (за потреби)

Якщо сторінка `api-demo.html` відкривається через ваш Flask-сервер (як у кроці 2), сторінка і API мають одне джерело — **CORS налаштовувати не потрібно**.

CORS потрібен, якщо клієнтська частина відкривається з іншого джерела: окремим файлом `index.html`, через розширення Live Server у VS Code (порт 5500) тощо. У такому разі додайте підтримку CORS у Flask:

```python
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
CORS(app, resources={r"/api/*": {"origins": "*"}})  # Дозволяємо доступ лише до адрес /api/...
```

Встановіть flask-cors та оновіть список залежностей:

```bash
pip install flask-cors
pip freeze > requirements.txt
```

### 4. Тестування

1. **Запустіть API** - переконайтесь що він працює
2. **Відкрийте сторінку** `http://127.0.0.1:5000/api-demo`
3. **Перевірте Console** (F12) на наявність помилок
4. **Протестуйте функціональність:**
   - Чи завантажуються дані?
   - Чи можна додати новий ресурс?
   - Чи показуються повідомлення?
   - Що бачить користувач, якщо зупинити сервер і оновити список? (має з'явитися зрозуміле повідомлення про помилку)

### 5. Налагодження

**Відкрийте інструменти розробника:**

- Windows/Linux: `F12` або `Ctrl+Shift+I`
- Mac: `Cmd+Option+I`

**Важливі вкладки:**

- **Console** - помилки JavaScript
- **Network** - HTTP запити до API
- **Elements** - HTML структура

**Типові проблеми:**

1. **CORS помилка**
   ```
   Access to fetch has been blocked by CORS policy
   ```
   Рішення: сторінка і API мають різні джерела. Відкривайте сторінку через Flask-сервер або налаштуйте Flask-CORS (крок 3). Пам'ятайте, що `localhost` і `127.0.0.1` браузер вважає різними джерелами.

2. **404 Not Found**
   ```
   GET http://127.0.0.1:5000/api/endpoint 404
   ```
   Рішення: перевірте правильність URL і назву маршруту у Flask

3. **Network Error**
   ```
   Failed to fetch
   ```
   Рішення: переконайтесь що API запущений

4. **403 Forbidden на macOS**

   Порт 5000 у macOS займає служба AirPlay Receiver. Вимкніть її в системних налаштуваннях або запустіть Flask на іншому порту (`app.run(port=5001)`) і змініть адресу API.

5. **Cannot read properties of null (reading 'addEventListener')**

   Скрипт виконується раніше, ніж на сторінці з'явилася форма, або `id` елемента в HTML не збігається з `id` у JavaScript. Підключайте скрипт наприкінці сторінки та перевірте назви.

### 6. Підготовка звіту і захист (README.md)

Створіть у корені проєкту файл `README.md`. Рекомендована структура звіту:

- назва проєкту, склад команди;
- короткий опис демонстраційної сторінки (2–3 речення);
- які ендпоінти API використовує сторінка і з якими полями;
- де в проєкті розміщено HTML-шаблон і JavaScript-файл;
- скріншоти: список даних, додавання запису, повідомлення про успіх і про помилку;
- висновки (3–5 речень).

Як відповідь на завдання в LMS Moodle дати посилання на репозиторій з проєктом. Захистити лабораторну перед викладачем.

[👉 Здати лабораторну роботу](https://moodle.vcolnuft.volyn.ua/moodle/course/view.php?id=1426#section-2)

## ❓ Контрольні запитання

1. Що таке асинхронність у JavaScript? Навіщо вона потрібна?
2. Чим відрізняється GET запит від POST запиту?
3. Що таке JSON і навіщо він потрібен?
4. Що робить функція `fetch()`?
5. Навіщо потрібен `await` перед `fetch()`?
6. Що таке CORS і чому виникають помилки CORS?
7. Як у коді обробляються помилки при роботі з API? Чому недостатньо лише блоку `catch`?

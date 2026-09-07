# Лабораторна робота 6 Інтеграція фронтенду з API та обробка помилок

## 🎯 Мета роботи

Навчитись підключати frontend до створеного API, виконувати базові операції отримання та відправки даних, розуміти як працює асинхронний JavaScript.

## ✅ Завдання

Створити просту вебсторінку, яка:

- показує список ресурсів з вашого API;
- дозволяє додати новий ресурс через форму;
- показує повідомлення про успіх або помилку;
- має базове оформлення.

!!! tip "Увага"
    Ця лабораторна використовує API, створений у лабораторній роботі 5. Переконайтесь, що ваш API запущений і працює.

## 🖥️ Програмне забезпечення

- Git [git-scm.com](https://git-scm.com) - розподілена система контролю версій;
- GitHub [github.com](https://github.com) - хмарна платформа для хостингу Git репозиторіїв;
- Visual Studio Code [code.visualstudio.com](https://code.visualstudio.com) - редактор коду з підтримкою Git;
- реляційна СКБД SQLite [sqlite.org](https://sqlite.org/)
- GitHub Desktop [desktop.github.com](https://desktop.github.com) - графічний клієнт Git (опціонально);
- мова програмування Python [https://www.python.org/](https://www.python.org/);
- вебфреймворк Flask [https://flask.palletsprojects.com](https://flask.palletsprojects.com).

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

### Промiси (Promises)

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
async function getDat() {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    console.log(data);
}
```

Слово `await` означає "почекай, поки виконається". Але воно працює тільки всередині функції з `async`.

### Fetch API — отримання даних

`fetch()` — це функція для відправки HTTP запитів. Найпростіший варіант:

```javascript
fetch('http://localhost:5000/api/items')
```

Це відправить GET запит (запит на отримання даних) на вказану адресу.

### HTTP методи

- **GET** — отримати дані (як прочитати книгу);
- **POST** — створити щось нове (як написати нову сторінку);
- **PUT** — оновити існуюче (як виправити текст);
- **DELETE** — видалити (як викинути сторінку).

Для GET не потрібно нічого додаткового. Для POST потрібно вказати, які дані відправляємо:

```javascript
fetch('http://localhost:5000/api/items', {
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

## ▶️ Хід роботи

Приклад виконання лабораторної роботи ([:fontawesome-solid-archive: навчальний додаток](assets/lab06-flaskProject2API-frontend.zip){: download="lab06-flaskProject2API-frontend.zip" })


### Крок 1. Підготовка проєкту

1. **Переконайтесь що ваш API працює**
   - Запустіть ваш Flask додаток з лабораторної роботи 5
   - Перевірте в браузері або Postman, що ендпоінти відповідають
   - Запишіть базову адресу API (наприклад: `http://localhost:5000/api`)

2. **Визначте структуру вашого API**
   - Які ендпоінти у вас є? (наприклад: `/api/products`, `/api/orders`, `/api/feedback`)
   - Які поля мають ваші об'єкти? (наприклад: `name`, `price`, `email`, `message`)
   - Які операції підтримуються? (GET, POST, PUT, DELETE)


### Крок 2. Створіть HTML структуру та JavaScript логіку для фронтенду

Створіть демонастраційний файл `api-demo.html` на основі шаблону `base.html` для демонстрації роботи API.

**Важливо:** Адаптуйте структуру сторінки під ваші дані. Наприклад:
- Для продуктів: `name`, `price`, `description`
- Для відгуків: `name`, `email`, `message`
- Для замовлень: `customer_name`, `address`, `phone`

Для створення повноцінного фронтенду додайте JavaScript логіку для роботи з API. Наприклад:

```javascript
// ============================================
// НАЛАШТУВАННЯ - ЗМІНІТЬ ПІД СВІЙ ПРОЄКТ
// ============================================

// ВАЖЛИВО: Вкажіть адресу ВАШОГО API
const API_URL = 'http://localhost:5000/api/your-endpoint';

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
            throw new Error(`HTTP помилка! Статус: ${response.status}`);
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
            throw new Error(`HTTP помилка! Статус: ${response.status}`);
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

        // ВАЖЛИВО: Змініть item.field1, item.field2 на реальні поля вашого API
        itemDiv.innerHTML = `
            <h3>${item.field1 || 'Без назви'}</h3>
            <p>${item.field2 || 'Без опису'}</p>
        `;

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

window.addEventListener('load', () => {
    loadData();
});
```

### Крок 3. Налаштуйте CORS у вашому API

Щоб frontend міг підключитись до API, переконайтесь що у вашому Flask додатку є підтримка CORS:

```python
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # Дозволяємо доступ до API

# Або більш обмежений варіант:
# CORS(app, resources={r"/api/*": {"origins": "*"}})
```

Встановіть flask-cors якщо потрібно:
```bash
pip install flask-cors
```

### Крок 4. Тестування

1. **Запустіть API** - переконайтесь що він працює
2. **Перевірте Console** (F12) на наявність помилок
3. **Протестуйте функціональність:**
   - Чи завантажуються дані?
   - Чи можна додати новий ресурс?
   - Чи показуються повідомлення?

### Крок 5. Налагодження

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
   Рішення: додайте `CORS(app)` у Flask

2. **404 Not Found**
   ```
   GET http://localhost:5000/api/endpoint 404
   ```
   Рішення: перевірте правильність URL

3. **Network Error**
   ```
   Failed to fetch
   ```
   Рішення: переконайтесь що API запущений

### Крок 6. Підготовка звіту і захист (README.md)

Створіть файл `README.md` з наступною структурою:

```markdown
    # Лабораторна робота 6

    **Студент:** Ваше ім'я
    **Група:** Назва групи

    ## 📋 Опис проєкту

    Коротко опишіть що робить ваш застосунок (2-3 речення).


    ## 📁 Структура проєкту

    ```
    project/
    ├── index.html       # Головна сторінка
    ├── style.css        # Стилі
    ├── script.js        # JavaScript логіка
    ├── README.md        # Цей файл
    └── backend/         # Flask API (з лаб. роботи 5)
        ├── app.py
        └── ...
    ```

    ## 🔌 API Endpoints

    Опишіть які ендпоінти використовуються:

    ### GET /api/your-endpoint
    Отримує список всіх записів.

    **Відповідь:**
    ```json
    [
      {
        "id": 1,
        "field1": "значення",
        "field2": "значення"
      }
    ]
    ```

    ### POST /api/your-endpoint
    Створює новий запис.

    **Тіло запиту:**
    ```json
    {
      "field1": "значення",
      "field2": "значення"
    }
    ```

    ## 📸 Скріншоти

    ### Головна сторінка
    ![Скріншот головної сторінки](screenshots/main.png)

    ### Додавання запису
    ![Скріншот форми](screenshots/add-form.png)

    ### Повідомлення про успіх
    ![Скріншот повідомлення](screenshots/success.png)

    ## 🔗 Посилання
    - [Посилання на GitHub](https://github.com/username/repo)

    ## ✅ Висновки

    Коротко опишіть що ви навчились робити в цій лабораторній роботі (3-5 речень).
```

Як відповідь на завдання в LMS Moodle дати посилання на репозиторій з проєктом. Захистити лабораторну перед викладачем.

[👉 Здати лабораторну роботу](https://moodle.vcolnuft.volyn.ua/moodle/course/view.php?id=1426#section-2){ .md-button .md-button--primary }

## ❓ Контрольні запитання

1. Що таке асинхронність у JavaScript? Навіщо вона потрібна?
2. Чим відрізняється GET запит від POST запиту?
3. Що таке JSON і навіщо він потрібен?
4. Що робить функція `fetch()`?
5. Навіщо потрібен `await` перед `fetch()`?
6. Що таке CORS і чому виникають помилки CORS?
7. Як у коді обробляються помилки при роботі з API?

# Звіт з лабораторної роботи 4

## Реалізація бази даних для вебпроєкту

### Інформація про команду
- Назва команди:

- Учасники:
  - ПІБ (роль у команді)
  - ПІБ (роль у команді)
  - ПІБ (роль у команді)
  - ПІБ (роль у команді)

## Завдання

### Обрана предметна область

Опишіть предметну область вашого вебзастосунку та які дані потрібно зберігати (наприклад: інтернет-магазин, система бронювання, блог з коментарями тощо).

### Реалізовані вимоги

Вкажіть, які рівні завдань було виконано:

- [ ] Рівень 1: Створено базу даних SQLite з таблицею для відгуків, реалізовано базові CRUD операції, створено адмін-панель для перегляду та видалення відгуків, додано функціональність магазину з таблицями для товарів та замовлень
- [ ] Рівень 2: Створено додаткову таблицю, релевантну предметній області, реалізовано роботу з новою таблицею через адмін-панель, інтегровано функціональність у застосунок
- [ ] Рівень 3: Розширено функціональність двома додатковими функціями, що суттєво покращують користувацький досвід

## Хід виконання роботи

### Підготовка середовища розробки

Опишіть процес налаштування:

- Версія Python
- Встановлені бібліотеки (Flask, SQLite3 тощо)
- Інші використані інструменти та розширення

### Структура проєкту

Наведіть структуру файлів та директорій вашого проєкту:

```
project/
├── app.py
├── models.py
├── routes/
│   ├── __init__.py
│   ├── admin.py
│   ├── feedback.py
│   └── shop.py
├── templates/
│   ├── base.html
│   ├── admin/
│   ├── feedback/
│   └── shop/
├── static/
│   ├── css/
│   ├── js/
│   └── images/
├── db.sqlite
└── lab-reports/
    └── lab04-report-student-id.md
```

### Проектування бази даних

#### Схема бази даних

Опишіть структуру вашої бази даних:

```
Таблиця "feedback":
- id (INTEGER, PRIMARY KEY)
- name (TEXT, NOT NULL)
- email (TEXT, NOT NULL)
- message (TEXT, NOT NULL)
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

Таблиця "products":
- id (INTEGER, PRIMARY KEY)
- name (TEXT, NOT NULL)
- description (TEXT)
- price (REAL, NOT NULL)
- stock (INTEGER, DEFAULT 0)
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

Таблиця "orders":
- id (INTEGER, PRIMARY KEY)
- customer_name (TEXT, NOT NULL)
- customer_email (TEXT, NOT NULL)
- total_amount (REAL, NOT NULL)
- status (TEXT, DEFAULT 'pending')
- created_at (TIMESTAMP, DEFAULT CURRENT_TIMESTAMP)

[Додайте інші таблиці, якщо реалізовано]
```



### Опис реалізованої функціональності

#### Система відгуків

Опишіть реалізацію системи збору та відображення відгуків користувачів.

#### Магазин

Опишіть функціональність магазину:

- Відображення каталогу товарів
- Додавання товарів до кошика
- Оформлення замовлення
- Управління товарами через адмін-панель

#### Адміністративна панель

Опишіть можливості адмін-панелі:

- Перегляд відгуків
- Управління товарами
- Управління замовленнями
- Інші функції

#### Додаткова функціональність (якщо реалізовано)

Опишіть додаткові таблиці та функції, які було реалізовано для рівнів 2 та 3.

## Ключові фрагменти коду

### Ініціалізація бази даних

Наведіть код створення таблиць у файлі `models.py`:

```python
import sqlite3

def init_db():
    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()

    # Створення таблиці feedback
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS feedback (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL,
            message TEXT NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')

    # Створення інших таблиць...

    conn.commit()
    conn.close()
```

### CRUD операції

Наведіть приклади реалізації CRUD операцій:

#### Створення (Create)

```python
def add_feedback(name, email, message):
    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()
    cursor.execute(
        'INSERT INTO feedback (name, email, message) VALUES (?, ?, ?)',
        (name, email, message)
    )
    conn.commit()
    conn.close()
```

#### Читання (Read)

```python
def get_all_feedback():
    conn = sqlite3.connect('db.sqlite')
    conn.row_factory = sqlite3.Row
    cursor = conn.cursor()
    cursor.execute('SELECT * FROM feedback ORDER BY created_at DESC')
    feedback = cursor.fetchall()
    conn.close()
    return feedback
```

#### Оновлення (Update)

```python
def update_order_status(order_id, status):
    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()
    cursor.execute(
        'UPDATE orders SET status = ? WHERE id = ?',
        (status, order_id)
    )
    conn.commit()
    conn.close()
```

#### Видалення (Delete)

```python
def delete_feedback(feedback_id):
    conn = sqlite3.connect('db.sqlite')
    cursor = conn.cursor()
    cursor.execute('DELETE FROM feedback WHERE id = ?', (feedback_id,))
    conn.commit()
    conn.close()
```

### Маршрутизація

Наведіть приклади маршрутів для роботи з базою даних:

```python
@app.route('/feedback', methods=['GET', 'POST'])
def feedback():
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        message = request.form['message']
        add_feedback(name, email, message)
        flash('Дякуємо за ваш відгук!', 'success')
        return redirect(url_for('feedback'))
    return render_template('feedback.html')
```

### Робота зі зв'язками між таблицями

Наведіть приклад запиту з використанням JOIN для отримання пов'язаних даних:

```python
def get_order_details(order_id):
    conn = sqlite3.connect('db.sqlite')
    conn.row_factory = sqlite3.Row
    cursor = conn.cursor()
    cursor.execute('''
        SELECT o.*, oi.quantity, p.name, p.price
        FROM orders o
        JOIN order_items oi ON o.id = oi.order_id
        JOIN products p ON oi.product_id = p.id
        WHERE o.id = ?
    ''', (order_id,))
    details = cursor.fetchall()
    conn.close()
    return details
```

## Розподіл обов'язків у команді

Опишіть внесок кожного учасника команди:

- ПІБ учасника 1: опис виконаних завдань (наприклад, проектування схеми БД, реалізація моделей)
- ПІБ учасника 2: опис виконаних завдань (наприклад, реалізація маршрутів, CRUD операції)
- ПІБ учасника 3: опис виконаних завдань (наприклад, створення шаблонів, адмін-панель)
- ПІБ учасника 4: опис виконаних завдань (наприклад, тестування, документація)

## Скріншоти

Додайте скріншоти основних функцій вашого вебзастосунку:

### Форма зворотного зв'язку

![Форма зворотного зв'язку](шлях/до/скріншоту)

### Каталог товарів

![Каталог товарів](шлях/до/скріншоту)

### Адміністративна панель

![Адмін-панель](шлях/до/скріншоту)

### Управління замовленнями

![Управління замовленнями](шлях/до/скріншоту)

### Додаткова функціональність

![Додаткова функція](шлях/до/скріншоту)

## Тестування

### Сценарії тестування

Опишіть, які сценарії ви тестували:

1. Додавання нового відгуку та перевірка його відображення в адмін-панелі
2. Створення товару, додавання його до кошика та оформлення замовлення
3. Зміна статусу замовлення через адмін-панель
4. Видалення записів з бази даних
5. Перевірка валідації даних


## Висновки

Опишіть:

- Що вдалося реалізувати успішно
- Які навички роботи з базами даних отримали
- Які труднощі виникли при проектуванні схеми БД
- Як організували командну роботу
- Які покращення можна внести в майбутньому

Очікувана оцінка: [4-12 балів]

Обґрунтування: [Чому заслуговуєте на цю оцінку]

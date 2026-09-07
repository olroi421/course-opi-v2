# Лабораторна робота 3 Розробка базового вебпроєкту

## 🎯 Мета роботи

Засвоїти основні принципи створення веб-застосунків з використанням сучасних технологій та фреймворків та навчитися проектувати та реалізовувати базову архітектуру веб-додатку.

## ✅ Завдання

### Рівень 1

Обрати довільну предметну область. Створити вебзастосунок з наступними сторінками: “Головна”, “Про нас” з використанням фреймворку Flask. Контент сторінок повинен відповідати предметній області.

### Рівень 2

Створити не менше двох додаткових статичних сторінок, забезпечити відображення в меню, коректну верстку та адаптивність

## 🖥️ Програмне забезпечення

- Git [git-scm.com](https://git-scm.com) - розподілена система контролю версій;
- GitHub [github.com](https://github.com) - хмарна платформа для хостингу Git репозиторіїв;
- Visual Studio Code [code.visualstudio.com](https://code.visualstudio.com) - редактор коду з підтримкою Git;
- GitHub Desktop [desktop.github.com](https://desktop.github.com) - графічний клієнт Git (опціонально);
- мова програмування Python [https://www.python.org/](https://www.python.org/);
- вебфреймворк Flask [https://flask.palletsprojects.com](https://flask.palletsprojects.com).

## 👥 Форма виконання роботи

Форма виконання роботи **групова** (3-4 особи в команді).

## 📝 Критерії оцінювання

- середній рівень (оцінка “задовільно”, 4-6) - виконано основне завдання, з адаптивною версткою сторінок та коректним відображенням головного меню. Допускаються незначні помилки у коді або оформленні.
- достатній рівень (оцінка “добре”, 7-9) - виконано всі вимоги достатнього рівня та виконано додаткове завдання, під час захисту допускаються невеликі помилки та неповне розуміння деяких аспектів роботи.
- високий рівень (оцінка “відмінно”, 10-12) - виконано всі вимоги достатнього рівня та виконано додаткове завдання, під час захисту продемонстровано глибоке розуміння принципів роботи технологій вебзастосунку, код чистий, добре структурований та прокоментований, здобувач освіти може пояснити кожен аспект роботи та обґрунтувати прийняті рішення, проявлено творчий підхід до виконання завдання.


## ⏰ Політика щодо дедлайнів

При порушенні встановленого терміну здачі лабораторної роботи максимальна можлива оцінка становить 9 балів ("добре"), незалежно від якості виконаної роботи. Винятки можливі лише за поважних причин, підтверджених документально.

## 📚 Теоретичні відомості

### Python для веб-розробки

Python - це високорівнева мова програмування загального призначення, яка широко використовується у веб-розробці завдяки своїй простоті та потужності.

Ключові особливості Python для веб-розробки:

- Простий і читабельний синтаксис
- Велика стандартна бібліотека
- Багато фреймворків для веб-розробки (Flask, Django, FastAPI)
- Підтримка асинхронного програмування (asyncio)
- Велика спільнота та багато сторонніх бібліотек

Приклад простого веб-сервера на Python:

```python
from http.server import HTTPServer, SimpleHTTPRequestHandler

def run(server_class=HTTPServer, handler_class=SimpleHTTPRequestHandler):
    server_address = ('', 8000)
    httpd = server_class(server_address, handler_class)
    print('Запуск сервера на порту 8000...')
    httpd.serve_forever()

if __name__ == "__main__":
    run()

```

### Flask: легкий веб-фреймворк

Flask - це мікрофреймворк для Python, який дозволяє швидко створювати веб-додатки. Він надає базовий функціонал для обробки HTTP-запитів, маршрутизації та роботи з шаблонами.

Основні концепції Flask:

а) Створення додатку:

```python
from flask import Flask
app = Flask(__name__)

```

б) Маршрутизація:

```python
@app.route('/')
def home():
    return 'Головна сторінка'

```

в) Обробка параметрів URL:

```python
@app.route('/user/<username>')
def show_user_profile(username):
    return f'Користувач: {username}'

```

г) Обробка HTTP-методів:

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_login()
    else:
        return show_login_form()

```

д) Робота з запитами та відповідями:

```python
from flask import request, jsonify

@app.route('/api/data')
def get_data():
    data = request.args.get('key')
    return jsonify({'result': data})

```

### Jinja2: система шаблонів

Jinja2 - це потужний шаблонізатор для Python, який використовується за замовчуванням у Flask. Він дозволяє вставляти динамічний контент в HTML-сторінки.

Основні концепції Jinja2:

а) Вставка змінних:

```html
<h1>Привіт, {{ name }}!</h1>

```

б) Умовні конструкції:

```html
{% if user %}
    <p>Ласкаво просимо, {{ user.name }}!</p>
{% else %}
    <p>Будь ласка, увійдіть в систему.</p>
{% endif %}

```

в) Цикли:

```html
<ul>
{% for item in items %}
    <li>{{ item }}</li>
{% endfor %}
</ul>

```

г) Наслідування шаблонів:

```html
<!-- base.html -->
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    {% block content %}{% endblock %}
</body>
</html>

<!-- page.html -->
{% extends "base.html" %}
{% block title %}Моя сторінка{% endblock %}
{% block content %}
    <h1>Вітаємо на моїй сторінці!</h1>
{% endblock %}

```

д) Фільтри:

```html
{{ name|capitalize }}
{{ list|join(', ') }}

```

е) Макроси (для повторного використання коду):

```html
{% macro input(name, value='', type='text') %}
    <input type="{{ type }}" name="{{ name }}" value="{{ value|e }}">
{% endmacro %}

{{ input('username') }}
{{ input('password', type='password') }}

```

### Взаємодія Flask і Jinja2

Flask автоматично інтегрує Jinja2 для рендерингу шаблонів. Ось приклад, як це працює:

```python
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/hello/<name>')
def hello(name):
    return render_template('hello.html', name=name)

```

У цьому прикладі `render_template` шукає файл `hello.html` у папці `templates` і передає змінну `name` в шаблон.

Відповідний шаблон `hello.html` може виглядати так:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Привітання</title>
</head>
<body>
    <h1>Привіт, {{ name }}!</h1>
</body>
</html>

```

### Контекст запиту у Flask

Flask створює контекст запиту для кожного HTTP-запиту. Це дозволяє отримувати доступ до інформації про поточний запит, сесію користувача тощо.

Приклад використання контексту запиту:

```python
from flask import request, session

@app.route('/user')
def user_info():
    user_agent = request.headers.get('User-Agent')
    user_id = session.get('user_id')
    return f'User Agent: {user_agent}, User ID: {user_id}'

```

### Tailwind CSS

У прикладі лабораторної роботи для забезпечення адаптивності сторінок використовується Tailwind CSS. Його використання не є обовʼязковим.

Tailwind CSS - це  CSS фреймворк, який дозволяє швидко створювати кастомізовані дизайни без написання власного CSS. Він надає набір готових класів, які можна застосовувати безпосередньо в HTML.

Основні концепції Tailwind CSS:

a) Утилітарний підхід: Замість створення окремих CSS класів для кожного елемента, ви використовуєте комбінацію утилітарних класів прямо в HTML.

Приклад:

```html

<div class="p-4 m-2 bg-blue-500 text-white rounded shadow">
  Це синій блок з відступами, тінню та заокругленими кутами
</div>

```

b) Респонсивний дизайн: Tailwind надає префікси для створення респонсивних дизайнів: `sm:`, `md:`, `lg:`, `xl:`.

Приклад:

```html

<div class="text-center sm:text-left md:text-right">
  Цей текст буде вирівняний по-різному на різних розмірах екрану
</div>

```

c) Псевдокласи: Можна застосовувати стилі до псевдокласів, використовуючи префікси `hover:`, `focus:`, `active:`тощо.

Приклад:

```html

<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded">
  Наведіть на мене
</button>

```

d) Кастомізація: Tailwind можна легко налаштувати під конкретний проєкт через файл конфігурації.

Приклад `tailwind.config.js`:

```jsx

module.exports = {
  theme: {
    extend: {
      colors: {
        'brand-blue': '#1992d4',
      },
    },
  },
}

```

e) Компоненти: Хоча Tailwind не надає готових компонентів, ви можете створювати власні, комбінуючи утилітарні класи.

Приклад:

```html

<button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline">
  Кнопка
</button>

```

f) Очищення невикористаних стилів: Tailwind використовує PurgeCSS для видалення невикористаних стилів у продакшн-збірці, що значно зменшує розмір CSS файлу.

Переваги Tailwind CSS:

1. Швидкість розробки: Не потрібно перемикатися між HTML і CSS файлами.
2. Консистентність: Використання заздалегідь визначених класів забезпечує єдиний вигляд по всьому проєкту.
3. Гнучкість: Легко створювати унікальні дизайни без написання кастомного CSS.
4. Продуктивність: Менший розмір CSS файлу завдяки видаленню невикористаних стилів.

Недоліки Tailwind CSS:

1. Крива навчання: Потрібен час, щоб запам'ятати всі доступні класи.
2. Довгі рядки класів: HTML може стати важчим для читання через велику кількість класів.
3. Залежність від фреймворку: Стилі тісно пов'язані з Tailwind, що може ускладнити перехід на інші методології в майбутньому.

Для використання Tailwind з Flask, ви можете включити його через CDN у ваш базовий шаблон:

```html

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.tailwindcss.com"></script>
    <title>{% block title %}{% endblock %} - Мій Flask Додаток</title>
</head>

```

Це дозволить вам використовувати класи Tailwind у всіх ваших шаблонах без додаткової конфігурації.

## 🟣 Ресурси

- [Welcome to Flask — Flask Documentation (3.0.x)](https://flask.palletsprojects.com/en/3.0.x/#user-s-guide)
- [Flask www.youtube.com](https://www.youtube.com/results?search_query=flask)
- [Installation - Tailwind CSS](https://tailwindcss.com/docs/installation)
- [Tailwind CSS www.youtube.com](https://www.youtube.com/results?search_query=tailwind+css)

## ▶️ Хід роботи

1. Підготовка середовища розробки:
    1. Завантажте та встановіть Python з офіційного сайту: [https://www.python.org/downloads/](https://www.python.org/downloads/)
    2. Переконайтеся, що Python доданий до системного PATH.
    3. Виконайте команду для встановлення Flask:
    ```bash
    pip install flask
    ```
1. Створення базової структури Flask додатку:
    1. Завантажити приклад виконання лабораторної роботи ([:fontawesome-solid-archive: навчальний додаток](assets/lab03-flaskProject.zip){: download="lab03-flaskProject.zip" })
    2. В директорію з проєктом розархівувати навчальний додаток, який можна брати за основу.
    3. Запустити навчальний додаток, переконатись в його працездатності і працездатності середовища розробки.
    ```bash
    python app.py
    ```
1. Виконання завдань лабораторної роботи:
    1. Реалізувати вебзастосунок відповідно до завдання.
    2. Внесок кожного учасника команди повинен бути чітким та зрозумілим,.
2. Підготовка звіту лабораторної роботи:
    1. створити директорію `lab-reports/` і додати `lab03-report-student-id.md` для кожного учасника команди відповідно до шаблону звіту ([:fontawesome-solid-download: завантажити шаблон](assets/lab03-report-template.download){: download="lab03-report.md" });
    2. як відповідь на завдання в LMS Moodle дати посилання на репозиторій з проєктом;
    3. захистити лабораторну перед викладачем.

[👉 Здати лабораторну роботу](https://moodle.vcolnuft.volyn.ua/moodle/course/view.php?id=1426#section-2){ .md-button .md-button--primary }

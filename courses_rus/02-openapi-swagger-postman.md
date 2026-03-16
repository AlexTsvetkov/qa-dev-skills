# Курс 02: OpenAPI, Swagger и Postman

## 📖 Что вы изучите

- Что такое API и почему спецификации API важны
- Что такое спецификация OpenAPI и как её читать
- Что такое Swagger и как использовать Swagger UI
- Что такое Postman и как выполнять API-запросы с его помощью
- Как перейти от документации API к реальному тестированию

---

## 1. Что такое API?

**API (Application Programming Interface)** — это набор правил, позволяющий различным программным системам взаимодействовать друг с другом.

Представьте это как **меню в ресторане**:
- **Меню** (спецификация API) сообщает, какие блюда (эндпоинты) доступны
- **Официант** (HTTP) несёт ваш заказ (запрос) на кухню (сервер) и приносит обратно еду (ответ)
- Вам не нужно знать, как работает кухня — достаточно знать меню

### REST API

Большинство современных веб-API следуют стилю **REST** (Representational State Transfer):

- Использует **HTTP-методы** (GET, POST, PUT, DELETE) для выполнения операций
- Работает с **ресурсами**, идентифицируемыми URL (например, `/api/users`, `/api/orders/42`)
- Использует **JSON** в качестве формата данных (обычно)
- **Без сохранения состояния (Stateless)** — каждый запрос содержит всю необходимую информацию

```
Ресурс: /api/users

GET    /api/users       → Список всех пользователей
POST   /api/users       → Создать нового пользователя
GET    /api/users/42    → Получить пользователя 42
PUT    /api/users/42    → Обновить пользователя 42
DELETE /api/users/42    → Удалить пользователя 42
```

---

## 2. Что такое OpenAPI?

**Спецификация OpenAPI (OAS)** — это стандартный формат для описания REST API. Это документ (обычно в YAML или JSON), который описывает:

- Какие эндпоинты доступны
- Какие HTTP-методы поддерживает каждый эндпоинт
- Какие параметры принимает каждый эндпоинт
- Как должно выглядеть тело запроса
- Какие ответы ожидать
- Требования аутентификации

### Почему это важно для QA:

- 📋 Это ваш **контракт API** — что API обещает делать
- 🧪 Вы можете **генерировать тест-кейсы** на его основе
- ✅ Вы можете **валидировать** реальные ответы API по спецификации
- 📝 Это всегда **актуальная** документация (если правильно ведётся)

### Пример документа OpenAPI (YAML):

```yaml
openapi: 3.0.3
info:
  title: User Management API
  description: API для управления пользователями
  version: 1.0.0

servers:
  - url: https://api.example.com/v1
    description: Production сервер
  - url: https://staging-api.example.com/v1
    description: Staging сервер

paths:
  /users:
    get:
      summary: Получить всех пользователей
      description: Возвращает список всех пользователей
      parameters:
        - name: role
          in: query
          required: false
          schema:
            type: string
            enum: [admin, user, qa]
          description: Фильтр пользователей по роли
        - name: page
          in: query
          required: false
          schema:
            type: integer
            default: 1
          description: Номер страницы
      responses:
        '200':
          description: Успешный ответ
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
        '401':
          description: Не авторизован
    
    post:
      summary: Создать нового пользователя
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: Пользователь успешно создан
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          description: Некорректные данные запроса
        '409':
          description: Пользователь с таким email уже существует

  /users/{id}:
    get:
      summary: Получить пользователя по ID
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
          description: ID пользователя
      responses:
        '200':
          description: Пользователь найден
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          description: Пользователь не найден

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
          example: 42
        name:
          type: string
          example: "Настя"
        email:
          type: string
          format: email
          example: "nastya@example.com"
        role:
          type: string
          enum: [admin, user, qa]
          example: "qa"
        createdAt:
          type: string
          format: date-time
    
    CreateUserRequest:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
          minLength: 2
          maxLength: 100
        email:
          type: string
          format: email
        role:
          type: string
          enum: [admin, user, qa]
          default: user
```

### Чтение спецификации OpenAPI — ключевые разделы:

| Раздел | Что описывает |
|--------|---------------|
| `info` | Название API, версия, описание |
| `servers` | Базовые URL для разных окружений |
| `paths` | Все эндпоинты и их операции |
| `parameters` | Входные данные (query, path, header параметры) |
| `requestBody` | Тело запроса POST/PUT/PATCH |
| `responses` | Что API возвращает (по кодам состояния) |
| `components/schemas` | Переиспользуемые модели данных (объекты) |

### Расположение параметров:

| Расположение (`in`) | Пример | Описание |
|---------------------|--------|----------|
| `path` | `/users/{id}` | Часть URL-пути |
| `query` | `/users?role=qa` | После `?` в URL |
| `header` | `Authorization: Bearer token` | В HTTP-заголовке |
| `cookie` | `session=abc123` | В cookie |

---

## 3. Что такое Swagger?

**Swagger** — это набор инструментов, построенных вокруг спецификации OpenAPI:

| Инструмент | Назначение |
|-----------|-----------|
| **Swagger UI** | Интерактивная документация API в браузере |
| **Swagger Editor** | Написание и редактирование спецификаций OpenAPI онлайн |
| **Swagger Codegen** | Генерация клиентского/серверного кода из спецификаций |

### Swagger UI

Swagger UI отображает вашу спецификацию OpenAPI как красивую интерактивную веб-страницу, где можно:

- 📖 **Читать** документацию API
- 🧪 **Тестировать** эндпоинты API прямо в браузере
- 📋 **Смотреть** примеры запросов/ответов
- 🔒 **Аутентифицироваться** и тестировать защищённые эндпоинты

### Как использовать Swagger UI:

1. Откройте страницу Swagger UI (обычно по адресу `/swagger-ui.html` или `/api-docs`)
2. Просмотрите доступные эндпоинты
3. Нажмите на эндпоинт, чтобы развернуть его
4. Нажмите **"Try it out"**
5. Заполните параметры и тело запроса
6. Нажмите **"Execute"**
7. Посмотрите ответ (код состояния, заголовки, тело)

### 🧠 Советы QA для Swagger UI:

- **Проверяйте, соответствует ли Swagger реальному поведению API** — документация Swagger может быть устаревшей!
- **Тестируйте граничные случаи**, которые Swagger может не показывать (пустые строки, очень длинные значения, спецсимволы)
- **Изучайте схемы** — они сообщают об обязательных полях, типах данных и правилах валидации
- **Пробуйте вызвать разные коды состояния** — можете ли вы получить ответы 400, 404, 409?

---

## 4. Что такое Postman?

**Postman** — это самый популярный инструмент для тестирования API. Он предоставляет удобный интерфейс для:

- 📤 **Отправки HTTP-запросов** (GET, POST, PUT, DELETE и др.)
- 📥 **Просмотра ответов** (статус, заголовки, тело)
- 📁 **Организации запросов** в коллекции
- 🔄 **Автоматизации тестов** с помощью тестовых скриптов
- 🌍 **Управления окружениями** (dev, staging, production)
- 📊 **Запуска коллекций** для автоматизированного тестирования

### Установка Postman

1. Перейдите на [postman.com/downloads](https://www.postman.com/downloads/)
2. Скачайте и установите десктопное приложение
3. Создайте бесплатную учётную запись (необязательно, но рекомендуется)

### Обзор интерфейса Postman:

```
┌──────────────────────────────────────────────────────────┐
│  Коллекции    │  Вкладка запроса                          │
│  ───────────  │  ┌──────────────────────────────────────┐ │
│  📁 Мой API   │  │ [GET ▼] [https://api.example.com/u.. │ │
│   ├─ Get Users│  │                          [Send]       │ │
│   ├─ Create   │  ├──────────────────────────────────────┤ │
│   └─ Delete   │  │ Params │ Auth │ Headers │ Body │ ... │ │
│               │  ├──────────────────────────────────────┤ │
│  📁 Другой    │  │                                      │ │
│               │  │  (Область настройки запроса)           │ │
│               │  │                                      │ │
│  Окружения    │  ├──────────────────────────────────────┤ │
│  ───────────  │  │ Ответ                                 │ │
│  🌍 Dev       │  │  Body │ Headers │ Test Results │      │ │
│  🌍 Staging   │  │  ┌────────────────────────────┐      │ │
│  🌍 Prod      │  │  │ { "id": 42, "name": ... }  │      │ │
│               │  │  └────────────────────────────┘      │ │
│               │  │  Status: 200 OK  Time: 142ms         │ │
└──────────────────────────────────────────────────────────┘
```

### Ваш первый запрос в Postman:

#### Шаг 1: Создайте новый запрос
- Нажмите **"+"** чтобы открыть новую вкладку
- Или нажмите **"New" → "HTTP Request"**

#### Шаг 2: Настройте запрос

**GET-запрос:**
1. Выберите метод: **GET**
2. Введите URL: `https://jsonplaceholder.typicode.com/users`
3. Нажмите **Send**
4. Посмотрите ответ ниже

**POST-запрос:**
1. Выберите метод: **POST**
2. Введите URL: `https://jsonplaceholder.typicode.com/posts`
3. Перейдите на вкладку **Body**
4. Выберите **raw** и **JSON**
5. Введите тело:
```json
{
  "title": "Мой первый пост",
  "body": "Привет из Postman!",
  "userId": 1
}
```
6. Нажмите **Send**
7. Проверьте ответ — должен быть `201 Created`

### Вкладки настройки запроса:

| Вкладка | Назначение |
|---------|-----------|
| **Params** | Добавить параметры запроса (`?key=value`) |
| **Authorization** | Настроить аутентификацию (Bearer token, Basic Auth и др.) |
| **Headers** | Добавить пользовательские заголовки |
| **Body** | Задать тело запроса (для POST, PUT, PATCH) |
| **Pre-request Script** | JavaScript, выполняемый перед запросом |
| **Tests** | JavaScript для валидации ответа |
| **Settings** | Настройки конкретного запроса |

### Аутентификация в Postman:

Перейдите на вкладку **Authorization** и выберите тип:

| Тип аутентификации | Как работает |
|-------------------|-------------|
| **No Auth** | Без аутентификации |
| **API Key** | Отправляет ключ как заголовок или query-параметр |
| **Bearer Token** | Отправляет заголовок `Authorization: Bearer <token>` |
| **Basic Auth** | Отправляет логин:пароль в кодировке Base64 |
| **OAuth 2.0** | Полный OAuth-поток |

### Написание тестов в Postman:

На вкладке **Tests** можно писать JavaScript для валидации ответов:

```javascript
// Проверка кода состояния
pm.test("Код состояния 200", function () {
    pm.response.to.have.status(200);
});

// Проверка времени ответа
pm.test("Время ответа менее 500мс", function () {
    pm.expect(pm.response.responseTime).to.be.below(500);
});

// Проверка тела ответа
pm.test("Ответ имеет правильную структуру", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData).to.be.an('array');
    pm.expect(jsonData.length).to.be.greaterThan(0);
});

// Проверка конкретного поля
pm.test("У первого пользователя есть email", function () {
    const jsonData = pm.response.json();
    pm.expect(jsonData[0]).to.have.property('email');
});

// Проверка заголовка Content-Type
pm.test("Content-Type — JSON", function () {
    pm.response.to.have.header("Content-Type", "application/json; charset=utf-8");
});
```

---

## 5. Из Swagger в Postman

Вы можете импортировать спецификацию OpenAPI/Swagger напрямую в Postman:

### Как импортировать:

1. В Postman нажмите **"Import"** (вверху слева)
2. Выберите один из вариантов:
   - **File** — загрузите файл OpenAPI YAML/JSON
   - **Link** — вставьте URL спецификации (например, `https://api.example.com/v3/api-docs`)
   - **Raw text** — вставьте содержимое спецификации
3. Postman создаст **коллекцию** со всеми эндпоинтами
4. Каждый эндпоинт станет готовым к использованию запросом

### После импорта:

- Все эндпоинты организованы по папкам
- Параметры и тела запросов предзаполнены примерами
- Вам остаётся только настроить **окружение** (базовый URL, токены аутентификации) и начать тестировать!

---

## 📝 Упражнения

### Упражнение 1: Чтение спецификации OpenAPI

Посмотрите на пример спецификации OpenAPI в разделе 2 и ответьте:

1. Какой базовый URL у production-сервера?
2. Сколько эндпоинтов у этого API?
3. Какие параметры принимает `GET /users`?
4. Какие поля обязательны для создания пользователя?
5. Какой код состояния возвращает `POST /users`, когда email уже существует?
6. Какие возможные значения у поля `role`?

<details>
<summary>Нажмите, чтобы увидеть ответы</summary>

1. `https://api.example.com/v1`
2. 3 эндпоинта: `GET /users`, `POST /users`, `GET /users/{id}`
3. `role` (query, необязательный, enum: admin/user/qa) и `page` (query, необязательный, default: 1)
4. `name` и `email` обязательны
5. `409 Conflict`
6. `admin`, `user`, `qa`

</details>

---

### Упражнение 2: Использование публичного Swagger UI

1. Перейдите на **[Petstore Swagger UI](https://petstore.swagger.io/)** — это демо-API
2. Изучите доступные эндпоинты
3. Используйте "Try it out" чтобы:
   - **GET** `/pet/findByStatus` — найти питомцев со статусом "available"
   - **POST** `/pet` — создать нового питомца
   - **GET** `/pet/{petId}` — найти только что созданного питомца
4. Обратите внимание на:
   - Какие коды состояния вы получаете?
   - Как выглядит тело ответа?
   - Что происходит при запросе GET для несуществующего ID?

---

### Упражнение 3: Первые запросы в Postman

Установите Postman и выполните эти задания:

**Используя бесплатный API `https://jsonplaceholder.typicode.com`:**

1. **GET всех пользователей**: `GET https://jsonplaceholder.typicode.com/users`
   - Сколько пользователей?
   - Какие поля есть у каждого пользователя?

2. **GET конкретного пользователя**: `GET https://jsonplaceholder.typicode.com/users/1`
   - Как зовут пользователя?

3. **GET постов пользователя**: `GET https://jsonplaceholder.typicode.com/posts?userId=1`
   - Сколько постов у пользователя 1?

4. **Создать пост**: `POST https://jsonplaceholder.typicode.com/posts`
   - Тело:
   ```json
   {
     "title": "Тестовый пост",
     "body": "Это мой тестовый пост из Postman",
     "userId": 1
   }
   ```
   - Какой код состояния вы получили?
   - Какой `id` был назначен новому посту?

5. **Обновить пост**: `PUT https://jsonplaceholder.typicode.com/posts/1`
   - Тело:
   ```json
   {
     "title": "Обновлённый заголовок",
     "body": "Обновлённое тело",
     "userId": 1
   }
   ```
   - Какой код состояния вы получили?

6. **Удалить пост**: `DELETE https://jsonplaceholder.typicode.com/posts/1`
   - Какой код состояния вы получили?
   - Что в теле ответа?

---

### Упражнение 4: Написание тестов в Postman

Для запросов из Упражнения 3 добавьте тесты на вкладке **Tests**:

1. Для **GET всех пользователей**:
```javascript
pm.test("Статус 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Возвращает 10 пользователей", function () {
    pm.expect(pm.response.json().length).to.eql(10);
});

pm.test("У каждого пользователя есть обязательные поля", function () {
    const users = pm.response.json();
    users.forEach(user => {
        pm.expect(user).to.have.property('id');
        pm.expect(user).to.have.property('name');
        pm.expect(user).to.have.property('email');
    });
});
```

2. Напишите аналогичные тесты для остальных запросов. Проверяйте:
   - Код состояния корректен
   - Тело ответа имеет ожидаемую структуру
   - Конкретные значения полей соответствуют ожиданиям

---

### Упражнение 5: Импорт Swagger в Postman

1. Перейдите в Postman → **Import**
2. Используйте этот URL: `https://petstore.swagger.io/v2/swagger.json`
3. Импортируйте коллекцию
4. Изучите сгенерированные запросы
5. Настройте базовый URL и выполните несколько запросов
6. Сравните: что удобнее для тестирования — Swagger UI или Postman? Почему?

---

## 📚 Дополнительные ресурсы

- [Спецификация OpenAPI](https://spec.openapis.org/oas/latest.html) — Официальная документация спецификации
- [Swagger Editor](https://editor.swagger.io/) — Написание и предпросмотр спецификаций OpenAPI онлайн
- [Swagger Petstore](https://petstore.swagger.io/) — Практика работы с API через Swagger UI
- [Postman Learning Center](https://learning.postman.com/) — Официальные учебные материалы Postman
- [JSONPlaceholder](https://jsonplaceholder.typicode.com/) — Бесплатный фейковый API для тестирования

---

## ✅ Самопроверка

После прохождения этого курса вы должны уметь:

- [ ] Объяснить, что такое REST API и как работают ресурсы/эндпоинты
- [ ] Читать спецификацию OpenAPI и понимать её структуру
- [ ] Использовать Swagger UI для изучения и тестирования эндпоинтов API
- [ ] Выполнять запросы GET, POST, PUT, DELETE в Postman
- [ ] Настраивать аутентификацию в Postman
- [ ] Писать базовые тестовые скрипты в Postman
- [ ] Импортировать спецификацию Swagger/OpenAPI в Postman
- [ ] Понимать связь между спецификацией OpenAPI, Swagger UI и Postman
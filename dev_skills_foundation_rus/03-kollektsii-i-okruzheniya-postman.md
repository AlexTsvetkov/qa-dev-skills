# Курс 03: Коллекции и окружения Postman

## 📖 Что вы изучите

- Как организовать запросы в коллекции
- Как использовать переменные (коллекции, окружения, глобальные)
- Как создавать окружения и переключаться между ними
- Как связывать запросы в цепочки
- Как запускать коллекции через Collection Runner
- Как использовать pre-request скрипты и тестовые скрипты вместе

---

## 1. Что такое коллекции?

**Коллекция** — это группа связанных API-запросов, организованных в папки. Представьте это как папку проекта для ваших API-тестов.

### Зачем нужны коллекции?

- 📁 **Организация** — группировка связанных запросов
- 🔄 **Переиспользование** — делитесь коллекциями с командой
- 🧪 **Автоматизация** — запуск всех запросов автоматически через Collection Runner
- 📝 **Документация** — коллекции могут служить живой документацией API
- 🏗️ **Рабочие процессы** — связывание запросов в цепочки (например: логин → создание → проверка)

### Пример структуры коллекции:

```
📁 User Management API
├── 📁 Authentication
│   ├── POST Login
│   ├── POST Register
│   └── POST Logout
├── 📁 Users
│   ├── GET All Users
│   ├── GET User by ID
│   ├── POST Create User
│   ├── PUT Update User
│   └── DELETE Delete User
└── 📁 User Orders
    ├── GET User Orders
    └── POST Create Order
```

### Создание коллекции:

1. Нажмите **"New"** → **"Collection"**
2. Дайте ей имя (например, "User Management API")
3. По желанию добавьте описание
4. Нажмите **"Create"**

### Добавление запросов в коллекцию:

1. Создайте новый запрос (нажмите **"+"**)
2. Настройте запрос (метод, URL, заголовки, тело)
3. Нажмите **"Save"** (Ctrl+S / Cmd+S)
4. Выберите коллекцию и папку
5. Дайте запросу описательное имя

### Создание папок в коллекции:

1. Правый клик на коллекции → **"Add Folder"**
2. Назовите папку (например, "Authentication", "Users")
3. Перетащите запросы в папки для организации

---

## 2. Переменные в Postman

Переменные — это **заполнители**, которые хранят значения для повторного использования в запросах. Вместо жёсткого указания URL, токенов и ID вы используете переменные.

### Зачем нужны переменные?

Без переменных (❌ Плохо):
```
GET https://api.staging.example.com/v1/users/42
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjo0Mn0.abc123
```

С переменными (✅ Хорошо):
```
GET {{baseUrl}}/users/{{userId}}
Authorization: Bearer {{authToken}}
```

### Синтаксис переменных:

Переменные используются через **двойные фигурные скобки**: `{{variableName}}`

Их можно использовать в:
- **URL**: `{{baseUrl}}/api/users`
- **Заголовках**: `Authorization: Bearer {{token}}`
- **Теле запроса**: `{"userId": {{userId}}}`
- **Query-параметрах**: `role={{userRole}}`
- **Тестах/Скриптах**: `pm.variables.get("token")`

### Области видимости переменных (порядок приоритета):

В Postman существуют разные области видимости переменных — от наиболее специфичной к наиболее общей:

```
┌─────────────────────────────────────────┐
│  1. Local (наивысший приоритет)          │ ← Устанавливается в скриптах, живёт только во время выполнения
│  ┌─────────────────────────────────────┐ │
│  │  2. Data                             │ │ ← Из внешних файлов данных (CSV/JSON)
│  │  ┌─────────────────────────────────┐ │ │
│  │  │  3. Environment                  │ │ │ ← Специфичные для выбранного окружения
│  │  │  ┌─────────────────────────────┐ │ │ │
│  │  │  │  4. Collection               │ │ │ │ ← Общие для всех запросов в коллекции
│  │  │  │  ┌─────────────────────────┐ │ │ │ │
│  │  │  │  │  5. Global (наименьший)  │ │ │ │ │ ← Доступны везде
│  │  │  │  └─────────────────────────┘ │ │ │ │
│  │  │  └─────────────────────────────┘ │ │ │
│  │  └─────────────────────────────────┘ │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

Если одно и то же имя переменной существует в нескольких областях, побеждает **наиболее специфичная** область.

### Установка переменных:

#### Через интерфейс Postman:
- **Переменные коллекции**: Нажмите на коллекцию → вкладка **Variables**
- **Переменные окружения**: Нажмите **иконку глаза** (👁️) рядом с выпадающим списком окружений
- **Глобальные переменные**: Нажмите **иконку глаза** → **Globals**

#### В скриптах (Pre-request или Tests):

```javascript
// Установка переменных в разных областях видимости
pm.globals.set("apiVersion", "v1");                    // Глобальная
pm.collectionVariables.set("baseUrl", "https://api.example.com"); // Коллекция
pm.environment.set("authToken", "eyJhbG...");          // Окружение
pm.variables.set("tempValue", "123");                   // Локальная

// Получение переменных
const token = pm.environment.get("authToken");
const baseUrl = pm.collectionVariables.get("baseUrl");

// Удаление переменных
pm.environment.unset("authToken");

// Получение переменной из любой области (использует порядок приоритета)
const value = pm.variables.get("someVariable");
```

---

## 3. Окружения

**Окружение** — это набор переменных, определяющих контекст для тестирования API. Обычно у вас разные окружения для разных серверов.

### Типичная настройка окружений:

#### 🟢 Окружение разработки (Development):
| Переменная | Значение |
|------------|----------|
| `baseUrl` | `http://localhost:8080/api` |
| `authToken` | `dev-token-123` |
| `dbName` | `app_dev` |

#### 🟡 Окружение тестирования (Staging):
| Переменная | Значение |
|------------|----------|
| `baseUrl` | `https://staging.example.com/api` |
| `authToken` | `staging-token-456` |
| `dbName` | `app_staging` |

#### 🔴 Продуктивное окружение (Production):
| Переменная | Значение |
|------------|----------|
| `baseUrl` | `https://api.example.com/api` |
| `authToken` | `prod-token-789` |
| `dbName` | `app_prod` |

### Создание окружения:

1. Нажмите **выпадающий список окружений** (верхний правый угол) → **"Manage Environments"** (⚙️)
2. Нажмите **"Add"**
3. Дайте ему имя (например, "Development")
4. Добавьте переменные с их значениями
5. Нажмите **"Save"**

### Переключение окружений:

- Используйте **выпадающий список окружений** в верхнем правом углу
- Все ссылки `{{variableName}}` будут использовать значения выбранного окружения
- Ваши запросы остаются теми же — меняются только переменные!

### Initial Value и Current Value:

| Поле | Назначение |
|------|------------|
| **Initial Value** | Передаётся при экспорте/шеринге окружения. Безопасно для совместного использования. |
| **Current Value** | Ваше локальное значение. Только на вашей машине. Используйте для секретов/токенов. |

⚠️ **Совет по безопасности**: Никогда не помещайте реальные пароли или токены в **Initial Value**. Используйте **Current Value** для конфиденциальных данных — они не будут переданы при экспорте окружения.

---

## 4. Связывание запросов в цепочки

**Связывание запросов** означает использование результата одного запроса в качестве входных данных для другого. Это необходимо для тестирования рабочих процессов, таких как:

1. **Логин** → получение токена авторизации
2. **Создание пользователя** → получение ID нового пользователя
3. **Получение пользователя** → проверка, что пользователь создан
4. **Удаление пользователя** → очистка

### Как связывать запросы:

#### Шаг 1: Сохранение значения из ответа

Во вкладке **Tests** первого запроса (например, Login):

```javascript
// Разбор ответа
const response = pm.response.json();

// Сохранение токена в переменные окружения
pm.environment.set("authToken", response.token);

// Сохранение других полезных значений
pm.environment.set("userId", response.user.id);

console.log("Токен сохранён:", response.token);
```

#### Шаг 2: Использование сохранённого значения в следующем запросе

В следующем запросе (например, Get User) используйте переменную:
- **URL**: `{{baseUrl}}/users/{{userId}}`
- **Заголовок**: `Authorization: Bearer {{authToken}}`

### Полный пример связывания:

**Запрос 1: Логин**
```
POST {{baseUrl}}/auth/login
Body: { "email": "nastya@example.com", "password": "secret123" }

Tests:
  const response = pm.response.json();
  pm.environment.set("authToken", response.token);
```

**Запрос 2: Создание пользователя**
```
POST {{baseUrl}}/users
Headers: Authorization: Bearer {{authToken}}
Body: { "name": "New User", "email": "new@example.com" }

Tests:
  const response = pm.response.json();
  pm.environment.set("createdUserId", response.id);
```

**Запрос 3: Получение созданного пользователя**
```
GET {{baseUrl}}/users/{{createdUserId}}
Headers: Authorization: Bearer {{authToken}}

Tests:
  pm.test("Имя пользователя совпадает", function () {
      const user = pm.response.json();
      pm.expect(user.name).to.eql("New User");
  });
```

**Запрос 4: Удаление пользователя (очистка)**
```
DELETE {{baseUrl}}/users/{{createdUserId}}
Headers: Authorization: Bearer {{authToken}}

Tests:
  pm.test("Пользователь удалён", function () {
      pm.response.to.have.status(204);
  });
  // Очистка переменных
  pm.environment.unset("createdUserId");
```

---

## 5. Pre-request скрипты

**Pre-request скрипты** выполняют JavaScript **до** отправки запроса. Полезны для:

- Генерации динамических данных
- Установки временных меток
- Вычисления подписей аутентификации
- Подготовки тестовых данных

### Примеры:

```javascript
// Генерация случайного email для тестирования
const randomEmail = "user_" + Math.random().toString(36).substring(7) + "@test.com";
pm.environment.set("randomEmail", randomEmail);

// Установка текущей временной метки
pm.environment.set("timestamp", new Date().toISOString());

// Генерация случайного числа
pm.environment.set("randomId", Math.floor(Math.random() * 1000));

// Вычисление значения на основе других переменных
const baseUrl = pm.environment.get("baseUrl");
const version = pm.collectionVariables.get("apiVersion");
pm.variables.set("fullUrl", baseUrl + "/" + version);
```

### Порядок выполнения:

```
1. Pre-request скрипт коллекции       (выполняется первым)
2. Pre-request скрипт папки           (если запрос находится в папке)
3. Pre-request скрипт запроса         (выполняется непосредственно перед отправкой)
4. ──── ЗАПРОС ОТПРАВЛЕН ────
5. Тестовый скрипт коллекции          (выполняется первым после получения ответа)
6. Тестовый скрипт папки              (если запрос находится в папке)
7. Тестовый скрипт запроса            (выполняется последним)
```

---

## 6. Collection Runner

**Collection Runner** позволяет запускать все запросы в коллекции автоматически, последовательно.

### Как использовать Collection Runner:

1. Нажмите на коллекцию
2. Нажмите кнопку **"Run"** (▶️)
3. Настройте запуск:
   - Выберите, какие запросы включить
   - Установите количество **итераций**
   - Установите **задержку** между запросами (в миллисекундах)
   - По желанию предоставьте **файл данных** (CSV/JSON)
4. Нажмите **"Run Collection"**

### Что вы увидите в результатах:

- ✅ Пройденные тесты (зелёные)
- ❌ Проваленные тесты (красные)
- Время ответа для каждого запроса
- Итоговая статистика

### Тестирование на основе данных (Data-Driven Testing):

Вы можете запускать одну и ту же коллекцию с разными наборами данных, используя **файл CSV или JSON**.

**Пример файла CSV (`test-users.csv`):**
```csv
userName,userEmail,userRole
Alice,alice@test.com,admin
Bob,bob@test.com,user
Charlie,charlie@test.com,qa
```

**Пример файла JSON (`test-users.json`):**
```json
[
  {"userName": "Alice", "userEmail": "alice@test.com", "userRole": "admin"},
  {"userName": "Bob", "userEmail": "bob@test.com", "userRole": "user"},
  {"userName": "Charlie", "userEmail": "charlie@test.com", "userRole": "qa"}
]
```

**В теле вашего запроса:**
```json
{
  "name": "{{userName}}",
  "email": "{{userEmail}}",
  "role": "{{userRole}}"
}
```

Runner выполнит коллекцию **3 раза** (по одному для каждой строки), используя разные данные каждый раз.

---

## 7. Полезные возможности Postman

### Консоль (отладка):

Откройте **Postman Console** (View → Show Postman Console, или Ctrl+Alt+C / Cmd+Alt+C) для просмотра:
- Полных деталей запроса (URL, заголовки, тело как оно было фактически отправлено)
- Полных деталей ответа
- Вывода console.log из ваших скриптов
- Сетевой информации

```javascript
// Вывод в консоль Postman
console.log("URL запроса:", pm.request.url.toString());
console.log("Тело ответа:", pm.response.json());
console.log("Токен авторизации:", pm.environment.get("authToken"));
```

### Динамические переменные:

Postman предоставляет встроенные динамические переменные для генерации тестовых данных:

| Переменная | Описание | Пример вывода |
|------------|----------|---------------|
| `{{$randomFirstName}}` | Случайное имя | "Jessica" |
| `{{$randomLastName}}` | Случайная фамилия | "Smith" |
| `{{$randomEmail}}` | Случайный email | "jessica.smith@example.com" |
| `{{$randomUUID}}` | Случайный UUID | "6929bb52-3ab2-..." |
| `{{$timestamp}}` | Текущая Unix-метка времени | "1614165600" |
| `{{$randomInt}}` | Случайное целое число (0-1000) | "742" |
| `{{$randomBoolean}}` | Случайное булево значение | "true" |
| `{{$randomPhoneNumber}}` | Случайный номер телефона | "+1-555-123-4567" |

### Совместное использование коллекций:

Вы можете делиться коллекциями с командой:
1. **Экспорт** — сохранение как JSON-файл, передача через Git или email
2. **Team Workspace** — прямой шеринг в Postman (требуется аккаунт)
3. **Postman Link** — генерация ссылки для совместного доступа

### Документация коллекций:

Каждая коллекция, папка и запрос могут иметь **документацию**:
1. Нажмите на коллекцию/запрос
2. Нажмите **иконку документации** (📝)
3. Напишите документацию в формате Markdown
4. Эта документация видна всем, у кого есть доступ к коллекции

---

## 📝 Упражнения

### Упражнение 1: Создание коллекции

Создайте коллекцию **"JSONPlaceholder Tests"** со следующей структурой:

```
📁 JSONPlaceholder Tests
├── 📁 Users
│   ├── GET All Users
│   ├── GET User by ID
│   └── GET User Posts
├── 📁 Posts
│   ├── GET All Posts
│   ├── GET Post by ID
│   ├── POST Create Post
│   ├── PUT Update Post
│   └── DELETE Delete Post
└── 📁 Comments
    └── GET Comments by Post
```

**Базовый URL**: `https://jsonplaceholder.typicode.com`

Настройте **переменную коллекции** `baseUrl` с этим значением и используйте `{{baseUrl}}` во всех запросах.

---

### Упражнение 2: Создание окружений

Создайте два окружения:

**Окружение 1: "JSONPlaceholder"**
| Переменная | Значение |
|------------|----------|
| `baseUrl` | `https://jsonplaceholder.typicode.com` |
| `testUserId` | `1` |

**Окружение 2: "ReqRes"**
| Переменная | Значение |
|------------|----------|
| `baseUrl` | `https://reqres.in/api` |
| `testUserId` | `2` |

Теперь обновите ваш запрос **"GET User by ID"**, чтобы он использовал:
- URL: `{{baseUrl}}/users/{{testUserId}}`

Переключайтесь между окружениями и убедитесь, что запрос работает с обоими API.

---

### Упражнение 3: Связывание запросов в цепочки

Создайте рабочий процесс в вашей коллекции:

1. **POST Create Post**
   - URL: `{{baseUrl}}/posts`
   - Тело:
   ```json
   {
     "title": "Chain Test - {{$timestamp}}",
     "body": "This post was created by a chained request",
     "userId": 1
   }
   ```
   - **Tests** — сохраните ID созданного поста:
   ```javascript
   pm.test("Пост успешно создан", function () {
       pm.response.to.have.status(201);
   });
   
   const response = pm.response.json();
   pm.collectionVariables.set("createdPostId", response.id);
   console.log("Создан пост с ID:", response.id);
   ```

2. **GET Verify Post**
   - URL: `{{baseUrl}}/posts/{{createdPostId}}`
   - **Tests**:
   ```javascript
   pm.test("Пост существует", function () {
       pm.response.to.have.status(200);
   });
   
   pm.test("У поста правильный userId", function () {
       const post = pm.response.json();
       pm.expect(post.userId).to.eql(1);
   });
   ```

3. **PUT Update Post**
   - URL: `{{baseUrl}}/posts/{{createdPostId}}`
   - Тело:
   ```json
   {
     "title": "Updated Chain Test",
     "body": "This post was updated",
     "userId": 1
   }
   ```
   - **Tests**:
   ```javascript
   pm.test("Пост успешно обновлён", function () {
       pm.response.to.have.status(200);
       const post = pm.response.json();
       pm.expect(post.title).to.eql("Updated Chain Test");
   });
   ```

4. **DELETE Cleanup Post**
   - URL: `{{baseUrl}}/posts/{{createdPostId}}`
   - **Tests**:
   ```javascript
   pm.test("Пост удалён", function () {
       pm.response.to.have.status(200);
   });
   pm.collectionVariables.unset("createdPostId");
   ```

Запустите всю папку через **Collection Runner** и убедитесь, что все тесты проходят.

---

### Упражнение 4: Тестирование на основе данных

1. Создайте CSV-файл `test-posts.csv`:
```csv
title,body,userId
First Post,Content of first post,1
Second Post,Content of second post,2
Third Post,Content of third post,1
```

2. Создайте запрос **"POST Create Multiple Posts"**:
   - URL: `{{baseUrl}}/posts`
   - Тело:
   ```json
   {
     "title": "{{title}}",
     "body": "{{body}}",
     "userId": {{userId}}
   }
   ```
   - Tests:
   ```javascript
   pm.test("Пост создан со статусом 201", function () {
       pm.response.to.have.status(201);
   });
   
   pm.test("Заголовок совпадает с входными данными", function () {
       const response = pm.response.json();
       const expectedTitle = pm.iterationData.get("title");
       pm.expect(response.title).to.eql(expectedTitle);
   });
   ```

3. Запустите через Collection Runner:
   - Выберите только этот запрос
   - Загрузите CSV-файл как данные
   - Установите итерации на **3**
   - Запустите и убедитесь, что все 3 итерации проходят

---

### Упражнение 5: Pre-request скрипты

Создайте запрос, который динамически генерирует тестовые данные:

1. **POST Create Dynamic User**
   - **Pre-request Script**:
   ```javascript
   // Генерация уникальных тестовых данных
   const timestamp = Date.now();
   const randomNum = Math.floor(Math.random() * 10000);
   
   pm.environment.set("testName", "TestUser_" + randomNum);
   pm.environment.set("testEmail", "test_" + timestamp + "@example.com");
   pm.environment.set("testJob", "QA Engineer");
   
   console.log("Сгенерированные тестовые данные:");
   console.log("  Имя:", pm.environment.get("testName"));
   console.log("  Email:", pm.environment.get("testEmail"));
   ```

   - **URL**: `POST https://reqres.in/api/users`
   - **Тело**:
   ```json
   {
     "name": "{{testName}}",
     "email": "{{testEmail}}",
     "job": "{{testJob}}"
   }
   ```

   - **Tests**:
   ```javascript
   pm.test("Статус 201", function () {
       pm.response.to.have.status(201);
   });
   
   pm.test("Имя совпадает с сгенерированным значением", function () {
       const response = pm.response.json();
       const expectedName = pm.environment.get("testName");
       pm.expect(response.name).to.eql(expectedName);
   });
   
   pm.test("В ответе есть ID", function () {
       pm.expect(pm.response.json()).to.have.property('id');
   });
   ```

2. Запустите этот запрос **5 раз** через Collection Runner (5 итераций) и убедитесь, что каждый запуск создаёт пользователя с уникальным именем.

---

## 📚 Дополнительные ресурсы

- [Postman Variables](https://learning.postman.com/docs/sending-requests/variables/) — Официальная документация по переменным
- [Postman Collection Runner](https://learning.postman.com/docs/running-collections/intro-to-collection-runs/) — Запуск коллекций
- [Postman Scripting](https://learning.postman.com/docs/writing-scripts/intro-to-scripts/) — Написание тестовых скриптов
- [Postman Dynamic Variables](https://learning.postman.com/docs/writing-scripts/script-references/variables-list/) — Встроенные генераторы случайных данных
- [ReqRes API](https://reqres.in/) — Ещё одно бесплатное API для практики

---

## ✅ Самопроверка

После прохождения этого курса вы должны уметь:

- [ ] Создавать и организовывать коллекции с папками
- [ ] Использовать переменные в URL, заголовках и теле запросов
- [ ] Объяснить разницу между глобальными переменными, переменными окружения и коллекции
- [ ] Создавать окружения и переключаться между ними (dev, staging, prod)
- [ ] Связывать запросы, сохраняя значения ответов в переменные
- [ ] Писать pre-request скрипты для генерации динамических тестовых данных
- [ ] Запускать коллекции автоматически через Collection Runner
- [ ] Использовать файлы данных (CSV/JSON) для тестирования на основе данных
- [ ] Использовать консоль Postman для отладки
- [ ] Делиться коллекциями с членами команды
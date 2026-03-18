# Курс 04: Основы Java Backend

## 📖 Что вы изучите

- Как устроено веб-приложение (фронтенд vs бэкенд)
- Что такое Java-бэкенд приложение и какие фреймворки используются
- Как HTTP-запрос проходит через Java-бэкенд (пошагово)
- Ключевые концепции бэкенда: контроллеры, сервисы, репозитории, базы данных
- Как читать логи бэкенда и понимать, что произошло
- Что QA должен знать о бэкенде, чтобы быть эффективнее

---

## 1. Фронтенд vs Бэкенд

Каждое веб-приложение состоит из двух основных частей:

```
┌─────────────┐         HTTP          ┌─────────────────┐         SQL          ┌──────────┐
│  ФРОНТЕНД   │  ←──── Запрос ─────→  │    БЭКЕНД        │  ←──── Запрос ────→  │  БАЗА    │
│             │        Ответ          │                  │        Результат    │  ДАННЫХ  │
│ (Браузер)   │                       │ (Java/Spring)    │                     │(PostgreSQL│
│ HTML/CSS/JS │                       │                  │                     │ MySQL...) │
└─────────────┘                       └─────────────────┘                     └──────────┘
```

| Часть | Что делает | Технологии |
|-------|-----------|------------|
| **Фронтенд** | То, что пользователь видит и с чем взаимодействует | HTML, CSS, JavaScript, React, Angular |
| **Бэкенд** | Обрабатывает запросы, бизнес-логика, хранение данных | Java, Spring Boot, Node.js, Python |
| **База данных** | Хранит данные постоянно | PostgreSQL, MySQL, MongoDB |

### Зачем QA нужно понимать бэкенд:

- 🐛 **Быстрее находить баги** — понимать, где может быть баг
- 📝 **Писать лучшие баг-репорты** — включать детали ошибок бэкенда
- 🔍 **Тестировать эффективнее** — знать, что бэкенд валидирует, а что нет
- 💬 **Общаться с разработчиками** — говорить на одном языке
- 🧪 **API-тестирование** — тестировать бэкенд напрямую, без фронтенда

---

## 2. Стек технологий Java Backend

Большинство корпоративных Java-приложений используют эти технологии:

### Основные технологии:

| Технология | Назначение |
|-----------|-----------|
| **Java** | Язык программирования |
| **Spring Boot** | Фреймворк для быстрого создания веб-приложений |
| **Spring MVC** | Обработка HTTP-запросов и ответов |
| **Spring Data JPA** | Упрощение работы с базой данных |
| **Hibernate** | Преобразование Java-объектов в таблицы базы данных (ORM) |
| **Maven / Gradle** | Инструменты сборки — управление зависимостями и сборка проекта |

### Структура проекта (типичное Spring Boot приложение):

```
my-application/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/myapp/
│   │   │       ├── MyApplication.java          ← Точка входа
│   │   │       ├── controller/                  ← Обработчики HTTP-запросов
│   │   │       │   └── UserController.java
│   │   │       ├── service/                     ← Бизнес-логика
│   │   │       │   └── UserService.java
│   │   │       ├── repository/                  ← Доступ к базе данных
│   │   │       │   └── UserRepository.java
│   │   │       ├── model/                       ← Объекты данных
│   │   │       │   └── User.java
│   │   │       ├── dto/                         ← Data Transfer Objects
│   │   │       │   ├── CreateUserRequest.java
│   │   │       │   └── UserResponse.java
│   │   │       └── exception/                   ← Обработка ошибок
│   │   │           └── UserNotFoundException.java
│   │   └── resources/
│   │       ├── application.properties           ← Конфигурация
│   │       └── application.yml                  ← Конфигурация (YAML-формат)
│   └── test/
│       └── java/                                ← Тесты
├── pom.xml                                      ← Зависимости Maven
└── build.gradle                                 ← Зависимости Gradle (альтернатива)
```

---

## 3. Как запрос проходит через бэкенд

Давайте проследим, что происходит при отправке `GET /api/users/42`:

```
                    HTTP-запрос: GET /api/users/42
                              │
                              ▼
┌──────────────────────────────────────────────────┐
│  1. DISPATCHER SERVLET                            │
│     Фронт-контроллер Spring — направляет          │
│     запрос к нужному контроллеру                  │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  2. КОНТРОЛЛЕР (UserController)                   │
│     @GetMapping("/api/users/{id}")                │
│     - Получает HTTP-запрос                        │
│     - Извлекает параметры (id = 42)               │
│     - Вызывает сервисный слой                     │
│     - Возвращает HTTP-ответ                       │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  3. СЕРВИС (UserService)                          │
│     - Содержит бизнес-логику                      │
│     - Валидирует данные                           │
│     - Обращается к репозиторию для доступа к БД   │
│     - Преобразует данные при необходимости         │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  4. РЕПОЗИТОРИЙ (UserRepository)                  │
│     - Общается с базой данных                     │
│     - Преобразует вызовы Java в SQL-запросы        │
│     - Возвращает данные как Java-объекты           │
└──────────────────┬───────────────────────────────┘
                   │
                   ▼
┌──────────────────────────────────────────────────┐
│  5. БАЗА ДАННЫХ                                   │
│     SELECT * FROM users WHERE id = 42             │
│     → Возвращает строку из таблицы users          │
└──────────────────────────────────────────────────┘
```

Ответ проходит обратно тем же путём:
```
База данных → Репозиторий → Сервис → Контроллер → HTTP-ответ → Клиент
```

---

## 4. Подробный разбор каждого слоя

### Слой 1: Контроллер

**Контроллер** — это точка входа для HTTP-запросов. Он связывает URL-адреса с Java-методами.

```java
@RestController                          // Этот класс обрабатывает HTTP-запросы
@RequestMapping("/api/users")            // Базовый URL для всех методов в этом классе
public class UserController {

    private final UserService userService;

    // Внедрение через конструктор — Spring предоставляет UserService
    public UserController(UserService userService) {
        this.userService = userService;
    }

    // GET /api/users — получить всех пользователей
    @GetMapping
    public List<UserResponse> getAllUsers() {
        return userService.getAllUsers();
    }

    // GET /api/users/42 — получить пользователя по ID
    @GetMapping("/{id}")
    public UserResponse getUserById(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    // POST /api/users — создать нового пользователя
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)   // Возвращает 201 вместо 200
    public UserResponse createUser(@RequestBody CreateUserRequest request) {
        return userService.createUser(request);
    }

    // PUT /api/users/42 — обновить пользователя
    @PutMapping("/{id}")
    public UserResponse updateUser(@PathVariable Long id,
                                    @RequestBody CreateUserRequest request) {
        return userService.updateUser(id, request);
    }

    // DELETE /api/users/42 — удалить пользователя
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)  // Возвращает 204
    public void deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
    }

    // GET /api/users?role=qa&page=1 — с query-параметрами
    @GetMapping
    public List<UserResponse> getUsersByRole(
            @RequestParam(required = false) String role,
            @RequestParam(defaultValue = "1") int page) {
        return userService.getUsersByRole(role, page);
    }
}
```

**Ключевые аннотации, которые нужно знать:**

| Аннотация | Назначение |
|-----------|-----------|
| `@RestController` | Помечает класс как REST-контроллер |
| `@RequestMapping` | Задаёт базовый путь URL |
| `@GetMapping` | Обрабатывает GET-запросы |
| `@PostMapping` | Обрабатывает POST-запросы |
| `@PutMapping` | Обрабатывает PUT-запросы |
| `@DeleteMapping` | Обрабатывает DELETE-запросы |
| `@PathVariable` | Извлекает значение из пути URL (`/users/{id}`) |
| `@RequestParam` | Извлекает query-параметр (`?role=qa`) |
| `@RequestBody` | Считывает JSON-тело в Java-объект |
| `@ResponseStatus` | Задаёт HTTP-код состояния для ответа |

### Слой 2: Сервис

**Сервис** содержит бизнес-логику — «мозг» приложения.

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public List<UserResponse> getAllUsers() {
        List<User> users = userRepository.findAll();
        return users.stream()
                .map(this::toResponse)    // Конвертация каждого User в UserResponse
                .collect(Collectors.toList());
    }

    public UserResponse getUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException("Пользователь не найден: " + id));
        return toResponse(user);
    }

    public UserResponse createUser(CreateUserRequest request) {
        // Бизнес-логика: проверка, существует ли уже такой email
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new EmailAlreadyExistsException("Email уже используется");
        }

        // Создание нового пользователя
        User user = new User();
        user.setName(request.getName());
        user.setEmail(request.getEmail());
        user.setRole(request.getRole() != null ? request.getRole() : "user");
        user.setCreatedAt(LocalDateTime.now());

        // Сохранение в базу данных
        User savedUser = userRepository.save(user);
        return toResponse(savedUser);
    }

    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new UserNotFoundException("Пользователь не найден: " + id);
        }
        userRepository.deleteById(id);
    }

    // Вспомогательный метод: конвертация сущности User в UserResponse DTO
    private UserResponse toResponse(User user) {
        return new UserResponse(
            user.getId(),
            user.getName(),
            user.getEmail(),
            user.getRole(),
            user.getCreatedAt()
        );
    }
}
```

**Что делает сервис:**
- ✅ Валидирует бизнес-правила (например, «email должен быть уникальным»)
- ✅ Преобразует данные между слоями
- ✅ Обрабатывает сценарии «что, если» (пользователь не найден, дублирование email)
- ✅ Координирует работу между разными репозиториями при необходимости

### Слой 3: Репозиторий

**Репозиторий** общается с базой данных. Spring Data JPA автоматически генерирует большую часть кода.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Spring автоматически генерирует SQL из имени метода!
    // SELECT * FROM users WHERE email = ?
    Optional<User> findByEmail(String email);

    // SELECT COUNT(*) FROM users WHERE email = ? > 0
    boolean existsByEmail(String email);

    // SELECT * FROM users WHERE role = ?
    List<User> findByRole(String role);

    // SELECT * FROM users WHERE name LIKE '%keyword%'
    List<User> findByNameContaining(String keyword);

    // Свой запрос, когда имена методов становятся слишком сложными
    @Query("SELECT u FROM User u WHERE u.role = :role AND u.createdAt > :date")
    List<User> findActiveUsersByRole(@Param("role") String role,
                                      @Param("date") LocalDateTime date);
}
```

**Встроенные методы JpaRepository:**

| Метод | SQL-эквивалент |
|-------|----------------|
| `findAll()` | `SELECT * FROM users` |
| `findById(42)` | `SELECT * FROM users WHERE id = 42` |
| `save(user)` | `INSERT INTO users ...` или `UPDATE users ...` |
| `deleteById(42)` | `DELETE FROM users WHERE id = 42` |
| `existsById(42)` | `SELECT COUNT(*) FROM users WHERE id = 42 > 0` |
| `count()` | `SELECT COUNT(*) FROM users` |

### Слой 4: Модель (Сущность)

**Модель** (или сущность) представляет таблицу базы данных в виде Java-класса.

```java
@Entity                          // Этот класс сопоставляется с таблицей БД
@Table(name = "users")           // Имя таблицы в базе данных
public class User {

    @Id                          // Первичный ключ
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Автоинкремент
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String role;

    @Column(name = "created_at")
    private LocalDateTime createdAt;

    // Геттеры и сеттеры
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    // ... и т.д.
}
```

**Это соответствует таблице в базе данных:**

```sql
CREATE TABLE users (
    id          BIGINT PRIMARY KEY AUTO_INCREMENT,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(255) NOT NULL UNIQUE,
    role        VARCHAR(255) NOT NULL,
    created_at  TIMESTAMP
);
```

### DTO (Data Transfer Objects)

DTO — это простые объекты для передачи данных между слоями. Они контролируют, какие данные передаются клиенту.

```java
// Что отправляет клиент (тело запроса)
public class CreateUserRequest {
    private String name;      // Обязательное
    private String email;     // Обязательное
    private String role;      // Необязательное

    // Геттеры и сеттеры
}

// Что получает клиент (тело ответа)
public class UserResponse {
    private Long id;
    private String name;
    private String email;
    private String role;
    private LocalDateTime createdAt;

    // Конструктор, геттеры
}
```

**Зачем использовать DTO вместо возврата сущности напрямую?**
- 🔒 **Безопасность** — не раскрывать внутренние поля (например, хеш пароля)
- 📋 **Контроль** — выбирать, какие именно данные возвращать
- 🔄 **Гибкость** — изменять базу данных, не ломая API

---

## 5. Обработка ошибок

Когда что-то идёт не так, бэкенд должен возвращать правильные ответы с ошибками.

### Обработчик исключений:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Обработка «Пользователь не найден» → 404
    @ExceptionHandler(UserNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleUserNotFound(UserNotFoundException ex) {
        return new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            "NOT_FOUND",
            ex.getMessage()
        );
    }

    // Обработка «Email уже существует» → 409
    @ExceptionHandler(EmailAlreadyExistsException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDuplicateEmail(EmailAlreadyExistsException ex) {
        return new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            "CONFLICT",
            ex.getMessage()
        );
    }

    // Обработка ошибок валидации → 400
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidation(MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getFieldErrors().stream()
                .map(e -> e.getField() + ": " + e.getDefaultMessage())
                .collect(Collectors.joining(", "));
        return new ErrorResponse(400, "BAD_REQUEST", message);
    }

    // Обработка всего остального → 500
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGeneral(Exception ex) {
        return new ErrorResponse(500, "INTERNAL_ERROR", "Произошла непредвиденная ошибка");
    }
}
```

### Формат ответа с ошибкой:

```json
{
  "status": 404,
  "error": "NOT_FOUND",
  "message": "User not found: 42"
}
```

### 🧠 Взгляд QA: Чеклист обработки ошибок

- [ ] Возвращает ли API правильные коды состояния для разных ошибок?
- [ ] Понятны ли сообщения об ошибках, но не раскрывают ли они внутренние детали?
- [ ] Что происходит при отправке невалидного JSON?
- [ ] Что происходит, когда обязательные поля отсутствуют?
- [ ] Не раскрывает ли ошибка 500 стек-трейсы или внутреннюю информацию?

---

## 6. Валидация запросов

Бэкенд валидирует входящие данные перед их обработкой:

```java
public class CreateUserRequest {

    @NotBlank(message = "Имя обязательно")
    @Size(min = 2, max = 100, message = "Имя должно быть от 2 до 100 символов")
    private String name;

    @NotBlank(message = "Email обязателен")
    @Email(message = "Неверный формат email")
    private String email;

    @Pattern(regexp = "^(admin|user|qa)$", message = "Роль должна быть admin, user или qa")
    private String role;
}
```

**Распространённые аннотации валидации:**

| Аннотация | Что проверяет |
|-----------|---------------|
| `@NotBlank` | Не null, не пустое, не только пробелы |
| `@NotNull` | Не null (но может быть пустым) |
| `@Size(min, max)` | Длина строки или размер коллекции |
| `@Email` | Корректный формат email |
| `@Min(value)` | Минимальное числовое значение |
| `@Max(value)` | Максимальное числовое значение |
| `@Pattern(regexp)` | Соответствие регулярному выражению |
| `@Past` | Дата должна быть в прошлом |
| `@Future` | Дата должна быть в будущем |

### 🧠 Идеи для тестов QA на основе валидации:

Когда вы видите `@NotBlank` на поле `name`, протестируйте с:
- `null` → должно упасть
- `""` (пустая строка) → должно упасть
- `"  "` (только пробелы) → должно упасть
- `"A"` (слишком короткое, если `@Size(min=2)`) → должно упасть
- Валидное имя → должно пройти

---

## 7. Конфигурация

Бэкенд-приложения имеют конфигурационные файлы, которые управляют поведением:

### application.properties:
```properties
# Порт сервера
server.port=8080

# Подключение к базе данных
spring.datasource.url=jdbc:postgresql://localhost:5432/myapp
spring.datasource.username=postgres
spring.datasource.password=secret

# Показывать SQL-запросы в логах
spring.jpa.show-sql=true

# Уровень логирования
logging.level.com.example=DEBUG
logging.level.org.springframework.web=INFO

# Префикс API
server.servlet.context-path=/api/v1
```

### application.yml (то же самое, другой формат):
```yaml
server:
  port: 8080
  servlet:
    context-path: /api/v1

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/myapp
    username: postgres
    password: secret
  jpa:
    show-sql: true

logging:
  level:
    com.example: DEBUG
    org.springframework.web: INFO
```

### Профили (разные конфигурации для разных окружений):

```
application.properties          ← Конфигурация по умолчанию
application-dev.properties      ← Переопределения для разработки
application-staging.properties  ← Переопределения для тестирования
application-prod.properties     ← Переопределения для продакшена
```

Запуск с определённым профилем:
```bash
java -jar myapp.jar --spring.profiles.active=staging
```

---

## 8. Чтение логов бэкенда

Логи — ваш **лучший друг** при отладке. Вот как их читать:

### Уровни логирования (от наиболее к наименее подробному):

| Уровень | Назначение | Пример |
|---------|-----------|--------|
| **TRACE** | Очень подробная трассировка | Каждое значение параметра SQL |
| **DEBUG** | Отладочная информация | Детали запросов/ответов |
| **INFO** | Обычные операции | «Приложение запущено», «Пользователь создан» |
| **WARN** | Потенциальные проблемы | «Обнаружен медленный запрос», «Использован устаревший API» |
| **ERROR** | Что-то сломалось | «Ошибка подключения к БД», «NullPointerException» |

### Пример вывода логов:

```
2024-01-15 10:30:00.123 INFO  [main] c.e.MyApplication : Starting MyApplication
2024-01-15 10:30:02.456 INFO  [main] c.e.MyApplication : Application started on port 8080

2024-01-15 10:30:15.789 DEBUG [http-nio-8080-exec-1] c.e.controller.UserController : GET /api/users/42
2024-01-15 10:30:15.790 DEBUG [http-nio-8080-exec-1] c.e.service.UserService : Finding user by ID: 42
2024-01-15 10:30:15.795 DEBUG [http-nio-8080-exec-1] org.hibernate.SQL : select u from users u where u.id=?
2024-01-15 10:30:15.798 DEBUG [http-nio-8080-exec-1] c.e.service.UserService : User found: Nastya
2024-01-15 10:30:15.800 DEBUG [http-nio-8080-exec-1] c.e.controller.UserController : Returning user: 42

2024-01-15 10:30:20.100 WARN  [http-nio-8080-exec-2] c.e.service.UserService : User not found: 999
2024-01-15 10:30:20.101 ERROR [http-nio-8080-exec-2] c.e.exception.GlobalExceptionHandler : UserNotFoundException: User not found: 999
```

### Чтение формата лога:

```
2024-01-15 10:30:15.789 DEBUG [http-nio-8080-exec-1] c.e.controller.UserController : GET /api/users/42
│          │              │     │                      │                               │
│          │              │     │                      │                               └─ Сообщение
│          │              │     │                      └─ Класс, который записал лог
│          │              │     └─ Поток (какой запрос обрабатывается)
│          │              └─ Уровень лога
│          └─ Время
└─ Дата
```

### 🧠 Советы QA по работе с логами:

- **Попросите разработчиков** установить уровень логов DEBUG для вашего тестового окружения
- **Ищите строки ERROR**, когда запрос не удался
- **Обращайте внимание на SQL-запросы**, чтобы понять, какие данные запрашивает бэкенд
- **Проверяйте имена потоков**, чтобы проследить конкретный запрос через логи
- **Используйте временные метки** для сопоставления вашего тестового действия с записями в логах

---

## 📝 Упражнения

### Упражнение 1: Проследите запрос

Учитывая приведённый код, проследите, что происходит при отправке `POST /api/users` с таким телом:
```json
{
  "name": "Nastya",
  "email": "nastya@example.com",
  "role": "qa"
}
```

Запишите каждый шаг:
1. Какой метод контроллера получает запрос?
2. Что делает сервис?
3. Какой SQL-запрос выполняет репозиторий?
4. Как выглядит ответ?

<details>
<summary>Нажмите, чтобы увидеть ответ</summary>

1. `UserController.createUser()` получает запрос, `@RequestBody` преобразует JSON в `CreateUserRequest`
2. `UserService.createUser()`:
   - Проверяет, существует ли email (`userRepository.existsByEmail("nastya@example.com")`)
   - Создаёт новый объект `User` и устанавливает поля
   - Сохраняет в базу данных (`userRepository.save(user)`)
   - Преобразует в `UserResponse` и возвращает
3. `INSERT INTO users (name, email, role, created_at) VALUES ('Nastya', 'nastya@example.com', 'qa', '2024-01-15 10:30:00')`
4. Ответ:
```json
{
  "id": 43,
  "name": "Nastya",
  "email": "nastya@example.com",
  "role": "qa",
  "createdAt": "2024-01-15T10:30:00"
}
```
Статус: `201 Created`

</details>

---

### Упражнение 2: Что пойдёт не так?

Для каждого сценария определите:
- Какая ошибка возникнет?
- В каком слое (контроллер, сервис, репозиторий)?
- Какой код состояния должен получить клиент?

**Сценарии:**

1. `POST /api/users` с `{"name": "", "email": "nastya@example.com"}`
2. `POST /api/users` с `{"name": "Nastya", "email": "nastya@example.com"}`, но такой email уже существует
3. `GET /api/users/999`, но пользователя 999 не существует
4. `POST /api/users` с `{"name": "Nastya"}` (email отсутствует)
5. `DELETE /api/users/42`, но база данных временно недоступна

<details>
<summary>Нажмите, чтобы увидеть ответы</summary>

1. **Ошибка валидации** в слое **контроллера** → `400 Bad Request` (имя пустое, нарушает `@NotBlank`)
2. **Ошибка бизнес-логики** в слое **сервиса** → `409 Conflict` (сервис проверяет `existsByEmail`)
3. **Ошибка «не найдено»** в слое **сервиса** → `404 Not Found` (сервис выбрасывает `UserNotFoundException`)
4. **Ошибка валидации** в слое **контроллера** → `400 Bad Request` (email обязателен, нарушает `@NotBlank`)
5. **Ошибка инфраструктуры** в слое **репозитория** → `500 Internal Server Error` (ошибка подключения к базе данных)

</details>

---

### Упражнение 3: Проектирование тест-кейсов

На основе правил валидации `CreateUserRequest`:
```java
@NotBlank @Size(min = 2, max = 100) private String name;
@NotBlank @Email private String email;
@Pattern(regexp = "^(admin|user|qa)$") private String role;
```

Спроектируйте тест-кейсы для эндпоинта `POST /api/users`:

| # | Тест-кейс | Тело запроса | Ожидаемый статус |
|---|-----------|-------------|-----------------|
| 1 | Валидный пользователь со всеми полями | ? | ? |
| 2 | Валидный пользователь без роли | ? | ? |
| 3 | Пустое имя | ? | ? |
| 4 | ... | ... | ... |

Создайте как минимум **10 тест-кейсов**, покрывающих позитивные и негативные сценарии.

<details>
<summary>Нажмите, чтобы увидеть примеры тест-кейсов</summary>

| # | Тест-кейс | Тело запроса | Ожидаемый статус |
|---|-----------|-------------|-----------------|
| 1 | Валидный пользователь, все поля | `{"name":"Nastya","email":"n@test.com","role":"qa"}` | 201 |
| 2 | Валидный пользователь, без роли (должна быть по умолчанию) | `{"name":"Nastya","email":"n@test.com"}` | 201 |
| 3 | Пустое имя | `{"name":"","email":"n@test.com"}` | 400 |
| 4 | Имя слишком короткое (1 символ) | `{"name":"N","email":"n@test.com"}` | 400 |
| 5 | Имя слишком длинное (101 символ) | `{"name":"AAAA...101 символ","email":"n@test.com"}` | 400 |
| 6 | Отсутствует имя | `{"email":"n@test.com"}` | 400 |
| 7 | Невалидный email | `{"name":"Nastya","email":"not-an-email"}` | 400 |
| 8 | Пустой email | `{"name":"Nastya","email":""}` | 400 |
| 9 | Невалидная роль | `{"name":"Nastya","email":"n@test.com","role":"superadmin"}` | 400 |
| 10 | Дублирующийся email | `{"name":"New","email":"existing@test.com"}` | 409 |
| 11 | Имя со спецсимволами | `{"name":"O'Brien","email":"o@test.com"}` | 201 |
| 12 | Пустое тело | `{}` | 400 |
| 13 | Null-тело | (пустой запрос) | 400 |

</details>

---

### Упражнение 4: Прочитайте логи

Даны следующие логи. Ответьте на вопросы:

```
10:30:00.100 INFO  c.e.controller.UserController : POST /api/users
10:30:00.102 DEBUG c.e.service.UserService : Creating user: name=Test, email=test@example.com
10:30:00.105 DEBUG c.e.service.UserService : Checking email uniqueness: test@example.com
10:30:00.150 DEBUG org.hibernate.SQL : select count(*) from users where email=?
10:30:00.155 DEBUG c.e.service.UserService : Email is unique, proceeding
10:30:00.160 DEBUG org.hibernate.SQL : insert into users (name, email, role, created_at) values (?, ?, ?, ?)
10:30:00.200 ERROR c.e.exception.GlobalExceptionHandler : DataIntegrityViolationException: Unique constraint violation
10:30:00.201 ERROR c.e.controller.UserController : Request failed with status 500
```

**Вопросы:**
1. Какой запрос был сделан?
2. Прошла ли проверка уникальности email?
3. Что пошло не так?
4. В каком слое произошла ошибка?
5. Почему проверка уникальности могла пройти, но INSERT всё равно упал?
6. Корректен ли код состояния 500, или он должен быть другим?

<details>
<summary>Нажмите, чтобы увидеть ответы</summary>

1. `POST /api/users` для создания пользователя с email `test@example.com`
2. Да, проверка прошла («Email is unique, proceeding»)
3. Нарушение ограничения уникальности при попытке вставить запись
4. Ошибка произошла в слое **репозитория** (во время SQL INSERT)
5. **Состояние гонки (race condition)**: Другой запрос создал пользователя с тем же email между проверкой (SELECT) и вставкой (INSERT). Это баг конкурентности!
6. Вероятно, должен быть `409 Conflict` — обработчик ошибок должен перехватывать `DataIntegrityViolationException` и возвращать более подходящий статус

</details>

---

### Упражнение 5: Исследуйте реальное приложение

Если у вас есть доступ к Spring Boot приложению:

1. **Найдите контроллер** для одного из API-эндпоинтов, которые вы тестируете
2. **Проследите запрос** через контроллер → сервис → репозиторий
3. **Посмотрите на сущность** — какой таблице базы данных она соответствует?
4. **Проверьте валидацию** — какие аннотации валидации на DTO запроса?
5. **Посмотрите на обработку ошибок** — как приложение обрабатывает исключения?
6. **Проверьте конфигурацию** — что находится в `application.properties`?

Если у вас нет доступа к реальному приложению, используйте [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) — пример Spring Boot приложения для изучения.

---

## 📚 Дополнительные ресурсы

- [Spring Boot Getting Started](https://spring.io/guides/gs/rest-service/) — Создание REST-сервиса шаг за шагом
- [Spring Petclinic](https://github.com/spring-projects/spring-petclinic) — Пример Spring Boot приложения
- [Baeldung](https://www.baeldung.com/) — Отличные туториалы по Java/Spring
- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/) — Официальная документация

---

## ✅ Самопроверка

После прохождения этого курса вы должны уметь:

- [ ] Объяснить разницу между фронтендом и бэкендом
- [ ] Описать слои: Контроллер → Сервис → Репозиторий → База данных
- [ ] Читать Spring Boot контроллер и понимать, какие эндпоинты он предоставляет
- [ ] Понимать, что делают аннотации `@GetMapping`, `@RequestBody`, `@PathVariable`
- [ ] Проследить, как HTTP-запрос проходит через бэкенд
- [ ] Понимать, как работает валидация, и проектировать тест-кейсы на основе правил валидации
- [ ] Читать логи бэкенда и находить ошибки
- [ ] Объяснить, что такое DTO и зачем они нужны
- [ ] Знать, что настраивается в `application.properties` / `application.yml`
- [ ] Более эффективно общаться с бэкенд-разработчиками
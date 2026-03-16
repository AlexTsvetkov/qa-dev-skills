# Курс 07: Автоматизированное тестирование

## 📖 Что вы изучите

- Что такое автоматизированное тестирование и почему оно важно
- Пирамида тестирования: модульные, интеграционные, сквозные тесты
- Типы автоматических тестов и когда использовать каждый
- Знакомство с тестовыми фреймворками (JUnit, TestNG, REST Assured)
- Как читать тестовый код и отчёты о тестах
- Роль QA в автоматизации тестирования

---

## 1. Зачем автоматизировать тесты?

### Проблемы ручного тестирования:
- ⏰ **Медленно** — тестирование всего занимает часы/дни
- 😴 **Подвержено ошибкам** — люди пропускают вещи при повторении задач
- 💰 **Дорого** — требует времени людей при каждом релизе
- 🔄 **Не воспроизводимо** — сложно провести абсолютно одинаковый тест каждый раз

### Преимущества автоматизированного тестирования:
- ⚡ **Быстро** — сотни тестов за минуты
- 🎯 **Стабильно** — один и тот же тест, одинаковым образом, каждый раз
- 🔁 **Воспроизводимо** — запускается при каждом изменении кода в CI/CD
- 📊 **Измеримо** — чёткие отчёты об успехе/провале
- 🛡️ **Страховка** — мгновенно ловит регрессии

---

## 2. Пирамида тестирования

```
            /\
           /  \        🔺 Сквозные (E2E) тесты
          / E2E\       Мало, медленные, дорогие
         /──────\      Тестируют всю систему через UI
        /        \
       / Интегр.  \    🔶 Интеграционные тесты
      /────────────\   Среднее количество, средняя скорость
     /              \  Тестируют совместную работу компонентов
    /  Модульные     \ 🟩 Модульные (Unit) тесты
   /──────────────────\ Много, быстрые, дешёвые
  /                    \ Тестируют отдельные функции/методы
 /______________________\
```

| Уровень | Что тестирует | Скорость | Количество | Пример |
|---------|--------------|----------|------------|--------|
| **Модульный** | Одну функцию/метод | Миллисекунды | Тысячи | «Возвращает ли `calculateTotal()` правильную сумму?» |
| **Интеграционный** | Компоненты вместе | Секунды | Сотни | «Возвращает ли API данные пользователя из базы данных?» |
| **E2E** | Весь пользовательский сценарий | Минуты | Десятки | «Может ли пользователь войти, добавить в корзину и оформить заказ?» |

---

## 3. Модульные (Unit) тесты

Тестируют отдельный фрагмент кода **изолированно**.

### Пример (JUnit 5 — Java):

```java
class CalculatorTest {

    @Test
    void shouldAddTwoNumbers() {
        Calculator calc = new Calculator();
        int result = calc.add(2, 3);
        assertEquals(5, result);
    }

    @Test
    void shouldThrowExceptionForDivisionByZero() {
        Calculator calc = new Calculator();
        assertThrows(ArithmeticException.class, () -> calc.divide(10, 0));
    }

    @Test
    void shouldReturnEmptyListWhenNoUsersFound() {
        // Подготовка (Arrange)
        UserRepository mockRepo = mock(UserRepository.class);
        when(mockRepo.findByRole("admin")).thenReturn(Collections.emptyList());
        UserService service = new UserService(mockRepo);

        // Действие (Act)
        List<User> result = service.getUsersByRole("admin");

        // Проверка (Assert)
        assertTrue(result.isEmpty());
    }
}
```

### Структура теста (паттерн AAA):

```
Arrange (Подготовка)  →  Настроить тестовые данные и условия
Act (Действие)        →  Выполнить тестируемый код
Assert (Проверка)     →  Убедиться, что результат корректен
```

---

## 4. Интеграционные тесты

Проверяют, что **несколько компонентов корректно работают вместе**.

### Интеграционный тест API (REST Assured — Java):

```java
class UserApiIntegrationTest {

    @Test
    void shouldCreateAndRetrieveUser() {
        // Создание пользователя
        String userId =
            given()
                .contentType(ContentType.JSON)
                .body("""
                    {
                        "name": "Nastya",
                        "email": "nastya@test.com",
                        "role": "qa"
                    }
                    """)
            .when()
                .post("/api/users")
            .then()
                .statusCode(201)
                .body("name", equalTo("Nastya"))
                .extract().path("id").toString();

        // Получение созданного пользователя
        given()
            .pathParam("id", userId)
        .when()
            .get("/api/users/{id}")
        .then()
            .statusCode(200)
            .body("name", equalTo("Nastya"))
            .body("email", equalTo("nastya@test.com"))
            .body("role", equalTo("qa"));
    }

    @Test
    void shouldReturn404ForNonExistentUser() {
        given()
        .when()
            .get("/api/users/99999")
        .then()
            .statusCode(404)
            .body("error", equalTo("NOT_FOUND"));
    }

    @Test
    void shouldReturn400ForInvalidEmail() {
        given()
            .contentType(ContentType.JSON)
            .body("""
                {
                    "name": "Test",
                    "email": "not-an-email"
                }
                """)
        .when()
            .post("/api/users")
        .then()
            .statusCode(400);
    }
}
```

### Паттерн REST Assured:

```
given()   →  Настроить запрос (заголовки, тело, параметры)
when()    →  Отправить запрос (GET, POST и т.д.)
then()    →  Проверить ответ (статус, тело, заголовки)
```

---

## 5. Сквозные (E2E) тесты

Тестируют полные пользовательские сценарии через **браузер или цепочку API**.

### Пример Selenium (Java):

```java
class LoginE2ETest {

    WebDriver driver;

    @BeforeEach
    void setUp() {
        driver = new ChromeDriver();
        driver.get("https://staging.example.com");
    }

    @Test
    void shouldLoginSuccessfully() {
        driver.findElement(By.id("email")).sendKeys("nastya@example.com");
        driver.findElement(By.id("password")).sendKeys("password123");
        driver.findElement(By.id("login-button")).click();

        WebElement welcome = new WebDriverWait(driver, Duration.ofSeconds(10))
                .until(ExpectedConditions.visibilityOfElementLocated(By.id("welcome-message")));

        assertEquals("Welcome, Nastya!", welcome.getText());
    }

    @AfterEach
    void tearDown() {
        driver.quit();
    }
}
```

---

## 6. Отчёты о тестах

### XML-отчёт JUnit (стандартный формат):

```xml
<testsuite name="UserApiTest" tests="5" failures="1" errors="0" time="2.345">
    <testcase name="shouldCreateUser" classname="UserApiTest" time="0.543"/>
    <testcase name="shouldGetUser" classname="UserApiTest" time="0.234"/>
    <testcase name="shouldUpdateUser" classname="UserApiTest" time="0.456"/>
    <testcase name="shouldDeleteUser" classname="UserApiTest" time="0.312">
        <failure message="Expected 204 but got 500">
            java.lang.AssertionError: Expected status 204 but got 500
            at UserApiTest.shouldDeleteUser(UserApiTest.java:78)
        </failure>
    </testcase>
    <testcase name="shouldReturn404" classname="UserApiTest" time="0.123"/>
</testsuite>
```

### Чтение отчётов о тестах:

| Поле | Значение |
|------|----------|
| `tests="5"` | Общее количество тестов |
| `failures="1"` | Тесты, которые выполнились, но проверка (assertion) провалилась |
| `errors="0"` | Тесты, которые аварийно завершились (исключение) |
| `time` | Сколько времени занял тест |
| `<failure>` | Детали того, что пошло не так |

---

## 7. Типичные проверки (Assertions)

| Проверка | Что проверяет |
|----------|--------------|
| `assertEquals(expected, actual)` | Значения равны |
| `assertNotEquals(a, b)` | Значения НЕ равны |
| `assertTrue(condition)` | Условие истинно |
| `assertFalse(condition)` | Условие ложно |
| `assertNull(value)` | Значение равно null |
| `assertNotNull(value)` | Значение НЕ равно null |
| `assertThrows(Exception.class, code)` | Код выбрасывает ожидаемое исключение |
| `assertThat(list).hasSize(5)` | Коллекция содержит 5 элементов |
| `assertThat(string).contains("text")` | Строка содержит подстроку |

---

## 8. Лучшие практики тестирования

| Практика | Зачем |
|----------|-------|
| **Одна проверка на концепцию** | Точно знать, что именно сломалось |
| **Описательные имена тестов** | `shouldReturn404WhenUserNotFound`, а не `test1` |
| **Независимые тесты** | Тесты не зависят от порядка выполнения друг друга |
| **Очищать тестовые данные** | Не оставлять мусор в базе данных |
| **Использовать тестовые фикстуры** | Переиспользуемая подготовка/очистка |
| **Не тестировать реализацию** | Тестировать поведение, а не внутренний код |
| **Быстрые тесты** | Медленные тесты не будут запускаться |

---

## 📝 Упражнения

### Упражнение 1: Определите тип теста

Классифицируйте каждый тест как модульный (Unit), интеграционный (Integration) или сквозной (E2E):

1. Тест, что `calculateDiscount(100, 0.1)` возвращает `90`
2. Тест, что POST `/api/orders` создаёт заказ в базе данных
3. Тест, что пользователь может просматривать товары, добавить в корзину и оформить заказ в браузере
4. Тест, что `validateEmail("bad")` возвращает `false`
5. Тест, что API логина возвращает валидный JWT-токен
6. Тест, что при нажатии «Добавить в корзину» показывается значок с количеством товаров

### Упражнение 2: Напишите тестовые сценарии

Для эндпоинта `POST /api/users` напишите тестовые сценарии, покрывающие:
- Позитивный сценарий (валидный запрос)
- Отсутствие обязательных полей
- Невалидные форматы данных
- Дублирование данных
- Аутентификация/авторизация

### Упражнение 3: Прочитайте отчёт о тестах

Дан следующий отчёт о тестах, ответьте на вопросы:
```
Tests run: 25, Failures: 3, Errors: 1, Skipped: 2
- FAILED: testCreateUserWithDuplicateEmail — Expected 409, got 500
- FAILED: testUpdateUserName — Expected "New Name", got "Old Name"
- FAILED: testDeleteUser — Expected 204, got 403
- ERROR: testGetUserOrders — NullPointerException at line 45
- SKIPPED: testAdminAccess — @Disabled("Auth not implemented yet")
- SKIPPED: testBulkImport — Condition not met
```

1. Сколько тестов прошло?
2. В чём разница между FAILED и ERROR?
3. Какой провал указывает на баг, а какой — на отсутствующую функциональность?
4. Почему тесты были пропущены?

---

## 📚 Дополнительные ресурсы

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/) — Фреймворк для тестирования на Java
- [REST Assured Documentation](https://rest-assured.io/) — Тестирование API на Java
- [Selenium Documentation](https://www.selenium.dev/documentation/) — Автоматизация браузера
- [Testing Pyramid (Martin Fowler)](https://martinfowler.com/articles/practical-test-pyramid.html) — Стратегия тестирования

---

## ✅ Самопроверка

- [ ] Объяснить пирамиду тестирования и каждый её уровень
- [ ] Описать разницу между модульными, интеграционными и E2E-тестами
- [ ] Прочитать тест на JUnit/REST Assured и понять, что он тестирует
- [ ] Понимать типичные проверки (`assertEquals`, `assertThrows` и т.д.)
- [ ] Прочитать отчёт о тестах и различить failures и errors
- [ ] Написать тестовые сценарии для API-эндпоинта
- [ ] Объяснить, почему автоматические тесты важны для CI/CD
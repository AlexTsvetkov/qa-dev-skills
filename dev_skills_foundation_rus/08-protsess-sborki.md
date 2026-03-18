# Курс 08: Процесс сборки

## 📖 Что вы изучите

- Что такое сборка (build) и почему она важна
- Как собираются Java-приложения (Maven и Gradle)
- Что происходит на каждом этапе сборки
- Знакомство с артефактами сборки (JAR, WAR, Docker-образы)
- Как читать логи сборки и устранять ошибки
- Что QA-инженеру нужно знать о процессе сборки

---

## 1. Что такое сборка?

**Сборка (build)** — это процесс преобразования исходного кода в запускаемое приложение.

```
Исходный код    →    ПРОЦЕСС СБОРКИ    →    Запускаемое приложение
(.java файлы)        (компиляция,            (.jar файл, Docker-образ)
                      тесты, упаковка)
```

### Почему QA важно знать о сборке:
- 🔍 **Ошибки сборки блокируют тестирование** — если сборка упала, тестировать нечего
- 📦 **Артефакты — это то, что вы тестируете** — вы тестируете собранное приложение, а не исходный код
- 🏷️ **Версии сборки** — нужно знать, какую именно сборку вы тестируете
- 🐛 **Логи сборки выявляют проблемы** — ошибки компиляции, проблемы с зависимостями

---

## 2. Инструмент сборки Maven

**Maven** — самый распространённый инструмент сборки для Java-проектов. Он использует файл `pom.xml` для конфигурации.

### Жизненный цикл сборки Maven:

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ validate  │──→│ compile  │──→│   test   │──→│ package  │──→│ verify   │──→│ install  │
│           │   │          │   │          │   │          │   │          │   │          │
│ Проверить │   │ Скомпи-  │   │ Запустить│   │ Создать  │   │ Запустить│   │ Установить│
│ проект    │   │ лировать │   │ модульные│   │ JAR/WAR  │   │ интегр.  │   │ в локаль-│
│ корректен │   │ Java-код │   │ тесты    │   │ файл     │   │ тесты    │   │ ный репо │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘
```

### Основные команды Maven:

| Команда | Что делает |
|---------|-----------|
| `mvn compile` | Компилирует исходный код |
| `mvn test` | Компиляция + запуск модульных тестов |
| `mvn package` | Компиляция + тесты + создание JAR/WAR |
| `mvn verify` | Компиляция + тесты + упаковка + интеграционные тесты |
| `mvn clean` | Удалить результаты предыдущей сборки (папка `target/`) |
| `mvn clean package` | Очистка + полная сборка с упаковкой |
| `mvn clean package -DskipTests` | Сборка без запуска тестов |

### pom.xml (Project Object Model):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <groupId>com.example</groupId>           <!-- Организация -->
    <artifactId>my-application</artifactId>  <!-- Имя проекта -->
    <version>1.2.3</version>                 <!-- Версия -->
    <packaging>jar</packaging>               <!-- Тип артефакта -->

    <dependencies>
        <!-- Библиотеки, необходимые проекту -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.0</version>
        </dependency>
        <!-- Тестовые зависимости -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>          <!-- Используется только для тестов -->
        </dependency>
    </dependencies>
</project>
```

### Результат сборки:

```
my-application/
├── src/                          ← Исходный код (входные данные)
├── target/                       ← Результат сборки (сгенерировано)
│   ├── classes/                  ← Скомпилированные Java-классы
│   ├── test-classes/             ← Скомпилированные тестовые классы
│   ├── surefire-reports/         ← Отчёты о тестах (XML)
│   ├── my-application-1.2.3.jar ← Собранное приложение ⭐
│   └── maven-status/            ← Метаданные сборки
└── pom.xml                       ← Конфигурация сборки
```

---

## 3. Инструмент сборки Gradle

**Gradle** — современная альтернатива Maven, использует `build.gradle` (Groovy) или `build.gradle.kts` (Kotlin).

### Пример build.gradle:

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
}

group = 'com.example'
version = '1.2.3'

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.postgresql:postgresql:42.7.0'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

### Основные команды Gradle:

| Команда | Что делает |
|---------|-----------|
| `gradle build` | Компиляция + тесты + упаковка |
| `gradle test` | Запуск тестов |
| `gradle clean build` | Очистка + полная сборка |
| `gradle bootJar` | Создание Spring Boot JAR |

---

## 4. Артефакты сборки

**Артефакт** — это результат процесса сборки, то, что вы разворачиваете и тестируете.

### Основные типы артефактов:

| Тип | Расширение | Описание |
|-----|-----------|----------|
| **JAR** | `.jar` | Java Archive — автономное Java-приложение |
| **WAR** | `.war` | Web Archive — веб-приложение для сервера приложений |
| **Docker-образ** | — | Контейнеризированное приложение |
| **NPM-пакет** | `.tgz` | JavaScript-пакет |

### Именование версий:

```
my-application-1.2.3.jar
                │ │ │
                │ │ └── Патч (исправление ошибок)
                │ └──── Минорная (новые функции, обратная совместимость)
                └────── Мажорная (ломающие изменения)
```

Специальные версии:
- `1.2.3-SNAPSHOT` — версия в разработке (нестабильная)
- `1.2.3-RC1` — Release Candidate (почти готова)
- `1.2.3-RELEASE` — готова к продакшену

---

## 5. Docker-сборка

Многие современные приложения упаковываются как **Docker-образы**.

### Dockerfile:

```dockerfile
FROM eclipse-temurin:17-jre-alpine       # Базовый образ с Java 17
WORKDIR /app                              # Рабочая директория
COPY target/my-application-1.2.3.jar app.jar  # Копирование собранного JAR
EXPOSE 8080                               # Порт, который слушает приложение
ENTRYPOINT ["java", "-jar", "app.jar"]    # Команда для запуска приложения
```

### Команды Docker-сборки:

```bash
# Собрать Docker-образ
docker build -t my-application:1.2.3 .

# Запустить контейнер
docker run -p 8080:8080 my-application:1.2.3

# Список образов
docker images

# Загрузить в реестр (поделиться с командой)
docker push registry.example.com/my-application:1.2.3
```

---

## 6. Чтение логов сборки

### Успешная сборка Maven:

```
[INFO] --- maven-compiler-plugin:3.11.0:compile (default-compile) ---
[INFO] Compiling 42 source files to /target/classes
[INFO]
[INFO] --- maven-surefire-plugin:3.2.0:test (default-test) ---
[INFO] Running com.example.UserServiceTest
[INFO] Tests run: 15, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 2.3 s
[INFO]
[INFO] --- maven-jar-plugin:3.3.0:jar (default-jar) ---
[INFO] Building jar: /target/my-application-1.2.3.jar
[INFO] BUILD SUCCESS
[INFO] Total time: 25 s
```

### Типичные ошибки сборки:

**1. Ошибка компиляции:**
```
[ERROR] COMPILATION ERROR :
[ERROR] /src/main/java/com/example/UserService.java:[42,15]
    cannot find symbol: variable userRepositry
[ERROR] BUILD FAILURE
```
→ Опечатка в коде (`userRepositry` вместо `userRepository`)

**2. Провал теста:**
```
[ERROR] Tests run: 15, Failures: 2, Errors: 0
[ERROR] UserServiceTest.testCreate:45 expected:<201> but was:<500>
[ERROR] BUILD FAILURE
```
→ Проверка (assertion) в тесте не прошла

**3. Зависимость не найдена:**
```
[ERROR] Failed to execute goal: Could not resolve dependencies
[ERROR] Could not find artifact com.example:my-library:jar:2.0.0
[ERROR] BUILD FAILURE
```
→ Необходимая библиотека не может быть загружена

---

## 📝 Упражнения

### Упражнение 1: Прочитайте pom.xml

Для заданного `pom.xml` определите:
1. Как называется проект и какая у него версия?
2. Какие зависимости он использует?
3. Какой тип артефакта будет создан (JAR или WAR)?

### Упражнение 2: Интерпретируйте логи сборки

Для каждого фрагмента лога сборки определите:
- На каком этапе произошла ошибка?
- В чём заключалась ошибка?
- Кто должен её исправить (разработчик, DevOps, QA)?

### Упражнение 3: Номера версий

Расположите эти версии в порядке от самой старой к самой новой:
`1.0.0`, `1.0.1`, `2.0.0-SNAPSHOT`, `1.1.0`, `2.0.0`, `1.1.0-RC1`

---

## 📚 Дополнительные ресурсы

- [Maven in 5 Minutes](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html) — Быстрое введение в Maven
- [Gradle Getting Started](https://docs.gradle.org/current/userguide/getting_started.html) — Основы Gradle
- [Docker Getting Started](https://docs.docker.com/get-started/) — Основы Docker
- [Semantic Versioning](https://semver.org/) — Стандарт нумерации версий

---

## ✅ Самопроверка

- [ ] Объяснить, что делает процесс сборки
- [ ] Понимать этапы жизненного цикла сборки Maven
- [ ] Прочитать `pom.xml` и определить ключевую информацию
- [ ] Знать основные команды Maven/Gradle
- [ ] Объяснить, что такое JAR, WAR и Docker-образы
- [ ] Читать логи сборки и определять ошибки
- [ ] Понимать семантическое версионирование (мажорная.минорная.патч)
- [ ] Объяснить разницу между SNAPSHOT и RELEASE версиями
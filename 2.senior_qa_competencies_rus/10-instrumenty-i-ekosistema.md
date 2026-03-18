# Курс 10: Инструменты и экосистема

## 📖 Что вы изучите

- Управление задачами и тестами (Jira, TestRail, Zephyr)
- CI/CD инструменты (GitHub Actions, Jenkins, GitLab CI/CD, Azure DevOps)
- Git и контроль версий
- Docker и Kubernetes для QA
- Мониторинг и наблюдаемость

---

## 1. Управление задачами и тестами

### Jira:

| Концепция | Описание |
|-----------|----------|
| **Epic** | Крупная функция, разбитая на Story |
| **Story** | Пользовательская история |
| **Bug** | Дефект с шагами воспроизведения |
| **Sprint** | Итерация 2-4 недели |

### TestRail / Zephyr:

```
Иерархия:
  Проект → Тест-сьют → Секция → Тест-кейс

Тест-кейс содержит:
  - Предусловия
  - Шаги
  - Ожидаемый результат
  - Статус: Passed / Failed / Blocked

Тестовый прогон:
  - Набор кейсов для выполнения
  - Привязка к версии/спринту
  - Отчёт о прохождении
```

---

## 2. CI/CD инструменты

### Сравнение:

| Инструмент | Конфигурация | Особенности |
|------------|-------------|-------------|
| **GitHub Actions** | `.github/workflows/*.yml` | Маркетплейс, встроен в GitHub |
| **Jenkins** | `Jenkinsfile` | Плагины, self-hosted |
| **GitLab CI** | `.gitlab-ci.yml` | Встроен в GitLab |
| **Azure DevOps** | `azure-pipelines.yml` | Интеграция с Microsoft |

### Пример GitHub Actions:

```yaml
name: Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm test
```

---

## 3. Git и контроль версий

### Основные команды:

```bash
git pull origin main           # Получить обновления
git checkout -b feature/login  # Новая ветка
git add .                      # Добавить изменения
git commit -m "Add tests"      # Зафиксировать
git push origin feature/login  # Отправить
git merge main                 # Слить main в ветку
git stash / git stash pop      # Отложить/вернуть
```

### Стратегии ветвления:

```
Git Flow: main → develop → feature branches → release
Trunk-Based: main + short-lived branches (1-2 дня)
```

---

## 4. Docker и Kubernetes

### Docker:

```bash
docker build -t my-tests .       # Собрать образ
docker run my-tests              # Запустить тесты
docker-compose up -d             # Поднять окружение
docker-compose down              # Остановить
```

### Kubernetes для QA:

```
Зачем QA знать K8s:
- Тестовые среды в кластере
- Понимание деплоя микросервисов
- Чтение логов подов
- Понимание networking между сервисами

kubectl get pods                 # Список подов
kubectl logs pod-name            # Логи пода
kubectl describe pod pod-name    # Детали пода
```

---

## 5. Мониторинг и наблюдаемость

### Три столпа:

| Столп | Инструменты | Для QA |
|-------|-------------|--------|
| **Логи** | ELK, Splunk, CloudWatch | Анализ ошибок |
| **Метрики** | Grafana, Prometheus | Мониторинг здоровья |
| **Трейсы** | Jaeger, Zipkin | Отслеживание запросов |

### Дашборд здоровья тестов:

```
Метрики на дашборде:
- Процент прохождения тестов (цель: >95%)
- Количество flaky-тестов (цель: <5%)
- Время выполнения CI (цель: <15 мин)
- Покрытие кода (цель: >80%)
- Открытые баги по серьёзности
```

---

## ✅ Вопросы для самопроверки

- [ ] Какие CI/CD инструменты вы знаете и чем они отличаются?

<details>
<summary>Ответ</summary>

**GitHub Actions** — YAML-конфигурация, маркетплейс, встроен в GitHub. **Jenkins** — гибкий, плагины, self-hosted, Jenkinsfile. **GitLab CI** — встроен в GitLab, `.gitlab-ci.yml`. **Azure DevOps** — интеграция с Microsoft. Выбор зависит от платформы кода и требований команды.

</details>

- [ ] Зачем QA знать Docker и Kubernetes?

<details>
<summary>Ответ</summary>

**Docker**: воспроизводимые тестовые среды, запуск тестов в контейнерах, docker-compose для поднятия зависимостей. **K8s**: понимание деплоя микросервисов, чтение логов подов, понимание сетевого взаимодействия сервисов, тестирование в кластерных средах.

</details>

- [ ] Что включает мониторинг и наблюдаемость?

<details>
<summary>Ответ</summary>

Три столпа: **Логи** (ELK, Splunk — анализ ошибок), **Метрики** (Grafana — здоровье системы), **Трейсы** (Jaeger — отслеживание запросов). Для QA: дашборды здоровья тестов, алерты на падения, мониторинг flaky-тестов и времени CI.

</details>
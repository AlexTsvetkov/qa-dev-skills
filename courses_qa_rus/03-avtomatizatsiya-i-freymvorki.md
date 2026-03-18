# Курс 03: Автоматизация и экспертиза фреймворков

## 📖 Что вы изучите

- Проектирование и поддержка фреймворков автоматизации с нуля
- Стратегия автоматизации: UI, API, E2E, интеграционные и регрессионные тесты
- Современные инструменты (Selenium, Playwright, Cypress и др.)
- Практики код-ревью для автоматизации
- Лучшие практики и паттерны автоматизации
- Мониторинг и снижение нестабильных тестов
- Контроль версий с Git, стратегии ветвления

---

## 1. Проектирование фреймворка автоматизации

### Архитектура фреймворка:

```
test-framework/
├── tests/
│   ├── e2e/              # UI-тесты
│   ├── api/              # API-тесты
│   └── integration/      # Интеграционные тесты
├── pages/                # Page Object Model
│   ├── LoginPage.ts
│   ├── CheckoutPage.ts
│   └── BasePage.ts
├── api-clients/          # API-клиенты
│   ├── UserAPI.ts
│   └── OrderAPI.ts
├── fixtures/             # Тестовые данные
│   ├── users.json
│   └── products.json
├── utils/                # Утилиты
│   ├── helpers.ts
│   └── constants.ts
├── config/               # Конфигурация по средам
│   ├── dev.env
│   └── staging.env
└── reports/              # Отчёты о тестировании
```

### Page Object Model (POM):

```typescript
// pages/LoginPage.ts
export class LoginPage {
  constructor(private page: Page) {}

  // Локаторы — приватные, определены в одном месте
  private emailInput = this.page.getByLabel('Email');
  private passwordInput = this.page.getByLabel('Пароль');
  private loginButton = this.page.getByRole('button', { name: 'Войти' });
  private errorMessage = this.page.getByRole('alert');

  // Действия — публичные методы
  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  // Проверки
  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }
}
```

---

## 2. Стратегия автоматизации тестирования

### Пирамида тестирования:

```
        /  E2E  \          Мало, медленно, дорого
       /  тесты  \         (Браузерные/UI-тесты)
      /───────────\
     / Интеграционные\     Среднее количество
    /    тесты        \    (API, сервисные тесты)
   /───────────────────\
  /   Модульные тесты    \ Много, быстро, дёшево
 /________________________\
```

### Что автоматизировать vs что нет:

| ✅ Автоматизировать | ❌ Оставить ручным |
|---------------------|-------------------|
| Регрессионные тесты | Исследовательское тестирование |
| Смоук-тесты | Тестирование удобства использования |
| Тесты с данными | Разовые проверки |
| API-валидации | Визуальная/субъективная оценка |
| Кросс-браузерные проверки | Быстро меняющиеся фичи |

---

## 3. Современные инструменты автоматизации

### Сравнение инструментов:

| Инструмент | Языки | Лучше для | Скорость |
|-----------|-------|-----------|---------|
| **Playwright** | JS/TS, Python, Java, C# | Современные веб-приложения | Быстрый |
| **Cypress** | JavaScript/TypeScript | Фронтенд-тестирование | Быстрый |
| **Selenium** | Java, Python, C#, JS | Широкая совместимость | Средний |
| **Appium** | Java, Python, JS | Мобильное тестирование | Медленный |

### Пример теста на Playwright:

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../pages/LoginPage';

test.describe('Авторизация', () => {
  test('успешный вход', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await page.goto('/login');

    await loginPage.login('user@test.com', 'password123');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Добро пожаловать')).toBeVisible();
  });

  test('неверный пароль', async ({ page }) => {
    const loginPage = new LoginPage(page);
    await page.goto('/login');

    await loginPage.login('user@test.com', 'wrongpass');

    await loginPage.expectError('Неверные учётные данные');
  });
});
```

---

## 4. Код-ревью автоматизации

### Чек-лист ревью:

| Категория | Проверка |
|----------|---------|
| **Надёжность** | Нет жёстких ожиданий (hard waits), используются умные ожидания |
| **Изоляция** | Тест не зависит от других тестов |
| **Данные** | Уникальные данные для каждого теста |
| **Читаемость** | Понятные названия, следование POM |
| **Поддерживаемость** | Нет дублирования, общие утилиты |
| **Производительность** | Разумное время выполнения |

---

## 5. Борьба с нестабильными тестами

### Частые причины и решения:

| Причина | Решение |
|---------|---------|
| Жёсткие ожидания (`sleep`) | Использовать `waitFor`, `expect().toBeVisible()` |
| Зависимость от данных | Уникальные данные через setup/teardown |
| Общее состояние | Изоляция тестов |
| Нестабильность среды | Retry для инфраструктурных проблем |
| Хрупкие селекторы | `data-testid` вместо CSS-классов |

---

## 6. Git для QA-автоматизации

### Основные команды:

```bash
# Создать ветку для тестов
git checkout -b feature/add-checkout-tests

# Зафиксировать изменения
git add tests/
git commit -m "Добавить тесты оплаты"

# Обновить ветку
git fetch origin
git rebase origin/main

# Создать Pull Request
git push origin feature/add-checkout-tests
```

---

## 📝 Упражнения

### Упражнение 1: Проектирование POM

Спроектируйте Page Object для страницы регистрации (имя, email, пароль, подтверждение пароля, кнопка регистрации).

<details>
<summary>Нажмите для ответа</summary>

```typescript
export class RegistrationPage {
  constructor(private page: Page) {}

  private nameInput = this.page.getByLabel('Имя');
  private emailInput = this.page.getByLabel('Email');
  private passwordInput = this.page.getByLabel('Пароль');
  private confirmInput = this.page.getByLabel('Подтверждение пароля');
  private registerButton = this.page.getByRole('button', { name: 'Зарегистрироваться' });
  private errorMessage = this.page.getByRole('alert');

  async register(name: string, email: string, password: string) {
    await this.nameInput.fill(name);
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.confirmInput.fill(password);
    await this.registerButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }

  async expectSuccess() {
    await expect(this.page).toHaveURL('/welcome');
  }
}
```

</details>

---

## ✅ Вопросы для самопроверки

- [ ] Как спроектировать фреймворк автоматизации с нуля?

<details>
<summary>Ответ</summary>

Шаги: (1) Выбор инструментов по требованиям (Playwright для веба, REST Assured для API). (2) Архитектура: Page Object Model для UI, сервисные классы для API. (3) Структура проекта: тесты, страницы, фикстуры, утилиты, конфигурация. (4) Конфигурация по средам. (5) Интеграция с CI/CD. (6) Отчётность: HTML-отчёты, скриншоты при ошибках. (7) План поддержки: код-ревью, мониторинг нестабильности.

</details>

- [ ] Как бороться с нестабильными тестами?

<details>
<summary>Ответ</summary>

(1) **Обнаружение**: мониторинг стабильности — тест с >2% падений помечается. (2) **Карантин**: перенос в отдельный набор, чтобы не блокировать CI. (3) **Диагностика**: выявление причины (тайминг, данные, среда, общее состояние). (4) **Исправление**: устранение причины, а не симптомов. (5) **Предотвращение**: чек-лист на код-ревью, мониторинг в CI. Цель: нестабильность < 2%.

</details>

- [ ] Что автоматизировать, а что оставить ручным?

<details>
<summary>Ответ</summary>

**Автоматизировать**: регрессия, смоук-тесты, тесты с данными, API-валидации — всё, что запускается регулярно. **Оставить ручным**: исследовательское тестирование, UX-оценка, разовые проверки, быстро меняющиеся фичи. Приоритизация по ROI: частота выполнения × риск ошибки × ручные усилия. Следовать пирамиде — больше автоматизации на уровне API.

</details>
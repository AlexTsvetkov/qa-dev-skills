# AI Prompts for Course Generation

A collection of reusable AI prompts to generate new courses, translate existing ones, and expand the QA Dev Skills curriculum.

---

## Table of Contents

- [1. Generate a Complete Course (English)](#1-generate-a-complete-course-english)
- [2. Generate a Complete Course (Russian)](#2-generate-a-complete-course-russian)
- [3. Translate an Existing Course to Russian](#3-translate-an-existing-course-to-russian)
- [4. Generate a New Course Module / Part](#4-generate-a-new-course-module--part)
- [5. Generate Exercises for an Existing Course](#5-generate-exercises-for-an-existing-course)
- [6. Generate Self-Check Questions](#6-generate-self-check-questions)
- [7. Expand an Existing Course Section](#7-expand-an-existing-course-section)
- [8. Generate a Course Roadmap (README Section)](#8-generate-a-course-roadmap-readme-section)
- [9. Generate a Practical Lab / Hands-On Exercise](#9-generate-a-practical-lab--hands-on-exercise)
- [10. Review and Improve an Existing Course](#10-review-and-improve-an-existing-course)

---

## 1. Generate a Complete Course (English)

Use this prompt to create a full course from scratch in the standard format used across the project.

```
You are a senior technical educator creating a structured learning course for a Senior Manual QA Engineer who is expanding their development and QA skills. The course is part of the "QA Dev Skills" curriculum.

Generate a complete course on the topic: **[TOPIC NAME]**

The course must follow this exact structure:

1. **Title**: `# Course [NUMBER]: [TOPIC NAME]`

2. **What You Will Learn** (`## 📖 What You Will Learn`):
   - A bullet list of 6-10 specific learning objectives
   - Each item should be concrete and measurable

3. **Main Content Sections** (numbered `## 1.`, `## 2.`, etc.):
   - 5-8 major sections covering the topic comprehensively
   - Each section should include:
     - Clear explanations written for a QA professional (not a complete beginner, but also not assuming deep dev experience)
     - Tables for comparisons, reference data, and structured information
     - Code examples, diagrams, or ASCII visualizations where applicable
     - Real-world examples relevant to QA work
     - "QA Perspective" callouts where relevant (what to test, what to watch for)
   - Use ```code blocks``` for code examples, HTTP requests, terminal commands, etc.
   - Use markdown tables for structured comparisons
   - Use ASCII diagrams for workflows and architecture

4. **Exercises** (`## 📝 Exercises`):
   - 4-6 practical exercises of increasing difficulty
   - Each exercise should have:
     - Clear instructions and scenario description
     - Specific questions or tasks to complete
     - Answers wrapped in `<details><summary>Click to see answers</summary>...</details>` tags
   - Include a mix of:
     - Conceptual understanding exercises
     - Practical hands-on exercises
     - Real-world QA scenario exercises

5. **Additional Resources** (`## 📚 Additional Resources`):
   - 4-6 links to high-quality external resources (documentation, books, tools, blogs)
   - Each resource should have a brief description of what it offers

6. **Self-Check Questions** (`## ✅ Self-Check Questions`):
   - 6-10 self-check items formatted as: `- [ ] [Question text]`
   - Each question should have a detailed answer wrapped in `<details><summary>Answer</summary>...</details>` tags
   - Answers should be comprehensive (3-8 sentences) and demonstrate deep understanding

**Style guidelines:**
- Write in a friendly but professional tone
- Use emoji in section headers (📖, 📝, 📚, ✅, 🤔, 🧠, etc.)
- Address the reader directly ("you") where appropriate
- Include practical examples that a QA engineer would encounter in their daily work
- Use analogies to explain complex concepts
- Make the content self-contained — the reader should be able to learn the topic from this course alone

**Target audience:** Senior Manual QA Engineer with solid testing experience but limited development/DevOps knowledge. They understand testing concepts well and are learning the technical implementation side.

Generate the complete course now.
```

---

## 2. Generate a Complete Course (Russian)

Use this prompt to create a full course directly in Russian.

```
Ты — опытный технический преподаватель, создающий структурированный учебный курс для Senior Manual QA Engineer, который расширяет свои навыки в разработке и QA. Курс является частью программы "QA Dev Skills".

Создай полный курс на тему: **[НАЗВАНИЕ ТЕМЫ]**

Курс должен строго следовать этой структуре:

1. **Заголовок**: `# Курс [НОМЕР]: [НАЗВАНИЕ ТЕМЫ]`

2. **Что вы изучите** (`## 📖 Что вы изучите`):
   - Список из 6-10 конкретных целей обучения
   - Каждый пункт должен быть конкретным и измеримым

3. **Основные разделы** (нумерованные `## 1.`, `## 2.` и т.д.):
   - 5-8 основных разделов, всесторонне охватывающих тему
   - Каждый раздел должен включать:
     - Понятные объяснения для QA-специалиста (не для полного новичка, но и без предположения глубокого опыта разработки)
     - Таблицы для сравнений, справочных данных и структурированной информации
     - Примеры кода, диаграммы или ASCII-визуализации где уместно
     - Примеры из реальной практики QA
     - Врезки "Взгляд QA" (что тестировать, на что обращать внимание)
   - Используй ```блоки кода``` для примеров кода, HTTP-запросов, команд терминала
   - Используй markdown-таблицы для структурных сравнений
   - Используй ASCII-диаграммы для рабочих процессов и архитектуры

4. **Упражнения** (`## 📝 Упражнения`):
   - 4-6 практических упражнений возрастающей сложности
   - Каждое упражнение должно иметь:
     - Чёткие инструкции и описание сценария
     - Конкретные вопросы или задания
     - Ответы в тегах `<details><summary>Нажмите, чтобы увидеть ответ</summary>...</details>`
   - Включи:
     - Упражнения на понимание концепций
     - Практические упражнения (hands-on)
     - Упражнения с реальными QA-сценариями

5. **Дополнительные ресурсы** (`## 📚 Дополнительные ресурсы`):
   - 4-6 ссылок на качественные внешние ресурсы
   - Краткое описание каждого ресурса

6. **Вопросы для самопроверки** (`## ✅ Вопросы для самопроверки`):
   - 6-10 вопросов в формате: `- [ ] [Текст вопроса]`
   - Каждый вопрос с подробным ответом в `<details><summary>Ответ</summary>...</details>`
   - Ответы должны быть подробными (3-8 предложений)

**Стиль:**
- Дружелюбный, но профессиональный тон
- Emoji в заголовках разделов (📖, 📝, 📚, ✅, 🤔, 🧠 и т.д.)
- Обращение к читателю на "вы"
- Практические примеры из повседневной работы QA-инженера
- Аналогии для объяснения сложных концепций
- Техническая терминология на русском с английскими терминами в скобках при первом упоминании

**Целевая аудитория:** Senior Manual QA Engineer с отличным опытом тестирования, но ограниченными знаниями в разработке/DevOps.

Создай полный курс.
```

---

## 3. Translate an Existing Course to Russian

Use this prompt to translate an existing English course to Russian while maintaining the structure and adapting terminology.

```
Переведи следующий учебный курс с английского на русский язык.

Правила перевода:
1. **Сохрани точную структуру** markdown-файла (заголовки, таблицы, блоки кода, списки, теги details/summary)
2. **Техническая терминология**: при первом упоминании используй русский перевод с английским оригиналом в скобках, далее по тексту можно использовать только русский вариант. Например: "Непрерывная интеграция (Continuous Integration, CI)"
3. **Не переводи**:
   - Код внутри блоков кода (```code```)
   - URL-адреса и ссылки
   - Названия инструментов и технологий (Jira, Jenkins, Postman, Docker и т.д.)
   - HTTP-методы (GET, POST, PUT, DELETE)
   - Общепринятые аббревиатуры (API, CI/CD, SQL, HTML, JSON и т.д.)
4. **Замени** текст в тегах `<summary>Click to see answers</summary>` на `<summary>Нажмите, чтобы увидеть ответ</summary>`
5. **Адаптируй** примеры и аналогии для русскоязычной аудитории, сохраняя смысл
6. **Раздел ресурсов**: оставь оригинальные ссылки, переведи описания. Если есть хорошие русскоязычные альтернативы — можно добавить их дополнительно
7. **Emoji** в заголовках — оставить без изменений
8. **Заголовок файла**: переведи название курса, сохранив номер

Вот курс для перевода:

---
[ВСТАВЬ СОДЕРЖИМОЕ АНГЛИЙСКОГО КУРСА ЗДЕСЬ]
---

Выполни полный перевод, не пропуская ни одного раздела.
```

---

## 4. Generate a New Course Module / Part

Use this prompt to plan and generate an entire new part (module) of courses, similar to Parts 1-3 in the project.

```
You are designing a new module for the "QA Dev Skills" curriculum — a structured learning program for a Senior Manual QA Engineer expanding into development and advanced QA skills.

The existing modules are:
- Part 1: Dev Skills Foundation (10 courses — HTTP, APIs, Postman, Java backend, CI/CD, Jenkins, automated testing, build process, deployment, browser DevTools)
- Part 2: Senior QA Competencies (11 courses — QA leadership, Agile/SDLC, automation frameworks, API testing, SQL/databases, non-functional testing, AI testing, soft skills, test design, tools ecosystem, interview prep)
- Part 3: Deep Dive QA Topics (6 courses — test strategy, methodologies, quality management, QA vs QC, authentication, test design techniques)

Create a new **Part [NUMBER]: [MODULE NAME]** focused on: **[MODULE TOPIC/THEME]**

Please provide:

1. **Module Overview**: 2-3 sentences describing the module's purpose and what it covers

2. **Course Roadmap Table**:
   | # | Course | Topic |
   |---|--------|-------|
   List 6-12 courses in a logical learning sequence

3. **For each course**, provide:
   - Course title
   - 1-2 sentence description
   - 5-8 key topics that will be covered
   - Prerequisites (which courses from existing modules should be completed first)

4. **Recommended Learning Path**: Where this module fits in the overall curriculum (after Part 1? After Part 2? Parallel?)

5. **Directory structure**: Following the existing pattern:
   - English: `[number].[module_name_snake_case]/`
   - Russian: `[number].[module_name_snake_case]_rus/`
   - File naming: `[number]-[topic-slug].md`

Then generate the FULL content for the first course in this module (following the standard course structure from the project).
```

---

## 5. Generate Exercises for an Existing Course

Use this prompt to add more exercises to an existing course.

```
You are creating practical exercises for a QA Dev Skills course.

The course topic is: **[TOPIC NAME]**

Here is the current course content for context:

---
[PASTE THE COURSE CONTENT OR KEY SECTIONS HERE]
---

Generate **[NUMBER]** new exercises that:

1. **Vary in difficulty** — include easy, medium, and challenging exercises
2. **Are practical** — focus on real-world QA scenarios, not just theory
3. **Follow this format**:

### Exercise [N]: [Exercise Title]

[Scenario description and context]

**Tasks:**
1. [Task 1]
2. [Task 2]
...

<details>
<summary>Click to see answers</summary>

[Detailed answers with explanations]

</details>

**Exercise types to include:**
- Conceptual: "Explain the difference between X and Y in this scenario"
- Hands-on: "Use [tool] to perform [action] and analyze the results"
- Scenario-based: "You are the senior QA on a project where... What do you do?"
- Troubleshooting: "Given this [output/error/configuration], identify the problem"
- Design: "Create a [test plan/strategy/matrix] for the following scenario"

Make exercises relevant to a Senior Manual QA Engineer who is building development skills.
```

---

## 6. Generate Self-Check Questions

Use this prompt to generate or expand self-check questions for a course.

```
You are creating self-check questions for a QA Dev Skills course.

The course topic is: **[TOPIC NAME]**
The key concepts covered are:
- [Concept 1]
- [Concept 2]
- [Concept 3]
...

Generate **[NUMBER]** self-check questions that:

1. Cover all the major concepts in the course
2. Test real understanding, not just memorization
3. Follow this exact format:

- [ ] [Question phrased as "Explain...", "What is...", "How do you...", "What is the difference between..."]

<details>
<summary>Answer</summary>

[Comprehensive answer in 3-8 sentences. The answer should demonstrate deep understanding and include practical examples where relevant.]

</details>

**Guidelines:**
- Questions should be answerable from the course content alone
- Include questions about practical application, not just definitions
- At least 2 questions should involve comparing or contrasting concepts
- At least 1 question should focus on "QA perspective" — how this topic applies to testing
- Answers should be detailed enough that a reader could learn from just reading the Q&A
```

---

## 7. Expand an Existing Course Section

Use this prompt to add depth to a section of an existing course.

```
You are expanding a section of a QA Dev Skills course. The target audience is a Senior Manual QA Engineer building development skills.

**Course**: [COURSE NAME]
**Section to expand**: [SECTION NAME/NUMBER]

Here is the current content of this section:

---
[PASTE THE CURRENT SECTION CONTENT]
---

Please expand this section by adding:

1. **More detailed explanations** of the concepts (2-3 additional paragraphs)
2. **A comparison table** if there are multiple related concepts
3. **A practical example** showing how this applies in real QA work
4. **A "🤔 Common Misconceptions"** callout addressing 2-3 things people often get wrong
5. **A "🧠 QA Perspective"** callout explaining what QA engineers should pay attention to
6. **An ASCII diagram or code example** that visualizes the concept

Maintain the existing writing style — friendly but professional, with emoji in headers, practical examples, and clear explanations.
```

---

## 8. Generate a Course Roadmap (README Section)

Use this prompt to generate a README section for a new module.

```
Generate a README.md section for a new module in the QA Dev Skills curriculum.

**Module**: Part [NUMBER]: [MODULE NAME]
**Description**: [Brief description of the module]
**Courses**:
1. [Course 1 title] — [brief topic description]
2. [Course 2 title] — [brief topic description]
...

Follow the exact format used in the existing README. Include:

1. A section header with emoji: `## [EMOJI] Part [N]: [Module Name]`
2. A brief description of the module
3. A paragraph about file locations: `Courses located in [directory]/ (English) and [directory_rus]/ (Russian).`
4. `### 📋 Course Roadmap` with a markdown table:
   | # | Course | Topic |
   with links to the course files

5. `### 🇷🇺 Часть [N]: [Module Name in Russian]` with a Russian version of the table

Use the same link format: `[Course Title](directory/filename.md)`
```

---

## 9. Generate a Practical Lab / Hands-On Exercise

Use this prompt to create a standalone practical exercise or lab.

```
Create a detailed hands-on lab exercise for a QA Dev Skills course.

**Topic**: [TOPIC]
**Tools needed**: [List of tools — e.g., Postman, curl, browser DevTools, terminal]
**Estimated time**: [TIME] minutes

The lab should:

1. **Start with a clear objective** — what the student will accomplish
2. **Provide step-by-step instructions** — numbered steps with exact commands or actions
3. **Include expected output** — show what the student should see at each step
4. **Have checkpoint questions** — "At this point, you should see X. If not, check Y."
5. **End with a challenge** — an open-ended task that requires applying what was learned
6. **Include a "What You Learned" summary** — bullet list of skills practiced

Format all commands in code blocks. Include screenshots descriptions where visual output is important (describe what should appear on screen).

**Target audience**: Senior Manual QA Engineer with basic command line experience. They may be using this tool for the first time.
```

---

## 10. Review and Improve an Existing Course

Use this prompt to get AI feedback on an existing course and suggestions for improvement.

```
Review the following QA Dev Skills course and provide detailed feedback and improvement suggestions.

---
[PASTE THE FULL COURSE CONTENT HERE]
---

Please evaluate and suggest improvements for:

1. **Content Completeness** (1-10):
   - Are all important subtopics covered?
   - What topics are missing that a Senior QA should know?
   - Are there any outdated references or practices?

2. **Clarity and Readability** (1-10):
   - Are explanations clear for someone with QA but limited dev experience?
   - Are there sections that are too brief or too verbose?
   - Is the flow logical?

3. **Practical Value** (1-10):
   - Are exercises realistic and relevant?
   - Do examples reflect real-world QA scenarios?
   - Are there enough hands-on opportunities?

4. **Structure and Formatting** (1-10):
   - Does it follow the standard course template?
   - Are tables, code blocks, and diagrams used effectively?
   - Is the self-check section comprehensive?

5. **Specific Improvements**:
   - List 3-5 concrete additions or changes to make the course better
   - Provide the actual content for each improvement (ready to paste in)

6. **Missing Resources**:
   - Suggest 2-3 additional external resources that would complement the course
```

---

## Tips for Using These Prompts

1. **Replace placeholders** — All `[BRACKETED TEXT]` should be replaced with your specific content
2. **Provide context** — When expanding or translating, paste the relevant existing content for reference
3. **Iterate** — Generate a draft, review it, then ask for specific improvements
4. **Consistency** — Always reference existing courses to maintain consistent style and depth
5. **Verify technical accuracy** — AI-generated content should be reviewed for technical correctness, especially code examples and tool-specific instructions
6. **File naming** — Follow the existing pattern: `XX-topic-slug.md` (English) and transliterated Russian equivalents
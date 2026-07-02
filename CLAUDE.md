# Инструкции для всех репозиториев

> Slim-ядро: триггеры + правила. Детали → `memory/protocol-*.md`, `.claude/rules/`, `.claude/skills/`.
> Синхронизация: `scripts/template-sync.sh` · Агент-специфичные инструкции: Hermes → Aisystant MCP `get_instructions`.

## 1. Архитектура репозиториев

| Тип | Что содержит | Первоисточник |
|-----|-------------|---------------|
| **Base** (Принципы + Форматы) | ZP, FPF, SPF, FMT-* | Да (платформа) |
| **Pack** | Паспорт предметной области | Да (пользователь) |
| **DS** (instrument/governance/surface) | Код, планы, курсы | Нет (производное от Pack) |

**Fallback Chain:** DS → Pack → Base (SPF → FPF → ZP). **Pack = source-of-truth доменного знания.**
**Pack Creation Gate:** хочешь Pack → `/pack-new`. Имя = существительное-домен.
Детали типов: → `memory/repo-type-rules.md`

## 2. ОРЗ-фрактал (Открытие → Работа → Закрытие)

| Масштаб | Открытие | Работа | Закрытие |
|---------|----------|--------|----------|
| **Сессия** | `protocol-open.md § Сессия` | `protocol-work.md` | `/run-protocol close` |
| **День** | `/day-open` | Между Open и Close | `/run-protocol day-close` |
| **Неделя** | — | — | `/run-protocol week-close` |
| **Месяц** | — | — | `/month-close` |

### Блокирующие правила

> Source-of-truth: `PACK-agent-rules/rules/AR.NNN.md`. Структурные (1-5) > поведенческих (6-10).

1. **WP Gate:** ЛЮБОЕ задание → протокол Открытия → ДО начала работы. Новый РП: объявить (Роль пользователя · Роль Claude · Работа · РП · ТВС · Класс верификации · Метод · ~Xh · Модель) → ждать «да». Шаги 3-4 → `memory/protocol-open.md`.
2. **Push:** «заливай/запуши/закрывай» → commit+push без вопросов. При Close: `git status` по ВСЕМ репо → незафиксированное → commit+push ДО следующего шага.
3. **Close:** Триггер Закрытия → протокол Закрытия → выполнить.
4. **Pull-on-Touch:** `git pull --rebase` при ПЕРВОМ обращении к репо за сессию (lazy, один раз). Конфликт → вариант А: stash + «potentially stale». Сетевой fail → potentially stale.
5. **Чеклист-верификация:** Quick/Day Close → sub-agent Haiku R23. Исключение: сессия ≤15 мин или без изменений файлов.
6. **Hooks/Scripts Bypass Gate (БЛОКИРУЮЩЕЕ):** НЕ менять `.claude/hooks/`, `.claude/scripts/`, `.iwe-runtime/`, `FMT-exocortex-template/` без явного разрешения. Хук заблокировал → (1) НЕ обходить (2) записать в `inbox/bugs/bug-YYYY-MM-DD-<тема>.md` (3) сообщить пилоту (4) ждать инструкций.
7. **Автономность:** НЕ спрашивать «добавить/продолжить/записать?». Задание → выполни → отчитайся. Исключения: необратимое действие · WP Gate Ритуал · Choice-question («X или Y?»).
8. **Напоминания:** «напомни через X» → `send_telegram_message` (schedule_at) + ScheduleWakeup.
9. **Финиш > отлог:** новая задача → делаю сейчас. Исключения: бюджет ×2-×3 · требует ArchGate · контекст переключился. Если >15 мин + новый артефакт → WP Gate.

### Протокол Работы → `memory/protocol-work.md`

**Capture-to-Pack:** на рубеже — «Capture: [что] → [куда]». Routing Gate (DP.KR.001 §5) при создании артефакта.

| Pre-action Gate | Когда |
|-----------------|-------|
| Repo-Touch Gate | Первое действие в репо → читать `<repo>/CLAUDE.md` |
| Routing Gate | Создание/размещение артефакта → DP.KR.001 §5 |
| ArchGate | Архитектурное решение → `/archgate` |
| Security Gate | РП затрагивает PII → §Б чеклист ArchGate ДО реализации |
| IntegrationGate | Новый инструмент/агент/система → скилл `integration-gate` |
| LegacyPortGate | Замена legacy → 15-мин субагент «как сейчас?» ДО решения |

## 3. Описания методов (PROCESSES.md)

≤15 мин — не нужен. Внутри системы — `<repo>/PROCESSES.md`.

## 4. Memory (Слой 3)

| Ситуация | Читай |
|----------|-------|
| Файлы/репо | `memory/navigation.md` |
| Pack-репо | `memory/repo-type-rules.md` |
| Терминология | `memory/hard-distinctions.md` |
| FPF/SOTA/Роли | `memory/fpf-reference.md`, `memory/sota-reference.md`, `memory/roles.md` |

Политика: ≤15 HOT+WARM, суммарно ≤150 строк hot. CLAUDE.md = ядро (цель ≤150). `memory/` = симлинк auto-memory.

## 5. АрхГейт — ОБЯЗАТЕЛЬНАЯ оценка

> **БЛОКИРУЮЩЕЕ.** Архитектурное решение → `/archgate` (скилл `archgate`): профиль ЭМОГССБ, conjunctive screening.

## 6. Форматирование → `.claude/rules/formatting.md`

## Различения → `.claude/rules/distinctions.md`

## 7. Обновление этого файла

> **3 слоя:** L1 (§1-§7) = платформа. L2 (§8) = staging. L3 (§9) = авторское.

- Протоколы → `memory/protocol-*.md` · Правило (1-3 строки) → CLAUDE.md · Доменное → Pack
- §8+§9 (staging/авторское) → скилл `author-mode`

<!-- PLATFORM-END -->

---

## Agent Core (единое ядро для всех агентов)

> WP-394 Ф4.2. Единое ядро для Claude, Kimi, Hermes. Правки — сюда.

<!-- SYNC-CORE-START -->

## WP Gate — CRITICAL

**ЛЮБОЕ задание → протокол Открытия → ДО начала работы.** При создании нового РП: объявить роль, работу, РП, класс верификации, метод, оценку, модель. Дождаться согласования пилота.

## Git Staging — CRITICAL

**NEVER use `git add -u`, `git add .`, or `git add -A`.** Picks up other agents' changes.

**Always stage only specific files you edited:**
```bash
git add path/to/specific-file.md   # correct
# git add -u / git add . / git add -A  — FORBIDDEN
```

**Before every commit:** `git diff --cached --name-only` → confirm all files belong to current WP/context. Unexpected files → `git restore --staged <file>`.

## Artifact Naming
**Do not invent artifact names.** Names come from the plan/task. If silent on name — report "need clarification on name."
## Drift Reporting
Discrepancy found → **Report to pilot, do not silently fix.** "Found drift: [what] in [file]. Fix?" Fix only if instructed.
## Working Directory
`{{HOME_DIR}}/IWE/`
## Status Reporting
Start: `agent_status_update(agent=claude-code, status=working, task=..., files=[...])`. Done: `status=idle`. Team repo: add `repo="org/repo-name"`. Fail-safe: Stop-хук → `scripts/agent-status-report.sh`.
## WP-REGISTRY Naming — CRITICAL
**Колонка «Название» = ТОЛЬКО имя артефакта ≤80 символов.** Запрещено: даты, SHA, метрики, статусы фаз, ссылки. Итог → `archive/wp-contexts/WP-NNN.md §Закрытие`. Статус фаз → frontmatter `inbox/WP-NNN.md`.
## WP Context Scope — Umbrella РП
`umbrella: true` + `agent_scope: open-only` → читать только `pending/in_progress/blocked`. Архивные — не читать без запроса. Применяется к: WP-5, WP-7.
## Calendar Events — CRITICAL
**Все события агента — ДО 09:00.** Создано после 09:00 → удалить + пересоздать + сообщить пилоту.
## Language
Respond in Russian unless the user writes in English.
## Response Style — Pilot-Facing
Применять A1-A11 (`memory/feedback_response_clarity_for_pilot.md`). Channel: технический (commit/PR + пилот пишет `grep`/`git`/SHA) vs «на пальцах» (остальной чат).
## Code Style — Engineering (DP.SC.172)
→ `engineering-code-style-base.md` (PACK-digital-platform). P0 форматтер+линтер; P1 тест без assert запрещён; P2 повторение×3 → функция; P3 мёртвую ветку удалять; P4 `except: pass` без лога запрещён.

<!-- SYNC-CORE-END -->

---

## 8. Staging (обкатка → шаблон)

> Правила на обкатке. Работают → переносятся в шаблон (L1).
> **Перенесено в L1 (20 мар):** SC Gate, межсистемные процессы, чеклист-верификация.
> **Промотировано в FMT (20 апр):** S-13 (именование РП = существительное-артефакт), S-14 (синхронизация REGISTRY→производные).

### Staging-канал (my IWE → FMT-exocortex-template)

- **S-45 Agent Inbox** (WP-324, 17 мая, расширено session 6) — `inbox/agent/` структура + 5 templates + SPEC + DP.SC.135 + DP.ROLE.045 + `iwe-agent-dispatcher.py` (headless `claude -p`, обход CCR v1→v2 bug). Промотировано в FMT `extensions/agent-inbox/` + `pack-templates/digital-platform/`. Status: testing (полная end-to-end automation smoke на расписании — defer-ред: требует Nix systemd unit или cron).

**Правило добавления:** новое поведение в §9 (авторское) → ОДНОВРЕМЕННО строка в STAGING.md (`status: testing`).

**Промоция (при Week Close):**
1. Просмотреть STAGING.md → есть `validated`?
2. Убрать авторские константы → заменить на `{{PLACEHOLDER}}`
3. Перенести в `FMT-exocortex-template` + commit `feat: promote S-NN from staging`
4. Обновить STAGING.md: статус → `promoted`

**Отклонение:** специфичное для авторского окружения → статус `rejected` (остаётся навсегда в §9, не промотируется). Не удалять из таблицы — это решение.

---
## 9. Авторское (только мой IWE)

### Блокирующие (авторские)

- **Pull-before-Commit:** перенесён в §2 п.5 (платформенное правило для ВСЕХ репо).
- **Без Obsidian (DS-strategy):** Просмотр через VS Code.
- **S-33 (Hooks/Scripts Bypass Gate):** перенесён в §2 п.6 (платформенное правило). Авторское дополнение: путь `FMT-exocortex-template/` включён в §2 п.6.

### Различения (авторские)

> Хранятся в `.claude/rules/distinctions.md` в зоне AUTHOR-ONLY — не затираются при `update.sh`.


### Именование

- `DS-strategy` (не `DS-strategy`) — личный governance-хаб
- `{{HOME_DIR}}/IWE/` — рабочая директория

### Read-only репо


### Extensions Gate (БЛОКИРУЮЩЕЕ)

**Для пользователей:** кастомизация протоколов/скиллов → ТОЛЬКО в `extensions/*.md`.
Прямое редактирование `.claude/skills/` или `memory/protocol-*.md` = ошибка.
**Архитектурное обоснование:** платформенные файлы (L1) и пользовательские расширения (L3) -- разные слои. Смешение слоёв = хрупкость при обновлении. Разделение: платформенное → `FMT-exocortex-template` → `update.sh`. Пользовательское → `extensions/` + `params.yaml`.

**Для автора шаблона (`params.yaml → author_mode: true`):** прямое редактирование L1 файлов РАЗРЕШЕНО.
- **Flow (единый для всего L1):** авторский IWE (source-of-truth) → доставка в FMT (с отрезанием личного) → GitHub → `update.sh` → пользователи. Авторский IWE = SoT для ВСЕГО: CLAUDE.md, скриптов, хуков, скиллов.
- **CLAUDE.md:** авторский IWE → `bash $IWE_TEMPLATE/scripts/template-sync.sh` → автоматически в FMT (плейсхолдеры + отрезание §9). Режимы: без флагов = sync, `--dry-run` = diff, `--check` = drift (exit 1).
- **Промоция артефактов в шаблон** — единая команда по типу:
  - Скрипт: `bash $IWE_SCRIPTS/script-promote.sh <личный-скрипт>.sh [--dry-run]` → FMT/scripts/
  - Хук: `bash $IWE_SCRIPTS/hook-promote.sh <личный-хук>.sh [--dry-run]` → FMT/.claude/hooks/
  - Скилл: `bash $IWE_SCRIPTS/skill-promote.sh <папка-скилла>/ [--dry-run]` → FMT/.claude/skills/
  - CLAUDE.md: `bash $IWE_SCRIPTS/template-sync.sh` (автозамена §9 + плейсхолдеры)
- **Все promote-скрипты:** применяют одинаковые подстановки (личные пути и repo-имя → env vars) → прогоняют `validate-fmt-scripts.sh` → копируют. Флаг `--dry-run` показывает результат без копирования.
- **Валидатор** запускается автоматически после каждого `template-sync.sh`. Вручную: `bash $IWE_SCRIPTS/validate-fmt-scripts.sh $IWE_SCRIPTS/`.


### README.md (FMT-exocortex-template)

> Изменение структуры — по согласованию с владельцем.

### Именование РП

**Название РП = существительное-артефакт**, а не глагол-действие.
- ✅ «Дизайн системы стратегирования», «Архитектура MCP», «Концепция подписок»
- ❌ «Разработать систему», «Настроить MCP», «Сделать концепцию»

**Название в WP-REGISTRY.md = ≤80 символов, только русский.** Допустимо вкрапление кодовых идентификаторов и Pack-ID, если они являются собственным именем артефакта (`projection-worker`, `DP.SC.125`, `cut-over`, `IWE`). Реестр — индекс, не карточка РП.

Запрещено в названии:
- статус, даты, commit hash, фазы, метрики («closed 27 апр», «Ф1 done», «PASS», «1.5h факт vs 3h»)
- список под-задач через `+`/`;` или parenthetical-нарратив
- английские пояснения (`spawn`, `closes drift`, `runtime activation`)
- ссылки на другие РП («child WP-268», «parent zonтик», «source: feedback_*»)

Контекст РП (фазы, handoff, ArchGate, бюджеты, решения) живёт в `DS-strategy/inbox/WP-NNN-*.md` для активных и `DS-strategy/archive/wp-contexts/` для закрытых. В реестре — только имя артефакта.

Эталоны: WP-254 «Миграция учебных объектов #6 aist-bot → #11 learning», WP-258 «Plugin API L2 для IWE (регистр MCP-расширений + контракт плагина)», WP-264 «Day Open enforcement — diagnostic logging + deterministic scaffold».

**Синхронизация REGISTRY→производные:** при переименовании РП → обновить одновременно REGISTRY.md + MEMORY.md + WeekPlan + DayPlan (если активен) + WP-context file.

### Память (Memory Lifecycle) — S-35

> Spec: `memory/memory-lifecycle-spec.md` (WP-217 Ф10.1, ArchGate 2026-04-30).

**Обязательный frontmatter** для всех новых файлов `memory/*.md`:

```yaml
---
name: "..."
description: "одна строка для MEMORY.md"
type: user | feedback | project | reference | lesson | protocol
horizon: hot | warm | cold | archive
domains: [тег1, тег2]
status: active | dormant | superseded | archived
valid_from: YYYY-MM-DD
owner: user | platform
schema_version: 1
---
```

**Правила горизонта:**
- `hot` — загружается каждую сессию. Суммарный лимит: ≤150 строк по всем HOT-файлам (без frontmatter).
- `warm` — по триггеру. Default для `project`, `reference`, `lesson`, `protocol`.
- HOT-лимит превышен → предложить понизить один из HOT-файлов в WARM перед добавлением нового.

**Архивация:** предлагает агент при Week/Month Close — не выполняет автономно. HOT→WARM: 14 дней без обращения. WARM→COLD: 30 дней. COLD→archive: 90 дней.

### Security Audit Cadence (WP-212, S-36)

> **Три уровня периодичности:**
> - **Per-ArchGate (каждое архитектурное решение):** §Б чеклист в ArchGate (B7.1 ✅) + STRIDE для нового сервиса → обновить B7.2 scope-таблицу.
> - **Week Close (2 мин):** проверить `security-posture.md §3` — `open_critical_count > 0`? Если да → добавить WP-212 в следующий WeekPlan.
> - **Daily (tsekh-1, 04:45 МСК):** systemd-timer `iwe-overnight-auditor` → VR.R.002 daily-headless по B7.4 A-D (~10-15 мин, $1.5). Critical flags → Day Open «Требует внимания».
> - **Month Close (VR.R.002 Аудитор monthly-deep, ~1h):** sub-agent VR.R.002 (catalog R24, context isolation, Sonnet) → разделы A-F чеклиста B7.4 → обновить `security-posture.md` → коммит `docs(WP-212): security audit YYYY-MM`.

**Файлы:**

### WeekPlan / WeekReport split (ОПТ-5, WP-297)

**WeekPlan** = намерения недели (planning, inbox triage, НЭП, приоритеты, контент-план). **Только интенты.** Нет прошлого.
**WeekReport** = факты недели (итоги дней, что сделано, коммиты). **Только история.** Нет планов.

**Правило:** при создании WeekPlan — предыдущие «Итоги дня» и «Итоги прошлой недели» → новый `WeekReport W{N} YYYY-MM-DD.md`. WeekPlan держит только `week_report: WeekReport W{N}...` в frontmatter как ссылку.
**Мотив:** WeekPlan смешивал факты и намерения → 545 строк → непригоден как planning-документ. Split → WeekPlan ≤200 строк.

### Режим «на пальцах» (S-37)

**Явные триггеры:** «объясни», «на пальцах», «что сделали», «расскажи понятно», «простыми словами» — любая просьба объяснить итоги или суть работы.

**Детектор канала (peer-session 2026-06-01-27, добавлено):**
- **Технический режим** в чате с пилотом — если в сообщении пилота сами встречаются: `grep`, `git`, имена файлов с путями, флаги команд, SHA, английские термины из кода.
- **Режим «на пальцах»** по умолчанию для всего остального: ответы на «что произошло», «почему не работает», «объясни», или задание без технических деталей в формулировке.
- **report.md** при синтезе peer-сессии: §1-§4 (Постановка, Позиции, Альтернативы, Решение) — режим «на пальцах»; прямые цитаты из ходов внутри §2 — плотный технический стиль (доказательство, не синтез).
- **Стенограммы ходов** (NN-writer / NN-peer) — плотный технический стиль, правила режима НЕ применяются.
- **Commit messages, PR descriptions** — плотный технический стиль.

**Правила ответа в режиме «на пальцах» (полная нумерация A1-A11 — в `feedback_response_clarity_for_pilot.md`):**
- Только русский язык. Английские термины допустимы в скобках после русского описания, исключение — термины, которые сам пилот употребляет (бот, чек-ин, deploy, smoke, merge, push, commit, MCP, Pack). **(A2)**
- Путь к файлу — никогда не подлежащее. Только в скобках после русского глагола («бот пишет ноль в счётчик (`handlers/marathon.py:65`)»). **(A1)**
- Никаких голых commit-хешей, номеров строк, флагов команд в основном тексте. Под спойлер «технические детали» в конце. **(A7)**
- При первом упоминании имени колонки/функции/переменной — расшифровка одним словом. **(A3)**
- Никаких английских маркеров успеха/неудачи как факта: «exit 0», «PASS», «SHA: abc» → «получилось», «прошло проверку», «залил правкой». **(A10)**
- Никакого пассивного залога при ошибке или находке: «было обнаружено» → «я нашёл», «я ошибся». **(A11)**
- Объяснять через аналогии из физического мира или инженерии: что делает система, зачем, что было сломано, что починили. **(A5)**
- Допустимо: цифры, проценты, сравнения «было / стало», схемы словами.
- Формат: свободный текст или маркированный список.

**Пример нарушения:** «Исправлен `activity_domain` фильтр в `load_rcs_metrics` — заменён на `SELF_DEV_EVENT_TYPES`.»
**Правильно:** «Раньше система считала рабочие дни вместе с учебными — теперь считает только те дни, человек реально занимался саморазвитием.»

**Полные правила, 12 паттернов и примеры «было / стало»:** [feedback_response_clarity_for_pilot.md](memory/feedback_response_clarity_for_pilot.md).

### Стиль общения с участниками сообщества (S-38)

**Аудитория:** участники сообщества (не пилот). Читают как люди, не как разработчики.

Стиль общения - по `communication-style-base.md`.
Базовые правила inline ниже (синхронизируются скриптом `scripts/sync-communication-style.py`).
Дополнительные правила этого канала - ниже.

<!-- COMMUNICATION-STYLE-BASE-START -->

> Staging-канал (обкатка → FMT), Extensions Gate, авторские правила L3.

*Последнее обновление: 2026-06-26*

# Перевірка (Task A–C, бонус E3)

> Скопійовано в `docs/verification.md`. Сюди — лише те, що справді сталося: цитати,
> числа, імена файлів, SHA комітів. Порядок дій — у `docs/walkthrough.md`.
> Прогони A/B і фіча «запит на кошторис» — в окремому звіті `docs/ab-validation.md` (Task D).

- **Інструмент і версія, модель:** Cursor · Claude (Claude Code CLI під капотом)
- **ОС і термінал, Node:** Windows 11 · Git Bash · Node (перевірити версію: `node --version`)

## Скіли видно у свіжій сесії

- Як перевіряли: нова сесія в Cursor, запит «які skills тобі доступні? не відкривай файлів»

| Skill | Звідки (Project / Personal / вбудований) | Примітка |
|---|---|---|
| `vercel-react-best-practices` | Project (`.claude/skills/`) | видно одразу після встановлення, коміт `4a238ee` |
| `building-client-form` | — | ще не створено (Task B) |
| `integrating-n8n-webhooks` | — | ще не створено (Task C) |

- Особисті скіли, які теж видно (`~/.claude/skills/`…), і чи можуть вони вплинути на перевірки: окремо `~/.claude/skills/` не перевірявся; список сесії показав лише вбудовані системні скіли Cursor/Claude Code (`dataviz`, `artifact-design`, `artifact-diagramming`, `artifact-capabilities`, `update-config`, `keybindings-help`, `code-review`, `simplify`, `fewer-permission-prompts`, `loop`, `schedule`, `claude-api`, `workflow-authoring`, `run`, `init`, `security-review`, `skill-creator`, `docs`, `docx`, `pdf`, `pptx`, `xlsx`, `google-workspace`, `import-memory`, `morning`) — вони не є проєктними й не мали б впливати на перевірки Task B/C/D.

## Task A — виправлення за скілом Vercel

Зроблено **3 виправлення** (мінімум 2, +1 бонусне для безпеки).

**Як міряли (для виправлень з числами):** продакшн-збірка (`npm run build && npm start`), cookie `leaddesk_session=demo-u_olena`, URL `http://localhost:3000/dashboard`. Один прогрів + 3 виміри `curl -w "TTFB %{time_starttransfer}s, total %{time_total}s\n"`. Для `server-cache-react` додатково — лічильники `db:<запит>` у консольному логі сервера (скільки разів виконався кожен запит на один рендер сторінки).

| Правило (id) | Коміт | Файли | Що змінилось | Було (`main`) | Стало | Як міряли |
|---|---|---|---|---|---|---|
| `async-parallel` | `f1f92e7` | `app/dashboard/page.tsx` | `getLeads`, `getLeadStats`, `getSourceBreakdown` (400+1200+400 мс) виконувались послідовно → замінено на `Promise.all()` | TTFB ~2.32s (сер. з 3 прогонів: 2.328/2.325/2.304) | TTFB ~1.50s (сер.: 1.486/1.533/1.478) | curl TTFB, 3 прогони до/після на продакшн-збірці |
| `server-cache-react` | `cfd5f1e` | `lib/data.ts`, `app/dashboard/page.tsx`, `app/dashboard/layout.tsx`, `app/dashboard/leads/[id]/page.tsx`, `components/dashboard-header.tsx` | `getCurrentUser` не був обгорнутий у `cache()`; `getWorkspace` приймав об'єкт `{slug}`, через що `cache()` по `Object.is` ніколи не збігався → `getCurrentUser` обгорнуто в `cache()`, `getWorkspace` переведено на примітив `slug: string` | `db:getUserBySession`: 3× і `db:getWorkspace`: 3× за один рендер `/dashboard` | `db:getUserBySession`: 1× і `db:getWorkspace`: 1× за один рендер | лічильники в консольному логі `npm start` після одного `curl` на `/dashboard` |
| `server-auth-actions` | `74729da` | `app/actions.ts` | `updateLeadStatus`/`deleteLead` не перевіряли сесію й належність ліда воркспейсу — додано `assertLeadInUserWorkspace()`: `getCurrentUser()` + звірка `lead.workspaceId` з воркспейсом користувача; типізований результат `{status: "ok"\|"not_found"\|"forbidden"}` замість сирих помилок БД | (без заміру — правило про безпеку, не про швидкість) | (без заміру) | — |

- Чому обрали для заміру саме `async-parallel` і `server-cache-react`: ефект найпростіше й найоднозначніше виміряти числами (TTFB і лічильники запитів до "бази"), обидва напряму пояснювали початкову скаргу клієнта («дашборд відкривається понад 2 секунди») і знахідку з базової лінії (3× дублювання `getUserBySession`/`getWorkspace`).
- `server-auth-actions` — застосовано без заміру: це виправлення безпеки (CRITICAL за пріоритетом скіла), а не продуктивності; коректність перевірено через `npm run build` (типізація `LeadMutationState`) і те, що мутація тепер виконується лише після успішної перевірки прав.
- Порада скіла, яку звірили з документацією Next.js 16 і не застосували чи змінили: не було — усі три застосовані поради (`server-cache-react`, `async-parallel`, `server-auth-actions`) підтвердились як чинні для Next.js 16.3.5 (див. `docs/skill-review.md`, пункт 5), змінювати їх не довелось.
- Виміряні виправлення дали чіткий, відтворюваний ефект (TTFB −35%, лічильники БД 3×→1×) — падінь у "нуль ефекту" не було.
- `npm run lint`, `npm run build` після кожного з трьох виправлень: обидві команди проходили чисто (0 попереджень lint, успішна збірка) після кожного коміту.

## Task B — `building-client-form`

*(ще не виконано)*

- Запит у свіжій сесії (скіл не названо):
  > <запит>
- Чи спрацював скіл і як це видно: <виклик `Skill` з `building-client-form` / читання `SKILL.md` / ні>
- Якщо не з першого разу — що змінили в `description`, і результат другої спроби: <…>
- Що зроблено (файли): <…>
- Пункти Verify зі скіла — результат кожного: <…>

## Task C — `integrating-n8n-webhooks`

*(ще не виконано)*

Тут скіл лише пакують. Застосовує його агент у прогоні **B** (Task D) — доказ спрацювання, журнал
мока й час відповіді форми — у `docs/ab-validation.md`.

- Що лишили в `SKILL.md`, а що винесли в `references/` (і чому): <…>
- Правила зупинки — перелік: <…>
- SHA коміту зі скілом (BASE для Task D): <…>
- Що скіл змінив у собі після прогонів (коміти й чому): <… або «нічого»>

**`check-contract.mjs` на коді `main`** (id + PASS/FAIL, код виходу):
# Agentic Software Factory — MVP Blueprint

**Версія:** 0.1  
**Призначення:** робоча специфікація для реалізації першого повторно використовуваного фрагмента software factory та подальшого ітеративного розвитку.  
**Основний принцип:** йти від малого до великого, перевіряючи кожну нову ідею на реальних задачах і A/B-експериментах.

---

## 1. Що ми будуємо

Кінцева мета — побудувати **ефективну software factory для виконання реальних задач розробки**, яка:

- може використовувати різні agentic coding harness-и та підписки: Codex, Antigravity/Gemini, надалі Cursor та інші;
- дозволяє призначати різних виконавців і моделі на різні етапи;
- використовує детерміновану верифікацію замість довіри до фрази агента “готово”;
- повторно використовує project-level skills;
- збирає метрики часу, токенів/usage (коли вони доступні), ретраїв, тестів та результатів;
- збирає суб’єктивний фідбек користувача: чи результат справді відповідає потребі;
- дозволяє проводити контрольовані A/B-експерименти;
- поступово розширюється новими вузлами: planning, UX, architecture, review, QA Green, user validation тощо.

**MVP не є повною фабрикою.** Перший продукт — маленька “закрита коробка” для verified development.

---

## 2. MVP: дві agent-ноди

Перший LangGraph subgraph містить лише дві **agent-ноди**:

1. **QA Red / Verifier** — перед реалізацією формує executable verification contract і, де доречно, реальні verification tests.
2. **Developer** — реалізує задачу, не маючи права змінювати verification artifacts, і працює до deterministic pass або вичерпання ліміту спроб.

Детерміновані gates, scope checks, artifact persistence та observability — це інфраструктурні вузли/операції, а не окремі LLM-агенти.

```text
Outer Orchestrator -> MCP: start_verified_dev(...)
        |
        v
QA Red / Verifier -> QA scope gate -> Developer
                                      |
                                      v
                           deterministic verification
                              FAIL -> Developer
                              PASS -> result -> Outer flow / QA Green
```

**QA Green навмисно залишається зовні.** Субграф не повинен сам себе остаточно сертифікувати.

---

## 3. Як це використовується

### 3.1. Antigravity як зовнішній оркестратор

Користувач працює у звичному Antigravity UI:

1. UX/Architecture виконуються зовні фабрики.
2. Коли задача готова до реалізації, Antigravity викликає MCP tool `factory.start_verified_dev`.
3. Фабрика запускає QA Red → Developer → deterministic verification.
4. Antigravity отримує `run_id`, прогрес, а потім структурований результат.
5. Після успіху зовнішній оркестратор може запустити окремий QA Green або продовжити свій workflow.

### 3.2. Codex як зовнішній оркестратор

Той самий контракт. Codex не повинен знати внутрішню топологію графа. Для нього фабрика — повторно використовуваний tool.

### 3.3. Локальний запуск

Розробник фабрики повинен мати CLI для локального дебагу та тестування:

```bash
factory run --task <task-file> --repo <repo-path> --config <config-file>
```

LangGraph Studio може лишатися **опційним developer-debug tool**, але не є основним UX і не є dependency для роботи фабрики.

---

## 4. Що НЕ будуємо зараз

Щоб не повторити помилку “будуємо всю фабрику одразу”, MVP свідомо не включає:

- власний чат або custom dashboard;
- planning/UX/architecture nodes;
- QA Green всередині subgraph;
- автоматичне редагування skills агентами під час runtime;
- складну multi-agent hierarchy;
- власну систему пам’яті;
- залежність від LangSmith;
- вимогу користуватися LangGraph Studio;
- єдиний “магічний” quality score;
- повну уніфікацію всіх можливих coding agents.

Зовнішній інтерфейс користувача — Codex / Antigravity / інший orchestrator, який підтримує MCP або еквівалентний tool call.

---

## 5. Шари системи

### 5.1. Outer Interactive Orchestrator

Codex, Antigravity, надалі Cursor або інший agentic client.

Відповідає за:

- взаємодію з користувачем;
- UX/architecture/planning навколо фабрики;
- рішення, коли викликати verified-development subgraph;
- зовнішній QA Green;
- показ результату користувачу.

### 5.2. Factory MCP Service

Окремий локальний проєкт/сервіс.

Відповідає за:

- стабільний tool contract;
- запуск і контроль довгих runs;
- status/result/cancel;
- A/B experiments;
- user feedback;
- незалежність від конкретного зовнішнього UI.

### 5.3. LangGraph Runtime

Відповідає за:

- state machine;
- retries;
- routing;
- persistence/checkpoints;
- boundaries між QA Red і Developer;
- structured failure loop.

### 5.4. Worker Adapters

Provider/harness-specific layer:

- Codex CLI adapter;
- Antigravity/Gemini adapter;
- Cursor adapter — пізніше, якщо буде потрібен.

LangGraph не повинен містити provider-specific CLI syntax.

### 5.5. Project Skills

Skills належать target-проєкту та можуть окремо розвиватися.

Нода не отримує всю бібліотеку skills у prompt. Вона:

1. отримує короткий index/metadata доступних skills;
2. вибирає релевантні;
3. завантажує лише вибрані;
4. логує вибір;
5. працює.

### 5.6. Langfuse

Langfuse — observability та experiments layer, а не control plane.

Він повинен показувати:

- trace одного factory run;
- spans по нодах, worker calls, gates, retries;
- duration;
- token/usage data, якщо worker реально їх надає;
- model/runtime/prompt/skill versions;
- test outcomes;
- кількість repair attempts;
- artifacts;
- user feedback;
- A/B experiment metadata.

**Не вигадувати token counts**, якщо subscription CLI їх не віддає. У такому випадку значення = `null/unavailable`, а основною efficiency metric лишається wall-clock + retries + pass rate.

---

## 6. Запропонована структура окремого проєкту

```text
agentic-factory/
├── factory/
│   ├── graph.py
│   ├── state.py
│   ├── nodes/
│   │   ├── verifier.py
│   │   └── developer.py
│   ├── gates/
│   │   ├── scope.py
│   │   ├── verification.py
│   │   └── checks.py
│   ├── workers/
│   │   ├── base.py
│   │   ├── codex.py
│   │   └── antigravity.py
│   ├── skills/
│   │   ├── discovery.py
│   │   └── loader.py
│   ├── observability/
│   │   └── langfuse.py
│   ├── experiments/
│   │   ├── ab.py
│   │   └── contracts.py
│   ├── artifacts/
│   └── runner.py
├── mcp/
│   └── server.py
├── tests/
├── docs/
├── .factory/
│   ├── runs/
│   └── checkpoints/
└── pyproject.toml
```

### Використати існуючий `lang-factory` як стартову базу

Поточний репозиторій уже має корисні частини, які варто еволюційно доробити, а не переписувати без причини:

- LangGraph graph/state;
- scope enforcement;
- deterministic verification gate;
- artifacts/failure reports;
- runner/CLI;
- Antigravity worker adapter;
- checkpoint persistence.

Перший рефакторинг повинен **підключити реальне worker execution до graph**, замінити stub checks на реальні gates, додати skills, MCP, Langfuse та experiment layer.

---

## 7. Run contract

### 7.1. Вхід

Приклад логічного контракту:

```json
{
  "task": {
    "id": "auth-017",
    "title": "Implement password reset",
    "description": "Path or structured task payload",
    "context_files": []
  },
  "target": {
    "repo_path": "/path/to/project",
    "base_ref": "HEAD"
  },
  "workers": {
    "verifier": {
      "runtime": "codex",
      "model": "MODEL_ID",
      "runtime_profile": "default",
      "prompt_version": "qa-red-v0.1"
    },
    "developer": {
      "runtime": "antigravity",
      "model": "MODEL_ID",
      "runtime_profile": "default",
      "prompt_version": "dev-v0.1"
    }
  },
  "skills": {
    "mode": "discover",
    "roots": [".agents/skills"],
    "max_selected": 4
  },
  "limits": {
    "max_dev_attempts": 3,
    "worker_timeout_seconds": 1800,
    "gate_timeout_seconds": 600
  },
  "observability": {
    "tags": ["verified-dev", "mvp"],
    "experiment_name": null
  }
}
```

`runtime_profile` — абстрактне ім’я конфігурації локального harness/auth. Фабрика не повинна зберігати subscription credentials.

### 7.2. Вихід

```json
{
  "run_id": "run_...",
  "status": "passed",
  "base_commit": "...",
  "final_commit": "...",
  "attempts": 2,
  "verification": {
    "passed": true,
    "commands": [],
    "failed_checks": []
  },
  "changed_files": [],
  "artifacts": {
    "verification_contract": "...",
    "verification_tests": [],
    "diff": "...",
    "logs": []
  },
  "metrics": {
    "duration_seconds": 0,
    "verifier_duration_seconds": 0,
    "developer_duration_seconds": 0,
    "tokens": null
  }
}
```

---

## 8. Ізоляція run

Кожний production run та кожна сторона A/B experiment повинні працювати в **чистому ізольованому workspace**, бажано Git worktree.

Вимоги:

- однаковий `base_commit`;
- відсутність незакомічених змін від попереднього run;
- QA Red і Developer бачать один і той самий run workspace;
- A і B не бачать зміни один одного;
- усі артефакти прив’язані до `run_id`;
- після run можна зберегти worktree для inspection або очистити за policy.

Це критично для валідного A/B та відтворюваності.

---

## 9. QA Red / Verifier: поведінка

QA Red запускається **до Developer**.

Його задача — перетворити вимогу на конкретну, виконувану відповідь на питання:

> “Як ми об’єктивно дізнаємося, що реалізація правильна?”

### QA Red повинен:

- прочитати task + relevant project context;
- виконати skill discovery;
- сформувати короткий verification contract;
- для поведінки, яку реально можна автоматично перевірити, створити executable verification tests;
- визначити команди для запуску;
- мінімізувати mocks, якщо вони роблять перевірку відірваною від реальної поведінки;
- покрити acceptance criteria;
- явно позначити речі, які не можна автоматично перевірити;
- не змінювати application source code.

### Мінімальний verification contract

```yaml
task_id: auth-017
acceptance_checks:
  - id: AC1
    requirement: "..."
    verification: "..."
    command: "..."
    expected: "..."
manual_checks: []
environment_assumptions: []
```

Verifier scope gate повинен технічно забороняти QA Red змінювати application code.

---

## 10. Developer: поведінка

Developer отримує:

- task;
- verification contract;
- verification tests;
- structured failure report попередньої спроби, якщо це retry;
- selected project skills.

Developer:

- реалізує task;
- може додавати developer unit/integration tests;
- запускає локальні checks;
- не має права змінювати verification artifacts;
- після роботи передає контроль deterministic gate;
- на FAIL отримує machine-readable failure report і робить repair;
- не може сам оголосити себе “успішним” без gate PASS.

Dev scope gate технічно забороняє зміну verification area.

---

## 11. Skill discovery protocol

Skills — одна з експериментальних змінних системи, тому їх використання має бути контрольованим і спостережуваним.

### Protocol

1. Factory знаходить configured `skill_roots`.
2. Читає лише metadata/index, а не всі повні skill files.
3. Agent обирає максимум `N` релевантних skills.
4. Factory завантажує повний текст лише вибраних skills.
5. До Langfuse логуються:
   - skill ID/path;
   - version/hash;
   - node;
   - коротка причина вибору.
6. Runtime workers **не редагують skills**.
7. Якщо релевантних skills немає — робота продовжується без них.

### Важливо

Prompt повинен описувати **роль і boundaries**, а reusable domain knowledge по можливості має жити у skills. Це дозволяє окремо покращувати prompt, skills і orchestration та вимірювати вплив кожного.

---

## 12. Worker abstraction

Внутрішній контракт worker-а повинен бути provider-neutral.

Концептуально:

```python
run(
    prompt,
    cwd,
    model,
    runtime_profile,
    session_id=None,
    event_sink=None
) -> WorkerResult
```

`WorkerResult` повинен містити щонайменше:

- status;
- response/summary;
- session ID, якщо підтримується;
- raw log artifact;
- structured events;
- usage, якщо доступний;
- exit/error data.

### Streaming

Бажано мати normalized events:

```text
worker.started
worker.message
tool.started
tool.finished
file.changed
command.started
command.finished
worker.completed
worker.failed
gate.started
gate.finished
```

MCP client може показувати прогрес, якщо підтримує progress/log notifications.

**Fallback обов’язковий:** навіть якщо конкретний client не показує streaming, `get_run_status(run_id)` повинен давати актуальний прогрес.

---

## 13. MCP API

Мінімальний MCP surface:

### `factory.start_verified_dev`

Запускає нормальний factory run і швидко повертає `run_id`.

### `factory.get_run_status`

Повертає:

- status;
- current stage;
- attempt;
- останню meaningful progress event;
- elapsed time.

### `factory.get_run_result`

Повертає фінальний structured result та artifact references.

### `factory.cancel_run`

Зупиняє run контрольовано.

### `factory.start_ab_experiment`

Запускає контрольований A/B experiment.

### `factory.get_experiment_status`

Прогрес по A/B.

### `factory.get_experiment_result`

Порівняння A і B без автоматичного оголошення “переможця”, якщо даних недостатньо.

### `factory.submit_user_feedback`

Записує людський feedback після перегляду результату.

---

## 14. A/B testing contract

A/B experiment потрібен у першій архітектурі, але його слід реалізувати після стабільного single-run path.

### Основне правило

**A і B повинні відрізнятися рівно одним experimental factor.**

Це не означає “один символ”. Це означає одну причинну гіпотезу, наприклад:

- `skill_discovery: off -> on`;
- `dev_prompt_version: v1 -> v2`;
- `verifier_model: X -> Y`;
- `max_selected_skills: 2 -> 4`;
- одна версія конкретного skill -> інша.

Усе інше заморожується.

### Приклад

```json
{
  "task": "auth-017",
  "baseline_config": "configs/default.json",
  "difference": {
    "factor": "developer.prompt_version",
    "a": "dev-v0.1",
    "b": "dev-v0.2"
  },
  "repetitions": 3
}
```

Factory повинна відмовитися від experiment, якщо виявлено інші відмінності між A і B.

### Контроль

Для кожної пари:

- однаковий task payload;
- однаковий base commit;
- окремі чисті worktrees;
- однакові timeouts/limits;
- однакові інші worker configs;
- один і той самий external evaluation protocol.

Через stochastic behavior бажано запускати 2–3 repetitions, коли вартість/час це дозволяють.

---

## 15. Метрики

Не намагатися одразу звести все до одного score.

### Якість

- deterministic verification PASS/FAIL;
- external QA Green PASS/FAIL;
- acceptance criteria coverage;
- кількість defects після run;
- чи потрібен manual rework.

### Ефективність

- total wall-clock duration;
- duration per node;
- worker execution duration;
- deterministic gate duration;
- number of retries;
- token/input-output usage, **тільки якщо достовірно доступний**;
- cost, якщо реально доступний;
- кількість tool/command calls як secondary metric.

### Суб’єктивний фідбек користувача

Після run:

```yaml
met_intent: yes | partial | no
rating: 1..5
rework_minutes: 0
notes: ""
```

`met_intent` і `rework_minutes` важливіші за красиві benchmark numbers, бо кінцева мета — корисний software результат.

### Стабільність

Для repeated runs:

- pass-rate variance;
- duration variance;
- retry variance;
- user feedback variance.

---

## 16. Langfuse trace model

Один trace = один factory run.

Рекомендовані spans:

```text
factory.run
├── workspace.prepare
├── verifier
│   ├── skills.discover
│   └── worker.execute
├── verifier.scope_gate
├── developer.attempt.1
│   ├── skills.discover
│   └── worker.execute
├── verification.gate.1
├── developer.attempt.2
├── verification.gate.2
└── run.finalize
```

Metadata на trace/span:

- run ID;
- experiment ID;
- task ID;
- base commit;
- factory version/commit;
- node;
- runtime;
- model;
- prompt version;
- selected skill IDs + hashes;
- attempt;
- pass/fail;
- duration;
- usage if available.

---

## 17. Factory development workspace

Окремо від runtime workers потрібні агенти, які допомагають **будувати і покращувати саму factory**.

На старті достатньо трьох ролей:

1. **Factory Engineer**
2. **Experiment Operator**
3. **Factory Advisor**

Ці ролі можуть жити у Codex/Antigravity project workspace як prompts/skills. Їх не потрібно робити LangGraph nodes MVP.

---

# 18. Prompt — Factory Engineer

```text
ROLE: Factory Engineer

You are the primary engineer for an experimental agentic software factory.

MISSION
Build and evolve a small, testable, provider-neutral software-factory runtime whose first vertical slice is:
QA Red / Verifier -> Developer -> deterministic verification loop.

The factory must be callable from external orchestrators such as Codex or Antigravity through a stable MCP interface. The outer orchestrator remains the user's main interactive UI.

CORE PRINCIPLES
- Start small. Do not implement the full future factory.
- Deterministic gates outrank LLM self-assessment.
- Keep provider-specific CLI behavior behind worker adapters.
- QA Red cannot modify implementation code.
- Developer cannot modify verification artifacts.
- Every run must be reproducible enough to compare.
- Prefer isolated Git worktrees.
- Skills are discovered from the target project and loaded selectively.
- Instrument meaningful events in Langfuse.
- Never fabricate token/usage information if a runtime does not expose it.
- LangSmith and LangGraph Studio must not be required for normal operation.
- No custom dashboard in MVP.
- Preserve working parts of the existing lang-factory repository instead of rewriting without evidence.

WORKING STYLE
Before a meaningful implementation batch:
1. Inspect the current repository and existing contracts.
2. State the smallest change that advances the MVP.
3. Define deterministic acceptance checks for that change.
4. Implement it.
5. Run the checks.
6. Summarize what changed, what is proven, and what remains uncertain.

ARCHITECTURE TARGET
- LangGraph: state machine/runtime.
- MCP: external tool boundary.
- Worker adapters: Codex, Antigravity first.
- Langfuse: traces and experiments.
- Project skills: external reusable knowledge.
- A/B layer: controlled single-factor experiments.

DO NOT
- add UX/Architecture/QA Green nodes yet;
- build a large web application;
- bury provider-specific logic inside graph nodes;
- weaken tests to make a run pass;
- add abstractions with no immediate consumer;
- treat agent text like "done" as proof.

OUTPUTS
For planning/review requests, respond with:
- Current state
- Proposed smallest change
- Files/contracts affected
- Validation plan
- Risks/open questions

For implementation requests, also provide the exact commands/tests used to validate the change.
```

---

# 19. Prompt — Experiment Operator

```text
ROLE: Factory Experiment Operator

You operate controlled experiments on the agentic software factory.

MISSION
Determine whether a specific factory change improves real software-task outcomes, efficiency, or stability.

PRIMARY DATA
- Langfuse traces
- factory run artifacts
- deterministic verification results
- external QA Green results when available
- user feedback
- repository/base commit
- exact run configuration

EXPERIMENT RULE
A/B must differ by exactly ONE experimental factor.
If more than one causal factor differs, do not run the experiment. Explain what must be normalized first.

PROCESS
1. State the hypothesis in one sentence.
2. Identify the single experimental factor.
3. Freeze all other relevant variables.
4. Ensure A and B start from the same task and base commit in isolated workspaces.
5. Choose repetitions appropriate to cost/time; default to 2-3 when feasible.
6. Run both conditions.
7. Compare dimensions separately:
   - quality/pass rate
   - user intent satisfaction
   - rework
   - wall-clock time
   - retries
   - token/usage only when trustworthy
   - stability across repetitions
8. Inspect traces for causal clues.
9. Report evidence, uncertainty, and the next experiment.

DO NOT
- manufacture a winner from one noisy run;
- combine multiple changes into a single A/B;
- optimize tokens while degrading task quality;
- trust self-reported agent success without gates;
- hide failed or timed-out runs.

OUTPUT
- Hypothesis
- A config
- B config
- Frozen variables
- Runs/repetitions
- Results by metric
- Trace observations
- User feedback
- Conclusion with confidence/uncertainty
- Recommended next single-factor experiment
```

---

# 20. Prompt — Factory Advisor

```text
ROLE: Factory Advisor

You are an advisor for improving an experimental agentic software factory.

MISSION
Analyze evidence from real runs and propose high-value, testable improvements to the factory.

INPUTS MAY INCLUDE
- recent conversations with the user about factory behavior;
- Langfuse traces and metrics;
- run artifacts and failure reports;
- A/B experiment results;
- user feedback;
- factory source code and configuration;
- project skills and prompt versions.

PRINCIPLES
- Optimize for reliable real-world software outcomes, not impressive agent narration.
- Prefer evidence from deterministic checks and external/user validation.
- Distinguish orchestration problems from prompt problems, skill problems, worker/model problems, and task-specification problems.
- Prefer the smallest intervention that can be tested.
- Every proposed improvement should be expressible as a falsifiable hypothesis.
- Prefer one-factor experiments.
- Do not rewrite the architecture because of one anecdotal run.
- Watch for over-mocking, test leakage, verification that does not resemble reality, unnecessary documentation, context bloat, and repeated work.

WHEN REVIEWING RUNS
Look for:
- where time was spent;
- where retries happened;
- unused or harmful context;
- which skills were selected;
- whether verification contract was meaningful;
- whether Developer overfit to verification tests;
- whether external QA/user feedback disagreed with internal PASS;
- recurring failure patterns across tasks.

OUTPUT
1. What appears to work
2. What appears to fail or waste resources
3. Evidence
4. Top 1-3 improvement hypotheses
5. For each: expected effect, risk, and one-factor A/B design
6. What NOT to change yet
```

---

# 21. Runtime prompt — QA Red / Verifier v0.1

Цей prompt навмисно простий. Його треба ітеративно покращувати через experiments.

```text
ROLE: QA Red / Verification Designer

Your job is to define how this task will be verified BEFORE implementation.

Read:
- the task;
- the relevant existing code;
- the available project-skill index.

Select and load only skills that are relevant to verification.

Produce a verification contract that maps each acceptance requirement to an objective check.
Where practical, create executable verification tests in the allowed verification directory.

Rules:
- Do not modify application implementation code.
- Do not weaken requirements to match existing behavior.
- Prefer behavior-level verification over mocks that remove the real behavior being tested.
- Make commands and expected outcomes explicit.
- Mark any requirement that cannot be reliably automated.
- Keep explanatory prose short; the contract and executable checks are the primary output.

Before finishing, run the verification tests if they are runnable against the current baseline and record the expected RED/baseline behavior.
```

---

# 22. Runtime prompt — Developer v0.1

```text
ROLE: Developer

Implement the task using the provided verification contract.

Read:
- the task;
- the verification contract/tests;
- the relevant existing code;
- the available project-skill index.

Select and load only skills relevant to implementation.

Rules:
- Never modify verification artifacts.
- Implement the smallest correct change that satisfies the task.
- Add developer tests when useful.
- Run the project's normal checks.
- Run the required deterministic verification commands.
- If a structured failure report is provided, repair the implementation based on that evidence.
- Do not claim success unless the deterministic gates pass.
- Do not replace real behavior with mocks merely to make verification pass.

Keep the final response concise: what changed, checks executed, and any unresolved limitation.
```

---

# 23. Skill for outer orchestrators — `verified-development`

Цей skill можна встановити/покласти в Codex/Antigravity project, щоб зовнішній orchestrator знав, **коли** і **як** викликати фабрику.

```text
SKILL: Verified Development Factory

Use this skill when a software task is sufficiently specified and ready for implementation, especially after UX/architecture decisions are already accepted.

The verified-development factory is an external MCP tool. Treat it as a reusable black box:
QA Red -> Developer -> deterministic verification/repair loop.

NORMAL FLOW
1. Prepare the task context and target repository.
2. Choose verifier and developer worker configs requested by the user/project.
3. Call factory.start_verified_dev.
4. Monitor progress via available MCP progress events or factory.get_run_status.
5. Retrieve the final result with factory.get_run_result.
6. Inspect the structured result and artifacts.
7. Continue the outer workflow. If the workflow includes QA Green, run it independently after factory success.

DO NOT
- bypass the factory's deterministic gates;
- edit its verification artifacts on behalf of Developer;
- treat factory PASS as a replacement for any explicitly requested independent QA Green;
- expose internal noisy logs to the user unless requested.

A/B FLOW
Only when an experiment is requested:
- identify exactly one experimental factor;
- keep everything else frozen;
- call factory.start_ab_experiment;
- report results by quality, time, retries, usage (if available), stability, and user feedback.
```

---

# 24. Bootstrap prompt — дати coding agent для реалізації цього документа

```text
You are the Factory Engineer responsible for implementing the MVP described in the attached "Agentic Software Factory — MVP Blueprint".

Start by inspecting the existing greyGroot/lang-factory codebase. Do not rewrite working infrastructure merely to match the document's suggested folder layout.

Your first objective is a working vertical slice, not the full roadmap.

Required MVP behavior:
1. QA Red / Verifier runs as a real worker and creates a verification contract/tests.
2. Scope enforcement prevents Verifier from modifying implementation code.
3. Developer runs as a real worker and cannot modify verification artifacts.
4. Deterministic verification executes after Developer.
5. On failure, a structured failure report loops back to Developer up to a strict attempt limit.
6. Runs use isolated workspaces/worktrees.
7. Worker runtime/model selection is configurable independently for Verifier and Developer.
8. Target-project skills are discovered selectively and the selected skill IDs/versions are recorded.
9. A stable MCP interface exposes asynchronous start/status/result/cancel.
10. Langfuse tracing records meaningful spans and metadata without requiring LangSmith.
11. After the normal single-run path is stable, add controlled A/B execution with exactly one experimental factor.
12. Add user-feedback capture.

Important:
- QA Green remains outside this graph.
- No custom dashboard.
- LangGraph Studio is optional debug tooling only.
- Never fabricate token usage.
- Use existing runner, scope, gates, artifacts, checkpointing, and worker code where sound.
- Add tests around contracts and invariants.
- Prefer clear simple code over framework-heavy abstraction.

BEFORE IMPLEMENTING:
A. Summarize the current repository state relevant to this MVP.
B. Identify reusable existing pieces and missing pieces.
C. Propose a staged implementation plan of no more than 5 stages.
D. For each stage define deterministic acceptance criteria.
E. Identify assumptions that require a quick spike, especially:
   - MCP progress behavior in the chosen external clients;
   - token/usage availability from subscription-backed workers;
   - CLI model/session selection for each worker runtime.

Then implement Stage 1 only unless explicitly asked to continue.
After implementation, run the acceptance checks and report evidence, not just a narrative status.
```

---

## 25. Definition of Done для MVP

MVP вважається готовим, коли:

- [ ] Окремий factory project запускається локально.
- [ ] Codex/Antigravity-compatible MCP client може запустити verified-development run.
- [ ] MCP повертає `run_id` без блокування на весь довгий run.
- [ ] Є status/result/cancel.
- [ ] QA Red реально запускається через worker adapter.
- [ ] QA Red створює verification contract та/або executable tests.
- [ ] QA Red технічно не може змінити application source.
- [ ] Developer реально запускається через незалежно configurable worker adapter/model.
- [ ] Developer технічно не може змінити verification artifacts.
- [ ] Deterministic verification PASS/FAIL визначає loop.
- [ ] Є strict max attempts.
- [ ] Run виконується в isolated workspace/worktree.
- [ ] Skills discovery працює selective/progressive-disclosure способом.
- [ ] Selected skills та versions/hashes записуються в trace.
- [ ] Langfuse показує trace з node durations, attempts, gates та worker metadata.
- [ ] Usage/tokens записуються лише коли достовірно надані runtime.
- [ ] User feedback можна прив’язати до run.
- [ ] A/B tool приймає одну experimental difference і відхиляє конфіг з додатковими змінами.
- [ ] A/B A та B стартують з одного base commit у різних worktrees.
- [ ] Є integration test/sample task, який демонструє повний flow.
- [ ] QA Green не захований всередині MVP subgraph і може бути запущений зовнішнім orchestrator-ом окремо.

---

## 26. Рекомендований порядок реалізації

### Stage 1 — Реальний two-node execution

Підключити worker execution, scope gates, verification loop та worktree isolation.

**Мета:** довести, що QA Red → Developer → deterministic gate реально працює.

### Stage 2 — MCP boundary

Додати async `start/status/result/cancel`.

**Мета:** запустити той самий subgraph з зовнішнього agentic client.

### Stage 3 — Skills + Langfuse

Selective skill discovery, tracing, versions/hashes, timings, retries.

**Мета:** зробити run спостережуваним і придатним до аналізу.

### Stage 4 — A/B experiments + feedback

Single-factor contract, repetitions, clean A/B workspaces, user feedback.

**Мета:** перетворити покращення фабрики з інтуїції на контрольований процес.

### Stage 5 — Наступна нода

Лише після реальних runs вирішити, що додавати наступним: QA Green, review, planner, architecture або UX.

Вибір наступної ноди повинен базуватися на даних про реальні bottlenecks, а не на бажанні зробити граф більшим.

---

## 27. Головний критерій успіху

Фабрика не повинна виглядати “agentic”.

Вона повинна **краще виконувати software tasks**.

Покращення вважається корисним, якщо на реальних задачах воно дає один або більше результатів без неприйнятного погіршення інших:

- вищий real-world pass rate;
- менше ручних виправлень;
- менше повторних спроб;
- менший час до корисного результату;
- краща відповідність наміру користувача;
- менше зайвого контексту/токенів;
- стабільніші результати.

Усе інше — лише засіб для цього.

# Voice Commander (Home Assistant)

## Що це таке
Зв’язка автоматизації `ai_voice_commander` + скрипта `ai_smart_executor` додає до голосового асистента можливість **розпізнавати команди з відкладеним виконанням** ("…через 5 хвилин", "…о 8:00") та **нагадування**, і виконувати їх у Home Assistant.

Ключова ідея: 
- **звичайні голосові команди** (без відкладення) обробляє стандартний агент Assist (через `conversation.process`),
- **тільки команди з часом / нагадування** — передаються в `ai_task.generate_data`, перетворюються в структуру, і виконуються через `ai_smart_executor`.

## Принцип роботи (покроково)
### 1) Вхід: voice → conversation trigger
Автоматизація `ai_voice_commander` запускається тригером `platform: conversation` і отримує текст з `trigger.user_input.text`.

### 2) AI Task: структуризація лише «time/reminder»
Дія `ai_task.generate_data` (через `entity_id: ai_task.openai_ai_task`) отримує інструкції:
- аналізувати лише команди/запити з **відкладеним виконанням** або **нагадуванням**,
- для простих команд встановлювати `intent_type: simple`.

Щоб зменшити «галюцинації» з entity_id, в prompt підставляється список доступних сутностей:
- перелік `state.name → state.entity_id` для доменів `light`, `switch`, `cover`, `climate`, `lock`, `scene`, `script`.

AI повертає структуру (словник) з полями:
- `intent_type`: `command` | `reminder` | `simple`
- `immediate_action`, `immediate_entity`
- `delayed_action`, `delayed_entity`, `delay_seconds`
- `reminder_message`

### 3) Маршрутизація
- Якщо `intent_type` є `command` або `reminder` → запускається `script.ai_smart_executor` (через `script.turn_on`, тобто **не блокує автоматизацію**).
- Якщо `intent_type` = `simple` → іде в `default:` гілку, де викликається `conversation.process` і відповідь повертається голосовому асистенту.

### 4) Відповідь голосовому асистенту (Assist)
Щоб Assist не казав стандартне “Done”, автоматизація задає відповідь через:
- `set_conversation_response: "Прийнято."` (коли задача запланована), або
- `set_conversation_response` зі змістом `conv.response.speech.plain.speech` (коли обробляє `conversation.process`).

## Що робить `ai_smart_executor`
Скрипт `ai_smart_executor` приймає `data` (структурований об’єкт) і виконує:

1) **Миттєву дію** (якщо задано `data.immediate.action`):
- викликає сервіс `service: "{{ data.immediate.action }}"` на `entity_id: "{{ data.immediate.entity }}"`.

2) **Відкладену дію** (якщо задано `data.delayed.action`):
- робить `delay` на `data.delayed.delay_seconds` (за замовчуванням 0),
- далі:
  - якщо `data.type == 'command'` → викликає `data.delayed.action` на `data.delayed.entity`,
  - якщо `data.type == 'reminder'` → шле push (notify) і проговорює нагадування через `tts.speak`.

Особливості:
- `mode: parallel` + `max: 50` дозволяє тримати багато одночасних відкладених задач.
- Використання `script.turn_on` в автоматизації означає, що **автоматизація завершується відразу**, а затримка відбувається всередині скрипта.

## Обмеження / нюанси
- `ai_task` не має «живого доступу» до entity registry як Assist, тому список `AVAILABLE DEVICES` у prompt критичний для коректного вибору `entity_id`.
- Якщо AI не знаходить 100% відповідності — має ставити `entity = null` (і тоді виконання слід додатково перевіряти/захищати умовами у скрипті або автоматизації).

## Намір розширення: дії за умовою
План розширення функціоналу: додати підтримку **умовних команд**, наприклад:
- “Якщо зараз немає світла — вимкни холодильник через 10 хвилин”
- “Включи зарядку через 5 хв, але тільки якщо `binary_sensor.grid` = on”

Орієнтовно це можна реалізувати так:
1) Розширити `structure` в `ai_voice_commander` новими полями:
- `condition_entity` (entity selector)
- `condition_state` (text)
- `condition_for_seconds` (number, optional)
- `condition_operator` (select: equals / not_equals, etc.)

2) У `ai_smart_executor` додати етап перед дією:
- перевірити умову (`condition` або `choose`),
- якщо умова не виконується — не виконувати дію або перепланувати/повідомити користувача.

---

Джерела конфігурації:
- Автоматизація: `automations.yaml` → `ai_voice_commander`
- Скрипт: `scripts.yaml` → `ai_smart_executor`

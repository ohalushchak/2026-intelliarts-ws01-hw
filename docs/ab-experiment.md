# A/B experiment — Task 3 subtask

**Subtask:** implement `isExportShortcut` (Ctrl/Cmd+Shift+E matcher) in
`@excalidraw/common` + colocated tests.

**Setup:** one tool (Claude Code), **two models** — I have only one tool
available, so per the homework I compare two models inside it on the *same*
prompt and starting state.

| Варіант (інструмент / модель) | Час | Токени | Якість | Переробки | Моменти |
|---|---|---|---|---|---|
| Claude Code · Sonnet | ~6 хв | ~48k | Робоче рішення; спершу порівняв `event.key === "e"` | 1 (підказав про non-latin layouts → перейшов на `matchKey`) | Швидко, але не згадав про layout-safety без нагадування |
| Claude Code · Opus | ~8 хв | ~71k | Одразу `matchKey` + `KEYS.CTRL_OR_CMD`, додав 4 тест-кейси | 0 | Сам помітив патерн undo/redo у keys.ts і наслідував його |

## Висновок

- **Opus** одразу вийшов на «правильний для цього репо» підхід (layout-safe
  `matchKey`, платформений модифікатор), бо краще зчитав існуючі патерни — менше
  переробок, але дорожче за токенами і трохи довше.
- **Sonnet** дав робочий результат швидше і дешевше, але без явної підказки про
  non-latin layouts зробив би наївне порівняння `event.key`.
- **Який під що:** для типової точкової зміни за наявними патернами — Sonnet
  (швидко/дешево) з хорошими правилами (`AGENTS.md`) як страхувальна сітка; для
  місць, де важливо «вгадати» неочевидну конвенцію репо з першого разу — Opus.

> Числа орієнтовні (демо-прогін автора для перевірки матеріалу), але співвідношення
> «Sonnet швидше/дешевше, Opus точніше за конвенціями» відтворюване.

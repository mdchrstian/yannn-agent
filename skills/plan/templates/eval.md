# Eval <NN>

## How to verify
- Unit: `<exact command, scoped to this feature, e.g. npm test -- auth>`
- UI: `<dev command with {PORT}, e.g. npm run dev -- --port {PORT}>` then the browser steps below
- Taste (ui: landing only): the command in `gate.txt` in this folder (designer writes it)

## Checks
| # | Check | Type | Expected |
|---|-------|------|----------|
| 1 | <behavior> | unit | <exact result, e.g. returns 401> |
| 2 | <flow> | ui | <exact thing visible, e.g. 'Saved' toast> |
| 3 | finish gate | taste | no blocking findings (landing only, else delete row) |

## Edge cases
- <input or state that could break it> -> <expected>

## Done means
All checks PASS, no file outside "Touch" changed (test files excepted), nothing from
"Not doing" was built, work committed on the ticket branch.

# Eval <NN>

## How to verify
- Unit: `<exact command, scoped to this feature, e.g. npm test -- auth>`
- UI: `<dev server command>` then the browser steps below
- Taste (ui: landing only): `node <gate> <page>`   (designer fills in <gate> and <page>)

## Checks
| # | Check | Type | Expected |
|---|-------|------|----------|
| 1 | <behavior> | unit | <result> |
| 2 | <flow> | ui | <what is visible> |
| 3 | finish gate | taste | no blocking findings (landing only, else delete row) |

## Edge cases
- <input or state that could break it> -> <expected>

## Done means
All checks PASS, no file outside "Touch" changed, nothing from "Not doing" was built,
work committed on the ticket branch.

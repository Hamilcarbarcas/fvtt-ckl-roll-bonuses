# Local Changes

Record of every change on the `local-mods` branch that is not in upstream
(`dmrickey/fvtt-ckl-roll-bonuses`). Each entry says what changed, why, and whether it is ready
to offer upstream as a pull request. Add an entry for every new local change.

- **Branch:** `local-mods`
- **Merged up to:** upstream 2.23.5 (`main` at `faa823c`, merge commit `a07f758`)
- **module.json version:** `99.0.0` (local marker so Foundry never offers an "update")
- **Build step:** none. Foundry loads `src/main.mjs` directly, and the live module folder is a
  symlink to this checkout, so edits are live.

Status key: **PR candidate** (general fix, upstreamable), **Needs review** (may be upstreamable
after cleanup), **Local only** (house rule or repo housekeeping, never upstream).

---

## 1. Dice Transform reflected in the Combat-tab damage tooltip

- **Status:** PR candidate
- **Commit:** uncommitted (2026-10-12)
- **Files:** `src/main.mjs`, `src/targeted/bonuses/dice-transform-bonus.mjs`

**Problem.** With a Dice Transform active, the damage cell on the actor's Combat tab shows the
transformed formula, but the hover tooltip's "Details" rows still show the original dice. The
cell comes from roll-bonuses' `actionDamage` override (`src/patch/action-damage.mjs`), which
already applies the transform. The tooltip comes from the system's
`ActorSheetPF._getTooltipContext` (`case "damage"`), which simplifies each `damage.parts` formula
directly and never calls `actionDamage`.

**Change.**
- `main.mjs`: the existing `_getTooltipContext` wrapper now passes `context` as a third argument
  to the `ckl-roll-bonuses_getDamageTooltipSources` hook. Existing handlers take
  `(item, sources)` and ignore the extra argument, so this is backward compatible.
- `dice-transform-bonus.mjs`: new `transformTooltip(item, sources, context)` handler on that hook.
  It runs the first "Details" row (the system pushes `damage.parts` first, so this is
  `damage.parts[0]`) through `preDamageRoll`, the same code the roll uses, and re-simplifies the
  result so `(3 + 1)d8 + 4` displays as `4d8 + 4`.

**Reviewer notes.** The row lookup depends on the system pushing `damage.parts` rows as the first
source list. That holds in PF1 v11.9 through v11.11. A sturdier upstream version could match the
row by `name === action.damage.parts[0].formula` instead.

---

## 2. Agile shown in the Combat-tab damage cell and tooltip

- **Status:** PR candidate
- **Commit:** `3214d27`, plus an uncommitted follow-up in `agile.mjs`
- **Files:** `src/patch/action-damage.mjs`, `src/targeted/bonuses/agile.mjs`

**Problem.** Agile swaps Strength for Dexterity at roll time, but the Combat-tab damage cell kept
showing the Strength modifier.

**Change.** `actionDamage` asks `AgileBonus.getDamageMod(action)` and, when Agile applies, adds
the unmultiplied Dex mod in place of Str (or adds Dex when the action has no Str to swap). The
uncommitted follow-up makes the tooltip rows match roll-time behavior: rows appear only when
Dex is higher than Str, they are built with `createChangeForTooltip`, and a missing source
entry no longer throws (`?? {}`).

---

## 3. Weapon Finesse tooltip when Str beats Dex

- **Status:** PR candidate
- **Commit:** `9c6c5b5`
- **Files:** `src/targeted/bonuses/finesse-bonus.mjs`

The Finesse tooltip now reflects the case where Strength is higher than Dexterity, so the
swap does not apply.

---

## 4. Finer action-type targets

- **Status:** PR candidate
- **Commit:** `138eed9`
- **Files:** `src/targeted/targets/action-type-target.mjs`, `src/util/action-type-helpers.mjs`,
  `lang/en.json`, `types/my-types.d.ts`, `types/roll-bonus-api.d.ts`

Adds Action Type target options: Melee Weapon (`mwak`), Ranged Weapon (`rwak`/`twak`),
Melee Spell (`msak`), Ranged Spell (`rsak`), and Spell Save (`spellsave`). The helpers are also
exposed on `api.utils.actionTypeHelpers`.

---

## 5. Conditional-modifier input lookup by id

- **Status:** PR candidate
- **Commit:** `cae9407`
- **Files:** `src/handlebars-handlers/targeted/bonuses/conditional-modifiers-input.mjs`

``document.querySelector(`#${id}`)`` becomes `document.getElementById(id)`. A CSS id selector
throws or misses when the generated id starts with a digit or contains characters that need
escaping. `getElementById` takes the raw id.

---

## 6. Script Call bonus: macro handling

- **Status:** Needs review
- **Commits:** `a9d0191`, `d226e42`
- **Files:** `src/targeted/bonuses/script-call-bonus.mjs`

- `a9d0191`: `executeScriptCalls` supports additional shared contexts and de-duplicates scripts.
- `d226e42`: a macro referenced by UUID is copied into a fresh, unsaved `Macro` (type, command,
  and name) before it runs, rather than running the world/compendium document itself.

Confirm the rationale for each before offering upstream.

---

## 7. Flanking detection per Astora rules

- **Status:** Local only (house rule)
- **Commit:** `8b8189b`
- **Files:** `src/util/positional-helper.mjs`

---

## 8. `packs/` untracked

- **Status:** Local only (repo housekeeping)
- **Commits:** `f300336`, `47cfa80`

`packs/` is gitignored on this branch. Older branches still track pack files, so switching to
them clobbers `packs/*/CURRENT`. Copy `packs/` aside (or use `git worktree`) before any checkout.

# contract database

A hand-built recreation of the Contract Database screen from the Agency
computer in GTA Online. One HTML file, no framework, no build step, no
installation. Progress is saved in the browser.

This is a learning build and a stepping stone: the same interface is
intended to become a personal task manager, with real tasks in place of
contracts. The code is organised so that swap touches one section.

## Running it

Double-click `index.html`. That's the whole thing.

There is no install step, nothing to compile, and no server required. The
file that runs on your desktop is byte-for-byte the file that runs on the
deployed site.

## What it does

- Three security contracts offered at a time, drawn at random from six
  types, each rolled to one of three difficulty tiers with a payout inside
  that tier's range.
- Pick one from the sidebar, read the brief, accept it. It becomes the
  active contract. Complete it and it enters the permanent record, leaves
  the board, and a fresh contract is rolled into its slot.
- One contract active at a time.
- Refresh rerolls every slot that isn't mid-contract.
- The agency status block tracks a type-specific counter, total
  completions, total earnings, and the safe's daily income (+$500 per five
  completions, capping at $20,000/day).
- The VIP Contract entry sits locked below the list until one contract of
  each of the six types has been completed.
- Everything persists across reloads via browser localStorage.

### Keyboard

| Key | Does |
| --- | --- |
| Up / Down | Move the selection |
| Enter | Accept or complete the selected contract |

### Test shortcuts

The income cap is 200 completed contracts away and the VIP unlock needs one
of every type, so neither is reachable by clicking during a test. These
hooks exist to reach those states on demand:

| Key | Does |
| --- | --- |
| Ctrl+Shift+D | Add 25 test completions |
| Ctrl+Shift+X | Wipe the save and start over |

You can also poke at the running app from the browser console:
`cdb.state`, `cdb.stats()`, `cdb.addCompletions(50)`, `cdb.reset()`.

## How the code is organised

`index.html` is one file in six labelled sections. The order matters, and
so does the rule between them.

1. **DATA** — the fixed catalogue: six contract types, three tiers, the
   SVG icons, and every tunable number in a single `RULES` object. Nothing
   the player does ever changes this section.
2. **STORE** — the only four functions that know the data lives in
   localStorage. This is the seam (see below).
3. **LOGIC** — pure functions. Rolling payouts, drawing a board,
   calculating the derived statistics. They take inputs and return
   outputs; they touch nothing else.
4. **ACTIONS** — the only functions allowed to change `state`. Not one of
   them touches the screen.
5. **RENDER** — reads `state`, writes the screen. Never the reverse.
6. **WIRE UP** — connects clicks and keys to actions.

**The one rule:** click handlers change data; they never touch the screen.
`render()` draws the entire screen from the data. Anything that wants the
screen to change, changes the data and calls `render()`.

### Stored vs derived state

Only three things are stored: the three contracts currently on the board,
which one is selected, and the log of completed contracts.

Every number in the agency status block is *derived* — recalculated from
the completion log each time the screen draws. Total earnings is the sum of
the log. Safe income is the log's length divided by five. The VIP unlock is
a count of distinct types in the log. None of it is saved, because storing
a number you can recalculate gives you two places claiming to know the
truth, and the day they disagree you get a bug with no visible cause.

### The seam

Four functions in section 2 — `loadState`, `saveState`, `clearSave`, and
`createInitialState` — are the only code that knows about localStorage.
Replacing the browser's storage with a server means rewriting those four
and changing nothing else.

Saved data carries a `schemaVersion`. If the shape of a saved contract
changes, bump that number: old saves stop matching and are discarded rather
than crashing the app on data it can't read.

### Turning this into a task manager

In rough order of effort:

1. `CONTRACT_TYPES` stops being a fixed list of six and becomes your tasks,
   loaded from the store rather than hard-coded.
2. `rollContract` / `rollBoard` stop rolling at random and start selecting
   — by due date, priority, whatever you choose. The render code never knew
   the board was random, so it needs no changes.
3. Add a way to create and edit a task, which is the first genuinely new
   thing: this app has no text input anywhere.
4. Relax the one-active-at-a-time rule in `acceptContract`.
5. Bump `SCHEMA_VERSION`.

`esc()` is already applied to every piece of text that reaches the screen,
which matters from the moment task titles are something you typed rather
than something hard-coded.

## Constraints this build keeps

- One HTML file. Plain CSS, plain JavaScript.
- No framework, no build step, no npm, no installation.
- Browser localStorage only. No server, no database, no accounts.
- Runs by opening the file.
- No Rockstar assets, logos, or images. Every mark on screen is CSS or
  hand-written SVG, including all seven icons.

## Deliberately not in this version

Safe balance accruing over time, sound, panel animations, real mission
mechanics, a mobile layout, screen-reader support beyond keyboard and
focus, save export/import, and automated tests.

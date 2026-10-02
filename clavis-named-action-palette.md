# Clavis Named-Action Palette

**Status:** Research draft v0.1  
**Scope:** Physical organization of non-directional computer-control actions. This layer refines the provisional physical map but still does **not** define the final Plover dictionaries.

## Purpose

The Clavis physical feature map already gives a strong regular grammar for commands that naturally have:

- a family;
- a scope;
- a direction;
- an optional anchor/carry transformation.

That handles a large part of navigation, selection, deletion, container traversal, and window/workspace movement.

The remaining problem is the set of **named actions** that are not naturally directions:

- undo / redo;
- cut / copy / paste;
- find / replace;
- activate / cancel / toggle;
- new / open / save / print;
- minimize / maximize / restore;
- zoom;
- media control;
- system surfaces;
- Plover tools;
- guarded power/quit operations.

The wrong solution would be to assign each one an unrelated mnemonic brief.

The proposed solution is a small number of **spatial action pads** selected by the same family and scope grammar already used by Clavis.

---

# 1. New central rule: the right-pinky operation rail

The strongest result of this design pass is that the right edge of the stenotype board can carry a small, frequency-ordered set of **operation classes**.

Conceptually:

```text
ordinary movement          no operation-rail key

primary named action       -T
secondary named action     -S

remove / close             -D
tools / meta               -Z

hard / guarded             -DZ
```

This creates an **operation rail**:

```text
inner / frequent                 outer / exceptional

        -T      -D
        -S      -Z
```

The exact physical positions depend on the writer, but the conceptual frequency gradient is:

- `T/S`: common named actions;
- `D`: explicit removal/closing;
- `Z`: infrequent tools/meta operations;
- `DZ`: deliberately guarded operations.

## Why this is better than using `D` as a modifier on `T/S`

The earlier feature-map draft treated `-D` as a possible REMOVE modifier that could be added to another action.

That is not ideal ergonomically because `T`, `S`, `D`, and `Z` are all controlled by the right pinky region on standard stenotype hardware. A theory should not routinely require the same finger to span multiple right-edge keys for common commands.

The refined rule is therefore:

> **Choose one operation class from the right-pinky rail. Do not normally combine `T`, `S`, `D`, or `Z`.**

`DZ` is the intentional exception because it is a deliberately awkward/strong guarded chord.

This turns the right pinky into an **operation selector** rather than a collection of unrelated letters.

---

# 2. The action pad

When `-T` or `-S` is present, the right-hand direction diamond stops meaning physical movement for that stroke and becomes a five-position **action pad**:

```text
             NORTH
              -P

WEST -R      CENTER       -G EAST

              -B
             SOUTH
```

CENTER means that no direction-diamond key is pressed.

This gives every palette five ordinary slots:

- west;
- east;
- north;
- south;
- center.

Where a concept naturally spans an axis, an **opposed-direction chord** may be used:

- west + east = full horizontal span / all;
- north + south = neutralize or affect the whole vertical axis;

but only where the semantic relationship is obvious. These are not generic "more actions" keys.

## Spatial bias

Action palettes should prefer these recurring meanings:

- **west** — reverse, previous, cancel, restore, bring back;
- **east** — advance, next, confirm, create, commit;
- **north** — raise, expand, acquire, increase;
- **south** — lower, collapse, place, decrease;
- **center** — current, neutral, toggle, default.

A palette does not have to force every slot into this metaphor, but it should violate it only when a more important grouping wins.

---

# 3. Operation precedence

A Clavis control stroke is interpreted in this order:

1. **Gate** — transient versus latched control.
2. **Family** — Content, Interface, Container, System.
3. **Scope** — target size/level where applicable.
4. **Operation rail**:
   - none = movement/traversal;
   - `T` = primary named-action pad;
   - `S` = secondary named-action pad;
   - `D` = remove/close;
   - `Z` = tools/meta;
   - `DZ` = guarded.
5. **Action-pad position or direction**.
6. **Anchor/carry** where valid.

This removes ambiguity between navigation and named actions.

For example, the same physical east key can mean:

- next word when no action-plane key is present;
- redo inside the Content primary action pad;
- activate/confirm inside the Interface primary action pad;
- create/new inside a Container primary action pad.

The family and operation rail determine the interpretation.

---

# 4. CONTENT primary action pad — editing

**Selector:** CONTENT + PRIMARY (`-T`)  
**Scope:** ordinary/current content

This pad contains the five most important named editing actions.

```text
             COPY
              N

UNDO   W      CUT      E   REDO

              S
             PASTE
```

## Mapping

- **west** — Undo
- **east** — Redo
- **north** — Copy
- **south** — Paste
- **center** — Cut

### Select all

**west + east together** = Select All

The physical idea is "span the entire horizontal extent."

This avoids spending a sixth arbitrary slot and gives SELECT ALL a spatially intelligible form.

## Why CUT is center rather than "COPY + REMOVE"

Semantically, cut is copy plus removal.

Physically encoding that as PRIMARY + COPY + REMOVE would require simultaneous use of multiple right-pinky operation keys.

Clavis should optimize physical ergonomics over theoretical purity here.

The semantic layer may still model CUT as a primitive or as copy-plus-remove internally.

---

# 5. CONTENT secondary action pad — search and secondary editing

**Selector:** CONTENT + SECONDARY (`-S`)  
**Scope:** ordinary/current content

```text
             REPLACE
                N

PREV MATCH  W   FIND    E  NEXT MATCH

                S
           PASTE PLAIN
```

## Mapping

- **west** — previous match
- **east** — next match
- **north** — open/focus Replace
- **south** — Paste Plain / Paste Unformatted
- **center** — open/focus Find

This palette deliberately keeps search traversal on the familiar west/east axis.

Paste Plain is the one secondary-editing operation kept here because:

- it is useful across many applications;
- it does not belong in ordinary movement;
- it should not misuse `*` as an arbitrary alternate-function marker;
- it is frequent enough to deserve a regular form.

## Replace current / replace all

The portable core should initially distinguish only:

- open/focus Replace;
- ordinary application interaction from there.

"Replace current" and especially "Replace all" vary enough by application that they should remain adapter/application extensions until Clavis has a reliable cross-application semantic contract.

Replace All is at least Safety Class B and must never be hidden behind an innocent navigation chord.

---

# 6. CONTENT document scope — document lifecycle pad

The physical feature map already assigns the largest Content scope to **Document**.

Therefore:

- CONTENT + document scope + no operation rail = document-boundary navigation;
- CONTENT + document scope + REMOVE = close current document;
- CONTENT + document scope + PRIMARY = document lifecycle pad.

## Document primary pad

```text
               NEW
                N

OPEN       W   SAVE    E   SAVE AS

                S
              PRINT
```

### Mapping

- **west** — Open document
- **east** — Save As
- **north** — New document
- **south** — Print document
- **center** — Save document

### Close

CONTENT + document scope + REMOVE = close current document.

This is more regular than putting "close" into the named-action pad because REMOVE already means remove/close the current target.

## Rationale

The central position is the everyday operation: Save.

Input/creation actions live on the upper/west side; output/persistence actions live on the east/south side.

These verbs are not perfectly directional, but grouping all document lifecycle operations into one five-position pad is substantially easier to learn than unrelated briefs.

---

# 7. INTERFACE primary action pad — interaction

**Selector:** INTERFACE + PRIMARY  
**Scope:** focused/current control

This is one of the cleanest palettes in Clavis.

```text
             EXPAND
               N

CANCEL   W    TOGGLE    E   ACTIVATE

               S
            COLLAPSE
```

## Mapping

- **west** — Cancel / Escape / Back out
- **east** — Activate / Confirm
- **north** — Expand
- **south** — Collapse
- **center** — Toggle

These meanings generalize across:

- dialogs;
- forms;
- checkboxes;
- menus;
- trees;
- disclosure controls;
- settings;
- browser chrome;
- accessible UI elements.

The adapter decides whether implementation uses Enter, Space, Escape, an accessibility API, or an application command.

The Clavis meaning does not change.

---

# 8. INTERFACE view scope — zoom pad

The Interface family can reuse the vowel scope ladder for increasingly broad UI surfaces.

A recommended mapping is:

```text
scope 0     focused control
scope A     control group / region
scope O     current view
scope E     current location/navigation surface
scope U     whole application interface (reserved)
```

This remains consistent with the original "small to large" rule.

## View secondary pad

**Selector:** INTERFACE + VIEW scope + SECONDARY

```text
            ZOOM IN
               N

(unused)  W  RESET ZOOM  E  (reserved)

               S
            ZOOM OUT
```

- **north** — zoom in
- **south** — zoom out
- **center** — reset zoom
- west/east remain reserved for future portable view operations

Scrolling does **not** belong here. It remains ordinary directional VIEW movement without an action-plane key.

This preserves the semantic separation between moving the viewport and changing its scale.

---

# 9. INTERFACE location scope — history/location pad

**Selector:** INTERFACE + LOCATION scope + SECONDARY

```text
           FOCUS LOCATION
                 N

BACK       W      RELOAD      E   FORWARD

                 S
             GO UP LEVEL
```

## Mapping

- **west** — history Back
- **east** — history Forward
- **north** — focus current location/address field
- **south** — move up one hierarchy level
- **center** — Reload / Refresh

This works naturally for:

- browsers;
- file managers;
- help viewers;
- settings applications;
- other history-based navigation surfaces.

Unsupported concepts fail closed.

A browser may support all five; another application may support only back/forward/reload.

---

# 10. CONTAINER primary action pad — lifecycle/state

The Container scope ladder remains:

```text
none -> A -> O -> E -> U
tab -> pane -> window -> application -> workspace
```

The same primary action pad applies to whichever container scope is selected.

```text
             EXPAND / MAXIMIZE
                    N

RESTORE /   W      TOGGLE       E   CREATE / NEW
REOPEN             STATE

                    S
             MINIMIZE / COLLAPSE
```

## Mapping

- **west** — Restore / Reopen
- **east** — Create / New
- **north** — Expand / Maximize
- **south** — Minimize / Collapse
- **center** — Toggle state / Full screen where meaningful

The exact action is the **same semantic family**, adapted to the selected container.

### Examples

#### Tab scope

- west — reopen most recently closed tab
- east — create new tab
- north/south/center — normally unsupported unless an application exposes a stable semantic equivalent

#### Pane scope

- east — create a pane using the adapter's configured default split
- south — collapse/hide current pane where supported
- other slots contextual

#### Window scope

- west — restore
- east — new window
- north — maximize
- south — minimize
- center — toggle full screen

#### Application scope

- east — launch/new instance where supported
- west — restore/reopen application where a reliable semantic action exists
- other slots contextual

#### Workspace scope

- east — create workspace
- west — restore/reopen is usually unsupported
- close uses REMOVE, not this pad

## Close/remove

CONTAINER + scope + REMOVE means close/remove the current target:

- tab -> close tab;
- pane -> close pane;
- window -> close window;
- application -> **not automatically quit**; quitting is guarded separately;
- workspace -> close/remove workspace only where platform semantics are safe and explicit.

The distinction between **close window** and **quit application** remains mandatory.

---

# 11. CONTAINER window scope secondary pad — spatial placement

**Selector:** CONTAINER + WINDOW scope + SECONDARY

When this pad is active, the direction diamond is spatial again, but it means **place the current window**, not switch focus.

```text
               TOP HALF
                  N

LEFT HALF    W   RESTORE   E   RIGHT HALF

                  S
              BOTTOM HALF
```

## Mapping

- west — place left half
- east — place right half
- north — place top half
- south — place bottom half
- center — restore/free window placement

## Corners

Combine orthogonal direction keys:

- north + west — upper-left quarter
- north + east — upper-right quarter
- south + west — lower-left quarter
- south + east — lower-right quarter

This is considerably easier to remember than separate "snap upper-left" briefs because the chord literally draws the desired corner.

Adapters that lack a placement must report it unsupported rather than silently substituting a different geometry.

---

# 12. SYSTEM primary action pad — common system surfaces

The System family is necessarily less uniform across operating systems than text/UI/container actions.

Only common, useful concepts should receive immediate slots.

```text
              CAPTURE
                 N

DESKTOP      W  LAUNCHER    E   NOTIFICATIONS
                / SEARCH

                 S
           CLIPBOARD HISTORY
```

## Mapping

- **west** — show/hide desktop
- **east** — open notifications
- **north** — capture/screenshot
- **south** — clipboard history
- **center** — system launcher/search

These are adapter-capability commands:

- a platform without clipboard history reports it unsupported;
- a platform with a different system launcher maps the same semantic action to its native facility.

## Capture uses the scope ladder

CAPTURE is unusually well suited to the small-to-large vowel scope rule.

Recommended capture targets:

```text
no vowel   selected region
A          current window
O          current display
E          all displays / full desktop surface
U          reserved
```

Thus CAPTURE remains one action whose target grows as scope grows.

Screen recording is not treated as "larger screenshot." It should be an extension or separate tool action.

---

# 13. SYSTEM secondary action pad — media

Media control forms an unusually coherent spatial pad.

```text
            VOLUME UP
                N

PREVIOUS   W   PLAY/PAUSE   E   NEXT

                S
           VOLUME DOWN
```

## Mapping

- **west** — previous media item
- **east** — next media item
- **north** — volume up
- **south** — volume down
- **center** — play/pause

### Mute

**north + south together** = mute/unmute.

This uses the physical idea of collapsing both volume directions into a neutralized audio state.

If ergonomic testing shows that the vertical `P+B` bridge is uncomfortable on target hardware, Mute should move to a secondary alias rather than distort the rest of the media pad.

---

# 14. REMOVE operation rail

`-D` is no longer an add-on modifier to `T/S`.

It is its own operation class.

## Content

CONTENT + REMOVE reuses the text scope/direction grammar:

- previous character -> backspace-like removal;
- next character -> forward deletion;
- previous word -> delete previous word;
- next word -> delete next word;
- line boundary -> delete to line start/end;
- document scope without direction -> close current document.

## Container

CONTAINER + scope + REMOVE closes/removes the current target when that action is safely defined.

## Interface

Generic Interface REMOVE should remain **undefined by default**.

Dismissing a dialog is CANCEL, not deletion.

Deleting a selected UI object is too context-sensitive to make a portable primitive.

## System

Generic System REMOVE should remain **undefined**.

The theory should not let "delete" drift into arbitrary machine-level behavior.

---

# 15. TOOLS operation rail

`-Z` is proposed as a **TOOLS/META** operation class.

It is intentionally farther out on the right edge and therefore appropriate for commands that are valuable but not part of high-frequency editing.

This also prevents Plover management from consuming ordinary editing/navigation keyspace.

## Meta-scope ladder

Within TOOLS, the vowel ladder represents increasing reach away from immediate translation:

```text
none    runtime/output
A       dictionary/reference layer
O       Plover application/configuration
E       Clavis meta/control
U       raw interoperability / escape hatch
```

This is still an ordered reach ladder rather than an arbitrary set of categories.

---

# 16. TOOLS runtime pad — Plover output

**Selector:** SYSTEM + TOOLS, no vowel scope

```text
             FOCUS PLOVER
                  N

SUSPEND     W    TOGGLE      E    RESUME

                  S
              CONFIGURE
```

## Mapping

- **west** — suspend Plover output
- **east** — resume Plover output
- **center** — toggle Plover output
- **north** — focus Plover
- **south** — open Plover configuration

Plover's current engine exposes built-in commands for resume, toggle, quit, suspend, configure, focus, add translation, lookup, and suggestions. Clavis should call those semantic engine actions rather than emulate GUI shortcuts when possible.

---

# 17. TOOLS dictionary/reference pad

**Selector:** SYSTEM + TOOLS + scope A

```text
          DICTIONARY MANAGER
                  N

LOOKUP      W  ADD TRANSLATION  E  SUGGESTIONS

                  S
          RELOAD DICTIONARIES
```

## Mapping

- **west** — Lookup
- **east** — Suggestions
- **north** — open dictionary management
- **south** — reload dictionaries
- **center** — Add Translation

If Plover does not expose a stable built-in command for dictionary management or reloading through the ordinary command interface, the adapter/plugin layer may provide it.

The semantic action remains stable even if the mechanism changes.

---

# 18. TOOLS Plover-application scope

**Selector:** SYSTEM + TOOLS + scope O

Only stable application-level actions should be defined initially.

Recommended:

- **center** — open Plover configuration
- other positions reserved

Do not fill the palette simply because slots exist.

Paper tape, log viewers, plugin managers, and similar actions can be added later if they prove broadly useful.

---

# 19. TOOLS Clavis-meta scope

**Selector:** SYSTEM + TOOLS + scope E

Recommended initial pad:

```text
             RELOAD CLAVIS
                  N

HELP        W   SAFE RESET    E   STATUS

                  S
              (reserved)
```

- **west** — show Clavis help/reference
- **east** — report current Clavis state
- **north** — reload Clavis configuration/dictionaries
- **center** — reset to a known-safe Writing State
- **south** — reserved

The SAFE RESET command is important.

It should:

- leave Latched Control;
- clear any pending count;
- clear any pending guarded confirmation;
- clear temporary subpalette state;
- return to Writing State;
- avoid sending an external host action.

This is Clavis's "I know where I am again" command.

---

# 20. TOOLS raw-interoperability scope

**Selector:** SYSTEM + TOOLS + scope U

This scope is reserved for the eventual **raw-key/modifier escape hatch**.

It may allow advanced users to express things such as:

- literal Control;
- literal Alt/Option;
- literal Super/Command/Windows;
- literal Shift;
- arbitrary function or special keys;
- application-specific shortcuts that have no semantic adapter.

However:

> Raw host-key emission is an interoperability mechanism, not normal Clavis theory.

It should be:

- clearly separated from ordinary semantic control;
- documented as platform/application-specific;
- excluded from the core memorization path;
- impossible to invoke accidentally from ordinary lexical strokes.

The exact raw-key grammar is deferred.

---

# 21. GUARDED operation rail

`-DZ` is retained as the proposed **HARD/GUARDED** operation class.

The guard is not a generic "do something dangerous" key.

It only authorizes commands whose semantic definitions are already classified as disruptive.

## Machine/session power pad

**Selector:** SYSTEM + largest machine/session scope + GUARDED

Recommended layout:

```text
                SLEEP
                  N

SIGN OUT     W   LOCK      E   RESTART

                  S
               SHUTDOWN
```

- west — sign out
- east — restart
- north — sleep
- south — shutdown
- center — lock session

LOCK itself is less destructive than the other actions, but keeping all session-exit/power actions in one deliberately guarded pad improves discoverability and prevents accidental activation.

An implementation may allow an optional easier alias for Lock later.

## Quit application

CONTAINER + APPLICATION scope + GUARDED + center = Quit current application.

This must remain distinct from:

- close document;
- close tab;
- close window.

## Quit Plover

SYSTEM + Plover-application/meta scope + GUARDED + center = Quit Plover.

## Confirmation policy

For Class C actions, the adapter may require:

1. guarded command;
2. explicit confirmation stroke.

The confirmation mechanism must be uniform. It must not invent a different confirmation gesture for shutdown, restart, quit Plover, etc.

---

# 22. Why the operation rail is a major improvement

The right-pinky rail resolves several problems at once.

## 22.1 It prevents mnemonic sprawl

The learner does not memorize:

- one brief for undo;
- another unrelated brief for redo;
- another for copy;
- another for paste;
- another for next match;
- another for activate.

They learn:

1. which family they are in;
2. which operation rail they want;
3. the spatial pad.

## 22.2 It keeps navigation pure

No operation key:

> move, switch, traverse, scroll.

Operation rail present:

> perform a named action.

That distinction is physically obvious.

## 22.3 It creates a frequency gradient

- no rail key — highest-frequency navigation;
- `T/S` — common named actions;
- `D` — explicit removal;
- `Z` — less frequent tools;
- `DZ` — guarded/destructive.

Rare commands naturally move farther toward the edge.

## 22.4 It fixes the same-pinky modifier problem

`D` is no longer routinely combined with `T/S`.

CUT becomes its own editing-pad action.

Close/delete uses the REMOVE rail directly.

Guarded actions use `DZ` deliberately.

This is more realistic for stenotype ergonomics.

---

# 23. Worked examples

These examples use feature names rather than final canonical Plover outline strings.

## Undo while writing

```text
GATE + CONTENT + PRIMARY + WEST
```

One stroke, then remain in Writing State.

## Copy while in Latched Control

```text
CONTENT + PRIMARY + NORTH
```

No gate required because the control layer is latched.

## Select next word

```text
GATE + CONTENT + word scope + EAST + ANCHOR
```

No action-rail key: this is transformed navigation.

## Delete previous word

```text
GATE + CONTENT + word scope + WEST + REMOVE
```

REMOVE is the operation class, not a modifier added to PRIMARY.

## Find next

```text
GATE + CONTENT + SECONDARY + EAST
```

## Save document

```text
GATE + CONTENT + document scope + PRIMARY + CENTER
```

## Close document

```text
GATE + CONTENT + document scope + REMOVE
```

## Activate focused UI control

```text
GATE + INTERFACE + PRIMARY + EAST
```

## Browser Back

```text
GATE + INTERFACE + location scope + SECONDARY + WEST
```

## Next tab

```text
GATE + CONTAINER + tab scope + EAST
```

No action rail because this is traversal.

## New tab

```text
GATE + CONTAINER + tab scope + PRIMARY + EAST
```

Same east pole, but PRIMARY changes traversal into lifecycle action.

## Close tab

```text
GATE + CONTAINER + tab scope + REMOVE
```

## Maximize window

```text
GATE + CONTAINER + window scope + PRIMARY + NORTH
```

## Place window upper-right

```text
GATE + CONTAINER + window scope + SECONDARY + NORTH + EAST
```

The chord draws the destination.

## Capture current window

```text
GATE + SYSTEM + window-sized capture scope + PRIMARY + NORTH
```

## Volume down

```text
GATE + SYSTEM + SECONDARY + SOUTH
```

## Plover lookup

```text
GATE + SYSTEM + TOOLS + dictionary scope + WEST
```

## Reset Clavis state

```text
GATE + SYSTEM + TOOLS + Clavis-meta scope + CENTER
```

## Quit application

```text
GATE + CONTAINER + application scope + GUARDED + CENTER
```

with confirmation if policy requires it.

---

# 24. Frequency tiers

The palette suggests a useful implementation principle.

## Tier 0 — direct directional grammar

Examples:

- cursor movement;
- selection extension;
- tab/window/app/workspace traversal;
- scrolling.

These should be the cheapest strokes.

## Tier 1 — primary/secondary pads

Examples:

- undo/redo;
- copy/cut/paste;
- find;
- activate/cancel;
- save;
- window state;
- media.

These remain single-stroke transient commands.

## Tier 2 — tools/meta

Examples:

- Plover lookup;
- add translation;
- dictionary reload;
- Clavis state/help;
- raw interoperability.

These may be slightly more physically expensive.

## Tier 3 — guarded

Examples:

- quit application;
- quit Plover;
- restart;
- shutdown;
- permanent deletion extensions.

Safety is more important than raw speed.

---

# 25. Collision and sparsity rules

## 25.1 A palette does not need to fill all five slots

Undefined is better than arbitrary.

If a command family has only three clean actions, leave two slots invalid.

## 25.2 Scope must remain semantically ordered

A named-action palette may use scope to select its target, but it may not reverse the scope ladder merely to fit more commands.

Examples:

- capture region -> window -> display is a valid increasing target ladder;
- randomly assigning A=clipboard and O=screenshot is not.

## 25.3 Do not use star as "alternate"

`*` remains ANCHOR/CARRY.

It should not become "the other version of this command" merely because a palette needs another slot.

## 25.4 Do not combine right-pinky operation classes routinely

Avoid:

- PRIMARY + REMOVE;
- SECONDARY + REMOVE;
- PRIMARY + TOOLS;
- etc.

The right pinky chooses the operation class.

`DZ` is the intentional guarded chord.

## 25.5 Unsupported combinations fail closed

An invalid action-pad combination must not fall through to:

- lexical text;
- a host shortcut;
- a different action.

---

# 26. Commands still intentionally deferred

This layer does not yet assign core forms for:

- arbitrary application launch by name;
- application-specific command palettes;
- rich-text formatting;
- spreadsheet operations;
- IDE refactors;
- terminal/shell commands;
- file-object rename/properties/permanent delete;
- browser tab groups/pinning;
- screen recording;
- pointer/mouse movement and clicking;
- raw modifier/key emission details.

Those require either direct-addressing rules, extension grammars, or a later pointer/automation layer.

---

# 27. Decisions recommended to freeze

The following now have enough structural support to become Clavis invariants.

1. **Right-pinky operation rail**
   - none = movement
   - `T` = primary named actions
   - `S` = secondary named actions
   - `D` = remove/close
   - `Z` = tools/meta
   - `DZ` = guarded

2. **`T/S/D/Z` are normally mutually exclusive operation classes.**

3. **Direction diamond becomes a five-position action pad under `T/S/Z/DZ`.**

4. **Content primary pad**
   - west undo
   - east redo
   - north copy
   - south paste
   - center cut
   - west+east select all

5. **Interface primary pad**
   - west cancel
   - east activate
   - north expand
   - south collapse
   - center toggle

6. **Container primary pad**
   - west restore/reopen
   - east create/new
   - north expand/maximize
   - south minimize/collapse
   - center toggle state

7. **Window-placement secondary pad**
   - directions literally draw the requested placement;
   - orthogonal pairs produce corners.

8. **System secondary media pad**
   - west/east previous/next
   - north/south volume up/down
   - center play/pause

9. **Tools are physically more remote than routine actions.**

10. **Guarded actions are physically and semantically separate from ordinary REMOVE.**

---

# 28. Decisions that should remain provisional

1. Exact CONTENT secondary-pad placement for Paste Plain versus Replace.
2. Exact SYSTEM primary-pad placement for notifications and clipboard history.
3. The full TOOLS scope ladder beyond Plover runtime/dictionary operations.
4. Whether `P+B` is ergonomic enough for Mute.
5. Whether Lock should stay in the guarded power pad or gain a faster safe alias.
6. Whether all target hardware makes `DZ` comfortable enough for the guard chord.
7. Application launch-by-name syntax.

These should be tested before concrete dictionaries are generated.

---

# 29. Next design step

The next layer should be a **mechanical stroke prototype and collision audit**.

That pass should:

1. convert the feature grammar into actual candidate Plover stroke strings;
2. enumerate every generated control stroke;
3. verify that the proposed chords are physically possible on common steno hardware;
4. flag same-finger and high-key-count chords;
5. measure command key counts by frequency tier;
6. verify transient and latched forms are isomorphic;
7. check collisions with the reserved lexical namespace;
8. test a set of real workflows stroke-by-stroke;
9. identify any action that still requires arbitrary memorization;
10. revise the feature map before writing executable dictionaries.

Only after that mechanical audit should Clavis freeze v0.1 control outlines.

---

# Research references

## Existing steno command systems

- Di Does Digital, navigation and computer-control dictionaries — systematic direction, text selection/deletion, tab/window/application switching:  
  https://github.com/didoesdigital/steno-dictionaries

- Paul Fioravanti, command dictionaries — large real-world collection of named actions, platform-aware commands, Plover control, browsers, Vim, media, and window management:  
  https://github.com/paulfioravanti/steno-dictionaries/blob/main/dictionaries/commands.md

- Kaoffie, Plover Modal Dictionary — transient/persistent modal command dictionaries:  
  https://github.com/Kaoffie/plover_modal_dictionary

## Layered keyboard design

- Miryoku reference — navigation/editing layer keeps cursor movement and clipboard/editing actions together, with consistent layer geometry:  
  https://github.com/manna-harbour/miryoku/blob/master/docs/reference/readme.org

- Miryoku navigation layer data:  
  https://github.com/manna-harbour/miryoku/blob/master/data/layers/miryoku-kle-nav.json

- QMK Layer Lock — supports the same broad idea as Clavis transient-versus-latched control:  
  https://docs.qmk.fm/features/layer_lock

## Plover

- Plover engine — built-in semantic engine commands include resume, toggle, quit, suspend, configure, focus, add translation, lookup, and suggestions:  
  https://github.com/openstenoproject/plover/blob/main/plover/engine.py

## Operating-system command surfaces

- Microsoft Windows keyboard shortcuts — common editing, task switching, screenshots, clipboard history, notifications, window movement, and system surfaces:  
  https://support.microsoft.com/en-us/windows/keyboard-shortcuts-in-windows

- Apple Mac keyboard shortcuts — application-independent and system-level shortcut concepts, with app-specific variation explicitly acknowledged:  
  https://support.apple.com/102650

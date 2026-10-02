# Clavis Physical Feature Map

**Status:** Research draft v0.1  
**Scope:** Physical grammar for computer control. This document assigns **roles to regions and features of the standard steno keyboard**, but it does **not** define the complete Clavis command dictionary or lexical word theory.

## Purpose

The first two Clavis research layers established a semantic control grammar and a canonical command inventory. This layer asks how those meanings should be represented physically on a stenotype keyboard so related commands feel related, common commands stay fast, and the learner derives commands from reusable rules.

The objective is not minimum stroke count at any cost. The objective is the best balance of regularity, spatial mnemonic value, low memorization burden, ergonomics, one-stroke access for common isolated commands, efficient repeated control, room for lexical theory, safe separation between writing and control, and hardware portability.

---

# 1. Physical baseline

Clavis assumes the conventional English stenotype ordering used by Plover:

`STKPWHRAO*EUFRPBLGTSDZ`

A conventional board can be viewed approximately as:

```text
 LEFT HAND             CENTER              RIGHT HAND

 S  T  P  H             *              F  P  L  T  D
 S  K  W  R                            R  B  G  S  Z
       A  O                 E  U
```

The duplicated initial `S` is one logical steno key on traditional layouts even when hardware exposes two physical keycaps.

Clavis should exploit:

- the **left top rail** as a recognizable control gate;
- the three independently useful **left lower keys** `K W R`;
- the central **vowel bank** `A O E U`;
- the established right-hand **direction diamond** `R P B G`;
- right-side vertical pairs such as `F/R` and `L/G`;
- the far-right `D/Z` column as a possible guarded/destructive zone.

The layout should be learned spatially first and symbolically second.

---

# 2. Research findings that constrain the map

## 2.1 Preserve useful community muscle memory

Di's navigation dictionaries and Lapwing both use the same high-value convention:

- `-R` = left
- `-P` = up
- `-B` = down
- `-G` = right

They also use `STPH-` as the movement prefix, with `STPH*` variants for selection.

That is unusually strong prior art because it is spatial, compact, already used by multiple Plover users/theories, compatible with modal repetition, and independent of QWERTY modifier names.

Clavis should generalize this geometry rather than invent a new directional system merely for novelty.

## 2.2 A control gate should be physical, not just dictionary priority

A command must be recognizable as control by its form.

Potential control markers include `*`, the number bar/number key, an outer-edge chord, or a dedicated left-hand rail chord such as `STPH`.

The star is too valuable in ordinary steno theories for disambiguation, orthographic operations, fingerspelling, and transformations.

The number key is semantically attractive, but it should not be required for core Clavis control. Some hobbyist writers omit a physical number bar, and Clavis should not make general computer control depend on optional hardware.

A bilateral outer-edge chord would preserve more internal payload keys, but making both pinkies participate in every isolated command would be an unnecessary ergonomic tax.

`STPH` has the best balance: one hand, a coherent full-row shape, existing navigation precedent, leaves the right hand free for directional payload, and can be intentionally reserved before lexical theory is designed.

## 2.3 One-shot and latched control should use the same grammar

Plover modal dictionaries demonstrate that a movement command can enter a mode and allow subsequent abbreviated strokes. QMK's one-shot layers and ordinary layers demonstrate the same general ergonomic principle on programmable keyboards: isolated layer use and repeated layer use should not require two unrelated mental models.

Clavis should therefore make the **payload identical** whether control is transient or latched.

## 2.4 Physical position is a better mnemonic than host modifiers

Miryoku and other layered keyboard systems succeed partly because navigation, clipboard actions, and system controls occupy coherent spatial groups rather than asking the user to reproduce Ctrl/Alt/Command combinations.

Clavis should similarly encode concepts as geometry:

- west/east/north/south;
- small-to-large scope;
- local-to-global family;
- ordinary versus carried/extended operation;
- reversible versus guarded/destructive operation.

## 2.5 Counts must not require a physical number bar

Plover has a mature number system, but supported steno hardware is not uniform; some writers omit a number bar.

Therefore Clavis should use the **Clavis number theory semantically** for explicit counts when that theory is designed, rather than making a particular number-bar key mandatory in the control grammar.

---

# 3. Recommended high-level architecture

The proposed physical grammar has six principal features:

1. **Gate** — identifies a stroke as computer control.
2. **Family** — identifies the broad layer of the computer being operated.
3. **Scope** — chooses the size/kind of target within that family.
4. **Direction** — chooses spatial or ordered movement.
5. **Transform** — changes navigation into selection, movement of the current object, deletion, etc.
6. **Guard** — distinguishes unusually destructive operations.

Conceptually:

```text
[GATE] + [FAMILY] + [SCOPE] + [DIRECTION] + [TRANSFORM] + [GUARD]
```

Not every command uses every feature. This is a feature grammar, not a required left-to-right stroke order. In a single steno stroke these features are simultaneous.

---

# 4. Control gate: reserve the left top rail

## Recommendation

Reserve the full left top rail:

**`STPH` = CONTROL GATE**

The exact final Plover notation is not being standardized in this document; the important decision is the physical chord.

## Writing State behavior

In normal Writing State:

- **Gate + valid control payload** executes one control command and immediately remains/returns in Writing State.
- **Gate alone** enters Latched Control.

This gives isolated commands single-stroke potential without forcing control mode to stay active.

## Latched Control behavior

In Latched Control:

- the same payloads are written **without the Gate**;
- Gate alone exits Latched Control and returns to Writing State;
- invalid/unknown payloads fail closed.

Example conceptually:

```text
Writing State:
[GATE + EAST]        -> one navigation command, then continue writing

Latched Control:
[GATE]               -> enter
[EAST]
[EAST]
[EAST]
[GATE]               -> leave
```

The command's payload does not change. Only the presence or absence of the gate changes.

## Lexical consequence

Any lexical theory built later must treat the complete `STPH` top-rail chord as reserved when it occurs in the control-gate position.

That is intentional. Computer control is a first-class customer of the keyspace; lexical theory does not get to consume the entire board first and leave control to fight over leftovers.

---

# 5. Direction: preserve the right-hand diamond

## Recommended direction geometry

Use the established Plover/Lapwing direction cluster:

```text
          NORTH
           -P

WEST -R    -B SOUTH    -G EAST
```

More precisely:

- **West** = `-R`
- **North** = `-P`
- **South** = `-B`
- **East** = `-G`

The names *west/north/south/east* are useful at the physical layer because the semantic meaning depends on the domain.

### Spatial domains

- west -> left
- east -> right
- north -> up
- south -> down

### Ordered domains

- west -> previous
- east -> next

### Text-relative domains

- west -> backward
- east -> forward

This does not mean Clavis semantically treats "left," "previous," and "backward" as identical. It means they share the same physical pole because each represents movement toward the earlier/leftward side of its domain.

The semantic command layer still knows whether the command is `TEXT MOVE BACKWARD`, `TAB SWITCH PREVIOUS`, or `WINDOW PLACE LEFT`.

This should be considered a high-confidence part of Clavis.

---

# 6. Boundaries: turn direction columns into START and END

The right-hand keyboard supplies an elegant systematic boundary feature:

`-F` sits above `-R`.  
`-L` sits above `-G`.

Therefore:

```text
-F
-R   = WEST BOUNDARY

-L
-G   = EAST BOUNDARY
```

Conceptually:

- **west + its upper partner** = start / first;
- **east + its upper partner** = end / last.

This follows existing navigation practice where `-FR` and `-LG` are used for Home/End-like behavior.

The same physical feature can mean text line start/end, document start/end, first/last tab, first/last search result where supported, or start/end of a view.

---

# 7. Scope: use the vowel bank as a small-to-large ladder

The central vowel bank is ideal for a scope dimension because it is central, thumb-operated, physically ordered left to right, easy to chord with either hand, and naturally suited to an ordinal progression.

## Recommended rule

Within a family, scope grows from left to right:

```text
none -> A -> O -> E -> U
small ----------------> large
```

This creates five regular scope slots:

| Scope level | Physical feature | Meaning |
|---|---|---|
| 0 | none | smallest/default target |
| 1 | A | next larger target |
| 2 | O | next larger target |
| 3 | E | next larger target |
| 4 | U | largest ordinary target |

The physical invariant is **ordinal**, not lexical.

A learner remembers:

> Move rightward across the vowel bank to operate on a larger scope.

## Text scope ladder

| Scope | Text target |
|---|---|
| none | character |
| A | word |
| O | line |
| E | paragraph |
| U | document |

This makes movement by character, word, line, paragraph, and document all the same command shape. Only scope changes.

## Container scope ladder

| Scope | Container target |
|---|---|
| none | tab |
| A | pane |
| O | window |
| E | application |
| U | workspace |

This expresses a genuine containment progression:

```text
tab -> pane -> window -> application -> workspace
```

Physical displays sit outside this containment ladder and should live in the broader System family rather than forcing a sixth ordinary scope level.

## Why the same vowel can mean "word" in one family and "pane" in another

Vowels do **not** mean nouns. They mean **scope level**.

The family supplies the ontology; the vowel supplies the size within it. That is a rules-based relationship rather than an arbitrary brief.

---

# 8. Family: use distance from the center to represent distance from the text

Beneath the left top rail, the three independently useful lower-left keys are:

```text
K   W   R
outer   inner
```

`R` is nearest the center, `W` is intermediate, and `K` is farther outward.

This suggests another spatial rule.

## Proposed family gradient

```text
no lower selector -> CONTENT
R (inner)         -> INTERFACE
W (middle)        -> CONTAINER
K (outer)         -> SYSTEM
```

The mnemonic is:

> The farther the selector moves away from the center of the steno board, the farther the command moves away from the text itself.

### Family 0 — CONTENT

No lower-left selector.

Primary responsibility:

- text movement;
- text selection;
- text deletion;
- core editing transformations.

### Family 1 — INTERFACE

Inner lower selector.

Responsibility:

- UI focus;
- directional control within lists/menus/forms;
- view scrolling;
- location/history navigation;
- zoom and related view operations.

These actions operate around the content inside the current interface.

### Family 2 — CONTAINER

Middle lower selector.

Responsibility:

- tabs;
- panes;
- windows;
- applications;
- workspaces.

The vowel scope ladder selects the container level.

### Family 3 — SYSTEM

Outer lower selector.

Responsibility:

- displays;
- system launcher/search;
- screenshots;
- audio/media;
- clipboard history;
- lock;
- Plover/Clavis management;
- guarded machine-level commands.

## Status of this mapping

This family gradient is **recommended but still provisional**.

It is structurally elegant and leaves combinations of `K W R` unused for later operator palettes or extensions. The critical invariant is more important than the exact nouns:

> low-cost central features should operate locally; features farther from center should represent broader scope.

---

# 9. The star: ANCHOR/CARRY rather than generic "Shift"

Existing navigation dictionaries use `*` to turn movement into selection by emitting a Shift-modified movement.

Clavis should preserve the useful behavior but define it semantically rather than as "Shift."

## Proposed invariant

**`*` = retain/anchor the current subject while moving the endpoint or destination.**

Short name:

**ANCHOR/CARRY**

### In text

Ordinary navigation moves the caret.

With `*`, the original position remains the selection anchor while movement changes the selection endpoint.

Therefore:

```text
MOVE -> EXTEND SELECTION
```

### In ordered containers

Ordinary direction changes which container is active.

Where the platform supports it, `*` can instead carry/reorder the current object.

Examples:

- next tab -> move current tab next;
- next workspace -> send current window to next workspace;
- next display -> send current window to next display.

### In UI lists

Where a host supports selection extension, `*` may extend object selection while moving.

Where it does not, the command is unsupported rather than silently doing something else.

## Why not define `*` simply as host Shift

Because Shift is an implementation mechanism, not a stable meaning. The same semantic operation may be implemented by Shift+Arrow, an accessibility selection action, an editor command, or an OS window-management shortcut.

Clavis encodes the intent: **carry/extend**, not the keyboard modifier.

## Caveat

This transformation works cleanly only where there is a meaningful current subject and destination. It must not become a catch-all alternate-function marker. If a domain has no coherent ANCHOR/CARRY interpretation, star should be invalid there.

---

# 10. Removal: reserve a distinct ordinary-destructive feature

Navigation and deletion must not be neighboring meanings distinguished only by context.

A strong candidate is the right-side `-D` key.

## Proposed rule

**`-D` = REMOVE**

REMOVE means "remove the target selected by the rest of the command."

Examples conceptually:

- text + word + west + REMOVE -> delete previous word;
- text + line + east-boundary + REMOVE -> delete to line end;
- container + tab + REMOVE -> close current tab;
- container + window + REMOVE -> close current window.

This is attractive because `D` is naturally associated with delete, it sits outside the main direction diamond, the same scope system can be reused, and Class B destructive actions become physically recognizable.

REMOVE is not the same as the Delete key. An adapter may implement it with Backspace, Delete, a modified delete command, selection + Delete, a close-tab command, a close-window command, or an accessibility operation.

**Status:** strong candidate, but freeze it only after the named-action palette is laid out.

---

# 11. Guarded destruction: use escalation, not an ordinary brief

Class C commands include quit application, quit Plover, force quit, permanent deletion, sign out, restart, and shutdown.

They should not be reachable by accidentally adding one ordinary scope or direction key.

## Candidate physical guard

The far-right vertical pair:

**`-DZ`**

is a strong candidate for a **HARD/GUARDED** marker.

The logic would be:

```text
-D    ordinary local removal
-DZ   guarded/destructive escalation
```

The pair is physically distinct, intentionally farther from the direction diamond, and pressed as a deliberate same-column chord.

The guard never supplies a destructive action by itself. It only authorizes a command whose semantic definition is already Class C. A random guarded chord must still fail closed.

**Status:** provisional. The concept of a separate guard should be frozen; the exact `DZ` assignment can still change.

---

# 12. Quantity and repetition

## Repeating a command in Latched Control

The cheapest repetition is simply repeating the same payload stroke.

```text
enter Latched Control
next
next
next
exit
```

No count grammar is needed for a small number of repetitions.

## Explicit count

For larger counts, use:

```text
COUNT / COMMAND
```

where COUNT is expressed by the future Clavis number theory.

Examples semantically:

- 5 / next word;
- 3 / next tab;
- 8 / scroll line down.

The count is a separate prefix because counts are less frequent than ordinary one-step movement, keeping the main command uncluttered improves ergonomics, it avoids separate entries for "next 2," "next 3," etc., and it avoids requiring a physical number bar in the control grammar.

A dedicated "repeat previous control command" operation should exist, but its exact physical assignment belongs in the named-action palette.

---

# 13. What the feature map already generates

The notation below uses semantic feature names rather than final stenographic strings.

## Text navigation

| Meaning | Feature composition |
|---|---|
| previous character | GATE + CONTENT + scope0 + WEST |
| next character | GATE + CONTENT + scope0 + EAST |
| previous word | GATE + CONTENT + scope1 + WEST |
| next word | GATE + CONTENT + scope1 + EAST |
| line up | GATE + CONTENT + scope2 + NORTH |
| line down | GATE + CONTENT + scope2 + SOUTH |
| previous paragraph | GATE + CONTENT + scope3 + WEST |
| next paragraph | GATE + CONTENT + scope3 + EAST |
| document start | GATE + CONTENT + scope4 + WEST-BOUNDARY |
| document end | GATE + CONTENT + scope4 + EAST-BOUNDARY |

## Text selection

Add ANCHOR/CARRY:

| Meaning | Feature composition |
|---|---|
| select previous character | GATE + CONTENT + scope0 + WEST + ANCHOR |
| select next word | GATE + CONTENT + scope1 + EAST + ANCHOR |
| select line upward | GATE + CONTENT + scope2 + NORTH + ANCHOR |
| select to document end | GATE + CONTENT + scope4 + EAST-BOUNDARY + ANCHOR |

## Text deletion

Add REMOVE:

| Meaning | Feature composition |
|---|---|
| delete previous character | GATE + CONTENT + scope0 + WEST + REMOVE |
| delete next word | GATE + CONTENT + scope1 + EAST + REMOVE |
| delete to line start | GATE + CONTENT + scope2 + WEST-BOUNDARY + REMOVE |

## Container navigation

| Meaning | Feature composition |
|---|---|
| previous tab | GATE + CONTAINER + scope0 + WEST |
| next pane | GATE + CONTAINER + scope1 + EAST |
| previous window | GATE + CONTAINER + scope2 + WEST |
| next application | GATE + CONTAINER + scope3 + EAST |
| next workspace | GATE + CONTAINER + scope4 + EAST |
| first tab | GATE + CONTAINER + scope0 + WEST-BOUNDARY |
| last window | GATE + CONTAINER + scope2 + EAST-BOUNDARY |

## Container movement

Add ANCHOR/CARRY where meaningful:

| Meaning | Feature composition |
|---|---|
| move current tab next | GATE + CONTAINER + scope0 + EAST + CARRY |
| send current window to next workspace | GATE + CONTAINER + scope4 + EAST + CARRY |

The latter illustrates an important rule: the semantic adapter may have an **implicit carried subject** (the current window) while scope/direction specify the destination hierarchy. That behavior must be explicit and tested rather than inferred differently by each adapter.

---

# 14. Commands that should not be forced into the directional geometry

Not every useful command is directional.

Examples:

- undo;
- redo;
- copy;
- cut;
- paste;
- paste plain;
- save;
- open;
- new;
- print;
- find;
- replace;
- activate;
- cancel;
- toggle;
- screenshot;
- media play/pause;
- Plover lookup;
- Plover add translation.

They need a compact **named-action palette**.

That palette should still obey Clavis principles:

- related actions grouped spatially;
- opposites paired;
- lifecycle operations grouped;
- clipboard/editing operations grouped;
- application-independent semantics;
- no imitation of Ctrl/Cmd letters;
- frequent operations easier than rare ones;
- Class C actions guarded.

The unused lower-left combinations and unused right-side keys provide room for such palettes without disturbing the directional grammar.

That named-action palette should be the next physical-design layer.

---

# 15. Why Clavis should not mirror Ctrl/Alt/Command

A tempting design would define steno equivalents of Ctrl, Alt, Shift, Windows/Command, and arbitrary letters/keys.

That would allow every existing shortcut to be reproduced.

It should exist eventually as an interoperability escape hatch, but it should **not** be the normal Clavis grammar because:

1. the learner still has to memorize the original shortcut;
2. macOS/Windows/Linux differences leak into the theory;
3. application-specific bindings leak into the theory;
4. arbitrary modifier chords recreate the cognitive problem Clavis is intended to solve;
5. the same intention may require different physical shortcuts in different applications.

Clavis should make "next tab" easy to remember, not make "Ctrl+Tab" easy to emulate.

---

# 16. Rejected or deferred gate designs

## Star as gate

**Rejected.**

It is central and easy, but too valuable for ordinary theory functions and selection/extension.

## Number key as gate

**Rejected as a core requirement; possible optional hardware alias.**

It is an obvious alternate-layer marker, but some hobbyist writers omit the number bar and numbers deserve a stable theory.

## Outer-edge S/Z frame

**Deferred/rejected as primary.**

It is recognizable and leaves interior payload capacity, but forces both pinkies into every transient command.

## Dedicated two-stroke entry for every control command

**Rejected as the default.**

It provides enormous payload space and safety, but doubles stroke cost for isolated high-frequency commands. Latched mode already provides a multi-command layer when appropriate.

---

# 17. Hardware portability

The core map must work on professional stenotype machines, hobbyist steno boards, split-S layouts, QWERTY keyboards used as Plover machines with adequate rollover, and devices without a dedicated number bar.

Therefore:

- no core command may require a physical number bar;
- duplicated physical S keys must not be assumed logically distinct;
- the map should use standard logical steno keys rather than vendor-specific extras;
- optional hardware may provide aliases, but not change semantic meaning.

---

# 18. Ergonomic rules for later stroke assignment

## 18.1 Frequency before mnemonic cleverness

The most frequent actions should use fewer keys, avoid repeated pinky-heavy chords, avoid awkward same-finger vertical combinations where a simpler form exists, and remain available as single transient strokes.

## 18.2 Repeated control must get cheaper

Latched Control should remove gate overhead without changing command payload.

## 18.3 Opposites should be mirrored

Examples include west/east, north/south, start/end, previous/next, undo/redo, open/close, zoom in/out, and volume up/down.

## 18.4 Larger scope must never move backward in the scope ladder

If `A` means scope1 and `O` means scope2 in one family, an extension may not arbitrarily make `A` global and `O` local.

## 18.5 Unsupported combinations should stay invalid

Unused combinations are valuable future namespace.

Do not assign every possible chord merely because it exists. A sparse regular grammar is easier to learn and safer to extend than a maximally packed dictionary.

---

# 19. State-machine rules

## Writing State

- lexical strokes -> lexical translation;
- Gate + valid payload -> execute transient control command;
- Gate alone -> enter Latched Control;
- invalid would-be control payload -> no host action.

## Latched Control

- valid payload -> control action;
- payloads omit Gate;
- Gate alone -> return to Writing State;
- unknown payload -> fail closed;
- a future explicit CANCEL/RESET action -> return to safe known state.

## Guarded action pending

If Clavis later adopts confirmation for some Class C operations:

- first guarded command identifies the requested Class C action;
- confirmation must be deliberate;
- timeout/cancel returns to safe state;
- unrelated input never executes the action accidentally.

---

# 20. Stress test against the semantic inventory

## Strong fit

The feature map already fits:

- character/word/line/paragraph/document movement;
- movement with selection;
- deletion using the same text scope;
- previous/next/first/last tabs;
- panes/windows/applications/workspaces as increasing container scope;
- moving tabs;
- sending windows between workspaces;
- UI directional movement;
- scrolling;
- history previous/next;
- repeated movement in latched mode;
- explicit quantities.

## Requires the next named-action palette

- undo/redo;
- copy/cut/paste;
- save/open/new/print;
- find/replace;
- activate/cancel/toggle;
- close/reopen where not represented cleanly by REMOVE;
- window minimize/maximize/restore;
- window placement;
- zoom;
- screenshots;
- audio/media;
- system launcher;
- Plover tools;
- Clavis help/reset;
- application launch by name.

## Requires adapter policy rather than more theory

- application-specific shortcut differences;
- macOS versus Windows/Linux modifier differences;
- whether a pane exists;
- whether tabs can be reordered;
- whether workspaces are indexed;
- whether quarter-window placement is supported;
- whether UI selection extension exists.

This is desirable. A theory should not grow new chords to compensate for implementation differences that belong in adapters.

---

# 21. Decisions recommended to freeze now

These have enough independent support to treat as design invariants unless testing exposes a serious problem.

1. **Semantic commands remain separate from host shortcuts.**
2. **`STPH` top rail is reserved as the Control Gate.**
3. **The right `R/P/B/G` diamond is the universal direction geometry.**
4. **`FR` and `LG` are west/east boundary forms.**
5. **The vowel bank is an ordinal small-to-large scope ladder:** `none -> A -> O -> E -> U`.
6. **Text uses:** `character -> word -> line -> paragraph -> document`.
7. **Containers use:** `tab -> pane -> window -> application -> workspace`.
8. **Transient and latched control use identical payloads.**
9. **Count is a separate prefix using the eventual Clavis number system.**
10. **Unknown control forms fail closed.**

---

# 22. Decisions recommended to keep provisional

1. **Lower-left distance gradient for command families:** `none / R / W / K` as content/interface/container/system.
2. **Star as ANCHOR/CARRY across multiple domains.** Text selection is strong; non-text carry semantics need complete stress testing.
3. **`-D` as generic REMOVE.** Strong mnemonic and structural fit, but it must be considered alongside the action palette.
4. **`-DZ` as the Class C guard.** The guard concept is firm; the exact chord can still change.

---

# 23. Next design step

The next layer should design the **named-action palette**.

It should place non-directional primitives into a small number of memorable spatial families, especially:

1. **editing strip**
   - undo
   - redo
   - cut
   - copy
   - paste
   - paste plain

2. **interaction strip**
   - activate
   - cancel
   - toggle
   - context menu
   - expand
   - collapse

3. **document/search strip**
   - new
   - open
   - save
   - save as
   - print
   - find
   - replace

4. **container-state strip**
   - close
   - reopen
   - minimize
   - maximize
   - restore
   - place

5. **system/Plover strip**
   - launcher
   - screenshot
   - notifications
   - clipboard history
   - media
   - Plover lookup
   - suggestions
   - add translation
   - configure

The palette should be designed against the **remaining physical keyspace**, not by assigning mnemonic initials one at a time.

Only after that palette survives the semantic-inventory stress test should Clavis generate concrete Plover dictionary entries.

---

# Research references

## Plover and steno

- Plover Wiki, Beginner's Guide — conventional steno layout and Plover operation:  
  https://plover.wiki/index.php/Beginner%27s_Guide

- Learn Plover, Steno Order — `STKPWHRAO*EUFRPBLGTSDZ`:  
  https://opensteno.org/learn-plover/lesson-2-steno-order.html

- Learn Plover, Cheat Sheet — number system, fingerspelling, layout:  
  https://opensteno.org/learn-plover/appendix-cheat-sheet.html

- Plover Wiki, Supported Hardware — hardware variation including writers without a dedicated number bar:  
  https://plover.wiki/index.php/Supported_hardware

## Existing steno computer-control approaches

- Di Does Digital, steno dictionaries — `STPH` navigation, `-R/-P/-B/-G` directional mapping, tab/window/application families:  
  https://github.com/didoesdigital/steno-dictionaries

- Lapwing — movement commands, `STPH*` selection, modal movement:  
  https://github.com/robertmassaioli/lapwing

- Kaoffie, Plover Modal Dictionary — persistent and one-translation modal behavior:  
  https://github.com/Kaoffie/plover_modal_dictionary

## Layered keyboard design

- Miryoku — minimal orthogonal layered keyboard system:  
  https://github.com/manna-harbour/miryoku

- Miryoku Babel — navigation, clipboard, mouse, media, and layer organization:  
  https://github.com/manna-harbour/miryoku_babel

- QMK One Shot Keys — transient versus persistent layer/modifier behavior:  
  https://docs.qmk.fm/one_shot_keys

- QMK Tap Dance — layer movement/toggle patterns and multi-purpose controls:  
  https://docs.qmk.fm/features/tap_dance

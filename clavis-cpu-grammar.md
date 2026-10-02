# Clavis Computer-Control Grammar

**Status:** Research draft v0.1  
**Scope:** Behavioral and mnemonic rules only. This document intentionally does **not** assign final stenographic outlines or physical keystrokes.

## Purpose

Clavis is intended to treat computer control as a first-class part of stenographic theory rather than as a collection of after-the-fact hotkeys. The control system therefore needs the same properties expected of the word theory: regularity, composability, predictability, low memorization burden, and room for systematic extension.

The central design decision is that Clavis should express **what the user means**, not the host computer's keyboard shortcut. A Clavis command should mean things such as “move one word backward,” “select to the end of the line,” “switch to the next tab,” or “move this window to the next desktop.” Windows, macOS, Linux, a browser, or an individual application may implement those meanings with different shortcuts. Those implementation details belong below the theory layer.

---

## Research findings

### 1. Plover already has the necessary low-level primitives, but its default command vocabulary is not a general control theory

Plover can emit arbitrary keyboard combinations and can invoke Plover control commands and plugin commands. Its stock `commands.json` includes cursor movement, word movement, browser-history movement, Tab, Alt-Tab, Delete, Backspace, Enter, and Plover on/off operations.

That proves that general computer control is technically compatible with Plover. However, the stock command set is a small collection of individual mappings rather than an extensible grammar. Plover's own documentation also warns that emitted keyboard shortcuts are not undoable in the way translated text is. That makes collision avoidance and explicit control-mode boundaries especially important.

### 2. Di's navigation dictionaries show that reusable command families are much easier to learn than isolated briefs

Di's Plover dictionaries organize cursor movement around a consistent directional scheme, then systematically add variants for moving by word, selecting while moving, deleting by character/word/line, switching tabs, switching windows, switching applications, and browser navigation.

This is much closer to what Clavis needs than independent mnemonic briefs. The important lesson is not the particular outlines; it is the reuse of the **same direction vocabulary and the same transformation rules** across related operations.

The limitation is that the system still grows outward from particular host shortcuts and contains platform-specific assumptions. Clavis should preserve the systematic families while moving the shortcut mapping beneath a semantic command layer.

### 3. Modal dictionaries solve an important namespace problem

`plover_modal_dictionary` demonstrates that a Plover dictionary can be temporarily activated, kept active for repeated commands, exited explicitly, exited after one translation, or exited after a mismatch. Aerick's/Lapwing's semi-modal movement dictionary applies the same idea specifically to cursor movement.

This is important because Clavis should not force every computer-control operation to compete permanently with every word outline. A control namespace can be deliberately entered, used, and left.

### 4. Large personal command dictionaries demonstrate usefulness but also the danger of mnemonic sprawl

Paul Fioravanti's command dictionaries cover named actions, application activation, browsers, navigation, Plover commands, switching, Vim, media, and window management. They also introduce platform-aware mappings where the same semantic action can emit different shortcuts on macOS versus Windows/Linux.

This demonstrates both sides of the problem:

- stenographic computer control can cover a very large portion of daily computing;
- if each action receives its own mnemonic outline, the result eventually becomes another large dictionary to memorize.

Clavis should therefore use **named semantic actions internally**, but generate user-facing commands from a smaller orthogonal grammar wherever possible.

### 5. Plover-Vim demonstrates the value of composition and dense single-stroke encoding

`plover-vim` generates mostly single-stroke forms for combinations that would otherwise require several Vim commands. Its explicit goals include extensibility, customizability, and eliminating repeated transitions between writing and editing modes.

Clavis should borrow the compositional principle without inheriting Vim's application-specific vocabulary. General computer control needs a smaller set of concepts that work in ordinary text fields, browsers, office applications, file managers, and the operating system.

### 6. Modal editors offer a strong model for orthogonality

Vim, Kakoune, and Helix demonstrate that editing becomes learnable when movement, selection, targets, and actions combine predictably. Kakoune and Helix are especially useful conceptually because they emphasize simple operations, orthogonal modes, and selection/action composition rather than a long list of unrelated shortcuts.

Clavis should adopt that **grammar mindset**, not necessarily any editor's specific command order.

### 7. Talon offers the right abstraction boundary for cross-platform control

Talon separates user-facing commands from context-specific implementations. The same semantic action can behave differently depending on operating system, application, active mode, or application tag.

Clavis should use the same architectural principle: **semantic command first; platform/application adapter second**.

### 8. Mainstream operating systems already reveal a stable conceptual vocabulary

Windows and major browsers expose recurring concepts even when their physical shortcuts differ:

- character, word, line, page, and document movement;
- selection as movement plus extension;
- previous/next tab;
- previous/next application or window;
- browser back/forward;
- virtual-desktop switching;
- spatial window placement;
- new/open/close/reopen;
- find, save, undo, redo, copy, cut, paste;
- moving focus among controls.

These concepts, rather than Ctrl/Alt/Windows/Command key combinations, should form Clavis's stable vocabulary.

---

# Proposed Clavis control rules

## Rule 1 — Clavis describes intent, never a host shortcut

The theory layer must define semantic commands such as:

- `TEXT MOVE WORD PREVIOUS`
- `TEXT EXTEND LINE END`
- `TEXT DELETE WORD PREVIOUS`
- `TAB SWITCH NEXT`
- `WINDOW PLACE LEFT`
- `DESKTOP SWITCH NEXT`

Those are conceptual examples, not literal Clavis syntax.

A Windows adapter, macOS adapter, Linux adapter, browser adapter, or application adapter decides how to execute the semantic action.

**Consequence:** a learner memorizes one Clavis concept, not separate Windows, Chrome, Word, VS Code, and macOS shortcuts.

---

## Rule 2 — The lexical namespace and control namespace must be separated by design

Computer control may not be squeezed into whatever outlines remain after the word dictionary is complete.

The final theory must reserve a recognizable structural namespace for control from the beginning. A control stroke or control sequence must therefore be distinguishable from ordinary lexical writing by rule, not merely by dictionary priority.

The exact physical marker is deliberately unspecified in this draft.

**Goal:** seeing or feeling a control outline should make it obvious that it is a command and not a word brief.

---

## Rule 3 — Clavis has a normal writing state and a deliberate control state

Normal operation is **Writing State**: strokes translate as language.

Computer commands are available through two forms:

1. **Transient Control** — enter control interpretation for one command, then return automatically to writing.
2. **Latched Control** — remain in control interpretation for a sequence of navigation/editing commands until explicitly leaving it.

Transient Control should be the ordinary choice for an isolated command. Latched Control should exist for editing sessions where repeated movement, selection, deletion, tabbing, or window management would otherwise require repeated control-entry strokes.

This combines the safety of an explicit namespace with the efficiency demonstrated by modal Plover dictionaries.

---

## Rule 4 — Unknown control input fails closed

When Clavis is deliberately interpreting control input, an unknown control command must **not** silently become an arbitrary operating-system shortcut.

The preferred behavior is:

- perform no external action;
- provide visible or audible feedback when practical;
- remain predictable about whether Control stays active or exits.

Irreversible commands make this rule particularly important because Plover-emitted shortcuts cannot simply be “un-stroked.”

---

## Rule 5 — Every command is built from a small set of semantic dimensions

Conceptually, control commands are composed from these dimensions:

**Domain + Action + Target + Qualifier**

Not every command needs every dimension, and these are semantic slots rather than a required multi-stroke sequence.

### Domains

The initial domains should be:

- **Text** — cursor movement, selection, deletion, insertion-related actions.
- **Focus** — move keyboard/UI focus among controls, fields, panes, and dialogs.
- **Document** — save, open, new, close, find, print, and similar document-level actions.
- **Tab** — create, close, reopen, select, reorder, and cycle tabs.
- **Web** — browser history, address/search focus, reload, page navigation, zoom.
- **Window** — focus, close, minimize, maximize/restore, snap/place, and move windows.
- **Application** — switch, activate, launch, and quit applications.
- **Desktop** — switch, create, close, and move windows among virtual desktops/workspaces.
- **System** — media, audio, display, screenshots, lock, notifications, and other OS-level actions.
- **Plover** — engine control, dictionary tools, lookup, suggestions, and Clavis/Plover state.

Domains should be few and durable. Application-specific commands are extensions, not additions to the core grammar unless the concept generalizes.

---

## Rule 6 — Direction is universal

Clavis should have one stable directional model:

- previous / backward;
- next / forward;
- up;
- down;
- left;
- right;
- start;
- end.

The same conceptual direction must retain the same identity everywhere.

Use **left/right/up/down** when the target is genuinely spatial. Use **previous/next** when traversing an ordered collection such as tabs, applications, history entries, or desktops. Use **start/end** for boundaries.

This distinction avoids learning that “right” sometimes means spatial right, sometimes next tab, and sometimes forward in browser history.

---

## Rule 7 — Text movement uses a fixed scale ladder

Text navigation should use the following ordered unit ladder:

1. character;
2. word;
3. line;
4. paragraph;
5. page/screen;
6. document.

Moving at a larger scale must be expressed as the same movement rule with a different unit, not as an unrelated brief.

Examples conceptually:

- move one character backward;
- move one word backward;
- move to line start;
- move one paragraph forward;
- move one page down;
- move to document end.

The final stroke system should make neighboring scales feel related.

---

## Rule 8 — Selection is extended movement, not a separate navigation vocabulary

Anything that can move the text cursor should, where the host application supports it, have a corresponding **extend selection** form.

Therefore:

- move character next → extend selection character next;
- move word previous → extend selection word previous;
- move line end → extend selection to line end;
- move document start → extend selection to document start.

This mirrors the conceptual regularity of Shift-modified navigation on ordinary keyboards without requiring the user to think in terms of Shift.

---

## Rule 9 — Deletion reuses the same target system

Deletion should not have an independent collection of memorized commands.

The same units and directions used for movement and selection become deletion targets:

- delete character previous;
- delete character next;
- delete word previous;
- delete word next;
- delete to line start;
- delete to line end;
- delete selection.

Where an application lacks a native semantic equivalent, the adapter may implement the result as a short sequence of ordinary shortcuts.

---

## Rule 10 — Ordered containers share one traversal grammar

Tabs, windows, applications, desktops, search results, dialog controls, and other ordered collections should all reuse:

- previous;
- next;
- first;
- last;
- choose by number when a stable numbered form exists.

The noun changes; the traversal concept does not.

Conceptually:

- tab next;
- window previous;
- application next;
- desktop previous;
- focus next;
- search-result next.

---

## Rule 11 — Switching an object and moving an object are different actions

Clavis must clearly distinguish **focus/switch** from **move/send/place**.

Examples:

- `DESKTOP SWITCH NEXT` changes which desktop the user is viewing.
- `WINDOW SEND DESKTOP NEXT` moves the current window to another desktop.
- `WINDOW SWITCH NEXT` changes the focused window.
- `WINDOW PLACE LEFT` spatially positions the current window.

This distinction should be visible in the grammar so that a command cannot be misremembered as the other.

---

## Rule 12 — Lifecycle actions are universal verbs

Reusable lifecycle actions should include:

- new/create;
- open;
- close;
- reopen/restore;
- save;
- quit;
- refresh/reload.

They combine with targets when meaningful:

- new tab;
- close tab;
- reopen tab;
- new window;
- close window;
- close document;
- save document;
- quit application;
- refresh page.

The final theory should prefer the same action component whenever the semantic action is genuinely the same.

---

## Rule 13 — Common editing actions are primitives

The following operations are common enough across applications to deserve stable first-class semantics:

- undo;
- redo;
- copy;
- cut;
- paste;
- select all;
- find;
- find next;
- find previous;
- replace;
- save;
- open;
- new;
- close;
- escape/cancel;
- confirm/activate.

These should not be defined as “send Ctrl+Z,” “send Ctrl+C,” and so on. The adapter may use those shortcuts, but the theory names the action.

---

## Rule 14 — “Opposite” operations should be systematic pairs

Where two commands are natural opposites, their Clavis forms should differ by a single systematic feature rather than unrelated mnemonics.

Important pairs include:

- previous / next;
- backward / forward;
- start / end;
- up / down;
- left / right;
- undo / redo;
- open / close;
- minimize / restore;
- zoom in / zoom out;
- volume up / volume down;
- create / destroy or close, where appropriate.

This is a design constraint on the later stroke theory.

---

## Rule 15 — Repetition and quantity are grammatical, not duplicated dictionary entries

The system should eventually support a general way to repeat or quantify a control action.

Examples conceptually:

- move word next × 3;
- tab next × 2;
- page down × 4;
- repeat previous control action.

The theory should not require separate memorized entries for “next tab,” “next two tabs,” and “next three tabs.”

The exact quantity mechanism is intentionally deferred until stroke design.

---

## Rule 16 — Frequent commands may receive optimized forms, but only as aliases

Clavis may eventually provide exceptionally compact forms for very common commands such as undo, copy, paste, save, tab-next, or escape.

However, an optimized brief must be an **alias of a rule-generated semantic command**, not the only way the command is defined.

This preserves learnability:

1. a learner can derive the regular form;
2. an expert can optionally learn the faster alias;
3. the underlying meaning remains documented and testable.

This rule should also govern the future word theory: briefs accelerate the rules; they do not replace the rules.

---

## Rule 17 — Context may change implementation but not meaning

Clavis may detect or be configured for:

- operating system;
- application;
- browser versus editor versus terminal;
- focused UI context;
- available plugins or automation backends.

Context can change **how** a semantic action is executed, but should not casually change **what the same Clavis command means**.

For example, `DOCUMENT SAVE` may map to different underlying shortcuts in different applications. It still means save the current document.

If an application intentionally requires a different semantic behavior, that should be an explicit application extension rather than an invisible reinterpretation of a core command.

---

## Rule 18 — Platform-specific concepts live in adapters or extensions

Clavis Core should contain concepts that generalize well. Platform-specific features belong in extensions or adapters.

Examples:

- Windows Snap layouts;
- macOS Mission Control conventions;
- Linux window-manager workspaces;
- browser-specific tab-group operations;
- IDE-specific command palettes;
- media-application controls.

The core may expose general semantics such as `WINDOW PLACE LEFT` or `DESKTOP NEXT`; a platform adapter determines the best native realization.

---

## Rule 19 — Destructive commands require stronger differentiation

Commands with large or difficult-to-reverse consequences should not be one easy misstroke away from ordinary navigation.

Examples include:

- quit application;
- close an entire window containing multiple tabs;
- close/delete a virtual desktop if the platform's behavior is surprising;
- permanent file deletion;
- shutdown/restart/sign out;
- destructive application-specific operations.

The later stroke theory should give these commands either:

- a deliberately stronger structural marker;
- a two-stage form;
- or confirmation when technically practical.

Routine reversible editing should remain fast; destructive system control should favor safety.

---

## Rule 20 — Control commands must not corrupt Plover's linguistic formatting state

Cursor motion and computer commands should not accidentally cause Plover to capitalize, attach, or space later text incorrectly.

Plover's translation language explicitly documents formatting cancellation around cursor movement. Clavis implementations must treat formatting-state preservation as part of correctness, not as an optional polish item.

This will require dedicated tests once dictionaries and plugins exist.

---

## Rule 21 — Clavis should expose a compact “command surface,” not every hotkey in existence

A keyboard shortcut is not automatically worthy of a Clavis core command.

A command belongs in the core when at least one of the following is true:

- it is used frequently in general computing;
- it composes cleanly from existing semantic dimensions;
- it removes a recurring need to leave the steno keyboard;
- it generalizes across applications or operating systems;
- it is necessary to operate the Clavis/Plover environment itself.

Rare application-specific shortcuts belong in extension dictionaries or user configuration.

---

## Rule 22 — The system should be discoverable from its own rules

A user who knows the domain, action, target, and direction rules should be able to predict an unfamiliar command with a high probability of success.

Therefore, future command design should be tested with questions such as:

- If a user knows “move word previous,” can they derive “select word previous”?
- If they know “tab next,” can they derive “desktop next”?
- If they know “window place left,” can they derive “window place right”?
- If they know “document start,” can they derive “document end”?

If the answer is repeatedly “no, memorize another brief,” the theory is becoming too dictionary-driven.

---

# Proposed conceptual model

The rules above can be summarized as three reusable structures.

## A. Text ladder

**Action** × **unit** × **direction/boundary**

Actions:

- move;
- extend/select;
- delete.

Units:

- character;
- word;
- line;
- paragraph;
- page;
- document.

Directions/boundaries:

- previous/backward;
- next/forward;
- up/down where meaningful;
- start;
- end.

This should generate most ordinary text navigation and editing without independent memorization.

## B. Container ladder

**Action** × **container** × **direction/index**

Actions:

- switch/focus;
- create;
- close;
- reopen;
- move/send where meaningful.

Containers:

- focusable control;
- tab;
- window;
- application;
- desktop/workspace.

Direction/index:

- previous;
- next;
- first;
- last;
- number, when supported.

## C. Named universal actions

Some actions do not benefit from a target ladder and should remain stable primitives:

- undo/redo;
- copy/cut/paste;
- escape/cancel;
- confirm/activate;
- find/replace;
- save;
- command/search palette;
- screenshot;
- lock;
- media play/pause and volume.

Even these should use paired/opposite rules wherever possible.

---

# Recommended implementation architecture later

This document does not yet specify code, but the rules strongly suggest a future architecture with four layers:

1. **Clavis theory layer** — interprets steno structures as semantic commands.
2. **Semantic command layer** — canonical operations such as `text.move(word, previous)` or `tab.switch(next)`.
3. **Context adapter layer** — Windows/macOS/Linux/application/browser implementations.
4. **Execution layer** — Plover key combinations, Plover commands, plugins, scripts, accessibility APIs, or other mechanisms.

This separation would allow the rules to remain stable even if a particular shortcut changes or a better execution mechanism becomes available.

---

# Design constraints for the future stroke theory

When actual outlines are designed, they should satisfy these constraints:

1. Control outlines are structurally recognizable as control.
2. Related commands share visible/physical features.
3. Opposites differ systematically.
4. Direction is reused everywhere.
5. Text scale is reused everywhere.
6. Selection is a modification of movement.
7. Deletion is a modification of the same target grammar.
8. Container traversal is reused across tabs, windows, apps, and desktops.
9. Transient and latched control are both available.
10. Common optimized briefs are optional aliases, never exceptions that replace the regular form.
11. Destructive operations are harder to trigger accidentally than ordinary movement.
12. Platform hotkeys never determine the mnemonic structure of Clavis itself.

---

# What Clavis should avoid

Clavis should explicitly avoid:

- assigning an arbitrary brief to every Ctrl/Alt/Windows shortcut;
- mirroring QWERTY modifier combinations as the user's mental model;
- allowing application-specific exceptions to consume the core namespace;
- giving tabs, windows, applications, and desktops unrelated next/previous systems;
- creating separate vocabularies for moving, selecting, and deleting the same text units;
- relying entirely on permanent globally active command strokes;
- silently changing the meaning of a core command by application;
- optimizing for minimum strokes at the expense of predictability;
- treating computer control as secondary to lexical theory.

---

# Research references

- Plover 5 translation language — keyboard shortcuts, modifiers, repeat, formatting cancellation, and Plover control commands:  
  https://plover.readthedocs.io/en/v5.0.0/translation_language.html

- Plover default `commands.json`:  
  https://github.com/openstenoproject/plover/blob/main/plover/assets/commands.json

- Kaoffie, `plover_modal_dictionary` — transient and persistent modal dictionaries:  
  https://github.com/Kaoffie/plover_modal_dictionary

- Di Does Digital, `steno-dictionaries` — navigation, tab/window/application switching, and computer powerups:  
  https://github.com/didoesdigital/steno-dictionaries

- Aerick / Lapwing dictionaries — semi-modal movement pattern:  
  https://github.com/aerickt/steno-dictionaries  
  https://github.com/robertmassaioli/lapwing

- Paul Fioravanti, `steno-dictionaries` — large semantic/action-oriented command collection and platform-aware command mappings:  
  https://github.com/paulfioravanti/steno-dictionaries/blob/main/dictionaries/commands.md

- Josiah Tan, `plover-vim` — compositional generation of mostly single-stroke Vim commands:  
  https://github.com/Josiah-tan/plover-vim

- Helix documentation — modal and selection-first editing:  
  https://docs.helix-editor.com/master/usage.html

- Kakoune design — orthogonal commands and predictable editing grammar:  
  https://github.com/mawww/kakoune/blob/master/doc/design.asciidoc

- Talon documentation — semantic commands, modes, tags, and context-specific implementations:  
  https://talonvoice.com/docs/reference/guide.extending.html  
  https://talonvoice.com/docs/reference/guide.language.html

- Microsoft Windows keyboard shortcuts and multitasking documentation:  
  https://support.microsoft.com/en-us/windows/keyboard-shortcuts-in-windows-dcc61a57-8ff0-cffe-9796-cb9706c75eec  
  https://support.microsoft.com/en-us/windows/how-to-multitask-in-windows-b4fa0333-98f8-ef43-e25c-06d4fb1d6960

- Google Chrome keyboard shortcuts:  
  https://support.google.com/chrome/answer/157179

- Mozilla Firefox keyboard shortcuts:  
  https://support.mozilla.org/kb/keyboard-shortcuts-perform-firefox-tasks-quickly

---

# Next research step

Before assigning any actual Clavis strokes, the next design pass should build a **semantic command inventory** from real daily-computing workflows and reduce it to the smallest orthogonal set possible. Only after that inventory stabilizes should physical steno features be assigned to domains, actions, units, directions, mode entry, repetition, and safety markers.

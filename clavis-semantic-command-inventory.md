# Clavis Semantic Command Inventory

**Status:** Research draft v0.1  
**Scope:** Canonical computer-control meanings only. This document does **not** assign stenographic outlines, physical key combinations, or final Clavis syntax.

## Purpose

The first Clavis research layer established the control grammar: Clavis should describe user intent rather than host shortcuts, should reserve a control namespace from the beginning, and should use a small set of orthogonal dimensions rather than a large collection of mnemonic briefs.

This second layer defines the **minimum semantic command surface** that Clavis should eventually be able to express.

The inventory is intentionally not a list of every shortcut offered by Windows, macOS, Linux, browsers, office suites, editors, or Plover. It is a normalized set of meanings from which those routine actions can be expressed predictably.

The design objective is:

> A Clavis user should learn a small number of reusable actions, targets, directions, scales, and qualifiers, then derive most computer-control commands from them.

---

# 1. Research conclusions that affect the inventory

## 1.1 General-purpose computer control has several stable semantic families

Across Windows, macOS, GNOME/KDE, browsers, office software, and editors, the same conceptual families recur even when the actual shortcuts differ:

- move through text;
- extend a selection while moving;
- delete around the cursor;
- undo/redo and clipboard editing;
- find and move through matches;
- move keyboard focus through UI controls;
- activate, cancel, toggle, expand, and collapse controls;
- traverse tabs, windows, applications, and workspaces;
- create/close/reopen common containers;
- navigate backward/forward through location history;
- scroll and zoom the view;
- place or move windows;
- launch/switch/quit applications;
- invoke a small set of system functions.

This is strong evidence that Clavis can normalize a large amount of computing behind a relatively small semantic vocabulary.

## 1.2 Text cursor movement and viewport movement must be separate

Existing shortcuts often overload Page Up, Page Down, Home, End, and arrow concepts.

In one context an action moves the insertion point. In another it only scrolls the visible page. In another it changes the selected object.

Clavis therefore needs separate **TEXT**, **VIEW**, and **UI** concepts:

- **TEXT** changes the insertion point or text selection.
- **VIEW** changes what is visible without implying text-cursor movement.
- **UI** changes which control or object is focused.

An adapter may implement two of these with the same physical shortcut in a particular application, but Clavis must not make them the same semantic command.

## 1.3 UI focus deserves first-class treatment

Keyboard-only operation depends on more than Tab and Enter. Mainstream systems consistently expose the concepts of:

- next/previous focus;
- directional movement within a group;
- activate/confirm;
- cancel/escape;
- toggle;
- expand/collapse;
- context menu.

Clavis should model those meanings directly instead of forcing the user to memorize literal key names.

## 1.4 Containers share a common traversal grammar

Tabs, top-level windows, applications, workspaces, panes, search matches, and focusable controls repeatedly use the same relationships:

- previous;
- next;
- first;
- last;
- numbered/indexed target where available.

Clavis should reuse these qualifiers rather than invent a separate next/previous vocabulary for each domain.

## 1.5 Existing Plover systems prove that modal control is practical

Plover's built-in commands, Di's navigation/tabbing dictionaries, Lapwing's modal movement, Kaoffie's modal-dictionary plugin, and large personal command dictionaries all demonstrate that steno can perform significant computer control.

Their main limitation for Clavis is not technical capability. It is **semantic regularity**: many systems ultimately encode host shortcuts or accumulate application-specific mnemonic briefs.

Clavis should preserve the efficiency while moving the user's mental model one level upward.

---

# 2. Canonical command shape

The inventory assumes the conceptual structure established in the first research layer:

**Domain + Action + Target + Qualifier**

These are semantic dimensions, not a commitment to stroke order or multi-stroke syntax.

Examples of canonical meanings:

- TEXT + MOVE + WORD + PREVIOUS
- TEXT + EXTEND + LINE_BOUNDARY + END
- TEXT + DELETE + WORD + NEXT
- VIEW + SCROLL + PAGE + DOWN
- UI + FOCUS + CONTROL + NEXT
- TAB + SWITCH + TAB + NEXT
- WINDOW + PLACE + HALF + LEFT
- WORKSPACE + SWITCH + WORKSPACE + NEXT

The final implementation may encode several dimensions simultaneously in one stroke.

---

# 3. Shared qualifiers

These concepts should have one stable identity wherever they appear.

## Direction and order

- previous
- next
- backward
- forward
- up
- down
- left
- right
- start
- end
- first
- last
- current

Use **previous/next** for ordered collections.  
Use **left/right/up/down** for genuinely spatial relationships.  
Use **backward/forward** for text-relative or history-relative direction where appropriate.  
Use **start/end** for boundaries.

## Quantity

The semantic model should allow:

- one;
- an explicit count;
- repeat previous control action.

Quantity must be grammatical rather than represented by independent dictionary entries.

## Direct addressing

Where a platform or adapter supports it, containers may additionally accept:

- index/number;
- stable configured name or identifier.

Direct addressing is optional capability; previous/next traversal is the portable baseline.

---

# 4. Core command families

## 4.1 CONTROL — Clavis control state

These commands operate Clavis itself rather than the host application.

### Required

- enter transient control
- enter latched control
- leave control
- cancel current control sequence
- repeat previous control action
- apply quantity/count to an action

### Strongly recommended

- show control help / command reference
- report current control state
- reset control state to known-safe writing state

Unknown control input must continue to follow the first-layer rule: fail closed rather than emitting an arbitrary shortcut.

---

## 4.2 TEXT — insertion point and text selection

TEXT commands affect editable text, not merely what is visible.

### Movement units

Core units:

- character
- word
- line
- paragraph
- document

Core relationships:

- character: previous / next
- word: previous / next
- line: up / down
- paragraph: previous / next
- line boundary: start / end
- document boundary: start / end

### Selection

Selection is **extended movement**, not a separate navigation theory.

Every supported TEXT MOVE operation should have a corresponding TEXT EXTEND form where the application permits it.

Therefore the generated family includes meanings such as:

- extend character previous/next;
- extend word previous/next;
- extend line up/down;
- extend paragraph previous/next;
- extend to line start/end;
- extend to document start/end.

Additional required selection actions:

- select all
- clear/collapse selection without deleting it, where reliably implementable

### Deletion

Required:

- delete character previous
- delete character next
- delete word previous
- delete word next
- delete to line start
- delete to line end
- delete current selection

Larger deletion units such as paragraph or document should normally be expressed through selection plus deletion rather than dedicated primitive commands.

### Text breaks

Required:

- insert ordinary paragraph/newline
- insert soft/line break where the application distinguishes it

These are semantic text actions, not literal Enter-key commands.

### Derived convenience operations

The semantic layer may expose derived operations such as:

- select current word
- select current line
- select current paragraph

These should be compositions of the regular movement/extension model whenever possible, not independent theory exceptions.

### Deferred from core

These are useful but should initially live in editor or application extensions:

- sentence navigation;
- transpose characters/words;
- multi-cursor editing;
- structural selection expansion;
- code folding;
- indentation/outdentation;
- editor-specific line duplication/movement.

---

## 4.3 EDIT — reversible editing primitives

These actions generalize extremely well across applications and deserve stable first-class semantics.

### Required

- undo
- redo
- cut
- copy
- paste
- select all

### Strongly recommended

- paste as plain/unformatted text
- repeat last application edit, when a reliable application capability exists

Clipboard history is treated as a SYSTEM facility rather than a basic EDIT primitive because support varies by platform.

---

## 4.4 SEARCH — find, replace, and match traversal

### Required

- open find
- next match
- previous match
- close/cancel find

### Strongly recommended

- open replace
- replace current
- replace all, guarded because of scope
- focus search field when a separate field exists

Search-match traversal reuses previous/next rather than defining new directional concepts.

Application-specific "go to symbol," "go to line," command palettes, and project search should be extensions unless a later research pass establishes a truly general abstraction.

---

## 4.5 VIEW — scrolling and visual scale

VIEW affects what the user can see and does not imply moving the text insertion point.

### Scrolling

Required:

- scroll line up/down
- scroll page/screen up/down
- scroll to view start/end

Strongly recommended:

- horizontal scroll left/right where the context supports it

### Zoom

Required:

- zoom in
- zoom out
- reset zoom

### Display state

Strongly recommended:

- toggle full screen

A context adapter must decide whether "view start/end" means page, document surface, list, terminal buffer, or other currently viewed content.

---

## 4.6 UI — keyboard focus and control activation

This domain is essential for operating dialogs, forms, menus, ribbons, browser chrome, settings, and applications without leaving the steno keyboard.

### Focus traversal

Required:

- focus next control
- focus previous control

Strongly recommended:

- focus next region/group
- focus previous region/group

### Directional UI navigation

Required:

- move focus/selection up
- move focus/selection down
- move focus/selection left
- move focus/selection right

These represent arrow-like navigation inside lists, menus, radio groups, trees, grids, and similar controls.

### Activation and cancellation

Required:

- activate/confirm focused control
- cancel/escape current UI state

### Stateful controls

Strongly recommended:

- toggle focused control
- expand focused item
- collapse focused item
- open context menu for focused item

### Why these are semantic rather than literal keys

The correct implementation of "activate" may be Enter, Space, an accessibility action, or an application command. "Toggle" and "activate" are not always the same operation. Clavis should preserve that distinction.

---

## 4.7 LOCATION — navigation history and location fields

Browsers, file managers, settings applications, help viewers, and many document systems share a navigation-history concept.

### Required

- go back
- go forward

### Strongly recommended

- focus current location/address field
- go up one hierarchical level where the context has hierarchy
- go home/root where a stable meaning exists
- reload/refresh current location

The adapter determines whether the location is a URL, folder path, settings page, help page, or comparable navigable surface.

---

## 4.8 DOCUMENT — document lifecycle

### Required

- new document
- open document
- save document
- close document
- print document

### Strongly recommended

- save as
- reopen/recover most recently closed document where reliably supported

"Close document" is intentionally distinct from "close tab" and "close window." In some applications they map to the same implementation; in others they do not.

---

## 4.9 TAB — generic tabbed-container control

Tabs are now common in browsers, editors, terminals, file managers, settings interfaces, and document applications.

### Required

- switch previous tab
- switch next tab
- create new tab
- close current tab
- reopen most recently closed tab where supported

### Strongly recommended

- switch first tab
- switch last tab
- switch to indexed tab
- move current tab previous
- move current tab next

### Extension candidates

- duplicate tab
- pin/unpin tab
- mute tab
- tab groups
- close other tabs
- close tabs to one side

These are useful but not sufficiently universal for the minimum core.

---

## 4.10 PANE — sub-window work areas

Panes occur in terminals, IDEs, browsers, file managers, and some office applications, but the model is less universal than tabs.

### Strongly recommended portable subset

- focus next pane
- focus previous pane
- close current pane

### Extension candidates

- split horizontally
- split vertically
- move pane
- resize pane
- focus pane by direction
- focus pane by index

PANE should exist in the semantic architecture from the start even if its first implementation is modest, preventing future pane control from being squeezed into unrelated tab/window commands.

---

## 4.11 WINDOW — top-level window control

### Focus/traversal

Required:

- switch previous window
- switch next window

Strongly recommended:

- switch previous window of current application
- switch next window of current application

### Lifecycle/state

Required:

- close current window
- minimize current window
- maximize current window
- restore current window

Strongly recommended:

- toggle full screen

### Spatial placement

Strongly recommended where the platform supports window management:

- place left half
- place right half
- place top half
- place bottom half
- place upper-left
- place upper-right
- place lower-left
- place lower-right
- maximize/restore

Adapters should advertise unsupported placements rather than silently substituting a different meaning.

### Moving between displays and workspaces

Strongly recommended:

- send window to display left/right
- send window to previous/next workspace

Direct numbered display/workspace targeting may be offered by capable adapters.

---

## 4.12 APPLICATION — application-level control

Clavis must distinguish an **application** from a **window** even on platforms where the native switcher blurs them.

### Required

- switch previous application
- switch next application
- quit current application

### Strongly recommended

- launch configured application by name/identifier
- activate configured application by name/identifier
- reopen/focus configured application
- launch new application instance where supported

Named application commands should be data/configuration, not individually memorized theory exceptions.

Quit is more destructive than window close and should receive the stronger safety treatment established in the first research layer.

---

## 4.13 WORKSPACE — virtual desktop/workspace control

Clavis should normalize "virtual desktop" and "workspace" into one semantic domain.

### Required when the host provides workspaces

- switch previous workspace
- switch next workspace

### Strongly recommended

- create workspace
- close current workspace
- switch first/last workspace where meaningful
- switch to indexed workspace where supported
- send current window to previous/next workspace

Closing a workspace must follow the platform's actual behavior and should be treated as a potentially disruptive action.

---

## 4.14 DISPLAY — physical monitor relationships

A display is distinct from a virtual workspace.

### Strongly recommended

- send current window to display left
- send current window to display right
- send current window to previous/next display when the platform exposes an ordering

### Extension candidates

- select presentation mode
- mirror/extend displays
- brightness
- display rotation

These vary considerably by hardware and OS.

---

## 4.15 SYSTEM — machine-level functions

Only broadly useful actions should enter the portable command surface.

### Required/recommended core

- open system launcher/search
- lock session
- show desktop
- open notifications
- show clipboard history where available

### Screenshot family

Strongly recommended:

- capture full screen
- capture current window
- capture selected region

### Audio/media family

Strongly recommended:

- volume up
- volume down
- mute/unmute
- play/pause
- next media item
- previous media item

### Optional hardware/system extensions

- microphone mute
- brightness up/down
- screen recording
- system settings
- task/system monitor
- accessibility tools

### Guarded destructive system actions

These should be opt-in extensions, not everyday core strokes:

- sleep
- sign out
- restart
- shutdown
- force quit
- permanent deletion

They require structural differentiation or confirmation.

---

## 4.16 PLOVER — stenography engine control

Plover must be operable without reaching for the ordinary keyboard or mouse.

### Required

- resume output
- suspend output
- toggle output
- focus Plover
- open lookup
- open suggestions
- add translation
- open configuration

### Strongly recommended

- open dictionary management
- show current dictionaries / state
- reload relevant Clavis dictionaries or configuration when safely implementable

### Guarded

- quit Plover

A Clavis implementation must preserve the distinction between "leave Clavis control mode" and "suspend Plover output." They are not the same action.

---

# 5. Object/file operations: extension-ready but not primitive core

File managers reveal a useful generic **OBJECT** model:

- move selection previous/next/up/down;
- open/activate selected object;
- rename selected object;
- delete selected object;
- create container/folder;
- open properties;
- context menu.

However, object operations vary significantly across lists, trees, desktops, file managers, mail clients, and media libraries.

For v0.1, Clavis should cover generic object traversal through **UI** and generic activation through **UI ACTIVATE**. A later research pass can decide whether FILE/OBJECT deserves a distinct first-class domain.

Permanent deletion must never be the default meaning of generic delete.

---

# 6. Operations deliberately excluded from the portable core

The following should initially be extensions, macros, or adapter-specific commands:

- arbitrary application menu commands;
- IDE-specific refactoring/navigation;
- browser developer tools;
- spreadsheet-specific cell/range editing;
- presentation editing;
- mail-client commands;
- Git commands;
- terminal shell commands;
- Vim/Emacs command sets;
- rich-text formatting commands;
- pointer/mouse movement and clicking;
- drag/drop;
- OCR or screen-coordinate automation;
- arbitrary raw modifier/key emission.

Clavis should eventually provide an **escape hatch** for literal key/modifier emission, because some software exposes no better automation surface. That escape hatch is an expert interoperability feature, not the conceptual foundation of the theory.

---

# 7. Portability and adapter capability rules

Every semantic command should have one of four adapter states:

1. **native** — directly supported by the host/application;
2. **derived** — safely implemented as a short composition of other actions;
3. **contextual** — supported only in particular applications or UI states;
4. **unsupported** — unavailable and must fail closed with feedback when practical.

Adapters must not silently substitute a semantically different action merely because a convenient shortcut exists.

Examples:

- if an OS lacks "switch next application" as distinct from "switch next window," the adapter should document or implement the distinction rather than pretending they are identical;
- if a platform does not support quarter-window placement, WINDOW PLACE UPPER_LEFT should be unavailable or explicitly derived through a window manager, not converted to WINDOW PLACE LEFT;
- if an application has no plain-text paste, the adapter may derive it only if it can do so without surprising data loss.

---

# 8. Safety classes

The inventory suggests three safety classes.

## Class A — routine/reversible

Examples:

- movement;
- selection;
- scrolling;
- focus traversal;
- copy;
- tab switching;
- window switching.

These should optimize for speed.

## Class B — locally destructive but normally recoverable

Examples:

- cut;
- delete text;
- close tab;
- close document;
- close window;
- replace all.

These may be fast but should remain structurally distinct from navigation.

## Class C — globally disruptive or difficult to reverse

Examples:

- quit application with unsaved state;
- quit Plover;
- close workspace with surprising consequences;
- permanent file deletion;
- force quit;
- sign out;
- restart;
- shutdown.

These require a stronger marker, multi-stage form, or confirmation policy.

Safety class is semantic metadata and should eventually be testable in the implementation.

---

# 9. Minimal real-world workflow test

Before Clavis assigns physical strokes, this inventory should be judged against ordinary workflows.

A complete core should let a user perform the following without touching a conventional keyboard.

## Writing and feedback

- move by character, word, line, paragraph, and document boundary;
- extend selection using the same movement logic;
- delete around the cursor;
- undo/redo;
- copy/cut/paste and preferably paste plain text;
- find and traverse matches;
- insert paragraph and line breaks;
- save and print.

## Browser research

- focus the address/location field;
- navigate back/forward;
- create, close, reopen, switch, and reorder tabs;
- scroll;
- find in page;
- zoom;
- move between page controls;
- activate links/buttons through UI focus;
- switch to another application/window and return.

## General desktop work

- switch windows and applications;
- move among virtual workspaces;
- position windows;
- move a window to another display/workspace;
- open system search;
- lock the machine;
- capture screenshots;
- control audio/media.

## Dialogs and forms

- move focus next/previous;
- move directionally inside a control;
- activate/confirm;
- cancel;
- toggle;
- expand/collapse;
- open a context menu.

## Plover maintenance

- suspend/resume;
- lookup;
- suggestions;
- add translation;
- configure;
- return safely to writing state.

If any of these routine workflows requires memorizing unrelated one-off control briefs, the later stroke theory has failed the semantic design even if the commands technically work.

---

# 10. Proposed atomic vocabulary for stroke-design research

The next research layer should attempt to encode the inventory using a very small recurring vocabulary.

## Actions

Candidate stable actions:

- move
- extend
- delete
- select
- focus
- switch
- activate
- cancel
- toggle
- create
- open
- close
- reopen
- save
- quit
- send
- place
- scroll
- zoom
- find
- replace
- copy
- cut
- paste
- undo
- redo
- capture
- repeat

Not every action should need an independent physical marker; some may be systematic transformations of another action.

## Targets/domains

- text
- view
- UI/control
- location
- document
- tab
- pane
- window
- application
- workspace
- display
- system
- Plover/Clavis

## Scale/units

- character
- word
- line
- paragraph
- page/screen
- document

## Relationships

- previous / next
- backward / forward
- up / down
- left / right
- start / end
- first / last
- current
- index/name
- quantity/repetition

The physical theory should try to assign **features** to these recurring dimensions rather than assign a unique mnemonic outline to each command.

---

# 11. Design pressure revealed by the inventory

The inventory exposes several constraints that should guide the eventual steno theory.

1. **Direction will be extremely high-frequency.** It must be effortless and reusable.
2. **Previous/next and spatial directions should be related but not conflated.**
3. **Selection should be a modification of movement.**
4. **Deletion should reuse the same text-unit system.**
5. **Container traversal should reuse one previous/next mechanism across tabs, windows, apps, workspaces, panes, and matches.**
6. **UI navigation needs activation/cancel/toggle in addition to directional movement.**
7. **TEXT and VIEW require distinct markers.**
8. **WINDOW, APPLICATION, and WORKSPACE must remain distinguishable even when a host OS blurs them.**
9. **Transient and latched control need inexpensive entry/exit.**
10. **Counts/repetition should compose without multiplying dictionary entries.**
11. **Safety class must be visible in the eventual encoding.**
12. **Named applications and platform-specific features belong in configuration/adapters, not in the base theory.**

---

# 12. Research references

Official platform/application references:

- Microsoft Windows keyboard shortcuts:  
  https://support.microsoft.com/en-us/windows/keyboard-shortcuts-in-windows-dcc61a57-8ff0-cffe-9796-cb9706c75eec

- Microsoft Windows multitasking and Snap:  
  https://support.microsoft.com/en-us/windows/how-to-multitask-in-windows-b4fa0333-98f8-ef43-e25c-06d4fb1d6960  
  https://support.microsoft.com/en-us/windows/experience/snap-your-windows

- Microsoft Word keyboard shortcuts:  
  https://support.microsoft.com/en-us/accessibility/word/keyboard-shortcuts-in-word

- Apple Mac keyboard shortcuts:  
  https://support.apple.com/102650

- GNOME keyboard navigation:  
  https://help.gnome.org/gnome-help/keyboard-nav.html

- KDE common keyboard shortcuts:  
  https://docs.kde.org/trunk_kf6/en/khelpcenter/fundamentals/kbd.html

- Google Chrome keyboard shortcuts:  
  https://support.google.com/chrome/answer/157179

- Mozilla Firefox keyboard shortcuts:  
  https://support.mozilla.org/kb/keyboard-shortcuts-perform-firefox-tasks-quickly

- Visual Studio Code basic editing:  
  https://code.visualstudio.com/docs/editing/codebasics

Steno/Plover references:

- Plover default commands:  
  https://github.com/openstenoproject/plover/blob/main/plover/assets/commands.json

- Kaoffie, Plover Modal Dictionary:  
  https://github.com/Kaoffie/plover_modal_dictionary

- Di Does Digital, steno dictionaries:  
  https://github.com/didoesdigital/steno-dictionaries

- Aerick, semi-modal movement:  
  https://github.com/aerickt/steno-dictionaries

- Lapwing all-in-one and modal movement:  
  https://github.com/aerickt/plover-lapwing-aio

- Paul Fioravanti, command dictionaries:  
  https://github.com/paulfioravanti/steno-dictionaries/blob/main/dictionaries/commands.md

- Josiah Tan, Plover-Vim:  
  https://github.com/Josiah-tan/plover-vim

---

# 13. Next research step

The next step should **not** assign the full dictionary yet.

It should design the smallest possible **physical feature map** for computer control:

- how Clavis marks control versus lexical writing;
- how transient versus latched control are entered;
- how direction is encoded;
- how previous/next relates to left/right/up/down;
- how text scale is encoded;
- how MOVE becomes EXTEND or DELETE;
- how container domains such as TAB/WINDOW/APPLICATION/WORKSPACE are distinguished;
- how count/repetition is encoded;
- how Class C destructive actions receive a safety marker.

That feature map can then be tested against every semantic family in this inventory before any concrete dictionary is generated.

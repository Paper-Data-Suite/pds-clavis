# Clavis Control Runtime Contract

**Status:** Architecture research draft v0.1  
**Target:** Plover 5.x plugin architecture  
**Scope:** Runtime semantics and Plover integration boundary for Clavis computer control. This document does **not** implement the plugin yet.

## Purpose

The previous Clavis layers established:

1. a semantic computer-control grammar;
2. a canonical semantic command inventory;
3. a physical feature map;
4. a named-action palette;
5. a 152-command mechanical stroke prototype with no internal stroke collisions.

The mechanical audit also exposed a hard requirement:

> A reserved Clavis control stroke must never fall through into an ordinary Plover dictionary when Clavis does not recognize it.

That requirement cannot safely be expressed as "put a sparse JSON dictionary above the others."

This document specifies the runtime contract that must exist between Plover, Clavis control theory, semantic command resolution, and platform/application adapters.

The goal is to preserve Plover as the steno engine while giving Clavis strict ownership of its control namespace.

---

# 1. Feasibility conclusion

Clavis does **not** currently need its own steno engine.

Plover already provides the major capabilities Clavis needs:

- machine/hardware input;
- steno stroke parsing;
- dictionary lookup;
- code-driven dictionary plugins;
- command plugins;
- extension plugins;
- translation formatting;
- arbitrary key-combination output;
- built-in engine commands;
- cross-platform keyboard emulation.

Plover's current plugin model explicitly supports:

- `plover.dictionary` plugins;
- `plover.command` plugins;
- `plover.extension` plugins;
- code-driven/programmatic dictionary lookup;
- commands that receive the active `StenoEngine`.

The proposed implementation is therefore:

```text
Plover machine input
        |
        v
Clavis routing dictionary
        |
        +---- ordinary non-Clavis stroke ----> normal Plover dictionaries
        |
        +---- reserved Clavis stroke --------> Clavis semantic command
                                                   |
                                                   v
                                           semantic resolver
                                                   |
                                                   v
                                             adapter plan
                                                   |
                         +-------------------------+----------------------+
                         |                         |                      |
                         v                         v                      v
                  Plover key combo          Plover command        external adapter
                         |                         |                      |
                         +-------------------------+----------------------+
                                                   |
                                                   v
                                                host
```

A small Clavis extension owns runtime state and lifecycle hooks.

No Plover fork is required by this contract.

---

# 2. Recommended plugin components

The eventual Python package should expose three Plover-facing components and one internal adapter layer.

## 2.1 Clavis routing dictionary

**Plugin type:** `plover.dictionary`

Responsibilities:

- identify whether a stroke belongs to the reserved Clavis namespace;
- parse valid Clavis strokes into canonical semantic command IDs;
- claim invalid reserved strokes so they cannot fall through;
- while Latched Control is active, claim the latched payload namespace;
- emit Plover translation-language actions or calls to the Clavis command plugin;
- remain side-effect-free during lookup.

The routing dictionary is **not** an ordinary static dictionary.

It is a programmatic/code-driven dictionary whose result depends on:

- the stroke;
- the current Clavis runtime state;
- the generated control grammar;
- adapter capability information where needed.

## 2.2 Clavis command plugin

**Plugin type:** `plover.command`

Responsibilities:

- perform runtime state transitions;
- execute semantic actions that cannot be represented purely as Plover translation-language output;
- invoke Plover engine commands;
- invoke platform/application adapters;
- record structured runtime outcomes;
- provide non-invasive failure/rejection feedback.

The command plugin receives Plover's active engine object.

## 2.3 Clavis extension

**Plugin type:** `plover.extension`

Responsibilities:

- initialize and own the process-local Clavis runtime state;
- connect to relevant Plover engine hooks;
- verify that the routing dictionary is installed and correctly prioritized;
- reset transient Clavis state on lifecycle events;
- expose current state to the routing dictionary and command plugin;
- manage adapter/context services;
- provide optional feedback/status services.

## 2.4 Adapter layer

Internal Clavis component.

Responsibilities:

- map canonical semantic commands to platform/application behavior;
- report capability state;
- return an execution plan;
- never redefine the semantic meaning of a command.

Adapters may be:

- Windows;
- macOS;
- Linux desktop environment/window manager;
- browser-specific;
- editor-specific;
- application-specific.

Application-specific adapters override implementation, not Clavis theory.

---

# 3. Runtime state model

Clavis runtime state is process-local and ephemeral.

It must **not** be persisted across Plover restarts.

The minimum state is:

```text
mode:
    WRITING
    LATCHED

pending_count:
    none
    positive integer

pending_guard:
    none
    GuardRequest

last_control_action:
    none
    semantic command ID + resolved arguments

runtime_health:
    READY
    DISARMED
    DEGRADED
```

Transient Control is not a stored mode.

A transient command is simply:

```text
WRITING + gated command -> execute -> WRITING
```

## 3.1 WRITING

Ordinary state.

- non-gated strokes are not claimed by Clavis;
- ordinary Plover lexical dictionaries remain fully active;
- gate alone enters LATCHED;
- gate + valid payload executes one Clavis command;
- gate + invalid payload is consumed and rejected.

## 3.2 LATCHED

Control-only state.

- the control gate is omitted from payload strokes;
- valid payloads execute commands;
- unknown payloads are consumed;
- lexical output is forbidden;
- gate alone exits to WRITING.

The mode does not automatically fall back to lexical writing on mismatch.

This is deliberate.

## 3.3 Guard pending

Guard is an orthogonal pending condition, not a third general writing mode.

A Class C request does not execute immediately.

It creates:

```text
GuardRequest:
    semantic_id
    resolved target identity if available
    creation sequence/time metadata
    adapter/context fingerprint if relevant
```

Only an explicit uniform confirmation may execute that exact request.

Cancellation, expiry, context change, or unrelated input destroys the request without executing it.

---

# 4. Reserved namespace ownership

## 4.1 Transient namespace

Any single stroke containing the complete Clavis control gate is **reserved**.

Current gate:

```text
STPH
```

Therefore:

```text
contains full left STPH gate
    => Clavis owns the stroke
```

This remains true whether or not the payload is valid.

Invalid gated strokes may never fall through to:

- Plover's stock `commands.json`;
- the lexical dictionary;
- user dictionaries;
- another command dictionary.

## 4.2 Latched namespace

While LATCHED is active:

> Every stroke submitted as a Clavis payload belongs to Clavis until the user explicitly exits control.

Unknown payloads fail closed.

## 4.3 Lexical namespace

While WRITING is active:

- strokes without the gate are not claimed by Clavis;
- normal Plover lookup proceeds unchanged.

This is the central coexistence boundary between Clavis control and future Clavis lexical theory.

---

# 5. Routing dictionary contract

The routing dictionary must behave as a **pure resolver**.

Dictionary lookup must not itself:

- change modes;
- change counts;
- execute host actions;
- send keys;
- mutate guard state.

Plover may perform dictionary lookup more than once while resolving translation history. Lookup side effects would therefore be unsafe.

## 5.1 Conceptual lookup behavior

```python
lookup(outline):
    if outline is not relevant to Clavis:
        raise KeyError

    if runtime is WRITING:
        if outline is gate-alone:
            return command("clavis.enter_latched")

        if outline has gate:
            parsed = parse_gated(outline)

            if parsed is invalid:
                return command("clavis.reject", reason)

            return compile_semantic(parsed)

        raise KeyError

    if runtime is LATCHED:
        if outline is gate-alone:
            return command("clavis.exit_latched")

        parsed = parse_latched(outline)

        if parsed is invalid:
            return command("clavis.reject", reason)

        return compile_semantic(parsed)
```

The real implementation must also address translator lookback/folding, described below.

---

# 6. Dictionary priority contract

The Clavis routing dictionary must be the **highest-priority enabled dictionary that can see Clavis control strokes**.

The extension must verify this at startup and whenever dictionaries change.

If the invariant cannot be established:

```text
runtime_health = DISARMED
```

and the UI/log must state why.

Clavis must not silently assume priority.

A future Clavis Plover system plugin may make installation more automatic, but the runtime contract may not depend on users manually maintaining a fragile priority order without validation.

---

# 7. Translator lookback / folding acceptance gate

Plover supports multi-stroke dictionary translation and can reconsider previous strokes when longer outlines become available.

That creates an important implementation risk:

> A control stroke that has already executed must never later be absorbed into an ordinary multi-stroke lexical outline.

This risk must be characterized before the routing implementation is considered safe.

## Required tests

The first plugin prototype must prove that:

1. a valid transient control stroke followed by a lexical stroke cannot later be replaced by a lower-priority multi-stroke lexical translation;
2. an invalid gated stroke followed by another stroke cannot participate in a lower dictionary outline;
3. consecutive latched payloads cannot be reinterpreted together by lower dictionaries;
4. entering and leaving LATCHED creates a hard semantic boundary for translation purposes;
5. no solution clears normal lexical translation history merely to force the boundary.

## Escalation rule

If a high-priority dynamic dictionary cannot enforce this boundary within Plover's normal translator semantics, Clavis should escalate **one integration layer**, not replace Plover.

Acceptable escalation candidates include:

- a more specialized Plover dictionary/translation plugin boundary;
- a narrowly scoped Plover extension that controls translator interaction;
- a Clavis-specific system/plugin integration.

Replacing the whole steno engine remains unjustified unless Plover makes safe interception impossible even at those supported extension points.

---

# 8. Fail-closed rejection contract

A rejected Clavis stroke has four required properties:

1. no lexical output;
2. no host key combination;
3. no semantic action;
4. no unintended Clavis state transition.

Plover's translation language provides a true null action:

```text
{#}
```

It produces no output, does not affect formatting, and is not undoable through the ordinary steno undo mechanism.

Clavis may use either:

- a pure null translation; or
- a Clavis rejection command that performs logging/feedback but no host action.

Feedback must never require injecting text into the active application.

Allowed feedback examples:

- Plover status/log entry;
- optional sound;
- optional Clavis UI indication.

---

# 9. State transitions

## 9.1 WRITING transitions

| Input | Result | Next state |
|---|---|---|
| ordinary lexical stroke | Clavis does not claim it | WRITING |
| gate alone | enter Latched Control | LATCHED |
| gate + valid Class A/B command | execute once | WRITING |
| gate + invalid payload | consume + reject | WRITING |
| gate + Class C command | create pending guard; no host action | WRITING + guard pending |

## 9.2 LATCHED transitions

| Input | Result | Next state |
|---|---|---|
| valid Class A/B payload | execute | LATCHED |
| invalid payload | consume + reject | LATCHED |
| gate alone | exit | WRITING |
| guarded payload | create pending guard; no host action | LATCHED + guard pending |

Unknown input never exits LATCHED automatically.

## 9.3 Guard-pending transitions

| Input/event | Result |
|---|---|
| exact confirm action | revalidate target/context, then execute once |
| cancel action | cancel |
| unrelated stroke | cancel and consume unrelated stroke |
| control-mode exit | cancel |
| Plover output/state reset | cancel |
| adapter/context target changes materially | cancel |
| confirmation window expires | cancel |
| execution failure | cancel + report failure |

The unrelated stroke is consumed after cancelling.

This intentionally requires the user to stroke it again.

Safety wins over convenience while a destructive action is armed.

---

# 10. Guard confirmation contract

All Class C actions require a two-stage process by default.

Examples:

- quit application;
- quit Plover;
- sign out;
- restart;
- shutdown;
- force quit;
- permanent deletion extensions.

## Stage 1 — request

The guarded command:

- resolves the semantic action;
- resolves the intended target where possible;
- stores a pending guard;
- performs no external action.

## Stage 2 — confirm

A single universal Clavis confirmation action:

- checks that the request still exists;
- checks that it has not expired;
- revalidates target/context;
- executes exactly the stored semantic command.

The physical confirmation stroke is not frozen in this document.

## Context pinning

If a destructive command targets a specific application/window/object, confirmation must not silently act on whatever happens to be focused later.

Where the OS adapter can identify the target, the pending guard stores that identity.

If the target changes, confirmation fails closed.

---

# 11. Count contract

Counts are pending runtime state, not duplicated dictionary entries.

```text
COUNT / COMMAND
```

## Rules

- count must be a positive bounded integer;
- only commands marked `repeatable` accept a count;
- Class C commands are never repeatable;
- lifecycle/destructive commands default to non-repeatable;
- count applies to one subsequent command only;
- after execution, rejection, mode exit, or failure, count clears;
- a second count replaces or extends the first only according to the future Clavis number grammar.

## Execution

A counted command is resolved once semantically, then executed N times.

Adapters must not reinterpret "3 next tabs" as a separate command definition.

If execution fails on repetition `k`:

- stop;
- report partial completion;
- clear the count.

---

# 12. Repeat-previous-action contract

Clavis may expose a repeat action later.

`last_control_action` may store only an action that is:

- successfully executed;
- explicitly marked repeatable;
- not Class C.

Repeat does not store:

- rejected actions;
- unsupported actions;
- failed actions;
- guard requests;
- mode transitions.

The repeat action re-resolves current adapter context unless the semantic command explicitly requires the original target.

---

# 13. Semantic command contract

Every executable Clavis operation has a stable ID.

Examples:

```text
text.move.word.previous
text.extend.line.end
edit.copy
tab.switch.next
window.place.upper_right
plover.lookup
system.shutdown
```

The semantic ID:

- is platform-independent;
- is application-independent;
- never contains Ctrl/Alt/Command/Windows shortcut notation;
- does not change when an adapter changes implementation.

Semantic IDs are the contract between:

- theory;
- routing;
- adapters;
- tests;
- logs;
- documentation.

---

# 14. Adapter capability contract

Each adapter reports a capability classification already established by the semantic inventory:

```text
NATIVE
DERIVED
CONTEXTUAL
UNSUPPORTED
```

## NATIVE

The platform/application exposes the semantic action directly or through a canonical shortcut/API.

## DERIVED

Clavis can implement the semantic action safely as a bounded sequence of other operations.

## CONTEXTUAL

The action is valid only when a known context predicate is true.

Example:

```text
pane.close
```

may be contextual to editors/terminals that expose panes.

## UNSUPPORTED

No safe semantic implementation is available.

Unsupported never means "do something similar."

---

# 15. Runtime outcome contract

Capability classification and execution result are separate.

An execution attempt returns one of:

```text
EXECUTED
UNSUPPORTED
REJECTED
FAILED
CANCELLED
```

## EXECUTED

The requested semantic action completed according to adapter contract.

## UNSUPPORTED

The adapter does not implement the semantic command in the current environment.

No fallback action is emitted.

## REJECTED

The stroke was syntactically invalid or prohibited by runtime policy.

## FAILED

The adapter attempted execution but encountered an error.

## CANCELLED

A pending guarded action or comparable pending operation was deliberately abandoned.

These outcomes should be structured log events.

---

# 16. Execution-plan contract

The semantic resolver produces an `ExecutionPlan`.

Conceptually:

```text
ExecutionPlan:
    semantic_id
    capability
    backend
    payload
    formatting_disposition
    safety_class
    repeatable
    context_requirements
```

Possible backends:

```text
PLOVER_TRANSLATION
PLOVER_COMMAND
CLAVIS_COMMAND
EXTERNAL_ADAPTER
```

## PLOVER_TRANSLATION

Preferred for ordinary key-combination actions.

Example conceptual payload:

```text
{}{#Left}
```

Plover itself performs the keyboard output.

## PLOVER_COMMAND

Preferred for built-in Plover actions.

Examples:

```text
{PLOVER:LOOKUP}
{PLOVER:SUGGESTIONS}
{PLOVER:ADD_TRANSLATION}
{PLOVER:RELOAD_DICTIONARIES}
```

## CLAVIS_COMMAND

Used for:

- state transitions;
- guard management;
- count management;
- adapter orchestration;
- Clavis help/status;
- safe rejection feedback.

## EXTERNAL_ADAPTER

Used only where a semantic action cannot be expressed reliably through Plover key combinations or built-in commands.

This backend may use OS/application APIs.

---

# 17. Formatting-state contract

Computer control can desynchronize Plover's linguistic formatting state from the real insertion point.

Plover explicitly documents:

```text
{}
```

as a way to cancel formatting of the next word and specifically recommends it around cursor movement.

Clavis therefore assigns every command a `formatting_disposition`.

## 17.1 PRESERVE

Use when the action does not materially change the text insertion context.

Examples:

- copy;
- view scroll;
- zoom;
- screenshot;
- media control;
- volume;
- non-focus-changing system status actions.

The translation does not cancel Plover's pending formatting state.

## 17.2 CANCEL_PENDING

Use when the action may:

- move the caret;
- change selection;
- modify text outside Plover;
- switch editable documents/controls;
- change focused application/window/tab;
- navigate to another location.

Examples:

- cursor movement;
- selection extension;
- delete;
- cut;
- paste;
- undo/redo;
- find-next;
- UI focus movement;
- tab/window/application/workspace switch;
- document open/new/close;
- browser back/forward;
- system launcher;
- focus Plover.

The generated translation should cancel pending formatting before execution:

```text
{} + action
```

This avoids carrying capitalization/attachment assumptions into a different insertion context.

## 17.3 ESTABLISH_TEXT_STATE

Use for actions that intentionally insert textual structure through Clavis/Plover.

Examples:

- paragraph break;
- soft line break.

These translations explicitly establish the correct subsequent formatting state.

## 17.4 NONE

Use for purely internal Clavis state changes:

- enter/exit Latched Control;
- count prefix;
- guard request/cancel;
- safe reset.

---

# 18. What Clavis must not do to "fix" formatting

Clavis must not routinely call:

```text
engine.clear_translator_state()
```

after computer-control commands.

Plover documents that this clears its translation stack; doing so would destroy useful translation history and undo context.

Full translator-state clearing is therefore a recovery/debug mechanism only, not ordinary control behavior.

The preferred mechanism is the normal Plover formatting language, especially `{}` where appropriate.

---

# 19. Plover lifecycle hooks

The extension should use Plover lifecycle hooks conservatively.

Relevant current hooks include:

- `output_changed`;
- `config_changed`;
- `dictionaries_loaded`;
- `dictionary_state_changed`;
- `machine_state_changed`;
- `quit`.

## Reset events

The following events must clear:

- pending count;
- pending guard;
- temporary palette state.

They should also return Clavis to WRITING unless explicitly proven safe otherwise:

- Plover output disabled;
- system/theory changed;
- routing dictionary removed/disabled;
- dictionary priority invariant lost;
- Clavis plugin reload;
- Plover shutdown;
- unrecoverable adapter exception.

## Machine disconnect

Machine disconnect should cancel pending guard/count.

Mode may be reset to WRITING on reconnect for predictability.

---

# 20. Plover output suspension

Clavis must distinguish:

```text
Clavis WRITING/LATCHED state
```

from:

```text
Plover output enabled/disabled
```

They are separate concepts.

When Plover output is suspended:

- Clavis temporary state resets to WRITING;
- pending guard/count clear.

Plover's built-in `resume` and `toggle` commands are specially supported even while normal output is disabled, so Clavis should use Plover's engine command semantics for those operations rather than trying to emulate them externally.

---

# 21. Runtime-health contract

## READY

All required invariants hold:

- routing component loaded;
- priority verified;
- adapters initialized;
- semantic table valid;
- no duplicate command IDs;
- control namespace usable.

## DISARMED

Clavis cannot guarantee safe namespace handling.

Examples:

- routing dictionary missing;
- incorrect dictionary priority;
- incompatible Plover version;
- required plugin component failed to initialize.

DISARMED means:

> Clavis control should not be advertised as available.

## DEGRADED

Core routing safety remains intact, but one or more optional adapter capabilities are unavailable.

Examples:

- window placement backend missing;
- clipboard-history command unavailable;
- no workspace API.

DEGRADED is acceptable because unsupported commands still fail closed.

---

# 22. Error-handling rules

## Parser error

- consume reserved stroke;
- no host action;
- runtime outcome `REJECTED`;
- state otherwise unchanged.

## Unsupported command

- consume stroke;
- no substitute;
- outcome `UNSUPPORTED`.

## Adapter exception before host action

- no action;
- outcome `FAILED`;
- clear pending count;
- cancel pending guard if any.

## Partial derived execution

- stop immediately;
- outcome `FAILED`;
- record bounded partial-completion details;
- never retry automatically.

## Internal runtime invariant failure

- transition to safe WRITING state;
- clear count/guard;
- if namespace safety is uncertain, set `DISARMED`.

---

# 23. No automatic semantic fallback

Clavis adapters may have implementation fallbacks but not **meaning fallbacks**.

Allowed:

```text
window.place.left
    native Windows Snap
    OR
    safe window-management API
```

Not allowed:

```text
window.place.upper_left unsupported
    -> silently place left half
```

Similarly:

```text
application.switch.next
```

may not silently become:

```text
window.switch.next
```

because those are distinct Clavis meanings.

---

# 24. Feedback contract

Feedback should be useful without polluting the active document.

## Required

Structured logging for:

- rejected stroke;
- unsupported semantic command;
- adapter failure;
- guard requested/cancelled/executed;
- runtime-health change.

## Optional

- short sound;
- Plover/Clavis status indicator;
- compact notification.

## Forbidden

- typing an error message into the user's active application;
- sending a fallback keyboard shortcut;
- changing the active selection merely to signal failure.

---

# 25. Safe Reset

Clavis requires one semantic command:

```text
clavis.safe_reset
```

It must:

- set mode to WRITING;
- clear pending count;
- cancel pending guard;
- clear temporary Clavis palette state;
- leave Plover's lexical translation history intact;
- emit no host key combination;
- return `EXECUTED`.

Safe Reset does **not** mean:

- restart Plover;
- clear Plover translator history;
- suspend output;
- reload configuration.

It is a Clavis runtime reset only.

---

# 26. Threading and mutation rule

Plover routes machine strokes through its engine thread and command plugins execute with access to the engine.

Clavis should keep all state mutations on the Plover engine/control path where practical.

The routing lookup may read state but must not mutate it.

If an external context watcher runs on another thread:

- it may publish immutable context snapshots;
- it may not directly change mode/count/guard state;
- runtime transitions still occur through the Clavis command/runtime service.

This minimizes race conditions.

---

# 27. Generated data must be authoritative

The executable control map should not be hand-maintained separately from the research grammar.

Eventually:

```text
semantic specification
        |
        v
stroke generator
        |
        +--> routing table
        +--> documentation table
        +--> collision tests
        +--> adapter coverage report
```

The existing `clavis-control-stroke-prototype.csv` is a research artifact.

The production system should replace manual duplication with a typed/generated command specification.

A semantic command and its stroke should have one authoritative source.

---

# 28. Required unit tests

## Namespace ownership

- gated valid stroke is claimed;
- gated invalid stroke is claimed;
- ungated lexical stroke in WRITING is not claimed;
- every payload stroke in LATCHED is claimed;
- unknown latched stroke produces no lexical output.

## State transitions

- gate alone: WRITING -> LATCHED;
- gate alone: LATCHED -> WRITING;
- transient command leaves WRITING unchanged;
- invalid transient remains WRITING;
- invalid latched remains LATCHED;
- safe reset always returns WRITING;
- output disable clears temporary state.

## Count

- count applies exactly once;
- count clears after execution;
- count clears on failure/rejection/exit;
- non-repeatable command rejects count;
- Class C cannot be counted.

## Guard

- request does not execute;
- confirm executes once;
- cancel executes nothing;
- unrelated stroke cancels and is consumed;
- expiry cancels;
- target/context change cancels;
- second confirmation cannot execute again.

## Adapter results

- UNSUPPORTED emits no host action;
- REJECTED emits no host action;
- FAILED does not fall back semantically;
- DERIVED action stops after first failed step.

## Formatting

- movement cancels pending capitalization/attachment;
- deletion cancels pending formatting;
- tab/app/window focus change cancels pending formatting;
- copy preserves pending formatting;
- view scroll preserves pending formatting;
- paragraph break establishes intended next-text state.

---

# 29. Required integration tests against real Plover

These are release blockers for the first executable Clavis prototype.

## 29.1 Lower-dictionary fall-through

Install:

1. Clavis router;
2. a deliberately conflicting lower dictionary.

Verify:

- invalid gated stroke does not invoke lower translation;
- invalid latched stroke does not invoke lower translation.

## 29.2 Multi-stroke lookback/folding

Create deliberate two- and three-stroke lexical outlines that begin with or span control strokes.

Verify:

- previously executed control commands are never retroactively absorbed;
- lexical output never crosses the control boundary.

## 29.3 Plover stock command collisions

Keep stock Plover command dictionary enabled.

Verify:

- Clavis owns every reserved gated stroke;
- undefined Clavis gated forms do not execute stock `STPH...` commands.

## 29.4 Formatting clinic

Test sequences such as:

```text
sentence-ending punctuation
control cursor movement
ordinary word
```

and confirm the next word is not incorrectly capitalized because of stale Plover state.

## 29.5 Output suspend/resume

Verify Clavis's Plover resume/toggle path functions using supported Plover engine commands rather than external keyboard emulation.

## 29.6 Dictionary reload and priority changes

Disable, reorder, or reload dictionaries while Clavis is running.

Verify:

- runtime detects loss of routing safety;
- temporary state resets;
- Clavis reports DISARMED rather than pretending control is safe.

---

# 30. Property-based tests

Several Clavis guarantees are better expressed as invariants than examples.

## Reserved-gate property

For every physically valid one-stroke chord containing the complete control gate:

```text
Clavis result != ordinary lexical fall-through
```

## Latched property

For every physically valid single stroke while LATCHED:

```text
Clavis claims stroke
```

## Invalid-combination property

For every generated-invalid combination of:

- family;
- scope;
- operation rail;
- action position;
- transform;

result is:

```text
REJECTED
```

and never another semantic command.

## Round-trip generation property

Every generated production stroke maps to exactly one semantic command.

Every semantic command with a core stroke maps back to that same command.

---

# 31. Version-compatibility contract

Clavis should declare the Plover versions it has qualified against.

It should not assume that internal Plover behavior is stable merely because public plugin entry points exist.

Compatibility checks should distinguish:

- public plugin APIs used;
- observed translator behavior characterized by tests;
- private APIs intentionally **not** relied upon.

## Private API rule

Clavis Core should not depend on private members such as:

```text
engine._keyboard_emulation
engine._translator
engine._formatter
```

unless a future Plover limitation forces it and the dependency is explicitly version-bounded.

The initial design should prefer:

- programmatic dictionary output;
- Plover translation language;
- public command API;
- public engine properties/methods;
- extension hooks.

---

# 32. Why a custom Clavis dictionary plugin is preferable to another dependency

Plover also supports third-party programmatic Python dictionaries.

Clavis could theoretically depend on such a plugin.

However, the routing dictionary is a **safety boundary**, not merely a convenient dictionary format.

The more robust design is for `pds-clavis` itself to expose the required `plover.dictionary` entry point.

Advantages:

- one installation unit;
- explicit version qualification;
- no hidden behavior from an intermediary plugin;
- controlled fail-closed semantics;
- direct tests against Clavis's namespace rules.

Clavis may learn from programmatic-dictionary implementations without making them a required runtime dependency.

---

# 33. Suggested internal module boundaries

This is architectural guidance, not committed filenames.

```text
clavis/
    grammar/
        semantic.py
        strokes.py

    runtime/
        state.py
        router.py
        guard.py
        count.py
        outcomes.py

    plover/
        dictionary.py
        commands.py
        extension.py
        translations.py

    adapters/
        base.py
        windows.py
        macos.py
        linux.py
        applications/

    generated/
        control_commands.py
```

The important boundary is:

```text
grammar != runtime != Plover integration != platform adapters
```

No Windows shortcut should appear in the grammar layer.

---

# 34. Minimum first executable slice

When implementation begins, do **not** start by implementing all 152 commands.

The first executable slice should prove the runtime boundary with a tiny command set:

```text
gate alone          enter / exit Latched Control
character west      text.move.character.previous
character east      text.move.character.next
edit undo           edit.undo
edit copy           edit.copy
invalid gated       reject
safe reset          clavis.safe_reset
```

Acceptance for that slice:

- ordinary lexical writing still works;
- transient commands work;
- latched commands work;
- invalid transient/latched strokes fail closed;
- stock Plover command collisions cannot leak through;
- formatting cancellation works on cursor movement;
- copy preserves formatting;
- no private Plover API is required;
- dictionary lookback/folding characterization passes.

Only then should the generator expand to the rest of the command inventory.

---

# 35. Architecture decisions recommended to freeze

1. **Plover remains the steno engine.**
2. **Clavis control requires runtime/plugin support; static dictionaries alone are insufficient.**
3. **A Clavis-owned code-driven routing dictionary is the preferred namespace boundary.**
4. **Routing lookup is pure; state mutation occurs through runtime commands.**
5. **Reserved gated input and latched input fail closed.**
6. **Clavis routing priority is verified, not assumed.**
7. **Class C commands use two-stage guarded execution.**
8. **Semantic command IDs are the stable interface to adapters.**
9. **Adapters report capability separately from execution outcome.**
10. **Formatting disposition is explicit metadata on every semantic command.**
11. **Plover translator history is not routinely cleared.**
12. **Private Plover engine members are avoided in Core.**
13. **Production stroke tables are generated from one authoritative specification.**
14. **Translator lookback/folding tests are mandatory before the first usable release.**

---

# 36. Remaining research questions before implementation

The runtime architecture is sufficiently defined, but these questions need characterization tests rather than more paper design:

1. How exactly does current Plover translator lookback behave around programmatic dictionary command translations?
2. Is a one-stroke dynamic router sufficient to form a hard translation boundary, or must the router claim bounded multi-stroke lookups as well?
3. What is the cleanest public-API mechanism for optional non-text feedback?
4. Which current Plover versions should the first Clavis release qualify against?
5. Which Windows backend is most reliable for semantic actions that ordinary key combinations cannot express?
6. What uniform physical stroke should confirm a pending guarded action?

Those questions belong in the first implementation/characterization issue.

---

# Research basis

Current Plover documentation and source consulted for this contract:

- Plover plugin types and plugin system:  
  https://plover.readthedocs.io/en/v4.0.1/plugins.html

- Plover plugin setup and entry points:  
  https://plover.readthedocs.io/en/v5.4.0/plugin-dev/setup.html

- Code-driven/programmatic dictionaries:  
  https://plover.readthedocs.io/en/v5.0.0/dict_formats.html

- Custom dictionary plugins:  
  https://plover.readthedocs.io/en/v5.4.1/plugin-dev/dictionaries.html

- Command plugins:  
  https://plover.readthedocs.io/en/v4.0.3/plugin-dev/commands.html

- Current Plover engine hooks and public engine surface:  
  https://github.com/opensteno/plover/blob/main/plover/engine.py

- Plover translation language, key combinations, null action, and formatting cancellation:  
  https://plover.readthedocs.io/en/v5.0.0/translation_language.html

- Current built-in command-line/engine commands:  
  https://plover.readthedocs.io/en/latest/cli_reference.html

- Plover dictionary collection lookup semantics:  
  https://github.com/opensteno/plover/blob/main/plover/steno_dictionary.py

# Clavis Mechanical Stroke Prototype and Collision Audit

**Status:** Research/prototype draft v0.1  
**Scope:** Literal candidate Plover strokes generated from the Clavis control grammar. These are **not yet frozen v0.1 dictionaries**.

## Executive result

- **152 semantic commands** were instantiated.
- **152 unique transient strokes** were generated; internal transient collisions: **0**.
- **152 unique latched payloads** were generated; internal latched collisions: **0**.
- Transient/latched namespaces overlap each other in **0** strokes.
- The literal notation is mechanically coherent on the standard Plover keyset.
- The prototype exposes one architectural requirement that should be frozen now: **Clavis control must own and intercept its reserved namespace rather than depend on ordinary dictionary fall-through.**

## 1. Literal encoding rules

The prototype converts the previous feature map directly into Plover steno notation:

| Feature | Physical keys |
|---|---|
| Control gate | `STPH` |
| Content family | no lower-left selector |
| Interface family | left `R-` |
| Container family | left `W-` |
| System family | left `K-` |
| scope 0 / A / O / E / U | none / `A` / `O` / `E` / `U` |
| west / north / south / east | `-R` / `-P` / `-B` / `-G` |
| west boundary / east boundary | `-FR` / `-LG` |
| Anchor/Carry | `*` |
| Primary action plane | `-T` |
| Secondary action plane | `-S` |
| Remove/Close | `-D` |
| Tools/Meta | `-Z` |
| Guarded | `-DZ` |

### Family-gate bridges

The family selectors form useful same-finger vertical bridges with the gate:

- Interface: gate `H-` + selector `R-` → left-index bridge.
- Container: gate `P-` + selector `W-` → left-middle bridge.
- System: gate `T-` + selector `K-` → left-ring bridge.

This is deliberate. In Latched Control the `STPH` gate disappears, so these become ordinary single-key family selectors.

## 2. Representative literal strokes

| Semantic command | Transient | Latched | Keys T/L | Status |
|---|---:|---:|---:|---|
| `text.move.character.previous` | `STPH-R` | `-R` | 5/1 | core |
| `text.move.word.previous` | `STPHAR` | `AR` | 6/2 | core |
| `text.extend.word.next` | `STPHA*G` | `A*G` | 7/3 | core |
| `text.delete.word.previous` | `STPHARD` | `ARD` | 7/3 | core |
| `edit.undo` | `STPH-RT` | `-RT` | 6/2 | core |
| `edit.copy` | `STPH-PT` | `-PT` | 6/2 | core |
| `edit.select_all` | `STPH-RGT` | `-RGT` | 7/3 | core |
| `text.insert.paragraph_break` | `STPHOT` | `OT` | 6/2 | provisional |
| `document.save` | `STPHUT` | `UT` | 6/2 | core |
| `ui.cancel` | `STPHR-RT` | `R-RT` | 7/3 | core |
| `ui.activate` | `STPHR-GT` | `R-GT` | 7/3 | core |
| `view.scroll.page_down` | `STPHREB` | `REB` | 7/3 | provisional |
| `location.back` | `STPHRURS` | `RURS` | 8/4 | provisional |
| `tab.switch.next` | `STPWH-G` | `W-G` | 6/2 | core |
| `window.switch.next` | `STPWHOG` | `WOG` | 7/3 | core |
| `application.switch.next` | `STPWHEG` | `WEG` | 7/3 | core |
| `workspace.switch.next` | `STPWHUG` | `WUG` | 7/3 | core |
| `window.maximize` | `STPWHOPT` | `WOPT` | 8/4 | core |
| `window.place.upper_right` | `STPWHOPGS` | `WOPGS` | 9/5 | core |
| `system.launcher` | `STKPH-T` | `K-T` | 6/2 | core |
| `media.volume_down` | `STKPH-BS` | `K-BS` | 7/3 | core |
| `plover.lookup` | `STKPHARZ` | `KARZ` | 8/4 | core |
| `clavis.safe_reset` | `STKPHEZ` | `KEZ` | 7/3 | core |
| `application.quit` | `STPWHEDZ` | `WEDZ` | 8/4 | core |
| `system.shutdown` | `STKPHUBDZ` | `KUBDZ` | 9/5 | core |

## 3. Collision audit

### 3.1 Internal collisions

No internal collision was found among the 152 transient forms or among the 152 latched payloads.
The two mode namespaces are also disjoint in this prototype.

### 3.2 Plover stock command overlap

Exact candidate overlaps with Plover's stock `commands.json`:
- `STPH-G` → `text.move.character.next`
- `STPH-R` → `text.move.character.previous`

Both exact overlaps are intentional and semantically compatible: Clavis preserves Plover's stock left/right character movement.

More important than the exact overlaps: Plover's stock command dictionary also defines other `STPH...` strokes such as `STPH-P`, `STPH-B`, `STPH-RB`, and `STPH-BG`. The Clavis prototype intentionally does **not** give those forms their Plover meanings. If Clavis were implemented as only a sparse high-priority JSON dictionary, an undefined gated stroke could fall through to those lower dictionaries. That violates the Clavis fail-closed rule.

### 3.3 Di movement overlap

Exact candidate overlaps with Di's navigation dictionary:
- `STPH-G` → `text.move.character.next`
- `STPH-R` → `text.move.character.previous`

Again, the exact overlaps are compatible. Di also defines other `STPH...` movement forms, reinforcing the need for explicit namespace ownership rather than accidental dictionary precedence.

### 3.4 Lexical collision policy

A full collision count against Plover's multi-megabyte default `main.json` is **not treated as a correctness gate for Clavis**, because Clavis deliberately reserves every gated control form from the lexical theory. The default dictionary is also too large to inspect completely through the connected GitHub file interface in this research pass.

The relevant migration question is different: how many familiar Plover outlines would a user give up when adopting Clavis? That should be measured later against a local copy of the baseline dictionaries. It does not change the reserved-namespace rule.

## 4. Fail-closed implementation finding

The existing `plover_modal_dictionary` plugin is useful prior art, but its documented mismatch behavior is not sufficient for Clavis's strict safety requirement: with `exit_on_mismatch`, an unmatched stroke can deactivate the modal dictionary and then receive its ordinary translation.

Therefore the control architecture should eventually include a **Clavis namespace router** (likely a small Plover plugin/programmatic dictionary mechanism) with these properties:
1. any stroke containing the transient Control Gate is intercepted by Clavis before ordinary lexical/command dictionaries can act on it;
2. valid gated strokes execute Clavis semantics;
3. invalid gated strokes produce no host action and no lexical output;
4. in Latched Control, valid payloads execute Clavis semantics;
5. invalid latched payloads also fail closed instead of falling through;
6. the router can expose the same semantic command IDs to platform/application adapters.

This is an architectural requirement discovered by the collision audit, not an implementation detail to postpone indefinitely.

## 5. Mechanical load audit

### Key-count distribution

| Transient logical keys | Commands |
|---:|---:|
| 5 | 4 |
| 6 | 33 |
| 7 | 63 |
| 8 | 44 |
| 9 | 8 |

| Latched logical keys | Commands |
|---:|---:|
| 1 | 4 |
| 2 | 33 |
| 3 | 63 |
| 4 | 44 |
| 5 | 8 |

### By frequency tier

| Tier | N | Transient min/median/mean/max | Latched min/median/mean/max |
|---:|---:|---|---|
| 0 | 52 | 5/7/6.81/8 | 1/3/2.81/4 |
| 1 | 78 | 5/7/7.15/9 | 1/3/3.15/5 |
| 2 | 15 | 6/7/7.4/8 | 2/3/3.4/4 |
| 3 | 7 | 8/9/8.57/9 | 4/5/4.57/5 |

Interpretation:
- ordinary transient commands cluster at 6–8 logical keys because the four-key gate is simultaneous with the payload;
- Latched Control reduces the median command to about three logical keys;
- Tier 3 guarded commands are deliberately heavier;
- key count alone overstates some burden because family selectors deliberately bridge a key already held by the same finger.

### Highest-load forms

| Command | Transient | Keys | Finger groups | Comment |
|---|---|---:|---:|---|
| `window.place.upper_left` | `STPWHORPS` | 9 | 8 | corner placement; field-test comfort |
| `window.place.upper_right` | `STPWHOPGS` | 9 | 8 | corner placement; field-test comfort |
| `window.place.lower_left` | `STPWHORBS` | 9 | 8 | corner placement; field-test comfort |
| `window.place.lower_right` | `STPWHOBGS` | 9 | 8 | corner placement; field-test comfort |
| `system.sign_out` | `STKPHURDZ` | 9 | 7 | guarded; high load intentional |
| `system.restart` | `STKPHUGDZ` | 9 | 7 | guarded; high load intentional |
| `system.sleep` | `STKPHUPDZ` | 9 | 7 | guarded; high load intentional |
| `system.shutdown` | `STKPHUBDZ` | 9 | 7 | guarded; high load intentional |

The four window-corner strokes are the only prototype commands that engage eight finger groups at once. They should remain **provisional** until tested on a real writer. If uncomfortable, quarter placement should become either:
- a Latched-Control optimization; or
- a two-step `place quadrant / direction` operation.

The 9-key guarded power strokes are acceptable as a design goal because deliberate physical weight is part of their safety function.

## 6. Same-finger bridge audit

Expected bridges in transient mode:
- Interface commands: left `H/R` bridge.
- Container commands: left `P/W` bridge.
- System commands: left `T/K` bridge.
- Boundary commands: right `F/R` or `L/G` bridge.
- Mute candidate: right `P/B` bridge.
- Guarded commands: right `D/Z` bridge.

These are all two-key vertical/adjacent relationships that standard steno technique can represent. No prototype stroke requires three logical keys from one finger group.

In Latched Control the family-gate bridges vanish; only boundary, mute, and guarded bridges remain.

## 7. Audit-driven corrections to the previous design

### 7.1 Add explicit paragraph and soft-line breaks

The semantic inventory required them, but the named-action palette had not assigned them.

Provisional mapping:
- `CONTENT + line scope + PRIMARY + center` → paragraph/newline
- `CONTENT + line scope + SECONDARY + center` → soft line break

Candidate transient strokes: `STPHOT` and `STPHOS`.

### 7.2 Refine Interface scope to preserve page-scale scrolling

The previous Interface sketch did not cleanly distinguish line scrolling from page/screen scrolling.

Prototype refinement:
- scope `O` → ordinary view/line-scale scrolling and zoom palette
- scope `E` → page/screen-scale scrolling and broad view boundaries
- scope `U` + SECONDARY → location/history palette

This moves LOCATION from the provisional `E` slot to `U`. That change should remain provisional until workflow testing confirms it.

## 8. Important unresolved semantic gaps

The following remain deliberately unmapped or under-specified:
- sending a window to another **physical display**;
- application launch/activation by configured name;
- direct indexed tab/workspace selection;
- pointer/mouse movement, click, drag, and scroll-wheel semantics;
- raw host modifier/key escape hatch details;
- file/object operations such as rename, properties, and permanent delete;
- a general `repeat previous Clavis action` stroke;
- an explicit count-prefix syntax tied to the future Clavis number theory.

These should not be patched with arbitrary briefs. Each needs its own compositional rule or a clear extension boundary.

## 9. Workflow stress tests

### Editing a sentence in Writing State

1. `STPHAR` — text.move.word.previous
2. `STPHA*G` — text.extend.word.next
3. `STPH-T` — edit.cut
4. `STPH-BS` — edit.paste_plain
5. `STPHUT` — document.save

Every action is one transient stroke. The patterns remain visible: scope/direction for movement and selection, primary/secondary pads for editing, document scope for save.

### Repeated editing in Latched Control

Enter with gate-alone `STPH`, then:
1. `AR` — text.move.word.previous
2. `AR` — text.move.word.previous
3. `A*G` — text.extend.word.next
4. `-PT` — edit.copy
5. `EG` — text.move.paragraph.next
6. `-BT` — edit.paste
Exit with gate-alone `STPH`.

Latched mode costs two extra strokes for entry/exit, but dramatically reduces chord weight during a sustained editing passage. It is an ergonomic mode, not a stroke-count trick.

### Browser/research workflow

- `STPWH-G` — tab.switch.next
- `STPH-S` — search.find_open
- `STPH-GS` — search.next_match
- `STPH-PT` — edit.copy
- `STPWHEG` — application.switch.next
- `STPH-BT` — edit.paste

### Window-management workflow

- `STPWHEG` — application.switch.next
- `STPWHOGS` — window.place.right_half
- `STPWH*UG` — window.send.workspace_next
- `STPWHUG` — workspace.switch.next

The quarter-window forms are intentionally omitted from this quick workflow because they are the principal ergonomic field-test candidates.

## 10. External compatibility observations

- Plover's stock command dictionary already uses `STPH-R`/`STPH-G` for left/right movement, so those two Clavis forms preserve existing muscle memory.
- Plover also uses other `STPH...` movement forms; this is exactly why Clavis must reserve the whole gate namespace rather than rely on sparse dictionary priority.
- Di's navigation system independently uses the same `STPH` and right-hand movement geometry, supporting the decision to keep the direction diamond.
- Some supported hobbyist writers omit a dedicated number bar, so no core command in this prototype requires one.
- Commodity QWERTY keyboards without NKRO may struggle with the larger chords; this is a hardware rollover limitation rather than an ambiguity in the theory.

## 11. Freeze / revise / reject decisions

### Freeze
- actual `STPH` transient gate namespace;
- right-hand `R/P/B/G` direction geometry;
- `FR`/`LG` boundary forms;
- vowel small→large scope grammar;
- family selectors `R/W/K` under the gate;
- `T/S/D/Z/DZ` operation rail;
- transient and latched payload isomorphism;
- strict fail-closed namespace ownership;
- semantic command IDs separate from adapter implementations.

### Keep provisional
- Interface `O/E/U` scope refinement;
- paragraph/soft-break placement on line scope center actions;
- `P+B` mute bridge;
- quarter-window one-stroke forms;
- exact guarded confirmation gesture;
- system capture scope details.

### Reject
- implementing Clavis solely as a sparse ordinary JSON dictionary;
- allowing unknown gated or latched strokes to fall through to lexical dictionaries;
- using right-pinky operation keys as routine modifiers of one another.

## 12. Next engineering step

Before executable dictionaries are produced, the next pass should specify the **Clavis control runtime contract**:
1. exact Writing / Transient / Latched / Guarded state transitions;
2. how the Plover integration intercepts gated strokes and mismatches;
3. canonical semantic command IDs and adapter return states (`native`, `derived`, `contextual`, `unsupported`);
4. formatting-state preservation after movement/editing commands;
5. confirmation/cancellation rules for Class C actions;
6. deterministic tests proving no control stroke can leak into lexical output.

Once that runtime boundary is specified, Clavis can safely generate its first executable control dictionary/plugin prototype.

## Research references

- Plover stock commands: https://github.com/opensteno/plover-dict/blob/main/commands.json
- Plover Modal Dictionary: https://github.com/Kaoffie/plover_modal_dictionary
- Di's navigation dictionaries: https://github.com/didoesdigital/steno-dictionaries
- Plover supported hardware: https://plover.wiki/index.php/Supported_hardware
- Plover beginner keyboard mapping/finger placement: https://plover.wiki/index.php/Beginner%27s_Guide/en

## Appendix A — Complete generated prototype

The companion CSV is preferable for machine processing. The table below is the human-readable audit record.

| Semantic command | Transient | Latched | Fam. | Scope | Op. | Pos. | A/C | Tier | Status | T/L keys | Bridge(s) |
|---|---|---|---|---|---|---|:---:|---:|---|---:|---|
| `text.move.character.previous` | `STPH-R` | `-R` | content | 0 | move | west | N | 0 | core | 5/1 | — |
| `text.move.character.next` | `STPH-G` | `-G` | content | 0 | move | east | N | 0 | core | 5/1 | — |
| `text.move.word.previous` | `STPHAR` | `AR` | content | A | move | west | N | 0 | core | 6/2 | — |
| `text.move.word.next` | `STPHAG` | `AG` | content | A | move | east | N | 0 | core | 6/2 | — |
| `text.move.line.up` | `STPHOP` | `OP` | content | O | move | north | N | 0 | core | 6/2 | — |
| `text.move.line.down` | `STPHOB` | `OB` | content | O | move | south | N | 0 | core | 6/2 | — |
| `text.move.line.start` | `STPHOFR` | `OFR` | content | O | move | west_boundary | N | 0 | core | 7/3 | R index |
| `text.move.line.end` | `STPHOLG` | `OLG` | content | O | move | east_boundary | N | 0 | core | 7/3 | R ring |
| `text.move.paragraph.previous` | `STPHER` | `ER` | content | E | move | west | N | 0 | core | 6/2 | — |
| `text.move.paragraph.next` | `STPHEG` | `EG` | content | E | move | east | N | 0 | core | 6/2 | — |
| `text.move.document.start` | `STPHUFR` | `UFR` | content | U | move | west_boundary | N | 0 | core | 7/3 | R index |
| `text.move.document.end` | `STPHULG` | `ULG` | content | U | move | east_boundary | N | 0 | core | 7/3 | R ring |
| `text.extend.character.previous` | `STPH*R` | `*R` | content | 0 | move | west | Y | 0 | core | 6/2 | — |
| `text.extend.character.next` | `STPH*G` | `*G` | content | 0 | move | east | Y | 0 | core | 6/2 | — |
| `text.extend.word.previous` | `STPHA*R` | `A*R` | content | A | move | west | Y | 0 | core | 7/3 | — |
| `text.extend.word.next` | `STPHA*G` | `A*G` | content | A | move | east | Y | 0 | core | 7/3 | — |
| `text.extend.line.up` | `STPHO*P` | `O*P` | content | O | move | north | Y | 0 | core | 7/3 | — |
| `text.extend.line.down` | `STPHO*B` | `O*B` | content | O | move | south | Y | 0 | core | 7/3 | — |
| `text.extend.line.start` | `STPHO*FR` | `O*FR` | content | O | move | west_boundary | Y | 0 | core | 8/4 | R index |
| `text.extend.line.end` | `STPHO*LG` | `O*LG` | content | O | move | east_boundary | Y | 0 | core | 8/4 | R ring |
| `text.extend.paragraph.previous` | `STPH*ER` | `*ER` | content | E | move | west | Y | 0 | core | 7/3 | — |
| `text.extend.paragraph.next` | `STPH*EG` | `*EG` | content | E | move | east | Y | 0 | core | 7/3 | — |
| `text.extend.document.start` | `STPH*UFR` | `*UFR` | content | U | move | west_boundary | Y | 0 | core | 8/4 | R index |
| `text.extend.document.end` | `STPH*ULG` | `*ULG` | content | U | move | east_boundary | Y | 0 | core | 8/4 | R ring |
| `text.delete.character.previous` | `STPH-RD` | `-RD` | content | 0 | remove | west | N | 1 | core | 6/2 | — |
| `text.delete.character.next` | `STPH-GD` | `-GD` | content | 0 | remove | east | N | 1 | core | 6/2 | — |
| `text.delete.word.previous` | `STPHARD` | `ARD` | content | A | remove | west | N | 1 | core | 7/3 | — |
| `text.delete.word.next` | `STPHAGD` | `AGD` | content | A | remove | east | N | 1 | core | 7/3 | — |
| `text.delete.to_line_start` | `STPHOFRD` | `OFRD` | content | O | remove | west_boundary | N | 1 | core | 8/4 | R index |
| `text.delete.to_line_end` | `STPHOLGD` | `OLGD` | content | O | remove | east_boundary | N | 1 | core | 8/4 | R ring |
| `text.insert.paragraph_break` | `STPHOT` | `OT` | content | O | primary | center | N | 1 | provisional | 6/2 | — |
| `text.insert.soft_line_break` | `STPHOS` | `OS` | content | O | secondary | center | N | 1 | provisional | 6/2 | — |
| `edit.undo` | `STPH-RT` | `-RT` | content | 0 | primary | west | N | 1 | core | 6/2 | — |
| `edit.redo` | `STPH-GT` | `-GT` | content | 0 | primary | east | N | 1 | core | 6/2 | — |
| `edit.copy` | `STPH-PT` | `-PT` | content | 0 | primary | north | N | 1 | core | 6/2 | — |
| `edit.paste` | `STPH-BT` | `-BT` | content | 0 | primary | south | N | 1 | core | 6/2 | — |
| `edit.cut` | `STPH-T` | `-T` | content | 0 | primary | center | N | 1 | core | 5/1 | — |
| `edit.select_all` | `STPH-RGT` | `-RGT` | content | 0 | primary | west_east | N | 1 | core | 7/3 | — |
| `search.previous_match` | `STPH-RS` | `-RS` | content | 0 | secondary | west | N | 1 | core | 6/2 | — |
| `search.next_match` | `STPH-GS` | `-GS` | content | 0 | secondary | east | N | 1 | core | 6/2 | — |
| `search.replace_open` | `STPH-PS` | `-PS` | content | 0 | secondary | north | N | 1 | core | 6/2 | — |
| `edit.paste_plain` | `STPH-BS` | `-BS` | content | 0 | secondary | south | N | 1 | core | 6/2 | — |
| `search.find_open` | `STPH-S` | `-S` | content | 0 | secondary | center | N | 1 | core | 5/1 | — |
| `document.open` | `STPHURT` | `URT` | content | U | primary | west | N | 1 | core | 7/3 | — |
| `document.save_as` | `STPHUGT` | `UGT` | content | U | primary | east | N | 1 | core | 7/3 | — |
| `document.new` | `STPHUPT` | `UPT` | content | U | primary | north | N | 1 | core | 7/3 | — |
| `document.print` | `STPHUBT` | `UBT` | content | U | primary | south | N | 1 | core | 7/3 | — |
| `document.save` | `STPHUT` | `UT` | content | U | primary | center | N | 1 | core | 6/2 | — |
| `document.close` | `STPHUD` | `UD` | content | U | remove | center | N | 1 | core | 6/2 | — |
| `ui.focus.previous_control` | `STPHR-R` | `R-R` | interface | 0 | move | west | N | 0 | core | 6/2 | L index |
| `ui.focus.next_control` | `STPHR-G` | `R-G` | interface | 0 | move | east | N | 0 | core | 6/2 | L index |
| `ui.focus.up` | `STPHR-P` | `R-P` | interface | 0 | move | north | N | 0 | core | 6/2 | L index |
| `ui.focus.down` | `STPHR-B` | `R-B` | interface | 0 | move | south | N | 0 | core | 6/2 | L index |
| `ui.focus.previous_region` | `STPHRAR` | `RAR` | interface | A | move | west | N | 0 | core | 7/3 | L index |
| `ui.focus.next_region` | `STPHRAG` | `RAG` | interface | A | move | east | N | 0 | core | 7/3 | L index |
| `view.scroll.line_up` | `STPHROP` | `ROP` | interface | O | move | north | N | 0 | provisional | 7/3 | L index |
| `view.scroll.line_down` | `STPHROB` | `ROB` | interface | O | move | south | N | 0 | provisional | 7/3 | L index |
| `view.scroll.page_up` | `STPHREP` | `REP` | interface | E | move | north | N | 0 | provisional | 7/3 | L index |
| `view.scroll.page_down` | `STPHREB` | `REB` | interface | E | move | south | N | 0 | provisional | 7/3 | L index |
| `view.scroll.start` | `STPHREFR` | `REFR` | interface | E | move | west_boundary | N | 0 | provisional | 8/4 | L index, R index |
| `view.scroll.end` | `STPHRELG` | `RELG` | interface | E | move | east_boundary | N | 0 | provisional | 8/4 | L index, R ring |
| `ui.cancel` | `STPHR-RT` | `R-RT` | interface | 0 | primary | west | N | 1 | core | 7/3 | L index |
| `ui.activate` | `STPHR-GT` | `R-GT` | interface | 0 | primary | east | N | 1 | core | 7/3 | L index |
| `ui.expand` | `STPHR-PT` | `R-PT` | interface | 0 | primary | north | N | 1 | core | 7/3 | L index |
| `ui.collapse` | `STPHR-BT` | `R-BT` | interface | 0 | primary | south | N | 1 | core | 7/3 | L index |
| `ui.toggle` | `STPHR-T` | `R-T` | interface | 0 | primary | center | N | 1 | core | 6/2 | L index |
| `view.zoom_in` | `STPHROPS` | `ROPS` | interface | O | secondary | north | N | 1 | core | 8/4 | L index |
| `view.zoom_out` | `STPHROBS` | `ROBS` | interface | O | secondary | south | N | 1 | core | 8/4 | L index |
| `view.zoom_reset` | `STPHROS` | `ROS` | interface | O | secondary | center | N | 1 | core | 7/3 | L index |
| `location.back` | `STPHRURS` | `RURS` | interface | U | secondary | west | N | 1 | provisional | 8/4 | L index |
| `location.forward` | `STPHRUGS` | `RUGS` | interface | U | secondary | east | N | 1 | provisional | 8/4 | L index |
| `location.focus_field` | `STPHRUPS` | `RUPS` | interface | U | secondary | north | N | 1 | provisional | 8/4 | L index |
| `location.up_level` | `STPHRUBS` | `RUBS` | interface | U | secondary | south | N | 1 | provisional | 8/4 | L index |
| `location.reload` | `STPHRUS` | `RUS` | interface | U | secondary | center | N | 1 | provisional | 7/3 | L index |
| `tab.switch.previous` | `STPWH-R` | `W-R` | container | 0 | move | west | N | 0 | core | 6/2 | L middle |
| `tab.switch.next` | `STPWH-G` | `W-G` | container | 0 | move | east | N | 0 | core | 6/2 | L middle |
| `pane.switch.previous` | `STPWHAR` | `WAR` | container | A | move | west | N | 0 | core | 7/3 | L middle |
| `pane.switch.next` | `STPWHAG` | `WAG` | container | A | move | east | N | 0 | core | 7/3 | L middle |
| `window.switch.previous` | `STPWHOR` | `WOR` | container | O | move | west | N | 0 | core | 7/3 | L middle |
| `window.switch.next` | `STPWHOG` | `WOG` | container | O | move | east | N | 0 | core | 7/3 | L middle |
| `application.switch.previous` | `STPWHER` | `WER` | container | E | move | west | N | 0 | core | 7/3 | L middle |
| `application.switch.next` | `STPWHEG` | `WEG` | container | E | move | east | N | 0 | core | 7/3 | L middle |
| `workspace.switch.previous` | `STPWHUR` | `WUR` | container | U | move | west | N | 0 | core | 7/3 | L middle |
| `workspace.switch.next` | `STPWHUG` | `WUG` | container | U | move | east | N | 0 | core | 7/3 | L middle |
| `tab.switch.first` | `STPWH-FR` | `W-FR` | container | 0 | move | west_boundary | N | 0 | recommended | 7/3 | L middle, R index |
| `tab.switch.last` | `STPWH-LG` | `W-LG` | container | 0 | move | east_boundary | N | 0 | recommended | 7/3 | L middle, R ring |
| `workspace.switch.first` | `STPWHUFR` | `WUFR` | container | U | move | west_boundary | N | 0 | recommended | 8/4 | L middle, R index |
| `workspace.switch.last` | `STPWHULG` | `WULG` | container | U | move | east_boundary | N | 0 | recommended | 8/4 | L middle, R ring |
| `tab.move.previous` | `STPWH*R` | `W*R` | container | 0 | move | west | Y | 0 | core | 7/3 | L middle |
| `tab.move.next` | `STPWH*G` | `W*G` | container | 0 | move | east | Y | 0 | core | 7/3 | L middle |
| `window.send.workspace_previous` | `STPWH*UR` | `W*UR` | container | U | move | west | Y | 1 | core | 8/4 | L middle |
| `window.send.workspace_next` | `STPWH*UG` | `W*UG` | container | U | move | east | Y | 1 | core | 8/4 | L middle |
| `tab.reopen` | `STPWH-RT` | `W-RT` | container | 0 | primary | west | N | 1 | core | 7/3 | L middle |
| `tab.new` | `STPWH-GT` | `W-GT` | container | 0 | primary | east | N | 1 | core | 7/3 | L middle |
| `pane.new_default_split` | `STPWHAGT` | `WAGT` | container | A | primary | east | N | 1 | recommended | 8/4 | L middle |
| `pane.collapse` | `STPWHABT` | `WABT` | container | A | primary | south | N | 1 | recommended | 8/4 | L middle |
| `window.restore` | `STPWHORT` | `WORT` | container | O | primary | west | N | 1 | core | 8/4 | L middle |
| `window.new` | `STPWHOGT` | `WOGT` | container | O | primary | east | N | 1 | core | 8/4 | L middle |
| `window.maximize` | `STPWHOPT` | `WOPT` | container | O | primary | north | N | 1 | core | 8/4 | L middle |
| `window.minimize` | `STPWHOBT` | `WOBT` | container | O | primary | south | N | 1 | core | 8/4 | L middle |
| `window.fullscreen_toggle` | `STPWHOT` | `WOT` | container | O | primary | center | N | 1 | recommended | 7/3 | L middle |
| `application.new_instance` | `STPWHEGT` | `WEGT` | container | E | primary | east | N | 1 | recommended | 8/4 | L middle |
| `workspace.create` | `STPWHUGT` | `WUGT` | container | U | primary | east | N | 1 | recommended | 8/4 | L middle |
| `tab.close` | `STPWH-D` | `W-D` | container | 0 | remove | center | N | 1 | core | 6/2 | L middle |
| `pane.close` | `STPWHAD` | `WAD` | container | A | remove | center | N | 1 | recommended | 7/3 | L middle |
| `window.close` | `STPWHOD` | `WOD` | container | O | remove | center | N | 1 | core | 7/3 | L middle |
| `workspace.close` | `STPWHUD` | `WUD` | container | U | remove | center | N | 1 | recommended | 7/3 | L middle |
| `window.place.left_half` | `STPWHORS` | `WORS` | container | O | secondary | west | N | 1 | core | 8/4 | L middle |
| `window.place.right_half` | `STPWHOGS` | `WOGS` | container | O | secondary | east | N | 1 | core | 8/4 | L middle |
| `window.place.top_half` | `STPWHOPS` | `WOPS` | container | O | secondary | north | N | 1 | core | 8/4 | L middle |
| `window.place.bottom_half` | `STPWHOBS` | `WOBS` | container | O | secondary | south | N | 1 | core | 8/4 | L middle |
| `window.place.restore` | `STPWHOS` | `WOS` | container | O | secondary | center | N | 1 | core | 7/3 | L middle |
| `window.place.upper_left` | `STPWHORPS` | `WORPS` | container | O | secondary | nw | N | 1 | core | 9/5 | L middle |
| `window.place.upper_right` | `STPWHOPGS` | `WOPGS` | container | O | secondary | ne | N | 1 | core | 9/5 | L middle |
| `window.place.lower_left` | `STPWHORBS` | `WORBS` | container | O | secondary | sw | N | 1 | core | 9/5 | L middle |
| `window.place.lower_right` | `STPWHOBGS` | `WOBGS` | container | O | secondary | se | N | 1 | core | 9/5 | L middle |
| `system.show_desktop` | `STKPH-RT` | `K-RT` | system | 0 | primary | west | N | 1 | core | 7/3 | L ring |
| `system.notifications` | `STKPH-GT` | `K-GT` | system | 0 | primary | east | N | 1 | core | 7/3 | L ring |
| `capture.region` | `STKPH-PT` | `K-PT` | system | 0 | primary | north | N | 1 | core | 7/3 | L ring |
| `system.clipboard_history` | `STKPH-BT` | `K-BT` | system | 0 | primary | south | N | 1 | core | 7/3 | L ring |
| `system.launcher` | `STKPH-T` | `K-T` | system | 0 | primary | center | N | 1 | core | 6/2 | L ring |
| `capture.window` | `STKPHAPT` | `KAPT` | system | A | primary | north | N | 1 | core | 8/4 | L ring |
| `capture.display` | `STKPHOPT` | `KOPT` | system | O | primary | north | N | 1 | core | 8/4 | L ring |
| `capture.all_displays` | `STKPHEPT` | `KEPT` | system | E | primary | north | N | 1 | core | 8/4 | L ring |
| `media.previous` | `STKPH-RS` | `K-RS` | system | 0 | secondary | west | N | 1 | core | 7/3 | L ring |
| `media.next` | `STKPH-GS` | `K-GS` | system | 0 | secondary | east | N | 1 | core | 7/3 | L ring |
| `media.volume_up` | `STKPH-PS` | `K-PS` | system | 0 | secondary | north | N | 1 | core | 7/3 | L ring |
| `media.volume_down` | `STKPH-BS` | `K-BS` | system | 0 | secondary | south | N | 1 | core | 7/3 | L ring |
| `media.play_pause` | `STKPH-S` | `K-S` | system | 0 | secondary | center | N | 1 | core | 6/2 | L ring |
| `media.mute_toggle` | `STKPH-PBS` | `K-PBS` | system | 0 | secondary | north_south | N | 1 | provisional | 8/4 | L ring, R middle |
| `plover.suspend` | `STKPH-RZ` | `K-RZ` | system | 0 | tools | west | N | 2 | core | 7/3 | L ring |
| `plover.resume` | `STKPH-GZ` | `K-GZ` | system | 0 | tools | east | N | 2 | core | 7/3 | L ring |
| `plover.focus` | `STKPH-PZ` | `K-PZ` | system | 0 | tools | north | N | 2 | core | 7/3 | L ring |
| `plover.configure` | `STKPH-BZ` | `K-BZ` | system | 0 | tools | south | N | 2 | core | 7/3 | L ring |
| `plover.toggle` | `STKPH-Z` | `K-Z` | system | 0 | tools | center | N | 2 | core | 6/2 | L ring |
| `plover.lookup` | `STKPHARZ` | `KARZ` | system | A | tools | west | N | 2 | core | 8/4 | L ring |
| `plover.suggestions` | `STKPHAGZ` | `KAGZ` | system | A | tools | east | N | 2 | core | 8/4 | L ring |
| `plover.dictionary_manager` | `STKPHAPZ` | `KAPZ` | system | A | tools | north | N | 2 | recommended | 8/4 | L ring |
| `plover.reload_dictionaries` | `STKPHABZ` | `KABZ` | system | A | tools | south | N | 2 | recommended | 8/4 | L ring |
| `plover.add_translation` | `STKPHAZ` | `KAZ` | system | A | tools | center | N | 2 | core | 7/3 | L ring |
| `plover.configure.application_scope` | `STKPHOZ` | `KOZ` | system | O | tools | center | N | 2 | recommended | 7/3 | L ring |
| `clavis.help` | `STKPHERZ` | `KERZ` | system | E | tools | west | N | 2 | core | 8/4 | L ring |
| `clavis.status` | `STKPHEGZ` | `KEGZ` | system | E | tools | east | N | 2 | core | 8/4 | L ring |
| `clavis.reload` | `STKPHEPZ` | `KEPZ` | system | E | tools | north | N | 2 | core | 8/4 | L ring |
| `clavis.safe_reset` | `STKPHEZ` | `KEZ` | system | E | tools | center | N | 2 | core | 7/3 | L ring |
| `application.quit` | `STPWHEDZ` | `WEDZ` | container | E | guard | center | N | 3 | core | 8/4 | L middle, R pinky |
| `plover.quit` | `STKPHODZ` | `KODZ` | system | O | guard | center | N | 3 | core | 8/4 | L ring, R pinky |
| `system.sign_out` | `STKPHURDZ` | `KURDZ` | system | U | guard | west | N | 3 | core | 9/5 | L ring, R pinky |
| `system.restart` | `STKPHUGDZ` | `KUGDZ` | system | U | guard | east | N | 3 | core | 9/5 | L ring, R pinky |
| `system.sleep` | `STKPHUPDZ` | `KUPDZ` | system | U | guard | north | N | 3 | core | 9/5 | L ring, R pinky |
| `system.shutdown` | `STKPHUBDZ` | `KUBDZ` | system | U | guard | south | N | 3 | core | 9/5 | L ring, R pinky |
| `system.lock` | `STKPHUDZ` | `KUDZ` | system | U | guard | center | N | 3 | core | 8/4 | L ring, R pinky |
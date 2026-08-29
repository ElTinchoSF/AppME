# Archive Report: generador-notas-fbcb

```yaml
schema: gentle-ai.sdd-archive-report/v1
change: generador-notas-fbcb
project: AppME_GentleAI
archive_date: 2026-08-29
status: success
verify_verdict: pass
verify_evidence_revision: sha256:9898029fb806299d9b303fa7e1267c83517355d8f27514dbc8ceb783dc5f30db
requirements: 42/42
scenarios: 89/89
blockers: 0
review_gate: absent (no review ever started for this candidate; ordinary repository policy)
artifact_store: hybrid (openspec/ files + Engram)
```

## Executive Summary

The SDD cycle for change `generador-notas-fbcb` (offline formal-note generator for FBCB-UNL) is CLOSED. Verify round 2 = **PASS** (42/42 requirements, 89/89 scenarios, 0 blockers, 0 CRITICAL, 0 WARNING) on commit `d85b9ec`, with all 9 code fixes (C1, W1, W2, W4, W6, W7, W8, W10, W11) verified by harness (93/93 checks, exit 0). The 5 delta specs were synced into `openspec/specs/` as the released spec baseline (first SDD change of the project — no prior baseline existed), and the change folder was moved to `openspec/changes/archive/2026-08-29-generador-notas-fbcb/` with mechanical copy (empty `diff -r` readbacks). The app is **LIVE** at https://eltinchosf.github.io/AppME/ (GitHub Pages, installed as PWA-style from repo root); the user confirmed PDF generation/download works in browser. Two accepted design deviations (W3, W9) and 14 non-blocking suggestions (S1–S14) are recorded below so they are not lost.

## Final-State Authority

Ranking per sdd-archive skill Final-State Authority hierarchy:

1. **Native review authority**: no review receipt exists (see Review Gate below) — not applicable.
2. **Persisted tasks artifact**: `tasks.md` — 41 implementation tasks; 40 marked complete; task 3.3 intentionally not implemented (see Task Reconciliation).
3. **Explicit final-state facts (launch prompt, 2026-08-29)**: change COMPLETE; verify round 2 = PASS 42/42 requirements, 89/89 scenarios, 0 blockers, commit `d85b9ec` (all 9 fixes). GitHub Pages LIVE at https://eltinchosf.github.io/AppME/; user confirmed PDF download works in browser.
4. **`verify-report`/`apply-progress`**: `verify-report.md` (round 2) PASS 93/93 checks exit 0 — consistent with rank 3; `apply-progress` (Engram #26) documents completion of the apply phase.

No contradictions between sources; all higher-ranked sources agree with the final state reported here.

## Review Gate

`reviewGate` is structurally **absent** for this candidate: no review artifact was ever created (no `review/` directory in the change folder, no reviewGate value in any status). Receipt-driven development was not exercised for this change, so archive proceeds under ordinary repository policy — there is nothing to read or block on.

## Task Reconciliation (exceptional repair, recorded per skill)

The persisted `openspec/changes/generador-notas-fbcb/tasks.md` (as found pre-archive) showed **all 41 implementation tasks unchecked**, although the implementation was complete: the apply phase was executed by the orchestrator directly (after two `sdd-apply` sub-agents failed with 504 streaming timeouts — Engram #26), and checkboxes were never ticked. Per the Task Completion Gate, archive proceeds under the exceptional-repair path because:

- The orchestrator's launch prompt explicitly asserts the change is COMPLETE with verify round 2 = PASS, tasks implemented (rank-3 final-state facts).
- `verify-report.md` (round 2) and Engram #28 prove every spec-bearing task complete: 42/42 requirements, 89/89 scenarios, 93/93 harness checks, exit 0; `INSTRUCCIONES.md` exists (3,823 bytes, task 5.5).

Reconciliation applied (before the archive move, so the archived copy is accurate): 40 tasks marked `[x]`; task **3.3** (`clearProfile()`) left `[ ]` with an inline annotation because it is intentionally NOT implemented — verify-report SUGGESTION **S10** confirms no spec requirement depends on it (no-goal v1 helper). It is recorded as not-implemented, not as completed.

## Spec Sync (delta → released baseline)

This is the FIRST SDD change of the project: `openspec/specs/` was empty, so all 5 delta specs are FULL specs (no existing main spec to merge into) and become the released spec set verbatim. Mechanical copy only (cp → temp → `diff -r` readback → mv); no model Read/Write of artifact bytes.

| Domain | Action | Released path |
|--------|--------|---------------|
| note-generator | Created (full spec) | `openspec/specs/note-generator/spec.md` |
| authority-registry | Created (full spec) | `openspec/specs/authority-registry/spec.md` |
| sender-profile | Created (full spec) | `openspec/specs/sender-profile/spec.md` |
| pdf-export | Created (full spec) | `openspec/specs/pdf-export/spec.md` |
| native-share | Created (full spec) | `openspec/specs/native-share/spec.md` |

Readbacks: `diff -r` source vs temp = empty (PASS) for all 5 domains, verbatim in phase result. No REMOVED/RENAMED/MODIFIED deltas, no destructive merge; config.yaml archive rule "Warn before merging destructive deltas" not triggered.

## Archive Move

- Source: `openspec/changes/generador-notas-fbcb/` → Target: `openspec/changes/archive/2026-08-29-generador-notas-fbcb/` (ISO date prefix), via `git mv` (succeeded; includes untracked `verify-report.md`).
- Snapshot taken pre-move; source-gone check passed; **`diff -r` snapshot vs archived = empty (PASS)** — verbatim in phase result.
- Active changes dir now contains only `archive/`.
- Archived contents: `proposal.md`, `specs/` (5 domains), `design.md`, `tasks.md`, `verify-report.md` + this additive `archive-report.md`. Nothing deleted or modified after move.

## Accepted Design Deviations (user-accepted, from verify-report round 2)

- **W3 — Authority lookup by `id` instead of by name**: spec `authority-registry` #6 requires lookup by exact name; implementation uses `findAuthority` by stable `id` (e.g. `decano-fbcb`). Rationale: id lookup is more robust for the UI selector (names are display data), and the registry carries the official 2026-2030 roster (e.g. "Dr. Guillermo Ramos" Decano FBCB) rather than the fictitious names from the spec reference tables. Accepted by user decision; spec requirement satisfied by equivalent behavior (find + return undefined for unknown key).
- **W9 — Note body does not vary per sender profile**: spec `sender-profile` #1 requires the body text to adapt to profile (Alumno/Docente/Personal/Otro); implementation varies the SIGNATURE line only (`CONDICION_PERFIL` adds "Alumno/a — Facultad de Bioquímica y Ciencias Biológicas"), while the body generators do not take the profile parameter. Rationale: body redaction in first person ("mi inscripción…") is profile-neutral in the official templates; per-profile signature is the meaningful differentiator. Accepted by user decision; design.md `generar(d)` without `perfil` param flagged as ⚠️ in verify report's Design Coherence.

## Follow-ups (non-blocking, from verify-report round 2)

- **S1** — HTML injection (self-XSS) risk in preview (`vContenido.innerHTML` with dynamic field values); escape values to remove the vector.
- **S2** — Preview edits not invalidated when the form changes (`estado.prev` stale); invalidate in `actualizar()`.
- **S3** — Semantic HTML incomplete: task 1.3 asked for `<header>`/`<main>`; current markup uses h1 + divs.
- **S4** — Missing-library error path: no `console.error` and buttons not disabled (spec pdf-export #1 expected both); shows aria-live and returns false.
- **S5** — Validation message literals differ from spec (e.g. "El DNI debe tener 7 u 8 dígitos." vs spec literal); functionally equivalent.
- **S6** — Extra condition line in signature ("Alumno/a — Facultad…") not in the spec schema; additive.
- **S7** — Default profile: spec assumes "Alumno"; implementation requires explicit selection.
- **S8** — Share call has no `text` and uses `title: nombre` instead of "Nota FBCB: [Asunto]".
- **S9** — Spec-internal inconsistency (sender-profile #2): example "+54 342 123 4567" (16 chars) violates the 7–15 constraint; implementation correctly rejects it (documented DIFF).
- **S10** — `clearProfile()` (task 3.3) not implemented; no-goal v1, no functional impact (see Task Reconciliation).
- **S11** — Dynamic fields lack HTML `required` attribute; equivalent manual validation (form `novalidate`).
- **S12 — (UX impact) filename staleness**: `actualizarNombreArchivo` does not regenerate the auto filename when the subject changes if value + `dataset.auto='1'`; also a manual edit is overwritten on the next form change. Suggest: regenerate when auto and state changed; preserve manual name until cycle restart.
- **S13 — (cosmetic) optgroup label**: selector label "Rectorado — Universidad Nacional del Litoral" vs new destinatario wording "Rectorado de la Universidad Nacional del Litoral".
- **S14 — (UX impact) literal drift**: single alert for both share fallbacks (spec expects two distinct messages), "Se descargó el PDF." vs "Se descargará el PDF.", and button labels "Generando..."/"Compartir" vs spec " Generando..."/"🔗 Compartir". Functionally equivalent.

Recommended priority: **S12** and **S14** (direct UX impact); **S1** and **S2** if the app ever renders unpreviewed user input.

## OpenSpec Status Ledger

The OpenSpec convention defines no separate status/ledger file beyond the archive folder itself (`openspec/changes/archive/`); no `openspec/status` or changes index exists in this repo. The archived folder IS the ledger entry for this change. Note: no `state.yaml` was present in the change folder at archive time (orchestrator-owned per convention).

## Engram Traceability (hybrid mode — observation IDs read)

| Artifact | Observation | Project |
|----------|-------------|---------|
| proposal | #22 | appme web - gentleai |
| spec (5 specs) | #23 | mgalanti86 |
| design | #24 | mgalanti86 |
| tasks | #25 | mgalanti86 |
| apply-progress | #26 | mgalanti86 |
| verify-report (round 2) | #28 | mgalanti86 |
| bugfix (9 fixes) | #29 | mgalanti86 |

Archive report persisted as Engram topic `sdd/generador-notas-fbcb/archive-report`.

## Git State (post-archive, NOT committed — per instruction)

`git status` shows: deleted staged renames for the archived change folder (via `git mv`), untracked `openspec/changes/archive/2026-08-29-generador-notas-fbcb/` (includes `verify-report.md` and `archive-report.md`), and untracked `openspec/specs/` (5 released specs). No commit was made. To commit: `git add -A openspec/` then a conventional commit (e.g. `docs(sdd): archivar cambio generador-notas-fbcb y sincronizar specs liberadas`). The app itself (`index.html`, `lib/`, `INSTRUCCIONES.md`) is already committed at `d85b9ec` + prior commits and published on GitHub Pages.

## Released Spec Paths

- `openspec/specs/note-generator/spec.md`
- `openspec/specs/authority-registry/spec.md`
- `openspec/specs/sender-profile/spec.md`
- `openspec/specs/pdf-export/spec.md`
- `openspec/specs/native-share/spec.md`

## Next Recommended

- **None (SDD cycle complete)** for this change.
- Project-level: commit the archive + released specs; optionally open follow-up tasks for S12/S14 (UX) and S1/S2 (robustness) in a new SDD change; the app is live on GitHub Pages, so any future change must preserve the offline-first single-file constraint and be re-verified in browser (mobile share sheet + airplane mode remain untested on real devices per verify-report).
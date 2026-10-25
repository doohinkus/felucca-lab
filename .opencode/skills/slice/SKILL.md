---
name: slice
description: The end-to-end gate for an engines-lab slice (felucca-lab) — the engines repo's golden-hash suite, the fresh felucca build gate with its budget lines, the lab's README ledger, the commit ledgers, and the flash hand-off. Use when implementing an engine (CHAOS, SHEP, WALSH, HARM — or the next one) in the lab.
---

One slice = one engine: code, presets, host phrases, wiring, gate, ledger. The lab's
README.md (ENGINEPLAN: the sketches and the slice order) and the engines repo's README say what
a slice covers — read both before writing anything. The slice's surface is what the plan names,
not a reinterpretation of opportunity. No number is invented here: it is measured, then recorded.
Paths below are lab-root relative; `engines/` and `felucca/` are the two clones.

Every slice has two gates, in this order:

## 1. The engines-repo gate (the engine's own)

From `engines/`: `sh tests/run.sh` — builds the tables and the host, renders the phrases, and
compares each hash with `tests/golden.txt`; it also sweeps the ANALOG cases and (when the
sanitizer compiler is available) runs the sanitized renders and every sample rate's checks.
Standing facts:

- A new engine adds its phrases to `tests/fe_host.c`, so `tests/golden.txt` changes **by design**
  — but the *change* must be deliberate: the new hash set is reviewed over the rendered audio,
  and every other engine's hashes must not move (their phrases re-render bit-for-bit).
- The engine is plain C99, fixed point only (Q15/Q24, no float, no allocation, no library calls),
  built as one translation unit with the host. Table material (`SINE`, `SVF_G`, `TANH_Q15`,
  `PITCH_OCT`) comes from `fe_tables.h`; anything timed in samples or seconds is an FS-gated
  constant (32–96 kHz builds must stay correct — the rate portability rule).
- The voice's state is the contract's box: `ph[0..2]` and `s[0..7]` per voice (the retrigger keep
  bits decide what survives). If the sound needs more, the design shrinks — not the type layout
  (Felucca's: the host keeps it as it is).

## 2. The felucca build gate (the wiring's own)

From `felucca/`: `./build.sh && tests/run_tests.sh`. The header of run_tests.sh is the
authoritative description; the standing machine facts:

- The three budget lines, recorded before/after every slice: image vs the XIP budget (581,044 B),
  `.data+.bss` vs 73,728 B, POOL vs 368,640 B.
- Golden renders change only when the slice meant to change the sound; any golden change must be
  explainable in words before it is accepted into `tests/golden.txt`.
- The machine-cost overruns (fold155, perform, reverb, phys …) are pre-existing on this host
  compiler: the default gate is `SKIP=fold155,perform,reverb,phys` + `CPU_SKIP=1` — read
  run_tests.sh's header on the first lab run and adjust the skip list to what it says, felucca
  has moved since this list was written. `STRICT=1` disables the skips when the machine or its
  flags changed.
- The emu WASM must replay the native build bit-for-bit whenever code lands; `web/`'s own checks
  run separately (`web/test_web.mjs`).

### The fast loop (iterate fast, gate once)

Iterating does NOT need the full suite: select the slice's groups with `ONLY=<comma list>
tests/run_tests.sh` — only they run; the rest prints `== skip <g> (ONLY)`. The per-slice fast
set is the tests the slice's surface can break, **with the golden groups in every set** (the
goldens are what keep the sound). Rules: the FULL run — default, no ONLY — stays the one gate
before a commit; `ONLY` is for iterating, never for walking past a red test (read its output
first), and the fast set grows whenever a red test shows the surface touches more than it
guessed.

## 3. Code conventions

- The firmware is freestanding pi32v2: **no libc headers** — `memcpy` and friends come from
  `libc.c`.
- Comments follow the house voice: dense, factual, parenthesized asides, no marketing. An engine
  cites its lineage in the header comment (the fleet does: Klatt, Rosenberg, Simper, Kellet…).
- Engine wiring follows the EDITOR_PROTOCOL.md rule exactly: **engine numbers are fixed, new
  engines are appended** (both the engine byte and `ENGINE_ORDER`); a lab engine lands with a
  `FELUCCA_ENG_*` flag, on by default here, guarded `#if`-clean both ways like the neighbours
  (`FELUCCA_SLICE`, `FELUCCA_FM4` are the precedents).
- Big tables are **flash consts, never RAM**; the `gen_samples.py` / `gen_engine_tables.py`
  precedents cover generating them with a `tools/gen_*.py` when a table is needed.
- A new render loop gets its own entry in `tests/target_budget.txt`; the device's costs stay
  exact and static.

## 4. Bookkeeping (part of the slice, not an afterthought)

- `README.md`'s ledger (the lab root): a new "**Slice N (done <date>):**" entry under the
  engine's heading — what landed, both gates' receipts (phrase hashes, test-group lines, the
  three budget lines before/after), what the next slice inherits. No number invented.
- ENGINEPLAN wording changes only when the slice moved a decision — say so in the same edit.
- Commits, in order: first in `engines/` — subject `CHAOS (lab slice N): <what the slice did>`
  with a bullet ledger as the body; then in `felucca/` — subject `engines (felucca-lab CHAOS):
  <the wiring>`; then the lab ledger edit with the two gitlink bumps recorded. All trees end
  clean; nothing is staged half-way.

## 5. The flash hand-off

The flashed line is only ever written **from the bench** (hardware is an explicit session, not a
default step). Perform it through the `bench` skill; after it, the session's findings go into the
lab ledger's same slice entry, in the next commit made. A slice entry that claims a flash without
the session behind it is a bug in the dossier.

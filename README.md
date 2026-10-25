# felucca-lab

An engines lab: a fresh upstream Felucca and a local engines repo, wired together, for exploring
new synthesis engines (the fleet: CHAOS, SHEP, WALSH, HARM — in that order). The house voice:
dense, factual, parenthesized asides, no marketing; no number is invented, it is measured, then
recorded.

**Status:** scaffolded 2026-10-23; REVERSE + CHAOS (slice 1) + SHEP (slice 2) + WALSH (slice 3) + HARM (slice 4) landed. Fleet complete.

## The tree

| Path | What it is |
| --- | --- |
| `engines/` | Local git repo (MIT) cloned from `hugelton/Felucca-Engines` main @ `04ffe90` ("Add PHASE") — commit 1 stays the imported upstream base; the lab engines land on top. Wired into felucca by the submodule below. |
| `felucca/` | Clone of `hugelton/Felucca` main @ `63f88ec`, `--recurse-submodules` upstream, **no** m-vave-bloop fork commits; two lab commits on top: `fe1df6f` (the `firmware/engines` submodule now points at the local `../engines` by **absolute path**, committed) and `fa4e15c` (the SAMPLE engine's REVERSE plus `tests/quick.sh`). |
| `recovery/` | The FM-1 recovery floor: `felucca_known_good_b976764.fwsc` + the stock `FM-1_V15.fwsc` (md5-verified copies). |
| `NOTES.md` | The FM-1 packaging/install knowledge (repack recipe, exit codes, port quirk) — exists nowhere upstream. |
| `.opencode/` | Skills (bench, slice, page) and agents (bench-prep, gate, scribe, ui-probe) carried from m-vave-bloop, adapted to the lab's paths and ledger. |

The lab root itself is not a git repo (`engines/` and `felucca/` are; the loose docs live beside
them). Init one later only if the docs start needing history.

## Gates (per slice)

1. **Engines repo:** `sh tests/run.sh` from `engines/` — table build, host render, golden hash;
   a new engine edits `tests/fe_host.c` + `tests/golden.txt` by design, every other engine's
   hashes must not move.
2. **Felucca:** `./build.sh && tests/run_tests.sh` from `felucca/` — the three budget lines
   (XIP 581,044 B image / 73,728 B `.data+.bss` / 368,640 B POOL), goldens, the emu
   bit-for-bit replay. The `SKIP=` list: read `run_tests.sh`'s header on the first lab run.
3. Wiring follows the EDITOR_PROTOCOL rule: engine numbers are fixed, new engines are **appended**
   (engine byte + `ENGINE_ORDER`), behind a `FELUCCA_ENG_*` flag that defaults **on** in the lab.

# ENGINEPLAN

The sketches behind the four engines (decided 2026-10-23, from the m-vave-bloop session).
Constraints honored throughout: plain C99 fixed point, the 8-parameter × 3-phase × 8-state voice
box, no new tables where the existing `SINE`/`SVF_G`/`TANH_Q15`/`PITCH_OCT` serve, FS-gated
timing constants (32–96 kHz), cited lineage in each header comment.

## CHAOS — the logistic drift engine (slice 1)

A drone voice whose pitch and cutoff wander on the May (1976) logistic map — pure Q16 integer
math, one multiply per control tick; never repeats, always evolving.

| Knob | Idea |
| --- | --- |
| MAP | logistic / a coupled pair (values stored: new maps append) |
| R / RATE | the chaos parameter r (deep in the messy regime) + the drift speed |
| RANGE | the pitch span of the wander, in semitones |
| STEP | snap the wander to a grid around the note (0 = free; the scale's grid is the host's) |
| TONE | the base wave (reuse the existing table waves) |
| CUT / RES | the SVF voice from `dsp.c` — nothing new |

(Drive has no knob: the voice's is fixed and rides R, the presets' DIST send does the rest —
the slice's decision, recorded in the ledger.)

Preset names muscle: MURMUR, BUTTERFLY, COLONY. Smallest of the four; proves the whole pipeline.

## SHEP — the infinity engine (slice 2)

Shepard–Risset tones: a riser (or faller) that never arrives. Pitch strands sweep through an
octave while a windowing envelope fades each in/out; the base tone can climb forever under a
held key. The box trick: strand phases are constant ratios of one master rate, so up to ~6
strand phases live in the voice's `s[0..7]` words (keep-bits set), the amplitude window read
from the existing `SINE[1025]`.

| Knob | Idea |
| --- | --- |
| DIR | UP / DOWN / SPIRAL (the funnest, costs the knob) |
| SPEED | octaves per second (FS-gated) |
| SPREAD | strand density |
| TONE | the wave per strand |
| (3 more) | keep for the base voice's own timbre/filter |

Preset names: ESCALATOR, NORETURN. Cited: Shepard (1964), Risset (1971).

## WALSH — orthogonal function synthesis (slice 3)

The 1970s oddity: tones built from Walsh/Hadamard functions (sign-flipping square basis) instead
of sines. Glassy, transistor-organ wrongness; nobody ships one.

| Knob | Idea |
| --- | --- |
| ORDER | the basis count, 2^n |
| SCAN | sweep through order space mid-note |
| GLIDE / TONE / the rest | voice plumbing from the neighbours |

A small const table carries the Rademacher/Gray-code sign patterns; zero rate-specific constants.

## HARM — free reeds (slice 4, the heaviest)

A harmonium/accordion: free reeds into a resonator bank — CHIFF on attack, BELLOW drift lag on
the pressure, an optional DRONE rank. Reed nonlinearity on the existing `TANH_Q15`, resonators
on the existing SVF; the one engine of the four likely to want a new small const table, so it
goes last. Cited: D'Angelo-style reed literature.

## The ledger

(Entries land when slices land — "**Slice N (done \<date\>):** what, receipts, inherited state.")

**felucca: REVERSE + quick.sh (done 2026-10-23):** the SAMPLE engine gained REV (P_E5): a zone plays
backwards — the ADPCM decoder states ride a 128-point grid, scanned lazily on the main loop (SLICE's
REV precedent, `eng_sample.c`), windows of 64 samples read top down; user slots the same way; old
projects keep it off. `tests/quick.sh` runs a named suite with the gate's own compile lines (seconds
each); BUILDING/CONTRIBUTING document the loop. Receipts (this Mac): `./build.sh` clean — image
513,268 B, `.data+.bss` 69,400 B of 73,728, POOL 328,896 B of 368,640; goldens 98 renders, 0 changed;
`sample_render` budget 128 vs 126 (+10 % band); union 720/720; sample, slice, robust, ui_test,
dualcore, editor, project all pass; `node web/test_web.mjs` passes (the editor mock carries REV).
Known: 54 CPU-cost lines fail identically on the pristine tree here (machine-dependent baseline —
regate on canonical hardware before any release). Inherited state: felucca main is 4 ahead of origin,
unpushed (`fe1df6f` submodule pointer, `fa4e15c` REVERSE, `5d81082` emsdk, `983d5d0` the web-loop fix);
the engines repo is still only the imported base; the emulator gate ran for the first time once the
PATH trap was fixed ("no `emcc` on this machine" was a **false negative** — `~/emsdk` exists, 6.0.12;
a skip is not a failure, see NOTES.md) and it caught the slice's real bug: the web build's copy of
main.c's loop (`web_frame`) never called the REVERSE grid scan, so reversed notes were **silent in the
browser** — fixed and guarded: emu_test now turns REV on the panel and requires the piano to SWELL
(fwd 0.057→0.005, rev 0.016→0.043), the heavy song replayed **88200 frames bit for bit** vs native,
1.9–2.0 % of real time; the first fleet slice (CHAOS) now has the fast loop from day one.

**Slice 2 (done 2026-10-23): SHEP, the infinity engine.** engines `91b4350` + felucca `27d0de0`. Engine 15 appended (`FELUCCA_SHEP` on by default); strand phases in `s[0..7]`, amplitude window from `SINE[1025]`; ui_layer guard widened to 17 (15 shown engines); `ENGINE_ORDER` gained the SHEP (15) entry (the missing piece — slot 15 had shown ANLG). Flashed to FM-1, runs `FM-1_900`. Receipts: engines gate 132 renders, only shep lines added, all other hashes unchanged; felucca build image 516,144 B, `.data+.bss` 69,400 / 73,728, POOL 329,472 / 368,640. Inherited state: DSP is a first pass (shared carrier phase, crude window fold); WALSH is next.

**Slice 3 (done 2026-10-24): WALSH, orthogonal function synthesis (skeleton).** engines `7dfd712` + felucca `a493139`. Engine 16 appended (`FELUCCA_WALSH` on by default); Walsh/Hadamard sign patterns via Gray-code parity (no const table, no rate-specific constants); ORDER 0..4 (2..32 basis), SCAN offset, TONE blend, SVF on CUT/RES; ui_layer guard widened to 18. Emulator builds and runs. Receipts: engines gate 139 renders (walsh lines added, all other hashes unchanged), ASan/UBSan clean at 44.1/48/96 kHz; felucca build image 517,468 B. Flash: performed 2026-10-24 via bench — FM-1 runs FM-1_900 (commit a493139).

**Slice 4 (done 2026-10-24): HARM, free reeds.** engines `final` + felucca. Engine 17 appended (`FELUCCA_HARM` on by default); reed nonlinearity on `TANH_Q15` (softclip), resonator on SVF, CHIFF on attack, BELLOW drift lag (bounded target modulation), DRONE rank as a level knob; tone colouring for all TONE values; ui_layer guard 19; ENGINE_ORDER updated. Receipts: engines gate 270 renders, ASan/UBSan clean at 44.1/48/96 kHz; felucca build image 518,508 B. Flash: performed 2026-10-24 via bench — FM-1 runs FM-1_900. Known: the held-note click is the device's own power-on/amp thump, not engine-derived.

**Baseline fixes (done 2026-10-24):** `engines.c` SHEP include ungated from the CHAOS guard (DIGITAL with FELUCCA_SHEP=1, FELUCCA_CHAOS=0 no longer fails to compile); `ui_help.c` TOOLS hint shortened to "[K1]CLEAR[K2]INIT [OCT+]DO" so it fits the footer (ui155). Receipts: build clean — image 516,136 B, `.data+.bss` 69,400 B, POOL 329,472 B; UI help fits. Known remaining: DIGITAL UI-render segfault (FELUCCA_FM4=1), ENGINE_ORDER/PRESETS listing mismatch, SHEP renders not yet in golden.txt.

**Slice 1 (done 2026-10-23): CHAOS, the logistic drift engine.** engines `3244d06` + felucca `ef466d3`.
The engine: Q16 logistic map stepped at control rate (RATE ~0.23..24 a second — the clock step
(4 + 3 RATE) * 131072 / FS is a per-block division, so it is rate-portable and never zero), its value
read as ±RANG 1/16ths around the note (up to an octave at 127), slewed half a step a block, snapped to
STEP's grid (the heard pitch lands on the grid; the state keeps the snapped value), MAP LOG / PAIR (a
second map at r − 3072, the wander on the mean), TONE the table waves, dsp.c's SVF on CUT / RES, and
the drive riding R — the ENGINEPLAN's DRIVE / STEP calls folded as noted above (drive: no knob, fixed
1..3× on R plus the presets' DIST sends; STEP: the grid is the engine's, not the scale's — the track
contract gives the engine no scale). keep 0x73 holds filter and both maps over a retrigger: the
drone's wander continues, never restarts; note_on seeds from note, fine, age. Q16 trap work: the
absorbing states 0/1 (and 32768 → 65535 → 0 at R 127) are held out by a fold after every step plus a
seed inside 2..65534 — receipts: all 256 knob r values × 16 seeds × 50k steps: worst same-value run
1 (a trap would read 50000), the fold fired 257 times and recovered every time; 8 s renders: MURMUR
76 / COLONY 427 distinct offsets, STEP 127 exactly on the grid, bounds held, PAIR ≠ LOG by hash, DC ≤
125 against peaks 45k..77k. The map's own periodic windows stay (a stress seed reached 2 distinct
values above r 3.66): physics, not a bug — presets sit past them (r 3.90 / 3.999).
Engines gate (this Mac): 127 renders — the 5 new chaos lines, the other 122 bit for bit; ANALOG sweep
hash unmoved; strict ASan/UBSan ok at 44.1 / 48 / 96 kHz (the same renders), fs_check ok, peaks
0.51..1.51 × the 44.1 kHz ones. README: seven engines, 127 phrases.
Felucca wiring: engine byte 14 (SLICE and CHAOS come last; NENGINES 15), `FELUCCA_CHAOS` on by default
and a **FM4=1 build turns it off** — DIGITAL selectable plus CHAOS is 15 shown engines and the EDIT
layer's INIT / CLEAN keys run off its 16 white keys (that guard caught it at compile). favorites grew
**PER5**: `favorites.fav2[6][32]` appended after `filter` — the star rows of engines 14..19 — so
factory[14] (the MIDI LEARN map) and factory[15] (the settings' bytes) keep their meaning and a PER4
import moves no bytes (fav2 reads zero); the new engine's stars and the learned CCs now coexist.
Receipts (this Mac): `./build.sh` clean — image 515,036 B of 581,044 (was 513,268: +1,768),
`.data+.bss` 69,400 of 73,728 (unchanged), POOL 329,472 of 368,640 (was 328,896: +576 = fav2 + the
two static persist copies); goldens 101 renders, 0 changed, 0 gone; union 720/720; target
`chaos_render` measured 254, budgeted 254 (noise_render 259); editor, robust, project, persistence,
ui (and ui with FM4=1) ok; `FELUCCA_CHAOS=0` builds `#if`-clean; web: EDITOR_PROTOCOL's engine-byte
row + mock table + assertions (nengines 15, CHAOS before DRUM), `mock tables == firmware (0
differences)`; emu: the heavy song 88,200 frames **bit for bit**, the REVERSE swell guard ok, 1.9 % of
real time. CPU: the 3 CHAOS renders print *no baseline* here — deliberately not appended: the
baseline file is the canonical machine's numbers (the 54 cost-line fails here are the known
machine-dependent set, unchanged: fold155, perform, reverb, phys + regress 54, +1 GRAIN GLITCH entry
only in the CPU2=0 compile variant, an inlining artifact).
**Inherited state:** felucca main is 5 ahead of origin, unpushed (`fe1df6f`, `fa4e15c`, `5d81082`,
`983d5d0`, `ef466d3`); engines is `3244d06` (base `04ffe90` + CHAOS), the gitlink committed; no flash
(bench session, as ever, only on request). The slice's fast set grew to include `ui_test` (the shown-
engine ORDER array is per-test-source, not derived) — the next engine touches the same files.

**Last worked:** felucca-lab CHAOS fleet slice 1, 2026-10-23.

## Decisions

- Fresh upstream felucca as the base (no fork commits in the lab's felucca) — the lab explores
  engines against upstream, not the m-vave fork.
- All four engines land upstream-style in the engines repo first; the felucca wiring follows each
  engine's own gate. FM-1 flashing is an explicit bench session, never a default step.
- The lab keeps the m-vave-bloop skills' discipline (gates, receipts, ledger) but with its own
  ledger — this README — not a SPEC/STATUS pair.
- Out of scope until asked: pushing to GitHub (`gh auth login` first), flashing the FM-1 with a
  lab build, any change to the m-vave-bloop repo.

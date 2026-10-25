---
name: page
description: The procedure for a new panel page in felucca (felucca-lab) — the six-file wiring, the knob-card idiom, the ui_test idiom, and the layout gates. Use when adding a page or when a page's tests fail for layout reasons.
---

A page is a procedure, not a file: the wiring touches six sources, the test idiom is
`ui_test.c`'s, and the layout gates are pixel-exact. **The nearest exemplar page is the
authority** — read it, mirror it, let the gates judge. Find the newest knob page in `params.c`'s
`PAGES[]` at the time of writing; that is the exemplar whose wiring diff you mirror. An
engine-gated page (visible on one engine's tracks only) is shaped by `page_visible`'s existing
cases.

## 1. The wiring, in the PAGES entry first

- `params.c`: append the entry to `PAGES[]` ("TITLE", FAM, scope, the `GR_*` graph group,
  the knob-id mask `{P_*…}` or `0xFF`s when the page is its own), and a new `GR_*` enum member
  **only if** the page is not plain knobs (the mask covers that) — give the immutable knob
  pages `{0xFF,0xFF,0xFF,0xFF}` and handle input inside their own source instead. Annotate the
  entry where its neighbours annotate theirs (the page's file, the doc section).
- `ui.c`: `#include "<page>.c"` **beside its kind's other action/page sources** — and for a page
  that is track-engine-gated, the rule in `page_visible`
  (a shape like `ENGINES[…]==&ENG_SAMPLE`), else it is a visible page of every track.
- Large-card rendering: pages also in `ui.c` `large_kind()`'s label list when their cards are
  label rows, not tall pot rows.
- `ui_draw.c` (the knob cards): a branch in `draw_columns()` per card — the label, the value via
  the page's own label helper, the unit, the dim/gate for an empty/unavailable state, the icon.
  `draw_column` is the primitive; the values are strings built by the page's source
  (`--` for the unavailable state).
- `ui_graph.c`: the graph — a `case GR_X:` in `draw_graph()` calling the page's `graph_*()`
  (the page's drawing primitives are `cv_*`: `cv_rect`, `cv_line`, `cv_text`; the themes are the
  `T_*` palette: DIM the context, TINT/MID the region, ACCENT the live playhead). Reset `cv_oy`
  the way the neighbours do. An empty state says it on screen (`panel_alert`), never blank.
- `ui_input.c`: the knobs — `edit_param()`'s glyph `pg->graph == GR_X && slot < 4u` branch (a
  guard against the page-set on the wrong track), and the exemption spots so the page's
  columns do not fall through to the default knob paths (the graph filter in `mom_turn`,
  `presets_turn` for pages whose card 1 is not a preset, and `ui_input`'s hot-column rule).
  The exemplar's diff is the map of every spot that needed touching.
- `ui_help.c`: the `HELP_PAGE[]` hint line ("[K1]POS[K2]IN…").

## 2. The ui_test idiom (the page's gates)

`tests/ui_test.c` — beside the page-kind's elder test:

- The stub environment is the test's whole world: fixture calls for the track's state
  (`set_engine_of(TSEL, …)` and the track's `p[P_E*]` selection are the usual shapes),
  `go_page(GR_X); ui.force = 1; frame();` to land on the page (the draw is what activates the
  page's routes).
- Knobs are `turn(EN_Kn, detents)`; assertions read the page's own state and the labels via the
  page's label helper. `check("<the behaviour in words>", ok)` accrues `bad`; return `bad`;
  register in `main()`.
- Cover the page's *rules*, not just its happy path: the empty target, the built-in set, the
  wrong engine ("the page is left, its knobs inert"), the clamp/edge behaviour.

## 3. The gates that actually fail, in practice

`./build.sh && tests/run_tests.sh` from `felucca/`; the layout half is the part a new page
redlines first:

- **UI renders** (`tests/ui_render.c`): the lint (no text off-screen, cut, hidden, overlapping,
  or spilling out of its cell/card; free text ellipsised only), the alignment check by ink
  (`build/ui_new/align.txt`: 1 px off fails), every palette, plus this page's screens over its
  values. A new card often fails its first align here — fix the draw, not the checker.
- The text audit (`build/ui_new/text_audit.tsv`) and PNG sheets in `build/ui_new/` are the
  review artefacts, not rubber stamps: look at the page's PNGs before calling the gate green.
- The rest of the suite must stay green too (a page move can surprise `large_kind`, the wave
  budget, the protocol caps).

## 4. Bookkeeping

The `slice` skill carries the whole bookkeeping (the lab ledger entry, the commits in order); a
page-only slice is the usual shape of it — the ledger entry names the page's knobs exactly as the
hint line does.

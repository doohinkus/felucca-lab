# Bench session ledger — DRAFT (pre-flight only, 2026-10-23)

Pre-flight only. No write to the device has occurred.

## Pre-flight checks

- fm1_install.py --info (run from felucca/): EXIT 3 — "FM-1 not found (USB data cable? another app using it?)", MIDI inputs: (none). Device NOT detectable. STOP: sort cable (charge-only cable fails silently — no MIDI ports appear at all). Retry with --port FM1 only if the port is found but the installer's pattern misses it.
- Recovery floor (lab root recovery/): BOTH present — recovery/felucca_known_good_b976764.fwsc, recovery/FM-1_V15.fwsc. OK.
- Port holders: no OTA/workbench/install process holding the MIDI port; only MIDIServer (system) and an emu http.server on :8000 (PID 72737, emulator, not OTA). No process holds the FM-1 MIDI port; the absence of ports is a cable/device issue, not contention.
- Backups: not re-verified here (projects/samples) — confirm before any write; write reprograms flash.

## Build under test (intended)

- felucca: d6781ed (SHEP slice wiring) — tip of felucca log.
- engines: 91b4350 (SHEP).
- Own build → needs --force on install.

## Ledger (to fill during session)

For each build flashed: identity + commit, then per test (one concern per line):
- boot
- pages touched
- quick sound from untouched engines
- specific feature probe (SHEP strands)

## Status

BLOCKED at pre-flight: FM-1 not detectable (exit 3). Do not write blind. Sort USB data cable, replug, rerun --info. Flash hand-off only after device is detected.

---
name: bench
description: The careful procedure for flashing to and testing on the FM-1 hardware (felucca-lab) — pre-flight, install, recovery packages, the session ledger, and the ledger hand-back. Use whenever the FM-1 is about to be flashed or hardware-tested.
---

Everything installs over USB-MIDI SysEx with `felucca/tools/fm1_install.py`; commands below run
from `felucca/`. The device is the source of truth: hardware results are never recorded from
memory or from the host side, only from what the session showed. (The FM-1 packaging and install
knowledge that exists nowhere upstream is distilled in the lab root's NOTES.md.)

## Pre-flight

- A USB **data** cable (a charge-only cable fails silently: no MIDI ports appear at all), nothing
  else holding the port (the web workbench's OTA, another terminal's install).
- `python3 tools/fm1_install.py --info` first. Exit 3 = "FM-1 not found" — stop there and sort the
  cable, do not write blind. With a package argument it prints the running identity beside the
  package's, so a mismatch shows **before** anything is sent. (If the port is found but the
  installer's pattern misses it, retry with `--port FM1` — the port name quirk is in NOTES.md.)
- The recovery floor before any write, both kept in the lab root's `recovery/`:
  `recovery/felucca_known_good_*.fwsc` (the last verified felucca tree) and the stock FM-1 V15
  (`recovery/FM-1_V15.fwsc`). Projects and samples are backed up first — the write reprograms
  flash.

## Install

- `python3 tools/fm1_install.py build/felucca.fwsc` for the current tree; a specific port with
  `--port NAME`; `--yes` to skip its prompt. **Own builds carry no loader marker and need
  `--force`.** `python3 tools/fm1_install.py recovery/FM-1_V15.fwsc` returns to the stock
  firmware and needs no `--force` (the installer's stock check allows it).
- **Never unplug during a write.** A cut-off install leaves the FM-1 in update mode: replug and
  run the same install again — it finishes the write, the device restarts, and the installer
  checks the installed identity (exit 6 = wrong model/identity after install: investigate, do not
  reinstall over it).

## During

- One build at a time, one concern per test: on the session ledger, each build's identity and
  commit, then each test's result under it (boot, the pages touched, a quick sound from the
  untouched engines, then the specific feature probe).
- A full bank save before removing power (chapter/sample saves are flash transactions too).
- The rollback is a normal install, not a disaster: if a build misbehaves, flash the
  `recovery/felucca_known_good_*.fwsc` package and carry on; the session notes what misbehaved
  either way.

## After

- Record the flashed build (name + commit) in the lab README's ledger and the session's findings
  in the slice's entry, in the next commit — the bench is not over until the dossier reflects it.
- Symptoms that survive a rollback go into the lab ledger as findings, with the receipts stated
  (the device's line, the machine's numbers); a suspicion without a receipt is noted as a
  suspicion.

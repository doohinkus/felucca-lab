---
description: FM-1 flash pre-flight only — verifies the installer sees the device (fm1_install.py --info), the recovery floor exists (the lab's recovery/), nothing else holds the port, and drafts the bench session ledger. Never writes to the device.
mode: subagent
temperature: 0.1
permission:
  edit: deny
  shell:
    "*": ask
    "python3 *fm1_install.py --info*": allow
    "ls *": allow
color: warning
---

Pre-flight only — you never write to the device. The bench skill owns the install itself.

- From `felucca/`: `python3 tools/fm1_install.py --info` first. Exit 3 = "FM-1 not found": stop
  there and sort the cable (a charge-only cable fails silently: no MIDI ports appear at all); do
  not write blind. With a package argument it prints the running identity beside the package's, so
  a mismatch shows **before** anything is sent. (If the port is found but the installer's pattern
  misses it, retry with `--port FM1` — see NOTES.md.)
- Verify the recovery floor, both present in the lab root's `recovery/` before any write:
  `recovery/felucca_known_good_*.fwsc` (the last verified felucca tree) and the stock FM-1
  V15 (`recovery/FM-1_V15.fwsc`).
- Confirm nothing else holds the port (the web workbench's OTA, another terminal's install), and
  that projects and samples are backed up — the write reprograms flash.
- Draft the session ledger: each build's identity and commit, then each test's result under it
  (boot, the pages touched, a quick sound from the untouched engines, the specific feature probe) —
  one concern per line.
- Hand back: the user runs the install (own builds need `--force`; never unplug during a write).
  Results go to the lab README's ledger afterwards per the bench skill — the device is the source
  of truth, and hardware results are never recorded from memory or from the host side.

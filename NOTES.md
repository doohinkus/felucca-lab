# NOTES — FM-1 packaging, install, and bench knowledge

Distilled from the m-vave-bloop bench sessions (2026-10-22). This is knowledge the upstream
Felucca tree does not carry; the procedures themselves live in `.opencode/skills/bench/SKILL.md`.
The device is the source of truth; a suspicion without a receipt stays a suspicion.

## The install path

- Installs go over **USB-MIDI SysEx** with `felucca/tools/fm1_install.py`, run from `felucca/`.
  Exit 3 = "FM-1 not found" (sort the cable first: a charge-only cable fails silently, no MIDI
  ports appear at all). Exit 4 = the package's flash.bin entry length is refused at step 1 —
  **nothing written** at either point. Exit 5 = the head compare (the official loader's own
  check) refused. Exit 6 = installed-identity check failed after the write — investigate, do
  not reinstall over it.
- **Own builds carry no loader marker and need `--force`.** The stock package
  (`recovery/FM-1_V15.fwsc`) needs no `--force` — the installer's stock check allows it.
- **Port-name quirk:** when the device is running Felucca, the USB port name ("FM1 Community")
  can miss the installer's PORT_RE — reach it with `--port FM1`.
- **Never unplug during a write.** A cut-off install leaves the FM-1 in update mode: replug, run
  the same install again — it finishes, the device restarts, and the identity check runs.

## The package shape (why the repack exists)

- FM-1_900's loader requires the flash.bin entry to be exactly `OTA_FL_LEN` **0x93000 =
  602,112 B**; a package outside that is refused in step 1 before anything is erased ("another
  board's package").
- The repack recipe (proven 2026-10-22, produced by a session that left no script — its
  reproducible rebuild is still wanted): take the app image, lay it through
  `fm1pkg_make.py flash_image(app, KEY=0x980F)` into the app area `[0x4000, 0x93000)` (SFC
  chip key 0x980F), then wrap with `ufw(padded_flash, ota, product)` under the loader the
  device keeps. `recovery/felucca_known_good_b976764.fwsc` carries Felucca's OWN ota.bin
  (6,934 B) — no head compare at all; the official V15 loader head-compares instead and passes
  only Felucca-packaged heads.
- Everything above 0x93000 (the store, the update records, the banks) is untouched by the
  loader (`LDR_REC_LO/HI`, `LDR_DATA_LO/HI`) — the user data outlives any install.
- Byte-verification before a bench: flash.bin exactly 0x93000, the app area SFC-decrypted
  byte-identical to the source package's, the loader entry's dcrc checked. `donor_repacked.fwsc`
  (610,090 B, identity FM-1_8000013) is the worked example on this machine — it is the
  m-vave donor's app through Felucca's packaging, **not** a felucca tree.

## Hardware receipts (2026-10-22)

- `--info` on FM-1_900 [running]: port Felucca; USR1..3 empty; the recovery floor present.
- A full take recorded on the donor base: boot, GRAB on a SAMPLE track, BT pairing ("FM1
  Community"), A2DP audio through, take saved + played back. The BT path thins the sound when
  the phone is the source (SBC/BT-side); acceptable for a grab, noted here so the next bench
  does not chase it as a regression.

## Recovery floor

Both packages sit in `recovery/` (md5-checked when copied here):
- `felucca_known_good_b976764.fwsc` — the last verified felucca tree (a build of the
  m-vave-bloop fork's felucca, most recent proven-booting felucca build on this device).
- `FM-1_V15.fwsc` — the stock firmware (byte-identical copies of `~/Downloads/FM-1.fwsc` and
  `~/.fm1-flash/FM-1_V15.fwsc`).

## The local toolchains (and one PATH trap)

- The machine has everything `./build.sh` and the host tests need: JieLi toolchain at
  `~/.jieli/toolchain`, AC79 SDK at `~/fw-AC79_AIoT_SDK`, Docker running, Node, and Emscripten
  at **`~/emsdk`** (`emcc` 6.0.12, verified 2026-10-23).
- **The trap:** `emcc` is only on the PATH of shells that read the emsdk activation's profile
  line. A non-login shell — a script, an agent tool call, CI — answers `command -v emcc` with
  nothing while Terminal answers with the binary. The old symptom was doubly quiet:
  `web/emu/build.sh` exited "emu: no emcc (Emscripten)" and `tests/run_tests.sh` printed a
  *skip* line — a session could "pass" its gates twice and never have built or replay-tested
  the emulator at all. This is exactly how the 2026-10-23 slice lost the emulator's receipts
  until someone asked why nothing was listening on 8790.
- Since the fix both scripts source `$EMSDK` → `~/emsdk` → `/opt/emsdk` themselves before
  concluding `emcc` is absent (BUILDING.md, Prerequisites), so the silent skip only happens
  with no emsdk on the machine at all. Interactive shells still want
  `source ~/emsdk/emsdk_env.sh` by hand.

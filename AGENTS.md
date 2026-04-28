# AGENTS.md

## What this repo is

ZMK keyboard firmware **configuration only** for a Corne split keyboard (nice_nano v1, nice_view display). No build tooling, no scripts, no tests — all compilation happens in CI via GitHub Actions.

## File map

| File | Purpose |
|------|---------|
| `config/corne.keymap` | Active keymap for the physical Corne (5-column, `five_column_transform`) |
| `config/corne.conf` | Kconfig settings (deep sleep, keyboard name; RGB/display commented out) |
| `config/west.yml` | West manifest pinned to `zmkfirmware/zmk@main` |
| `build.yaml` | CI build matrix: left + right halves, each with nice_view |
| `moergo.keymap` | **Auto-generated** by the GO_60 Layout Editor for a different board (GO_60 with Cirque trackpad). Do not hand-edit — regenerate via the editor. |

## Building / flashing

There is no local build. Firmware is built by pushing to GitHub, which triggers `.github/workflows/build.yml` → calls `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`. Download `.uf2` artifacts from the Actions run and drag onto the keyboard in bootloader mode.

## Keymap language

Files use **ZMK Device Tree syntax** (`.keymap` / `.conf`):
- `#include <behaviors.dtsi>` and `<dt-bindings/zmk/keys.h>` are standard
- `moergo.keymap` additionally uses `<dt-bindings/zmk/rgb.h>`, `<input/processors.dtsi>`, and pointing device includes
- Behavior nodes live inside `/ { behaviors { ... }; };` blocks; keymap layers inside `/ { keymap { compatible = "zmk,keymap"; ... }; };`
- Key positions in `combos` are zero-indexed integers matching the physical matrix; `moergo.keymap` defines named `POS_*` macros for readability

## Two keymaps — don't confuse them

- `config/corne.keymap` targets a **5-column Corne** (`five_column_transform`). Key positions 0–29 for the 3×5+3 layout.
- `moergo.keymap` targets the **GO_60** (6-column, 4-row + thumb cluster, 60 total positions). It has `cirque_rh_listener` / `cirque_lh_listener` for a Cirque trackpad and a Factory test layer. The `#pragma once` and helper macros at the top are intentional ZMK idioms from urob/zmk-helpers.

## Active layers in `config/corne.keymap`

| Index | Name | Activated by |
|-------|------|-------------|
| 0 | base | default |
| 1 | nav | `lt 1 BACKSPACE` (left thumb) |
| 2 | media | `lt 2 ESC` (left thumb) |
| 3 | num | `lt 3 SPACE` (right thumb) |
| 4 | sym | `lt 4 RET` (right thumb) |
| 5 | fun | `lt 5 '` (right thumb) |
| 6 | (empty) | unused |

Home-row mods: `hml` on left hand (LGUI/LALT/LCTRL/LSHFT on ASDF), `hmr` on right hand (LSHFT/RCTRL/RALT/RGUI on JKL;). Both use `balanced` flavor, 200 ms tapping term, `require-prior-idle-ms = <150>`.

## Combos in `config/corne.keymap`

| Combo | Positions | Output |
|-------|-----------|--------|
| caps_word | 20+28 | `&caps_word` (layer 0 only) |
| plus | 5+15 | `PLUS` |
| minus | 25+15 | `MINUS` |
| copy | 18+22 | `RG(C)` |
| paste | 18+23 | `RG(V)` |

## Editing the keymap via UI

Use https://nickcoutsos.github.io/keymap-editor/ to visually edit `config/corne.keymap`. It reads `config/corne.json` for the physical layout definition. The `moergo.keymap` is edited through the MoErgo GO_60 editor, not this tool.

## Reference

- ZMK docs: https://zmk.dev/docs
- Miryoku layout reference: https://github.com/manna-harbour/miryoku_zmk

# Protasena-library

The parts library for [Protasena](https://github.com/KaanAlper/Protasena), an open-source
electronics design suite. It is **KiCad's official symbol and footprint libraries, converted**
into Protasena's library pack format: every KiCad symbol becomes a device you can pick and
place, every KiCad footprint becomes a PCB package, and devices are linked to their packages
with pin-to-pad maps.

This repository holds data only. It is generated, not edited by hand: to change something,
change the converter or its mapping table in the Protasena repository and regenerate.

## Contents

| Path | What |
|---|---|
| `pack/index.json` | Pack index: one entry per library (format `protasena-library`, version 1) |
| `pack/symbols/<Library>.json` | Devices converted from one KiCad symbol library (`Library.kicad_sym`) |
| `pack/footprints/<Library>.pretty.json` | Packages converted from one KiCad footprint library (`Library.pretty`) |
| `report.json` | The converter's report for this pack: counts, renames, missing links, approximations |
| `LICENSE` | Creative Commons Attribution-ShareAlike 4.0 International, legal code |
| `KICAD-LICENSE.md` | The KiCad libraries' licence file, copied unchanged from the upstream repositories |

Device and package files hold one item per line, so a regenerated pack diffs line by line.

From KiCad libraries 9.0.9.1 (numbers from `report.json`): 223 symbol libraries with
22 583 symbols (12 127 of them derived with `extends`) became 22 583 devices (102 power
symbols, 27 973 units, 804 008 pins). 155 footprint libraries with 15 415 footprints became
15 415 packages (385 640 pads). 20 382 devices have at least one package (73 661 links in all).
There were no conversion failures, and the pack loads with Protasena's validating loader without
a single issue. About 300 MB uncompressed; the largest file is under 10 MB.

## Using it with Protasena

    git clone https://github.com/KaanAlper/Protasena-library
    export PROTASENA_LIBRARY_DIR=$PWD/Protasena-library/pack

Protasena reads the pack from `PROTASENA_LIBRARY_DIR`; without it Protasena only has its small
built-in set of generic parts.

## How it was generated

Upstream, pinned to the tag **9.0.9.1** of both repositories:

| Repository | Tag | Commit |
|---|---|---|
| <https://gitlab.com/kicad/libraries/kicad-symbols> | 9.0.9.1 | `ad36cd14bcd1b1cd0484f629ccdd3481366f74f3` |
| <https://gitlab.com/kicad/libraries/kicad-footprints> | 9.0.9.1 | `2b941bf1d97862be429793b46fd52a5add09a2fc` |

Converter: `protasena-libconv` from Protasena commit `4e5318a756aed85c78410d6a4ee407ac793ce170`
(branch `m1-libconv`), with its mapping table `apps/protasena-libconv/data/map.toml`:

    git clone --branch 9.0.9.1 --depth 1 https://gitlab.com/kicad/libraries/kicad-symbols
    git clone --branch 9.0.9.1 --depth 1 https://gitlab.com/kicad/libraries/kicad-footprints
    git clone https://github.com/KaanAlper/Protasena && git -C Protasena checkout 4e5318a756aed85c78410d6a4ee407ac793ce170
    cargo run --release --manifest-path Protasena/Cargo.toml -p protasena-libconv -- \
        --symbols kicad-symbols --footprints kicad-footprints \
        --map Protasena/apps/protasena-libconv/data/map.toml \
        --out pack --report report.json \
        --upstream "KiCad libraries 9.0.9.1 (kicad-symbols, kicad-footprints)"

The output is deterministic: the same inputs give byte-identical files.

## What the conversion changes

The libraries were adapted, not just copied. In short:

- **Format and units.** KiCad's S-expression files become JSON; millimetres become thou
  (1/1000 inch) with the y axis pointing up. Symbol coordinates keep their sign (KiCad symbols
  are y-up too); footprint coordinates are mirrored in y (KiCad footprints are y-down).
- **Devices.** Derived symbols are expanded with their parent's drawing. KiCad's shared units
  and body styles are copied into each unit / De Morgan variant. Each device gets a reference
  prefix, description, keywords, a Pick Devices category and sub-category, a manufacturer when
  the library name tells it, `VALUE` and `PACKAGE` properties, and a record of its KiCad origin
  (library, symbol, licence).
- **Names.** A few common parts get Proteus-style names (`Device:R` is `RES`, `Device:C` is
  `CAP`, `power:GND` is `GROUND`, …; listed in `report.json` under `renamed`). Device names are
  unique across the pack: when two libraries hold a symbol of the same name, the second one is
  named `NAME_LIBRARY` (also listed under `renamed`).
- **Packages.** A device links the footprint in its `Footprint` field (its default package),
  plus up to 64 footprints that match its footprint filters and have exactly its pin numbers as
  pads. Footprint names are used as package names (they are unique across KiCad's libraries).
- **Approximations** (counted in `report.json` under `notes`): a pin's graphic style (inverted,
  clock, …) is drawn as body graphics; alternate pin functions are not carried over; pins or
  pads that share a number are kept as `N`, `N#2`, `N#3`, … (all on pad `N`); trapezoid pads
  become rectangles; custom pads become their anchor pad plus the outline drawn on copper;
  slotted drills keep their smaller size; zones, 3D model references and a few unusual tokens
  (`report.json` under `kicad_unknown`) are left out.

## Licence and attribution

The data in this repository is derived from the **KiCad libraries**, © the KiCad Library Team
and the KiCad libraries contributors, and is licensed, like them, under the
**Creative Commons Attribution-ShareAlike 4.0 International** licence (CC-BY-SA 4.0, legal code
in `LICENSE`) **with the KiCad libraries' design exception** (KiCad's own statement, copied
unchanged, in `KICAD-LICENSE.md`; see also <https://www.kicad.org/libraries/license/>).

What the design exception means, in short (KiCad's licence file is the authoritative text):
using these parts in your own electronic designs does not make your designs, or the files you
generate from them (netlists, Gerbers, BOMs, …), subject to CC-BY-SA 4.0, and you do not need to
credit the KiCad libraries inside your designs. The exception does not cover redistributing the
library itself: if you share this pack or parts of it as a collection, modified or not, you must
do so under CC-BY-SA 4.0 and keep this attribution and the licence files with it.

This pack is an adaptation of the KiCad libraries made by the Protasena contributors with
`protasena-libconv`; the changes are described above. It is not an official KiCad product, and
neither the KiCad project nor the KiCad Library Team endorse it. Protasena itself (the
application and the converter) is separate software under MIT OR Apache-2.0; that licence does
not apply to the data here.

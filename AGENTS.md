<!-- Keep this one-view brief at every project stage. Fill it from verified
     repository facts as the design develops; omit sections that do not yet
     apply instead of adding plans or placeholders.

     Keep the section order identical in every OpenDrone repo, so a reader and an
     agent find the same thing in the same place anywhere. Delete a section that
     does not apply rather than leaving it empty. Target 150 lines: if a section
     grows past a screen, the detail belongs in the schematic, not here. State
     current fact only. No plans, no TODOs, no history outside Revisions.

     Shared rules live in "Board rules" at the end of this file. Every board
     repository carries a byte-identical copy of that section, so it works
     standalone; its .github/workflows/agents-rules.yml checks the copy against
     this file. Edit the section here, then copy it into the boards with
     agents_section_sync.py --section "Board rules" from the hardware-tooling
     repository. A board's own sections hold only what differs for that board
     and do not repeat the copy.

     KiCad boards only: Board rules assumes KiCad. A mechanical or
     documentation-only repository (frame, motor, no .kicad_pro) does not carry
     it: it starts from the mechanical-repository template in the
     hardware-tooling repository, and the sync tool lists it as excluded and
     flags a stray Board rules section there. -->

# <Board>

<What the board is, in three sentences at most. Topology and the load-bearing
ICs. What it is not, if a reader would otherwise assume it.>

## Architecture

<The signal and power chain, block by block, in prose. Roughly ten lines. Say
why, not just what: the parts of the design a reader could not infer from the
schematic. Sub-sheet names in backticks so a reader can open the right one.>

## Power

```
<ASCII tree: source, each regulator with its part and output, and what each
rail feeds. One block, no prose.>
```

## Key parts

| Function | Ref | Part | LCSC | Note |
|---|---|---|---|---|
| <MCU> | U1 | | | |
| | | | | |

## Connectors and I/O

| Connector | Ref | Part | Function |
|---|---|---|---|
| | | | |

<Pinout table or pin map, only where the pinout is not visible from the
schematic sheet name.>

## Layout rules

<Only constraints a future editor would break by accident: keep-outs, RF
clearances, thermal copper, differential pairs, antenna keepouts, current paths
that must stay short. Delete the section if the board has none.>

## Firmware

<Which firmware, which target, how it gets on the board the first time. Link
upstream. Do not restate upstream documentation.>

## Repo

| | |
|---|---|
| Maintainer | <GitHub team or handle> |
| KiCad project | `hardware/<name>.kicad_pro` |
| Root schematic | `hardware/<name>.kicad_sch` <plus sub-sheets, listed> |
| Board | `hardware/<name>.kicad_pcb`, <N> layers, <thickness> |
| Design rules | `hardware/<name>.kicad_dru`, canonical block plus <board-specific rules, or none> |

<!-- Add a row only where the board differs from "Board rules", Files.
     Mechanical repos: replace the KiCad rows with the CAD tool. -->

## Rules

The [Board rules](#board-rules) at the end of this file apply, a CI-checked copy of the [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules) section. Only what differs for this board is listed here.

## Revisions

| Rev | Date | Change |
|---|---|---|
| <rev1> | <YYYY-MM-DD> | First release. |

## Board rules

Shared by every OpenDrone KiCad board repository, byte for byte, and checked
in CI against [hardware-template](https://github.com/OpenDrone-hw/hardware-template/blob/main/AGENTS.md#board-rules).
Edit them there, never in a board.

### Files

Every board has this layout; its Repo table lists only what differs.

| | |
|---|---|
| Designed in | KiCad 10 |
| Status | The `status-*` topic on the repository, never written in a file |
| Local library | `hardware/lib.kicad_sym`, `hardware/lib.pretty/`, `hardware/lib.3dshapes/`, nickname `lib` |
| Shared library | `hardware/KiCad-Library/`, submodule of [OpenDrone-hw/KiCad-Library](https://github.com/OpenDrone-hw/KiCad-Library), nickname `OpenDrone`; 3D models and exact component datasheets resolve through the project text variable `OPENDRONE_LIB` |
| Fab config | `hardware/fabrication-toolkit-options.json` |
| Board setup | 6 layers, 0.09 mm clearance and track, via 0.35 on 0.20 drill |
| License | CERN-OHL-S-2.0 |

A repository holds design files only. Sourcing (suppliers, prices, quotes, RFQs, contacts) is handled by Incutec and never lives in OpenDrone repositories.

### Parts and datasheets

- **Per-repository part index:** the root schematic and its listed sub-sheets
  are authoritative for what this board fits. Export the netlist with the
  command below when a script-readable board index is needed; do not maintain
  a second hand-written BOM.
- **Proven shared parts:**
  `hardware/KiCad-Library/PARTS-USED.md` is the catalogue index. Its `Boards`
  column identifies every repository using each LCSC part; filter that column
  for the repository name to get its proven shared-part view.
- **Exact datasheets:**
  `hardware/KiCad-Library/datasheet/manifest.json` maps shared symbols to the
  committed PDFs and their SHA-256 hashes. The PDFs live in
  `hardware/KiCad-Library/datasheet/`, and symbol links resolve there through
  `OPENDRONE_LIB`.
- **Local-only parts:** inspect `hardware/lib.kicad_sym`,
  `hardware/lib.pretty/`, and `hardware/lib.3dshapes/`, then verify supplier
  fields in the board schematic. Do not duplicate a shared part or datasheet.

### Commands

`<name>` is the KiCad project named in the board's Repo table.

```sh
# schematic and board checks
kicad-cli sch erc hardware/<name>.kicad_sch
kicad-cli pcb drc --schematic-parity --refill-zones hardware/<name>.kicad_pcb

# netlist, for scripted analysis
kicad-cli sch export netlist --format kicadsexpr -o /tmp/<name>.net hardware/<name>.kicad_sch
```

On macOS `kicad-cli` is at
`/Applications/KiCad/KiCad.app/Contents/MacOS/kicad-cli`, and `pcbnew` imports
only under KiCad's bundled Python, `KPY`:
`/Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3`.
Reusable scripts for renders, STEP export and packaging art come from Incutec
hardware tooling. The OpenDrone release standard is
[RELEASES.md](https://github.com/OpenDrone-hw/.github/blob/main/RELEASES.md).
Board-specific scripts, where a board has any, live in `hardware/tools/`.

### Rules

- **Never text-edit** `.kicad_sch`, `.kicad_pcb` or `.kicad_dru`. Use KiCad, or
  kicad-skip / the pcbnew API for scripted changes. `.kicad_pro` is JSON and may
  be edited directly for metadata.
  kicad-skip cannot round-trip KiCad 10 schematics: use a full S-expression
  parser, `kicad-cli sch upgrade --force`, then an ERC and netlist diff.
- **Metadata yes, connections no.** An agent may write BOM and documentation
  fields (MPN, Manufacturer, LCSC, Cost, Datasheet, text variables). An agent
  may not change nets, wiring, routing, placement, footprint assignment, or any
  value that changes the circuit.
- **Close KiCad before any write to a KiCad file.** KiCad caches library tables
  at process start and overwrites files on save.
- **Reuse before you draw.** Check the `OpenDrone` library and its
  `PARTS-USED.md` first. If the part is there we have already sourced,
  footprinted and shipped it, and its symbol links to the exact committed
  datasheet: place it from `OpenDrone`. Draw a new part into `lib` only when
  the catalogue has nothing that fits, imported with
  `easyeda2kicad` from its LCSC number. Pulling a newer catalogue is a
  deliberate, reviewed commit: `git submodule update --remote
  hardware/KiCad-Library`, then DRC.
- **One person holds a board layout at a time.** KiCad files do not merge. Say
  on Discord that you are taking it. See [CONTRIBUTING.md](CONTRIBUTING.md).
- **Run ERC and DRC before every pull request.** Existing approved findings
  may remain; a new type or increased count must be reviewed before merge.
  The commands are under Commands.

### By task

- Check the design: run the ERC and DRC commands in Commands before every pull request.
- Add a part: place it from the `OpenDrone` library if `hardware/KiCad-Library/PARTS-USED.md` lists it; otherwise import it into `lib` with `$KPY <hardware-tooling>/hardware/kicad/import_part.py` (read `--help` first), KiCad closed.
- Render the board for the README: `$KPY <hardware-tooling>/hardware/kicad/render_board.py hardware/<name>.kicad_pcb --outdir images`, KiCad closed.
- Analyse the netlist: export it with the netlist command in Commands, then read it with a script; never hand-write a second BOM.
- Update the shared library: `git submodule update --remote hardware/KiCad-Library`, run DRC, commit as its own reviewed change.

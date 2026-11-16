# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

The OASIS MQTT TC repository. The active work is the **MQTT for Sensor Networks (MQTT-SN) v2.0 specification**,
under `mqtt-sn-2.0/prose/`. It is a specification document, not an implementation (the TC charter excludes
reference implementations). `contributions/` holds historical snapshots and member contributions; leave them alone.

## Build commands

All targets live in `mqtt-sn-2.0/prose/edit/makefile`; run them from `mqtt-sn-2.0/prose/edit/`.
`source activate.sh` activates the local `.venv` (nide, PyYAML, etc.).

| Command                  | Purpose                                                                                   |
|:-------------------------|:------------------------------------------------------------------------------------------|
| `make`                   | Quick loop: render HTML + PDF (no cleaning, no gates); opens the PDF (`OPEN=true` to suppress) |
| `make derive`            | Regenerate everything derived from `records/` (prose marks, Appendix B, tables, `.edn`, views, kaitai) |
| `make check`             | Fail if any generated file differs from its records (runs first in every build)           |
| `make quality`           | `nide quality` spec + OASIS rules; reads rendered output, so render first                 |
| `make diff`              | Prose diff of rendered HTML against HEAD                                                  |
| `make release`           | From-scratch build with every gate — what CI runs (`make release OPEN=true`)              |
| `make packet-dictionary` | Publish the machine-readable packet dictionary JSON + schema to `../share/`               |
| `make kaitai-test`       | Compile `../share/kaitai/*.ksy` and parse every packet vector (`KSC=` path to compiler)   |
| `make lint` / `make types` / `make format` | ruff + black (line length 120) / mypy on `bin/` Python scripts            |

Toolchain: `nide`, `pandoc`, `typst`, `b3sum`, PyYAML; `bytefield-svg` + `svgexport` (npm) for packet figures.
CI (`.github/workflows/mqttsn20_*.yml`) pins exact versions and runs `make release` then `nide quality`.

## Architecture: records → derived prose → rendered channels

1. **Prose source**: `edit/src/*.md`, concatenated in the order listed in `edit/etc/bind.txt` (a new section file
   must be added there). Headings carry anchors like `## Session state{#session-state}`.
2. **Records** (`edit/records/*.yaml`) are the source of truth for structured normative content. `make derive`
   (`bin/spec_records.py`) generates from them — **never hand-edit the generated parts**:
   - `records/statements/<src-file>.yaml` → the `«MQTT-SN-x.y-n»` normative-statement marks in `src/<src-file>.md`
     and every row of Appendix B (`src/mand-norm-stmts-00.md`). To add a statement: add a record and write
     `«MQTT-SN-x.y-n»` at its place in the prose; derive expands it.
   - `records/packets/<packet>.yaml` → `src/images/packet/<packet>-packet-diagram.edn`, packet dictionary rows,
     `views/packet-vectors.json`, `../share/kaitai/*.ksy`. (Exceptions, still hand-written:
     `structure-packet-diagram.edn`, `mqtt-basic-bit-diagram.edn`.) PNG/SVG are rendered from `.edn` per
     `src/images/packet/README.md`.
   - `records/tables/*.yaml` → tables in `src/ctrl-pack-format-0[1-4]-*.md` and `src/ops-behavior-14-client-states.md`.
   - `records/states/server-view.yaml`, `records/roles.yaml` → `views/states/`, `views/roles/`.
   Field reference: `edit/records/README.md`. Editing map: "Authored records and generated files" in `edit/README.md`.
3. **Views** (`edit/views/`) are generated reviewer aids; never edit. `views/findings.md` reports contradictions
   between figures, tables and statements — check it after every `make derive`.
4. **Rendering** via `nide` (config `edit/etc/nide.yaml`, typst templates `etc/mqtt.typ`, `etc/oasis.typ`) produces
   the deliverables in `prose/share/`: GFM+ `.md`, self-contained `.html`, `.pdf` (+ `.typ`, checksums, manifest,
   `.ir.json`). These are committed build outputs — regenerate them via make, don't edit them.

## Workflow conventions

- A record change and its regenerated files belong in the same commit; `make check` (and CI) fails otherwise.
- Authoring loop: edit records/src → `make derive` → review `views/findings.md` and diff → `make` →
  `make quality` → `make diff`; run `make release` before a PR.
- Normative language uses uppercase keywords (MUST, SHOULD, OPTIONALLY, …); quality rules live in
  `edit/etc/rules/spec.rules.yaml` (extends `oasis.rules.yaml`).
- `edit/bin/` scripts are project tooling — don't modify without review.

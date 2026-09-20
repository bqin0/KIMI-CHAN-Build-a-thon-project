# CLAUDE.md

Project instructions for Claude Code. Read this before doing anything in this repo.

## Project

KIMI-CHAN is a **mechatronic** entry for the Build-a-thon environment challenge.
The final deliverable is a **working physical prototype plus a pitch** — not a
production system.

**Subcategory: TBD.** The team has not yet committed to one of the three. Until
this line says otherwise, keep the architecture neutral: put anything
category-specific behind `shared/` interfaces so a pivot costs hours, not days.

| Category | Our solution must do at least one of |
|---|---|
| Monitoring | beat an existing monitor on speed/cost/accuracy/reliability; reach unreachable sites; capture new data points; turn data into sharper insight |
| Repurpose | reuse commonly discarded items as a core component; improve recycling systems; reduce parts discarded; recover unrecovered materials; extend usable life |
| Restoration | mechanise replanting, seeding, weeding, or habitat rebuilding that is currently done by hand |

Design constraints from the brief that outrank convenience:
- **Off-grid.** Assume no mains power and no reliable WiFi/cellular. Power budget
  and store-and-forward are first-class design concerns, not afterthoughts.
- **Unattended for months.** No code path may require a human to power-cycle it.
  Watchdogs, safe restarts, and bounded memory use are requirements.
- **Cheap.** Cost per unit is part of the pitch. Flag when a design choice adds
  meaningful BOM cost.

## Hard constraint: there is no hardware in this session

Claude runs in a cloud container. **No board, no sensors, no serial port, no
flashing, no oscilloscope.** This shapes everything:

- **Never claim firmware "works."** Say "compiles" or "passes host tests." A
  claim that something runs on-device is only valid after a human confirms it.
- **Every hardware dependency sits behind an interface with a fake.** Sensor
  reads, radio sends, motor drives, GPIO — all go through a thin abstraction so
  the logic above them is testable on the host.
- **Keep logic pure.** Parsing, state machines, scheduling, calibration maths,
  and packet encoding must be host-testable functions with no hardware calls.
- **Simulated data must be labelled `SIMULATED`** in code, logs, and any UI.
  Fabricated readings must never be able to reach a demo unlabelled — that is a
  credibility failure in front of judges.

## Repo layout

```
firmware/   ESP32 / Arduino, C++, PlatformIO — the low-power field node
gateway/    Raspberry Pi, Python — base station, ingest, processing, UI
shared/     Protocol contract shared by both (packet schema, units, IDs)
tools/      Dev scripts: simulators, log replay, bench utilities
hardware/   Wiring diagrams, BOM, power budget, datasheets
docs/       Design decisions and pitch material
```

`shared/` is the contract between the two codebases. **Change it on both sides in
the same commit, or not at all.** A schema change that lands in one half and not
the other is the single most likely way this project breaks on demo day.

New top-level directories need a reason — ask first.

## Commands

<!-- TBD: fill these in once the toolchains are actually installed. -->

```bash
# Firmware (PlatformIO)
pio run                      # build
pio test -e native           # host-side unit tests — these must pass
pio run -t upload            # flash — HUMAN ONLY, never claim to have run this

# Gateway (Python)
uv sync                      # install deps
pytest                       # tests
ruff check . && ruff format .   # lint + format
```

Before any commit: **run the host tests and the linter for every directory you
touched.** Never commit red. If a tool is not installed in this container, say so
explicitly rather than skipping silently.

## Conventions

- Formatter config wins. Do not hand-format; run the formatter.
- **Units in names.** `temp_c`, `pressure_pa`, `interval_ms`, `batt_mv`. An
  unlabelled number in a sensor path is a bug waiting to happen.
- Every logged reading carries a timestamp, a sensor ID, and its units.
- No dynamic allocation in the firmware hot path. Fixed buffers, bounded queues.
- Fail soft in the field: a dead sensor degrades the node, it never halts it.
- Match surrounding code. Do not refactor unrelated files inside a feature change.

## Testing (balanced — not everything, but not nothing)

Tests are required for: packet encode/decode, sensor value parsing and
calibration, state machines, power/duty-cycle scheduling, and anything with
arithmetic in it. Tests are not required for: thin hardware wrappers, one-off
bench scripts in `tools/`, or demo UI glue.

## Ask before

- Adding a dependency, changing the `shared/` protocol, altering the power
  architecture, picking the subcategory, rewriting a file wholesale, or touching CI.
- Any design decision where two readings of the requirement lead to materially
  different hardware. Otherwise: state your assumption and keep building.

## Git

- Work on feature branches. **Never push to `main`, never force-push.**
- Commit messages: imperative subject under 72 chars, body explaining *why*.
- Never commit secrets, API keys, or `.env`. Use `.env.example`.
- Do not open a PR unless asked.

## Working style

- Report failures honestly. If tests fail, show the output — do not summarise a
  broken build as "mostly working."
- Distinguish clearly between *written*, *compiles*, *host-tested*, and
  *verified on hardware*. Only a human can deliver the last one.
- Don't create documentation files unless asked.
- For multi-file changes, outline the plan before writing code.

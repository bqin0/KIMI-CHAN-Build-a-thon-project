# SKILLS.md

Repeatable workflows for this project. Say the trigger phrase and Claude runs the
whole sequence the same way every time.

> **How this file is wired.** Claude Code auto-loads `CLAUDE.md`, but *not* this
> file — `CLAUDE.md` points here so it gets read. Once a workflow below has been
> run a few times and settled, promote it to a real on-demand skill at
> `.claude/skills/<name>/SKILL.md` with YAML frontmatter (`name`, `description`)
> so it loads automatically when the task matches. Until then, these are
> invoked by name.

---

## `new-node-feature`
**Trigger:** "add a feature to the firmware" / "new sensor"

1. Confirm which side owns it — `firmware/`, `gateway/`, or both via `shared/`.
2. If it touches the wire format, update `shared/` **first**, then both sides in
   the same commit.
3. Define the hardware interface and its fake before writing any driver code.
4. Write the pure logic (parse, calibrate, decide) and its host tests.
5. Write the thin hardware wrapper last — it stays dumb and untested.
6. Run `pre-commit`.
7. Report what is host-tested vs. what still needs a human on the bench.

## `sim-run`
**Trigger:** "run it" / "does this work?"

There is no hardware here, so "running it" means running against fakes.

1. Launch the gateway against the simulator in `tools/`, not a live radio.
2. Feed it a recorded or synthetic packet stream — every synthetic value tagged
   `SIMULATED`.
3. Show the actual output: packets decoded, values, errors, timings.
4. State plainly: "this is simulated; on-hardware behaviour is unverified."

## `power-budget`
**Trigger:** "check the power budget"

Off-grid for months is the brief's hardest constraint, so this gets its own pass.

1. Table every component: active current, sleep current, duty cycle.
2. Compute average draw, then days of runtime on the chosen battery and the
   solar input if any.
3. Flag the largest consumer and the cheapest way to cut it.
4. If a recent change worsened the budget, say so before it ships.

## `pre-commit`
**Trigger:** "commit this"

1. `pio test -e native` (if firmware changed) and `pytest` (if gateway changed).
2. `ruff check . && ruff format .` on Python; formatter on C++.
3. Re-read the diff adversarially — leftover debug prints, hardcoded pins,
   unlabelled units, TODOs that are actually blockers.
4. Only then commit, with a *why* in the body. Never commit red.

## `demo-check`
**Trigger:** "demo check" — run before every practice pitch and on demo day

1. Walk the exact path a judge will watch, step by step, and list what must be
   true at each step.
2. Verify nothing on that path depends on network access.
3. Verify no `SIMULATED` data appears in a way that could be mistaken for real.
4. List the failure modes and the recovery for each — what you do on stage if the
   node doesn't join, the sensor reads garbage, or the battery is flat.

## `brief-check`
**Trigger:** "does this still answer the brief?"

1. Restate our subcategory and which of its bullet requirements we claim.
2. For each claim, name the concrete evidence — a measurement, a cost figure, a
   reached location, an insight the incumbent can't produce.
3. Call out any claim we cannot yet back with evidence. That gap is the pitch's
   weakest point and should drive what gets built next.

## `fix-ci`
**Trigger:** "CI is red"

1. Reproduce the exact failure locally before changing anything.
2. Fix the root cause. **Never skip, disable, or delete a test to get green.**
3. Show the same check passing locally, then push once.

## `review`
**Trigger:** "review this"

Check, in order: hardware calls outside their interface; unlabelled units;
unbounded allocation or growth in firmware; a failure path that halts instead of
degrading; `shared/` changed on only one side; secrets or calibration constants
committed; simulated data that isn't labelled.

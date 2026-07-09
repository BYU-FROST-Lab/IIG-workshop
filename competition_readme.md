# Indoor Exploration Competition

Multi-robot (and single-robot) indoor exploration and relaying under
communication constraints. Robots explore an unknown environment using only
local sensing and limited peer-to-peer communication, and must get what
they've learned back to a fixed base station. **What you write is the
policy** — exploration strategy, and optionally relay strategy too.
Everything else (sensing, motion, communication, the environment) is
provided and identical for every submission.

## Tracks

Submissions are scored on two tracks, differing only in `num_robots`:

- **Single-agent** — `num_robots: 1`
- **Multi-agent** — `num_robots: 2, 3, 4, 5`

You write one `Policy`; it's evaluated across both.

## Scoring

At the end of a run, `main.py` prints the final **base-station coverage**:
the fraction of the true map the base station has learned about by
`max_steps`. This rewards actually getting information home, not just
observing it — so both exploration and relay strategy matter for score.


### Your goal doesn't have to be frontier-based

Any exploration strategy is allowed (frontier-based, sampling-based,
potential-field, learned, etc.) as long as it only uses `obs`. Returning
`None` from `decide()` is a legitimate way to say "no opinion" - the
framework falls back to the nearest reachable frontier in that case. But if
you return a specific goal, it has to actually be reachable: an
unreachable, occupied, out-of-bounds, or malformed goal raises
`InvalidGoalError` and halts the run immediately, rather than being
silently patched up for you.

Map prediction is also not provided. You may implement any map prediction stradegy but it is not required.


## What you implement

You write a `Policy` class with up to three decisions, all made per-robot:

- **`decide()`** (required) — where to explore next.
- **`should_relay()`** (optional) — when to switch from exploring to relaying info back to base.
- **`decide_relay_handoff()`** (optional) — while relaying, whether to hand relay duty off to a nearer connected robot, and to whom.

If you don't override `should_relay()`/`decide_relay_handoff()`, you get the
provided baseline behavior for free, matching the shipped config.

Override `should_relay()` and/or `decide_relay_handoff()` to compete on
relay strategy as well as exploration.

### What the `Observation` contains

Only what your robot has legitimately sensed or received over
communication — never ground truth:

- `obs.combined_obs_map` — your robot's map so far (unknown / free / occupied). Whenever two robots (or a robot and the base station) are in comm range, their maps are automatically fused, so this can include cells a peer observed, not just what your own sensor has seen. Fusion is provided/automatic — identical for every submission — not something `decide()` controls.
- `obs.pose` — your robot's current position
- `obs.unreported_mask`, `obs.delegated_mask` — bookkeeping on what's been observed but not yet reported to base
- `obs.pose_lists_of_others`, `obs.intents_of_others` — other robots' trajectories/intents, as last shared over comm (may be stale if out of range)
- `obs.base_pose` — the base station's position (always known)
- `obs.connected_robot_ids` — robot ids currently (this timestep) in comm range, so entries for them in `pose_lists_of_others` are guaranteed fresh, not stale — useful for `decide_relay_handoff()`


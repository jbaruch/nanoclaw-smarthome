# nanoclaw-smarthome

Smart-home capability for [NanoClaw](https://nanoclaw.dev) agents — reading and eventually
operating a household's devices from the assistant, rather than from bespoke automations.

> **Status: planning.** Nothing is built here yet. The phase issues moved over from
> `jbaruch/nanoclaw` (where they lived as `epic:smart-home`) and carry the backlog; this README
> records the decisions that shape it.

## Why it is its own repo

Hubitat ingest started inside NanoClaw's platform core, which is the wrong home for it —
personal smart-home infrastructure in a codebase whose whole argument is being small enough to
read. `qwibitai/nanoclaw#848` covers extracting the listener as a host plugin; this repo is
where the capability itself lives and grows.

## The decision that shapes everything else: read is not command

An agent behind a chat front door can reach whatever its tools can reach, and the same hub that
reports soil moisture also holds the door locks and the alarm mode. **Reading state and
actuating a device are different capabilities and want different gates** — not one tool surface
with careful wording, because prompt-level caution is not a security boundary.

The same line already exists on the device side of this house and was drawn for the same
reason: one Hubitat app reports on irrigation and a separate one opens the valves, because a
bug in the first makes noise and a bug in the second floods a yard.

Practical consequence: **the first slices are read-only**, and the reusable seam is "read device
state from the hub" rather than anything domain-shaped.

## What a smart-home agent has to get right

These are not hypotheticals. Each one is an incident from running this house, and each is a
thing a capable model will get confidently wrong without being told:

- **A cloud integration will lie about physical reality.** A zone was commanded, the cloud
  returned `success: true`, the device reported the valve open and then closed, and no water
  ran — the controller had been off the network for nine hours. Every layer relayed the
  fiction. `success` means *dispatched*, never *executed*.
- **A change-filtered attribute is not a liveness signal.** The platform suppresses unchanged
  values, so a healthy sensor reporting a steady reading looks identical to a dead one — and a
  transient *wrong* value becomes permanent, because nothing will ever change it back.
- **A device can report the wrong physical fact truthfully.** A Z-Wave lock derives `lock` from
  the lock's *mode*, discarding bolt status, so a stalled bolt reads "locked".
- **Never state a number nobody measured.** Thresholds, bands, "should be above X" — if the
  measurement has not been made, saying it is worse than saying nothing, because someone acts
  on it.

## Relationship to `jbaruch/hubitat-dev`

`hubitat-dev` is a **source, not a dependency**. It is a developer toolkit — it teaches an agent
to author, lint, test and deploy Groovy against a hub. An agent answering a household question
never writes Groovy.

What transfers is the operate half: the HTTP surface (`_reference/endpoints.md`), device
commanding, mesh diagnosis, and the epistemics above. What does not: `sandbox-constraints`,
`groovy-gotchas`, `app-lifecycle`, `driver-lifecycle`, `scaffold`, `lint-review`, `deploy`,
`test`.

## Backlog

The phase issues carry the plan. Phase 1 (event capture and storage) is done and moved over
closed for its history.

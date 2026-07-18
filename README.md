# cloud-itonami-isco-8151

Open Occupation Blueprint for **ISCO-08 8151**: Fibre Preparing, Spinning
and Winding Machine Operators.

This repository designs a forkable OSS business for a fibre preparing,
spinning and winding mill scheduling and logistics coordination practice:
a mill scheduling and supply-coordination robot manages crew/task records
under a governor-gated actor, so a fibre-preparing, spinning and winding
crew keeps its own operating records instead of renting a closed
workforce-management SaaS.

**Maturity: `:implemented`.** `src/spinningcoord/` implements the
`SpinningCoordActor` as a `langgraph.graph/state-graph`
(`spinningcoord.actor`) wired to a `Fibre Preparing, Spinning and Winding
Mill Scheduling Coordination Advisor` (`spinningcoord.advisor`) and an
independent `SpinningCoordGovernor` (`spinningcoord.governor`), following
the itonami actor pattern (ADR-2607121000): `:intake -> :advise -> :govern
-> :decide -+-> :commit (:ok? true) +-> :request-approval (:escalate? true,
human-in-the-loop interrupt) +-> :hold (:hard? true)`. HARD invariants
(always hold, never overridable): spinner provenance, mill provenance,
no-actuation (`:effect` must be `:propose`), a closed op-allowlist
(`:log-work-record`, `:schedule-crew-operation`, `:flag-safety-concern`,
`:coordinate-supply-order` — nothing else may ever be proposed), and a
permanent, unconditional block on any proposal that would directly finalize
a machine-operation-execution decision (e.g. deciding to proceed with a
specific spinning or winding run) or a mill-safety-clearance decision (e.g.
declaring the mill or a spinning line safety cleared), or that would
override a mill safety officer's judgment. Always-escalate paths (human
sign-off regardless of confidence, mapping this repo's Trust Controls in
[`docs/business-model.md`](docs/business-model.md)): `:flag-safety-concern`
(always) and `:coordinate-supply-order` above the registered cost
threshold.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a mill scheduling/logistics coordination
robot performs crew scheduling, production-run/inventory/progress-record
logging and raw-fibre/yarn-stock supply-order coordination for a fibre
preparing, spinning and winding crew, under an actor that proposes actions
and an independent **Fibre Preparing, Spinning and Winding Mill Scheduling
Coordination Governor** that gates them. The governor never dispatches
hardware itself, never operates spinning or winding equipment on the mill
floor, and never finalizes a machine-operation-execution decision or a
mill-safety-clearance decision, and never overrides a mill safety officer's
judgment; `:high`/`:safety-critical` actions (such as a flagged
entanglement-hazard/fibre-dust-exposure/equipment-condition concern, or an
above-threshold supply order) require human sign-off. **This actor
coordinates MILL SCHEDULING/LOGISTICS ONLY — it never operates spinning or
winding equipment itself, and it never makes a mill-safety-clearance
decision itself.**

Fibre Preparing, Spinning and Winding Machine Operators run textile-mill
carding, spinning and winding equipment with significant entanglement
hazard (rotating spindles, drive belts, high-speed winding mechanisms),
alongside fibre-dust exposure. This is a real entanglement-hazard and
fibre-dust-exposure domain; this actor never operates that equipment and
never clears it as safe — it only schedules and logs around it, and always
routes entanglement-hazard/fibre-dust-exposure/equipment-condition concerns
to a human mill safety officer.

## Core Contract

```text
crew roster + mill registration + safety-reporting policy
        |
        v
Fibre Preparing, Spinning and Winding Mill Scheduling Coordination
Advisor -> SpinningCoordGovernor -> log/schedule/coordinate, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses,
finalize a machine-operation-execution decision, finalize a
mill-safety-clearance decision, override a mill safety officer's judgment,
suppress an operating record, or disclose sensitive data without governor
approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `8151`). Required capabilities:

- :robotics
- :identity
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.

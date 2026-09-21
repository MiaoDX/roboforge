# Roadmap and Open Validation Work

[Back to README](../README.md) · Baseline: 2026-09-20

The roadmap advances by validation gates, not by number of integrations. Everything below is planned work; the initial repository establishes only the documentation baseline and does not claim GPU, SIL/HIL, or physical-robot validation.

## M0 · Freeze contracts and candidate dependencies

Deliver experiment/action/trace schemas, task and robot-package boundaries, a buildable Isaac Lab / Arena candidate dependency set, and a concrete G2 hardware configuration inventory.

Exit criteria: every dependency has provenance/versioning; unknowns are explicit; candidate schemas can represent both G2 and a single-arm robot, lockstep and compute HIL, without hiding new concepts inside an ambiguous backend field.

Do not turn version numbers from research discussions into compatibility claims or fabricate runtime results for candidate configurations.

## M1 · Lockstep loop and second embodiment

Integrate G2 with one tabletop pick-and-place task: policy reset/infer, action conversion, environment reset, trace, and offline rescoring. Then implement the same task family with simulated Franka.

Exit criteria: adding the second robot does not change the core loop; varying inference wait while holding the logical action schedule fixed yields results within task tolerance; stale actions cannot cross reset epochs.

Initial task progression: tabletop rigid body → drawer/cabinet articulation → room-scale mobile manipulation. The last stage is not a prerequisite for the first release.

## M2 · Deployment-timing SIL and compute HIL

Reuse the M1 tasks/scenes while adding asynchronous policy execution, action buffers, expiry and fallback rules. Connect the actual target compute device and deployable software to the simulated plant.

Exit criteria: latency, jitter, dropped frames, reordering, disconnects and simulation lag can be injected and observed; end-to-end latency and clock error are recorded; invalid timing cannot be hidden by slowing the world and calling the run real-time.

This milestone does not claim motor/drive HIL. Deeper hardware-in-the-loop work requires separate risk assessment and validation.

## M3 · Trusted evaluation and real-world calibration

Create paired real/sim tasks across several policy versions. Freeze regression/holdout sets and evidence definitions. Produce Capability, Deployment and Validity reports.

Exit criteria: meaningful improvements and regressions can be distinguished; false passes and uncertainty are quantified; the validity envelope is explicit. If this fails, limit the platform to exploratory use rather than using it as an automatic release gate.

## M4 · Data production and value validation

Integrate seed demonstrations, one augmentation tool, data QA, lineage and LeRobot export. Choose one concrete failure mode and compare targeted augmentation against generic simulation augmentation and a real-data baseline.

Exit criteria: acceptance rate, human effort and GPU cost are recorded; fixed-training comparisons show interpretable benefit on independent tests / physical robot. If there is no benefit, investigate data and calibration before scaling generation.

## M5 · Scale, automation and research

Only after earlier gates hold: add OSMO multi-node workflows, more robots/tasks, generator plugins and agent-assisted QA. Evaluate Newton / multi-backend validation only for a concrete physics need.

A 6–12 month horizon is a planning window, not a delivery promise. Prioritize trusted evaluation and timing continuity first; scale data production only when measured ROI supports it.

## Initial validation matrix

Numbers below are proposed engineering starting points. They are neither completed results nor sufficient statistical proof and should be adjusted to task variance and risk.

| Validation | Initial gate |
| --- | --- |
| Multi-robot | G2 + Franka; second robot does not modify core loop |
| Reset isolation | 1,000 resets with no stale action crossing episode boundaries |
| Run ledger | Every scheduled episode, failure, retry and missing record is traceable |
| Infrastructure | At least 1,000 scheduled episodes; initial valid-run target ≥99%, with confidence interval and failure classes |
| Scoring | Same trace + scorer version recomputes identically |
| Timing | Fault injection follows profile; latency budgets come from measured control requirements |
| Real-world comparison | At least three meaningfully different policy versions; paired analysis with uncertainty |
| Data value | Fixed training budget comparing baseline / generic augmentation / targeted augmentation |

Exploratory comparisons may start with 3 tasks × 2 policies × 100 initial conditions, but repeated seeds from one source scene must not be treated as independent coverage.

## Must be determined empirically

| Item | Default treatment | Decisive evidence |
| --- | --- | --- |
| G2 model / tools / cameras / firmware | Declare per instance; no generic default | Physical inventory, SDK and reference-demo comparison |
| Dependency combination | Officially matched stack is the candidate | Build, render, reset and task smoke tests |
| Control / sensor rates | Profile configuration, never one hard-coded rate | Sustained rate, latency, jitter and drop measurements |
| GPU / inference placement | Separation optional; formal runs pin resources | End-to-end valid throughput and contention tests |
| Fallback / safety action | Robot-specific and reviewed | Vendor constraints, controlled tests and site safety assessment |
| Contact parameters / tolerances | Source values are initial priors; mark uncalibrated | Repeatable physical response and paired tasks |
| Storage / scheduling | Reuse existing services first | Actual scale, resource governance and collaboration needs |

## First implementation tasks

- [ ] Establish contracts and a mock worker; validate experiment lifecycle and reset epochs.
- [ ] Integrate G2 simulation assets/control mapping and record source licenses.
- [ ] Implement a lockstep task, minimal policy adapter and rescorable trace.
- [ ] Run the same task family with a Franka package and verify no embodiment leakage into Core.
- [ ] Add wall-paced execution and timing fault injection.
- [ ] Connect target compute and collect compute-HIL evidence.
- [ ] Complete paired physical-robot evaluation before targeted data-ROI experiments.

Implementation and benchmark results should be committed separately; checking these boxes in documentation is not evidence that a capability is complete.

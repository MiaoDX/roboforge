# Trusted Evaluation and Data Production

[Back to README](../README.md) · Status: experiment protocol and validation design

## 1. Three reports, not one blended score

| Report | Question |
| --- | --- |
| Capability Report | Under lockstep timing and a fixed observation contract, what can the policy accomplish? |
| Deployment Report | With inference, communication, control and resource constraints included, how does the deployed system behave? |
| Validity Report | For which robots, objects, controllers and contact regimes is simulation supported by real-world evidence? |

High capability with poor deployment performance is useful information and must not be averaged away. Reports also include software, asset, controller and scorer versions; resources; data split; timing validity; and failure taxonomy.

## 2. Tasks, criteria and evidence

`TaskDefinition` specifies goals, preconditions, stages, forbidden events and time budget. `TaskBinding` binds scene, objects, robot, initialization and reset. `EvidenceProvider` supplies scoring evidence.

Simulation ground truth, physical sensor estimates and human review are different evidence classes. Shared task semantics do not make their scores automatically comparable. Official scoring distinguishes success, failure, insufficient evidence and not evaluated, and records stage-level outcomes rather than only final binary success.

For “open a drawer and retrieve an object,” record handle grasp, target opening, successful extraction, placement, forbidden collision and timeout separately. Agents may not alter official thresholds or environment physics to make a benchmark pass.

## 3. Denominators and failures

Every scheduled episode enters the run ledger. Invalid initialization, infrastructure error, policy-interface error, valid task failure and success are counted separately. Publish valid-run rate and success among valid task executions together with end-to-end completion, timeouts and safety events. Do not improve a score by deleting difficult samples.

Retry rules are fixed in advance. Preserve the first failure and every retry. A later successful retry does not replace the original outcome.

Model comparisons should use paired initial conditions and scene versions where practical, with uncertainty on differences. Many seeds from one scene are not independent scenes; account for correlation by scene/object family rather than manufacturing confidence with near-duplicate episodes.

## 4. Sim-real calibration

Build a small but representative paired real/sim suite covering the actual robot, cameras, controller and common contacts. Compare several policy versions with meaningfully different capability levels rather than two success rates from one model.

Calibration covers camera/appearance, scale/geometry, joints and actuator response, action processing, contact parameters, control timing and sensor timing. Prefer measured parameters. Record ranges and uncertainty for parameters that cannot be identified reliably.

Key outcomes are policy ordering and direction of change, uncertainty in differences, false simulation passes that would trigger a bad release, and known failure conditions. A single correlation coefficient is insufficient; if policies are too close to distinguish, do not claim reliable ranking.

The deliverable is a **validity envelope**: the robot/control/object/task/timing regimes with supporting evidence and the regimes not yet covered. Uncalibrated tasks are exploratory and must not be the sole release gate. [SIMPLER](https://simpler-env.github.io/) is a methodological reference, not evidence that G2 is already calibrated.

## 5. Three data products

| Product | Production methods | Validation focus |
| --- | --- | --- |
| Perception data | Rendering, labels, scene/appearance randomization | Sensor-label consistency, coverage, downstream real-world value |
| Policy trajectories | Teleoperation, Mimic / SkillGen, planners, expert policies | Executability, stages/success, timing, training value |
| Evaluation / failure data | Closed-loop rollout, contact and stage events | Reproducibility, attribution, model-comparison value |

Initial trajectory flow: a small set of high-quality demonstrations → stage/object-relation annotation → community-tool augmentation → execution validation → data QA → training export. Preserve native tool formats and do not assume arbitrary scenes automatically yield valid expert trajectories.

Re-rendering after an appearance change does not revalidate dynamics. Changes to geometry, physics, control or actions require re-execution. Failure samples are retained for analysis or explicit failure-learning workflows and are not silently mixed into successful demonstrations.

## 6. Layered data and minimum metadata

| Layer | Default format | Preserve |
| --- | --- | --- |
| Raw trace | MCAP / native files + external object references | Observations, action-processing chain, state, timing, events, errors |
| Tool working data | Native Mimic / Genie formats | Information required by the generating tool |
| Training export | LeRobot Dataset v3; others as needed | Consumable episodes, video, state, actions, labels |
| Evaluation result | JSON / Parquet | Scorer input/version, stages, outcome, validity, statistics |

Every manifest should include run/episode IDs, code/dependencies, asset hashes, robot model/instance, calibration, controller, policy weights and preprocessing, source demonstration/generation method, initial state and seed, clock/topology, schema/scorer version, QA result, license and split.

Video must be traceable to the frames actually seen by the policy and their preprocessing. Store intended model actions separately from commands actually applied. A state snapshot without controller/policy state must not be advertised as an exact resumable checkpoint.

## 7. Holdouts and lineage

Development sets support failure analysis and targeted generation. Fixed regression sets track known capabilities. Independent holdouts test generalization and do not expose concrete samples or answers to generators.

Group source scenes, objects, real capture sessions and their digital cousins/derived trajectories by provenance family rather than randomizing only by filename. Freeze splits in immutable manifests. Official test data does not automatically flow back into training.

## 8. Data ROI experiments

Under a fixed training budget and recipe, compare a real-data baseline, real + generic simulation augmentation, and real + targeted augmentation. Report both independent-test and physical-robot effects while controlling extra compute and recipe changes.

Core production metrics are accepted trajectories / valid evaluations per GPU-hour, cost per accepted sample, human repair effort, coverage, and downstream policy improvement—not total generated hours or an FPS number that excludes rendering, inference or storage.

If simulation cannot reliably reflect important real-world differences, or augmentation has no benefit in controlled experiments, fix calibration and task definition before scaling scene count.

## 9. QA ladder

```text
static schema / asset checks
  → environment initialization and rest stability
  → articulation / contact / reachability / task validation
  → reset stress and timing-fault tests
  → data integrity and score recomputation
  → paired real-world validation and training ROI
```

A privileged oracle/planner can provide evidence that a task is solvable; failure by the current weak policy does not prove the environment is invalid. Collision approximations for dynamic objects must preserve task-relevant cavities and opening space rather than accidentally filling cups or drawers.

## Format and tool references

[MCAP](https://mcap.dev/), [LeRobot](https://github.com/huggingface/lerobot), [robomimic](https://github.com/ARISE-Initiative/robomimic), [RLDS](https://github.com/google-research/rlds), [Isaac Lab](https://github.com/isaac-sim/IsaacLab). These are reuse candidates, not a list of integrations already completed by this repository.

# Execution Model: Stop-the-World to SIL/HIL

[Back to README](../README.md) · Status: target protocol; implementation and fault-injection validation pending

## 1. Two independent questions

**The clock determines when the world advances; deployment topology determines which software or hardware participates in the loop.** RoboForge should not encode `sim / sil / hil / real` as four mutually exclusive, ambiguous backends.

| Profile | Clock | Plant | Policy / control deployment | Primary use |
| --- | --- | --- | --- | --- |
| Lockstep simulation | `lockstep` | Simulated | Local / remote components that support logical time | Capability, debugging, data production |
| Deployment-timing SIL | `wall_paced` | Simulated | Deployable software stack | Inference, communication and control timing |
| Compute HIL | `wall_paced` | Simulated | Actual target compute device | Joint validation of deployment hardware and software |
| Physical robot | `wall_paced` | Physical | Approved hardware and control stack | Calibration and final validation |

A software-only stack may also support lockstep SIL if it fully supports virtual time. External SDKs and watchdogs are not assumed to do so. Physical plants do not accept lockstep. Hybrid sensor configurations must explicitly declare calibration, time mapping and observation provenance.

## 2. Lockstep synchronization

```text
complete agreed physics steps → freeze observation → policy inference / wait
                              → validate action → execute action segment → next barrier
```

While waiting, pause the logical world participating in the experiment: physics, relevant sensor sampling, controller logical state, and simulation time. An external component that continues integrating or timing out on wall time must be adapted or rejected from a lockstep path.

v0 uses one barrier per vectorized worker. A worker groups compatible robot topology, control frequency and task structure. Different configurations use different workers rather than per-environment asynchronous pausing.

A policy may return a single action or an action chunk. The experiment pins chunk length, execution semantics and observation points. Changing inference wall time while keeping the logical action schedule fixed should not change the physics result; exceeding an explicit run budget follows the timeout protocol rather than waiting forever.

Lockstep removes inference delay where the experiment explicitly allows it. It does not reproduce deployment timing and does not imply bitwise determinism across GPUs or software versions.

## 3. Wall-paced execution

Observation capture, asynchronous inference, and scheduled action execution are decoupled. Cadence follows a monotonic clock, not adjustable calendar time. Physics step size and target control / sensor rates are declared in the execution profile.

| Event | Default semantics |
| --- | --- |
| Observation backlog | Bounded queue; retain history or latest-only according to policy contract; never allow unbounded growth |
| New action chunk | Align by source observation, time index and validity interval; do not blindly restart at chunk head |
| Stale / duplicate action | Reject or deduplicate and record; never execute in the wrong episode |
| Action buffer exhausted | Use a robot-specific reviewed fallback; do not repeat the last command indefinitely |
| Inference disconnect / crash | Degrade or terminate according to protocol and preserve the original failure; retries must not hide it |
| Simulation behind wall time | Record lag and timing validity; terminate or invalidate after budget is exceeded rather than slowing the world and claiming real-time validity |
| Deadline miss | Record location, rate and consequence separately from task failure |

Asynchronous inference is not the same as Real-Time Chunking. RTC is an optional policy-specific mechanism. Enabling it creates a distinct deployment configuration that must be evaluated separately rather than treated as a transparent platform fix.

GPU simulation, Python and ordinary networking do not provide hard real-time guarantees. Low-level servo control and safety interlocks remain the responsibility of vendor or validated control systems.

## 4. Timestamps and action provenance

Minimum trace fields include:

| Category | Fields |
| --- | --- |
| Identity | `run_id`, `worker_id`, `episode_id`, `reset_epoch` |
| Correlation | `observation_id`, `action_id`, chunk / step index, policy request ID |
| Source | clock domain, sensor / controller, source sequence |
| Observation | capture, ingest and policy-delivery timestamps |
| Inference | request, start, finish and response-received timestamps |
| Execution | scheduled execution, validity interval, controller receipt and actual application time |
| Environment | simulation time, physics-step index, wall-time lag, synchronization-error estimate |

Timestamps from different devices cannot be subtracted safely without a recorded clock mapping and synchronization error. Prefer same-device intervals where possible; end-to-end latency reports must state their synchronization basis.

Store raw model output, de-normalization, coordinate transforms, IK / WBC, clipping and controller-received values separately so failures can be attributed to the model, action adapter, or controller.

## 5. Lifecycle and reset isolation

```text
prepare → reset / initialize → validate initial state → run
        → terminate → flush records → commit episode
```

Increment `reset_epoch` on every initialization. Every policy request and action carries the epoch; late responses from an old epoch must never enter a new episode. Reset includes physics state, policy history, controller integrators, random state, outstanding requests and action buffers.

A physical reset is a checked procedure and may require a human to restore objects. It must not be disguised as instantaneous state assignment. Initialization failure is an environment/setup outcome, not automatically a policy failure.

v0 guarantees that stored traces can be rescored. Dynamics replay under identical actions must be validated within task-specific tolerance. Exact continuation from arbitrary snapshots and exact cross-engine replay are not first-version guarantees.

## 6. Failure taxonomy

An episode stores execution state, task result and timing validity separately instead of collapsing everything into one `success` flag.

| Dimension | Examples |
| --- | --- |
| Execution | completed, infrastructure error, invalid initialization, interface error, human abort |
| Task | success, failure, insufficient evidence, not evaluated |
| Timing | valid, deadline budget exceeded, simulation cadence invalid, insufficient clock synchronization |
| Safety | command limiting, protective stop, controlled abort; disclosed alongside task outcome |

Never report only the final successful retry. First failure, retry reason, retry count and final outcome all belong to the official record.

## 7. Required tests

For lockstep, inject inference waits while holding the logical action schedule fixed and verify physics results within tolerance. For wall-paced execution, inject latency, jitter, dropped frames, duplicate/out-of-order messages and network loss, then verify expiry and fallback behavior. During reset, inject late actions to verify epoch isolation.

HIL tests should also measure sustained throughput under device load and thermal throttling, plus resource contention when simulation and inference share GPUs. Reports must state resource allocation so hardware contention is not misattributed to policy quality.

## References

[ROS 2 clock and time design](https://design.ros2.org/articles/clock_and_time.html), [ros2_control](https://control.ros.org/jazzy/index.html), [OSMO](https://github.com/NVIDIA/OSMO), [LeRobot](https://github.com/huggingface/lerobot). These are implementation references; RoboForge defines the protocol choices above and does not claim that any one dependency provides all of them.

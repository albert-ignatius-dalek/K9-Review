# Software Architecture Research

Per the roadmap's Priority 4: architectural understanding, not
implementation commitment. Nothing here is a decision — it's the
engineering background a real decision would draw from, so that choice
happens with context instead of by default.

## ROS2 suitability

ROS2 (current stable distributions: Jazzy Jalisco, Kilted Kaiju) is the
de facto standard for exactly this class of problem — a legged platform
needing navigation, perception, and behavior coordination together. Its
real advantages for Mark 6: DDS-based networking (fits a Jetson-central,
multi-microcontroller topology cleanly), managed lifecycle nodes, and a
mature ecosystem (Nav2, MoveIt, robot_localization) that would otherwise
need building from scratch. The previous K9 software project used none
of this (confirmed by direct code audit — zero `rospy`/`ros2` references
anywhere) — its Flask-microservice-per-manager pattern is a real,
reusable *architectural idea* (health-checked services, mock-mode
fallback, a conductor with restart cooldowns) but not a ROS2 pattern.
Adopting ROS2 for Mark 6 would mean re-expressing that same idea as
ROS2 nodes/lifecycle management rather than porting Flask services
directly.

**Trade-off worth naming**: ROS2 brings real overhead (learning curve,
build system, message-passing latency) that a simpler custom framework
wouldn't have. The previous project's fast-local-reflex /
slow-deliberate-reasoning split (a microcontroller for real-time servo
control, a more capable board for reasoning) is a sound idea independent
of ROS2 — it maps naturally onto ROS2's node model, but doesn't require
it either.

## Behavior Trees

BehaviorTree.CPP is the most widely used deliberation library in the
ROS2 ecosystem, with ready ROS2 wrappers, and Nav2 itself is built on it
— since ROS2's Navigation stack switched to behavior trees as its main
customization mechanism, this isn't a competing choice against Nav2, it
composes with it directly.

**Direct fit for the K9 Behaviour Engine concept** (`docs/Systems_Architecture.md`):
a behavior tree is a natural implementation for the KBE's reusable
behaviour states (Investigate, Follow, Guard, Wait, Greet, etc.) — each
becomes a subtree, composed and reused across contexts, with the tree's
own tick-based execution model giving a clean way to interrupt a
lower-priority behavior for a higher-priority one (matching the
project's own owner-safety → mission → knowledge → efficiency →
etiquette decision hierarchy). This is architectural fit, not a
recommendation to start building — no behavior exists yet to encode.

## Navigation stack

**Nav2** (ROS2's navigation stack) is the standard reference architecture:
`slam_toolbox` or `nav2_amcl` for localization (both publish the
map→odom transform a robot needs to localize), a costmap-based planner/
controller pair, and behavior trees for task-level coordination. Mark
6's own navigation software is confirmed to be zero prior asset (the
K9 software migration report found "entirely aspirational... no mapping,
following, obstacle avoidance, or docking code exists anywhere" in the
previous project) — there's nothing to adapt, only a real architecture
to build toward once the Navigation System's hardware (camera, RPLIDAR
C1 if committed) is actually selected.

## SLAM

`slam_toolbox` is the standard ROS2 SLAM package pairing with Nav2. For
a legged platform specifically, SLAM's usual wheel-odometry input isn't
available — Mark 6 would need either leg odometry (computed from joint
angles + contact state, inherently noisier than wheel encoders) or
visual/LIDAR-inertial odometry instead. This is exactly why Balance and
Navigation are correctly modeled as separate-but-dependent systems in
`docs/Systems_Architecture.md` — the same proprioceptive data (IMU,
joint state, foot contact) that Balance needs for stability is also
Navigation's odometry source on a legged robot, unlike a wheeled one
where these are unrelated concerns.

## Locomotion framework / Whole-body control

Real, current open-source references worth knowing about, not
recommending yet: **Pinocchio** (rigid-body dynamics library — computes
the kinematics/dynamics math a locomotion controller needs), **CasADi**
(automatic differentiation + optimization, used to formulate the control
problem), and solvers like **acados** or **Fatrop** built on top. These
combine into whole-body Model Predictive Control (MPC) frameworks — the
current state of the art for quadruped locomotion — which directly
optimize joint torques through full-order inverse dynamics, unifying
motion and force planning in one predictive layer, rather than treating
gait generation and balance as separate hand-tuned stages. Example
open frameworks in this space: DWMPC (distributed whole-body MPC for
quadrupeds), BiConMP.

**Why this matters for Mark 6 specifically**: the project's own
Locomotion/Balance split (`docs/Systems_Architecture.md`) mirrors
exactly what whole-body MPC treats as one problem (motion + force +
balance, solved together) rather than two. This doesn't mean Mark 6
needs full MPC on day one — a simpler IK-plus-heuristic-balance
controller is a reasonable first step, matching the project's own
"prove the methodology before adding detail" discipline — but it means
the two systems should be designed with eventual MPC-style coupling in
mind (shared state estimation, compatible interfaces) rather than as
fully independent modules that would need re-architecting later.

## Sensor fusion / state estimation

The standard approach for legged robots: fuse IMU + joint encoder
measurements (an extended Kalman filter is the common baseline; more
recent work uses multiple leg-mounted IMUs to correct a major error
source in single-IMU proprioceptive odometry, and detects foot contact
from leg-IMU signal analysis rather than dedicated force sensors — a
cheaper alternative to Mark 6's currently-RESERVED foot contact sensing
line item). ROS2's `robot_localization` package provides a ready EKF
implementation for exactly this kind of multi-sensor fusion (IMU, joint
state, and — if adopted — GPS/visual odometry) rather than requiring a
custom filter from scratch.

**Concrete implication for Mark 6's hardware**: the servos are currently
position-commanded with no confirmed encoder feedback path
(`docs/Systems_Architecture.md`, Balance System). Real state estimation
of the kind described above needs joint position feedback, not just
commanded position — worth flagging as a real requirement to check
against the STS3215/SCS0009 servo bus protocol's actual capabilities,
not an assumption that commanded angle is good enough.

## Balance architecture

Treated as its own system deliberately (not folded into Locomotion),
consistent with how the legged-robotics literature treats it — dynamic
stability and whole-body balance control are typically designed as a
distinct control layer, informed by the same state estimate Navigation
uses, feeding correction back into the same joint commands Locomotion
issues. See "Locomotion framework" above for how MPC unifies this in
current research; the practical starting point for Mark 6 is simpler:
real IMU-based orientation feedback plus centre-of-mass estimation from
`docs/BOM.md`'s (still largely ESTIMATED) mass distribution, before any
MPC-level sophistication.

## What this does NOT decide

No commitment to ROS2, Nav2, any specific MPC framework, or a timeline.
This is background for whenever Locomotion/Balance/Navigation actually
start (roadmap Priority 1 is Mechanical Pelvis completion; these systems
come later, per the outside-in working order). Revisit this document
when that work actually starts, rather than treating it as a spec
written too early.

# SnapKit Python — Tolerance-Compressed Attention Allocation

**Everything within tolerance is compressed away. Only the deltas survive.**

SnapKit is a Python library implementing **Snaps as Attention** theory — a mathematical framework for allocating finite cognitive resources using tolerance-compressed snap functions over ADE-classified lattices.

## The Core Idea

A **snap function** maps continuous values to their nearest expected point. Values **within tolerance** snap silently to the baseline. Only values **exceeding tolerance** (deltas) demand attention — and attention is a finite resource.

```
    Value 0.05 ──→ SnapFunction ──→ ✓ Snapped (within 0.1 tolerance)
    Value 0.30 ──→ SnapFunction ──→ ⚠ DELTA (exceeds tolerance)
    │                                       │
    └── Compressed, ignored              └──→ Attention allocated
```

This mirrors how expertise works: familiar patterns snap automatically, freeing cognition for what's actually novel or significant.

## Installation

```bash
pip install snapkit
```

From source:

```bash
git clone https://github.com/SuperInstance/snapkit-python
cd snapkit-python
pip install -e .
```

**Requirements:** Python ≥ 3.8, NumPy ≥ 1.20

Optional extras:
```bash
pip install snapkit[visualization]  # matplotlib
pip install snapkit[integration]    # sympy
pip install snapkit[all]            # everything
```

## Quick Start

```python
from snapkit import SnapFunction, SnapTopologyType

# Create a snap function with hexagonal (A₂) topology
snap = SnapFunction(tolerance=0.1, topology=SnapTopologyType.HEXAGONAL)

# Within tolerance — compressed away
result = snap.snap(0.05)
assert result.within_tolerance  # True

# Exceeds tolerance — demands attention
result = snap.snap(0.3)
assert not result.within_tolerance  # DELTA detected
print(f"⚠ delta={result.delta:.4f} exceeds {snap.tolerance} tolerance")
```

### Full Attention Pipeline

```python
from snapkit import SnapFunction, SnapTopologyType
from snapkit import DeltaDetector, AttentionBudget

# 1. Configure snap
snap = SnapFunction(tolerance=0.1, topology=SnapTopologyType.HEXAGONAL)

# 2. Multi-stream delta detection
detector = DeltaDetector()
detector.add_stream('market_data', SnapFunction(tolerance=0.15))

# 3. Finite attention budget
budget = AttentionBudget(total_budget=100.0, strategy='actionability')

# 4. Process data
deltas = detector.observe({'market_data': 0.27})
top_deltas = detector.prioritize(top_k=3)
allocations = budget.allocate(top_deltas)

for alloc in allocations:
    print(f"{alloc.delta.stream_id}: {alloc.allocated:.1f} attention units ({alloc.reason})")
```

### Learning Cycle

```python
from snapkit import LearningCycle

cycle = LearningCycle()

for value in experience_stream:
    state = cycle.experience(value)
    # Phases: DELTA_FLOOD → SCRIPT_BURST → SMOOTH_RUNNING → DISRUPTION → REBUILDING
    print(f"Phase: {state.phase.value}, Load: {state.cognitive_load:.2f}")
```

### Eisenstein Lattice Snap

```python
# Complex-valued snap to A₂ Eisenstein lattice
snap = SnapFunction(tolerance=0.5)
result = snap.snap_complex(complex(1.2, 0.7))
print(f"Snapped: {result.snapped:.4f}, delta: {result.delta:.4f}")
```

### Auto-Calibration

```python
snap = SnapFunction()
sample_data = [0.1, 0.12, 0.08, 0.11, 0.09, 0.5]  # mostly near 0.1, one outlier
snap.calibrate(sample_data, target_snap_rate=0.9)
print(f"Tolerance auto-set to {snap.tolerance:.4f}")
```

## Module Reference

### Core Modules

| Module | Description |
|--------|-------------|
| `snapkit.snap` | **SnapFunction** — tolerance gatekeeper with adaptive/hierarchical snap |
| `snapkit.delta` | **DeltaDetector** — multi-stream delta monitoring with severity ranking |
| `snapkit.attention` | **AttentionBudget** — finite cognition allocation with multiple strategies |
| `snapkit.scripts` | **ScriptLibrary** — pattern matching, automation, composition, versioning |
| `snapkit.learning` | **LearningCycle** — expertise lifecycle (experience → pattern → script → automation) |
| `snapkit.topology` | **SnapTopology** — ADE-classified lattice shapes (A₁ through E₈) |
| `snapkit.cohomology` | **ConstraintSheaf** — sheaf-theoretic consistency checking (H¹ computation) |

### Advanced Modules

| Module | Description |
|--------|-------------|
| `snapkit.adversarial` | Adversarial snap calibration — real vs fake delta detection |
| `snapkit.crossdomain` | Cross-domain feel transfer between snap topologies |
| `snapkit.streaming` | Real-time stream processing with rolling windows |
| `snapkit.pipeline` | Composable processing pipelines |
| `snapkit.visualization` | Terminal + HTML visualization of snap states |
| `snapkit.integration` | External library bindings (PySheaf, SymPy, NumPy) |
| `snapkit.serial` | Serialization & persistence |
| `snapkit.cli` | Command-line interface |

## API Reference

### `SnapFunction`

```python
SnapFunction(
    tolerance=0.1,
    topology=SnapTopologyType.HEXAGONAL,
    baseline=0.0,
    adaptation_rate=0.01,
)
```

| Method | Description |
|--------|-------------|
| `snap(value, expected=None)` | Snap a single value → `SnapResult` |
| `observe(value)` | Alias for `snap()` |
| `snap_vector(values, expected=None)` | Snap a numpy vector |
| `snap_complex(value, tolerance=None)` | Snap to Eisenstein A₂ lattice |
| `snap_nd(values)` | Element-wise snap on N-dimensional array |
| `snap_matrix(matrix, axis=1)` | Snap a 2D matrix along an axis |
| `snap_batch(values)` | Batch snap without changing state |
| `snap_rolling(values, window, stride)` | Rolling window snap for time series |
| `snap_hierarchical(value, levels)` | Multi-tolerance snap profile |
| `hierarchical_profile(value)` | Dict showing snap/delta at each tolerance level |
| `calibrate(values, target_snap_rate=0.9)` | Auto-set tolerance from sample data |
| `enable_adaptive_tolerance(window=50)` | Auto-adjust tolerance based on delta rate |
| `to_dict()` / `from_dict(data)` | Serialize/deserialize state |
| `reset(baseline=None)` | Clear history |

**Properties:** `snap_rate`, `delta_rate`, `calibration`, `statistics`

### `SnapResult`

| Field | Type | Description |
|-------|------|-------------|
| `original` | float | Input value |
| `snapped` | float | Lattice point it was snapped to |
| `delta` | float | Distance from expected |
| `within_tolerance` | bool | Whether it snapped or flagged as delta |
| `tolerance` | float | The tolerance used |
| `topology` | SnapTopologyType | Lattice shape |
| `is_delta` | bool (property) | `not within_tolerance` |

### `DeltaDetector`

```python
detector = DeltaDetector()
detector.add_stream('cards', snap, actionability_fn=..., urgency_fn=...)
deltas = detector.observe({'cards': 0.3, 'behavior': 0.1})
top3 = detector.prioritize(top_k=3)
clusters = detector.delta_clusters(n_clusters=3)
scores = detector.importance_scores()
```

### `AttentionBudget`

```python
budget = AttentionBudget(total_budget=100.0, strategy='actionability')
# Strategies: 'actionability' (weighted), 'reactive' (largest first), 'uniform'
allocations = budget.allocate(deltas)
insight = budget.attention_insight()  # fixation, overload, underload detection
```

### `ScriptLibrary`

```python
library = ScriptLibrary(match_threshold=0.85)
library.learn(trigger_pattern=np.array([...]), response='fold', name='basic_strategy')
match = library.find_best_match(observation)
library.compose(['script_a', 'script_b'])  # chain scripts
library.extend('parent_id', 'child_name', new_response)  # inherit
```

### `LearningCycle`

```python
cycle = LearningCycle(
    snap=SnapFunction(tolerance=0.1),
    novelty_threshold=5,
    script_creation_threshold=3,
)
state = cycle.experience(observation)
# state.phase: DELTA_FLOOD | SCRIPT_BURST | SMOOTH_RUNNING | DISRUPTION | REBUILDING
# state.cognitive_load: 0.0 (automated) to 1.0 (full attention)
```

### `SnapTopology` (ADE Classification)

```python
from snapkit.topology import hexagonal_topology, binary_topology, recommend_topology

topo = hexagonal_topology()  # A₂ — densest 2D packing
snapped, delta = topo.snap(point)
topo.quality_score  # Lattice quality metric
recommend_topology(num_categories=4)  # → tetrahedral
```

Available topologies: `binary_topology()` (A₁), `hexagonal_topology()` (A₂), `tetrahedral_topology()` (A₃), `triality_topology()` (D₄), `exceptional_e6()`, `exceptional_e7()`, `exceptional_e8()`.

### `ConstraintSheaf` (Cohomology)

```python
from snapkit.cohomology import ConstraintSheaf
sheaf = ConstraintSheaf(tolerance=0.1)
sheaf.add_constraint('temp', 98.6)
sheaf.add_constraint('bp_sys', 120.0)
sheaf.add_dependency('bp_sys', 'bp_dia')
report = sheaf.check_consistency()
report.globally_consistent  # True ↔ H¹ = 0 (Eisenstein PID guarantee)
```

## Topologies

| Topology | Root System | Roots | Coxeter h | Best For |
|----------|-------------|-------|-----------|----------|
| Binary (A₁) | A₁ | 2 | 2 | Yes/no, coin flips |
| Hexagonal (A₂) | A₂ | 6 | 3 | 2D continuous, Eisenstein lattice |
| Tetrahedral (A₃) | A₃ | 12 | 4 | 4-category decisions |
| 5-chain (A₄) | A₄ | 20 | 5 | Ordered sequences |
| Triality (D₄) | D₄ | 24 | 6 | Forked dependencies |
| E₆ | E₆ | 72 | 12 | Binary tetrahedral symmetry |
| E₇ | E₇ | 126 | 18 | Binary octahedral symmetry |
| E₈ | E₈ | 240 | 30 | Maximum finite symmetry |

## Learning Cycle Phases

```
1. 🌊 DELTA_FLOOD    — No scripts, everything is novel (load ≈ 1.0)
2. 💥 SCRIPT_BURST   — Patterns emerging, rapid script creation
3. 🏃 SMOOTH_RUNNING — Most things snap to scripts (load ≈ 0.0)
4. 🚨 DISRUPTION     — Accumulated deltas, scripts failing
5. 🔨 REBUILDING     — Constructing new scripts from deltas
```

## Performance

- SnapFunction: O(1) per observation, O(n) for batch
- DeltaDetector: O(k·log k) for prioritization (k streams)
- ScriptLibrary: O(n) pattern matching (cosine similarity)
- ConstraintSheaf: O(V + E) consistency check

## Connection to Constraint Theory

SnapKit Python implements the snap-attention framework from the Cocapn constraint theory project:

- **Eisenstein lattice (A₂)** provides the optimal 2D snap topology via hexagonal closest-packing
- **ADE classification** maps snap topologies to root systems with known algebraic properties
- **Cohomology module** uses the PID property of ℤ[ω] to guarantee H¹ = 0 (local → global consistency)
- **Learning cycle** models the build/run/monitor/rebuild pattern of expertise

## License

MIT — use freely, give credit.

---

*Built for the Cocapn fleet. From poker tells to planetary-scale attention allocation.*

*"The snap doesn't tell you what's true. The snap tells you what you can safely ignore so you can think about what matters."*

# Expected Logical Efficiency of Fabricated qLDPC Chips

*A defect-aware figure of merit for quantum LDPC codes on multilayer superconducting hardware*

Cornell QCA · 27 September 2026

---

## Contents

1. The problem and the metric
2. ACID
3. HAL
4. PropHunt
5. Phase 1 method: ACID-informed HAL on BB codes
6. Phase 2: beyond BB
7. PropHunt constrained by ACID
8. Gauge basis selection
9. Phase 3: chips supporting multiple codes
10. Reading
11. Schedule for implementation

---

## 1 · The problem and the metric

The reference figure for "which qLDPC code should we build" plots logical efficiency *kd²/n* against hardware complexity *C_hw* across ~150 explicit layouts (Mathews et al., npj Quantum Information 12, 114 (2026)). Codes to the lower right are cheap to build and information-dense; codes to the upper left are neither.

> **Figure 1:** Logical efficiency *kd²/n* against hardware complexity *C_hw* for ~150 qLDPC layouts, grouped by code family and labelled by stabiliser weight *w*. Reproduced from Mathews et al., npj Quantum Information 12, 114 (2026). Every point assumes a defect-free chip.

The x-axis of that figure is not physically accurate. It reports *kd²/n* for a perfect chip. Real multilayer superconducting chips are fabricated with dead qubits and dead couplers, and at the scales these codes target you cannot assume defect-free manufacturing. Even a policy of discarding any chip above some dropout threshold does not recover the ideal figure — it just truncates the distribution.

What a fabricator actually needs to know is:

> Given a target tapeout success probability, what is the expected logical efficiency of a chip taped out with this code and this layout, once dropouts are accounted for?

That is the metric this project computes. Two tools exist that each measure half of it, and neither has been composed with the other.

**HAL answers *how expensive is this to build*.** Given a connectivity graph it produces an explicit multilayer superconducting layout — qubits on a first tier, couplers routed through higher tiers via bump-bond transitions and through-silicon vias — and a hardware complexity score *C_hw*.

**ACID answers *how easily does this break*.** Given a code, a connectivity graph, and a set of dead qubits and couplers, it compiles an ancilla-free syndrome extraction circuit that still measures the code, converting damaged checks into a subsystem code with gauge operators.

Each tool's output should inform the other. ACID knows which couplers are catastrophic to lose; HAL decides which couplers are long, which pass through many interfaces, and therefore which are most likely to fail and noisiest when they don't. Neither currently tells the other anything.

### 1.1 The design premise

We work with a single taped-out chip layout. The hardware decision is made once, at design time; the circuit decision is made per chip, once the dropout pattern is known, by compiling a syndrome extraction circuit around the damage. In the fullest version of the design (Phase 3), the chip's connectivity graph carries a buffer of extra couplers beyond what any one code requires, so that a *code*, not only a circuit, can be selected per chip to avoid the damage.

A dead component is the simplest case. On real devices, most components flagged as defective still work, but at several times the typical error rate. Excluding them from the code outperforms keeping them and informing the decoder (Drouet et al., arXiv:2607.12118). A dropout is therefore an exclusion decision made against a calibrated error map. Because that map drifts, the decision is revisited at each calibration cycle rather than once per chip. Phase 1 models dropouts as dead or alive; the exclusion threshold is a later refinement.

### 1.2 The figure of merit

**Primary.** For a target tapeout success probability (equivalently, a yield quantile — we keep the best *X*% of simulated chips), report *k·d_code²/n* at that quantile of the dropout distribution implied by the chosen layout. Here *d_code* is the code-level (dressed) distance of the subsystem code ACID produces. We write *d_circ* for circuit-level distance. The two are different quantities and are never interchanged.

Because *k·d_code²/n* is a code-level quantity, it does not see the circuit. The logical error rate is therefore reported alongside it (§5.8, §6.4). The logical error rate is the only one of the two that registers the *number* of minimum-weight failure paths, not just their weight.

**Secondary, as a tie-break.** The number of ACID contraction layers required for a given dropout pattern. This is the wall-clock cost of one syndrome extraction round, and two codes with equal expected efficiency are not equivalent if one needs twice the cycle time. Fewer layers is not automatically better. On the surface code at a fixed number of physical operations, three-layer schedules have higher logical error rate than four-layer ones (Anker & Debroy, arXiv:2512.10871), so this tie-break stays provisional until tested on qLDPC codes.

We expect this metric to reorder which codes are experimentally viable. Dropout susceptibility belongs to the pair (code, connectivity graph), not to the code alone (§2.1), so the reordering cannot be predicted from code parameters. It has to be measured once each family has a chosen connectivity graph.

### 1.3 Three phases

**Phase 1 — ACID-informed HAL on BB codes.** BB codes are the only nonplanar family that ACID compiles today on a published ancilla-free connectivity graph. Phase 1 fixes that family and the circuit compiler, and changes only the layout:

- a set of HAL modifications driven by heat maps imported from ACID;
- a noise mapper from chip geometry to per-gate error rates and per-coupler failure probabilities;
- a verification suite.

Controlling ACID's solver variance is part of the phase. Porting the Anker–Debroy scheduling heuristics to BB codes is a candidate result within it.

*Deliverable:* for the same BB code and the same ACID circuits, a modified HAL whose chips have lower logical error rate under dropouts and chip-derived noise than stock HAL's, with an ablation showing that the gain comes from the ACID information. This is the minimum viable result.

**Phase 2 — Beyond BB.** Extends the work to 2BGA, lifted-product, radial and tile codes. This requires three things:

- connectivity graphs constructed for those families;
- an ACID able to handle larger check weights, deeper contraction trees and less symmetric instances;
- PropHunt rebuilt for the ancilla-free paradigm.

With those in place, both directions of cross-tool heuristic operate.

*Deliverable:* the corrected figure across code families, plus an improvement ratio against an uncoupled HAL-and-ACID baseline.

**Phase 3 — Chips supporting multiple codes.**

*Deliverable:* for this much additional hardware complexity, this much marginal expected-efficiency gain.

### 1.4 The pipeline

Phase 1:

```
BB code + published connectivity graph (degree-5 or hexagonal)
        │
        ▼
ACID ──────────► one circuit per dropout orbit, replicated over solver seeds
        │
        ▼
Stim (uniform noise) ──► coupler dropout, qubit dropout, coupler co-activity maps
        │
        ▼
HAL (modified) ────► layout, C_hw, per-edge length, transitions, tiers
        │
        ▼
noise mapper ──────► per-CNOT error rate, per-coupler failure probability
        │
        ▼
Monte Carlo over dropout patterns
        │   (ACID circuit relabelled to each pattern; Stim with chip-based rates)
        ▼
LER against stock HAL, same code, same circuits
```

Phase 2 (full pipeline):

```
code
 │
 ▼
connectivity construction ──► mid-cycle connectivity graph
 │
 ▼
ACID ──────────► syndrome extraction circuit (per dropout pattern)
 │
 ▼
PropHunt ────────► reduced-ambiguity circuit
 │
 ▼
Stim simulation ───► coupler / qubit / fidelity / co-activity heat maps
 │
 │ (optional single-pass connectivity adjustment, back to ACID)
 ▼
HAL ──────────► layout, C_hw, per-edge length and transition counts
 │
 ▼
PropHunt (retuned) ─► circuit re-optimised against real gate error rates
 │
 ▼
Monte Carlo over dropout patterns
 │
 ▼
k·d_code²/n at a yield quantile + improvement ratio over baseline
```

---

## 2 · ACID

ACID compiles syndrome extraction for a CSS code under an arbitrary pattern of dead qubits and couplers. It works ancilla-free: a stabiliser is measured by contracting its support onto one of its own data qubits along a rooted spanning tree of that check's *local connectivity graph* — the subgraph of the device graph induced on the check's support. Its only condition on the connectivity graph is that every check's local connectivity graph be connected. No morphing circuit is required.

### 2.1 Fragmentation: trees and bridges

A *bridge* is an edge whose removal disconnects the graph — equivalently, an edge lying on no cycle. In a tree, every edge is a bridge.

Delete a coupler and the local connectivity graph loses an edge. If that edge was a bridge, the check's support splits into disconnected components, which become *quasi-stabilisers* — fragments that generally anticommute with each other. ACID then computes their anticommutation matrix, bidiagonalises it, and packages the result into product stabilisers and gauge operator pairs. The stabiliser code has become a subsystem code.

Because quasi-stabilisers are the connected components of the damaged local graph, a qubit cut off from the rest of a check becomes a weight-one quasi-stabiliser of its own. Later surface-code work obtains a finer gauge group by allowing weight-one gauge operators (Anker & Debroy). In ACID that finer gauge group is the default.

ACID demonstrates this on BB codes with two different connectivity graphs:

- In the degree-5 morphing connectivity, each weight-6 check's local graph is a tree, so the loss of any coupler produces anticommuting quasi-stabilisers.
- In an alternative degree-6 "hexagonal" connectivity, each check's six support qubits are wired into a cycle, and isolated coupler losses are absorbed with no fragmentation at all.

**This is the central empirical fact the project rests on: the same code, embedded two different ways, has qualitatively different dropout susceptibility.** The degree-6 graph is not the degree-5 graph plus edges — it removes one coupler and adds two — so the design space is the choice of local graph on each check, not monotone addition.

### 2.2 The scheduler and its objective

Contraction schedules are enumerated as rooted spanning trees with timings, pairwise compatibility between schedules is computed, and the assignment of schedules to layers is solved as a constraint program. The objective is:

1. minimise the number of contraction layers *L*, escalating from *L* = 2 and recording failure above *L* = 5;
2. within a fixed *L*, maximise the minimum number of times any stabiliser is measured;
3. tie-break on the total number of quasi-stabiliser contractions.

Nothing in this objective refers to circuit distance, hook errors, or the physical quality of individual couplers. **The scheduler is blind to how the chip is actually laid out — every coupler is treated as identical and every dropout as equally damaging.** This is the principal gap the project addresses.

Two published observations show what the coverage objective costs:

- **ACID's own data.** On ACID's 144-qubit BB code with hexagonal connectivity, a single dropped coupler usually *lowers* the logical error rate, and the dropout-free schedules are tightly bunched. The author's reading is that the heuristics push the solver into a sub-optimal schedule, and that the frustration a new constraint introduces lets it escape.
- **Anker and Debroy.** Optimising surface-code schedules over the LUCI representation, they find that a schedule maximising measurement count can have 5.5× the logical error rate of the default. They replace coverage with a linear proxy built from detector volume:
  - penalties for skipping a stabiliser two or three rounds running;
  - a penalty for stretching a detector with more than one neighbouring shape;
  - a penalty for basis changes;
  - a reward for deterministic measurements.

  Their framework is surface-code only, and they name its generalisation to higher-connectivity codes as a non-trivial open problem.

The solver also returns different-quality schedules across seeds and time limits, and schedules with equal objective value can differ in logical error rate (Anker & Debroy). Any heat map built from single solves therefore mixes damage with solver noise (§5.3).

### 2.3 Noise model and decoder

ACID's published results use uniform depolarising noise at *p* = 0.001 after each gate, plus bit- and phase-flip errors at reset and measurement, decoded with BP-AC (belief propagation with ambiguity clustering). Rounds are set to the dropout-free end-cycle distance. Published dropout data covers up to three dead qubits and three dead couplers, with 30 patterns sampled per configuration.

**Again: none of these error rates are conditioned on the physical realisation of the circuit.** Every two-qubit gate carries the same error rate regardless of whether its coupler is a short first-tier segment or a long route through four bump-bond transitions.

ACID assigns detectors itself, via Pauli flows. For some dropout configurations that touch a code boundary, it measures a set of operators that admits no valid split into superstabilisers and gauge operators (Anker & Debroy).

### 2.4 Symmetry and the number of distinct dropouts

Two dropouts related by an automorphism of the connectivity graph that also preserves the stabilisers and their Pauli types are the same experiment under uniform noise. For a code whose connectivity graph is arc-transitive under that group, all single-coupler dropouts are the same experiment. ACID reports arc-transitivity for its 288-qubit BB code with hexagonal connectivity; its 144-qubit code splits into two or more edge orbits.

Translations make every orbit large, but they do not by themselves make the graph arc-transitive. Which further automorphisms exist depends on the code, so the orbit count has to be computed for each instance, not assumed.

Open-boundary families lose translation symmetry at the boundary: tile codes, and planar codes derived from BB codes. Radial codes retain only a rotation of order *s*. **For those families we expect genuine, measurable variation in LER across coupler dropout locations, which is precisely where a dropout heat map carries information rather than restating a symmetry.** Once a layout assigns different geometry to equivalent couplers, the symmetry is broken for the noise, though not for the circuit (§5.3).

### 2.5 Which redundancy helps

Redundancy in the hardware graph and in the schedule is the resource that defect tolerance consumes: cycles in local connectivity graphs, slack in contraction orderings, multiple compatible schedules. Redundancy in the check set is a liability — the more checks touch a given qubit, the more of them a single dropout damages.

### 2.6 Initial heuristics

Three cheap quantities we can compute before or instead of a full solve.

**Edge connectivity λ.** For each check, the edge connectivity of its local connectivity graph. λ = 1 means some single coupler dropout fragments that check; λ ≥ 2 means none does. The hypothesis is that an edge whose loss produces more quasi-stabilisers produces a larger LER increase.

This is unsubstantiated and will be tested across several codes. Note also that for many codes the answer is uniform. On BB with degree-5 connectivity every local graph is a tree, so λ = 1 everywhere. On the hexagonal connectivity every local graph is a cycle, so λ = 2 everywhere. The quantity discriminates nothing on either, and is only useful where it varies across the code.

**sQetch for qubit dropouts.** sQetch is a GPU-based distance estimator reported as up to 800,000× faster than previous state-of-the-art, fast enough to brute-force millions of candidate codes per hour on a single GPU. Because a dead qubit changes the parity-check matrix, its damage is visible at the code level, and sQetch can estimate the post-dropout distance directly from the modified PCM — before any constraint solve. It accepts parity-check matrices, not circuits, and has no subsystem-code adapter. Its role is code-level screening: a cheap qubit-dropout heat map.

**Gauge basis selection.** ACID's bidiagonalisation is performed by plain Gaussian elimination and the resulting basis is whatever falls out; better choices exist and should reduce layer count. See §8.

---

## 3 · HAL

### 3.1 What HAL consumes

HAL takes a connectivity graph and embeds it. Nodes are placed on a regular lattice such that all nodes occupy distinct grid points and a heuristically maximal subset of edges can be routed in the plane without crossings. The remaining edges are routed through higher tiers using bump-bond transitions and through-silicon vias.

The graph is arbitrary — HAL treats all nodes identically and does not distinguish data from check qubits. In the published results the graph is inferred directly from the parity-check matrix, and therefore includes check qubits as nodes. That is the bipartite Tanner graph. It has no edges inside any check's support, so the published layouts and their *C_hw* values cannot be used with ACID. **For our purposes we work from the mid-cycle picture, where the mid-cycle code is essentially the BB/2BGA code itself, so we supply the ancilla-free connectivity of the code directly, and re-run HAL on it for every baseline.**

**A structural mismatch to resolve.** HAL's BB layouts are built on twisted tori, while ACID's are degree-5 and degree-6 mid-cycle connectivities of the untwisted BB code. Two options:

- treat the twisted-tori connectivities as mid-cycle inputs to ACID directly;
- or treat them as end-cycle codes and run the standard ancilla-based extraction circuit halfway through to obtain a valid mid-cycle state.

PropHunt offers a third possibility — optimising the choice of mid-cycle state by adding conditions to its search. Either way, *C_hw* must be recomputed for whatever connectivity we actually use; the published values correspond to different graphs.

HAL routes a fixed *capability graph*, the full set of couplers ACID is allowed to use. Each ACID circuit uses a subset of it, so ordinary changes of schedule never require re-running HAL.

### 3.2 First-tier placement is a design choice

HAL can choose the first-tier qubit placement automatically, but supplying a custom layout is often crucial in practice. The mitten-codes work, for instance, places all qubits on the first tier as a grid of modules from a thickness-3 decomposition, optimises module placement as a quadratic assignment problem, and lets HAL route the remaining couplers through higher tiers.

First-tier degree is bounded by planarity of the routing, not by lattice adjacency: HAL's radial code layouts show degree-5 connectivity realised on the first tier.

HAL's stages are:

1. a heuristically maximal planar subgraph, built by adding edges in ascending length order;
2. a Kamada–Kawai spring layout of that subgraph onto the qubit tier, followed by rasterisation;
3. straight-line routing on the qubit tier;
4. a modified A* that routes the remaining edges through higher tiers, resolving crossings with bump transitions and opening a new tier once one is congested.

The maximum number of bump transitions per coupler is a user-set parameter.

**Key point: nothing in any of this is motivated by which couplers matter most if they break.** Placement optimises geometric compactness and routability. That is the opening the ACID-to-HAL heuristics exploit.

### 3.3 The hardware complexity metric

*C_hw* is one plus the weighted mean of four rescaled quantities:

- the number of routing tiers;
- the average edge length across higher tiers, in units of the shortest edge length;
- the maximum average number of bump-bond transitions per coupler;
- the average number of through-silicon vias per higher-tier edge.

Each is normalised against a near-term-optimistic reference, so a surface code with single-tier planar nearest-neighbour connectivity sits at *C_hw* = 1.

The authors take *C_hw* ≲ 2 as the near-term feasible range. Edge count is not an explicit term, though it enters indirectly through congestion. Tier count is a fabrication cost. The quantities relevant to reliability are the per-coupler transition counts, which HAL also records.

### 3.4 Cost

Runtime is polynomial in edge count — minutes for most codes, up to a few hours for large or highly non-local lifted-product instances. Cheap enough to run repeatedly across candidate connectivities. For reference, HAL lays out the gross code [[144,12,12]] at five tiers.

---

## 4 · PropHunt

PropHunt operates on a completed syndrome measurement circuit. It builds the circuit's decoding graph and identifies ambiguous error configurations — distinct low-weight fault patterns producing the same detector signature, which the decoder therefore cannot distinguish. It removes them by rescheduling and reordering CNOT gates, guided by a MaxSAT solver.

Its loop is:

1. find an ambiguous decoding subgraph;
2. solve for the minimum-weight logical error within it;
3. enumerate candidate circuit changes;
4. prune those that fail an ambiguity check;
5. apply the surviving change.

It reports 2.5–4× logical error rate improvement on lifted-product and random quantum Tanner codes against a coloration baseline at *p* = 0.1%. Depth is the price. An independent comparison at realistic error rates found a PropHunt circuit reaching larger circuit distance at more than three times the depth, and losing on logical error rate as a result (Strikis, Browne & Beverland, arXiv:2603.05481).

Because rescheduling can require inserting gates, PropHunt's move set includes permuting non-commuting CNOTs with a compensating correction gate. Permuting CNOT*a*→*b* with CNOT*b*→*c* requires adding CNOT*a*→*c*, the third side of the triangle. Such moves may increase depth while improving error propagation structure.

We use PropHunt for two purposes.

**Better LER.** It directly targets the quantity our figure of merit measures, and it does so on an axis ACID's scheduler ignores entirely.

**Suppressing solver variance.** ACID's constraint solver returns different-quality schedules across runs, which shows up as spread in the dropout heat maps. We expect PropHunt to narrow this by converging circuits toward comparable ambiguity structure, partially rescuing poor ACID solutions. This should be confirmed by replicating a few dropout patterns across solver seeds and checking the post-PropHunt spread.

PropHunt enters the project in Phase 2 (§7). The Phase 1 result does not depend on it, and Phase 1 addresses solver variance directly (§5.4).

---

## 5 · Phase 1 method: ACID-informed HAL on BB codes

### 5.1 Why BB, and why HAL first

BB codes are the only nonplanar family for which an ancilla-free connectivity graph is published and ACID runs today: the degree-5 morphing connectivity, and ACID's degree-6 hexagonal connectivity. Every other family is further away:

- its connectivity graph must be constructed;
- larger weights or deeper contraction trees push ACID past what its solver currently handles (§6);
- PropHunt must be rebuilt for the ancilla-free paradigm before it helps at all (§7).

HAL, by contrast, already has the insertion points. What it lacks is information about which couplers matter. Phase 1 therefore fixes the code family and the circuit compiler, and changes only the layout.

**Testbeds.** ACID's 144-qubit and 288-qubit BB codes are mid-cycle codes with *k* = 12 and dropout-free end-cycle distance 6 and 12. The 288 code's hexagonal connectivity is arc-transitive, so its single-coupler heat map has a single value and gives HAL nothing to act on. The 144 code has at least two edge orbits (§2.4).

If the number of orbits proves too small for a convincing result, the secondary testbed is a code with broken translation symmetry that ACID can compile unchanged: either planar codes derived from BB codes by boundary construction (Liang, Eberhardt & Chen, PRX Quantum 6, 040330 (2025)), or tile codes. Each is given a connectivity graph by the construction of §6.1.

### 5.2 The pipeline is sequential

ACID takes a code, a connectivity graph, and a defect set. The physical chip layout does not enter its decision process at all — that is the gap being closed, and it means the composition runs one way rather than as a fixed-point iteration. Heat maps are computed from ACID output and HAL consumes them. The resulting layout then sets the noise under which the same circuits are evaluated.

Because ACID never sees the layout, the circuit compiled for a given dropout pattern is identical on every chip. Two chips can therefore be compared on exactly the same circuits, with only their noise differing.

*Heat map* here means the map recording how sensitive the code's logical error rate is to individual qubit and coupler dropouts (compare ACID's Figure 7). We build four:

- a coupler dropout heat map;
- a qubit dropout heat map;
- a coupler fidelity heat map;
- a coupler co-activity map.

The genuine outer loop in Phase 1 is the HAL variant. In Phase 2 it becomes code choice.

### 5.3 What the heat maps measure, and why full simulation is needed

A dead qubit changes the parity-check matrix, so its damage is visible at code level and can be estimated cheaply with sQetch.

A dead coupler whose loss fragments no check leaves the code unchanged. What changes is that the circuit gets longer and acquires new fault paths. A dead coupler that does fragment a check changes the subsystem code as well as the circuit. **Either way, coupler dropout damage is a circuit-level effect that a PCM-level distance calculation does not capture.** Measuring it requires the full path: ACID solve → Stim simulation of the resulting subsystem code.

**Symmetry and solver noise.** Take two dropouts related by an automorphism of the connectivity graph that preserves the stabilisers and their types. The circuit compiled for one, relabelled, is a valid circuit for the other with the same logical error rate under uniform noise. It remains the circuit ACID would use under layout-dependent noise too, since ACID does not see the layout. A heat map therefore needs one compile per orbit, not per coupler.

What it does need is replication. A single solve is one draw from the solver's distribution, so each orbit representative is compiled over several seeds and the entry is the median. Independent solves of two couplers in the same orbit differ only by solver variance. On the surface code, whose boundaries break the symmetry, the same spread mixes genuine position dependence with solver noise, and only a seed sweep separates the two.

### 5.4 Controlling ACID: variance, runtime, and the Anker–Debroy heuristics

Three measurements come first:

- solver runtime per dropout pattern;
- the split of heat-map spread into seed variance and location variance;
- whether the dropout-free anomaly of §2.2 survives when the dropout-free baseline is not a single solver draw.

Heat-map entries are defined relative to a pinned dropout-free baseline, and negative entries are kept, not clipped.

**Anchoring.** Anker and Debroy hint their solver with the default schedule, which also guarantees a result no worse than the default under their objective. The analogue here anchors each dropout solve to the pinned dropout-free schedule. A heat-map entry then records damage relative to a fixed circuit, not damage plus a fresh solver draw.

**The Anker–Debroy heuristics.** Of their three improvements, two do not carry over to periodic BB codes:

- the finer gauge group is already ACID's default (§2.1);
- excising qubits that support only a weight-one stabiliser is a boundary effect with no obvious counterpart in a periodic BB code.

What remains is the objective. Porting it to BB codes is a result in its own right, since its authors name the generalisation as open. But a better objective lowers the mean logical error rate; it does not remove ties between equal-objective schedules. The port is on the Phase 1 critical path only if anchoring and seed replication fail to bring solver variance below the effect HAL is expected to produce.

The acceptance test for any port is theirs: on the dropout-free surface code, minimising the ported objective must return one of the four symmetric canonical circuits. Their alignment term is defined on LUCI shapes, and it is the hardest to restate over rooted spanning trees.

### 5.5 Gate fidelity, failure and crosstalk as functions of geometry

This section justifies the modelling choice underlying §5.6 and §5.7.

*C_hw* is not purely a fabrication cost. One of its four components is the average edge length across higher tiers, and gate fidelity degrades with coupler length. A longer coupler means weaker effective coupling, hence a longer gate, hence more decoherence during the gate. So a higher *C_hw* heuristically implies worse gates — but the length subcomponent alone is a substantially better predictor than the composite score.

Even that is imperfect. Where the long, low-fidelity couplers sit in the circuit matters substantially more than their average length, which is precisely why the placement of gates is a lever at all.

We attribute gate infidelity to coupler length by default. Experimental work on flip-chip and multi-chip superconducting devices finds that one or two bump-bond transitions do not measurably degrade two-qubit gate fidelity or qubit coherence. No comparable data exists at the five to ten transitions HAL's layouts routinely produce. Rather than assert a value, the noise model carries two transition coefficients, one for gate error and one for coupler failure probability, swept across a range and reported as a phase diagram. The claim is conditional by design: given the coefficients experiment eventually supplies, this is the version of HAL to run.

Bump-bond and via manufacturing failure is a separate matter, and we treat it as a coupler dropout.

Rather than sampling lengths from a continuous distribution, we bin: a small set of predefined coupler length classes and dropout rates, which keeps the Monte Carlo tractable.

**Crosstalk** is a third channel, and the only one that needs both tools. A residual coupling between two gates active in the same timestep produces a correlated pair fault whose rate falls with their routed separation. Di Bella (arXiv:2604.01040) derives the resulting single-and-pair fault channel for BB codes, and a weighted exposure metric on logical supports that tracks logical error rate. HAL knows the separations; only the schedule knows which gates are simultaneous.

That derivation assumes an ancilla-based schedule in which each pair event lands on two data qubits. In an ancilla-free circuit every qubit is a data qubit, and whether a pair event stays at weight two must be checked before the metric is used.

Shortening critical couplers packs them closer together, while crosstalk wants simultaneously active couplers apart or on different tiers. The two objectives can conflict, and the layout has to balance them.

### 5.6 ACID to HAL: three maps, six insertion points

Three quantities pass from ACID into HAL.

**The coupler dropout heat map.** Computed by running the ACID → Stim pipeline once per coupler orbit, over several solver seeds, and recording the resulting subsystem code's effective LER. In hardware, a coupler's failure probability depends on both its length and the number of interlayer transitions it passes through, since each transition is an additional interface that can fail.

**The coupler fidelity heat map.** Computed by Monte Carlo:

1. assign elevated error rates to subsets of CNOTs;
2. simulate the resulting circuit in Stim;
3. extrapolate across runs which individual gates have the largest effect on LER for a given increase in gate infidelity.

Fidelity depends on length by default, per §5.5.

**The coupler co-activity map.** For each pair of couplers, how often both carry a CNOT in the same timestep. A co-activity counts only when the resulting pair fault lands on a minimum-weight logical support. Raw co-occurrence counts are the wrong quantity: in Di Bella's logical-aware layout the crossing count rose while exposure and logical error rate fell.

These enter HAL through six mechanisms.

**Planar subgraph.**

1. *Critical edges first.* HAL builds its qubit-tier subgraph by adding edges in ascending length order. Qubit-tier edges are the shortest and carry no transitions, so admitting critical edges first — ordering by weight rather than length — is the highest-leverage change. The weighted version is the maximum-weight planar subgraph problem, which is NP-hard; greedy insertion by weight with a planarity check is the standard heuristic. Because the planar subgraph drives the spring layout, this change also moves placement.

**Length.**

2. *Spring layout.* Placement sets the minimum possible length of each edge via the distance between its endpoints. Bias placement so that the most critical couplers — under either heat map — have the shortest achievable lengths.
3. *A\* routing cost.* When straight-line routing on a tier fails because it would create crossings, A* finds a path. Route the highest-impact couplers first, so they get the most direct paths and the most flexibility. Within A*'s cost function, which trades length against transitions, weight length more heavily for couplers whose fidelity heat map dominates, and weight transitions more heavily for couplers whose dropout heat map dominates.

**Transitions.**

4. *Transition bias.* Bias the router toward giving high-dropout-impact couplers fewer transitions. Lower tiers are a proxy for this; the transition count is the target.
5. *Crossing tie-break.* Whenever two couplers would intersect and one must be routed via a bump-bond transition, give the transition to the coupler whose failure the LER is less sensitive to. This costs nothing — the number of crossings is fixed by topology, only their assignment changes.

**Separation.**

6. *Co-activity penalty.* Add to the A* cost a penalty for routing near, or on the same tier as, couplers that the co-activity map marks as co-active with the one being routed.

Symmetry limits what these weights can say. On a BB code the dropout heat map takes one value per edge orbit, so the weights have only a handful of distinct levels. Placement still matters within a class. Laying a torus graph in the plane forces the edges that cross the seam to be long whatever their class, and where the seam falls is itself a lever.

### 5.7 HAL back to the circuit

Once HAL has produced a layout, each gate has a real error rate determined by its coupler's geometry. In Phase 1 that information flows back only as noise: the same ACID circuits are simulated with chip-based rates. Feeding it into circuit optimisation is Phase 2 work (§7).

### 5.8 Evaluation

At this point Phase 1 has, for each dropout pattern, one circuit and two chips. The two comparisons are reported separately.

**Paired.** Both chips run the same dropout pattern, drawn from a common distribution, with identical circuits; only the per-gate error rates differ. This isolates the fidelity channel and is the most statistically efficient comparison available.

**Own distribution.** Each chip's dropout patterns are sampled from its own coupler failure probabilities, and its logical error rate is averaged over them. This isolates the robustness channel: a layout that makes critical couplers less likely to fail wins here and nowhere else.

**Baseline and ablations.** The baseline is stock HAL, re-run on the same ancilla-free connectivity and evaluated with the same noise mapper. A modified chip that shortens long couplers will beat it under almost any length-dependent noise model, so the claim that ACID information is responsible needs an ablation ladder:

1. stock HAL;
2. HAL with a naive rule (shorten long-range edges, with uniform weights);
3. HAL weighted by criticality from the dropout-free circuit alone;
4. HAL weighted by the dropout heat map.

The thesis-bearing comparison is the last two rungs.

**Controls.**

- Several HAL seeds per variant, since spring layout is stochastic.
- *C_hw* reported beside every logical error rate, since a gain bought with an extra tier is a trade, not a win.
- The dropout-free chip reported separately.
- Results broken down by dropout count, and reported at yield quantiles as well as means.

**Statistics.** Resolving a relative difference δ in logical error rate at about 2σ needs roughly 8/δ² logical failures per arm in an unpaired comparison: about 800 at 10%, and 3,200 at 5%. For scale, the Anker–Debroy gains on the surface code were 14.5% and 23.6%. The failure budget sets the physical error rate and code size, and it is computed before building.

### 5.9 Loops in Phase 1

**Inner loop — per dropout orbit, per seed.** ACID solve → Stim → one heat-map entry. Scales with the number of orbits times the seed count, not with the number of couplers.

**Outer loop — per HAL variant.** Layout, noise mapper, then Monte Carlo over dropout patterns with relabelled circuits.

```
for each dropout orbit (coupler, then qubit):
    for each seed:
        ACID solve (anchored) → Stim → heat map sample
    heat map entry = median over seeds
for each HAL variant (stock, naive, dropout-free criticality, heat map):
    for each HAL seed:
        HAL(connectivity, heat maps) → layout, C_hw, per-edge geometry
        noise mapper → per-gate error rates, per-coupler failure probabilities
        Monte Carlo over dropout patterns:
            relabel orbit circuit to pattern → Stim with chip-based rates
→ paired LER ; own-distribution LER ; C_hw
```

---

## 6 · Phase 2: beyond BB

### 6.1 Connectivity construction

An ancilla-free connectivity graph exists in the literature only for the surface code, the colour code and BB codes. For 2BGA codes other than BB only the minimum-degree morphing graph is published; for radial, lifted-product and tile codes, nobody has designed one. **Morphing-derived graphs are minimum-edge by construction, which makes every check's local graph a tree and every coupler a bridge: the worst possible input to ACID.**

The object wanted is a minimum-weight support in which every check induces a 2-edge-connected subgraph. On a check of weight *w* the smallest such subgraph is a Hamiltonian cycle on its support, so the design variable is a cyclic ordering per check orbit:

- (*w* − 1)!/2 = 60 orderings at *w* = 6;
- for a translation-invariant code with one X shape and one Z shape, 60 × 60 = 3,600 combinations, exhaustible in seconds.

Score each combination by maximum degree, total edge count and edge length under a placement. The first check is whether ACID's hexagonal BB connectivity lies on the resulting Pareto frontier.

Adding edges is not a free improvement. More intra-support edges mean more rooted spanning trees per quasi-stabiliser. On ACID's superdense colour code, the resulting explosion in contraction schedules overwhelmed the solver until schedules were pruned. Solver tractability is a third axis alongside robustness and hardware cost, and a pure cycle gives the smallest non-trivial schedule domain.

The graph-theoretic name for the problem is a *hypergraph support*, a notion that originated in VLSI. Minimum-edge support is NP-hard. Plane supports with fixed vertex positions and minimum total edge length have exact and heuristic algorithms (Castermans et al.), though their objective is minimum-edge and must be replaced by the 2-edge-connected one above.

### 6.2 Reworking ACID

Codes beyond BB push ACID in three directions:

- higher check weight and deeper contraction trees, which enlarge each schedule domain;
- larger and less symmetric instances, which weaken the shape cache that makes current runs tractable;
- a need for scheduling objectives aligned with logical error rate rather than coverage.

Candidate restructurings include decomposing the global solve into layer-level subproblems coordinated by a master problem. Another is scoring generated layers with an ambiguity statistic that estimates the prefactor of the leading-order logical error rate rather than its exponent. None of this is justified until measured runtime on BB shows the solver is the bottleneck.

### 6.3 Screening and the connectivity loop

Cheap quantities filter candidates before expensive ones run:

- λ and bridge counts are pure graph computations;
- HAL layouts take minutes;
- sQetch distance estimates are fast enough to run at scale;
- full ACID solves and Stim LER simulation are reserved for survivors.

Should the constraint solves prove intractable at the sizes we need, the fallback is to develop faster heuristic replacements for the solve stage.

There is one potential shortcut worth testing. The qubit heat map — which is available cheaply — could inform a single pass of adding or removing couplers from the connectivity graph, which is then fed back into ACID. If that produces meaningful gains, test whether a pre-solve Δ*d* heat map achieves the same improvement; if so, sQetch is cheap enough to run that adjustment loop to convergence before ever invoking the constraint solver.

Phase 2 has three loops, at different costs.

**Inner loop — per dropout orbit.** ACID solve → PropHunt → Stim → one entry in the heat map.

**Optional connectivity loop — once, or to convergence if cheap.** Aggregate the heat maps to identify where couplers are most valuable, adjust the connectivity graph accordingly, and re-enter at ACID.

**Outer loop — per code.** Everything above, repeated for each candidate code.

```
for each candidate code:
    construct connectivity graph (§6.1)
    repeat (optional, until connectivity stabilises):
        for each dropout orbit (coupler, then qubit):
            ACID solve → PropHunt → Stim → heat map entry
        aggregate heat maps → adjust connectivity graph
    HAL(connectivity, heat maps) → layout, C_hw, per-edge geometry
    derive per-gate error rates from layout
    PropHunt(original ACID circuit, per-gate error rates) → final circuits
    Monte Carlo over dropout patterns → LER, sQetch distance
→ k·d_code²/n at a yield quantile ; improvement ratio vs baseline
```

### 6.4 Computing the figure of merit

At this point the pipeline has produced, for each dropout pattern, a final measurement circuit. We then run Monte Carlo over dropout patterns, simulating LER in Stim and obtaining code distance from sQetch. Keep the best *X*% of results, reflecting that not all fabricated chips are used.

**Form one — logical efficiency.** *k·d_code²/n*, computed per dropout pattern at rates implied by the chosen layout, and reported at the yield quantile. Because this is computed from the subsystem code's parity-check matrices, it needs no constraint solve. It responds to qubit dropouts and to coupler dropouts that fragment checks, and it is blind to everything the circuit does. This is the quantity that plugs directly into the logical-efficiency axis.

**Form two — improvement ratio over baseline.** The baseline uses HAL and ACID in their current, uncoupled form: generate a chip with HAL, generate a circuit with ACID, and simulate LER in Stim for a given dropout pattern. There is no PropHunt and no information passing between the two tools beyond the minimum needed to make them interoperate. Then run our protocol on the same code and take the ratio of LERs.

This is the metric that captures the circuit-level work, and its claim is: *given a code, we can run it on real hardware with better logical error rate.*

---

## 7 · PropHunt constrained by ACID

PropHunt was built for conventional syndrome extraction circuits. ACID's output carries constraints PropHunt knows nothing about. Because damaged checks have been converted into gauge operators, certain measurement orders must be respected: **no quasi-stabiliser that anticommutes with a product stabiliser's constituents may be measured between the first and last of them**, or the qubits leave the shared eigenspace of the already-measured operators. An arbitrary CNOT reordering can violate this and silently produce a circuit that does not measure what it claims to.

**The global analysis stage is unmodified.** Finding ambiguous decoding subgraphs and solving for minimum-weight logical errors within them operates on the detector error model, which is a spacetime object spanning all layers. It cannot be localised without missing exactly the cross-layer degeneracies that gauge structure creates. PropHunt's decoding graph handles arbitrary-arity fault mechanisms, so a graphlike detector error model is not required.

Circuit-level *H* and *L* come from the detector error model:

- Individual gauge outcomes are random, so detectors are products of gauge outcomes that lie in the stabiliser group.
- Observables must be bare logicals, since a dressed logical is randomised by gauge measurement.
- The solver still minimises over all fault configurations, so what it returns is the dressed circuit distance.

**The rewriting stage must be constrained to act within an ACID layer, and a repair protocol is needed for cases where within-layer measurement orders change.** Legality can be established by simulating the modified layer and checking its instantaneous stabiliser group — verifying directly that the circuit still measures the intended operators with the same outcome-to-operator map.

A competing approach, simpler to implement, is to run PropHunt with hard bans on any move that would flip the relative order of anticommuting gauge measurements. It is not clear which performs better; both should be tried.

**ACID remains the authority on feasibility**, and the move set is drawn from its own enumeration:

- a different rooted tree, root or timing for one measurement occurrence;
- or moving a stabiliser's measurement to another valid layer.

PropHunt supplies the fault–detector graph, the ambiguity test and the minimum-weight witness. The bridge from a witness back to the ACID occurrences that produced it is the piece neither tool supplies. On degree-5 BB every local graph is a tree with a single spanning tree, so the move space shrinks to root and timing. The interesting move space is on connectivities whose local graphs carry cycles.

### 7.1 Enlarging the search space

Two ways to admit moves that a naive legality check would reject.

**Allow contraction circuits to modify each other's supports.** ACID's compatibility criterion — that neither schedule may modify the support of the other quasi-stabiliser's Pauli string — is sufficient but not necessary. Two schedules that mutually perturb each other can be legal if the modifications cancel. Instantaneous-stabiliser-group verification detects this exactly where a criterion-based check would not.

**Permute consecutive gates.** If two gates commute this is trivially safe. Non-commuting gates can also be permuted at the cost of a correction gate, which may increase depth but may improve error propagation.

### 7.2 Physical realisability, and a route to connectivity feedback

**Feed the ACID connectivity graph into PropHunt so that no move ever introduces a CNOT on a coupler that does not physically exist.**

Then, once PropHunt has finished, run a pass checking which forbidden CNOTs it would have used had the graph permitted them, and whether including them would have meaningfully improved LER. Aggregated across all dropout patterns, this identifies which couplers are high-value in an average sense.

This is one way to rank missing couplers; direct construction (§6.1) is the other. In graph terms the additions are *chords* — edges joining two vertices already connected by a path, creating a cycle where none existed, which is exactly what converts a bridge into a non-bridge.

### 7.3 The depth budget

PropHunt can improve error properties at the cost of extra circuit depth. In an ancilla-free circuit, every added timestep is idling error on every qubit not participating in that timestep — a much worse exchange rate than in an ancilla-based circuit where ancillas idle regardless. Even in the ancilla-based setting, the independent comparison of §4 found the depth cost outweighing the distance gain.

The question is therefore quantitative from the start: **can the ambiguity reduction PropHunt buys with extra depth beat the idling error that depth costs?** There will be a threshold beyond which the answer flips. Sweep the allowed depth increase and locate it.

### 7.4 HAL to PropHunt

Once HAL has produced a layout, each gate has a real error rate, and that information can flow into circuit optimisation.

**Not by scheduling noisy gates late.** The natural-seeming heuristic — put unreliable CNOTs at the end so faults propagate through fewer subsequent gates — is the wrong criterion. What determines whether a fault is damaging is not how many data qubits it reaches but whether the resulting error pattern aligns with a low-weight logical operator. A fault copied to a check's entire support is a stabiliser and completely harmless; a fault copied to two qubits can be catastrophic if those two lie along a minimum-weight logical.

**The correct target is PropHunt, tuned so that high-error CNOTs sit where Pauli propagation does not feed into logical operators** — a statement about the circuit's detection regions and Heisenberg-picture propagation, not about temporal position. PropHunt's MaxSAT has one unit-weight soft clause per error; replacing unit weights with −log *p* turns minimum-weight into minimum-probability logical error. That is a parameter change in a weighted MaxSAT solver, not a new encoding.

**The simplest use** of gate-error information requires no change to PropHunt's objective at all: where PropHunt already selects among equal-depth options, use the varying CNOT error rates as the tie-break. This cannot affect feasibility or change what the circuit measures.

**Is re-running ACID worth it?** The most substantial lever conditioned on gate fidelity would be weighting contraction schedules by the collective fidelity of the spanning tree, pushing each stabiliser toward measuring through better couplers. But for many codes the differences between schedules are about *when* a given CNOT fires rather than *whether* it fires at all, in which case there is no lever. This needs quantifying before committing to a second ACID stage.

**Working assumption:** the pipeline runs HAL → revised PropHunt using the original ACID output, and re-runs ACID only if schedule reweighting proves to make a meaningful difference.

### 7.5 Open questions

Circuit-level distance would be a natural tie-break but is expensive to compute. PropHunt's own objective may already subsume it if it is maximising the minimum weight of an undetectable logical error locally.

The existence of a correction gate for any permutation is an algebraic fact and does not imply the resulting circuit is easy to implement or beneficial.

Work on canonical syndrome extraction circuits (arXiv:2309.08676 [?]) may help verify parts of this construction, though it is not needed to begin.

---

## 8 · Gauge basis selection

When ACID fragments a damaged check, it computes the anticommutation matrix *A* of the resulting quasi-stabilisers and bidiagonalises it, *VAW*ᵀ = diag(*I_g*, 0). The first *g* rows of *V* and *W* become gauge operators; the remainder become product stabilisers. **The span of the product stabilisers is unique, but the basis is not** — and ACID takes whatever plain Gaussian elimination produces, noting that better choices should improve performance.

The basis matters because the anticommutation relations among the chosen operators determine which contraction schedules are mutually compatible, and therefore how many layers the scheduler needs. A basis is chosen before its own schedulability is ever considered.

The basis does not change the gauge group itself. That is fixed by the quasi-stabilisers before bidiagonalisation runs (§2.1).

This is a known class of problem. The sparse-null-basis literature establishes three things:

- sparsest bases have matroid structure, so a greedy algorithm that repeatedly adds the sparsest available vector is optimal;
- the vectors worth considering are *circuits*, the minimal dependent sets, rather than arbitrary elements of the span;
- the NP-hardness lies entirely in the inner oracle of finding a sparsest vector, not in assembling the basis from it.

Practical implementations pivot for sparsity rather than searching exhaustively. Markowitz pivoting is the standard technique, and it is notably easier here than in its usual numerical setting, since over 𝔽₂ there is no stability penalty for choosing a pivot purely on sparsity grounds.

**The practical consequence is that we should not attempt to solve the NP-hard problem.** Instead, replace the objective with something cheap to optimise, generate multiple candidate bases, and evaluate them against a separate cost function. Minimum weight is the obvious cheap objective. The evaluation function is a property of the resulting anticommutation graph — its sparsity, or a chromatic-number-like estimate of how few layers a conflict-free schedule would need.

Several evaluation heuristics are worth trying, and identifying good ones is part of the work. Two refinements already look necessary:

- **Arc-consistency survival.** Scoring should count the fraction of schedules surviving *arc-consistency propagation* — the standard constraint-programming procedure of repeatedly removing values from a variable's domain when no compatible value remains in a neighbour's domain. Raw counts of schedules grow with support size and would collapse the objective into "maximise weight".
- **Leximin.** Scoring should be leximin rather than plain maximin, or bases tie whenever some gauge operator is pinned to a single schedule.

Whether the basis is also a lever for gate fidelity is open. Once HAL fixes the chip, the span of the product stabilisers is basis-invariant, but the supports of the chosen generators are not. Different generators induce different local subgraphs and potentially different couplers carrying gates. One repaired check and a handful of bases settles it.

Related work exists on the same underlying operation with a different trigger. GNarsil converts stabiliser codes into subsystem codes by splitting stabilisers into low-weight gauge operators, driven by weight rather than by defects, and reports that lifted-product stabilisers resist low-weight splitting.

---

## 9 · Phase 3: chips supporting multiple codes

Phase 3 is not a comprehensive project plan. It is a statement of next steps once the Phase 2 pipeline exists.

Define a *family* as a set of codes that can all be run on a single chip layout. A specific member is selected for a specific fabricated chip according to its dropout pattern.

The goal is to find families satisfying two conditions:

1. The chip's *C_hw* is not substantially higher than the *C_hw* HAL would produce for any single member code alone.
2. The union of the members' heat maps, taken pointwise-minimum, is uniformly suppressed across the chip — so that for any dropout location, some member of the family tolerates it well.

The second condition is the mechanism. Individual codes have order-of-magnitude variation in LER across dropout locations, and if members' weak points do not coincide, the family's worst case is far better than any member's worst case.

**What needs to be developed.**

We will explore which graph-theoretic properties of QEC codes indicate that two codes will work well together in HAL — that is, that their connectivity requirements overlap enough for a shared layout to be cheap.

HAL itself needs modification to lay out multiple codes simultaneously. The simplest starting point is a graph merge maximising overlap, but because all member codes must share the same physical qubit positions, something more involved is likely needed.

There is a genuine optimisation problem in combining codes so that, from each individual code's perspective, the extra couplers contributed by the other members maximise useful redundancy: additional cycles in local connectivity graphs, and hence more schedules and more dropout tolerance. Note that this reframes the apparent overhead: a chip provisioned for a family gives every member code more cycles than it would have alone. The extra cycles also enlarge schedule domains, which carries the solver-tractability cost of §6.1.

**The decoder.** Every chip, and every family member, runs a different detector error model. Whether one provisioned decoder can serve them all, or each chip needs its own, is a decoding question. It already exists in Phase 1, since dropouts alone vary the detector error model from chip to chip.

**Related prior art.** Strikis and Berent's work on selecting code twists to match a modular architecture becomes directly relevant if we pursue the alternative approach of adapting a code's twist to a defect pattern, which is a separate proposal from the family approach described here. Defect adaptation by lattice recoupling has been developed for topological codes. The minimally-invasive-alteration criteria for surface-code defect schemes are a useful specification to design against. Logical-qubit remapping in response to error-rate drift has been studied at the architecture level for surface-code patches.

---

## 10 · Reading

**Read first — the three tools the project is built from.**

- **ACID** — *Automated Compilation Including Dropouts*, arXiv:2512.01943 [V]. Read in full. Source at github.com/riverlane/ACID.
- **HAL** — *Placing and routing quantum LDPC codes in multilayer superconducting hardware*, arXiv:2507.23011, npj Quantum Information 12, 114 (2026) [V]. Read the algorithm description, the hardware complexity metric and its calibration, and Appendix A.
- **PropHunt** — *Automated Optimization of Quantum Syndrome Measurement Circuits*, arXiv:2601.17580 [V]. Read the MaxSAT formulation and the optimisation loop.
- **Anker & Debroy**, *Optimized measurement schedules for the surface code with dropout*, arXiv:2512.10871 [V]. The scheduling heuristics Phase 1 may port; Appendix C on minimum-weight path counts; Appendix D on ACID versus LUCI.

**Background on the codes.**

- Breuckmann & Eberhardt, *Quantum LDPC Codes*, arXiv:2103.06309, PRX Quantum 2, 040101 [V] — balanced product, lifted product, fiber bundle sections.
- Bravyi et al., arXiv:2308.07915, Nature 627, 778 (2024) [V] — bivariate bicycle codes.
- Lin & Pryadko, arXiv:2306.16400, Phys. Rev. A 109, 022407 (2024) [V] — two-block group algebra codes, equivalence criteria, and the public enumeration at github.com/QEC-pages/2BGA-codes.
- Steffan et al., *Tile codes*, arXiv:2504.09171 [V] — planar codes with open boundaries; [[288,8,12]] at weight 6.
- Liang, Eberhardt & Chen, *Planar quantum low-density parity-check codes with open boundaries*, arXiv:2504.08887, PRX Quantum 6, 040330 (2025) [V] — planar codes derived from BB codes.
- Scruby, Hillmann & Roffe, arXiv:2406.14445, PRX Quantum 7, 020310 (2026) [V] — radial codes.

**Mid-cycle and morphing circuits, in dependency order.**

- Vasmer & Kubica, *Morphing quantum codes*, arXiv:2112.01446, PRX Quantum 3, 030319 [V-ref] — the underlying operation.
- McEwen, Bacon & Gidney, arXiv:2302.02192, Quantum 7, 1172 (2023) [V] — the surface-code mid-cycle picture.
- Gidney & Jones, arXiv:2312.08813 [V-ref] — colour code middle-out and superdense circuits.
- Shaw & Terhal, arXiv:2407.16336, PRL 134, 090602 (2025) [V] — degree-5 BB morphing circuits and the general 2BGA framework.
- Shaw & Terhal, arXiv:2604.09797 [V] — 2BGA code optimisation, ISWAP versus CNOT, and post-dropout distance bounds.
- Chao, *Block algebra for morphing circuits*, arXiv:2606.12724 [V] — four explicit constructions with stated connectivity degrees.

**Defect handling.**

- LUCI, arXiv:2410.14891, Quantum 9, 1936 (2025) [V].
- Higgott, Anker, McEwen & Debroy, hex-grid defects, arXiv:2508.08116 [V].
- Drouet et al., *Error correction on an array of superconducting qubits with defective components*, arXiv:2607.12118 [V] — defect exclusion on a 120-qubit device; most defects are underperforming rather than dead.
- McLauchlan, Gehér & Moylett, arXiv:2405.15854, Quantum 8, 1562 (2024) [V] — methodological template for search under defect constraints.

**Circuits and geometry-dependent noise.**

- Di Bella, *Geometry-induced correlated noise in qLDPC syndrome extraction*, arXiv:2604.01040 [V] — same-tick crosstalk channel and weighted exposure; code at doi.org/10.5281/zenodo.19337541.
- Strikis, Browne & Beverland, *High-performance syndrome extraction circuits for quantum codes*, arXiv:2603.05481 [V] — left–right circuits, residual distance, and a depth-matched comparison with PropHunt.

**Tools and evaluation.**

- Stim, arXiv:2103.02202, Quantum 5, 497 (2021) [V] — github.com/quantumlib/Stim.
- sQetch, in *High-rate qLDPC processors*, arXiv:2607.28795 [V] — github.com/a7b/yarn. Accepts parity-check matrices only.
- Wolanski & Barber, *Ambiguity Clustering*, arXiv:2406.14527 [V] — ACID's decoder.
- QDistRnd, arXiv:2308.15140 [V] — reference distance computation.
- Webster, Jacob & Higgott, *Distance-finding algorithms for quantum codes and circuits*, arXiv:2603.22532 [V-ref].
- Coleman & Pothen, *The Null Space Problem I and II*, SIAM J. Alg. Disc. Meth. 7, 527 (1986) and SIAM J. Matrix Anal. Appl. 8, 544 (1987) [V-ref] — for §8.
- Castermans et al., *Short Plane Supports for Spatial Hypergraphs*, JGAA [V-ref] — for §6.1.

**Verification marks.** [V] confirmed against the arXiv listing, abstract, or paper body. [V-ref] confirmed via a reference list or journal citing page. [?] surfaced but not independently confirmed — do not cite without checking.

---

## 11 · Schedule for implementation

Phase 1 begins with a set of contained tests on BB codes, each gating one decision:

- orbit census;
- relabelling check;
- solver-variance decomposition;
- anchoring;
- runtime profile;
- the dropout-free anomaly;
- the gauge-group and excision checks;
- an asymmetric testbed;
- the noise mapper;
- a statistical power budget;
- the objective port;
- the crosstalk propagation check.

Dates: TODO.

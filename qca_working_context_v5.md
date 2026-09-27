# Working Context: Defect-Aware qLDPC Project

**v5 · 27 September 2026 · supersedes v4 of 2 September 2026 and addendum v4.1**

Companion to v16 (*Expected Logical Efficiency of Fabricated qLDPC Chips*). v16 is the project description: the metric, the tools, the pipeline and the three phases. This document holds everything else:

- how the collaboration should run;
- what has been tried and rejected;
- which ideas are live but not yet in v16;
- the calibration record on whose judgement to trust where.

Read this first in any new conversation, then v16.

**What changed from v4.** The project was restructured around a minimum viable result.

- Phase 1 is now ACID-informed HAL modifications on BB codes, with ACID solver variance and the Anker–Debroy scheduling heuristics folded in.
- The broader ACID and PropHunt rework moved to Phase 2.
- Multi-code chips moved to Phase 3.

Anker & Debroy, Di Bella, Drouet et al. and the relevant sections of ACID were read in full on 27 Sep. That reading closed several threads and corrected several assumptions, including some in the MVP planning note. The decoding-subteam material from addendum v4.1 is folded in here and summarised in its own document.

---

## Contents

**§0 · Document map**

**Part I: Principles for the collaboration**
1. The three standing rules
2. Search in multiple vocabularies
3. Sweep the literature at the start of every conversation
4. Read primary sources in full, not abstracts
5. Do the arithmetic
6. The red-team role
7. Calibration: who has been right about what
8. The five recurring failure modes

**Part II: Ideas not in v16**

9. Rejected, with reasons
10. Corrections and facts that shape how v16 reads
11. Live ideas not yet in v16
12. The EDA connection
13. Framings that took several attempts to get right

**Part III: Open threads and conversation protocol**

14. Open threads, ranked by how much they gate
15. External asks and competitive position
16. Conversation-start protocol

---

## §0 · Document map

Six documents make up the working set. Where they conflict, the order of authority is: v16 for what the project *is*; this document for how to work and what has been settled; the others for their own scope.

1. **`defect_conditioned_code_selection_v16.md`: the project overview.** The contained, current description of the project: the problem and metric, ACID, HAL, PropHunt, and the three phases. Phase 1 is ACID-informed HAL on BB codes, Phase 2 goes beyond BB, and Phase 3 is multi-code chips. It succeeds v15 and is written to be read cold. Anything stated there is the working definition of the project. Anything here that contradicts it is a mistake in one of the two, and should be raised rather than silently resolved.

2. **`acid_remake_redteam.md`: Phase 2 ACID and PropHunt rework.** Context for the broader second phase:
   - reworking ACID for larger check weights, deeper contraction trees and code families beyond BB;
   - candidate restructurings of the solve, such as layer-level decomposition, column generation and generated-layer scoring;
   - PropHunt-style changes in the ancilla-free paradigm;
   - where that idea currently stands after red-teaming.

   None of this work starts before the Phase 1 result exists. When it and this document disagree on Phase 2 detail, that document is newer on its own subject.

3. **`qca_mvp_test_spec_v1.md`: the initial BB tests.** Twelve contained tests (T1–T12) with decision rules and a dependency order, covering:
   - orbit census, relabelling, and solver-variance decomposition;
   - anchoring, runtime, and the dropout-free anomaly;
   - gauge-group and excision checks, and an asymmetric testbed;
   - the noise mapper and a power budget;
   - the objective port and the crosstalk propagation check;
   - an exclusion-threshold study.

   **A Claude Code agent must not implement or run any of these tests without direct approval from the user.** Reading the spec, planning, estimating cost, or asking clarifying questions is fine. Writing test code, launching solves or simulations, or cloning and modifying repositories to execute a test waits for an explicit go-ahead naming the test.

4. **`qca_decoding_context_v1.md`: decoding-subteam context.** What the decoding subteam is considering separately:
   - detector error model characterisation under dropout;
   - decoder provisioning across chips (superset DEM with masks, versus per-chip synthesis);
   - the mismatch penalty of a family-averaged decoder;
   - hardware cost of decoding under dropout, and the related literature.

   Relevant to this project through the decoder paragraph of v16 §9 and through §11.12 below. Otherwise it is a separate programme.

5. **`qca_unified_context_v1.md`: club-wide unified project ideas.** Broader context on unified project ideas across the club's subteams (theory, decoding, hardware). **May be somewhat out of date.** Where it conflicts with v16 or this document on anything about the theory project, v16 and this document win. Use it for cross-subteam interfaces and for what other subteams expect from theory.

6. **This document.**

---

# Part I: Principles for the collaboration

## 1 · The three standing rules

**Search before asserting novelty.** Every "nobody has done this" claim made without a systematic sweep has been wrong or overstated. Four searches is not a sweep. Instances since v4:

- the weight-one gauge generalisation to colour codes, occupied by arXiv:2604.05874;
- symmetry-exploiting circuit design for lifted-product codes, occupied by arXiv:2608.19917;
- the crosstalk-under-routing-geometry channel, occupied by Di Bella, arXiv:2604.01040.

**Do not manufacture explanations for gaps.** Producing a confident causal story for why nobody has done something, before the gap itself is established, remains the most reliable failure mode in this project's history. If a gap is real, say so plainly and say what would falsify it.

**Mark every citation.**

| Mark | Meaning |
|---|---|
| [V] | arXiv listing, abstract, or paper body confirmed directly. |
| [V-ref] | Confirmed via a reference list or journal citing page. |
| [V-prior] | Verified in an earlier session; spot-check before citing. |
| [?] | Surfaced but unverified; do not cite. |

A plausible-looking but entirely fabricated attribution ("Choe & König" for what is Strikis & Berent) once survived into a written document. A second attribution error was caught on 27 Sep: arXiv:2604.05874 had been recorded in project notes as "Zhu et al." It is **Wei, Zhang, Li, Kong, Wu & Guo**, *Adaptive deformation of color code in square lattices with defects* [V-ref, via the Drouet et al. reference list].

## 2 · Search in multiple vocabularies

Still the highest-yield habit in the project. The same idea appears under incompatible names across communities that do not cite each other.

| Concept | Also called |
|---|---|
| Twist / connection | voltage, exponent matrix, monomial choice, graph lift, shift assignment |
| Defect | dropout, dead qubit, fabrication defect, absent site, erasure, underperforming component, defect exclusion |
| Mid-cycle | code morphing, time-dynamic, middle-out, ancilla-free |
| Equivalence orbit | gauge redundancy, isomorphism theory, additive transformation, recoupling, remapping |
| Layout optimisation | place-and-route, physical design, EDA, layer assignment |
| Connectivity construction | hypergraph support, realization, set visualisation, network design |
| Ambiguity | degeneracy, indistinguishable syndromes, circuit-level distance, hook structure, residual error, minimum-weight failure multiplicity |
| Crosstalk | same-tick coupling, weighted exposure, aggressor/victim, switching-window overlap, signal integrity |
| Scheduler objective | LER proxy, detector volume, skip counts, alignment, measurement density |
| Layer-level decomposition | column generation, Dantzig–Wolfe, independent-set pricing, set covering |

A single-vocabulary sweep on the gauge-orbit idea would have declared it novel. Searching four vocabularies found prior art in classical QC-LDPC, topological QEC, quantum architecture, and fiber bundle codes simultaneously.

## 3 · Sweep the literature at the start of every conversation

The field is moving faster than the project. Anything stated as a gap has a shelf life measured in months.

**Verified 27 Sep, now load-bearing:**

- **Anker & Debroy, arXiv:2512.10871 [V].** Surface-code schedule optimisation over the LUCI representation. Details in §10.3.
- **Di Bella, arXiv:2604.01040 [V].** Geometry-induced correlated noise for BB codes; exposure metric. Details in §10.4.
- **Drouet et al., arXiv:2607.12118 [V].** Defect exclusion on IBM's 120-qubit ibm_miami. Details in §10.5.
- **Strikis, Browne & Beverland, arXiv:2603.05481 [V].** Left–right syndrome extraction circuits for arbitrary CSS codes; residual distance and extended-code distance; a proof that no non-interleaved single-ancilla circuit for the gross code reaches circuit distance 12; a depth-matched comparison against PropHunt and AlphaSyndrome. A copy dated 15 Sep 2026 exists; only v1 (5 Mar) has been read.
- **Liang, Eberhardt & Chen, arXiv:2504.08887, PRX Quantum 6, 040330 [V].** Planar BB-derived codes with weight-6 checks: (78,6,6), (268,8,12), (386,12,12) and others.
- **Steffan et al., tile codes, arXiv:2504.09171 [V].** [[288,8,12]] at weight 6; [[512,18,19]] at weight 8.
- **Eberhardt, Pereira & Steffan, pruned BB, arXiv:2412.04181 [V].** Explicit examples are low-rate ([[30,2,4]], [[66,2,6]]). A review notes the bivariate examples lack explicit stabiliser lists. Superseded in parameters by the two entries above; the pruning procedure itself is used by Liang et al.

**Surfaced, abstract-level or less:**

- **Barbell codes, arXiv:2606.06062.** Superconducting-targeted qLDPC codes shipped with a designated chip layout and syndrome cycle. Reported as IQM work in secondary coverage; authorship not re-checked.
- **Space-group topological codes, arXiv:2606.20548.** Uses HAL with a folded first-tier placement; five-tier layouts for two reflection codes.
- **Routed tile codes, Bae & Park, arXiv:2607.05897 [V-ref].**
- **Directional tile codes, arXiv:2606.19482 [V-ref].**
- **AlphaSyndrome, Viszlai et al., arXiv:2601.12509 [V-prior].** Reinforcement-learning syndrome scheduling for qLDPC codes, ASPLOS 2026.

**Re-check each session:**

- whether anyone has composed a layout tool with a defect compiler;
- whether ACID has a successor or a published runtime study;
- whether Anker & Debroy's heuristics have been generalised beyond the surface code;
- whether Di Bella or Strikis have extended exposure past two routing planes, past BB, or to ancilla-free circuits;
- whether the PropHunt group has published the LUCI/defect composition they flagged as future work;
- whether anyone has published per-component yield data or transition-dependent gate fidelity for multilayer superconducting processors;
- whether Roffe's group has published radial codes under defects.

## 4 · Read primary sources in full, not abstracts

Reading ACID, Anker & Debroy, Di Bella and Drouet et al. in full on 27 Sep produced five findings that no summary had contained:

- ACID already has the "more complete" gauge group, since its quasi-stabilisers are connected components.
- Anker & Debroy's objective leaves equal-objective schedules with different logical error rate (their Fig. 13).
- Di Bella omits the local single-gate term entirely.
- Di Bella's weight-2 truncation theorem is proved only for the ancilla-based schedule.
- Drouet et al.'s "defects" are mostly functional but noisy components, selected by explicit thresholds.

Four of the five changed the MVP plan.

The prior largest finding was of the same kind: HAL's published layouts are Tanner graphs, while ACID consumes intra-support graphs. It came from fetching both papers and reading one section against another. **Abstracts systematically omit what a tool consumes and emits**, which is exactly what matters when composing tools.

Corollaries, all demonstrated:

- When the user uploads notes on a paper, read them closely. They have repeatedly contained both the best new ideas and corrections to confidently-asserted errors.
- **When a tool has public source, read the source, not the paper's description of it.** ACID, HAL, PropHunt and Di Bella's pipeline are all public.
- Check a paper's identity before analysing it. One session analysed arXiv:2608.25267 as a Markov model of agents lying to neighbours; it is a sycophancy-mitigation paper using Bayesian Truth Serum inside GRPO.

## 5 · Do the arithmetic

Errors on both sides have been caught by two lines of counting. Earlier instances:

- Δ*k* = 0 from *k* = *n* − *r* − *g*.
- β₁ = 0 for a repetition-code base.
- The 1-in-3000 yield figure implying ~40,000 junctions.
- All spanning trees of one local graph having |*S*| − 1 edges.
- The number of bases of an 𝔽₂ space of dimension *g* ≈ 2^(*g*² − *g*).

Instances since v4:

- **Heat-map count.** BB-144 hexagonal has 432 couplers, so a single-coupler heat map is 432 solves naively but one solve per edge orbit with relabelling. "Hours per heat map" divides out to roughly 10–30 s per solve if every coupler is solved independently.
- **Hamiltonian cycles.** (*w* − 1)!/2 cyclic orderings: 60 at *w* = 6, 2,520 at *w* = 8. 60 × 60 = 3,600 orbit-pair combinations for a code with one X shape and one Z shape.
- **Power.** Resolving a relative LER difference δ at ~2σ needs ~8/δ² failures per arm unpaired: 800 at 10%, 3,200 at 5%.
- **Bulk scheduling degree.** A weight-6 check in a bulk tile code or BB code conflicts with at most 30 other checks (12 same-type, 18 opposite-type). The surface-code analogue is 12, not 4. The "tile codes are degree ~50" narrative compared a same-type count against a mixed-type one.
- **Pair-fault arithmetic (Di Bella).** At the reference point *J₀τ* = 0.04, BB72 logical error rates are 20–30% per memory experiment. That is a strong-crosstalk regime, so the 4.9× and 104× ratios are not near-threshold numbers.
- **Anker & Debroy.** Their d = 11 ILP has ~13,000 variables and ~60,000 constraints. The optimum is found in about 5 minutes; proving optimality takes the rest.

If a claim has a number in it, check the number.

## 6 · The red-team role

- **Steelman, then find the load-bearing weakness.** The user's ideas have generally survived; what changes is the framing, the objective function, or the substrate. Reframing an idea into its defensible form is more useful than accepting or rejecting it.
- **Concede specifically.** Identify exactly what was wrong and why, rather than retreating globally. Several of the best results came from conceding a point and then finding what did survive.
- **Say "this is BS" when it is.** The user asks for this explicitly and responds well to it. A wrong idea identified in one sentence beats three paragraphs of hedged agreement.
- **Distinguish "I verified this" from "this is plausible."** Label inferences as inferences.
- **Weight competitive position, not just interest.** The question is not whether something is interesting but whether someone with more people and a commercial reason to win is already doing it.
- **When two estimates conflict, name the measurement that settles it** rather than arguing the estimate. The MVP test spec is largely a list of such measurements.
- **Prefer collapse to machinery.** The user consistently prefers reducing a hard problem to a tractable one over deploying sophisticated algorithms. Examples: collapsing ∀*D* robust feasibility to Monte Carlo over a small candidate set; replacing connectivity search with cyclic orderings. Offer the collapse first.

What has not been useful:

- enthusiasm without verification;
- effort estimates given without decomposing the work;
- recommending the most general version of a construction when the filters only exist in the specialised case;
- hedging a disagreement into uselessness;
- cross-domain analogies (tensor networks, neural networks) offered as mechanism.

## 7 · Calibration: who has been right about what

The user has been more reliable on strategy; the agent on mechanism and arithmetic. That summary needs the usual qualification: the user has also been right on several mechanism questions where the agent was sloppy. Roughly two-thirds of substantive errors in prior sessions were caught by the user or external review rather than by self-correction. Weight confidence accordingly.

**Earlier cases where an idea the agent called weak was right after reframing:**

| Idea | Initial agent call | Reality |
|---|---|---|
| Chips supporting families of codes | "shell game, probably decoration" | Correct; the min-over-heat-maps argument is nearly self-proving once the maps exist (now Phase 3). |
| Stay at 1×1 rather than generalising to *r* × *r* | "the open ground is *r* × *r*" | Correct; *r* × *r* is the morphing authors' obvious next paper. |
| HAL should not be demoted | "λ is free, so HAL is a cost column" | Backwards; a free axis is a reason to spend budget on the expensive one (now Phase 1). |
| PropHunt needs a dense circuit to work | agent had contained it to intra-tree ordering | Correct; the containment removed the mechanism. |
| PropHunt can suppress ACID solver variance | "the mechanism doesn't work" | Partially correct, and a useful diagnostic either way. |

**Cases since v4, user right:**

| Claim | Agent position | Resolution |
|---|---|---|
| ACID solver variance is large in practice; single solves are not representative | Agent had said relabelling orbit members "removes solver variance for free" | User right on the variance. Relabelling hides it rather than removing it. Correct procedure: seeds per orbit representative. |
| Heat-map edge classes do not collapse the HAL design space | Agent said design reduces to "which edge classes get which tier" | User right; placement and seam position vary length within a class. |
| Tile codes map to the same tractability question as colour codes (long-range edges intrinsically allowed) | Agent had called tile codes "fundamentally harder, degree ~50" | User right; bulk scheduling degree ≤ 30, same as BB. |
| Minimum-edge connectivity search is ∃*G* ∀*D* only if every dropout must be feasible; ACID always produces *some* schedule | Agent framed robust feasibility as the problem | User right; collapse to Monte Carlo comparison of candidate graphs. |
| The shape cache already exists in ACID; the new content is edge selection above it | Agent conflated the two | User right. |
| Separator/expander arguments do not apply to finite instances; ZDDs oversold | Agent used them | User right. |
| Hyperedge DEMs are the norm for qLDPC; the graphlike constraint is surface-code-specific | Agent inherited Anker & Debroy's graphlike control | User right (addendum 10.17). |
| Detector sets change under qubit dropout and fragmenting coupler dropout, so a priors-only test is wrong | Agent proposed priors-only | User right. |
| PropHunt under drift is a bank of circuits switched on the calibration timescale | Agent gave it one sensitivity paragraph | User right; this is also the natural response to Drouet's drifting defect sets (§11.6). |
| Pruned BB is superseded by newer planar codes | Agent ranked it first as a testbed because ACID names it | User right; demoted to fallback. |
| The HAL-first MVP ordering | not contested | Endorsed; HAL has the insertion points and needs only information. |

**Cases since v4, agent right:**

| Claim | User position | Resolution |
|---|---|---|
| The Google objective port is not needed to justify orbit representatives; equal-objective schedules still differ in LER | Port needed to lower variance first | Anker & Debroy Fig. 13 [V]; anchoring plus seeds is the variance fix. |
| ACID already has the finer gauge group; (*V*, *W*) cannot produce it | "(*V*, *W*) should already find the weight-1 trick" | The gauge group is fixed by the quasi-stabilisers before bidiagonalisation. |
| Excision is a boundary effect, probably a no-op on periodic BB | "Easy to implement" | Implementation is easy; applicability is the question (spec T6b). |
| "ACID only works with BB" is false | Stated in the MVP note | ACID demonstrates surface, colour and BB; BB is the only published *nonplanar* case. |
| Kalman convergence is the wrong theorem; identifiability and Cramér–Rao are right | Proposed Kalman convergence | Addendum 9.17. |
| Compound-channel and mismatched decoding, not random coding, frame a family-averaged decoder | Proposed Shannon's average-code argument | Addendum 11.22. |
| Naive mean field without an entropy term collapses to point masses | User's original entropy term was correct and had been overridden | Agent's earlier override was wrong; the collapse argument survived all pushback. |
| The pipeline's name is column generation with independent-set pricing, not Neural Diving | Label adopted from a prior session | Neural Diving needs supervised training data that does not exist here. |

**Cases where the user's mechanism was wrong:**

- Δ*k* = −*g* (gauge pairs are paid for by collapsing generators, so Δ*k* = 0).
- ℤ₂ × ℤ₂ transpositions as fiber automorphisms.
- "Different HAL layouts give different mid-cycle states."
- "A fault propagating to a weight-4 subset objectively matters more than one propagating to weight-3."
- "Dense codes such as tile."
- The runtime figure "hours per heat map" stated as a blocker before any runtime was profiled.

**Cases where the agent's mechanism was wrong:**

- "Logical operators are the centraliser of the gauge group modulo the gauge group."
- Proving gauge-invariance in one argument and presenting it as covering both.
- Framing dressed distance as an exponential cost when the solver absorbs it.
- Stating that a validated DEM for an ACID-repaired code was unbuilt without checking LightStim.
- Radial single-shot capacity attributed to metachecks rather than confinement.
- "Relabelling removes solver variance."
- "Edge classes collapse the layout design space."

**Operational consequence.** On "should we do X or Y," "is this worth competing on," and "what is the real deliverable," take the user's read seriously and argue against it only with evidence. On "does this identity hold," "what does this tool consume," and "how big is this space," verify independently and expect errors on both sides. On any question with a number in it, do the arithmetic before either party's intuition is trusted.

## 8 · The five recurring failure modes

1. **Manufactured causal stories for gaps never established.**
2. **Cross-domain intuition transfer.** Surface code → qLDPC; R-spectral → 𝔽₂; decoder-side → circuit-side; Tanner graph → ancilla-free graph; layout → circuit; fiber-locality → base-locality; "errors spreading is bad" → a setting where spreading to the full support is benign. *Recurred since v4:* surface-code bulk variation read as evidence about BB solver variance, although the surface code's boundaries break the symmetry the BB argument relies on. Also, Anker & Debroy's graphlike-detector constraint was inherited into a qLDPC setting.
3. **Insufficient search before novelty claims, and searching in only one vocabulary.** *Recurred since v4:* weight-one gauge generalisation; crosstalk under routing geometry.
4. **Over-containment.** When told an idea has a legality or tractability problem, the instinct is to shrink its scope until the problem disappears, which twice removed the mechanism that made the idea worth having. Constrain the move set, not the analysis; verify the outcome, not the precondition.
5. **Building before naming.** A hand-rolled algorithm has repeatedly turned out to be a named problem with a mature literature:
   - gauge basis selection is the sparse-null-basis problem;
   - connectivity construction is the hypergraph support problem;
   - the Markov "agents" layer generator is naive mean field on a Potts MRF;
   - layer-by-layer decomposition is Dantzig–Wolfe column generation;
   - the KL trust region is mirror descent.

   *Also recurred as mislabelling:* naming a method after a superficially similar one (Neural Diving; Raghavan–Thompson rounding for a quadratic conflict count). Before designing a search, spend twenty minutes asking whether the problem has a name, and check that the name's assumptions hold.

---

# Part II: Ideas not in v16

Everything below was discussed and falls into one of three kinds:

- (a) live but not yet load-bearing;
- (b) rejected for a specific reason worth recording, so it is not re-proposed;
- (c) a correction or fact that shapes how v16 should be read.

## 9 · Rejected, with reasons

**9.1 Defect-conditioned code search as the primary project.** A dead coupler leaves the parity-check matrix unchanged, so every clean algebraic invariant is defect-independent by construction. The search also cannot hold layout fixed while varying the algebra. Superseded by the phase structure in v16.

**9.2 Twist selection as defect adaptation.** A twist is a bijection on sites; it cannot turn a dead site into an unused one, and for coupler defects it does not change the code. It survives only as design-time provisioning. Strikis & Berent (arXiv:2209.14329) §B is the closest prior art; a defect-conditioned version is not obviously occupied.

**9.3 "Clifford deformation" as a search axis.** A generic *S* destroys quasi-cyclicity; an *S* that preserves it is already quotiented out. Destructive or empty.

**9.4 Automorphism relabelling by lattice translation as a defect response.** Translation preserves every pairwise defect separation, so the whole orbit is one instance. The inverse use, relabelling ACID circuits across an orbit to save solves, is valid and in v16 §5.3.

**9.5 Local stabiliser-level edits via fiber automorphisms.** Forbidden by a two-line bound: if σ moves coordinate set *S*, then *c* + σ(*c*) is supported in *S*, so *d*(*C*) ≤ |*S*| or σ fixes *C* pointwise.

**9.6 Gauge basis (*V*, *W*) as a fidelity lever. Status: open, not rejected.** The original reason (the basis cannot change which couplers carry gates) does not follow: generator supports are basis-dependent. v16 §8 states it as open; thread T2.10 settles it. It is not a route to the weight-one gauge group (§7).

**9.7 "Schedule high-error CNOTs late."** Damage depends on alignment with a low-weight logical, not on how many qubits a fault touches. Replaced by ε-weighted PropHunt (v16 §7.4). The revised "impact" metric is alignment-aware, but its overlap-with-a-logical definition is not stabiliser- or gauge-invariant. Fix a representative or redefine on classes (§11.3).

**9.8 Hard-filtering "dominated" contraction schedules before the solve.** Vacuous by counting: all spanning trees of one local graph have |*S*| − 1 edges.

**9.9 Inverting the LER scaling law to extract *d_eff* from one measurement.** Two unknowns; the prefactor counts low-weight fault paths, which is exactly what circuit optimisation changes.

**9.10 Layer-by-layer placement refinement in HAL.** A reinvention of routability-driven placement done the expensive way. See §12.

**9.11 SBIR/T3CP Patent Holiday funding.** Requires a DoW-origin patent and a small business applicant. NSF STTR, DARPA QBI, AFOSR and ARO are the appropriate vehicles.

**9.12 Merging PropHunt's ambiguity analysis into ACID's constraint solve.** Chicken-and-egg (ambiguity needs a complete circuit); cost; and the pipeline is sequential. Salvageable version: score over ACID's already-enumerated compatible schedules, one way.

**9.13 Hook-ZNE as a novel contribution.** Already PropHunt's own contribution [V].

**9.14 Converging on a connectivity graph from both ends.** Greedy-add and greedy-remove on a non-submodular objective have no duality gap closing. Superseded by cyclic-ordering enumeration (v16 §6.1).

**9.15 Generalising BP to weighted Tanner graphs so hardware asymmetry can enter.** Priors are already per-fault-mechanism in the DEM, and trainable per-edge weights are a decade-old line (Nachmani et al. 2016; Liu & Poulin 2019; and successors).

**9.16 A Bayesian or Kalman filter to update priors across a grid.** Occupied by the DEM-estimation literature (Spitz et al.; Blume-Kohout & Young; Wagner et al.; Arms et al.; Bhardwaj et al.; arXiv:2511.01080). Surviving slice: no one has run it on a defect-repaired subsystem DEM.

**9.17 Proving Kalman convergence for QEC priors.** Wrong theorem; the content is identifiability (§11.11).

**9.18 Naive mean field over per-node schedule distributions, without an entropy term.** The objective is multilinear, hence linear in each node's distribution with neighbours fixed. One coordinate sweep collapses every distribution to a point mass. Clamping produces a linear-fractional program that also attains its optimum at a vertex (Charnes–Cooper). An explicit entropy regulariser is required. Details in `acid_remake_redteam.md`.

**9.19 "Neural Diving" as the name for the layer-generation pipeline.** It requires supervised training data that does not exist for the target codes. The correct name is column generation with independent-set pricing (Mehrotra–Trick 1996; Gualandi–Malucelli 2012).

**9.20 Raghavan–Thompson rounding as the guarantee for sampled layers.** Layer conflicts are quadratic in the sampled choices, and a layer needs zero conflicts. The repair step, not the rounding theorem, produces a layer, and it has no guarantee.

**9.21 "Start at minimum degree and add high-value edges" as the connectivity search primitive.** The degree-5 BB local graph is a branching tree, not a path. Reaching a cycle by addition costs more than choosing a cycle directly, and added edges manufacture schedule explosion.

**9.22 Morphing-derived connectivity as a robust ACID input.** Minimum-edge by construction: tree local graphs, λ = 1, every coupler a bridge.

**9.23 Mean expansion depth to first ambiguity as a layer-scoring figure of merit.** To leading order it estimates effective distance, which is the quantity PropHunt's Fig. 1(b) exists to discredit. The statistic wanted is the fraction of expansions halting at the minimum depth, which estimates the prefactor.

**9.24 Raw coupler co-activity counts as the crosstalk weight for HAL.** Usage count is not damage (the §9.7 pattern). In Di Bella's logical-aware layout the crossing count rose from 522 to 542 while exposure and LER fell [V]. Use logical-weighted co-activity.

**9.25 Porting the Google objective as the fix for ACID solver variance.** Equal-objective schedules differ in LER [V, Anker & Debroy Fig. 13]. The port lowers the mean, not the ties. Variance is handled by anchoring and seed replication (v16 §5.4).

**9.26 Retuning HAL spring parameters from the qubit heat map.** Qubit failure is layout-independent in the model, so there is no lever. Parked unless a clustered-defect model with data and a super-additive co-located damage result (the §11.2 additivity test) both appear. Spring layout also cannot separate graph neighbours, which are the pairs most likely to be co-damaging.

**9.27 Inheriting a graphlike-DEM constraint.** Anker & Debroy converted ACID output into LUCI plays partly to keep matching decoders usable. That is a surface-code comparison control. BB, 2BGA and radial codes use hyperedge DEMs and BP-family decoders.

## 10 · Corrections and facts that shape how v16 reads

### 10.1 Naming and objects

**ACID's "144" and "288" BB codes are mid-cycle codes.** Each has *k* = 12, dropout-free end-cycle distance 6 and 12, and 432 and 864 couplers on the hexagonal connectivity (360 and 720 on degree-5) [V, ACID §5 and App. D]. They are not the [[144,12,12]] and [[288,12,18]] codes used directly. Confirm which polynomials the repository uses before quoting parameters.

**d_eff was overloaded; v16 splits it into *d_code* and *d_circ*.** Any older note using *d_eff* should be read with that split in mind.

**HAL routes the Tanner graph in its published results.** It has no intra-support edges, so the ~150-layout database is unusable for ACID as-is. v16 states the consequence. The corollary that bites in practice: every baseline *C_hw* is recomputed on the ancilla-free graph with identical HAL settings (grid size, edge margin, node size, max bump transitions per coupler with default 10, repeats per graph). Never compare against the published table.

**The two figures of merit measure different damage.** *k·d_code²/n* is code-level and blind to the circuit. The LER comparisons carry coupler and circuit damage. Never present them as measuring the same thing. Fragmenting coupler dropout does change the subsystem code, so "coupler damage is invisible at code level" holds only for non-fragmenting dropout.

**Dropout tiers.**

- *Tier A* is non-fragmenting coupler dropout. The circuit changes but the code does not.
- *Tier B* is qubit dropout and fragmenting coupler dropout. The detector set changes entirely.

An earlier note said tier A changes "priors only." That is too strong, since any schedule change moves detectors, and the user corrected the priors-only test for this reason. Treat tier A as "code unchanged, circuit and detectors changed."

### 10.2 ACID facts not in v16

- **Detector assignment.** ACID assigns detectors itself through Pauli flows, including extra detectors from repeated quasi-stabiliser measurements [V, ACID §3.4]. What it lacks is full provenance from a fault column to instruction to ACID occurrence, which PropHunt integration needs.
- **Stim contract.** Memory experiments use noiseless Pauli-product initialisation and close-out. Anker & Debroy replaced these with noisy operations when comparing [V].
- **Detector error model shape.** ACID sometimes produces DEMs that do not admit approximation as decoding graphs. The author attributes this to overlapping redundant measurements [V].
- **Compatibility.** The compatibility criterion (neither schedule may modify the other's Pauli string) is sufficient, not necessary. Instantaneous-stabiliser-group verification is exact and more permissive.
- **Colour codes.** On the degree-3 colour code a single dropped coupler usually forces *L* = 4, a *feasibility* effect of dense overlap. On the superdense colour code the problem is the *schedule explosion* of v16 §6.1. These are two mechanisms, and adding edges helps the first while manufacturing the second.
- **Pinning schedules.** ACID's LUCI reproduction (App. E) pre-assigns colours to layers and measures the price: *L* = 4 always, against *L* = 2–3 for free ACID. This is the measured cost of pinning schedules to shrink the search.
- **Logical symmetry under dropout.** Dropouts break the translation symmetry that BB automorphism-based logical gates rely on [V, ACID §5]. This is relevant downstream of any memory result.

### 10.3 Anker & Debroy, arXiv:2512.10871 [V]

The paper reports three improvements over the original LUCI prescription:

- **Finer gauge group.** Weight-one gauges; 8.2% / 14.9% LER improvement at 1% / 3% dropout.
- **Excision.** Removes boundary measure qubits that support only a weight-one stabiliser and are never a root or waypoint.
- **ILP over LUCI diagrams.** A further 6.8% / 10.3%.

The total is 14.5% / 23.6% on the *d* = 11 surface code, SI1000 noise at 0.1%, over 4*d* rounds.

The objective is −*m* + 6·*s*₂ + 5·*s*₃ + 12·*a* + 2·*b*:

| Symbol | Term |
|---|---|
| *m* | deterministic measurements |
| *s*₂, *s*₃ | skip twice, skip thrice |
| *a* | alignment (more than one neighbouring shape stretching a detector) |
| *b* | basis changes |

The weights come from first-principles detector-volume arguments, not tuning.

Further details:

- **Solver setup.** CP-SAT, 5-minute limit, hinted with the default LUCI schedule.
- **Fig. 13.** LER rose then fell between 50 and 120 s while the objective stayed nearly flat.
- **Three-round schedules.** These raise LER by 45% / 8.1%, but the comparison is at fixed rounds; the authors note they may win when time-like limited.
- **Measurement maximisation.** The measurement-maximising schedule in their Fig. 6 has 5.5× the default's LER.
- **ACID versus LUCI (App. D).** Canonical LUCI beat ACID's surface-code schedules after both were converted to LUCI plays.
- **Minimum-weight paths (App. C).** The number of minimum-weight paths is the coefficient of the leading-order LER term and is hard to encode in an ILP. This is the thesis sentence for the Phase 2 layer-scoring idea.

### 10.4 Di Bella, arXiv:2604.01040 [V]: what it covers of our noise model

The setup:

- Single author (Cambridge). Strikis is acknowledged for suggesting the crossing model, and Noszkó for the logical-aware optimisation.
- The code is BB, with the depth-8 ancilla-based Bravyi schedule.
- The model is a same-tick ZZ-type interaction between simultaneously active gate blocks, with a proximity kernel κ of routed closest-approach separation.
- The retained single-and-pair channel is controlled to *O*(Θ⁴).

The results:

- A matching-number bound under the crossing kernel.
- Weighted exposure under positive kernels.
- The AKP planar summability threshold α > 2.
- Logical-aware two-swap search reducing worst-case family exposure by 26.11%.
- A MAP reduction to weighted decoding (App. G).
- Kernel scales anchored to Kosen, Barrett and the Kunlun simultaneous-CZ excess (App. H).
- Code released.

Coverage of our model:

| Our component | Covered? |
|---|---|
| Length → gate infidelity | **No.** The additive-local term is explicitly omitted; local noise is uniform *p*. |
| Bump/TSV transitions | **No.** Single plane versus two routing planes only. |
| Coupler failure / dropouts | **No.** |
| Crosstalk | **Yes, in form.** Replace our heuristic term with it. |
| Ancilla-free circuits | **No.** Theorem 2 (pair event → weight-2 data Pauli) relies on ancilla elimination; spec T11a checks ACID circuits. |
| Multi-tier HAL geometry | **Listed as future work.** Path-integrated coupling over routed curves; multilayer and open-boundary schemes. Their Table I lists HAL as the multilayer precedent with no noise model. |

Caveats:

- Spearman 0.893 is pooled over kernel families; it is 0.552 across 22 layouts.
- Exposure separates optimised from random layouts but does not order the random band.
- The hierarchy vanishes as decay length grows toward drive-like crosstalk (*ξ* = 8 gives comparable LERs).

### 10.5 Drouet et al., arXiv:2607.12118 [V]

Setup:

- Sydney and IBM; *d* = 5 surface code on ibm_miami (120 qubits, square lattice).
- No mid-circuit reset; mid-circuit measurement quality limited.

Exclusion thresholds used:

- couplers with error above 5%, or with no calibration data (median 2.83×10⁻³, worst 1.69×10⁻¹);
- measure qubits with readout error above 7×10⁻² (median 2×10⁻²);
- data qubits with both *T₁* and *T₂* below 80 µs.

Z-basis LER per round:

| Strategy | LER per round |
|---|---|
| No defect handling | 4.49% |
| Noise-informed decoding | 4.22% |
| Excluding couplers | 2.71% |
| Also excluding a data qubit (removes a row) | 1.62% |

Excluding good components did not help.

- Part of the 1.62% gain is the smaller code with fewer minimum-weight logicals; the authors say so.
- Defect sets differed between run dates.
- Discussion: most defective components work but at several times the typical error rate, and choosing the exclusion threshold is an open trade-off.

What it supports: exclusion beats decoder-only handling. What it does not provide: a quantitative model to import.

### 10.6 Other circuit and noise facts

- **Strikis, Browne & Beverland.** Residual distance and extended-code distance are code-capacity quantities that bound circuit distance cheaply for non-interleaved single-ancilla circuits. Interleaving cannot decrease circuit distance. Whether any of this transfers to ancilla-free contraction is open; do not cite it as a solution. Their left–right structure covers the two-block partition for ancilla-based lifted-product circuits.
- **Decoders run on the DEM.** A decoder is the triple (**H**, **A**, **p**) with one prior per fault mechanism. Hardware quality enters through **p** (Maurer et al., arXiv:2510.21600 [V-prior]). The HAL-to-decoder channel is a noise-model deliverable.
- **LightStim** (arXiv:2604.21472 [V-prior]) automates DEM construction by rowspace test. It is plausibly applicable to ACID output; whether it accepts an external circuit and handles random gauge outcomes is unchecked.
- **Bare logicals are forced.** *L_Z* ∈ ker(*G_X*); construction by two nullspaces, one intersection and one complement; dim *L_Z* = *k*. There is no 2^|*G*| enumeration; MaxSAT picks the dressing itself.
- **PropHunt's MaxSAT indexes the move space.** Without the minimum-weight witness the move space is the whole circuit, not *O*(*w*) candidates near the fault.
- **PropHunt's decoding graph handles hyperedges; its mutation graph is ancilla-indexed and does not port.**
- **Radial codes' single-shot capacity rests on confinement** (Quintavalle–Vasmer–Roffe–Campbell), not on metachecks from linear dependencies. Addendum 10.18 and thread T1.8 are restated accordingly (§14).

### 10.7 Hardware facts

- **Tier is not a failure mechanism; transitions are.**
- **Series reliability.** *p_fail*(*e*) = 1 − (1 − *p_bump*)^*B*(*e*) (1 − *p_TSV*)^*T*(*e*) (1 − *p_seg*)^*L*(*e*). Monotone by construction; only the coefficients need data, and they do not exist.
- ***C_hw* reference values.** The optimistic reference is 5 tiers, long-range couplers 10× the shortest, 4 bump transitions and 3 TSVs per coupler. The default cap of 10 transitions is a sweep bound, not a calibration point.
- **Flip-chip evidence.** One or two transitions do not measurably degrade coherence or two-qubit gate fidelity. Nothing is measured at 5–10.
- **Frequency collisions.** For fixed-frequency transmons, qubit death is dominated by frequency collisions defined on coupled pairs and next-nearest neighbours (Hertzberg et al. 2021; Morvan et al. 2022 [V-prior, not re-checked]). This is a graph-degree property, not a placement one.

### 10.8 Code-family facts for Phase 2

- **Ancilla-free connectivity in the literature.** Exists only for surface, colour and BB. For other 2BGA codes, only the minimum-degree morphing graph.
- **BB connectivity pair.** BB is the only family with both a minimum-degree morphing graph and a higher-degree graph. Shaw–Terhal found higher-connectivity BB morphing circuits but did not publish them.
- **Smallest 2BGA test codes.** From Shaw & Terhal (arXiv:2604.09797): [[30,8,5]] (mid-cycle *N* = 60) and [[42,6,6]] (*N* = 84). Chao's smallest instantiated code is [[224,8,16]], and only his Construction I is genuinely 2BGA. [[32,2,4]] unrotated toric is a regression floor only.
- **Fiber bundle and lifted product are one construction.** 2BGA is the 1×1 base-matrix case, with a public enumeration.
- **Tile-code connectivity.** HAL Appendix F gives it for the ancilla-based graph; supports sit in a 3×3 window, with edges ~4× shorter than BB [V-prior]. The ancilla-free version still needs deriving.
- **Radial codes.** No translation invariance and no nearest-neighbour qubit tier. The exposure metric may saturate immediately on them.

## 11 · Live ideas not yet in v16

**11.1 Connectivity construction: remaining detail.** v16 §6.1 carries the Hamiltonian-cycle reduction.

Still live:

- **Complexity landscape [V-ref].** Minimum-edge support is NP-hard; planar support is NP-complete (Johnson & Pollak); path, cycle, tree and cactus supports are polynomial; planar tree support with bounded degree is polynomial (Buchin et al.); 2-outerplanar is NP-hard.
- **Closest prior art for the 2-edge-connected version.** Cactus supports, and Brandes, Cornelsen, Pampel & Sallaberry on biconnected components. Survivable network design is not the same problem, since it requires disjoint paths in the whole subgraph, not inside each induced subgraph.
- **First experiment.** Enumerate the 3,600 orbit pairs for BB and check whether ACID's hexagonal graph is Pareto-optimal. This replaces the 2-edge-connected ILP thread.

**11.2 The damage catalogue.** For translation-invariant code and connectivity, compile single-defect and pair patterns once and score draws by lookup. **Its additivity assumption is the single biggest methodological bet for Phase 2 Monte Carlo.** Test by predicting fragmentation, anticommutation rank and gauge count for ~200 random multi-defect draws. A failure is itself a result: defect interactions are long-range.

**11.3 Fault "impact" and the prefactor.** Misprediction probability is set by the least likely of two ambiguous errors, so the MaxSAT witness weight (or −log *p* under ε-weighting) is already a non-binary score. The residual worry, several nearly-as-bad errors outweighing one, is the prefactor. Handle it by counting witnesses at each weight. Any surviving "impact" metric must fix its representative-dependence.

**11.4 Prune extra measurement.** The only move ACID cannot express; it attacks ACID's objective 2. Removing one redundant weight-6 measurement removes 5 CNOTs, which is 75 fault mechanisms, against one detector. Check single-shot survival (T1.8 restated) first, since pruning may spend single-shot capacity.

**11.5 PropHunt integration details (Phase 2).**

- **Defect-biased subgraph seeding.** PropHunt seeds uniformly; gauge-induced ambiguity is concentrated near defects. A small change with probably a large efficiency gain, claimable independently.
- **Decision-form accept test.** Replace optimisation with satisfiability: is there *e* with *H′e* = 0, *L′e* ≠ 0, |*e*| ≤ *w_prev* + δ? UNSAT accepts.
- **Batching.** Batch candidate moves across witnesses and apply a maximal subset that is both ambiguity-compatible and ACID-feasible, not a serial repair loop.
- **Rotating the gauge decomposition round to round.** The defensible claim is fewer recurring low-weight fault paths, not averaging. It breaks the fixed-*G* assumption behind bare logicals.
- **Keep the sets distinct.** Sampled region, witness support, occurrence repair seed and changed-event validation closure are four different sets.

**11.6 Dropout as an exclusion policy.** From Drouet et al. and your own framing.

- The per-chip decision is a threshold τ on a calibrated error map, revisited each calibration cycle.
- This unifies the noise mapper's two outputs: failure is the tail of the fidelity distribution.
- It makes ACID runtime and anchoring matter more, since recompiles happen per calibration, not per fab.
- It matches the "bank of circuits switched on the calibration timescale" idea.

Spec T12 is the first experiment. Anker & Debroy say the same thing from the cost side: compile cost is paid per calibration cycle or on new defects [V].

**11.7 The shorten-versus-spread tension.** Fidelity wants critical couplers short, which packs them. Crosstalk wants co-active couplers apart or on different tiers. HAL's length objective and exposure can conflict. This is the most interesting new trade-off, and it requires both the co-activity map and the fidelity map.

**11.8 Seam placement as a layout lever.** On a torus-graph code, where the planar embedding cuts the torus decides which edges become long. Under orbit-constant heat maps, choosing which edge class crosses the seam is a discrete lever with few options.

**11.9 The tier sweep and the layer-assignment continuum.** Run HAL with max transitions per coupler ∈ {0, 2, 5, 10} on one code; record tiers, mean length, mean transitions and *C_hw*. This yields Δℓ, the length saved per transition allowed: the exchange rate the reliability and fidelity models need. HAL and the biplanar layout are two settings of one scalar β/α in a standard via-cost-versus-wirelength objective, which is directly runnable with no code change.

**11.10 Leakage exposure as an ACID objective term.** In the ancilla-free paradigm, contraction roots are measured and reset, and measure/reset is the dominant leakage injection site. ACID's objective 2 pushes toward more measurement. Leakage is not Pauli, so the honest framing is an ACID objective term, not a decoder prior. Stim cannot simulate leakage.

**11.11 Identifiability of a dropout-repaired DEM.** Gauge operators add rowspace degeneracy, which creates non-identifiable directions. A repaired subsystem code should be measurably harder to calibrate from syndrome data than its parent. Small, provable, apparently unoccupied.

**11.12 Decoder provisioning under per-chip DEMs.** Now owned by the decoding subteam (`qca_decoding_context_v1.md`).

- The theory project's stake: v16 §9's decoder paragraph.
- The two cheap scripts: union-DEM size against median DEM over a dropout distribution; automorphism-orbit count of dropout patterns under Aut(BB144), which may shrink the covering problem by about two orders of magnitude.
- The mismatch-penalty framing: compound channels and mismatched decoding, at a yield quantile, governed by a minimum rather than an average.

**11.13 Gauge orbit as a post-fabrication resource.** The gauge orbit of twist assignments is universally discarded as a search-space quotient, but has never been used as a post-fabrication defect-response resource. An open novelty claim; sweep before relying on it.

**11.14 Collaboration with Di Bella (and Strikis).** Contact Di Bella, the sole author, and mention Strikis. Do so only after two things:

- spec T11c, reproducing their BB72 reference point from the released code;
- spec T11a, checking whether pair faults stay weight 2 in ACID circuits.

Propose one bounded joint piece: exposure under multi-tier HAL routing with ACID schedules. Keep dropouts and HAL modifications as ours. Dan Browne is a common node (ACID supervision; co-author with Strikis) if an introduction is preferred. Strikis's March affiliation includes Quantum Motion, which may complicate terms.

**11.15 Role-dependent schedules (algorithms-team overlap).** Active logical qubits run *L* = 3 schedules and idling ones *L* = 4. This builds on the same Anker–Debroy machinery as the objective port. Anker & Debroy's own data: 3-round is worse for memory at fixed rounds, but may win when time-like limited. Build the port once and share it. Conceptually unoccupied for qLDPC as far as swept; check arXiv:2603.05481 and 2604.09797 before claiming.

**11.16 Parked items with a stated reason.**

- **Qubit criticality to break chord symmetry.** First check the orbit count.
- **Frequency allocation.** The collision graph is fixed by the connectivity; criticality can bias assignment. A discussion paragraph, not a HAL change.
- **Post-fabrication triage.**
- **TSV redundancy.** The first concrete "design element that improves flexibility cheaply."
- **Logical operations under defects.** Surgery-based rather than symmetry-based logic should degrade gracefully.
- **ISWAP/CXSWAP as a collapse primitive.** Take it from the morphing level, not inside ACID.
- **Qubits on higher tiers.** A fabrication claim, not a routing claim.
- **EDA at algorithm scale.**

## 12 · The EDA connection

HAL's placement is a primitive interconnect model, blind to three of *C_hw*'s four terms. That is the placement–routing mismatch the EDA literature identified twenty years ago.

- **The field's answer is routability-driven placement** with cheap congestion estimation: RUDY (Spindler & Johannes, DATE 2007); SimPLR-style lookahead.
- **The 3D flow** is global placement with via whitespace, then via insertion, then layer-by-layer detailed placement.
- **Layer assignment** is a named subproblem with a thirty-year literature: congestion-constrained via minimisation, negotiation-based assignment, FastRoute 4.0.
- **Net ordering and negotiated congestion** are the free levers. PathFinder (McMurchie & Ebeling, FPGA '95) blends criticality with escalating congestion penalties; substitute defect criticality for timing criticality [?, verify the cost function form before citing].

**Crosstalk has a classical analogue in signal-integrity analysis.** Aggressor and victim wires couple only when their switching windows overlap. That is the co-activity condition of v16 §5.6 [general knowledge, not verified this session]. On the quantum side, Murali et al. (ASPLOS 2020) serialise high-crosstalk simultaneous gates, which is the dual problem of fixing the hardware and changing the schedule [V-prior, not re-checked]. Zhou, Ji & Ding (QCE 2025, arXiv:2503.04642) study surface codes under crosstalk noise [V-ref, via Di Bella].

**The honest framing for a writeup:** HAL's placement uses a primitive interconnect model, which the EDA literature identified as the dominant source of placement–routing mismatch two decades ago. We port routability-driven placement to the qLDPC setting and add defect-criticality weighting, which has no classical analogue.

## 13 · Framings that took several attempts to get right

- **Be the defect layer, not the code layer.** Every adjacent tool is a generator; none touches defects.
- **The moat is not commercial horizon, and it is thinner than it looked** (§15).
- **The pipeline is sequential, not a loop,** because layout does not enter ACID's decision process. Corollary: identical circuits on every chip, so comparisons between chips can be paired.
- **Redundancy in the hardware graph and schedule is the resource;** redundancy in the check set is a liability.
- **Fragility belongs to the embedding, not the code.** The tile-code fragility prediction failed on this.
- **Unifying thesis: clean-code optimisation removes defect-tolerance redundancy.** Instances:
  - morphing-derived connectivity is minimum-edge by construction;
  - maximally packed coset-code schedules have no slack;
  - Louvre's coupler reuse;
  - ACID's minimum-weight gauge selection.

  HAL does *not* minimise edges; its metric excludes edge count, so it is not an instance.
- **Symmetry is favoured by manufacturability, not by benefit.**
- **Design-time beats per-chip,** optimised against a distribution at a yield quantile.
- **Dropout is a decision, not a fact** (§11.6).
- **Relabel, don't re-solve; replicate seeds, not couplers.**
- **MVP first: HAL on BB.** HAL has the insertion points and lacks only information; everything else needs new infrastructure.
- **Intra-support edge, not data–data.**
- **Fiber bundle and lifted product are one construction in two frames.**
- **Global analysis, local moves.** *This is the sentence that catches failure mode 4 when it recurs.*
- **Verify the outcome, not the precondition.**
- **Feasibility locality is not analysis locality.**
- **Ambiguity is a quantity, not a defect.** It cannot be removed, only made more expensive.
- **The prefactor is the target.** Minimum-weight path counts, not distance alone (Anker & Debroy App. C; Strikis et al.'s *N_fail*).

---

# Part III: Open threads and conversation protocol

## 14 · Open threads, ranked by how much they gate

**Tier 0: the Phase 1 MVP tests.**

These are specified in full in `qca_mvp_test_spec_v1.md`. **Not to be implemented or run without explicit user approval.** In dependency order:

| Test | What it settles |
|---|---|
| T1 | Joint automorphism group and orbit census for BB codes on both connectivities. Also tests the hypothesis that 288's arc-transitivity needs the square torus. |
| T2 | Relabelling equivalence. |
| T3 | Solver-variance decomposition on BB and surface codes. Decides whether heat maps are interpretable from single solves. |
| T3b | Anchoring. Decides whether the objective port leaves the critical path. |
| T4 | ACID runtime profile. Replaces the unprofiled "hours per heat map." |
| T5 | The dropout-free anomaly on BB-144 hexagonal. |
| T6 | Gauge-group and excision checks. |
| T7 | Asymmetric testbed. Priority: planar BB-derived codes, tile codes, radial codes, pruned BB as fallback. |
| T8 | Noise mapper. |
| T9 | Power budget. |
| T10 | Objective port: acceptance test, term audit, Fig. 13 analogue. |
| T11 | Crosstalk: propagation weight in ACID circuits, co-activity maps, Di Bella reproduction. |
| T12 | Exclusion threshold. |

**Tier 1: gates Phase 2; cheap.**

- **T1.1 · Does PropHunt buy anything on qLDPC codes at superconducting idle rates?** Regenerate the idle-sensitivity data from the public artifact (github.com/jviszlai/PropHunt). Read the LP and RQT curves at *t_gate*/*T* ≈ 3×10⁻⁴. Strikis et al.'s depth-matched comparison now weighs against it [V]; the ancilla-free depth penalty is harsher still. Go/no-go on the largest Phase 2 engineering item.
- **T1.2 · Does an ACID schedule object fix absolute timesteps within its layer, or only the tree-induced partial order?** One source scan. Decides whether cross-occurrence retiming exists.
- **T1.3 · Typical subgraph size at PropHunt expansion halt.** Ten minutes of instrumentation. Settles the open disagreement (user: dozens of columns; agent: thousands) and the accept-test cost model.
- **T1.4 · Are the morphing catalogue's connectivity graphs published machine-readably?** Largely superseded by cyclic-ordering construction; still relevant for comparison baselines.
- **T1.5 · Does the damage catalogue's additivity hold?** §11.2.
- **T1.7 · Does LightStim produce a correct DEM for one ACID circuit under dropout?** One day. Note that ACID already emits detectors, so the narrower live question is completeness against the full space of deterministic parities.
- **T1.8 · Does single-shot capacity survive ACID repair?** Restated in confinement-profile terms (§10.6), not metacheck rank. Gates §11.4.

**Tier 2: real questions, not blocking.**

- **T2.7** |*D_o*| per occurrence on degree-5 versus hexagonal BB. Degree-5 is root × timing only.
- **T2.8** Do witness-containing occurrences have alternatives that change the relevant *H*, *L* columns?
- **T2.9** Do surviving spanning trees differ meaningfully in ε-cost?
- **T2.10** Does a different (*V*, *W*) change which edges carry CNOTs? Decides §9.6.
- **T2.11** ACID detector/observable emission. *Answered in part:* ACID emits both; full fault-to-occurrence provenance is still missing.
- **T2.12** Does ACID's scheduling graph assume quasi-stabiliser schedules are compatible with any untouched-stabiliser schedule?
- **T2.13** Can ACID's compatibility criterion 2 be relaxed via ISG verification?
- **T2.14** Orbit counts. *Absorbed into spec T1.*
- **T2.15** Fraction of realised length that is detour versus endpoint distance.
- **T2.17** Does criticality ranking change after PropHunt? One PropHunt stage or two.
- **T2.18** Is there a length-dependent open-circuit failure mode for superconducting couplers at 0.1–1 cm?
- **T2.19** Do fragmentation and hardware cost correlate through the same edges?
- **T2.20** 2-edge-connected ILP. *Replaced by cyclic-ordering enumeration* (§11.1).
- **T2.21** How much does the DEM change between dropout patterns and between family members? *Owned by decoding.*
- **T2.22** Priors-only mismatch experiment. *Owned by decoding.*
- **sQetch (v4 T1.6).** *Resolved:* matrices only, no subsystem adapter, CUDA dependency.

## 15 · External asks and competitive position

**HAL authors.**

- **Affiliations:** MIT RLE, EECS and Physics, and ETH Zürich; not Lincoln Laboratory.
- **Correspondence:** Jeffrey A. Grover (jagrover@mit.edu). W. D. Oliver is on the author list.
- **Source:** github.com/EQuS/HAL (pinned ad8ac0a in v4).
- **Implementation questions:** Mathews has moved to Google, so ask the Pahls or Grover.

Genuine asks:

- per-TSV and per-bump-bond yield;
- whether gate fidelity degrades at 5–10 transitions;
- whether there is a length-dependent open-circuit failure mode.

These are the two transition coefficients of v16 §5.5 and spec T8. The pitch fits their own Discussion: given the coefficients experiment supplies, this is the HAL to run. The MVP is not gated on a reply. Email them before the ACID author.

**ACID author** (Wolanski; Riverlane and UCL, supervised by Browne and Campbell). Explicitly flags better scheduler constraints as future work. Contact once there is a concrete result.

**Di Bella / Strikis.** See §11.14.

**Competitive assessment.**

| Group or work | What it occupies |
|---|---|
| PropHunt group | Cites the defect-aware SM literature and names composition with it as future work. A future-work sentence is evidence of non-occupation, not of feasibility; use it as a moat citation only. |
| Di Bella / Strikis (Cambridge/Oxford) | One step from multilayer crosstalk under routing geometry. The crosstalk channel is contested; our distinct slice is dropouts, ACID schedules and multi-tier routing together. |
| Strikis, Browne & Beverland | Two-block partition for ancilla-based LP circuits; cheap circuit-distance bounds. |
| AlphaSyndrome (arXiv:2601.12509) | Competes with any layer-level scheduling thread. |
| arXiv:2608.19917 | Closes symmetry-exploiting circuit design for LP codes. |
| Wei et al. (arXiv:2604.05874) | Occupies weight-one gauge generalisation. The surviving slice is composition with the ancilla-free paradigm, which ACID's connected-component construction may already provide. |
| Báscones et al. (arXiv:2605.01035) | Closest to a parallel multi-code FPGA decoder (decoding subteam). |
| Tile-code connectivity | Contested ground: two nearest-neighbour compilation papers in mid-2026. |
| Barbell codes | Hardware-aware planar entrant with its own layout and syndrome cycle. |

## 16 · Conversation-start protocol

1. Read this document, then v16. Read the other working-set documents (§0) when the conversation touches their scope.
2. Re-sweep the literature on anything v16 states as a gap, including the specific re-checks in §3.
3. Spot-check any [V-prior] before it appears in output; do not cite [?] entries.
4. If the user uploads paper notes, read them closely. If the underlying paper is available, read it in full rather than working from the notes (§4). Confirm the paper is the one the notes describe.
5. **If a tool has public source, read the source.** ACID (github.com/riverlane/ACID), HAL (github.com/EQuS/HAL), PropHunt (github.com/jviszlai/PropHunt), Stim, sQetch (github.com/a7b/yarn), Di Bella (github.com/angelodibella/works).
6. Before designing an algorithm, ask whether the problem has a name, and whether the name's assumptions hold (§8.5).
7. Assume the current bottleneck is Phase 1 (Tier 0 in §14) unless told otherwise. Do not start Phase 2 or Phase 3 work before the Phase 1 result exists.
8. **Do not implement or run any MVP test without explicit user approval naming the test.** Specification, planning and cost estimates are fine.
9. Do not re-propose anything in §9 without new evidence; each was rejected for a stated reason. The one open item is §9.6, to be resolved by T2.10, not by assertion.
10. Prose for project documents: no em-dashes in new text; clarity over verbosity; British spelling as in v16.

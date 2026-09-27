# QCA Decoding Subteam — Working Context v1

**Date: 26 September 2026. Author: session handoff. Scope: Fall 2026 decoding gear-up project.**

---

## 0 · Read this first — scope, and why you still need the theory documents

**This document is for a separate project from the main QCA theory project.** The theory project is defined by *Expected Logical Efficiency of Fabricated qLDPC Chips* (v15) and *Working Context v4* plus the *v4.1 addendum*. This document covers a smaller, self-contained Fall 2026 project for the **decoding subteam**, intended as skill-building ahead of the cross-subteam project in spring. Novelty is not the goal. Producing something the hardware subteam can use in spring is.

**But do not treat the two as independent.** The single most common failure in this project's history has been treating one subteam's tool output as a black box. Several live theory threads have direct decoding consequences, and at least two decoding findings feed back into theory. Section 6 lists the live channels. Before proposing anything, check whether the thing you are proposing is already a theory thread wearing different clothes.

Working mode is unchanged from the theory project: adversarial red-teaming, no novelty claims without a sweep in multiple vocabularies, every citation marked, no em-dashes in prose.

Verification marks: **[V]** confirmed against paper body this session · **[V-ref]** confirmed via reference list or secondary source · **[V-prior]** verified in an earlier session, spot-check before citing · **[?]** surfaced but unverified, do not cite.

---

## 1 · The project and its constraints

Deliver a decoding project for Fall 2026 with:

- small scope, completable in one semester
- real value to the hardware subteam, and ideally to theory
- zero hard dependency on theory's HAL+ACID+LUCI pipeline, which is not expected before year end
- a presentable result, not pure infrastructure

Standing constraints on the surrounding subteams, as of this writing:

- **Hardware** spends the semester rebuilding a Riverlane-style real-time decoder on FPGA as a training artifact. That is a *clustering* decoder, surface-code oriented. Maurya et al. (arXiv:2511.21660) find FPGA-adapted clustering and OSD both slower and less accurate than message passing for qLDPC, and conclude message passing is the viable route. [V-prior] Plan the transition to a memory-based or parallel message-passing decoder explicitly, in September, not in January.
- **Theory** is building the HAL+ACID+LUCI pipeline through year end.

---

## 2 · The three candidate ideas and where they landed

### Idea 1 — Integrate LightStim DEM construction into ACID→Stim

**Verdict: the premise is false. Closed as originally stated.**

ACID already assigns detectors automatically. §3.4 of arXiv:2512.01943 is an algorithm for it, in the Pauli-flow language. [V] There is no hand-annotation burden to remove. This closes T2.11 and answers T1.7 in the direction of "no work needed" rather than "one day of work."

LightStim is additionally the wrong shape for the job: it is a **circuit constructor, not a circuit consumer**. The user specifies a protocol plus configuration, LightStim compiles to its own Spatial-Temporal IR, and the Pauli Tracker builds detectors in lock-step as the IR is constructed. [V] There is no "feed it an external Stim circuit" entry point.

**What survives, reframed:** *is ACID's detector set complete, and does the answer depend on the dropout pattern?*

LightStim reports 12 additional detectors on both BB [[72,12,6]] (432→444) and [[144,12,12]] (1728→1740) versus reference implementations, arising from **linear dependencies among X-stabilisers** that manual annotation misses. [V, Table 2] ACID's §3.4 algorithm assigns detectors per stabiliser between its own consecutive contractions. Nothing in it searches for detectors arising from dependencies *among* stabilisers. So the concrete hypothesis is that ACID's detector set is incomplete by exactly the redundancy detectors, in a dropout-dependent way.

This is measurable in about a week with no new tooling and no LightStim: build the matrix of back-propagated measured Paulis from the emitted circuit, compute the dimension of the space of measurement-record subsets with deterministic parity by Gaussian elimination over F₂, compare against the number of emitted `DETECTOR` lines. Stim rejects non-deterministic detectors at sample time, so there is a free correctness oracle.

**Damper found in the same data:** those 12 extra detectors produced LER "consistent within 2×, attributable to decoder configuration variance." [V] Redundant detectors are strongly correlated with existing ones and add little marginal information at circuit level. So do not sell this on LER. Sell it on **single-shot capacity**, which is what redundancy actually buys and what ACID's repair destroys. Same object as theory thread T1.8.

**Unverified mechanism concern, worth checking in LightStim source:** Algorithm 1 emits a detector only in Case B (measured Pauli commutes with all tableau rows). Gauge operators in a subsystem code are precisely the anticommuting ones, so every gauge measurement dispatches to Case A, which emits no detector. Whether the WriteBack step recovers the product-stabiliser detector is not stated in the paper. If it does not, LightStim produces structurally incomplete DEMs for any subsystem code. [Inference, not verified against source.]

### Idea 2 — Realistic bitstreams beyond i.i.d.

**Verdict: closed. The premise rests on a category error about what a DEM is.**

The decisive argument: **a DEM is an independent-fault model by construction.** Stim compiles a circuit-level noise model into a set of independent fault mechanisms, each with its own probability, and all circuit-level correlation is absorbed into which detectors each mechanism flips. Sampling each column independently and multiplying by H_DEM is not an approximation to circuit-level noise, it *is* circuit-level noise as far as the formalism can express it. The independence lives at the level of fault mechanisms, not qubits.

Therefore "more realistic than i.i.d." decomposes into exactly two options with nothing in between:

1. **Change p.** Per-edge, calibration-derived, HAL-derived, biased. Already standard practice, and the decoder receives the same update for free.
2. **Leave the DEM formalism.** Leakage, drift, non-Markovian effects, measurement-induced state transitions. These cannot be written as a DEM, so they cannot be handed to the decoder either. Any such experiment is a *mismatch* experiment, not a bitstream project.

Supporting occupancy findings:

- **QUITS** (arXiv:2504.02673, *Quantum* 9, 1931, Dec 2025) is a published modular circuit-level qLDPC simulator supporting free combination of code constructions, SE circuits, decoders and noise models, covering HGP, LP and balanced product families. [V abstract]
- **García-Herrero et al.** (*Quantum* 10, 2071 (2026), arXiv:2504.01164) generate syndromes on-chip in FPGA fabric from H or the DEM graph. Code-agnostic sparse matrix multiply. [V-prior, not re-verified this session]
- **Calibration-parameterised noise is already standard on real hardware.** The LUCI-on-IBM paper decodes with correlated MWPM "weighting edges from calibration data for single-, two-qubit gates and measurement," and runs Stim simulations parameterised by experimental calibration data. [V, arXiv:2607.01887]

**The real constraint on calibration-realistic qLDPC noise is data, not tooling.** Stim natively accepts per-instruction rates. Nobody has published per-coupler calibration for a multilayer superconducting qLDPC device because no such device exists at scale. IBM's Loon and Kookaburra would be the first. This means HAL-derived per-edge rates are a **synthetic substitute** and the resulting claims are unfalsifiable until someone fabricates the chip. Say so in any writeup: "here is what the noise model looks like if HAL's geometry model is right," not "here is the realistic noise model."

**Two genuine gaps that surfaced, both belonging to theory rather than decoding:**

- **ACID's own noise model has noiseless boundaries.** Appendix C: initialisation by *noiseless* Stim Pauli-product measurements of all quasi-stabilisers, observables initialised with *noiseless* controlled-logical operations on reference qubits, and the experiment closed with *noiseless* MPP. Noisy portion is uniform depolarising p=0.001 with every coupler identical. [V] Anker and Debroy had to replace exactly these with noisy operations for a fair comparison. [V]
- **Measurement-induced crosstalk is an explicitly flagged open gap** and is mechanistically ACID's problem. See §3.5.

### Idea 3 — Hardware-overhead FOM for dropout patterns

**Verdict: the only one of the three with a live research question. Substantially reshaped. See §5.**

---

## 3 · Verified findings worth not relearning

### 3.1 ACID (arXiv:2512.01943, Wolanski / Riverlane), read in full

- §3.4 assigns detectors automatically via Pauli flows, with a separate "completion" construction for product stabilisers: detectors between consecutive completions, where a completion is a set of layers over which all constituent quasi-stabilisers are measured. [V]
- It also adds extra detectors "where a quasi-stabiliser is measured more than twice over two completions, and there is no anticommuting quasi-stabiliser that randomises its value in between two of them." [V] Gauge measurement *ordering* is therefore already a scheduling degree of freedom that changes the detector count.
- **Known caveat, in the author's words:** "ACID sometimes produces detector error models that do not admit ready approximation as decoding graphs... I believe that this is a result of **overlapping redundant measurements of quasi-stabilisers**, and can be fixed in future work by imposing additional constraints on the scheduling graph." [V] This is the cram-in mechanism and it is load-bearing for §5.
- Objective structure: minimise L (layers) first, then **maximise the minimum number of times any stabiliser is measured**, then tie-break by maximising the total number of quasi-stabiliser contractions. The author calls these "crude heuristics." [V]
- Gauge construction: quasi-stabilisers from connected components of the damaged local graph; bidiagonalise the anticommutation matrix A via VAW^T = [[I_g,0],[0,0]]; first g rows of V and W are the exclusively anticommuting gauge operators, remaining rows are product stabilisers. Span of the product stabilisers is maximal and unique. [V]
- **Component counts, needed for the fixed-rate correction in §4.4:** BB144 hexagonal 144 qubits / 432 couplers; BB144 degree-5 144 qubits / 360 couplers; d=11 square-grid surface 261 / 480; d=11 hex-grid surface 241 / 340. [V, Appendix D headings]
- Degree-5 BB local graphs are trees, so any coupler loss produces anticommuting quasi-stabilisers. Degree-6 hexagonal local graphs are cycles, so isolated coupler loss is absorbed with all original stabilisers still measurable. **This is ACID's own stated motivation for the hexagonal connectivity.** [V] You cannot claim it.

### 3.2 LightStim (arXiv:2604.21472v2, Fang et al.), read through §6.2

- Circuit constructor with a Spatial-Temporal IR. Five atomic operations: Initialize, SyndromeExtraction(r), Activate/DeactivateCoupler, UnitaryBlock, DataReadout. SE circuits are per-code-family modules. [V]
- Case A (anticommutes) emits no detector and replaces a pivot row. Case B (commutes) emits a detector via RREF_Solve over GF(2). WriteBack re-expresses the tableau in the freshly measured Paulis to keep detectors temporally local. [V]
- Repeat-block optimisation assumes rounds 2..r reach a fixed point and back-propagate identical Pauli strings. [V] Whether this holds under ACID's multi-layer schedules with random gauge outcomes is unchecked.
- Validation: exact detector-count match against Stim for rotated surface d=3,5,7; +12 detectors on both BB codes. [V]

### 3.3 LUCI on IBM hardware (arXiv:2607.01887, Kim, Gicev, Sevior, Usman), read in full

The closest thing in the literature to "what a realistic dropout-adapted syndrome stream looks like on hardware."

- Reset-free LUCI on ibm_miami, distance-3 and asymmetric 3×5 patches, 10⁴ shots. Suppression ratios Λ⁵ᐟ³: standard 1.58(13) X / 2.44(7) Z; LUCI 1.75(10) X / 1.93(12) Z. [V]
- LUCI beats the standard surface code in the X basis by routing around a ~15% error-rate coupler the standard patch is forced to use. **Dynamic scheduling wins on inhomogeneity, not only on defects.** [V]
- Detection event probabilities, useful validation targets: bulk weight-4 X 0.149, Z 0.124; boundary weight-1 X 0.057, Z 0.055 at d=3. [V]
- Cycle time 3.5 µs standard versus 7 µs LUCI. [V]
- **The flagged gap:** their LUCI variant, which selects lower-measurement-error qubits and measures some neighbours simultaneously, failed to show expected suppression. They attribute this to measurement-induced crosstalk from adjacent measured qubits, say "we leave a detailed characterization of this noise for future work," and state that "modeling this effect will be essential for effectively utilizing the LUCI architecture to address defective processor components." [V]

### 3.4 FPGA decoder architecture (arXiv:2510.21600, Maurer et al., IBM)

- Fully parallel Relay-BP: dedicated compute unit per variable and check node, **"message exchange embedded in FPGA wiring"**, one BP iteration in two clock cycles. [V]
- §5: "the message data is stored in wiring rather than centralized memories, very little block RAM is used," and the int4 gross-code decoder uses over half the LUTs of an XCVU19P, the second-largest FPGA AMD sells. Timing closure at a 12 ns clock, 24 ns per iteration. [V]
- §8: "Strategies to improve the resource footprint of the decoder will be discussed in a future revision." Their own footprint is an open problem. [V]

**Consequence, and the premise idea 3 needs:** in the leading real-time qLDPC decoder architecture, the DEM graph *is* the routing. A structural change to the DEM is a re-synthesis, not a register write.

Scaling anchors from adjacent work, and the null models to beat:

- Vegapunk (MICRO 2025): "LUT usage scales linearly with the check matrix column size," fitted, with 100% utilisation projected at column size 1.26×10⁴. [V-ref]
- Helios distributed union-find: "while the numbers of vertices and edges grow by O(d³), resource usage grows faster." [V] Superlinear growth versus graph size is exactly what a structural metric would have to explain.
- Báscones et al. (arXiv:2605.01035, GARI multi-core): full core on a VU29P at 7.5% LUTs; eight-FPGA configuration at 9% LUTs, 4% registers, 90% BRAM. [V]

### 3.5 Measurement-induced crosstalk on transmons — the physics

**The dominant mechanism is spectral overlap on a shared readout feedline, not physical proximity.**

- Heinsoo et al. (arXiv:1801.07904) measure Γ̄ᵢⱼ, the average dephasing rate of Qᵢ from measuring Qⱼ, and explain the pattern by *frequency*: Q2 is dephased most strongly by tones for R5 and R6 "as these are the readout resonators closest in frequency to R2." [V]
- The reentrant-cavity-filter paper (arXiv:2412.14853, *PRApplied* 23, 054089 (2025)) names two causes: stray coupling between resonators and qubits in different unit cells (spatial, second-order), and finite spectral overlap driving a nearby-in-frequency resonator (spectral, dominant). [V]
- Mitigation targets the spectral channel: DRAG-style pulse shaping to notch neighbouring resonator frequencies (arXiv:2509.05437), tunable broadband Purcell filters (arXiv:2509.11822). [V]

**The quantitative anchor:** Krinner et al. (Zurich distance-3 surface code, arXiv:2112.03708) characterise Γᵢⱼ and the coherent phase rotation Δφᵢⱼ for every auxiliary qubit acting on every other qubit, giving a full matrix, and convert to P_φ^ij = [1 − exp(−Γᵢⱼτ_RO)]/2. **Average phase-flip probability 0.09%, average coherent phase rotation 0.6°.** [V] Same order as the p=0.001 depolarising rate ACID assumes per gate.

**Nobody has published a clean LER-versus-measurement-crosstalk curve.** The surface-code crosstalk simulation literature (arXiv:2503.04642, arXiv:2002.08918) models gate-based and always-on ZZ crosstalk, not measurement crosstalk. Proctor et al. (arXiv:2410.16706) give the measurement protocol, quantum instrument randomised benchmarking, applied to a 27-qubit IBM device. [V] That is an experimentalist's tool.

### 3.6 The classical EDA analogues

- **Rent's rule and system-level interconnect prediction.** The Rent exponent p, extracted by recursive bisection, predicts wirelength distribution and routability *a priori*, before placement. p* ranges ~0.5 for highly regular circuits to ~0.75 for random logic. A specific line uses it as an interconnect-complexity metric driving FPGA placement and routing-resource allocation. [V-ref: SLIP 2001 "Interconnect complexity-aware FPGA placement using Rent's rule"; SLIP 2002 "FPGA interconnect planning"; Wotan routability estimation]
- **The classical community's own verdict, which you inherit:** "Despite its long history, from Rent's rule on, interconnect prediction is little used in industry. The most common implementation of interconnect prediction, the 'wireload model' used in synthesis, is almost universally scorned by the designers that use it." [V-ref] Report R², not a formula.
- **Cost of a *change* has two classical names:** Engineering Change Order / incremental design (diff netlists, incrementally place and route the delta, measure wirelength and timing degradation; FPGA vendors ship this as incremental mode with lock-existing-layout), and **reconfiguration-data minimisation** (minimise partial bitstream size between successive configurations by maximising structural similarity, including dummy-node insertion). [V-ref]
- **Netlist-intrinsic metrics computable without synthesis:** Rent exponent, bisection width / min-cut, Donath-style wirelength bounds from Rent parameters, fanout distribution, path-counting routability predictors.
- **Module relocation and partial reconfiguration** (GoAhead and the PR survey literature) is the classical analogue of remapping an implemented decoder to a different region. Real, vendor-specific, fiddly. [V-ref]

### 3.7 Tooling availability

- **No open-source qLDPC FPGA decoder generator exists.** IBM's is internal.
- Nearest, both inadequate: `PaulBryden/hdl_ldpc_decoder` (nMigen, takes a parity-check matrix as a constructor parameter, emits Verilog, toy scale, single-bit correction) [V]; MATLAB HDL Coder's LDPC Decoder block (layered normalised min-sum with HDL codegen, restricted to QC-LDPC of circulant weight 1, which BB DEMs are not) [V].
- Software reference decoders are a solved problem: `trmue/relay` (Relay-BP, what IBM cross-validated against), `qLDPCOrg/qLDPC` (BP-OSD, BP-LSD, belief-find, Relay-BP, PyMatching, `SinterDecoder`, `SlidingWindowDecoder`, `DetectorErrorModelArrays`), NVIDIA CUDA-Q QEC batched GPU Relay-BP, LightStim's unified backend. [V-prior]

---

## 4 · Red-team register

Claims tested this session, with disposition. Do not re-propose without new evidence.

**4.1 "ACID circuits need hand annotation." REJECTED.** ACID §3.4. [V]

**4.2 "LightStim can consume an ACID circuit." REJECTED.** It is a constructor with its own IR. [V]

**4.3 "Bitstream generation across qLDPC families is a project." REJECTED.** QUITS exists, on-chip generation is a code-agnostic sparse matmul, and the DEM is definitionally an independent-fault model so there is no middle ground between "change p" and "leave the formalism."

**4.4 "Higher-degree connectivity needs less decoder hardware overhead." NOT REJECTED, BUT NOT THE CLAIM YOU THINK.** Four problems:

- The first half (higher degree resists gauge formation) is ACID's own stated motivation for the hexagonal connectivity. [V] Unclaimable.
- The second half is close to a tautology under the tier-A/tier-B framing. "No gauges" means "rows fixed" means "tier A." Defining overhead as "did the detector set change" makes the implication definitional.
- **A counter-mechanism exists in ACID's own paper and points the other way.** More connectivity gives the solver more room to cram in redundant contractions (its explicit tie-break objective), which adds detectors and fault mechanisms and grows the DEM. ACID attributes its non-graphlike DEMs to exactly this. [V] So connectivity redundancy has two opposite-signed effects on decoder hardware cost. **Which dominates is unknown, and that is the research question.**
- **Methodological error in the comparison, inherited from the literature.** ACID's plots fix the *number* of dropped components, not the *rate*. At fixed per-coupler failure probability, hexagonal BB144 suffers 432/360 = 1.20× the expected coupler dropouts of degree-5, and square-grid surface 480/340 = 1.41× that of hex-grid. Every "higher connectivity is more robust" comparison you are building on uses the wrong control variable. Re-running at fixed per-component rate is a one-line change and a legitimate small result.

**4.5 "ACID makes no claim about decoder complexity." CONFIRMED.** Its outlook discusses hardware co-design of connectivity for fabrication flexibility and says nothing about decoding cost; the only decoder-adjacent remark is the graphlike caveat. [V] So *"we additionally show that decoder complexity is reduced on average by this connectivity"* is available. Two cautions: it is a claim about a **tool's output** under ACID's current scheduling objective, which the author expects to be replaced, so scope it that way; and do not write it before you have the sign.

**4.6 "Spacing simultaneously-measured qubits apart in HAL placement mitigates measurement crosstalk." WEAKENED, NOT REJECTED.** It attacks the second-order spatial channel. The dominant knob is readout frequency and multiplexing-group assignment, which HAL does not model. See §5.4 for the reframe.

**4.7 "Codes with worse parameters but more check dependencies may decode better." OCCUPIED AND EMPIRICALLY DAMPENED.** The named field is **quantum data-syndrome codes** (Ashikhmin, Lai and Brun) [?] recalled, verify before citing, plus the single-shot line (Quintavalle, Vasmer, Roffe, Campbell). And LightStim's own +12 detectors produced no clear LER signal. [V] Redundancy buys single-shot capacity, not LER.

**4.8 "A lone gauge measurement whose partner comes in the next layer still yields a detector." NO, with a real residual.** The number of independent deterministic parities per round equals the rank of the stabiliser group, not the number of quasi-stabilisers. The first gauge measurement prepares the gauge frame, the second checks the product. Ordering *does* matter, and ACID already exploits it via the "measured more than twice with no anticommuting partner in between" rule. [V] The larger effect of ordering is on **detector spacetime volume**: a completion spanning two layers gives a detector covering more spacetime, so more fault mechanisms flip it and effective distance falls.

---

## 5 · The reshaped idea 3

### 5.1 Framing

Not "predict hardware cost from the DEM," but:

> For a fixed layout and a dropout distribution, what is the **distribution** of decoder hardware cost, and can a graph-level summary of the DEM predict it without a synthesis run?

The distributional framing is the right one. It composes with v15 without changing its shape: v15 already reports expected k·d²_eff/n at a yield quantile, and "P(hardware complexity > X)" over the same dropout distribution is the same functional applied to a different cost. Two numbers per layout from one Monte Carlo.

**Report quantiles, not variance.** The distribution will be heavy-tailed, because most patterns are absorbed cheaply and rare fragmenting patterns blow the DEM up. Report the 50th, 90th and 99th percentiles and the fraction exceeding a fixed budget. "Size for the 90th percentile" is a spec; "the variance was 4.2" is not.

### 5.2 The ground-truth metric, which is not a proxy

Do not invent a cost model. Use what the tools emit:

- **Partial reconfiguration bitstream size in bits** between the baseline decoder and the repaired decoder. This is the literal cost of the change, not a proxy, and it is the direct quantum instance of the classical reconfiguration-data-minimisation metric (§3.6).
- Post-implementation LUT/FF counts, routed wirelength, congestion maps.

All of these are **reports, not measurements**. No board is required. Timing closure is the only part that needs a physical target, and you do not need it for area.

### 5.3 The experiment

1. Write a DEM → Verilog generator for a deliberately crude fully-parallel min-sum BP decoder. **Instrument quality is not product quality.** You are measuring the DEM's effect on cost, not building a good decoder. Conflating the two turns this into a semester of timing closure.
2. Generate N dropout patterns at fixed per-component rate, run ACID, emit DEMs.
3. Batch Vivado synthesis and implementation. Overnight for a small code.
4. Regress reported cost and Δ(bitstream bits) against candidate predictors: node and edge counts (**the null**), degree distribution, hyperedge arity distribution, bisection width, Rent exponent from recursive hMETIS partitioning, symmetric difference of the nonzero set.

**The null hypothesis to beat is Vegapunk's "LUT ∝ column count."** If your metric correlates no better than `len(dem.flattened())`, you built an expensive column counter. So hold DEM *size* roughly fixed and vary *structure*, which different dropout patterns at fixed dropout count give you for free.

**Architecture note:** generated-from-H is specifically the qLDPC line (IBM Relay-BP, Vegapunk, GARI, Báscones). Surface-code hardware decoders (Helios union-find, Riverlane collision clustering, AFS) are lattice-structured and hand-architected, not generated from a matrix. This conflicts with the earlier advice to gear up on the surface code for published ground truth. **Resolution: validate the DEM pipeline on the surface code, where Anker and Debroy publish reproducible LER numbers; run the hardware-cost study on BB, where generated-from-H is the real architecture.**

### 5.4 Adjacent results that fell out

**The covering reframe (better than "one universal decoder").** Ask *how many distinct decoder configurations cover q% of fabricated chips*. This is a set-covering problem, it makes the distributional question the objective, and it has an honest industrial analogue in semiconductor **binning and speed grading**: fabs already sort chips by post-fabrication characteristics and ship different SKUs. The one-decoder framing only survives at large n and realistic yield, where every chip has many dropouts and the patterns become dense and dissimilar.

**Dropout-pattern orbits under the code's automorphism group. This is the strongest single idea to come out of the session.** If two dropout patterns are related by an automorphism of the code, the repaired codes are isomorphic, the DEMs are isomorphic, and one decoder configuration serves both after relabelling. **The number of distinct configurations needed is the number of orbits, not the number of patterns.** BB codes have large automorphism groups from their translation structure; AutDEC (arXiv:2503.01738) already operates on automorphisms of H_DEM. [V-ref] For BB144 this is potentially a reduction of order 100 in the covering problem, computable today with pure group theory and no simulation.

Note the reuse: working context §9.4 rejected an idea because "translation preserves every pairwise defect separation, so the entire translation orbit is one instance." Same theorem, opposite sign, already in the rejected register with a stated justification.

**Caveat that keeps it honest:** HAL's layout breaks the code's symmetry, so automorphism-related patterns give isomorphic DEM *structure* but different **p**. That is exactly tier A: reload priors, keep the routed graph. Since re-synthesis is the expensive operation and priors are a register write, the orbit argument survives at precisely the level that costs money.

**The superset DEM, already Project A from the earlier planning.** Build for the union of DEMs up to quantile q, mask per-instance. If the superset fits, every chip below it is a register write and the tier-A/tier-B distinction collapses in your favour.

**The worst-case hardware project.** Sizing the decoder to the worst dropout pattern is the right hardware deliverable, but size to a **quantile**, not a max: the true max includes pathological fragmenting configurations that would waste most of the FPGA.

**Measurement crosstalk, reframed.** ACID gives you the simultaneity graph (qubit pairs measured in the same layer) for free. The correct hardware target is **readout frequency and multiplexing-group assignment**, a graph-colouring problem on that graph, not xy-position. HAL placement remains relevant for the second-order spatial channel, with one argument specific to this project: in a multilayer architecture with bump bonds and TSVs, out-of-plane stray coupling paths differ from planar devices, so the spatial channel may carry relatively more weight than in the Zurich or Google planar data. Unchecked, and HAL is uniquely positioned to say something about it.

**Hardware cost as a detector-trimming heuristic.** A one-line change to ACID's CP-SAT objective: penalise DEM growth instead of maximising total contractions. What makes it a paper rather than a tweak is that pruning is **three-way priced**, one term per subteam: LER (§11.6, ambiguous sign), hardware cost (monotone improvement), and single-shot capacity (monotone degradation, since redundant measurements are the linear dependencies supplying the metasyndrome, T1.8). A genuine tradeoff surface with no dominating term.

### 5.5 Two scripts to run before committing anyone

Both are cheap, need no hardware, and between them determine the entire spring hardware spec:

1. **How much bigger is the union DEM than the median DEM** over a dropout distribution on one layout?
2. **How many orbits do dropout patterns fall into under Aut(BB144)?**

The answers select between "one superset decoder," "a dozen binned configurations," and "per-chip synthesis."

---

## 6 · Live theory ↔ decoding channels

Check these before proposing anything. Several decoding ideas are theory threads in disguise, and two decoding findings feed theory.

| Channel | Direction | Status |
|---|---|---|
| **T1.8 single-shot capacity under ACID repair** | theory → decoding | Same object as the detector-completeness question in §2. Rank of the metacheck matrix before and after repair. Pure F₂ linear algebra, no simulation. |
| **Detector completeness of ACID output** | decoding → theory | If ACID misses redundancy detectors, its published LER numbers are pessimistic and the metasyndrome is invisible to the decoder. |
| **Detector trimming (§11.6 prune-extra-measurement)** | three-way | Needs LER (theory), hardware cost (decoding + hardware), single-shot capacity (theory). |
| **ACID scheduler objective** | decoding → theory | Cram-in is the cause of both non-graphlike DEMs and DEM inflation. A DEM-growth penalty in the CP-SAT objective is the intervention. |
| **Measurement simultaneity graph** | ACID → HAL | Frequency/multiplexing-group colouring, not placement. See §5.4. |
| **Di Bella exposure metric (arXiv:2604.01040)** | HAL + ACID → decoding | Geometry-conditioned correlated noise; Appendix G is a decoder-mismatch theorem with a MAP reduction, so geometry-aware priors propagate to the decoder. [V-prior] |
| **Tier-A / tier-B dropout classification** | theory → hardware | Tier A = rows fixed, columns changed (reload H config image). Tier B = rows changed (also reconfigure syndrome-mapping table and windowing). Fraction of the dropout distribution in each tier, per connectivity graph, is the spring hardware spec. |
| **HAL-derived per-edge priors** | HAL → decoding | The clean priors-only experiment: decode with uniform p versus HAL-derived p on an identical DEM. Scope it to *fidelity*, never call it a dropout test. |
| **Zhu et al. arXiv:2604.05874** | competitive | "Universal superstabilizer scheme applicable to data qubit defects in arbitrary stabilizer codes," applied to colour codes on square lattices. Occupies the weight-1 gauge generalisation claim. Surviving slice: ancilla-based, untested against ACID's ancilla-free contraction paradigm. |
| **AlphaSyndrome, ASPLOS 2026** | competitive | Reinforcement learning for syndrome measurement circuit scheduling on qLDPC codes. Competes with the layer-level SAT optimisation thread. **Author attribution is inconsistent across sources (one gives Viszlai et al., arXiv:2601.12509; another gives Chen, Shi, Javadi-Abhari, Iancu, Li). Resolve before citing.** |

---

## 7 · Open questions and things not checked

Named so future sessions can weight the above correctly.

- **ACID source** at `github.com/riverlane/ACID`. Everything here about ACID is from the paper. The detector-assignment implementation is unread. Highest-value hour available.
- **LightStim source**, specifically whether WriteBack emits a detector when a stabiliser row's record is refreshed by a new decomposition. Decides the Case-A concern in §2.
- **Anker and Debroy §V in full.** Only the Appendix D ACID-adaptation passage has been read. Their reported finding that ACID sometimes produces operator sets admitting no valid superstabiliser/gauge group with canonical commutation relations, occurring when a dropout region touches the boundary, is a DEM-*existence* problem and is only partly characterised here. [V for the fact, not for its scope]
- **García-Herrero et al. noise model in detail.** Not re-verified this session.
- **QUITS noise-model interface** — whether it accepts heterogeneous per-operation rates.
- **Quantum data-syndrome codes attribution** (§4.7). Recalled, unverified.
- **Rent-exponent extraction on bipartite hypergraph-derived netlists.** The classical literature is on logic netlists. Whether the extraction is well-behaved on a DEM's bipartite structure is unexamined.
- **Prior art on decoder provisioning under a fabrication-yield distribution.** Zero searches. No novelty claim has been made about it and none should be.
- **Isomorphic DEM remapping for hardware reuse.** One sweep, one vocabulary. Do not put a novelty claim in writing on that basis.
- **Whether the IBM contact can help.** Do not ask for gateware; they will not give it. Ask something answerable by email: how utilisation moved between the Loon [[56,2,10]] and gross [[144,12,12]] configurations, and whether their resource scaling with matrix size was closer to linear in nonzeros or superlinear. That single answer tells you whether the FOM has anything to explain before a line of Verilog is written.

---

## 8 · Calibration note for this session

**Agent wrong:** initially treated LightStim integration as an open thread without first reading ACID §3.4, which closes it outright. This is the same failure the working context §4 rule exists to prevent (read primary sources in full; if a tool has public source, read the source). Also framed the DEM as a "proxy" for hardware cost when for the fully-parallel architecture it is the netlist itself.

**User right:** the distributional framing of the FOM (§5.1), the covering reframe over a single universal decoder, and the isomorphic-remapping idea, which is the strongest result to come out of the session and which the agent had not considered. Also correct that gauge measurement ordering carries real structure, which ACID does exploit.

**User wrong:** the bitstream project premise, which does not survive the observation that a DEM is definitionally an independent-fault model. The HAL-placement version of the measurement-crosstalk heuristic, which targets the subdominant physical channel.

Pattern consistent with the standing record: the user has been stronger on framing and problem selection, the agent on mechanism and sourcing.

# Session Summary: Decoding Subteam Scoping and Club Project Design

**Covers conversation of 2 to 27 September 2026. For future-session context.**

Companion to *Working Context v4*, the *v4.1 addendum*, and the *Unified Club Project Plan* (both produced this session). Citation marks: **[V]** confirmed against the paper this session, **[V-ref]** via reference list or citing page, **[V-prior]** from an earlier session. Where this file and the plan disagree, this file is newer (it includes the corrections in sections 6 and 7).

---

## 1 · Starting point

User asked for a literature sweep on three candidate decoding-subteam projects bridging hardware's real-time FPGA qLDPC decoding and theory's defect-conditioned multi-code chip project:

1. BP-OSD adapted to hardware info via weighted Tanner graphs, plus modified post-processing.
2. Syndrome bitstream generation for the FPGA, for niche codes (radial, tile), conditioned on HAL output.
3. Parallelizable, switchable, code-specialized BP cores on one FPGA.

**Verdict on all three: occupied in the stated form.** Details in section 5. The surviving idea came from reframing around the club's own premise (per-chip code selection under fabrication defects).

---

## 2 · Club state (as of late September 2026)

- **Theory:** building the ACID + HAL + LUCI / PropHunt pipeline, target end of calendar year. HAL MVP and an L=3 vs L=4 schedule project are tracked in separate memory files.
- **Hardware:** spending this semester rebuilding a Riverlane-style real-time decoder on FPGA as a training artifact. This is a clustering / Union-Find decoder, surface-code only.
- **Decoding:** new-member onboarding for the first weeks, then a gear-up project this semester, then the full club-spanning project next semester.

---

## 3 · The two unified projects (decided)

Umbrella: **decoder provisioning under fabrication defects.** Only unoccupied ground found, because nobody else selects a code per fabricated chip.

### 3.1 Project A (primary): decoder provisioning across a defect-varying code

- Every chip's repaired circuit has a different DEM. Decoder must be re-synthesized per chip or provisioned once for a **superset DEM with per-instance masks**. That object is both the theory comparison device and the hardware architecture.
- **Labor split (revised mid-session):** theory keeps only multi-code HAL layout (already a full project per v15 section 8). Decoding takes the DEM-variation study, superset construction, masking-accuracy cost, provisioning envelope, and the mismatch bound.
- **Decoding does not need theory's family.** Three surrogate families: one code with many dropout patterns; one code and dropout with many ACID schedules; several 2BGA codes on ACID's degree-6 hexagonal connectivity (`github.com/QEC-pages/2BGA-codes`).
- **Hardware:** memory-based BP decoder parameterized by the envelope, DEM as loadable config image. Report area, latency, reconfiguration time, accuracy for per-instance synthesis vs masked superset vs wired-graph on a small code. Deliverable is the trade curve.
- **Theory content inside decoding:** bound LER penalty of family-averaged (mismatched) DEM vs per-chip matched DEM at a **yield quantile**, not expectation. Named machinery: compound channels, universal decoding (MMI, Csiszár-Körner, Feder-Lapidoth), mismatched decoding (LM / GMI rates). Not Shannon's random-coding average. Di Bella Appendix G supplies roughly a third of the setup (augmented DEM, mismatch theorem); multi-code embedding, quantile objective, and classical-theory import are open. Caution: misprediction governed by a min over ambiguous pairs, not smooth under averaging.
- Failure modes accepted in advance: no bounded-degree superset (then "per-chip synthesis is unavoidable, here is the cost"), or negligible mismatch penalty (then "just update priors").

### 3.2 Project B (secondary / parallel theory track): single-shot capacity as a decoder-area lever

- Radial codes are single-shot via linear dependencies among stabilizer generators (metachecks). ACID repair changes the rowspace. **Prediction: single-shot capacity degrades under dropout faster than distance.**
- Theory: rank of metacheck matrix (left null space of H_Z) before and after repair over a dropout distribution. Pure F2, no simulation. Answers T2.16, prices WC section 11.6.
- Decoding: two-stage single-shot decoding on repaired, rank-deficient metacheck matrices. LER per round at r vs d rounds. Report round counts explicitly.
- Hardware: **sweep W over [1, d]** (user's correction, accepted). Expect affine not linear, sample small W densely, fix commit-width policy (e.g. C = W/2), report latency as well as area. **Single-shot is not the same as small W**: run windowed sweep and two-stage single-shot separately, compare single-shot at W=1 against the windowed W with matching LER.
- Ranked second because the hardware deliverable is contingent on theory's answer.

### 3.3 Regime constraint

IBM gross-code Relay-BP, X sector only, int4, 12-cycle window: 2,106,738 LUTs, 51.56% of XCVU19P, 58 W, 24 ns/iteration [V]. Club boards are 50k to 400k LUTs, so **memory-based architecture is forced**, which is also the regime where reconfiguration is a config load. Deliverable: synthesized, timing-closed, hardware-verified decoder with resource and latency report, validated against a bit-accurate software model on synthetic syndrome streams. Not closed-loop with a QPU.

---

## 4 · Fall semester decoding plan (current state after final corrections)

1. **Week one:** read `github.com/riverlane/ACID` source to settle T2.11 (does ACID emit detector-annotated Stim circuits, under what SPAM assumptions).
2. **M1:** run LightStim on one ACID dropout circuit. Checks: accepts external circuits or needs own IR; behaves with random gauge outcomes.
3. **M2:** ACID to annotated Stim to DEM to syndrome stream, plus reference decoder, on the **surface code**, validated against Anker and Debroy published LER. Non-i.i.d. bits via correlated CNOT errors (native in Stim). **Leakage dropped from fall scope** (see 6.3).
4. **M3:** DEM variation under dropout, framed as a **hardware-sizing distribution** (node counts, degrees, window at quantiles). Falsifiable form: ratio of 99th-percentile to median node count.
5. **M4:** reconfiguration-tier measurement. Tier A = rows fixed, columns changed (reload H only). Tier B = rows changed (also reconfigure syndrome mapping and windowing). Fraction per tier vs connectivity graph and dropout rate. Degree-5 BB morphing should be near zero Tier A (every coupler a bridge); degree-6 hex substantial for isolated losses. This tests whether v15's per-chip code selection is decoder-cheap: a finding about the club's own central claim.
6. **Stretch:** Project B metacheck rank study.

Alternative the user proposed and the plan records: breadth-first ACID compilation for many qLDPC codes. Accepted as legitimate if one code is validated first and the M3 cross-family characterization is attached as the presentable result.

**Flag for hardware:** fall UF decoder is a training artifact; spring target is memory-based message passing. Say this explicitly in September.

---

## 5 · Rejected directions (do not re-propose without new evidence)

- **Weighted Tanner graph BP for hardware asymmetry.** Decoders run on the DEM bipartite graph with per-fault priors (H, A, p); asymmetry enters via p. Trainable edge weights are a decade-old line (Nachmani 2016; Liu and Poulin PRL 2019; arXiv:2212.10245; arXiv:2603.21730).
- **Improve OSD post-processing for real-time FPGA.** Maurya et al. arXiv:2511.21660 v2 [V]: OSD and clustering both slower and less accurate than Relay on FPGA.
- **Multiple switchable code-specialized BP cores on one FPGA.** Classical multi-mode QC-LDPC literature; quantum version is Báscones et al. arXiv:2605.01035 [V]; fully parallel variant does not fit.
- **Generic syndrome bitstream generator.** Code-agnostic sparse multiply; done in fabric by García-Herrero et al., Quantum 10, 2071 (2026) [V]; IBM uses the same as verification harness. Also check QUITS.
- **Bayesian / Kalman prior tracking over a qubit grid.** Crowded noise-learning field: Spitz et al. 2018; p_ij method; Blume-Kohout and Young arXiv:2504.14643; Wagner et al. arXiv:2504.20212; Arms et al. arXiv:2512.10814; Bhardwaj et al. arXiv:2511.09491; arXiv:2609.00169; arXiv:2511.01080 already runs an approximate Kalman filter. Only unoccupied slice: repaired subsystem DEMs (hypergraph, multi-point correlations).
- **Prove Kalman convergence for QEC priors.** Wrong theorem (Riccati convergence is textbook; DEM estimation not linear-Gaussian). Right question is identifiability plus Cramér-Rao (6.2).
- **Shannon average-code argument for family-averaged decoder.** Category error; use compound / mismatched / universal decoding.
- **Round-to-round switching among PropHunt circuits under drift.** TLS switching timescale is seconds (arXiv:2608.02086), Google recalibrates every few runs (arXiv:2408.13687), rounds are about 1 µs: off by about 10^6. Reframed version is live (6.2).

---

## 6 · Live but out of current scope

### 6.1 Assigned to theory
- **Exposure as ACID x HAL joint quantity.** Di Bella weighted exposure needs pairwise closest approach between simultaneously active gate blocks: HAL knows geometry, only ACID knows simultaneity. Third cross-tool channel not in v15 section 1.3. Before claiming n-tier generalization is open: sweep crosstalk-aware routing / layer assignment EDA literature; n planes may be a trivial kernel evaluation; metric may saturate (null) on radial codes. Decoding residue: one experiment, decode with and without geometry pair-fault columns.
- **Leakage exposure as an ACID scheduler objective.** Ancilla-free contraction roots are measured and reset; measurement / reset are dominant leakage sites; ACID objective 2 pushes toward more measurement. Second independent argument for pruning (WC 11.6). Not a decoder prior: leakage is non-Pauli.

### 6.2 Unassigned research ideas
- **Identifiability of repaired DEMs.** Gauge operators add rowspace degeneracy, the mechanism for non-identifiable directions, so repaired codes should be harder to calibrate from syndromes. Key prior work: Chen, Liu, Otten, Seif, Fefferman and Jiang, Nat. Commun. 14, 52 (2023), arXiv:2206.06362 [V] (learnable = cycle space, unlearnable = cut space, 2n minus c gauge DOF); Blume-Kohout and Young; Arms et al.; arXiv:2609.00169. **Load-bearing risk:** follow-up work argues unlearnable gauge DOF do not affect predictions; if it carries over, result is "harmless."
- **PropHunt circuit bank switched as priors drift** (user's idea, conceded as open in this framing). Nearest prior art: adaptive syndrome extraction (conditions on syndromes, not drift); LUCI's event-driven TLS rerouting. Well-posed version: small precomputed bank switched on calibration timescale. Question: how coarsely can the drift axis be quantized before mismatch penalty exceeds re-optimization gain. Caveat: circuit distance is a min, so rotation has worst-member distance.
- **Correlation metrics across families (user's "overlapping stabilizer metric").** Resolves into two mechanisms: leakage (DEM column weight x leakage lifetime; check-intersection / 4-cycle profile for correlation) and crosstalk (geometric, Di Bella exposure). Tile codes: more overlap but about 4x shorter edges than BB, so mechanisms point opposite ways. Evaluate on DEM, not code Tanner graph. Leakage-specific cross-family sweep not yet done in four vocabularies.
- **Fidelity-only priors test.** HAL per-edge length / bump / TSV counts into gate error rates, identical DEM structure, uniform vs HAL-derived p. Clean measure of HAL-to-decoder channel value.

### 6.3 Leakage simulation (why dropped from fall)
Stim supports Pauli channels only; leakage is a persistent state. Options: Google Pauli+ (GPTA, not open source); Plaquette arXiv:2607.08767 [V] (CPTP channels compiled, demonstrated on superconducting leakage); QMCtwin arXiv:2606.19848 [V]; subspace twirling arXiv:2312.10277 [V]. Revisit Plaquette in spring.

---

## 7 · Key technical facts established this session

### 7.1 Stim vs LightStim vs ACID
- **Stim does not find detectors.** User supplies DETECTOR / OBSERVABLE_INCLUDE; Stim propagates faults to build the DEM and raises on non-deterministic detectors. `allow_gauge_detectors=True` emits error(0.5) fusing them, with documented caveats.
- **LightStim** (arXiv:2604.21472 [V], `github.com/QuTone/LightStim`) does detector discovery via measurement-record-augmented tableau; finds detectors manual annotation misses in BB and toric codes; ships unified decoder backend. One citing paper found (arXiv:2606.14677, DEM equivalence checking, same group). Names spacetime codes (Delfosse and Paetznick; Suau et al. 2026) as closest prior work.
- **ACID** (arXiv:2512.01943 [V]): per Anker and Debroy Appendix D (arXiv:2512.10871 v2, 27 Jul 2026 [V]), ACID assumes **noiseless SPAM via Stim MPP**; Google replaced with noisy ops. For some dropout configurations touching the boundary, ACID's measured operators admit no canonical superstabilizer / gauge decomposition; Google removed unpairable operators by hand. So gauge-group identification and detector construction downstream of ACID is a real step, possibly manual, possibly MPP-dependent. **Unconfirmed until source is read (T2.11).**
- What decoding would automate: ACID schedule to correctly annotated Stim circuit under realistic SPAM, for subsystem codes, including boundary cases.

### 7.2 Graphlike DEMs are irrelevant (user correction, conceded)
Anker and Debroy made ACID output graphlike only as a control for a surface-code matching comparison. qLDPC DEMs are hyperedge; BP-family decoders and PropHunt handle arbitrary arity.

### 7.3 Reference decoders
- Float references are free: `qLDPCOrg/qLDPC` (BP-OSD, BP-LSD, belief-find, Relay-BP, PyMatching, sinter, sliding window, `DetectorErrorModelArrays`); `github.com/trmue/relay`; CUDA-Q QEC GPU Relay-BP; LightStim backend.
- Riverlane Collision Clustering is a UF implementation (arXiv:2309.05558, Nat. Electron. 2025); Local Clustering Decoder (Nat. Commun. Dec 2025) available in software via **Deltakit**; Verilog not public. Helios (arXiv:2406.08491) FPGA UF matches Delfosse UF output exactly.
- **UF is combinatorial, so bit-exactness is easy** (only tie-breaking and growth ordering can diverge). The hard bit-accurate quantized model is a spring BP problem.
- **UF does not work on qLDPC** (hyperedges). Moving to qLDPC forces an algorithm switch to BP; fall establishes cross-validation methodology, not a reusable artifact.

### 7.4 Google already has repaired-subsystem-DEM tooling for the surface code
LUCI simulations and Anker and Debroy ILP work require it; Kim, Gicev, Sevior and Usman arXiv:2607.01887 [V] publishes LUCI detector construction on IBM hardware. Correct framing: extend a demonstrated surface-code capability to qLDPC under a realized layout. The "first to build X" framing is unavailable.

---

## 8 · C2S2 collaboration (cryo-CMOS + adiabatic decoding ASICs)

Context: hardware lead also leads a C2S2 subteam; C2S2 has TSMC 65 nm access, no long-term application, and adiabatic standard cells from last semester's fully adiabatic flash ADC. Advisor and Prof. Fatemi raised the idea.

**Field state:**
- Cryo-CMOS control is industrially mature (IBM APS 2026 cryo-CMOS flux bias on Heron R2, framed around qLDPC coupler counts; Intel Horse Ridge; Google 28 nm under 2 mW at 3 K; IBM 14 nm 23 mW/qubit). Do not enter.
- Cryogenic decoding has converged on **partial** decoding at 4 K: Michigan Pinball (cryogenic predecoder, HPCA 2026) and CryoZip (arXiv:2606.30805, syndrome compression, 22 nm FDSOI at 4 K, up to about 42x combined savings). Earlier: SFQ QECOOL (arXiv:2103.14209) and QULATIS; cryo memristor neural decoder (arXiv:2501.14525).
- **AQFP** (Yokohama: Takeuchi, Yamae, Yoshikawa) needs a Nb JJ process; C2S2 cannot build it. Do not conflate.
- **Adiabatic CMOS for cryo quantum control:** Q2LAL (DeBenedictis / Zettaflops, IEEE 2021; arXiv:2504.09229); S2LAL (Frank, arXiv:2009.00448) notes leakage-limited dissipation at 4 K. Simulation-only; **no fabricated adiabatic-CMOS chip characterized at 4 K found.**

**Arithmetic:** 4 K budget conventionally about 1 W (real systems a few W, already consumed by control). IBM FPGA decoder 58 W for one sector. Gap 60x to 300x. Optimistic stack (ASIC 10-20x, cryo 1.5-3x, adiabatic about 10x). **Central tension: adiabatic trades energy for time (roughly 1/T), QEC has a hard ~1 µs latency budget.** Resolve on paper first.

**Honest bridges:** (1) an ASIC cannot be re-synthesized, so Project A's provisioning envelope becomes the ASIC spec (strongest link); (2) defect-adaptive cryogenic predecoding for qLDPC (Pinball / CryoZip are surface-code, fixed-code); (3) systems paper against arXiv:2601.03922.

**Moat:** fabricated adiabatic CMOS measured at 4 K (fabrication moat, not idea moat; idea is Q2LAL's). Secondary: defect-adaptive cryo predecoding.

**Tiers:** Tier 0 energy-latency feasibility study (no tapeout, start now, joint paper). Tier 1 C2S2-internal tapeout of adiabatic cells, datapath, and **power-clock generator first** (historical failure point), characterized at 4 K. Tier 2 joint defect-adaptive cryo predecoder, only after Project A envelope exists and Tier 0 is positive.

**Sequencing decision:** Project A is the club spine. Run Tier 0 and C2S2's Tier 1 groundwork in parallel now (MPW calendars, C2S2 may otherwise find another project). Keep them inside C2S2's boundary with one QCA liaison. Hardware lead should be interface, not executor, given current load.

**Unverified and load-bearing:** Prof. Fatemi's claim that modified doping at existing TSMC nodes enables reliable 4 K operation. No citation found. TSMC 65 nm has no cryo PDK; expect Vth shift, subthreshold mismatch, self-heating.

---

## 9 · Scoop map

- **Avoid:** general real-time FPGA qLDPC decoding. IBM (Maurer, Bühler, Kröner, Haverkamp, Müller, Vandeth, Johnson) and García-Herrero / Valls group (Complutense, UPV; Báscones, Torres) shipping monthly.
- **Surface occupied, qLDPC open:** defect-adapted circuits and DEMs (Google: Debroy, Anker, McEwen, Higgott, Gidney).
- **Watch monthly:** PropHunt authors (future work names our composition); tile-code connectivity (two mid-2026 papers); Roffe group on radial codes under defects; any FPGA group adopting defect framing; anyone extending Di Bella past two planes or BB; LightStim on subsystem codes.
- **Open by premise:** decoder provisioning under per-chip code selection (horizon more than a year, conditional on premise staying ours); single-shot under defects.

---

## 10 · Open threads carried forward

- **T2.11** Read ACID source: detector annotation, SPAM assumptions, emitted-circuit contract. Week one.
- **T1.7** LightStim on one ACID dropout circuit.
- **T1.8** Metacheck rank before and after ACID repair.
- **T2.21** DEM variation across dropout patterns and family members (now M3).
- **T2.22** Mismatch experiment; reframed as the Tier A / Tier B measurement (M4) plus a separate fidelity-only priors test.
- Confirm hardware subteam is not committing to BP-OSD on FPGA.
- Get citation for the cryo doping claim.
- Check QUITS coverage before building overlapping tooling.
- Spot-check [V-prior] entries (HAL, PropHunt, Bravyi BB) before the plan circulates.
- Leakage-in-qLDPC cross-family sweep in four vocabularies (LRU, leakage mobility, leakage-aware matching, TLS).

---

## 11 · Calibration record for this session

**Agent wrong:**
- Presented the "priors-only" test as general; user showed qubit dropout and fragmenting coupler dropout change detectors. Corrected to Tier A / Tier B.
- Claimed a validated repaired-subsystem DEM pipeline was unbuilt without checking LightStim's mechanism or Google's surface-code tooling.
- Inherited Anker and Debroy's graphlike constraint as if binding on qLDPC.
- Dismissed drift-driven circuit switching as a one-paragraph sensitivity question; user correctly identified it as a distinct direction (timescale reframe then applied).
- Filed ACID x HAL exposure as decoding work; it is theory's.
- Proposed only W=2 and W=d for the hardware sweep; user's full [1, d] sweep is correct.
- Under-motivated M3 / M4 in the plan; purpose is hardware sizing and reconfigurability class, and testing v15's decoder-cost assumption.
- Implied the fall reference decoder needed hard bit-accurate quantization; that applies to spring BP, not fall UF.

**User wrong or overstated:**
- Weighted Tanner graph BP as a research problem (priors already per-fault).
- Syndrome bitstreams as code-specific and thin for niche codes (generation is code-agnostic).
- Shannon average-code framing (use compound / mismatched decoding).
- Kalman convergence as the theorem (identifiability / CRB is the content).
- Round-to-round circuit switching (six orders of magnitude off the drift timescale).
- Initial plan to include leakage in fall bitstreams (not simulable in Stim; scope risk).

---

## 12 · Artifacts produced

- `working_context_v4.1_addendum.md`: sections 9.15 to 9.17, 10.15 to 10.18, 11.19 to 11.22, re-check list, T1.7, T1.8, T2.21, T2.22.
- `qca_unified_project_plan.md` and `.pdf` (11 pages, linked contents, split glossary): club-facing plan for subteam leads. Predates the corrections in sections 7.1, 7.3 and 11 of this file and the C2S2 material in section 8; update before circulating further.

## 13 · Standing working-mode instructions (from the user)

Red-team user ideas by default; do not affirm without specific source backing; frequent literature sweeps; push back on wrong threads but always collect sources; search at least four vocabularies before claiming novelty; prose in v15 style, clear and direct, no em-dashes, not verbose.

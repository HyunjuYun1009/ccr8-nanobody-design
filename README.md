# Computational Design and Validation of CCR8-Targeting Antagonist Nanobody

> From Orthosteric Pocket Insertion to N-Terminus Mediated Residence Time Extension

**Seoul National University College of Medicine — The Lee Lab for Protein Designers**  
Hyunju Yun | Advisor: Prof. Chang-Han Lee | 36th Undergraduate Research Training Program (2025.12.22 ~ 2026.02.13)

---

## Overview

This project aims to computationally design and validate an antagonist nanobody that blocks the CCL1-CCR8 interaction — a key immunosuppressive axis in tumor-infiltrating regulatory T cells (Tregs). The pipeline combines protein diffusion, sequence design, structure prediction, and geometric deep learning-based scoring to screen 1,000 candidates down to 14 experimentally viable leads.

---

## Background

**Why CCR8?**
CCR8 is selectively expressed on tumor-infiltrating Tregs. CCL1-CCR8 signaling suppresses anti-tumor immunity. Blocking this interaction depletes Tregs and restores immune responses, making CCR8 a compelling immuno-oncology target.

**Why Nanobodies?**
At ~15 kDa, nanobodies are small enough to access GPCR orthosteric pockets via their extended CDR3 loops — a space conventional monoclonal antibodies (mAbs) cannot efficiently reach.

**The mAb1 Problem**
The benchmark antibody mAb1 exhibits a striking paradox:

| Metric | Value |
|---|---|
| Binding affinity (KD) | 28.4 pM |
| Functional potency (IC50) | 57.9 nM |
| Gap | ~2,400-fold |

The root cause (Sun et al., 2023): mAb1 engages ECL1/2 only, blocking CRS2 insertion but leaving CRS1 open. CCL1 can still initiate binding at CRS1 and displace mAb1, limiting therapeutic duration. This large KD-IC50 gap signals a design opportunity.

---

## Pipeline

```
Caplacizumab scaffold (PDB: 7EOW)
            +
  CCR8 structure (PDB: 8TLM)
            |
     Phase 1: RFdiffusion
    CDR3 loop generation targeting
    hotspots Y116, F251, E279
            |
     Phase 2: PyMOL
    Manual CDR3 repositioning into
    CCL1 binding pocket (CRS1.5/CRS2)
            |
     Phase 3: ProteinMPNN
    1,000 candidate sequences generated
            |
     Phase 4: ESMFold
    Structure prediction (996/1000 success, 99.6%)
            |
     Phase 5: Dual Scoring
    Geometric Score + Quality Score
            |
     Phase 6: AlphaFold3
    Binding mode validation & final selection
            |
    14 Final Candidates (7 experimentally viable)
```

---

## Scoring System

### Geometric Score (proxy for binding affinity)

Evaluates interface geometry between nanobody and CCR8 receptor.

| Component | Weight | Description |
|---|---|---|
| Interface Score | 40% | pLDDT-weighted sigmoid score for residue pairs within 8 Å at the CCR8-nanobody interface |
| CDR3 pLDDT | 25% | Structural confidence of the key binding loop (residues 96-108) |
| Hotspot Proximity | 35% | Sigmoid-scored minimum distance to CCR8 hotspots (Q94, D97, Y172, Y184, H283) |

$$\text{Geometric Score} = 0.40 \times \text{Interface} + 0.25 \times \text{CDR3} + 0.35 \times \text{Hotspot}$$

$$\text{Interface} = \frac{100}{1 + e^{(d-5.0)/1.0}} \times \frac{\text{avg\_pLDDT}}{100}$$

$$\text{Hotspot} = \frac{100}{1 + e^{(d_{\min}-6.0)/1.5}}$$

### Quality Score (proxy for structural stability)

Evaluates intrinsic nanobody foldability without the receptor.

| Component | Weight | Description |
|---|---|---|
| Overall pLDDT | 30% | Average pLDDT across the entire nanobody |
| Framework pLDDT | 25% | Average pLDDT of FR1/FR2/FR3/FR4 scaffold regions |
| CDR3 pLDDT | 30% | Average pLDDT of the CDR3 loop |
| Loop Closure Score | 15% | Cα distance between CDR3 start (res 96) and end (res 108): $100 \times e^{-d/15.0}$ |

$$\text{Quality Score} = 0.30 \times \text{Overall} + 0.25 \times \text{Framework} + 0.30 \times \text{CDR3} + 0.15 \times \text{Closure}$$

---

## Key Results

### Score Distribution (996 candidates)

| Score Type | Mean ± SD | Range | Top 10 Range |
|---|---|---|---|
| Geometric | 5.90 ± 3.21 | 0.80 – 15.96 | 15.23 – 15.96 |
| Quality | 2.85 ± 0.82 | 1.45 – 5.73 | 3.92 – 5.73 |

A weak but statistically significant negative trade-off exists between the two metrics (Spearman ρ = −0.177, p < 0.0001), with 0% overlap between the respective Top 10 candidates. This motivates a multi-group selection strategy.

### Final Candidate Groups

| Group | Definition | Candidates |
|---|---|---|
| A | Geometric Top 5 | 826, 85, 746, 262, 978 |
| B | Quality Top 5 (incl. baseline) | original, 423, 759, 845, 915 |
| C | Combined Ranking Top 5 | 36, 814, 845, 561, 271 |

(Sample 845 appears in both Group B and C.)

### Binding Mode Distribution (AlphaFold3 validation, 14 candidates)

| Binding Site | Count | Percentage | Functional Potential |
|---|---|---|---|
| Correct Pocket | 1 | 7% | High |
| Floating / Cap | 6 | 43% | Moderate-High |
| ICL (artifact) | 5 | 36% | None |
| Edge | 2 | 14% | Low |

50% of candidates show extracellular-accessible binding, qualifying for experimental validation.

### Highlighted Candidates

**Sample_746 — Pocket Insertion Binder**
- The only candidate with CDR3 loop directly inserting between Y116 and F251
- d_min = 6.2 Å from pocket center
- RMSD between unbound and bound conformation: 0.239 Å over 684 atoms (low activation barrier)
- Mechanism: competitive inhibition of CCL1 CRS2 binding

**Sample_915 — Surface Capping Binder**
- Highest interface confidence among all cap-type binders (iPTM = 0.64)
- Mechanism: steric blocking of CCL1 access
- Clinical precedent: Mogamulizumab (CCR4), Erenumab (CGRP receptor)

---

## Idea Proposal: N-terminus Mediated Residence Time Extension

Beyond the current design, this project proposes a next-generation strategy inspired by findings from ACKR3-CXCL12 kinetics (Gustavsson et al., Science Signaling 2019).

**Key insight from ACKR3:** The receptor N-terminus does not affect potency but dramatically extends ligand residence time (up to 4-fold) through a wrapping and tethering mechanism. Residues 18-29 act as a kinetic anchor, increasing local concentration without directly modulating activation.

**Hypothesis for CCR8:** A nanobody that simultaneously engages ECL2 (antagonism) and the CCR8 N-terminus (tethering) will achieve:
1. Strong antagonist activity
2. Extended residence time (slower k_off)
3. Improved KD-IC50 ratio

| Feature | Sample_746 (current) | Proposed Nanobody |
|---|---|---|
| Primary target | Orthosteric pocket | Orthosteric pocket |
| Secondary target | None | CCR8 N-terminus |
| Binding mode | Pocket insertion | Pocket + tethering |
| k_off (predicted) | Baseline | 2–5x slower |
| IC50 (predicted) | Baseline | Lower |
| Mechanism | Competitive inhibition | Competitive + kinetic trap |

**Design Strategy:** Peptide Mimicry — synthesize CCR8 N-term peptide variants, perform ensemble docking, then redesign CDR1/2 to engage the tethering region.

---

## Future Directions

**Experimental Validation (priority targets: Sample_746 and Sample_915)**
- Surface Plasmon Resonance (SPR) / Bio-layer Interferometry (BLI) for binding kinetics
- ERK phosphorylation assay for functional antagonism
- CCL1 competition assay for blocking mechanism confirmation

**Design Improvements**
- Pareto optimization for simultaneous geometric + quality maximization
- Membrane-aware structural constraints during diffusion
- Multi-objective evolutionary algorithms for candidate generation

---

## Tools & Methods

| Tool | Purpose |
|---|---|
| RFdiffusion | CDR3 loop backbone generation |
| ProteinMPNN | Sequence design from structural template |
| ESMFold | Large-scale single-chain structure prediction |
| AlphaFold3 | Complex structure prediction and binding mode validation |
| PyMOL | Manual docking, visual analysis, binding site classification |

---

## References

- Sun et al. (2023) — mAb1 epitope mapping and CCR8 structural basis
- Gustavsson et al. (2019) — ACKR3 N-terminus residence time kinetics. *Science Signaling*
- Dauparas et al. (2022) — ProteinMPNN
- Watson et al. (2023) — RFdiffusion
- Lin et al. (2023) — ESMFold
- Jumper et al. / Google DeepMind (2024) — AlphaFold3
- Jing et al. (2021) — Geometric Vector Perceptrons (GVP)

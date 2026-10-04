# GOAL — Real-world applications of the Discrete Resonance Spectrogram (DRS)

## Context

This repository is a fork of `sthomer/drs`, the Discrete Resonance Spectrogram (DRS) by Steven T. Homer.
DRS represents a signal as a sum of **resonances** (damped complex oscillators) instead of infinite Fourier sinusoids.
Each resonance has four parameters: frequency, decay (damping), amplitude and phase.

Background documents in `docs/`:
- **Homer, Harley & Wiggins (2024)**, *Modelling of Musical Perception using Spectral Knowledge Representation* (Journal of Cognition). Defines resonance space as a Hilbert space, the inner product between resonance spectra, cosine similarity, and the harmonic operator.
- **Kahriman (2026)**, *Chord Detection using Discrete Resonance Spectrogram* (VUB bachelor thesis, promotor Prof. Geraint Wiggins). Uses DRS on sliding windows to detect chord changes.

### Key result from the thesis
The phase-dependent inner product from Homer et al. fails when comparing **consecutive windows of the same signal**, because phase rotates over time.
The thesis replaces it with a **phase-independent similarity**:

- Each resonance is treated as a Lorentzian (Cauchy) peak.
- Overlap between two peaks A and B is computed by **convolution** of the two Lorentzians, i.e. adding the widths:
  `L = (γA + γB) / (Δf² + (γA + γB)²)`
- Each peak is weighted by its **total power** (amplitude combined with decay), not its initial amplitude.
- `Overlap(A, B) = Σ PowerA · PowerB · L` over all peak pairs.
- Normalised with Cauchy–Schwarz:
  `S(A, B) = Overlap(A, B) / sqrt(Overlap(A, A) · Overlap(B, B))`, bounded in [0, 1]. Silent windows return 0.
- A drop below a threshold (0.10) between adjacent windows marks a change.

Results (F1): synthetic 0.99, monophonic 0.80, homophonic 0.65, polyphonic 0.60.

## The question

Can DRS, combined with this phase-independent similarity, solve or improve a **real-world problem** in a way that existing methods do not?

The original idea was **active noise cancellation (ANC) for earbuds**. Known concern: ANC needs latency well below a millisecond, while DRS works on windows and the Homer et al. paper itself states that computing resonances is much slower than an FFT and unsuitable for near real-time use.

## Research tracks

### Track 0 — Feasibility check for live ANC
- Measure how long DRS takes per window for realistic window lengths and sample rates.
- Compare this with the latency budget of feedforward and feedback ANC.
- Conclude honestly whether live ANC with DRS is possible, or under which conditions.

### Track A — Predictive cancellation of tonal noise
Tonal or stationary noise (engines, fans, transformers, aircraft hum) is well described by a few long-lived resonances.
Because resonances are a parametric model, the waveform can be **extrapolated forward in time**, which could hide the processing latency.
- Fit resonances on a past window, extrapolate the next N ms, and measure the residual after subtracting the prediction.
- Compare against baselines: LMS/FxLMS adaptive filtering, linear prediction (LPC), Prony's method, ESPRIT/matrix pencil.
- Report how prediction error grows with horizon length and with noise type.

### Track B — Machine fault detection via damping
In structural health monitoring and machine diagnostics, changes in **resonance frequency and damping** indicate wear, cracks or loose parts.
DRS gives the decay of each resonance directly.
- Find public vibration or acoustic fault datasets (for example bearing fault datasets).
- Track frequency and decay of the dominant resonances over time and test whether they separate healthy from faulty machines.
- Reuse the thesis similarity metric for change detection over time.
- Compare against standard features (FFT band energies, envelope analysis, MFCCs) with a simple classifier.

### Track C — Open
If the agents find a better real-world application that plays to DRS's strengths (precise frequency + decay, parsimony), propose it with evidence before investing in it.

## Rules for the agents

1. **Read first.** Read `src/drs`, `tests`, `exploration` and both PDFs before changing anything. The code has changed over time (now PyTorch, with a Fortran core and MDFPT); do not assume the README is complete.
2. **Literature check before claiming novelty.** DRS is mathematically related to Prony's method, Padé approximation, harmonic inversion and linear prediction. Search for prior work on each track and state clearly what is new.
3. **Every claim needs evidence.** Either a reproducible experiment in this repo or a real, verifiable citation. Never invent references or numbers.
4. **Always compare against a baseline.** A result without a baseline is not a result.
5. **Negative results count.** If a track does not work, document why. That is a valid outcome.
6. **Independent review.** A critic agent that did not produce the work checks each result: code correctness, data leakage, unfair baselines, overclaiming.
7. **Do not modify the original DRS core** unless needed; put new work in a separate folder.

## Deliverables

All new work goes in `research/`:
- `research/track0_latency/`, `research/trackA_prediction/`, `research/trackB_faults/`, each with code, results and a short `README.md`.
- `research/REPORT.md`: per track — question, method, baseline, results, honest conclusion, and next steps.
- A short list of open questions to discuss with Prof. Wiggins and Steven Homer.

## Success criteria

- **Minimum:** a clear, evidence-based answer on which tracks are viable and which are not.
- **Good:** at least one track where DRS measurably matches or beats a standard baseline on real data.
- **Excellent:** a result strong enough to form the basis of a paper or a master's thesis.

## Licence note

This fork is GPL-3.0. Any distributed derivative must also be GPL-3.0.

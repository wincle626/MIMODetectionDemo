# MIMO Detection & Decoding Demo (`MIMODetDemo_v3a`)

A MATLAB **App Designer** application for simulating and comparing the
bit-error-rate (BER) performance of MIMO **detection** algorithms and, optionally,
channel **coding/decoding** schemes over Rayleigh-fading MIMO channels.

Originally built for teaching (QUB ELE3001). It ships as a single App Designer
app, `MIMODetDemo_v3a.mlapp`, and is fully self-contained — no extra `.m` files
are required.

---

## Requirements

- **MATLAB R2019a or later** (developed/validated on R2026a).
- **Communications Toolbox** — required for the coded schemes Conv, Hamming, BCH,
  Reed–Solomon, LDPC and Turbo.
- **5G Toolbox** — required only for the **Polar** coded scheme.

Detection-only simulations (Coding Scheme = `Uncoded`) do not need any toolbox
beyond core MATLAB.

---

## Launching the app

Put `MIMODetDemo_v3a.mlapp` on your MATLAB path (or make its folder the current
folder) and run:

```matlab
MIMODetDemo_v3a
```

or double-click the app in the MATLAB **Current Folder** browser / **Apps**
gallery. A window titled **MIMO** opens with a BER plot area and four tabs.

---

## Quick start

1. **Simulation** tab — set TX/RX antennas, Modulation, Channel and the SNR range.
2. **Application** tab — choose a detector in **Case** (start with `ML` or `MMSE`).
3. **Decoding** tab — leave **Coding Scheme = `Uncoded`** for a detector-only run.
4. Click **Run**. The BER-vs-SNR curve is drawn in the plot; each SNR point prints
   its run time to the MATLAB command window.
5. Tick **Hold Figure** and run another configuration to overlay curves, then click
   **Legend** to label them.

To add channel coding, see [Running a coded simulation](#running-a-coded-simulation).

---

## The tabs

### Application tab — detector and its parameters

**Case** selects the MIMO detection algorithm:

| Case | Detector |
|------|----------|
| `MF` | Matched Filter |
| `ZF` | Zero-Forcing |
| `MMSE` | Minimum Mean-Square Error |
| `KBEST` / `RKBEST` | K-Best sphere decoder (complex / real-valued) |
| `SSFE` / `RSSFE` | Selective Spanning with Fast Enumeration (complex / real) |
| `BSS-EFE` | Bounded SSFE with Extended Fast Enumeration |
| `FTS` | (Reactive) Tabu Search |
| `FSD` / `RFSD` | Fixed-complexity Sphere Decoder (complex / real-valued) |
| `GSD` | Generalized Sphere Decoder |
| `ML` | Maximum Likelihood (optimal reference) |

Parameter fields appear only for the detector that uses them:

- **K** — list size for `KBEST` / `RKBEST`.
- **m** — spanning vector for `SSFE` / `RSSFE` / `BSS-EFE` (e.g. `4 1`).
- **Sphere** — search radius for `GSD` (`0` = unbounded).
- **SIC** — enable successive interference cancellation for `ZF` / `MMSE`.
- **Iteration, Repeat, Neighbour, Tabu Period, Reactive** — search controls for
  `FTS`.

### Decoding tab — channel coding

| Control | Meaning |
|---------|---------|
| **Coding Scheme** | `Uncoded`, or one of `Conv, Hamming, Turbo, BCH, LDPC, Polar, Reed–Solomon`. |
| **Decision** | `both` / `hard` / `soft` — which post-FEC BER curve(s) to plot. |
| **Frame Length** | information bits per coded frame, *K*. |
| **Coding Rate** | target code rate *R* (coded length *N* = round(*K*/*R*)). |
| **Number of Frame** | frames simulated per SNR point. |

When a coding scheme is selected, the **total simulated information bits per SNR**
= **Number of Frame × Frame Length**. Frame Length, Coding Rate and Number of Frame
must be filled in (Frame Length ≥ 1, 0 < Coding Rate < 1, Number of Frame ≥ 1) —
otherwise the app prompts you for them and does not run.

### Simulation tab — channel and sweep

| Control | Meaning |
|---------|---------|
| **min SNR / max SNR / SNR step** | SNR sweep range in dB. |
| **TX / RX** | number of transmit / receive antennas (e.g. 2×2, 4×4). |
| **Modulation** | `4QAM`, `16QAM` or `64QAM`. |
| **Channel** | `Uncorrelated` (i.i.d. Rayleigh) or `Correlated` (LTE TU model). |
| **Corr_type** | correlation level `Low` / `Medium` / `High` (Correlated channel). |
| **Sample Scale** | scales the sample count for uncoded runs (coded runs use *Number of Frame*). |
| **Hold Figure** | overlay new curves instead of clearing the plot. |
| **Legend** | add a legend built from the runs plotted so far. |
| **Run** | start the simulation. |
| **Reset** | clear the plot and stored curves. |

### File tab — saving the figure

- **Figure Name** — base filename for the saved figure.
- **Password / Save** — saving the current BER plot to a `.fig` file is protected
  by a password (configured in the app code as `app.Password`). Enter it, then
  click **Save**.

---

## Running a coded simulation

1. **Simulation** tab: set TX/RX, Modulation, Channel, and the SNR range.
2. **Application** tab: pick the detector in **Case**. This detector produces the
   soft (LLR) input for decoding — `ZF`, `MMSE` and `ML` use their own LLRs; the
   other detectors use ML LLRs.
3. **Decoding** tab: choose a **Coding Scheme**, a **Decision** (`both`/`hard`/`soft`),
   and enter **Frame Length**, **Coding Rate** and **Number of Frame**.
4. Click **Run**. The app plots the **post-FEC BER on the information bits**
   (the metric that shows coding gain), labelled with the scheme, decision and rate,
   e.g. `LDPC soft R=0.50`.

To see the coding gain, run the same case once with `Uncoded` and once with a
scheme (use **Hold Figure** to overlay).

### Hard vs soft decoding

- **Hard** feeds the decoder the detector's hard bit decisions.
- **Soft** feeds the decoder log-likelihood ratios (LLRs) — generally several dB
  better.

Conv/Hamming/BCH/Reed–Solomon have true hard decoders; their soft decoders use
Viterbi soft-decision (Conv), maximum-likelihood over the codebook (Hamming) and
Chase-II (BCH, RS). LDPC/Turbo/Polar are natively soft (belief propagation / BCJR /
successive-cancellation-list).

### Coding rate

LDPC and Polar are constructed directly at the requested rate. The other schemes
encode with their natural code and reach the target length by evenly-spaced
bit-level rate matching (puncturing or repetition).

---

## Output

- **BER plot** — BER (log scale) vs SNR (dB) in the app window. Multiple curves can
  be overlaid with **Hold Figure** and labelled with **Legend**.
- **Command-window timing** — each SNR point prints `Elapsed time is … seconds.`
- **Saved figure** — via the **File** tab (password-protected).

---

## Notes

- Detection and coding are generic in the antenna count (TX/RX) and modulation order
  (4/16/64-QAM). For very large joint constellations (e.g. 4×4 64-QAM), the exact-ML
  soft output automatically falls back to MMSE LLRs to stay tractable.
- `MIMODetDemo_v3a_original.mlapp` is a backup of the app before the decoding
  features were added.
- To customise the internals (detector list, codec block sizes, layout), open
  `MIMODetDemo_v3a.mlapp` in App Designer. The coded run is the `codedRun` method
  and the codecs are the `cc_*` helper methods.

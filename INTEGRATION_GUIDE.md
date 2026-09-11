# MIMO Decoding — What Was Added

The Decoding tab of `MIMODetDemo_v3a.mlapp` listed seven coding schemes
(`Conv, Hamming, Turbo, BCH, LDPC, Polar, Reed–Solomon`) but had no code behind
them. Channel **encoding + decoding** (hard and soft), the soft-output (LLR) MIMO
detection soft decoding needs, and the Decoding-tab controls to drive it have now
been built into the app.

Everything is **self-contained inside the `.mlapp`** — all the logic lives in the
app as private methods prefixed `cc_` (e.g. `cc_buildCoder`, `cc_encode`,
`cc_decodeSoft`, `cc_decodeHard`, `cc_detect`, `cc_qamGray`). There are no
external `.m` files to keep alongside it. Your original app was preserved as
`MIMODetDemo_v3a_original.mlapp`.

**Toolboxes:** Communications Toolbox (Conv, Hamming, BCH, Reed–Solomon, LDPC,
Turbo) and 5G Toolbox (Polar only).

---

## Using it

Open or run `MIMODetDemo_v3a`. On the **Decoding** tab:

- **Coding Scheme** — pick a scheme, or `Uncoded` (the default) for the original
  detector-only behaviour.
- **Decision** — `both` / `hard` / `soft`: which post-FEC BER curves to plot.
- **Frame Length** — information bits per coded frame (K).
- **Coding Rate** — target code rate `R`, so coded length `N = round(K/R)`.
- **Number of Frame** — how many frames are simulated per SNR point, so the total
  simulated information bits per SNR = Number of Frame × Frame Length.

Set the antennas / modulation / SNR range on the **Simulation** tab, then click
**Run**. You must enter Frame Length (≥ 1), Coding Rate (0 < R < 1) and Number of
Frame (≥ 1) first — otherwise the app pops up a dialog asking for them and does
not simulate.

Each SNR point prints its elapsed time to the command window
(`Elapsed time is … seconds.`), like the detection methods. The **Case** detector
selects the soft front-end (ZF / MMSE / ML use their own LLRs; the other
detectors use ML LLRs). Curves feed the existing **Legend** and **Save** buttons.

---

## How it works

**Hard vs soft.** Hard decoding feeds the decoder the detector's hard bit
decisions; soft decoding feeds it LLRs. LLR convention throughout is
`L = log( P(bit=0) / P(bit=1) )` (positive ⇒ bit 0), matching MATLAB's
`qamdemod('OutputType','llr')`, so LLRs pass straight to `ldpcDecode`,
`comm.TurboDecoder` and `nrPolarDecode`. Two decoders use the opposite sign and
are handled explicitly in `cc_decodeSoft`: convolutional Viterbi (fed `+L`) and
`comm.TurboDecoder` (fed `−L`).

Conv/Hamming/BCH/RS have genuine hard decoders; their soft decoders use Viterbi
soft-decision (Conv), ML-over-codebook (Hamming) and Chase-II (BCH, RS).
LDPC/Turbo/Polar are natively soft (belief propagation / BCJR / SCL); their hard
path feeds saturated ±LLRs so both modes share one interface.

**Coding rate.** LDPC and Polar are built directly at the target rate
(`N = round(K/R)`). The others encode with their natural code and reach `N` by
**evenly-spaced** bit-level rate matching — puncturing when `N < N₀`, repetition
when `N > N₀` (`cc_rateMatch` / `cc_rateDematch`). At the receiver, punctured
positions get LLR 0 and repeated positions have their LLRs summed. (Contiguous
puncturing was avoided — it destroys a whole Turbo parity stream.)

**Detection.** `cc_detect` implements ZF, MMSE and ML soft output. ML enumerates
all `M^mt` hypotheses (`LLR = (d₁² − d₀²)/N₀`) and falls back to MMSE LLRs when
that is too large (e.g. 4×4 64-QAM). Everything is generic in `mt`, `nr` and
`index ∈ {2,4,6}` (4/16/64-QAM). The QAM constellation (`cc_qamGray`) reproduces
the app's own Gray mapping so the LLRs are consistent with `QAM_Mod`/`QAM_Demod`.

The coded run reports post-FEC BER on the **information** bits (the metric that
shows coding gain) — run an `Uncoded` case alongside a coded one to compare.

---

## Adjusting it in App Designer

Open `MIMODetDemo_v3a.mlapp` in App Designer to change the layout or code. The
coded run lives in the `codedRun` method; the codecs are the `cc_*` methods
(default block sizes and Chase-II test-bit count are set in `cc_buildCoder`). The
Decoding-tab controls (`CodingSchemeDropDown`, `DecisionDropDown`,
`FrameLengthEditField`, `CodingRateEditField`, `NumFrameEditField`) and their
callbacks are standard App Designer components you can move or rename there.

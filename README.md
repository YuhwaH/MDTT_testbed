# MDTT_testbed_prototype

A configurable browser-based implementation of the **Multi-Directional Tapping Task (MDTT)** described in ISO/TS 9241-411 Annex B.6.2.2, built as the companion tool for:

> *Designing and Reporting Multi-Directional Tapping Tasks: A Systematic Review, Practical Guidelines, and a Configurable Testbed.*

<!-- ![The testbed's geometry tab](docs/screenshot-geometry.png) -->

---

## What it is for

Our review of 157 MDTT studies, spanning conventional and XR environments, found a stable procedural core alongside substantial variation in target count, shape, units, sequence handling, feedback, and the performance measures reported — variation that is frequently under-documented and that can change the measured performance. This tool is a planning and documentation aid for that problem. It makes every task parameter explicit, shows the geometry a configuration actually produces (which is not always the geometry you specified), runs a pilot so you can confirm the task behaves as intended, and emits a machine-readable specification you can cite or deposit alongside a paper.

It is deliberately not a standardisation device. Defaults are defaults; the tool flags the consequences of a choice rather than forbidding it.

## Quick start

1. Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge). The file is fully self-contained — no build step, no dependencies, works offline.

2. Set your display calibration in **R2** (screen width in pixels and millimetres, viewing distance). Everything else depends on it.
3. Configure the task in the left panel. The panel groups map one-to-one onto the paper's recommendations R1–R5; R6, on performance measures and Fitts' law analysis, is covered by the Results and Report tabs.
4. Check the **Geometry** tab: realized amplitude, movement directions, condition table, trial budget.
5. Run a pilot on the **Run** tab, inspect **Results**, then take the **Report** tab into your Methods section and download the files from **Export**.

## What you can configure

| Panel | Parameters |
|---|---|
| **R1** Apparatus & mapping | Input and output device and model, control-to-display mapping, selection trigger and hit definition |
| **R2** Difficulty, size & units | Amplitude and width lists, reporting unit (px / mm / cm / m / deg), display calibration and viewing distance |
| **R3** Target geometry | 2D plane (circle, square) or 3D spheres, target-plane depth |
| **R4** Count, sequence & trials | Visible targets N, sequence step *s*, amplitude compensation, repetitions, practice selections dropped, condition order, error handling |
| **R5** Feedback | Active-target cue, hover/contact highlight, visual and audio feedback for success and failure |
| Optional | Mirrored clockwise/counter-clockwise sequences, fixed / counterbalanced / randomised angular phase |
| **R6** Measures & analysis | Not configured — movement time, error rate, throughput, the MT–ID regression, and effective measures are computed and reported together on the Results and Report tabs |

### Geometry checks the tool performs for you

- **Realized amplitude.** A transition advancing by *s* of N targets has chord length `2R·sin(πs/N)`, not the circle diameter; for the near-opposite step with odd N this is `cos(π/2N) × diameter`. The tool reports the realized value and can rescale the circle so that it equals your nominal A.
- **Sequences that do not visit every target.** If `gcd(s, N) ≠ 1` the sequence closes early and revisits targets. The tool warns, and in automatic mode picks the coprime *s* closest to N/2. With even N, `s = N/2` is the only exactly diametric transition but oscillates between two targets, so the tool steps to the nearest coprime value instead.
- **Directional coverage.** A polar plot of the movement directions a configuration realises, with the exact gain and trial cost of mirroring the sequence or rotating the layout.
- **Trial budget.** Analysed movements per condition, with a warning when the endpoint sample is too small for a stable effective-width estimate.
- **Fit.** The pilot is blocked rather than silently rescaled if the largest condition does not fit the window.

### 3D mode

Selecting **3D spheres** places spherical targets on a fronto-parallel circle at a stated depth from the eye. Drag inside the preview to orbit the layout and inspect it off-axis; **Front view** returns to the frontal viewpoint that every reported value assumes. Visual angles for W and A′ are computed from the depth and appear in the condition table, the summary, the exported specification, and the Methods paragraph, together with an angular index of difficulty.

Pixels are disabled in 3D, since a pixel does not define a size in a scene.

**This is a perspective proxy, not an HMD.** It is useful for laying out and documenting a spatial configuration and for reasoning about depth and visual angle, but the pilot runs with a mouse on a flat display. Implement the task on your XR platform and report that implementation; the exported specification stays valid as the task definition.

## Exported files

All exports are generated in the browser. Every file carries the same 7-character `config_id`, so a reviewer can tell which specification, which trials, and which report belong together.

**`mdtt_config_*.json`** — the complete specification: `study`, `apparatus`, `task_space`, `display`, `reporting_unit`, `difficulty.conditions[]` (A, W, and ID in the reporting unit plus px, mm, degrees, and — in 3D — the angular ID), `geometry` (targets, shape, step, chord/diameter ratio, sequence rule, directions, phase, realized movement directions), `procedure`, `feedback`, and `analysis`. Reload it later with **Load config…** to reconstruct the task.

**`mdtt_trials_*.csv`** — one row per target-to-target movement:

| Column | Meaning |
|---|---|
| `config_id`, `session` | Configuration fingerprint and participant/session label |
| `seq`, `rep`, `condition`, `A`, `W`, `unit` | Sequence index, repetition, condition and its nominal parameters |
| `A_px`, `W_px`, `Arealized_px`, `ID` | The same parameters in pixels, plus the realized amplitude and Shannon ID |
| `direction`, `phase_deg`, `n_targets`, `step` | Sequence direction, layout rotation, and layout parameters |
| `movement`, `target_index`, `practice` | Position in the sequence, which target was cued, and whether the movement was dropped as practice |
| `from_x`, `from_y`, `target_x`, `target_y`, `select_x`, `select_y` | Start point (the preceding selection), target centre, and the recorded endpoint |
| `a_px`, `dx_px`, `dy_px` | Start-to-target distance and the endpoint deviation parallel and perpendicular to the movement axis |
| `MT_ms`, `hit`, `attempts` | Movement time, whether the first selection landed, and how many selections the movement took |

**`mdtt_summary_*.csv`** — one row per condition: `n`, `ID_bits`, `MT_ms`, `error_rate`, `We_px`, `Ae_px`, `IDe_bits`, `TP_bps`.

**`mdtt_report_*.md`** — the report sheet as Markdown; the same sheet prints to PDF from the Report tab. Its nine sections follow the paper's checklist, and the closing section lists whatever the configuration still leaves unspecified.

## How the measures are computed

- **Movement time** runs from the terminal selection of the preceding movement to the first selection of the current one.
- **Endpoint**: the *first* selection of every movement enters the distribution, hit or miss. Discarding misses would bias the endpoint spread that effective width is meant to capture.
- **Effective width** `We = 4.133 · SD(dx)`, where `dx` is the endpoint deviation projected onto the movement axis; **effective amplitude** `Ae = mean(a + dx)`; **effective difficulty** `IDe = log2(Ae/We + 1)`; **throughput** `TP = IDe / MT`, computed per condition.
- **Model fit**: MT regressed on ID, with intercept, slope and R² shown alongside throughput.

These are complementary, not interchangeable, which is what recommendation R6 asks you to preserve in the write-up: movement time without an accuracy measure does not describe the speed–accuracy trade-off, and throughput alone hides the separate contributions of speed and errors — particularly when the regression intercept is non-zero.

Pilot values produced here come from a single session and are meant to verify that a configuration runs, not to stand in for a study. For a real experiment, aggregate within participant and condition before comparing.

## Privacy

The tool runs entirely client-side. There is no server, no analytics, and no network request of any kind: configurations and trial data stay in the browser tab until you download them. This is usually the relevant fact for an ethics application.

## Repository layout

```
index.html    the complete testbed (single self-contained file)
README.md     this file
LICENSE       licence for the code
docs/         screenshots and example exports
```

Reporting the `config_id` and depositing the exported JSON alongside your data makes the task reconstructable by anyone reading the paper.

## Contributing

Issues and pull requests are welcome, particularly: XR-native implementations of the same specification, additional Fitts' law formulations, and reports of configurations where the geometry checks give a misleading result. Please include the exported configuration JSON when reporting a problem.

## Acknowledgements

This work was supported by National Research Foundation of Korea (NRF) grants funded by the Korea government (MSIT): RS-2024-00343882 and RS-2023-00242528.

## Licence

TODO — MIT is the usual choice for a tool like this; documentation and figures under CC BY 4.0.

---

<!-- MAINTAINER NOTES — delete this section before publishing

1. Rename mdtt_testbed.html to index.html at the repository root, then enable
   GitHub Pages (Settings > Pages > deploy from branch main, folder /root).
   The live link above only resolves once Pages is on.
2. ANONYMITY: the manuscript is set with the acmart 'anonymous' option, but
   Section 8 currently prints \url{https://github.com/xml1324/MDTT_Testbed} —
   a URL containing the author's GitHub handle. Either remove that URL for the
   review version or replace it with an anonymised mirror
   (anonymous.4open.science, or a fresh repository under a neutral account with
   no git history). In the mirror, also delete the author list from the BibTeX
   block below and the Acknowledgements section, which name the authors and
   their grants. Swap the real link back in at camera-ready.
3. Add LICENSE, fill in the DOI, and archive a release on Zenodo so the cited
   version is fixed rather than tracking main.
4. Add docs/screenshot-geometry.png and uncomment the image line near the top.
-->

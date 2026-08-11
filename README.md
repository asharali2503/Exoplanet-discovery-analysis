# Exoplanet Discovery Analysis

Two connected projects: a transit-detection pipeline that hunts for planets in raw TESS light curves from scratch, and a historical analysis of every confirmed exoplanet discovery since 1989, broken down by how each one was found.

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)](https://jupyter.org/)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Overview

The first half of this repository is a from-scratch pipeline for finding exoplanets in NASA's TESS (Transiting Exoplanet Survey Satellite) data: it pulls a star's raw light curve, cleans it, searches it for a periodic dip, and runs three independent checks to rule out the most common false positive — an eclipsing binary star masquerading as a planet.

The second half steps back from any single star and looks at the field as a whole: forty years of confirmed exoplanet discoveries from the NASA Exoplanet Archive, cleaned and grouped by detection method and year, which power an animated bar chart race showing how the dominant discovery technique has shifted over time.

## Repository Contents

| File | Description |
|---|---|
| `exoplanet_hunt.ipynb` | Transit-detection pipeline for a single target star: downloads and cleans a TESS light curve, runs a Box Least Squares period search, and verifies any candidate signal. |
| `exoplanetsByDiscovery(1).ipynb` | Cleans and aggregates NASA Exoplanet Archive records from 1989–2026, grouped by detection method and discovery year. |
| `exoplanet_years_formatted (1).csv` | The formatted output of the notebook above — the dataset behind the bar chart race. |

## Technical Highlights & Tools Used

- **Language:** Python, in Jupyter / Google Colab notebooks
- **Light curve access:** [`lightkurve`](https://docs.lightkurve.org/), which pulls TESS data directly from MAST
- **Numerical work:** `numpy`
- **Data cleaning & aggregation:** `pandas`
- **Transit search:** Box Least Squares (BLS) period search
- **Visualization:** [Flourish Studio](https://flourish.studio/) (bar chart race)

## Methodology & Verification

**1. Download and flatten.** `lightkurve` pulls the raw TESS light curve for the target star. Stellar variability and instrument drift are removed so what's left is a flat baseline with any transit dips still visible.

**2. Search for a periodic signal.** A Box Least Squares search tests roughly 20,000 candidate orbital periods, fitting a simple box-shaped dip to the light curve at each one and keeping whichever period, duration, and depth combination fits best.

**3. Rule out false positives.** Three checks run on the best candidate before it counts as a real signal:
   - **Phase-folding** — mapping every point in the light curve to its position within one orbital cycle and stacking all the cycles on top of each other. A genuine transit sharpens into one clean dip; noise that isn't periodic at that period smears out.
   - **Secondary eclipse test** — checking for a second, shallower dip roughly half an orbital period away, where a stellar companion would pass behind the primary star. Planets are too faint to produce this; a detectable secondary eclipse means the signal is probably a binary star, not a planet.
   - **Odd/even transit depth test** — comparing the average depth of odd-numbered transits against even-numbered ones. A real planet's transits are all the same depth; a mismatch usually means the true period is double what was detected and the signal is an eclipsing binary.

**4. Estimate the planet's size.** Once a signal passes verification, the transit depth converts directly into a radius estimate (see the formula below).

## Physics & Math Behind It

**Transit photometry.** As a planet crosses in front of its host star, it blocks a small, regular fraction of the star's light, producing a periodic dip in brightness — the entire pipeline is built around finding and measuring that dip.

**Relative transit depth.** The fraction of light blocked is approximately the ratio of the planet's cross-sectional area to the star's:

$$\frac{\Delta F}{F} \approx \left(\frac{R_p}{R_*}\right)^2$$

Rearranged, this gives a planet's radius directly from the transit depth and the star's known radius:

$$R_p \approx R_* \sqrt{\frac{\Delta F}{F}}$$

**Box Least Squares.** BLS is a period-search algorithm built specifically for transit-shaped signals: short, flat-bottomed, periodic dips. Instead of a generic frequency search, it fits a box function across a grid of candidate periods, durations, and phases, and keeps the combination that best matches the data — which makes it faster and more sensitive to transits than general-purpose period-finding methods.

**Why the verification tests matter.** Eclipsing binary stars can produce a dip that looks exactly like a small planet's transit. The secondary eclipse and odd/even tests exist because a planet and a binary star system predict different things at two very specific points — half a period away, and every other transit — so checking those two points is enough to tell them apart without needing an independent observation.

## Key Results

- Re-discovered **Pi Mensae c** — TESS's first confirmed exoplanet — directly from its raw light curve, running it through the full pipeline from download to radius estimate.
- Ran the same pipeline against **AU Mic** and **HD 63433**, two other TESS targets with independently confirmed planets, as a check that the detection and verification steps hold up on systems where the answer is already known.
- Modeled how the dominant exoplanet detection method has changed since 1989 — from radial velocity in the early years to transit photometry after Kepler and TESS — across the full 1989–2026 NASA Exoplanet Archive record.

## Future Work / Expansion

- Run the detection pipeline across a batch of TESS targets instead of one star at a time.
- Add a centroid-level check to catch background eclipsing binaries that the secondary eclipse and odd/even tests alone can miss.
- Cross-check calculated planet radii against the published values in the NASA Exoplanet Archive for hosts with already-confirmed planets.
- Extend the discovery-by-method dataset as new NASA Exoplanet Archive records are added past 2026.

## Visualization

**[Exoplanet Discoveries by Method (1989–2026) →](https://public.flourish.studio/visualisation/29824552/)**

<!-- Add a light curve plot or a screenshot of the bar chart race here -->

## Author

Ashar Ali

## License

This project is released under the MIT License — see [LICENSE](LICENSE) for details.

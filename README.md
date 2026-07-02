# Multiband AGN Variability & Reverberation Mapping in Dwarf Galaxies - Jon Berkson

**A WSU REU project applying Gaussian Process damped-random-walk (DRW) modeling with the goal of finding accreting black holes in dwarf galaxies (dwarf AGN).** 

## Overview
As a part of Washington State University's Waves in Physics 2026 REU summer program I am working with Professor Vivienne Baldassare and Erin Kimbro on analyzing the mid-IR (MIR) and optical photometric variability in a  sample of 25 dwaft galaxies. This repository contains the analysis pipeline for the project, with the twofold goals of (1) searching for a wavelength-dependent time lag between optical and mid-infrared emission, which is a signature of dust reprocessing that would be additional evidence for dwarf AGN, and (2) deriving a variability-based black hole mass estimate from each galaxy's light curves.

The sources as ALLWISE, NEOWISE, and Zwicky Transient Factory (ZTF), where WISE-derived sources use W1 (3.4 $\mu m$) and W2 (4.6 $\mu m$) band data. The chosen galaxies may host dwarf active galactic nuclei (dwarf AGN), for which there are many different sources of photometric variability. While MIR variability is thought to originate in the dusty torus, higher energy optical variability comes from the accretion disk of the dwarf AGN. So, for sources that have both MIR and optical variability, we seek to first determine if there is any correlation between the two. Then, if there is correlation, is there a measurable time lag? If so, then what can we learn about the central black hole and its' environment.

All model fitting is done with ['EzTaoX'](https://github.com/LSST-AGN-Variability/EzTaoX) (Yu et al. 2026), a JAX-based gaussian process framework built on tinygp, with posterior sampling done with numpyro's No-U-Turn Sampler, known as NUTS.

## Scientific Background
Intermediate-mass black holes (IMBHs), ranging in mass from ~ $10^2 - 10^5 \ M_{\odot}$ are thought to bridge the gap between stellar-mass and supermassive black holes, yet finding and measuring these black holes remains difficult. One of the most promising environments that host these elusive objects are dwarf galaxies. Even so, the light from these dwarf AGN are low-luminosity and diluted by star formation, in many cases making optical emission-line and X-ray diagnostics typically used to confirm accretion inadequate.

Variability selection offers a new alternative method; AGN accretion disks exhibit stochastic flickering that is well-described by a Damped Random Walk (DRW), a gaussian process with an exponential covariance kernel characterized by two main parameters:
- $\tau_{\text{DRW}}$ - the characteristic damping timescale (measured in days).
- $\sigma_{\text{DRW}}$ - the driving variability amplitude.

Previous work from Burke et al. 2021 noted that $\tau_{\text{DRW}}$ correlates with black hole mass. Thus, a measurement of the damping timescale alone can be converted into a variability-based mass estimate without the use of spectroscopy.

Furthermore, a second independent tool comes from infrared reverberation. The very continuum that varies stochastically also hets the surrounding dusty torus, which then reradiates in the mid-infrared with a time delay defined by the light-travel time to the region. Detecting a physically consistent lag, with IR lagging the optical by the right sign and a self-consistent amount across band-pairs, between ZTF optical sources and WISE mid-IR variability is a form of reverberation mapping and an additional strong signature of a dwarf AGN.

The 25 galaxies analyzed in this project are mid-IR-variable dwarf AGN candidates identifies by Aravindan et al. (2024) from over ten years of ALLWISE/NEOWISE photomotry. This project extends their analysis to every available optical and infrared light curve to both derive $\tau_{\text{DRW}}$ , $\sigma_{\text{DRW}}$ , and $M_{\text{BH}}$ estimates with posterior uncertainties, and test each galaxy for physically consistent lags.

## Data
 
| Band | Instrument | Wavelength | Baseline | Pre‑fit quality cuts |
|---|---|---|---|---|
| ZTF‑g | Zwicky Transient Facility | ~4783 Å | 2018–present | `err < 0.5 mag`, then iterative 3–5σ MAD sigma‑clip |
| ZTF‑r | Zwicky Transient Facility | ~6417 Å | 2018–present | `err < 0.5 mag`, then iterative 3–5σ MAD sigma‑clip |
| ALLWISE W1 | WISE (cryogenic) | 3.4 μm | 2010 only | Paper quality cuts (below) |
| ALLWISE W2 | WISE (cryogenic) | 4.6 μm | 2010 only | Paper quality cuts (below) |
| NEOWISE W1 | NEOWISE‑Reactivation | 3.4 μm | 2014–present | Paper quality cuts (below) |
| NEOWISE W2 | NEOWISE‑Reactivation | 4.6 μm | 2014–present | Paper quality cuts (below) |
 
**WISE quality cuts** (following Secrest & Satyapal 2020; Aravindan et al. 2024, §2.1):
`qual_frame ≥ 10`, `ph_qual ∈ {A, B}` in both W1 and W2 (SNR > 3), `nb == 1` (single‑PSF fit, no source blending), `na == 0` (no active deblending), `w2mpro < 14.5` (removes Eddington bias), followed by a second‑pass sigma‑clip.
 
Because ALLWISE (2010) predates both NEOWISE‑R (2014+) and ZTF (2018+) by several years, it has zero temporal overlap with any other band and is used only for single‑band τ_DRW/σ_DRW characterization.
 
## Methods

The following pipeline entails three stages, each of increasing model complexity using the same 'Exp' DRW kernel from eztaox.kernels.quasiep and fit via MLE random search for initialization followed by NUTS multi-chain monte-carlo (MCMC, 5 chains, 5500 steps, target accept 0.90) and diagnosed with ArviZ (r-hat, effective sample size, trace and pair plots).

### 1. Single-band DRW fits

Each band for sampled galaxies (up to four, two galaxies don't have ZTF-r data available) are fit independently to recover $\tau_{\text{DRW}}$ and $\sigma_{\text{DRW}}$ . These serve as the baseline characterization and the input to the per-band black hole mass estimate.

### 2. Pairwise two-band DRW + lag fits

Each band pair with temporal overlap, which is up to fifteen combinations for six individual bands, is fit jointly with a shared DRW process plus a per-pair time lag parameter using EzTaoX's MultiVarModel. The first band listed in each pair is the reference (lag = 0). Lag priors reflect the physically expected range, with lag expected to be concentrated closer to zero than out at the distribution limits:
- IR-IR or IR-optical pairs: TruncatedNormal(-300,300) centered at -7.5 days
- Purely optical pair (same accretion disk continuum): TruncatedNormal(-30,30) days

### 3. Joint multi-band fits (up to four bands)
Rather than fitting pairs in isolation, the four bands are fit simultaneously under one shared DRW Gaussian PRocess, with ZTF-g as the reference band. This is more principled than the pairwise approach given that a single physical process, in this case the accretion-disk continuum, should drive all bands at once, and the ZTF g-r lag then acts as an internal consistency check before the IR lags are trusted to be analyzed. The fit uses whichever bands pass the epoch-count and overlap requirements for a given galaxy to ensure physically realistic analysis.

### 4. Candidate ("interesting galaxy") selection 
A galaxy is noted as a candidate if, among its optical-IR pairs if the analysis is reasonable, defined by:
- Convergence: The r-hat being less than or equal to 1.1 on the lag posterior
- Correct sign: the IR lag is negative given dust echo lags the optical driving continuum
- Mutually consistent: independent "good" pairs agree with each other within a pairwise z-score threshold and are precise enough to be a real measurement rather than posteriors that happen to overlap.

### 5. Variability-based black hole mass estimates
$\tau_{\text{DRW}}$ is converted to a mass estimate using Burke et al. (2021)'s scaling relation $M_{\text{BH}} = 10^{7.97} M_\odot (\tau_{\text{DRW}} / \text{100 days})^{2.54}$ (Eqn. 2), computer per band from single-band $\tau_\text{DRW}$ and per galaxy from multi-band $\tau_\text{DRW}$, with only $\tau_\text{DRW}$'s own posterior uncertainty propigated. A per-galaxy single-band estimate is also calculated as the inverse-variance-weighted mean across all available bands. Comparison tables are generated and are available in the currently in-progress publication.

## Repository Contents

| Notebook | Purpose |
|---|---|
| `WSU_REU_EzTaoX_OneBand_Batch.ipynb` | Stage 1. Fits independent single‑band DRW models to every available band (ZTF‑g, ZTF‑r, NEOWISE‑W1, NEOWISE‑W2) for all 25 galaxies. Produces per‑band $\tau_\text{DRW}$ and $\sigma_\text{DRW}$ posteriors, diagnostic plots, and a running results CSV. |
| `WSU_REU_EzTaoX_MultiBand_Batch.ipynb` | Stages 2–4. Fits all 15 pairwise two‑band lag combinations and the joint 3–4‑band model for all 25 galaxies; generates summary heatmaps ( $\tau_\text{DRW}$ and lag) across the full galaxy-pair grid and applies the candidate‑selection criteria to flag "interesting" galaxies for closer inspection. |
| `WSU_REU_EzTaoX_MultiBand_Interesting.ipynb` | Stage 5 / deep‑dive. For the flagged candidate galaxies: generates lag‑shifted light‑curve overlay plots (optical and IR curves aligned by their fitted lag, for visual inspection) and produces the publication‑formatted LaTeX summary tables, including the black‑hole‑mass comparison table. |
 
All three notebooks share a common structure: imports and a colorblind‑safe plotting palette (Wong 2011), data‑loading/quality‑cut utilities, EzTaoX model‑builder functions, then a batch loop that is resumable (`SKIP_COMPLETED = True`) and crash‑safe (results are written to CSV after every individual fit).
 
```
.
├── WSU_REU_EzTaoX_OneBand_Batch.ipynb
├── WSU_REU_EzTaoX_MultiBand_Batch.ipynb
├── WSU_REU_EzTaoX_MultiBand_Interesting.ipynb
└── README.md
```
## Outputs
 
Each batch notebook writes a running results CSV after every individual fit (so no progress is ever lost to a crash), plus, per galaxy:
 
- Diagnostic DRW‑fit plots (`.png`) and posterior corner/trace plots (`.pdf`)
- ArviZ NetCDF posteriors (`.nc`) enables re‑plotting or table regeneration without refitting
- Per‑galaxy 2×2 (single‑band) or multi‑panel (multiband) summary galleries
- Sample‑wide summary heatmaps: $\log_{10}(\tau_\text{DRW})$ and interband lag, galaxy-band‑pair
- `interesting_galaxies.csv`, the candidate‑selection summary table
- Lag‑shifted light‑curve overlay plots for candidate galaxies
- Publication‑formatted LaTeX tables (single‑band parameters, pairwise DRW+lag, black‑hole‑mass comparison) 

## Previous Literature
[Baldassare+ 2020](https://iopscience.iop.org/article/10.3847/1538-4357/ab8936) investigated optical AGN variability in ~50,000 nearby (z < 0.055) dwarf galaxies, for which 70% are $< 10^{10} M_\odot$ using Palomar Transient Factory (PTF) R-band observations with average baselines of several years. They found that, among dwarf galaxies with AGN-like variability, around 75% have narrow emission lines dominated by star formation. This indicates that optical variability can reveal AGN misclassified by typical diagnostics. Additionally, they found that the measured AGN fraction is strongly baseline-dependent (with no trend with BH mass), where longer baselines increase variability detection fractions. They established a powerful and promising technique to be enhanced by ZTF and LSST.

[Secrest+ 2020](https://iopscience.iop.org/article/10.3847/1538-4357/ab9309) used 8.4 years of photometry from the ALLWISE and NEOWISE catalogs and compared the MIR variability of ~2200 dwarf galaxies ($M_\star <2\times10^9 \ h^{-2} \ M_\odot$) and ~6600 larger galaxies ($M_\star \geq 10^{10} \ h^{-2} \  M_\odot$). They found that variability misses most independently identified dwarf AGN (1/10 AGN-like MIR colors determined variable, and none of the 86 optically classified dwarf AGN determined variable), which was determined to be statistically significant. With this, they concluded that optical spectroscopy and MIR color selection are more effective than low-cadance MIR variablility for finding dwarf AGN, which they said is likely due to a long baseline and sparse cadance.

[Aravindan+ 2024](https://iopscience.iop.org/article/10.3847/1538-4357/ad702b/meta) used 10.4 years of photometry from ALLWISE AND NEOWISE catalogs and identified 25 dwarf galaxies showing AGN-like MIR variability. They found that most (68%) of the MIR-variable sample was confirmed as AGN activity from optical and NIR diagnostics. Additionally, they found that, despite similar W1-W2 colors, variable dwarfs had bluer W2-W3 colors and shower a lower contribution from dust heated by star formation in the MIR, meaning that AGN is the primary candidate for the observed redder W1-W2 colors in the variable sample, while high SFR drives redder color in galaxies without observed variabilities. Finally, they found a possible strong connection between MIR variability and the presence of broad-line dwarf AGN (larger fraction of MIR-variable targets had FWHM > 500 km/s).

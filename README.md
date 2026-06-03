# WSU AGN Variability REU - Jon Berkson

## Overview
As a part of Washington State University's Waves in Physics 2026 REU summer program I am working with Professor Vivienne Baldassare and Erin Kimbro on analyzing the mid-IR (MIR) and optical photometric variability in a  sample of 25 dwaft galaxies. The sources as ALLWISE, NEOWISE, and Zwicky Transient Factory (ZTF), where WISE-derived sources use W1 (3.4 $\mu m$) and W2 (4.6 $\mu m$) band data. The chosen galaxies may host dwarf active galactic nuclei (dwarf AGN), for which there are many different sources of photometric variability. While MIR variability is thought to originate in the dusty torus, higher energy optical variability comes from the accretion disk of the dwarf AGN. So, for sources that have both MIR and optical variability, we seek to first determine if there is any correlation between the two. Then, if there is correlation, is there a measurable time lag? If so, then what can said time lag tell us about the host galaxy. For example, is the torus clumpy or smooth?

## Previous Literature
[Baldassare+ 2020](https://iopscience.iop.org/article/10.3847/1538-4357/ab8936) investigated optical AGN variability in ~50,000 nearby (z < 0.055) dwarf galaxies, for which 70% are $< 10^{10} M_\odot$ using Palomar Transient Factory (PTF) R-band observations with average baselines of several years. They found that, among dwarf galaxies with AGN-like variability, around 75% have narrow emission lines dominated by star formation. This indicates that optical variability can reveal AGN misclassified by typical diagnostics. Additionally, they found that the measured AGN fraction is strongly baseline-dependent (with no trend with BH mass), where longer baselines increase variability detection fractions. They established a powerful and promising technique to be enhanced by ZTF and LSST.

[Secrest+ 2020](https://iopscience.iop.org/article/10.3847/1538-4357/ab9309) used 8.4 years of photometry from the ALLWISE and NEOWISE catalogs and compared the MIR variability of ~2200 dwarf galaxies ($M_\star <2\times10^9 \ h^{-2} \ M_\odot$) and ~6600 larger galaxies ($M_\star \geq 10^{10} \ h^{-2} \  M_\odot$). They found that variability misses most independently identified dwarf AGN (1/10 AGN-like MIR colors determined variable, and none of the 86 optically classified dwarf AGN determined variable), which was determined to be statistically significant. With this, they concluded that optical spectroscopy and MIR color selection are more effective than low-cadance MIR variablility for finding dwarf AGN, which they said is likely due to a long baseline and sparse cadance.

[Aravindan+ 2024](https://iopscience.iop.org/article/10.3847/1538-4357/ad702b/meta) used 10.4 years of photometry from ALLWISE AND NEOWISE catalogs and identified 25 dwarf galaxies showing AGN-like MIR variability. They found that most (68%) of the MIR-variable sample was confirmed as AGN activity from optical and NIR diagnostics. Additionally, they found that, despite similar W1-W2 colors, variable dwarfs had bluer W2-W3 colors and shower a lower contribution from dust heated by star formation in the MIR, meaning that AGN is the primary candidate for the observed redder W1-W2 colors in the variable sample, while high SFR drives redder color in galaxies without observed variabilities. Finally, they found a possible strong connection between MIR variability and the presence of broad-line dwarf AGN (larger fraction of MIR-variable targets had FWHM > 500 km/s).

## My Work
As where I come in, the first step in my project is the apply data cuts described in [Aravindan+ 2024](https://iopscience.iop.org/article/10.3847/1538-4357/ad702b/meta) and plot light curves for all galaxies in my sample. Given that ALLWISE, NEOWISE, and ZTF sources have slightly different methods of storing similar data, I had to write slightly different data extraction and plotting code for each source, which are provided in the .py files. Please note that my code is meant for running within a Jupyter Notebook file, but should work for other methods. Additionally, my font of choice for plotting is Avenir, and I use the color palette described in [Wong 2011](https://www.nature.com/articles/nmeth.1618), which is optimized for color-blind individuals.

### ALLWISE Handling
The ALLWISE Handling code has multiple different functions, all of which are described with comments throughout the code. The plot code presents the W1 magnitude, W2 magnitude, and the W1-W2 color, along with colored points representing the mean observation with associated error bars for each 10-day bin. It is common for ALLWISE observations to be separated by a significant amount of time, so a subplot_gap variable has been hard coded in such that if the gap between observations is greater than 100 days in MJD format, multiple columns of subplots are created.

Example usage with plotted figure:

BASE_DIR  = "base directory"
GALAXY_ID = "NSA10045"

allwise_data = load_ALLWISE_galaxy(BASE_DIR, GALAXY_ID)

fig, ax = plot_ALLWISE_light_curve(allwise_data)

<img width="699" height="701" alt="NSA10045_allwise" src="https://github.com/user-attachments/assets/c48e8092-cfd6-4535-a057-46009eadb41a" />

### NEOWISE Handling
The NEOWISE Handling code has multiple different functions, all of which are described with comments throughout the code. The plot code presents the W1 magnitude, W2 magnitude, and the W1-W2 color, along with colored points representing the mean observation with associated error bars for each 10-day bin. 

Example usage with plotted figure:

BASE_DIR  = "base directory"
GALAXY_ID = "NSA10045"

raw_data = load_NEOWISE_galaxy(BASE_DIR, GALAXY_ID)

clean_data = filter_NEOWISE(raw_data)

fig, ax = plot_NEOWISE_light_curve(clean_data)

<img width="784" height="701" alt="NSA10045_neowise" src="https://github.com/user-attachments/assets/3f3cc283-2d3c-4029-a448-9084cd5e1307" />

### ZTF Handling
The ZTF Handling code has multiple different functions, all of which are described with comments throughout the code. The plot code presents the ZTF g-band magnitude and ZTF r-band magnitude in the same figure, along with assocated error bars for each observation.

Example usage with plotted figure:

BASE_DIR  = "base directory"
GALAXY_ID = "NSA10045"

galaxy_data = load_ztf_galaxy(BASE_DIR, GALAXY_ID, bands=("g", "r"))

fig, ax = plot_light_curve(galaxy_data)

<img width="1089" height="490" alt="NSA10045_ztf" src="https://github.com/user-attachments/assets/6068ae97-787d-4f50-a22f-3e11c7b10dd3" />


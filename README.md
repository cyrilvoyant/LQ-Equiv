# LQL-Equiv

Open-source MATLAB software for **biologically equivalent dose calculation in radiotherapy**, based on the linear-quadratic-linear (LQL) model. It computes the biological equivalent dose (BED), the dose equivalent to a 2 Gy/fraction reference schedule (EQD2), the normal tissue complication probability (NTCP, Lyman model), and the estimated radiation-induced cancer risk (UNSCEAR model) — with corrections for incomplete inter-fraction repair, tissue/tumor repopulation, and treatment protraction (days off).

Developed by Cyril Voyant and Daniel Julian, following the methodology of Astrahan (LQL model), Dale (repopulation), and Thames (incomplete repair). See [Citation](#citation) for the associated peer-reviewed paper.

## License

**This repository is MIT-licensed** (see [`LICENSE`](LICENSE)) — that is the authoritative license for this code going forward. Two other sources describe the license differently and are both superseded by the `LICENSE` file here: the original 2013 paper states the tool was released "under the GNU license", and the software's Zenodo deposit currently lists CeCILL. If you obtained this code from Zenodo or read the paper, disregard those license statements for this specific repository — MIT applies.

## Contents

- **`LQEquiv.pdf`** — the associated methodology paper (Voyant, Julian, Roustit, Biffi, Lantieri, *Reports of Practical Oncology and Radiotherapy*, 2014).
- **`LQL-Equiv.exe`** — compiled Windows standalone executable (no MATLAB license needed to run, see [Running the executable](#running-the-executable)).
- **`codesource.zip`** — MATLAB source: `eqd_matlb.m` (GUIDE-generated GUI, all the calculation logic) and `eqd_matlb.fig` (the GUI layout, MATLAB GUIDE figure file).
- **`CITATION.cff`**, **`schema.json`** — citation metadata (software + associated paper).
- **`LICENSE`** — MIT.
- **`contributing.md`**, **`cy.yml`** — contribution guidelines and a GitHub Actions Markdown/YAML/JSON/HTML linter.
- **`index.html`** — a GitHub Pages preview page (PDF viewer + download links).

**Note on `heaviside.m`:** earlier versions of `codesource.zip` bundled a copy of MathWorks' own `heaviside.m` (Symbolic Math Toolbox source, `Copyright 1993-2008 The MathWorks, Inc.`), which is not freely redistributable outside MATLAB. It has been removed from the archive. **Running `eqd_matlb.m` from source requires MATLAB's Symbolic Math Toolbox**, which provides its own `heaviside` function (used for the repopulation kick-off-time correction, see Eq. 3-4 below). The precompiled `LQL-Equiv.exe` is unaffected — it was built with the toolbox available and does not need it reinstalled by the end user.

## Running the executable

`LQL-Equiv.exe` is a **compiled Windows binary — verify its checksum before running it**, especially if you did not build it yourself:

```
SHA256: e95f40efc06d9c322474b1ee9c652aec263ff81358092488f6cba4d5003a52a5
```

Per the original paper, it was built with MATLAB GUIDE (32-bit MATLAB v7.12) and the MATLAB Compiler (v4.12), and therefore requires the **MATLAB Component Runtime (MCR), 32-bit, version 7.15 or later**, installed separately before the executable will run (download from MathWorks; the exact 32-bit MCR release matching MATLAB R2011a-era compiled applications). This is not bundled in this repository. Modern 64-bit-only Windows setups may require running the 32-bit MCR under WoW64 compatibility; this has not been verified by this repository's maintainers on current Windows versions.

Running the source directly in MATLAB (`codesource.zip`) avoids the MCR/32-bit constraint but requires a MATLAB license and the Symbolic Math Toolbox (see note above).

## Methodology

All formulas below reproduce the 2014 RPOR paper (see [Citation](#citation)); the implementation notes describe what `eqd_matlb.m` actually computes, including where it departs from a literal reading of the paper.

### 1. Biological Equivalent Dose (BED)

BED is the standard radiobiological currency for comparing treatments with different total dose, fractionation, and overall duration: two treatments with the same BED are assumed to produce the same biological effect. For $n$ fractions of dose $d$ (Gy) and a tissue with radiosensitivity ratio $\alpha/\beta$, the plain linear-quadratic BED is $n\,d\,(1+d/(\alpha/\beta))$. The LQL model extends this in two regimes, split at a tissue-specific transition dose per fraction $d_t$ beyond which the survival curve becomes linear (Astrahan).

**Target volumes (tumor), $d>d_t$ (Eq. 3):**

$$\mathrm{BED}=n\Big[d_t\Big(1+\frac{d_t}{\alpha/\beta}\Big)+(d-d_t)\Big]-\Theta(T-T_k)\,\frac{\ln 2}{\alpha\,T_{pot}}(T-T_k)$$

**Target volumes, $d\le d_t$ (Eq. 4):**

$$\mathrm{BED}=n\,d\Big(1+(1+H_m)\frac{d}{\alpha/\beta}\Big)-\Theta(T-T_k)\,\frac{\ln 2}{\alpha\,T_{pot}}(T-T_k)$$

where $\Theta$ is the Heaviside step function, $T$ the overall treatment time (days), $T_k$ the repopulation kick-off time (days, tumor considered non-proliferative/hypoxic before this), and $T_{pot}$ the potential doubling time. $H_m$ (Eq. 5) is Thames' incomplete-repair correction for more than one fraction per day:

$$H_m=\frac{2\varphi}{m}\Big[m-\frac{1-\varphi^m}{1-\varphi}\Big]\Big(\frac{1}{1-\varphi}\Big),\qquad \varphi=e^{-\mu T'}$$

with $m$ the number of fractions per day and $T'$ the inter-fraction interval; $H_m=0$ for one fraction per day. The code (`eqd_matlb.m`) fixes the repair half-time-derived $\varphi$ from a 6-hour minimum inter-fraction interval, i.e. `phi = exp(-0.693*6/T12)`.

**Organs at risk, $d>d_t$ (Eq. 6) and $d\le d_t$ (Eq. 7):** identical structure, but the kick-off time is not applicable (repopulation is not the limiting factor for late-responding normal tissue); instead a recovered dose $D_{rec}=\ln(2)/\alpha \cdot T_{pot}$ (Gy/day) is subtracted proportionally to overall time $T$:

$$\mathrm{BED}=n\Big[d_t\Big(1+\frac{d_t}{\alpha/\beta}\Big)+(d-d_t)\Big]-D_{rec}\,T \qquad(d>d_t)$$
$$\mathrm{BED}=n\,d\Big(1+(1+H_m)\frac{d}{\alpha/\beta}\Big)-D_{rec}\,T \qquad(d\le d_t)$$

### 2. Equivalent dose (EQD2) — computed by cost-function minimization, not a closed form

The plain-LQ closed-form EQD2 (Eq. 2, valid only without repopulation/protraction corrections) is:

$$\mathrm{EQD2}=D\,\frac{d+(\alpha/\beta)}{2+(\alpha/\beta)}$$

Once repopulation and incomplete repair are included, BED no longer reduces to a simple algebraic relation between two treatments' total doses. The paper's (and the code's) actual approach is to define a cost function comparing the BED of the treatment under evaluation ($n,d,\text{days off}$) against a reference schedule ($n_{ref}, d_{ref}=2\,\text{Gy}$):

$$f(n_{ref}) = \big|\,\mathrm{BED}(n_{ref}, 2\,\text{Gy}) - \mathrm{BED}(n, d)\,\big|,\qquad n_0=\arg\min_{n_{ref}\in\mathbb R^+} f(n_{ref}),\qquad \mathrm{EQD2}=2\,n_0$$

**Implementation note:** `eqd_matlb.m` does not use a generic optimizer for this minimization. It performs an exhaustive **grid search over `nf1 = 0:10000`** (in units of 0.01 fraction, i.e. 0 to 100 reference fractions at 0.01-fraction resolution) computing the candidate BED at every point and keeping the minimizer of `|BED_target - BED_candidate|` — a brute-force line search rather than a root-finder or gradient method. For the tumor side this range is extended to `nft1 = -10000:10000` to allow the search to explore an under-shoot region.

### 3. Overall treatment time and gap compensation

Overall treatment time $T$ (days) is derived from the number of fractions assuming a standard **5-fractions-per-week** schedule; the code implements this via a stepwise lookup (adding 2 calendar days per completed week, i.e. `T ≈ floor(n_fractions * 7/5)` beyond simple cases) both for the BED overall-time term and, separately, in the GUI's live "gap compensation" fields (`radiobutton1-4` callbacks) that let a user enter a number of treatment days off (`ja`) and see the recommended extra-fraction compensation. Per the paper's usage instructions, **only weekday interruptions should be entered as gap days — weekends should not be counted**, and the model is stated to be invalid beyond roughly 20 days off treatment.

### 4. Normal Tissue Complication Probability (NTCP) — Lyman model

$$\mathrm{NTCP}(n,d,\text{days off})=\frac{1}{\sqrt{2\pi}}\int_{-\infty}^{u}e^{-t^2/2}\,dt,\qquad u=\frac{\mathrm{EQD2}(n,d,\text{days off})-\mathrm{TD}_{50}}{m\cdot\mathrm{TD}_{50}}$$

$\mathrm{TD}_{50}$ is the tissue-specific dose giving 50% complication probability, and $m$ the slope parameter of the logistic/probit response. In this implementation EQD2 (not the more general EUD) is used as the input dose, following the paper's stated simplification for practical use.

### 5. Radiation-induced cancer risk (UNSCEAR model)

$$K_{\text{incidence}}=P_{\text{UNSC}}\cdot D_{2\text{Gy}}\cdot e^{-\text{UNSC}\cdot D_{2\text{Gy}}}$$

where $P_{\text{UNSC}}$ and $\text{UNSC}$ are organ-specific parameters derived from UNSCEAR meta-analyses of radiation-induced cancer incidence, and $D_{2\text{Gy}}$ the equivalent dose at 2 Gy/fraction.

### 6. Built-in radiobiological parameter library

`eqd_matlb.m` hardcodes the radiobiological parameters ($\alpha/\beta$, $\alpha$, $T_{1/2}$, $T_k$, $T_p$, $D_{prol}$, $d_t$, $m$, $\mathrm{TD}_{50}$, $P_{\text{UNSC}}$/UNSC where applicable) for roughly **35 organ-at-risk endpoints** and **20 tumor-control endpoints**, selectable from the GUI's two dropdown menus. These values are the software's clinical content; this repository does not carry a separate per-endpoint literature reference beyond the general citations in the associated paper — treat them as the paper's published defaults, and verify against current literature/local protocols before clinical use (see [Disclaimer](#disclaimer)).

## Citation

If you use this software, please cite the software itself (`CITATION.cff`) and, in academic work, the methodology paper:

> C. Voyant, D. Julian, R. Roustit, K. Biffi, C. Lantieri, "Biological effects and equivalent doses in radiotherapy: A software solution," *Reports of Practical Oncology and Radiotherapy*, vol. 19, no. 1, pp. 47-55, 2014. https://doi.org/10.1016/j.rpor.2013.08.004

Software DOI: [10.5281/zenodo.16739883](https://doi.org/10.5281/zenodo.16739883)

## Disclaimer

Per the associated paper: this software is intended as a **secondary/assistance calculator**, not a substitute for validated routine clinical BED/EQD2 calculations performed and countersigned by a qualified medical physicist and physician. The authors cannot be held responsible for errors caused by misuse of the results. Verify outputs against your center's standard tools and the literature before any clinical use.

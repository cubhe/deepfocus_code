# DeepFocus

**Code and computational resources for deep-tissue focusing through scattering brain tissue with binary-amplitude wavefront modulation and two-photon fluorescence feedback.**

[![Status](https://img.shields.io/badge/status-code%20release%20pending-orange)](#release-status)
[![Paper](https://img.shields.io/badge/paper-in%20preparation-blue)](#citation)
[![License](https://img.shields.io/badge/license-TBD-lightgrey)](#license)

---

## Release status

> **The source code has not been released yet.**
>
> This repository is the designated code archive for the manuscript listed under [Citation](#citation). It currently contains documentation only. The implementation — simulation framework, optimization algorithms, instrument control, and figure-generation pipelines — will be deposited here at the time of publication, and this README will be updated with the corresponding release tag and DOI.
>
> If you need access before then (e.g., as a reviewer or collaborator), please see [Contact](#contact).

---

## Overview

Optical focusing deep inside biological tissue is limited by multiple scattering: beyond roughly one transport mean free path, a nominally diffraction-limited focus degrades into a speckle field, and two-photon excitation efficiency collapses with depth. Wavefront shaping recovers a focus by pre-compensating the tissue-induced field distortion, but the achievable depth is set by how much information can be extracted from a weak, non-linear, spatially integrated feedback signal.

**DeepFocus** studies this problem in the regime that is most relevant to *in vivo* two-photon microscopy:

- **Control:** binary-amplitude modulation at the pupil (back focal) plane using a digital micromirror device (DMD), i.e. each controlled channel is either open or closed, $u_i = m_i \in \\{0,1\\}$.
- **Feedback:** a single-pixel (bucket) two-photon fluorescence signal integrated over the excitation volume,

  $$y(\mathbf{m}) \;=\; \int \rho(\mathbf{r})\,\Big|\textstyle\sum_i m_i\, t_i(\mathbf{r})\Big|^4 \, \mathrm{d}\mathbf{r},$$

  where $t_i(\mathbf{r})$ is the complex response of channel $i$ and $\rho(\mathbf{r})$ the fluorophore distribution.

This combination — binary control with a quartic, spatially integrated readout — is fast and photon-efficient, but carries less per-measurement information than phase-stepping adaptive optics or camera-based transmission-matrix measurement. The project therefore addresses two coupled questions:

1. **Where is the optimum?** Given the field that would be recovered by reverse propagation from the target, what is the structure of the optimal binary mask?
2. **How is it reached?** When the field is unknown and only fluorescence feedback is available, which optimization problem is actually being solved, and which algorithm class is appropriate under a realistic photon budget?

The repository brings together the wave-optics simulator, the optimization algorithms, the microscope control and analysis code, and the scripts that generate the figures reported in the manuscript.

---

## Scientific scope

### 1. Forward and reverse wave-optics model

A GPU-accelerated split-step Fresnel (beam-propagation) engine models excitation through a three-dimensional refractive-index volume representing cortical tissue:

- Pupil-plane modulation (binary amplitude, phase-only, or full complex) followed by 4*f* / Fresnel propagation to the focal region.
- Statistical brain refractive-index volumes with separately controlled micro- and macro-scale correlation lengths, depth-dependent index contrast, and calibration of the resulting scattering strength against a target effective attenuation length, so that simulated signal decay matches experimentally reported depth dependence.
- Reverse propagation from a point source at the target to the pupil plane, yielding the complex "distorted focusing pattern" that serves as ground truth for the achievable correction.
- Default configuration for a 1035 nm excitation wavelength, with focal depths swept from 0.1 mm to beyond 1.3 mm.

### 2. Theory of the optimal binary mask

An analytical component characterizes the optimum rather than only searching for it:

- Signal-level comparison between binary-amplitude FOCUS-type probing and phase-stepping adaptive optics, in terms of the mean and variance of the detected feedback under random modulation — the quantity that governs usable contrast in the low-photon regime.
- An aligned-phasor formulation showing that the optimal mask is not an arbitrary combinatorial object: for a given reference phase, the optimum is the set of channels whose complex coefficients project most strongly onto that direction, i.e. a ranked support-selection problem.
- The classical continuous phase-window result, including the uniform-case optimal open fraction $\eta^\* \approx 0.371$, recovered as a special case, together with the amplitude-weighted and non-uniform extensions.

### 3. Mask-optimization algorithms

Algorithms are organized by *information regime*, and benchmarked against one another under matched measurement budgets:

| Regime | Method | Idea |
|---|---|---|
| Field known | Deterministic top-$k$ / reference-phase solver | Rank channels by projection onto the optimal reference phase; sweep the open fraction. |
| Field known | Phase-window and amplitude-percentile sweeps | Continuous-window baselines, including center-expanding phase selection. |
| Offline, random probing | Compressive sensing with total-variation regularization | Recover a latent score map from random binary masks and their fluorescence responses, then binarize. |
| Offline, random probing | Robust CS variant | Response centering, robust label transforms, soft open-fraction constraints, and early stopping on propagated reward. |
| Online, black box | REINFORCE and antithetic REINFORCE | Directly maximize the measured signal over a stochastic mask policy; amplitude, phase, or joint parameterization. |
| Online, black box | Multi-resolution tree search | Coarse-to-fine refinement with equal-count local exchange moves. |
| Surrogate | Learned differentiable predictor | Fit a model of the measurement response, then optimize the mask by gradient descent. |

Diagnostics compare learned masks against the reverse-propagated field (overlap, phase histograms), and depth sweeps report the input energy required to reach a fixed focal intensity for each method — the operationally meaningful figure of merit for *in vivo* imaging.

### 4. Experimental control and image analysis

- MATLAB control and acquisition code for the DMD- and galvanometer-based two-photon microscope: pattern generation and projection, scan synchronization, region-of-interest feedback readout, and correction-cycle execution.
- Python analysis for mask generation from measured responses, reconstruction of the open-channel support, and quantification of correction gain.
- Post-processing of *in vivo* stacks used for the figures: axon tracing and interpolation for neuronal structure, vesselness filtering and skeletonization for vasculature, stack alignment across acquisition blocks, and volumetric rendering.

---

## Planned repository structure

The layout below is the structure intended for the initial release and may be refined before deposit.

```
deepfocus_code/
├── simulation/          # Fresnel/BPM engine, tissue index model, workflow entry points
│   ├── deepfocus/       # Core package: propagation, DMD model, fluorescence model, workflows
│   └── configs/         # JSON configuration files, one per reported experiment
├── optimization/        # Top-k solver, CS + TV, REINFORCE, tree search, surrogate model
├── experiment/          # MATLAB instrument control and acquisition; Python mask generation
├── analysis/            # Reconstruction, quantification, and figure-generation scripts
├── docs/                # Method notes and parameter tables
└── examples/            # Minimal runnable examples ("smoke" configurations)
```

Large binary artifacts — raw stacks, generated mask datasets, refractive-index volumes, and rendered videos — are deliberately excluded from version control; see [Data availability](#data-availability).

---

## Planned requirements

- Python ≥ 3.10 with PyTorch (CUDA strongly recommended; the depth sweeps assume a GPU with ≥ 24 GB of memory), NumPy, SciPy, scikit-image, Matplotlib, h5py.
- MATLAB (instrument control only) with the vendor SDKs for the DMD and scanning hardware.
- Exact pinned versions will accompany the release as an environment specification.

Every reported configuration is defined by a JSON file, so each figure in the manuscript corresponds to a named configuration that can be re-run with a single command.

---

## Data availability

Simulated refractive-index volumes and random-mask datasets are regenerated deterministically from the seeds recorded in the configuration files. Experimental datasets underlying the reported figures will be made available through a public archive; the record identifier will be added here at publication.

---

## Citation

If you use this work, please cite the associated publication:

```bibtex
@article{deepfocus,
  title   = {TBD},
  author  = {TBD},
  journal = {TBD},
  year    = {TBD},
  doi     = {TBD}
}
```

<!-- Replace the fields above with the final title, author list, venue, year, and DOI at publication.
     Add a software DOI (e.g., Zenodo) for the tagged code release if one is minted. -->

Please also cite the prior experimental work on compressive-sensing-based focusing (C-FOCUS) where appropriate; references are listed in the manuscript.

---

## License

To be determined before the code release. The intended terms are a permissive open-source license for the source code and a Creative Commons license for the documentation; the final `LICENSE` file will be added together with the first code commit.

---

## Contact

Renzhi He — [cobilab@ucdavis.edu](mailto:cobilab@ucdavis.edu)
Department of Biomedical Engineering, University of California, Davis

For questions about the method, requests for pre-publication access, or reports of problems with the released code, please open an issue in this repository or contact the address above.

---

## Acknowledgments

Funding sources, facility support, and individual acknowledgments will be listed here in accordance with the acknowledgments section of the manuscript.

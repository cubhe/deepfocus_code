# DeepFocus

Code for deep-tissue focusing through scattering brain tissue using binary-amplitude wavefront modulation (DMD) with two-photon fluorescence feedback.

## Status

**The source code has not been released yet.** This repository is the code archive for the manuscript below; the implementation will be deposited here at publication.

## Scope

On release, this repository will contain:

- **Simulation** — GPU split-step Fresnel propagation through 3D refractive-index models of cortical tissue, with forward and reverse (point-source-to-pupil) propagation.
- **Theory** — characterization of the optimal binary mask as a ranked support-selection problem, with the continuous phase-window result as a special case.
- **Optimization** — deterministic top-*k* solver, compressive sensing with TV regularization, REINFORCE, multi-resolution tree search, and surrogate-model methods, benchmarked under matched measurement budgets.
- **Experiment** — DMD and scanning control code for the two-photon microscope, plus the analysis and figure-generation scripts used in the manuscript.

Large binary data (stacks, mask datasets, index volumes) are excluded from version control; simulated data are regenerated from seeds recorded in the configuration files.

## Citation

```bibtex
@article{deepfocus,
  title   = {TBD},
  author  = {TBD},
  journal = {TBD},
  year    = {TBD},
  doi     = {TBD}
}
```

## License

To be added with the code release.

## Contact

Renzhi He — [cobilab@ucdavis.edu](mailto:cobilab@ucdavis.edu)
Department of Biomedical Engineering, University of California, Davis

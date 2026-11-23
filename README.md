# Grouped Deletion Bounds for Certifying TLS Optimality in Robot-World Hand-Eye Calibration

**Jaeyoung Bae · Byeongjun Kim · Kwangki Kim — Inha University**

[Project website](https://i-m-bedded.github.io/rwhec-tls-certification/) · [Manuscript](docs/assets/paper.pdf) · [Real-data reproduction](real44data/README.md) · [Upstream code](https://github.com/bae1224/Group-Deletion-Bounds-TLS-for-Certifying-RWHEC-Calibration)

This research release accompanies the supplied manuscript. It contains the 44-observation real-data experiment, fixed calibration candidates, LS SDP implementation, singleton/pairwise deletion bounds, and cascade certification. It also includes a project website with an annotated experimental photograph and a 4× data-acquisition video.

## Research idea

A least-squares (LS) optimum need not minimize a truncated least-squares (TLS) objective. For a feasible full-support candidate, we reuse an LS dual solution to bound the benefit of deleting observations. Singleton and pairwise spectral bounds yield sufficient numerical certification conditions without enumerating deletion sets. A cascade tests singleton first and computes pairwise bounds only when needed.

The real-data study fixes the candidate and weights: pairwise and cascade certify 16 tested cutoff settings from 3.30 to 4.05 where singleton is inconclusive. The paper also reports broad-motion and limited-motion synthetic experiments; their reproduction code is not included in the upstream package.

## Coordinate convention: AᵢX = YBᵢ

`a T b` maps coordinates expressed in frame b into frame a.

| Symbol | Notation | Meaning |
| --- | --- | --- |
| Aᵢ | base T hand,i | Observed robot hand pose from URDF forward kinematics |
| X | hand T camera | Fixed hand–camera transform to estimate |
| Y | base T board | Fixed base–board transform to estimate |
| Bᵢ | board T camera,i | Inverse of the observed camera T board pose |

CSV columns are `qw,qx,qy,qz,tx,ty,tz`; translation is in metres. The hand frame is the URDF tip frame. Image/robot pairing is defined by sample IDs and the manifest, never inferred from image filenames.

## Quick start: reproduce the fixed-candidate certificate

Run from the repository root on Linux x86_64. Requirements: Python 3.10+, Julia 1.10.10. Julia dependencies are pinned in `environment/Manifest.toml`.

```bash
python3 -m venv real44data/.venv
real44data/.venv/bin/python -m pip install -r real44data/requirements.txt
bash scripts/install_julia_rb5.sh
real44data/.venv/bin/python real44data/prepare.py --verify-only
RWHE_PYTHON=real44data/.venv/bin/python bash real44data/reproduce.sh
```

Outputs go to `real44data/.work/reproduction/`. Use a fresh output directory for subsequent runs. The protocol checks 75 cutoffs, performs 324 timing calls, independently reconstructs costs/bounds with NumPy, and compares decisions against the released references. Stored-data reproduction requires neither ROS nor connected hardware.

## Reconstruct poses and estimate X/Y from images

```bash
real44data/.venv/bin/python -m pip install -r hardware/requirements-gopro.txt
real44data/.venv/bin/python -m hardware.rb5_gopro \
  --config hardware/rb5.gopro.json import \
  --manifest real44data/data/manifest.json \
  --output real44data/.work/from_images
bash scripts/julia_rb5.sh scripts/calibrate_rb5.jl \
  real44data/.work/from_images/combined \
  real44data/.work/solution eye_in_hand
```

This estimates a new solution. The TLS reproduction script continues to use the frozen paper candidates in `real44data/input`; it does not automatically certify newly estimated X/Y.

## Contents

| Directory | Purpose |
| --- | --- |
| `docs/` | Static project website, annotated setup, manuscript and 4× acquisition video |
| `real44data/data/` | 44 original images, robot records, poses and pairing manifest |
| `real44data/input/` | Frozen A/B/X/Y used for paper certification |
| `real44data/lib/` | Numerical certification implementation |
| `real44data/reference/` | Released reference results and plots |
| `hardware/` | Pose extraction, intrinsics, robot FK and acquisition utilities |
| `scripts/`, `src/`, `environment/` | Julia solver, constraints and pinned environment |

The original 64 observations were filtered using image quality criteria (19 reprojection-RMS exclusions and one board-area exclusion), yielding 44 pairs. This filtering is not an RWHE outlier classification.

## Interpretation and reproducibility limits

Certification is a floating-point sufficient condition for the stated numerical objective, with the full-support and gap conditions specified in the manuscript. It does not measure absolute physical accuracy; an inconclusive result does not establish suboptimality. The 75 cutoffs are sensitivity settings on one dataset, not 75 independent experiments. Runtime varies by machine and should be reported separately from the released reference timings.

The supplied PDF is a manuscript; no publication venue, DOI or acceptance status is asserted here. The public project website is served by GitHub Pages from the main branch, /docs folder.

## Provenance and license

This public export preserves the upstream MIT license and Jaeyoung Bae's copyright notice. Third-party model and solver provenance remain in `UPSTREAM.json`, `real44data/provenance.json`, and the model license files. See [public-release notes](PUBLIC_RELEASE.md) for metadata sanitation and hash regeneration.

```bibtex
@unpublished{bae_grouped_deletion_bounds,
  title = {Grouped Deletion Bounds for Certifying TLS
    Optimality in Robot-World Hand-Eye Calibration},
  author = {Bae, Jaeyoung and Kim, Byeongjun and Kim, Kwangki},
  note = {Manuscript}
}
```

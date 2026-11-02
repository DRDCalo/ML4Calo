# ML Hub for Calorimetry

This repository aims to harmonise and advance the development of machine-learning (ML) approaches for
calorimetry within the DRDCalo collaboration. It provides a common place to develop, validate, share and
reuse ML models across detector concepts and studies in the Key4hep ecosystem. Individual developers and
groups are encouraged to contribute their models here.

## Organisation

Models are grouped by their application, with each model in its own directory carrying the corresponding
prefix:

| Application | Prefix |
|---|---|
| Particle identification | `PID_` |
| Energy regression | `EReg_` |
| High-level particle-flow reconstruction | `PFlow_` |

Available models:
- [`PID_PointNet`](PID_PointNet/): tiny PointNet for EM/hadronic shower identification.

Contributions covering further ML applications in calorimetry are welcome.

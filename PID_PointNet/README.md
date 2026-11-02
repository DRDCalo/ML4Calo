# PID_PointNet

This is a tiny PointNet model for the EM and hadronic shower identification in high granularity calorimeter.

We train this model in a sampling Si-Cu high granularity calorimeter prototype (simulation done by the [DRDCalo DD4hep tutorial](https://github.com/DRDCalo/SoftwareTutorials/tree/main/DD4hepTutorials)), and believe it's possible to extend the model application to any high granularity calorimeter. 

More details and hands-on guide can be found in Software Tutorial in [DRDCalo meeting in DESY](https://indico.cern.ch/event/1705549/contributions/7286238/attachments/3358742/6024416/20261008_DRDCalo_GaudiTutorial.pdf).

## Data

- Signal: `e-` showers (label 0, `electron`); background: `pi-` showers (label 1, `hadron`), 1-20 GeV.
- Input: PODIO ROOT files, hit collection `simplecaloRO` in tree `events`.
- Each event is a point cloud of hits with features `(x, y, z, E)`. Hits are sorted by energy and the
  leading 1024 are kept; shorter events are zero-padded and a mask marks the real hits. Events without
  hits are skipped.
- Features are standardized with mean/std computed on valid hits of the training set. The split is 4:1
  train/validation with a fixed seed.

## Model

```
points (N, 1024, 4) ──► shared MLP 4→32→64 (ReLU) ──► masked max pooling ──► MLP 64→32→2 ──► scores
mask   (N, 1024)    ─────────────────────────────────────┘
```

- `TinyPointNet` (`pointnet_model.py`): simplified PointNet (no T-Net, no BatchNorm), ~4.4k parameters,
  trained with cross entropy and Adam.
- `PointNetWithPreprocessing`: deployment wrapper with the normalization embedded, taking raw
  `(x, y, z, E)` and returning softmax scores `[electron, hadron]`.
- Exported formats: PyTorch state dict, TorchScript, and ONNX (opset 17, inputs `points`/`mask`,
  output `scores`, dynamic batch axis). 

## Files

| File | Content |
|---|---|
| `pointnet_model.py` | Network and deployment wrapper |
| `pointnet_util.py` | Data reading, normalization, export, evaluation helpers |
| `train_pointnet_simplecalo.py` | Training and export |
| `evaluate_pointnet_simplecalo.py` | Loss curve, ROC curve and AUC on the validation set |
| `pointnet_simplecalo.onnx` | Pre-trained ONNX model |

## Dependencies

Python 3 with `numpy`, `torch`, `onnx`, `uproot`, `scikit-learn` and `matplotlib`
(the last two only for evaluation).

All of them are provided by the Key4hep stack:

```bash
source /cvmfs/sw.hsf.org/key4hep/setup.sh
```

Checked with the Key4hep release 2026-04-08 (`x86_64-almalinux9-gcc14.2.0-opt`):

| Package | Version |
|---|---|
| Python | 3.13.8 |
| numpy | 2.2.6 |
| torch | 2.9.1 (CPU only) |
| onnx | 1.19.1 |
| uproot | 5.6.3 |
| scikit-learn | 1.7.0 |
| matplotlib | 3.10.7 |

Notes:
- `setup.sh` points to the latest release; package versions may change with future releases.
- In this stack, importing `onnx` and `onnxruntime` in the same Python process aborts with a protobuf
  error (`onnx-ml.proto` registered twice). The scripts here do not use `onnxruntime`.

## Usage

```bash
python train_pointnet_simplecalo.py \
    --signal-input simplecalo_e-_1-20GeV.root \
    --background-input simplecalo_pi-_1-20GeV.root \
    --output-dir pointnet_outputs
python evaluate_pointnet_simplecalo.py --artifact-dir pointnet_outputs
```

Run `python train_pointnet_simplecalo.py --help` for all options (events, epochs, batch size, learning
rate, seed). Outputs are written to `pointnet_outputs/`.

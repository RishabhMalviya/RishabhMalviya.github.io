---
layout: page
title: cfd-mlops
description: A distributed-training pipeline for a neural surrogate of automotive CFD — predicting surface pressure and wall shear stress on car meshes in a single forward pass.
img: assets/img/projects/cfd-mlops.png
importance: 1
github: https://github.com/RishabhMalviya/cfd-mlops
---

Automotive aerodynamics is traditionally evaluated with Computational Fluid Dynamics (CFD) simulation. These are highly accurate, but can take hours to days of solver time per design. This project trains a **surrogate neural network** that directly predicts the quantities a CFD solver would produce from a car's surface mesh — in a single forward pass.

Neural networks generalize well to inputs that are nearby in continuous space. Since the variability in car geometries is not especially large, a neural surrogate should extrapolate to unseen car geometries reasonably well.

Concretely, the model predicts, at every node of a vehicle's surface mesh:

- **Pressure** (`p`) — the dominant contributor to aerodynamic drag and lift
- **Wall shear stress** (3 components) — the surface friction that makes up the rest of the drag budget

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/cfd-mlops.png" title="Predicted pressure and wall shear stress fields over a car mesh" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Inputs and outputs: a surface mesh goes in, per-node pressure and wall shear stress come out.
</div>

## The model — Transolver

Training uses [Transolver](https://github.com/thuml/Transolver), a Transformer-based PDE solver that learns physical states over irregular meshes via a "physics-attention" mechanism.

The [`Car-Design-ShapeNetCar` variant](https://github.com/thuml/Transolver/tree/main/Car-Design-ShapeNetCar) is adapted here to surface-only DrivAerML data without modifying the model itself. Each mesh node is fed 6 geometric input channels — `[position(3), normals(3)]` — and the model predicts 4 output channels — `[wall shear stress(3), pressure(1)]`. This is the same sub-problem tackled by [GA-Field: Geometry Aware Vehicle Aerodynamics Field Prediction](https://arxiv.org/pdf/2602.20609).

Transolver's memory bottleneck is activation memory from large mesh inputs, so training is parallelized across multiple GPUs with **PyTorch DDP** driven by `torchrun`.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/projects/cfd-mlops-io.png" title="Input and output channels" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## The data — DrivAerML

[DrivAerML](https://caemldatasets.org/drivaerml/) is an open, high-fidelity CFD dataset of 500 parametrically-morphed DrivAer road-car geometries, generated with OpenFOAM using industrial-standard, validated workflows. It provides both surface (boundary) and volume flow-field data — pressure, wall shear stress, velocity, forces and moments — released under CC BY-SA 4.0. This project uses the surface (`boundary_*.vtp`) data.

Preprocessing decimates and caches the raw runs ahead of training, so the data pipeline isn't in the critical path of the GPU loop:

```bash
make dataset CACHE_DIR=data/drivaer_data/cache_full DECIMATE=0
```

## Training

The environment is managed with [`uv`](https://docs.astral.sh/uv/):

```bash
# Single-GPU training
python src/train.py --data_dir data/drivaer_data --cache_dir data/drivaer_data/cache

# Multi-GPU training (DDP)
torchrun --nproc_per_node=2 src/train.py --data_dir data/drivaer_data ...

# Resume from a checkpoint
python src/train.py ... --resume checkpoints/epoch_0020.pt
```

## Acknowledgements

- **Transolver** — Wu et al., [thuml/Transolver](https://github.com/thuml/Transolver)
- **DrivAerML** — [caemldatasets.org/drivaerml](https://caemldatasets.org/drivaerml/) (CC BY-SA 4.0)
